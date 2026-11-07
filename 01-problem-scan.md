# 01 — Problem Scan & Quick Cards (Vin Smart Future)

> **Học viên:** Ngô Gia Bảo — Branch: `ngogiabao`
> **Mảng quan tâm:** 🚗 **VinFast (Dịch vụ đổi pin — Battery Swap)** và 🎢 **Vinpearl / VinWonders**
>
> ⚠️ *Các con số thời gian/tỷ lệ trong tài liệu là **ước tính giả định** phục vụ scoping, chưa được kiểm chứng bằng dữ liệu vận hành thật. Cần xác lập baseline trước khi build.*

---

## 🏛️ Bối cảnh

Tôi là AI Product Engineer tại **Vin Smart Future**. Tôi tập trung vào hai mảng:

* **VinFast — mô hình đổi pin (battery swap) cho xe máy điện:** người dùng (đặc biệt là tài xế công nghệ chạy nhiều km/ngày) đến trạm, trả pin cạn và nhận pin đầy trong vài phút thay vì sạc hàng giờ. Mô hình này vận hành tốt khi **pin đầy luôn sẵn ở đúng trạm, pin trả về được đánh giá đúng, và tranh chấp được xử lý nhanh**.
* **Vinpearl / VinWonders:** khách sạn, resort và khu vui chơi với lượng khách lớn, đa quốc tịch, nhiều kênh đặt phòng và review.

---

# 🔍 Phase 1 — SCAN (7 bài toán, dùng 4 Lenses)

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|------------|------|---------------------|
| 1 | **VinFast** (Battery Swap) | Pain từ người khác | **Trạm đổi pin hết pin đầy giờ cao điểm.** Tài xế đến trạm không còn pin đầy, phải chờ sạc hoặc chạy sang trạm khác (mất ~15–25 phút/lần). Việc điều chuyển pin giữa các trạm do nhân viên vận hành quyết định thủ công theo kinh nghiệm. |
| 2 | **VinFast** (Battery Swap) | Lặp lại | **Đánh giá tình trạng pin khi khách trả.** Nhân viên trạm kiểm tra ngoại quan + đọc thông số BMS (SoH, số chu kỳ sạc, mã lỗi, nhiệt độ) để quyết định pin *quay vòng / bảo trì / loại bỏ*. ~3–5 phút/viên, tiêu chí mỗi người một kiểu. |
| 3 | **VinFast** (Battery Swap) | Tốn thời gian | **Xử lý tranh chấp pin trả về bị hư hỏng.** Pin trả về bị móp vỏ, vào nước, hoặc SoH tụt bất thường. CSKH phải đối chiếu lịch sử thuê, ảnh chụp tại trạm, log BMS, điều khoản gói thuê rồi soạn phản hồi quy trách nhiệm phí (~30–45 phút/ca), dễ gây khiếu nại. |
| 4 | **VinFast** (Battery Swap) | AI có thể tốt hơn | **Tổng đài/chatbot hỏi đáp gói thuê pin & đổi pin.** Khách hỏi giá gói, cách tính km/lượt đổi, trạm gần nhất, lỗi "trạm không nhận pin". Chatbot kịch bản cứng trả lời rập khuôn, ~40% câu hỏi là lặp lại nhưng vẫn phải chuyển nhân viên. |
| 5 | **VinWonders** | Tốn thời gian | **Điều tiết hàng chờ trò chơi giờ cao điểm.** Trò "hot" chờ 60–90 phút trong khi trò khác vắng. Quản lý khu theo dõi qua camera + bộ đàm, điều hướng khách thủ công, thông báo thời gian chờ không cập nhật. |
| 6 | **Vinpearl** | Pain từ người khác | **Lọc review khẩn cấp đa kênh, đa ngôn ngữ.** Review trên Booking, Agoda, Google Maps (Việt, Anh, Hàn, Trung) được nhân viên đọc thủ công mỗi sáng; phàn nàn nghiêm trọng ("phòng bẩn", "ngộ độc thực phẩm") có thể tới tay Manager trễ 24–48 giờ. |
| 7 | **Vinpearl** | Lặp lại | **Xử lý email đặt phòng theo đoàn (Group Booking).** Đại lý lữ hành gửi email không theo mẫu (danh sách khách, ngày, loại phòng, đôi khi kèm Excel). Nhân viên Reservation nhập tay vào PMS và kiểm tra quỹ phòng (~20–40 phút/email), dễ sai ngày hoặc loại phòng. |

