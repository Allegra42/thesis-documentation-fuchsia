# Driver Development - Zircon Driver Development Kit (DDK)



## Device Model

https://fuchsia.googlesource.com/zircon/+/HEAD/docs/ddk/device-model.md

- Drivers are built as ELF shared libraries (DSOs)
- loaded into the Device Host (devhost) process
- Device Manager Process (devmgr) contains the Device Coordinator -> keeps track of drivers and devices, manages discovery, creation and direction of Device Host processes, and maintains the Device Filesystem (devfs) -> mechanism through which userspace services and applications (constrained by namespaces)  gain access to devices



- Protocolls
  - PCI
  - USB
  - Block Core
  - Ethermac
- Protocols are usually in-process interactions between devices in the same devhost, but in cases of driver isolation, they may take place via RPC to a “higher” devhost (via proxy).

- Devices may implement Interfaces, which are RPC protocols that clients (services, applications, etc) use. The base device interface supports POSIX style open/close/read/write IO. Currently, Interfaces are supported via the ioctl operation in the base device interface. In the future, Fuchsia's interface definition language and bindings (FIDL) will be supported.



### Driver Lifecycle

- Device drivers are loaded into devhost processes when it is determined they are needed -> Binding Program (ddk/binding.h)

- ```
  ZIRCON_DRIVER_BEGIN(intel_ethernet, intel_ethernet_driver_ops, "zircon", "0.1", 9)
      BI_ABORT_IF(NE, BIND_PROTOCOL, ZX_PROTOCOL_PCI),
      BI_ABORT_IF(NE, BIND_PCI_VID, 0x8086),
      BI_MATCH_IF(EQ, BIND_PCI_DID, 0x100E), // Qemu
      BI_MATCH_IF(EQ, BIND_PCI_DID, 0x15A3), // Broadwell
      BI_MATCH_IF(EQ, BIND_PCI_DID, 0x1570), // Skylake
      BI_MATCH_IF(EQ, BIND_PCI_DID, 0x1533), // I210 standalone
      BI_MATCH_IF(EQ, BIND_PCI_DID, 0x15b7), // Skull Canyon NUC
      BI_MATCH_IF(EQ, BIND_PCI_DID, 0x15b8), // I219
      BI_MATCH_IF(EQ, BIND_PCI_DID, 0x15d8), // Kaby Lake NUC
  ZIRCON_DRIVER_END(intel_ethernet)
  ```

  The ZIRCON_DRIVER_BEGIN and _END macros include the necessary compiler directives to put the binding program into an ELF NOTE section, allowing it to be inspected by the Device Coordinator without needing to fully load the driver into its process. The second parameter to the _BEGIN macro is a `zx_driver_ops_t` structure pointer (defined by `[ddk/driver.h](../../system/ulib/ddk/include/ddk/driver.h)` which defines the init, bind, create, and release methods.

- `init()` is invoked when a driver is loaded into a Device Host process and allows for any global initialization. Typically none is required. If the `init()` method is implemented and fails, the driver load will fail.

- `bind()` is invoked to offer the driver a device to bind to. The device is one that has matched the bind program the driver has published. If the `bind()` method succeeds, the driver **must** create a new device and add it as a child of the device passed in to the `bind()` method. See Device Lifecycle for more information.

- `create()` is invoked for platform/system bus drivers or proxy drivers. For the vast majority of drivers, this method is not required.

- `release()` is invoked before the driver is unloaded, after all devices it may have created in `bind()` and elsewhere have been destroyed. Currently this method is **never** invoked. Drivers, once loaded, remain loaded for the life of a Device Host process.



### Device Lifecycle

- Within a Device Host process, devices exist as a tree of `zx_device_t` structures which are opaque to the driver

  - created with `device_add()`, which the driver provides a `zx_protocol_device_t` structure

  - methods (function pointers) in  `zx_protocol_device_t`  are the device ops -> see device.h

    - The `device_add()` function creates a new device, adding it as a child to the provided parent device. That parent device **must** be either the device passed in to the `bind()` method of a device driver, or another device which has been created by the same device driver.

      -> created device will be added to the global device filesystem devfs

      -> except: **DEVICE_ADD_INVISIBLE** flag -> will not be accessible via opening its node in devfs until `device_make_visible()` is invoked.

  - When a device's parent is removed, its `unbind()` method is invoked. This signals to the driver that it should start shutting the device down and remove any child devices it has created by calling `device_remove()` on them.

  - The `release()` method is only called after the creating driver has called `device_remove()` on the device, all open instances of that device have been closed, and all children of that device have been removed and released.

    - not valid to refer to the `zx_device_t` for that device after `release()` returns
    - not valid to call any device/protocol methods -> crash



## Device Ops - The Device Protocol

https://fuchsia.googlesource.com/zircon/+/HEAD/docs/ddk/device-ops.md

Device drivers implement a set of hooks (methods) to support the operations that may be done on the devices that they publish.

- version

  - ```
    uint64_t version; 	# <- must be set to DEVICE_OPS_VERSION
    ```

- open

  - ```
    zx_status_t (*open)(void* ctx, zx_device_t** dev_out, uint32_t flags);
    ```

  - called when the device is opened via the devfs, or when an existing open connection is cloned (fd is shared with another process)

  - default -> returns ZX_OK

  - own implementation for access control or to return a new device instance instead

    - -> optional dev_out parameter -> allows a device to create and return a device instance child device -> MUST be created with **DEVICE_ADD_INSTANCE** flag set in the arguments to **device_add()**

- open_at

  - ```
    zx_status_t (*open_at)(void* ctx, zx_device_t** dev_out, const char* path, uint32_t flags);
    ```

  - The open_at hook is called in the event that the open path to the device contains segments after the device name itself. For example, if a device exists as `/dev/misc/foo` and an attempt is made to `open("/dev/misc/foo/bar",...)`, the open_at hook would be invoked with a *path* of `"bar"`.

  - default -> returns **ZX_ERR_NOT_SUPPORTED**

- close

  - ```
    zx_status_t (*close)(void* ctx, uint32_t flags);
    ```

  - called when a connection to a device is closed

  - **Note:** If open or open_at return a **device instance**, the balancing close hook that is called is the close hook on the **instance**, not the parent.

- unbind

  - ```
    void (*unbind)(void* ctx);
    ```

  - called when the parent of this device is being removed (due to hot unplug, fatal error, etc). At the point unbind is called, it is not possible for further open or open_at calls to occur, but io operations, etc may continue until those client connections are closed

  - encourage its client connections to close (cause IO to error out, etc), call **device_remove()** on itself when ready

  - The driver must continue to handle all device hooks until the **release** hook is invoked.

- release

  - ```
    void (*release)(void* ctx);
    ```

  - called after this device has been removed by **device_remove()** and all open client connections have been closed, and all child devices have been removed and released

  - driver will not receive any further calls and absolutely must not use the underlying **zx_device_t** once this method returns

  - must free all memory and  release all resources related to this device before returning

- read

  - ```
    zx_status_t (*read)(void* ctx, void* buf, size_t count,
                        zx_off_t off, size_t* actual);
    ```

  - non-blocking read operation **must not block**

  - success *actual* must be set to the number of bytes read (which may be less than the number requested in *count*), and return **ZX_OK**

    - successful read of 0 bytes is generally treated as an End Of File notification by clients

  - no data is available now, **ZX_ERR_SHOULD_WAIT** must be returned and when data becomes available `device_state_set(DEVICE_STATE_READABLE)` may be used to signal waiting clients

- write

  - ```
    zx_status_t (*write)(void* ctx, const void* buf, size_t count,
                         zx_off_t off, size_t* actual);
    ```

  - non-blocking write operation  **must not block**

  - success *actual* must be set to the number of bytes written (which may be less than the number requested in *count*), and **ZX_OK** should be returned

  - not possible to write data at present **ZX_ERR_SHOULD_WAIT** must be returned and when it is again possible to write, `device_state_set(DEVICE_STATE_WRITABLE)` may be used to signal waiting clients

  - default write implementation returns **ZX_ERR_NOT_SUPPORTED**

- get_size

  - ```
    zx_off_t (*get_size)(void* ctx);
    ```

  - If the device is seekable, the get_size hook should return the size of the device

  - it is the offset at which no more reads or writes are possible

  - default -> returns 0

- ioctl

  - ```
    zx_status_t (*ioctl)(void* ctx, uint32_t op,
                         const void* in_buf, size_t in_len,
                         void* out_buf, size_t out_len, size_t* out_actual);
    ```

  - support for device-specific operations

  - must not block

  - on success, **ZX_OK** must be returned and *out_actual* must be set to the number of output bytes provided (0 if none)

  -  default ioctl implementation returns **ZX_ERR_NOT_SUPPORTED**



#### Device State Bits

```
#define DEV_STATE_READABLE DEVICE_SIGNAL_READABLE
#define DEV_STATE_WRITABLE DEVICE_SIGNAL_WRITABLE
#define DEV_STATE_ERROR DEVICE_SIGNAL_ERROR
#define DEV_STATE_HANGUP DEVICE_SIGNAL_HANGUP
#define DEV_STATE_OOB DEVICE_SIGNAL_OOB
```

#### device_state_set

```
void device_state_set(zx_device_t* dev, zx_signals_t stateflag);
```



## Zircon Driver Development 

https://fuchsia.googlesource.com/zircon/+/HEAD/docs/ddk/driver-development.md