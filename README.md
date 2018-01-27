# The AltimatOS System Initialization Daemon

MotherD is a portable systems initialization daemon that tries to overcome
many of the failures of design that are inherient in other *NIX initialization
daemons, such as SysV Init, Upstart, and SystemD. It is a core foundational
component of AltimatOS. 

## Protecting PID 1

On UNIX and unix-like systems, PID 1 has special behaviour. This process ID is
granted special rights, and in most cases, if it crashes, or is terminated in
anyway, will cause the rest of the operating system to crash. To protect the
runtime of AltimatOS from this kind of failure as much as possible, the actual
service manager and session services are not run out of PID 1, rather MotherD
is split into discrete components: `initldr`, which is run as PID 1, and
`motherd`, and other ancilliary services that are managed by `motherd`.

### The early initialization loader - initldr

The `initldr` part of MotherD has the role of loading the service manager
process, `motherd`, keeping it alive, and re-parenting orphaned processes.
Unlike SysV Init, `initldr` does not manage run levels, nor does it have
concerns about power management. The lightweight aspect of `initldr` means that
there is less in the code to possibly have bugs, resulting in less chance for
it to crash and bring down the core operating system.

### The Service Manager - motherd

The actual service manager that does what most would consider the task of an
init daemon is handled by `motherd`. Its job is to manage service lifecycle,
from start to stop.

### Boot Time Behaviour

During boot of the system, the hand-off will look something like this:

  `Kernel`<br />
  +- `initldr`<br />
  +--- `motherd`

Where `initldr` will remain running as PID 1, and `motherd` runs as a higher
PID number assigned by the underlying Kernel of the operating system and will
also run in the background to deal with marshalling power management concerns,
such as traps from UPS hardware, and user initiated system reboots via Ctrl-
Alt-Delete, etc.

## Supported Operating Systems

While the intention is for MotherD to run on all UNIX and unix-like operating
systems, for immediate needs, it will be written to run on Linux-based systems
for the AltimatOS distribution.

<!--
## Languages Used for the Code

MotherD's base loader, <code>initldr</code>, is written in the Go language,
instead of C or C++ to allow us to limit dependencies at run time. Primarily,
the installed binary is statically built, which means that updating the core
C library or linker shouldn't cause the system to require restarting the low
level initldr binary.

For the service manager, <code>motherd</code>, we've chosen to use Perl for
allowing the development to be rapidly proved out. The other ancilliary 
services, will also be written in Perl. This does mean that early boot time
initialization will require a Perl install with adequate modules required by
the codebase.
-->
