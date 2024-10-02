+++
title = 'My work'
date = 2024-09-25T18:47:52+03:00
ShowToc = "true"
[markup]
  [markup.goldmark]
    [markup.goldmark.renderer]
      unsafe = true
+++

---
# Linux Contributions

List of ARM/ARM64 hardware that I've brought up from the zero.
It might or might not have been merged yet (or is still sitting in my torvalds/linux fork):

### System on Chips (SoCs)

| SoC                 | Component                        | Status/Description |
|---------------------|----------------------------------|---------------------------------------------------------------------------------------------|
| **ATM7051**         | Timer                            | Main block works                                                                                                                                               |
|                     | Multicore                        | All cores work                                                                                                                                                 |
|                     | I2C/UART                         | Works with fixed clocks                                                                                                                                        |
|                     | SIRQ                             | Secondary interrupt controller with support for up to 3 external interrupt lines (e.g., for MMC and PMIC)                                                      |
|                     | SPS                              | Smart Power System for managing CPU-related power gates                                                                                                        |
|                     | Pinctrl                          | Partial support due to limited kernel source availability                                                                                                      |
|                     | DMA                              | Works                                                                                                                                                          |
|                     | Regulators                       | Works                                                                                                                                                          |

This was relatively hard to bring up, because there are no absolutely no sources, nor any kind of documentations for this platform. Both Powkiddy and Actions Semiconductor ignored my emails, so I had to reverse-engineer a lot of things. Thankfully I managed to also find ancient sources for a somewhat similar platform, from where I borrowed information about registers.

There are still some essential things that have to be brought up. For example:
- due to the lack of information I only managed to write a partial pinctrl driver. It doesn't include any pin groups nor funcs, only gpio.
- the sd card mmc might be a bit difficult to bring up. Device trees suggest that ATM7051 isn't using DMA for that, which is a bit... weird. The upstream driver only has DMA support.
- there's absolutely no nand drivers for Actions SoCs. Anywhere. And I'm not gonna put in the effort to reverse engineer it for such a weak platform.
- the GPU driver isn't open sourced. We can use proprietary blobs with hacks, but that'll never land in mainline.
- the keys aren't wired through GPIO, but the pmic. That's bad since we only have regulators functionality for the PMIC upstream.

