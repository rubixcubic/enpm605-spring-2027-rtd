====================================================
References
====================================================

.. dropdown:: Lecture 7
    :class-container: sd-border-secondary
    :open:

    .. card::
        :class-card: sd-border-secondary

        **ENPM605 L7: Object-Oriented Programming, Part II**

        Covers decorators, from ``@property`` to decorators that take an
        argument, with ``functools.wraps``; class methods and static
        methods; data classes, ``__post_init__`` checks, and frozen data
        classes; ``__slots__`` and the memory it saves; composition,
        aggregation, and association in the robotics competition;
        inheritance with ``super()``, overriding, and the method
        resolution order; polymorphism and duck typing; abstract base
        classes; and protocols checked with mypy.


.. dropdown:: Course
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Course code repository
            :link: https://github.com/rubixcubic/enpm605-spring-2027-python
            :class-card: sd-border-secondary

            The Lecture 7 programs are in the ``lecture7/`` folder, and
            the competition's classes in
            ``lecture7/robotics_competition/``.

        .. grid-item-card:: Memory reading
            :link: /reading_material/memory/mem_index
            :link-type: doc
            :class-card: sd-border-secondary

            "Names and Objects" and "Names Live in Namespaces": a
            decorator rebinds a name to a new function object, and a
            class's namespace is searched along the method resolution
            order.


.. dropdown:: Decorators and Methods
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Python Glossary: decorator
            :link: https://docs.python.org/3.12/glossary.html#term-decorator
            :class-card: sd-border-secondary

            A function that returns another function, applied with the
            ``@`` syntax.

        .. grid-item-card:: Python Library: functools.wraps
            :link: https://docs.python.org/3.12/library/functools.html#functools.wraps
            :class-card: sd-border-secondary

            Copies the name, the docstring, and other attributes of the
            wrapped function to the wrapper.

        .. grid-item-card:: PEP 318: Decorators for Functions and Methods
            :link: https://peps.python.org/pep-0318/
            :class-card: sd-border-secondary

            The proposal that added the ``@`` syntax.

        .. grid-item-card:: Python Library: classmethod and staticmethod
            :link: https://docs.python.org/3.12/library/functions.html#classmethod
            :class-card: sd-border-secondary

            What each kind of method receives as its first argument.


.. dropdown:: Data Classes and Slots
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Python Library: dataclasses
            :link: https://docs.python.org/3.12/library/dataclasses.html
            :class-card: sd-border-secondary

            Fields, ``field()``, ``__post_init__``, ``frozen``, and
            ``slots``.

        .. grid-item-card:: PEP 557: Data Classes
            :link: https://peps.python.org/pep-0557/
            :class-card: sd-border-secondary

            Why data classes were added, and what they write.

        .. grid-item-card:: Python Reference: __slots__
            :link: https://docs.python.org/3.12/reference/datamodel.html#slots
            :class-card: sd-border-secondary

            What ``__slots__`` changes, with notes on inheritance.

        .. grid-item-card:: Python Library: tracemalloc
            :link: https://docs.python.org/3.12/library/tracemalloc.html
            :class-card: sd-border-secondary

            Measures the memory Python allocates, as in
            ``slots_memory.py``.

        .. grid-item-card:: Ruff: RUF008 (mutable-dataclass-default)
            :link: https://docs.astral.sh/ruff/rules/mutable-dataclass-default/
            :class-card: sd-border-secondary

            Reports a list or dictionary used as a data class default.


.. dropdown:: Inheritance, ABCs, and Protocols
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Python Tutorial: Inheritance
            :link: https://docs.python.org/3.12/tutorial/classes.html#inheritance
            :class-card: sd-border-secondary

            Subclasses, overriding, ``isinstance()``, and
            ``issubclass()``.

        .. grid-item-card:: Python Library: super
            :link: https://docs.python.org/3.12/library/functions.html#super
            :class-card: sd-border-secondary

            How ``super()`` follows the method resolution order.

        .. grid-item-card:: Python Library: abc
            :link: https://docs.python.org/3.12/library/abc.html
            :class-card: sd-border-secondary

            ``ABC`` and ``@abstractmethod``.

        .. grid-item-card:: Python Library: typing.Protocol
            :link: https://docs.python.org/3.12/library/typing.html#typing.Protocol
            :class-card: sd-border-secondary

            Protocols and ``@runtime_checkable``.

        .. grid-item-card:: PEP 544: Protocols
            :link: https://peps.python.org/pep-0544/
            :class-card: sd-border-secondary

            Structural subtyping: a class fits by its names, not by its
            base classes.

        .. grid-item-card:: mypy: Protocols and structural subtyping
            :link: https://mypy.readthedocs.io/en/stable/protocols.html
            :class-card: sd-border-secondary

            How mypy checks a class against a protocol.

        .. grid-item-card:: Python Glossary: duck-typing
            :link: https://docs.python.org/3.12/glossary.html#term-duck-typing
            :class-card: sd-border-secondary

            Using an object by its methods, not by its type.


.. dropdown:: Design
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: OMG: the UML specification
            :link: https://www.omg.org/spec/UML/
            :class-card: sd-border-secondary

            The standard behind the class diagrams: diamonds,
            triangles, and multiplicities.

        .. grid-item-card:: Mermaid: class diagrams
            :link: https://mermaid.js.org/syntax/classDiagram.html
            :class-card: sd-border-secondary

            Class diagrams, with their relationships, written as text.
