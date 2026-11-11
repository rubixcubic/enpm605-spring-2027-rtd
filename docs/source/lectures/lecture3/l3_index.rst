====================================================
L3: Python Fundamentals, Part II
====================================================

Overview
--------

This lecture covers loops and the built-in collections you use to hold
robot data. It starts with ``for`` and ``while`` loops, shows what a
``for`` loop does with an iterator, and then covers lists, tuples,
dictionaries, and sets: how to build them, change them, copy them, and
choose between them. The examples come from robots: lidar ranges,
waypoints, poses, robot settings, and detected objects.


.. admonition:: Learning Objectives
   :class: learning-objectives

   By the end of this lecture, you will be able to:

   - Write ``for`` and ``while`` loops with ``range()``, ``enumerate()``,
     ``break``, ``continue``, and ``else``.
   - Explain what an iterable and an iterator are, and what a ``for``
     loop does with them.
   - Create, index, change, sort, and copy lists, and tell an in-place
     method from one that returns a new object.
   - Build lists, dictionaries, and sets with comprehensions.
   - Use tuples for fixed records, and unpack them.
   - Store and look up data in dictionaries, and explain why a key must
     be hashable.
   - Use sets to remove duplicates, test membership, and compare two
     collections.


.. toctree::
   :hidden:
   :maxdepth: 2
   :titlesonly:

   l3_lecture
   l3_exercises
   l3_quiz
   l3_references

Next Steps
----------

- In the next lecture (**Feb 23**), we will cover
  :doc:`L4: Functions </lectures/lecture4/l4_index>`.
- **Quiz 1** takes the first 20 minutes of the next class (Feb 23):
  closed notes, no slides, and no software to test code.
- **Assignment 1** is posted today (Feb 16) and is due on **Mar 2**
  (:doc:`Assignments </assignments/index>`). AI tools are not permitted
  for it.
- **Pre-work:**

  - Run every program in ``lecture3/``, change its values, and predict
    each output before you run it.
  - Complete the Lecture 3 :doc:`Python exercises <l3_exercises>` and
    :doc:`self-check quiz <l3_quiz>`.
  - Read the :doc:`Memory reading </reading_material/memory/mem_index>`,
    "Names Live in Namespaces" and "Call Frames". Lecture 4 builds on
    both.
