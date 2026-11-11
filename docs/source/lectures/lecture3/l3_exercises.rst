====================================================
Python Exercises
====================================================

Four exercises on :doc:`Lecture 3 <l3_lecture>`. They are the exercises
from the slides. Run every program inside the course environment.

.. note::

   Try each exercise before you open its solution. Every output in a
   solution was printed by running the solution program.


----


.. dropdown:: Exercise 1: Loops
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Write ``for`` and ``while`` loops with ``range()``, ``enumerate()``,
    ``break``, ``continue``, and ``else``.

    **Tasks**

    .. code-block:: python

       # 1. With range(), print the multiples of 3 from 3 to 30.

       # 2. With a for loop, count the letter "o" in this string.
       robot_name = "TurtleBot3 Waffle Robot"

       # 3. With a while loop, print the characters before the
       #    first space.
       sentence = "Robotics is fun"

       # 4. With range(), build this string in reverse.
       text = "ENPM605"

       # 5. With enumerate(), continue, break, and else:
       #    skip readings outside 0.12 to 12.0 m, print the index
       #    of the first reading below 0.5 m, and print "No obstacle"
       #    when there is none. Values chosen for this exercise.
       scan = [2.5, 0.0, 1.1, 0.4, 13.5, 0.8]

    For task 5, run your loop a second time with
    ``scan = [2.5, 0.0, 13.5, 0.8]``.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex1_loops.py all

        .. code-block:: python

           # 1. Multiples of 3 from 3 to 30
           print(list(range(3, 31, 3)))

           # 2. Count the letter "o"
           robot_name = "TurtleBot3 Waffle Robot"
           count = 0
           for ch in robot_name:
               if ch == "o":
                   count += 1
           print(count, robot_name.count("o"))

           # 3. The characters before the first space
           sentence = "Robotics is fun"
           i = 0
           while i < len(sentence) and sentence[i] != " ":
               i += 1
           print(sentence[:i])

           # 4. A string in reverse, with range()
           text = "ENPM605"
           result = ""
           for i in range(len(text) - 1, -1, -1):
               result += text[i]
           print(result)

           # 5. The first obstacle in a scan (values chosen for this exercise)
           scan = [2.5, 0.0, 1.1, 0.4, 13.5, 0.8]
           for i, r in enumerate(scan):
               if not (0.12 <= r <= 12.0):
                   continue
               if r < 0.5:
                   print(f"Obstacle at index {i}: {r} m")
                   break
           else:
               print("No obstacle")

           # 5. The same loop on a scan with no obstacle
           scan = [2.5, 0.0, 13.5, 0.8]
           for i, r in enumerate(scan):
               if not (0.12 <= r <= 12.0):
                   continue
               if r < 0.5:
                   print(f"Obstacle at index {i}: {r} m")
                   break
           else:
               print("No obstacle")

        .. output: exercises/ex1_loops.txt all

        .. code-block:: text

           [3, 6, 9, 12, 15, 18, 21, 24, 27, 30]
           3 3
           Robotics
           506MPNE
           Obstacle at index 3: 0.4 m
           No obstacle

        1. ``range(3, 31, 3)`` starts at 3 and steps by 3. The stop, 31,
           is not included, so the last number is 30.
        2. The loop counts 3 lowercase ``o``: one in ``TurtleBot3`` and two
           in ``Robot``. ``str.count()`` (Lecture 2) gives the same
           number.
        3. The ``while`` condition stops at the end of the string or at the
           first space, whichever comes first. ``sentence[:i]`` is the
           part before index ``i``.
        4. ``range(len(text) - 1, -1, -1)`` gives the indices 6, 5, 4, 3, 2,
           1, 0: the stop, -1, is not included, so index 0 is the last.
           ``text[::-1]`` (Lecture 2) does the same in one slice.
        5. ``0.0`` and ``13.5`` are outside 0.12 to 12.0 m and are
           skipped. ``1.1`` is valid but not below 0.5 m. ``0.4``, at index
           3, is the first valid reading below 0.5 m, so the loop breaks.
           In the second scan, no valid reading is below 0.5 m, so the
           loop ends without ``break`` and ``else`` prints
           ``No obstacle``. Lecture 4 turns the repeated loop into a
           function.


