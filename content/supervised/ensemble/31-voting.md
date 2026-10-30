# Voting — Kết hợp phiếu nhãn hoặc xác suất của nhiều mô hình

> **Nhánh:** MACHINE LEARNING → Supervised: Ensemble. **Nguồn:** dòng 32 bảng Excel. **Đối chiếu:** 28/10/2026. Tập trung hard/soft voting cho phân loại cảm xúc, có trọng số và không trọng số. Mã kiểm tra với scikit-learn 1.8.0; trọng số minh họa không được tối ưu trên test. Ví dụ và dữ liệu khách sạn đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bản chất | Quy tắc tổng hợp đầu ra các classifier |
| Mốc tham khảo | Không gán một năm phát minh duy nhất; tham khảo ensemble learning và API [S1–S2] |
| Đầu vào | Các classifier, mapping nhãn và quy tắc/trọng số tổng hợp |
| Đầu ra | Nhãn tổng hợp; soft voting có xác suất trung bình theo trọng số |
| Điểm phân biệt | Hard dùng nhãn; soft dùng phân bố xác suất |
| Phần mềm | sklearn.ensemble.VotingClassifier; ví dụ base models là các pipeline văn bản |

## 1. Định nghĩa học thuật

Voting là cách tổng hợp đầu ra của nhiều bộ phân loại để quyết định nhãn cuối. Hard voting dùng các nhãn dự đoán; soft voting dùng xác suất cho từng lớp rồi chọn lớp có giá trị tổng hợp lớn nhất. Có thể cho các mô hình trọng số khác nhau. Trong đa lớp, lớp nhiều phiếu nhất có thể thắng dù không quá một nửa tổng phiếu; cần phân biệt plurality với đa số tuyệt đối.

Voting mô tả quy tắc kết hợp, không tự chỉ định cách tạo dữ liệu cho từng mô hình như bootstrap của bagging. Nó cũng không mặc định học một meta-classifier như stacking. Hệ thống có thể gồm nhiều thuật toán hoặc nhiều cấu hình cùng thuật toán; cần mô tả đầy đủ các thành viên.

## 2. Giải thích đơn giản

Ba người cùng đọc một review. Hard voting chỉ hỏi mỗi người chọn tích cực hay tiêu cực rồi đếm phiếu. Soft voting còn sử dụng các con số xác suất mà họ đưa ra và lấy trung bình theo quy tắc.

Nếu hai người chỉ hơi nghiêng tích cực còn người thứ ba cho xác suất tích cực rất thấp, hard và soft có thể chọn hai nhãn khác nhau. Các con số chỉ hữu ích khi mang ý nghĩa xác suất phù hợp; một mô hình rất tự tin nhưng sai có thể kéo soft voting lệch mạnh.

## 3. Mục đích và vấn đề giải quyết

Voting có thể làm baseline ensemble đơn giản cho cảm xúc khách sạn, kiểm tra liệu sự kết hợp các mô hình có bổ sung giá trị so với từng mô hình riêng. Nó cũng là đối chứng để xem một tầng học thêm của stacking có thực sự cần thiết không.

Có thể thử mô hình word n-gram với char n-gram hoặc biểu diễn khác nếu có lý do. Câu hỏi nên là cải thiện metric ngoài mẫu nào, với chi phí ra sao. Không chọn thêm mô hình chỉ vì tên khác, vì nhiều mô hình có thể cùng học một shortcut.

## 4. Điều kiện và giả định sử dụng

Các mô hình phải dự đoán cùng một định nghĩa nhãn và được áp dụng lên cùng đơn vị dữ liệu. Với soft voting, cần xác suất hợp lệ có mapping lớp nhất quán; các API/định dạng khác nhau cần được căn chỉnh trước.

Để Voting có ích, các thành viên thường cần chất lượng và lỗi bổ sung phù hợp; đây là động cơ thiết kế chứ không phải bảo đảm. Không đòi các lỗi độc lập trong mọi ứng dụng, nhưng nếu dùng công thức lý thuyết giả định độc lập phải nêu rõ. Dữ liệu đánh giá vẫn cần độc lập với quá trình lựa chọn thành viên/trọng số.

## 5. Cơ chế hoạt động bên trong

Fit các base pipelines theo dữ liệu train cho phép. Với hard voting, mỗi base tạo nhãn, sau đó cộng phiếu cho từng lớp. Với soft voting, lấy các vector xác suất, căn chỉnh lớp, nhân trọng số và chuẩn hóa tổng trọng số rồi chọn lớp lớn nhất.

