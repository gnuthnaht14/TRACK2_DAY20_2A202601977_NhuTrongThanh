# Reflection — Day 20 Lab (Personal Report)

> **Đây là báo cáo cá nhân.** Số liệu của bạn **không** so sánh được với bạn cùng lớp
> — chỉ so **before vs after trên chính máy bạn**. Rubric chấm độ rõ ràng của setup,
> đo lường và **lập luận**, không chấm tốc độ tuyệt đối.
>
> `make verify` sẽ fail nếu còn placeholder chưa điền. Đó là cố ý.

**Họ Tên:** Nhu Trong Thanh
**Cohort:** 3A
**Ngày submit:** 2026-08-20

---

## 1. Hardware & runtime  *(rubric 1, 2 — 10 điểm)*

> Từ `make probe`. Paste output hoặc điền tay.

- **OS:** Windows 11 (AMD64)
- **CPU:** AMD Ryzen 5 5500U with Radeon Graphics
- **Cores:** 6 physical / 12 logical
- **CPU extensions:** AVX2 / FMA
- **RAM:** 15.3 GB
- **Accelerator:** CPU only (`ngl=0`)
- **llama.cpp asset đã tải:** `llama-b10488-bin-win-cpu-x64.zip`
- **Model đã dùng:** Qwen3.5 0.8B (`LAB_MODEL=qwen35-0.8b`)
- **Quantization:** Q4_K_M + UD-Q2_K_XL (từ `models/active.json`)

**Chạy ở đâu:** laptop của tôi, không dùng cloud fallback.

**Setup story** (≤ 80 chữ): điều gì cần thay đổi để lab chạy trên máy bạn? Có bước
nào fail rồi phải workaround không?

Máy chưa có Python hệ thống; lệnh `python` chỉ là Microsoft Store alias. Tôi dùng
`uv` tạo `.venv` với CPython 3.12.13, chạy `ensurepip`, rồi cài `requirements.txt`.
Probe ban đầu còn gặp lỗi console cp1252 nên tôi bật `PYTHONUTF8=1`. Sau đó setup
nhận đúng hai GGUF đã tải và tự lấy llama.cpp CPU build b10488.

---

## 2. Đo lường  *(rubric 3, 4, 5 — 20 điểm)*

> Paste bảng từ `benchmarks/01-quickstart-results.md` (`make bench` tự sinh).

| Quantization | Size (GB) | Load (ms) | TTFT P50/P95 (ms) | TPOT P50/P95 (ms) | E2E P50/P95/P99 (ms) | Decode (tok/s) |
|---|--:|--:|--:|--:|--:|--:|
| Q4_K_M | 0.50 | 3452 | 311 / 359 | 31.7 / 33.0 | 2303 / 2408 / 2408 | 31.6 |
| UD-Q2_K_XL | 0.39 | 3707 | 449 / 547 | 37.1 / 39.3 | 2802 / 2968 / 2968 | 27.0 |

**Quan sát** (≤ 60 chữ): 2-bit nhanh hơn bao nhiêu, và **có đáng không**? Bạn đã thử
hỏi cùng một câu trên cả hai (`make serve` vs `.venv/bin/python labs/02-serve/serve.py --compare`)
chưa? Chất lượng khác nhau thế nào?

Q2 nhỏ hơn 22% nhưng decode chậm hơn 1.17x và TTFT P50 tăng 44%. Với cùng bài toán
phần trăm, Q4 còn diễn giải được dù tính sai, còn Q2 lặp phép nhân và suy biến. Vì
vậy Q2 không đáng dùng trên CPU này: tiết kiệm dung lượng nhưng thua cả tốc độ lẫn
chất lượng.

---

## 3. Serving under load  *(rubric 8, 9, 10 — 20 điểm)*

> Từ `benchmarks/02-server-results.md` (`make load-report`).

| Users | RPS | P50 (ms) | P95 (ms) | P99 (ms) | Eff. concurrency | Failures |
|--:|--:|--:|--:|--:|--:|--:|
| 10 | 0.48 | 17000 | 30000 | 31000 | 8.8 | 0.0% |
| 50 | 0.48 | 27000 | 57000 | 57000 | 14.4 | 0.0% |

- **Offered load tăng 5×, throughput thực tăng:** 1.00×
- **P95 tăng:** 1.90×
- **Effective concurrency ở 50 users:** 14.4 so với `--parallel` = 4 slots

**Peak `llamacpp:n_busy_slots_per_decode`** (từ `make metrics` khi `make load-50` đang
chạy): 3.65 / 4 slots

**Saturation reading** (≤ 80 chữ): server của bạn bão hoà ở đâu, và **bằng chứng nào**
thuyết phục bạn? Nếu P95 tăng nhanh hơn RPS thì phần latency thêm đó là queue time hay
compute time — bạn biết bằng cách nào? Nếu bạn phải nâng goodput@SLO, bạn sẽ đổi knob
nào **trước**, và vì sao knob đó?

Server đã bão hòa ở hoặc trước 10 users vì effective concurrency 8.8 đã vượt 4
slot. Ở 50 users, RPS vẫn 0.48 nhưng P95 tăng 30→57 giây, concurrency lên 14.4
và 46 request bị deferred: phần tăng là queue time. Với SLO P95 ≤30 giây, tôi sẽ
thử `--parallel 8` trước, nhưng chỉ giữ nếu goodput tăng mà TPOT không xấu đi vì
CPU bị chia sẻ quá mức.

