====================================================
Lecture
====================================================

Every program on this page is in the ``lecture4/`` folder of the
`course repository <https://github.com/rubixcubic/enpm605-spring-2027-python>`_.
Run each one from the root of the repository, for example:

.. code-block:: bash

   uv run lecture4/functions_demo.py

Every output on this page was printed by running the program with the
course environment (CPython 3.12.3 on Ubuntu 24.04). In paths, ``...``
marks a path shortened to fit. The values in the examples (ranges,
speeds, battery levels, waypoints) are chosen for the examples. Some
listings leave out a blank line of the file to save space.


Defining and Calling Functions
====================================================

A Function Is a Named Block of Code You Call
--------------------------------------------

.. code: lecture4/functions_demo.py 4-16

.. code-block:: python
   :caption: From lecture4/functions_demo.py

   # Lecture 3, Exercise 1, task 5, as a function
   def first_obstacle(scan):
       """Index of the first valid reading under 0.5 m."""
       for i, r in enumerate(scan):
           if not (0.12 <= r <= 12.0):
               continue
           if r < 0.5:
               return i
       return None


   print(first_obstacle([2.5, 0.0, 1.1, 0.4, 13.5, 0.8]))
   print(first_obstacle([2.5, 0.0, 13.5, 0.8]))

.. output: functions_demo.txt 1-2

.. code-block:: text

   3
   None

- Lecture 3's :doc:`Exercise 1 <../lecture3/l3_exercises>` wrote this
  loop twice, once per scan. Here it is written once, in ``def``, and
  called twice.
- In each call, ``scan`` refers to the list passed. In the first,
  ``0.0`` and ``13.5`` are skipped as invalid, ``1.1`` is not below
  0.5 m, and ``0.4`` at index 3 is: ``return i`` ends the call with 3.
- In the second, no valid reading is below 0.5 m: the loop ends, and
  ``return None`` runs.

The Parts of a Function
-----------------------

.. code: lecture4/functions_demo.py 19-27

.. code-block:: python

   # The parts of a function
   def drive_time(distance_m, speed_mps):
       """Return the time in seconds to drive distance_m at speed_mps."""
       time_s = distance_m / speed_mps
       return time_s


   t = drive_time(3.0, 0.25)  # Values chosen for this example
   print(t)

.. output: functions_demo.txt 3

.. code-block:: text

   12.0

.. list-table::
   :header-rows: 1
   :widths: 20 80
   :class: compact-table

   * - Part
     - In this example
   * - Name
     - ``drive_time``: lowercase words joined by ``_``. PEP 8: "Function
       names should be lowercase, with words separated by underscores as
       necessary to improve readability."
   * - Parameters
     - ``distance_m`` and ``speed_mps``: the names the call fills in
   * - Docstring
     - The string on the first line of the body
   * - Body
     - The indented block: it runs at each call
   * - ``return``
     - Sends ``time_s`` back to the caller and ends the call
   * - Arguments
     - ``3.0`` and ``0.25``: the values in the call

A **parameter** is a name in the definition. An **argument** is the
value a call passes for it.

Step by Step: What the Call drive_time(3.0, 0.25) Does
------------------------------------------------------

1. Python evaluates the arguments: ``3.0`` and ``0.25``.
2. It creates a **frame** for the call, with its own namespace (tag 2
   in the figure).
3. It binds ``distance_m`` to ``3.0`` and ``speed_mps`` to ``0.25``.
4. The body runs: 3.0 / 0.25 = 12.0, and ``time_s`` refers to it.
5. ``return`` sends ``12.0`` back. The frame and the names of tag 2 are
   discarded.
6. ``t`` now refers to ``12.0``.

.. figure:: /_static/images/l4/call_frame.svg
   :alt: Three regions. Tag 1, the module's namespace, holds the name drive_time. Tag 2, the namespace of the call drive_time(3.0, 0.25), holds the names distance_m, speed_mps and time_s. Tag 3 holds the objects: the function drive_time and the floats 3.0, 0.25 and 12.0. Each name has an arrow to its object.
   :width: 80%

   At the ``return`` statement. 1: the module's names. 2: the names of
   this call. 3: the objects.

.. tip::

   Each call gets a new frame. The :doc:`Memory reading
   </reading_material/memory/mem_index>`'s "Call Frames" shows the
   frames of calls in progress.

A Function Without return Returns None
--------------------------------------

.. code: lecture4/functions_demo.py 30-36

.. code-block:: python

   # A function without return returns None
   def report(scan):
       print(f"{len(scan)} readings")


   result = report([2.5, 0.9])
   print(result)

.. output: functions_demo.txt 4-5

.. code-block:: text

   2 readings
   None

- ``report()`` prints, but has no ``return``. The call still gives back
  a value: ``None``.
- The tutorial: "even functions without a return statement do return a
  value, albeit a rather boring one. This value is called None".
- A bare ``return``, with no value, also returns ``None``.
- Printing is not returning: ``2 readings`` goes to the screen, and
  ``result`` gets ``None``.

A Missing return Shows Up Later as None
---------------------------------------

.. code: lecture4/errors/none_result.py all

.. code-block:: python
   :caption: lecture4/errors/none_result.py

   def mean_range(scan):
       total = sum(scan)
       total / len(scan)  # The return is missing


   mean = mean_range([2.5, 0.9, 3.1])
   print(f"Mean: {mean:.2f} m")

.. output: err_none_result.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../none_result.py", line 7, in <module>
       print(f"Mean: {mean:.2f} m")
                     ^^^^^^^^^^
   TypeError: unsupported format string passed to NoneType.__format__

- Line 3 computes the mean and drops it: nothing returns it, so
  ``mean_range()`` returns ``None``.
- The error comes at line 7, where the program first uses ``mean``, not
  in the function.
- When a function's result is ``None`` and you did not expect it, look
  for a missing ``return``.

Several Values Come Back as One Tuple
-------------------------------------

.. code: lecture4/functions_demo.py 39-48

.. code-block:: python

   # return gives back one object; several values travel as a tuple
   def nearest_and_farthest(scan):
       """Return the smallest and the largest reading of scan."""
       return min(scan), max(scan)


   pair = nearest_and_farthest([2.5, 0.9, 3.1])
   print(pair, type(pair).__name__)
   nearest, farthest = nearest_and_farthest([2.5, 0.9, 3.1])
   print(f"nearest {nearest} m, farthest {farthest} m")

.. output: functions_demo.txt 6-7

.. code-block:: text

   (0.9, 3.1) tuple
   nearest 0.9 m, farthest 3.1 m

- ``return min(scan), max(scan)`` returns one object: a tuple. The comma
  makes the tuple (:doc:`Lecture 3 <../lecture3/l3_lecture>`).
- Unpacking gives each value its own name, as in the second call.
- The number of names must match the number of values, or Python raises
  ``ValueError``, as in Lecture 3's ``unpack_count.py``.

def Creates a Function Object and Binds a Name to It
----------------------------------------------------

.. code: lecture4/functions_demo.py 51-55

.. code-block:: python

   # A function is an object, and def binds a name to it
   print(type(drive_time).__name__, drive_time.__name__)
   go = drive_time
   print(go is drive_time, go(1.0, 0.5))
   print(drive_time.__doc__)

.. output: functions_demo.txt 8-10

.. code-block:: text

   function drive_time
   True 2.0
   Return the time in seconds to drive distance_m at speed_mps.

- The tutorial: "A function definition associates the function name
  with the function object in the current symbol table."
- ``go = drive_time`` has no parentheses, so nothing is called: ``go``
  is a second name for the same function object.
- ``go(1.0, 0.5)`` calls it: 1.0 / 0.5 = 2.0.
- ``__doc__`` holds the docstring. `Functions as Values`_ builds on
  this.

A Second def with the Same Name Replaces the First
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``def`` binds a name, so a second ``def`` with the same name rebinds it.
Python gives no warning:

.. code: lecture4/errors/redefined_function.py all

.. code-block:: python
   :caption: lecture4/errors/redefined_function.py

   def stop():
       return "brake"


   def stop():  # A second def with the same name
       return "coast"


   print(stop())

.. output: err_redefined_function.txt all

.. code-block:: text

   coast

Ruff's default rules report it:

.. code-block:: bash

   uv run ruff check lecture4/errors/redefined_function.py

.. output: ruff_redefined.txt all

.. code-block:: text

   F811 Redefinition of unused `stop` from line 1
    --> lecture4/errors/redefined_function.py:5:5
     |
   5 | def stop():  # A second def with the same name
     |     ^^^^ `stop` redefined here
   6 |     return "coast"
     |
    ::: lecture4/errors/redefined_function.py:1:5
     |
   1 | def stop():
     |     ---- previous definition of `stop` here
   2 |     return "brake"
     |
   help: Remove definition: `stop`

   Found 1 error.

A Function Must Exist Before a Call Runs
----------------------------------------

.. code: lecture4/errors/call_before_def.py all

.. code-block:: python
   :caption: lecture4/errors/call_before_def.py

   print(drive_time(3.0, 0.25))


   def drive_time(distance_m, speed_mps):
       return distance_m / speed_mps

