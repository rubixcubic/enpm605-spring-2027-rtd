====================================================
Quiz
====================================================

This self-check quiz covers :doc:`Lecture 8 <l8_lecture>`.

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

   A talker runs in a terminal with ``ROS_DOMAIN_ID=77``. In a second
   terminal, with ``ROS_DOMAIN_ID=78``, you run ``ros2 node list``. What
   does it print?

   A. ``/talker``
   B. Nothing
   C. An error: the domain is wrong
   D. ``/talker`` with a warning about the domain

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, nothing

   *Explanation:* Nodes on different domains do not discover each other. Domain 78 has no nodes, so the list is empty; it is not an error.


.. admonition:: Question 2
   :class: hint

   Which file of a Python package says that ``ros2 run my_pkg talker``
   runs the function ``main()`` of ``my_pkg/talker.py``?

   A. ``package.xml``
   B. ``setup.cfg``
   C. ``setup.py``
   D. ``resource/my_pkg``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``setup.py``

   *Explanation:* Its ``entry_points``, under ``console_scripts``, hold lines of the form ``talker = my_pkg.talker:main``. ``setup.cfg`` only says where the executables are installed.


.. admonition:: Question 3
   :class: hint

   A node with no namespace calls
   ``self.create_publisher(BatteryState, 'battery', 10)``. What is the
   full name of the topic?

   A. ``battery``
   B. ``/battery``
   C. ``/battery_publisher/battery``
   D. ``/sensor_msgs/battery``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``/battery``

   *Explanation:* A name without a leading ``/`` is completed with ``/``; ``ros2 topic list`` shows ``/battery`` while ``battery_publisher`` runs.


.. admonition:: Question 4
   :class: hint

   In a node's ``main()``, which call must come before the node is made?

   A. ``rclpy.spin(node)``
   B. ``rclpy.init(args=args)``
   C. ``node.destroy_node()``
   D. ``rclpy.try_shutdown()``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``rclpy.init(args=args)``

   *Explanation:* ``rclpy.init()`` starts ROS 2 in the process; a node is made after it, then spun, then destroyed, then ROS 2 is shut down.


.. admonition:: Question 5
   :class: hint

   The battery publisher sends a battery of 10% as
   ``msg.percentage = 10 / 100``. The field is a ``float32``, so the
   monitor receives 0.10000000149011612. What does the monitor of the
   lecture log? (Rule 3: below 10%, the status is ``needs_recharge``.)

   A. ``[WARN] [battery_monitor]: 10%: needs_recharge``
   B. ``[INFO] [battery_monitor]: 10%: active``
   C. ``[INFO] [battery_monitor]: 9%: active``
   D. ``[WARN] [battery_monitor]: 9%: needs_recharge``

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``10%: active``

   *Explanation:* 0.10000000149011612 × 100 rounds to 10, and 10 is not below 10. The fourth line of the monitor's output in the lecture is this case.


.. admonition:: Question 6
   :class: hint

   Which command prints the fields of the message type
   ``sensor_msgs/msg/Range``?

   A. ``ros2 topic info sensor_msgs/msg/Range``
   B. ``ros2 topic echo sensor_msgs/msg/Range``
   C. ``ros2 interface show sensor_msgs/msg/Range``
   D. ``ros2 node info sensor_msgs/msg/Range``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**, ``ros2 interface show``

   *Explanation:* ``topic`` and ``node`` commands take a topic or a node name. ``interface show`` prints the ``.msg`` definition of a type.


.. admonition:: Question 7
   :class: hint

   A workspace was built with ``colcon build --symlink-install``. You
   change a log message in a node's ``.py`` file. What must you do
   before ``ros2 run`` uses the new message?

   A. Nothing: run it again
   B. ``colcon build`` again
   C. ``source install/setup.bash`` again
   D. Delete ``build``, ``install``, and ``log``, then build

.. dropdown:: Answer
   :class-container: sd-border-success

   **A**, nothing

   *Explanation:* With ``--symlink-install``, the installed module is a link to the file in ``src``. Exercise 2's solution changed ``hello_node.py`` and ran it again without a build. A new entry point would need a build.


.. admonition:: Question 8
   :class: hint

   A node runs ``rclpy.spin(node)`` and you press Ctrl+C. What does
   ``spin()`` raise in ROS 2 Jazzy?

   A. ``ExternalShutdownException``
   B. ``KeyboardInterrupt``
   C. ``RCLError``
   D. Nothing: it returns normally

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**, ``KeyboardInterrupt``

   *Explanation:* ``spin_probe.py`` printed ``spin raised KeyboardInterrupt`` for Ctrl+C, and ``ExternalShutdownException`` for ``SIGTERM``. ``RCLError`` came from a second ``rclpy.shutdown()``.


.. admonition:: Question 9
   :class: hint

   In ``__init__`` of a node class,
   ``self._timer = self.create_timer(0.5, self._tick)``. What happens?

   A. ``self._tick()`` runs once now, then every 0.5 s
   B. ``self._tick()`` runs every 0.5 s while the node spins
   C. ``self._tick()`` runs every 0.5 s, even if the node never spins
   D. ``self._tick()`` runs once, 0.5 s from now

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**

   *Explanation:* The timer's callback is called by ``spin()``; without spinning, it never runs. ``self._tick`` is passed, not called, so nothing runs at once.


.. admonition:: Question 10
   :class: hint

   Which domain ID does a ROS 2 node use when ``ROS_DOMAIN_ID`` is not
   set?

   A. 0
   B. 1
   C. 77
   D. 101

.. dropdown:: Answer
   :class-container: sd-border-success

   **A**, 0

   *Explanation:* The ROS 2 docs: "All ROS 2 nodes use domain ID 0 by default." 77 is the value chosen for the lecture's recordings; 101 is the top of the docs' safe range.


True or False
=============

.. admonition:: Question 11
   :class: hint

   **True or False:** A publisher waits until a subscriber exists before
   it sends its first message.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* "There may be zero or more publishers and zero or more subscribers on any particular topic." A publisher sends even when no one listens. The lecture starts each subscriber first so that it gets the first message.


.. admonition:: Question 12
   :class: hint

   **True or False:** With the default settings, ``msg.percentage =
   "full"`` on a ``BatteryState`` raises an error at that line.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* ``message_demo.py`` printed ``full``: the assignment is not checked. The error comes at ``publish()``, which stops the process, or at the assignment if ``ROS_PYTHON_CHECK_FIELDS=1`` is set.


.. admonition:: Question 13
   :class: hint

   **True or False:** After you source ROS 2 and then your workspace,
   ``AMENT_PREFIX_PATH`` lists your workspace's package before
   ``/opt/ros/jazzy``.

.. dropdown:: Answer
   :class-container: sd-border-success

   **True**

   *Explanation:* The overlay is searched first: ``.../ros_ws/install/competition_nodes:/opt/ros/jazzy``.


.. admonition:: Question 14
   :class: hint

   **True or False:** ``ros2 run`` runs a node with the Python of the
   course's uv environment, ``.venv``.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* The executable that ``colcon`` writes starts with ``#!/usr/bin/python3``: Ubuntu's Python. A package added with ``uv add`` is not seen by a node.
