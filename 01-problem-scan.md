# Phase 1 — SCAN & Phase 2 — QUICK-ASSESS: Vin Smart Future (Vingroup)

> **Báo cáo cá nhân:** Quét bài toán & Đánh giá nhanh  
> **Người viết:** Phạm Nguyễn Tất Đạt — MSSV `2A202603401`  
> **Vai trò:** AI Product Engineer, Vin Smart Future  
> **Lĩnh vực trọng tâm:** **VinSOC — An ninh mạng Vingroup** (cập nhật lỗ hổng mới hàng ngày)

---

# 🔍 Phase 1 — SCAN: Bảng Quét Cơ Hội AI (4 Lenses)

Dùng 4 thấu kính **Lặp lại (Repetitive)**, **Tốn thời gian (Time-consuming)**, **AI-upgrade**, **Pain từ người khác (Stakeholder Pain)** để quét vận hành các đơn vị thành viên Vingroup. Tối thiểu 5 bài toán.

| # | Đơn vị thành viên | Lens áp dụng | Mô tả ngắn bài toán vận hành & Bottleneck |
|---|-------------------|--------------|------------------------------------------|
| 1 | **VinSOC** | Tốn thời gian (Time-consuming) | Mỗi ca analyst đọc feed lỗ hổng mới (NVD, mail vendor, Slack intel), copy CVE / CVSS / sản phẩm dính vào ticket, map sang hệ thống VinFast–Xanh SM–Vinhomes–Vinmec–Vinpearl, rồi gán đội vá. 1 advisory ~25–30 phút; 10–15 advisory/ca thì cả buổi chỉ để *update lỗ hổng*, chưa kịp săn threat. |
| 2 | **Vinhomes** | Lặp lại (Repetitive) | Ghi số điện từng căn theo tháng: nhân viên đi từng tầng, chép chỉ số vào sổ/Excel, nhập lại hệ thống thu phí. Sai 1 số thì hóa đơn lệch cả tháng. ⏱ ~3–5 phút/căn, lặp toàn khu. |
| 3 | **Vinhomes** | AI-upgrade | Phản ánh cư dân trên App (mất nước, hỏng đèn, ồn, gửi xe) vào 1 hộp chung. CSKH đọc tay, gắn nhãn, chuyển đúng BQL tòa. Phản hồi rập khuôn, ticket lạc tòa, SLA 12–24 giờ. |
| 4 | **Xanh SM** | Pain từ người khác (Stakeholder Pain) | Tài xế hết pin giữa đường gọi tổng đài. Điều phối mở GPS, mở dashboard trạm sạc, soạn tin chỉ đường. Tài xế chờ 10–15 phút, khách hủy cuốc. Pain nằm ở tài xế và khách. |
| 5 | **Vinpearl** | Pain từ người khác (Stakeholder Pain) | Review 1 sao trên Booking.com / Agoda / Google đến muộn vì nhân viên quét tay, dịch Nga/Hàn/Trung. Khách đã checkout rồi GM mới biết. |
| 6 | **VinFast** | Lặp lại (Repetitive) | Đối chiếu hóa đơn sạc điện đối tác với log trụ sạc nội bộ mỗi tuần. Hàng nghìn dòng, copy Excel, lệch số thì finance gọi lại. |

---

# 🃏 Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

Chọn top 3 từ bảng SCAN: **#1 (VinSOC update lỗ hổng)**, **#3 (Vinhomes phản ánh)**, **#4 (Xanh SM hết pin)**.

---

### 📇 Quick Problem Card #1: VinSOC — Cập nhật lỗ hổng mới hàng ngày

