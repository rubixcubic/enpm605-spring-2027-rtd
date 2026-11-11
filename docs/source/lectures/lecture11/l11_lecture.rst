====================================================
Lecture
====================================================

This lecture uses the ROS repository,
`enpm605-spring-2027-ros <https://github.com/rubixcubic/enpm605-spring-2027-ros>`_.
Its folder ``lecture11/`` holds two new packages and one script:

- ``competition_motion``: the arena launch file (Gazebo Harmonic, a
  TurtleBot3 Waffle, and RViz2), the world file ``worlds/arena.sdf``, the
  RViz2 settings ``rviz/arena.rviz``, the helpers ``pose_tools.py``, and
  the nodes ``odom_reader`` and ``go_to_goal``.
- ``competition_frames``: the nodes ``goal_broadcaster``,
  ``frame_listener``, and ``tf_go_to_goal``.
- ``scripts/pose_demo.py``: a script, run with ``python3``, that prints
  quaternions.

Run ``git pull`` in the clone in ``~/ros_ws/src``, then build the
workspace with ``colcon build --symlink-install`` and source
``install/setup.bash``. ``competition_frames`` uses
``competition_motion``. This lecture has no plain Python program, so the
course repository has no ``lecture11/`` folder.

The simulation needs Gazebo Harmonic and two ROS 2 packages that a ROS 2
desktop install does not bring. Install them once:

.. code-block:: bash

   sudo apt install ros-jazzy-ros-gz ros-jazzy-nav2-minimal-tb3-sim

The Gazebo documentation, "Installing Gazebo with ROS", gives
``sudo apt-get install ros-${ROS_DISTRO}-ros-gz`` and says: "This command
will install the Gazebo libraries recommended for your ROS installation."

Every output on this page was printed by running the code with ROS 2
Jazzy and Gazebo Harmonic on Ubuntu 24.04, with ``ROS_DOMAIN_ID=77`` and
``ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST``, in a workspace whose ROS
repository held ``lecture8/`` to ``lecture11/``: what the repository
holds on Apr 20. The simulation ran without its windows
(``gui:=false rviz:=false``). Log lines were recorded without their time
stamp (Lecture 8, "Reading a Log Line"). A long warning is folded to fit,
where the text says so. The goal, the gains, the speeds of the examples,
and the size and place of the task site are chosen for the example.

.. note::

   The recordings set one more variable, ``GZ_PARTITION=enpm605_l11``,
   because another simulation ran on the same computer. Gazebo's own
   messages do not use ROS domains. The Gazebo Transport documentation:
   ``GZ_PARTITION`` "Specifies a partition name for all the nodes declared
   inside this process", and its default value is
   "<HOSTNAME>:<USERNAME>". On your own computer, you do not need it.


Pose
====================================================

Where Lecture 10 Stopped
------------------------

Lecture 10's action server "drove" with ``time.sleep(STEP_S)``: nothing
moved, and the distance was a sum of steps. This lecture drives a
simulated robot, a TurtleBot3 Waffle, in Gazebo Harmonic. The scout's job
is to stop in front of the task site, a box at (3.0, 1.0) m. To do that,
the robot needs five things, one per part of the lecture:

.. list-table::
   :header-rows: 1
   :widths: 45 55
   :class: compact-table

   * - Question
     - This lecture's answer
   * - Where is the robot, and which way does it face?
     - A pose: a position and a quaternion
   * - How do two wheels move it?
     - The differential drive model
   * - How far has it gone?
     - Odometry, on ``/odom``
   * - Which command takes it to the goal?
     - A proportional controller
   * - Where is the goal, seen from the robot?
     - Coordinate frames and TF2

A Pose Is a Position and an Orientation
---------------------------------------

A **pose** says where something is and which way it faces, in a frame.
ROS 2's message for it, with its comments removed:

.. code-block:: bash

   ros2 interface show geometry_msgs/msg/Pose

.. output: i_pose_fields.txt all

.. code-block:: text

   Point position
     float64 x
     float64 y
     float64 z
   Quaternion orientation
     float64 x 0
     float64 y 0
     float64 z 0
     float64 w 1

``position`` is in meters along the frame's x, y, and z. ``orientation``
is a quaternion, four numbers; its defaults, w = 1 and the rest 0, mean
"no turn".

.. figure:: /_static/images/l11/body_axes.svg
   :alt: Seen from above. Tag 1: the odom frame at the bottom left, its x axis to the right and its y axis up, in gray. A two-wheeled robot stands at position (x, y), shown by dashed lines to odom's axes, facing 30 degrees to the left of odom's x. Its own frame: tag 2, x points forward; tag 3, y points to its left; z points up, out of the page, drawn as a dot in a circle. Tag 4: the yaw, an arc from a dashed line parallel to odom's x to the robot's x, counterclockwise.
   :width: 60%

   1: the frame the pose is given in, ``odom`` here. 2 and 3: the robot's
   own x (forward) and y (to its left); z points up, out of the page. 4:
   the yaw, from ``odom``'s x to the robot's x.

REP 103: Which Way the Axes Point
---------------------------------

