01 — Problem Scan & Quick Cards

Học viên: Nguyễn Quang Trung — Branch: nguyenquangtrung-03389 Mảng quan tâm: Vinhomes, Vinfast và Vinpearl

⚠️ Các con số thời gian/tỷ lệ trong tài liệu là ước tính giả định phục vụ scoping, chưa được kiểm chứng bằng dữ liệu vận hành thật. Cần xác lập baseline trước khi build.

# 🔍 Phase 1 — SCAN

### 📝 List bài toán của tôi:
| # | Subsidiary (VinFast/Xanh SM...) | Lens | Mô tả ngắn bài toán |
|---|----------------------------------|------|---------------------|
| 1 |Vinhomes |Lặp lại | Phân loại tự động các khiếu nại (ví dụ: mất nước, hỏng đèn, tiếng ồn) gửi qua App Vinhomes Resident đến đúng ban quản lý hoặc đội kỹ thuật của từng tòa nhà.|
| 2 |Vinfast |Tốn thời gian/Lặp lại|Tự động phân loại và điều phối cứu hộ xe điện (EV Roadside Assistance): AI phân tích mô tả sự cố để xác định nhanh cần điều xe sạc lưu động hay xe cứu hộ và tự động đề xuất phương án điều phối tối ưu.|
| 3 |Vinmec|Tốn thời gian|Phân loại tự động và sàng lọc triệu chứng đặt lịch khám bệnh: AI tự động phân tích triệu chứng của bệnh nhân để gắn nhãn mức độ ưu tiên lâm sàng và gợi ý chuyên khoa phù hợp nhằm hỗ trợ đội ngũ y tế ra quyết định phân luồng.|
| 4 |Vinpearl|Lặp lại/tốn thời gian|Trợ lý ảo thông minh cá nhân hóa lịch trình nghỉ dưỡng và dịch vụ Concierge: AI co-pilot giúp tổng hợp sở thích và lịch sử của khách, tự động soạn thảo lịch trình chi tiết và đề xuất các gói dịch vụ gia tăng phù hợp cho nhân viên CSKH duyệt.|
| 5 |Vinschool|Lặp lại|Trợ lý trả lời tự động câu hỏi thủ tục học vụ cho phụ huynh và học sinh: Hệ thống AI kết hợp kho dữ liệu quy chế nội bộ của trường để tự động soạn thảo câu trả lời chính xác, trích dẫn đúng số điều khoản liên quan.|

---

# 🃏 Phase 2 — QUICK-ASSESS

┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #Phân loại & Điều hướng phản ánh Vinhomes│
│                                                             │
│ Bài toán (1 câu): Phân loại phản ánh cư dân Vinhomes        │
│ Công ty thành viên: [ ] VinFast  [ ] Xanh SM  [X] Vinhomes  │
│                     [ ] Vinmec   [ ] Khác (Ghi rõ)________  │
│                                                             │
│ Ai đang đau (Actor)? Nhân viên CSKH / Ban Quản Lý           │
│                                                             │
│ Workflow thủ công hiện tại (3-5 bước):                      │
│   1. Cư dân gửi phản ánh                                    |
|   2. nhân viên mở ra đọc thủ công để phân loại              |
|   3. Xác định phản ánh và tìm tòa nhà/ số phòng             |
|   4. chuyển sang cho thợ / người thực thi                   │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất?                            |
|   => bước 2 và 3 (⏱ 10-15p phút/lượt)                      |
│ AI có thể nhảy vào hỗ trợ ở bước nào?                       |
| Bước 2 và 3 (AI tự phân loại và gửi đến người nhận)         │
│                                                             │
│ Đo thành công bằng gì (Metric có số)?                       |
|   => Giảm thời gian phân loại phản ánh từ 10 phút xuống < 30 giây│
│ và đạt độ chính xác phân loại nhóm lỗi >= 90%.              │
│                                                             │
│                                                             │
│ Quick Architecture: [ ] No AI  [X] Rule  [X] LLM  [ ] Agent │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #Phân loại & Điều phối cứu hộ VinFast    │
│                                                             │
│ Bài toán (1 câu): Phân loại tự động & điều phối cứu hộ xe   │
│ điện khẩn cấp (VinFast EV Roadside Assistance).             │
│ Công ty thành viên: [X] VinFast  [ ] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [ ] Khác (Ghi rõ)________  │
│                                                             │
│ Ai đang đau (Actor)? Đội ngũ tổng đài kỹ thuật cứu hộ       │
│                                                             │
│ Workflow thủ công hiện tại (3-5 bước):                      │
│   1. Tiếp nhận tin nhắn/cuộc gọi sự cố của tài xế ──>       │
│   2. Nhân viên phỏng vấn lại & đọc thủ công mô tả ──>       │
│   3. Tra cứu vị trí GPS & kiểm tra trạng thái pin/lỗi ──>    │
│   4. Quyết định điều xe sạc lưu động hoặc xe kéo về xưởng   │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 và 3 (⏱ 12-15p/lượt) │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2 và 3 (AI trích  │
│ xuất lỗi/vị trí & đề xuất phương án điều phối phù hợp)     │
│                                                             │
│ Đo thành công bằng gì (Metric có số)? Giảm thời gian xác    │
│ định phương án cứu hộ từ 15 phút xuống < 1 phút, đạt độ     │
│ chính xác định tuyến phương án cứu hộ >= 95%.               │
│                                                             │
│ Quick Architecture: [ ] No AI  [ ] Rule  [X] LLM  [ ] Agent │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│ QUICK PROBLEM CARD #Cá nhân hóa lịch trình Vinpearl Concierge│
│                                                             │
│ Bài toán (1 câu): Soạn thảo tự động lịch trình nghỉ dưỡng &  │
│ gợi ý dịch vụ cho khách lưu trú Vinpearl.                   │
│ Công ty thành viên: [ ] VinFast  [ ] Xanh SM  [ ] Vinhomes  │
│                     [ ] Vinmec   [X] Khác: Vinpearl         │
│                                                             │
│ Ai đang đau (Actor)? Nhân viên Chăm sóc khách hàng (Concierge)│
│                                                             │
│ Workflow thủ công hiện tại (3-5 bước):                      │
│   1. Tiếp nhận yêu cầu đặc biệt của khách qua app/email ──>  │
│   2. Đọc thủ công và tra cứu lịch sử sở thích của khách ──> │
│   3. Soạn thảo lịch trình tư vấn (F&B, Spa, Tour) thủ công ──>│
│   4. Chuyển thông tin task tới các bộ phận dịch vụ nội bộ   │
│                                                             │
│ Bước nào tốn thời gian/lỗi nhất? Bước 2 và 3 (⏱ 20-30p/lượt) │
│ AI có thể nhảy vào hỗ trợ ở bước nào? Bước 2 và 3 (AI tự    │
│ tổng hợp sở thích & tạo bản nháp lịch trình cá nhân hóa)    │
│                                                             │
│ Đo thành công bằng gì (Metric có số)? Giảm thời gian soạn    │
│ đề xuất lịch trình từ 20 phút xuống dưới 2 phút; tăng tỷ lệ │
│ khách chấp nhận dịch vụ gia tăng thêm 25%.                  │
│                                                             │
│ Quick Architecture: [ ] No AI  [ ] Rule  [X] LLM  [ ] Agent │
└─────────────────────────────────────────────────────────────┘
--

### 🧪 Stress-test các thẻ (vai CFO / Trưởng phòng Vận hành)

| Card | Điểm yếu bị phản biện | Phản hồi / điều chỉnh |
| --- | --- | --- |
| **#1 Phân loại khiếu nại Vinhomes** | Phản ánh của cư dân thường theo mẫu sẵn trên App hoặc ngắn gọn (mất nước, hỏng đèn), một bảng keyword-matching hoặc rule-based đơn giản có thể xử lý nhanh, rẻ và minh bạch hơn LLM. | Đồng ý: ưu tiên dùng rule-based cho các lỗi cơ bản, cấu trúc rõ ràng. Chỉ cân nhắc LLM khi cư dân nhập văn bản dài, miêu tả phức tạp, cảm xúc hoặc đa ý trong một câu. |
| **#2 Hỗ trợ cứu hộ xe điện VinFast** | (a) Nếu số lượng sự cố thấp, bảng tra cứu mã lỗi là đủ. (b) AI hướng dẫn sai có thể khiến khách tự can thiệp kỹ thuật vào pin/xe điện ──> rủi ro an toàn cực lớn. | (a) Đo lường thực tế tần suất cuộc gọi; đặt ngưỡng tự động hóa. (b) AI chỉ được phép trích xuất bước từ danh mục thao tác chuẩn đã duyệt, có cổng an toàn (safety guardrails) chặn mọi ca nguy hiểm để chuyển thẳng cho kỹ thuật viên. |
| **#3 Cá nhân hóa lịch trình Vinpearl** | LLM có thể "bịa" dịch vụ, khung giờ hoặc hiểu sai yêu cầu đặc biệt về ăn kiêng/dị ứng của khách VIP. | Dữ liệu dịch vụ, giá phòng và lịch trình luôn lấy từ hệ thống gốc (PMS/CRM), LLM chỉ đóng vai trò tổng hợp và viết văn bản; mọi yêu cầu đặc biệt bắt buộc nhân viên CSKH/Concierge duyệt trước khi gửi khách. |

---

### 🗳️ Đề xuất cho Deep-Dive

Đề xuất mang **Card #1 — Vinhomes: Phân loại tự động và điều hướng phản ánh cư dân** vào Deep-Dive nhóm:

* **Nỗi đau sinh ra từ chính mô hình vận hành:** Hàng trăm khiếu nại đổ về mỗi ngày khiến nhân viên CSKH mất nhiều thời gian đọc thủ công và tra cứu sơ đồ tòa nhà để phân loại, tạo điểm nghẽn lớn (10-15 phút/lượt).
* **Phù hợp LLM thật sự:** Phản ánh của cư dân thường dùng ngôn ngữ tự nhiên, không theo chuẩn, có thể lộn xộn nhiều vấn đề ──> đúng thế mạnh của LLM trong việc phân tích ngữ nghĩa và trích xuất ý định.
* **Ranh giới vận hành rõ ràng:** AI chỉ đề xuất phân loại và soạn thảo bản nháp hướng dẫn (bắt buộc gắn thẻ `[DRAFT_ONLY]`), nhân viên duyệt trước khi gửi, và các ca khẩn cấp (cháy, rò rỉ khí ga) có cơ chế JSON command ghi đè để chuyển thẳng đội ứng phó.

**Lý do không chọn các thẻ khác:**

* **Card #2 (VinFast):** Rủi ro an toàn vật lý cao (liên quan đến điện áp cao và phương tiện di chuyển), việc kiểm thử an toàn thực tế trong khuôn khổ lab phức tạp hơn.
* **Card #3 (Vinpearl):** Phụ thuộc sâu vào hệ thống dữ liệu đặt phòng (PMS) và lịch sử khách hàng độc quyền, khó mô phỏng đầy đủ workflow thực tế trong phạm vi đồ án học phần.


# 🏗️ Phase 3 — DEEP-DIVE

## 3.1. Current-State Workflow Mapping 

* 🔴 **Bottleneck:** Bước gây tắc nghẽn, tốn thời gian, hoặc sai sót nhiều nhất.
* 🔄 **Handoff:** Điểm chuyển giao thông tin giữa người và hệ thống, hoặc giữa các bộ phận.
* Ghi rõ thời gian vận hành trung bình: **Tổng cộng = ____ phút/lượt**.

   - Bước 1: Cư dân nhập nội dung phàn nàn trên app (⏱ 1 phút).
   - Bước 2 (🔴 Bottleneck): Hàng trăm tin nhắn đổ về, nhân viên CSKH đọc thủ công từng dòng để lọc xem đây là lỗi điện nước, an ninh hay vệ sinh (⏱ 8 phút).
   - Bước 3 (🔴 Bottleneck): Tra cứu thủ công xem căn hộ đó thuộc tòa nào, ai là kỹ sư phụ trách ca trực (⏱ 4 phút).
   - Bước 4: Gửi thông tin sang bộ phận liên quan (⏱ 2 phút).
   => Tổng thời gian xử lý thủ công: Khoảng 15 phút cho một yêu cầu ban đầu.



## 3.2. Problem Statement (6-field) & Metrics
Điền đầy đủ 6 trường thông tin của bài toán:

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** |Nhân viên CSKH & Ban quản lý cụm tòa nhà Vinhomes
| **2. Current Workflow** |Xử lý thủ công qua app và phân loại đến cho đối tượng thực thi
| **3. Bottleneck** |Khâu đọ hiểu và gán nhãn phân loại bị quá tải dễ dẫn đến việc nhầm bộ phận, làm chậm thời gian xử lý
| **4. Business Impact**  |Tăng thời gian chờ đợi của ư dân, tốn nhân lực trực app đọc tin nhắn rác hoặc không rõ ràng
| **5. Success Metric** |Tỉ lệ phân loại đúng bộ phận chuyên trách đạt trên 90%. Thời gian từ lúc cư dân gửi đến khi chuyển đúng đội kỹ thuật rút gọn dưới 1 phút
| **6. Operational Boundary** |AI được phép đọc hiểu văn bản, gán nhãn danh mục (điện/ nước/ an ninh/ khác) và soạn sẵn bản điều phối dạng draft. Cấm AI không được tự ý gửi tin nhắn phản hòi trực tiếp cho cư dân hoặc tự ý đóng ticket khi chưa có sự kiểm duyệt của nhân viên

## 3.3. Future-State Flow & AI Fit
  * 🔵 **AI Step:** Tác vụ LLM xử lý.
  * 🟢 **Human Step (HITL):** Bước con người phê duyệt/review (Human-in-the-loop).
  * ↩️ **Fallback:** Kế hoạch dự phòng khi LLM trả về kết quả lỗi hoặc không tự tin.

   Luồng tương lai: 
   1. Cư dân gửi phản ánh  
   2. [🔵 AI Step] AI tự động đọc, phân loại nhóm lỗi và gán đúng phòng ban 
   3. [🟢 HITL Step] Nhân viên CSKH nhìn màn hình, bấm nút "Phê duyệt" (Approve) 
   4. Hệ thống tự động đẩy việc cho đội kỹ thuật. Nếu AI phân loại sai, nhân viên bấm sửa tay (Fallback).

