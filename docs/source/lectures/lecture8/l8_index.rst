====================================================
L8: ROS 2 Foundations
====================================================

Overview
--------

This lecture starts the ROS 2 half of the course. It says what ROS is
and why the course uses ROS 2 Jazzy, then splits a robot's software into
nodes: separate processes that send each other messages. You meet the
core parts of ROS 2 (nodes, topics, services, actions, and messages),
the layers under a Python node down to DDS, and the domain ID that keeps
your nodes apart from your classmates'. You run the talker and the
listener that come with ROS 2, inspect them with the ``ros2`` command,
build a workspace with ``colcon``, and write your own nodes in Python
with ``rclpy``: a battery publisher and a battery monitor for Lecture
6's robotics competition. **Quiz 3** takes the first 20 minutes of this
class, and **Team Project 1** is posted today.


.. admonition:: Learning Objectives
   :class: learning-objectives

   By the end of this lecture, you will be able to:

   - Say what ROS 2 is, which distribution this course uses, and why a
     robot's software is split into nodes.
   - Name the core parts of ROS 2: nodes, topics, services, actions, and
     messages, and the layers from ``rclpy`` down to DDS.
   - Keep your nodes apart from other people's with ``ROS_DOMAIN_ID``.
   - Explain the publish/subscribe model, run nodes, and inspect them
     with the ``ros2`` command.
   - Build a workspace with ``colcon``, and make a Python package with
     its ``package.xml``, ``setup.py``, and entry points.
   - Write nodes with ``rclpy``: a node class, a timer callback,
     spinning, a publisher, and a subscriber.


.. toctree::
   :hidden:
   :maxdepth: 2
   :titlesonly:

   l8_lecture
   l8_exercises
   l8_quiz
   l8_references

Next Steps
----------

- In the next lecture (**Apr 6**), we will cover
  :doc:`L9: Launch Files, Namespaces, Remapping, and Executors </lectures/lecture9/l9_index>`.
- **Team Project 1** is posted today (**Mar 30**) and due on **Apr 13**
  (:doc:`Assignments </assignments/index>`). AI tools are permitted for
  it, as long as you document how you used them.
- **Pre-work:**

  - Clone the
    `ROS repository <https://github.com/rubixcubic/enpm605-spring-2027-ros>`_
    into ``~/ros_ws/src``, build it, and run every node of
    ``competition_nodes`` with your own domain ID.
  - Complete the Lecture 8 :doc:`Python exercises <l8_exercises>`,
    Exercise 4 included, and the :doc:`self-check quiz <l8_quiz>`.
  - If you skipped it, read the
    :doc:`Memory reading </reading_material/memory/mem_index>`'s
    "Objects Live on the Heap": each process has its own heap, so two
    nodes in two processes share no objects.
