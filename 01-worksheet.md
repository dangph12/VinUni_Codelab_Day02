# Lab 02 — Worksheet: AI Product Scoping (Vin Smart Future)

---

## 🏛️ 1. Bối cảnh thực tế: Vin Smart Future (Vingroup)

**Vingroup** — Tập đoàn tư nhân lớn nhất Việt Nam — vừa sáp nhập toàn bộ các phòng ban công nghệ thuộc các công ty thành viên thành một đơn vị công nghệ thống nhất mang tên **Vin Smart Future**. 

Nhiệm vụ của **Vin Smart Future** là xây dựng các giải pháp AI, số hóa, và tự động hóa cốt lõi để nâng cao hiệu suất vận hành và trải nghiệm khách hàng xuyên suốt các công ty thành viên:
* 🚗 **VinFast:** Hệ thống xe điện thông minh (EV), trợ lý AI ảo trong xe, dự đoán bảo trì pin, và quản lý chuỗi cung ứng sản xuất.
* 🚕 **Xanh SM (GSM):** Vận hành đội xe taxi/xe máy điện thông minh, điều vận thông minh (Smart Dispatching), tối ưu hóa lộ trình di chuyển.
* 🏢 **Vinhomes:** Quản lý đô thị thông minh (Smart Cities), trợ lý cư dân thông minh, tối ưu hóa mức tiêu thụ năng lượng.
* 🏥 **Vinmec:** Y tế thông minh, chẩn đoán hình ảnh bằng AI, tối ưu hóa quản lý hồ sơ bệnh án.
* 🎢 **Vinpearl / VinWonders:** Trải nghiệm du lịch số hóa, quản lý phòng và luồng khách thông minh tại các khu vui chơi.

Trong buổi Lab hôm nay, nhóm chúng tôi đóng vai trò là **AI Product Engineer** tại **Vin Smart Future**, tiến hành tìm kiếm, scoping, phân tích độ khả thi, thiết lập ranh giới vận hành, và xây dựng bản mẫu kỹ thuật cho bài toán: **VinSOC — cập nhật lỗ hổng mới hàng ngày (Daily Vuln Intake Co-pilot)**.

---

# 🔍 Phase 1 — SCAN (Cá nhân, 20 min)

### 📝 List bài toán của tôi (4 Lenses):
| # | Subsidiary (VinFast/Xanh SM...) | Lens | Mô tả ngắn bài toán |
|---|----------------------------------|------|---------------------|
| 1 | **VinSOC** | Time-consuming | Đọc advisory/CVE mỗi ca, bóc CVE–CVSS–sản phẩm, map CMDB, sinh ticket nháp cho đội vá. |
| 2 | **Vinhomes** | Repetitive | Ghi số điện từng căn hộ theo tháng, chép sổ/Excel rồi nhập hệ thống thu phí. |
| 3 | **Vinhomes** | AI-upgrade | Phân loại và định tuyến phản ánh cư dân trên App Resident về đúng BQL tòa. |
| 4 | **Xanh SM** | Stakeholder Pain | Điều phối cứu hộ / trạm sạc khi taxi điện báo pin nguy cấp (< 5%). |
| 5 | **Vinpearl** | Stakeholder Pain | Gom review 1 sao OTA, dịch, cảnh báo GM trong 30 phút. |
| 6 | **VinFast** | Repetitive | Đối chiếu hóa đơn sạc điện đối tác với log trụ sạc nội bộ mỗi tuần. |

---

# 🃏 Phase 2 — QUICK-ASSESS (Cá nhân, 30 min)

