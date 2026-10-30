# Maximum Entropy — Nguyên lý entropy cực đại và bộ phân loại log-linear

> **Nhánh:** MACHINE LEARNING → Supervised: Probabilistic. **Nguồn:** dòng 19 bảng Excel. **Đối chiếu:** 28/10/2026. Phạm vi là MaxEnt có điều kiện cho phân loại, không bao trùm mọi ứng dụng nguyên lý entropy. Ví dụ và dữ liệu tự xây. Mã kiểm tra với scikit-learn 1.8.0.

| Trường | Nội dung |
|---|---|
| Bản chất | Nguyên lý chọn phân bố dưới ràng buộc; trong NLP thường dẫn tới classifier log-linear có điều kiện |
| Mốc tham khảo | Berger, Della Pietra & Della Pietra, 1996 về ứng dụng NLP [S1]; không phải năm khai sinh toàn bộ nguyên lý entropy cực đại |
| Đầu vào | Đặc trưng kết hợp đầu vào–nhãn, dữ liệu có nhãn và regularization nếu dùng |
| Đầu ra | Phân bố xác suất lớp, nhãn theo quy tắc và trọng số feature |
| Tiếng Việt | Phụ thuộc feature và nhãn tiếng Việt; không yêu cầu các từ độc lập theo mô hình NB |
| Nhãn/tài nguyên | Cần dữ liệu ước lượng thống kê/huấn luyện; mô hình tuyến tính sparse thường dùng CPU được |
| Liên hệ LR | Binary logistic và multinomial softmax là các dạng tương ứng khi feature/cấu hình phù hợp [S2] |

## 1. Định nghĩa học thuật

Nguyên lý Maximum Entropy chọn phân bố có entropy lớn nhất trong tập phân bố thỏa các ràng buộc đã biết. Với classifier có điều kiện, các ràng buộc thường liên quan kỳ vọng feature quan sát từ dữ liệu; nghiệm có dạng log-linear/exponential dưới các điều kiện phù hợp [S1].

Trong NLP, “MaxEnt classifier” thường chỉ mô hình xác suất lớp tỷ lệ với exponential của tổng feature có trọng số. Không đồng nhất mọi ứng dụng entropy cực đại với một estimator duy nhất.

## 2. Giải thích đơn giản

Hãy tưởng tượng bạn biết một số sự kiện thống kê từ dữ liệu nhưng chưa biết toàn bộ cách phân bố nhãn. Nguyên lý yêu cầu thỏa những thông tin đó, rồi tránh thêm sự thiên lệch không có căn cứ trong phần còn lại.

Điều này không có nghĩa luôn dự đoán các lớp bằng nhau. Khi ràng buộc cho thấy “bẩn” liên quan lớp tiêu cực, phân bố cần phản ánh bằng chứng đó. Đồng đều chỉ xuất hiện trong những tình huống thiếu ràng buộc phân biệt thích hợp.

## 3. Mục đích và vấn đề giải quyết

Đề xuất nghiên cứu khách sạn: kết hợp feature từ, cụm, phủ định và metadata được phép để dự đoán lớp phản hồi. MaxEnt có thể dùng feature chồng lấn mà không đặt giả định likelihood độc lập kiểu NB.

Nếu dùng để so mô hình, cần xem LR đã dùng trước có cùng dạng tham số không. Nếu chỉ đổi tên mà giữ feature, loss, regularization và solver tương đương, đó không phải một phương pháp dự đoán mới độc lập.

## 4. Điều kiện và giả định sử dụng

Cần tập nhãn, feature và dữ liệu đủ để ước lượng. Ràng buộc/feature phải có ý nghĩa và được tạo bằng thông tin sẵn có lúc dự đoán. Không dùng điểm sao làm feature nếu chính điểm sao được dùng tạo nhãn rồi gọi kết quả là hiểu sentiment từ chữ.

