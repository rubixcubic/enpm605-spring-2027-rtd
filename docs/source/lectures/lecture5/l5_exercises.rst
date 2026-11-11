====================================================
Python Exercises
====================================================

Four exercises on :doc:`Lecture 5 <l5_lecture>`. Exercises 1 to 3 are
the exercises from the slides; Exercise 4 is for after class. Run every
program inside the course environment. Exercises 2 to 4 import
``geometry.py``, ``scan_data.py``, or ``plots.py``: write them in the
``lecture5/`` folder of the course repository, next to those files, and
run them from that folder, for example ``uv run ex3_scan.py``.

.. note::

   Try each exercise before you open its solution. Every output in a
   solution was printed by running the solution program.


----


.. dropdown:: Exercise 1: Arrays, Indexing, and Masks
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Read an array's shape and data type, index rows and columns, and
    select and change items with a Boolean mask.

    **Tasks**

    .. code-block:: python

       import numpy as np

       # Battery (%) and position (x, y in m) of 6 robots (chosen)
       battery = np.array([87, 15, 52, 9, 64, 30])
       x = np.array([0.5, 2.0, 4.0, 1.5, 3.0, 5.0])
       y = np.array([1.0, 3.5, 0.5, 2.5, 3.0, 1.5])
       xy = np.column_stack([x, y])  # One row per robot: (x, y)

       # 1. Print the shape and the dtype of battery and of xy.
       # 2. Print every robot's x (from xy), and the position of robot 3.
       # 3. With a mask, print the batteries below 20% and how many.
       # 4. Print the positions of the robots below 20%.
       # 5. In a copy, cap every battery at 80. Check battery is unchanged.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex1_arrays.py all

        .. code-block:: python

           import numpy as np

           # Battery (%) and position (x, y in m) of 6 robots (chosen)
           battery = np.array([87, 15, 52, 9, 64, 30])
           x = np.array([0.5, 2.0, 4.0, 1.5, 3.0, 5.0])
           y = np.array([1.0, 3.5, 0.5, 2.5, 3.0, 1.5])
           xy = np.column_stack([x, y])  # One row per robot: (x, y)

           # 1. Shape and data type
           print(battery.shape, battery.dtype, xy.shape, xy.dtype)

           # 2. Every x (column 0), and robot 3 (row 3)
           print(xy[:, 0])
           print(xy[3])

           # 3. The batteries below 20%, and how many
           low = battery < 20
           print(battery[low], low.sum())

           # 4. The positions of those robots: the same mask selects rows
           print(xy[low])

           # 5. Cap at 80 in a copy
           capped = battery.copy()
           capped[capped > 80] = 80
           print(capped)
           print(battery)

        .. output: exercises/ex1_arrays.txt all

        .. code-block:: text

           (6,) int64 (6, 2) float64
           [0.5 2.  4.  1.5 3.  5. ]
           [1.5 2.5]
           [15  9] 2
           [[2.  3.5]
            [1.5 2.5]]
           [80 15 52  9 64 30]
           [87 15 52  9 64 30]

        1. ``battery`` holds whole numbers, so its type is ``int64``;
           ``xy`` has 6 rows and 2 columns of ``float64``.
        2. ``xy[:, 0]`` is column 0, every ``x``. ``xy[3]`` is row 3:
           robot 3, counted from 0, at (1.5, 2.5).
        3. ``battery < 20`` is ``True`` for 15 and 9, so ``low.sum()``
           is 2.
        4. A mask with one item per row selects whole rows: robots 1 and
           3, at (2.0, 3.5) and (1.5, 2.5).
        5. Only 87 is above 80. ``capped`` is a copy, so ``battery``
           still prints 87.


