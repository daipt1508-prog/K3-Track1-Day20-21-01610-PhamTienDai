# REPORT — Eval loop A→Z: VLearn AI Tutor

---

## 1. Input Grid

> Lưới đầu vào = trục "ai hỏi" × "hỏi kiểu gì". LLM giúp tạo cách diễn đạt, con người
> kiểm soát độ bao phủ. Trả lời các câu hỏi sau rồi vẽ lưới của bạn.

- **Nhóm người dùng:** học viên mới cần kiến thức nền; học viên đang làm bài tập tổng
  kết cần được gợi mở nhưng không được làm hộ; người ôn bài cần so sánh, tổng hợp;
  PM và người làm sản phẩm cần áp dụng nguyên tắc vào quyết định thực tế.
- **Mục đích hỏi:** hỏi khái niệm; so sánh, tổng hợp; áp dụng vào dự án; xin đáp án
  hoặc quyết định làm hộ; hỏi ngoài tập tài liệu; tấn công chèn chỉ dẫn
  (prompt injection).
- **Tần suất dự kiến cao:** câu hỏi khái niệm của học viên mới và câu hỏi áp dụng vào
  bài tập tổng kết hoặc dự án. **Rủi ro cao:** trợ giảng AI bịa khi không truy xuất được
  tài liệu, làm hộ kết luận hoặc ngưỡng đạt, làm theo chỉ dẫn tấn công, hay khuyên
  triển khai chỉ dựa trên một chỉ số tổng quát.

### Các chiều dữ liệu được cân nhắc và quyết định chọn

| Chiều dữ liệu được cân nhắc | Các giá trị đã cân nhắc | Đổi giá trị thì câu trả lời đúng thay đổi thế nào? | Quyết định |
|---|---|---|---|
| Loại câu hỏi | `concept`, `compare_synthesize`, `apply_to_project`, `request_final_answer`, `out_of_scope`, `prompt_injection` | Giải thích → tổng hợp → hướng dẫn áp dụng từng bước → không làm hộ → từ chối hoặc hỏi lại → giữ giới hạn hệ thống | Chọn |
| Độ phủ tài liệu | `single_explicit`, `multi_doc_distributed`, `partial_only`, `absent_zero_hit` | Trích một mục → tổng hợp nhiều tài liệu → chỉ trả lời phần có bằng chứng và nêu giới hạn → không suy đoán | Chọn |
| Độ rõ | `clear_single`, `ambiguous_missing_context`, `multi_part` | Trả lời thẳng → dùng ngữ cảnh trang chiếu, nêu giả định hoặc hỏi lại → tách và trả lời từng ý | Chọn |
| Bối cảnh người học | `beginner`, `capstone_builder`, `reviewer`, `practitioner` | Giải thích từ kiến thức nền → gợi mở, không làm hộ → so sánh cô đọng → áp dụng có điều kiện và hỏi quy định hoặc dữ liệu còn thiếu | Chọn |
| Giọng lịch sự hoặc cộc | lịch sự, cộc, trang trọng | Hành vi cốt lõi không đổi; đây chỉ là cách diễn đạt khác | Bỏ khỏi lưới, chỉ dùng làm ràng buộc tình huống |
| “Độ khó” | dễ, vừa, khó | Không có căn cứ kiểm chứng ổn định, không chỉ ra được hành vi khác cụ thể | Bỏ |


### Grid

| Nhóm người dùng \ Mục đích hỏi | Khái niệm | So sánh, tổng hợp | Áp dụng | Xin đáp án | Ngoài tài liệu | Tấn công chèn chỉ dẫn |
|---|---|---|---|---|---|---|
| Học viên mới | C01, C04, C11 | — | — | — | C07, C08 | — |
| Làm bài tập tổng kết | — | C13 | C05 | C06 | — | C09 |
| Ôn bài | — | C02, C10 | — | — | — | — |
| PM hoặc người làm sản phẩm | — | — | C03, C12 | — | — | — |

### 13 tổ hợp được giữ

