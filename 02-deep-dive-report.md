# 02 — Deep-Dive Report: VinFast Battery Swap — Hỗ trợ từ xa sự cố tại tủ đổi pin không người trực

> **Học viên:** Ngô Gia Bảo — Branch: `ngogiabao`
> **Bài toán chọn từ:** Quick Card #2 trong [01-problem-scan.md](01-problem-scan.md)
> **Mảng kinh doanh:** 🚗 **VinFast / V-Green — Dịch vụ thuê & đổi pin xe máy điện (Battery Swap)**
>
> ⚠️ *Mọi con số (thời gian, số cuộc gọi, tỷ lệ) là **ước tính giả định** để scoping, chưa có dữ liệu vận hành thật. Phase 5 giải thích vì sao điều này ảnh hưởng tới quyết định.*

---

# 🏗️ Phase 3 — DEEP-DIVE

## 3.1. Current-State Workflow Mapping

**Bối cảnh:** Tủ đổi pin được đặt tại cây xăng, cửa hàng đối tác, trạm điện lực... và **không có nhân viên của dịch vụ đổi pin trực tại chỗ**. Khi khách gặp sự cố — tủ không nhận pin trả về, báo mã lỗi, kẹt khay, không còn pin đầy — kênh hỗ trợ duy nhất là **gọi tổng đài / chat trong app**. Tổng đài viên phải xử lý từ xa mà không nhìn thấy tủ.

| Bước | Ai thực hiện | Mô tả | Công cụ | ⏱ Thời gian | Ký hiệu |
|---|---|---|---|---:|---|
| 1 | Khách hàng | Gặp lỗi tại tủ, tự thử lại vài lần (rút pin, cắm lại, quét lại QR) | Tủ đổi pin, app | 3 phút | |
| 2 | Khách ──> Tổng đài | Gọi hotline / chat app, chờ kết nối | Tổng đài | *Chờ 2–5 phút* | 🔄 **Handoff:** Khách ──> Tổng đài |
| 3 | Tổng đài viên | Hỏi mã tủ, tài khoản, nghe khách mô tả sự cố (thường mơ hồ: *"tủ kêu bíp, đèn đỏ, không mở"*) | Điện thoại, CRM | 4 phút | 🔴 **Bottleneck** |
| 4 | Tổng đài viên | Tra mã lỗi trong tài liệu (PDF), xem log tủ và trạng thái pin trên hệ thống giám sát, đoán nguyên nhân | Tài liệu mã lỗi, dashboard IoT | 6 phút | 🔴 **Bottleneck** |
| 5 | Tổng đài viên ──> Khách | Hướng dẫn khách xử lý từng bước qua điện thoại; nếu không được thì thử cách khác (~40% lần hướng dẫn đầu không hiệu quả) | Điện thoại | 7 phút | 🔴 **Bottleneck** · 🔄 vòng lặp về bước 4 |
| 6 | Tổng đài viên ──> Kỹ thuật | ~25% ca không xử lý được từ xa: tạo ticket điều kỹ thuật viên, chỉ khách sang tủ khác gần nhất | Hệ thống ticket, bản đồ tủ | 3 phút | 🔄 **Handoff:** Tổng đài ──> Kỹ thuật viên *(chờ 30–90 phút)* |

**Tổng thời gian thao tác của tổng đài viên (bước 3–6) = 4 + 6 + 7 + 3 = ~20 phút/cuộc gọi.**
Phía khách: ~25–30 phút đứng tại tủ (gồm tự thử + chờ kết nối + được hướng dẫn), chưa tính 30–90 phút chờ kỹ thuật viên nếu bị chuyển tiếp.

**Vì sao bước 3–5 là bottleneck (~17/20 phút ≈ 85%):**
* Tổng đài viên **không nhìn thấy tủ**: phải dựa vào lời mô tả không chuẩn của khách, rồi tự "dịch" sang mã lỗi kỹ thuật.
* Thông tin nằm ở **3 nơi khác nhau** (lời khách, tài liệu mã lỗi PDF, log trên dashboard IoT), phải tự ghép lại khi khách đang chờ trên điện thoại.
* Kịch bản hướng dẫn mỗi người nói một kiểu ──> khách thao tác sai ──> lặp lại bước 4–5.

> 🖼️ Sơ đồ trực quan của workflow này: [04-workflow-diagram.png](04-workflow-diagram.png)

---

