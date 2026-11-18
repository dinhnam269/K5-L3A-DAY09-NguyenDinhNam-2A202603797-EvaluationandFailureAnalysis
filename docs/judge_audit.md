# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | Có (PASS) | dùng |
| helpful | 100 | 0.375 | 1.000 | 0.300 | Không (FAIL: κ<0.6, R<0.7) | loại |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | `H-008` (`tone_ok`) | PASS | FAIL | Người | Câu *"Em đã hoàn tiền cho anh rồi nha, anh check giúp em."* xưng hô thân mật ("em/anh", "nha"). Cần bổ sung rubric: cấm từ suồng sã ("nha", "nhé", "check"). |
| 2 | `H-017` (`tone_ok`) | PASS | FAIL | Người | Câu *"Dạ, bạn ơi LogiX đã hoàn 25.000đ..."* dùng "bạn ơi", "rồi nha". Rubric cần quy định chuẩn xưng hô "anh/chị" hoặc khách hàng. |
| 3 | `H-029` (`tone_ok`) | PASS | FAIL | Người | *"Hệ thống đang xử lý, không cần nhắn thêm."* mang giọng ra lệnh/đuổi khách. Rubric cần phạt câu ngăn khách phản hồi. |
| 4 | `H-042` (`tone_ok`) | FAIL | PASS | Người | *"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."* hoàn toàn lịch sự, đầy đủ thông tin. Rubric cần nêu ví dụ câu PASS chuẩn để giảm false positive. |
| 5 | `H-051` (`tone_ok`) | FAIL | PASS | Người | *"Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ."* lịch sự, chỉ xen từ tiếng Anh "refund". Rubric cần làm rõ từ vay mượn thông dụng không bị tính là lỗi tone. |
| 6 | `HL-016` (`helpful`) | PASS | FAIL | Người | Chỉ xin lỗi chung chung (*"thành thật xin lỗi... trân trọng sự kiên nhẫn"*), không trả lời kết quả xử lý. Rubric `helpful` cần yêu cầu có hành động/kết luận cụ thể cho claim. |

## Kết luận
- **Tiêu chí bị loại:** `helpful` bị loại vì `κ = 0.375 < 0.6` và `Recall = 0.300 < 0.7` (ma trận nhầm lẫn: `tp=9, fp=0, fn=21, tn=70`). Judge bỏ sót 21/30 ca không hữu ích (70% lỗi thật bị cho qua) do bị đánh lừa bởi giọng điệu xin lỗi lịch sự, nên hoàn toàn không thể dùng làm CI gate; nếu dùng sẽ khiến tỉ lệ lỗi quan sát được bị méo nghiêm trọng (`0.09` so với tỉ lệ lỗi thật sau hiệu chỉnh Rogan-Gladen là `0.30`).
- **Tiêu chí được dùng:** `tone_ok` đạt cả 3 ngưỡng (`κ = 0.757`, `P = 0.857`, `R = 0.800`; `tp=24, fp=4, fn=6, tn=66`; Rogan-Gladen hiệu chỉnh `0.28 -> 0.30`), đủ tin cậy để theo dõi chất lượng giọng điệu sau khi siết rubric 5 ca bất đồng ở trên.
- Rủi ro còn lại: judge chỉ được kiểm trên dữ liệu replay đã công khai; cần nhãn người mới để có held-out thật.
