# 🏗️ Phase 3 — DEEP-DIVE (Nhóm, 85 min)

**Bài toán chọn:** VinUni — xác nhận nộp bài lab trên LMS (Card #1 trong [01-problem-scan.md](01-problem-scan.md)). Báo cáo này theo Phase 3 và Phase 5 của [01-worksheet.md](01-worksheet.md). Dữ liệu hiện có là trải nghiệm ghi ở bước 1; chưa có log LMS, số liệu vận hành hay xác nhận từ quản trị viên. Thời gian dưới đây là **ước tính cá nhân**, không phải kết quả đo.

**Lựa chọn trong ba quick cards:** VinUni có quy trình và điểm nghẽn cụ thể, có thể kiểm chứng bằng bản ghi nộp bài. **Loại VinFast:** chưa xác minh luồng đổi lịch hiện hành, tần suất hay chính sách xưởng; nếu vấn đề được xác nhận, sửa lịch bằng quy tắc thông thường là hướng thử trước. **Loại Vinpearl:** mới có một phản ánh về trang trống/vòng lặp khi mở ưu đãi, chưa tái hiện hay có log lỗi; cần sửa luồng điều hướng sau khi xác định nguyên nhân. Đây là quyết định chọn đề tài, không phải kết luận hai sản phẩm không có lỗi.

## 3.1. Current-State Workflow Mapping (25 min)

Quy trình hiện tại của sinh viên khi nộp liên kết bài lab: [04-workflow-diagram.png](04-workflow-diagram.png)

**Điểm nghẽn:** Sau khi bấm nộp, giao diện chưa cung cấp bằng chứng đủ rõ để sinh viên tự phân biệt “đã lưu”, “đang xử lý” và “chưa lưu”. Handoff sang Discord tạo thêm việc cho cả sinh viên và coach. Cần xác minh chính xác thông báo LMS hiện có trước khi chốt thiết kế. Con số 5 phút từ quick card gộp phần kiểm tra/hỏi, chưa tách thời gian chờ coach.

## 3.2. Problem Statement (6-field) & Metrics (15 min)

| Field                       | Nội dung                                                                                                                                                                                                                                                                                                                                                                                                   |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Actor / Operator**     | Sinh viên nộp bài lab trên LMS; lab coach là người được hỏi xác nhận khi trạng thái không rõ.                                                                                                                                                                                                                                                                                                              |
| **2. Current Workflow**     | Sinh viên tìm bài trên LMS, dán URL bài làm, gửi, đọc phản hồi rồi hỏi lab coach qua Discord nếu chưa chắc hệ thống đã lưu.                                                                                                                                                                                                                                                                                |
| **3. Bottleneck**           | Xác nhận sau nộp thiếu tín hiệu có thể kiểm chứng; sinh viên phải tự đối chiếu hoặc hỏi người khác. Chưa xác định đây là lỗi lưu dữ liệu hay chỉ là lỗi truyền đạt trạng thái.                                                                                                                                                                                                                             |
| **4. Business Impact**      | Chi phí khả dĩ là thời gian người học chờ/hỏi và thời gian coach xử lý câu hỏi lặp lại. **Chưa có** số lượt nộp, số câu hỏi, thời gian xử lý thực đo, chi phí tiền hoặc SLA để định lượng tác động cho VinUni. Cần đo số lượt nộp, số lượt hỏi xác nhận và thời gian coach xử lý trước khi ước tính tác động.                                                                                              |
| **5. Success Metric**       | **Mục tiêu thử nghiệm, chưa phải số liệu đạt được:** 100% lần gửi có kết quả rõ trong ≤10 giây: biên nhận **chỉ khi** bản ghi đã được LMS xác nhận, hoặc lỗi/trạng thái đang xử lý với cách kiểm tra lại. Giảm ≥80% tỷ lệ lượt nộp dẫn đến câu hỏi “bài đã nộp chưa?” trên Discord so với baseline cùng loại lab; cần đo baseline trước.                                                                   |
| **6. Operational Boundary** | Dịch vụ chỉ đọc/ghi trạng thái từ nguồn dữ liệu nộp bài có thẩm quyền và hiển thị biên nhận kèm mã bài, thời điểm máy chủ, trạng thái và URL đã lưu. Không tuyên bố “đã nộp” dựa riêng vào việc bấm nút hoặc phản hồi suy đoán; không tự sửa URL, nộp hộ, thay đổi hạn nộp hoặc điểm. Khi kết quả lưu không chắc chắn, hiển thị “chưa xác nhận”, cho kiểm tra lại và chuyển hỗ trợ cho coach/quản trị LMS. |

**Cách đo đề xuất:** Ghi log sự kiện `submit_clicked`, `save_confirmed`, `save_failed`, `receipt_rendered` và thời gian máy chủ; dùng mẫu thử có ca thành công, lỗi mạng, gửi trùng và hết hạn. Với cùng loại lab, đo hai tuần trước và hai tuần sau thử nghiệm: tỷ lệ = số lượt nộp dẫn đến câu hỏi xác nhận / tổng số lượt nộp; ghi thời gian coach xử lý mỗi câu hỏi để ước tính thời gian mất. Ghép câu hỏi với lượt nộp theo dữ liệu được phép truy cập; không tính mọi tin Discord là câu hỏi về nộp bài. Chỉ dùng dữ liệu đã được phép truy cập và hạn chế thông tin cá nhân trong log.

## 3.3. Future-State Flow & AI Fit (25 min)

| Phương án                | Khả năng giải quyết điểm nghẽn                                                                                                   | Đánh giá                                                                                                                                       |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Rule / state machine** | Đọc kết quả lưu thật từ LMS, quản lý trạng thái `đang xử lý / đã lưu / lỗi / chưa xác nhận`, phát biên nhận và cho kiểm tra lại. | **Chọn cho bản thử nghiệm**: đầu ra cần chính xác, có thể kiểm chứng và có cấu trúc.                                                           |
| **LLM feature**          | Có thể diễn giải thông báo tự do nếu LMS chỉ có lỗi dạng văn bản.                                                                | Chưa cần; không được dùng nội dung LLM làm bằng chứng bài đã lưu. Chỉ xem xét sau khi có mẫu lỗi thực tế không xử lý được bằng mã lỗi/quy tắc. |
| **Agentic loop**         | Tự tìm trạng thái hoặc tác động lên bài nộp qua nhiều bước.                                                                      | Không phù hợp: tăng quyền và rủi ro thao tác sai mà không thêm giá trị cho xác nhận giao dịch.                                                 |

- **AI Fit:** [x] Rule / State-Machine [ ] LLM Feature [ ] Agentic Loop. Trạng thái nộp bài là dữ liệu giao dịch có cấu trúc; LLM hoặc Agent không thể tự chứng minh bản ghi đã được lưu.
- **Quy trình tương lai (Future-State):**

```text
┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐
│ Bước 1           │   │ Bước 2           │   │ Bước 3           │
│ Sinh viên gửi URL│ → │ LMS kiểm tra và  │ → │ Rule đọc kết quả │
│ cho đúng bài lab │   │ lưu bản nộp      │   │ lưu có thẩm quyền│
└──────────────────┘   └──────────────────┘   └──────────────────┘
                                                   │
                                                   ▼
                         ┌──────────────────┐   ┌──────────────────┐
                         │ Bước 5           │   │ Bước 4           │
                         │ 🟢 Sinh viên     │ ← │ Hiển thị biên    │
                         │ kiểm tra URL,    │   │ nhận nếu đã lưu; │
                         │ trạng thái       │   │ lỗi/chưa xác nhận│
                         └──────────────────┘   └──────────────────┘
                                                   │
                                                   ▼
                                      ↩️ Fallback: Tra cứu lại bản ghi;
                                      nếu chưa xác minh, chuyển coach/
                                      quản trị LMS, không báo thành công.
```

**🔵 AI step:** Không có trong phương án được chọn. **🟢 HITL:** Sinh viên kiểm tra URL và trạng thái; lab coach/quản trị LMS xử lý ngoại lệ có mã yêu cầu. **↩️ Fallback:** Tra cứu lại bản ghi; nếu không xác minh được, chuyển cho người phụ trách và tuyệt đối không tạo biên nhận thành công. Trạng thái và quyền xử lý cụ thể cần chốt với chủ hệ thống LMS.

---

# 🏁 Phase 5 — EVALUATE (Nhóm, 20 min)

### AI Readiness Checklist

1. [ ] **Có dữ liệu mẫu/log sạch để test?** Chưa được cung cấp log LMS hoặc quyền truy cập dữ liệu nộp bài. Cần chủ hệ thống cung cấp mẫu ẩn danh và quy tắc trạng thái.
2. [x] **Có thể kiểm soát rủi ro sai?** Thiết kế chỉ phát biên nhận khi xác minh được bản ghi; còn lại báo chưa xác nhận, cho tra cứu lại và chuyển người xử lý. Chưa xác minh được quyền truy cập trạng thái LMS hoặc kiểm thử cơ chế này.
3. [ ] **Stakeholders sẵn sàng đổi quy trình?** Chưa có xác nhận từ lab coach, quản trị LMS hay nhóm học viên; cần xem thử luồng và đồng ý cách xử lý ngoại lệ.

### Quyết định cuối cùng của Ban Giám Đốc Vin Smart Future:

- [ ] **GO (Bắt đầu xây dựng Prototype)** cho dự án AI.
- [ ] **NOT YET (Cần tích lũy thêm dữ liệu/xác lập baseline)** cho dự án AI.
- [x] **NO-GO (Rule-based tốt hơn)** cho dự án AI này.

**Đôi lời góp ý từ học sinh khoá V AI thực chiến:** Vấn đề này có thể sẽ chỉ xảy ra vào những lúc đầu tiên của các khoá AI thực chiến, và vấn đề này có thể xuất hiện ở phần trăm ít những người học và không gây nhiều tổn thất. Tuy nhiên việc này có thể gây ra không ít phiền toái cho cả người học và lab coach, đặc biệt là với những lab coach. Đề xuất một chút tinh chỉnh cho trang báo lại kết quả: Thay vì ghi "Waiting for admin to verify" thì hãy ghi thành "Summited successfully", hiển thị thời điểm summit bài thay vì ghi thời gian hoàn thành. Tính năng cho người học nộp lại bài có thể để sau. Về phía lab coach dạy trên giảng đường, hãy hướng dẫn người học nộp bài trước khi để người học bắt đầu thực sự làm bài. Cảm ơn vì đã đọc.

---
