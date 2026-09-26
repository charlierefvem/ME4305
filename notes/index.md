---
title: ME 4305 Course Outline
type: outline
status: draft
source:
  course: ME4305
  term: 2268
  document: ME4305_2268_Syllabus.pdf
cssclasses:
  - clean-syllabus
---

## Documentation

Refer to [[Documentation/index|ME4305 MicroPython Firmware and API Documentation]].

## Lectures

This outline follows the tentative Fall 2026 lecture sequence. Refer to Canvas for the current schedule.

### Module 1 - Hardware Interfacing and Abstraction

1. **Week 1 (August 24-27) - Philosophy of Mechatronics; Python Fundamentals; File I/O in Python**
    1. Topics
        1. [[topic_philosophy_of_mechatronics|Philosophy of Mechatronics]]
        1. [[topic_python_fundamentals|Python Fundamentals]]
        1. [[topic_file_io|File Input and Output]]
2. **Week 2 (August 31-September 3) - Collections and Mutability; Crystals, Oscillators, Timers, and Pulse Width Modulation**
    1. Reading
        1. [[reference_mutability|Mutability]]
    1. Topics
        1. [[topic_collections|Collections]]
        1. [[topic_crystals_timers_and_counters|Crystals, Oscillators, Timers, and Pulse Width Modulation]]
3. **Week 3 (September 7-10) - Labor Day; Quadrature Encoders and Accounting for Reload**
    1. Topics
        1. [[topic_encoders|Encoders and Decoding Quadrature Output]]
4. **Week 4 (September 14-17) - Finite State Machines and State Transition Diagrams; Tenets of Object-Oriented Programming**
    1. Topics
        1. [[topic_fsms_and_state_transition_diagrams|Finite State Machines and State Transition Diagrams]]
        1. [[topic_classes_and_objects|Classes and Objects]]
5. **Week 5 (September 21-24) - Software Timing and Python Generator Functions; Task Diagrams and the Scheduler**
    1. Topics
        1. [[topic_software_timing|Software Timing]]
        1. [[topic_intro_to_generators|Introduction to Generator Functions]]
        1. [[topic_task_diagrams|Task Diagrams]]
        1. [[topic_scheduling_tasks|Scheduling Tasks]]
        1. [[topic_priority_schedulers|Priority Schedulers]]

### Module 2 - Real-Time Mechatronic Systems

6. **Week 6 (September 28-October 1) - Virtual Communication Ports and String Processing; Dynamics of the Romi Robot Platform**
    1. Reading
        1. [[reference_formatting_strings|Formatting Strings]]
    1. Topics
        1. [[topic_serial_communication|Serial Communication]]
        1. [[topic_virtual_com_ports|Virtual Communication Ports]]
        1. [[topic_coop_user_io|Cooperative User Input and Output]]
        1. [[topic_romi_modeling|Romi Dynamic Modeling]]
7. **Week 7 (October 5-8) - Basics of Feedback Control; GPIO Port Pin Circuitry**
    1. Reading
        1. [[reference_PID|PID Controllers]]
        1. [[reference_battery_droop|Battery Droop Compensation]]
    1. Topics
        1. [[topic_intro_to_feedback_and_controls|Introduction to Feedback and Controls]]
        1. [[topic_gpio|General Purpose IO (GPIO)]]
8. **Week 8 (October 12-15) - I2C Introduction; Review of Binary and Hexadecimal Numbers and Sign Extension**
    1. Reading
        1. [[reference_bit_manipulation|Bit Manipulation Techniques]]
        1. [[reference_memoryviews|Buffers and memoryview Objects]]
    1. Topics
        1. [[topic_i2c_communication|I2C Communication]]
        1. [[topic_review_of_signed_numbers|Review of Signed Numbers]]
9. **Week 9 (October 19-22) - Introduction to IMUs, Euler Angles, and Quaternions; Exam 1**
    1. Reading
        1. [[reference_euler_angles|Euler Angles]]
        1. [[reference_quaternions|Quaternions]]
    1. Topics
        1. [[topic_inertial_measurement_units|Inertial Measurement Units (IMUs)]]
10. **Week 10 (October 26-29) - Planning for System Integration; Exam 1 Recap**
    1. Placeholder
        1. Planning for System Integration *(note not yet written)*

### Module 3 - Autonomous Systems and Project Integration

11. **Week 11 (November 2-5) - State Feedback Fundamentals; Discretization Techniques**
    1. Reading
        1. [[reference_z_domain|Discrete Time Systems]]
        1. [[reference_continuous_to_discrete|Continuous to Discrete Conversion]]
        1. [[reference_matrix_exponential|Matrix Exponential]]
    1. Topics
        1. [[topic_intro_to_state_feedback|Introduction to State Feedback]]
        1. [[topic_discrete_PID|Discrete PID Implementation]]
12. **Week 12 (November 9-12) - Observer Fundamentals; Veterans Day Holiday**
    1. Reading
        1. [[reference_observer_design|Observer Design]]
        1. [[reference_disturbance_observer|Disturbance Observer Design]]
    1. Case Study
        1. [[case_motor_observer|Practical Motor Control]]
13. **Week 13 (November 16-19) - Trajectory Planning; Switch Bounce Phenomenon and Debounce Techniques**
    1. Reading
        1. [[reference_switch_bounce|Mechanical Switch Bounce and Methods for Debouncing]]
    1. Topics
        1. [[topic_path_planning|Path Planning and Dynamic Trajectory Generation]]
14. **Week 14 (November 23-26) - Fall Break Holiday**
15. **Week 15 (November 30-December 3) - Project Strategy Student Presentations**
16. **Week 16 (December 7-10) - TBD (Reserved for Lecture Overflow); Exam 2**
