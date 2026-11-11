====================================================
Lecture
====================================================

This lecture uses two repositories:

- The ROS repository,
  `enpm605-spring-2027-ros <https://github.com/rubixcubic/enpm605-spring-2027-ros>`_,
  holds the ROS 2 package of this lecture, ``lecture8/competition_nodes``.
  Clone it into the ``src`` folder of a workspace (see
  `A Workspace Is a Folder with Your Packages in src`_).
- The course repository,
  `enpm605-spring-2027-python <https://github.com/rubixcubic/enpm605-spring-2027-python>`_,
  holds four plain Python programs in ``lecture8/``. Run them from its
  root folder, for example ``uv run lecture8/message_demo.py``.
  ``process_demo.py`` needs nothing else; ``message_demo.py``,
  ``spin_probe.py``, and ``errors/bad_publish.py`` need ROS 2 sourced
  first.

Every output on this page was printed by running the code with ROS 2
Jazzy on Ubuntu 24.04, with ``ROS_DOMAIN_ID=77`` and
``ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST``. Log lines were
recorded without their time stamp (see `Reading a Log Line`_), and a
node that runs until Ctrl+C shows its first lines only. In paths,
``...`` marks a path shortened to fit. The values in the examples
(battery levels, ranges, the domain ID 77) are chosen for the examples.


What Is ROS?
====================================================

ROS Is a Set of Libraries and Tools for Robot Software
------------------------------------------------------

The ROS 2 Jazzy documentation's home page: "The Robot Operating System
(ROS) is a set of software libraries and tools for building robot
applications. From drivers and state-of-the-art algorithms to powerful
developer tools, ROS has the open source tools you need for your next
robotics project."

- **Libraries** you import in your code. This course uses ``rclpy``, the
  library for Python.
- **Tools** you run in a terminal: the ``ros2`` command and the
  ``colcon`` build tool, both in this lecture.
- **Packages** that other people wrote: sensor drivers, navigation,
  simulation (Lectures 11 and 12).

The same page: "Since ROS was started in 2007, a lot has changed in the
robotics and ROS community. The goal of the ROS 2 project is to adapt to
these changes". This course uses ROS 2 only.

A Distribution Is a Versioned Set of Packages
---------------------------------------------

The ROS 2 documentation: "A ROS distribution is a versioned set of ROS
packages." Each one has a name, a release date, and an end-of-life (EOL)
date, after which it gets no more fixes. From the
`Distributions <https://docs.ros.org/en/jazzy/Releases.html>`_ page,
read on October 14, 2026:

.. list-table::
   :header-rows: 1
   :widths: 40 30 30
   :class: compact-table

   * - Distribution
     - Released
     - End of life
   * - Humble Hawksbill
     - May 23, 2022
     - May 2027
   * - Jazzy Jalisco
     - May 23, 2024
     - May 2029
   * - Kilted Kaiju
     - May 23, 2025
     - December 2026
   * - Lyrical Luth
     - May 22, 2026
     - May 2031

This course uses **Jazzy Jalisco**, on Ubuntu Noble (24.04).
`REP 2000 <https://www.ros.org/reps/rep-2000.html>`_ lists Ubuntu Noble
as a Tier 1 platform for Jazzy: "Tier 1 platforms are subjected to our
unit test suite and other testing tools on a frequent basis". Jazzy's
fixes run past the end of the semester (May 2027).

Sourcing Sets Up a Terminal for ROS 2
-------------------------------------

In every new terminal:

.. code-block:: bash

   source /opt/ros/jazzy/setup.bash
   printenv ROS_DISTRO ROS_VERSION ROS_PYTHON_VERSION

.. output: printenv.txt all

.. code-block:: text

   jazzy
   2
   3

- ``source`` runs the file in the current shell, so the variables it
  sets stay set. The ROS 2 docs: "Without sourcing the setup files, you
  won't be able to access ROS 2 commands, or find or use ROS 2
  packages."
- To source it in every new terminal, add the line to ``~/.bashrc``
  once: ``echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc``. A zsh
  user sources ``setup.zsh`` instead.


ROS 2 Architecture
====================================================

A Process Is a Running Program with Its Own Memory
--------------------------------------------------

.. code: lecture8/process_demo.py all

.. code-block:: python
   :caption: lecture8/process_demo.py

   """Two processes do not share memory: a change in one stays in it."""

   import multiprocessing
   import os

   readings = [3.2]  # Range readings in meters, chosen for this example


   def child():
       readings.append(2.8)  # Changes the child's copy of the list
       print(f"child sees  {readings}")


   if __name__ == "__main__":
       process = multiprocessing.Process(target=child)
       process.start()  # A new process, with its own copy of memory
       process.join()  # Wait for it to finish
       print(f"parent sees {readings}")
       print("different process IDs:", process.pid != os.getpid())

.. output: process_demo.txt all

.. code-block:: text

   child sees  [3.2, 2.8]
   parent sees [3.2]
   different process IDs: True

- A program on disk is a file. When the operating system runs it, it
  becomes a **process**: it gets its own memory and its own process ID
  (PID).
- The child's ``append()`` changed only the child's list. Two processes
  share data only by sending it to each other. The output is the same
  with or without ROS 2 sourced (checked when it was recorded).

A Monolithic Design: One Program Does Everything
------------------------------------------------

.. figure:: /_static/images/l8/monolithic.svg
   :alt: Tag 1: a large gray box, one process, labeled robot_program.py: one program, one process. Inside it, four white boxes in a row, tag 2 next to the first: lidar driver, battery check, task planner and motor control, with an arrow from each box to the next.
   :width: 90%

   1: one process, the whole robot program, started and stopped as one.
   2: one part of it, a function or a class. The gray arrows are calls
   from one part to the next.

- An exception that no part handles stops the whole process (Lecture 6),
  so a bug in the battery check also stops motor control.
- A change to one part means stopping and restarting all four.

A Distributed Design: One Node per Job
--------------------------------------

.. figure:: /_static/images/l8/distributed.svg
   :alt: A graph of four nodes, drawn as ellipses, and three topics, drawn as gray rectangles. Tag 1 marks the node lidar, tag 2 the topic /scan. Arrows: lidar to /scan, /scan to planner, planner to /cmd_vel, /cmd_vel to motors, battery to /battery, and /battery to planner.
   :width: 90%

   1: a **node**, a program that does one job; here each node runs in
   its own process. 2: a **topic**, a named channel. An arrow into a
   topic means "sends messages on it"; an arrow out means "receives
   them".

The ROS 2 docs: "each node should do one logical thing." This graph is
a sketch; its four nodes are not course code.

What Splitting into Nodes Gives, and What It Costs
--------------------------------------------------

.. list-table::
   :header-rows: 1
   :widths: 26 34 40
   :class: compact-table

   * -
     - Monolithic
     - Distributed (nodes)
   * - An unhandled error
     - Stops the whole program
     - Stops one node's process
   * - A change to one part
     - Restart everything
     - Restart one node
   * - Languages
     - One
     - Each node its own (Python or C++)
   * - Computers
     - One
     - Nodes can run on several
   * - Data between parts
     - A function call
     - A message, copied between processes
   * - Starting the system
     - One command
     - One command per node (Lecture 9: one launch file)

