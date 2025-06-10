# Convex_Hull_Implementation
## Introduction
In computational geometry, the convex hull is defined as the smallest convex polygon that can enclose a given set of points. It forms the boundary that wraps around all the given points such that every line segment between any two points in the polygon lies entirely inside or on the polygon. This project focuses on implementing the Convex Hull using the Jarvis March Algorithm (a.k.a. Gift Wrapping Algorithm) entirely in hardware.

The objective is to demonstrate that even geometric and algorithmic problems, which are typically solved in software, can be systematically and efficiently solved at the hardware level using basic digital design principles such as ROMs, multiplexers, counters, FSMs, comparators, and arithmetic units.

## High-Level Algorithm Overview
The Jarvis March algorithm operates by identifying the leftmost point and then wrapping the set of points in an anti-clockwise direction by repeatedly selecting the next point on the hull based on the orientation of triplets of points.

At each step, the algorithm:

1. Starts from the leftmost point.

2. Scans all other points to find the one that makes the most counterclockwise turn.

3. Repeats this process until it returns to the starting point, thereby completing the hull.

4. In our hardware implementation, we translate each of these logical steps into dedicated subcircuits, carefully synchronizing data flow using FSMs and delays to ensure accurate timing and data validity.

## Hardware Architecture Overview
The system is composed of the following core components:

1. ROM – Stores point data (index, x-coordinate, y-coordinate).
2. Multiplexers – Select which point is read from ROM.
3. Registers – Temporarily store point data for calculations.
4. Counters – Control ROM traversal and module activity.
5. Arithmetic Units – Compute differences, orientations, and perform multiplications.
6. Control Logic – Handles updates, data loading, and stopping conditions.
   
## Detailed Module Descriptions
### 1. ROM Access and Data Synchronization
All the coordinates are stored in ROM, which is read using two 5-bit MUXs to select the required addresses. The addresses depend on the phase:

Finding the leftmost point, or
Determining orientation among points.

Each point comprises three values (index, x, y), and since these are read sequentially, synchronization delays are introduced:
A 2-cycle delay for the point number.
A 1-cycle delay for the x-coordinate.
This ensures that all three components (point number, x, y) are available simultaneously for use in calculations.

### 2. Finding the Leftmost Point
Finding the initial point on the convex hull involves scanning all points to find the one with the minimum x-coordinate. If two points share the same x-coordinate, the one with the minimum y-coordinate is selected.

Three registers MIN_X, MIN_Y, and MIN_POINT are used to store the current best candidate.
These registers are initialized to the maximum possible value (e.g., 4'b1111) to ensure that any valid point from ROM will replace the initial value.

The traversal is completed when a counter reaches 3*N (where N is the number of points), as each point has three entries (index, x, y). Once this process finishes, the system switches to orientation calculation mode.

### 3. Orientation Computation
To determine the correct next point on the hull, we need to compare orientations using three points:

(x0, y0) – The current point on the convex hull.

(x', y') – A candidate point to be added to the hull.

(x”, y”) – A point being tested.

The orientation formula used is:

Orientation = (y' - y0)*(x” - x') - (y” - y')*(x' - x0)
Depending on the sign of the result:

-Positive → Clockwise
-Negative → Counter-clockwise (preferred direction for the hull)
-Zero → Collinear

This is computed using two subcircuits:

TERM1: (y' - y0)*(x” - x')
TERM2: (y” - y')*(x' - x0)

The subtraction operations are handled using a 4-bit 2’s complement unit, and multiplication is done via a subcircuit that outputs the product and sign. A set of comparators and XOR gates determine the sign and magnitude relationships of TERM1 and TERM2 to compute orientation accurately.

### 4. Handling Collinear Points
When points are collinear (Orientation = 0), we update (x', y') only if (x”, y”) is farther from (x0, y0) than (x', y'). This ensures the hull contains only its vertices.

This is achieved through:

A COLLINEAR CASE subcircuit, which uses magnitude comparators to determine which point is farther based on the following rules:

If x” > x0, then choose the one with the greater x.
If x” < x0, then choose the one with the lesser x.
If x” = x0, compare y-values accordingly.

This logic ensures that even in the event of collinearity, the correct boundary point is selected for the convex hull.

### 5. Finite State Machine (FSM) and Decode Logic
An FSM circuit implements a modulo-3 cycle that enables loading of:

Point number (mod 0)
x-coordinate (mod 1)
y-coordinate (mod 2)

This helps in properly loading the three components of each point in sync with the ROM traversal counter. The FSM is activated only when the counter is non-zero and enables data collection every three clock cycles.

### 6. Counter Logic
Two counters, COUNT and COUNT2, manage different phases of the algorithm:

COUNT: Traverses all points once to find the leftmost point.
COUNT2: Used repeatedly to perform orientation calculations during each iteration of the hull wrapping.

An auxiliary logic computes 3*N by left-shifting N (multiply by 2) and adding N again, used to determine when a full pass has completed.

### 7. Update Logic
There are several carefully orchestrated update conditions:

update_x’: When a better candidate point is found based on orientation or collinear distance.

up: Used to initialize or update (x0, y0).

enn, enot: Various enable signals used to control when registers should be written.

These ensure only the correct points are captured as vertices of the convex hull and avoid redundant updates.

### 8. ADDITION Subcircuit
A subcircuit accumulates the sum of point numbers that are part of the convex hull. Every time (x0, y0) is updated, the corresponding point index is added to a register. This can be useful for validation, debugging, or extending functionality (e.g., output ordering or performance analysis).

### 9. Stopping Condition
The process continues until the point (x0, y0) becomes equal to the initial leftmost point, indicating the convex hull has been fully wrapped.
A comparator monitors this condition, and once satisfied, halts the system, signifying completion.

## Nomenclature Summary
Symbol	Meaning
(x0, y0)	Current hull point (reference point)
(x’, y’)	Current best candidate for the next hull vertex
(x”, y”)	Point under examination
Orientation	Determines turn direction using three points


Given the complexity and depth of this project—from orientation logic to precise synchronization across multiple subcircuits—a detailed write-up has been created to supplement this README. It includes circuit-level explanations, design justifications, and elaborated working of all custom components.

Please refer to convexhull_writeup.pdf for deeper technical insights and complete breakdowns of the internal circuitry.




