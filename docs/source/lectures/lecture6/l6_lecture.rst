====================================================
Lecture
====================================================

Every program on this page is in the ``lecture6/`` folder of the
`course repository <https://github.com/rubixcubic/enpm605-spring-2027-python>`_.
Run each one from the root of the repository, for example:

.. code-block:: bash

   uv run lecture6/robot_v1.py

Every output on this page was printed by running the program in the
course environment (``.venv``, what ``uv run`` uses): Python 3.12.3 on
Ubuntu 24.04. In paths, ``...`` marks a path shortened to fit. The
values in the examples (battery levels, task lengths, sensor ranges) are
chosen for the examples. A program that prints an object without a
``__repr__`` prints its memory address, which differs on every run.


Introduction to Object-Oriented Programming
====================================================

A Robot as a Dictionary, Changed by Functions
---------------------------------------------

.. code: lecture6/procedural_demo.py all

.. code-block:: python
   :caption: lecture6/procedural_demo.py

   """A robot as a dictionary, and functions that change it (Lecture 4)."""


   def make_robot(name, battery=100):
       return {"name": name, "battery": battery}


   def drain(robot, percent):
       robot["battery"] -= percent


   scout = make_robot("Scout")
   drain(scout, 30)  # 30%: chosen for this example
   print(scout)

   # Nothing checks a change made outside the functions
   scout["battery"] = 150
   scout["batery"] = 50  # A typo adds a new key
   print(scout)

.. output: procedural_demo.txt all

.. code-block:: text

   {'name': 'Scout', 'battery': 70}
   {'name': 'Scout', 'battery': 150, 'batery': 50}

- Lecture 4's ``make_robot()`` also returned a dictionary. The robot's
  data is one object; the functions that change it are separate.
- ``drain()`` works only if every caller passes a dictionary with a
  ``"battery"`` key.
- A battery level of 150% is accepted, and the misspelled key
  ``"batery"`` becomes a second battery. Any line of the program can
  change the dictionary, so a bug can come from anywhere.

.. note::

   **Object-oriented programming** (OOP) puts the data and the functions
   that use it together, in an object. A **class** says which data its
   objects hold (**attributes**) and what they can do (**methods**), and
   its methods can check every change.

Every Value Is an Object, and Its Type Is a Class
-------------------------------------------------

.. code: lecture6/objects_demo.py 3-12

.. code-block:: python
   :caption: From lecture6/objects_demo.py

   import numpy as np

   ranges = [2.5, 0.9]  # Meters, chosen for this example
   ranges.append(3.1)  # A method of the list object
   print(type(ranges), ranges)
   kind = "lidar"
   print(type(kind), kind.upper())
   scan = np.array(ranges)
   print(type(scan), scan.shape, scan.max())
   print(isinstance(scan, np.ndarray), isinstance(ranges, list))

.. output: objects_demo.txt all

.. code-block:: text

   <class 'list'> [2.5, 0.9, 3.1]
   <class 'str'> LIDAR
   <class 'numpy.ndarray'> (3,) 3.1
   True True

- ``type()`` returns an object's class: ``list``, ``str``,
  ``numpy.ndarray``. ``isinstance(x, C)`` asks whether ``x`` is an
  object of class ``C``, or of a subclass (Lecture 7).
- ``append()``, ``upper()`` and ``max()`` are methods: functions that
  belong to a class, called on an object with a dot. ``shape`` is an
  attribute. The Memory reading's "Names and Objects": an object holds
  a value and knows its type.

The Four Principles of OOP
--------------------------

.. list-table::
   :header-rows: 1
   :widths: 20 60 20
   :class: compact-table

   * - Principle
     - What it means
     - Taught in
   * - Encapsulation
     - An object keeps its data with the methods that change it, and
       controls every change
     - This lecture
   * - Abstraction
     - A caller uses what an object does, not how it does it
     - This lecture
   * - Inheritance
     - A new class reuses and extends an existing class
     - Lecture 7
   * - Polymorphism
     - Objects of different classes can be used the same way, through
       the same method names
     - Lecture 7

Python supports several styles: functions (Lecture 4), array code
(Lecture 5), and classes. You pick the one that fits the problem.

When a Class Helps
------------------

A class helps when:

- Several values belong together and rules apply to them: a battery
  level must stay from 0 to 100.
- The program needs many objects of one kind: several robots, each with
  its own battery.
- A library is built on classes. In this course's ROS 2 code (Lecture 8
  on), every node you write is a class, and every message is an object.

A function is enough when:

- The code is one computation on its inputs, with no data to keep
  between calls: Lecture 4's ``drive_time()``.
- A short script runs once. A class adds code to read and to design.


The Design Phase
====================================================

Five Steps from a Problem to Code
---------------------------------

1. **Requirements:** what the system must do.
2. **Business rules:** the limits the system must enforce.
3. **Nouns and verbs:** candidate classes, attributes, and methods, read
   from the requirements.
4. **Modeling:** draw each class, for example as a UML (Unified Modeling
   Language) class box.
5. **Implementation:** write the classes, one module per class.

The steps repeat: writing the code shows what the design missed, and you
go back to an earlier step. This section takes one example, a robotics
competition, through all five steps. The rest of the lecture writes its
``Robot`` class.

Step 1: What the System Must Do
-------------------------------

The problem, in the organizer's words:

   In our robotics competition, teams of robots carry out tasks in an
   arena. Our software tracks each robot's name, battery level, and
   status; it assigns tasks to robots and records the points each team
   scores. Each robot carries sensors, such as a lidar or a camera, and
   each sensor has a range.

Requirements read from it:

- Track each robot: its name, its battery level, and its status.
- Assign tasks to robots, and know whether a task was done.
- Record which robots form each team, and each team's points.
- Describe each robot's sensors and their ranges.

Step 2: Business Rules
----------------------

Rules for the competition, chosen for this example:

1. A robot's battery level is a whole percent from 0 to 100.
2. A task's priority is ``"low"``, ``"medium"``, ``"high"``, or
   ``"critical"``.
3. Below 10%, a robot's status is ``"needs_recharge"``, and it does no
   tasks.
4. A task uses 2% of the battery for each minute it takes.
5. A robot's name does not change after the robot is made.
6. A sensor's range is a positive number of meters.

Each rule becomes a check in a method, or a value computed by the class:
the dictionary of `A Robot as a Dictionary, Changed by Functions`_ had
no place for either. Rules about teams and scores come with the classes
that hold them, in Lecture 7.

Step 3: Nouns and Verbs
-----------------------

The same description, with the nouns in bold and the verbs in italics:

   In our robotics **competition**, **teams** of **robots** *carry out*
   **tasks** in an **arena**. Our **software** *tracks* each robot's
   **name**, **battery level**, and **status**; it *assigns* tasks to
   robots and *records* the **points** each team *scores*. Each robot
   carries **sensors**, such as a **lidar** or a **camera**, and each
   sensor has a **range**.

- A noun that has data of its own and things it does is a candidate
  class: robot, task, team, sensor, arena.
- A noun that describes another noun is an attribute: a robot's name,
  battery level, and status; a sensor's range.
