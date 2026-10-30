# Self-training — Tự huấn luyện bằng nhãn giả

> **Nhánh:** MACHINE LEARNING → Ít nhãn. **Nguồn:** dòng 42 bảng Excel. **Đối chiếu:** 29/10/2026. Ví dụ dùng SelfTrainingClassifier với MultinomialNB, sklearn 1.8.0; gold labels và pseudo-labels được tách rõ. Dữ liệu và tình huống khách sạn đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bối cảnh | Phân tích cảm xúc review khách sạn tiếng Việt |
| Nguồn tín hiệu | Phân biệt gold labels, pseudo/weak labels và target tự tạo trong bài |
| Đánh giá | Cần protocol và dữ liệu độc lập phù hợp, không có benchmark review thật ở ví dụ |

## 1. Định nghĩa học thuật

Self-training huấn luyện một mô hình từ dữ liệu có nhãn, dùng chính mô hình dự đoán nhãn cho dữ liệu chưa nhãn rồi chọn một phần nhãn giả để huấn luyện tiếp. Đây là một cách học bán giám sát, không phải nhãn do con người xác nhận. Các biến thể khác nhau ở tiêu chí chọn, cập nhật và trọng số nhãn giả.

## 2. Giải thích đơn giản

Bạn dạy mô hình bằng ít review đã gán nhãn. Nó thử đọc review mới và giữ những dự đoán có vẻ đủ tự tin làm bài luyện tiếp. Nguy cơ là nó có thể tự tin sai và học lại lỗi của chính mình. Lặp nhiều vòng không tự sửa điều đó; cần gold evaluation và kiểm soát nhãn giả.

## 3. Mục đích và vấn đề giải quyết

Mục đích là khai thác U để cải thiện classifier với ít nhãn, trong điều kiện mô hình khởi đầu đủ hữu ích. Nghiên cứu khách sạn nên so chất lượng cùng ngân sách gold labels với baseline không thêm pseudo-labels. Đo cả tỷ lệ được chọn và lỗi lớp hiếm thay vì chỉ số lượng nhãn giả tạo ra.

## 4. Điều kiện và giả định sử dụng

Cần L có các lớp đủ đại diện, U phù hợp miền và base estimator có đầu ra phù hợp tiêu chí chọn. Với threshold dựa probability, calibration rất quan trọng. P=0.95 của một mô hình lệch không bảo đảm đúng 95% khi chuyển miền. Nhãn giả sai có thể tạo confirmation bias.

## 5. Cơ chế hoạt động bên trong

Fit base trên L; predict_proba cho U còn lại; chọn mẫu theo threshold hoặc k-best; gán nhãn dự đoán; thêm vào tập huấn luyện rồi refit. Lặp đến hết U, không có mẫu mới hoặc đạt giới hạn. Trong API ví dụ, nhãn đã chọn được giữ theo cơ chế estimator; không khẳng định mọi self-training đều gán lại mềm ở mỗi vòng [S1].

## 6. Quy trình áp dụng trong nghiên cứu

1. Tạo gold L và gold development/test độc lập.
2. Xác định U-train và phạm vi học biểu diễn.
3. Chạy base chỉ L, kiểm tra calibration khi có dữ liệu phù hợp.
4. Chọn threshold/k-best, số vòng và weighting bằng development.
5. Lưu nhãn giả, confidence và vòng được chọn.
6. Đánh giá toàn model trên gold test theo protocol.

Không đưa nhãn test hoặc test input vào U trong thiết kế inductive; nếu dùng test input phải công bố transductive.

## 7. Ví dụ minh họa xuyên suốt

L gồm “phòng sạch”, “nhân viên tốt” tích cực và “phòng bẩn”, “nhân viên tệ” tiêu cực. U gồm các câu tương tự nhưng lặp tín hiệu. Ví dụ thiết kế để quan sát cơ chế nhận nhãn giả, không chứng minh hiểu tiếng Việt. “Không sạch” vẫn có thể bị gán sai nếu mô hình chủ yếu học từ sạch.

