====================================================
L6: Object-Oriented Programming, Part I
====================================================

Overview
--------

This lecture introduces object-oriented programming, with a robotics
competition as the running example. It starts with what a class does
that a dictionary and functions cannot, then takes the competition
through the design phase: requirements, business rules, nouns and verbs,
a UML class box, and one module per class. The implementation phase
writes the ``Robot`` class: the ``class`` statement, ``__init__`` and
``self``, methods, and instance and class attributes, with where Python
looks for an attribute. Dunder methods give objects their text, ``==``
and ``<``, arithmetic, and ``len()``, ``in``, and ``for``. The lecture
ends with abstraction and encapsulation: raising and handling
exceptions, non-public names, and properties that check every change,
as in the fields of a ROS 2 message. **Quiz 2** takes the first 20
minutes of this class.


.. admonition:: Learning Objectives
   :class: learning-objectives

   By the end of this lecture, you will be able to:

   - Explain what object-oriented programming is, and when a class helps
     more than a dictionary and functions.
   - Go from a problem description to a design: requirements, business
     rules, nouns and verbs, and a UML class box.
   - Write a class with ``__init__`` and methods; make objects; explain
     what ``self`` refers to.
   - Tell instance attributes from class attributes, and say where
     Python looks for an attribute.
   - Write dunder methods for printing, comparing, arithmetic, and
     ``len()``, ``in``, and ``for``.
   - Raise an exception for a bad value, and handle one with ``try`` and
     ``except``.
   - Use abstraction and encapsulation: non-public names, and properties
     that check every change.


.. toctree::
   :hidden:
   :maxdepth: 2
   :titlesonly:

   l6_lecture
   l6_exercises
   l6_quiz
   l6_references

Next Steps
----------

- There is no class on **Mar 16** (Spring Break).
- In the next lecture (**Mar 23**), we will cover
  :doc:`L7: Object-Oriented Programming, Part II </lectures/lecture7/l7_index>`.
- **Assignment 2** is due on **Mar 23**
  (:doc:`Assignments </assignments/index>`). AI tools are not permitted
  for it.
- **Pre-work:**

  - Run every program in ``lecture6/``, change its values, and predict
    each output before you run it.
  - Complete the Lecture 6 :doc:`Python exercises <l6_exercises>`,
    Exercise 4 included, and the :doc:`self-check quiz <l6_quiz>`.
  - If you skipped them, read the
    :doc:`Memory reading </reading_material/memory/mem_index>`'s "Names
    and Objects", "Names Live in Namespaces", and "Call Frames", which
    this lecture used.
