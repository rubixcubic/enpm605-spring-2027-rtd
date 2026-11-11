====================================================
Quiz
====================================================

This self-check quiz covers :doc:`Lecture 4 <l4_lecture>`.

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

      def double(x):
          x = x * 2

      print(double(3))

   A. ``6``
   B. ``None``
   C. ``3``
   D. Nothing: it raises an error

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``None``

   *Explanation:* ``double()`` computes 6 and binds its local ``x`` to it, but has no ``return``. A function without ``return`` returns ``None``.


.. admonition:: Question 2
   :class: hint

   With ``def drive_time(distance_m, speed_mps=0.2): return distance_m / speed_mps``, what does ``drive_time(speed_mps=0.5, distance_m=2.0)`` return?

   A. ``4.0``
   B. ``0.25``
   C. ``10.0``
   D. ``TypeError``: keyword arguments must follow the order of the parameters

.. dropdown:: Answer
   :class-container: sd-border-success

   **A**, ``4.0``

   *Explanation:* Keyword arguments are matched by name, in any order: 2.0 / 0.5 = 4.0. The default, 0.2, is not used because the call passes ``speed_mps``.


.. admonition:: Question 3
   :class: hint

   What does this program print?

   .. code-block:: python

      def log(r, history=[]):
          history.append(r)
          return history

      log(1)
      print(log(2))

   A. ``[2]``
   B. ``[1, 2]``
   C. ``[]``
   D. ``TypeError``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``[1, 2]``

   *Explanation:* The tutorial: "The default value is evaluated only once." Both calls append to the one list made when ``def`` ran. Use ``history=None`` and make the list in the body.


.. admonition:: Question 4
   :class: hint

   With ``def f(*args, **kwargs): print(args, kwargs)``, what does ``f(1, 2, x=3)`` print?

   A. ``[1, 2] {'x': 3}``
   B. ``(1, 2, 3) {}``
   C. ``(1, 2) {'x': 3}``
   D. ``(1, 2) ('x', 3)``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``(1, 2) {'x': 3}``

   *Explanation:* ``*args`` collects the extra positional arguments in a tuple, and ``**kwargs`` collects the extra keyword arguments in a dictionary.


.. admonition:: Question 5
   :class: hint

   What happens when this program runs?

   .. code-block:: python

      count = 0

      def tick():
          count += 1

      tick()

   A. ``count`` becomes 1
   B. ``count`` stays 0, and nothing is reported
   C. ``NameError``
   D. ``UnboundLocalError``

.. dropdown:: Answer
   :class-container: sd-border-success

   **D**, ``UnboundLocalError``

   *Explanation:* ``count += 1`` assigns, so ``count`` is local in all of ``tick()``, and it has no value when ``+=`` reads it. ``global count`` would make it the module's name.


.. admonition:: Question 6
   :class: hint

   With ``def clip(scan): scan = [0.0]``, what does ``ranges = [2.5]; clip(ranges); print(ranges)`` print?

   A. ``[2.5]``
   B. ``[0.0]``
   C. ``[2.5, 0.0]``
   D. ``None``

.. dropdown:: Answer
   :class-container: sd-border-success

   **A**, ``[2.5]``

   *Explanation:* ``scan = [0.0]`` binds the call's name to a new list. The caller's list is not changed, so ``ranges`` is still ``[2.5]``.


.. admonition:: Question 7
   :class: hint

   With ``def add(xs): xs.append(1)``, what does ``a = []; add(a); add(a); print(a)`` print?

   A. ``[]``
   B. ``[1]``
   C. ``[1, 1]``
   D. ``None``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``[1, 1]``

   *Explanation:* ``xs`` refers to the caller's list, and ``append()`` changes that list in place. Two calls append twice.


.. admonition:: Question 8
   :class: hint

   With ``def h(x: int) -> int: return x * 2``, what does ``print(h("ab"))`` do?

   A. Prints ``abab``
   B. Raises ``TypeError``: the hint says ``int``
   C. Prints ``None``
   D. Raises ``SyntaxError``

.. dropdown:: Answer
   :class-container: sd-border-success

   **A**, Prints ``abab``

   *Explanation:* "The Python runtime does not enforce function and variable type annotations." ``"ab" * 2`` is ``"abab"``. A type checker such as mypy would report the call.


.. admonition:: Question 9
   :class: hint

   What does ``sorted([(1, "b"), (2, "a")], key=lambda p: p[1])`` return?

   A. ``[(1, 'b'), (2, 'a')]``
   B. ``[(2, 'a'), (1, 'b')]``
   C. ``['a', 'b']``
   D. ``[1, 2]``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``[(2, 'a'), (1, 'b')]``

   *Explanation:* ``sorted()`` compares ``key(item)``, here the second item of each tuple: ``"a"`` comes before ``"b"``. It returns the items themselves, not the keys.


.. admonition:: Question 10
   :class: hint

   With ``from functools import partial`` and ``p = partial(pow, 2)``, what does ``p(5)`` return?

   A. ``25``
   B. ``10``
   C. ``32``
   D. ``7``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``32``

   *Explanation:* The stored positional argument fills the first parameter, so ``p(5)`` calls ``pow(2, 5)``: 2 to the power 5 is 32.


True or False
=============

.. admonition:: Question 11
   :class: hint

   A function without a ``return`` statement returns ``None``.

.. dropdown:: Answer
   :class-container: sd-border-success

   **True**

   *Explanation:* The tutorial: "even functions without a return statement do return a value, albeit a rather boring one. This value is called None".


.. admonition:: Question 12
   :class: hint

   Python checks type hints when a function is called, and raises ``TypeError`` when an argument has the wrong type.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* Python stores the hints in ``__annotations__`` and does nothing else with them. In the lecture, ``repeat_label(2.5, 2)`` ran and printed ``5.0``; mypy reported it.


.. admonition:: Question 13
   :class: hint

   ``def factorial(n): return n * factorial(n - 1)`` has no base case, so ``factorial(5)`` raises ``RecursionError``.

.. dropdown:: Answer
   :class-container: sd-border-success

   **True**

   *Explanation:* Every call makes another call and adds a frame. CPython stops at its recursion limit, 1000 by default, and raises ``RecursionError: maximum recursion depth exceeded``.


.. admonition:: Question 14
   :class: hint

   After ``fs = []`` and ``for k in range(3): fs.append(lambda: k)``, the calls ``[f() for f in fs]`` give ``[0, 1, 2]``.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* Each lambda reads ``k`` when it is called, after the loop, when ``k`` is 2: the result is ``[2, 2, 2]``. ``lambda k=k: k`` keeps each value: a default is evaluated when the lambda is made.
