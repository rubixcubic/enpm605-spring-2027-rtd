====================================================
Lecture
====================================================

Every program on this page is in the ``lecture2/`` folder of the
`course repository <https://github.com/rubixcubic/enpm605-spring-2027-python>`_.
Run each one from the root of the repository, for example:

.. code-block:: bash

   uv run lecture2/packages_demo.py

Every output on this page was printed by running the program with the
course environment (CPython 3.12.3 on Ubuntu 24.04). In paths, ``<repo>``
stands for the folder of the course repository on your PC, and ``...``
marks a path shortened to fit.


Packages and Modules
====================================================

A Module Is a File, a Package Is a Folder
-----------------------------------------

- A **module** is one ``.py`` file. Another file can ``import`` it and
  use the names it defines (variables, functions, classes).
- A **package** is a folder of modules. The Python documentation: "A
  regular package is typically implemented as a directory containing an
  ``__init__.py`` file."
- Python runs ``__init__.py`` when the package is imported. Ours holds
  only a docstring.

Two folders of ``lecture2``, printed by ``tree``:

.. code-block:: text

   lecture2/sensors
   ├── __init__.py
   ├── lidar.py
   └── sonar.py
   lecture2/tools
   ├── range_check_insert.py
   └── range_check.py

``sensors/`` is a package with two modules. ``tools/`` is a plain folder,
used later in this section. Each module defines two names, the sensor's
name and its longest range in meters, and both modules use the same two
names.

.. code-block:: python
   :caption: lecture2/sensors/lidar.py

   """A 2D lidar. The values are chosen for this example."""

   NAME = "lidar"
   MAX_RANGE_M = 12.0

   if __name__ == "__main__":
       print(f"Self-test: {NAME} reads up to {MAX_RANGE_M} m")

.. code-block:: python
   :caption: lecture2/sensors/sonar.py

   """An ultrasonic range sensor. The values are chosen for this example."""

   NAME = "sonar"
   MAX_RANGE_M = 4.0

The last two lines of ``lidar.py`` are the **main guard**, explained at
the end of this section.

Four Ways to Import
-------------------

.. code-block:: python
   :caption: lecture2/packages_demo.py

   """Four ways to import names from the sensors package."""

   import sensors.lidar
   from sensors import lidar
   from sensors.lidar import MAX_RANGE_M
   from sensors.sonar import MAX_RANGE_M as SONAR_MAX_RANGE_M

   print(sensors.lidar.MAX_RANGE_M)  # Full path to the name
   print(lidar.MAX_RANGE_M)  # The module, by its own name
   print(MAX_RANGE_M)  # The name itself
   print(SONAR_MAX_RANGE_M)  # The name itself, renamed

.. code-block:: text

   12.0
   12.0
   12.0
   4.0

1. ``import sensors.lidar``: you write the full path,
   ``sensors.lidar.MAX_RANGE_M``, every time.
2. ``from sensors import lidar``: the module under its own name,
   ``lidar``.
3. ``from sensors.lidar import MAX_RANGE_M``: the name itself, with no
   prefix.
4. ``as`` gives the imported name a new name, here to keep the two
   ``MAX_RANGE_M`` apart. Lecture 5 writes ``import numpy as np`` the
   same way.

A Wildcard Import Replaces Names Without a Warning
--------------------------------------------------

.. code-block:: python
   :caption: lecture2/wildcard_demo.py

   """A wildcard import replaces names without a warning."""

   from sensors.lidar import *
   from sensors.sonar import *

   print(NAME, MAX_RANGE_M)

.. code-block:: text

   sonar 4.0

- ``import *`` copies every public name of a module into your file.
- The second line rebinds ``NAME`` and ``MAX_RANGE_M`` to the sonar's
  values. The program prints the sonar's range, ``4.0``, and raises no
  error.
- A reader cannot tell which module a name came from without opening
  both modules.

.. tip::

   Import the names you use, and rename them with ``as`` when two modules
   use the same name.

With its default rules, Ruff 0.17 reports nothing:

.. code-block:: bash

   uv run ruff check --output-format concise lecture2/wildcard_demo.py

.. code-block:: text

   All checks passed!

Rules ``F403`` and ``F405`` report it. Add them with ``--extend-select``
(Lecture 1):

.. code-block:: bash

   uv run ruff check --output-format concise \
         --extend-select F403,F405 lecture2/wildcard_demo.py

.. code-block:: text

   lecture2/wildcard_demo.py:3:1: F403 `from sensors.lidar import *` used; unable to detect undefined names
   lecture2/wildcard_demo.py:4:1: F403 `from sensors.sonar import *` used; unable to detect undefined names
   lecture2/wildcard_demo.py:6:7: F405 `NAME` may be undefined, or defined from star imports
   lecture2/wildcard_demo.py:6:13: F405 `MAX_RANGE_M` may be undefined, or defined from star imports
   Found 4 errors.

Python Looks for Modules in the Folders of sys.path
---------------------------------------------------

.. code-block:: python
   :caption: lecture2/search_path_demo.py

   """Print the module search path."""

   import sys

   for folder in sys.path:
       print(folder)

.. code-block:: text

   <repo>/lecture2
   /usr/lib/python312.zip
   /usr/lib/python3.12
   /usr/lib/python3.12/lib-dynload
   <repo>/.venv/lib/python3.12/site-packages
   ...

The lines after line 5 depend on the PC (Ubuntu's packages, from
``--system-site-packages``) and are cut.

- ``sys.path`` is the list of folders Python searches, in order, for each
  ``import``. The ``for`` loop prints one folder per line
  (:doc:`Lecture 3 </lectures/lecture3/l3_index>` covers loops).
- Line 1 is the folder of the program you ran. The documentation:
  "python script.py command line: prepend the script's directory." So
  ``packages_demo.py`` finds ``sensors/`` next to it.
- Lines 2 to 4 are Python's standard library: ``import math`` finds
  ``math`` there.
- Line 5 is ``.venv``. ``uv sync`` installed NumPy there, as a package: a
  folder ``numpy/`` with an ``__init__.py``.

.. code-block:: bash

   uv run python -c "import numpy; print(numpy.__file__)"

.. code-block:: text

   <repo>/.venv/lib/python3.12/site-packages/numpy/__init__.py

Your own packages, the standard library, and installed packages are all
imported the same way. Only the folder on ``sys.path`` differs.

A Package in Another Folder Is Not Found
----------------------------------------

