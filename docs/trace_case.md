# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi bạn 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11` (chạy qua `lab9.trace_case`; trên `swarm` lỗi tương tự xuất hiện ở `C-002@55`) |
| Triệu chứng | Lab 8 grader FAIL (`status resolved != denied`); `reply` nói `"Dạ, LogiX đã hoàn 20.000đ cho đơn O-202."`, còn ledger ghi `refund_vnd=20000`, `refund_count=1` trong khi nhãn kỳ vọng là `refund_vnd=0`, `refund_count=0`, `status=denied`. |
| Bước sai đầu tiên | Event `3 model_override {"delay_days": 5}` |
| Vì sao là bước *đầu tiên* | Event `1 handoff` truyền đúng `task: "check_applicable_policy"` với `facts: {}`; event `2 investigator` gọi `lookup_order` trả về đúng dữ kiện đơn `O-202` từ DB (`visible: true`, `status: "delivered"`). Tới bước `3`, `model_override` mới ghi đè `delay_days: 5` lấy từ ticket khách hàng. |
| Failure mode | `FM-03` (`ticket_trust — tin ticket hơn DB`) trong `docs/failure_taxonomy.md` |
| Đối chứng | Trên `fixed C-002@11`: không có event `model_override`; ngay sau bước `2 investigator` là bước `3 quote {"amount_vnd": 0, "reason": "no_entitlement", "clause_ids": [], "parts": {}}` và bước `4 reply` từ chối (`status=denied`, `ledger=0`). |
| Regression test | Item `E-002` (`C-002`, `failure_modes: ["FM-03", "FM-08"]`) trong `data/eval/eval_set_v1.jsonl` + grader `db_fact_kept` / `RefundAmountCorrect` (hoàn thành ở M3) — sẽ fail trên `seeded_regression` và pass trên `fixed`. |
| Chưa chắc chắn | Trace trên MockLLM là deterministic theo seed nên không phản ánh xác suất model thật tự bịa `model_override` ở các câu hỏi diễn đạt khác. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
