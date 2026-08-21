# AI Support Log

> Ghi lại bạn đã dùng AI (ChatGPT/Claude/Kimi...) ở những bước nào khi làm deliverables.
> Trung thực là một phần của bài nộp — không ai làm một mình, quan trọng là bạn giữ
> quyền kiểm soát chất lượng.

| # | Bước | AI dùng để làm gì | Bạn kiểm chứng kết quả thế nào |
|---|------|-------------------|-------------------------------|
| 1 | Input Grid | Codex gợi ý candidate dimensions, 13 combinations và các tổ hợp loại | Đối chiếu quy tắc “đổi value → đổi behavior” với slide s25–s30; ghi rõ đây là draft cần PM owner sign-off |
| 2 | Paraphrase Dataset v1 | Codex viết 2 biến thể tự nhiên cho mỗi combination và gắn metadata | Kiểm intent, ambiguity, near-duplicate, expected scope và topic trong corpus; 20 Keep, 6 Rewrite; không nhận là trace thật |
| 3 | Dataset QA | Codex kiểm JSONL, tag distribution, ID duy nhất, slide id và bản evidence | Chạy parser/offline tests; so hash root với evidence; không gọi tutor/judge nên không phát sinh chi phí/trace giả |

- **Gợi ý bị bác bỏ:** dùng “độ khó” và giọng lịch sự/cộc làm dimension; hai thuộc
  tính này không tạo expected behavior ổn định. Các draft tự thêm tên model vào câu
  mơ hồ hoặc chỉ đổi vài từ cũng bị loại/rewrite.
- **Phần cần người nộp tự quyết:** xác nhận 4 dimensions, Keep/Rewrite/Reject cuối
  cùng cho 26 inputs và ký human review. AI không được ghi thay quyết định sản phẩm
  hoặc giả mạo review/trace của con người.