.. code-block:: python
   :caption: lecture2/tools/range_check.py

   """Import the sensors package from a folder that does not contain it."""

   from sensors.lidar import MAX_RANGE_M

   print(f"Lidar range: {MAX_RANGE_M} m")

.. code-block:: bash

   uv run lecture2/tools/range_check.py

.. code-block:: text

   Traceback (most recent call last):
     File ".../lecture2/tools/range_check.py", line 3, in <module>
       from sensors.lidar import MAX_RANGE_M
   ModuleNotFoundError: No module named 'sensors'

The first folder on ``sys.path`` is now ``lecture2/tools/``, which has no
``sensors/``. No other folder on the list has one either. The next two
sections show two ways to tell Python where the package is.

Option 1: PYTHONPATH Adds Folders to the Search
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``PYTHONPATH`` is an environment variable. The documentation: "Augment
the default search path for module files." Its folders go on
``sys.path`` after the program's folder, before the standard library.
Set it for one command, from the root of the course repository:

.. code-block:: bash

   PYTHONPATH=lecture2 uv run lecture2/tools/range_check.py

.. code-block:: text

   Lidar range: 12.0 m

``export PYTHONPATH=lecture2`` keeps it for every later command in that
terminal. A new terminal starts without it. Several folders are
separated by colons, as in ``PATH``.

**ROS 2 uses PYTHONPATH.** ``rclpy`` is the ROS 2 library for Python
(:doc:`Lecture 8 </lectures/lecture8/l8_index>`). In a terminal where
ROS 2 is not sourced, Ubuntu's Python cannot find it:

.. code-block:: bash

   python3 -c "import rclpy"

.. code-block:: text

   ModuleNotFoundError: No module named 'rclpy'

Sourcing ROS 2 sets ``PYTHONPATH``. Print it, one folder per line:

.. code-block:: bash

   source /opt/ros/jazzy/setup.bash
   echo "$PYTHONPATH" | tr ":" "\n"

.. code-block:: text

   /opt/ros/jazzy/opt/gz_sim_vendor/lib/python
   /opt/ros/jazzy/opt/sdformat_vendor/lib/python
   /opt/ros/jazzy/opt/gz_transport_vendor/lib/python
   /opt/ros/jazzy/opt/gz_msgs_vendor/lib/python
   /opt/ros/jazzy/opt/gz_math_vendor/lib/python
   /opt/ros/jazzy/lib/python3.12/site-packages

The last folder holds ``rclpy``. After sourcing, ``import rclpy`` loads
``/opt/ros/jazzy/lib/python3.12/site-packages/rclpy/__init__.py``.

Option 2: Change sys.path Inside the Program
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python
   :caption: lecture2/tools/range_check_insert.py

   """Find the sensors package by editing sys.path inside the program."""

   import sys
   from pathlib import Path

   sys.path.insert(0, str(Path(__file__).resolve().parents[1]))

   from sensors.lidar import MAX_RANGE_M

   print(f"Lidar range: {MAX_RANGE_M} m")

.. code-block:: text

   Lidar range: 12.0 m

- ``__file__`` is the path of this file. ``.resolve()`` makes it
  absolute, and ``.parents[1]`` goes up two levels: from
  ``range_check_insert.py`` to ``tools/``, then to ``lecture2/``.
- ``sys.path.insert(0, ...)`` puts ``lecture2/`` first on the list, so
  the next ``import`` finds ``sensors/``.
- The cost: every program that needs the package repeats this line, and
  it breaks when a folder moves. The course's
  ``tests/test_verify_setup.py`` uses it to reach ``lecture1/``.

Where Python Finds a Package
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. list-table::
   :header-rows: 1
   :widths: 25 40 35
   :class: compact-table

   * - Where the package is
     - How Python finds it
     - Who sets it up
   * - Next to the program
     - First folder on ``sys.path``
     - Python, always
   * - In ``.venv``
     - The ``site-packages`` folder of ``.venv``
     - ``uv sync``
   * - Anywhere else
     - ``PYTHONPATH``
     - You, or ROS 2's ``setup.bash``
   * - Anywhere else
     - ``sys.path.insert()``
     - The program itself

.. tip::

   In this course, keep a program next to the package it imports, and run
   it with ``uv run`` from the root of the repository. From Lecture 8,
   ROS 2 sets ``PYTHONPATH`` for you.

__name__ Says How a Module Was Started
--------------------------------------

.. code-block:: python
   :caption: lecture2/name_demo.py

   """Print __name__ in the file that runs and in a module it imports."""

   import sensors.lidar

   print(f"In name_demo.py, __name__ is {__name__!r}")
   print(f"In lidar.py, __name__ is {sensors.lidar.__name__!r}")

.. code-block:: text

   In name_demo.py, __name__ is '__main__'
   In lidar.py, __name__ is 'sensors.lidar'

- Python sets the variable ``__name__`` in every module.
- The file you run gets the value ``"__main__"``.
- A module that is imported gets its full name, here
  ``"sensors.lidar"``.
- ``!r`` in an f-string prints the value with its quotes (see
  `f-Strings Put Values into Text`_).

The Main Guard Runs Code Only When the File Is Run
--------------------------------------------------

The end of ``lecture2/sensors/lidar.py``:

.. code-block:: python

   if __name__ == "__main__":
       print(f"Self-test: {NAME} reads up to {MAX_RANGE_M} m")

Run directly:

.. code-block:: bash

   uv run lecture2/sensors/lidar.py

.. code-block:: text

   Self-test: lidar reads up to 12.0 m

Imported by ``packages_demo.py``, the guarded line is skipped:
``uv run lecture2/packages_demo.py`` prints only its own four lines
(``12.0``, ``12.0``, ``12.0``, ``4.0``).

- Run directly, ``__name__`` is ``"__main__"``, so the guarded line runs.
- Imported, ``__name__`` is ``"sensors.lidar"``, so the guarded line is
  skipped.

.. tip::

   Put a module's own test or demo under the main guard. Importing the
   module then never runs it.


Indentation
====================================================

Indentation Marks a Block
-------------------------

.. code-block:: python
   :caption: lecture2/indentation_demo.py

   """Indentation marks which lines belong to the if block."""

   battery_pct = 15  # Chosen for this example

   if battery_pct < 20:
       print("Battery low")
       print("Returning to the dock")
   print("This line runs every time")

.. code-block:: text

   Battery low
   Returning to the dock
   This line runs every time

- A line that opens a block ends with a colon: ``if``, ``else``, ``for``,
  ``def``, and others.
- Every line of the block is indented by the same amount. The first line
  back at the outer level ends the block.
