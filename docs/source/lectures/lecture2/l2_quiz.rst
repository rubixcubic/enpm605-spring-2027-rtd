====================================================
Quiz
====================================================

This self-check quiz covers :doc:`Lecture 2 <l2_lecture>`.

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

   You run ``uv run lecture2/packages_demo.py`` from the root of the course repository. Which folder is first on ``sys.path``?

   A. The root of the course repository
   B. ``lecture2/``
   C. ``.venv/lib/python3.12/site-packages``
   D. ``/usr/lib/python3.12``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``lecture2/``

   *Explanation:* For ``python script.py``, Python puts the folder of the script first on ``sys.path``, not the folder you ran the command from. That is why ``packages_demo.py`` finds ``sensors/`` next to it.


.. admonition:: Question 2
   :class: hint

   ``lidar.py`` and ``sonar.py`` both define ``MAX_RANGE_M``, as 12.0 and 4.0. A program runs ``from sensors.lidar import *``, then ``from sensors.sonar import *``, then ``print(MAX_RANGE_M)``. What happens?

   A. It prints ``12.0``
   B. It prints ``4.0``
   C. It raises a ``NameError``
   D. Ruff's default rules report the clash

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, It prints ``4.0``

   *Explanation:* The second wildcard import rebinds ``MAX_RANGE_M`` to the sonar's value, with no warning. Ruff 0.17's default rules report nothing; rules ``F403`` and ``F405`` do, when you add them.


.. admonition:: Question 3
   :class: hint

   ``lidar.py`` ends with ``if __name__ == "__main__":`` and a ``print()`` under it. When does that ``print()`` run?

   A. Every time the module is imported
   B. Only when ``lidar.py`` is the file you run
   C. Only the first time the module is imported
   D. Never

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, Only when ``lidar.py`` is the file you run

   *Explanation:* The file you run gets ``__name__ == "__main__"``. An imported module gets its full name, ``"sensors.lidar"``, so the guarded line is skipped.


.. admonition:: Question 4
   :class: hint

   What does ``print(-7 // 2, -7 % 3)`` print?

   A. ``-3 -1``
   B. ``-4 2``
   C. ``-3 2``
   D. ``-4 -1``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``-4 2``

   *Explanation:* ``//`` rounds toward negative infinity: ``-3.5`` becomes ``-4``. For ``-7 % 3``: ``-7 // 3`` is ``-3``, 3 times ``-3`` is ``-9``, and ``-7`` is 2 more than ``-9``.


.. admonition:: Question 5
   :class: hint

   What does ``print(-2**2)`` print?

   A. ``4``
   B. ``-4``
   C. ``0.25``
   D. A ``SyntaxError``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``-4``

   *Explanation:* ``**`` runs before a minus sign on its left, so ``-2**2`` is ``-(2**2)``. ``ruff format`` writes the parentheses for you.


.. admonition:: Question 6
   :class: hint

   ``requested_speed = 0.0``. What is ``requested_speed or 0.5``?

   A. ``0.0``
   B. ``0.5``
   C. ``True``
   D. ``False``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``0.5``

   *Explanation:* ``x or y`` gives ``x`` if ``x`` is true, else ``y``. ``0.0`` is falsy, so the result is ``0.5``: the robot that was told to stop keeps driving. Test for ``None`` with ``is None`` instead.


.. admonition:: Question 7
   :class: hint

   Which call returns ``False``?

   A. ``bool("False")``
   B. ``bool([0])``
   C. ``bool(" ")``
   D. ``bool(0.0)``

.. dropdown:: Answer
   :class-container: sd-border-success

   **D**, ``bool(0.0)``

   *Explanation:* Zero of any numeric type is falsy. ``"False"`` and ``" "`` are non-empty strings, and ``[0]`` is a list with one item, so all three are truthy.


.. admonition:: Question 8
   :class: hint

   ``sensor = "lidar"``. What does ``print(sensor[1:4], sensor[::-1])`` print?

   A. ``lid radil``
   B. ``ida radil``
   C. ``idar radil``
   D. ``ida lidar``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``ida radil``

   *Explanation:* A slice goes from ``start`` up to, but not including, ``stop``: indices 1, 2, 3. A step of ``-1`` walks backward over the whole string.


.. admonition:: Question 9
   :class: hint

   What does ``round(2.5)`` return in Python 3.12?

   A. ``2``
   B. ``3``
   C. ``2.5``
   D. ``3.0``

.. dropdown:: Answer
   :class-container: sd-border-success

   **A**, ``2``

   *Explanation:* When two multiples are equally close, ``round()`` picks the even one. 2.5 is as close to 2 as to 3, and 2 is even. ``round(3.5)`` is 4.


True or False
=============

.. admonition:: Question 10
   :class: hint

   ``0.1 + 0.2 == 0.3`` is ``True`` in Python.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* 0.1 cannot be stored exactly as a binary fraction. ``0.1 + 0.2`` prints ``0.30000000000000004``. Compare floats with ``math.isclose()`` or with a tolerance you choose, such as ``abs(a - b) < 0.01``.


.. admonition:: Question 11
   :class: hint

   In a ``match`` statement, ``case STOP:`` (with ``STOP = "stop"`` defined earlier) matches only when the value equals ``"stop"``.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* A bare name in a ``case`` is a capture pattern, which always succeeds. With another ``case`` after it, Python refuses the file: ``SyntaxError: name capture 'STOP' makes remaining patterns unreachable``. Write ``case "stop":`` instead.


.. admonition:: Question 12
   :class: hint

   A line indented with a tab, right after a line of the same block indented with 4 spaces, makes Python raise an error before any line of the program runs.

.. dropdown:: Answer
   :class-container: sd-border-success

   **True**

   *Explanation:* Python raises ``TabError: inconsistent use of tabs and spaces in indentation`` while it compiles the file, so no line runs. PEP 8 recommends 4 spaces per indentation level.