.. output: err_call_before_def.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../call_before_def.py", line 1, in <module>
       print(drive_time(3.0, 0.25))
             ^^^^^^^^^^
   NameError: name 'drive_time' is not defined

- ``def`` is a statement. Python runs the file from top to bottom, so at
  line 1 the name ``drive_time`` does not exist yet.
- A function body may call a function defined further down. The call
  works as long as both ``def`` statements have run when it happens.
- Ruff finds this one before the program runs:

.. code-block:: bash

   uv run ruff check lecture4/errors/call_before_def.py

.. output: ruff_call_before_def.txt all

.. code-block:: text

   F821 Undefined name `drive_time`
    --> lecture4/errors/call_before_def.py:1:7
     |
   1 | print(drive_time(3.0, 0.25))
     |       ^^^^^^^^^^

   Found 1 error.

A Docstring Documents a Function, and help() Prints It
------------------------------------------------------

.. code: lecture4/docstring_demo.py 4-17

.. code-block:: python
   :caption: From lecture4/docstring_demo.py

   def drive_time(distance_m, speed_mps):
       """Return the time to drive a distance at a constant speed.

       Args:
           distance_m: The distance to drive, in meters.
           speed_mps: The speed, in meters per second. Must not be 0.

       Returns:
           The time, in seconds.
       """
       return distance_m / speed_mps


   help(drive_time)

.. output: docstring_demo.txt 1-11

.. code-block:: text

   Help on function drive_time in module __main__:

   drive_time(distance_m, speed_mps)
       Return the time to drive a distance at a constant speed.

       Args:
           distance_m: The distance to drive, in meters.
           speed_mps: The speed, in meters per second. Must not be 0.

       Returns:
           The time, in seconds.

- ``help()`` prints the signature, then the docstring. This docstring
  uses the Google style: an ``Args:`` and a ``Returns:`` section. In a
  terminal, ``help()`` shows the text in a pager: press ``q`` to quit.
- The tutorial: "it's good practice to include docstrings in code that
  you write, so make a habit of it."
- VS Code's Python extension shows this docstring when the mouse is over
  ``drive_time``.


Arguments
====================================================

Positional Arguments Are Matched by Order
-----------------------------------------

.. code: lecture4/arguments_demo.py 4-10

.. code-block:: python
   :caption: From lecture4/arguments_demo.py

   def drive_time(distance_m, speed_mps):
       return distance_m / speed_mps


   # Positional arguments are matched by order
   print(drive_time(3.0, 0.25))
   print(drive_time(0.25, 3.0))  # Swapped: no error, wrong answer

.. output: arguments_demo.txt 1-2

.. code-block:: text

   12.0
   0.08333333333333333

- The first argument goes to the first parameter, the second to the
  second.
- The second call swaps them: ``distance_m`` is 0.25 and ``speed_mps``
  is 3.0, so it computes 0.25 / 3.0 = 0.0833...
- Python reports nothing: both values are floats, and the division
  works. The answer is wrong.

.. tip::

   When two parameters have the same type, a swap is easy to make and
   hard to see. Keyword arguments (`Keyword Arguments Are Matched by
   Name`_) prevent it.

Keyword Arguments Are Matched by Name
-------------------------------------

.. code: lecture4/arguments_demo.py 12-14

.. code-block:: python

   # Keyword arguments are matched by name
   print(drive_time(speed_mps=0.25, distance_m=3.0))
   print(drive_time(3.0, speed_mps=0.25))

.. output: arguments_demo.txt 3-4

.. code-block:: text

   12.0
   12.0

``name=value`` in a call passes the value to the parameter with that
name, in any order. A call can mix both kinds, positional arguments
first. A positional argument after a keyword argument is a
``SyntaxError``: Python rejects the whole file before it runs any line.

.. code: lecture4/errors/positional_after_keyword.py all

.. code-block:: python
   :caption: lecture4/errors/positional_after_keyword.py

   def drive_time(distance_m, speed_mps):
       return distance_m / speed_mps


   print(drive_time(distance_m=3.0, 0.25))

.. output: err_positional_after_keyword.txt all short

.. code-block:: text

     File ".../positional_after_keyword.py", line 5
       print(drive_time(distance_m=3.0, 0.25))
                                            ^
   SyntaxError: positional argument follows keyword argument

A Missing or an Extra Argument Is a TypeError
---------------------------------------------

.. code: lecture4/errors/missing_argument.py all

.. code-block:: python
   :caption: lecture4/errors/missing_argument.py

   def drive_time(distance_m, speed_mps):
       return distance_m / speed_mps


   print(drive_time(3.0))

.. output: err_missing_argument.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../missing_argument.py", line 5, in <module>
       print(drive_time(3.0))
             ^^^^^^^^^^^^^^^
   TypeError: drive_time() missing 1 required positional argument: 'speed_mps'

.. code: lecture4/errors/too_many_arguments.py 5

.. code-block:: python
   :caption: Line 5 of lecture4/errors/too_many_arguments.py

   print(drive_time(3.0, 0.25, 0.1))

.. output: err_too_many_arguments.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../too_many_arguments.py", line 5, in <module>
       print(drive_time(3.0, 0.25, 0.1))
             ^^^^^^^^^^^^^^^^^^^^^^^^^^
   TypeError: drive_time() takes 2 positional arguments but 3 were given

``drive_time(distance_m, speed_mps)`` needs exactly two arguments.
Unlike the swap of `Positional Arguments Are Matched by Order`_, a wrong
count fails at once, at the call, and the message names the problem.

A Default Value Makes an Argument Optional
------------------------------------------

.. code: lecture4/arguments_demo.py 17-23

.. code-block:: python

   # A default value makes an argument optional
   def drive_time_at(distance_m, speed_mps=0.2):  # 0.2: chosen
       return distance_m / speed_mps


   print(drive_time_at(3.0))
   print(drive_time_at(3.0, 0.25))

.. output: arguments_demo.txt 5-6

.. code-block:: text

   15.0
   12.0

- A call without ``speed_mps`` uses the default, 0.2: 3.0 / 0.2 = 15.0.
  A call that passes it gets 3.0 / 0.25 = 12.0.
- A parameter with a default must come after every parameter without
  one:

.. code: lecture4/errors/default_order.py all

.. code-block:: python
   :caption: lecture4/errors/default_order.py

   def drive_time(distance_m=1.0, speed_mps):
       return distance_m / speed_mps

.. output: err_default_order.txt all short

.. code-block:: text

     File ".../default_order.py", line 1
       def drive_time(distance_m=1.0, speed_mps):
                                      ^^^^^^^^^
   SyntaxError: parameter without a default follows parameter with a default

A Default Value Is Created Once, When def Runs
----------------------------------------------

.. code: lecture4/errors/mutable_default.py all

.. code-block:: python
   :caption: lecture4/errors/mutable_default.py

   def log_reading(r, history=[]):
       history.append(r)
       return history


   a = log_reading(2.5)
   b = log_reading(0.9)
   print(a, b, a is b)
   print(log_reading.__defaults__)

.. output: err_mutable_default.txt all

.. code-block:: text

   [2.5, 0.9] [2.5, 0.9] True
   ([2.5, 0.9],)

- The intent: each call without ``history`` starts a new list. Instead,
  the second call returns ``[2.5, 0.9]``.
- The tutorial: "The default value is evaluated only once." The list
  ``[]`` is made when ``def`` runs and kept in
  ``log_reading.__defaults__``.
- Every call without ``history`` appends to that one list, so ``a`` and
  ``b`` are the same object: ``a is b`` is ``True``. The
  :doc:`Memory reading </reading_material/memory/mem_index>`'s "Where
  This Comes Back" lists this case.

Use None as the Default for a Mutable Value
-------------------------------------------

.. code: lecture4/arguments_demo.py 26-36

.. code-block:: python

   # None as the default: a new list at each call
   def log_reading(r, history=None):
       if history is None:
           history = []
       history.append(r)
       return history


   a = log_reading(2.5)
   b = log_reading(0.9)
   print(a, b, a is b)

.. output: arguments_demo.txt 7

.. code-block:: text

   [2.5] [0.9] False

``None`` is immutable, so sharing it is safe. The body makes a new list
at each call that passes no list. Ruff's default rules report the shared
default, and suggest this fix:

.. code-block:: bash

   uv run ruff check lecture4/errors/mutable_default.py

.. output: ruff_mutable_default.txt all

.. code-block:: text

   B006 Do not use mutable data structures for argument defaults
    --> lecture4/errors/mutable_default.py:1:28
     |
   1 | def log_reading(r, history=[]):
     |                            ^^
   2 |     history.append(r)
   3 |     return history
     |
   help: Replace with `None`; initialize within function
     |
     - def log_reading(r, history=[]):
   1 + def log_reading(r, history=None):
   2 +     if history is None:
   3 +         history = []
   4 |     history.append(r)
     |
   note: This is an unsafe fix and may change runtime behavior

   Found 1 error.
   1 unsafe fix available with `--fix --unsafe-fixes`.

\*args Collects Extra Positional Arguments in a Tuple
-----------------------------------------------------

.. code: lecture4/arguments_demo.py 39-46

