# Rob's TensorFlow Notes

## Variables
* Maintain the state of the graph between calls to run
* Variables can be used like tensors as inputs for other Ops in the graph
* **collections** are named lists of tensors and other objects that may appear in different parts of the graph but have some common trait
* When a variable is created, it is added to collections `tf.GraphKeys.GLOBAL_VARIABLES` and `tf.GraphKeys.TRAINABLE_VARIABLES`
  * `GLOBAL_VARIABLES` is the collection of variables that can be shared across multiple devices
  * `TRAINABLE_VARIABLES` are variables for which TensorFlow calculates gradients
* Before a graph can be executed, variables must be initialized.  The function `global_variables_initializer()` adds an Op to initialize all variables.  By default, the `glorot_uniform_initializer` (Xavier initialization) is used.

## Static and Dynamic Shape
* A tensor's static shape is the shape provided when creating a tensor or inferred after defining an operation that creates a new tensor.  While TensorFlow can infer shape changes caused by ops, it may not be able to if an input has unknown dimensions (like a placeholder with an unknown dimension, maybe for batch size)
  * static shapes are known before a session run.  A tensor's static shape is obtained with `get_shape`.  A tensor's static shape, can be updated with the `set_shape` method.
  * dynamic shape is the true shape of the tensor.  Since a partially or undefined shape cannot be determined until session runtime, the `tf.shape(input)` op can be used to get tensor's dynamic shape.  The `tf.reshape` operation is used to reshape a tensor at runtime.

    ```python
    # shape has static shape (?, 32, 32, 3)
    batch_size = tf.shape(input)[0]  # this is a tensor value, could be used in tf.reshape op
    ```

## Naming Scope
* Names attributes are global in TensorFlow which can create problems:

    ```python
    def conv(input, kernel_shape, bias_shape):
      weights = tf.get_variable('weights'... # creates variable named 'weights'
      biases = tf.get_variable('biases'...  # creates variable named 'biases'

    x = conv(input1, [5, 5, 32, ...
    x = conv(x, [5, 5, 32, ...              # Fails! 'weights' and 'biases' reused!
    ```

* To simplify creation of unique global variable names, use variable scopes.  Actual variables names are then the `name` parameters appended to the scope name

    ```python
    with tf.variable_scope('conv1'):
    x = conv(input1, [5, 5, 32, ...     # variable names are 'conv1/weights' and 'conv1/biases'
    with tf.variable_scope('conv2'):
    x = conv(x, [5, 5, 32 ...
    ```
* In addition to variable scopes, name scopes created with `tf.name_scope` are a weaker form of scoping.  
  * Within a name scope, Ops and variables _created with the `tf.Variable` constructor_ will get names with the scope name prepended.  Variables created using `get_variable` **do not** have the name scope name prepended.
  * Variable scopes within name scopes will basically shadow the outer name scope.
  * Variable scopes take a parameter that affects the variable reuse behavior of `get_variable`
    * `reuse = True` puts scope in reuse mode.  Variables fetched by `get_variable` are expected to exist.
    * `reuse = tf.AUTO_REUSE` will create variables if they don't exist or return existing ones
    * `reuse = None` inherits the parent scope's `reuse` value.  
* When fetching a variable that already exists, `get_variable` does not require a `shape` argument.
* Finally, name scopes affect the Graph display of Tensorboard.  Items within the same name scope will be grouped in the graph as a single, expandable node by default.

## Sessions

* A `Session` represents a connection to the underlying C++ runtime.  Sessions are used as a context manager (in a Python `with` block) to manage the connection to the runtime and resources (like GPUs) held by the runtime.
* To execute operations, a list of "fetches" are passed to the `Session.run` function.  These fetches (tensors or variables) determine which subgraph of the existing graph is executed to retrieve the value of the fetches.  The values are returned as numpy arrays.
* `Session.run` can be passed a "feed dictionary" to set values of placeholders in the graph.  This is one form of providing input values to the graph (but not the recommended way).  The recommended way is to use an input pipline like the `Dataset` API described below.  These pipelines use additional ops at the beginning of the Graph to fetch the data off the disk (or from memory).
* If you did use feed dictionary / placeholders to provide input, each looping call to `Session.run` would have to use Python to provide input to the feed dictionary.  With a pipeline, each call to `Session.run` will use iterator ops in the graph to traverse an input dataset.