The ROS 2 docs: "Nodes can communicate with other nodes within the same
process, in a different process, or on a different machine." This
lecture runs one node per process.


Core Components and DDS
====================================================

Nodes Talk Through Topics, Services, and Actions
------------------------------------------------

The ROS 2 docs: "Nodes can publish to named topics to deliver data to
other nodes, or subscribe to named topics to get data from other nodes."
A node that sends is a **publisher**; a node that receives is a
**subscriber**.

.. list-table::
   :header-rows: 1
   :widths: 14 44 24 18
   :class: compact-table

   * - Kind
     - What it is for (ROS 2 docs, Topics and Nodes)
     - Competition example
     - Taught
   * - Topic
     - "continuous data streams, like sensor data, robot state"
     - Battery level
     - Now
   * - Service
     - "to have another node perform a computation on their behalf"
     - Assign a task
     - Lecture 10
   * - Action
     - "For long-running computations"
     - Drive to a goal
     - Lecture 10

The data on a topic travels as **messages**. The docs on interfaces:
"ROS applications typically communicate through interfaces of one of
three types: topics, services, or actions."

A Message Type Lists Typed Fields
---------------------------------

.. code-block:: bash

   ros2 interface show sensor_msgs/msg/BatteryState

The fields of the output (lines 36 to 58 of 58; lines 1 to 34 define
constants):

.. output: interface_battery.txt 36-58

.. code-block:: text

   std_msgs/Header  header
   	builtin_interfaces/Time stamp
   		int32 sec
   		uint32 nanosec
   	string frame_id
   float32 voltage          # Voltage in Volts (Mandatory)
   float32 temperature      # Temperature in Degrees Celsius (If unmeasured NaN)
   float32 current          # Negative when discharging (A)  (If unmeasured NaN)
   float32 charge           # Current charge in Ah  (If unmeasured NaN)
   float32 capacity         # Capacity in Ah (last full capacity)  (If unmeasured NaN)
   float32 design_capacity  # Capacity in Ah (design capacity)  (If unmeasured NaN)
   float32 percentage       # Charge percentage on 0 to 1 range  (If unmeasured NaN)
   uint8   power_supply_status     # The charging status as reported. Values defined above
   uint8   power_supply_health     # The battery health metric. Values defined above
   uint8   power_supply_technology # The battery chemistry. Values defined above
   bool    present          # True if the battery is present

   float32[] cell_voltage   # An array of individual cell voltages for each cell in the pack
                            # If individual voltages unknown but number of cells known set each to NaN
   float32[] cell_temperature # An array of individual cell temperatures for each cell in the pack
                              # If individual temperatures unknown but number of cells known set each to NaN
   string location          # The location into which the battery is inserted. (slot number or plug)
   string serial_number     # The best approximation of the battery serial number

- A message type is written in a ``.msg`` file: one field per line, its
  type and then its name; ``#`` starts a comment. The docs: ".msg files
  are simple text files that describe the fields of a ROS message."
- ``sensor_msgs/msg/BatteryState``: the package, then ``msg``, then the
  type. A field can be another message: ``header`` is a
  ``std_msgs/Header``, shown indented.
- The field this lecture uses is ``float32 percentage``, "Charge
  percentage on 0 to 1 range".

The Layers Under a Python Node
------------------------------

.. figure:: /_static/images/l8/layers.svg
   :alt: Five boxes stacked from top to bottom, joined by arrows going down. Tag 1: battery_publisher.py, your node. Tag 2: rclpy, Python client library. Tag 3: rcl, C library: rclpy and rclcpp. Tag 4: rmw_fastrtps_cpp, ROS middleware interface. Tag 5: Fast DDS, sends the messages on the network.
   :width: 55%

   1: your code, which imports ``rclpy``. 2: ``rclpy``, "the Python
   counterpart to the C++ client library", ``rclcpp``. 3: ``rcl``: the
   docs say ``rclpy`` "builds on top of the rcl C API". 4: the ROS
   middleware interface (rmw), which hides which DDS is used. 5: the DDS
   that sends the messages on the network.

A Python node and a C++ node can talk to each other. The docs: "all
client libraries implement code generators that provide users with the
capability to interact with ROS 2 interface files in the respective
language": both send the messages that the same ``.msg`` files describe.

DDS Finds Nodes and Carries Their Messages
------------------------------------------

- The Data Distribution Service (DDS) is a standard for publish/subscribe
  communication. The ROS 2 docs: "DDS is an industry standard which is
  implemented by a range of vendors".
- What it gives ROS 2: "distributed discovery (not centralized like in
  ROS 1) and control over different 'Quality of Service' options for the
  transportation." Quality of Service (QoS): settings for how messages
  are delivered, such as how many wait in a queue.
- Jazzy's default: "The default is Fast DDS, which works without any
  additional installation steps". ``ros2 doctor`` names the rmw on this
  PC:

.. code-block:: bash

   ros2 doctor --report | grep -A1 "RMW MIDDLEWARE"

.. output: doctor_rmw.txt all

.. code-block:: text

      RMW MIDDLEWARE
   middleware name    : rmw_fastrtps_cpp

The docs' table of middleware lists ``rmw_fastrtps_cpp`` as the rmw of
eProsima Fast DDS, "Default RMW." Quality of Service comes back in
`Step 1 of a Publisher: create_publisher() and a Timer`_.

Discovery: Nodes on the Same Domain Find Each Other
---------------------------------------------------

- The docs on discovery: "When a node is started, it advertises its
  presence to other nodes on the network with the same ROS domain (set
  with the ROS_DOMAIN_ID environment variable)."
- The docs on domains: "ROS 2 nodes on the same domain can freely
  discover and send messages to each other, while ROS 2 nodes on
  different domains cannot. All ROS 2 nodes use domain ID 0 by default."
- So in a classroom on one network, every node of every student on
  domain 0 sees every other one.

The course's two lines in ``~/.bashrc`` (77 is the domain of this
lecture's examples):

.. code-block:: bash

   export ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST
   export ROS_DOMAIN_ID=77
   printenv ROS_AUTOMATIC_DISCOVERY_RANGE ROS_DOMAIN_ID

.. output: printenv_domain.txt all

.. code-block:: text

   LOCALHOST
   77

- The docs: "LOCALHOST means a node will only try to discover other
  nodes on the same machine." Your nodes no longer see your classmates'
  computers. The ROS 2 setup file sets this variable to ``SUBNET`` only
  when it is not set yet, so the ``export`` line may come before or after
  the ``source`` line (checked).
