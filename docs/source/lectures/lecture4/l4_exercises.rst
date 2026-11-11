====================================================
Python Exercises
====================================================

Four exercises on :doc:`Lecture 4 <l4_lecture>`. They are the exercises
from the slides. Run every program inside the course environment.

.. note::

   Try each exercise before you open its solution. Every output in a
   solution was printed by running the solution program.


----


.. dropdown:: Exercise 1: Functions and Arguments
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Write functions with docstrings, default values, keyword arguments,
    and ``*args``, and unpack a list into a call.

    **Tasks**

    .. code-block:: python

       # Scans in meters (chosen for this exercise)
       scans = [[2.5, 0.0, 3.1], [14.0, 0.9], [0.05, 13.0]]

       # 1. Write mean_range(scan): the mean of the readings from 0.12
       #    to 12.0 m, or None when there is none. Give it a docstring.
       #    Call it on scans[0] and on scans[2].
       # 2. Make the limits parameters with defaults, min_m=0.12 and
       #    max_m=12.0. Call it on [3.5, 0.01, 4.2] for a sonar that
       #    reads 0.02 to 4.0 m, with keyword arguments.
       # 3. Write summarize(*scans): a list with one (nearest, farthest)
       #    tuple per scan. Call it with two scans.
       # 4. Call summarize() once more, with the list scans unpacked by *.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex1_functions.py all

        .. code-block:: python

           # Scans in meters (chosen for this exercise)
           scans = [[2.5, 0.0, 3.1], [14.0, 0.9], [0.05, 13.0]]


           # 1 and 2. The mean of the valid readings, or None
           def mean_range(scan, min_m=0.12, max_m=12.0):
               """Return the mean of the readings in [min_m, max_m], or None."""
               valid = [r for r in scan if min_m <= r <= max_m]
               if not valid:
                   return None
               return sum(valid) / len(valid)


           print(mean_range(scans[0]))
           print(mean_range(scans[2]))
           print(mean_range([3.5, 0.01, 4.2], min_m=0.02, max_m=4.0))


           # 3. One (nearest, farthest) tuple per scan
           def summarize(*scans):
               """Return a list of (nearest, farthest) tuples, one per scan."""
               return [(min(s), max(s)) for s in scans]


           print(summarize([2.5, 0.9], [1.2]))

           # 4. The list of scans, unpacked with *
           print(summarize(*scans))

        .. output: exercises/ex1_functions.txt all

        .. code-block:: text

           2.8
           None
           3.5
           [(0.9, 2.5), (1.2, 1.2)]
           [(0.0, 3.1), (0.9, 14.0), (0.05, 13.0)]

        1. ``scans[0]`` has two valid readings, 2.5 and 3.1:
           2.5 + 3.1 = 5.6, and 5.6 / 2 = 2.8. ``scans[2]`` has none
           (0.05 is below 0.12 and 13.0 is above 12.0), so the function
           returns ``None``.
        2. For the sonar, only 3.5 is from 0.02 to 4.0 m (0.01 is below,
           4.2 is above), so the mean is 3.5. The keyword arguments make
           the call say which limit is which.
        3. ``*scans`` collects the two lists in a tuple; the comprehension
           makes one ``(min, max)`` tuple per list. These are the raw
           readings: ``summarize()`` does not filter invalid ones, so
           ``0.0`` and ``14.0`` appear in task 4.
        4. ``summarize(*scans)`` passes the three lists as three
           positional arguments, the same as
           ``summarize(scans[0], scans[1], scans[2])``.


