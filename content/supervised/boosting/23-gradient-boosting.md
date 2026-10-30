# Gradient Boosting — Học tăng cường theo gradient cho phân loại cảm xúc

> **Nhánh:** MACHINE LEARNING → Supervised: Boosting. **Nguồn:** dòng 24 bảng Excel. **Đối chiếu:** 28/10/2026. Tập trung Gradient Boosted Decision Trees (GBDT), dùng GradientBoostingClassifier để minh họa; đây không phải reinforcement learning. Mã Python được chạy với scikit-learn 1.8.0. Dữ liệu và tình huống khách sạn trong ví dụ đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bản chất | Mô hình cộng dồn các cây, học tuần tự để giảm hàm mất mát |
| Mốc tham khảo | Friedman, 2001 [S1] |
| Đầu vào | Đặc trưng số hoặc ma trận văn bản đã vector hóa, nhãn |
| Đầu ra | Điểm số, xác suất và nhãn dự đoán |
| Tiếng Việt | Cần biểu diễn văn bản; giữ dấu và xử lý phủ định có chủ đích |
| Phần mềm minh họa | sklearn.ensemble.GradientBoostingClassifier |

## 1. Định nghĩa học thuật

Gradient Boosting là khuôn khổ xây mô hình cộng tính bằng cách thêm từng hàm học cơ sở theo hướng làm giảm loss. Trong GBDT, hàm cơ sở thường là cây hồi quy, kể cả khi bài toán cuối cùng là phân loại. Mỗi cây mới xấp xỉ thông tin từ gradient âm tại trạng thái mô hình hiện tại [S1].

“Boosting” ở đây không có tác nhân, hành động và phần thưởng như reinforcement learning. Nó cũng không có nghĩa chỉ lấy nhiều cây rồi bỏ phiếu ngang nhau.

## 2. Giải thích đơn giản

Hãy tưởng tượng nhóm đọc đánh giá khách sạn. Người đầu đưa một dự đoán sơ bộ; người sau xem mô hình còn cần điều chỉnh ở đâu rồi thêm một phần sửa nhỏ. Kết quả cuối là tổng các phần sửa, với mức đóng góp được kiểm soát.

Phép ví von “sửa lỗi” có giới hạn: với phân loại, mô hình không chỉ đánh dấu đúng/sai mà sử dụng mức sai theo loss và xác suất hiện tại. Một câu đã dự đoán đúng nhưng còn thiếu tự tin vẫn có thể tạo gradient.

## 3. Mục đích và vấn đề giải quyết

Có thể dùng GBDT để phân loại cảm xúc từ chỉ báo văn bản, đặc trưng ngôn ngữ và biến có cấu trúc có sẵn lúc dự đoán. Nó phù hợp để khảo sát giá trị dự đoán của tương tác phi tuyến, chẳng hạn kết hợp số cụm chê với dấu hiệu phủ định.

Trong nghiên cứu khách sạn, câu hỏi nên là “mô hình có cải thiện dự đoán trên khách sạn mới không?” thay vì mặc định mô hình nhiều cây luôn vượt mô hình tuyến tính. Cần đối chứng và cùng thiết kế đánh giá.

## 4. Điều kiện và giả định sử dụng

Cần tập nhãn đủ tin cậy, định nghĩa tích cực/tiêu cực/trung tính hoặc hỗn hợp rõ ràng, cùng đặc trưng đo được ở thời điểm dự đoán. Train phải đại diện đủ cho bối cảnh ứng dụng; không có giả định rằng mọi đặc trưng độc lập.

Ví dụ dùng GradientBoostingClassifier với dữ liệu số hữu hạn. Không suy rộng khả năng xử lý missing/categorical của HistGradientBoostingClassifier sang estimator này. Chuỗi văn bản thô cần vector hóa; dữ liệu thiếu cần xử lý trong từng fold.

