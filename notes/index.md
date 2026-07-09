---
title: ME 405 Course Outline
cssclasses:
  - clean-syllabus
---

## Lectures

1. Module 1 - Hardware Interfacing and Abstraction
    1. Lecture 1 - Philosophy of Mechatronics and Python Fundamentals
        1. Topics
            * [[topic_philosophy_of_mechatronics|Philosophy of Mechatronics]]
            * [[topic_python_fundamentals|Python Fundamentals]]
    1. Lecture 2 - Collections and File IO in Python
        1. Topics
            * [[topic_collections|Collections]]
            * [[topic_file_io|File I/O]]
    1. Lecture 3 - Lab Hardware Overview and Toolchain Setup
        1. Topics
            * [[topic_hardware_overview|Hardware and Software Toolchain]]
            * [[topic_serial_communication|Serial Communication]]
    1. Lecture 4 - Crystals, Oscillators, Timers, and Pulse Width Modulation
        1. Topics
            * [[topic_crystals_timers_and_counters|Crystals and Timers]]
    1. Lecture 5 - Quadrature Encoders and Accounting for Reload
        1. Topics
            * [[topic_encoders|Encoders and Decoding Quadrature Output]]
    1. Lecture 6 - Finite State Machines and State Transition Diagrams
        1. Topics
            * [[topic_fsms_and_state_transition_diagrams|Finite State Machines and State Transition Diagrams]]
    1. Lecture 7 - Tenets of Object Oriented Programming
        1. Topics
            * [[topic_classes_and_objects|Classes and Objects]]
    1. Lecture 8 - Software Timing and Python Generator Functions
        1. Topics
            * [[topic_software_timing|Software Timing]]
            * [[topic_intro_to_generators|Introduction to Generator Functions]]
    1. Lecture 9 - Task Diagrams and the Scheduler
        1. Topics
            * [[topic_priority_schedulers|Cooperative Multitasking and the Scheduler]]
1. Module 2 - Real-Time Mechatronic Systems
    1. Lecture 10 - Basics of Feedback Control
        1. Reading
            * [[reference_PID|PID Controllers]]
            * [[reference_battery_droop|Battery Droop Compensation]]
        1. Topics
            * [[topic_intro_to_feedback_and_controls|Introduction to Feedback and Controls]]
    1. Lecture 11 - Dynamics of the Romi Robot Platform
        1. Topics
            * [[topic_romi_modeling|Romi Dynamic Modeling]]
    1. Lecture 12 - Virtual Communication Ports and String Processing
        1. Topics
            * [[topic_virtual_com_ports|Virtual Communication Ports]]
            * [[topic_coop_user_io|Cooperative User Input and Output]]
            * [[reference_formatting_strings|Formatting Strings]]
    1. Lecture 13 - GPIO Port Pin Circuitry
        1. Topics
            * [[topic_gpio|General Purpose IO (GPIO)]]
    1. Lecture 14 - I2C Introduction
        1. Topics
            * [[topic_i2c_communication|Introduction to I2C]]
    1. Lecture 15 - Review of Binary and Hexadecimal Numbers and Sign Extension
        1. Reading
            * [[reference_bit_manipulation|Bit Manipulation Techniques]]
            * [[reference_memoryviews|Buffers and memoryview Objects]]
        1. Topics
            * [[topic_review_of_signed_numbers|Review of Signed Numbers]]
    1. Lecture 16 - Introduction to IMUs, Euler Angles, and Quaternions
        1. Reading
            * [[reference_euler_angles|Euler Angles]]
            * [[reference_quaternions|Quaternions]]
        1. Topics
            * [[topic_inertial_measurement_units|Introduction to IMUs]]
    1. Lecture 17 - Planning for System Integration
        1. Topics
            * \[\[Planning for System Integration\]\]
2. Module 3 - Autonomous Systems and Project Integration
    1. Lecture 18 - State Feedback Fundamentals
        1. Topics
            * [[topic_intro_to_state_feedback|Fundamentals of State Feedback]]
    2. Lecture 19 - Discretization Techniques
        1. Reading
            * [[reference_z_domain|Discrete Time Systems]]
            * [[reference_continuous_to_discrete|Continuous to Discrete Conversion]]
        2. Topics
            * [[topic_discrete_PID|Discrete PID Implementation]]
    3. Lecture 20 - Observer Fundamentals
        1. Reading
            * [[reference_observer_design|Observer Design]]
            * [[reference_disturbance_observer|Disturbance Observer Design]]
        2. Topics
            * [[case_motor_observer|Practical Motor Control]]
    4. Lecture 21 - Trajectory Planning
        1. Topics
            * [[topic_path_planning|Path Planning and Dynamic Trajectory Generation]]
    5. Lecture 22 - Switch Bounce Phenomenon and  Debounce Techniques
        1. Reading
            * [[reference_switch_bounce|Mechanical Switch Bounce and Methods for Debouncing]]
        1. Topics
            * \[\[Switch Bounce\]\]