```text
┌─────────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #1                                                   │
│                                                                         │
│ Bài toán (1 câu): Đọc advisory/CVE mới mỗi ca, bóc tách CVE-ID, CVSS,   │
│ sản phẩm dính, map sang tài sản Vingroup, sinh ticket nháp để analyst   │
│ duyệt rồi mới gán đội vá.                                               │
│ Công ty thành viên: [ ] VinFast  [ ] Xanh SM  [ ] Vinhomes              │
│                     [ ] Vinmec   [x] Khác: VinSOC (An ninh mạng)        │
│                                                                         │
│ Ai đang đau (Actor)? SOC Analyst (ca ngày/ca đêm) và On-call VinSOC.    │
│ Team vá (IT VinFast, Xanh SM, Vinhomes…) chờ ticket chuẩn mới bắt đầu.  │
│                                                                         │
│ Workflow thủ công hiện tại (5 bước):                                    │
│   1. Mở NVD / mail vendor / Slack intel, hốt advisory mới               │
│   → 2. Đọc PDF/HTML, ghi tay CVE-ID, CVSS, mô tả, link vá               │
│   → 3. So với CMDB: sản phẩm này có trong Vingroup không, đơn vị nào    │
│   → 4. Viết ticket tiếng Việt (tóm tắt, ảnh hưởng, hạn SLA)             │
│   → 5. Gán đội vá, theo dõi đến khi đóng                               │
│                                                                         │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2, 3 & 4 (⏱ 20–25 phút/advisory)  │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2, 3 & 4:                    │
│   LLM đọc advisory ──> JSON (CVE, CVSS, product) ──> gợi ý đơn vị       │
│   ──> sinh ticket [DRAFT_ONLY]. Analyst soi map tài sản rồi Approve.    │
│                                                                         │
│ Đo thành công bằng gì (Metric có số)?                                   │
│   - Thời gian từ advisory vào inbox → ticket nháp: 30 phút ──> dưới 5.  │
│   - Trích xuất đúng CVE-ID + CVSS + tên sản phẩm: >= 98% trên bộ mẫu.   │
│   - 100% lỗ hổng Critical (CVSS >= 9.0) có draft trong 15 phút.         │
│                                                                         │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM Feature  [ ] Agent     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### 📇 Quick Problem Card #2: Vinhomes — Phân loại & điều hướng phản ánh cư dân

```text
┌─────────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #2                                                   │
│                                                                         │
│ Bài toán (1 câu): Đọc phản ánh trên App Vinhomes Resident, gắn nhãn     │
│ (điện nước, ồn, gửi xe, an ninh), đẩy đúng BQL tòa, sinh câu trả lời    │
│ nháp cho CSKH duyệt.                                                    │
│ Công ty thành viên: [ ] VinFast  [ ] Xanh SM  [x] Vinhomes              │
│                     [ ] Vinmec   [ ] Khác                               │
│                                                                         │
│ Ai đang đau (Actor)? CSKH App Resident và Ban Quản lý từng tòa.         │
│                                                                         │
│ Workflow thủ công hiện tại (4 bước):                                    │
│   1. Cư dân gửi ticket tự do trên app                                   │
│   → 2. CSKH đọc, đoán loại sự cố                                        │
│   → 3. Chuyển tay sang BQL tòa / kỹ thuật                               │
│   → 4. Soạn phản hồi rập khuôn gửi cư dân                               │
│                                                                         │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 & 3 (⏱ 8–12 phút/ticket,        │
│   lạc tòa thì + vài giờ).                                               │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2, 3 & 4:                    │
│   LLM gắn nhãn + gợi ý tòa + draft reply [DRAFT_ONLY].                  │
│                                                                         │
│ Đo thành công bằng gì (Metric có số)?                                   │
│   - Thời gian route đúng tòa: 12 phút ──> dưới 2 phút.                  │
│   - Tỉ lệ lạc tòa: giảm dưới 5% ticket.                                 │
│                                                                         │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM Feature  [ ] Agent     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### 📇 Quick Problem Card #3: Xanh SM — Điều phối cứu hộ khi xe hết pin

```text
┌─────────────────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #3                                                   │
│                                                                         │
│ Bài toán (1 câu): Khi tài xế Xanh SM báo pin nguy cấp (< 5%), gom GPS   │
│ + loại xe, soạn draft chỉ dẫn trạm gần hoặc kích hoạt cứu hộ pin        │
│ di động; điều phối viên duyệt rồi gửi.                                  │
│ Công ty thành viên: [ ] VinFast  [x] Xanh SM  [ ] Vinhomes              │
│                     [ ] Vinmec   [ ] Khác                               │
│                                                                         │
│ Ai đang đau (Actor)? Tài xế Xanh SM và Điều phối viên tổng đài.         │
│                                                                         │
│ Workflow thủ công hiện tại (5 bước):                                    │
│   1. Tài xế gọi tổng đài báo hết pin                                    │
│   → 2. Điều phối tìm GPS xe                                             │
│   → 3. Lọc trụ sạc trống đúng cổng (CCS2/GBT) và đúng dòng xe           │
│   → 4. Soạn tin chỉ đường gửi App                                       │
│   → 5. Gọi cứu hộ pin nếu xe không tới trạm nổi                         │
│                                                                         │
│ Bước nào tốn thời gian/lỗi nhất? Bước 3 & 4 (⏱ 10–12 phút/lượt)         │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3 & 4:                       │
│   Auto-pull GPS + trụ trống ──> draft SMS. Pin < 5% và trạm > 5 km thì  │
│   bắt buộc JSON dispatch_mobile_charger, cấm chỉ đường trạm xa.         │
│                                                                         │
│ Đo thành công bằng gì (Metric có số)?                                   │
│   - Thời gian xử lý sự cố: 15 phút ──> dưới 3 phút.                     │
│   - Chỉ đúng trụ / đúng cổng: >= 98%.                                   │
│                                                                         │
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM Feature  [ ] Agent     │
└─────────────────────────────────────────────────────────────────────────┘
```

---

# 🎯 Quyết định lựa chọn bài toán Deep-Dive

Nhóm chọn **Quick Problem Card #1 — VinSOC: cập nhật lỗ hổng mới hàng ngày**.

**Lý do chọn Card #1**
1. **Pain gốc của nhóm:** Việc *update lỗ hổng mỗi ngày* đang ăn hết ca analyst. Trễ 1 advisory Critical thì cửa sổ tấn công mở trên hệ thống VinFast / Xanh SM / bệnh viện Vinmec.
2. **AI Fit đúng chỗ:** Advisory là PDF, mail HTML, bản tin tiếng Anh. Rule-based không đọc nổi. LLM làm NER (CVE, CVSS, product). Rule giữ ngưỡng Critical / blast radius. Người map tài sản và Approve.
3. **Ranh giới an toàn bắt buộc:** AI không được tự đóng “không dính”, không được bắn patch, không được gán ticket khi chưa có `[DRAFT_ONLY]` + HITL. Ảnh hưởng ước lượng **> 5%** endpoint của một đơn vị thì escalate on-call, không đi luồng thường.

**Lý do để Card #2 và #3 sang sau**
- **Card #2 (Vinhomes):** Route ticket cư dân có giá trị, rủi ro thấp hơn vá lỗ hổng. Làm phase 2.
- **Card #3 (Xanh SM pin):** Pain realtime rõ. Lab prototype vẫn giữ rule `pin < 5%` + `dispatch_mobile_charger` để stress-test ranh giới dùng chung (`[DRAFT_ONLY]`, ngưỡng 5%). Không phải bài Deep-Dive chính.
