# Bagging — Bootstrap aggregating cho phân loại cảm xúc

> **Nhánh:** MACHINE LEARNING → Supervised: Ensemble. **Nguồn:** dòng 30 bảng Excel. **Đối chiếu:** 28/10/2026. Tập trung bagging phân loại với cây cơ sở; phân biệt bỏ phiếu nhãn với trung bình xác suất trong API. Mã kiểm tra với scikit-learn 1.8.0. Ví dụ và dữ liệu khách sạn đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bản chất | Tạo nhiều bộ học từ các mẫu train biến đổi rồi tổng hợp dự đoán |
| Mốc tham khảo | Breiman, Bagging Predictors, bài tạp chí 1996; có báo cáo kỹ thuật trước đó [S1] |
| Đầu vào | Đặc trưng, nhãn, bộ học cơ sở và cấu hình lấy mẫu |
| Đầu ra | Nhãn/xác suất tổng hợp; có thể có ước lượng out-of-bag theo điều kiện |
| Tiếng Việt | Cần biểu diễn văn bản; bootstrap không tự hiểu phủ định |
| Phần mềm | sklearn.ensemble.BaggingClassifier; dùng estimator, không dùng tên API cũ base_estimator |

## 1. Định nghĩa học thuật

Bagging, viết tắt của bootstrap aggregating, tạo nhiều phiên bản bộ dự đoán bằng cách huấn luyện trên các tập bootstrap của dữ liệu train rồi tổng hợp đầu ra. Bootstrap thông thường lấy mẫu có hoàn lại, nên một quan sát có thể xuất hiện nhiều lần hoặc không xuất hiện ở một bộ học [S1].

Trong phân loại có thể tổng hợp bằng phiếu nhãn hoặc trung bình xác suất, tùy định nghĩa/triển khai. Trong hồi quy thường lấy trung bình giá trị dự đoán. Bài này tập trung phân loại cảm xúc, không đồng nhất bagging với mọi phương pháp ensemble.

## 2. Giải thích đơn giản

Hình dung nhiều người được phát các chồng review khác nhau rút từ cùng kho train. Một review có thể được phát lặp lại, còn review khác không nằm trong một chồng. Mỗi người học một mô hình rồi cả nhóm kết hợp dự đoán.

Khác biệt giữa các chồng giúp mô hình không lệ thuộc hoàn toàn vào một phiên bản dữ liệu. Nhưng nếu mọi mô hình đều mắc cùng lỗi vì không nhận ra “không sạch”, lấy trung bình không tự tạo hiểu biết về phủ định. Nhóm đông chỉ hữu ích khi có thông tin và mức đa dạng phù hợp.

## 3. Mục đích và vấn đề giải quyết

Bagging có thể dùng để giảm độ biến động của bộ học nhạy với thay đổi mẫu, chẳng hạn cây quyết định. Trong nghiên cứu khách sạn, có thể so một cây đơn với bagging nhiều cây trên cùng biểu diễn và split để kiểm tra ích lợi thực tế.

Câu hỏi nên là “bagging có cải thiện chất lượng và độ ổn định trên khách sạn giữ lại không?” thay vì “nhiều mô hình chắc chắn tốt hơn”. Cần đo cả lỗi theo lớp và chi phí lưu/huấn luyện/dự đoán. Không dùng điểm train như bằng chứng giảm overfit.

## 4. Điều kiện và giả định sử dụng

Cần nhãn hợp lệ, đặc trưng có sẵn khi dự đoán và bộ học cơ sở tương thích với đầu vào. Bagging thường có lý do mạnh hơn khi base learner biến động theo mẫu; với mô hình vốn ổn định, mức cải thiện có thể nhỏ hoặc không có.

Bootstrap theo từng review không tự tôn trọng cấu trúc phụ thuộc. Nếu các review trong cùng khách sạn hoặc người viết liên quan nhau, cần thiết kế split và phương án resampling phù hợp câu hỏi. BaggingClassifier mặc định lấy mẫu theo hàng không phải cluster bootstrap theo khách sạn.

## 5. Cơ chế hoạt động bên trong

Lấy m chỉ số có hoàn lại từ n hàng train cho từng bộ học, fit một bản sao estimator và lưu mô hình. Có thể lấy thêm một tập feature theo cấu hình. Các bộ học không cần đợi dự đoán của nhau như boosting, nên có thể tổ chức huấn luyện song song khi tài nguyên cho phép.

