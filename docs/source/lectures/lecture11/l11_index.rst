=====================================================
L11: Coordinate Frames, TF2, and Mobile Robot Control
=====================================================

Overview
--------

Lecture 10's action server only pretended to drive. This lecture drives
a simulated TurtleBot3 Waffle in Gazebo Harmonic, to a goal in front of
the competition's task site. It starts with a robot's pose: a position
and a quaternion. It then starts the simulation and RViz2, drives the
robot with ``/cmd_vel``, and checks the differential drive model against
the wheel speeds Gazebo reports. Odometry adds up the robot's motion and
drifts, measured against Gazebo's own pose of the robot. A proportional
controller drives the robot to the goal, with three gains compared. The
last two parts are coordinate frames (REP 105 and the robot's frame tree)
and TF2: a static broadcaster that makes the goal a frame, a listener
that finds the goal from the robot, and the same controller with TF2
doing the geometry.


.. admonition:: Learning Objectives
   :class: learning-objectives

   By the end of this lecture, you will be able to:

   - Describe a robot's pose with a position and a quaternion, and
     convert a yaw to a quaternion and back.
   - Start a TurtleBot3 in Gazebo Harmonic, drive it with ``/cmd_vel``,
     and show its data in RViz2.
   - Compute wheel speeds from a linear and an angular velocity with the
     differential drive model.
   - Read odometry, and say why it drifts.
   - Write a proportional controller that drives the robot to a goal.
   - Broadcast a static transform, look up a transform with a listener,
     and read the frame tree.


.. toctree::
   :hidden:
   :maxdepth: 2
   :titlesonly:

   l11_lecture
   l11_exercises
   l11_quiz
   l11_references

Next Steps
----------

- In the next lecture (**Apr 27**), we will cover
  :doc:`L12: Mapping and Navigation </lectures/lecture12/l12_index>`.
- **Team Project 2** is due at the next class (**Apr 27**), and the
  **Final Project** is posted that day (:doc:`Assignments </assignments/index>`).
- **Pre-work:**

  - Install the simulation once:
    ``sudo apt install ros-jazzy-ros-gz ros-jazzy-nav2-minimal-tb3-sim``.
  - ``git pull`` in the
    `ROS repository <https://github.com/rubixcubic/enpm605-spring-2027-ros>`_,
    build it, start the arena, and run every executable of
    ``lecture11/``.
  - Complete the Lecture 11 :doc:`Python exercises <l11_exercises>`,
    Exercises 3 and 4 included, and the :doc:`self-check quiz <l11_quiz>`.
  - Read "Reference Counting" in the
    :doc:`Memory reading </reading_material/memory/mem_lecture>`:
    ``go_to_goal`` replaces its pose tuple about 28 times a second, and
    each old tuple is freed when its reference count drops to 0.
