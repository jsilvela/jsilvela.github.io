+++
tags = []
categories = []
date = "2019-08-15T15:45:19+02:00"
title = "Linux on the Desktop, n-th try"
+++

I love Linux. I've been using it at home since 1996 (togeter with macs).
How far we've come in 20+ years.

## Desktop environment

I used to be against desktop environments, which I saw as bloated, but since getting
a HiDPI laptop in 2015, I have changed my mind. Having services and DPI
settings taken care of in one place is too nice to pass up.

As of mid-August, I have settled on KDE Plasma. I'm running Debian 10 Buster at home
and at work. \
I find KDE very polished, and in some ways I like the UX better than mac or windows. The
font smoothing and rendering is great. \
In others areas it needs work, and in particlular, it lacks a per-monitor DPI/scale setting.

## Firmware upgrades

As I was having some issues with my home laptop, I looked at its firmware/BIOS, and noticed
the firmware was out of date.

I was about to follow instructions on installation, and I saw references to
[Linux Vendor Firmware Service](https://fwupd.org/), and how it automates
upgrades via the
[`fwupd` daemon](https://wiki.archlinux.org/index.php/Fwupd#Setup_for_UEFI_BIOS_upgrade).

My computer was supported by this magic.
All I had to do was install `fwupd`, get a listing of devices (`fwupdmgr get-devices`),
which listed my BIOS, and unlock it (`fwupdmgr unlock <device-id>`),
then do `fwupdmgr update`.

This is an enormous improvement over messing with FAT32-formatted USB thumb-drives.

## Current issues with wifi

At the moment, my personal laptop, a
[Dell XPS 13 9350](https://wiki.archlinux.org/index.php/Dell_XPS_13_(9350)),
is having problems with the wifi card driver. After some time, becomes unresponsive.

When that happens, this has been happening on `systemd`:

``` log
[...] kernel: brcmfmac: brcmf_msgbuf_get_pktid: Invalid packet id 588 (not in use)
[...] kernel: brcmfmac: brcmf_msgbuf_get_pktid: Invalid packet id 294 (not in use)
[...] kernel: brcmfmac: brcmf_msgbuf_get_pktid: Invalid packet id 579 (not in use)
[...] kernel: brcmfmac: brcmf_msgbuf_get_pktid: Invalid packet id 337 (not in use)
[...] kernel: brcmfmac: brcmf_msgbuf_query_dcmd: Timeout on response for query command
[...] kernel: brcmfmac: brcmf_cfg80211_get_station: GET STA INFO failed, -5
[...] kernel: brcmfmac: brcmf_msgbuf_query_dcmd: Timeout on response for query command
[...] kernel: brcmfmac: brcmf_msgbuf_query_dcmd: Timeout on response for query command
[...] kernel: brcmfmac: brcmf_cfg80211_get_station: GET STA INFO failed, -5
...
...
[...] kernel: brcmfmac: brcmf_msgbuf_tx_ioctl: Failed to reserve space in commonring
[...] kernel: brcmfmac: brcmf_run_escan: error (-12)
[...] kernel: brcmfmac: brcmf_cfg80211_scan: scan error (-12)
[...] kernel: brcmfmac: brcmf_msgbuf_tx_ioctl: Failed to reserve space in commonring
[...] kernel: brcmfmac: brcmf_run_escan: error (-12)
[...] kernel: brcmfmac: brcmf_cfg80211_scan: scan error (-12)
[...] kernel: brcmfmac: brcmf_msgbuf_tx_ioctl: Failed to reserve space in commonring
[...] kernel: brcmfmac: brcmf_run_escan: error (-12)
```

(get the above from `sudo journalctl -k -f | grep -i brcmfmac`)

The issue has been reported already.

- [Archlinux on XPS 13 wifi](https://bbs.archlinux.org/viewtopic.php?id=242382)
- [Linux kernel bug](https://bugzilla.kernel.org/show_bug.cgi?id=201853)
