# SVM — Máy vector hỗ trợ trong phân loại văn bản

> **Nhánh:** MACHINE LEARNING → Supervised: Linear. **Nguồn:** dòng 15 bảng Excel, SVM. **Phạm vi:** phân loại bằng SVM tuyến tính; có giới thiệu kernel để phân biệt. **Đối chiếu nguồn:** 28/10/2026. Ví dụ và dữ liệu khách sạn là tự soạn. Mã đã được kiểm tra với scikit-learn 1.8.0; tài liệu trực tuyến có thể thuộc phiên bản mới hơn.

| Trường | Nội dung |
|---|---|
| Bản chất | Bộ phân loại có giám sát; phải nhận đặc trưng số thay vì hiểu văn bản thô trực tiếp |
| Mốc tham khảo | Cortes & Vapnik, 1995 về support-vector networks; Joachims, 1998 về phân loại văn bản [S1–S2] |
| Đầu vào | Ma trận đặc trưng, nhãn và cấu hình tối ưu |
| Đầu ra | Nhãn, điểm quyết định, hệ số ở trường hợp tuyến tính; xác suất cần phương án bổ sung phù hợp |
| Tiếng Việt | Có thể dùng với biểu diễn tiếng Việt; chất lượng phụ thuộc dữ liệu, nhãn và đặc trưng |
| Nhãn và tính toán | Cần nhãn huấn luyện. Linear SVM với TF-IDF sparse thường có thể dùng CPU; chi phí kernel khác |
| Phần mềm | scikit-learn: `LinearSVC`, `SVC(kernel="linear")`; hai lớp không phải cùng triển khai [S3–S4] |

## 1. Định nghĩa học thuật

Support Vector Machine là họ phương pháp học tìm hàm quyết định dựa trên nguyên lý biên phân cách và kiểm soát sai lệch huấn luyện. Với SVM tuyến tính, điểm quyết định có dạng $f(x)=w^Tx+b$, còn nhãn phụ thuộc phía của siêu phẳng mà vector nằm trên.

SVM không giới hạn ở tuyến tính: kernel cho phép một biểu diễn khác và ranh giới có thể phi tuyến trong không gian đầu vào. Do vị trí trong bảng, bài này tập trung biến thể tuyến tính, không gọi mọi SVM là mô hình tuyến tính trên đặc trưng gốc [S1, S3].

## 2. Giải thích đơn giản

Hình dung các đánh giá đã được biến thành các điểm trên bản đồ đặc trưng. SVM tìm một đường hoặc mặt phân chia, đồng thời cân nhắc khoảng trống quanh ranh giới. Với dữ liệu không tách hoàn hảo, nó chấp nhận một số vi phạm để tránh bám quá chặt các mẫu khó hoặc nhiễu.

Trong văn bản, bản đồ có thể có hàng nghìn chiều là từ/cụm. Mô hình học tổ hợp trọng số từ nhãn, khác việc tra một từ điển điểm khen/chê dựng sẵn.

## 3. Mục đích và vấn đề giải quyết

Đề xuất nghiên cứu khách sạn: dự đoán một đánh giá tích cực hay tiêu cực từ văn bản có nhãn. SVM có thể là baseline để so với Logistic Regression hoặc mô hình ngữ cảnh trên cùng dữ liệu.

Nếu mục tiêu là khía cạnh, có thể xây mô hình theo từng cặp văn bản–khía cạnh, nhưng phải thiết kế nhãn và đặc trưng tương ứng. Dùng SVM toàn văn không tự tạo ra một hệ ABSA biết lời khen đang hướng vào phòng hay nhân viên.

## 4. Điều kiện và giả định sử dụng

Cần tập nhãn được định nghĩa nhất quán, đặc trưng có thể học quan hệ với nhãn và tập kiểm tra phù hợp mục tiêu áp dụng. Soft-margin không yêu cầu dữ liệu tách tuyến tính hoàn toàn; tuy nhiên, biểu diễn quá thiếu thông tin sẽ hạn chế chất lượng.

