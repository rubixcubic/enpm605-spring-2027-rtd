====================================================
Python Exercises
====================================================

Four exercises on :doc:`Lecture 9 <l9_lecture>`. Exercises 1 to 3 are
the exercises from the slides; Exercise 4 is for after class. Do them in
your workspace, ``~/ros_ws``, in terminals where ROS 2 and the workspace
are sourced, ``ROS_DOMAIN_ID`` is your own number, and
``ROS_AUTOMATIC_DISCOVERY_RANGE`` is ``LOCALHOST`` (Lecture 8). Exercises
2 to 4 add files to your own package, ``my_competition``, from Lecture
8's Exercise 2.

.. note::

   Try each exercise before you open its solution. Every output in a
   solution was printed by running the solution, with
   ``ROS_DOMAIN_ID=77`` and the log format without time stamps of the
   lecture page. Exercises 2 and 4 were built in a new workspace with
   ``my_competition`` made as Lecture 8's Exercise 2 makes it. Exercise
   3's file was run with ``python3 file.py``, the Python that
   ``ros2 run`` uses; the node's name is set in the code, so the log
   lines are the same as with ``ros2 run``.


----


.. dropdown:: Exercise 1: Names from the Command Line
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Put nodes in a namespace, rename one, and remap a topic, with
    ``--ros-args`` only.

    **Tasks**

    In five terminals:

    1. Start the listener of ``demo_nodes_py`` in the namespace
       ``/team_red``.
    2. Start a second listener in ``/team_red``, named ``listener2``.
    3. Start a third listener named ``red_fan``, in no namespace, that
       hears ``/team_red/chatter``.
    4. Start the talker in ``/team_red``. Do all three listeners print its
       messages?
    5. What do ``ros2 node list`` and ``ros2 topic list`` show? How many
       subscribers does ``/team_red/chatter`` have?

    .. dropdown:: Solution
        :class-container: sd-border-success

        1 to 4. The four commands:

        .. code-block:: bash

           ros2 run demo_nodes_py listener --ros-args -r __ns:=/team_red
           ros2 run demo_nodes_py listener --ros-args -r __ns:=/team_red -r __node:=listener2
           ros2 run demo_nodes_py listener --ros-args -r __node:=red_fan -r chatter:=/team_red/chatter
           ros2 run demo_nodes_py talker --ros-args -r __ns:=/team_red

        All three listeners print every message. ``red_fan`` is outside
        the namespace, so its ``chatter`` would be ``/chatter``; the rule
        makes it ``/team_red/chatter``. Its first lines:

        .. output: exercises/ex1_fan.txt all

        .. code-block:: text

           [INFO] [red_fan]: I heard: [Hello World: 0]
           [INFO] [red_fan]: I heard: [Hello World: 1]

        5. ``ros2 node list``:

        .. output: exercises/ex1_nodes.txt all

        .. code-block:: text

           /red_fan
           /team_red/listener
           /team_red/listener2
           /team_red/talker

        ``ros2 topic list``:

        .. output: exercises/ex1_topics.txt all

        .. code-block:: text

           /parameter_events
           /rosout
           /team_red/chatter

        ``ros2 topic info /team_red/chatter``: one publisher, three
        subscribers.

        .. output: exercises/ex1_topic_info.txt all

        .. code-block:: text

           Type: std_msgs/msg/String
           Publisher count: 1
           Subscription count: 3