.. dropdown:: Exercise 2: Vectors, Transforms, and Broadcasting
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Compute a distance and a direction, move points between the
    robot's frame and the world frame, and build a table with
    broadcasting.

    **Tasks**

    .. code-block:: python

       import numpy as np
       from geometry import rot

       # The robot is at (1.0, 2.0) with heading 45 degrees (chosen).
       # 1. Print the distance and the direction (degrees, world frame)
       #    from the robot to the goal (4.0, 6.0).
       # 2. A landmark is 2.0 m ahead of the robot and 1.0 m to its left.
       #    Find it in the world frame, then transform it back to check.
       # 3. Transform the robot-frame points [1, 0], [0, 1], and [-1, 0]
       #    to the world frame in one expression.
       # 4. Robots at x = 0.5, 3.0, and 7.0 m and goals at x = 1.0, 4.0,
       #    and 6.0 m are on one line. With broadcasting, build the table
       #    of distances and print the nearest goal of each robot.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex2_transforms.py all

        .. code-block:: python

           import numpy as np
           from geometry import rot

           # The robot is at (1.0, 2.0) with heading 45 degrees (chosen)
           t = np.array([1.0, 2.0])
           R = rot(np.radians(45))

           # 1. Distance and direction to the goal (4.0, 6.0)
           d = np.array([4.0, 6.0]) - t
           print(np.linalg.norm(d), np.degrees(np.arctan2(d[1], d[0])))

           # 2. A landmark 2.0 m ahead and 1.0 m to the left, and back
           landmark = R @ [2.0, 1.0] + t
           print(np.round(landmark, 3))
           print(np.round(R.T @ (landmark - t), 3))

           # 3. Three points at once: one row per point
           pts = np.array([[1.0, 0.0], [0.0, 1.0], [-1.0, 0.0]])
           print(np.round(pts @ R.T + t, 3))

           # 4. The nearest goal of each robot, with broadcasting
           robots_x = np.array([[0.5], [3.0], [7.0]])  # (3, 1)
           goals_x = np.array([1.0, 4.0, 6.0])  # (3,)
           dist = np.abs(robots_x - goals_x)  # (3, 3)
           print(dist)
           print(dist.argmin(axis=1))

        .. output: exercises/ex2_transforms.txt all

        .. code-block:: text

           5.0 53.13010235415598
           [1.707 4.121]
           [2. 1.]
           [[1.707 2.707]
            [0.293 2.707]
            [0.293 1.293]]
           [[0.5 3.5 5.5]
            [2.  1.  3. ]
            [6.  3.  1. ]]
           [0 1 2]

        1. The goal is 3.0 m along x and 4.0 m along y from the robot:
           √(3² + 4²) = 5.0 m, in the direction 53.13°.
        2. "Ahead and to the left" is (2.0, 1.0) in the robot's frame.
           With cos 45° = sin 45° = 0.707: turned, it is
           (2 × 0.707 − 0.707, 2 × 0.707 + 0.707) = (0.707, 2.121); plus
           (1.0, 2.0), it is (1.707, 4.121). Back: subtract (1.0, 2.0),
           then turn by −45° with ``R.T``: (2.0, 1.0) again.
        3. One row per point: ``pts @ R.T + t``. The point 1 m ahead
           lands at (1.707, 2.707), the one 1 m behind at
           (0.293, 1.293).
        4. A ``(3, 1)`` column minus a ``(3,)`` row gives a ``(3, 3)``
           table. Row 2: the robot at 7.0 m is 6.0, 3.0, and 1.0 m from
           the goals, so its nearest goal is 2, at 6.0 m.


