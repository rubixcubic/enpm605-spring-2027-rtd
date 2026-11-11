====================================================
Lecture
====================================================

Every program on this page is in the ``lecture7/`` folder of the
`course repository <https://github.com/rubixcubic/enpm605-spring-2027-python>`_.
Run each one from the root of the repository, for example:

.. code-block:: bash

   uv run lecture7/decorator_demo.py

Every output on this page was printed by running the program in the
course environment (``.venv``, what ``uv run`` uses): Python 3.12.3 on
Ubuntu 24.04. In paths, ``...`` marks a path shortened to fit. The
values in the examples (battery levels, task lengths, sensor ranges,
speeds) are chosen for the examples.

This lecture continues Lecture 6's robotics competition. Its classes are
in the package ``lecture7/robotics_competition/``: ``robot.py`` (Lecture
6's ``Robot``, with four changes), ``sensor.py``, ``task.py``,
``team.py``, ``scout.py``, and ``hauler.py``. The whole files are at the
end of this page.


Decorators
====================================================

@property, Written Without the @ Line
-------------------------------------

.. code: lecture7/property_by_hand.py all

.. code-block:: python
   :caption: lecture7/property_by_hand.py

   """What @property does, written without the @ line."""


   class Robot:
       def __init__(self, name: str):
           self._name = name

       def name(self) -> str:
           """The robot's name. It cannot change."""
           return self._name

       name = property(name)  # What @property above def name would do


   scout = Robot("Scout")
   print(scout.name, type(vars(Robot)["name"]).__name__)
   print(Robot.name.__doc__)

.. output: property_by_hand.txt all

.. code-block:: text

   Scout property
   The robot's name. It cannot change.

- ``property(name)`` takes the function ``name`` and returns a
  ``property`` object; the assignment binds ``name`` in the class to it.
  ``scout.name`` then runs the function.
- Lecture 6's ``@property`` line above ``def name`` does the same
  assignment. A function used this way is a **decorator**.

A Decorator Takes a Function and Returns a New One
--------------------------------------------------

.. code: lecture7/decorator_demo.py all

.. code-block:: python
   :caption: lecture7/decorator_demo.py

   """A decorator: a function that takes a function and returns one."""


   def log_calls(func):
       print(f"Decorating {func.__name__}")  # Runs once, when applied

       def wrapper(distance_m, speed_mps):
           print(f"Calling {func.__name__}{(distance_m, speed_mps)}")
           result = func(distance_m, speed_mps)
           print(f"{func.__name__} returned {result}")
           return result

       return wrapper


   def drive_time(distance_m, speed_mps):
       """Seconds to drive distance_m at speed_mps (Lecture 4)."""
       return distance_m / speed_mps


   drive_time = log_calls(drive_time)  # The name now refers to wrapper
   print(drive_time(12.0, 1.5))  # Values chosen for this example

.. output: decorator_demo.txt all

.. code-block:: text

   Decorating drive_time
   Calling drive_time(12.0, 1.5)
   drive_time returned 8.0
   8.0

- The glossary: a decorator is "A function returning another function,
  usually applied as a function transformation using the ``@wrapper``
  syntax".
- ``log_calls()`` defines a new function, ``wrapper``, and returns it.
  ``wrapper`` prints, calls the original function ``func``, prints its
  result, and returns it. ``wrapper`` uses ``func``, a name of the
  enclosing call: it is a closure (Lecture 4), so it keeps ``func`` after
  ``log_calls()`` returns.

What the last two lines do:

1. ``log_calls(drive_time)`` runs once: it prints ``Decorating
   drive_time`` and returns ``wrapper``.
2. The assignment binds the name ``drive_time`` to ``wrapper``.
3. ``drive_time(12.0, 1.5)`` calls ``wrapper``, which calls the original
   function: 12.0 m at 1.5 m/s is 8.0 s.

.. figure:: /_static/images/l7/decorator_rebind.svg
   :alt: Tag 1: a gray box, the module's namespace, holds the name drive_time. It has an arrow to a red box, a function object named wrapper. Tag 2: the wrapper's __closure__ has an arrow down to a cell object for the name func. The cell has an arrow to tag 3: a function object named drive_time, the function that def drive_time made.
   :width: 70%

   The names after ``drive_time = log_calls(drive_time)``. 1: the
   module's name ``drive_time``. 2: the ``wrapper`` function, and its
   closure cell for ``func``. 3: the function that ``def drive_time``
   made.

Before the assignment, ``drive_time`` referred to 3. Now it refers to 2,
and only the cell of 2 still refers to 3. A caller writes
``drive_time(...)`` as before and gets the logging, with no change to the
code of 3.

The @ Line Decorates the Function When def Runs
-----------------------------------------------

.. code: lecture7/at_syntax_demo.py 16-24

.. code-block:: python
   :caption: From lecture7/at_syntax_demo.py (log_calls as in decorator_demo.py)

   @log_calls  # The same as drive_time = log_calls(drive_time)
   def drive_time(distance_m, speed_mps):
       """Seconds to drive distance_m at speed_mps (Lecture 4)."""
       return distance_m / speed_mps


   print("Before the first call")
   print(drive_time(9.0, 1.5))  # Values chosen for this example
   print(drive_time.__name__, drive_time.__doc__)

.. output: at_syntax_demo.txt all

.. code-block:: text

   Decorating drive_time
   Before the first call
   Calling drive_time(9.0, 1.5)
   drive_time returned 6.0
   6.0
   wrapper None

- The glossary: "The decorator syntax is merely syntactic sugar". The
  ``@`` line runs ``drive_time = log_calls(drive_time)`` right after the
  ``def``: once, before any call. ``Decorating drive_time`` prints before
  ``Before the first call``.
- ``wrapper None``: the name refers to ``wrapper``, which has no
  docstring.

A Wrapper with Fixed Parameters Fits One Kind of Function
---------------------------------------------------------

.. code: lecture7/errors/wrapper_two_args.py all

.. code-block:: python
   :caption: lecture7/errors/wrapper_two_args.py

   def log_calls(func):
       def wrapper(distance_m, speed_mps):  # Fits drive_time() only
           print(f"Calling {func.__name__}")
           return func(distance_m, speed_mps)

       return wrapper


   @log_calls
   def recharge_time(battery):
       """Minutes to fill the battery at 5% per minute."""
       return (100 - battery) / 5


   print(recharge_time(40))

.. output: err_wrapper_two_args.txt all short

.. code-block:: text

   Traceback (most recent call last):
     File ".../wrapper_two_args.py", line 15, in <module>
       print(recharge_time(40))
             ^^^^^^^^^^^^^^^^^
   TypeError: log_calls.<locals>.wrapper() missing 1 required positional argument: 'speed_mps'

``recharge_time(40)`` calls ``wrapper(40)``, and ``wrapper`` needs two
arguments. The error names ``wrapper``, not ``recharge_time``: after
decorating, the name refers to the wrapper.

\*args, \*\*kwargs, and functools.wraps
---------------------------------------

.. code: lecture7/wraps_demo.py all

.. code-block:: python
   :caption: lecture7/wraps_demo.py

   """A wrapper with *args and **kwargs, and functools.wraps."""

   import functools


   def log_calls(func):
       @functools.wraps(func)  # Copies __name__, __doc__, ... from func
       def wrapper(*args, **kwargs):
           print(f"Calling {func.__name__} with {args} {kwargs}")
           return func(*args, **kwargs)

       return wrapper


   @log_calls
   def drive_time(distance_m, speed_mps):
       """Seconds to drive distance_m at speed_mps (Lecture 4)."""
       return distance_m / speed_mps


   @log_calls
   def recharge_time(battery, pct_per_min=5):
       """Minutes to fill the battery; 5% per minute is an example rate."""
       return (100 - battery) / pct_per_min


   print(drive_time(12.0, speed_mps=1.5))
   print(recharge_time(40))
   print(drive_time.__name__, "|", drive_time.__doc__)

.. output: wraps_demo.txt all

.. code-block:: text

   Calling drive_time with (12.0,) {'speed_mps': 1.5}
   8.0
   Calling recharge_time with (40,) {}
   12.0
   drive_time | Seconds to drive distance_m at speed_mps (Lecture 4).

- ``*args`` collects the positional arguments in a tuple and
  ``**kwargs`` the keyword arguments in a dictionary (Lecture 4).
  ``func(*args, **kwargs)`` passes them all on, so one wrapper fits any
  function: two arguments for ``drive_time()``, one for
  ``recharge_time()``. (100 − 40) / 5 = 12.0 minutes, at the example rate
  of 5% per minute.
- ``@functools.wraps(func)`` is itself a decorator. It copies the name,
  the docstring, and other attributes of ``func`` to ``wrapper``: the
  last line shows the original name and docstring.
- The library, for ``update_wrapper()``, which ``wraps`` calls: "If the
  wrapper function is not updated, the metadata of the returned function
  will reflect the wrapper definition rather than the original function
  definition".

A Decorator That Takes an Argument: retry(times)
------------------------------------------------

.. code: lecture7/retry_demo.py all

.. code-block:: python
   :caption: lecture7/retry_demo.py

   """A decorator with an argument: retry(times) makes the decorator."""

   import functools


   def retry(times):
       """Make a decorator that tries the function up to times times."""

       def decorator(func):
           @functools.wraps(func)
           def wrapper(*args, **kwargs):
               for attempt in range(1, times + 1):
                   try:
                       return func(*args, **kwargs)
                   except TimeoutError as err:
                       print(f"Attempt {attempt} failed: {err}")
               raise TimeoutError(f"{func.__name__} failed {times} times")

           return wrapper

       return decorator


   replies = iter([None, None, 2.5])  # Two timeouts, then 2.5 m (chosen)


   @retry(times=3)
   def read_lidar() -> float:
       """Read one range from the lidar, in meters."""
       reply = next(replies)
       if reply is None:
           raise TimeoutError("no reply from the lidar")
       return reply


   print(read_lidar())

.. output: retry_demo.txt all

.. code-block:: text

   Attempt 1 failed: no reply from the lidar
   Attempt 2 failed: no reply from the lidar
   2.5

- ``retry(times)`` is not a decorator: it makes one. ``decorator`` and
  ``wrapper`` are closures that keep ``times``.
- ``wrapper`` calls the function up to ``times`` times, and returns the
  first result that comes back without a ``TimeoutError``.

What ``@retry(times=3)`` does:

1. ``retry(times=3)`` runs first and returns ``decorator``.
2. ``decorator(read_lidar)`` returns ``wrapper``, and the name
   ``read_lidar`` is bound to it.
3. ``read_lidar()`` calls ``wrapper``: two timeouts, then 2.5 m on the
   third attempt.

Decorators in This Course
-------------------------

.. list-table::
   :header-rows: 1
   :widths: 34 44 22
   :class: compact-table

   * - Decorator
     - What it makes
     - Taught in
   * - ``@property``, ``@name.setter``
     - A property: a method run on reading or assigning
     - Lecture 6
   * - ``@functools.wraps(func)``
     - A wrapper with the name and docstring of ``func``
     - This section
   * - ``@classmethod``, ``@staticmethod``
     - A method that receives the class, or nothing
     - `Class Methods and Static Methods`_
   * - ``@dataclass``
     - A class with ``__init__``, ``__repr__``, and ``__eq__`` written
       for it
     - `Data Classes`_
   * - ``@abstractmethod``
     - A method that every subclass you make objects of must define
     - `Abstract Base Classes`_
   * - ``@runtime_checkable``
     - A protocol that ``isinstance()`` can check
     - `Protocols`_

Two ``@`` lines: the one closest to ``def`` applies first. ``@a`` above
``@b`` above ``def f`` is ``f = a(b(f))``.

Exercise 1 on the :doc:`exercises page <l7_exercises>` practices this
section.


Class Methods and Static Methods
====================================================

@classmethod Passes the Class as cls
------------------------------------

.. code: lecture7/robotics_competition/robot.py 29-38

.. code-block:: python
   :caption: From lecture7/robotics_competition/robot.py, in class Robot

       @classmethod
       def from_dict(cls, config: dict) -> "Robot":
           """Make a robot from a dictionary, such as one read from a file."""
           return cls(config["name"], config.get("battery", 100))

       @staticmethod
       def battery_bar(level: int) -> str:
           """Draw a battery level as ten characters: [######----]."""
           bars = level // 10
           return "[" + "#" * bars + "-" * (10 - bars) + "]"

.. code: lecture7/class_methods_demo.py all

.. code-block:: python
   :caption: lecture7/class_methods_demo.py

   """A class method makes a robot; a static method draws a battery."""

   from robotics_competition.robot import Robot

   config = {"name": "Scout", "battery": 80}  # As read from a file
   scout = Robot.from_dict(config)  # cls is Robot
   print(repr(scout), Robot.total_robots)
   print(Robot.battery_bar(scout.battery), scout.battery_bar(35))

.. output: class_methods_demo.txt all

.. code-block:: text

   Robot('Scout', battery=80) 1
   [########--] [###-------]

- The library: "A class method receives the class as an implicit first
  argument, just like an instance method receives the instance."
  ``Robot.from_dict(config)`` runs with ``cls`` bound to ``Robot``, so
  ``cls(...)`` is ``Robot(...)``. A class method that makes objects is a
  second way to construct one.
- The library: "A static method does not receive an implicit first
  argument." ``battery_bar()`` is a plain function kept in the class,
  because it belongs with robots. It can be called on the class or on an
  object: ``scout.battery_bar(35)`` passes only 35. 80 // 10 = 8 and
  35 // 10 = 3 marks.

Three Kinds of Methods
----------------------

.. list-table::
   :header-rows: 1
   :widths: 18 18 22 42
   :class: compact-table

   * - Kind
     - Decorator
     - First parameter
     - Use it for
   * - Instance method
     - none
     - ``self``, the object
     - Work on one object's data: ``perform_task()``
   * - Class method
     - ``@classmethod``
     - ``cls``, the class
     - Making objects, or class data: ``from_dict()``
   * - Static method
     - ``@staticmethod``
     - none
     - A helper that needs neither: ``battery_bar()``

- The library: "If a class method is called for a derived class, the
  derived class object is passed as the implied first argument." The
  `Inheritance`_ section uses this.
- ROS 2 uses class methods to make objects: ``Time.from_msg(msg)`` in
  ``rclpy.time`` is one (checked in ROS 2 Jazzy).


Data Classes
====================================================

@dataclass Writes __init__, __repr__, and __eq__
------------------------------------------------

.. code: lecture7/dataclass_demo.py all

.. code-block:: python
   :caption: lecture7/dataclass_demo.py

   """@dataclass writes __init__, __repr__, and __eq__ from the fields."""

   import inspect
   from dataclasses import dataclass


   @dataclass
   class Task:
       """A competition task; rule 2 is not checked yet."""

       name: str
       priority: str
       minutes: int


   pick = Task("pick widget", "high", 5)  # Values chosen for this example
   print(pick)
   print(pick == Task("pick widget", "high", 5), pick.minutes)
   print(inspect.signature(Task))
   pick.minutes = 7  # Fields are plain attributes
   print(pick, Task.__hash__)

.. output: dataclass_demo.txt all

.. code-block:: text

   Task(name='pick widget', priority='high', minutes=5)
   True 5
   (name: str, priority: str, minutes: int) -> None
   Task(name='pick widget', priority='high', minutes=7) None

- The library: "A field is defined as a class variable that has a type
  annotation." From them, ``@dataclass`` writes ``__init__`` with one
  parameter per field (output line 3), ``__repr__``, and an ``__eq__``
  that compares the fields. Lecture 6's Exercise 4 wrote these three
  methods for ``Task`` by hand.
- Fields are ordinary instance attributes: nothing stops
  ``pick.minutes = 7``, and nothing checks rule 2 yet.
- The library: "If ``eq`` is true and ``frozen`` is false, ``__hash__()``
  will be set to ``None``, marking it unhashable (which it is, since it
  is mutable)." Lecture 6's ``Sensor`` with ``__eq__`` was unhashable the
  same way.

A Field Needs an Annotation, and a List Default Is Refused
----------------------------------------------------------

.. code: lecture7/errors/mutable_default.py all

.. code-block:: python
   :caption: lecture7/errors/mutable_default.py

   from dataclasses import dataclass


   @dataclass
   class Route:
       name: str
       waypoints: list[str] = []  # One list for every route: refused

The class is never made. The traceback's last line:

.. output: err_mutable_default.txt 16

.. code-block:: text

   ValueError: mutable default <class 'list'> for field waypoints is not allowed: use default_factory

- One list would be shared by every route, the trap of Lecture 4's
  mutable default and Lecture 6's class attribute list. Ruff reports it
  before the program runs:

.. output: ruff_mutable_default.txt 1

.. code-block:: text

   RUF008 Do not use mutable default values for dataclass attributes

- A name without an annotation, such as ``points = 0``, is a class
  attribute, not a field: ``__init__`` has no parameter for it.

field(default_factory=list) Makes a New List for Each Object
------------------------------------------------------------

.. code: lecture7/field_demo.py all

.. code-block:: python
   :caption: lecture7/field_demo.py

   """field(default_factory=list) makes a new list for each object."""

   from dataclasses import dataclass, field


   @dataclass
   class Route:
       name: str
       waypoints: list[str] = field(default_factory=list)


   patrol = Route("patrol")
   dock = Route("dock")
   patrol.waypoints.append("A")
   print(patrol, dock)

.. output: field_demo.txt all

.. code-block:: text

   Route(name='patrol', waypoints=['A']) Route(name='dock', waypoints=[])

``default_factory`` is a function with no parameters. The generated
``__init__`` calls it for each new object that gets no ``waypoints``:
``list()`` makes an empty list. ``patrol`` got ``'A'``; ``dock`` still
has its own empty list.

The Competition's Task: __post_init__ Checks Rule 2
---------------------------------------------------

Rule 8, chosen for this example and used by ``Team`` in `Object
Relationships`_: a team scores points for each task one of its robots
does: 1 for ``"low"``, 2 for ``"medium"``, 3 for ``"high"``, 5 for
``"critical"``.

.. code: lecture7/robotics_competition/task.py all

.. code-block:: python
   :caption: lecture7/robotics_competition/task.py

   """The Task class of the robotics competition, as a data class."""

   from dataclasses import dataclass

   POINTS = {"low": 1, "medium": 2, "high": 3, "critical": 5}  # Rules 2, 8


   @dataclass(frozen=True)
   class Task:
       """A task: a name, a priority (rule 2), and a length in minutes."""

       name: str
       priority: str
       minutes: int

       def __post_init__(self) -> None:
           if self.priority not in POINTS:
               raise ValueError(f"priority must be one of {list(POINTS)}")
           m = self.minutes
           if not isinstance(m, int) or isinstance(m, bool):
               raise TypeError(f"minutes must be an int, got {m!r}")
           if m <= 0:
               raise ValueError(f"minutes must be positive, got {m}")

       @property
       def points(self) -> int:
           """The points a team scores when one of its robots does it."""
           return POINTS[self.priority]

- The library: ``__post_init__`` "will be called by the generated
  ``__init__()``", after the fields are set. Raising there refuses the
  object.
- ``POINTS`` holds the four priorities of rule 2 as its keys, and the
  points of rule 8 as its values. ``points`` is a property, as in Lecture
  6: a task's points are read from ``POINTS`` at each use.
- ``frozen=True`` makes every assignment to a field raise an error.

.. code: lecture7/task_demo.py all

.. code-block:: python
   :caption: lecture7/task_demo.py

   """The competition's Task: checked by __post_init__, and frozen."""

   from robotics_competition.task import Task

   pick = Task("pick widget", "high", 5)  # Values chosen for this example
   print(pick, pick.points)
   for args in [("dance", "urgent", 1), ("lift", "low", 2.5)]:
       try:
           Task(*args)
       except (ValueError, TypeError) as err:
           print(type(err).__name__, err)
   try:
       pick.minutes = 7
   except AttributeError as err:
       print(type(err).__name__, err)
   done = {pick, Task("pick widget", "high", 5)}  # A set needs __hash__
   print(len(done), hash(pick) == hash(Task("pick widget", "high", 5)))

.. output: task_demo.txt all

.. code-block:: text

   Task(name='pick widget', priority='high', minutes=5) 3
   ValueError priority must be one of ['low', 'medium', 'high', 'critical']
   TypeError minutes must be an int, got 2.5
   FrozenInstanceError cannot assign to field 'minutes'
   1 True

- ``pick.points`` is 3: ``"high"`` is worth 3 points (rule 8). ``Task(*args)``
  unpacks each tuple into the three fields (Lecture 4). ``"urgent"`` is
  not a priority; 2.5 is not an ``int``.
- ``FrozenInstanceError`` is a kind of ``AttributeError``, so ``except
  AttributeError`` handles it.
- The library: "If ``eq`` and ``frozen`` are both true, by default
  ``@dataclass`` will generate a ``__hash__()`` method for you." Two
  equal tasks make a set of 1.


__slots__
====================================================

__slots__ Lists the Only Attribute Names an Object Can Have
-----------------------------------------------------------

.. code: lecture7/slots_demo.py all

.. code-block:: python
   :caption: lecture7/slots_demo.py

   """__slots__ lists the only attribute names an object can have."""


   class Pose:
       __slots__ = ("theta", "x", "y")

       def __init__(self, x: float, y: float, theta: float = 0.0):
           self.x = x  # Meters
           self.y = y
           self.theta = theta  # Radians


   p = Pose(1.0, 2.0)  # Values chosen for this example
   p.x = 1.5  # A listed name: allowed
   try:
       p.thetta = 0.5  # Misspelled
   except AttributeError as err:
       print("AttributeError", err)
   print(p.x, p.theta, hasattr(p, "__dict__"))

.. output: slots_demo.txt all

.. code-block:: text

   AttributeError 'Pose' object has no attribute 'thetta'
   1.5 0.0 False

- The reference: "Attempts to assign to an unlisted variable name raises
  ``AttributeError``." Lecture 6's misspelled ``batery`` made a new
  attribute; here ``thetta`` is refused.
- The object has no ``__dict__``, so ``vars(p)`` raises ``TypeError``.

The Memory of 100,000 Poses, Measured
-------------------------------------

.. code: lecture7/slots_memory.py all

.. code-block:: python
   :caption: lecture7/slots_memory.py

   """Memory used by 100,000 poses, with and without __slots__."""

   import tracemalloc
   from dataclasses import dataclass


   class Pose:
       def __init__(self, x: float, y: float, theta: float):
           self.x, self.y, self.theta = x, y, theta


   class SlotPose:
       __slots__ = ("theta", "x", "y")

       def __init__(self, x: float, y: float, theta: float):
           self.x, self.y, self.theta = x, y, theta


   @dataclass(slots=True)
   class DataPose:
       x: float
       y: float
       theta: float


   def bytes_per_pose(cls, ask_dict=False, n=100_000):
       """Bytes allocated per object (list included) to make n of them."""
       tracemalloc.start()
       poses = [cls(1.0, 2.0, 0.5) for _ in range(n)]
       if ask_dict:
           for pose in poses:
               vars(pose)  # Asks each object for its __dict__
       used, _ = tracemalloc.get_traced_memory()
       tracemalloc.stop()
       del poses
       return used / n


   runs = [(Pose, False), (Pose, True), (SlotPose, False)]
   runs.append((DataPose, False))
   for cls, ask_dict in runs:
       # The smallest of 5 runs
       best = min(bytes_per_pose(cls, ask_dict) for _ in range(5))
       label = cls.__name__ + (" + vars()" if ask_dict else "")
       print(f"{label:16} {best:6.1f} bytes per pose")
   print(DataPose.__slots__)

.. output: slots_memory.txt all

.. code-block:: text

   Pose               96.0 bytes per pose
   Pose + vars()     160.0 bytes per pose
   SlotPose           64.0 bytes per pose
   DataPose           64.0 bytes per pose
   ('x', 'y', 'theta')

The program was run three times, with the same numbers each time.

- ``tracemalloc`` counts the bytes Python allocates between ``start()``
  and ``get_traced_memory()``. The count includes the list's 8-byte
  reference to each pose.
- With slots, a pose takes 64 bytes instead of 96: 32 fewer, a third
  less. ``@dataclass(slots=True)`` writes ``__slots__`` for you: also 64.
- A plain pose grows to 160 bytes once ``vars()`` asks for its
  ``__dict__``. `What's New in Python 3.11
  <https://docs.python.org/3.12/whatsnew/3.11.html#whatsnew311-faster-cpython-misc>`_:
  "Objects now require less memory due to lazily created object
  namespaces."

When to use slots:

- Many small objects of one class: poses, waypoints, readings. ROS 2
  message classes define ``__slots__`` (Lecture 6:
  ``Point.__slots__``).
- A subclass without its own ``__slots__`` gets a ``__dict__`` back, and
  with it new attribute names.

Exercise 2 on the :doc:`exercises page <l7_exercises>` practices the
last two sections.


Object Relationships
====================================================

Rules 7 and 8, and the Classes That Hold Them
---------------------------------------------

Business rules for teams and scores, chosen for this example, after
Lecture 6's rules 1 to 6:

7. A team has at most three robots, and a robot is added to a team
   once.
8. As in `The Competition's Task: __post_init__ Checks Rule 2`_: a team
   scores 1, 2, 3, or 5 points for each ``"low"``, ``"medium"``,
   ``"high"``, or ``"critical"`` task one of its robots does.

.. list-table:: The package lecture7/robotics_competition/
   :header-rows: 1
   :widths: 30 44 26
   :class: compact-table

   * - Module
     - Holds
     - Written in
   * - ``robot.py``
     - ``Robot``, now with sensors
     - Lecture 6, then here
   * - ``sensor.py``, ``task.py``
     - ``Sensor``, ``Task``: frozen data classes
     - This lecture
   * - ``team.py``
     - ``Team``: rules 7 and 8
     - This lecture
   * - ``scout.py``, ``hauler.py``
     - Two kinds of robot
     - `Inheritance`_
   * - ``arena.py``
     - ``Arena``
     - Exercise 4, by you

Three Relationships in robotics_competition
-------------------------------------------

.. figure:: /_static/images/l7/relationships_uml.svg
   :alt: A UML class diagram with four class boxes: Team, Robot and Sensor in a row, and Task below Team. Tag 1: a line from Team to Robot with a hollow diamond at Team and 0..3 at Robot. Tag 2: a line from Robot to Sensor with a filled diamond at Robot and 0..* at Sensor. Tag 3: a line from Team down to Task with an open arrowhead at Task and 0..* at Task.
   :width: 85%

   1: aggregation, a hollow diamond at the whole. 2: composition, a
   filled diamond at the whole. 3: association, an open arrow to the
   class that is known. A number at a line's end says how many objects:
   ``0..3`` is 0 to 3, ``0..*`` is any number.

.. list-table::
   :header-rows: 1
   :widths: 16 32 22 30
   :class: compact-table

   * - Relationship
     - Who makes the other object
     - Without the first
     - In the competition
   * - Composition
     - The whole, in its own methods; by design, no one else keeps the
       part
     - The part goes with it, if no other name refers to it
     - A robot and its sensors
   * - Aggregation
     - Someone else; the whole receives it as a part
     - The part stays
     - A team and its robots
   * - Association
     - Someone else; the object is known, not a part
     - The object stays
     - A team and its tasks done

All three keep a reference in an attribute. The difference is who made
the object and what it is to the class: its part, or something it knows.
Python keeps an object while any name refers to it: after
``kept = scout.sensors[0]`` and ``del scout``, the sensor is still
there.
In Lecture 8, a ROS 2 node makes its own publishers and timers
(composition).

Composition: A Robot Makes Its Sensors
--------------------------------------

.. code: lecture7/robotics_competition/robot.py 65-72

.. code-block:: python
   :caption: From lecture7/robotics_competition/robot.py, in class Robot

       @property
       def sensors(self) -> tuple[Sensor, ...]:
           """The robot's sensors, as a tuple: a caller cannot add one."""
           return tuple(self._sensors)

       def add_sensor(self, kind: str, range_m: float) -> None:
           """Make a sensor and mount it on this robot (composition)."""
           self._sensors.append(Sensor(kind, range_m))

.. code: lecture7/robotics_competition/sensor.py all

.. code-block:: python
   :caption: lecture7/robotics_competition/sensor.py

   """The Sensor class of the robotics competition, as a data class."""

   from dataclasses import dataclass


   @dataclass(frozen=True)
   class Sensor:
       """A range sensor: its kind and its range in meters (rule 6)."""

       kind: str
       range_m: float

       def __post_init__(self) -> None:
           r = self.range_m
           if not isinstance(r, (int, float)) or isinstance(r, bool):
               raise TypeError(f"range_m must be a number, got {r!r}")
           if r <= 0:
               raise ValueError(f"range_m must be positive, got {r}")

.. code: lecture7/composition_demo.py all

.. code-block:: python
   :caption: lecture7/composition_demo.py

   """Composition: a robot makes its own sensors and keeps them."""

   from robotics_competition.robot import Robot

   scout = Robot("Scout")
   scout.add_sensor("lidar", 12.0)  # Ranges in meters, chosen
   scout.add_sensor("sonar", 4.0)
   print(scout.sensors)
   print(max(sensor.range_m for sensor in scout.sensors))
   try:
       scout.add_sensor("camera", -1.0)
   except ValueError as err:
       print("ValueError", err)
   print(len(scout.sensors))

.. output: composition_demo.txt all

.. code-block:: text

   (Sensor(kind='lidar', range_m=12.0), Sensor(kind='sonar', range_m=4.0))
   12.0
   ValueError range_m must be positive, got -1.0
   2

- The caller passes a kind and a range; ``add_sensor()`` makes the
  ``Sensor``, and ``__init__`` starts the list,
  ``self._sensors = []``.
- ``sensors`` returns a tuple: a caller can read the sensors, but cannot
  add one to the robot's list. The ``Sensor`` checks rule 6 in its
  ``__post_init__``: a sensor's range is a positive number of meters.

Aggregation: A Team Holds Robots Made Elsewhere
-----------------------------------------------

.. code: lecture7/robotics_competition/team.py 7-32

.. code-block:: python
   :caption: From lecture7/robotics_competition/team.py

   class Team:
       """A team of robots made elsewhere, and the tasks they did."""

       MAX_ROBOTS = 3  # Rule 7, chosen for this example

       def __init__(self, name: str):
           self._name = name
           self._robots: list[Robot] = []  # Aggregation: robots are added
           self._done: list[Task] = []  # Association: tasks done

       @property
       def name(self) -> str:
           return self._name

       @property
       def score(self) -> int:
           """The points of the tasks done (rule 8), added at each read."""
           return sum(task.points for task in self._done)

       def add_robot(self, robot: Robot) -> None:
           """Add a robot made elsewhere, once; at most MAX_ROBOTS."""
           if robot in self._robots:
               raise ValueError(f"{robot.name} is already in {self._name}")
           if len(self._robots) == Team.MAX_ROBOTS:  # Rule 7
               raise ValueError(f"{self._name} is full")
           self._robots.append(robot)

``add_robot()`` receives a robot that the caller made. Rule 7:
``robot in self._robots`` uses ``==``, and ``Robot`` has no ``__eq__``,
so it asks whether this very object is already there (Lecture 6).

.. code: lecture7/team_demo.py all

.. code-block:: python
   :caption: lecture7/team_demo.py

   """Aggregation: a team holds robots that were made outside it."""

   from robotics_competition.robot import Robot
   from robotics_competition.team import Team

   scout = Robot("Scout")
   hauler = Robot("Hauler", 60)  # 60%: chosen for this example
   alpha = Team("Alpha")
   alpha.add_robot(scout)
   alpha.add_robot(hauler)
   print(alpha, len(alpha))
   for robot in [scout, Robot("Carrier"), Robot("Lifter")]:
       try:
           alpha.add_robot(robot)
       except ValueError as err:
           print("ValueError", err)
   del alpha  # The team is gone; the robots are not
   print(scout, hauler)

.. output: team_demo.txt all

.. code-block:: text

   Team('Alpha', ['Scout', 'Hauler'], score=0) 2
   ValueError Scout is already in Alpha
   ValueError Alpha is full
   Scout [100%, active] Hauler [60%, active]

- ``Carrier`` joins as the third robot without a message; ``Lifter``
  would be the fourth (rule 7).
- ``del alpha`` removes the name ``alpha``. The robots still have their
  own names, ``scout`` and ``hauler``, so they stay.

Association: A Team Keeps the Tasks It Did
------------------------------------------

.. code: lecture7/robotics_competition/team.py 21-24,34-40

.. code-block:: python
   :caption: From lecture7/robotics_competition/team.py, in class Team

       @property
       def score(self) -> int:
           """The points of the tasks done (rule 8), added at each read."""
           return sum(task.points for task in self._done)
       def assign(self, task: Task) -> bool:
           """Give the task to the first robot that does it; True if done."""
           for robot in self._robots:
               if robot.perform_task(task.name, task.minutes):
                   self._done.append(task)
                   return True
           return False

- ``assign()`` offers the task to each robot in order. The first that
  does it ends the loop, and the team keeps the task in
  ``self._done``.
- The tasks are made outside the team and are not parts of it.
  ``score`` adds their points at each read (rule 8), like Lecture 6's
  ``status``.

.. code: lecture7/assign_demo.py all

.. code-block:: python
   :caption: lecture7/assign_demo.py

   """Association: a team uses each Task it is given, and scores it."""

   from robotics_competition.robot import Robot
   from robotics_competition.task import Task
   from robotics_competition.team import Team

   alpha = Team("Alpha")
   alpha.add_robot(Robot("Scout", 30))  # 30%: chosen for this example
   alpha.add_robot(Robot("Hauler"))
   deliver = Task("deliver part", "critical", 12)  # Chosen tasks
   scan = Task("scan shelf", "medium", 3)
   for task in [deliver, scan]:
       print(task.name, alpha.assign(task), alpha.score)
   for robot in alpha:
       print(robot)

.. output: assign_demo.txt all

.. code-block:: text

   deliver part True 5
   scan shelf True 7
   Scout [6%, needs_recharge]
   Hauler [94%, active]

Rule 4, 2% per minute:

1. Deliver part: 12 min uses 24%. Scout has 30%: done, 6% left. Score: 5
   (critical).
2. Scan shelf: Scout is below 10% (rule 3) and refuses. Hauler uses 6%:
   94% left. Score: 5 + 2 = 7.


Inheritance
====================================================

A Subclass Is a Kind of Its Base Class
--------------------------------------

.. note::

   **Inheritance** makes a new class, the **subclass**, from an existing
   one, the **base class**. The subclass has every attribute and method
   of the base class, and adds or replaces some.

- Write the base class in parentheses: ``class Scout(Robot):``. A scout
  *is a* robot, so everything a robot does, a scout does.
- The tutorial: "if a requested attribute is not found in the class, the
  search proceeds to look in the base class."
- Use inheritance for *is a*. For *has a*, use composition: a robot has
  sensors; it is not a sensor.
- In Lecture 8, every ROS 2 node you write is a subclass of ``rclpy``'s
  ``Node``.

.. figure:: /_static/images/l7/inheritance_uml.svg
   :alt: A UML class diagram. Tag 1, at the top: the class Robot with plus battery: int; plus DRAIN_PCT_PER_MIN = 2, underlined; plus perform_task(task, minutes): bool; plus from_dict(config): Robot, underlined. Below it, two classes, each with a line up to a hollow triangle at Robot's bottom edge. Tag 2: Scout with plus DRAIN_PCT_PER_MIN = 1, underlined; plus max_speed_mps: float; plus drive_time(distance_m): float. Tag 3: Hauler with plus DRAIN_PCT_PER_MIN = 3, underlined; plus payload_kg: float; plus __str__(): str.
   :width: 85%

   1: the base class (some of its members). 2 and 3: each subclass lists
   only what it adds or replaces. The hollow triangle means "inherits
   from". Underlined: a class attribute or a class method.

super().__init__() Lets Robot Set Up Its Part
---------------------------------------------

.. code: lecture7/robotics_competition/scout.py all

.. code-block:: python
   :caption: lecture7/robotics_competition/scout.py

   """The Scout class: a light robot, a kind of Robot."""

   from robotics_competition.robot import Robot


   class Scout(Robot):
       """A light, fast robot: 1% of the battery per minute."""

       DRAIN_PCT_PER_MIN = 1  # Replaces Robot's 2; chosen for this example

       def __init__(
           self, name: str, battery: int = 100, max_speed_mps: float = 2.0
       ) -> None:
           super().__init__(name, battery)  # Robot sets name and battery
           self.max_speed_mps = max_speed_mps

       def drive_time(self, distance_m: float) -> float:
           """Seconds to drive distance_m at the scout's top speed."""
           return distance_m / self.max_speed_mps

- ``super()`` returns an object that sends a method call to the base
  class. ``super().__init__(name, battery)`` runs ``Robot``'s
  ``__init__`` on this scout: the name, the battery check, the sensor
  list, and the count.
- Then ``Scout`` adds its own attribute and method. Its default top
  speed, 2.0 m/s, is chosen for this example.

Without super().__init__(), the Base Class Sets Nothing
-------------------------------------------------------

.. code: lecture7/errors/missing_super_init.py all

.. code-block:: python
   :caption: lecture7/errors/missing_super_init.py

   class Robot:
       def __init__(self, name: str):
           self.name = name


   class Scout(Robot):
       def __init__(self, name: str, max_speed_mps: float = 2.0):
           self.max_speed_mps = max_speed_mps  # No super().__init__


   scout = Scout("Scout")
   print(scout.max_speed_mps)
   print(scout.name)

.. output: err_missing_super_init.txt all short

.. code-block:: text

   2.0
   Traceback (most recent call last):
     File ".../missing_super_init.py", line 13, in <module>
       print(scout.name)
             ^^^^^^^^^^
   AttributeError: 'Scout' object has no attribute 'name'

``Scout`` has its own ``__init__``, so Python calls it and never
``Robot``'s: ``name`` is never set. A subclass with no ``__init__`` of
its own uses the base class's, found by the same search.

Overriding: Hauler Replaces __str__ and Calls Robot's
-----------------------------------------------------

.. code: lecture7/robotics_competition/hauler.py all

.. code-block:: python
   :caption: lecture7/robotics_competition/hauler.py

   """The Hauler class: a robot that carries loads, a kind of Robot."""

   from robotics_competition.robot import Robot


   class Hauler(Robot):
       """A robot that carries loads: 3% of the battery per minute."""

       DRAIN_PCT_PER_MIN = 3  # Replaces Robot's 2; chosen for this example

       def __init__(
           self, name: str, battery: int = 100, payload_kg: float = 120.0
       ) -> None:
           super().__init__(name, battery)
           self.payload_kg = payload_kg

       def __str__(self) -> str:
           return f"{super().__str__()} carrying {self.payload_kg} kg"

- A method with the name of a base class method **overrides** it:
  ``print(hauler)`` runs ``Hauler``'s ``__str__``.
- ``super().__str__()`` runs ``Robot``'s version, and the override adds
  to its text instead of writing it again.
- The default payload, 120.0 kg, is chosen for this example.

Making a Scout and a Hauler
---------------------------

.. code: lecture7/inheritance_demo.py all

.. code-block:: python
   :caption: lecture7/inheritance_demo.py

   """Scout and Hauler inherit from Robot."""

   from robotics_competition.hauler import Hauler
   from robotics_competition.robot import Robot
   from robotics_competition.scout import Scout

   scout = Scout("Scout", max_speed_mps=2.5)  # Values chosen
   hauler = Hauler("Hauler", 80)
   print(repr(scout), scout.max_speed_mps, scout.drive_time(10.0))
   print(hauler.name, hauler.battery, hauler.payload_kg)
   print(hauler)

   print(isinstance(scout, Robot), isinstance(scout, Hauler))
   print(issubclass(Scout, Robot), issubclass(Robot, Scout))
   print([cls.__name__ for cls in Scout.__mro__])

   print(Scout.DRAIN_PCT_PER_MIN, Robot.DRAIN_PCT_PER_MIN)
   print("perform_task" in vars(Scout), "perform_task" in vars(Robot))
   print(Robot.total_robots, Scout.total_robots)
   print(repr(Scout.from_dict({"name": "S2", "battery": 70})))

.. output: inheritance_demo.txt all

.. code-block:: text

   Scout('Scout', battery=100) 2.5 4.0
   Hauler 80 120.0
   Hauler [80%, active] carrying 120.0 kg
   True False
   True False
   ['Scout', 'Robot', 'object']
   1 2
   False True
   2 2
   Scout('S2', battery=70)

- Lines 1 to 3: ``name``, ``battery`` and ``__repr__`` come from
  ``Robot``. The repr says ``Scout``: Lecture 7's ``__repr__`` writes
  ``type(self).__name__``, the object's own class. 10.0 m at 2.5 m/s
  takes 4.0 s.
- Lines 4 to 6: the tutorial says ``isinstance(obj, int)`` "will be
  ``True`` only if ``obj.__class__`` is ``int`` or some class derived from
  ``int``." A scout is a ``Robot``, not a ``Hauler``. ``__mro__``, the
  **method resolution order**, lists the classes Python searches for an
  attribute, after the object itself: ``Scout``, ``Robot``, and
  ``object``, the base of every class.

Robot's Methods Read the Subclass's Class Attribute
---------------------------------------------------

.. code: lecture7/robotics_competition/robot.py 74-80

.. code-block:: python
   :caption: From lecture7/robotics_competition/robot.py, in class Robot

       def perform_task(self, task: str, minutes: int) -> bool:
           """Do the task if active and the battery covers it; True if done."""
           drain = minutes * self.DRAIN_PCT_PER_MIN
           if self.status != "active" or drain > self._battery:
               return False
           self.battery = self._battery - drain
           return True

What the last four lines of ``inheritance_demo.py`` show (output lines 7
to 10):

1. ``Scout`` replaces ``DRAIN_PCT_PER_MIN`` with 1; ``Robot`` keeps 2.
2. ``perform_task`` is defined only in ``Robot``. On a scout, Python
   finds it there by the method resolution order, and
   ``self.DRAIN_PCT_PER_MIN`` then finds the scout's 1 first.
3. Lecture 6's ``robot.py`` read ``Robot.DRAIN_PCT_PER_MIN``, which
   always gives 2. Lecture 7's reads it through ``self``, so a subclass
   can change it.
4. ``Robot.total_robots += 1`` counts every robot, scouts and haulers
   too: 2.
5. ``Scout.from_dict(...)`` passes ``Scout`` as ``cls``, so it makes a
   scout.

Two Underscores Keep a Base Class's Name Apart
----------------------------------------------

.. code: lecture7/mangling_demo.py all

.. code-block:: python
   :caption: lecture7/mangling_demo.py

   """Two leading underscores: a base class and a subclass keep apart."""


   class Robot:
       def __init__(self):
           self.__serial = "R-100"  # Stored as _Robot__serial
           self._code = "R"


   class Scout(Robot):
       def __init__(self):
           super().__init__()
           self.__serial = "S-200"  # Stored as _Scout__serial: no clash
           self._code = "S"  # One underscore: replaces Robot's _code


   print(vars(Scout()))

.. output: mangling_demo.txt all

.. code-block:: text

   {'_Robot__serial': 'R-100', '_code': 'S', '_Scout__serial': 'S-200'}

- Lecture 6's name mangling: in class ``Scout``, ``__serial`` becomes
  ``_Scout__serial``, so it cannot overwrite ``Robot``'s
  ``_Robot__serial``.
- With one underscore, the subclass's ``self._code = "S"`` replaced
  ``Robot``'s ``"R"``.

A ROS 2 Node Is an Object of a Subclass of Node
-----------------------------------------------

This program needs ROS 2 Jazzy (Lecture 8). It makes one node and shuts
it down: nothing is published. It was run with ROS 2 sourced:

.. code-block:: bash

   source /opt/ros/jazzy/setup.bash
   uv run lecture7/ros_node_check.py

.. code: lecture7/ros_node_check.py all

.. code-block:: python
   :caption: lecture7/ros_node_check.py

   """A ROS 2 node is an object of a class that inherits from Node.

   Needs ROS 2 Jazzy sourced (Lecture 8). Makes one node and shuts it
   down: nothing is published.
   """

   import rclpy
   from rclpy.node import Node


   class ScoutNode(Node):
       def __init__(self):
           super().__init__("scout")  # Node.__init__ takes the node's name
           self.max_speed_mps = 2.0  # Chosen for this example


   rclpy.init()
   node = ScoutNode()
   print(node.get_name(), node.max_speed_mps, isinstance(node, Node))
   print([cls.__name__ for cls in ScoutNode.__mro__])
   node.destroy_node()
   rclpy.shutdown()

.. output: ros_node_check.txt all

.. code-block:: text

   scout 2.0 True
   ['ScoutNode', 'Node', 'object']

``super().__init__("scout")`` lets ``Node`` set up the node and give it
its name. Lecture 8 starts every node this way.


Polymorphism
====================================================

One Loop, Three Classes, Three Results
--------------------------------------

.. code: lecture7/polymorphism_demo.py all

.. code-block:: python
   :caption: lecture7/polymorphism_demo.py

   """The same calls on a Robot, a Scout, and a Hauler."""

   from robotics_competition.hauler import Hauler
   from robotics_competition.robot import Robot
   from robotics_competition.scout import Scout

   fleet = [Robot("Basic"), Scout("Scout"), Hauler("Hauler")]
   for robot in fleet:
       robot.perform_task("patrol", 10)  # 10 min: chosen for this example
       print(robot)

.. output: polymorphism_demo.txt all

.. code-block:: text

   Basic [80%, active]
   Scout [90%, active]
   Hauler [70%, active] carrying 120.0 kg

- **Polymorphism**: the same call does what each object's class says.
  The loop never asks which class it has.
- 10 minutes uses 10 × 2 = 20%, 10 × 1 = 10%, and 10 × 3 = 30%;
  ``print()`` runs ``Hauler``'s ``__str__`` for the hauler.
- Lecture 6's dunder methods work the same way: ``len()`` calls
  whichever ``__len__`` the object's class has.

Duck Typing: A Team Accepts Any Object with the Right Methods
-------------------------------------------------------------

.. code: lecture7/duck_typing_demo.py all

.. code-block:: python
   :caption: lecture7/duck_typing_demo.py

   """Duck typing: Team works with any object that has the methods."""

   from robotics_competition.task import Task
   from robotics_competition.team import Team


   class SimRobot:
       """A robot in a simulator: not a Robot, but the same names."""

       def __init__(self, name: str):
           self.name = name

       def perform_task(self, task: str, minutes: int) -> bool:
           print(f"[sim] {self.name} does {task} ({minutes} min)")
           return True


   beta = Team("Beta")
   beta.add_robot(SimRobot("Sim-1"))  # Not a Robot
   print(beta.assign(Task("scan shelf", "medium", 3)), beta.score)

.. output: duck_typing_demo.txt all

.. code-block:: text

   [sim] Sim-1 does scan shelf (3 min)
   True 2

- ``Team`` only reads ``name`` and calls ``perform_task()``.
  ``SimRobot`` has both, so it works, although it is not a ``Robot``.
- The glossary: duck typing "does not look at an object's type to
  determine if it has the right interface; instead, the method or
  attribute is simply called or used".

Exercise 3 on the :doc:`exercises page <l7_exercises>` practices the
`Inheritance`_ section and this one.


Abstract Base Classes
====================================================

An Abstract Base Class Says Which Methods a Subclass Must Have
--------------------------------------------------------------

.. code: lecture7/abc_demo.py all

.. code-block:: python
   :caption: lecture7/abc_demo.py

   """An abstract base class: every range reader must define read_m()."""

   from abc import ABC, abstractmethod


   class RangeReader(ABC):
       """Reads ranges from a sensor. Subclasses say how."""

       def __init__(self, max_range_m: float):
           self.max_range_m = max_range_m

       @abstractmethod
       def read_m(self) -> float:
           """Return one range, in meters."""

       def obstacle_within(self, limit_m: float) -> bool:
           """True if the reading is closer than limit_m."""
           return self.read_m() < limit_m


   class Lidar(RangeReader):
       def __init__(self, reading_m: float):
           super().__init__(12.0)  # A 12 m lidar, as in Lecture 2
           self._reading_m = reading_m  # A recorded reading, chosen

       def read_m(self) -> float:
           return min(self._reading_m, self.max_range_m)


   class Sonar(RangeReader):
       def __init__(self, echo_s: float):
           super().__init__(4.0)  # A 4 m sonar, chosen for this example
           self._echo_s = echo_s  # Time for the sound to go and come back

       def read_m(self) -> float:
           return round(343.0 * self._echo_s / 2, 2)  # Sound: 343 m/s


   for reader in [Lidar(2.5), Sonar(0.004)]:
       near = reader.obstacle_within(1.0)  # 1 m: chosen for this example
       print(type(reader).__name__, reader.read_m(), near)

.. output: abc_demo.txt all

.. code-block:: text

   Lidar 2.5 False
   Sonar 0.69 True

- An **abstract base class** (ABC) is a base class for others, never
  used for objects of its own. ``@abstractmethod`` marks ``read_m()``:
  each kind of sensor reads in its own way.
- ``obstacle_within()`` is **concrete**: it has code, and every subclass
  inherits it. It calls ``read_m()``, which the subclass defines.

What the output shows:

1. The lidar reads 2.5 m, more than 1.0 m: no obstacle.
2. The sonar's echo comes back after 0.004 s. Sound travels 343 m/s,
   there and back: 343 × 0.004 / 2 = 0.686 m, rounded to 0.69 m. That
   is less than 1.0 m: an obstacle.
3. Both answers came from ``RangeReader``'s ``obstacle_within()``, which
   called each subclass's own ``read_m()``.

The tutorial: "a method of a base class that calls another method
defined in the same base class may end up calling a method of a derived
class that overrides it."

A Missing Method Stops the Object from Being Made
-------------------------------------------------

.. code: lecture7/errors/abstract_missing.py all

.. code-block:: python
   :caption: lecture7/errors/abstract_missing.py

   from abc import ABC, abstractmethod


   class RangeReader(ABC):
       @abstractmethod
       def read_m(self) -> float:
           """Return one range, in meters."""


   class Camera(RangeReader):
       def read_range(self) -> float:  # Not read_m: a different name
           return 3.0


   print("Camera is defined")
   cam = Camera()

.. output: err_abstract_missing.txt all short

.. code-block:: text

   Camera is defined
   Traceback (most recent call last):
     File ".../abstract_missing.py", line 16, in <module>
       cam = Camera()
             ^^^^^^^^
   TypeError: Can't instantiate abstract class Camera without an implementation for abstract method 'read_m'

- The class is defined, but ``Camera()`` fails with a ``TypeError``.
- The library: such a class "cannot be instantiated unless all of its
  abstract methods and properties are overridden."
  ``RangeReader(5.0)`` fails the same way.
- The error comes when the object is made, not later, when ``read_m()``
  would be called. The class must inherit from ``ABC``: without it,
  nothing is checked.


Protocols
====================================================

mypy Rejects SimRobot: add_robot() Is Hinted Robot
--------------------------------------------------

.. code-block:: bash

   uvx mypy --pretty lecture7/duck_typing_demo.py

.. output: mypy_duck_typing_demo.txt all

.. code-block:: text

   lecture7/duck_typing_demo.py:19: error: Argument 1 to "add_robot" of
   "Team" has incompatible type "SimRobot"; expected "Robot"  [arg-type]
       beta.add_robot(SimRobot("Sim-1"))  # Not a Robot
                      ^~~~~~~~~~~~~~~~~
   Found 1 error in 1 file (checked 1 source file)

- The program ran (`Duck Typing: A Team Accepts Any Object with the
  Right Methods`_), but ``team.py`` hints ``robot: Robot``, and
  ``SimRobot`` is not a subclass of ``Robot``.
- mypy (Lecture 4) checks the hints without running the code. It found
  the ``robotics_competition`` package next to the file. Recorded with
  mypy 2.4.0 in a 72-column terminal; mypy wraps its messages to the
  width of yours.

A Protocol Names What a Caller Needs
------------------------------------

.. code: lecture7/protocol_demo.py all

.. code-block:: python
   :caption: lecture7/protocol_demo.py

   """A Protocol: any class with these names fits, with no inheritance."""

   from typing import Protocol, runtime_checkable

   from robotics_competition.robot import Robot
   from robotics_competition.scout import Scout
   from robotics_competition.sensor import Sensor


   @runtime_checkable
   class Performer(Protocol):
       """What a team needs from a robot: a name and perform_task()."""

       @property
       def name(self) -> str: ...

       def perform_task(self, task: str, minutes: int) -> bool: ...


   class SimRobot:
       """A robot in a simulator: not a Robot, but the same names."""

       def __init__(self, name: str):
           self.name = name

       def perform_task(self, task: str, minutes: int) -> bool:
           return True


   def run_all(performers: list[Performer], task: str) -> None:
       for performer in performers:
           print(performer.name, performer.perform_task(task, 5))


   run_all([Robot("Basic"), Scout("Scout"), SimRobot("Sim-1")], "patrol")
   print(isinstance(SimRobot("Sim-2"), Performer))
   print(isinstance(Sensor("lidar", 12.0), Performer))

.. output: protocol_demo.txt all

.. code-block:: text

   Basic True
   Scout True
   Sim-1 True
   True
   False

- A class fits ``Performer`` if it has a ``name`` and a
  ``perform_task()`` with these types. It does not inherit from
  ``Performer``.
- The library: protocols are "primarily used with static type checkers
  that recognize structural subtyping (static duck-typing)".

mypy Accepts Every Class That Fits, and Reports One That Does Not
-----------------------------------------------------------------

.. code-block:: bash

   uvx mypy --pretty lecture7/protocol_demo.py
   uvx mypy --pretty lecture7/errors/not_a_performer.py

.. output: mypy_protocol_demo.txt all

.. code-block:: text

   Success: no issues found in 1 source file

.. output: mypy_not_a_performer.txt all

.. code-block:: text

   lecture7/errors/not_a_performer.py:21: error: List item 0 has
   incompatible type "Sensor"; expected "Performer"  [list-item]
       run_all([Sensor("lidar")])  # No name, no perform_task
                ^~~~~~~~~~~~~~~
   Found 1 error in 1 file (checked 1 source file)

.. code: lecture7/errors/not_a_performer.py all

.. code-block:: python
   :caption: lecture7/errors/not_a_performer.py

   from typing import Protocol


   class Performer(Protocol):
       @property
       def name(self) -> str: ...

       def perform_task(self, task: str, minutes: int) -> bool: ...


   class Sensor:
       def __init__(self, kind: str):
           self.kind = kind


   def run_all(performers: list[Performer]) -> None:
       for performer in performers:
           print(performer.name, performer.perform_task("patrol", 5))


   run_all([Sensor("lidar")])  # No name, no perform_task

``not_a_performer.py`` passes a ``Sensor``, a class of that file with
only a ``kind``: no ``name`` and no ``perform_task()``. mypy reports it without running the program. Run
with Python, the program fails at ``performer.name``:

.. output: err_not_a_performer.txt 8

.. code-block:: text

   AttributeError: 'Sensor' object has no attribute 'name'

@runtime_checkable Lets isinstance() Check the Names
----------------------------------------------------

The last two lines of ``protocol_demo.py`` print ``True`` for a
``SimRobot`` and ``False`` for a ``Sensor``.

- ``@runtime_checkable`` on ``Performer`` lets ``isinstance()`` test an
  object against it while the program runs.
- The library: it "will check only the presence of the required methods
  or attributes, not their type signatures or types." A class whose
  ``perform_task`` is the number 5 passes too.

An Abstract Base Class or a Protocol
------------------------------------

.. list-table::
   :header-rows: 1
   :widths: 22 39 39
   :class: compact-table

   * -
     - Abstract base class
     - Protocol
   * - A class fits if
     - It inherits from the ABC
     - It has the names and types
   * - Checked by
     - Python, when an object is made
     - mypy, before the program runs
   * - Shared code
     - Concrete methods, inherited
     - None, for a class that fits without inheriting
   * - Example here
     - ``RangeReader``
     - ``Performer``

Use an ABC when you write the subclasses and they share code. Use a
protocol to accept classes you did not write, such as a simulator's
robot.

Exercise 4 on the :doc:`exercises page <l7_exercises>` (after class)
uses the relationships, inheritance, and polymorphism of this lecture.


The Package robotics_competition
====================================================

.. dropdown:: lecture7/robotics_competition/robot.py
    :class-container: sd-border-secondary

    .. code: lecture7/robotics_competition/robot.py all

    .. code-block:: python

       """The Robot class of the robotics competition (Lecture 7 version).

       Lecture 6's robot.py with four changes: from_dict() and battery_bar()
       (a class method and a static method); add_sensor(), which makes the
       robot's sensors (composition); the rule constants read through self, so
       a subclass can change them; and a repr that names the object's class.
       """

       from robotics_competition.sensor import Sensor


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
               self._sensors: list[Sensor] = []
               Robot.total_robots += 1

           @classmethod
           def from_dict(cls, config: dict) -> "Robot":
               """Make a robot from a dictionary, such as one read from a file."""
               return cls(config["name"], config.get("battery", 100))

           @staticmethod
           def battery_bar(level: int) -> str:
               """Draw a battery level as ten characters: [######----]."""
               bars = level // 10
               return "[" + "#" * bars + "-" * (10 - bars) + "]"

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
               if self._battery < self.LOW_BATTERY_PCT:
                   return "needs_recharge"
               return "active"

           @property
           def sensors(self) -> tuple[Sensor, ...]:
               """The robot's sensors, as a tuple: a caller cannot add one."""
               return tuple(self._sensors)

           def add_sensor(self, kind: str, range_m: float) -> None:
               """Make a sensor and mount it on this robot (composition)."""
               self._sensors.append(Sensor(kind, range_m))

           def perform_task(self, task: str, minutes: int) -> bool:
               """Do the task if active and the battery covers it; True if done."""
               drain = minutes * self.DRAIN_PCT_PER_MIN
               if self.status != "active" or drain > self._battery:
                   return False
               self.battery = self._battery - drain
               return True

           def recharge(self) -> None:
               """Fill the battery to 100%."""
               self.battery = 100

           def __repr__(self) -> str:
               cls = type(self).__name__
               return f"{cls}({self._name!r}, battery={self._battery})"

           def __str__(self) -> str:
               return f"{self._name} [{self._battery}%, {self.status}]"

.. dropdown:: lecture7/robotics_competition/team.py
    :class-container: sd-border-secondary

    .. code: lecture7/robotics_competition/team.py all

    .. code-block:: python

       """The Team class of the robotics competition."""

       from robotics_competition.robot import Robot
       from robotics_competition.task import Task


       class Team:
           """A team of robots made elsewhere, and the tasks they did."""

           MAX_ROBOTS = 3  # Rule 7, chosen for this example

           def __init__(self, name: str):
               self._name = name
               self._robots: list[Robot] = []  # Aggregation: robots are added
               self._done: list[Task] = []  # Association: tasks done

           @property
           def name(self) -> str:
               return self._name

           @property
           def score(self) -> int:
               """The points of the tasks done (rule 8), added at each read."""
               return sum(task.points for task in self._done)

           def add_robot(self, robot: Robot) -> None:
               """Add a robot made elsewhere, once; at most MAX_ROBOTS."""
               if robot in self._robots:
                   raise ValueError(f"{robot.name} is already in {self._name}")
               if len(self._robots) == Team.MAX_ROBOTS:  # Rule 7
                   raise ValueError(f"{self._name} is full")
               self._robots.append(robot)

           def assign(self, task: Task) -> bool:
               """Give the task to the first robot that does it; True if done."""
               for robot in self._robots:
                   if robot.perform_task(task.name, task.minutes):
                       self._done.append(task)
                       return True
               return False

           def __len__(self) -> int:
               return len(self._robots)

           def __iter__(self):
               return iter(self._robots)

           def __repr__(self) -> str:
               names = [robot.name for robot in self._robots]
               return f"Team({self._name!r}, {names}, score={self.score})"


Next Class
====================================================

:doc:`L8: ROS 2 Foundations </lectures/lecture8/l8_index>` (Tuesday,
March 30). **Quiz 3** takes the first 20 minutes of the class.

- What ROS is
- ROS 2 architecture: monolithic versus distributed systems
- Core components and the Data Distribution Service (DDS)
- The publish/subscribe model
- Introspection tools
- Workspaces and packages
- Writing nodes in Python with ``rclpy``
