# 03 - Integrate: RAG pipeline run

Host `Windows-AMD64` · llama.cpp `b10488` ·
retrieval backend: **keyword overlap** · 3 queries

| Query | Contexts retrieved | embed (ms) | retrieve (ms) | llm (ms) | total (ms) |
|:--|--:|--:|--:|--:|--:|
| Why is goodput more useful than raw throughp... | goodput, paged, radix | 0.0 | 0.1 | 10506.5 | 10506.7 |
| What problem does PagedAttention actually so... | paged, radix, disagg | 0.0 | 0.1 | 7834.4 | 7834.6 |
| When does splitting prefill and decode help?... | disagg, radix, batching | 0.0 | 0.1 | 7786.9 | 7787.1 |

Mean per stage (ms): embed **0.0** · retrieve **0.1** ·
llm **8709.3** · total **8709.5**
Dominant stage: **llm** (100% of total)

## Answers returned

**Why is goodput more useful than raw throughput?**

> Based on the context provided, **Goodput** is more useful than raw throughput because it focuses exclusively on requests that met the Target Time-to-Fill (TTFT) and Target Time-to-Poll (TPOT) targets.

Raw throughput ignores SLOs (Service Level Objectives), meaning it does not account for the actual performance of the system relative to its capacity limits. Goodput, conversely, counts only the req

**What problem does PagedAttention actually solve?**

> PagedAttention solves the problem of **internal fragmentation in GPU memory** caused by storing the Key-Value (KV) cache in non-contiguous pages.

By utilizing non-contiguous pages, it removes the wasted space that would otherwise exist within the contiguous memory blocks of the GPU's VRAM. This allows the engine to utilize more of the available GPU memory for inference tasks, which is particularl

**When does splitting prefill and decode help?**

> Based on the context provided, splitting prefill and decode helps when **prefill is compute-bound and decode is memory-bound**.

The context explicitly states that this split is necessary because:
1.  Prefill (the initial generation phase) is bound by **compute** resources.
2.  Decode (the generation phase) is bound by **memory** bandwidth.

By separating these phases, the system can utilize compu


## Thành phần real và stub

- N16 Cloud/IaC: stub, chỉ chạy localhost.
- N17 Data pipeline: stub, dữ liệu nằm trong bộ nhớ.
- N18 Lakehouse: stub, dùng `TOY_DOCS` thay cho Delta/Iceberg.
- N19 Vector + features: stub, keyword overlap thay cho vector index và không có
  embedding endpoint (`embed = 0.0 ms`).
- N20 Serving: real, gọi HTTP tới `llama-server` và nhận completion thật.

LLM chiếm 8709.3/8709.5 ms, xấp xỉ 100%, đúng kỳ vọng với corpus đồ chơi. Muốn
giảm tổng latency 2x, tôi sẽ tối ưu stage LLM trước (giảm output budget, giữ prefix
ổn định để tận dụng prompt cache hoặc dùng runtime/accelerator nhanh hơn); tối ưu
retrieval 0.1 ms hầu như không thay đổi tổng thời gian.