Khi dự đoán, mỗi bộ học nhận đúng các feature đã dùng. Trong ví dụ cây với sklearn, BaggingClassifier trung bình predict_proba rồi chọn lớp lớn nhất; nếu base estimator không có predict_proba, API dùng cơ chế phiếu [S2]. Không mô tả ví dụ này như luôn bỏ phiếu cứng giữa các nhãn cây.

## 6. Quy trình áp dụng trong nghiên cứu

1. Quy định nhãn và cách xử lý review trung tính/hỗn hợp.
2. Loại trùng, chia train/development/test theo mục tiêu thời gian/nhóm.
3. Fit biểu diễn trong train của từng lượt đánh giá.
4. Chạy cây đơn và bagging với base learner được mô tả rõ.
5. Chọn số bộ học, max_samples, max_features, độ sâu/kích thước lá bằng development/CV.
6. Kiểm tra độ biến động, lỗi theo nhóm và chi phí.
7. Khóa cấu hình rồi đánh giá test.

Nếu sử dụng out-of-bag (OOB), xác định nó ước lượng điều gì và có thỏa điều kiện độc lập mong muốn không. OOB không thay thế tự động một test giữ khách sạn hoặc tương lai, và không phải tập test độc lập để tuning vô hạn.

## 7. Ví dụ minh họa xuyên suốt

Tập train giả định gồm tám review, ID 0–7. Một bootstrap có chỉ số [0,0,1,2,4,4,6,7]. Review 0 và 4 được lặp; 3 và 5 không được bộ học này thấy. Hai review bị bỏ lại là OOB đối với riêng bộ học đó.

Cho một review mới, ba bộ học giả định trả xác suất tích cực 0.8, 0.6 và 0.3. Trung bình là 0.5667 nên chọn tích cực ở ngưỡng 0.5. Đây là số tự tạo để giải thích tổng hợp, không phải độ chính xác đã đo trên khách sạn thật.

## 8. Kết quả và cách diễn giải

Xác suất ensemble trong ví dụ là trung bình xác suất của các cây, không phải tỷ lệ khách hài lòng hay mức đồng thuận của người gán nhãn. Mức trung bình cao vẫn có thể sai nếu các bộ học cùng học shortcut.

Báo macro-F1, precision/recall từng lớp và confusion matrix. Nếu báo OOB score của API, ghi rõ metric mặc định của nó và không gọi nó là macro-F1 khi chưa tính. Đánh giá xác suất cần thêm calibration/proper scoring rule phù hợp mục tiêu, không chỉ nhãn đúng/sai.

## 9. Điểm mạnh, điểm yếu và giới hạn

Điểm mạnh là có thể giảm phương sai của learner không ổn định, tổng hợp nhiều phiên bản dữ liệu và hỗ trợ tổ chức huấn luyện độc lập. Cấu hình đơn giản còn tạo đối chứng hữu ích với một cây đơn hoặc Random Forest.

Điểm yếu là tăng chi phí, khó giải thích toàn bộ và không sửa được thiên lệch hệ thống hoặc feature thiếu thông tin. Hiệu quả giảm phương sai bị hạn chế khi dự đoán các mô hình tương quan cao. Bootstrap còn có thể làm một bộ học thiếu lớp hiếm; cần kiểm tra thay vì chỉ tăng số cây một cách máy móc.

## 10. So sánh với phương pháp gần nhau

| Phương pháp | Phân biệt |
|---|---|
| Cây đơn | Một cấu trúc học từ dữ liệu; bagging tổng hợp nhiều phiên bản |
| Random Forest | Có thêm cơ chế ngẫu nhiên feature khi tìm split; bagging cây không mặc định tương đương |
| Boosting | Xây tuần tự theo trạng thái/lỗi mô hình trước, khác cơ chế lấy mẫu độc lập cơ bản |
| Voting | Quy tắc tổng hợp; bagging còn chỉ định cách tạo nhiều bộ học bằng resampling |
| Stacking | Học mô hình kết hợp từ dự đoán ngoài fold, thay vì chỉ tổng hợp cố định |

Việc lấy feature subset một lần cho mỗi estimator trong BaggingClassifier khác với lấy feature ứng viên tại từng split của một Random Forest.

## 11. Sai lầm thường gặp và cách khắc phục

