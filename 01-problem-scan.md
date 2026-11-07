# 01 — Problem Scan & Quick Cards (Vin Smart Future)

> **Học viên:** Ngô Gia Bảo — Branch: `ngogiabao`
> **Mảng quan tâm:** 🚗 **VinFast / V-Green (Đổi pin xe máy điện — Battery Swap)** và 🎢 **Vinpearl / VinWonders**
>
> ⚠️ *Các con số thời gian/tỷ lệ trong tài liệu là **ước tính giả định** phục vụ scoping, chưa được kiểm chứng bằng dữ liệu vận hành thật. Cần xác lập baseline trước khi build.*

---

## 🏛️ Bối cảnh

Tôi là AI Product Engineer tại **Vin Smart Future**. Tôi tập trung vào hai mảng:

* **VinFast — mô hình đổi pin xe máy điện (do V-Green vận hành):** người dùng thuê pin, đến **tủ đổi pin tự động** để trả pin cạn và nhận pin đầy (phí ~9.000đ/lượt). Tủ được đặt "ké" tại cây xăng PVOIL, cửa hàng FPT Shop, trạm EVNHANOI, Viettel hoặc điểm của chủ nhượng quyền — **không có nhân viên của dịch vụ đổi pin trực tại tủ**. Tủ tự đọc dữ liệu BMS của pin để nhận/từ chối pin. Mạng lưới đang mở rộng rất nhanh (từ vài nghìn lên mục tiêu hàng chục nghìn tủ).
* **Vinpearl / VinWonders:** khách sạn, resort và khu vui chơi với lượng khách lớn, đa quốc tịch, nhiều kênh đặt phòng và review.

> 📝 *Ghi chú sửa đổi:* bản đầu của file này giả định trạm đổi pin **có nhân viên** kiểm tra pin và tạo ticket. Sau khi tra cứu, mô hình thực tế là tủ tự động không người trực, nên các bài toán #2, #3 và Quick Card #1, #2 đã được viết lại (xem [03-ai-log.md](03-ai-log.md)).

---

# 🔍 Phase 1 — SCAN (7 bài toán, dùng 4 Lenses)

| # | Subsidiary | Lens | Mô tả ngắn bài toán |
|---|------------|------|---------------------|
| 1 | **VinFast** (Đổi pin) | Pain từ người khác | **Tủ đổi pin hết pin đầy giờ cao điểm.** Tài xế đến tủ không còn pin đầy, phải chờ sạc hoặc chạy sang tủ khác (~15–25 phút/lần). Việc điều chuyển pin giữa các tủ do đội vận hành quyết định thủ công theo kinh nghiệm. |
| 2 | **VinFast** (Đổi pin) | Lặp lại | **Phân loại pin lỗi bị tủ gắn cờ.** Pin bị tủ từ chối/gắn cờ nằm lại trong tủ; kỹ thuật viên vận hành đi thu gom, đọc log BMS (SoH, số chu kỳ, mã lỗi, nhiệt độ) và kiểm tra ngoại quan để quyết định *quay vòng / bảo trì / loại bỏ*. ~3–5 phút/viên, tiêu chí mỗi người một kiểu. |
| 3 | **VinFast** (Đổi pin) | Tốn thời gian | **Hỗ trợ từ xa sự cố tại tủ không người trực.** Khách đứng trước tủ gặp lỗi (tủ không nhận pin, báo mã lỗi, kẹt khay, hết pin đầy) và không có ai tại chỗ ──> phải gọi tổng đài. Tổng đài viên hỏi mô tả mơ hồ, tra mã lỗi + log tủ, hướng dẫn qua điện thoại (~20 phút/cuộc gọi), trong khi khách đứng chờ với xe hết pin. |
| 4 | **VinFast** (Đổi pin) | AI có thể tốt hơn | **Chatbot hỏi đáp gói thuê pin & đổi pin.** Khách hỏi giá gói, cách tính phí/lượt đổi, tủ gần nhất, cách thao tác lần đầu. Chatbot kịch bản cứng trả lời rập khuôn, ~40% câu hỏi là lặp lại nhưng vẫn phải chuyển nhân viên. |
| 5 | **VinWonders** | Tốn thời gian | **Điều tiết hàng chờ trò chơi giờ cao điểm.** Trò "hot" chờ 60–90 phút trong khi trò khác vắng. Quản lý khu theo dõi qua camera + bộ đàm, điều hướng khách thủ công, thông báo thời gian chờ không cập nhật. |
| 6 | **Vinpearl** | Pain từ người khác | **Lọc review khẩn cấp đa kênh, đa ngôn ngữ.** Review trên Booking, Agoda, Google Maps (Việt, Anh, Hàn, Trung) được nhân viên đọc thủ công mỗi sáng; phàn nàn nghiêm trọng ("phòng bẩn", "ngộ độc thực phẩm") có thể tới tay Manager trễ 24–48 giờ. |
| 7 | **Vinpearl** | Lặp lại | **Xử lý email đặt phòng theo đoàn (Group Booking).** Đại lý lữ hành gửi email không theo mẫu (danh sách khách, ngày, loại phòng, đôi khi kèm Excel). Nhân viên Reservation nhập tay vào PMS và kiểm tra quỹ phòng (~20–40 phút/email), dễ sai ngày hoặc loại phòng. |

