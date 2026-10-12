====================================================
Lecture
====================================================

.. admonition:: Disclaimer
   :class: caution

   I teach this course solely in my capacity as a lecturer at the
   University of Maryland.

   - I do **not** represent NIST in this course.
   - Opinions, findings, and recommendations expressed here are my own
     and do not necessarily reflect the views of NIST or the U.S.
     Department of Commerce.
   - Nothing stated in this course should be interpreted as an official
     NIST position, policy, or endorsement.
   - Course content, assignments, and grading are governed by UMD
     policy only.


Course Introduction
====================================================

ENPM605 teaches Python through robotics. The course focuses on:

- Python programming with robotics applications
- NumPy and Matplotlib for robot data
- ROS 2 Jazzy and Gazebo Harmonic
- Navigation with Nav2 and behavior trees
- Learned models in a robot system: perception and language commands

You do not need prior robotics knowledge. You need basic programming
experience in any language.

The grading, the late policy, the AI policy, and the full schedule are on
the :doc:`syllabus </syllabus/syllabus>` page.

Course Structure
----------------

.. grid:: 1 1 3 3
    :gutter: 2

    .. grid-item-card:: Before Class
        :class-card: sd-border-info

        - Get the lecture materials from Canvas and/or GitHub.
        - Complete the pre-work activities.

    .. grid-item-card:: During Class
        :class-card: sd-border-info

        - Questions and discussion on the previous lectures.
        - Quiz (if any).
        - Lecture. Bring your laptop with the required software
          installed. Participation counts for 5% of your grade.

    .. grid-item-card:: After Class
        :class-card: sd-border-info

        - Review the slides.
        - Re-run the code provided to you.
        - Start working on the next assignment (if any).

Quizzes
-------

- Quizzes take the first 20 minutes of class.
- If you cannot come to class (and you are not a remote/online
  student), take the quiz another day, with a 10% penalty, or with no
  penalty if you are excused (doctor's note, supervisor's note, and so
  on).
- No proctor is needed. Quizzes are closed-notes: no slides and no
  software to test code.
- **Online/remote students:** email me the day and time you plan to
  take each quiz.

Peer Reviews
------------

After each team project and the final project, you give each teammate a
score from 1 to 10 for their productive involvement. You cannot rate
yourself, and you get 1 point for emailing me your reviews. Your score
is 60% project grade and 40% peer review.

.. dropdown:: Worked example
    :class-container: sd-border-secondary

    1. Your group got 30/30 points on a project.
    2. Your teammates' average review of you is 4/10. You emailed your
       reviews, which adds 1 point: 5/10, or 50%.
    3. Project portion: 60% of 30 = 18 points.
    4. Peer review portion: 40% of 30 = 12 points, times 50% = 6 points.
    5. Your grade: 18 + 6 = **24/30 points**.

Final Project
-------------

There is no final exam. The final project is a ROS 2 package that makes
a mobile robot navigate a simulated environment in Gazebo Harmonic on
its own. It combines sensor processing, coordinate transforms, Nav2
navigation, and an application-level behavior tree written with
``py_trees``.

- **Nav2 handles** (you configure it): path planning and trajectory
  following, obstacle avoidance through costmaps, and recovery
  behaviors (spin, back up, wait).
- **You build** a behavior tree that coordinates high-level tasks:
  patrolling waypoints (through the ``/navigate_to_pose`` action),
  reacting to events, and handling task priorities.

Your behavior tree calls Nav2 action servers. It does **not** replace
Nav2's internal navigation logic.


Software and Hardware
====================================================

.. list-table::
   :header-rows: 1
   :widths: 30 70
   :class: compact-table

   * - Software
     - Role
   * - Ubuntu 24.04 LTS (Noble Numbat)
     - The operating system
   * - Visual Studio Code
     - The editor
   * - uv
     - Creates the course virtual environment and installs Python packages
   * - ROS 2 Jazzy Jalisco and Gazebo Harmonic
     - Needed from Lecture 8

Minimum hardware: an Intel i5 or equivalent with 4 or more cores, 16 GB
of RAM (32 GB recommended), and 50 GB of free storage.

Use this course to learn Linux. You should know at least these shell
commands: ``ls``, ``pwd``, ``cd``, ``mv``, ``mkdir``, ``rm``,
``source``, ``touch``, ``man``, and ``grep``.

.. note::

   Install everything natively on Ubuntu 24.04. Docker is a fallback for
   one case only: you need Gazebo Classic or ROS 2 Humble for another
   course. The two Gazebo versions cannot be installed on the same
   system.