- The domain keeps you apart from other ROS 2 programs on the same
  computer. The instructor gives each student a domain number from 1 to
  101 (the docs: "simply choose a domain ID between 0 and 101,
  inclusive").


The Publish/Subscribe Model
====================================================

Publishers and Subscribers Meet at a Topic Name
-----------------------------------------------

The ROS 2 docs, Topics: "A publish/subscribe system is one in which
there are producers of data (publishers) and consumers of data
(subscribers). The publishers and subscribers know how to contact each
other through the concept of a 'topic', which is a common name so that
the entities can find each other."

- A publisher does not call a subscriber, and does not know it exists.
  Both use the same topic name, and discovery connects them.
- The docs call this "anonymous": "when a subscriber gets a piece of
  data, it doesn't generally know or care which publisher originally
  sent it".
- So you can replace a node with another one that publishes on the same
  topic, and the subscribers do not change.

Four Rules of a Topic
---------------------

1. **Same name.** "Any publishers and subscribers that are on the same
   topic name can directly communicate with each other."
2. **Same message type.** A subscriber receives only from publishers of
   its own message type (measured in
   `Rule 2, Measured: Two Types on One Topic`_).
3. **Zero or more of each.** "There may be zero or more publishers and
   zero or more subscribers on any particular topic." A publisher sends
   even when no one listens; a subscriber waits when no one sends.
4. **Every subscriber gets each message.** "When data is published to
   the topic by any of the publishers, all subscribers in the system will
   receive the data."

The quotes are from the Topics page. The discovery page adds a
condition to rule 4: "Nodes will only establish connections with other
nodes if they have compatible Quality of Service settings." Every node in
this lecture uses the default settings.

Rule 2, Measured: Two Types on One Topic
----------------------------------------

Terminal 1, started first: an ``Int32`` subscriber. It printed nothing.

.. code-block:: bash

   ros2 topic echo /mix std_msgs/msg/Int32

Terminal 2: a ``String`` publisher, 2 messages per second:

.. code-block:: bash

   ros2 topic pub -r 2 /mix std_msgs/msg/String '{data: hello}'

.. output: mix_pub.txt all

.. code-block:: text

   publisher: beginning loop
   publishing #1: std_msgs.msg.String(data='hello')

Terminal 3, while both run:

.. code-block:: bash

   ros2 topic info /mix

.. output: mix_topic_info.txt all

.. code-block:: text

   Type: ['std_msgs/msg/Int32', 'std_msgs/msg/String']
   Publisher count: 1
   Subscription count: 1

- ``ros2 topic pub`` publishes from the command line. The topic has two
  types, one publisher and one subscriber, and no message reached the
  subscriber in 6 s of publishing.
- "Strongly-typed", in the Topics page, is about fields: "The types of
  each field in a ROS message are typed, and that type is enforced at
  various levels."


One Topic, Many Subscribers or Many Publishers
----------------------------------------------

.. figure:: /_static/images/l8/patterns.svg
   :alt: Two graphs, nodes drawn as ellipses and topics as gray rectangles. Tag 1: the node battery_publisher has an arrow to the topic /battery, which has arrows to two nodes, battery_monitor and low_battery_alarm. Tag 2: three nodes, talker, listener and battery_monitor, each have an arrow to the topic /rosout.
   :width: 85%

   1: one publisher, two subscribers: both receive every battery message
   (rule 4). These nodes are written later in this lecture and in
   Exercise 3. 2: many publishers, one topic: by default, every node
   sends its log messages to ``/rosout``.

The ROS 2 docs on logging: "By default, log messages in ROS 2 nodes will
go out to the console (on stderr), to log files on disk, and to the
/rosout topic on the ROS 2 network."

A First Run: the Talker and the Listener
----------------------------------------

Terminal 1, started first:

.. code-block:: bash

   ros2 run demo_nodes_py listener

.. output: listener.txt all

.. code-block:: text

   [INFO] [listener]: I heard: [Hello World: 0]
   [INFO] [listener]: I heard: [Hello World: 1]
   [INFO] [listener]: I heard: [Hello World: 2]

Terminal 2:

.. code-block:: bash

   ros2 run demo_nodes_py talker

.. output: talker.txt all

.. code-block:: text

   [INFO] [talker]: Publishing: "Hello World: 0"
   [INFO] [talker]: Publishing: "Hello World: 1"
   [INFO] [talker]: Publishing: "Hello World: 2"

- ``ros2 run <package> <executable>`` starts one node. ``demo_nodes_py``
  comes with ROS 2: ``talker`` publishes a ``std_msgs/msg/String`` on
  ``/chatter`` every 1.0 s; ``listener`` subscribes to it.
- Both terminals sourced ROS 2 and use the same domain. Each node runs
  until Ctrl+C; the first three lines of each are shown.

Reading a Log Line
------------------

The talker's first line, as ROS 2 prints it by default (one run):

.. output: talker_default_format.txt all

.. code-block:: text

   [INFO] [1791963070.982392390] [talker]: Publishing: "Hello World: 0"

- ``[INFO]``: the severity. The ones in this lecture are ``INFO`` and
  ``WARN``.
- The number: "The time in seconds since the epoch" (January 1, 1970).
  It changes on every run; this one is from one run on the instructor's
  PC.
- ``[talker]``: the name of the node's logger, which is the node's name.
  Then the message.

The docs: "If no format is given, a default of
``[{severity}] [{time}] [{name}]: {message}`` is used." The outputs on
this page were recorded with a format that leaves out the time, so they
are the same on every run:

.. code-block:: bash

   export RCUTILS_CONSOLE_OUTPUT_FORMAT="[{severity}] [{name}]: {message}"

Two Domains Do Not See Each Other
---------------------------------

.. code-block:: bash

   # Terminal 1
   export ROS_DOMAIN_ID=77
   ros2 run demo_nodes_py talker

.. code-block:: bash

   # Terminal 2: no output in 4 s
   export ROS_DOMAIN_ID=78
   ros2 run demo_nodes_py listener

- The listener on domain 78 printed nothing in 4 s, then was stopped
  with Ctrl+C. The talker was publishing on domain 77 the whole time. On
  domain 77, the same listener prints the lines of
  `A First Run: the Talker and the Listener`_.
- Both terminals had ``ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST``, so
  only the domain kept them apart. The ROS 2 docs on that variable: it
  "is helpful in certain settings, such as classrooms, where multiple
  robots may publish to the same topic causing strange behaviors."


Introspection Tools
====================================================

The ros2 Command Inspects a Running System
------------------------------------------

.. list-table::
   :header-rows: 1
   :widths: 45 55
   :class: compact-table

   * - Command
     - What it prints
   * - ``ros2 node list``
     - The names of the running nodes
   * - ``ros2 node info <node>``
     - A node's publishers, subscribers, and services
   * - ``ros2 topic list -t``
     - The topics, each with its message type
   * - ``ros2 topic info <topic>``
     - A topic's type, and how many publishers and subscribers it has
   * - ``ros2 topic echo <topic>``
     - Each message on the topic, as it arrives
   * - ``ros2 topic hz <topic>``
     - How many messages per second arrive
   * - ``ros2 interface show <type>``
     - The fields of a message type
   * - ``ros2 run rqt_graph rqt_graph``
     - A window with the graph of nodes and topics

Each command sees only the nodes of its own domain. The commands below
ran while the talker and the listener ran, all on domain 77.

ros2 node list and ros2 topic list
----------------------------------

.. code-block:: bash

   ros2 node list

.. output: node_list.txt all

.. code-block:: text

   /listener
   /talker

.. code-block:: bash

   ros2 topic list -t

.. output: topic_list_t.txt all

.. code-block:: text

   /chatter [std_msgs/msg/String]
   /parameter_events [rcl_interfaces/msg/ParameterEvent]
   /rosout [rcl_interfaces/msg/Log]

- A name that starts with ``/`` is a full name. ``-t`` adds each topic's
  message type.
- ``/chatter`` is the talker's topic. ``/rosout`` carries log messages;
  ``/parameter_events`` belongs to parameters, in Lecture 10.

ros2 node info and ros2 topic info
----------------------------------

.. code-block:: bash

   ros2 node info /talker

.. output: node_info_talker.txt 1-20

.. code-block:: text

   /talker
     Subscribers:

     Publishers:
       /chatter: std_msgs/msg/String
       /parameter_events: rcl_interfaces/msg/ParameterEvent
       /rosout: rcl_interfaces/msg/Log
     Service Servers:
       /talker/describe_parameters: rcl_interfaces/srv/DescribeParameters
       /talker/get_parameter_types: rcl_interfaces/srv/GetParameterTypes
       /talker/get_parameters: rcl_interfaces/srv/GetParameters
       /talker/get_type_description: type_description_interfaces/srv/GetTypeDescription
       /talker/list_parameters: rcl_interfaces/srv/ListParameters
       /talker/set_parameters: rcl_interfaces/srv/SetParameters
       /talker/set_parameters_atomically: rcl_interfaces/srv/SetParametersAtomically
     Service Clients:

     Action Servers:

     Action Clients:



.. code-block:: bash

   ros2 topic info /chatter

.. output: topic_info.txt all

.. code-block:: text

   Type: std_msgs/msg/String
   Publisher count: 1
   Subscription count: 1

- The talker publishes ``/chatter``, its log on ``/rosout``, and
  ``/parameter_events``. Its 7 service servers are for parameters
  (Lecture 10) and for ``get_type_description``.
- ``/chatter`` has one publisher, the talker, and one subscriber, the
  listener: same name, same type (rules 1 and 2 of
  `Four Rules of a Topic`_).

ros2 topic echo and ros2 topic hz
---------------------------------

Started before the talker, with the type, so that it gets the first
message:

.. code-block:: bash

   ros2 topic echo --once /chatter std_msgs/msg/String

.. output: topic_echo.txt all

.. code-block:: text

   data: 'Hello World: 0'
   ---

While the talker runs (one run; stopped with Ctrl+C):

.. code-block:: bash

   ros2 topic hz /chatter

.. output: topic_hz.txt all

.. code-block:: text

   average rate: 1.000
   	min: 1.000s max: 1.000s std dev: 0.00018s window: 2

- ``echo`` prints each field as ``name: value``, and ``---`` after each
  message; ``--once`` stops after one. ``echo`` is itself a subscriber.
- 1.000 messages per second: the talker's timer period is 1.0 s. Every
  number on the second line (min, max, std dev, window) changes from run
  to run.

The Talker's Message Type Is Deprecated
---------------------------------------

.. code-block:: bash

   ros2 interface show std_msgs/msg/String

.. output: interface_string.txt all

.. code-block:: text

   # This was originally provided as an example message.
   # It is deprecated as of Foxy
   # It is recommended to create your own semantically meaningful message.
   # However if you would like to continue using this please use the equivalent in example_msgs.

   string data

- Foxy is the ROS 2 distribution of 2020, before Jazzy. A ``String``
  says nothing about what the text means; a semantically meaningful
  message names its fields and units.
- This lecture's battery nodes use ``sensor_msgs/msg/BatteryState``
  (`A Message Type Lists Typed Fields`_), whose ``percentage`` field
  says "0 to 1 range".


Workspaces and Packages
====================================================

A Workspace Is a Folder with Your Packages in src
-------------------------------------------------

The ROS 2 docs: "'Workspace' is a ROS term for the location on your
system where you're developing with ROS 2." A **package** is a folder of
code that is built and installed as one unit. The course's packages are
in a Git repository. Clone it into the ``src`` folder of a new
workspace, ``~/ros_ws``:

.. code-block:: bash

   mkdir -p ~/ros_ws/src
   cd ~/ros_ws/src
   git clone https://github.com/rubixcubic/enpm605-spring-2027-ros.git

Clone it once. Later, ``git pull`` in ``enpm605-spring-2027-ros`` gets
each new lecture's packages.

ros2 pkg create Makes a Python Package
--------------------------------------

How the course's package was made. Do not run this command: see
`One Name per Package in a Workspace`_.

.. code-block:: bash

   ros2 pkg create competition_nodes --build-type ament_python \
       --dependencies rclpy sensor_msgs --license Apache-2.0 \
       --description "Lecture 8 nodes: robotics competition" \
       --maintainer-name "Zeid Kootbally" --maintainer-email zeidk@umd.edu

.. output: pkg_create.txt all

.. code-block:: text

   going to create a new package
   package name: competition_nodes
   destination directory: .../src
   package format: 3
   version: 0.0.0
   description: Lecture 8 nodes: robotics competition
   maintainer: ['Zeid Kootbally <zeidk@umd.edu>']
   licenses: ['Apache-2.0']
   build type: ament_python
   dependencies: ['rclpy', 'sensor_msgs']
   creating folder ./competition_nodes
   creating ./competition_nodes/package.xml
   creating source folder
   creating folder ./competition_nodes/competition_nodes
   creating ./competition_nodes/setup.py
   creating ./competition_nodes/setup.cfg
   creating folder ./competition_nodes/resource
   creating ./competition_nodes/resource/competition_nodes
   creating ./competition_nodes/competition_nodes/__init__.py
   creating folder ./competition_nodes/test
   creating ./competition_nodes/test/test_copyright.py
   creating ./competition_nodes/test/test_flake8.py
   creating ./competition_nodes/test/test_pep257.py

``ament_python``: a package of Python code. ``--dependencies``: the
packages it imports. Path shortened.

One Name per Package in a Workspace
-----------------------------------

A new ``competition_nodes`` next to the clone of the course repository:

.. code-block:: bash

   cd ~/ros_ws/src
   ros2 pkg create competition_nodes --build-type ament_python
   cd ~/ros_ws
   colcon build --symlink-install

.. output: dup_package.txt all

.. code-block:: text

   ERROR:colcon:colcon build: Duplicate package names not supported:
   - competition_nodes:
     - src/competition_nodes
     - src/enpm605-spring-2027-ros/lecture8/competition_nodes

- colcon builds nothing: two packages in one workspace may not share a
  name. The course's ``competition_nodes`` is already in the clone.
- Your own packages get their own names, such as ``my_competition`` in
  Exercise 2. colcon's time prefix, such as ``[0.2s]``, is left out.


The Files of a Python Package
-----------------------------

.. code-block:: bash

   find competition_nodes | LC_ALL=C sort

The package in the ROS repository, after its nodes were added (the
``__pycache__`` folders that running a node writes are left out):

.. output: pkg_tree_full.txt all

.. code-block:: text

   competition_nodes
   competition_nodes/LICENSE
   competition_nodes/competition_nodes
   competition_nodes/competition_nodes/__init__.py
   competition_nodes/competition_nodes/battery_monitor.py
   competition_nodes/competition_nodes/battery_publisher.py
   competition_nodes/competition_nodes/heartbeat_node.py
   competition_nodes/competition_nodes/minimal_node.py
   competition_nodes/competition_nodes/scout_node.py
   competition_nodes/package.xml
   competition_nodes/resource
   competition_nodes/resource/competition_nodes
   competition_nodes/setup.cfg
   competition_nodes/setup.py
   competition_nodes/test
   competition_nodes/test/test_copyright.py
   competition_nodes/test/test_flake8.py
   competition_nodes/test/test_pep257.py

- The inner ``competition_nodes/``: the Python package (Lecture 6), with
  one module per node. The five node files were added after
  ``ros2 pkg create``.
- ``package.xml``: the package's name, version, and dependencies.
- ``setup.py``, ``setup.cfg``: how to install it.
- ``resource/competition_nodes``: an empty file that marks the package
  as installed.
- ``test/``: style checks that ``colcon test`` runs.

package.xml: Name, Version, and Dependencies
--------------------------------------------

.. code: ros/lecture8/competition_nodes/package.xml all

.. code-block:: xml
   :caption: lecture8/competition_nodes/package.xml

   <?xml version="1.0"?>
   <?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
   <package format="3">
     <name>competition_nodes</name>
     <version>0.0.0</version>
     <description>Lecture 8 nodes: robotics competition</description>
     <maintainer email="zeidk@umd.edu">Zeid Kootbally</maintainer>
     <license>Apache-2.0</license>

     <depend>rclpy</depend>
     <depend>sensor_msgs</depend>

     <test_depend>ament_copyright</test_depend>
     <test_depend>ament_flake8</test_depend>
     <test_depend>ament_pep257</test_depend>
     <test_depend>python3-pytest</test_depend>

     <export>
       <build_type>ament_python</build_type>
     </export>
   </package>

``<depend>``: a package this one needs to build and to run.
``<test_depend>``: needed only for the tests. Add a ``<depend>`` line
when a node imports from another package.

setup.py: Entry Points Name the Executables
-------------------------------------------

.. code: ros/lecture8/competition_nodes/setup.py all

.. code-block:: python
   :caption: lecture8/competition_nodes/setup.py

   from setuptools import find_packages, setup

   package_name = 'competition_nodes'

   setup(
       name=package_name,
       version='0.0.0',
       packages=find_packages(exclude=['test']),
       data_files=[
           ('share/ament_index/resource_index/packages',
               ['resource/' + package_name]),
           ('share/' + package_name, ['package.xml']),
       ],
       install_requires=['setuptools'],
       zip_safe=True,
       maintainer='Zeid Kootbally',
       maintainer_email='zeidk@umd.edu',
       description='Lecture 8 nodes: robotics competition',
       license='Apache-2.0',
       extras_require={
           'test': [
               'pytest',
           ],
       },
       entry_points={
           'console_scripts': [
               'minimal_node = competition_nodes.minimal_node:main',
               'scout_node = competition_nodes.scout_node:main',
               'heartbeat_node = competition_nodes.heartbeat_node:main',
               'battery_publisher = competition_nodes.battery_publisher:main',
               'battery_monitor = competition_nodes.battery_monitor:main',
           ],
       },
   )

- ``ros2 pkg create`` wrote ``setup.py`` with an empty
  ``console_scripts`` list. The five lines in it were added by hand; the
  difference from the generated file:

.. output: setup_py_diff.txt all

.. code-block:: text

   26a27,31
   >             'minimal_node = competition_nodes.minimal_node:main',
   >             'scout_node = competition_nodes.scout_node:main',
   >             'heartbeat_node = competition_nodes.heartbeat_node:main',
   >             'battery_publisher = competition_nodes.battery_publisher:main',
   >             'battery_monitor = competition_nodes.battery_monitor:main',

- Each line is ``executable = package.module:function``.
  ``ros2 run competition_nodes battery_publisher`` runs ``main()`` of
  ``competition_nodes/battery_publisher.py``.
- ``setup.cfg`` sends the executables to ``lib/competition_nodes`` in
  the install folder, where ``ros2 run`` looks for them:

.. code: ros/lecture8/competition_nodes/setup.cfg all

.. code-block:: text
   :caption: lecture8/competition_nodes/setup.cfg

   [develop]
   script_dir=$base/lib/competition_nodes
   [install]
   install_scripts=$base/lib/competition_nodes

Build with colcon, Then Source the Workspace
--------------------------------------------

.. code-block:: bash

   cd ~/ros_ws
   colcon build --symlink-install

.. output: colcon_build.txt all

.. code-block:: text

   Starting >>> competition_nodes
   Finished <<< competition_nodes [-.--s]

   Summary: 1 package finished [-.--s]

.. code-block:: bash

   source install/setup.bash
   ros2 pkg executables competition_nodes

.. output: pkg_executables.txt all

.. code-block:: text

   competition_nodes battery_monitor
   competition_nodes battery_publisher
   competition_nodes heartbeat_node
   competition_nodes minimal_node
   competition_nodes scout_node

Build from the workspace folder: colcon writes ``build``, ``install``, and
``log`` into the folder it runs in. Run in ``src``, it put them there:

.. output: build_in_src_ls.txt all

.. code-block:: text

   build
   enpm605-spring-2027-ros
   install
   log

The build times are blanked: they change on every build.

The package's style tests pass:

.. code-block:: bash

   colcon test --packages-select competition_nodes
   colcon test-result --all

.. output: colcon_test.txt all

.. code-block:: text

   build/competition_nodes/pytest.xml: 3 tests, 0 errors, 0 failures, 1 skipped

   Summary: 3 tests, 0 errors, 0 failures, 1 skipped

The skipped test is ``test_copyright.py``, which ``ros2 pkg create``
writes with a ``skip`` mark until the files have a copyright header.

The Four Folders of a Workspace, After a Build
----------------------------------------------

.. code-block:: bash

   cd ~/ros_ws
   ls

.. output: ws_ls.txt all

.. code-block:: text

   build
   install
   log
   src

- ``src``: your packages, as source code. You edit only this folder.
- The colcon tutorial: "The build directory will be where intermediate
  files are stored." "The install directory is where each package will
  be installed to." "The log directory contains various logging
  information about each colcon invocation."
- ``build``, ``install``, and ``log`` are made again by the next
  ``colcon build``: never edit or commit them. The course repository is
  inside ``src``, so they are not in it.

Underlay and Overlay: Source ROS 2 First, Then Your Workspace
-------------------------------------------------------------

The ROS 2 docs: "In a typical ROS 2 setup, the core ROS 2 installation
is the underlay. A local
workspace sourced after that installation is an overlay".

A terminal that sourced ROS 2 but not the workspace:

.. code-block:: bash

   ros2 run competition_nodes minimal_node

.. output: run_unsourced.txt all

.. code-block:: text

   Package 'competition_nodes' not found

After ``source install/setup.bash`` (paths shortened):

.. code-block:: bash

   ros2 pkg prefix competition_nodes
   printenv AMENT_PREFIX_PATH

.. output: pkg_prefix.txt all

.. code-block:: text

   .../ros_ws/install/competition_nodes
   .../ros_ws/install/competition_nodes:/opt/ros/jazzy

``AMENT_PREFIX_PATH`` lists where ROS 2 looks for packages: the overlay
first, then the underlay ``/opt/ros/jazzy``. Source your workspace in
every new terminal, after ROS 2.

A Symlink Install, and the Python That Runs a Node
--------------------------------------------------

- The colcon tutorial: ``--symlink-install`` "allows the installed files
  to be changed by changing the files in the source space (e.g. Python
  files or other non-compiled resources) for faster iteration." Edit a
  node's ``.py`` file and run it again, with no new build (Exercise 2
  does this).