REP 103 sets the conventions every ROS package follows. "All systems are
right handed." For a body, "x forward", "y left", "z up". A turn about x
is a **roll**, about y a **pitch**, about z a **yaw**. A robot on a flat
floor turns only about z: only its yaw changes. REP 103 on the sign of
the yaw: "By the right hand rule, the yaw component of orientation
increases as the child frame rotates counter-clockwise". Angles are in
radians (REP 103's table of units).

REP 103 lists the ways to write a rotation, the preferred one first: a
quaternion ("Compact representation", "No singularities"), a rotation
matrix (Lecture 5), roll, pitch, and yaw about fixed axes, and last, Euler
angles: "Euler angles are generally discouraged due to having 24 'valid'
conventions with different domains using different conventions by
default."

A robot that turns only about z still needs a quaternion: the message
types are shared with drones and arms, which turn about all three axes,
and ``Pose`` has one way to store an orientation.

A Quaternion for a Turn About z
-------------------------------

A turn by an angle θ about an axis (u\ :sub:`x`, u\ :sub:`y`,
u\ :sub:`z`) of length 1 is the quaternion

.. math::

   x = u_x \sin\tfrac{\theta}{2}, \quad y = u_y \sin\tfrac{\theta}{2},
   \quad z = u_z \sin\tfrac{\theta}{2}, \quad w = \cos\tfrac{\theta}{2}

For a yaw ψ, the axis is z, (0, 0, 1), so the quaternion is
(0, 0, sin(ψ/2), cos(ψ/2)). A yaw of 90 degrees, in three steps:

1. Half the angle: 90 / 2 = 45 degrees.
2. sin 45° = 0.707 and cos 45° = 0.707.
3. (x, y, z, w) = (0, 0, 0.707, 0.707).

The order matters. The ROS 2 tutorial "Quaternion fundamentals": "In ROS
2, w is last, but in some libraries like Eigen, w can be placed at the
first position." And: "The magnitude of a quaternion should always be
one." Here, 0.707\ :sup:`2` + 0.707\ :sup:`2` = 1.0 (to 3 decimals).

pose_tools.py: Both Ways, and an Angle Wrap
-------------------------------------------

.. code: ros/lecture11/competition_motion/competition_motion/pose_tools.py all

.. code-block:: python
   :caption: competition_motion/competition_motion/pose_tools.py

   """Quaternions and angles for a robot that turns only about z (Lecture 11)."""

   import math


   def quaternion_from_yaw(yaw):
       """Return (x, y, z, w), w last, for a turn of yaw radians about z."""
       return (0.0, 0.0, math.sin(yaw / 2), math.cos(yaw / 2))


   def yaw_from_quaternion(q):
       """Return the yaw, in radians, of a geometry_msgs Quaternion."""
       return math.atan2(2 * (q.w * q.z + q.x * q.y),
                         1 - 2 * (q.y * q.y + q.z * q.z))


   def wrap_angle(angle):
       """Return the same angle, in radians, between -pi and pi."""
       return (angle + math.pi) % (2 * math.pi) - math.pi

``yaw_from_quaternion()`` is the general formula for the yaw. For a pure
yaw, x = y = 0, it gives atan2(2wz, 1 − 2z\ :sup:`2`), which is
atan2(sin ψ, cos ψ) = ψ (because 2 sin(ψ/2) cos(ψ/2) = sin ψ and
1 − 2 sin\ :sup:`2`\ (ψ/2) = cos ψ). ``math.atan2(y, x)`` uses both
signs, so it returns an angle between −π and π in the right quarter.
``wrap_angle()`` brings any angle back to that range; the controller
needs it. ``test/test_pose_tools.py`` checks all three functions, and
``colcon test`` runs it.

Five Yaws, Measured
-------------------

.. code: ros/lecture11/scripts/pose_demo.py all

.. code-block:: python
   :caption: lecture11/scripts/pose_demo.py

   """Quaternions for a robot that turns about z (Lecture 11, run by path)."""

   import math

   from competition_motion.pose_tools import (quaternion_from_yaw, wrap_angle,
                                              yaw_from_quaternion)
   from geometry_msgs.msg import Quaternion

   print('yaw (deg)       x      y      z      w')
   for degrees in (0, 30, 90, 180, -90):
       q = quaternion_from_yaw(math.radians(degrees))
       print(f'{degrees:9}' + ''.join(f'{v:7.3f}' for v in q))

   x, y, z, w = quaternion_from_yaw(math.radians(30))
   print('length:', round(math.sqrt(x * x + y * y + z * z + w * w), 6))
   msg = Quaternion(x=x, y=y, z=z, w=w)
   print('back to yaw:', round(math.degrees(yaw_from_quaternion(msg)), 6))
   minus = Quaternion(x=-x, y=-y, z=-z, w=-w)
   print('-q gives yaw:', round(math.degrees(yaw_from_quaternion(minus)), 6))
   print('Quaternion():', Quaternion())
   print('wrap 190 deg:', round(math.degrees(wrap_angle(math.radians(190))), 6))

Run it by path, in a terminal where ROS 2 and the workspace are sourced
(it imports ``competition_motion`` and ``geometry_msgs``):

.. code-block:: bash

   python3 lecture11/scripts/pose_demo.py

.. output: pose_demo.txt all

.. code-block:: text

   yaw (deg)       x      y      z      w
           0  0.000  0.000  0.000  1.000
          30  0.000  0.000  0.259  0.966
          90  0.000  0.000  0.707  0.707
         180  0.000  0.000  1.000  0.000
         -90  0.000  0.000 -0.707  0.707
   length: 1.0
   back to yaw: 30.0
   -q gives yaw: 30.0
   Quaternion(): geometry_msgs.msg.Quaternion(x=0.0, y=0.0, z=0.0, w=1.0)
   wrap 190 deg: -170.0

- 90 degrees gives (0, 0, 0.707, 0.707), the three steps above.
- 180 degrees gives w = 0; −90 degrees gives a negative z.
- q and −q are the same turn: both give a yaw of 30 degrees.
- ``Quaternion()`` is w = 1: no turn.
- 190 degrees wraps to −170: a turn of 170 degrees to the right.


Gazebo Harmonic and RViz2
====================================================

Gazebo Simulates; the Bridge Turns It into Topics
-------------------------------------------------

.. figure:: /_static/images/l11/sim_graph.svg
   :alt: A graph of the simulation. Tag 1: a box, Gazebo, holding two boxes, DiffDrive and lidar. A dashed line labeled gz joins Gazebo to tag 2, the node ros_gz_bridge. The bridge publishes the topics /odom, /scan and /tf, and subscribes to /cmd_vel. It also publishes /joint_states, to which tag 3, the node robot_state_publisher, subscribes; it publishes /tf, /tf_static and /robot_description. On the right, the node go_to_goal publishes /cmd_vel and subscribes to /odom, and tag 4, the node rviz2, subscribes to /odom, /scan, /tf, /tf_static and /robot_description.
   :width: 95%

   1: Gazebo, with the robot's ``DiffDrive`` system and its lidar. 2:
   ``ros_gz_bridge``. 3: ``robot_state_publisher``. 4: RViz2. The topics
   are those of the recordings below; ``/clock`` and ``/imu`` are left
   out. ``/joint_states`` goes from the bridge to
   ``robot_state_publisher``, which turns the wheel angles into ``/tf``.

Gazebo computes the physics. The robot's ``DiffDrive`` system turns the
wheels and reports odometry; the lidar measures ranges. Gazebo has its own
topics (gz), not ROS 2's. ``ros_gz_bridge``, in the Gazebo documentation
"Use ROS 2 to interact with Gazebo", "provides a network bridge which
enables the exchange of messages between ROS 2 and Gazebo Transport."
``robot_state_publisher`` publishes where the robot's links are, from its
description. RViz2 only subscribes.

The Arena Launch File
---------------------

.. code: ros/lecture11/competition_motion/launch/arena_launch.py all

.. code-block:: python
   :caption: competition_motion/launch/arena_launch.py

   """Gazebo Harmonic, a TurtleBot3 Waffle in the arena, and RViz2 (Lecture 11)."""

   # The robot, its bridge to ROS 2, and its URDF come from nav2_minimal_tb3_sim,
   # the package that Nav2's own simulation uses (Lecture 12).

   import os

   from ament_index_python.packages import get_package_share_directory
   from launch import LaunchDescription
   from launch.actions import (DeclareLaunchArgument, ExecuteProcess,
                               IncludeLaunchDescription)
   from launch.conditions import IfCondition
   from launch.launch_description_sources import PythonLaunchDescriptionSource
   from launch.substitutions import LaunchConfiguration
   from launch_ros.actions import Node


   def generate_launch_description():
       here = get_package_share_directory('competition_motion')
       tb3 = get_package_share_directory('nav2_minimal_tb3_sim')
       world = os.path.join(here, 'worlds', 'arena.sdf')
       with open(os.path.join(tb3, 'urdf', 'turtlebot3_waffle.urdf')) as f:
           robot_description = f.read()  # The robot's links and joints
       gui = LaunchConfiguration('gui')
       rviz = LaunchConfiguration('rviz')
       return LaunchDescription([
           DeclareLaunchArgument('gui', default_value='true',
                                 description='Open the Gazebo window'),
           DeclareLaunchArgument('rviz', default_value='true',
                                 description='Open RViz2'),
           # Spawn the robot at (0, 0), bridge its topics to ROS 2, and tell
           # Gazebo where its meshes are (first, so the server sees it)
           IncludeLaunchDescription(
               PythonLaunchDescriptionSource(
                   os.path.join(tb3, 'launch', 'spawn_tb3.launch.py')),
               launch_arguments={'x_pose': '0.0', 'y_pose': '0.0'}.items()),
           # The simulation itself (-s: server, -r: run at once)
           ExecuteProcess(cmd=['gz', 'sim', '-r', '-s', world]),
           # The Gazebo window, a client of the server (-g: GUI only)
           ExecuteProcess(cmd=['gz', 'sim', '-g'], condition=IfCondition(gui)),
           # Publish the transforms between the robot's links
           Node(package='robot_state_publisher',
                executable='robot_state_publisher',
                parameters=[{'robot_description': robot_description,
                             'use_sim_time': True}]),
           Node(package='rviz2', executable='rviz2',
                arguments=['-d', os.path.join(here, 'rviz', 'arena.rviz')],
                parameters=[{'use_sim_time': True}],
                condition=IfCondition(rviz)),
       ])

- ``spawn_tb3.launch.py`` comes from ``nav2_minimal_tb3_sim``, the
  package Nav2's own simulation uses (Lecture 12). It starts the bridge,
  puts the robot in the world at (0, 0), and tells Gazebo where the
  robot's meshes are; it comes first so that the Gazebo server sees that
  setting.
- ``ExecuteProcess`` runs a program that is not a ROS 2 node: ``gz sim``.
  The server (``-s``) runs the physics; the window (``-g``) is a client of
  it, skipped with ``gui:=false``.
- RViz2 gets ``use_sim_time``: the transforms are stamped with the
  simulation's time (see "The Odometry Message").

``setup.py`` installs the three folders:

.. code: ros/lecture11/competition_motion/setup.py 11-18

.. code-block:: python
   :caption: competition_motion/setup.py, data_files

       data_files=[
           ('share/ament_index/resource_index/packages',
               ['resource/' + package_name]),
           ('share/' + package_name, ['package.xml']),
           ('share/' + package_name + '/launch', glob('launch/*')),
           ('share/' + package_name + '/worlds', glob('worlds/*')),
           ('share/' + package_name + '/rviz', glob('rviz/*')),
       ],

Start the Arena, Then List Its Topics
-------------------------------------

.. code-block:: bash

   ros2 launch competition_motion arena_launch.py
   ros2 topic list -t

The topics, without ``/parameter_events`` and ``/rosout``:

.. output: s_topics.txt all

.. code-block:: text

   /clock [rosgraph_msgs/msg/Clock]
   /cmd_vel [geometry_msgs/msg/Twist]
   /imu [sensor_msgs/msg/Imu]
   /joint_states [sensor_msgs/msg/JointState]
   /odom [nav_msgs/msg/Odometry]
   /robot_description [std_msgs/msg/String]
   /scan [sensor_msgs/msg/LaserScan]
   /tf [tf2_msgs/msg/TFMessage]
   /tf_static [tf2_msgs/msg/TFMessage]

``/cmd_vel`` is a ``Twist``, the robot's input; everything else comes out
of the simulation. The arena, ``worlds/arena.sdf``, is a flat floor and
the task site, a 0.4 m box at (3.0, 1.0); its plugins are those of the
sandbox world of ``nav2_minimal_tb3_sim``.

Driving with ros2 topic pub
---------------------------

A ``Twist`` has a linear and an angular velocity, three numbers each:

.. output: i_twist.txt all

.. code-block:: text

   # This expresses velocity in free space broken into its linear and angular parts.

   Vector3  linear
   	float64 x
   	float64 y
   	float64 z
   Vector3  angular
   	float64 x
   	float64 y
   	float64 z

Only ``linear.x`` (forward speed, m/s) and ``angular.z`` (turn rate,
rad/s) move a ground robot. ``odom_reader`` logs the pose and speed once
per second:

.. code: ros/lecture11/competition_motion/competition_motion/odom_reader.py all

.. code-block:: python
   :caption: competition_motion/competition_motion/odom_reader.py

   """Log the robot's pose and speed from /odom once per second (Lecture 11)."""

   import math

   from competition_motion.pose_tools import yaw_from_quaternion
   from nav_msgs.msg import Odometry
   import rclpy
   from rclpy.executors import ExternalShutdownException
   from rclpy.node import Node


   class OdomReader(Node):
       """Keeps the last odometry message and logs it once per second."""

       def __init__(self):
           super().__init__('odom_reader')
           self._last = None
           self._subscription = self.create_subscription(
               Odometry, 'odom', self._on_odom, 10)
           self._timer = self.create_timer(1.0, self._report)

       def _on_odom(self, msg):
           self._last = msg

       def _report(self):
           if self._last is None:
               return
           pose = self._last.pose.pose
           twist = self._last.twist.twist
           yaw = math.degrees(yaw_from_quaternion(pose.orientation))
           self.get_logger().info(  # m, m, degrees, m/s, rad/s
               f'x={pose.position.x:.2f} y={pose.position.y:.2f} '
               f'yaw={yaw:.1f} v={twist.linear.x:.2f} w={twist.angular.z:.2f}')


   def main(args=None):
       rclpy.init(args=args)
       node = OdomReader()
       try:
           rclpy.spin(node)
       except (KeyboardInterrupt, ExternalShutdownException):
           pass
       except Exception:  # Ctrl+C can also end spin() with an RCLError
           if rclpy.ok():  # ROS 2 still running: a real error
               raise
       finally:
           node.destroy_node()
           rclpy.try_shutdown()

Two terminals, the arena running:

.. code-block:: bash

   ros2 run competition_motion odom_reader
   ros2 topic pub -r 10 /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.2}}"

.. output: d_straight.txt all

