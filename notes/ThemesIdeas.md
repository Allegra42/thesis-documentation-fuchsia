# Themes and Ideas





## Zircon Kernel 

see Documentation in Kernel Repo, detailed for most subtopics

### System Calls

syscalls.md

Syscall generation -> abigen tool

https://github.com/fuchsia-mirror/zircon/blob/master/kernel/syscalls/syscalls.cpp

build-x64-clang/gen/global/include/zircon/syscall-kernel-wrappers.inc

### Monitor Context Switches

Linux:  

```sh
$ pid=307
$ grep ctxt /proc/$pid/status
voluntary_ctxt_switches:        41
nonvoluntary_ctxt_switches:     16
```

- pidstat

- getrusage

- tracing? perf?



Fuchsia:

- tracing?  -> insntrace, trace, trace-benchmark, trace-example, traceme?
  - -> insntrace!
  - trace_acquire_context
  - trace_release_context
  - docs/tracing/trace_format.md -> context switch

- cpuperf --list-events
- docs/syscalls/object_get_info.md ?
- docs/safestack.md
  44:is implementing its own kinds of non-local exits or context-switching
  97:implementing something like a non-local exit or context switch.  Such
  102:The context-switch code in the kernel handles switching the unsafe stack
  105:`ZX_TLS_UNSAFE_SP_OFFSET`; `arch_context_switch` copies this into the
  112:New code implementing some new kind of non-local exit or context switch
- docs/ddk/getting_started.md
  193:the kernel must effect at least a context switch operation (if not a data transfer

### Handles and Rights

handles.md

Garbage Collection

### Memory Management

VMARs, mappings, VMOs

Tools: ps, memgraph & treemap.py, vmaps, vmos (k zx vmos hidden), kstats, (k zx ads kernel)

### Kernel Objects

-> IDs, Lifetime, ...

-> Dispatchers

-> Object Security

https://github.com/Allegra42/zircon/tree/master/docs/objects



- IPC
  - Channel
  - Socket
  - FIFO
- Tasks
  - Process - process.md
  - Thread - thread.md
  - Job
  - Task
- Signaling
  - Event
  - Event Pair
  - Futex
- Memory and address space
  - Virtual Memory Object    	zircon/blob/master/docs/vdso.md
  - Virtual Memory Address Region
  - bus_transaction_initiator
- Waiting
  - Port
- Kernel objects for drivers
  - Interrupts
  - Resource
  - Log

### Scheduling

Algorithm, Realtime, Priority Management

https://github.com/Allegra42/zircon/blob/master/docs/kernel_scheduling.md

https://github.com/Allegra42/zircon/blob/master/docs/lockdep-design.md

### Posix Compatibility

libc, Posix (the-book/libc.md) , libfdio (the-book/life\_of\_an\_open.md)

https://github.com/Allegra42/zircon/blob/master/docs/libc.md

zircon/blob/master/docs/program_loading.md



## Driver Development

### Device Protocol (Hooks)

//TODO

zircon/tree/master/system/ulib/ddktl

zxcrypt is written as a DDKTL device driver.  [ulib/ddktl](https://github.com/Allegra42/zircon/blob/master/system/ulib/ddktl) is a C++ framework for writing drivers in Fuchsia.  It allows authors to automatically supply the [ulib/ddk](https://github.com/Allegra42/zircon/blob/master/system/ulib/ddk) function pointers and callbacks by using templatized mix-ins.  In the case of zxcrypt, the [device](https://github.com/Allegra42/zircon/blob/master/system/dev/block/zxcrypt/device.h) is "Ioctlable", "IotxnQueueable", "GetSizable", "Unbindable", and implements the methods listed in DDKTL's [BlockProtocol](https://github.com/Allegra42/zircon/blob/master/system/ulib/ddktl/include/ddktl/protocol/block.h).

There are two small pieces of functionality which cannot be written in DDKTL and C++:

- The driver binding logic, written using the C preprocessor macros of DDK's [binding.h](https://github.com/Allegra42/zircon/blob/master/system/public/zircon/driver/binding.h).
- The completion routines of [ulib/sync](https://github.com/Allegra42/zircon/blob/master/system/ulib/sync), which are used for synchronous I/O and are incompatible with C++ atomics.

## Framework

Core libs, Application model (FIDL), Namespaces, Sandboxing, Story, Module, Agent





#Additional Topics (needed?)

## Storage

Block devs, Filesystems, Ledger



## Networking?



##Graphics



## C++14 in Kernel

https://github.com/Allegra42/zircon/blob/master/docs/cxx.md