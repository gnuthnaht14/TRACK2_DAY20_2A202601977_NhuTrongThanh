# 01 - Measure: latency baseline

Model `Qwen3.5 0.8B` · host `Windows-AMD64` · llama.cpp `b10488`
Settings: `threads=6` `ngl=0` `ctx=2048`
`max_tokens=64` · warm-up discarded
Completed requests: `Q4_K_M` 10/10 · `UD-Q2_K_XL` 10/10

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|:--|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 3452 | 311 / 359 | 31.7 / 33.0 | 2303 / 2408 / 2408 | 31.6 |
| UD-Q2_K_XL | 0.39 | 3707 | 449 / 547 | 37.1 / 39.3 | 2802 / 2968 / 2968 | 27.0 |

- **TTFT** = prefill. Short prompts keep it small; long-context RAG is where it explodes.
- **TPOT** = per-output-token decode cost, bounded by memory bandwidth. `decode tok/s = 1000 / TPOT_p50`.
- `UD-Q2_K_XL` decodes **1.17x SLOWER** than `Q4_K_M` here, despite being 0.11 GB smaller. That is a real result, not a mistake: fewer bits only buys speed when decode is limited by memory bandwidth. On a machine that is compute-limited instead — few cores, no GPU offload — the extra dequantization work of a heavily-quantized format can cost more than the bytes it saves. Say which case yours is.

## Nhận xét của tôi

UD-Q2_K_XL nhỏ hơn 0.11 GB (22%) nhưng decode chậm hơn 1.17x: 27.0 so với
31.6 tok/s; TTFT P50 cũng tăng từ 311 lên 449 ms. Với cùng bài toán phần trăm,
Q4 còn diễn giải được dù tính sai, còn Q2 lặp phép nhân và suy biến. Vì vậy mức
tiết kiệm 22% dung lượng không đáng đổi lấy cả latency lẫn chất lượng trên máy này.
