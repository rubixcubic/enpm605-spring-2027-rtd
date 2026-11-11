====================================================
Python Exercises
====================================================

Four exercises on :doc:`Lecture 11 <l11_lecture>`. Exercises 1 and 2 are
the exercises from the slides; Exercises 3 and 4 are for after class. Do
them in your workspace, ``~/ros_ws``, in terminals where ROS 2 and the
workspace are sourced, ``ROS_DOMAIN_ID`` is your own number, and
``ROS_AUTOMATIC_DISCOVERY_RANGE`` is ``LOCALHOST`` (Lecture 8). The ROS
repository must be pulled and built (its ``lecture11/`` packages), and
Exercises 2 to 4 need the arena running:

.. code-block:: bash

   ros2 launch competition_motion arena_launch.py

.. note::

   Try each exercise before you open its solution. Every output in a
   solution was printed by running the solution, with
   ``ROS_DOMAIN_ID=77``, the log format without time stamps of the
   lecture page, and the arena started without its windows. Exercise 4's
   node was run by path with ``python3``; in your own package, give it an
   entry point instead.


----


.. dropdown:: Exercise 1: Quaternions and Wheel Speeds
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Convert a yaw to a quaternion by hand, and use the differential drive
    model.

    **Tasks**

    By hand first, then check with ``pose_tools`` in Python:

    1. Write the quaternion (x, y, z, w) for a yaw of 135 degrees, then
       for −45 degrees.
    2. The Waffle gets v = 0.1 m/s and ω = −0.5 rad/s. Which wheel turns
       faster? Compute both turn rates in rad/s.
    3. What is the radius of the circle it drives on, and is the center
       to its left or to its right?

    Use L = 0.287 m and r = 0.033 m (lecture, "Two Wheels on One Axle").

    .. dropdown:: Solution
        :class-container: sd-border-success

        1. A yaw ψ gives (0, 0, sin(ψ/2), cos(ψ/2)).

           - 135 degrees: half is 67.5; sin 67.5° = 0.924 and
             cos 67.5° = 0.383, so (0, 0, 0.924, 0.383).
           - −45 degrees: half is −22.5; sin(−22.5°) = −0.383 and
             cos(−22.5°) = 0.924, so (0, 0, −0.383, 0.924).

        2. ωL/2 = −0.5 × 0.1435 = −0.07175 m/s, a negative number.

           - Right wheel: 0.1 + (−0.07175) = 0.02825 m/s, so
             0.02825 / 0.033 = 0.856 rad/s.
           - Left wheel: 0.1 − (−0.07175) = 0.17175 m/s, so
             0.17175 / 0.033 = 5.205 rad/s.

           The left wheel turns faster: the robot turns right, as
           ω < 0 says.

        3. R = 0.1 / 0.5 = 0.2 m: v over the size of ω, without its sign.
           The robot turns right, so the center is 0.2 m to its right.

        The check, ``ex1_check.py``, run by path in a terminal where the
        workspace is sourced:

        .. code: exercises/ex1_check.py all

        .. code-block:: python
           :caption: ex1_check.py

           """Exercise 1: check the hand answers with pose_tools and the model."""

           import math

           from competition_motion.pose_tools import quaternion_from_yaw

           for degrees in (135, -45):
               q = quaternion_from_yaw(math.radians(degrees))
               print(f'yaw {degrees:4} deg: (x, y, z, w) =', tuple(round(v, 3) for v in q))

           R_M, L_M = 0.033, 0.287  # Waffle wheel radius and separation
           v, w = 0.1, -0.5  # The command of task 2, m/s and rad/s
           right = (v + w * L_M / 2) / R_M  # Wheel speeds, rad/s
           left = (v - w * L_M / 2) / R_M
           print(f'right wheel {right:.3f} rad/s, left wheel {left:.3f} rad/s')
           print(f'radius of the circle: {abs(v / w):.3f} m')

        .. output: exercises/ex1_check.txt all

        .. code-block:: text

           yaw  135 deg: (x, y, z, w) = (0.0, 0.0, 0.924, 0.383)
           yaw  -45 deg: (x, y, z, w) = (0.0, 0.0, -0.383, 0.924)
           right wheel 0.856 rad/s, left wheel 5.205 rad/s
           radius of the circle: 0.200 m


