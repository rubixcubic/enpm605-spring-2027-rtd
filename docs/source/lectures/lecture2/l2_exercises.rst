====================================================
Python Exercises
====================================================

Four exercises on :doc:`Lecture 2 <l2_lecture>`. They are the exercises
from the slides. Run every program inside the course environment.

.. note::

   Try each exercise before you open its solution. Every output in a
   solution was printed by running the solution program.


----


.. dropdown:: Exercise 1: Operators
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Predict what floor division, modulo, power, and the logical operators
    return.

    **Tasks**

    Write down the output of each line **before** you run it. Then run it
    and explain every line you got wrong.

    .. code-block:: python

       # Arithmetic
       print(17 // 5, 17 % 5)
       print(-17 // 5, -17 % 5)
       print(2**0.5)
       print(-3**2)

       # Logical operators with values that are not bool
       print(0 or "default")
       print("lidar" and "camera")
       print(not [])

       # Chained comparisons
       x = 15
       print(10 < x < 20)
       print(10 < x > 20)

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code-block:: text

           3 2
           -4 3
           1.4142135623730951
           -9
           default
           camera
           True
           True
           False

        1. ``17 // 5`` is 3 (5 fits 3 times in 17), and 17 is 2 more than
           15, so ``17 % 5`` is 2.
        2. ``-17 / 5`` is ``-3.4``. Rounded down, ``-17 // 5`` is ``-4``.
           5 times ``-4`` is ``-20``, and ``-17`` is 3 more than ``-20``,
           so ``-17 % 5`` is 3.
        3. ``2**0.5`` is the square root of 2, a ``float``.
        4. ``**`` runs before the minus sign: ``-3**2`` is ``-(3**2)``,
           which is ``-9``.
        5. ``0`` is falsy, so ``or`` returns its right operand,
           ``"default"``.
        6. ``"lidar"`` is truthy, so ``and`` returns its right operand,
           ``"camera"``.
        7. ``[]`` is falsy, so ``not []`` is ``True``.
        8. ``10 < x < 20`` means ``10 < x and x < 20``: both are true.
        9. ``10 < x > 20`` means ``10 < x and x > 20``: the second part,
           ``15 > 20``, is false.


.. dropdown:: Exercise 2: Strings
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Index, slice, split, and format strings.

    **Tasks**

    Part A: predict, then run.

    .. code-block:: python

       text = "Learn Python, be happy!"
       print(text[6:12])
       print(text[-6:])
       print(text[::3])

    Part B: with ``text = "Learn Python, be happy!"``:

    1. Extract ``"Python"`` with positive indices only.
    2. Extract ``"Python"`` with negative indices only.
    3. Get ``"nohtyP"`` with one slice.
    4. Reverse the whole string.

    Part C: with ``topic = "/robot3/scan"``, print the robot's namespace,
    ``"robot3"``, in two ways: once with ``split()``, once with a slice.
    Then print the f-string ``robot3 publishes scan``.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code-block:: python

           # Part A
           text = "Learn Python, be happy!"
           print(text[6:12])
           print(text[-6:])
           print(text[::3])

           # Part B
           print(text[6:12])  # 1. Positive indices
           print(text[-17:-11])  # 2. Negative indices
           print(text[11:5:-1])  # 3. "nohtyP"
           print(text[::-1])  # 4. The whole string, reversed

           # Part C
           topic = "/robot3/scan"
           print(topic.split("/")[1])  # With split()
           print(topic[1:7])  # With a slice
           namespace = topic[1:7]
           data = topic[8:]
           print(f"{namespace} publishes {data}")

        .. code-block:: text

           Python
           happy!
           LrPh,eay
           Python
           Python
           nohtyP
           !yppah eb ,nohtyP nraeL
           robot3
           robot3
           robot3 publishes scan

        - ``"Python"`` starts at index 6 (after ``Learn`` and a space)
          and has 6 characters, so it ends before index 12.
        - The string has 23 characters, so index 6 is also index
          6 - 23 = -17, and index 12 is -11.
        - ``text[11:5:-1]`` starts at the ``n`` (index 11) and walks back
          to the ``P`` (index 6), stopping before index 5.
        - ``topic.split("/")`` gives ``['', 'robot3', 'scan']``: the
          string starts with ``/``, so item 0 is empty and item 1 is the
          namespace.


.. dropdown:: Exercise 3: Control Flow
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Combine conditions with ``and`` and ``or``, and extend a ``match``
    statement.

    **Tasks**

    Part A: write a program that prints ``Leap year`` or
    ``Not a leap year``. A year is a leap year if it is divisible by 4
    and not divisible by 100, or if it is divisible by 400. Test with
    2024, 2026, 1900, and 2000. Hint: ``year % 4 == 0`` is ``True`` when
    ``year`` is divisible by 4.

    Part B: copy ``lecture2/match_words_demo.py`` and extend it:

    1. Accept ``"turn left"`` and ``"turn right"`` only: any other
       direction goes to ``_``.
    2. Add a command ``"speed 0.8"`` that prints
       ``Speed set to 0.8 m/s``, but only for speeds from 0 to 1.0 m/s
       (limits chosen for this exercise).

    .. dropdown:: Solution
        :class-container: sd-border-success

        **Part A.** The solution reads the year from the command line, as
        ``match_demo.py`` does, so each test is one run.

        .. code-block:: python

           """Run with a year:  uv run ex3_leap_year.py 2024"""

           import sys

           year = int(sys.argv[1])

           if (year % 4 == 0 and year % 100 != 0) or year % 400 == 0:
               print(f"{year}: Leap year")
           else:
               print(f"{year}: Not a leap year")

        Four separate runs:

        .. code-block:: text

           2024: Leap year
           2026: Not a leap year
           1900: Not a leap year
           2000: Leap year

        1900 is divisible by 4 and by 100, and not by 400, so it is not a
        leap year. 2000 is divisible by 400, so it is.

        **Part B.**

        .. code-block:: python

           """Run with a command in quotes:  uv run ex3_match_words.py "speed 0.8" """

           import sys

           words = sys.argv[1].split()

           match words:
               case ["move", distance] if float(distance) > 0:
                   print(f"Moving {distance} m")
               case ["turn", "left" | "right" as direction]:
                   print(f"Turning {direction}")
               case ["speed", value] if 0 <= float(value) <= 1.0:
                   print(f"Speed set to {value} m/s")
               case ["stop"]:
                   print("Stopping")
               case _:
                   print(f"Cannot run: {words}")

        Six separate runs:

        .. list-table::
           :header-rows: 1
           :class: compact-table

           * - Command in quotes
             - Output
           * - ``"turn left"``
             - ``Turning left``
           * - ``"turn right"``
             - ``Turning right``
           * - ``"turn up"``
             - ``Cannot run: ['turn', 'up']``
           * - ``"speed 0.8"``
             - ``Speed set to 0.8 m/s``
           * - ``"speed 1.5"``
             - ``Cannot run: ['speed', '1.5']``
           * - ``"move 2.5"``
             - ``Moving 2.5 m``

        ``"left" | "right" as direction`` matches either word and binds
        the one that matched to ``direction``. The guard on ``speed``
        is a chained comparison.


.. dropdown:: Exercise 4: Robot Status Monitor
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Use every section of the lecture in one small package.

    **Tasks**

    Start from these parameters (chosen for this exercise):

    .. code-block:: python

       robot_name = "waffle_01"
       battery_pct = 65
       speed_mps = 0.8
       status_log = "IDLE:MOVING:CHARGING:MOVING:IDLE"

    1. Print with an f-string: ``Robot waffle_01 | Battery: 65%``.
    2. Classify the battery with ``if``/``elif``/``else``: 80 or more is
       ``OK``, 50 to 79 is ``LOW``, 20 to 49 is ``WARNING``, below 20 is
       ``CRITICAL``.
    3. With string methods: count how many times ``MOVING`` appears in
       ``status_log``, split ``status_log`` at ``:`` into a list, and
       check whether ``status_log`` ends with ``IDLE``.
    4. With a slice, extract the first status in ``status_log``.
    5. Print: ``waffle_01 | Battery: LOW | Speed: 0.80 m/s | States: 5``.

    Put the program in a module of a package ``monitor/``, with a main
    guard, and import it from a file next to the package.

    .. dropdown:: Solution
        :class-container: sd-border-success

        The files, printed by ``tree``:

        .. code-block:: text

           .
           ├── monitor
           │   ├── __init__.py
           │   └── status.py
           └── run_monitor.py

        .. code-block:: python
           :caption: monitor/status.py

           """Print a robot's status from its parameters."""

           # Robot parameters (chosen for this exercise)
           robot_name = "waffle_01"
           battery_pct = 65
           speed_mps = 0.8
           status_log = "IDLE:MOVING:CHARGING:MOVING:IDLE"

           # 2. Classify the battery
           if battery_pct >= 80:
               battery_level = "OK"
           elif battery_pct >= 50:
               battery_level = "LOW"
           elif battery_pct >= 20:
               battery_level = "WARNING"
           else:
               battery_level = "CRITICAL"

           # 3. String methods
           moving_count = status_log.count("MOVING")
           states = status_log.split(":")
           ends_idle = status_log.endswith("IDLE")

           # 4. A slice: the first status ends before the first ":"
           first_status = status_log[: status_log.find(":")]

           if __name__ == "__main__":
               print(f"Robot {robot_name} | Battery: {battery_pct}%")  # 1.
               print(f"Battery level: {battery_level}")
               print(f"MOVING appears {moving_count} times")
               print(states)
               print(f"Ends with IDLE: {ends_idle}")
               print(f"First status: {first_status}")
               # 5. len() counts the items of the list
               print(
                   f"{robot_name} | Battery: {battery_level} | Speed: {speed_mps:.2f} m/s"
                   f" | States: {len(states)}"
               )

        Run directly, ``uv run monitor/status.py``:

        .. code-block:: text

           Robot waffle_01 | Battery: 65%
           Battery level: LOW
           MOVING appears 2 times
           ['IDLE', 'MOVING', 'CHARGING', 'MOVING', 'IDLE']
           Ends with IDLE: True
           First status: IDLE
           waffle_01 | Battery: LOW | Speed: 0.80 m/s | States: 5

        .. code-block:: python
           :caption: run_monitor.py

           """Import the monitor package from the folder next to it."""

           from monitor import status

           print(f"{status.robot_name} | Battery: {status.battery_level}")

        ``uv run run_monitor.py`` prints one line. The guarded lines of
        ``status.py`` do not run, because ``__name__`` is
        ``"monitor.status"`` there:

        .. code-block:: text

           waffle_01 | Battery: LOW

        - ``status_log.find(":")`` is 4, the index of the first ``:``, so
          the slice ``status_log[:4]`` is ``IDLE``.
        - Two string literals next to each other inside parentheses are
          joined into one string. The last ``print()`` uses that to keep
          each line short.
