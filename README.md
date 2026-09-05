# OpenCore-Config-For-Haswell — 台式 Haswell 黑苹果 OpenCore 配置

<p align="center">
  <img src="https://img.shields.io/badge/Platform-Haswell-6ee7b7?style=flat-square" alt="Platform">
  <img src="https://img.shields.io/badge/Bootloader-OpenCore-818cf8?style=flat-square" alt="Bootloader">
  <img src="https://img.shields.io/badge/SMBIOS-iMac18,3-6ee7b7?style=flat-square" alt="SMBIOS">
  <img src="https://img.shields.io/badge/License-MIT-818cf8?style=flat-square" alt="License">
</p>

> [切换到 English](/README.en.md) · [English Version](/README.en.md)

---

**一份作者初中时期折腾出来的台式 Haswell 黑苹果 OpenCore 配置，在技嘉 H81M-S1 + i3-4160 + GT 720 实测可开机。**

这是作者很早以前的作品，原汁原味保留当年的 `config.plist` 与驱动组合，仅补齐文档、许可证与使用说明，不保证适配 2026 年的新版 macOS。如果你也是 Haswell 台式机用户，欢迎拿去对照参考，但请务必按自己的硬件重做 SMBIOS 与 USB 定制。

> ⚠️ **必读安全提醒**：本仓库 `config.plist` 内含一套**示例三码**（序列号 / MLB / ROM），仅供占位，**严禁直接使用**。首次开机前请用 GenSMBIOS 重新生成（见[三码](#-三码必须重做)）。

---

## 📌 目录

- [实测环境](#-实测环境)
- [仓库内容](#-仓库内容)
- [驱动与补丁清单](#-驱动与补丁清单)
- [关键配置说明](#-关键配置说明)
- [三码必须重做](#-三码必须重做)
- [使用方法](#-使用方法)
- [已知问题与局限](#-已知问题与局限)
- [兼容性说明](#-兼容性说明)
- [贡献指南](#-贡献指南)
- [许可证](#-许可证)
- [致谢](#-致谢)

---

## 🖥️ 实测环境

| 部件 | 型号 |
| --- | --- |
| 主板 | Gigabyte H81M-S1（H81 / LGA1150） |
| CPU | Intel i3-4160（Haswell 双核四线程，HD 4400 核显） |
| 独显 | NVIDIA GT 720（开普勒架构，免驱思路；新版 macOS 已无 NVIDIA Web Driver） |
| 无线网卡 | Broadcom（BrcmPatchRAM3 + AirportBrcmFixup 驱动） |
| 有线网卡 | Realtek RTL8111（RealtekRTL8111.kext） |
| 声卡 | 8 Series/C220 HD Audio（AppleALC，`alcid=2`） |
| 机型伪装 | iMac18,3 |

> "理论上适用于所有台式 Haswell 平台"——这是作者当年的原话。今天的诚实版本是：同为 H81/H87/Z87/Z97 + Haswell CPU 的台式机最有参考价值，其他主板请只借鉴思路，不要照搬 EFI。

## 📁 仓库内容

```text
OpenCore-Config-For-Haswell/
├── README.md            # 中文说明（本文件）
├── README.en.md         # English README
├── LICENSE              # MIT License
├── BOOT/
│   └── BOOTx64.efi      # UEFI 回退引导
└── OC/
    ├── OpenCore.efi     # OpenCore 主程序
    ├── config.plist     # 全套配置（SMBIOS 为 iMac18,3，含示例三码）
    ├── ACPI/
    │   ├── SSDT-EC-DRTNIA.aml / SSDT-EC-DESKTOP.aml
    │   └── SSDT-PLUG-DRTNIA.aml
    ├── Drivers/
    │   ├── OpenRuntime.efi / OpenCanopy.efi
    │   └── OpenHfsPlus.efi / ResetNvramEntry.efi
    ├── Kexts/           # 见下表
    └── Resources/       # OpenCanopy 图形界面资源
```

## 🧩 驱动与补丁清单

### ACPI

| 文件 | 作用 |
| --- | --- |
| SSDT-EC-DESKTOP.aml | 台式机 EC 仿真，macOS 必备 |
| SSDT-PLUG-DRTNIA.aml | CPU 电源管理注入（PLUG） |

### Drivers

| 文件 | 作用 |
| --- | --- |
| OpenRuntime.efi | OpenCore 运行时基础，必备 |
| OpenCanopy.efi | 图形化启动菜单（配合 `PickerMode=External`） |
| OpenHfsPlus.efi | HFS+ 分区识别 |
| ResetNvramEntry.efi | 启动菜单提供"重置 NVRAM"选项 |

### Kexts（均为当年版本，已在括号注明）

| Kext | 版本 | 状态 | 说明 |
| --- | --- | --- | --- |
| Lilu.kext | 1.6.3 | ✅ 启用 | 补丁框架，几乎所有其他 kext 的前置 |
| VirtualSMC.kext | 1.3.0 | ✅ 启用 | SMC 仿真，必备 |
| WhateverGreen.kext | 1.6.4 | ✅ 启用 | 显卡补丁 |
| SMCProcessor.kext | 1.3.0 | ✅ 启用 | CPU 温度传感器 |
| SMCSuperIO.kext | 1.3.0 | ✅ 启用 | 风扇传感器 |
| AppleALC.kext | 1.7.9 | ✅ 启用 | 声卡驱动，配合 `alcid=2` |
| USBPorts.kext | 1.0 | ✅ 启用 | 作者按本机定制的 USB 端口（**不要照搬**） |
| USBToolBox.kext | 1.1.1 | ⏸️ 停用 | USB 定制工具，备用 |
| UTBMap.kext | 1.1 | ⏸️ 停用 | USB 定制映射，备用 |
| RealtekRTL8111.kext | 2.4.2 | ✅ 启用 | 有线网卡 |
| AirportBrcmFixup.kext | 2.1.6 | ✅ 启用 | 博通无线网卡修复（含 NIC 注入器） |
| BlueToolFixup.kext | 2.6.4 | ✅ 启用 | 蓝牙修复 |
| BrcmFirmwareData.kext | 2.6.4 | ✅ 启用 | 博通蓝牙固件 |
| BrcmPatchRAM3.kext | 2.6.4 | ✅ 启用 | 博通蓝牙固件注入 |

## ⚙️ 关键配置说明

- **SMBIOS**：`iMac18,3`。Haswell 时代更常见的选择是 iMac15,1 / iMac14,x，作者当年选了 18,3 且实测可开机；如果你要装较新的 macOS，建议按 [Dortania 指南](https://dortania.github.io/OpenCore-Install-Guide/config.plist/haswell.html)重新选择并重做三码。
- **启动参数**：`-v keepsyms=1 debug=0x100 alcid=2 amfi_get_out_of_my_way=1 ipc_control_port_options=0`。其中 `amfi_get_out_of_my_way=1` 会放宽系统完整性检查，仅建议调试阶段使用，稳定后请移除。
- **DeviceProperties**：核显注入 `AAPL,ig-platform-id=04001204`（HD 4400 桌面端常用值之一），声卡路径为 HDEF；换主板/换 CPU 核显请重做。
- **Kernel Quirks**：`AppleCpuPmCfgLock`、`AppleXcpmCfgLock`、`DisableIoMapper` 等均为 Haswell + 技嘉 H81 常见组合（CFG Lock 未解锁时需要）。
- **SecureBootModel**：`Disabled`，Vault 为 `Optional`——典型调试期配置，正式使用可按需收紧。
- **APFS**：`EnableJumpstart=True`，常规设置。

## 🔑 三码必须重做

仓库里的序列号 / MLB / ROM 是**示例占位**，全网可见，直接使用会导致 Apple ID、iMessage、FaceTime 异常，甚至与他人冲突。**第一次使用前必须重做：**

1. 下载 [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS)，选择与你 SMBIOS 对应的机型生成三码。
2. 用 ProperTree 或 OCAuxiliaryTools 打开 `OC/config.plist`，填入 `PlatformInfo → Generic` 的 `SystemSerialNumber`、`MLB`、`ROM`（ROM 一般填网卡 MAC 去冒号）。
3. 检查 `SystemUUID` 一并更新，保存后重启，确认"关于本机"显示的新序列号与生成的一致。

## 🚀 使用方法

1. 准备一个 FAT32 格式的 U 盘（GPT 分区），新建 `EFI` 文件夹。
2. 把本仓库的 `BOOT`、`OC` 两个文件夹完整拷入 `EFI/`，得到 `EFI/BOOT/BOOTx64.efi` 与 `EFI/OC/...`。
3. 按[三码](#-三码必须重做)一节生成并填入自己的三码。
4. BIOS 设置：关闭 VT-d（或保持 `DisableIoMapper=True`）、关闭 Secure Boot、SATA 设为 AHCI、XHCI Handoff 开启、CFG Lock 若能关闭则关闭（关闭后可尝试去掉对应 Quirks）。
5. U 盘 UEFI 启动，选择安装介质；建议全程保留 `-v` 跑码，排错更快。
6. 安装成功后把 U 盘的 EFI 复制到硬盘 EFI 分区，拔掉 U 盘再测一次能否独立引导。

## 🧪 已知问题与局限

- **年代久远**：驱动与 OpenCore 均为 2023 年前后版本，未跟进后续 macOS 与 OpenCore 的大版本变化，仅作存档与思路参考。
- **USB 定制是按作者本机做的**（`USBPorts.kext`）：换主板/换机箱前面板接线必翻车，请用 USBToolBox 重新定制。
- **GT 720**：开普勒免驱只在老系统吃香，macOS Monterey 及之后对 NVIDIA 支持急剧收窄，独显请以实际测试为准，必要时切核显输出。
- **iMac18,3 配 Haswell**：属于"能开机但不讲究"的组合，强迫症请按 Dortania Haswell 章节换成 iMac15,1 等同期机型并重测。
- **作者当年英语不好**：原 README 有不少中式英语，本次重写已修正，但配置本身未经 2026 年环境复测。

## 🧭 兼容性说明

| 项目 | 说明 |
| --- | --- |
| CPU | Haswell 台式（i3/i5/i7-4xxx，LGA1150）最有参考价值 |
| 主板 | H81/H87/Z87/Z97 台式机；笔记本请另找对应仓库 |
| 核显 | HD 4400/4600，`ig-platform-id` 需按自己 CPU 调整 |
| macOS | 当年验证的老版本；新版系统请先升级 OpenCore 与全套 kext 再试 |
| OpenCore | 建议对照 [Dortania Haswell 指南](https://dortania.github.io/OpenCore-Install-Guide/config.plist/haswell.html)把配置逐项过一遍 |

## 🤝 贡献指南

欢迎提交 Issue 与 PR：

- 报问题请附主板/CPU/显卡/网卡型号、OpenCore 版本、目标 macOS 版本、跑码照片或 `opencore-*.txt` 日志。
- 改配置请说明改了哪一项、为什么改、在哪块主板上测过。
- 升级驱动/OpenCore 版本请注明新旧版本号与实测结果。

## 📄 许可证

本项目采用 **MIT License**，详见 [LICENSE](/LICENSE)。ventoy 等第三方二进制不在本仓库内；ACPI/驱动均来自上游开源项目，各自许可证归原作者所有。

## 🙏 致谢

- **[OpenCore](https://github.com/acidanthera/OpenCore)（Acidanthera）**：本仓库存在的全部基础，感谢 bootloader 本体与详尽文档。
- **[Dortania OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/)**：Haswell 配置几乎逐项可查，新手请先读它再动 config。
- **[Lilu](https://github.com/acidanthera/Lilu) / [WhateverGreen](https://github.com/acidanthera/WhateverGreen) / [AppleALC](https://github.com/acidanthera/AppleALC) / [VirtualSMC](https://github.com/acidanthera/VirtualSMC)**：Acidanthera 全家桶，没有它们就没有黑苹果。
- **[USBToolBox](https://github.com/USBToolBox/tool)**：USB 定制思路来源，备用 kext 亦随仓附带。
- **Broadcom 蓝牙/Wi-Fi 相关驱动作者**（BrcmPatchRAM / AirportBrcmFixup）：让博通卡在 macOS 下活过来的人。
- 当年一起折腾 H81 的网友与论坛前辈：排错帖救过作者很多次。

---

**初中生的旧 EFI，给同代 Haswell 台式机留个能开机的起点。**

[English Version](/README.en.md)
