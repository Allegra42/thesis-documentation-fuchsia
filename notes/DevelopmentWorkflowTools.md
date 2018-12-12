# Development Workflow and Tools for Zircon/Fuchsia



## General Development Setup

### Coding & Analyze

```bash
./scripts/analyze-zircon # arguments? needs whole fuchsia tree?
```

https://github.com/Allegra42/zircon/blob/master/docs/editors.md



## Platforms

### Pixelbook



### HiKey960

https://fuchsia.googlesource.com/zircon/+/master/docs/targets/hikey960.md

- UART: 
  - GND -> GND (Pin 0) on HiKey
  - TX     -> RX (Pin 11) on HiKey
  - RX     -> TX (Pin 13) on HiKey
- Network/Netboot
  - Works just with Smartphone?



## Device Driver Development
- located within `/system/dev/<subsystem>`


### C Driver (DDK)



### C++ Driver (DDKTL)

- binding needs to be implemented in C -> add a `bind.c` file to perform the binding

==TODO==  Binding with MISC Device:

grep -Ril "device_get_protocol(parent(), ZX_PROTOCOL_MISC"

https://fuchsia.googlesource.com/zircon/+/HEAD/system/ulib/ddktl/include/ddktl/protocol/empty-protocol.h ?

### FIDL Driver Interface

- located within `/system/fidl/`
- directory for a driver named usually `<zircon/fuchsia>-<name>`
- includes a `<name>.fidl` and a `rules.mk`

TODO add rules/fidl for example



#### Pitfalls

- `string` in FIDL for drivers needs a defined maximum size -> `string:30`
- `string` as a return value for a call defined using FIDL 
  - Kernel implementation is fine
  - Userspace implementation 

### User Application (within Kernel)



### Questions

- Is Out-of-Tree possible?


