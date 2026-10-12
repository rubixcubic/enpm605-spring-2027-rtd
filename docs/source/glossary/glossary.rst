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

   Module
      One ``.py`` file. Another file can ``import`` it and use the names it defines. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Package
      A folder of modules. A regular package holds an ``__init__.py`` file, which Python runs when the package is imported. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Module search path
      The list of folders, ``sys.path``, that Python searches in order for each ``import``. The folder of the program you run comes first. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   PYTHONPATH
      An environment variable whose folders Python adds to ``sys.path``. Sourcing ROS 2 sets it, which makes ``rclpy`` importable. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Main guard
      The block ``if __name__ == "__main__":``. Its code runs when the file is run, and not when the file is imported. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Falsy
      Describes a value that counts as false in a condition: ``None``, ``False``, zero of any numeric type, and empty sequences and collections. Every other value is truthy. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Short-circuit evaluation
      ``and`` and ``or`` evaluate their right side only when the left side does not already decide the result. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Floor division
      The operator ``//``. It divides and rounds toward negative infinity, so ``-7 // 2`` is ``-4``. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   f-string
      A string literal that starts with ``f``, in which each ``{expression}`` is replaced by its value, for example ``f"{speed:.2f} m/s"``. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Slice
      A copy of part of a sequence, written ``s[start:stop:step]``. It holds the items from ``start`` up to, but not including, ``stop``. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Capture pattern
      A bare name in a ``case`` of a ``match`` statement. It always matches and binds the value to that name. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

