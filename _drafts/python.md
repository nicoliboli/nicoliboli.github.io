---
layout: post
title: python
categories: dev
tags:
- python
author: nicoliboli
description: python info sheet
---

- `pdb` module can be used for debugging (when whoever thinks they're better than print statements comes along)
- view builtins with `dir(__builtins__)`
- `kwargs` are useful when function can use variable number of parameters
- using `yield` preserves state of variable 

# creating a package/module

1. create a directory with the name of the package
2. put a file called `__init__.py` in the directory. ensures that python sees the directory as a legit package. can be empty if you always import modules by name. needs to have list of modules if doing the classic lazy (`from PACKAGE import *`) 
3. create python file `MODULE_NAME.py` with object and functions

```
$ tree
.
└── python
    ├── testmod
    │   ├── __init__.py
    │   ├── __pycache__
    │   │   ├── __init__.cpython-313.pyc
    │   │   └── test.cpython-313.pyc
    │   └── test.py
    └── x.py

$ cat ./python/testmod/__init__.py 
__all__ = ["test"]

$ cat ./python/testmod/test.py    
def tester(s):
    print("this is a test {}".format(s))


$ cat ./python/x.py           
from testmod import *

s = "friend"
test.tester(s)
```

> you can list imported modules with `sys.module`

# classes

# socket

## scapy