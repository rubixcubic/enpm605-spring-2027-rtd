====================================================
References
====================================================

.. dropdown:: Lecture 10
    :class-container: sd-border-secondary
    :open:

    .. card::
        :class-card: sd-border-secondary

        **ENPM605 L10: Parameters, Custom Interfaces, Services, and Actions**

        Covers declaring a node's parameters with a type, a range, and a
        description; setting them at start, from a file, from a launch
        file, and while the node runs; an interface package with a
        message, a service, and an action; a service server, a client
        that calls once, and a client inside a node; an action server
        with feedback and cancel, and an action client; and Quality of
        Service: reliability, durability, history depth, and when a
        publisher's and a subscriber's settings are compatible.


.. dropdown:: Course
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: ROS repository
            :link: https://github.com/rubixcubic/enpm605-spring-2027-ros
            :class-card: sd-border-secondary

            The packages ``competition_interfaces``,
            ``competition_params``, ``competition_services``,
            ``competition_actions``, and ``competition_qos``, and the
            script ``errors/field_checks.py``, are in ``lecture10/``.

        .. grid-item-card:: Lecture 9
            :link: /lectures/lecture9/l9_index
            :link-type: doc
            :class-card: sd-border-secondary

            Parameters set at start with ``-p``, launch files,
            ``FindPackageShare``, executors, and callback groups, which
            this lecture uses for service clients and action servers.

        .. grid-item-card:: Lecture 8
            :link: /lectures/lecture8/l8_index
            :link-type: doc
            :class-card: sd-border-secondary

            Topics, messages, the battery nodes, and the parameter
            services and ``/parameter_events`` that this lecture
            explains.

        .. grid-item-card:: Memory reading
            :link: /reading_material/memory/mem_index
            :link-type: doc
            :class-card: sd-border-secondary

            "Objects Live on the Heap": a node's attributes, such as the
            server's battery, belong to one node object, which every
            callback of the node sees; each accepted task changes it.


.. dropdown:: Parameters
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: ROS 2 documentation: Parameters
            :link: https://docs.ros.org/en/jazzy/Concepts/Basic/About-Parameters.html
            :class-card: sd-border-secondary

            Declaring parameters, types, descriptors, the three parameter
            callbacks, and the parameter services.

        .. grid-item-card:: ROS 2 tutorial: Using parameters in a class (Python)
            :link: https://docs.ros.org/en/jazzy/Tutorials/Beginner-Client-Libraries/Using-Parameters-In-A-Class-Python.html
            :class-card: sd-border-secondary

            A Python node that declares and reads a parameter.

        .. grid-item-card:: ROS 2 tutorial: Understanding parameters
            :link: https://docs.ros.org/en/jazzy/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Parameters/Understanding-ROS2-Parameters.html
            :class-card: sd-border-secondary

            ``ros2 param list``, ``get``, ``set``, ``dump``, and ``load``.

        .. grid-item-card:: ROS 2 tutorial: Monitoring for parameter changes (Python)
            :link: https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Monitoring-For-Parameter-Changes-Python.html
            :class-card: sd-border-secondary

            Watching another node's parameters through
            ``/parameter_events``.


.. dropdown:: Custom Interfaces
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: ROS 2 documentation: Interfaces
            :link: https://docs.ros.org/en/jazzy/Concepts/Basic/About-Interfaces.html
            :class-card: sd-border-secondary

            Field types, field names and defaults, constants, and the
            ``.srv`` and ``.action`` formats.

        .. grid-item-card:: ROS 2 tutorial: Creating custom msg and srv files
            :link: https://docs.ros.org/en/jazzy/Tutorials/Beginner-Client-Libraries/Custom-ROS2-Interfaces.html
            :class-card: sd-border-secondary

            An interface package, its ``CMakeLists.txt`` and
            ``package.xml``, and nodes that use its types.

        .. grid-item-card:: ROS 2 tutorial: Creating an action
            :link: https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Creating-an-Action.html
            :class-card: sd-border-secondary

            The three parts of an ``.action`` file, and how to build it.


.. dropdown:: Services and Actions
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: ROS 2 documentation: Services
            :link: https://docs.ros.org/en/jazzy/Concepts/Basic/About-Services.html
            :class-card: sd-border-secondary

            The service server and the service client.

        .. grid-item-card:: ROS 2 tutorial: Writing a simple service and client (Python)
            :link: https://docs.ros.org/en/jazzy/Tutorials/Beginner-Client-Libraries/Writing-A-Simple-Py-Service-And-Client.html
            :class-card: sd-border-secondary

            ``create_service()``, ``create_client()``, and
            ``call_async()``.

        .. grid-item-card:: ROS 2 how-to: Synchronous vs. asynchronous service clients
            :link: https://docs.ros.org/en/jazzy/How-To-Guides/Sync-Vs-Async.html
            :class-card: sd-border-secondary

            Why ``call()`` deadlocks in a callback, and why the docs
            recommend ``call_async()``.

        .. grid-item-card:: ROS 2 how-to: Using Callback Groups
            :link: https://docs.ros.org/en/jazzy/How-To-Guides/Using-callback-groups.html
            :class-card: sd-border-secondary

            "Avoiding deadlocks": the groups that let a synchronous call
            in a callback return.

        .. grid-item-card:: ROS 2 documentation: Actions
            :link: https://docs.ros.org/en/jazzy/Concepts/Basic/About-Actions.html
            :class-card: sd-border-secondary

            Goals, feedback, results, and cancel.

        .. grid-item-card:: ROS 2 tutorial: Writing an action server and client (Python)
            :link: https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Writing-an-Action-Server-Client/Py.html
            :class-card: sd-border-secondary

            ``ActionServer``, ``ActionClient``, feedback, and the result.


.. dropdown:: Quality of Service
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: ROS 2 documentation: Quality of Service settings
            :link: https://docs.ros.org/en/jazzy/Concepts/Intermediate/About-Quality-of-Service-Settings.html
            :class-card: sd-border-secondary

            The policies, the predefined profiles, the compatibility
            tables, and the QoS events.
