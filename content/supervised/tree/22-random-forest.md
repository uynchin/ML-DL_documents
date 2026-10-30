# Random Forest — Rừng cây ngẫu nhiên cho phân loại cảm xúc

> **Nhánh:** MACHINE LEARNING → Supervised: Tree. **Nguồn:** dòng 23 bảng Excel. **Đối chiếu:** 28/10/2026. Bài dùng RandomForestClassifier của scikit-learn 1.8.0, phân biệt triển khai này với mô tả majority vote thường gặp. Ví dụ, xác suất và phản hồi khách sạn tự soạn; không phải benchmark thực.

| Trường | Nội dung |
|---|---|
| Bản chất | Ensemble cây có ngẫu nhiên hóa dữ liệu/feature và tổng hợp dự đoán |
| Mốc tham khảo | Breiman, 2001 [S1]; không gán mọi ý tưởng tập hợp cây hoặc random subspace cho riêng mốc này |
| Đầu vào | Feature số, nhãn và cấu hình cây/rừng/lấy mẫu |
| Đầu ra | Nhãn, xác suất tổng hợp, dự đoán từng cây, OOB nếu có và feature importance theo phương pháp |
| Tiếng Việt | Cần biểu diễn phù hợp; dùng rừng cây không tự giải ngữ cảnh, phủ định và khía cạnh |
| Nhãn/tính toán | Cần nhãn; thường chạy CPU, có thể song song theo cây; chi phí/bộ nhớ tăng với số và kích thước cây |
| Triển khai | `sklearn.ensemble.RandomForestClassifier`; không đồng nhất Gradient Boosting hoặc Extra Trees |

## 1. Định nghĩa học thuật

Random Forest là họ mô hình kết hợp nhiều cây được phát triển với nguồn ngẫu nhiên, nhằm tạo các bộ dự đoán đủ tốt nhưng không hoàn toàn giống nhau. Công trình Breiman mô tả rừng các tree predictor phụ thuộc các vector ngẫu nhiên và cơ chế tổng hợp [S1].

Trong cấu hình phổ biến của bài, mỗi cây học từ bootstrap sample và xét tập con feature ở từng nút. RandomForestClassifier của scikit-learn tổng hợp xác suất lớp của cây [S2]; không nên mô tả mọi triển khai như cùng một quy tắc bỏ phiếu cứng.

## 2. Giải thích đơn giản

Thay vì hỏi một cây, bạn hỏi nhiều cây được học từ các phiên bản ngẫu nhiên của tập train và các lựa chọn feature khác nhau. Sau đó bạn kết hợp kết quả.

Nếu mọi cây mắc cùng lỗi, thêm cây khó giúp nhiều. Nếu cây khác nhau quá mức nhưng từng cây hầu như không có thông tin, rừng cũng không tự tốt. Cần cân bằng thông tin mỗi cây và sự khác biệt giữa chúng, rồi kiểm tra bằng dữ liệu ngoài mẫu.

## 3. Mục đích và vấn đề giải quyết

Đề xuất nghiên cứu khách sạn: thử liệu một ensemble giảm bất ổn của cây đơn và cải thiện phân loại trên cùng feature/split không. Có thể dùng feature văn bản, biến có cấu trúc hoặc kết hợp khi thông tin hợp lệ tại lúc dự đoán.

Nếu mục tiêu là phân tích yếu tố liên quan trải nghiệm, rừng có thể hỗ trợ dự đoán và kiểm tra feature. Nhưng importance không tự là hệ số tác động, và feature càng dễ dự đoán nhãn không có nghĩa càng nên can thiệp.

## 4. Điều kiện và giả định sử dụng

Cần nhãn/feature phù hợp và dữ liệu đủ để các cây nhận tín hiệu. Ngẫu nhiên hóa không sửa nhãn sai, trùng lặp hoặc leakage. Cấu hình học/test cần phản ánh tình huống áp dụng như khách sạn mới hoặc thời gian sau.

Với text sparse nhiều chiều, việc chỉ xét ít feature ở nút có thể bỏ qua tín hiệu hữu ích ở một số cây. Nên thử và so baseline tuyến tính, không mặc định rừng phi tuyến luôn thắng. Khả năng xử lý missing/categorical phụ thuộc phiên bản/triển khai [S2].

