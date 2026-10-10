# UPerfect QSL02 Custom Firmware

[한국어 (README.md)](README.md)

> [!CAUTION]
> **⚠️ WARNING & DISCLAIMER**
>
> 1. **Target Hardware Compatibility**:
>    * This firmware is exclusively intended for the **UPERFECT QSL02** (16-inch 2560×1600) portable monitor.
>    * You must physically disassemble the device and verify that the mainboard scaler chipset is **Realtek RTD2775QT** and the LCD panel is **CSOT MNG007DA1-Q** before flashing. (While physically marked as RTD2775QT, the runtime register architecture and operation match the full-spec RTD2795T series supporting 4-lane eDP and frame buffer rotation.)
>    * Flashing this firmware onto devices with different revisions, different scaler chipsets, or different display panels may result in a blank display, timing distortion, or permanent hardware damage.
> 2. **Flashing Tool Requirement**:
>    * Flashing the firmware requires **Realtek MonitorCustomerTool V1.6** (or a compatible Realtek ISP / programmer tool).
>    * This tool is not distributed in this repository; users must obtain it independently.
> 3. **Limitation of Liability**:
>    * This is an unofficial, experimental custom firmware created for personal research and bug mitigation.
>    * **The author assumes NO responsibility or liability** for any device malfunction, bugs, data loss, hardware damage, or bricking resulting from flashing or using this firmware.
>    * All flashing and usage are performed entirely at your own risk. Always make a verified backup of your original firmware dump and ensure you have hardware recovery equipment (such as a CH341A SPI programmer) ready before proceeding.

This directory contains experimental custom firmware for the UPerfect QSL02 2560x1600 portable monitor. The hardware identification in this repository is based on one disassembled test device; revisions and component substitutions are possible.

## Hardware Specification Correction (Advertised/Stock vs. Actual Hardware)

This table describes one disassembled test device and the corresponding panel datasheet. Other manufacturing revisions may differ.

| Parameter | Manufacturer Claim / Stock ROM Data | Actual Hardware (Inspection & CSOT Datasheet) | Notes & Corrections |
| :--- | :--- | :--- | :--- |
| **Panel Manufacturer / Model** | BOE (Stock ROM remnant: `NE160QDM`) | **CSOT `MNG007DA1-Q`** | Confirmed by physical panel label upon disassembly |
| **Panel Technology** | Advertised as "QLED" (Quantum Dot) | **LTPS TFT-LCD (FFS mode, Hardware Low Blue Light)** | Neither QLED nor a-Si; Low-Temperature Poly-Silicon (LTPS) process with hardware low blue light filtering |
| **Peak Luminance** | 500 cd/m² | **Typ. 350 cd/m²** (Min 297.5 / Max 402.5 cd/m²) | Datasheet specifies ~350 nits typical (not 500 nits) |
| **Color Gamut** | sRGB 139.4% (Wide gamut claim) | **sRGB 100% (Typ) / 96% (Min)** (CIE1976) | Standard sRGB 100% panel, not a wide-gamut (DCI-P3) display (not 139.4%) |
| **Contrast Ratio** | 1200:1 | **Typ. 1200:1** (Min 1000:1) | Matches claim |
| **Screen Size / Aspect Ratio** | 16-inch, 16:10 (2560×1600) | **16.0-inch, 16:10 (2560×1600)** | Matches claim (Active Area: 344.68 × 215.42 mm) |
| **Max Refresh Rate** | HDMI 120Hz, Type-C 144Hz | **Panel Native 60~165Hz** (Driven by monitor board: Type-C 144Hz, HDMI 130Hz) | Panel hardware natively supports 165Hz; driven at 144Hz/130Hz due to scaler link bandwidth limits |
| **Surface Treatment** | (Unspecified) | **Anti-Glare (3H Polarizer)** | Matte anti-reflective finish |
| **Response Time** | (Unspecified) | **GTG 3ms (OD On) / 5ms (OD Off)** | Datasheet specification (Tr+Tf 9ms) |
| **Power Consumption** | (Unspecified) | **Panel Logic 1.9W + Backlight 3.2W (Total 5.1W Max)** | Low-power design typical of LTPS process |
| **Color Depth** | 1.07B (10-bit) | **1.07B (8-bit + Hi-FRC)** | Native 8-bit panel with dithering (FRC) to achieve 10-bit |
| **Connectivity** | Mini HDMI, USB-C × 2, 3.5mm Audio | **Mini HDMI, USB-C × 2, 3.5mm Audio** | Matches claim |
| **Built-in Speakers** | 8Ω 1W × 2 | **8Ω 1W × 2** | Matches claim |

## Firmware Files

| Filename | Size | SHA256 (First 16 chars) | Intended Use |
| :--- | :---: | :---: | :--- |
| `UPerfect_QSL02_0deg.bin` | 1,048,576 B | `b84c9782e073e858` | Standard 0-degree orientation build |
| `UPerfect_QSL02_180deg.bin` | 1,048,576 B | `7507d72395dfcf25` | 180-degree hardware display rotation build |

Both files were generated following thorough byte-level diffing against the stock dump and `UPerfect_QSL02_DelNVRAM.bin`, with complete EDID and checksum validation. Always verify file checksums and prepare recovery procedures before flashing.

## Common Modifications

### Burn-in Default State

