====================================================
Lecture
====================================================

Every program on this page is in the ``lecture5/`` folder of the
`course repository <https://github.com/rubixcubic/enpm605-spring-2027-python>`_.
Run each one from the root of the repository, for example:

.. code-block:: bash

   uv run lecture5/arrays_demo.py

Every output on this page was printed by running the program in the
course environment (``.venv``, what ``uv run`` uses): Python 3.12.3 on
Ubuntu 24.04, with NumPy 1.26.4 and Matplotlib 3.11.2 installed by uv
(`Installing NumPy next to ROS 2`_). Before you run the plotting
programs, run ``uv sync`` in the course repository: it installs
Matplotlib. In paths,
``...`` marks a path shortened to fit. The values in the examples
(ranges, poses, positions) are chosen for the examples, and the laser
scan is synthetic: see `This Lecture's Scan Is Synthetic`_.


Arrays versus Lists
====================================================

One Array Expression Replaces a Loop over a List
------------------------------------------------

.. code: lecture5/arrays_demo.py 3-9

.. code-block:: python
   :caption: From lecture5/arrays_demo.py

   import numpy as np

   # The same computation with a list and with an array
   ranges_list = [2.5, 0.9, 3.1, 1.2]  # Meters, chosen for this example
   print([r * 100 for r in ranges_list])  # To centimeters
   ranges = np.array(ranges_list)
   print(ranges * 100)

.. output: arrays_demo.txt 1-2

.. code-block:: text

   [250.0, 90.0, 310.0, 120.0]
   [250.  90. 310. 120.]

- ``np.array()`` makes a NumPy array, type ``numpy.ndarray``, from a
  list. The NumPy guide for beginners: "We recommend to always use
  ``import numpy as np``."
- ``ranges * 100`` multiplies every item. Your code has no loop; NumPy
  runs the loop. Its guide
  (`What is NumPy? <https://numpy.org/doc/1.26/user/whatisnumpy.html>`_):
  "the element-by-element operation is speedily executed by pre-compiled
  C code."
- An array prints without commas, and ``250.`` is the float 250.0.

The Operators + and * Do Different Things to Lists and Arrays
-------------------------------------------------------------

.. code: lecture5/arrays_demo.py 11-15

.. code-block:: python

   # + and * mean different things for a list and for an array
   print(ranges_list + ranges_list)
   print(ranges + ranges)
   print(ranges_list * 2)
   print(ranges * 2)

.. output: arrays_demo.txt 3-6

.. code-block:: text

   [2.5, 0.9, 3.1, 1.2, 2.5, 0.9, 3.1, 1.2]
   [5.  1.8 6.2 2.4]
   [2.5, 0.9, 3.1, 1.2, 2.5, 0.9, 3.1, 1.2]
   [5.  1.8 6.2 2.4]

- For lists (Lecture 3), ``+`` joins two lists and ``* 2`` repeats the
  list.
- For arrays, ``+`` adds item by item and ``* 2`` doubles each number.

.. warning::

   A function written for a list of readings can return a different
   answer when it gets an array. Python reports no error.

Functions That Make an Array
----------------------------

.. code: lecture5/arrays_demo.py 17-21

.. code-block:: python

   # Ways to make an array
   print(np.zeros(3), np.ones(2))
   print(np.full(3, np.inf))
   print(np.arange(0.0, 1.0, 0.25))
   print(np.linspace(0.0, 1.0, 5))

.. output: arrays_demo.txt 7-10

.. code-block:: text

   [0. 0. 0.] [1. 1.]
   [inf inf inf]
   [0.   0.25 0.5  0.75]
   [0.   0.25 0.5  0.75 1.  ]

- ``np.zeros(n)``, ``np.ones(n)``, ``np.full(n, value)``: ``n`` items
  with one value. ``np.inf`` is the float infinity.
- ``np.arange(start, stop, step)``: like ``range()``, ``stop`` left out,
  but the step can be a float.
- ``np.linspace(start, stop, num)``: ``num`` evenly spaced numbers,
  ``stop`` included.
- The `numpy.arange
  <https://numpy.org/doc/1.26/reference/generated/numpy.arange.html>`_
  page: "When using a non-integer step, such as 0.1, it is often better
  to use ``numpy.linspace``."

Every Item of an Array Has the Same Data Type
---------------------------------------------

.. code: lecture5/arrays_demo.py 23-26

.. code-block:: python

   # One data type for every item
   print(np.array([1, 2, 3]).dtype)
   print(np.array([1, 2.5, 3]).dtype)
   print(repr(np.array([1, 2.5, "lidar"])))

.. output: arrays_demo.txt 11-13

.. code-block:: text

   int64
   float64
   array(['1', '2.5', 'lidar'], dtype='<U32')

- ``.dtype`` is the **data type** of the items: ``int64`` (8-byte
  integers) or ``float64`` (8-byte floats).
- One float makes every item a float. One string makes every item a
  string: ``<U32`` is a Unicode string of up to 32 characters.
- A list can mix types (Lecture 3); an array cannot. That is what lets
  NumPy store the numbers side by side.

An Array Stores the Numbers Side by Side
----------------------------------------

.. figure:: /_static/images/l5/list_vs_array.svg
   :alt: Two parts. Tag 1: a Python list is a block of slots; each slot holds a reference, drawn as an arrow, to a separate float object (0.0, 1.0, 2.0) somewhere else on the heap. Tag 2: a NumPy array of float64 is one block in which the numbers 0.0, 1.0, 2.0, 3.0 sit side by side, with no separate objects.
   :width: 80%

   1: a list of floats. 2: a NumPy array of ``float64``.

- A list holds references to separate float objects. The array holds
  the numbers themselves.
- The :doc:`Memory reading </reading_material/memory/mem_index>`,
  "Measuring Memory", measured 1,000,000 floats: 32,448,672 bytes as a
  list, 8,000,000 bytes as an array.

nbytes and itemsize Count an Array's Bytes
------------------------------------------

.. code: lecture5/arrays_demo.py 28-31

.. code-block:: python

   # An array holds the numbers themselves, 8 bytes each
   a = np.arange(1_000_000, dtype=np.float64)
   print(a.nbytes, a.itemsize)
   print(type(a).__name__, type(a[0]).__name__)

.. output: arrays_demo.txt 14-15

.. code-block:: text

   8000000 8
   ndarray float64

- ``np.arange(1_000_000, dtype=np.float64)``: the floats 0.0 to
  999999.0.
- ``.itemsize`` is the bytes per item, 8. ``.nbytes`` is all of them:
  1,000,000 × 8 = 8,000,000.
- An item read from the array is a ``numpy.float64``, which NumPy makes
  when you read it. It behaves like a ``float``.

Timing One Conversion on a List and on an Array
-----------------------------------------------

.. code: lecture5/array_timing.py all

.. code-block:: python
   :caption: lecture5/array_timing.py

   """Time one operation on 1,000,000 ranges: a list against an array."""

   from timeit import repeat

   import numpy as np

   N = 1_000_000
   rng = np.random.default_rng(605)  # Seeded: the same ranges every run
   ranges = rng.uniform(0.12, 12.0, N)  # Meters
   ranges_list = ranges.tolist()  # The same numbers, as a list of floats

   # Convert every range to centimeters; repeat() times 5 rounds of n
   # conversions, and the fastest round, divided by n, is one conversion
   n = 10
   to_cm = "[r * 100 for r in ranges_list]"
   rounds = repeat(to_cm, globals=globals(), number=n)
   t_list = min(rounds) / n
   rounds = repeat("ranges * 100", globals=globals(), number=n)
   t_array = min(rounds) / n
   print(f"list:  {t_list * 1e3:7.3f} ms per conversion")
   print(f"array: {t_array * 1e3:7.3f} ms per conversion")
   print(f"The array is {t_list / t_array:.0f} times faster")

- 1,000,000 ranges, seeded, so every run converts the same numbers.
  ``.tolist()`` gives the same numbers as a list.
- As in Lecture 3's ``membership_timing.py``: ``repeat()`` times 5
  rounds of 10 conversions; the fastest round, divided by 10, is one
  conversion.

The Array Was 75 to 81 Times Faster in 10 Runs
----------------------------------------------

Three of the ten recorded runs:

.. output: array_timing_run1.txt all

.. code-block:: text

   list:   30.843 ms per conversion
   array:   0.382 ms per conversion
   The array is 81 times faster

.. output: array_timing_run4.txt all

.. code-block:: text

   list:   29.669 ms per conversion
   array:   0.385 ms per conversion
   The array is 77 times faster

.. output: array_timing_run10.txt all

.. code-block:: text

   list:   29.454 ms per conversion
   array:   0.385 ms per conversion
   The array is 76 times faster

- Across all 10 runs on the instructor's PC (Intel Core i9-14900HX): the
  list took 28.654 to 31.244 ms, the array 0.380 to 0.386 ms, and the
  array was 75 to 81 times faster. Another PC gives other numbers.
- The list comprehension runs Python code for each item and makes a new
  float object for each result. The array runs one compiled loop over
  8-byte numbers.


Shape, Data Type, and Indexing
====================================================

A 2D Array of Poses Has a Shape
-------------------------------

.. code: lecture5/indexing_demo.py 5-6,8-13,15-16

.. code-block:: python
   :caption: From lecture5/indexing_demo.py

   # Poses of a robot: one row per pose; columns x (m), y (m), theta (rad).
   # Chosen for this example.
   poses = np.array([
       [0.0, 0.0, 0.00],
       [1.0, 0.0, 0.00],
       [2.0, 0.5, 0.46],
       [2.5, 1.5, 1.11],
   ])
   print(poses.ndim, poses.shape, poses.size)
   print(poses.dtype, poses.itemsize, poses.nbytes)

.. output: indexing_demo.txt 1-2

.. code-block:: text

   2 (4, 3) 12
   float64 8 96

