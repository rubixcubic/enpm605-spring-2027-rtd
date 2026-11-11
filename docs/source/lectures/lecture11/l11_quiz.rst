====================================================
Quiz
====================================================

This self-check quiz covers :doc:`Lecture 11 <l11_lecture>`.

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

   In which order does a ROS 2 ``geometry_msgs/msg/Quaternion`` hold its
   four numbers?

   A. (w, x, y, z)
   B. (x, y, z, w)
   C. (roll, pitch, yaw, w)
   D. It depends on the node that publishes it

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**

   *Explanation:* The tutorial "Quaternion fundamentals": "In ROS 2, w is last, but in some libraries like Eigen, w can be placed at the first position." ("A Quaternion for a Turn About z").


.. admonition:: Question 2
   :class: hint

   Which quaternion (x, y, z, w) is a yaw of 90 degrees?

   A. (0, 0, 1, 0)
   B. (0, 0, 0.707, 0.707)
   C. (0.707, 0, 0, 0.707)
   D. (0, 0, 0.5, 0.5)

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**

   *Explanation:* A yaw ψ is (0, 0, sin(ψ/2), cos(ψ/2)); half of 90 is 45, and sin 45° = cos 45° = 0.707. ``pose_demo.py`` printed this line ("Five Yaws, Measured"). (0, 0, 1, 0) is 180 degrees, and (0, 0, 0.5, 0.5) does not have length 1.


.. admonition:: Question 3
   :class: hint

   ``ros2 topic pub -r 10 /cmd_vel ...`` sends 0.2 m/s to the Waffle in
   the arena. You stop ``ros2 topic pub`` and send nothing else. What
   does the robot do?

   A. It stops at once
   B. It slows down and stops within 1 s
   C. It keeps driving at 0.2 m/s
   D. It drives backward until it stops

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**

   *Explanation:* In the recording shown, the robot kept moving at 0.2 m/s through the 3 s pause after ``ros2 topic pub`` stopped, and 1.0 s more until the zero command arrived: 1.63 m in all (1.61 to 1.65 m in 10 runs), for 5 s of ``ros2 topic pub`` ("Driving with ros2 topic pub"). Send a zero ``Twist`` to stop.


.. admonition:: Question 4
   :class: hint

   The Waffle (L = 0.287 m, r = 0.033 m) gets v = 0.2 m/s and ω = 0.5
   rad/s. How fast does the right wheel turn?

   A. 3.886 rad/s
   B. 6.061 rad/s
   C. 8.235 rad/s
   D. 0.272 rad/s

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**

   *Explanation:* v + ωL/2 = 0.2 + 0.07175 = 0.27175 m/s, and 0.27175 / 0.033 = 8.235 rad/s; ``/joint_states`` reported 8.2348... ("Measured: The Wheels Turn as the Model Says"). 3.886 is the left wheel, and 0.272 is the right wheel's speed in m/s, not its turn rate.


.. admonition:: Question 5
   :class: hint

   The same command, v = 0.2 m/s and ω = 0.5 rad/s, is held. On what
   radius of circle does the middle of the axle drive?

   A. 0.1 m
   B. 0.287 m
   C. 0.4 m
   D. 2.5 m

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**

   *Explanation:* R = v / ω = 0.2 / 0.5 = 0.4 m; the logged points were about 0.4 m from the center ("A Circle of Radius v / ω").


.. admonition:: Question 6
   :class: hint

   Which sentence about the ``odom`` frame does REP 105 support?

   A. The pose in it never drifts
   B. The pose in it can jump when a sensor corrects it
   C. The pose in it can drift without bounds, and changes smoothly, without jumps
   D. It is attached to the robot

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**

   *Explanation:* REP 105: the pose in ``odom`` "can drift over time, without any bounds" and "is guaranteed to be continuous". Jumps belong to ``map``; the frame on the robot is ``base_link`` ("REP 105: map, odom, and base_link Form a Tree").


.. admonition:: Question 7
   :class: hint

   ``go_to_goal`` runs with ``k_rho`` 0.5. The goal is 0.4 m straight
   ahead (heading error 0). Which ``v`` does it send?

   A. 0.2 m/s
   B. 0.3 m/s
   C. 0.4 m/s
   D. 0.8 m/s

