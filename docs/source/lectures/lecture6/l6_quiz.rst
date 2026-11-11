====================================================
Quiz
====================================================

This self-check quiz covers :doc:`Lecture 6 <l6_lecture>`.

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

      class Robot:
          def __init__(self, name):
              self.name = name


      a = Robot("Scout")
      b = Robot("Scout")
      print(a == b, a.name == b.name)

   A. ``True True``
   B. ``False True``
   C. ``False False``
   D. ``True False``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``False True``

   *Explanation:* ``Robot`` has no ``__eq__``, so ``==`` compares identity, and ``a`` and ``b`` are two objects. Their ``name`` attributes are equal strings.


.. admonition:: Question 2
   :class: hint

   Which call is the same as ``scout.perform_task("pick widget", 5)``, where ``scout`` is a ``Robot``?

   A. ``Robot.perform_task("pick widget", 5)``
   B. ``Robot.perform_task(scout, "pick widget", 5)``
   C. ``perform_task(scout, "pick widget", 5)``
   D. ``scout.perform_task(scout, "pick widget", 5)``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``Robot.perform_task(scout, "pick widget", 5)``

   *Explanation:* A method call passes the object as the first argument, ``self``. Answer D would pass ``scout`` twice, and answer C looks for a function that is not defined outside the class.


.. admonition:: Question 3
   :class: hint

   What does this program print?

   .. code-block:: python

      class Counter:
          total = 0

          def __init__(self):
              Counter.total += 1


      c1 = Counter()
      c2 = Counter()
      c1.total = 10
      print(Counter.total, c1.total, c2.total)

   A. ``10 10 10``
   B. ``2 10 2``
   C. ``2 2 2``
   D. ``3 10 3``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``2 10 2``

   *Explanation:* Two objects were made, so the class attribute is 2. ``c1.total = 10`` adds an instance attribute to ``c1`` only; ``c2.total`` is not in ``c2``, so Python finds the class's 2.


.. admonition:: Question 4
   :class: hint

   What does this program print?

   .. code-block:: python

      class Robot:
          tasks = []

          def add(self, t):
              self.tasks.append(t)


      a = Robot()
      b = Robot()
      a.add("pick")
      print(b.tasks)

   A. ``[]``
   B. ``['pick']``
   C. Nothing: ``AttributeError``
   D. ``None``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``['pick']``

   *Explanation:* ``tasks`` is a class attribute: one list for every robot. ``self.tasks.append(t)`` changes that list; it does not make an instance attribute. Make the list in ``__init__`` instead.


.. admonition:: Question 5
   :class: hint

   What does this program print?

   .. code-block:: python

      class Robot:
          def __repr__(self):
              return "Robot('Scout')"


      r = Robot()
      print(r, [r])

   A. ``Robot('Scout') [Robot('Scout')]``
   B. ``<__main__.Robot object at 0x...> [Robot('Scout')]``
   C. ``Robot('Scout') [<__main__.Robot object at 0x...>]``
   D. Nothing: ``TypeError``, because ``__str__`` is missing

.. dropdown:: Answer
   :class-container: sd-border-success

   **A**, ``Robot('Scout') [Robot('Scout')]``

   *Explanation:* A list uses ``__repr__`` for its items, and ``print(r)`` uses ``__repr__`` when the class has no ``__str__``.


.. admonition:: Question 6
   :class: hint

   A class defines ``__str__`` and no ``__repr__``. Which of these does **not** use its ``__str__``?

   A. ``print(x)``
   B. ``str(x)``
   C. ``f"{x}"``
   D. ``print([x])``

.. dropdown:: Answer
   :class-container: sd-border-success

   **D**, ``print([x])``

   *Explanation:* A list shows its items with ``__repr__``, so ``print([x])`` shows the default ``<__main__... object at 0x...>`` text.