## 5. Cơ chế hoạt động bên trong

Bắt đầu bằng dự đoán hằng phù hợp với loss. Tại vòng m, tính gradient âm theo điểm số hiện tại cho các mẫu train, fit cây hồi quy vào tín hiệu này, rồi xác định giá trị cập nhật tại lá theo loss. Nhân cập nhật với learning rate và cộng vào mô hình.

Với log loss nhị phân, nếu dùng logit F thì gradient âm là y−p. Cây không trực tiếp cộng vào xác suất p. Sau khi cộng vào F, dùng sigmoid để thu xác suất mới. Mỗi vòng phụ thuộc mô hình từ các vòng trước.

## 6. Quy trình áp dụng trong nghiên cứu

1. Chốt đơn vị đánh giá, quy tắc gán nhãn và tiêu chí loại trùng.
2. Chia train/development/test theo khách sạn hoặc thời gian nếu đó là đích suy rộng.
3. Fit vocabulary, TF-IDF, imputer và các phép chọn đặc trưng chỉ trong train của từng fold.
4. Chạy baseline đa số, mô hình tuyến tính và GBDT trên cùng split.
5. Chọn learning rate, số cây, độ sâu và kích thước lá bằng development/CV.
6. Khóa cấu hình, đánh giá test và phân tích lỗi theo nhóm.

Nếu dùng early stopping, tập theo dõi là development. Chia validation tự động ngẫu nhiên bên trong estimator có thể không tôn trọng nhóm khách sạn/thời gian; hãy thiết kế vòng huấn luyện phù hợp thay vì coi đó là biện pháp chống rò rỉ tự động.

## 7. Ví dụ minh họa xuyên suốt

Một nghiên cứu giả định muốn nhận diện đánh giá tích cực và tiêu cực. “Phòng sạch đẹp” thuộc lớp 1, “phòng bẩn tệ” thuộc lớp 0. Các câu mang cả khen lẫn chê cần quy tắc gán nhãn riêng; ví dụ nhỏ không bao gồm lớp trung tính.

Ở bước đầu có p=0.5, một mẫu tích cực tạo tín hiệu y−p=0.5; mẫu tiêu cực tạo −0.5. Cây mới học cách tách các vùng có tín hiệu điều chỉnh khác nhau. Trong dữ liệu thật, vùng có “không sạch” cần được biểu diễn đủ để tránh hiểu từ “sạch” riêng lẻ.

## 8. Kết quả và cách diễn giải

Đầu ra có thể là logit, xác suất hoặc nhãn. Logit là điểm trước sigmoid; xác suất dự đoán không tự động được hiệu chỉnh tốt. Ngưỡng mặc định chỉ là một quy tắc quyết định, không phản ánh mọi chi phí bỏ sót khiếu nại.

Báo macro-F1 cùng precision/recall theo lớp, confusion matrix và phân bố mẫu. Nếu quan tâm xác suất, kiểm tra calibration và proper scoring rule phù hợp. Không diễn giải số cây hay feature importance như mức tác động nhân quả.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là học tương tác, dùng được nhiều loss và điều chỉnh mức phức tạp bằng cây nhỏ, shrinkage và lấy mẫu. Có thể kết hợp đặc trưng văn bản với số liệu cấu trúc trong một thiết kế dự đoán.

Nhược điểm là quá trình học tuần tự, nhạy cấu hình, dễ học noise khi số vòng lớn và không tự giải quyết ngữ nghĩa. Văn bản có hàng trăm nghìn chiều sparse có thể khiến baseline tuyến tính hấp dẫn hơn về chi phí; cần đo trên dữ liệu nghiên cứu.

## 10. So sánh với phương pháp gần nhau

