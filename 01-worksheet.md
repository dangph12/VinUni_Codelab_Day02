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
│ Bài toán (1 câu): Đọc advisory/CVE mới mỗi ca, map tài sản  │
│ Vingroup, sinh ticket nháp [DRAFT_ONLY] cho analyst duyệt.  │
│ Công ty thành viên: [ ] VinFast  [ ] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [x] Khác: VinSOC           │
│                                                             │
│ Ai đang đau (Actor)? SOC Analyst ca ngày/ca đêm VinSOC      │
│                                                             │
│ Workflow thủ công hiện tại (5 bước):                        │
│   1. Hốt advisory ──> 2. Đọc bóc CVE/CVSS ──>               │
│   3. Map CMDB ──> 4. Viết ticket ──> 5. Gán đội vá          │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2, 3, 4 (⏱ 25 phút)   │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2, 3, 4 (NER     │
│ advisory -> gợi ý đơn vị -> ticket [DRAFT_ONLY])            │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                        │
│   Giảm thời gian intake từ 30 phút ──> dưới 5 phút/advisory.│
│                                                             │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent │
└─────────────────────────────────────────────────────────────┘
```

```text
┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                       │
│                                                             │
│ Bài toán (1 câu): Phân loại phản ánh cư dân App Resident,    │
│ đẩy đúng BQL tòa, sinh câu trả lời nháp cho CSKH duyệt.     │
│ Công ty thành viên: [x] Vinhomes                            │
│ Ai đang đau? CSKH App Resident, Ban Quản lý tòa             │
│ Workflow thủ công: Đọc ticket -> Đoán loại -> Chuyển tòa    │
│ Bước tốn nhất: Gắn nhãn + route đúng tòa (8–12 phút)        │
│ AI hỗ trợ: LLM gắn nhãn + draft reply [DRAFT_ONLY]          │
│ Metric: Route đúng tòa 12 phút ──> dưới 2 phút              │
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
* 🔴 **Bottleneck:** Bước 2 (Đọc advisory - 10 phút), Bước 3 (Map CMDB - 8 phút), Bước 4 (Viết ticket - 7 phút).
* 🔄 **Handoff:** Nguồn intel ──> Inbox VinSOC ──> Analyst ──> Jira/SNOW ──> Đội vá đơn vị.
* **Tổng thời gian xử lý trung bình:** **30 phút/advisory**.

## 3.2. Problem Statement (6-field) & Metrics

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | SOC Analyst ca ngày/ca đêm tại VinSOC. On-call nhận escalate Critical. |
| **2. Current Workflow** | Hốt advisory NVD/mail/Slack, đọc tay CVE–CVSS–sản phẩm, map CMDB sang đơn vị Vingroup, viết ticket tiếng Việt, gán đội vá. ~30 phút/advisory. |
| **3. Bottleneck** | Đọc PDF/mail tự do, map tên sản phẩm sang tài sản thật, soạn ticket (25/30 phút). Critical ca đêm nằm inbox đến sáng. |
| **4. Business Impact** | 5–7 giờ công/ca chỉ để intake. ~30% ticket Critical/High trễ SLA. Đóng nhầm “không dính” thì lỗ hổng sống trên production. |
| **5. Success Metric** | - Inbox → ticket nháp: 30 phút xuống dưới 5 phút.<br>- Trích xuất đúng CVE-ID + CVSS + sản phẩm >= 98%.<br>- 100% CVSS >= 9.0 có draft trong 15 phút. |
| **6. Operational Boundary** | AI chỉ sinh `[DRAFT_ONLY]`. Cấm tự đóng not-affected, cấm patch. CVSS >= 9 hoặc blast radius > 5% endpoint → on-call. |

## 3.3. Future-State Flow & AI Fit
* **AI Fit:** **LLM Feature** kết hợp Human-in-the-loop (HITL) và Rule validation.
* **Quy trình tương lai:**
  1. Connector ingest NVD/mail/Slack.
  2. 🔵 **AI Step:** Gemini bóc JSON (CVE, CVSS, product) + tóm tắt tiếng Việt.
  3. 🔵 **Rule Step:** Bucket CVSS + ngưỡng 5% blast radius + gắn `[DRAFT_ONLY]`.
  4. 🟢 **HITL Step:** Analyst map CMDB 1–2 phút, Approve rồi gán đội vá.
  5. ↩️ **Fallback:** PDF mờ / thiếu CVE / không map được → hàng đợi thủ công. Không đoán “không dính”.

---

# 💻 Phase 4 — TECHNICAL PROMPT PROTOTYPE (Nhóm, 30 min)

Mã nguồn đã hoàn thiện và kiểm thử thành công tại `starter-code/prompt_prototype.py`.
* Thực hiện đúng chuẩn **Google Gemini 2.5 Flash**.
* Tích hợp chỉ thị ranh giới cấm nghiêm ngặt: tiền tố `[DRAFT_ONLY]`, ngưỡng an toàn `< 5%`, tự động kích hoạt `dispatch_mobile_charger` khi xảy ra tình huống cứu hộ pin khẩn cấp.
* Vượt qua 100% bài kiểm thử đối kháng (Adversarial Tests) và đạt điểm tuyệt đối trên `autograder/autograder.py`.

---

# 🏁 Phase 5 — EVALUATE (Nhóm, 20 min)

### AI Readiness Checklist:
1. [x] Chúng tôi có sẵn dữ liệu mẫu/logs sạch để test (NVD + mail vendor để gắn nhãn CVE/CVSS/product).
2. [x] Rủi ro khi AI sai có nằm trong tầm kiểm soát (`[DRAFT_ONLY]` + cấm auto-close + cấm patch + HITL + Fallback).
3. [x] Stakeholders sẵn sàng thay đổi quy trình làm việc cũ (Analyst VinSOC ủng hộ vì cắt khâu đọc PDF/copy CVSS).

### Quyết định cuối cùng của Ban Giám đốc Vin Smart Future:
[x] **GO (Bắt đầu xây dựng Prototype & Triển khai Thí điểm):** Thí điểm 4 tuần luồng advisory NVD + mail vendor, chỉ sinh ticket nháp.

**Justification:** Giảm ~85% thời gian intake (30 phút → dưới 5 phút), Critical ca đêm có draft trong 15 phút, ranh giới cứng chặn auto-patch / auto-close. Production rộng chờ CMDB sạch và đo 98% trên advisory thật.

---

# 📝 Phase 6 — REFLECTION (Cá nhân)
*(Chi tiết xem tại file `03-ai-log.md`)*
* Đã phản ánh trung thực quá trình dùng AI làm Thought Partner, các ảo giác ban đầu của AI về việc tự động hoá quá đà (Autonomous Agent) và cách thiết lập ranh giới cứng (Hard Guardrails) để kiểm soát chất lượng giải pháp.
