# KẾ HOẠCH PHÂN CÔNG NHÓM 3 NGƯỜI
## Codelab 03: The Multi-Modal Minefield — Data Pipeline Engineering

---

## Tổng quan dự án

Mục tiêu: Xây dựng pipeline xử lý dữ liệu phi cấu trúc từ hai nguồn (PDF và Video), chuẩn hóa về một schema thống nhất, và xuất ra `processed_knowledge_base.json` phục vụ AI Agent.

---

## Phân công vai trò

Vì nhóm có 3 người thay vì 4, vai trò **ETL/ELT Builder** và **Observability & QA Engineer** được gộp lại vì hai nhiệm vụ này có liên quan chặt chẽ về mặt luồng dữ liệu.

---

### Thành viên 1 — Kiến trúc sư Dữ liệu (Lead Data Architect)
**File phụ trách:** `starter_code/schema.py`

**Nhiệm vụ:**
- Định nghĩa model `UnifiedDocument` bằng Pydantic với 6 trường bắt buộc:
  - `document_id` (str)
  - `source_type` (str)
  - `author` (str)
  - `category` (str)
  - `content` (str)
  - `timestamp` (str)
- Đảm bảo schema tương thích với cả dữ liệu PDF (Group A) và Video (Group B).
- Trao đổi với Thành viên 2 để xác nhận các key mapping từ JSON thô sang schema.

**Ưu tiên:** Hoàn thành **trước tiên** — các thành viên còn lại phụ thuộc vào schema này.

**Thời hạn nội bộ:** Hoàn thành trước khi Thành viên 2 bắt đầu viết hàm xử lý.

---

### Thành viên 2 — ETL Builder kiêm QA Engineer (Transformation & Quality Lead)
**File phụ trách:** `starter_code/process_unstructured.py` + `starter_code/quality_check.py`

**Nhiệm vụ — ETL (`process_unstructured.py`):**
- Viết hàm `process_pdf_data(raw_json)`:
  - Dùng `re.sub` để loại bỏ noise dạng `HEADER_PAGE_X` và `FOOTER_PAGE_X` khỏi trường `extractedText`.
  - Map các key của JSON thô PDF sang schema chuẩn của `UnifiedDocument`.
- Viết hàm `process_video_data(raw_json)`:
  - Map các key Video (`video_id`, `creator_name`, `transcript`, `category`, `published_timestamp`) sang schema chuẩn.

**Nhiệm vụ — QA (`quality_check.py`):**
- Viết hàm `run_semantic_checks(doc_dict)`:
  - Kiểm tra `content` không trống và có ít nhất 10 ký tự.
  - Phát hiện các từ khóa lỗi: `"Null pointer exception"`, `"OCR Error"`, `"Traceback"` — nếu xuất hiện thì trả về `False`.
  - Trả về `True` nếu tài liệu hợp lệ.

**Lưu ý:** Chờ Thành viên 1 hoàn thành schema trước khi code các hàm `return {}`.

---

### Thành viên 3 — DevOps & Integration Specialist (The Connector)
**File phụ trách:** `starter_code/orchestrator.py`

**Nhiệm vụ:**
- Hoàn thiện hàm `run_pipeline()`:
  - **Group A (PDF):** Đọc tất cả file JSON trong `raw_data/group_a_pdfs/`, gọi `process_pdf_data()`, chạy `run_semantic_checks()`, thêm vào `final_kb` nếu đạt.
  - **Group B (Video):** Làm tương tự với `raw_data/group_b_videos/` và `process_video_data()`.
  - Lưu kết quả ra `processed_knowledge_base.json`.
- Chạy `python orchestrator.py` để kiểm tra toàn bộ pipeline.
- Xử lý lỗi nếu file JSON không đọc được hoặc pipeline báo 0 records.

**Lưu ý:** Có thể bắt đầu đọc code orchestrator song song, nhưng chỉ chạy test sau khi Thành viên 1 và 2 hoàn thành.

---

## Trình tự thực hiện

```
[Thành viên 1] schema.py
        │
        ▼
[Thành viên 2] process_unstructured.py + quality_check.py
        │
        ▼
[Thành viên 3] orchestrator.py → chạy python orchestrator.py
```

---

## Tiêu chí hoàn thành

- [x] `schema.py`: `UnifiedDocument` có đủ 6 trường kiểu `str`. ✅
- [x] `process_unstructured.py`: Hai hàm trả về dict đúng cấu trúc schema, PDF đã loại bỏ noise. ✅
- [x] `quality_check.py`: Hàm lọc đúng tài liệu lỗi, giữ lại tài liệu hợp lệ. ✅
- [ ] `orchestrator.py`: Pipeline chạy thành công, xuất `processed_knowledge_base.json` với số records > 0.

---

## Ghi chú

- Thứ tự phụ thuộc: **Schema → ETL+QA → Orchestrator**. Không nên bỏ qua thứ tự này.
- Dùng nhánh Git riêng cho từng thành viên nếu cần, merge vào `main` theo thứ tự.
- Giao tiếp trong nhóm quan trọng: nếu schema thay đổi, Thành viên 1 phải thông báo ngay cho Thành viên 2.
