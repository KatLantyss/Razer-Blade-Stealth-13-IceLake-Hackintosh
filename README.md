# Razer Blade Stealth 13 Icelake Hackintosh (OpenCore)

## Support Version
![OpenCore](https://img.shields.io/badge/OpenCore-1.0.2-grey?style=for-the-badge&logo=okta&logoColor=white&labelColor=blue&link=https%3A%2F%2Fgithub.com%2Facidanthera%2FOpenCorePkg%2Freleases%2Ftag%2F1.0.2
) &emsp; ![MacOS](https://img.shields.io/badge/MacOS-Sonoma_14.6-grey?style=for-the-badge&logo=apple&logoColor=white&labelColor=black
)

## Screenshot
### About this Mac
<p align="center"><img src="https://github.com/KatLantyss/Razer-Blade-Stealth-13-IceLake-Hackintosh/blob/main/Screenshot/Sonoma%2014.6.png"></p>

### Idle power consumption
<p align="center"><img src="https://github.com/KatLantyss/Razer-Blade-Stealth-13-IceLake-Hackintosh/blob/main/Screenshot/idle.png"></p>

> [!NOTE]
> The battery hardly drains during sleep mode. With the brightness set to about 60%, it drains about 5% of the battery over 30 minutes of regular use (watching videos, listening to music, and word processing). Theoretically, the battery life can last up to 10 hours.

## Tested Model Spec
* ### Razer Blade Stealth 13 (2020) RZ09-0310
    <p align="center"><img src="https://assets2.razerzone.com/images/blade-stealth-2020/razer-blade-stealth-2020-120hz-model.png"></p>

    | Specifications      | Detail                                             |
    | ------------------- | -------------------------------------------------- |
    | Processor           | i7-1065G7                                          |
    | Integrated Graphics | Intel Iris Plus Graphics                           |
    | Graphic Card        | NVIDIA® GeForce GTX 1650 Ti Max-Q (4GB GDDR5 VRAM) |
    | Memory              | 16GB<sup>8G*2</sup> LPDDR4X 3733 MHz               |
    | Sound Card          | Realtek ALC298 (0x0298)                            |
    | Wireless Card       | AX201                                              |
    | Battery             | 53.1Wh                                             |


## What is Working?
- CPU power management
- Hardware acceleration
- Sleep/Wake
- Battery read-out
- Audio (Internal speakers, 3.5mm headphone jack)
- Keyboard & trackpad/touchscreen with all macOS gestures
- Wi-Fi & Bluetooth
- USB ports

## What is Not Working?
- AWDL (Apple Wireless Direct Link) <sup>**Not Supported in MacOS while using Intel Wireless Card**</sup>
- NVIDIA® GeForce GTX 1650 Ti Max-Q <sup>**Disable in BIOS**</sup>
- ThunderBolt 3 and External Display <sup>**Type-C port is fine to use.**</sup>

---
---
# Guidence
## Getting started with [OpenCore](https://dortania.github.io/OpenCore-Install-Guide/)

