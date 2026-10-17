# Verified picks from awesome-dgx-spark

*Reviewed 2026-10-17.* [awesome-dgx-spark](https://github.com/bidual/awesome-dgx-spark) by bidual (CC0) lists about 370 DGX Spark resources. Only one of them mentions N1X or RTX Spark at all, so this page records which ones were checked for this list and why they were kept or left out.

**Selection rule:** a resource is considered if it is about the architecture (aarch64, sm_121, unified memory, drivers, power, measurement) rather than one model recipe, and if its claims are backed by code or data in the repository. Model checkpoints, single-model deployment recipes, multi-node model ports, media and application stacks were not reviewed.

**Compatibility:** every resource below was measured on GB10 desktops (DGX Spark, ASUS Ascent GX10 or similar). None has evidence on an N1X laptop, Windows on Arm or WSL2 on N1X. "Generic" means it depends only on aarch64 Linux, sm_121 and NVML/CUDA, so it is a reasonable thing to try on N1X; it does not mean it works there. Anything that touches the embedded controller, the desktop power brick, `spark_hwmon` or ConnectX does not carry over to laptops.

## Added to the README

| Resource | Scope | Why it was kept | Caveat |
|---|---|---|---|
| [nv-monitor](https://github.com/wentbackward/nv-monitor) | Generic | Single C binary on NVML with CI, soak and meminfo tests; screenshots across GB10, Jetson and x86. | Only WSL evidence is a merged fix that hides missing CPU sysfs data. |
| [spark-doctor](https://github.com/joeynyc/spark-doctor) | GB10, partly generic | Read-only, redacts by default, about 20 test files and CI. | Some rules only fire when the GPU name contains "GB10"; runs `sudo -n dmidecode` and `fwupdmgr`. |
| [ds4](https://github.com/antirez/ds4) | GB10 | `make cuda-spark` builds for sm_121 with MXFP4 kernels; docs mark single runs versus medians. | Two-node mode needs a direct ConnectX link; "DSpark" in ds4 is a drafter, not DGX Spark. |
| [sm121-kernels](https://github.com/blake-snc/sm121-kernels) | Generic | 259 PTX kernels with warm-up plus 100–200 timed iterations and medians. | GB10 clocks cannot be locked; competitor baselines are not reproducible from the repo; NCCL layer unvalidated. |
| [nvfp4bench](https://github.com/secYOUre/nvfp4bench) | Generic | Real probes with an FP32 correctness checker. | [Issue #1](https://github.com/secYOUre/nvfp4bench/issues/1) questions the peak kernel; part of the README is stale; no raw results committed. |
| [vLLM NVFP4 KV-cache repro](https://github.com/jethac/vllm-nvfp4-kv-consumer-blackwell-repro) | Generic | Model-free repro with committed evidence. | Committed log is from an RTX 5090; GB10 run described but not logged. Upstream [vLLM#50084](https://github.com/vllm-project/vllm/issues/50084) open; fix [PR #55976](https://github.com/vllm-project/vllm/pull/55976) unmerged. |
| [vLLM spin-wait fix](https://github.com/drowzeys/vllm-gb10-spin-wait-fix) | Generic (vLLM) | Honest write-up: the −24 °C figure is from a two-node GX10 TP=2 setup, and the author's own TP=1 A/B found no change. | Benefit on a single-GPU laptop is likely small; GPU clock capping helped more in the same notes. |

## Worth knowing, not in the README

| Resource | Scope | Notes |
|---|---|---|
| [sparkview](https://github.com/parallelArchitect/sparkview) | GB10 | Unified-memory and PSI-aware monitor; thresholds and power row assume the desktop power brick and `spark_hwmon`; lint-only CI. |
| [spark-gpu-throttle-check](https://github.com/hoesing/spark-gpu-throttle-check) | GB10 | Generic cuBLAS load generator, but its diagnosis (USB-PD power-brick negotiation) is desktop-specific; no tests. |
| [sparkfit](https://github.com/engineering87/sparkfit) | Generic | Memory and roofline calculator with tests; DGX defaults must be overridden for other memory sizes and bandwidths. |
| [gb10-toolboxes](https://github.com/kyuz0/gb10-toolboxes) | Generic | Automated arm64 CUDA 13 images of llama.cpp, ds4 and vLLM for sm_121; no license, no published results. |
| [dgx-spark-setup](https://github.com/natolambert/dgx-spark-setup) | Generic | Measured training memory tables from January 2026; version advice is likely dated; no license. |
| [dgx-spark-ml-guide](https://github.com/martimramos/dgx-spark-ml-guide) | Generic | Useful PyTorch/NVRTC/aarch64 failure-to-fix table; no LICENSE file and an inconsistent changelog. |
| [gb10-uma-inference-notes](https://github.com/GaelicThunder/gb10-uma-inference-notes) | GB10 | Short unified-memory claims; evidence lives in sibling repos such as [moe-offload-findings](https://github.com/GaelicThunder/moe-offload-findings). |
| [dgx-spark-kv-cache-benchmark](https://github.com/Memoriant/dgx-spark-kv-cache-benchmark) | GB10 | llama.cpp KV-cache type comparison with raw CSVs and a published correction; corrected table is a single run. |
| [spark-bench](https://github.com/Weschera/spark-bench) | Engine-agnostic | Strong agentic-task methodology with thousands of committed results, but it rates models and recipes, not hardware. |
| [vllm-gemma4-nvfp4-kv-repro](https://github.com/jethac/vllm-gemma4-nvfp4-kv-repro) | Generic | Further NVFP4 KV-cache bugs with raw outputs and patched aarch64 wheels; several upstream fixes still open. |
| [TensorFold](https://github.com/ashhart/TensorFold) | GB10 | Ships a linux-aarch64 build, but the qualified CUDA model list is narrow and Windows support is x86_64 only. |
| [llm-engine-benchmark](https://github.com/ramsred/llm-engine-benchmark) | GB10 | vLLM/SGLang/TensorRT-LLM long-context comparison; cross-engine results are preliminary single runs. |
| [DGXSparkGaming](https://github.com/AtomicGaryBusey/DGXSparkGaming) | GB10 | Linux gaming log on DGX Spark. Its notes say the Windows on Arm display driver binds only N1X device IDs, not GB10's; unverified here. |

## Left out

| Resource | Reason |
|---|---|
| [dgx-spark-fan-override](https://github.com/Z841973620/dgx-spark-fan-override) | Talks to the DGX Spark EC through a hard-coded FF-A mailbox; must not be tried on a laptop EC. |
| [dgx_spark_config](https://github.com/GuigsEvt/dgx_spark_config) | GEMM benchmark lacks synchronization, sample output reports 128 SMs (GB10 has 48), invalid commands, unverified prebuilt wheels, stale. |
| ICE-CoreOS and ainode | Repositories no longer exist. |
