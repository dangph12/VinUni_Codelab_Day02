# 02 — Deep-Dive Report: VinFast Battery Swap — Xử lý tranh chấp pin trả về bị hư hỏng

> **Học viên:** Ngô Gia Bảo — Branch: `ngogiabao`
> **Bài toán chọn từ:** Quick Card #2 trong [01-problem-scan.md](01-problem-scan.md)
> **Mảng kinh doanh:** 🚗 **VinFast — Dịch vụ thuê & đổi pin xe máy điện (Battery Swap)**
>
> ⚠️ *Mọi con số (thời gian, số ca, tỷ lệ) là **ước tính giả định** để scoping, chưa có dữ liệu vận hành thật. Phase 5 giải thích vì sao điều này ảnh hưởng tới quyết định.*

---

# 🏗️ Phase 3 — DEEP-DIVE

## 3.1. Current-State Workflow Mapping

**Bối cảnh:** Khi khách trả pin tại trạm, nếu pin có dấu hiệu bất thường (vỏ móp/nứt, dấu hiệu vào nước, SoH tụt mạnh so với lần nhận, mã lỗi BMS), trạm phải mở ticket. CSKH xử lý ticket để xác định **ai chịu chi phí** (khách hàng gây hư hỏng / hao mòn tự nhiên / lỗi nhà sản xuất) và phản hồi khách.

| Bước | Ai thực hiện | Mô tả | Công cụ | ⏱ Thời gian xử lý | Ký hiệu |
|---|---|---|---|---:|---|
| 1 | Hệ thống trạm / Nhân viên trạm | Phát hiện pin bất thường khi khách trả (BMS báo lỗi hoặc nhân viên thấy hư hỏng ngoại quan) | Tủ đổi pin, BMS | 2 phút | |
| 2 | Nhân viên trạm | Chụp ảnh pin, ghi chú tình trạng, tạo ticket | App nội bộ trạm | 5 phút | 🔄 **Handoff:** Trạm ──> CSKH |
| — | *(Hàng đợi)* | Ticket nằm chờ trong queue CSKH | Hệ thống ticket | *Chờ 4–24 giờ* | 🔴 *(thời gian chờ)* |
| 3 | CSKH | Tra lịch sử thuê/đổi pin của khách (gói thuê, các lần đổi gần nhất, ai giữ pin) | Hệ thống quản lý thuê pin | 8 phút | |
| 4 | CSKH (+ Kỹ thuật nếu cần) | Đối chiếu ảnh tại trạm với log BMS (va đập, nhiệt độ, độ ẩm, dòng sạc bất thường) để dựng lại diễn biến | Ảnh + export log BMS (Excel) | 12 phút | 🔴 **Bottleneck** · 🔄 **Handoff:** CSKH ──> Kỹ thuật (khi log khó đọc) |
| 5 | CSKH | Tra điều khoản gói thuê, xác định trường hợp: khách chịu phí / miễn phí / cần kỹ thuật kết luận | Tài liệu điều khoản (PDF) | 7 phút | 🔴 **Bottleneck** |
| 6 | CSKH | Soạn phản hồi giải thích cho khách (bằng chứng + điều khoản + mức phí nếu có) và gửi | Email / App / Zalo | 8 phút | 🔴 **Bottleneck** · 🔄 **Handoff:** CSKH ──> Khách |
| 7 | Khách hàng ──> CSKH | ~30% khách không đồng ý, khiếu nại lại ──> lặp lại bước 4–6 | — | (+~20 phút/lần lặp) | 🔄 Vòng lặp |

**Tổng cộng thời gian xử lý (bước 1–6) = 2 + 5 + 8 + 12 + 7 + 8 = ~42 phút/lượt**, chưa tính thời gian chờ trong hàng đợi (4–24 giờ) và vòng khiếu nại lại.

