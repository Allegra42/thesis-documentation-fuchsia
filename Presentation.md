# Presentation 



- Introduction

  -  Monolithic and Microkernel Architectures
    - Operating systems in General
    - Monolithic kernel: one binary in kernelspace, pros and cons of this architecture
    - Microkernel: just the most essential in kernel space, any thing else in userspace, pros and cons (especially reliability, ...)
    - Background from Linux and Zircon
  - Device Driver Theory
    - What's a driver? (tasks and goals)
    - Why is it special? Why not a normal userspace application? (dual mode cpu, priviledged options, virtual addresses)

- Main Part

  - Driver Model in Linux
    - Basic driver structure
      - ioctl
    - Part of the kernel vs. Module (how is a driver loaded, life cycle, ...)
    - Tools (short)
  - Driver Model in Zircon
    - Basic driver structure
      - FIDL
    - Driver Management in Zircon (Coordinator, Driver loading, retries, ...)
    - Tools (short)
  - Case Study
    - Setup
      - Grove LCD
      - HiKey respectively why not a HiKey for Linux?
      - Level shifting (just a few words, see point above)
    - APIs and Driver variants
      - Two drivers, one for each I2C controller
      - One driver for both controllers
      - Zircon specific: C++ drivers (or other languages) (does Linux not allow anything else than C? -> User Driver -> limited)
    - User API

- Conclusion

  - Differences

    - Reliability, Efficiency, .. (driver side, reload module manually (best case) vs coordinator, ...)
    - ioctl vs FIDL interfaces (type safety, well-defined, ...)
    - C vs Cpp 

  - Summary

    


