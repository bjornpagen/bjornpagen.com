## Ideas for a secure Linux Desktop

- Don't need to port full ChromeOS, we just need proper sandboxing
  for Chrome and system daemons (minijail, cgroups, etc...)
- Single user system
- Stateless configuration (a la Clear Linux)
- Read only rootfs by default
- Managed application state/cache in containers
- Mandatory Access Controls
- Uses s6 for init, uses the single user to it's advantage
  by scheduling every single program on the system via s6
- NixOS style filesystem namespacing
- Probably built off of Alpine linux to start