Environment Setup
====================================================

Packages, Virtual Environments, and uv
--------------------------------------

A **package** is Python code that someone else wrote, which you install
and then ``import``: NumPy, Ruff, and pytest are packages. ``pip`` is
Python's standard tool for installing them. Ubuntu's Python 3.12 runs
parts of Ubuntu itself, so Ubuntu refuses to let ``pip`` install into
it:

.. code-block:: bash

   python3 -m pip install requests

.. code-block:: text

   error: externally-managed-environment

   × This environment is externally managed
   ╰─> To install Python packages system-wide, try apt install
       python3-xyz, where xyz is the package you are trying to
   ...

A **virtual environment** is a folder, here ``.venv``, with its own
Python packages, separate from Ubuntu's. Installing into ``.venv`` does
not change Ubuntu's Python, so Ubuntu allows it, and each project gets
the versions it needs. With ``--system-site-packages``, ``.venv`` can
also use Ubuntu's packages without changing them; ROS 2 needs this from
Lecture 8.

**uv** is a Python package and project manager, written in Rust by
Astral, the company that makes Ruff. It replaces several tools, among
them ``pip`` and ``virtualenv`` (`uv README
<https://github.com/astral-sh/uv>`_). The course repository's
``pyproject.toml`` lists what the project needs:

.. code-block:: toml

   [project]
   name = "enpm605-spring-2027"
   version = "0.1.0"
   requires-python = "==3.12.*"
   dependencies = ["numpy<2"]

   [dependency-groups]
   dev = ["pytest", "ruff"]

   [tool.uv]
   package = false

- ``requires-python``: Python 3.12 only.
- ``dependencies``: what the programs need.
- ``dev``: tools for writing the code (pytest runs tests).
- ``package = false``: the repository itself is not installed as a
  package.

``uv.lock``, kept in the repository, records the exact version of every
package (NumPy 1.26.4). ``uv sync`` installs those versions into
``.venv``, so every student gets the same ones. ``uv run`` runs a command with the Python in ``.venv``.

Setup Steps
-----------

The steps are on the :doc:`Python setup page </setup/python_setup>`. In
short:

1. Install Visual Studio Code with the **Python** (Microsoft) and
   **Ruff** (Astral Software) extensions.
2. Install uv:

   .. code-block:: bash

      curl -LsSf https://astral.sh/uv/0.13.0/install.sh | sh
      uv --version

   .. code-block:: text

      uv 0.13.0 (x86_64-unknown-linux-gnu)

3. Clone the `course repository
   <https://github.com/rubixcubic/enpm605-spring-2027-python>`_ and,
   from its root folder, create the course environment:

   .. code-block:: bash

      uv venv --python /usr/bin/python3 --system-site-packages
      uv sync

   ``uv venv`` creates ``.venv`` with Ubuntu's Python 3.12.
   ``--system-site-packages`` lets ``.venv`` see Ubuntu's packages,
   which ROS 2 needs from Lecture 8. ``uv sync`` installs the packages
   listed in ``pyproject.toml``.

   .. warning::

      Run ``uv venv`` first. If you run ``uv sync`` first, uv creates a
      ``.venv`` that cannot see Ubuntu's packages.

4. Keep NumPy below version 2. ROS 2 Jazzy's packages are built on
   Ubuntu 24.04 against NumPy 1.26.4, and ``pyproject.toml`` pins
   ``numpy<2``. With NumPy 2 in ``.venv``, Ubuntu's OpenCV fails to
   import:

   .. code-block:: text

      A module that was compiled using NumPy 1.x cannot be run in
      NumPy 2.5.4 as it may crash. To support both 1.x and 2.x
      ...
      ImportError: numpy.core.multiarray failed to import

5. Find packages in your home folder that hide Ubuntu's.
   ``pip install --user`` puts packages in
   ``~/.local/lib/python3.12/site-packages``. ``.venv`` sees that folder
   too, and a package there hides Ubuntu's copy of the same package.
   Check which copy of OpenCV Python loads:

   .. code-block:: bash

      uv run python -c "import cv2; print(cv2.__file__)"

   .. code-block:: text

      ~/.local/lib/python3.12/site-packages/cv2/__init__.py

   If it prints ``No module named 'cv2'``, skip this step and the next:
   Ubuntu's OpenCV arrives with ROS 2 in Lecture 8 (``cv_bridge``
   depends on ``python3-opencv``). On this PC, a pip copy in
   ``~/.local`` hides Ubuntu's. List the packages pip installed there:

   .. code-block:: bash

      python3 -m pip list --user

   .. code-block:: text

      Package                       Version   Editable project location
      ----------------------------- --------- ---------------------------
      opencv-python                 4.12.0.88