## 3.2. Problem Statement (6-field)

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | **Tổng đài viên hỗ trợ dịch vụ đổi pin** (người xử lý cuộc gọi sự cố hằng ngày). Liên quan: khách hàng thuê pin (đang đứng tại tủ), kỹ thuật viên vận hành (nhận ca chuyển tiếp), đối tác đặt tủ (cây xăng, cửa hàng — không chịu trách nhiệm hỗ trợ). |
| **2. Current Workflow** | Khách gặp lỗi tại tủ ──> gọi hotline/chat ──> tổng đài viên hỏi mã tủ + mô tả ──> tra mã lỗi trong PDF + xem log tủ trên dashboard IoT ──> hướng dẫn khách qua điện thoại ──> không được thì tạo ticket điều kỹ thuật viên và chỉ khách sang tủ khác. Công cụ: tổng đài, CRM, tài liệu mã lỗi, dashboard giám sát tủ, hệ thống ticket. |
| **3. Bottleneck** | Bước **hiểu mô tả mơ hồ của khách + tra mã lỗi/log + soạn hướng dẫn từng bước** (bước 3–5), chiếm ~17/20 phút mỗi cuộc gọi. Đây là phần **xử lý ngôn ngữ và tổng hợp thông tin**, cộng với áp lực khách đang đứng chờ. |
| **4. Business Impact** | *Giả định* khi mạng lưới mở rộng: ~300 cuộc gọi sự cố/ngày × 20 phút = **~100 giờ công/ngày (~12 tổng đài viên toàn thời gian)**. Khách đứng tại tủ 25–30 phút với xe hết pin — trải nghiệm tệ nhất của dịch vụ, dễ dẫn tới **bỏ gói thuê pin** và review xấu, đi ngược mục tiêu phổ cập đổi pin. ~25% ca phải điều kỹ thuật viên, tốn chi phí di chuyển cho những lỗi lẽ ra xử lý được từ xa. |
| **5. Success Metric** | (a) Thời gian tổng đài viên xử lý **~20 phút ──> dưới 8 phút/cuộc gọi**. (b) Tỷ lệ xử lý xong ngay cuộc gọi đầu **~45% ──> ≥ 65%**. (c) Tỷ lệ phải điều kỹ thuật viên **~25% ──> dưới 15%**. (d) **100%** ca có dấu hiệu nguy hiểm được chuyển kỹ thuật/cứu hộ; **0** hướng dẫn nằm ngoài danh mục thao tác đã duyệt. |
| **6. Operational Boundary** | ✅ **AI được phép:** đọc mô tả của khách + mã lỗi + log tủ + trạng thái pin; **đề xuất** nguyên nhân khả dĩ kèm độ tự tin; soạn kịch bản hướng dẫn **chỉ gồm các bước trong danh mục thao tác đã được kỹ thuật duyệt**; gợi ý tủ thay thế **lấy từ hệ thống** (không tự bịa); soạn nháp ticket cho kỹ thuật viên. Mọi đầu ra gắn thẻ `[DRAFT_ONLY]`. ❌ **AI tuyệt đối KHÔNG được:** hướng dẫn khách tháo vỏ, mở khoang hay can thiệp vào pin/tủ; tự gửi lệnh điều khiển tủ (mở khay, reset); hứa hoàn tiền/miễn phí; bịa vị trí tủ hoặc tình trạng pin; xử lý bất kỳ ca nào có dấu hiệu nguy hiểm. 🟢 **Điểm bắt buộc người duyệt:** tổng đài viên duyệt kịch bản trước khi đọc/gửi cho khách; mọi lệnh điều khiển tủ từ xa do tổng đài viên bấm; ca có **pin phồng, nóng, bốc khói, mùi khét, ngấm nước** ──> bỏ qua AI, dùng kịch bản an toàn cố định và điều kỹ thuật/cứu hộ ngay. |

---

## 3.3. Future-State Flow & AI Fit

### AI-Fit Matrix — So sánh 3 hướng tiếp cận

| Tiêu chí | Rule / State-Machine | LLM Feature | Agentic Loop |
|---|---|---|---|
| **Lấy log tủ, mã lỗi, trạng thái pin theo mã tủ** | ✅ Tốt — gọi API cố định | — | Không cần |
| **Mã lỗi chuẩn ──> thao tác xử lý chuẩn** | ✅ Tốt — bảng tra, rẻ, giải thích được | Không cần | Không cần |
| **Phát hiện ca nguy hiểm (từ khóa + cảm biến nhiệt)** | ✅ Bắt buộc dùng rule — phải chắc chắn, không xác suất | Hỗ trợ bắt cách nói lạ | — |
| **Hiểu mô tả tự do ("kêu bíp", "đèn nháy đỏ") khi không có / không khớp mã lỗi; soạn hướng dẫn dễ hiểu theo tình huống** | ❌ Không làm được — kịch bản cứng không bao hết cách khách mô tả | ✅ Thế mạnh của LLM | Có thể, nhưng thừa |
| **Tự gửi lệnh mở khay / reset tủ** | — | — | ⚠️ Làm được nhưng **rủi ro cao**: tác động vật lý lên thiết bị ngoài hiện trường |
| **Kết luận** | Dùng cho **dữ liệu có cấu trúc + cổng an toàn** | ✅ **Chọn cho phần hiểu ngôn ngữ & soạn hướng dẫn** | ❌ Không chọn |

