# Phase 1 — SCAN (Cá nhân, 20 min)

### Danh sách bài toán của tôi:

| #   | Công ty/đơn vị | Góc nhìn (lens)    | Mô tả ngắn bài toán                                                                                                                                                                     |
| --- | -------------- | ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Vinhomes       | Pain từ người khác | Cư dân gặp lỗi khi xác thực khuôn mặt, khiến tác vụ cần xác thực bị gián đoạn.                                                                                                          |
| 2   | Vinpearl       | Tốn thời gian      | Một khách hàng phản ánh rằng khi nhấp vào ưu đãi đặt phòng trên MyVinpearl, ứng dụng/website hiển thị trang trống hoặc lặp lại thông báo ưu đãi, khiến họ phải thử đặt phòng nhiều lần. |
| 3   | VinUni         | Pain từ người khác | Quy trình nộp bài lab qua cổng LMS của chương trình AI Thực chiến chưa cho sinh viên thông báo rõ ràng rằng bài đã được ghi nhận thành công.                                            |
| 4   | Vinmec         | Pain từ người khác | Người dùng MyVinmec phản ánh không muốn nhận thông báo mời khám về các bệnh và dịch vụ không liên quan.                                                                                 |
| 5   | VinFast        | Lặp lại            | Khách hàng muốn thay đổi lịch bảo dưỡng đã tạo phải hủy lịch cũ và đặt lại lịch mới, lặp lại thao tác đặt hẹn.                                                                          |

---

# Phase 2 — QUICK-ASSESS (Cá nhân, 30 min)

Chọn ba bài toán: **VinUni (#3), VinFast (#5), Vinpearl (#2)**. Các metric là mục tiêu đề xuất; thời gian chưa đo được sẽ được ghi rõ.

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1 — VINUNI (SCAN #3)                    │
│                                                             │
│ Bài toán: Sinh viên không rõ bài lab đã được LMS ghi nhận.  │
│ Công ty: [ ] VinFast  [ ] Xanh SM  [ ] Vinhomes             │
│          [ ] Vinmec   [x] Khác: VinUni                      │
│                                                             │
│ Ai đang đau? Sinh viên nộp bài và lab coach xác nhận hộ.    │
│                                                             │
│ Workflow hiện tại (5 bước):                                 │
│   1. Mở LMS                                                 │
│   → 2. Chọn đúng bài lab của lớp giữa nhiều mục             │
│   → 3. Dán liên kết bài giải và nộp                         │
│   → 4. Đọc thông báo/trạng thái nhưng vẫn chưa rõ trạng     |
|        thái bài làm                                         │
│   → 5. Hỏi lab coach trên Discord để xác nhận               │
│                                                             │
│ Bước tốn nhất: 4–5, khoảng 5 phút/lượt (ước tính cá nhân).  │
│ AI hỗ trợ ở đâu? Không cần AI; hệ thống kiểm tra bản ghi    │
│ nộp và hiển thị biên nhận hoặc lỗi cụ thể.                  │
│                                                             │
│ Metric mục tiêu: 100% lượt nộp có trạng thái rõ trong       │
│ ≤10 giây; thời gian hỏi coach: ~5 → 0 phút/lượt.            │
│                                                             │
│ Quick Architecture: [ ] No AI [x] Rule [ ] LLM [ ] Agent    │
└─────────────────────────────────────────────────────────────┘
```

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2 — VINFAST (SCAN #5)                   │
│                                                             │
│ Bài toán: Đổi lịch bảo dưỡng phải hủy lịch cũ và đặt lại.   │
│ Công ty: [x] VinFast  [ ] Xanh SM  [ ] Vinhomes             │
│          [ ] Vinmec   [ ] Khác                              │
│                                                             │
│ Ai đang đau? Chủ xe cần đổi lịch bảo dưỡng.                 │
│                                                             │
│ Workflow hiện tại (4 bước):                                 │
│   1. Đăng nhập để xem lịch đã đặt                           │
│   → 2. Hủy lịch cần thay đổi                                │
│   → 3. Điền lại thông tin khách, xe, dịch vụ, nơi và giờ    │
│   → 4. Gửi yêu cầu và chờ xưởng xác nhận                    │
│                                                             │
│ Bước tốn nhất: 3, nhập lại biểu mẫu; 5 phút/lượt.           │
│ AI hỗ trợ ở đâu? Chưa cần AI; cho sửa lịch và giữ lại       │
│ thông tin không thay đổi bằng quy tắc thông thường.         │
│                                                             │
│ Metric mục tiêu: ≥90% lượt đổi lịch xong trong ≤2 phút,     │
│ không nhập lại thông tin khách hàng và xe đã có.            │
│                                                             │
│ Quick Architecture: [ ] No AI [x] Rule [ ] LLM [ ] Agent    │
└─────────────────────────────────────────────────────────────┘
```

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3 — VINPEARL (SCAN #2)                  │
│                                                             │
│ Bài toán: Mở ưu đãi nhưng không tới trang đặt phòng.        │
│ Công ty: [ ] VinFast  [ ] Xanh SM  [ ] Vinhomes             │
│          [ ] Vinmec   [x] Khác: Vinpearl                    │
│                                                             │
│ Ai đang đau? Khách hàng muốn đặt phòng theo ưu đãi.         │
│                                                             │
│ Workflow hiện tại (4 bước, theo một phản ánh):              │
│   1. Mở ứng dụng/website để tìm phòng và ưu đãi             │
│   → 2. Nhấp vào ưu đãi                                      │
│   → 3. Gặp trang trống hoặc vòng lặp thông báo              │
│   → 4. Thử lại trên ứng dụng, web hoặc thiết bị khác        │
│                                                             │
│ Bước tốn nhất: 3–4; một khách thử trong 3 ngày, 15          │
│ giây/lượt. Cần kiểm tra lại trên phiên bản hiện tại.        │
│ AI hỗ trợ ở đâu? Chưa cần AI; kiểm tra ưu đãi và sửa luồng  │
│ chuyển trang nếu tái hiện được lỗi.                         │
│                                                             │
│ Metric mục tiêu: ≥95% lượt nhấp vào ưu đãi còn hiệu lực     │
│ mở được trang đặt phòng hợp lệ trong ≤5 giây.               │
│                                                             │
│ Quick Architecture: [x] No AI [ ] Rule [ ] LLM [ ] Agent    │
└─────────────────────────────────────────────────────────────┘
```