.. dropdown:: Exercise 2: Scopes and Pass-by-Assignment
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Predict what a call does to the caller's objects, and which name each
    assignment changes.

    **Tasks**

    Part A: predict the two printed lines, then run.

    .. code-block:: python

       def edit(route, sensors, battery_pct):
           route.append((3.0, 1.0))
           route[0] = (0.5, 0.0)
           route = [(9.0, 9.0)]
           sensors = ["sonar"]
           battery_pct += 5
           return battery_pct


       route = [(0.0, 0.0), (1.0, 0.5)]
       sensors = ["lidar", "imu"]
       battery_pct = 50
       print(edit(route, sensors, battery_pct))
       print(route, sensors, battery_pct)

    Part B: predict the three printed lines, then run.

    .. code-block:: python

       a = 1
       b = 1


       def outer():
           a = 2
           b = 2

           def inner():
               global b
               b = 3
               a = 3
               print("inner:", a, b)

           inner()
           print("outer:", a, b)


       outer()
       print("module:", a, b)

    Part C: change one line of ``inner()`` so that ``outer()`` prints
    ``b`` as 3 and the module's ``b`` stays 1. Start again from ``a = 1``
    and ``b = 1``, then predict the three lines.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex2_scopes.py all

        .. code-block:: python

           # Part A: predict, then run
           def edit(route, sensors, battery_pct):
               route.append((3.0, 1.0))
               route[0] = (0.5, 0.0)
               route = [(9.0, 9.0)]
               sensors = ["sonar"]
               battery_pct += 5
               return battery_pct


           route = [(0.0, 0.0), (1.0, 0.5)]
           sensors = ["lidar", "imu"]
           battery_pct = 50
           print(edit(route, sensors, battery_pct))
           print(route, sensors, battery_pct)

           # Part B: predict, then run
           a = 1
           b = 1


           def outer():
               a = 2
               b = 2

               def inner():
                   global b
                   b = 3
                   a = 3
                   print("inner:", a, b)

               inner()
               print("outer:", a, b)


           outer()
           print("module:", a, b)


           # Part C: nonlocal b instead of global b, from a = 1 and b = 1 again
           a = 1
           b = 1


           def outer():
               a = 2
               b = 2

               def inner():
                   nonlocal b
                   b = 3
                   a = 3
                   print("inner:", a, b)

               inner()
               print("outer:", a, b)


           outer()
           print("module:", a, b)

        .. output: exercises/ex2_scopes.txt all

        .. code-block:: text

           55
           [(0.5, 0.0), (1.0, 0.5), (3.0, 1.0)] ['lidar', 'imu'] 50
           inner: 3 3
           outer: 2 2
           module: 1 3
           inner: 3 3
           outer: 2 3
           module: 1 1

        **Part A.**

        - ``route.append(...)`` and ``route[0] = ...`` change the
          caller's list: the parameter refers to it. The list ends as
          ``[(0.5, 0.0), (1.0, 0.5), (3.0, 1.0)]``.
        - ``route = [(9.0, 9.0)]`` and ``sensors = ["sonar"]`` only bind
          the call's names to new lists; the caller sees neither. Ruff
          flags only ``sensors = ["sonar"]``, as F841: the function never
          reads ``sensors``. It does read ``route`` (``route.append``), and
          Ruff does not flag ``route = [(9.0, 9.0)]``, even though that new
          list is never used.
        - ``battery_pct += 5`` makes a new int, 50 + 5 = 55, bound to the
          call's name. The call returns 55; the module's ``battery_pct``
          stays 50.

        **Part B.**

        - ``global b`` makes ``b`` in ``inner()`` the module's ``b``, so
          ``b = 3`` changes the module's ``b`` from 1 to 3.
        - ``a = 3`` makes a local ``a`` in ``inner()``; ``outer()``'s
          ``a`` and ``b`` stay 2.
        - So: ``inner: 3 3``, ``outer: 2 2``, ``module: 1 3``.

        **Part C.** ``nonlocal b`` instead of ``global b`` makes ``b`` the
        name of ``outer()``: ``outer: 2 3`` and ``module: 1 1``.


