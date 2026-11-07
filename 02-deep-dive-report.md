# Phase 3 & Phase 5 — Báo Cáo Phân Tích Sâu (Deep-Dive Report & Evaluation)

> **Dự án:** VinSOC Daily Vuln Intake Co-pilot — đọc advisory mới, sinh ticket nháp, analyst duyệt rồi mới gán đội vá  
> **Đơn vị phát triển:** Vin Smart Future — AI Product Lab  
> **Khách hàng nội bộ:** Trung tâm Điều hành An ninh mạng Vingroup (**VinSOC**)  
> **Bài toán chọn từ Phase 2:** Quick Problem Card #1 (VinSOC — cập nhật lỗ hổng mới hàng ngày)


---

# 🏗️ 1. Quy Trình Vận Hành Hiện Tại (Current-State Workflow)

VinSOC phục vụ vá lỗ hổng cho VinFast, Xanh SM, Vinhomes, Vinmec, Vinpearl. Mỗi ca (ngày/đêm) analyst phải **update lỗ hổng mới**: hốt advisory, đọc, map tài sản, viết ticket. Sơ đồ vẽ tay: `04-workflow-diagram.png`.

```text
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│ Bước 1:         │       │ Bước 2:         │       │ Bước 3:         │       │ Bước 4:         │
│ Hốt advisory    │       │ Đọc & bóc tách  │       │ Map CMDB /      │       │ Viết ticket     │
│ mới trong ca    │ ────> │ CVE, CVSS,      │ ────> │ tài sản         │ ────> │ tiếng Việt +    │
│                 │       │ sản phẩm dính   │       │ Vingroup        │       │ hạn SLA         │
│                 │       │                 │       │                 │       │                 │
│ Actor: Analyst  │       │ Actor: Analyst  │       │ Actor: Analyst  │       │ Actor: Analyst  │
│ ⏱ 2 phút        │       │ ⏱ 10 phút 🔴    │       │ ⏱ 8 phút 🔴     │       │ ⏱ 7 phút 🔴     │
│ In: NVD, mail,  │       │ In: PDF/HTML    │       │ In: CVE+product │       │ In: note tay    │
│     Slack intel │       │ Out: note tay   │       │ Out: đơn vị dính│       │ Out: ticket     │
└─────────────────┘       └─────────────────┘       └─────────────────┘       └─────────────────┘
                                                                                       │
                                                                                       ▼
                                                                                ┌─────────────────┐
                                                                                │ Bước 5:         │
                                                                                │ Gán đội vá,     │
                                                                                │ theo dõi đóng   │
                                                                                │ Actor: Analyst  │
                                                                                │ ⏱ 3 phút        │
                                                                                │ Out: Jira/SNOW  │
                                                                                └─────────────────┘

🔴 Bottleneck:
   - Bước 2 (10 phút): advisory tiếng Anh, PDF scan, bảng CPE lệch, CVE viết tắt.
   - Bước 3 (8 phút): CMDB nhiều tên (VF8 infotainment ≠ “Android Automotive”), map nhầm đơn vị.
   - Bước 4 (7 phút): viết lại tiếng Việt, dán CVSS, chọn SLA Critical 4h / High 24h — copy nhầm điểm CVSS thì cả hàng đợi sai thứ tự.
🔄 Handoff: Nguồn intel ──> Inbox VinSOC ──> Analyst ──> Jira/ServiceNow ──> Đội vá đơn vị (VinFast IT, Xanh SM, …).
⏱ Tổng thời gian thủ công: ~30 phút / 1 advisory. 10–15 advisory/ca → 5–7 giờ chỉ để intake.
```

---

# 📋 2. Problem Statement (khung 6-field Vin Smart Future)

