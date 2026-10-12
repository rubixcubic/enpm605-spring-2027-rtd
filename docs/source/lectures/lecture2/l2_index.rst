====================================================
L2: Python Fundamentals, Part I
====================================================

Overview
--------

This lecture covers the building blocks of a Python program: how code is
split into modules and packages and how Python finds them, indentation,
the ``bool`` type, operators, numbers, strings, and decisions with
``if`` and ``match``. The examples come from robots: range sensors,
battery levels, headings, and ROS 2 topic names.


.. admonition:: Learning Objectives
   :class: learning-objectives

   By the end of this lecture, you will be able to:

   - Split code into modules and packages, import them, and explain
     where Python looks for them.
   - Explain how indentation marks a block, and read an indentation
     error.
   - Predict whether a value counts as true or false in a condition.
   - Use arithmetic, comparison, logical, and membership operators,
     including ``//`` and ``%`` on negative numbers.
   - Explain why floats are approximate, and compare them with a
     tolerance.
   - Build, format, index, and slice strings.
   - Write decisions with ``if``, ``elif``, ``else``, and ``match``.


.. toctree::
   :hidden:
   :maxdepth: 2
   :titlesonly:

   l2_lecture
   l2_exercises
   l2_quiz
   l2_references

Next Steps
----------

- In the next lecture (**Feb 16**), we will cover
  :doc:`L3: Python Fundamentals, Part II </lectures/lecture3/l3_index>`.
- **Pre-work:**

  - Run every program in ``lecture2/``, change its values, and predict
    each output before you run it.
  - Complete the Lecture 2 :doc:`Python exercises <l2_exercises>` and
    :doc:`self-check quiz <l2_quiz>`.
  - Read the :doc:`Memory reading </reading_material/memory/mem_index>`,
    at least "Containers Hold References" and "Shallow and Deep Copies".
    Lecture 3 uses both for lists.
