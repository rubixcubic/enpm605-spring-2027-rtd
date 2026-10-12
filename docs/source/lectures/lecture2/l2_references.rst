References
==========

.. dropdown:: Lecture 2
    :class-container: sd-border-secondary
    :open:

    .. card::
        :class-card: sd-border-secondary

        **ENPM605 L2: Python Fundamentals, Part I**

        Covers modules and packages, the four forms of ``import``, where
        Python looks for modules (``sys.path``, ``PYTHONPATH``, ROS 2's
        ``setup.bash``), and the main guard; indentation; the ``bool``
        type and truth value testing; arithmetic, comparison, logical,
        and membership operators; ``int``, ``float``, and float
        tolerance; strings, f-strings, methods, indexing, and slicing;
        and ``if``, ``elif``, ``else``, and ``match``.


.. dropdown:: Course
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Course code repository
            :link: https://github.com/rubixcubic/enpm605-spring-2027-python
            :class-card: sd-border-secondary

            The Lecture 2 programs are in the ``lecture2/`` folder.

        .. grid-item-card:: Memory reading
            :link: /reading_material/memory/mem_index
            :link-type: doc
            :class-card: sd-border-secondary

            "Small Integers and Interned Strings" explains why ``256`` is
            one shared object and ``257`` is not.


.. dropdown:: Modules and Packages
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Python Tutorial: Modules
            :link: https://docs.python.org/3.12/tutorial/modules.html
            :class-card: sd-border-secondary

            Modules, packages, and ``__name__``.

        .. grid-item-card:: The import system
            :link: https://docs.python.org/3.12/reference/import.html
            :class-card: sd-border-secondary

            Regular packages and namespace packages.

        .. grid-item-card:: The initialization of sys.path
            :link: https://docs.python.org/3.12/library/sys_path_init.html
            :class-card: sd-border-secondary

            What goes first on ``sys.path``, and why.

        .. grid-item-card:: PYTHONPATH
            :link: https://docs.python.org/3.12/using/cmdline.html#envvar-PYTHONPATH
            :class-card: sd-border-secondary

            The environment variable ROS 2 sets when you source it.


.. dropdown:: Types and Operators
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Truth value testing
            :link: https://docs.python.org/3.12/library/stdtypes.html#truth-value-testing
            :class-card: sd-border-secondary

            The objects Python considers false.

        .. grid-item-card:: Boolean operations: and, or, not
            :link: https://docs.python.org/3.12/library/stdtypes.html#boolean-operations-and-or-not
            :class-card: sd-border-secondary

            What ``and`` and ``or`` return, and short-circuit evaluation.

        .. grid-item-card:: Numeric types: int, float, complex
            :link: https://docs.python.org/3.12/library/stdtypes.html#numeric-types-int-float-complex
            :class-card: sd-border-secondary

            The operators, floor division, and conversions.

        .. grid-item-card:: Floating-point arithmetic
            :link: https://docs.python.org/3.12/tutorial/floatingpoint.html
            :class-card: sd-border-secondary

            Why ``0.1 + 0.2`` is not ``0.3``.

        .. grid-item-card:: math.isclose()
            :link: https://docs.python.org/3.12/library/math.html#math.isclose
            :class-card: sd-border-secondary

            Relative and absolute tolerances.

        .. grid-item-card:: The power operator
            :link: https://docs.python.org/3.12/reference/expressions.html#the-power-operator
            :class-card: sd-border-secondary

            Why ``-1**2`` is ``-1``.


.. dropdown:: Strings
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Text sequence type: str
            :link: https://docs.python.org/3.12/library/stdtypes.html#text-sequence-type-str
            :class-card: sd-border-secondary

            String literals, indexing, and slicing.

        .. grid-item-card:: String methods
            :link: https://docs.python.org/3.12/library/stdtypes.html#string-methods
            :class-card: sd-border-secondary

            ``strip()``, ``split()``, ``join()``, ``find()``, and the rest.

        .. grid-item-card:: f-strings
            :link: https://docs.python.org/3.12/reference/lexical_analysis.html#f-strings
            :class-card: sd-border-secondary

            Format specifiers and the ``=`` specifier.

        .. grid-item-card:: What's New in Python 3.12
            :link: https://docs.python.org/3.12/whatsnew/3.12.html#other-language-changes
            :class-card: sd-border-secondary

            The ``SyntaxWarning`` for invalid escape sequences.


.. dropdown:: Control Flow and Style
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Python Tutorial: match statements
            :link: https://docs.python.org/3.12/tutorial/controlflow.html#match-statements
            :class-card: sd-border-secondary

            Literal, sequence, and capture patterns, and guards.

        .. grid-item-card:: PEP 636: Structural Pattern Matching, Tutorial
            :link: https://peps.python.org/pep-0636/
            :class-card: sd-border-secondary

            A longer tutorial built on a text-command example.

        .. grid-item-card:: The match statement (reference)
            :link: https://docs.python.org/3.12/reference/compound_stmts.html#the-match-statement
            :class-card: sd-border-secondary

            Irrefutable patterns and capture patterns.

        .. grid-item-card:: PEP 8: Style Guide for Python Code
            :link: https://peps.python.org/pep-0008/#indentation
            :class-card: sd-border-secondary

            4 spaces per indentation level, and no mixing of tabs and
            spaces.