6. Remove them, by the name ``pip list`` prints (``opencv-python``, not
   ``cv2``):

   .. code-block:: bash

      python3 -m pip uninstall --break-system-packages opencv-python

   ``--break-system-packages`` is needed because Ubuntu blocks ``pip``
   (the error above); here it removes the copy in your home folder. Run
   the step 5 check again: ``cv2`` should now load from
   ``/usr/lib/python3/dist-packages``. If ``python3 -m pip`` prints
   ``No module named pip``, pip is not installed: skip this step.

7. Open the course repository in VS Code: ``code .`` from its root
   folder. That folder becomes your **workspace**, and VS Code stores
   its settings in ``.vscode/settings.json``.


Running Python in VS Code
====================================================

First, tell VS Code which interpreter to use: open the **Command
Palette** (``Ctrl+Shift+P``), type **Python: Select Interpreter**, and
select the interpreter in ``.venv``.

.. code-block:: python
   :caption: lecture1/running_demo.py

   def greet(name):
       print("Hello", name)


   greet("Bob")

.. grid:: 1 2 2 2
    :gutter: 2

    .. grid-item-card:: Run button
        :class-card: sd-border-secondary

        Click **Run Python File** (the triangle at the top right). The
        terminal shows:

        .. code-block:: text

           Hello Bob

    .. grid-item-card:: Integrated terminal
        :class-card: sd-border-secondary

        Open the terminal with ``Ctrl`` plus the backtick key and run:

        .. code-block:: bash

           uv run lecture1/running_demo.py

    .. grid-item-card:: Interactive mode (REPL)
        :class-card: sd-border-secondary

        Select lines of code and press ``Shift+Enter``. VS Code runs
        them in a Python REPL in the terminal.

    .. grid-item-card:: Debugger
        :class-card: sd-border-secondary

        Set a breakpoint (click in the gutter, or ``F9``), then press
        ``F5``. The first time, select **Python Debugger**, then
        **Python File**. Use the debug toolbar to step through the code,
        and the **Debug Console** to inspect variables.


Linting and Formatting with Ruff
====================================================

A **linter** reads your code and reports likely errors and style
problems, without running it. A **formatter** rewrites the layout of
your code (spaces, line breaks) without changing what it does. This
course uses **Ruff**, which does both. The Ruff project reports it as 10
to 100 times faster than Flake8 and Black (`Ruff README
<https://github.com/astral-sh/ruff>`_).

``uv sync`` installed Ruff into ``.venv``, because ``pyproject.toml``
lists it as a development dependency:

.. code-block:: bash

   uv run ruff --version

.. code-block:: text

   ruff 0.17.0

.. code-block:: bash

   uv run ruff check my_script.py        # Report problems
   uv run ruff check --fix my_script.py  # Apply the safe fixes
   uv run ruff format my_script.py       # Rewrite the layout

Ruff Reports Problems
---------------------

.. code-block:: python
   :caption: lecture1/linters_demo.py

   import os
   import sys
   import math


   def calculate_area(radius):
       pi = 3.14159  # Magic number
       area=pi*radius**2  # No spaces
       return area


   x = 10
   if x == True:  # Compare to True
       print("yes")

.. code-block:: bash

   uv run ruff check --output-format concise lecture1/linters_demo.py

.. code-block:: text

   lecture1/linters_demo.py:1:1: I001 [*] Import block is un-sorted or un-formatted
   lecture1/linters_demo.py:1:8: F401 [*] `os` imported but unused
   lecture1/linters_demo.py:2:8: F401 [*] `sys` imported but unused
   lecture1/linters_demo.py:3:8: F401 [*] `math` imported but unused
   Found 4 errors.
   [*] 4 fixable with `--fix`.

- ``I001``: the imports are not sorted. ``F401``: an import is never
  used.
- ``[*]`` marks a problem that ``ruff check --fix`` can fix safely.
- Ruff 0.17 turns on 407 rules by default
  (``ruff check --show-settings`` lists them). The comparison
  ``x == True`` (rule ``E712``) is not one of them, so it is not
  reported.

Turning On More Rules
---------------------

``--extend-select`` adds rules to the default set:

