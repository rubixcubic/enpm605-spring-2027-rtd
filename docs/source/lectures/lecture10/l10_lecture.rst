====================================================
Lecture
====================================================

This lecture uses the ROS repository,
`enpm605-spring-2027-ros <https://github.com/rubixcubic/enpm605-spring-2027-ros>`_.
Its folder ``lecture10/`` holds five new packages and one script:

- ``competition_interfaces`` (an ament_cmake package): the message
  ``RobotStatus``, the service ``AssignTask``, and the action
  ``DriveToGoal``.
- ``competition_params``: Lecture 7's scout, with its top speed as a
  parameter, a parameters file, and a launch file.
- ``competition_services``: a service server that assigns tasks, and
  three kinds of client.
- ``competition_actions``: an action server that drives a distance, and
  an action client.
- ``competition_qos``: a publisher and a subscriber whose QoS profile is
  chosen by parameters.
- ``errors/field_checks.py``: a script, run with ``python3``, that sets
  wrong values in a message.

Run ``git pull`` in the clone in ``~/ros_ws/src``, then build the
workspace with ``colcon build --symlink-install`` and source
``install/setup.bash``. The packages use Lecture 8's
``competition_nodes`` and Lecture 9's ``competition_executors``. This
lecture has no plain Python program, so the course repository has no
``lecture10/`` folder.

Every output on this page was printed by running the code with ROS 2
Jazzy on Ubuntu 24.04, with ``ROS_DOMAIN_ID=77`` and
``ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST``, in a workspace whose ROS
repository held ``lecture8/`` to ``lecture10/``: what the repository
holds on Apr 13. Log lines were recorded without their time stamp
(Lecture 8, "Reading a Log Line"), and a node that runs until Ctrl+C
shows its first lines only. Goal IDs and process IDs are replaced by
``...``, and a long error line is folded to fit, where the text says so.
The numbers of the examples (a 30% battery, a speed limit of 3.0 m/s, a
step of 0.5 s, a message every 0.5 s) are chosen for the example.


Parameters
====================================================

Where Lecture 9 Stopped
-----------------------

Lecture 9 set ``max_speed_mps`` on ``parameter_blackboard``, a demo node
that keeps any parameter it is given. Lecture 8's scout writes its speed
in the code, ``self.max_speed_mps = 2.0``. Given ``-p`` anyway:

.. code-block:: bash

   ros2 run competition_nodes scout_node --ros-args -p max_speed_mps:=1.5

.. output: p_l8_scout.txt all

.. code-block:: text

   [INFO] [scout]: Top speed 2.0 m/s

The rule changed nothing, and nothing warned. The ROS 2 documentation,
"Parameters": "By default, a node needs to declare all of the parameters
that it will accept during its lifetime." To **declare** a parameter is
to tell the node its name and its default value; only then does ``-p``
reach it.

Declaring and Reading a Parameter
---------------------------------

``competition_params/scout_node.py`` declares two parameters:

.. code: ros/lecture10/competition_params/competition_params/scout_node.py all

.. code-block:: python
   :caption: competition_params/competition_params/scout_node.py

   """Lecture 7's scout, with its top speed as a parameter (Lecture 10)."""

   from rcl_interfaces.msg import FloatingPointRange, ParameterDescriptor
   import rclpy
   from rclpy.executors import ExternalShutdownException
   from rclpy.node import Node

   SPEED_LIMIT_MPS = 3.0  # Highest top speed allowed (chosen)


   class ScoutNode(Node):
       """Declares two parameters and logs the top speed once per second."""

       def __init__(self):
           super().__init__('scout')
           speed = ParameterDescriptor(
               description='Top speed, in m/s',
               floating_point_range=[FloatingPointRange(
                   from_value=0.0, to_value=SPEED_LIMIT_MPS)])
           name = ParameterDescriptor(
               description='Name of the robot',
               read_only=True)  # Lecture 6, rule 5: names do not change
           self.declare_parameter('max_speed_mps', 2.0, speed)
           self.declare_parameter('robot_name', 'Scout', name)
           self._name = self.get_parameter('robot_name').value
           self._speed = self.get_parameter('max_speed_mps').value  # Once
           self._timer = self.create_timer(1.0, self._report)

       def _report(self):
           self.get_logger().info(f'{self._name}: {self._speed} m/s')


   class LiveScoutNode(ScoutNode):
       """The same node, told when a parameter has changed."""

       def __init__(self):
           super().__init__()
           self.add_post_set_parameters_callback(self._on_parameters_set)

       def _on_parameters_set(self, parameters):
           for parameter in parameters:  # Each parameter just set
               if parameter.name == 'max_speed_mps':
                   self._speed = parameter.value


   def run(node):
       """Spin the node until Ctrl+C (Lecture 8's main())."""
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


   def main(args=None):
       rclpy.init(args=args)
       run(ScoutNode())


   def live(args=None):
       rclpy.init(args=args)
       run(LiveScoutNode())

- ``declare_parameter(name, default, descriptor)``. The default's type
  is the parameter's type: ``2.0`` makes ``max_speed_mps`` a double.
- The **descriptor** (``rcl_interfaces/msg/ParameterDescriptor``) holds
  facts about the parameter: a description, a range, and whether it is
  read-only.
- ``get_parameter(name).value`` reads the value now. ``ScoutNode`` reads
  each value once, in ``__init__``. ``LiveScoutNode`` comes in `A
  Callback After Each Set`_.
- ``run()`` is the body of Lecture 8's ``main()``: spin, and stop on
  Ctrl+C.

With the default, with ``-p``, and with a parameter the node does not
declare (the first line of each run; the node logs once per second):

.. code-block:: bash

   ros2 run competition_params scout_node
   ros2 run competition_params scout_node --ros-args -p max_speed_mps:=1.5
   ros2 run competition_params scout_node --ros-args -p top_speed:=1.0

.. output: p_default.txt 1

.. code-block:: text

   [INFO] [scout]: Scout: 2.0 m/s

.. output: p_override.txt 1

.. code-block:: text

   [INFO] [scout]: Scout: 1.5 m/s

.. output: p_undeclared.txt 1

.. code-block:: text

   [INFO] [scout]: Scout: 2.0 m/s

``top_speed`` is not declared, so the node ignored it, with no warning.

A Wrong Value Stops the Node at Start
-------------------------------------

An integer for a double, then a value above the range of the descriptor
(0.0 to 3.0, ``SPEED_LIMIT_MPS``). The last lines of each run, folded to
fit:

.. code-block:: bash

   ros2 run competition_params scout_node --ros-args -p max_speed_mps:=2

.. output: p_wrong_type.txt all

.. code-block:: text

   InvalidParameterTypeException: Trying to set parameter 
   'max_speed_mps' to '2' of type 'INTEGER', expecting type 
   'DOUBLE': max_speed_mps
   [ros2run]: Process exited with failure 1

.. code-block:: bash

   ros2 run competition_params scout_node --ros-args -p max_speed_mps:=5.0

.. output: p_out_of_range.txt all

.. code-block:: text

   InvalidParameterValueException: Invalid parameter value (5.0) for 
   parameter. Reason: Parameter max_speed_mps out of range Min: 0.0, 
   Max: 3.0, value: 5.0: max_speed_mps
   [ros2run]: Process exited with failure 1

``declare_parameter()`` raised an exception in both runs, and the node
stopped. Write ``2.0``, not ``2``, for a double. The docs on types: "This
prevents common mistakes, such as putting a boolean value into an
integer parameter."

What the Descriptor Says
------------------------

While the scout runs:

.. code-block:: bash

   ros2 param list /scout
   ros2 param describe /scout max_speed_mps robot_name

.. output: p_list.txt all

.. code-block:: text

     max_speed_mps
     robot_name
     start_type_description_service
     use_sim_time

.. output: p_describe.txt all

.. code-block:: text

   Parameter name: max_speed_mps
     Type: double
     Description: Top speed, in m/s
     Constraints:
       Min value: 0.0
       Max value: 3.0
   Parameter name: robot_name
     Type: string
     Description: Name of the robot
     Constraints:
       Read only: true

The docs: "By default all descriptors are empty, but can contain
parameter descriptions, value ranges, type information, and additional
constraints." ``robot_name`` is read-only, for Lecture 6's rule 5: "A
robot's name does not change after the robot is made." It can be set
when the node starts, never later. ``use_sim_time`` and
``start_type_description_service`` are declared by every node.

A Parameters File
-----------------

A YAML file holds the values for one or more nodes. Its first key is the
node's name; the values go under ``ros__parameters``:

.. code: ros/lecture10/competition_params/config/scout.yaml all

.. code-block:: yaml
   :caption: competition_params/config/scout.yaml

   # Parameters of the node named scout, read when it starts
   scout:
     ros__parameters:
       max_speed_mps: 1.5   # Lecture 7's is 2.0; 1.5 is chosen
       robot_name: Scout_2

``setup.py`` installs the file, with one line added to what
``ros2 pkg create`` wrote (and the launch folder, as in Lecture 9):

.. code: ros/lecture10/competition_params/setup.py 1-17