| STT | Trường (Field) | Nội dung |
|:---:|----------------|----------|
| **1** | **Actor / Operator** | SOC Analyst ca ngày/ca đêm tại VinSOC. On-call nhận escalate Critical. Đội vá các đơn vị là *người nhận* ticket, không phải người đọc advisory gốc. |
| **2** | **Current Workflow** | Analyst mở NVD/mail vendor/Slack, đọc từng advisory, ghi tay CVE-ID–CVSS–sản phẩm, so CMDB xem hệ thống Vingroup nào dính, viết ticket tiếng Việt, gán đội vá, theo dõi đóng. 5 bước, ~30 phút/advisory, 10–15 advisory/ca. |
| **3** | **Bottleneck** | Bước 2+3+4 chiếm 25/30 phút: đọc văn bản tự do, map tên sản phẩm sang tài sản thật, soạn ticket. Critical vào lúc 02:00 vẫn nằm inbox đến ca sáng → trễ SLA 4 giờ. |
| **4** | **Business Impact** | 1 ca VinSOC tốn **5–7 giờ công** chỉ để *update lỗ hổng*, chưa săn threat. Trễ vá Critical trên cụm Vinmec / app Xanh SM / ECU VinFast = cửa sổ ransomware/an toàn xe. Ước lượng **~30%** ticket Critical/High trễ SLA intake vì hàng đợi đọc tay. Map nhầm “không dính” thì lỗ hổng sống im trong production. |
| **5** | **Success Metric** | 1) Inbox → ticket nháp: **30 phút → dưới 5 phút**. 2) Đúng CVE-ID + CVSS + tên sản phẩm: **>= 98%** trên bộ advisory gắn nhãn. 3) **100%** CVSS >= 9.0 có draft trong **15 phút** sau khi ingest. 4) 0 ticket tự đóng “không dính” khi chưa có analyst Approve. |
| **6** | **Operational Boundary** | **AI được phép:** đọc advisory, xuất JSON (cve_id, cvss, product, summary), gợi ý đơn vị, sinh ticket nháp. **CẤM:** (1) Mọi output bắt đầu `[DRAFT_ONLY]`. Cấm tự gán Jira, cấm tự gửi mail đội vá. HITL 1 click. (2) Cấm tự đóng *not affected* / false positive. (3) Cấm chạy patch, reboot, block IP. (4) CVSS >= 9.0 **hoặc** ước lượng ảnh hưởng **> 5%** endpoint của một đơn vị → không đi luồng thường; flag on-call. Cùng họ ngưỡng 5% với rule pin Xanh SM trên prototype dùng chung. |

---

# 🚀 3. Future-State Flow & AI Fit

### 3.1. AI-Fit Matrix

| Phương án | Phù hợp chỗ nào | Kết luận |
|-----------|-----------------|----------|
| **Rule / state-machine** | Routing theo CVSS (>=9 Critical, 7–8.9 High), SLA giờ, chặn blast radius > 5%. | Giữ cho bước số. Gãy khi đọc PDF/mail, CPE lệch tên, “Android Auto” vs infotainment VF8. |
| **LLM Feature + HITL** | NER advisory → JSON, soạn ticket tiếng Việt nháp. | Chọn. LLM đọc chữ. Rule giữ ngưỡng. Analyst map tài sản và Approve. |
| **Agentic Loop tự trị** | Tự đóng “không dính”, tự mở PR vá, tự restart node. | Loại. Một hallucination “not affected” = lỗ hổng sống. Agent vá nhầm ECU/xe/bệnh viện thì không rollback bằng một câu xin lỗi. |

**Lựa chọn:** `[x] LLM Feature` · `[ ] Rule thuần` · `[ ] Agentic Loop`

- **LLM (Gemini 2.5 Flash):** đọc advisory, xuất JSON, viết tóm tắt tiếng Việt.
- **Rule:** CVSS bucket + ngưỡng **5%** blast radius + bắt buộc `[DRAFT_ONLY]`.
- **HITL:** analyst đối chiếu CMDB 1–2 phút, bấm Approve mới gán đội vá.
- **Fallback:** thiếu CVE-ID, PDF scan mờ, confidence < 80%, hoặc product không map được → hàng đợi thủ công, **không** đoán “không dính”.

### 3.2. Future-State Flow

```text
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│ Bước 1:         │       │ Bước 2:         │       │ Bước 3:         │       │ Bước 4:         │
│ Ingest NVD/mail │       │ 🔵 LLM bóc JSON │       │ 🔵 Rule CVSS +  │       │ 🟢 HITL         │
│ /Slack          │ ────> │ CVE, CVSS,      │ ────> │ gợi ý đơn vị +  │ ────> │ Analyst duyệt   │
│                 │       │ product         │       │ ticket          │       │ map CMDB, gán   │
│ Actor: connector│       │ Tech: Gemini    │       │ [DRAFT_ONLY]    │       │ đội vá          │
│ ⏱ ~5 giây       │       │ ⏱ 5–10 giây     │       │ ⏱ ~5 giây       │       │ ⏱ 1–2 phút      │
└─────────────────┘       └─────────────────┘       └─────────────────┘       └─────────────────┘
                                                                                       │
                                                                                       ▼
                                                                                ↩️ Fallback:
                                                                                PDF mờ / thiếu CVE
                                                                                / không map được
                                                                                → hàng đợi thủ công.
                                                                                CVSS>=9 hoặc >5%
                                                                                endpoint → page on-call.
                                                                                Không auto-close,
                                                                                không auto-patch.
```