| Phương pháp | Quan hệ với GBDT |
|---|---|
| Decision Tree | Một cây đơn; GBDT cộng nhiều phần cập nhật |
| Random Forest | Chủ yếu xây các cây được ngẫu nhiên hóa rồi tổng hợp, không sửa gradient tuần tự như GBDT |
| AdaBoost | Điều chỉnh trọng số mẫu theo sai phân loại trong dạng kinh điển |
| XGBoost/LightGBM/CatBoost | Các hệ thống boosting có kỹ thuật và lựa chọn triển khai riêng |
| Logistic Regression | Tuyến tính theo biểu diễn đầu vào; thường là đối chứng cần có cho văn bản |

Tên thư viện và khuôn khổ thuật toán không nằm cùng một cấp khái niệm. Không cộng chúng như các họ hoàn toàn tách biệt trong phân loại lý thuyết.

## 11. Sai lầm thường gặp và cách khắc phục

Sai lầm phổ biến là nói cây sau luôn học residual y−ŷ. Điều này đúng trực tiếp cho squared loss ở một quy ước phù hợp; với loss khác phải tính gradient tương ứng. Với log loss, phải xác định ŷ đang là xác suất hay logit.

Một lỗi khác là đổi số cây nhiều lần sau khi xem test. Hãy chọn bằng development, lưu lại toàn bộ lượt thử và chỉ dùng test cho đánh giá đã khóa. Không fit vectorizer trước khi chia dữ liệu.

## 12. Tài liệu tham khảo