Giả định của mô hình nằm ở dạng log-linear theo feature. Không cần feature độc lập có điều kiện kiểu NB, nhưng vẫn có thể sai mô hình, overfit hoặc lệch miền. Regularization và chia tập vẫn cần thiết.

## 5. Cơ chế hoạt động bên trong

Quy trình thực hành: thiết kế feature → mã hóa vector → cộng điểm theo nhãn → exponential và chuẩn hóa → tính conditional log loss → tối ưu trọng số có/không regularization → dự đoán.

Nguyên lý ràng buộc và cách tối ưu là hai lớp giải thích. Tài liệu lịch sử có các phương pháp iterative scaling; mã hiện đại có thể dùng L-BFGS [S1–S2]. Chọn solver không tự thay đổi nhiệm vụ thành một họ mô hình hoàn toàn khác.

## 6. Quy trình áp dụng trong nghiên cứu

1. Xác định nhãn, đơn vị và mục tiêu phân loại.
2. Liệt kê feature, tránh thông tin chỉ có sau kết quả.
3. Chia train/development/test trước khi học vocabulary/chọn feature.
4. Xây một baseline feature đơn giản và mô hình log-linear.
5. Chọn regularization, solver và feature bổ sung qua development/CV.
6. Kiểm tra hội tụ, thứ tự lớp và xác suất.
7. Đánh giá nhãn, calibration và lỗi theo nhóm; đóng băng trước test.

Nếu nhánh trước đã có Logistic Regression tương đương, dùng phần này để đào sâu nguyên lý hoặc thử feature khác có ghi rõ, không trình bày là so “hai thuật toán” chỉ từ tên.

## 7. Ví dụ minh họa xuyên suốt

Với ba lớp tiêu cực, trung tính và tích cực, giả định một mẫu có điểm log-linear tương ứng $(0,0,\log 2)$. Sau exponential, trọng số là $(1,1,2)$, tổng 4 nên xác suất là $(0.25,0.25,0.5)$.

Đó là số tự đặt để học chuẩn hóa. Điểm lớp tích cực không tự đến từ “entropy” mà từ feature và trọng số đã học/đặt. Một câu có cả phòng tốt và dịch vụ kém vẫn cần định nghĩa nhãn toàn câu.

## 8. Kết quả và cách diễn giải

Đầu ra là phân bố lớp theo mô hình; tổng xác suất bằng 1. Entropy cao có thể cho thấy phân bố ít tập trung, nhưng không chứng minh mẫu thật khó hoặc mô hình biết mình sai. Một mô hình sai có thể rất tự tin.

Hệ số feature là đóng góp vào score theo nhãn trong cấu hình cụ thể. Với softmax, nhiều tham số hóa có thể tạo cùng phân bố; không đọc một hệ số tách khỏi lớp đối chiếu/quy ước mô hình như tác động độc lập phổ quát.

## 9. Điểm mạnh, điểm yếu và giới hạn

Lợi thế là kết hợp được feature linh hoạt, có xác suất và khung toán học rõ. Feature có thể chồng lấn, như từ “sạch” và bigram “không sạch”, dù mức hữu ích vẫn cần học và kiểm tra.

Hạn chế là phụ thuộc thiết kế feature, dữ liệu và regularization. Tên gọi “ít áp đặt giả định” không có nghĩa không có giả định hay không thiên lệch. Ràng buộc sai, mẫu lệch hoặc metadata rò rỉ vẫn dẫn tới mô hình không phù hợp.

## 10. So sánh với phương pháp gần nhau

| Đối tượng | Quan hệ |
|---|---|
| Logistic Regression | Tương đương MaxEnt classifier trong các tham số hóa binary/multinomial phù hợp |
| Naive Bayes | Mô hình likelihood + prior với độc lập có điều kiện; không cùng objective |
| MEMM | Mô hình chuỗi chuẩn hóa cục bộ, không chỉ classifier cả văn bản |
| CRF chuỗi | Chuẩn hóa trên chuỗi nhãn, mô hình hóa liên kết cấu trúc |
| Entropy regularization | Một thành phần mục tiêu có thể dùng ở hệ khác; không đồng nhất nguyên lý MaxEnt dưới ràng buộc |