- A **pose** is a position and a heading: ``x``, ``y``, and the angle
  ``theta`` from the x axis. Here, one pose per row.
- ``ndim`` is 2 (rows and columns). ``shape`` is ``(4, 3)``: 4 rows, 3
  columns. ``size`` is 4 × 3 = 12 items.
- ``nbytes``: 12 items × 8 bytes = 96.
- From pose 1 on, each ``theta`` is ``np.arctan2(dy, dx)`` from the
  previous pose: ``np.arctan2(0.5, 1.0)`` is 0.4636 and
  ``np.arctan2(1.0, 0.5)`` is 1.1071, rounded to 0.46 and 1.11.
- The file keeps the array on 6 lines with ``# fmt: off`` and
  ``# fmt: on``, which tell Ruff's formatter to leave those lines alone.

[row, column] Picks an Item, a Row, or a Column
-----------------------------------------------

.. code: lecture5/indexing_demo.py 18-22

.. code-block:: python

   # Indexing: [row, column]
   print(poses[2, 0])  # Row 2, column 0
   print(poses[-1])  # The last row
   print(poses[:, 0])  # Column 0: every x
   print(poses[1:3, :2])

.. output: indexing_demo.txt 3-7

.. code-block:: text

   2.0
   [2.5  1.5  1.11]
   [0.  1.  2.  2.5]
   [[1.  0. ]
    [2.  0.5]]

- ``poses[2, 0]``: one item, row 2 and column 0, counted from 0 as in a
  list.
- ``poses[-1]``: one index picks a whole row, here the last pose.
- ``poses[:, 0]``: ``:`` means every row, so this is column 0, every
  ``x``.
- ``poses[1:3, :2]``: a slice on each axis gives a smaller 2D array,
  rows 1 and 2, columns 0 and 1.

Slicing a 1D Array Works as for a List
--------------------------------------

.. code: lecture5/indexing_demo.py 24-27

.. code-block:: python

   # Slicing a 1D array works as for a list
   ranges = np.array([2.5, 0.9, 3.1, 1.2, 0.4, 2.2])
   print(ranges[1:4], ranges[::2])
   print(ranges[::-1])

.. output: indexing_demo.txt 8-9

.. code-block:: text

   [0.9 3.1 1.2] [2.5 3.1 0.4]
   [2.2 0.4 1.2 3.1 0.9 2.5]

- ``[start:stop:step]``, with the same rules as for a list (Lecture 3):
  ``stop`` is left out.
- ``ranges[::2]`` keeps every other reading. A scan with 360 readings
  becomes 180, half the data to process.
- ``ranges[::-1]`` gives the readings in reverse order.

A Slice of a List Is a Copy; a Slice of an Array Is a View
----------------------------------------------------------

.. code: lecture5/indexing_demo.py 29-37

.. code-block:: python

   # A slice of a list is a copy; a slice of an array is a view
   ranges_list = [2.5, 0.9, 3.1]
   part = ranges_list[0:2]
   part[0] = 99.0
   print(ranges_list)
   part = ranges[0:2]
   part[0] = 99.0
   print(ranges)
   print(part.base is ranges)

.. output: indexing_demo.txt 10-12

.. code-block:: text

   [2.5, 0.9, 3.1]
   [99.   0.9  3.1  1.2  0.4  2.2]
   True

- Changing the list slice leaves ``ranges_list`` as it was. Changing the
  array slice changes ``ranges``: ``99.`` is now its first item.
- A **view** is a new array object that uses the same numbers as
  another. ``.base`` is the array it views.
- NumPy's guide (`Copies and views
  <https://numpy.org/doc/1.26/user/basics.copies.html>`_): "basic
  indexing always creates views." A function that gets a slice can
  change the caller's array (Lecture 4, pass-by-assignment).

copy() Makes Its Own Numbers; reshape() Gives a New Shape
---------------------------------------------------------

.. code: lecture5/indexing_demo.py 38-46

.. code-block:: python

   safe = ranges[2:4].copy()
   safe[0] = -1.0
   print(ranges[2], safe.base is None)

   # reshape gives the same numbers a new shape
   xy = np.array([0.0, 0.0, 1.0, 0.0, 2.0, 0.5])
   pairs = xy.reshape(3, 2)
   print(pairs)
   print(xy.reshape(-1, 2).shape, pairs.base is xy)

.. output: indexing_demo.txt 13-17

.. code-block:: text

   3.1 True
   [[0.  0. ]
    [1.  0. ]
    [2.  0.5]]
   (3, 2) True

- ``.copy()`` makes an array with its own numbers: changing ``safe``
  leaves ``ranges[2]`` at 3.1, and ``safe.base`` is ``None``.
- ``xy`` holds ``x0, y0, x1, y1, x2, y2``. ``reshape(3, 2)`` reads the 6
  numbers in order into 3 rows of 2: one ``(x, y)`` point per row.
- ``-1`` lets NumPy compute that size: 6 items / 2 columns = 3 rows.
  Here ``reshape()`` returned a view. NumPy's guide: "The
  ``numpy.reshape`` function creates a view where possible or a copy
  otherwise."

float32 Is Less Precise than float64
------------------------------------

.. code: lecture5/dtypes_demo.py 5-10

.. code-block:: python
   :caption: From lecture5/dtypes_demo.py

   # A LaserScan's ranges are float32: 4 bytes, less precise
   r32 = np.array([0.1, 2.5], dtype=np.float32)
   print(r32.dtype, r32.itemsize)
   print(r32[0], float(r32[0]))
   r64 = r32.astype(np.float64)  # A converted copy
   print(r64.dtype, r64[0])

.. output: dtypes_demo.txt 1-3

.. code-block:: text

   float32 4
   0.1 0.10000000149011612
   float64 0.10000000149011612

- A ROS 2 ``LaserScan`` message stores its ranges as ``float32[]``: 4
  bytes per number, half of a ``float64``.
- ``float32`` cannot hold 0.1 exactly. It prints as ``0.1``, but as a
  Python ``float`` the value it holds is 0.10000000149011612.
- ``astype()`` makes a converted copy. Converting to ``float64`` keeps
  that value: the lost digits do not come back.

uint8 Pixels Wrap Around Past 255
---------------------------------

.. code: lecture5/dtypes_demo.py 12-15

.. code-block:: python

   # 8-bit pixel values: uint8 holds 0 to 255, and a sum wraps around
   pixels = np.array([200, 250], dtype=np.uint8)
   print(pixels + 100)
   print(pixels.astype(np.int64) + 100)

.. output: dtypes_demo.txt 4-5

.. code-block:: text

   [44 94]
   [300 350]

- ``uint8`` holds whole numbers from 0 to 255, one byte each: the pixel
  values of an 8-bit image.
- 200 + 100 = 300, which is 256 too many, so the item becomes
  300 − 256 = 44. Likewise 250 + 100 gives 350 − 256 = 94. NumPy reports
  no error.
- Convert first, with ``astype()``: as ``int64``, 200 + 100 gives 300.


Boolean Masks
====================================================

A Comparison Gives One Boolean per Reading
------------------------------------------

.. code: lecture5/masks_demo.py 5-9

.. code-block:: python
   :caption: From lecture5/masks_demo.py

   ranges = np.array([2.5, 0.9, 3.1, 0.4, 14.0, 0.7])  # Chosen
   close = ranges < 1.0
   print(close)
   print(ranges[close])
   print(close.sum(), np.flatnonzero(close))

.. output: masks_demo.txt 1-3

.. code-block:: text

   [False  True False  True False  True]
   [0.9 0.4 0.7]
   3 [1 3 5]

- ``ranges < 1.0`` compares every item and gives an array of ``bool``,
  the same shape: a **Boolean mask**.
- ``ranges[close]`` keeps the items where the mask is ``True``: 0.9,
  0.4, and 0.7.
- ``close.sum()`` counts the ``True`` items (each counts as 1).
  ``np.flatnonzero()`` gives their positions: 1, 3, and 5.

.. _l5-combine-tests:

Combine Tests with &, \|, and ~
-------------------------------

.. code: lecture5/masks_demo.py 11-15

.. code-block:: python

   # Combine tests with & (and), | (or), and ~ (not), in parentheses
   valid = (ranges >= 0.12) & (ranges <= 12.0)
   print(valid)
   print(ranges[valid & ~close])
   print(ranges[(ranges < 0.5) | (ranges > 12.0)])

.. output: masks_demo.txt 4-6

.. code-block:: text

   [ True  True  True  True False  True]
   [2.5 3.1]
   [ 0.4 14. ]

- ``&`` is "and", ``|`` is "or", ``~`` is "not", item by item. Each test
  goes in parentheses (`& Runs Before >=, So Each Test Needs
  Parentheses`_).
- ``valid``: from 0.12 to 12.0 m, the lidar of Lecture 2. Only 14.0
  fails.
- ``valid & ~close``: valid and not closer than 1.0 m, so 2.5 and 3.1.
- A mask selects a copy, not a view. NumPy's guide: "Advanced indexing,
  on the other hand, always creates copies."

and Does Not Work on Arrays
---------------------------

.. code: lecture5/errors/and_on_arrays.py all

.. code-block:: python
   :caption: lecture5/errors/and_on_arrays.py

   import numpy as np

   ranges = np.array([2.5, 0.9, 14.0])
   valid = ranges >= 0.12 and ranges <= 12.0
   print(valid)

.. output: err_and_on_arrays.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../and_on_arrays.py", line 4, in <module>
       valid = ranges >= 0.12 and ranges <= 12.0
               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   ValueError: The truth value of an array with more than one element is ambiguous. Use a.any() or a.all()

- ``and`` needs one ``True`` or ``False``. An array of 3 Booleans is not
  one answer, so NumPy refuses to pick one.
- Use ``&`` for item-by-item "and". ``.any()`` and ``.all()`` turn a
  mask into one answer: is any item ``True``, or are all?