**Vì sao bước 4–6 là bottleneck:**
* Thông tin nằm rải rác ở **4 nguồn khác nhau** (lịch sử thuê, ảnh, log BMS, điều khoản) — CSKH phải tự ghép lại.
* Log BMS là dữ liệu kỹ thuật, CSKH không chuyên đọc chậm và dễ hiểu sai ──> phải chuyển kỹ thuật.
* Văn bản phản hồi mỗi người viết một kiểu, thiếu bằng chứng cụ thể ──> khách không thuyết phục ──> khiếu nại lại.

> 🖼️ Sơ đồ trực quan của workflow này: [04-workflow-diagram.png](04-workflow-diagram.png)

---

## 3.2. Problem Statement (6-field)

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | **Nhân viên CSKH dịch vụ thuê pin VinFast** (người xử lý ticket hằng ngày). Liên quan: nhân viên trạm đổi pin (tạo ticket), kỹ thuật viên pin (hỗ trợ đọc log), khách hàng thuê pin (người nhận kết quả). |
| **2. Current Workflow** | Trạm phát hiện pin bất thường ──> chụp ảnh, tạo ticket ──> CSKH tra lịch sử thuê ──> đối chiếu ảnh + log BMS (đôi khi nhờ kỹ thuật) ──> tra điều khoản gói thuê ──> soạn và gửi phản hồi thủ công. Công cụ: app trạm, hệ thống ticket, hệ thống quản lý thuê pin, file export log BMS, PDF điều khoản, email/app. |
| **3. Bottleneck** | Bước **tổng hợp bằng chứng từ 4 nguồn + đọc log BMS + soạn văn bản phản hồi** (bước 4–6), chiếm ~27/42 phút mỗi ca. Đây là phần **xử lý ngôn ngữ và tổng hợp thông tin phi cấu trúc**, không phải tính toán. |
| **4. Business Impact** | *Giả định* ~60 ca tranh chấp/ngày toàn hệ thống × 42 phút = **~42 giờ công/ngày (~5 nhân sự toàn thời gian)**. Thời gian chờ 4–24 giờ khiến khách bức xúc; ~30% ca bị khiếu nại lại; mỗi tranh chấp xử lý kém là nguy cơ **mất khách thuê pin dài hạn** và review xấu cho dịch vụ đổi pin. |
| **5. Success Metric** | (a) Thời gian xử lý trung bình **42 phút ──> dưới 10 phút/ca**. (b) **≥ 85%** bản nháp AI được CSKH duyệt chỉ với chỉnh sửa nhỏ. (c) Tỷ lệ khiếu nại lại **~30% ──> dưới 15%**. (d) **0 ca** AI tự gửi cho khách hoặc tự áp phí mà không qua người duyệt. |
| **6. Operational Boundary** | ✅ **AI được phép:** tóm tắt lịch sử thuê và log BMS thành timeline; trích dẫn điều khoản liên quan; **đề xuất** phân loại trường hợp kèm độ tự tin; soạn **nháp** phản hồi gắn thẻ `[DRAFT_ONLY]`. ❌ **AI tuyệt đối KHÔNG được:** tự gửi tin cho khách; tự kết luận/áp phí; đưa ra số tiền không có trong bảng giá chính thức; hứa bồi thường/miễn phí; suy đoán khi thiếu bằng chứng; tiết lộ dữ liệu của khách khác. 🟢 **Điểm bắt buộc người duyệt:** mọi phản hồi trước khi gửi; mọi quyết định thu phí; ca phí vượt ngưỡng (ví dụ > 2 triệu VNĐ) hoặc nghi gian lận ──> trưởng nhóm CSKH duyệt; ca có dấu hiệu an toàn (phồng pin, cháy nổ) ──> kỹ thuật xử lý, không qua AI. |

---

## 3.3. Future-State Flow & AI Fit

### AI-Fit Matrix — So sánh 3 hướng tiếp cận

