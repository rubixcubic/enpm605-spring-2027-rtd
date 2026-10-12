====================================================
Python Development Environment
====================================================

.. _setup-python:

You write and run course code in **VS Code**, inside **Docker**
containers. A container comes with every dependency already installed,
so everyone in the class works with the same versions.

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
   * - Docker Desktop
     - `Docker Desktop on Ubuntu <https://docs.docker.com/desktop/setup/install/linux/ubuntu/>`_

On Linux, you may need to add your user to the ``docker`` group, then
log out and back in:

.. code-block:: bash

   sudo usermod -aG docker $USER

See `Linux post-installation steps for Docker Engine
<https://docs.docker.com/engine/install/linux-postinstall/>`_.

VS Code Extensions
------------------

- `Python <https://marketplace.visualstudio.com/items?itemName=ms-python.python>`_
  (Microsoft): Python language support.
- `Dev Containers <https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers>`_
  (Microsoft): develop inside a container.
- `Container Tools <https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-containers>`_
  (Microsoft): build and manage containers from VS Code.

See `Developing inside a Container
<https://code.visualstudio.com/docs/devcontainers/containers>`_.

.. note::

   The course container and the steps to open it in VS Code are posted
   before the first lecture.
