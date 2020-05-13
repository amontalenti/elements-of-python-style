# The Elements of Python Style

This document goes beyond PEP8 to cover the core of what I think of as great Python style. It is opinionated, but not too opinionated. It goes beyond mere issues of syntax and module layout, and into areas of paradigm, organization, and architecture. I hope it can be a kind of condensed ["Strunk & White"][strunk-white] for Python code.

[strunk-white]: https://en.wikipedia.org/wiki/The_Elements_of_Style

# Table of Contents

  * [The Elements of Python Style](#the-elements-of-python-style)
    * [Follow Most PEP8 Guidelines](#follow-most-pep8-guidelines)
    * [Flexibility on Line Length](#flexibility-on-line-length)
    * [Consistent Naming](#consistent-naming)
    * [Nitpicks That Aren't Worth It](#nitpicks-that-arent-worth-it)
    * [Writing Good Docstrings](#writing-good-docstrings)
    * [Paradigms and Patterns](#paradigms-and-patterns)
    * [A Little Zen for Your Code Style](#a-little-zen-for-your-code-style)
    * [Six of One, Half a Dozen of the Other](#six-of-one-half-a-dozen-of-the-other)
    * [Standard Tools and Project Structure](#standard-tools-and-project-structure)
    * [Some Inspiration](#some-inspiration)
    * [Contributors](#contributors)

## Follow Most [PEP8 Guidelines][pep8]

... but, be flexible on naming and line length.

PEP8 covers lots of mundane stuff like whitespace, line breaks between functions/classes/methods, imports, and warning against use of deprecated functionality. Pretty much everything in there is good.

The best tool to enforce these rules, while also helping you catch silly Python syntax errors, is [flake8][flake8].

PEP8 is meant as a set of guidelines, not rules to be strictly, or religiously, followed. Make sure to read the section of PEP8 that is titled: "A Foolish Consistency is the Hobgoblin of Little Minds." Also see Raymond Hettinger's excellent talk, ["Beyond PEP8"](https://www.youtube.com/watch?v=wf-BqAjZb8M) for more on this.

The only set of rules that seem to cause a disproportionate amount of controversy are around the line length and naming. These can be easily tweaked.

## Flexibility on Line Length

If the strict 79-character line length rule in `flake8` bothers you, feel free to ignore or adjust that rule. It's probably still a good rule-of-thumb -- like a "rule" that says English sentences should have 50 or fewer words, or that paragraphs should have fewer than 10 sentences. Here's the link to [flake8 config][f8config], see the `max-line-length` config option. Note also that often a `# noqa` comment can be added to a line to have a `flake8` check ignored, but please use these sparingly.

90%+ of your lines should be 79 characters or fewer, though, for the simple reason that "Flat is better than nested". If you find a function where all the lines are longer than this, something else is wrong, and you should look at your code rather than at your flake8 settings.

[pep8]: https://www.python.org/dev/peps/pep-0008/
[flake8]: https://flake8.readthedocs.org
[f8config]: https://flake8.readthedocs.org/en/latest/config.html

## Consistent Naming

On naming, following some simple rules can prevent a whole lot of team-wide grief.

### Preferred Naming Rules

Many of these were adapted from [the Pocoo team][pocoo].

- Class names: `CamelCase`, and capitalize acronyms: `HTTPWriter`, not `HttpWriter`.
- Variable names: `lower_with_underscores`.
- Method and function names: `lower_with_underscores`.
- Modules: `lower_with_underscores.py`. (But, prefer names that don't need underscores!)
- Constants: `UPPER_WITH_UNDERSCORES`.
- Precompiled regular expressions: `name_re`.

[pocoo]: https://flask.palletsprojects.com/en/1.1.x/styleguide/

You should generally follow these rules, unless you are mirroring some other tool's naming convention, like a database schema or message format.

You can also choose to use `CamelCase` for things that are class-like but not quite classes -- the main benefit of `CamelCase` is calling attention to something as a "global noun", rather than a local label or a verb. Notice that Python names `True`, `False`, and `None` use `CamelCase` even though they are not classes.

### Avoid Name Adornments

... like `_prefix` or `suffix_`. Functions and methods can have a `_prefix` notation to indicate "private", but this should be used sparingly and only for APIs that are expected to be widely used, and where the `_private` indicator assists with [information hiding][infohiding].

[infohiding]: http://c2.com/cgi/wiki?InformationHiding

PEP8 suggests using a trailing underscore to avoid aliasing a built-in, e.g.

```python
sum_ = sum(some_long_list)
print(sum_)
```

This is OK in a pinch, but it might be better to just choose a different name.

You should rarely use `__mangled` double-underscore prefixes for class/instance/method labels, which have special [name mangling behavior][mangling] -- it's rarely necessary. Never create your own names using `__dunder__` adornments unless you are implementing a Python standard protocol, like `__len__`; this is a namespace specifically reserved for Python's internal protocols and shouldn't be co-opted for your own stuff.

[mangling]: https://docs.python.org/2/tutorial/classes.html#private-variables-and-class-local-references

### Avoid One-Character Names

There are some one-character label names that are common and acceptable.

With `lambda`, using `x` for single-argument functions is OK. For example:

```python
encode = lambda x: x.encode("utf-8", "ignore")
```

With tuple unpacking, using `_` as a throwaway label is also OK. For example:

```python
_, url, urlref = data
```

This basically means, "ignore the first element."

Similar to `lambda`, inside list/dict/set comprehensions, generator expressions, or very short (1-2 line) for loops, a single-char iteration label can be used. This is also typically `x`, e.g.

```python
sum(x for x in items if x > 0)
```

to sum all positive integers in the sequence `items`.

It is also very common to use `i` as shorthand for "index", and commonly with the `enumerate` built-in. For example:

```python
for i, item in enumerate(items):
    print("%4s: %s" % (i, item))
```

Outside of these cases, you should rarely, perhaps **never**, use single-character label/argument/method names. This is because it just makes it impossible to `grep` for stuff.

### Use `self` and similar conventions

You should:

- always name a method's first argument `self`
- always name `@classmethod`'s first argument `cls`
- always use `*args` and `**kwargs` for variable argument lists

## Nitpicks That Aren't Worth It

There's nothing to gain from not following these rules, so you should just follow them.

### Always [inherit from `object`][newstyle] and use new-style classes

```python
# bad
class JSONWriter:
    pass

# good
class JSONWriter(object):
    pass
```

In Python 2, it's important to follow this rule. In Python 3, all classes implicitly inherit from `object` and this rule isn't necessary any longer.

### Don't repeat instance labels in the class

```python
# bad
class JSONWriter(object):
    handler = None
    def __init__(self, handler):
        self.handler = handler

# good
class JSONWriter(object):
    def __init__(self, handler):
        self.handler = handler
```

### Prefer [list/dict/set comprehensions][mapfilter] over map/filter.

```python
 # bad
map(truncate, filter(lambda x: len(x) > 30, items))

 # good
[truncate(x) for x in items if len(x) > 30]
```

Though you should prefer comprehensions for most of the simple cases, there are occasions where `map()` or `filter()` will be more readable, so use your judgment.

### Use parens `(...)` for continuations

```python
# bad
from itertools import groupby, chain, \
                      izip, islice

# good
from itertools import (groupby, chain,
                       izip, islice)
```

### Use parens `(...)` for fluent APIs

```python
# bad
response = Search(using=client) \
           .filter("term", cat="search") \
           .query("match", title="python")

# good
response = (Search(using=client)
            .filter("term", cat="search")
            .query("match", title="python"))
```

### Use implicit continuations in function calls

```python
# bad -- simply unnecessary backslash
return set((key.lower(), val.lower()) \
           for key, val in mapping.iteritems())

# good
return set((key.lower(), val.lower())
           for key, val in mapping.iteritems())
```

### Use `isinstance(obj, cls)`, not `type(obj) == cls`

This is because `isinstance` covers way more cases, including sub-classes and ABC's. Also, rarely use `isinstance` at all, since you should usually be doing duck typing, instead!

### Use `with` for files and locks

The `with` statement subtly handles file closing and lock releasing even in the case of exceptions being raised. So:

```python
# bad
somefile = open("somefile.txt", "w")
somefile.write("sometext")
return

# good
with open("somefile.txt", "w") as somefile:
    somefile.write("sometext")
return
```

### Use `is` when comparing to `None`

The `None` value is a singleton but when you're checking for `None`, you rarely want to actually call `__eq__` on the LHS argument. So:

```python
# bad
if item == None:
    continue

# good
if item is None:
   continue
```

Not only is the good form faster, it's also more correct. It's no more concise to use `==`, so just remember this rule!

### Avoid `sys.path` hacks

It can be tempting to do `sys.path.insert(0, "../")` and similar to control Python's import approach, but you should avoid these like the plague.

Python has a somewhat-complex, but very comprehensible, approach to module path resolution. You can adjust how Python loads modules via `PYTHONPATH` or via tricks like `setup.py develop`. You can also run Python using `-m` to good effect, e.g. `python -m mypkg.mymodule` rather than `python mypkg/mymodule.py`. You should not rely upon the current working directory that you run python out of for your code to work properly. David Beazley saves the day once more with his PDF slides which are worth a skim, ["Modules and Packages: Live and Let Die!"][modules]

[modules]: http://www.dabeaz.com/modulepackage/ModulePackage.pdf

### Rarely create [your own exception types][exceptiontypes]

... and when you must, don't make too many.

```python
 # bad
 class ArgumentError(Exception):
     pass
 ...
 raise ArgumentError(url)

 # good
 raise ValueError("bad value for url: %s" % url)
```

Note that Python includes [a rich set of built-in exception classes][ex-tree]. Leverage these appropriately, and you should "customize" them simply by instantiating them with string messages that describe the specific error condition you hit. It is most common to raise `ValueError` (bad argument), `LookupError` (bad key), or `AssertionError` (via the `assert` statement) in user code.

A good rule of thumb for whether you should create your own exception type is to figure out whether a caller should catch it **every time** they call your function. If so, you probably **should** make your own type. But this is relatively rare. A good example of an exception type that clearly had to exist is [tornado.web.HTTPError][http-error]. But notice how Tornado did not go overboard: there is one exception class for **all** HTTP errors raised by the framework or user code.

[ex-tree]: https://docs.python.org/2/library/exceptions.html#exception-hierarchy
[http-error]: http://www.tornadoweb.org/en/stable/web.html#tornado.web.HTTPError

### Short docstrings are proper one-line sentences

```python
# bad
def reverse_sort(items):
    """
    sort items in reverse order
    """

# good
def reverse_sort(items):
    """Sort items in reverse order."""
```

Keep the triple-quote's on the same line `"""`, capitalize the first letter, and include a period. Four lines become two, the `__doc__` attribute doesn't have crufty newlines, and the pedants are pleased!

### Use [reST for docstrings][docstrings]

It's done by the stdlib and most open source projects. It's supported out-of-the-box by Sphinx. Just do it! The Python `requests` module uses these to extremely good effect. See the [`requests.api`][requests-api] module, for example.

### Strip trailing whitespace

This is perhaps the ultimate nitpick, but if you don't do it, it will drive people crazy. There are no shortage of tools that will do this for you in your text editor automatically; here's [a link to the one I use for vim][whitespace].

[whitespace]: https://github.com/amontalenti/home/blob/master/.vim/bundle/whitespace/plugin/whitespace.vim

## Writing Good Docstrings

Here's a quick reference to using Sphinx-style reST in your function docstrings:

```python
def get(url, qsargs=None, timeout=5.0):
    """Send an HTTP GET request.

    :param url: URL for the new request.
    :type url: str
    :param qsargs: Converted to query string arguments.
    :type qsargs: dict
    :param timeout: In seconds.
    :rtype: mymodule.Response
    """
    return request('get', url, qsargs=qsargs, timeout=timeout)
```

Don't document for the sake of documenting. The way to think about this is:

```python
good_names + explicit_defaults > verbose_docs + type_specs
```

That is, in the example above, there is no need to say `timeout` is a `float`, because the default value is `5.0`, which is clearly a `float`. It is useful to indicate in the documentation that the semantic meaning is "seconds", thus `5.0` means 5 seconds. Meanwhile, the caller has no clue what `qsargs` should be, so we give a hint with the `type` annotation, and the caller also has no clue what to expect back from the function, so an `rtype` annotation is appropriate.

One last point. Guido once said that his key insight for Python is that, "code is read much more often than it is written." Well, a corollary of this is that **some documentation helps, but too much documentation hurts**.

You should basically only document functions you expect to be widely re-used. If you document every function in an internal module, you'll just end up with a less maintainable module, since the documentation needs to be refactored when the code is refactored. Don't "cargo cult" your docstrings and definitely don't auto-generate them with tooling!

[newstyle]: https://docs.python.org/2/reference/datamodel.html#new-style-and-classic-classes
[mapfilter]: http://www.artima.com/weblogs/viewpost.jsp?thread=98196
[exceptiontypes]: https://twitter.com/amontalenti/status/665338326396727297
[docstrings]: https://www.python.org/dev/peps/pep-0287/
[requests-api]: https://github.com/kennethreitz/requests/blob/master/requests/api.py

## Paradigms and Patterns

### Functions vs classes

You should usually prefer functions to classes. Functions and modules are the basic units of code re-use in Python, and they are the most flexible form. Classes are an "upgrade path" for certain Python facilities, such as implementing containers, proxies, descriptors, type systems, and more. But usually, functions are a better option.

Some might like the code organization benefits of grouping related functions together into classes. But this is a mistake. You should group related functions together into **modules**.

Though sometimes classes can act as a helpful "mini namespace" (e.g. with `@staticmethod`), more often a group of methods should be contributing to the internal operation of an object, rather than merely being a behavior grouping.

It's always better to have a `lib.time` module for time-related functions than to have a `TimeHelper` class with a bunch of methods you are forced to subclass in order to use! Classes proliferate other classes, which proliferates complexity and decreases readability.

### Generators and iterators

Generators and iterators are Python's most powerful features -- you should master the iterator protocol, the `yield` keyword, and generator expressions.

Not only are generators important for any function that needs to be called over a large stream of data, but they also have the effect of simplifying code by making it easy for you to write your own iterators. Refactoring code to generators often simplifies it while making it work in more scenarios.

Luciano Ramalho, author of "Fluent Python", has a 30-minute presentation, ["Iterators & Generators: the Python Way"](https://www.youtube.com/watch?v=z4P6hSa6K9g), which gives an excellent, fast-paced overview. David Beazley, author of "Python Essential Reference" and "Python Cookbook", has a mind-bending three-hour video tutorial entitled ["Generators: The Final Frontier"](https://www.youtube.com/watch?v=5-qadlG7tWo) that is a satisfying exposition of generator use cases. Mastering this topic is worth it because it applies everywhere.

### Declarative vs imperative

You should prefer declarative to imperative programming. This is code that says  **what** you want to do, rather than code that describes **how** to do it. Python's [functional programming guide][func] includes some good details and examples of how to use this style effectively.

[func]: https://docs.python.org/3/howto/functional.html

You should use lightweight data structures like `list`, `dict`, `tuple`, and `set` to your advantage. It's always better to lay out your data, and then write some code to transform it, than to build up data by repeatedly calling mutating functions/methods.

An example of this is the common list comprehension refactoring:

```python
# bad
filtered = []
for x in items:
    if x.endswith(".py"):
        filtered.append(x)
return filtered
```

This should be rewritten as:

```python
# good
return [x
        for x in items
        if x.endswith(".py")]
```

But another good example is rewriting an `if`/`elif`/`else` chain as a `dict` lookup.

### Prefer "pure" functions and generators

This is a concept that we can borrow from the functional programming community.  These kinds of functions and generators are alternatively described as "side-effect free", "referentially transparent", or as having "immutable inputs/outputs".

As a simple example, you should avoid code like this:

```python
# bad
def dedupe(items):
    """Remove dupes in-place, return items and # of dupes."""
    seen = set()
    dupe_positions = []
    for i, item in enumerate(items):
        if item in seen:
            dupe_positions.append(i)
        else:
            seen.add(item)
    num_dupes = len(dupe_positions)
    for idx in reversed(dupe_positions):
        items.pop(idx)
    return items, num_dupes
```

This same function can be written as follows:

```python
# good
def dedupe(items):
    """Return deduped items and # of dupes."""
    deduped = set(items)
    num_dupes = len(items) - len(deduped)
    return deduped, num_dupes
```

This is a somewhat shocking example. In addition to making this function pure, we also made it much, much shorter. It's not only shorter: it's better. Its purity means `assert dedupe(items) == dedupe(items)` always holds true for the "good" version. In the "bad" version, `num_dupes` will **always** be `0` on the second call, which can lead to subtle bugs when using the function.

This also illustrates imperative vs declarative style: the function now reads like a description of what we need, rather than a set of instructions to build up what we need.

### Prefer simple argument and return types

Functions should operate on data, rather than on custom objects, wherever possible. Prefer simple argument types like `dict`, `set`, `tuple`, `list`, `int`, `float`, and `bool`. Upgrade from there to standard library types like `datetime`, `timedelta`, `array`, `Decimal`, and `Future`. Only upgrade to your own custom types when absolutely necessary.

As a good rule of thumb for whether your function is simple enough, ask yourself whether its arguments and return values could always be JSON-serializable. It turns out, this rule of thumb matters more than you might think: JSON-serializability is often a prerequisite to make the functions usable in parallel computing contexts. But, for the purpose of this document, the main benefits are: readability, testability, and overall function simplicity.

### Avoid "traditional" OOP

In "traditional OOP languages" like Java and C++, code re-use is achieved through class hierarchies and polymorphism, or so those languages claim. In Python, though we have the ability to subclass and to do class-based polymorphism, in practice, these capabilities are used rarely in idiomatic Python programs.

It's more common to achieve re-use through modules and functions, and it's more common to achieve dynamic dispatch through duck typing. If you find yourself using super classes as a form of code re-use, stop what you're doing and reconsider. If you find yourself using lots of polymorphism, consider whether one of Python's dunder protocols or duck typing strategies might apply better.

See also the excellent Python talk, ["Stop Writing Classes"][stop-classes], by a Python core contributor. In it, the presenter suggests that if you have built a class with a single method that is named like a class (e.g. `Runnable.run()`), then what you've done is modeled a function as a class, and you should just stop. Since in Python, functions are "first-class", there is **no reason** to do this!

[stop-classes]: https://www.youtube.com/watch?v=o9pEzgHorH0

### Mixins are sometimes OK

One way to do class-based re-use without going overboard on type hierarchies is to use Mixins. Don't overuse these, though. "Flat is better than nested" applies to type hierarchies, too, so you should avoid introducing needless required layers of hierarchy just to decompose behavior.

Mixins are not actually a Python language feature, but are possible thanks to its support for multiple inheritance. You can create base classes that "inject" functionality into your subclass without forming an "important" part of a type hierarchy, simply by listing that base class as the first entry in the `bases` list. An example:

```python
class APIHandler(AuthMixin, RequestHandler):
    """Handle HTTP/JSON requests with security."""
```

The order matters, so may as well remember the rule: `bases` forms a hierarchy bottom-to-top. One readability benefit here is that everything you need to know about this class is contained in the `class` definition itself: "it mixes in auth behavior and is a specialized Tornado RequestHandler."

### Be careful with frameworks

Python has a slew of frameworks for web, databases, and more. One of the joys of the language is that it's easy to create your own frameworks. When using an open source framework, you should be careful not to couple your "core code" too closely to the framework itself.

When considering building your own framework for your code, you should err on the side of caution. The standard library has a lot of stuff built-in, PyPI has even more, and usually, [YAGNI applies][yagni].

[yagni]: http://c2.com/cgi/wiki?YouArentGonnaNeedIt

### Respect metaprogramming

Python supports "metaprogramming" via a number of features, including decorators, context managers, descriptors, import hooks, metaclasses and AST transformations.

You should feel comfortable using and understanding these features -- they are a core part of the language and are fully supported by it. But you should realize that when you use these features, you are opening yourself up to complex failure scenarios. Thus, treat the creation of metaprogramming facilities for your code similarly to the decision to "build your own framework". They amount to the same thing. When and if you do it, make the facilities into their own modules and document them well!

### Don't be afraid of "dunder" methods

Many people conflate Python's metaprogramming facilities with its support for "double-underscore" or "dunder" methods, such as `__getattr__`.

As described in  the blog post, ["Python double-under, double-wonder"][dunder], there is nothing "special" about dunders. They are nothing more than a lightweight namespace the Python core developers picked for all of Python's internal protocols. After all, `__init__` is a dunder, and there's nothing magic about it.

It's true that some dunders can create more confusing results than others -- for example, it's probably not a good idea to overload operators without good reason. But many of them, such as `__repr__`,  `__str__`, `__len__`, and `__call__` are really full parts of the language you should be leveraging in idiomatic Python code. Don't shy away!

[dunder]: http://www.pixelmonkey.org/2013/04/11/python-double-under-double-wonder

## A Little Zen for Your Code Style

Barry Warsaw, one of the core Python developers, once said that it frustrated him that "The Zen of Python" ([PEP 20][pep20]) is used as a style guide for Python code, since it was originally written as a poem about Python's **internal** design. That is, the design of the language and language implementation itself. One can acknowledge that, but a few of the lines from PEP 20 serve as pretty good guidelines for idiomatic Python code, so we'll just go with it.

[pep20]:  https://www.python.org/dev/peps/pep-0020/
[pocoo-naming]: http://www.pocoo.org/internal/styleguide/#naming-conventions

### Beautiful is better than ugly

This one is subjective, but what it usually amounts to is this: will the person who inherits this code from you be impressed or disappointed? What if that person is you, three years later?

### Explicit is better than implicit

Sometimes in the name of refactoring out repetition in our code, we also get a little bit abstract with it. It should be possible to translate the code into plain English and basically understand what's going on. There shouldn't be an excessive amount of "magic".

### Flat is better than nested

This one is really easy to understand. The best functions have no nesting, neither by loops nor `if` statements. Second best is one level of nesting. Two or more levels of nesting, and you should probably start refactoring to smaller functions.

Also, don't be afraid to refactor a nested if statement into a multi-part boolean conditional. For example:

```python
# bad
if response:
    if response.get("data"):
        return len(response["data"])
```

is better written as:

```python
# good
if response and response.get("data"):
    return len(response["data"])
```

### Readability counts

Don't be afraid to add line-comments with `#`. Don't go overboard on these or over-document, but a little explanation, line-by-line, often helps a whole lot. Don't be afraid to pick a slightly longer name because it's more descriptive. No one wins any points for shortening "`response`" to "`rsp`". Use doctest-style examples to illustrate edge cases in docstrings. Keep it simple!

### Errors should never pass silently

The biggest offender here is the bare `except: pass` clause. Never use these. Suppressing **all** exceptions is simply dangerous. Scope your exception handling to single lines of code, and always scope your `except` handler to a specific type. Also, get comfortable with the `logging` module and `log.exception(...)`.

### If the implementation is hard to explain, it's a bad idea

This is a general software engineering principle -- but applies very well to Python code. Most Python functions and objects can have an easy-to-explain implementation. If it's hard to explain, it's probably a bad idea. Usually you can make a hard-to-explain function easier-to-explain via "divide and conquer" -- split it into several functions.

### Testing is one honking great idea

OK, we took liberty on this one -- in "The Zen of Python", it's actually "namespaces" that's the honking great idea.

But seriously: beautiful code without tests is simply worse than even the ugliest tested code. At least the ugly code can be refactored to be beautiful, but the beautiful code can't be refactored to be verifiably correct, at least not without writing the tests! So, write tests! Please!

## Six of One, Half a Dozen of the Other

This is a section for arguments we'd rather not settle. Don't rewrite other people's code because of this stuff. Feel free to use these forms interchangeably.

### `str.format` vs overloaded format `%`

`str.format` is more robust, yet `%` with `"%s %s"` printf-style strings is more concise. Both will be around forever.

Remember to use unicode strings for your format pattern, if you need to preserve unicode:

```python
u"%s %s" % (dt.datetime.utcnow().isoformat(), line)
```

If you do end up using `%`, you should consider the `"%(name)s"` syntax which allows you to use a dictionary rather than a tuple, e.g.

```python
u"%(time)s %(line)s" % {"time": dt.datetime.utcnow().isoformat(), "line": line}
```

Also, don't re-invent the wheel. One thing `str.format` does unequivocally better is support various [formatting modes][str-format], such as humanized numbers and percentages. Use them.

But use whichever one you please. We choose not to care.

[str-format]: https://docs.python.org/2/library/string.html#formatspec

### `if item` vs `if item is not None`

This is unrelated to the earlier rule on `==` vs `is` for `None`. In this case, we are actually taking advantage of Python's "truthiness rules" to our benefit in `if item`, e.g. as a shorthand "item is not None or empty string."

Truthiness is a [tad complicated][truth-values] in Python and certainly the latter is safer against some classes of bugs. The former, however, is very common in much Python code, and it's shorter. We choose not to care.

[truth-values]: https://docs.python.org/2/library/stdtypes.html#truth-value-testing

### Implicit multi-line strings vs triple-quote `"""`

Python's compiler will automatically join multiple quoted strings together into a single string during the parse phase if it finds nothing in between them, e.g.

```python
msg = ("Hello, wayward traveler!\n"
       "What shall we do today?\n"
       "=>")
print(msg)
```

This is roughly equivalent to:

```python
msg = """Hello, wayward traveler!
What shall we do today?
=>"""
print(msg)
```

In the former's case, you keep the indentation clean, but need the ugly newline characters. In the latter case, you don't need the newlines, but break indentation. We choose not to care.

### Using `raise` with classes vs instances

It turns out Python lets you pass either an exception **class** or an exception **instance** to the `raise` statement. For example, these two lines are roughly equivalent:

```python
raise ValueError
raise ValueError()
```

Essentially, Python turns the [first line into the second automatically][raises]. You should probably prefer the second form, if for no other reason than to **actually provide a useful argument**, like a helpful message about why the `ValueError` occurred. But these two lines **are** equivalent and you shouldn't rewrite one style into the other just because. We choose not to care.

[raises]: https://docs.python.org/2.7/reference/simple_stmts.html#the-raise-statement

## Standard Tools and Project Structure

We've made some choices on "best-of-breed" tools for things, as well as the very minimal starting structure for a proper Python project.

### The Standard Library

- `import datetime as dt`: always import `datetime` this way
- `dt.datetime.utcnow()`: preferred to `.now()`, which does local time
- `import json`: the standard for data interchange
- `from collections import namedtuple`: use for lightweight data types
- `from collections import defaultdict`: use for counting/grouping
- `from collections import deque`: a fast double-ended queue
- `from itertools import groupby, chain`: for declarative style
- `from functools import wraps`: use for writing well-behaved decorators
- `argparse`: for "robust" CLI tool building
- `fileinput`: to create quick UNIX pipe-friendly tools
- `log = logging.getLogger(__name__)`: good enough for logging
- `from __future__ import absolute_import`: fixes import aliasing

### Common Third-Party Libraries

- `python-dateutil` for datetime parsing and calendars
- `pytz` for timezone handling
- `tldextract` for better URL handling
- `msgpack-python` for a more compact encoding than JSON
- `futures` for Future/pool concurrency primitives
- `docopt` for quick throwaway CLI tools
- `py.test` for unit tests, along with `mock` and `hypothesis`

### Local Development Project Skeleton

For all Python packages and libraries:

- no `__init__.py` in root folder: give your package a folder name!
- `mypackage/__init__.py` preferred to `src/mypackage/__init__.py`
- `mypackage/lib/__init__.py` preferred to `lib/__init__.py`
- `mypackage/settings.py` preferred to `settings.py`
- `README.rst` describes the repo for a newcomer; use reST
- `setup.py` for simple facilities like `setup.py develop`
- `requirements.txt` describes package dependencies for `pip`
- `dev-requirements.txt` additional dependencies for tests/local
- `Makefile` for simple (!!!) build/lint/test/run steps

Also, always [pin your requirements](http://nvie.com/posts/better-package-management/).

## Some Inspiration

The following links may give you some inspiration about the core of writing Python code with great style and taste.

- Python's stdlib [`Counter` class][Counter], implemented by Raymond Hettinger
- The [`rq.queue` module][rq], originally by Vincent Driessen
- This document's author also wrote [this blog post on "Pythonic" code][idiomatic]

Go forth and be Pythonic!

```
$ python
>>> import antigravity
```

[Counter]: https://github.com/python/cpython/blob/57b569d8af2b3263c5d9e6d75fb308f89ea17ac6/Lib/collections/__init__.py#L446-L841
[rq]: https://github.com/nvie/rq/blob/master/rq/queue.py
[idiomatic]: http://www.pixelmonkey.org/2010/11/03/pythonic-means-idiomatic-and-tasteful

## Contributors

- Andrew Montalenti ([twitter][amontalenti]): original author
- Vincent Driessen ([twitter][nvie]): edits and suggestions
- William Feng ([github][williamfzc]): translation to zh-cn

[amontalenti]: https://twitter.com/amontalenti
[nvie]: https://twitter.com/nvie
[williamfzc]: https://github.com/williamfzc

---

Like good Python style? Then perhaps you'd like to [work on our team of Pythonistas][tweet] at Parse.ly!

[tweet]: https://twitter.com/amontalenti/status/682968375702716416
