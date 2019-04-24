# Fuchsia

- https://fuchsia.miraheze.org/wiki/Main_Page
- https://ci.chromium.org/p/fuchsia/g/garnet/console
- https://fuchsia-dashboard.appspot.com/

## Talks & Texts

- Security in Zepyr and Fuchsia, Stephen Smalley & James Carter, NSA on Linux Security Summit 2018

  - https://youtu.be/Jov4dTnjm2o

  - https://www.linux.com/blog/2018/9/redefining-security-technology-zephyr-and-fuchsia

    

## Normal Media

- https://mgoulao.github.io/fuchsia-web-demo/
- https://www.pcwelt.de/a/fuchsia-das-neue-android-im-ueberblick,3462815
- https://www.googlewatchblog.de/2018/03/fuchsia-googles-geheimwaffe-flutter/
- https://www.googlewatchblog.de/2018/05/fuchsia-android-p-komponenten/
- https://www.googlewatchblog.de/2018/11/fuchsia-android-betriebssystem-unterstuetzung/
- http://binaryinformatics.com/google-fuchsia-operating-system-future/



Fuchsia/Zircon on Huawei (Honor Play):

- <https://www.googlewatchblog.de/2018/11/fuchsia-android-nachfolger-huawei/>
- <https://www.computerbase.de/2018-11/fuchsia-os-honor-play/>



Bluetooth Tests/Show

- <https://www.googlewatchblog.de/2018/11/fuchsia-googles-betriebssystem-testern/>

Fuchsia SDK & Fuchsia as a Part of the AOSP
- <https://www.googlewatchblog.de/2018/12/fuchsia-sdk-entwickler-download/>



### Security in Zepyr and Fuchsia

#### What is Fuchsia

- Microkernel-based OS
- Primarily developed by Google, but open-source
  - Rumored to be replacement for Android and/or ChromeOS
- Targets modern HW (phones, laptops)
  - 64-bit Intel and ARM application processors
- (Object) Capability-based security
- Work in progress

#### Fuchsia: The Zircon Microkernel

- Initially derived from Little Kernel (LK)
  - Embedded Kernel / RTOS similar to FreeRTOS
  - Used in Android bootloader, Trusty TEE
- Extended/rewritten to be a microkernel
  - Support for 64-bit, usermode / process model, object capabilities, IPC, virtualizations, ...
- The only part of Fuchsia, that runs in supervisor mode
  - Drivers, filesystem, network run in usermode!

#### Fuchsia Security Mechanisms