.. code-block:: python

   # *args collects extra positional arguments in a tuple
   def mean_of(*readings):
       print(type(readings).__name__, readings)
       return sum(readings) / len(readings)


   print(f"{mean_of(2.5, 0.9, 3.1):.2f}")
   print(mean_of(1.2))

.. output: arguments_demo.txt 8-11

.. code-block:: text

   tuple (2.5, 0.9, 3.1)
   2.17
   tuple (1.2,)
   1.2

- ``*readings`` accepts any number of positional arguments, and
  ``readings`` is a tuple of them.
- The name after ``*`` is yours to choose; ``*args`` is the usual name
  when nothing better fits.
- The first mean, in steps:

  1. 2.5 + 0.9 + 3.1 = 6.5.
  2. 6.5 / 3 = 2.166..., printed with 2 decimals: ``2.17``.

\*\*kwargs Collects Extra Keyword Arguments in a Dictionary
-----------------------------------------------------------

.. code: lecture4/arguments_demo.py 49-56

.. code-block:: python

   # **kwargs collects extra keyword arguments in a dictionary
   def make_robot(name, **settings):
       print(type(settings).__name__, settings)
       settings["name"] = name
       return settings


   print(make_robot("waffle_01", max_speed_mps=0.26, lidar=True))

.. output: arguments_demo.txt 12-13

.. code-block:: text

   dict {'max_speed_mps': 0.26, 'lidar': True}
   {'max_speed_mps': 0.26, 'lidar': True, 'name': 'waffle_01'}

- ``**settings`` collects every keyword argument that matches no
  parameter, in a dictionary: name to value.
- ``name`` is a normal parameter, so ``"waffle_01"`` goes there and not
  into ``settings``.
- The dictionary keeps the order of the call (insertion order, Lecture
  3); ``"name"`` is added last.
- ``**kwargs`` is the usual name when nothing better fits. The settings
  in the call are chosen for this example.

Parameters After a Bare \* Are Keyword-Only
-------------------------------------------

.. code: lecture4/arguments_demo.py 59-64

.. code-block:: python

   # Parameters after * are keyword-only
   def move(distance_m, *, speed_mps=0.2):
       return f"move {distance_m} m at {speed_mps} m/s"


   print(move(1.5, speed_mps=0.1))

.. output: arguments_demo.txt 14

.. code-block:: text

   move 1.5 m at 0.1 m/s

.. code: lecture4/errors/keyword_only.py 5

.. code-block:: python
   :caption: Line 5 of lecture4/errors/keyword_only.py

   print(move(1.5, 0.1))

.. output: err_keyword_only.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../keyword_only.py", line 5, in <module>
       print(move(1.5, 0.1))
             ^^^^^^^^^^^^^^
   TypeError: move() takes 1 positional argument but 2 were given

- After a bare ``*``, a parameter can only be passed by name. The call
  must say ``speed_mps=0.1``.
- The reverse exists: parameters before a ``/`` are positional-only.
  ``inspect.signature(len)`` shows ``(obj, /)``.

The Order of the Kinds of Parameters
------------------------------------

.. code: lecture4/arguments_demo.py 67-72

.. code-block:: python

   # Every kind of parameter, in the order Python requires
   def show(a, b, *args, option=True, **kwargs):
       print(f"{a=}, {b=}, {args=}, {option=}, {kwargs=}")


   show(1, 2, 3, 4, option=False, x=10)

.. output: arguments_demo.txt 15

.. code-block:: text

   a=1, b=2, args=(3, 4), option=False, kwargs={'x': 10}

.. list-table::
   :header-rows: 1
   :widths: 34 22 44
   :class: compact-table

   * - Kind
     - In ``show()``
     - Gets, in the call above
   * - Positional or keyword
     - ``a, b``
     - ``1`` and ``2``
   * - Extra positional
     - ``*args``
     - The tuple ``(3, 4)``
   * - Keyword-only, with a default
     - ``option=True``
     - ``False``
   * - Extra keyword
     - ``**kwargs``
     - The dictionary ``{'x': 10}``

- A parameter after ``*args`` is keyword-only, as after a bare ``*``.
  Without a default, a call must pass it by name:
  ``def f(*args, k)`` called as ``f(1, 2)`` raises
  ``TypeError: f() missing 1 required keyword-only argument: 'k'``.
- ``**kwargs`` comes last. ``f"{a=}"`` prints the name and the value
  (Lecture 2).

\* and \*\* Unpack Values into the Arguments of a Call
------------------------------------------------------

.. code: lecture4/arguments_demo.py 74-78

.. code-block:: python

   # * and ** unpack a sequence and a dictionary into arguments
   leg = (3.0, 0.25)
   print(drive_time(*leg))
   config = {"distance_m": 3.0, "speed_mps": 0.25}
   print(drive_time(**config))

.. output: arguments_demo.txt 16-17

.. code-block:: text

   12.0
   12.0

- In a call, ``*leg`` passes the items of ``leg`` as positional
  arguments: ``drive_time(3.0, 0.25)``.
- ``**config`` passes each key and value as a keyword argument:
  ``drive_time(distance_m=3.0, speed_mps=0.25)``.
- The keys must be the parameter names. A key that matches no parameter
  raises ``TypeError``, for example
  ``f() got an unexpected keyword argument 'b'``.
- In a definition, ``*`` and ``**`` collect arguments; in a call, they
  spread values out.

Exercise 1 on the :doc:`exercises page <l4_exercises>` practices this
section and the one before it.


Scopes
====================================================

A Name Assigned in a Function Is Local to the Call
--------------------------------------------------

.. code: lecture4/scopes_demo.py 3-13

.. code-block:: python
   :caption: From lecture4/scopes_demo.py

   speed_mps = 0.2  # A global name (chosen for this example)


   def drive_time(distance_m):
       time_s = distance_m / speed_mps  # time_s is local
       print(locals())
       return time_s


   print(drive_time(3.0))
   print("time_s" in globals(), "speed_mps" in globals())

.. output: scopes_demo.txt 1-3

.. code-block:: text

   {'distance_m': 3.0, 'time_s': 15.0}
   15.0
   False True

- A **scope** is the part of a program where a name can be used without
  a prefix.
- ``locals()`` lists the call's own names: the parameter and
  ``time_s``. ``speed_mps`` is not among them; the function reads the
  module's name.
- After the call, ``time_s`` is gone:

.. code: lecture4/errors/local_after_call.py all

.. code-block:: python
   :caption: lecture4/errors/local_after_call.py

   def drive_time(distance_m, speed_mps):
       time_s = distance_m / speed_mps
       return time_s


   drive_time(3.0, 0.25)
   print(time_s)

.. output: err_local_after_call.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../local_after_call.py", line 7, in <module>
       print(time_s)
             ^^^^^^
   NameError: name 'time_s' is not defined

Each Name Below Is Found in a Different Scope
---------------------------------------------

.. code: lecture4/scopes_demo.py 15-29

.. code-block:: python

   # LEGB: each name below is found in a different scope
   robot = "waffle_01"  # G: assigned at the top of the file


   def outer():
       sensor = "lidar"  # E, for inner: assigned in outer

       def inner():
           reading = 2.5  # L: assigned in inner
           print(reading, sensor, robot, max(reading, 1.0))  # max: B

       inner()


   outer()

.. output: scopes_demo.txt 4

.. code-block:: text

   2.5 lidar waffle_01 2.5

- ``inner()`` is a **nested function**: defined inside ``outer()``, and
  called there.
- ``inner()`` assigns only ``reading``. It reads three names it did not
  assign: ``sensor``, ``robot``, and ``max``.

Python Looks in Four Scopes, Innermost First
--------------------------------------------

.. figure:: /_static/images/l4/legb.svg
   :alt: Four nested regions. Tag 4, the outermost, holds the built-in names, such as max. Tag 3, the module (global), holds robot and outer. Tag 2, the call outer() (enclosing), holds sensor and inner. Tag 1, the innermost, the call inner() (local), holds reading.
   :width: 55%

   The LEGB rule. 1: **L**\ ocal, the call ``inner()``. 2: **E**\
   nclosing, the call ``outer()``. 3: **G**\ lobal, the module. 4:
   **B**\ uilt-in: ``print``, ``len``, ``max``, and the others.

Python looks in 1, then 2, then 3, then 4. The first match wins; a name
found nowhere is a ``NameError``. `The tutorial
<https://docs.python.org/3.12/tutorial/controlflow.html#defining-functions>`_:
variable references "first look
in the local symbol table, then in the local symbol tables of enclosing
functions, then in the global symbol table, and finally in the table of
built-in names."

The same name in three scopes is three different names. Each
assignment below makes a new name in its own scope:

.. code: lecture4/scopes_demo.py 31-47

.. code-block:: python

   # The same name in three scopes: three different names
   label = "global"


   def outer2():
       label = "enclosing"

       def inner2():
           label = "local"
           print("inner2:", label)

       inner2()
       print("outer2:", label)


   outer2()
   print("module:", label)

.. output: scopes_demo.txt 5-7

.. code-block:: text

   inner2: local
   outer2: enclosing
   module: global

