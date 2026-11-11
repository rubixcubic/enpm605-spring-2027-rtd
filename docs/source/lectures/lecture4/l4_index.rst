====================================================
L4: Functions
====================================================

Overview
--------

This lecture covers functions, from writing and calling them to passing
them around as values. It starts with ``def``, ``return``, and what a
call does with a frame, then covers every kind of argument, where Python
looks for a name, and what a call can do to the caller's objects. It
adds type hints checked with mypy, and recursion over a tree of
coordinate frames. It ends with functions as values: key functions,
lambda functions, closures, callbacks (the pattern ROS 2 uses for
subscriptions and timers), and ``functools.partial``. The examples come
from robots: lidar scans, drive times, battery levels, waypoints, and
topics.


.. admonition:: Learning Objectives
   :class: learning-objectives

   By the end of this lecture, you will be able to:

   - Define and call functions, return one value or several, and write
     docstrings.
   - Pass positional, keyword, and default arguments, and collect extra
     ones with ``*args`` and ``**kwargs``.
   - Explain where Python looks for a name (LEGB), and when ``global``
     and ``nonlocal`` are needed.
   - Predict what a call does to the caller's objects
     (pass-by-assignment).
   - Add type hints, and check them with a type checker.
   - Write recursive functions with a base case.
   - Use functions as values: key functions, lambda functions, closures,
     callbacks, and ``functools.partial``.


.. toctree::
   :hidden:
   :maxdepth: 2
   :titlesonly:

   l4_lecture
   l4_exercises
   l4_quiz
   l4_references

Next Steps
----------

- In the next lecture (**Mar 2**), we will cover
  :doc:`L5: NumPy and Matplotlib for Robotics </lectures/lecture5/l5_index>`.
- **Assignment 1** is due on **Mar 2**
  (:doc:`Assignments </assignments/index>`). AI tools are not permitted
  for it.
- **Pre-work:**

  - Run every program in ``lecture4/``, change its values, and predict
    each output before you run it.
  - Complete the Lecture 4 :doc:`Python exercises <l4_exercises>` and
    :doc:`self-check quiz <l4_quiz>`.
  - Read the :doc:`Memory reading </reading_material/memory/mem_index>`,
    "Measuring Memory": a list of floats against a NumPy array.
    Lecture 5 builds on it.
