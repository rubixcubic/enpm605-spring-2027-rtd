====================================================
References
====================================================

.. dropdown:: Lecture 5
    :class-container: sd-border-secondary
    :open:

    .. card::
        :class-card: sd-border-secondary

        **ENPM605 L5: NumPy and Matplotlib for Robotics**

        Covers NumPy arrays against lists, measured for memory and
        speed; shape, data type, indexing, slicing, views and copies;
        Boolean masks; universal functions, reductions along an axis,
        and broadcasting; vectors, rotation matrices, and homogeneous 2D
        transforms; a ROS 2 ``LaserScan`` as an array, with NaN and inf
        readings and the nearest obstacle; plotting a trajectory and a
        scan with Matplotlib; and NumPy and Matplotlib as uv
        dependencies of the course project, next to ROS 2 Jazzy.


.. dropdown:: Course
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Course code repository
            :link: https://github.com/rubixcubic/enpm605-spring-2027-python
            :class-card: sd-border-secondary

            The Lecture 5 programs are in the ``lecture5/`` folder.

        .. grid-item-card:: Memory reading
            :link: /reading_material/memory/mem_index
            :link-type: doc
            :class-card: sd-border-secondary

            "Measuring Memory": a list of floats against a NumPy array,
            measured with ``tracemalloc``.

        .. grid-item-card:: ROS 2 setup
            :link: /setup/ros_setup
            :link-type: doc
            :class-card: sd-border-secondary

            Why the course keeps NumPy below version 2.

        .. grid-item-card:: uv: Managing dependencies
            :link: https://docs.astral.sh/uv/concepts/projects/dependencies/
            :class-card: sd-border-secondary

            ``uv add``, ``pyproject.toml``, and the lockfile.

        .. grid-item-card:: PEP 420: Implicit Namespace Packages
            :link: https://peps.python.org/pep-0420/
            :class-card: sd-border-secondary

            Why a regular package in a later folder wins over a folder
            without ``__init__.py``.


.. dropdown:: NumPy
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: NumPy: the absolute basics for beginners
            :link: https://numpy.org/doc/1.26/user/absolute_beginners.html
            :class-card: sd-border-secondary

            Making arrays, shape and size, indexing, and basic
            operations.

        .. grid-item-card:: NumPy: What is NumPy?
            :link: https://numpy.org/doc/1.26/user/whatisnumpy.html
            :class-card: sd-border-secondary

            Why array operations run at "near-C speeds": vectorization
            and broadcasting.

        .. grid-item-card:: NumPy: Indexing on ndarrays
            :link: https://numpy.org/doc/1.26/user/basics.indexing.html
            :class-card: sd-border-secondary

            Basic indexing and slicing, Boolean masks, and indexing with
            arrays of positions.

        .. grid-item-card:: NumPy: Copies and views
            :link: https://numpy.org/doc/1.26/user/basics.copies.html
            :class-card: sd-border-secondary

            When an operation shares the numbers of another array and
            when it copies them.

        .. grid-item-card:: NumPy: Data types
            :link: https://numpy.org/doc/1.26/user/basics.types.html
            :class-card: sd-border-secondary

            ``float64``, ``float32``, ``uint8``, and the other types.

        .. grid-item-card:: NumPy: Universal functions (ufunc)
            :link: https://numpy.org/doc/1.26/reference/ufuncs.html
            :class-card: sd-border-secondary

            Functions that work item by item, such as ``np.sqrt`` and
            ``np.arctan2``.

        .. grid-item-card:: NumPy: Broadcasting
            :link: https://numpy.org/doc/1.26/user/basics.broadcasting.html
            :class-card: sd-border-secondary

            The rule that matches the shapes of two arrays.

        .. grid-item-card:: NumPy: For downstream package authors
            :link: https://numpy.org/doc/stable/dev/depending_on_numpy.html
            :class-card: sd-border-secondary

            Why packages built against NumPy 1.x do not work with
            NumPy 2.


.. dropdown:: Laser Scans and Frames in ROS
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: sensor_msgs/msg/LaserScan.msg (Jazzy)
            :link: https://github.com/ros2/common_interfaces/blob/jazzy/sensor_msgs/msg/LaserScan.msg
            :class-card: sd-border-secondary

            The fields of a scan, with their comments on angles and
            valid ranges.

        .. grid-item-card:: REP 117: Informational Distance Measurements
            :link: https://www.ros.org/reps/rep-0117.html
            :class-card: sd-border-secondary

            -Inf, NaN, and +Inf in range readings, and the check that
            rejects all three.

        .. grid-item-card:: REP 103: Standard Units and Coordinate Conventions
            :link: https://www.ros.org/reps/rep-0103.html
            :class-card: sd-border-secondary

            Meters and radians; x forward, y left, z up for a robot's
            body.

        .. grid-item-card:: Python: array module
            :link: https://docs.python.org/3.12/library/array.html
            :class-card: sd-border-secondary

            The ``array.array`` type that rclpy uses for
            ``float32[]`` fields.


.. dropdown:: Matplotlib
    :class-container: sd-border-secondary

    .. grid:: 1 1 2 2
        :gutter: 2

        .. grid-item-card:: Matplotlib 3.11.2: Quick start guide
            :link: https://matplotlib.org/3.11.2/users/explain/quick_start.html
            :class-card: sd-border-secondary

            Figures, Axes, and the explicit (object-oriented) interface.

        .. grid-item-card:: Matplotlib 3.11.2: Axes.plot
            :link: https://matplotlib.org/3.11.2/api/_as_gen/matplotlib.axes.Axes.plot.html
            :class-card: sd-border-secondary

            Lines and markers, and format strings such as ``"o"``.

        .. grid-item-card:: Matplotlib 3.11.2: Axes.set_aspect
            :link: https://matplotlib.org/3.11.2/api/_as_gen/matplotlib.axes.Axes.set_aspect.html
            :class-card: sd-border-secondary

            Equal scales on both axes, for a top view.

        .. grid-item-card:: Matplotlib 3.11.2: savefig
            :link: https://matplotlib.org/3.11.2/api/_as_gen/matplotlib.pyplot.savefig.html
            :class-card: sd-border-secondary

            Saving a figure to a file, and the ``dpi`` setting.
