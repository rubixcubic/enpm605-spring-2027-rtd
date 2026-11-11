====================================================
Python Exercises
====================================================

Four exercises on :doc:`Lecture 7 <l7_lecture>`. Exercises 1 to 3 are
the exercises from the slides; Exercise 4 is for after class. Write each
program in the ``lecture7/`` folder of the course repository and run it
in the course environment, for example ``uv run lecture7/ex1_decorators.py``.
Exercises 3 and 4 import from the ``robotics_competition`` package: run
them from the ``lecture7/`` folder (``uv run ex3_drone.py``), so Python
finds the package next to them.

.. note::

   Try each exercise before you open its solution. Every output in a
   solution was printed by running the solution program.


----


.. dropdown:: Exercise 1: Two Decorators for a Range Reading
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Write a decorator, a function that makes a decorator, and apply both
    to one function.

    **Tasks**

    .. code-block:: python

       # 1. Write a decorator count_calls. Its wrapper adds 1 to
       #    wrapper.calls, then calls the function. Before it returns
       #    wrapper, count_calls sets wrapper.calls = 0.
       # 2. Write clamp(low, high), which makes a decorator: the wrapper
       #    returns the function's result, limited to low to high.
       # 3. Use functools.wraps in both wrappers.
       # 4. Decorate read_range(raw_cm), which returns raw_cm / 100
       #    (centimeters to meters), with @count_calls on the first line
       #    and @clamp(0.1, 12.0) on the second.
       # Test: print read_range(250), read_range(5), read_range(2000),
       #   then read_range.calls and read_range.__name__.

    A function is an object, so a wrapper can have an attribute:
    ``wrapper.calls = 0``. The limits 0.1 m and 12.0 m are chosen for
    this exercise.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex1_decorators.py all

        .. code-block:: python

           import functools


           def count_calls(func):
               """Count the calls of func in wrapper.calls."""

               @functools.wraps(func)
               def wrapper(*args, **kwargs):
                   wrapper.calls += 1
                   return func(*args, **kwargs)

               wrapper.calls = 0
               return wrapper


           def clamp(low, high):
               """Make a decorator that limits a result to low to high."""

               def decorator(func):
                   @functools.wraps(func)
                   def wrapper(*args, **kwargs):
                       return min(max(func(*args, **kwargs), low), high)

                   return wrapper

               return decorator


           @count_calls
           @clamp(0.1, 12.0)
           def read_range(raw_cm):
               """Convert a raw reading in centimeters to meters."""
               return raw_cm / 100


           print(read_range(250), read_range(5), read_range(2000))
           print(read_range.calls, read_range.__name__, read_range.__doc__)

        .. output: exercises/ex1_decorators.txt all

        .. code-block:: text

           2.5 0.1 12.0
           3 read_range Convert a raw reading in centimeters to meters.

        1. The two ``@`` lines run from the bottom up:
           ``read_range = count_calls(clamp(0.1, 12.0)(read_range))``.
           Each call goes through the counting wrapper, then the
           clamping wrapper, then the original function.
        2. 250 cm is 2.5 m, inside the limits. 5 cm is 0.05 m, below
           0.1 m, so the result is 0.1. 2000 cm is 20.0 m, above 12.0 m,
           so the result is 12.0.
        3. ``read_range.calls`` is 3: the three calls. The name and the
           docstring are ``read_range``'s, because both wrappers use
           ``functools.wraps``.
        4. ``wrapper.calls += 1`` reads and assigns an attribute of the
           function object. It needs no ``nonlocal``: the name
           ``wrapper`` is only read, from the enclosing call.