An Assignment Makes a Name Local in the Whole Function
------------------------------------------------------

.. code: lecture4/errors/unbound_local.py all

.. code-block:: python
   :caption: lecture4/errors/unbound_local.py

   scans_seen = 0


   def on_scan(scan):
       scans_seen += 1  # An assignment: scans_seen is local here
       print(f"Scan {scans_seen}: nearest {min(scan)} m")


   on_scan([2.5, 0.9])

.. output: err_unbound_local.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../unbound_local.py", line 9, in <module>
       on_scan([2.5, 0.9])
     File ".../unbound_local.py", line 5, in on_scan
       scans_seen += 1  # An assignment: scans_seen is local here
       ^^^^^^^^^^
   UnboundLocalError: cannot access local variable 'scans_seen' where it is not associated with a value

- ``scans_seen += 1`` assigns, so ``scans_seen`` is local in all of
  ``on_scan()``, and it has no value yet when ``+=`` reads it.
- The FAQ: "when you make an assignment to a variable in a scope, that
  variable becomes local to that scope and shadows any similarly named
  variable in the outer scope."
- Ruff reports it before the program runs:

.. code-block:: bash

   uv run ruff check lecture4/errors/unbound_local.py

.. output: ruff_unbound_local.txt all

.. code-block:: text

   F823 Local variable `scans_seen` referenced before assignment
    --> lecture4/errors/unbound_local.py:5:5
     |
   4 | def on_scan(scan):
   5 |     scans_seen += 1  # An assignment: scans_seen is local here
     |     ^^^^^^^^^^
   6 |     print(f"Scan {scans_seen}: nearest {min(scan)} m")
     |

   Found 1 error.

global Assigns to the Module's Name
-----------------------------------

.. code: lecture4/scopes_demo.py 49-61

.. code-block:: python

   # global: assign to the module's name
   scans_seen = 0


   def on_scan(scan):
       global scans_seen
       scans_seen += 1
       print(f"Scan {scans_seen}: nearest {min(scan)} m")


   on_scan([2.5, 0.9])
   on_scan([0.4])
   print(scans_seen)

.. output: scopes_demo.txt 8-10

.. code-block:: text

   Scan 1: nearest 0.9 m
   Scan 2: nearest 0.4 m
   2

- ``global scans_seen`` says: in this function, ``scans_seen`` is the
  module's name. The ``+=`` now changes the module's count.
- Any function can change a global, so a bug in one is hard to trace.
  Prefer passing a value in and returning the result.
- `A Closure Keeps the Names of Its Enclosing Call`_ keeps a count
  without a global.

Changing a Global Object Needs No global
----------------------------------------

.. code: lecture4/scopes_demo.py 63-73

.. code-block:: python

   # Changing a global list needs no global statement
   history = []


   def log_reading(r):
       history.append(r)


   log_reading(2.5)
   log_reading(0.9)
   print(history)

.. output: scopes_demo.txt 11

.. code-block:: text

   [2.5, 0.9]

- ``history.append(r)`` reads the name ``history`` and changes the list
  it refers to. It never assigns ``history``.
- With no assignment, the name is not local: LEGB finds the module's
  ``history``.
- ``history += [r]`` is an assignment, even though ``+=`` on a list
  changes the list in place. So it makes ``history`` local, and the call
  fails:

.. code: lecture4/errors/augmented_global.py all

.. code-block:: python
   :caption: lecture4/errors/augmented_global.py

   history = []


   def log_reading(r):
       history += [r]


   log_reading(2.5)

.. output: err_augmented_global.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../augmented_global.py", line 8, in <module>
       log_reading(2.5)
     File ".../augmented_global.py", line 5, in log_reading
       history += [r]
       ^^^^^^^
   UnboundLocalError: cannot access local variable 'history' where it is not associated with a value

A function that assigns a module's name, with ``=``, ``+=``, or any
other augmented assignment, needs ``global``.

nonlocal Assigns to the Enclosing Function's Name
-------------------------------------------------

.. code: lecture4/scopes_demo.py 76-90

.. code-block:: python

   # nonlocal: assign to a name of the enclosing function
   def process(scans):
       count = 0

       def on_scan(scan):
           nonlocal count
           count += 1
           print(f"Scan {count}: nearest {min(scan)} m")

       for scan in scans:
           on_scan(scan)
       return count


   print(process([[2.5, 0.9], [0.4], [1.0, 1.1, 1.2]]))

.. output: scopes_demo.txt 12-15

.. code-block:: text

   Scan 1: nearest 0.9 m
   Scan 2: nearest 0.4 m
   Scan 3: nearest 1.0 m
   3

- ``nonlocal count`` makes ``count`` the name of ``process()``, the
  enclosing call.
- Without it, ``count += 1`` makes ``count`` local:
  ``UnboundLocalError``, as in ``unbound_local.py``.
- ``global`` goes straight to the module. ``nonlocal`` looks in the
  enclosing functions, nearest first, and uses the first one that binds
  the name; it never reaches the module.

A Global Name Can Hide a Built-in
---------------------------------

.. code: lecture4/errors/shadow_builtin.py all

.. code-block:: python
   :caption: lecture4/errors/shadow_builtin.py

   max = 12.0  # The lidar's largest range, in meters
   ranges_m = [2.5, 0.9, 3.1]
   print(max(ranges_m))

.. output: err_shadow_builtin.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../shadow_builtin.py", line 3, in <module>
       print(max(ranges_m))
             ^^^^^^^^^^^^^
   TypeError: 'float' object is not callable

LEGB finds the global ``max``, the float ``12.0``, before the built-in
``max()``. A float cannot be called. Ruff reports it when you turn on
rule A001:

.. code-block:: bash

   uv run ruff check --extend-select A001 lecture4/errors/shadow_builtin.py

.. output: ruff_shadow_builtin.txt all

.. code-block:: text

   A001 Variable `max` is shadowing a Python builtin
    --> lecture4/errors/shadow_builtin.py:1:1
     |
   1 | max = 12.0  # The lidar's largest range, in meters
     | ^^^
   2 | ranges_m = [2.5, 0.9, 3.1]
   3 | print(max(ranges_m))
     |

   Found 1 error.

Avoid built-in names such as ``max``, ``list``, ``id``, and ``type``:
write ``max_range_m``.


Pass-by-Assignment
====================================================

A Parameter Is a New Name for the Caller's Object
-------------------------------------------------

.. code: lecture4/pass_by_assignment_demo.py 4-10

.. code-block:: python
   :caption: From lecture4/pass_by_assignment_demo.py

   def show_id(scan):
       print("inside: ", id(scan))


   ranges_m = [2.5, 14.0, 3.1]
   print("outside:", id(ranges_m))
   show_id(ranges_m)

.. output: pass_by_assignment_demo.txt 1-2

.. code-block:: text

   outside: 132333178161984
   inside:  132333178161984

- The same ``id()`` inside and outside: ``scan`` and ``ranges_m`` refer
  to one list. Nothing was copied.
- A call binds each parameter to its argument's object, as
  ``scan = ranges_m`` would.
- The FAQ: "Remember that arguments are passed by assignment in
  Python."
- Your ``id()`` values will differ; the two lines still match.

Changing the Object or Assigning the Name
-----------------------------------------

.. code: lecture4/pass_by_assignment_demo.py 13-29

.. code-block:: python

   # Changing the object: the caller sees it
   def clip_in_place(scan, max_m):
       for i, r in enumerate(scan):
           if r > max_m:
               scan[i] = max_m


   # Assigning the parameter: the caller does not see it
   def clip_rebind(scan, max_m):
       scan = [min(r, max_m) for r in scan]


   ranges_m = [2.5, 14.0, 3.1]
   clip_rebind(ranges_m, 12.0)
   print(ranges_m)
   clip_in_place(ranges_m, 12.0)
   print(ranges_m)

.. output: pass_by_assignment_demo.txt 3-4

.. code-block:: text

   [2.5, 14.0, 3.1]
   [2.5, 12.0, 3.1]

- ``clip_rebind()`` makes a new list and binds the local ``scan`` to it.
  ``ranges_m`` still shows ``14.0``.
- ``clip_in_place()`` changes the caller's list through
  ``scan[i] = max_m``, so ``14.0`` became ``12.0``.

.. figure:: /_static/images/l4/pass_by_assignment.svg
   :alt: Two panels, each with an upper gray box for the module's names and a lower one for the call's names. Tag 1, inside clip_in_place: the names ranges_m and scan both have arrows to one list, [2.5, 12.0, 3.1]. Tag 2, inside clip_rebind after scan is assigned: ranges_m has an arrow to the list [2.5, 14.0, 3.1], and scan has an arrow to a new list, [2.5, 12.0, 3.1].
   :width: 95%

   Upper box: the module's names; lower box: the call's names. 1: inside
   ``clip_in_place()``, after ``scan[1] = 12.0``: one list, two names.
   2: inside ``clip_rebind()``, after ``scan = [...]``: ``scan`` refers
   to a new list. The caller's list is untouched, and the new one is lost
   when the call ends.

.. tip::

   Changing an object through a parameter changes the caller's object.
   Assigning the parameter changes only the call's name.