& Runs Before >=, So Each Test Needs Parentheses
------------------------------------------------

.. code: lecture5/errors/mask_precedence.py all

.. code-block:: python
   :caption: lecture5/errors/mask_precedence.py

   import numpy as np

   ranges = np.array([2.5, 0.9, 14.0])
   valid = ranges >= 0.12 & ranges <= 12.0
   print(valid)

.. output: err_mask_precedence.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../mask_precedence.py", line 4, in <module>
       valid = ranges >= 0.12 & ranges <= 12.0
                         ~~~~~^~~~~~~~
   TypeError: ufunc 'bitwise_and' not supported for the input types, and the inputs could not be safely coerced to any supported types according to the casting rule ''safe''

- In Python's `precedence table
  <https://docs.python.org/3.12/reference/expressions.html#operator-precedence>`_,
  ``&`` binds more tightly than the comparisons. Python reads line 4 as
  ``ranges >= (0.12 & ranges) <= 12.0``.
- ``0.12 & ranges`` runs first. ``&`` on floats is not defined, so
  NumPy raises the ``TypeError``.
- The fix is the one in :ref:`l5-combine-tests`:
  ``(ranges >= 0.12) & (ranges <= 12.0)``.

A Mask Can Change Items, and np.where() Picks Item by Item
----------------------------------------------------------

.. code: lecture5/masks_demo.py 17-21

.. code-block:: python

   # Assign through a mask, in a copy
   capped = ranges.copy()
   capped[~valid] = 12.0
   print(capped)
   print(np.where(valid, ranges, np.nan))

.. output: masks_demo.txt 7-8

.. code-block:: text

   [ 2.5  0.9  3.1  0.4 12.   0.7]
   [2.5 0.9 3.1 0.4 nan 0.7]

- ``capped[~valid] = 12.0`` assigns 12.0 to every invalid item of the
  copy. ``ranges`` keeps its 14.0.
- ``np.where(cond, a, b)`` takes the item of ``a`` where ``cond`` is
  ``True``, and of ``b`` where it is ``False``. A number is used for
  every item.
- ``np.nan`` is "not a number", a float that marks a missing value.
  `A Laser Scan as an Array`_ meets it in the lecture's scan.

An Array of Positions Selects Items: The 3 Nearest
--------------------------------------------------

.. code: lecture5/masks_demo.py 23-26

.. code-block:: python

   # An array of positions also selects items: here, the 3 nearest
   order = np.argsort(ranges)
   print(order)
   print(ranges[order[:3]])

.. output: masks_demo.txt 9-10

.. code-block:: text

   [3 5 1 0 2 4]
   [0.4 0.7 0.9]

- ``np.argsort()`` returns the positions that would sort the array:
  ``ranges[3]`` = 0.4 is smallest, then ``ranges[5]`` = 0.7, then
  ``ranges[1]`` = 0.9.
- ``ranges[order[:3]]``: an array of positions as the index picks those
  items, in that order. Like a mask, it selects a copy.

Exercise 1 on the :doc:`exercises page <l5_exercises>` practices this
section and the one before it.


Vectorized Operations and Broadcasting
====================================================

A Universal Function Works on Every Item
----------------------------------------

.. code: lecture5/vectorized_demo.py 5-10

.. code-block:: python
   :caption: From lecture5/vectorized_demo.py

   # Element by element, with no loop (offsets chosen for this example)
   dx = np.array([3.0, 0.5, 1.0])  # m
   dy = np.array([4.0, 0.0, 2.0])  # m
   print(np.sqrt(dx**2 + dy**2))
   print(np.hypot(dx, dy))
   print(np.degrees(np.arctan2(dy, dx)))

.. output: vectorized_demo.txt 1-3

.. code-block:: text

   [5.         0.5        2.23606798]
   [5.         0.5        2.23606798]
   [53.13010235  0.         63.43494882]

- NumPy (`Universal functions
  <https://numpy.org/doc/1.26/reference/ufuncs.html>`_): "A universal
  function (or ufunc for short) is a function that operates on ndarrays
  in an element-by-element fashion".
- Three offsets at once: ``(3, 4)`` is 5.0 m away, √(3² + 4²) = 5.
  ``np.hypot(dx, dy)`` computes the same length in one call.
- ``np.arctan2(dy, dx)``, ``y`` first: the direction of ``(dx, dy)``
  from the x axis, in radians. ``np.degrees()`` converts.

A Reduction Turns Many Numbers into One
---------------------------------------

.. code: lecture5/vectorized_demo.py 12-14

.. code-block:: python

   # Reductions: one number from many
   ranges = np.array([2.5, 0.9, 3.1, 1.2])
   print(ranges.sum(), ranges.mean(), ranges.min(), ranges.argmin())

.. output: vectorized_demo.txt 4

.. code-block:: text

   7.7 1.925 0.9 1

- ``.sum()``: 2.5 + 0.9 + 3.1 + 1.2 = 7.7. ``.mean()``: 7.7 / 4 = 1.925.
- ``.min()`` gives the smallest value, 0.9. ``.argmin()`` gives its
  position, 1. ``.max()`` and ``.argmax()`` work the same way.
- The built-in ``min()`` and ``sum()`` also accept an array, but they
  take its items one at a time, and NumPy makes a ``numpy.float64``
  object for each. The methods work on the numbers directly.

axis Says Which Way a Reduction Runs
------------------------------------

.. code: lecture5/vectorized_demo.py 16-26

.. code-block:: python

   # axis: the direction a reduction runs in
   # fmt: off
   poses = np.array([  # x (m), y (m), theta (rad), as in indexing_demo.py
       [0.0, 0.0, 0.00],
       [1.0, 0.0, 0.00],
       [2.0, 0.5, 0.46],
       [2.5, 1.5, 1.11],
   ])
   # fmt: on
   print(poses.mean(axis=0))
   print(poses.max(axis=1))

.. output: vectorized_demo.txt 5-6

.. code-block:: text

   [1.375  0.5    0.3925]
   [0.  1.  2.  2.5]

.. figure:: /_static/images/l5/axis.svg
   :alt: A 4 by 3 grid of poses: rows 0.0 0.0 0.00; 1.0 0.0 0.00; 2.0 0.5 0.46; 2.5 1.5 1.11. Tag 1: an arrow runs down each column to a row of three means below the grid, 1.375, 0.5 and 0.3925 (axis=0). Tag 2: an arrow runs along each row to a column of four maxima at the right, 0.0, 1.0, 2.0 and 2.5 (axis=1).
   :width: 60%

   1: ``axis=0``, down each column. 2: ``axis=1``, along each row.

- ``axis=0`` runs over the rows and leaves one value per column: the
  mean ``x`` is (0.0 + 1.0 + 2.0 + 2.5) / 4 = 1.375.
- ``axis=1`` runs over the columns and leaves one value per row: the
  largest of each pose's three numbers.

np.cumsum() Adds Up Steps, np.diff() Takes Them Apart
-----------------------------------------------------

.. code: lecture5/vectorized_demo.py 28-31

.. code-block:: python

   # cumsum: running totals, here of the distances a wheel reports
   steps = np.array([0.1, 0.1, 0.2, 0.2])  # m per step, chosen
   print(np.cumsum(steps))
   print(np.diff(np.cumsum(steps)))

.. output: vectorized_demo.txt 7-8

.. code-block:: text

   [0.1 0.2 0.4 0.6]
   [0.1 0.2 0.2]

- ``np.cumsum()`` gives the running total: 0.1, then 0.1 + 0.1 = 0.2,
  then 0.2 + 0.2 = 0.4, then 0.4 + 0.2 = 0.6. That is the distance
  driven after each step.
- ``np.diff()`` gives the difference between neighbors: the steps again,
  without the first one. The result has one item fewer.
- Odometry adds up small steps in the same way. `Plotting with
  Matplotlib`_ builds a whole path with ``np.cumsum()``.

Broadcasting Uses a Number for Every Item
-----------------------------------------

.. code: lecture5/broadcasting_demo.py 5-7

.. code-block:: python
   :caption: From lecture5/broadcasting_demo.py

   # An array and a number: the number is used for every item
   ranges = np.array([2.5, 0.9, 3.1])
   print(ranges - 0.1)

.. output: broadcasting_demo.txt 1

.. code-block:: text

   [2.4 0.8 3. ]

- **Broadcasting** lets one operation combine arrays of different
  shapes. NumPy's guide (`Broadcasting
  <https://numpy.org/doc/1.26/user/basics.broadcasting.html>`_): "The
  term broadcasting describes how NumPy treats arrays with different
  shapes during arithmetic operations."
- The simplest case: ``ranges - 0.1`` uses the number 0.1 for every
  item, as ``ranges * 100`` did in `One Array Expression Replaces a Loop
  over a List`_.

Broadcasting Uses One Row for Every Row
---------------------------------------

.. code: lecture5/broadcasting_demo.py 9-15

.. code-block:: python

   # A (3, 2) array minus a (2,) array: the row is used for every row
   points = np.array([[1.0, 2.0], [3.0, 1.0], [4.0, 4.0]])  # x, y (m)
   robot = np.array([1.0, 1.0])  # Chosen for this example
   rel = points - robot
   print(rel.shape)
   print(rel)
   print(np.linalg.norm(rel, axis=1))

.. output: broadcasting_demo.txt 2-6

.. code-block:: text

   (3, 2)
   [[0. 1.]
    [2. 0.]
    [3. 3.]]
   [1.         2.         4.24264069]

- ``points`` has shape ``(3, 2)``, ``robot`` has ``(2,)``. NumPy uses
  the one row of ``robot`` for each of the 3 rows: each point relative
  to the robot.
- ``np.linalg.norm(rel, axis=1)``: the length of each row. ``(3, 3)`` is
  √(9 + 9) = 4.24 m from the robot.

