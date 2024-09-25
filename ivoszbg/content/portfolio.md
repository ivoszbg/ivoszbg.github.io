+++
title = 'Portfolio'
date = 2024-09-25T18:47:52+03:00
draft = true
+++

---
# Linux Contributions

List of brought up hardware that might or might not have been merged yet (or sit still in a close-to-mainline tree):

### System on Chips (SoCs)

| SoC                 | Component                        | Status/Description                                                                                                                                             |
|---------------------|-----------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **ATM7051**         | Timer                            | Main block works                                                                                                                                               |
|                     | Multicore                        | All cores work                                                                                                                                                 |
|                     | I2C/UART                         | Works with fixed clocks                                                                                                                                        |
|                     | SIRQ                             | Secondary interrupt controller with support for up to 3 external interrupt lines (e.g., for MMC and PMIC)                                                      |
|                     | SPS                              | Smart Power System for managing CPU-related power gates                                                                                                        |
|                     | Pinctrl                          | Partial support due to limited kernel source availability                                                                                                      |
|                     | DMA                              | Works                                                                                                                                                          |
|                     | Regulators                       | Works                                                                                                                                                          |
| **Exynos 3475**     | Multicore                        | 2 out of 4 cores enabled via SMP                                                                                                                               |
|                     | ChipID                           | Works                                                                                                                                                          |
|                     | MCT (Multi-Core Timer)           | Works                                                                                                                                                          |
|                     | Pinctrl                          | Works                                                                                                                                                          |
|                     | Clock Controller                 | Full support for all blocks                                                                                                                                    |
|                     | HSIC/I2C                         | Works                                                                                                                                                          |
|                     | eMMC                             | Works                                                                                                                                                          |
|                     | MIPI DSIM/CSIM PHY               | Works                                                                                                                                                          |
|                     | Decon                            | Works                                                                                                                                                          |
|                     | Mali T710                        | Enabled via Panfrost                                                                                                                                           |
| **Exynos 8895**     | Multicore                        | Enabled via PSCI                                                                                                                                                |
|                     | ChipID                           | Works                                                                                                                                                          |
|                     | MCT & ARMv8 Generic Timer        | Both work fine; MCT no longer requires a fixed clock                                                                                                           |
|                     | Pinctrl                          | Works                                                                                                                                                          |
|                     | Clock Controller                 | CMU-{TOP, PERIS, PERIC0, PERIC1, FSYS0, FSYS1} supported                                                                                                       |
|                     | HSIC/I2C/SPI                     | Works                                                                                                                                                          |
| **MSM8212/MSM8610**  | Multicore                        | Enabled via SMP                                                                                                                                                |
|                     | QGIC2                            | Qualcomm Generic Interrupt Controller works                                                                                                                   |
|                     | ARMv7 Generic Timer              | Works                                                                                                                                                          |
|                     | Pinctrl                          | Works                                                                                                                                                          |
|                     | I2C/UART                         | Works                                                                                                                                                          |
|                     | USB                              | Works                                                                                                                                                          |

### Phones

| Phone                          | Component                          | Status/Description                                                                                                                                             |
|---------------------------------|-------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------|
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
uniLoader is a secondary bootloader that I've been working on in the meanwhile. It is capable of loading the upstream Linux kernel for Android and iOS-based devices.

The purpose behind it is to provide a small shim for avoiding vendors' bootloader quirks. For example:
  - Exynos devices past 2016 leave decon's framebuffer refreshing disabled right before jumping to kernel, which makes initial debugging efforts when bringing up the platform to upstream linux hard
  - Some S-Boot releases have CNTFRQ_EL0 left unset, which causes issues since the linux timer driver cannot read its value and it SErrors. It either has to be set as a hardcoded frequency property in the DT, or passed via libfdt from uniLoader (the latter is cleaner, but still has to be implemented)
  
The currently supported architectures are ARMV7 and AARCH64. A lot of Samsung devices, as well as a few Apple and Qualcomm ones are supported.
