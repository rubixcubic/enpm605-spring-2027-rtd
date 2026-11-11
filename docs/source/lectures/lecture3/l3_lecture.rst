====================================================
Lecture
====================================================

Every program on this page is in the ``lecture3/`` folder of the
`course repository <https://github.com/rubixcubic/enpm605-spring-2027-python>`_.
Run each one from the root of the repository, for example:

.. code-block:: bash

   uv run lecture3/loops_demo.py

Every output on this page was printed by running the program with the
course environment (CPython 3.12.3 on Ubuntu 24.04). In paths, ``...``
marks a path shortened to fit. The values in the examples (ranges,
battery levels, waypoints) are chosen for the examples.


Loops
====================================================

for Runs a Block Once for Each Item
-----------------------------------

.. code: lecture3/loops_demo.py 5-9

.. code-block:: python
   :caption: From lecture3/loops_demo.py

   # for over a list
   ranges_m = [2.5, 0.9, 3.1]  # Chosen for this example
   for r in ranges_m:
       print(f"Range: {r} m")
   print(f"After the loop, r is {r}")

.. output: loops_demo.txt 1-4

.. code-block:: text

   Range: 2.5 m
   Range: 0.9 m
   Range: 3.1 m
   After the loop, r is 3.1

- ``for name in iterable:`` binds ``name`` to the first item, runs the
  block, then binds it to the next item, until no item is left.
- :doc:`Lecture 2 <../lecture2/l2_lecture>` printed ``sys.path`` this
  way: ``for folder in sys.path:``.
- After the loop, ``r`` still refers to the last item, ``3.1``.
- A ``for`` loop also walks over the characters of a string, the keys of
  a dictionary, and more (see `Iterables`_).

range() Gives a Sequence of Integers
------------------------------------

.. code: lecture3/loops_demo.py 11-18

.. code-block:: python

   # range
   print(list(range(5)))
   print(list(range(2, 7)))
   print(list(range(0, 10, 2)))
   print(list(range(5, 0, -1)))
   print(range(5))
   for _ in range(2):
       print("Beep")

.. output: loops_demo.txt 5-11

.. code-block:: text

   [0, 1, 2, 3, 4]
   [2, 3, 4, 5, 6]
   [0, 2, 4, 6, 8]
   [5, 4, 3, 2, 1]
   range(0, 5)
   Beep
   Beep

.. list-table::
   :header-rows: 1
   :widths: 35 65
   :class: compact-table

   * - Call
     - Integers
   * - ``range(stop)``
     - ``0`` up to, not including, ``stop``
   * - ``range(start, stop)``
     - ``start`` up to, not including, ``stop``
   * - ``range(start, stop, step)``
     - The same, ``step`` apart; a negative ``step`` counts down

- ``stop`` is never included, as in a slice (Lecture 2). ``list()``
  shows the integers; ``print(range(5))`` shows only ``range(0, 5)``.
- ``for _ in range(2):`` runs a block twice. The name ``_`` says that the
  number is not used.

range Stores Start, Stop, and Step, Not the Numbers
---------------------------------------------------

.. code: lecture3/loops_demo.py 3-4,20-23

.. code-block:: python

   import sys

   # range stores start, stop, and step
   numbers = range(1_000_000)
   print(len(numbers), numbers[10], 999_999 in numbers)
   print(sys.getsizeof(numbers), sys.getsizeof(list(numbers)))

.. output: loops_demo.txt 12-13

.. code-block:: text

   1000000 10 True
   48 8000056

- ``1_000_000`` is one million. Python ignores the underscores; they make
  long numbers easier to read.
- A ``range`` computes an item when you ask for it: its length,
  ``numbers[10]``, and ``in`` need no list.
- ``sys.getsizeof()`` gives the size of one object in bytes: 48 for the
  range, 8,000,056 for the list of the same million numbers.
- The documentation: a range "will always take the same (small) amount
  of memory, no matter the size of the range it represents".

.. tip::

   Loop over ``range()`` directly. Build a list with
   ``list(range(...))`` only when you need to keep the numbers.

enumerate() Gives the Index and the Item
----------------------------------------

.. code: lecture3/loops_demo.py 25-32

.. code-block:: python

   # enumerate
   sensors = ["lidar", "camera", "imu"]
   for i in range(len(sensors)):
       print(i, sensors[i])
   for i, name in enumerate(sensors):
       print(i, name)
   for i, name in enumerate(sensors, start=1):
       print(f"{i}. {name}")

.. output: loops_demo.txt 14-22

.. code-block:: text

   0 lidar
   1 camera
   2 imu
   0 lidar
   1 camera
   2 imu
   1. lidar
   2. camera
   3. imu

- The first two loops print the same lines. ``range(len(sensors))``
  gives indices only, so the loop reads ``sensors[i]`` by hand.
- ``enumerate()`` gives pairs: the index and the item. ``i, name`` takes
  each pair apart (unpacking, Lecture 1; see `Tuples`_).
- ``start=1`` numbers from 1, for a list printed for people.

.. tip::

   When you need the index and the item, use ``enumerate()``, not
   ``range(len(...))``.

while Repeats a Block While a Condition Is True
-----------------------------------------------

.. code: lecture3/loops_demo.py 34-40

.. code-block:: python

   # while
   battery_pct = 100  # Chosen for this example
   tasks = 0
   while battery_pct >= 20:
       battery_pct -= 30  # Each task: 30%, chosen for this example
       tasks += 1
   print(f"{tasks} tasks, battery at {battery_pct}%")

.. output: loops_demo.txt 23

.. code-block:: text

   3 tasks, battery at 10%

``while condition:`` tests the condition before each pass, and runs the
block when it is truthy. Use ``while`` when you do not know in advance
how many passes the loop needs. The battery, pass by pass:

1. ``100 >= 20``: one task. 100 minus 30 is 70.
2. ``70 >= 20``: a second task. 70 minus 30 is 40.
3. ``40 >= 20``: a third task. 40 minus 30 is 10.
4. ``10 >= 20`` is ``False``, so the loop ends after 3 tasks.

A while Loop Whose Condition Never Changes Runs Forever
-------------------------------------------------------

.. code: lecture3/errors/infinite_loop.py all

.. code-block:: python
   :caption: lecture3/errors/infinite_loop.py

   battery_pct = 100
   tasks = 0
   while battery_pct >= 20:
       tasks += 1  # The line battery_pct -= 30 is missing

``battery_pct`` stays at 100, so ``100 >= 20`` is always true. The
program prints nothing and never ends. Press Ctrl+C in the terminal.
Python stops at the line it is running and raises
``KeyboardInterrupt``:

.. output: err_infinite_loop.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../infinite_loop.py", line 3, in <module>
       while battery_pct >= 20:
             ^^^^^^^^^^^^^^^^^
   KeyboardInterrupt

This output was recorded with ``timeout -s INT 1``, which sends the same
signal as Ctrl+C after 1 second. Every ``while`` block must change
something its condition reads.

break Leaves the Loop at Once
-----------------------------

.. code: lecture3/loops_demo.py 42-49

.. code-block:: python

   # break
   ranges_m = [2.5, 0.9, 0.3, 3.1]  # Chosen for this example
   SAFE_M = 0.5  # Chosen for this example
   for i, r in enumerate(ranges_m):
       if r < SAFE_M:
           print(f"Obstacle at index {i}: {r} m")
           break
       print(f"Index {i}: clear")

.. output: loops_demo.txt 24-26

.. code-block:: text

   Index 0: clear
   Index 1: clear
   Obstacle at index 2: 0.3 m

- The loop looks for the first reading closer than 0.5 m.
- Index 0 (2.5 m) and index 1 (0.9 m) are not below 0.5 m, so the loop
  prints ``clear`` for each.
- Index 2 (0.3 m) is below 0.5 m: the loop prints the obstacle, and
  ``break`` ends the loop. Index 3 (3.1 m) is never read.
- ``break`` leaves the innermost loop it is in.

continue Skips to the Next Item
-------------------------------

.. code: lecture3/loops_demo.py 51-61