.. code-block:: bash

   uv run ruff check --output-format concise \
       --extend-select E712,PLR2004 lecture1/linters_demo.py

.. code-block:: text

   lecture1/linters_demo.py:1:1: I001 [*] Import block is un-sorted or un-formatted
   lecture1/linters_demo.py:1:8: F401 [*] `os` imported but unused
   lecture1/linters_demo.py:2:8: F401 [*] `sys` imported but unused
   lecture1/linters_demo.py:3:8: F401 [*] `math` imported but unused
   lecture1/linters_demo.py:13:4: E712 Avoid equality comparisons to `True`; use `x:` for truth checks
   Found 5 errors.
   [*] 4 fixable with `--fix` (1 unsafe fix available with `--fix --unsafe-fixes`).

``E712`` now reports ``x == True``. ``PLR2004`` (magic value
comparison) still reports nothing: it checks numbers in comparisons, and
``3.14159`` is in an assignment. To keep extra rules for the whole
project, list them under ``[tool.ruff.lint]`` in ``pyproject.toml``.

Ruff Formats the Code
---------------------

``--diff`` shows what ``ruff format`` would change, without changing the
file:

.. code-block:: bash

   uv run ruff format --diff lecture1/linters_demo.py

.. code-block:: text

   --- lecture1/linters_demo.py
   +++ lecture1/linters_demo.py
   @@ -5,7 +5,7 @@

    def calculate_area(radius):
        pi = 3.14159  # Magic number
   -    area=pi*radius**2  # No spaces
   +    area = pi * radius**2  # No spaces
        return area



   1 file would be reformatted

A line starting with ``-`` is removed; a line starting with ``+``
replaces it. The formatter added spaces around ``=`` and ``*``. It left
``radius**2`` without spaces: Ruff writes a power of two simple operands
that way.

Ruff in VS Code
---------------

With the Ruff extension installed, put this in
``.vscode/settings.json``:

.. code-block:: json

   {
       "editor.formatOnSave": true,
       "editor.defaultFormatter": "charliermarsh.ruff",
       "editor.codeActionsOnSave": {
           "source.fixAll.ruff": "explicit",
           "source.organizeImports.ruff": "explicit"
       }
   }

- **Format on save**: formats the file each time you save it.
- **Fix all**: applies the fixes marked ``[*]``, such as removing an
  unused import.
- **Organize imports**: sorts the imports (rule ``I001``).


Environment Verification
====================================================

``lecture1/verify_setup.py`` checks five things:

1. Python is version 3.12, the version ROS 2 Jazzy uses.
2. Python runs inside a virtual environment.
3. The environment can see Ubuntu's packages (ROS 2 needs them from
   Lecture 8).
4. NumPy is installed and below version 2.
5. The ``ruff`` command is available.

.. code-block:: bash

   uv run lecture1/verify_setup.py

A working setup prints:

.. code-block:: text

   [ OK ] Python 3.12 (need 3.12)
   [ OK ] virtual environment at .../enpm605-spring-2027-python/.venv
   [ OK ] system site packages visible
   [ OK ] NumPy 1.26.4 (need below 2)
   [ OK ] Ruff at .../enpm605-spring-2027-python/.venv/bin/ruff
   5 of 5 checks passed

If ``uv sync`` was run before ``uv venv``, the third check fails:

.. code-block:: text

   [ OK ] Python 3.12 (need 3.12)
   [ OK ] virtual environment at .../enpm605-spring-2027-python/.venv
   [FAIL] system site packages hidden: run 'uv venv --clear --python /usr/bin/python3 --system-site-packages', then 'uv sync'
   [ OK ] NumPy 1.26.4 (need below 2)
   [ OK ] Ruff at .../enpm605-spring-2027-python/.venv/bin/ruff
   4 of 5 checks passed

Paths are shortened to ``.../enpm605-spring-2027-python/.venv``; yours
show your full path. To fix it, run the two commands the ``[FAIL]``
line prints.


Python Execution Pipeline
====================================================

From Source Code to Running Program
-----------------------------------

- **Source code**: the ``.py`` file you write.
- **Lexer (tokenizer)**: breaks the source into tokens (names,
  operators, numbers).
- **Parser**: checks the syntax and builds a tree.
- **Abstract syntax tree (AST)**: the tree, reduced to what the code
  means.
- **Bytecode compiler**: turns the AST into low-level instructions.
- **Interpreter loop**: executes the bytecode, instruction by
  instruction.

