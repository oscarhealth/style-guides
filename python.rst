====================
 Python Style Guide
====================

Background
====

The goal of this document is to guide the style of Python code at
Oscar to reduce bugs and improve readability.

Many teams use the `black <https://github.com/psf/black>`_
auto-formatter to avoid having to worry (or even worse, argue!) about
formatting. We recommend using it, but it does not cover everything
here.

Code of Conduct
===============

- Boy Scout Rule: Always check a module in cleaner than you checked it
  out. Clean-up and compliance to the style guide is the
  responsibility of whoever touches the code.
- Don't Boil the Ocean: if you weren't already working on a module for a diff, you
  probably shouldn't be cleaning it up. It confuses reviewers and makes it harder
  to spot bugs. (Feel free to open a separate diff for the clean-up, though!)
- Don't be afraid to ask for help or clarification. We're all
  learning.

Language Rules
==============

Testing
-------

Use pytest-style asserts in unit tests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Use ``assert`` statements in unit tests. `Our test runner
  <https://docs.pytest.org/en/latest/assert.html>`_ will intelligently
  report the failure, so you can avoid the many names of JUnit
  legacy methods (self.assert*). It's easier to read and write.

    .. code-block:: python

        # Good
        assert foo == bar
        assert foo != bar
        assert foo in bar
        assert {1, 2, 3} == {3, 2, 1}
        with pytest.raises(SomeException):
            do_something()

        # Bad
        self.assertEqual(foo, bar)
        self.assertNotEqual(foo, bar)
        self.assertIn(foo, bar)
        self.assertSetEqual({1, 2, 3}, {3, 2, 1})
        with self.assertRaises(SomeException):
            do_something()