| Tiêu chí | Rule / State-Machine | LLM Feature | Agentic Loop |
|---|---|---|---|
| **Gom dữ liệu từ 4 nguồn** | ✅ Tốt — gọi API cố định, xác định | — | Không cần, nguồn dữ liệu đã biết trước |
| **Phân loại theo ngưỡng số (SoH, mã lỗi, nhiệt độ)** | ✅ Tốt — rẻ, giải thích được | Không cần | Không cần |
| **Đọc ghi chú tự do của trạm, ghép timeline, viết phản hồi dễ hiểu** | ❌ Không làm được — template cứng bị khách chê rập khuôn | ✅ Thế mạnh của LLM | Có thể, nhưng thừa |
| **Rủi ro / chi phí** | Thấp nhất | Trung bình — kiểm soát được bằng HITL + `[DRAFT_ONLY]` | Cao — tự gọi tool, khó kiểm soát, dễ vượt ranh giới tài chính |
| **Kết luận** | Dùng cho **phần dữ liệu có cấu trúc** | ✅ **Chọn cho phần tổng hợp & soạn thảo** | ❌ Không chọn |

**Quyết định kiến trúc:** [x] **Rule + LLM Feature (hybrid)** · [ ] Agentic Loop.
Rule-based lo phần lấy dữ liệu và kiểm tra ngưỡng; LLM chỉ lo phần ngôn ngữ (tóm tắt, trích điều khoản, soạn nháp). Không dùng Agent vì luồng xử lý cố định, không cần AI tự lập kế hoạch, và việc để AI tự hành động trong một quy trình liên quan tới tiền là rủi ro không cần thiết.

### Future-State Flow

| Bước | Loại | Mô tả | ⏱ Ước tính |
|---|---|---|---:|
| 1 | ⚙️ Rule | Trạm phát hiện pin bất thường, chụp ảnh, tạo ticket (giữ nguyên) | 5 phút |
| 2 | ⚙️ Rule | Hệ thống tự gom: lịch sử thuê/đổi pin, log BMS, ảnh, điều khoản gói thuê của khách ──> 1 hồ sơ | tự động |
| 3 | ⚙️ Rule | **Cổng an toàn:** nếu log có dấu hiệu phồng/quá nhiệt nghiêm trọng ──> chuyển thẳng Kỹ thuật, dừng luồng AI | tự động |
| 4 | 🔵 **AI Step** | LLM tóm tắt timeline bằng chứng, trích điều khoản liên quan, đề xuất loại trường hợp (`khach_gay_hu_hong` / `hao_mon_tu_nhien` / `loi_san_xuat` / `thieu_bang_chung`) kèm độ tự tin, và soạn nháp phản hồi bắt đầu bằng `[DRAFT_ONLY]` — trả về JSON có cấu trúc | ~10 giây |
| 5 | ⚙️ Rule | Kiểm tra output: JSON hợp lệ, có `[DRAFT_ONLY]`, mức phí (nếu có) khớp bảng giá chính thức | tự động |
| 6 | 🟢 **Human Step (HITL)** | CSKH đọc tóm tắt + nháp, chỉnh sửa, **quyết định phí**, bấm gửi. Phí > 2 triệu VNĐ hoặc nghi gian lận ──> trưởng nhóm duyệt | 5–8 phút |
| 7 | — | Khách nhận phản hồi có bằng chứng rõ ràng | — |

**↩️ Fallback (khi AI lỗi hoặc không chắc chắn):**
* LLM trả về JSON lỗi, thiếu `[DRAFT_ONLY]`, hoặc mức phí không khớp bảng giá ──> loại bỏ bản nháp, ticket quay về **quy trình thủ công** hiện tại, kèm hồ sơ dữ liệu đã gom ở bước 2 (vẫn tiết kiệm thời gian tra cứu).
* Độ tự tin thấp (< 0.7) hoặc loại `thieu_bang_chung` ──> không soạn kết luận, chỉ soạn **tin nhắn xin thêm thông tin** từ khách/trạm.
* API LLM ngừng hoạt động ──> CSKH dùng **mẫu phản hồi chuẩn** theo từng loại trường hợp.
* Trong mọi trường hợp: **không có đường nào để hệ thống tự gửi tin hoặc tự trừ tiền khách.**

