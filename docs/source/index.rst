ENPM605: Python Applications for Robotics
==========================================

Course Description
------------------
ENPM605 teaches Python programming through robotics applications. The
first half of the course covers the Python language: its core types,
functions, object-oriented programming, and NumPy for the math robots
need. The second half uses Python to program mobile robots with ROS 2,
in simulation with Gazebo.

The course ends with navigation, behavior trees, and a look at how
learned models (vision models and large language models) fit into a
ROS 2 system.

Key topics covered include:

- Python development in VS Code, inside Docker containers
- Python fundamentals: types, operators, control flow, and collections
- Functions: arguments, scopes, type hints, closures, and callbacks
- NumPy and Matplotlib for robotics math and plots
- Object-oriented programming: classes, inheritance, data classes, and protocols
- ROS 2 Jazzy: nodes, topics, launch files, executors, parameters,
  services, and actions
- Coordinate frames, TF2, and mobile robot control in Gazebo Harmonic
- Mapping and navigation with SLAM Toolbox and Nav2
- Behavior trees, lifecycle nodes, and ROS 2 bags
- Learned models in a robot system: perception and language commands

Prerequisites
-------------
Basic programming experience. No prior Python or ROS experience required.

Learning Outcomes
-----------------
After successfully completing this course, you will be able to:

- Set up and use a Python development environment (VS Code, Docker, Git).
- Write correct, readable Python programs with type hints.
- Use NumPy arrays to work with sensor data, poses, and transforms.
- Design object-oriented programs with classes, inheritance, and protocols.
- Build ROS 2 applications in Python with topics, services, actions,
  parameters, and launch files.
- Control a mobile robot in simulation, and use TF2 to reason about
  coordinate frames.
- Map an environment and navigate it with Nav2 and behavior trees.
- Explain where learned models fit in a robot system, and check their
  outputs before the robot acts on them.

.. toctree::
   :hidden:
   :maxdepth: 3
   :titlesonly:

   syllabus/syllabus
   setup/setup
   lectures/index
   assignments/index
   glossary/glossary
   changelog/changelog
