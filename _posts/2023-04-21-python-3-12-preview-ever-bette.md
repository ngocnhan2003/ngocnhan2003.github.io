---
author: Martin Breuss
header_img: _posts/img/2023-04-21-python-3-12-preview-ever-bette.png
source: realpython
source_file: realpython_2023-04-21-T10-48-49
source_url: https://realpython.com/python312-error-messages/
subtitle: 'Python 3.12 will be released in '
tags:
- intermediate
- python
title: 'Python 3.12 Preview: Ever Better Error Messages'

---
![Python 3.12 Preview: Ever Better Error Messages]({{site.cdn_img_raw}}/_posts/img/2023-04-21-python-3-12-preview-ever-bette.png)



Table of Contents



* [Better Error Messages in Python 3.12](#better-error-messages-in-python-312)
* [Unimported Standard Library Modules](#unimported-standard-library-modules)
* [Missing self. in Instance Attributes](#missing-self-in-instance-attributes)
* [Wrong Order for Import Statements](#wrong-order-for-import-statements)
* [Misspelled Names in Imported Namespaces](#misspelled-names-in-imported-namespaces)
* [Conclusion](#conclusion)










Python 3.12 will be released in [October 2023](https://peps.python.org/pep-0693/). Even though October is still months away, you can already preview some of the upcoming features, including how Python 3.12 will offer even more readable and actionable **error messages**.


**In this tutorial, you’ll:**


* Experience the **improved communication** in various error situations
* Learn about the **background and limitations** of these changes
* Peek into the **CPython source code** of the pull requests that made these error messages sparkle


There are many other improvements and new features coming in Python 3.12. Keep track of [what’s new](https://docs.python.org/3.12/whatsnew/3.12.html) in the changelog for an up-to-date list. The list will be complete when Python 3.12 development reaches feature freeze in May 2023.



**Free Bonus:** [Click here to download sample code](https://realpython.com/bonus/python-312-code/) that’ll give you a sneak peek at Python 3.12, coming in October 2023.



## Better Error Messages in Python 3.12


Even Python can get more user-friendly! When Python 3.9 introduced a new [parsing expression grammar (PEG)](https://en.wikipedia.org/wiki/Parsing_expression_grammar) parser for the language, it opened up the door for [better error messages in Python 3.10](https://realpython.com/python310-new-features/#better-error-messages). Python 3.11 followed up with [even better error messages](https://realpython.com/python311-error-messages/), and that same quest continues in Python 3.12.


The error message improvements have primarily focused on more precisely pinpointing where an error occurred and then more clearly communicating that information to you. Additionally, some error messages give suggestions for misspelled names or attributes. Python communicates these suggestions to you with a friendly sentence that starts with *Did you mean …?*.


With these changes and more, Python 3.12 expands on the tradition of offering helpful suggestions. Specifically, it brings several improvements to import-related errors:





| Topic | Error | Old message | Update |  |
| --- | --- | --- | --- | --- |
| [Unimported Standard Library Modules](#unimported-standard-library-modules) | NameError | name ‘sys’ is not defined. | Did you forget to import ‘sys’? |  |
| [Missing `self.` in Instance Attributes](#missing-self-in-instance-attributes) | NameError | name ‘message’ is not defined. | Did you mean: ‘self.message’? |  |
| [Wrong Order for `from`-`import` Statements](#wrong-order-for-import-statements) | SyntaxError | invalid syntax | Did you mean to use ‘from … import …’ instead? |  |
| [Misspelled Names in Imported Namespaces](#misspelled-names-in-imported-namespaces) | ImportError | cannot import name ‘path’ from ‘pathlib’. | Did you mean: ‘Path’? |  |



You’ll walk through an example for each of these improvements in the following sections of this tutorial. You’ll also learn about the context and limitations of the improvements, and take a look at the GitHub commits and the CPython source code adaptations that made them possible.


While Python 3.12 isn’t officially released yet, you can get your hands on a pre-release version and work alongside the tutorial to see the improved error messages in action. If you haven’t already installed Python 3.12, then you can follow the guide on [how to install a pre-release version of Python](https://realpython.com/python-pre-release/).


When you’re done, confirm that you’re now using a Python 3.12 pre-release version and open up a [Python REPL](https://realpython.com/python-repl/):



```
$ python3.12 --version
Python 3.12.0a6
$ python3.12

```

At the time of writing, the newest pre-release version was alpha 6. You should most likely see the same behavior if you’re on a slightly different version.


With the setup out of the way, it’s time to make some mistakes and learn how the upcoming release of Python will tell you about them through improved error messages.



Even them most seasoned programmers make mistakes. For example, maybe you’ve attempted to use a module from the standard library without first importing it. In that case, the plain `NameError` that Python used to raise was probably not too helpful in figuring out that you only forgot the import.


When Python 3.12 raises a `NameError` that’s caused by attempting to use a standard library module that hasn’t been imported yet, then it’ll suggest that you import that module. This suggestion only works for standard library modules, not for your own modules or third-party libraries.


Before exploring the improvement, take a look what such an error message looks like in Python 3.11 and before.


You want to know the names of all the standard library modules, and you know that you can get these using [`sys.stdlib_module_names`](https://docs.python.org/3/library/sys.html#sys.stdlib_module_names). If you forget to import `sys`, then Python will raise an exception with a short error message:


```
>>> sys.stdlib_module_names
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'sys' is not defined

```

The message is already fairly helpful. It lets you know that you’re trying to work with something called `sys` but that it hasn’t been defined. Therefore, Python doesn’t know what to do with your code.


You’ll get the same message if you attempt to access any variable name that isn’t accessible in the current scope. For example, say you call the name `three_twelve`, which you haven’t previously defined:


```
>>> three_twelve
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'three_twelve' is not defined

```

Because the name `three_twelve` isn’t defined, Python raises a `NameError` and tells you that the name isn’t defined. That makes sense in a case where you really didn’t define anything like that. Python can’t work with names that don’t exist!


But maybe you just misspelled a variable name—human that you are! It’d be nice if your digital friend Python could give you some pointers on what to fix.


In Python 3.10, the quest for [better error messages](https://realpython.com/python310-new-features/#better-error-messages) began, bringing improvements to `NameError`, among others. Starting from that release, you now get a hint on how you might be able to fix the issue:


```
>>> pint
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'pint' is not defined. Did you mean: 'print'?

```

Starting from Python 3.10, `NameError` can suggest variable names of built-ins as well as names that you defined yourself.


But standard library modules weren’t part of these suggestions. Therefore, [Pamela Fox](https://discuss.python.org/u/pamelafox) started a discussion thread on the Python Discourse:



> 
> First, I think the *did you mean* feature is really cool and will be very helpful, as I’ve spent a good amount of time helping students spot spelling errors in their code. […] I suggest that it can consider standard library module names for suggestions. ([Source](https://discuss.python.org/t/suggestions-could-consider-standard-library-module-names-as-well/19348))
> 
> 
> 


Promptly, [Pablo Galindo Salgado](https://github.com/pablogsal), who was the main driver behind the earlier error message improvements, took up the suggestion and added it to the Python 3.12 release plan.


If you reproduce the first code example from this section with Python 3.12, then Python will give you an updated, even more helpful tip on what might’ve gone wrong:


```
>>> sys.stdlib_module_names
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'sys' is not defined. Did you forget to import 'sys'?

```

Indeed, in this case, you attempted to access a variable in the built-in `sys` module, but you forgot to import it first. With Python’s helpful pointer, you can quickly fix the issue:


```
>>> import sys
>>> sys.stdlib_module_names
frozenset({'gc', '_operator', 'pipes', ..., 'socket', 'this', 'urllib'})

```

After importing `sys`, you can quickly see the names of all the available standard library modules.



**Note:** Notice that the standard library modules show up in a [frozen set](https://realpython.com/python-sets/#frozen-sets). Because frozen sets are unordered data structures, the output that you see in your REPL session may look different. But don’t worry—everything’s in there!



Incidentally, this frozen set is also what Python 3.12 compares your code to when there’s a `NameError`. That’s how it figures out whether you might’ve forgotten to import a standard library module.


In the code block below, you’ll see the relevant code that Pablo added to the [CPython source code](https://realpython.com/cpython-source-code-guide/), displayed with corresponding line numbers so that you can find the added code [in the source code file](https://github.com/pablogsal/cpython/blob/bb56dead336357153a0c3b8cc9d9d6856d2c5a03/Lib/traceback.py#L715-L721):



```
711# cpython/Lib/traceback.py
712
713# ...
714
715if issubclass(exc_type, NameError):
716    wrong_name = getattr(exc_value, "name", None)
717 if wrong_name is not None and wrong_name in sys.stdlib_module_names:
718        if suggestion:
719            self._str += f" Or did you forget to import '{wrong_name}'"
720        else:
721            self._str += f". Did you forget to import '{wrong_name}'"
722
723# ...

```

Pablo added the code shown above to check for suggestions when Python raises a `NameError`. When you try to use a name that hasn’t been defined, Python saves it as `.name` on the exception object. Next, it assigns the name to `wrong_name` and checks whether it’s in `sys.stdlib_module_names`.


It then considers whether there’s already a `suggestion` for `wrong_name`.



**Note:** The `suggestion` got computed [earlier in the code](https://github.com/pablogsal/cpython/blob/bb56dead336357153a0c3b8cc9d9d6856d2c5a03/Lib/traceback.py#L712) and is part of [suggesting fixes](https://github.com/pablogsal/cpython/blob/bb56dead336357153a0c3b8cc9d9d6856d2c5a03/Lib/traceback.py#L713-L714) for misspelled names. This is the *Did you mean* feature that came about in earlier iterations of improving Python’s error messages.



You can then get one of two slightly different tips about importing a module, depending on whether or not a `suggestion` exists:


1. A suggestion, followed by `Or did you forget to import '{wrong_name}'`
2. A termination of the previous sentence followed by `Did you forget to import '{wrong_name}'`


The code snippet above shows the main part of the Python code that makes up the new error message feature of suggesting unimported standard library modules. The overall implementation is more complicated than the few lines of Python code that you’ve seen. If you’re curious and know a little about the [CPython internals](https://realpython.com/products/cpython-internals-book/), then you can check out all the changes for the merged [gh-98255 - Include stdlib module names in error messages for NameErrors](https://github.com/python/cpython/pull/98255/files).


The discussion about this implementation also brought up the idea of extending these import suggestions to third-party libraries. Pablo mentioned in a reply that this require several steps:


* Importing `pkgutil` or something similar
* Importing some of the C-API to iterate over `pkgutil.iter_modules()` and extract the names
* Performing input/output on the exception handler
* Setting up some bounds for the potentially unbound list of possible packages


Tackling all of these requirements increases the complexity of the implementation. With that said, he doesn’t rule it out as a possible future improvement:



> 
> I don’t mean this to suggest we won’t be adding it, just to tame expectations around it. Maintaining CPython can be very challenging and we have limited resources so we need to control the complexity. **Especially when involving critical C code**, even if the feature sounds very useful. ([Source](https://discuss.python.org/t/suggestions-could-consider-standard-library-module-names-as-well/19348/8))
> 
> 
> 


So suggestions for unimported external packages won’t be part of the upcoming Python release, and they might not be for a while. However, there’s a possibility that something like that might come at some point in the future.


In the meantime, Python 3.12 will now suggest that you import an unimported standard library module when your code raises a `NameError` based on a name that’s in `sys.stdlib_module_names`.



Have you ever forgotten to prepend `self.` when you’re building a Python class and defining instance attributes? The `NameError` that Python used to raise in that case might not have been a great help in identifying the mistake.


When Python 3.12 raises a `NameError` in an instance method, and the instance has an attribute with the same name as `NameError.name`, then it’ll suggest that you prepend `self.` when referencing the name.



**Note:** If you’re new to object-oriented programming in Python, then you can read up on how to use `self` in the context of [how to define a class](https://realpython.com/python3-object-oriented-programming/#how-to-define-a-class).



Before diving into the new and improved version, take a moment to explore the error message from Python 3.11 and before.


You want to build a `Greeter` class that has an instance method, `greet()`, that’ll say hello to your program’s users:


```
>>> class Greeter:
...     def __init__(self):
...         self.message = "Hello"
...     def greet(self, whom="World"):
...         print(message, whom)

```

There’s currently a bug in `.greet()`, which is that you’re attempting to reference the instance attribute `self.message` but forgot to add `self.` as part of the name.


If you attempt to use `.greet()`, on an instance of `Greeter` in Python 3.11 and before, then the raised `NameError` will only tell you that the name `message` isn’t defined:


```
>>> Greeter().greet()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 5, in greet
NameError: name 'message' is not defined

```

Throughout many Python versions, this was the only form that this error message could take. But Python 3.10 already introduced some improvements.


If you had a similar name in the [local scope](https://realpython.com/python-scope-legb-rule/#functions-the-local-scope), then Python 3.10 would give that name as a suggestion following the *Did you mean* pattern that you saw earlier:


```
>>> class Greeter:
...     def __init__(self):
...         self.message = "Hello"
...     def greet(self, whom="World"):
...  messenger = "Python"
...         print(message, whom)
...
>>> Greeter().greet()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 6, in greet
NameError: name 'message' is not defined. Did you mean: 'messenger'?

```

While that suggestion could be helpful, in practice it’s arguably more often misleading. In a situation where the name that raised the `NameError` is exactly equal to an instance attribute, it’s probably more likely that you forgot to add `self.` before the name.


Pablo raised an issue describing just that:



> 
> If we are in a method and a NameError is raised for the name `X`, first check if `X` is an attribute of the instance as `self.X` is likely more common than whatever closest match we find in the scope. ([Source](https://github.com/python/cpython/issues/99139#issue-1437146807))
> 
> 
> 


This issue addresses the potentially misleading error message that you saw further up.


After raising the issue, Pablo went ahead and wrote an [improvement for `NameError` suggestions for instances](https://github.com/python/cpython/pull/99140) that first checks whether the wrong name exists as an instance attribute. If it does, then the error message now suggests that you might’ve forgotten to use `self` with dot notation to reference the instance:


```
>>> class Greeter:
...     def __init__(self):
...         self.message = "Hello"
...     def greet(self, whom="World"):
...         print(message, whom)
...
>>> Greeter().greet()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 5, in greet
NameError: name 'message' is not defined. Did you mean: 'self.message'?

```

Suggesting `self.message` is definitely an actionable improvement over a plain error message that just states that the name you attempted to access isn’t defined. But this update also improves the situation when a suggested alternative name could be more misleading than helpful, as shown in the earlier example.


In Python 3.12, even if there’s a similar name in the local scope of the method, Python will still suggest an equally named instance attribute first:


```
>>> class Greeter:
...     def __init__(self):
...         self.message = "Hello"
...     def greet(self, whom="World"):
...  messenger = "Python"
...         print(message, whom)
...
>>> Greeter().greet()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "<stdin>", line 6, in greet
NameError: name 'message' is not defined. Did you mean: 'self.message'?

```

This improved pointer in the error message has the potential to help programmers more quickly identify a bug in their code. It might also help programmers who switched to Python from another language, as a commenter on the issue mentioned:



> 
> This is great! I learnt C++ before Python so I used to forget self all the time when I first started programming in Python. Thank you! ([Source](https://github.com/python/cpython/issues/99139#issuecomment-1304724314))
> 
> 
> 


Because other programming languages don’t use `self`, programmers who came to Python from a different language might forget to add it. A helpful message that points to the missing `self.` in an instance attribute can go a long way in resolving this confusion.


To tackle this improvement, Pablo [added code](https://github.com/pablogsal/cpython/blob/00706e4a8142e4e5695a9309e34b73ac9f715d39/Lib/traceback.py#L1041-L1046) that inspects the scope of the code object that raised the exception and then checks whether `"self"` exists as a key. If it does, then Python checks whether the instance referenced with `"self"` has an attribute that’s exactly equal to `wrong_name`. If it does, then the function returns before moving on to [later parts](https://github.com/pablogsal/cpython/blob/00706e4a8142e4e5695a9309e34b73ac9f715d39/Lib/traceback.py#L1048) that compute similar name suggestions based on [Levenshtein distance](https://en.wikipedia.org/wiki/Levenshtein_distance):



```
1037# cpython/Lib/traceback.py
1038
1039# ...
1040
1041# Check first if we are in a method and the instance
1042# has the wrong name as attribute
1043if "self" in frame.f_locals:
1044    self = frame.f_locals["self"]
1045    if hasattr(self, wrong_name):
1046        return f"self.{wrong_name}"
1047
1048# ...

```

The added code effectively checks whether the instance that you’re calling the method on has an instance attribute whose name is the same as what raised your `NameError`. If it does, then the error message will suggest that you add `self.` to the name that you’re using. If it doesn’t, then the code will continue and potentially compute suggestions, if there are similar names in the local scope.


You can reproduce the functionality in the code snippet that you’ve been working with in this section:



```
 1# local_self.py
 2
 3import inspect
 4
 5class Greeter:
 6    def __init__(self):
 7        self.message = "Hello"
 8
 9    def greet(self, whom="World"):
10 frame = inspect.currentframe()
11 wrong_name = "message"
12
13 if "self" in frame.f_locals:
14 self = frame.f_locals["self"]
15 if hasattr(self, wrong_name):
16 raise NameError(
17 (
18 f"name '{wrong_name}' is not defined. "
19 f"Did you mean: 'self.{wrong_name}'?"
20 )
21 )
22
23Greeter().greet()

```

In this approximation of the actual code, you used Python’s `inspect` module to access the context of `.greet()` in line 10. In line 11, you assigned `"message"` to the variable `wrong_name` to mirror how during actual exception handling, Python has access to the name that caused the `NameError` under the same variable name.


Then, in lines 13 to 21, you essentially copied the code that Pablo added to Python 3.12, only instead of returning a shorter string, you directly raised the `NameError` with an appropriate message.


If you run this code, then you’ll get a similar error message, following an approach that’s similar to what the improved `NameError` for instances does:



```
Traceback (most recent call last):
  File "/Users/martin/local_self.py", line 23, in <module>
    Greeter().greet()
  File "/Users/martin/local_self.py", line 16, in greet
    raise NameError(
NameError: name 'message' is not defined. Did you mean: 'self.message'?

```

You could even run your script now with an older version of Python and still get the same suggestion. Of course, it’s not feasible to add such a code snippet to each of your methods, but doing so here can help you better understand how the improvement was implemented in Python 3.12.


As with all updates, there are ways this could be extended, as well as boundaries that need to be set. In response to [a comment](https://github.com/python/cpython/issues/99139#issuecomment-1304836386) about extending the suggestion so that it would also trigger for class attributes, Pablo responded:



> 
> Thanks for the proposal!
> 
> 
> I will consider it but is unlikely that will happen as we are not interested in supporting all possible variations of this, especially since the name of the variable here (cls) is less standardised than “self” and is less common.
> 
> 
> Also, this will force us to distinguish the type of function to avoid inspecting random variables called “cls”. ([Source](https://github.com/python/cpython/issues/99139#issuecomment-1304871057))
> 
> 
> 


There likely won’t be an extension of this error message improvement that would also trigger for class attributes, as this would involve more complexities, such as needing to check if the error got raised in an [instance, class, or static method](https://realpython.com/instance-class-and-static-methods-demystified/).


But there’s still cause for celebration. After all, when Python 3.12 raises a `NameError` in an instance method, and the instance has an attribute with the same name as `NameError.name`, then it’ll suggest that you prepend `self.` when referencing the name.



Have you ever mixed up the order of the [keywords](https://realpython.com/python-keywords/) when you’re [importing a name from a module](https://realpython.com/python-modules-packages/#from-module_name-import-names)? Maybe you wrote `import ... from ...` instead of `from ... import ...`. It makes perfect sense grammatically, but Python doesn’t know what to do with it. The plain `SyntaxError` that Python used to give you probably didn’t help a lot in pointing you to a solution.


When Python 3.12 raises a `SyntaxError` from a switched-up `from`-`import` statement, then the updated error message will suggest that you fix the order.



**Note:** In this tutorial, you’ll see the short name **`from`-`import` statement** used for the Python language construct that seems to have different names in different sources across the Python community:


* `import-from` statement ([Source](https://docs.python.org/3/reference/import.html#submodules))
* `from <> import <>` syntax ([Source](https://docs.python.org/3/reference/import.html#package-relative-imports))
* `from <module_name> import <name(s)>` ([Source](https://realpython.com/python-modules-packages/#from-module_name-import-names))
* `from <module> import <name>` statement ([Source](https://docs.python.org/3.12/whatsnew/3.12.html#improved-error-messages))
* ‘from … import …’ ([Source](https://docs.python.org/3.12/whatsnew/3.12.html#improved-error-messages))


There doesn’t seem to be a clear agreement on how to refer to this construct. Calling it the `from`-`import` statement seems to be a readable compromise.



In a moment, you’ll explore the new error message. But first, take a look at what Python 3.11 and before offered.


Say your friend sent you an encrypted string that they asked you to decode with the help of a key saved in a variable called `d` in Python’s `this` module:



```
message = "Gunaxf sbe nyy gur nqqrq pynevgl va reebe zrffntrf, Cnoyb!"

```

Of course you want to decode the mysterious message! All eager to see the solution, you try to import the obscure key called `d` from `this`:


```
>>> import d from this
  File "<stdin>", line 1
    import d from this
             ^^^^
SyntaxError: invalid syntax

```

What greets you is yet another riddle: a `SyntaxError` pointing to `from`. While the little arrows under `from` look great, there’s not a lot of additional context.


If you attempt the same mixed-up import in Python 3.12, then you’ll instead be greeted with an actionable suggestion:


```
>>> import d from this
  File "<stdin>", line 1
    import d from this
    ^^^^^^^^^^^^^^^^^^
SyntaxError: Did you mean to use 'from ... import ...' instead?

```

Equipped with this helpful tip, you can quickly fix the import statement and access the mysterious `d` key to decipher your friend’s message:


```
>>> from this import d
The Zen of Python, by Tim Peters
...

>>> message = "Gunaxf sbe nyy gur nqqrq pynevgl va reebe zrffntrf, Cnoyb!"

>>> decoded_characters = []
>>> for character in message:
...     if character.isalpha():
...         decoded_characters.append(d.get(character, ""))
...     else:
...         decoded_characters.append(character)
...
>>> decoded_message = "".join(decoded_characters)

>>> print(decoded_message)
Thanks for all the added clarity in error messages, Pablo!

```

The implementation for this update is different from the previous ones, in that you’re not dealing with an exception, but instead with a [`SyntaxError`](https://realpython.com/invalid-syntax-python/). When you mix up the order of the `from`-`import` statement, then you aren’t writing valid Python code.


The parser doesn’t know what to do with this sequence of characters that’s meaningless from its perspective. It can’t convert it to bytecode, which means that you’re running into an error at the parsing stage.


That’s also why the code changes for this update needed to happen inside of the definition of Python’s [PEG](https://en.wikipedia.org/wiki/Parsing_expression_grammar) parser:



```
# cpython/Grammar/python.gram

# ...

import_stmt[stmt_ty]:
 | invalid_import
    | import_name
    | import_from

# ...

invalid_import:
 | a='import' dotted_name 'from' dotted_name {
 RAISE_SYNTAX_ERROR_STARTING_FROM(a, "Did you mean to use 'from ... import ...' instead?") }

# ...

```

In the code snippet above, you see the two changes highlighted. First, there’s an addition of `invalid_import`, which you can find in [line 198](https://github.com/pablogsal/cpython/blob/9f705afc159eb03172c8d1415f38f97897bbe8b0/Grammar/python.gram#L198) of `python.gram`. The new grammar rule is then implemented in [lines 1236 to 1238](https://github.com/pablogsal/cpython/blob/9f705afc159eb03172c8d1415f38f97897bbe8b0/Grammar/python.gram#L1236/L1238).


The `invalid_import` grammar rule defines the shape that triggers the `SyntaxError`. Specifically, it looks for an import that starts with the token `import`, followed by a name, then `from`, and then another name. If the parser encounters this structure, then it should raise a `SyntaxError` with a helpful message.


Currently, this improved error message only triggers when you mix up the order while attempting to import a single name. Importing multiple names at once still triggers the old error message:


```
>>> import s, d from this
  File "<stdin>", line 1
    import s, d from this
                ^^^^
SyntaxError: invalid syntax

```

The quest for improvements is everlasting, which is also true for error messages, and Pablo [might tackle](https://github.com/python/cpython/issues/98931#issuecomment-1458470104) including multiple names in the future.


In short, when Python 3.12 encounters `import ... from ...`, the parser triggers a `SyntaxError` with a new error message suggesting that you use `from ... import ...` instead.



Order isn’t the only problem that can arise when you’re importing. You can also misspell a name that you’re attempting to import from a module when using the `from`-`import` statement. With the `ImportError` that Python used to raise, you might’ve been left guessing as to what went wrong.


When Python 3.12 raises an `ImportError` from a failed `from ... import ...` statement, then the error message now includes suggestions for the name that you might want to import. The suggestions are based on available names in the module that you’re importing from.


As before, you’ll first refresh your memory on how Python 3.11 and earlier versions communicate such an error.


You want to [build a command-line interface](https://realpython.com/command-line-interfaces-python-argparse/) that uses [Python’s `pathlib`](https://realpython.com/python-pathlib/) to work with file path inputs, but you’re running into an error already while you’re drafting your program in the REPL:


```
>>> from pathlib import path
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: cannot import name 'path' from 'pathlib' (/pypath/pathlib.py)

```

That’s not a very motivating start to your new project! The error message tells you that Python couldn’t do what you asked it to do—but there’s not a lot of information about what went wrong.


In Python 3.12, the message is more actionable and points you right to the problem within your attempted import:


```
>>> from pathlib import path
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: cannot import name 'path' from 'pathlib' (/pypath/pathlib.py)
⮑ Did you mean: 'Path'?

```

What a relief! It looks like you only forgot to capitalize `Path`! With this helpful suggestion, you’re on your way and can continue working on your project:


```
>>> from pathlib import Path

```

The idea for this improvement for `ImportError` messages was raised at the beginning of 2022 in [gh-91058](https://github.com/python/cpython/issues/91058) and sparked some discussion before Pablo implemented it about six months later in [gh-98305](https://github.com/python/cpython/pull/98305).


Making this improvement happen without compromising performance wasn’t entirely straightforward. The implementation renames `PyErr_SetImportErrorSubclass` to [`_PyErr_SetImportErrorSubclassWithNameFrom`](https://github.com/pablogsal/cpython/blob/de553f63340230efdd2e2c05a60b8abca7b8992c/Python/errors.c#L979) in CPython’s source code for errors, and adds the new `PyObject` called `from_name`:



```
974/* cpython/Python/errors.c */
975
976/* ... */
977
978static PyObject *
979_PyErr_SetImportErrorSubclassWithNameFrom(
980 PyObject *exception, PyObject *msg,
981 PyObject *name, PyObject *path, PyObject* from_name)
982
983/* ... */

```

Further down in the code for this class, `from_name` is set to `Py_None` if there’s no value provided for it. After that, `from_name` is added to the kwargs of the object’s dictionary under the name `"name_from"`:



```
1020/* cpython/Python/errors.c */
1021
1022/* ... */
1023
1024if (PyDict_SetItemString(kwargs, "name_from", from_name) < 0) {
1025 goto done;
1026}
1027
1028/* ... */

```

The adapted error subclass is then used for both the pre-existing `PyErr_SetImportErrorSubclass` as well as the sparkling new `_PyErr_SetImportErrorWithNameFrom`.


The Python interpreter uses `_PyErr_SetImportErrorSubclassWithNameFrom()` to embellish `ImportError` objects with additional context, such as the name of the module where the error occurred. It now also adds the information of which name you attempted to import from a module, under `"name_from"`.


Later, Python can use this information to compute the suggested names in the [new code added](https://github.com/pablogsal/cpython/blob/de553f63340230efdd2e2c05a60b8abca7b8992c/Lib/traceback.py#L710-#L715) to `TracebackException`:



```
706# cpython/Lib/traceback.py
707
708# ...
709
710elif exc_type and issubclass(exc_type, ImportError) and \
711        getattr(exc_value, "name_from", None) is not None:
712    wrong_name = getattr(exc_value, "name_from", None)
713    suggestion = _compute_suggestion_error(exc_value, exc_traceback, wrong_name)
714    if suggestion:
715        self._str += f". Did you mean: '{suggestion}'?"
716
717# ...

```

In line 712, the name that you attempted to import from the module gets fetched from the newly added `"name_from"` attribute of the exception object, and then it gets assigned to `wrong_name`.


It was a bit more difficult to get access to that name, but now that you have access to it, you can pass it to `_compute_suggestion_error()` just like other names.


But accessing the name isn’t the only detail that’s implemented differently than in the similarly improved error message for names in the current scope that Pablo implemented in Python 3.10. When computing the suggestions for this improved `ImportError` message in Python 3.12, you want to compare `wrong_name` to names in the namespace of the module that you tried importing from.


Pablo [implemented this functionality](https://github.com/pablogsal/cpython/blob/de553f63340230efdd2e2c05a60b8abca7b8992c/Lib/traceback.py#L1024-L1029) by fetching `.name` from the raised `ImportError`, then passing it to the built-in [`__import__()`](https://docs.python.org/3/library/functions.html#import__) and passing the module to `dir()` to get access to the names in the module’s namespace:



```
1020# cpython/Lib/traceback.py
1021
1022# ...
1023
1024elif isinstance(exc_value, ImportError):
1025    try:
1026        mod = __import__(exc_value.name)
1027        d = dir(mod)
1028    except Exception:
1029        return None
1030
1031# ...

```

The code attempts to import the module through its name fetched from the exception, then assigns the return value of `dir()` to `d`. Just like with other name suggestions, `d` is used further down in `_compute_suggestion_error()` to calculate possible names.


Just like for names within the current namespace, the suggestions won’t trigger for long and short names, which you can confirm by [reading the tests](https://github.com/python/cpython/pull/98305/files#diff-8a52124ffceebd98f91bf96caa0f341ae109f79e190cd5acc1c56c8968d75252) that Pablo wrote as part of this pull request.


To recap, when Python 3.12 raises an `ImportError` from a failed `from ... import ...` statement, then the error message now includes suggestions for the name that you might want to import. The suggestions are based on available names in the module that you’re importing from.


You can look forward to ever better error messages in Python 3.12! The new Python version will be released in October 2023, but you can already preview the more readable and actionable error messages that are coming your way.


**In this tutorial, you’ve learned about:**


* Improvements in **readability** and **actionable messaging** of `NameError`, `SyntaxError`, and `ImportError`
* The **background and limitations** of these changes
* The **implementation details** in the CPython source code of the pull requests that made these error messages sparkle


There are many other improvements and features coming in Python 3.12. Keep track of [what’s new](https://docs.python.org/3.12/whatsnew/3.12.html) in the changelog for an up-to-date list. Are you looking forward to ever better error messaging in Python? Or is there another feature that you’re eagerly awaiting in Python 3.12? Share your thoughts in the comments below.



**Free Bonus:** [Click here to download sample code](https://realpython.com/bonus/python-312-code/) that’ll give you a sneak peek at Python 3.12, coming in October 2023.




