# Logistic Regression — Hồi quy logistic cho phân loại cảm xúc

> **Nhánh:** MACHINE LEARNING → Supervised: Linear. **Nguồn:** dòng 16 bảng Excel. **Đối chiếu nguồn:** 28/10/2026. Bài dùng logistic nhị phân làm nền rồi phân biệt đa lớp và mục đích suy luận thống kê. Ví dụ, hệ số và dữ liệu khách sạn tự soạn. Mã kiểm tra với scikit-learn 1.8.0; cần đối chiếu API khi dùng phiên bản khác.

| Trường | Nội dung |
|---|---|
| Bản chất | Mô hình có giám sát, mô hình hóa xác suất lớp qua hàm logistic; không phải hồi quy tuyến tính cho điểm liên tục |
| Mốc lịch sử | Có lịch sử phát triển dài trong thống kê; bài không gán một năm hoặc một tác giả duy nhất khi chưa khảo cứu lịch sử |
| Đầu vào | Đặc trưng số, nhãn, cấu hình mô hình và tối ưu |
| Đầu ra | Ước lượng xác suất, nhãn theo quy tắc, hệ số và intercept |
| Tiếng Việt | Dùng được với biểu diễn phù hợp; không tự biết tiếng Việt nếu chỉ nhận văn bản thô |
| Nhãn/tài nguyên | Cần nhãn huấn luyện; TF-IDF sparse và mô hình tuyến tính thường có thể dùng CPU |
| Triển khai | `sklearn.linear_model.LogisticRegression`; một lựa chọn khác cho mục đích suy luận là các công cụ GLM/logit có thống kê suy luận phù hợp |

## 1. Định nghĩa học thuật

Logistic Regression nhị phân mô hình hóa xác suất $P(y=1\mid x)$ bằng hàm sigmoid của một tổ hợp tuyến tính. Tương đương, mô hình giả định **log-odds** tuyến tính theo các đặc trưng đã chọn.

Mặc dù tên có “regression”, nó thường được dùng như bộ phân loại khi nhãn là rời rạc. Tài liệu scikit-learn phân biệt triển khai phân loại này với hồi quy cho mục tiêu liên tục [S1]. Trong nghiên cứu, phải nói rõ đang dùng để dự đoán hay suy luận mối liên hệ giữa biến.

## 2. Giải thích đơn giản

Mô hình cộng các tín hiệu có trọng số thành điểm $z$. Hàm sigmoid biến điểm đó thành số từ 0 đến 1 để diễn tả xác suất theo mô hình.

Nếu nhãn `1` là tích cực, mô hình có thể cho một đánh giá xác suất tích cực 0.8. Đó là ước lượng từ mô hình học trên dữ liệu, không phải bằng chứng chắc chắn người khách hài lòng 80% hoặc lời dự đoán được hiệu chuẩn hoàn hảo.

## 3. Mục đích và vấn đề giải quyết

Đề xuất nghiên cứu: xây TF-IDF + Logistic Regression làm baseline phân loại đánh giá khách sạn, so với SVM và Perceptron bằng cùng dữ liệu. Nếu cần xếp ưu tiên phản hồi, đầu ra xác suất có thể hữu ích sau khi kiểm định.

Một mục tiêu khác là nghiên cứu xác suất khách quay lại theo các biến khảo sát. Khi đó đơn vị, cấu trúc biến, giả định và cách suy luận khác bài toán phân loại văn bản; không áp nguyên cách đọc hệ số TF-IDF vào nghiên cứu hành vi.

## 4. Điều kiện và giả định sử dụng

Cần nhãn rõ, mẫu phù hợp và đặc trưng đủ thông tin. Logistic không yêu cầu các đặc trưng có phân phối chuẩn. Mô hình nhị phân chuẩn đặt tuyến tính ở **log-odds**, không ở trực tiếp xác suất.

