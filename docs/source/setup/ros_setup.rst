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

.. warning::

   **Use Ubuntu's NumPy, not one from pip.** ROS 2 Jazzy's packages are
   built on Ubuntu 24.04 against its ``python3-numpy`` package, NumPy
   1.26.4. The NumPy documentation says that packages built against
   NumPy 1.x "will not work with NumPy 2.0"
   (`Depending on NumPy <https://numpy.org/doc/stable/dev/depending_on_numpy.html>`_).
   A NumPy 2 from pip that Python finds first can break the compiled
   ROS 2 packages. Install with ``sudo apt install python3-numpy``. Do
   the same for Matplotlib and pandas: ``python3-matplotlib`` and
   ``python3-pandas``.

.. note::

   The course container for ROS 2 and the steps to use it are posted
   before Lecture 8.