.. dropdown:: Exercise 2: Lists
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Predict what list operations do to the list and to other names, and
    build lists with comprehensions.

    **Tasks**

    Part A: predict, then run.

    .. code-block:: python

       a = [1, 2, 3]
       b = a
       c = a[:]
       b.append(4)
       print(a, c)
       print(a.append(5))
       print(a)
       nums = [3, 1, 2]
       print(sorted(nums), nums)
       grid = [[0] * 2] * 2
       grid[1][0] = 7
       print(grid)

    Part B: with ``scan = [2.5, 0.0, 3.1, 13.5, 1.2, 0.4, 0.05]`` (values
    chosen for this exercise), write one comprehension each for:

    1. The valid readings, from 0.12 to 12.0 m.
    2. The valid readings in centimeters, rounded to whole numbers with
       ``round()``.
    3. The indices, in ``scan``, of the valid readings below 0.5 m. Hint:
       ``enumerate()``.

    Then print the mean of the valid readings with 2 decimals, and
    whether any valid reading is below 0.5 m.

    Part C: build a grid of 4 rows and 5 columns of zeros, with one list
    per row. Set the cell at row index 2, column index 3 to 1, and print
    the grid with ``#`` for 1 and ``.`` for 0, one row per line.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex2_lists.py all

        .. code-block:: python

           # Part A: predict, then run
           a = [1, 2, 3]
           b = a
           c = a[:]
           b.append(4)
           print(a, c)
           print(a.append(5))
           print(a)
           nums = [3, 1, 2]
           print(sorted(nums), nums)
           grid = [[0] * 2] * 2
           grid[1][0] = 7
           print(grid)

           # Part B: one comprehension each (values chosen for this exercise)
           scan = [2.5, 0.0, 3.1, 13.5, 1.2, 0.4, 0.05]
           valid = [r for r in scan if 0.12 <= r <= 12.0]
           valid_cm = [round(r * 100) for r in valid]
           near = [i for i, r in enumerate(scan) if 0.12 <= r < 0.5]
           print(valid)
           print(valid_cm)
           print(near)
           print(f"Mean: {sum(valid) / len(valid):.2f} m", any(r < 0.5 for r in valid))

           # Part C: a 4 x 5 grid, one list per row
           grid = [[0] * 5 for _ in range(4)]
           grid[2][3] = 1
           for row in grid:
               for cell in row:
                   print("#" if cell else ".", end="")
               print()

        .. output: exercises/ex2_lists.txt all

        .. code-block:: text

           [1, 2, 3, 4] [1, 2, 3]
           None
           [1, 2, 3, 4, 5]
           [1, 2, 3] [3, 1, 2]
           [[7, 0], [7, 0]]
           [2.5, 3.1, 1.2, 0.4]
           [250, 310, 120, 40]
           [5]
           Mean: 1.80 m True
           .....
           .....
           ...#.
           .....

        **Part A.**

        1. ``b = a`` makes no copy, so ``b.append(4)`` changes the one list
           that ``a`` and ``b`` share. ``c = a[:]`` made a copy before the
           append, so ``c`` is still ``[1, 2, 3]``.
        2. ``append()`` changes the list in place and returns ``None``, so
           the second line prints ``None``. The list now ends with 5.
        3. ``sorted()`` returns a new list; ``nums`` is unchanged.
        4. ``[[0] * 2] * 2`` holds two references to one inner list, so
           ``grid[1][0] = 7`` shows in both rows.

        **Part B.** The valid readings are 2.5, 3.1, 1.2, and 0.4 m. Their
        mean, in steps:

        1. 2.5 + 3.1 + 1.2 + 0.4 = 7.2.
        2. 7.2 / 4 = 1.80.

        ``0.05`` is below 0.12 m, so it is not valid, and only ``0.4``, at
        index 5, is a valid reading below 0.5 m. ``round(r * 100)`` turns
        meters into whole centimeters.

        **Part C.** The comprehension builds a new ``[0] * 5`` for each of
        the 4 rows, so setting ``grid[2][3]`` changes one cell only.


.. dropdown:: Exercise 3: Dictionaries
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Update, rename, count, and filter with dictionaries.

    **Tasks**

    Part A: start from

    .. code-block:: python

       sensor = {
           "type": "lidar",
           "range_m": 100,
           "manufacturer": "Velodyne",
           "channels": 64,
       }

    1. The range was upgraded to 120 m: update the value.
    2. Rename the key ``"range_m"`` to ``"max_range_m"`` in one line.
       Hint: ``pop()`` returns the value it removes.
    3. Print the dictionary. Where is the renamed key now, and why?

    Part B: with
    ``detections = ["person", "cone", "person", "pallet", "cone", "person"]``,
    count each label in a dictionary with ``get()``. Then, with a
    dictionary comprehension, keep the labels seen at least twice.

    Part C: with
    ``battery = {"robot3": 42, "robot4": 9, "robot1": 87, "robot2": 15}``,
    print the names of the robots below 20%, sorted by name.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex3_dicts.py all

        .. code-block:: python

           # Part A (values from the exercise)
           sensor = {
               "type": "lidar",
               "range_m": 100,
               "manufacturer": "Velodyne",
               "channels": 64,
           }
           sensor["range_m"] = 120
           sensor["max_range_m"] = sensor.pop("range_m")
           print(sensor)

           # Part B: count detections (labels chosen for this exercise)
           detections = ["person", "cone", "person", "pallet", "cone", "person"]
           counts = {}
           for label in detections:
               counts[label] = counts.get(label, 0) + 1
           print(counts)
           frequent = {label: n for label, n in counts.items() if n >= 2}
           print(frequent)

           # Part C: robots below 20%, sorted by name (values chosen for this exercise)
           battery = {"robot3": 42, "robot4": 9, "robot1": 87, "robot2": 15}
           low = [name for name, pct in battery.items() if pct < 20]
           print(sorted(low))

        .. output: exercises/ex3_dicts.txt all

        .. code-block:: text

           {'type': 'lidar', 'manufacturer': 'Velodyne', 'channels': 64, 'max_range_m': 120}
           {'person': 3, 'cone': 2, 'pallet': 1}
           {'person': 3, 'cone': 2}
           ['robot2', 'robot4']

        - ``sensor.pop("range_m")`` removes the old pair and returns 120;
          ``sensor["max_range_m"] = ...`` adds a new key with that value.
        - The new key is a new entry, so it goes at the **end**: a
          dictionary keeps insertion order, and the old position of
          ``"range_m"`` is gone with it.
        - ``counts.get(label, 0) + 1`` starts each label at 1 the first
          time it is seen. ``collections.Counter(detections)`` gives the
          same counts.
        - The comprehension of Part C keeps the names whose value is
          below 20, in the order of the dictionary (``robot4`` before
          ``robot2``). ``sorted()`` puts them in name order.


