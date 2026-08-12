# Day 14 — Reflection

## Báo cáo Đánh giá & Phân tích Lỗi

Sử dụng kết quả thật trong `artifacts/benchmark_results.json` và đối chiếu với
trace answer/context trong `artifacts/actual_answers.json`.

---

## 1. Tổng hợp kết quả Benchmark

**Tỷ lệ pass tổng thể:** 0.0%

| Metric | Trung bình | Min | Max | Nhận xét |
|---|---:|---:|---:|---|
| Context Recall | 0.879 | 0.483 | 1.000 | Độ bao phủ của retrieval khá tốt. |
| Context Precision | 0.939 | 0.679 | 1.000 | Các chunk liên quan được xếp hạng lên đầu. |
| Faithfulness | 0.472 | 0.000 | 1.000 | Mức grounding của câu trả lời chỉ ở mức trung bình. |
| Relevance | 0.236 | 0.000 | 0.636 | Đây là metric yếu nhất. |
| Completeness | 0.098 | 0.000 | 0.471 | Phần lớn câu trả lời bỏ sót các fact bắt buộc. |
| Overall Score | 0.269 | 0.000 | 0.638 | Chất lượng tổng thể bị giới hạn chủ yếu bởi heuristic relevance. |

**Diễn giải theo thang điểm**

- Mức Good (0.8–1.0): 2 case đạt overall, cùng với phần lớn điểm retrieval
- Mức Needs Work (0.6–0.8): 5 case theo overall
- Mức Significant Issues (< 0.6): 13 case theo overall

**Phân bố loại lỗi**

| Loại lỗi | Số lượng | Tỷ lệ |
|---|---:|---:|
| hallucination | 7 | 35.0% |
| irrelevant | 7 | 35.0% |
| incomplete | 5 | 25.0% |
| off_topic | 1 | 5.0% |
| refusal | 0 | 0.0% |

**Chẩn đoán tổng thể:** Vấn đề chính không nằm ở retrieval. Context recall và
context precision đều cao, nhưng bản thân câu trả lời sinh ra lại thường thiếu ý
hoặc bịa thêm so với expected answer.

> Retrieval đang khỏe; chất lượng generation mới là điểm nghẽn.

---

## 2. Ba lỗi nghiêm trọng nhất — 5 Whys

### Lỗi 1

**ID và câu hỏi:**

> E04: Một sinh viên phải đáp ứng những điều kiện gì trước khi được đăng ký môn học?

**Câu trả lời mong đợi:**

> Sinh viên chỉ được đăng ký khi không còn hold nào đang hiệu lực về học vụ, kỷ luật, cố vấn hay tài chính, và số tín chỉ ở mức bình thường là 12 đến 18 tín chỉ trong học kỳ Fall hoặc Spring.

**Câu trả lời thực tế:**

> Context 5*: Prerequisites: A prerequisite is satisfied

**Điểm số:** Context Recall: 1.000 | Context Precision: 1.000 | Faithfulness: 0.000 | Relevance: 0.000 | Completeness: 0.000 | Overall: 0.000

**Kiểm chứng bằng evidence:** Context lấy về nhìn chung là phù hợp, nhưng câu trả
lời sinh ra bỏ sót các chi tiết policy bắt buộc hoặc thêm nội dung không có
trong context.

| Cấp độ | Câu hỏi | Trả lời |
|---|---|---|
| Triệu chứng | Vì sao điểm thấp? | Câu trả lời thiếu một hoặc nhiều fact bắt buộc, hoặc thêm nội dung không được hỗ trợ. |
| Why 1 | Vì sao điều đó xảy ra? | Model rút gọn câu trả lời hoặc diễn đạt lại quá mạnh tay. |
| Why 2 | Vì sao đó là vấn đề? | Metric completeness so sánh trực tiếp với nội dung của expected answer. |
| Why 3 | Vì sao retrieval không cứu được? | Các chunk cần thiết đã có mặt, nhưng bước generation không dùng hết chúng. |
| Why 4 | Vì sao heuristic không bù lại được? | Cách chấm điểm của lab rất chặt về overlap phía answer và mức độ grounding. |
| Why 5 | Nguyên nhân gốc? | Chính sách generation cần định dạng câu trả lời chặt hơn và bao phủ đủ mọi fact bắt buộc. |

**Nguyên nhân gốc do `find_root_cause()` đưa ra:**

> Context bị thiếu hoặc không liên quan — cần cải thiện retrieval

**Bạn có đồng ý không?**

> Không hoàn toàn. Evidence cho thấy khoảng trống chính nằm ở tính đầy đủ và cách diễn đạt của câu trả lời, không phải ở retrieval.

