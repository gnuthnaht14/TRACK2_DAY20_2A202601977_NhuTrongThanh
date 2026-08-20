# 02 - Serve: load test + saturation reading

Host `Windows-AMD64` · llama.cpp `b10488` ·
`--parallel 4` · `ctx=2048` · `threads=6` ·
`ngl=0`

| Users | Reqs | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|:--|--:|--:|--:|--:|--:|--:|--:|
| 10 | 28 | 0.48 | 17000 | 30000 | 31000 | 8.8 | 0.0% |
| 50 | 28 | 0.48 | 27000 | 57000 | 57000 | 14.4 | 0.0% |

*Effective concurrency = RPS x average latency (Little's Law) -- how many requests were
really in flight, regardless of how many users locust simulated. It counts queued requests
too, so the occupancy/slot ratio can legitimately exceed 1.0; it is occupancy, not
utilisation. For true slot utilisation use the server's own gauges (`make metrics`).*

## What these two runs say

| Going from 10 to 50 users | |
|:--|--:|
| Offered load | 5x |
| Throughput actually delivered | **1.00x** (20% of linear) |
| P95 latency | **1.90x** |
| Effective concurrency at 50 users | 14.4 vs `--parallel 4` slots (occupancy/slot ratio 3.61) |

**Saturated.** Throughput delivered only 1.00x for 5x the offered load, and effective concurrency (14.4) is at or above all 4 decode slots. Saturation sets in somewhere at or below 50 users; the load you added beyond that point became queue time rather than throughput.

Throughput moved 1.00x while P95 moved 1.90x. That gap is the goodput argument: past saturation you buy throughput by spending latency, and if your SLO is a P95 target then the requests you added are no longer being served within it. (This lab does not fix an SLO number for you -- pick one in your write-up and state how much goodput you keep at it.)

## Phân tích của tôi

Server đã bão hòa từ mức không quá 10 users: effective concurrency 8.8 đã vượt
4 slot. Khi tăng lên 50 users, throughput vẫn 0.48 RPS nhưng P95 tăng 30 lên 57
giây và concurrency đạt 14.4; latency thêm là queue time, được xác nhận bởi 46
request deferred. Với SLO P95 ≤ 30 giây, tôi sẽ thử tăng `--parallel` từ 4 lên 8
trước vì áp lực nằm trực tiếp ở slot/queue, rồi giữ thay đổi chỉ nếu goodput tăng
mà TPOT không suy giảm do CPU bị chia sẻ quá mức.