.. dropdown:: Exercise 4: Robot Sensor Analysis
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Use every section of the lecture on one data set.

    **Tasks**

    .. code-block:: python

       # Readings of three robots, in meters (chosen for this exercise)
       sensor_data = {
           "robot_1": [0.5, 1.2, 0.8, 2.1, 0.3],
           "robot_2": [1.1, 0.9, 1.5, 0.7, 1.3],
           "robot_3": [0.2, 0.4, 0.6, 0.8, 1.0],
       }

    1. With a ``for`` loop, print each robot's name and its readings.
    2. Print each robot's mean reading, with 2 decimals.
    3. With a list comprehension, get ``robot_1``'s readings above 1.0 m.
    4. Build a set of every distinct reading over all robots. Print its
       size and the readings, sorted.
    5. With a dictionary comprehension, map each robot to its maximum
       reading.
    6. With ``items()`` and unpacking, find the robot with the largest
       maximum.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex4_sensor_analysis.py all

        .. code-block:: python

           # Readings of three robots, in meters (chosen for this exercise)
           sensor_data = {
               "robot_1": [0.5, 1.2, 0.8, 2.1, 0.3],
               "robot_2": [1.1, 0.9, 1.5, 0.7, 1.3],
               "robot_3": [0.2, 0.4, 0.6, 0.8, 1.0],
           }

           # 1. Each robot and its readings
           for robot, readings in sensor_data.items():
               print(robot, readings)

           # 2. The mean reading of each robot
           for robot, readings in sensor_data.items():
               print(f"{robot}: mean {sum(readings) / len(readings):.2f} m")

           # 3. robot_1's readings above 1.0 m
           print([r for r in sensor_data["robot_1"] if r > 1.0])

           # 4. Every distinct reading, over all robots
           unique = set()
           for readings in sensor_data.values():
               unique.update(readings)
           print(len(unique), sorted(unique))

           # 5. Each robot's maximum reading
           max_reading = {robot: max(readings) for robot, readings in sensor_data.items()}
           print(max_reading)

           # 6. The robot with the largest maximum
           best_robot, best_max = "", 0.0
           for robot, top in max_reading.items():
               if top > best_max:
                   best_robot, best_max = robot, top
           print(f"{best_robot} has the largest reading: {best_max} m")

        .. output: exercises/ex4_sensor_analysis.txt all

        .. code-block:: text

           robot_1 [0.5, 1.2, 0.8, 2.1, 0.3]
           robot_2 [1.1, 0.9, 1.5, 0.7, 1.3]
           robot_3 [0.2, 0.4, 0.6, 0.8, 1.0]
           robot_1: mean 0.98 m
           robot_2: mean 1.10 m
           robot_3: mean 0.60 m
           [1.2, 2.1]
           14 [0.2, 0.3, 0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0, 1.1, 1.2, 1.3, 1.5, 2.1]
           {'robot_1': 2.1, 'robot_2': 1.5, 'robot_3': 1.0}
           robot_1 has the largest reading: 2.1 m

        - The means, in steps: ``robot_1`` 0.5 + 1.2 + 0.8 + 2.1 + 0.3 = 4.9,
          and 4.9 / 5 = 0.98; ``robot_2`` 5.5 / 5 = 1.10; ``robot_3``
          3.0 / 5 = 0.60.
        - The three lists hold 15 readings, but ``0.8`` appears in
          ``robot_1`` and ``robot_3``, so the set holds 14.
        - ``set.update()`` adds every item of an iterable, as
          ``list.extend()`` does for a list.
        - Task 6 keeps the best robot so far in two names, and replaces
          them when a larger maximum comes. ``max()`` can do it in one
          call when you give it a function as its ``key``; Lecture 4
          covers functions as values.