.. dropdown:: Exercise 2: A Launch File for Exercise 1
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Start the four nodes of Exercise 1 with one command.

    **Tasks**

    In your package ``my_competition``:

    1. Write ``launch/chatter_launch.py``: it starts the four nodes of
       Exercise 1, with the same names, namespaces, and remapping.
    2. Add the ``data_files`` line and the ``glob`` import of "Installing
       the Launch Files" to ``setup.py``.
    3. Add ``<exec_depend>`` lines for ``launch``, ``launch_ros``, and
       ``demo_nodes_py`` to ``package.xml``.
    4. Build in ``~/ros_ws``, source ``install/setup.bash``, and run it
       with ``ros2 launch``. Check the names with ``ros2 node list``.

    .. dropdown:: Solution
        :class-container: sd-border-success

        1. ``my_competition/launch/chatter_launch.py``:

        .. code: exercises/ex2_chatter_launch.py all

        .. code-block:: python

           """Exercise 2: a talker and three listeners, started by one launch file."""

           from launch import LaunchDescription
           from launch_ros.actions import Node


           def generate_launch_description():
               talker = Node(package='demo_nodes_py', executable='talker',
                             namespace='team_red')
               listener = Node(package='demo_nodes_py', executable='listener',
                               namespace='team_red')
               listener2 = Node(package='demo_nodes_py', executable='listener',
                                namespace='team_red', name='listener2')
               fan = Node(package='demo_nodes_py', executable='listener',
                          name='red_fan',  # Outside the namespace, a new name
                          remappings=[('chatter', '/team_red/chatter')])
               return LaunchDescription([listener, listener2, fan, talker])

        2. ``setup.py``, compared with the file ``ros2 pkg create``
           wrote (``diff -u``; ``+`` marks an added line):

        .. output: exercises/ex2_setup_diff.txt all

        .. code-block:: text

           @@ -1,3 +1,5 @@
           +from glob import glob
           +
            from setuptools import find_packages, setup
            
            package_name = 'my_competition'
           @@ -10,6 +12,7 @@
                    ('share/ament_index/resource_index/packages',
                        ['resource/' + package_name]),
                    ('share/' + package_name, ['package.xml']),
           +        ('share/' + package_name + '/launch', glob('launch/*')),
                ],
                install_requires=['setuptools'],
                zip_safe=True,

        3. In ``package.xml``, after ``<depend>rclpy</depend>``:

        .. code-block:: xml

           <exec_depend>launch</exec_depend>
           <exec_depend>launch_ros</exec_depend>
           <exec_depend>demo_nodes_py</exec_depend>

        4. After ``colcon build --symlink-install`` and
           ``source install/setup.bash``:

        .. code-block:: bash

           ros2 launch my_competition chatter_launch.py
           ros2 node list

        .. output: exercises/ex2_nodes.txt all

        .. code-block:: text

           /red_fan
           /team_red/listener
           /team_red/listener2
           /team_red/talker

        ``red_fan`` is the third process of the launch, so its label is
        ``[listener-3]``: the label names the executable, and the node
        name follows it.

        .. output: exercises/ex2_fan.txt all

        .. code-block:: text

           [listener-3] [INFO] [red_fan]: I heard: [Hello World: 0]
           [listener-3] [INFO] [red_fan]: I heard: [Hello World: 1]


.. dropdown:: Exercise 3: Predict, Then Run
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Predict what one reentrant group for both timers of
    ``RobotController`` does, then check it.

    **Tasks**

    1. In ``my_competition``, write ``both_reentrant.py`` with a
       ``main()`` that makes one ``ReentrantCallbackGroup()`` and passes it
       to ``RobotController(plan_s=1.5, plan_group=..., check_group=...)``,
       then spins it with ``MultiThreadedExecutor(num_threads=4)``.
    2. Before you run it, write down its first 7 log lines. Is any check
       late? Do plans overlap? What counts of plans done are logged?
    3. Add an entry point and ``<depend>competition_executors</depend>``,
       build, and run it for 5 s. Compare with your prediction.

    .. dropdown:: Solution
        :class-container: sd-border-success

        1. ``my_competition/both_reentrant.py``:

        .. code: exercises/ex3_both_reentrant.py all

        .. code-block:: python

           """Exercise 3: both timers of RobotController in one reentrant group."""

           from competition_executors.controller_runs import spin_until_ctrl_c
           from competition_executors.robot_controller import RobotController
           import rclpy
           from rclpy.callback_groups import ReentrantCallbackGroup
           from rclpy.executors import MultiThreadedExecutor


           def main(args=None):
               rclpy.init(args=args)
               group = ReentrantCallbackGroup()  # One group for both timers
               node = RobotController(plan_s=1.5, plan_group=group, check_group=group)
               spin_until_ctrl_c(node, MultiThreadedExecutor(num_threads=4))


           if __name__ == '__main__':
               main()

        2. The prediction. A reentrant group puts no limit on its
           callbacks, so the result is ``reentrant_plan``'s, with the
           check in the same group:

           - Every plan starts on time, at 1.00, 2.00, 3.00, and 4.00 s,
             and ends 1.5 s later; from 2.00 to 2.50 s and from 3.00 to
             3.50 s, two plans run at once.
           - No check is late: a check never waits for a plan.
           - The counts are 1, 1, 2, as in "Two Plans, One Attribute" on the
             lecture page: the plan of 2.00 s reads the count (0) before the
             plan of 1.00 s writes 1, so it writes 0 + 1 = 1 too, and one
             update is lost.

        3. The run (first 7 lines):

        .. output: exercises/ex3_both_reentrant.txt all

        .. code-block:: text

           [INFO] [robot_controller]: 1.00 s: plan start
           [INFO] [robot_controller]: 2.00 s: plan start
           [INFO] [robot_controller]: 2.51 s: plan end, plans done: 1
           [INFO] [robot_controller]: 3.00 s: plan start
           [INFO] [robot_controller]: 3.50 s: plan end, plans done: 1
           [INFO] [robot_controller]: 4.00 s: plan start
           [INFO] [robot_controller]: 4.50 s: plan end, plans done: 2

        Three plans ended by 4.50 s, and the last count is 2: the lost
        update. The entry point in ``setup.py``:
        ``'both_reentrant = my_competition.both_reentrant:main'``.


