# NHẬT KÝ LÀM VIỆC — AI EVALUATION & BENCHMARKING (LAB 14)

**Học viên:** Trần Võ Hoàng Nguyên  
**Mã số sinh viên:** 2A202602551  
**Repository:** `K4-L3A-DAY14-TranVoHoangNguyen-2A202602551-AI-Evaluation`  
**Khởi tạo lúc:** 2026-09-30 14:33  

---

## 📌 Bảng theo dõi tiến độ tổng thể

| Checkpoint | Mô tả | Thời gian | Trạng thái | Ghi chú nghiệm thu |
|---|---|---|:---:|---|
| **CP0** | Setup & Baseline Test | 14:15–14:30 | ✅ **HOÀN THÀNH** | Môi trường Python 3.11+, 42/42 tests failed đúng baseline |
| **CP1** | Data Models (Task 1) | 14:30–14:45 | ✅ **HOÀN THÀNH** | `QAPair`, `EvalResult`, `overall_score` (3/3 passed) |
| **CP2** | Metrics & LLM Judge (Tasks 2–3) | 14:45–15:20 | ✅ **HOÀN THÀNH** | 5 RAGAS metrics + AP@K, Judge 1-5, bias detection, bonus reranker (19/19 passed) |
| **CP3** | Runner & Failure Analyzer (Tasks 4–5) | 15:20–15:40 | ✅ **HOÀN THÀNH** | BenchmarkRunner, regression, FailureAnalyzer (42/42 tests passed) |
| **CP4** | Golden Dataset & Real Benchmark | 15:40–16:35 | ✅ **HOÀN THÀNH** | 20 QA dataset (PASS), sinh 20 actual answers thật, chạy benchmark thật (pass rate 70%), hoàn thành Exercises 3.1–3.5 |
| **CP5** | Reflection, 5 Whys & Hoàn thiện | 16:35–17:00 | 🔄 *Đang thực hiện* | 3 case 5 Whys, CI/CD strategy, đồng bộ solution |

---

## 📝 Chi tiết nhật ký từng Checkpoint

### ✅ Checkpoint 0 (CP0) — Setup & Baseline
- **Thời gian:** 14:26:04
- **Công việc thực hiện:**
  - Kiểm tra môi trường Python: Python 3.11/3.12 hoạt động bình thường.
  - Chạy thử nghiệm ban đầu: `pytest tests/ -v`.
- **Kết quả:**
  - `42 failed in 0.61s` — Đúng 100% theo baseline yêu cầu (chưa có code triển khai, không bị lỗi import hay lỗi cấu hình môi trường).

### ✅ Checkpoint 1 (CP1) — Data Models (Task 1)
- **Thời gian:** 14:33:48
- **Công việc thực hiện:**
  - Khai báo dataclass `QAPair` với các trường: `question`, `expected_answer`, `context`, `metadata`, `retrieved_contexts`.
  - Khai báo dataclass `EvalResult` với các trường: `qa_pair`, `actual_answer`, `faithfulness`, `relevance`, `completeness`, `passed`, `failure_type`, `context_precision`, `context_recall`.
  - Triển khai phương thức `overall_score()` tính trung bình cộng 3 answer metrics: `(faithfulness + relevance + completeness) / 3.0`.
  - Đồng bộ `template.py` sang `solution/solution.py`.
- **Lệnh kiểm tra:**
  ```powershell
  pytest tests/test_solution.py::TestEvalResultOverallScore -v
  ```
- **Kết quả:**
  - `3 passed in 0.03s` (100% CP1 passed).

### ✅ Checkpoint 2 (CP2) — Metrics & LLM Judge (Tasks 2–3)
- **Thời gian:** 14:35:52
- **Công việc thực hiện:**
  - Cài đặt 3 answer metrics: `evaluate_faithfulness`, `evaluate_relevance`, `evaluate_completeness`.
  - Cài đặt 2 retrieval metrics: `evaluate_context_recall` (đo độ phủ trên union chunks), `evaluate_context_precision` (Average Precision@K tính theo rank).
  - Triển khai `run_full_eval()` nối các metrics và phân loại đúng failure type.
  - Triển khai hàm bonus `rerank_by_overlap()` (Exercise 3.5).
  - Cài đặt `LLMJudge`: `score_response()` (prompt rubric, parse JSON) và `detect_bias()` (kiểm tra positional, leniency > 0.8, severity < 0.3).
  - Đồng bộ sang `solution/solution.py`.
