====================================================
Glossary
====================================================

This glossary covers the key terms of ENPM605, grouped by the lecture that
introduces them and sorted A to Z within each lecture. Click a lecture in the
right panel to jump to its terms. A term that a later lecture uses again is
listed at the end of that lecture's section. Terms are added as each lecture
is posted.

.. only:: html

   .. raw:: html

      <div id="glossary-search-wrap" style="margin: 1.2em 0 0.6em 0;">
        <input
          id="glossary-search"
          type="search"
          placeholder="Filter terms..."
          autocomplete="off"
          spellcheck="false"
          style="
            width: 100%;
            max-width: 480px;
            padding: 0.45em 0.75em;
            font-size: 1em;
            border: 1px solid #ccc;
            border-radius: 4px;
            box-sizing: border-box;
          "
        />
        <span
          id="glossary-search-count"
          style="margin-left: 0.8em; font-size: 0.88em; color: #666;"
        ></span>
      </div>
      <div id="glossary-letters" aria-label="Show the terms that start with a letter"
           style="display: flex; flex-wrap: wrap; gap: 0.3em; margin: 0 0 1.4em 0;"></div>

      <style>
        #glossary-letters button {
          min-width: 2em;
          padding: 0.15em 0.45em;
          font: inherit;
          font-size: 0.95em;
          color: var(--pst-color-primary, #2f6fb5);
          background: transparent;
          border: 1px solid var(--pst-color-border, #ccc);
          border-radius: 4px;
          cursor: pointer;
        }
        #glossary-letters button.active {
          color: var(--pst-color-background, #fff);
          background: var(--pst-color-primary, #2f6fb5);
          border-color: var(--pst-color-primary, #2f6fb5);
        }
      </style>

      <script>
      (function () {
        /* The search box filters by text; a letter button shows the terms
           that start with that letter, across all lectures. Both apply
           together. A lecture section with no matching term is hidden. */
        function initGlossarySearch() {
          var input   = document.getElementById('glossary-search');
          var count   = document.getElementById('glossary-search-count');
          var letters = document.getElementById('glossary-letters');
          if (!input) return;
          var letter = '';

          function firstLetter(dt) {
            var m = dt.textContent.trim().match(/[a-z]/i);
            return m ? m[0].toUpperCase() : '';
          }

          /* One button per letter that starts a term, plus "All". */
          var present = {};
          document.querySelectorAll('dl.glossary dt').forEach(function (dt) {
            var c = firstLetter(dt);
            if (c) present[c] = true;
          });
          function addButton(label, value) {
            var btn = document.createElement('button');
            btn.type = 'button';
            btn.textContent = label;
            btn.dataset.letter = value;
            if (value === '') btn.classList.add('active');
            btn.addEventListener('click', function () {
              letter = value;
              letters.querySelectorAll('button').forEach(function (b) {
                b.classList.toggle('active', b.dataset.letter === letter);
              });
              run();
            });
            letters.appendChild(btn);
          }
          addButton('All', '');
          Object.keys(present).sort().forEach(function (c) { addButton(c, c); });

          function run() {
            var query = input.value.trim().toLowerCase();
            var filtering = query !== '' || letter !== '';
            var visible = 0;

            document.querySelectorAll('dl.glossary dt').forEach(function (dt) {
              var match = (!query || dt.textContent.toLowerCase().indexOf(query) !== -1)
                       && (!letter || firstLetter(dt) === letter);
              dt.style.display = match ? '' : 'none';
              var node = dt.nextElementSibling;
              while (node && node.tagName === 'DD') {
                node.style.display = match ? '' : 'none';
                node = node.nextElementSibling;
              }
              if (match) visible++;
            });

            /* Each lecture section: hide it when nothing in it matches, and
               hide its "Also used here" line while a filter is on. */
            document.querySelectorAll('section > h2').forEach(function (h2) {
              var section = h2.parentElement;
              var dts = section.querySelectorAll('dl.glossary dt');
              var any = Array.prototype.some.call(dts, function (dt) {
                return dt.style.display !== 'none';
              });
              section.style.display = (!filtering || any) ? '' : 'none';
              section.querySelectorAll('dl.glossary').forEach(function (dl) {
                dl.style.display = (!filtering || any) ? '' : 'none';
              });
              section.querySelectorAll('p.glossary-also').forEach(function (p) {
                p.style.display = filtering ? 'none' : '';
              });
            });

            count.textContent = filtering
              ? (visible === 1 ? '1 term found' : visible + ' terms found')
              : '';
          }

          input.addEventListener('input', run);
          input.addEventListener('keydown', function (e) {
            if (e.key === 'Escape') { input.value = ''; run(); input.blur(); }
          });
        }

        if (document.readyState === 'loading') {
          document.addEventListener('DOMContentLoaded', initGlossarySearch);
        } else {
          initGlossarySearch();
        }
      })();
      </script>

----


.. _g-l1:

L1: Course Introduction and Python Tooling
==========================================

.. glossary::

   Bytecode
      The low-level instructions CPython's compiler produces from a program and its interpreter loop executes. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   CPython
      The reference implementation of Python, written in C. The ``python3`` on Ubuntu is CPython. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Formatter
      A tool that rewrites the layout of code (spaces, line breaks) without changing what it does. Ruff includes one. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Immutable
      Describes an object that cannot change after it is created, such as an ``int``, a ``str``, or a ``tuple``. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Keyword
      A word with a special meaning in Python, such as ``class`` or ``if``, that cannot be a variable name. A soft keyword, such as ``match``, is a keyword only in some places. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Linter
      A tool that reads code and reports likely errors and style problems without running it. Ruff includes one. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Mutable
      Describes an object that can change in place, such as a ``list``, a ``dict``, or a ``set``. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Object identity
      The integer ``id()`` returns for an object. It stays the same while the object exists; in CPython it is the memory address. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   uv
      A tool that creates Python virtual environments and installs packages. The course uses version 0.13.0. (:doc:`L1 </lectures/lecture1/l1_lecture>`)

   Virtual environment
      A folder (``.venv``) with its own Python packages, separate from the system's. The course environment also sees Ubuntu's packages. (:doc:`L1 </lectures/lecture1/l1_lecture>`)


.. _g-l2:

L2: Python Fundamentals, Part I
===============================

.. glossary::

   Capture pattern
      A bare name in a ``case`` of a ``match`` statement. It always matches and binds the value to that name. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   f-string
      A string literal that starts with ``f``, in which each ``{expression}`` is replaced by its value, for example ``f"{speed:.2f} m/s"``. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Falsy
      Describes a value that counts as false in a condition: ``None``, ``False``, zero of any numeric type, and empty sequences and collections. Every other value is truthy. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Floor division
      The operator ``//``. It divides and rounds toward negative infinity, so ``-7 // 2`` is ``-4``. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Main guard
      The block ``if __name__ == "__main__":``. Its code runs when the file is run, and not when the file is imported. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Module
      One ``.py`` file. Another file can ``import`` it and use the names it defines. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Module search path
      The list of folders, ``sys.path``, that Python searches in order for each ``import``. The folder of the program you run comes first. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Package
      A folder of modules. A regular package holds an ``__init__.py`` file, which Python runs when the package is imported. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   PYTHONPATH
      An environment variable whose folders Python adds to ``sys.path``. Sourcing ROS 2 sets it, which makes ``rclpy`` importable. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Short-circuit evaluation
      ``and`` and ``or`` evaluate their right side only when the left side does not already decide the result. (:doc:`L2 </lectures/lecture2/l2_lecture>`)

   Slice
      A copy of part of a sequence, written ``s[start:stop:step]``. It holds the items from ``start`` up to, but not including, ``stop``. (:doc:`L2 </lectures/lecture2/l2_lecture>`)


.. _g-l3:

L3: Python Fundamentals, Part II
================================

.. glossary::

   Dictionary view
      The object that ``keys()``, ``values()``, or ``items()`` returns. It is not a copy: it shows the dictionary's current entries. (:doc:`L3 </lectures/lecture3/l3_lecture>`)

   Generator expression
      Written like a list comprehension without the brackets, such as ``(r * 100 for r in valid)``. It needs its own parentheses, except as the only argument of a call: ``any(r < 1.0 for r in valid)``. It gives its values one at a time and builds no list. (:doc:`L3 </lectures/lecture3/l3_lecture>`)

   Hashable
      Describes an object whose hash value never changes while it exists, so it can be a dictionary key or a set item. Lists, dictionaries, and sets are not hashable; a tuple is hashable when its items are. (:doc:`L3 </lectures/lecture3/l3_lecture>`)

   In-place operation
      An operation that changes an object instead of building a new one, such as ``list.sort()`` or ``list.append()``. List methods that work in place return ``None``. (:doc:`L3 </lectures/lecture3/l3_lecture>`)

   Iterable
      An object that can give its items one at a time, such as a list, a string, a ``range``, or a dictionary. A ``for`` loop accepts any iterable. (:doc:`L3 </lectures/lecture3/l3_lecture>`)

   Iterator
      An object that gives the items of an iterable one at a time: each ``next()`` returns the next item, and ``StopIteration`` is raised when none is left. ``iter()`` makes one; a ``for`` loop uses one. (:doc:`L3 </lectures/lecture3/l3_lecture>`)

   List comprehension
      An expression that builds a list from an iterable in one line, written ``[expression for item in iterable if condition]``. Dictionary and set comprehensions use braces. (:doc:`L3 </lectures/lecture3/l3_lecture>`)

   Unpacking
      Binding each item of an iterable to its own name, as in ``x, y, heading = pose``. ``*rest`` collects the items left over in a list. (:doc:`L3 </lectures/lecture3/l3_lecture>`)


.. _g-l4:

L4: Functions
=============

.. glossary::

   Argument
      A value passed to a function in a call. A positional argument is matched by its place, a keyword argument (``name=value``) by its name. (:doc:`L4 </lectures/lecture4/l4_lecture>`)

   Callable
      An object you can call with ``()``, such as a function, a built-in like ``print``, a type like ``int``, or a ``functools.partial`` object. (:doc:`L4 </lectures/lecture4/l4_lecture>`)

   Callback
      A function passed to other code, which calls it later, when something happens: a message arrives, or a timer fires. (:doc:`L4 </lectures/lecture4/l4_lecture>`)

   Closure
      A nested function that uses a name of its enclosing function, and keeps it in a cell after that function has returned. (:doc:`L4 </lectures/lecture4/l4_lecture>`)

   Lambda function
      A function written as one expression, ``lambda parameters: expression``, often passed as a ``key=``. Its name is ``<lambda>``. (:doc:`L4 </lectures/lecture4/l4_lecture>`)

   Parameter
      A name in a function's definition, such as ``distance_m`` in ``def drive_time(distance_m, speed_mps):``. A call binds it to an argument. (:doc:`L4 </lectures/lecture4/l4_lecture>`)

   Pass-by-assignment
      How Python passes arguments: each parameter is bound to the caller's object, as an assignment would. A function can change a mutable argument, but assigning the parameter does not change the caller's name. (:doc:`L4 </lectures/lecture4/l4_lecture>`)

   Recursive function
      A function that calls itself on a smaller problem. It needs a base case, which returns without another call. (:doc:`L4 </lectures/lecture4/l4_lecture>`)

   Scope
      The part of a program where a name can be used without a prefix. Python looks for a name in the local, enclosing, global, and built-in scopes, in that order (the LEGB rule). (:doc:`L4 </lectures/lecture4/l4_lecture>`)

   Type hint
      An annotation such as ``speed_mps: float`` or ``-> int | None`` that says what type a parameter or a return value should have. Python does not check it; a type checker such as mypy does. (:doc:`L4 </lectures/lecture4/l4_lecture>`)


.. _g-l5:

L5: NumPy and Matplotlib for Robotics
=====================================

.. glossary::

   Array view
      A NumPy array that uses the numbers of another array, such as a slice. Changing a view changes the original; ``.copy()`` makes an array with its own numbers. (:doc:`L5 </lectures/lecture5/l5_lecture>`)

   Boolean mask
      An array of ``True`` and ``False``, one per item, made by a comparison such as ``ranges < 1.0``. Indexing with it keeps the items where it is ``True``. (:doc:`L5 </lectures/lecture5/l5_lecture>`)

   Broadcasting
      How NumPy combines arrays of different shapes in one operation: comparing shapes from the right, a size of 1 or a missing size is used for every item of the other. (:doc:`L5 </lectures/lecture5/l5_lecture>`)

   Data type (dtype)
      The type of every item of a NumPy array, such as ``float64`` (8-byte floats), ``float32``, ``int64``, or ``uint8`` (whole numbers from 0 to 255). (:doc:`L5 </lectures/lecture5/l5_lecture>`)

   Homogeneous transform
      A 3 by 3 matrix that holds a rotation and a translation, so that one matrix product moves a 2D point, written with a third item 1, from one frame to another. (:doc:`L5 </lectures/lecture5/l5_lecture>`)

   NumPy array
      An object of type ``numpy.ndarray`` that stores numbers of one data type side by side in memory, with a shape. Arithmetic on it works item by item, with no loop in your code. (:doc:`L5 </lectures/lecture5/l5_lecture>`)

   Rotation matrix
      A 2 by 2 matrix, built from the cosine and sine of an angle, that turns a vector by that angle counterclockwise. Its transpose turns the vector back. (:doc:`L5 </lectures/lecture5/l5_lecture>`)

   Shape
      The size of a NumPy array along each axis, as a tuple: ``(4, 3)`` is 4 rows and 3 columns. (:doc:`L5 </lectures/lecture5/l5_lecture>`)

   Universal function (ufunc)
      A NumPy function that works on arrays item by item, such as ``np.sqrt()``, ``np.hypot()``, or ``np.arctan2()``. (:doc:`L5 </lectures/lecture5/l5_lecture>`)

   Vectorization
      Writing an operation on whole arrays instead of a loop over items. NumPy runs the loop in compiled code. (:doc:`L5 </lectures/lecture5/l5_lecture>`)


.. _g-l6:

L6: Object-Oriented Programming, Part I
=======================================

.. glossary::

   Abstraction
      Showing a caller what an object does and hiding how it does it: the caller uses the public interface, the methods and attributes it is meant to use, with their docstrings. (:doc:`L6 </lectures/lecture6/l6_lecture>`)

   Class
      A type you define with the ``class`` statement. It says which attributes its objects hold and which methods they have; calling it, as in ``Robot("Scout")``, makes an object. (:doc:`L6 </lectures/lecture6/l6_lecture>`)

   Class attribute
      A name assigned in the class body, outside any method, such as ``total_robots = 0``. It is shared by every object of the class; an assignment through an object makes an instance attribute instead. (:doc:`L6 </lectures/lecture6/l6_lecture>`)

   Dunder method
      A method whose name starts and ends with two underscores, such as ``__init__``, ``__repr__``, or ``__eq__``. Python calls it from a built-in operation: ``repr()``, ``==``, ``+``, ``len()``. (:doc:`L6 </lectures/lecture6/l6_lecture>`)

   Encapsulation
      Keeping an object's data with the methods that change it, so that the class can check every change. In Python, non-public names start with an underscore. (:doc:`L6 </lectures/lecture6/l6_lecture>`)

   Exception
      An object that reports an error. ``raise`` stops the code that detects the error; a ``try`` statement with an ``except`` clause handles it, and an unhandled exception stops the program with a traceback. (:doc:`L6 </lectures/lecture6/l6_lecture>`)

   Instance
      An object made by calling a class: ``scout = Robot("Scout")`` makes an instance of ``Robot``. ``isinstance(scout, Robot)`` is ``True``. (:doc:`L6 </lectures/lecture6/l6_lecture>`)

   Instance attribute
      A name in one object's own namespace, usually set in ``__init__`` with ``self.name = value``. ``vars(obj)`` lists them. (:doc:`L6 </lectures/lecture6/l6_lecture>`)

   Method
      A function defined in a class body. A call ``obj.m(x)`` passes the object as the first parameter, ``self``: it runs ``type(obj).m(obj, x)``. (:doc:`L6 </lectures/lecture6/l6_lecture>`)

   Property
      An attribute whose reading and assignment run methods, made with ``@property`` and ``@name.setter``. A property with no setter is read-only. (:doc:`L6 </lectures/lecture6/l6_lecture>`)


.. _g-l7:

L7: Object-Oriented Programming, Part II
========================================

.. glossary::

   __slots__
      A class attribute that lists the only attribute names an object of the class can have. The objects have no ``__dict__`` and take less memory. (:doc:`L7 </lectures/lecture7/l7_lecture>`)

   Abstract base class
      A base class, made with ``ABC``, whose methods marked ``@abstractmethod`` every subclass you make objects of must define; Python refuses to make an object of a class that does not. (:doc:`L7 </lectures/lecture7/l7_lecture>`)

   Aggregation
      A relationship in which an object holds parts made elsewhere, which stay when it is gone: a team and its robots. (:doc:`L7 </lectures/lecture7/l7_lecture>`)

   Association
      A relationship in which an object knows another object, made elsewhere, that is not its part: a team and the tasks it did. (:doc:`L7 </lectures/lecture7/l7_lecture>`)

   Class method
      A method made with ``@classmethod``. It receives the class as its first argument, ``cls``, and is often used to make objects, as ``Robot.from_dict()`` does. (:doc:`L7 </lectures/lecture7/l7_lecture>`)

   Composition
      A relationship in which an object makes its parts and, by design, no other object keeps them: a robot and the sensors it makes. Python still keeps a part while any name refers to it. (:doc:`L7 </lectures/lecture7/l7_lecture>`)

   Data class
      A class made with ``@dataclass``, which writes ``__init__``, ``__repr__``, and ``__eq__`` from the fields: the class variables that have a type annotation. With ``frozen=True``, its fields cannot change and it gets a ``__hash__``. (:doc:`L7 </lectures/lecture7/l7_lecture>`)

   Decorator
      A function that takes a function and returns a new one, applied with an ``@`` line above a ``def`` or ``class``: ``@log_calls`` above ``def drive_time`` is ``drive_time = log_calls(drive_time)``. (:doc:`L7 </lectures/lecture7/l7_lecture>`)

   Duck typing
      Using an object by the methods and attributes it has, without checking its class. (:doc:`L7 </lectures/lecture7/l7_lecture>`)

   Inheritance
      Making a new class, the subclass, from an existing one, the base class: ``class Scout(Robot):``. The subclass has every attribute and method of the base class, and adds or replaces some. (:doc:`L7 </lectures/lecture7/l7_lecture>`)

   Method resolution order
      The order in which Python searches classes for an attribute: the object's class, then its base classes, up to ``object``. A class's ``__mro__`` lists it. (:doc:`L7 </lectures/lecture7/l7_lecture>`)

   Polymorphism
      The same call does what each object's class says: ``robot.perform_task()`` drains a scout, a hauler, and a robot differently. (:doc:`L7 </lectures/lecture7/l7_lecture>`)

   Protocol
      A class made from ``typing.Protocol`` that lists the methods and attributes a caller needs. A type checker such as mypy accepts any class that has them, with no inheritance. (:doc:`L7 </lectures/lecture7/l7_lecture>`)

   Static method
      A method made with ``@staticmethod``. It receives neither the object nor the class: a plain function kept in the class. (:doc:`L7 </lectures/lecture7/l7_lecture>`)


.. _g-l8:

L8: ROS 2 Foundations
=====================

.. glossary::

   colcon
      The command that builds the packages of a workspace and installs them in its ``install`` folder. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   DDS (Data Distribution Service)
      A standard for publish/subscribe communication that ROS 2 is built on. It finds nodes (discovery) and carries their messages. Jazzy's default is Fast DDS. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Discovery
      How ROS 2 nodes find each other: each node advertises itself to the other nodes of its domain, with no central server. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Domain ID
      The number in ``ROS_DOMAIN_ID`` that groups nodes: nodes on the same domain discover each other, nodes on different domains do not. The default is 0. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Entry point
      A line in ``setup.py`` that names an executable and the function it runs, such as ``battery_publisher = competition_nodes.battery_publisher:main``. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Message (ROS 2)
      The data sent on a topic: typed fields described in a ``.msg`` file, such as ``sensor_msgs/msg/BatteryState``. In Python, an object with one attribute per field. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Node
      A program in a ROS 2 system that does one job and talks to other nodes through topics, services, and actions. In this course each node runs in its own process. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Overlay
      A workspace sourced after its underlay. Its packages are found first. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Process
      A running program. The operating system gives it its own memory and its own process ID (PID). (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Publisher
      The part of a node that sends messages on a topic, made with ``create_publisher()``. It sends whether or not a subscriber exists. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   rmw (ROS middleware interface)
      The layer between ``rcl`` and a DDS, so that ROS 2 code does not depend on one DDS. On Jazzy the default is ``rmw_fastrtps_cpp``, for Fast DDS. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   ROS distribution
      A versioned set of ROS packages, with a name, a release date, and an end-of-life date. This course uses ROS 2 Jazzy Jalisco. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   ROS package
      A folder of ROS 2 code built and installed as one unit, with a ``package.xml`` that names it and lists its dependencies. A Python one also has ``setup.py`` and ``setup.cfg``. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Sourcing
      Running a setup file with ``source`` so that the variables it sets stay set in the current shell. ROS 2 commands and packages work only in a shell that sourced ``/opt/ros/jazzy/setup.bash``. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Spin
      To run a node's callbacks as their events come (timers, messages), with ``rclpy.spin(node)``, until ROS 2 shuts down or Ctrl+C. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Subscriber
      The part of a node that receives the messages of a topic, made with ``create_subscription()``. Its callback runs once for each message while the node spins. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Timer (ROS 2)
      An object made with ``create_timer(period, callback)`` that calls its callback once per period, while the node spins. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Topic
      A named channel that carries messages from publishers to subscribers. There may be zero or more of each, and a subscriber receives only from publishers of its own message type. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Underlay
      A ROS 2 installation or workspace sourced first, which another workspace builds on, such as ``/opt/ros/jazzy``. (:doc:`L8 </lectures/lecture8/l8_lecture>`)

   Workspace
      A folder for ROS 2 development, with the packages in ``src``. ``colcon build`` adds ``build``, ``install``, and ``log`` next to it. (:doc:`L8 </lectures/lecture8/l8_lecture>`)


.. _g-l9:

L9: Launch Files, Namespaces, Remapping, and Executors
======================================================

.. glossary::

   Callback group
      A set of a node's callbacks that tells a multi-threaded executor which may run at the same time: a mutually exclusive group runs one at a time, a reentrant group has no limit. A node's default group is mutually exclusive. (:doc:`L9 </lectures/lecture9/l9_lecture>`)

   Executor
      The object that runs a node's callbacks: a ``SingleThreadedExecutor`` in one thread, a ``MultiThreadedExecutor`` in a pool of threads. ``rclpy.spin()`` uses a single-threaded one. (:doc:`L9 </lectures/lecture9/l9_lecture>`)

   Global interpreter lock (GIL)
      The lock in CPython that lets only one thread run Python bytecode at a time, so threads that compute in Python take turns. (:doc:`L9 </lectures/lecture9/l9_lecture>`)

   Launch argument
      A value given to a launch file as ``name:=value``, declared with ``DeclareLaunchArgument`` and read with ``LaunchConfiguration``. (:doc:`L9 </lectures/lecture9/l9_lecture>`)

   Launch file
      A file, in this course a Python module with ``generate_launch_description()``, that starts many nodes with one ``ros2 launch`` command, with their names, namespaces, remappings, and parameters. (:doc:`L9 </lectures/lecture9/l9_lecture>`)

   Namespace (ROS 2)
      A prefix, such as ``/robot1``, put in front of every relative name of a node: its name, its topics, and its services. Set with ``-r __ns:=/robot1`` or ``namespace=`` in a launch file. (:doc:`L9 </lectures/lecture9/l9_lecture>`)

   Parameter (ROS 2)
      A named setting that belongs to one node, found under the node's full name. ``-p name:=value`` sets its value when the node starts. (:doc:`L9 </lectures/lecture9/l9_lecture>`)

   Remapping
      Replacing a name used in a node's code with another one when the node starts, with a rule ``old:=new``, so that the code does not change. (:doc:`L9 </lectures/lecture9/l9_lecture>`)

   ROS arguments
      Command-line arguments after ``--ros-args`` that configure a node when it starts: ``-r`` for a remapping rule, ``-p`` for a parameter. ``rclpy.init()`` reads them from ``sys.argv``. (:doc:`L9 </lectures/lecture9/l9_lecture>`)

   Substitution (launch)
      A value in a launch file that is worked out only when the launch runs, such as ``LaunchConfiguration('robot')`` or ``FindPackageShare('pkg')``. (:doc:`L9 </lectures/lecture9/l9_lecture>`)

   Thread
      One sequence of steps inside a process. A process can run several threads at once; they share its memory. In CPython, their Python code takes turns (see Global interpreter lock). (:doc:`L9 </lectures/lecture9/l9_lecture>`)


.. _g-l10:

L10: Parameters, Custom Interfaces, Services, and Actions
=========================================================

.. glossary::

   Action (ROS 2)
      A long task that one node asks another to do: the client sends a goal, the server sends feedback while it works and one result at the end, and the client may cancel the goal. (:doc:`L10 </lectures/lecture10/l10_lecture>`)

   Deadlock
      A state in which a thread waits for something that can only happen on that same thread, so it waits forever. ``call()`` in a callback of a single-threaded executor deadlocks. (:doc:`L10 </lectures/lecture10/l10_lecture>`)

   Durability (QoS)
      A QoS policy: with transient local on both sides, a publisher keeps its last messages (up to its depth) for subscriptions that start later; volatile keeps none. (:doc:`L10 </lectures/lecture10/l10_lecture>`)

   Future (rclpy)
      An object returned by ``call_async()`` or ``send_goal_async()`` that holds the reply once it has arrived; ``add_done_callback()`` runs a function then. (:doc:`L10 </lectures/lecture10/l10_lecture>`)

   Goal handle
      The object that stands for one goal of an action: the server reads the goal and sends feedback through it; the client asks for the result or a cancel through it. (:doc:`L10 </lectures/lecture10/l10_lecture>`)

   Interface (ROS 2)
      A message (``.msg``), service (``.srv``), or action (``.action``) type, defined in a text file from which colcon generates a class for each language. (:doc:`L10 </lectures/lecture10/l10_lecture>`)

   Interface package
      An ament_cmake package that holds ``.msg``, ``.srv``, and ``.action`` files and lists them in ``rosidl_generate_interfaces`` of its ``CMakeLists.txt``. (:doc:`L10 </lectures/lecture10/l10_lecture>`)

   Parameter descriptor
      The facts a node gives about one of its parameters when it declares it: a description, a range of values, and whether it is read-only. ``ros2 param describe`` prints it. (:doc:`L10 </lectures/lecture10/l10_lecture>`)

   Quality of Service (QoS)
      The settings that decide how messages are delivered between a publisher and a subscription, such as reliability, durability, and history depth. A set of these settings is a QoS profile. (:doc:`L10 </lectures/lecture10/l10_lecture>`)

   Reliability (QoS)
      A QoS policy: reliable delivery retries a lost message; best effort may lose it. A reliable subscription does not connect to a best-effort publisher. (:doc:`L10 </lectures/lecture10/l10_lecture>`)

   Service (ROS 2)
      A request and one reply between two nodes: a client sends the request, and the one server of that service name answers it. (:doc:`L10 </lectures/lecture10/l10_lecture>`)


.. _g-l11:

L11: Coordinate Frames, TF2, and Mobile Robot Control
=====================================================

.. glossary::

   Coordinate frame
      An origin and three axes, with a name, such as ``odom``, ``base_link``, or ``base_scan``. A robot's frames form a tree: each has one parent. (:doc:`L11 </lectures/lecture11/l11_lecture>`)

   Differential drive
      A robot with two driven wheels on one axle, steered by their difference in speed. With v the speed and ω the turn rate, the wheels move at v ± ωL/2, L apart. (:doc:`L11 </lectures/lecture11/l11_lecture>`)

   Odometry
      An estimate of a robot's pose from its own motion, such as how far each wheel turned, added up step by step. It drifts, because each step's error stays in the sum. (:doc:`L11 </lectures/lecture11/l11_lecture>`)

   Pose
      Where something is and which way it faces, in a given frame: a position (x, y, z, in meters) and an orientation (a quaternion). ROS 2's message is ``geometry_msgs/msg/Pose``. (:doc:`L11 </lectures/lecture11/l11_lecture>`)

   Proportional controller
      A controller that sends a command equal to a gain times the error, u = k e: a large error gives a large command, no error gives none. (:doc:`L11 </lectures/lecture11/l11_lecture>`)

   Quaternion
      Four numbers (x, y, z, w) of length 1 that describe a turn. In ROS 2, w is last. A turn by ψ about z is (0, 0, sin(ψ/2), cos(ψ/2)); q and −q are the same turn. (:doc:`L11 </lectures/lecture11/l11_lecture>`)

   Simulation time
      The clock of a simulation, published by Gazebo on ``/clock``. A node with ``use_sim_time:=true`` uses it, so that its time stamps match those of the simulated robot. (:doc:`L11 </lectures/lecture11/l11_lecture>`)

   TF2
      The ROS 2 library that keeps the tree of frames over time. Broadcasters publish transforms on ``/tf`` and ``/tf_static``; a listener fills a buffer, and ``lookup_transform()`` answers where one frame is in another. (:doc:`L11 </lectures/lecture11/l11_lecture>`)

   Transform
      The pose of a child frame in its parent frame: a translation and a rotation. Transforms chain along the tree; T_odom,scan = T_odom,footprint T_footprint,scan. (:doc:`L11 </lectures/lecture11/l11_lecture>`)

   Yaw
      A turn about the z axis, counterclockwise positive (REP 103). Roll is a turn about x, pitch a turn about y. (:doc:`L11 </lectures/lecture11/l11_lecture>`)


.. _g-memory:

Reading: Memory
===============

.. glossary::

   Call frame
      The record of one function call in progress: its local namespace and where it returns. Discarded when the call returns. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Cycle collector
      The part of CPython (the ``gc`` module) that frees groups of objects that refer only to each other. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Deep copy
      A copy that also copies the objects inside, made with ``copy.deepcopy()``. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Heap
      The memory CPython manages for itself while the program runs. Every Python object lives there. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Immortal object
      In CPython 3.12, an object that is never freed, such as ``None``; its reference count stays at a fixed large value. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Namespace
      A table that maps names to objects: the module's global names, a call's local names, or the built-in names. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Reference
      How a name, a container slot, or another object points to an object. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Reference count
      The number of references to an object. CPython frees the object when it reaches zero. (:doc:`Memory </reading_material/memory/mem_lecture>`)

   Shallow copy
      A new container whose slots refer to the same objects as the original, made with ``copy.copy()``. (:doc:`Memory </reading_material/memory/mem_lecture>`)
