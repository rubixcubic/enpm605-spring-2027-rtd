====================================================
Python Exercises
====================================================

Four exercises on :doc:`Lecture 6 <l6_lecture>`. Exercises 1 to 3 are
the exercises from the slides; Exercise 4 is for after class. Write each
program in the ``lecture6/`` folder of the course repository and run it
in the course environment, for example ``uv run lecture6/ex1_arm.py``.
Exercise 4 imports ``Robot`` from the ``robotics_competition`` package:
run it from the ``lecture6/`` folder (``uv run ex4_task.py``), so Python
finds the package next to it.

.. note::

   Try each exercise before you open its solution. Every output in a
   solution was printed by running the solution program.


----


.. dropdown:: Exercise 1: A RobotArm Class
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Write a class with ``__init__``, instance attributes, methods that
    change them, and a class attribute shared by every object.

    **Tasks**

    .. code-block:: python

       # 1. Write a class RobotArm. __init__ takes station (an int) and
       #    sets joint_angle to 0.0 (degrees) and gripping to False.
       # 2. pick_up() sets gripping to True; release() sets it to False.
       # 3. rotate(angle) adds angle to joint_angle.
       # 4. A class attribute total_arms counts the arms made so far.
       # 5. Make arms at stations 1 and 2. Arm 1 picks up, then turns
       #    45.0 degrees, then 30.0. Print vars() of both arms and
       #    RobotArm.total_arms.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex1_arm.py all

        .. code-block:: python

           class RobotArm:
               """A robot arm at a work station."""

               total_arms = 0  # Shared by every arm

               def __init__(self, station: int):
                   self.station = station
                   self.joint_angle = 0.0  # Degrees
                   self.gripping = False
                   RobotArm.total_arms += 1

               def pick_up(self) -> None:
                   self.gripping = True

               def release(self) -> None:
                   self.gripping = False

               def rotate(self, angle: float) -> None:
                   self.joint_angle += angle


           arm_1 = RobotArm(1)
           arm_2 = RobotArm(2)
           arm_1.pick_up()
           arm_1.rotate(45.0)
           arm_1.rotate(30.0)
           print(vars(arm_1))
           print(vars(arm_2))
           print(RobotArm.total_arms)

        .. output: exercises/ex1_arm.txt all

        .. code-block:: text

           {'station': 1, 'joint_angle': 75.0, 'gripping': True}
           {'station': 2, 'joint_angle': 0.0, 'gripping': False}
           2

        1. ``__init__`` sets three instance attributes; ``vars()`` shows
           them for each arm.
        2. Arm 1 turned 45.0 + 30.0 = 75.0 degrees and is gripping. Arm 2
           was never changed: each object has its own attributes.
        3. ``total_arms`` is a class attribute, so ``__init__`` adds 1
           through the class name, ``RobotArm.total_arms += 1``. Two arms
           were made: 2. With ``self.total_arms += 1``, each arm would
           get its own ``total_arms`` of 1 and the class would stay at 0.


.. dropdown:: Exercise 2: A Route Class with Dunder Methods
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Give a class its text (``__repr__``, ``__str__``), make it act like a
    container (``__len__``, ``__contains__``, ``__iter__``), and define
    ``==`` and ``+`` for it.

    **Tasks**

    .. code-block:: python

       # 1. Route(name) has a name and an empty list of waypoint names.
       # 2. add(waypoint) appends a waypoint name to the list.
       # 3. __repr__ returns  Route('patrol', ['A', 'B'])
       #    __str__ returns   patrol: A -> B
       # 4. __len__, __contains__, and __iter__ use the list of waypoints.
       # 5. __eq__: two routes are equal when their waypoints are equal
       #    (their names may differ).
       # 6. __add__: patrol + dock is a new Route named 'patrol+dock' with
       #    the waypoints of patrol, then those of dock.
       # Test: patrol has A and B, dock has C, other has A and B.
       #   Print repr(patrol), len(patrol), 'B' in patrol, patrol + dock,
       #   each waypoint of patrol, patrol == other, and patrol == dock.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex2_route.py all

        .. code-block:: python

           class Route:
               """A named route: waypoint names, in order."""

               def __init__(self, name: str):
                   self.name = name
                   self.waypoints = []

               def add(self, waypoint: str) -> None:
                   self.waypoints.append(waypoint)

               def __repr__(self) -> str:
                   return f"Route({self.name!r}, {self.waypoints!r})"

               def __str__(self) -> str:
                   return f"{self.name}: " + " -> ".join(self.waypoints)

               def __len__(self) -> int:
                   return len(self.waypoints)

               def __contains__(self, waypoint: str) -> bool:
                   return waypoint in self.waypoints

               def __iter__(self):
                   return iter(self.waypoints)

               def __eq__(self, other):
                   if not isinstance(other, Route):
                       return NotImplemented
                   return self.waypoints == other.waypoints

               def __add__(self, other):
                   if not isinstance(other, Route):
                       return NotImplemented
                   joined = Route(f"{self.name}+{other.name}")
                   joined.waypoints = self.waypoints + other.waypoints
                   return joined


           patrol = Route("patrol")
           patrol.add("A")
           patrol.add("B")
           dock = Route("dock")
           dock.add("C")
           print(repr(patrol), len(patrol), "B" in patrol)
           print(patrol + dock)
           for waypoint in patrol:
               print("-", waypoint)
           other = Route("other")
           other.add("A")
           other.add("B")
           print(patrol == other, patrol == dock)

        .. output: exercises/ex2_route.txt all

        .. code-block:: text

           Route('patrol', ['A', 'B']) 2 True
           patrol+dock: A -> B -> C
           - A
           - B
           True False

        1. ``{self.waypoints!r}`` puts the list's own ``repr()`` into the
           text, quotes included.
        2. ``print(patrol + dock)`` calls ``__add__``, then ``__str__``
           on the new route: ``" -> ".join()`` puts an arrow between the
           names.
        3. The ``for`` loop uses ``__iter__``, which returns the list's
           iterator.
        4. ``patrol == other`` is ``True``: same waypoints, different
           names. ``__eq__`` and ``__add__`` return ``NotImplemented``
           for anything that is not a ``Route``.


