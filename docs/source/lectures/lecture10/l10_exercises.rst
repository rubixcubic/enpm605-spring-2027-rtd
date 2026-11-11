====================================================
Python Exercises
====================================================

Four exercises on :doc:`Lecture 10 <l10_lecture>`. Exercises 1 to 3 are
the exercises from the slides; Exercise 4 is for after class. Do them in
your workspace, ``~/ros_ws``, in terminals where ROS 2 and the workspace
are sourced, ``ROS_DOMAIN_ID`` is your own number, and
``ROS_AUTOMATIC_DISCOVERY_RANGE`` is ``LOCALHOST`` (Lecture 8). The ROS
repository must be pulled and built (its ``lecture10/`` packages).
Exercises 1 and 4 add files to your own package, ``my_competition``,
from Lecture 8's Exercise 2.

.. note::

   Try each exercise before you open its solution. Every output in a
   solution was printed by running the solution, with
   ``ROS_DOMAIN_ID=77`` and the log format without time stamps of the
   lecture page. Exercises 1 and 4 were built in a new workspace, on top
   of the ROS repository's workspace, with ``my_competition`` made as
   Lecture 8's Exercise 2 makes it and Lecture 8's Exercise 3 node added.


----


.. dropdown:: Exercise 1: The Alarm's Threshold as a Parameter
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Declare a parameter with a range, set it when the node starts, and
    read its descriptor.

    **Tasks**

    In ``my_competition``, start from Lecture 8's Exercise 3 node,
    ``low_battery_alarm.py``:

    1. Declare ``threshold_pct``: an integer, default 10, with a
       description and an ``IntegerRange`` from 0 to 100 (both from
       ``rcl_interfaces.msg``). Use it instead of ``LOW_BATTERY_PCT``.
    2. Build, then run the alarm with ``-p threshold_pct:=14``, then
       Lecture 8's ``battery_publisher``. Which message warns, at what
       level? Predict it first: the publisher sends 16%, 14%, 12%, and so
       on.
    3. Run ``ros2 param describe`` on the threshold. What does
       ``-p threshold_pct:=150`` do?

    .. dropdown:: Solution
        :class-container: sd-border-success

        1. ``my_competition/low_battery_alarm.py``:

        .. code: exercises/ex1_low_battery_alarm.py all

        .. code-block:: python
           :caption: my_competition/low_battery_alarm.py

           """Exercise 1: Lecture 8's alarm, with its threshold as a parameter."""

           from rcl_interfaces.msg import IntegerRange, ParameterDescriptor
           import rclpy
           from rclpy.executors import ExternalShutdownException
           from rclpy.node import Node
           from sensor_msgs.msg import BatteryState


           class LowBatteryAlarm(Node):
               """Counts battery messages and warns once below threshold_pct."""

               def __init__(self):
                   super().__init__('low_battery_alarm')
                   self.declare_parameter('threshold_pct', 10, ParameterDescriptor(
                       description='Warn below this battery level, in percent',
                       integer_range=[IntegerRange(from_value=0, to_value=100, step=1)]))
                   self._threshold = self.get_parameter('threshold_pct').value
                   self._count = 0
                   self._warned = False
                   self._subscription = self.create_subscription(
                       BatteryState, 'battery', self._on_battery, 10)

               def _on_battery(self, msg):
                   self._count += 1
                   level = round(msg.percentage * 100)
                   if level < self._threshold and not self._warned:
                       self._warned = True
                       self.get_logger().warning(
                           f'Message {self._count}: {level}%, recharge now')


           def main(args=None):
               rclpy.init(args=args)
               node = LowBatteryAlarm()
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

        The node reads the threshold once, in ``__init__``, as the scout
        does in the lecture; a ``ros2 param set`` would change the
        parameter and not ``self._threshold`` (lecture, "Setting a
        Parameter While the Node Runs"). Add ``<depend>sensor_msgs</depend>``
        to ``package.xml`` if Lecture 8's Exercise 3 did not.

        2. The two commands, the alarm first:

        .. code-block:: bash

           ros2 run my_competition low_battery_alarm --ros-args -p threshold_pct:=14
           ros2 run competition_nodes battery_publisher

        .. output: exercises/ex1_alarm.txt all

        .. code-block:: text

           [WARN] [low_battery_alarm]: Message 3: 12%, recharge now

        16 and 14 are not below 14; 12 is, and it is the third message.

        3. ``ros2 param describe /low_battery_alarm threshold_pct``:

        .. output: exercises/ex1_describe.txt all

        .. code-block:: text

           Parameter name: threshold_pct
             Type: integer
             Description: Warn below this battery level, in percent
             Constraints:
               Min value: 0
               Max value: 100
               Step: 1

        150 is out of the range, so ``declare_parameter()`` raises an
        exception and the node stops (the last line, folded to fit):

        .. output: exercises/ex1_out_of_range.txt all

        .. code-block:: text

           InvalidParameterValueException: Invalid parameter value (150) for 
           parameter. Reason: Parameter threshold_pct out of range. Min: 0, 
           Max: 100, value: 150: threshold_pct


.. dropdown:: Exercise 2: Services and Actions from the Command Line
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Call a service and send action goals without writing code, and
    predict their replies.

    **Tasks**

    Terminal 1: ``ros2 run competition_services task_server --ros-args -p battery_pct:=50``.
    Terminal 2: ``ros2 run competition_actions drive_server``. In terminal 3:

    1. Predict, then call ``/assign_task`` three times, in this order:
       ``pick widget`` for 15 minutes, ``carry crate`` for 10, ``scan
       shelf`` for 2. What does each reply say?
    2. Send a goal of 3.0 m with ``--feedback``. How many feedback
       messages come?
    3. Set ``max_speed_mps`` of ``/drive_server`` to 1.0 with
       ``ros2 param set``, and send the same goal. How many now, and why?

    .. dropdown:: Solution
        :class-container: sd-border-success

        1. The three calls:

        .. code-block:: bash

           ros2 service call /assign_task competition_interfaces/srv/AssignTask "{task: pick widget, minutes: 15}"
           ros2 service call /assign_task competition_interfaces/srv/AssignTask "{task: carry crate, minutes: 10}"
           ros2 service call /assign_task competition_interfaces/srv/AssignTask "{task: scan shelf, minutes: 2}"

        The end of each reply:

        .. output: exercises/ex2_tasks.txt all

        .. code-block:: text

           accepted=True, battery_pct=20, message='done'
           accepted=True, battery_pct=0, message='done'
           accepted=False, battery_pct=0, message='needs_recharge'

        15 minutes use 2 × 15 = 30%: 50 − 30 = 20%. 10 minutes use
        2 × 10 = 20%, all that is left: 20 − 20 = 0%, accepted, because
        the server refuses a task only below 10% at the start or when the
        task needs more than the robot has. At 0%, rule 3 refuses the
        third task.

        2. The goal:

        .. code-block:: bash

           ros2 action send_goal --feedback /drive_to_goal competition_interfaces/action/DriveToGoal "{distance_m: 3.0}"

        .. output: exercises/ex2_goal_fast.txt all

        .. code-block:: text

           Waiting for an action server to become available...
           Sending goal:
                distance_m: 3.0

           Goal accepted with ID: ...

           Feedback:
               distance_remaining_m: 2.0

           Feedback:
               distance_remaining_m: 1.0

           Feedback:
               distance_remaining_m: 0.0

           Result:
               distance_driven_m: 3.0

           Goal finished with status: SUCCEEDED

        3 feedback messages: each step of 0.5 s drives 2.0 × 0.5 = 1.0 m,
        and 3.0 m take 3 steps.

        3. The set, then the same goal:

        .. code-block:: bash

           ros2 param set /drive_server max_speed_mps 1.0

        .. output: exercises/ex2_set.txt all

        .. code-block:: text

           Set parameter successful

        .. output: exercises/ex2_goal_slow.txt all

        .. code-block:: text

           Waiting for an action server to become available...
           Sending goal:
                distance_m: 3.0

           Goal accepted with ID: ...

           Feedback:
               distance_remaining_m: 2.5

           Feedback:
               distance_remaining_m: 2.0

           Feedback:
               distance_remaining_m: 1.5

           Feedback:
               distance_remaining_m: 1.0

           Feedback:
               distance_remaining_m: 0.5

           Feedback:
               distance_remaining_m: 0.0

           Result:
               distance_driven_m: 3.0

           Goal finished with status: SUCCEEDED

        The counts of both goals:

        .. output: exercises/ex2_counts.txt all

        .. code-block:: text

           feedback messages at 2.0 m/s: 3
           feedback messages at 1.0 m/s: 6

        Each step now drives 1.0 × 0.5 = 0.5 m, so 3.0 m take
        3.0 / 0.5 = 6 steps. The server reads the parameter at the start
        of each goal (``_drive()``), so the new value counts for the next
        goal without a callback.


.. dropdown:: Exercise 3: QoS, Predict, Then Run
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Predict whether a publisher and a subscriber connect, and what a late
    subscriber gets.

    **Tasks**

    Start ``status_monitor`` first, then ``status_publisher``, of
    ``competition_qos``, with these ``-p`` values (reliability, then
    durability):

    a. Publisher ``reliable``, ``transient_local``; monitor
       ``best_effort``, ``transient_local``.
    b. Publisher ``best_effort``, ``transient_local``; monitor
       ``best_effort``, ``volatile``.
    c. Publisher ``best_effort``, ``volatile``; monitor ``best_effort``,
       ``transient_local``.
    d. Publisher ``transient_local`` with ``depth:=4``; monitor
       ``transient_local``, started 4 s after the publisher.

    For each pair: do they connect? How many of the 5 messages does the
    monitor log, and which ones in d? Write your answers, then run the
    pairs. A compatible pair can miss messages, even all 5, if the
    publisher sends before the two have found each other: run it again.

    .. dropdown:: Solution
        :class-container: sd-border-success

        Pair a, for example:

        .. code-block:: bash

           ros2 run competition_qos status_monitor --ros-args -p reliability:=best_effort -p durability:=transient_local
           ros2 run competition_qos status_publisher --ros-args -p reliability:=reliable -p durability:=transient_local

        a to c, recorded as in the lecture's "Seven Pairs, Measured":

        .. output: exercises/ex3_pairs.txt all

        .. code-block:: text

           pub_rel     pub_dur         sub_rel     sub_dur         got  warning
           reliable    transient_local best_effort transient_local 5/5  
           best_effort transient_local best_effort volatile        5/5  
           best_effort volatile        best_effort transient_local 0/5  DURABILITY

        - a: the monitor asks for less than the publisher offers on both
          policies, so they connect.
        - b: a transient-local publisher and a volatile monitor connect
          ("New messages only").
        - c: the monitor asks for transient local and the publisher offers
          volatile: no connection, and the warning names ``DURABILITY``.

        d, recorded as in "A Subscriber That Starts Late":

        .. output: exercises/ex3_late.txt all

        .. code-block:: text

           pub_dur         pub_depth sub_dur         sub_depth late monitor got
           transient_local 4         transient_local 10        28% 26% 24% 22%

        The publisher keeps its last 4 messages (its depth), and the late
        monitor, of depth 10, gets all 4: 28% to 22%. The first, 30%, was
        not kept.


.. dropdown:: Exercise 4: An Interface Package and a Service of Your Own (after class)
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Make an interface package, and a service server that uses its type.

    **Tasks**

    1. In ``~/ros_ws/src``, make an ament_cmake package ``my_interfaces``
       with ``srv/Recharge.srv``: the request has ``uint8 target_pct``;
       the response has ``bool accepted`` and ``uint8 battery_pct``. Add
       the lines of the lecture's "CMakeLists.txt and package.xml".
    2. In ``my_competition``, write ``recharge_server.py``: a node
       ``recharge_server`` with a battery at 30% that serves ``recharge``.
       It recharges to ``target_pct`` if that is at least the level now
       and at most 100, and refuses otherwise. Add the entry point and
       ``<depend>my_interfaces</depend>``.
    3. Build, check the type with ``ros2 interface show``, and call the
       service with targets 80, 120, then 40. Predict each reply.

    .. dropdown:: Solution
        :class-container: sd-border-success

        1. Made with ``ros2 pkg create my_interfaces --build-type ament_cmake --license Apache-2.0``;
           the empty ``src/`` and ``include/`` removed, and ``srv/``
           made:

        .. code: exercises/ex4_Recharge.srv all

        .. code-block:: text
           :caption: my_interfaces/srv/Recharge.srv

           # Request
           uint8 target_pct     # The battery level to recharge to
           ---
           # Response
           bool accepted        # False if target_pct is over 100 or below the level now
           uint8 battery_pct    # The battery level after the reply

        In ``CMakeLists.txt``, after ``find_package(ament_cmake REQUIRED)``:

        .. code-block:: cmake

           find_package(rosidl_default_generators REQUIRED)

           rosidl_generate_interfaces(${PROJECT_NAME}
             "srv/Recharge.srv"
           )

        In ``package.xml``, after ``<buildtool_depend>ament_cmake</buildtool_depend>``:

        .. code-block:: xml

           <buildtool_depend>rosidl_default_generators</buildtool_depend>
           <exec_depend>rosidl_default_runtime</exec_depend>

        and just before ``<export>``, after the ``<test_depend>`` lines (the
        order the schema wants, as in the lecture):

        .. code-block:: xml

           <member_of_group>rosidl_interface_packages</member_of_group>

        2. ``my_competition/recharge_server.py``, with
           ``'recharge_server = my_competition.recharge_server:main'`` in
           ``setup.py`` and ``<depend>my_interfaces</depend>`` in
           ``package.xml``:

        .. code: exercises/ex4_recharge_server.py all

        .. code-block:: python
           :caption: my_competition/recharge_server.py

           """Exercise 4: a service that recharges a robot's battery."""

           from my_interfaces.srv import Recharge
           import rclpy
           from rclpy.executors import ExternalShutdownException
           from rclpy.node import Node


           class RechargeServer(Node):
               """Recharges to target_pct if it is at least the level now, up to 100."""

               def __init__(self):
                   super().__init__('recharge_server')
                   self._battery = 30  # Percent, chosen for this exercise
                   self._service = self.create_service(
                       Recharge, 'recharge', self._recharge)

               def _recharge(self, request, response):
                   if self._battery <= request.target_pct <= 100:
                       self._battery = request.target_pct
                       response.accepted = True
                   response.battery_pct = self._battery
                   self.get_logger().info(
                       f'target {request.target_pct}%: battery {self._battery}%')
                   return response


           def main(args=None):
               rclpy.init(args=args)
               node = RechargeServer()
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

        3. ``colcon build --symlink-install`` builds ``my_interfaces``
           before ``my_competition``, because of the ``<depend>``:

        .. output: exercises/ex4_build.txt all

        .. code-block:: text

           Starting >>> my_interfaces
           Finished <<< my_interfaces [-.--s]
           Starting >>> my_competition
           Finished <<< my_competition [-.--s]
           Summary: 2 packages finished [-.--s]

        ``ros2 interface show my_interfaces/srv/Recharge``:

        .. output: exercises/ex4_show.txt all

        .. code-block:: text

           # Request
           uint8 target_pct     # The battery level to recharge to
           ---
           # Response
           bool accepted        # False if target_pct is over 100 or below the level now
           uint8 battery_pct    # The battery level after the reply

        The three calls,
        ``ros2 service call /recharge my_interfaces/srv/Recharge "{target_pct: 80}"``
        and the same with 120 and 40; the end of each reply:

        .. output: exercises/ex4_calls.txt all

        .. code-block:: text

           accepted=True, battery_pct=80
           accepted=False, battery_pct=80
           accepted=False, battery_pct=80

        80 is between 30 and 100: accepted. 120 is over 100: refused, the
        battery stays at 80. 40 is below the level now, 80: refused. The
        server's log:

        .. output: exercises/ex4_server.txt all

        .. code-block:: text

           [INFO] [recharge_server]: target 80%: battery 80%
           [INFO] [recharge_server]: target 120%: battery 80%
           [INFO] [recharge_server]: target 40%: battery 80%