### Top 3 Quick Problem Cards:

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                       │
│                                                             │
│ Bài toán (1 câu): Đọc email đặt phòng đoàn phức tạp, tự     │
│ động đối chiếu phòng trống trên PMS Opera và tạo draft đơn. │
│ Công ty thành viên: [ ] VinFast  [ ] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [x] Khác: Vinpearl         │
│                                                             │
│ Ai đang đau (Actor)? Nhân viên Đặt phòng (Reservation Ops)  │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                        │
│   1. Nhận email đoàn ──> 2. Đọc bóc tách thông tin ──>      │
│   3. Tra cứu PMS Opera ──> 4. Tính giá báo giá ──>          │
│   5. Gõ lệnh tạo booking nháp trên PMS                      │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2, 3, 5 (⏱ 26 phút)   │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2, 3, 5 (Trích   │
│ xuất thông tin -> Tra cứu API -> Sinh draft booking)        │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│   Giảm thời gian xử lý từ 35 phút ──> dưới 5 phút/booking.  │
│                                                             │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent │
└─────────────────────────────────────────────────────────────┘
```

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán (1 câu): Tổng hợp và cảnh báo review khẩn cấp của  │
│ khách hàng trên các trang OTA gửi Ban quản lý Vinpearl.     │
│ Công ty thành viên: [x] Vinpearl                            │
│ Ai đang đau? Giám đốc Vận hành Khách sạn, Chăm sóc Khách hàng│
│ Workflow thủ công: Quét từng trang -> Đọc dịch -> Lập Excel │
│ Bước tốn nhất: Đọc và dịch review đa ngôn ngữ (60 phút)     │
│ AI hỗ trợ: Tự động dịch và gom cụm khiếu nại nghiêm trọng   │
│ Metric: Rút ngắn thời gian phát hiện từ 24h ──> dưới 30 phút│
│ Quick Architecture: [x] LLM Feature                         │
└─────────────────────────────────────────────────────────────┘
```

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                       │
│                                                             │
│ Bài toán (1 câu): Hỗ trợ điều phối viên tra cứu trạm sạc xe │
│ điện Xanh SM và điều xe cứu hộ khi pin dưới 5%.             │
│ Công ty thành viên: [x] Xanh SM                             │
│ Ai đang đau? Tài xế taxi, Điều phối viên tổng đài           │
│ Bước tốn nhất: Tra cứu trạm sạc còn trụ trống (10 phút)     │
│ AI hỗ trợ: Tự động gom dữ liệu định vị và sinh tin chỉ dẫn  │
│ Metric: Giảm thời gian xử lý từ 15 phút ──> dưới 3 phút     │
│ Quick Architecture: [x] LLM Feature                         │
└─────────────────────────────────────────────────────────────┘
```

---

# 🏗️ Phase 3 — DEEP-DIVE (Nhóm, 85 min)

## 3.1. Current-State Workflow Mapping
*(Chi tiết xem file sơ đồ trực quan `04-workflow-diagram.png` và báo cáo `02-deep-dive-report.md`)*
* 🔴 **Bottleneck:** Bước 2 (Trích xuất thông tin đoàn - 8 phút), Bước 3 (Tra cứu quỹ phòng PMS Opera - 10 phút), Bước 5 (Gõ dữ liệu tạo booking - 8 phút).
* 🔄 **Handoff:** Khách/Đại lý ──> Email ──> Nhân viên Đặt phòng ──> PMS Opera ──> Email phản hồi.
* **Tổng thời gian xử lý trung bình:** **35 phút/booking đoàn**.

## 3.2. Problem Statement (6-field) & Metrics

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Chuyên viên Đặt phòng (Reservation Officer) và Sales B2B Vinpearl. |
| **2. Current Workflow** | Nhận email đặt phòng đoàn từ đại lý, đọc bóc tách thủ công danh sách khách, mở PMS Opera kiểm tra phòng trống, tính giá chiết khấu B2B, soạn email báo giá và nhập tay từng trường thông tin tạo booking. Mất 35 phút. |
| **3. Bottleneck** | Đọc hiểu văn bản email tự do, tra cứu tính khả dụng của nhiều hạng phòng cùng lúc và nhập tay vào hệ thống PMS (chiếm 26/35 phút). |
| **4. Business Impact** | Mỗi ngày tiếp nhận 60–100 booking đoàn, tốn 35–50 giờ công/ngày. Chậm trễ phản hồi khiến rò rỉ 8–12% doanh thu vào tay đối thủ. Sai sót nhập liệu gây khiếu nại. |
| **5. Success Metric** | - Rút ngắn thời gian xử lý từ 35 phút xuống dưới 5 phút.<br>- Tỉ lệ trích xuất chính xác thông tin phòng và số khách đạt >= 98%.<br>- 100% email booking đoàn có báo giá sơ bộ trong vòng 15 phút. |
| **6. Operational Boundary** | AI chỉ sinh bản nháp và luôn gắn tag `[DRAFT_ONLY]` ở đầu để bắt buộc nhân viên phê duyệt (HITL). Tuyệt đối cấm tự ý xác nhận booking khi quỹ phòng ở mức nguy cấp (< 5% công suất). |

## 3.3. Future-State Flow & AI Fit
* **AI Fit:** **LLM Feature** kết hợp Human-in-the-loop (HITL) và Rule validation.
* **Quy trình tương lai:**
  1. Hệ thống tự động nhận email.
  2. 🔵 **AI Step:** Gemini LLM bóc tách thông tin thành JSON chuẩn.
  3. 🔵 **AI & API Step:** Kiểm tra quỹ phòng trống thời gian thực trên PMS và tạo bản nháp `[DRAFT_ONLY]`.
  4. 🟢 **HITL Step:** Nhân viên đặt phòng kiểm tra 1 phút và nhấn Duyệt (Approve).
  5. ↩️ **Fallback:** Nếu email mờ hoặc format không xác định, chuyển về hàng đợi thủ công.

---

# 💻 Phase 4 — TECHNICAL PROMPT PROTOTYPE (Nhóm, 30 min)

Mã nguồn đã hoàn thiện và kiểm thử thành công tại `starter-code/prompt_prototype.py`.
* Thực hiện đúng chuẩn **Google Gemini 2.5 Flash**.
* Tích hợp chỉ thị ranh giới cấm nghiêm ngặt: tiền tố `[DRAFT_ONLY]`, ngưỡng an toàn `< 5%`, tự động kích hoạt `dispatch_mobile_charger` khi xảy ra tình huống cứu hộ pin khẩn cấp.
* Vượt qua 100% bài kiểm thử đối kháng (Adversarial Tests) và đạt điểm tuyệt đối trên `autograder/autograder.py`.

---

# 🏁 Phase 5 — EVALUATE (Nhóm, 20 min)

### AI Readiness Checklist:
1. [x] Chúng tôi có sẵn dữ liệu mẫu/logs sạch để test (Lịch sử email booking và dữ liệu PMS Opera).
2. [x] Rủi ro khi AI sai có nằm trong tầm kiểm soát (Bảo vệ bằng thẻ `[DRAFT_ONLY]` + HITL + Fallback).
3. [x] Stakeholders sẵn sàng thay đổi quy trình làm việc cũ (Bộ phận Đặt phòng rất ủng hộ vì giảm tải việc thủ công).

### Quyết định cuối cùng của Ban Giám Đốc Vin Smart Future:
[x] **GO (Bắt đầu xây dựng Prototype & Triển khai Thí điểm):** Triển khai thí điểm tại Vinpearl Resort & Spa Nha Trang trong 4 tuần.

**Justification:** ROI cực cao (giảm 85% thời gian xử lý, tăng tỉ lệ chuyển đổi booking lên 10%), chi phí vận hành API rất nhỏ, ranh giới an toàn kiểm soát chặt chẽ ngăn chặn triệt để nguy cơ overbooking.

---

# 📝 Phase 6 — REFLECTION (Cá nhân)
*(Chi tiết xem tại file `03-ai-log.md`)*
* Đã phản ánh trung thực quá trình dùng AI làm Thought Partner, các ảo giác ban đầu của AI về việc tự động hoá quá đà (Autonomous Agent) và cách thiết lập ranh giới cứng (Hard Guardrails) để kiểm soát chất lượng giải pháp.