## 8. Kết quả và cách diễn giải

transduction_ trong sklearn chứa gold ban đầu cùng nhãn được mô hình thêm; các hàng chưa được chọn có thể vẫn -1. labeled_iter_ theo dõi vòng được gán theo API. Cần phân biệt số mẫu được chọn với độ chính xác nhãn giả: nhiều nhãn hơn có thể chỉ làm tăng noise.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là bọc được nhiều classifier có probability và dễ tạo baseline. Hạn chế là khuếch đại lỗi, nhạy confidence/threshold, có thể chỉ chọn lớp đông/câu dễ. Điều đó làm train mở rộng thiếu đa dạng dù số hàng tăng. Không có bảo đảm pseudo-labels giúp mọi cỡ L hoặc miền U.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| Graph-based SSL | Truyền/làm trơn nhãn theo quan hệ mẫu |
| Active learning | Hỏi nhãn thật ở mẫu cần thiết, không tự lấy dự đoán làm đáp án |
| Weak supervision | Nguồn nhãn có thể là quy tắc/metadata thay vì chính classifier |
| Self-supervised | Học target từ dữ liệu cho pretext task, không nhất thiết giả lập sentiment |

Teacher-student/pseudo-labeling là họ rộng hơn, không mặc định giống hệt wrapper sklearn.

## 11. Sai lầm thường gặp và cách khắc phục

Không gọi confidence là gold accuracy; không đánh giá bằng nhãn tự sinh. Đừng dùng base_estimator từ API cũ khi bản đang dùng nhận estimator. Không hạ threshold vì test kém rồi vẫn báo test độc lập. Không đưa -1 như một lớp bình thường vào NB; wrapper mới chịu trách nhiệm chọn phần có nhãn để fit.

## 12. Tài liệu tham khảo

