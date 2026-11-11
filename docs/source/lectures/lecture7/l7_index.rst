====================================================
L7: Object-Oriented Programming, Part II
====================================================

Overview
--------

This lecture continues Lecture 6's robotics competition. It starts with
decorators, from the ``@property`` that Lecture 6 used, to a decorator
that takes an argument, with ``functools.wraps``. Class methods and
static methods, data classes, and ``__slots__`` write classes with less
code: the competition's ``Task`` and ``Sensor`` become frozen data
classes that check their fields. The second half has classes working
together: composition, aggregation, and association in a ``Team``;
inheritance with ``super()``, overriding, and the method resolution
order in ``Scout`` and ``Hauler``; polymorphism and duck typing;
abstract base classes; and protocols checked with mypy. A ROS 2 node, a
subclass of ``rclpy``'s ``Node``, links the lecture to Lecture 8.
**Assignment 2** is due at the start of this class.


.. admonition:: Learning Objectives
   :class: learning-objectives

   By the end of this lecture, you will be able to:

   - Explain what a decorator does; write one with ``functools.wraps``,
     and one that takes an argument.
   - Write class methods and static methods, and say what each one
     receives.
   - Write data classes, check their fields in ``__post_init__``, and
     freeze them; limit an object's attributes with ``__slots__``.
   - Tell association, aggregation, and composition apart, in code and
     in a UML diagram.
   - Write subclasses with ``super()``, replace methods and class
     attributes, and read the method resolution order.
   - Use polymorphism and duck typing, and require methods with an
     abstract base class.
   - Describe what a caller needs with a protocol, and check it with
     mypy.


.. toctree::
   :hidden:
   :maxdepth: 2
   :titlesonly:

   l7_lecture
   l7_exercises
   l7_quiz
   l7_references

Next Steps
----------

- In the next lecture (**Mar 30**), we will cover
  :doc:`L8: ROS 2 Foundations </lectures/lecture8/l8_index>`.
- **Quiz 3** takes the first 20 minutes of the next class (**Mar 30**):
  closed notes, no slides, and no software to test code.
- **Team Project 1** is posted on **Mar 30**
  (:doc:`Assignments </assignments/index>`). AI tools are permitted for
  it, as long as you document how you used them.
- **Pre-work:**

  - Run every program in ``lecture7/``, change its values, and predict
    each output before you run it.
  - Complete the Lecture 7 :doc:`Python exercises <l7_exercises>`,
    Exercise 4 included, and the :doc:`self-check quiz <l7_quiz>`.
  - Install ROS 2 Jazzy before the next class, as the
    :doc:`ROS 2 setup page </setup/ros_setup>` says: Lecture 8 runs ROS 2
    from its first section.
  - If you skipped them, read the
    :doc:`Memory reading </reading_material/memory/mem_index>`'s "Names
    and Objects" and "Names Live in Namespaces", which this lecture used.
