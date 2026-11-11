====================================================
Quiz
====================================================

This self-check quiz covers :doc:`Lecture 5 <l5_lecture>`. Every code
snippet starts with ``import numpy as np``.

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

   What does this program print?

   .. code-block:: python

      a = np.array([0.5, 1.0])
      print(a * 2)

   A. ``[0.5, 1.0, 0.5, 1.0]``
   B. ``[1. 2.]``
   C. ``[0.5 1.  0.5 1. ]``
   D. Nothing: it raises ``TypeError``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``[1. 2.]``

   *Explanation:* For an array, ``* 2`` multiplies each number by 2. For a list, ``* 2`` would repeat the list (answer A).


.. admonition:: Question 2
   :class: hint

   What is the shape of ``np.zeros((4, 3))[:, 1]``?

   A. ``(3,)``
   B. ``(4,)``
   C. ``(4, 1)``
   D. ``(1, 3)``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``(4,)``

   *Explanation:* ``:`` keeps every one of the 4 rows, and the single index ``1`` picks one column, so the result is a 1D array of 4 items.


.. admonition:: Question 3
   :class: hint

   What does this program print?

   .. code-block:: python

      r = np.array([2.5, 0.9, 3.1])
      part = r[0:2]
      part[1] = 0.0
      print(r)

   A. ``[2.5 0.9 3.1]``
   B. ``[2.5 0.  3.1]``
   C. ``[0.  0.9 3.1]``
   D. ``[2.5 0. ]``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``[2.5 0.  3.1]``

   *Explanation:* A slice of an array is a view: ``part`` uses the numbers of ``r``, so ``part[1] = 0.0`` changes ``r[1]``. A slice of a list would be a copy (answer A).


.. admonition:: Question 4
   :class: hint

   What does this program print?

   .. code-block:: python

      r = np.array([0.05, 2.0, 15.0, 1.0])
      print(r[(r >= 0.12) & (r <= 12.0)])

   A. ``[2. 1.]``
   B. ``[0.05 15.  ]``
   C. ``[False  True False  True]``
   D. ``[2. 15. 1.]``

.. dropdown:: Answer
   :class-container: sd-border-success

   **A**, ``[2. 1.]``

   *Explanation:* The mask is ``True`` where both tests pass: 2.0 and 1.0. Indexing with the mask keeps those items; answer C is the mask itself.


.. admonition:: Question 5
   :class: hint

   With ``r = np.array([0.05, 2.0, 15.0, 1.0])``, which expression raises an error?

   A. ``(r > 1) & (r < 3)``
   B. ``r > 1 and r < 3``
   C. ``~(r > 1)``
   D. ``(r > 1) | (r < 3)``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``r > 1 and r < 3``

   *Explanation:* ``and`` needs one ``True`` or ``False``, and an array of 4 Booleans is not one answer: ``ValueError: The truth value of an array with more than one element is ambiguous``. ``&``, ``|``, and ``~`` work item by item.


.. admonition:: Question 6
   :class: hint

   What is the shape of ``np.zeros((5, 1)) + np.zeros(3)``?

   A. ``(5, 1)``
   B. ``(3,)``
   C. ``(5, 3)``
   D. It raises ``ValueError``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``(5, 3)``

   *Explanation:* From the right: 1 and 3 are compatible (one is 1), giving 3; then 5 and a missing size (1), giving 5. The column is used for every column and the row for every row.


.. admonition:: Question 7
   :class: hint

   What does ``np.array([np.nan, 1.0, 2.0]).min()`` return?

   A. ``1.0``
   B. ``nan``
   C. ``0``
   D. It raises ``ValueError``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``nan``

   *Explanation:* A NaN in the array makes ``.min()`` return ``nan``. Drop invalid readings with a mask first, as the lecture's valid mask does.


.. admonition:: Question 8
   :class: hint

   ``R`` is a 2 by 2 rotation matrix and ``p`` a vector of shape ``(2,)``. Which expression gives ``p`` turned by ``R``?

   A. ``R * p``
   B. ``p * R``
   C. ``R @ p``
   D. ``R.T * p``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``R @ p``

   *Explanation:* ``@`` is the matrix product. ``*`` broadcasts ``p`` to each row of ``R`` and multiplies item by item: the result has shape ``(2, 2)``, not a vector.


.. admonition:: Question 9
   :class: hint

   A robot is at (1.0, 0.0) m in the world frame, heading 90° (it faces +y). A point is 2.0 m straight ahead of it. Where is the point in the world frame?

   A. (3.0, 0.0)
   B. (1.0, 2.0)
   C. (−1.0, 0.0)
   D. (2.0, 1.0)

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, (1.0, 2.0)

   *Explanation:* In the robot's frame the point is (2.0, 0.0). Turned by 90° it is (0.0, 2.0); plus the robot's position (1.0, 0.0), it is (1.0, 2.0).


.. admonition:: Question 10
   :class: hint

   What does ``np.array([250], dtype=np.uint8) + 10`` give?

   A. ``[260]``
   B. ``[255]``
   C. ``[4]``
   D. It raises ``OverflowError``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``[4]``

   *Explanation:* ``uint8`` holds 0 to 255. 250 + 10 = 260, which is 256 too many, so the item wraps around to 260 − 256 = 4, with no error.


True or False
=============

.. admonition:: Question 11
   :class: hint

   ``np.arange(0.0, 1.0, 0.25)`` includes 1.0.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* As with ``range()``, ``stop`` is left out: the result is ``[0.   0.25 0.5  0.75]``. ``np.linspace(0.0, 1.0, 5)`` includes 1.0.


.. admonition:: Question 12
   :class: hint

   Indexing an array with a Boolean mask returns a view, so changing the result changes the original array.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* A mask is advanced indexing, and NumPy's guide says "Advanced indexing, on the other hand, always creates copies." Only basic indexing (an integer or a slice) gives a view.


.. admonition:: Question 13
   :class: hint

   The test ``(r >= range_min) & (r <= range_max)`` is ``False`` for NaN, +inf, and −inf readings.

.. dropdown:: Answer
   :class-container: sd-border-success

   **True**

   *Explanation:* Every ``<``, ``<=``, ``>``, ``>=``, and ``==`` with NaN is ``False`` (only ``!=`` is ``True``); +inf is above any ``range_max``, and −inf below any ``range_min``. REP 117 says this check "will successfully reject all errors cases."


.. admonition:: Question 14
   :class: hint

   To get Matplotlib in the course environment, run ``pip install --user matplotlib``.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* Matplotlib is a dependency of the course project: ``uv add matplotlib`` put it in ``pyproject.toml`` and ``uv.lock``, and ``uv sync`` installs it into ``.venv``. ``pip install --user`` puts a copy in ``~/.local``, outside the project, where it can hide Ubuntu's packages.