.. code-block:: python

   # continue
   RANGE_MIN_M, RANGE_MAX_M = 0.12, 12.0  # The lidar of Lecture 2
   scan = [2.5, 0.0, 3.1, float("inf"), 1.2]
   total = 0.0
   count = 0
   for r in scan:
       if not (RANGE_MIN_M <= r <= RANGE_MAX_M):
           continue
       total += r
       count += 1
   print(f"{count} valid readings, mean {total / count:.2f} m")

.. output: loops_demo.txt 27

.. code-block:: text

   3 valid readings, mean 2.27 m

- ROS 2's ``LaserScan`` message, on its ranges: "values < range_min or >
  range_max should be discarded". ROS's
  `REP 117 <https://www.ros.org/reps/rep-0117.html>`_: "out of range
  detections will be represented by +Inf."
- ``continue`` skips the rest of the block for this item, and the loop
  goes on with the next one.
- ``0.0`` and ``float("inf")`` are outside 0.12 to 12.0 m, so they are
  skipped. The mean of the other three:

  1. 2.5 + 3.1 + 1.2 = 6.8.
  2. 6.8 / 3 = 2.266..., printed with 2 decimals: ``2.27``.

else on a Loop Runs When No break Happened
------------------------------------------

No reading below 0.5 m:

.. code: lecture3/loops_demo.py 64-69

.. code-block:: python

   for r in [2.5, 0.9, 3.1]:
       if r < SAFE_M:
           print(f"Stop: {r} m")
           break
   else:
       print("Path clear")

.. output: loops_demo.txt 28

.. code-block:: text

   Path clear

One reading below 0.5 m:

.. code: lecture3/loops_demo.py 72-77

.. code-block:: python

   for r in [2.5, 0.3, 3.1]:
       if r < SAFE_M:
           print(f"Stop: {r} m")
           break
   else:
       print("Path clear")

.. output: loops_demo.txt 29

.. code-block:: text

   Stop: 0.3 m

- The tutorial: "If the loop finishes without executing the break, the
  else clause executes."
- In the first loop, no reading is below ``SAFE_M`` (0.5 m), so the loop
  ends normally and ``else`` prints ``Path clear``.
- In the second, 0.3 m triggers ``break``, so the ``else`` block is
  skipped.
- The ``else`` lines up with ``for``, not with ``if``: it belongs to the
  loop.


Iterables
====================================================

An Iterable Gives Its Items One at a Time
-----------------------------------------

.. code: lecture3/iterables_demo.py 3-6

.. code-block:: python
   :caption: From lecture3/iterables_demo.py

   # Iterables
   print(list("imu"))
   print(list(range(3)))
   print(list({"x": 1.0, "y": 2.0}))

.. output: iterables_demo.txt 1-3

.. code-block:: text

   ['i', 'm', 'u']
   [0, 1, 2]
   ['x', 'y']

.. code: lecture3/errors/not_iterable.py all

.. code-block:: python
   :caption: lecture3/errors/not_iterable.py

   count = 3
   for i in count:
       print(i)

.. output: err_not_iterable.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../not_iterable.py", line 2, in <module>
       for i in count:
   TypeError: 'int' object is not iterable

- The glossary: an **iterable** is "An object capable of returning its
  members one at a time."
- ``for`` and ``list()`` accept any iterable: a string gives characters,
  a ``range`` gives integers, a dictionary gives its keys.
- An ``int`` is one value, not a collection, so a ``for`` loop over it
  raises a ``TypeError``. Loop over ``range(count)`` instead.

iter() Makes an Iterator, next() Reads the Next Item
----------------------------------------------------

.. code: lecture3/iterables_demo.py 8-14

.. code-block:: python

   # iter() and next()
   ranges_m = [2.5, 0.9, 3.1]  # Chosen for this example
   it = iter(ranges_m)
   print(type(it))
   print(next(it))
   print(next(it))
   print(next(it))

.. output: iterables_demo.txt 4-7

.. code-block:: text

   <class 'list_iterator'>
   2.5
   0.9
   3.1

- ``iter(ranges_m)`` returns an **iterator**: an object that gives the
  items of the list one at a time. Its type here is ``list_iterator``.
- Each ``next(it)`` returns the next item and moves the iterator one
  place on.
- The iterator does not copy the list. It refers to the list and
  remembers where it is:

.. figure:: /_static/images/l3/iterator.svg
   :alt: The name ranges_m has an arrow to a list object, tag 1, with three slots holding 2.5, 0.9 and 3.1 at indices 0, 1 and 2. The name it has an arrow to a list_iterator object, tag 2, which holds the text next index: 2 and has its own arrow up to the same list. The iterator does not hold a copy of the items.
   :width: 75%

   After ``it = iter(ranges_m)`` and two calls of ``next(it)``. 1: the
   list. 2: the iterator: it refers to the list and keeps the index of
   the next item.

next() Raises StopIteration When No Item Is Left
------------------------------------------------

.. code: lecture3/errors/stop_iteration.py all

.. code-block:: python
   :caption: lecture3/errors/stop_iteration.py

   ranges_m = [2.5, 0.9, 3.1]
   it = iter(ranges_m)
   print(next(it), next(it), next(it))
   print(next(it))

.. output: err_stop_iteration.txt all short

.. code-block:: text

   2.5 0.9 3.1
   Traceback (most recent call last):
     File ".../stop_iteration.py", line 4, in <module>
       print(next(it))
             ^^^^^^^^
   StopIteration

The first three calls of ``next()`` return the three items. The fourth
has nothing to return. The glossary, on an iterator: "When no more data
are available a StopIteration exception is raised instead."

``for r in ranges_m:`` does these steps for you:

1. Call ``iter(ranges_m)`` once, to get an iterator.
2. Before each pass, call ``next()`` on the iterator, and bind the item
   it returns to ``r``.
3. When ``next()`` raises ``StopIteration``, end the loop. No error
   reaches you.

An Iterator Is Used Up After One Pass
-------------------------------------

.. code: lecture3/iterables_demo.py 16-20

.. code-block:: python

   # One pass uses up an iterator
   it = iter(ranges_m)
   print(list(it))
   print(list(it))
   print(list(ranges_m), list(ranges_m))

.. output: iterables_demo.txt 8-10

.. code-block:: text

   [2.5, 0.9, 3.1]
   []
   [2.5, 0.9, 3.1] [2.5, 0.9, 3.1]

- The first ``list(it)`` reads every item, so the iterator's next index
  is past the end. The second ``list(it)`` gets nothing: ``[]``.
- A list is an iterable, not an iterator. The glossary: "A container
  object (such as a list) produces a fresh new iterator each time you
  pass it to the iter() function or use it in a for loop." So
  ``list(ranges_m)`` works twice.
- ``zip()``, below, returns an iterator: you can loop over its result
  only once.

zip() Walks Several Iterables Together
--------------------------------------

.. code: lecture3/iterables_demo.py 22-26

.. code-block:: python

   # zip
   angles_deg = [-30, 0, 30]  # Chosen for this example
   for angle, r in zip(angles_deg, ranges_m):
       print(f"{angle:>4} deg: {r} m")
   print(list(zip(angles_deg, ranges_m)))

.. output: iterables_demo.txt 11-14

.. code-block:: text

    -30 deg: 2.5 m
      0 deg: 0.9 m
     30 deg: 3.1 m
   [(-30, 2.5), (0, 0.9), (30, 3.1)]

- A lidar scan stores a list of ranges; the angle of each range comes
  from its position. Two lists of the same length, read side by side,
  pair each angle with its range.
- ``zip(a, b)`` returns an iterator of pairs: item 0 of ``a`` with item
  0 of ``b``, then item 1 with item 1, and so on.
- Each pair is a tuple, written ``(-30, 2.5)``. ``angle, r`` unpacks it,
  as with ``enumerate()``.
- ``:>4`` right-aligns the angle in 4 characters (Lecture 2, f-strings).

zip() Stops at the Shortest Input Unless strict=True
----------------------------------------------------

.. code: lecture3/iterables_demo.py 28-29

.. code-block:: python

   # zip stops at the shortest input
   print(list(zip([-30, 0, 30], [2.5, 0.9])))

