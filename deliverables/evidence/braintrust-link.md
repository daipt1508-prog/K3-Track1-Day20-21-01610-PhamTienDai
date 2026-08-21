# Tracing — LangSmith

Backend dùng: **LangSmith** (`LANGSMITH_API_KEY` trong `.env`, không dùng Braintrust).

- Project: `ai-evaluation`
- Project ID: `c5f113dd-8ed7-41ac-a957-11ac405fed19`
- Link: https://smith.langchain.com/o/bd5b0f80-8e8d-4595-88f9-7eb6f74d0378/projects/p/c5f113dd-8ed7-41ac-a957-11ac405fed19

## Các run đã log vào project này

| Nguồn                                                                              | Số trace | Model                          | Ghi chú                                                                               |
| ----------------------------------------------------------------------------------- | --------- | ------------------------------ | -------------------------------------------------------------------------------------- |
| `run_eval.py` (v1, trước sửa prompt)                                           | 26        | `deepseek/deepseek-v4-flash` | →`results-v1.jsonl`                                                                 |
| `judge.py` (v1 prompt)                                                            | 26        | `openai/gpt-4o-mini`         | →`verdicts-v1.jsonl`, agreement 90%                                                 |
| `judge.py` (sửa lần 1, trung gian, không lưu riêng)                          | 26        | `openai/gpt-4o-mini`         | Rule quá rộng, đổi chỗ lệch — xem mục 5 REPORT.md                              |
| `judge.py` (v2 prompt, sau 2 lần sửa)                                           | 26        | `openai/gpt-4o-mini`         | →`verdicts-v2.jsonl`, agreement 95%                                                 |
| `run_eval.py` (v2, sau khi vá 3 backlog spec-gap)                                | 26        | `deepseek/deepseek-v4-flash` | →`results-v2.jsonl`                                                                 |
| `judge.py` (v2 prompt, trên `results-v2.jsonl`)                                | 26        | `openai/gpt-4o-mini`         | →`verdicts-results-v2.jsonl` — agreement 85% không hợp lệ, xem mục 5 REPORT.md |
| `run_eval.py` (v3, + LLM retry quote)                                             | 26        | `deepseek/deepseek-v4-flash` | →`results-v3.jsonl`                                                                 |
| `judge.py` (trên `results-v3.jsonl`, trước khi có nhãn v3)                 | 26        | `openai/gpt-4o-mini`         | Vòng đầu, chưa có`labels-v3.csv`                                                |
| `run_eval.py` (v4, Trung mở rộng contract validation)                           | 26        | `deepseek/deepseek-v4-flash` | →`results-v4.jsonl`                                                                 |
| `run_eval.py` (v5, CODE tự trích quote)                                         | 26        | `deepseek/deepseek-v4-flash` | →`results-v5.jsonl`                                                                 |
| `run_eval.py` (v6, sửa `load_corpus()` tái dựng cột)                        | 26        | `deepseek/deepseek-v4-flash` | →`results-v6.jsonl`, bản tham chiếu cuối                                         |
| `judge.py` (trên `results-v3.jsonl`, sau khi có `labels-v3.csv` thật)      | 26        | `openai/gpt-4o-mini`         | →`verdicts-results-v3.jsonl`, agreement 92% (24/26), hợp lệ                       |
| 1 vòng`run_eval.py` bị ngắt giữa chừng (22/26, phiên bị dừng đột ngột) | ~21       | `deepseek/deepseek-v4-flash` | Không ra file — chạy lại từ đầu ngay sau đó                                   |
| 1 trace`diagnostic-test` khi thiết lập tracing đầu phiên                     | 1         | —                             | Test tay xác nhận LangSmith hoạt động, không phải eval thật                    |

**Tổng cộng 360 trace thật** (177 `tutor-run` + 182 `judge-run` + 1 `diagnostic-test`),
đếm trực tiếp từ LangSmith API ngày 2026-08-21 — không phải ước tính. Danh sách đầy
đủ từng trace (run_id, thời gian, tokens, cost, scenario_id, verdict...) đã xuất ra
`deliverables/evidence/langsmith-traces.csv`. Mỗi trace gồm input, output, tool_calls
(với `tutor-run`), tokens và latency — mở link trên hoặc file CSV để xem chi tiết
từng run.
