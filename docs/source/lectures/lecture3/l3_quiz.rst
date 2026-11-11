====================================================
Quiz
====================================================

This self-check quiz covers :doc:`Lecture 3 <l3_lecture>`.

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

   What does ``print(list(range(10, 0, -3)))`` print?

   A. ``[10, 7, 4, 1]``
   B. ``[10, 7, 4, 1, -2]``
   C. ``[10, 7, 4]``
   D. ``[]``

.. dropdown:: Answer
   :class-container: sd-border-success

   **A**, ``[10, 7, 4, 1]``

   *Explanation:* A negative step counts down from ``start``, 3 at a time: 10, 7, 4, 1. The next value, -2, is past the stop, 0, and the stop itself is never included.


.. admonition:: Question 2
   :class: hint

   A loop ``for r in [2.5, 0.9, 3.1]:`` has an ``if r < 0.5:`` that runs ``break``, and an ``else:`` lined up with ``for``. When does the ``else`` block run?

   A. Once for every reading of 0.5 or more
   B. Only when no reading is below 0.5, so ``break`` never ran
   C. Only when ``break`` ran
   D. Never: ``else`` needs an ``if``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, Only when no reading is below 0.5, so ``break`` never ran

   *Explanation:* The tutorial: "If the loop finishes without executing the break, the else clause executes." Here no reading is below 0.5, so the loop ends normally and ``else`` runs once.


.. admonition:: Question 3
   :class: hint

   ``it = iter([2.5, 0.9])``. What happens on the third ``next(it)``?

   A. It returns ``None``
   B. It returns ``2.5`` again
   C. It raises ``StopIteration``
   D. It raises ``IndexError``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, It raises ``StopIteration``

   *Explanation:* The iterator has given both items. The glossary: "When no more data are available a StopIteration exception is raised instead." A ``for`` loop catches it and ends; a direct call to ``next()`` does not.


.. admonition:: Question 4
   :class: hint

   What does ``print(list(zip([-30, 0, 30], [2.5, 0.9])))`` print?

   A. ``[(-30, 2.5), (0, 0.9), (30, None)]``
   B. ``[(-30, 2.5), (0, 0.9)]``
   C. A ``ValueError``
   D. ``[(-30, 0, 30), (2.5, 0.9)]``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``[(-30, 2.5), (0, 0.9)]``

   *Explanation:* By default, ``zip()`` stops when the shortest input runs out, with no error. ``zip(..., strict=True)`` raises a ``ValueError`` when the lengths differ.


.. admonition:: Question 5
   :class: hint

   ``ranges_m = [2.5, 0.9, 3.1]``, then ``ranges_m = ranges_m.sort()``. What is ``ranges_m`` now?

   A. ``[0.9, 2.5, 3.1]``
   B. ``[2.5, 0.9, 3.1]``
   C. ``None``
   D. ``[3.1, 2.5, 0.9]``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``None``

   *Explanation:* ``sort()`` sorts the list in place and returns ``None``, and the assignment binds ``ranges_m`` to that ``None``. Write ``ranges_m.sort()`` alone, or ``ranges_m = sorted(ranges_m)``.


.. admonition:: Question 6
   :class: hint

   ``bad = [[0] * 3] * 3``, then ``bad[0][0] = 1``. What is ``bad``?

   A. ``[[1, 0, 0], [0, 0, 0], [0, 0, 0]]``
   B. ``[[1, 0, 0], [1, 0, 0], [1, 0, 0]]``
   C. ``[[1, 1, 1], [0, 0, 0], [0, 0, 0]]``
   D. A ``TypeError``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``[[1, 0, 0], [1, 0, 0], [1, 0, 0]]``

   *Explanation:* ``* 3`` repeats the reference to one inner list, so the three rows are one list. Build the grid with ``[[0] * 3 for _ in range(3)]`` to get three inner lists.


.. admonition:: Question 7
   :class: hint

   ``path = [[0.0, 0.0], [1.0, 0.0]]`` and ``shallow = path.copy()``. Then ``path[0][0] = 9.9`` and ``path.append([2.0, 0.0])``. What is ``shallow``?

   A. ``[[0.0, 0.0], [1.0, 0.0]]``
   B. ``[[9.9, 0.0], [1.0, 0.0]]``
   C. ``[[9.9, 0.0], [1.0, 0.0], [2.0, 0.0]]``
   D. ``[[0.0, 0.0], [1.0, 0.0], [2.0, 0.0]]``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``[[9.9, 0.0], [1.0, 0.0]]``

   *Explanation:* A shallow copy is a new outer list whose items refer to the same inner lists. The change inside ``path[0]`` shows through it; the append to ``path``'s own outer list does not.


.. admonition:: Question 8
   :class: hint

   Which of these can be a dictionary key?

   A. ``[0, 1]``
   B. ``(0, 1)``
   C. ``{0, 1}``
   D. ``("lidar", [2.5, 0.9])``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``(0, 1)``

   *Explanation:* A key must be hashable. Lists and sets can change, so they are not hashable. A tuple is hashable only if its items are, and D holds a list.


.. admonition:: Question 9
   :class: hint

   ``robot = {"name": "waffle_01"}``. What does ``print(robot.get("battery_pct", 100), "waffle_01" in robot)`` print?

   A. ``100 True``
   B. ``None False``
   C. ``100 False``
   D. A ``KeyError``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``100 False``

   *Explanation:* ``get()`` returns the default for a missing key instead of raising a ``KeyError``. ``in`` tests the keys, and ``"waffle_01"`` is a value.


.. admonition:: Question 10
   :class: hint

   ``a = {"wall", "person", "cone", "pallet"}`` and ``b = {"person", "cone", "door"}``. What is ``sorted(a ^ b)``?

   A. ``['cone', 'person']``
   B. ``['pallet', 'wall']``
   C. ``['door', 'pallet', 'wall']``
   D. ``['cone', 'door', 'pallet', 'person', 'wall']``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``['door', 'pallet', 'wall']``

   *Explanation:* ``^`` is the symmetric difference: the items in exactly one of the two sets. ``"person"`` and ``"cone"`` are in both, so they are left out.


True or False
=============

.. admonition:: Question 11
   :class: hint

   Removing items from a list inside a ``for`` loop over that same list always removes every item the condition matches.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* The loop's iterator moves to the next index after each item, while ``remove()`` moves the later items one place left, so the item after a removed one is skipped. In the lecture, a scan with three ``0.0`` readings kept one. Build a new list with a comprehension instead.


.. admonition:: Question 12
   :class: hint

   ``(42)`` is a tuple with one item.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* The comma makes the tuple. ``(42)`` is the number 42 in parentheses; ``(42,)`` is a tuple with one item.


.. admonition:: Question 13
   :class: hint

   A set of strings prints its items in the same order every time the program runs.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* A set places each item by its hash, and the hashes of strings are seeded with a random value at each run unless ``PYTHONHASHSEED`` is set. In the lecture, seeds 1 and 2 printed the same four topics in two orders. Use ``sorted()`` for a fixed order.


.. admonition:: Question 14
   :class: hint

   Deleting a key from a dictionary inside a ``for`` loop over that dictionary, when the loop continues after the deletion, raises a ``RuntimeError``.

.. dropdown:: Answer
   :class-container: sd-border-success

   **True**

   *Explanation:* Python raises ``RuntimeError: dictionary changed size during iteration`` at the next step of the loop. A ``break`` right after the deletion ends the loop first, so nothing is raised. Loop over ``list(d)``, a copy of the keys, or build a new dictionary with a comprehension.
