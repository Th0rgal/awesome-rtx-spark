# Benchmarks

*Last reviewed 2026-10-17.* This page defines how to submit measurements, lists published reference numbers with their sources, and keeps proposed experiments separate. **No results on this page were produced by this list's maintainers.**

## Submission format

Open a [benchmark report](https://github.com/th0rgal/awesome-rtx-spark/issues/new?template=hardware-or-benchmark-report.yml) and fill in this block. Raw output is required; summaries alone are not accepted.

```yaml
platform: N1X Linux | WoA | WSL2 | GB10          # one of the list's platform tags
hardware: ASUS ProArt P14 H7407BA, 128 GB         # model, SKU, RAM
firmware: BIOS x.y.z, EC x.y.z
power: AC | battery; power profile; GPU clock cap if any
os: distro + version, `uname -r`, page size (`getconf PAGESIZE`)
software:                                         # exact revisions or commit hashes
  driver: 615.78.08
  cuda: 13.4.2
  engine: llama.cpp b1234 (commit abcdef0) | vLLM 0.31.0 | ...
  flags: full command line
model: name + revision/hash
quantization: NVFP4 | MXFP4 | FP8 | Q4_K_M | ...
context: max context configured, and context depth at measurement
prompt_tokens: 2048
output_tokens: 128
concurrency: 1                                    # parallel requests or batch size
speculative_decoding: none | MTP-n | EAGLE3 | draft model name
results:
  prefill_tok_s: mean ± stddev
  decode_tok_s: mean ± stddev
  ttft_ms: p50 / p95
  itl_ms: p50 / p95                               # inter-token latency
  peak_memory_gib: from nvidia-smi or /proc/meminfo, say which
  power_w: average and source (NVML, hwmon, wall meter)
cache_state: cold (after drop_caches/reboot) | warm; prefix cache on/off
runs: number of repeated runs; warm-up runs discarded
raw: link to raw JSON/log output
```

Rules for comparisons:

- Change one variable at a time and report both runs with identical fields.
- Record whether the model file was in the page cache, whether prefix caching was enabled, and use unique prompt prefixes for serving benchmarks.
- Repeat each measurement (llama-bench `-r 5` or more) and report spread, not just the best run.
- On unified-memory systems, state how memory was measured; GPU "used memory" and system RAM overlap.
- For speculative decoding, report acceptance rate and confirm output matches non-speculative output for the same seed.
- For storage tests with `fio`, keep the test file well below SSD capacity: StorageReview's 1 TB and 128 GB footprints locked up the internal SSD on DGX Spark while 64 GB worked ([review](https://www.storagereview.com/review/nvidia-dgx-spark-review-the-ai-appliance-bringing-datacenter-capabilities-to-desktops)).

Suggested starting commands (see each tool's upstream documentation for details):

- LLM: [llama-bench](https://github.com/ggml-org/llama.cpp/tree/master/tools/llama-bench) with `-p 2048 -n 128 -d 0,8192,32768 -r 5 -o json`.
- Serving: [dgx-spark-bench](https://github.com/jvr0x/dgx-spark-bench) concurrency sweeps against an OpenAI-compatible server.
- Memory: [nvbandwidth](https://github.com/NVIDIA/nvbandwidth) with `-F json`, plus a CPU-side STREAM run.
- Storage: [fio](https://github.com/axboe/fio) sequential 1 MiB and random 4 KiB jobs with `--direct=1`.
- Power: NVML via `nvidia-smi`, or firmware rails via [spark_hwmon](https://github.com/antheas/spark_hwmon) on GB10; say which.

## Published reference measurements

Third-party numbers, quoted with their sources. Early-software results age quickly; check the source date.

### N1X laptops

| Source | Platform | Measurement |
|---|---|---|
| [StorageReview, Surface Laptop Ultra](https://www.storagereview.com/review/microsoft-surface-laptop-ultra-for-business-review-rtx-spark-n1x) (2026-10-15) | WoA | STREAM Triad 122.1 GB/s; memory latency 251 ns; SSD sequential 14,134 MB/s read and 11,461 MB/s write; Llama 3.1 8B 56.4 tok/s in MLPerf Client; all-core OpenBLAS powered the laptop off three times. |

No native N1X Linux performance measurements were found.

### GB10 desktops

| Source | Measurement |
|---|---|
| [llama.cpp DGX Spark results](https://github.com/ggml-org/llama.cpp/discussions/16578) (maintainer-run, driver 580.95.05, CUDA 13.0) | gpt-oss-120b: pp2048 2443.9 t/s and tg32 58.7 t/s, falling to 42.8 t/s tg at 32k depth; mmap disabled. |
| [Chips and Cheese, memory subsystem](https://chipsandcheese.com/p/inside-nvidia-gb10s-memory-subsystem) (2025-12-31) | DRAM latency 113 ns; about 38 GB/s from one X925 core; CPU latency above 351 ns while the GPU pulls 231 GB/s. |
| [NVIDIA forum bandwidth thread](https://forums.developer.nvidia.com/t/the-ddr-bandwidth-is-significantly-lower-than-the-claimed-273gb-s/363238) (2026-03) | Community GPU copy and triad tests at about 229–230 GB/s, roughly 84% of the 273 GB/s specification. |
| [StorageReview, DGX Spark](https://www.storagereview.com/review/nvidia-dgx-spark-review-the-ai-appliance-bringing-datacenter-capabilities-to-desktops) | GDSIO sequential read 11.4 GiB/s and write about 8.3 GiB/s. |
| [LMSYS, DGX Spark](https://www.lmsys.org/blog/2025-10-13-nvidia-dgx-spark/) (2025-10-13, launch software) | SGLang Llama 3.1 8B FP8 batch 1: 7,991 prefill and 20.5 decode tok/s; prompt and output lengths not stated. |
| [Qwen3.8-27B study](https://forums.developer.nvidia.com/t/comprehensive-qwen3-8-27b-study-on-dgx-sparks-quantization-speculative-decoding-and-tp-dp-scaling/381102) (2026-08-24) | Single-user decode: BF16 4.5, FP8 7.9, NVFP4 9.7 tok/s; with MTP 9.9, 17.1 and 18.5 tok/s. |
| [gb10-clock-cap](https://github.com/agjs/gb10-clock-cap) | A 2200 MHz GPU cap measured at −12 °C, −36% GPU power and about −1% decode. |
| [moe-offload-findings](https://github.com/GaelicThunder/moe-offload-findings) (ASUS GX10) | Full speculative decoding slowed NVMe-streamed MoE decode from 0.86 to 0.70 tok/s; drafting only over resident experts gained 18–21%. |
| [dgx-spark-memory-saver](https://github.com/christopherowen/dgx-spark-memory-saver) | 1.78–1.86 GiB net memory recovered per node on 64 KiB kernels versus stock 4 KiB. |

### Offload terms used in these results

- **Weight / expert offload** keeps model weights or MoE experts outside GPU-resident memory and streams them, often from NVMe (SparkLab `--moe-storage disk`, llama.cpp `--n-cpu-moe`, vLLM `--cpu-offload-gb`).
- **Embedding-table offload** keeps very large embedding tables on disk and reads rows on demand (b12x disk tables); this is different from llama.cpp's whole-model `mmap`.
- **KV-cache offload** moves attention cache blocks out of GPU memory (vLLM KV offloading, llama.cpp `--no-kv-offload`); on unified memory it mostly helps prefix reuse rather than capacity.

## Proposed experiments

Not yet run by anyone we know of. Reports are welcome.

1. Native N1X Linux versus WoA versus WSL2 on the same laptop, same model and llama.cpp commit, with `llama-bench -r 5` at several depths.
2. Active-use battery drain on native Linux (idle desktop, video playback, sustained decode) with `upower` and wall-meter readings.
3. NVMe throughput on N1X Linux with and without the ASPM L1.2 patch, compared to the Windows figures above.
4. Visible memory and model capacity with the 4 GiB GPU reservation setting on the XPS 16.
5. 64 KiB-page kernel on N1X: does `open-gpu-kernel-modules#1269` reproduce, and how much memory does it save?
6. NVFP4 versus FP8 versus Q4 GGUF decode on N1X, with output-equality checks.
7. Speculative decoding acceptance rate and speed-up on N1X under battery power profiles.
8. Thermal throttling during sustained decode on battery versus AC, with clocks logged.
