====================================================
L1: Course Introduction and Python Tooling
====================================================

Overview
--------

This lecture introduces the course and sets up the tools you use all
semester: VS Code, uv, and Ruff on Ubuntu 24.04. It shows how CPython
runs a program, from source code to bytecode, and ends with Python
variables: names, objects, identity, and mutability.


.. admonition:: Learning Objectives
   :class: learning-objectives

   By the end of this lecture, you will be able to:

   - Describe the course structure, grading, schedule, and policies.
   - Install the course toolchain: VS Code, uv, and Ruff on Ubuntu 24.04.
   - Create the course virtual environment and explain why it uses
     Ubuntu's Python 3.12.
   - Run and debug Python programs in VS Code.
   - Use Ruff to find and fix problems in Python code.
   - Explain how CPython runs a program: source, bytecode, virtual
     machine.
   - Create variables and explain names, objects, identity, and
     mutability.


.. toctree::
   :hidden:
   :maxdepth: 2
   :titlesonly:

   l1_lecture
   l1_exercises
   l1_quiz
   l1_references

Next Steps
----------

- In the next lecture (**Feb 9**), we will cover
  :doc:`L2: Python Fundamentals, Part I </lectures/lecture2/l2_index>`.
- **Pre-work:**

  - Finish the environment setup, and run
    ``uv run lecture1/verify_setup.py``.
  - Complete the Lecture 1 :doc:`Python exercises <l1_exercises>` and
    :doc:`self-check quiz <l1_quiz>`.
  - Read the `Python Tutorial <https://docs.python.org/3.12/tutorial/>`_,
    Sections 3, 4, and 5.
  - Read the :doc:`Memory reading </reading_material/memory/mem_index>`.
    The lectures refer to it throughout the course.