## 5. Cơ chế hoạt động bên trong

Với bootstrap bật: lấy mẫu có hoàn lại các hàng train → phát triển từng cây, chọn tập feature ứng viên tại mỗi nút → tìm split trong các lựa chọn → lặp đến điều kiện dừng → tổng hợp đầu ra.

Bootstrap tạo mẫu in-bag và OOB cho từng cây. Mỗi hàng có thể xuất hiện nhiều lần ở một cây hoặc không xuất hiện. Không phải mỗi cây được chia một phần train không chồng lấn, và tập feature được xét có thể đổi qua từng nút thay vì cố định một tập cho cả cây.

## 6. Quy trình áp dụng trong nghiên cứu

1. Xác định nhãn, biểu diễn và mục tiêu chuyển miền.
2. Loại trùng, chia train/development/test đúng nhóm/thời gian.
3. Fit vectorizer/biến đổi học được chỉ trên train của từng fold.
4. Xây đối chứng cây đơn, tuyến tính và Random Forest.
5. Chọn depth/leaf size, feature sampling, số cây và weighting bằng development/CV.
6. Dùng OOB như thông tin nội bộ khi giả định phù hợp, không tự thay test cuối.
7. Đánh giá phân lớp, calibration và chi phí; chỉ diễn giải importance sau kiểm tra chất lượng.

Nếu dùng importance để chọn feature hoặc sửa mô hình, dữ liệu dùng cho quyết định đó thuộc giai đoạn phát triển; cần giữ phần đánh giá cuối riêng.

## 7. Ví dụ minh họa xuyên suốt

Giả định ba cây cho xác suất tích cực lần lượt 0.51, 0.51 và 0.01. Bỏ phiếu cứng theo ngưỡng 0.5 sẽ có hai cây chọn tích cực. Nhưng trung bình xác suất là khoảng 0.3433, nên quy tắc trung bình xác suất chọn tiêu cực.

Ví dụ cho thấy mô tả tổng hợp ảnh hưởng kết quả. Các con số tự đặt không phải xác suất đã học từ corpus. Trong mã mục 23, ta kiểm tra tổng hợp của rừng thật bằng cách tính trung bình `predict_proba` của các cây.

## 8. Kết quả và cách diễn giải

Đầu ra xác suất của scikit-learn là trung bình xác suất lớp từ các cây, rồi nhãn theo lớp có xác suất cao nhất [S2]. Kiểm tra `classes_` để đọc đúng cột.

Xác suất 0.8 không nhất thiết có nghĩa đúng 80% hoặc đúng 80% cây bỏ phiếu cho lớp. Nếu lá có nhiều lớp, xác suất mỗi cây không phải chỉ 0/1. Cần đánh giá calibration; nhiều cây không tự bảo đảm xác suất là độ tin cậy thực được xác nhận.

## 9. Điểm mạnh, điểm yếu và giới hạn

Lợi thế là tổng hợp cây có thể giảm biến thiên, biểu diễn tương tác và hỗ trợ OOB/importance. Có thể huấn luyện nhiều cây độc lập tương đối về quy trình tính toán để dùng song song.

Hạn chế là khó đọc toàn bộ, tốn bộ nhớ, nhạy biểu diễn và vẫn có thể học shortcut. Tăng số cây không sửa được feature thiếu, dữ liệu lệch hoặc nhãn mâu thuẫn. Rừng có thể ổn định hơn cây đơn mà vẫn dự đoán sai có hệ thống ở nhóm khách chưa được đại diện.

## 10. So sánh với phương pháp gần nhau

| Phương pháp | Khác biệt |
|---|---|
| Decision Tree | Một cấu trúc, dễ đọc hơn nhưng thường bất ổn hơn |
| Bagging cây | Random Forest phổ biến thêm ngẫu nhiên hóa feature tại split; chi tiết cấu hình cần nêu |
| Extra Trees | Có cách ngẫu nhiên hóa ngưỡng/feature khác, không chỉ đổi tên rừng |
| Boosting | Cây được xây theo trình tự/định hướng objective khác, không phải cùng bootstrap ensemble |
| Linear text classifier | Có thể tận dụng feature sparse hiệu quả; khả năng phi tuyến khác |

