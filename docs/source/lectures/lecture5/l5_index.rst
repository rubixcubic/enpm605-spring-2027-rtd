====================================================
L5: NumPy and Matplotlib for Robotics
====================================================

Overview
--------

This lecture covers NumPy arrays and Matplotlib plots, with robot data
as the running example. It starts with what an array does differently
from a list, measured for memory and speed, then covers shape, data
type, indexing, views and copies, and Boolean masks. It adds vectorized
operations and broadcasting, then vectors, rotation matrices, and 2D
transforms between a robot's frame and the world frame. A synthetic
laser scan with the fields of a ROS 2 ``LaserScan`` message brings these
together: its invalid readings (NaN and inf), the nearest obstacle, and
its points in the world frame. The lecture plots a trajectory and the
scan with Matplotlib, and ends with how NumPy and Matplotlib come into
the course environment with uv, next to ROS 2 Jazzy.


.. admonition:: Learning Objectives
   :class: learning-objectives

   By the end of this lecture, you will be able to:

   - Explain why robot code stores numbers in NumPy arrays instead of
     lists, and measure the difference.
   - Read an array's shape and data type; index, slice, and reshape it;
     tell a view from a copy.
   - Select readings with Boolean masks.
   - Compute with vectorized operations, reductions along an axis, and
     broadcasting.
   - Rotate 2D vectors, and transform points between a robot's frame and
     the world frame.
   - Process a laser scan: angles, invalid readings, the nearest
     obstacle, and points.
   - Plot a trajectory and a scan with Matplotlib, and save the figure.
   - Install NumPy and Matplotlib so that they work with ROS 2 Jazzy.


.. toctree::
   :hidden:
   :maxdepth: 2
   :titlesonly:

   l5_lecture
   l5_exercises
   l5_quiz
   l5_references

Next Steps
----------

- In the next lecture (**Mar 9**), we will cover
  :doc:`L6: Object-Oriented Programming, Part I </lectures/lecture6/l6_index>`.
- **Quiz 2** takes the first 20 minutes of the next class (**Mar 9**):
  closed notes, no slides, and no software to test code.
- **Assignment 2** is posted on **Mar 2** and due on **Mar 23**
  (:doc:`Assignments </assignments/index>`). AI tools are not permitted
  for it.
- **Pre-work:**

  - Run every program in ``lecture5/``, change its values, and predict
    each output before you run it.
  - Complete the Lecture 5 :doc:`Python exercises <l5_exercises>`,
    Exercise 4 included, and the :doc:`self-check quiz <l5_quiz>`.
  - In the course repository, run ``uv sync`` (it installs Matplotlib),
    then check: ``uv run lecture5/where_from.py`` must print NumPy 1.26.4
    and Matplotlib 3.11.2, both from ``.venv`` (Lecture 5, "Installing
    NumPy next to ROS 2").
  - If you skipped it, read the
    :doc:`Memory reading </reading_material/memory/mem_index>`,
    "Measuring Memory", which this lecture used.