.. code-block:: text

   [INFO] [odom_reader]: x=0.00 y=0.00 yaw=0.0 v=-0.00 w=-0.00
   [INFO] [odom_reader]: x=0.00 y=0.00 yaw=0.0 v=0.00 w=-0.00
   [INFO] [odom_reader]: x=0.00 y=0.00 yaw=0.0 v=0.00 w=0.00
   [INFO] [odom_reader]: x=0.18 y=0.00 yaw=0.0 v=0.20 w=0.00
   [INFO] [odom_reader]: x=0.38 y=0.00 yaw=0.0 v=0.20 w=0.00
   [INFO] [odom_reader]: x=0.58 y=0.00 yaw=0.0 v=0.20 w=-0.00
   [INFO] [odom_reader]: x=0.78 y=0.00 yaw=0.0 v=0.20 w=-0.00
   [INFO] [odom_reader]: x=0.98 y=0.00 yaw=0.0 v=0.20 w=-0.00
   [INFO] [odom_reader]: x=1.18 y=0.00 yaw=0.0 v=0.20 w=0.00
   [INFO] [odom_reader]: x=1.38 y=0.00 yaw=0.0 v=0.20 w=0.00
   [INFO] [odom_reader]: x=1.58 y=0.00 yaw=0.0 v=0.20 w=0.00
   [INFO] [odom_reader]: x=1.63 y=0.00 yaw=0.0 v=0.00 w=0.00
   [INFO] [odom_reader]: x=1.63 y=0.00 yaw=0.0 v=0.00 w=0.00
   [INFO] [odom_reader]: x=1.63 y=0.00 yaw=0.0 v=0.00 w=0.00

This is one recording, of 10. ``odom_reader`` logs x and y in m, the
yaw in degrees, v in m/s, and ``w`` (ω) in rad/s. The commands:
``ros2 topic pub`` ran 5 s, then nothing was sent for 3 s, then a zero
command (also with ``ros2 topic pub``). The robot covered 1.63 m, not
0.2 × 5 = 1.0 m: it moved for 8.1 s at 0.2 m/s. A recorder of ``/odom``
and the times the script noted split those 8.1 s into 4.1 s while the
first ``ros2 topic pub`` sent (it needed the rest of its 5 s to start),
the 3.0 s pause, and 1.0 s from starting the zero ``ros2 topic pub``
until the robot stopped. Each of the 10 runs, in a new simulation:

.. output: d_straight_runs.txt all

.. code-block:: text

   run  1: 1.65 m; moved 8.2 s = pub 4.1 s + pause 3.0 s + zero 1.1 s
   run  2: 1.63 m; moved 8.1 s = pub 4.1 s + pause 3.0 s + zero 1.0 s
   run  3: 1.63 m; moved 8.1 s = pub 4.1 s + pause 3.0 s + zero 1.0 s
   run  4: 1.63 m; moved 8.1 s = pub 4.1 s + pause 3.0 s + zero 1.0 s
   run  5: 1.64 m; moved 8.1 s = pub 4.1 s + pause 3.0 s + zero 1.0 s
   run  6: 1.64 m; moved 8.1 s = pub 4.1 s + pause 3.0 s + zero 1.0 s
   run  7: 1.61 m; moved 8.0 s = pub 4.0 s + pause 3.0 s + zero 1.0 s
   run  8: 1.65 m; moved 8.2 s = pub 4.1 s + pause 3.0 s + zero 1.1 s
   run  9: 1.63 m; moved 8.1 s = pub 4.1 s + pause 3.0 s + zero 1.0 s
   run 10: 1.63 m; moved 8.1 s = pub 4.1 s + pause 3.0 s + zero 1.0 s

The run shown above is the median one (run 4).

.. warning::

   The robot keeps the last command. Stopping ``ros2 topic pub`` does not
   stop the robot (measured). Send a zero ``Twist``:

   .. code-block:: bash

      ros2 topic pub --once /cmd_vel geometry_msgs/msg/Twist "{}"

RViz2 Shows What ROS 2 Knows
----------------------------

``arena.rviz`` opens four displays. The QoS settings are those the
publishers offer, measured with ``ros2 topic info -v``:

.. output: s_qos.txt all

.. code-block:: text

   /odom
     PUBLISHER ros_gz_bridge: RELIABLE, VOLATILE
   /scan
     PUBLISHER ros_gz_bridge: RELIABLE, VOLATILE
   /tf
     PUBLISHER ros_gz_bridge: RELIABLE, VOLATILE
     PUBLISHER robot_state_publisher: RELIABLE, VOLATILE
   /tf_static
     PUBLISHER robot_state_publisher: RELIABLE, TRANSIENT_LOCAL
   /robot_description
     PUBLISHER robot_state_publisher: RELIABLE, TRANSIENT_LOCAL
   /cmd_vel
     SUBSCRIPTION ros_gz_bridge: RELIABLE, VOLATILE

.. list-table::
   :header-rows: 1
   :widths: 18 27 55
   :class: compact-table

   * - Display
     - Topic
     - What it draws
   * - TF
     - ``/tf``, ``/tf_static``
     - Every frame as three axes. ``/tf_static`` is transient local.
   * - RobotModel
     - ``/robot_description``
     - The robot's links. Transient local: RViz2 gets it even when it
       starts last (Lecture 10, "A Subscriber That Starts Late"). The
       display subscribes transient local too.
   * - LaserScan
     - ``/scan``
     - The lidar's ranges as points. The bridge offers reliable; the
       display asks for best effort, which a reliable publisher serves
       (Lecture 10).
   * - Odometry
     - ``/odom``
     - The pose as arrows, the last 50 kept.

The **Fixed Frame** is ``odom`` in ``arena.rviz``. The RViz User Guide:
"The fixed frame is the reference frame used to denote the world frame."
And: "For correct results, the fixed frame should not be moving relative
to the world." With ``base_footprint`` as the Fixed Frame, the robot would
stay still on the screen and everything else would move around it.

.. code: ros/lecture11/competition_motion/rviz/arena.rviz all

.. code-block:: yaml
   :caption: competition_motion/rviz/arena.rviz

   Panels:
     - Class: rviz_common/Displays
       Name: Displays
     - Class: rviz_common/Views
       Name: Views
   Visualization Manager:
     Class: ""
     Displays:
       - Class: rviz_default_plugins/Grid
         Cell Size: 1
         Enabled: true
         Name: Grid
         Plane: XY
         Plane Cell Count: 10
         Reference Frame: <Fixed Frame>
         Value: true
       - Class: rviz_default_plugins/RobotModel
         Description Source: Topic
         Description Topic:
           Depth: 1
           Durability Policy: Transient Local
           History Policy: Keep Last
           Reliability Policy: Reliable
           Value: /robot_description
         Enabled: true
         Name: RobotModel
         Value: true
         Visual Enabled: true
       - Class: rviz_default_plugins/TF
         Enabled: true
         Frames:
           All Enabled: true
         Marker Scale: 0.5
         Name: TF
         Show Arrows: true
         Show Axes: true
         Show Names: true
         Value: true
       - Class: rviz_default_plugins/LaserScan
         Color: 255; 0; 0
         Color Transformer: FlatColor
         Enabled: true
         Name: LaserScan
         Size (m): 0.05
         Style: Flat Squares
         Topic:
           Depth: 5
           Durability Policy: Volatile
           History Policy: Keep Last
           Reliability Policy: Best Effort
           Value: /scan
         Value: true
       - Class: rviz_default_plugins/Odometry
         Enabled: true
         Keep: 50
         Name: Odometry
         Shape:
           Value: Arrow
         Topic:
           Depth: 5
           Durability Policy: Volatile
           History Policy: Keep Last
           Reliability Policy: Reliable
           Value: /odom
         Value: true
     Enabled: true
     Global Options:
       Background Color: 48; 48; 48
       Fixed Frame: odom
       Frame Rate: 30
     Name: root
     Tools:
       - Class: rviz_default_plugins/MoveCamera
       - Class: rviz_default_plugins/Select
     Value: true
     Views:
       Current:
         Class: rviz_default_plugins/TopDownOrtho
         Name: Current View
         Scale: 100
         X: 1.5
         Y: 0.5
       Saved: ~


The Differential Drive Model
====================================================

Two Wheels on One Axle
----------------------

.. figure:: /_static/images/l11/diff_drive.svg
   :alt: Seen from above, a two-wheeled robot faces right. Its left wheel is at the top and its right wheel at the bottom. Three arrows point right, drawn to scale: tag 1, a short one from the left wheel; tag 3, a middle one, v, from the middle of the axle; tag 2, a long one from the right wheel. Tag 4: a curved arrow, the turn rate omega, around the middle of the axle, counterclockwise. Tag 5: L, the distance between the two wheels. A dashed line goes up from the axle to a dot, the center of the circle the robot drives on (tag 6), at distance R; a second dashed line through the three arrow tips meets it at that dot.
   :width: 55%

   1 and 2: each wheel's speed over the floor. 3: v, the speed of the
   middle of the axle. 4: ω, the turn rate. 5: L, the distance between
   the wheels. 6: the center of the circle the robot drives on, R = v / ω
   from the middle of the axle. The arrows are to scale for v = 0.2 m/s
   and ω = 0.5 rad/s.

A differential drive robot has two driven wheels on one axle, and
casters. Each wheel has its own motor; steering comes from a difference
in speed. v is the speed of the middle of the axle, forward, in m/s; ω is
the turn rate, in rad/s, counterclockwise positive. The Waffle's numbers
come from its file ``gz_waffle.sdf.xacro`` in ``nav2_minimal_tb3_sim``:
``wheel_separation`` 0.287 m, ``wheel_radius`` 0.033 m,
``max_linear_velocity`` 0.46 m/s, and ``max_angular_velocity`` 1.9 rad/s.

From v and ω to Wheel Speeds
----------------------------

Each wheel is L/2 from the middle of the axle. Turning adds ωL/2 to the
right wheel's speed and takes it from the left one's:

.. math::

   v_R = v + \omega \tfrac{L}{2}, \qquad v_L = v - \omega \tfrac{L}{2},
   \qquad v = \tfrac{v_R + v_L}{2}, \qquad \omega = \tfrac{v_R - v_L}{L}

