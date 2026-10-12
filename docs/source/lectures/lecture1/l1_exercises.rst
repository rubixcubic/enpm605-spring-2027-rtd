====================================================
Python Exercises
====================================================

Four exercises on :doc:`Lecture 1 <l1_lecture>`. Run every program from
the root of the course repository, inside the course environment.

.. note::

   Try each exercise before you open its solution. Your ``id()`` values
   and paths will differ from the ones shown.


----


.. dropdown:: Exercise 1: Set Up and Verify the Environment
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Create the course environment in the right order and check it.

    **Tasks**

    1. Install uv 0.13.0 and VS Code with the Python and Ruff
       extensions (see :doc:`/setup/python_setup`).
    2. Clone the course repository. From its root folder, create the
       environment with ``uv venv --python /usr/bin/python3
       --system-site-packages``, then run ``uv sync``.
    3. Run ``uv run lecture1/verify_setup.py`` and read every line.
    4. Which of the five checks would fail if you had run ``uv sync``
       before ``uv venv``? Why?

    .. dropdown:: Solution
        :class-container: sd-border-success

        A working setup prints:

        .. code-block:: text

           [ OK ] Python 3.12 (need 3.12)
           [ OK ] virtual environment at .../enpm605-spring-2027-python/.venv
           [ OK ] system site packages visible
           [ OK ] NumPy 1.26.4 (need below 2)
           [ OK ] Ruff at .../enpm605-spring-2027-python/.venv/bin/ruff
           5 of 5 checks passed

        If ``uv sync`` runs first, uv creates ``.venv`` without
        ``--system-site-packages``, so check 3 fails: ``system site
        packages hidden``. The other four still pass, which is why the
        mistake is easy to miss until Lecture 8, when ROS 2 needs
        Ubuntu's packages.


.. dropdown:: Exercise 2: Fix linters_demo.py with Ruff
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Use Ruff to report and fix problems, and decide which fix is safe.

    **Tasks**

    1. Copy ``lecture1/linters_demo.py`` to ``my_linters.py``.
    2. Run ``uv run ruff check my_linters.py``. How many problems does
       Ruff report, and which ones are marked ``[*]``?
    3. Run ``uv run ruff check --fix my_linters.py``, then
       ``uv run ruff format my_linters.py``. Run ``ruff check`` again.
    4. Ruff's default rules do not report ``if x == True:``. Run
       ``uv run ruff check --extend-select E712 my_linters.py``. Ruff
       says the fix for ``E712`` is unsafe. Run the program below and
       explain why.

       .. code-block:: python

          x = 10
          print(x == True)
          print(bool(x))

    .. dropdown:: Solution
        :class-container: sd-border-success

        Step 2 reports four problems, all marked ``[*]``: one ``I001``
        (unsorted imports) and three ``F401`` (unused imports). After
        step 3 the file is:

        .. code-block:: python

           def calculate_area(radius):
               pi = 3.14159  # Magic number
               area = pi * radius**2  # No spaces
               return area


           x = 10
           if x == True:  # Compare to True
               print("yes")

        and ``ruff check`` prints ``All checks passed!``.

        Step 4 prints:

        .. code-block:: text

           False
           True

        ``x == True`` compares ``10`` with ``True`` (which equals ``1``),
        so it is ``False``. ``if x:`` tests whether ``x`` is truthy, and
        a non-zero number is truthy. Replacing one with the other changes
        what the program does: ``print("yes")`` would start running. A
        fix that can change behavior is unsafe, so Ruff does not apply
        it with ``--fix`` alone.


.. dropdown:: Exercise 3: Identity and Equality
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Predict ``is`` and ``==`` before running the code.

    **Tasks**

    Write down what each ``print`` shows, then run it.

    .. code-block:: python

       a = 7
       b = 7
       print(a is b)

       c = [1, 2]
       d = c
       print(c is d)

       e = c.copy()
       print(c is e, c == e)

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code-block:: text

           True
           True
           False True

        - ``a is b`` is ``True`` here because CPython keeps a single
          object for each small integer and reuses it. Do not rely on
          this: compare numbers with ``==``.
        - ``d = c`` makes a second name for the same list, so
          ``c is d`` is ``True``.
        - ``c.copy()`` makes a new list with the same values: ``c is e``
          is ``False``, and ``c == e`` is ``True``.


.. dropdown:: Exercise 4: Keywords and Soft Keywords
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Tell a keyword from a soft keyword.

    **Tasks**

    1. Run this program and explain the last line of output.

       .. code-block:: python

          import keyword

          print(len(keyword.kwlist))
          print(keyword.softkwlist)
          match = 5
          print(match)

    2. Put ``class = 5`` alone in a file named ``ex4b.py`` and run it with
       ``uv run python ex4b.py``. What happens, and why is it different
       from ``match = 5``?

    .. dropdown:: Solution
        :class-container: sd-border-success

        Task 1 prints:

        .. code-block:: text

           35
           ['_', 'case', 'match', 'type']
           5

        ``match`` is a soft keyword: it is a keyword only at the start of
        a ``match`` statement, so it can still be a variable name. Task 2
        fails before the program runs, because ``class`` is a keyword
        everywhere:

        .. code-block:: text

             File "ex4b.py", line 1
               class = 5
                     ^
           SyntaxError: invalid syntax
