# Scikit-Learn Notes

## Data-Preprocessing
 * `model_selection.train_test_split()` shuffles and splits the a dataset into training and testing subsets.  Input is a sequence of arrays (of data) followed by keyword arg options.

```python
X_train, X_test, y_train, y_test = train_test_split(
                full_features, full_y, 
                test_size=0.2, random_state=rand_state)
```

 * StandardScaler normalizes data and can be used to apply a common normalization on training and test feature sets.  StandardScaler should only be fit on the training data to ensure no information from the test set is leaked to the model training.
```python
X_scaler = StandardScaler().fit(X_train)
X_train_sc = X_scaler.transform(X_train)
X_test_sc = X_scaler.transform(X_test)
```

## Parameter Tuning
 * `model_selection.GridSearchCV` simplifies evaluating classifiers over ranges of hyperparameter values.
```python
param_grid = {'C' : [1.0, 0.1, 0.001],
            'kernel' : ['linear', 'rbf']}
print('beginning grid search')
clf = GridSearchCV(svc, param_grid, verbose=1)
clf.fit(X_train_sc[0:config.M], y_train[0:config.M])

# retrieve best results
svc = clf.best_estimator
params = clf.best_params_
score = clf.best_score_
```

## Support Vector Machines
 * SVM