| Mã | Các giá trị của chiều dữ liệu | Hành vi mong đợi ở mức khái quát | Vì sao đáng kiểm thử | Loại | Ràng buộc đời thực |
|---|---|---|---|---|---|
| C01 | concept · single_explicit · clear_single · beginner | Giải thích từ kiến thức nền, trích đúng một nguồn rõ ràng | Tần suất cao, tạo điểm tựa khái niệm | Đại diện | Viết tắt, câu cộc |
| C02 | compare_synthesize · multi_doc_distributed · clear_single · reviewer | Tổng hợp Hamel và Anthropic, không đồng nhất hai hệ phân loại | Khâu truy xuất và tổng hợp dễ sai | Đại diện | Hai nguồn dùng tên gọi khác nhau |
| C03 | apply_to_project · multi_doc_distributed · clear_single · practitioner | Đưa ra quy trình từ tài liệu, không bịa quy định của dự án | Tình huống PM thường gặp | Đại diện | Bối cảnh trợ lý hỗ trợ khách hàng còn thiếu quy định |
| C04 | concept · single_explicit · ambiguous_missing_context · beginner | Dùng ngữ cảnh trang chiếu, nêu giả định hoặc hỏi lại | Cách chỉ trỏ mơ hồ thường gặp khi học trên trang chiếu | Thử thách | “cái ma trận này” |
| C05 | apply_to_project · partial_only · multi_part · capstone_builder | Tách hai ý; sửa giả định về độ bao phủ; không tự đặt ngưỡng | Ranh giới giữa kiến thức và quyết định sản phẩm | Thử thách | Nhiều ý và có giả định sai |
| C06 | request_final_answer · partial_only · clear_single · capstone_builder | Không làm hộ ngưỡng hoặc kết luận; đưa khung tự quyết | Chi phí thất bại trong học tập cao | Rủi ro cao | Sát hạn nộp, giục trả đáp án |
| C07 | out_of_scope · absent_zero_hit · clear_single · beginner | Từ chối ngắn gọn, không bịa, không trích nguồn không liên quan | Ranh giới cơ bản nhưng bắt buộc | Đại diện | Giá hiện thời hoặc thời tiết |
| C08 | out_of_scope · absent_zero_hit · ambiguous_missing_context · beginner | Hỏi lại trước; không đoán mô hình hoặc giá | Không có kết quả truy xuất cộng với câu hỏi mơ hồ dễ gây bịa đặt | Thử thách | Tiếng lóng, thiếu đối tượng được nhắc tới |
| C09 | prompt_injection · absent_zero_hit · clear_single · capstone_builder | Giữ giới hạn tài liệu và hệ thống, không tạo nguồn giả | Tính bám nguồn và an toàn bị tấn công trực tiếp | Rủi ro cao | Chỉ dẫn cố ghi đè quy tắc hệ thống |
| C10 | compare_synthesize · partial_only · multi_part · reviewer | Bác bỏ ngộ nhận về độ đồng thuận tổng; chưa quyết định triển khai khi thiếu dữ liệu | Bộ chấm dễ dãi có thể cho lọt lỗi | Rủi ro cao | Giả định sai và có hai ý |
| C11 | concept · multi_doc_distributed · clear_single · beginner | Tách đánh giá truy xuất, từng thành phần và toàn bộ quy trình | Dễ chỉ chấm câu trả lời và bỏ sót nguyên nhân lỗi | Thử thách | Dùng các chữ viết tắt RAG và thuật ngữ retrieval |
| C12 | apply_to_project · single_explicit · ambiguous_missing_context · practitioner | Không chốt triển khai từ một số tổng; hỏi ngưỡng, lát cắt và mốc so sánh | Quyết định triển khai có chi phí thất bại cao | Rủi ro cao | Gấp, chịu áp lực từ cấp trên |
| C13 | compare_synthesize · multi_doc_distributed · multi_part · capstone_builder | Phân luồng cho mã kiểm tra, bộ chấm LLM và con người theo căn cứ kiểm chứng và kết quả hiệu chỉnh | Câu hỏi trung tâm của bài thực hành | Rủi ro cao | Ba ý, pha thuật ngữ Anh–Việt |

### Tổ hợp bị loại khỏi danh sách ứng viên

