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
concerns about power management, preferring to only dispatch events based off
the UNIX signals that are passed to it from the Kernel or other operating
system components to other lightweight daemons from other projects.

The lightweight aspect of `initldr` means that there is less logic in the code
to possibly have grave system breaking bugs, resulting in less chance for it
to crash and bring down the core operating system. Additionally, this lean
approach allows for better review of the code for security concerns.

### The Service Manager - motherd

The actual service manager that does what most would consider the task of an
init daemon is handled by `motherd`. Its job is to manage service lifecycle,
from start to stop.

Please take a look at the [docs](https://github.com/AltimatOS/docs/) directory
for more information about how the tools shipped with MotherD work.

## Supported Operating Systems

While the intention is for MotherD to run on all UNIX and unix-like operating
systems, for immediate needs, it will be written to run on Linux-based systems
for the AltimatOS distribution.
