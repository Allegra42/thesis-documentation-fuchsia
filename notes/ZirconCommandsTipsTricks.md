# Zircon (and Fuchsia) Commands, Tips & Tricks

Commands, Tools, Tips and Tricks to handle working with Zircon and Fuchsia. 
Most commands are related to the Zircon source root. Fuchsia commands are marked and mostly started with `fx`.


## Build

### Zircon

```bash
make -j32 USE_CLANG=true x64	(arm64)
./scripts/buildall -r (-q -c)
./scripts/build-zircon
./scripts/build-zircon-arm64
./scripts/build-zircon-x64 (ENABLE_DRIVER_TRACING=true)
./scripts/build-zircon -a x64 -C -v -d ENABLE_DRIVER_TRACING=true
```

### Fuchsia
```bash
fx set x64 --zircon-arg ENABLE_DRIVER_TRACING=true --packages='garnet/packages/products/devtools,topaz/packages/default'
fx build-zircon
fx build
fx netboot -- driver.tracing.enable=1
```



## Run

### Zircon on Pixelbook
```bash
# Build USB boot stick (Fuchsia tool)
fx mkzedboot /dev/sdb

# Boot into Zedboot (Network needed)

./scripts/netboot-zircon build-x64-clang/
```

### Zircon on HiKey960
```bash
# Boot into Fastboot
  Auto Power up(Switch 1)   closed/ON
  Recovery(Switch 2)        open/OFF
  Fastboot(Switch 3)        closed/ON
  -> unplug all powering cables, reboot
  
# Install Firmware (one time)
./scripts/flash-hikey -f

# Install Zircon
./scripts/flash-hikey

# UART Debugger/Serial Console
  Pin 1 - GND
  Pin 11 - UART TX (HiKey960 --> Host)
  PIN 13 - UART RX (HiKey960 <-- Host)
```

### Fuchsia on Pixelbook
```bash
# Build USB boot stick (Fuchsia tool)
fx mkzedboot /dev/sdb

# Boot into Zedboot (Network needed)

fx netboot
```

## QEMU

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

## GDB

https://github.com/Allegra42/zircon/blob/master/docs/qemu.md


## Commands, Tips and Tricks on a running Zircon Kernel

Zircon commands are also working on Fuchsia, but not all Fuchsia commands are also available with the pure kernel (e.g. `cat`).

### Kernel command *k*
`k` is a kernel internal command providing lots of information and helper utilities. For a complete listing see `k help`.

```bash
# Some useful k subcommands 
k sysreg (arguments?)		# read armv8 system register

k lspci			# enumerate the devices detected in PCIe ECAM space
	pciunplug
	pcireset
	pcirescan
	pciregions
	psci 		# execute PSCI command
	
k thread/threadstats/threadload/threadq

k dw/dh/db 		# display memory in words/halfwords/bytes  -> mw/mh/mb to modify

# OOM commands 
k oom info
k oom start/stop
k oom print
k oom lowmem

# Kernel object diagnosis
k zx vmos hidden	# Dump hidden VMOs
k zx asd kernel		# Dump kernel address space

# Memory manager
k pmm dump/free		# dump/free via physical memory manager 	
k vmm help			# see help for more infos to virtual memory manager
k vm help			
k vm_object dump/dump_pages
```


### Tools for Memory and Resource monitoring

```bash
ps

vmaps <process-koid> (help)

vmos <process-koid>  (help)		# Dumps a process's VMOs to stdout

kstats 	# System stats
	-c CPU stats
	-m Memory stats
	help 	# for more options

# Fuchsia (Host tool)
./scripts/fx shell memgraph -vt | ./scripts/memory/treemap.py > mem.html
```

### Device Manager tools
```bash
dm
dm help
dm reboot
dm shutdown
dm drivers
```

## Tracing

See:
- zircon/docs/ddk/tracing.md
- zircon/docs/tracing
- https://fuchsia.googlesource.com/garnet/+/master/docs/tracing_usage_guide.md (Fuchsia)

The `trace` command relies on Fuchsia!

Needed:
- Compile flag
- Kernel command line setting

### Enable Tracing 
```bash
# Fuchsia Commands

$ fx set $arch --zircon-arg ENABLE_DRIVER_TRACING=true

# To include devtools into the build
$ fx set x64 --zircon-arg ENABLE_DRIVER_TRACING=true --packages='garnet/packages/products/devtools,topaz/packages/default'

$ fx build-zircon
$ fx build
```

### Boot with Tracing enabled
#### Qemu (Fuchsia)
```bash
$ fx run -k -N -c driver.tracing.enable=1
```

#### Pixelbook (Fuchsia)
```bash
$ fx netboot -- driver.tracing.enable=1
```

### Usage
```bash
fuchsia$ trace list-categories
fuchsia$ trace help

fuchsia$ trace record --categories=example:example1
host$ fx cp --to-host /data/trace.json trace.json
```

```bash
# Not working at the moment -> right usage?
host$ fx traceutil record --categories=example:example1
fx traceutil record -target "stole-attic-wish-agony" ?
```

### Actually get Traces on Fuchsia/Zircon

1. run a trace-enabled Zircon kernel
2. build Zircon standalone to get `netcp` as a tool
3. run `trace record (+more args)` on the device
4. use `<zircon-only-install-dir>/build-<arch>/tools/netcp :/data/trace.json trace.json` to copy the trace to the host system
5. use `fx traceutil convert trace.json ` (Fuchsia tool) to convert the trace to HTML 
6. view in Chrome(ium); Firefox has no trace parser included


## Network Log Viewing

- over link local IPv6 UDP
- Zircon and Fuchsia Tool are equivalent

```bash
# Zircon Tool
zirconsrc$ ./tools/loglistener
zirconsrc$ ./build-x64/tools/loglistener | ./scripts/symbolize

#Fuchsia Tools
fuchsiasrc$ loglistener | scripts/colorize_logs
fuchsiasrc$ ./scripts/flog		# Shotcut for the command above
```


## SSH
It is needed to copy the ssh keys to the device while netbooting. 
Not working at the moment/current setup.

`extra_authorized_keys_file=\"$HOME/.ssh/id_rsa.pub\"`

`fx netboot --authorized-keys .ssh/authorized_keys` 

To test: 

`fx ssh`


## Copying files to and from Zircon
See zircon/docs/getting_started.md   

```bash
#netcp is located in the zircon only build (build-<arch>/tools/netcp)

# Copy the file myprogram to Zircon
netcp myprogram :/tmp/myprogram
  
# Copy the file myprogram back to the host
netcp :/tmp/myprogram myprogram
```

## Other useful Zircon/Fuchsia tools

- Generate syscalls: `abigen`
- Get netadd of a running Fuchsia/Zircon device: `./<zirconbuild>/tools/netaddr`
- netls: `./<zirconbuild>/tools/netls`
- netruncmd: `./<zirconbuild>/tools/netruncmd stole-attic-wish-agony dm shutdown`
- Memory Log: `/fuchsia/scripts/memory$ ./log.sh`

- Modify kernel command line: https://fuchsia.googlesource.com/zircon/+/master/docs/kernel_cmdline.md