---

## 4. Integration  *(rubric 12, 13 — 15 điểm)*

> Từ `make pipeline`. Nói thật cái nào real, cái nào stub — stub **không** mất điểm.

| Day | Piece | Real hay stub? |
|---|---|---|
| N16 Cloud/IaC | localhost only | stub |
| N17 Data pipeline | in-memory toy data | stub |
| N18 Lakehouse | `TOY_DOCS` dictionary | stub |
| N19 Vector + features | keyword overlap, không có embed endpoint | stub |
| N20 Serving | `llama-server` | real |

**Latency split** (mean của 3 query, từ output của `pipeline.py`):

- embed: 0.0 ms
- retrieve: 0.1 ms
- llm: 8709.3 ms
- **stage chiếm nhiều nhất:** llm (xấp xỉ 100% của total)

**Reflection** (≤ 60 chữ): bottleneck ở đâu? Có khớp với kỳ vọng của bạn không? Nếu
phải giảm latency của pipeline này 2×, bạn sẽ tấn công vào đâu?

LLM là bottleneck đúng như kỳ vọng vì corpus và keyword retrieval chỉ chạy trong
bộ nhớ. Muốn giảm tổng latency 2×, tôi sẽ tối ưu LLM: giảm output budget, giữ
prefix ổn định để dùng prompt cache, hoặc chuyển sang accelerator. Tối ưu retrieval
0.1 ms không tạo khác biệt đo được.

---

## 5. The single change that mattered most  *(rubric 11 — 10 điểm)*

> **Phần quan trọng nhất của report.** Không cần bonus track: `make tune` đã cho bạn
> một before/after thật (`benchmarks/01-tuning-tg128.md`). Đổi quantization,
> `LAB_N_CTX`, hay `--parallel` rồi đo lại cũng được.

**Change:** hạ `-t` từ 24 thread xuống 6 thread vật lý

```
before:  15.8 tok/s (-t 24)
after:   33.1 tok/s (-t 6)
speedup: 2.09×
```

**Tại sao nó work** (1–2 đoạn — đây là phần grader đọc kỹ nhất):

Kết quả peak ở 6 thread, đúng bằng số core vật lý. Với một thread, model chưa tận
dụng hết khả năng song song; tăng tới 6 giúp các core cùng xử lý và throughput tăng
đều. Tuy nhiên decode phải đọc trọng số lặp lại và nhanh chóng bị giới hạn bởi băng
thông bộ nhớ, không chỉ bởi số phép tính.

Ở 24 thread, bốn thread phần mềm tranh nhau trên mỗi core vật lý. Chúng không tạo
thêm memory channel hay cache, nhưng làm tăng context switching, cache eviction và
chi phí scheduler. Vì vậy giảm 24 xuống 6 loại bỏ oversubscription và tăng throughput
từ 15.8 lên 33.1 tok/s (2.09×). Mức 12 thread chỉ đạt 31.8 tok/s cũng củng cố cơ chế
này: SMT gần peak nhưng không thắng được một thread trên mỗi core vật lý.

---

## 6. Bonus  *(optional — tối đa 20 điểm)*

> Bỏ trống nếu không làm. Xem `bonus/README.md`. Đừng làm hết — **một** finding sâu
> ăn điểm hơn năm bảng nông.

**Đã làm:** _<B1 build-compare / B2 sweep nào / B4 challenge nào / B5 lựa chọn nào>_

**Numbers:**

```
before:  <số>
after:   <số>
speedup: <X.Y>×
```

**Điều này nói lên gì mà deck chưa nói:**

_(để trống nếu bạn không làm phần này)_

---

## 7. Điều làm bạn ngạc nhiên nhất  *(optional)*

_(1–2 câu. Không bắt buộc, nhưng grader đọc hết.)_

_(để trống nếu bạn không làm phần này)_

---

## 8. Self-check trước khi push

- [ ] `hardware.json` committed
- [ ] `models/active.json` committed
- [ ] `benchmarks/01-quickstart-results.md` committed (`make bench`)
- [ ] `benchmarks/01-tuning-tg128.md` committed (`make tune`)
- [ ] `benchmarks/02-server-results.md` committed (`make load-report`)
- [ ] `benchmarks/02-server-batching-u50.md` hoặc `-metrics-u50.csv` committed (`make metrics`)
- [ ] `benchmarks/locust-10_stats.csv` + `locust-50_stats.csv` committed (`make load-10` / `load-50`)
- [ ] `benchmarks/03-integration-results.md` committed (`make pipeline`)
- [ ] Mọi section **"required — replace this line"** trong các file `benchmarks/*.md`
      đã được thay bằng nhận xét của bạn
- [ ] 5 screenshots trong `submission/screenshots/`
- [ ] `make verify` → **exit 0**
- [ ] Repo GitHub ở chế độ **public**
- [ ] Đã paste public URL vào VinUni LMS
- [ ] **Không** commit `models/*.gguf` hay `runtime/` (đã có trong `.gitignore`)

**Quan trọng:** repo phải **public** đến khi điểm được công bố. Private → grader không
xem được → 0 điểm.
