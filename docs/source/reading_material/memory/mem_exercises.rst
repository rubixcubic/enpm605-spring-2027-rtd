====================================================
Exercises
====================================================

Four exercises on the :doc:`Memory reading <mem_lecture>`. These
exercises are not graded and not submitted. Predict each answer first,
then run the code from the root of the course repository with
``uv run python <file>``.


----


.. dropdown:: Exercise 1: Rebinding Is Not Changing
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Tell assigning a new object from changing an object.

    **Tasks**

    Draw the names and objects after each line, then predict the output.

    .. code-block:: python

       a = [1]
       b = a
       b = [2]
       print(a, b, a is b)

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code-block:: text

           [1] [2] False

        ``b = a`` makes ``b`` refer to the list ``[1]``. ``b = [2]``
        creates a new list and makes ``b`` refer to it; the first list is
        unchanged, and ``a`` still refers to it. Compare
        ``b.append(2)``, which would change the one list both names
        refer to.


.. dropdown:: Exercise 2: Count the References
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Count every reference to an object, including the hidden one.

    **Tasks**

    Predict the number this prints, then run it.

    .. code-block:: python

       import sys

       scan = []
       log = [scan, scan]
       print(sys.getrefcount(scan))

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code-block:: text

           4

        Four references: the name ``scan`` (1), the two slots of ``log``
        (2 and 3), and the temporary reference that passing ``scan`` to
        ``getrefcount`` adds (4).


.. dropdown:: Exercise 3: Copying a Robot Configuration
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Predict what a shallow copy and a deep copy share.

    **Tasks**

    Predict both printed dictionaries, then run the code.

    .. code-block:: python

       import copy

       config = {"wheels": [0.1, 0.1], "name": "rover"}
       shallow = copy.copy(config)
       deep = copy.deepcopy(config)
       config["wheels"].append(0.2)
       config["name"] = "scout"
       print(shallow)
       print(deep)

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code-block:: text

           {'wheels': [0.1, 0.1, 0.2], 'name': 'rover'}
           {'wheels': [0.1, 0.1], 'name': 'rover'}

        - ``shallow`` is a new dictionary whose ``"wheels"`` entry refers
          to the same list as ``config``, so the appended ``0.2`` shows.
        - ``config["name"] = "scout"`` makes ``config``'s entry refer to a
          new string; ``shallow``'s entry still refers to ``"rover"``.
        - ``deep`` has its own list, so neither change shows.


.. dropdown:: Exercise 4: The Real Cost of a List
    :icon: gear
    :class-container: sd-border-primary
    :class-title: sd-font-weight-bold

    **Goal**

    Estimate a list's total memory, then measure it.

    **Tasks**

    1. Using the sizes in the reading (a float is 24 bytes), estimate the
       total memory of the list and its floats.
    2. Run the code and compare.

    .. code-block:: python

       import sys
       import tracemalloc

       tracemalloc.start()
       speeds = [float(i) for i in range(100_000)]
       current, _ = tracemalloc.get_traced_memory()
       tracemalloc.stop()
       print(sys.getsizeof(speeds), current)

    .. dropdown:: Solution
        :class-container: sd-border-success

        .. code-block:: text

           800984 3200832

        1. The list itself: 800,984 bytes (the first number).
        2. The floats: 100,000 × 24 = 2,400,000 bytes.
        3. Estimate: 800,984 + 2,400,000 = 3,200,984 bytes.
        4. ``tracemalloc`` counted 3,200,832 bytes: 152 bytes less than
           the estimate, a difference of under 0.005%. The estimate is a
           good model of where the memory goes.
