# KV260 250 MHz timing closeout

The board-qualified baseline is the two-column **225 MHz** design with 2,048 instructions and 2,048 WAIT entries. Its build ID is `d638198b`, its architecture ID is `e9ed9d11`, and its board counter measured **225,002,599 Hz**. On November 23, 2026, we stopped the 250 MHz implementation runs and their queue. The best 250 MHz design connected every net and missed setup by **19 ps across 850 endpoints**. Vivado withheld bitstream export. The benchmark rates below come from the qualified 225 MHz image. [Measurements and artifact digests](results.json)

## Physical results

| Design | Evidence | Worst setup (ns) | Total negative setup (ns) | Failing setup endpoints | Worst hold (ns) | Worst pulse (ns) |
|---|---|---:|---:|---:|---:|---:|
| 225 MHz, 2K, `d638198b` | Routed and run on the KV260 | +0.003 | 0.000 | 0 | 0.000 | +0.722 |
| 250 MHz, 2K, `3a691612` | Fully routed | −0.019 | −6.873 | 850 | +0.007 | +0.500 |
| 250 MHz, 2K, `a6b77d40` | Fully routed, DSP operand registers and paired column-end adder | −0.094 | −37.357 | 1,101 | +0.004 | +0.500 |
| 250 MHz, 2K, `21969753` | Placed, collective return registers | −0.047 | −4.154 | 220 | −0.133 | +0.500 |
| 300 MHz, 1K, `b5aae74d` | Fully routed clock-only diagnostic | −1.035 | −24,002.367 | 60,436 | +0.006 | +0.167 |

The best 250 MHz route connected all 181,700 nets. Its worst path ran from the collective read-enable register to a tensor-memory BRAM address pin through six LUT levels. Routing accounted for 2.450 ns of 3.363 ns of data delay. Eight weight registers launched 463 failing endpoints, one row-scale register launched 96, and four vector operand registers launched 192. The 48 endpoints in the column-end DSP cascade accounted for 0.336 ns of total negative slack. Local rerouting and post-route clock optimization retained the −19 ps worst slack. Two runs with 50 ps of additional setup margin terminated inside Vivado's leaf-clock optimizer before nominal routed signoff. [Routed checkpoint and report fingerprints](results.json)

| KV260 resource | Qualified 225 MHz | Best routed 250 MHz | Available |
|---|---:|---:|---:|
| Occupied CLB sites | 14,590 | 14,587 | 14,640 |
| LUTs | 93,385 | 94,552 | 117,120 |
| Flip-flops | 80,119 | 80,758 | 234,240 |
| BRAM36 equivalents | 117.5 | 117.5 | 144 |
| UltraRAM | 32 | 32 | 64 |
| DSP48E2 | 413 | 414 | 1,248 |

At 225 MHz, doubling the instruction and WAIT stores cut time to first token on SmolLM2's 2,048-token prompt from 59.22 s to 39.36 s. Decode stayed near 23.68 tokens/s. Bigger models frequently spent most of their time moving or expanding coefficients on the A53. On Gemma E4B, the local coefficient cache saved 198.39 s per 64 tokens. The FPGA ran for 45.18 s of the remaining 2,724.36 s. Even if every FPGA cycle scaled with the clock, moving from 225 to 250 MHz would save about 4.52 s on that request.

## Batch-one decode at closeout

The unit is wall-clock tokens per second after the first token. SmolLM2 uses 32 outputs and times the last 31. The remaining rows use a 512-token prompt and 64 fresh decode steps. The Kintex figures are the upstream authors’ measurements. The KV260 OpenTPU figures use two 128-bit HPC ports and a 256 MiB accelerator window. The three FOUR12 processors run at 500 MHz each with six independent 128-bit DMA ports at 333.33 MHz. Their complete design uses 33 DSP48E2 slices. Each dash marks a configuration without a matched published or physical run at closeout. [Original Kintex LFM and Qwen](../board.md) · [SmolLM3 and Phi](../llama.md) · [Gemma E2B](../gemma4.md) · [Gemma E4B](../gemma4_e4b.md)

| Workload | OpenTPU Kintex | OpenTPU KV260, 225 MHz | P-RISC P16 KV260 | Three FOUR12 processors, KV260 |
|---|---:|---:|---:|---:|
| SmolLM2-135M-Instruct (64-token prompt) | — | 37.20 | — | 14.58 |
| SmolLM2-135M-Instruct (512-token prompt) | — | 32.67 | — | 12.03 |
| SmolLM2-135M-Instruct (2,048-token prompt) | — | 23.68 | — | 7.378 |
| LFM2.5-230M, W8 | 52.25 | 24.56 | — | 13.42 |
| LFM2.5-230M, FP4 layers | 82.12 | 34.75 | — | — |
| Qwen3-0.6B, OpenTPU W8 / DSP W12 | 21.33 | 3.801 | — | 1.984 |
| Qwen3-0.6B, FP4 layers | 30.67 | 5.494 | — | — |
| Qwen3.5-0.8B, OpenTPU W8 / DSP W12 | 16.31 | 3.185 | — | 1.607 |
| Qwen3.5-0.8B, FP4 layers | 23.27 | 4.509 | — | — |
| SmolLM3-3B, W8 | 4.990 | 0.7750 | — | — |
| SmolLM3-3B, OpenTPU FP4 layers / DSP S6 layers | 8.720 | 1.340 | — | 0.3648 |
| Phi-4-mini, W8 | 3.980 | 0.02172 | — | — |
| Phi-4-mini, OpenTPU FP4 layers / DSP S6 layers | 6.550 | 1.005 | — | 0.1435 |
| Gemma 4 E2B, OpenTPU FP4 layers / DSP S6 layers, W8 head | 10.53 | 0.9228 | — | 0.4775 |
| Gemma 4 E2B, FP4 layers and head | 12.09 | 1.048 | — | — |
| Gemma 4 E4B, OpenTPU mixed/FP4 head, DSP S6/S8 head | 3.750 | 0.02349 | — | 0.06701 |

Three-DSP SmolLM2 used W12A12, and OpenTPU used W8A8. The larger three-DSP S6 runs used A12 activations and an S8 tied head. Gemma E4B OpenTPU used W8 with 24 FP4 down projections and an FP4 head. Each KV260 design was checked against its own numerical reference. On the ten rows measured on both designs, OpenTPU led on nine. The three DSPs led on E4B by 2.85×. On Qwen3 the three-DSP decode median rose from 0.926 to 1.984 tokens/s after native DMA packet retention and host scheduling. The archived programmable P16 R15 result was 0.39852 **frames/s** on trained YOLO26n at 290 MHz. Its TinyGPT2 and BERT matrix kernels did not provide an end-to-end LLM token rate.

## Reconstructing the physical inputs

Start at upstream revision `d84db55712aba0d40811d7762cb052f05d55b7e5` in a disposable checkout. Apply the 225 MHz patch first. Applying the second patch on top recovers the physical source inputs of the best 250 MHz run. Both source sets use the same 2K architecture and Vivado 2026.1. The result record identifies the saved route and the qualified 225 MHz bitstream by SHA-256. A fresh route requires its own timing signoff. [225 MHz source patch](225mhz-sources.patch) · [250 MHz source patch](250mhz-sources.patch) · [Source identities](source-pins.json)