Không nhầm bootstrap với chia dữ liệu thành các phần không giao nhau: bootstrap có lặp và có bỏ sót. Không nhầm n_estimators với số mẫu mỗi bootstrap hoặc số vòng sửa gradient.

Đừng báo OOB như kiểm định trên khách sạn chưa gặp nếu cùng khách sạn vẫn ở các mẫu in-bag. Nếu vectorizer được fit một lần trên toàn train trước bagging, OOB review đã tham gia học vocabulary/IDF; OOB lúc đó không hoàn toàn giữ lại cả pipeline tiền xử lý. Cần một thiết kế ngoài mẫu phù hợp nếu muốn đánh giá end-to-end.

## 12. Tài liệu tham khảo

- [S1] Breiman, L. (1996). *Bagging Predictors*. Machine Learning, 24, 123–140. [Bài tạp chí/DOI](https://doi.org/10.1007/BF00058655). [Trang báo cáo kỹ thuật Berkeley](https://statistics.berkeley.edu/tech-reports/421) là bản trước bài tạp chí.
- [S2] scikit-learn. [BaggingClassifier, phiên bản 1.8](https://scikit-learn.org/1.8/modules/generated/sklearn.ensemble.BaggingClassifier.html).
- [S3] scikit-learn. [Hướng dẫn ensemble, phiên bản 1.8](https://scikit-learn.org/1.8/modules/ensemble.html).

[S1] cho ý tưởng nền tảng; [S2] cho quy tắc tổng hợp/API thực dùng; [S3] cho bối cảnh các biến thể. Công thức phương sai và các ví dụ dưới là giải thích với giả định được nêu rõ, không phải lời hứa mức cải thiện thực nghiệm.

## 13. Vị trí trong toàn bộ nghiên cứu

Bagging nằm ở phần xây và so sánh mô hình, sau khi chọn mẫu, gán nhãn và thiết kế biểu diễn. Nó không thay thế việc kiểm tra sự nhất quán nhãn hay tính đại diện của review.

Nếu dùng dự đoán để tạo chỉ số cảm xúc theo khách sạn, cần kiểm tra sai số phân loại trước khi tổng hợp và không coi số cây lớn là bằng chứng phép đo đáng tin. Việc suy rộng đến khách không viết review còn đòi hỏi phân tích cơ chế chọn mẫu.

## 14. Kiến thức nền cần biết trước

Nên hiểu bootstrap, lấy mẫu có/không hoàn lại, cây quyết định, bias–variance và cách đọc xác suất. Cần phân biệt bootstrap dùng để huấn luyện ensemble với bootstrap dùng để xây khoảng bất định; chúng không phải cùng mục đích chỉ vì dùng cùng phép lấy mẫu.

Với văn bản, cần biết ma trận sparse, vocabulary, TF-IDF và lý do mọi bước được học phải nằm trong train của mỗi fold đánh giá. Với dữ liệu có nhóm, cần hiểu khác biệt giữa lấy mẫu review và lấy mẫu cả khách sạn.

## 15. Nền tảng lý thuyết và logic hình thành

Một learner nhạy dữ liệu có thể thay đổi nhiều khi train biến động. Tổng hợp các phiên bản có thể làm dự đoán bớt biến động nếu các sai lệch không hoàn toàn cùng nhau. Tuy nhiên, lấy trung bình không triệt tiêu một sai lệch chung mà mọi mô hình cùng mắc.

Lập luận phương sai ở mục 22 áp dụng cho đầu ra số với các giả định tương quan/phương sai cụ thể. Nó giúp hiểu cơ chế, không chứng minh mọi metric phân loại sẽ tăng theo số bộ học. Chất lượng nhãn, biểu diễn và phân bố ngoài mẫu vẫn quyết định mức hữu ích.

## 16. Các thành phần cấu tạo

Các thành phần gồm base estimator, bộ sinh chỉ số bootstrap, quy tắc lấy feature, số bản sao, seed và bộ tổng hợp. Nếu dùng OOB còn có tập estimator không thấy từng hàng và cách gộp dự đoán riêng cho hàng đó.

Ở pipeline văn bản của bài, vectorizer được fit trên outer-train, sau đó bagging hoạt động trên ma trận đã khóa. Đây là lựa chọn chia sẻ biểu diễn; không khẳng định mỗi cây có vocabulary riêng. Bộ biến đổi dùng chung cần được lưu cùng ensemble.

## 17. Các loại và biến thể

Bootstrap aggregating lấy hàng có hoàn lại; pasting lấy hàng không hoàn lại; random subspaces lấy tập feature; random patches kết hợp lấy hàng và feature. API có thể hỗ trợ các chế độ qua tham số, nhưng không phải mọi chế độ đều là bootstrap bagging kinh điển [S3].

Bagging áp dụng cho phân loại hoặc hồi quy với bộ học phù hợp. Base learner không bắt buộc là cây. Khi đổi learner, kiểm tra hỗ trợ sparse, trọng số mẫu và cách tổng hợp; không chuyển nguyên giả định của cây sang một mô hình khác.

## 18. Đầu vào và đầu ra

Đầu vào là X, y, cấu hình lấy mẫu và estimator. Các ma trận mới phải giữ schema/thứ tự cột. Nếu mỗi estimator dùng subset feature, phải chuyển đúng subset khi tái tạo phép tổng hợp thủ công.

Đầu ra có nhãn/xác suất và danh sách estimator đã học; có thể truy cập indices mẫu/feature theo API. Nhãn thiếu trong một bootstrap cần được căn chỉnh vào không gian lớp chung khi tổng hợp. Không cộng các cột xác suất chỉ theo vị trí mà bỏ qua mapping lớp.

## 19. Căn cứ lựa chọn

Đáng thử khi base learner có biến động đáng kể và có đủ tài nguyên để tổng hợp. So sánh cần giữ cùng dữ liệu, biểu diễn và thiết kế đánh giá với base learner đơn, đồng thời công bố ngân sách tuning.

Căn cứ lựa chọn cuối là cải thiện ngoài mẫu có ý nghĩa với mục tiêu và độ bền chấp nhận được. Nếu mức tăng nhỏ nhưng chi phí nhiều lần, mô hình đơn giản có thể phù hợp hơn. Không lấy số cây hoặc danh tiếng ensemble làm tiêu chí thay metric thực nghiệm.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu learner vốn ổn định và đối chứng không cho thấy lợi ích, nếu tài nguyên dự đoán quá hạn chế, hoặc nếu bắt buộc giải thích bằng vài quy tắc ngắn. Bagging không tạo nhãn cho dữ liệu chưa gán nhãn.

Không dùng OOB để bỏ qua thiết kế nghiên cứu có nhóm/thời gian. Nếu nguồn dữ liệu đầy review trùng hoặc nhãn cảm xúc được tạo từ biến đầu vào đã có sẵn, resampling không sửa được vấn đề nền tảng đó.

## 21. Các quyết định quan trọng khi thực hiện

Chọn base learner, độ phức tạp, số estimator, số hàng mỗi lần lấy, có hoàn lại không và số feature mỗi estimator. Lựa chọn sampling phải được phân biệt với class weighting hoặc resampling để cân bằng lớp.

Quyết định trước có dùng OOB không và diễn giải nó ra sao. Nếu cần uncertainty hoặc nhóm khách sạn, thiết kế riêng phương án đánh giá phù hợp. Giới hạn số luồng để tránh đồng thời song song ở quá nhiều tầng; ghi thời gian và bộ nhớ với môi trường cụ thể.

## 22. Giải thích công thức và ký hiệu

Với n mẫu và mỗi bootstrap lấy m lần độc lập, đồng đều, có hoàn lại, xác suất một hàng không được chọn là:

$$P(OOB)=\left(1-\frac1n\right)^m.$$

Khi m=n và n lớn, giá trị tiến đến $e^{-1}\approx0.368$. Đây là kỳ vọng về cơ chế lấy mẫu, không phải mỗi cây bỏ đúng 36.8% review. Với n=8,m=8, xác suất khoảng 0.3436; số hàng phân biệt kỳ vọng là $8(1-0.3436)\approx5.25$.

Với B đầu ra số có cùng phương sai $\sigma^2$ và cùng tương quan từng cặp $\rho$:

$$\operatorname{Var}(\overline f)=\sigma^2\left[\rho+\frac{1-\rho}{B}\right].$$

Giả định này chỉ minh họa vai trò tương quan. Khi $\rho=1$, thêm B không giảm phương sai; khi $\rho=0$, phương sai giảm theo B. Với xác suất phân loại trong ví dụ, $\overline p(c\mid x)=B^{-1}\sum_bp_b(c\mid x)$ rồi chọn lớp lớn nhất; không áp dụng công thức phương sai như bảo đảm macro-F1.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mã chạy pipeline thật trên tám câu train tự tạo. Chọn cấu hình để minh họa, không tuning bằng bốn câu kiểm tra. Không bật OOB trong ví dụ nhỏ để tránh diễn giải một ước lượng không phù hợp thiết kế. Trong API 1.8, max_samples=None lấy số lần rút bằng số hàng train hiện tại; ví dụ không truyền sample_weight.

```python
import numpy as np
from sklearn.pipeline import make_pipeline
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import BaggingClassifier

assert np.isclose((1-1/8)**8, 0.3436089158)
assert set(range(8)) - set([0, 0, 1, 2, 4, 4, 6, 7]) == {3, 5}
assert np.isclose(np.mean([0.8, 0.6, 0.3]), 0.5666666667)
train = ["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
         "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
         "dịch vụ tệ bẩn", "khách sạn bẩn tệ"]
y = np.array([1, 1, 1, 1, 0, 0, 0, 0])
test = ["phòng sạch zzholdout", "dịch vụ tốt", "phòng bẩn", "dịch vụ tệ"]
pipe = make_pipeline(
    CountVectorizer(ngram_range=(1, 2)),
    BaggingClassifier(
        estimator=DecisionTreeClassifier(max_depth=2, random_state=42),
        n_estimators=20, max_samples=None, max_features=0.8,
        bootstrap=True, oob_score=False, random_state=42, n_jobs=1))
pipe.fit(train, y)
vec, bag = pipe[0], pipe[1]
assert "zzholdout" not in vec.vocabulary_
Xt = vec.transform(test)
prob = pipe.predict_proba(test)
assert np.isfinite(prob).all() and np.allclose(prob.sum(axis=1), 1)

# Bagging mã hóa nhãn cơ sở thành chỉ số lớp 0..K-1.
# Căn chỉnh cả feature và cột lớp khi tự tổng hợp.
manual = np.zeros_like(prob)
for estimator, features in zip(bag.estimators_, bag.estimators_features_):
    local = estimator.predict_proba(Xt[:, features])
    manual[:, estimator.classes_.astype(int)] += local
manual /= len(bag.estimators_)
assert np.allclose(prob, manual)
assert len(bag.estimators_samples_) == 20
assert all(len(sample) == len(train) for sample in bag.estimators_samples_)
print("Nhãn trên ví dụ:", pipe.predict(test))
print("Tổng hợp thủ công khớp predict_proba:", np.allclose(prob, manual))
```

Tập feature mỗi cây có thể khác nhau, nhưng vectorizer chỉ học một lần trên train trong pipeline này. Nếu đánh giá outer-CV, đưa cả pipeline vào từng fold. Không dùng OOB của estimator cuối để mặc định xác nhận tính độc lập của toàn bộ tiền xử lý.

## 24. Tình huống sai và phản ví dụ

Phản ví dụ: mọi cây dùng tên nền tảng như shortcut vì nhãn bị lệch theo nguồn. Bootstrap thay đổi review nhưng vẫn giữ liên hệ đó, nên ensemble cùng dự đoán sai khi chuyển nền tảng. Lấy trung bình không loại được một lệch chung.

Một phản ví dụ khác là lớp tiêu cực rất hiếm: một số bootstrap không có ví dụ tiêu cực đủ phong phú. Số estimator lớn không thay thế việc có dữ liệu lớp hiếm và thiết kế đánh giá lớp đó. Cần kiểm tra recall/precision thay vì chỉ accuracy chung.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đánh giá trên cùng split với base learner đơn; xem độ biến động theo seed và theo nhóm giữ lại. Đo learning curve theo số estimator bằng development để biết khi nào lợi ích bão hòa; ghi chi phí tương ứng.

Kiểm tra lỗi phủ định, câu hỗn hợp và khách sạn mới. Nếu báo OOB, kiểm tra mỗi hàng có đủ estimator OOB và đầu ra hữu hạn. Nếu cần khoảng bất định, dùng phương án phù hợp đơn vị độc lập của nghiên cứu, không lấy mức khác nhau giữa cây như khoảng tin cậy hợp lệ mặc định.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu không cải thiện, xem base learner có đủ biến động và các mô hình có lỗi tương tự nhau không. Có thể điều chỉnh độ phức tạp, lượng hàng/feature lấy mẫu hoặc biểu diễn bằng development. Không tăng số cây vô hạn khi sai lệch chủ yếu do nhãn hoặc thiếu ngữ nghĩa.

Nếu chi phí quá cao, giảm số estimator theo đánh đổi được đo hoặc chuyển mô hình đơn giản hơn. Khi phát hiện rò rỉ, sửa dữ liệu và chạy lại cả lựa chọn mô hình; không chỉ chỉnh dự đoán test cho đẹp.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ mô tả ensemble đạt metric nào, trên dữ liệu giữ lại nào, biến động ra sao và tốn bao nhiêu tài nguyên. Có thể kết luận bagging hữu ích hơn cây đơn trong phạm vi đó nếu đối chứng hỗ trợ.

Không kết luận “review khách sạn luôn độc lập”, “OOB là test hoàn toàn mới” hay “bagging loại hết overfit”. Không suy importance hoặc nhãn dự đoán thành bằng chứng yếu tố quản trị gây hài lòng; ensemble vẫn là công cụ dự đoán.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu sử dụng bagging với B=[...] bộ học cơ sở [...]. Mỗi bộ học nhận m=[...] mẫu có hoàn lại từ train và [cấu hình feature]. Toàn bộ biểu diễn được fit trong train của từng fold đánh giá. Cấu hình chọn trên [development/CV], test giữ lại theo [...].”

Nêu quy tắc tổng hợp thực tế: trung bình xác suất hay phiếu nhãn. Nếu dùng OOB, ghi mục đích, metric và giới hạn. Bổ sung phiên bản, seed, luồng, ngân sách tuning và số đo thật; không đưa dữ liệu ví dụ như kết quả nghiên cứu.

## 29. Khả năng tái lập và tính minh bạch

Lưu pipeline, mapping nhãn, schema, split ID, seed, tham số sampling và cấu hình base learner. Giữ phiên bản sklearn và môi trường thực thi vì API có thể thay đổi. Nếu nghiên cứu cần tái lập từng bootstrap, lưu indices hoặc thông tin đủ tái tạo chúng.

Mã minh họa được chạy với sklearn 1.8.0, xác nhận tổng hợp thủ công khớp API trên dữ liệu tự tạo. Điều đó không xác nhận hiệu năng trên bộ dữ liệu người dùng. Chỉ chia sẻ dữ liệu review khi có quyền và đã xử lý rủi ro nhận diện.

## 30. Đạo đức nghiên cứu liên quan

Bootstrap có thể lặp lại một review chứa thông tin cá nhân trong nhiều bản huấn luyện; việc nhân bản không tạo quyền sử dụng mới. Cần quản lý văn bản gốc, bản sao và model theo chính sách dữ liệu phù hợp.

Không dùng nhãn ensemble làm căn cứ duy nhất để phạt nhân viên hoặc bác bỏ khiếu nại. Kiểm tra sai số theo nhóm/bối cảnh khi có cơ sở dữ liệu hợp lệ và duy trì người rà soát. Ví dụ trong bài đều tự tạo để tránh đưa thông tin khách thật vào hướng dẫn.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Bootstrap khác chia train thành B phần rời nhau thế nào?
2. Với n=8,m=8, xác suất OOB là bao nhiêu?
3. Vì sao thêm cây không giúp nhiều nếu dự đoán tương quan gần 1?
4. BaggingClassifier với cây trong bài dùng trung bình xác suất hay luôn bỏ phiếu cứng?
5. Thiết kế đánh giá khi muốn suy rộng sang khách sạn chưa gặp.

Gợi ý: xác suất OOB khoảng 0.3436; cần giữ nhóm khách sạn. Bài mở rộng: so cây đơn và bagging trên cùng pipeline, đo metric theo lớp và tài nguyên, khóa ngân sách tuning trước. Không chọn số cây bằng test.

## 32. Những điểm còn tranh luận

Các câu hỏi còn cần kiểm chứng gồm mức ích lợi khi dùng biểu diễn văn bản hiện đại, cách resampling dữ liệu có nhóm và đánh đổi số estimator với tài nguyên. Không có số cây tốt nhất hoặc mức cải thiện cố định cho mọi nguồn review.

Một điểm khác là sử dụng OOB khi tiền xử lý được học chung. Cần nói rõ phạm vi giữ lại để người đọc không hiểu quá mức tính độc lập. Nghiên cứu nên công bố protocol và dữ liệu phụ thuộc thay vì chỉ báo một OOB score thuận lợi.

