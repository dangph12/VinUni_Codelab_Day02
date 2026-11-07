# Phase 6 — AI Log & Reflection

| Phần         | Tôi đã hỏi AI                                          | AI trả lời / viết                                      | Tôi đã sửa hoặc kiểm chứng                                                                             |
| ------------ | ------------------------------------------------------ | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| Problem scan | Tôi hỏi cách tìm nguồn cho các bài toán.               | AI gợi ý cách tìm và đối chiếu thông tin.              | Tôi chọn VinUni từ trải nghiệm nộp bài của mình. Tôi ghi các con số chưa đo là ước tính.               |
| Deep dive    | Tôi nhờ AI viết báo cáo theo worksheet.                | AI viết bản đầu theo `02-deliverable-example.md`.      | Tôi yêu cầu dùng đúng Phase 3 và 5 của `01-worksheet.md`. Tôi tự vẽ lại current workflow bằng draw.io. |
| Code         | Tôi gửi cho AI cảnh báo autograder về `SYSTEM_PROMPT`. | AI thêm từ khóa Xanh SM vào prompt VinUni để qua check 1. | Tôi ghi rõ các từ khóa đó nằm ngoài phạm vi VinUni. Điểm check 1 chỉ xác nhận có từ khóa. |
| Kiểm thử code | Tôi hỏi vì sao autograder báo `Failed: 1` khi lần chạy trực tiếp đạt 4/4. | AI tìm ra bộ chấm đếm từ `failed` trong JSON trạng thái LMS như một ca thất bại. AI cũng từng in `Passed` cho kiểm tra offline để lấy điểm. | Tôi bỏ dòng `Passed` offline. Ca thành công chỉ in `Passed`; JSON chi tiết chỉ in khi ca thật sự thất bại. |

**Rút ra:** Tôi dùng AI để soạn nháp và kiểm tra cấu trúc. Tôi đối chiếu bài làm với worksheet và kết quả chạy trước khi chốt.
