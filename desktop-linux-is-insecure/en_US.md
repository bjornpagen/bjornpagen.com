Desktop Linux distributions, including Debian, Fedora,
Ubuntu, Arch, Gentoo, all suffer from a complete neglect
of basic security principles.

This is totally heterodox to the average Linux enthusiast
dogma. *Everyone knows* that Windows is insecure, right?
Linux is open source™, everyone can look at the code and
fix bugs, right?

## On Security

Really, exploits are just programs written to take
advantage of a bug. To minimize exploits, we minimize bugs.

Software bugs come from complexity. The software stacks
that make up our software necessarily come with layers
upon layers of complexity: they solve *hard* problems,
and require a lot of code to do so. TCP and SSL, for
instance, are highly complex protocols that require
a team effort of engineers to implement correctly.

It is rather reasonable to expect that all our software
will have bugs. The dream of formally verifying everything
is a pipe dream, mostly
because formal verification is slow and costly.

It's no suprise that the server world has, with containers
and deployment frameworks like Kubernetes, totally embraced
the concept of a **sandbox**. The logic goes---if we can't
ensure our software is secure, we will separate it and
quarantine its side effects from the rest of the system.

## App level sandboxing

Let's take an example of Chrome's browser. The GUI,
HTML renderer, V8 JavaScript engine, browser extensions---all
these parts of Chrome individually are heaping behemoths
of code. So, all these components are separated and live in
different operating system processes, and can only speak
to each other via an IPC mechanism.

This way, a rogue website isn't able to access your home
directory, since only the GUI part of Chrome has access.
The GUI (officially called the `Browser` process), is the
only part that has access to your home directory. So, the Renderer
is "Sandboxed".

This doesn't mean you're totally safe though. No sandbox is perfect,
and all it takes is a bug in the separation mechanism for infected code to *escape*
into the rest of the system. If the attacker has figured out how
to exploit the HTML Renderer, they can try to use the IPC mechanism
to exploit the GUI. This is called an exploit **chain**.

This is super common, and all modern exploits involve long chains
of exploits, one after another. You can read about an Android full
exploit chain that mimics the above scenario [almost exactly](https://github.blog/2021-03-24-real-world-exploit-chains-explained/).

## Game over

In the case that a Renderer exploit and GUI exploit are chained
together, you know have a rogue process that can do whatever
it likes as your user. This is game over.

It can install a keylogger,
record your screen, open a root shell, and quietly listen for you
to type in your root password.

At this point, the attacker can even proceed to exploit the kernel,
hack into the firmware, and brick your device, or install a permanent
backdoor onto the device.

At least, this is what it was like 10 years ago. To prevent exploitation from being so easy, most modern desktop operating systems
implement their *own* sandbox, and separate the entire Chrome
app from the rest of the system. This way, extra effort is needed to create
a doomsday scenario.

## Proper Sandboxing

A modern "Linux distribution" that actually does sandboxing
incredibly well is ChromeOS. There are a whole [bible of strategies](https://www.chromium.org/chromium-os/chromiumos-design-docs/system-hardening/)
that ChromeOS implements to keep Chrome in it's own little world.
Among the strategies involve cgroups, namespacing, seccomp, etc...
This technologies basically do what Docker does, but without the overhead
of a virtualized Linux kernel. Chrome cannot see your files unless you give
it explicit permission to do so, nor can it execute other programs, or wipe your hard drive.

Not only is Chrome sandboxed---*every* important system process
is sandboxed in ChromeOS. The system logger, the display server, the
wifi daemon... A lot of architecting has gone into minimizing the
attack surface of these various services by giving them the least
amount of privilege possible to do their job. This is the
**principle of least privilege**.

Each process has it's view of the filestystem stripped away to the
bare minimum it needs to execute. It cannot make new network requests
unless authorized. It cannot access Linux device nodes like `/dev/sda`.
It cannot access other processes information, as it runs as PID 1 in
it's own process tree. It can't even allocate more memory than allowed
by the calling process.

Of course sandboxing isn't *everything*---there are a ton of
different [[mitigations]] and additional details. Mitigations
are mostly useful for memory unsafe languages like C and C++,
making it more difficult for an attacker to develop an exploit
for any given bug.

MacOS, OpenBSD, and even Windows have all made serious progress in sandboxing. ChromeOS
has the best, simply because it is the newest, has been designed securely from the start, and doesn't allow unvirtualized user software.

## Improper Sandboxing

Remember the doomsday scenario I discussed above, where a rogue Chrome
process can take over the whole system? From 10 years ago? The modern Linux
desktop is still right here today.

Here, we find every other desktop Linux distribution. Debian, Fedora,
Ubuntu, Arch, Gentoo, have *zero* meaningful system level sandboxing.

Generally what happens here, is that distros don't implement any sandboxing at all.
They leave it up to the user to configure and put security policies in place,
so by default, if you install `chromium`, you are getting *zero* additional
sandboxing beyond the default Chromium settings. Contrary to ChromeOS,
Chrome can see all your personal files *without* explicit permission to do so.

It's not the maintainers's fault. Maintainers simply don't know
enough about dependencies and what resources a service needs access to. Meanwhile, developers
don't know the contexts in which their software is being run, and need to release portable programs.
Developers therefore can't assume what permissions their program will need with the rest of the system.

Also, Linux desktop software has a very *all-or-nothing* approach to permissions.
Root, or user. You want to install packages? `sudo apt`, `sudo dnf`, `sudo pacman` will serve you just right.
There's really no reason why a system upgrade requires root privileges, especially for installing user software.
[NixOS](https://nixos.org) explores this concept, albeit with an utterly incomprehensible configuration format.

Overuse of sudo really deteriorates the separation between root and user,
usually to zero, since it's not hard at all to install a keylogger, even
[on Wayland](https://github.com/Aishou/wayland-keylogger).

## Potential improvements

Popular approaches to the Linux sandboxing problem are all lacking.

Flatpak forgets about the problem of *system* sandboxing.
Even if you use these technologies, the base distribution of software
that makes up the core OS still is completely unsandboxed, and there are
many criticisms against [Flatpak](https://github.com/flatpak/flatpak/issues/4983)'s
viablility as a security container in the first place.

Although I have no idea of the quality of implementation, systemd service
files support a bunch of namespacing and sandboxing flags declaratively.
No Linux distribution actually seriously uses them very much, and it
certainly isn't as comprehensive as ChromeOS's [minijail](https://github.com/google/minijail).

Firejail runs as root and has paved the way for [privilege escalation bugs](https://www.cvedetails.com/cve/CVE-2019-12499/).

Pipewire was a good solution to the Linux audio problem that theoretically
allows for better sandboxing via a capability oriented design. It helps
pave the work for proper sandboxing to be possible in the future

## New Distribution?

It's interesting to consider a Linux distribution that fixes these problems.
I've thought a little bit about a [[secure linux desktop]]. But alas,
what a waste of time that would be, right?
