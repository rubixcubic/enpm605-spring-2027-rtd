====================================================
Quiz
====================================================

This self-check quiz covers :doc:`Lecture 10 <l10_lecture>`.

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

   A node never calls ``declare_parameter('top_speed', ...)``. You start
   it with ``--ros-args -p top_speed:=1.0``. What happens?

   A. The node stops with an exception
   B. The node runs, and ``top_speed`` is ignored, with no warning
   C. The node runs, and ``top_speed`` is declared with the value 1.0
   D. The node runs, and a warning says ``top_speed`` is not declared

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**

   *Explanation:* A node accepts only the parameters it declares. The scout ran with ``-p top_speed:=1.0`` and logged its usual speed; nothing was printed about ``top_speed`` ("Declaring and Reading a Parameter").


.. admonition:: Question 2
   :class: hint

   The scout declares ``max_speed_mps`` with the default ``2.0``. You
   start it with ``--ros-args -p max_speed_mps:=2``. What happens?

   A. The node runs at 2.0 m/s: 2 is turned into 2.0
   B. The node runs at 2.0 m/s, after a warning
   C. ``declare_parameter()`` raises an exception, and the node stops
   D. The node runs with the default, and ignores the 2

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**

   *Explanation:* The default ``2.0`` makes the parameter a double, and ``2`` is an integer. rclpy raised ``InvalidParameterTypeException`` and the process exited with failure 1 ("A Wrong Value Stops the Node at Start"). Write ``2.0``.


.. admonition:: Question 3
   :class: hint

   ``ScoutNode`` copies ``max_speed_mps`` into ``self._speed`` in
   ``__init__`` and logs ``self._speed`` once per second. While it runs,
   ``ros2 param set /scout max_speed_mps 1.0`` prints ``Set parameter
   successful``. What do ``ros2 param get`` and the log show next?

   A. ``get``: 1.0; the log: 1.0
   B. ``get``: 1.0; the log: still 2.0
   C. ``get``: 2.0; the log: 2.0
   D. ``get``: 2.0; the log: 1.0

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**

   *Explanation:* The parameter changed, and nothing told the node, which keeps the copy it made at start ("Setting a Parameter While the Node Runs"). A post set callback, as in ``LiveScoutNode``, updates the copy.


.. admonition:: Question 4
   :class: hint

   Which parameter callback do the ROS 2 docs give for reacting to a
   change that was accepted, such as copying the new value into an
   attribute?

   A. The pre set callback, ``add_pre_set_parameters_callback()``
   B. The set callback, ``add_on_set_parameters_callback()``
   C. The post set callback, ``add_post_set_parameters_callback()``
   D. A timer that calls ``ros2 param get``

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**

   *Explanation:* The docs: the post set callback is there "to react to changes from parameters that have successfully been accepted." The set callback is for refusing a value, and "It is important that “set parameter” callbacks have no side-effects." ("A Callback After Each Set").


.. admonition:: Question 5
   :class: hint

   You want to define ``RobotStatus.msg`` for nodes written in Python.
   Where does the file go?

   A. In the ``msg/`` folder of an ament_python package, next to the nodes
   B. In the ``msg/`` folder of an ament_cmake package, listed in
      ``rosidl_generate_interfaces`` of its ``CMakeLists.txt``
   C. Anywhere in the workspace: colcon finds every ``.msg`` file
   D. In the Python module that uses it, as a class

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**

   *Explanation:* The tutorial: "it is, and can only be, an ament_cmake package, but this doesn’t restrict in which type of packages you can use your messages and services." The file is listed in ``rosidl_generate_interfaces`` ("An Interface Package", "CMakeLists.txt and package.xml").


.. admonition:: Question 6
   :class: hint

   ``battery_pct`` is a ``uint8``. Without ``ROS_PYTHON_CHECK_FIELDS``, a
   node sets ``msg.battery_pct = 300`` and publishes the message. What
   does a subscriber receive?

   A. 300
   B. 255
   C. 44
   D. Nothing: ``publish()`` raises an exception

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**

   *Explanation:* Without the check, the value is not tested; 300 does not fit in 8 bits and arrives as 300 − 256 = 44. ``ros2 topic echo`` printed 44 ("A Field Is Checked Only If You Ask"). With ``ROS_PYTHON_CHECK_FIELDS=1``, the assignment itself raises an ``AssertionError``.


