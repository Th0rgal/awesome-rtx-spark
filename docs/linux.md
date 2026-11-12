# Linux compatibility

*Last reviewed 2026-10-17.* This page records **only evidenced** results. Each row names its source; "?" means no public evidence. Send corrections or new reports with the [hardware report form](https://github.com/th0rgal/awesome-rtx-spark/issues/new?template=hardware-or-benchmark-report.yml).

Legend: **Yes** evidenced working · **Partial** works with caveats or patches · **No** evidenced failing · **?** unknown.

## Platform overview

| Platform | Status | Evidence |
|---|---|---|
| Native Linux on N1X laptops | Community bring-up only (Omarchy edge channel). NVIDIA lists it as *deferred* and says not to claim N1X as supported. | [omarchy#14820](https://github.com/omacom/omarchy/pull/14820), [NemoClaw platform matrix](https://github.com/NVIDIA/NemoClaw/blob/main/ci/platform-matrix.json) |
| Windows on Arm (ARM64) on N1X | NVIDIA's supported developer platform: Windows driver 616.41 or later, CUDA 13.4 Update 2. | [Porting guide](https://docs.nvidia.com/rtx-spark/rtx-spark-porting-guide/latest/), [CUDA release notes](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/) |
| WSL2 on N1X | NVIDIA's documented route to Linux engines on RTX Spark; caveated and without CI in NemoClaw. | [local-ai-inference-recipes](https://github.com/NVIDIA/local-ai-inference-recipes), [NemoClaw platform matrix](https://github.com/NVIDIA/NemoClaw/blob/main/ci/platform-matrix.json) |
| GB10 desktops (DGX Spark and OEM boxes) | Supported by NVIDIA (DGX OS); several community distros. **Not evidence for N1X laptops.** | [NV-Kernels](https://github.com/NVIDIA/NV-Kernels), [GB10 desktops](#gb10-desktops-for-context) |

NVIDIA also runs internal N1X Linux images ("N1x FASTOS" on Ubuntu 24.04 with kernel 7.0.0-1015-nvidia, and "Concept ISO 1.0.2" on Ubuntu 26.04.1 with 7.0.0-2020-nvidia-bos). They appear only in NemoClaw issues such as [#8096](https://github.com/NVIDIA/NemoClaw/issues/8096) and [#12737](https://github.com/NVIDIA/NemoClaw/issues/12737), on unnamed laptops, and have no public download.

## Software stack used by the evidenced N1X ports

- **Kernel:** `linux-omarchy` for aarch64, 7.2.8 (pkgrel up to 11), with NVIDIA's NV-Kernels N1X patches (`1021.21`/`1022.23` series) plus Omarchy patches; NVIDIA's 7.0 `linux-n1x` kernel stays as a fallback boot entry. See the [package README](https://github.com/omacom/omarchy-pkgs/blob/master/pkgbuilds/linux-omarchy-aarch64/README.md) and [omarchy-pkgs#930](https://github.com/omacom/omarchy-pkgs/pull/930).
- **Page size:** 4 KiB (`CONFIG_ARM64_4K_PAGES=y`). 64 KiB is avoided because of [open-gpu-kernel-modules#1269](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/1269), which was reported on GB10, not N1X.
- **GPU driver:** `nvidia-open-dkms` 615.78.08 with Omarchy patches. The upstream [driver README](https://github.com/NVIDIA/open-gpu-kernel-modules) lists GB10 (`2E12`) but none of the N1X IDs (`2E03`, `2E06`, `2E13`). Nouveau has not been tested on N1X.
- **Boot:** ACPI (no device tree), Limine with a unified kernel image.
- **Upstream status:** most N1X platform code (MT8901 pin control, I2C, SoundWire/ASoC, FF-A EC, power_wrap, USB4 platform NHI, GPU IOMMU quirk, DRIPS/ASPM) is out of tree; see [UPGRADING-7.3.md](https://github.com/omacom/omarchy-pkgs/blob/master/pkgbuilds/linux-omarchy-aarch64/UPGRADING-7.3.md). The ALSA UCM profiles are pending in [alsa-ucm-conf#862](https://github.com/alsa-project/alsa-ucm-conf/pull/862).

## ASUS ProArt P14 (H7407BA)

Single tester: ryanrhughes, 2026-10-02 to 2026-10-14. Sources: the [package README](https://github.com/omacom/omarchy-pkgs/blob/master/pkgbuilds/linux-omarchy-aarch64/README.md) validation log, [omarchy#14820](https://github.com/omacom/omarchy/pull/14820), [omarchy-pkgs#930](https://github.com/omacom/omarchy-pkgs/pull/930), [omarchy-pkgs#991](https://github.com/omacom/omarchy-pkgs/pull/991) and [n1x.sh](https://github.com/omacom/omarchy/blob/quattro/install/hardware/n1x.sh).

| Item | Status | Notes |
|---|---|---|
| BIOS | ? exact version | `n1x.sh` says BIOS H7407BA.302 and later can boot the GPU firmware; the validated version is not stated. |
| Kernel / driver | 7.2.5 → 7.2.8-11; 615.71.09 → 615.78.08 | Patched kernel and driver as above. |
| Boot, internal panel | Yes | Early KMS on the NVIDIA GPU; encrypted-disk prompt works. |
| Brightness | Yes | |
| GPU compute (CUDA) | Yes | Includes a 16 GiB CUDA workload. |
| Suspend | Partial | s2idle works and reaches DRIPS; `deep` returns immediately; RTC/TAD wake alarms do not wake it. |
| Battery drain | Partial | 0.77 W over a 7.3 h night in s2idle (about 6.7 W before the DRIPS/NVMe L1.2 patches); active drain ?. |
| Thermals / fans | ? | Thermal-zone patch intentionally not applied until P14 zones are checked. |
| Wi-Fi / Bluetooth | Yes / Yes | Wi-Fi returns about 9 s after resume. |
| Audio | Yes (patched) | Speakers, headphones, mics and HDMI audio; needs ASUS amplifier tuning firmware that is not in linux-firmware. |
| Webcam | Yes | Listed as working; no further detail. |
| Keyboard / touchpad | Yes (patched) | `hid-asus` patches for Fn keys; libinput pressure-pad quirk for the touchpad. |
| USB4 / Thunderbolt | Yes (patched) | CalDigit TS4 at 40 Gb/s on all three ports, 2.5 GbE, two monitors, hotplug and docked s2idle; needs extra kernel parameters set by `n1x.sh`. |
| eGPU | ? | PCIe tunnelling works, but no eGPU has been tested. |
| NVMe performance | ? | No measurements. |
| Visible memory | Partial | Firmware reserves 62.5 GiB for the GPU, leaving about 61 GiB; setting the reservation to 4 GiB gives about 122.5 GiB. |
| Not validated | — | Warm-reboot loops; shutdown on 7.3. |

## Dell XPS 16 (DX16263)

Single tester: spencerbull (Dell), 2026-09-11 to 2026-10-04, on a unit described as a development unit. Sources: [omarchy#14214](https://github.com/omacom/omarchy/pull/14214) and [omarchy-pkgs#795](https://github.com/omacom/omarchy-pkgs/pull/795) (both superseded by #14820/#930, which carry the work).

| Item | Status | Notes |
|---|---|---|
| BIOS | 1.2.0 validated | 1.0.4 is the first BIOS where the GPU firmware boots; pre-release 0.x BIOS leaves the GPU disabled and rendering in software. |
| Kernel / driver | 7.2.5 (`linux-omarchy-n1x`); 610.57.04 → 615.71.09 | Current Omarchy builds use the shared 7.2.8 package; no newer Dell validation published. |
| Boot, internal panel | Yes | 1920x1200 at 120 Hz on BIOS 1.0.4. |
| Brightness | Partial | Works only with an out-of-tree `nvidia-open` luminance patch. |
| GPU compute (CUDA) | ? | GPU firmware boots; CUDA not tested. |
| Suspend | Partial | s2idle cycles work; lid suspend untested; `deep` fails and can wedge the HDMI audio controller. |
| Battery | Partial | Correct readings need a kernel patch; no AC/battery change events on BIOS 1.2.0; drain ?. |
| Thermals / fans | Partial / ? | 23 thermal zones read sanely with a patch; fans ?. |
| Wi-Fi / Bluetooth | ? / ? | Not reported. |
| Audio | Partial | Speaker output and mics work; the headset mic is missing with NVIDIA's UCM profiles. |
| Webcam | ? | Not reported. |
| Keyboard / touchpad | Partial / ? | Mic-mute and brightness keys work through an EC-polling helper; touchpad not reported. |
| USB4 / Thunderbolt | Partial | Three Thunderbolt domains bind and a TS4 dock was used; eGPU ?. |
| NVMe performance | ? | No measurements. |
| Visible memory | Partial | About 61 GiB stock; 119.5 GiB with an earlier reclaim workaround that corrupted memory with a 5K monitor until narrowed. The 4 GiB reservation setting is not separately confirmed on Dell. |

## Microsoft Surface Laptop Ultra

Two testers: teo-casares (from 2026-10, UEFI 7.1.0, Project Mu firmware) and ikegam (from 2026-10). Both run Omarchy aarch64 with out-of-tree modules on top of `linux-omarchy`. Sources: [teo-casares's notes](https://gist.github.com/teo-casares/8b64949638e1952602a7f435bb34d5b8), [omarchy discussion #15885](https://github.com/omacom/omarchy/discussions/15885), [omarchy#15888](https://github.com/omacom/omarchy/pull/15888), [surface-ultra-misc](https://github.com/ikegam/surface-ultra-misc) and [surface-ultra-camera](https://github.com/ikegam/surface-ultra-camera).

| Item | Status | Notes |
|---|---|---|
| Boot | Partial | Limine resets the machine before drawing anything ([Limine#694](https://github.com/Limine-Bootloader/Limine/issues/694)); GRUB chainloading the Omarchy UKIs works. Needs `acpi_osi=!`. |
| Internal panel / GPU | Partial (patched) | The GPU carveout is missing from the EFI memory map and nvidia-drm oopses without an out-of-tree memmap module ([open-gpu-kernel-modules#1458](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/1458)). |
| GPU clocks | Yes | No 754 MHz floor under sustained load, unlike the P16 and HP ([open-gpu-kernel-modules#1521](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/1521)). |
| Brightness | Partial | No `/sys/class/backlight`; set through the GPU's eDP luminance from userspace. |
| GPU compute (CUDA) | Yes | |
| Battery / AC / fan (SAM) | Yes (patched) | The SAM UART needs its 52 MHz clock and the `MSHW0733` clients, both out of tree ([linux-surface#2307](https://github.com/linux-surface/linux-surface/issues/2307)). |
| Thermals | Partial | No ACPI trip points; userspace control. |
| Suspend | Partial | s2idle with `NVreg_EnableS0ixPowerManagement=1`; fails with a dock attached; DRIPS residency ?. |
| Audio | Partial (patched) | Speakers and internal mics with out-of-tree SoundWire changes; jack detection needs rt721-sdca from 7.4; headset mic ?. |
| Webcam | Yes (patched) | RGB and IR cameras with an out-of-tree driver. |
| Keyboard / touchpad | Partial | Delayed key releases; touchpad needs the Win8 multitouch class. |
| Buttons | Yes | Power button; no volume buttons. `soc_button_array` skips this model since `linux-omarchy-aarch64` 7.3-23 ([omarchy-pkgs#2166](https://github.com/omacom/omarchy-pkgs/pull/2166)). |
| Wi-Fi / Bluetooth | Yes / ? | |
| USB4 / Thunderbolt | Partial | Dock with two 4K60 displays; needs a runtime-PM udev rule. |
| NPU | No | The DLA channel never answers ([n1x-npu-linux#30](https://github.com/johannhartmann/n1x-npu-linux/issues/30)). |

## Other N1X laptops

| Model | Status |
|---|---|
| ASUS ProArt P16, HP OmniBook, Lenovo Yoga 9n / Pro 9n, MSI Prestige | **Unknown.** Announced N1X models with no public Linux evidence. |

## GB10 desktops (for context)

These run Linux well but say nothing about laptop firmware, EC, audio, USB4 or battery behaviour.

| Project | Base | Kernel / driver | Caveats |
|---|---|---|---|
| [DGX OS / NV-Kernels](https://github.com/NVIDIA/NV-Kernels) | Ubuntu | 6.17 and 7.0 `linux-nvidia`, 4 KiB and 64 KiB flavours | KHO memory reservation of about 9.3 GiB broke RDMA pinning until disabled. |
| [nixos-dgx-spark](https://github.com/graham33/nixos-dgx-spark) | NixOS | NVIDIA 7.0.14 (4 KiB/64 KiB), R615 | Firmware update must be done from DGX OS first; sustained-load power-off needs a clock cap. |
| [ubuntu-gb10](https://github.com/timothystewart6/ubuntu-gb10) | Ubuntu 24.04 | 6.17.0-1032-nvidia, 580.178.04 | Written on an ASUS GX10. |
| [spark-rocky](https://github.com/maxspevack/spark-rocky) | Rocky 10.2 | CIQ 6.18, 610.57.04 | Personal and unsupported; one DGX Spark tested. |
| [kindling-spark-os](https://github.com/kindlingai/kindling-spark-os) | Ubuntu | 7.0 NVIDIA, 64 KiB, 580.178.04 | Beta; vLLM needs eager safetensors loading on 64 KiB kernels. |

Known GB10 driver problems: 64 KiB-page mapping faults ([#1269](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/1269)) and host hangs after unified-memory exhaustion ([#1358](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/1358)).

## Evidence gaps

- An independent second report for either laptop; every N1X row above comes from one developer per model.
- The exact ProArt P14 BIOS version, and whether newer Dell BIOS releases restore EC power notifications.
- Active-use battery drain, fan control, NVMe throughput and eGPU behaviour on any N1X laptop.
- 64 KiB-page kernels and Nouveau on N1X.
- Any native Linux report for the Surface Laptop Ultra or other N1X models.