- Parameterize tests instead of copy-pasting or using
  for loops.

    .. code-block:: python

            # Good
            @parameterized([
                (2, 2, 4),
                (2, 3, 8),
                (1, 9, 1),
                (0, 9, 0),
            ])
            def test_pow(base, exponent, expected
               assert math.pow(base, exponent) == expected

            # Bad
            def test_pow():
                assert math.pow(2, 2) == 4

            def test_pow2():
                assert math.pow(2, 3) == 8

            def test_pow3():
                assert math.pow(1, 9) == 1

            def test_pow4():
                assert math.pow(0, 9) == 0

- `Familiarize yourself with pytest's <https://docs.pytest.org/en/7.2.x/getting-started.html#create-your-first-test>`_
  features. For example, you can just write test functions at the top level instead of needing to subclass
  ``unittest.TestCase``.

Lint
----

Lint is important in Python due to the dynamic nature of the
language. However, spurious warnings and errors do happen.

Disabling lint rules
~~~~~~~~~~~~~~~~~~~~

- When disabling warnings, prefer the symbolic name to the
  alpha-numeric code. `Pylint messages
  <http://docs.pylint.org/features.html>`_.

Bad
+++

.. code-block:: python

   # pylint: disable=C0302

Good
++++

.. code-block:: python

   # pylint: disable=max-module-lines

Imports
-------

Import packages and modules only, do not import objects or functions directly
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Once an object has been imported into another module’s namespace, a
  reference to that object lives in two places. Changing either
  reference will result in two separate objects in each
  namespace. See:
  https://docs.python.org/2/howto/doanddont.html#from-module-import-name1-name2

- It's easier to read.

Bad
+++

.. code-block:: python

   from geordi.services.base import OscarServiceBase

   class ExampleService(OscarServiceBase):
       pass

.. code-block:: python

   from utils.iterables import chunk
 
   for c in chunk([1, 2, 3, 4, 5], 2):
       pass

Good
++++

.. code-block:: python

   import math
   math.abs(-10)

.. code-block:: python

   import geordi.services.base as geordi_base

   class ExampleService(geordi_base.OscarServiceBase):
       pass



- When you write new classes or modules, make sure to name them in a
  way that reads well when imported this way, instead of being redundant.

Good
++++

.. code-block:: python

   from provider_data import kafka

   kafka.BatchedConsumer(batch_size=5)

Bad
+++

.. code-block:: python

   from provider_data import kafka_consumer

   kafka_consumer.BatchedKafkaConsumer(batch_size=5)


- As the link notes, this "don't" is much weaker than others. If a
  module name doesn't read well, then importing the objects you need
  may make things more readable (for example, @parameterized.parameterized).
  Unless you're sure, though, stick to the rule.

Do not use wildcard imports ``from foo import *``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- This clutters a namespace in a way that is completely out of the
  control of the importer. Imagine redefining ``list`` or ``dict`` in
  the imported module.

Prefer importing at the top of a module, and only at the top of a module
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Do not import in function bodies
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Often this is done to circumvent circular imports. Refactor these
  instead.

- Rarely this may be done to avoid side effects in imported third
  party modules. This is an acceptable exception.

- Rarely this may be done to avoid loading modules. This may be
  acceptable if the system is otherwise not used or imported anywhere
  else. Example: debug middleware.


Deliberately order imports
~~~~~~~~~~~~~~~~~~~~~~~~~~

- Organize imports so they are easy to find. Use three sections
  separated by a new line. The three sections (in order) are:

  - Standard Library Imports

  - Third Party Imports

  - Project Imports

- Within each section, imports should be sorted lexicographically,
  ignoring case, according to each module's full package path. Import
  statements of the form ``import module`` should always precede
  import statements of the form ``from module import identifier``.

- Your IDE should be able to do this for you (for example: PyCharm's "Optimize Imports" action).
  If it can't, you can use the ``isort <https://pycqa.github.io/isort/>`` _ tool instead.

Modules
-------

Avoid global variables
~~~~~~~~~~~~~~~~~~~~~~

Exceptions
++++++++++

- Constants, which should be denoted by UPPER_SNAKE_CASE.

- If absolutely necessary, internalize and provide access through
  functions or accessors.

Avoid excessive side-effects
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Module side-effects should be limited to mutating values in that
  module only.

- Side-effects at import should be as limited as possible, and should
  also not interact with other modules or do anything that can fail
  (e.g. network IO).

Exceptions
----------

Do not catch-all without re-raising
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Consider catching specific Exception classes in these cases, as not
  all Exceptions are program errors (e.g. ``StopIteration``,
  ``KeyboardInterrupt``). See:
  https://docs.python.org/2/library/exceptions.html#exception-hierarchy

Do not use ``assert`` outside of tests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- The python interpreter removes ``assert``  statements when we
  run it with the ``-O`` flag. We use that flag in production.
  Use ``raise`` instead.

Nested classes and functions
----------------------------

Nested classes and functions are ok and useful
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Be aware that they cannot typically be serialized.

Nested functions cannot write to values in an enclosing scope
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Workarounds to do so (such as mutating a dictionary in enclosing
  scope) should be avoided.

List, generator and dict comprehensions
---------------------------------------

Keep it simple
~~~~~~~~~~~~~~

- Complicated comprehensions are difficult to read and
  understand. Each component (mapping, for, filter) should fit on a
  single line. Do not nest comprehensions.

Use generators where possible
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Prefer generator comprehensions to list comprehension when possible.

Lambda
------

Keep it simple
~~~~~~~~~~~~~~

- Lambdas should fit on a single line.

Beware the binding
~~~~~~~~~~~~~~~~~~

- If you need to bind to a variable in an outer scope, you probably
  need to use the form ``lambda x=x: f(x)``.

- See discussion here: http://markmail.org/message/fypalne4rp5curta or
  here: http://docs.python-guide.org/en/latest/writing/gotchas/

Conditionals
------------

Keep it simple
~~~~~~~~~~~~~~

- Should be simple and fit on a single line.

- Should be limited to assignment and avoid side-effects.

Prefer if/else ternary to and/or ternary
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Prefer the syntax ``a = b if c else d`` to ``a = c and b or d``

- Using simply ``or`` with truth-value testing is ok, e.g. ``a = a or
  b``

Default Arguments
-----------------

Never use mutable default arguments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Default argument values are global values. Mutable objects as
  defaults are almost never desired.

Bad
+++

.. code-block:: python

   # The default value for a will be shared across all calls to foo.
   def foo(a=[]):
       a.append(1)

Good
++++

.. code-block:: python

   # Use None in these cases, and test using is None:
   def foo(a=None):
       a = a or []
       a.append(1)

Properties
----------

Use @property versus getter/setter methods
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Use ``@property`` to override property access.

- Do not use java-style property accessors, e.g. ``get_foo`` or
  ``set_foo``.

- Do not use ``@property`` for attributes that require heavy
  computation (ie: parsing json). Let attribute access signal to a
  developer that accessing this value is essentially free.

Prefer instance variables to properties
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Use instance variables if there is no need to capture property
  access. The mantra from Java to always use accessors is not valid in
  Python, since property access can be overridden after the fact.

Avoid mutable class properties except where explicitly needed
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Setting properties on a class can be used as a default value for
  instances which is overwritten on the instance when set by an
  instance, but mutable values may be mutated class-wide.

- Beware of accessing class properties through an instance handle
  (e.g. self). Instance properties shadow class properties.

- Class properties are very nearly module globals, and should be
  treated as such.

Implicit True/False
-------------------

Use the implicit True/False provided
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Prefer testing for implicit True/False versus tests such as
  ``len(foo) == 0``.

- Implement ``__len__`` or ``__nonzero__`` when appropriate.

- See: https://docs.python.org/2/library/stdtypes.html#truth-value-testing

Use ``is`` for comparing against singletons
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Most notably: ``is None``.

- Useful to test for sentinels.

Magic methods and values
------------------------

Do not access magic values directly if possible
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Use ``type()`` to retrieve an object’s class/type versus ``__class__``.

- Not all classes contain a ``__dict__``.

- If there is no built-in for accessing a magic value, it may be
  accessed directly, though care should be taken to understand the
  full implications (e.g. ``__file__`` does not exist on objects
  created in an interactive interpreter).

Do not call magic methods directly
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Invoke magic methods via their syntax or built-ins:

  - ``__repr__``: ``repr(foo)``

  - ``__lt__``: ``a < b``

  - ``__str__``: ``str(foo)``

  - ``__nonzero__``: ``bool(foo); if foo:``

  - ``__len__``: ``len(foo)``

- It may sometimes be necessary to call magic methods directly, such
  as ``__init__`` in a subclass.

Functional programming built-ins
--------------------------------

Avoid map and filter when the argument would be a lambda
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- If the argument to map or filter is a lambda, use a list
  comprehension or for loop instead.

Avoid reduce
~~~~~~~~~~~~

- Use a for loop to reduce instead of the built-in function.

Decorators
----------

Use sparingly
~~~~~~~~~~~~~

- Errors in decorators are nearly impossible to recover from.

- Decorators execute at module load time, making them a module import
  side-effect.

- Decorators can change anything about the decorated
  class/function/method.

- Decorators should be thoroughly tested and robust.

- A decorator provided with valid inputs should always succeed.

Avoid external dependencies in decorators
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Because decorators evaluate at module load time, they should not
  rely on the existence of external resources which may not exist.
  This is a special case of the general rule that modules should not
  have side-effects at import time.

Threading/Concurrency
---------------------

Never rely on the atomicity of builtin types and functions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Some access in Python is guaranteed to be synchronized or atomic,
  but not all. Locks and semaphores are very cheap, use them instead
  of relying on built-in atomicity.

- Atomicity assumptions and guarantees change from platform to
  platform.

- Furthermore, assume nothing in the standard library is thread-safe
  unless it is clearly documented as synchronized or atomic
  (e.g. Queue_).

Share memory by communicating
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Use Queue_ to communicate versus locking and sharing memory. Queue_
  is synchronized and thread-safe.

- If sharing memory is necessary, try to use `threading.Condition`_.

Never wait on a thread during import
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Imports are guarded by an import lock (this is not the GIL, and it
  exists on platforms where the GIL does not) and can result in
  deadlock.

- See:
  https://docs.python.org/2/library/threading.html#importing-in-threaded-code

Beware of mixing synchronization primitives
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Tornado, gevent, Twisted, etc all provide their own synchronization
  primitives for use on their event-driven platforms. Using primitives
  from the threading module in this case will cause a deadlock in the
  event loop.

- Mixing synchronization primitives may be necessary in some rare
  situations, such as mixing threaded and asynchronous code.

Synchronization is cheap
~~~~~~~~~~~~~~~~~~~~~~~~

- Locks are cheap. It’s easier to remove locks later than to debug a
  synchronization issue.

Signals and interrupts
~~~~~~~~~~~~~~~~~~~~~~

- Beware of the issues around sending signals to a multi-threaded
  Python application:
  http://snakesthatbite.blogspot.com/2010/09/cpython-threading-interrupting.html

Power Features
--------------

Python is a very rich and powerful language that attempts to toe the
line between something like Ruby and something like Java. Power
features should be used sparingly. It might be easier to write, but it
can end up being hard to understand. Readability should always win
over writability.

Metaclasses
~~~~~~~~~~~

- Avoid writing metaclasses. If you feel that you absolutely must use
  a metaclass, consider a class decorator (with the caveats and
  warnings mentioned above). If you still feel you must use a
  metaclass, please get a second opinion.

- Use metaclasses sparingly. `abc.ABCMeta`_ is probably the only
  metaclass that should ever be used directly.

- If providing a metaclass for use, consider hiding the metaclass from
  users and placing it on a base class which is public.

Descriptor Protocol
~~~~~~~~~~~~~~~~~~~

- Understand the implications of a non-data descriptor versus a
  data-descriptor before setting out.

- Descriptors are useful and powerful, but also difficult to
  debug. Each possible invocation should be thoroughly tested and
  understood. See:
  https://docs.python.org/2/howto/descriptor.html#invoking-descriptors

Monkey Patching
~~~~~~~~~~~~~~~

- Monkey Patching should be considered a last ditch-effort. Monkey
  patching may have unintended consequences with other modules. It is
  almost certainly better to fork and modify code that needs monkey
  patching.

Mixins
~~~~~~

Bad
+++

- Implementing methods through a class’s public interface may decrease
  encapsulation. If a method can be implemented purely through a
  class’s public interface, consider a free function, which keeps the
  class interface minimal.

- Mixins are harder to extend and change later, as modifying a mixin’s
  internal interface modifies every class that uses it. Explicit
  composition relies only on the public interface of the composed
  objects, and the internals are free to change.

Good
++++

- Because of magic methods in Python, mixins may be very beneficial to
  adapt a class to a specific interface that interacts with Python’s
  syntax. Examples of this are the `collections abstract base
  classes`_. Generally, a purely functional mixin which adapts one
  well-known interface to another well-known interface is an
  acceptable use of the mixin pattern.

Formatting
===========

Many teams use the `black <https://github.com/psf/black>`_
auto-formatter to avoid having to worry (or even worse, argue!) about
formatting. We recommend using it.

PEP 8
-----

- Follow the style recommendations in `PEP 8`_.

Line Length
-----------

- Maximum line length is 120 characters.

Documentation
-------------

Docstrings
~~~~~~~~~~
- When in doubt, follow `PEP 257`

- The first line of the docstring should be a summary that fits on a
  single line. This may be sufficient for simple cases.

- The rest of the docstring should follow, separated from the summary
  by a blank line.

- Prefer type annotations over docstrings for documenting types.

Example method docstring - note the use of type hinting, as well as descriptions:

.. code-block:: python

   def send_message(sender: str, recipient: str, message_body: str, priority: Optional[int]=None) -> int:
       """Send a message to a recipient.

       :param priority: The priority of the message, can be a number 1-5
       :return: the message id
       :raises ValueError: if the message_body exceeds 160 characters
       """
       pass

README
~~~~~~

- Supply a README.md (markdown format) or README.rst (restructuredText
  format) to document any oddities, usage or gotchas. A readme is not
  strictly required.

Comments
~~~~~~~~

- If a block of code is probably going to be discussed in a code
  review, explain it in a comment.

- Assume the next person knows Python. Don’t describe code.

- Mark code that is less-than-desirable or needing some update with a
  comment using ``TODO(ldap): description``. This allows the code base
  to be searched by TODO and filtered by user. E.g. ``grep -rnH
  'TODO(waldo)' *``

Calling functions
-----------------

Readability
~~~~~~~~~~~

Use keyword args when calling a function with three or more args.

Bad
+++

.. code-block:: python

   foo(bar.baz(), some_function(), blah, x, y)

Good
++++

.. code-block:: python

   foo(baz=bar.baz(),
       some_result=some_function(),
       blah=blah,
       x=x,
       y=y)

Exceptions
+++++++
- While this is generally a good idea, it is not a hard and fast rule.
  For example, well named args may be readable enough.

.. code-block:: python

   move_to(x_coordinate, y_coordinate, speed_per_second)


Strings
-------

- Prefer f-strings over ``.format()`` or ``%``.
  
- Use utf-8 characters directly instead of their byte representations
  or html entity tags. ex: u'Put é instead of \xe9'

- Use your best judgement with regard to readability when putting
  together strings. Simple concatenation is ok when it is very simple.

- When concatenating a large number of strings, either add strings to
  a list and use ``''.join(...)`` or use ``io.BytesIO``. Strings are
  immutable in Python; concatenation always allocates a new string.

Resources
---------

- Explicitly clean up resources such as files, transports, connections
  and sockets. Use ``try/finally``, or use ``with`` and contextlib_ to
  simplify management.

Inversion of Control and Dependency Injection
---------------------------------------------

- Inject objects and resources versus creating them. This will
  simplify testing (injectable mocks versus patching) and increase
  flexibility (injected objects and resources need only meet an
  interface).

- You can keep your method signatures simple by injecting the
  dependencies in the constructor. This is a good idea for objects that
  are instantiated once and used many times. For example, a class
  might take a gRPC client in its constructor and use it to implement
  its methods - this lets setup code worry about creating the client,
  and business logic code can call the methods of the instantiated
  class without knowing about the client.

- It’s a one-liner to add object creation to a function that
  accepts an object as an argument; the converse requires rewriting
  the function.

Constructors
------------

- Use `dataclasses <https://docs.python.org/3/library/dataclasses.html>`_
  to create simple data objects.

- Limit the amount of “real work” done in a constructor. Dependency
  injection is a tremendous help here.

- If an object requires expensive initialization (e.g. the creation of
  a zookeeper session, communication over the network, file IO,
  concurrency) use a separate classmethod to initialize the object. Also
  consider the thread/concurrency safety of this initializer
  function. Remember that an object may be created elsewhere as a
  side-effect of module import.

Naming
------

- Use a single underscore prefix to denote protected access.

- Use a double underscore prefix to denote private access (and effect
  name mangling).

- Avoid stutters: ``foo.FooThing``, ``bar.bar_function``.

- Avoid smurf-naming - when almost everything shares some similar
  prefix.

Main
----

Every “main” should be an importable Python module. Importing that
module should never cause it to execute itself as a script. Python
files that are scripts should use the execution guard ``if __name__ ==
"__main__":``. Not only does this allow "mains" to be imported and
used elsewhere, many tools require modules to be importable
(documentation tools, test frameworks, some refactoring and analysis
tools).

BUILD Guidelines
================

BUILD files related to the pants_ build system.

PEP 8
-----

BUILD files are Python and should follow pep8 style. Use the
build-deps goal if you want to get BUILD files right without fussing
over the details.

Dependencies
------------

Depend on all direct dependencies
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Do not rely on transitive dependencies to satisfy module
  requirements. For example, we have many wrappers around SQLAlchemy,
  but any target depending on these wrappers which uses SQLAlchemy
  should also directly depend on SQLAlchemy.

- Generally any import in any file in a target should be backed by a
  dependency unless it is standard library.

Organize to minimize dependency overlap
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

- Users of a library should not be unknowingly bundling entire
  frameworks that are not used. If you find yourself depending on
  several large, unrelated dependencies that are not strictly
  necessary, you might need to split your modules and targets.

Tests
-----

- Tests should live next to the targets they test and be suffixed with
  "_test".

Sources
-------

- Avoid ``globs`` and ``rglobs``. There are exceptions (such as
  generated code and templates), but do not use globs as a shortcut to
  include files as sources.

Exceptions to the Style Guide
=============================

There are bound to be exceptions born of necessity.

Exceptions must be reviewed
---------------------------

- Any violation of best practice and style should not escape code
  review, and should be explicitly reviewed based on its necessity to
  break the rules. Style and language rules are meant to reduce
  gotchas and corner cases while increasing readability through
  consistency, but they are most effective in aggregate.

Exceptions should be isolated
-----------------------------

- E.g. a common module designed to be used as a wildcard import would
  proliferate bad style, while a case for mixins could probably be
  made if they were isolated to a specific application.

.. _unittest.TestCase: https://docs.python.org/2/library/unittest.html#unittest.TestCase

.. _Queue: https://docs.python.org/2/library/queue.html

.. _threading.Condition: https://docs.python.org/2/library/threading.html#condition-objects

.. _abc.ABCMeta: https://docs.python.org/2/library/abc.html#abc.ABCMeta

.. _wrapt: https://wrapt.readthedocs.org/en/latest/

.. _`collections abstract base classes`: https://docs.python.org/2/library/collections.html#collections-abstract-base-classes

.. _`PEP 8`: https://www.python.org/dev/peps/pep-0008/

.. _contextlib: https://docs.python.org/2/library/contextlib.html

.. _pants: https://pantsbuild.github.io/
