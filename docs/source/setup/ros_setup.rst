====================================================
ROS 2 Development Environment
====================================================

.. _setup-ros:

ROS 2 is needed starting in **Lecture 8 (March 30)**. The course uses:

.. list-table::
   :header-rows: 1
   :widths: 30 70
   :class: compact-table

   * - Software
     - Install guide
   * - ROS 2 Jazzy Jalisco
     - `Ubuntu (deb packages) <https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html>`_
   * - Gazebo Harmonic
     - `Installing Gazebo with ROS <https://gazebosim.org/docs/harmonic/ros_installation/>`_

ROS 2 Jazzy runs on Ubuntu 24.04, with Ubuntu's Python 3.12.

.. note::

   Install everything natively on Ubuntu 24.04. Docker is a fallback for
   one case only: you need Gazebo Classic or ROS 2 Humble for another
   course. The two Gazebo versions cannot be installed on the same
   system.

.. warning::

   **Keep NumPy below version 2.** ROS 2 Jazzy's packages are built on
   Ubuntu 24.04 against its ``python3-numpy`` package, NumPy 1.26.4.
   The NumPy documentation says that packages built against NumPy 1.x
   "will not work with NumPy 2.0"
   (`Depending on NumPy <https://numpy.org/doc/stable/dev/depending_on_numpy.html>`_).
   With NumPy 2.5.4 in the course environment, importing Ubuntu's
   OpenCV fails with ``ImportError: numpy.core.multiarray failed to
   import`` (measured on Ubuntu 24.04 with ROS 2 Jazzy, October 2026).
   The course ``pyproject.toml`` pins ``numpy<2`` for this reason.

.. note::

   Detailed ROS 2 steps are posted before Lecture 8.
