---
author: Jan Giacomelli
header_img: https://testdriven.io/static/images/blog/python_code_quality_social_card.png
source: testdriven
source_file: testdriven_2023-04-19-T16-51-01
source_url: https://testdriven.io/blog/python-code-quality/
subtitle: What exactly is code quality? How do we measure it? How do we improve code
  quality and clean up our Python code?
tags:
- python
- testing
title: Python Code Quality

---
![Python Code Quality](https://testdriven.io/static/images/blog/python_code_quality_social_card.png)


*What exactly is code quality? How do we measure it? How do we improve code quality and clean up our Python code?*


  



[Code quality](https://en.wikipedia.org/wiki/Software_quality) generally refers to how functional and maintainable your code is. Code is considered to be of high quality when:


1. It serves its purpose
2. It's behavior can be tested
3. It follows a consistent style
4. It's understandable
5. It doesn't contain security vulnerabilities
6. It's documented well
7. It's easy to maintain


Since we already addressed the first two points in the [Testing in Python](/blog/testing-python/) and [Modern Test-Driven Development in Python](/blog/modern-tdd/) articles, the focus of this article is on points three through seven.


This article looks how to improve the quality of your Python code with linters, code formatters, and security vulnerability scanners.


  




> 
> The [Complete Python](/guides/complete-python/) Guide:
> 
> 
> 1. [Modern Python Environments - dependency and workspace management](/blog/python-environments/)
> 2. [Testing in Python](/blog/testing-python/)
> 3. [Modern Test-Driven Development in Python](/blog/modern-tdd/)
> 4. [Python Code Quality](/blog/python-code-quality/) (this article!)
> 5. [Python Type Checking](/blog/python-type-checking/)
> 6. [Documenting Python Code and Projects](/blog/documenting-python/)
> 7. [Python Project Workflow](/blog/python-project-workflow/)
> 
> 
> 


## Contents



* [Linters](#linters)
	+ [Flake8](#flake8)
* [Code Formatters](#code-formatters)
	+ [isort](#isort)
	+ [Black](#black)
* [Security Vulnerability Scanners](#security-vulnerability-scanners)
	+ [Bandit](#bandit)
	+ [Safety](#safety)
* [Running Code Quality Tools](#running-code-quality-tools)
	+ [Inside Your IDE or Code Editor](#inside-your-ide-or-code-editor)
	+ [Pre-commit Hooks](#pre-commit-hooks)
	+ [CI Pipeline](#ci-pipeline)
* [Real Project](#real-project)
* [Conclusion](#conclusion)



## Linters


[Linters](https://en.wikipedia.org/wiki/Lint_(software)) flag programming errors, bugs, stylistic errors, and suspicious constructs through source code analysis. Linting tools are easy to set up, provide sane defaults, and improve the overall developer experience by removing friction between developers who have differing opinions on style.



> 
> While linting is a common practice, it's still frowned upon by many developers since developers tend to be, well, hyper opinionated.
> 
> 
> 


Let's look at a quick example.


Version one:



```
numbers = []

while True:
    answer = input('Enter a number: ')
    if answer != 'quit':
        numbers.append(answer)
    else:
        break

print('Numbers: %s' % numbers)

```

Version two:



```
numbers = []

while (answer := input("Enter a number: ")) != "quit":
    numbers.append(answer)

print(f"Numbers: {numbers}")

```

Version three:



```
numbers = []

while True:
    answer = input("Enter a number: ")
    if answer == "quit":
        break
    numbers.append(answer)

print(f"Numbers: {numbers}")

```

Which one is better?


In terms of functionality they are the same.


Which one do you prefer? Which one is preferred by your project's collaborators?


As a software developer you're very likely working in a team. And, in a team setting, it's very important that all developers follow the same coding standards. Otherwise, it's much harder to read someone else's code. The focus of code reviews should be on higher level issues rather than mundane syntactical formatting issues.


For example, if you decided to end every sentence with an exclamation mark, it would be very difficult for a reader to infer the tone. If you took it a step further and ignored common standards like capitalization and spacing rules, your sentences would be very difficult to read. It would take a lot more brain power to read your writing. You'd lose readers and collaborators. It's similar with code. We use style guides to make it easier for our fellow developers (ourselves included) to infer intent and collaborate with us.


We're fortunate as Python developers to have the [PEP-8](https://pep8.org/) style guide at our disposal, which provides a set of conventions, guidelines, and best practices for making our code easier to read and maintain. It focuses on naming conventions, code comments, and layout issues (like indentation and whitespace). A few examples:


* 79 character [max line length](https://pep8.org/#maximum-line-length)
* [spaces](https://pep8.org/#tabs-or-spaces) instead of tabs
* [lower case](https://pep8.org/#function-names) function names


In terms of linting tools, while there are a number of them out there, for the most part each look for errors in either code logic or enforce code standards:


1. **code logic** - these check for programming errors, enforce code standards, search for code smells, and check code complexity. [Pyflakes](https://github.com/PyCQA/pyflakes) and [McCabe](https://github.com/PyCQA/mccabe) (complexity checker) are the most popular tools for linting code logic.
2. **code style** - these just enforce code standards (based on PEP-8). [pycodestyle](https://github.com/pycqa/pycodestyle) falls into this category.


### Flake8


[Flake8](https://flake8.pycqa.org/) is a wrapper around Pyflakes, pycodestyle, and McCabe.


It can be installed like any other PyPI package:



```
$ pip install flake8

```

Say you have the following code saved to a file called *my_module.py*:



```
from requests import *

def get_error_message(error_type):
    if error_type == 404:
        return 'red'
    elif error_type == 403:
        return 'orange'
    elif error_type == 401:
        return 'yellow'
    else:
        return 'blue'


def main():
    res = get('https://api.github.com/events')
    STATUS = res.status_code
    if res.ok:
        print(f'{STATUS}')
    else:
        print(get_error_message(STATUS))




if __name__ == '__main__':
    main()

```

To lint this file, you can simply run:



```
$ python -m flake8 my_module.py

```

This should produce the following output:



```
my_module.py:1:1: F403 'from requests import *' used; unable to detect undefined names
my_module.py:3:1: E302 expected 2 blank lines, found 1
my_module.py:15:11: F405 'get' may be undefined, or defined from star imports: requests
my_module.py:25:1: E303 too many blank lines (4)

```


> 
> You may also see a `my_module.py:26:11: W292 no newline at end of file` error depending on your code editor's configuration.
> 
> 
> 


For every violation a line is printed that contains the following data:


1. file path (relative to the directory where Flake8 ran from)
2. line number
3. column number
4. ID of violated [rule](https://www.flake8rules.com/)
5. description of rule


The violations that start with `F` are errors from [Pyflakes](https://flake8.pycqa.org/en/latest/user/error-codes.html) while violations that start with `E` are from [pycodestyle](https://pycodestyle.pycqa.org/en/latest/intro.html#error-codes).


After correcting the violations, you should have:



```
from requests import get


def get_error_message(error_type):
    if error_type == 404:
        return 'red'
    elif error_type == 403:
        return 'orange'
    elif error_type == 401:
        return 'yellow'
    else:
        return 'blue'


def main():
    res = get('https://api.github.com/events')
    STATUS = res.status_code
    if res.ok:
        print(f'{STATUS}')
    else:
        print(get_error_message(STATUS))


if __name__ == '__main__':
    main()

```

Along with PyFlakes and pycodestyle, you can use Flake8 to check for [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity) as well.


For example, the `get_error_message` function has a complexity of four, since there are four possible branches (or code paths):



```
def get_error_message(error_type):
    if error_type == 404:
        return 'red'
    elif error_type == 403:
        return 'orange'
    elif error_type == 401:
        return 'yellow'
    else:
        return 'blue'

```

To enforce a max complexity of 3 or lower, run:



```
$ python -m flake8 --max-complexity 3 my_module.py

```

Flake8 should fail with:



```
my_module.py:4:1: C901 'get_error_message' is too complex (4)

```

Refactor the code like so:



```
def get_error_message(error_type):
    colors = {
        404: 'red',
        403: 'orange',
        401: 'yellow',
    }
    return colors[error_type] if error_type in colors else 'blue'

```

Flake8 should now pass:



```
$ python -m flake8 --max-complexity 3 my_module.py

```

You can add additional checks to Flake8 via its powerful plugin system. For example, to enforce PEP-8 naming conventions, install [pep8-naming](https://github.com/PyCQA/pep8-naming):



```
$ pip install pep8-naming

```

Run:



```
$ python -m flake8 my_module.py

```

You should see:



```
my_module.py:15:6: N806 variable 'STATUS' in function should be lowercase

```

Fix:



```
def main():
    res = get('https://api.github.com/events')
    status = res.status_code
    if res.ok:
        print(f'{status}')
    else:
        print(get_error_message(status))

```

Check out [Awesome Flake8 Extensions](https://github.com/DmytroLitvinov/awesome-flake8-extensions) for a list of the most popular extensions.



> 
> [Pylama](https://github.com/klen/pylama) is a popular linting tool as well, which, like Flake8, glues together several linters.
> 
> 
> 


## Code Formatters


While linters just check for issues in your code, code formatters actually reformat your code based on a set of standards.


Keeping your code in a proper format is a necessary yet dull job that should be performed by a computer.


Why is it necessary?


Well-formatted code that follows a style guide for consistency is easier to read, which makes it easier to find bugs and onboard new developers. It also reduces merge conflicts.



> 
> Readability counts.
> 
> 
> – The [Zen of Python](https://en.wikipedia.org/wiki/Zen_of_Python)
> 
> 
> 


Again, since this is a dull job that developers are often opinionated about (tabs vs spaces, single vs double quotes, etc.), use a code formatting tool to automatically reformat your code in place based on a set of standards.


### isort


[isort](https://pycqa.github.io/isort/) is used to automatically separate imports in your code into the following groups:


1. standard library
2. third-party
3. local


The imports in groups are then individually alphabetized.



```
# standard library
import datetime
import os

# third-party
import requests
from flask import Flask
from flask.cli import AppGroup

# local
from your_module import some_method

```

Install:



```
$ pip install isort

```


> 
> If you'd prefer a Flake8 plugin, check out [flake8-isort](https://github.com/gforcada/flake8-isort). [flake8-import-order](https://github.com/PyCQA/flake8-import-order) is quite popular as well.
> 
> 
> 


To run it against the files in the current directory and sub directories:



```
$ python -m isort .

```

To run it against a single file:



```
$ python -m isort my_module.py

```

Before:



```
import os
import datetime
from your_module import some_method
from flask.cli import AppGroup
import requests
from flask import Flask

```

After:



```
import datetime
import os

import requests
from flask import Flask
from flask.cli import AppGroup

from your_module import some_method

```

To check if your imports are correctly sorted and ordered without making changes, use the `--check-only` flag:



```
$ python -m isort my_module.py --check-only

ERROR: my_module.py Imports are incorrectly sorted and/or formatted.

```

To see the changes, without applying them use the `--diff` flag:



```
$ python -m isort my_module.py --diff

--- my_module.py:before      2022-02-28 22:04:45.977272
+++ my_module.py:after       2022-02-28 22:04:48.254686
@@ -1,6 +1,7 @@
+import datetime
 import os
-import datetime
+
+import requests
+from flask import Flask
+from flask.cli import AppGroup
 from your_module import some_method
-from flask.cli import AppGroup
-import requests
-from flask import Flask

```

You should use the `--profile black` [option](https://pycqa.github.io/isort/docs/configuration/black_compatibility.html) when using isort with Black to avoid code style collisions:



```
$ python -m isort --profile black .

```

### Black


[Black](https://github.com/psf/black) is a Python code formatter that's used to reformat your code based on the Black's [code style guide](https://black.readthedocs.io/en/stable/the_black_code_style/current_style.html), which is pretty close to PEP-8.



```
$ pip install black

```


> 
> Prefer a Flake8 plugin? Check out [flake8-black](https://github.com/peterjc/flake8-black).
> 
> 
> 


To edit your files recursively inside the current directory:



```
$ python -m black .

```

It can also be ran against a single file:



```
$ python -m black my_module.py

```

Before:



```
import pytest

@pytest.fixture(scope="module")
def authenticated_client(app):
    client = app.test_client()
    client.post("/login", data=dict(email="[[email protected]](/cdn-cgi/l/email-protection)", password="notreal"), follow_redirects=True)
    return client

```

After:



```
import pytest


@pytest.fixture(scope="module")
def authenticated_client(app):
    client = app.test_client()
    client.post(
        "/login",
        data=dict(email="[[email protected]](/cdn-cgi/l/email-protection)", password="notreal"),
        follow_redirects=True,
    )
    return client

```

If you just want to check if your code follows the Black code style standards, you can use the `--check` flag:



```
$ python -m black my_module.py --check

would reformat my_module.py
Oh no! 💥 💔 💥
1 file would be reformatted.

```

The `--diff` flag, meanwhile, shows the diff between your current code and the reformatted code:



```
$ python -m black my_module.py --diff

--- my_module.py        2022-02-28 22:04:45.977272 +0000
+++ my_module.py        2022-02-28 22:05:15.124565 +0000
@@ -1,7 +1,12 @@
 import pytest
+

 @pytest.fixture(scope="module")
 def authenticated_client(app):
     client = app.test_client()
-    client.post("/login", data=dict(email="[[email protected]](/cdn-cgi/l/email-protection)", password="notreal"), follow_redirects=True)
-    return client
\ No newline at end of file
+    client.post(
+        "/login",
+        data=dict(email="[[email protected]](/cdn-cgi/l/email-protection)", password="notreal"),
+        follow_redirects=True,
+    )
+    return client
would reformat my_module.py

All done! ✨ 🍰 ✨
1 file would be reformatted.

```


> 
> [YAPF](https://github.com/google/yapf) and [autopep8](https://github.com/hhatto/autopep8) are code formatters similar to Black that are worth looking at as well.
> 
> 
> 


## Security Vulnerability Scanners


Security vulnerabilities are arguably the most important aspect of code quality, and yet they are often ignored. Your code is only as secure as its weakest link. Thankfully, there are a number of tools that can help detect possible vulnerabilities in our code. Let's take a look at two of them.


### Bandit


[Bandit](https://github.com/PyCQA/bandit) is a tool designed to find common security [issues](https://bandit.readthedocs.io/en/latest/plugins/index.html#complete-test-plugin-listing) in Python code such as hardcoded password strings, deserializing untrusted code, using `pass` in except blocks, to name a few.



```
$ pip install bandit

```


> 
> Prefer a Flake8 plugin? Check out [flake8-bandit](https://github.com/tylerwince/flake8-bandit).
> 
> 
> 


Run it like so:



```
$ bandit my_module.py

```

Code:



```
evaluate = 'print("Hi!")'
eval(evaluate)


evaluate = 'open("secret_file.txt").read()'
eval(evaluate)

```

You should see the following warning:



```
>> Issue: [B307:blacklist] Use of possibly insecure function - consider using safer
    ast.literal_eval.
   Severity: Medium   Confidence: High
   Location: my_module.py:2
   More Info:
    https://bandit.readthedocs.io/en/latest/blacklists/blacklist_calls.html#b307-eval
1   evaluate = 'print("Hi!")'
2   eval(evaluate)
3

--------------------------------------------------
>> Issue: [B307:blacklist] Use of possibly insecure function - consider using safer
    ast.literal_eval.
   Severity: Medium   Confidence: High
   Location: my_module.py:6
   More Info:
    https://bandit.readthedocs.io/en/latest/blacklists/blacklist_calls.html#b307-eval
5   evaluate = 'open("secret_file.txt").read()'
6   eval(evaluate)

--------------------------------------------------

```

### Safety


[Safety](https://github.com/pyupio/safety) is another tool that comes in handy for keeping your code free of security issues.


It's used to check your installed dependencies for known security vulnerabilities against [Safety DB](https://github.com/pyupio/safety-db), which is a database of known security vulnerabilities in Python packages.



```
$ pip install safety

```

With your virtual environment activated, you run it like so:



```
$ safety check

```

Sample output when Flask v0.12.2 is installed:



```
+==============================================================================+
|                                                                              |
|                               /$$$$$$            /$$                         |
|                              /$$__  $$          | $$                         |
|           /$$$$$$$  /$$$$$$ | $$  \__//$$$$$$  /$$$$$$   /$$   /$$           |
|          /$$_____/ |____  $$| $$$$   /$$__  $$|_  $$_/  | $$  | $$           |
|         |  $$$$$$   /$$$$$$$| $$_/  | $$$$$$$$  | $$    | $$  | $$           |
|          \____  $$ /$$__  $$| $$    | $$_____/  | $$ /$$| $$  | $$           |
|          /$$$$$$$/|  $$$$$$$| $$    |  $$$$$$$  |  $$$$/|  $$$$$$$           |
|         |_______/  \_______/|__/     \_______/   \___/   \____  $$           |
|                                                          /$$  | $$           |
|                                                         |  $$$$$$/           |
|  by pyup.io                                              \______/            |
|                                                                              |
+==============================================================================+
| REPORT                                                                       |
| checked 37 packages, using default DB                                        |
+============================+===========+==========================+==========+
| package                    | installed | affected                 | ID       |
+============================+===========+==========================+==========+
| flask                      | 0.12.2    | <0.12.3                  | 36388    |
| flask                      | 0.12.2    | <1.0                     | 38654    |
+==============================================================================+

```

## Running Code Quality Tools


Now that you know the tools, the next question is: When should they be used?


Typically, the tools are run:


1. While coding (inside your IDE or code editor)
2. At commit time (with pre-commit hooks)
3. When code is checked in to source control (via a CI pipeline)


### Inside Your IDE or Code Editor


It's best to check for issues that could have a negative impact on quality early and often. Therefore, it's strongly recommended to lint and format your code during development. Many of the popular IDEs have linters and formatters built-in. You'll be able to find a plugin for your code editor for most of the aforementioned tools. Such plugins warn you in real-time about code style violations and potential programming errors.


Resources:


1. [Linting](https://code.visualstudio.com/docs/python/linting) and [Formatting](https://code.visualstudio.com/docs/python/editing#_formatting) Python in Visual Studio Code
2. [Black Editor Integrations](https://black.readthedocs.io/en/stable/integrations/editors.html)
3. [Sublime Text Package Finder](https://packagecontrol.io/)
4. [Atom Packages](https://atom.io/packages)


### Pre-commit Hooks


Since you'll inevitably miss a warning here and there as you're coding, it's a good practice to check for quality issues at commit time with pre-commit git hooks. You can first format your code before you lint it. This way you can avoid committing code that won't pass code quality checks inside your CI pipeline.


The [pre-commit](https://pre-commit.com/) framework is recommended for managing git hooks.



```
$ pip install pre-commit

```

Once installed, add a pre-commit config file called *.pre-commit-config.yaml* to your project. To run Flake8, add the following config:



```
repos:
- repo: https://gitlab.com/PyCQA/flake8
 rev: 4.0.1
 hooks:
 - id: flake8

```

Finally, to set up the git hook scripts, run:



```
(venv)$ pre-commit install

```

Now, every time you run `git commit` Flake8 will run before the actual commit is made. And if there are any issues, the commit will be aborted.


### CI Pipeline


Although you may be using code quality tools inside your code editor and pre-commit hook, you can't always count on your teammates and other collaborators to do the same. So, you should run code quality checks inside your CI pipeline. At this point, you should run linters and security vulnerabilities detectors and ensure that the code follows a particular code style. You can run such checks in parallel with your tests.


## Real Project


Let's create a simple project to see how all of this works.


First, create a new folder:



```
$ mkdir flask_example
$ cd flask_example

```

Next, initialize your project with [Poetry](https://python-poetry.org):



```
$ poetry init

Package name [flask_example]:
Version [0.1.0]:
Description []:
Author [Your name <[[email protected]](/cdn-cgi/l/email-protection)>, n to skip]:
License []:
Compatible Python versions [^3.10]:

Would you like to define your main dependencies interactively? (yes/no) [yes] no
Would you like to define your development dependencies interactively? (yes/no) [yes] no
Do you confirm generation? (yes/no) [yes]

```

After that, add Flask, pytest, Flake8, Black, isort, Bandit, and Safety:



```
$ poetry add flask
$ poetry add --dev pytest flake8 black isort safety bandit

```

Create a file to hold tests called *test_app.py*:



```
from app import app
import pytest


@pytest.fixture
def client():
    app.config['TESTING'] = True

    with app.test_client() as client:
        yield client


def test_home(client):
    response = client.get('/')

    assert response.status_code == 200

```

Next, add a file for the Flask app called *app.py*:



```
from flask import Flask

app = Flask(__name__)


@app.route('/')
def home():
    return 'OK'


if __name__ == '__main__':
    app.run()

```

Now, we're ready to add the pre-commit configuration.


First, initialize a new git repository:



```
$ git init

```

Next, install pre-commit and set up the git hook scripts:



```
$ poetry add --dev pre-commit
$ poetry run pre-commit install

```

Create a file for the config called *.pre-commit-config.yaml*:



```
repos:
- repo: https://gitlab.com/PyCQA/flake8
 rev: 4.0.1
 hooks:
 - id: flake8

```

Before committing, run isort and Black:



```
$ poetry run isort . --profile black
$ poetry run black .

```

Commit your changes to trigger the pre-commit hook:



```
$ git add .
$ git commit -m 'Initial commit'

```

Finally, let's configure a CI pipeline via [GitHub Actions](https://github.com/features/actions).


Create the following file and folders:



```
.github
└── workflows
    └── main.yaml

```

*.github/workflows/main.yaml*:



```
name: CI
on: [push]

jobs:
 test:
 strategy:
 fail-fast: false
 matrix:
 python-version: [3.10.2]
 poetry-version: [1.1.13]
 os: [ubuntu-latest]
 runs-on: ${{ matrix.os }}
 steps:
 - uses: actions/[[email protected]](/cdn-cgi/l/email-protection)
 - uses: actions/[[email protected]](/cdn-cgi/l/email-protection)
 with:
 python-version: ${{ matrix.python-version }}
 - name: Run image
 uses: abatilo/[[email protected]](/cdn-cgi/l/email-protection)
 with:
 poetry-version: ${{ matrix.poetry-version }}
 - name: Install dependencies
 run: poetry install
 - name: Run tests
 run: poetry run pytest
 code-quality:
 strategy:
 fail-fast: false
 matrix:
 python-version: [3.10.2]
 poetry-version: [1.1.13]
 os: [ubuntu-latest]
 runs-on: ${{ matrix.os }}
 steps:
 - uses: actions/[[email protected]](/cdn-cgi/l/email-protection)
 - uses: actions/[[email protected]](/cdn-cgi/l/email-protection)
 with:
 python-version: ${{ matrix.python-version }}
 - name: Run image
 uses: abatilo/[[email protected]](/cdn-cgi/l/email-protection)
 with:
 poetry-version: ${{ matrix.poetry-version }}
 - name: Install dependencies
 run: poetry install
 - name: Run black
 run: poetry run black . --check
 - name: Run isort
 run: poetry run isort . --check-only --profile black
 - name: Run flake8
 run: poetry run flake8 .
 - name: Run bandit
 run: poetry run bandit .
 - name: Run saftey
 run: poetry run safety check

```

This configuration:


* runs on every push - `on: [push]`
* runs on the latest version of Ubuntu - `ubuntu-latest`
* uses Python 3.10.2 - `python-version: [3.10.2]`, `python-version: ${{ matrix.python-version }}`
* uses Poetry version 1.1.13 - `poetry-version: [1.1.13]`, `poetry-version: ${{ matrix.poetry-version }}`


There are two jobs defined: `test` and `code-quality`. As the names' suggest, the tests run in the `test` job while our code quality checks run in the `code-quality` job.


Add CI config to git and commit:



```
$ git add .github/workflows/main.yaml
$ git commit -m 'Add CI config'

```

Create a new repository on [GitHub](https://github.com/new) and push your project to the newly created remote.


For example:



```
$ git remote add origin [[email protected]](/cdn-cgi/l/email-protection):jangia/flask_example.git
$ git branch -M main
$ git push -u origin main

```

You should see your workflow running on the *Actions* tab on your GitHub repository.


## Conclusion


Code quality is one of the most opinionated topics in software development. Code style, in particular, is a sensitive issue amongst developers since we spend much of our development time reading code. It's much easier to read and infer intent when code has a consistent style that adheres to PEP-8 standards. Since this is a dull, mundane process, it should be handled by a computer via code formatters like Black and isort. Similarly, Flake8, Bandit, and Safety help ensure your code is safe and free of errors.




---



> 
> The [Complete Python](/guides/complete-python/) Guide:
> 
> 
> 1. [Modern Python Environments - dependency and workspace management](/blog/python-environments/)
> 2. [Testing in Python](/blog/testing-python/)
> 3. [Modern Test-Driven Development in Python](/blog/modern-tdd/)
> 4. [Python Code Quality](/blog/python-code-quality/) (this article!)
> 5. [Python Type Checking](/blog/python-type-checking/)
> 6. [Documenting Python Code and Projects](/blog/documenting-python/)
> 7. [Python Project Workflow](/blog/python-project-workflow/)
> 
> 
> 