## Datasets
* `tf.data.Dataset` provides an API for representing an input pipeline, including obtaining and transforming data, as a structured collection of data.  After initializing a `Dataset`, data is accessed through use of an `Iterator`
* The API provides various ways of sourcing a `Dataset` from file or memory.  The simplest is to use `tf.data.Dataset.from_tensor_slices` or `tf.data.from_tensor`.  These accept tensors but also `numpy.ndarrays`, `Dataframe`, lists, or anything that can be implicitly turned into a tensor.  `from_tensor_slices` treats the first axis as samples and the rest as data.  `from_tensor` produces a `Dataset` with a single element.  (see example below)
  * When `from_tensor_slices` is passed a tuple of tensors, the first axis of each tensor is being treated as a samples.  This means they must match in size.  With `from_tensor`, the first axis is treated as a dimension of the sample input volume.
  * For data on disk, a `tf.data.TFRecordDataset` is recommended
* A dataset consists of a series of elements that are iterated over one at a time.  Each element has the same structure, which can be a single tuple, tuple of tensors, a nested tuple of tensors, a `collections.namedtuple`, or a dictionary.  The `tf.Tensor` objects of an element are called _components_.
* After defining the source, can apply various transformations which produce new `Dataset` objects.  A couple useful ones:
  * `Dataset.batch` pushes a new dimension on the front of each tensor of size `batch_size`
  * `Dataset.map` maps functions to each element in the Dataset.  If the structure of the element is a tuple of tensors (not nested), each tensor is treated as an argument to an argument to the function.  A nested tuple is passed as a single argument.
  * `Dataset.shuffle` randomly shuffles the elements of the `Dataset`.  To shuffle all elements of the `Dataset` make sure the `buffer_size` is big enough for the whole `Dataset`.
  * `repeat` repeats the dataset when the end is reached.  Can be limited with `count` argument 

```python
images = ... # shape is (1000, 32, 32, 3) -> 1000 32x32 RGB images
labels = ... # shape is (1000, 42)        -> 1000 labels of 42 classes

# from_tensor_slices treates first axis as the N samples.  Each element is an (image, label) tuple.
dataset = tf.data.Dataset.from_tensor_slices((images, labels)) # accepts tensors, tuples of tensors, tuples of dictionaries of tensors, etc.

# tuple elements are arguments to lambda function
dataset = dataset.map(lambda image, label: ...)

# each element is ((32,32,3), ())
element_default = dataset.make_one_shot_iterator().get_next()   

# element is tuple of tensor with shapes ((64, 32, 32, 3), (64,))
element_batched = dataset.batch(64).make_one_shot_iterator().get_next()

with tf.Session() as sess:
    # standard way to traverse entire dataset (assuming repeat() isn't used)
    while True:
        try:
            print(element_batched.shape) # prints shape of each batch
        except tf.errors.OutOfRangeError:
            break
```

* To iterate over a `Dataset` make an `Iterator`.  Each call to `get_next()` or each time a graph is run with a an input tensor coming from `get_next()`, another element will be pulled out of the `Dataset` and fed into the graph

## Estimators
* At 10,000 feet, Estimators let you have a workflow that looks something like:

```python
classifer = myMLModule.MyEstimator(
                         feature_columns = ...,
                         my_param = ...,         # maybe num_fc_layers
                         num_classes = ...)

# train models over a set of hyperparameters
for lr in learning_rates:

    classifier.train(input_fn = get_train_dataset, # common to use lambda like: input_fn = 
                     learning_rate = lr)           # lambda:get_train_dataset(X_train, y_train)

    validation_results = classifier.evaluate(input_fn = get_valid_dataset)

...

# evaluate performance of our best choice
model_results = classifier.evaluate(input_fn = get_test_dataset)
```

### Input Functions

* A function passed to `input_fun` should return a tuple of:
  * a dictionary of feature names to Tensors of values
  * a Tensor of one or more labels
* `input_fun` should require no arguments.  A function's behavior can be modified (with arguments) as follows:

```python
classifier.train(input_fun=lambda: my_input_fn(batch_size))
```

and `my_input_fun` could look like:

```python
def my_input_fn(batch_size):
   dataset = tf.data.Dataset.from_tensor_slices(({'feat1': feat1s, `feat2', feat2s}, labels))
   dataset = dataset.batch(batch_size)
   return dataset.make_one_shot_iterator().get_next()