A wheel of radius r that turns at φ̇ rad/s moves r φ̇ m/s over the floor,
so its turn rate is its speed divided by r. For the Waffle, with
v = 0.2 m/s and ω = 0.5 rad/s:

1. ωL/2 = 0.5 × 0.1435 = 0.07175 m/s.
2. Right wheel: 0.2 + 0.07175 = 0.27175 m/s. Left wheel:
   0.2 − 0.07175 = 0.12825 m/s.
3. Turn rates: 0.27175 / 0.033 = 8.235 rad/s, and
   0.12825 / 0.033 = 3.886 rad/s.

The right wheel turns faster, so the robot turns left: ω > 0.

Measured: The Wheels Turn as the Model Says
-------------------------------------------

.. code-block:: bash

   ros2 topic pub -r 10 /cmd_vel geometry_msgs/msg/Twist \
       "{linear: {x: 0.2}, angular: {z: 0.5}}"
   ros2 topic echo --once /joint_states

The lines from ``name`` to ``velocity``:

.. output: d_joint_states.txt all

.. code-block:: text

   name:
   - wheel_left_joint
   - wheel_right_joint
   position:
   - 80.37855713043656
   - 114.97291097419347
   velocity:
   - 3.886363636353046
   - 8.234848484852321

``velocity`` holds the turn rate of each wheel, in rad/s, in the order
of ``name``: 3.886 and 8.235 rad/s, the numbers of step 3 above to 3
decimals. ``position`` is how far each wheel has turned since the start,
in rad; the two differ because of the turns.

A Circle of Radius v / ω
------------------------

The same command, logged by ``odom_reader`` (the first lines are the
robot at rest, where the straight drive left it):

.. output: d_circle.txt all

.. code-block:: text

   [INFO] [odom_reader]: x=1.63 y=0.00 yaw=0.0 v=0.00 w=0.00
   [INFO] [odom_reader]: x=1.63 y=0.00 yaw=0.2 v=0.04 w=0.07
   [INFO] [odom_reader]: x=1.81 y=0.04 yaw=26.8 v=0.20 w=0.50
   [INFO] [odom_reader]: x=1.96 y=0.17 yaw=55.7 v=0.20 w=0.50
   [INFO] [odom_reader]: x=2.03 y=0.36 yaw=83.5 v=0.20 w=0.50
   [INFO] [odom_reader]: x=2.00 y=0.55 yaw=112.4 v=0.20 w=0.50
   [INFO] [odom_reader]: x=1.88 y=0.71 yaw=141.3 v=0.20 w=0.50
   [INFO] [odom_reader]: x=1.70 y=0.79 yaw=170.2 v=0.20 w=0.50
   [INFO] [odom_reader]: x=1.50 y=0.78 yaw=-161.0 v=0.20 w=0.50
   [INFO] [odom_reader]: x=1.34 y=0.67 yaw=-133.1 v=0.20 w=0.50
   [INFO] [odom_reader]: x=1.24 y=0.50 yaw=-104.2 v=0.20 w=0.50
   [INFO] [odom_reader]: x=1.25 y=0.30 yaw=-75.4 v=0.20 w=0.50
   [INFO] [odom_reader]: x=1.34 y=0.12 yaw=-46.5 v=0.20 w=0.50
   [INFO] [odom_reader]: x=1.50 y=0.02 yaw=-18.6 v=0.20 w=0.50
   [INFO] [odom_reader]: x=1.70 y=0.01 yaw=10.2 v=0.20 w=0.50
   [INFO] [odom_reader]: x=1.88 y=0.09 yaw=39.1 v=0.20 w=0.50
   [INFO] [odom_reader]: x=2.00 y=0.25 yaw=68.0 v=0.20 w=0.50
   [INFO] [odom_reader]: x=2.03 y=0.45 yaw=96.9 v=0.20 w=0.50
   [INFO] [odom_reader]: x=1.96 y=0.63 yaw=124.7 v=0.20 w=0.50
   [INFO] [odom_reader]: x=1.91 y=0.68 yaw=135.8 v=0.00 w=0.00
   [INFO] [odom_reader]: x=1.91 y=0.68 yaw=135.8 v=0.00 w=0.00

The model gives R = v / ω = 0.2 / 0.5 = 0.4 m. The robot started at
(1.63, 0.00) facing x, so the center of the circle is 0.4 m to its left,
at (1.63, 0.40). Every logged point is about 0.4 m from it; for example,
(2.03, 0.36) is 2.03 − 1.63 = 0.40 m to the right of the center and
0.40 − 0.36 = 0.04 m below it, so √(0.40² + 0.04²) = 0.40 m away. The
yaw grows by about 29 degrees per second (26.8, 55.7, 83.5): 0.5 rad/s is
28.6 degrees per second.


Odometry
====================================================

Odometry Adds Up Small Steps
----------------------------

**Odometry** estimates the pose from the robot's own motion, here from
how far each wheel turned, with no outside reference. In each short time
Δt, the robot moves vΔt along its heading ψ and turns ωΔt:

.. math::

   x \leftarrow x + v\,\Delta t \cos\psi, \qquad
   y \leftarrow y + v\,\Delta t \sin\psi, \qquad
   \psi \leftarrow \psi + \omega\,\Delta t

How often ``/odom`` arrives (``ros2 topic hz /odom``):

.. output: s_hz_odom.txt all

.. code-block:: text

   average rate: 27.756
   	min: 0.035s max: 0.037s std dev: 0.00052s window: 25

About 27.8 messages per second, so Δt is about 1 / 27.8 = 0.036 s. The
plugin asks for 30 per second (``odom_publish_frequency`` in
``gz_waffle.sdf.xacro``). One step at 0.2 m/s, facing x (ψ = 0), adds
0.2 × 0.036 = 0.0072 m to x and nothing to y. Lecture 5 built a whole
path with ``np.cumsum()`` the same way: a running sum of steps. A sum of
steps keeps every step's error (a wheel that slips, a radius that is
slightly off); nothing in odometry corrects it.

The Odometry Message
--------------------

.. code-block:: bash

   ros2 topic echo --once /odom

The message up to the pose's covariance, recorded at rest at the start:

.. output: o_echo_rest.txt all

.. code-block:: text

   header:
     stamp:
       sec: 54
       nanosec: 324000000
     frame_id: odom
   child_frame_id: base_footprint
   pose:
     pose:
       position:
         x: 2.2371907111862354e-07
         y: 3.3227224807934025e-16
         z: 0.0
       orientation:
         x: 0.0
         y: 0.0
         z: 1.4852209758053142e-09
         w: 1.0

- ``header.frame_id``: the frame the pose is given in, ``odom``.
  ``child_frame_id``: the frame that moves, ``base_footprint``.
- ``stamp``: seconds of **simulation time**, the time Gazebo publishes on
  ``/clock``, not the time of day.
- A node that compares time stamps with the simulation's runs with
  ``use_sim_time:=true``: its clock is then ``/clock``, not the
  computer's. ``arena_launch.py`` sets it for ``robot_state_publisher``
  and RViz2. Exercise 4 shows a lookup that fails without it.
- After ``pose`` come its ``covariance`` (36 numbers) and ``twist`` (v and
  ω) with its own 36. In the message recorded, all 72 were 0:

.. output: o_covariance.txt all

.. code-block:: text

   covariance numbers in one /odom message: 72
   not zero: 0

The whole message, from ``ros2 interface show nav_msgs/msg/Odometry``
with the comments removed:

.. output: i_odometry_fields.txt all

.. code-block:: text

   std_msgs/Header header
     builtin_interfaces/Time stamp
       int32 sec
       uint32 nanosec
     string frame_id
   string child_frame_id
   geometry_msgs/PoseWithCovariance pose
     Pose pose
       Point position
         float64 x
         float64 y
         float64 z
       Quaternion orientation
         float64 x 0
         float64 y 0
         float64 z 0
         float64 w 1
     float64[36] covariance
   geometry_msgs/TwistWithCovariance twist
     Twist twist
       Vector3  linear
         float64 x
         float64 y
         float64 z
       Vector3  angular
         float64 x
         float64 y
         float64 z
     float64[36] covariance

The Same Pose Is Also a Transform
---------------------------------

.. code-block:: bash

   ros2 run tf2_ros tf2_echo odom base_footprint

One reading, after the circle:

.. output: o_echo_tf_end.txt all

.. code-block:: text

   At time 95.616000000
   - Translation: [1.915, 0.683, 0.000]
   - Rotation: in Quaternion (xyzw) [0.000, 0.000, 0.927, 0.376]
   - Rotation: in RPY (radian) [0.000, -0.000, 2.370]
   - Rotation: in RPY (degree) [0.000, -0.000, 135.809]
   - Matrix:
    -0.717 -0.697  0.000  1.915
     0.697 -0.717  0.000  0.683
     0.000  0.000  1.000  0.000
     0.000  0.000  0.000  1.000

``/odom``, recorded just before, with the robot at rest (the pose):

.. output: d_circle_odom.txt all

.. code-block:: text

       position:
         x: 1.9145580497549293
         y: 0.6834043877864573
         z: 0.0
       orientation:
         x: 0.0
         y: -0.0
         z: -0.926558269128124
         w: -0.3761512646666167

The same position, and the same quaternion with all four signs flipped
(q and −q are the same turn). The bridge publishes this pose twice: as a
message on ``/odom``, and as the transform from ``odom`` to
``base_footprint`` on ``/tf``. "Coordinate Frames" below explains the
matrix.

Odometry Drifts: Measured Against Gazebo
----------------------------------------

Gazebo knows where the robot really is, and ``gz model -m
turtlebot3_waffle -p`` prints it. Odometry does not use it. After the
straight drive shown above:

.. output: d_straight_truth.txt all

.. code-block:: text

     - Pose [ XYZ (m) ] [ RPY (rad) ]:
       [1.630280 -0.000003 0.010008]
       [-0.000000 -0.005692 -0.000006]

