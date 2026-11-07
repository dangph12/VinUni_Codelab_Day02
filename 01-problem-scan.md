# 01 — Problem Scan & Quick Cards (Vin Smart Future)

> **Người thực hiện:** Nguyễn Ngọc Toàn — branch `NguyenNgocToan-03405`
> **Vai trò:** AI Product Engineer tại Vin Smart Future
> **Lưu ý về số liệu:** Các con số thời gian và khối lượng dưới đây là **ước tính giả định** dựa trên quan sát người dùng và thảo luận nhóm. Chưa được xác nhận bằng dữ liệu nội bộ của Xanh SM.

---

## 🔍 Phase 1 — SCAN

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|------------|------|---------------------|
| 1 | **Xanh SM** | Tốn thời gian | **Tìm đồ thất lạc trên xe:** khách mô tả đồ để quên bằng tiếng Việt tự do (*"ví da nâu, quên ở ghế sau chuyến chiều qua"*). CSKH phải tự tra lịch sử chuyến, gọi từng tài xế rồi đối chiếu mô tả với đồ nhặt được (~30 phút/yêu cầu). |
| 2 | **Xanh SM** | Pain từ người khác | **Khiếu nại cước phí / đi đường vòng:** khách phản ánh tài xế đi vòng. CSKH phải mở lộ trình GPS và so với tuyến tối ưu bằng mắt trước khi quyết định có hoàn tiền hay không. |
| 3 | **Xanh SM** | Lặp lại | **Kiểm duyệt hồ sơ tài xế mới:** nhân sự đọc và đối chiếu tay CCCD, bằng lái, giấy khám sức khỏe (hạn hiệu lực, hạng bằng, thông tin khớp giữa các giấy tờ) cho hàng trăm hồ sơ/tuần. |
| 4 | **Vinhomes** | Lặp lại | **Phân loại phản ánh cư dân:** khiếu nại (mất nước, hỏng đèn, ồn ào) gửi qua App Vinhomes Resident được chuyển thủ công đến đúng ban quản lý từng tòa. |
| 5 | **Vinpearl** | AI có thể tốt hơn | **Tổng hợp review khách sạn:** quét review trên Booking/Agoda/Google Maps để lọc phàn nàn khẩn cấp (*"phòng bẩn"*, *"nhân viên thái độ"*) và gửi về Manager. Hiện quản lý phải đọc tay và thường chậm vài ngày. |
| 6 | **VinFast** | AI có thể tốt hơn | **Chẩn đoán lỗi sơ bộ từ mô tả của khách:** khách tả *"qua gờ giảm tốc kêu cụp cụp bánh trước"*, hệ thống gợi ý nhóm mã lỗi để xưởng chuẩn bị trước (kỹ sư duyệt). |

---

## 🃏 Phase 2 — QUICK-ASSESS (Top 3: #1, #2, #3)

### Card #1 — Xanh SM: Tìm đồ thất lạc trên xe

```text
┌──────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                            │
│                                                                  │
│ Bài toán: Khách báo quên đồ trên xe Xanh SM, CSKH mất ~30 phút   │
│ để tìm ra chuyến đúng, liên lạc tài xế và xác minh món đồ.       │                                 
│                                                                  │
│ Ai đang đau? Nhân viên CSKH (quá tải), khách (lo lắng, chờ lâu), │
│ tài xế (bị gọi nhiều lần khi đang chạy, đôi khi bị nghi oan).    │
│                                                                  │
│ Workflow thủ công hiện tại:                                      │
│   1. Khách gọi hotline/chat báo mất đồ (mô tả tự do)             │
│   → 2. CSKH tra lịch sử chuyến, đoán chuyến nào khả nghi          │
│   → 3. CSKH gọi tài xế (thường phải gọi 2–3 lần)                  │
│   → 4. Tài xế kiểm tra xe, CSKH đối chiếu mô tả với đồ nhặt được  │
│   → 5. Hẹn trả đồ, đóng ticket                                    │
│                                                                  │
│ Bước nào tốn nhất? Bước 2 + 4 (⏱ ~15 phút/lượt), bước 3 (~8 phút)│
│ AI nhảy vào ở bước nào? Bước 2 + 4: trích xuất mô tả của khách,  │
│ xếp hạng chuyến ứng viên, khớp với báo cáo nhặt đồ, draft tin.   │
│                                                                  │
│ Metric: Giảm thời gian CSKH xử lý từ ~30 phút → ≤ 8 phút/yêu cầu;│
│ chuyến đúng nằm trong top-3 gợi ý ở ≥ 90% yêu cầu.               │
│                                                                  │
│ Quick Architecture: [ ] No AI [x] Rule [x] LLM [ ] Agent         │
│ (Rule lọc chuyến theo SĐT/thời gian + LLM hiểu mô tả & soạn nháp)│
└──────────────────────────────────────────────────────────────────┘
```