The Broadcasting Rule Compares Shapes from the Right
----------------------------------------------------

NumPy: "When operating on two arrays, NumPy compares their shapes
element-wise. It starts with the trailing (i.e. rightmost) dimension and
works its way left. Two dimensions are compatible when they are equal,
or one of them is 1." The same page adds: "missing dimensions are
assumed to have size one." Each size of the result is the larger of the
two.

.. list-table::
   :header-rows: 1
   :widths: 25 25 25 25
   :class: compact-table

   * - Shapes
     - Rightmost
     - Next
     - Result
   * - ``(3, 2)`` and ``(2,)``
     - 2 and 2: equal
     - 3 and missing (1)
     - ``(3, 2)``
   * - ``(3, 1)`` and ``(4,)``
     - 1 and 4: one is 1
     - 3 and missing (1)
     - ``(3, 4)``
   * - ``(3,)`` and ``(2,)``
     - 3 and 2: neither
     -
     - ``ValueError``

A Column against a Row Gives a Table
------------------------------------

.. code: lecture5/broadcasting_demo.py 17-23

.. code-block:: python

   # A (3, 1) column against a (4,) row gives a (3, 4) table
   robots_x = np.array([[0.0], [2.5], [5.5]])  # 3 robots on a line (m)
   goals_x = np.array([1.0, 3.0, 4.0, 6.0])  # 4 goals on that line (m)
   dist = np.abs(robots_x - goals_x)
   print(dist.shape)
   print(dist)
   print(dist.argmin(axis=1))

.. output: broadcasting_demo.txt 7-11

.. code-block:: text

   (3, 4)
   [[1.  3.  4.  6. ]
    [1.5 0.5 1.5 3.5]
    [4.5 2.5 1.5 0.5]]
   [0 1 3]

.. figure:: /_static/images/l5/broadcasting.svg
   :alt: Three 3 by 4 grids. Tag 1, robots_x, shape (3, 1): the column 0.0, 2.5, 5.5, with faded copies of it in the other three columns. Tag 2, goals_x, shape (4,): the row 1.0, 3.0, 4.0, 6.0, with faded copies of it in the other two rows. A minus sign between them and an arrow to tag 3, dist, shape (3, 4), the absolute differences: rows 1.0 3.0 4.0 6.0; 1.5 0.5 1.5 3.5; 4.5 2.5 1.5 0.5.
   :width: 100%

   1: ``robots_x``, shape ``(3, 1)``: its one column is used for all 4
   columns (dashed copies). 2: ``goals_x``, shape ``(4,)``: its one row
   is used for all 3 rows (dashed copies). 3: ``dist``, shape
   ``(3, 4)``: ``np.abs()`` of tag 1 minus tag 2, item by item.

- 3 robots and 4 goals on one line, positions in meters (chosen for this
  example).
- Row ``r``, column ``g`` holds the distance from robot ``r`` to goal
  ``g``. Row 1, column 1: robot 1 at 2.5 m, goal 1 at 3.0 m, so 0.5 m.
- ``dist.argmin(axis=1)``: the nearest goal of each robot, 0, 1, and 3.
- NumPy does not store the dashed copies: it reads the same column or
  row again. No copies of the inputs are made.

Shapes That Do Not Match Raise ValueError
-----------------------------------------

.. code: lecture5/errors/broadcast_mismatch.py all

.. code-block:: python
   :caption: lecture5/errors/broadcast_mismatch.py

   import numpy as np

   ranges = np.array([2.5, 0.9, 3.1])
   offsets = np.array([0.1, 0.2])
   print(ranges - offsets)

.. output: err_broadcast_mismatch.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../broadcast_mismatch.py", line 5, in <module>
       print(ranges - offsets)
             ~~~~~~~^~~~~~~~~
   ValueError: operands could not be broadcast together with shapes (3,) (2,) 

- 3 and 2 are the rightmost sizes; they are not equal and neither is 1
  (the table in `The Broadcasting Rule Compares Shapes from the
  Right`_).
- When a shape error appears, print the ``.shape`` of each operand
  first.


Vectors, Rotation Matrices, and 2D Transforms
====================================================

The Vector from a Robot to Its Goal
-----------------------------------

.. code: lecture5/transforms_demo.py 1-11

.. code-block:: python
   :caption: From lecture5/transforms_demo.py

   """Vectors, rotation matrices, and 2D transforms."""

   import numpy as np
   from geometry import rot, transform

   # The vector from a robot to a goal (positions chosen, in m)
   robot = np.array([2.0, 1.0])
   goal = np.array([5.0, 5.0])
   d = goal - robot
   print(d, np.linalg.norm(d))
   print(np.degrees(np.arctan2(d[1], d[0])))

.. output: transforms_demo.txt 1-2

.. code-block:: text

   [3. 4.] 5.0
   53.13010235415598

- A 2D **vector** is an array of 2 numbers. ``goal - robot`` is the step
  from the robot to the goal: 3.0 m along x, 4.0 m along y.
- ``np.linalg.norm(d)``: its length, √(3² + 4²) = 5.0 m.
- ``np.arctan2(d[1], d[0])``: its direction, 53.13° counterclockwise
  from the x axis. It gives the right angle in all four quadrants;
  ``y / x`` alone cannot tell ``(1, 1)`` from ``(-1, -1)``.
- ``geometry.py`` sits in ``lecture5/`` next to the program, so
  ``from geometry import rot, transform`` finds it (Lecture 2, modules).

A Rotation Matrix Turns a Vector by an Angle
--------------------------------------------

.. code: lecture5/geometry.py all

.. code-block:: python
   :caption: lecture5/geometry.py

   """2D rotation matrices and homogeneous transforms."""

   import numpy as np


   def rot(theta):
       """Return the 2x2 matrix that rotates a vector by theta (rad)."""
       c, s = np.cos(theta), np.sin(theta)
       return np.array([[c, -s], [s, c]])


   def transform(x, y, theta):
       """Return the transform of a frame at (x, y), turned by theta."""
       T = np.eye(3)
       T[:2, :2] = rot(theta)
       T[:2, 2] = [x, y]
       return T

.. math::

   R(\theta) = \begin{bmatrix} \cos\theta & -\sin\theta \\
   \sin\theta & \cos\theta \end{bmatrix}

.. code: lecture5/transforms_demo.py 13-17

.. code-block:: python

   # Turn (1, 0) by 90 degrees: (0, 1)
   R = rot(np.radians(90))
   p = np.array([1.0, 0.0])
   print(R @ p)
   print(np.round(R @ p, 12))

.. output: transforms_demo.txt 3-4

.. code-block:: text

   [6.123234e-17 1.000000e+00]
   [0. 1.]