- A verb is a candidate method of the class that does it: a robot
  carries out a task, so ``Robot`` gets ``perform_task()``.

.. list-table::
   :header-rows: 1
   :widths: 14 28 28 30
   :class: compact-table

   * - Class
     - Attributes
     - Methods
     - Written in
   * - ``Robot``
     - ``name``, ``battery``, ``status``
     - ``perform_task()``, ``recharge()``
     - This lecture
   * - ``Sensor``
     - ``kind``, ``range_m``
     - none yet
     - Exercise 3
   * - ``Task``
     - ``name``, ``priority``, ``minutes``
     - none yet
     - Exercise 4
   * - ``Team``, ``Arena``
     - ``name``, ``score``; size
     - ``add_robot()``
     - Lecture 7

Not every noun becomes a class:

- "Competition" is the whole system, and "software" is the program
  itself.
- "Lidar" and "camera" are values of a sensor's ``kind``, not classes of
  their own.
- "Points" is a number kept by a team: an attribute (``score``).

Step 4: A UML Class Box for Robot
---------------------------------

.. figure:: /_static/images/l6/uml_robot.svg
   :alt: A UML class box with three compartments. Tag 1, the top: the class name Robot. Tag 2, the middle: the attributes, one per line: plus name: str, marked readOnly in braces; plus battery: int; plus slash status: str; plus total_robots: int, underlined. Tag 3, the bottom: the operations: plus perform_task(task: str, minutes: int): bool; plus recharge().
   :width: 70%

   1: the class name. 2: the attributes, ``name: type``. 3: the methods,
   with their parameters and return type.

- UML is a standard way to draw a design. ``+`` marks a public member,
  one callers may use; a ``-`` would mark a non-public one, for the
  class's own code only.
- ``{readOnly}``: cannot change (rule 5). ``/``: computed from other
  attributes (rule 3). Underlined: one value shared by every robot.

Step 5: One Module per Class
----------------------------

The package ``lecture6/robotics_competition/`` (Lecture 2: a package is
a folder):

.. list-table::
   :header-rows: 1
   :widths: 30 35 35
   :class: compact-table

   * - Module
     - Holds
     - Written in
   * - ``__init__.py``
     - The package's docstring
     - This lecture
   * - ``robot.py``
     - ``Robot``
     - This lecture
   * - ``sensor.py``, ``task.py``
     - ``Sensor``, ``Task``
     - Exercises 3 and 4, by you
   * - ``team.py``, ``arena.py``
     - ``Team``, ``Arena``
     - Lecture 7

A program next to the package imports a class by its module:
``from robotics_competition.robot import Robot``. One class per module
keeps each file short, and a change to one class touches one file.


The Implementation Phase
====================================================

A class Statement Makes a Class; Calling It Makes an Object
-----------------------------------------------------------

.. code: lecture6/first_class.py all

.. code-block:: python
   :caption: lecture6/first_class.py

   """The smallest class, and two objects made from it."""


   class Robot:
       """A robot in the competition."""


   scout = Robot()
   hauler = Robot()
   print(type(Robot), type(scout))
   print(scout is hauler, isinstance(scout, Robot))
   print(scout)

.. output: first_class.txt all

.. code-block:: text

   <class 'type'> <class '__main__.Robot'>
   False True
   <__main__.Robot object at 0x7b00aba7a180>

- ``class Robot:`` makes a class object and binds the name ``Robot`` to
  it. Its body here is only a docstring. PEP 8: "Class names should
  normally use the CapWords convention."
- ``Robot()`` calls the class and returns a new object, an **instance**
  of ``Robot``. Two calls make two objects, so ``scout is hauler`` is
  ``False``.
- With no method to print it, an object prints as its module, its
  class, and its address in hex (``id()``). The address differs on every
  run.

__init__ Gives Each New Object Its Attributes
---------------------------------------------

.. code: lecture6/robot_v1.py 4-16

.. code-block:: python
   :caption: From lecture6/robot_v1.py

   class Robot:
       """A robot in the competition."""

       def __init__(self, name: str, battery: int = 100):
           self.name = name
           self.battery = battery  # Percent, 0 to 100

       def perform_task(self, task: str, minutes: int) -> None:
           self.battery -= 2 * minutes  # 2% per minute: an example value
           print(f"{self.name} did {task}: battery {self.battery}%")

       def recharge(self) -> None:
           self.battery = 100

- A ``def`` in the class body defines a **method**. Its first parameter,
  ``self``, is the object it works on.
- The tutorial: "class instantiation automatically invokes
  ``__init__()`` for the newly created class instance."
- ``self.name = name`` makes an attribute ``name`` on that object.
  ``__init__`` is a dunder method (`Dunder Methods`_).

Making Two Robots, Step by Step
-------------------------------

.. code: lecture6/robot_v1.py 19-21

.. code-block:: python

   scout = Robot("Scout")
   hauler = Robot("Hauler", battery=60)
   print(scout.name, scout.battery, hauler.name, hauler.battery)

.. output: robot_v1.txt 1

.. code-block:: text

   Scout 100 Hauler 60

What ``Robot("Scout")`` does:

1. Python makes a new, empty ``Robot`` object.
2. It calls ``__init__`` with ``self`` bound to that object,
   ``name="Scout"``, and ``battery=100``, the default.
3. ``__init__`` sets the object's two attributes and returns ``None``.
4. ``Robot("Scout")`` returns the object, and ``scout = ...`` binds the
   name ``scout`` to it.

You never pass ``self``: the call has one argument, ``"Scout"``, for the
parameter ``name``.

A Method Call Passes the Object as self
---------------------------------------

.. code: lecture6/robot_v1.py 23-28

.. code-block:: python

   scout.perform_task("pick widget", 5)
   print(scout.battery, hauler.battery)

   Robot.perform_task(hauler, "carry crate", 10)  # The same kind of call
   hauler.recharge()
   print(hauler.battery)

.. output: robot_v1.txt 2-5

.. code-block:: text

   Scout did pick widget: battery 90%
   90 60
   Hauler did carry crate: battery 40%
   100

- The tutorial: "the call ``x.f()`` is exactly equivalent to
  ``MyClass.f(x)``." So ``scout.perform_task("pick widget", 5)`` runs
  ``Robot.perform_task(scout, "pick widget", 5)``.
- ``scout`` lost 10% (5 minutes at 2% per minute); ``hauler`` kept its
  60%. A method changes only the object it was called on.
- ``hauler`` lost 20% (10 minutes), from 60% to 40%, then
  ``recharge()`` set it to 100%.

.. figure:: /_static/images/l6/self_binding.svg
   :alt: Tag 1: a gray box, the module's namespace, holds the name scout. Tag 2: a gray box, the namespace of the call perform_task, holds the names self, task and minutes. Tag 3: a red box, a Robot object, holds its own names name and battery, with arrows to a str object 'Scout' and an int object 100. Both scout and self have arrows to the Robot object. task has an arrow to a str object 'pick widget', and minutes to an int object 5.
   :width: 70%

   The names while ``scout.perform_task("pick widget", 5)`` runs, as the
   call starts. 1: the module's names. 2: the names of the call. 3: the
   ``Robot`` object, with its own names.