- A new entry point in ``setup.py`` needs ``colcon build`` again: the
  build writes the executable file.

The first line of the executable the build wrote:

.. code-block:: bash

   cd install/competition_nodes/lib/competition_nodes
   head -n 1 battery_publisher

.. output: shebang.txt all

.. code-block:: text

   #!/usr/bin/python3

``ros2 run`` runs that file with Ubuntu's Python, ``/usr/bin/python3``,
not the course's uv environment (``.venv``, Lecture 1). A node imports
Ubuntu's and ROS 2's packages; a package added with ``uv add`` is not
there.


Writing Nodes with rclpy
====================================================

A Minimal Node, Written as a Script
-----------------------------------

.. code: ros/lecture8/competition_nodes/competition_nodes/minimal_node.py all

.. code-block:: python
   :caption: competition_nodes/minimal_node.py

   """A node written as a script, without a class (Lecture 8)."""

   import rclpy


   def main(args=None):
       rclpy.init(args=args)
       node = rclpy.create_node('minimal_node')  # The node's name
       node.get_logger().info('Hello from ROS 2')
       node.destroy_node()
       rclpy.shutdown()

.. code-block:: bash

   ros2 run competition_nodes minimal_node

.. output: minimal_node.txt all

.. code-block:: text

   [INFO] [minimal_node]: Hello from ROS 2

