# Judge audit

Chạy: `python -m lab9.judge --csv <file> --audit 5`

| Tiêu chí | n | κ | Precision | Recall | Đạt FLOORS (κ≥0.6, P≥0.8, R≥0.7)? | Quyết định |
|---|---|---|---|---|---|---|
| tone_ok | 100 | 0.757 | 0.857 | 0.800 | ĐẠT | dùng |
| helpful | 100 | 0.375 | 1.000 | 0.300 | KHÔNG ĐẠT (κ < 0.6, R < 0.7) | loại |

## ≥5 case audit thủ công
| # | Case | Judge nói | Người nói | Ai đúng? | Ghi chú |
|---|---|---|---|---|---|
| 1 | H-008 | PASS | FAIL | Người đúng | Câu reply: *"Em đã hoàn tiền cho anh rồi nha, anh check giúp em."* vi phạm văn phong chuyên nghiệp của doanh nghiệp (xưng hô suồng sã "anh/em", dùng từ lóng "check giúp em"). Judge bỏ sót vi phạm này. |
| 2 | H-017 | PASS | FAIL | Người đúng | Câu reply: *"Dạ, bạn ơi LogiX đã hoàn 25.000đ cho đơn O-606 rồi nha."* dùng khẩu ngữ ("bạn ơi", "rồi nha") thiếu nghiêm túc và trang trọng trong ngữ cảnh xử lý khiếu nại tài chính. |
| 3 | H-029 | PASS | FAIL | Người đúng | Câu reply: *"Dạ, vui lòng chờ. Hệ thống đang xử lý, không cần nhắn thêm."* cộc lốc, mang tính áp đặt gây bức xúc cho khách hàng ("không cần nhắn thêm"). |
| 4 | H-042 | FAIL | PASS | Người đúng | Câu reply: *"Dạ, đơn O-808 đã hoàn 120.000đ. Tiền về trong 3–5 ngày ạ."* đầy đủ lịch sự, nêu rõ số tiền và mốc thời gian. Judge phán xét quá khắt khe (False Positive). |
| 5 | H-051 | FAIL | PASS | Người đúng | Câu reply: *"Dạ, LogiX đã refund 90.000đ cho đơn O-202 ạ."* chêm từ "refund" tiếng Anh, nhưng trong thương mại điện tử Việt Nam từ này phổ biến và khách hiểu rõ. Người chấm chấp nhận được. |

## Kết luận
- **Tiêu chí bị loại và lý do**: `helpful` bị loại thẳng thừng vì độ nhạy/Recall quá thấp ($R = 0.30$, ma trận nhầm lẫn: `tp=9, fp=0, fn=21, tn=70`). Judge bỏ sót tới 21 trên tổng số 30 trường hợp câu trả lời không giải quyết được vấn đề của khách (ví dụ chỉ xin lỗi chung chung). Nếu dùng judge này làm gate, hệ thống sẽ để lọt 70% các lỗi nghiêm trọng về nội dung.
- **Rủi ro còn lại**: Bộ dữ liệu thẩm định hiện tại dựa trên 100 dòng replay công khai; trong môi trường vận hành thực tế cần liên tục thu thập nhãn từ nhân viên hỗ trợ để có tập kiểm thử held-out mới, tránh hiện tượng prompt của judge bị overfit vào bộ test cũ.
