====================================================
Glossary
====================================================

This glossary collects the key terms from the ENPM605 lectures, in
alphabetical order. Terms are added as each lecture is posted, and each
entry links back to the lecture that defines it.

.. glossary::
   :sorted:

   Bytecode
      The low-level instructions CPython's compiler produces from a program and its interpreter loop executes. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   CPython
      The reference implementation of Python, written in C. The ``python3`` on Ubuntu is CPython. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Formatter
      A tool that rewrites the layout of code (spaces, line breaks) without changing what it does. Ruff includes one. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Immutable
      Describes an object that cannot change after it is created, such as an ``int``, a ``str``, or a ``tuple``. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Keyword
      A word with a special meaning in Python, such as ``class`` or ``if``, that cannot be a variable name. A soft keyword, such as ``match``, is a keyword only in some places. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Linter
      A tool that reads code and reports likely errors and style problems without running it. Ruff includes one. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Mutable
      Describes an object that can change in place, such as a ``list``, a ``dict``, or a ``set``. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Object identity
      The integer ``id()`` returns for an object. It stays the same while the object exists; in CPython it is the memory address. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   uv
      A tool that creates Python virtual environments and installs packages. The course uses version 0.13.0. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Virtual environment
      A folder (``.venv``) with its own Python packages, separate from the system's. The course environment also sees Ubuntu's packages. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Call frame
      The record of one function call in progress: its local namespace and where it returns. Discarded when the call returns. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Cycle collector
      The part of CPython (the ``gc`` module) that frees groups of objects that refer only to each other. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Deep copy
      A copy that also copies the objects inside, made with ``copy.deepcopy()``. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Heap
      The memory CPython manages for itself while the program runs. Every Python object lives there. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Immortal object
      In CPython 3.12, an object that is never freed, such as ``None``; its reference count stays at a fixed large value. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Namespace
      A table that maps names to objects: the module's global names, a call's local names, or the built-in names. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Reference
      How a name, a container slot, or another object points to an object. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Reference count
      The number of references to an object. CPython frees the object when it reaches zero. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Shallow copy
      A new container whose slots refer to the same objects as the original, made with ``copy.copy()``. (:doc:`Memory </reading_material/memory/mem_lecture>`)