.. admonition:: Question 7
   :class: hint

   What does this program print?

   .. code-block:: python

      class Sensor:
          def __init__(self, r):
              self.r = r

          def __lt__(self, other):
              return self.r < other.r


      print(Sensor(4.0) > Sensor(12.0))

   A. ``True``
   B. ``False``
   C. Nothing: ``TypeError``, because ``__gt__`` is missing
   D. ``NotImplemented``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``False``

   *Explanation:* ``Sensor`` defines no ``__gt__``, so Python calls the reflected method, ``Sensor(12.0).__lt__(Sensor(4.0))``: 12.0 < 4.0 is ``False``.


.. admonition:: Question 8
   :class: hint

   What does this program print?

   .. code-block:: python

      class Robot:
          def __init__(self, battery):
              if not 0 <= battery <= 100:
                  raise ValueError("bad battery")
              self.battery = battery


      try:
          r = Robot(150)
          print("made")
      except ValueError as err:
          print("error:", err)
      print("end")

   A. ``made`` then ``end``
   B. ``error: bad battery`` then ``end``
   C. ``error: bad battery`` only
   D. A traceback ending in ``ValueError: bad battery``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``error: bad battery`` then ``end``

   *Explanation:* ``raise`` stops ``__init__``, so the rest of the ``try`` block (``print("made")``) is skipped. The ``except`` block handles the error, and the program goes on.


.. admonition:: Question 9
   :class: hint

   What happens at the last line?

   .. code-block:: python

      class Robot:
          def __init__(self, name):
              self._name = name

          @property
          def name(self):
              return self._name


      r = Robot("Scout")
      r.name = "Bob"

   A. ``r.name`` is now ``"Bob"``
   B. ``r._name`` is now ``"Bob"``
   C. ``AttributeError``: the property has no setter
   D. A new instance attribute ``name`` hides the property

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``AttributeError``: the property has no setter

   *Explanation:* A property with no setter is read-only. Python 3.12 says: ``property 'name' of 'Robot' object has no setter``.


.. admonition:: Question 10
   :class: hint

   In the design phase, you read "a robot carries out tasks in an arena". Which is the best candidate for a method of the ``Robot`` class?

   A. ``arena()``, from "arena"
   B. ``task()``, from "tasks"
   C. ``perform_task()``, from "carries out"
   D. ``robot()``, from "robot"

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``perform_task()``

   *Explanation:* A verb is a candidate method of the class that does it. The nouns are candidate classes or attributes.


True/False
==========

.. admonition:: Question 11
   :class: hint

   **True or False:** An attribute with one leading underscore, such as
   ``_battery``, cannot be changed from outside the class.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* One underscore is a convention: "should be treated as a non-public part of the API". Python allows ``scout._battery = 5``; Ruff's rule SLF001 can report it.


.. admonition:: Question 12
   :class: hint

   **True or False:** Inside a class ``Robot``, the attribute name
   ``__battery`` is stored as ``_Robot__battery``.

.. dropdown:: Answer
   :class-container: sd-border-success

   **True**

   *Explanation:* This is name mangling: ``vars()`` of such an object shows ``{'_Robot__battery': 100}`` for a battery of 100.


.. admonition:: Question 13
   :class: hint

   **True or False:** An object of a class that defines ``__eq__`` but
   not ``__hash__`` cannot be a dictionary key.

.. dropdown:: Answer
   :class-container: sd-border-success

   **True**

   *Explanation:* Defining ``__eq__`` without ``__hash__`` sets ``__hash__`` to ``None``, so the object is unhashable: ``TypeError: unhashable type``.


.. admonition:: Question 14
   :class: hint

   **True or False:** For a class ``Vec2`` that defines ``__mul__`` and
   ``__rmul__``, the first method Python calls for ``4 * v`` is
   ``v.__mul__(4)``.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* The left operand, ``4``, goes first: Python first calls ``int``'s ``__mul__``, which returns ``NotImplemented`` for a ``Vec2``, and then the reflected method, ``v.__rmul__(4)``. Python does not call ``v.__mul__`` here; the lecture's ``__rmul__`` calls it.