Giả định chuyển từ train sang ứng dụng cần được kiểm tra: khác thời gian, nền tảng hoặc khách sạn có thể làm phân bố thay đổi. Nếu nhiều phản hồi trùng/mẫu viết sẵn xuất hiện hai tập, kết quả có thể quá lạc quan.

## 5. Cơ chế hoạt động bên trong

Pipeline minh họa: văn bản → TF-IDF → học trọng số/intercept bằng mục tiêu SVM → tính điểm trên văn bản mới → chọn nhãn. Phần học cân bằng độ phức tạp của trọng số với các mẫu vi phạm margin.

Ở thời điểm dự đoán, mô hình tuyến tính chỉ cần tích trọng số với vector. Kernel SVM có cách biểu diễn quyết định khác liên quan mẫu hỗ trợ; không suy rằng mọi API đều cung cấp danh sách support vector giống nhau. `LinearSVC` không có các thuộc tính hỗ trợ như `SVC` [S3–S4].

## 6. Quy trình áp dụng trong nghiên cứu

1. Xác định nhãn, đơn vị và cách xử lý đánh giá hỗn hợp/trung tính.
2. Loại trùng và chia train/development/test theo mục tiêu, giữ nhóm khách sạn nếu cần.
3. Đặt vectorizer và mô hình trong pipeline; chỉ fit bằng dữ liệu train ở từng lượt.
4. So baseline đa số, TF-IDF + SVM và các đối chứng phù hợp.
5. Chọn `C`, đặc trưng và class weight qua development hoặc CV chỉ trên train.
6. Đóng băng quyết định, đánh giá test; báo lỗi từng lớp và từng nhóm khó.
7. Nếu cần xác suất, thêm hiệu chuẩn bằng dữ liệu/quy trình tách biệt phù hợp.

Không fit TF-IDF trên toàn corpus trước CV rồi coi pipeline đã tránh rò rỉ; cả vocabulary và IDF cũng là thành phần được học [S5].

## 7. Ví dụ minh họa xuyên suốt

Giả định có các phản hồi “phòng sạch”, “dịch vụ tốt” gán tích cực và “phòng bẩn”, “dịch vụ tệ” gán tiêu cực. TF-IDF chuyển chúng thành vector, SVM học các trọng số giúp phân biệt.

Khi gặp “phòng không sạch”, unigram có thể vẫn kích hoạt trọng số của “sạch”. Thêm bigram hoặc đặc trưng phủ định là giả thuyết cần thử. Nhãn không tự đúng chỉ vì mô hình có khả năng học từ dữ liệu; dữ liệu và biểu diễn phải chứa tín hiệu phù hợp.

## 8. Kết quả và cách diễn giải

Trong bài toán nhị phân scikit-learn, điểm quyết định dương thường hướng đến `classes_[1]`; phải kiểm tra thứ tự lớp của mô hình thay vì mặc định số dương luôn là “tích cực”. Giá trị `decision_function` không phải xác suất.

Hệ số dương của một feature cho thấy tăng feature đó làm điểm tuyến tính tăng khi giữ các feature khác theo phép tính. Nó không chứng minh từ đó mang ý khen trong mọi ngữ cảnh hoặc gây ra sự hài lòng. Với đa lớp, cần đọc từng bộ hệ số theo chiến lược và lớp.

## 9. Điểm mạnh, điểm yếu và giới hạn

Lợi thế thực hành là baseline tuyến tính dùng được biểu diễn sparse và có thể kiểm tra trọng số/đóng góp. Hạn chế là phi tuyến và ngữ cảnh dài phụ thuộc chất lượng biểu diễn; không có xác suất trực tiếp từ `LinearSVC`.

SVM có thể học dấu hiệu ngoài mục tiêu, như tên khách sạn chỉ xuất hiện với một nhãn. Điểm số tốt trên chia ngẫu nhiên chưa bảo đảm dùng được cho khách sạn mới. Khả năng học nhiều chiều không loại bỏ nhu cầu chọn tham số và kiểm định chuyển miền.

## 10. So sánh với phương pháp gần nhau

