======================================================
L9: Launch Files, Namespaces, Remapping, and Executors
======================================================

Overview
--------

Lecture 8 started every node with its own ``ros2 run``, in its own
terminal, under the name written in its code. This lecture changes that
in three steps. First, names: ROS arguments after ``--ros-args`` put a
node in a namespace, rename it, remap its topics, and set its
parameters, with no change to the code. Second, launch files: one
Python file, and one command, starts many nodes with their names and
settings, takes arguments, starts a node only if a condition holds,
groups nodes under one namespace, and includes other launch files.
Third, executors: what runs a node's callbacks, one thread or a pool of
threads, and how callback groups decide which callbacks may run at the
same time. Every claim about timing is measured on a robot controller
with a slow plan and a fast safety check, in five runs of each setup.


.. admonition:: Learning Objectives
   :class: learning-objectives

   By the end of this lecture, you will be able to:

   - Pass ROS arguments to a node on the command line, and say what
     ``rclpy.init()`` reads.
   - Put nodes in namespaces, and remap node and topic names without
     changing the code.
   - Set a parameter's value when a node starts, and find it under the
     node's full name.
   - Write Python launch files that start many nodes, with arguments,
     conditions, groups, and includes.
   - Explain how a single-threaded and a multi-threaded executor run
     callbacks, and measure the difference.
   - Choose a mutually exclusive or a reentrant callback group for each
     callback of a node.


.. toctree::
   :hidden:
   :maxdepth: 2
   :titlesonly:

   l9_lecture
   l9_exercises
   l9_quiz
   l9_references

Next Steps
----------

- In the next lecture (**Apr 13**), we will cover
  :doc:`L10: Parameters, Custom Interfaces, Services, and Actions </lectures/lecture10/l10_index>`,
  and Quality of Service (QoS): reliability, durability, history depth,
  and when a publisher's and a subscriber's settings are compatible.
- **Team Project 1** is due next week (**Apr 13**), and **Team Project
  2** is posted that day (:doc:`Assignments </assignments/index>`).
- **Pre-work:**

  - ``git pull`` in the
    `ROS repository <https://github.com/rubixcubic/enpm605-spring-2027-ros>`_,
    build it, and run ``team_launch.py`` and every executable of
    ``competition_executors``.
  - Complete the Lecture 9 :doc:`Python exercises <l9_exercises>`,
    Exercise 4 included, and the :doc:`self-check quiz <l9_quiz>`.
  - Read the :doc:`Memory reading </reading_material/memory/mem_index>`'s
    "Objects Live on the Heap" again: the threads of one process share
    its heap, so two callbacks running at the same time see the same
    node object.
