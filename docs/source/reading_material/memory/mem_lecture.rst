====================================================
Reading
====================================================

This reading explains where a running Python program keeps its data. The
lectures refer back to it whenever memory matters: argument passing in
Lecture 4, NumPy arrays in Lecture 5, ``__slots__`` in Lecture 7, and
ROS 2 messages and callbacks from Lecture 8 on.

Every program on this page is in ``reading/memory/memory_demo.py`` in
the `course repository
<https://github.com/rubixcubic/enpm605-spring-2027-python>`_. Every
output was printed by running it with the course environment (CPython
3.12.3 on Ubuntu 24.04, 64-bit):

.. code-block:: bash

   uv run reading/memory/memory_demo.py

Your ``id()`` values and memory addresses will differ; the other
numbers should match.


Names and Objects
=================

A running Python program holds two kinds of things:

- **Names**, such as ``speed_mps``. A name is a label you use in code.
- **Objects**, such as the float ``0.5``. An object holds a value and
  knows its type.

A name **refers** to an object. Assignment (``=``) makes a name refer to
an object; it never copies the object. Lecture 1 showed this with
``id()`` and ``is``; this reading shows where each part lives.

CPython, the Python on Ubuntu, is a C program. It keeps every Python
object in memory it manages for itself, called the **heap**: memory a
program requests while it runs, as opposed to memory laid out before it
starts.


Names Live in Namespaces
========================

A **namespace** is a table that maps names to objects. Python keeps
several, and a name's namespace depends on where the code assigns it:

- The **module namespace** (global names): names assigned at the top
  level of a file. ``globals()`` returns it as a dictionary.
- A **local namespace**: names assigned inside a function. Each call
  gets a new one, and it disappears when the call returns. ``locals()``
  returns it inside the function.
- The **built-in namespace**: names Python provides, such as ``print``
  and ``len``.

.. code-block:: python

   speed_mps = 0.5  # A global name, in the module's namespace


   def drive(distance_m):
       time_s = distance_m / speed_mps  # A local name, in this call's namespace
       print(locals())
       return time_s


   print(globals()["speed_mps"])
   print(drive(10.0))
   print("time_s" in globals())

.. code-block:: text

   0.5
   {'distance_m': 10.0, 'time_s': 20.0}
   20.0
   False

The call ``drive(10.0)`` prints its local namespace: ``distance_m``
(the argument) and ``time_s``. After the call, ``time_s`` is gone: it
was never in the module namespace, so ``"time_s" in globals()`` is
``False``.

.. figure:: /_static/images/reading_material/memory/namespaces_heap.svg
   :alt: Three regions. Tag 1, the module's namespace, holds the names speed_mps and drive. Tag 2, the namespace of the call drive(10.0), holds the names distance_m and time_s. Tag 3, the heap, holds the objects: a float 0.5, the function drive, a float 10.0 and a float 20.0. Each name has an arrow to its object.
   :width: 90%

   While ``drive(10.0)`` runs. 1: the module namespace. 2: the namespace
   of the call ``drive(10.0)``. 3: the heap, where every object lives. The
   arrows: each name refers to its object.

Even the function ``drive`` is an object on the heap, and ``drive`` is a
name that refers to it.


Objects Live on the Heap
========================

Every object has:

- an **identity**: ``id()`` returns it; in CPython it is the object's
  memory address, and it never changes while the object exists;
- a **type**: ``type()`` returns it;
- a **value**;
- a **reference count**: how many references point to it (next
  section).

Two names can refer to one object:

.. code-block:: python

   battery_pct = 87
   level = battery_pct
   print(type(battery_pct), id(battery_pct) == id(level), battery_pct is level)

.. code-block:: text

   <class 'int'> True True

Object Sizes
------------

``sys.getsizeof()`` returns an object's size in bytes. The Python
documentation is precise about what it counts: "Only the memory
consumption directly attributed to the object is accounted for, not the
memory consumption of objects it refers to."

.. code-block:: python

   print(sys.getsizeof(87), sys.getsizeof(0.5), sys.getsizeof(""), sys.getsizeof("rover"))
   print(sys.getsizeof([]), sys.getsizeof([1, 2, 3]))
   print(sys.getsizeof(["a" * 1000, "b" * 1000, "c" * 1000]))