**Quyết định kiến trúc:** [x] **Rule + LLM Feature (hybrid)** · [ ] Agentic Loop.
Rule-based lo phần dữ liệu có cấu trúc và cổng an toàn; LLM chỉ lo phần ngôn ngữ. Không dùng Agent vì không có lý do để AI tự điều khiển thiết bị vật lý không người trực — một lệnh sai có thể làm kẹt pin của khách hoặc gây mất an toàn.

### Future-State Flow

| Bước | Loại | Mô tả | ⏱ Ước tính |
|---|---|---|---:|
| 1 | — | Khách gặp lỗi, bấm "Báo sự cố" trong app tại tủ (app tự gửi mã tủ qua QR/GPS) hoặc gọi hotline | 1–2 phút |
| 2 | ⚙️ Rule | Hệ thống tự lấy: log tủ 30 phút gần nhất, mã lỗi, trạng thái khay và pin, lịch sử đổi pin của khách, danh sách tủ gần nhất còn pin đầy | tự động |
| 3 | ⚙️ Rule | **Cổng an toàn:** mô tả có từ khóa nguy hiểm (phồng, nóng, khói, khét, nước) hoặc cảm biến nhiệt tủ bất thường ──> kịch bản an toàn cố định + điều kỹ thuật/cứu hộ, **dừng luồng AI** | tự động |
| 4 | 🔵 **AI Step** | LLM đọc mô tả + dữ liệu bước 2, trả về JSON: nguyên nhân khả dĩ, độ tự tin, các bước hướng dẫn (chỉ chọn mã thao tác từ danh mục đã duyệt), có cần kỹ thuật viên không, tin nháp cho khách bắt đầu bằng `[DRAFT_ONLY]` | ~5 giây |
| 5 | ⚙️ Rule | Kiểm tra output: JSON hợp lệ, có `[DRAFT_ONLY]`, mọi bước hướng dẫn nằm trong danh mục, tủ thay thế khớp dữ liệu hệ thống | tự động |
| 6 | 🟢 **Human Step (HITL)** | Tổng đài viên xem tóm tắt + kịch bản đề xuất, chỉnh nếu cần, đọc/gửi cho khách; tự bấm lệnh điều khiển tủ nếu kịch bản yêu cầu; quyết định điều kỹ thuật viên | 4–6 phút |

**↩️ Fallback (khi AI lỗi hoặc không chắc chắn):**
* LLM trả về JSON lỗi, thiếu `[DRAFT_ONLY]`, hoặc có bước hướng dẫn **ngoài danh mục** ──> loại bỏ đề xuất, tổng đài viên dùng **bảng tra mã lỗi rule-based** như hiện tại (vẫn tiết kiệm thời gian nhờ dữ liệu đã được gom sẵn ở bước 2).
* Độ tự tin thấp (< 0.7) hoặc mô tả không khớp log ──> AI chỉ đề xuất **câu hỏi làm rõ** cho tổng đài viên hỏi khách, không đề xuất cách xử lý.
* API LLM ngừng hoạt động hoặc quá chậm ──> hiển thị ngay kịch bản chuẩn theo mã lỗi; khách được chỉ sang tủ gần nhất còn pin đầy.
* Trong mọi trường hợp: **AI không có đường nào để tự gửi lệnh cho tủ hay tự gửi tin cho khách.**

**Thời gian xử lý tương lai ước tính:** ~4–6 phút/cuộc gọi phía tổng đài viên (giảm ~70% so với 20 phút); khách được hướng dẫn trong vài phút thay vì 25–30 phút.

**Hướng phát triển sau này (ngoài phạm vi hiện tại):** khi đã đo được độ chính xác đủ cao, cho phép trợ lý trả lời **trực tiếp cho khách trong app** với các lỗi đơn giản, an toàn — vẫn giữ cổng an toàn và chuyển người khi cần.

---

# 🏁 Phase 5 — EVALUATE

### AI Readiness Checklist