The default Burn-in flag in Bank 2 was modified from `0x022BD2: 0x01 -> 0x00`. This change is designed to disable the firmware's default factory test pattern mode on boot; it does not retroactively erase existing states already written to NVRAM.

### NVRAM Sector Reset

In images with `0xFB000~0xFCFFF` cleared, factory burn-in mode looping, spontaneous settings resets, and boot stalls did not reproduce, and custom settings persist across power cycles. These are confirmed results on physical test hardware with cleared NVRAM; it does not conclusively prove that power-on hour logging (Item `0x0F`) will never regenerate over prolonged usage.

### Complete EDID Reconstruction (Datasheet-Accurate Optical Profile & VESA Compatibility)

The stock EDID contained different physical dimensions and chromaticity values from those used in this reconstruction. The effect of those values can vary by operating system and color-management path.

In this custom firmware, the EDID blocks were reconstructed using values from the official **CSOT MNG007DA1-Q datasheet**:

- **Chromaticity Coordinates**: Injected the datasheet coordinates for primaries (Red: x=0.647, y=0.328; Green: x=0.301, y=0.603; Blue: x=0.142, y=0.054) and White Point (x=0.313, y=0.329, D65). These values may help the operating system select a profile closer to the panel characteristics, but do not guarantee measured color accuracy or sRGB coverage.
- **Physical Dimensions**: Corrected to `34cm × 22cm`, accurately representing the 16.0" 16:10 format (eliminating 16:9 distortion).
- **Naming Clean-up**: Clearly labeled as `UPERFECT USBC` for Type-C ports and `UPERFECT HDMI` for HDMI.
- **Refresh Rates & FreeSync**: Configured 60~144Hz for Type-C and 60~130Hz for HDMI with valid CEA extension descriptors.
- **VESA Established & Standard Timings Restoration**:
  - Restored standard VESA baseline resolutions that were omitted from the raw eDP laptop datasheet profile to match the factory stock ROM behavior.
  - **Established Timings**: Enabled `640×480 @ 60Hz` (IBM VGA / legacy boot), `800×600 @ 60Hz`, and `1024×768 @ 60Hz` (standard UEFI BIOS setup GUI).
  - **Standard Timings**: Registered `1920×1080 @ 60Hz` (modern motherboard Full HD UEFI GUI), `1920×1200 @ 60Hz` (16:10 WUXGA), `1680×1050 @ 60Hz`, `1280×720 @ 60Hz` (HD 720p), etc.
  - Resolves the "No Signal" black screen issue encountered during PC POST and BIOS setup screens over HDMI, while maintaining full compatibility with legacy devices and gaming consoles.

Note that having VRR descriptors in an EDID does not guarantee VRR activation across all GPUs or input interfaces. While Type-C DP Alt Mode demonstrated working VRR under both Windows and macOS, VRR / G-SYNC did not activate on the tested NVIDIA HDMI setup. The exact limitation was not isolated to EDID alone.

The HDMI EDID also includes a fixed DTD at approximately 129.997Hz, which may appear as 130Hz in Windows. The FreeSync upper limit is also aligned to 130Hz, providing complete VRR coverage across both 120Hz and 130Hz modes.

## Differences: 0-Degree vs. 180-Degree

The two firmware binaries differ by exactly 17 bytes:

| File Offset | Bank & Address | 0° Value | 180° Value | Verified Meaning |
| :--- | :--- | :---: | :---: | :--- |
| `0x022BC4` | Bank 02: `0x2BC4` | `0x0A` | `0x1A` | Rotation bit altered in default user settings structure |
| `0x05FD5B~0x05FD6C` | Bank 05: `0xFD5B` | Stock rotation query | Returns rotation degree 2 (180°) | Forces 180° scaler pipeline upon boot & signal input |

In the 180-degree build, the main display image rotates 180 degrees on physical hardware. The OSD menu engine is intentionally kept unrotated, preserving all menu icons, progress bars, and graphic UI elements intact without visual corruption. These findings were validated on specific test hardware and input conditions, and do not guarantee compatibility across all conceivable panel revisions or timing modes.

While earlier analysis identified Bit 7 of Page 20 registers as H-Flip, direct register manipulation was not used in the final 180-degree implementation. Direct writes to `0x20FB` were rejected (ErrorCode `0x207`) and thus excluded from the patching strategy.

## Verified Hardware Behaviors

- Settings persist across power cycles and 24h cold reboots after NVRAM clearing
- No automatic entry into factory burn-in test mode observed
- Type-C: Verified connection and 60~144Hz VRR operation on both Windows and macOS
- HDMI: Verified connection and accurate color rendering on both Windows and macOS
- HDMI: Verified fixed refresh rate options for 60.01Hz, 120Hz, and ~130Hz
- HDMI: NVIDIA driver reports VRR / G-SYNC as unsupported
- 180-degree build: Display image rotates 180°, OSD graphics remain clean and uncorrupted

## Unconfirmed / Pending Items

- Whether NVIDIA VRR / G-SYNC can ever be activated over this HDMI 2.0 interface via EDID manipulation alone
- Whether power-on hour logging Item `0x0F` will reappear after extended runtime
- General compatibility across every GPU architecture, cable variety, sleep/wake cycle, or rapid port-switching scenario

Therefore, these images represent custom experimental firmwares validated on specific hardware, not official commercial releases. Please preserve your original factory dump and ensure recovery equipment is available prior to flashing.
