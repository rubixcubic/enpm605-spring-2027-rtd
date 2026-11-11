====================================================
References
====================================================

.. dropdown:: Lecture 11
    :class-container: sd-border-secondary
    :open:

    .. card::
        :class-card: sd-border-secondary

        **ENPM605 L11: Coordinate Frames, TF2, and Mobile Robot Control**

        Covers a robot's pose, a position and a quaternion; REP 103's
        axes; a TurtleBot3 Waffle in Gazebo Harmonic, the bridge to ROS 2,
        and RViz2's displays; the differential drive model, checked
        against the wheel speeds; odometry and its drift, measured against
        Gazebo; a proportional controller that drives to a goal; REP 105's
        frames and the robot's frame tree; and TF2: a static broadcaster,
        ``static_transform_publisher``, a listener, and a controller that
        reads the goal from TF2.


.. dropdown:: Course
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: ROS repository
            :link: https://github.com/rubixcubic/enpm605-spring-2027-ros
            :class-card: sd-border-secondary

            The packages ``competition_motion`` and
            ``competition_frames``, and the script
            ``scripts/pose_demo.py``, are in ``lecture11/``.

        .. grid-item-card:: Lecture 10
            :link: /lectures/lecture10/l10_index
            :link-type: doc
            :class-card: sd-border-secondary

            Parameters, which set the controller's goal and gains, and
            QoS: why ``/tf_static`` and ``/robot_description`` reach a
            node that starts late.

        .. grid-item-card:: Lecture 5
            :link: /lectures/lecture5/l5_index
            :link-type: doc
            :class-card: sd-border-secondary

            Rotation matrices, homogeneous transforms such as
            ``T_world_robot``, composing transforms, and a path as a
            running sum of steps.

        .. grid-item-card:: Memory reading
            :link: /reading_material/memory/mem_lecture
            :link-type: doc
            :class-card: sd-border-secondary

            "Reference Counting": the controller replaces its pose tuple
            about 28 times a second, and each old tuple is freed when its
            reference count drops to 0.


.. dropdown:: Pose and Conventions
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: REP 103: Standard Units of Measure and Coordinate Conventions
            :link: https://www.ros.org/reps/rep-0103.html
            :class-card: sd-border-secondary

            Right-handed frames, x forward, y left, z up, radians, and
            the preferred ways to write a rotation.

        .. grid-item-card:: ROS 2 tutorial: Quaternion fundamentals
            :link: https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Tf2/Quaternion-Fundamentals.html
            :class-card: sd-border-secondary

            The order (x, y, z, w), unit length, and quaternion
            operations in ROS 2.


.. dropdown:: Simulation
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Gazebo: Installing Gazebo with ROS
            :link: https://gazebosim.org/docs/harmonic/ros_installation/
            :class-card: sd-border-secondary

            Which Gazebo goes with which ROS 2, and the
            ``ros-${ROS_DISTRO}-ros-gz`` package.

        .. grid-item-card:: Gazebo: Use ROS 2 to interact with Gazebo
            :link: https://gazebosim.org/docs/harmonic/ros2_integration/
            :class-card: sd-border-secondary

            ``ros_gz_bridge``: the syntax of a bridged topic, and a bridge
            set up from a YAML file.

        .. grid-item-card:: Gazebo API: the DiffDrive system
            :link: https://gazebosim.org/api/sim/8/classgz_1_1sim_1_1systems_1_1DiffDrive.html
            :class-card: sd-border-secondary

            The settings of the plugin that drives the Waffle's wheels and
            publishes its odometry.

        .. grid-item-card:: Gazebo Transport: environment variables
            :link: https://gazebosim.org/api/transport/13/envvars.html
            :class-card: sd-border-secondary

            ``GZ_PARTITION`` and the other variables that decide which
            Gazebo processes see each other.

        .. grid-item-card:: ROS 2 tutorial: RViz User Guide
            :link: https://docs.ros.org/en/jazzy/Tutorials/Intermediate/RViz/RViz-User-Guide/RViz-User-Guide.html
            :class-card: sd-border-secondary

            Displays, and the fixed frame.


.. dropdown:: Frames and TF2
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: REP 105: Coordinate Frames for Mobile Platforms
            :link: https://www.ros.org/reps/rep-0105.html
            :class-card: sd-border-secondary

            ``base_link``, ``odom``, ``map``, and ``earth``, and why they
            form a tree.

        .. grid-item-card:: ROS 2 documentation: Tf2
            :link: https://docs.ros.org/en/jazzy/Concepts/Intermediate/About-Tf2.html
            :class-card: sd-border-secondary

            What tf2 keeps, broadcasting and listening, and static
            transforms.

        .. grid-item-card:: ROS 2 tutorial: Introducing tf2
            :link: https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Tf2/Introduction-To-Tf2.html
            :class-card: sd-border-secondary

            The turtlesim demo, ``view_frames``, and ``tf2_echo``.

        .. grid-item-card:: ROS 2 tutorial: Writing a static broadcaster (Python)
            :link: https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Tf2/Writing-A-Tf2-Static-Broadcaster-Py.html
            :class-card: sd-border-secondary

            ``StaticTransformBroadcaster``, and
            ``static_transform_publisher`` on the command line and in
            launch files.

        .. grid-item-card:: ROS 2 tutorial: Writing a broadcaster (Python)
            :link: https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Tf2/Writing-A-Tf2-Broadcaster-Py.html
            :class-card: sd-border-secondary

            ``TransformBroadcaster``, a transform that changes over time
            (Exercise 4).

        .. grid-item-card:: ROS 2 tutorial: Writing a listener (Python)
            :link: https://docs.ros.org/en/jazzy/Tutorials/Intermediate/Tf2/Writing-A-Tf2-Listener-Py.html
            :class-card: sd-border-secondary

            ``Buffer``, ``TransformListener``, and ``lookup_transform()``.
