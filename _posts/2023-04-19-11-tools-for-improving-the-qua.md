---
author: John Bencina
header_img: https://media.licdn.com/dms/image/C5612AQGa6LbIE8S75g/article-cover_image-shrink_720_1280/0/1638425243407?e=2147483647&v=beta&t=EJfEJRfB1b_FTGlDI8DUclvEuZp_O7ybS0c_L-Se8_w
source: linkedin
source_file: linkedin_2023-04-19-T16-50-32
source_url: https://www.linkedin.com/pulse/11-tools-improving-quality-python-code-john-bencina/
subtitle: 'Writing high quality code is important. A great codebase is easier to maintain
  and makes your project more likely to be adopted by others. It’s tempting to overlook '
tags: []
title: 11 Tools for Improving the Quality of Python Code

---
![11 Tools for Improving the Quality of Python Code](https://media.licdn.com/dms/image/C5612AQGa6LbIE8S75g/article-cover_image-shrink_720_1280/0/1638425243407?e=2147483647&v=beta&t=EJfEJRfB1b_FTGlDI8DUclvEuZp_O7ybS0c_L-Se8_w)

Writing high quality code is important. A great codebase is easier to maintain and makes your project more likely to be adopted by others. It’s tempting to overlook *boring* details like documentation, testing, and styling when most of our excitement is spent on translating ideas into code. This is especially true within the Python open-source community where such standards are generally self enforced and developer backgrounds are highly varied. 

In this article, I introduce several Python tools any developer / coder / scripter can implement which will greatly improve the quality of Python projects with minimal effort. All of these tools are easy to get started with, but I encourage you to explore and learn more about the great features of each one.

## **1. black**

**TL;DR - Format my code for me.**

Black is an automated code formatter which applies the coding conventions outlined in [PEP 8](https://www.python.org/dev/peps/pep-0008/). Black ensures all of your code follows a similar style by automatically rewriting files where necessary. This helps you write code which is easier for you to maintain and for others to follow. Black is especially great for projects with multiple developers where the enforcement of a single style convention keeps code easy to read.

Learn more - <https://pypi.org/project/black/>

## **2. isort**

**TL;DR - Organize my imports.**

We all import code, but who has time to organize imports in a consistent way across an entire project? isort takes care of this for you by automatically sorting those imports for you across all of your files. Pair it with Black for the ultimate in automated standardization.

Learn more - <https://pypi.org/project/isort/>

## **3. flake8**

**TL;DR - Tell me what style conventions I’m violating.**

Flake8 alerts you to coding style violations. While black and isort actively enforce these conventions, flake8 can raise these alerts as warnings. When coupled with an IDE (*cough* [Visual Studio Code](https://code.visualstudio.com/)), flake8 can surface these warnings within the UI in real-time. We all love standards and this library is a must have for any Python developer.

Learn more - <https://pypi.org/project/flake8/>

## **4. pytest**

**TL;DR - Make sure my code runs correctly.**

You've written a great project, but how can you be sure it works? Maybe you test each change you made manually, but this is tedious and impractical. One function may depend on another requiring lots of checks. Pytest solves this for you by automating unit testing. Simply write a test once and pytest helps you re-run dozens or hundreds of tests with a single command. You can even perform more complex testing like spinning up a local web server to simulate REST API calls.

Learn more - <https://pypi.org/project/pytest/>

## **5. pytest-cov**

**TL;DR -  Am I testing all parts of my code?**

So you’re using pytest, but now you lost track of what code being tested in your project. Pytest-cov helps you by generating a coverage report. Coverage reporting shows the % of your code within each module which is covered by at least one unit test. Testing 100% of code may not be feasible, but you can use this to achieve the best coverage possible.

Learn more - <https://pypi.org/project/pytest-cov/>

## **6. mypy**

**TL;DR - Do inputs/outputs within my code make sense?**

Python is (in)famous for its weakly typed language. While powerful in some scenarios, it can cause issues where inputs to your function are in the wrong data types. Mypy helps you catch this early by checking the inputs and outputs of your code to see whether everything is in the expected type(s). It does this by relying on Python's [type hints](https://docs.python.org/3/library/typing.html) annotations in code and will alert you to any compatibility issues. 

Learn more - <https://pypi.org/project/mypy/>

## **7. pre-commit**

**TL;DR - Automatically run checks & commands.**

Ok these are great tools, but who wants to run several commands each time we change our code? This is where pre-commit saves the day. Pre-commit performs automated execution of these tools, and more. These automated tasks are called hooks and can be configured to run on-demand or with Git actions, such as with each commit. The latter is a great way to ensure you’re only committing code passing all checks. Many pre-made hooks are available and you can write your own.

Learn more - <https://pypi.org/project/pre-commit/>

## **8. sphinx**

**TL;DR - Make documentation great again.**

Most developers don’t like writing documentation. Luckily we have Sphinx to help alleviate some of the burden. Sphinx works with [RST files](https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html) and can compile these into HTML, PDF, and other formats. Sphinx documentation can also be loaded into [readthedocs.org](https://readthedocs.org/) so others can learn about your project. Since the RST files are stored in your project, Sphinx has the added benefit of placing your docs under version control for consistency. 

Additionally, Sphinx can automatically read your docstrings and import those into the project docs. This means you only need to document most of your code once and let Sphinx handle the rest. The [Numpy Style Guide](https://numpydoc.readthedocs.io/en/latest/format.html) is a great resource for a common format that works well with Sphinx. Docstings can also contain examples which can be used with [doctest](https://docs.python.org/3/library/doctest.html) to ensure the examples perform as expected. 

Learn more - <https://pypi.org/project/Sphinx/>

## **9. pydantic**

**TL;DR - Easy to use data models.**

While Python gives you the flexibility to work in a non-object oriented way (eg. dicts, tuples), leveraging strongly typed data models (classes) gives your code more robustness. Defining a Python class is fairly straight forward, but pydantic makes it even easier. It has data model specific features like type/value enforcement and helper functions for exporting data to formats like JSON without having to write encoders.

Learn more - <https://pypi.org/project/pydantic/>

## **10. click**

**TL;DR - Quickly build command line interaces.**

We generally write code so others can consume it. In many cases, this means invoking the code though the command line. Click is an alternative to Python’s native [argparse](https://docs.python.org/3/library/argparse.html) library and streamlines the generation of CLI menus. Commands are written as functions and clicks’s decorators allow for easy nesting and more advanced functionality.

Learn more - <https://pypi.org/project/click/>

## **11. Poetry**

**TL;DR - Manage packages and virtual environments.**

The last tool on the list is not a library, but rather an environment manager and alternative to the setup tools approach to package distribution. Even if you’re not planning to make your code available on [pypi](https://pypi.org/), Poetry helps manage virtual environments for you and other developers of your project. I encourage you to check it out!

Learn more - <https://python-poetry.org/>

I hope this article helped inspire you on improvements within your own coding projects. If so, give it a quick like 👍! Also, I would love to hear from others what tooling they use. So drop a comment with anything I may have missed!