- :math:`R(\theta)` turns a vector by :math:`\theta` counterclockwise.
  ``@`` is the matrix product (Python's operator for it).
- ``(1, 0)``, turned by 90°, is ``(0, 1)``. The first item prints as
  ``6.123234e-17``: cos 90° in floating point is not exactly 0. Rounded
  to 12 places, it is ``0.``

@ Multiplies Matrices; * Multiplies Item by Item
------------------------------------------------

.. code: lecture5/transforms_demo.py 18

.. code-block:: python

   print(R * p)  # * multiplies item by item: not a rotation

.. output: transforms_demo.txt 5-6

.. code-block:: text

   [[ 6.123234e-17 -0.000000e+00]
    [ 1.000000e+00  0.000000e+00]]

- ``R * p`` broadcasts ``p`` to each row of ``R`` (shapes ``(2, 2)`` and
  ``(2,)``) and multiplies item by item. The result is a ``(2, 2)``
  array, not a vector.
- ``R @ p`` sums each row's products: row 0 gives
  cos 90° × 1 + (−sin 90°) × 0, row 1 gives sin 90° × 1 + cos 90° × 0.

.. warning::

   For a rotation or any other matrix product, write ``@``. ``*`` runs
   without an error and gives a different array.

Two Checks That a Matrix Is a Rotation
--------------------------------------

.. code: lecture5/transforms_demo.py 20-24

.. code-block:: python

   # What makes R a rotation
   R30 = rot(np.radians(30))
   print(np.round(R30, 3))
   print(np.allclose(R30.T @ R30, np.eye(2)))
   print(np.allclose(rot(0.2) @ rot(0.3), rot(0.5)))

.. output: transforms_demo.txt 7-10

.. code-block:: text

   [[ 0.866 -0.5  ]
    [ 0.5    0.866]]
   True
   True

- :math:`R(30°)`: cos 30° = 0.866 and sin 30° = 0.5, rounded.
- :math:`R^T R = I`: the transpose (``.T``) undoes the rotation, so
  turning back by :math:`\theta` needs no matrix inverse.
  ``np.allclose()`` compares within a small tolerance, because floats
  rarely match exactly.
- :math:`R(0.2)\,R(0.3) = R(0.5)`: two turns, one after the other, add
  their angles.

The World Frame and the Robot's Frame
-------------------------------------

.. figure:: /_static/images/l5/frames.png
   :alt: A plot in the world frame, x across and y up, in meters, at equal scales, from -0.5 to 4 and from -0.5 to 2.5. Tag 1: the world frame at the origin, two black arrows labeled x, pointing right, and y, pointing up. Tag 2: a gray dashed arrow, t, from the origin to the robot at (2.0, 1.0). Tag 3: the robot's frame at (2.0, 1.0), two blue arrows labeled x and y, turned 30 degrees counterclockwise: x points up and to the right, y up and to the left. Tag 4: an orange dot, the point p, at about (3.3, 1.75), on the line of the robot's x axis.
   :width: 70%

   1: the world frame, fixed in the room. 2: ``t``, from the world
   origin to the robot, (2.0, 1.0) m. 3: the robot's frame: x straight
   ahead, y to its left, turned θ = 30°. 4: point ``p``, 1.5 m straight
   ahead of the robot. Made by ``lecture5/plot_frames.py``.

- A **frame** is an origin and two axes. The robot's pose
  ``(2.0, 1.0, 30°)`` (chosen) places its frame in the world frame.
- `REP 103 <https://www.ros.org/reps/rep-0103.html>`_ for a robot's
  body: "x forward, y left, z up". Angles are counterclockwise, from x.

Step by Step: Point p in the World Frame
----------------------------------------

1. In the robot's frame, ``p`` = (1.5, 0.0): 1.5 m ahead.
2. Turn it by θ = 30°: (1.5 × 0.866, 1.5 × 0.5) = (1.299, 0.75).
3. Add ``t`` = (2.0, 1.0): (1.299 + 2.0, 0.75 + 1.0) = (3.299, 1.75).

.. math::

   p_\text{world} = R(\theta)\, p_\text{robot} + t, \qquad
   p_\text{robot} = R(\theta)^T (p_\text{world} - t)

Back to the robot's frame: subtract ``t``, then turn by −θ, which is
:math:`R^T`.

.. code: lecture5/transforms_demo.py 26-32

.. code-block:: python

   # From the robot's frame to the world frame
   pose = (2.0, 1.0, np.radians(30))  # x (m), y (m), theta: chosen
   t = np.array(pose[:2])
   p_robot = np.array([1.5, 0.0])  # 1.5 m straight ahead of the robot
   p_world = rot(pose[2]) @ p_robot + t
   print(np.round(p_world, 3))
   print(np.round(rot(pose[2]).T @ (p_world - t), 3))  # And back

.. output: transforms_demo.txt 11-12

.. code-block:: text

   [3.299 1.75 ]
   [1.5 0. ]

Many Points at Once: One Row per Point
--------------------------------------

.. code: lecture5/transforms_demo.py 34-36

.. code-block:: python

   # Many points at once: one row per point
   pts = np.array([[1.5, 0.0], [0.0, 1.0], [1.0, -1.0]])
   print(np.round(pts @ rot(pose[2]).T + t, 3))

.. output: transforms_demo.txt 13-15

.. code-block:: text

   [[3.299 1.75 ]
    [1.5   1.866]
    [3.366 0.634]]

- ``pts`` has one point per row, shape ``(3, 2)``. ``pts @ R.T`` turns
  every row: for one row, :math:`p^T R^T = (R\,p)^T`.
- ``+ t`` broadcasts ``t`` to every row (`Broadcasting Uses One Row for
  Every Row`_).
- Row 1: ``(0, 1)`` is 1 m to the robot's left. Turned,
  (0 − 0.5, 0.866) = (−0.5, 0.866); plus ``t``: (1.5, 1.866).
- A scan gives hundreds of points; this one line moves all of them.

A Homogeneous Transform Holds R and t in One Matrix
---------------------------------------------------

.. math::

   T = \begin{bmatrix} R(\theta) & t \\ 0 \;\; 0 & 1 \end{bmatrix},
   \qquad
   T \begin{bmatrix} p \\ 1 \end{bmatrix} =
   \begin{bmatrix} R(\theta)\, p + t \\ 1 \end{bmatrix}

- ``transform()`` in ``geometry.py`` builds :math:`T`: ``T[:2, :2]`` is
  the top-left 2 by 2 block, set to :math:`R(\theta)`; ``T[:2, 2]`` is
  the top of the last column, set to ``t``.
- The point gets a third item, 1. Then one matrix product does the
  rotation and the translation.

.. code: lecture5/transforms_demo.py 38-41

.. code-block:: python

   # A homogeneous transform: rotation and translation in one matrix
   T_world_robot = transform(*pose)
   print(np.round(T_world_robot, 3))
   print(np.round(T_world_robot @ [1.5, 0.0, 1.0], 3))

.. output: transforms_demo.txt 16-19

.. code-block:: text

   [[ 0.866 -0.5    2.   ]
    [ 0.5    0.866  1.   ]
    [ 0.     0.     1.   ]]
   [3.299 1.75  1.   ]

- ``transform(*pose)`` unpacks the tuple into three arguments
  (Lecture 4).
- ``T_world_robot @ [1.5, 0.0, 1.0]`` gives (3.299, 1.75), the result of
  `Step by Step: Point p in the World Frame`_.

Composing Transforms Is a Matrix Product
----------------------------------------

.. code: lecture5/transforms_demo.py 43-47

.. code-block:: python

   # Composing: if the lidar were 0.2 m ahead of the robot's center
   T_robot_lidar = transform(0.2, 0.0, 0.0)
   T_world_lidar = T_world_robot @ T_robot_lidar
   print(np.round(T_world_lidar @ [1.3, 0.0, 1.0], 3))
   print(np.round(np.linalg.inv(T_world_robot) @ [3.299, 1.75, 1.0], 3))

.. output: transforms_demo.txt 20-21

.. code-block:: text

   [3.299 1.75  1.   ]
   [1.5 0.  1. ]

- A what-if: if the lidar were 0.2 m ahead of the robot's center, a
  reading 1.3 m ahead of the lidar would be 0.2 + 1.3 = 1.5 m ahead of
  the center. The product gives the same world point, (3.299, 1.75).
- ``T_world_robot @ T_robot_lidar``: the names say which frame each
  matrix goes from and to. The inner names match.
- ``np.linalg.inv(T)`` goes the other way: world to robot. Lecture 11
  (TF2) keeps transforms like these for every frame of a robot.

Exercise 2 on the :doc:`exercises page <l5_exercises>` practices this
section and broadcasting.


A Laser Scan as an Array
====================================================

The Fields of a LaserScan Message
---------------------------------

.. output: laserscan_fields.txt all

.. code-block:: text
   :caption: The fields of sensor_msgs/msg/LaserScan.msg (ROS 2 Jazzy), comments left out

   std_msgs/Header header
   float32 angle_min
   float32 angle_max
   float32 angle_increment
   float32 time_increment
   float32 scan_time
   float32 range_min
   float32 range_max
   float32[] ranges
   float32[] intensities

- One message is one sweep of a planar lidar.
- Its comment on angles: "angles are measured around the positive Z
  axis (counterclockwise, if Z is up) with zero angle being forward
  along the x axis".
- ``angle_min``, ``angle_increment``: the angle of the first reading,
  and the angle between two neighbors, in radians.
- ``range_min``, ``range_max``: the sensor's limits, in meters.
- ``ranges``: one distance per reading. Its comment: "values <
  range_min or > range_max should be discarded".

Reading i Is at angle_min + i × angle_increment
-----------------------------------------------

.. figure:: /_static/images/l5/scan_layout.svg
   :alt: Not to scale. Tag 1: the robot at the center, a triangle facing right, with its x axis pointing right and its y axis pointing up. Four dashed rays leave the robot, each labeled with its index: 0 pointing backward (left), 90 pointing down (the robot's right), 180 pointing ahead (right), 270 pointing up (the robot's left). Tag 2: an arc from the x axis, counterclockwise, to one more ray at about 40 degrees, the angle theta_i of reading i. Tag 3: that ray, whose length is the range r_i. Tag 4: a dot at its end, the point the reading hit.
   :width: 50%

   Not to scale. 1: the robot, facing its x axis. 2: the angle
   :math:`\theta_i` of reading ``i``, from the x axis. 3: its range
   :math:`r_i`, along the ray. 4: the point it hit,
   :math:`(r_i \cos\theta_i,\; r_i \sin\theta_i)`. The number at each
   dashed ray is its index ``i``.

In this lecture's scan, ``angle_min`` is −180°, ``angle_increment`` is
1°, and there are 360 readings. So reading 0 points backward, 90 to the
robot's right (−90°), 180 straight ahead (−180° + 180 × 1° = 0°), and
270 to its left.

This Lecture's Scan Is Synthetic
--------------------------------

.. code: lecture5/scan_data.py 23-30

.. code-block:: python
   :caption: The values in lecture5/scan_data.py, all chosen for this lecture

   ROOM_W_M, ROOM_H_M = 8.0, 6.0  # Walls at x = 0, x = 8, y = 0, y = 6
   DOOR_X_M = (5.0, 6.0)  # A doorway in the wall y = 6
   PILLAR_XY_M = np.array([4.5, 2.0])  # A round pillar: center ...
   PILLAR_R_M = 0.3  # ... and radius
   POSE = (2.0, 1.0, np.radians(30.0))  # Robot x (m), y (m), heading (rad)
   NOISE_M = 0.01  # Standard deviation of the noise on each range
   N_ERRORS = 3  # Number of NaN readings
   N_RAYS = 360

- ``make_scan()`` computes each reading's distance to the walls of an
  8 m by 6 m room and to a round pillar, adds noise, and returns an
  object with the field names of a ``LaserScan``
  (``types.SimpleNamespace``: an object whose attributes are the keyword
  arguments you give it).
- The robot stands at the pose of `The World Frame and the Robot's
  Frame`_: (2.0, 1.0, 30°).
- Rays that leave through the doorway get no return: ``inf``. Three
  readings, picked at random with seed 605, are ``nan``. REP 117 uses
  both values for these cases (`The Valid Mask Drops NaN and inf`_).
- The rest of ``scan_data.py`` computes the distances with the tools of
  this lecture: masks, ``np.where()``, broadcasting, and matrix
  products. Read it after class.

np.asarray() Turns the Ranges into a float32 Array
--------------------------------------------------

.. code: lecture5/scan_demo.py 1-13

.. code-block:: python
   :caption: From lecture5/scan_demo.py

   """A laser scan as an array: invalid readings, nearest obstacle."""

   import numpy as np
   from geometry import rot
   from scan_data import POSE, make_scan

   scan = make_scan()  # Synthetic, with the fields of a LaserScan message
   ranges = np.asarray(scan.ranges)
   print(type(scan.ranges).__name__, ranges.dtype, ranges.shape)

   # The angle of each reading, in the robot's frame
   angles = scan.angle_min + np.arange(ranges.size) * scan.angle_increment
   print(np.degrees(angles[[0, 1, 180, 359]]))

.. output: scan_demo.txt 1-2

.. code-block:: text

   array float32 (360,)
   [-180. -179.    0.  179.]

- As in rclpy (`In rclpy, msg.ranges Is an array.array`_),
  ``scan.ranges`` is an ``array.array`` of 4-byte floats.
  ``np.asarray()`` turns it into a ``float32`` NumPy array of 360 items.
- One line computes all 360 angles: ``np.arange(360)`` times the
  increment, plus ``angle_min``. The output checks indexes 0, 1, 180,
  and 359, in degrees.

One NaN Makes min() Return NaN
------------------------------

.. code: lecture5/scan_demo.py 15-16

.. code-block:: python

   # min() of an array with a NaN is NaN
   print(ranges.min(), ranges.argmin())

.. output: scan_demo.txt 3

.. code-block:: text

   nan 50

.. code: lecture5/errors/nan_min.py all

.. code-block:: python
   :caption: lecture5/errors/nan_min.py

   import numpy as np

   ranges = np.array([2.5, np.nan, 0.9, 3.1])
   print(ranges.min(), ranges.argmin())
   print(min([2.5, np.nan, 0.9]), min([np.nan, 2.5, 0.9]))

.. output: err_nan_min.txt all

.. code-block:: text

   nan 1
   0.9 nan

- One NaN reading, at index 50, makes ``.min()`` return ``nan``, and
  ``.argmin()`` returns its index.
- Every ``<``, ``<=``, ``>``, ``>=``, and ``==`` with NaN is ``False``,
  even ``np.nan == np.nan``; ``!=`` is ``True``. So the built-in
  ``min()`` keeps or skips a NaN by its place in the list: 0.9 in one
  order, ``nan`` in the other.

The Valid Mask Drops NaN and inf
--------------------------------

.. code: lecture5/scan_demo.py 18-21

.. code-block:: python

   # The invalid readings, by kind
   print(np.isnan(ranges).sum(), np.isposinf(ranges).sum())
   valid = (ranges >= scan.range_min) & (ranges <= scan.range_max)
   print(valid.sum(), "valid of", ranges.size)

.. output: scan_demo.txt 4-5

.. code-block:: text

   3 8
   349 valid of 360

- `REP 117 <https://www.ros.org/reps/rep-0117.html>`_: "Detections that
  are too close to the sensor to quantify shall be represented by -Inf.
  Erroneous detections shall be represented by quiet (non-signaling)
  NaNs. Finally, out of range detections will be represented by +Inf."
- 3 NaN and 8 inf readings: 360 − 3 − 8 = 349 valid.
- REP 117 on the check
  ``if(minimum_range < value && value < maximum_range){}``: "Because
  NaNs compares will always return false, -Inf will always be less than
  a valid, numerical minimum range, and +Inf will always be greater than
  a valid, numerical maximum range, this commonly used check will
  successfully reject all errors cases."
- The opposite check, "invalid if ``value < min`` or ``value > max``",
  lets NaN through: REP 117 warns that "NaNs will slip through this
  check".

The Nearest Valid Reading: Invalid Ones Become inf
--------------------------------------------------

.. code: lecture5/scan_demo.py 23-26

.. code-block:: python

   # The nearest valid reading: invalid ones become inf, then argmin
   i = np.where(valid, ranges, np.inf).argmin()
   deg = np.degrees(angles[i])
   print(f"nearest: index {i}, {ranges[i]:.3f} m at {deg:.0f} deg")

.. output: scan_demo.txt 6

.. code-block:: text

   nearest: index 65, 0.980 m at -115 deg

- ``np.where(valid, ranges, np.inf)`` replaces every invalid reading by
  ``inf``, which is never the smallest. ``.argmin()`` then gives the
  nearest valid one.
- Reading 65 is at −180° + 65 × 1° = −115°: behind the robot, to its
  right.
- Is 0.980 m right? −115° in the robot's frame is −115° + 30° = −85° in
  the room, toward the wall y = 0, which is 1.0 m from the robot. Along
  that ray, the wall is 1.0 / sin 85° = 1.004 m away. The noise made
  this reading 0.023 m shorter, the shortest of its neighbors.

Only the Readings in Front of the Robot
---------------------------------------

.. code: lecture5/scan_demo.py 28-32

.. code-block:: python

   # Only the front: within 30 degrees of straight ahead
   front = valid & (np.abs(angles) <= np.radians(30))
   j = np.where(front, ranges, np.inf).argmin()
   deg = np.degrees(angles[j])
   print(f"front: index {j}, {ranges[j]:.3f} m at {deg:.0f} deg")

.. output: scan_demo.txt 7

.. code-block:: text

   front: index 171, 2.379 m at -9 deg

- ``np.abs(angles) <= np.radians(30)``: 60 readings, from −30° to 29°.
  A robot that drives ahead cares about these first.
- Reading 210 should be at 30°, but its computed angle is
  30.000000000000018°, a little over 30° in floating point, so the test
  leaves it out. A test at the exact edge of a range of floats can go
  either way.
- The nearest one, 2.379 m at −9°, is the pillar. Its center is 2.69 m
  from the robot at −8.2° (robot frame), so its surface is about
  2.69 − 0.3 = 2.39 m away.
- The 8 ``inf`` readings (the doorway, 22° to 29°) are in this sector
  too; the mask leaves them out.

From Readings to Points, and Then to the World Frame
----------------------------------------------------

.. code: lecture5/scan_demo.py 34-43

.. code-block:: python

   # Each valid reading as a point (x, y) in the robot's frame
   x = ranges[valid] * np.cos(angles[valid])
   y = ranges[valid] * np.sin(angles[valid])
   points = np.column_stack([x, y])
   print(points.shape, points.dtype)

   # The nearest point, in the world frame: on the wall y = 0?
   x0, y0, theta = POSE
   p = ranges[i] * np.array([np.cos(angles[i]), np.sin(angles[i])])
   print(np.round(rot(theta) @ p + [x0, y0], 3))

.. output: scan_demo.txt 8-9

.. code-block:: text

   (349, 2) float64
   [2.085 0.023]

- (r cos θ, r sin θ) for all 349 valid readings at once;
  ``np.column_stack()`` makes one row per point.
- ``float64``: a ``float32`` array times a ``float64`` array gives a
  ``float64`` array. (A ``float32`` array times one ``np.float64``
  number stays ``float32`` in NumPy 1.26.)
- The nearest reading, moved to the world frame as in `Step by Step:
  Point p in the World Frame`_, lands at y = 0.023 m: on the wall y = 0,
  off by the same 0.023 m of noise.

In rclpy, msg.ranges Is an array.array
--------------------------------------

This program needs ROS 2 (Lecture 8).

.. code: lecture5/ros_scan_check.py all

.. code-block:: python
   :caption: lecture5/ros_scan_check.py

   """In rclpy, LaserScan.ranges is an array.array of float32.

   Needs ROS 2 Jazzy: source /opt/ros/jazzy/setup.bash first (Lecture 8).
   """

   import numpy as np
   from sensor_msgs.msg import LaserScan

   msg = LaserScan()
   msg.ranges = [2.5, 0.9, float("inf")]
   print(type(msg.ranges).__name__, msg.ranges.typecode)
   ranges = np.asarray(msg.ranges)
   print(ranges, ranges.dtype)

.. code-block:: bash

   source /opt/ros/jazzy/setup.bash
   uv run lecture5/ros_scan_check.py

.. output: ros_scan_check.txt all

.. code-block:: text

   array f
   [2.5 0.9 inf] float32

- rclpy stores a ``float32[]`` field as an ``array.array`` with type
  code ``'f'`` (4-byte floats), even when you assign a list.
- In a scan callback (Lecture 8), ``np.asarray(msg.ranges)`` gives the
  ``float32`` array this section used.

Exercise 3 on the :doc:`exercises page <l5_exercises>` processes this
scan.


Plotting with Matplotlib
====================================================

A Figure Holds Axes, and Axes Hold the Plot
-------------------------------------------

Matplotlib's `Quick start guide
<https://matplotlib.org/3.11.2/users/explain/quick_start.html>`_:
"Matplotlib graphs your data on Figures (e.g., windows, Jupyter
widgets, etc.), each of which can contain one or more Axes, an area
where points can be specified in terms of x-y coordinates". Its first
example:

