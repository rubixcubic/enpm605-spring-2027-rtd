====================================================
References
====================================================

.. dropdown:: Lecture 8
    :class-container: sd-border-secondary
    :open:

    .. card::
        :class-card: sd-border-secondary

        **ENPM605 L8: ROS 2 Foundations**

        Covers what ROS is and the Jazzy distribution; processes, and a
        monolithic design against a distributed one; nodes, topics,
        services, actions, and messages; the layers from ``rclpy`` down
        to DDS; discovery and the domain ID; the publish/subscribe model;
        the ``ros2`` introspection commands; workspaces, packages, and
        ``colcon``; and nodes written with ``rclpy``: a node class, a
        timer, spinning, a publisher, and a subscriber for the robotics
        competition's battery.


.. dropdown:: Course
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: ROS repository
            :link: https://github.com/rubixcubic/enpm605-spring-2027-ros
            :class-card: sd-border-secondary

            The package ``competition_nodes`` is in ``lecture8/``. Clone
            the repository into ``~/ros_ws/src``.

        .. grid-item-card:: Course code repository
            :link: https://github.com/rubixcubic/enpm605-spring-2027-python
            :class-card: sd-border-secondary

            ``process_demo.py``, ``message_demo.py``, ``spin_probe.py``,
            and ``errors/bad_publish.py`` are in ``lecture8/``.

        .. grid-item-card:: Memory reading
            :link: /reading_material/memory/mem_index
            :link-type: doc
            :class-card: sd-border-secondary

            "Objects Live on the Heap": each process has its own heap, so
            two nodes in two processes share no objects.

        .. grid-item-card:: ROS 2 setup
            :link: /setup/ros_setup
            :link-type: doc
            :class-card: sd-border-secondary

            Installing ROS 2 Jazzy and Gazebo Harmonic on Ubuntu 24.04.


.. dropdown:: ROS 2 and Its Distributions
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: ROS 2 Jazzy documentation
            :link: https://docs.ros.org/en/jazzy/index.html
            :class-card: sd-border-secondary

            The home page: what ROS is, with the tutorials, how-to
            guides, and concepts.

        .. grid-item-card:: ROS 2 documentation: Distributions
            :link: https://docs.ros.org/en/jazzy/Releases.html
            :class-card: sd-border-secondary

            Every ROS 2 distribution, with its release and end-of-life
            dates.

        .. grid-item-card:: REP 2000: ROS 2 Releases and Target Platforms
            :link: https://www.ros.org/reps/rep-2000.html
            :class-card: sd-border-secondary

            The platforms and middleware each distribution targets.

        .. grid-item-card:: ROS 2 tutorial: Configuring environment
            :link: https://docs.ros.org/en/jazzy/Tutorials/Beginner-CLI-Tools/Configuring-ROS2-Environment.html
            :class-card: sd-border-secondary

            Sourcing, ``ROS_DOMAIN_ID``, and
            ``ROS_AUTOMATIC_DISCOVERY_RANGE``.


.. dropdown:: Concepts
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Nodes
            :link: https://docs.ros.org/en/jazzy/Concepts/Basic/About-Nodes.html
            :class-card: sd-border-secondary

            What a node is, and how nodes talk to each other.

        .. grid-item-card:: Topics
            :link: https://docs.ros.org/en/jazzy/Concepts/Basic/About-Topics.html
            :class-card: sd-border-secondary

            Publish/subscribe, anonymous and strongly typed.

        .. grid-item-card:: Interfaces
            :link: https://docs.ros.org/en/jazzy/Concepts/Basic/About-Interfaces.html
            :class-card: sd-border-secondary

            ``.msg``, ``.srv``, and ``.action`` files, and field types.

        .. grid-item-card:: Discovery
            :link: https://docs.ros.org/en/jazzy/Concepts/Basic/About-Discovery.html
            :class-card: sd-border-secondary

            How nodes find each other on a domain.

        .. grid-item-card:: The ROS_DOMAIN_ID
            :link: https://docs.ros.org/en/jazzy/Concepts/Intermediate/About-Domain-ID.html
            :class-card: sd-border-secondary

            Why domains exist, and which IDs are safe to use.

        .. grid-item-card:: Client libraries
            :link: https://docs.ros.org/en/jazzy/Concepts/Basic/About-Client-Libraries.html
            :class-card: sd-border-secondary

            ``rclpy``, ``rclcpp``, and the ``rcl`` C library under both.

        .. grid-item-card:: Different ROS 2 middleware vendors
            :link: https://docs.ros.org/en/jazzy/Concepts/Intermediate/About-Different-Middleware-Vendors.html
            :class-card: sd-border-secondary

            DDS, the rmw layer, and Fast DDS as the default.

        .. grid-item-card:: Logging and logger configuration
            :link: https://docs.ros.org/en/jazzy/Concepts/Intermediate/About-Logging.html
            :class-card: sd-border-secondary

            Severities, ``/rosout``, and
            ``RCUTILS_CONSOLE_OUTPUT_FORMAT``.


.. dropdown:: Tutorials
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Understanding nodes
            :link: https://docs.ros.org/en/jazzy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Nodes/Understanding-ROS2-Nodes.html
            :class-card: sd-border-secondary

            ``ros2 run``, ``ros2 node list``, and ``ros2 node info``.

        .. grid-item-card:: Understanding topics
            :link: https://docs.ros.org/en/jazzy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html
            :class-card: sd-border-secondary

            ``rqt_graph``, ``ros2 topic echo``, ``info``, and ``hz``.

        .. grid-item-card:: Using colcon to build packages
            :link: https://docs.ros.org/en/jazzy/Tutorials/Beginner-Client-Libraries/Colcon-Tutorial.html
            :class-card: sd-border-secondary

            Workspaces, underlays and overlays, and
            ``--symlink-install``.

        .. grid-item-card:: Creating a package
            :link: https://docs.ros.org/en/jazzy/Tutorials/Beginner-Client-Libraries/Creating-Your-First-ROS2-Package.html
            :class-card: sd-border-secondary

            ``ros2 pkg create`` and the files it makes.

        .. grid-item-card:: Writing a simple publisher and subscriber (Python)
            :link: https://docs.ros.org/en/jazzy/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Publisher-And-Subscriber.html
            :class-card: sd-border-secondary

            The tutorial's publisher and subscriber, with the queue size.

        .. grid-item-card:: rclpy API documentation
            :link: https://docs.ros.org/en/jazzy/p/rclpy/
            :class-card: sd-border-secondary

            ``Node``, ``create_publisher()``, ``create_subscription()``,
            ``create_timer()``, and ``spin()``.