- [S1] Friedman, J. H. (2001). *Greedy Function Approximation: A Gradient Boosting Machine*. The Annals of Statistics, 29(5), 1189–1232. [DOI](https://doi.org/10.1214/aos/1013203451).
- [S2] scikit-learn. [GradientBoostingClassifier, phiên bản 1.8](https://scikit-learn.org/1.8/modules/generated/sklearn.ensemble.GradientBoostingClassifier.html).
- [S3] scikit-learn. [Hướng dẫn các mô hình ensemble](https://scikit-learn.org/1.8/modules/ensemble.html).

[S1] cho nền tảng thuật toán; [S2–S3] cho hành vi phần mềm. Công thức minh họa chọn tham số hóa logit; dữ liệu và mã ví dụ do người soạn tạo.

## 13. Vị trí trong toàn bộ nghiên cứu

GBDT nằm ở giai đoạn học dự đoán sau thu thập, làm sạch, gán nhãn và xây biểu diễn. Nó không thay thế thiết kế mẫu hay kiểm tra độ nhất quán giữa người gán nhãn.

Nếu đầu ra dùng để tính tỷ lệ phản hồi tiêu cực, cần đánh giá sai số phân loại và độ lệch theo nhóm trước khi tổng hợp. Mô hình phân loại tốt trên một split chưa đủ xác nhận chỉ số cảm xúc là phép đo hợp lệ cho mọi khách sạn.

## 14. Kiến thức nền cần biết trước

Nên hiểu train/validation/test, cây hồi quy, loss, đạo hàm và sigmoid. Với văn bản, cần biết vocabulary, count/TF-IDF, n-gram và lý do giữ tiền xử lý bên trong cross-validation.

Có thể học bằng ví dụ bốn mẫu trước: tính p từ F, tính y−p, gom mẫu vào lá rồi cộng cập nhật vào F. Khi đọc tài liệu, kiểm tra quy ước điểm số vì một số cách trình bày dùng một nửa log-odds.

## 15. Nền tảng lý thuyết và logic hình thành

Tối ưu trong không gian hàm xem mô hình hiện tại như một điểm cần cải thiện. Gradient âm chỉ hướng giảm loss ở các giá trị dự đoán trên train; cây cung cấp một hàm xấp xỉ hướng đó bằng cấu trúc đơn giản.

Thêm từng cây là tìm kiếm tiến theo từng bước, không tối ưu đồng thời mọi cấu trúc cây có thể có. Shrinkage giảm độ lớn bước đi; nó thường được chọn cùng số vòng và độ phức tạp cây, không có một learning rate tốt nhất cho mọi dữ liệu.

## 16. Các thành phần cấu tạo

Các thành phần gồm loss; mô hình khởi tạo F₀; cơ chế tính pseudo-residual; bộ học cây; cách ước lượng giá trị lá; learning rate; số vòng và các giới hạn cây. Pipeline văn bản còn có vectorizer và bước kiểm tra đầu vào.

Phân biệt tham số học như ngưỡng chia, giá trị lá với siêu tham số như max_depth, min_samples_leaf, subsample. Các siêu tham số được quyết định trong quy trình lựa chọn mô hình, không phải ước lượng từ test.

## 17. Các loại và biến thể

Có gradient boosting cho hồi quy, nhị phân, đa lớp và các loss phù hợp khác. Stochastic gradient boosting lấy mẫu một phần dữ liệu ở các vòng; histogram boosting hạn chế ứng viên split theo bins.

Không phải mọi triển khai có cùng hỗ trợ sparse, missing, categorical, GPU hoặc early stopping. Bài dùng GradientBoostingClassifier để tránh lẫn API với HistGradientBoostingClassifier hay ba thư viện chuyên biệt ở các bài tiếp theo.

## 18. Đầu vào và đầu ra

Đầu vào huấn luyện là X kích thước n×d, nhãn y và có thể có sample weight. Ở dự đoán, X mới phải có cùng định nghĩa, thứ tự cột và phép biến đổi đã học. Từ chưa thấy có thể không đóng góp đặc trưng count trong vectorizer đã khóa.

Đầu ra nhị phân là một điểm số mỗi mẫu, hai xác suất theo classes_ và nhãn theo quy tắc. Đa lớp có cấu trúc cập nhật khác; không lấy nguyên công thức sigmoid một logit cho mọi trường hợp.

## 19. Căn cứ lựa chọn

Chọn thử GBDT khi có lý do tin tương tác giữa các chỉ báo mang giá trị dự đoán, có đủ nhãn và chấp nhận chi phí huấn luyện. Quyết định cuối dựa trên hiệu quả ngoài mẫu, calibration nếu cần và tài nguyên.

Nên giữ một baseline đơn giản cùng biểu diễn để phân biệt ích lợi của thuật toán với ích lợi của đặc trưng. Khi so với mô hình khác, ghi rõ ngân sách tìm siêu tham số và không chỉ chọn lượt chạy tốt nhất của riêng GBDT.

## 20. Trường hợp không nên sử dụng

Không nên ưu tiên nếu nhãn quá ít hoặc không nhất quán, nếu chỉ cần một mô hình tuyến tính rẻ và đã đạt yêu cầu, hoặc nếu nghiên cứu yêu cầu giải thích toàn bộ bằng vài quy tắc. Hàng trăm cây không tương đương một cây nhỏ dễ đọc.

Nếu mục tiêu là kiểm định quan hệ nhân quả giữa trải nghiệm và ý định quay lại, GBDT dự đoán không tự cung cấp thiết kế nhận diện nhân quả. Cần xác định công cụ thống kê phù hợp với câu hỏi.

## 21. Các quyết định quan trọng khi thực hiện

Quyết định loss theo mục tiêu; số lớp theo codebook; learning rate cùng số vòng; độ sâu cùng kích thước lá; cách weighting lớp; và thước đo lựa chọn. Nếu chọn threshold, làm trên development và ghi rõ chi phí mong muốn.

Cũng cần quyết định có cho điểm sao vào mô hình không. Nếu nhãn cảm xúc được tạo từ chính điểm sao, sử dụng điểm sao làm đầu vào có thể biến bài toán thành tái tạo nhãn bằng biến đã định nghĩa nó.

## 22. Giải thích công thức và ký hiệu

Với mẫu $(x_i,y_i)$ và loss $L$:

$$r_{im}=-\left.\frac{\partial L(y_i,F(x_i))}{\partial F(x_i)}\right|_{F=F_{m-1}},\qquad F_m(x)=F_{m-1}(x)+\eta f_m(x).$$

$f_m$ là cây cập nhật, bao gồm giá trị lá đã ước lượng; $\eta$ là learning rate. Với binary log loss và logit chuẩn:

$$p=\sigma(F)=\frac{1}{1+e^{-F}},\quad L=-y\log p-(1-y)\log(1-p),\quad r=y-p.$$

Khi $F=0$, $p=0.5$. Nếu một lá cập nhật có giá trị 2 và $\eta=0.1$, logit mới là 0.2, nên xác suất khoảng 0.5498. Giá trị lá 2 ở đây là giả định minh họa, không phải luôn bằng residual.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Ví dụ chạy được dưới đây chỉ kiểm tra pipeline và phép đổi logit. Tám câu train và bốn câu kiểm tra tự tạo quá nhỏ để ước lượng hiệu năng thực tế.

```python
import numpy as np
from sklearn.pipeline import make_pipeline
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.metrics import f1_score

train = ["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
         "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
         "dịch vụ tệ bẩn", "khách sạn bẩn tệ"]
y = [1, 1, 1, 1, 0, 0, 0, 0]
test = ["phòng sạch zzholdout", "dịch vụ tốt", "phòng bẩn", "dịch vụ tệ"]
yt = [1, 1, 0, 0]
pipe = make_pipeline(
    CountVectorizer(ngram_range=(1, 2)),
    GradientBoostingClassifier(n_estimators=30, learning_rate=0.1,
                               max_depth=1, random_state=42))
pipe.fit(train, y)
p = pipe.predict_proba(test)
assert "zzholdout" not in pipe[0].vocabulary_
assert np.isfinite(p).all() and np.allclose(p.sum(axis=1), 1)
logit = pipe.decision_function(test)
assert np.allclose(p[:, 1], 1 / (1 + np.exp(-logit)))
assert np.isclose(1 / (1 + np.exp(-0.2)), 0.5498339973)
print("Macro-F1 trên dữ liệu đồ chơi:", f1_score(yt, pipe.predict(test), average="macro"))
```

Không thay câu hoặc tham số để tối đa điểm bốn mẫu này rồi báo như kết quả nghiên cứu. Trong dự án thật cần tập development độc lập và phân tích độ bất định.

## 24. Tình huống sai và phản ví dụ

Phản ví dụ: mô hình luôn dự đoán tích cực đạt accuracy cao ở tập có 95% câu tích cực. Điều đó không chứng minh nó phát hiện khiếu nại tốt; recall lớp tiêu cực có thể bằng 0.

Một phản ví dụ khác là cùng đánh giá sao chép xuất hiện ở train và test. Thêm nhiều cây có thể giúp ghi nhớ tốt hơn và cho điểm test cao giả tạo. Loại trùng theo nội dung và nhóm phải diễn ra trước khi đánh giá.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Xem đường loss train/development, khoảng cách hiệu năng và lỗi theo độ dài câu, phủ định, khách sạn, thời gian. Lặp split hợp lý hoặc dùng đánh giá theo nhóm; ghi độ biến động thay vì chỉ một số điểm.

Nếu kiểm tra xác suất, tách dữ liệu calibration phù hợp. Với bootstrap, lấy mẫu theo đơn vị độc lập của thiết kế, chẳng hạn khách sạn khi các phản hồi cùng khách sạn phụ thuộc nhau. Không mặc định mỗi câu là quan sát độc lập.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu train tốt nhưng development kém, kiểm tra rò rỉ và nhãn trước, rồi giảm độ sâu, tăng kích thước lá hoặc điều chỉnh learning rate/số vòng. Nếu cả hai kém, xem lại biểu diễn, mục tiêu và mức underfit.

Khi chi phí quá lớn hoặc văn bản sparse không phù hợp, so với mô hình tuyến tính hoặc biểu diễn gọn hơn. Chọn phương án bằng protocol đã định; không dùng test để quyết định chuyển phương pháp rồi vẫn gọi test là độc lập.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ: “Trên tập khách sạn giữ lại, mô hình đạt macro-F1 X theo thiết kế Y.” Kết luận có thể là mô hình hỗ trợ phân loại trong phạm vi dữ liệu đó, với giới hạn đã phân tích.

Không suy ra “khách hàng ngành khách sạn đều hài lòng” từ nhãn của mẫu thuận tiện. Cũng không suy ra một feature quan trọng gây ra cảm xúc; nó có thể đại diện ngôn ngữ nền tảng, cơ chế gán nhãn hoặc khách sạn cụ thể.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết phần phương pháp: “Nghiên cứu sử dụng GBDT với log loss cho phân loại [số lớp]. Toàn bộ biểu diễn được fit trong train của từng fold. Learning rate, số cây và độ sâu được chọn theo [metric] trên [thiết kế development]. Test gồm [đơn vị giữ lại] và chỉ được dùng sau khi khóa cấu hình.”

Điền giá trị thật và báo phiên bản, seed, không gian tìm kiếm, tiêu chí dừng. Phần kết quả cần bảng theo lớp và mô tả lỗi; không sao chép điểm số của dữ liệu đồ chơi.

## 29. Khả năng tái lập và tính minh bạch

Lưu codebook, danh sách ID của split, quy tắc loại trùng, cấu hình vectorizer, mapping nhãn, các siêu tham số, seed và phiên bản thư viện. Lưu pipeline để thứ tự cột không bị đổi giữa train và dự đoán.

Công khai dữ liệu chỉ khi được phép. Nếu không, công khai dữ liệu tổng hợp, mã tái tạo và mô tả quy trình cùng thống kê đủ đánh giá. Ghi nhận mọi lượt tìm mô hình để tránh chỉ báo cáo thử nghiệm thuận lợi.

## 30. Đạo đức nghiên cứu liên quan

Đánh giá khách sạn có thể chứa tên, liên hệ hoặc thông tin nhạy cảm về khách và nhân viên. Cần giảm dữ liệu nhận diện, kiểm soát quyền truy cập và dùng trích dẫn minh họa đã ẩn danh hoặc tự tạo.

Kiểm tra lỗi theo nhóm ngôn ngữ và bối cảnh khi có cơ sở thu thập hợp lệ. Không dùng điểm số tự động làm căn cứ duy nhất để phạt nhân viên hoặc gán động cơ cho người viết; sai phân loại cần cơ chế xem xét.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Vì sao y−p của log loss không phải cập nhật xác suất trực tiếp?
2. Với F=0 và cập nhật +0.2, tính xác suất mới.
3. Thiết kế split để đánh giá trên khách sạn chưa có trong train.
4. Nêu hai nguyên nhân feature importance cao nhưng không có tác động nhân quả.

Gợi ý: xác suất mới khoảng 0.5498; split cần giữ khách sạn theo nhóm. Bài mở rộng: thêm câu phủ định vào một tập thách thức độc lập, quan sát lỗi và giải thích vai trò biểu diễn thay vì kết luận thuật toán “hiểu” phủ định.

## 32. Những điểm còn tranh luận

Cần nghiên cứu thực nghiệm mức lợi ích của phi tuyến trên từng biểu diễn văn bản, tính ổn định của giải thích và khả năng chuyển miền. Đó là các câu hỏi có điều kiện dữ liệu, không thể chốt bằng tên thuật toán.

Một đánh đổi khác là thời gian tìm siêu tham số so với độ cải thiện ngoài mẫu. Nếu mức tăng nhỏ và không ổn định, mô hình đơn giản hơn có thể phù hợp hơn với mục đích triển khai và tái lập của nghiên cứu.

