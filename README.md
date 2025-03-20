# Convex_Hull_Implementation
This project implements a hardware-based Convex Hull computation using Finite State Machines (FSMs) and digital logic components. The design follows the Jarvis March Algorithm, ensuring efficient detection of the convex boundary of a given set of points.

Developed as part of a team effort, this implementation features:
1. A leftmost point detection system using counters and multiplexers to determine the starting point.
2. An orientation calculator with sub-circuits for strict convex hull detection, ensuring collinear points are handled correctly.
3. Modulo-3 FSM synchronization for register control and timing, enabling a fully automated hardware execution of the algorithm.

The circuit was designed and simulated in Proteus
