====================================================
Python Development Environment
====================================================

.. _setup-python:

You write and run course code in **VS Code**, in a Python virtual
environment that **uv** creates. Everything is installed natively on
Ubuntu 24.04. Lecture 1 walks through these steps in class.

A **virtual environment** is a folder (``.venv``) with its own Python
packages, separate from Ubuntu's. **uv** is a Python package and project
manager that creates the virtual environment and installs the packages
the course lists in ``pyproject.toml``. :doc:`Lecture 1
</lectures/lecture1/l1_lecture>` explains both, and why Ubuntu needs
them.

Required Software
-----------------

.. list-table::
   :header-rows: 1
   :widths: 30 70
   :class: compact-table

   * - Software
     - Install guide
   * - Visual Studio Code
     - `VS Code on Linux <https://code.visualstudio.com/docs/setup/linux>`_
   * - uv
     - `Installing uv <https://docs.astral.sh/uv/getting-started/installation/>`_

VS Code Extensions
------------------

- `Python <https://marketplace.visualstudio.com/items?itemName=ms-python.python>`_
  (Microsoft): Python language support.
- `Ruff <https://marketplace.visualstudio.com/items?itemName=charliermarsh.ruff>`_
  (Astral Software): linting and formatting in the editor.

Step 1: Install uv
------------------

Install the version the course uses, then open a new terminal and check it:

.. code-block:: bash

   curl -LsSf https://astral.sh/uv/0.13.0/install.sh | sh
   uv --version

Expected output:

.. code-block:: text

   uv 0.13.0 (x86_64-unknown-linux-gnu)

Step 2: Create the Course Environment
-------------------------------------

Clone the `course repository
<https://github.com/rubixcubic/enpm605-spring-2027-python>`_, then from
its root folder:

.. code-block:: bash

   uv venv --python /usr/bin/python3 --system-site-packages
   uv sync

- ``uv venv`` creates ``.venv`` with Ubuntu's Python 3.12.
- ``--system-site-packages`` lets ``.venv`` see Ubuntu's packages. ROS 2
  needs them from Lecture 8.
- ``uv sync`` installs the packages listed in ``pyproject.toml``.

.. warning::

   Run ``uv venv`` first. If you run ``uv sync`` first, uv creates a
   ``.venv`` that cannot see Ubuntu's packages. To fix it, delete
   ``.venv``, then run ``uv venv`` and ``uv sync`` in that order.

Step 3: Keep NumPy Below Version 2
----------------------------------

ROS 2 Jazzy's packages are built on Ubuntu 24.04 against NumPy 1.26.4.
``pyproject.toml`` pins ``numpy<2``. See the warning on the
:ref:`ROS 2 setup page <setup-ros>` for what happens with NumPy 2.

Step 4: Find and Remove Packages That Hide Ubuntu's
---------------------------------------------------

``pip install --user`` puts packages in
``~/.local/lib/python3.12/site-packages``. ``.venv`` sees that folder
too, and a package there hides Ubuntu's copy of the same package. Check
which copy of OpenCV Python loads, and list what pip installed there:

.. code-block:: bash

   uv run python -c "import cv2; print(cv2.__file__)"
   python3 -m pip list --user

If ``cv2`` loads from ``~/.local``, remove it by the name ``pip list``
prints (``opencv-python``, not ``cv2``):

.. code-block:: bash

   python3 -m pip uninstall --break-system-packages opencv-python

If the first command prints ``No module named 'cv2'``, skip this step:
Ubuntu's OpenCV arrives with ROS 2 in Lecture 8. :doc:`Lecture 1
</lectures/lecture1/l1_lecture>` shows the output of each command.

Step 5: Verify the Setup
------------------------

.. code-block:: bash

   uv run lecture1/verify_setup.py

A working setup prints five ``[ OK ]`` lines and ``5 of 5 checks
passed``. See :doc:`Lecture 1 </lectures/lecture1/l1_index>` for the
full output.