Với suy luận thống kê, cần xem cấu trúc phụ thuộc, dạng hàm, đa cộng tuyến, separation và thiết kế mẫu. Với dự đoán, cần đánh giá khái quát hóa và calibration. Không dùng danh sách giả định suy luận một cách máy móc thay thế kiểm tra dự đoán hoặc ngược lại.

## 5. Cơ chế hoạt động bên trong

Pipeline minh họa: fit biểu diễn trên train → tính điểm tuyến tính → sigmoid/softmax → tính loss với nhãn → solver cập nhật hệ số để giảm mục tiêu có regularization → dùng hệ số để dự đoán mới.

Không phải mọi solver đều cập nhật từng mẫu bằng cùng một bước gradient. Ví dụ mã dùng `lbfgs`; công thức gradient đơn giản ở phần giải thích không có nghĩa đã mô tả toàn bộ thuật toán solver của thư viện.

## 6. Quy trình áp dụng trong nghiên cứu

1. Định nghĩa nhãn và quy định lớp tích cực, âm, trung tính nếu có.
2. Chia train/development/test, xử lý trùng và nhóm khách sạn/người viết.
3. Fit vectorizer trong pipeline chỉ bằng train.
4. Chọn biểu diễn, regularization, solver và class weight theo development/CV.
5. Kiểm tra hội tụ, nhãn `classes_` và ý nghĩa cột xác suất.
6. Chọn threshold theo mục tiêu nếu cần; tách dữ liệu dùng chọn threshold khỏi test.
7. Đánh giá phân loại và xác suất; đóng băng trước triển khai.

Nếu dùng nhãn sao làm nhãn sentiment, phải mô tả quy tắc chuyển và trường hợp đánh giá chữ mâu thuẫn với sao.

## 7. Ví dụ minh họa xuyên suốt

Giả định hai feature tự xây là mức tín hiệu “sạch” và “bẩn”. Đặt $w=(1,-1),b=0$, đánh giá có $x=(0.8,0.1)$ cho $z=0.7$ và xác suất lớp 1 xấp xỉ 0.6682.

Các số là để học công thức, không phải kết quả fit. Câu “phòng sạch nhưng nhân viên thiếu nhiệt tình” còn cần chuẩn nhãn hoặc phân tích khía cạnh. Một xác suất cao không sửa được nhiệm vụ gán nhãn chưa rõ.

## 8. Kết quả và cách diễn giải

`predict_proba` trả các cột theo thứ tự `classes_` [S2]. Không mặc định cột 1 luôn là lớp mà người nghiên cứu quan tâm nếu nhãn khác cấu hình demo.

Xác suất 0.8 là ước lượng có điều kiện theo mô hình/đặc trưng, không phải độ hài lòng 8/10. “Hiệu chuẩn tốt” nghĩa là các nhóm dự đoán khoảng 0.8 có tần suất nhãn đúng tương ứng trên dữ liệu phù hợp; cần đo điều này, không suy chỉ từ việc có sigmoid [S3].

## 9. Điểm mạnh, điểm yếu và giới hạn

Mô hình cho baseline tuyến tính gọn, có xác suất theo mô hình và hệ số có thể kiểm tra. Với TF-IDF, có thể xem các từ/cụm đẩy logit về từng phía.

Hạn chế là quan hệ phức tạp phụ thuộc feature, xác suất có thể lệch dưới shift và hệ số dễ bị hiểu quá mức. Regularization, feature tương quan và cách scale ảnh hưởng hệ số. Dự đoán tốt không tự chứng minh hệ số có ý nghĩa nhân quả hoặc có p-value phù hợp.

## 10. So sánh với phương pháp gần nhau

| Phương pháp | Khác biệt |
|---|---|
| Linear Regression | Mục tiêu và hàm liên kết khác; dự đoán có thể ngoài [0,1] nếu dùng cho nhãn nhị phân |
| Linear SVM | Margin-based loss, decision score không tự là xác suất |
| Perceptron | Luật cập nhật khác, không có likelihood logistic chuẩn |
| Multinomial Logistic | Nhiều lớp loại trừ nhau, có thể dùng softmax |
| Ordinal Logistic | Dành cho nhãn có thứ tự theo mô hình/giả định riêng, không đồng nhất multinomial |