.. code-block:: python

   fig, ax = plt.subplots()             # Create a figure containing a single Axes.
   ax.plot([1, 2, 3, 4], [1, 4, 2, 3])  # Plot some data on the Axes.
   plt.show()                           # Show the figure.

- ``import matplotlib.pyplot as plt``, then ``plt.subplots()`` makes a
  **Figure** and one **Axes**: the box with the x and y axes.
- You call methods on ``ax``: ``ax.plot()``, ``ax.set_xlabel()``. The
  guide calls this the explicit, or object-oriented, interface. The
  course uses it.
- Each plot here takes NumPy arrays, made with what the earlier sections
  taught.

A Path Is a Running Sum of Small Steps
--------------------------------------

.. code: lecture5/plot_trajectory.py 1-21

.. code-block:: python
   :caption: From lecture5/plot_trajectory.py

   """Plot a robot's path and its GPS fixes, seen from above."""

   import matplotlib.pyplot as plt
   import numpy as np
   from plots import BLUE, ORANGE, save, tag

   # The path: add up small steps of constant speed (values chosen)
   dt = 0.1  # s
   t = np.arange(0.0, 15.0, dt)  # 150 times
   v = 0.5  # m/s
   omega = np.where((t >= 5.0) & (t < 9.0), 0.4, 0.0)  # rad/s
   theta = np.cumsum(omega * dt)  # Heading after each step
   x = np.cumsum(v * np.cos(theta) * dt)
   y = np.cumsum(v * np.sin(theta) * dt)
   print(f"end: x = {x[-1]:.2f} m, y = {y[-1]:.2f} m")

   # A GPS fix once a second, with noise (synthetic, seeded)
   rng = np.random.default_rng(605)
   gps = np.column_stack([x[::10], y[::10]])
   gps = gps + rng.normal(0.0, 0.15, gps.shape)
   print(gps.shape)

