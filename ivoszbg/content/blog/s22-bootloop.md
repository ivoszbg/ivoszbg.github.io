+++
title = 'Finding the reason behind the infamous S22 boot loops'
date = 2025-01-12T20:10:30+02:00
ShowToc = "true"
type = "post"
+++
### Issues
As some of you may know, there's been quite a lot of issues with the S22 lineup - from screen defects appearing out of nowhere to phones suddenly starting to crash and boot loop, with close to no information regarding the issues apart from posts here and there. If you don't want to read about specifics of how I found the problem and how I fixed it (in a way that's not not preferable), skip to the end of the post for a conclusion.

**Please not that every device is different. What I'm describing here is my experience with my unit, yours may be different and could very well have a completely different issue.**

Well, I've been focusing on mainline Linux kernel and driver development for Exynos devices in the past year or two and recently a friend of mine was kind enough to give me a small budget to get rid of the S8 I was working on and buy another device to work on. So, I went ahead and bought a boot looping Samsung Galaxy S22+ thinking it just had a software issue which would be fixed by flashing the stock firmware. And oh boy, was I wrong.

### Investigating
Initially, I got the phone in a state where it was boot looping. This made me wonder - was the issue I was experiencing a software problem or a hardware? My first thought was to reset it to factory settings. For that I needed to either boot into download mode or reset it from the recovery. However, it wasn't even booting into recovery - it was consistently rebooting 2-3 seconds after the "Samsung Galaxy" logo was appearing. I went out and restored it with Odin via the download mode and the boot looping issue was still present.

So, the issue was definitely hardware related. UFS seemed to work fine as restoring it via Odin gave no errors. So we're left with a few options - an issue with the SoC, the PMIC or some malfunctioning external component that could be causing early kernel panics like the camera. However, in order to find out what the issue is, I needed to be able to boot into android and unlock the bootloader.

I don't know how many of you know about this, but massively the Nexus 5X phones had a defect with the SoC silicon, apparently due to issues with the perf cluster, where it would also reboot loop. Some people fixed it by reflowing/reballing the SoC. The other solution around this was to flash a modified kernel and device tree that disable the performance cluster. Well, there existed this temporary method of getting the device alive for a few minutes by putting it in the freezer. Knowing that, I thought it'd be a good idea to try the same and see what happens. I put my S22+ the fridge for 15 minutes and took it out. Guess what - it booted straight into OneUI! Well, obviously the issue was gonna appear again so I had to act quick. I unlocked the bootloader and started stress testing the phone with benchmarks while looking around for recovery files and doing other stuff.

After about 3 hours, the phone suddenly crashed and rebooted. It then booted into android, worked for a minute or two and went into the boot loop again. However, this time I actually had a way to debug the problem.

Some people suggested that the issue is caused by a software change in some OneUI 6.1 update. Since I could get into the download mode, I started experimenting with flashing old versions of TWRP recovery that were meant to be used with older OneUI versions to see if they'll also boot loop. Well, they did, which ruled the issue being a software one out of the way. I wanted to make sure that the initial bootloader (which I will refer to as S-LK) can boot payloads fine, so I quickly got my custom second-stage shim bootloader called uniLoader booting and within 3 hours I had Linux 6.13rc7 booting on the phone with all cores, which was relieving.

In order to get anywhere I needed to get a hold of kernel panic logs. To do that, I needed to get a faulty boot and then a successful one to TWRP recovery. This meant having the phone loop and then getting it to boot by putting it in the fridge. I tried this a few times and managed to get into recovery, with the /proc/last_kmsg in my hands. So, after scrolling around for a minute I saw the following kernel panic:

```
...
[    2.158962] [7:connecting_thre:  417] tuill_hw connecting to /service/tuill_iwd_server
[    2.159916] [2:connecting_thre:  417] tuill_hw connected to /service/tuill_iwd_server
[    2.391611] [7:tz_iwlog_thread:  414] SW> Samsung Secure OS Release Version  built on: 2023-12-20 13:41:50, binary version: 0f2e6c24
[    2.391614] [7:tz_iwlog_thread:  414] SW> ERR: [tuill_osdrv][14]: cannot obtain client credentials: 22
[    2.392209]I[0:  kworker/u16:3:  435] SError Interrupt on CPU0, code 0xbe000011 -- SError
[    2.392219]I[0:  kworker/u16:3:  435] CPU: 0 PID: 435 Comm: kworker/u16:3 Tainted: G S       C        5.10.223-android12-9-29544049-abS906BXXSDEXKA #1
[    2.392221]I[0:  kworker/u16:3:  435] Hardware name: Samsung G0S board based on S5E9925 (DT)
[    2.392222]I[0:  kworker/u16:3:  435] Workqueue: events_unbound deferred_probe_work_func
[    2.392226]I[0:  kworker/u16:3:  435] pstate: 22400085 (nzCv daIf +PAN -UAO +TCO BTYPE=--)
[    2.392228]I[0:  kworker/u16:3:  435] pc : el1_irq+0x88/0x1c0
[    2.392229]I[0:  kworker/u16:3:  435] lr : exynos_pcie_rc_pcie_phy_config+0xeb4/0x5120 [pcie_exynosS5E9925_rc_cal]
[    2.392230]I[0:  kworker/u16:3:  435] sp : ffffffc0157fb7a0
[    2.392232]I[0:  kworker/u16:3:  435] x29: ffffffc0157fb8d0 x28: ffffff890d048000 
[    2.392235]I[0:  kworker/u16:3:  435] x27: ffffffc00d0e2000 x26: ffffffc00170a000 
[    2.392238]I[0:  kworker/u16:3:  435] x25: 00000000000700d5 x24: 000000000000001e 
[    2.392241]I[0:  kworker/u16:3:  435] x23: 0000000020c00005 x22: ffffffc0016de024 
[    2.392244]I[0:  kworker/u16:3:  435] x21: ffffffc0157fb9f0 x20: 0000007fffffffff 
[    2.392247]I[0:  kworker/u16:3:  435] x19: 00000000000003e8 x18: ffffffc00d085018 
[    2.392249]I[0:  kworker/u16:3:  435] x17: 00006d726f667461 x16: ffffffc015838000 
[    2.392252]I[0:  kworker/u16:3:  435] x15: 00000000000640a5 x14: 0000000000000001 
[    2.392255]I[0:  kworker/u16:3:  435] x13: ffffffc015838124 x12: ffffffc015838174 
[    2.392258]I[0:  kworker/u16:3:  435] x11: ffffffc015838100 x10: 0000000006666998 
[    2.392261]I[0:  kworker/u16:3:  435] x9 : ffffffc008bc4904 x8 : 0000000000000000 
[    2.392264]I[0:  kworker/u16:3:  435] x7 : 4c41432065494350 x6 : ffffffc00a108d08 
[    2.392267]I[0:  kworker/u16:3:  435] x5 : ffffffffffffffff x4 : 0000000000000000 
[    2.392270]I[0:  kworker/u16:3:  435] x3 : ffffffc009995855 x2 : 0000000000000000 
[    2.392273]I[0:  kworker/u16:3:  435] x1 : 0000000000000001 x0 : 00000000094c9aaf 
[    2.392276]I[0:  kworker/u16:3:  435] Kernel panic - not syncing: Asynchronous SError Interrupt
...
[    2.392286]I[0:  kworker/u16:3:  435] Call trace:
...
[    2.392296]I[0:  kworker/u16:3:  435]  el1_irq+0x88/0x1c0
[    2.392298]I[0:  kworker/u16:3:  435]  exynos_pcie_rc_pcie_phy_config+0xed0/0x5120 [pcie_exynosS5E9925_rc_cal]
[    2.392299]I[0:  kworker/u16:3:  435]  exynos_pcie_rc_resumed_phydown+0xd4/0x120 [pcie_exynos_rc]
[    2.392301]I[0:  kworker/u16:3:  435]  exynos_pcie_rc_probe+0xe44/0x16d0 [pcie_exynos_rc]
[    2.392302]I[0:  kworker/u16:3:  435]  platform_drv_probe+0x94/0xbc4.2.1.0
```

This made it obvious that the issue was PCIe-related. The stock device tree suggests that there are two PCIe instances - one that is used for WIFI and the other one for modem. The call trace showed that we're dealing with the one that's used for WIFI (``pcie_exynosS5E9925_rc_cal``, the Shannon one has a different driver with slightly different init sequences). Looking into the function exynos_pcie_rc_pcie_phy_config, we were failing early with writes to PHY.

Another log also confirmed that:

```
[    2.367855]I[0:tz_worker_threa:  409]     > Master         : CPU 
[    2.367855]I[0:tz_worker_threa:  409]     > Target         : HSI1_P
[    2.367855]I[0:tz_worker_threa:  409]     > Target Address : 0x113E01B4 
[    2.367855]I[0:tz_worker_threa:  409]     > Type           : WRITE
[    2.367855]I[0:tz_worker_threa:  409]     > Error code     : Timeout error - response timeout in timeout value
```

The target address 0x113E01B4 consists of the MMIO register base + an offset. The closest write to this that I could find without testing the kernel myself is:

```
writel(0x00, phy_base_regs + 0x01B4);
```

So now that I know that PCIe0's PHY is failing, the quick way to fix this was to disable PCIe0 entirely in the device tree. I did that, repackaged the TWRP recovery, flashed it and it worked - android was boot looping while TWRP recovery was working fine! The next step was getting android to boot. After repackaging vendor_boot a few times until S-LK was not complaining about VBMETA issues and cleaning the data and metadata partitions, it booted straight into android. Awesome! Eeexcept that now WIFI doesn't work, only the modem.

### Conclusion

After all this mess, the conclusion is not so simple: there's a multitude of reasons this could be happening:

- An issue with a cold solder joint on some SoC pin
- A power rail that has gone bad
- A speedbin change from an update that causes power issues
- A crack in the substrate to silicon connection
- Or, the worst case - a common errata in Exynos 2200 SoCs.

In my limited opinion, at least with my device, it's a SoC silicon issue. It's hard to say anything without having schematics and further testing with multiple boot looping devices. It'd be best to have Samsung themselves take responsibility for once, as this is very unlikely to be caused by a user error.

As I'm writing this, I'm stress testing the phone to see if another issue besides the PCIe phy failing will appear. I'm planning on documenting this a bit further and making a clean software-side fix, but for that I need to be absolutely sure that other boot looping S22's are experiencing the exact same issue and not some other problem. At some point I'll also release test binaries to flash.