| Phương pháp | Điểm khác |
|---|---|
| Logistic Regression | Tối ưu log loss, xuất ước lượng xác suất theo mô hình |
| Perceptron | Luật cập nhật dựa trên mẫu sai/ranh giới, không tối ưu margin như soft-margin SVM |
| Kernel SVM | Có thể phi tuyến trong đặc trưng gốc; chi phí và tham số khác |
| Từ điển cảm xúc | Điểm gắn theo tài nguyên/luật thay vì học toàn bộ trọng số từ nhãn văn bản |

So sánh cần giữ cùng tập chia và ngân sách chọn tham số hợp lý, không so SVM được tối ưu kỹ với mô hình khác dùng cấu hình tùy ý.

## 11. Sai lầm thường gặp và cách khắc phục

- Đọc decision score 0.8 như xác suất 80%: cần hiệu chuẩn riêng nếu mục tiêu đòi hỏi xác suất.
- Cho rằng tăng `C` là tăng regularization: với cách tham số thông dụng, `C` lớn làm phạt vi phạm mạnh hơn tương đối.
- Fit vectorizer trước khi chia: đặt trong pipeline và CV đúng tập.
- Nhầm `SVC()` mặc định là SVM tuyến tính: kiểm tra kernel và tên estimator.
- Dùng margin nhỏ làm nhãn trung tính: điểm gần ranh giới chỉ là khó phân biệt hai lớp, không tự tạo lớp trung tính.

## 12. Tài liệu tham khảo