``scout`` and ``self`` refer to the same object, so
``self.battery -= 2 * minutes`` (10 for 5 minutes) changes the robot that ``scout`` refers to. The
call's names disappear when it returns; the object stays, because
``scout`` still refers to it (Memory reading, "Call Frames").

A Method Without self Fails When It Is Called
---------------------------------------------

.. code: lecture6/errors/missing_self.py all

.. code-block:: python
   :caption: lecture6/errors/missing_self.py

   class Robot:
       def report():  # self is missing
           print("Reporting")


   scout = Robot()
   scout.report()

.. output: err_missing_self.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../missing_self.py", line 7, in <module>
       scout.report()
   TypeError: Robot.report() takes 0 positional arguments but 1 was given

``scout.report()`` passes ``scout`` as the first argument, but
``report()`` has no parameter for it. The class itself is defined
without an error: the mistake shows only at the first call.


Classes and Objects
====================================================

Each Object Keeps Its Attributes in Its Own Namespace
-----------------------------------------------------

.. code: lecture6/attributes_demo.py 4-18

.. code-block:: python
   :caption: From lecture6/attributes_demo.py

   class Robot:
       """A robot in the competition."""

       total_robots = 0  # A class attribute: one value, shared

       def __init__(self, name: str, battery: int = 100):
           self.name = name  # Instance attributes: one per object
           self.battery = battery
           Robot.total_robots += 1


   scout = Robot("Scout")
   hauler = Robot("Hauler", 60)
   print(vars(scout))
   print(vars(hauler))

.. output: attributes_demo.txt 1-2

.. code-block:: text

   {'name': 'Scout', 'battery': 100}
   {'name': 'Hauler', 'battery': 60}

