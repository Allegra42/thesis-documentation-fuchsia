# Zircon Commands, Tips & Tricks

-> in zircon sources

###Coding & Analyze

```bash
./scripts/analyze-zircon # arguments? needs whole fuchsia tree?
```

https://github.com/Allegra42/zircon/blob/master/docs/editors.md

### Build

```bash
make -j32 USE_CLANG=true x64	(arm64)
./scripts/buildall -r (-q -c)
./scripts/build-zircon
./scripts/build-zircon-arm64
./scripts/build-zircon-x64 (ENABLE_DRIVER_TRACING=true)
```



### Run

```bash
# Build USB boot stick (Fuchsia tool)
fx mkzedboot /dev/sdb

./scripts/netboot-zircon build-x64-clang/
```



### QEMU

```bash
./scripts/run-zircon
./scripts/run-zircon-arm64
./scripts/run-zircon-x64 

# Symbolize backtraces
./scripts/run-zircon -a (x64|arm64) | ./scripts/symbolize
```

Quite QEMU: `ctrl+a x`

Help `ctrl+a ?`  or `ctrl+a h`

-N flag for network??

### GDB

https://github.com/Allegra42/zircon/blob/master/docs/qemu.md

### Within the running Kernel

```bash
# OOM commands
k oom info
k oom start/stop
k oom print
k oom lowmem

# Dump hidden VMOs
k zx vmos hidden

# Dump kernel address space
k zx asd kernel

# Kernel tests
k ut all


```



Memory/Resource monitoring

```bash
ps

./scripts/fx shell memgraph -vt | ./scripts/memory/treemap.py > mem.html

vmaps

vmos <pid>

kstats -m
```



Userspace

```bash
# Userspace tests
runtests
```





## Booting with tracing

docs/ddk/tracing.md



To be super conservative, not only does tracing currently require a special compile flag to enable it: `ENABLE_DRIVER_TRACING=true`, it also requires an extra kernel command line flag to enable it: `driver.tracing.enable=1`

Example:

First build with driver tracing enabled:

```
$ fx set $arch --zircon-arg ENABLE_DRIVER_TRACING=true
$ fx build-zircon
$ fx build
```

Then boot. With QEMU:

```
$ fx run -k -N -c driver.tracing.enable=1
```

Or on h/w (augment with options specific to your h/w):

```
$ fx serve -- -- driver.tracing.enable=1
```

### Using tracing

Once the system is booted you can collect traces on the target and then manually copy them to your development host. These examples use the category from the source additions described above.

Example:

```
fuchsia$ trace record --categories=example:example1
host$ fx cp --to-host /data/trace.json trace.json
```

However, it's easier to invoke the `traceutil` program on your development host and it will copy the files directly to your host and prepare them for viewing with the Chrome trace viewer.

```
host$ fx traceutil record --categories=example:example1
fx traceutil record -target "stole-attic-wish-agony" ?
```

https://fuchsia.googlesource.com/garnet/+/master/docs/tracing_usage_guide.md



```bash
fx set x64 --zircon-arg ENABLE_DRIVER_TRACING=true --packages='garnet/packages/products/devtools,topaz/packages/default'

fuchsia$ trace list-categories
fuchsia$ trace help
```

traceutil -> stream over tcp



## Network Log Viewing

fxThe default build of Zircon includes a network log service that multicasts the system log over the link local IPv6 UDP. Please note that this is a quick hack and the protocol will certainly change at some point.

For now, if you're running Zircon on QEMU with the -N flag or running on hardware with a supported ethernet interface (ASIX USB Dongle or Intel Ethernet on NUC), the loglistener tool will observe logs broadcast over the local link:

```
$ZIRCONBUILDDIR/tools/loglistener

zircon$ ./build-x64/tools/loglistener | ./scripts/symbolize

fuchsia tool: loglistener | scripts/colorize_logs

fuchsia/scripts$ ./flog

```

## 





## SSH

`extra_authorized_keys_file=\"$HOME/.ssh/id_rsa.pub\"`

`fx netboot --authorized-keys .ssh/authorized_keys` 



The host and target must be able to communicate over IP. In particular
 it must be possible to SSH from the development host to the target device, and
 the target device must be able to connect via TCP to the development host on
 port 8083. The SSH connection is used to issue commands to the target device.

 The development host will run a simple, static file, HTTP server which makes the
 updates available to the target. This HTTP server is part of the Fuchsia source
 code and built automatically.

 The target is instructed to look for changes on the development host via a
 couple of commands that are run manually. When the update system on the target
 sees these changes it will fetch the new software from the HTTP server running
 on the host. The new software will be available until the target is rebooted.



 ## Copying files to and from Zircon
zircon/docs/getting_started.md   

With local link IPv6 configured, the host tool ./build-ARCH/tools/netcp can be used to copy files.

 ```
netcp is located in the zircon only build (build-<arch>/tools/netcp)

# Copy the file myprogram to Zircon
netcp myprogram :/tmp/myprogram
  
# Copy the file myprogram back to the host
netcp :/tmp/myprogram myprogram
 ```





## Actually get Traces on Fuchsia/Zircon

1. run a trace-enabled Zircon kernel
2. build Zircon standalone to get `netcp` as a tool
3. run `trace record (+more args)` on the device
4. use `<zircon-only-install-dir>/build-<arch>/tools/netcp :/data/trace.json trace.json` to copy the trace to the host system
5. use `fx traceutil convert trace.json ` (Fuchsia tool) to convert the trace to HTML 
6. view in Chrome(ium); Firefox has no trace parser included



## Generate systemcalls

```bash
abigen 
```



## DDK Tools

```bash
usage: banjoc [--ddk-header HEADER_PATH]
              [--ddktl-header HEADER_PATH]
              [--json JSON_PATH]
              [--name LIBRARY_NAME]
              [--files [BANJO_FILE...]...]
              [--help]

 * `--ddk-header HEADER_PATH`. If present, this flag instructs `banjoc` to output
   a C ddk header at the given path.

 * `--ddktl-header HEADER_PATH`. If present, this flag instructs `banjoc` to output
   a C++ ddktl header at the given path.

```





```bash
./netaddr 
fe80::4a65:ee4d:fe1a:495f%eth0

./mkkdtb 
mkkdtb kernfile dtbfile outfile

./netls 
    device stole-attic-wish-agony (fe80::4a65:ee4d:fe1a:495f/2)

./netruncmd stole-attic-wish-agony dm shutdown
```



To disable reboot-on-panic, pass the kernel commandline argument [`kernel.halt-on-panic=true`](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/kernel_cmdline.md#kernel_halt_on_panic_bool).



https://fuchsia.googlesource.com/zircon/+/master/docs/kernel_cmdline.md



/fuchsia/scripts/memory$ ./log.sh