- **[S1]** Cortes, C., & Vapnik, V. (1995). *Support-vector networks*. Machine Learning, 20, 273–297. [Trang công trình](https://research.google/pubs/support-vector-networks/). [DOI](https://doi.org/10.1007/BF00994018).
- **[S2]** Joachims, T. (1998). *Text Categorization with Support Vector Machines: Learning with Many Relevant Features*. [Bản tác giả](https://www.cs.cornell.edu/people/tj/publications/joachims_98a.pdf).
- **[S3]** scikit-learn. [Support Vector Machines](https://scikit-learn.org/stable/modules/svm.html).
- **[S4]** scikit-learn. [LinearSVC API](https://scikit-learn.org/stable/modules/generated/sklearn.svm.LinearSVC.html).
- **[S5]** scikit-learn. [Common pitfalls: data leakage](https://scikit-learn.org/stable/common_pitfalls.html).
- **[S6]** scikit-learn. [Probability calibration](https://scikit-learn.org/stable/modules/calibration.html).

Nguồn giới thiệu phương pháp và triển khai; các ví dụ khách sạn và quy trình kiểm định đề xuất trong bài không phải kết quả được đo trong các nguồn trên.

## 13. Vị trí trong toàn bộ nghiên cứu

SVM nằm sau biểu diễn đặc trưng và trước đánh giá hoặc sử dụng nhãn dự đoán cho phân tích tiếp. Chất lượng phân loại phải được kiểm tra trước khi dùng nhãn máy để mô tả tỷ lệ phản hồi tích cực của ngành/địa phương.

Nếu mục tiêu là nghiên cứu một quan hệ quản trị, SVM chỉ tạo biến dự đoán. Nó không thay thế thiết kế lấy mẫu, kiểm định độ giá trị hoặc chiến lược nhận diện nhân quả.

## 14. Kiến thức nền cần biết trước

Cần hiểu vector, tích vô hướng, siêu phẳng, regularization, loss và margin. **Functional margin** nhị phân là $y_i(w^Tx_i+b)$; khoảng cách hình học còn chia cho chuẩn $\|w\|$.

Hiểu sparse matrix giúp tránh chuyển TF-IDF lớn thành ma trận dày không cần thiết. Cũng cần phân biệt **tham số học** như $w,b$ với **siêu tham số** như `C`, được chọn bằng development/CV.

## 15. Nền tảng lý thuyết và logic hình thành

Logic của margin là không chỉ tìm một ranh giới phân đúng train mà còn kiểm soát cách phân tách. Khi dữ liệu có nhiễu hoặc chồng lấn, soft-margin cho phép đánh đổi giữa vi phạm và độ phức tạp.

Đây là động cơ học thống kê, không bảo đảm ranh giới phản ánh khái niệm “hài lòng” đúng nghĩa. Nếu nhãn được tạo từ sao hoặc chủ đề thiên lệch, mô hình tối ưu đúng hàm mục tiêu vẫn có thể học điều khác với cấu trúc nghiên cứu mong muốn.

## 16. Các thành phần cấu tạo

Một hệ cần hướng dẫn nhãn, vectorizer, mô hình, cấu hình chọn tham số, quy tắc nhãn và bộ đánh giá. Tùy mục tiêu có thêm class weighting hoặc calibration.

Tách riêng các thành phần trong nhật ký. Nếu đổi từ unigram sang bigram và đổi `C` cùng lúc, chưa biết phần nào làm kết quả đổi; có thể dùng ablation để kiểm tra đóng góp khi điều đó quan trọng với câu hỏi nghiên cứu.

## 17. Các loại và biến thể

Phân biệt hard-margin, soft-margin, linear kernel và kernel phi tuyến. Bài này không hướng dẫn SVR hay One-Class SVM vì chúng giải nhiệm vụ khác.

Trong scikit-learn, `LinearSVC` thường dùng squared hinge, còn `SVC(kernel="linear")` là triển khai khác [S3–S4]. Với đa lớp còn có one-vs-rest hoặc one-vs-one tùy estimator. Nêu đúng biến thể thay vì dùng một công thức minh họa cho mọi triển khai.

## 18. Đầu vào và đầu ra

Đầu vào số có dạng $X\in\mathbb R^{n\times p}$ và vector nhãn $y$. Với văn bản, cần lưu mapping cột feature về từ/cụm. Đầu ra nghiên cứu nên giữ `review_id`, nhãn thật, nhãn máy, decision score, fold và phiên bản mô hình.

Văn bản chỉ chứa từ chưa thấy có thể biến thành vector 0; dự đoán khi đó phụ thuộc intercept/quy tắc của hệ. Cần phát hiện để không diễn giải như mô hình đã hiểu đầy đủ nội dung.

## 19. Căn cứ lựa chọn

SVM tuyến tính đáng thử khi cần baseline có giám sát cho đặc trưng sparse và có đủ nhãn để đánh giá. Quyết định dựa trên macro-F1, lỗi lớp quan trọng, chi phí và nhu cầu xác suất.

Nếu LR đạt tương đương nhưng dễ đáp ứng yêu cầu xác suất hơn, đó là yếu tố lựa chọn. Nếu SVM tốt hơn một chút nhưng không chuyển miền, cần ưu tiên tiêu chuẩn gắn với mục tiêu ứng dụng thay vì điểm trung bình duy nhất.

## 20. Trường hợp không nên sử dụng

Không dùng kernel tốn kém mặc định cho corpus lớn khi chưa thử baseline tuyến tính và đo tài nguyên. Không dùng mô hình chỉ học hai lớp để đưa ra nhãn trung tính mà không bổ sung thiết kế.

Khi không có nhãn đáng tin hoặc nhiệm vụ cần hiểu ngữ cảnh dài ngoài biểu diễn hiện có, cần bổ sung dữ liệu/biểu diễn. Không thể giải bằng tăng `C` hoặc số vòng tối ưu đơn thuần.

## 21. Các quyết định quan trọng khi thực hiện

Quyết định loại feature, `C`, loss, class weight, phép chia dữ liệu và cách chọn threshold nếu có. Với đặc trưng số kết hợp TF-IDF, cần kiểm soát thang đo và chỉ fit bước học thang trên train.

Nếu ưu tiên phát hiện phàn nàn, chọn trade-off precision/recall dựa trên development và chi phí sai; không chọn sau khi nhìn test. Class weight là một lựa chọn huấn luyện, không thay thế việc báo riêng chất lượng từng lớp.

## 22. Giải thích công thức và ký hiệu

Một dạng soft-margin tuyến tính với hinge loss để giảng giải là:

$$\min_{w,b}\frac12\|w\|^2+C\sum_{i=1}^{n}\max(0,1-y_i(w^Tx_i+b)).$$

$x_i$ là vector, $y_i\in\{-1,+1\}$, $C>0$ cân bằng phạt vi phạm với chuẩn trọng số. Điểm $f(x)=w^Tx+b$; quyết định nhị phân theo dấu với chính sách hòa được quy định. Functional margin lớn hơn hoặc bằng 1 có hinge loss 0 trong dạng này.

Ví dụ mã dùng `LinearSVC(loss="squared_hinge")`: phần phạt tương ứng bình phương $\max(0,1-y_if_i)^2$, và chi tiết xử lý intercept tùy triển khai [S4]. Vì vậy, công thức hinge trên là nền tảng giải thích, không khẳng định trùng hoàn toàn objective/API trong mã.

Khoảng cách có dấu đến siêu phẳng là $f(x)/\|w\|$ nếu $w\ne0$. Raw score không có thang xác suất; ngay cả khoảng cách cũng không tự cho xác suất đúng.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Giả định vector hai feature là “sạch”, “bẩn”; $w=(1,-1)$, $b=0$, $x=(0.8,0.1)$. Khi đó $f=0.7$. Với $y=+1$, hinge loss là $0.3$, squared hinge là $0.09$. Các số tự đặt không phải hệ số học từ ví dụ sau.

Mã cần Python và scikit-learn. Dữ liệu cực nhỏ chỉ minh họa pipeline, không dùng để tuyên bố hiệu năng. Nhãn `1` tích cực, `0` tiêu cực; không có lớp trung tính trong demo.

```python
import numpy as np
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.svm import LinearSVC
from sklearn.metrics import f1_score

train = ["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
         "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
         "dịch vụ tệ bẩn", "khách sạn bẩn tệ"]
y_train = [1, 1, 1, 1, 0, 0, 0, 0]
test = ["phòng sạch zzholdout", "dịch vụ tốt", "phòng bẩn", "dịch vụ tệ"]
y_test = [1, 1, 0, 0]

model = Pipeline([
    ("tfidf", TfidfVectorizer(ngram_range=(1, 2), lowercase=True)),
    ("clf", LinearSVC(C=1.0, loss="squared_hinge", dual=True,
                      max_iter=10000, random_state=42)),
])
model.fit(train, y_train)
pred = model.predict(test)
scores = model.decision_function(test)
assert "zzholdout" not in model.named_steps["tfidf"].vocabulary_
assert scores.shape == (len(test),) and np.isfinite(scores).all()
assert set(model.named_steps["clf"].classes_) == {0, 1}
print("Pred:", pred.tolist())
print("Macro-F1 minh họa:", f1_score(y_test, pred, average="macro"))
```

`zzholdout` là token giả để kiểm tra test không đi vào vocabulary. Pipeline không tự chống mọi rò rỉ: vẫn phải chia đúng nhóm, kiểm soát trùng và chọn tham số ngoài test. Với tiếng Việt thật, cần thẩm định tokenization, phủ định và nhãn.

## 24. Tình huống sai và phản ví dụ

Nếu mọi đánh giá từ khách sạn A được gán tích cực, mô hình có thể dùng tên A thay nội dung. Chia ngẫu nhiên làm nó trông rất tốt, nhưng sang khách sạn B sẽ không còn dấu hiệu đó.

“Phòng sạch nhưng nhân viên tệ” cũng không có một nhãn duy nhất đúng tự nhiên nếu nhiệm vụ chưa quy định đánh giá hỗn hợp. Cần sửa thiết kế nhãn trước khi đổ lỗi hoàn toàn cho thuật toán.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đề xuất macro-F1, precision/recall từng lớp, confusion matrix và kích thước các nhóm. So theo thời gian/khách sạn khi phù hợp. Chọn hyperparameter bằng CV trên train, giữ pipeline bên trong mỗi fold; nested CV hữu ích nếu phải vừa chọn vừa ước lượng hiệu năng mà chưa có test riêng.

Kiểm tra nhạy theo `C`, feature và split, không lặp vô hạn khi đã có bằng chứng đủ. Nếu hiệu chuẩn xác suất, đánh giá thêm log loss/Brier và calibration trên dữ liệu độc lập [S6].

## 26. Xử lý khi không đáp ứng yêu cầu

Train tốt/test kém: xem leakage, lệch miền, overfit và nhãn. Cả train/test kém: xem biểu diễn, nhãn, regularization và lỗi tối ưu. Có cảnh báo chưa hội tụ: kiểm tra thang, `max_iter`, tolerance và cấu hình solver; không chỉ bỏ cảnh báo.

Nếu lỗi tập trung ở phủ định/khía cạnh, thử đặc trưng hoặc mô hình đúng vấn đề trên development. Không thay đổi bằng cách đọc test rồi báo lại như đánh giá độc lập.

## 27. Phân biệt kết quả với kết luận

“SVM gán 70% mẫu tích cực” chưa có nghĩa 70% toàn bộ khách hài lòng. Cần độ đúng mô hình, cách lấy mẫu, định nghĩa nhãn và sai số theo nhóm.

“Từ sạch có trọng số dương” không chứng minh vệ sinh gây hài lòng theo một độ lớn tương ứng. Hệ số phân loại là tham số dự đoán trong hệ đặc trưng, không tự là hệ số tác động nhân quả.

## 28. Cách viết vào bài nghiên cứu

Ghi estimator, version, feature, loss, `C`, class weight, chiến lược đa lớp, cách chia và quy trình chọn tham số. Mẫu: “TF-IDF được fit trong từng fold huấn luyện; [estimator] được chọn theo [metric], sau đó đánh giá một lần trên [test].”

Phần kết quả báo đối chứng, lỗi và bất định phù hợp. Nếu cần probability, mô tả calibration và nguồn dữ liệu riêng; không đổi tên decision score thành probability trong bảng.

## 29. Khả năng tái lập và tính minh bạch

Lưu cả vectorizer và classifier, mapping nhãn, cấu hình split, phiên bản thư viện và random seed. Lưu riêng bản dữ liệu/hướng dẫn tạo dữ liệu được phép chia sẻ; các tệp model cần đi kèm hướng dẫn môi trường.

Có thể công khai trọng số feature và ví dụ lỗi đã ẩn danh. Khi dùng tệp mô hình pickle/joblib từ bên ngoài, chỉ nạp nguồn tin cậy; kho tài liệu không cần phân phối dữ liệu khách riêng tư để minh họa pipeline.

## 30. Đạo đức nghiên cứu liên quan

Kiểm tra sai số theo ngôn ngữ, cách viết và nhóm mẫu; không xem một nhóm ít được biểu diễn trong train là có thái độ kém hơn chỉ vì mô hình gán âm nhiều hơn.

Phản hồi có thể chứa tên nhân viên/mã đặt phòng. Cần ẩn thông tin trước công khai và tránh dùng nhãn máy đơn độc để xếp hạng cá nhân. Dữ liệu có nhãn vẫn có thể mang thiên lệch của người gán nhãn.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **SVM có luôn tuyến tính trên đầu vào gốc không?** Không; bài này chỉ tập trung tuyến tính.
2. **Decision score 0.7 có phải xác suất 70% không?** Không.
3. **Bài tập tính:** giữ $w=(1,-1),b=0$, đổi $x=(0.1,0.8),y=-1$. Điểm là −0.7, hinge loss vẫn 0.3.
4. **Bài tập thiết kế:** so unigram và unigram+bigram, giữ cùng split. Đọc lỗi phủ định và kiểm tra feature tên khách sạn.
5. **Bài tập phản biện:** giải thích vì sao vectorizer fit toàn corpus trước CV vẫn có leakage dù nhãn test chưa được dùng.

## 32. Những điểm còn tranh luận

Các lựa chọn cần cân nhắc là biểu diễn, margin/loss, khả năng hiệu chuẩn và chuyển miền. Không có một thuật toán luôn đứng đầu mọi corpus; kết quả cần gắn với quy trình chọn tham số và thiết kế đánh giá.

Tính dễ đọc của trọng số không đồng nghĩa giải thích toàn bộ ý nghĩa hoặc tác động xã hội. Một baseline tuyến tính có thể rất hữu ích, nhưng vẫn cần kiểm tra nó đang dựa vào nội dung có ý nghĩa nghiên cứu hay dấu hiệu ngẫu nhiên của tập dữ liệu.
