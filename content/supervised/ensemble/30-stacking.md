# Stacking — Học mô hình kết hợp từ dự đoán ngoài fold

> **Nhánh:** MACHINE LEARNING → Supervised: Ensemble. **Nguồn:** dòng 31 bảng Excel. **Đối chiếu:** 28/10/2026. Tập trung stacking phân loại hai tầng, tạo meta-feature out-of-fold và giữ tiền xử lý bên trong từng base pipeline. Mã kiểm tra với scikit-learn 1.8.0. Ví dụ và dữ liệu khách sạn đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bản chất | Học một meta-model để kết hợp đầu ra các base models |
| Mốc tham khảo | Wolpert, Stacked Generalization, 1992 [S1] |
| Đầu vào | Dữ liệu có nhãn, base pipelines, cách tạo OOF và meta-model |
| Đầu ra | Dự đoán của meta-model từ các meta-features đúng schema |
| Rủi ro trọng tâm | Dùng dự đoán in-sample hoặc tiền xử lý ngoài fold làm rò rỉ |
| Phần mềm | sklearn.ensemble.StackingClassifier; không dùng cv="prefit" trên cùng tập đã fit base |

## 1. Định nghĩa học thuật

Stacking, hay stacked generalization, huấn luyện một mô hình tầng sau để kết hợp đầu ra của một hoặc nhiều mô hình tầng trước. Trong thiết kế hai tầng thường gặp, base models tạo các dự đoán ngoài fold cho train, rồi meta-model học quan hệ giữa các dự đoán này với nhãn [S1].

Điểm cốt lõi không chỉ là xếp nhiều thuật toán vào một danh sách. Meta-model cần được học trên thông tin có hành vi ngoài mẫu phù hợp; nếu dùng dự đoán của base model đã học chính nhãn từng mẫu, nó có thể học cách kết hợp một mức tự tin không tồn tại khi triển khai.

## 2. Giải thích đơn giản

Hình dung hai người phân tích review, một dùng từ/cụm từ và một nhìn các mảnh ký tự. Người thứ ba học khi nào nên dựa nhiều hơn vào đầu ra của mỗi người. Để dạy người thứ ba công bằng, cần cho họ xem dự đoán của hai người đầu trên những review mà hai người đó chưa học nhãn.

Nếu hai người đầu đã thuộc đáp án của các review luyện tập, người thứ ba chỉ thấy thành tích quá đẹp. Đó là lý do cần dự đoán ngoài fold, không chỉ thêm một tầng Logistic Regression sau các xác suất train.

## 3. Mục đích và vấn đề giải quyết

Stacking phù hợp để kiểm tra liệu các nguồn dự đoán bổ sung có thể được kết hợp tốt hơn từng mô hình riêng. Trong cảm xúc khách sạn, có thể thử base model từ word n-gram và char n-gram, hoặc các biểu diễn khác có lý do.

Câu hỏi nghiên cứu là liệu quy tắc kết hợp được học có cải thiện ngoài mẫu so với base tốt nhất và Voting đơn giản. Không mặc định nhiều tầng luôn tốt hơn. Cần đủ nhãn để học tầng sau và đánh giá mức tăng so với chi phí tạo OOF/huấn luyện lại.

## 4. Điều kiện và giả định sử dụng

Cần nhãn tin cậy, các base pipelines có đầu ra tương thích và một thiết kế tạo dự đoán ngoài mẫu phù hợp phụ thuộc dữ liệu. Mỗi fold train phải có dữ liệu/lớp đủ cho estimator; số fold mặc định không luôn phù hợp tập nhỏ.

Mỗi meta-feature của một hàng phải được tạo mà không học nhãn hàng đó trong base fit tương ứng. Vocabulary, IDF, feature selection, calibration hay metric learning được học cũng phải tôn trọng phạm vi fold. OOF không đồng nghĩa độc lập về khách sạn nếu splitter chỉ ngẫu nhiên theo review.

## 5. Cơ chế hoạt động bên trong

Chia outer-train thành các fold nội bộ. Với mỗi fold, fit từng base pipeline trên phần còn lại và dự đoán phần giữ lại. Ghép các dự đoán về đúng thứ tự hàng để có ma trận Z_OOF; huấn luyện meta-model với Z_OOF và y.