- PEP 8: "Use 4 spaces per indentation level."

The same program in C++:

.. code-block:: cpp

   int battery_pct{15};  // Chosen for this example

   if (battery_pct < 20) {
       std::cout << "Battery low\n";
       std::cout << "Returning to the dock\n";
   }
   std::cout << "This line runs every time\n";

C++ marks the block with braces. Its indentation is only for the reader:
the compiler ignores it. In Python, the indentation **is** the block.
Moving the last ``print()`` 4 spaces to the right would put it inside
the ``if``, and the program would print nothing at all for a battery at
20% or more.

Indentation Errors
------------------

Python finds these errors while it compiles the file
(:doc:`Lecture 1 <../lecture1/l1_lecture>`, Python Execution Pipeline),
so no line of the program runs.

.. code-block:: python
   :caption: lecture2/errors/missing_indent.py

   battery_pct = 15

   if battery_pct < 20:
   print("Battery low")

.. code-block:: text

     File ".../missing_indent.py", line 4
       print("Battery low")
       ^
   IndentationError: expected an indented block after 'if' statement on line 3

The ``if`` on line 3 ends with a colon, so Python expects an indented
block on line 4. A block cannot be empty. To leave one empty for now,
write ``pass``, a statement that does nothing.

.. code-block:: python
   :caption: lecture2/errors/unexpected_indent.py

   battery_pct = 15
       print(battery_pct)

.. code-block:: text

     File ".../unexpected_indent.py", line 2
       print(battery_pct)
   IndentationError: unexpected indent

Line 2 is indented, but no line above it opens a block. A new statement
may be indented only where a block starts. Inside parentheses, a line
that continues a statement can be indented as you like.

In ``lecture2/errors/tabs_and_spaces.py``, line 4 is indented with 4
spaces and line 5 with one tab. They look the same in an editor:

.. code-block:: python

   battery_pct = 15

   if battery_pct < 20:
       print("Battery low")
       print("Returning to the dock")

.. code-block:: text

     File ".../tabs_and_spaces.py", line 5
       print("Returning to the dock")
   TabError: inconsistent use of tabs and spaces in indentation

- ``Battery low`` is not printed: here too, Python stops before the
  program runs.
- PEP 8: "Spaces are the preferred indentation method." and "Python
  disallows mixing tabs and spaces for indentation."
- VS Code inserts 4 spaces when you press Tab in a Python file, and
  ``uv run ruff format`` (Lecture 1) rewrites the indentation of a file
  that parses.


Boolean Type
====================================================

bool Has Two Values and Is a Kind of int
----------------------------------------

.. code-block:: python
   :caption: From lecture2/boolean_demo.py

   # bool is a kind of int
   print(True + True)
   print(isinstance(True, int))
   print(int(False))

.. code-block:: text

   2
   True
   0

- The type ``bool`` has exactly two objects: ``True`` and ``False``.
- Lecture 1's table listed ``bool`` as a subclass of ``int``: ``True``
  acts as ``1`` and ``False`` as ``0``. So ``True + True`` is ``2``.
- ``isinstance(x, T)`` is ``True`` when ``x`` is an object of type ``T``
  or of a subclass of ``T``.
- Comparisons (``<``, ``==``, and the others) return a ``bool``.

Every Object Counts as True or False in a Condition
---------------------------------------------------

.. code-block:: python
   :caption: From lecture2/boolean_demo.py

   # Falsy values
   print(bool(0), bool(0.0), bool(""), bool([]), bool({}), bool(None))

   # Truthy values
   print(bool(1), bool(-2), bool("hi"), bool([0]), bool(" "), bool(0.001))
   print(bool("False"))  # A non-empty string

.. code-block:: text

   False False False False False False
   True True True True True True
   True

- ``bool(x)`` gives the truth value that ``if x:`` uses. A value whose
  truth value is ``False`` is called **falsy**; any other value is
  **truthy**.
- The documentation lists "most of the built-in objects considered
  false": ``None`` and ``False``, zero of any numeric type (``0``,
  ``0.0``, and others), and empty sequences and collections (``""``,
  ``[]``, ``{}``, and others).
- ``[0]`` is a list with one item, so it is truthy. ``" "`` holds a
  space, so it is truthy.
- ``"False"`` is a non-empty string, so it is truthy. A setting read from
  a text file as ``"False"`` does not turn anything off.

A Zero Is a Value, Not a Missing Value
--------------------------------------

.. code-block:: python
   :caption: From lecture2/boolean_demo.py

   # A position of 0.0 m is a value, not a missing one
   goal_x_m = 0.0  # The goal is at the origin
   if not goal_x_m:
       print("not goal_x_m: no goal set (wrong)")
   if goal_x_m is None:
       print("No goal set")
   else:
       print(f"Goal: x = {goal_x_m} m")

.. code-block:: text

   not goal_x_m: no goal set (wrong)
   Goal: x = 0.0 m

- ``0.0`` is falsy, so ``if not goal_x_m:`` treats a goal at the origin
  as no goal.
- When ``None`` means "no value" (Lecture 1), test for it with
  ``is None``. A goal at ``0.0`` m is then kept.
- For a collection, empty and missing usually mean the same thing, so
  ``if readings:`` is the usual test for "has at least one reading".

.. warning::

   Use truthiness when zero and empty should count as "nothing". When
   zero is a real measurement, compare with ``is None`` or with a number.


Operators
====================================================

All the programs in this section are in ``lecture2/operators_demo.py``.

Arithmetic Operators
--------------------

.. list-table::
   :header-rows: 1
   :widths: 15 45 40
   :class: compact-table

   * - Operator
     - Operation
     - Example
   * - ``+``
     - Addition
     - ``7 + 3``
   * - ``-``
     - Subtraction
     - ``7 - 3``
   * - ``*``
     - Multiplication
     - ``7 * 3``
   * - ``/``
     - Division
     - ``7 / 3``
   * - ``//``
     - Floor division
     - ``7 // 3``
   * - ``%``
     - Modulo (remainder)
     - ``7 % 3``
   * - ``**``
     - Power
     - ``2**10``