- [S1] scikit-learn. [SelfTrainingClassifier](https://scikit-learn.org/1.8/modules/generated/sklearn.semi_supervised.SelfTrainingClassifier.html).
- [S2] scikit-learn. [Semi-supervised learning](https://scikit-learn.org/1.8/modules/semi_supervised.html).
- [S3] Yarowsky, D. (1995). *Unsupervised Word Sense Disambiguation Rivaling Supervised Methods*. [Bài gốc](https://aclanthology.org/P95-1026/).

[S3] là mốc tham khảo bootstrapping trong NLP, không tuyên bố cùng nhiệm vụ sentiment hoặc cùng API hiện tại.

## 13. Vị trí trong toàn bộ nghiên cứu

Self-training thuộc huấn luyện classifier, sau khâu chuẩn hóa nhãn và split. Nếu đầu ra dùng làm biến nghiên cứu, cần đánh giá sai số riêng, không tính pseudo-labels như thêm người trả lời khảo sát. Nguồn nhãn và mức xác thực phải được giữ rõ trong dữ liệu và báo cáo.

## 14. Kiến thức nền cần biết trước

Cần hiểu probability, calibration, threshold, class imbalance và rò rỉ. Biết một mẫu chưa có nhãn thật vẫn có thể có dự đoán rất tự tin. Với vectorizer, cần xác định fit trên L hay L+U-train; cả hai có thể là thiết kế hợp lệ nhưng không được âm thầm gồm test.

## 15. Nền tảng lý thuyết và logic hình thành

Lập luận là nhãn giả đủ đúng có thể thêm tín hiệu ở các vùng U giúp mô hình học. Nhưng mô hình tạo nhãn phụ thuộc chính sai lệch từ L, nên dữ liệu mới không độc lập như gold annotation mới. Trade-off giữa chất lượng và lượng mẫu cần kiểm chứng, không suy từ threshold đơn lẻ.

## 16. Các thành phần cấu tạo

Các thành phần gồm L, U còn lại, base model, confidence, selector, nhãn giả, vòng lặp và điều kiện dừng. Với biến thể khác có teacher, augmentation hoặc weighting riêng. Ghi rõ dùng nhãn cứng/mềm và có cập nhật nhãn cũ không, vì các lựa chọn thay đổi thuật toán.

## 17. Các loại và biến thể

Có threshold, k-best, cân bằng theo lớp, soft labels, teacher-student và consistency kết hợp. Không khẳng định wrapper sklearn hỗ trợ mọi biến thể đó. Tiêu chí k-best chỉ xếp độ tự tin, không bảo đảm đủ tin cậy tuyệt đối; nếu mọi mẫu đều khó, vẫn có thể chọn sai.

## 18. Đầu vào và đầu ra

Đầu vào X gồm L+U-train và y có -1 ở U trong API bài. Đầu ra là classifier đã fit, nhãn mở rộng và thông tin vòng gán. Predict trên mẫu mới không trả -1 như chính sách abstain mặc định; muốn từ chối phải thiết kế thêm và đánh giá coverage.

## 19. Căn cứ lựa chọn

Đáng thử khi base model đã có chất lượng ban đầu và U cùng miền, có gold development để chọn quy trình. Chọn dựa trên test độc lập, quality/coverage của pseudo-labels trên mẫu audit và lỗi theo lớp. So cùng lượng gold labels với baseline, không so với một mô hình bị cắt mất quyền truy cập feature.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu nhãn seed sai, thiếu lớp, U lệch miền mạnh hoặc xác suất quá kém mà không có cách đánh giá. Không dùng khi nhiệm vụ đòi mọi nhãn được con người xác thực. Tăng vòng không thay thế quy trình gán nhãn cho các nhóm diễn đạt hiếm hoặc câu hỗn hợp.

## 21. Các quyết định quan trọng khi thực hiện

Chọn threshold/k-best, số vòng, có weighting theo confidence hay không và cách giữ gold/pseudo. Chốt điều kiện dừng và audit trước khi chạy. Nếu dùng calibration, fit đúng phạm vi train/development và tính chi phí nhãn đó vào ngân sách, không gọi nó là “không cần nhãn bổ sung”.

## 22. Giải thích công thức và ký hiệu

Với classifier ở vòng t:

$$\widehat y_j=\arg\max_cp_t(c\mid x_j),\quad q_j=\max_cp_t(c\mid x_j),\quad A_t=\{j\in U_t:q_j>\tau\}.$$

Tập học tiếp gồm L và nhãn giả đã chấp nhận. Với $p(1)=(0.95,0.55,0.08)$, $\tau=0.9$, chọn mẫu 1 với nhãn 1 và mẫu 3 với nhãn 0; không chọn mẫu 2. Threshold so trên max probability, không chỉ xác suất lớp tích cực. Quy tắc bằng ngưỡng cần nêu theo triển khai.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Ví dụ dùng CountVectorizer fit trên L+U-train và NB trong wrapper thật; không có test tham gia fit.

```python
import numpy as np
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.naive_bayes import MultinomialNB
from sklearn.semi_supervised import SelfTrainingClassifier
texts = ["phòng sạch", "nhân viên tốt", "phòng sạch sạch", "nhân viên tốt tốt",
         "phòng bẩn", "nhân viên tệ", "phòng bẩn bẩn", "nhân viên tệ tệ"]
y = np.array([1,1,-1,-1,0,0,-1,-1])
vec = CountVectorizer(ngram_range=(1,2))
X = vec.fit_transform(texts)
model = SelfTrainingClassifier(estimator=MultinomialNB(alpha=0.2),
                               threshold=0.7, max_iter=5).fit(X,y)
known = y != -1
assert np.array_equal(model.transduction_[known], y[known])
new = vec.transform(["phòng sạch zzholdout"])
assert "zzholdout" not in vec.vocabulary_
prob = model.predict_proba(new)
assert np.isfinite(prob).all() and np.allclose(prob.sum(axis=1),1)
print("Nhãn sau self-training:", model.transduction_)
print("Vòng gán:", model.labeled_iter_)
p = np.array([0.95,0.55,0.08])
assert np.array_equal(np.maximum(p,1-p)>0.9, [True,False,True])
```

Threshold 0.7 chỉ đặt cho ví dụ dễ; không là khuyến nghị tối ưu cho dữ liệu thật. Không có đo pseudo-label accuracy hoặc benchmark cảm xúc trong mã.

## 24. Tình huống sai và phản ví dụ

Một model học từ sạch là tích cực có thể gán “phòng không sạch” tích cực với confidence cao, thêm nó vào train và củng cố lỗi. Một model thiên lớp đông có thể chỉ chọn pseudo-label lớp đông. Hai tình huống cho thấy confidence/coverage cần được audit bằng gold chứ không tự xác thực.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đánh giá gold test theo lớp, learning curve theo lượng L/U và seed. Theo dõi số pseudo-label mỗi vòng, phân bố lớp và mẫu audit có nhãn thật. Audit cũng tốn nhãn, cần báo ngân sách. Nếu không có mẫu nào được chọn, đó là tín hiệu cấu hình/base confidence, không tự là lỗi phần mềm.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu sai tăng, kiểm tra seed labels, U và calibration trước hạ/tăng threshold. Có thể dùng weighting, chọn theo lớp hoặc quay lại supervised nếu bằng chứng không ủng hộ. Nếu phát hiện giả nhãn sai hàng loạt, tạo lại pipeline từ gold và protocol đã sửa; không sửa test để hợp mô hình.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ mô tả lợi ích hoặc tác hại của pseudo-labels trên gold evaluation. Không kết luận đã tạo thêm nhãn thật hoặc tiết kiệm nhãn một tỷ lệ cố định. Không dùng nhãn giả để chứng minh khách hàng thực sự hài lòng khi chưa kiểm tra lỗi và tính đại diện.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Base model [...] fit trên L=[...], U-train=[...]. Pseudo-labels được chọn bằng [tiêu chí], threshold/số vòng chọn qua gold development. Gold và pseudo được lưu riêng; test [đơn vị] không tham gia lựa chọn.” Báo số nhãn thêm từng lớp/vòng và audit accuracy nếu có, không bịa mức đúng từ confidence.

## 29. Khả năng tái lập và tính minh bạch

Lưu IDs gold/U, nhãn giả, confidence, vòng chấp nhận, model/seed và cấu hình vectorizer. Không ghi đè gold bằng pseudo. Mã chạy sklearn 1.8.0 với estimator; môi trường khác cần kiểm tra API. Tái lập thực nghiệm phải giữ lịch sử chọn mẫu, không chỉ nhãn cuối.

## 30. Đạo đức nghiên cứu liên quan

Cần thể hiện rõ nhãn nào do mô hình tạo, tránh trình bày như xác nhận của người viết hoặc người gán nhãn. Bảo vệ review và audit các nhóm ít được mô hình chọn. Không để vòng tự học tự động quyết định xử lý nhân viên/khách mà thiếu người xem xét ở tình huống tác động.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Confidence 0.95 có bảo đảm đúng 95% không?
2. Vì sao nhãn giả làm tăng confirmation bias?
3. Với τ=0.9, p(positive)=0.08 có được chọn không?
4. Dùng dữ liệu gì đánh giá?

Gợi ý: mẫu đó có max probability 0.92, nên được chọn theo quy tắc bài. Bài mở rộng: audit chất lượng/coverage theo lớp.

## 32. Những điểm còn tranh luận

Các vấn đề cần kiểm chứng là ngưỡng hiệu quả theo miền, sai lệch lớp và cách kiểm soát feedback loop. Không có threshold tốt nhất chung. Nghiên cứu nên báo cả trường hợp không thêm được nhãn hoặc thêm làm kém đi, thay vì chỉ đếm số nhãn giả như thành công.

