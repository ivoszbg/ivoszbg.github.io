+++
title = 'My work'
date = 2024-09-25T18:47:52+03:00
[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true
+++

---
## Linux


The Powkiddy X51 handheld console is relatively hard to develop mainline Linux for, because there are no official kernel sources, nor any kind of documentations for this platform. Both Powkiddy and Actions Semiconductor ignored all my emails, so a lot of things had to be reverse-engineered. Thankfully there also ancient kernel sources for a somewhat similar platform, from where I can borrow information about some registers.

There are still some essential things that have to be brought up. For example:
- due to the lack of information I only managed to write a partial pinctrl driver. It doesn't include any pin groups nor funcs, only gpio.
- the SDMMC might be a bit difficult to bring up. Device trees suggest that ATM7051 isn't using DMA for that, which is a bit... weird. The upstream driver only has DMA support.
- there's absolutely no nand drivers for Actions SoCs. Anywhere. And I'm not gonna put in the effort to reverse engineer it for such a weak platform.
- the GPU driver isn't open sourced. We can use proprietary blobs with hacks, but that'll never land in mainline.
- the keys aren't wired through GPIO, but the pmic. That's bad since we only have regulators functionality for the PMIC upstream.

![alt](/images/bringing-mainline-linux-to-life-on-atm7051-handhelds-v0-rp5vjhpag0ld1.webp)

---

The Samsung Galaxy Xcover 3 Value edition, alongside the other devices with this SoC, have reached a point where they are close to being usable on my 6.11 kernel fork, with important hardware blocks like the Display engine, GPU, touchscreen and WIFI/BT working. Nevertheless it still has some issues that need resolving:
- SMP isn't fully working and heavy wayland DEs like phosh are a bit unstable.
- the DWC2 usb is.. weird.

It still has to undergo upstreaming. Here's some pictures [and a short video showcasing hardware accelerated phosh.](https://i.imgur.com/fyVvY6o.mp4)

{{< twocolumn 
    "![alt](/images/xcover3ve_2.jpg)" 
    "![alt](/images/xcover3ve_1.jpg)" >}}

---

The Samsung Galaxy S8 is definitely not an easy platform to work on. As a high-end exynos SoCs (even though it's from 2017), it has complex peripherals and a large amount of buses like I2Cs, UARTs, HSI2Cs. Some more things need to be brought up, like:
- the SPEEDY bus, which is used for PMIC
- UFS
- DECON
- Mali G71 via Panfrost

{{< twocolumn 
    "![alt](/images/s8_1.jpg)" 
    "" >}}

---

MSM8x12 series SoCs are a cut-off version of MSM8x74. MSM8x12 feature 4 cores and MSM8x10 only 2. The biggest issue currently is that mainline does not have support for MDP3, which means that I'll have to write a driver for it if we want a working panel driver. Meanwhile the Adreno GPU should be doable with a small amount of changes, simpleDRM and a small hack that allows MESA to use simpleDRM.

All progress for {MSM8x10, MSM8x12} has been done under the [Mainline4Lumia project](https://github.com/Mainline4Lumia).

{{< twocolumn 
    "![alt](/images/535_1.jpeg)" 
    "![alt](/images/435_1.jpg)" >}}

---

# uniLoader
[uniLoader](https://github.com/ivoszbg/uniLoader) is a secondary bootloader that is capable of booting the upstream Linux kernel for Android and iOS-based devices. It allows embedding the kernel and DTB to it, making it a very minimalistic bootloader that doesn't require complex hardware drivers for stuff like storage. In fact, even MMU does not get enabled.

The purpose behind it is to provide a small shim for avoiding vendors' bootloader quirks. For example:
  - Exynos devices past 2016 leave decon's framebuffer refreshing disabled right before jumping to kernel, which makes initial debugging efforts when bringing up the platform to upstream linux hard
  - Some S-Boot releases have CNTFRQ_EL0 left unset, which causes issues since the linux timer driver cannot read its value and it SErrors. It either has to be set as a hardcoded frequency property in the DT, or passed via libfdt from uniLoader (the latter is cleaner, but still has to be implemented)
  
The currently supported architectures are ARMV7 and AARCH64. Here's a list of the currently supported devices:
- Apple iPhone 6
- Samsung Galaxy Note 5
- Samsung Galaxy Note 20
- Samsung Galaxy A8 2018
- Samsung Galaxy S6
- Samsung Galaxy S8
- Samsung Galaxy S9
- Samsung Galaxy S20
- Samsung Galaxy J4
- Samsung Galaxy J5 2015
- Samsung Galaxy Tab S6 Lite

---
# U-Boot
In order to avoid the stock BL's limitations, I've gotten u-boot working on few devices:

## iPhone 5
There were many issues. To begin with, S5L8950x either entirely doesn't have VBAR, or Apple's implementation of it is broken, so I had to disable it. Then there were a ton of small issues here and there, which I fixed thankfully due to an emulator that I specifically wrote for S5L8950x, called "Legacy Apple Emulator". Then I had another memory issue, where parsing the framebuffer address via DT was broken, so I had to fix that as well. And finally I brought up USB! - well, partially, since it probably needs some PHY and DWC2 config work. Only USB plug-in detection works for now.
{{< twocolumn 
    "![alt](/images/iphone5_u-boot.jpg)" 
    "![alt](/images/LAEMU.jpg)" >}}

## Xcover 3 Value Edition, Powkiddy X51
{{< twocolumn 
    "![alt](/images/xcover3ve_u-boot.jpg)" 
    "![alt](/images/atm7051_u-boot.png)" >}}