.. output: plot_trajectory.txt 3-4

.. code-block:: text

   end: x = 3.64 m, y = 4.31 m
   (15, 2)

- The program's first two lines of output, a warning about Axes3D, are
  left out here: see `A Leftover from apt: Matplotlib's 3D Tools`_.
- 150 steps of 0.1 s at 0.5 m/s. From 5 s to 9 s the robot turns left
  at 0.4 rad/s: 0.4 × 4 = 1.6 rad, about 92°. The turn's radius is
  0.5 / 0.4 = 1.25 m.
- ``np.cumsum()`` adds up the heading changes, then the x and y steps.
  ``x[::10]``: one position per second, to which the noise is added.

Drawing the Path and the GPS Fixes
----------------------------------

.. code: lecture5/plot_trajectory.py 23-35

.. code-block:: python

   plt.rcParams["font.size"] = 9  # Points, at the size the slide shows
   fig, ax = plt.subplots(figsize=(3.6, 2.6), layout="constrained")
   ax.plot(x, y, color=BLUE, linewidth=2)
   ax.plot(gps[:, 0], gps[:, 1], "o", color=ORANGE, markersize=4)
   ax.plot(x[0], y[0], "s", color="black", markersize=5)
   ax.set_xlabel("x (m)")
   ax.set_ylabel("y (m)")
   ax.set_aspect("equal")
   ax.grid(alpha=0.3)
   tag(ax, x[30], y[30] + 0.45, 1)
   tag(ax, gps[13, 0] - 0.5, gps[13, 1], 2)
   tag(ax, x[0], y[0] + 0.45, 3)
   save(fig, "trajectory")

- ``figsize`` is in inches: 3.6 in is the width of the figure on the
  slide, so 9-point text stays 9 points.
- ``"o"`` draws markers with no line; ``"s"`` draws a square.
  ``set_aspect("equal")``: 1 m is the same length on both axes, so the
  turn has its true shape.
- ``tag()`` and ``save()`` come from ``lecture5/plots.py`` (`Saving a
  Figure, or Showing It in a Window`_).

.. figure:: /_static/images/l5/trajectory.png
   :alt: A plot with x in meters across and y in meters up, at equal scales. Tag 1: a blue line, the path, runs right along y = 0 from x = 0 to about 2.5 m, curves left through a quarter turn, and runs up near x = 3.7 m to y = 4.3 m. Tag 2: orange dots, one GPS fix per second, scattered within about 0.3 m of the line. Tag 3: a black square at the start, near (0, 0).
   :width: 60%

   1: the path (blue line). 2: GPS fixes, one per second, with noise
   (orange dots; synthetic, 0.15 m standard deviation, seed 605). 3: the
   first position, after one step: (0.05, 0.0) m (black square). A top
   view: both axes are positions, in meters.

Drawing the Scan from Above
---------------------------

.. code: lecture5/plot_scan.py 1-31

.. code-block:: python
   :caption: From lecture5/plot_scan.py

   """Plot the scan of scan_data.py: a top view, and range against angle."""

   import matplotlib.pyplot as plt
   import numpy as np
   from plots import BLUE, ORANGE, save, tag
   from scan_data import make_scan

   scan = make_scan()
   ranges = np.asarray(scan.ranges)
   angles = scan.angle_min + np.arange(ranges.size) * scan.angle_increment
   valid = (ranges >= scan.range_min) & (ranges <= scan.range_max)
   i = np.where(valid, ranges, np.inf).argmin()  # The nearest reading
   x = ranges * np.cos(angles)  # Robot frame; NaN and inf stay NaN, inf
   y = ranges * np.sin(angles)

   plt.rcParams["font.size"] = 9

   # 1. Top view, in the robot's frame: x straight ahead
   fig, ax = plt.subplots(figsize=(3.6, 2.9), layout="constrained")
   ax.plot(x[valid], y[valid], ".", color=BLUE, markersize=3)
   ax.plot(x[i], y[i], "o", color=ORANGE, markersize=6)
   ax.plot(0, 0, ">", color="black", markersize=7)  # The robot, facing +x
   ax.set_xlabel("x, robot frame (m)")
   ax.set_ylabel("y, robot frame (m)")
   ax.set_aspect("equal")
   ax.grid(alpha=0.3)
   tag(ax, x[i] - 0.6, y[i] - 0.6, 1)
   tag(ax, x[171] + 0.1, y[171] - 0.6, 2)
   tag(ax, 5.8, 3.2, 3)
   tag(ax, 0.0, 0.7, 4)
   save(fig, "scan_top")

- Lines 8 to 14 compute ``ranges``, ``angles``, ``valid``, the nearest
  index ``i``, and ``x``, ``y``, as ``scan_demo.py`` does.
- ``"."`` draws small dots: one per valid reading. ``">"`` is a triangle
  that points along +x: the robot faces that way.

.. figure:: /_static/images/l5/scan_top.png
   :alt: A plot of the scan from above, in the robot's frame: x straight ahead of the robot and y to its left, in meters, at equal scales. Blue dots trace the walls of a rectangle turned about 30 degrees clockwise, from about x = -2.2 to 7.7 m and y = -3.8 to 5.3 m. Tag 4: the robot, a black triangle at (0, 0) pointing toward +x. Tag 1: an orange dot, the nearest valid reading, at about (-0.4, -0.9), on the wall behind the robot to its right. Tag 2: a short arc of dots near (2.5, -0.4), the pillar. Tag 3: a gap with no dots in the far wall, near (5.5, 2.6), the doorway.
   :width: 60%

   1: the nearest valid reading (orange). 2: the pillar. 3: the doorway:
   no points. 4: the robot, at (0, 0), facing +x. The room looks turned
   by −30° because the robot is turned by +30° in the room.

From the Top View to a Graph of Range against Angle
---------------------------------------------------

- The top view puts each reading where it hit: x = r cos θ,
  y = r sin θ, in meters. It is a map of the room.
- The next graph puts each reading at (θ, r): angle across, range up. It
  is not a map: a straight wall becomes a curve, and the two axes have
  different units.

The nearest reading (tag 1) in both pictures:

1. Reading 65: θ = −115°, r = 0.980 m.
2. Top view: (0.9805 cos(−115°), 0.9805 sin(−115°)) = (−0.414, −0.889) m,
   behind the robot and to its right (0.980 is 0.9805 rounded).
3. Graph: the point at −115 across (the angle, in degrees) and 0.980 up
   (the range, in meters).

In the graph, the pillar is a dip (tag 2), and the doorway a gap with
no line (tag 3).

The Same Scan as a Graph: Range against Angle
---------------------------------------------

.. code: lecture5/plot_scan.py 33-45

.. code-block:: python

   # 2. Range against angle: a graph, not a map
   fig, ax = plt.subplots(figsize=(3.6, 2.4), layout="constrained")
   deg = np.degrees(angles)
   ax.plot(deg, ranges, color=BLUE, linewidth=1.5)
   ax.plot(deg[i], ranges[i], "o", color=ORANGE, markersize=6)
   ax.set_xlabel("angle (deg)")
   ax.set_ylabel("range (m)")
   ax.set_xticks(np.arange(-180, 181, 60))
   ax.grid(alpha=0.3)
   tag(ax, deg[i], ranges[i] + 0.7, 1)
   tag(ax, -9.0, 1.6, 2)
   tag(ax, 36.0, 7.0, 3)
   save(fig, "scan_graph")

- ``ranges`` goes in whole, NaN and inf included. Matplotlib draws no
  line through them: the line breaks there.
- ``ax.set_xticks()`` puts a tick every 60°, from −180° to 180°.

.. figure:: /_static/images/l5/scan_graph.png
   :alt: A graph, not a map: range in meters up, from 0.6 to 8, against angle in degrees across, from -180 to 180. A blue line starts near 2 m at -180 degrees, falls to its lowest point, about 1 m, near -120 degrees, rises to about 6 m near -40 degrees, drops in a narrow dip to about 2.4 m between about -15 and -2 degrees (tag 2, the pillar), peaks near 7.8 m at about 10 degrees, breaks off between about 20 and 30 degrees (tag 3, the doorway), falls to about 5 m near 60 degrees, has a corner near 5.4 m at 80 degrees, and falls to about 2 m from 140 to 180 degrees. Tag 1: an orange dot at the lowest point, -115 degrees and 0.98 m.
   :width: 60%

   A graph, not a map: each wall is a curve. 1: the nearest valid
   reading, −115° (orange). 2: the pillar, a dip near −9°. 3: the
   doorway, no line from 22° to 29° (``inf``). The NaN readings leave
   one-reading gaps at −130°, −67°, and 38°.

Saving a Figure, or Showing It in a Window
------------------------------------------

.. code: lecture5/plots.py all

.. code-block:: python
   :caption: lecture5/plots.py

   """Helpers shared by the Lecture 5 plotting programs."""

   from pathlib import Path

   PLOTS = Path(__file__).parent / "plots"  # Where the programs save PNGs
   BLUE, ORANGE, GRAY = "#2a78d6", "#eb6834", "#777777"  # Marks
   BLUE_TEXT = "#2f6fb5"  # For text: 5.17:1 on white (BLUE is 4.42:1)


   def tag(ax, x, y, n):
       """Draw a numbered tag (white on a dark circle) at (x, y)."""
       ax.annotate(
           str(n),
           (x, y),
           ha="center",
           va="center",
           color="white",
           fontsize=8,
           fontweight="bold",
           bbox={"boxstyle": "circle,pad=0.2", "fc": "#1b1b1b", "lw": 0},
       )


   def save(fig, name):
       """Save fig as plots/<name>.png, 300 dots per inch."""
       PLOTS.mkdir(exist_ok=True)
       fig.savefig(PLOTS / f"{name}.png", dpi=300)
       print(f"saved plots/{name}.png")

