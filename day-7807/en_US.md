Shanghai: Woke up to some rain today, heard air in NY is shit.

Imported my old [[workout]] from my previous notes. Did
[workout a](workout.html#workout-a).

```pgp
-----BEGIN PGP MESSAGE-----

hQIMAwBlwr2rqfDCAQ//WXxni8vf1LKFZxa6WyRhYoTYeMutJLwDCPNv13lT6tRx
1Hct4JVWNF90z1FiYrJUxA4XyCh8kmW84koMxPgZQPIIg6axNS+jReub6T+m0uij
EP+QITdkErmFe3uNBkQULvL4j5LCAtVG00xcR3mzcYSQ8LEg3Es0HD08XI+GMvBs
RR3WzXU/1TOVvSyrlZXdXGkcq837vSgDsSi5rlIW745FW7SxvS2qO2fpBn9xuZvI
NJYRlnI9zn9rhAhVAyq0Lej8sPavnBD8W4dMko0qRvyvrjXrTIwEcRCTDbyxqqqJ
Dmndi/KsQwS9EdnhMpq+tBaW7sYb49f9O7FdmFJ5g8zvUAhGaEBbfwE3rpbACWwr
ws6g/aZPPIqGby1b/z/pljqx1KQJSiJ0xqSZCnVIU1ZZRakzahz7DijxuaJK9X3r
5O4CG5npOjgaLWPmHOcbXDicvb0d3wEnVRkzwMUVk7lYHHE6TDQS/ijdIjeDujCf
Qs5vnGGor7aShRRrz9oxs0UIdauT3zUsrgb8nDZBKZBde3FuRieHjlMeATVcWIH9
b+IgzmBjBiLXaqYD/ribaZgKnx1WnBUEg6UhkbFnpzEJG4k5tmKZwZGSzxMF7z+d
9c+qb5mL4s+XlrOuS80wlkYMnN01F9xzv2q5gj0yZJkphriiNVMmGQ0uxw4/FwzS
fAEmgM+fWc1Bbm9XP1itwqO3ZD823LHAlcjjqX45qZ6f5bVrGWQlCyvZ5bZL/lPi
JKdGrgfe7DWCkAZle6yeGJBmOAqcIri0+f2vyJ1sSthID+3XuBPo6gbUy79HbKmO
6E4ki35FPmvcCWHd9gtHYbtTpAV0ZW4czdxukQ0=
=Iv+5
-----END PGP MESSAGE-----
```

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
- NixOS style package namespacing