**Hướng sửa cụ thể:**

> Siết lại prompt sinh câu trả lời để bắt buộc nêu đủ mọi điều kiện policy trong phản hồi cuối, đồng thời bổ sung regression case cho các câu hỏi nhiều phần.

### Lỗi 2

**ID và câu hỏi:**

> M06: Chuyện gì xảy ra nếu sinh viên rút môn trước so với sau mốc census?

**Câu trả lời mong đợi:**

> Trước hoặc đúng ngày census, cổng thông tin ghi nhận là drop. Sau census và cho tới hạn withdrawal, môn học bị ghi điểm W; sau hạn đó, muốn rút môn phải nộp đơn xin xét trường hợp đặc biệt.

**Câu trả lời thực tế:**

> Based on the provided contexts, the consequences

**Điểm số:** Context Recall: 1.000 | Context Precision: 1.000 | Faithfulness: 0.000 | Relevance: 0.000 | Completeness: 0.000 | Overall: 0.000

**Kiểm chứng bằng evidence:** Context lấy về nhìn chung là phù hợp, nhưng câu trả
lời sinh ra bỏ sót các chi tiết policy bắt buộc hoặc thêm nội dung không có
trong context.

| Cấp độ | Câu hỏi | Trả lời |
|---|---|---|
| Triệu chứng | Vì sao điểm thấp? | Câu trả lời thiếu một hoặc nhiều fact bắt buộc, hoặc thêm nội dung không được hỗ trợ. |
| Why 1 | Vì sao điều đó xảy ra? | Model rút gọn câu trả lời hoặc diễn đạt lại quá mạnh tay. |
| Why 2 | Vì sao đó là vấn đề? | Metric completeness so sánh trực tiếp với nội dung của expected answer. |
| Why 3 | Vì sao retrieval không cứu được? | Các chunk cần thiết đã có mặt, nhưng bước generation không dùng hết chúng. |
| Why 4 | Vì sao heuristic không bù lại được? | Cách chấm điểm của lab rất chặt về overlap phía answer và mức độ grounding. |
| Why 5 | Nguyên nhân gốc? | Chính sách generation cần định dạng câu trả lời chặt hơn và bao phủ đủ mọi fact bắt buộc. |

**Nguyên nhân gốc do `find_root_cause()` đưa ra:**

> Context bị thiếu hoặc không liên quan — cần cải thiện retrieval

**Bạn có đồng ý không?**

> Không hoàn toàn. Evidence cho thấy khoảng trống chính nằm ở tính đầy đủ và cách diễn đạt của câu trả lời, không phải ở retrieval.

**Hướng sửa cụ thể:**

> Siết lại prompt sinh câu trả lời để bắt buộc nêu đủ mọi điều kiện policy trong phản hồi cuối, đồng thời bổ sung regression case cho các câu hỏi nhiều phần.

### Lỗi 3

**ID và câu hỏi:**

> M07: Các yêu cầu học vụ chính để tốt nghiệp là gì và sinh viên nên nộp đơn khi nào?

**Câu trả lời mong đợi:**

> Sinh viên đại học đủ điều kiện học vụ để tốt nghiệp sau khi hoàn thành ít nhất 120 tín chỉ được tính, tất cả các môn bắt buộc của chương trình, yêu cầu capstone, và GPA tích lũy tối thiểu 2.00. Đơn xin tốt nghiệp chính thức phải nộp trước ngày census của học kỳ dự kiến tốt nghiệp.

**Câu trả lời thực tế:**

> Based on the provided contexts:

* **

**Điểm số:** Context Recall: 0.483 | Context Precision: 0.833 | Faithfulness: 0.000 | Relevance: 0.000 | Completeness: 0.000 | Overall: 0.000

**Kiểm chứng bằng evidence:** Context lấy về nhìn chung là phù hợp, nhưng câu trả
lời sinh ra bỏ sót các chi tiết policy bắt buộc hoặc thêm nội dung không có
trong context.

| Cấp độ | Câu hỏi | Trả lời |
|---|---|---|
| Triệu chứng | Vì sao điểm thấp? | Câu trả lời thiếu một hoặc nhiều fact bắt buộc, hoặc thêm nội dung không được hỗ trợ. |
| Why 1 | Vì sao điều đó xảy ra? | Model rút gọn câu trả lời hoặc diễn đạt lại quá mạnh tay. |
| Why 2 | Vì sao đó là vấn đề? | Metric completeness so sánh trực tiếp với nội dung của expected answer. |
| Why 3 | Vì sao retrieval không cứu được? | Các chunk cần thiết đã có mặt, nhưng bước generation không dùng hết chúng. |
| Why 4 | Vì sao heuristic không bù lại được? | Cách chấm điểm của lab rất chặt về overlap phía answer và mức độ grounding. |
| Why 5 | Nguyên nhân gốc? | Chính sách generation cần định dạng câu trả lời chặt hơn và bao phủ đủ mọi fact bắt buộc. |

