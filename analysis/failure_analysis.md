# Failure Analysis — Lab 18: Production RAG

**Nhóm:** RAG-Master  
**Thành viên:** Phạm Quang Huy — Phụ trách toàn bộ Pipeline (M1 - M5)

---

## RAGAS Scores

| Metric | Naive Baseline | Production | Δ |
|--------|---------------|------------|---|
| Faithfulness | 0.0000 | 0.0000 | +0.0000 (Offline Fallback) |
| Answer Relevancy | 0.0000 | 0.0000 | +0.0000 (Offline Fallback) |
| Context Precision | 0.0000 | 0.0000 | +0.0000 (Offline Fallback) |
| Context Recall | 0.0000 | 0.0000 | +0.0000 (Offline Fallback) |

> [!NOTE]
> Do API Key của OpenAI bị giới hạn (Mock/Offline mode), hệ thống RAGAS Evaluation tự động chạy chế độ fallback trả về điểm số 0.0. Các phân tích lỗi dưới đây được chẩn đoán thủ công dựa trên kết quả câu trả lời thực tế (Actual answers) và tài liệu tham chiếu (Contexts).

---

## Bottom-5 Failures

### #1
- **Question:** Thâm niên bao nhiêu năm thì được cộng thêm ngày phép?
- **Expected:** Theo chính sách v2024 hiện hành, thâm niên từ 3 năm trở lên được cộng thêm 1 ngày phép cho mỗi 3 năm. Chính sách cũ v2023 là 5 năm đã bị thay thế.
- **Got:** Trích từ nghi_phep_nam_v2023.md. Nhân viên có thâm niên từ 5 năm trở lên được cộng thêm 1 ngày phép cho mỗi 5 năm làm việc liên tục.
- **Worst metric:** Context Precision / Faithfulness (do trả về thông tin lỗi thời v2023).
- **Error Tree:** Output sai (v2023) → Context sai (retrieved v2023 ở top 1) → Query OK? → Có.
- **Root cause:** Mô hình Dense Embedding / BM25 Search có độ tương đồng từ vựng cao với phiên bản cũ (v2023) và xếp hạng nó cao hơn phiên bản mới (v2024). Do chạy offline fallback nên LLM không thể tổng hợp hay chọn lọc giữa 2 thông tin xung đột.
- **Suggested fix:** Cấu hình Metadata filter để lọc bỏ các tài liệu cũ (ví dụ: chỉ lấy tài liệu có hiệu lực năm 2024), hoặc bổ sung Reranker có nhận biết về ngày hiệu lực / phiên bản của tài liệu.

### #2
- **Question:** Nhân viên được nghỉ bao nhiêu ngày phép năm?
- **Expected:** Theo chính sách v2024 hiện hành, nhân viên được nghỉ 15 ngày phép năm có lương.
- **Got:** Trích từ nghi_phep_nam_v2024.md. Mỗi nhân viên chính thức được hưởng 15 ngày phép năm có lương, tăng từ 12 ngày so với chính sách năm 2023.
- **Worst metric:** Faithfulness / Answer Relevancy.
- **Error Tree:** Output đúng (15 ngày) → Context đúng (nghi_phep_nam_v2024.md) → Query OK? → Có.
- **Root cause:** Mặc dù retriever trả về context của năm 2024 ở top 1, nhưng vì chạy offline fallback, câu trả lời thực chất chỉ là trích đoạn thô từ tài liệu thay vì câu trả lời tổng hợp ngắn gọn từ LLM.
- **Suggested fix:** Sử dụng OpenAI API key hợp lệ để LLM sinh câu trả lời tự nhiên, tránh trích xuất thô.

### #3
- **Question:** Nhân viên thử việc có được nghỉ phép năm không?
- **Expected:** Nhân viên thử việc KHÔNG được nghỉ phép năm.
- **Got:** Trích từ thu_viec.md. Phụ cấp ăn trưa được áp dụng đầy đủ từ ngày đầu tiên. ## Quyền lợi trong thử việc. Nhân viên thử việc KHÔNG được nghỉ phép năm.
- **Worst metric:** Answer Relevancy (câu trả lời chứa nhiều thông tin thừa về phụ cấp ăn trưa).
- **Error Tree:** Output đúng một phần nhưng thừa → Context đúng → Query OK? → Có.
- **Root cause:** Do fallback lấy toàn bộ text chunk ở vị trí đầu tiên dẫn tới việc câu trả lời bị thừa các ý khác nằm chung trong chunk.
- **Suggested fix:** Implement LLM generation tốt hơn hoặc giảm kích thước child chunk trong Hierarchical Chunking để tăng tính cô đọng của kết quả tìm kiếm.