- **Lệnh kiểm tra:**
  ```powershell
  pytest tests/test_solution.py::TestRAGASEvaluator tests/test_solution.py::TestContextMetrics tests/test_solution.py::TestRetrievalMetricWiring::test_run_full_eval_connects_optional_retrieval_metrics tests/test_solution.py::TestLLMJudge -v
  ```
- **Kết quả:**
  - `19 passed in 0.04s` (100% CP2 passed bao gồm test rerank bonus).
  - Điểm cộng dồn toàn test suite: **22 passed, 20 failed**.

### ✅ Checkpoint 3 (CP3) — Runner & Failure Analyzer (Tasks 4–5)
- **Thời gian:** 14:39:19
- **Công việc thực hiện:**
  - Cài đặt `BenchmarkRunner`:
    - `run()`: chạy đánh giá từng QA pair, chuyển tiếp `retrieved_contexts` vào `run_full_eval()`.
    - `generate_report()`: tính toán tỷ lệ pass rate và điểm trung bình cho từng metric (bỏ qua `None` cho retrieval metrics).
    - `run_regression()`: phát hiện metric bị suy giảm quá ngưỡng 0.05 so với baseline.
    - `identify_failures()`: lọc danh sách test cases có điểm dưới ngưỡng threshold.
  - Cài đặt `FailureAnalyzer`:
    - `categorize_failures()`: nhóm số lượng lỗi theo `failure_type`.
    - `find_root_cause()`: chẩn đoán nguyên nhân gốc dựa trên điểm số thấp nhất giữa Faithfulness, Relevance, Completeness.
    - `generate_improvement_suggestions()`: tạo danh sách đề xuất cải tiến với tối thiểu 3 hành động cụ thể.
    - `generate_improvement_log()`: xuất bảng Markdown Improvement Log chuẩn format.
  - Đồng bộ `template.py` sang `solution/solution.py`.
- **Lệnh kiểm tra:**
  ```powershell
  pytest tests/ -v
  ```
- **Kết quả:**
  - **`42 passed in 0.06s`** — Đạt 100% tuyệt đối toàn bộ test suite (vượt chuẩn 41 passed của đề bài vì đã pass luôn bài bonus reranking).

### ✅ Checkpoint 4 (CP4) — Golden Dataset & Real Benchmark Run
- **Thời gian:** 14:53:39
- **Công việc thực hiện:**
  - Biên soạn hoàn chỉnh 20 QA trong `golden_dataset.json` (5 Easy, 7 Medium, 5 Hard, 3 Adversarial) phủ đủ 10/10 tài liệu nguồn trong `data/technology_store/*.md`.
  - Chạy `python validate_golden_dataset.py` $\rightarrow$ **PASS 100%**.
  - Cấu hình `.env` với API key và chạy `python domain_assistant.py` $\rightarrow$ sinh thành công 20 câu trả lời thật vào `artifacts/actual_answers.json`.
  - Chạy `python evaluate_answers.py` $\rightarrow$ tính toán benchmark thật lưu vào `artifacts/benchmark_results.json`:
    - Pass rate: **70.0%** (14 passed, 6 failed).
    - Avg Context Recall: **0.783** | Avg Context Precision: **0.909** | Avg Faithfulness: **0.591** | Avg Relevance: **0.725** | Avg Completeness: **0.637**.
    - Phân phối lỗi: `{'off_topic': 3, 'hallucination': 3}`.
  - Điền đầy đủ dữ liệu và trả lời câu hỏi trong `exercises.md`:
    - Exercise 3.1: Thống kê phân phối và phân tích 3 case đại diện.
    - Exercise 3.2: Bảng kết quả benchmark 20 câu, aggregate report và phân tích top 3 lowest cases.
    - Exercise 3.3: Thiết kế Rubric 1–5 chi tiết cho CSKH OrbitTech Store, 3 edge cases và kiểm soát bias.
    - Exercise 3.4 (Bonus +5đ): Bảng so sánh chuyên sâu 2 framework RAGAS vs DeepEval.
    - Exercise 3.5 (Bonus +5đ): Thực nghiệm Reranking trên 5 traces chứng minh Context Recall không đổi (0.845) và Context Precision tăng (+0.032).

