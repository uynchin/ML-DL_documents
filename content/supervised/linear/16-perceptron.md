# Perceptron — Học bộ phân loại tuyến tính qua cập nhật sai số

> **Nhánh:** MACHINE LEARNING → Supervised: Linear. **Nguồn:** dòng 17 bảng Excel. **Đối chiếu nguồn:** 28/10/2026. Bài tập trung perceptron tuyến tính cơ bản, phân biệt với mạng nhiều tầng. Ví dụ, nhãn và phép tính tự soạn. Mã kiểm tra với scikit-learn 1.8.0; đối chiếu API khi đổi phiên bản.

| Trường | Nội dung |
|---|---|
| Bản chất | Bộ phân loại tuyến tính có giám sát; có thể học theo từng mẫu |
| Mốc | Rosenblatt, 1958 là công trình nền tảng [S1]; mã hiện đại không tái hiện mọi chi tiết mô hình sinh học trong bài lịch sử |
| Đầu vào | Vector đặc trưng, nhãn, tốc độ học, thứ tự mẫu và điều kiện dừng |
| Đầu ra | Nhãn, decision score, trọng số và intercept; không có xác suất logistic mặc định |
| Tiếng Việt | Dùng được với biểu diễn phù hợp; không tự có năng lực ngôn ngữ từ tên thuật toán |
| Nhãn/tài nguyên | Cần nhãn; cập nhật tuyến tính sparse có thể dùng CPU; chất lượng phụ thuộc dữ liệu/biểu diễn |
| Phần mềm | `sklearn.linear_model.Perceptron`; có cách fit theo đợt hoặc cập nhật qua `partial_fit` theo quy định API [S2] |

## 1. Định nghĩa học thuật

Perceptron tuyến tính học một hàm điểm $f(x)=w^Tx+b$ và quyết định lớp theo ranh giới tuyến tính. Luật nhị phân cơ bản cập nhật khi mẫu bị phân sai hoặc nằm trên ranh giới, theo quy ước $y_if(x_i)\le0$.

Tên này gắn với công trình Rosenblatt năm 1958 [S1]. Trong bài, perceptron là bộ phân loại tuyến tính thực hành; **Multilayer Perceptron — MLP** có các tầng ẩn và cơ chế học khác, không phải cùng mô hình chỉ tăng số lần lặp.

## 2. Giải thích đơn giản

Hình dung một người đang học cách chấm đánh giá. Khi gặp mẫu sai, họ điều chỉnh trọng số theo hướng giúp mẫu đó được chấm đúng hơn. Mẫu đã nằm đúng phía theo điều kiện cơ bản thì không gây cập nhật.

Quá trình lặp qua dữ liệu. Một chỉnh sửa có thể giúp mẫu hiện tại nhưng làm mẫu trước khó hơn; nếu dữ liệu mâu thuẫn hoặc không tách được tuyến tính, không có bảo đảm cuối cùng sẽ hết sai.

## 3. Mục đích và vấn đề giải quyết

Perceptron là baseline phù hợp để học cơ chế cập nhật và so hiệu quả một bộ tuyến tính đơn giản trên đánh giá khách sạn. Có thể kiểm tra chi phí huấn luyện, số vòng và chất lượng so với SVM/LR.

Khả năng cập nhật theo mẫu có thể hữu ích khi dữ liệu có nhãn đến dần, nhưng mô hình online cần thiết kế đánh giá theo thời gian. Không thể cập nhật bằng nhãn chưa được biết rồi dùng kết quả đó như dự đoán trước thời điểm nhãn có sẵn.

## 4. Điều kiện và giả định sử dụng

Cần vector có thang phù hợp, nhãn rõ và quy định dừng. Với định lý hội tụ cơ bản, dữ liệu hữu hạn cần tách tuyến tính với margin dương và luật học đáp ứng các điều kiện tương ứng. Đây không phải bảo đảm cho mọi corpus thực hoặc mọi biến thể regularized.

