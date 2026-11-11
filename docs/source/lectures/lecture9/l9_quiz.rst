====================================================
Quiz
====================================================

This self-check quiz covers :doc:`Lecture 9 <l9_lecture>`.

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

   Lecture 8's battery publisher calls
   ``self.create_publisher(BatteryState, 'battery', 10)``. It is started
   with ``--ros-args -r __ns:=/robot1``. What is the full name of its
   topic?

   A. ``/battery``
   B. ``/robot1/battery``
   C. ``/robot1/battery_publisher/battery``
   D. ``battery``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``/robot1/battery``

   *Explanation:* ``'battery'`` is a relative name, so the namespace is put in front of it. ``ros2 topic list`` showed ``/robot1/battery`` and ``/robot2/battery`` ("A Namespace Is a Prefix for Names").


.. admonition:: Question 2
   :class: hint

   You run ``ros2 run demo_nodes_py listener -r __node:=fan``, without
   ``--ros-args``. What happens in Jazzy?

   A. An error: ``-r`` is not an argument of the listener
   B. The node is named ``listener``: the rule is ignored
   C. The node is named ``fan``, after a warning that this syntax is deprecated
   D. The node is named ``fan``, and nothing else is printed

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**

   *Explanation:* rcl still reads a rule written without ``--ros-args``, and warns: "This syntax is deprecated. Use ``'--ros-args --remap __node:=fan'`` instead." (checked by running it; "The ROS Arguments of a Node"). Write ROS arguments after ``--ros-args``, as the docs ask.


.. admonition:: Question 3
   :class: hint

   A node's ``main()`` calls ``rclpy.init(args=[])`` and then makes a
   node named ``scout``. It is run with
   ``--ros-args -r __node:=scout_1``. What is the node's name?

   A. ``/scout_1``
   B. ``/scout``
   C. An error: the arguments are not used
   D. ``/scout/scout_1``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``/scout``

   *Explanation:* A list passed to ``rclpy.init()`` is read instead of ``sys.argv``, and the empty list has no ROS arguments (measured on "From ros2 run to main(args=None)"). With ``args=None``, ``rclpy.init()`` reads ``sys.argv`` and the node is ``/scout_1``.


.. admonition:: Question 4
   :class: hint

   The battery monitor runs in the namespace ``/robot1`` with the rule
   ``-r battery:=status``. Which topic does it subscribe to?

   A. ``/robot1/battery``
   B. ``/status``
   C. ``/robot1/status``
   D. ``/battery``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``/robot1/status``

   *Explanation:* Both sides of the rule are relative names, so both are completed in the node's namespace: ``/robot1/battery`` becomes ``/robot1/status`` (checked with ``resolve_topic_name()``).


.. admonition:: Question 5
   :class: hint

   In a launch file, which argument of ``Node()`` gives the node the name
   ``robot2_watch`` in place of the name written in its code?

   A. ``executable='robot2_watch'``
   B. ``namespace='robot2_watch'``
   C. ``name='robot2_watch'``
   D. ``remappings=[('battery_monitor', 'robot2_watch')]``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``name='robot2_watch'``

   *Explanation:* Launch passes it as ``-r __node:=robot2_watch`` ("Renaming and Remapping in a Launch File"). ``executable`` names the program to run, and a namespace is a prefix, not a name.


.. admonition:: Question 6
   :class: hint

   ``robot_launch.py`` starts the monitor with
   ``condition=IfCondition(with_monitor)``. What happens with
   ``with_monitor:=yes``?

   A. The monitor starts: ``yes`` is true
   B. The monitor does not start: only ``true`` is true
   C. launch stops with an error; the publisher it had already started keeps running
   D. launch asks for a new value

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**

   *Explanation:* Only ``true``, ``1``, ``false``, and ``0`` (in any case) are valid. The error said so, ``ros2 launch`` ended with status 1, and ``ros2 node list`` still showed ``/robot1/battery_publisher`` ("What Counts as True").


.. admonition:: Question 7
   :class: hint

   A launch file includes ``robot_launch.py`` twice, without groups:
   first with ``robot:=robot2`` and ``with_monitor:=false``, then with
   ``robot:=robot1`` only. Which nodes run?

   A. ``/robot1/battery_publisher``, ``/robot1/battery_monitor``, ``/robot2/battery_publisher``
   B. ``/robot1/battery_publisher``, ``/robot2/battery_publisher``
   C. All four battery nodes
   D. Only ``/robot2/battery_publisher``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**

   *Explanation:* The value ``false`` set by the first include stays set for the second one, so robot1 has no monitor either ("Each Include in Its Own Group"). Putting each include in its own ``GroupAction`` keeps each include's arguments inside it.


.. admonition:: Question 8
   :class: hint

   A node has two timers and no callback group of its own. It is spun
   with ``MultiThreadedExecutor(num_threads=4)``. While one timer's
   callback runs, can the other one run?

   A. Yes: there are four threads
   B. No: both are in the node's default group, which is mutually exclusive
   C. Only if the callbacks are short
   D. Yes, but only on a computer with four CPUs

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**

   *Explanation:* ``multi_threaded`` printed the same late checks as ``single_threaded`` in all five runs ("Five Runs of Each"). The default group is a ``MutuallyExclusiveCallbackGroup``.


.. admonition:: Question 9
   :class: hint

   In ``reentrant_plan``, a plan is due every 1.0 s, takes 1.5 s, and is
   in a reentrant group. How many plans run at 2.2 s after the node was
   made?

   A. 0
   B. 1
   C. 2
   D. 3

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, 2

   *Explanation:* The plan of 1.00 s runs until 2.51 s (one run), and the plan of 2.00 s has started ("Reentrant: The Plan Overlaps Itself").


.. admonition:: Question 10
   :class: hint

   One thread that counts in a Python loop takes 0.20 s. About how long
   do two such threads take when started at the same time?

   A. 0.10 s
   B. 0.20 s
   C. 0.40 s
   D. 0.80 s

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, about 0.40 s

   *Explanation:* The global interpreter lock lets one thread run Python bytecode at a time, so the threads take turns: 2 × 0.20 = 0.40 s ("Python Threads Take Turns on Python Code", one run). Two threads that only wait take 0.20 s.


True/False
==========

.. admonition:: Question 11
   :class: hint

   In a workspace built with ``colcon build --symlink-install``, you must
   build again after every edit to an existing launch file.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* The installed launch file is a link that leads back to the file in ``src`` ("Installing the Launch Files"), so an edit is used at once. A new launch file does need a build (Exercise 4's solution).


.. admonition:: Question 12
   :class: hint

   ``rclpy.spin(node)`` runs the node's callbacks in a pool of threads.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* It adds the node to a ``SingleThreadedExecutor`` (``get_global_executor()`` in ``rclpy/__init__.py``), which runs callbacks in the thread that called ``spin()``.


.. admonition:: Question 13
   :class: hint

   The rule ``-r max_speed_mps:=top_speed`` renames the parameter
   ``max_speed_mps`` of the node.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* ``ros2 param list`` still showed ``max_speed_mps`` ("A Parameter Follows Its Node's Name"). A parameter moves with its node's name and namespace; a remapping rule does not rename it.


.. admonition:: Question 14
   :class: hint

   In a ``MultiThreadedExecutor``, a callback of one mutually exclusive
   group can run while a callback of another mutually exclusive group
   runs.

.. dropdown:: Answer
   :class-container: sd-border-success

   **True**

   *Explanation:* ``two_groups`` gives each timer its own group, and no safety check was late during the plans in all five runs ("Five Runs of Each").