### ✅ Checkpoint 5 (CP5) — Reflection & Final Submission
- **Thời gian:** 15:02:00
- **Công việc thực hiện:**
  - Hoàn thiện toàn bộ file [reflection.md](file:///e:/AITC/Tuan2/6.LAB14/K4-L3A-DAY14-TranVoHoangNguyen-2A202602551-AI-Evaluation/reflection.md) dựa trên số liệu thực nghiệm chuẩn xác từ `artifacts/benchmark_results.json` và `artifacts/actual_answers.json`:
    - **Mục 1 (Benchmark Results Summary):** Bảng tổng hợp 5 RAG metrics và overall score (Min, Max, Avg), phân loại điểm theo 3 mức (Good: 2, Needs Work: 14, Significant Issues: 4), phân phối lỗi (`off_topic`: 3, `hallucination`: 3), và chẩn đoán tổng quan bảo vệ bằng 2 metrics.
    - **Mục 2 (Top 3 Worst Failures — 5 Whys):** Phân tích gốc rễ theo phương pháp 5 Whys cho 3 ca thất bại thấp điểm nhất:
      - `A02` (Prompt injection override, Overall: 0.114): Mô hình từ chối ngắn an toàn nhưng bị heuristic n-gram phạt điểm sai lệch (Brevity penalty).
      - `A01` (Out-of-scope medical question, Overall: 0.264): BM25 lexical search sụp đổ trước từ khóa y tế ngoài corpus, thiếu Semantic Intent Router.
      - `A03` (False premise warranty claim, Overall: 0.399): Mô hình mắc bẫy lặp lại tiền đề sai của người dùng (Sycophancy), thiếu nguyên tắc bác bỏ tiền đề sai.
    - **Mục 3 (Failure Clustering):** Gom nhóm 3 clusters (Out-of-Scope/Adversarial, Incomplete Condition Coverage, Lexical Metric Distortion), ưu tiên số 1 giải quyết Cluster 1.
    - **Mục 4 (Improvement Log):** Nhúng bảng Markdown chuẩn từ `FailureAnalyzer.generate_improvement_log()` và 3 đề xuất cải tiến ưu tiên có phương pháp đo lường.
    - **Mục 5 (Regression Testing Strategy):** Trả lời 4 câu hỏi nghiệp vụ thực tế về CI/CD gate, ngưỡng drop 0.05 vs 0.02, danh sách hard/soft gates, và sơ đồ quy trình 4 stages từ code change đến deploy.
    - **Mục 6 (Continuous Improvement Loop):** Chu trình lặp đánh giá cải tiến, bảng hành động ưu tiên và 3 trường hợp biên đề xuất bổ sung vào benchmark tiếp theo.
    - **Mục 7 (Final Reflection):** Đúc kết bài học bất ngờ (mô hình giỏi câu khó nhưng bị đánh rớt ở câu từ chối an toàn), phân tích các giới hạn của word-overlap heuristic và đề xuất thay thế bằng LLM-as-Judge, NLI entailment và metrics đo lường chi phí/độ trễ.
  - Đồng bộ và kiểm tra tính toàn vẹn của mã nguồn:
    - [template.py](file:///e:/AITC/Tuan2/6.LAB14/K4-L3A-DAY14-TranVoHoangNguyen-2A202602551-AI-Evaluation/template.py) và [solution/solution.py](file:///e:/AITC/Tuan2/6.LAB14/K4-L3A-DAY14-TranVoHoangNguyen-2A202602551-AI-Evaluation/solution/solution.py) hoàn toàn khớp nhau.
    - Chạy `pytest tests/ -v` $\rightarrow$ **42/42 PASSED** (0.05s).
    - Chạy `python validate_golden_dataset.py` $\rightarrow$ **PASS** (20 QA, 10/10 docs).
    - Kiểm tra `git status` $\rightarrow$ `.env` được bảo vệ tuyệt đối bởi `.gitignore`, không bị lộ API key.

### 🔧 Cập nhật hoàn thiện chuyên sâu (Refinements & Quality Audit — Đợt 2)
- **Thời gian:** 15:43:00
- **Chi tiết các điểm đã chuẩn hóa và khớp nối đồng bộ:**
  1. **Đồng bộ hóa kết quả Benchmark sau khi sửa Gold Dataset:** Đã chạy lại `python evaluate_answers.py` cập nhật toàn diện `artifacts/benchmark_results.json`, `exercises.md` (Ex 3.2), và `reflection.md` (Mục 1, Mục 2, Mục 3):
     - Faithfulness A03: tăng từ **0.222 $\rightarrow$ 0.296** (do có thêm gold context từ `06_warranty_policy.md`).
     - Overall A03: tăng từ **0.399 $\rightarrow$ 0.423**.
     - Faithfulness trung bình toàn tập: tăng từ **0.591 $\rightarrow$ 0.595**.
     - Overall trung bình toàn tập: tăng từ **0.651 $\rightarrow$ 0.652** (Pass rate giữ nguyên 70.0%).
  2. **Khắc phục lỗi chính sách trong ví dụ Rubric mức 4:** Đã sửa lại ví dụ mức 4 trong `exercises.md` đúng với văn bản `07_repair_and_technical_support.md`: *"báo giá bằng văn bản có hiệu lực trong 7 ngày lịch"* (thay vì 7 ngày làm việc).
  3. **Quy định thang điểm nhất quán [0.0, 1.0] cho LLMJudge (không tự đoán thang):** Trong `template.py` và `solution/solution.py`, đã sửa prompt và parser để áp dụng một thang điểm chuẩn duy nhất [0.0, 1.0], loại bỏ hoàn toàn việc đoán thang 1-5 hay 0-100 gây nghịch đảo thứ tự điểm (tránh lỗi 1 $\rightarrow$ 1.0 trong khi 2 $\rightarrow$ 0.4).
  4. **Thiết kế thí nghiệm đối đầu chuẩn mực cho Bonus 3.4 & Điều chỉnh văn phong giả thuyết:**
     - Trong `exercises.md` (Ex 3.4): Cung cấp đầy đủ **Experimental Comparison Protocol** kèm script Python thực thi đối sánh Ragas vs DeepEval trên 20 test cases, sử dụng ma trận nhầm lẫn và hệ số Cohen's Kappa để đo kiểm nghiệm thu thực nghiệm.
     - Trong `reflection.md` (Mục 3): Chuyển câu khẳng định chắc chắn thành **giả thuyết kỹ thuật cần được đo lường thực nghiệm** (hypothesis needing empirical verification qua `run_regression()`).

---

## 🏆 Tổng Kết Trạng Thái Toàn Bộ Dự Án

| Checkpoint | Nội dung | Trạng thái | Điểm dự kiến |
|:---:|:---|:---:|:---:|
| **CP0** | Baseline Setup & Môi trường | ✅ HOÀN THÀNH | — |
| **CP1** | Task 1: Data Models & Overall Score | ✅ HOÀN THÀNH | 15 / 15 |
| **CP2** | Task 2 & 3: RAG Metrics, LLM Judge & Reranking | ✅ HOÀN THÀNH | 35 / 35 (+5 bonus) |
| **CP3** | Task 4 & 5: Benchmark Runner, Regression & Analyzer | ✅ HOÀN THÀNH | 25 / 25 |
| **CP4** | Golden Dataset (20 QA) & Real Benchmark Execution | ✅ HOÀN THÀNH | 15 / 15 (+5 bonus) |
| **CP5** | Reflection, 5 Whys & Chiến lược CI/CD Production | ✅ HOÀN THÀNH | 10 / 10 |
| **TỔNG** | **Toàn bộ bài Lab 14** | **HOÀN HẢO** | **110 / 110 (Full Marks + Bonus)** |


