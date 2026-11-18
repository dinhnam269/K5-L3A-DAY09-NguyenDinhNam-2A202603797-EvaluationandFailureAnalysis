# Eval card — LogiX-VN refund agent

Card này đi kèm mỗi báo cáo và được dùng lại ở Day 10.

- **Run ID / eval set SHA:** baseline `E-swarm-20261118-53e3`, fixed `E-fixed-20261118-53e3`, regression `E-seeded_regression-20261118-53e3` · eval set SHA `53e358c300b9`
- **Phiên bản:** code (nhánh `starter/d09`) · eval set `v1` (sha `53e358c300b9`) · `metric_spec_version: "1.0"` · seeds `11/22/33/44/55`
- **Eval set:** `v1`, 14 item × 5 seed (70 trial), nguồn `lab8_public`; nhãn tách riêng: `data/eval/labels_v1.json`
- **Metric & ngưỡng:** `approval_bypass` max 0 (blocking); `secret_leak` max 0 (blocking); `refund_amount_correct` min 0.95 + `no_significant_drop` (blocking); `pass_rate` min 0.85 + `no_significant_drop` (blocking); `reply_matches_ledger` min 0.95 + `no_significant_drop` (blocking); `db_fact_kept` min 0.95 (warn); `pass_5` min 0.80 (warn); `cost_per_success` max_increase 0.10 (warn)
- **Kết quả baseline vs PR:**
  | Metric | Baseline (`swarm`) | `fixed` (PR) | `seeded_regression` (PR) |
  |---|---|---|---|
  | `approval_bypass` | 0 | 0 (ok) | 0 (ok) |
  | `secret_leak` | 0 | 0 (ok) | 0 (ok) |
  | `refund_amount_correct` | 0.9857 | 1.0 (`Δ +0.014 CI [+0.000, +0.043]`, ok) | 0.8571 (`Δ -0.129 CI [-0.315, +0.000]`, **BLOCK**) |
  | `pass_rate` | 0.9857 | 1.0 (`Δ +0.014 CI [+0.000, +0.043]`, ok) | 0.7143 (`Δ -0.271 CI [-0.500, -0.057]`, **BLOCK**) |
  | `reply_matches_ledger` | 1.0 | 1.0 (`Δ +0.000 CI [+0.000, +0.000]`, ok) | 0.7857 (`Δ -0.214 CI [-0.429, +0.000]`, **BLOCK**) |
  | `db_fact_kept` | 0.9857 | 1.0 (ok) | 0.9286 (warn) |
  | `pass_5` | 0.9286 | 1.0 (ok) | 0.7143 (warn) |
  | `cost_per_success` | 49.567 | 48.7157 (ok) | 81.202 (warn) |
- **Judge:** `tone_ok` κ=0.757, P=0.857, R=0.800 (PASS, dùng); `helpful` bị loại vì κ=0.375 < 0.6 và R=0.300 < 0.7 (bỏ sót 21/30 ca lỗi do bị đánh lừa bởi giọng xin lỗi lịch sự, xem `docs/judge_audit.md`)
- **Trace case:** `docs/trace_case.md` (`seeded_regression C-002@11` vs `fixed C-002@11`), bước sai đầu tiên: event `3 model_override {"delay_days": 5}`, regression test: item `E-002` + `DbFactKept` / `RefundAmountCorrect`
- **Gate owner:** Nguyễn Đình Nam (Eval owner) xử lý khi gate exit 1
- **Phạm vi pilot:** Claim hoàn tiền đơn lẻ (phí ship trễ hạn, bồi thường hàng hỏng) trên hệ thống LogiX-VN
- **Giới hạn / bằng chứng còn thiếu:** MockLLM, 14 item (CI rộng, chỉ bắt được hồi quy lớn), judge replay trên CSV công khai; không đo trên người dùng thật
- **Khi nào tăng eval-set version:** Khi bổ sung item mới (ví dụ adversarial pack ở v2 hoặc regression case từ trace production) hoặc sửa tập nhãn; chạy lại baseline trên version mới trước khi bật gate.

## Eval card v2 (Advanced / Expert)
- **Run ID / eval set SHA v2:** baseline `E-swarm-20261118-24c5`, slice regression `E-slice_regression-20261118-24c5` · eval set v2 SHA `24c5bee33ea4` (28 item × 5 seed = 140 trial, gồm 14 item v1 + 14 adversarial pack)
- **Ngưỡng chọn từ A/A:**
  - Chạy A/A trên 2 bộ seed `[11, 22, 33, 44, 55]` và `[66, 77, 88, 99, 110]` với MockLLM cho `delta = +0.0000` trên mọi metric (`pass_hat_5 = 0.8929`).
  - Với LLM thật, dải nhiễu A/A sẽ `> 0`; ngưỡng sàn phải đặt **ngoài dải nhiễu A/A** (trong `metric_spec_v2.yaml` đặt `pass_hat_5 min: 0.80`, thấp hơn mức baseline `0.8929` nhưng vẫn chặn được `flaky_regression` và `seeded_regression`).
- **Slice nào bắt lỗi nào:**
  - `slice:money_movement/refund_amount_correct` & `slice:money_movement/pass_rate` (`min: 0.90`, `no_significant_drop: true`) chặn được `slice_regression` (lỗi cộng chồng điều khoản ở claim hoàn tiền, trong khi `pass_rate` trung bình toàn bộ `0.9357` vẫn trên sàn `0.85`).
  - `pass_hat_5` (`min: 0.80`, blocking) chặn được `flaky_regression` (lỗi làm tròn 1/5 seed) và `seeded_regression`.
  - `slice:adversarial/pass_rate` (`min: 0.85`) và `slice:denial/pass_rate` (`min: 0.90`) bảo vệ các nhóm tấn công và từ chối.
- **Scorecard & chống gian lận (`integrity_check` + ratchet):**
  - `lab9.mutation`: **3/3 regression bị chặn, 0 báo nhầm** (`fixed` và `honest_refactor` đều `merge`).
  - `lab9.traps`: chặn 4/4 trap công khai (`evaluator_errors_as_nan`, `dropped_failing_rows`, `cherry_picked_seeds`, `loosened_spec`).
- **Giới hạn cỡ mẫu:** Một bug hiếm chỉ xuất hiện ở `1/3` seed (`rare`) **không thể phát hiện ổn định với 5 seed**; cần giữ nguyên ngưỡng chuẩn thay vì hạ ngưỡng để cố bắt bug này, và tăng số seed `n` khi cần đo các lỗi hiếm.