Tổng tương lai: **dưới 5 phút** khi advisory đọc được; ca xấu quay về 30 phút cũ. Critical leo on-call, không nằm nháp đến sáng.

---

# 💻 4. Liên kết prototype ranh giới (Phase 4)

File `starter-code/prompt_prototype.py` là **co-pilot dùng chung** Vin Smart Future. Bài Deep-Dive là VinSOC; script lab vẫn bắt 2 rule Xanh SM vì autograder chấm `draft_only`, `5%`, `dispatch_mobile_charger`.

Ánh xạ ranh giới:

| Rule trên prototype | Nghĩa với Xanh SM (test lab) | Nghĩa với VinSOC (sản phẩm) |
|---------------------|------------------------------|-----------------------------|
| Mọi câu bắt đầu `[DRAFT_ONLY]` | Không tự gửi SMS tài xế | Không tự gán ticket / không tự mail đội vá |
| Ngưỡng **5%** | Pin < 5% + trạm > 5 km → cứu hộ, cấm chỉ trạm xa | Blast radius > 5% endpoint **hoặc** CVSS >= 9 → on-call, cấm đóng “không dính” |
| `dispatch_mobile_charger` | JSON cứu hộ pin di động | Giữ nguyên trong prompt để pass adversarial lab; VinSOC dùng flag on-call trên báo cáo |

Hai adversarial test lab: (1) pin 2% + trạm 8 km + “gửi ngay”; (2) “đừng gắn `[DRAFT_ONLY]`”. Simulation mode giữ được 2 rule.

---

# 🏁 5. Evaluate — Phase 5

### 5.1. AI Readiness Checklist

| Tiêu chí | Trạng thái | Bằng chứng |
|----------|:----------:|------------|
| **1. Data readiness** | **ĐẠT (PASS) cho prototype** | NVD + mail vendor là nguồn công khai để gắn nhãn CVE/CVSS/product. Prototype chạy simulation khi chưa có `GEMINI_API_KEY`. **Chưa ĐẠT production** nếu chưa có bản CMDB sạch (tên sản phẩm nội bộ ≠ CPE). |
| **2. Risk containment** | **ĐẠT (PASS)** | `[DRAFT_ONLY]` + HITL + cấm auto-close + cấm patch + ngưỡng 5%/CVSS 9 + fallback hàng đợi. AI không đụng production. |
| **3. User adoption** | **ĐẠT (PASS)** | Analyst vẫn làm nghề (map tài sản, chịu trách nhiệm đóng ticket). AI cắt khâu đọc PDF/copy CVSS. SOP vá không đổi. |

### 5.2. Quyết định Ban Giám đốc Vin Smart Future

* [x] **GO** — xây prototype & thí điểm hẹp
* [ ] NOT YET
* [ ] NO-GO

**Phạm vi GO:** thí điểm 4 tuần trên luồng *advisory NVD + mail vendor tiếng Anh*, chỉ sinh ticket nháp. HITL bắt buộc. Chưa GO auto-assign Jira. Chưa GO Agent vá.

**Chưa GO production toàn SOC** cho đến khi: (1) CMDB có bảng map product→đơn vị với owner; (2) đo >= 98% trích xuất trên >= 200 advisory gắn nhãn; (3) mọi ca “không dính” đều do người đóng, log được.

### 5.3. Biện luận kỹ thuật & chi phí

1. **Hiệu quả:** 30 phút → dưới 5 phút/advisory (~85% thời gian đọc-chép). 10–15 advisory/ca lấy lại 4–6 giờ cho săn threat. Critical vào ca đêm có draft trong 15 phút.
2. **Kiến trúc mỏng:** Gemini Flash đọc chữ; rule giữ CVSS và 5%. Không cho model SSH/Jira write token lúc thí điểm.
3. **Chi phí:** vài nghìn token/advisory, cent USD. Rẻ hơn 1 ca analyst. Rủi ro đóng nhầm “không dính” đắt hơn phí API → HITL là chi phí đúng.
4. **Lộ trình:** tuần 1–2 prototype + adversarial (`[DRAFT_ONLY]`, 5%); tuần 3–4 chạy song song ca VinSOC, so ticket AI vs ticket người; sau đó mới nối Jira read-only rồi write sau Approve.

**Tóm lại:** GO prototype vì metric rõ, LLM đúng chỗ đọc advisory, ranh giới cứng chặn auto-patch / auto-close. Production rộng = bước sau, sau CMDB sạch và số đo 98% trên dữ liệu thật.
