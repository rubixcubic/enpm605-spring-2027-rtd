====================================================
Quiz
====================================================

This self-check quiz covers :doc:`Lecture 1 <l1_lecture>`.

.. note::

   **Instructions:**

   - Multiple choice questions have exactly one correct answer.
   - True/False questions ask whether the statement is correct as
     written.
   - Answer first, then open the dropdown.


----


Multiple Choice
===============

.. admonition:: Question 1
   :class: hint

   In a fresh clone, you run ``uv sync`` without running ``uv venv`` first. Which ``verify_setup.py`` check fails?

   A. Python 3.12
   B. The virtual environment check
   C. The system site packages check
   D. The Ruff check

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, The system site packages check

   *Explanation:* ``uv sync`` with no ``.venv`` creates a virtual environment without ``--system-site-packages``. Python is still 3.12, the environment exists, and Ruff is installed, but ``.venv`` cannot see Ubuntu's packages.


.. admonition:: Question 2
   :class: hint

   Why does the course environment pin ``numpy<2``?

   A. NumPy 2 does not support Python 3.12
   B. ROS 2 Jazzy's packages are built against NumPy 1.26.4, and with NumPy 2 Ubuntu's OpenCV fails to import
   C. uv cannot install NumPy 2
   D. Ruff reports an error for NumPy 2

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ROS 2 Jazzy's packages are built against NumPy 1.26.4, and with NumPy 2 Ubuntu's OpenCV fails to import

   *Explanation:* Packages built against NumPy 1.x do not work with NumPy 2. On Ubuntu 24.04, importing OpenCV with NumPy 2.5.4 fails with ``ImportError: numpy.core.multiarray failed to import``.


.. admonition:: Question 3
   :class: hint

   What does ``--system-site-packages`` do in ``uv venv --python /usr/bin/python3 --system-site-packages``?

   A. It installs every Ubuntu package into ``.venv``
   B. It lets ``.venv`` see the packages installed for Ubuntu's Python
   C. It makes ``.venv`` use the newest Python
   D. It turns off ``pip install --user``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, It lets ``.venv`` see the packages installed for Ubuntu's Python

   *Explanation:* The flag gives the environment access to the system's packages without copying them. ROS 2 needs this from Lecture 8.


.. admonition:: Question 4
   :class: hint

   What is the difference between ``ruff check`` and ``ruff format``?

   A. They are two names for the same command
   B. ``check`` reports likely errors and style problems; ``format`` rewrites the layout of the code
   C. ``check`` runs the program; ``format`` compiles it
   D. ``format`` reports problems; ``check`` fixes them

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``check`` reports likely errors and style problems; ``format`` rewrites the layout of the code

   *Explanation:* ``ruff check`` is the linter. ``ruff format`` is the formatter: it changes spaces and line breaks, not what the code does.


.. admonition:: Question 5
   :class: hint

   In Ruff's output, what does ``[*]`` next to a problem mean?

   A. The problem is an error, not a warning
   B. ``ruff check --fix`` can fix it safely
   C. The rule is turned off
   D. The problem is in a test file

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``ruff check --fix`` can fix it safely

   *Explanation:* ``[*]`` marks a fix Ruff can apply safely. ``E712`` is reported without ``[*]`` because its fix can change what the program does.


.. admonition:: Question 6
   :class: hint

   Why does Ruff's default rule set not report ``if x == True:`` in ``linters_demo.py``?

   A. The comparison is correct Python style
   B. Rule ``E712`` is not among the default rules; ``--extend-select E712`` turns it on
   C. Ruff cannot read ``if`` statements
   D. ``x`` is not defined

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, Rule ``E712`` is not among the default rules; ``--extend-select E712`` turns it on

   *Explanation:* Ruff 0.17 turns on 407 rules by default, and ``E712`` is not one of them.


.. admonition:: Question 7
   :class: hint

   In the bytecode of ``x = 5 + 3``, the instruction is ``LOAD_CONST 1 (8)``. What happened?

   A. The interpreter added 5 and 3 while the program ran
   B. The compiler computed 5 + 3 before the program ran (constant folding)
   C. The lexer replaced the tokens with 8
   D. Python stores every number as 8

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, The compiler computed 5 + 3 before the program ran (constant folding)

   *Explanation:* The bytecode compiler evaluated the constant expression ahead of time and stored the result.


.. admonition:: Question 8
   :class: hint

   After ``a = [1, 2, 3]`` and ``b = a``, you run ``b.append(4)``. What is ``a``?

   A. ``[1, 2, 3]``
   B. ``[1, 2, 3, 4]``
   C. ``[4]``
   D. An error

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``[1, 2, 3, 4]``

   *Explanation:* ``b = a`` makes a second name for the same list object, so the change is visible through both names.


.. admonition:: Question 9
   :class: hint

   Which of these objects is mutable?

   A. ``int``
   B. ``str``
   C. ``tuple``
   D. ``list``

.. dropdown:: Answer
   :class-container: sd-border-success

   **D**, ``list``

   *Explanation:* A list can be changed in place. ``int``, ``str``, and ``tuple`` are immutable: "changing" one creates a new object.


True or False
=============

.. admonition:: Question 10
   :class: hint

   ``a == b`` and ``a is b`` always give the same result.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* ``==`` compares values; ``is`` compares identity. Two separate lists ``[1, 2, 3]`` are equal but not the same object.


.. admonition:: Question 11
   :class: hint

   ``match`` cannot be used as a variable name in Python 3.12.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* ``match`` is a soft keyword: it is a keyword only at the start of a ``match`` statement. ``match = 5`` works; ``class = 5`` is a ``SyntaxError``.


.. admonition:: Question 12
   :class: hint

   The ``id()`` of an object can change while the object exists.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* An object keeps the same identity for its whole lifetime. Some ``id()`` values differ from one run of a program to the next, but never during the life of one object.

