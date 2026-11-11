====================================================
References
====================================================

.. dropdown:: Lecture 4
    :class-container: sd-border-secondary
    :open:

    .. card::
        :class-card: sd-border-secondary

        **ENPM605 L4: Functions**

        Covers ``def``, ``return``, and what a call does with a frame;
        positional, keyword, and default arguments, the shared mutable
        default, ``*args``, ``**kwargs``, and keyword-only parameters;
        scopes and the LEGB rule, ``global``, and ``nonlocal``;
        pass-by-assignment; type hints and mypy; recursion; functions as
        values, key functions, ``map()``, and ``filter()``; lambda
        functions and closures; callbacks; and ``functools.partial``.


.. dropdown:: Course
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Course code repository
            :link: https://github.com/rubixcubic/enpm605-spring-2027-python
            :class-card: sd-border-secondary

            The Lecture 4 programs are in the ``lecture4/`` folder.

        .. grid-item-card:: Memory reading
            :link: /reading_material/memory/mem_index
            :link-type: doc
            :class-card: sd-border-secondary

            "Names Live in Namespaces" and "Call Frames" draw the
            namespaces and frames this lecture uses.


.. dropdown:: Defining Functions and Arguments
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Python Tutorial: Defining Functions
            :link: https://docs.python.org/3.12/tutorial/controlflow.html#defining-functions
            :class-card: sd-border-secondary

            ``def``, the local symbol table, and ``None`` as the result
            of a function without ``return``.

        .. grid-item-card:: Python Tutorial: More on Defining Functions
            :link: https://docs.python.org/3.12/tutorial/controlflow.html#more-on-defining-functions
            :class-card: sd-border-secondary

            Default values, keyword arguments, ``*args``, ``**kwargs``,
            and unpacking in a call.

        .. grid-item-card:: Python Tutorial: Special Parameters
            :link: https://docs.python.org/3.12/tutorial/controlflow.html#special-parameters
            :class-card: sd-border-secondary

            Positional-only (``/``) and keyword-only (``*``) parameters.

        .. grid-item-card:: Language Reference: Function Definitions
            :link: https://docs.python.org/3.12/reference/compound_stmts.html#function-definitions
            :class-card: sd-border-secondary

            The exact rules, including when default values are
            evaluated.

        .. grid-item-card:: Python Tutorial: Documentation Strings
            :link: https://docs.python.org/3.12/tutorial/controlflow.html#documentation-strings
            :class-card: sd-border-secondary

            How to write a docstring.

        .. grid-item-card:: PEP 8: Naming Conventions
            :link: https://peps.python.org/pep-0008/#function-and-variable-names
            :class-card: sd-border-secondary

            Function names in lowercase, with words joined by
            underscores.


.. dropdown:: Scopes and Argument Passing
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Python Tutorial: Python Scopes and Namespaces
            :link: https://docs.python.org/3.12/tutorial/classes.html#python-scopes-and-namespaces
            :class-card: sd-border-secondary

            Local, enclosing, global, and built-in names, ``global``,
            and ``nonlocal``.

        .. grid-item-card:: Programming FAQ: UnboundLocalError
            :link: https://docs.python.org/3.12/faq/programming.html#why-am-i-getting-an-unboundlocalerror-when-the-variable-has-a-value
            :class-card: sd-border-secondary

            Why an assignment makes a name local in the whole function.

        .. grid-item-card:: Programming FAQ: Output Parameters
            :link: https://docs.python.org/3.12/faq/programming.html#how-do-i-write-a-function-with-output-parameters-call-by-reference
            :class-card: sd-border-secondary

            "Arguments are passed by assignment in Python", and how to
            return several results.


.. dropdown:: Type Hints and Recursion
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: typing: Support for Type Hints
            :link: https://docs.python.org/3.12/library/typing.html
            :class-card: sd-border-secondary

            The hints Python understands, and the note that it does not
            enforce them.

        .. grid-item-card:: mypy: Type Hints Cheat Sheet
            :link: https://mypy.readthedocs.io/en/stable/cheat_sheet_py3.html
            :class-card: sd-border-secondary

            The common hints, with examples.

        .. grid-item-card:: sys.getrecursionlimit()
            :link: https://docs.python.org/3.12/library/sys.html#sys.getrecursionlimit
            :class-card: sd-border-secondary

            The limit that stops a recursion with no base case.


.. dropdown:: Functions as Values
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Functional Programming HOWTO
            :link: https://docs.python.org/3.12/howto/functional.html
            :class-card: sd-border-secondary

            Functions as values, ``map()``, ``filter()``, and lambdas.

        .. grid-item-card:: Glossary: key function
            :link: https://docs.python.org/3.12/glossary.html#term-key-function
            :class-card: sd-border-secondary

            What ``sorted()``, ``min()``, and ``max()`` do with ``key=``.

        .. grid-item-card:: Python Tutorial: Lambda Expressions
            :link: https://docs.python.org/3.12/tutorial/controlflow.html#lambda-expressions
            :class-card: sd-border-secondary

            The ``lambda`` syntax.

        .. grid-item-card:: Programming FAQ: Lambdas Defined in a Loop
            :link: https://docs.python.org/3.12/faq/programming.html#why-do-lambdas-defined-in-a-loop-with-different-values-all-return-the-same-result
            :class-card: sd-border-secondary

            Why functions made in a loop read the loop variable late.

        .. grid-item-card:: Glossary: callback
            :link: https://docs.python.org/3.12/glossary.html#term-callback
            :class-card: sd-border-secondary

            A function passed as an argument, to be called later.

        .. grid-item-card:: functools.partial
            :link: https://docs.python.org/3.12/library/functools.html#functools.partial
            :class-card: sd-border-secondary

            A new callable with some arguments filled in.