- Microkernel securit primitives
  - (regular) Handles -> object capabilities (zircon)
  - Resource handles -> object capabilities (zircon)
  - Job policy
  - vDSO enforcement (https://www.linuxjournal.com/content/creating-vdso-colonels-other-chicken)
- Userspace mechanisms
  - Namespaces
  - Sandboxing

#### Fuchsia: (Regular) Handles

- Only way (usually) that userspace can access kernel objects
  - They are object capabilities
  - Uses a push model where client creates handles and passes it to a server
- Per-process (like file descriptors) and unforgeable
- Identify both the object and a set of access rights to the object
  - duplicate, transfer, read, write, execute, map, get_property, set_porperty, enumerate, destory, ...
- Can be duplicated with equal or lesser rights (if allows duplicate)
- Can be passed across IPC (if allows transfer)
- Can be used to obtain handles to "child" objects (object_get_chilf) with equal or lesser rights (if allowd enumerate)



- Good:

  - Seperate rights for propagation vs use
  - Seperate rights for different operations
  - Ability to reduce rights through handle duplication

- Of concern:

  - object_get_child() - have a handle to a job, could get a handle to anything in that job or child job
  - Leak of root job handle (e.g. /dev/misc/sysinfo)
  - Refining default rights down to least priviledge
  - Handle propagation and revocation
  - Operations that do not check rights
  - Unimplemented rights

#### Fuchsia: Resource Handles

- Variant of handles for platform resources
  - memory mapped I/O, I/O port, IRQ, hypervisor guests
  - specify allowed resource kind and optionally range
  - "root" resource handle allows access to all resources
- Can be used to obtain more restrictive resource handles
- root resource handle provided to inital process (userboot)




- Good:
  - Supports fine-grained, hierarchical resource restrictions
- Of concern:
  - Coarse granularity of root resource checks
  - Leak of root resource handle (dev/misc/sysinfo)
  - Handle propagation and revocation
  - Refining root resource down to least privilege



#### Fuchsia: Job Policy

- Every process is part of a job
- Jobs can have child jobs (nesting)
  - root job contains all other jobs/processes
- Job policy applied to all processes within the job
  - But can only be set on an empty job (no processes yet)
- Policies inherited from parent and can only be made more restrictive
- Policies include error handling behaviour, object creation, and mapping of WX memory

#### Fuchsia: vDSO Enforcement

https://www.linuxjournal.com/content/creating-vdso-colonels-other-chicken

- Goal: vDSO is the only means for invoking system calls
- vDSO is fully read-only
- vDSO mapping constrained by the kernel
  - Can only occur once per process
  - Must cover entire vDSO
  - Can't be modified/removed/overwritten
- System call entry must occur from expected location in vDSO
- vDSO variants can expose a subset of the system call interface



- Good:
  - Limits kernel attack surface
  - Enforces the use of the public ABI
  - Supports per-process system call restrictions
  - vDSO code is NOT trusted by kernel which fully validates system call arguments
- Of concern:
  - Potential for tampering with or bypassing the vDSO
    - process\_write\_memory()
  - limited flexibility, e.g. as compared to seccomp



#### Fuchsia: Namespaces and Sandboxing

- Namespace is a collection of objects that can be enumerated and accessed by name
  - Composite hierarchy of services, files, devices
- Per component, not global
- Constructed by environment which instantiates a component
- Used and extended by components
- Sandbox is the configuration of a process's namespace created based on its manifest



- Good:
  - No global namespace
  - Object reachability determined by initial namespace
- Of concern:
  - Sandbox only for application packages (and not for system services)
  - Namespace and sandbox granularity
  - No independent validation of sandbox configuration
  - Currently uses global /data and /tmp
    - Docs do mention per-package /data and /tmp (future?)



#### Fuchsia: Bootstrap / Process Creation

- userboot creates devmgr and exits (not like init)
- devmgr creates zircon drivers and services, including svchost
- devmgr creates fuchsia job and appmgr
- svchost provides process creation facility for fuchsia processes
  - But caller must supply all kernel handles for new process
- appmgr provides component creation facility
  - But appmgr is not allowed to create processes (because of the job policy of fuchsia job)
  - Caller identifies component, appmgr constructs namespace based on sandbox, uses svchost to create the actual Zircon process



#### Fuchsia: A Case for MAC (not existing in Fuchsia yet)

- A MAC framework could address gaps left by Fuchsia's existing mechanisms, e.g.
  - Control propagation, support revocation, apply least privilege
  - Support finer-grained resource checks, generalize job policy
  - Validate namespace/sandbox, support finer granularity
- It could also provide a unified framework for defining, enforcing and validating security goals for Fuchsia (as it has in Android)



#### Fuchsia: Back to the Future

- Our early work was in the context of capability-based microkernel operating systems
  - Mach and Fluke
- We've revisited MAC and capabilities repeatedly
  - SELinux & Unix file descriptors
  - SE Darwin & Mach ports
  - Android & Binder



#### Fuchsia & MAC: Design Options

- Entirely userspace, no microkernel support
  - Build on top of existing capability-based mechanism
- Mostly userspace, limited microkernel support
  - Minimalist extensions to capability-based mechanism
- Security policy logic in userspace, full microkernel enforcement for its objects
  - As in our prior work (DTMach, CTOS, Flask, SE Darwin)



#### Full Kernel Support for MAC (like SELinux)

- The Flask security architecture

- Userspace security server provides labeling and access decisions

- Object managers bind lables to objects, enforce security server decisions

  - Both microkernel and userspace servers

- Microkernel provides peer labeling, fine-grained control over transfer and use



####Flask Approach to MAC: Benefits

- Assurable implementation
  - Direct support for labeling and access control in microkernel
  - Capability leak by userspace component can be mitigated by microkernel checks
  - Reduced assurance burden on userspace components
  - Disaggregated TCB - userspace object managers, limited trust in each
- Centralized security policy
  - Amenable to analysis, audit, management
- Support for flexible, fine-grained access control



#### Current Work

- Investigate creation and flows of handles among Fuchsia components
- Analyzing reachability of security-critical handles/objects in the system
- Assessing effectiveness of existing mechanisms
- Exploring options for providing MAC-like properties
- Examples
  - see Slides
  - VMO
  - Resource
  - Channel



#### Fuchsia vs Linux OS security

| Fuchsia                          | Linux                                      |
| -------------------------------- | ------------------------------------------ |
| RO/NX memory protection          | RO/NX memory protection                    |
| Stack depth overflow prevention  | Stack depth overflow prevention            |
| Stack buffer overflow detection  | Stack buffer overflow detection            |
| Kernel and userspace ASLR        | Kernel and userspace ASLR                  |
| Process isolation                | Process isolation                          |
| Self-protection not examined yed | Mitigations for many kernel vulnerabilites |
| Small, decomposed TCB            | Large, monolithic TCB                      |
| Object capabilities              | DAC, MAC                                   |

TCB trusted computing based
