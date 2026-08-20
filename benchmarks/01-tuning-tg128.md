# 01 - Tune: thread-count sweep

Model `Qwen3.5-0.8B-Q4_K_M.gguf` · host `Windows-AMD64` · llama.cpp `b10488`
CPU: **6 physical · 12 logical** cores · `ngl=0` · metric `tg128`

| threads (-t) | tg128 (tok/s) | vs best |
|:--|--:|--:|
| 1 | 11.6 | 35% |
| 3 | 23.2 | 70% |
| 6 | 33.1 | 100% |
| 12 | 31.8 | 96% |
| 24 | 15.8 | 48% |

**Best**: `-t 6` at 33.1 tok/s
**Slowest tested**: `-t 1` at 11.6 tok/s (2.85x spread)
**Against the physical-core default** (`-t 6`, 33.1 tok/s): 1.00x

Use this in your run:

```bash
LAB_N_THREADS=6 make bench
```

## Giải thích của tôi

Knee nằm ở 6 thread, đúng bằng 6 core vật lý: throughput tăng từ 11.6 lên
33.1 tok/s khi đi từ 1 lên 6 thread. Dùng 12 thread chỉ còn 31.8 tok/s và 24
thread rơi xuống 15.8 tok/s. Sau khi các core vật lý đã được dùng hết, SMT và
oversubscription không tạo thêm memory bandwidth; các thread cạnh tranh cache,
băng thông RAM và thời gian scheduler nên chi phí điều phối vượt lợi ích song song.