.. dropdown:: Exercise 2: A Frozen Waypoint with Slots
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Write a data class that checks its fields, cannot change, can go in
    a set, and has no ``__dict__``.

    **Tasks**

    .. code-block:: python

       # 1. Write a data class Waypoint with frozen=True and slots=True:
       #    label (str), x and y (floats, meters).
       # 2. __post_init__ raises ValueError if label is empty.
       # 3. Make a = Waypoint("A", 0.0, 0.0) and b = Waypoint("B", 3.0, 4.0).
       #    Put a, b, and a second Waypoint("A", 0.0, 0.0) in a set; print
       #    its length and whether a is in it.
       # 4. Try Waypoint("", 1.0, 1.0) and b.x = 5.0: handle each error and
       #    print its type and message.
       # 5. Print hasattr(b, "__dict__") and Waypoint.__slots__.
       # 6. Print the distance from a to b: math.dist((a.x, a.y), (b.x, b.y)).

    Import with ``from dataclasses import dataclass,
    FrozenInstanceError``. The coordinates are chosen for this exercise.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex2_waypoint.py all

        .. code-block:: python

           import math
           from dataclasses import FrozenInstanceError, dataclass


           @dataclass(frozen=True, slots=True)
           class Waypoint:
               """A named point on the arena floor, in meters."""

               label: str
               x: float
               y: float

               def __post_init__(self) -> None:
                   if not self.label:
                       raise ValueError("label must not be empty")


           a = Waypoint("A", 0.0, 0.0)
           b = Waypoint("B", 3.0, 4.0)
           route = {a, b, Waypoint("A", 0.0, 0.0)}  # The second A equals the first
           print(len(route), a in route)
           try:
               Waypoint("", 1.0, 1.0)
           except ValueError as err:
               print("ValueError", err)
           try:
               b.x = 5.0
           except FrozenInstanceError as err:
               print("FrozenInstanceError", err)
           print(hasattr(b, "__dict__"), Waypoint.__slots__)
           print(math.dist((a.x, a.y), (b.x, b.y)))

        .. output: exercises/ex2_waypoint.txt all

        .. code-block:: text

           2 True
           ValueError label must not be empty
           FrozenInstanceError cannot assign to field 'x'
           False ('label', 'x', 'y')
           5.0

        1. ``frozen=True`` gives the class a ``__hash__``, so waypoints
           can go in a set. The second ``A`` equals the first (same
           fields), so the set has 2 items.
        2. ``__post_init__`` runs after the generated ``__init__`` has
           set the fields; ``not self.label`` is ``True`` for an empty
           string.
        3. ``slots=True`` writes ``__slots__`` from the fields, in their
           order, and the object has no ``__dict__``.
        4. From (0.0, 0.0) to (3.0, 4.0): the square root of 3.0² + 4.0²
           = 25.0 is 5.0 m.


.. dropdown:: Exercise 3: A Drone Joins a Team
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Write a subclass of the competition's ``Robot`` with ``super()``,
    replace a class attribute and a method, and see polymorphism in a
    team of three kinds of robot.

    **Tasks**

    .. code-block:: python

       # Import from robotics_competition: run this file from lecture7/.
       # 1. Write Drone(Robot). It uses 4% of the battery per minute.
       #    __init__ takes name, battery (default 100), and max_altitude_m
       #    (default 30.0), and calls super().__init__.
       # 2. Override __str__: Robot's text, then " up to 30.0 m".
       # 3. Make Team("Gamma") with Drone("Wasp"), Scout("Kit"), and
       #    Hauler("Mule", 40).
       # 4. Assign Task("survey roof", "high", 20), then
       #    Task("move pallet", "low", 10). After each, print the task's
       #    name, the result, and the score. Then print each robot, and
       #    whether it is a Robot.

    Predict the results before you run the program. The drain rate and
    the tasks are chosen for this exercise.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex3_drone.py all

        .. code-block:: python

           from robotics_competition.hauler import Hauler
           from robotics_competition.robot import Robot
           from robotics_competition.scout import Scout
           from robotics_competition.task import Task
           from robotics_competition.team import Team


           class Drone(Robot):
               """A flying robot: 4% of the battery per minute."""

               DRAIN_PCT_PER_MIN = 4  # Chosen for this exercise

               def __init__(
                   self, name: str, battery: int = 100, max_altitude_m: float = 30.0
               ) -> None:
                   super().__init__(name, battery)
                   self.max_altitude_m = max_altitude_m

               def __str__(self) -> str:
                   return f"{super().__str__()} up to {self.max_altitude_m} m"


           gamma = Team("Gamma")
           for robot in [Drone("Wasp"), Scout("Kit"), Hauler("Mule", 40)]:
               gamma.add_robot(robot)
           for task in [Task("survey roof", "high", 20), Task("move pallet", "low", 10)]:
               print(task.name, gamma.assign(task), gamma.score)
           for robot in gamma:
               print(robot, isinstance(robot, Robot))

        .. output: exercises/ex3_drone.txt all

        .. code-block:: text

           survey roof True 3
           move pallet True 4
           Wasp [20%, active] up to 30.0 m True
           Kit [90%, active] True
           Mule [40%, active] carrying 120.0 kg True

        1. ``DRAIN_PCT_PER_MIN = 4`` in ``Drone`` replaces ``Robot``'s 2.
           ``Robot.perform_task()`` reads ``self.DRAIN_PCT_PER_MIN``, so
           it finds the drone's 4.
        2. Survey roof, 20 min: the drone goes first and uses
           20 × 4 = 80%. 100% becomes 20%: done, 3 points (high).
        3. Move pallet, 10 min: the drone would need 10 × 4 = 40%, more
           than its 20%, so it refuses. The scout uses 10 × 1 = 10%:
           100% becomes 90%, done, 1 point. Score: 3 + 1 = 4.
        4. The hauler did nothing and keeps 40%. ``print(robot)`` runs
           each class's own ``__str__``: the drone's and the hauler's add
           to ``Robot``'s text with ``super().__str__()``. All three are
           robots.


