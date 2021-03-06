---
layout: post
keywords: python, executable script, __main__
description: 记录对 __main__ 的理解
title: "可执行的脚本"
categories: [python]
tags: [python]
group: archive
icon: compass
---
{% include mathsyouth/setup %}


### A module's `"__main__"`

When the Python interpreter reads a module, it executes all of the code found in it. Before executing the code, it will define a few special variables. For example, if the python interpreter is running that module as the main program, it sets the special `__name__` variable to have a value `"__main__"`. After setting up the special variables, it will execute the `import` statement and load that module. If this file is being imported from another module,  `__name__` will be set to the module's name.

One of the reasons for doing this is that sometimes you write a module (a `.py` file) where it can be executed directly. Alternatively, it can also be imported and used in another module. By doing the main check, you can have that code only execute when you want to run the module as a program and not have it execute when someone just wants to import your module and call your functions themselves.


### The module `__main__.py`

The module `__main__.py` is not quite the same as an executable script, but it allows a package to be executed via `python -m`. For example, `passacre/__main__.py ` means that one can execute `python -m passacre` and the `__main__.py` file will be executed. [PEP 338](https://www.python.org/dev/peps/pep-0338/) has more details on exactly how this works.

`python -m` is not limited to executing packages. For times when you want to quickly run some module that has an `if __name__ == '__main__':` block in it, you can use `python -m passacre.module`. Importantly, this will ensure that python recognizes your whole package, which means that your imports won’t fail.

Sometimes a project wants explicit binaries instead of requiring their users to use `python -m`. For this case, setuptools has automatic script creation. Here’s an excerpt from passacre’s `setup.py` file:

```python
from setuptools import setup

setup(
    entry_points={
        'console_scripts': ['passacre = passacre.application:main'],
    },
)
```

This `console_scripts` definition makes a `passacre` executable that does the same thing as the `__main__.py` script did: imports the `main` function from `passacre.application` and calls it.


### Common pitfalls

#### Don’t directly run modules inside packages

What I mean specifically is doing `python passacre/__main__.py`, or `python passacre/test/test_application.py`, or anything else that starts with `python passacre/`. This will prevent python from knowing where the root of your package is, and so absolute imports and explicit relative imports will fail, or not import the code you think it should be importing. Instead, you probably want to use `python -m`, or generate an executable script to run, or use a test runner.

#### Don’t set `PYTHONPATH` to try to make it go

If you think you have to set `PYTHONPATH`, you’ve probably fallen victim to the first pitfall and are trying to execute a module directly. With proper package layout and proper imports, you won’t need to set `PYTHONPATH` to run your code.

#### Don’t modify `sys.path` from code in your package

Like modifying `PYTHONPATH`, but worse because it’s easier to affect other people using your code. Never do this, as it will break and make people trying to use your code very annoyed.

### 参考材料

1. http://stackoverflow.com/questions/419163/what-does-if-name-main-do
1. [Python Packages and You](http://blog.habnab.it/blog/2013/07/21/python-packages-and-you/)