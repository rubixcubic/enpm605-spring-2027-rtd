====================================================
References
====================================================

.. dropdown:: Lecture 6
    :class-container: sd-border-secondary
    :open:

    .. card::
        :class-card: sd-border-secondary

        **ENPM605 L6: Object-Oriented Programming, Part I**

        Covers what object-oriented programming is and when a class
        helps; the design phase (requirements, business rules, nouns and
        verbs, a UML class box, one module per class) for a robotics
        competition; classes, objects, ``__init__``, and ``self``;
        instance and class attributes and where Python looks for an
        attribute; dunder methods for text, comparison, arithmetic, and
        containers; raising and handling exceptions; abstraction;
        encapsulation with non-public names and properties; and a ROS 2
        message as an object with properties.


.. dropdown:: Course
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Course code repository
            :link: https://github.com/rubixcubic/enpm605-spring-2027-python
            :class-card: sd-border-secondary

            The Lecture 6 programs are in the ``lecture6/`` folder, and
            the competition's ``Robot`` class in
            ``lecture6/robotics_competition/``.

        .. grid-item-card:: Memory reading
            :link: /reading_material/memory/mem_index
            :link-type: doc
            :class-card: sd-border-secondary

            "Names and Objects", "Names Live in Namespaces", and "Call
            Frames": an object's attributes are a namespace, and ``self``
            is a name in a call's namespace.


.. dropdown:: Python Classes
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Python Tutorial: Classes
            :link: https://docs.python.org/3.12/tutorial/classes.html
            :class-card: sd-border-secondary

            Class objects, instance objects, method objects, class and
            instance variables, and private variables.

        .. grid-item-card:: Python Reference: Special method names
            :link: https://docs.python.org/3.12/reference/datamodel.html#special-method-names
            :class-card: sd-border-secondary

            ``__init__``, ``__repr__``, ``__str__``, the comparison
            methods, ``__hash__``, ``__len__``, and the arithmetic
            methods with their reflected forms.

        .. grid-item-card:: Python Library: property
            :link: https://docs.python.org/3.12/library/functions.html#property
            :class-card: sd-border-secondary

            Getters, setters, and read-only properties.

        .. grid-item-card:: PEP 8: Naming Conventions
            :link: https://peps.python.org/pep-0008/#naming-conventions
            :class-card: sd-border-secondary

            CapWords class names, and public and non-public names.


.. dropdown:: Exceptions
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Python Tutorial: Errors and Exceptions
            :link: https://docs.python.org/3.12/tutorial/errors.html
            :class-card: sd-border-secondary

            Handling exceptions with ``try`` and ``except``, and raising
            them with ``raise``.

        .. grid-item-card:: Python Library: Built-in Exceptions
            :link: https://docs.python.org/3.12/library/exceptions.html
            :class-card: sd-border-secondary

            When Python raises ``ValueError``, ``TypeError``, and
            ``AttributeError``.


.. dropdown:: Design and Tools
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: OMG: the UML specification
            :link: https://www.omg.org/spec/UML/
            :class-card: sd-border-secondary

            The standard behind the class box of the design phase.

        .. grid-item-card:: Mermaid: class diagrams
            :link: https://mermaid.js.org/syntax/classDiagram.html
            :class-card: sd-border-secondary

            Class diagrams written as text, drawn by many documentation
            tools.

        .. grid-item-card:: Ruff: SLF001 (private-member-access)
            :link: https://docs.astral.sh/ruff/rules/private-member-access/
            :class-card: sd-border-secondary

            Reports a non-public attribute used from outside its class.

        .. grid-item-card:: Ruff: RUF012 (mutable-class-default)
            :link: https://docs.astral.sh/ruff/rules/mutable-class-default/
            :class-card: sd-border-secondary

            Reports a list or dictionary used as a class attribute.