.. dropdown:: Exercise 4: An Arena for the Competition (After Class)
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Write the ``Arena`` class of Lecture 6's design: an aggregation of
    teams that runs a round of tasks and ranks the teams by score.

    **Tasks**

    .. code-block:: python

       # In lecture7/ex4_arena.py, next to robotics_competition/:
       # 1. Arena(name, width_m, height_m) keeps its size and a list of
       #    teams made outside it (aggregation).
       # 2. add_team(team) raises ValueError if the team is already in
       #    the arena.
       # 3. run_round(tasks) offers each task to each team, in order,
       #    until one team does it (Team.assign() returns True).
       # 4. ranking() returns (name, score) pairs, best score first.
       # Test: team Alpha with Robot("Basic", 20), team Beta with
       #   Scout("Kit"), in Arena("Hall A", 20.0, 10.0). Add Alpha twice
       #   (handle the error). Run a round of Task("deliver part",
       #   "critical", 5), Task("scan shelf", "high", 30), and
       #   Task("carry crate", "high", 8). Print the ranking.

    Predict the ranking before you run it. ``sorted()`` takes a ``key``
    function (Lecture 4) and ``reverse=True``.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex4_arena.py all

        .. code-block:: python

           from robotics_competition.robot import Robot
           from robotics_competition.scout import Scout
           from robotics_competition.task import Task
           from robotics_competition.team import Team


           class Arena:
               """An arena: its size, and the teams that compete in it."""

               def __init__(self, name: str, width_m: float, height_m: float):
                   self.name = name
                   self.width_m = width_m
                   self.height_m = height_m
                   self._teams: list[Team] = []  # Teams are made outside: aggregation

               def add_team(self, team: Team) -> None:
                   if team in self._teams:
                       raise ValueError(f"{team.name} is already in {self.name}")
                   self._teams.append(team)

               def run_round(self, tasks: list[Task]) -> None:
                   """Offer each task to each team, in order, until one does it."""
                   for task in tasks:
                       for team in self._teams:
                           if team.assign(task):
                               break

               def ranking(self) -> list[tuple[str, int]]:
                   """(name, score) of each team, best score first."""
                   scores = [(team.name, team.score) for team in self._teams]
                   return sorted(scores, key=lambda pair: pair[1], reverse=True)


           alpha = Team("Alpha")
           alpha.add_robot(Robot("Basic", 20))
           beta = Team("Beta")
           beta.add_robot(Scout("Kit"))
           hall = Arena("Hall A", 20.0, 10.0)
           hall.add_team(alpha)
           hall.add_team(beta)
           try:
               hall.add_team(alpha)
           except ValueError as err:
               print("ValueError", err)
           hall.run_round(
               [
                   Task("deliver part", "critical", 5),
                   Task("scan shelf", "high", 30),
                   Task("carry crate", "high", 8),
               ]
           )
           print(hall.ranking())

        .. output: exercises/ex4_arena.txt all

        .. code-block:: text

           ValueError Alpha is already in Hall A
           [('Beta', 6), ('Alpha', 5)]

        1. Deliver part, 5 min: Alpha's robot uses 5 × 2 = 10%. 20%
           becomes 10%, not below 10%, so it is still active: done,
           Alpha 5 points (critical).
        2. Scan shelf, 30 min: Alpha's robot would need 60% and has 10%.
           Beta's scout uses 30 × 1 = 30%: 100% becomes 70%, Beta 3
           points (high).
        3. Carry crate, 8 min: Alpha's robot would need 16%. Beta's scout
           uses 8%: 70% becomes 62%, Beta 3 + 3 = 6 points.
        4. The ranking puts Beta (6) before Alpha (5), although Alpha was
           added first.