**Nguyên nhân gốc do `find_root_cause()` đưa ra:**

> Context bị thiếu hoặc không liên quan — cần cải thiện retrieval

**Bạn có đồng ý không?**

> Không hoàn toàn. Evidence cho thấy khoảng trống chính nằm ở tính đầy đủ và cách diễn đạt của câu trả lời, không phải ở retrieval.

**Hướng sửa cụ thể:**

> Siết lại prompt sinh câu trả lời để bắt buộc nêu đủ mọi điều kiện policy trong phản hồi cuối, đồng thời bổ sung regression case cho các câu hỏi nhiều phần.


---

## 3. Gom nhóm lỗi (Failure Clustering)

| Cụm | Nguyên nhân gốc | ID các lỗi | Ưu tiên |
|---|---|---|---|
| 1 | Câu trả lời thiếu ý hoặc bịa thêm dù retrieval tốt | E02, E05, M01, M02, M03, M04, M06, M07, H01, H02, H03, H04, H05 | Cao |
| 2 | Model đôi khi tóm tắt policy đúng nhưng chưa đầy đủ | E01, M05, A01, A02, A03 | Trung bình |
| 3 | Một số prompt adversarial được xử lý an toàn nhưng vẫn bị heuristic chấm thấp | A01, A02, A03 | Trung bình |

**Nếu chỉ được sửa một cụm, bạn chọn cụm nào và vì sao?**

> Chọn Cụm 1, vì nó chiếm phần lớn số lỗi và sửa được nó sẽ cải thiện đồng thời cả tính đúng đắn lẫn tính đầy đủ.

---

## 4. Improvement Log

```text
| Failure ID | Loại lỗi | Nguyên nhân gốc | Hướng sửa đề xuất | Trạng thái |
|------------|----------|-----------------|-------------------|------------|
| F001 | irrelevant | Câu trả lời thiếu thông tin then chốt — mở rộng context window hoặc cải thiện generation | Bổ sung hallucination checker để lọc các claim không được hỗ trợ | Open |
| F002 | incomplete | Câu trả lời thiếu thông tin then chốt — mở rộng context window hoặc cải thiện generation | Thêm few-shot example minh họa câu trả lời đầy đủ để tăng completeness | Open |
| F003 | irrelevant | Câu trả lời thiếu thông tin then chốt — mở rộng context window hoặc cải thiện generation | Làm rõ prompt và cải thiện intent detection để câu trả lời bám sát câu hỏi | Open |
| F004 | hallucination | Context bị thiếu hoặc không liên quan — cải thiện retrieval |  | Open |
| F005 | incomplete | Câu trả lời thiếu thông tin then chốt — mở rộng context window hoặc cải thiện generation |  | Open |
| F006 | incomplete | Câu trả lời thiếu thông tin then chốt — mở rộng context window hoặc cải thiện generation |  | Open |
| F007 | hallucination | Câu trả lời thiếu thông tin then chốt — mở rộng context window hoặc cải thiện generation |  | Open |
| F008 | incomplete | Câu trả lời thiếu thông tin then chốt — mở rộng context window hoặc cải thiện generation |  | Open |
| F009 | incomplete | Câu trả lời thiếu thông tin then chốt — mở rộng context window hoặc cải thiện generation |  | Open |
| F010 | irrelevant | Câu trả lời thiếu thông tin then chốt — mở rộng context window hoặc cải thiện generation |  | Open |
| F011 | hallucination | Context bị thiếu hoặc không liên quan — cải thiện retrieval |  | Open |
| F012 | hallucination | Context bị thiếu hoặc không liên quan — cải thiện retrieval |  | Open |
| F013 | off_topic | Câu trả lời không giải quyết câu hỏi — làm rõ prompt |  | Open |
| F014 | hallucination | Context bị thiếu hoặc không liên quan — cải thiện retrieval |  | Open |
| F015 | hallucination | Context bị thiếu hoặc không liên quan — cải thiện retrieval |  | Open |
| F016 | irrelevant | Câu trả lời không giải quyết câu hỏi — làm rõ prompt |  | Open |
| F017 | hallucination | Context bị thiếu hoặc không liên quan — cải thiện retrieval |  | Open |
| F018 | irrelevant | Câu trả lời thiếu thông tin then chốt — mở rộng context window hoặc cải thiện generation |  | Open |
| F019 | irrelevant | Câu trả lời không giải quyết câu hỏi — làm rõ prompt |  | Open |
| F020 | irrelevant | Câu trả lời không giải quyết câu hỏi — làm rõ prompt |  | Open |
```

