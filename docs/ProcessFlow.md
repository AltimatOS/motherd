# Process Flow for MotherD

This file describes the process flow of MotherD from kernel hand-off to login 
prompt. This document at this time is written from the viewpoint of using the
Linux Kernel as the core of the operating system, so some behaviours described
are only available on that platform.

## Inside the Initial RAM Disk

The Linux Kernel is designed to allow loading external drivers before storage
is presented by using an initrd, an initial RAM disk, that is backed by either
a filesystem image, or a CPIO archive that contains the drivers and any needed
tooling and libraries needed to load them. After the boot loader loads the
kernel into RAM, it loads the initramfs into another segment of RAM, noting
the location to be passed to the kernel on execution.

At this point the boot loader executes the kernel image and passes in any boot
arguments required to run the system, including the location of the initramfs
in RAM, and exits to hand control of the system to the kernel. After the kernel
starts execution, it loads it's compiled in drivers, including the initramfs
support, and starts up the various internal kernel subsystems for IO, memory
control, and process scheduling. At this point, it expands the CPIO archive
into a special tmpfs mount, and then attempts to load one of the following as
PID 1:

- /sbin/init
- /etc/init
- /bin/init
- /bin/sh

In our case /sbin/init is a symlink to /sbin/initldr. The kernel loads it as
PID 1.

At this point, InitLrd sets itself as the process leader and disables specific
signals from interfering with it's operation, while setting signal handlers for
a couple that ease communication from /sbin/motherd to /sbin/initlrd without
need of DBus or some other IPC mechanism. After it's environment is set as
it desires, it then spawns /sbin/motherd with extra flags to let it know that 
it is executing inside the early userspace phase of boot.

At the beginning of execution, MotherD attempts to open its UNIX Socket to
allow legacy tools to send commands to it. After the socket is opened, MotherD
inspects it's command line flags, noting that it is started in early userspace
which toggles the run mode to load only service units that are single-shot,
which in the initrd are:

- mountd-specials
- eudev-pre-rootfs
- mountd-rootfs-pre-pivot
- mountd-specials-pre-pivot

The mountd-specials single-shot service unit asks mountd to run without going
into the background and to mount a Device Temporary Filesystem on /dev, the 
Linux System Filesystem on /sys, and the Linux Process Filesystem at /proc.

The eudev-pre-rootfs single-shot service unit loads /sbin/eudev in the fore-
ground and to trigger the kernel to populate the various device nodes inside
/dev on the initrd.

Next, the mountd-rootfs-pre-pivot single-shot service runs mountd in the fore-
ground with the flags to discover which volume the root filesystem resides on
by reading the kernel boot arguments from /proc/cmdline, then mounts it at
the /rootfs directory in the initramfs.

After successful completion of executing the single-shot service units, motherd
runs a pivot_root call to force the rootfs namespace to transfer to the volume
mounted at /rootfs, then sends a signal to initldr to exec /sbin/initldr on the
new rootfs once motherd terminates. Then, motherd shuts down to remove any
remaining stale file locks.

Once /sbin/motherd exits, the initldr instance in the initrd runs exec on
/sbin/initldr with no flags to pass PID 1 control to it.

## After the Pivot to the Root Filesystem

The typical AltimatOS installation will run the following daemons to bring up
the core operating system. Note that the various daemons listed other than
InitLdr and MotherD are not supplied by the MotherD project, and live in their
own Git repositories.

```
kernel                 The operating system kernel. In AltimatOS, this is the
|                      Linux kernel
'-> initldr:           A very simple init. It's primary job is to hold PID 1
    |                  and keep the Service Manager running. On non-Linux
    |                  platforms, it must act as process parent for orphaned
    |                  processes and direct descendents. All actions for
    |                  UNIX signals are managed by other tools, and are only
    |                  dispatched
    '-> MotherD:       Service Manager, does not run as PID 1. On Linux, it
        |              uses the `PR_SET_CHILD_SUBREAPER` capability to act as
        |              the parent of any new processes that descend from it,
        |              and any that are orphaned by their parent process
        |-> dbusd:     The D-Bus message bus daemon. The only part of SystemD 
        |              that AltimatOS will utilize. Eventually, this as well
        |              will be re-implemented to simplify system dependencies.
        |              All core services use D-Bus for IPC and will take
        |              advantage of Bus-1 when it is offered from the Kernel
        |-> MountsD:   Filesystems mount manager. All mount requests flow 
        |              through this daemon to manage owner identity on mounts.
        |              It must be started first since it must mount /dev as
        |              devtmpfs for proper operation of `/sbin/eudevd` and
        |              to mount the special filesystems, `/proc`, `/sys`, etc.
        |-> EudevD:    The Gentoo fork of udev that populates the devtmpfs
        |              mounted at `/dev` based on Kernel events. During device
        |              enumeration, MountD will be sent events to mount the
        |              volumes that the site administrator has registered with
        |              it
        |-> PowerD:    The power management control daemon for AltimatOS
        |-> SecurityD: The Security Daemon. This abstracts the various identity
        |              sources that AltimatOS supports. This normally will send
        |              requests to the PAM and NSS layer on behalf of the 
        |              `/bin/login` command and any X11 login manager
        |-> SessionsD: Login session manager. The `/bin/login` command will
        |              send requests to this daemon to manage the information
        |              about a session
        |-> ConsoleD:  The Virtual Console manager. This service spawns and
            |          manages the virtual terminals and running getty 
            |          processes
            '-> getty: The Terminal control command that spawns `/bin/login`
                       after opening the terminal

```

After standing up the standard tooling used inside the initrd, MotherD sends a signal to initldr, which is running as /sbin/init to clean and pivot to the mounted rootfs, which then brings down consoled, eudevd, and mountsd, asks motherd to terminate, then pivot_root is called with exec to /sbin/initldr.

  