.. dropdown:: Exercise 3: Recursion, Lambdas, and Closures
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Write a recursive function with a base case, a sort key as a lambda,
    and a closure that keeps state between calls.

    **Tasks**

    .. code-block:: python

       # 1. With recursion, write sum_digits(n) for an int n >= 0:
       #    sum_digits(605) is 6 + 0 + 5 = 11. Add type hints.
       #    Hint: n % 10 is the last digit; n // 10 is the rest.

       # 2. Sort these robots by battery, highest first, with a lambda.
       #    (Values chosen for this exercise.)
       robots = [("waffle_01", 87), ("burger_02", 15), ("waffle_03", 52)]

       # 3. Write make_accumulator(start), which returns a function.
       #    Each call of that function adds its argument to a running
       #    total and returns the total. With acc = make_accumulator(100),
       #    acc(10) gives 110, then acc(20) gives 130.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex3_values.py all

        .. code-block:: python

           # 1. The sum of the digits of n, with recursion
           def sum_digits(n: int) -> int:
               if n < 10:  # Base case: one digit
                   return n
               return n % 10 + sum_digits(n // 10)  # Last digit + the rest


           print(sum_digits(605), sum_digits(7), sum_digits(1234))

           # 2. Robots sorted by battery, highest first (chosen for this exercise)
           robots = [("waffle_01", 87), ("burger_02", 15), ("waffle_03", 52)]
           print(sorted(robots, key=lambda r: r[1], reverse=True))


           # 3. A running total kept in a closure
           def make_accumulator(start: float):
               total = start

               def add(amount: float) -> float:
                   nonlocal total
                   total += amount
                   return total

               return add


           acc = make_accumulator(100)
           print(acc(10), acc(20))
           other = make_accumulator(0)
           print(other(5), acc(0))

        .. output: exercises/ex3_values.txt all

        .. code-block:: text

           11 7 10
           [('waffle_01', 87), ('waffle_03', 52), ('burger_02', 15)]
           110 130
           5 130

        1. The base case is a single digit, ``n < 10``. Otherwise the
           function adds the last digit to the sum of the rest. For 605:
           5 + sum_digits(60); 60 gives 0 + sum_digits(6); 6 is the base
           case. So 5 + 0 + 6 = 11. For 1234: 4 + 3 + 2 + 1 = 10.
        2. ``key=lambda r: r[1]`` sorts by the battery, and
           ``reverse=True`` puts the highest first: 87, 52, 15.
        3. ``total`` lives in the closure's cell, and ``nonlocal total``
           lets ``add()`` change it: 100 + 10 = 110, then 110 + 20 = 130.
           ``other`` is a second accumulator with its own cell, so
           ``other(5)`` gives 5 and ``acc(0)`` still gives 130.


.. dropdown:: Exercise 4: Callbacks for Two Robots (After Class)
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Use several sections of the lecture in one program: callbacks, a
    function factory, ``functools.partial``, and a closure.

    **Tasks**

    .. code-block:: python

       # 1. Write subscribe(topic, callback) and publish(topic, msg),
       #    as in the lecture.
       # 2. Write make_filter(min_m, max_m). It returns a function that
       #    keeps the readings of a scan from min_m to max_m.
       # 3. Write report(robot, scan). It prints the robot's name, the
       #    number of valid readings (0.12 to 12.0 m), and the nearest
       #    one (None when there is none). Subscribe it to
       #    "/robot_1/scan" and "/robot_2/scan" with functools.partial.
       # 4. With a closure, count the scans published on both topics.
       # 5. Publish these scans, in this order, then print the count
       #    (values chosen for this exercise):
       #    "/robot_1/scan": [2.5, 0.0, 3.1]
       #    "/robot_2/scan": [14.0, 0.9, 0.3]
       #    "/robot_1/scan": [0.05, 13.0]

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex4_pipeline.py all

        .. code-block:: python

           from functools import partial

           subscribers = {}


           # 1. subscribe() and publish()
           def subscribe(topic, callback):
               if topic not in subscribers:
                   subscribers[topic] = []
               subscribers[topic].append(callback)


           def publish(topic, msg):
               for callback in subscribers.get(topic, []):
                   callback(msg)


           # 2. A factory for range filters
           def make_filter(min_m, max_m):
               def keep_valid(scan):
                   return [r for r in scan if min_m <= r <= max_m]

               return keep_valid


           lidar_filter = make_filter(0.12, 12.0)


           # 3. One callback, two robots, with partial
           def report(robot, scan):
               valid = lidar_filter(scan)
               nearest = min(valid) if valid else None
               print(f"{robot}: {len(valid)} valid, nearest {nearest}")


           subscribe("/robot_1/scan", partial(report, "robot_1"))
           subscribe("/robot_2/scan", partial(report, "robot_2"))


           # 4. A closure that counts every scan published
           def make_counter():
               count = 0

               def increment(_scan):
                   nonlocal count
                   count += 1

               def total():
                   return count

               return increment, total


           on_any_scan, scans_seen = make_counter()
           subscribe("/robot_1/scan", on_any_scan)
           subscribe("/robot_2/scan", on_any_scan)

           # 5. Publish (scans chosen for this exercise)
           publish("/robot_1/scan", [2.5, 0.0, 3.1])
           publish("/robot_2/scan", [14.0, 0.9, 0.3])
           publish("/robot_1/scan", [0.05, 13.0])
           print("scans published:", scans_seen())

        .. output: exercises/ex4_pipeline.txt all

        .. code-block:: text

           robot_1: 2 valid, nearest 2.5
           robot_2: 2 valid, nearest 0.3
           robot_1: 0 valid, nearest None
           scans published: 3

        - ``make_filter(0.12, 12.0)`` returns a closure that keeps
          ``min_m`` and ``max_m``.
        - ``partial(report, "robot_1")`` fills in the name; ``publish()``
          adds the scan.
        - ``make_counter()`` returns two functions that share one cell:
          ``on_any_scan`` adds 1, ``scans_seen`` reads the count. Both
          topics subscribe ``on_any_scan``, so the count is 3.
        - The first scan keeps 2.5 and 3.1 (``0.0`` is below 0.12 m); the
          second keeps 0.9 and 0.3 (``14.0`` is above 12.0 m); the third
          keeps nothing, so its nearest is ``None``.