.. dropdown:: Answer
   :class-container: sd-border-success

   **A**

   *Explanation:* v = 0.5 × 0.4 = 0.2 m/s, under the 0.3 m/s clamp, so it is sent as it is. The run logged ``rho=0.30 ... v=0.15`` the same way ("Run: From (0, 0) to the Goal (2.0, 1.0)").


.. admonition:: Question 8
   :class: hint

   The robot's yaw is 170 degrees, and the direction to the goal is −170
   degrees. Which heading error does ``errors()`` return, in degrees?

   A. −340
   B. −20
   C. +20
   D. +340

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**

   *Explanation:* −170 − 170 = −340, and ``wrap_angle()`` brings it to −340 + 360 = +20: a small turn to the left ("Two Errors: Distance and Heading"). The test ``test_wrap_angle`` checks −340 to 20.


.. admonition:: Question 9
   :class: hint

   A node calls ``self._buffer.lookup_transform('base_footprint',
   'goal', Time())``. What does it get?

   A. The pose of ``goal`` in ``base_footprint``, the latest available
   B. The pose of ``base_footprint`` in ``goal``
   C. The transform at the time 0 s of the simulation
   D. The transform the node itself last broadcast

.. dropdown:: Answer
   :class-container: sd-border-success

   **A**

   *Explanation:* The first argument is the target, the second the source: it returns the source's pose in the target. The tutorial: "Providing rclpy.time.Time() will just get us the latest available transform." ``frame_listener`` printed ``goal in base_footprint: x=2.00 y=1.00`` ("Where Is the Goal, Seen from the Robot?").


.. admonition:: Question 10
   :class: hint

   ``goal_broadcaster`` sent its transform once, at start. A
   ``frame_listener`` started several seconds later found ``goal`` at
   its first lookup. Why?

   A. The broadcaster sends the transform again every second
   B. ``/tf_static`` is transient local, so a late subscriber gets the last message
   C. The listener's buffer keeps 10 s of the past
   D. ``tf2_echo`` keeps the transform for other nodes

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**

   *Explanation:* ``ros2 topic info -v /tf_static`` showed both publishers RELIABLE and TRANSIENT_LOCAL (Lecture 10's durability). The listener's buffer only holds what it received after it started ("Run It, Then Ask tf2_echo").


True/False
==========

.. admonition:: Statement 1
   :class: hint

   A robot that only turns about z can store its orientation in a
   ``geometry_msgs/msg/Pose`` as one number, its yaw.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* ``Pose`` has one way to store an orientation, a ``Quaternion`` of four numbers ("A Pose Is a Position and an Orientation"). For a pure yaw, x and y are 0, but z and w are both needed.


.. admonition:: Statement 2
   :class: hint

   ``ros2 run tf2_ros tf2_echo base_footprint base_scan`` printed the
   pose of ``base_scan`` in ``base_footprint``: 0.064 m behind it and
   0.132 m above it.

.. dropdown:: Answer
   :class-container: sd-border-success

   **True**

   *Explanation:* The recording printed ``Translation: [-0.064, 0.000, 0.132]`` ("A Transform Is the Child's Pose in the Parent").


.. admonition:: Statement 3
   :class: hint

   With the robot at the origin of ``odom``, and ``task_site`` at (3.0,
   1.0) turned 90 degrees, the robot seen from ``task_site`` is at (−3,
   −1).

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* The inverse turns the translation too: ``tf2_echo task_site base_footprint`` printed ``[-1.000, 3.000, 0.000]`` (Exercise 2).


.. admonition:: Statement 4
   :class: hint

   In the simulation, a node that broadcasts a moving frame on ``/tf``
   can stamp its transforms with the computer's clock, as long as it
   runs on the same computer as the listener.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* The carrot of Exercise 4, a moving frame on ``/tf`` stamped with the computer's clock, could not be looked up with the robot's transforms, stamped with the simulation's clock: "Lookup would require extrapolation into the past". With ``use_sim_time:=true``, it worked. A static transform on ``/tf_static`` is different: it holds for every time, so ``goal_broadcaster``'s goal, stamped with the computer's clock, was found ("Where Is the Goal, Seen from the Robot?").