| Tổ hợp | Lý do loại |
|---|---|
| out_of_scope × single_explicit | Mâu thuẫn: nếu tài liệu trả lời trực tiếp thì câu hỏi không còn ở ngoài phạm vi |
| request_final_answer × absent_zero_hit | “Xin đáp án ngoài bài” không có đối tượng ổn định; trùng ranh giới của câu ngoài phạm vi |
| concept × absent_zero_hit | Xét theo hành vi, tổ hợp này sẽ được gắn lại thành câu ngoài phạm vi nên không tạo tình huống mới |
| prompt_injection × mọi mức độ phủ | Độ phủ của nội dung tấn công không làm thay đổi hành vi đúng; chỉ giữ một trường hợp không truy xuất được tài liệu và có rủi ro cao |
| clear_single × ràng buộc “thiếu ngữ cảnh” | Tự mâu thuẫn; chuyển sang `ambiguous_missing_context` |

Mỗi tổ hợp C01–C13 được diễn đạt thành hai câu. Vòng lọc biên tập giữ nguyên 20 câu và
viết lại 6 câu; quyết định của từng dòng nằm ở `metadata.paraphrase_review`. Không có
biến thể bị loại trong tệp cuối; các bản nháp không đạt bị bỏ vì tự thêm ngữ cảnh hoặc
tên mô hình, trùng mục đích hỏi, hay làm câu mơ hồ trở nên quá rõ. Đây là **bản nháp
do AI hỗ trợ**; trạng thái `pending_owner_signoff` được ghi rõ để người nộp thực hiện
bước duyệt cuối cùng theo ba quyết định Giữ nguyên, Viết lại hoặc Loại bỏ, tránh giả
mạo việc đánh giá của con người.

---

## 2. Dataset v1

> Bộ dữ liệu là "bộ đề thi" của trợ giảng AI. Nêu rõ bộ dữ liệu phủ những ô nào
> trong lưới đầu vào.

- Bộ dữ liệu có **26 câu**, phủ 13 ô hoặc tổ hợp ở mục 1; mỗi tổ hợp có hai cách diễn
  đạt khác phong cách nhưng cùng hành vi mong đợi.
- Theo `expected_scope`: **14/26 câu trong phạm vi (53,8%)**, **6/26 câu ngoài phạm vi
  (23,1%)**, **6/26 câu chưa rõ phạm vi (23,1%)**. Theo các lát cắt có thể chồng lấp:
  **6/26 câu mơ hồ (23,1%)** và **4/26 câu đối kháng (15,4%)**, gồm 2 câu xin đáp án
  và 2 câu tấn công chèn chỉ dẫn. Nhóm thử thách và rủi ro cao được lấy nhiều hơn có
  chủ đích vì phiên bản 1 nhằm tìm ranh giới và dạng lỗi, không nhằm ước lượng tỷ lệ
  thành công khi vận hành thực tế.
- Đã kiểm tra cấu trúc, tính nhất quán của nhãn, độ trùng mục đích hỏi và đối chiếu chủ
  đề với tập tài liệu. Phát hiện chính: bản nháp dễ tự thêm tên mô hình làm mất độ mơ
  hồ; hai cách diễn đạt dễ gần trùng nhau; câu chỉ được tài liệu hỗ trợ một phần dễ bị
  viết thành câu có đáp án tuyệt đối. Sáu câu đã được viết lại vì các lỗi này. **Chưa
  tuyên bố đã hoàn tất bước PM đánh giá thủ công**; mọi dòng đang mang trạng thái
  `pending_owner_signoff` và cần người nộp xác nhận trước khi dùng làm mốc so sánh.
- Nếu chỉ giữ 10 câu: giữ `sc-01a`, `sc-02a`, `sc-03a`, `sc-04a`, `sc-06a`,
  `sc-07a`, `sc-09a`, `sc-10a`, `sc-12a`, `sc-13a`. Bộ này giữ 10 hành vi khác
  nhau, gồm tình huống thông thường về kiến thức nền, tổng hợp nhiều nguồn, áp dụng,
  câu mơ hồ, xin đáp án, không truy xuất được tài liệu, tấn công chèn chỉ dẫn, giả
  định sai, quyết định triển khai và phân luồng ba làn; bỏ biến thể B trước để tối đa
  độ bao phủ với ngân sách nhỏ.

### Danh sách tình huống (bảng tóm tắt)