**Phân bổ Lens:** Lặp lại (#2, #7) · Tốn thời gian (#3, #5) · AI có thể tốt hơn (#4) · Pain từ người khác (#1, #6) — đủ cả 4 lenses.

---

# 🃏 Phase 2 — QUICK-ASSESS (3 Quick Problem Cards)

Chọn top 3: **#2 (Phân loại pin lỗi)**, **#3 (Hỗ trợ sự cố tại tủ)**, **#7 (Group Booking Vinpearl)**.

## Card #1 — VinFast: Phân loại pin lỗi bị tủ gắn cờ

```text
┌─ QUICK PROBLEM CARD #1 ───────────────────────────────────────
│
│ Bài toán: Kỹ thuật viên phải tự đánh giá từng viên pin bị tủ
│   gắn cờ để quyết định quay vòng / bảo trì / loại bỏ.
│ Công ty thành viên: [x] VinFast (Battery Swap — V-Green)
│
│ Ai đang đau (Actor)? Kỹ thuật viên vận hành đi thu gom pin;
│   gián tiếp: khách nhận phải pin kém chất lượng.
│
│ Workflow thủ công hiện tại (5 bước):
│   1. Tủ đọc BMS, từ chối/gắn cờ pin bất thường, giữ lại trong tủ
│   ──> 2. Kỹ thuật viên lên lịch tuyến thu gom pin bị gắn cờ
│   ──> 3. Kiểm tra ngoại quan (vỏ, chân tiếp xúc, ẩm)
│   ──> 4. Đọc lại log BMS, tự phán đoán: quay vòng/bảo trì/loại bỏ
│   ──> 5. Ghi chú vào hệ thống, chuyển pin về khu tương ứng
│
│ Bước nào tốn thời gian/lỗi nhất? Bước 3–4 (⏱ ~3–5 phút/viên),
│   tiêu chí không đồng nhất giữa các kỹ thuật viên.
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 4: tự áp ngưỡng
│   SoH/chu kỳ/mã lỗi, đề xuất phân loại kèm lý do.
│
│ Đo thành công bằng gì (Metric có số)?
│   - Giảm thời gian đánh giá từ ~4 phút ──> dưới 30 giây/viên
│   - ≥ 95% đề xuất trùng với kết luận của kỹ thuật viên
│
│ Quick Architecture: [ ] No AI  [x] Rule  [ ] LLM  [ ] Agent
│   (Ngưỡng SoH/chu kỳ/mã lỗi là dữ liệu có cấu trúc ──> rule-based
│    là đủ; tủ vốn đã tự đọc BMS. LLM không cần thiết.)
└───────────────────────────────────────────────────────────────
```

## Card #2 — VinFast: Hỗ trợ từ xa sự cố tại tủ đổi pin không người trực

```text
┌─ QUICK PROBLEM CARD #2 ───────────────────────────────────────
│
│ Bài toán: Khách gặp lỗi tại tủ đổi pin không có người trực,
│   phải gọi tổng đài; tổng đài viên mất ~20 phút/cuộc gọi để
│   hiểu sự cố, tra log và hướng dẫn từ xa.
│ Công ty thành viên: [x] VinFast (Battery Swap — V-Green)
│
│ Ai đang đau (Actor)? Tổng đài viên hỗ trợ đổi pin (quá tải);
│   khách hàng (đứng chờ tại tủ với xe hết pin).
│
│ Workflow thủ công hiện tại (5 bước):
│   1. Khách gặp lỗi tại tủ, tự thử lại, rồi gọi hotline
│   ──> 2. Tổng đài viên hỏi mã tủ, tài khoản, mô tả sự cố
│        (khách mô tả mơ hồ: "tủ kêu bíp, đèn đỏ, không mở")
│   ──> 3. Tra mã lỗi trong tài liệu + xem log tủ, trạng thái pin
│   ──> 4. Hướng dẫn khách xử lý từng bước qua điện thoại
│   ──> 5. Không được: tạo ticket điều kỹ thuật viên, chỉ tủ khác
│
│ Bước nào tốn thời gian/lỗi nhất? Bước 2–4 (⏱ ~17/20 phút)
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2–4: hiểu mô tả
│   tự do + mã lỗi + log ──> đề xuất nguyên nhân và kịch bản hướng
│   dẫn (chỉ từ danh mục thao tác đã duyệt), tổng đài viên duyệt.
│
│ Đo thành công bằng gì (Metric có số)?
│   - Giảm thời gian xử lý từ ~20 phút ──> dưới 8 phút/cuộc gọi
│   - Tỷ lệ xử lý xong ngay cuộc gọi đầu ~45% ──> ≥ 65%
│
│ Quick Architecture: [ ] No AI  [ ] Rule  [x] LLM  [ ] Agent
│   (Rule cho mã lỗi chuẩn + cổng an toàn; LLM cho mô tả tự do.)
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
| #1 Phân loại pin lỗi | SoH, số chu kỳ, mã lỗi đều là số và tủ đã tự đọc BMS ──> một bảng ngưỡng rule-based làm nhanh, rẻ, giải thích được hơn LLM. | Đồng ý: chọn **Rule**. Chỉ cân nhắc AI (vision) cho ảnh hư hỏng ngoại quan, sau khi có dữ liệu ảnh gán nhãn. |
| #2 Hỗ trợ sự cố tại tủ | (a) Nếu lượng đổi pin còn thấp thì số cuộc gọi sự cố ít, một bảng tra mã lỗi là đủ. (b) AI hướng dẫn sai có thể khiến khách tự can thiệp vào pin ──> rủi ro an toàn. | (a) Phải đo số cuộc gọi thật; đặt ngưỡng: dưới ~50 cuộc/ngày thì chỉ làm tra cứu mã lỗi rule-based. (b) AI chỉ được chọn bước từ **danh mục thao tác đã duyệt**, có cổng an toàn chặn mọi ca pin phồng/nóng/khói, và tổng đài viên duyệt trước khi đọc cho khách. |
| #3 Group Booking | LLM có thể "bịa" số phòng trống hoặc hiểu sai định dạng ngày (dd/mm vs mm/dd). | Quỹ phòng **luôn lấy từ PMS**, LLM chỉ trích xuất; ngày mơ hồ bắt buộc hỏi lại đại lý; nhân viên duyệt trước khi gửi. |

---

# 🗳️ Đề xuất cho Deep-Dive

Đề xuất mang **Card #2 — VinFast: Hỗ trợ từ xa sự cố tại tủ đổi pin không người trực** vào Deep-Dive nhóm:

* **Nỗi đau sinh ra từ chính mô hình vận hành:** tủ không người trực nên mọi sự cố đều dồn về tổng đài, và số cuộc gọi tăng theo tốc độ mở rộng mạng lưới tủ.
* **Phù hợp LLM thật sự:** phải hiểu mô tả tự do của khách ("kêu bíp", "đèn đỏ nhấp nháy") và ghép với mã lỗi, log tủ để soạn hướng dẫn dễ hiểu — đúng thế mạnh của LLM.
* **Ranh giới vận hành rõ ràng:** AI chỉ đề xuất từ danh mục thao tác an toàn, tổng đài viên duyệt, ca nguy hiểm đi thẳng tới kỹ thuật viên/cứu hộ.

Lý do không chọn các thẻ khác:
* **Card #1:** Rule-based giải quyết tốt hơn ──> là ví dụ cho quyết định **NO-GO với LLM**, không cần Deep-Dive AI.
* **Card #3:** Giá trị cao nhưng phụ thuộc tích hợp PMS của Vinpearl, khó mô phỏng workflow thực tế trong phạm vi lab.
* **Bài toán #1 (cân bằng pin giữa các tủ):** bản chất là dự báo nhu cầu + tối ưu điều chuyển (forecasting/optimization), không phải bài toán ngôn ngữ ──> LLM không phải công cụ phù hợp.

---

**Nguồn tham khảo về mô hình đổi pin:**
[VnEconomy — Mô hình đổi pin xe máy điện](https://vneconomy.vn/vinfast-trien-khai-toan-quoc-chinh-sach-uu-dai-chuyen-doi-xanh-va-mo-hinh-doi-pin-xe-may-dien.htm) ·
[VnReview — 4 mẫu xe máy điện đổi pin, 4.500 trạm](https://vnreview.vn/threads/vinfast-ra-mat-cung-luc-4-mau-xe-may-dien-moi-ho-tro-doi-pin-gia-doi-pin-khoang-gan-2-coc-tra-da-da-hoan-tat-4-500-tram-doi-pin.77440/latest) ·
[Mekong ASEAN — Tủ đổi pin tại cây xăng PVOIL](https://mekongasean.vn/v-green-trien-khai-hon-3000-tu-doi-pin-xe-may-dien-tai-cay-xang-pvoil-54965.html) ·
[The Leader — V-Green hợp tác EVNHANOI](https://theleader.vn/v-green-hop-tac-evnhanoi-trien-khai-10000-tu-doi-pin-xe-may-dien-d46517.html)