Không dùng tên “Maximum Entropy” để suy mô hình dự đoán sẽ luôn entropy cao trên mọi đầu vào.

## 11. Sai lầm thường gặp và cách khắc phục

- Coi MaxEnt và LR luôn khác thuật toán: đối chiếu phương trình, feature và regularization.
- Hiểu “maximum entropy” là bắt mọi xác suất bằng nhau: xét ràng buộc dữ liệu.
- Bỏ regularization vì nghĩ mô hình ít giả định: kiểm tra khả năng overfit và separation.
- Dùng feature chứa nhãn ẩn: rà thông tin có sẵn tại thời điểm dự đoán.
- Xem tổng xác suất bằng 1 là đủ calibration: đánh giá độc lập.

## 12. Tài liệu tham khảo

- **[S1]** Berger, A. L., Della Pietra, S. A., & Della Pietra, V. J. (1996). *A Maximum Entropy Approach to Natural Language Processing*. Computational Linguistics, 22(1), 39–71. [ACL Anthology](https://aclanthology.org/J96-1002/).
- **[S2]** scikit-learn. [Linear Models: Logistic Regression](https://scikit-learn.org/stable/modules/linear_model.html#logistic-regression).
- **[S3]** scikit-learn. [LogisticRegression API](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html).
- **[S4]** scikit-learn. [Probability calibration](https://scikit-learn.org/stable/modules/calibration.html).
- **[S5]** scikit-learn. [Common pitfalls](https://scikit-learn.org/stable/common_pitfalls.html).

S1 là nguồn nguyên lý/ứng dụng NLP; S2–S3 giúp đối chiếu dạng classifier và triển khai. Các feature và kết quả minh họa trong bài là thiết kế riêng, không phải tái lập các thí nghiệm lịch sử.

## 13. Vị trí trong toàn bộ nghiên cứu

MaxEnt classifier nằm sau biểu diễn và trước đánh giá/sử dụng dự đoán. Nếu dùng feature được trích từ mô hình khác, cần kiểm tra mô hình trích có được fit bằng test hay nhãn ngoài phạm vi không.

Trong nghiên cứu quản trị, nó có thể là công cụ đo biểu hiện văn bản. Nguyên lý entropy không thay thế độ giá trị thang đo, tính đại diện mẫu hoặc thiết kế nhân quả.

## 14. Kiến thức nền cần biết trước

Cần hiểu entropy, kỳ vọng, ràng buộc, exponential, softmax, log loss và regularization. Feature $f_j(x,y)$ có thể phụ thuộc cả đầu vào và nhãn, không chỉ là một cột văn bản không phân biệt lớp.

Phân biệt entropy của một phân bố với sai số dự đoán. Entropy đo mức tập trung của phân bố đang xét, không tự là xác suất mô hình sai hoặc mức đồng thuận của con người.

## 15. Nền tảng lý thuyết và logic hình thành

Ở dạng cổ điển, ta chọn phân bố có điều kiện có entropy lớn nhất với các kỳ vọng feature khớp những thống kê đã đặt. Dạng đối ngẫu liên hệ với ước lượng likelihood của mô hình exponential [S1].

Trong thực hành có regularization, các điều kiện khớp thống kê không nên được mô tả máy móc như equality cứng y nguyên bài toán không regularization. Cần nói rõ mục tiêu thực dùng và lý do chọn cấu hình.

## 16. Các thành phần cấu tạo

Hệ gồm tập lớp, bộ feature, trọng số, partition function, loss, regularization, solver và quy tắc quyết định. Bộ feature có thể là count, TF-IDF hoặc đặc trưng được thiết kế.

Nếu thêm metadata, ghi rõ đó là bằng chứng dự đoán nào và có phù hợp câu hỏi không. Ví dụ khách sạn hoặc nền tảng có thể giúp dự đoán trong mẫu nhưng tạo shortcut ngoài mục tiêu ngôn ngữ.

## 17. Các loại và biến thể

Có MaxEnt binary và multiclass có điều kiện, dạng feature tuyến tính phổ biến và những cấu hình biểu diễn khác. Dạng joint maximum entropy và các mô hình chuỗi không nên gộp vào cùng công thức classifier đơn vị.

Regularization, class weight và solver là các cấu hình ảnh hưởng kết quả. “Dùng LR để triển khai MaxEnt” hợp lý khi công bố rõ tương đương, không phải cách tạo một đối chứng mới chỉ bằng nhãn tên.

## 18. Đầu vào và đầu ra

Đầu vào gồm văn bản/feature và nhãn; đầu ra lưu score/xác suất mỗi lớp theo thứ tự, nhãn, entropy nếu dùng, ID và phiên bản. Feature dictionary cần được fit/mã hóa nhất quán.

Nếu tất cả feature của câu mới không được biết, phân bố có thể chủ yếu theo intercept. Không gọi đó là “không chắc theo nội dung” nếu chưa phân tích; raw output vẫn có thể tập trung dù không có bằng chứng từ câu.

## 19. Căn cứ lựa chọn

Chọn khi cần kết hợp feature và xác suất có điều kiện trong một baseline rõ. Nếu LR đã giải cùng bài toán, có thể giữ một triển khai và dùng tên thống nhất “LR/MaxEnt” trong bảng so sánh.

Nếu mục tiêu nghiên cứu là kiểm tra lợi ích feature thủ công, giữ cùng classifier và so các nhóm feature. Thiết kế đó tách tác động biểu diễn tốt hơn việc thay đồng thời mọi thành phần rồi gán cải thiện cho nguyên lý MaxEnt.

## 20. Trường hợp không nên sử dụng

Không dùng softmax một nhãn để mô tả nhiều cảm xúc có thể cùng hiện diện mà chưa thiết kế nhiệm vụ multi-label. Không dùng classifier độc lập token khi mục tiêu đòi chuỗi nhãn hợp lệ rồi nhận nó tương đương CRF.

Nếu nhãn ít, feature nhiều và gần tách hoàn hảo, cần regularization và kiểm tra ổn định. Không xem hệ số rất lớn hoặc entropy dự đoán gần 0 là bằng chứng mô hình biết chắc sự thật.

## 21. Các quyết định quan trọng khi thực hiện

Quyết định feature, regularization, tham số, lớp tham chiếu/cấu hình đa lớp và threshold. Nếu so với bài LR, xác định thành phần nào thật sự khác.

Chọn feature theo nhãn là bước học phải nằm trong train/CV. Với entropy dùng để chọn mẫu gán nhãn, cần lưu cách chọn và không dùng lại một tập được chọn thiên lệch như test đại diện quần thể mà không giải thích.

## 22. Giải thích công thức và ký hiệu

Với nhãn $y$ thuộc tập $\mathcal Y$:

$$P_\lambda(y\mid x)=\frac{\exp(\sum_j\lambda_j f_j(x,y))}{Z_\lambda(x)},
\quad Z_\lambda(x)=\sum_{y'\in\mathcal Y}\exp(\sum_j\lambda_j f_j(x,y')).$$

$f_j$ là feature, $\lambda_j$ là trọng số; $Z$ chuẩn hóa theo tất cả lớp. Nếu score lớp có dạng $w_y^Tx+b_y$, đây là softmax classifier. Nhị phân có thể rút về logistic theo hiệu hai score.

Entropy có điều kiện dưới phân bố đầu vào thực nghiệm $\tilde p(x)$ là:

$$H(p)=-\sum_x\tilde p(x)\sum_y p(y\mid x)\log p(y\mid x).$$

Trong bài toán cổ điển, tối đa hóa $H$ với các ràng buộc kỳ vọng feature và chuẩn hóa. Mã minh họa dùng conditional log loss có L2 qua `LogisticRegression`, không cài trực tiếp bài toán equality-constrained này. Dùng log tự nhiên cho đơn vị nat; đổi cơ số log đổi đơn vị entropy, không thay quy tắc argmax.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Score $(0,0,\log2)$ cho phân bố $(1/4,1/4,1/2)$, entropy $\approx1.0397$ nat, thấp hơn phân bố đồng đều ba lớp có entropy $\log3$. Đó là minh họa phân bố bị ràng buộc bởi score, không phải tối ưu MaxEnt đầy đủ từ ba con số đó.

Mã triển khai classifier ba lớp bằng multinomial logistic, đồng thời kiểm tra softmax trên score học được. Khác demo LR nhị phân trước, bài này minh họa ba lớp nên không dùng điểm giữa các demo để xếp hạng phương pháp.

```python
import numpy as np
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import f1_score

train = ["phòng sạch", "dịch vụ tốt", "sạch tốt",
         "phòng bẩn", "dịch vụ tệ", "bẩn tệ",
         "giờ nhận phòng", "địa chỉ khách sạn", "khách sạn có hồ bơi"]
y_train = ["positive"] * 3 + ["negative"] * 3 + ["neutral"] * 3
test = ["phòng sạch zzholdout", "dịch vụ tệ", "địa chỉ hồ bơi"]
y_test = ["positive", "negative", "neutral"]
model = Pipeline([
    ("count", CountVectorizer(ngram_range=(1, 2))),
    ("maxent", LogisticRegression(C=1.0, solver="lbfgs", max_iter=2000)),
])
model.fit(train, y_train)
scores = model.decision_function(test)
shifted = scores - scores.max(axis=1, keepdims=True)
manual = np.exp(shifted)
manual /= manual.sum(axis=1, keepdims=True)
prob = model.predict_proba(test)
assert scores.shape == (3, 3)
assert np.allclose(manual, prob) and np.allclose(prob.sum(axis=1), 1)
assert "zzholdout" not in model.named_steps["count"].vocabulary_
toy = np.array([1 / 4, 1 / 4, 1 / 2])
assert np.isclose(-(toy * np.log(toy)).sum(), 1.0397207708399179)
print("Classes:", model.named_steps["maxent"].classes_.tolist())
print("Pred:", model.predict(test).tolist())
print("Macro-F1 minh họa:", f1_score(y_test, model.predict(test), average="macro"))
```

Mã cần NumPy/scikit-learn; corpus tự soạn quá nhỏ để kết luận hiệu quả. Các câu trung tính phản ánh quy ước minh họa, không thay hướng dẫn nhãn thật cho nghiên cứu.

## 24. Tình huống sai và phản ví dụ

Một nghiên cứu đặt LR và MaxEnt vào bảng hai mô hình nhưng thực chất gọi cùng estimator với cùng feature/tham số. Kết quả giống nhau không chứng minh hai phương pháp độc lập hội tụ bằng chứng; đó có thể là cùng mô hình.

Một lỗi khác là nghĩ entropy dự đoán cao đồng nghĩa “khách trung tính”. Một câu khen và chê trộn có thể khiến mô hình không chắc giữa hai nhãn; cần định nghĩa nhãn, không đổi tên bất định thành trung tính.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đề xuất macro-F1, confusion matrix và từng lớp; với probability, thêm log loss và calibration. Kiểm tra ablation feature từ/cụm/phủ định bằng cùng split và lựa chọn tham số công bằng.

Kiểm tra feature rò rỉ, từ tên riêng và độ ổn định qua khách sạn/thời gian. Entropy có thể là chỉ báo phụ để phân tích, không thay metric độ đúng. Không dùng cùng test để quyết định feature và báo kết quả cuối.

## 26. Xử lý khi không đáp ứng yêu cầu

Không hội tụ hoặc hệ số rất lớn: xem separation, regularization, thang feature và solver. Probability kém: kiểm tra shift, nhãn và calibration. Feature thủ công không giúp: xem chúng có thực sự thêm thông tin hay chỉ lặp feature cũ.

Nếu các câu hỗn hợp thường sai, xem nhiệm vụ đa lớp có phù hợp không; có thể cần khía cạnh hoặc multi-label. Đổi tên LR thành MaxEnt không chữa lỗi biểu diễn hoặc thiết kế nhãn.

## 27. Phân biệt kết quả với kết luận

“Phân bố dự đoán có entropy cao” mô tả mô hình, không phải sự do dự của khách. “Trọng số một feature lớn” không chứng minh nó là nguyên nhân tạo hài lòng.

Một mô hình dự đoán tốt có thể phục vụ đo văn bản, nhưng suy rộng tỷ lệ/nguyên nhân cần thêm độ giá trị, mẫu và chiến lược thống kê. Nguyên lý MaxEnt không tự làm các kết luận xã hội trở nên khách quan.

## 28. Cách viết vào bài nghiên cứu

Nêu tên thống nhất “MaxEnt/log-linear” hoặc “multinomial LR” tùy bối cảnh, cùng phương trình, feature, regularization và solver. Nếu dùng LR làm triển khai, ghi rõ để người đọc không hiểu là tái lập thuật toán iterative scaling lịch sử.

Báo phần nào khác baseline LR nếu có. Với feature tùy chỉnh $f(x,y)$, mô tả cách tạo và thời điểm thông tin có sẵn. Không dùng một mô tả nguyên lý tổng quát thay cho cấu hình có thể chạy lại.

## 29. Khả năng tái lập và tính minh bạch

Lưu pipeline, mapping feature/lớp, objective, solver, phiên bản, tham số, threshold và chia tập. Nếu xuất entropy, ghi cơ số log và chính sách xác suất 0 trong tính toán.

Kho GitHub có thể chứa code và dữ liệu giả định. Các biểu diễn được học từ corpus ngoài cần nguồn/version và quyền dùng. Ghi rõ demo không bao gồm mô hình nhận diện khía cạnh hoặc chuỗi chỉ vì có feature NLP.

## 30. Đạo đức nghiên cứu liên quan

Feature và ràng buộc xuất phát từ mẫu có thể mang thiên lệch. “Ít giả định hơn” không đồng nghĩa ít thiên lệch hơn nếu thông tin đầu vào đã lệch hoặc bỏ sót một nhóm khách.

Không dùng entropy hoặc probability để suy đoán tâm lý cá nhân vượt khả năng phép đo. Khi công bố ví dụ, ẩn tên, thông tin đặt phòng và chi tiết nhận diện không cần cho câu hỏi nghiên cứu.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **MaxEnt có luôn cho các lớp bằng nhau không?** Không khi có ràng buộc phân biệt.
2. **MaxEnt classifier có thể tương đương LR không?** Có với dạng feature và cấu hình phù hợp.
3. **Bài tập tính:** score $(0,\log3)$ cho xác suất $(0.25,0.75)$. Cộng 10 vào cả hai score không đổi phân bố.
4. **Bài tập thiết kế:** liệt kê feature từ, bigram và phủ định có thể chồng lấn; nêu cách tránh feature chứa nhãn.
5. **Bài tập phản biện:** so phương trình và cấu hình của hai triển khai mang tên LR/MaxEnt trước khi nhận chúng là đối chứng độc lập.

## 32. Những điểm còn tranh luận

Các lựa chọn gồm ràng buộc/feature nào đáng tin, regularization, khả năng hiệu chuẩn và cách tách tên mô hình khỏi nguyên lý. Trong dữ liệu hữu hạn, việc khớp thống kê train thật sát có thể không giúp khái quát.

Nên đánh giá mục tiêu cụ thể thay vì dựa vào ý nghĩa triết lý của tên gọi. Việc có một phân bố hợp lệ và nền tảng toán học đẹp không tự chứng minh phép đo đúng khái niệm cảm xúc hoặc công bằng giữa nhóm khách.