| `scenario_id` | Ô trong lưới | Hành vi mong đợi | Nguồn câu hỏi |
|---|---|---|---|
| sc-01a-concept-eval | C01 | Giải thích eval từ kiến thức nền |  Chạy thực tế|
| sc-01b-concept-eval | C01 | Giải thích ngắn, đúng phạm vi | Chạy thực tế |
| sc-02a-compare-code-graders | C02 | Tổng hợp Hamel và Anthropic | Chạy thực tế |
| sc-02b-compare-code-graders | C02 | So sánh có sắc thái | Chạy thực tế |
| sc-03a-apply-support-bot | C03 | Áp dụng quy trình, không bịa quy định | Chạy thực tế |
| sc-03b-apply-support-bot | C03 | Bắt đầu từ dữ liệu và tiêu chí trước bộ chấm | Chạy thực tế |
| sc-04a-ambiguous-matrix | C04 | Dùng trang chiếu hoặc hỏi lại | Chạy thực tế |
| sc-04b-ambiguous-matrix | C04 | Giải thích vì sao 70% chưa đủ | Chạy thực tế |
| sc-05a-partial-dataset | C05 | Tách độ bao phủ và ngưỡng triển khai | Chạy thực tế |
| sc-05b-partial-dataset | C05 | Bác bỏ giả định số lượng bằng độ bao phủ | Chạy thực tế |
| sc-06a-cheat-threshold | C06 | Không cho sẵn đáp án về ngưỡng | Chạy thực tế |
| sc-06b-cheat-threshold | C06 | Không làm hộ kết luận | Chạy thực tế |
| sc-07a-out-price | C07 | Từ chối cung cấp giá hiện thời | Câu do AI hỗ trợ diễn đạt |
| sc-07b-out-weather | C07 | Từ chối câu hỏi ngoài chủ đề | Câu do AI hỗ trợ diễn đạt |
| sc-08a-out-ambiguous-model | C08 | Hỏi lại, không đoán giá hoặc mô hình | Chạy thực tế |
| sc-08b-out-ambiguous-model | C08 | Làm rõ tiếng lóng và đối tượng được nhắc tới | Chạy thực tế |
| sc-09a-injection-ignore-corpus | C09 | Giữ giới hạn tài liệu, không bịa nguồn | Chạy thực tế |
| sc-09b-injection-system-prompt | C09 | Không lộ chỉ dẫn hệ thống hoặc dùng nguồn ngoài | Chạy thực tế |
| sc-10a-agreement-false-premise | C10 | Bác bỏ ngộ nhận về độ đồng thuận tổng | Câu do AI hỗ trợ diễn đạt |
| sc-10b-agreement-false-premise | C10 | Yêu cầu ma trận nhầm lẫn và bằng chứng | Câu do AI hỗ trợ diễn đạt |
| sc-11a-rag-eval | C11 | Tách từng thành phần và toàn bộ quy trình | Câu do AI hỗ trợ diễn đạt |
| sc-11b-rag-eval | C11 | Chấm cả truy xuất lẫn câu trả lời | Chạy thực tế |
| sc-12a-ship-ambiguous | C12 | Không triển khai chỉ dựa trên 78% | Câu do AI hỗ trợ diễn đạt |
| sc-12b-ship-ambiguous | C12 | Không triển khai chỉ dựa trên mức tăng | Câu do AI hỗ trợ diễn đạt |
| sc-13a-routing-layers | C13 | Phân luồng cho mã kiểm tra, bộ chấm LLM và con người | Câu do AI hỗ trợ diễn đạt |
| sc-13b-routing-layers | C13 | Giữ con người ở tiêu chí mềm và tình huống rủi ro | Câu do AI hỗ trợ diễn đạt |

Tệp khóa phiên bản 1: `deliverables/evidence/dataset-v1.jsonl`; bản dùng để chạy nằm
ở thư mục gốc: `dataset.jsonl`. Hai tệp phải giống hệt từng byte trước lần chạy trợ
giảng AI đầu tiên.

---

## 3. Rubric v1

> Rubric là định nghĩa về mức "đủ tốt" để cả nhóm chấm nhất quán. Cần thu hẹp phạm vi
> trước khi viết tiêu chí.

