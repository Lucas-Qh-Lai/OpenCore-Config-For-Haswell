# OpenCore-Config-For-Haswell — OpenCore Config for Desktop Haswell Hackintosh

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Haswell-6ee7b7?style=flat-square" alt="Platform">
  <img src="https://img.shields.io/badge/Bootloader-OpenCore-818cf8?style=flat-square" alt="Bootloader">
  <img src="https://img.shields.io/badge/SMBIOS-iMac18,3-6ee7b7?style=flat-square" alt="SMBIOS">
  <img src="https://img.shields.io/badge/License-MIT-818cf8?style=flat-square" alt="License">
</p>

> [切换到中文](/README.md) · [Chinese Version](/README.md)

---

**An OpenCore configuration for desktop Haswell Hackintosh, built by the author in middle school and verified booting on a Gigabyte H81M-S1 + i3-4160 + GT 720.**

This is a very early project of mine. The original `config.plist` and kext set are preserved as-is; this update only adds documentation, a license, and usage notes. It is not guaranteed to work with 2026-era macOS releases. If you also run a Haswell desktop, use it as a reference—but always regenerate the SMBIOS and USB mapping for your own hardware.

> ⚠️ **Must-read safety note**: the `config.plist` in this repo contains **placeholder serials** (serial number / MLB / ROM) that are visible to everyone. **Never boot with them.** Regenerate with GenSMBIOS before your first boot (see [SMBIOS must be regenerated](#-smbios-must-be-regenerated)).

---

## 📌 Table of Contents

- [Tested Hardware](#️-tested-hardware)
- [Repository Layout](#-repository-layout)
- [Drivers & Patches](#-drivers--patches)
- [Key Settings](#️-key-settings)
- [SMBIOS Must Be Regenerated](#-smbios-must-be-regenerated)
- [Usage](#-usage)
- [Known Issues & Limits](#-known-issues--limits)
- [Compatibility](#-compatibility)
- [Contributing](#-contributing)
- [License](#-license)
- [Acknowledgements](#-acknowledgements)

---

## 🖥️ Tested Hardware

| Part | Model |
| --- | --- |
| Motherboard | Gigabyte H81M-S1 (H81 / LGA1150) |
| CPU | Intel i3-4160 (Haswell, 2C4T, HD 4400 iGPU) |
| dGPU | NVIDIA GT 720 (Kepler; no-driver approach; newer macOS has no NVIDIA Web Driver) |
| Wi-Fi | Broadcom (BrcmPatchRAM3 + AirportBrcmFixup) |
| Ethernet | Realtek RTL8111 (RealtekRTL8111.kext) |
| Audio | 8 Series/C220 HD Audio (AppleALC, `alcid=2`) |
| SMBIOS | iMac18,3 |

> "Theoretically works on all desktop Haswell platforms"—that was the author's original claim. The honest 2026 version: Haswell desktops with H81/H87/Z87/Z97 boards get the most value; other boards should borrow ideas only, never copy the EFI blindly.

## 📁 Repository Layout

```text
OpenCore-Config-For-Haswell/
├── README.md            # Chinese README
├── README.en.md         # English README (this file)
├── LICENSE              # MIT License
├── BOOT/
│   └── BOOTx64.efi      # UEFI fallback booter
└── OC/
    ├── OpenCore.efi     # OpenCore binary
    ├── config.plist     # Full config (SMBIOS iMac18,3, placeholder serials)
    ├── ACPI/
    │   ├── SSDT-EC-DRTNIA.aml / SSDT-EC-DESKTOP.aml
    │   └── SSDT-PLUG-DRTNIA.aml
    ├── Drivers/
    │   ├── OpenRuntime.efi / OpenCanopy.efi
    │   └── OpenHfsPlus.efi / ResetNvramEntry.efi
    ├── Kexts/           # see table below
    └── Resources/       # OpenCanopy UI assets
```

## 🧩 Drivers & Patches

### ACPI

| File | Purpose |
| --- | --- |
| SSDT-EC-DESKTOP.aml | Desktop EC emulation, required by macOS |
| SSDT-PLUG-DRTNIA.aml | CPU power-management injection (PLUG) |

### Drivers

| File | Purpose |
| --- | --- |
| OpenRuntime.efi | OpenCore runtime, required |
| OpenCanopy.efi | Graphical boot picker (with `PickerMode=External`) |
| OpenHfsPlus.efi | HFS+ partition support |
| ResetNvramEntry.efi | Adds a "Reset NVRAM" picker entry |

### Kexts (versions as shipped back then)

| Kext | Version | State | Notes |
| --- | --- | --- | --- |
| Lilu.kext | 1.6.3 | ✅ Enabled | Patch framework, prerequisite for most others |
| VirtualSMC.kext | 1.3.0 | ✅ Enabled | SMC emulation, required |
| WhateverGreen.kext | 1.6.4 | ✅ Enabled | Graphics patching |
| SMCProcessor.kext | 1.3.0 | ✅ Enabled | CPU temperature sensors |
| SMCSuperIO.kext | 1.3.0 | ✅ Enabled | Fan sensors |
| AppleALC.kext | 1.7.9 | ✅ Enabled | Audio, with `alcid=2` |
| USBPorts.kext | 1.0 | ✅ Enabled | USB map customized for the author's machine (**do not copy**) |
| USBToolBox.kext | 1.1.1 | ⏸️ Disabled | USB mapping tool, spare |
| UTBMap.kext | 1.1 | ⏸️ Disabled | USB mapping data, spare |
| RealtekRTL8111.kext | 2.4.2 | ✅ Enabled | Ethernet |
| AirportBrcmFixup.kext | 2.1.6 | ✅ Enabled | Broadcom Wi-Fi fix (with NIC injector) |
| BlueToolFixup.kext | 2.6.4 | ✅ Enabled | Bluetooth fix |
| BrcmFirmwareData.kext | 2.6.4 | ✅ Enabled | Broadcom BT firmware |
| BrcmPatchRAM3.kext | 2.6.4 | ✅ Enabled | Broadcom BT firmware injection |

## ⚙️ Key Settings

- **SMBIOS**: `iMac18,3`. The more period-correct choices for Haswell are iMac15,1 / iMac14,x; the author picked 18,3 and it booted. For newer macOS, re-pick per the [Dortania guide](https://dortania.github.io/OpenCore-Install-Guide/config.plist/haswell.html) and regenerate serials.
- **Boot-args**: `-v keepsyms=1 debug=0x100 alcid=2 amfi_get_out_of_my_way=1 ipc_control_port_options=0`. `amfi_get_out_of_my_way=1` weakens system integrity checks—debug sessions only; remove once stable.
- **DeviceProperties**: iGPU `AAPL,ig-platform-id=04001204` (a common desktop HD 4400 value), HDEF audio path; redo for a different board/iGPU.
- **Kernel Quirks**: `AppleCpuPmCfgLock`, `AppleXcpmCfgLock`, `DisableIoMapper`, etc.—the usual Gigabyte H81 + Haswell combo when CFG Lock is left locked.
- **SecureBootModel**: `Disabled`, Vault `Optional`—typical debug-era settings; tighten for daily use.
- **APFS**: `EnableJumpstart=True`, standard.

## 🔑 SMBIOS Must Be Regenerated

The serial / MLB / ROM in this repo are **public placeholders**. Booting with them breaks Apple ID, iMessage, and FaceTime, and collides with other users. **Regenerate before first use:**

1. Download [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) and generate a set for your SMBIOS model.
2. Open `OC/config.plist` with ProperTree or OCAuxiliaryTools and fill in `PlatformInfo → Generic`: `SystemSerialNumber`, `MLB`, `ROM` (ROM is usually your NIC MAC without colons).
3. Update `SystemUUID` as well, save, reboot, and confirm the new serial in About This Mac.

## 🚀 Usage

1. Prepare a FAT32 USB stick (GPT), create an `EFI` folder.
2. Copy this repo's `BOOT` and `OC` folders into `EFI/`, giving `EFI/BOOT/BOOTx64.efi` and `EFI/OC/...`.
3. Generate and fill in your own serials per [SMBIOS Must Be Regenerated](#-smbios-must-be-regenerated).
4. BIOS: disable VT-d (or keep `DisableIoMapper=True`), disable Secure Boot, SATA to AHCI, XHCI Handoff on; disable CFG Lock if available (then try dropping the related quirks).
5. Boot the stick in UEFI mode and pick the installer; keep `-v` verbose for faster troubleshooting.
6. After a successful install, copy the stick's EFI to the disk's EFI partition and test booting without the stick.

## 🧪 Known Issues & Limits

- **Old snapshot**: drivers and OpenCore date from ~2023 and have not tracked later macOS/OpenCore majors; treat as archive + reference.
- **USB map is machine-specific** (`USBPorts.kext`): a different board or front-panel wiring will break it—remap with USBToolBox.
- **GT 720**: Kepler no-driver tricks only shine on old systems; macOS Monterey and later have little room for NVIDIA—verify on your target OS, fall back to iGPU output if needed.
- **iMac18,3 on Haswell**: a "boots but not principled" combo; purists should switch to a contemporary model like iMac15,1 per Dortania's Haswell chapter and retest.
- **Written by a middle schooler**: the original README had rough English; rewritten now, but the config itself has not been retested in a 2026 environment.

## 🧭 Compatibility

| Item | Notes |
| --- | --- |
| CPU | Desktop Haswell (i3/i5/i7-4xxx, LGA1150) gets the most value |
| Motherboard | H81/H87/Z87/Z97 desktops; laptops need a different repo |
| iGPU | HD 4400/4600; adjust `ig-platform-id` for your CPU |
| macOS | Old releases verified back then; upgrade OpenCore + all kexts before trying new ones |
| OpenCore | Walk your config against the [Dortania Haswell guide](https://dortania.github.io/OpenCore-Install-Guide/config.plist/haswell.html) item by item |

## 🤝 Contributing

Issues and PRs welcome:

- Bug reports: include board/CPU/GPU/NIC models, OpenCore version, target macOS version, verbose-boot photo or `opencore-*.txt` log.
- Config changes: explain what changed, why, and on which board it was tested.
- Driver/OpenCore upgrades: note old/new versions and test results.

## 📄 License

This project is under the **MIT License**, see [LICENSE](/LICENSE). Third-party binaries are not vendored here beyond the EFI; ACPI/kexts come from upstream open-source projects and remain under their own licenses.

## 🙏 Acknowledgements

- **[OpenCore](https://github.com/acidanthera/OpenCore) (Acidanthera)**—the entire foundation of this repo, bootloader plus excellent docs.
- **[Dortania OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)**—the Haswell config is checkable item by item; read it before touching config.
- **[Lilu](https://github.com/acidanthera/Lilu) / [WhateverGreen](https://github.com/acidanthera/WhateverGreen) / [AppleALC](https://github.com/acidanthera/AppleALC) / [VirtualSMC](https://github.com/acidanthera/VirtualSMC)**—the Acidanthera family; no Hackintosh without them.
- **[USBToolBox](https://github.com/USBToolBox/tool)**—source of the USB-mapping approach; spare kexts included.
- Broadcom BT/Wi-Fi driver authors (BrcmPatchRAM / AirportBrcmFixup)—the people who made Broadcom cards live on macOS.
- Fellow H81 tinkerers and forum veterans whose troubleshooting posts saved the author many times.

---

**A middle schooler's old EFI—a bootable starting point for Haswell desktops of the same era.**

[Chinese Version](/README.md)