> Essentially follow the [Laptop Icelake | OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/icelake.html#intel-bios-settings) and incorporate the insights listed below.
## BIOS
> [!CAUTION]
> We accept no liability for any loss or damage howsoever changing BIOS with this guidance and cause damage on your device. Please be careful and make sure you know what you are doing.

> Due to some BIOS setting options are hidden by Razer, we can use several tools to help us matching [Intel BIOS Settings | OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/config-laptop.plist/icelake.html#intel-bios-settings), here are the steps in  [StoneEvil](https://github.com/stonevil/Razer_Blade_Advanced_early_2019_Hackintosh) guide.

### Settings
- **Advanced**
  - **CPU Configuration**
    - **Software Guard Extensions(SGX)** →&nbsp;`Disabled`
  - **Power and Performance**
    - **CPU Power Management**
      - **CPU Lock Configuration**
        - **CFG Lock** →&nbsp;`Disabled`
  - **Thunderbolt™ Configuration**
    - **Intergrated Thunderbolt™ Support** →&nbsp;`Disabled`
  - **USB Configuration** (No need `ReleaseUsbOwnership` in UEFI > Quirks when the follow options are enabled.)
    - **XHCI Hand-off** →&nbsp;`Enabled`
    - **EHCI Hand-off** →&nbsp;`Enabled`
- **Chipset**
  - **System Agent(SA) Configuration**
    - **Graphics Configuration**
      - **Primary Display** →&nbsp;`IGFX` (Optional, or disable in ACPI)
      - **DVMT Pre-Allocated** →&nbsp;`64M`
      - **DVMT Total Gfx Memory** →&nbsp;`Max`
    - **TCSS Setup Menu** (Optional. Used to disable Thunderbolt™ devices.)
        - **ITBI PCI0 Root Port** →&nbsp;`Disabled`
        - **ITBI PCI1 Root Port** →&nbsp;`Disabled`
        - **ITBT DMA0** →&nbsp;`Disabled`
    - **VT-d** →&nbsp;`Disabled`
    - **SATA and RST Configuration**
      - **SATA Mode Selection** →&nbsp;`AHCI`
  - **Security**
    - **Secure Boot**
      - **Secure Boot** →&nbsp;`Disabled`
- **Boot**
    - **Fast Boot** →&nbsp;`Disabled`
    - **CSM Support** →&nbsp;`Disabled`

---
---
# Specific Patch
## ACPI
### Add
> [!NOTE]
> This part refers to [OC-little](https://github.com/daliansky/OC-little) for patching and adding missing devices, as well as verifying all functionalities, including power consumption and sleep.

#### Requirements
| SSDTs       | Description                                                                                                                                                                                                                                                                                                                                                                              |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SSDT-AWAC   | This is a problem because macOS cannot communicate with AWAC clocks, so this requires us to either force on the legacy RTC clock or if unavailable create a fake one for macOS.                                                                                                                                                                                                          |
| SSDT-EC     | Fixes embedded controller power by creating a fake embedded controller.                                                                                                                                                                                                                                                                                                                  |
| SSDT-PLUG   | Allows for native CPU power management on Haswell and newer, see [Getting Started With ACPI](https://dortania.github.io/Getting-Started-With-ACPI/) for more details.                                                                                                                                                                                                                    |
| SSDT-PNLF   | Fixes brightness control.                                                                                                                                                                                                                                                                                                                                                                |
| SSDT-PPRW   | Fixes the instant wake issue caused by RP05. RP05 appears to be the root port for the Thunderbolt™ controller. On IceLake, Thunderbolt™ is integrated into the CPU and cannot be patched, unlike discrete Thunderbolt™ controllers. In this case, I wrote this SSDT to prevent RP05 from waking the system and to improve power consumption.                                             |
| SSDT-PTSWAK | Fixed the wake issue where the lid had to be opened twice for the system to wake properly.                                                                                                                                                                                                                                                                                               |
| SSDT-RHUB   | Needed to fix Root-device errors on many IceLake laptops                                                                                                                                                                                                                                                                                                                                 |
| SSDT-TPD0   | Razer's trackpad interrupt mode is APIC, which is more compatible with macOS than GPIO. However, the trackpad is on I2C1, and I2C0 acts as a controller for I2C1 through I2C3. This causes macOS to mistakenly recognize I2C0 as the trackpad instead of I2C1. To resolve this, I wrote this SSDT to disable the other I2CX devices and enable only I2C1, ensuring proper functionality. |
| SSDT-USBX   | Fixes USB power.                                                                                                                                                                                                                                                                                                                                                                         |
| SSDT-XOSI   | Fixes I2C trackpads and enables them in ACPI.                                                                                                                                                                                                                                                                                                                                            |

#### Optional
| Option SSDTs | Description                                                                                                                                                                                                                                    |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| SSDT-DGPU    | More details in [Disabling laptop dGPUs \| Getting Started With ACPI](https://dortania.github.io/Getting-Started-With-ACPI/Laptops/laptop-disable.html#disabling-laptop-dgpus-ssdt-dgpu-off-nohybgfx). No need when you disabled dGPU in BIOS. |
| Other SSDTs  | More details in [添加缺失的部件 \| OC-little](https://github.com/daliansky/OC-little/tree/master/06-%E6%B7%BB%E5%8A%A0%E7%BC%BA%E5%A4%B1%E7%9A%84%E9%83%A8%E4%BB%B6#%E6%B7%BB%E5%8A%A0%E7%BC%BA%E5%A4%B1%E7%9A%84%E9%83%A8%E4%BB%B6).          |


## DeviceProperties
### Add
- **PciRoot(0x0)/Pci(0x2,0x0)**
   - [**Support all possible Core Display Clock (CDCLK) frequencies on ICL platforms**](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#support-all-possible-core-display-clock-cdclk-frequencies-on-icl-platforms)
   - [**Fix the issue that the builtin display remains garbled after the system boots on ICL platforms**](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#fix-the-issue-that-the-builtin-display-remains-garbled-after-the-system-boots-on-icl-platforms)
   - More details in [**Intel Iris Plus Graphics (Ice Lake processors) | WhateverGreen (Intel® HD Graphics FAQs)**](https://github.com/acidanthera/WhateverGreen/blob/master/Manual/FAQ.IntelHD.en.md#intel-iris-plus-graphics-ice-lake-processors)
> [!TIP]
> Set the `pci-aspm-default` parameter to L1 for all PCIe devices detected in Hackintool and configure ASPM to L0 to optimize power-saving effects. More details in [here](https://github.com/daliansky/OC-little/tree/master/16-%E7%A6%81%E6%AD%A2PCI%E8%AE%BE%E5%A4%87%E5%8F%8A%E8%AE%BE%E7%BD%AEASPM%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F/16-2-%E8%AE%BE%E7%BD%AEASPM%E5%B7%A5%E4%BD%9C%E6%A8%A1%E5%BC%8F). 

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
- [**daliansky**](https://github.com/daliansky) for the guidance on the ACPI patches and optimizations.
- [**StoneEvil**](https://github.com/stonevil/Razer_Blade_Advanced_early_2019_Hackintosh) for BIOS setting.
- [**jnr0602**](https://www.reddit.com/user/jnr0602/) for providing SSDT-PTSWAK.
- [**mfpss95134**](https://github.com/mfpss95134) for giving advices and helps.
