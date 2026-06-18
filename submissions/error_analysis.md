# Lab 7 – Error Analysis Report

## 1. Định nghĩa bài toán

- **Tác vụ:** Phân loại cảm xúc (Sentiment Analysis) cho các bài đánh giá phim.
- **Input:** Một đoạn review phim.
- **Output:** Kết quả JSON gồm:
  - sentiment
  - explanation
  - evidence
  - (Prompt v2 bổ sung aspect và confidence)
- **LLM sử dụng:** Gemini 2.5 Flash thông qua Gemini API.

---

## 2. Testset

- Số lượng review: **30**
- Positive: **25**
- Negative: **5**
- Có một số review chứa cả ưu điểm và nhược điểm nhằm kiểm tra khả năng đánh giá tổng thể của mô hình.

---

## 3. Prompt v1

Prompt v1 sử dụng cách yêu cầu đơn giản:

- Xác định sentiment.
- Giải thích ngắn.
- Trích dẫn evidence.

Mục tiêu của Prompt v1 là tạo baseline để so sánh.

---

## 4. Prompt v2

Prompt v2 được cải tiến bằng cách:

- Yêu cầu JSON chặt chẽ hơn.
- Bổ sung aspect.
- Bổ sung polarity cho từng aspect.
- Thêm confidence.
- Yêu cầu evidence phải lấy trực tiếp từ review.

Mục tiêu là giảm lỗi định dạng và giúp kết quả đầy đủ hơn.

---

## 5. So sánh định lượng

| Metric | Prompt v1 | Prompt v2 | Nhận xét |
|---------|----------:|----------:|-----------|
| Accuracy | 100% | 100% | Hai prompt đều phân loại đúng trên testset. |
| Valid JSON rate | 100% | 96.7% | Prompt v2 có một trường hợp JSON lỗi do API trả về bất thường. |
| Evidence exactness rate | 100% | 100% | Evidence đều lấy trực tiếp từ review. |
| Hallucination count | 0 | 0 | Không phát hiện nội dung tự bịa. |
| Error count | 0 | 1 | Prompt v2 có một lỗi JSON ở R029. |

---

## 6. Error Buckets

| Error bucket | Count v1 | Count v2 | Example | Comment |
|---------------|---------:|---------:|----------|---------|
| wrong_sentiment | 0 | 0 | - | Không xảy ra |
| invalid_json | 0 | 1 | R029 | API trả về thêm nội dung ngoài JSON |
| hallucinated_evidence | 0 | 0 | - | Không có |
| missed_negative_aspect | 0 | 0 | - | Không có |
| overconfident | 0 | 0 | - | Không phát hiện |

---

## 7. Ba lỗi đáng chú ý

### Error 1

**Review ID:** R029

**Điều xảy ra**

Prompt v2 nhận được phản hồi từ Gemini không chỉ chứa JSON mà còn xuất hiện thêm một đoạn văn bản không liên quan, làm JSON không còn hợp lệ.

**Ý nghĩa**

Đây không phải lỗi phân loại sentiment mà là lỗi định dạng đầu ra. Khi sử dụng API thực tế, mô hình đôi khi vẫn sinh thêm nội dung ngoài yêu cầu.

**Hướng cải thiện**

Tăng ràng buộc trong prompt bằng cách yêu cầu:

> Return JSON only. Do not output any explanation outside JSON.

---

### Error 2

**Review ID:** Không có.

Trong quá trình thử nghiệm không ghi nhận trường hợp dự đoán sai sentiment.

---

### Error 3

**Review ID:** Không có.

Không phát hiện hallucination hoặc sử dụng kiến thức ngoài review.

---

## 8. Nhận xét

Qua kết quả thử nghiệm, Gemini 2.5 Flash cho độ chính xác rất cao trên tập dữ liệu gồm 30 review.

Prompt v2 tạo đầu ra đầy đủ hơn Prompt v1 nhờ bổ sung aspect, confidence và evidence rõ ràng.

Tuy nhiên Prompt v2 cũng dài hơn nên đôi khi mô hình trả về kết quả không hoàn toàn đúng định dạng JSON như mong muốn. Đây là điểm cần cải thiện nếu triển khai trong hệ thống tự động.

Trong tương lai có thể tiếp tục cải tiến prompt bằng cách:

- tăng ràng buộc định dạng JSON;
- bổ sung nhiều review khó hơn;
- thêm nhiều review mixed sentiment;
- đánh giá trên tập dữ liệu lớn hơn để có kết luận khách quan hơn.

---

## Kết luận

Prompt v1 phù hợp cho bài toán phân loại sentiment cơ bản.

Prompt v2 tuy phức tạp hơn nhưng cung cấp nhiều thông tin hữu ích như aspect và confidence, giúp kết quả dễ phân tích hơn.

Nhìn chung Prompt v2 là phiên bản tốt hơn mặc dù vẫn cần cải thiện tính ổn định của định dạng JSON khi sử dụng với API.