Sau đó fit các base pipelines trên toàn outer-train để phục vụ dự đoán mới. Với một review mới, lấy đầu ra của những base đã refit, tạo vector z rồi đưa vào meta-model đã học. Tập huấn luyện ít hơn trong OOF và toàn train ở bước refit có thể tạo khác biệt phân bố; cần đánh giá hệ thống cuối trên ngoài mẫu.

## 6. Quy trình áp dụng trong nghiên cứu

1. Định nghĩa nhãn, đơn vị và mục tiêu suy rộng; loại trùng và khóa test.
2. Chọn base pipelines và meta-model theo cơ sở rõ ràng.
3. Trong mỗi lượt đánh giá, tạo OOF chỉ từ phần train của lượt đó.
4. Đặt vectorizer/scaler/chọn feature bên trong từng base pipeline để chúng được fit lại trong fold nội bộ.
5. Huấn luyện meta-model trên OOF, refit base trên toàn train tương ứng.
6. Chọn cấu hình hệ thống bằng development hoặc outer-CV bao quanh toàn bộ stacking.
7. Khóa hệ thống rồi đánh giá test theo lớp, nhóm và chi phí.

CV dùng tạo meta-feature không tự cung cấp điểm đánh giá độc lập của toàn stack. Nếu tuning base trong quá trình tạo OOF, phạm vi tuning cũng cần nằm trong phần train tương ứng khi muốn giữ OOF nghiêm ngặt. Test không tham gia lựa chọn base, meta-model hay threshold.

## 7. Ví dụ minh họa xuyên suốt

Giả định hai base models cho review “phòng sạch nhưng cách âm kém” xác suất tích cực lần lượt 0.8 và 0.4. Meta-model nhận vector z=(0.8,0.4), không nhất thiết lấy trung bình 0.6.

Trong phép tính tự tạo ở mục 22, một meta Logistic Regression cho xác suất khoảng 0.6411. Hệ số ở ví dụ được đặt để hiểu công thức, không được ước lượng từ review thật. Việc câu này cần nhãn nào phải theo codebook; không để nhãn thay đổi tùy thuật toán nào thắng.

## 8. Kết quả và cách diễn giải

Đầu ra cuối thuộc meta-model. Nó không phải tỷ lệ số base đồng ý và không nhất thiết nằm giữa xác suất nhỏ nhất/lớn nhất của base nếu meta-model không bị ràng buộc theo kiểu convex averaging.

Hệ số meta-model mô tả cách nó kết hợp các meta-features trong không gian đã chọn. Hệ số lớn không chứng minh một base model “quan trọng gấp đôi” theo nghĩa phổ quát, nhất là khi các cột tương quan. Xác suất cuối vẫn cần kiểm tra calibration và lỗi ngoài mẫu.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là có thể học quy tắc kết hợp linh hoạt, tận dụng các biểu diễn/thuật toán bổ sung và điều chỉnh đóng góp bằng một meta-model. Một meta-model đơn giản có thể giới hạn mức phức tạp so với nhiều tầng lớn.

Nhược điểm là huấn luyện tốn hơn, nhiều điểm dễ leakage và khó kiểm toán hơn Voting. Meta-model có thể overfit khi ít nhãn hoặc nhiều cột dự đoán tương quan. Độ đa dạng tên thuật toán không bảo đảm lỗi bổ sung; hệ thống phức tạp có thể thua base tốt nhất.

## 10. So sánh với phương pháp gần nhau

| Phương pháp/thiết kế | Khác với stacking OOF |
|---|---|
| Voting | Tổng hợp bằng quy tắc/trọng số định trước trong cấu hình, không mặc định học meta-classifier |
| Bagging | Tạo các bộ học qua resampling rồi tổng hợp |
| Boosting | Thêm learner tuần tự theo trạng thái/lỗi ensemble |
| Blending | Thường dùng một holdout riêng để huấn luyện bộ kết hợp thay vì OOF nhiều fold |
| In-sample stacking | Dùng đầu ra từ mô hình đã thấy chính nhãn mẫu; nguy cơ lạc quan |

Từ blending được dùng khác nhau trong thực hành, nên hãy mô tả dữ liệu nào fit base, dữ liệu nào fit combiner. Tên gọi không thay thế sơ đồ thông tin.

## 11. Sai lầm thường gặp và cách khắc phục

Sai lầm nghiêm trọng: fit TF-IDF trên toàn outer-train, rồi đặt riêng classifiers vào StackingClassifier và gọi toàn bộ OOF là hoàn toàn chưa thấy dữ liệu. Các hàng giữ lại đã tham gia học IDF/vocabulary. Ví dụ bài đặt vectorizer bên trong mỗi base pipeline để tránh điều đó.