.. output: iterables_demo.txt 15

.. code-block:: text

   [(-30, 2.5), (0, 0.9)]

The documentation: "By default, zip() stops when the shortest iterable
is exhausted." The angle ``30`` has no range to pair with, so it is
dropped, with no error and no warning. When two lists should have the
same length, this silent cut hides the bug that made them differ.

.. code: lecture3/errors/zip_strict.py all

.. code-block:: python
   :caption: lecture3/errors/zip_strict.py

   angles_deg = [-30, 0, 30]
   ranges_m = [2.5, 0.9]
   for angle, r in zip(angles_deg, ranges_m, strict=True):
       print(angle, r)

.. output: err_zip_strict.txt all short

.. code-block:: text

   -30 2.5
   0 0.9
   Traceback (most recent call last):
     File ".../zip_strict.py", line 3, in <module>
       for angle, r in zip(angles_deg, ranges_m, strict=True):
   ValueError: zip() argument 2 is shorter than argument 1

- ``strict=True`` (added in Python 3.10) raises a ``ValueError`` when one
  input runs out before the others.
- The error comes when the short list runs out, so the first two pairs
  were already printed.
- The documentation: "zip() is often used in cases where the iterables
  are assumed to be of equal length. In such cases, it's recommended to
  use the strict=True option."


Lists
====================================================

A List Is an Ordered, Mutable Sequence
--------------------------------------

.. code: lecture3/lists_demo.py 5-11

.. code-block:: python
   :caption: From lecture3/lists_demo.py

   # Creating lists
   ranges_m = [2.5, 0.9, 3.1]  # Chosen for this example
   mixed = ["lidar", 12.0, True, None]
   empty = []
   print(len(ranges_m), len(mixed), len(empty))
   print(list("imu"), list(range(4)))
   print([0.0] * 4, [1, 2] + [3])

.. output: lists_demo.txt 1-3

.. code-block:: text

   3 4 0
   ['i', 'm', 'u'] [0, 1, 2, 3]
   [0.0, 0.0, 0.0, 0.0] [1, 2, 3]

- A ``list`` holds items in order, between square brackets. Its items
  can have any type, but a list usually holds many items of one kind,
  such as the ranges of a scan.
- ``list()`` builds a list from any iterable: the characters of a
  string, the integers of a range.
- ``*`` repeats a list and ``+`` joins two lists into a new one, as for
  strings (Lecture 2).
- :doc:`Lecture 1 <../lecture1/l1_lecture>`: a list is **mutable**. It
  can change after it is created, and every name that refers to it sees
  the change.

Indexing and Slicing Work as for Strings, and Items Can Change
--------------------------------------------------------------

.. code: lecture3/lists_demo.py 13-20

.. code-block:: python

   # Indexing, slicing, and changing items
   ranges_m = [2.5, 0.9, 3.1, 0.3, 4.2]
   print(ranges_m[0], ranges_m[-1])
   print(ranges_m[1:3], ranges_m[::2])
   ranges_m[0] = 2.6
   print(ranges_m)
   ranges_m[1:3] = [1.0]
   print(ranges_m)

.. output: lists_demo.txt 4-7

.. code-block:: text

   2.5 4.2
   [0.9, 3.1] [2.5, 3.1, 4.2]
   [2.6, 0.9, 3.1, 0.3, 4.2]
   [2.6, 1.0, 0.3, 4.2]

- Indices and slices follow the rules of Lecture 2: index 0 is the first
  item, ``-1`` the last, and a slice stops before ``stop``.
- In Lecture 2, ``sensor[0] = "L"`` raised a ``TypeError``, because a
  string is immutable. A list accepts it: ``ranges_m[0] = 2.6`` replaces
  item 0.
- A slice can be replaced too, even by a list of another length:
  ``ranges_m[1:3] = [1.0]`` replaces two items (0.9 and 3.1) by one, so
  the list gets shorter.

Adding Items: append(), extend(), and insert()
----------------------------------------------

.. code: lecture3/lists_demo.py 22-29

.. code-block:: python

   # Adding items
   scan = [2.5]
   scan.append(0.9)
   scan.extend([3.1, 0.3])
   scan.insert(0, 1.7)
   print(scan)
   scan.append([4.2, 5.0])
   print(scan, len(scan))

.. output: lists_demo.txt 8-9

.. code-block:: text

   [1.7, 2.5, 0.9, 3.1, 0.3]
   [1.7, 2.5, 0.9, 3.1, 0.3, [4.2, 5.0]] 6

- ``append(x)`` adds one item at the end. ``extend(iterable)`` adds every
  item of the iterable at the end.
- ``insert(i, x)`` puts ``x`` at index ``i`` and moves the items after it
  one place to the right.
- ``append([4.2, 5.0])`` adds **one** item, a list. The list now has 6
  items, and the last one is ``[4.2, 5.0]``. To add two numbers, use
  ``extend()``.

Removing Items: pop(), remove(), del, and clear()
-------------------------------------------------

.. code: lecture3/lists_demo.py 31-41

.. code-block:: python

   # Removing items
   waypoints = ["dock", "A", "B", "A", "C"]
   last = waypoints.pop()
   first = waypoints.pop(0)
   print(last, first, waypoints)
   waypoints.remove("A")
   print(waypoints)
   del waypoints[0]
   print(waypoints)
   waypoints.clear()
   print(waypoints)

.. output: lists_demo.txt 10-13

.. code-block:: text

   C dock ['A', 'B', 'A']
   ['B', 'A']
   ['A']
   []

- ``pop()`` removes the last item and returns it. ``pop(0)`` removes and
  returns item 0.
- ``remove(x)`` removes the **first** item equal to ``x``: here the first
  ``"A"``. The second ``"A"`` stays.
- ``del waypoints[0]`` removes by index and returns nothing. ``clear()``
  removes every item.

Searching a List: in, index(), and count()
------------------------------------------

.. code: lecture3/lists_demo.py 43-46

.. code-block:: python

   # Searching
   waypoints = ["dock", "A", "B", "A"]
   print("B" in waypoints, "Z" in waypoints)
   print(waypoints.index("A"), waypoints.count("A"))

.. output: lists_demo.txt 14-15

.. code-block:: text

   True False
   1 2

- ``x in waypoints`` is ``True`` when an item is equal to ``x``
  (Lecture 2, membership operators).
- ``index(x)`` gives the index of the first item equal to ``x``: ``"A"``
  is first at index 1. ``count(x)`` counts the items equal to ``x``: two
  ``"A"``.
- ``in`` and ``index()`` compare ``x`` with the items one by one, from
  the start. `Sets`_ shows a faster way to test membership.

``index()`` and ``remove()`` raise a ``ValueError`` when no item is equal
to ``x``. Test with ``in`` first when the item may be missing. The last
line of each traceback:

.. code: lecture3/errors/index_missing.py all

.. code-block:: python
   :caption: lecture3/errors/index_missing.py

   waypoints = ["dock", "A", "B"]
   print(waypoints.index("Z"))

.. output: err_index_missing.txt 5

.. code-block:: text

   ValueError: 'Z' is not in list

.. code: lecture3/errors/remove_missing.py all

.. code-block:: python
   :caption: lecture3/errors/remove_missing.py

   waypoints = ["dock", "A", "B"]
   waypoints.remove("Z")

.. output: err_remove_missing.txt 4

.. code-block:: text

   ValueError: list.remove(x): x not in list

Methods That Change a List in Place Return None
-----------------------------------------------

.. code: lecture3/lists_demo.py 48-54

.. code-block:: python

   # In-place methods return None
   ranges_m = [2.5, 0.9, 3.1]
   result = ranges_m.sort()
   print(result, ranges_m)
   ranges_m = [2.5, 0.9, 3.1]
   print(sorted(ranges_m), ranges_m)
   print(sorted(ranges_m, reverse=True))

.. output: lists_demo.txt 16-18

.. code-block:: text

   None [0.9, 2.5, 3.1]
   [0.9, 2.5, 3.1] [2.5, 0.9, 3.1]
   [3.1, 2.5, 0.9]

