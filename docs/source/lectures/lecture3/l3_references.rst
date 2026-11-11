References
==========

.. dropdown:: Lecture 3
    :class-container: sd-border-secondary
    :open:

    .. card::
        :class-card: sd-border-secondary

        **ENPM605 L3: Python Fundamentals, Part II**

        Covers ``for`` and ``while`` loops, ``range()``,
        ``enumerate()``, ``break``, ``continue``, and ``else`` on a loop;
        iterables, iterators, ``iter()``, ``next()``, and ``zip()``;
        lists, in-place methods, comprehensions, and copies; tuples and
        unpacking; dictionaries, hashable keys, and views; sets and set
        operations; and how to choose a collection.


.. dropdown:: Course
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Course code repository
            :link: https://github.com/rubixcubic/enpm605-spring-2027-python
            :class-card: sd-border-secondary

            The Lecture 3 programs are in the ``lecture3/`` folder.

        .. grid-item-card:: Memory reading
            :link: /reading_material/memory/mem_index
            :link-type: doc
            :class-card: sd-border-secondary

            "Containers Hold References" and "Shallow and Deep Copies"
            draw what a list of lists holds and what a copy shares.


.. dropdown:: Loops and Iterables
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Python Tutorial: for statements
            :link: https://docs.python.org/3.12/tutorial/controlflow.html#for-statements
            :class-card: sd-border-secondary

            ``for``, ``range()``, ``break``, ``continue``, and ``else`` on
            a loop.

        .. grid-item-card:: The range type
            :link: https://docs.python.org/3.12/library/stdtypes.html#ranges
            :class-card: sd-border-secondary

            Why a range takes the same small amount of memory at any
            size.

        .. grid-item-card:: Glossary: iterable and iterator
            :link: https://docs.python.org/3.12/glossary.html#term-iterator
            :class-card: sd-border-secondary

            What ``iter()``, ``next()``, and ``StopIteration`` do.

        .. grid-item-card:: Built-in functions: zip()
            :link: https://docs.python.org/3.12/library/functions.html#zip
            :class-card: sd-border-secondary

            The shortest input, and ``strict=True``.

        .. grid-item-card:: REP 117: Informational Distance Measurements
            :link: https://www.ros.org/reps/rep-0117.html
            :class-card: sd-border-secondary

            How ROS range sensors report readings out of range.


.. dropdown:: Lists and Tuples
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Python Tutorial: More on Lists
            :link: https://docs.python.org/3.12/tutorial/datastructures.html#more-on-lists
            :class-card: sd-border-secondary

            List methods and list comprehensions.

        .. grid-item-card:: Mutable sequence types
            :link: https://docs.python.org/3.12/library/stdtypes.html#mutable-sequence-types
            :class-card: sd-border-secondary

            The operations that change a list in place.

        .. grid-item-card:: Common sequence operations
            :link: https://docs.python.org/3.12/library/stdtypes.html#common-sequence-operations
            :class-card: sd-border-secondary

            Why ``[[0] * 3] * 3`` repeats one inner list.

        .. grid-item-card:: copy: shallow and deep copy
            :link: https://docs.python.org/3.12/library/copy.html
            :class-card: sd-border-secondary

            ``copy.copy()`` and ``copy.deepcopy()``.

        .. grid-item-card:: Python Tutorial: Tuples and Sequences
            :link: https://docs.python.org/3.12/tutorial/datastructures.html#tuples-and-sequences
            :class-card: sd-border-secondary

            Packing, unpacking, and the one-item tuple.


.. dropdown:: Dictionaries and Sets
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Python Tutorial: Dictionaries
            :link: https://docs.python.org/3.12/tutorial/datastructures.html#dictionaries
            :class-card: sd-border-secondary

            Keys, values, and looping over a dictionary.

        .. grid-item-card:: Mapping types: dict
            :link: https://docs.python.org/3.12/library/stdtypes.html#mapping-types-dict
            :class-card: sd-border-secondary

            Every dictionary method, and the views.

        .. grid-item-card:: Glossary: hashable
            :link: https://docs.python.org/3.12/glossary.html#term-hashable
            :class-card: sd-border-secondary

            Which objects can be keys and set items.

        .. grid-item-card:: Python Tutorial: Sets
            :link: https://docs.python.org/3.12/tutorial/datastructures.html#sets
            :class-card: sd-border-secondary

            Set operations and set comprehensions.

        .. grid-item-card:: collections.Counter
            :link: https://docs.python.org/3.12/library/collections.html#collections.Counter
            :class-card: sd-border-secondary

            A dictionary built for counting.

        .. grid-item-card:: timeit
            :link: https://docs.python.org/3.12/library/timeit.html
            :class-card: sd-border-secondary

            Timing small pieces of code.