```

### Feature Columns and Model Functions
* After defining the input function(s), the next step is define the feature columns.  These are the intermediaries between raw data and Estimators.  This defines how the model uses each feature.  Each `tf.feature_column` idenfies:
  * a feature name (actually a `key` in the feature dictionary passed to model function)
  * type
  * any input pre-processing 
* The `feature_column` module provides a number of functions to convert input features into numerical representations suitable for a neural network.  The `feature_column` itself does not contain the data.  The feature name is the key used in a `features` dictionary passed into the model function.  The remainder provides the functionality to make the raw data suitable for input into a neural network.  For example:

```python

# enables a dataset of class name strings to become OHE input data
vocabulary_feature_column=
    tf.feature_column.categorical_column_with_vocabulary_file(
        key = 'product_class',
        vocabulary_file = 'product_classes.txt', # text file of 120 categories
        vocabulary = 120)

...

    # features['product_class'] is tensor of ['electronics', 'grocery', ...]
    def my_model(features, labels, mode, params):
        # feature columns defines how features are turned into input
        in_layer = tf.feature_column.input_layer(features, params['feature_columns'])

```

* `params` passed to model function above can include more than just feature columns and can include parameters that alter the structure of the network.  For example a `hidden_units` parameter could define size of hidden layers.  `n_classes` can define the number of output values.  The model function itself is not called directly by a client (like `train`, `evaluate`, or `predict`) so `params` should be passed to the Estimator's constructor.
* `mode` affects how the Estimator framework invokes the model function.  The model function is called by one of three methods that correspond to the three mode values:
  * `ModeKeys.TRAIN` called by `train()`
  * `ModeKeys.EVAL` called by `evaluate()`
  * `ModeKeys.PREDICT` called by `predict()`
* The input layer to the model can be generated using `feature_column.input_layer` that converts the features tensor into numerical input tensors for the NN
* Subsequent hidden layers can be modified by `params` passed to input function
* For a dense output layer, `units` should equal `params['n_classes']`

### Training, Evaluation, and Prediction

* The model function is called in three different ways but must return an instance of `EstimatorSpec`.  `EstimatorSpec` contains a reference to the `ModeKeys` value passed into the model function and a flexible set of types that will differ for each mode it is called with.
  * `predict` calls the modle function to return an `EstimatorSpec` with predictions.  The model must already be trained.  The trained model is stored in `model_dir` which is defined when the Estimator is instantiated.  `EstimatorSpec` contains a dictionary, `predictions`, that contains  the class labels, probabilities, and logits.
  * The `EstimatorSpec` returned by `evaluate`'s model function should contain the loss value and can optionaly contain a dictionary, `eval_metric_ops` that contains more metrics.  A common, optional, metric is to return an accuracy value.  `tf.metrics` module contains functions for common metrics.
  * In the `train` function the model function is used to calculate a loss over the current set of parameters; and to define a training op.  The `train` function itself uses the returned values to execute model training
    * Optimization functions often use `global_step` parameters for limiting execution time of training function.  TensorBoard also relies on this value.  The value can be obtained using `tf.train.get_global_step`.

## TensorBoard 
### Summary Operations
* Summaries ar a way to export condensed information about a model
* Graph nodes of interest are annoted with `tf.summary` ops which then export information at runtime.
* Summary ops are given a `tag` which should be meaningful
* TensorFlow ops don't do anything unless explicitly run or a dependent op is explicitly run.  Since no ops depend on summary ops, they **must be** explicitly run.  To make this convenient, the `tf.summary.merge_all` combines summaries into a single op that generates all the data.
* Running the ops generates a serialized `Summary` protobuf object.  To write this to disk, pass to `add_summary` method of a `tf.summary.FileWriter` object
* Running the merged summary op every step is likely to give you far more information (and consume far more disk space) than you need.  Should run every _n_ steps instead.
* In addition to summary information obtained from `add_summary`, can also use `add_run_metadata` to get timing and memory information

### Graph Visualization
* Visualization uses hierarchy of naming scopes to organize the graph and by default, only displays the top of the hierarchy.
* TensorFlow graphs have two kinds of connections, _data dependencies_ and _control dependencies_.  
  * Data dependencies are shown as solid arrows
  * Control dependencies are shown as dotted lines
* To reduce clutter,  high-degree nodes are separated to an _auxiliary_ area to the right of the main graph.  The connection is indicated with a small node icon.
* _series collapsing_ is a clutter reducing mechanism that stacks nodes with names that only differ with a number at the end and that have isomorphic structures.
