====================================================
Changelog
====================================================

All notable changes to the ENPM605 Spring 2027 course documentation are recorded here.


.. dropdown:: v1.2.0: Lecture 2 (2026-10-12)
   :icon: tag
   :class-container: sd-border-success
   :open:

   **Lecture 2**

   - New pages from the L2 deck: lecture, Python exercises, quiz, and
     references. Every program output on them comes from running the
     course code.
   - The lecture covers the ``match`` statement, as the planned topics
     said.
   - The packages section matches the course environment: Python finds
     a package next to the program, in ``.venv``, or through
     ``PYTHONPATH``, which is how ROS 2 makes ``rclpy`` importable.
   - Two new SVG figures: string indices and slicing.
   - The glossary gains eleven Lecture 2 terms.


.. dropdown:: v1.1.0: Lecture 1 and Setup with uv (2026-10-12)
   :icon: tag
   :class-container: sd-border-success

   **Lecture 1**

   - New pages from the L1 deck: lecture, Python exercises, quiz, and
     references. Every program output on them comes from running the
     course code.
   - The glossary gains ten Lecture 1 terms.

   **Setup**

   - The setup pages now describe a native install with uv and Ruff.
     They said course code runs inside Docker containers, which was
     wrong: Docker is only a fallback for students who need Gazebo
     Classic or ROS 2 Humble for another course.
   - The ROS 2 page states the measured NumPy 2 error and the
     ``numpy<2`` pin.

   **Syllabus**

   - New Quizzes and Peer Reviews policy cards.

   **Reading Material**

   - New section, with its first module, **Memory**: where a running
     Python program keeps its data. Reading, exercises, quiz, and
     references; every output comes from running the course code.
     Lecture 1 links to it.

   **Figures**

   - The Lecture 1 diagrams are redrawn as SVG in the names-and-objects
     style the Memory reading uses. The pipeline diagram no longer says
     the interpreter executes "line by line": it executes bytecode,
     instruction by instruction.


.. dropdown:: v1.0.0: Original Version (2026-10-11)
   :icon: tag
   :class-container: sd-border-success

   Initial release of the ENPM605 Spring 2027 course documentation.

   - Course structure: 14 lectures (7 Python, 7 ROS 2), 2 individual
     assignments, 2 team projects, a final project, and 5 quizzes.
   - Syllabus with the 15-week schedule, February 2 to May 11, 2027,
     with Spring Break on March 16.
   - Setup pages for Ubuntu 24.04, VS Code with Docker, and ROS 2 Jazzy
     with Gazebo Harmonic.
   - A page for each lecture with its date and planned topics.
