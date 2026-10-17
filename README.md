# Awesome RTX Spark [![Awesome](https://awesome.re/badge.svg)](https://awesome.re)

> Native Linux, drivers, kernels, inference and reproducible measurements for NVIDIA RTX Spark (N1X) laptops and related GB10 systems.

This is an independent community list. It is not affiliated with or endorsed by NVIDIA or any laptop vendor. Per-model compatibility lives in [docs/linux.md](docs/linux.md); the measurement submission format, reference numbers and proposed experiments live in [docs/benchmarks.md](docs/benchmarks.md).

**Platform tags** — every entry states what it was actually built or tested on:

- **N1X Linux** — native Linux on an RTX Spark/N1X laptop.
- **WoA** — Windows on Arm (ARM64) on an N1X laptop.
- **WSL2** — Linux under WSL2 on an N1X laptop.
- **GB10** — GB10 desktops such as DGX Spark and OEM equivalents. Shared architecture alone does not make these resources work on N1X laptops.

## Contents

- [Status](#status)
- [Hardware & Firmware](#hardware--firmware)
- [Linux](#linux)
- [Kernels & Drivers](#kernels--drivers)
- [Inference](#inference)
- [Tests & Benchmarks](#tests--benchmarks)
- [Experiments](#experiments)
- [People & Related Lists](#people--related-lists)

## Status

*Last reviewed 2026-10-17.*

- NVIDIA's public RTX Spark developer material targets Windows on Arm; its own CI matrix lists native N1X Linux as deferred and says not to claim it as supported.
- Native N1X Linux exists only as community bring-up: Omarchy's aarch64 edge channel ships a patched 7.2.8 kernel and `nvidia-open` 615.78.08, tested on one ASUS ProArt P14 and one Dell XPS 16.
- Most N1X platform patches (EC, USB4, audio, pin control, power) are not upstream, and the Linux driver's published device list names GB10 but no N1X device IDs.
- GB10 desktops have mature Linux support from NVIDIA and several community distros; treat their results as context, not as N1X evidence.

Known N1X Linux limitations (evidence links are in the compatibility notes):

- Firmware reserves about 62.5 GiB of RAM for the GPU by default; Omarchy lowers this to 4 GiB on the ProArt P14, but the same setting is unconfirmed on the XPS 16.
- Only s2idle works (about 0.77 W overnight on the ProArt); `deep` sleep returns immediately and RTC wake alarms do not fire.
- The XPS 16 lacks AC/battery change events on BIOS 1.2.0, and its brightness depends on an out-of-tree NVIDIA driver patch.
- Kernels stay on 4 KiB pages because of a 64 KiB driver fault reported on GB10; 64 KiB has not been tested on N1X.
- There is no public data yet for active battery drain, fan control, NVMe throughput, eGPU over USB4, Nouveau or any laptop besides the two above.

## Hardware & Firmware

- [Windows on Arm Porting Guide](https://docs.nvidia.com/rtx-spark/rtx-spark-porting-guide/latest/) - **WoA** NVIDIA's official N1X reference for CPU topology, GPU device IDs, unified memory, NVML power data and Arm64EC porting.
- [RTX Spark Developer Preview](https://forums.developer.nvidia.com/t/rtx-spark-developer-preview/377106) - **WoA** NVIDIA forum announcement listing the preview driver, CUDA toolkit and known issues.
- [spark_hwmon](https://github.com/antheas/spark_hwmon) - **GB10** Reverse-engineered hwmon driver that exposes firmware power rails, energy counters, temperatures and power limits *(experimental, firmware-dependent)*.
- [The Energy Blind Spot](https://arxiv.org/abs/2605.27599) - **GB10** Paper auditing which energy measurements GB10 firmware telemetry can and cannot support.
- [dgx-spark-fan-control](https://github.com/christopherowen/dgx-spark-fan-control) - **GB10** Kernel driver and CLI that enforce a minimum fan speed on top of the stock EC curve.
- [gb10-clock-cap](https://github.com/agjs/gb10-clock-cap) - **GB10** Measured GPU clock cap that mitigates hard power-offs under sustained load at a small decode cost.
- [dgx-spark-networking](https://github.com/christopherowen/dgx-spark-networking) - **GB10** Switchless ConnectX-7 clustering of two to four systems with a low-latency RDMA all-reduce and patched NCCL.

## Linux

Native N1X laptops:

- [Omarchy N1x support](https://github.com/omacom/omarchy/pull/14820) - **N1X Linux** Merged runtime support for the ProArt P14 and XPS 16, covering detection, USB4, audio profiles, power profiles and GPU memory reservation.
- [linux-omarchy-aarch64](https://github.com/omacom/omarchy-pkgs/blob/master/pkgbuilds/linux-omarchy-aarch64/README.md) - **N1X Linux** Kernel package notes explaining every N1X patch, with dated hardware validation logs.
- [Upgrading to 7.3](https://github.com/omacom/omarchy-pkgs/blob/master/pkgbuilds/linux-omarchy-aarch64/UPGRADING-7.3.md) - **N1X Linux** Per-patch upstream status, open problems and a verification checklist for the next kernel rebase.
- [Omarchy on ARM](https://github.com/omacom/omarchy/discussions/15702) - **N1X Linux** Hub for the aarch64 ISO and edge channel, open for per-laptop reports.
- [MT8901 UCM profiles](https://github.com/alsa-project/alsa-ucm-conf/pull/862) - **N1X Linux** NVIDIA's pending ALSA profiles for the SoundWire audio found in RTX Spark laptops *(unmerged)*.
- [NemoClaw platform matrix](https://github.com/NVIDIA/NemoClaw/blob/main/ci/platform-matrix.json) - **N1X Linux / WSL2 / GB10** NVIDIA CI matrix that records which platforms are tested, caveated or deferred.

GB10 desktops:

- [nixos-dgx-spark](https://github.com/graham33/nixos-dgx-spark) - **GB10** NixOS module and installer image built on NVIDIA's kernel, with documented firmware and stability caveats.
- [ubuntu-gb10](https://github.com/timothystewart6/ubuntu-gb10) - **GB10** Guide and Ansible roles for running stock Ubuntu with NVIDIA drivers, ConnectX-7 and multi-node NCCL instead of DGX OS.
- [spark-rocky](https://github.com/maxspevack/spark-rocky) - **GB10** Unpatched Rocky Linux live image with CI-enforced zero source modifications *(personal, unsupported)*.
- [kindling-spark-os](https://github.com/kindlingai/kindling-spark-os) - **GB10** Read-only image booted alongside DGX OS with automatic rollback and extra model memory from 64 KiB pages *(beta)*.

## Kernels & Drivers

Linux kernels and GPU drivers only; CUDA compute kernels belong under Inference.

- [NV-Kernels](https://github.com/NVIDIA/NV-Kernels) - **GB10 / N1X Linux** NVIDIA's Ubuntu kernel trees with 4 KiB and 64 KiB flavours and board-tested N1X platform patches.
- [open-gpu-kernel-modules](https://github.com/NVIDIA/open-gpu-kernel-modules) - **GB10** Open NVIDIA kernel modules; Omarchy patches them for N1X, but the published support list names only GB10.
- [64 KiB page mapping faults](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/1269) - **GB10** Open report of Xid 31 faults on large GPU mappings with 64 KiB kernels and newer drivers, which is why N1X ports stay on 4 KiB.
- [dgx-spark-memory-saver](https://github.com/christopherowen/dgx-spark-memory-saver) - **GB10** Source patch to `nvidia-uvm` that packs GPU page tables on 64 KiB kernels and measures the memory recovered per node.
- [Nouveau on GB10](https://airlied.blogspot.com/2026/09/nouveau-on-nvidia-spark-gb10-its-alive.html) - **GB10** Dave Airlie's report on booting Nouveau with display and NVK, with no upstream plan yet and no N1X testing.
- [nouveau-610-wip-spark](https://gitlab.freedesktop.org/nouvelles/kernel/-/commits/nouveau-610-wip-spark) - **GB10** Work-in-progress Nouveau kernel branch from that report *(experimental)*.
- [nvk-spark-wip](https://gitlab.freedesktop.org/airlied/mesa/-/commits/nvk-spark-wip) - **GB10** Matching work-in-progress Mesa branch for NVK *(experimental)*.

## Inference

- [RTX Spark playbooks](https://build.nvidia.com/rtx-spark) - **WoA** NVIDIA step-by-step guides, including a native Windows ARM64 CUDA build of llama.cpp.
- [local-ai-inference-recipes](https://github.com/NVIDIA/local-ai-inference-recipes) - **WoA / WSL2 / GB10** NVIDIA recipes per device, with Linux engines on RTX Spark limited to WSL2 *(work in progress)*.
- [pytorch-windows-ci](https://github.com/NVIDIA/pytorch-windows-ci) - **WoA** NVIDIA-published CUDA PyTorch wheels for Windows ARM64.
- [TensorRT-RTX](https://github.com/NVIDIA/TensorRT-RTX) - **WoA / GB10** Inference library whose release notes add Windows on Arm for RTX Spark and Linux SBSA for DGX Spark.
- [DGX Spark playbooks](https://github.com/NVIDIA/dgx-spark-playbooks) - **GB10** Official playbooks for vLLM, SGLang, TensorRT-LLM, NVFP4 and speculative decoding.
- [llama.cpp](https://github.com/ggml-org/llama.cpp) - **WoA / WSL2 / GB10** Ships Linux and Windows ARM64 CUDA builds with native FP4 paths for sm_121.
- [vLLM](https://github.com/vllm-project/vllm) - **WSL2 / GB10** Serving engine with aarch64 wheels covering sm_121, plus open GB10 issues on NVFP4, FP8 KV cache and speculative decoding.
- [SGLang](https://github.com/sgl-project/sglang) - **GB10** Serving engine with aarch64 wheels; check its open multi-GPU correctness issues before trusting results.
- [spark-vllm-docker](https://github.com/eugr/spark-vllm-docker) - **GB10** eugr's tested vLLM images and recipes for one to eight Sparks, mostly NVFP4 with speculative decoding.
- [sparkrun](https://github.com/spark-arena/sparkrun) - **GB10** CLI that launches vLLM, SGLang or llama.cpp recipes across Sparks and feeds the Spark Arena leaderboard, with telemetry on by default.
- [FlashInfer b12x](https://github.com/flashinfer-ai/flashinfer/tree/main/flashinfer/experimental/b12x) - **GB10** SM120/SM121 CUDA compute kernels for NVFP4/MXFP4/FP8 GEMM, MoE, attention and disk-backed embedding tables, which replace the archived [b12x](https://github.com/local-inference-lab/b12x) *(experimental)*.
- [SparkLab](https://github.com/sixteen-miles-labs/sparklab) - **GB10** Single-box engine that plans the unified-memory budget and streams MoE experts from NVMe *(beta, self-reported numbers)*.

## Tests & Benchmarks

- [llama-bench](https://github.com/ggml-org/llama.cpp/tree/master/tools/llama-bench) - Repeatable prefill and decode measurements with context depth, repetitions and JSON output.
- [llama.cpp on DGX Spark](https://github.com/ggml-org/llama.cpp/discussions/16578) - **GB10** Maintainer-run llama-bench results across models and context depths, kept current in the repository.
- [dgx-spark-bench](https://github.com/jvr0x/dgx-spark-bench) - **GB10** Concurrency sweep harness for OpenAI-compatible servers with pinned versions, cache-busting prompts and schema-checked results.
- [nvbandwidth](https://github.com/NVIDIA/nvbandwidth) - Copy-engine and kernel-copy bandwidth tests; on unified memory, host-to-device figures are not PCIe numbers.
- [CUDA Samples](https://github.com/NVIDIA/cuda-samples) - Provides `deviceQuery` and `bandwidthTest` for a quick sanity check of driver, toolkit and architecture.
- [nccl-tests](https://github.com/NVIDIA/nccl-tests) - **GB10** Collective bandwidth and latency tests for multi-node ConnectX-7 setups.
- [fio](https://github.com/axboe/fio) - Flexible I/O tester for NVMe throughput and latency, but keep test files well below internal-SSD capacity on GB10 systems.
- [Phoronix Test Suite](https://github.com/phoronix-test-suite/phoronix-test-suite) - Automated system benchmarks whose GB10 results are published on OpenBenchmarking.org.
- [Inside GB10's memory subsystem](https://chipsandcheese.com/p/inside-nvidia-gb10s-memory-subsystem) - **GB10** Chips and Cheese measurements of CPU-side latency, bandwidth and contention with the GPU.
- [Surface Laptop Ultra review](https://www.storagereview.com/review/microsoft-surface-laptop-ultra-for-business-review-rtx-spark-n1x) - **WoA** StorageReview's N1X laptop measurements of memory, SSD and local LLM performance, including load-induced power-offs.

## Experiments

Documented investigations with their own evidence; findings are the authors' and are not reproduced here.

- [moe-offload-findings](https://github.com/GaelicThunder/moe-offload-findings) - **GB10** Measured lessons from streaming a 744B MoE from NVMe, including where speculative decoding hurts.
- [External GPU over OCuLink](https://github.com/chappa-ai-llc/spark-smi/issues/1) - **GB10** EvilTCell's write-up of attaching an RTX 3090 through the M.2 slot, with link-training failures and cooling results.
- [Qwen3.8-27B study](https://forums.developer.nvidia.com/t/comprehensive-qwen3-8-27b-study-on-dgx-sparks-quantization-speculative-decoding-and-tp-dp-scaling/381102) - **GB10** Community comparison of BF16, FP8 and NVFP4 with and without MTP across one to four Sparks.
- [cuda-blackwell-carry-bug](https://github.com/Th0rgal/cuda-blackwell-carry-bug) - **GB10** Case study of a carry chain split across inline PTX statements breaking on sm_12x, which PTX does not guarantee and NVIDIA has not confirmed as a compiler bug *(repro needs fixes)*.

## People & Related Lists

- [awesome-dgx-spark](https://github.com/bidual/awesome-dgx-spark) - **GB10** bidual's broad collection of DGX Spark models, recipes, tools and multi-node guides.
- [ryanrhughes](https://github.com/ryanrhughes) - Led Omarchy's N1X support and the ProArt P14 bring-up and validation.
- [spencerbull](https://github.com/spencerbull) - Started Omarchy's N1X work and brought up the Dell XPS 16, including battery and OLED brightness patches.
- [emirb](https://github.com/emirb) - Moved Omarchy's aarch64 kernel builds to CI and added aarch64 install tests.
- [birkskyum](https://github.com/birkskyum) - Built Omarchy's single aarch64 ISO and maintains its ARM hub.
- [Dave Airlie](https://github.com/airlied) - Ported Nouveau and NVK to GB10.
- [graham33](https://github.com/graham33) - Maintains NixOS for GB10 and the kernel configuration other ports reuse.
- [Christopher Owen](https://github.com/christopherowen) - Publishes GB10 fan, memory and switchless networking tools measured on a four-node cluster.
- [eugr](https://github.com/eugr) - Maintains spark-vllm-docker and joined NVIDIA's Spark team.
- [EvilTCell](https://github.com/EvilTCell) - Builds spark-smi and documented GB10 eGPU and cooling modifications.
- [Antheas Kapenekakis](https://github.com/antheas) - Reverse-engineered GB10 power telemetry for spark_hwmon.

## Footnotes

This list started as a fork of [sindresorhus/awesome](https://github.com/sindresorhus/awesome) and keeps its Git history and CC0 license. See [CONTRIBUTING.md](CONTRIBUTING.md) to add a resource or report hardware results.