Return a New Object; an int Cannot Be Changed
---------------------------------------------

.. code: lecture4/pass_by_assignment_demo.py 32-49

.. code-block:: python

   # Return a new object instead
   def clipped(scan, max_m):
       return [min(r, max_m) for r in scan]


   raw = [2.5, 14.0, 3.1]
   clean = clipped(raw, 12.0)
   print(raw, clean)


   # An immutable argument: += makes a new object
   def add_charge(pct):
       pct += 10
       return pct


   battery_pct = 80  # Chosen for this example
   print(add_charge(battery_pct), battery_pct)

.. output: pass_by_assignment_demo.txt 5-6

.. code-block:: text

   [2.5, 14.0, 3.1] [2.5, 12.0, 3.1]
   90 80

- ``clipped()`` leaves ``raw`` as it was and returns the clean list. The
  caller decides what to keep.
- An ``int`` is immutable: ``pct += 10`` makes a new int,
  80 + 10 = 90, and binds the local ``pct``. ``battery_pct`` stays 80.

Exercise 2 on the :doc:`exercises page <l4_exercises>` practices scopes
and pass-by-assignment.


Type Hints
====================================================

A Type Hint Says What a Function Expects and Returns
----------------------------------------------------

.. code: lecture4/type_hints_demo.py 4-10

.. code-block:: python
   :caption: From lecture4/type_hints_demo.py

   def drive_time(distance_m: float, speed_mps: float) -> float:
       return distance_m / speed_mps


   print(drive_time(3.0, 0.25))
   for name, hint in drive_time.__annotations__.items():
       print(name, hint)

.. output: type_hints_demo.txt 1-4

.. code-block:: text

   12.0
   distance_m <class 'float'>
   speed_mps <class 'float'>
   return <class 'float'>

- ``name: type`` after a parameter, and ``-> type`` before the colon,
  for the return value.
- Python stores the hints in the function's ``__annotations__``
  dictionary, and does nothing else with them.
- :doc:`Lecture 1 <../lecture1/l1_lecture>` showed a name rebound to
  another type. Hints let a tool find such bugs before the program runs.

Python Does Not Check Type Hints When It Runs
---------------------------------------------

.. code: lecture4/type_hints_demo.py 13-19

.. code-block:: python

   # Python does not check the hints when it runs the code
   def repeat_label(label: str, count: int) -> str:
       return label * count


   print(repeat_label("ab", 3))
   print(repeat_label(2.5, 2))  # Not a str, and it runs

.. output: type_hints_demo.txt 5-6

.. code-block:: text

   ababab
   5.0

- The hint says ``label`` is a ``str``. The second call passes the float
  ``2.5``, and Python runs it: 2.5 * 2 = 5.0.
- The ``typing`` documentation: "The Python runtime does not enforce
  function and variable type annotations. They can be used by third
  party tools such as type checkers, IDEs, linters, etc."
- VS Code's Python extension reads the hints too, to suggest names as
  you type.

Hints for Collections and for a Value That Can Be None
------------------------------------------------------

.. code: lecture4/type_hints_demo.py 22-44

.. code-block:: python

   # Hints for collections, and for a value that can be None
   def nearest(scan: list[float]) -> float:
       return min(scan)


   def pose() -> tuple[float, float, float]:
       return 1.5, -0.3, 90.0


   def limits() -> dict[str, float]:
       return {"min_m": 0.12, "max_m": 12.0}


   def find_sensor(names: list[str], wanted: str) -> int | None:
       for i, name in enumerate(names):
           if name == wanted:
               return i
       return None


   print(nearest([2.5, 0.9]), pose(), limits())
   print(find_sensor(["lidar", "imu"], "imu"))
   print(find_sensor(["lidar", "imu"], "gps"))

.. output: type_hints_demo.txt 7-9

.. code-block:: text

   0.9 (1.5, -0.3, 90.0) {'min_m': 0.12, 'max_m': 12.0}
   1
   None

.. list-table::
   :header-rows: 1
   :widths: 40 60
   :class: compact-table

   * - Hint
     - Means
   * - ``list[float]``
     - A list of floats
   * - ``tuple[float, float, float]``
     - A tuple of exactly three floats
   * - ``dict[str, float]``
     - A dictionary from strings to floats
   * - ``int | None``
     - An int or ``None``; older code writes ``Optional[int]``

mypy Reads the Hints and Reports the Wrong Call
-----------------------------------------------

mypy is a type checker. It is not in the course environment: ``uvx``
runs it in a temporary environment, and does not add mypy to the
project's dependencies.

.. code-block:: bash

   uvx mypy --pretty lecture4/type_hints_demo.py

.. output: mypy_type_hints_demo.txt all

.. code-block:: text

   lecture4/type_hints_demo.py:19: error: Argument 1 to "repeat_label" has
   incompatible type "float"; expected "str"  [arg-type]
       print(repeat_label(2.5, 2))  # Not a str, and it runs
                          ^~~
   Found 1 error in 1 file (checked 1 source file)

- mypy reads the file without running it, and reports line 19:
  ``repeat_label(2.5, 2)``, the call Python ran without a complaint in
  `Python Does Not Check Type Hints When It Runs`_.
- ``[arg-type]`` is the name of the check, and ``^~~`` marks the
  argument.
- Recorded with mypy 2.4.0 in a 72-column terminal; mypy wraps its
  messages to the width of yours.

A Value That Can Be None Fails Only on Some Runs
------------------------------------------------

.. code: lecture4/type_check.py 4-14

.. code-block:: python
   :caption: From lecture4/type_check.py

   def first_obstacle(scan: list[float], safe_m: float) -> int | None:
       """Return the index of the first reading below safe_m, or None."""
       for i, r in enumerate(scan):
           if r < safe_m:
               return i
       return None


   scan = [2.5, 0.9, 3.1]
   i = first_obstacle(scan, 0.5)
   print(f"Obstacle {scan[i]} m away")  # i can be None

.. output: type_check.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../type_check.py", line 14, in <module>
       print(f"Obstacle {scan[i]} m away")  # i can be None
                         ~~~~^^^
   TypeError: list indices must be integers or slices, not NoneType

- No reading is below 0.5 m, so ``first_obstacle()`` returns ``None``,
  and ``scan[None]`` fails.
- With a reading below 0.5 m in the scan, the same program runs without
  an error.

mypy Finds the None Before the Program Runs
-------------------------------------------

.. code-block:: bash

   uvx mypy --pretty lecture4/type_check.py

.. output: mypy_type_check.txt all

.. code-block:: text

   lecture4/type_check.py:14: error: Invalid index type "int | None" for
   "list[float]"; expected type "SupportsIndex"  [index]
       print(f"Obstacle {scan[i]} m away")  # i can be None
                              ^
   Found 1 error in 1 file (checked 1 source file)

- The hint ``-> int | None`` tells mypy that ``i`` can be ``None``, and
  a list index must be an int.
- mypy reports it on every run, whatever the scan holds. The fix, in
  ``type_check_fixed.py``: check before you index.

.. code: lecture4/type_check_fixed.py 14-17

.. code-block:: python
   :caption: From lecture4/type_check_fixed.py

   if i is not None:
       print(f"Obstacle {scan[i]} m away")
   else:
       print("No obstacle below 0.5 m")

.. code-block:: bash

   uvx mypy --pretty lecture4/type_check_fixed.py

.. output: mypy_type_check_fixed.txt all

.. code-block:: text

   Success: no issues found in 1 source file

After ``if i is not None:``, mypy knows that ``i`` is an int.


Recursive Functions
====================================================

A Recursive Function Calls Itself on a Smaller Problem
------------------------------------------------------

.. code: lecture4/recursion_demo.py 4-10

.. code-block:: python
   :caption: From lecture4/recursion_demo.py

   def factorial(n: int) -> int:
       if n == 0:  # Base case: no further call
           return 1
       return n * factorial(n - 1)  # Recursive case: a smaller n


   print(factorial(5))

.. output: recursion_demo.txt 1

.. code-block:: text

   120

- *n*! ("*n* factorial") is *n* × (*n* - 1) × ... × 1, and 0! = 1. So
  *n*! = *n* × (*n* - 1)!.
- The **base case**, ``n == 0``, returns without another call. The
  **recursive case** calls the function with ``n - 1``.
- Each call is one step closer to the base case: 5, 4, 3, 2, 1, 0.
- The result: 5 × 4 × 3 × 2 × 1 × 1 = 120.

Each Call Has Its Own n
-----------------------

The same function, printing each call and each return:

.. code: lecture4/recursion_demo.py 13-25

.. code-block:: python

   # The same function, printing each call and each return
   def factorial_traced(n: int, depth: int = 0) -> int:
       pad = "    " * depth
       print(f"{pad}factorial({n})")
       if n == 0:
           result = 1
       else:
           result = n * factorial_traced(n - 1, depth + 1)
       print(f"{pad}returns {result}")
       return result


   factorial_traced(3)

.. output: recursion_demo.txt 2-9

.. code-block:: text

   factorial(3)
       factorial(2)
           factorial(1)
               factorial(0)
               returns 1
           returns 1
       returns 2
   returns 6