- ``ranges_m.sort()`` sorts the list itself and returns ``None``. The
  documentation: "To remind users that it operates by side effect, it
  does not return the sorted sequence".
- ``sorted()`` takes any iterable, returns a **new** sorted list, and
  leaves the original unchanged. ``reverse=True`` sorts from largest to
  smallest.

.. warning::

   ``ranges_m = ranges_m.sort()`` sorts the list, then binds
   ``ranges_m`` to ``None``. The sorted list is lost.

In Place or a New Object
^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table::
   :header-rows: 1
   :widths: 50 50
   :class: compact-table

   * - Changes the list in place
     - Returns a new object, list unchanged
   * - ``append()``, ``extend()``, ``insert()``
     - ``a + b``, ``a * n``
   * - ``remove()``, ``pop()``, ``clear()``
     - A slice: ``a[1:3]``, ``a[:]``
   * - ``sort()``, ``reverse()``
     - ``sorted(a)``, ``reversed(a)``
   * - ``del a[i]``, ``a[i] = x``, ``a += b``
     - ``a.copy()``, ``list(a)``

- Lecture 2's string methods returned new strings, because a string
  cannot change. Most list methods change the list instead.
- The methods on the left return ``None``, except ``pop()``, which also
  returns the item it removed. ``del``, item assignment, and ``+=`` are
  statements: they return nothing.
- ``reversed(a)`` returns an iterator over the items in reverse order,
  not a list.
- ``a += b`` changes the list ``a`` in place, while ``a = a + b`` builds a
  new list and rebinds ``a``. Another name that refers to the old list
  sees the first change but not the second.

.. tip::

   Before you use the result of a list method, check whether it returns
   a value or ``None``.

Removing Items While Looping Skips Items
----------------------------------------

.. code: lecture3/lists_demo.py 56-62

.. code-block:: python

   # Removing items while looping skips items
   RANGE_MIN_M = 0.12  # The lidar of Lecture 2
   scan = [0.0, 0.0, 2.5, 0.0, 3.1]
   for r in scan:
       if r < RANGE_MIN_M:
           scan.remove(r)
   print(scan)

.. output: lists_demo.txt 19

.. code-block:: text

   [2.5, 0.0, 3.1]

The intent: drop the readings below 0.12 m, here the three ``0.0``. One
``0.0`` is left, and Python gives no error. The loop's iterator keeps an
index and adds 1 after each item (see `Iterables`_), while ``remove()``
moves every later item one place to the left:

.. figure:: /_static/images/l3/remove_while_looping.svg
   :alt: Four rows show the list scan each time the for loop reads an item; the index read is drawn as a red cell and the column numbers 0 to 4 are at the top. Tag 1: 0.0, 0.0, 2.5, 0.0, 3.1 with index 0 (0.0) in red. Tag 2: 0.0, 2.5, 0.0, 3.1 with index 1 (2.5) in red. Tag 3: the same four values with index 2 (0.0) in red. Tag 4: 2.5, 0.0, 3.1 with an empty dashed red cell at index 3, past the end of the list.
   :width: 55%

   Red: the index the loop reads.

1. Index 0 is ``0.0``: removed. The other items move one place left.
2. Index 1 is ``2.5``: kept. The second ``0.0`` moved to index 0 and is
   never read.
3. Index 2 is ``0.0``. ``remove(0.0)`` deletes the **first** ``0.0``, at
   index 0.
4. Index 3 is past the end of the 3 items left, so the loop stops.

.. warning::

   Do not add or remove items of a list while a ``for`` loop walks over
   it. Build a new list (next section) or loop over a copy,
   ``scan[:]``.

A List Comprehension Builds a List in One Expression
----------------------------------------------------

.. code: lecture3/lists_demo.py 64-70

.. code-block:: python

   # List comprehensions
   squares = [n**2 for n in range(5)]
   print(squares)
   squares = []
   for n in range(5):
       squares.append(n**2)
   print(squares)

.. output: lists_demo.txt 20-21

.. code-block:: text

   [0, 1, 4, 9, 16]
   [0, 1, 4, 9, 16]

- ``[expression for item in iterable]`` is a **list comprehension**. It
  evaluates the expression once for each item and collects the results
  in a new list.
- The two versions build the same list. The comprehension says in one
  line what the list holds; the loop says how to build it, step by step.
- The loop variable ``n`` of a comprehension exists only inside the
  brackets.

A Comprehension Can Filter with if and Choose with if-else
----------------------------------------------------------

.. code: lecture3/lists_demo.py 72-77

.. code-block:: python

   # A comprehension with if, and with a conditional expression
   scan = [2.5, 0.0, 3.1, 13.5, 0.8]
   valid = [r for r in scan if 0.12 <= r <= 12.0]
   labels = ["near" if r < 1.0 else "far" for r in valid]
   print(valid)
   print(labels)

.. output: lists_demo.txt 22-23

.. code-block:: text

   [2.5, 3.1, 0.8]
   ['far', 'far', 'near']

- An ``if`` at the **end** filters: an item goes into the new list only
  when the condition is true. ``0.0`` and ``13.5`` are outside 0.12 to
  12.0 m, so ``valid`` keeps three readings.
- A conditional expression at the **front** (Lecture 2) chooses a value
  for every item: ``0.8`` is below 1.0, so it is ``"near"``.
- This fixes the loop of `Removing Items While Looping Skips Items`_:
  build the readings you keep, instead of removing the others.

Functions That Take an Iterable
-------------------------------

.. code: lecture3/lists_demo.py 79-82

.. code-block:: python

   # Functions that take an iterable
   print(len(valid), min(valid), max(valid), sum(valid))
   print(f"Mean: {sum(valid) / len(valid):.2f} m")
   print(any(r < 1.0 for r in valid), all(r < 1.0 for r in valid))

.. output: lists_demo.txt 24-26

.. code-block:: text

   3 0.8 3.1 6.4
   Mean: 2.13 m
   True False

``valid`` is ``[2.5, 3.1, 0.8]``, from the previous section. ``len()``,
``min()``, ``max()``, and ``sum()`` each read it once. The mean, in
steps:

1. 2.5 + 3.1 + 0.8 = 6.4.
2. 6.4 / 3 = 2.133..., printed with 2 decimals: ``2.13``.

``any()`` is ``True`` when at least one item is truthy; ``all()`` when
every item is. 0.8 is below 1.0, so ``any`` is ``True``; 2.5 is not, so
``all`` is ``False``.

``r < 1.0 for r in valid``, without brackets, is a **generator
expression**: it gives the values one at a time, like an iterator, and
builds no list. Ruff's default rule C419 asks for it. On a file whose
line 2 is ``print(any([r < 1.0 for r in valid]))``:

.. output: ruff_c419.txt 1

.. code-block:: text

   any_list.py:2:11: C419 Unnecessary list comprehension

A List of Lists Stores a Grid
-----------------------------

.. code: lecture3/lists_demo.py 84-90

.. code-block:: python

   # A 3 x 3 occupancy grid: 1 means occupied
   grid = [[0, 0, 0], [0, 1, 0], [0, 0, 0]]
   print(grid[1][1], len(grid), len(grid[0]))
   for row in grid:
       for cell in row:
           print("#" if cell else ".", end="")
       print()

.. output: lists_demo.txt 27-30

.. code-block:: text

   1 3 3
   ...
   .#.
   ...

- An **occupancy grid** splits the floor into cells and marks each cell
  free or occupied. ROS 2 sends a map as an occupancy grid
  (``nav_msgs/OccupancyGrid``, :doc:`Lecture 12 </lectures/lecture12/l12_index>`).
- ``grid`` is a list of 3 rows, and each row is a list of 3 cells.
  ``grid[1][1]`` reads row 1, then item 1 of that row: the center cell.
- A loop inside a loop is a **nested loop**. The inner loop prints the
  cells of one row on one line (``end=""``), then ``print()`` ends the
  line.

[[0] * 3] * 3 Repeats One Inner List
------------------------------------

.. code: lecture3/lists_demo.py 92-99

