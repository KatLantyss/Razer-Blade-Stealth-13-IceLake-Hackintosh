# Razer Blade Stealth 13 Icelake Hackintosh (OpenCore)

## Support Version
---
* OpenCore: `0.8.2`
* MacOS: `Monterey 12.4`
![](https://i.imgur.com/rgKySm7.png)
    >I changed graphic card name in this picture.

## Tested Model Spec
---
* ### Razer Blade Stealth 13 (2020) RZ09-0310
    ![](https://assets2.razerzone.com/images/blade-stealth-2020/razer-blade-stealth-2020-120hz-model.png)
    |  Specifications   |              Detail                               |
    |-------------------|---------------------------------------------------|
    |Processor          |i7-1065G7                                          |
    |Integrated Graphics|Intel Iris Plus Graphics                           |
    |Graphic Card       |NVIDIA® GeForce GTX 1650 Ti Max-Q (4GB GDDR5 VRAM) |
    |Memory             |16GB<sup>8G*2</sup> LPDDR4X 3733 MHz               |   
    |Sound Card         |Realtek ALC298 (0x0298)                            |
    |Wireless Card      |AX201                                              |
    |Battery            |53.1Wh                                             |


## What is Working?
---
- CPU power management
- Hardware acceleration
- Sleep/Wake
- Battery read-out
- Audio (Internal speakers, 3.5mm headphone jack) <sup>**Unstable(Boot from Windows)**</sup>
- Keyboard & trackpad/touchscreen with all macOS gestures
- Wi-Fi & Bluetooth
- USB ports

## What is Not Working?
---
- Airdrop <sup>**Not Supported in MacOS when using Intel Wireless Card**</sup>
- NVIDIA® GeForce GTX 1650 Ti Max-Q <sup>**Disable in ACPI**</sup>
- ThunderBolt 3 <sup>**Type-C port is fine to use.**</sup>

---
---
# Guidence
## Getting started with [OpenCore](https://dortania.github.io/OpenCore-Install-Guide/)
> Essentially follow the [Laptop Icelake | OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/icelake.html#intel-bios-settings) and incorporate the insights listed below.
## BIOS
> We accept no liability for any loss or damage howsoever changing BIOS with this guidance and cause damage on your device. Please be careful and make sure you know what you are doing.
* Due to some BIOS setting options are hidden by Razer, we can use several tools to help us matching [Intel BIOS Settings | OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/icelake.html#intel-bios-settings), here are the steps in  [StoneEvil](https://github.com/stonevil/Razer_Blade_Advanced_early_2019_Hackintosh) guide.
---
---
# Specific Patch
## ACPI
---
### Add
* SSDT-dGPU-Off
    * More details in [Disabling laptop dGPUs | Getting Started With ACPI](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/laptop-disable.html#disabling-laptop-dgpus-ssdt-dgpu-off-nohybgfx).
* SSDT-PTSWAK
    * More details in [Here](https://www.reddit.com/r/hackintosh/comments/vz2lfq/success_macos_monterey_124_on_a_2021_razer_blade/).
* SSDT-TPAD
    * I wrote this by myself, and it made the touchpad work.

### Patch
* SSDT-TPAD
    |Comment|String |Change _STA to XSTA in TPD0|
    |-------|-------|---------------------------|
    |Enabled|Boolean|YES                        |
    |Find   |Data   |141C5F53 544100            |
    |Replace|Data   |141C5853 544100            |

* SSDT-PTSWAK
    |Comment        |String |Rename _PTS to ZPTS (Black Screen on Wake Fix)|
    |---------------|-------|----------------------------------------------|
    |Enabled        |Boolean|YES                                           |
    |Find           |Data   |5F505453 01                                   |
    |Replace        |Data   |5A505453 01                                   |
    |TableSignature |Data   |44534454                                      |

    |Comment        |String |Rename _WAK to ZWAK (Black Screen on Wake Fix)|
    |---------------|-------|----------------------------------------------|
    |Enabled        |Boolean|YES                                           |
    |Find           |Data   |5F57414B 01                                   |
    |Replace        |Data   |5A57414B 01                                   |
    |TableSignature |Data   |44534454                                      |

    |Comment        |String |Rename RTEC to XTEC (Black Screen on Wake Fix)|
    |---------------|-------|----------------------------------------------|
    |Enabled        |Boolean|YES                                           |
    |Find           |Data   |47055254 45430070                             |
    |Replace        |Data   |47055854 45430070                             |
    |TableSignature |Data   |44534454                                      |

## DeviceProperties
---
### Add
- **PciRoot(0x0)/Pci(0x2,0x0)**
   - [**Support all possible Core Display Clock (CDCLK) frequencies on ICL platforms**](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#support-all-possible-core-display-clock-cdclk-frequencies-on-icl-platforms)
   - [**Fix the issue that the builtin display remains garbled after the system boots on ICL platforms**](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#fix-the-issue-that-the-builtin-display-remains-garbled-after-the-system-boots-on-icl-platforms)
   - More details in [**Intel Iris Plus Graphics (Ice Lake processors) | WhateverGreen (Intel® HD Graphics FAQs)**](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#intel-iris-plus-graphics-ice-lake-processors)

## NVRAM
### Add
- 7C436110-AB2A-4BBB-A880-FE41995C9F82
    - boot-args
        - `-noDC9` 	
            - This will fix sleep problem
        - `-igfxdbeo` `-igfxcdc`
            - Ignore when you place in **DeviceProperties**

---
---
# Credits
- [**Apple**](https://www.apple.com/tw/) for the macOS.
- [**Acidanthera**](https://github.com/acidanthera) for awesome kexts and first-class support for hackintosh enthusiasts.
- [**Dortania**](https://github.com/dortania) for the great guides.
- [**StoneEvil**](https://github.com/stonevil/Razer_Blade_Advanced_early_2019_Hackintosh) for BIOS setting.
- [**jnr0602**](https://www.reddit.com/user/jnr0602/) for providing SSDT-PTSWAK.