VotingClassifier.fit trong sklearn fit các bản clone của estimator được khai báo, không mặc định tái sử dụng đúng những đối tượng đã fit bên ngoài [S2]. Mã minh họa dùng các base pipeline chứa vectorizer riêng; khi cần kiểm tra mô hình đã fit, truy cập estimators_ hoặc named_estimators_.

## 6. Quy trình áp dụng trong nghiên cứu

1. Quy định nhãn, đơn vị phân tích và xử lý review hỗn hợp/trung tính.
2. Loại trùng, chia train/development/test theo nhóm/thời gian phù hợp.
3. Xây từng base pipeline, fit mọi biến đổi trong train của từng lượt đánh giá.
4. Đánh giá base models trên development và xem kiểu lỗi bổ sung.
5. Chọn hard/soft, danh sách thành viên, trọng số và calibration nếu cần bằng development/CV.
6. Khóa cả quy tắc tổng hợp và threshold, rồi đánh giá test.
7. Báo so sánh với từng base, Voting đồng đều và phương án đã chọn.

Chọn trọng số theo test cũng là tuning bằng test dù không có lời gọi fit nào ở bộ tổng hợp. Việc lấy trung bình là thao tác đơn giản không làm quá trình lựa chọn hệ thống trở thành độc lập.

## 7. Ví dụ minh họa xuyên suốt

Với một review giả định, ba base models cho xác suất tích cực 0.51, 0.51, 0.01. Ở quy tắc nhị phân ngưỡng 0.5, hai mô hình đầu chọn tích cực, mô hình cuối chọn tiêu cực. Hard voting đồng đều chọn tích cực với 2 phiếu trên 3.

Soft voting đồng đều lấy (0.51+0.51+0.01)/3≈0.3433, nên chọn tiêu cực. Đây không phải lỗi tính toán. Hai quy tắc sử dụng mức thông tin khác nhau. Chưa biết nhãn thật của review nên cũng chưa thể kết luận quy tắc nào đúng từ riêng ba con số này.

## 8. Kết quả và cách diễn giải

Hard voting đưa nhãn theo tổng phiếu; tỷ lệ phiếu có thể mô tả mức đồng thuận nhưng không tự là xác suất được hiệu chỉnh. Trong sklearn, predict_proba được cung cấp cho soft voting, không suy rằng hard voting cũng có cùng đầu ra API.

Soft voting trả trung bình xác suất theo trọng số, nhưng trung bình không bảo đảm calibration ngoài mẫu. Cần kiểm tra mapping nhãn, hiệu năng theo lớp và chất lượng xác suất nếu dùng cho quyết định. Mức đồng thuận cao giữa các base có thể phản ánh cùng một thiên lệch.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là quy tắc tổng hợp rõ, dễ tạo đối chứng, không cần huấn luyện meta-model nếu trọng số đã cố định. Có thể kết hợp các hệ thống dùng biểu diễn khác nhau và quan sát tác động của việc thêm/bớt thành viên.

Nhược điểm là tăng tài nguyên, dễ làm giảm chất lượng khi thêm mô hình yếu hoặc lỗi tương quan, và soft voting nhạy với xác suất lệch. Một mô hình tốt có thể bị nhiều bản sao của một mô hình khác áp đảo. Tính đơn giản của công thức không bảo đảm cả hệ thống dễ diễn giải.

## 10. So sánh với phương pháp gần nhau

| Phương pháp/khái niệm | Phân biệt |
|---|---|
| Hard voting | Cộng phiếu nhãn, bỏ qua độ lớn xác suất bên trong base |
| Soft voting | Trung bình xác suất theo trọng số rồi chọn lớp |
| Stacking | Học meta-model từ đầu ra ngoài mẫu/OOF |
| Bagging | Cách tạo nhiều bộ học qua resampling; có thể dùng tổng hợp dạng voting/probability averaging |
| Boosting | Tạo ensemble tuần tự theo trạng thái/lỗi hiện tại |
| Bayesian model averaging | Có cấu trúc suy luận/trọng số theo mô hình xác suất; không đồng nhất với trung bình tùy ý |

Nếu trọng số Voting được tìm bằng tối ưu trên dữ liệu, hãy mô tả quy trình học/chọn đó. Tên Voting không miễn yêu cầu kiểm soát overfit ở bước lựa chọn trọng số.

## 11. Sai lầm thường gặp và cách khắc phục

