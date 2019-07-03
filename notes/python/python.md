# Python Notes
1. [Argument Unpacking](#argument-and-unpacking)
1. [String Formatting](#string-formatting)
1. [List Comprehensions](#list-comprehensions)
1. [Iterators and Generators](#iterators-and-generators)
1. [Decorators](#decorators)
1. [Docstrings](#docstrings)
1. [Namespaces](#namespaces)
1. [Classes](#classes)
1. [Inheritance](#inheritance)

## General Language
### Assignment and Naming
All right hand side operations are completed before any assignment:
```
a, b = b, a + b     # first adds a+b, then assigns b to a and result to b
```
When a comparison operation is assigned to a variable, the evaluation terminates when it's evaluated true or end of the expression if it is false.  The expression is evaluated left to right, recursively.  The assignment made is the last expression evaluated true.
```python
a = (0 or None) and 2 # a=2
a (1 or 0) or 2 # a is 1 (second expression not evaluated)
```
The `_` variable stores last value in interactive mode.
```python
>>> 3**2 # power operator
9
>>> _ // 2 # floor division
6
```
At any point, the following scopes could exist (and are searched in this order):
1. Current (innermost) scope (usually function, could be class, or global)
1. Scopes of enclosing functions, starting at nearest.
1. Global (module-level)
1. Built-in names

By default, variables in enclosing (outside) scopes are read-only.  An attempt to assign would create a new copy.  Two options for assigning to a variable in an enclosing scope:
* `global` binds variables to the module-level (global) scope
* `nonlocal` binds to variable in enclosing scope but NOT in module scope

```python
a = 1
def f():
    global a
    a = 2
    def g():
        a = 3
        def h():
            nonlocal a
            a = 9 # binds to g.a
        h()
        print(f"g.a: {a}")
    g()
    print(f"f.a: {a}")
f()
print(f"a: {a}")
# Prints:
# g.a: 9
# f.a: 2
# a: 2
```
Assignments, function definitions, and `import` statements, operations that introduce names to the current scope, always default to the local scope.
```python
def load_module():
    global pdb # Otherwise pdb only introduced to function scope!
    import pdb
load_module()
pdb.set_trace()
```
`dir()`, `local()`, and `globals()` will gives views of the dictionaries of current variables.  Statements executed at the top level interpreter invocation, either command line or from a directly-executed script file, form a module called `__main__`.
### Control Flow
#### `for` and `if`
Looping through a list doesn’t not implicitly make a copy of that sequence.  If modifying the sequence, it is recommended you make a copy.  This is done easily with the slice operator.
```python
for w in words[:]:  # makes a copy of words with slice operator
    …               # modifications to words in loop won’t affect words[:]
```
#### `else` in Different Contexts
`for` and `while` loops can have an `else` statement that is executed on natural termination of loop (not executed if terminated by `break`)
```python
for i in range(3):
    if i % 2 == 0:
        break
else:
    print("never executed")
```
Similarly, a `try` statement can have an `else` statement that is executed when no exception occurs
```python
    try:
        process(file)
    except FileNotFoundError:
        ...
    except FileExistsError:
        ...
    else:
        print("successfully opened")
```
###  Function
All functions return something, if something isn’t explicitly returned, None is returned.  
```python
def f():
    print("f()")
print(f())
# prints:
# f()
# None
```
A class with `__call__()` defined makes class objects functors.
```python
class Foo:
    def__call__(a):
        print(a)
f = Foo(); f(10)
```
Function default arguments are evaluated at the point of definition and are only evaluated once.
```python
ll = []
def foo(a=len(ll)):
    print(a)
ll.append(1)
foo() # prints 0

# One way to address:
def bar(a=None):
    if a is None:
        a=len(ll)
    print(a)
ll.append(2)
bar() # prints 2
```
#### Arguments and  Unpacking
All arguments can be passed as keyword arguments and the order isn’t important, as long as all arguments without default values are passed.
```python
def foo(num1, num2, is_w=True):
    ...
foo(1,2,True) # okay
foo(is_valid = False, num2=1, num1=2) # okay
foo(is_valid = False, num2=1) # Error: "missing 1 required positional argument..."
```
Positional arguments can be unpacked from a tuple or list with `*` operator.
```python
def foo(a,b,c,d):
    ...
some = (1,2)
foo(*some, 3, 4)
rest = [3,4]
foo(*some, *rest)
```
Although positional arguments cannot follow keyword arguments, positional unpacking _can_ follow keyword arguments.  The order arguments are found is:
1. Positional arguments
1. Positional unpacking
1. Keyword arguments
1. Keyword unpacking

```python
def foo(a, b, c, d):
    ...
foo(c=2, *(0,1), *(3,)) # okay: 0,1 unpacked into a,b; 3, unpacked into d; 2 into c
foo(0, d=3, *(1,2)) # okay: 0 into a; 1,2 unpacked into b,c; 3 into d
foo(c=2, 0, 1) # SyntaxError: positional argument follows keyword argument
```
Keyword arguments can be unpacked from dictionaries using the `**` operator.  Positional unpacking using `*` operator must precede keyword unpacking.
```python
def foo(a, b, c, d=4):
    ...
args = {'c':3, 'd':5}
foo(1,2,**args)
```
After unpacking, a `*parameter` receives excess positional arguments that don't map to formal arguments.
```python
def foo(a, b, *extra):
    print(extra)
foo(1,2,3,4) #prints (3,4)
```
This syntax can be used to force arguments to be keyword arguments.
```python
def foo(a, b, *ignore, key=None):
    ...
foo(1,2,3,key=4)
def bar(a, b, *, key=None) # Special syntax...
    ...
bar(1,2,3,key=4) # TypeError: bar() takes 2 positional arguments...
```
A `**parameter` receives any keyword arguments that don't map to formal arguments.
```python
def foo(a, b, **extra):
    print('a' in extra) # False
    print('b' in extra) # False
    print('c' in extra) # True
    print('d' in extra) # True
foo(3, b=5, c=6, d=7): #kwargs MUST come after positional arguments
```
### Strings
To ignore \ prefaced special characters use raw strings (r’...’)
```python
print(r"\n"*3) # prints "\n\n\n"
print(r"\n" + "\n")    # prints "\n
                       #        "
```
For string literals that span multiple lines, use triple quotes.  Newlines are included in these strings.  End lines with \ to ignore it.
```python
str = """This string
spans multiple\
 lines.""
## This string
## spans muliple lines.
```
Adjacent strings are automatically concatenated.
```python
s = "this is" "one"\
" string"
```
Put several strings in parentheses to join, strings can be separated by whitespace.
```python
my_string =  (“Python is”
                   “ great”)
```
Strings can be sliced like any sequence.
```python
word = "foo"
word[:2] + word[2:] # "fo" + "o"
```
Out of index _slicing_ is handled gracefully:
```python
word = "foobar"
word[2:20] # "obar"
word[10] # IndexError: string index...
```
Strings are immutable.
```python
 `word[3]=g` is not possible.
```
`str()` and `repr()` built-in functions both create strings from object argument
* `str()` attempts to return a human readable representation
* `repr()` (attempts to) return a string the interpreter could parse to return the original value
* `repr()` and `eval()` try to be complementary operations
* Behavior detremined by a class' special member functions `__str__` and `__repr__`

#### String Formatting
Old method is C `printf` inspired style. See [docs](https://docs.python.org/3/library/stdtypes.html#printf-style-string-formatting) for `printf` style operations.
```python
"Integer value: %d" % 2011
"My name is %s" % "Bob"
"This year is: %(year)d" % {'year' : 2018}
"Pi is: %(pie)04.2f" % {'pie' : math.pi} # Pi is: 03.14
"Object: %s" % obj  # use obj.__str__()
"Object: %r" % obj # use obj.__repr()
```
More modern method is to use "formatted string literals" or the string `format` member function.  Format specification syntax is similar except '%' is replace with ':' and the expression is surrounded by curly braces.  The syntax follows:

`{[field name] [!conversion] [:format string]}`.

##### Conversion Field
The conversion field describes which class function is used to convert the object to a string:
* `!s` - use `__str()__` (default and can be omitted)
* `!a` - use `__ascii()__`
* `!r` - use `__repr()__`
```python
val = get_foo()
f"The value is {val!r}" # "The value is " + val.__repr__()
```
##### Format String
The format string, following the `:`, is complicated and is fully described [here](https://docs.python.org/3/library/string.html#format-specification-mini-language).  A quick summary:
* `fill` (any char) controls what character is used to fill extra space if a minimum width is specified.
* `align` (<, ^, >, =) controls justification, the last value used for padding after sign for numeric types.
* `sign` (+, -, ' ') controls positive or negative sign for numeric types.
* `#` controls `0b`, `0o, `0x` printing for binary, octal, hexidecimal numbers.
* `0` enables "sign aware zero padding".  Equivalent to '0' for padding with `=` alignment.
* `width` is minimum field width.
* `grouping options` (',' or '\_') groups decimal values into thousandths with that character.
* `precision` (first number following `.`) How many digits displayed after decimal.
* `type` ('b', 'c', 'd', 'o', ...) Display number as binary, char, decimal, octal, etc.
```python
"My weight change has been: {:+05.1f}".format(change) # ...has been +02.1
"My age in binary: {:+#b}".format(29) # ...binary: +0b11100
f"value: {math.pi:05.2f} # "value: 003.14"
f"value: {math.pi:05.2g} # "value: 0003.1"
f"value: {math.pi:5.2g} # "value:     3.1"
```

#### Formatted String Literals and `format`

Formatted string literals are prepended with 'f' and are passed values from context.
```python
val = 2.22222222
f"The value is {val:05.2f}" #The value is 02.22
w = get_widget()
val_ascii_str = f"{w!a}"
f"Double {{curly braces}} are replaced with one"    # "Double {curly braces} ..."
```
String's `format()` creates formatted strings:
 ```python
 “{} is {}”.format(“sky”, “blue”) # "sky is blue"
 “{1} is {0}”.format(“blue”, “sky”) # using position arguments
 “{b} is {a}”.format(a=“blue”, b=“sky”) # using keyword arguments    
 ```
Can pass variables in a dictionary and reference from format string:
```python
dict = {‘first’: ‘Rob’, ‘second’, ‘Crane’}
“first: {0[first]} second: {0[second]}”.format(dict)
```
Or use keyword unpacking:
```python
“first: {first} second: {second}”.format(**dict)
```
This can be neatly combined with built-in `vars()` function that returns a dictionary of local variables!  Easily print locals!

###  Lists  and Tuples
`extend()` is like `append()` but takes an iterable.
```python
l1 = [1,2,3]
l2 = [4,5,6]
l1[:].append(l2) # [1,2,3,[4,5,6]]
l1.extend(l2) # [1,2,3,4,5,6]
```
`pop()` and `append()` can be used to implement a stack (and are optimized for it).  For FIFO, use `collections.queue`.
```python
`list.index(x)` returns an index value of x in that list with optional start/end limits.  `list.count(x)` counts instances of x in list.
```python
l = [0,1,2,3,4,0]
l.pop(l.index(3)) # same as l.remove(3) except here it's returned
l.count(0) # 2
l.pop(l.index(0, -1)) # removes last 0
l.count(0) # 1
```
`enumerate()` applied to a sequence will return index, value pairs 
```python
for i, (x, y) in enumerate(set_of_points):
    print(f"Ind: {i}, X: {x}, y: {y}")
```
`zip()` takes sequences as arguments and returns an iterator that pairs entries from each.
```python
for n, c in zip([1,2,3], ['a', 'b', 'c']:
    ... # n=1, c='a', etc.
```
`reversed()` returns reverse iterator.
```python
l = [1,2,3]
print(list(reversed(l))) # [3,2,1]
# Can be called on object if class defines:
#   * __reversed__()
#   * __len__() AND __getitem__() (with integer args starting at 0)
for o in reversed(my_obj):
    ...
```
`sorted()` returns sorted sequence and is stable.  Takes optional `key` argument which is applied to each entry to get the sort-key.
```python
l=['d', 'D', 'a', 'b', 'A']
print(list(sorted(l, key=str.lower))) # ['a', 'A', 'b', 'd', 'D']
```
Sequences of the same type can be compared with lexicographical ordering - first, the first two items are compared, if different, that difference is returned as result.  If equal, the next two are compared the same way, etc.  If one list runs out first, the shorter is less.
```python
[1,2] < [1,2,3] # true
[2,1] < [1,2,3] # false
```
`del` operator can be applied to list indexes or slices and doesn’t return a value like pop

####  List Comprehensions
List comprehensions are a concise way to create a list in a single line of code.  Common application is to create a new list from another
```python
squares = []
for x in range(10):
    squares.append(x**2)
```
Is equivalent to:
```python
squares = list(map(lambda x: x**2, range(10)))
```
Is equivalent to:
```python
squares = [x**2 for x in range(10)]
```
A list comprehension consists of at least one `for` clause and then `for` and `if` clauses.  The order of the clauses is equivalent to walking from outer to inner blocks.  For example:
```python
diff_sums = [ (abs(x-y), x+y) for x in [1, 4, 5, 9] if x%2 == 0 for y in [4, 5, 6] ]  # note: tuple expressions MUST be parenthesized
```
Is equivalent to:
```python
diff_sums = []
for x in [1, 4, 5, 9]:
    if x%2 == 0:
        for y in [4, 5, 6]:
            diff_sums.append( (abs(x-y), x+y) )
```
The initial expression can be arbitrary, can be itself a list comprehension (nested list comprehensions)
```python
# transposes 4x4 matrix
[ [row[i] for row in matrix] for i in range(4) ]
```
Which is equivalent to:
```python
new_mat = []
for i in range(4):
    new_mat.append([row[i] for row in matrix])
```
Which is equivalent to:
```python
new_mat = []
for i in range(4):
    new_row = []
    for row in matrix:
        new_row.append(row[i])
    new_mat.append(new_row)
```
...or it can be a 'ternary' operator:
```python
['o' if num%2 else 'e' for num in num_list]
```
While Lists are mutable and usually contain homogenous data types, tuples are immutable and often contain heterogenous data, often obtained by unpacking.
```python
tup = ()        # empty tuple
tup = (item,)   # single-item tuple
tup = (item)    # NOT a tuple
```
### Sets and Dictionaries
#### Sets
Sets are also a basic data type that support mathematical set operations like union, intersection, etc.
```python
a = {0,1,2}
b = {2,3}
a.union(b) # returns {0,1,2,3}
a.update(b) # a is {0,1,2,3}
a.intersect(b) # returns {2,3}
a.intersect_update(b) # a is now {2,3}
{0,1}.isdisjoint({2,3}) # returns true
{0,1}.issubset({1,2}) # returns false
```
An empty set is created by `set()`.  `{}` is an empty dictionary
```python
set().issubset({1,2}) # returns true
{}.issubset({1,2}) # Attribute Error!
```
Set comprehensions can be made just like list comprehensions (except using curly brackets).
```python
l = [0,0,1,1,2,3]
{i for i in l} # {1,2,3}
```
#### Dictionaries
Treating a dictionary like a sequence acts on its keys.
```python
d = {0:'a', 1:'b', 2:'c'}
l = []
for i in sorted(d):
    l.append(i)
# l is [0,1,2]
```
Keys cannot contain any mutable types, even if key type is itself immutable.
```python
d = { (12,[13]) : "values"} # TypeError: unhashable type: list
```
dict() constructor can build dictionaries from sequences of key-value pairs:
```python
student_dict = dict(
    [(0, 'janet'),
     (1, 'marsha'),
     (2, 'billy')])
student_dict[3] = 'graham'
```
(key, value) pairs can be retrieved simultaneously with iterator `dictionary.items()`

### Iterators and Generators
Calling `iter` on a container returns an iterator object.  An iterator object returns the elements of the container, one at a time, it's member function `__next__` is called:
```python
l = [1,2,3,4,5]
i = iter(l)
i.__next__() # '1'
i.__next__() # '2'
next(i)      # '3' (built-in function calls __next__)
```
When the end of the container is reached, a `StopIteration` exception is raised.  To define a custom iterable, must define function `__iter__` which returns the object defining `__next__`.
```python
class TwoList:
    def __init__(self, l1, l2):
        self.l1, self.l2 = l1, l2

    def __iter__(self):
        return TwoListIter(self)

class TwoListIter:
    def __init__(self, tl):
        self.i1, self.i2 = 0,0
        self.tl = tl
    def __next__(self):
        if self.i1 < len(self.tl.l1):
            self.i1+=1
            return self.tl.l1[self.i1-1]
        elif self.i2 < len(self.tl.l2):
            self.i2+=1
            return self.tl.l2[self.i2-1]
        else:
            raise StopIteration

tl = TwoList([1,2],[3,4])
for i in tl:
    print(i), # 1,2,3,4

```

A generator is a slick tool for generating iterator objects.  They take the form of a function that return a value with the `yield` keyword instead of `return`.  Each time `__next__` is called on the generator, the function resumes from where last yielded.  If the function runs out of executable statements, the `StopIteration` exception is raised.
```python
def fib(max_val):
    i,j = 1,1
    yield i
    while j < max_val:
        yield j
        new_i, j = j, i+j
        i = new_i
for f in fib(50):
    print(f)
```
A generator expression uses syntax like a list comprehension without square brackets:
```python
sum(i*i for i in range(10)) # useful in situations when computed value consumed immediately 
```

### Decorators
Decorators are syntactic sugar for functions that modify functions.
```python
def print_before_after(func):
    def wrapper(*args, **kwargs):   # new function should take any arg that could be passed to func
        print("before")
        ret = func(*args, **kwargs) # forward the args to original func
        print("after")
        return ret
    return wrapper

def add(a, b):
    return a+b
add = print_before_after(add)

# Is equivalent to:

@print_before_after
def add(a, b):
    return a+b
```
### Errors and Exceptions
An `except` clause can name multiple exceptions as a parenthesized tuple:
```python
try:
    complicated_stuff()
except (RuntimeError, TypeError):
    recover()
```
`Exception` class can be extended, except clauses will catch all children of `Exception` class
```python
class MyError(Exception):
    ...
try:
    raise MyError()
except Exception:
    print("silly example")
```
Exceptions can be thrown with arguments like `MyError(‘foo’, ‘spam’)`
```python
class MyError(Exception):
    def __init__(self,arg):
        self.arg=arg
    def info(self):
        print(self.arg)
    ...
try:
    raise MyError(2011)
except MyError as ex: # bind exception to ex
    print(ex.info())
```
Most Exceptions are given names that end in "Error" by convention.

## File I/O
### Read and Write
`File` object created using call to `open(filename, mode)` where `mode` can be:
* ‘r’ - read-only (default)
* ‘w’ - write-only
* ‘r+’ - read and write
* ‘a’ - append
* ‘b’ appended to mode opens file in binary mode and data is written to file in byte objects.

If ‘b’ isn’t specified, file is open in text mode.  `open` takes an optional `encoding` argument, meaningful only for text mode, which defaults to `locale.getpreferredencoding()`.
```python
with open("file.slog", 'rb') as f:
    ...
```
Newline characters are translated behind the scenes to platform dependent encoding but always appear in Python as ‘\n’.  This can be dangerous if opening a binary file which contains ASCII newline characters.  Could be automatically replaced by locale-encoded newlines unless file.  Important to open binary files in binary mode (not default)!

`read(size)` reads data and returns a string in text mode or bytes in binary mode.  It reads at most size bytes up to the end of file
```python
with open("big_file.txt") as f:
    hundred_bytes = f.read(100)
    f.read() # reads whole file...MemoryError!
with open("small_file.txt") as f:
    f.read()
    end = f.read(100) # end is ''
```
Lines of a file can be moved into a list or iterated over:
```python
with open("item_list.txt") as f:
    for line in f:
        process(line)
    chunk = f.readLines(1000)  # read about 1000 bytes into a list of lines
```
To write other objects to file, they must be converted to a string (for writing to file in text mode) or to bytes (for writing in binary mode).
```python
class Widget:
    def __init__(self, a, b):
        self.a=a; self.b=b
    def __str__(self):
        return f"Widget: ({self.a}, {self.b})
w = Widget(1,2)
with open('widget_fie.wdg', 'w') as f:
    f.write(w) # writes "Widget: (1,2)" to file
```
To write an object to a binary file, that object must comply with the [buffer protocol](https://docs.python.org/3/c-api/buffer.html) which exposes the underlying object memory.

### Position in File
`tell()` returns the current stream position.
```python
with open('text.txt') as f:
    f.readline()
    f.tell()    # returns number of bytes of first line
```
`seek(offset, whence)` moves position in file to offset relative to `whence`:
* `io.SEEK_SET` - from start of stream (default)
* `io.SEEK_CUR` - from current position in stream
* `io.SEEK_END` - to end of stream (offset must be zero)

Text files support only `io.SEEK_SET`.
```python
with open('text.txt', 'rb') as f:
    f.seek(10) # move forward ten bytes
    f.seek(5, io.SEEK_CUR) # move forward another five
    f.seek(0, io.SEEK_END) # move to end of stream
```
### Serialization to File
To save more complicated data to file, `json` module serializes complex data types like nested lists and dictionaries.
```python
d = {0 : 'a', 1 : 'b', '2' : (1,2,3)}
s = json.dumps(d) # returns "{0:'a',1:'b',2:[1,2,3]}"
with open('out.json', 'w') as f:
    json.dump(d, f) # write JSON to file


with open('out.json', 'r') as f:
    orig = json.load(f)
orig2 = json.loads(s) 
```
While json module can handle lists and dictionaries, `pickle` module handles arbitrarily complex objects but is specific to Python.
```python
w = Widget()
with open('w.pkl' 'wb') as f:
    pickle.dump(w,f)

with open('w.pkl' 'rb') as f:
    w2 = pickle.load(f)
```
## Object Oriented Python
### Classes
Class definitions have to be executed to have any effect, could theoretically place a class definition in an if statement
```python
# Not really any reason to do this...
if __name__ == '__main__':
    class AppRunner:
        ...
```
Two operations supported by class object:
* "Attribute References" are just qualified references to the attributes of the class, functions or other declarations in the class.  They are made using the class object created by the class definition.
```python
class MyClass:
    """Example class""
    i = 2011
    def foo():
        pass
MyClass.i # 2011
MyClass.foo()
MyClass.__doc__ # "Example Class"
```
* An "instantiation operation" uses function notation and creates a new instance of the class.
```python
class MyClass:
    def __init__(self, a): # __init__ invoked automatically instantiation operation
        self.a = a
c = MyClass('foo')
```
An _instance object_ of a class has it's own unique attributes created by assignment (often in `__init__`).
```python
c = MyClass('bar')
print(c.a) # 'bar'
c.b = 'other' # assign instance object another attribute
```
An instance object can also call _methods_ which are functions that "belong to" objects.  All __function__ class attributes define methods.  Methods are themselves objects.
```python
class A:
    def foo(self):
        ...
a = A()
a_f = a.foo
...
a_f()
```
Because a method is executed by automatically filling a reference to it's object as the first argument, the above example method is equivalent to:
```python
A.foo(a)
```

#### Attribute Lookup
Python objects each have a dictionary-like attribute `__dict__` which contains the mapping of object attributes to value.
```python
class A:
    x=11
    def __init__(self, y):
        self.y=y

a = A(12)
print(a.__dict__)    # {'y' : 12}
```
Since classes are object's too, they have a `__dict__` attribute too (that above would map `{'x' : 11}`).  Unlike objects, class `__dict__` attributes aren't regular Python dictionaries but are instead `dictproxy` types, a _dictionary-like_ structure with additional restrictions in place for performance/stability reasons.

When an object attribute access is attempted, the default behavior is implemented in `object.__getattribute__(self, attr)`.  This method looks up `attr` in `self.__dict__` (the object's dictionary) and in the `dictproxy`'s of the object's class inheritance tree. and either returns the value found or raises `AttributeError`.  A class can override `__getattribute__`:
```python
class MyClass:
    def __init__(self, x):
        self.x=x
    def __getattribute__(self, attr):
        print('in MyClass.__getattribute__')
        return object.__getattribute__(self, attr)
c = MyClass(11)
print(c.x)

# Prints:
# 'in MyClass.__getattribute__'
# 11
```
It is noted in the python reference documentation that, in an overriden `__getattribute__` method, attribute accesses should directly call a base class `__getattribute__` (which would likely just call `object.__getattribute__` unless another class in the object's hierarchy had overridden it too).
```python
class MyClass:
    def __init__(self, x):
        self.x=x
    def __getattribute__(self, attr):
        print('in MyClass.__getattribute__')
        return self.x
c = MyClass(11)
print(c.x)

# Prints:
# 'in MyClass.__getattribute__'
# 'in MyClass.__getattribute__'
# 'in MyClass.__getattribute__'
# 'in MyClass.__getattribute__'
# 'in MyClass.__getattribute__'
# ...
```

If `__getattribute__` raises an `AttributeError`, a second method, `__getattr__(self, attr)` is called.  It is __not__ called when an attribute access is successful.  This normally does nothing but can be overridden to provide a default value.  The reason to have both `__getattr__` and `__getattribute__` is that it lets the user-implemented `__getattr__` access instance attributes through `object.attribute` syntax (using `__getattribute__`) without causing the recursion problem described above.  For example:
```python
class A:
    def __init__(self, default):
        self.default=default
    def __getattr__(self, attr):
        return self.default # uses instance attribute 
                            # (through __getattribute__) to compute return
a=A(-1)
print(a.b)   # prints -1
```
##### Super
`super(type, object)` returns a _proxy object_ that that delegates method calls to a parent or sibling class.  It returns an object that conducts method resolution look-up (in `object.__getattribute__`) in the same order as `object` but that skips `type`.  Useful for calling an overridden method:
```python
class Widget:
    def process(self, config):
        do_work(config)

class Sprocket(Widget):
    def process(self):
        super(Sprocket, self).process(get_config())
```

#### Descriptors
A descriptor is a mechanism to customize the behavior of accessing an object or class attribute.
```python
record.email = 'foo@foo.com' # can throw exception if ill-formed
del(record.id) # could make read-only and throw exception here
print(record.address) # could add formatting
```
A descriptor is created by adding special member functions to the _attribute's_ class.
```python
class Attr:
    def __get__(self, instance, owner):
        return self._process_out(instance.a)
    def __set__(self, instance, value):
        instance.a = self._process_in(value)
    def _process_out(value):  # do something to customize output
        ...
    def _process_in(value):   # do something to customize input
        ...
class Owner:
    a = Attr()      # <- set as a CLASS attribute
o = Owner()
o.a = 'foo' # calls type(o).__dict__['a'].__set__(o, 'foo')
print(o.a) # calls type(o).__dict__['a'].__get__(o, type(o))
```
When calling `o.a` for an instance `o` of class `Owner`, the default behavior is to first check `o`'s `__dict__` for 'a' and that failing, check `Owner`'s `__dict__`.  It is at the class-level lookup that `a` is inspected for descriptor methods and why `a` must be a _class_ attribute.  This behavior is in `object.__getattribute__` so overriding `__getattribute__` risks short-circuiting this behavior.
```python
class Owner:
    a = Attr()
    def __getattribute__(self, attr): # overriding this eliminates any attribute behavior
        return self._process(self.__dict__[attr])
```
Attribute behavior also affects accessing the class attribute:
```python
print(Owner.a)      # calls Owner.__dict__['a'].get__(None, Owner)
Owner.a = 'bar'     # just overwrites Owner.a with a string
```
To summarize:
* If `__get__` defined:
  * `o.a` calls `__get__(a, o, Owner)`
  * `Owner.a` calls `__get__(a, None, Owner)`
* If `__set__` defined:
  * `o.a = value` calls `__set__(o, value)`
  * `Owner.a = value` overwrites 'a' in `Owner.__dict__`

##### Property Function
Properties use the descriptor mechanism to streamline the creation of descriptor behaviors.  The `property` function returns an attribute instance ready for assignment in a class definition.
```python
class Owner:
    def get_fn(self):
        ...
    def set_fn(self, val):
        ...
    a = property(get_fn, set_fn)

o = Owner():
o.a = "foo" # calls set_fn(o, "val")
```
This can be further streamlined using the `@property` decorator
```python
class Owner:
    @property
    def a(self):    # defines "getter" method and name of property
        ...
    @a.setter       # @{name of property}.setter
    def set_a(self, val):
        ...
o.a = "foo" # calls set_a()
print(o.a)  # calls a()
```

* __TODO__: classmethod and staticmethod decorators
* __TODO__: super

### Inheritance
All methods in Python are effectively virtual.
```python
class Animal:
    def __init__(self, name):
        self.name=name
    def make_noise(self):
        print(self.name + " says hakuna matata")
class Dog(Animal):
    def make_noise(self):
        print(self.name + " growls!")
class Cat(Animal):
    def make_noise(self):
        print(self.name + " hisses!")

def hear_noise(animal, no_worries=False):
    if no_worries:
        Animal.make_noise(animal) # Avoid the "virtual" behavior
    else:
        animal.make_noise()

d = Dog("Fido")
c = Cat("Whiskers")
hear_noise(d)
hear_noise(c, True)

```
* `isinstance(obj, className)` checks to see if `obj.__class__` is of type `className` __or derived__ from `className`
* `issubclass(derived, base)` checks to see if `derived` is a subclass of `base`

#### Name Mangling
Although there are no private variables, a common convention is to prefix underscore (`_spam`) for any attribute that is non public part of API.
```python
class Widget:
    _x, _y = 1, 1
    def _get_sum(self):
        return self._x + self._y
    def process(self):
        return self._get_sum()
```
Two underscores before identifier (`__spam`) invokes a mechanism called name-mangling where identifier is textually replaced with `_classname__spam`.  This can be used to prevent overidden methods and attributesfrom breaking a super class' _private_ methods and attributes:
```python
# dir(Widget) produces ['_Widget__get_sum', '_Widget__x', '_Widget__y', ...
class Widget:
    __x,__y = 1, 1
    def __get_sum(self):
        return self.__x + self.__y
    def process(self):
        return self.__get_sum()

class Sprocket(Widget):
    __x, __y, __z = 10, 10, 10
    def __get_sum(self):
        return self.__x + self.__y + self.__z
    def actualize(self):
        return self.__get_sum()

s = Sprocket()
print(s.process(), s.actualize())
```
In Python 3, all classes are subclasses of `object`.  In Python 2, classes _should_ inherit from `object` since that adds several Descriptor-enabled perks, additional static methods, and other perks.
### Metaclasses
__TODO__: metaclasses

##  Style
* If there are more lines in a document string, second line should be empty
* [PEP8](https://www.python.org/dev/peps/pep-0008/) is accepted Python style guide
* 4 space tabs, lines <= 79 characters
* CamelCase for classes, lower\_cases\_with\_underscores for functions and methods

###  docstrings
* If the first statement of a function body is a string literal, it is a docstring.  Automated tools can be used to generate online/printed documentation of your code
* First line is short, concise summary of object’s purpose.  Do not state name or type.  Should be a proper sentence and end in punctuation mark.
* [PEP257](https://www.python.org/dev/peps/pep-0257/) describes best practices for writing docstrings.
* Google style guide has [good docstring example](https://google.github.io/styleguide/pyguide.html?showone=Comments#Comments).

