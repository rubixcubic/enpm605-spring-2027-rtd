====================================================
Python Exercises
====================================================

Four exercises on :doc:`Lecture 8 <l8_lecture>`. Exercises 1 to 3 are
the exercises from the slides; Exercise 4 is for after class. Do them in
your workspace, ``~/ros_ws``, in terminals where ROS 2 is sourced and
``ROS_DOMAIN_ID`` is set to your own number (the
:doc:`lecture page <l8_lecture>`, "Discovery: Nodes on the Same Domain
Find Each Other", says how to choose it). Exercises 2 to 4 add nodes to
your own package, ``my_competition``, in ``~/ros_ws/src``.

.. note::

   Try each exercise before you open its solution. Every output in a
   solution was printed by running the solution, with
   ``ROS_DOMAIN_ID=77`` and the log format without time stamps of the
   lecture page. The solution files of Exercises 3 and 4 were run with
   ``python3 file.py``, the Python that ``ros2 run`` uses; each sets its
   node's name, so the log lines are the same as with ``ros2 run``.


----


.. dropdown:: Exercise 1: Inspect the Talker and the Listener
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Run two nodes that come with ROS 2, and answer questions about them
    with the ``ros2`` command.

    **Tasks**

    In three terminals, each with ROS 2 sourced and your domain set:

    1. Start the listener, then the talker, with ``ros2 run``.
    2. Which topic does ``/listener`` subscribe to, and with which
       message type?
    3. How many publishers and how many subscribers does ``/chatter``
       have? Start a second listener in a fourth terminal: what changes?
    4. Stop the talker with Ctrl+C. What does the listener print? Which
       rule of "Four Rules of a Topic" does this show?
    5. Print one message of ``/chatter`` with ``ros2 topic echo``.

    A second listener started with the same command has the same node
    name, ``listener``. ROS 2 runs both, and ``ros2 node list`` warns
    about the repeated name; Lecture 9 shows how to rename a node.

    .. dropdown:: Solution
        :class-container: sd-border-success

        1. ``ros2 run demo_nodes_py listener`` in one terminal, then
           ``ros2 run demo_nodes_py talker`` in another.
        2. ``ros2 node info /listener`` lists one subscriber,
           ``/chatter`` with type ``std_msgs/msg/String``:

        .. output: node_info_listener.txt 1-3

        .. code-block:: text

           /listener
             Subscribers:
               /chatter: std_msgs/msg/String

        3. ``ros2 topic info /chatter`` gives 1 publisher and 1
           subscriber. With a second listener running, the subscription
           count is 2:

        .. output: exercises/ex1_topic_info_two.txt all

        .. code-block:: text

           Type: std_msgs/msg/String
           Publisher count: 1
           Subscription count: 2

        ``ros2 node list`` shows ``/listener`` twice, after a warning:

        .. output: exercises/ex1_node_list_two.txt all

        .. code-block:: text

           WARNING: Be aware that there are nodes in the graph that share an exact name, which can have unintended side effects.
           /listener
           /listener
           /talker

        4. The listener prints nothing, and no error: a subscriber waits
           when no one sends (rule 3, "zero or more" of each). Run alone
           for 4 s, the listener printed no line at all.
        5. ``ros2 topic echo --once /chatter`` prints one message,
           then ``---``. Which number the message has depends on when
           you run it. Started before the talker, with the type, it gets
           the first one:

        .. output: topic_echo.txt all

        .. code-block:: text

           data: 'Hello World: 0'
           ---


.. dropdown:: Exercise 2: Your First Package
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Make a Python package, build it with ``colcon``, and change a node
    without building again.

    **Tasks**

    In ``~/ros_ws/src``, with ROS 2 sourced:

    1. Make a package with a starter executable:

       .. code-block:: bash

          ros2 pkg create my_competition --build-type ament_python \
              --dependencies rclpy --node-name hello_node

    2. Open ``setup.py``: which entry point did ``--node-name`` add?
    3. In ``~/ros_ws``, build with ``colcon build --symlink-install``,
       source ``install/setup.bash``, and run ``hello_node``.
    4. Rewrite ``my_competition/hello_node.py`` as a node, like
       ``minimal_node.py``, that logs ``Hello from my_competition``. Run
       it again without building.

    Add your name and email with ``--maintainer-name`` and
    ``--maintainer-email``, or edit ``package.xml`` afterwards.

    .. dropdown:: Solution
        :class-container: sd-border-success

        The commands, with a name and an email chosen for this example
        (``example.com`` is a domain kept for examples):

        .. code-block:: bash

           cd ~/ros_ws/src
           ros2 pkg create my_competition --build-type ament_python \
               --dependencies rclpy --node-name hello_node --license Apache-2.0 \
               --maintainer-name "Your Name" --maintainer-email you@example.com
           cd ~/ros_ws
           colcon build --symlink-install
           source install/setup.bash
           ros2 run my_competition hello_node

        ``--node-name hello_node`` added the entry point
        ``'hello_node = my_competition.hello_node:main'`` to
        ``setup.py``, and the file ``my_competition/hello_node.py``,
        whose ``main()`` prints ``Hi from my_competition.``. The new
        ``hello_node.py``:

        .. code: exercises/ex2_hello_node.py all

        .. code-block:: python
           :caption: my_competition/hello_node.py

           """Exercise 2: hello_node.py of the package my_competition, as a node."""

           import rclpy


           def main(args=None):
               rclpy.init(args=args)
               node = rclpy.create_node('hello_node')
               node.get_logger().info('Hello from my_competition')
               node.destroy_node()
               rclpy.shutdown()


           if __name__ == '__main__':
               main()

        The two runs, before and after the file was replaced, with no
        build between them:

        .. output: exercises/ex2_hello.txt all

        .. code-block:: text

           Hi from my_competition.
           [INFO] [hello_node]: Hello from my_competition

        1. The first line is the generated ``print()``; the second is the
           node's log line, with its severity and node name.
        2. The second run used the new file without a build: with
           ``--symlink-install``, the installed module is a link to the
           file in ``src``.
        3. A new entry point would need ``colcon build`` again: the
           build writes the executable.


.. dropdown:: Exercise 3: A Low-Battery Alarm
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Write a subscriber that keeps state between messages.

    **Tasks**

    Add the node to your package ``my_competition`` (Exercise 2):

    .. code-block:: python

       # 1. Write low_battery_alarm.py: a class LowBatteryAlarm(Node),
       #    named 'low_battery_alarm', that subscribes to /battery
       #    (sensor_msgs/msg/BatteryState, queue size 10).
       # 2. Its callback counts the messages it receives. The first time
       #    the level is below 10% (rule 3), it logs a warning once:
       #    'Message 5: 8%, recharge now'. It logs nothing else.
       # 3. Add main() as in heartbeat_node.py, an entry point in
       #    setup.py, and sensor_msgs in package.xml; build again.
       # Test: run battery_monitor, low_battery_alarm, then
       #   battery_publisher of competition_nodes, each in a terminal.

    Keep the count and a "warned already" flag as attributes of the
    node, as ``HeartbeatNode`` keeps ``self._count``.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex3_low_battery_alarm.py all

        .. code-block:: python
           :caption: my_competition/low_battery_alarm.py

           """Exercise 3: warn once, the first time the battery drops below 10%."""

           import rclpy
           from rclpy.executors import ExternalShutdownException
           from rclpy.node import Node
           from sensor_msgs.msg import BatteryState

           LOW_BATTERY_PCT = 10  # Rule 3 of Lecture 6


           class LowBatteryAlarm(Node):
               """Counts battery messages and warns once below LOW_BATTERY_PCT."""

               def __init__(self):
                   super().__init__('low_battery_alarm')
                   self._count = 0
                   self._warned = False
                   self._subscription = self.create_subscription(
                       BatteryState, 'battery', self._on_battery, 10)

               def _on_battery(self, msg):
                   self._count += 1
                   level = round(msg.percentage * 100)
                   if level < LOW_BATTERY_PCT and not self._warned:
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

        In ``setup.py``, the new entry point is
        ``'low_battery_alarm = my_competition.low_battery_alarm:main'``,
        and ``package.xml`` gets ``<depend>sensor_msgs</depend>``. The
        alarm's whole output:

        .. output: exercises/ex3_alarm.txt all

        .. code-block:: text

           [WARN] [low_battery_alarm]: Message 5: 8%, recharge now

        The monitor, running at the same time, logged its usual lines:

        .. output: exercises/ex3_monitor.txt all

        .. code-block:: text

           [INFO] [battery_monitor]: 16%: active
           [INFO] [battery_monitor]: 14%: active
           [INFO] [battery_monitor]: 12%: active
           [INFO] [battery_monitor]: 10%: active
           [WARN] [battery_monitor]: 8%: needs_recharge

        and ``ros2 topic info /battery`` showed two subscribers:

        .. output: exercises/ex3_topic_info.txt all

        .. code-block:: text

           Type: sensor_msgs/msg/BatteryState
           Publisher count: 1
           Subscription count: 2

        1. The publisher sends 16, 14, 12, 10, 8 (rule 4). The fifth
           message, 8%, is the first below 10, so the warning says
           ``Message 5``.
        2. ``self._warned`` becomes ``True`` at the first warning, so 6%
           and the levels after it log nothing.
        3. Both subscribers received every message (rule 4): the alarm
           counted to 5 while the monitor logged the same five levels.


.. dropdown:: Exercise 4: A Sonar and an Obstacle Watcher (After Class)
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Write a publisher and a subscriber for a message type you read with
    ``ros2 interface show``.

    **Tasks**

    .. code-block:: python

       # 1. Read sensor_msgs/msg/Range with ros2 interface show.
       # 2. Write sonar_publisher.py: every 0.5 s (chosen for this
       #    exercise), it publishes a Range on
       #    front_range for Lecture 7's 4 m sonar: radiation_type
       #    ULTRASOUND, min_range 0.02, max_range 4.0, and range taken in
       #    turn from [2.5, 1.5, 0.9, 0.6] (meters, chosen for this
       #    exercise). It logs 'Published <range> m'.
       # 3. Write obstacle_watcher.py: it subscribes to front_range,
       #    skips a range outside min_range to max_range, and logs a
       #    warning 'Obstacle at <range> m' below 1.0 m (Lecture 7's
       #    obstacle_within(1.0)), else 'Clear: <range> m', with two
       #    decimals.
       # Test: run the watcher, then the publisher.

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code: exercises/ex4_sonar_publisher.py all

        .. code-block:: python
           :caption: my_competition/sonar_publisher.py

           """Exercise 4: publish sonar readings as sensor_msgs/msg/Range."""

           import rclpy
           from rclpy.executors import ExternalShutdownException
           from rclpy.node import Node
           from sensor_msgs.msg import Range

           READINGS_M = [2.5, 1.5, 0.9, 0.6]  # Chosen for this exercise


           class SonarPublisher(Node):
               """Publishes one reading every 0.5 s on front_range, in a loop."""

               def __init__(self):
                   super().__init__('sonar_publisher')
                   self._index = 0
                   self._publisher = self.create_publisher(Range, 'front_range', 10)
                   # One reading every 0.5 s (chosen for this exercise)
                   self._timer = self.create_timer(0.5, self._publish_range)

               def _publish_range(self):
                   msg = Range()
                   msg.radiation_type = Range.ULTRASOUND
                   msg.min_range = 0.02  # Chosen for this exercise
                   msg.max_range = 4.0  # The 4 m sonar of Lecture 7
                   msg.range = READINGS_M[self._index % len(READINGS_M)]
                   self._publisher.publish(msg)
                   self.get_logger().info(f'Published {msg.range} m')
                   self._index += 1


           def main(args=None):
               rclpy.init(args=args)
               node = SonarPublisher()
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

        .. code: exercises/ex4_obstacle_watcher.py all

        .. code-block:: python
           :caption: my_competition/obstacle_watcher.py

           """Exercise 4: warn when the sonar sees something closer than 1 m."""

           import rclpy
           from rclpy.executors import ExternalShutdownException
           from rclpy.node import Node
           from sensor_msgs.msg import Range

           LIMIT_M = 1.0  # The limit of Lecture 7's obstacle_within(1.0)


           class ObstacleWatcher(Node):
               """Logs each range; a warning when it is below LIMIT_M."""

               def __init__(self):
                   super().__init__('obstacle_watcher')
                   self._subscription = self.create_subscription(
                       Range, 'front_range', self._on_range, 10)

               def _on_range(self, msg):
                   if not msg.min_range <= msg.range <= msg.max_range:
                       return  # Range.msg: such values should be discarded
                   if msg.range < LIMIT_M:
                       self.get_logger().warning(f'Obstacle at {msg.range:.2f} m')
                   else:
                       self.get_logger().info(f'Clear: {msg.range:.2f} m')


           def main(args=None):
               rclpy.init(args=args)
               node = ObstacleWatcher()
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

        The publisher and the watcher, first four lines of each:

        .. output: exercises/ex4_sonar.txt all

        .. code-block:: text

           [INFO] [sonar_publisher]: Published 2.5 m
           [INFO] [sonar_publisher]: Published 1.5 m
           [INFO] [sonar_publisher]: Published 0.9 m
           [INFO] [sonar_publisher]: Published 0.6 m

        .. output: exercises/ex4_watcher.txt all

        .. code-block:: text

           [INFO] [obstacle_watcher]: Clear: 2.50 m
           [INFO] [obstacle_watcher]: Clear: 1.50 m
           [WARN] [obstacle_watcher]: Obstacle at 0.90 m
           [WARN] [obstacle_watcher]: Obstacle at 0.60 m

        1. ``Range.msg`` says "values < range_min or > range_max should
           be discarded"; the watcher's first ``if`` does that.
        2. 2.5 and 1.5 are not below 1.0: ``Clear``. 0.9 and 0.6 are:
           ``WARN``.
        3. ``range`` is a ``float32``, so the watcher receives
           0.8999999761581421 for 0.9; ``:.2f`` prints ``0.90``.
