====================================================
Memory
====================================================

Overview
--------

This reading explains where a running Python program keeps its data:
names in namespaces, objects on the heap, call frames, references and
reference counts, and when CPython frees an object. It also covers the
CPython details you will meet in practice (small-integer caching,
interned strings, immortal objects) and how to measure memory with
``sys.getsizeof`` and ``tracemalloc``.

The lectures refer to this reading throughout the course. Read it after
:doc:`Lecture 1 </lectures/lecture1/l1_index>`, and come back to it when
a lecture points here.


.. admonition:: Learning Objectives
   :class: learning-objectives

   By the end of this reading, you will be able to:

   - Tell a name from an object, and say which namespace a name is in.
   - Explain what an object's identity, type, value, and reference count
     are.
   - Describe the call frames that exist while a function runs.
   - Predict what a shallow copy and a deep copy share.
   - Explain when CPython frees an object: reference counting, and the
     cycle collector.
   - Explain why ``is`` must not be used to compare numbers or strings.
   - Measure an object's size and a data structure's total memory.


.. toctree::
   :hidden:
   :maxdepth: 2
   :titlesonly:

   mem_lecture
   mem_exercises
   mem_quiz
   mem_references