**Ba đề xuất theo thứ tự ưu tiên**

1. Làm rõ prompt và cải thiện intent detection để câu trả lời bám sát câu hỏi.
2. Cải thiện intent detection và lọc chủ đề để tránh câu trả lời lạc đề.
3. Tăng chunk size trong RAG pipeline để giảm phân mảnh context.

| Đề xuất | Metric mục tiêu | Cách kiểm chứng |
|---|---|---|
| Làm rõ prompt và cải thiện intent detection | Relevance | Chạy lại đúng benchmark 20 case và so sánh relevance cùng tỷ lệ pass tổng. |
| Cải thiện intent detection và lọc chủ đề | Relevance / tỷ lệ pass | Thêm các case adversarial và refusal, rồi kiểm tra tỷ lệ false positive trên prompt ngoài phạm vi. |
| Tăng chunk size trong RAG pipeline | Context recall / completeness | Chạy lại các metric retrieval và xác nhận recall không giảm trong khi completeness tăng. |

---

## 5. Chiến lược Regression Testing

**Q1: Khi nào nên chạy `run_regression()` trong quy trình production?**

> Chạy sau mỗi thay đổi về prompt, retrieval hoặc generation, và chạy trước khi merge/deploy, để phát hiện sớm việc tụt chất lượng.

**Q2: Ngưỡng sụt giảm 0.05 có phù hợp với lab này không? Vì sao?**

> Phù hợp ở vai trò ngưỡng cảnh báo, nhưng không nên coi là tín hiệu đúng tuyệt đối, vì một metric dựa trên từ vựng vẫn có thể dao động dù câu trả lời vẫn đúng về mặt ngữ nghĩa.

**Q3: Metric/lỗi nào nên chặn deploy, và loại nào chỉ nên cảnh báo?**

> Chặn deploy khi faithfulness hoặc completeness giảm, vì chúng cho thấy câu trả lời sai hoặc thiếu. Relevance chỉ nên cảnh báo trước, vì đây là metric nhiễu nhất trong lab này.

**Q4: Điền các giai đoạn evaluation vào luồng sau.**

```text
Thay đổi code/prompt/retrieval -> offline eval -> regression check -> deploy
```

> Ý tưởng là chạy benchmark offline trước, so sánh với baseline, và chỉ đưa thay đổi lên nếu không có regression thuộc nhóm chặn deploy.

---

## 6. Vòng lặp Cải tiến Liên tục

```text
Đánh giá -> Phân tích -> Cải tiến -> Mở rộng benchmark -> Lặp lại
```

| Ưu tiên | Hành động | Metric kỳ vọng cải thiện | Tác động kỳ vọng |
|---:|---|---|---|
| 1 | Thay heuristic relevance dựa trên từ vựng bằng một semantic judge | Relevance | Giảm false negative với các câu diễn đạt lại và các câu từ chối hợp lệ. |
| 2 | Bổ sung regression case nhiều paraphrase và nhiều tình huống refusal | Tỷ lệ pass / độ bền vững | Bao phủ tốt hơn cách diễn đạt thật của người dùng. |
| 3 | Siết chặt template câu trả lời cho các câu hỏi nhiều phần | Completeness | Bao phủ đầy đủ hơn các fact bắt buộc. |

**Nên bổ sung những failure case nào tiếp theo?**

> Bổ sung thêm các câu hỏi policy nhiều paraphrase, thêm các case prompt injection, và một vài câu hỏi nhiều phần về deadline mà câu trả lời đúng phải kết hợp hai tài liệu.

---

## 7. Reflection cuối

**Điều gì khiến bạn bất ngờ nhất trong benchmark?**

> Các metric retrieval rất tốt trong khi các metric phía generation lại yếu hơn nhiều, cho thấy pipeline lấy đúng tài liệu nhưng không phải lúc nào cũng tổng hợp chúng thành câu trả lời đầy đủ.

**Giới hạn của heuristic dựa trên overlap từ là gì, và trong production bạn sẽ dùng gì?**

> Chúng bỏ sót các câu diễn đạt lại, các câu từ chối an toàn, và những câu trả lời đúng nhưng dùng cách nói khác. Trong production, tôi sẽ giữ các metric retrieval, rồi thay heuristic phía answer bằng LLM judge hoặc chấm điểm ngữ nghĩa dựa trên embedding, kèm calibrate với con người.