.. figure:: /_static/images/l1/pipeline.svg
   :alt: Six stages, numbered 1 to 6: source code, lexer, parser, syntax tree (AST), bytecode compiler, interpreter loop. Arrows run from 1 to 3 along the top row, down to 4, then back along the bottom row to 6. A shaded region tagged A covers stages 1 to 5, the compilation phase; a region tagged B covers stage 6, the runtime phase.
   :width: 100%

   Stages 1 to 6, read along the arrows. A: the compilation phase, which turns
   the source file into bytecode. B: the runtime phase, which executes that
   bytecode, instruction by instruction.

.. grid:: 1 2 2 2
    :gutter: 2

    .. grid-item-card:: Compilation phase
        :class-card: sd-border-secondary

        - Lexer, parser, AST, bytecode.
        - **Syntax errors** are caught here, before any code runs.
        - The script you run is compiled on every run.
        - An imported module is compiled once; its bytecode is saved in
          ``__pycache__`` and reused until the source file changes.

    .. grid-item-card:: Runtime phase
        :class-card: sd-border-secondary

        - Happens every time the code executes.
        - The interpreter loop reads and executes the bytecode.
        - **Runtime errors** happen here (``NameError``, ``TypeError``,
          and others).
        - Uses a stack-based execution model, and manages memory and
          garbage collection.

Compiled and Interpreted Languages
----------------------------------

A **compiled** language (C, C++, Rust, Go) is translated into native
machine code, which the CPU executes directly: faster, but one binary
per platform. In an **interpreted** language such as Python (CPython),
the source is compiled to intermediate bytecode, which an interpreter executes
instruction by instruction: portable, but slower. Python compiles to
bytecode, then interprets that bytecode. The bytecode is not native
machine code.

CPython
-------

**CPython** is the reference (default) implementation of Python; the
``python3`` on Ubuntu is CPython. The interpreter is written in C, which
gives it its name. When you run ``uv run script.py``, a C program lexes, parses,
and compiles your code to bytecode, then executes the bytecode in its
evaluation loop (``_PyEval_EvalFrameDefault`` in ``ceval.c``). The
"Python Virtual Machine" is that evaluation loop, not a separate
program.

- Libraries with C extensions, such as NumPy, are built for CPython
  first.
- CPython has a **Global Interpreter Lock (GIL)**: one thread runs
  Python bytecode at a time.
- Python 3.13 added an optional build without the GIL. The course's
  Python 3.12 has the GIL.

Other implementations exist: PyPy (a just-in-time compiler for
long-running code), Jython (the JVM), IronPython (.NET), MicroPython
(microcontrollers), and GraalPy (GraalVM). Cython is not an interpreter:
it compiles Python-like code to C, as extension modules for CPython. This course uses CPython, because ``rclpy`` and NumPy
are built for it.

Inspecting the Pipeline
-----------------------

``lecture1/internals_demo.py`` shows each stage for the line
``x = 5 + 3``.

.. tab-set::

    .. tab-item:: Step 1: Tokens

        .. code-block:: python

           code = "x = 5 + 3"

           print("== Tokens ==")
           for token in tokenize.generate_tokens(io.StringIO(code).readline):
               print(token)

        .. code-block:: text

           TokenInfo(type=1 (NAME), string='x', start=(1, 0), end=(1, 1), line='x = 5 + 3')
           TokenInfo(type=55 (OP), string='=', start=(1, 2), end=(1, 3), line='x = 5 + 3')
           TokenInfo(type=2 (NUMBER), string='5', start=(1, 4), end=(1, 5), line='x = 5 + 3')
           TokenInfo(type=55 (OP), string='+', start=(1, 6), end=(1, 7), line='x = 5 + 3')
           TokenInfo(type=2 (NUMBER), string='3', start=(1, 8), end=(1, 9), line='x = 5 + 3')
           TokenInfo(type=4 (NEWLINE), string='', start=(1, 9), end=(1, 10), line='x = 5 + 3')
           TokenInfo(type=0 (ENDMARKER), string='', start=(2, 0), end=(2, 0), line='')

        Each token has a type (``NAME``, ``OP``, ``NUMBER``) and the text
        it came from.

    .. tab-item:: Step 2: Abstract syntax tree

        .. code-block:: python

           print("== Abstract syntax tree ==")
           print(ast.dump(ast.parse(code), indent=2))

        .. code-block:: text

           Module(
             body=[
               Assign(
                 targets=[
                   Name(id='x', ctx=Store())],
                 value=BinOp(
                   left=Constant(value=5),
                   op=Add(),
                   right=Constant(value=3)))],
             type_ignores=[])

        ``Assign``: the statement stores a value in ``x``. ``BinOp``: the
        value is ``5`` ``Add`` ``3``. The tree keeps the meaning and drops
        the spaces.

    .. tab-item:: Step 3: Bytecode

        .. code-block:: python

           def example():
               x = 5 + 3
               return x


           print("== Bytecode ==")
           dis.dis(example)

        .. code-block:: text

            18           0 RESUME                   0

            19           2 LOAD_CONST               1 (8)
                         4 STORE_FAST               0 (x)

            20           6 LOAD_FAST                0 (x)
                         8 RETURN_VALUE

        The first column is the line number in ``internals_demo.py``
        (``def example():`` is line 18). ``LOAD_CONST 1 (8)``: the
        compiler computed ``5 + 3`` before the program runs. This is
        called **constant folding**. This is Python 3.12 bytecode; other
        Python versions print different instructions.