The ROS 2 demos use single quotes, and the package's style test
(``test_flake8.py``) asks for them: ``ament_flake8`` reports
``Q000 Double quotes found but single quotes preferred`` for a file in
double quotes. So the course's ROS code uses single quotes; the course's
plain Python keeps Ruff's double quotes.

What Each Call in minimal_node.py Does
--------------------------------------

1. ``rclpy.init(args=args)`` starts ROS 2 in this process. It comes
   before any node is made.
2. ``rclpy.create_node('minimal_node')`` makes a node named
   ``minimal_node``. The name is what ``ros2 node list`` and the log line
   show.
3. ``node.get_logger().info(...)`` logs a message with severity ``INFO``:
   to the terminal and to ``/rosout``
   (`One Topic, Many Subscribers or Many Publishers`_).
4. ``node.destroy_node()`` removes the node from the ROS 2 graph.
5. ``rclpy.shutdown()`` stops ROS 2 in this process.

- ``main(args=None)``: ``ros2 run`` calls ``main()`` with no argument,
  through the entry point of `setup.py: Entry Points Name the
  Executables`_. With ``args=None``, ``rclpy.init()`` reads the command
  line itself (``rclpy/context.py`` passes ``sys.argv`` when ``args`` is
  ``None``); Lecture 9 adds ROS arguments to it.