Một lỗi khác là gọi stack.transform(train) sau fit rồi coi đó là Z_OOF dùng huấn luyện meta. Transform dùng base đã refit, nên đây là đầu ra in-sample của chúng. Không dùng cv="prefit" với base và meta cùng học trên một tập rồi báo như OOF; tài liệu cảnh báo nguy cơ overfit [S2].

## 12. Tài liệu tham khảo

- [S1] Wolpert, D. H. (1992). *Stacked Generalization*. Neural Networks, 5(2), 241–259. [DOI](https://doi.org/10.1016/S0893-6080(05)80023-1), [trang nhà xuất bản](https://www.sciencedirect.com/science/article/pii/S0893608005800231).
- [S2] scikit-learn. [StackingClassifier, phiên bản 1.8](https://scikit-learn.org/1.8/modules/generated/sklearn.ensemble.StackingClassifier.html).
- [S3] scikit-learn. [cross_val_predict, phiên bản 1.8](https://scikit-learn.org/1.8/modules/generated/sklearn.model_selection.cross_val_predict.html).
- [S4] scikit-learn. [Common pitfalls and recommended practices](https://scikit-learn.org/1.8/common_pitfalls.html).

[S1] là nền tảng; [S2–S3] xác định hành vi tạo dự đoán và API; [S4] đối chiếu phạm vi tiền xử lý. Ví dụ và hướng dẫn protocol trong bài được tự soạn, không phải benchmark của tài liệu gốc.

## 13. Vị trí trong toàn bộ nghiên cứu

Stacking nằm ở giai đoạn kết hợp mô hình sau khi đã xác định dữ liệu, nhãn và các ứng viên cơ sở. Đánh giá hệ thống cần bao quanh toàn bộ quy trình tạo OOF, fit meta và refit base, không chỉ đánh giá một tầng riêng lẻ.

Nếu mục tiêu nghiên cứu là so biểu diễn, cần phân biệt giá trị của word/char/embedding với lợi ích của meta-model. Một hệ thống được thêm nguồn dữ liệu không thể được so như chỉ thay thuật toán kết hợp.

## 14. Kiến thức nền cần biết trước

Cần hiểu cross-validation, train/development/test, clone/refit estimator, xác suất, decision score và Logistic Regression. Nên biết vì sao dự đoán in-sample thường có hành vi khác ngoài mẫu và vì sao tiền xử lý cũng có thể leakage.

Với dữ liệu khách sạn, cần thêm group split và time split. Đối với mỗi con số trong ma trận meta, hãy trả lời được: mô hình nào tạo ra nó, fit trên hàng nào, dùng vocabulary nào và có từng dùng nhãn của hàng đang dự đoán không.

## 15. Nền tảng lý thuyết và logic hình thành

Stacking chuyển bài toán kết hợp sang một không gian mới, nơi các đầu ra base models là đặc trưng. Meta-model học từ bằng chứng về cách các base hoạt động ngoài mẫu trong dữ liệu phát triển, thay vì dựa vào độ khớp train của chúng.

Mục tiêu là học một quy tắc kết hợp có khả năng suy rộng. OOF giúp tạo thông tin huấn luyện phù hợp hơn nhưng không đảm bảo hết mọi leakage: lựa chọn mô hình, feature ngoài pipeline, dữ liệu trùng và nhóm phụ thuộc vẫn cần được kiểm soát. Thêm tầng còn tăng độ phức tạp cần regularization.

## 16. Các thành phần cấu tạo

Thành phần gồm base pipelines, splitter nội bộ, hàm tạo meta-feature, mapping cột/lớp, ma trận OOF, meta-model và cơ chế refit/predict. Cần quản lý thứ tự hàng khi ghép OOF và thứ tự base models ở train lẫn deploy.

Trong ví dụ nhị phân dùng predict_proba, sklearn bỏ một cột xác suất của mỗi base vì hai cột cộng bằng 1 [S2]. Với hai base, passthrough=False, ma trận đưa vào meta có hai cột. Không tự suy cùng kích thước cho đa lớp hoặc khi dùng decision_function.

## 17. Các loại và biến thể

Có stacking hai tầng, nhiều tầng, dùng probability, decision score hoặc nhãn làm meta-feature. Meta-model có thể tuyến tính hoặc phi tuyến. Mỗi biến thể thay đổi mức thông tin, chi phí và nguy cơ overfit.

Passthrough bổ sung feature gốc vào tầng meta, khác chỉ dùng đầu ra base. Với raw text, không bật passthrough=True rồi đưa chuỗi thô vào một Logistic Regression số học. Cần một thiết kế biến đổi phù hợp, giữ phạm vi fit trong fold và quản lý chiều feature rõ ràng.

## 18. Đầu vào và đầu ra

Đầu vào gồm X/y, base estimators có đầu ra đã định nghĩa và splitter. Z_OOF có n hàng, số cột phụ thuộc số base, số lớp và stack_method. Không trộn cột p(positive) của base này với p(negative) của base khác mà không biết mapping.

Đầu ra cuối là dự đoán meta-model; các base refit cũng được lưu. Tại deploy, không tạo OOF cho một review mới: chỉ dùng base đã huấn luyện để tạo vector đầu vào meta. Việc fit lại bằng nhãn của review mới là một quy trình cập nhật khác cần quản lý.

## 19. Căn cứ lựa chọn

Đáng thử khi các base models có chất lượng đủ tốt, lỗi bổ sung và dữ liệu đủ để học bộ kết hợp. Cần có baseline base tốt nhất cùng Voting đơn giản để biết tầng học thêm có đem lợi ích thực tế không.

Căn cứ chọn cuối là metric ngoài mẫu ổn định và chi phí chấp nhận được. Nếu mức tăng chỉ có trên OOF đã dùng fit meta, bằng chứng chưa đủ. Việc có nhiều base models hoặc nhiều biểu diễn không tự chứng minh nên dùng stacking.

## 20. Trường hợp không nên sử dụng

Không ưu tiên khi dữ liệu quá ít so với số meta-features, base models gần như giống nhau, hoặc không có nguồn lực kiểm soát OOF/leakage. Không dùng để bù một codebook cảm xúc mơ hồ chưa được rà soát.

Nếu dữ liệu thời gian cần dự đoán tương lai, một splitter mặc định có thể không đúng mục tiêu. Đặc biệt, không thay TimeSeriesSplit vào mọi API rồi cho rằng tự hoạt động: cross_val_predict yêu cầu cấu trúc phần giữ lại phù hợp; các quan sát warm-up thường không có OOF theo thời gian và cần thiết kế riêng [S3].

## 21. Các quyết định quan trọng khi thực hiện

Chọn base models, biểu diễn, stack_method, meta-model/regularization, số fold và chiến lược nhóm/thời gian. Nếu dùng group folds, có thể chuẩn bị iterable split hợp lệ trước khi fit thay vì giả định groups tự được truyền đúng qua mọi tầng/phiên bản.

Với forward stacking theo thời gian, có thể cần tự tạo meta-data trên các giai đoạn sau warm-up, chỉ dùng lịch sử để dự đoán từng giai đoạn. Không ép các hàng warm-up có dự đoán bằng mô hình đã thấy tương lai. Chọn cấu hình và threshold ở development hoặc outer-CV, không ở test cuối.

## 22. Giải thích công thức và ký hiệu

Gọi $I_k$ là các hàng giữ lại ở fold k; $f_m^{(-k)}$ là toàn bộ base pipeline m fit trên phần train tương ứng. Với $i\in I_k$:

$$z_{im}=f_m^{(-k)}(x_i).$$

Trong ví dụ nhị phân, $z_{im}$ là xác suất lớp 1. Meta-model g được fit bằng $(z_i,y_i)$. Với dữ liệu mới, dùng các base đã refit trên toàn train D:

$$\widehat y(x)=g\big(f_1^{(D)}(x),\ldots,f_M^{(D)}(x)\big).$$

Nếu g là Logistic Regression nhị phân:

$$p(1\mid z)=\sigma\left(\beta_0+\sum_m\beta_mz_m\right).$$

Đặt giả định $\beta_0=-0.5$, $\beta_1=1.2$, $\beta_2=0.3$ và $z=(0.8,0.4)$ thì logit $=-0.5+0.96+0.12=0.58$, xác suất khoảng 0.6411. Các hệ số được đặt để minh họa; không phải output thật của model ở mục 23.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mã fit StackingClassifier thật trên raw text; vectorizer nằm trong mỗi base pipeline. Hai fold nội bộ chỉ để minh họa trên dữ liệu tự tạo. Tập kiểm tra nhỏ không được dùng chọn cấu hình.

```python
import numpy as np
from sklearn.pipeline import make_pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.naive_bayes import MultinomialNB
from sklearn.model_selection import StratifiedKFold
from sklearn.ensemble import StackingClassifier

train = np.array(["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
                  "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
                  "dịch vụ tệ bẩn", "khách sạn bẩn tệ"])
y = np.array([1, 1, 1, 1, 0, 0, 0, 0])
test = np.array(["phòng sạch zzholdout", "dịch vụ tốt", "phòng bẩn", "dịch vụ tệ"])
word = make_pipeline(
    TfidfVectorizer(ngram_range=(1, 2)),
    LogisticRegression(C=1.0, max_iter=1000, random_state=42))
char = make_pipeline(
    TfidfVectorizer(analyzer="char", ngram_range=(2, 4)),
    MultinomialNB(alpha=1.0))
cv = StratifiedKFold(n_splits=2, shuffle=True, random_state=42)
splits = list(cv.split(train, y))
heldout_count = np.zeros(len(y), dtype=int)
for fit_idx, hold_idx in splits:
    assert set(fit_idx).isdisjoint(hold_idx)
    assert set(y[fit_idx]) == {0, 1}
    heldout_count[hold_idx] += 1
assert np.all(heldout_count == 1)
stack = StackingClassifier(
    estimators=[("word", word), ("char", char)],
    final_estimator=LogisticRegression(C=1.0, max_iter=1000),
    stack_method="predict_proba", cv=splits,
    passthrough=False, n_jobs=1)
stack.fit(train, y)
assert "zzholdout" not in stack.named_estimators_["word"][0].vocabulary_
prob = stack.predict_proba(test)
assert np.isfinite(prob).all() and np.allclose(prob.sum(axis=1), 1)

# transform(test) dùng base đã refit; không phải ma trận OOF train.
Znew = stack.transform(test)
manual_Z = np.column_stack([base.predict_proba(test)[:, 1]
                            for base in stack.estimators_])
assert Znew.shape == (len(test), 2)
assert np.allclose(Znew, manual_Z)
assert np.allclose(prob, stack.final_estimator_.predict_proba(Znew))
assert np.isclose(1/(1+np.exp(-0.58)), 0.6410674063)
print("Nhãn trên ví dụ:", stack.predict(test))
print("Số cột meta cho dữ liệu mới:", Znew.shape[1])
```

Mã kiểm tra partition, hình dạng meta-feature và đường dự đoán của hệ thống, không đo hiệu quả ngoài miền. Việc vocabulary chỉ học trong fold nội bộ được bảo đảm bởi cấu trúc base pipeline được clone/fit trong quy trình OOF của API; bộ vocabulary truy cập sau fit là bản refit toàn train, không phải các bản nội bộ đã dùng tạo OOF.

## 24. Tình huống sai và phản ví dụ

Phản ví dụ: một cây sâu học thuộc nhãn train, tạo xác suất gần 0/1 hoàn hảo; meta-model được fit trên các xác suất in-sample này và quá tin cây. Khi gặp review mới, cây sai nhiều hơn, nhưng meta-model chưa học cách đối phó.

Một ví dụ khác là dùng cùng một review sao chép ở hai fold. Hàng được gọi là OOF vẫn có bản gần trùng ở base-train. Chỉ kiểm tra chỉ số không giao nhau chưa đủ; cần xử lý trùng nội dung và phụ thuộc theo nhóm trước.

## 25. Kiểm tra chất lượng và độ vững của kết quả

So toàn stack với từng base và Voting trên cùng outer split. Kiểm tra các kiểu lỗi mà ensemble sửa được hoặc tạo thêm; đo tương quan lỗi ngoài mẫu thay vì chỉ tương quan xác suất train.

Kiểm toán OOF: mỗi hàng được giữ lại đúng số lần theo thiết kế, không mất thứ tự, mapping lớp nhất quán và mọi biến đổi fit đúng phạm vi. Kiểm tra độ nhạy fold/seed và số meta-features. Nếu kiểm tra calibration/threshold, thực hiện theo thiết kế development độc lập phù hợp, không dùng test cuối.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu stack không hơn base tốt nhất, xem base có bổ sung thông tin không và meta-model có quá phức tạp không. Có thể giảm số base, tăng regularization hoặc dùng Voting đơn giản. Đừng thêm nhiều tầng chỉ để khớp một tập development nhỏ.

Nếu phát hiện leakage, tái tạo toàn bộ OOF và lựa chọn mô hình. Nếu nhóm/time split không được API hỗ trợ đúng cấu trúc cần thiết, tự xây quy trình OOF phù hợp hoặc dùng thiết kế blending có holdout độc lập; báo rõ lượng dữ liệu và giới hạn thay vì đổi tên cho quy trình sai.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ là chất lượng của toàn hệ thống trên dữ liệu chưa tham gia lựa chọn/huấn luyện theo protocol. Có thể kết luận tầng kết hợp có ích trong phạm vi đánh giá khi đối chứng đủ mạnh và chênh lệch ổn định.

Không suy hệ số meta thành tỷ lệ đóng góp nhân quả của thuật toán hay yếu tố khách sạn. Không gọi điểm meta fit trên OOF là điểm test của stack; OOF là dữ liệu huấn luyện tầng meta. Cũng không suy nhiều tầng nghĩa là hiểu cảm xúc sâu hơn.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu xây stacking hai tầng với base pipelines [...]. Meta-features được tạo bằng K-fold [theo nhóm/thời gian] trong phần train; toàn bộ biểu diễn được fit lại trong từng fold. Meta-classifier là [...]. Base models được refit trên toàn train trước dự đoán ngoài mẫu.”

Nêu stack_method, mapping cột/lớp, passthrough, cách chọn hyperparameters và outer evaluation. Thêm phiên bản, seed và chi phí. Nếu một bước dùng holdout/prefit, mô tả nguồn dữ liệu đó cụ thể, không trình bày như OOF khi không đúng.

## 29. Khả năng tái lập và tính minh bạch

Lưu split ở cả tầng đánh giá và tầng tạo OOF, thứ tự hàng/cột Z, base pipelines, meta-model, mapping nhãn, seed và phiên bản. Nếu lưu OOF để kiểm toán, kiểm soát quyền truy cập vì nó liên kết với nhãn và ID dữ liệu.

Lưu cả base đã refit phục vụ deploy; chỉ ma trận OOF không đủ dự đoán câu mới. Mã mục 23 đã chạy với sklearn 1.8.0 trên dữ liệu tự tạo. Tái lập thực nghiệm thật cần code và môi trường cùng dữ liệu được phép chia sẻ.

## 30. Đạo đức nghiên cứu liên quan

Stacking có thể làm tăng số bản dữ liệu/model và khó truy vết một lỗi ra quyết định. Cần quản lý quyền truy cập, nguồn dữ liệu, phiên bản và quy trình rà soát. Không đưa tên hoặc liên hệ của khách vào feature chỉ vì làm điểm cao hơn.

Các base có thể chia sẻ cùng thiên lệch theo ngôn ngữ hoặc nền tảng. Tầng meta không tự đảm bảo công bằng. Kiểm tra sai số theo bối cảnh có dữ liệu hợp lệ và không dùng kết quả tự động làm căn cứ duy nhất cho quyết định bất lợi với nhân viên.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Vì sao meta-model cần dự đoán OOF thay vì dự đoán in-sample?
2. Vectorizer nằm ngoài StackingClassifier gây vấn đề gì cho OOF?
3. stack.transform(train) sau fit có phải Z_OOF không?
4. Tính xác suất với logit 0.58.
5. Vì sao CV tạo meta-feature chưa là đánh giá cuối của stack?

Gợi ý: xác suất khoảng 0.6411; transform(train) dùng base refit. Bài mở rộng: lập bảng đường đi thông tin cho mỗi fold, chỉ rõ tập fit vectorizer/base/meta và tập đánh giá. Thiết kế đối chứng Voting trước khi chạy, không lựa chọn theo điểm test.

## 32. Những điểm còn tranh luận

Những câu hỏi còn cần thực nghiệm gồm mức lợi ích của meta-model trên biểu diễn tiếng Việt, độ nhạy với cỡ mẫu và độ lệch giữa base fit trên fold với refit toàn train. Không có bảo đảm thêm base hay thêm tầng sẽ cải thiện.

Đánh đổi khác là mức tăng chất lượng so với chi phí và khả năng kiểm toán. Với dữ liệu nhóm/thời gian, thiết kế OOF đúng có thể quan trọng hơn lựa chọn meta-classifier. Một nghiên cứu đáng tin cần công bố protocol thông tin, không chỉ liệt kê tên các tầng.