.. code-block:: python

   # Repetition copies references, not inner lists
   bad = [[0] * 3] * 3
   bad[0][0] = 1
   print(bad)
   good = [[0] * 3 for _ in range(3)]
   good[0][0] = 1
   print(good)
   print(bad[0] is bad[1], good[0] is good[1])

.. output: lists_demo.txt 31-33

.. code-block:: text

   [[1, 0, 0], [1, 0, 0], [1, 0, 0]]
   [[1, 0, 0], [0, 0, 0], [0, 0, 0]]
   True False

The intent: a 3 by 3 grid of zeros, then one occupied cell. With
``bad``, the whole first column changes. On ``s * n``, the
documentation: "items in the sequence s are not copied; they are
referenced multiple times." The last line checks it: ``bad[0] is
bad[1]`` is ``True``, ``good[0] is good[1]`` is ``False``.

.. figure:: /_static/images/l3/grid_repeat.svg
   :alt: Two panels. Tag 1: the name bad refers to a list with three slots, and all three slots have arrows to the same inner list, [1, 0, 0]. Tag 2: the name good refers to a list with three slots, and each slot has an arrow to its own inner list: [1, 0, 0], [0, 0, 0] and [0, 0, 0].
   :width: 85%

   Both grids after their first cell is set to 1. 1: the grid ``bad``.
   2: the grid ``good``.

- A list holds references to objects (the
  :doc:`Memory reading </reading_material/memory/mem_lecture>`,
  "Containers Hold References"). ``[[0] * 3] * 3`` repeats the reference
  to one inner list three times, so a change through ``bad[0]`` shows in
  every row.
- The comprehension evaluates ``[0] * 3`` once per row, so ``good`` has
  three inner lists.

Assignment, Shallow Copy, and Deep Copy
---------------------------------------

.. code: lecture3/lists_demo.py 3-4,101-110

.. code-block:: python

   import copy

   # Copies of a list of lists: waypoints as [x, y] lists
   path = [[0.0, 0.0], [1.0, 0.0]]
   alias = path
   shallow = path.copy()
   deep = copy.deepcopy(path)
   path[0][0] = 9.9
   path.append([2.0, 0.0])
   print(alias)
   print(shallow)
   print(deep)

.. output: lists_demo.txt 34-36

.. code-block:: text

   [[9.9, 0.0], [1.0, 0.0], [2.0, 0.0]]
   [[9.9, 0.0], [1.0, 0.0]]
   [[0.0, 0.0], [1.0, 0.0]]

``alias`` sees both changes: it is the same list. ``shallow`` sees the
change to ``path[0][0]``, but not the new waypoint. ``deep`` sees
neither.

.. list-table::
   :header-rows: 1
   :widths: 35 65
   :class: compact-table

   * - Code
     - What you get
   * - ``alias = path``
     - No copy: one list, two names (Lecture 1)
   * - ``path.copy()``, ``path[:]``, ``list(path)``
     - A **shallow copy**: a new outer list whose items refer to the
       same inner lists
   * - ``copy.deepcopy(path)``
     - A **deep copy**: new inner lists too

- ``shallow`` shares the inner list ``[0.0, 0.0]`` with ``path``, so
  ``path[0][0] = 9.9`` shows through it. Its outer list is its own, so
  the appended waypoint does not.
- A deep copy costs more time and memory, because it copies every inner
  object. Use it when the inner lists must stay independent.
- The :doc:`Memory reading </reading_material/memory/mem_lecture>`,
  "Shallow and Deep Copies", draws both copies.


Tuples
====================================================

A Tuple Is an Immutable Sequence
--------------------------------

.. code: lecture3/tuples_demo.py 3-6

.. code-block:: python
   :caption: From lecture3/tuples_demo.py

   # A pose: x (m), y (m), heading (deg), chosen for this example
   pose = (1.5, -0.3, 90)
   print(pose[0], pose[-1], pose[:2], len(pose))
   print(type(pose))

.. output: tuples_demo.txt 1-2

.. code-block:: text

   1.5 90 (1.5, -0.3) 3
   <class 'tuple'>

- A ``tuple`` holds items in order, between parentheses. Indexing,
  slicing, and ``len()`` work as for a list.
- A robot's pose in 2D is three numbers that belong together: ``x``,
  ``y``, and the heading. Each position has its own meaning.

.. code: lecture3/errors/tuple_assign.py all

.. code-block:: python
   :caption: lecture3/errors/tuple_assign.py

   pose = (1.5, -0.3, 90)
   pose[2] = 45

.. output: err_tuple_assign.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../tuple_assign.py", line 2, in <module>
       pose[2] = 45
       ~~~~^^^
   TypeError: 'tuple' object does not support item assignment

- A tuple is **immutable** (Lecture 1): once created, its items cannot
  be replaced, added, or removed. It has no ``append()`` and no
  ``sort()``.
- To change the heading, build a new tuple:
  ``pose = (pose[0], pose[1], 45)``.

The Comma Makes the Tuple, Not the Parentheses
----------------------------------------------

.. code: lecture3/tuples_demo.py 8-15

.. code-block:: python

   # The comma makes the tuple
   point = 3.0, 4.0
   single = (42,)
   not_a_tuple = (42)  # fmt: skip
   empty = ()
   print(type(point), type(single))
   print(type(not_a_tuple), type(empty))
   print(tuple(range(3)), tuple("xy"))

.. output: tuples_demo.txt 3-5

.. code-block:: text

   <class 'tuple'> <class 'tuple'>
   <class 'int'> <class 'tuple'>
   (0, 1, 2) ('x', 'y')

- ``3.0, 4.0`` is a tuple with no parentheses. ``(42,)`` is a tuple of
  one item; ``(42)`` is only the number 42 in parentheses.
- The tutorial: "a tuple with one item is constructed by following a
  value with a comma (it is not sufficient to enclose a single value in
  parentheses)."
- ``tuple()`` builds a tuple from any iterable, as ``list()`` builds a
  list.

Ruff's formatter removes parentheses that do nothing. On a file that
holds only ``not_a_tuple = (42)``,
``uv run ruff format --diff paren.py`` shows:

.. output: ruff_format_paren.txt all

.. code-block:: text

   --- paren.py
   +++ paren.py
   @@ -1 +1 @@
   -not_a_tuple = (42)
   +not_a_tuple = 42

   1 file would be reformatted

``tuples_demo.py`` ends that line with ``# fmt: skip``, which tells the
formatter to leave the line as written, so the demo still shows the
mistake. For a tuple of one item, write the comma: ``(42,)``.

Unpacking Gives Each Item a Name
--------------------------------

.. code: lecture3/tuples_demo.py 17-24

.. code-block:: python

   # Unpacking
   x, y, heading = pose
   print(x, y, heading)
   waypoints = [(0.0, 0.0), (1.0, 0.5), (2.0, 1.5)]
   for x, y in waypoints:
       print(f"Go to ({x}, {y})")
   first, *rest = waypoints
   print(first, rest)

.. output: tuples_demo.txt 6-10

.. code-block:: text

   1.5 -0.3 90
   Go to (0.0, 0.0)
   Go to (1.0, 0.5)
   Go to (2.0, 1.5)
   (0.0, 0.0) [(1.0, 0.5), (2.0, 1.5)]

- Lecture 1 unpacked ``a, b = 1, 2``. Any iterable on the right works:
  ``x, y, heading = pose`` binds one name per item.
- ``for x, y in waypoints:`` unpacks each tuple of the list, as
  ``enumerate()`` and ``zip()`` did.
- ``*rest`` collects every item not bound to another name, in a
  **list**: ``first`` gets the first waypoint, ``rest`` the other two.

Unpacking Needs One Name per Item
---------------------------------

.. code: lecture3/errors/unpack_count.py all

.. code-block:: python
   :caption: lecture3/errors/unpack_count.py

   pose = (1.5, -0.3, 90)
   x, y = pose

.. output: err_unpack_count.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../unpack_count.py", line 2, in <module>
       x, y = pose
       ^^^^
   ValueError: too many values to unpack (expected 2)

``pose`` has 3 items and the left side has 2 names, so Python refuses to
guess which item to drop. To skip an item, bind it to ``_``, the name
that means "not used":

.. code: lecture3/tuples_demo.py 25-26