``vars(scout)`` returns the object's namespace, a dictionary from
attribute names to objects (Memory reading, "Names Live in
Namespaces"). Each object has its own.

A Class Attribute Is Shared by Every Object
-------------------------------------------

.. code: lecture6/attributes_demo.py 19-20

.. code-block:: python

   print(Robot.total_robots, scout.total_robots)
   print("total_robots" in vars(scout), "total_robots" in vars(Robot))

.. output: attributes_demo.txt 3-4

.. code-block:: text

   2 2
   False True

- ``total_robots = 0`` in the class body is a **class attribute**: one
  name in the class's namespace, not in any object's. Two calls to
  ``__init__`` added 1 each: 2.
- ``scout.total_robots`` also reads 2, although ``"total_robots"`` is
  not in ``vars(scout)``: `Python Looks in the Object First, Then in
  Its Class`_.
- The search never goes from the class to an object: ``Robot.name``
  raises ``AttributeError: type object 'Robot' has no attribute
  'name'`` (``lecture6/errors/instance_attr_on_class.py``).
- The tutorial: "instance variables are for data unique to each
  instance and class variables are for attributes and methods shared by
  all instances of the class".

Python Looks in the Object First, Then in Its Class
---------------------------------------------------

.. figure:: /_static/images/l6/class_lookup.svg
   :alt: A gray box at the left, the module's namespace, holds the names scout and Robot. Tag 1: scout has an arrow to a red box, a Robot object, that holds its own names name and battery, with arrows to a str object 'Scout' and an int object 100. Tag 2: Robot has an arrow to a red box, the class object, of type type, that holds the names total_robots, with an arrow to an int object 2, and __init__, with an arrow to a function object. Tag 3: a dashed arrow from the Robot object down to the class object: the object's link to its class.
   :width: 70%

   1: the object's namespace, ``vars(scout)``. 2: the class's namespace,
   ``vars(Robot)``. 3: the object's link to its class, ``type(scout)``.

- ``scout.battery``: found at 1. ``scout.total_robots``: not at 1, so
  Python follows 3 and finds it at 2.
- Methods are found the same way: ``__init__`` is a name in the class,
  used by every object.

Assigning Through an Object Makes an Instance Attribute
-------------------------------------------------------

.. code: lecture6/attributes_demo.py 22-25

.. code-block:: python

   # Assigning through an object makes an instance attribute
   scout.total_robots = 99
   print(scout.total_robots, hauler.total_robots, Robot.total_robots)
   print(vars(scout))

.. output: attributes_demo.txt 5-6

.. code-block:: text

   99 2 2
   {'name': 'Scout', 'battery': 100, 'total_robots': 99}

- Reading looks in the object, then the class. Assigning always writes
  to the object's own namespace.
- ``scout.total_robots = 99`` adds ``total_robots`` to ``scout``'s
  namespace. It hides the class attribute for ``scout`` only; ``hauler``
  and ``Robot`` still read 2.
- So ``__init__`` writes ``Robot.total_robots += 1``. With
  ``self.total_robots += 1``, each object would read 0 from the class
  and keep 1 for itself, and the class would stay at 0.

A Misspelled Name: Assigning Works, Reading Fails
-------------------------------------------------

.. code: lecture6/attributes_demo.py 27-29

.. code-block:: python

   # A misspelled name in an assignment makes one more attribute
   hauler.batery = 50
   print(hauler.battery, vars(hauler))

.. output: attributes_demo.txt 7

.. code-block:: text

   60 {'name': 'Hauler', 'battery': 60, 'batery': 50}

.. code: lecture6/errors/misspelled_read.py all

.. code-block:: python
   :caption: lecture6/errors/misspelled_read.py

   class Robot:
       def __init__(self, name: str, battery: int = 100):
           self.name = name
           self.battery = battery


   scout = Robot("Scout")
   print(scout.batery)

.. output: err_misspelled_read.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../misspelled_read.py", line 8, in <module>
       print(scout.batery)
             ^^^^^^^^^^^^
   AttributeError: 'Robot' object has no attribute 'batery'. Did you mean: 'battery'?

- An assignment with a misspelled name adds a new attribute, ``batery``,
  with no error; ``battery`` keeps 60.
- Reading a name that exists nowhere raises ``AttributeError``, and
  Python suggests the closest name.
- Lecture 7's ``__slots__`` makes the misspelled assignment an
  ``AttributeError`` too.

Create Every Attribute in __init__
----------------------------------

.. code: lecture6/errors/attr_outside_init.py all

.. code-block:: python
   :caption: lecture6/errors/attr_outside_init.py

   class Robot:
       def __init__(self, name: str):
           self.name = name

       def perform_task(self, task: str) -> None:
           self.last_task = task  # Made only when this method runs

       def report(self) -> None:
           print(self.name, self.last_task)


   scout = Robot("Scout")
   scout.report()

.. output: err_attr_outside_init.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../attr_outside_init.py", line 13, in <module>
       scout.report()
     File ".../attr_outside_init.py", line 9, in report
       print(self.name, self.last_task)
                        ^^^^^^^^^^^^^^
   AttributeError: 'Robot' object has no attribute 'last_task'

``last_task`` exists only after ``perform_task()`` has run once. Write
``self.last_task = None`` in ``__init__``, so every attribute is listed
in one place.

A List as a Class Attribute Is One List for Every Object
--------------------------------------------------------

.. code: lecture6/errors/shared_log.py all

.. code-block:: python
   :caption: lecture6/errors/shared_log.py

   class Robot:
       log = []  # A class attribute: one list for every robot

       def __init__(self, name: str):
           self.name = name

       def perform_task(self, task: str) -> None:
           self.log.append(task)


   scout = Robot("Scout")
   hauler = Robot("Hauler")
   scout.perform_task("pick widget")
   print(hauler.log, scout.log is hauler.log)

.. output: err_shared_log.txt all

.. code-block:: text

   ['pick widget'] True

- ``self.log.append(task)`` does not assign ``log``: it finds the
  class's list and changes it. Every robot sees ``"pick widget"``, as
  with Lecture 4's mutable default.
- The fix: ``self.log = []`` in ``__init__``, one list per robot.

Ruff reports it:

.. output: ruff_shared_log.txt all

.. code-block:: text
   :caption: uv run ruff check lecture6/errors/shared_log.py

   RUF012 Mutable default value for class attribute
    --> lecture6/errors/shared_log.py:2:11
     |
   1 | class Robot:
   2 |     log = []  # A class attribute: one list for every robot
     |           ^^
   3 |
   4 |     def __init__(self, name: str):
     |
   help: Consider initializing in `__init__` or annotating with `typing.ClassVar`

   Found 1 error.

Exercise 1 on the :doc:`exercises page <l6_exercises>` practices this
section and the one before it.


Dunder Methods
====================================================

__str__ Sets the Text print() Shows
-----------------------------------

.. code: lecture6/str_demo.py 4-18

.. code-block:: python
   :caption: From lecture6/str_demo.py

   class Robot:
       """A robot in the competition."""

       def __init__(self, name: str, battery: int = 100):
           self.name = name
           self.battery = battery

       def __str__(self) -> str:
           return f"{self.name} ({self.battery}%)"


   scout = Robot("Scout", 80)
   print(scout)
   print(str(scout), f"Robot: {scout}")
   print([scout])

.. output: str_demo.txt all

.. code-block:: text

   Scout (80%)
   Scout (80%) Robot: Scout (80%)
   [<__main__.Robot object at 0x7b0267d7a5a0>]

- A **dunder method** ("double underscore") such as ``__str__`` is
  called by Python from a built-in operation.
- ``print()``, ``str()`` and f-strings call ``__str__``. A list uses
  ``__repr__`` for its items: still the default.

__repr__ Sets the Text for Developers
-------------------------------------

``Robot`` has both methods; ``Sensor`` has only ``__repr__``.

.. code: lecture6/repr_demo.py 4-34

.. code-block:: python
   :caption: From lecture6/repr_demo.py

   class Robot:
       """A robot in the competition."""

       def __init__(self, name: str, battery: int = 100):
           self.name = name
           self.battery = battery

       def __repr__(self) -> str:
           return f"Robot({self.name!r}, battery={self.battery})"

       def __str__(self) -> str:
           return f"{self.name} ({self.battery}%)"


   class Sensor:
       """A range sensor, with __repr__ only."""

       def __init__(self, kind: str, range_m: float):
           self.kind = kind
           self.range_m = range_m

       def __repr__(self) -> str:
           return f"Sensor({self.kind!r}, {self.range_m})"


   scout = Robot("Scout", 80)
   print(repr(scout))
   print([scout, Robot("Hauler")])
   print(f"{scout} and {scout!r}")
   lidar = Sensor("lidar", 12.0)  # 12.0 m: the lidar of Lecture 2
   print(lidar, str(lidar))

.. output: repr_demo.txt all

.. code-block:: text

   Robot('Scout', battery=80)
   [Robot('Scout', battery=80), Robot('Hauler', battery=100)]
   Scout (80%) and Robot('Scout', battery=80)
   Sensor('lidar', 12.0) Sensor('lidar', 12.0)

- The reference
  (`object.__repr__ <https://docs.python.org/3.12/reference/datamodel.html#object.__repr__>`_):
  ``__repr__`` "should look like a valid Python expression that could be
  used to recreate an object with the same value".
  ``Robot('Scout', battery=80)`` does.
- ``!r`` in an f-string calls ``repr()`` on the value, so a string keeps
  its quotes.
- ``Sensor`` has no ``__str__``, so ``print()`` and ``str()`` use its
  ``__repr__``.

Which Method Each Operation Calls
---------------------------------

.. list-table::
   :header-rows: 1
   :widths: 45 55
   :class: compact-table

   * - Operation
     - Method called
   * - ``print(x)``, ``str(x)``, ``f"{x}"``
     - ``__str__``, or ``__repr__`` if the class has no ``__str__``
   * - ``repr(x)``, ``f"{x!r}"``
     - ``__repr__``
   * - ``print([x])``: an item of a list, tuple, or dictionary
     - ``__repr__``
   * - A class with neither
     - The default: ``<__main__.Robot object at 0x...>``

Write ``__repr__`` for every class: it is what you see in lists, in the
debugger, and in error messages. Add ``__str__`` when users need a
shorter text. Both must return a ``str``.

Without __eq__, == Compares Identity; Without __lt__, sorted() Fails
--------------------------------------------------------------------

.. code: lecture6/errors/default_eq.py all

.. code-block:: python
   :caption: lecture6/errors/default_eq.py

   class Sensor:
       def __init__(self, kind: str, range_m: float):
           self.kind = kind
           self.range_m = range_m


   lidar = Sensor("lidar", 12.0)
   same = lidar  # A second name for the same object
   print(lidar == Sensor("lidar", 12.0), lidar == same)
   print(sorted([lidar, Sensor("sonar", 4.0)]))

.. output: err_default_eq.txt all short

.. code-block:: text

   False True
   Traceback (most recent call last):
     File ".../default_eq.py", line 10, in <module>
       print(sorted([lidar, Sensor("sonar", 4.0)]))
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   TypeError: '<' not supported between instances of 'Sensor' and 'Sensor'

- The reference: "By default, ``object`` implements ``__eq__()`` by
  using ``is``". Two sensors with the same values are two objects, so
  ``==`` gives ``False``.
- "order comparisons raise ``TypeError``", so ``sorted()`` fails on the
  first ``<``.

__eq__ Compares Values, and Returns NotImplemented for Other Types
------------------------------------------------------------------

.. code: lecture6/sensor_compare.py 4-28

.. code-block:: python
   :caption: From lecture6/sensor_compare.py

   class Sensor:
       """A range sensor: its kind and its range in meters."""

       def __init__(self, kind: str, range_m: float):
           self.kind = kind
           self.range_m = range_m

       def __repr__(self) -> str:
           return f"Sensor({self.kind!r}, {self.range_m})"

       def __eq__(self, other):
           if not isinstance(other, Sensor):
               return NotImplemented
           return self.kind == other.kind and self.range_m == other.range_m

       def __lt__(self, other):
           if not isinstance(other, Sensor):
               return NotImplemented
           return self.range_m < other.range_m


   # Ranges in meters, chosen for this example
   lidar = Sensor("lidar", 12.0)
   print(lidar == Sensor("lidar", 12.0), lidar is Sensor("lidar", 12.0))
   print(lidar == "lidar", lidar != Sensor("sonar", 4.0))

.. output: sensor_compare.txt 1-2

.. code-block:: text

   True False
   False True

- ``a == b`` calls ``a.__eq__(b)``. Equal values now give ``True``;
  ``is`` still gives ``False``.
- ``NotImplemented`` is a value, not an error: Python then tries the
  other operand, and ``==`` falls back to ``is``: ``False``.
- The reference: for ``__ne__``, "by default it delegates to
  ``__eq__()`` and inverts the result unless it is ``NotImplemented``".

__lt__ Lets sorted(), min(), and > Compare Sensors
--------------------------------------------------

.. code: lecture6/sensor_compare.py 30-32

.. code-block:: python

   sensors = [lidar, Sensor("sonar", 4.0), Sensor("camera", 10.0)]
   print(sorted(sensors))
   print(min(sensors), lidar > Sensor("camera", 10.0))

.. output: sensor_compare.txt 3-4

.. code-block:: text

   [Sensor('sonar', 4.0), Sensor('camera', 10.0), Sensor('lidar', 12.0)]
   Sensor('sonar', 4.0) True

- ``sorted()`` and ``min()`` compare items with ``<`` only: by range,
  4.0 m, 10.0 m, 12.0 m.
- ``>``: ``Sensor`` defines no ``__gt__``, and the default one returns
  ``NotImplemented``, so Python calls the reflected method: the camera's
  ``__lt__``, with ``lidar`` as ``other``. The reference: "``__lt__()``
  and ``__gt__()`` are each other's reflection".

A Class with __eq__ and No __hash__ Cannot Go in a Set
------------------------------------------------------

.. code: lecture6/errors/unhashable.py all

.. code-block:: python
   :caption: lecture6/errors/unhashable.py

   class Sensor:
       def __init__(self, kind: str, range_m: float):
           self.kind = kind
           self.range_m = range_m

       def __eq__(self, other):
           if not isinstance(other, Sensor):
               return NotImplemented
           return self.kind == other.kind and self.range_m == other.range_m


   print(Sensor.__hash__)
   mounted = {Sensor("lidar", 12.0)}

.. output: err_unhashable.txt all short

.. code-block:: text

   None
   Traceback (most recent call last):
     File ".../unhashable.py", line 13, in <module>
       mounted = {Sensor("lidar", 12.0)}
                 ^^^^^^^^^^^^^^^^^^^^^^^
   TypeError: unhashable type: 'Sensor'

- The reference
  (`object.__hash__ <https://docs.python.org/3.12/reference/datamodel.html#object.__hash__>`_):
  "A class that overrides ``__eq__()`` and does not define
  ``__hash__()`` will have its ``__hash__()`` implicitly set to
  ``None``."
- Set items and dictionary keys must be hashable (Lecture 3), so these
  sensors cannot be either. A class whose objects never change can
  define ``__hash__``; Lecture 7's data classes can write it for you.

__add__, __mul__, __rmul__, and __abs__ for a 2D Vector
-------------------------------------------------------

.. code: lecture6/vec2_demo.py all

.. code-block:: python
   :caption: lecture6/vec2_demo.py

   """+, *, and abs() for a 2D vector class."""

   import math


   class Vec2:
       """A 2D vector, in meters."""

       def __init__(self, x: float, y: float):
           self.x = x
           self.y = y

       def __repr__(self) -> str:
           return f"Vec2({self.x}, {self.y})"

       def __add__(self, other):
           if not isinstance(other, Vec2):
               return NotImplemented
           return Vec2(self.x + other.x, self.y + other.y)

       def __mul__(self, k):
           if not isinstance(k, (int, float)):
               return NotImplemented
           return Vec2(self.x * k, self.y * k)

       def __rmul__(self, k):
           return self.__mul__(k)

       def __abs__(self) -> float:
           return math.hypot(self.x, self.y)


   # Positions and steps in meters, chosen for this example
   start = Vec2(1.0, 2.0)
   step = Vec2(0.5, 0.0)
   print(start + step, step * 4, 4 * step)
   print(start + 4 * step, abs(Vec2(3.0, 4.0)))
   print(start + 3)  # No rule for Vec2 + int: TypeError

.. output: vec2_demo.txt all short

.. code-block:: text

   Vec2(1.5, 2.0) Vec2(2.0, 0.0) Vec2(2.0, 0.0)
   Vec2(3.0, 2.0) 5.0
   Traceback (most recent call last):
     File ".../vec2_demo.py", line 38, in <module>
       print(start + 3)  # No rule for Vec2 + int: TypeError
             ~~~~~~^~~
   TypeError: unsupported operand type(s) for +: 'Vec2' and 'int'

- ``a + b`` calls ``a.__add__(b)``; ``a * k`` calls ``a.__mul__(k)``.
  Each returns a new ``Vec2`` and leaves both operands unchanged.
- A vector times a vector is not defined here, so ``__mul__`` returns
  ``NotImplemented`` for anything but a number.
- ``4 * step`` first tries ``int``'s multiplication, which cannot
  multiply a ``Vec2`` and returns ``NotImplemented``. Python then calls
  ``step.__rmul__(4)``, the reflected method. The reference: the
  reflected methods "are only called if the left operand does not
  support the corresponding operation and the operands are of different
  types."
- ``start + 4 * step``: x is 1.0 + 4 × 0.5 = 3.0 m, y stays 2.0 m.
  ``abs(Vec2(3.0, 4.0))`` calls ``__abs__``: √(3² + 4²) = 5.0 m.
- ``start + 3``: ``Vec2.__add__`` and ``int``'s reflected addition both
  return ``NotImplemented``, so Python raises ``TypeError``.
- For many points, use a NumPy array (Lecture 5). A small class shows
  how the operators work.

len(), in, for, and bool() on a TaskLog
---------------------------------------

.. code: lecture6/task_log_demo.py 4-29

.. code-block:: python
   :caption: From lecture6/task_log_demo.py

   class TaskLog:
       """The tasks a robot has done, in order."""

       def __init__(self):
           self.tasks = []

       def add(self, task: str) -> None:
           self.tasks.append(task)

       def __len__(self) -> int:
           return len(self.tasks)

       def __contains__(self, task: str) -> bool:
           return task in self.tasks

       def __iter__(self):
           return iter(self.tasks)


   log = TaskLog()
   print(len(log), bool(log))
   log.add("pick widget")
   log.add("carry crate")
   print(len(log), bool(log), "carry crate" in log)
   for task in log:
       print("-", task)

.. output: task_log_demo.txt all

.. code-block:: text

   0 False
   2 True True
   - pick widget
   - carry crate

- ``len(log)`` calls ``log.__len__()``, and ``x in log`` calls
  ``log.__contains__(x)``.
- A ``for`` loop calls ``__iter__``, which must return an iterator
  (Lecture 3). The list's own iterator does the work.
- Each method hands the work to the list: the caller uses a ``TaskLog``
  and never sees the list.
- ``bool(log)`` uses ``__len__``: a length of 0 is ``False``, as for an
  empty list (Lecture 2). The reference: "an object that doesn't define
  a ``__bool__()`` method and whose ``__len__()`` method returns zero is
  considered to be false in a Boolean context".

Exercise 2 on the :doc:`exercises page <l6_exercises>` practices this
section.


Abstraction and Encapsulation
====================================================

Abstraction: Use What an Object Does, Not How It Does It
--------------------------------------------------------

.. note::

   **Abstraction** shows a caller what an object does and hides how it
   does it. The caller depends on the object's **public interface**:
   the methods and attributes it is meant to use, with their docstrings.

- You call ``len(scan)`` and ``ranges.append(3.1)`` without reading how
  lists and arrays store their items.
- A caller of ``hauler.recharge()`` needs its docstring, "Fill the
  battery to 100%", not its code. The class can change how it recharges
  without changing any caller.
- This lecture: docstrings, and the names a caller is meant to use
  (`help() Lists the Public Interface`_). Lecture 7: abstract base
  classes, which require a subclass to define a set of methods.

Without Encapsulation, Any Line Can Break an Object
---------------------------------------------------

.. code: lecture6/errors/battery_string.py all

.. code-block:: python
   :caption: lecture6/errors/battery_string.py

   class Robot:
       def __init__(self, name: str, battery: int = 100):
           self.name = name
           self.battery = battery

       def perform_task(self, task: str, minutes: int) -> None:
           self.battery -= 2 * minutes


   scout = Robot("Scout")
   scout.battery = "full"  # Any code can assign any value
   scout.perform_task("pick widget", 5)

.. output: err_battery_string.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../battery_string.py", line 12, in <module>
       scout.perform_task("pick widget", 5)
     File ".../battery_string.py", line 7, in perform_task
       self.battery -= 2 * minutes
   TypeError: unsupported operand type(s) for -=: 'str' and 'int'

- The bad value goes in at line 11; the error shows at line 7, inside
  the method. In a large program, the two lines can be far apart.
- **Encapsulation**: the class keeps its data with its methods and
  checks every change. The rest of this section builds it.

raise Stops a Call That Gets a Bad Value
----------------------------------------

.. code: lecture6/raise_demo.py all

.. code-block:: python
   :caption: lecture6/raise_demo.py

   """raise stops __init__ when it gets a battery level out of range."""


   class Robot:
       """A robot in the competition."""

       def __init__(self, name: str, battery: int = 100):
           if not 0 <= battery <= 100:
               raise ValueError(f"battery must be 0 to 100, got {battery}")
           self.name = name
           self.battery = battery


   scout = Robot("Scout", 80)
   print(scout.name, scout.battery)
   hauler = Robot("Hauler", 150)
   print("This line does not run")

.. output: raise_demo.txt all short

.. code-block:: text

   Scout 80
   Traceback (most recent call last):
     File ".../raise_demo.py", line 16, in <module>
       hauler = Robot("Hauler", 150)
                ^^^^^^^^^^^^^^^^^^^^
     File ".../raise_demo.py", line 9, in __init__
       raise ValueError(f"battery must be 0 to 100, got {battery}")
   ValueError: battery must be 0 to 100, got 150

- ``raise ValueError("...")`` makes an exception with that message and
  stops the call: ``hauler`` is never bound, and the last ``print()``
  never runs.
- With no handler, Python prints the traceback and stops the program.
- Pick the type that names the problem. The
  `built-in exceptions <https://docs.python.org/3.12/library/exceptions.html>`_
  page: ``ValueError`` is "Raised when an operation or function receives
  an argument that has the right type but an inappropriate value";
  ``TypeError`` is "Raised when an operation or function is applied to
  an object of inappropriate type".

try and except Handle an Exception
----------------------------------

.. code: lecture6/try_demo.py 14-20

.. code-block:: python
   :caption: From lecture6/try_demo.py, after the same class

   for level in [80, 150, 5]:  # Battery levels, chosen for this example
       try:
           robot = Robot("Hauler", level)
           print("Made", robot.name, "with", robot.battery)
       except ValueError as err:
           print("Not made:", err)
   print("Done")

.. output: try_demo.txt all

.. code-block:: text

   Made Hauler with 80
   Not made: battery must be 0 to 100, got 150
   Made Hauler with 5
   Done

- Python runs the ``try`` block. If it raises a ``ValueError``, the rest
  of the block is skipped and the ``except`` block runs, with ``err``
  bound to the exception. Then the program goes on.
- ``print(err)`` prints the message given to ``raise``.
- PEP 8: "When catching exceptions, mention specific exceptions whenever
  possible instead of using a bare ``except:`` clause". A bare
  ``except:`` also hides your own bugs.

A Leading Underscore Marks a Name as Non-Public
-----------------------------------------------

.. code: lecture6/underscore_demo.py all

.. code-block:: python
   :caption: lecture6/underscore_demo.py

   """A leading underscore marks an attribute as non-public."""


   class Robot:
       """A robot in the competition."""

       def __init__(self, name: str, battery: int = 100):
           self._name = name  # Non-public: use the methods
           self._battery = battery

       def report(self) -> str:
           return f"{self._name}: {self._battery}%"


   scout = Robot("Scout")
   print(scout.report())
   scout._battery = 5  # Python allows it; Ruff's SLF001 reports it
   print(scout.report())

.. output: underscore_demo.txt all

.. code-block:: text

   Scout: 100%
   Scout: 5%

The tutorial
(`Private Variables <https://docs.python.org/3.12/tutorial/classes.html#private-variables>`_):
"a name prefixed with an underscore (e.g. ``_spam``) should be treated
as a non-public part of the API". Python itself does not stop
``scout._battery = 5``. Ruff reports it when you select its rule SLF001:

.. output: ruff_slf001.txt all

.. code-block:: text
   :caption: uv run ruff check --select SLF001 lecture6/underscore_demo.py

   SLF001 Private member accessed: `_battery`
     --> lecture6/underscore_demo.py:17:1
      |
   15 | scout = Robot("Scout")
   16 | print(scout.report())
   17 | scout._battery = 5  # Python allows it; Ruff's SLF001 reports it
      | ^^^^^^^^^^^^^^
   18 | print(scout.report())
      |

   Found 1 error.

Two Leading Underscores Change the Name
---------------------------------------

.. code: lecture6/errors/mangled_name.py all

.. code-block:: python
   :caption: lecture6/errors/mangled_name.py

   class Robot:
       def __init__(self, battery: int = 100):
           self.__battery = battery  # Two underscores: the name is mangled


   scout = Robot()
   print(vars(scout))
   print(scout.__battery)

.. output: err_mangled_name.txt all short

.. code-block:: text

   {'_Robot__battery': 100}
   Traceback (most recent call last):
     File ".../mangled_name.py", line 8, in <module>
       print(scout.__battery)
             ^^^^^^^^^^^^^^^
   AttributeError: 'Robot' object has no attribute '__battery'

- The tutorial: "Any identifier of the form ``__spam`` (at least two
  leading underscores, at most one trailing underscore) is textually
  replaced with ``_classname__spam``". This is **name mangling**.
