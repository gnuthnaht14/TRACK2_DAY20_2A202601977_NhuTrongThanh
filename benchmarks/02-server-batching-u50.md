# 02 - Continuous batching under load (u50)

Host `Windows-AMD64` · `--parallel 4` · 15 samples over
60s at 2.0s intervals · raw CSV: `02-server-metrics-u50.csv`

| Gauge | Peak observed |
|:--|--:|
| `n_busy_slots_per_decode` (avg/decode) | 3.65 of 4 slots (91%) |
| `requests_processing` | 4 |
| `requests_deferred` | 46 |
| `kv_cache_usage_ratio` | n/a — not exported by llama.cpp `b10488` |
| `tokens_predicted_total` (final) | 3413 |

Highest sampled value was **3.65 of 4** slots. Note this gauge is llama.cpp's *average* busy slots per decode step, so the number below is the highest average we sampled, not an instantaneous maximum batch width. A peak near 1 means
requests were served one at a time -- either the load was too light to overlap, or
they arrived too far apart. A peak approaching `--parallel` means the scheduler was
genuinely packing concurrent requests into shared decode steps.
`requests_deferred` went above zero: more requests arrived than there were slots, so some waited. That wait is the queue time in your P95.

## Nhận xét của tôi

Peak busy width là 3.65/4 slot (91%), đồng thời có tối đa 46 request deferred:
continuous batching thực sự hoạt động và bốn slot gần kín. Effective concurrency
14.4 cao hơn bốn slot vì Little's Law tính cả request đang đợi. Hai số không mâu
thuẫn; tôi dùng gauge 3.65 để đọc mức sử dụng slot và dùng 14.4 để đọc tổng occupancy
gồm cả queue.