.. code-block:: python

   x, y, _ = pose
   print(x, y)

.. output: tuples_demo.txt 11

.. code-block:: text

   1.5 -0.3

A Tuple Can Hold a Mutable Object
---------------------------------

.. code: lecture3/tuples_demo.py 28-31

.. code-block:: python

   # A tuple can hold a mutable object
   record = ("lidar", [2.5, 0.9])
   record[1].append(3.1)
   print(record)

.. output: tuples_demo.txt 12

.. code-block:: text

   ('lidar', [2.5, 0.9, 3.1])

.. code: lecture3/errors/unhashable_tuple.py all

.. code-block:: python
   :caption: lecture3/errors/unhashable_tuple.py

   record = ("lidar", [2.5, 0.9])
   print(hash(record))

.. output: err_unhashable_tuple.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../unhashable_tuple.py", line 2, in <module>
       print(hash(record))
             ^^^^^^^^^^^^
   TypeError: unhashable type: 'list'

- The tuple holds two references. They cannot change, but the list that
  item 1 refers to can.
- ``hash()`` computes a number from a value; dictionaries and sets use
  it (next sections). The glossary: tuples "are only hashable if their
  elements are hashable." A list is not, so this tuple is not either.

Tuple or List
-------------

.. list-table::
   :header-rows: 1
   :widths: 20 40 40
   :class: compact-table

   * -
     - ``tuple``
     - ``list``
   * - Number of items
     - Fixed when it is created
     - Grows and shrinks
   * - Each position
     - Has its own meaning: ``x``, ``y``, heading
     - Holds one more item of the same kind
   * - Can change
     - No
     - Yes
   * - Dictionary key
     - Yes, when its items are hashable
     - No
   * - Example
     - A pose, an (angle, range) pair
     - The ranges of one scan

- A tuple tells the reader that the record will not change. Python
  enforces it.
- ``zip()`` and ``enumerate()`` give tuples.
  :doc:`Lecture 4 </lectures/lecture4/l4_index>` returns several values
  from a function as a tuple.


Dictionaries
====================================================

A Dictionary Maps Keys to Values
--------------------------------

.. code: lecture3/dicts_demo.py 6-14

.. code-block:: python
   :caption: From lecture3/dicts_demo.py

   # Keys and values (chosen for this example)
   robot = {
       "name": "waffle_01",
       "max_speed_mps": 0.26,
       "sensors": ["lidar"],
   }
   print(robot["name"], robot["max_speed_mps"])
   print(len(robot))
   print(robot)

.. output: dicts_demo.txt 1-3

.. code-block:: text

   waffle_01 0.26
   3
   {'name': 'waffle_01', 'max_speed_mps': 0.26, 'sensors': ['lidar']}

- A ``dict`` holds **key: value** pairs between braces. You look a value
  up by its key, ``robot["name"]``, not by a position.
- Each key appears once, and ``len()`` counts the pairs: 3. A value can
  be anything, even a list.
- The documentation: "Dictionaries preserve insertion order." (guaranteed
  since Python 3.7).

Three Ways to Create a Dictionary
---------------------------------

.. code: lecture3/dicts_demo.py 16-24

.. code-block:: python

   # Ways to create a dictionary
   a = {"x": 1.0, "y": 2.0}
   pairs = [("x", 1.0), ("y", 2.0)]
   b = dict(pairs)
   keys = ["x", "y"]
   values = [1.0, 2.0]
   c = dict(zip(keys, values))
   print(a == b == c, a)
   print({}, type({}))

.. output: dicts_demo.txt 4-5

.. code-block:: text

   True {'x': 1.0, 'y': 2.0}
   {} <class 'dict'>

- Write the pairs out in braces when you know them. Use ``dict()`` when
  the pairs already exist as data: a list of (key, value) tuples, or two
  lists joined with ``zip()``.
- The three dictionaries are equal: ``a == b == c`` is a chained
  comparison (Lecture 2).
- ``{}`` is an empty **dictionary**, not an empty set (see `Sets`_).

A Missing Key Raises KeyError
-----------------------------

.. code: lecture3/errors/key_error.py all

.. code-block:: python
   :caption: lecture3/errors/key_error.py

   robot = {"name": "waffle_01", "max_speed_mps": 0.26}
   print(robot["battery_pct"])

.. output: err_key_error.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../key_error.py", line 2, in <module>
       print(robot["battery_pct"])
             ~~~~~^^^^^^^^^^^^^^^
   KeyError: 'battery_pct'

- ``robot["battery_pct"]`` asks for a key the dictionary does not have,
  so Python raises a ``KeyError`` that names the key.
- A list raises an ``IndexError`` for a missing index (Lecture 2). A
  dictionary has no positions, so its error names the missing key.

get() and in Handle a Missing Key
---------------------------------

.. code: lecture3/dicts_demo.py 26-30

.. code-block:: python

   # A missing key: get() and in
   robot = {"name": "waffle_01", "max_speed_mps": 0.26}
   print(robot.get("battery_pct"))
   print(robot.get("battery_pct", 100))
   print("battery_pct" in robot, "name" in robot)

.. output: dicts_demo.txt 6-8

.. code-block:: text

   None
   100
   False True

- ``robot.get(key)`` returns ``None`` for a missing key;
  ``robot.get(key, 100)`` returns the default you give. Neither raises an
  error.
- ``key in robot`` tests the **keys**, not the values: ``"name" in
  robot`` is ``True``, ``"waffle_01" in robot`` would be ``False``.
- Test with ``in`` when a missing key needs its own branch; use ``get()``
  when a default value is enough.

Adding, Changing, and Removing Entries
--------------------------------------

.. code: lecture3/dicts_demo.py 32-40

.. code-block:: python

   # Adding, changing, and removing entries
   robot["battery_pct"] = 87  # A new key: added at the end
   robot["max_speed_mps"] = 0.2  # An existing key: value replaced
   print(robot)
   robot.update({"mode": "map", "battery_pct": 80})
   print(robot["mode"], robot["battery_pct"])
   speed = robot.pop("max_speed_mps")
   del robot["mode"]
   print(speed, robot)

.. output: dicts_demo.txt 9-11

.. code-block:: text

   {'name': 'waffle_01', 'max_speed_mps': 0.2, 'battery_pct': 87}
   map 80
   0.2 {'name': 'waffle_01', 'battery_pct': 80}

- ``robot[key] = value`` adds the pair when the key is new, and replaces
  the value when the key exists. The documentation: "updating a key does
  not affect the order."
- ``update()`` does the same for every pair of another dictionary:
  ``"mode"`` is added, and ``"battery_pct"`` changes from 87 to 80.
- ``pop(key)`` removes the pair and returns its value (0.2).
  ``del robot[key]`` removes it and returns nothing. Both raise a
  ``KeyError`` for a missing key.

A Key Must Be Hashable
----------------------

.. code: lecture3/dicts_demo.py 42-44

.. code-block:: python

   # Keys must be hashable: a grid cell (row, col) as the key
   costs = {(0, 0): 1.0, (0, 1): 5.0}
   print(costs[(0, 1)], costs[0, 1])

.. output: dicts_demo.txt 12

.. code-block:: text

   5.0 5.0

.. code: lecture3/errors/unhashable_key.py all

.. code-block:: python
   :caption: lecture3/errors/unhashable_key.py

   costs = {[0, 0]: 1.0, [0, 1]: 5.0}

.. output: err_unhashable_key.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../unhashable_key.py", line 1, in <module>
       costs = {[0, 0]: 1.0, [0, 1]: 5.0}
               ^^^^^^^^^^^^^^^^^^^^^^^^^^
   TypeError: unhashable type: 'list'

- A dictionary finds a key through its hash. The glossary: "An object is
  hashable if it has a hash value which never changes during its
  lifetime".
- A list can change, so its hash could change: lists are not hashable. A
  tuple of numbers is, so a grid cell ``(row, col)`` can be a key.
  ``costs[0, 1]`` is the same lookup: the comma makes the tuple.

keys(), values(), and items() Are Live Views
--------------------------------------------

.. code: lecture3/dicts_demo.py 46-53

