====================================================
Quiz
====================================================

This self-check quiz covers the :doc:`Memory reading <mem_lecture>`.

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

   Where does CPython keep the object that ``speed_mps = 0.5`` creates?

   A. In the module namespace
   B. On the heap
   C. In the call frame of the module
   D. Inside the name ``speed_mps``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, On the heap

   *Explanation:* Every object lives on the heap. The module namespace holds the **name** ``speed_mps``, which refers to the object.


.. admonition:: Question 2
   :class: hint

   Inside a function, ``time_s = distance_m / speed_mps``. After the call returns, what happens to the name ``time_s``?

   A. It moves to the module namespace
   B. It is gone: the call's local namespace is discarded
   C. It keeps its value for the next call
   D. It becomes ``None``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, It is gone: the call's local namespace is discarded

   *Explanation:* Each call gets its own local namespace, discarded when the call returns. In the reading, ``"time_s" in globals()`` printed ``False``.


.. admonition:: Question 3
   :class: hint

   ``sys.getrefcount(waypoints)`` prints 2 right after ``waypoints = [1, 2]``. Why 2?

   A. The list is stored twice
   B. The name, plus the temporary reference from passing it to ``getrefcount``
   C. Lists always start at 2
   D. The two elements each count once

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, The name, plus the temporary reference from passing it to ``getrefcount``

   *Explanation:* The documentation says the count "includes the (temporary) reference as an argument to getrefcount()".


.. admonition:: Question 4
   :class: hint

   ``sys.getsizeof([1, 2, 3])`` and ``sys.getsizeof(["a" * 1000, "b" * 1000, "c" * 1000])`` both print 88. Why?

   A. The strings are compressed
   B. ``getsizeof`` counts the list's own references, not the objects they refer to
   C. Python stores only the first character
   D. Both lists are the same object

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``getsizeof`` counts the list's own references, not the objects they refer to

   *Explanation:* "Only the memory consumption directly attributed to the object is accounted for, not the memory consumption of objects it refers to."


.. admonition:: Question 5
   :class: hint

   After ``rover = Robot()``, ``backup = rover``, and ``del rover``, is the object freed?

   A. Yes, ``del`` frees the object
   B. No: ``backup`` still refers to it, so its reference count is 1
   C. Only after ``gc.collect()``
   D. Only when the program ends

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, No: ``backup`` still refers to it, so its reference count is 1

   *Explanation:* ``del`` removes a name. The object is freed when its reference count reaches 0, which happens at ``del backup``.


.. admonition:: Question 6
   :class: hint

   Two objects refer to each other, and no name refers to either. What frees them?

   A. Reference counting, at once
   B. The cycle collector (``gc``)
   C. Nothing: they leak until the program ends
   D. ``del``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, The cycle collector (``gc``)

   *Explanation:* Each still has a reference from the other, so neither count reaches 0. The cycle collector finds such groups and frees them.


.. admonition:: Question 7
   :class: hint

   ``shallow = copy.copy(grid)`` with ``grid = [[0, 0], [0, 0]]``, then ``grid[0][0] = 1``. What is ``shallow``?

   A. ``[[0, 0], [0, 0]]``
   B. ``[[1, 0], [0, 0]]``
   C. ``[[1, 0], [1, 0]]``
   D. An error

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``[[1, 0], [0, 0]]``

   *Explanation:* A shallow copy is a new outer list whose slots refer to the same inner lists, so the change shows.


.. admonition:: Question 8
   :class: hint

   Why does a NumPy array of 1,000,000 floats take about a quarter of the memory of a list of the same floats?

   A. NumPy compresses numbers
   B. The array stores the numbers side by side, 8 bytes each; the list stores 8-byte references to separate 24-byte float objects
   C. NumPy uses 16-bit numbers
   D. The list stores each number twice

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, The array stores the numbers side by side, 8 bytes each; the list stores 8-byte references to separate 24-byte float objects

   *Explanation:* Measured in the reading: 8,000,000 bytes for the array against 32,448,728 for the list and its floats.


True or False
=============

.. admonition:: Question 9
   :class: hint

   In CPython, ``id()`` of an object can change while the object exists.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* An object's identity is fixed for its lifetime; in CPython it is the object's memory address.


.. admonition:: Question 10
   :class: hint

   ``x is y`` is a safe way to check whether two integers are equal.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* ``is`` checks identity. Whether two equal integers are one object depends on CPython details (small-integer cache, constants per compiled unit) and differed between a script and the REPL in the reading. Use ``==``.


.. admonition:: Question 11
   :class: hint

   In CPython 3.12, ``sys.getrefcount(None)`` reports how many references to ``None`` exist.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* ``None`` is immortal (PEP 683): its count stays at a fixed large value, 4294967295, which does not match the number of references.


.. admonition:: Question 12
   :class: hint

   Each call of a function gets its own frame, which is discarded when the call returns.

.. dropdown:: Answer
   :class-container: sd-border-success

   **True**

   *Explanation:* The frames of the calls in progress form a stack; each points back to its caller's frame (``f_back``).