.. code-block:: python

   # Arithmetic
   print(7 + 3, 7 - 3, 7 * 3)
   print(7 / 3)
   print(7 // 3, 7 % 3, 2**10)

.. code-block:: text

   10 4 21
   2.3333333333333335
   2 1 1024

- The output gives the results in the order of the table.
- ``/`` always returns a ``float``, even when the division is exact.
- ``//`` returns how many whole times 3 fits in 7: 2. ``%`` returns what
  is left: 7 is 1 more than 2 times 3.
- The last digit of ``2.3333333333333335`` comes from float precision
  (see `Floats Are Approximate`_).

Floor Division and Modulo Round Toward Negative Infinity
--------------------------------------------------------

.. code-block:: python

   # Floor division and modulo round toward negative infinity
   print(7 // 2, -7 // 2)
   print(7 % 3, -7 % 3)
   heading_deg = -30
   print(heading_deg % 360)
   print(370 % 360)

.. code-block:: text

   3 -4
   1 2
   330
   10

The documentation, on ``//``: "The result is always rounded towards
minus infinity". ``-7 / 2`` is ``-3.5``. Rounding down gives ``-4``, not
``-3``.

``-7 % 3``, in steps:

1. ``-7 // 3`` is ``-3``, because ``-7 / 3`` is about ``-2.33``, rounded
   down.
2. 3 times ``-3`` is ``-9``.
3. ``-7`` is 2 more than ``-9``, so ``-7 % 3`` is ``2``.

With a positive right side, ``%`` never returns a negative number. A
heading of ``-30`` degrees and one of ``330`` degrees point the same way,
and ``heading_deg % 360`` always gives a value from 0 up to, but not
including, 360.

Precedence Decides Which Operator Runs First
--------------------------------------------

.. code-block:: python

   # Precedence
   print(2 + 3 * 4)
   print(-(2**2))
   print((-2) ** 2)

.. code-block:: text

   14
   -4
   4

- ``*`` runs before ``+``, as in algebra: ``3 * 4`` is ``12``, then
  ``2 + 12`` is ``14``.
- ``**`` runs before a minus sign on its left. The documentation:
  "-1**2 results in -1."
- Ruff writes the parentheses for you. ``uv run ruff format`` on a file
  holding ``print(-2**2)`` changes the line:

  .. code-block:: text

     -print(-2**2)
     +print(-(2**2))

- To square ``-2``, write ``(-2) ** 2``. When you are not sure of the
  order, add parentheses.

Augmented Assignment Updates a Name
-----------------------------------

.. code-block:: python

   # Augmented assignment
   distance_m = 0.0
   distance_m += 0.5
   distance_m += 0.5
   distance_m *= 2
   print(distance_m)

.. code-block:: text

   2.0

``x += 0.5`` means ``x = x + 0.5``. Every arithmetic operator has one:
``-=``, ``*=``, ``/=``, ``//=``, ``%=``, ``**=``. The value of
``distance_m``, in steps:

1. ``0.0 + 0.5`` is ``0.5``.
2. ``0.5 + 0.5`` is ``1.0``.
3. ``1.0 * 2`` is ``2.0``.

.. warning::

   Python has no ``++`` or ``--`` operator. In Python, ``++x`` is
   ``+(+x)``, which leaves ``x`` unchanged. Ruff's rule ``B002``, on by
   default, reports it.

Comparison Operators Return a bool
----------------------------------

.. list-table::
   :header-rows: 1
   :widths: 25 75
   :class: compact-table

   * - Operator
     - Meaning
   * - ``==``
     - Equal
   * - ``!=``
     - Not equal
   * - ``<``, ``>``
     - Less than, greater than
   * - ``<=``, ``>=``
     - At most, at least

.. code-block:: python

   # Comparison
   range_m = 3.2
   print(range_m == 3.2, range_m != 3.2)
   print(range_m > 1.0, range_m <= 1.0)
   print(0.12 <= range_m <= 12.0)  # Within the lidar's range

.. code-block:: text

   True False
   True False
   True

- ``0.12 <= range_m <= 12.0`` is a **chained comparison**. It means
  ``0.12 <= range_m and range_m <= 12.0``. Here it checks that a reading
  lies between the lidar's shortest and longest range (both chosen for
  this example).
- ``=`` binds a name; ``==`` compares two values.
- Lecture 1 covered ``is`` and ``is not``, which compare identity, not
  value.

Logical Operators Combine Conditions
------------------------------------

.. list-table::
   :header-rows: 1
   :class: compact-table

   * - ``a``
     - ``b``
     - ``a and b``
     - ``a or b``
   * - ``True``
     - ``True``
     - ``True``
     - ``True``
   * - ``True``
     - ``False``
     - ``False``
     - ``True``
   * - ``False``
     - ``True``
     - ``False``
     - ``True``
   * - ``False``
     - ``False``
     - ``False``
     - ``False``

.. code-block:: python

   # Logical
   battery_ok = True
   obstacle_ahead = False
   print(battery_ok and not obstacle_ahead)
   print(battery_ok or obstacle_ahead)
   print(not battery_ok)

.. code-block:: text

   True
   True
   False

- ``a and b`` is true only when both are true. ``a or b`` is true when
  at least one is true. ``not a`` reverses ``a``.
- ``battery_ok and not obstacle_ahead`` reads as "the battery is fine and
  nothing is ahead": the robot may drive.
- Python spells these as words. C++'s ``&&``, ``||``, and ``!`` are
  syntax errors in Python.

and and or Return One of Their Operands
---------------------------------------

.. code-block:: python

   # and and or return one of their operands
   label = "lidar"
   count = 0
   missing = ""
   print(label and count)
   print(label and "sonar")
   print(missing or "default")
   print(count or None)

.. code-block:: text

   0
   sonar
   default
   None

The documentation's rules: ``x or y`` gives "if x is true, then x, else
y"; ``x and y`` gives "if x is false, then x, else y". They "return the
last evaluated argument", not always ``True`` or ``False``.

- ``label and count``: ``"lidar"`` is truthy, so the result is
  ``count``, ``0``.
- ``label and "sonar"``: ``"lidar"`` is truthy, so the result is
  ``"sonar"``.
- ``missing or "default"``: ``""`` is falsy, so the result is
  ``"default"``.
- ``count or None``: ``0`` is falsy, so the result is ``None``, even
  though ``None`` is falsy too.
- ``not`` always returns a ``bool``.

A Default Written with or Also Replaces 0.0
-------------------------------------------

.. code-block:: python

   # A default with or replaces 0.0 too
   requested_speed = 0.0  # The operator asks the robot to stop
   speed = requested_speed or 0.5  # 0.5 m/s when no speed is given
   print(speed)
   speed = 0.5 if requested_speed is None else requested_speed
   print(speed)

.. code-block:: text

   0.5
   0.0

- The plan: use 0.5 m/s (chosen for this example) when no speed is given
  (``None``).
- ``requested_speed or 0.5`` gives ``0.5``, because ``0.0`` is falsy.
  The operator asked the robot to stop, and it drives at 0.5 m/s.
- The second version tests for ``None`` only, so ``0.0`` is kept.
  `A Conditional Expression Picks One of Two Values`_ explains the
  ``... if ... else ...`` expression.

.. warning::

   ``x or default`` replaces every falsy ``x``: ``None``, but also ``0``,
   ``0.0``, and ``""``. Use it only when all of them mean "not given".

Short-Circuit Evaluation Skips the Right Side
---------------------------------------------

.. code-block:: python

   # Short-circuit evaluation
   readings = []
   print(len(readings) > 0 and readings[0] > 1.0)

.. code-block:: text

   False

Without the check, in ``lecture2/errors/empty_index.py``:

.. code-block:: python

   readings = []
   print(readings[0] > 1.0)

.. code-block:: text

   Traceback (most recent call last):
     File ".../empty_index.py", line 2, in <module>
       print(readings[0] > 1.0)
             ~~~~~~~~^^^
   IndexError: list index out of range

- ``readings[0]`` is the first item of the list. An empty list has none,
  so reading it raises an ``IndexError``.
- The documentation, on ``and``: "it only evaluates the second argument
  if the first one is true."
- ``len(readings) > 0`` is ``False``, so ``readings[0]`` never runs. Put
  the check that guards the risky part first.

Membership Operators: in and not in
-----------------------------------

.. code-block:: python

   # Membership
   topic = "/robot1/cmd_vel"
   print("cmd_vel" in topic)
   print("scan" not in topic)
   print(3 in [1, 2, 3])

.. code-block:: text

   True
   True
   True

- ``x in s`` is ``True`` when ``s`` contains ``x``. ``x not in s`` is
  the opposite.
- For strings, ``in`` looks for a substring: ``"cmd_vel"`` appears in the
  topic name ``"/robot1/cmd_vel"``.
- For a list, ``in`` looks for an item equal to ``x``. Lecture 3 covers
  lists.


Numeric Types
====================================================

All the programs in this section are in ``lecture2/numeric_demo.py``.

Python Has Three Numeric Types
------------------------------

.. list-table::
   :header-rows: 1
   :widths: 15 35 25 25
   :class: compact-table

   * - Type
     - Holds
     - Size
     - Examples
   * - ``int``
     - Whole numbers
     - No fixed size
     - ``42``, ``-7``, ``2**100``
   * - ``float``
     - Real numbers, approximately
     - 64 bits (IEEE 754)
     - ``3.14``, ``-0.001``, ``1e10``
   * - ``complex``
     - Complex numbers
     - Two floats
     - ``2 + 1j``

.. code-block:: python

   count = 42
   ratio = 3.14
   impedance = 2 + 1j
   print(type(count), type(ratio), type(impedance))
   print(2**100)  # An int has no fixed size

.. code-block:: text

   <class 'int'> <class 'float'> <class 'complex'>
   1267650600228229401496703205376

- A C++ ``int`` has a fixed size and overflows. A Python ``int`` grows to
  hold any whole number: ``2**100`` has 31 digits.
- ``1e10`` means 1 times 10 to the power 10. ``j`` marks the imaginary
  part of a complex number.

Converting Between Types and Rounding
-------------------------------------

.. code-block:: python

   # Conversions
   print(int(3.7), int(-3.7))
   print(int("42"), int("101011", 2))
   print(float("3.5"), float(3), float("inf"))
   print(round(2.5), round(3.5), round(2.567, 2))

.. code-block:: text

   3 -3
   42 43
   3.5 3.0 inf
   2 4 2.57

- ``int()`` of a float: "For floating-point numbers, this truncates
  towards zero." So ``int(-3.7)`` is ``-3``, while ``-7 // 2`` rounded
  down to ``-4``.
- ``int("101011", 2)`` reads the string in base 2: 32 + 8 + 2 + 1 = 43.
- ``float("inf")`` is infinity, larger than every other float.
- ``round()``: "if two multiples are equally close, rounding is done
  toward the even choice". ``2.5`` is as close to 2 as to 3, and 2 is
  even. ``3.5`` rounds to 4 for the same reason.
- ``round(2.567, 2)`` keeps 2 decimals: ``2.57``.

Floats Are Approximate
----------------------

.. code-block:: python

   # Floats are approximate
   print(0.1 + 0.2)
   print(0.1 + 0.2 == 0.3)

.. code-block:: text

   0.30000000000000004
   False

- A ``float`` stores a number in binary, in 64 bits.
- The Python tutorial: "the decimal value 0.1 cannot be represented
  exactly as a base 2 fraction." Python stores the closest value it can.
  The same holds for ``0.2`` and ``0.3``.
- The small errors of ``0.1`` and ``0.2`` add up, and the sum is not the
  float closest to ``0.3``. So ``==`` returns ``False``.
- Robot code adds floats all the time: distances, angles, time steps.

.. warning::

   Do not compare two floats with ``==``. Compare them with a tolerance.

Compare Floats with a Tolerance
-------------------------------

.. code-block:: python

   import math

   # Compare floats with a tolerance
   goal_x_m = 0.3
   x_m = 0.1 + 0.2  # Two moves: 0.1 m, then 0.2 m
   print(x_m == goal_x_m)
   print(math.isclose(x_m, goal_x_m))
   print(abs(x_m - goal_x_m) < 0.01)  # 0.01 m, chosen for this example
   print(math.isclose(1e-12, 0.0))
   print(math.isclose(1e-12, 0.0, abs_tol=1e-9))

.. code-block:: text

   False
   True
   True
   False
   True

- The robot made two moves, 0.1 m and 0.2 m, toward a goal at 0.3 m.
  ``==`` says it has not arrived.
- ``math.isclose(a, b)`` allows a relative difference of ``1e-09``: "the
  two values are the same within about 9 decimal digits."
- ``abs(a - b) < tol`` uses a tolerance in meters that you choose for the
  task.
- Next to ``0.0``, a relative tolerance never passes: "isclose(x, 0) is
  computed as abs(x) <= rel_tol \* abs(x), which is False for any
  nonzero x". Give ``abs_tol`` when one side can be zero.

``import math`` is the first line of ``numeric_demo.py``, shown here next
to the code that uses it.

Small Integers Are Shared Objects in CPython
--------------------------------------------

.. code-block:: python

   # Small integers are shared objects in CPython
   a = 256
   b = int("256")
   print(a is b)
   c = 257
   d = int("257")
   print(c is d, c == d)

.. code-block:: text

   True
   False True

- Lecture 1 printed ``id(10)`` twice and got the same number. This is
  why.
- The CPython documentation: "The current implementation keeps an array
  of integer objects for all integers between -5 and 256. When you
  create an int in that range you actually just get back a reference to
  the existing object."
- ``256`` built two ways is one object. ``257`` built two ways is two
  objects with equal values.
- The :doc:`Memory reading </reading_material/memory/mem_lecture>` covers
  this and interned strings, in "Small Integers and Interned Strings".

.. warning::

   This is a CPython optimization, not a rule of the language. Compare
   numbers with ``==``, never with ``is``.


String Type
====================================================

All the programs in this section are in ``lecture2/strings_demo.py``,
except the errors, which are in ``lecture2/errors/``.

A String Is an Immutable Sequence of Characters
-----------------------------------------------

.. code-block:: python

   # Literals
   frame_id = "base_link"
   quote = 'He said "stop"'  # Single quotes can hold double quotes
   note = """A string
   on two lines"""
   print(frame_id, len(frame_id))
   print(quote)
   print(note)
   print(str(42) + "%", type(str(42)))

.. code-block:: text

   base_link 9
   He said "stop"
   A string
   on two lines
   42% <class 'str'>

- A ``str`` holds a sequence of characters. Once created, it cannot
  change (immutable, Lecture 1).
- Single and double quotes make the same kind of string. Triple quotes
  let a string span several lines.
- ``len()`` counts the characters: ``"base_link"`` has 9.
- ``str()`` converts a value to a string. ``+`` joins two strings.
- ROS 2 names its coordinate frames with strings such as
  ``"base_link"`` (Lecture 11).

Escape Sequences Start with a Backslash
---------------------------------------

.. code-block:: python

   # Escape sequences
   print("x:\t1.5\ny:\t2.0")
   print("Backslash: \\")
   print(r"C:\new\table")  # Raw string: no escapes

.. code-block:: text

   x:      1.5
   y:      2.0
   Backslash: \
   C:\new\table

In the output, each tab is shown as the spaces up to the next multiple of
8 columns.

.. list-table::
   :header-rows: 1
   :widths: 30 70
   :class: compact-table

   * - Escape sequence
     - Character
   * - ``\n``
     - New line
   * - ``\t``
     - Tab
   * - ``\\``
     - One backslash
   * - ``\"`` and ``\'``
     - A quote that does not end the string

A raw string, written ``r"..."``, keeps every backslash as it is. In
``r"C:\new\table"``, ``\n`` and ``\t`` are not a new line and a tab.

An Unknown Escape Sequence Gives a SyntaxWarning
------------------------------------------------

.. code-block:: python
   :caption: lecture2/errors/escape_warning.py

   path = "C:\data\new"
   print(path)

.. code-block:: text

   .../escape_warning.py:1: SyntaxWarning: invalid escape sequence '\d'
     path = "C:\data\new"
   C:\data
   ew

- ``\d`` is not an escape sequence. Python 3.12 warns, then keeps the
  backslash: ``C:\data``.
- ``\n`` is an escape sequence: the string holds a new line, so ``ew``
  is printed on its own line. Python gives no warning for it.
- What's New in Python 3.12: "In a future Python version, SyntaxError
  will eventually be raised, instead of SyntaxWarning."

.. tip::

   For Windows paths and regular expressions, write a raw string:
   ``r"C:\data\new"``.

f-Strings Put Values into Text
------------------------------

.. code-block:: python

   # f-strings
   robot = "rover"
   battery_pct = 87
   speed_mps = 0.4567
   print(f"{robot}: {battery_pct}% battery")
   print(f"Next: {battery_pct - 1}%")
   print(f"Speed: {speed_mps:.2f} m/s")
   print(f"[{robot:>8}] [{robot:<8}]")
   print(f"{battery_pct=}")

.. code-block:: text

   rover: 87% battery
   Next: 86%
   Speed: 0.46 m/s
   [   rover] [rover   ]
   battery_pct=87

- An **f-string** starts with ``f`` before the quote. Python replaces
  each ``{expression}`` with the value of the expression.
- Any expression works inside the braces: ``{battery_pct - 1}`` prints
  ``86``.
- After a colon comes a format: ``:.2f`` prints 2 decimals (``0.4567``
  becomes ``0.46``). ``:>8`` right-aligns in 8 characters, ``:<8``
  left-aligns.
- ``{battery_pct=}`` prints the expression, ``=``, and its value (added
  in Python 3.8). It is a quick way to print a variable while you debug.

.. tip::

   Use f-strings in all your code: the values sit where they are printed.

Older Formatting You Will Read in Other Code
--------------------------------------------

.. code-block:: python

   # Older formatting you will read in other code
   print("%s: %d%% battery" % (robot, battery_pct))  # noqa: UP031
   print("{}: {}% battery".format(robot, battery_pct))  # noqa: UP032

.. code-block:: text

   rover: 87% battery
   rover: 87% battery

- ``%``-formatting: ``%s`` takes a string, ``%d`` an integer, and ``%%``
  prints one ``%``.
- ``str.format()``: each ``{}`` takes the next argument.
- Both print the same line as the first f-string above.
- Ruff reports both lines. ``# noqa: UP031`` tells Ruff to skip that rule
  on that line. Without the two comments:

.. code-block:: bash

   uv run ruff check --output-format concise --ignore-noqa lecture2/strings_demo.py

.. code-block:: text

   lecture2/strings_demo.py:29:7: UP031 Use format specifiers instead of percent format
   lecture2/strings_demo.py:30:7: UP032 [*] Use f-string instead of `format` call
   Found 2 errors.
   [*] 1 fixable with `--fix`.

String Methods Return New Strings
---------------------------------

.. code-block:: python

   # Methods return new strings
   topic = "  /robot1/cmd_vel  "
   clean = topic.strip()
   print(repr(clean))
   print(clean.upper())
   print(clean.replace("robot1", "robot2"))
   print(clean.split("/"))
   print(clean.startswith("/"), clean.endswith("vel"))
   print(clean.find("cmd"), clean.count("o"))
   print(repr(topic))  # Unchanged

.. code-block:: text

   '/robot1/cmd_vel'
   /ROBOT1/CMD_VEL
   /robot2/cmd_vel
   ['', 'robot1', 'cmd_vel']
   True True
   8 2
   '  /robot1/cmd_vel  '

- A **method** is a function that belongs to an object. You call it with
  a dot: ``topic.strip()``.
- ``strip()`` removes the spaces at both ends. ``repr()`` shows the
  quotes, so you can see the spaces.
- ``split("/")`` cuts at each ``/``. The topic starts with ``/``, so the
  first piece is the empty string.
- ``find("cmd")`` gives the index where ``"cmd"`` starts (8), and
  ``count("o")`` counts the two ``o`` of ``robot1``.
- The last line prints ``topic`` unchanged: a method never changes a
  string, it returns a new one.

Building Strings with +, \*, and join()
---------------------------------------

.. code-block:: python

   # Concatenation, repetition, join
   namespace = "robot1"
   print("/" + namespace + "/scan")
   print("-" * 20)
   parts = ["", "robot1", "odom"]
   print("/".join(parts))

.. code-block:: text

   /robot1/scan
   --------------------
   /robot1/odom

- ``+`` joins two strings into a new one. ``*`` repeats a string:
  ``"-" * 20`` is 20 hyphens.
- ``sep.join(parts)`` puts ``sep`` between the items of a list of
  strings. The first item is empty, so the result starts with ``/``.
- ``join()`` is the reverse of ``split()``.
- To mix strings with numbers, use an f-string: ``"/" + 1`` raises a
  ``TypeError``.

Indexing Reads One Character
----------------------------

.. figure:: /_static/images/l2/string_indices.svg
   :alt: The name sensor has an arrow to a str object holding the five characters "l", "i", "d", "a", "r", each in its own cell. Under the cells, the row marked tag 1 gives the positive indices 0, 1, 2, 3, 4, and the row marked tag 2 gives the negative indices -5, -4, -3, -2, -1.
   :width: 70%

   1: positive index, counted from the start. 2: negative index, counted
   from the end.

.. code-block:: python

   # Indexing
   sensor = "lidar"
   print(sensor[0], sensor[4])
   print(sensor[-1], sensor[-5])

.. code-block:: text

   l r
   r l

- ``sensor[i]`` returns the character at index ``i``, as a string of
  length 1.
- Positive indices start at ``0``, so the last one is the length minus 1:
  5 - 1 = 4.
- Negative indices count from the end: ``-1`` is the last character,
  ``-5`` the first.

Reading past the end, or writing a character, is an error:

.. code-block:: python
   :caption: lecture2/errors/string_index.py

   sensor = "lidar"
   print(sensor[5])

.. code-block:: text

   Traceback (most recent call last):
     File ".../string_index.py", line 2, in <module>
       print(sensor[5])
             ~~~~~~^^^
   IndexError: string index out of range

.. code-block:: python
   :caption: lecture2/errors/string_assign.py

   sensor = "lidar"
   sensor[0] = "L"

.. code-block:: text

   Traceback (most recent call last):
     File ".../string_assign.py", line 2, in <module>
       sensor[0] = "L"
       ~~~~~~^^^
   TypeError: 'str' object does not support item assignment

- ``"lidar"`` has 5 characters, at indices 0 to 4. Index 5 does not
  exist.
- A string is immutable, so you cannot replace one of its characters.
  Build a new string instead: ``"L" + sensor[1:]`` uses a slice.

Slicing Copies a Range of Characters
------------------------------------

.. figure:: /_static/images/l2/string_slice.svg
   :alt: The five characters "l", "i", "d", "a", "r" in cells. The boundaries between the cells are numbered 0 to 5 underneath. Boundary 1 is marked tag 1 and boundary 4 is marked tag 2. The three cells between them, "i", "d", "a", are drawn in red; "l" and "r" are drawn in gray, outside the slice.
   :width: 55%

   ``sensor[1:4]``. 1: ``start``, 1. 2: ``stop``, 4. Red: the characters
   in the slice.

.. code-block:: python

   # Slicing
   print(sensor[1:4], sensor[:3], sensor[2:])
   print(sensor[:])

.. code-block:: text

   ida lid dar
   lidar

- ``s[start:stop]`` returns a new string with the characters from index
  ``start`` up to, but not including, index ``stop``.
- Picture the numbers between the characters: the slice holds what lies
  between ``start`` and ``stop``. Its length is stop - start = 4 - 1 = 3.
- Leave out ``start`` and the slice begins at 0. Leave out ``stop`` and
  it runs to the end. ``sensor[:]`` copies the whole string.

.. code-block:: python

   print(sensor[-4:-1], sensor[-3:])
   print(sensor[::2], sensor[::-1], sensor[4:1:-1])
   print(sensor[1:100])

.. code-block:: text

   ida dar
   ldr radil rad
   idar

- ``s[start:stop:step]``: the third number is the **step** between
  indices. It defaults to 1.
- ``sensor[-4:-1]`` is ``sensor[1:4]``: ``-4`` and ``1`` name the same
  position in a 5-character string.
- ``sensor[::2]`` takes indices 0, 2, 4: ``l``, ``d``, ``r``.
- A negative step walks backward. ``sensor[::-1]`` reverses the string.
  ``sensor[4:1:-1]`` takes indices 4, 3, 2 (``r``, ``a``, ``d``) and
  stops before 1.
- A slice never raises an ``IndexError``: ``sensor[1:100]`` stops at the
  end of the string.


Control Flow
====================================================

if Runs a Block When a Condition Is True
----------------------------------------

.. code-block:: python
   :caption: From lecture2/control_flow_demo.py

   battery_pct = 15  # Chosen for this example

   # if
   if battery_pct < 20:
       print("Battery low")
   print("Checked the battery")

   # if and else
   if battery_pct >= 20:
       print("Continue the mission")
   else:
       print("Return to the dock")

.. code-block:: text

   Battery low
   Checked the battery
   Return to the dock

- ``if condition:`` runs its block when the condition is truthy. Then the
  program goes on after the block.
- ``else:`` runs its block when the condition is falsy. Exactly one of
  the two blocks runs.
- ``15 >= 20`` is ``False``, so the ``else`` block runs.

elif Checks Conditions in Order
-------------------------------

.. code-block:: python
   :caption: From lecture2/control_flow_demo.py

   # if, elif, and else
   if battery_pct >= 80:
       level = "full"
   elif battery_pct >= 50:
       level = "moderate"
   elif battery_pct >= 20:
       level = "low"
   else:
       level = "critical"
   print(f"Battery level: {level}")

.. code-block:: text

   Battery level: critical

With ``battery_pct = 15``:

1. ``15 >= 80`` is ``False``.
2. ``15 >= 50`` is ``False``.
3. ``15 >= 20`` is ``False``.
4. No condition was true, so the ``else`` block runs:
   ``level = "critical"``.

Python tests the conditions from the top and runs the block of the
**first** true one. It skips the rest. So ``elif battery_pct >= 50`` only
runs when ``battery_pct`` is below 80: the order of the conditions
matters. ``elif`` and ``else`` are optional. ``elif`` is short for "else
if".

A Conditional Expression Picks One of Two Values
------------------------------------------------

.. code-block:: python
   :caption: From lecture2/control_flow_demo.py

   # Conditional expression
   mode = "charge" if battery_pct < 20 else "explore"
   print(mode)

.. code-block:: text

   charge

``a if condition else b`` is an expression: it gives ``a`` when the
condition is truthy, ``b`` otherwise. It does the same as these four
lines:

.. code-block:: python

   if battery_pct < 20:
       mode = "charge"
   else:
       mode = "explore"

Use it to choose between two values. When each branch does several
things, write an ``if`` statement.

Nested Conditions
-----------------

.. code-block:: python
   :caption: From lecture2/control_flow_demo.py

   # Nested conditions
   obstacle_ahead = True
   if battery_pct >= 20:
       if obstacle_ahead:
           print("Plan a path around the obstacle")
       else:
           print("Drive forward")
   else:
       print("Stay at the dock")

.. code-block:: text

   Stay at the dock

- An ``if`` can sit inside another ``if``. The inner one is tested only
  when the outer condition is true.
- The indentation shows which ``else`` belongs to which ``if``.
- Here ``15 >= 20`` is ``False``, so Python never looks at
  ``obstacle_ahead``.
- Past two or three levels, the code gets hard to read. ``and`` can often
  replace one level: ``if battery_pct >= 20 and obstacle_ahead:``.

match Compares a Value Against Patterns
---------------------------------------

.. code-block:: python
   :caption: lecture2/match_demo.py

   """Match a command against literal patterns.

   Run it with a command:  uv run lecture2/match_demo.py stop
   """

   import sys

   command = sys.argv[1]  # The first word after the file name

   match command:
       case "stop":
           print("Stopping")
       case "forward" | "back":
           print(f"Driving {command}")
       case _:
           print(f"Unknown command: {command}")

Three separate runs, one command each:

.. list-table::
   :header-rows: 1
   :class: compact-table

   * - Command
     - Output
   * - ``uv run lecture2/match_demo.py stop``
     - ``Stopping``
   * - ``uv run lecture2/match_demo.py back``
     - ``Driving back``
   * - ``uv run lecture2/match_demo.py jump``
     - ``Unknown command: jump``

- ``sys.argv[1]`` is the first word after the file name on the command
  line.
- ``match`` (added in Python 3.10) tries each ``case`` from the top. The
  tutorial: "Only the first pattern that matches gets executed".
- ``|`` means "or": ``"forward" | "back"`` matches either string.
- ``_`` "acts as a wildcard and never fails to match", so it catches
  every other command.
- Unlike a C++ ``switch``, a ``case`` never falls through to the next
  one, so there is no ``break``.
- The same program with ``if``, ``elif``, and ``else`` works too.
  ``match`` becomes useful when the value has a shape to take apart.

Sequence Patterns Take a Command Apart
--------------------------------------

.. code-block:: python
   :caption: lecture2/match_words_demo.py

   """Match the words of a command against sequence patterns.

   Run it with a command in quotes:  uv run lecture2/match_words_demo.py "move 2.5"
   """

   import sys

   words = sys.argv[1].split()

   match words:
       case ["move", distance] if float(distance) > 0:
           print(f"Moving {distance} m")
       case ["turn", direction]:
           print(f"Turning {direction}")
       case ["stop"]:
           print("Stopping")
       case _:
           print(f"Cannot run: {words}")

- ``split()`` with no argument cuts the string at runs of spaces:
  ``"move 2.5"`` becomes the list ``["move", "2.5"]``.
- ``["move", distance]`` matches a list of exactly two items whose first
  item is ``"move"``. It binds the second item to the name ``distance``:
  a **capture pattern**.
- ``["stop"]`` matches a list of exactly one item, ``"stop"``.
- ``if float(distance) > 0`` is a **guard**: the ``case`` matches only
  when the guard is also true. ``float()`` turns the string ``"2.5"``
  into the number ``2.5``.
- The tutorial: a match statement "can also extract components (sequence
  elements or object attributes) from the value into variables." An
  ``if`` statement cannot.

