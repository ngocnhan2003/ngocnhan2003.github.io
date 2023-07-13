---
title: Python any() built-in function
permalink: /series/builtin-any/
description: Return True if any element of the iterable is true. If the iterable is empty, return False.
---


<base-disclaimer>
  <base-disclaimer-title>
    From the <a target="_blank" href="https://docs.python.org/3/library/functions.html#any">Python 3 documentation</a>
  </base-disclaimer-title>
  <base-disclaimer-content>
    Return True if any element of the iterable is true. If the iterable is empty, return False.
  </base-disclaimer-content>
</base-disclaimer>

## Examples

```python
>>> any([False, False, False])
# False

>>> any((0, True, False))
# True

>>> any({0, 0, 0})
# False
```
