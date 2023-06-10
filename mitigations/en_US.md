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

Secure DNS.

Capability based architecture: processes are seperated using cgroups, seperate namespacing,
capabilities instead of root, as well as mandatory access control. Principle of least privilege
is extended across the entire OS. Every single system daemon is associated with a minijail command,
like this for `syslog.conf`:

```sh
exec /sbin/minijail0 -l --uts -i -v -e -t -P /var/empty -T static \
    -b / -b /dev,,1 -b /proc \
    -k tmpfs,/run,tmpfs,0xe -b /run/systemd/journal,,1 \
    -k tmpfs,/var,tmpfs,0xe -b /var/log,,1 -b /var/lib/timezone \
    /usr/sbin/rsyslogd -n -f /etc/rsyslog.chromeos -i /tmp/rsyslogd.pid
```

Or like this for `wpa_supplicant.conf`:

```sh
script
  ARGS=""
  case ${WPA_DEBUG} in
    excessive) ARGS='-ddd';;
    msgdump)   ARGS='-dd';;
    debug)     ARGS='-d';;
    info)      ARGS='';;
    warning)   ARGS='-q';;
    error)     ARGS='-qq';;
  esac
  exec minijail0 -u wpa -g wpa -c 3000 -n -i -- \
    /usr/sbin/wpa_supplicant -u -s ${ARGS} -O/run/wpa_supplicant
end script
```

All these services integrate with SELinux.