Không cộng margin của SVM với xác suất Naive Bayes rồi gọi đó là soft voting. Các đại lượng khác thang và không cùng ngữ nghĩa. Nếu cần đưa classifier không có xác suất vào soft voting, phải có phương án xác suất/calibration hợp lệ trong train.

Không trộn mapping nhãn: cột 0 của hệ thống này có thể là tích cực còn của hệ thống khác là tiêu cực. Không dùng accuracy của test làm trọng số. Không cho nhiều bản sao gần giống nhau nhiều phiếu rồi diễn giải như nhiều nguồn bằng chứng độc lập.

## 12. Tài liệu tham khảo

- [S1] Dietterich, T. G. (2000). *Ensemble Methods in Machine Learning*. Multiple Classifier Systems. [Bài từ trang tác giả](https://web.engr.oregonstate.edu/~tgd/publications/mcs-ensembles.pdf).
- [S2] scikit-learn. [VotingClassifier, phiên bản 1.8](https://scikit-learn.org/1.8/modules/generated/sklearn.ensemble.VotingClassifier.html).
- [S3] scikit-learn. [Ensemble methods, phiên bản 1.8](https://scikit-learn.org/1.8/modules/ensemble.html).
- [S4] scikit-learn. [Probability calibration, phiên bản 1.8](https://scikit-learn.org/1.8/modules/calibration.html).

[S1] cung cấp bối cảnh ensemble, không được xem là năm phát minh duy nhất của bỏ phiếu mô hình. [S2–S4] dùng đối chiếu API và xác suất. Ví dụ, công thức thao tác và protocol khách sạn trong bài do người soạn tạo.

## 13. Vị trí trong toàn bộ nghiên cứu

Voting thuộc giai đoạn kết hợp mô hình sau khi đã xác định nhãn, biểu diễn và các ứng viên cơ sở. Danh sách thành viên và trọng số là một phần hệ thống nghiên cứu, không phải quyết định có thể đổi tùy test mà không báo.

Nếu dùng dự đoán để tổng hợp chỉ số cảm xúc khách sạn, cần kiểm tra sai số phân loại, calibration khi liên quan và đại diện mẫu. Đồng thuận giữa mô hình không thay thế độ nhất quán giữa người gán nhãn hay xác thực thang đo.

## 14. Kiến thức nền cần biết trước

Nên hiểu classifier, predict/predict_proba, mapping lớp, trung bình có trọng số và argmax. Cần phân biệt xác suất, log-probability, logit và decision margin; không lấy trung bình các loại này rồi coi là cùng một phương pháp.

Với nhiều mô hình, cần biết clone/fit, pipeline và dữ liệu development. Nếu muốn hiệu chỉnh xác suất, cần hiểu calibration phải dùng dữ liệu/OOF phù hợp, không calibrate trực tiếp trên test cuối. Các base pipelines vẫn cần tránh leakage tiền xử lý.

## 15. Nền tảng lý thuyết và logic hình thành

Động cơ của ensemble là kết hợp các dự đoán có thông tin bổ sung để giảm một số lỗi lựa chọn/ước lượng. Hard voting giữ quyết định lớp; soft voting giữ thêm mức phân bố xác suất. Cả hai vẫn phụ thuộc chất lượng các thành viên và sự phụ thuộc giữa lỗi.

Không thể dùng lập luận “đa số luôn đúng” cho mô hình học từ cùng dữ liệu. Các lỗi có thể tương quan mạnh do chung feature, nhãn nhiễu hoặc lệch nguồn. Đa dạng chỉ ở tên thuật toán không đủ; cần kiểm tra trên dữ liệu ngoài mẫu và đối chiếu với base tốt nhất.

## 16. Các thành phần cấu tạo

Thành phần gồm các base pipelines, tập nhãn chung, cách lấy đầu ra, vector trọng số, bộ căn chỉnh lớp, hàm tổng hợp và quy tắc xử lý hòa. Có thể có calibration ở từng base nếu thiết kế cần.

Phân biệt trọng số mô hình với trọng số lớp và trọng số review. Trọng số mô hình tăng phần đóng góp của một thành viên; không tự làm tăng recall lớp tiêu cực. Class imbalance cần được phân tích ở dữ liệu, base training, metric và quy tắc quyết định.

## 17. Các loại và biến thể

Có hard/soft voting đồng đều, có trọng số và các cơ chế tổng hợp khác như chọn mô hình theo điều kiện. Không gọi mọi phương pháp conditional gating hay meta-learning là VotingClassifier đơn giản nếu có thành phần học thêm.

Trong bài, trọng số cố định không âm và tổng dương để soft voting là convex combination của phân bố xác suất. Nếu dùng trọng số âm trong một công thức khác, đầu ra có thể không còn là xác suất hợp lệ; không áp nguyên diễn giải xác suất của bài.

## 18. Đầu vào và đầu ra

Đầu vào huấn luyện gồm X/y và cấu hình các estimator. Một số base có thể dùng word features, số khác char features, miễn pipeline nhận đúng dữ liệu và xuất cùng bộ nhãn. Dữ liệu deploy phải thỏa schema của mọi base được sử dụng.

Đầu ra hard là nhãn; soft có nhãn và phân bố trung bình. Khi dùng mô hình ngoài sklearn, phải quản lý thứ tự lớp theo tên, không chỉ số cột. Nếu một base thiếu một lớp do dữ liệu fit, cần xử lý hoặc từ chối cấu hình thay vì âm thầm coi cột khác là lớp bị thiếu.

## 19. Căn cứ lựa chọn

Đáng thử khi có vài mô hình chất lượng, lỗi bổ sung và nhu cầu baseline kết hợp minh bạch. Bắt đầu bằng Voting đồng đều để có đối chứng trước khi tìm nhiều trọng số hoặc thêm thành viên.

Chọn cuối bằng metric ngoài mẫu phù hợp nhiệm vụ, kèm chi phí. Nếu mục tiêu phát hiện khiếu nại, kiểm tra precision/recall tiêu cực và mức cảnh báo sai, không chỉ điểm trung bình. Khi chênh lệch không ổn định, một base tốt và rẻ hơn có thể hợp lý hơn ensemble.

## 20. Trường hợp không nên sử dụng

Không ưu tiên khi các thành viên gần như giống nhau, một thành viên tốt bị các mô hình yếu kéo xuống hoặc hệ thống không đủ tài nguyên duy trì nhiều pipeline. Soft voting không phù hợp nếu chưa có xác suất đáng tin và không có thiết kế kiểm tra/hiệu chỉnh.

Nếu vấn đề là nhãn sai, trùng review hoặc mục tiêu phân loại không phản ánh câu hỗn hợp, thêm phiếu không sửa căn nguyên. Nếu cần giải thích ngắn gọn từng quyết định, phải đánh giá độ phức tạp toàn hệ thống chứ không chỉ công thức bỏ phiếu.

## 21. Các quyết định quan trọng khi thực hiện

Chốt thành viên, biểu diễn, hard/soft, trọng số, calibration và quy tắc hòa. Chọn bằng development/CV, ghi số cấu hình đã thử. Trọng số theo hiệu năng cần có cơ sở và được đánh giá như hyperparameter, không lấy một số accuracy rồi mặc định đó là tối ưu.

Trong hard voting nhị phân, số mô hình lẻ giảm một kiểu hòa nếu phiếu đồng đều, nhưng không loại mọi hòa khi đa lớp/trọng số. Trong sklearn 1.8, hòa hard vote chọn lớp có mã nhỏ hơn sau LabelEncoder; soft argmax cũng có quy tắc theo thứ tự cột. Ghi mapping và đừng diễn giải cơ chế hòa như ưu tiên có cơ sở nghiên cứu.

## 22. Giải thích công thức và ký hiệu

Với M mô hình, nhãn $h_m(x)$ và trọng số $a_m\ge0$, $\sum_ma_m>0$:

$$\widehat y_{hard}=\operatorname*{argmax}_{c}\sum_{m=1}^{M}a_m\mathbf{1}[h_m(x)=c].$$

Với xác suất $p_m(c\mid x)$ được căn chỉnh theo lớp:

$$\overline p(c\mid x)=\frac{\sum_ma_mp_m(c\mid x)}{\sum_ma_m},\qquad \widehat y_{soft}=\operatorname*{argmax}_c\overline p(c\mid x).$$

Ví dụ $p_m(1\mid x)=(0.51,0.51,0.01)$, phiếu đồng đều: hard chọn 1, soft có $\overline p(1)=1.03/3\approx0.3433$ nên chọn 0. Nếu trọng số $(2,2,1)$ thì soft có $(1.02+1.02+0.01)/5=0.41$, vẫn chọn 0.

Lấy trung bình các logit rồi sigmoid tạo một quy tắc khác với trung bình các xác suất. Không thay hai cách cho nhau khi trình bày phương pháp. Với nhiều lớp, chọn argmax toàn bộ vector, không dùng một ngưỡng nhị phân chung cho mọi lớp.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mã kiểm tra phản ví dụ số và fit VotingClassifier với ba base pipelines thật. Dữ liệu tự tạo rất nhỏ; trọng số đặt trước để minh họa. Không có tối ưu trọng số bằng các câu kiểm tra.

```python
import numpy as np
from sklearn.pipeline import make_pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.naive_bayes import MultinomialNB
from sklearn.ensemble import VotingClassifier

positive = np.array([0.51, 0.51, 0.01])
hard_label = int((positive > 0.5).sum() > len(positive)/2)
soft_label = int(positive.mean() > 0.5)
assert hard_label == 1 and soft_label == 0
assert np.isclose(positive.mean(), 1.03/3)
assert np.isclose(np.average(positive, weights=[2, 2, 1]), 0.41)

train = ["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
         "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
         "dịch vụ tệ bẩn", "khách sạn bẩn tệ"]
y = np.array([1, 1, 1, 1, 0, 0, 0, 0])
test = ["phòng sạch zzholdout", "dịch vụ tốt", "phòng bẩn", "dịch vụ tệ"]
word_lr = make_pipeline(
    TfidfVectorizer(ngram_range=(1, 2)),
    LogisticRegression(C=1.0, max_iter=1000, random_state=42))
word_nb = make_pipeline(TfidfVectorizer(ngram_range=(1, 2)),
                        MultinomialNB(alpha=1.0))
char_nb = make_pipeline(TfidfVectorizer(analyzer="char", ngram_range=(2, 4)),
                        MultinomialNB(alpha=1.0))
weights = np.array([2., 1., 1.])
voter = VotingClassifier(
    estimators=[("word_lr", word_lr), ("word_nb", word_nb), ("char_nb", char_nb)],
    voting="soft", weights=weights, n_jobs=1)
voter.fit(train, y)
prob = voter.predict_proba(test)
assert "zzholdout" not in voter.named_estimators_["word_lr"][0].vocabulary_
assert np.isfinite(prob).all() and np.allclose(prob.sum(axis=1), 1)
base_prob = np.stack([model.predict_proba(test) for model in voter.estimators_])
# VotingClassifier fit base clones trên nhãn đã mã hóa cùng thứ tự.
assert all(np.array_equal(model.classes_, [0, 1]) for model in voter.estimators_)
manual = np.average(base_prob, axis=0, weights=weights)
assert np.allclose(prob, manual)
print("Ví dụ số: hard =", hard_label, "; soft =", soft_label)
print("Nhãn từ soft VotingClassifier:", voter.predict(test))
print("Trung bình thủ công khớp API:", np.allclose(prob, manual))
```

Ví dụ chỉ xác nhận quy tắc tính và đường chạy API, chưa chứng minh các base được calibration tốt hoặc ensemble hơn từng base. Cần development riêng và test phù hợp thiết kế để đánh giá những câu hỏi đó.

## 24. Tình huống sai và phản ví dụ

Phản ví dụ: một mô hình tốt bị kết hợp với hai bản sao của một mô hình kém. Hard voting đồng đều có thể để hai bản sao áp đảo, dù chúng không mang hai nguồn thông tin độc lập.

Phản ví dụ khác là base rất tự tin sai với p(positive)=0.01, trong khi hai base tốt hơn hơi nghiêng positive. Soft voting có thể bị kéo sang negative như mục 7. Chưa biết nhãn thật thì không được mặc định đây là lỗi của soft; cần kiểm tra calibration và hiệu năng theo loại câu ngoài mẫu.

## 25. Kiểm tra chất lượng và độ vững của kết quả

So ensemble với từng base và baseline đơn giản trên cùng split; báo metric theo lớp và độ biến động hợp lý. Làm ablation bỏ từng thành viên trên development để kiểm tra thông tin bổ sung, không chỉ nhìn accuracy riêng.

Với soft voting, kiểm tra calibration và proper scoring rule; với hard voting, ghi trường hợp hòa và mức đồng thuận. Phân tích lỗi tương quan theo phủ định, câu hỗn hợp, khách sạn/thời gian. Nếu cần khoảng bất định, dùng đơn vị lấy mẫu phù hợp thay vì coi các mô hình là những quan sát thống kê độc lập.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu Voting kém hơn base tốt nhất, xem các thành viên yếu hoặc lỗi quá giống nhau. Có thể giảm thành viên, đổi hard/soft, hiệu chỉnh xác suất hoặc điều chỉnh trọng số bằng development với ngân sách minh bạch. Chấp nhận dùng base đơn khi dữ liệu không ủng hộ ensemble.

Nếu thiếu predict_proba, chọn hard voting hoặc xây phương án calibration đúng phạm vi train, không gắn margin vào cột xác suất. Nếu lỗi đến từ biểu diễn/nhãn, sửa căn nguyên rồi đánh giá lại toàn bộ. Không tìm một bộ trọng số khớp test rồi giữ nguyên nhãn “đánh giá độc lập”.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ là Voting đạt metric nào và thay đổi thế nào so với các base trên thiết kế giữ lại. Có thể kết luận quy tắc tổng hợp hữu ích trong phạm vi đó nếu bằng chứng và độ ổn định đủ hỗ trợ.

Không suy “ba mô hình đồng ý” thành ba bằng chứng độc lập hoặc xác suất đúng cao theo một công thức chung. Không suy xác suất cảm xúc thành mức hài lòng được đo chuẩn hóa, và không suy feature hay thuật toán có trọng số cao gây kết quả quản trị.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu kết hợp các base pipelines [...] bằng [hard/soft] voting. Trọng số [...] được [đặt trước/chọn trên development theo protocol]. Các mô hình dùng cùng định nghĩa nhãn và được căn chỉnh thứ tự lớp. Test giữ lại theo [...] chỉ dùng sau khi khóa hệ thống.”

Bổ sung calibration nếu có, quy tắc hòa, threshold, phiên bản, tài nguyên và toàn bộ ngân sách chọn thành viên/trọng số. Bảng kết quả cần từng base, Voting đồng đều và cấu hình cuối để người đọc thấy ích lợi của việc kết hợp.

## 29. Khả năng tái lập và tính minh bạch

Lưu danh sách/thứ tự estimator, pipeline từng base, mapping nhãn, trọng số, chế độ voting, calibration, quy tắc hòa, split ID và phiên bản. Chỉ lưu trọng số mà không giữ các model/pipeline không đủ tái tạo.

Khi thay hoặc cập nhật một thành viên, cần kiểm tra lại schema và calibration của tổng thể. Mã mục 23 đã chạy trong sklearn 1.8.0 trên dữ liệu tự tạo; thực nghiệm thật cần dữ liệu được phép chia sẻ và script tái tạo. Ghi rõ các thành viên đã fit nằm trong estimators_, không nhầm với đối tượng khai báo ban đầu.

## 30. Đạo đức nghiên cứu liên quan

Các base dùng cùng review có thể chia sẻ thiên lệch theo ngôn ngữ, nền tảng hoặc nhóm được lấy mẫu. Voting không tự tạo tính công bằng bằng số phiếu. Cần kiểm tra sai số theo bối cảnh có dữ liệu hợp lệ và ghi giới hạn đại diện.

Bảo vệ thông tin nhận diện trong dữ liệu và các bản sao pipeline. Không dùng đồng thuận mô hình để bác bỏ khiếu nại hay phạt nhân viên mà không có người xem xét. Các ví dụ bài này tự tạo, không chứa thông tin khách hoặc nhân viên thật.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Hard voting giữ thông tin nào và bỏ thông tin nào so với soft?
2. Với 0.51,0.51,0.01, tính nhãn hard và soft đồng đều.
3. Vì sao không cộng margin SVM với xác suất NB?
4. Vì sao ba bản sao cùng mô hình không phải ba bằng chứng độc lập?
5. Thiết kế lựa chọn trọng số mà không nhìn test.

Gợi ý: hard chọn 1, soft chọn 0; dùng development hoặc CV trong phần phát triển. Bài mở rộng: so Voting đồng đều và trọng số trên cùng split; báo base tốt nhất và chi phí. Giải thích bằng chứng từ kết quả thật, không dùng số giả định ở mục 7 để khẳng định quy tắc nào tốt hơn.

## 32. Những điểm còn tranh luận

Các vấn đề cần kiểm chứng là mức lợi ích của tính đa dạng, calibration của xác suất tổng hợp và khả năng giữ chất lượng khi chuyển miền tiếng Việt. Không có quy tắc hard hay soft luôn thắng trong mọi dữ liệu.

Một đánh đổi khác là chọn trọng số linh hoạt với nguy cơ overfit development và tăng chi phí kiểm toán. Nghiên cứu cần báo đầy đủ quá trình chọn thành viên/trọng số và đối chứng base đơn, thay vì chỉ công bố cấu hình ensemble tốt nhất sau nhiều lượt thử.

