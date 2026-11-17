# Trace case: từ trace tới quyết định sai đầu tiên

Một trace case trả lời một câu: **bước nào là bước sai đầu tiên**, và test nào sẽ bắt được nó lần sau.

```bash
cd src
python -m lab9.trace_case --shape seeded_regression --claim C-002 --seed 11
python -m lab9.trace_case --shape fixed --claim C-002 --seed 11   # đối chứng
```

Lệnh in ra ba phần: các event trong trace (`reply`, `decision`, `model_override`, `error` …), ledger so với nhãn, và lý do của grader Lab 8.

## Mẫu điền (mỗi nhóm 1 case, lấy từ trial đang fail trong report)

| Trường | Nội dung |
|---|---|
| Trial | `seeded_regression C-002@11` |
| Triệu chứng | Grader Lab 8 báo `FAIL (status resolved != denied)`. Reply nói "Dạ, LogiX đã hoàn 20.000đ cho đơn O-202." và ledger ghi `refund_vnd=20000`, `refund_count=1` trong khi nhãn kỳ vọng là `status=denied`, `refund_vnd=0`. |
| Bước sai đầu tiên | Event 3: `model_override {"delay_days": 5}` dẫn tới Event 4: `quote {"amount_vnd": 20000, "reason": "entitled", "clause_ids": ["SHIP-LATE-03"]}` |
| Vì sao là bước *đầu tiên* | Event 1 (handoff) và Event 2 (investigator lookup_order đơn O-202) hoàn toàn hợp lệ, thông tin đơn hàng đúng thực tế. Bắt đầu từ Event 3 xuất hiện ghi đè ngày trễ sai lệch so với điều khoản chính sách. |
| Failure mode | `FM-02` (ignore_denied / bypass policy) |
| Đối chứng | Cùng claim@seed trên shape `fixed`: không có `model_override`. Event 3 là `quote {"amount_vnd": 0, "reason": "no_entitlement"}` và trả về status `denied` chuẩn xác. |
| Regression test | Viết assertion kiểm tra quyết định của agent: `assert decision["status"] == "denied"` và `assert ledger["refund_vnd"] == 0` cho claim C-002. Test này fail trên `seeded_regression` và pass trên `fixed`. |
| Chưa chắc chắn | Trace chỉ thể hiện event `model_override` can thiệp vào facts nhưng không giải thích prompt nội bộ hay reasoning nào đã kích hoạt override này. |

## Quy tắc

- Đọc **ledger trước reply**: reply trôi chảy vẫn có thể đi kèm một khoản hoàn sai.
- "Bước sai đầu tiên" phải dẫn số event cụ thể, không phải "agent bị nhầm".
- Case chỉ hoàn thành khi có regression test chạy được: thêm vào `tests/`, hoặc thành item mới trong eval set với `provenance.from_trace`.