### Card #2 — Xanh SM: Khiếu nại cước phí / đi đường vòng

```text
┌──────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                            │
│                                                                  │
│ Bài toán: CSKH xử lý tay khiếu nại "tài xế đi vòng, tính sai     │
│ tiền" bằng cách so lộ trình GPS với tuyến tối ưu.                │
│ Công ty thành viên: [x] Xanh SM                                  │
│                                                                  │
│ Ai đang đau? CSKH, khách hàng, tài xế (bị trừ điểm oan)          │
│                                                                  │
│ Workflow thủ công hiện tại:                                      │
│   1. Nhận khiếu nại → 2. Mở log GPS chuyến → 3. Tự vẽ tuyến tối  │
│   ưu trên bản đồ để so → 4. Hỏi tài xế lý do (kẹt xe, cấm đường) │
│   → 5. Quyết định hoàn tiền / bác bỏ                             │
│                                                                  │
│ Bước nào tốn nhất? Bước 3–4 (⏱ ~12 phút/lượt)                    │
│ AI nhảy vào ở bước nào? Bước 3 phần lớn là tính toán (rule +     │
│ routing API). LLM chỉ dùng để tóm tắt lý do và draft phản hồi.   │
│                                                                  │
│ Metric: Giảm thời gian xử lý từ ~20 phút → ≤ 5 phút/khiếu nại.   │
│                                                                  │
│ Quick Architecture: [ ] No AI [x] Rule [x] LLM (phụ) [ ] Agent   │
└──────────────────────────────────────────────────────────────────┘
```

### Card #3 — Xanh SM: Kiểm duyệt hồ sơ tài xế mới

```text
┌──────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                            │
│                                                                  │
│ Bài toán: Nhân sự đối chiếu tay giấy tờ của tài xế ứng tuyển.    │
│ Công ty thành viên: [x] Xanh SM                                  │
│                                                                  │
│ Ai đang đau? Nhân viên tuyển dụng, ứng viên (chờ duyệt lâu)      │
│                                                                  │
│ Workflow thủ công hiện tại:                                      │
│   1. Ứng viên upload ảnh giấy tờ → 2. Nhân sự đọc và nhập tay    │
│   thông tin → 3. Đối chiếu tên/ngày sinh/hạn/hạng bằng           │
│   → 4. Duyệt hoặc yêu cầu bổ sung                                │
│                                                                  │
│ Bước nào tốn nhất? Bước 2–3 (⏱ ~10 phút/hồ sơ)                   │
│ AI nhảy vào ở bước nào? OCR + trích xuất trường, gắn cờ khi các  │
│ giấy tờ không khớp nhau.                                         │
│                                                                  │
│ Metric: ≥ 95% trường được trích xuất đúng; thời gian duyệt       │
│ ≤ 3 phút/hồ sơ.                                                  │
│                                                                  │
│ Quick Architecture: [ ] No AI [x] Rule [x] LLM/OCR [ ] Agent     │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🗳️ Lựa chọn Deep-Dive: Card #1 — Tìm đồ thất lạc trên xe Xanh SM

**Lý do chọn:** Mô tả của khách là ngôn ngữ tự nhiên, mơ hồ (*"tầm chiều qua"*, *"cái túi màu sẫm"*), nên rule-based khó xử lý mà LLM lại làm tốt. Bài toán có metric đo được, và rủi ro kiểm soát được nhờ CSKH luôn duyệt trước khi gửi.

**Lý do loại các thẻ khác:**
- **Card #2 (Cước phí):** phần cốt lõi là so sánh lộ trình, tức là bài toán tính toán. Rule + routing API giải quyết tốt hơn LLM, giá trị LLM tăng thêm thấp.
- **Card #3 (Hồ sơ tài xế):** xử lý giấy tờ tùy thân (PII nhạy cảm) nên vướng yêu cầu pháp lý về bảo vệ dữ liệu cá nhân. Ngoài ra, OCR giấy tờ đã có giải pháp eKYC thương mại sẵn, không cần tự xây.