Không kết luận rừng tốt hơn mọi cây đơn hay mọi mô hình tuyến tính chỉ từ lý thuyết ensemble; kết quả cần đo đúng bài toán.

## 11. Sai lầm thường gặp và cách khắc phục

- Mô tả probability là tỷ lệ phiếu cứng trong mọi triển khai: đọc API và quy tắc tổng hợp.
- Gọi OOB là test độc lập hoàn chỉnh của pipeline: kiểm tra cả bước học biểu diễn và cấu trúc nhóm.
- Xem MDI importance là phần trăm tác động nhân quả: xác định rõ đại lượng.
- Tăng `n_estimators` để chữa mọi overfit: kiểm tra kích thước cây, feature, nhãn và split.
- Bật dùng mọi CPU trong ví dụ lớn mà không đo: đặt tài nguyên phù hợp và báo chi phí.

## 12. Tài liệu tham khảo

- **[S1]** Breiman, L. (2001). *Random Forests*. Machine Learning, 45, 5–32. [Bản tác giả](https://www.stat.berkeley.edu/users/breiman/randomforest2001.pdf). [DOI](https://doi.org/10.1023/A:1010933404324).
- **[S2]** scikit-learn 1.8. [RandomForestClassifier API](https://scikit-learn.org/1.8/modules/generated/sklearn.ensemble.RandomForestClassifier.html).
- **[S3]** scikit-learn. [Permutation feature importance](https://scikit-learn.org/stable/modules/permutation_importance.html).
- **[S4]** scikit-learn. [Common pitfalls](https://scikit-learn.org/stable/common_pitfalls.html).
- **[S5]** scikit-learn. [Probability calibration](https://scikit-learn.org/stable/modules/calibration.html).

S1 là công trình nền tảng; S2 xác định chi tiết triển khai dùng trong mã. Các ví dụ khách sạn, xác suất tự đặt và kế hoạch kiểm định của bài không phải benchmark trong các nguồn này.

## 13. Vị trí trong toàn bộ nghiên cứu

Rừng là bộ dự đoán sau biểu diễn, có thể làm baseline hoặc mô hình chính tùy kết quả. Độ quan trọng feature là bước kiểm tra mô hình sau học, không tự là phương pháp kiểm định giả thuyết quản trị.

Nếu nhãn máy được đưa vào phân tích tiếp, cần xem sai số theo nhóm và dữ liệu mẫu. Một rừng ổn định nhiều lần chạy không chứng minh chỉ số sentiment đo đủ trải nghiệm khách hàng hoặc đại diện toàn thị trường.

## 14. Kiến thức nền cần biết trước

Cần hiểu cây, bootstrap, lấy mẫu có hoàn lại, feature subsampling, variance, tương quan và ensemble. **OOB** là mẫu không được lấy vào train của một cây cụ thể; không phải một tập cố định chung bị loại khỏi toàn bộ rừng.

Phân biệt **số cây** với **độ sâu cây**, và **random seed** với nguồn dữ liệu. Cố định seed làm một lần tái lập dễ hơn nhưng không đo được mọi bất định của lấy mẫu hoặc chuyển miền.

## 15. Nền tảng lý thuyết và logic hình thành

Tổng hợp các bộ dự đoán có thể làm giảm thành phần biến thiên nếu sai số không hoàn toàn đồng biến. Ngẫu nhiên hóa feature hướng tới khác biệt giữa cây, nhưng cũng có thể làm mỗi cây yếu đi.

Vì vậy, thêm ngẫu nhiên không phải mục tiêu tự thân. Cần đánh giá độ mạnh của hệ, tương quan/sự khác biệt hữu ích và giới hạn dữ liệu. Tính hội tụ của một phép tổng hợp khi số cây lớn không đồng nghĩa dữ liệu mới luôn được dự đoán đúng hoặc không có overfit theo mọi nghĩa.

## 16. Các thành phần cấu tạo

Hệ gồm biểu diễn, bootstrap/subsample, feature sampling, cấu hình cây, tổng hợp, OOB nếu dùng và bộ đánh giá. Các cây dùng chung tập feature theo tên/cột dù ở từng split chỉ xét một phần.

Nếu dùng class weight hoặc sample weight, phải giữ cách tạo và ảnh hưởng đến học/lá. Một bảng probability trong lá có thể dựa trên trọng số, không chỉ tỷ lệ số khách thật ở lá.

## 17. Các loại và biến thể

Có classifier/regressor, cấu hình bootstrap bật/tắt, max samples và các tiêu chuẩn cây. Random forest là họ rộng; bài mã hóa một cấu hình cụ thể của scikit-learn.

Nếu tắt bootstrap, OOB chuẩn theo cơ chế lấy mẫu đó không còn dùng như cấu hình đã mô tả. Nếu thay classifier bằng regressor, cách tổng hợp/metric đổi; không mượn quy tắc nhãn classifier cho mục tiêu số liên tục.

## 18. Đầu vào và đầu ra

Đầu vào là ma trận feature và nhãn, cùng cấu hình rừng. Đầu ra cần giữ nhãn/xác suất, thứ tự lớp, model version và split. Có thể lưu leaf ID từng cây hoặc dự đoán từng cây để kiểm tra.

OOB output cần ghi metric và số cây có đóng góp cho mẫu. Với quá ít cây, một số mẫu có thể chưa có dự đoán OOB đủ hợp lệ theo triển khai [S2]. Không thay các trường hợp thiếu bằng số 0 rồi xem như dự đoán âm chắc chắn.

## 19. Căn cứ lựa chọn

Nên thử khi cần tương tác phi tuyến hoặc tổng hợp cây, có nhãn và tài nguyên phù hợp. So chất lượng ngoài mẫu, calibration, thời gian/bộ nhớ và yêu cầu giải thích với cây đơn/LR/SVM.

Với feature rất sparse, kết quả có thể phụ thuộc `max_features` và hỗ trợ mẫu. Thử nghiệm cần đủ đại diện; metric trên corpus tự soạn nhỏ không có giá trị chọn rừng làm mô hình tốt nhất cho nghiên cứu thật.

## 20. Trường hợp không nên sử dụng

Không dùng OOB ngẫu nhiên thay đánh giá theo nhóm/thời gian khi mẫu trong cùng khách sạn phụ thuộc mạnh. OOB theo hàng có thể vẫn học những phản hồi gần giống từ cùng nhóm.

Không dùng rừng đồ sộ khi mục tiêu cần một quy tắc ngắn có thể kiểm tra trực tiếp mà không giải thích đánh đổi. Không dùng feature importance để đưa khuyến nghị can thiệp nhân quả khi nghiên cứu chỉ là dự đoán quan sát.

## 21. Các quyết định quan trọng khi thực hiện

Chọn số cây, depth, leaf size, `max_features`, bootstrap, `max_samples`, weighting và seed. Số feature được xét ở nút khác số feature cuối cùng được chọn để split tại nút đó.

Nếu vectorizer được fit một lần trên toàn train rồi lấy OOB ở classifier, OOB không đánh giá đúng một quy trình mà vectorizer cũng phải được học lại chỉ từ in-bag. Với TF-IDF/chọn feature dựa nhãn, khác biệt có thể quan trọng. Muốn đo toàn pipeline ngoài mẫu, dùng split/CV chứa cả các bước học biểu diễn và phù hợp nhóm/thời gian.

## 22. Giải thích công thức và ký hiệu

Trong triển khai trung bình probability với $B$ cây:

$$\hat p_k(x)=\frac1B\sum_{b=1}^{B}\hat p_{bk}(x),
\qquad\hat y=\arg\max_k\hat p_k(x).$$

$\hat p_{bk}$ là xác suất lớp tại lá của cây $b$. Với cấu hình không trọng số đó là tỷ lệ mẫu lớp trong lá; weighting làm thay đổi tỷ lệ tương ứng. Bỏ phiếu cứng $B^{-1}\sum_b\mathbf1[\hat y_b=k]$ là đại lượng khác, dù hai cách có thể trùng trong một số trường hợp.

Với bootstrap lấy đúng $n$ lượt có hoàn lại từ $n$ hàng, xác suất một hàng không được lấy là:

$$P(\text{OOB ở một cây})=(1-1/n)^n\approx e^{-1}.$$

Đây là xác suất/kỳ vọng theo cơ chế lấy mẫu, không phải mọi cây đều có chính xác 36.8% mẫu OOB. Nếu `max_samples` khác $n$, công thức số mũ cần đổi theo số lượt lấy.

Một trực giác về phương sai trung bình các đại lượng cùng phương sai $\sigma^2$ và cùng tương quan cặp $\rho$ là $\sigma^2[\rho+(1-\rho)/B]$. Đây là mô hình đơn giản để thấy tương quan giới hạn lợi ích tăng $B$, không phải công thức sai số phân loại đúng cho mọi forest.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Ba probability tích cực (0.51,0.51,0.01) có trung bình $1.03/3\approx0.3433$, khác đa số phiếu dương 2/3. Đây là phản ví dụ tự đặt để đọc API.

Mã dùng cùng corpus nhỏ với bài Decision Tree. Kiểm tra rừng bằng trung bình probability từng cây và xem OOB; không tối ưu bằng OOB/test trong ví dụ. Cần NumPy/scikit-learn.

```python
import numpy as np
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import f1_score

toy = np.array([0.51, 0.51, 0.01])
assert (toy > 0.5).sum() == 2 and toy.mean() < 0.5

train = ["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
         "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
         "dịch vụ tệ bẩn", "khách sạn bẩn tệ"]
y_train = [1, 1, 1, 1, 0, 0, 0, 0]
test = ["phòng sạch zzholdout", "dịch vụ tốt", "phòng bẩn", "dịch vụ tệ"]
y_test = [1, 1, 0, 0]
model = Pipeline([
    ("count", CountVectorizer(ngram_range=(1, 2))),
    ("forest", RandomForestClassifier(
        n_estimators=200, max_depth=3, min_samples_leaf=1,
        max_features="sqrt", bootstrap=True, oob_score=True,
        random_state=42, n_jobs=1)),
])
model.fit(train, y_train)
X_test = model.named_steps["count"].transform(test)
forest = model.named_steps["forest"]
prob = model.predict_proba(test)
per_tree = np.stack([tree.predict_proba(X_test) for tree in forest.estimators_])
assert np.allclose(prob, per_tree.mean(axis=0))
assert np.isfinite(prob).all() and np.allclose(prob.sum(axis=1), 1.0)
assert "zzholdout" not in model.named_steps["count"].vocabulary_
pred = model.predict(test)
assert np.array_equal(pred, forest.classes_[prob.argmax(axis=1)])
assert forest.oob_decision_function_.shape == (len(train), 2)
assert np.isfinite(forest.oob_decision_function_).all()
print("Pred:", pred.tolist())
print("Macro-F1 minh họa:", f1_score(y_test, pred, average="macro"))
print("OOB accuracy nội bộ, minh họa:", round(forest.oob_score_, 4))
```

Tất cả cây trong demo dùng cùng thứ tự cột xác suất nội bộ tương ứng rừng để phép kiểm tra có ý nghĩa; khi tự kết hợp mô hình độc lập phải căn chỉnh lớp. OOB score mặc định ở đây là accuracy, không phải macro-F1. Dữ liệu quá nhỏ để kết luận về calibration, ưu thế hoặc hiệu năng thật.

## 24. Tình huống sai và phản ví dụ

Một feature ID có nhiều giá trị có thể giúp chia train rất tốt và được importance cao, nhưng không dự đoán khách mới. Một feature tương quan với feature khác có thể có permutation importance thấp vì thông tin thay thế vẫn còn.

Nếu phản hồi gần trùng từ cùng khách sạn có mặt trong in-bag và OOB, OOB tốt chưa chứng minh chuyển sang khách sạn mới tốt. Lỗi nằm ở cách đánh giá mục tiêu, không chỉ ở số cây.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đề xuất macro-F1, từng lớp, confusion matrix và chi phí; với probability thêm log loss/Brier/calibration. So nhiều seed hợp lý để kiểm tra biến thiên ngẫu nhiên, nhưng cũng kiểm tra split nhóm/thời gian vì đó là nguồn bất định khác.

MDI là mức giảm impurity được tổng hợp, có thể ưu ái feature nhiều điểm chia [S3]. Permutation importance đo mức giảm score của mô hình đã fit khi xáo feature trên dữ liệu đánh giá, phụ thuộc metric/tập mẫu và feature tương quan. Nó cũng không phải causal effect. Nếu dùng để sửa/chọn feature, giữ test cuối ngoài lựa chọn.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu cây quá tương tự, thử feature sampling hoặc lấy mẫu phù hợp trên development. Nếu mỗi cây quá yếu, xem feature/leaf/depth; thêm nhiều cây yếu không tự phục hồi thông tin thiếu. Overfit hoặc lá quá nhỏ: điều chỉnh mức phức tạp.

Nếu OOB và test nhóm khác nhau mạnh, xem dependence/leakage/shift. Nếu importance không ổn định, kiểm tra tương quan, sample support và metric. Không chọn cấu hình theo test rồi nhận điểm test đó là xác nhận độc lập.

## 27. Phân biệt kết quả với kết luận

“Rừng cho xác suất tích cực 0.8” là đầu ra của ensemble. “80% cây chọn tích cực” là một đại lượng khác. “Khách hài lòng 80%” hoặc “dự đoán đúng thực 80%” còn cần định nghĩa/kiểm định.

“Feature vệ sinh quan trọng” chỉ có ý nghĩa dưới phương pháp importance cụ thể, mô hình và dữ liệu. Nó không tự chứng minh cải thiện vệ sinh sẽ gây mức tăng hài lòng tương ứng hay là ưu tiên can thiệp tốt nhất.

## 28. Cách viết vào bài nghiên cứu

Ghi estimator/version, feature, số cây, depth/leaf, `max_features`, bootstrap/max samples, weighting, seed và nguồn chọn tham số. Nêu quy tắc tổng hợp probability nếu đúng triển khai.

Báo OOB metric riêng với metric test và giải thích vai trò của nó. Nếu trình bày importance, nêu MDI/permutation, tập dữ liệu, metric và số lần lặp nếu có; không chỉ ghi “mức ảnh hưởng (%)” làm người đọc hiểu nhầm là tác động nhân quả.

## 29. Khả năng tái lập và tính minh bạch

Lưu pipeline, mapping feature/lớp, cấu hình sampling, cây/rừng, seed, split và version. Ghi tài nguyên/parallelism nếu cần so chi phí; khác số cây làm thời gian và kích thước model khác nhau.

Kho GitHub có thể chứa ví dụ nhỏ, hướng dẫn và cấu hình. Model lớn có thể cần phương án lưu phù hợp; dữ liệu khách thật chỉ chia sẻ theo quyền. Với OOB, lưu định nghĩa metric và cấu hình để người tái lập không nhầm với CV toàn pipeline.

## 30. Đạo đức nghiên cứu liên quan

Ensemble có thể che các shortcut vì khó đọc hàng trăm cây. Kiểm tra feature nhạy cảm/proxy và sai số theo nhóm trước khi dùng kết quả quản trị.

Không gọi một bảng importance là bằng chứng để quy trách nhiệm nhân viên hoặc xếp phẩm chất nhóm khách. Bảo vệ thông tin nhận diện trong dữ liệu/log/model, đặc biệt khi cây có lá rất ít mẫu hoặc feature ghi ID trực tiếp.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **Probability của forest có luôn là tỷ lệ phiếu cứng không?** Không; cần xem triển khai.
2. **OOB có thay test theo khách sạn mới không?** Không tự động.
3. **Bài tập tính:** với $n=10$, xác suất một hàng OOB trong bootstrap 10 lượt là $0.9^{10}\approx0.3487$.
4. **Bài tập thực nghiệm:** so vài `max_features` và leaf size trên development; ghi chất lượng, thời gian và kích thước, không chọn bằng test.
5. **Bài tập phản biện:** giải thích vì sao feature tương quan có importance thấp vẫn có thể mang thông tin và vì sao importance cao chưa đủ đưa khuyến nghị can thiệp.

## 32. Những điểm còn tranh luận

Các lựa chọn chính là mức đa dạng cây, biểu diễn text, calibration, OOB theo hàng so với mục tiêu nhóm/thời gian và cách diễn giải importance. Không có một cấu hình rừng bảo đảm tốt nhất mọi corpus.

Một rừng có thể dự đoán tốt mà khó giải thích về khoa học, hoặc có importance ổn định nhưng dựa vào dữ liệu lệch. Nghiên cứu cần đánh giá riêng dự đoán, độ ổn định, phép đo và phạm vi suy luận thay vì dùng tên ensemble như bảo đảm chất lượng.