.. dropdown:: Exercise 3: An Encapsulated Sensor
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Protect a class's data with a read-only property and a property
    whose setter raises an exception for a bad value, and handle those
    exceptions with ``try`` and ``except``.

    **Tasks**

    .. code-block:: python

       # 1. Write a class Sensor: __init__(self, kind: str, range_m: float)
       # 2. kind is a read-only property.
       # 3. range_m is a property. Its setter raises TypeError if the value
       #    is not an int or a float, or is a bool, and ValueError if it is
       #    0 or less (rule 6). __init__ must use the setter.
       # 4. __repr__ returns  Sensor('lidar', 12.0)
       # 5. Make Sensor("lidar", 12.0). Then try range_m = -1.0,
       #    range_m = "far", and kind = "sonar": handle each error and
       #    print its type and message. Print the sensor at the end.

    ``isinstance(value, (int, float))`` is ``True`` for an int or a
    float, and also for ``True``: a ``bool`` is an ``int`` (Lecture 2).

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex3_sensor.py all

        .. code-block:: python

           class Sensor:
               """A range sensor: its kind and its range in meters (rule 6)."""

               def __init__(self, kind: str, range_m: float):
                   self._kind = kind
                   self.range_m = range_m  # Runs the setter

               @property
               def kind(self) -> str:
                   """The kind of sensor. It cannot change."""
                   return self._kind

               @property
               def range_m(self) -> float:
                   """The range, in meters."""
                   return self._range_m

               @range_m.setter
               def range_m(self, value: float) -> None:
                   if not isinstance(value, (int, float)) or isinstance(value, bool):
                       raise TypeError(f"range_m must be a number, got {value!r}")
                   if value <= 0:
                       raise ValueError(f"range_m must be positive, got {value}")
                   self._range_m = value

               def __repr__(self) -> str:
                   return f"Sensor({self._kind!r}, {self._range_m})"


           lidar = Sensor("lidar", 12.0)
           print(lidar)
           for value in [-1.0, "far"]:
               try:
                   lidar.range_m = value
               except (TypeError, ValueError) as err:
                   print(type(err).__name__, err)
           try:
               lidar.kind = "sonar"
           except AttributeError as err:
               print("AttributeError", err)
           print(lidar)

        .. output: exercises/ex3_sensor.txt all

        .. code-block:: text

           Sensor('lidar', 12.0)
           ValueError range_m must be positive, got -1.0
           TypeError range_m must be a number, got 'far'
           AttributeError property 'kind' of 'Sensor' object has no setter
           Sensor('lidar', 12.0)

        1. ``__init__`` writes ``self.range_m = range_m``, so a sensor
           made with a bad range is refused too.
        2. The type check comes first: with ``"far"``, the test
           ``value <= 0`` would itself raise a ``TypeError`` with a less
           clear message.
        3. ``kind`` has no setter, so assigning it raises
           ``AttributeError``. The sensor prints the same at the end:
           every bad change was refused.
        4. This class could go in ``robotics_competition/sensor.py``,
           the module the design gave it.