.. code-block:: python
   :caption: competition_params/setup.py, lines 1 to 17

   from glob import glob

   from setuptools import find_packages, setup

   package_name = 'competition_params'

   setup(
       name=package_name,
       version='0.0.0',
       packages=find_packages(exclude=['test']),
       data_files=[
           ('share/ament_index/resource_index/packages',
               ['resource/' + package_name]),
           ('share/' + package_name, ['package.xml']),
           ('share/' + package_name + '/config', glob('config/*.yaml')),
           ('share/' + package_name + '/launch', glob('launch/*')),
       ],

From ``~/ros_ws``, where ``install`` is:

.. code-block:: bash

   cd install/competition_params/share/competition_params
   ros2 run competition_params scout_node --ros-args \
       --params-file config/scout.yaml

.. output: p_yaml.txt 1

.. code-block:: text

   [INFO] [scout]: Scout_2: 1.5 m/s

``ros2 pkg prefix competition_params`` prints the package's install
folder, if you run from another folder.

A Parameters File in a Launch File
----------------------------------

.. code: ros/lecture10/competition_params/launch/scout_launch.py all

.. code-block:: python
   :caption: competition_params/launch/scout_launch.py

   """Start the scout with the parameters of config/scout.yaml."""

   from launch import LaunchDescription
   from launch.substitutions import PathJoinSubstitution
   from launch_ros.actions import Node
   from launch_ros.substitutions import FindPackageShare


   def generate_launch_description():
       config = PathJoinSubstitution([
           FindPackageShare('competition_params'), 'config', 'scout.yaml'])
       scout = Node(package='competition_params', executable='scout_node',
                    parameters=[config])  # A file, not a dictionary
       return LaunchDescription([scout])

.. code-block:: bash

   ros2 launch competition_params scout_launch.py

.. output: p_launch.txt all

.. code-block:: text

   [INFO] [scout_node-1]: process started with pid [...]
   [scout_node-1] [INFO] [scout]: Scout_2: 1.5 m/s
   [scout_node-1] [INFO] [scout]: Scout_2: 1.5 m/s

What launch passed to the process (``ps -o args``, folders shortened):

.. output: p_launch_ps.txt all

.. code-block:: text

   scout_node --ros-args
       --params-file .../share/competition_params/config/scout.yaml

``FindPackageShare`` and ``PathJoinSubstitution`` are Lecture 9's. A
path in ``parameters=`` is passed as it is, with ``--params-file``;
Lecture 9's dictionary was first written to a file by launch.

Setting a Parameter While the Node Runs
---------------------------------------

In terminal 1, ``ros2 run competition_params scout_node``; in terminal 2,
after its third line:

.. code-block:: bash

   ros2 param set /scout max_speed_mps 1.0
   ros2 param get /scout max_speed_mps

.. output: p_set_scout_node_cli.txt all

.. code-block:: text

   Set parameter successful
   Double value is: 1.0

Terminal 1: its first three lines, then the three lines it logged after
``ros2 param get`` returned (set and get take about 2 s each, and the
lines logged meanwhile are left out):

.. output: p_set_scout_node.txt all

.. code-block:: text

   [INFO] [scout]: Scout: 2.0 m/s
   [INFO] [scout]: Scout: 2.0 m/s
   [INFO] [scout]: Scout: 2.0 m/s
   [INFO] [scout]: Scout: 2.0 m/s
   [INFO] [scout]: Scout: 2.0 m/s
   [INFO] [scout]: Scout: 2.0 m/s

The parameter is 1.0, and the node still logs 2.0: it copied the value
into ``self._speed`` once, in ``__init__``, and nothing told it about the
change. A node that reads the parameter each time it needs it, as the
action server of `Actions`_ does, sees the new value at once.

A Callback After Each Set
-------------------------

``LiveScoutNode`` (in the same file, a subclass as in Lecture 7) adds a
**post set callback**: rclpy calls it with the list of parameters that
were just set. With ``ros2 run competition_params scout_live`` and the
same ``ros2 param set`` (the same lines kept: 1 to 3, then the three
after ``ros2 param get`` returned):

.. output: p_set_scout_live.txt all

.. code-block:: text

   [INFO] [scout]: Scout: 2.0 m/s
   [INFO] [scout]: Scout: 2.0 m/s
   [INFO] [scout]: Scout: 2.0 m/s
   [INFO] [scout]: Scout: 1.0 m/s
   [INFO] [scout]: Scout: 1.0 m/s
   [INFO] [scout]: Scout: 1.0 m/s

The docs name three parameter callbacks. The post set callback is there
"to react to changes from parameters that have successfully been
accepted." The "set parameter" callback, added with
``add_on_set_parameters_callback()``, can refuse a value, and must not
change the node: "It is important that “set parameter” callbacks have no
side-effects." (A later callback could still refuse the change, and the
node would then hold a value the parameter does not have.) A range in the
descriptor refuses values without any callback.

Three Sets the Node Refuses
---------------------------

.. code-block:: bash

   ros2 param set /scout max_speed_mps 4.0
   ros2 param set /scout max_speed_mps 2
   ros2 param set /scout robot_name Scout_9

.. output: p_set_refused.txt all

.. code-block:: text

   Setting parameter failed: Parameter max_speed_mps out of range Min: 0.0, Max: 3.0, value: 4.0
   Setting parameter failed: Wrong parameter type, expected 'Type.DOUBLE' got 'Type.INTEGER'
   Setting parameter failed: Trying to set a read-only parameter: robot_name.

1. Out of the range: refused, and the value stays.
2. The wrong type: the docs, "By default, attempts to change the type of
   a declared parameter at runtime will fail."
3. Read-only: refused.

The node keeps running in all three. At start, a wrong type or a value
out of range stopped it (`A Wrong Value Stops the Node at Start`_). A
read-only parameter is different: it can be set at start, and is refused
only on later sets. With ``--ros-args -p robot_name:=Scout_7``:

.. output: p_readonly_start.txt all

.. code-block:: text

   [INFO] [scout]: Scout_7: 2.0 m/s

ros2 param Calls the Node's Services
------------------------------------

Lecture 8 saw seven service servers on every node. Those of ``/scout``
(``ros2 service list -t``, the lines that start with ``/scout/``); six
of them are the parameter services the docs list, and
``get_type_description`` (``type_description_interfaces``) is not one:

.. output: p_services.txt all

.. code-block:: text

   /scout/describe_parameters [rcl_interfaces/srv/DescribeParameters]
   /scout/get_parameter_types [rcl_interfaces/srv/GetParameterTypes]
   /scout/get_parameters [rcl_interfaces/srv/GetParameters]
   /scout/get_type_description [type_description_interfaces/srv/GetTypeDescription]
   /scout/list_parameters [rcl_interfaces/srv/ListParameters]
   /scout/set_parameters [rcl_interfaces/srv/SetParameters]
   /scout/set_parameters_atomically [rcl_interfaces/srv/SetParametersAtomically]

The docs: "External processes can perform parameter operations via
parameter services that are created by default when a node is
instantiated." and "ros2 param uses the parameter service API as
described above to perform the various operations." ``ros2 param set``
calls ``/scout/set_parameters``. Services are the third part of
this lecture.

Every change is also published on ``/parameter_events``, the topic
Lecture 8 left for this lecture. ``ros2 topic echo /parameter_events``
during ``ros2 param set /scout max_speed_mps 1.0`` (the message, without
its time stamp). The echo can miss the first set if it has not found the
topic's publishers yet (it missed it once in each of the last recordings):
run the set again.

.. output: p_events.txt all

.. code-block:: text

   node: /scout
   new_parameters: []
   changed_parameters:
   - name: max_speed_mps
     value:
       type: 3
       bool_value: false
       integer_value: 0
       double_value: 1.0
       string_value: ''
       byte_array_value: []
       bool_array_value: []
       integer_array_value: []
       double_array_value: []
       string_array_value: []
   deleted_parameters: []
   ---

A ``ParameterValue`` has one field per type; ``type: 3`` is a double,
so ``double_value`` holds the value.


Custom Interfaces
====================================================

Three Kinds of Interface
------------------------

Lecture 8's battery nodes used ``sensor_msgs/msg/BatteryState``: it has
a ``percentage``, but no robot name and no status of Lecture 6. A
message type of our own can have both. The ROS 2 documentation,
"Interfaces":

.. list-table::
   :header-rows: 1
   :widths: 15 55 30
   :class: compact-table

   * - File
     - What the docs say
     - This lecture
   * - ``.msg``
     - "simple text files that describe the fields of a ROS message"
     - ``RobotStatus``
   * - ``.srv``
     - "composed of two parts: a request and a response"
     - ``AssignTask``
   * - ``.action``
     - "composed of three parts: a goal, a result, and feedback"
     - ``DriveToGoal``

From each file, colcon generates a class for Python and one for C++, so
nodes in either language send the same messages (Lecture 8).

An Interface Package
--------------------

``competition_interfaces`` was made in ``lecture10/`` with:

.. code-block:: bash

   ros2 pkg create competition_interfaces --build-type ament_cmake \
       --license Apache-2.0

Its files (``find competition_interfaces | LC_ALL=C sort``):

.. output: i_tree.txt all

.. code-block:: text

   competition_interfaces
   competition_interfaces/CMakeLists.txt
   competition_interfaces/LICENSE
   competition_interfaces/action
   competition_interfaces/action/DriveToGoal.action
   competition_interfaces/msg
   competition_interfaces/msg/RobotStatus.msg
   competition_interfaces/package.xml
   competition_interfaces/srv
   competition_interfaces/srv/AssignTask.srv

The tutorial "Creating custom msg and srv files": "Note that it is, and
can only be, an ament_cmake package, but this doesn’t restrict in which
type of packages you can use your messages and services." Each file goes
in the folder of its kind: "The .msg and .srv files are required to be
placed in directories called msg and srv respectively", and an action
in ``action/``. The empty ``src/`` and ``include/`` folders that
``ros2 pkg create`` made were removed.

The Message, the Service, and the Action
----------------------------------------

.. code: ros/lecture10/competition_interfaces/msg/RobotStatus.msg all

