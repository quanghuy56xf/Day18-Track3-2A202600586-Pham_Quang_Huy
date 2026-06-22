# Individual Reflection — Lab 18

**Tên:** Phạm Quang Huy  
**Module phụ trách:** Cả 5 Module (M1 - M5)

---

## 1. Đóng góp kỹ thuật

- **Module đã implement:** Đã triển khai và tối ưu hóa toàn bộ pipeline 5 module của Production RAG.
- **Các hàm/class chính đã viết:**
  - `chunk_semantic`, `chunk_hierarchical`, `chunk_structure_aware` (trong [m1_chunking.py](file:///d:/HocAI/Day18/src/m1_chunking.py))
  - `segment_vietnamese`, `BM25Search`, `DenseSearch` (local mode), `reciprocal_rank_fusion` (trong [m2_search.py](file:///d:/HocAI/Day18/src/m2_search.py))
  - `CrossEncoderReranker` (trong [m3_rerank.py](file:///d:/HocAI/Day18/src/m3_rerank.py))
  - `evaluate_ragas`, `failure_analysis` (trong [m4_eval.py](file:///d:/HocAI/Day18/src/m4_eval.py))
  - `enrich_chunks`, `_enrich_single_call` (trong [m5_enrichment.py](file:///d:/HocAI/Day18/src/m5_enrichment.py))
- **Số tests pass:** 37/37 tests pass hoàn toàn trong bộ test suite.

## 2. Kiến thức học được

- **Khái niệm mới nhất:** 
  - Hiểu sâu sắc cơ chế hoạt động của **Parent-Child Retrieval** (Hierarchical Chunking) để cân bằng giữa độ chính xác khi tìm kiếm và độ đầy đủ của ngữ cảnh truyền vào LLM.
  - Kỹ thuật **Reciprocal Rank Fusion (RRF)** giúp ghép kênh tìm kiếm từ khóa (BM25) và ngữ nghĩa (Dense) một cách trực quan, hiệu quả.
  - Kỹ thuật làm giàu dữ liệu **Contextual Prepend** giúp cải thiện đáng kể khả năng định vị thông tin của LLM.
- **Điều bất ngờ nhất:** Việc chạy song song Ragas evaluation có thể bị nghẽn và timeout lâu nếu API key gặp sự cố xác thực. Cách viết fallback logic cho evaluation giúp pipeline luôn hoạt động trơn tru.
- **Kết nối với bài giảng (slide nào):** Slide "Advanced Chunking & Indexing Techniques" và "Hybrid Retrieval & Reranking Strategy".

## 3. Khó khăn & Cách giải quyết

- **Khó khăn lớn nhất:** Lỗi Windows Segmentation Fault (native crash exit code 1) âm thầm xảy ra khi import thư viện `pypdf` cùng lúc với `sentence_transformers`.
- **Cách giải quyết:** Debug phân vùng lỗi bằng cách cô lập các module nhập thư viện. Giải quyết bằng cách cấu hình import `sentence_transformers` lên trên cùng ở các entry points để tránh xung đột tải DLL động trên Windows.
- **Thời gian debug:** 2.5 giờ.

## 4. Nếu làm lại

- **Sẽ làm khác điều gì:** Sẽ tích hợp thêm Metadata filters tự động ngay ở lớp index để giải quyết bài toán phiên bản tài liệu bị xung đột thời gian (như tài liệu năm 2023 và 2024).
- **Module nào muốn thử tiếp:** Muốn nghiên cứu sâu hơn về Reranker và thử nghiệm học cách tinh chỉnh (fine-tune) chéo Cross-Encoder cho tiếng Việt.

## 5. Tự đánh giá

| Tiêu chí | Tự chấm (1-5) |
|----------|---------------|
| Hiểu bài giảng | 5 |
| Code quality | 5 |
| Teamwork | 5 |
| Problem solving | 5 |