.. admonition:: Question 7
   :class: hint

   ``task_server`` starts at 30%. It gets a task of 10 minutes, then a
   task of 1 minute. What are the two replies?

   A. Accepted, 10%; then refused, 10% (below the limit)
   B. Accepted, 10%; then accepted, 8%
   C. Refused, 30%; then accepted, 28%
   D. Accepted, 10%; then refused, 8%

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**

   *Explanation:* 10 minutes use 2 × 10 = 20%: 30 − 20 = 10%. At 10% the robot is not below 10%, so rule 3 does not refuse, and 1 minute uses 2%: 10 − 2 = 8%. Checked by running the server (``code/quiz_checks.sh`` of the slides): ``accepted=True, battery_pct=10`` then ``accepted=True, battery_pct=8``.


.. admonition:: Question 8
   :class: hint

   A node calls ``self._client.call(request)`` in its timer callback.
   The timer and the client are both in the node's default group, and the
   node spins a ``SingleThreadedExecutor``. The server is running. What
   happens?

   A. ``call()`` returns the reply after the server answers
   B. ``call()`` raises an exception after a timeout
   C. ``call()`` never returns, and nothing reports it
   D. The executor starts a second thread for the reply

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**

   *Explanation:* The one thread waits in ``call()``, and the reply can only be taken in on that thread. ``sync_requester`` logged ``Sending pick widget`` and nothing more, though the server logged ``done`` ("call() Inside a Callback Never Returns"). The how-to: "When deadlock occurs, you will not receive any indication that the service is blocked."


.. admonition:: Question 9
   :class: hint

   An action server is made with an execute callback only: no
   ``cancel_callback``. A client asks to cancel a running goal. What
   happens?

   A. The goal is canceled at the next check of ``is_cancel_requested``
   B. The cancel is refused, and the goal runs to the end
   C. The goal stops at once, in the middle of a step
   D. The server stops with an exception

.. dropdown:: Answer
   :class-container: sd-border-success

   **B**

   *Explanation:* rclpy's default cancel callback returns ``CancelResponse.REJECT`` (``rclpy/action/server.py``). Checked with the lecture's ``drive_server`` made without a cancel callback (``code/quiz_checks.sh``): the client logged ``Canceling the goal``, then ``Result: SUCCEEDED, driven 5.0 m``.


.. admonition:: Question 10
   :class: hint

   A publisher offers best effort. A subscription on the same topic asks
   for reliable. Both are volatile. What happens?

   A. They connect, and every message arrives
   B. They connect, and some messages may be lost
   C. They do not connect, and both nodes log a warning that names
      ``RELIABILITY``
   D. They do not connect, and nothing is logged

.. dropdown:: Answer
   :class-container: sd-border-success

   **C**

   *Explanation:* The subscription asks for more than the publisher offers. Measured: 0 of 5 messages, and both warned, "Last incompatible policy: RELIABILITY" ("A Best-Effort Publisher, a Reliable Monitor", "Seven Pairs, Measured").


True/False
==========

.. admonition:: Statement 1
   :class: hint

   An interface package can be an ament_python package if all the nodes
   that use it are written in Python.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* The tutorial: "it is, and can only be, an ament_cmake package". The nodes that use it can be in any package ("An Interface Package").


.. admonition:: Statement 2
   :class: hint

   ``ros2 param set`` changes a node's parameter by calling a service of
   that node.

.. dropdown:: Answer
   :class-container: sd-border-success

   **True**

   *Explanation:* Every node has the parameter services, such as ``/scout/set_parameters``, and the docs: "ros2 param uses the parameter service API as described above to perform the various operations." ("ros2 param Calls the Node's Services").


.. admonition:: Statement 3
   :class: hint

   ``ros2 topic info -v`` shows the history depth of each publisher and
   subscription.

.. dropdown:: Answer
   :class-container: sd-border-success

   **False**

   *Explanation:* It printed ``History (Depth): UNKNOWN`` for every endpoint measured in the lecture ("The Default Profile, and One for Sensors"). It does show reliability and durability.


.. admonition:: Statement 4
   :class: hint

   A reliable publisher and a best-effort subscription on the same topic
   connect.

.. dropdown:: Answer
   :class-container: sd-border-success

   **True**

   *Explanation:* The subscription asks for less than the publisher offers. Measured: 5 of 5 messages arrived ("Seven Pairs, Measured").