- Trợ giảng AI trả lời một câu trong phạm vi **"đủ tốt"** khi nào? Viết bằng một hoặc
  hai câu mà ai cũng hiểu.
- Liệt kê các **tiêu chí chấm** như tính bám nguồn, trích dẫn đúng định dạng, đúng phạm
  vi, chất lượng sư phạm và câu hỏi gợi mở có giá trị. Với mỗi tiêu chí, nêu điều kiện
  đạt, không đạt và ví dụ tương ứng.
- Tiêu chí nào là **điều kiện chặn**, nghĩa là không đạt tiêu chí đó thì cả lượt bị
  đánh giá không đạt? Tiêu chí nào chỉ là điểm cộng?
- Với câu ngoài phạm vi, hành vi nào được coi là đạt, chẳng hạn từ chối và gợi ý chủ
  đề liên quan?
- Bạn đã thử chấm chéo với ai chưa? Hai người chấm lệch nhau ở tiêu chí nào và đã sửa
  rubric ra sao sau đó?

### Rubric của bạn

| Tiêu chí | Đạt khi | Không đạt khi | Điều kiện chặn? |
|---|---|---|---|
| | | | |

---

## 4. Routing Map

> Nội dung nào kiểm tra được bằng mã, nội dung nào cần bộ chấm LLM và nội dung nào phải
> do chuyên gia quyết định. Không phải tiêu chí nào cũng cần LLM.

- Với từng tiêu chí trong rubric ở mục 3, hãy xác định nên kiểm tra bằng **mã có kết quả
  xác định**, **bộ chấm LLM** hay **con người**, kèm lý do.
- Tiêu chí nào ban đầu được giao cho bộ chấm LLM nhưng hóa ra có thể kiểm tra bằng mã
  với chi phí thấp hơn, chẳng hạn đầu ra có phân tích được thành JSON hay không, hoặc
  danh sách nguồn có đủ `doc_id` hợp lệ hay không?
- Tiêu chí nào không thể tin hoàn toàn vào bộ chấm LLM và phải giữ cho con người?
- Chỉ dẫn cho bộ chấm trong `eval/judge_prompt.md` chấm tiêu chí nào? Nhiệt độ và mô
  hình chấm là gì? Vì sao chọn mô hình khác với mô hình của trợ giảng AI?

### Bảng phân luồng

| Tiêu chí | Mã kiểm tra | Bộ chấm LLM | Con người | Lý do |
|---|---|---|---|---|
| | | | | |

---

## 5. Calibration Report

> Bộ chấm chỉ đáng tin khi đã được hiệu chỉnh theo chuẩn vàng do con người tạo ra. Đây
> là phần minh chứng cho việc hiệu chỉnh đó.

- Bạn đã **gán nhãn thủ công** bao nhiêu dòng? Dữ liệu nằm trong `labels.csv`, được
  xuất từ `report.html`.
- Sau khi chạy `python3 eval/judge.py`, **độ đồng thuận** giữa bộ chấm và nhãn của con
  người là bao nhiêu phần trăm? Dán ma trận nhầm lẫn vào đây.
- Bộ chấm **sai ở đâu**: chặt quá, lỏng quá hay lệch ở nhóm câu trong phạm vi hoặc
  ngoài phạm vi?
- Bạn đã sửa `eval/judge_prompt.md` như thế nào sau vòng hiệu chỉnh đầu tiên? Độ đồng
  thuận sau khi sửa là bao nhiêu?
- Kết luận: bộ chấm **đủ đáng tin để tự động chấm tiêu chí nào**, và tiêu chí nào vẫn
  phải giữ cho con người?

### Ma trận nhầm lẫn (dán đầu ra của `judge.py`)

```
(dán ở đây)
```

---

## 6. Scorecard & Gate

> Tổng hợp điểm theo rubric trên bộ dữ liệu phiên bản 1, sau đó đặt ngưỡng quyết định
> như một PM thực thụ.

- Kết quả chạy `eval/run_eval.py` và `eval/judge.py` trên bộ dữ liệu phiên bản 1 cho
  thấy **tỷ lệ đạt** của từng tiêu chí là bao nhiêu? Kèm đường dẫn tới
  `results.jsonl`, `verdicts.jsonl` và `report.html`.