Introduction to Python Variables
====================================================

The code in this section is in ``lecture1/variables_demo.py``; run it
with ``uv run lecture1/variables_demo.py``.

A Variable Is a Name for an Object
----------------------------------

A **variable** is a name that refers to a value stored in memory. The
**assignment operator** (``=``) creates it. You do not declare the type:
Python takes it from the value. Python is **dynamically typed**: a
variable's type is decided at runtime, not at compile time.

.. grid:: 1 2 2 2
    :gutter: 2

    .. grid-item-card:: Python (dynamic)
        :class-card: sd-border-secondary

        .. code-block:: python

           # No type declaration needed
           name = "Alice"
           age = 25
           gpa = 3.85
           is_student = True

    .. grid-item-card:: C++ (static)
        :class-card: sd-border-secondary

        .. code-block:: cpp

           // Must declare types
           std::string name{"Alice"};
           int age{25};
           double gpa{3.85};
           bool is_student{true};

Assignment Styles
-----------------

The values below are chosen for this example.

.. code-block:: python

   name = "rover"  # Standard assignment
   x = y = 10  # Chained: two names, one object
   robot, speed_mps, battery_pct = "rover", 0.5, 87  # Unpacking
   a, b = 1, 2
   a, b = b, a  # Swap, no temporary variable
   print(name, x, y, robot, speed_mps, battery_pct, a, b)

.. code-block:: text

   rover 10 10 rover 0.5 87 2 1

Everything Is an Object
-----------------------

In Python, **everything is an object**. A variable is a name bound to an
object in memory, not a storage location of its own. After ``x = 2``,
``x`` is a **name**, ``2`` is an **object**, and the name is bound to
the object.

.. figure:: /_static/images/l1/name_object.svg
   :alt: After x = 2: on the left, tag 1, the name x; on the right, tag 2, an int object with the value 2. An arrow goes from the name x to the object.
   :width: 60%

   1: names. 2: objects. The arrow: ``x`` refers to the object.

Every object has a type:

.. code-block:: python

   count = 10
   ratio = 3.14
   word = "hello"
   print(type(count))  # <class 'int'>
   print(type(ratio))  # <class 'float'>
   print(type(word))  # <class 'str'>
   print(type(print))  # <class 'builtin_function_or_method'>

.. code-block:: text

   <class 'int'>
   <class 'float'>
   <class 'str'>
   <class 'builtin_function_or_method'>

Even ``print`` is an object: its type is ``builtin_function_or_method``.

Object Identity
---------------

Every object has an identity, an integer that stays the same while the
object exists. In CPython, it is the object's memory address.

.. code-block:: python

   a = 10
   b = 10.5
   c = "hello"
   print(id(a))
   print(id(b))
   print(id(c))
   print(id(10))  # Same as id(a)?
   print(id("hello"))  # Same as id(c)?

.. code-block:: text

   11760072
   138314699568464
   138314689029840
   11760072
   138314689029840

.. warning::

   Most of your numbers will differ from these. Run the program twice: the
   ``id()`` of ``10.5`` and ``"hello"`` change from run to run.

.. seealso::

   The :doc:`Memory reading </reading_material/memory/mem_index>` explains
   where objects live, and why small integers such as ``10`` behave
   differently.

Dynamic Typing
--------------

The type is attached to the **value (object)**, not to the variable
name. A name can be rebound to objects of different types while the
program runs, and types are checked at **runtime**.