Đánh giá khách sạn có nhiễu, ý kiến hỗn hợp và khác biệt người gán nhãn; cần dự kiến khả năng dữ liệu không tách được trong biểu diễn đang dùng, thay vì đòi mô hình phải hội tụ về 0 lỗi.

## 5. Cơ chế hoạt động bên trong

Quy trình cơ bản: khởi tạo $w,b$ → lấy một mẫu → tính điểm và signed margin → nếu vi phạm điều kiện thì cập nhật → chuyển mẫu → lặp theo epoch → dừng theo quy tắc đã chọn.

Triển khai scikit-learn là wrapper của SGD với perceptron loss và learning rate cố định theo cấu hình [S2]. Nếu bật regularization hoặc early stopping, hành vi còn có thành phần ngoài luật tay cơ bản. Không giả định một ví dụ một bước mô tả đủ mọi cấu hình API.

## 6. Quy trình áp dụng trong nghiên cứu

1. Xác định nhãn và cách xử lý trung tính/hỗn hợp.
2. Loại trùng, chia tập theo mục tiêu khái quát hóa.
3. Fit biểu diễn trong pipeline trên train; giữ nhãn test ngoài học.
4. Xác định tốc độ học, shuffle, số epoch, regularization và dừng.
5. Chọn cấu hình qua development/CV, không nhìn test để chọn epoch.
6. Đánh giá theo lớp, split và random seed khi có nguồn ngẫu nhiên.
7. Với online, dự đoán trước, chấm khi nhãn xuất hiện rồi mới cập nhật nếu đúng thiết kế.

Đừng refit vocabulary tùy ý giữa các lần `partial_fit`: thay đổi cột feature sẽ làm trọng số cũ không còn đúng nghĩa.

## 7. Ví dụ minh họa xuyên suốt

Giả định feature hai chiều là tín hiệu “sạch” và “bẩn”. Ban đầu $w=(0,0),b=0$. Gặp mẫu tích cực $x=(1,0),y=+1$, luật $yf\le0$ cho cập nhật vì điểm bằng 0.

Với tốc độ học 1, trọng số thành $(1,0)$ và intercept thành 1. Đó là một bước minh họa, không phải mô hình hoàn chỉnh. Cần xem tiếp mẫu tiêu cực để thấy một điều chỉnh ảnh hưởng các mẫu sau thế nào.

## 8. Kết quả và cách diễn giải

Đầu ra gồm nhãn và điểm tuyến tính. Trong API nhị phân, kiểm tra `classes_` để biết dấu điểm hướng về lớp nào. Decision score không phải xác suất và không có ý nghĩa “khách tích cực 90%” chỉ vì số bằng 0.9.

Trọng số thể hiện đóng góp feature theo mô hình hiện tại, có thể đổi theo thứ tự mẫu. Số epoch đã chạy hoặc đạt điều kiện dừng không đồng nghĩa đã tìm lời giải tối ưu toàn cục về F1 hay hiểu đúng mọi câu.

## 9. Điểm mạnh, điểm yếu và giới hạn

Lợi thế là luật cập nhật dễ theo dõi, có thể dùng dữ liệu sparse và cập nhật theo đợt. Nó giúp làm rõ khác biệt giữa chấm điểm, dự đoán và học từ lỗi.

Hạn chế là nhạy thứ tự/nhiễu, không bảo đảm margin tối ưu, không có xác suất chuẩn và không bảo đảm hết lỗi trên dữ liệu không tách tuyến tính. Chu kỳ hoặc dao động không có nghĩa chỉ cần tăng epoch là sẽ giải được.

## 10. So sánh với phương pháp gần nhau

| Phương pháp | Điểm khác |
|---|---|
| SVM tuyến tính | Tối ưu mục tiêu margin có regularization/vi phạm, không phải cùng luật cập nhật |
| Logistic Regression | Log loss và xác suất logistic theo mô hình |
| SGDClassifier | Khung huấn luyện hỗ trợ nhiều loss; không phải một mô hình duy nhất |
| Averaged Perceptron | Trung bình trọng số qua cập nhật theo thiết kế, có thể ổn định hơn nhưng không bảo đảm luôn thắng |
| MLP | Mạng nhiều tầng, có thể phi tuyến nhờ tầng ẩn/activation |

