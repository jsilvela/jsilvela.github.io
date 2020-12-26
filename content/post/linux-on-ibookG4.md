+++
categories = []
date = "2017-01-06T21:21:06+01:00"
tags = []
title = "Linux on the iBook G4"

+++

## Installing Debian Jessie on an iBook G4

There is some good information out there, but scattered.  Thought I
might put something together.

### Wireless

The wireless modules are in the `non-free` and `contrib`
repositories.  You'll have to add them to `/etc/apt/sources.list`.

After that, install `firmware-b43-installer`. After reboot, you
should see wireless networks being picked up in the network manger.

References:
[Debian Testing on iBook G4](http://powerpcliberation.blogspot.com.es/2015/04/debian-testing-for-my-ibook-g4.html)

### Battery indicator

Add `pmu_battery` to `/etc/modules` and reload/reboot.  You
may need to tweak your desktop or window manager to get a battery
meter icon displayed.

References:
[Debian Testing on iBook G4](http://powerpcliberation.blogspot.com.es/2015/04/debian-testing-for-my-ibook-g4.html)

### Sound

The biggest pain. Debian came with some default configurations that
need to be unset.

1. Delete or comment out `snd-powermac` from `/etc/modules`.
2. Remove blacklisted lines `snd-aoa*` from
   `/etc/modprobe.d/blacklist.local.conf`.
3. Reload the modules or reboot.
4. Execute `alsamixer`. Select your soundcard, and ensure the PCM
   and Speaker channels have some volume on them, in addition to
   Master.

References:
[Debian mailing list thread](https://lists.debian.org/debian-powerpc/2013/02/msg00007.html)