.. code-block:: python

   # Views
   robot = {"name": "waffle_01", "battery_pct": 87}
   keys = robot.keys()
   print(keys)
   robot["mode"] = "explore"
   print(keys)
   print(list(robot.values()))
   print(list(robot.items()))

.. output: dicts_demo.txt 13-16

.. code-block:: text

   dict_keys(['name', 'battery_pct'])
   dict_keys(['name', 'battery_pct', 'mode'])
   ['waffle_01', 87, 'explore']
   [('name', 'waffle_01'), ('battery_pct', 87), ('mode', 'explore')]

- ``keys()``, ``values()``, and ``items()`` return **view objects**. The
  documentation: "when the dictionary changes, the view reflects these
  changes."
- ``keys`` was created before ``"mode"`` was added, and the second print
  shows ``"mode"`` anyway: the view is not a copy.
- ``items()`` gives (key, value) tuples. ``list()`` copies a view into a
  list that no longer changes.

Looping over a Dictionary
-------------------------

.. code: lecture3/dicts_demo.py 55-59

.. code-block:: python

   # Looping over a dictionary
   for key in robot:
       print(key)
   for key, value in robot.items():
       print(f"{key:>11}: {value}")

.. output: dicts_demo.txt 17-22

.. code-block:: text

   name
   battery_pct
   mode
          name: waffle_01
   battery_pct: 87
          mode: explore

- A ``for`` loop over a dictionary gives its keys, in insertion order.
- ``for key, value in robot.items():`` gives each pair and unpacks it.
  It reads better than ``robot[key]`` inside the loop.
- ``:>11`` right-aligns each key in 11 characters, the length of
  ``"battery_pct"``, so the colons line up.

Removing Keys While Looping Is a RuntimeError
---------------------------------------------

.. code: lecture3/errors/dict_size_change.py all

.. code-block:: python
   :caption: lecture3/errors/dict_size_change.py

   battery = {"robot1": 87, "robot2": 15, "robot3": 42}
   for name in battery:
       if battery[name] < 20:
           del battery[name]

.. output: err_dict_size_change.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../dict_size_change.py", line 2, in <module>
       for name in battery:
   RuntimeError: dictionary changed size during iteration

The loop deletes ``"robot2"`` (15%), and the next step of the loop
raises a ``RuntimeError``. A list skipped items without an error (see
`Lists`_); a dictionary checks its size at each step and refuses. The
fix: loop over a copy of the keys.

.. code: lecture3/dicts_demo.py 61-66

.. code-block:: python

   # Removing entries while looping: loop over a copy of the keys
   battery = {"robot1": 87, "robot2": 15, "robot3": 42}
   for name in list(battery):
       if battery[name] < 20:
           del battery[name]
   print(battery)

.. output: dicts_demo.txt 23

.. code-block:: text

   {'robot1': 87, 'robot3': 42}

``list(battery)`` copies the keys into a list first. The loop reads that
list, not the dictionary it changes. A dictionary comprehension (next
section) builds the robots to keep instead, with no loop to break.

Dictionary Comprehensions
-------------------------

.. code: lecture3/dicts_demo.py 68-76

.. code-block:: python

   # Dictionary comprehensions
   sensors = ["lidar", "camera", "imu"]
   sensor_ids = {name: i for i, name in enumerate(sensors)}
   battery = {"robot1": 87, "robot2": 15, "robot3": 42}
   ok = {name: pct for name, pct in battery.items() if pct >= 20}
   by_id = {i: name for name, i in sensor_ids.items()}
   print(sensor_ids)
   print(ok)
   print(by_id)

.. output: dicts_demo.txt 24-26

.. code-block:: text

   {'lidar': 0, 'camera': 1, 'imu': 2}
   {'robot1': 87, 'robot3': 42}
   {0: 'lidar', 1: 'camera', 2: 'imu'}

- ``{key: value for item in iterable}`` builds a dictionary, as a list
  comprehension builds a list. An ``if`` at the end filters.
- ``sensor_ids`` gives each sensor name its position in the list.
- ``ok`` keeps the robots at 20% or more: a second fix for
  `Removing Keys While Looping Is a RuntimeError`_, with no loop over the
  dictionary it changes.
- ``by_id`` swaps keys and values. This works when every value is
  hashable and appears once.

Counting with a Dictionary
--------------------------

.. code: lecture3/dicts_demo.py 3,5,78-84

.. code-block:: python

   from collections import Counter

   # Counting with a dictionary (the log of Lecture 2, Exercise 4)
   status_log = "IDLE:MOVING:CHARGING:MOVING:IDLE"
   counts = {}
   for state in status_log.split(":"):
       counts[state] = counts.get(state, 0) + 1
   print(counts)
   print(Counter(status_log.split(":")))

.. output: dicts_demo.txt 27-28

.. code-block:: text

   {'IDLE': 2, 'MOVING': 2, 'CHARGING': 1}
   Counter({'IDLE': 2, 'MOVING': 2, 'CHARGING': 1})

- Lecture 2's Exercise 4 counted one state with ``count("MOVING")``. A
  dictionary counts every state in one pass.
- ``counts.get(state, 0)`` is 0 the first time a state is seen, so the
  first count becomes 1.
- ``collections.Counter`` is a dictionary built for counting: it gives
  the same counts in one line.

A Value Can Be Another Dictionary
---------------------------------

.. code: lecture3/dicts_demo.py 4-5,86-93

.. code-block:: python

   from pprint import pp

   # Nested dictionaries
   robot = {
       "name": "waffle_01",
       "lidar": {"range_max_m": 12.0, "rate_hz": 5},
       "camera": {"fps": 30, "width_px": 640},
   }
   print(robot["lidar"]["range_max_m"])
   pp(robot, width=40)

.. output: dicts_demo.txt 29-33

.. code-block:: text

   12.0
   {'name': 'waffle_01',
    'lidar': {'range_max_m': 12.0,
              'rate_hz': 5},
    'camera': {'fps': 30, 'width_px': 640}}

- The lidar's settings and the camera's settings are dictionaries
  inside ``robot``. Each ``[...]`` goes one level down:
  ``robot["lidar"]`` is the lidar's dictionary, and ``["range_max_m"]``
  reads one value from it.
- ROS 2 parameter files (:doc:`Lecture 10 </lectures/lecture10/l10_index>`)
  nest settings the same way: the name of a node, then its parameters.
- ``print(robot)`` would put the whole dictionary on one line of 107
  characters: ``len(str(robot))`` gives 107. ``pp()``, from the ``pprint`` module ("pretty print"),
  breaks it into lines no wider than ``width`` characters, and shows the
  nesting by indentation.
- ``pp()`` keeps insertion order. The older ``pprint()`` sorts the keys
  by default.


Sets
====================================================

A Set Holds Each Item Once
--------------------------

.. code: lecture3/sets_demo.py 3-8

.. code-block:: python
   :caption: From lecture3/sets_demo.py

   # A set holds each item once (values chosen for this example)
   readings = [1.5, 2.3, 1.5, 3.7, 2.3]
   unique = set(readings)
   print(unique)
   print(len(readings), len(unique))
   print(type(set()), type({}))

.. output: sets_demo.txt 1-3

.. code-block:: text

   {1.5, 2.3, 3.7}
   5 3
   <class 'set'> <class 'dict'>

- The documentation: "A set object is an unordered collection of
  distinct hashable objects."
- ``set(readings)`` keeps one copy of each value: 5 readings, 3 distinct
  values. Each item must be hashable, as a dictionary key must be.
- ``set()`` is an empty set; ``{}`` is an empty dictionary. The tutorial:
  "to create an empty set you have to use set(), not {}".
- A set has no positions, so it has no indexing and no slicing:
  ``unique[0]`` raises a ``TypeError``.

The Order of a Set of Strings Changes Between Runs
--------------------------------------------------

.. code: lecture3/set_order_demo.py all

.. code-block:: python
   :caption: lecture3/set_order_demo.py

   """Print a set of strings, then the same strings sorted."""

   topics = {"/scan", "/odom", "/cmd_vel", "/tf"}
   print(topics)
   print(sorted(topics))