- Outside the class, ``scout.__battery`` is not found.
  ``scout._Robot__battery`` still works: mangling avoids name clashes
  with Lecture 7's subclasses; it does not hide data.
- Use one underscore for non-public names.

A Getter and a Setter Method Check Every Change
-----------------------------------------------

.. code: lecture6/getter_setter_demo.py all

.. code-block:: python
   :caption: lecture6/getter_setter_demo.py

   """A getter and a setter method that checks every change."""


   class Robot:
       """A robot in the competition."""

       def __init__(self, name: str, battery: int = 100):
           self._name = name
           self.set_battery(battery)

       def get_battery(self) -> int:
           return self._battery

       def set_battery(self, value: int) -> None:
           if not 0 <= value <= 100:
               raise ValueError(f"battery must be 0 to 100, got {value}")
           self._battery = value


   scout = Robot("Scout")
   scout.set_battery(scout.get_battery() - 30)
   print(scout.get_battery())

.. output: getter_setter_demo.txt all

.. code-block:: text

   70

Every change, even in ``__init__``, goes through ``set_battery()``,
which checks rule 1. The cost: callers write
``scout.set_battery(scout.get_battery() - 30)`` instead of
``scout.battery -= 30``.

A Property Runs a Method When You Read an Attribute
---------------------------------------------------