.. code-block:: text

   28 24 41 46
   56 88
   88

Reading the numbers:

1. The int ``87`` takes 28 bytes and the float ``0.5`` takes 24: every
   object carries a header (its reference count and a pointer to its
   type) besides its value.
2. The empty string takes 41 bytes and ``"rover"`` takes 46: 41 + 5,
   one byte per character for these ASCII characters.
3. The empty list takes 56 bytes and ``[1, 2, 3]`` takes 88. A list
   stores a reference (8 bytes) per slot, and 88 − 56 = 32 bytes is
   32 / 8 = 4 slots: the list keeps one slot spare.
4. A list of three strings of 1,000 characters each also takes 88 bytes:
   ``getsizeof`` counts the list's references, not the strings.


References and Reference Counts
===============================

A **reference** is how a name, a list slot, or any other object points to
an object. CPython counts the references to each object: its
**reference count**.

.. code-block:: python

   waypoints = [1, 2]
   print(sys.getrefcount(waypoints))
   route = waypoints
   print(sys.getrefcount(waypoints))
   del route
   print(sys.getrefcount(waypoints))

.. code-block:: text

   2
   3
   2

The first count is 2, not 1: the name ``waypoints`` is one reference,
and passing the list to ``getrefcount`` adds a temporary second one (the
Python documentation: "The count returned is generally one higher than
you might expect, because it includes the (temporary) reference as an
argument to getrefcount()"). ``route = waypoints`` adds a reference (3);
``del route`` removes it (2). ``del`` removes a **name**, not an object.

Immortal Objects
----------------

.. code-block:: python

   print(sys.getrefcount(None), sys.getrefcount(True), sys.getrefcount(1))

.. code-block:: text

   4294967295 4294967295 4294967295

``None``, ``True``, and small integers such as ``1`` are **immortal** in
CPython 3.12 (`PEP 683 <https://peps.python.org/pep-0683/>`_): they are
never freed, and their reference count stays at a fixed large value,
4294967295 (2\ :sup:`32` − 1), however many references exist. The
documentation of ``sys.getrefcount`` says these "very large refcounts
... do not match the actual number of references to the object".


Call Frames
===========

Each function call creates a **frame**: a record of the running call,
which holds its local namespace and the place where it will return. A
frame exists while its call runs and is discarded when the call returns.
The frames of the calls in progress form a stack, newest on top: each
frame points back to the frame of its caller (``f_back``).

.. code-block:: python

   def g():
       frame = sys._getframe()
       return [
           frame.f_code.co_name,
           frame.f_back.f_code.co_name,
           frame.f_back.f_back.f_code.co_name,
       ]


   def f():
       return g()


   print(f())
   print(sys.getrecursionlimit())


   def down(n):
       return down(n + 1)


   try:
       down(0)
   except RecursionError as error:
       print(error)

.. code-block:: text

   ['g', 'f', '<module>']
   1000
   maximum recursion depth exceeded

Inside ``g``, the frame of ``g`` points back to the frame of ``f``,
which points back to the module's frame (``<module>``). Every call adds
a frame, so a function that calls itself forever runs out of room: CPython
stops at its recursion limit, 1000 frames by default, and raises
``RecursionError``.

.. figure:: /_static/images/reading_material/memory/call_frames.svg
   :alt: Three frames stacked, newest on top. Tag 1, the frame of g, is running. Tag 2, the frame of f, waits for g to return. Tag 3 is the module's frame. A curved arrow goes from each frame to the frame below it: the f_back link.
   :width: 50%

   While ``g()`` runs. 1: the frame of ``g``, running. 2: the frame of
   ``f``, waiting for ``g`` to return. 3: the module's frame. The arrows
   are the ``f_back`` links.

.. note::

   "Python frames, holding execution information, are created whenever
   Python calls a Python function." Python 3.11 made creating them
   cheaper (`What's New in Python 3.11
   <https://docs.python.org/3.11/whatsnew/3.11.html>`_).


Containers Hold References
==========================

A list does not hold objects; it holds **references** to objects. Two
slots can refer to the same object:

.. code-block:: python

   scan = [0.5, 0.7]
   history = [scan, scan]
   scan.append(0.9)
   print(history)

   grid = [[0, 0], [0, 0]]
   shallow = copy.copy(grid)
   deep = copy.deepcopy(grid)
   grid[0][0] = 1
   print(shallow, deep)
   print(shallow is grid, shallow[0] is grid[0], deep[0] is grid[0])

.. code-block:: text

   [[0.5, 0.7, 0.9], [0.5, 0.7, 0.9]]
   [[1, 0], [0, 0]] [[0, 0], [0, 0]]
   False True False

``history`` has two slots, and both refer to the one list ``scan``, so
appending to ``scan`` shows in both.

.. figure:: /_static/images/reading_material/memory/container_refs.svg
   :alt: The name history has an arrow to a list object with two slots. Each slot holds a reference: both arrows go to the same list object [0.5, 0.7, 0.9], which the name scan also refers to.
   :width: 75%

   Both slots of ``history`` refer to the one list that ``scan`` refers
   to.

Shallow and Deep Copies
-----------------------

``copy.copy()`` makes a **shallow copy**: a new outer list whose slots
refer to the same inner objects. ``copy.deepcopy()`` makes a **deep
copy**: new inner objects too. The second part of the code above shows
the difference after ``grid[0][0] = 1``: the change shows through
``shallow`` but not through ``deep``. The last line confirms it:
``shallow`` is a different outer list (``False``) that shares
``grid``'s first inner list (``True``); ``deep`` does not
(``False``).

.. figure:: /_static/images/reading_material/memory/copies.svg
   :alt: Two panels, after grid[0][0] = 1. Tag 1: grid and shallow are two different outer lists, but the slots of both refer to the same two inner lists, [1, 0] and [0, 0], so the change shows through shallow. Tag 2: deep is an outer list whose slots refer to its own two inner lists, both still [0, 0].
   :width: 90%

   After ``grid[0][0] = 1``. 1: ``grid`` and ``shallow`` share the inner
   lists. 2: ``deep`` has its own.


When Objects Are Freed
======================

Reference Counting
------------------

When an object's reference count drops to zero, nothing can reach it any
more, and CPython frees it **at once**. ``weakref.finalize`` runs a
function when an object is freed, which makes the moment visible:

.. code-block:: python

   class Robot:
       pass


   rover = Robot()
   weakref.finalize(rover, print, "rover freed")
   backup = rover
   del rover
   print("after del rover")
   del backup
   print("after del backup")

   gc.collect()
   a = Robot()
   b = Robot()
   a.partner = b
   b.partner = a
   weakref.finalize(a, print, "a freed")
   del a, b
   print("after del a, b")
   gc.collect()
   print("after gc.collect()")

.. code-block:: text

   after del rover
   rover freed
   after del backup
   after del a, b
   a freed
   after gc.collect()

After ``del rover``, the name ``backup`` still refers to the object, so
it survives. ``del backup`` removes the last reference, and
``rover freed`` prints before the next line runs.

.. figure:: /_static/images/reading_material/memory/lifetime.svg
   :alt: Three panels. Tag 1: the names rover and backup refer to one Robot object; its reference count is 2. Tag 2, after del rover: only backup refers to it; the count is 1 and the object is alive. Tag 3, after del backup: no name refers to it, the count is 0, and the object, drawn faded, is freed.
   :width: 90%

   1: two names refer to the object (count 2). 2: after ``del rover``
   (count 1). 3: after ``del backup`` (count 0): freed.

The Cycle Collector
-------------------

Reference counting cannot free a **cycle**: objects that refer to each
other. In the second part of the code, ``a`` and ``b`` refer to each
other through ``partner``; after ``del a, b`` each still has a
reference from the other, so neither count reaches zero, and ``a freed``
does not print. CPython's **cycle collector** (the ``gc`` module) finds
groups of objects that only refer to each other and frees them. It runs
automatically from time to time; ``gc.collect()`` runs it now, and
``a freed`` prints.


CPython Details You Will Meet
=============================

Small Integers and Interned Strings
-----------------------------------

.. code-block:: python

   x = 256
   y = int("256")
   print(x is y)
   x = 257
   y = int("257")
   print(x is y, x == y)
   name = "rover"
   parts = ["ro", "ver"]
   built = "".join(parts)  # Built while the program runs
   print(name is built, name == built, name is sys.intern(built))

.. code-block:: text

   True
   False True
   False True True

- **Small integers.** The Python documentation: "The current
  implementation keeps an array of integer objects for all integers
  between -5 and 256. When you create an int in that range you actually
  just get back a reference to the existing object." So ``256`` built
  two ways is one object (``True``), and ``257`` built two ways is two
  objects (``False``) with equal values (``True``).
- **Interned strings.** CPython keeps one copy of some strings, such as
  ``"rover"`` written in the code. A string built while the program runs
  (``built``) is a separate object (``False``) with an equal value
  (``True``). ``sys.intern()`` returns the kept copy (``True``).

.. warning::

   These are CPython optimizations, not rules of the language, and they
   can change between versions. The same two lines can even give
   different answers in a script and in the REPL:

   .. code-block:: python

      first = 257
      second = 257
      print(first is second)  # One code object: the two 257s are one constant
      repl = code.InteractiveInterpreter()
      for typed in ["first = 257", "second = 257", "print(first is second)"]:
          repl.runsource(typed)  # Each line compiled on its own, as in the REPL

   .. code-block:: text

      True
      False

   In the file, both ``257`` literals belong to one compiled unit, which
   keeps one constant object, so ``is`` prints ``True``. Typed line by
   line, each line is compiled on its own and creates its own ``257``, so
   ``is`` prints ``False``. Compare values with ``==``. Use ``is`` only to
   check for ``None``.


Measuring Memory
================

``sys.getsizeof`` counts one object. ``tracemalloc`` traces every
allocation the program makes, which shows the full cost of a data
structure. Here, 1,000,000 numbers stored as a list and as a NumPy
array (Lecture 5):

.. code-block:: python

   tracemalloc.start()
   readings = [float(i) for i in range(1_000_000)]
   current, peak = tracemalloc.get_traced_memory()
   tracemalloc.stop()
   print(sys.getsizeof(readings), current)
   array = np.arange(1_000_000, dtype=np.float64)
   print(array.nbytes, array.itemsize)

.. code-block:: text

   8448728 32448672
   8000000 8

Reading the numbers:

1. The list itself takes 8,448,728 bytes: its slots of 8-byte
   references (with spare room, as before).
2. Each float is a separate object of 24 bytes, so the floats take
   1,000,000 × 24 = 24,000,000 bytes.
3. ``tracemalloc`` counted 32,448,728 bytes in all:
   8,448,728 + 24,000,000 = 32,448,728.
4. The NumPy array stores the numbers themselves, 8 bytes each
   (``itemsize``): 1,000,000 × 8 = 8,000,000 bytes, about a quarter of
   the list's total.

.. figure:: /_static/images/reading_material/memory/list_vs_array.svg
   :alt: Two parts. Tag 1: a Python list is a block of slots; each slot holds a reference, drawn as an arrow, to a separate float object (0.0, 1.0, 2.0) somewhere else on the heap. Tag 2: a NumPy array of float64 is one block in which the numbers 0.0, 1.0, 2.0, 3.0 sit side by side, with no separate objects.
   :width: 80%

   1: a list holds references to separate float objects. 2: a NumPy
   array holds the numbers side by side.

A laser scan with 1,000 ranges per message, at 10 messages a second,
is the kind of data where this difference matters.


Where This Comes Back
=====================

- **Lecture 4 (Functions):** passing an argument binds a parameter name
  to the same object, so a function can change a mutable argument; a
  mutable default value is created once and shared by every call.
- **Lecture 5 (NumPy):** arrays store numbers side by side, as in the
  last section.
- **Lecture 7 (OOP, Part II):** ``__slots__`` changes how an object stores
  its attributes.
- **Lectures 8 to 14 (ROS 2):** messages are objects; a callback keeps
  references to the objects it uses, so they stay alive as long as the
  callback exists.

