====================================================
Quiz
====================================================

This self-check quiz covers :doc:`Lecture 7 <l7_lecture>`.

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

      def shout(func):
          def wrapper():
              return func().upper()

          return wrapper


      @shout
      def status():
          return "active"


      print(status(), status.__name__)

   A. ``active status``
   B. ``ACTIVE wrapper``
   C. ``ACTIVE status``
   D. ``active wrapper``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``ACTIVE wrapper``

   *Explanation:* ``@shout`` binds ``status`` to ``wrapper``, which returns the original result in upper case. Without ``functools.wraps``, the name is the wrapper's.


.. admonition:: Question 2
   :class: hint

   In what order does this program print?

   .. code-block:: python

      def tag(func):
          print("tagging")
          return func


      @tag
      def f():
          return 1


      print("start")
      print(f())

   A. ``start``, ``tagging``, ``1``
   B. ``start``, ``1``
   C. ``tagging``, ``start``, ``1``
   D. ``start``, ``1``, ``tagging``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``tagging``, ``start``, ``1``

   *Explanation:* The ``@`` line calls the decorator once, right after the ``def`` runs, before any call of ``f``.


.. admonition:: Question 3
   :class: hint

   Which line does the same as ``@a`` on one line, ``@b`` on the next,
   then ``def f(): ...``?

   A. ``f = b(a(f))``
   B. ``f = a(b(f))``
   C. ``f = a(f) + b(f)``
   D. ``f = a(f)``, and ``b`` is ignored

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``f = a(b(f))``

   *Explanation:* The decorator closest to ``def`` applies first. Run with two decorators that print their names, the order is ``b`` then ``a``.


.. admonition:: Question 4
   :class: hint

   What does this program print?

   .. code-block:: python

      class Robot:
          def __init__(self, name):
              self.name = name

          @classmethod
          def make(cls, name):
              return cls(name)


      class Scout(Robot):
          pass


      print(type(Scout.make("x")).__name__)

   A. ``Robot``
   B. ``Scout``
   C. ``type``
   D. Nothing: ``TypeError``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``Scout``

   *Explanation:* A class method called on a subclass receives the subclass as ``cls``, so ``cls(name)`` makes a ``Scout``.


.. admonition:: Question 5
   :class: hint

   ``Robot`` has a method ``bar(level)`` decorated with
   ``@staticmethod``, and ``scout = Robot()``. Which calls work?

   A. ``Robot.bar(50)`` only
   B. ``scout.bar(50)`` only
   C. Both ``Robot.bar(50)`` and ``scout.bar(50)``
   D. Neither: a static method needs ``self``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, both

   *Explanation:* A static method receives no implicit first argument, from the class or from an object, so both calls pass only ``50``.


.. admonition:: Question 6
   :class: hint

   What does this program print?

   .. code-block:: python

      from dataclasses import dataclass


      @dataclass
      class Pose:
          x: float
          y: float = 0.0
          label = "origin"


      print(Pose(1.0))

   A. ``Pose(x=1.0, y=0.0)``
   B. ``Pose(x=1.0, y=0.0, label='origin')``
   C. Nothing: ``TypeError``, ``label`` has no value
   D. ``<__main__.Pose object at 0x...>``

.. dropdown:: Answer
   :class-container: sd-border-success

   **A**, ``Pose(x=1.0, y=0.0)``

   *Explanation:* ``label`` has no type annotation, so it is a class attribute, not a field: the generated ``__repr__`` and ``__init__`` leave it out.


.. admonition:: Question 7
   :class: hint

   Objects of which data class can go in a set? Each class has one
   field, ``x: int``.

   A. ``@dataclass``
   B. ``@dataclass(frozen=True)``
   C. ``@dataclass(slots=True)``
   D. None of them

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``@dataclass(frozen=True)``

   *Explanation:* With ``eq`` true (the default) and ``frozen`` true, ``@dataclass`` writes a ``__hash__``. In A and C, ``frozen`` is false, so ``__hash__`` is ``None``: ``TypeError: unhashable type``.


.. admonition:: Question 8
   :class: hint

   A ``Team`` receives ``Robot`` objects that the caller made, keeps them
   in a list, and the robots stay when the team is deleted. Which
   relationship is this?

   A. Association
   B. Aggregation
   C. Composition
   D. Inheritance

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, aggregation

   *Explanation:* The robots are the team's parts, made elsewhere, and they outlive the team. In composition the whole makes its parts; in an association the other object is known but is not a part.


.. admonition:: Question 9
   :class: hint

   What does this program print?

   .. code-block:: python

      class Robot:
          DRAIN = 2

          def drain(self, minutes):
              return minutes * self.DRAIN


      class Hauler(Robot):
          DRAIN = 3


      print(Robot().drain(10), Hauler().drain(10))

   A. ``20 20``
   B. ``20 30``
   C. ``30 30``
   D. Nothing: ``AttributeError``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``20 30``

   *Explanation:* ``drain()`` is found in ``Robot`` for both, but ``self.DRAIN`` is looked up from the object's own class first: ``Hauler`` has 3.


.. admonition:: Question 10
   :class: hint

   What happens when this program runs?

   .. code-block:: python

      from abc import ABC, abstractmethod


      class Reader(ABC):
          @abstractmethod
          def read(self): ...


      class Lidar(Reader):
          pass


      print("defined")
      x = Lidar()

   A. It prints ``defined``, then ``Lidar()`` raises ``TypeError``
   B. ``class Lidar(Reader):`` raises ``TypeError``; nothing is printed
   C. It prints ``defined``, and ``x.read()`` would return ``None``
   D. It prints ``defined`` and ends without an error

.. dropdown:: Answer
   :class-container: sd-border-success

   **A**

   *Explanation:* A subclass that does not define every abstract method can be defined, but not instantiated: the error comes when the object is made.


True/False
==========

.. admonition:: Question 11
   :class: hint

   **True or False:** When a subclass defines its own ``__init__``,
   Python also runs the base class's ``__init__``, before it.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* Python calls only the subclass's ``__init__``. It must call ``super().__init__(...)`` itself, or the base class sets none of its attributes.


.. admonition:: Question 12
   :class: hint

   **True or False:** An object of a class with
   ``__slots__ = ("x", "y")``, which inherits directly from ``object``,
   can be given a new attribute ``z``.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* The object has no ``__dict__``, so assigning a name not listed in ``__slots__`` raises ``AttributeError``. A base class without ``__slots__`` would bring a ``__dict__`` back.


.. admonition:: Question 13
   :class: hint

   **True or False:** mypy accepts an object for a parameter hinted with
   a protocol only if the object's class inherits from that protocol.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* A protocol is structural: mypy accepts any class with the names and types the protocol lists. In ``protocol_demo.py``, ``SimRobot`` fits ``Performer`` without inheriting from it.


.. admonition:: Question 14
   :class: hint

   **True or False:** ``isinstance(obj, P)``, for a protocol ``P``
   marked ``@runtime_checkable``, checks the types of the parameters of
   ``obj``'s methods.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* It checks only that the names exist. A class whose ``read_m`` takes three extra parameters and returns a string still passes against a protocol with ``read_m(self) -> float``.
