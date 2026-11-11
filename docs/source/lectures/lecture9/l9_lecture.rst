====================================================
Lecture
====================================================

This lecture uses two repositories:

- The ROS repository,
  `enpm605-spring-2027-ros <https://github.com/rubixcubic/enpm605-spring-2027-ros>`_,
  holds two new packages in ``lecture9/``: ``competition_bringup``
  (launch files only) and ``competition_executors`` (a robot controller
  run by different executors). They use Lecture 8's
  ``competition_nodes``. Run ``git pull`` in the clone in
  ``~/ros_ws/src``, then build the workspace with
  ``colcon build --symlink-install``.
- The course repository,
  `enpm605-spring-2027-python <https://github.com/rubixcubic/enpm605-spring-2027-python>`_,
  holds two plain Python programs in ``lecture9/``. Run them from its
  root folder. ``gil_demo.py`` needs nothing else
  (``uv run lecture9/gil_demo.py``); ``ros_args_demo.py`` needs ROS 2
  sourced.

Every output on this page was printed by running the code with ROS 2
Jazzy on Ubuntu 24.04, with ``ROS_DOMAIN_ID=77`` and
``ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST``. Log lines were recorded
without their time stamp (Lecture 8, "Reading a Log Line"), and a node
that runs until Ctrl+C shows its first lines only. In paths, ``...``
marks a path shortened to fit; in ``ros2 launch`` output, process IDs and
the dated log folder are replaced by ``...``. The periods and durations
of the robot controller (1.0 s, 0.1 s, 0.5 s, 1.5 s) are chosen for the
example. Times measured on one run say so; the executor outputs were
compared over five runs of each (`Five Runs of Each`_).


ROS Arguments
====================================================

Where Lecture 8 Stopped
-----------------------

In Lecture 8, each node had its own terminal and its own ``ros2 run``,
and ran under the name written in its code. Two copies of one node
therefore had one name. With two listeners, each started with
``ros2 run demo_nodes_py listener``:

.. code-block:: bash

   ros2 node list

.. output: same_name_list.txt all

.. code-block:: text

   WARNING: Be aware that there are nodes in the graph that share an exact name, which can have unintended side effects.
   /listener
   /listener

The same problem comes with topics: two robots running Lecture 8's
battery publisher would both publish on ``/battery``. This lecture
changes names when a node starts, without changing its code; starts many
nodes with one command; and chooses how a node's callbacks run.

The ROS Arguments of a Node
---------------------------

The ROS 2 how-to "Passing ROS arguments to nodes via the command-line":
"All ROS-specific arguments have to be specified after a ``--ros-args``
flag". They configure "the name/namespace of the node, topic/service
names used, and parameters on the node."

.. list-table::
   :header-rows: 1
   :widths: 40 60
   :class: compact-table

   * - Argument
     - What it changes (section)
   * - ``-r __node:=<name>``
     - The node's name (`Remapping`_)
   * - ``-r __ns:=/<namespace>``
     - The node's namespace (`Namespaces`_)
   * - ``-r <old>:=<new>``
     - A topic or service name (`Remapping`_)
   * - ``-p <name>:=<value>``
     - The value of a parameter (`Remapping`_)

.. code-block:: bash

   ros2 run competition_nodes minimal_node --ros-args -r __node:=hello_node

.. output: minimal_renamed.txt all

.. code-block:: text

   [INFO] [hello_node]: Hello from ROS 2

Lecture 8's ``minimal_node.py`` writes the name ``minimal_node``; this
run's node is ``hello_node``, and the code did not change.

Without ``--ros-args``, Jazzy still reads the rule, but warns that the
syntax is deprecated:

.. code-block:: bash

   ros2 run competition_nodes minimal_node -r __node:=hello_node

.. output: deprecated_remap.txt all

.. code-block:: text

   [WARN] [rcl]: Found remap rule '__node:=hello_node'. This syntax is deprecated. Use '--ros-args --remap __node:=hello_node' instead.
   [INFO] [hello_node]: Hello from ROS 2

Always write ``--ros-args``.

What rclpy.init() Reads
-----------------------

.. code: lecture9/ros_args_demo.py all

.. code-block:: python
   :caption: lecture9/ros_args_demo.py

   """What rclpy.init() reads from the command line.

   Needs ROS 2 Jazzy sourced. Run it with arguments of its own and ROS
   arguments, for example:

       python3 lecture9/ros_args_demo.py --laps 3 --ros-args -r __node:=scout_1
   """

   import sys

   import rclpy
   from rclpy.utilities import remove_ros_args

   print("sys.argv[1:]:", sys.argv[1:])
   rclpy.init(args=None)  # None: rclpy reads sys.argv itself
   node = rclpy.create_node("scout")  # The name written in the code
   print("node name:", node.get_fully_qualified_name())
   print("program's own arguments:", remove_ros_args(sys.argv)[1:])
   node.destroy_node()
   rclpy.shutdown()

.. code-block:: bash

   python3 lecture9/ros_args_demo.py --laps 3 --ros-args -r __node:=scout_1

.. output: ros_args_demo.txt all

.. code-block:: text

   sys.argv[1:]: ['--laps', '3', '--ros-args', '-r', '__node:=scout_1']
   node name: /scout_1
   program's own arguments: ['--laps', '3']

Run with no arguments:

.. output: ros_args_none.txt all

.. code-block:: text

   sys.argv[1:]: []
   node name: /scout
   program's own arguments: []

- ``sys.argv`` holds every word after the program's name. With
  ``args=None``, ``rclpy.init()`` reads it and keeps what follows
  ``--ros-args``.
- ``remove_ros_args()`` gives the program's own arguments, here
  ``--laps 3``, for the program to read.

From ros2 run to main(args=None)
--------------------------------

1. ``ros2 run <package> <executable> <words>`` runs the executable with
   the words: they are its ``sys.argv``, as for ``python3`` in
   `What rclpy.init() Reads`_.
2. The executable that colcon wrote calls ``main()`` with no argument
   (Lecture 8, "What Each Call in minimal_node.py Does"), so ``args`` is
   ``None``.
3. ``rclpy.init(args=None)`` reads ``sys.argv``: ``rclpy/context.py``
   uses ``args if args is not None else sys.argv``.
4. The node is made after ``rclpy.init()``, so the name and the
   namespace on the command line replace the ones in the code.

A list passed on is read instead of ``sys.argv``. With
``rclpy.init(args=[])``, the same
``--ros-args -r __node:=scout_1`` left the node named:

.. output: init_empty_args.txt all

.. code-block:: text

   /scout


Namespaces
====================================================

Two Robots, One Topic Name
--------------------------

Terminal 1 first, then terminals 2 and 3 at the same time:

.. code-block:: bash

   ros2 run competition_nodes battery_monitor       # Terminal 1
   ros2 run competition_nodes battery_publisher     # Terminals 2 and 3

.. output: two_pubs_monitor.txt all

.. code-block:: text

   [INFO] [battery_monitor]: 16%: active
   [INFO] [battery_monitor]: 16%: active
   [INFO] [battery_monitor]: 14%: active
   [INFO] [battery_monitor]: 14%: active

.. code-block:: bash

   ros2 topic info /battery

.. output: two_pubs_info.txt all

.. code-block:: text

   Type: sensor_msgs/msg/BatteryState
   Publisher count: 2
   Subscription count: 1

