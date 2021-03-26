Python packaging & project workflow
===================================

Packaging Python projects
-------------------------

Tutorial: https://packaging.python.org/tutorials/packaging-projects/

```
packaging_tutorial/
├── LICENSE
├── pyproject.toml
├── README.md
├── setup.cfg
├── setup.py  # optional, needed to make editable pip installs work
├── example_pkg/
│   └── __init__.py
└── tests/
```

### pyproject.toml

```
[build-system]
requires = [
    "setuptools>=42",
    "wheel"
]
build-backend = "setuptools.build_meta"
```

### setup.cfg

```
[metadata]
# replace with your username:
name = example-pkg-YOUR-USERNAME-HERE
version = 0.0.1
author = Example Author
author_email = author@example.com
description = A small example package
long_description = file: README.md
long_description_content_type = text/markdown
url = https://github.com/pypa/sampleproject
project_urls =
    Bug Tracker = https://github.com/pypa/sampleproject/issues
classifiers =
    Programming Language :: Python :: 3
    License :: OSI Approved :: MIT License
    Operating System :: OS Independent

[options]
packages = find:
python_requires = >=3.6
install_requires =
    requests
```

### setup.py

```
import setuptools

setuptools.setup()
```

### LICENSE

Usually I generate this file when creating new project on Github. You may also look here:

- https://choosealicense.com/
- https://opensource.org/licenses


Packaging binary extensions
---------------------------

https://packaging.python.org/guides/packaging-binary-extensions/

Example in my own project: https://github.com/messa/bloom/blob/main/setup.py


Links
-----

https://github.com/mongodb/mongo-python-driver/blob/master/RELEASE.rst