| # | Câu hỏi | Trạng thái | Bằng chứng / Ghi chú |
|---|---|---|---|
| 1 | Có sẵn dữ liệu mẫu/logs sạch để test? | [ ] **Một phần** | Tủ là thiết bị IoT nên **log, mã lỗi, trạng thái khay/pin đã có sẵn**. Nhưng **lịch sử cuộc gọi chưa được ghi lại có cấu trúc** (khách mô tả gì, nguyên nhân thật, cách xử lý nào hiệu quả), và tài liệu mã lỗi chưa được chuẩn hóa thành **danh mục thao tác an toàn** ──> chưa có bộ dữ liệu để đo độ chính xác. |
| 2 | Rủi ro khi AI sai có nằm trong tầm kiểm soát (HITL / Fallback)? | [x] **Có** | Cổng an toàn rule-based chặn ca nguy hiểm trước khi tới AI; AI chỉ chọn bước từ danh mục đã duyệt; tổng đài viên duyệt mọi kịch bản và tự bấm mọi lệnh điều khiển tủ; fallback về bảng tra mã lỗi. |
| 3 | Stakeholders sẵn sàng thay đổi quy trình? | [ ] **Một phần** | Tổng đài có động lực (giảm tải). Cần **đội kỹ thuật** duyệt danh mục thao tác an toàn, và cần làm rõ phân chia trách nhiệm giữa VinFast, V-Green và các đối tác đặt tủ khi có sự cố. |

### Quyết định cuối cùng của Ban Giám Đốc Vin Smart Future

[ ] **GO** · [x] **NOT YET (Cần tích lũy thêm dữ liệu / xác lập baseline)** · [ ] **NO-GO**

### Justification

> **Bài toán có thật và gắn với chính mô hình tủ không người trực, nhưng chưa đủ bằng chứng để build phần LLM.**
>
> 1. **Phù hợp AI là có thật:** tổng đài viên phải "dịch" lời mô tả không chuẩn của khách sang mã lỗi và soạn hướng dẫn ngay khi khách đang chờ — rule-based không bao hết được, LLM làm tốt. Rủi ro kiểm soát được nhờ cổng an toàn, danh mục thao tác và người duyệt. Vì vậy đây **không phải NO-GO**.
> 2. **Nhưng chưa thể GO** vì **khối lượng thật chưa rõ**. Một thống kê cũ của VinFast cho thấy giai đoạn đầu cả nước chỉ có vài chục lượt đổi pin mỗi tháng; mạng lưới đang tăng nhanh, nhưng con số "300 cuộc gọi sự cố/ngày" trong báo cáo hoàn toàn là giả định. Nếu số cuộc gọi thật thấp, **một bảng tra mã lỗi rule-based là đủ** và LLM không đáng chi phí. Ngoài ra chưa có lịch sử cuộc gọi để đo độ chính xác, và chưa có danh mục thao tác an toàn do kỹ thuật duyệt — thứ bắt buộc phải có trước khi cho AI đề xuất hướng dẫn.
> 3. **Chi phí chờ thấp và việc chuẩn bị có giá trị ngay:** chuẩn hóa mã lỗi và danh mục thao tác giúp tổng đài viên ngay cả khi không có AI.
>
> **Điều kiện để chuyển sang GO (dự kiến sau 6–8 tuần):**
> * Đo baseline 4 tuần: số cuộc gọi sự cố/ngày, thời gian xử lý, tỷ lệ xử lý xong ngay cuộc gọi đầu, tỷ lệ phải điều kỹ thuật viên.
> * **Ngưỡng quyết định:** nếu < ~50 cuộc gọi sự cố/ngày ──> chỉ làm bảng tra mã lỗi rule-based (**NO-GO cho LLM**); nếu cao hơn ──> **GO** cho LLM.
> * Ghi lại có cấu trúc ≥ 300 cuộc gọi (mô tả, mã lỗi, nguyên nhân thật, cách xử lý) làm bộ test.
> * Đội kỹ thuật ban hành **danh mục thao tác an toàn** cho khách và danh sách dấu hiệu nguy hiểm.
> * Pilot đầu tiên chạy **chế độ shadow** (AI đề xuất song song, tổng đài viên vẫn xử lý như cũ, chỉ so sánh) tại 1 thành phố.

---

**Nguồn tham khảo về mô hình đổi pin:**
[VnEconomy — Mô hình đổi pin xe máy điện](https://vneconomy.vn/vinfast-trien-khai-toan-quoc-chinh-sach-uu-dai-chuyen-doi-xanh-va-mo-hinh-doi-pin-xe-may-dien.htm) ·
[VnReview — 4 mẫu xe máy điện đổi pin, 4.500 trạm](https://vnreview.vn/threads/vinfast-ra-mat-cung-luc-4-mau-xe-may-dien-moi-ho-tro-doi-pin-gia-doi-pin-khoang-gan-2-coc-tra-da-da-hoan-tat-4-500-tram-doi-pin.77440/latest) ·
[Mekong ASEAN — Tủ đổi pin tại cây xăng PVOIL](https://mekongasean.vn/v-green-trien-khai-hon-3000-tu-doi-pin-xe-may-dien-tai-cay-xang-pvoil-54965.html)
