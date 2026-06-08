# RAG Evaluation Results

## Framework sử dụng

RAGAS (local/offline implementation). Script chạy deterministic, không cần API key, và báo cáo đủ 4 metric bắt buộc: faithfulness, answer relevance, context recall, context precision.

---

## Overall Scores

| Metric | Config A (hybrid + rerank) | Config B (dense-only) | Δ |
|--------|---------------------------|----------------------|---|
| Faithfulness | 0.990 | 0.998 | -0.008 |
| Answer Relevance | 0.447 | 0.279 | +0.168 |
| Context Recall | 0.592 | 0.477 | +0.115 |
| Context Precision | 0.973 | 0.760 | +0.213 |
| Average | 0.750 | 0.629 | +0.121 |

---

## A/B Comparison Analysis

**Config A:** Hybrid retrieval gồm semantic search + BM25 lexical search, merge bằng RRF, sau đó rerank local theo coverage/overlap và score gốc.

**Config B:** Dense-only retrieval chỉ dùng semantic search local TF-IDF/cosine, không BM25 và không reranking.

**Kết luận:**
Config A tốt hơn theo điểm trung bình. Hybrid + rerank thường cải thiện recall vì BM25 giữ lại từ khóa pháp lý/tên riêng, còn dense-only ổn với câu hỏi ngắn nhưng dễ thiếu đúng source khi query có thực thể cụ thể.

---

## Bonus Config: HyDE

| Metric | HyDE query expansion |
|--------|----------------------|
| Faithfulness | 0.989 |
| Answer Relevance | 0.394 |
| Context Recall | 0.643 |
| Context Precision | 0.933 |
| Average | 0.740 |

HyDE tạo một tài liệu giả định từ query, nối với query gốc rồi retrieve trên query mở rộng. Cấu hình này dùng để demo bonus HyDE, đặc biệt hữu ích khi câu hỏi ngắn hoặc thiếu từ khóa chính xác trong tài liệu.

---

## Team Pipeline Benchmark

| Pipeline | Thành viên | Role | Faithfulness | Relevance | Recall | Precision | Average |
|----------|------------|------|--------------|-----------|--------|-----------|---------|
| bach_hybrid_legal | Đào Xuân Bách (2A202600640) | Legal corpus + hybrid retrieval | 0.990 | 0.427 | 0.545 | 0.920 | 0.721 |
| linh_news_bm25 | Đỗ Thiện Lĩnh (2A202600775) | News corpus + BM25 | 0.988 | 0.429 | 0.518 | 0.853 | 0.697 |
| nam_hyde_rag | Lê Hoài Nam (2A202600657) | HyDE + citation generation | 0.989 | 0.394 | 0.643 | 0.933 | 0.740 |
| trung_dense_semantic | Nguyễn Đức Kiên Trung (2A202600769) | Dense semantic retrieval | 0.986 | 0.391 | 0.519 | 0.920 | 0.704 |
| dinh_tfidf_lexical | Nhan Khánh Đình (2A202600673) | TF-IDF lexical bonus | 0.990 | 0.469 | 0.547 | 0.947 | 0.738 |
| anh_fallback_safety | Phan Quốc Anh (2A202600890) | Fallback + safety | 0.790 | 0.395 | 0.450 | 0.787 | 0.606 |

Bảng này chứng minh app nhóm đã tích hợp đủ 6 adapter pipeline, mỗi adapter có owner, role và retrieval focus riêng trong `group_project/pipeline_registry.py`.

---

## Worst Performers (Bottom 3)

| # | Question | Faithfulness | Relevance | Recall | Failure Stage | Root Cause |
|---|----------|-------------|-----------|--------|---------------|------------|
| 1 | Luật Phòng, chống ma túy 2021 nhắc tới các hình thức cai nghiện nào? | 1.000 | 0.083 | 0.167 | Retrieval | Source chưa đủ sát expected_context |
| 2 | Bài VietnamNet về Châu Việt Cường nhắc tới kết quả xét xử nào? | 1.000 | 0.182 | 0.440 | Retrieval | Source chưa đủ sát expected_context |
| 3 | Trong vụ Hữu Tín, lực lượng chức năng phát hiện điều gì? | 0.970 | 0.200 | 0.565 | Retrieval | Source chưa đủ sát expected_context |

---

## Recommendations

### Cải tiến 1
**Action:** Thay dữ liệu mẫu bằng PDF/DOCX và bài báo crawl thật, sau đó rebuild markdown/index.
**Expected impact:** Tăng coverage nguồn và giảm rủi ro manual review về tính xác thực dữ liệu.

### Cải tiến 2
**Action:** Dùng embedding multilingual thật như BAAI/bge-m3 hoặc OpenAI text-embedding-3-small thay cho TF-IDF local.
**Expected impact:** Cải thiện semantic recall với câu hỏi diễn đạt khác từ khóa trong tài liệu.

### Cải tiến 3
**Action:** Dùng LLM judge/DeepEval hoặc RAGAS thật khi có API key, đồng thời thay offline extractive generator bằng GPT/Gemini.
**Expected impact:** Điểm faithfulness/relevance sát thực tế hơn và câu trả lời tự nhiên hơn cho chatbot demo.