Sử dụng SGD không làm mọi mô hình có cùng loss hay cùng xác suất. Cần báo cả objective và cách tối ưu.

## 11. Sai lầm thường gặp và cách khắc phục

- Nhầm perceptron với MLP: nêu rõ có hay không tầng ẩn.
- Tin mọi dữ liệu đều hội tụ: kiểm tra giả định tuyến tính và nhiễu.
- Đọc raw score như probability: dùng đại lượng đúng, hiệu chuẩn riêng nếu cần.
- Thay vocabulary trong online mà giữ trọng số: cố định không gian feature hoặc thiết kế cập nhật biểu diễn rõ ràng.
- Dừng dựa trên test: dùng development và khóa test cho đánh giá cuối.

## 12. Tài liệu tham khảo

- **[S1]** Rosenblatt, F. (1958). *The perceptron: A probabilistic model for information storage and organization in the brain*. Psychological Review, 65(6), 386–408. [DOI công trình](https://doi.org/10.1037/h0042519).
- **[S2]** scikit-learn. [Perceptron API](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.Perceptron.html).
- **[S3]** scikit-learn. [Linear Models](https://scikit-learn.org/stable/modules/linear_model.html).
- **[S4]** scikit-learn. [Stochastic Gradient Descent](https://scikit-learn.org/stable/modules/sgd.html).
- **[S5]** scikit-learn. [Common pitfalls](https://scikit-learn.org/stable/common_pitfalls.html).

S1 là mốc lịch sử, không dùng từ “probabilistic” trong tên bài để suy rằng API Perceptron hiện đại xuất xác suất. Các số, corpus và thí nghiệm khách sạn bên dưới là minh họa riêng, không phải benchmark từ các nguồn này.

## 13. Vị trí trong toàn bộ nghiên cứu

Perceptron nằm ở bước học bộ phân loại sau khi có biểu diễn và nhãn. Nếu dùng nhãn dự đoán để phân tích cảm xúc theo tháng, cần đánh giá độ đúng và sự thay đổi phân bố trước.

Trong online, trật tự dữ liệu là một phần thiết kế nghiên cứu: thông tin nào đã có ở thời điểm nào quyết định cập nhật có hợp lệ hay không. Không coi mọi cách chia ngẫu nhiên là tương đương đánh giá hệ hoạt động theo thời gian.

## 14. Kiến thức nền cần biết trước

Cần hiểu vector, tích vô hướng, nhãn ±1, learning rate, epoch, shuffle và phân tách tuyến tính. Một epoch là một lượt qua tập train, không phải một mẫu.

Phân biệt **random seed** kiểm soát ngẫu nhiên với **seed word** trong xây lexicon. Phân biệt **online** cập nhật dữ liệu theo trình tự với việc ứng dụng có kết nối internet; đây là đặc tính học, không phải yêu cầu mạng.

## 15. Nền tảng lý thuyết và logic hình thành

Luật học điều chỉnh vector trọng số theo nhãn và hướng vector mẫu khi signed margin không dương. Với mẫu bị cập nhật, điểm theo nhãn trên chính mẫu đó tăng theo phép toán nếu learning rate dương và feature hữu hạn.

Điều này không bảo đảm mọi mẫu khác cũng cải thiện. Bảo đảm hội tụ cổ điển gắn với điều kiện phân tách; trên dữ liệu thực, nên đo trade-off và sử dụng quy tắc dừng chứ không suy diễn từ ví dụ một bước.

## 16. Các thành phần cấu tạo

Hệ gồm biểu diễn, trọng số/intercept, quy tắc quyết định, điều kiện cập nhật, learning rate, thứ tự mẫu và quy tắc dừng. Nếu thêm penalty, averaging hoặc early stopping thì cần cấu hình riêng.

Nên lưu số vòng thực chạy, cảnh báo và cấu hình shuffle. Chỉ lưu model cuối chưa đủ để hiểu khác biệt giữa hai lần chạy nếu quy trình học dùng thứ tự hoặc mẫu development khác.

## 17. Các loại và biến thể

Có perceptron nhị phân cơ bản, mở rộng đa lớp, averaged/voted perceptron và biến thể kernel. Bài này tập trung tuyến tính, không triển khai tất cả các biến thể.

Trong scikit-learn, tham số penalty có thể bổ sung regularization và class weight có thể đổi đóng góp lớp [S2]. Các thiết lập đó không được gộp vào định lý cơ bản một cách tự động. Thêm tầng ẩn tạo một họ mô hình khác, không chỉ biến thể của cùng luật tuyến tính.

## 18. Đầu vào và đầu ra

Đầu vào là $X$ và nhãn, cộng trạng thái mô hình nếu cập nhật dần. Đầu ra lưu nhãn, score, hệ số, intercept và version. Với online, lưu timestamp, phiên bản trước/sau và thời điểm nhãn được biết.

Nếu dùng TF-IDF, vocabulary và IDF đã fit phải đi cùng classifier. Nếu vector mới không có feature nào đã biết, cần ghi trạng thái thiếu tín hiệu; dự đoán từ intercept không phải bằng chứng mô hình hiểu nội dung.

## 19. Căn cứ lựa chọn

Perceptron đáng thử khi cần baseline đơn giản, dễ theo dõi cập nhật hoặc học tăng dần. Nên so với LR/SVM trên cùng feature và split để biết lợi ích và chi phí thực.

Nếu cần xác suất tin cậy, mô hình khác hoặc bước calibration phù hợp có thể thuận tiện hơn. Nếu dữ liệu quá nhiễu, cần kiểm tra xem các cập nhật có bị kéo qua lại bởi vài nhãn mâu thuẫn thay vì chỉ điều chỉnh tốc độ học.

## 20. Trường hợp không nên sử dụng

Không dùng phiên bản tuyến tính đơn giản để kỳ vọng giải mọi quan hệ kiểu XOR trong đúng biểu diễn gốc. Có thể cần feature phi tuyến hoặc mô hình khác; thêm nhiều epoch không đổi khả năng biểu diễn.

Không triển khai học dần bằng nhãn tự dự đoán như thể đó là nhãn chuẩn nếu chưa có thiết kế self-training riêng. Sai số có thể tự khuếch đại, đặc biệt khi ngoài miền hoặc mất cân bằng.

## 21. Các quyết định quan trọng khi thực hiện

Chọn learning rate, số epoch, shuffle, dừng, penalty và class weighting. Với demo cố định số vòng, `tol=None` có thể dùng để không dừng theo tolerance; nghiên cứu thật cần chọn quy tắc qua train/development.

Trong online, quyết định feature cố định, thứ tự lô, thời điểm cập nhật và xử lý drift. Nếu refit TF-IDF trên dữ liệu tương lai, ngay cả chưa dùng nhãn, đánh giá đã khác tình huống dự đoán chỉ bằng thông tin quá khứ.

## 22. Giải thích công thức và ký hiệu

Với $y_i\in\{-1,+1\}$, $f_i=w^Tx_i+b$ và $\eta>0$, luật cơ bản của bài là:

$$\text{nếu }y_if_i\le0:\quad w\leftarrow w+\eta y_ix_i,
\qquad b\leftarrow b+\eta y_i.$$

Nếu $y_if_i>0$, giữ nguyên. Điều kiện tại 0 và cách gán nhãn hòa phải được quy định; cập nhật tại ranh giới không nhất thiết trùng với mọi cách nói “chỉ cập nhật mẫu phân sai”.

Một loss liên quan là $\max(0,-yf(x))$, nhưng cách chọn cập nhật tại điểm không trơn và các tùy chọn triển khai cần được nêu rõ. Với dữ liệu tách tuyến tính, định lý lỗi cổ điển có dạng giới hạn theo $(R/\gamma)^2$ dưới các quy ước chuẩn về khởi tạo, chuẩn vector và margin; không dùng giới hạn đó như số vòng bảo đảm cho corpus không thỏa giả định. Nếu gộp bias thành feature, phải xét chuẩn/margin trong không gian đã mở rộng.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Khởi tạo $w=(0,0),b=0$, learning rate 1. Mẫu tích cực $(1,0)$ cho cập nhật $w=(1,0),b=1$. Mẫu âm $(0,1)$ lúc này có $f=1$, nên cập nhật $w=(1,-1),b=0$. Hai mẫu sau đó ở hai phía đúng trong ví dụ này.

Mã phần đầu kiểm tra đúng hai bước; phần sau dùng TF-IDF + Perceptron trên corpus tự soạn chung với hai bài trước. Cần Python và scikit-learn. Demo nhị phân không có lớp trung tính.

```python
import numpy as np
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import Perceptron
from sklearn.metrics import f1_score

w, b = np.zeros(2), 0.0
for x, y in [(np.array([1.0, 0.0]), 1), (np.array([0.0, 1.0]), -1)]:
    if y * (w @ x + b) <= 0:
        w = w + y * x
        b = b + y
assert np.allclose(w, [1.0, -1.0]) and b == 0.0

train = ["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
         "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
         "dịch vụ tệ bẩn", "khách sạn bẩn tệ"]
y_train = [1, 1, 1, 1, 0, 0, 0, 0]
test = ["phòng sạch zzholdout", "dịch vụ tốt", "phòng bẩn", "dịch vụ tệ"]
y_test = [1, 1, 0, 0]

model = Pipeline([
    ("tfidf", TfidfVectorizer(ngram_range=(1, 2), lowercase=True)),
    ("clf", Perceptron(penalty=None, eta0=1.0, max_iter=20,
                       tol=None, shuffle=True, random_state=42)),
])
model.fit(train, y_train)
pred = model.predict(test)
scores = model.decision_function(test)
assert "zzholdout" not in model.named_steps["tfidf"].vocabulary_
assert np.isfinite(scores).all() and scores.shape == (len(test),)
assert not hasattr(model.named_steps["clf"], "predict_proba")
print("Pred:", pred.tolist())
print("Macro-F1 minh họa:", f1_score(y_test, pred, average="macro"))
```

Hai phần dùng mã nhãn khác: tính tay ±1, API nhận 0/1. Không thay 0 vào trực tiếp $y_i$ trong luật tay vì sẽ làm cập nhật lớp đó bằng 0. Dữ liệu demo quá nhỏ để chứng minh ưu thế mô hình hoặc chọn số epoch thật.

## 24. Tình huống sai và phản ví dụ

Nếu cùng một vector có hai nhãn trái nhau, không tồn tại một quyết định tuyến tính cố định phân đúng cả hai. Thuật toán có thể cập nhật qua lại; vấn đề là nhãn/biểu diễn mâu thuẫn chứ không phải chưa chạy đủ lâu.

Quan hệ XOR trên hai feature cũng không tách được bằng một đường thẳng trong chính không gian đó. Có thể thêm feature tương tác hoặc đổi mô hình; cần đánh giá lại trên dữ liệu độc lập.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đề xuất macro-F1, từng lớp, confusion matrix, thời gian và số vòng; theo dõi metric development qua epoch khi thiết kế cho phép. Thử nhiều random seed hợp lý vì shuffle có thể đổi đường học, nhưng cũng kiểm tra biến thiên do split/corpus.

Với online, dùng giao thức dự đoán rồi đánh giá trước cập nhật theo thời gian. Báo theo các khoảng thời gian và lớp, không trộn dữ liệu tương lai vào feature learning. Metric train cao không đủ chứng minh hệ đã học ổn định.

## 26. Xử lý khi không đáp ứng yêu cầu

Dao động: xem nhãn mâu thuẫn, nhiễu và biểu diễn không tách được. Sai số train thấp nhưng test cao: xem overfit, shift và leakage. Nhạy thứ tự: thử averaging/regularization hoặc một đối chứng khác trên development, ghi đó là thay đổi phương pháp.

Nếu cần probability, chọn mô hình phù hợp hoặc thiết kế calibration riêng trên dữ liệu tách biệt; không áp sigmoid tùy ý rồi gọi là xác suất đã kiểm định. Nếu model không cải thiện, giữ nó làm baseline hợp lệ thay vì ép mọi mô hình phải đạt cùng chất lượng.

## 27. Phân biệt kết quả với kết luận

“Perceptron đã cập nhật n lần” là kết quả quá trình học, không phải n ví dụ đã được hiểu về mặt ngôn ngữ. “Không còn lỗi train” không chứng minh không có overfit hoặc dữ liệu rò rỉ.

“Trọng số từ tích cực tăng” chỉ là thay đổi của mô hình. Muốn kết luận cảm xúc khách đổi theo thời gian, cần cố định/kiểm tra phép đo và thành phần mẫu; model thay đổi cũng có thể là một nguyên nhân của thay đổi nhãn máy.

## 28. Cách viết vào bài nghiên cứu

Nêu estimator, version, biểu diễn, learning rate, epoch, shuffle, random seed, dừng và penalty. Với online, mô tả thứ tự, thời điểm nhãn, feature space và giao thức đánh giá.

Khung: “TF-IDF được fit trên [train]; Perceptron học với [cấu hình], chọn theo [development/CV]. Chất lượng cuối đo trên [test] bằng [metric].” Không dùng định lý hội tụ như bằng chứng hiệu năng khi chưa xác minh điều kiện trên dữ liệu.

## 29. Khả năng tái lập và tính minh bạch

Lưu pipeline, mapping nhãn, thứ tự/chia dữ liệu, seed, checkpoint khi cần và phiên bản. Với online, cần trạng thái mô hình trước mỗi lô nếu muốn kiểm tra lại các dự đoán đã công bố.

Kho GitHub nên chứa ví dụ tự soạn và cấu hình rõ, cùng hướng dẫn nguồn dữ liệu thật được phép dùng. Nếu dùng hashing để giữ không gian feature cố định, ghi kích thước và cấu hình; nó là lựa chọn khác TF-IDF vocabulary, không thay thế âm thầm.

## 30. Đạo đức nghiên cứu liên quan

Học dần có thể khuếch đại sai số nếu nhãn đến từ phản hồi thiên lệch hoặc những nhóm khách được xử lý nhiều hơn. Kiểm tra nguồn nhãn và lỗi theo nhóm/ngôn ngữ theo thời gian.

Không cập nhật tự động từ các nhãn chưa được xác thực rồi dùng mô hình để ra quyết định cá nhân thiếu giám sát phù hợp. Dữ liệu khách sạn cần ẩn thông tin nhận diện khi công bố ví dụ/checkpoint hoặc log.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **Perceptron có phải MLP không?** Không; bài này là mô hình tuyến tính không tầng ẩn.
2. **Dữ liệu bất kỳ có bảo đảm hết lỗi sau đủ epoch không?** Không.
3. **Bài tập tính:** từ $w=(1,-1),b=0$, mẫu $x=(1,0),y=+1$ không cần cập nhật theo điều kiện của bài vì signed margin dương.
4. **Bài tập phản ví dụ:** tạo hai mẫu cùng vector, trái nhãn; mô phỏng vài lượt và giải thích vì sao không có lời giải hết lỗi.
5. **Bài tập thực nghiệm:** chạy vài thứ tự train khác nhau, giữ test và feature cố định; phân biệt độ nhạy thứ tự với khả năng chuyển sang khách sạn mới.

## 32. Những điểm còn tranh luận

Các lựa chọn là khả năng cập nhật, tính ổn định dưới nhiễu, averaging và đánh giá online khi dữ liệu đổi. Hệ cập nhật nhanh không tự là hệ thích nghi tốt; cập nhật cũng có thể làm quên mẫu cũ hoặc học thiên lệch mới.

Trong nghiên cứu, cần tách rõ khả năng biểu diễn, thuật toán tối ưu và giao thức đánh giá. Nếu biểu diễn không chứa thông tin cần thiết, thay learning rate không giải quyết được; nếu đánh giá rò rỉ, điểm cao không chứng minh luật học có ưu thế.