.. dropdown:: Exercise 4: A Task Class for the Competition (After Class)
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Write the ``Task`` class of the design: properties that enforce
    business rule 2, a ``__lt__`` that sorts tasks by priority, and a
    program that has the competition's ``Robot`` perform the tasks.

    **Tasks**

    .. code-block:: python

       # In lecture6/ex4_task.py, next to robotics_competition/:
       # 1. A class Task with __init__(self, name: str, priority: str,
       #    minutes: int).
       # 2. name is a read-only property.
       # 3. priority is a property; its setter raises ValueError unless
       #    the value is "low", "medium", "high", or "critical" (rule 2).
       # 4. minutes is a property; its setter raises TypeError if the
       #    value is not an int (or is a bool), and ValueError if it is
       #    0 or less.
       # 5. __repr__ returns  Task('carry crate', 'high', 6)
       # 6. __lt__ puts the more urgent task first, so sorted() gives
       #    critical, then high, medium, low.
       # 7. Show that Task("dance", "urgent", 1) and Task("lift", "low",
       #    2.5) are refused. Then
       #    Robot("Scout", 40) performs these tasks in sorted order, with
       #    perform_task(task.name, task.minutes), and prints each result:
       #    sweep zone A (low, 4 min), carry crate (high, 6 min),
       #    deliver part (critical, 8 min), scan shelf (medium, 3 min).

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex4_task.py all

        .. code-block:: python

           from robotics_competition.robot import Robot

           PRIORITIES = ["critical", "high", "medium", "low"]  # Rule 2, most urgent first


           class Task:
               """A competition task: a name, a priority, and a length in minutes."""

               def __init__(self, name: str, priority: str, minutes: int):
                   self._name = name
                   self.priority = priority  # Runs the setter
                   self.minutes = minutes

               @property
               def name(self) -> str:
                   return self._name

               @property
               def priority(self) -> str:
                   return self._priority

               @priority.setter
               def priority(self, value: str) -> None:
                   if value not in PRIORITIES:
                       raise ValueError(f"priority must be one of {PRIORITIES}")
                   self._priority = value

               @property
               def minutes(self) -> int:
                   return self._minutes

               @minutes.setter
               def minutes(self, value: int) -> None:
                   if not isinstance(value, int) or isinstance(value, bool):
                       raise TypeError(f"minutes must be an int, got {value!r}")
                   if value <= 0:
                       raise ValueError(f"minutes must be positive, got {value}")
                   self._minutes = value

               def __repr__(self) -> str:
                   return f"Task({self._name!r}, {self._priority!r}, {self._minutes})"

               def __lt__(self, other):
                   if not isinstance(other, Task):
                       return NotImplemented
                   mine = PRIORITIES.index(self._priority)
                   return mine < PRIORITIES.index(other.priority)


           tasks = [
               Task("sweep zone A", "low", 4),
               Task("carry crate", "high", 6),
               Task("deliver part", "critical", 8),
               Task("scan shelf", "medium", 3),
           ]
           try:
               Task("dance", "urgent", 1)
           except ValueError as err:
               print("ValueError", err)
           try:
               Task("lift", "low", 2.5)
           except TypeError as err:
               print("TypeError", err)

           scout = Robot("Scout", 40)  # 40%: chosen for this exercise
           for task in sorted(tasks):
               done = scout.perform_task(task.name, task.minutes)
               print(f"{task!r}: done={done}, {scout}")

        .. output: exercises/ex4_task.txt all

        .. code-block:: text

           ValueError priority must be one of ['critical', 'high', 'medium', 'low']
           TypeError minutes must be an int, got 2.5
           Task('deliver part', 'critical', 8): done=True, Scout [24%, active]
           Task('carry crate', 'high', 6): done=True, Scout [12%, active]
           Task('scan shelf', 'medium', 3): done=True, Scout [6%, needs_recharge]
           Task('sweep zone A', 'low', 4): done=False, Scout [6%, needs_recharge]

        1. ``Task("lift", "low", 2.5)`` has the wrong type for ``minutes``, so
           its setter raises ``TypeError``; a 0 would raise ``ValueError``.
        2. ``PRIORITIES`` lists the priorities from most to least urgent,
           so ``PRIORITIES.index()`` turns a priority into a number:
           ``"critical"`` is 0. ``__lt__`` compares those numbers.
        3. The battery, at 2% per minute (rule 4): 40 − 16 = 24 after the
           8 min task, 24 − 12 = 12 after the 6 min task, and 12 − 6 = 6
           after the 3 min task. 6% is below 10% (rule 3), so the robot
           refuses the last task.
        4. ``Task`` and ``Robot`` meet only through ``perform_task()``'s
           arguments here. Lecture 7 connects classes to each other.