.. dropdown:: Exercise 2: Predict, Then Ask tf2_echo
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Predict a transform, and see that the inverse of a transform is not
    its translation with the signs flipped.

    **Tasks**

    The arena running, the robot at the start:

    1. Publish the frame ``task_site`` with the command of the lecture's
       "static_transform_publisher from the Command Line".
    2. Predict what ``ros2 run tf2_ros tf2_echo base_footprint task_site``
       prints: the translation and the quaternion. Then run it.
    3. Predict the translation for
       ``ros2 run tf2_ros tf2_echo task_site base_footprint``: where is the
       robot, seen from the task site? Then run it. Is the answer
       (−3, −1)?

    .. dropdown:: Solution
        :class-container: sd-border-success

        1. The command:

        .. code-block:: bash

           ros2 run tf2_ros static_transform_publisher --x 3.0 --y 1.0 \
               --yaw 1.5708 --frame-id odom --child-frame-id task_site

        2. The robot is at the origin of ``odom``, facing x, so
           ``base_footprint`` and ``odom`` are the same frame:
           ``task_site`` is at (3.0, 1.0), turned 90 degrees, quaternion
           (0, 0, 0.707, 0.707). One reading:

        .. output: exercises/ex2_echo_task_site.txt all

        .. code-block:: text

           At time 35.136000000
           - Translation: [3.000, 1.000, 0.000]
           - Rotation: in Quaternion (xyzw) [0.000, 0.000, 0.707, 0.707]
           - Rotation: in RPY (radian) [0.000, -0.000, 1.571]
           - Rotation: in RPY (degree) [0.000, -0.000, 90.000]
           - Matrix:
            -0.000 -1.000  0.000  3.000
             1.000 -0.000  0.000  1.000
             0.000  0.000  1.000  0.000
             0.000  0.000  0.000  1.000

        3. Not (−3, −1). Seen from ``odom``, the step from the task site
           to the robot is (−3, −1). The task site's own axes are turned
           90 degrees: its x points along ``odom``'s y, and its y along
           ``odom``'s −x. So, in its axes:

           - along its x (``odom``'s y): −1;
           - along its y (``odom``'s −x): −(−3) = 3.

           The robot is at (−1, 3), turned −90 degrees:

        .. output: exercises/ex2_echo_reverse.txt all

        .. code-block:: text

           At time 41.724000000
           - Translation: [-1.000, 3.000, 0.000]
           - Rotation: in Quaternion (xyzw) [0.000, 0.000, -0.707, 0.707]
           - Rotation: in RPY (radian) [0.000, 0.000, -1.571]
           - Rotation: in RPY (degree) [0.000, 0.000, -90.000]
           - Matrix:
            -0.000  1.000  0.000 -1.000
            -1.000 -0.000 -0.000  3.000
            -0.000  0.000  1.000  0.000
             0.000  0.000  0.000  1.000

        The inverse of a transform turns the translation as well as
        flipping it. TF2 does this for you whenever a lookup goes up the
        tree.


.. dropdown:: Exercise 3: Gains in the Arena
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Predict how the gain changes a proportional controller, then measure
    it.

    **Tasks**

    1. ``go_to_goal`` clamps v at 0.3 m/s. For ``k_rho`` = 1.0, 0.5, and
       0.2, compute the distance under which the clamp stops holding.
    2. Predict the order in which the three gains arrive.
    3. Run ``go_to_goal`` with each gain, restarting the arena before each
       run so that the robot starts at (0, 0):

       .. code-block:: bash

          ros2 run competition_motion go_to_goal --ros-args -p k_rho:=1.0

       Note the time of the "Arrived" line. With ``k_rho:=0.2``, how many
       seconds does the last meter take?

    .. dropdown:: Solution
        :class-container: sd-border-success

        1. The clamp holds while k\ :sub:`ρ` ρ > 0.3, so it stops at
           ρ = 0.3 / k\ :sub:`ρ`:

           - k\ :sub:`ρ` = 1.0: 0.3 / 1.0 = 0.3 m;
           - k\ :sub:`ρ` = 0.5: 0.3 / 0.5 = 0.6 m;
           - k\ :sub:`ρ` = 0.2: 0.3 / 0.2 = 1.5 m.

        2. All three start at 0.3 m/s. The larger the gain, the shorter
           the slow part at the end: 1.0 first, then 0.5, then 0.2.

        3. Three runs per gain, each in a new simulation (the lecture's
           "Three Gains, Three Runs Each"):

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

        The run with ``k_rho:=0.2``, in full:

        .. output: p_k0.2_run1.txt all

        .. code-block:: text

           [INFO] [go_to_goal]: t=0.0 rho=2.24 alpha=26.6 v=0.30 w=0.70
           [INFO] [go_to_goal]: t=1.0 rho=1.99 alpha=7.4 v=0.30 w=0.19
           [INFO] [go_to_goal]: t=2.0 rho=1.70 alpha=1.7 v=0.30 w=0.04
           [INFO] [go_to_goal]: t=3.0 rho=1.40 alpha=0.4 v=0.28 w=0.01
           [INFO] [go_to_goal]: t=4.0 rho=1.14 alpha=0.1 v=0.23 w=0.00
           [INFO] [go_to_goal]: t=5.0 rho=0.93 alpha=0.0 v=0.19 w=0.00
           [INFO] [go_to_goal]: t=6.0 rho=0.76 alpha=0.0 v=0.15 w=0.00
           [INFO] [go_to_goal]: t=7.0 rho=0.62 alpha=0.0 v=0.12 w=0.00
           [INFO] [go_to_goal]: t=8.0 rho=0.51 alpha=0.0 v=0.10 w=0.00
           [INFO] [go_to_goal]: t=9.0 rho=0.41 alpha=0.0 v=0.08 w=0.00
           [INFO] [go_to_goal]: t=10.0 rho=0.34 alpha=0.0 v=0.07 w=0.00
           [INFO] [go_to_goal]: t=11.0 rho=0.28 alpha=0.0 v=0.06 w=0.00
           [INFO] [go_to_goal]: t=12.0 rho=0.23 alpha=0.0 v=0.05 w=0.00
           [INFO] [go_to_goal]: t=13.0 rho=0.18 alpha=0.0 v=0.04 w=0.00
           [INFO] [go_to_goal]: t=14.0 rho=0.15 alpha=0.0 v=0.03 w=0.00
           [INFO] [go_to_goal]: t=15.0 rho=0.12 alpha=0.0 v=0.02 w=0.00
           [INFO] [go_to_goal]: t=16.0 rho=0.10 alpha=0.0 v=0.02 w=0.00
           [INFO] [go_to_goal]: t=17.0 rho=0.08 alpha=0.0 v=0.02 w=0.00
           [INFO] [go_to_goal]: t=18.0 rho=0.07 alpha=0.0 v=0.01 w=0.00
           [INFO] [go_to_goal]: t=19.0 rho=0.05 alpha=0.0 v=0.01 w=0.00
           [INFO] [go_to_goal]: Arrived at t=19.5 s: x=1.96 y=0.98 rho=0.049

        The last meter: ``rho`` is 1.14 m at ``t=4.0`` and 0.93 m at
        ``t=5.0``, so it passes 1.0 m about (1.14 − 1.0) / (1.14 − 0.93)
        = 0.67 s after ``t=4.0``, at 4.7 s. The robot arrives at 19.5 s,
        so the last meter takes about 19.5 − 4.7 = 14.8 s, while the first
        1.24 m (2.24 − 1.0) took 4.7 s.
        With the clamp no longer holding (it stops at 1.5 m, task 1), the
        speed is 0.2 ρ: 0.19 m/s at 0.93 m, 0.02 m/s at 0.12 m.


.. dropdown:: Exercise 4: A Frame That Moves
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Write a broadcaster whose transform changes, meet the clock of the
    simulation, and follow a moving goal.

    **Tasks**

    In your own package, ``my_competition``:

    1. Write ``carrot_broadcaster.py``: a node with a
       ``TransformBroadcaster`` (from ``tf2_ros``) and a timer at 10 Hz
       that sends ``odom`` to ``carrot``. The carrot goes around the task
       site, (3.0, 1.0), on a circle of radius 1.0 m, once a minute,
       starting on the side of the robot. Use the tutorial "Writing a
       broadcaster (Python)" and ``quaternion_from_yaw``.
    2. Run it, then
       ``ros2 run competition_frames frame_listener --ros-args -p source:=carrot``.
       What does the listener print? Read the message: which two times
       does it compare?
    3. Run the broadcaster again with ``--ros-args -p use_sim_time:=true``
       and the listener again. What changed?
    4. Run ``ros2 run competition_frames tf_go_to_goal --ros-args -p goal_frame:=carrot``
       for 40 s. Does the robot ever log "Arrived"? Compare its
       ``rho`` with the carrot's speed divided by ``k_rho``.

    .. dropdown:: Solution
        :class-container: sd-border-success

        1. The node:

        .. code: exercises/ex4_carrot_broadcaster.py all

        .. code-block:: python
           :caption: my_competition/carrot_broadcaster.py

           """Exercise 4: a carrot frame that circles the task site, 10 times a second."""

           import math

           from competition_motion.pose_tools import quaternion_from_yaw
           from geometry_msgs.msg import TransformStamped
           import rclpy
           from rclpy.executors import ExternalShutdownException
           from rclpy.node import Node
           from tf2_ros import TransformBroadcaster

           CENTER = (3.0, 1.0)  # The task site, in odom, m
           RADIUS_M = 1.0  # Chosen for this exercise
           TURN_S = 60.0  # One turn per minute (chosen): about 0.1 m/s


           class CarrotBroadcaster(Node):
               """Sends odom -> carrot, a point that moves on a circle."""

               def __init__(self):
                   super().__init__('carrot_broadcaster')
                   self._broadcaster = TransformBroadcaster(self)
                   self._start = None  # Set at the first send: the clock is ready then
                   self._timer = self.create_timer(0.1, self._send)

               def _send(self):
                   now = self.get_clock().now()
                   if self._start is None:
                       self._start = now
                   elapsed = (now - self._start).nanoseconds / 1e9
                   angle = math.pi + 2 * math.pi * elapsed / TURN_S  # Start west of it
                   t = TransformStamped()
                   t.header.stamp = now.to_msg()  # Sim time with use_sim_time:=true
                   t.header.frame_id = 'odom'
                   t.child_frame_id = 'carrot'
                   t.transform.translation.x = CENTER[0] + RADIUS_M * math.cos(angle)
                   t.transform.translation.y = CENTER[1] + RADIUS_M * math.sin(angle)
                   r = t.transform.rotation
                   r.x, r.y, r.z, r.w = quaternion_from_yaw(angle + math.pi / 2)
                   self._broadcaster.sendTransform(t)


           def main(args=None):
               rclpy.init(args=args)
               node = CarrotBroadcaster()
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


           if __name__ == '__main__':
               main()

        2. The listener's warnings (folded to fit):

        .. output: exercises/ex4_wall_clock.txt all

        .. code-block:: text

           [WARN] [frame_listener]: No transform: Lookup would require 
           extrapolation into the past.  Requested time 8.712000 but the earliest 
           data is at time 1792267302.977787, when looking up transform from frame 
           [carrot] to frame [base_footprint]
           [WARN] [frame_listener]: No transform: Lookup would require 
           extrapolation into the past.  Requested time 9.684000 but the earliest 
           data is at time 1792267302.977787, when looking up transform from frame 
           [carrot] to frame [base_footprint]
           [WARN] [frame_listener]: No transform: Lookup would require 
           extrapolation into the past.  Requested time 10.692000 but the earliest 
           data is at time 1792267302.977787, when looking up transform from frame 
           [carrot] to frame [base_footprint]
           [WARN] [frame_listener]: No transform: Lookup would require 
           extrapolation into the past.  Requested time 11.700000 but the earliest 
           data is at time 1792267302.977787, when looking up transform from frame 
           [carrot] to frame [base_footprint]
           [WARN] [frame_listener]: No transform: Lookup would require 
           extrapolation into the past.  Requested time 12.708000 but the earliest 
           data is at time 1792267302.977787, when looking up transform from frame 
           [carrot] to frame [base_footprint]
           [WARN] [frame_listener]: No transform: Lookup would require 
           extrapolation into the past.  Requested time 13.716000 but the earliest 
           data is at time 1792267302.977787, when looking up transform from frame 
           [carrot] to frame [base_footprint]

        The carrot is stamped with the computer's clock (seconds since
        1970), and the robot's transforms with the simulation's clock
        (seconds since Gazebo started). A lookup of the latest transform
        needs one time for the whole chain, and no time is in both: the
        lookup fails.

        3. With ``use_sim_time:=true``, the node's clock is the
        simulation's, from ``/clock``, and the stamps agree:

        .. output: exercises/ex4_sim_time.txt all

        .. code-block:: text

           [INFO] [frame_listener]: carrot in base_footprint: x=2.05 y=0.68 yaw=-71.3
           [INFO] [frame_listener]: carrot in base_footprint: x=2.09 y=0.58 yaw=-65.3

        4. The robot follows, and never logs "Arrived":

        .. output: exercises/ex4_follow.txt all

        .. code-block:: text

           [INFO] [tf_go_to_goal]: t=0.0 rho=2.20 alpha=12.6 v=0.30 w=0.33
           [INFO] [tf_go_to_goal]: t=1.0 rho=2.01 alpha=1.3 v=0.30 w=0.03
           [INFO] [tf_go_to_goal]: t=2.0 rho=1.77 alpha=-1.5 v=0.30 w=-0.04
           [INFO] [tf_go_to_goal]: t=3.0 rho=1.53 alpha=-2.0 v=0.30 w=-0.05
           [INFO] [tf_go_to_goal]: t=4.0 rho=1.31 alpha=-2.1 v=0.30 w=-0.06
           [INFO] [tf_go_to_goal]: t=5.0 rho=1.10 alpha=-2.1 v=0.30 w=-0.05
           [INFO] [tf_go_to_goal]: t=6.0 rho=0.90 alpha=-1.9 v=0.30 w=-0.05
           [INFO] [tf_go_to_goal]: t=7.0 rho=0.70 alpha=-1.6 v=0.30 w=-0.04
           [INFO] [tf_go_to_goal]: t=8.0 rho=0.51 alpha=-1.1 v=0.26 w=-0.03
           [INFO] [tf_go_to_goal]: t=9.0 rho=0.38 alpha=0.0 v=0.19 w=0.00
           [INFO] [tf_go_to_goal]: t=10.0 rho=0.31 alpha=1.5 v=0.16 w=0.04
           [INFO] [tf_go_to_goal]: t=11.0 rho=0.27 alpha=2.8 v=0.13 w=0.07
           [INFO] [tf_go_to_goal]: t=12.0 rho=0.24 alpha=3.7 v=0.12 w=0.10
           [INFO] [tf_go_to_goal]: t=13.0 rho=0.23 alpha=4.2 v=0.11 w=0.11
           [INFO] [tf_go_to_goal]: t=14.0 rho=0.22 alpha=4.3 v=0.11 w=0.11
           [INFO] [tf_go_to_goal]: t=15.0 rho=0.21 alpha=4.3 v=0.11 w=0.11
           [INFO] [tf_go_to_goal]: t=16.0 rho=0.21 alpha=4.2 v=0.11 w=0.11
           [INFO] [tf_go_to_goal]: t=17.0 rho=0.21 alpha=4.1 v=0.11 w=0.11
           [INFO] [tf_go_to_goal]: t=18.0 rho=0.21 alpha=4.1 v=0.10 w=0.11
           [INFO] [tf_go_to_goal]: t=19.0 rho=0.21 alpha=4.0 v=0.10 w=0.11
           [INFO] [tf_go_to_goal]: t=20.0 rho=0.21 alpha=4.0 v=0.10 w=0.11
           [INFO] [tf_go_to_goal]: t=21.0 rho=0.21 alpha=4.0 v=0.10 w=0.11
           [INFO] [tf_go_to_goal]: t=22.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=23.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=24.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=25.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=26.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=27.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=28.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=29.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=30.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=31.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=32.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=33.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=34.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=35.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=36.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=37.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=38.0 rho=0.21 alpha=4.0 v=0.10 w=0.10
           [INFO] [tf_go_to_goal]: t=39.0 rho=0.21 alpha=4.0 v=0.10 w=0.10

        The carrot moves at 2π × 1.0 / 60 = 0.105 m/s. Once the robot
        trails it at a steady distance, the robot must move at the
        carrot's speed too, and the controller sends v = 0.5 ρ. So
        0.5 ρ = 0.105, and ρ = 0.105 / 0.5 = 0.21 m: always more than the
        5 cm tolerance. A proportional controller that follows a moving
        goal trails it, by more when the gain is smaller.
