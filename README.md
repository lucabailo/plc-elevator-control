# PLC Elevator Control System

Logic control software for a single elevator serving 7 floors, written in **CODESYS 3.5** (IEC 61131-3).
Three-person exam project for *Control Systems Technologies* (BSc Automation Engineering, University of Bologna, 2022).

The course provided a simulated, uncontrolled elevator plant with a graphical interface. Our task was to design the controller that makes it work correctly and safely.

## Requirements

- Correct **initialisation** (homing) of the cabin at start-up
- **Efficient call handling** from the floor buttons and from inside the cabin:
  - calls made while the elevator is already serving a request are stored and served later
  - calls that match the current direction are served **on the fly** (e.g. moving from floor 2 to 7, a call at floor 5 is served on the way)
- Handling of **non-nominal conditions**, such as sensor faults

## Architecture: Generalized Actuators

The controller follows the **Generalized Actuator (GA)** pattern: each physical actuator is wrapped in a function block with a standard interface (`Do` / `DoWhat` command in, `Done` / `State` / `Fault` out). A supervisory policy decides *what* to do; each GA knows *how* to do it.

| GA | Responsibility | Commands |
|---|---|---|
| `Cabin_GA` | Cabin motion and speed | Up, Down, Homing |
| `Doors_GA` | Cabin doors | Open, Close |
| `HMI_GA` | Floor and cabin buttons, LEDs | Call collection, LED feedback |

Each GA moves through the states **Init → Ready → Busy → Done**, with dedicated **Fault** states. For example, `Cabin_GA` detects faults on the floor (deck) sensors, the ramp sensors that mark deceleration zones, and the limit switches, and raises a corresponding alarm.

## Call-handling policy

- Pending calls are split into an **up list** and a **down list**.
- While the cabin is moving, it keeps serving calls in its current direction (calls above it when going up, below it when going down) before reversing.
- When no calls are left in the current direction, it switches to the other list.
- The policy computes the next destination and passes it to `Cabin_GA`. The GA counts the ramp sensors crossed to know when to slow down and stop at the right floor.

This is a direction-preserving strategy, similar to the classic SCAN ("elevator") algorithm. It meets the on-the-fly requirement without needing an optimal scheduler.

## Implementation

- Sequences in **SFC** (Sequential Function Charts), with actions and transitions written in **Structured Text (ST)**
- Function blocks for each GA, a global variable list for the I/O
- Tested against the provided simulation and visualisation

## Opening the project

1. Install **CODESYS V3.5**.
2. Open `codesys/SingleElevator.project`.
3. Log in to the simulated device (*Online → Simulation*) and start the application.
4. Use the visualisation to place calls and inject faults.

## Tech

CODESYS 3.5 · IEC 61131-3 (SFC, ST) · Generalized Actuator pattern · PLC simulation