I wrote a small [reddit post to bring awareness about this.](https://www.reddit.com/r/SBCGaming/comments/1f1o4n2/bringing_mainline_linux_to_life_on_atm7051/)

![alt](/images/bringing-mainline-linux-to-life-on-atm7051-handhelds-v0-rp5vjhpag0ld1.webp)


| SoC                 | Component                        | Status/Description |
|---------------------|----------------------------------|---------------------------------------------------------------------------------------------|
| **Exynos3475**      | Multicore                         | 2 out of 4 cores enabled via SMP                                                                                                                               |
|                     | ChipID                           | Works                                                                                                                                                          |
|                     | MCT (Multi-Core Timer)           | Works                                                                                                                                                          |
|                     | Pinctrl                          | Works                                                                                                                                                          |
|                     | Clock Controller                 | Full support for all blocks                                                                                                                                    |
|                     | HSIC/I2C                         | Works                                                                                                                                                          |
|                     | eMMC                             | Works                                                                                                                                                          |
|                     | MIPI DSIM/CSIM PHY               | Works                                                                                                                                                          |
|                     | Decon                            | Works                                                                                                                                                          |
|                     | Mali T710                        | Enabled via Panfrost                                                                                                                                           |

This has been my biggest project so far. It's reached a somewhat mature point, where devices are usable for media consumption (without audio), with important hardware like Decon, GPU, touchscreen and WIFI/BT working. Nevertheless it still has some issues that need resolving:
- the SMP isn't fully working and heavy wayland DEs like phosh are a bit unstable.
- DWC2 is acting a bit weird, I've gotten the PHY to work but it still not working.

I will also start upstreaming it once most of my exynos8895 patches get merged. Here's some pictures [and a short video showcasing hardware accelerated phosh.](https://i.imgur.com/fyVvY6o.mp4)

{{< twocolumn 
    "![alt](/images/xcover3ve_2.jpg)" 
    "![alt](/images/xcover3ve_1.jpg)" >}}

| SoC                 | Component                        | Status/Description |
|---------------------|----------------------------------|---------------------------------------------------------------------------------------------|
| **Exynos8895**     | Multicore                         | Enabled via PSCI                                                                                                                                                |
|                     | ChipID                           | Works                                                                                                                                                          |
|                     | MCT & ARMv8 Generic Timer        | Both work fine                                                                                      |
|                     | Pinctrl                          | Works                                                                                                                                                          |
|                     | Clock Controller                 | CMU-{TOP, PERIS, PERIC0, PERIC1, FSYS0, FSYS1} supported                                                                                                       |
|                     | HSIC/I2C/SPI                     | Works                                                                                                                                                          |

Exynos8895 is definitely not an easy platform to work on. As one of the high-end modern exynos SoCs (even for 2017), it has complex peripherals and a large amount of buses like I2Cs, UARTs, HSI2Cs. I've already gotten a basic set of peripherals working, which I've gotten applied to the maintainer's tree, so they're gonne be merged in linux-next soon. Here's a list of things I'm working on right now:
- A driver for the new SPEEDY bus, which is used for PMIC.
- Usbdrd3.0 that needs somewhat complex PHY tuning.
- UFS phy.
- Mali G71 via Panfrost

{{< twocolumn 
    "![alt](/images/s8_1.jpg)" 
    "" >}}

| SoC                 | Component                        | Status/Description |
|---------------------|----------------------------------|---------------------------------------------------------------------------------------------|
| **MSM8x12/MSM8x10**  | Multicore                        | Enabled via SMP                                                                                                                                                |
|                     | QGIC2                            | Qualcomm Generic Interrupt Controller works                                                                                                                   |
|                     | ARMv7 Generic Timer              | Works                                                                                                                                                          |
|                     | Pinctrl                          | Works                                                                                                                                                          |
|                     | I2C/UART                         | Works                                                                                                                                                          |
|                     | USB                              | Works                                                                                                                                                          |

This SoC is a cut-off version of the MSM8974. MSM8212 features 4 cores and MSM8610 has 2. The biggest issue currently is that mainline does not have support for MDP3, which means that I'll have to write a driver for it if we want a working panel driver. Meanwhile the Adreno GPU should be doable with a small amount of changes, simpleDRM and a small hack that allows MESA to use simpleDRM.

All progress for {MSM8x10, MSM8x12} has been achieved under the [Mainline4Lumia project](https://github.com/Mainline4Lumia).

{{< twocolumn 
    "![alt](/images/535_1.jpeg)" 
    "![alt](/images/435_1.jpg)" >}}

### Phones

| Phone                          | Component                          | Status/Description                                                                                                                                                    |
|---------------------------------|-------------------------------------|--------------------------------------------------------------------------------------|
| **Samsung Galaxy S8**           | Memory bits                        | Works                                                                                                                                                          |
|                                 | SimpleDRM                          | Works                                                                                                                                                          |
|                                 | GPIO Keys                          | Works                                                                                                                                                          |
| **Samsung Galaxy Xcover 3 VE**  | Simple Panel & Mali T710            | Enabled via Panfrost                                                                                                                                           |
|                                 | Sensors                            | Works                                                                                                                                                          |
|                                 | Cyttsp5 Touchscreen                | Works                                                                                                                                                          |
| **Microsoft Lumia 540**         | SimpleDRM                          | Works                                                                                                                                                          |
|                                 | GPIO Keys                          | Works                                                                                                                                                          |
|                                 | Regulators                         | Works                                                                                                                                                          |
|                                 | Synaptics rmi4-f01 Touchscreen      | Works                                                                                                                                                          |

### Handheld Consoles

| Console              | Component                        | Status/Description                                                                                                                                             |
|----------------------|-----------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Powkiddy X51**      | Memory bits                      | Works                                                                                                                                                          |
|                      | SimpleDRM                        | Works                                                                                                                                                          |


All of this could be found either on my [github page](https://github.com/ivoszbg) or [the linux mail lists](https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/log/?qt=author&q=Ivaylo+Ivanov)

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

# U-Boot Contributions
In order to avoid the stock BL's limitations, I've gotten u-boot working on few devices:

## iPhone 5
There were many issues. To begin with, S5L8950x either entirely doesn't have VBAR, or Apple's implementation of it is broken, so I had to disable it. Then there were a ton of small issues here and there, which I fixed thankfully due to an emulator that I specifically wrote for S5L8950x, called "Legacy Apple Emulator". Then I had another memory issue, where parsing the framebuffer address via DT was broken, so I had to fix that as well. And finally I brought up USB! - well, partially, since it probably needs some PHY and DWC2 config work. Only USB plug-in detection works for now.
{{< twocolumn 
    "![alt](/images/iphone5_u-boot.jpg)" 
    "![alt](/images/LAEMU.jpg)" >}}
## Xcover 3 Value Edition && Powkiddy X51
{{< twocolumn 
    "![alt](/images/xcover3ve_u-boot.jpg)" 
    "![alt](/images/atm7051_u-boot.png)" >}}
{{< imgresize "" "50" "50" >}}