.. code: lecture6/robotics_competition/robot.py 20-28

.. code-block:: python
   :caption: From lecture6/robotics_competition/robot.py, the two getters

       @property
       def name(self) -> str:
           """The robot's name. It cannot change (rule 5)."""
           return self._name

       @property
       def battery(self) -> int:
           """The battery level, in percent."""
           return self._battery

- The library reference
  (`property <https://docs.python.org/3.12/library/functions.html#property>`_):
  "The ``@property`` decorator turns the ``voltage()`` method into a
  'getter' for a read-only attribute with the same name". Here: the
  getters of ``name`` and ``battery``; ``battery`` gets a setter in the
  next section.
- ``scout.battery`` has no parentheses, but it calls the method, which
  returns ``self._battery``. The value lives in the non-public
  ``_battery``.
- A line that starts with ``@`` is a decorator. Lecture 7 shows how
  decorators work; here, use ``@property`` as shown.

The Setter Checks Every Assignment, Even in __init__
----------------------------------------------------

.. code: lecture6/robotics_competition/robot.py 15-19,30-36

.. code-block:: python

       def __init__(self, name: str, battery: int = 100):
           self._name = name
           self.battery = battery  # Runs the battery setter
           Robot.total_robots += 1

       @battery.setter
       def battery(self, value: int) -> None:
           if not isinstance(value, int) or isinstance(value, bool):
               raise TypeError(f"battery must be an int, got {value!r}")
           if not 0 <= value <= 100:
               raise ValueError(f"battery must be 0 to 100, got {value}")
           self._battery = value