### #4
- **Question:** Nhân viên được tài trợ khóa học 25 triệu, nghỉ việc sau 8 tháng hoàn chi phí bao nhiêu?
- **Expected:** Khóa học 25 triệu nằm trong khung [20-50 triệu], cam kết làm việc 12 tháng. Nghỉ việc sau 8 tháng (còn thiếu 4 tháng) phải hoàn trả: (25.000.000 / 12) * 4 = 8.333.333 VNĐ.
- **Got:** Trích từ hoan_chi_dao_tao.md. Nhân viên nghỉ việc trước thời hạn cam kết phải hoàn trả chi phí đào tạo theo tỷ lệ: Hoàn trả = (Tổng chi phí / Thời gian cam kết) * Số tháng còn thiếu.
- **Worst metric:** Faithfulness (không thực hiện được phép tính số học cụ thể).
- **Error Tree:** Output là công thức thay vì số tiền cụ thể → Context chứa công thức đầy đủ → Query yêu cầu tính toán → Thất bại do thiếu LLM Reasoning.
- **Root cause:** Các câu hỏi mang tính lập luận logic/tính toán số học yêu cầu LLM có khả năng suy luận (như gpt-4o-mini) để tính toán dựa trên công thức lấy từ context. Khi chạy offline fallback, hệ thống không thể tính ra con số 8.333.333 VNĐ.
- **Suggested fix:** Cấp quyền API Key đầy đủ và thiết lập System Prompt yêu cầu LLM thực hiện tính toán từng bước (Chain-of-Thought) dựa trên công thức trong tài liệu.

### #5
- **Question:** Nghỉ phép không lương 20 ngày cần ai phê duyệt?
- **Expected:** Nghỉ phép không lương tối đa 30 ngày cần được Giám đốc bộ phận phê duyệt.
- **Got:** Trích từ nghi_phep_khong_luong.md. Nhân viên có thể xin nghỉ phép không lương tối đa 30 ngày mỗi năm. Đơn xin nghỉ phải được Giám đốc bộ phận phê duyệt.
- **Worst metric:** Answer Relevancy (trả về toàn bộ đoạn văn thay vì trả lời trực tiếp là "Giám đốc bộ phận").
- **Error Tree:** Output đúng ý chính nhưng dư thừa → Context đúng → Query OK? → Có.
- **Root cause:** Tương tự các trường hợp trên, việc thiếu LLM Generation làm giảm độ cô đọng của câu trả lời.
- **Suggested fix:** Sử dụng mô hình LLM để rút gọn câu trả lời dựa trên context được cung cấp.

---

## Case Study (cho presentation)

**Question chọn phân tích:**  
"Thâm niên bao nhiêu năm thì được cộng thêm ngày phép?" (Question #5)

**Error Tree walkthrough:**  
1. **Output đúng?** → Sai. Output trả về quy định năm 2023 (5 năm thâm niên được cộng 1 ngày phép) trong khi quy định hiện hành năm 2024 là 3 năm.
2. **Context đúng?** → Sai một phần. Retriever đã trả về cả hai tài liệu `nghi_phep_nam_v2023.md` (top 1) và `nghi_phep_nam_v2024.md` (top 2). Thứ tự xếp hạng của tài liệu cũ cao hơn tài liệu mới.
3. **Query rewrite OK?** → Đúng. Câu hỏi rõ ràng và trực tiếp.
4. **Fix ở bước:** Bước tìm kiếm (Retrieval - Module 2) và Reranking (Module 3).

**Nếu có thêm 1 giờ, sẽ optimize:**  
1. **Metadata Filtering:** Thêm trường `year` hoặc `status: "active|deprecated"` vào metadata của mỗi chunk lúc index. Khi query, tự động filter chỉ tìm kiếm trên các chunk active.
2. **Contextual Enrichment:** Thêm nhãn thời gian hoặc thông tin phiên bản trực tiếp vào nội dung chunk thông qua Enrichment Pipeline để mô hình embedding nhận diện được tính hiệu lực tốt hơn.