- ``depth`` counts how many calls are in progress above the first one.
  Each line is indented by 4 spaces per level.
- A call prints its line before it calls the next one, and prints
  ``returns`` after the next one has returned.
- The returns, from the top: 1, then 1 × 1 = 1, then 2 × 1 = 2, then
  3 × 2 = 6.

.. figure:: /_static/images/l4/recursion_stack.svg
   :alt: Five frames stacked, newest on top. Tags 1 to 4: the frames of factorial(0), running and drawn with a thick border, then factorial(1), factorial(2) and factorial(3), each with its own name n referring to the int 0, 1, 2 and 3. Tag 5: the module's frame, at the bottom. A curved arrow goes from each frame to the frame below it.
   :width: 55%

   While ``factorial(0)`` runs. 1 to 4: the frames of ``factorial(0)``
   to ``factorial(3)``, newest on top, each with its own ``n``; tag 1,
   with the thick border, is running. 5: the module's frame. The arrows
   point to each caller.

Recursion Follows a Tree of Coordinate Frames
---------------------------------------------

.. code: lecture4/recursion_demo.py 27-37

.. code-block:: python

   # Coordinate frames: each value is a dict of child frames
   frames = {"odom": {"base_link": {"lidar_link": {}, "camera_link": {}}}}


   def print_tree(tree: dict, depth: int = 0) -> None:
       for name, children in tree.items():
           print("  " * depth + name)
           print_tree(children, depth + 1)  # {} has no items: no call


   print_tree(frames)

.. output: recursion_demo.txt 10-13

.. code-block:: text

   odom
     base_link
       lidar_link
       camera_link

- A robot's coordinate frames form a tree (Lecture 11, TF2). Here each
  frame's value is a dictionary of its child frames.
- ``print_tree()`` prints each name, then calls itself on the children,
  one level deeper.
- The base case has no ``if``: an empty dictionary has no items, so the
  loop makes no call.

Without a Base Case, the Calls Never Stop
-----------------------------------------

.. code: lecture4/errors/no_base_case.py all

.. code-block:: python
   :caption: lecture4/errors/no_base_case.py

   def factorial(n):
       return n * factorial(n - 1)  # No base case


   print(factorial(5))

.. output: err_no_base_case.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../no_base_case.py", line 5, in <module>
       print(factorial(5))
             ^^^^^^^^^^^^
     File ".../no_base_case.py", line 2, in factorial
       return n * factorial(n - 1)  # No base case
                  ^^^^^^^^^^^^^^^^
     File ".../no_base_case.py", line 2, in factorial
       return n * factorial(n - 1)  # No base case
                  ^^^^^^^^^^^^^^^^
     File ".../no_base_case.py", line 2, in factorial
       return n * factorial(n - 1)  # No base case
                  ^^^^^^^^^^^^^^^^
     [Previous line repeated 996 more times]
   RecursionError: maximum recursion depth exceeded

Every call makes another call, and every call adds a frame. CPython
stops at its recursion limit and raises ``RecursionError``;
``sys.getrecursionlimit()`` returns 1000. The frames, in steps:

1. The module's frame: 1.
2. ``factorial`` frames shown in full: 3.
3. ``[Previous line repeated 996 more times]``: 996 more.
4. 1 + 3 + 996 = 1000, the recursion limit.

.. tip::

   Every recursive function needs a base case, and each recursive call
   must move toward it.


Functions as Values
====================================================

A Function Is an Object Like Any Other
--------------------------------------

.. code: lecture4/first_class_demo.py 6-17

.. code-block:: python
   :caption: From lecture4/first_class_demo.py

   def to_cm(m: float) -> float:
       return m * 100


   # Without parentheses: the function; with them: a call
   convert = to_cm
   print(convert(2.5), convert is to_cm)
   print(type(to_cm).__name__, type(to_cm(2.5)).__name__)

   # callable(): can this object be called?
   print(callable(to_cm), callable(print), callable(int))
   print(callable(2.5), callable("lidar"), callable(None))

.. output: first_class_demo.txt 1-4

.. code-block:: text

   250.0 True
   function float
   True True True
   False False False

- Functions are **first-class**: a name can refer to one, a list can
  hold one, a call can pass or return one.
- ``to_cm`` is the function; ``to_cm(2.5)`` is a call, and its result
  is a float.
- A **callable** is an object you can call with ``()``: functions,
  built-ins such as ``print``, and types such as ``int``.

A Function Can Take a Function as an Argument
---------------------------------------------

.. code: lecture4/first_class_demo.py 20-26

.. code-block:: python

   # A function that takes a function: a higher-order function
   def apply_to_scan(scan: list[float], func) -> list:
       return [func(r) for r in scan]


   print(apply_to_scan([2.5, 0.9], to_cm))
   print(apply_to_scan([2.5, 0.9], round))

.. output: first_class_demo.txt 5-6

.. code-block:: text

   [250.0, 90.0]
   [2, 1]

- A **higher-order function** takes a function as an argument, or
  returns one.
- ``apply_to_scan()`` does not know what ``func`` does. It calls it once
  per reading: 2.5 × 100 = 250.0 and 0.9 × 100 = 90.0.
- The second call passes the built-in ``round``: 2.5 rounds to 2 (to
  the even neighbor), and 0.9 to 1.
- Pass ``to_cm``, not ``to_cm()``: the parentheses would call it with no
  argument.

key= Calls a Function on Each Item Before Comparing
---------------------------------------------------

.. code: lecture4/first_class_demo.py 3-4,28-39

.. code-block:: python

   import math

   # key=: sorted(), min(), and max() call a function on each item
   waypoints = [(3.0, 0.5), (0.5, 0.0), (1.0, 2.0)]  # (x, y), chosen


   def distance(p: tuple[float, float]) -> float:
       return math.hypot(p[0], p[1])  # From (0, 0)


   print(sorted(waypoints, key=distance))
   print(max(waypoints, key=distance))
   maxima = {"robot_1": 2.1, "robot_2": 1.5, "robot_3": 1.0}
   print(max(maxima, key=maxima.get))  # Lecture 3, Exercise 4

.. output: first_class_demo.txt 7-9

.. code-block:: text

   [(0.5, 0.0), (1.0, 2.0), (3.0, 0.5)]
   (3.0, 0.5)
   robot_1

- ``sorted()``, ``min()``, and ``max()`` compare ``key(item)`` instead
  of the item.
- ``math.hypot(x, y)`` is the square root of x² + y². The distances:
  3.04, 0.50, and 2.24 m, so the order is ``(0.5, 0.0)``,
  ``(1.0, 2.0)``, ``(3.0, 0.5)``.
- ``maxima.get`` is the dictionary's own method: ``max()`` compares the
  values and returns the key. Lecture 3's Exercise 4, task 6, in one
  call.

map() and filter() Take a Function and an Iterable
--------------------------------------------------

.. code: lecture4/first_class_demo.py 42-50

.. code-block:: python

   # map() and filter() take a function and an iterable
   def is_valid(r: float) -> bool:
       return 0.12 <= r <= 12.0  # The lidar of Lecture 2


   scan = [2.5, 0.0, 3.1, 14.0, 1.2]
   print(list(filter(is_valid, scan)))
   print(list(map(to_cm, filter(is_valid, scan))))
   print([to_cm(r) for r in scan if is_valid(r)])

.. output: first_class_demo.txt 10-12

.. code-block:: text

   [2.5, 3.1, 1.2]
   [250.0, 310.0, 120.0]
   [250.0, 310.0, 120.0]

- ``filter(f, items)`` keeps the items for which ``f(item)`` is true;
  ``map(f, items)`` gives ``f(item)`` for each item.
- Both return iterators (Lecture 3), so ``list()`` collects the
  results.
- The comprehension gives the same list and reads in one direction.
  Prefer it when the function is short.

A Dictionary of Functions Picks the Code to Run
-----------------------------------------------

.. code: lecture4/first_class_demo.py 53-64

.. code-block:: python

   # A dictionary of functions: a dispatch table
   def forward(value: float) -> str:
       return f"drive at {value} m/s"


   def turn(value: float) -> str:
       return f"turn by {value} deg"


   handlers = {"forward": forward, "turn": turn}
   for word, value in [("forward", 0.2), ("turn", 90), ("turn", -45)]:
       print(handlers[word](value))

.. output: first_class_demo.txt 13-15

.. code-block:: text

   drive at 0.2 m/s
   turn by 90 deg
   turn by -45 deg

- ``handlers[word]`` looks up a function; ``(value)`` calls it. This is
  a **dispatch table**.
- :doc:`Lecture 2 <../lecture2/l2_lecture>`'s ``match`` picked a branch
  for each command. Here a new command is one more dictionary entry.

A Function Can Return a Function
--------------------------------

.. code: lecture4/first_class_demo.py 67-78

.. code-block:: python

   # A function that returns a function
   def make_range_check(min_m: float, max_m: float):
       def check(r: float) -> bool:
           return min_m <= r <= max_m

       return check


   lidar_ok = make_range_check(0.12, 12.0)  # The lidar of Lecture 2
   sonar_ok = make_range_check(0.02, 4.0)  # 4.0: the sonar of Lecture 2
   print(lidar_ok(5.0), sonar_ok(5.0))
   print(lidar_ok.__name__, lidar_ok is sonar_ok)