.. code-block:: text
   :caption: competition_interfaces/msg/RobotStatus.msg

   # The status of one robot (Lecture 6, rules 1 and 3)
   uint8 ACTIVE=0          # 10% or more
   uint8 NEEDS_RECHARGE=1  # Below 10%: the robot does no tasks

   string name             # The robot's name
   uint8 battery_pct       # A whole percent, 0 to 100
   uint8 status            # ACTIVE or NEEDS_RECHARGE

- A field is a type and a name: ``string``, ``uint8`` (an integer from 0
  to 255), ``float64``, ``bool``, or another message type such as
  ``geometry_msgs/Point``.
- A constant has an ``=`` and a value. The docs: "Each constant
  definition is like a field description with a default value, except
  that this value can never be changed programmatically." Constant names
  are in capitals. ``status`` holds one of the two.
- ``#`` starts a comment, as in Python.

.. code: ros/lecture10/competition_interfaces/srv/AssignTask.srv all

.. code-block:: text
   :caption: competition_interfaces/srv/AssignTask.srv

   # Request
   string task        # What to do
   uint32 minutes     # How long
   ---
   # Response
   bool accepted      # True: done
   uint8 battery_pct  # Level after
   string message     # In words

.. code: ros/lecture10/competition_interfaces/action/DriveToGoal.action all

.. code-block:: text
   :caption: competition_interfaces/action/DriveToGoal.action

   # Goal: meters to drive
   float64 distance_m
   ---
   # Result: meters driven
   float64 distance_driven_m
   ---
   # Feedback: meters left
   float64 distance_remaining_m

``---`` separates the parts. A service has the request, then the
response. The tutorial "Creating an action": "An action definition is
made up of three message definitions separated by ---." The goal, the
result, then the feedback.

CMakeLists.txt and package.xml
------------------------------

.. code: ros/lecture10/competition_interfaces/CMakeLists.txt all

.. code-block:: cmake
   :caption: competition_interfaces/CMakeLists.txt

   cmake_minimum_required(VERSION 3.8)
   project(competition_interfaces)

   if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
     add_compile_options(-Wall -Wextra -Wpedantic)
   endif()

   # find dependencies
   find_package(ament_cmake REQUIRED)
   find_package(rosidl_default_generators REQUIRED)

   # Generate the Python and C++ code of the three interfaces
   rosidl_generate_interfaces(${PROJECT_NAME}
     "msg/RobotStatus.msg"
     "srv/AssignTask.srv"
     "action/DriveToGoal.action"
   )

   if(BUILD_TESTING)
     find_package(ament_lint_auto REQUIRED)
     # the following line skips the linter which checks for copyrights
     # comment the line when a copyright and license is added to all source files
     set(ament_cmake_copyright_FOUND TRUE)
     # the following line skips cpplint (only works in a git repo)
     # comment the line when this package is in a git repo and when
     # a copyright and license is added to all source files
     set(ament_cmake_cpplint_FOUND TRUE)
     ament_lint_auto_find_test_dependencies()
   endif()

   ament_package()

.. code: ros/lecture10/competition_interfaces/package.xml all

.. code-block:: xml
   :caption: competition_interfaces/package.xml

   <?xml version="1.0"?>
   <?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
   <package format="3">
     <name>competition_interfaces</name>
     <version>0.0.0</version>
     <description>Lecture 10 interfaces: robotics competition</description>
     <maintainer email="zeidk@umd.edu">Zeid Kootbally</maintainer>
     <license>Apache-2.0</license>

     <buildtool_depend>ament_cmake</buildtool_depend>
     <buildtool_depend>rosidl_default_generators</buildtool_depend>
     <exec_depend>rosidl_default_runtime</exec_depend>

     <test_depend>ament_lint_auto</test_depend>
     <test_depend>ament_lint_common</test_depend>

     <member_of_group>rosidl_interface_packages</member_of_group>

     <export>
       <build_type>ament_cmake</build_type>
     </export>
   </package>

Lines 10 to 17 of ``CMakeLists.txt`` and lines 11, 12, and 17 of
``package.xml`` were added to what ``ros2 pkg create`` wrote.
``<member_of_group>`` goes after the ``<test_depend>`` lines: the
package format 3 schema wants that order, and ``colcon test`` (its
``xmllint`` test) failed with it placed after line 12.
``rosidl_generate_interfaces`` lists every file to generate code from.
The tutorial: "rosidl_default_runtime is a runtime or execution-stage
dependency, needed to be able to use the interfaces later." An interface
that uses another package's type, such as ``geometry_msgs/Point``, adds
that package after ``DEPENDENCIES`` in ``rosidl_generate_interfaces`` and
a ``<depend>`` in ``package.xml``; these three use only built-in types.

Build, Then Check with ros2 interface
-------------------------------------

.. code-block:: bash

   colcon build --symlink-install     # In ~/ros_ws
   source install/setup.bash
   ros2 interface package competition_interfaces
   ros2 interface show competition_interfaces/msg/RobotStatus
   ros2 interface show competition_interfaces/srv/AssignTask
   ros2 interface show competition_interfaces/action/DriveToGoal

.. output: i_package.txt all

.. code-block:: text

   competition_interfaces/action/DriveToGoal
   competition_interfaces/msg/RobotStatus
   competition_interfaces/srv/AssignTask

.. output: i_show_msg.txt all

.. code-block:: text

   # The status of one robot (Lecture 6, rules 1 and 3)
   uint8 ACTIVE=0          # 10% or more
   uint8 NEEDS_RECHARGE=1  # Below 10%: the robot does no tasks

   string name             # The robot's name
   uint8 battery_pct       # A whole percent, 0 to 100
   uint8 status            # ACTIVE or NEEDS_RECHARGE

.. output: i_show_srv.txt all

.. code-block:: text

   # Request
   string task        # What to do
   uint32 minutes     # How long
   ---
   # Response
   bool accepted      # True: done
   uint8 battery_pct  # Level after
   string message     # In words

.. output: i_show_action.txt all

.. code-block:: text

   # Goal: meters to drive
   float64 distance_m
   ---
   # Result: meters driven
   float64 distance_driven_m
   ---
   # Feedback: meters left
   float64 distance_remaining_m

A type's full name is the package, the kind, and the file's name:
``competition_interfaces/srv/AssignTask``. A Python node imports it as
``from competition_interfaces.srv import AssignTask``.

The Classes colcon Generated
----------------------------

``print()`` of a new object of each class, then the two constants:

.. output: i_python.txt all

.. code-block:: text

   competition_interfaces.msg.RobotStatus(name='', battery_pct=0, status=0)
   0 1
   competition_interfaces.srv.AssignTask_Request(task='', minutes=0)
   competition_interfaces.srv.AssignTask_Response(accepted=False, battery_pct=0, message='')
   competition_interfaces.action.DriveToGoal_Goal(distance_m=0.0)

Every field starts at a default: ``''``, ``0``, ``0.0``, ``False``. A
service gives ``Request`` and ``Response``, the two you fill in, and
``Event`` (a ``service_msgs/ServiceEventInfo`` and the request or
response, used for service introspection; ``dir(AssignTask)`` lists the
three); an action gives ``Goal``, ``Result``, and ``Feedback``. A constant is read-only:
``RobotStatus.NEEDS_RECHARGE = 5`` raised

.. output: i_constant.txt all

.. code-block:: text

   AttributeError: property 'NEEDS_RECHARGE' of 'Metaclass_RobotStatus' object has no setter

The generated Python files are in the install folder, for example
``install/competition_interfaces/lib/python3.12/site-packages/competition_interfaces/msg``:

.. output: i_generated.txt all

.. code-block:: text

   __init__.py
   _robot_status.py
   _robot_status_s.c

A Field Is Checked Only If You Ask
----------------------------------

``battery_pct`` is a ``uint8``. ``errors/field_checks.py`` sets it to
30, to 300, then to the string ``'30'``, and publishes each message:

.. code: ros/lecture10/errors/field_checks.py all

.. code-block:: python
   :caption: lecture10/errors/field_checks.py

   """Publish RobotStatus messages with wrong battery_pct values (Lecture 10).

   Run with python3 after sourcing the workspace, once as it is and once
   with ROS_PYTHON_CHECK_FIELDS=1. Each value is tried in a fresh message.
   """

   import time

   from competition_interfaces.msg import RobotStatus
   import rclpy

   rclpy.init()
   node = rclpy.create_node('field_checks')
   publisher = node.create_publisher(RobotStatus, 'status', 10)
   time.sleep(2.0)  # Time for a subscriber to connect (chosen)
   for value in [30, 300, '30']:
       msg = RobotStatus(name='scout')
       try:
           msg.battery_pct = value
           publisher.publish(msg)
           print(f'{value!r}: published', flush=True)
       except AssertionError as error:
           print(f'{value!r}: AssertionError: {error}', flush=True)
       time.sleep(0.3)

Run from ``~/ros_ws`` with the workspace sourced, while
``ros2 topic echo /status competition_interfaces/msg/RobotStatus --field battery_pct``
runs in another terminal:

.. code-block:: bash

   python3 src/enpm605-spring-2027-ros/lecture10/errors/field_checks.py

.. output: i_field_checks.txt all