.. dropdown:: Exercise 3: Process the Scan
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Use the scan of ``scan_data.py``: count invalid readings, find the
    nearest reading in a sector, and make a stop decision.

    **Tasks**

    .. code-block:: python

       import numpy as np
       from scan_data import make_scan

       scan = make_scan()
       # 1. Turn scan.ranges into an array, and compute the angle of
       #    each reading and the valid mask.
       # 2. Print the number of NaN readings and of inf readings.
       # 3. Print the range and the angle (degrees) of the nearest valid
       #    reading on the robot's left: angles from 60 to 120 degrees.
       # 4. Print the mean of the valid readings behind the robot,
       #    |angle| >= 150 degrees, and how many there are.
       # 5. Print "stop" if a valid reading within 30 degrees of straight
       #    ahead is under 0.5 m, else "go".

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex3_scan.py all

        .. code-block:: python

           import numpy as np
           from scan_data import make_scan

           # 1. The ranges as an array, and the angle of each reading
           scan = make_scan()
           ranges = np.asarray(scan.ranges)
           angles = scan.angle_min + np.arange(ranges.size) * scan.angle_increment
           deg = np.degrees(angles)
           valid = (ranges >= scan.range_min) & (ranges <= scan.range_max)

           # 2. The NaN readings and the inf readings
           print(np.isnan(ranges).sum(), np.isinf(ranges).sum())

           # 3. The nearest valid reading on the left: 60 to 120 degrees
           left = valid & (deg >= 60) & (deg <= 120)
           i = np.where(left, ranges, np.inf).argmin()
           print(f"left: {ranges[i]:.3f} m at {deg[i]:.0f} deg")

           # 4. The mean of the valid readings behind: |angle| >= 150 degrees
           behind = valid & (np.abs(deg) >= 150)
           print(f"behind: {ranges[behind].mean():.3f} m, {behind.sum()} readings")

           # 5. Stop if a valid reading in front (within 30 degrees) is < 0.5 m
           front = valid & (np.abs(deg) <= 30)
           print("stop" if (ranges[front] < 0.5).any() else "go")

        .. output: exercises/ex3_scan.txt all

        .. code-block:: text

           3 8
           left: 2.330 m at 119 deg
           behind: 1.767 m, 60 readings
           go

        1. As in the lecture: ``np.asarray()``, then
           ``angle_min + np.arange(n) * angle_increment``, then the
           valid mask. Converting the angles to degrees once makes the
           sector tests easy to read.
        2. ``np.isinf()`` is ``True`` for both ``inf`` and ``-inf``.
           This scan has 8 ``inf`` readings (the doorway) and no
           ``-inf``.
        3. On the left, the nearest wall is ``x = 0``, 2.0 m from the
           robot. At 119° in the robot's frame, the ray points at
           119° + 30° = 149° in the room, 31° from the wall's normal,
           so the wall is 2.0 / cos 31° = 2.33 m away. The output agrees
           to the noise.
        4. 60 readings, all valid. Readings 0 to 30 (−180° to −150°)
           and 331 to 359 (151° to 179°) pass the test. Reading 330
           should be at 150°, but its computed angle is
           149.99999999999997°, so ``>= 150`` leaves it out. For the
           same reason, task 3's sector has 59 readings, not 61: the
           angles of readings 240 and 300 come out as
           59.999999999999986° and 120.00000000000001°.
        5. The nearest valid reading in front is the pillar, 2.379 m
           away (the lecture's ``front`` result), so the answer is
           "go". ``.any()`` turns the mask into one answer.


.. dropdown:: Exercise 4: The Scan in the World Frame (After Class)
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Use several sections of the lecture in one program: masks, a
    rotation, broadcasting, and a Matplotlib plot.

    **Tasks**

    .. code-block:: python

       # Use make_scan(), POSE, ROOM_W_M, and ROOM_H_M from scan_data.py,
       # rot() from geometry.py, and save() from plots.py.
       # 1. Turn the valid readings into points in the world frame, and
       #    find the nearest valid point.
       # 2. Plot the points, the walls of the room as a closed line, and
       #    the robot as a dot with an arrow for its heading. Label both
       #    axes and use equal scales.
       # 3. Mark the nearest valid point, and save the figure as
       #    plots/scan_world.png.
       # 4. Count the points within 0.05 m of a wall. Which points are
       #    the others?

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex4_plot.py all

        .. code-block:: python

           import matplotlib.pyplot as plt
           import numpy as np
           from geometry import rot
           from plots import BLUE, GRAY, ORANGE, save
           from scan_data import POSE, ROOM_H_M, ROOM_W_M, make_scan

           scan = make_scan()
           ranges = np.asarray(scan.ranges)
           angles = scan.angle_min + np.arange(ranges.size) * scan.angle_increment
           valid = (ranges >= scan.range_min) & (ranges <= scan.range_max)

           # 1. The valid readings as points in the world frame
           r, a = ranges[valid], angles[valid]
           points = np.column_stack([r * np.cos(a), r * np.sin(a)])
           x0, y0, theta = POSE
           world = points @ rot(theta).T + [x0, y0]
           nearest = world[r.argmin()]
           print(world.shape, np.round(nearest, 3))

           # 2 and 3. Points, walls, robot, nearest point; saved as a PNG
           plt.rcParams["font.size"] = 9
           fig, ax = plt.subplots(figsize=(3.6, 2.9), layout="constrained")
           walls_x = [0, ROOM_W_M, ROOM_W_M, 0, 0]
           walls_y = [0, 0, ROOM_H_M, ROOM_H_M, 0]
           ax.plot(walls_x, walls_y, color=GRAY, linewidth=1)
           ax.plot(world[:, 0], world[:, 1], ".", color=BLUE, markersize=3)
           ax.plot(*nearest, "o", color=ORANGE, markersize=6)
           ax.plot(x0, y0, "o", color="black", markersize=4)  # The robot
           heading = [x0 + 0.8 * np.cos(theta), y0 + 0.8 * np.sin(theta)]
           arrow = {"arrowstyle": "-|>", "color": "black"}
           ax.annotate("", heading, (x0, y0), arrowprops=arrow)
           ax.set_xlabel("x, world frame (m)")
           ax.set_ylabel("y, world frame (m)")
           ax.set_aspect("equal")
           save(fig, "scan_world")

           # 4. How many points lie within 0.05 m of a wall?
           wx, wy = world[:, 0], world[:, 1]
           gap = np.min(np.abs([wx, wx - ROOM_W_M, wy, wy - ROOM_H_M]), axis=0)
           print((gap < 0.05).sum(), "of", len(world), "points")

        .. output: exercises/ex4_plot.txt all short

        .. code-block:: text

           .../site-packages/matplotlib/projections/__init__.py:63: UserWarning: Unable to import Axes3D. This may be due to multiple versions of Matplotlib being installed (e.g. as a system package and as a pip package). As a result, the 3D projection is not available.
             warnings.warn("Unable to import Axes3D. This may be due to multiple versions of "
           (349, 2) [2.085 0.023]
           saved plots/scan_world.png
           336 of 349 points

        .. figure:: /_static/images/l5/scan_world.png
           :alt: A plot in the world frame, x across and y up, in meters, at equal scales. A gray rectangle from (0, 0) to (8, 6) marks the walls. Blue dots, the valid readings, lie on the walls, with a gap in the top wall from x = about 4.9 to 6.0 m (the doorway) and a short arc of dots near (4.3, 2.0) (the pillar). A black dot at (2.0, 1.0) with a short black arrow pointing up and to the right at 30 degrees is the robot and its heading. An orange dot at about (2.1, 0.0), on the bottom wall, is the nearest valid reading.
           :width: 60%

           The saved figure, ``plots/scan_world.png``.

        1. ``r.argmin()`` works on the valid readings only, so no
           ``np.where()`` is needed: the nearest point is (2.085, 0.023),
           on the wall y = 0, as in the lecture.
        2. ``walls_x`` and ``walls_y`` go around the rectangle and back
           to the start, so the line closes. ``ax.annotate("", heading,
           (x0, y0), ...)`` draws an arrow from the robot to a point
           0.8 m ahead of it.
        3. In the world frame the room's walls are straight along x and
           y: the scan matches the room that ``scan_data.py``
           describes, with the gap of the doorway in the top wall.
        4. For each point, ``gap`` is its distance to the nearest of the
           four wall lines. 336 of the 349 points are within 0.05 m; the
           other 13 are on the pillar.