Với nhiều cảm xúc cùng hiện diện, cần thiết kế multi-label thay vì ép softmax để các nhãn loại trừ nhau.

## 11. Sai lầm thường gặp và cách khắc phục

- Gọi tên “regression” rồi dùng cho điểm 1–5 mà không xác định nhiệm vụ: phân biệt nhãn phân loại, thứ bậc và mục tiêu liên tục.
- Đọc hệ số như phần trăm đổi xác suất: logistic tuyến tính ở log-odds; hiệu ứng xác suất phụ thuộc điểm xuất phát.
- Tin xác suất mặc nhiên đúng: đánh giá calibration và chuyển miền.
- Chọn threshold trên test: dùng development hoặc thiết kế CV phù hợp.
- Báo p-value từ bộ phân loại chỉ xuất hệ số: dùng phương pháp suy luận và giả định tương ứng nếu đó là mục tiêu.

## 12. Tài liệu tham khảo

- **[S1]** scikit-learn. [Linear Models: Logistic regression](https://scikit-learn.org/stable/modules/linear_model.html#logistic-regression).
- **[S2]** scikit-learn. [LogisticRegression API](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html).
- **[S3]** scikit-learn. [Probability calibration](https://scikit-learn.org/stable/modules/calibration.html).
- **[S4]** scikit-learn. [Common pitfalls and data leakage](https://scikit-learn.org/stable/common_pitfalls.html).

Đây là tài liệu triển khai/hướng dẫn gốc của thư viện. Bài không gán một công trình khai sinh duy nhất cho toàn bộ lịch sử logistic. Các kế hoạch nghiên cứu khách sạn, số liệu và chương trình minh họa do người biên soạn xây dựng, không phải benchmark được công bố trong các nguồn trên.

## 13. Vị trí trong toàn bộ nghiên cứu

Trong phân loại văn bản, LR nằm sau biểu diễn và trước đánh giá/sử dụng nhãn. Trong nghiên cứu cấu trúc hành vi, nó có thể là mô hình kết quả nhị phân sau bước xác định giả thuyết và thiết kế biến.

Hai vai trò cần được trình bày riêng. Việc dùng nhãn máy làm biến tiếp theo cần tính đến lỗi đo; không coi một mô hình dự đoán đầu vào chính xác vừa đủ là thay thế hoàn toàn cho chuẩn đo lường nghiên cứu.

## 14. Kiến thức nền cần biết trước

Cần hiểu xác suất, odds $p/(1-p)$, log-odds, sigmoid, log loss và regularization. Odds không phải xác suất: $p=0.8$ cho odds bằng 4, không phải 0.8.

Cần phân biệt **classification threshold** với tham số huấn luyện. Đổi threshold làm nhãn thay đổi nhưng giữ cùng xác suất từ mô hình đã fit; đổi `C` hoặc đặc trưng yêu cầu fit lại và có thể thay cả xác suất.

## 15. Nền tảng lý thuyết và logic hình thành

Logistic đặt một mô hình xác suất cho nhãn nhị phân, rồi học các tham số làm dữ liệu quan sát phù hợp mô hình theo mục tiêu tối ưu. Regularization thêm ràng buộc ưu tiên để giảm hệ số quá lớn hoặc kiểm soát độ phức tạp.

Trong văn bản, điều này vận hành trên feature, không trên ý nghĩa xã hội đầy đủ. Nếu gán nhãn không nhất quán hoặc thiếu biểu diễn phủ định, hàm xác suất vẫn có thể tự tin theo tín hiệu học được mà hiểu sai câu.

## 16. Các thành phần cấu tạo

Hệ gồm feature extractor, trọng số/intercept, hàm liên kết, loss, regularization, solver và chính sách nhãn. Đánh giá xác suất có thể thêm bước hiệu chuẩn nếu cần.

Nên tách tham số `C`, cấu hình solver và threshold trong nhật ký. Nếu chỉ nói “dùng logistic”, người đọc chưa biết regularization, đa lớp hay cách biến xác suất thành quyết định; những lựa chọn đó ảnh hưởng kết quả và khả năng tái lập.

## 17. Các loại và biến thể

Có binary logistic, multinomial softmax, one-vs-rest và các mô hình ordinal theo giả định khác. Về regularization có L1, L2 hoặc Elastic Net tùy solver/phiên bản [S1–S2].

Logistic dùng TF-IDF và logistic dùng sentence embedding có cùng loại classifier nhưng khác biểu diễn. Logistic trong mô hình đa mức hoặc thiết kế survey cũng không hoàn toàn giống cấu hình classifier mặc định; cần chọn công cụ theo câu hỏi thay vì tên chung.

## 18. Đầu vào và đầu ra

Đầu vào $X$ chứa feature và $y$ chứa nhãn. Cần giữ mapping nhãn và vocabulary/model embedding. Đầu ra nên lưu `review_id`, nhãn thật, nhãn dự đoán, từng xác suất, logit, split và version.

Văn bản OOV hoàn toàn có thể thành vector 0; xác suất lúc đó chủ yếu do intercept. Nó vẫn là một số hợp lệ của mô hình nhưng không đồng nghĩa đã thu thập bằng chứng nội dung. Nên phát hiện để xem lại hoặc giới hạn diễn giải.

## 19. Căn cứ lựa chọn

LR đáng thử khi cần baseline tuyến tính và có nhu cầu xác suất/threshold theo chi phí. So với SVM trên cùng feature và split giúp xác định lợi ích thực tế.

Nếu mục tiêu suy luận odds ratio của biến quản trị, cần xem độ giá trị biến, mẫu và phương pháp ước lượng phù hợp. Không chọn chỉ vì mô hình có hệ số dễ xuất ra; hệ số dễ nhìn không đảm bảo suy luận hợp lệ.

## 20. Trường hợp không nên sử dụng

Không áp logistic nhị phân nguyên bản cho nhiều nhãn cùng lúc hoặc điểm liên tục mà không thiết kế lại. Không dùng xác suất chưa kiểm định làm quyết định có chi phí cao theo kiểu tự động.

Với dữ liệu quá ít, separation hoặc class rất hiếm, cần xem regularization, cách lấy mẫu và độ bất định. Thêm số vòng tối ưu không tự giải quyết thiếu thông tin hay sai mô hình.

## 21. Các quyết định quan trọng khi thực hiện

Cần chọn biểu diễn, regularization, `C`, solver, class weight và threshold. Trong tham số phổ biến của scikit-learn, `C` nhỏ thường tương ứng regularization mạnh hơn [S2]. Các lựa chọn penalty/solver phải tương thích phiên bản.

Class weighting hoặc sampling đổi đóng góp của các lớp vào việc học; xác suất sau đó cần được đánh giá trên phân bố mục tiêu. Không lấy tỷ lệ huấn luyện được cân bằng nhân tạo làm tỷ lệ tự nhiên của phản hồi ngoài thực tế.

## 22. Giải thích công thức và ký hiệu

Với $z=w^Tx+b$:

$$p=P(y=1\mid x)=\sigma(z)=\frac{1}{1+e^{-z}},
\qquad\log\frac{p}{1-p}=w^Tx+b.$$

Log loss nhị phân có dạng:

$$L=-\sum_i\left[y_i\log p_i+(1-y_i)\log(1-p_i)\right].$$

Có thể thêm regularization, ví dụ $\lambda\|w\|_2^2/2$. Hệ số/thang chuẩn hóa của objective phụ thuộc triển khai; không suy `C` luôn bằng đúng $1/\lambda$ dưới mọi quy ước tổng/trung bình.

Với feature số $x_j$, $e^{w_j}$ là tỷ số odds khi tăng $x_j$ một đơn vị và giữ các biến khác theo mô hình. Nó không phải tỷ số xác suất, không tự là tác động nhân quả. Với TF-IDF được chuẩn hóa, tăng một feature mà giữ mọi feature khác có thể không tương ứng một thao tác văn bản tự nhiên; cần thận trọng diễn giải.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Với $z=0.7$, $p\approx0.6682$, odds $\approx2.0138$. Nếu ngưỡng là 0.5 thì chọn lớp 1; nếu ngưỡng 0.8 thì không chọn lớp 1. Việc ngưỡng nào phù hợp cần mục tiêu và dữ liệu phát triển.

Mã sau dùng cùng dữ liệu tự soạn như bài SVM để minh họa cách fit, đọc xác suất và kiểm tra vocabulary. Không có development vì demo không tối ưu tham số; nghiên cứu thật phải thêm bước đó. Cần cài scikit-learn.

```python
import numpy as np
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import f1_score, log_loss

train = ["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
         "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
         "dịch vụ tệ bẩn", "khách sạn bẩn tệ"]
y_train = [1, 1, 1, 1, 0, 0, 0, 0]
test = ["phòng sạch zzholdout", "dịch vụ tốt", "phòng bẩn", "dịch vụ tệ"]
y_test = [1, 1, 0, 0]

model = Pipeline([
    ("tfidf", TfidfVectorizer(ngram_range=(1, 2), lowercase=True)),
    ("clf", LogisticRegression(C=1.0, solver="lbfgs", max_iter=2000)),
])
model.fit(train, y_train)
classes = model.named_steps["clf"].classes_
prob = model.predict_proba(test)
positive_column = np.flatnonzero(classes == 1).item()
pred = classes[prob.argmax(axis=1)]
assert "zzholdout" not in model.named_steps["tfidf"].vocabulary_
assert prob.shape == (len(test), 2)
assert np.isfinite(prob).all() and np.allclose(prob.sum(axis=1), 1.0)
assert np.allclose(prob[:, positive_column], 1 / (1 + np.exp(-model.decision_function(test))))
print("Pred:", pred.tolist())
print("Macro-F1 minh họa:", f1_score(y_test, pred, average="macro"))
print("Log loss minh họa:", round(log_loss(y_test, prob, labels=classes), 4))
```

Tổng xác suất bằng 1 và khớp sigmoid là kiểm tra tính toán, không phải kiểm định calibration. Dữ liệu 12 câu tự soạn không đủ đo khả năng áp dụng thật hoặc chọn mô hình thắng cuộc.

## 24. Tình huống sai và phản ví dụ

Nếu feature “phòng sạch” tương quan với tên khách sạn, hệ số tên khách sạn có thể lớn dù không có ý nghĩa cảm xúc. Dự đoán ngoài miền có thể sai dù xác suất cao.

Một lỗi diễn giải khác: hệ số $w=0.7$ không nghĩa xác suất tăng 70 điểm phần trăm. Odds nhân khoảng 2.01 theo mô hình; mức đổi xác suất phụ thuộc xác suất xuất phát và cấu hình các biến khác.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đề xuất macro-F1, precision/recall từng lớp và confusion matrix cho nhãn; log loss, Brier score và calibration curve cho xác suất. AUC, nếu dùng, đánh giá thứ tự/phân biệt, không tự chứng minh hiệu chuẩn.

Kiểm tra theo nhóm khách sạn, thời gian, độ dài và câu phủ định. Chọn tham số/threshold trong train-development, không lặp chọn bằng test. Nếu mục tiêu suy luận thống kê, cần thêm kiểm tra đặc thù mô hình, sai số chuẩn và thiết kế mẫu, không chỉ báo F1.

## 26. Xử lý khi không đáp ứng yêu cầu

Chưa hội tụ: kiểm tra thang, solver, regularization và số vòng; không chỉ tăng vô hạn. Xác suất quá tự tin: xem overfit, lệch phân bố và thử calibration phù hợp trên dữ liệu không dùng fit classifier.

Sai nhiều phủ định/hỗn hợp: xem nhãn và biểu diễn. Hệ số suy luận bất ổn: xem separation, biến tương quan, cỡ mẫu và phương pháp ước lượng. Nếu sửa theo test, cần một test mới cho kết luận độc lập.

## 27. Phân biệt kết quả với kết luận

“Mô hình ước lượng xác suất tích cực 0.8” không khẳng định cá nhân hài lòng ở mức 80%. “Feature có hệ số dương” mô tả quan hệ có điều kiện trong mô hình, không chứng minh nguyên nhân.

Muốn suy rộng tỷ lệ tích cực của khách hàng phải xem sai số phân loại và cách chọn mẫu. Người để lại phản hồi không nhất thiết đại diện toàn bộ khách, dù mô hình phân loại chính xác.

## 28. Cách viết vào bài nghiên cứu

Nêu vai trò dự đoán/suy luận, loại nhãn, feature, solver, regularization, `C`, ngưỡng và cách chọn. Mẫu: “Mô hình TF-IDF + LR được chọn theo [metric] qua [CV]; xác suất được kiểm tra bằng [metric/calibration], nhãn dựa trên [threshold].”

Nếu báo odds ratio, ghi đơn vị tăng, biến giữ cố định, regularization và giới hạn diễn giải. Không thêm p-value hoặc khoảng tin cậy từ một công thức không phù hợp estimator/thiết kế đã dùng.

## 29. Khả năng tái lập và tính minh bạch

Lưu toàn pipeline, vocabulary, thứ tự lớp, threshold, bộ hiệu chuẩn nếu có, version, split và cấu hình. Các lần fit mô hình khác phiên bản có thể khác nhỏ; cần ghi môi trường đã kiểm tra.

GitHub nên có dữ liệu giả định và hướng dẫn chạy, còn dữ liệu thật chỉ chia sẻ theo quyền. Với nghiên cứu xác suất, giữ đầu ra từng mẫu để tính lại metric nhưng phải ẩn thông tin nhận diện phù hợp.

## 30. Đạo đức nghiên cứu liên quan

Không gọi xác suất là mức chắc chắn tuyệt đối về con người. Kiểm tra calibration và lỗi theo nhóm để tránh nhóm ít đại diện bị quyết định thiếu công bằng.

Trong ưu tiên phản hồi, lỗi bỏ sót lời phàn nàn có thể khác chi phí với báo nhầm. Cần xác định mục đích và kiểm tra con người phù hợp, thay vì áp ngưỡng 0.5 như một quyết định quản trị phổ quát.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **Logistic tuyến tính ở xác suất hay log-odds?** Log-odds theo feature đã chọn.
2. **Có sigmoid là đủ calibration tốt không?** Không.
3. **Bài tập tính:** $z=0$ cho $p=0.5$ và odds=1. Đổi $z$ thành $\log 3$ cho $p=0.75$ và odds=3.
4. **Bài tập diễn giải:** với $w_j=\log 2$, giải thích odds nhân đôi khác xác suất nhân đôi thế nào.
5. **Bài tập thiết kế:** so hai threshold trên development và nêu chi phí sai trước khi chốt; giữ test cho đánh giá cuối.

## 32. Những điểm còn tranh luận

Những lựa chọn cần cân nhắc là mục tiêu dự đoán hay suy luận, cách chọn đặc trưng, regularization, calibration và chuyển miền. Một mô hình có AUC cao có thể có xác suất chưa tốt, còn hệ số dễ đọc có thể thiếu cơ sở để suy luận nguyên nhân.

Nghiên cứu thuyết phục cần công bố rõ đại lượng đang tối ưu và giới hạn phép đo. Không dùng cùng một từ “tốt” để gộp hiệu năng phân loại, chất lượng xác suất và giá trị suy luận thống kê.