.. code-block:: python

   x = "hello"
   print(type(x))  # <class 'str'>
   x = 10.5
   print(type(x))  # <class 'float'>

.. figure:: /_static/images/l1/rebinding.svg
   :alt: Two panels. Tag 1, after x = "hello": the name x has an arrow to a str object "hello". Tag 2, after x = 10.5: the name x has an arrow to a float object 10.5; the str object "hello" is drawn faded, and no arrow points to it.
   :width: 90%

   1: after ``x = "hello"``. 2: after ``x = 10.5``; no name refers to
   ``"hello"`` any more (faded).

Rebinding a name to a different type can lead to bugs.
:doc:`Lecture 4 </lectures/lecture4/l4_index>` covers **type hints**. A
type checker, such as mypy, reads them and reports some of these bugs
before the program runs.

Basic Data Types
----------------

.. list-table::
   :header-rows: 1
   :widths: 14 14 22 30 10
   :class: compact-table

   * - Category
     - Type
     - Examples
     - Notes
     - Mutable
   * - Numeric
     - ``int``
     - ``42``, ``-7``, ``0``
     - Unlimited precision
     - No
   * - Numeric
     - ``float``
     - ``3.14``, ``-0.001``
     - 64-bit (IEEE 754)
     - No
   * - Numeric
     - ``bool``
     - ``True``, ``False``
     - Subclass of ``int``
     - No
   * - Text
     - ``str``
     - ``"hello"``, ``'world'``
     - Immutable sequence
     - No
   * - Null
     - ``NoneType``
     - ``None``
     - Singleton object
     - No
   * - Sequence
     - ``list``
     - ``[1, 2, 3]``
     - Ordered, changeable
     - Yes
   * - Sequence
     - ``tuple``
     - ``(1, 2, 3)``
     - Ordered, unchangeable
     - No
   * - Mapping
     - ``dict``
     - ``{"a": 1}``
     - Key-value pairs
     - Yes
   * - Set
     - ``set``
     - ``{1, 2, 3}``
     - Unordered, unique
     - Yes

:doc:`Lecture 3 </lectures/lecture3/l3_index>` covers sequences, mappings,
and sets in detail.

Immutable Objects
-----------------

An immutable object cannot be changed after creation. "Changing" one
creates a **new** object. Examples: ``int``, ``float``, ``str``,
``tuple``.

.. code-block:: python

   x = 10
   print(id(x))
   x = x + 1  # New object created
   print(id(x))  # Different from the line above

.. code-block:: text

   11760072
   11760104

Mutable Objects
---------------

A mutable object can be changed in place: same object, new content.
Examples: ``list``, ``dict``, ``set``.

.. code-block:: python

   my_list = [1, 2, 3]
   print(id(my_list))
   my_list.append(4)  # Same object
   print(id(my_list))  # Same as the line above

.. code-block:: text

   138314700165632
   138314700165632

Variables Are References
------------------------

Assigning one variable to another makes both names refer to the **same
object**.

.. code-block:: python

   a = [1, 2, 3]
   b = a  # b refers to the same list
   print(id(a))
   print(id(b))  # Same as id(a)
   print(a is b)  # True
   b.append(4)
   print(a)  # [1, 2, 3, 4]: a changed too

.. code-block:: text

   138314688997312
   138314688997312
   True
   [1, 2, 3, 4]

.. figure:: /_static/images/l1/aliasing.svg
   :alt: Three panels. Tag 1, after a = [1, 2, 3]: the name a has an arrow to a list object [1, 2, 3]. Tag 2, after b = a: the names a and b both have arrows to the same list object. Tag 3, after b.append(4): a and b still point to that one list, which now holds [1, 2, 3, 4].
   :width: 90%

   1: after ``a = [1, 2, 3]``. 2: after ``b = a``. 3: after
   ``b.append(4)``.

.. warning::

   To get an independent copy, use ``b = a.copy()`` or ``b = list(a)``.

.. seealso::

   :doc:`Memory reading </reading_material/memory/mem_index>`: containers
   hold references, and shallow versus deep copies.

The None Object
---------------

``None`` represents the **absence of a value**. It is the only instance
of the ``NoneType`` class: a **singleton**. Common uses: default
function arguments, variables with no value yet, "no result".

.. code-block:: python

   result = None  # A name with no value yet
   print(result)


   def greet(name):
       print(f"Hello, {name}")


   x = greet("Alice")  # Prints "Hello, Alice"
   print(x)  # None
   print(type(x))  # <class 'NoneType'>