**Thời gian xử lý tương lai ước tính:** ~5 phút (trạm) + ~5–8 phút (CSKH duyệt) ≈ **10–13 phút/ca** phía con người, giảm ~70% so với 42 phút hiện tại.

---

# 🏁 Phase 5 — EVALUATE

### AI Readiness Checklist

| # | Câu hỏi | Trạng thái | Bằng chứng / Ghi chú |
|---|---|---|---|
| 1 | Có sẵn dữ liệu mẫu/logs sạch để test? | [ ] **Chưa đủ** | Log BMS và lịch sử thuê có sẵn trong hệ thống, nhưng **ticket tranh chấp cũ chưa được chuẩn hóa** (kết luận cuối cùng, mức phí, khách có khiếu nại lại không) ──> chưa có bộ dữ liệu đối chiếu để đo độ chính xác, và chưa có baseline thật cho metric "42 phút" và "30% khiếu nại lại". |
| 2 | Rủi ro khi AI sai có nằm trong tầm kiểm soát (HITL / Fallback)? | [x] **Có** | AI chỉ soạn nháp `[DRAFT_ONLY]`, người duyệt mọi quyết định phí, có kiểm tra rule-based sau LLM và fallback về quy trình thủ công. Ca an toàn pin bị tách khỏi luồng AI. |
| 3 | Stakeholders sẵn sàng thay đổi quy trình? | [ ] **Một phần** | CSKH có động lực (giảm tải), nhưng cần **bộ phận Pháp chế** duyệt việc dùng AI trích dẫn điều khoản và **Kỹ thuật pin** xác nhận cách diễn giải log BMS trước khi đưa vào prompt. |

### Quyết định cuối cùng của Ban Giám Đốc Vin Smart Future

[ ] **GO** · [x] **NOT YET (Cần tích lũy thêm dữ liệu / xác lập baseline)** · [ ] **NO-GO**

### Justification

> **Bài toán đúng, kiến trúc đúng, nhưng chưa đủ bằng chứng để build.**
>
> 1. **Phù hợp AI là có thật:** bottleneck nằm ở việc tổng hợp thông tin phi cấu trúc và soạn văn bản — việc rule-based không làm được và LLM làm tốt. Ranh giới vận hành rõ ràng (AI chỉ soạn nháp, con người quyết định tiền), nên rủi ro kiểm soát được. Vì vậy đây **không phải NO-GO**.
> 2. **Nhưng chưa thể GO** vì mọi con số trong báo cáo (42 phút/ca, 60 ca/ngày, 30% khiếu nại lại) đều là **giả định**. Không có baseline thì không chứng minh được ROI, và không có bộ ticket cũ đã gán kết luận thì **không đo được độ chính xác** của AI trước khi đưa cho CSKH dùng. Ngoài ra, Pháp chế chưa duyệt việc AI trích dẫn điều khoản trong văn bản gửi khách.
> 3. **Chi phí chờ thấp:** quy trình hiện tại vẫn chạy được, chỉ chậm. Trì hoãn 1–2 tháng để chuẩn bị không gây thiệt hại lớn, trong khi build sớm trên dữ liệu giả định có thể dẫn tới một sản phẩm không ai tin dùng.
>
> **Điều kiện để chuyển sang GO (dự kiến sau 6–8 tuần):**
> * Đo baseline thật: thời gian xử lý, số ca/ngày, tỷ lệ khiếu nại lại trong 4 tuần.
> * Chuẩn hóa ≥ 200 ticket cũ có kết luận cuối cùng làm bộ test.
> * Pháp chế duyệt danh sách điều khoản AI được phép trích dẫn; Kỹ thuật pin duyệt bảng diễn giải mã lỗi BMS.
> * Pilot đầu tiên chạy ở **chế độ shadow** (AI soạn nháp song song, CSKH vẫn làm thủ công, chỉ so sánh) tại 1 khu vực trước khi dùng thật.