- Use the logger, not ``print()``, in a node: a log line has a severity
  and a node name, and it also goes to ``/rosout``.

A Node Is an Object of a Subclass of Node
-----------------------------------------

.. code: ros/lecture8/competition_nodes/competition_nodes/scout_node.py all

.. code-block:: python
   :caption: competition_nodes/scout_node.py

   """Lecture 7's ScoutNode, run as a node of a package (Lecture 8)."""

   import rclpy
   from rclpy.node import Node


   class ScoutNode(Node):
       """A node for the competition's scout robot."""

       def __init__(self):
           super().__init__('scout')  # Node.__init__ takes the node's name
           self.max_speed_mps = 2.0  # Chosen for this example (Lecture 7)
           self.get_logger().info(f'Top speed {self.max_speed_mps} m/s')


   def main(args=None):
       rclpy.init(args=args)
       node = ScoutNode()
       node.destroy_node()
       rclpy.shutdown()

.. code-block:: bash

   ros2 run competition_nodes scout_node; ros2 node list

.. output: scout_node.txt all

.. code-block:: text

   [INFO] [scout]: Top speed 2.0 m/s

This is Lecture 7's ``ScoutNode``, now in a package. The class keeps the
node's data as attributes (``self.max_speed_mps``); every node from now
on is a class. ``ros2 node list`` printed nothing: the process had
ended.

A Timer Calls a Method Once per Period
--------------------------------------

.. code: ros/lecture8/competition_nodes/competition_nodes/heartbeat_node.py all

.. code-block:: python
   :caption: competition_nodes/heartbeat_node.py

   """A node that stays alive and runs a timer callback (Lecture 8)."""

   import rclpy
   from rclpy.executors import ExternalShutdownException
   from rclpy.node import Node


   class HeartbeatNode(Node):
       """Logs a numbered heartbeat once per second."""

       def __init__(self):
           super().__init__('heartbeat')
           self._count = 0
           # Call self._on_timer every 1.0 s (chosen for this example)
           self._timer = self.create_timer(1.0, self._on_timer)

       def _on_timer(self):
           self._count += 1
           self.get_logger().info(f'Heartbeat {self._count}')


   def main(args=None):
       rclpy.init(args=args)
       node = HeartbeatNode()
       try:
           rclpy.spin(node)  # Runs callbacks until Ctrl+C
       except (KeyboardInterrupt, ExternalShutdownException):
           pass
       except Exception:  # Ctrl+C can also end spin() with an RCLError
           if rclpy.ok():  # ROS 2 still running: a real error
               raise
       finally:
           node.destroy_node()
           rclpy.try_shutdown()

