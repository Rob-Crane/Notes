# Python Modules and Packages 

## Modules
A module is a set of Python declarations in a file named `module_name.py`.  When an `import` statement is executed:
1. search for `.pyc` or `.py` file corresponding to name
1. execute any module-level statements
1. load module-level declarations into variable(s) in original scope

General guidance is follow `from package import specific_item` sytnax.

```python
import other_module
other_module.__name__ # returns "other_module"
__name__              # returns "my_module" or "__main__" if executed directly
```

Python caches compiled versions of source modules in the `__pycache__` directory under name `module.version.pyc`.  Can affect compilation with following options:
* `-0` removes assert statements
* `-00` removes assert statements and docstrings
`.pyc` files don’t run faster, they are just loaded faster.

## Packages
Packages structure Python’s module namespace into a dotted namespace.
an __init__.py file in a directory causes Python to treat directory as a package
```python
# widget/
#   __init__.py             # defines Widget class
#   calibrate.py            # defines Caliper class
#     sprocket/
#       __init__.py         # defines Cog class
#       maintain.py         # defines Grease class
from widget import Widget                     
from widget.calibrate import Caliper
from widget.sprocket import Cog
from widget.sprocket.maintain import Grease
```

`from package import *` imports items defined by list `__all__` in `__init__.py`.  This is considered the public interface of the module (see [example](https://github.com/globality-corp/microcosm/blob/develop/microcosm/api.py)).  If `__all__` is not defined, declarations in the module itself are exported (but not submodules or other `import` items that could have been included in `__all__`).
```python
# if __all__ not defined:
from widget import * # only Widget imported
```
Within a package hierarchy, can use relative imports.  Relative names are based on `__name__`.  If a module is being run directly, relative imports won’t work since `__name__` will be `__main__`.
```python
# in calibrate.py
import sprocket.maintain
# if it's being run as __main__:
import widget.sprocket.maintain
```
Packages support a special attribute `__path__` which is name of directory holding `__init__.py`.

## Module Loading
When a module is loaded with `import`, the interpreter looks in the file paths listed in `sys.path` which is populated when the interpreter loads.  The contents of `sys.path` are merged from:
 * The directory of the script being executed
 * The contents of PYTHONPATH environment variable
 * Standard library directories whose location is dependent on the `sys.prefix` and `sys.exec_prefix` variables.  Usually the standard library directories are something like (`/usr/lib/pythonX.X/`) but virtual environments modify this to something local so dependencies are all kept separate.  On Debian and derivatives, `dist-packages` directories present in `sys.path` are packages installed by system package managers.
 * Finally, additions made by the `site` module which are additional modules found in the `site-packages` directory.  `site-packages` is the target for manually built python packages built from source (probably distutils/siteutils running `python setup.py install`).

`setuptools` "Development Mode" let's you install a local source package but instead of installing into the `site-packages/` directory directly, a link is placed there pointing back to the source folder.  This let's local changes be made to the source without having to reinstall into the `site-packages` directory.

### References
https://stackoverflow.com/questions/4271494/what-sets-up-sys-path-with-python-and-when
https://stackoverflow.com/questions/897792/where-is-pythons-sys-path-initialized-from

##  Setuptools Module
`setuptools` is a set of enhancements to the `distutils` package and allows for easy building and distributing of python packages.  It created the original binary package format (the "egg") which has since been superceded by the PEP standardized wheel format.  `setuptools` doesn't natively produce wheels but the `wheel` package extends `setuptools` to produce wheels.

`setup.py` and the `setup()` function within it contain the main configuration for running, testing, and building the package.  Optionally a `setup.cfg` can augment configuration in `setup.py`.

### Entry Points
If the package you're writing is meant to extend an existing application, you're `setup.py` would register appropriate "entry-points" (which would be predefined by the application you're extending) and place them in the `entry_points` argument of `setup.py`.  Entry points have an "entry point group name" which would be provided by the framework/service you're extending.  For each group name, you can provide a specifier or list of specifiers.  Each specifier is a name/value where the name means something in the context of the entry point (file extension, protocol, whatever) and the value is the function/value in your code the framework should execute/take.

For example let's say there's a web-server framework you want to extend.  You want the package you're writing to add a new markdown renderer to the web application.  The web application conveniently defines an entrypoint called "bigwebserver.renderers".  The "bigserver.renderers" entry-point expects "name" value to be the file extension of whatever it's rendering (png, XML, whatever).  In `setup.py` of the markdown renderer package you're writing, you would include:
```python
entry_points={
    'bigwebserver.renderers' : ['.md = myrenderer:render_markdown', 
                                '.mkd = myrenderer:render_markdown']}
```

The most commonly used entry-point is the "console-scripts" entrypoint which registers a function as executable directly from the console command line.  This is done like:
```python
# Execute run_foo from console with 'runf'
entry_points={
    'console_scripts': ['runf = foo_module:run_foo']}
```

See the second reference for a more fleshed-out explanation.
### References
https://setuptools.readthedocs.io/en/latest/setuptools.html#dynamic-discovery-of-services-and-plugins
https://setuptools.readthedocs.io/en/latest/setuptools.html#dynamic-discovery-of-services-and-plugins