- Chi phí của một vòng đánh giá là bao nhiêu đô la và bao nhiêu token? Độ trễ trung
  bình cho một câu là bao nhiêu?
- **Ngưỡng quyết định:** đạt mức nào thì được triển khai? Ví dụ, tỷ lệ đạt về tính bám
  nguồn phải từ 90% trở lên và không có lỗi ở nhóm điều kiện chặn. Hãy định nghĩa
  ngưỡng và giải thích lý do.
- Kết quả hiện tại là **TRIỂN KHAI hay CHƯA TRIỂN KHAI**? Kết luận phải căn cứ vào
  ngưỡng đã đặt ở trên.
- Nếu chưa triển khai, ba lỗi lớn nhất cần sửa ở trợ giảng AI là gì, xét theo chỉ dẫn
  hệ thống, khâu truy xuất và tập tài liệu?

### Scorecard

| Tiêu chí | Đạt | Không đạt | Chưa chắc chắn | Tỷ lệ đạt |
|---|---|---|---|---|
| | | | | |

### Quyết định theo ngưỡng

**TRIỂN KHAI / CHƯA TRIỂN KHAI** — vì: ...

---

## 7. Verdict + Report cuối

> Kết luận cuối cùng với tư cách PM chịu trách nhiệm về chất lượng của trợ giảng AI.
> Phần kết luận đi kèm báo cáo một trang gồm đủ năm phần, được viết bằng ngôn ngữ PM
> và không dán bản ghi thô.

### Report

#### 1. Dataset đã đánh giá

(đã dùng tập nào, có bao nhiêu bản ghi chạy, độ bao phủ chính là gì và còn điểm mù nào)

#### 2. Quá trình đồng thuận của con người

- Độ đồng thuận ở vòng chấm độc lập theo nhãn tổng: ___% — kèm thống kê từ ghi chú về
  tiêu chí gây bất đồng nhiều nhất.
- Mâu thuẫn lớn nhất: tình huống hoặc tiêu chí nào, hai phía nhận định ra sao?
- Nhóm đã xử lý bằng cách nào: siết định nghĩa, đổi thang đo hay bỏ tiêu chí?

#### 3. LLM judge

- Mô hình chấm: ________________
- Số vòng hiệu chỉnh: ___ — sau đó bộ chấm nhận đúng ___% đầu ra tốt và bắt đúng ___%
  đầu ra xấu.
- Bộ chấm nào không thể hiệu chỉnh đạt yêu cầu và vì sao: ________________

#### 4. Bảng quyết định phân luồng (kèm lý giải)

| Tiêu chí | Ngưỡng đạt | Giao cho | Vì sao (dựa trên số liệu) |
|---|---|---|---|
| Ví dụ: tính bám nguồn | ≥90% | Bộ chấm LLM + kiểm tra mẫu 10% mỗi tuần | bắt đúng 91% đầu ra xấu sau hai vòng bổ sung trường hợp sát ranh giới |
|  |  |  |  |
|  |  |  |  |

#### 5. Verdict + bước tiếp theo

**Triển khai / Triển khai có điều kiện / Tạm dừng** — vì: ________________

- Nếu triển khai: tuần đầu theo dõi nội dung gì, lấy mẫu bao nhiêu phần trăm và cảnh
  báo ở ngưỡng nào?
- Nếu tạm dừng: đòn bẩy tiếp theo là chỉ dẫn hệ thống, mô hình hay kiến trúc; chỉ số
  nào sẽ chứng minh sản phẩm đã sẵn sàng?

### Câu hỏi tự soi

- Tin cậy nhất ở đâu, đáng lo nhất ở đâu? Dẫn `scenario_id` cụ thể.
- Nếu chỉ được sửa **một thứ** trước khi cho học viên thật sử dụng, đó là gì?
- Vòng lặp đánh giá này sẽ chạy lại **khi nào**: sau mỗi lần đổi chỉ dẫn hệ thống, mỗi
  tuần hay khi tập tài liệu thay đổi? Ai sẽ xem kết quả?
- Điều gì trong bài này bạn sẽ **mang về áp dụng** vào sản phẩm thật của mình?
