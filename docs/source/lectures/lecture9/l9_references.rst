====================================================
References
====================================================

.. dropdown:: Lecture 9
    :class-container: sd-border-secondary
    :open:

    .. card::
        :class-card: sd-border-secondary

        **ENPM605 L9: Launch Files, Namespaces, Remapping, and Executors**

        Covers ROS arguments after ``--ros-args`` and what
        ``rclpy.init()`` reads; namespaces and relative, absolute, and
        private names; remapping node and topic names; setting a
        parameter's value when a node starts; Python launch files, with
        launch arguments, conditions, groups, and includes; the
        single-threaded and the multi-threaded executor; and mutually
        exclusive and reentrant callback groups, with Python's global
        interpreter lock.


.. dropdown:: Course
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: ROS repository
            :link: https://github.com/rubixcubic/enpm605-spring-2027-ros
            :class-card: sd-border-secondary

            The packages ``competition_bringup`` (launch files) and
            ``competition_executors`` are in ``lecture9/``; Lecture 8's
            ``competition_nodes`` is in ``lecture8/``.

        .. grid-item-card:: Course code repository
            :link: https://github.com/rubixcubic/enpm605-spring-2027-python
            :class-card: sd-border-secondary

            ``ros_args_demo.py`` and ``gil_demo.py`` are in
            ``lecture9/``.

        .. grid-item-card:: Memory reading
            :link: /reading_material/memory/mem_index
            :link-type: doc
            :class-card: sd-border-secondary

            "Objects Live on the Heap": the threads of one process share
            its heap, so two callbacks that run at the same time see the
            same objects.

        .. grid-item-card:: Lecture 8
            :link: /lectures/lecture8/l8_index
            :link-type: doc
            :class-card: sd-border-secondary

            Nodes, topics, packages, and the battery nodes that this
            lecture renames, remaps, and launches.


.. dropdown:: Names and Remapping
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Passing ROS arguments to nodes via the command-line
            :link: https://docs.ros.org/en/jazzy/How-To-Guides/Node-arguments.html
            :class-card: sd-border-secondary

            ``--ros-args``, ``-r`` for names and namespaces, and ``-p``
            for parameters.

        .. grid-item-card:: ROS 2 design: Topic and Service name mapping to DDS
            :link: https://design.ros2.org/articles/topic_and_service_names.html
            :class-card: sd-border-secondary

            Relative, absolute, and private names, and how a namespace
            completes them.

        .. grid-item-card:: ROS 2 design: Remapping Names
            :link: https://design.ros2.org/articles/static_remapping.html
            :class-card: sd-border-secondary

            The rule syntax ``match:=replacement``, ``__node``, and
            ``__ns``.

        .. grid-item-card:: ROS 2 documentation: Parameters
            :link: https://docs.ros.org/en/jazzy/Concepts/Basic/About-Parameters.html
            :class-card: sd-border-secondary

            What a parameter is, and how it is set when a node starts
            (Lecture 10 covers the rest).


.. dropdown:: Launch Files
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: ROS 2 documentation: Launch
            :link: https://docs.ros.org/en/jazzy/Concepts/Basic/About-Launch.html
            :class-card: sd-border-secondary

            What the launch system does, in one page.

        .. grid-item-card:: Integrating launch files into ROS 2 packages
            :link: https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Launch/Launch-system.html
            :class-card: sd-border-secondary

            The ``launch`` folder, ``setup.py``'s ``data_files``, and the
            ``launch.py`` ending.

        .. grid-item-card:: Using substitutions
            :link: https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Launch/Using-Substitutions.html
            :class-card: sd-border-secondary

            Launch arguments, ``LaunchConfiguration``, and including a
            launch file.

        .. grid-item-card:: Managing large projects
            :link: https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Launch/Using-ROS2-Launch-For-Large-Projects.html
            :class-card: sd-border-secondary

            Top-level launch files, ``PushROSNamespace``, remapping, and
            parameters in launch files.


.. dropdown:: Executors and Threads
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: ROS 2 documentation: Executors
            :link: https://docs.ros.org/en/jazzy/Concepts/Intermediate/About-Executors.html
            :class-card: sd-border-secondary

            Executor types, callback groups, and scheduling.

        .. grid-item-card:: Using Callback Groups
            :link: https://docs.ros.org/en/jazzy/How-To-Guides/Using-callback-groups.html
            :class-card: sd-border-secondary

            When to use a mutually exclusive or a reentrant group, and
            how to avoid deadlocks.

        .. grid-item-card:: Python glossary: global interpreter lock
            :link: https://docs.python.org/3.12/glossary.html#term-global-interpreter-lock
            :class-card: sd-border-secondary

            Why only one thread runs Python bytecode at a time.

        .. grid-item-card:: Python Library: threading
            :link: https://docs.python.org/3.12/library/threading.html
            :class-card: sd-border-secondary

            ``Thread``, ``start()``, ``join()``, and ``Lock``.