The first line in brackets is x, y, z; the second is roll, pitch, yaw, in
radians. On the straight line, odometry and Gazebo agree to the 2 decimals
``odom_reader`` logs. After about one and a half circles (19 s of
``ros2 topic pub`` at v = 0.2 m/s and ω = 0.5 rad/s), 8 runs, each in a
new simulation:

.. output: d_drift_runs.txt all

.. code-block:: text

   run 1: odom (-0.056, 0.795) -170.6; gz (-0.003, 0.795) -172.5; 5.3 cm
   run 2: odom (-0.060, 0.795) -170.0; gz (-0.007, 0.795) -171.9; 5.3 cm
   run 3: odom (-0.051, 0.796) -171.2; gz (0.002, 0.796) -173.2; 5.3 cm
   run 4: odom (0.197, 0.751) 152.0; gz (0.243, 0.730) 150.2; 5.0 cm
   run 5: odom (-0.047, 0.797) -171.8; gz (0.006, 0.796) -173.7; 5.3 cm
   run 6: odom (-0.098, 0.786) -164.3; gz (-0.046, 0.790) -166.2; 5.3 cm
   run 7: odom (-0.048, 0.797) -171.7; gz (0.005, 0.796) -173.6; 5.3 cm
   run 8: odom (-0.340, 0.603) -119.8; gz (-0.299, 0.630) -121.9; 4.9 cm

About 5 cm each time: 4.9 to 5.3 cm. Where a run ended varies with how
long ``ros2 topic pub`` took to start, so the runs drove slightly
different arcs. The difference built up in the circles, not on the
straight line.
Odometry assumes that each wheel rolls without slipping; any slip, or any
difference between the robot's real geometry and L and r, ends up in the
pose. REP 105 on the ``odom`` frame: "The pose of a mobile platform in the
odom frame can drift over time, without any bounds." And it "is
guaranteed to be continuous, meaning that the pose of a mobile platform in
the odom frame always evolves in a smooth way, without discrete jumps."
Lecture 12's ``map`` frame corrects the drift.


A Proportional Controller
====================================================

Feedback: Measure, Compare, Command
-----------------------------------

.. figure:: /_static/images/l11/p_loop.svg
   :alt: A feedback loop, left to right. Tag 1: a box, goal. An arrow goes to a dashed box, the node go_to_goal, which holds two boxes: errors(), tag 2, which gives rho and alpha, and _control(), tag 3, which gives v and omega. An arrow goes from _control() through the topic /cmd_vel to tag 4, a box: Gazebo, the robot. From Gazebo an arrow goes down and back left through the topic /odom, tag 5, up into errors().
   :width: 95%

   1: the goal. 2: ``errors()``, which compares it with the pose. 3:
   ``_control()``, which turns the errors into a command. 4: the robot in
   Gazebo. 5: odometry, back on ``/odom``.

Ten times a second (chosen), the node compares the goal with the pose
from ``/odom``: the difference is the **error**. A **proportional**
controller sends a command that is a fixed number, the **gain** k, times
the error: u = k e. A large error gives a large command; no error, no
command. The robot moves, odometry changes, and the next error is
smaller. That loop is **feedback**.

Two Errors: Distance and Heading
--------------------------------

.. figure:: /_static/images/l11/goal_errors.svg
   :alt: Seen from above, in the odom frame, whose x and y axes are drawn at the top left. A robot at the bottom left faces slightly up from the right. Tag 1: the goal, a dot at the top right. Tag 2: rho, a straight line from the robot to the goal. Tag 3: dx and dy, dashed lines along odom's x and y from the robot to the goal. Tag 4: the yaw, an arc from a dashed line parallel to odom's x to the robot's x axis. Tag 5: alpha, an arc from the robot's x axis to the line toward the goal.
   :width: 60%

   1: the goal (x\ :sub:`g`, y\ :sub:`g`). 2: the distance ρ. 3: dx and
   dy. 4: the yaw ψ. 5: the heading error α. All in ``odom``, whose axes
   are at the top left.

- dx = x\ :sub:`g` − x and dy = y\ :sub:`g` − y.
- The distance: ρ = √(dx² + dy²).
- The heading error: α = atan2(dy, dx) − ψ, wrapped to −π to π.
- One gain each: v = k\ :sub:`ρ` ρ and ω = k\ :sub:`α` α.

Why wrap: with a yaw of 170 degrees and the goal in the direction −170
degrees, −170 − 170 = −340 degrees, almost a full turn to the right.
Wrapped, it is +20 degrees: a small turn to the left.

go_to_goal.py
-------------

.. code: ros/lecture11/competition_motion/competition_motion/go_to_goal.py all

.. code-block:: python
   :caption: competition_motion/competition_motion/go_to_goal.py

   """Drive to a goal with a proportional controller, from /odom (Lecture 11)."""

   import math

   from competition_motion.pose_tools import wrap_angle, yaw_from_quaternion
   from geometry_msgs.msg import Twist
   from nav_msgs.msg import Odometry
   import rclpy
   from rclpy.executors import ExternalShutdownException
   from rclpy.node import Node

   PERIOD_S = 0.1  # The control loop runs at 10 Hz (chosen)
   MAX_V_MPS = 0.3  # Under the Waffle's 0.46 m/s (chosen)
   MAX_W_RPS = 1.0  # Under the Waffle's 1.9 rad/s (chosen)
   TURN_FIRST_RAD = math.radians(45)  # Turn in place above this (chosen)


   def clamp(value, limit):
       """Return value, kept between -limit and limit."""
       return max(-limit, min(limit, value))


   class GoToGoal(Node):
       """Turns toward the goal and drives to it, then stops."""

       def __init__(self, name='go_to_goal'):
           super().__init__(name)
           self.declare_parameter('goal_x', 2.0)  # In odom, m (chosen)
           self.declare_parameter('goal_y', 1.0)
           self.declare_parameter('k_rho', 0.5)  # Gain on distance, 1/s
           self.declare_parameter('k_alpha', 1.5)  # Gain on heading, 1/s
           self.declare_parameter('tolerance_m', 0.05)
           self._pose = None  # (x, y, yaw) in odom, from the last message
           self._ticks = 0
           self._done = False
           self._cmd = self.create_publisher(Twist, 'cmd_vel', 10)
           self._odom = self.create_subscription(
               Odometry, 'odom', self._on_odom, 10)
           self._timer = self.create_timer(PERIOD_S, self._control)

       def _on_odom(self, msg):
           p = msg.pose.pose
           self._pose = (p.position.x, p.position.y,
                         yaw_from_quaternion(p.orientation))

       def errors(self):
           """Return (distance, heading error) to the goal, or None."""
           if self._pose is None:
               return None
           x, y, yaw = self._pose
           dx = self.get_parameter('goal_x').value - x
           dy = self.get_parameter('goal_y').value - y
           return math.hypot(dx, dy), wrap_angle(math.atan2(dy, dx) - yaw)

       def _control(self):
           e = None if self._done else self.errors()
           if e is None:
               return
           rho, alpha = e
           cmd = Twist()  # All zeros: stop
           if rho < self.get_parameter('tolerance_m').value:
               self._done = True
               self._cmd.publish(cmd)
               x, y = self._pose[:2] if self._pose else (math.nan, math.nan)
               self.get_logger().info(
                   f'Arrived at t={self._ticks * PERIOD_S:.1f} s: '
                   f'x={x:.2f} y={y:.2f} rho={rho:.3f}')
               return
           k_rho = self.get_parameter('k_rho').value
           k_alpha = self.get_parameter('k_alpha').value
           if abs(alpha) < TURN_FIRST_RAD:
               cmd.linear.x = clamp(k_rho * rho, MAX_V_MPS)
           cmd.angular.z = clamp(k_alpha * alpha, MAX_W_RPS)
           self._cmd.publish(cmd)
           if self._ticks % 10 == 0:  # Once per second of ticks (wall clock)
               self.get_logger().info(  # s, m, degrees, m/s, rad/s
                   f't={self._ticks * PERIOD_S:.1f} rho={rho:.2f} '
                   f'alpha={math.degrees(alpha):.1f} '
                   f'v={cmd.linear.x:.2f} w={cmd.angular.z:.2f}')
           self._ticks += 1


   def main(args=None):
       rclpy.init(args=args)
       node = GoToGoal()
       try:
           rclpy.spin(node)
       except (KeyboardInterrupt, ExternalShutdownException):
           pass
       except Exception:  # Ctrl+C can also end spin() with an RCLError
           if rclpy.ok():  # ROS 2 still running: a real error
               raise
       finally:
           node.destroy_node()
           rclpy.try_shutdown()

- ``_on_odom()`` keeps only the last pose, as a tuple (x, y, ψ); it runs
  about 28 times a second. Each new tuple replaces the last one, whose
  reference count then drops to 0 (the
  :doc:`Memory reading </reading_material/memory/mem_lecture>`,
  "Reference Counting").
- ``errors()`` returns ``None`` until the first ``/odom`` message: the
  node then sends nothing.
