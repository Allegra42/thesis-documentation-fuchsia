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
./scripts/build-zircon-x64
```



### Run

```b
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
```

https://fuchsia.googlesource.com/garnet/+/master/docs/tracing_usage_guide.md



## Network Log Viewing

The default build of Zircon includes a network log service that multicasts the system log over the link local IPv6 UDP. Please note that this is a quick hack and the protocol will certainly change at some point.

For now, if you're running Zircon on QEMU with the -N flag or running on hardware with a supported ethernet interface (ASIX USB Dongle or Intel Ethernet on NUC), the loglistener tool will observe logs broadcast over the local link:

```
$BUILDDIR/tools/loglistener
```

## 