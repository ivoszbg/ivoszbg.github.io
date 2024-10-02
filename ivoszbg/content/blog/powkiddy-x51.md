+++
title = 'Bringing Mainline Linux to life on ATM7051 handhelds'
date = 2024-10-02T18:48:50+03:00
ShowToc = "true"
+++
## Mainline Linux on the Powkiddy X51

For those who don't know, the Mainline Linux kernel (upstream) is the newest released and under-development version of the Linux kernel. Quite literally: [github.com/torvalds/linux](https://github.com/torvalds/linux).

### A bit of history

I've been working on SoC and device bringup as a Linux kernel hobbyist for a while now (~4 years). One or two years ago, I got my hands on a handheld called Powkiddy X51, because why not :D. 

At first, there wasn't much I could do: the system firmware files were stored on its NAND, and I couldn't find any exposed UART ports. I tried running a small custom ARM assembly binary to write to the framebuffer, but that didn't work, so I abandoned any further work.

Recently (1 or 2 months ago), I decided to revisit the project since a friend of mine also bought the same console. After a lot of digging (there's almost **no** information about both the SoC and the device), I discovered the following:

- A tool from Actions Semi that allows flashing individual partitions on the NAND as long as a `.fw` file with a proper partition layout is provided (IH FW Burning tool).
- UART5 TX and RX are exposed on the SD card pins.
- Kernel source for the NeoGeo Arcade Stick Pro (which uses the same SoC, though with an older kernel - 3.4 instead of the newer 3.10 with device trees on my handheld).
- Android firmware files for ATM7051 tablets (though I'm likely not going to pursue that).
- Schematics for some random ATM7051 device.

### Getting to work

After disassembling the device, soldering a serial-to-USB adapter to the UART5 RX and TX, and finally firing up PuTTY, we got **U-Boot logs!**

![U-Boot logs](/images/stockubootlogs-x51.png#center)

And then it was time to do the hard work: getting mainline Linux to boot. The boot flow of the device is as follows:

- `bootrom` → `spl` → `u-boot` → `linux` → `userspace`

In order to boot my own build of the Linux kernel, we need to replace the downstream closed-source version. It resides in the first FAT16 partition of the device (called `MISC`). After dumping the whole NAND via ADB, I mounted the `MISC` partition image, modified the `uenv.txt`, and started testing/debugging my mainline Linux fork by flashing the modified `MISC` with the Actions IH FW Burning Tool.

### Finally seeing the penguins

Of course, not everything was smooth as butter:
![alt](/images/notsosmooth-x51.png#center)

It took quite a bit of work, but I managed to get the following peripherals working:

- Timer (`actions,atm7051-timer`)
- Multi-core (`actions,atm7051-smp`)
- Serial (`actions,owl-uart`)
- I2C (`actions,s500-i2c`)
- SIRQ (a second interrupt controller) (`actions,atm7051-sirq`)
- SPS (Smart Power System) (`actions,atm7051-sps`)
- DMA (`actions,s500-dma`)
- Partially pinctrl
- Regulators (`actions,atc2603c`)
- SimpleDRM (woo, display via framebuffer!)

And **BOOM!** Arch Linux INITRAMFS with 4 fancy penguins on my handheld:

![alt](/images/bringing-mainline-linux-to-life-on-atm7051-handhelds-v0-rp5vjhpag0ld1.webp#center)

### Remaining issues

There are still a few issues to tackle:

- **Pinctrl and clock drivers** will be difficult to implement due to differences in coding styles and lack of documentation on pinctrl groups, functions, clock gate/div values, etc.
- There's **no NAND driver** for any Actions SoC upstream. We’ll either need to write one or stick with MMC.
- I haven't explored MMC deeply yet since clocks are required first, but it seems likely MMC won't be controlled via the DMA block.
- The **GPU driver isn't open source**. We can use proprietary blobs with hacks, but it'll still be tricky.
- We can't use both **SD card and UART5** at the same time—using both results in garbled UART output.

I also [tried compiling and booting the 3.4 kernel](https://imgur.com/BPyupV1), which needs some changes to the board driver files since MMC isn't wired to SIRQ on our device. It shouldn't be too hard to get to initramfs. If anyone wants to work on that, feel free! It has more drivers available than my current mainline fork: [linux-atm7051-3.4](https://github.com/ivoszbg/linux-atm7051-3.4).

This device doesn't seem to have bootloader signing, hence why I've also started working on upstream U-Boot. Currently it's booting with working UART, which is capable of booting Linux via Kermit:

![alt](/images/atm7051_u-boot.png#center)

This will be useful if we ever get to fully replacing the outdated stock U-Boot.

### Conclusion

Honestly, don’t have huge expectations for the future. I don’t have *that* much motivation to keep working on this SoC, and I'd appreciate any help.

I’ll be upstreaming my work in the following weeks/months, and I also need to write more detailed instructions on how things work.

### Useful Links

(Note: I am **in no way responsible** for any damage you do to your device by trying to replicate what I've done. If you want to experiment with flashing directly to your NAND, proceed at your own risk.)

- My close-to-mainline Linux branch with ATM7051 commits: [linux-atm7051](https://github.com/ivoszbg/linux/tree/v6.11rc2-atm7051)
- Useful information like UART pins and schematics: [FoxExe/PowKiddy_fw](https://github.com/FoxExe/PowKiddy_fw)
- The Actions IH FW Burning tool: [Google Drive link](https://drive.google.com/file/d/1rtRJhpKrM6H6nkps-zROSNV0WDQ3FAfS/view)