Lecture 3 covers lists. Here you only need two facts: a list is written
with square brackets, and its items are counted from index 0, like the
characters of a string.

Five separate runs, each ``uv run lecture2/match_words_demo.py``
followed by the command in quotes:

.. list-table::
   :header-rows: 1
   :widths: 20 35 45
   :class: compact-table

   * - Command in quotes
     - Output
     - Why
   * - ``"move 2.5"``
     - ``Moving 2.5 m``
     - Two words, first is ``move``, and 2.5 is above 0
   * - ``"move -1"``
     - ``Cannot run: ['move', '-1']``
     - The guard fails: -1 is not above 0
   * - ``"turn left"``
     - ``Turning left``
     - Two words, first is ``turn``
   * - ``"stop"``
     - ``Stopping``
     - One word, ``stop``
   * - ``"stop now"``
     - ``Cannot run: ['stop', 'now']``
     - Two words: ``["stop"]`` needs exactly one

When a guard fails, ``match`` goes on to the next ``case``. For
``"move -1"``, no other ``case`` fits, so ``_`` catches it.

A Bare Name in a case Captures Every Value
------------------------------------------

.. code-block:: python
   :caption: lecture2/errors/match_capture.py

   STOP = "stop"
   command = "forward"

   match command:
       case STOP:
           print("Stopping")
       case _:
           print("Another command")

.. code-block:: text

     File ".../match_capture.py", line 5
       case STOP:
            ^^^^
   SyntaxError: name capture 'STOP' makes remaining patterns unreachable

- The intent: run the first ``case`` when ``command`` equals the constant
  ``STOP``.
- A bare name in a ``case`` is a capture pattern, like ``distance``
  above. The documentation: "Capture patterns always succeed." So
  ``case STOP:`` would match ``"forward"`` too, and bind it to ``STOP``.
- Python sees that the ``case _:`` after it can never run, and refuses
  the file.
- Fix: write the literal, ``case "stop":``, or compare with
  ``if command == STOP:``.


Next Class
====================================================

:doc:`L3: Python Fundamentals, Part II </lectures/lecture3/l3_index>`
(Tuesday, February 16): loops, iterables, lists, tuples, dictionaries,
and sets.