- ``@battery.setter`` adds a setter to the property ``battery``:
  ``scout.battery = value`` calls it. It stores the value only if both
  checks pass (rule 1). ``True`` is an ``int`` (Lecture 2), so the first
  check also refuses a ``bool``.
- ``__init__`` writes ``self.battery = battery``, not
  ``self._battery``, so a new robot goes through the same checks.
- ``name`` has no setter, so it is read-only (rule 5).

Using the Properties
--------------------

.. code: lecture6/property_demo.py all

.. code-block:: python
   :caption: lecture6/property_demo.py

   """The battery and name properties of the competition's Robot."""

   from robotics_competition.robot import Robot

   scout = Robot("Scout")
   scout.battery = 80  # Runs the setter
   print(scout.battery, scout.name)  # Run the getters
   print(vars(scout))

   for value in [150, "full"]:
       try:
           scout.battery = value
       except (ValueError, TypeError) as err:
           print(type(err).__name__, err)

   try:
       scout.name = "Bob"
   except AttributeError as err:
       print("AttributeError", err)
   print(scout)

.. output: property_demo.txt all

.. code-block:: text

   80 Scout
   {'_name': 'Scout', '_battery': 80}
   ValueError battery must be 0 to 100, got 150
   TypeError battery must be an int, got 'full'
   AttributeError property 'name' of 'Robot' object has no setter
   Scout [80%, active]

- The caller writes plain attribute code: ``scout.battery = 80`` runs
  the setter, and ``scout.battery`` runs the getter.
- ``vars(scout)`` shows the names the object stores, ``_name`` and
  ``_battery``. The properties are names in the class.
- ``except (ValueError, TypeError)`` handles either type.
  ``type(err).__name__`` is the name of the exception's class. After the
  three refused assignments, the robot still has its name and 80%.

A Property Can Compute Its Value: status
----------------------------------------

.. code: lecture6/robotics_competition/robot.py 38-51

.. code-block:: python

       @property
       def status(self) -> str:
           """'needs_recharge' below LOW_BATTERY_PCT, else 'active'."""
           if self._battery < Robot.LOW_BATTERY_PCT:
               return "needs_recharge"
           return "active"

       def perform_task(self, task: str, minutes: int) -> bool:
           """Do the task if active and the battery covers it; True if done."""
           drain = minutes * Robot.DRAIN_PCT_PER_MIN
           if self.status != "active" or drain > self._battery:
               return False
           self.battery = self._battery - drain
           return True

- ``status`` is computed from ``_battery`` at each read (rule 3, the
  ``/`` in the UML box), so it can never disagree with the battery.
- ``LOW_BATTERY_PCT = 10`` and ``DRAIN_PCT_PER_MIN = 2`` are class
  attributes of ``Robot`` (rules 3 and 4).
- ``perform_task()`` refuses a task when the status is not
  ``"active"`` or the task needs more than the battery holds. It writes
  ``self.battery = ...``, so the setter checks the result too.

The Robot Class Enforces Rules 3 and 4
--------------------------------------

.. code: lecture6/competition_demo.py all

.. code-block:: python
   :caption: lecture6/competition_demo.py

   """Business rules 3 and 4, enforced by the Robot class."""

   from robotics_competition.robot import Robot

   hauler = Robot("Hauler", 30)  # 30%: chosen for this example
   for minutes in [5, 6, 3]:  # Task lengths, chosen for this example
       done = hauler.perform_task("carry crate", minutes)
       print(f"{minutes} min: done={done}, {hauler}")
   hauler.recharge()
   print(hauler, Robot.total_robots)
   print(hauler.perform_task("long haul", 60), hauler)  # Needs 120%

.. output: competition_demo.txt all

.. code-block:: text

   5 min: done=True, Hauler [20%, active]
   6 min: done=True, Hauler [8%, needs_recharge]
   3 min: done=False, Hauler [8%, needs_recharge]
   Hauler [100%, active] 1
   False Hauler [100%, active]

The arithmetic (rule 4: 2% per minute; the hauler starts at 30%,
chosen for this example):

1. 5 min uses 10%: 30% becomes 20%, still active.
2. 6 min uses 12%: 20% becomes 8%, below 10%: ``"needs_recharge"``.
3. 3 min would use 6%, but the status is ``"needs_recharge"``: refused
   (rule 3). ``recharge()``: 100%. One robot was made: ``total_robots``
   is 1.
4. 60 min would use 120%, more than the battery holds: refused, and the
   battery stays at 100%.

The Whole Robot Class
---------------------

