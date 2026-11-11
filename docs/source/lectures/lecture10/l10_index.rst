=========================================================
L10: Parameters, Custom Interfaces, Services, and Actions
=========================================================

Overview
--------

Lecture 9 set a parameter's value when a node starts, on a demo node
that keeps any parameter. This lecture starts with your own nodes:
declaring a parameter with a type, a range, and a description, then
setting it at start, from a file, and while the node runs. It then
defines the competition's own message, service, and action in an
interface package, and uses them: a service server that assigns tasks
with Lecture 6's battery rules, and an action server that drives a
distance, sends feedback, and accepts a cancel. A service client inside a
node shows why ``call()`` deadlocks in a callback, and an action server
shows why it needs a multi-threaded executor. The last part is Quality of
Service (QoS): which publisher and subscriber settings connect, measured
on seven pairs, and what a subscriber that starts late receives, which
Lecture 12's map saver needs.


.. admonition:: Learning Objectives
   :class: learning-objectives

   By the end of this lecture, you will be able to:

   - Declare a node's parameters with a type, a range, and a
     description; set them at start, from a file, and while the node
     runs.
   - Define a message, a service, and an action in an interface package,
     and use the classes colcon generates.
   - Write a service server and a client that does not block its own
     executor.
   - Write an action server that sends feedback and accepts a cancel,
     and a client that sends a goal and reads the result.
   - Choose a QoS profile (reliability, durability, depth), and predict
     whether a publisher and a subscriber connect.


.. toctree::
   :hidden:
   :maxdepth: 2
   :titlesonly:

   l10_lecture
   l10_exercises
   l10_quiz
   l10_references

Next Steps
----------

- In the next lecture (**Apr 20**), we will cover
  :doc:`L11: Coordinate Frames, TF2, and Mobile Robot Control </lectures/lecture11/l11_index>`.
- **Team Project 1** is due today (**Apr 13**), and **Team Project 2** is
  posted today, due **Apr 27** (:doc:`Assignments </assignments/index>`).
- **Quiz 4** is at the start of the next class (**Apr 20**).
- **Pre-work:**

  - Install the simulation that Lecture 11 uses (Gazebo Harmonic's ROS
    packages and the TurtleBot3 simulation):
    ``sudo apt install ros-jazzy-ros-gz ros-jazzy-nav2-minimal-tb3-sim``.
  - ``git pull`` in the
    `ROS repository <https://github.com/rubixcubic/enpm605-spring-2027-ros>`_,
    build it, and run every executable of ``lecture10/``.
  - Complete the Lecture 10 :doc:`Python exercises <l10_exercises>`,
    Exercise 4 included, and the :doc:`self-check quiz <l10_quiz>`.
  - Read the :doc:`Memory reading </reading_material/memory/mem_index>`'s
    "Objects Live on the Heap" again: a server's battery is one
    attribute of one node object, seen by every request and changed by
    each accepted task.