- In ``_control()``: close enough (5 cm, chosen), the node sends zeros
  once and stops for good. ``clamp()`` keeps v under 0.3 m/s and ω under
  1.0 rad/s (chosen, under the Waffle's limits). If the goal is more than
  45 degrees (chosen) off the nose, v stays 0: the robot turns in place
  first.
- The goal, the gains, and the tolerance are parameters (Lecture 10):
  ``goal_x`` 2.0, ``goal_y`` 1.0, ``k_rho`` 0.5, ``k_alpha`` 1.5,
  ``tolerance_m`` 0.05, all chosen for this example.
- ``__init__`` takes the node's name as an argument, so that a subclass
  can give its own ("The Controller, with TF2 Doing the Geometry").

Run: From (0, 0) to the Goal (2.0, 1.0)
---------------------------------------

.. code-block:: bash

   ros2 run competition_motion go_to_goal

.. output: p_k0.5_run1.txt all

.. code-block:: text

   [INFO] [go_to_goal]: t=0.0 rho=2.24 alpha=26.6 v=0.30 w=0.70
   [INFO] [go_to_goal]: t=1.0 rho=2.00 alpha=7.7 v=0.30 w=0.20
   [INFO] [go_to_goal]: t=2.0 rho=1.70 alpha=1.7 v=0.30 w=0.04
   [INFO] [go_to_goal]: t=3.0 rho=1.40 alpha=0.4 v=0.30 w=0.01
   [INFO] [go_to_goal]: t=4.0 rho=1.09 alpha=0.1 v=0.30 w=0.00
   [INFO] [go_to_goal]: t=5.0 rho=0.80 alpha=0.0 v=0.30 w=0.00
   [INFO] [go_to_goal]: t=6.0 rho=0.50 alpha=0.0 v=0.25 w=0.00
   [INFO] [go_to_goal]: t=7.0 rho=0.30 alpha=0.0 v=0.15 w=0.00
   [INFO] [go_to_goal]: t=8.0 rho=0.18 alpha=0.0 v=0.09 w=0.00
   [INFO] [go_to_goal]: t=9.0 rho=0.11 alpha=0.0 v=0.05 w=0.00
   [INFO] [go_to_goal]: t=10.0 rho=0.06 alpha=0.0 v=0.03 w=0.00
   [INFO] [go_to_goal]: Arrived at t=10.5 s: x=1.96 y=0.98 rho=0.049

This is run 1 of 3 with the default gains; in the 3 runs, only lines 6
and 7 changed, by 0.01 m. The log: ``t`` in s, ``rho`` in m, ``alpha``
in degrees, ``v`` in m/s, ``w`` (ω) in rad/s. ``t`` counts the node's
0.1 s control steps, on the computer's clock, not simulation time. The
first line, by hand:

1. dx = 2.0 − 0 = 2.0 and dy = 1.0 − 0 = 1.0.
2. ρ = √(4 + 1) = 2.236 m.
3. α = atan2(1, 2) − 0 = 0.464 rad, 26.6 degrees.
4. v = 0.5 × 2.236 = 1.118, clamped to 0.30 m/s.
5. ω = 1.5 × 0.464 = 0.70 rad/s.

Three Gains, Three Runs Each
----------------------------

Each run started a new simulation, with the robot at (0, 0):

.. code-block:: bash

   ros2 run competition_motion go_to_goal --ros-args -p k_rho:=1.0

.. output: p_runs.txt all

.. code-block:: text

   k_rho=0.5 run 1: at 10.5 s; odom (1.96, 0.98); gz (1.959, 0.963)
   k_rho=0.5 run 2: at 10.5 s; odom (1.96, 0.98); gz (1.959, 0.963)
   k_rho=0.5 run 3: at 10.5 s; odom (1.96, 0.98); gz (1.959, 0.963)
   k_rho=1.0 run 1: at 8.3 s; odom (1.96, 0.98); gz (1.959, 0.963)
   k_rho=1.0 run 2: at 8.4 s; odom (1.96, 0.98); gz (1.963, 0.965)
   k_rho=1.0 run 3: at 8.4 s; odom (1.96, 0.98); gz (1.963, 0.965)
   k_rho=0.2 run 1: at 19.5 s; odom (1.96, 0.98); gz (1.958, 0.962)
   k_rho=0.2 run 2: at 19.5 s; odom (1.96, 0.98); gz (1.958, 0.962)
   k_rho=0.2 run 3: at 19.5 s; odom (1.96, 0.98); gz (1.958, 0.962)

- ``at``: the ``t`` of the "Arrived" line (the computer's clock).
  ``odom``: where odometry says
  the robot stopped; ``gz``: Gazebo's own pose of it.
- A larger ``k_rho`` arrives sooner. The three runs of a gain agree
  within 0.2 s.
- While k\ :sub:`ρ` ρ is over 0.3, the clamp holds v at 0.3 m/s: with
  k\ :sub:`ρ` = 0.5, until ρ = 0.3 / 0.5 = 0.6 m.
- After that, v = 0.5 ρ, and the distance shrinks by the same fraction
  each second. The run above logs 0.30, 0.18, and 0.11 m one second
  apart: each about 0.6 times the one before. It would never reach 0;
  that is why the node stops at 5 cm.
- Where odometry says the robot stopped, and where Gazebo says it is,
  differ by about 2 cm in y: the drift of the last section, after a
  curved path.


Coordinate Frames
====================================================

One Robot, Three Frames
-----------------------

.. figure:: /_static/images/l11/frames_top.svg
   :alt: Seen from above, to scale. At the bottom left, the odom frame's x and y axes. A robot on its way, slightly up and to the right of odom, facing up and to the right, with its own axes, labeled base_footprint. Further up and right, the goal frame's axes, labeled goal, and to its right a box, the task site. Tag 1: an arrow from odom to base_footprint. Tag 2: an arrow from odom to goal. Tag 3: a dashed arrow from base_footprint to goal.
   :width: 65%

   1: odometry says where ``base_footprint`` is in ``odom``. 2: the goal
   is given in ``odom``. 3: the controller needs the goal seen from the
   robot. Drawn to scale, with the robot at (1.1, 0.15) facing 30 degrees
   (chosen for the drawing).

A **coordinate frame** is an origin and three axes, with a name.
``odom`` is where the robot started; ``base_footprint`` is on the robot
and moves with it; ``goal`` is where the robot should stop. The lidar
measures each range from its own frame, ``base_scan``: a point it sees
must go through the same kind of chain to land on a map.

REP 105: map, odom, and base_link Form a Tree
---------------------------------------------

- ``base_link`` "is rigidly attached to the mobile robot base". The
  Waffle adds ``base_footprint``, on the floor under it.
- ``odom`` "is a world-fixed frame". The pose in it "can drift over time,
  without any bounds" and "is guaranteed to be continuous".
- ``map`` "is a world fixed frame". The pose in it "should not
  significantly drift over time" and "can change in discrete jumps at any
  time". Lecture 12 builds a map.
- The tree: "The map frame is the parent of odom, and odom is the parent
  of base_link."
- Why not both ``map`` and ``odom`` straight to ``base_link``? REP 105:
  "Although intuition would say that both map and odom should be attached
  to base_link, this is not allowed because each frame can only have one
  parent."

Every frame has one parent, so there is exactly one chain of transforms
from any frame to any other, and TF2 follows that chain.

The Waffle's Frame Tree, Measured
---------------------------------

.. code-block:: bash

   ros2 run tf2_tools view_frames

``view_frames`` listens for 5 s and writes the tree to
``frames_<date>.pdf`` (and ``.gv``). The parent, child, and rate of each
link, from that file:

.. output: f_frames.txt all

.. code-block:: text

   base_footprint -> base_link 10000.0
   base_footprint -> base_link 10000.0
   base_link -> base_scan 10000.0
   base_link -> base_scan 10000.0
   base_link -> camera_link 10000.0
   base_link -> camera_link 10000.0
   base_link -> caster_back_left_link 10000.0
   base_link -> caster_back_left_link 10000.0
   base_link -> caster_back_right_link 10000.0
   base_link -> caster_back_right_link 10000.0
   base_link -> imu_link 10000.0
   base_link -> imu_link 10000.0
   base_link -> wheel_left_link 14.09
   base_link -> wheel_left_link 14.12
   base_link -> wheel_right_link 14.09
   base_link -> wheel_right_link 14.12
   camera_depth_frame -> camera_depth_optical_frame 10000.0
   camera_depth_frame -> camera_depth_optical_frame 10000.0
   camera_link -> camera_depth_frame 10000.0
   camera_link -> camera_depth_frame 10000.0
   camera_link -> camera_rgb_frame 10000.0
   camera_link -> camera_rgb_frame 10000.0
   camera_rgb_frame -> camera_rgb_optical_frame 10000.0
   camera_rgb_frame -> camera_rgb_optical_frame 10000.0
   odom -> base_footprint 27.979
   odom -> base_footprint 28.005

.. figure:: /_static/images/l11/frame_tree.svg
   :alt: A tree of coordinate frames, drawn left to right. odom points to base_footprint (tag 1), which points to base_link. base_link points to seven frames: wheel_left_link and wheel_right_link (tag 2), base_scan, imu_link, camera_link (with four more frames under it), caster_back_left_link and caster_back_right_link. Tag 3 marks the fixed link from base_footprint to base_link. A dashed arrow from odom points down to a dashed box, goal (tag 4).
   :width: 95%

   The same tree, the camera's 4 child frames folded into one box. 1: from
   the bridge on ``/tf``. 2: the wheels, from ``robot_state_publisher`` on
   ``/tf``. 3: ``base_footprint`` to ``base_link``, one of the fixed
   links (the gray arrows) that ``robot_state_publisher`` sends on
   ``/tf_static`` (``view_frames`` shows them with a rate of 10000). 4:
   ``goal``, added by this lecture's ``goal_broadcaster``.

A Transform Is the Child's Pose in the Parent
---------------------------------------------

.. code-block:: bash

   ros2 run tf2_ros tf2_echo base_footprint base_scan

.. output: f_echo_scan.txt all

.. code-block:: text

   At time 0.0
   - Translation: [-0.064, 0.000, 0.132]
   - Rotation: in Quaternion (xyzw) [0.000, 0.000, 0.000, 1.000]
   - Rotation: in RPY (radian) [0.000, -0.000, 0.000]
   - Rotation: in RPY (degree) [0.000, -0.000, 0.000]
   - Matrix:
     1.000  0.000  0.000 -0.064
     0.000  1.000  0.000  0.000
     0.000  0.000  1.000  0.132
     0.000  0.000  0.000  1.000

- ``tf2_echo A B`` printed where B is in A: the lidar is 0.064 m behind
  ``base_footprint`` and 0.132 m above it, not turned. (The tutorial
  "Introducing tf2" names the arguments ``[source_frame] [target_frame]``;
  the recording shows which frame the numbers are given in.)
- The matrix is Lecture 5's homogeneous transform, in 3D: the rotation in
  the top left 3 by 3, the translation in the last column. Lecture 5
  called it ``T_world_robot``; here it is T\ :sub:`footprint, scan`.
- A range of 1.0 m straight ahead of the lidar is at
  x = −0.064 + 1.0 = 0.936 m ahead of ``base_footprint``.

The 0.132 m is a chain of two fixed links: ``base_footprint`` to
``base_link`` (0.010 m up), then ``base_link`` to ``base_scan``:

.. output: f_echo_link_scan.txt 2

.. code-block:: text

   - Translation: [-0.064, 0.000, 0.122]

Chaining: The Inner Names Match
-------------------------------

As in Lecture 5, T\ :sub:`odom, scan` = T\ :sub:`odom, footprint`
T\ :sub:`footprint, scan`. Recorded at the goal, after a run of
``go_to_goal``:

.. output: f_echo_odom_footprint_goal.txt all

.. code-block:: text

   At time 20.340000000
   - Translation: [1.958, 0.977, 0.000]
   - Rotation: in Quaternion (xyzw) [0.000, 0.000, 0.252, 0.968]
   - Rotation: in RPY (radian) [0.000, -0.000, 0.509]
   - Rotation: in RPY (degree) [0.000, -0.000, 29.184]
   - Matrix:
     0.873 -0.488  0.000  1.958
     0.488  0.873  0.000  0.977
     0.000  0.000  1.000  0.000
     0.000  0.000  0.000  1.000

.. output: f_echo_odom_scan_goal.txt all

.. code-block:: text

   At time 28.584000000
   - Translation: [1.902, 0.946, 0.132]
   - Rotation: in Quaternion (xyzw) [0.000, 0.000, 0.252, 0.968]
   - Rotation: in RPY (radian) [0.000, -0.000, 0.509]
   - Rotation: in RPY (degree) [0.000, -0.000, 29.184]
   - Matrix:
     0.873 -0.488  0.000  1.902
     0.488  0.873  0.000  0.946
     0.000  0.000  1.000  0.132
     0.000  0.000  0.000  1.000

1. The lidar is 0.064 m behind ``base_footprint``, along the robot's x,
   which points at the yaw ψ.
2. In ``odom``: x = 1.958 − 0.064 cos ψ = 1.958 − 0.064 × 0.873 = 1.902.
3. y = 0.977 − 0.064 sin ψ = 0.977 − 0.064 × 0.488 = 0.946, what
   ``tf2_echo odom base_scan`` printed.


TF2: Broadcasting and Listening
====================================================

TF2 Keeps the Tree for Every Node
---------------------------------

``ros2 interface show geometry_msgs/msg/TransformStamped``, with the
comments removed:

.. output: i_transform_stamped_fields.txt all

.. code-block:: text

   std_msgs/Header header
     builtin_interfaces/Time stamp
       int32 sec
       uint32 nanosec
     string frame_id
   string child_frame_id
   Transform transform
     Vector3 translation
       float64 x
       float64 y
       float64 z
     Quaternion rotation
       float64 x 0
       float64 y 0
       float64 z 0
       float64 w 1

- The ROS 2 documentation, "Tf2": "tf2 is the transform library, which
  lets the user keep track of multiple coordinate frames over time."
- A **broadcaster** publishes ``TransformStamped`` messages: a parent
  (``frame_id``), a child (``child_frame_id``), and the child's pose in the
  parent. Moving links go on ``/tf``; fixed ones on ``/tf_static``.
- A **listener** subscribes to both topics and fills a **buffer**. Each
  node that listens keeps its own buffer; there is no TF2 node to start.
- The same page, on static transforms: they "are published once and
  assumed to not change, so no history is stored."

A Static Broadcaster for the Goal
---------------------------------

.. code: ros/lecture11/competition_frames/competition_frames/goal_broadcaster.py all

.. code-block:: python
   :caption: competition_frames/competition_frames/goal_broadcaster.py

   """Publish the goal as a frame of its own, once (Lecture 11)."""

   import math

   from competition_motion.pose_tools import quaternion_from_yaw
   from geometry_msgs.msg import TransformStamped
   import rclpy
   from rclpy.executors import ExternalShutdownException
   from rclpy.node import Node
   from tf2_ros import StaticTransformBroadcaster


   class GoalBroadcaster(Node):
       """Sends the static transform from odom to goal at start."""

       def __init__(self):
           super().__init__('goal_broadcaster')
           self.declare_parameter('x', 2.0)  # In odom, m (chosen)
           self.declare_parameter('y', 1.0)
           self.declare_parameter('yaw_deg', 0.0)
           self._broadcaster = StaticTransformBroadcaster(self)
           t = TransformStamped()
           t.header.stamp = self.get_clock().now().to_msg()
           t.header.frame_id = 'odom'  # The parent frame
           t.child_frame_id = 'goal'  # The frame this node adds
           t.transform.translation.x = self.get_parameter('x').value
           t.transform.translation.y = self.get_parameter('y').value
           yaw = math.radians(self.get_parameter('yaw_deg').value)
           r = t.transform.rotation
           r.x, r.y, r.z, r.w = quaternion_from_yaw(yaw)
           self._broadcaster.sendTransform(t)
           self.get_logger().info(
               f'Sent odom -> goal: x={t.transform.translation.x}, '
               f'y={t.transform.translation.y}, yaw={math.degrees(yaw)} deg')


   def main(args=None):
       rclpy.init(args=args)
       node = GoalBroadcaster()
       try:
           rclpy.spin(node)
       except (KeyboardInterrupt, ExternalShutdownException):
           pass
       except Exception:  # Ctrl+C can also end spin() with an RCLError
           if rclpy.ok():  # ROS 2 still running: a real error
               raise
       finally:
           node.destroy_node()
           rclpy.try_shutdown()

The goal becomes a frame of its own, ``goal``, child of ``odom``: one
``sendTransform()``, in ``__init__``, and the node then spins.
``r.x, r.y, r.z, r.w = quaternion_from_yaw(yaw)`` unpacks a tuple into
four attributes, in ROS 2's order, w last. The entry points of the
package:

.. code: ros/lecture11/competition_frames/setup.py 25-31

.. code-block:: python
   :caption: competition_frames/setup.py, entry_points

       entry_points={
           'console_scripts': [
               'goal_broadcaster = competition_frames.goal_broadcaster:main',
               'frame_listener = competition_frames.frame_listener:main',
               'tf_go_to_goal = competition_frames.tf_go_to_goal:main',
           ],
       },

Run It, Then Ask tf2_echo
-------------------------

.. code-block:: bash

   ros2 run competition_frames goal_broadcaster
   ros2 run tf2_ros tf2_echo odom goal

.. output: t_goal_broadcaster.txt all

.. code-block:: text

   [INFO] [goal_broadcaster]: Sent odom -> goal: x=2.0, y=1.0, yaw=0.0 deg

.. output: t_echo_goal.txt all

.. code-block:: text

   At time 0.0
   - Translation: [2.000, 1.000, 0.000]
   - Rotation: in Quaternion (xyzw) [0.000, 0.000, 0.000, 1.000]
   - Rotation: in RPY (radian) [0.000, -0.000, 0.000]
   - Rotation: in RPY (degree) [0.000, -0.000, 0.000]
   - Matrix:
     1.000  0.000  0.000  2.000
     0.000  1.000  0.000  1.000
     0.000  0.000  1.000  0.000
     0.000  0.000  0.000  1.000

The goal is at (2.0, 1.0) in ``odom``, not turned. "At time 0.0": a
static transform holds for every time. Both publishers of ``/tf_static``
offer reliable and transient local (Lecture 10), so a listener that
starts later still gets both (``ros2 topic info -v /tf_static``, the
publishers):

.. output: t_tf_static_qos.txt all

.. code-block:: text

   PUBLISHER goal_broadcaster: RELIABLE, TRANSIENT_LOCAL
   PUBLISHER robot_state_publisher: RELIABLE, TRANSIENT_LOCAL

static_transform_publisher from the Command Line
------------------------------------------------

.. code-block:: bash

   ros2 run tf2_ros static_transform_publisher --x 3.0 --y 1.0 \
       --yaw 1.5708 --frame-id odom --child-frame-id task_site

.. output: t_static_cli.txt all

.. code-block:: text

   [INFO] [static_transform_publisher_...]: Spinning until stopped - publishing transform
   translation: ('3.000000', '1.000000', '0.000000')
   rotation: ('0.000000', '0.000000', '0.707108', '0.707105')
   from 'odom' to 'task_site'

A frame at the task site, turned 90 degrees (π / 2 = 1.5708 rad). The
node's name ends in a random suffix, replaced here by ``...``. The tutorial
"Writing a static broadcaster (Python)": "In ROS 2, roll/pitch/yaw refers
to rotation in radians about the x/y/z-axis, respectively." The rotation
it prints, (0, 0, 0.707, 0.707), is the 90 degree quaternion of "A
Quaternion for a Turn About z". The same tutorial: "static_transform_publisher
is designed both as a command-line tool for manual use, as well as for use
within launch files for setting static transforms."

A Listener Looks Up a Transform
-------------------------------

.. code: ros/lecture11/competition_frames/competition_frames/frame_listener.py all

.. code-block:: python
   :caption: competition_frames/competition_frames/frame_listener.py

   """Look up where one frame is in another, once per second (Lecture 11)."""

   import math

   from competition_motion.pose_tools import yaw_from_quaternion
   import rclpy
   from rclpy.executors import ExternalShutdownException
   from rclpy.node import Node
   from rclpy.time import Time
   from tf2_ros import Buffer, TransformException, TransformListener


   class FrameListener(Node):
       """Logs the pose of the frame source in the frame target."""

       def __init__(self):
           super().__init__('frame_listener')
           self.declare_parameter('target', 'base_footprint')
           self.declare_parameter('source', 'goal')
           self._buffer = Buffer()  # Keeps the transforms it receives
           self._listener = TransformListener(self._buffer, self)
           self._timer = self.create_timer(1.0, self._look_up)

       def _look_up(self):
           target = self.get_parameter('target').value
           source = self.get_parameter('source').value
           try:
               t = self._buffer.lookup_transform(target, source, Time())
           except TransformException as error:
               self.get_logger().warning(f'No transform: {error}')
               return
           p = t.transform.translation
           yaw = math.degrees(yaw_from_quaternion(t.transform.rotation))
           self.get_logger().info(  # m, m, degrees
               f'{source} in {target}: x={p.x:.2f} y={p.y:.2f} yaw={yaw:.1f}')


   def main(args=None):
       rclpy.init(args=args)
       node = FrameListener()
       try:
           rclpy.spin(node)
       except (KeyboardInterrupt, ExternalShutdownException):
           pass
       except Exception:  # Ctrl+C can also end spin() with an RCLError
           if rclpy.ok():  # ROS 2 still running: a real error
               raise
       finally:
           node.destroy_node()
           rclpy.try_shutdown()

- The tutorial "Writing a listener (Python)": once the
  ``TransformListener`` is created, "it starts receiving tf2
  transformations over the wire, and buffers them for up to 10 seconds."
- ``lookup_transform(target, source, time)`` returns the pose of
  ``source`` in ``target``. "Providing rclpy.time.Time() will just get us
  the latest available transform."

Where Is the Goal, Seen from the Robot?
---------------------------------------

The listener started 3.5 s before ``goal_broadcaster`` (folded to fit):

.. code-block:: bash

   ros2 run competition_frames frame_listener

.. output: t_listener_first.txt all

.. code-block:: text

   [WARN] [frame_listener]: No transform: "goal" passed to lookupTransform 
   argument source_frame does not exist. 
   [WARN] [frame_listener]: No transform: "goal" passed to lookupTransform 
   argument source_frame does not exist. 
   [WARN] [frame_listener]: No transform: "goal" passed to lookupTransform 
   argument source_frame does not exist. 
   [INFO] [frame_listener]: goal in base_footprint: x=2.00 y=1.00 yaw=-0.0
   [INFO] [frame_listener]: goal in base_footprint: x=2.00 y=1.00 yaw=-0.0
   [INFO] [frame_listener]: goal in base_footprint: x=2.00 y=1.00 yaw=-0.0
   [INFO] [frame_listener]: goal in base_footprint: x=2.00 y=1.00 yaw=-0.0
   [INFO] [frame_listener]: goal in base_footprint: x=2.00 y=1.00 yaw=-0.0
   [INFO] [frame_listener]: goal in base_footprint: x=2.00 y=1.00 yaw=-0.0

Until the listener received ``goal``, ``lookup_transform()`` raised, and
the node logged a warning and went on. The listener looks up once per
second; in 6 repeats (a new listener, then a new ``goal_broadcaster``
3.5 s later), it logged 3 warnings each time: 2 before the broadcaster
was started, 1 while the broadcaster started up and was found. Other
recordings gave up to 6, when the new node took longer to be found:

.. output: t_listener_warnings.txt all

.. code-block:: text

   repeat 1: 3 warnings, 2 before goal_broadcaster started
   repeat 2: 3 warnings, 2 before goal_broadcaster started
   repeat 3: 3 warnings, 2 before goal_broadcaster started
   repeat 4: 3 warnings, 2 before goal_broadcaster started
   repeat 5: 3 warnings, 2 before goal_broadcaster started
   repeat 6: 3 warnings, 2 before goal_broadcaster started

The robot stands at
the origin of ``odom``, facing x, so the goal seen from it is at
(2.00, 1.00): the same numbers. (−0.0 is a tiny negative number,
rounded.) A second listener, started after the broadcaster, found the
goal at its first lookup, because ``/tf_static`` is transient local:

.. output: t_listener_late.txt all

.. code-block:: text

   [INFO] [frame_listener]: goal in base_footprint: x=2.00 y=1.00 yaw=-0.0
   [INFO] [frame_listener]: goal in base_footprint: x=2.00 y=1.00 yaw=-0.0

When a Lookup Fails
-------------------

.. code-block:: bash

   ros2 run competition_frames frame_listener --ros-args -p source:=gaol

A misspelled frame (folded to fit):

.. output: t_listener_wrong.txt all

.. code-block:: text

   [WARN] [frame_listener]: No transform: "gaol" passed to lookupTransform 
   argument source_frame does not exist. 
   [WARN] [frame_listener]: No transform: "gaol" passed to lookupTransform 
   argument source_frame does not exist. 

A frame nobody broadcasts and a frame not broadcast yet give the same
message: TF2 cannot tell a typo from a late broadcaster.
``TransformException`` is the base class of TF2's five lookup errors
(checked with ``issubclass()``, Lecture 7):

.. output: t_exceptions.txt all

.. code-block:: text

   LookupException True
   ConnectivityException True
   ExtrapolationException True
   InvalidArgumentException True
   TimeoutException True

An ``except`` clause also catches every subclass of its class, so the
listener's one ``except`` catches all five. Exercise 4 meets a third
failure: a transform stamped with another clock.

The Controller, with TF2 Doing the Geometry
-------------------------------------------

.. code: ros/lecture11/competition_frames/competition_frames/tf_go_to_goal.py all

.. code-block:: python
   :caption: competition_frames/competition_frames/tf_go_to_goal.py

   """The go-to-goal controller, with TF2 doing the geometry (Lecture 11)."""

   import math

   from competition_motion.go_to_goal import GoToGoal
   import rclpy
   from rclpy.executors import ExternalShutdownException
   from rclpy.time import Time
   from tf2_ros import Buffer, TransformException, TransformListener


   class TfGoToGoal(GoToGoal):
       """Reads where the goal is from TF2, instead of from /odom."""

       def __init__(self):
           super().__init__('tf_go_to_goal')
           self.declare_parameter('goal_frame', 'goal')
           self._buffer = Buffer()
           self._listener = TransformListener(self._buffer, self)

       def errors(self):
           """Return (distance, heading error) to the goal, or None."""
           goal = self.get_parameter('goal_frame').value
           try:
               t = self._buffer.lookup_transform(
                   'base_footprint', goal, Time())
           except TransformException:
               return None  # No transform yet: send nothing
           p = t.transform.translation  # The goal, seen from the robot
           return math.hypot(p.x, p.y), math.atan2(p.y, p.x)


   def main(args=None):
       rclpy.init(args=args)
       node = TfGoToGoal()
       try:
           rclpy.spin(node)
       except (KeyboardInterrupt, ExternalShutdownException):
           pass
       except Exception:  # Ctrl+C can also end spin() with an RCLError
           if rclpy.ok():  # ROS 2 still running: a real error
               raise
       finally:
           node.destroy_node()
           rclpy.try_shutdown()

``TfGoToGoal`` is a subclass of ``GoToGoal`` (Lecture 7) that overrides
only ``errors()``; ``_control()``, the gains, and the stop are inherited.
Seen from ``base_footprint``, the robot is at (0, 0) facing x: the
distance is √(x² + y²) and the heading error is atan2(y, x). No yaw and no
wrap: TF2 did the rotation. The parameters ``goal_x`` and ``goal_y`` that
it inherits are not used.

Same Goal, Two Controllers
--------------------------

.. code-block:: bash

   ros2 run competition_frames goal_broadcaster
   ros2 run competition_frames tf_go_to_goal

Three runs, each in a new simulation:

.. output: t_tf_runs.txt all

.. code-block:: text

   tf_go_to_goal run 1: at 10.5 s; odom (1.96, 0.98); gz (1.959, 0.963)
   tf_go_to_goal run 2: at 10.5 s; odom (1.96, 0.98); gz (1.959, 0.963)
   tf_go_to_goal run 3: at 10.5 s; odom (1.96, 0.98); gz (1.959, 0.963)

One run in full:

.. output: t_tf_run1.txt all

.. code-block:: text

   [INFO] [tf_go_to_goal]: t=0.0 rho=2.24 alpha=26.6 v=0.30 w=0.70
   [INFO] [tf_go_to_goal]: t=1.0 rho=2.00 alpha=7.6 v=0.30 w=0.20
   [INFO] [tf_go_to_goal]: t=2.0 rho=1.70 alpha=1.6 v=0.30 w=0.04
   [INFO] [tf_go_to_goal]: t=3.0 rho=1.40 alpha=0.4 v=0.30 w=0.01
   [INFO] [tf_go_to_goal]: t=4.0 rho=1.09 alpha=0.1 v=0.30 w=0.00
   [INFO] [tf_go_to_goal]: t=5.0 rho=0.80 alpha=0.0 v=0.30 w=0.00
   [INFO] [tf_go_to_goal]: t=6.0 rho=0.50 alpha=0.0 v=0.25 w=0.00
   [INFO] [tf_go_to_goal]: t=7.0 rho=0.30 alpha=0.0 v=0.15 w=0.00
   [INFO] [tf_go_to_goal]: t=8.0 rho=0.18 alpha=0.0 v=0.09 w=0.00
   [INFO] [tf_go_to_goal]: t=9.0 rho=0.11 alpha=0.0 v=0.05 w=0.00
   [INFO] [tf_go_to_goal]: t=10.0 rho=0.06 alpha=0.0 v=0.03 w=0.00
   [INFO] [tf_go_to_goal]: Arrived at t=10.5 s: x=1.96 y=0.98 rho=0.049

The first line is the one of "Run: From (0, 0) to the Goal (2.0, 1.0)":
ρ = 2.24 m and α = 26.6 degrees. Both controllers stop at the same place,
in about the same time as ``go_to_goal`` with k\ :sub:`ρ` = 0.5. To move
the goal without touching the controller, start ``goal_broadcaster`` with
other ``-p x:=`` and ``-p y:=`` values, or give any other frame with
``-p goal_frame:=``.


Lecture Summary
---------------

- A pose is a position and a quaternion, w last; a yaw ψ is
  (0, 0, sin(ψ/2), cos(ψ/2)).
- Gazebo simulates the robot; the bridge gives ``/odom``, ``/scan``, and
  ``/tf``, and takes ``/cmd_vel``. The robot kept its last command.
- The wheel speeds were those of the differential drive model, to 3
  decimals.
- Odometry was about 5 cm off after one and a half circles (4.9 to
  5.3 cm in 8 runs): it drifts.
- A proportional controller reached the goal; a larger gain arrived
  sooner.
- TF2 keeps the frame tree. A static broadcaster added ``goal``; a
  listener found it from the robot.