.. dropdown:: lecture6/robotics_competition/robot.py
    :class-container: sd-border-secondary

    .. code: lecture6/robotics_competition/robot.py all

    .. code-block:: python

       """The Robot class of the robotics competition."""


       class Robot:
           """A competition robot with a name, a battery level, and a status.

           The battery level is a whole percent from 0 to 100. Below
           LOW_BATTERY_PCT, the robot needs a recharge and does no tasks.
           """

           LOW_BATTERY_PCT = 10  # Rule 3, chosen for this example
           DRAIN_PCT_PER_MIN = 2  # Rule 4, chosen for this example
           total_robots = 0

           def __init__(self, name: str, battery: int = 100):
               self._name = name
               self.battery = battery  # Runs the battery setter
               Robot.total_robots += 1

           @property
           def name(self) -> str:
               """The robot's name. It cannot change (rule 5)."""
               return self._name

           @property
           def battery(self) -> int:
               """The battery level, in percent."""
               return self._battery

           @battery.setter
           def battery(self, value: int) -> None:
               if not isinstance(value, int) or isinstance(value, bool):
                   raise TypeError(f"battery must be an int, got {value!r}")
               if not 0 <= value <= 100:
                   raise ValueError(f"battery must be 0 to 100, got {value}")
               self._battery = value

           @property
           def status(self) -> str:
               """'needs_recharge' below LOW_BATTERY_PCT, else 'active'."""
               if self._battery < Robot.LOW_BATTERY_PCT:
                   return "needs_recharge"
               return "active"

           def perform_task(self, task: str, minutes: int) -> bool:
               """Do the task if active and the battery covers it; True if done."""
               drain = minutes * Robot.DRAIN_PCT_PER_MIN
               if self.status != "active" or drain > self._battery:
                   return False
               self.battery = self._battery - drain
               return True

           def recharge(self) -> None:
               """Fill the battery to 100%."""
               self.battery = 100

           def __repr__(self) -> str:
               return f"Robot({self._name!r}, battery={self._battery})"

           def __str__(self) -> str:
               return f"{self._name} [{self._battery}%, {self.status}]"

help() Lists the Public Interface
---------------------------------

.. code: lecture6/help_demo.py all

.. code-block:: python
   :caption: lecture6/help_demo.py

   """help() on the competition's Robot: its public interface."""

   from robotics_competition.robot import Robot

   help(Robot)  # In a terminal, help() opens a pager: press q to quit

.. output: help_demo.txt 1-56

.. code-block:: text

   Help on class Robot in module robotics_competition.robot:

   class Robot(builtins.object)
    |  Robot(name: str, battery: int = 100)
    |
    |  A competition robot with a name, a battery level, and a status.
    |
    |  The battery level is a whole percent from 0 to 100. Below
    |  LOW_BATTERY_PCT, the robot needs a recharge and does no tasks.
    |
    |  Methods defined here:
    |
    |  __init__(self, name: str, battery: int = 100)
    |      Initialize self.  See help(type(self)) for accurate signature.
    |
    |  __repr__(self) -> str
    |      Return repr(self).
    |
    |  __str__(self) -> str
    |      Return str(self).
    |
    |  perform_task(self, task: str, minutes: int) -> bool
    |      Do the task if active and the battery covers it; True if done.
    |
    |  recharge(self) -> None
    |      Fill the battery to 100%.
    |
    |  ----------------------------------------------------------------------
    |  Readonly properties defined here:
    |
    |  name
    |      The robot's name. It cannot change (rule 5).
    |
    |  status
    |      'needs_recharge' below LOW_BATTERY_PCT, else 'active'.
    |
    |  ----------------------------------------------------------------------
    |  Data descriptors defined here:
    |
    |  __dict__
    |      dictionary for instance variables
    |
    |  __weakref__
    |      list of weak references to the object
    |
    |  battery
    |      The battery level, in percent.
    |
    |  ----------------------------------------------------------------------
    |  Data and other attributes defined here:
    |
    |  DRAIN_PCT_PER_MIN = 2
    |
    |  LOW_BATTERY_PCT = 10
    |
    |  total_robots = 0


- ``help()`` prints the methods and properties with their docstrings:
  what a caller needs, without the code (abstraction). In a terminal, it
  opens a pager, a viewer that shows one screen at a time: press ``q`` to
  quit.
- ``name`` and ``status`` are read-only properties; ``battery`` also has
  a setter, so ``help()`` lists it with the data descriptors.

A ROS 2 Message Is an Object, and Its Fields Are Properties
-----------------------------------------------------------

This program needs ROS 2 Jazzy (Lecture 8). It makes a message object
only: no node, nothing sent. It was run with ROS 2 sourced:

.. code-block:: bash

   source /opt/ros/jazzy/setup.bash
   uv run lecture6/ros_point_check.py

.. code: lecture6/ros_point_check.py all

.. code-block:: python
   :caption: lecture6/ros_point_check.py

   """A ROS 2 message is an object: repr, ==, and its x property.

   Needs ROS 2 Jazzy sourced (Lecture 8). Makes a message object only:
   no node, nothing sent.
   """

   from geometry_msgs.msg import Point

   p = Point(x=1.0, y=2.0)
   print(repr(p))
   print(p == Point(x=1.0, y=2.0, z=0.0))
   print(type(vars(Point)["x"]).__name__, Point.__slots__[:3])
   p.x = "far"  # Checked only when ROS_PYTHON_CHECK_FIELDS=1
   print(repr(p))

.. output: ros_point_check.txt all

.. code-block:: text

   geometry_msgs.msg.Point(x=1.0, y=2.0, z=0.0)
   True
   property ['_x', '_y', '_z']
   geometry_msgs.msg.Point(x='far', y=2.0, z=0.0)

- The class ``Point`` has a ``__repr__`` that looks like a call, an
  ``__eq__``, and a property per field, stored in ``_x``, ``_y``,
  ``_z``.
- Its setter checks the type only when ``ROS_PYTHON_CHECK_FIELDS=1``.
  Run that way (``ROS_PYTHON_CHECK_FIELDS=1 uv run
  lecture6/ros_point_check.py``), the assignment fails:

.. output: ros_point_check_checked.txt 4-11 short

.. code-block:: text

   Traceback (most recent call last):
     File ".../ros_point_check.py", line 13, in <module>
       p.x = "far"  # Checked only when ROS_PYTHON_CHECK_FIELDS=1
       ^^^
     File "/opt/ros/jazzy/lib/python3.12/site-packages/geometry_msgs/msg/_point.py", line 152, in x
       isinstance(value, float), \
       ^^^^^^^^^^^^^^^^^^^^^^^^
   AssertionError: The 'x' field must be of type 'float'

The last ``File`` line is in ROS 2's generated ``_point.py``, the
setter of ``x``. Its check is an ``assert``, so the error is an
``AssertionError``.

Exercise 3 on the :doc:`exercises page <l6_exercises>` practices this
section; Exercise 4 is for after class.


Next Class
====================================================

:doc:`L7: Object-Oriented Programming, Part II </lectures/lecture7/l7_index>`
(Tuesday, March 23). There is no class on March 16 (Spring Break).

- Decorators
- Class methods and static methods
- Object relationships
- Inheritance
- Polymorphism
- Abstract base classes
- Data classes
- ``__slots__``
- Protocols