.. output: first_class_demo.txt 16-17

.. code-block:: text

   True False
   check False

- Each call of ``make_range_check()`` defines a new ``check`` and
  returns it. ``lidar_ok`` and ``sonar_ok`` are two function objects.
- 5.0 m is inside 0.12 to 12.0, but outside 0.02 to 4.0. The 0.02 is
  chosen for this example.
- ``check`` still reads ``min_m`` and ``max_m`` after
  ``make_range_check()`` has returned: a closure, in the next section.


Lambda Functions and Closures
====================================================

A Lambda Is a Function Written as One Expression
------------------------------------------------

.. code: lecture4/lambda_demo.py 3-11

.. code-block:: python
   :caption: From lecture4/lambda_demo.py

   waypoints = [(3.0, 0.5), (0.5, 0.0), (1.0, 2.0)]  # (x, y), chosen


   def by_y(p):
       return p[1]


   print(sorted(waypoints, key=by_y))
   print(sorted(waypoints, key=lambda p: p[1]))

.. output: lambda_demo.txt 1-2

.. code-block:: text

   [(0.5, 0.0), (3.0, 0.5), (1.0, 2.0)]
   [(0.5, 0.0), (3.0, 0.5), (1.0, 2.0)]

- ``lambda parameters: expression`` makes a function that returns the
  expression. The two sorts give the same list.
- The glossary: "An anonymous inline function consisting of a single
  expression which is evaluated when the function is called."
- Sorted by ``y``: 0.0, then 0.5, then 2.0. (Sorted by distance, in
  `key= Calls a Function on Each Item Before Comparing`_,
  ``(1.0, 2.0)`` came second.)

A Lambda Fits Where a Short Function Is an Argument
---------------------------------------------------

.. code: lecture4/lambda_demo.py 13-22

.. code-block:: python

   robots = [
       {"name": "waffle_01", "battery_pct": 87},
       {"name": "burger_02", "battery_pct": 15},
       {"name": "waffle_03", "battery_pct": 52},
   ]
   low_first = sorted(robots, key=lambda r: r["battery_pct"])
   print([r["name"] for r in low_first])
   print(min(robots, key=lambda r: r["battery_pct"])["name"])
   scan = [2.5, 0.0, 3.1, 14.0, 1.2]
   print(list(filter(lambda r: 0.12 <= r <= 12.0, scan)))

.. output: lambda_demo.txt 3-5

.. code-block:: text

   ['burger_02', 'waffle_03', 'waffle_01']
   burger_02
   [2.5, 3.1, 1.2]

- ``key=lambda r: r["battery_pct"]`` sorts the robots by battery: 15,
  52, 87.
- A lambda holds one expression and no statements: no ``=``
  assignment, no ``for`` or ``while`` statement.
- Ruff's default rules flag ``list(map(lambda ...))`` and suggest a
  comprehension:

.. code-block:: bash

   uv run ruff check lecture4/errors/map_lambda.py

.. output: ruff_map_lambda.txt 1-6

.. code-block:: text

   C417 Unnecessary `map()` usage (rewrite using a list comprehension)
    --> lecture4/errors/map_lambda.py:2:7
     |
   1 | scan = [2.5, 0.0, 3.1]
   2 | print(list(map(lambda r: r * 100, scan)))
     |       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A Lambda Is a Function Object; to Name It, Use def
--------------------------------------------------

.. code: lecture4/lambda_demo.py 24-27

.. code-block:: python

   # A lambda is a function object; its name is <lambda>
   rules = [lambda r: r < 0.5, lambda r: "low" if r < 0.5 else "ok"]
   print(type(rules[0]).__name__, rules[0].__name__)
   print([rule(0.3) for rule in rules])

.. output: lambda_demo.txt 6-7

.. code-block:: text

   function <lambda>
   [True, 'low']

- Every lambda's name is ``<lambda>``. A conditional expression (Lecture
  2) is allowed: it is one expression.
- PEP 8: "Always use a def statement instead of an assignment statement
  that binds a lambda expression directly to an identifier".

.. code-block:: bash

   uv run ruff check --extend-select E731 lecture4/errors/lambda_assign.py

.. output: ruff_lambda_assign.txt all

.. code-block:: text

   E731 Do not assign a `lambda` expression, use a `def`
    --> lecture4/errors/lambda_assign.py:1:1
     |
   1 | to_cm = lambda m: m * 100
     | ^^^^^^^^^^^^^^^^^^^^^^^^^
   2 | print(to_cm(2.5))
     |
   help: Rewrite `to_cm` as a `def`
     |
     - to_cm = lambda m: m * 100
   1 + def to_cm(m):
   2 +     return m * 100
   3 | print(to_cm(2.5))
     |
   note: This is an unsafe fix and may change runtime behavior

   Found 1 error.
   1 unsafe fix available with `--fix --unsafe-fixes`.

A Closure Keeps the Names of Its Enclosing Call
-----------------------------------------------

.. code: lecture4/closures_demo.py 4-20

.. code-block:: python
   :caption: From lecture4/closures_demo.py

   def make_counter():
       count = 0

       def increment():
           nonlocal count
           count += 1
           return count

       return increment


   scans = make_counter()
   print(scans(), scans(), scans())
   images = make_counter()
   print(images(), scans())
   print(scans.__code__.co_freevars)
   print(scans.__closure__[0].cell_contents)

.. output: closures_demo.txt 1-4

.. code-block:: text

   1 2 3
   1 4
   ('count',)
   4

- A **closure** is a nested function that uses a name of its enclosing
  function, and keeps it after that function has returned.
- ``scans`` counts 1, 2, 3. ``images`` starts at 1: each call of
  ``make_counter()`` made its own ``count``. Then ``scans`` gives 4.
- ``co_freevars`` lists the names the function keeps; ``__closure__``
  holds where it keeps them.

.. figure:: /_static/images/l4/closure_cell.svg
   :alt: Two rows. Tag 1: the name scans has an arrow to a function object increment; its __closure__ has an arrow to a cell for count, which has an arrow to the int 4. Tag 2: the name images has an arrow to another function object increment, whose cell for count has an arrow to the int 1.
   :width: 90%

   After ``print(images(), scans())`` printed ``1 4``. 1: ``scans`` and
   its cell. 2: ``images`` and its own cell.

Both calls of ``make_counter()`` have returned, and their frames are
gone. Each cell lives on because a function refers to it.
``nonlocal count`` makes ``count += 1`` write to the cell, not to a new
local name.

A closure also keeps the arguments of its enclosing call, as
`A Function Can Return a Function`_ showed:

.. code: lecture4/closures_demo.py 23-33

.. code-block:: python

   # A closure that remembers its arguments
   def make_range_check(min_m, max_m):
       def check(r):
           return min_m <= r <= max_m

       return check


   lidar_ok = make_range_check(0.12, 12.0)
   print(lidar_ok.__code__.co_freevars)
   print([c.cell_contents for c in lidar_ok.__closure__])

.. output: closures_demo.txt 5-6

.. code-block:: text

   ('max_m', 'min_m')
   [12.0, 0.12]

A Function Made in a Loop Reads the Loop Variable When Called
-------------------------------------------------------------

.. code: lecture4/errors/late_binding.py all

.. code-block:: python
   :caption: lecture4/errors/late_binding.py

   checks = []
   for limit in [0.5, 1.0, 2.0]:
       checks.append(lambda r: r < limit)
   print(limit, [check(0.8) for check in checks])

.. output: err_late_binding.txt all

.. code-block:: text

   2.0 [True, True, True]

- The intent: three checks, ``r < 0.5``, ``r < 1.0``, and ``r < 2.0``,
  so 0.8 should give ``False, True, True``.
- Each lambda reads ``limit`` when it is called, after the loop, when
  ``limit`` is 2.0.
- The FAQ: the loop variable "is accessed when the lambda is called",
  not when the lambda is made.

.. code-block:: bash

   uv run ruff check lecture4/errors/late_binding.py

.. output: ruff_late_binding.txt all

.. code-block:: text

   B023 Function definition does not bind loop variable `limit`
    --> lecture4/errors/late_binding.py:3:33
     |
   1 | checks = []
   2 | for limit in [0.5, 1.0, 2.0]:
   3 |     checks.append(lambda r: r < limit)
     |                                 ^^^^^
   4 | print(limit, [check(0.8) for check in checks])
     |

   Found 1 error.

A Default Value Keeps Each Loop Value
-------------------------------------

.. code: lecture4/closures_demo.py 35-39

.. code-block:: python
   :caption: The fix, in lecture4/closures_demo.py

   # A default value is evaluated when the lambda is made
   checks = []
   for limit in [0.5, 1.0, 2.0]:
       checks.append(lambda r, limit=limit: r < limit)
   print([check(0.8) for check in checks])

.. output: closures_demo.txt 7

.. code-block:: text

   [False, True, True]

