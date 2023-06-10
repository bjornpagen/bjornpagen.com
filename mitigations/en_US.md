# ChromeOS

## Core principle - sandboxing

This is done via Mandatory Access Controls, control group device filtering,
chrooting, process namespacing. This is done across the whole operating system.

Toolchain hardening such as NX, ASLR, stack cookies...

State free, read-only root partition (makes persistence very difficult).
User home directories are noexec, nosuid, no device nodes.

Boot security ensures boot integrity, makes sure that no one can comprimise your device
with physical access. This is probably the most important. If anything is changed, the
changes will be detected and the user will know their system has been tampered with.

Secure autoupdate, signed updates sent over SSL, version numbers cannot go backwards,
must be signed by Google's key.

No root, although everyone knows root is bad, the Linux desktop heavily relies on root
for really basic operations.

Secure DNS.