.. code-block:: bash

   uv run lecture5/plot_scan.py

.. output: plot_scan.txt all short

.. code-block:: text

   .../site-packages/matplotlib/projections/__init__.py:63: UserWarning: Unable to import Axes3D. This may be due to multiple versions of Matplotlib being installed (e.g. as a system package and as a pip package). As a result, the 3D projection is not available.
     warnings.warn("Unable to import Axes3D. This may be due to multiple versions of "
   saved plots/scan_top.png
   saved plots/scan_graph.png

- ``tag()`` draws a numbered circle with ``ax.annotate()``. The slides
  use numbered tags with a key instead of words in the figure.
- ``fig.savefig()`` writes a PNG file. 300 dots per inch × 3.6 in = 1080
  pixels across, the width of every plot in this lecture.
- ``plt.show()`` opens a window instead and waits until you close it.
  The course programs only save, so they also run where no screen is
  attached.
- The first two lines, a ``UserWarning`` about Axes3D, appear on a PC
  with apt's ``python3-matplotlib``: see `A Leftover from apt:
  Matplotlib's 3D Tools`_.


Installing NumPy next to ROS 2
====================================================

uv Adds Matplotlib to the Course Project
----------------------------------------

The course repository lists Matplotlib as a dependency. It was added
with one command, from the root of the repository:

.. code-block:: bash

   uv add matplotlib

.. output: uv_add.txt all

.. code-block:: text

   Resolved 18 packages in 5ms
   Installed 9 packages in 15ms
    + contourpy==1.3.3
    + cycler==0.12.1
    + fonttools==4.66.1
    + kiwisolver==1.5.1
    + matplotlib==3.11.2
    + pillow==12.3.0
    + pyparsing==3.3.3
    + python-dateutil==2.9.0.post0
    + six==1.17.0

``pyproject.toml`` after it:

.. code-block:: toml

   [project]
   name = "enpm605-spring-2027"
   version = "0.1.0"
   requires-python = "==3.12.*"
   dependencies = [
       "matplotlib>=3.11.2",
       "numpy<2",
   ]

- uv's guide (`Managing dependencies
  <https://docs.astral.sh/uv/concepts/projects/dependencies/>`_): "An
  entry will be added in the ``project.dependencies`` field", with a
  lower bound, here ``>=3.11.2``, for the newest version that fits.
- ``uv.lock`` records the exact version of each package, and ``.venv``
  gets them: Matplotlib 3.11.2 and 8 packages it needs, directly or
  through another package.
- After you pull the course repository, ``uv sync`` installs what
  ``uv.lock`` lists. pandas (Lecture 13) comes the same way.

.. warning::

   ``numpy<2`` stays. ROS 2 Jazzy is built against Ubuntu's NumPy
   1.26.4 (:doc:`ROS 2 setup </setup/ros_setup>`), and NumPy's page for
   package authors (`For downstream package authors
   <https://numpy.org/doc/stable/dev/depending_on_numpy.html>`_) says:
   "When you build wheels for your package using a NumPy 1.xx version at
   build time, those will not work with NumPy 2.0."

Both Packages Come from the Course Environment
----------------------------------------------

.. code: lecture5/where_from.py all

.. code-block:: python
   :caption: lecture5/where_from.py

   """Print the version of NumPy and Matplotlib, and the folder of each."""

   from pathlib import Path

   import matplotlib
   import numpy

   for module in (numpy, matplotlib):
       folder = str(Path(module.__file__).parent)
       folder = folder.replace(str(Path.cwd()), ".")  # Shorter paths
       folder = folder.replace(str(Path.home()), "~")
       print(module.__name__, module.__version__, folder)

.. code-block:: bash

   uv run lecture5/where_from.py

.. output: where_from.txt all

.. code-block:: text

   numpy 1.26.4 ./.venv/lib/python3.12/site-packages/numpy
   matplotlib 3.11.2 ./.venv/lib/python3.12/site-packages/matplotlib

- ``__file__`` is the file a module was loaded from. Both come from
  ``.venv``, installed by uv.
- NumPy 1.26.4 is the version Jazzy was built against, so ROS 2's
  packages work with it when they run in ``.venv`` (Lecture 8).

Python Takes a Package from the First Folder That Has It
--------------------------------------------------------

.. code: lecture5/package_folders.py all

.. code-block:: python
   :caption: lecture5/package_folders.py

   """Print the folders Python searches for installed packages, in order."""

   import sys
   from pathlib import Path

   for folder in sys.path:
       if folder.endswith("packages"):
           folder = folder.replace(str(Path.cwd()), ".")  # Shorter paths
           print(folder.replace(str(Path.home()), "~"))

.. output: package_folders.txt all

.. code-block:: text

   ./.venv/lib/python3.12/site-packages
   ~/.local/lib/python3.12/site-packages
   /usr/local/lib/python3.12/dist-packages
   /usr/lib/python3/dist-packages
   /usr/lib/python3.12/dist-packages

- ``sys.path`` (Lecture 2) in ``.venv``: its own folder first, then
  ``~/.local`` (where ``pip install --user`` puts packages, Lecture 1,
  step 5), then Ubuntu's folders.
- The instructor's PC also has two other copies of Matplotlib:

.. output: other_matplotlib.txt all

.. code-block:: text

   ~/.local: 3.10.7
   apt: 3.6.3-1ubuntu5

- Python takes Matplotlib 3.11.2 from ``.venv``, the first folder that
  has it.
- A package that ``.venv`` does not have, such as Ubuntu's OpenCV
  (:doc:`Lecture 1 </lectures/lecture1/l1_lecture>`), still comes from a
  later folder, so a copy in ``~/.local`` can still hide Ubuntu's.

A Leftover from apt: Matplotlib's 3D Tools
------------------------------------------

.. code: lecture5/mpl_toolkits_check.py all

.. code-block:: python
   :caption: lecture5/mpl_toolkits_check.py

   """Print the folder Python loads mpl_toolkits from (Matplotlib's add-ons)."""

   from pathlib import Path

   import matplotlib.pyplot  # noqa: F401
   import mpl_toolkits

   for folder in mpl_toolkits.__path__:
       folder = folder.replace(str(Path.cwd()), ".")  # Shorter paths
       print(folder.replace(str(Path.home()), "~"))

On the instructor's PC:

.. output: mpl_toolkits_check.txt all short

.. code-block:: text

   .../site-packages/matplotlib/projections/__init__.py:63: UserWarning: Unable to import Axes3D. This may be due to multiple versions of Matplotlib being installed (e.g. as a system package and as a pip package). As a result, the 3D projection is not available.
     warnings.warn("Unable to import Axes3D. This may be due to multiple versions of "
   /usr/lib/python3/dist-packages/mpl_toolkits

.. output: mpl_toolkits_owner.txt all

.. code-block:: text
   :caption: dpkg -S /usr/lib/python3/dist-packages/mpl_toolkits/__init__.py

   python3-matplotlib: /usr/lib/python3/dist-packages/mpl_toolkits/__init__.py

- Importing ``matplotlib.pyplot`` prints a ``UserWarning``: "Unable to
  import Axes3D". 3D plots are then unavailable. 2D plots, all this
  course uses, work.
- ``mpl_toolkits`` is a folder of Matplotlib's add-ons, 3D plots
  among them. apt's ``python3-matplotlib`` 3.6.3 has
  ``mpl_toolkits/__init__.py``; the copy that comes with Matplotlib
  3.11.2 has none.
- `PEP 420 <https://peps.python.org/pep-0420/>`_: "If
  ``<directory>/foo/__init__.py`` is found, a regular package is
  imported and returned." A folder without ``__init__.py`` is only
  recorded while the search goes on, so apt's copy, in a later folder,
  wins.
- In a ``.venv`` made without ``--system-site-packages``, with the same
  NumPy and Matplotlib, Python loads the 3.11.2 copy and prints no
  warning:

.. output: mpl_toolkits_nosys.txt all

.. code-block:: text

   <scratch>/.venv/lib/python3.12/site-packages/mpl_toolkits

- Why apt's copy is there: ``ros-jazzy-desktop`` needs it, through
  ``rqt_plot`` (the matching line of ``apt-cache depends`` for each of
  three packages):

.. output: apt_chain.txt all

.. code-block:: text

   $ apt-cache depends ros-jazzy-desktop | grep rqt-common-plugins
     Depends: ros-jazzy-rqt-common-plugins
   $ apt-cache depends ros-jazzy-rqt-common-plugins | grep rqt-plot
     Depends: ros-jazzy-rqt-plot
   $ apt-cache depends ros-jazzy-rqt-plot | grep python3-matplotlib
     Depends: python3-matplotlib

Do not remove ``python3-matplotlib``: ROS 2's tools need it, and the
course environment needs ``--system-site-packages`` for ROS 2 from
Lecture 8.


Putting It All Together
====================================================

Exercise 4 on the :doc:`exercises page <l5_exercises>` is for after
class. It moves the scan into the world frame and plots it over the
walls of the room: arrays, masks, a rotation, broadcasting, and
Matplotlib in one program.


Next Class
====================================================

:doc:`L6: Object-Oriented Programming, Part I </lectures/lecture6/l6_index>`
(Tuesday, March 9). **Quiz 2** takes the first 20 minutes of class:
closed notes, no slides, and no software to test code.

- Introduction to object-oriented programming
- The design phase
- The implementation phase
- Classes and objects
- Dunder methods
- Abstraction and encapsulation