- A default value is evaluated when the function is made
  (`A Default Value Is Created Once, When def Runs`_). Each lambda gets
  the value ``limit`` has at that moment: 0.5, 1.0, then 2.0.
- 0.8 < 0.5 is ``False``; 0.8 < 1.0 and 0.8 < 2.0 are ``True``.

.. tip::

   When you make callbacks in a loop, one per topic or one per robot,
   bind the loop value when you make each one.

Exercise 3 on the :doc:`exercises page <l4_exercises>` practices
recursion, lambdas, and closures.


Callbacks and Partial Functions
====================================================

A Callback Is a Function That Other Code Calls Later
----------------------------------------------------

.. code: lecture4/callbacks_demo.py 3-19

.. code-block:: python
   :caption: From lecture4/callbacks_demo.py

   from collections.abc import Callable
   from functools import partial

   subscribers: dict[str, list[Callable]] = {}  # Topic -> callbacks


   def subscribe(topic: str, callback: Callable) -> None:
       """Call callback(msg) for each message published on topic."""
       if topic not in subscribers:
           subscribers[topic] = []
       subscribers[topic].append(callback)


   def publish(topic: str, msg) -> None:
       """Call every callback subscribed to topic, in order."""
       for callback in subscribers.get(topic, []):
           callback(msg)

- The glossary: a **callback** is "A subroutine function which is
  passed as an argument to be executed at some point in the future."
- ``subscribe()`` stores a function in a list, per topic. ``publish()``
  calls every stored function with the message.
- ``Callable`` (from ``collections.abc``) is the type hint for
  something you can call.

Publishing a Message Calls Each Subscriber's Callback
-----------------------------------------------------

.. code: lecture4/callbacks_demo.py 22-35

.. code-block:: python

   def on_scan(ranges_m: list[float]) -> None:
       print(f"on_scan: nearest {min(ranges_m)} m")


   def on_battery(pct: int) -> None:
       print(f"on_battery: {pct}%")


   subscribe("/scan", on_scan)
   subscribe("/battery", on_battery)
   publish("/scan", [2.5, 0.9, 3.1])
   publish("/battery", 87)  # 87%: chosen for this example
   publish("/scan", [0.4, 1.2])
   publish("/odom", (1.0, 2.0))  # No subscriber: nothing happens

.. output: callbacks_demo.txt 1-3

.. code-block:: text

   on_scan: nearest 0.9 m
   on_battery: 87%
   on_scan: nearest 0.4 m

- Your code never calls ``on_scan()``. ``publish()`` does, once per
  message on ``/scan``.
- ``/odom`` has no subscriber, so its message reaches no callback.

A Timer Calls Its Callback Once per Period
------------------------------------------

.. code: lecture4/callbacks_demo.py 38-48

.. code-block:: python

   # A timer calls its callback once per period (simulated time)
   def run_timer(period_s: float, callback: Callable, ticks: int) -> None:
       for k in range(1, ticks + 1):
           callback(k * period_s)


   def on_tick(t: float) -> None:
       print(f"tick at t = {t:.1f} s")


   run_timer(0.5, on_tick, 3)  # 0.5 s and 3 ticks: chosen

.. output: callbacks_demo.txt 4-6

.. code-block:: text

   tick at t = 0.5 s
   tick at t = 1.0 s
   tick at t = 1.5 s

- The time is simulated: ``k * period_s`` for ``k`` = 1, 2, 3 gives 0.5,
  1.0, and 1.5 s, without waiting.
- rclpy's ``create_timer()`` takes a period and a callback. Its
  docstring: "If autostart is ``True`` (the default), the timer will be
  started and every ``timer_period_sec`` number of seconds the provided
  callback function will be called."

.. output: rclpy_create_timer.txt all

.. code-block:: python
   :caption: From rclpy/node.py in ROS 2 Jazzy

   def create_timer(
       self,
       timer_period_sec: float,
       callback: Callable,
       callback_group: Optional[CallbackGroup] = None,
       clock: Optional[Clock] = None,
       autostart: bool = True,
   ) -> Timer:

Reading a Real Signature: rclpy's create_subscription()
-------------------------------------------------------

.. output: rclpy_create_subscription.txt all

.. code-block:: python
   :caption: From rclpy/node.py in ROS 2 Jazzy

   def create_subscription(
       self,
       msg_type,
       topic: str,
       callback: Callable[[MsgType], None],
       qos_profile: Union[QoSProfile, int],
       *,
       callback_group: Optional[CallbackGroup] = None,
       event_callbacks: Optional[SubscriptionEventCallbacks] = None,
       qos_overriding_options: Optional[QoSOverridingOptions] = None,
       raw: bool = False,
       content_filter_options: Optional[ContentFilterOptions] = None
   ) -> Subscription:

- ``callback: Callable[[MsgType], None]``: a callable that takes one
  message and returns nothing, like ``on_scan()``.
- The bare ``*``: the five parameters after it are keyword-only, each
  with a default.
- ``Optional[X]`` is the older spelling of ``X | None``. ``self``:
  Lecture 6. Lecture 8 calls this method.

functools.partial Fills In Some Arguments Now
---------------------------------------------

.. code: lecture4/partial_demo.py 3-21

.. code-block:: python
   :caption: From lecture4/partial_demo.py

   from functools import partial


   def in_range(r: float, min_m: float, max_m: float) -> bool:
       return min_m <= r <= max_m


   lidar_ok = partial(in_range, min_m=0.12, max_m=12.0)
   print(lidar_ok(5.0), lidar_ok(14.0))
   print(type(lidar_ok).__name__)
   print(lidar_ok.func.__name__, lidar_ok.args, lidar_ok.keywords)


   def drive_time(distance_m: float, speed_mps: float) -> float:
       return distance_m / speed_mps


   one_meter = partial(drive_time, 1.0)  # Fills distance_m, the first
   print(one_meter(0.25), one_meter.args)

.. output: partial_demo.txt 1-4

.. code-block:: text

   True False
   partial
   in_range () {'min_m': 0.12, 'max_m': 12.0}
   4.0 (1.0,)

- ``partial(f, ...)`` returns a new callable. Calling it calls ``f``
  with the stored arguments plus the new ones.
- ``.func``, ``.args``, and ``.keywords`` show what it stores. Stored
  positional arguments fill the first parameters: ``one_meter(0.25)``
  is ``drive_time(1.0, 0.25)``, 1.0 / 0.25 = 4.0.

A Callback That Needs More Than the Message
-------------------------------------------

.. code: lecture4/callbacks_demo.py 51-58

.. code-block:: python

   # A callback that needs more than the message
   def log_scan(robot: str, ranges_m: list[float]) -> None:
       print(f"{robot}: {len(ranges_m)} readings")


   subscribe("/scan", partial(log_scan, "waffle_01"))
   subscribe("/scan", lambda msg: log_scan("burger_02", msg))
   publish("/scan", [1.5, 2.0])

.. output: callbacks_demo.txt 7-9

.. code-block:: text

   on_scan: nearest 1.5 m
   waffle_01: 2 readings
   burger_02: 2 readings

- ``publish()`` passes one argument, the message. ``log_scan()`` also
  needs the robot's name.
- ``partial(log_scan, "waffle_01")`` fills in the name now; ``publish()``
  adds the message later. The lambda does the same.
- ``/scan`` now has three callbacks, called in the order they
  subscribed: ``on_scan``, then the partial, then the lambda.

Three Ways to Fill In an Argument
---------------------------------

.. code: lecture4/partial_demo.py 24-38

.. code-block:: python

   # A closure that fills in speed_mps
   def make_driver(speed_mps):
       def drive(distance_m):
           return drive_time(distance_m, speed_mps)

       return drive


   # Three ways to fill in an argument
   fillers = [
       partial(drive_time, speed_mps=0.25),
       lambda d: drive_time(d, 0.25),
       make_driver(0.25),
   ]
   print([f(3.0) for f in fillers])

.. output: partial_demo.txt 5

.. code-block:: text

   [12.0, 12.0, 12.0]

.. list-table::
   :header-rows: 1
   :widths: 16 42 42
   :class: compact-table

   * - Way
     - Good for
     - Note
   * - ``partial``
     - Fixing arguments of an existing function
     - Shows them in ``.args`` and ``.keywords``
   * - ``lambda``
     - A short expression, used once
     - One expression only
   * - Closure
     - Logic or state of your own
     - A full ``def``; can have a docstring


Putting It All Together
====================================================

Exercise 4 on the :doc:`exercises page <l4_exercises>` is for after
class. It builds a small publish and subscribe system for two robots,
with closures, a range filter factory, and ``functools.partial``:
several sections of this lecture in one program.


Next Class
====================================================

:doc:`L5: NumPy and Matplotlib for Robotics </lectures/lecture5/l5_index>`
(Tuesday, March 2):

- NumPy arrays versus Python lists
- Shape, data type, indexing, slicing, and Boolean masks
- Vectorized operations and broadcasting
- Vectors, rotation matrices, and 2D transforms
- A laser scan as an array: the nearest obstacle and invalid readings
- Plotting a trajectory and a scan with Matplotlib
- Installing NumPy next to ROS 2