.. code-block:: text

   30: published
   300: published
   python3: _robot_status_s.c: Assertion `PyLong_Check(field)' failed.
   exit status: 134

What the echo received:

.. output: i_field_echo.txt all

.. code-block:: text

   30
   44

The same with ``ROS_PYTHON_CHECK_FIELDS=1``:

.. code-block:: bash

   ROS_PYTHON_CHECK_FIELDS=1 python3 src/enpm605-spring-2027-ros/lecture10/errors/field_checks.py

.. output: i_field_checks_on.txt all

.. code-block:: text

   30: published
   300: AssertionError: The 'battery_pct' field must be an unsigned integer in [0, 255]
   '30': AssertionError: The 'battery_pct' field must be of type 'int'
   exit status: 0

1. Without the check, 300 arrived as 44 (300 − 256 = 44: only the low 8
   bits of 300 fit in a ``uint8``), and ``'30'`` ended the whole process
   with an assertion in C code (exit status 134, an abort; the C file's
   folder is left out).
2. The generated setter checks a value only if ``ROS_PYTHON_CHECK_FIELDS``
   is ``1``: the generated ``_robot_status.py`` reads it with
   ``getenv('ROS_PYTHON_CHECK_FIELDS', default='')``. Set it in your
   terminal while you develop.


Services
====================================================

A Service Is a Request and a Reply
----------------------------------

.. figure:: /_static/images/l10/service_call.svg
   :alt: Two nodes, task_client on the left and task_server on the right. Above them, the service name /assign_task. Tag 1: an arrow from task_client to task_server, the request. Tag 2: an arrow from task_server back to task_client, the reply.
   :width: 90%

   1: the request, from the client node ``task_client`` to the server
   node ``task_server``, on the service ``/assign_task``. 2: the reply,
   back to that client.

The ROS 2 documentation, "Services": "In ROS 2, a service refers to a
remote procedure call." and "In ROS 2, services are expected to return
quickly, as the client is generally waiting on the result." Lecture 8's
table gave a service the competition example "Assign a task". Also:
"There should only ever be one service server per service name."

The Server
----------

.. code: ros/lecture10/competition_services/competition_services/task_server.py all

.. code-block:: python
   :caption: competition_services/competition_services/task_server.py

   """A robot that accepts or refuses tasks, as a service (Lecture 10)."""

   from competition_interfaces.srv import AssignTask
   import rclpy
   from rclpy.executors import ExternalShutdownException
   from rclpy.node import Node

   LOW_BATTERY_PCT = 10  # Rule 3 of Lecture 6, chosen for that example
   DRAIN_PCT_PER_MIN = 2  # Rule 4 of Lecture 6, chosen for that example


   class TaskServer(Node):
       """Answers each AssignTask request with Lecture 6's rules 3 and 4."""

       def __init__(self):
           super().__init__('task_server')
           self.declare_parameter('battery_pct', 30)  # Percent, chosen
           self._battery = self.get_parameter('battery_pct').value
           self._service = self.create_service(
               AssignTask, 'assign_task', self._assign_task)

       def _assign_task(self, request, response):
           cost = DRAIN_PCT_PER_MIN * request.minutes
           if self._battery < LOW_BATTERY_PCT:  # Rule 3
               response.message = 'needs_recharge'
           elif cost > self._battery:
               response.message = f'needs {cost}%, has {self._battery}%'
           else:  # Rule 4: the task uses 2% per minute
               self._battery -= cost
               response.accepted = True
               response.message = 'done'
           response.battery_pct = self._battery
           self.get_logger().info(
               f'{request.task} ({request.minutes} min): {response.message}')
           return response


   def main(args=None):
       rclpy.init(args=args)
       node = TaskServer()
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

- ``create_service(type, name, callback)``: the server answers every
  request on ``assign_task`` with ``self._assign_task``.
- rclpy passes the ``request`` and an empty ``response``, both of
  ``AssignTask``; the callback fills the response and returns it.
- The battery starts at the parameter ``battery_pct``, 30% by default.
  Rule 3 of Lecture 6: below 10%, no tasks. Rule 4: a task uses 2% per
  minute, so a task needs ``2 * minutes`` percent.

Calling the Service from the Command Line
-----------------------------------------

Terminal 1, ``ros2 run competition_services task_server``; terminal 2:

.. code-block:: bash

   ros2 service list -t | grep -v /task_server/
   ros2 service type /assign_task
   ros2 service call /assign_task competition_interfaces/srv/AssignTask \
       "{task: carry crate, minutes: 20}"

.. output: s_list.txt all

.. code-block:: text

   /assign_task [competition_interfaces/srv/AssignTask]

.. output: s_type.txt all

.. code-block:: text

   competition_interfaces/srv/AssignTask

.. output: s_call.txt 1-5

.. code-block:: text

   waiting for service to become available...
   requester: making request: competition_interfaces.srv.AssignTask_Request(task='carry crate', minutes=20)

   response:
   competition_interfaces.srv.AssignTask_Response(accepted=False, battery_pct=30, message='needs 40%, has 30%')





The request is YAML, as for ``ros2 topic pub``; a field not given keeps
its default. 20 minutes need 2 × 20 = 40%, and the robot has 30%: the
task is refused, and the battery stays at 30%. The ``grep -v`` leaves out
the node's own services (its parameter services):

.. output: s_server_services.txt all

.. code-block:: text

   /task_server/describe_parameters [rcl_interfaces/srv/DescribeParameters]
   /task_server/get_parameter_types [rcl_interfaces/srv/GetParameterTypes]
   /task_server/get_parameters [rcl_interfaces/srv/GetParameters]
   /task_server/get_type_description [type_description_interfaces/srv/GetTypeDescription]
   /task_server/list_parameters [rcl_interfaces/srv/ListParameters]
   /task_server/set_parameters [rcl_interfaces/srv/SetParameters]
   /task_server/set_parameters_atomically [rcl_interfaces/srv/SetParametersAtomically]

A Client That Calls Once
------------------------

.. code: ros/lecture10/competition_services/competition_services/task_client.py all

.. code-block:: python
   :caption: competition_services/competition_services/task_client.py

   """Send one AssignTask request and print the reply (Lecture 10)."""

   # Run: ros2 run competition_services task_client "pick widget" 5

   import sys

   from competition_interfaces.srv import AssignTask
   import rclpy
   from rclpy.executors import ExternalShutdownException
   from rclpy.utilities import remove_ros_args


   def main(args=None):
       rclpy.init(args=args)
       task, minutes = remove_ros_args(sys.argv)[1:3]  # Lecture 9
       node = rclpy.create_node('task_client')
       client = node.create_client(AssignTask, 'assign_task')
       try:
           while not client.wait_for_service(timeout_sec=1.0):
               node.get_logger().info('Waiting for assign_task')
           request = AssignTask.Request(task=task, minutes=int(minutes))
           future = client.call_async(request)  # Sends it, does not wait
           rclpy.spin_until_future_complete(node, future)  # Waits here
           reply = future.result()
           node.get_logger().info(
               f'accepted: {reply.accepted}, battery: {reply.battery_pct}%, '
               f'{reply.message}')
       except (KeyboardInterrupt, ExternalShutdownException):
           pass
       except Exception:  # Ctrl+C can also end spin() with an RCLError
           if rclpy.ok():  # ROS 2 still running: a real error
               raise
       finally:
           node.destroy_node()
           rclpy.try_shutdown()

1. ``create_client(type, name)``. ``wait_for_service(timeout_sec=1.0)``
   returns ``False`` after 1 s if no server is up yet, or if the client
   has not found the running server yet, so the loop waits for it.
2. ``call_async()`` sends the request and returns a **future**: an
   object that holds the reply once it arrives.
3. ``rclpy.spin_until_future_complete()`` spins the node until the
   future is done: the reply is taken in by the node's executor.
4. ``remove_ros_args()`` (Lecture 9) gives the program's own arguments,
   the task and the minutes.

Three runs, in terminal 2, after the command-line call, while the same
server runs (one recording):

.. code-block:: bash

   ros2 run competition_services task_client "pick widget" 5
   ros2 run competition_services task_client "carry crate" 6
   ros2 run competition_services task_client "scan shelf" 3

.. output: s_client_runs.txt all

.. code-block:: text

   [INFO] [task_client]: accepted: True, battery: 20%, done
   [INFO] [task_client]: accepted: True, battery: 8%, done
   [INFO] [task_client]: accepted: False, battery: 8%, needs_recharge

A run can also begin with ``[INFO] [task_client]: Waiting for
assign_task``, even with the server up, when the client has not found it
within the first second: the independent check saw it in 2 of 33 runs.
Twenty more runs of the client in this recording, counted:

.. output: s_client_waiting.txt all

.. code-block:: text

   0 of 20 runs logged 'Waiting for assign_task' with the server up

The server's log (terminal 1), the command-line call included:

.. output: s_server.txt all

.. code-block:: text

   [INFO] [task_server]: carry crate (20 min): needs 40%, has 30%
   [INFO] [task_server]: pick widget (5 min): done
   [INFO] [task_server]: carry crate (6 min): done
   [INFO] [task_server]: scan shelf (3 min): needs_recharge

1. 5 minutes use 2 × 5 = 10%: 30 − 10 = 20%. 6 minutes use 2 × 6 = 12%:
   20 − 12 = 8%.
2. 8% is below 10%: rule 3 refuses the third task. The server keeps its
   battery between calls.

A Client Inside a Node
----------------------

``task_requester.py`` sends the three tasks from a timer callback, one
per second. Its three entry points differ in how they wait for the
reply:

.. code: ros/lecture10/competition_services/competition_services/task_requester.py all

.. code-block:: python
   :caption: competition_services/competition_services/task_requester.py

   """A node that sends a task from a timer callback (Lecture 10)."""

   from competition_executors.controller_runs import spin_until_ctrl_c
   from competition_interfaces.srv import AssignTask
   import rclpy
   from rclpy.callback_groups import MutuallyExclusiveCallbackGroup
   from rclpy.executors import MultiThreadedExecutor, SingleThreadedExecutor
   from rclpy.node import Node

   TASKS = [('pick widget', 5), ('carry crate', 6), ('scan shelf', 3)]
   PERIOD_S = 1.0  # One request per second, chosen for this example


   class TaskRequester(Node):
       """Sends the tasks of TASKS, one per timer callback."""

       def __init__(self, wait_in_callback=False, client_group=None):
           super().__init__('task_requester')
           self._wait = wait_in_callback  # True: call(); False: call_async()
           self._client = self.create_client(
               AssignTask, 'assign_task', callback_group=client_group)
           self._tasks = list(TASKS)
           self._timer = self.create_timer(PERIOD_S, self._send_next)

       def _send_next(self):
           if not self._tasks or not self._client.service_is_ready():
               return
           task, minutes = self._tasks.pop(0)
           request = AssignTask.Request(task=task, minutes=minutes)
           self.get_logger().info(f'Sending {task}')
           if self._wait:
               self._log_reply(self._client.call(request))  # Blocks here
           else:
               future = self._client.call_async(request)
               future.add_done_callback(
                   lambda done: self._log_reply(done.result()))

       def _log_reply(self, reply):
           self.get_logger().info(
               f'Reply: accepted {reply.accepted}, battery {reply.battery_pct}%')


   def async_requester(args=None):
       rclpy.init(args=args)
       spin_until_ctrl_c(TaskRequester(), SingleThreadedExecutor())


   def sync_requester(args=None):
       rclpy.init(args=args)
       node = TaskRequester(wait_in_callback=True)  # Timer, client: one group
       spin_until_ctrl_c(node, SingleThreadedExecutor())


   def sync_two_groups(args=None):
       rclpy.init(args=args)
       node = TaskRequester(wait_in_callback=True,
                            client_group=MutuallyExclusiveCallbackGroup())
       spin_until_ctrl_c(node, MultiThreadedExecutor(num_threads=2))

Each entry point spins with ``spin_until_ctrl_c()``, Lecture 9's helper
in ``competition_executors``: ``add_node()``, ``spin()``, and Lecture 8's
Ctrl+C handling. ``async_requester``: ``call_async()``, then
``add_done_callback()``. The
timer callback returns at once, and the executor runs the lambda when
the reply has come. Each run below was
stopped after 5.5 s (chosen) with Ctrl+C (``timeout -s INT``), which is why each
ends with exit status 124, ``timeout``'s code for a command it stopped.

.. code-block:: bash

   ros2 run competition_services async_requester

.. output: s_async_requester.txt all

.. code-block:: text

   [INFO] [task_requester]: Sending pick widget
   [INFO] [task_requester]: Reply: accepted True, battery 20%
   [INFO] [task_requester]: Sending carry crate
   [INFO] [task_requester]: Reply: accepted True, battery 8%
   [INFO] [task_requester]: Sending scan shelf
   [INFO] [task_requester]: Reply: accepted False, battery 8%
   exit status: 124

call() Inside a Callback Never Returns
--------------------------------------

``sync_requester`` uses ``call()``, which waits for the reply, in the
timer callback, with a ``SingleThreadedExecutor``:

.. code-block:: bash

   ros2 run competition_services sync_requester

.. output: s_sync_requester.txt all

.. code-block:: text

   [INFO] [task_requester]: Sending pick widget
   exit status: 124

The server's log during that run:

.. output: s_sync_requester_server.txt all

.. code-block:: text

   [INFO] [task_server]: pick widget (5 min): done

The server answered, and ``call()`` never returned: no reply was logged,
and no second task was sent. The ROS 2 how-to "Synchronous vs.
asynchronous service clients": "When deadlock occurs, you will not
receive any indication that the service is blocked." rclpy's docstring
of ``Client.call()``: "Do not call this method in a callback, or a
deadlock or timeout may occur."

.. figure:: /_static/images/l10/deadlock.svg
   :alt: Two time lanes, time going right along an arrow at the bottom. The top lane, task_requester, holds one long bar that starts at tag 1 and runs off the right edge: the timer callback, waiting in call(). The bottom lane, task_server, holds a short bar, tag 2, with an arrow to it from the start of the long bar and an arrow from it back up to a small dashed box, tag 3, under the long bar: the reply, waiting.
   :width: 90%

   1: the executor's one thread runs the timer callback, which calls
   ``call()`` and waits. 2: the server gets the request and replies. 3:
   the reply waits for the executor to take it in, on that same thread,
   which never comes free. Time goes right: a time axis, not a map.

The how-to: "When a client is blocking a thread waiting for a response,
but the response can only be returned on that same thread, the client
will never stop waiting, and nothing else can happen."

Two Groups and Two Threads Let call() Return
--------------------------------------------

``sync_two_groups`` puts the client in its own mutually exclusive group
and spins a ``MultiThreadedExecutor`` with two threads (Lecture 9):

.. code-block:: bash

   ros2 run competition_services sync_two_groups

.. output: s_sync_two_groups.txt all

.. code-block:: text

   [INFO] [task_requester]: Sending pick widget
   [INFO] [task_requester]: Reply: accepted True, battery 20%
   [INFO] [task_requester]: Sending carry crate
   [INFO] [task_requester]: Reply: accepted True, battery 8%
   [INFO] [task_requester]: Sending scan shelf
   [INFO] [task_requester]: Reply: accepted False, battery 8%
   exit status: 124

The second thread takes the reply in while the first one waits in
``call()``. The ROS 2 how-to "Using Callback Groups": if you make a
synchronous call in a callback, "this callback and the client making the
call need to belong to" "different callback groups (of any type), or a
Reentrant Callback Group." The sync how-to: "It is very easy to
mistakenly cause deadlock when calling services synchronously, so we do
not recommend using call()." This course uses ``call_async()``.


Actions
====================================================

A Goal, Feedback, and a Result
------------------------------

The ROS 2 documentation, "Actions": "In ROS 2, an action refers to a
long-running remote procedure call with feedback and the ability to
cancel or preempt the goal." Lecture 8's table gave an action the
competition example "Drive to a goal". Driving 5 m at 2.0 m/s (Lecture
7's scout) takes 5 / 2.0 = 2.5 s. A service client would wait that long
with nothing to show, and could not stop the drive; an action sends
feedback while it works and can be canceled.

The client sends a **goal** (5 m). The server sends **feedback** while it
works (the meters left), then one **result** (the meters driven). The
client may cancel the goal before the end. As for services: "There
should only ever be one action server per action name."

An Action Is Three Services and Two Topics
------------------------------------------

While ``drive_server`` runs, the hidden services and topics under
``/drive_to_goal``:

.. code-block:: bash

   ros2 service list --include-hidden-services | grep '^/drive_to_goal'
   ros2 topic list --include-hidden-topics -t | grep '^/drive_to_goal'

.. output: a_hidden_services.txt all

.. code-block:: text

   /drive_to_goal/_action/cancel_goal
   /drive_to_goal/_action/get_result
   /drive_to_goal/_action/send_goal

.. output: a_hidden_topics.txt all

.. code-block:: text

   /drive_to_goal/_action/feedback [competition_interfaces/action/DriveToGoal_FeedbackMessage]
   /drive_to_goal/_action/status [action_msgs/msg/GoalStatusArray]

.. figure:: /_static/images/l10/action_parts.svg
   :alt: The node drive_client on the left and the node drive_server on the right. Between them, the names under /drive_to_goal. Tag 1: three services, _action/send_goal, _action/cancel_goal and _action/get_result, each with an arrow from drive_client to it and from it to drive_server. Tag 2: two topics, _action/feedback and _action/status, each with an arrow from drive_server to it and from it to drive_client.
   :width: 90%

   1: three services, from the client to the server: send a goal, cancel
   it, get its result. 2: two topics, from the server to the client: the
   feedback, and the status of every goal.

``ros2 action list -t`` and ``ros2 action info -t`` show the action
itself:

.. output: a_list.txt all

.. code-block:: text

   /drive_to_goal [competition_interfaces/action/DriveToGoal]

.. output: a_info.txt all

.. code-block:: text

   Action: /drive_to_goal
   Action clients: 0
   Action servers: 1
       /drive_server [competition_interfaces/action/DriveToGoal]

The Action Server
-----------------

.. code: ros/lecture10/competition_actions/competition_actions/drive_server.py all

.. code-block:: python
   :caption: competition_actions/competition_actions/drive_server.py

   """Drive a distance as an action, with feedback and cancel (Lecture 10)."""

   import time

   from competition_executors.controller_runs import spin_until_ctrl_c
   from competition_interfaces.action import DriveToGoal
   import rclpy
   from rclpy.action import ActionServer, CancelResponse, GoalResponse
   from rclpy.executors import MultiThreadedExecutor, SingleThreadedExecutor
   from rclpy.node import Node

   STEP_S = 0.5  # Seconds between two feedback messages (chosen)


   class DriveServer(Node):
       """Drives at max_speed_mps until the goal's distance is covered."""

       def __init__(self):
           super().__init__('drive_server')
           self.declare_parameter('max_speed_mps', 2.0)  # Lecture 7
           self._server = ActionServer(
               self, DriveToGoal, 'drive_to_goal',
               execute_callback=self._drive,
               goal_callback=self._check_goal,
               cancel_callback=self._check_cancel)

       def _check_goal(self, goal):
           if goal.distance_m <= 0.0:
               self.get_logger().info(f'Goal refused: {goal.distance_m} m')
               return GoalResponse.REJECT
           self.get_logger().info(f'Goal accepted: {goal.distance_m} m')
           return GoalResponse.ACCEPT

       def _check_cancel(self, goal_handle):
           self.get_logger().info('Cancel accepted')
           return CancelResponse.ACCEPT

       def _drive(self, goal_handle):
           speed = self.get_parameter('max_speed_mps').value
           distance = goal_handle.request.distance_m
           driven = 0.0
           while driven < distance:
               if goal_handle.is_cancel_requested:
                   goal_handle.canceled()
                   self.get_logger().info(f'Canceled after {driven} m')
                   return DriveToGoal.Result(distance_driven_m=driven)
               time.sleep(STEP_S)  # Stands for driving for STEP_S seconds
               driven = min(distance, driven + speed * STEP_S)
               feedback = DriveToGoal.Feedback()
               feedback.distance_remaining_m = distance - driven
               goal_handle.publish_feedback(feedback)
           goal_handle.succeed()
           self.get_logger().info(f'Arrived after {driven} m')
           return DriveToGoal.Result(distance_driven_m=driven)


   def main(args=None):
       rclpy.init(args=args)
       executor = MultiThreadedExecutor(num_threads=4)
       spin_until_ctrl_c(DriveServer(), executor)


   def single(args=None):
       rclpy.init(args=args)
       spin_until_ctrl_c(DriveServer(), SingleThreadedExecutor())

- ``ActionServer(node, type, name, ...)`` with three callbacks:
  ``_check_goal`` accepts or refuses a goal, ``_check_cancel`` answers a
  cancel request, and ``_drive`` does the work.
- A goal of 0 m or less is refused before it starts. Without a cancel
  callback, rclpy refuses every cancel: its default returns
  ``CancelResponse.REJECT`` (``rclpy/action/server.py``; checked by the
  quiz's Question 9).
- A **goal handle** is the object that stands for one goal: the cancel
  callback and the execute callback get it. ``goal_handle.request`` is
  the goal. Each step checks for a cancel,
  drives (``time.sleep()`` stands for driving), and publishes feedback.
  One step of 0.5 s at the speed parameter, 2.0 m/s by default, drives
  2.0 × 0.5 = 1.0 m.
- At the end, ``succeed()`` or ``canceled()`` sets the goal's final
  status, and the returned ``Result`` goes to the client. The speed is
  read at the start of each goal, so a ``ros2 param set`` counts for the
  next goal (Exercise 2).

Sending a Goal from the Command Line
------------------------------------

Terminal 1, ``ros2 run competition_actions drive_server``; terminal 2:

.. code-block:: bash

   ros2 action send_goal --feedback /drive_to_goal \
       competition_interfaces/action/DriveToGoal "{distance_m: 2.0}"

.. output: a_cli.txt all

.. code-block:: text

   Waiting for an action server to become available...
   Sending goal:
        distance_m: 2.0

   Goal accepted with ID: ...

   Feedback:
       distance_remaining_m: 1.0

   Feedback:
       distance_remaining_m: 0.0

   Result:
       distance_driven_m: 2.0

   Goal finished with status: SUCCEEDED

2.0 m at 1.0 m per step: two steps, two feedback messages. The goal ID
is a random number for each goal, replaced by ``...``.

The Action Client
-----------------

.. code: ros/lecture10/competition_actions/competition_actions/drive_client.py all

.. code-block:: python
   :caption: competition_actions/competition_actions/drive_client.py

   """Send a DriveToGoal goal, log the feedback, maybe cancel (Lecture 10)."""

   from action_msgs.msg import GoalStatus
   from competition_interfaces.action import DriveToGoal
   import rclpy
   from rclpy.action import ActionClient
   from rclpy.executors import ExternalShutdownException
   from rclpy.node import Node

   STATUS = {GoalStatus.STATUS_SUCCEEDED: 'SUCCEEDED',
             GoalStatus.STATUS_CANCELED: 'CANCELED',
             GoalStatus.STATUS_ABORTED: 'ABORTED'}


   class DriveClient(Node):
       """Sends one goal; cancels it after cancel_after feedback messages."""

       def __init__(self):
           super().__init__('drive_client')
           self.declare_parameter('distance_m', 5.0)  # Chosen for this example
           self.declare_parameter('cancel_after', 0)  # 0: never cancel
           self._cancel_after = self.get_parameter('cancel_after').value
           self._feedback_count = 0
           self._goal_handle = None
           self._client = ActionClient(self, DriveToGoal, 'drive_to_goal')
           self._client.wait_for_server()
           goal = DriveToGoal.Goal(
               distance_m=self.get_parameter('distance_m').value)
           future = self._client.send_goal_async(
               goal, feedback_callback=self._on_feedback)
           future.add_done_callback(self._on_goal_response)

       def _on_goal_response(self, future):
           self._goal_handle = future.result()
           if not self._goal_handle.accepted:
               self.get_logger().info('Goal refused')
               rclpy.shutdown()  # Ends spin(): nothing more will come
               return
           self.get_logger().info('Goal accepted')
           result_future = self._goal_handle.get_result_async()
           result_future.add_done_callback(self._on_result)

       def _on_feedback(self, feedback_msg):
           remaining = feedback_msg.feedback.distance_remaining_m
           self.get_logger().info(f'Feedback: {remaining} m to go')
           self._feedback_count += 1
           if self._feedback_count == self._cancel_after:
               self.get_logger().info('Canceling the goal')
               self._goal_handle.cancel_goal_async()

       def _on_result(self, future):
           reply = future.result()  # The status and the result
           self.get_logger().info(
               f'Result: {STATUS[reply.status]}, '
               f'driven {reply.result.distance_driven_m} m')
           rclpy.shutdown()  # Ends spin(): the goal is over


   def main(args=None):
       rclpy.init(args=args)
       node = DriveClient()
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

1. Two parameters: the distance (5.0 m) and when to cancel (after that
   many feedback messages; 0 means never).
2. ``send_goal_async()`` returns a future, like ``call_async()``; each
   feedback message goes to ``_on_feedback``.
3. The first future gives the client's goal handle: did the server accept the
   goal? If so, ``get_result_async()`` asks for the result, a second
   future.
4. The result comes with the goal's final status. ``rclpy.shutdown()``
   ends ``spin()``, as in the ROS 2 tutorial "Writing an action server
   and client (Python)"; ``main()`` is Lecture 8's.

A goal to the end, a goal canceled after 2 feedback messages, and a
refused goal, while ``drive_server`` runs:

.. code-block:: bash

   ros2 run competition_actions drive_client
   ros2 run competition_actions drive_client --ros-args -p cancel_after:=2
   ros2 run competition_actions drive_client --ros-args -p distance_m:=-1.0

.. output: a_client.txt all

.. code-block:: text

   [INFO] [drive_client]: Goal accepted
   [INFO] [drive_client]: Feedback: 4.0 m to go
   [INFO] [drive_client]: Feedback: 3.0 m to go
   [INFO] [drive_client]: Feedback: 2.0 m to go
   [INFO] [drive_client]: Feedback: 1.0 m to go
   [INFO] [drive_client]: Feedback: 0.0 m to go
   [INFO] [drive_client]: Result: SUCCEEDED, driven 5.0 m

.. output: a_cancel.txt all

.. code-block:: text

   [INFO] [drive_client]: Goal accepted
   [INFO] [drive_client]: Feedback: 4.0 m to go
   [INFO] [drive_client]: Feedback: 3.0 m to go
   [INFO] [drive_client]: Canceling the goal
   [INFO] [drive_client]: Feedback: 2.0 m to go
   [INFO] [drive_client]: Result: CANCELED, driven 3.0 m

.. output: a_refused.txt all

.. code-block:: text

   [INFO] [drive_client]: Goal refused

The server's log for the four goals (the three clients, then the
command line):

.. output: a_server.txt all

.. code-block:: text

   [INFO] [drive_server]: Goal accepted: 5.0 m
   [INFO] [drive_server]: Arrived after 5.0 m
   [INFO] [drive_server]: Goal accepted: 5.0 m
   [INFO] [drive_server]: Cancel accepted
   [INFO] [drive_server]: Canceled after 3.0 m
   [INFO] [drive_server]: Goal refused: -1.0 m
   [INFO] [drive_server]: Goal accepted: 2.0 m
   [INFO] [drive_server]: Arrived after 2.0 m

- 5 m: five steps of 1.0 m, then ``SUCCEEDED``.
- The cancel came during step 3, which finished (2.0 m to go) before the
  loop checked again: ``CANCELED`` after 3.0 m.
- The refused goal never reached ``_drive()``: ``_check_goal()`` returned
  ``REJECT``, and the client stopped at the goal response.

Why the Server Spins a Multi-Threaded Executor
----------------------------------------------

``drive_server_single`` spins a ``SingleThreadedExecutor``;
``drive_server`` a ``MultiThreadedExecutor`` with 4 threads (chosen). With each, a
goal canceled after 2 feedback messages, and a second goal of 1.0 m sent
as soon as the first goal was accepted (``-r __node:=second_client``,
Lecture 9). The clients' lines,
with ``drive_server_single``:

.. output: a_drive_server_single.txt all

.. code-block:: text

   [INFO] [drive_client]: Goal accepted
   [INFO] [drive_client]: Feedback: 4.0 m to go
   [INFO] [drive_client]: Feedback: 3.0 m to go
   [INFO] [drive_client]: Canceling the goal
   [INFO] [drive_client]: Feedback: 2.0 m to go
   [INFO] [drive_client]: Feedback: 1.0 m to go
   [INFO] [drive_client]: Feedback: 0.0 m to go
   [INFO] [drive_client]: Result: SUCCEEDED, driven 5.0 m
   [INFO] [second_client]: Goal accepted
   [INFO] [second_client]: Feedback: 0.0 m to go
   [INFO] [second_client]: Result: SUCCEEDED, driven 1.0 m

and its log:

.. output: a_drive_server_single_server.txt all

.. code-block:: text

   [INFO] [drive_server]: Goal accepted: 5.0 m
   [INFO] [drive_server]: Arrived after 5.0 m
   [INFO] [drive_server]: Goal accepted: 1.0 m
   [INFO] [drive_server]: Arrived after 1.0 m

With ``drive_server``:

.. output: a_drive_server.txt all

.. code-block:: text

   [INFO] [drive_client]: Goal accepted
   [INFO] [drive_client]: Feedback: 4.0 m to go
   [INFO] [drive_client]: Feedback: 3.0 m to go
   [INFO] [drive_client]: Canceling the goal
   [INFO] [drive_client]: Feedback: 2.0 m to go
   [INFO] [drive_client]: Result: CANCELED, driven 3.0 m
   [INFO] [second_client]: Goal accepted
   [INFO] [second_client]: Feedback: 0.0 m to go
   [INFO] [second_client]: Result: SUCCEEDED, driven 1.0 m

.. output: a_drive_server_server.txt all

.. code-block:: text

   [INFO] [drive_server]: Goal accepted: 5.0 m
   [INFO] [drive_server]: Cancel accepted
   [INFO] [drive_server]: Goal accepted: 1.0 m
   [INFO] [drive_server]: Canceled after 3.0 m
   [INFO] [drive_server]: Arrived after 1.0 m

Five runs of each server: the results the two clients logged, and when
the server accepted the second goal (during the first goal, or after it
ended):

.. output: a_runs_summary.txt all

.. code-block:: text

   server               first goal       second goal      when    runs
   drive_server_single  SUCCEEDED 5.0 m  SUCCEEDED 1.0 m  after   5 of 5
   drive_server         CANCELED 3.0 m   SUCCEEDED 1.0 m  during  5 of 5

- With one thread, ``_drive()`` holds it for the whole goal. The log has
  no ``Cancel accepted``: the first client got ``SUCCEEDED`` after 5.0 m,
  and the second goal was accepted only after the first one arrived.
- With four threads, another thread accepted the cancel and the second
  goal while ``_drive()`` ran. The two goals ran at the same time; this
  server does not stop that.
- No callback group was needed: rclpy runs the execute callback as a
  task of the executor (``self._node.executor.create_task(...)`` in
  ``rclpy/action/server.py``), not as a callback of the node's default
  group.


Quality of Service
====================================================

Settings for How Messages Are Delivered
---------------------------------------

The ROS 2 documentation, "Quality of Service settings": "ROS 2 offers a
rich variety of Quality of Service (QoS) policies that allow you to tune
communication between nodes." A **policy** is one setting; a
**profile** is a set of policies, given to a publisher or a
subscription: "A set of QoS “policies” combine to form a QoS
“profile”."

Lecture 8 met QoS twice: the ``10`` of
``create_publisher(BatteryState, 'battery', 10)``, a queue size, and the
rule "Nodes will only establish connections with other nodes if they
have compatible Quality of Service settings."

Four Policies of a Profile
--------------------------

.. list-table::
   :header-rows: 1
   :widths: 15 20 65
   :class: compact-table

   * - Policy
     - Setting
     - What the docs say
   * - History
     - Keep last
     - "only store up to N samples, configurable via the queue depth
       option"
   * - Depth
     - Queue size
     - "only honored if the “history” policy was set to “keep last”"
   * - Reliability
     - Best effort
     - "attempt to deliver samples, but may lose them if the network is
       not robust"
   * -
     - Reliable
     - "guarantee that samples are delivered, may retry multiple times"
   * - Durability
     - Volatile
     - "no attempt is made to persist samples"
   * -
     - Transient local
     - "the publisher becomes responsible for persisting samples for
       “late-joining” subscriptions"

A **sample** is one message. History also has "Keep all"; the page
lists more policies (deadline, lifespan, liveliness), which this course
does not use.

The Default Profile, and One for Sensors
----------------------------------------

The docs: "By default, publishers and subscriptions in ROS 2 have “keep
last” for history with a queue size of 10, “reliable” for reliability,
“volatile” for durability, and “system default” for liveliness." Two
profiles of ``rclpy.qos``, printed (history, depth, reliability,
durability):

.. output: q_profiles.txt all

.. code-block:: text

   QoSProfile(depth=10)     KEEP_LAST 10 RELIABLE VOLATILE
   qos_profile_sensor_data  KEEP_LAST 5 BEST_EFFORT VOLATILE

Passing ``10`` to ``create_publisher()`` gives the default profile with
depth 10. The sensor data profile: the docs, "the sensor data profile
uses best effort reliability and a smaller queue size."

Lecture 8's two battery nodes, seen by
``ros2 topic info -v /battery`` (the lines that ``grep`` kept):

.. output: q_default_info.txt all

.. code-block:: text

   Publisher count: 1
   Node name: battery_publisher
   Endpoint type: PUBLISHER
     Reliability: RELIABLE
     History (Depth): UNKNOWN
     Durability: VOLATILE
   Subscription count: 1
   Node name: battery_monitor
   Endpoint type: SUBSCRIPTION
     Reliability: RELIABLE
     History (Depth): UNKNOWN
     Durability: VOLATILE

``History (Depth): UNKNOWN``: ``ros2 topic info -v`` does not show a
depth, on any endpoint measured in this lecture.

A Profile Chosen by Parameters
------------------------------

``competition_qos`` has a publisher and a subscriber whose profile three
parameters choose:

.. code: ros/lecture10/competition_qos/competition_qos/qos_choice.py all

.. code-block:: python
   :caption: competition_qos/competition_qos/qos_choice.py

   """Build a QoS profile from three parameters of a node (Lecture 10)."""

   from rclpy.qos import DurabilityPolicy, QoSProfile, ReliabilityPolicy

   RELIABILITY = {'reliable': ReliabilityPolicy.RELIABLE,
                  'best_effort': ReliabilityPolicy.BEST_EFFORT}
   DURABILITY = {'volatile': DurabilityPolicy.VOLATILE,
                 'transient_local': DurabilityPolicy.TRANSIENT_LOCAL}


   def declare_qos(node):
       """Declare reliability, durability and depth; return the profile."""
       node.declare_parameter('reliability', 'reliable')  # The defaults
       node.declare_parameter('durability', 'volatile')
       node.declare_parameter('depth', 10)
       depth = node.get_parameter('depth').value
       reliability = node.get_parameter('reliability').value
       durability = node.get_parameter('durability').value
       return QoSProfile(depth=depth,  # History: keep last, the default
                         reliability=RELIABILITY[reliability],
                         durability=DURABILITY[durability])

.. code: ros/lecture10/competition_qos/competition_qos/status_publisher.py all

.. code-block:: python
   :caption: competition_qos/competition_qos/status_publisher.py

   """Publish a robot's status with a QoS chosen by parameters (Lecture 10)."""

   from competition_interfaces.msg import RobotStatus
   from competition_qos.qos_choice import declare_qos
   import rclpy
   from rclpy.executors import ExternalShutdownException
   from rclpy.node import Node

   LOW_BATTERY_PCT = 10  # Rule 3 of Lecture 6, chosen for that example
   DRAIN_PCT = 2  # Rule 4 of Lecture 6: one message stands for one minute


   class StatusPublisher(Node):
       """Publishes `count` messages, one every 0.5 s, then keeps running."""

       def __init__(self):
           super().__init__('status_publisher')
           qos = declare_qos(self)
           self.declare_parameter('count', 5)  # Chosen for this example
           self._left = self.get_parameter('count').value
           self._battery = 30  # Percent, chosen for this example
           self._publisher = self.create_publisher(RobotStatus, 'status', qos)
           self._timer = self.create_timer(0.5, self._publish_status)  # Chosen

       def _publish_status(self):
           if self._left == 0:
               return
           msg = RobotStatus(name='scout', battery_pct=self._battery)
           if self._battery < LOW_BATTERY_PCT:
               msg.status = RobotStatus.NEEDS_RECHARGE
           else:
               msg.status = RobotStatus.ACTIVE
           self._publisher.publish(msg)
           self.get_logger().info(f'Published {self._battery}%')
           self._battery -= DRAIN_PCT
           self._left -= 1


   def main(args=None):
       rclpy.init(args=args)
       node = StatusPublisher()
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

.. code: ros/lecture10/competition_qos/competition_qos/status_monitor.py all

.. code-block:: python
   :caption: competition_qos/competition_qos/status_monitor.py

   """Subscribe to a robot's status with the QoS its parameters choose."""

   from competition_interfaces.msg import RobotStatus
   from competition_qos.qos_choice import declare_qos
   import rclpy
   from rclpy.executors import ExternalShutdownException
   from rclpy.node import Node

   NAMES = {RobotStatus.ACTIVE: 'active',
            RobotStatus.NEEDS_RECHARGE: 'needs_recharge'}


   class StatusMonitor(Node):
       """Logs each status it receives."""

       def __init__(self):
           super().__init__('status_monitor')
           qos = declare_qos(self)
           self._subscription = self.create_subscription(
               RobotStatus, 'status', self._on_status, qos)

       def _on_status(self, msg):
           self.get_logger().info(
               f'{msg.name}: {msg.battery_pct}%, {NAMES[msg.status]}')


   def main(args=None):
       rclpy.init(args=args)
       node = StatusMonitor()
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

``status_publisher`` publishes ``RobotStatus`` messages, 5 by default,
from 30% down by 2% (rule 4; one message stands for one minute), one
every 0.5 s (chosen), then keeps running.
``status_monitor`` logs each one. ``-p reliability:=best_effort`` and
``-p durability:=transient_local`` change a node's profile.

A Best-Effort Publisher, a Reliable Monitor
-------------------------------------------

.. code-block:: bash

   ros2 run competition_qos status_monitor                  # Terminal 1, first
   ros2 run competition_qos status_publisher --ros-args \
       -p reliability:=best_effort                          # Terminal 2

Terminal 1, all of it:

.. output: q_mismatch_monitor.txt all

.. code-block:: text

   [WARN] [status_monitor]: New publisher discovered on topic 'status', offering incompatible QoS. No messages will be received from it. Last incompatible policy: RELIABILITY

Terminal 2:

.. output: q_mismatch_publisher.txt all

.. code-block:: text

   [WARN] [status_publisher]: New subscription discovered on topic 'status', requesting incompatible QoS. No messages will be sent to it. Last incompatible policy: RELIABILITY
   [INFO] [status_publisher]: Published 30%
   [INFO] [status_publisher]: Published 28%
   [INFO] [status_publisher]: Published 26%
   [INFO] [status_publisher]: Published 24%
   [INFO] [status_publisher]: Published 22%

The monitor asks for reliable delivery; the publisher only offers best
effort. They do not connect: 5 messages published, none received. Both
nodes warn and name the policy.

Requested Versus Offered
------------------------

The docs: "Subscriptions request a QoS profile that is the “minimum
quality” that it is willing to accept, and publishers offer a QoS
profile that is the “maximum quality” that it is able to provide.
Connections are only made if every policy of the requested QoS profile
is not more stringent than that of the offered QoS profile." More
stringent means asking for more. The docs' tables, for these settings:

.. list-table::
   :header-rows: 1
   :widths: 25 25 15 35
   :class: compact-table

   * - Publisher offers
     - Subscription asks
     - Compatible
     - The docs' result
   * - Best effort
     - Best effort
     - Yes
     -
   * - Best effort
     - Reliable
     - No
     -
   * - Reliable
     - Best effort
     - Yes
     -
   * - Reliable
     - Reliable
     - Yes
     -
   * - Volatile
     - Volatile
     - Yes
     - "New messages only"
   * - Volatile
     - Transient local
     - No
     - "No communication"
   * - Transient local
     - Volatile
     - Yes
     - "New messages only"
   * - Transient local
     - Transient local
     - Yes
     - "New and old messages"

And: "In order for a connection to be made, all of the policies that
affect compatibility must be compatible."

Seven Pairs, Measured
---------------------

For each pair, the monitor started first, then a publisher of 5
messages (``code/make_outputs.sh`` of the slides). The count of messages
the monitor logged, and the policy its warning named:

.. output: q_matrix.txt all

.. code-block:: text

   pub_rel     pub_dur         sub_rel     sub_dur         got  warning
   best_effort volatile        best_effort volatile        5/5  
   best_effort volatile        reliable    volatile        0/5  RELIABILITY
   reliable    volatile        best_effort volatile        5/5  
   reliable    volatile        reliable    volatile        5/5  
   reliable    volatile        reliable    transient_local 0/5  DURABILITY
   reliable    transient_local reliable    volatile        5/5  
   reliable    transient_local reliable    transient_local 5/5  

- Two pairs did not connect, and the warning named the policy: each
  monitor asks for more than its publisher offers.
- Asking for less is fine: a best-effort monitor got all 5 messages of a
  reliable publisher.
- The same seven results in three rounds:

.. output: q_matrix_rounds.txt all

.. code-block:: text

   round 2: same as round 1
   round 3: same as round 1

A row was run again when the monitor got only some of the 5 messages
(the first ones were sent before the two nodes had found each other), or
none without a warning. So "alike in three rounds" holds for the rows
that passed this filter: a compatible pair can miss messages, even
all 5, when the publisher sends before discovery ends. In the recording
shown here, two compatible rows got 4 of 5 on their first try and 5 of 5
on the second; in an earlier recording, two compatible rows got 0 of 5
with no warning on a first try. Every retry of the recording shown here:

.. output: q_retries.txt all

.. code-block:: text

   retry: best_effort volatile best_effort volatile got 4/5, 0 warnings, on try 1; the monitor's log:
       [INFO] [status_monitor]: scout: 28%, active
       [INFO] [status_monitor]: scout: 26%, active
       [INFO] [status_monitor]: scout: 24%, active
       [INFO] [status_monitor]: scout: 22%, active
   retry: reliable volatile best_effort volatile got 4/5, 0 warnings, on try 1; the monitor's log:
       [INFO] [status_monitor]: scout: 28%, active
       [INFO] [status_monitor]: scout: 26%, active
       [INFO] [status_monitor]: scout: 24%, active
       [INFO] [status_monitor]: scout: 22%, active

A Subscriber That Starts Late
-----------------------------

The publisher's 5 messages (30% to 22%) are sent from 0.5 s to 2.5 s
after it starts; the monitor starts at 4 s and runs for 6 s (both
chosen). What the late monitor got,
for five pairs of durability and depth:

.. output: q_late.txt all

.. code-block:: text

   pub_dur         pub_depth sub_dur         sub_depth late monitor got
   volatile        10        volatile        10        
   transient_local 10        volatile        10        
   transient_local 10        transient_local 10        30% 28% 26% 24% 22%
   transient_local 3         transient_local 10        26% 24% 22%
   transient_local 10        transient_local 2         24% 22%

1. Rows 1 and 2: a volatile side, so nothing old reaches the late
   monitor.
2. Row 3: transient local on both sides, so all 5 old messages. The
   docs: "To achieve a “latched” topic that is visible to late
   subscribers, both the publisher and subscriber must agree to use
   ‘Transient Local’."
3. Row 4: the publisher kept only its last 3 messages, its depth. Row 5:
   a monitor of depth 2 got the last 2.

The same in three rounds:

.. output: q_late_rounds.txt all

.. code-block:: text

   round 2: same as round 1
   round 3: same as round 1

The late monitor of row 3, its own lines:

.. output: q_late_monitor.txt all

.. code-block:: text

   [INFO] [status_monitor]: scout: 30%, active
   [INFO] [status_monitor]: scout: 28%, active
   [INFO] [status_monitor]: scout: 26%, active
   [INFO] [status_monitor]: scout: 24%, active
   [INFO] [status_monitor]: scout: 22%, active

Finding a Mismatch with ros2 topic info -v
------------------------------------------

During the run of `A Best-Effort Publisher, a Reliable Monitor`_,
``ros2 topic info -v /status`` (the lines that ``grep`` kept):

.. output: q_mismatch_info.txt all

.. code-block:: text

   Node name: status_publisher
     Reliability: BEST_EFFORT
     History (Depth): UNKNOWN
     Durability: VOLATILE
   Node name: status_monitor
     Reliability: RELIABLE
     History (Depth): UNKNOWN
     Durability: VOLATILE

A subscriber that gets nothing, on a topic with a publisher: compare the
two profiles. Here, ``BEST_EFFORT`` offered and ``RELIABLE`` asked.

``ros2 topic echo`` chooses its profile to fit the publishers it finds.
``ros2 topic echo -h``, its ``--qos-durability`` option:

.. output: q_echo_help.txt all

.. code-block:: text

     --qos-durability {system_default,transient_local,volatile,unknown,best_available}
                           Quality of service durability setting to subscribe
                           with (overrides durability value of --qos-profile
                           option, default: Compatible profile with running
                           endpoints )

So an echo that receives messages does not prove that your node can.

Why Lecture 12 Needs Transient Local
------------------------------------

Nav2's map saver waits for a map on ``/map``. While it waited (no map was
published here), ``ros2 topic info -v /map`` showed its subscription,
with its default parameters:

.. code-block:: bash

   ros2 run nav2_map_server map_saver_cli -f my_map

.. output: q_map_saver.txt all

.. code-block:: text

   Node name: map_saver
   Endpoint type: SUBSCRIPTION
     Reliability: RELIABLE
     Durability: TRANSIENT_LOCAL

A map is published once, or seldom, and the saver often starts after
that. With transient local on both sides, the saver still gets the last
map (row 3 of `A Subscriber That Starts Late`_). A volatile map
publisher would not connect at all: the durability warning of `Seven
Pairs, Measured`_. Lecture 12 saves a map with this tool. Transient
local is the default of the saver's parameter
``map_subscribe_transient_local``; with
``--ros-args -p map_subscribe_transient_local:=false``, it subscribed
volatile:

.. output: q_map_saver_volatile.txt all

.. code-block:: text

   Node name: map_saver
     Reliability: RELIABLE
     Durability: VOLATILE

With no map, the saver gave up after its wait:

.. output: q_map_saver_log.txt all

.. code-block:: text

   [INFO] [map_saver]: Saving map from 'map' topic to 'my_map' file
   [ERROR] [map_saver]: Failed to spin map subscription

Lecture Summary
---------------

- A node accepts only the parameters it declares. Type and range are
  checked at start and on every set; a read-only parameter can be set at
  start and is refused on every later set. A post set callback tells the
  node about a change.
- An interface package (ament_cmake) turns ``.msg``, ``.srv``, and
  ``.action`` files into classes. Set ``ROS_PYTHON_CHECK_FIELDS=1`` to
  have fields checked.
- A service client in a node uses ``call_async()``: ``call()`` in a
  callback deadlocked.
- An action sends feedback and can be canceled; its server needs a free
  thread to take the cancel.
- A subscription connects only if it asks for no more than the publisher
  offers; transient local on both sides keeps the last messages for late
  subscribers.
