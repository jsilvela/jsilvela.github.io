+++
categories = []
date = "2016-11-28T20:41:07+01:00"
tags = []
title = "Linux on a DELL XPS 13 9350"

+++

My preferred Linux distro is Debian, but currently, `stable` won't support the Skybridge chipset. I'll switch to Debian when the new version comes out, but at
the moment, Ubuntu 16.04 is fine.

Installation was easy, after changing a couple of  the UEFI settings. Access them by pressing F2 during the boot sequence.

* Set *SATA Operation* to Disable, or to AHCI, **not** RAID On, which
 came configured on my laptop with Win 10.
* Disable *Secure Boot Enable*.

After changing these settings, I was able to install Ubuntu from the basic image burned to a USB thumb drive.

Only two after-installation nits:

* Though the WiFi card's driver was installed, `network-manager`
 was not able to join networks. I had to uninstall it, and install
 `wicd` instead.

* Ayyy, the HiDPI display ... you can sort of make the desktop work but not uniformly. GNOME has decent basic support.
	* [Tweaking GNOME and browsers](http://www.pcworld.com/article/2911509/how-to-make-linuxs-desktop-look-good-on-high-resolution-displays.html)

All going fine so far, and waking from sleep seems more reliable than on Win 10. Joining WiFi networks is less reliable than on Win 10, though.

Will happily upgrade to Debian 9 when it arrives, meanwhile, this is pretty good.
