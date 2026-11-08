# 03 — AI Log & Reflection

> **Học viên:** Nguyễn Quang Trung — Branch: nguyenquangtrung-03389
> **Công cụ AI sử dụng:** Google Gemini & ChatGPT (OpenAI)
> **Bài toán thực hiện:** Vinhomes Complaint Routing — Phân loại tự động & Điều phối khiếu nại cư dân qua App Vinhomes Resident

---

## Mở đầu

Trong Lab 02, tôi tận dụng sự kết hợp giữa **Google Gemini** và **ChatGPT** làm bộ đôi trợ lý tư duy (thought-partners) xuyên suốt các bước: từ đọc hiểu yêu cầu bài tập, quét bài toán hệ sinh thái, dựng Quick Cards, phân tích quy trình đến viết prompt prototype. Bài viết này ghi lại trung thực trải nghiệm thực tế: bộ đôi AI đã giúp tôi những gì, những pha "bịa" số liệu hay giả định sai nào tôi phải chủ động sửa, và cách tôi điều chỉnh prompt cũng như ranh giới để kiểm soát mô hình.

---

## 1. AI đã giúp tôi những gì

**Bóc tách yêu cầu và tiêu chí ẩn.** Khi đưa file hướng dẫn và yêu cầu bài lab vào, Gemini và ChatGPT hỗ trợ đọc nhanh và chỉ ra cách chấm điểm thực tế trong file `autograder.py`. Nhờ vậy tôi nắm được các chi tiết ngầm dễ mất điểm: script phải chạy xong dưới 30 giây, lỡ xuất hiện chữ "Failed" ở bất kỳ đâu trong output là trượt phần test ranh giới, và file `.py` không được merge trực tiếp vào nhánh `main`.

**Phát triển và hệ thống hóa bài toán.** Từ ý tưởng cốt lõi là phân loại phản ánh ở Vinhomes, các công cụ AI giúp tôi mở rộng dải ý tưởng ra các đơn vị thành viên khác trong tập đoàn Vingroup (VinFast, Vinmec, Vinpearl, Vinschool) để có một danh sách 5 bài toán phong phú, áp dụng đúng các góc nhìn (Lens lặp lại/tốn thời gian).

**Phản biện việc áp dụng AI (Problem First, AI Second).** Tôi đánh giá cao việc Gemini và ChatGPT không cố thần thánh hóa AI vào mọi khâu. Khi phân loại khiếu nại cư dân, AI chỉ ra rằng các phàn nàn ngắn, cấu trúc rõ ràng nên ưu tiên dùng Rule-based hoặc Keyword-matching cho nhanh và rẻ; LLM chỉ thực sự cần thiết khi gặp các câu miêu tả dài dòng, lộn xộn hoặc chứa nhiều cảm xúc.

**Tăng tốc khâu trình bày và định dạng.** AI hỗ trợ dựng rất chuẩn các bảng ma trận so sánh 3 hướng tiếp cận (Rule vs LLM vs Agent), bảng quy trình Future-State với các luồng HITL / Fallback, và khung Problem Card. Nhờ đó, tôi tiết kiệm được rất nhiều thời gian định dạng Markdown để dồn sức cho tư duy nội dung.

---

## 2. AI đã sai hoặc "bịa" ở đâu

**Số liệu không có nguồn (Hallucination).** Tất cả các con số vận hành ban đầu — 15 phút xử lý thủ công, 8 phút đọc tin nhắn, 4 phút tra cứu sơ đồ — đều do AI tự ước tính dựa trên cảm tính. Chúng nghe rất hợp lý và được trình bày trôi chảy, nhưng không có bất kỳ nguồn dữ liệu thực tế nào từ ban quản lý Vinhomes. Nếu không cẩn thận, người đọc báo cáo sẽ tưởng đó là số liệu thống kê thật.

**Giả định sai về thực tế vận hành.** Ở các phiên bản thảo luận đầu tiên, AI hay mặc định quy trình đã được tự động hóa nhiều bước, bỏ qua thực tế là nhân viên CSKH tại các tòa nhà vẫn phải thao tác thủ công qua nhiều hệ thống phân mảnh. Nếu không tỉnh táo rà soát, quy trình hiện tại sẽ bị mô tả lệch so với thực tế vận hành.

**Tự suy diễn khi yêu cầu mơ hồ.** Khi tôi gõ câu lệnh ngắn như "làm tiếp phase 2 đi", thay vì hỏi lại để xác nhận scope, AI tự đoán ý và viết thẳng sang báo cáo Deep-Dive, khiến tôi phải yêu cầu dừng lại để điều chỉnh đúng cấu trúc từng phần.

**Tự phản biện chính mình.** Phần stress-test các Quick Card do AI tự đặt câu hỏi rồi tự trả lời hộ nên mang tính "vừa đá bóng vừa thổi còi", thiếu đi độ khắt khe thực tế của một người làm vận hành hay tài chính. Do đó, tôi phải tự đặt thêm các câu hỏi phản biện góc cạnh hơn để ép AI trả lời đúng trọng tâm.

---

## 3. Tôi đã sửa prompt và ranh giới như thế nào

**Bắt buộc gắn nhãn giả định và dám chọn NO-GO.** Tôi yêu cầu mọi con số thời gian phải được ghi rõ là "ước tính giả định". Đỉnh điểm là ở Phase 5 (EVALUATE), khi nhận thấy nhóm chưa có sẵn tập dữ liệu logs ticket thực tế được gán nhãn sạch, tôi quyết định chọn **NO-GO** thay vì cố bấm GO theo kịch bản lý tưởng của AI. Thà trung thực với năng lực dữ liệu hiện tại còn hơn dựa dẫm vào số liệu ảo.

**Viết prompt cụ thể và đưa khung mẫu.** Để tránh việc AI tự suy diễn định dạng, tôi cung cấp sẵn khung template và yêu cầu AI xuất dữ liệu đúng cấu trúc bảng hoặc Markdown chuẩn trước khi đưa vào tài liệu chính.

**Thiết lập ranh giới an toàn (Operational Boundary):** Trải nghiệm AI có thể bịa thông tin buộc tôi phải thiết kế cơ chế phòng vệ nghiêm ngặt cho bài toán Vinhomes:

* AI **tuyệt đối không có quyền tự động gửi tin nhắn hoặc tự đẩy ticket** sang bộ phận kỹ thuật.
* Mọi output của AI bắt buộc phải mang tiền tố `[DRAFT_ONLY]` và phải qua nút bấm phê duyệt của nhân viên CSKH (Human-in-the-loop).
* Xây dựng cổng an toàn Rule-based ở cửa ngõ để quét các từ khóa khẩn cấp (cháy, nổ, rò gas, ngập điện) và chuyển thẳng cho đội ứng phó sự cố trước khi dữ liệu chạm đến LLM.

---

## Kết luận

Gemini và ChatGPT là bộ đôi phụ tá cực kỳ xuất sắc trong việc hiểu yêu cầu, gợi ý ý tưởng, lập ma trận so sánh và chuẩn hóa trình bày. Tuy nhiên, AI **không phải là nguồn sự thật** và không thể thay thế cho tư duy vận hành thực tế. Bài học lớn nhất của tôi qua bài lab này là mọi kết quả do AI tạo ra đều phải được kiểm định, gắn nhãn độ tin cậy và đặt dưới sự kiểm soát chặt chẽ của con người — đó cũng chính là nguyên tắc cốt lõi khi thiết kế kiến trúc AI cho hệ thống xử lý khiếu nại cư dân Vinhomes.