- ``create_timer(1.0, self._on_timer)`` asks ROS 2 to call
  ``self._on_timer`` every 1.0 s. ``self._on_timer`` has no parentheses:
  the method is passed, not called. It is a **callback** (Lecture 4, "A
  Callback Is a Function That Other Code Calls Later").
- ``self._count`` lives as long as the node, so each call adds 1 to the
  last count.
- The node keeps every timer it makes, even one not stored:
  ``create_timer`` adds it to the node's list. Measured with a timer
  whose result was not kept:

.. output: timer_kept.txt all

.. code-block:: text

   timers the node keeps: 1

- ``self._timer`` lets the class cancel or reset the timer later.

spin() Runs the Callbacks Until Ctrl+C
--------------------------------------

.. code-block:: bash

   ros2 run competition_nodes heartbeat_node

.. output: heartbeat.txt all

.. code-block:: text

   [INFO] [heartbeat]: Heartbeat 1
   [INFO] [heartbeat]: Heartbeat 2
   [INFO] [heartbeat]: Heartbeat 3

- The ROS 2 tutorial: the node "spins" "(using spin()) so its callbacks
  are called." Without ``spin()``, the timer never fires.
- The first ``except`` and the ``finally`` of ``main()`` are those of the
  talker of ``demo_nodes_py`` in Jazzy
  (``/opt/ros/jazzy/lib/python3.12/site-packages/demo_nodes_py/topics/talker.py``);
  `What Ctrl+C Does to spin()`_ explains both ``except`` clauses. The
  node ran until Ctrl+C; its first three lines are shown.

What Ctrl+C Does to spin()
--------------------------

.. code: lecture8/spin_probe.py all

.. code-block:: python
   :caption: lecture8/spin_probe.py

   """What rclpy.spin() raises when the process gets a signal.

   Needs ROS 2 Jazzy sourced. Stop it with Ctrl+C (SIGINT), or with kill
   (SIGTERM). Prints the exception, whether ROS 2 is still running, and
   what a second rclpy.shutdown() does.
   """

   import rclpy
   from rclpy.executors import ExternalShutdownException

   rclpy.init()
   node = rclpy.create_node("spin_probe")
   try:
       rclpy.spin(node)
   except (KeyboardInterrupt, ExternalShutdownException) as err:
       print("spin raised", type(err).__name__, "| rclpy.ok():", rclpy.ok())
   node.destroy_node()
   try:
       rclpy.shutdown()
   except RuntimeError as err:
       print("rclpy.shutdown() again:", type(err).__name__)
   rclpy.try_shutdown()
   print("rclpy.try_shutdown(): no error")

Run once and stopped with Ctrl+C (SIGINT), then run again and stopped
with ``kill`` (SIGTERM):

.. output: spin_probe.txt all

.. code-block:: text

   spin raised KeyboardInterrupt | rclpy.ok(): False
   rclpy.shutdown() again: RCLError
   rclpy.try_shutdown(): no error
   spin raised ExternalShutdownException | rclpy.ok(): False
   rclpy.shutdown() again: RCLError
   rclpy.try_shutdown(): no error

1. Ctrl+C makes ``spin()`` raise ``KeyboardInterrupt``; the signal
   ``SIGTERM`` (what ``kill`` sends) makes it raise
   ``ExternalShutdownException``. The first ``except`` of
   `spin() Runs the Callbacks Until Ctrl+C`_ catches both.
2. In both cases ROS 2 is already shut down (``rclpy.ok()`` is
   ``False``), and a second ``rclpy.shutdown()`` raises ``RCLError``.
3. ``rclpy.try_shutdown()`` shuts down only if needed, so ``finally``
   can call it in every case.
4. Sometimes Ctrl+C shuts ROS 2 down in the middle of a step of
   ``spin()``: while it starts its next wait, or while a callback
   publishes. ``spin()`` then raises ``RCLError`` ("failed to initialize
   wait set: the given context is not valid", or "Failed to publish:
   publisher's context is invalid"). With only the first ``except``, that
   printed a traceback in 9 of 107 measured runs of two nodes stopped
   together by a launch file (Lecture 9), and in 1 of 40 runs of two
   ``ros2 run`` commands. ``rclpy.ok()`` is ``False`` after the shutdown,
   so the second ``except`` drops the error; an error raised while ROS 2
   still runs is raised again. With it, Lecture 9 measured no traceback
   (Lecture 9, "Ctrl+C Stops Every Node of the Launch").

A Message Is a Python Object with One Attribute per Field
---------------------------------------------------------

.. code: lecture8/message_demo.py all

.. code-block:: python
   :caption: lecture8/message_demo.py

   """A ROS 2 message is a Python object with one attribute per field.

   Needs ROS 2 Jazzy sourced. Makes one message; no node, nothing sent.
   """

   from sensor_msgs.msg import BatteryState

   msg = BatteryState()  # Every field starts at its default
   print(msg.percentage, msg.present, msg.location == "")
   msg.percentage = 0.16  # The message uses 0 to 1: 16% is 0.16
   msg.present = True
   print(msg.percentage, msg.present)
   msg.percentage = "full"  # Not a float, and no error yet
   print(msg.percentage)
   try:
       msg.charge_level = 3  # Not a field of BatteryState
   except AttributeError as err:
       print(type(err).__name__, err)

.. output: message_demo.txt all

.. code-block:: text

   0.0 False True
   0.16 True
   full
   AttributeError 'BatteryState' object has no attribute 'charge_level'

The package ``sensor_msgs`` has a Python module ``msg`` with one class
per message type. A wrong field name is refused; a wrong field type is
not checked here (Lecture 6 found the same for a ``Point``).

A Wrong Field Type Stops the Publisher's Process
------------------------------------------------

.. code: lecture8/errors/bad_publish.py all

.. code-block:: python
   :caption: lecture8/errors/bad_publish.py

   """Publishing a message whose field has the wrong type.

   Needs ROS 2 Jazzy sourced, with ROS_DOMAIN_ID set (not 0 in class).
   """

   import rclpy
   from sensor_msgs.msg import BatteryState

   rclpy.init()
   node = rclpy.create_node("bad_publisher")
   publisher = node.create_publisher(BatteryState, "battery_test", 10)
   msg = BatteryState()
   msg.percentage = "full"  # Accepted: fields are not checked here
   print("publishing", flush=True)
   publisher.publish(msg)  # The conversion to C checks the type
   print("published")  # Never printed

.. output: err_bad_publish.txt all

.. code-block:: text

   publishing
   python: ./.obj-x86_64-linux-gnu/rosidl_generator_py/sensor_msgs/msg/_battery_state_s.c:133: sensor_msgs__msg__battery_state__convert_from_py: Assertion `PyFloat_Check(field)' failed.
   exit status 134

- The C code that converts the message checks the type, and its failed
  assertion stops the process: exit status 134 is 128 + 6, the signal
  ``SIGABRT``. No ``except`` can catch it.
- With ``ROS_PYTHON_CHECK_FIELDS=1`` set, the assignment in
  ``message_demo.py`` raises at once, in Python (last line of its
  traceback):

.. output: message_demo_checked.txt all

.. code-block:: text

   AssertionError: The 'percentage' field must be of type 'float'

Step 1 of a Publisher: create_publisher() and a Timer
-----------------------------------------------------

.. code: ros/lecture8/competition_nodes/competition_nodes/battery_publisher.py all

.. code-block:: python
   :caption: competition_nodes/battery_publisher.py

   """Publish a robot's battery level as it drains (Lecture 8)."""

   import rclpy
   from rclpy.executors import ExternalShutdownException
   from rclpy.node import Node
   from sensor_msgs.msg import BatteryState

   DRAIN_PCT_PER_MIN = 2  # Rule 4 of Lecture 6, chosen for that example
   # One message per second stands for one minute (chosen for this example)


   class BatteryPublisher(Node):
       """Publishes the battery level once per second."""

       def __init__(self):
           super().__init__('battery_publisher')
           self._battery = 16  # Percent, chosen to cross 10% soon
           self._publisher = self.create_publisher(
               BatteryState, 'battery', 10)  # Topic name, queue size
           self._timer = self.create_timer(1.0, self._publish_battery)

       def _publish_battery(self):
           msg = BatteryState()
           msg.percentage = self._battery / 100  # The message uses 0 to 1
           msg.present = True
           self._publisher.publish(msg)
           self.get_logger().info(f'Published {self._battery}%')
           self._battery = max(0, self._battery - DRAIN_PCT_PER_MIN)


   def main(args=None):
       rclpy.init(args=args)
       node = BatteryPublisher()
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

- ``create_publisher()`` takes the message type, the topic name, and the
  queue size. ``'battery'`` has no ``/``, so the full name is
  ``/battery``.
- The ROS 2 tutorial: the "queue size" is "a required Quality of Service
  (QoS) setting that limits the amount of queued messages if a
  subscriber is not receiving them fast enough." 10 is the tutorial's
  value.

Step 2 of a Publisher: The Timer Callback Publishes a Message
-------------------------------------------------------------

- Each call of ``_publish_battery()`` makes a new message, fills two
  fields, publishes it, and logs the level. The other fields keep their
  defaults.
- Rule 1 of Lecture 6 keeps the battery a whole percent; the message
  wants 0 to 1, so 16 is sent as 16 / 100 = 0.16.
- Rule 4 of Lecture 6: "A task uses 2% of the battery for each minute it
  takes." The publisher treats the robot as always working on a task,
  and one call stands for one minute, so the level drops by 2 at each
  call. ``max(0, ...)`` stops it at 0.
- ``main()`` is the one of `spin() Runs the Callbacks Until Ctrl+C`_,
  with ``BatteryPublisher()``.

A Subscriber: create_subscription() and Its Callback
----------------------------------------------------

.. code: ros/lecture8/competition_nodes/competition_nodes/battery_monitor.py all

.. code-block:: python
   :caption: competition_nodes/battery_monitor.py

   """Subscribe to the battery level and report the status (Lecture 8)."""

   import rclpy
   from rclpy.executors import ExternalShutdownException
   from rclpy.node import Node
   from sensor_msgs.msg import BatteryState

   LOW_BATTERY_PCT = 10  # Rule 3 of Lecture 6, chosen for that example


   class BatteryMonitor(Node):
       """Logs each battery level it receives, with the robot's status."""

       def __init__(self):
           super().__init__('battery_monitor')
           self._subscription = self.create_subscription(
               BatteryState, 'battery', self._on_battery, 10)

       def _on_battery(self, msg):
           level = round(msg.percentage * 100)  # A whole percent (rule 1)
           if level < LOW_BATTERY_PCT:
               self.get_logger().warning(f'{level}%: needs_recharge')
           else:
               self.get_logger().info(f'{level}%: active')


   def main(args=None):
       rclpy.init(args=args)
       node = BatteryMonitor()
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

- ``create_subscription()`` takes the type, the topic name, the
  callback, and the queue size. Type and name match the publisher's
  (rules 1 and 2 of `Four Rules of a Topic`_).
- ROS 2 calls ``_on_battery(msg)`` once for each message, while the node
  spins. Rule 3: below 10%, the status is ``needs_recharge``, logged as a
  warning.

Running the Publisher and the Monitor
-------------------------------------

Terminal 1, started first:

.. code-block:: bash

   ros2 run competition_nodes battery_monitor

.. output: battery_monitor.txt all

.. code-block:: text

   [INFO] [battery_monitor]: 16%: active
   [INFO] [battery_monitor]: 14%: active
   [INFO] [battery_monitor]: 12%: active
   [INFO] [battery_monitor]: 10%: active
   [WARN] [battery_monitor]: 8%: needs_recharge

Terminal 2:

.. code-block:: bash

   ros2 run competition_nodes battery_publisher

.. output: battery_publisher.txt all

.. code-block:: text

   [INFO] [battery_publisher]: Published 16%
   [INFO] [battery_publisher]: Published 14%
   [INFO] [battery_publisher]: Published 12%
   [INFO] [battery_publisher]: Published 10%
   [INFO] [battery_publisher]: Published 8%

Both ran until Ctrl+C; the first five lines of each are shown.

The Arithmetic Behind the Monitor's Output
------------------------------------------

1. The publisher starts at 16% (chosen for this example) and sends
   16 / 100 = 0.16.
2. Rule 4: "A task uses 2% of the battery for each minute it takes."
   The robot is always working on a task, and each timer call stands for
   one minute: 16 - 2 = 14, then 12, then 10, then 10 - 2 = 8.
3. The monitor turns each message back into a whole percent with
   ``round()`` (`Why the Monitor Rounds: percentage Is a float32`_ shows
   why it must round).
4. Rule 3: below 10%, the status is ``needs_recharge``. 16, 14, and 12
   are not below 10. 10 is not below 10 either, so 10% is still
   ``active``.
5. 8 is below 10: the monitor logs ``8%: needs_recharge`` with severity
   ``WARN``.

The rule says "below 10%", so the code uses ``<``, not ``<=``. A test at
exactly 10% is how you check that choice.

Why the Monitor Rounds: percentage Is a float32
-----------------------------------------------

Started before the publisher, so that it gets the first message:

.. code-block:: bash

   ros2 topic echo --once --field percentage \
       /battery sensor_msgs/msg/BatteryState

.. output: echo_percentage.txt all

.. code-block:: text

   0.1599999964237213
   ---

1. The publisher set 0.16, a Python ``float``. The field is ``float32``
   (`A Message Type Lists Typed Fields`_), so the message carries the
   nearest ``float32``: 0.1599999964237213 (Lecture 5, "float32 Is Less
   Precise than float64").
2. The monitor computes 0.1599999964237213 × 100 = 15.999999642372131,
   and ``round()`` gives 16.
3. Without ``round()``, ``int()`` would cut it to 15, one percent too
   low.

Inspecting the Battery Nodes
----------------------------

While both nodes run:

.. code-block:: bash

   ros2 node info /battery_monitor

.. output: node_info_monitor.txt 1-19

.. code-block:: text

   /battery_monitor
     Subscribers:
       /battery: sensor_msgs/msg/BatteryState
     Publishers:
       /parameter_events: rcl_interfaces/msg/ParameterEvent
       /rosout: rcl_interfaces/msg/Log
     Service Servers:
       /battery_monitor/describe_parameters: rcl_interfaces/srv/DescribeParameters
       /battery_monitor/get_parameter_types: rcl_interfaces/srv/GetParameterTypes
       /battery_monitor/get_parameters: rcl_interfaces/srv/GetParameters
       /battery_monitor/get_type_description: type_description_interfaces/srv/GetTypeDescription
       /battery_monitor/list_parameters: rcl_interfaces/srv/ListParameters
       /battery_monitor/set_parameters: rcl_interfaces/srv/SetParameters
       /battery_monitor/set_parameters_atomically: rcl_interfaces/srv/SetParametersAtomically
     Service Clients:

     Action Servers:

     Action Clients:



.. code-block:: bash

   ros2 topic info /battery

.. output: topic_info_battery.txt all

.. code-block:: text

   Type: sensor_msgs/msg/BatteryState
   Publisher count: 1
   Subscription count: 1

The graph is tag 1 of `One Topic, Many Subscribers or Many Publishers`_,
without ``low_battery_alarm``. With Exercise 3's node running too,
``Subscription count`` is 2 (see the Exercise 3 solution).