.. code-block:: text

   None
   Hello, Alice
   None
   <class 'NoneType'>

Compare with ``None`` using ``is``: write ``if x is None:``, not
``if x == None:``.

Identity and Equality
---------------------

``==`` compares **values** (equality). ``is`` compares **identity** (the
same object in memory).

.. code-block:: python

   a = [1, 2, 3]
   b = [1, 2, 3]
   c = a
   print(a == b)  # True: same values
   print(a is b)  # False: different objects
   print(a is c)  # True: same object

.. code-block:: text

   True
   False
   True

Use ``==`` to compare values. Use ``is`` only for ``None`` checks, or
when you need to know that two names refer to one object.

Variable Naming Rules
---------------------

.. grid:: 1 2 2 2
    :gutter: 2

    .. grid-item-card:: Rules (must follow)
        :class-card: sd-border-secondary

        - Must start with a letter or an underscore
        - Can contain letters, digits, and underscores
        - Case-sensitive (``Name`` and ``name`` are two names)
        - Cannot be a reserved keyword

        .. code-block:: python

           # Valid names
           my_variable = 1
           _private = 2
           camelCase = 3
           var123 = 5
           # Invalid names
           # 2fast = 10      # starts with digit
           # my-var = 5      # contains hyphen
           # class = "test"  # reserved keyword

    .. grid-item-card:: Conventions (PEP 8)
        :class-card: sd-border-secondary

        - ``snake_case`` for variables and functions
        - ``UPPER_CASE`` for constants
        - ``PascalCase`` for classes
        - Be descriptive but concise

        .. code-block:: python

           # Good style (PEP 8)
           student_name = "Alice"
           MAX_RETRIES = 3
           total_count = 0
           # Avoid
           x = "Alice"  # Too vague
           studentName = "Alice"  # camelCase
           STUDENTNAME = "Alice"  # ALL CAPS

Follow `PEP 8 <https://peps.python.org/pep-0008/>`_, Python's official
style guide. Assignments are checked against it.

Reserved Keywords
-----------------

Python 3.12 has 35 keywords. They have a special meaning and
**cannot** be used as variable names; using one raises a
``SyntaxError``.

``False``, ``None``, ``True``, ``and``, ``as``, ``assert``, ``async``, ``await``, ``break``, ``class``, ``continue``, ``def``, ``del``, ``elif``, ``else``, ``except``, ``finally``, ``for``, ``from``, ``global``, ``if``, ``import``, ``in``, ``is``, ``lambda``, ``nonlocal``, ``not``, ``or``, ``pass``, ``raise``, ``return``, ``try``, ``while``, ``with``, ``yield``.

.. code-block:: python
   :caption: lecture1/keywords_demo.py

   """Python's reserved keywords (Lecture 1)."""

   import keyword

   print(keyword.iskeyword("if"))  # True
   print(keyword.iskeyword("hello"))  # False
   print(keyword.kwlist)  # Every keyword
   print(keyword.softkwlist)  # Keywords only in some places, such as match

.. code-block:: text

   True
   False
   ['False', 'None', 'True', 'and', ..., 'yield']
   ['_', 'case', 'match', 'type']

The third line is shortened: ``keyword.kwlist`` prints all 35
keywords. **Soft keywords** (``_``, ``case``, ``match``, ``type``) are keywords only in some
places: ``match`` starts a ``match`` statement, but you can still name a
variable ``match``.

The print() Function
--------------------

``print()`` writes to the screen. It accepts any number of arguments.

.. code-block:: python

   print("Hello")
   print(3)
   print(2.4)
   print(2 + 3)
   print("*" * 10)
   print("Welcome", "to", "ENPM", 605)
   print("a", "b", "c", sep="-")
   print("No newline", end="")
   print()

.. code-block:: text

   Hello
   3
   2.4
   5
   **********
   Welcome to ENPM 605
   a-b-c
   No newline

Several arguments are separated by a space, unless ``sep`` says
otherwise. ``end=""`` replaces the newline at the end; the last
``print()`` ends the line.


Next Class
====================================================

:doc:`L2: Python Fundamentals, Part I </lectures/lecture2/l2_index>`,
Tuesday, February 9.

Before next class:

- Finish the environment setup, and run
  ``uv run lecture1/verify_setup.py``.
- Read the `Python Tutorial <https://docs.python.org/3.12/tutorial/>`_,
  Sections 3, 4, and 5.

