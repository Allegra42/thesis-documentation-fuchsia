# Development Workflow and Tools for Zircon/Fuchsia



## General Development Setup

### Coding & Analyze

```bash
./scripts/analyze-zircon # arguments? needs whole fuchsia tree?
```

https://github.com/Allegra42/zircon/blob/master/docs/editors.md

```
scripts/clang-fmt <path/to/file>
```



## Platforms

### Pixelbook



### HiKey960

####Reset HiKey
Unbrick HiKey https://github.com/96boards-hikey/tools-images-hikey960

​	Switch state for unfucking HiKey: 
​		1 -> on
​		2 -> on
​		3 -> off

####Switch States
(Meazzine Switch State for double reset methode: Auto(1) -> on, Boot(2) -> off, Ext(3) -> off)

- Flash HiKey (Fastboot Mode)
	Auto (1) -> on
	Boot (2) -> off
	Ext    (3) -> on
	
- Boot Zircon
	Auto (1) -> on
	Boot (2) -> off
	Ext    (3) -> off	

####Linux on HiKey

- Use SD Image from http://snapshots.linaro.org/96boards/hikey/linaro/debian/17/
  - Lastest is not working!
- Flash EFI, SD is booted with priority `recovery-flash-uefi-prebuilt.sh-r -t /dev/ttyUSB1`
  - Modify script to use latest version
- SD is booted via EFI/GRUB, modify grub.cfg to add your own kernel. If you are not able to create an own initram, just instrumentalize the existing one
  - Add devicetree support `devicetree /boot/devicetree.dtb`
- Install kernel modules to `/lib/modules/<kernelversion-name>`
- (Resize rootfs partition of the SD card)
- `i2cdetect -y -r 0`

#### Android on HiKey

https://source.android.com/setup/build/devices

Kernel is LTS 4.9, but on latest patchlevel (at least by building your own)


####Fuchsia on HiKey
https://fuchsia.googlesource.com/zircon/+/master/docs/targets/hikey960.md

####General
- UART: 
  - GND -> GND (Pin 0) on HiKey
  - TX     -> RX (Pin 11) on HiKey
  - RX     -> TX (Pin 13) on HiKey

- I2C0

  The HiKey works internally on a 1.8V level. Even if there is a 5V VCC out, and a 5V I2C on the Meazzine board, the levels that can be measured at SDA and SCL of the I2C are just about 2V. That's not enough for a device operating on 5V to detect proper signal levels. Use the external power supply for the Grove LCD! 

  I2C0 is on Pin 15 (SCL) and 17 (SDA).

- Network/Netboot

  - Works just with Smartphone?

 

## Device Driver Development
- located within `/system/dev/<subsystem>`

### ARM / Platform Dev Development

- Zircon uses board files to describe the actual board design, the location and details of peripherals, ...

  They are located within `zircon/system/dev/board/`

  Matching (Binding) happens via VID, PID and VID. These values are defined in `system/ulib/ddk/include/ddk/platform-defs.h`.

  You can group I2C device definitions (same for other types).

  The actual device definition for binding/matching may consist of various device definitions containing more than one device of a single type.

  -> You can have a single driver for a composed device like the Grove RGB LCD


### C Driver (DDK)

- straight forward, like described in guide
- define a struct (typedef) for private/context data (per device)
  - available on each point via`void* ctx*`
- ZIRCON_DRIVER_BEGIN(name, driverops_name, "zircon", "0.1", num_matching_rules)
  - defines the device on which this driver binds
  - should correspond the VID, PID and DID from board file (or the ones propagated via USB, ACPI, ...)
- driver_ops
  - .version = DRIVER_OPS_VERSION
  - .bind = bind impl
- bind impl
  - equivalent to probe on linux, initialization may run in an own thread 
    - Add device invisible, set to visible when thread finishes successfully
  - device_add(parent, args, device)
    - args contains a pointer to the device (protocol) ops
- device_protocol_ops
  - .version
  - .release
  - -> required
  - possible:
    - .write
    - .read
    - .open
    - .open_at
    - .message (FIDL)
- Implement Ops, FIDL interfaces, ...
- FIDL interfaces
  - .message must me there -> implement with the "dispatch" created by FIDL
  - fidl_ops -> definitions see FIDL generation
    - function ptrs to FIDL implementations
  - FIDL impls 
    - function definitions see generated output, if a function has one or more return types, see generated output for reply signature

### C++ Driver (DDKTL)

- binding needs to be implemented in C -> add a `bind.c` file to perform the binding

  - realization differs in zircon
  - in bind.c:
    - ZIRCON_DRIVER_BEGIN
    - driver_ops
    - function signature (extern) for C bind

- The device/driver is represented as a C++ class

  - inherit from `DeviceType` -> `using DeviceType = ddk::Device<NewClassName, ddk::Readable>;`
  - -> device_(protocol)_ops are inherit via DeviceType!
  - ddk::I2cChannel i2c 
    - non pdev variant -> call its constructor with parent device in class constructor
    - its a private class member
    - pdev variant -> pdev is acquired just like I2C before, pdev is a private class member (ddk::PDev) -> `pdev->GetI2c(0);`
    - GetI2c returns a `std::optional<ddk::I2cChannel>` -> may take advantage of this fact
  - ctx = this (Class Object), other C struct members become (private) class members
  - DdkRead() -> implement/overwrite
  - FIDL
    - overwrite DdkMessage
    - use C interfaces
    - `auto& self = *static_cast<GroveRgbDevice*>(ctx);` -> get `this`

- Bind()

  - ```cpp
    fbl::AllocChecker ac;
    
    auto dev = fbl::make_unique_checked<grove::GroveRgbDevice>(&ac, parent);
    ```

  - `dev->DdkAdd()`

### FIDL Driver Interface

- located within `/system/fidl/`
- directory for a driver named usually `<zircon/fuchsia>-<name>`
- includes a `<name>.fidl` and a `rules.mk`

#### Pitfalls

- `string` in FIDL for drivers needs a defined maximum size -> `string:30`
- `string` as a return value for a call defined using FIDL 
  - Kernel implementation is fine
  - Userspace implementation 

### User Application (within Kernel)



### Questions

- Is Out-of-Tree possible? -> rather no? even deeper nesting is not
- 