**Phân bổ Lens:** Lặp lại (#2, #7) · Tốn thời gian (#3, #5) · AI có thể tốt hơn (#4) · Pain từ người khác (#1, #6) — đủ cả 4 lenses.

---

# 🃏 Phase 2 — QUICK-ASSESS (3 Quick Problem Cards)

Chọn top 3: **#2 (Đánh giá pin trả về)**, **#3 (Tranh chấp pin hư hỏng)**, **#7 (Group Booking Vinpearl)**.

## Card #1 — VinFast: Đánh giá tình trạng pin khi khách trả

```text
┌─ QUICK PROBLEM CARD #1 ───────────────────────────────────────
│
│ Bài toán: Nhân viên trạm đổi pin phải tự đánh giá mỗi viên pin
│   khách trả về để quyết định quay vòng / bảo trì / loại bỏ.
│ Công ty thành viên: [x] VinFast (Battery Swap)
│
│ Ai đang đau (Actor)? Nhân viên vận hành trạm đổi pin;
│   gián tiếp: khách nhận phải pin kém chất lượng.
│
│ Workflow thủ công hiện tại (5 bước):
│   1. Khách trả pin vào khay/tủ đổi pin
│   ──> 2. Nhân viên kiểm tra ngoại quan (vỏ, chân tiếp xúc, ẩm)
│   ──> 3. Mở phần mềm BMS đọc SoH, số chu kỳ, mã lỗi, nhiệt độ
│   ──> 4. Tự phán đoán: quay vòng / bảo trì / loại bỏ
│   ──> 5. Ghi chú vào sổ/hệ thống, xếp pin vào khu tương ứng
│
│ Bước nào tốn thời gian/lỗi nhất? Bước 3–4 (⏱ ~3–5 phút/viên),
│   tiêu chí không đồng nhất giữa các nhân viên/ca.
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3–4: tự đọc log
│   BMS + ảnh chụp, đề xuất phân loại kèm lý do.
│
│ Đo thành công bằng gì (Metric có số)?
│   - Giảm thời gian đánh giá từ ~4 phút ──> dưới 30 giây/viên
│   - ≥ 95% đề xuất trùng với kết luận của kỹ thuật viên
│
│ Quick Architecture: [ ] No AI  [x] Rule  [ ] LLM  [ ] Agent
│   (Ngưỡng SoH/chu kỳ/mã lỗi là dữ liệu có cấu trúc ──> rule-based
│    là đủ; LLM chỉ cần nếu phải đọc ghi chú tự do/ảnh hư hỏng.)
└───────────────────────────────────────────────────────────────
```

## Card #2 — VinFast: Xử lý tranh chấp pin trả về bị hư hỏng

```text
┌─ QUICK PROBLEM CARD #2 ───────────────────────────────────────
│
│ Bài toán: CSKH mất nhiều thời gian tổng hợp bằng chứng và soạn
│   phản hồi khi pin khách trả về bị hư hỏng/chai bất thường.
│ Công ty thành viên: [x] VinFast (Battery Swap)
│
│ Ai đang đau (Actor)? Nhân viên CSKH dịch vụ thuê pin;
│   khách hàng (chờ kết quả, bức xúc vì bị tính phí).
│
│ Workflow thủ công hiện tại (5 bước):
│   1. Trạm gắn cờ pin bất thường, tạo ticket
│   ──> 2. CSKH tra lịch sử thuê/đổi pin của khách
│   ──> 3. Đối chiếu ảnh tại trạm + log BMS (va đập, nhiệt, nước)
│   ──> 4. Tra điều khoản gói thuê, xác định ai chịu phí
│   ──> 5. Soạn phản hồi gửi khách, chờ khách phản hồi lại
│
│ Bước nào tốn thời gian/lỗi nhất? Bước 3–5 (⏱ ~30–45 phút/ca)
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 3–5: tóm tắt
│   timeline bằng chứng, trích điều khoản liên quan, DRAFT phản hồi.
│   (Quyết định thu phí VẪN do nhân viên duyệt.)
│
│ Đo thành công bằng gì (Metric có số)?
│   - Giảm thời gian xử lý từ ~40 phút ──> dưới 10 phút/ca
│   - Giảm tỷ lệ khách khiếu nại lần 2 từ ~30% ──> dưới 15%
│
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent
└───────────────────────────────────────────────────────────────
```

## Card #3 — Vinpearl: Xử lý email đặt phòng theo đoàn

```text
┌─ QUICK PROBLEM CARD #3 ───────────────────────────────────────
│
│ Bài toán: Nhân viên Reservation nhập tay email đặt phòng đoàn
│   không theo mẫu vào PMS, chậm và dễ sai.
│ Công ty thành viên: [x] Khác: Vinpearl (Hotel & Resort)
│
│ Ai đang đau (Actor)? Nhân viên Reservation; đại lý lữ hành
│   (chờ xác nhận lâu, nhận xác nhận sai).
│
│ Workflow thủ công hiện tại (4 bước):
│   1. Đọc email đoàn (nội dung tự do + file Excel đính kèm)
│   ──> 2. Trích tên đoàn, ngày đến/đi, số phòng, loại phòng
│   ──> 3. Kiểm tra quỹ phòng trống trên PMS
│   ──> 4. Nhập booking + soạn email xác nhận/đề xuất thay thế
│
│ Bước nào tốn thời gian/lỗi nhất? Bước 2–3 (⏱ ~20–40 phút/email)
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2 + 4: trích xuất
│   thông tin ra JSON chuẩn, draft email xác nhận; bước 3 do
│   hệ thống PMS kiểm tra quỹ phòng (không để LLM tự đoán).
│
│ Đo thành công bằng gì (Metric có số)?
│   - Giảm thời gian xử lý từ ~30 phút ──> dưới 5 phút/email
│   - Độ chính xác trích xuất ngày & số phòng ≥ 98%
│
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent
└───────────────────────────────────────────────────────────────
```

---

# 🧪 Stress-test các thẻ (vai CFO / Trưởng phòng Vận hành)

| Card | Điểm yếu bị phản biện | Phản hồi / điều chỉnh |
|---|---|---|
| #1 Đánh giá pin | SoH, số chu kỳ, mã lỗi đều là số ──> một bảng ngưỡng rule-based làm nhanh, rẻ và giải thích được hơn LLM. | Đồng ý: chọn **Rule** làm kiến trúc chính. Chỉ cân nhắc AI (vision) cho phần ảnh hư hỏng ngoại quan, sau khi có dữ liệu ảnh gán nhãn. |
| #2 Tranh chấp pin | Rủi ro pháp lý/tài chính nếu AI kết luận sai trách nhiệm phí; metric "khiếu nại lần 2" chưa có baseline. | AI **chỉ soạn nháp**, nhãn `[DRAFT_ONLY]`, nhân viên duyệt mọi quyết định thu phí. Cần đo baseline 1–2 tháng ticket trước. |
| #3 Group Booking | LLM có thể "bịa" số phòng trống hoặc hiểu sai định dạng ngày (dd/mm vs mm/dd). | Quỹ phòng **luôn lấy từ PMS**, LLM chỉ trích xuất; ngày mơ hồ bắt buộc hỏi lại đại lý; nhân viên duyệt trước khi gửi. |

---

# 🗳️ Đề xuất cho Deep-Dive

Đề xuất mang **Card #2 — VinFast: Xử lý tranh chấp pin trả về bị hư hỏng** vào Deep-Dive nhóm:

* **Phù hợp LLM thật sự:** cần đọc nhiều nguồn phi cấu trúc (ghi chú trạm, lịch sử, điều khoản) và soạn văn bản phản hồi — đúng thế mạnh của LLM.
* **Ranh giới vận hành rõ ràng:** AI chỉ draft, con người quyết định tiền phí ──> rủi ro kiểm soát được qua HITL.
* **Gắn với trọng tâm đổi pin** mà tôi quan tâm, và có tác động trực tiếp đến trải nghiệm & giữ chân khách thuê pin.

Lý do không chọn các thẻ khác:
* **Card #1:** Rule-based giải quyết tốt hơn ──> là ví dụ cho quyết định **NO-GO với LLM**, không cần Deep-Dive AI.
* **Card #3:** Giá trị cao nhưng phụ thuộc tích hợp PMS của Vinpearl, khó mô phỏng workflow thực tế trong phạm vi lab.
* **Bài toán #1 (cân bằng pin giữa các trạm):** bản chất là dự báo nhu cầu + tối ưu điều chuyển (forecasting/optimization), không phải bài toán ngôn ngữ ──> LLM không phải công cụ phù hợp.
