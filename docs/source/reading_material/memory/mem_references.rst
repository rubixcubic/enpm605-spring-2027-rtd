References
==========

.. dropdown:: Memory reading
    :class-container: sd-border-secondary
    :open:

    .. card::
        :class-card: sd-border-secondary

        **ENPM605 Reading Material: Memory**

        Names and namespaces, objects on the heap, call frames,
        references and reference counts, copies, freeing objects,
        CPython details, and measuring memory. The code is in
        ``reading/memory/memory_demo.py`` in the course repository.


.. dropdown:: Python documentation
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Data model: objects, values and types
            :link: https://docs.python.org/3.12/reference/datamodel.html
            :class-card: sd-border-secondary

            What identity, type, and value mean.

        .. grid-item-card:: Execution model: naming and binding
            :link: https://docs.python.org/3.12/reference/executionmodel.html
            :class-card: sd-border-secondary

            Namespaces, scopes, and how names are bound.

        .. grid-item-card:: sys module
            :link: https://docs.python.org/3.12/library/sys.html
            :class-card: sd-border-secondary

            getsizeof, getrefcount, getrecursionlimit, intern.

        .. grid-item-card:: copy module
            :link: https://docs.python.org/3.12/library/copy.html
            :class-card: sd-border-secondary

            Shallow and deep copies.

        .. grid-item-card:: gc module
            :link: https://docs.python.org/3.12/library/gc.html
            :class-card: sd-border-secondary

            The cycle collector.

        .. grid-item-card:: weakref module
            :link: https://docs.python.org/3.12/library/weakref.html
            :class-card: sd-border-secondary

            weakref.finalize, used to see when an object is freed.

        .. grid-item-card:: tracemalloc module
            :link: https://docs.python.org/3.12/library/tracemalloc.html
            :class-card: sd-border-secondary

            Tracing memory allocations.

        .. grid-item-card:: Integer objects (C API)
            :link: https://docs.python.org/3.12/c-api/long.html
            :class-card: sd-border-secondary

            The small-integer cache, -5 to 256.


.. dropdown:: CPython changes
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: PEP 683: Immortal Objects
            :link: https://peps.python.org/pep-0683/
            :class-card: sd-border-secondary

            Why None, True, and small integers have a fixed reference count in 3.12.

        .. grid-item-card:: What's New in Python 3.11
            :link: https://docs.python.org/3.11/whatsnew/3.11.html
            :class-card: sd-border-secondary

            Cheaper, lazy Python frames.


.. dropdown:: Tools
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Python Tutor
            :link: https://pythontutor.com/
            :class-card: sd-border-secondary

            Draws names, frames, and objects for small programs, in the same style as this reading.

