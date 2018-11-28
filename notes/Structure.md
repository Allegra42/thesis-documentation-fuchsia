 # Thesis Structure



## Introduction

- What is Fuchsia? Why is it of interest?
- Research Objectives
  - not included:
    - UI
    - may be the topics marked with ?
  - included:
    - Difference between Micro- and Makrokernel
    - What means a Microkernel for Driver Development
    - -> including the tooling, the workflow and so on

## Modern Operating System Concepts  and Approaches

- Microkernel vs Makrokernel
- Prozesse & Threads
- Scheduling
- Memory Management
- I/O
- System Calls -> POSIX
- Security Concepts (in Kernel) ?
- Filesystem ?

## Case Study: Driver Development in Linux and Zircon

### Linux

- Concepts - specific
- Example Implementation
  - Platform Introduction, Development Setup

### Zircon

- Concepts - specific
  - Processes and Process Handling
    - Tasks - Process, Thread, Job, Task
    - IPC - Channel, Socket, FIFO
    - Signaling - Event, Event Pair, Futex
  - Handles and Rights
  - Memory Management
    - Memory and address space - Virtual Memory Object, Virtual Memory Region, bus_transaction_initiator
  - Scheduling
  - System Calls and POSIX compatibility, Core libs
  - Driver Development
    - Kernel objects for drivers - Interrupts, Resource, Log
    - Device Protocoll (Hooks)
    - DDKTL - framework for writing drivers in fuchsia
- Example Implementation
  - Platform Introduction, Development Setup

### Evaluation

- Results of the Case Studies

## Conclusion

- Resumee

## Outlook

- Outlook