Run with ``PYTHONHASHSEED=1 uv run lecture3/set_order_demo.py``:

.. output: set_order_seed1.txt all

.. code-block:: text

   {'/odom', '/tf', '/scan', '/cmd_vel'}
   ['/cmd_vel', '/odom', '/scan', '/tf']

Run with ``PYTHONHASHSEED=2 uv run lecture3/set_order_demo.py``:

.. output: set_order_seed2.txt all

.. code-block:: text

   {'/scan', '/odom', '/tf', '/cmd_vel'}
   ['/cmd_vel', '/odom', '/scan', '/tf']

- A set places each item by its hash. The documentation, on
  ``PYTHONHASHSEED``: "If this variable is not set or set to random, a
  random value is used to seed the hashes of str and bytes objects."
- So a set of strings can print in a different order on each run.
  ``sorted()`` gives the same list every time. Sort a set before you
  print it or compare its order.

in Is Much Faster on a Set than on a List
-----------------------------------------

.. code: lecture3/membership_timing.py all

.. code-block:: python
   :caption: lecture3/membership_timing.py

   """Time the in operator on a list and on a set of 1,000,000 integers."""

   from timeit import repeat

   N = 1_000_000
   as_list = list(range(N))
   as_set = set(as_list)
   target = N - 1  # The last item: the worst case for a list

   # repeat() times 5 rounds of n checks; keep the fastest round
   n = 20
   rounds = repeat("target in as_list", globals=globals(), number=n)
   t_list = min(rounds) / n
   n = 1_000_000  # A set check is fast: many more checks per round
   rounds = repeat("target in as_set", globals=globals(), number=n)
   t_set = min(rounds) / n
   print(f"list: {t_list * 1e6:10.3f} microseconds per check")
   print(f"set:  {t_set * 1e6:10.3f} microseconds per check")
   print(f"The set is {t_list / t_set:,.0f} times faster")

Three of ten separate runs:

.. output: membership_timing_run1.txt all

.. code-block:: text

   list:   4876.157 microseconds per check
   set:       0.018 microseconds per check
   The set is 270,950 times faster

.. output: membership_timing_run2.txt all

.. code-block:: text

   list:   4875.378 microseconds per check
   set:       0.018 microseconds per check
   The set is 271,686 times faster

.. output: membership_timing_run3.txt all

.. code-block:: text

   list:   5553.727 microseconds per check
   set:       0.018 microseconds per check
   The set is 309,915 times faster

- ``repeat()`` runs a statement ``number`` times, in 5 rounds, and
  returns the 5 total times in seconds. The fastest round is the one
  least slowed by other programs; dividing it by ``n`` gives one check.
- One set check takes about 0.02 microseconds, so the set needs a
  million checks per round to be timed reliably.
- ``in`` on a list compares the target with each item from the start:
  one million comparisons for the last item. A set computes the hash of
  the target and goes straight to the place that hash gives.
- In 10 runs on the instructor's PC (Intel Core i9-14900HX), the set was
  270,475 to 309,915 times faster. Expect other numbers on another PC;
  the gap stays.
- A dictionary finds a key the same way, so ``key in d`` is fast too.

.. tip::

   To test membership many times in a large collection, put the items in
   a set.

Set Operations Compare Two Collections
--------------------------------------

.. code: lecture3/sets_demo.py 10-16

.. code-block:: python

   # Set operations (detections chosen for this example)
   lidar_sees = {"wall", "person", "cone", "pallet"}
   camera_sees = {"person", "cone", "door"}
   print(sorted(lidar_sees | camera_sees))
   print(sorted(lidar_sees & camera_sees))
   print(sorted(lidar_sees - camera_sees))
   print(sorted(lidar_sees ^ camera_sees))

.. output: sets_demo.txt 4-7

.. code-block:: text

   ['cone', 'door', 'pallet', 'person', 'wall']
   ['cone', 'person']
   ['pallet', 'wall']
   ['door', 'pallet', 'wall']

.. figure:: /_static/images/l3/set_venn.svg
   :alt: Two overlapping circles. The name lidar_sees points to the left circle and camera_sees to the right one. Tag 1, the left part only: "wall" and "pallet". Tag 2, the overlap: "person" and "cone". Tag 3, the right part only: "door".
   :width: 60%

   1: only the lidar saw it. 2: both sensors. 3: only the camera.

.. list-table::
   :header-rows: 1
   :widths: 30 40 30
   :class: compact-table

   * - Operator
     - Method
     - Regions, with ``a`` = ``lidar_sees``, ``b`` = ``camera_sees``
   * - ``a | b``
     - ``a.union(b)``
     - 1, 2, and 3
   * - ``a & b``
     - ``a.intersection(b)``
     - 2
   * - ``a - b``
     - ``a.difference(b)``
     - 1
   * - ``b - a``
     - ``b.difference(a)``
     - 3
   * - ``a ^ b``
     - ``a.symmetric_difference(b)``
     - 1 and 3

- Region 2 holds the objects that both sensors report: ``"person"`` and
  ``"cone"``.
- A difference depends on the order of the two sets: ``b - a`` gives
  ``"door"``, ``a - b`` gives ``"pallet"`` and ``"wall"``.
- The outputs are sorted, so their order is the same on every run.

Adding and Removing Items
-------------------------

.. code: lecture3/sets_demo.py 18-25

.. code-block:: python

   # Adding and removing items
   seen = set()
   for label in ["person", "cone", "person"]:
       seen.add(label)
   print(sorted(seen))
   seen.discard("door")  # Not there: no error
   seen.remove("cone")
   print(sorted(seen))

.. output: sets_demo.txt 8-9

.. code-block:: text

   ['cone', 'person']
   ['person']

- ``add()`` adds one item; adding ``"person"`` a second time changes
  nothing.
- ``discard(x)`` does nothing when ``x`` is missing. ``remove(x)`` raises
  a ``KeyError`` then. The last line of the traceback:

.. code: lecture3/errors/set_remove_missing.py all

.. code-block:: python
   :caption: lecture3/errors/set_remove_missing.py

   seen = {"person", "cone"}
   seen.remove("door")

.. output: err_set_remove_missing.txt 4

.. code-block:: text

   KeyError: 'door'

A Set Comprehension
-------------------

.. code: lecture3/sets_demo.py 27-30

.. code-block:: python

   # A set comprehension
   topics = ["/scan", "/odom", "/robot1/scan", "/robot2/scan"]
   names = {t.split("/")[-1] for t in topics}
   print(sorted(names))

.. output: sets_demo.txt 10

.. code-block:: text

   ['odom', 'scan']

- A set comprehension is written in braces, with no ``key:`` part. Its
  result has each value once.
- ``t.split("/")[-1]`` keeps the last part of each topic name. Three
  topics end in ``scan``, so the set keeps two names.
- In a multi-robot system, this lists the kinds of topics in use,
  whatever the robot namespace
  (:doc:`Lecture 9 </lectures/lecture9/l9_index>` covers namespaces).


Choosing a Collection
====================================================

.. list-table::
   :header-rows: 1
   :widths: 20 20 20 20 20
   :class: compact-table

   * -
     - ``list``
     - ``tuple``
     - ``dict``
     - ``set``
   * - Order
     - Positions
     - Positions
     - Insertion order
     - None
   * - Can change
     - Yes
     - No
     - Yes
     - Yes
   * - Duplicates
     - Yes
     - Yes
     - Keys: no
     - No
   * - Find an item by
     - Index
     - Index
     - Key
     - Membership only
   * - ``x in c`` checks
     - Each item
     - Each item
     - The hash of a key
     - The hash
   * - Robot example
     - The ranges of a scan
     - A pose
     - A robot's settings
     - Topics already seen

- Ask what you look items up by. A position: a list or a tuple. A name
  or another key: a dictionary. Only "is it there?": a set.
- Ask whether the record changes. If it never does, a tuple says so and
  can be a dictionary key.


Next Class
====================================================

:doc:`L4: Functions </lectures/lecture4/l4_index>` (Tuesday, February
23): defining and calling functions, arguments, scopes, type hints, and
functions as values.