.. dropdown:: Exercise 4: Launch Arguments and Includes (After Class)
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Make Exercise 2's launch file reusable, then start two teams with it.

    **Tasks**

    1. Write ``launch/team_chatter_launch.py``: a launch argument
       ``team`` (default ``team_red``) is the namespace of a group with
       the talker, a listener, and a second listener named ``listener2``;
       a launch argument ``second_listener`` (default ``true``) starts
       ``listener2`` only if it is true.
    2. Write ``launch/two_teams_launch.py``: it includes
       ``team_chatter_launch.py`` twice, once with ``team:=team_blue``
       and ``second_listener:=false``, once with ``team:=team_red``,
       each include in its own group.
    3. Run ``two_teams_launch.py`` before you build again. What happens?
       Build, run it, and check ``ros2 node list``.

    .. dropdown:: Solution
        :class-container: sd-border-success

        1. ``my_competition/launch/team_chatter_launch.py``:

        .. code: exercises/ex4_team_chatter_launch.py all

        .. code-block:: python

           """Exercise 4: one team's talker and listeners, in a namespace you choose."""

           from launch import LaunchDescription
           from launch.actions import DeclareLaunchArgument, GroupAction
           from launch.conditions import IfCondition
           from launch.substitutions import LaunchConfiguration
           from launch_ros.actions import Node, PushROSNamespace


           def generate_launch_description():
               team = LaunchConfiguration('team')
               second = LaunchConfiguration('second_listener')
               return LaunchDescription([
                   DeclareLaunchArgument('team', default_value='team_red'),
                   DeclareLaunchArgument('second_listener', default_value='true'),
                   GroupAction([
                       PushROSNamespace(team),
                       Node(package='demo_nodes_py', executable='listener'),
                       Node(package='demo_nodes_py', executable='listener',
                            name='listener2', condition=IfCondition(second)),
                       Node(package='demo_nodes_py', executable='talker'),
                   ]),
               ])

        2. ``my_competition/launch/two_teams_launch.py``:

        .. code: exercises/ex4_two_teams_launch.py all

        .. code-block:: python

           """Exercise 4: two teams, from team_chatter_launch.py included twice."""

           from launch import LaunchDescription
           from launch.actions import GroupAction, IncludeLaunchDescription
           from launch.substitutions import PathJoinSubstitution
           from launch_ros.substitutions import FindPackageShare


           def generate_launch_description():
               team_launch = PathJoinSubstitution(
                   [FindPackageShare('my_competition'), 'launch',
                    'team_chatter_launch.py'])
               blue = GroupAction([IncludeLaunchDescription(
                   team_launch,
                   launch_arguments={'team': 'team_blue',
                                     'second_listener': 'false'}.items())])
               red = GroupAction([IncludeLaunchDescription(
                   team_launch, launch_arguments={'team': 'team_red'}.items())])
               return LaunchDescription([blue, red])

        3. A new launch file is not in the install folder until the next
           build (``setup.py``'s ``glob()`` runs at build time). Run
           before the build, ``ros2 launch`` says (workspace folder
           shortened):

        .. output: exercises/ex4_not_found.txt all

        .. code-block:: text

           file 'two_teams_launch.py' was not found in the share directory of package 'my_competition' which is at '.../install/my_competition/share/my_competition'

        After ``colcon build --symlink-install`` and
        ``source install/setup.bash``, ``ros2 node list`` shows two
        teams; ``team_blue`` has no ``listener2``:

        .. output: exercises/ex4_nodes.txt all

        .. code-block:: text

           /team_blue/listener
           /team_blue/talker
           /team_red/listener
           /team_red/listener2
           /team_red/talker