Both publishers use the topic ``/battery``, so the monitor gets every
message of both (rule 4 of Lecture 8's "Four Rules of a Topic"). A
message does not say which robot sent it.

A Namespace Is a Prefix for Names
---------------------------------

Four terminals, a monitor and a publisher for each robot:

.. code-block:: bash

   ros2 run competition_nodes battery_monitor --ros-args -r __ns:=/robot1
   ros2 run competition_nodes battery_publisher --ros-args -r __ns:=/robot1
   ros2 run competition_nodes battery_monitor --ros-args -r __ns:=/robot2
   ros2 run competition_nodes battery_publisher --ros-args -r __ns:=/robot2

``ros2 node list``:

.. output: ns_node_list.txt all

.. code-block:: text

   /robot1/battery_monitor
   /robot1/battery_publisher
   /robot2/battery_monitor
   /robot2/battery_publisher

``ros2 topic list``:

.. output: ns_topic_list.txt all

.. code-block:: text

   /parameter_events
   /robot1/battery
   /robot2/battery
   /rosout

The design doc "Topic and Service name mapping to DDS": "A relative name
does not begin with a forward slash (/) and does respect the namespace
of the node which created them." The code says ``'battery'``, a relative
name, so robot1's topic is ``/robot1/battery``. ``/rosout`` and
``/parameter_events`` start with ``/``: no prefix.

Each Robot Has Its Own Topic
----------------------------

.. figure:: /_static/images/l9/namespaces.svg
   :alt: Two dashed boxes, one above the other, each a namespace. Tag 1: the box /robot1 holds the node battery_publisher, an arrow to the topic battery, and an arrow from battery to the node battery_monitor. Tag 2: the box /robot2 holds the same two nodes and the same topic.
   :width: 90%

   1: the namespace ``/robot1``; its nodes are
   ``/robot1/battery_publisher`` and ``/robot1/battery_monitor``, its
   topic ``/robot1/battery``. 2: the namespace ``/robot2``, the same code
   with the prefix ``/robot2``.

Each Robot's Monitor Hears Only Its Robot
-----------------------------------------

robot1's publisher and monitor, the first lines of each:

.. output: ns_publisher.txt all

.. code-block:: text

   [INFO] [robot1.battery_publisher]: Published 16%
   [INFO] [robot1.battery_publisher]: Published 14%
   [INFO] [robot1.battery_publisher]: Published 12%

.. output: ns_monitor.txt all

.. code-block:: text

   [INFO] [robot1.battery_monitor]: 16%: active
   [INFO] [robot1.battery_monitor]: 14%: active
   [INFO] [robot1.battery_monitor]: 12%: active

.. code-block:: bash

   ros2 topic info /robot1/battery

.. output: ns_topic_info.txt all

.. code-block:: text

   Type: sensor_msgs/msg/BatteryState
   Publisher count: 1
   Subscription count: 1

- Each level arrives once: robot2's messages are on
  ``/robot2/battery``, a topic robot1's monitor does not subscribe to.
- The logger name is the node's full name with dots:
  ``robot1.battery_monitor``.

Relative, Absolute, and Private Names
-------------------------------------

A node ``battery_monitor`` in ``/robot1`` completes three names
(``Node.resolve_topic_name()``, run in a short script):

.. output: names_resolved.txt all

.. code-block:: text

   battery    /robot1/battery
   /rosout    /rosout
   ~/status   /robot1/battery_monitor/status

- **Relative**: no ``/`` at the start. The design doc: it "does respect
  the namespace of the node which created them."
- **Absolute**: "An absolute name begins with a forward slash (/) and
  does not respect namespaces". Every node logs to the same ``/rosout``.
- **Private**: starts with ``~``, which "will be replaced with a
  namespace snippet that is a concatenation of the namespace for the
  node and the node name."

``ros2 node info /robot1/battery_monitor`` shows the three kinds at
work: its subscription is in the namespace, its log publisher is not,
and its parameter services start with the node's full name.

.. output: ns_node_info.txt 1-15

.. code-block:: text

   /robot1/battery_monitor
     Subscribers:
       /robot1/battery: sensor_msgs/msg/BatteryState
     Publishers:
       /parameter_events: rcl_interfaces/msg/ParameterEvent
       /rosout: rcl_interfaces/msg/Log
     Service Servers:
       /robot1/battery_monitor/describe_parameters: rcl_interfaces/srv/DescribeParameters
       /robot1/battery_monitor/get_parameter_types: rcl_interfaces/srv/GetParameterTypes
       /robot1/battery_monitor/get_parameters: rcl_interfaces/srv/GetParameters
       /robot1/battery_monitor/get_type_description: type_description_interfaces/srv/GetTypeDescription
       /robot1/battery_monitor/list_parameters: rcl_interfaces/srv/ListParameters
       /robot1/battery_monitor/set_parameters: rcl_interfaces/srv/SetParameters
       /robot1/battery_monitor/set_parameters_atomically: rcl_interfaces/srv/SetParametersAtomically
     Service Clients:


Remapping
====================================================

A Rule Replaces a Name When the Node Starts
-------------------------------------------

The design doc "Remapping Names": "Names are hard coded in ROS nodes, but
they can be changed at runtime through remapping. Without remapping
every instance of a node would require changes in code."

- A rule is ``match:=replacement``, given after ``-r``. Two names are
  special: ``__node`` (the node's name) and ``__ns`` (its namespace).
- The ROS 2 how-to: "these remappings are 'static' remappings, in that
  they apply for the lifetime of the node." To change a name again,
  restart the node.

Lecture 8's Exercise 1 ran two listeners with one name. Renamed with
``__node``, the second one is ``/listener2``, and the warning is gone:

.. code-block:: bash

   # Terminals 1 and 2
   ros2 run demo_nodes_py listener
   ros2 run demo_nodes_py listener --ros-args -r __node:=listener2
   # Terminal 3
   ros2 node list

.. output: renamed_list.txt all

.. code-block:: text

   /listener
   /listener2

Remapping a Topic Name
----------------------

Terminal 1: Lecture 8's monitor, renamed, watching robot2's battery:

.. code-block:: bash

   ros2 run competition_nodes battery_monitor --ros-args \
       -r __node:=robot2_watch -r battery:=/robot2/battery

.. output: watch_cli.txt all

.. code-block:: text

   [INFO] [robot2_watch]: 16%: active
   [INFO] [robot2_watch]: 14%: active
   [INFO] [robot2_watch]: 12%: active

Terminal 2: robot2's publisher; terminal 3: ``ros2 node info``:

.. code-block:: bash

   ros2 run competition_nodes battery_publisher --ros-args -r __ns:=/robot2
   ros2 node info /robot2_watch

.. output: watch_info.txt 1-6

.. code-block:: text

   /robot2_watch
     Subscribers:
       /robot2/battery: sensor_msgs/msg/BatteryState
     Publishers:
       /parameter_events: rcl_interfaces/msg/ParameterEvent
       /rosout: rcl_interfaces/msg/Log

The node is outside every namespace, so its ``'battery'`` would be
``/battery``. The rule replaces it with ``/robot2/battery``, and the code
is unchanged.

How the Rule Matched
--------------------

.. figure:: /_static/images/l9/remap.svg
   :alt: A dashed box, the namespace /robot2, holds the node battery_publisher with an arrow to the topic battery. Below the box, outside it, the node robot2_watch, tag 1, gets an arrow from battery. Next to that arrow, tag 2, the rule battery:=/robot2/battery.
   :width: 75%

   1: ``__node:=robot2_watch`` replaced the name in the code,
   ``battery_monitor``. 2: ``battery:=/robot2/battery``.

The left side of the rule is completed like the node's own names
(``/battery``), and every name of the node that is ``/battery`` becomes
``/robot2/battery``. The design doc calls this "Exact Relative Name
Replacement": "It works by first expanding the relative name and then
doing FQN replacement." FQN is a fully qualified name, the name that
starts with ``/``. Both sides are completed in the node's namespace: in
``/robot1``, the rule ``battery:=status`` makes the topic
``/robot1/status`` (quiz question 4).

Parameters: Settings That Belong to a Node
------------------------------------------

The ROS 2 documentation on parameters: "Parameters in ROS 2 are
associated with individual nodes. Parameters are used to configure nodes
at startup (and during runtime), without changing the code."

``parameter_blackboard``, a node of ``demo_nodes_cpp``, keeps any
parameter it is given:

.. code-block:: bash

   ros2 run demo_nodes_cpp parameter_blackboard --ros-args \
       -r __ns:=/robot1 -p max_speed_mps:=2.0
   ros2 param get /robot1/parameter_blackboard max_speed_mps

.. output: param_get.txt all

.. code-block:: text

   Double value is: 2.0

``-p name:=value`` sets a parameter's value when the node starts; 2.0 is
Lecture 7's scout speed. ``ros2 param get`` reads it by the node's full
name and the parameter's name. Your own nodes declare and read
parameters in Lecture 10.

A Parameter Follows Its Node's Name
-----------------------------------

The same node, with one more rule, ``-r max_speed_mps:=top_speed``:

.. code-block:: bash

   ros2 param list /robot1/parameter_blackboard

.. output: param_remap.txt all

.. code-block:: text

     max_speed_mps
     qos_overrides./parameter_events.publisher.depth
     qos_overrides./parameter_events.publisher.durability
     qos_overrides./parameter_events.publisher.history
     qos_overrides./parameter_events.publisher.reliability
     start_type_description_service
     use_sim_time

- The parameter is still ``max_speed_mps``: the rule did not rename it.
  The Jazzy how-to names what rules rename: "Names within a node (e.g.
  topics/services) can be remapped". The design doc's example: a rule
  remaps a namespace, "but the parameter name remains unchanged".
- A parameter is found by its node's full name, so namespaces and
  ``__node`` move it: here it is a parameter of
  ``/robot1/parameter_blackboard``.
- The node made the other parameters itself. Started without ``-p``, it
  lists them and no ``max_speed_mps``:

.. output: param_list_plain.txt all

.. code-block:: text

     qos_overrides./parameter_events.publisher.depth
     qos_overrides./parameter_events.publisher.durability
     qos_overrides./parameter_events.publisher.history
     qos_overrides./parameter_events.publisher.reliability
     start_type_description_service
     use_sim_time


Launch Files
====================================================

One Command for Many Nodes
--------------------------

The ROS 2 documentation on launch: "The launch system in ROS 2 is meant
to automate the running of many nodes with a single command. It helps
the user describe the configuration of their system and then executes it
as described."

- The configuration: "what programs to run, where to run them, what
  arguments to pass them", and the names of `Namespaces`_ and
  `Remapping`_.
- Launch also watches the processes it started: "monitoring the state of
  the processes launched, and reporting and/or reacting to changes".
- A launch file "can be written in XML, YAML, or Python". This course
  writes them in Python.
- Lecture 8's table, "What Splitting into Nodes Gives, and What It
  Costs", listed one command per node as a cost. Exercise 1 needs five
  terminals; a launch file needs one.

A Launch File Is a Python Module
--------------------------------

.. code: ros/lecture9/competition_bringup/launch/battery_launch.py all

.. code-block:: python
   :caption: competition_bringup/launch/battery_launch.py

   """Start the battery monitor and the battery publisher (Lecture 9)."""

   from launch import LaunchDescription
   from launch_ros.actions import Node


   def generate_launch_description():
       monitor = Node(package='competition_nodes',
                      executable='battery_monitor')
       publisher = Node(package='competition_nodes',
                        executable='battery_publisher')
       return LaunchDescription([monitor, publisher])

- The tutorial "Integrating launch files into ROS 2 packages": "Your
  launch file should define the ``generate_launch_description()``
  function which returns a ``launch.LaunchDescription()`` to be used by
  the ros2 launch verb."
- ``Node(...)`` is a launch **action**: a step the launch system carries
  out. This one starts an executable of a package, like ``ros2 run``.
- The list gives the order in which the processes start: the monitor
  first. The file name "needs to end with launch.py to be recognized and
  autocompleted by ros2 launch."

Installing the Launch Files
---------------------------

``competition_bringup`` was made with ``ros2 pkg create``, then two
lines were added to its ``setup.py``:

.. code: ros/lecture9/competition_bringup/setup.py all

.. code-block:: python
   :caption: competition_bringup/setup.py

   from glob import glob

   from setuptools import find_packages, setup

   package_name = 'competition_bringup'

   setup(
       name=package_name,
       version='0.0.0',
       packages=find_packages(exclude=['test']),
       data_files=[
           ('share/ament_index/resource_index/packages',
               ['resource/' + package_name]),
           ('share/' + package_name, ['package.xml']),
           ('share/' + package_name + '/launch', glob('launch/*')),
       ],
       install_requires=['setuptools'],
       zip_safe=True,
       maintainer='Zeid Kootbally',
       maintainer_email='zeidk@umd.edu',
       description='Lecture 9 launch files: robotics competition',
       license='Apache-2.0',
       extras_require={
           'test': [
               'pytest',
           ],
       },
       entry_points={
           'console_scripts': [
           ],
       },
   )

and its ``package.xml`` lists what launching needs:

.. code: ros/lecture9/competition_bringup/package.xml 3-23

.. code-block:: xml
   :caption: competition_bringup/package.xml, after its first two lines

   <package format="3">
     <name>competition_bringup</name>
     <version>0.0.0</version>
     <description>Lecture 9 launch files: robotics competition</description>
     <maintainer email="zeidk@umd.edu">Zeid Kootbally</maintainer>
     <license>Apache-2.0</license>

     <exec_depend>launch</exec_depend>
     <exec_depend>launch_ros</exec_depend>
     <exec_depend>competition_nodes</exec_depend>
     <exec_depend>demo_nodes_cpp</exec_depend>

     <test_depend>ament_copyright</test_depend>
     <test_depend>ament_flake8</test_depend>
     <test_depend>ament_pep257</test_depend>
     <test_depend>python3-pytest</test_depend>

     <export>
       <build_type>ament_python</build_type>
     </export>
   </package>

- The tutorial: "By convention, all launch files for a package are
  stored in the launch directory inside of the package." ``colcon
  build`` copies them to ``share/competition_bringup/launch`` in the
  install folder, where ``ros2 launch`` looks.
- ``<exec_depend>``: needed to run, not to build.
- With ``--symlink-install``, the installed launch files are links. The
  launch folder in the install folder, and where its
  ``battery_launch.py`` leads in the end:

.. output: launch_installed.txt all

.. code-block:: text

   battery_launch.py
   robot_launch.py
   settings_launch.py
   team_launch.py
   two_robots_launch.py
   watch_launch.py
   lecture9/competition_bringup/launch/battery_launch.py

So an edited launch file needs no new build. A new one does: ``glob()``
in ``setup.py`` runs at build time (Exercise 4's solution shows the
error).

Running a Launch File
---------------------

.. code-block:: bash

   ros2 launch competition_bringup battery_launch.py

.. output: launch_battery.txt all

.. code-block:: text

   [INFO] [launch]: All log files can be found below ~/.ros/log/...
   [INFO] [launch]: Default logging verbosity is set to INFO
   [INFO] [battery_monitor-1]: process started with pid [...]
   [INFO] [battery_publisher-2]: process started with pid [...]
   [battery_monitor-1] [INFO] [battery_monitor]: 16%: active
   [battery_publisher-2] [INFO] [battery_publisher]: Published 16%
   [battery_publisher-2] [INFO] [battery_publisher]: Published 14%
   [battery_monitor-1] [INFO] [battery_monitor]: 14%: active
   [battery_publisher-2] [INFO] [battery_publisher]: Published 12%
   [battery_monitor-1] [INFO] [battery_monitor]: 12%: active

- ``ros2 launch <package> <file>``. Each process gets a label, its
  executable and a number in start order: ``[battery_monitor-1]``. Its
  log lines follow the label.
- This is one run's first lines: the order of lines from two processes
  can change from run to run.

Ctrl+C Stops Every Node of the Launch
-------------------------------------

The last lines of the same run, after Ctrl+C:

.. output: launch_battery_end.txt all

.. code-block:: text

   [INFO] [battery_monitor-1]: process has finished cleanly [pid ...]
   [INFO] [battery_publisher-2]: process has finished cleanly [pid ...]

Ctrl+C in a terminal sends the signal ``SIGINT`` to the terminal's
**foreground process group**: every process of the command running in
that terminal. The nodes that launch starts are in launch's process group
(measured, while ``battery_launch.py`` ran):

.. output: launch_pgid.txt all

.. code-block:: text

   children: 2; in launch's process group: yes

So each node gets the ``SIGINT``; its first ``except`` of Lecture 8
catches it, and its ``finally`` cleans up. Ten more runs, recorded with
the other outputs (one recording), each stopped with one ``SIGINT`` to all
its processes:

.. output: launch_runs.txt all

.. code-block:: text

   run  1: first: 16%: active, clean exits: 2, errors: none
   run  2: first: 16%: active, clean exits: 2, errors: none
   run  3: first: 16%: active, clean exits: 2, errors: none
   run  4: first: 16%: active, clean exits: 2, errors: none
   run  5: first: 16%: active, clean exits: 2, errors: none
   run  6: first: 16%: active, clean exits: 2, errors: none
   run  7: first: 16%: active, clean exits: 2, errors: none
   run  8: first: 16%: active, clean exits: 2, errors: none
   run  9: first: 16%: active, clean exits: 2, errors: none
   run 10: first: 16%: active, clean exits: 2, errors: none

``code/launch_ctrl_c.sh`` in the slides repository stopped the same launch
the same way 40 times; the last line of its record:

.. output: launch_ctrl_c.txt 41

.. code-block:: text

   40 of 40 runs: both processes finished cleanly, no error

An Error Raised by the Shutdown Itself
--------------------------------------

The ``SIGINT`` can shut ROS 2 down while ``spin()`` starts its next wait,
or while a callback runs. ``spin()`` then raises
``rclpy._rclpy_pybind11.RCLError``, with one of two messages (each cut
after its first comma):

.. output: shutdown_errors.txt all

.. code-block:: text

   failed to initialize wait set: the given context is not valid, ...
   Failed to publish: publisher's context is invalid, ...

- With only the first ``except`` (Lecture 8's code before 2026-10-16),
  two such nodes under launch printed this traceback in 9 of 107 runs (the
  independent check) and in 3 of 40 runs (2026-10-16), and two
  ``ros2 run`` commands in 1 of 40 runs. The traceback of the first
  message, recorded on 2026-10-14 (folders shortened):

.. output: shutdown_traceback_wait.txt all

.. code-block:: text

   [battery_monitor-1] Traceback (most recent call last):
   [battery_monitor-1]   File ".../ros_ws/install/competition_nodes/lib/competition_nodes/battery_monitor", line 33, in <module>
   [battery_monitor-1]     sys.exit(load_entry_point('competition-nodes', 'console_scripts', 'battery_monitor')())
   [battery_monitor-1]              ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   [battery_monitor-1]   File ".../ros_ws/build/competition_nodes/competition_nodes/battery_monitor.py", line 31, in main
   [battery_monitor-1]     rclpy.spin(node)
   [battery_monitor-1]   File ".../rclpy/__init__.py", line 247, in spin
   [battery_monitor-1]     executor.spin_once()
   [battery_monitor-1]   File ".../rclpy/executors.py", line 926, in spin_once
   [battery_monitor-1]     self._spin_once_impl(timeout_sec)
   [battery_monitor-1]   File ".../rclpy/executors.py", line 907, in _spin_once_impl
   [battery_monitor-1]     handler, entity, node = self.wait_for_ready_callbacks(
   [battery_monitor-1]                             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   [battery_monitor-1]   File ".../rclpy/executors.py", line 877, in wait_for_ready_callbacks
   [battery_monitor-1]     return next(self._cb_iter)
   [battery_monitor-1]            ^^^^^^^^^^^^^^^^^^^
   [battery_monitor-1]   File ".../rclpy/executors.py", line 757, in _wait_for_ready_callbacks
   [battery_monitor-1]     wait_set = _rclpy.WaitSet(
   [battery_monitor-1]                ^^^^^^^^^^^^^^^
   [battery_monitor-1] rclpy._rclpy_pybind11.RCLError: failed to initialize wait set: the given context is not valid, either rcl_init() was not called or rcl_shutdown() was called., at ./src/rcl/wait.c:130
   [INFO] [battery_publisher-2]: process has finished cleanly [pid ...]
   [ERROR] [battery_monitor-1]: process has died [pid ..., exit code 1, cmd '.../ros_ws/install/competition_nodes/lib/competition_nodes/battery_monitor --ros-args'].

- The second ``except`` of Lecture 8's ``main()`` drops any error once
  ``rclpy.ok()`` is ``False``, and raises it again while ROS 2 still
  runs. With it: 0 tracebacks in 40 launch runs and in 40 ``ros2 run``
  runs, every exit code 0.
- Without the second ``except``, a run is clean with a chance of about
  1 - 9/107 = 0.916, so ten clean runs in a row still happen about 4
  times in 10 (0.916 to the power 10 is 0.42). Ten clean runs alone prove
  little; the 40-run counts are the evidence.

What a Node Action Can Set
--------------------------

.. list-table::
   :header-rows: 1
   :widths: 40 28 32
   :class: compact-table

   * - Argument of ``Node()``
     - Command line
     - Example
   * - ``package=``, ``executable=``
     - ``ros2 run``
     - ``battery_launch.py``
   * - ``namespace='robot1'``
     - ``-r __ns:=/robot1``
     - ``two_robots_launch.py``
   * - ``name='robot2_watch'``
     - ``-r __node:=...``
     - ``watch_launch.py``
   * - ``remappings=[(old, new)]``
     - ``-r old:=new``
     - ``watch_launch.py``
   * - ``parameters=[{name: value}]``
     - ``-p name:=value``
     - ``settings_launch.py``

Each argument does what the command line of `Namespaces`_ and
`Remapping`_ did; the next sections check that launch passes them to the
node as ROS arguments. A namespace in ``Node()`` may leave out the first
``/``: ``'robot1'`` gave ``/robot1``.

Two Robots from One Launch File
-------------------------------

.. code: ros/lecture9/competition_bringup/launch/two_robots_launch.py all

.. code-block:: python
   :caption: competition_bringup/launch/two_robots_launch.py

   """Two robots, each with its battery nodes in its own namespace (Lecture 9)."""

   from launch import LaunchDescription
   from launch_ros.actions import Node


   def generate_launch_description():
       nodes = []
       for robot in ['robot1', 'robot2']:  # A launch file is Python code
           nodes.append(Node(package='competition_nodes', namespace=robot,
                             executable='battery_publisher'))
           nodes.append(Node(package='competition_nodes', namespace=robot,
                             executable='battery_monitor'))
       return LaunchDescription(nodes)

``ros2 node list`` while it runs:

.. output: two_robots_nodes.txt all

.. code-block:: text

   /robot1/battery_monitor
   /robot1/battery_publisher
   /robot2/battery_monitor
   /robot2/battery_publisher

The command line of each process it started (``ps -o args``, with the
folders left out):

.. output: launch_ps.txt all

.. code-block:: text

   battery_publisher --ros-args -r __ns:=/robot1
   battery_monitor --ros-args -r __ns:=/robot1
   battery_publisher --ros-args -r __ns:=/robot2
   battery_monitor --ros-args -r __ns:=/robot2

A launch file is Python: a loop makes four ``Node`` actions, and launch
turned ``namespace=`` into ``-r __ns:=``.

Renaming and Remapping in a Launch File
---------------------------------------

.. code: ros/lecture9/competition_bringup/launch/watch_launch.py all

.. code-block:: python
   :caption: competition_bringup/launch/watch_launch.py

   """Watch robot2's battery from outside its namespace (Lecture 9)."""

   from launch import LaunchDescription
   from launch_ros.actions import Node


   def generate_launch_description():
       watch = Node(
           package='competition_nodes',
           executable='battery_monitor',
           name='robot2_watch',  # Replaces the name written in the code
           remappings=[('battery', '/robot2/battery')],
       )
       return LaunchDescription([watch])

Its process (``ps``, one ROS argument per line), and its first lines,
while ``two_robots_launch.py`` runs:

.. output: watch_ps.txt all

.. code-block:: text

   battery_monitor --ros-args
       -r __node:=robot2_watch
       -r battery:=/robot2/battery

.. output: watch_launch.txt all

.. code-block:: text

   [battery_monitor-1] [INFO] [robot2_watch]: 16%: active
   [battery_monitor-1] [INFO] [robot2_watch]: 14%: active
   [battery_monitor-1] [INFO] [robot2_watch]: 12%: active

The same rules as in `Remapping a Topic Name`_, now in a file:
``remappings`` is a list of pairs, the name in the code and its
replacement.

A Parameter Set in a Launch File
--------------------------------

.. code: ros/lecture9/competition_bringup/launch/settings_launch.py all

.. code-block:: python
   :caption: competition_bringup/launch/settings_launch.py

   """A settings node for robot1, with a parameter set at start (Lecture 9)."""

   from launch import LaunchDescription
   from launch_ros.actions import Node


   def generate_launch_description():
       settings = Node(
           package='demo_nodes_cpp',
           executable='parameter_blackboard',  # Accepts any parameter
           namespace='robot1',
           parameters=[{'max_speed_mps': 2.0}],  # Lecture 7's scout speed
       )
       return LaunchDescription([settings])

.. code-block:: bash

   ros2 launch competition_bringup settings_launch.py         # Terminal 1
   ros2 param get /robot1/parameter_blackboard max_speed_mps  # Terminal 2

.. output: settings_param.txt all

.. code-block:: text

   Double value is: 2.0

.. output: settings_ps.txt all

.. code-block:: text

   parameter_blackboard --ros-args
       -r __ns:=/robot1
       --params-file /tmp/launch_params_...

``parameters`` is a list of dictionaries. Launch wrote them to a file and
passed it with ``--params-file`` (the ``ps`` lines; the file's random
name is replaced by ``...``).


Arguments, Conditions, Groups, Includes
====================================================

A Launch Argument Is Declared, Then Read
----------------------------------------

.. code: ros/lecture9/competition_bringup/launch/robot_launch.py all

.. code-block:: python
   :caption: competition_bringup/launch/robot_launch.py

   """One robot: its battery nodes in a namespace you choose (Lecture 9)."""

   from launch import LaunchDescription
   from launch.actions import DeclareLaunchArgument, GroupAction
   from launch.conditions import IfCondition
   from launch.substitutions import LaunchConfiguration
   from launch_ros.actions import Node, PushROSNamespace


   def generate_launch_description():
       robot = LaunchConfiguration('robot')
       with_monitor = LaunchConfiguration('with_monitor')
       return LaunchDescription([
           DeclareLaunchArgument('robot', default_value='robot1',
                                 description='Namespace of the robot'),
           DeclareLaunchArgument('with_monitor', default_value='true',
                                 description='Start the battery monitor'),
           GroupAction([
               PushROSNamespace(robot),  # For every node in this group
               Node(package='competition_nodes',
                    executable='battery_publisher'),
               Node(package='competition_nodes',
                    executable='battery_monitor',
                    condition=IfCondition(with_monitor)),
           ]),
       ])

- ``DeclareLaunchArgument()``: the argument's name, its default value,
  and a description. A value is always a string.
- ``LaunchConfiguration('robot')`` is a **substitution**. The tutorial
  "Using substitutions": "Substitutions are variables that are only
  evaluated during execution of the launch description".
- So ``robot`` holds no value yet when ``generate_launch_description()``
  returns; launch reads the value later, when it starts the nodes.

Seeing and Setting Launch Arguments
-----------------------------------

.. code-block:: bash

   ros2 launch competition_bringup robot_launch.py --show-args

.. output: show_args.txt all

.. code-block:: text

   Arguments (pass arguments as '<name>:=<value>'):

       'robot':
           Namespace of the robot
           (default: 'robot1')

       'with_monitor':
           Start the battery monitor
           (default: 'true')

.. code-block:: bash

   ros2 launch competition_bringup robot_launch.py robot:=robot2
   ros2 node list  # In a second terminal

.. output: robot2_nodes.txt all

.. code-block:: text

   /robot2/battery_monitor
   /robot2/battery_publisher

``name:=value`` after the file name sets an argument; an argument not
given keeps its default.

A Group with Its Own Namespace, and a Condition
-----------------------------------------------

- ``GroupAction``: a list of actions handled together.
  ``PushROSNamespace(robot)`` gives every node after it in the group the
  namespace. The tutorial "Managing large projects": it "has to be the
  first action in the list for the following actions to apply the
  namespace."
- ``condition=IfCondition(with_monitor)``: the monitor starts only if
  the argument is true. ``UnlessCondition`` does the opposite.

What Counts as True
-------------------

.. code-block:: bash

   ros2 launch competition_bringup robot_launch.py with_monitor:=false
   ros2 node list

.. output: no_monitor_nodes.txt all

.. code-block:: text

   /robot1/battery_publisher

With ``with_monitor:=yes``:

.. output: bad_condition.txt all

.. code-block:: text

   [INFO] [launch]: All log files can be found below ~/.ros/log/...
   [INFO] [launch]: Default logging verbosity is set to INFO
   [ERROR] [launch]: Caught exception in launch (see debug for traceback): invalid condition expression, expected one of [true, 1, false, 0] but got 'yes', expanded from '<launch.substitutions.launch_configuration.LaunchConfiguration object at 0x...>'
   ros2 launch exit status: 1

``ros2 node list`` a few seconds after ``ros2 launch`` ended:

.. output: bad_condition_after.txt all

.. code-block:: text

   /robot1/battery_publisher

1. launch's source (``evaluate_condition_expression()``): the value is
   stripped and lowercased; ``true`` and ``1`` are true, ``false`` and
   ``0`` are false. Any other value is an error.
2. The publisher had started before launch read the condition, and it
   kept running after launch ended. ``ros2 launch`` printed no process
   ID for it: find it with ``ps -ef | grep battery_publisher``, and stop it
   with ``kill`` and the process ID that shows.

Including a Launch File
-----------------------

.. code: ros/lecture9/competition_bringup/launch/team_launch.py all

.. code-block:: python
   :caption: competition_bringup/launch/team_launch.py

   """A team of two robots: robot_launch.py, included twice (Lecture 9)."""

   from launch import LaunchDescription
   from launch.actions import GroupAction, IncludeLaunchDescription
   from launch.substitutions import PathJoinSubstitution
   from launch_ros.substitutions import FindPackageShare


   def generate_launch_description():
       robot_launch = PathJoinSubstitution([
           FindPackageShare('competition_bringup'), 'launch',
           'robot_launch.py'])
       # Each include is in a group, so its arguments stay in that group
       robot1 = GroupAction([IncludeLaunchDescription(
           robot_launch, launch_arguments={'robot': 'robot1'}.items())])
       robot2 = GroupAction([IncludeLaunchDescription(
           robot_launch, launch_arguments={
               'robot': 'robot2', 'with_monitor': 'false'}.items())])
       return LaunchDescription([robot2, robot1])

- ``FindPackageShare('competition_bringup')``: the package's ``share``
  folder in the install folder. ``PathJoinSubstitution`` adds
  ``launch/robot_launch.py`` to it.
- ``IncludeLaunchDescription``: the actions of ``robot_launch.py``, with
  ``launch_arguments`` as if typed as ``name:=value``.
- Each include is in its own ``GroupAction``: `Each Include in Its Own
  Group`_ shows why.

Running the Team
----------------

.. figure:: /_static/images/l9/launch_tree.svg
   :alt: A tree of launch files. Tag 1: the box team_launch.py, with arrows down to two boxes. Tag 2: robot_launch.py with robot:=robot2 and with_monitor:=false, with an arrow down to a dashed box, the namespace /robot2, holding the node battery_publisher. Tag 3: robot_launch.py with robot:=robot1, with an arrow down to the dashed box /robot1, holding the nodes battery_publisher and battery_monitor.
   :width: 75%

   1: ``team_launch.py``, run with
   ``ros2 launch competition_bringup team_launch.py``. 2: robot2,
   ``with_monitor:=false``, so only its publisher starts. 3: robot1,
   ``with_monitor`` keeps its default, ``true``.

The Team's Nodes and Topics
---------------------------

.. output: team_started.txt all

.. code-block:: text

   [INFO] [battery_publisher-1]: process started with pid [...]
   [INFO] [battery_publisher-2]: process started with pid [...]
   [INFO] [battery_monitor-3]: process started with pid [...]

``ros2 node list``:

.. output: team_nodes.txt all

.. code-block:: text

   /robot1/battery_monitor
   /robot1/battery_publisher
   /robot2/battery_publisher

``ros2 topic list``:

.. output: team_topics.txt all

.. code-block:: text

   /parameter_events
   /robot1/battery
   /robot2/battery
   /rosout

- Three processes from one command: robot2 first, as in the list of
  ``team_launch.py``. The labels number the processes of the whole
  launch, across both includes.
- ``/robot2/battery`` has a publisher and no subscriber: robot2 has no
  monitor.

Each Include in Its Own Group
-----------------------------

``lecture9/errors/team_leak_launch.py`` makes the same includes, without
groups. Run it by its path, from the ROS repository's folder:

.. code: ros/lecture9/errors/team_leak_launch.py all

.. code-block:: python
   :caption: lecture9/errors/team_leak_launch.py

   """An include's arguments reach the next include (Lecture 9)."""

   # robot2 is included first, with with_monitor false; robot1 is included
   # next and does not set with_monitor. Run it by its path, from the ROS
   # repository's folder: ros2 launch lecture9/errors/team_leak_launch.py

   from launch import LaunchDescription
   from launch.actions import IncludeLaunchDescription
   from launch.substitutions import PathJoinSubstitution
   from launch_ros.substitutions import FindPackageShare


   def generate_launch_description():
       robot_launch = PathJoinSubstitution([
           FindPackageShare('competition_bringup'), 'launch',
           'robot_launch.py'])
       robot2 = IncludeLaunchDescription(
           robot_launch, launch_arguments={
               'robot': 'robot2', 'with_monitor': 'false'}.items())
       robot1 = IncludeLaunchDescription(  # Does not set with_monitor
           robot_launch, launch_arguments={'robot': 'robot1'}.items())
       return LaunchDescription([robot2, robot1])

.. code-block:: bash

   ros2 launch lecture9/errors/team_leak_launch.py
   ros2 node list  # In a second terminal

.. output: leak_nodes.txt all

.. code-block:: text

   /robot1/battery_publisher
   /robot2/battery_publisher

- robot1 has no monitor. The include of robot2 set ``with_monitor`` to
  ``false``, and the value stayed set for the next include.
- The docstring of ``GroupAction``: "When scoped=True, changes to launch
  configurations and environment variables are limited to the scope of
  actions in the group action." ``scoped`` is ``True`` by default.


Executors
====================================================

An Executor Runs the Callbacks
------------------------------

The ROS 2 documentation on executors: "Execution management in ROS 2 is
handled by Executors. An Executor uses one or more threads of the
underlying operating system to invoke the callbacks of subscriptions,
timers, service servers, action servers, etc. on incoming messages and
events."

- A **thread** is one sequence of steps inside a process. A process can
  run several threads at once; they share the process's memory, so they
  see the same objects (the
  :doc:`Memory reading </reading_material/memory/mem_index>`,
  "Objects Live on the Heap").
- Lecture 8's ``rclpy.spin(node)`` used an executor: it adds the node to
  a ``SingleThreadedExecutor`` that rclpy makes once
  (``get_global_executor()`` in ``rclpy/__init__.py``).
- rclpy's docstring: a ``SingleThreadedExecutor`` "Runs callbacks in the
  thread that calls" ``spin()``. One thread: one callback at a time.

Step 1: A Controller with Two Timers
------------------------------------

.. code: ros/lecture9/competition_executors/competition_executors/robot_controller.py all

.. code-block:: python
   :caption: competition_executors/robot_controller.py

   """A controller with a slow callback and a fast one (Lecture 9)."""

   import time

   from rclpy.node import Node

   PLAN_PERIOD_S = 1.0  # One plan per second, chosen for this example
   CHECK_PERIOD_S = 0.1  # Ten checks per second, chosen for this example


   class RobotController(Node):
       """Plans a route every 1.0 s and checks safety every 0.1 s."""

       def __init__(self, plan_s=0.5, plan_group=None, check_group=None):
           super().__init__('robot_controller')
           self._groups = (plan_group, check_group)  # Keep both groups
           self._plan_s = plan_s  # How long one plan takes, in seconds
           self._plans_done = 0
           self._t0 = time.monotonic()
           self._last_check = self._t0
           self._plan_timer = self.create_timer(
               PLAN_PERIOD_S, self._plan, callback_group=plan_group)
           self._check_timer = self.create_timer(
               CHECK_PERIOD_S, self._check_safety,
               callback_group=check_group)

       def _plan(self):
           done = self._plans_done  # Read at the start of the plan
           self._log('plan start')
           time.sleep(self._plan_s)  # Stands for the work of planning a route
           self._plans_done = done + 1  # Written back plan_s seconds later
           self._log(f'plan end, plans done: {self._plans_done}')

       def _check_safety(self):
           now = time.monotonic()
           gap = now - self._last_check
           self._last_check = now
           if gap > 1.5 * CHECK_PERIOD_S:  # Late: over 0.15 s since the last
               self._log(f'check late: {gap:.2f} s since the last')

       def _log(self, text):
           elapsed = time.monotonic() - self._t0
           self.get_logger().info(f'{elapsed:.2f} s: {text}')

- One plan per second, and ten safety checks per second (both chosen for
  this example). A plan takes ``plan_s`` seconds: 0.5 by default
  (chosen).
- ``plan_group`` and ``check_group`` are callback groups: sets of
  callbacks (`Callback Groups`_). Left as ``None``, both timers go to the
  node's **default group**, the one group every node has from the start.

Step 2: The Slow Callback and the Fast One
------------------------------------------

- ``time.sleep()`` stands for the work of planning: the thread waits for
  ``plan_s`` seconds. A plan reads the count of plans done at its start
  and writes it back at its end.
- A check logs only when it is late: more than 1.5 periods (0.15 s)
  since the check before. Every log line starts with the seconds since
  the node was made.

Step 3: Spinning with an Executor You Make
------------------------------------------

.. code: ros/lecture9/competition_executors/competition_executors/controller_runs.py all

.. code-block:: python
   :caption: competition_executors/controller_runs.py

   """One RobotController, run by different executors (Lecture 9)."""

   from competition_executors.robot_controller import RobotController
   import rclpy
   from rclpy.callback_groups import (
       MutuallyExclusiveCallbackGroup, ReentrantCallbackGroup)
   from rclpy.executors import (
       ExternalShutdownException, MultiThreadedExecutor, SingleThreadedExecutor)

   THREADS = 4  # Threads in the pool, chosen for this example


   def spin_until_ctrl_c(node, executor):
       """Run the node's callbacks with the executor until Ctrl+C."""
       executor.add_node(node)
       try:
           executor.spin()
       except (KeyboardInterrupt, ExternalShutdownException):
           pass
       except Exception:  # Ctrl+C can also end spin() with an RCLError
           if rclpy.ok():  # ROS 2 still running: a real error
               raise
       finally:
           executor.shutdown()
           node.destroy_node()
           rclpy.try_shutdown()


   def single_threaded(args=None):
       rclpy.init(args=args)
       node = RobotController()  # Both timers in the default group
       spin_until_ctrl_c(node, SingleThreadedExecutor())


   def multi_threaded(args=None):
       rclpy.init(args=args)
       node = RobotController()  # Both timers in the default group
       spin_until_ctrl_c(node, MultiThreadedExecutor(num_threads=THREADS))


   def two_groups(args=None):
       rclpy.init(args=args)
       node = RobotController(plan_group=MutuallyExclusiveCallbackGroup(),
                              check_group=MutuallyExclusiveCallbackGroup())
       spin_until_ctrl_c(node, MultiThreadedExecutor(num_threads=THREADS))


   def slow_plan(args=None):
       rclpy.init(args=args)
       node = RobotController(plan_s=1.5,  # Longer than its 1.0 s period
                              plan_group=MutuallyExclusiveCallbackGroup(),
                              check_group=MutuallyExclusiveCallbackGroup())
       spin_until_ctrl_c(node, MultiThreadedExecutor(num_threads=THREADS))


   def reentrant_plan(args=None):
       rclpy.init(args=args)
       node = RobotController(plan_s=1.5,  # Longer than its 1.0 s period
                              plan_group=ReentrantCallbackGroup(),
                              check_group=MutuallyExclusiveCallbackGroup())
       spin_until_ctrl_c(node, MultiThreadedExecutor(num_threads=THREADS))

- ``add_node()`` then ``spin()``: what ``rclpy.spin()`` did in Lecture
  8. ``executor.shutdown()`` stops the executor; the ``try`` is Lecture
  8's.
- The executor is made after ``rclpy.init()``: it needs the **ROS 2
  context**, the object in which ``init()`` keeps this process's ROS 2
  state (made by ``init()``, ended by ``shutdown()``). Made before, it
  fails:

.. output: executor_before_init.txt all

.. code-block:: text

   TypeError: 'NoneType' object does not support the context manager protocol

Single-Threaded: The Check Waits for the Plan
---------------------------------------------

.. code-block:: bash

   ros2 run competition_executors single_threaded

.. output: exec_single_threaded.txt all

.. code-block:: text

   [INFO] [robot_controller]: 1.00 s: plan start
   [INFO] [robot_controller]: 1.52 s: plan end, plans done: 1
   [INFO] [robot_controller]: 1.52 s: check late: 0.62 s since the last
   [INFO] [robot_controller]: 2.00 s: plan start
   [INFO] [robot_controller]: 2.50 s: plan end, plans done: 2
   [INFO] [robot_controller]: 2.50 s: check late: 0.60 s since the last
   [INFO] [robot_controller]: 3.00 s: plan start
   [INFO] [robot_controller]: 3.50 s: plan end, plans done: 3
   [INFO] [robot_controller]: 3.50 s: check late: 0.60 s since the last

1. At 1.00 s the plan and a check are both due. One thread: the plan
   ran first (in all 5 runs of `Five Runs of Each`_), until 1.52 s.
2. The late check ran at 1.52 s, 0.62 s after the check before it: that
   one ran at 1.52 - 0.62 = 0.90 s, just before the plan.
3. The checks due at 1.0 to 1.5 s did not run: a late timer runs once,
   not once per missed period.

When Each Callback Ran
----------------------

.. figure:: /_static/images/l9/timeline.svg
   :alt: A timeline from 0.6 s to 2.6 s after the node was made, in two panels. In each, the top row shows each plan as a bar and the bottom row each safety check as a tick. Tag 1, single-threaded: plans run from 1.00 to 1.52 s and from 2.00 to 2.50 s; no check runs during a plan, and the longest time between two checks is 0.62 s. Tag 2, multi-threaded, each timer in its own callback group: plans run from 1.00 to 1.50 s and from 2.00 to 2.50 s; the ticks go on during each plan, and the longest time between two checks is 0.10 s.
   :width: 100%

   1: a ``SingleThreadedExecutor``: no check during a plan. 2: a
   ``MultiThreadedExecutor``, each timer in its own callback group
   (`Two Groups: The Check Runs During the Plan`_): the checks go on.
   A bar is one plan, a tick one check, from one run traced by
   ``code/executor_trace.py`` in the slides repository. This is a time
   axis, not a map.

One Executor, Two Nodes, One Process
------------------------------------

.. code: ros/lecture9/competition_executors/competition_executors/battery_pair.py all

.. code-block:: python
   :caption: competition_executors/battery_pair.py

   """Lecture 8's battery publisher and monitor in one process (Lecture 9)."""

   from competition_nodes.battery_monitor import BatteryMonitor
   from competition_nodes.battery_publisher import BatteryPublisher
   import rclpy
   from rclpy.executors import ExternalShutdownException, SingleThreadedExecutor


   def main(args=None):
       rclpy.init(args=args)
       publisher = BatteryPublisher()
       monitor = BatteryMonitor()
       executor = SingleThreadedExecutor()
       executor.add_node(publisher)  # One executor, one thread, two nodes
       executor.add_node(monitor)
       try:
           executor.spin()
       except (KeyboardInterrupt, ExternalShutdownException):
           pass
       except Exception:  # Ctrl+C can also end spin() with an RCLError
           if rclpy.ok():  # ROS 2 still running: a real error
               raise
       finally:
           executor.shutdown()
           monitor.destroy_node()
           publisher.destroy_node()
           rclpy.try_shutdown()

.. code-block:: bash

   ros2 run competition_executors battery_pair

.. output: battery_pair.txt all

.. code-block:: text

   [INFO] [battery_publisher]: Published 16%
   [INFO] [battery_monitor]: 16%: active
   [INFO] [battery_publisher]: Published 14%
   [INFO] [battery_monitor]: 14%: active

The executors docs, about rclcpp's three executors: "All three executors
can be used with multiple nodes by calling add_node(..) for each node."
``battery_pair`` shows the same in rclpy. While it runs,
``ros2 node list`` shows both nodes:

.. output: pair_nodes.txt all

.. code-block:: text

   /battery_monitor
   /battery_publisher

and ``ps -o comm= -C battery_pair`` shows one process:

.. output: pair_ps.txt all

.. code-block:: text

   battery_pair

A Multi-Threaded Executor Has a Pool of Threads
-----------------------------------------------

- rclpy's docstring: ``MultiThreadedExecutor`` "Runs callbacks in a pool
  of threads." A ready callback goes to a free thread of the pool.
- Without ``num_threads``, rclpy uses the number of CPUs the process may
  use, ``len(os.sched_getaffinity(0))``. On the instructor's PC:

.. output: cpu_affinity.txt all

.. code-block:: text

   32

``multi_threaded`` uses 4 threads (``THREADS``, chosen):

.. code-block:: bash

   ros2 run competition_executors multi_threaded

.. output: exec_multi_threaded.txt all

.. code-block:: text

   [INFO] [robot_controller]: 1.00 s: plan start
   [INFO] [robot_controller]: 1.52 s: plan end, plans done: 1
   [INFO] [robot_controller]: 1.52 s: check late: 0.62 s since the last
   [INFO] [robot_controller]: 2.00 s: plan start
   [INFO] [robot_controller]: 2.50 s: plan end, plans done: 2
   [INFO] [robot_controller]: 2.50 s: check late: 0.60 s since the last
   [INFO] [robot_controller]: 3.00 s: plan start
   [INFO] [robot_controller]: 3.50 s: plan end, plans done: 3
   [INFO] [robot_controller]: 3.50 s: check late: 0.60 s since the last

Four threads, and the check still waits for the plan: the same lines as
with one thread. `Callback Groups`_ explains why.


Callback Groups
====================================================

Two Kinds of Group
------------------

A **callback group** is a set of a node's callbacks; it tells a
multi-threaded executor which of them may run at the same time. From the
how-to "Using Callback Groups":

.. list-table::
   :header-rows: 1
   :widths: 22 78
   :class: compact-table

   * - Group
     - What the docs say
   * - Mutually exclusive
     - "prevents its callbacks from being executed in parallel -
       essentially making it as if the callbacks in the group were
       executed by a SingleThreadedExecutor."
   * - Reentrant
     - "different callbacks being run parallel to each other, different
       instances of the same callback may also be executed
       concurrently."
   * - Two groups
     - "Callbacks belonging to different callback groups (of any type)
       can always be executed parallel to each other."

"In parallel" here means on different threads at the same time. With a
``SingleThreadedExecutor`` there is one thread, so groups change
nothing.

The Default Group Is Mutually Exclusive
---------------------------------------

- rclpy's ``node.py``:
  ``self._default_callback_group = MutuallyExclusiveCallbackGroup()``.
  A timer made without ``callback_group=`` joins it.
- The how-to: "if everything in a node uses the same Mutually Exclusive
  Callback Group, that node essentially acts as if it was handled by a
  Single-Threaded Executor, even if a multi-threaded one is specified!"
- That is ``multi_threaded``: both timers in the default group, so the
  check waited for the plan.
- The how-to: "A reference to the callback group should be retained,
  otherwise the callback associated with the callback group will not be
  called by the executor." That is rclcpp's rule. An rclpy timer keeps
  its group (``self.callback_group = callback_group`` in
  ``rclpy/timer.py``), and a timer whose group nothing else kept fired 10
  times in 1 s (measured). ``RobotController`` keeps its groups in
  ``self._groups`` anyway.

Two Groups: The Check Runs During the Plan
------------------------------------------

``two_groups`` in ``controller_runs.py`` gives each timer its own
mutually exclusive group:

.. code-block:: bash

   ros2 run competition_executors two_groups

.. output: exec_two_groups.txt all

.. code-block:: text

   [INFO] [robot_controller]: 1.00 s: plan start
   [INFO] [robot_controller]: 1.51 s: plan end, plans done: 1
   [INFO] [robot_controller]: 2.00 s: plan start
   [INFO] [robot_controller]: 2.50 s: plan end, plans done: 2
   [INFO] [robot_controller]: 3.00 s: plan start
   [INFO] [robot_controller]: 3.50 s: plan end, plans done: 3

The two timers may now run at the same time on two threads, and no check
was late (tag 2 of `When Each Callback Ran`_). Each group alone still
runs one callback at a time: a plan never overlaps another plan.

A Plan Longer Than Its Period, Mutually Exclusive
-------------------------------------------------

``slow_plan``: a plan takes 1.5 s (chosen), and one is due every 1.0 s.

.. output: exec_slow_plan.txt all

.. code-block:: text

   [INFO] [robot_controller]: 1.00 s: plan start
   [INFO] [robot_controller]: 2.52 s: plan end, plans done: 1
   [INFO] [robot_controller]: 2.52 s: plan start
   [INFO] [robot_controller]: 4.02 s: plan end, plans done: 2

1. The plan due at 2.0 s waits for the first to end, and starts at
   2.52 s. The next one starts at 4.02 s, when that one ends.
2. From 1.00 s to 4.02 s: 3 starts, where the period alone would give 4
   (1, 2, 3, 4). Late plans are not stacked up: the plan that starts at
   4.02 s is the call due at 3.0 s, and the call due at 4.0 s was skipped
   (measured with the timer's ``time_until_next_call()``).
3. The counts of plans done go 1, then 2.

``code/skip_check.py`` in the slides repository runs the same timer (1.0 s
period, a 1.5 s callback, one mutually exclusive group, four threads) and
prints, at each plan's start, when the timer's next call is due
(``time_until_next_call()``). The plan of 2.50 s serves the call due at
2.0 s; the plan of 4.00 s serves the call due at 3.0 s, and its next call
is due at 5.0 s: the call due at 4.0 s is skipped.

.. output: skip_check.txt all

.. code-block:: text

   plan start 1.00 s, next call due at 2.00 s
   plan start 2.50 s, next call due at 3.00 s
   plan start 4.00 s, next call due at 5.00 s
   plan start 5.51 s, next call due at 6.00 s

Reentrant: The Plan Overlaps Itself
-----------------------------------

``reentrant_plan``: the same 1.5 s plan, in a reentrant group.

.. output: exec_reentrant_plan.txt all

.. code-block:: text

   [INFO] [robot_controller]: 1.00 s: plan start
   [INFO] [robot_controller]: 2.00 s: plan start
   [INFO] [robot_controller]: 2.51 s: plan end, plans done: 1
   [INFO] [robot_controller]: 3.00 s: plan start
   [INFO] [robot_controller]: 3.50 s: plan end, plans done: 1
   [INFO] [robot_controller]: 4.00 s: plan start
   [INFO] [robot_controller]: 4.50 s: plan end, plans done: 2

1. Every plan starts on time: 1.00, 2.00, 3.00, 4.00 s.
2. The plan of 2.00 s starts before the plan of 1.00 s ends (2.51 s):
   from 2.00 to 2.51 s, two plans run at once, on two threads.
3. Both run the same method on the same ``self``: they share every
   attribute of the node.

Two Plans, One Attribute
------------------------

The count of plans done, from the lines of
`Reentrant: The Plan Overlaps Itself`_:

.. list-table::
   :header-rows: 1
   :widths: 16 28 30 26
   :class: compact-table

   * - Plan
     - Reads at its start
     - Writes at its end
     - Logged
   * - 1.00 s
     - 0
     - 0 + 1 = 1, at 2.51 s
     - ``plans done: 1``
   * - 2.00 s
     - 0 (plan 1 not done)
     - 0 + 1 = 1, at 3.50 s
     - ``plans done: 1``
   * - 3.00 s
     - 1
     - 1 + 1 = 2, at 4.50 s
     - ``plans done: 2``

1. Three plans ended, and the count says 2: the plan of 2.00 s read the
   count before the plan of 1.00 s wrote it, so one update was lost.
2. With a mutually exclusive group (``slow_plan``), a plan starts only
   after the last one ended: the counts were 1, then 2.

Callbacks that read and write the same attributes, and must not overlap,
belong in one mutually exclusive group.

Five Runs of Each
-----------------

Each executable was run 5 times (``code/make_outputs.sh`` in the slides
repository), and every line logged up to 4.6 s after the node was made
was compared:

.. output: executor_runs.txt all

.. code-block:: text

   executable       alike  spread  late  late by (s)  at once  counts
   single_threaded  5/5    0.01 s  4     0.60 to 0.62 1        1,2,3,4
   multi_threaded   5/5    0.01 s  4     0.60 to 0.62 1        1,2,3,4
   two_groups       5/5    0.01 s  0     none         1        1,2,3,4
   slow_plan        5/5    0.01 s  0     none         1        1,2
   reentrant_plan   5/5    0.02 s  0     none         2        1,1,2

- **alike**: the runs whose lines match run 1's with every number left
  out: the same messages in the same order.
- **spread**: the largest difference, over the 5 runs, between the time
  stamps of one line.
- **late**: late checks in each run; **late by (s)**: the smallest and
  largest of their gaps; **at once**: the most plans running at the same
  time; **counts**: the counts of plans done, in order.

The slides' outputs are not luck: every run gave the same lines, within
the spread shown. The times can differ on another computer.

Python Threads Take Turns on Python Code
----------------------------------------

.. code: lecture9/gil_demo.py all

.. code-block:: python
   :caption: lecture9/gil_demo.py

   """Two threads that wait, and two threads that compute, timed.

   Each task is run once alone, then twice at the same time in two threads.
   Every time is the best of 5 repeats, in seconds.
   """

   import threading
   import time

   N = 8_000_000  # Chosen so that one count takes about 0.2 s on the instructor PC


   def wait():
       time.sleep(0.2)  # Waiting: the thread runs no Python code


   def count():
       total = 0
       for i in range(N):  # Computing: the thread runs Python code
           total += i


   def best_time(task, threads):
       best = float("inf")
       for _ in range(5):
           workers = [threading.Thread(target=task) for _ in range(threads)]
           start = time.perf_counter()
           for w in workers:
               w.start()
           for w in workers:
               w.join()
           best = min(best, time.perf_counter() - start)
       return best


   for task in (wait, count):
       one = best_time(task, 1)
       two = best_time(task, 2)
       print(f"{task.__name__}: 1 thread {one:.2f} s, 2 threads {two:.2f} s")

One run (each time is the best of 5 repeats):

.. output: gil_demo.txt all

.. code-block:: text

   wait: 1 thread 0.20 s, 2 threads 0.20 s
   count: 1 thread 0.21 s, 2 threads 0.40 s

1. Two threads that wait take 0.20 s, the time of one: they wait at the
   same time.
2. Two threads that count take 0.40 s, about twice the 0.21 s of one:
   they took turns.
3. The Python glossary: the global interpreter lock (GIL) is "The
   mechanism used by the CPython interpreter to assure that only one
   thread executes Python bytecode at a time."

So threads let callbacks wait at the same time (``time.sleep()``, a
reply). Two callbacks that compute in Python take turns, and finish no
sooner than one after the other. Three more runs, to show how much the
times move:

.. output: gil_runs.txt all

.. code-block:: text

   run 1:
   wait: 1 thread 0.20 s, 2 threads 0.20 s
   count: 1 thread 0.20 s, 2 threads 0.40 s
   run 2:
   wait: 1 thread 0.20 s, 2 threads 0.20 s
   count: 1 thread 0.20 s, 2 threads 0.40 s
   run 3:
   wait: 1 thread 0.20 s, 2 threads 0.20 s
   count: 1 thread 0.20 s, 2 threads 0.40 s

Choosing an Executor and Groups
-------------------------------

.. list-table::
   :header-rows: 1
   :widths: 24 30 22 24
   :class: compact-table

   * - Executable
     - Executor, groups
     - Check late
     - Plans overlap
   * - ``single_threaded``
     - Single, default group
     - Yes, 0.6 s
     - No
   * - ``multi_threaded``
     - Multi, default group
     - Yes, 0.6 s
     - No
   * - ``two_groups``
     - Multi, two exclusive
     - No
     - No
   * - ``slow_plan``
     - Multi, two exclusive
     - No
     - No
   * - ``reentrant_plan``
     - Multi, plan reentrant
     - No
     - Yes

- The how-to's rule for callbacks that should never run at the same
  time: "Register them to the same Mutually Exclusive Callback Group".
  Its example: callbacks "accessing shared critical and non-thread-safe
  resources".
- A callback that must not overlap itself, like a control loop: a
  mutually exclusive group. A callback that may: a reentrant group.

Lecture Summary
---------------

- ROS arguments come after ``--ros-args``; with ``args=None``,
  ``rclpy.init()`` reads them from ``sys.argv``.
- A namespace puts a prefix on a node's relative names;
  ``-r old:=new`` replaces one name; ``-p`` sets a parameter, which stays
  under its node's full name.
- A launch file starts many nodes with one command and passes their
  names, namespaces, remappings, and parameters as ROS arguments.
- Launch arguments, conditions, groups, and includes make one launch
  file serve many robots; each include belongs in its own group.
- One thread runs one callback at a time. A multi-threaded executor runs
  callbacks of different groups, or of one reentrant group, at the same
  time; their Python code still takes turns (the GIL).
