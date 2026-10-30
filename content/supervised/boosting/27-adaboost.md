# AdaBoost — Boosting thích nghi bằng trọng số mẫu

> **Nhánh:** MACHINE LEARNING → Supervised: Boosting. **Nguồn:** dòng 28 bảng Excel. **Đối chiếu:** 28/10/2026. Phân biệt AdaBoost nhị phân kinh điển với SAMME đa lớp. Mã minh họa chạy với scikit-learn 1.8.0, dùng estimator và không truyền tham số algorithm đã bị loại khỏi API này. Dữ liệu và tình huống khách sạn trong ví dụ đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bản chất | Ensemble học tuần tự, tăng trọng số tương đối cho mẫu bị phân loại sai |
| Mốc tham khảo | Freund & Schapire: hội nghị 1995, bài tạp chí 1997 [S1] |
| Đầu vào | Đặc trưng, nhãn và estimator hỗ trợ sample_weight |
| Đầu ra | Tổng hợp có trọng số các bộ học cơ sở; nhãn/điểm/xác suất theo API |
| Ví dụ cơ sở | Decision stump: cây quyết định độ sâu 1 |
| Phần mềm | sklearn.ensemble.AdaBoostClassifier, scikit-learn 1.8.0 |

## 1. Định nghĩa học thuật

AdaBoost là một họ thuật toán boosting xây ensemble tuần tự bằng cách thay đổi trọng số phân phối trên mẫu huấn luyện. Trong dạng phân loại rời rạc, bộ học mới được huấn luyện trên phân phối hiện tại; sai số có trọng số quyết định mức đóng góp và cách cập nhật trọng số mẫu [S1].

Bài trình bày dạng nhị phân kinh điển và SAMME cho đa lớp. Chúng có quy ước hệ số khác nhau; phải nêu dạng đang dùng thay vì lấy công thức từ một dạng và so trực tiếp với thuộc tính thư viện của dạng khác.

## 2. Giải thích đơn giản

Hình dung nhiều người học lần lượt đọc review. Sau mỗi lượt, các câu bị phân loại sai được nhấn mạnh hơn trong lượt học tiếp theo. Kết quả cuối kết hợp các người học theo trọng số, không coi mọi người học quan trọng ngang nhau.

“Mẫu khó được chú ý” không có nghĩa mẫu đó có giá trị nghiên cứu cao hơn. Một câu gán nhãn sai, spam hoặc không thể phân loại nhất quán cũng có thể liên tục nhận trọng số lớn.

## 3. Mục đích và vấn đề giải quyết

AdaBoost có thể là ứng viên phân loại cảm xúc từ biểu diễn văn bản hoặc các chỉ báo đã trích xuất. Với decision stump, nó cho phép kết hợp nhiều quy tắc yếu thành một bộ phân loại phong phú hơn.

Trong nghiên cứu khách sạn, có thể đặt câu hỏi liệu tổ hợp stump có tăng chất lượng so với stump đơn, cây sâu hoặc mô hình tuyến tính. Cần dùng cùng dữ liệu/split và kiểm soát ngân sách tuning để câu trả lời có ý nghĩa.

## 4. Điều kiện và giả định sử dụng

Cần nhãn đủ tin cậy, dữ liệu được biểu diễn đúng và base estimator hỗ trợ sample_weight theo yêu cầu API. Điều kiện sai số tốt hơn ngưỡng ngẫu nhiên của dạng boosting đang xét phải được đáp ứng để có bước học hữu ích.

Không cần mọi feature độc lập. Tuy nhiên, dữ liệu gán nhãn nhiễu hoặc outlier có thể gây vấn đề vì trọng số tập trung vào mẫu sai. Dữ liệu thiếu và categorical phải được xử lý theo estimator/pipeline, không mặc định AdaBoost tự xử lý tất cả.

## 5. Cơ chế hoạt động bên trong

Khởi tạo trọng số mẫu, thường đồng đều. Fit base learner theo trọng số; tính tỷ lệ lỗi có trọng số. Nếu bộ học hoàn hảo hoặc không đạt điều kiện, thuật toán có quy tắc dừng/xử lý riêng. Nếu hợp lệ, tính trọng số của learner và tăng trọng số tương đối cho mẫu sai.

Chuẩn hóa trọng số để dùng ở vòng kế tiếp, rồi lặp. Ở dự đoán, tổng hợp các learner đã học bằng trọng số. Mẫu không bị loại hẳn chỉ vì đã dự đoán đúng; trọng số tương đối của chúng thường giảm sau cập nhật.

## 6. Quy trình áp dụng trong nghiên cứu

1. Xác định nhãn, xử lý lớp trung tính/hỗn hợp và mức nhất quán gán nhãn.
2. Loại trùng, tách train/development/test theo mục tiêu suy rộng.
3. Fit vectorizer và xử lý dữ liệu trong train từng fold.
4. Chọn base estimator hỗ trợ trọng số, chẳng hạn stump.
5. Chọn số vòng, learning rate và độ phức tạp base learner bằng development/CV.
6. Kiểm tra các mẫu nhiều lần bị sai, phân biệt nhãn sai với hiện tượng ngôn ngữ khó.
7. Khóa mô hình, đánh giá test theo lớp và phân tích độ bền.

Nếu sửa nhãn sau kiểm tra lỗi, cần quy trình rà soát độc lập, ghi lịch sử và không điều chỉnh test để hợp dự đoán. Không lấy test làm nơi chọn số vòng.

## 7. Ví dụ minh họa xuyên suốt

Giả định bốn review có trọng số đều 1/4. Stump đầu phân loại sai đúng một câu. Sai số có trọng số epsilon=0.25; sau bước AdaBoost nhị phân chuẩn với learning rate bằng 1, câu sai có trọng số mới 0.5 và mỗi câu đúng là 1/6.

Câu sai có thể là “phòng không sạch” trong khi biểu diễn chỉ có từ đơn. Cập nhật trọng số nhấn mạnh nó, nhưng nếu biểu diễn không phân biệt phủ định thì thêm vòng không bảo đảm học đúng. Cần kiểm tra feature và nhãn chứ không chỉ số vòng.

## 8. Kết quả và cách diễn giải

Đầu ra nhãn là tổng hợp có trọng số. Decision score phản ánh tổ hợp của các base learners theo quy ước API; không mặc định nó là xác suất hoặc dùng trực tiếp sigmoid của score như công thức phổ quát.

Nếu dùng predict_proba, cần đánh giá calibration riêng. Trọng số learner cao nói về đóng góp trong ensemble và sai số của bước học, không phải độ tin cậy tuyệt đối trên mọi mẫu tương lai. Báo metric theo lớp cùng dữ liệu đánh giá.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là cơ chế cập nhật rõ, có thể kết hợp learner đơn giản và có ít tham số hơn một số hệ thống boosting phức tạp khi dùng stump. Nó tạo đối chứng hữu ích để khảo sát giá trị của tổ hợp các quy tắc yếu.

Nhược điểm gồm nhạy với noise/outlier trong nhãn, học tuần tự và giới hạn của base learner/biểu diễn. Nhiều stump có thể không nắm được một số tương tác nếu biểu diễn không đủ; base learner quá mạnh lại có thể tăng overfit. Không có bảo đảm vượt baseline văn bản tuyến tính.

## 10. So sánh với phương pháp gần nhau

| Phương pháp | Khác biệt chính |
|---|---|
| Bagging/Random Forest | Ngẫu nhiên hóa và tổng hợp nhiều mô hình; không dùng cùng chuỗi cập nhật trọng số sai mẫu |
| Gradient Boosting | Mô tả hướng giảm loss bằng gradient; có liên hệ lý thuyết với boosting nhưng không cùng mọi cập nhật |
| Stump đơn | Chỉ một cây rất nông, chưa có tổ hợp tuần tự |
| SAMME | Mở rộng rời rạc cho đa lớp với hệ số phụ thuộc số lớp |
| SAMME.R | Biến thể dùng xác suất, không phải API đang minh họa |

Không dùng các tên SAMME và SAMME.R thay thế nhau hoặc đưa tham số cũ vào ví dụ của scikit-learn 1.8.

## 11. Sai lầm thường gặp và cách khắc phục

Lỗi thường gặp là xem epsilon như lỗi không trọng số sau vòng đầu. Khi D đã thay đổi, một mẫu sai có trọng số lớn có thể quan trọng hơn nhiều mẫu đúng nhẹ; phải dùng phân phối hiện tại.

Một lỗi khác là mong estimator_weights_ của SAMME nhị phân bằng hệ số một nửa log-ratio trong sách. SAMME dùng quy ước khác. Ngoài ra, không truyền base_estimator hoặc algorithm từ ví dụ cũ vào API 1.8; dùng estimator và kiểm tra tài liệu [S2].

## 12. Tài liệu tham khảo

- [S1] Freund, Y., & Schapire, R. E. (1997). *A Decision-Theoretic Generalization of On-Line Learning and an Application to Boosting*. Journal of Computer and System Sciences, 55(1), 119–139. [Bài tạp chí](https://www.sciencedirect.com/science/article/pii/S002200009791504X), [DOI](https://doi.org/10.1006/jcss.1997.1504). Bản hội nghị trước đó xuất hiện năm 1995.
- [S2] scikit-learn. [AdaBoostClassifier, phiên bản 1.8](https://scikit-learn.org/1.8/modules/generated/sklearn.ensemble.AdaBoostClassifier.html).
- [S3] Zhu, J., Zou, H., Rosset, S., & Hastie, T. (2009). *Multi-class AdaBoost*. Statistics and Its Interface, 2, 349–360. [Bài từ tác giả](https://dept.stat.lsa.umich.edu/~jizhu/pubs/Zhu-SII09.pdf).

[S1] cho nền tảng; [S3] cho SAMME; [S2] và mã thư viện cài đặt được dùng để kiểm tra API của ví dụ.

## 13. Vị trí trong toàn bộ nghiên cứu

AdaBoost là phương pháp học có giám sát trong chuỗi nghiên cứu sau xử lý và gán nhãn. Nó có thể đóng vai trò baseline hoặc một ứng viên trong so sánh phương pháp, không thay thế bước kiểm tra chất lượng nhãn.

Nếu dự đoán được dùng để ưu tiên xử lý phản hồi, cần đánh giá chi phí cảnh báo sai và bỏ sót. Nếu dùng để ước lượng tỷ lệ tiêu cực, cần kiểm tra sai số đo và độ lệch mẫu trước khi tổng hợp.

## 14. Kiến thức nền cần biết trước

Nên biết cây/stump, sample weight, lỗi phân loại, logarit và tổng hợp có trọng số. Cần phân biệt trọng số mẫu D_i với trọng số learner alpha_m; chúng không cùng đơn vị hay cùng ý nghĩa.

Thực hành cơ bản là làm một vòng với bốn mẫu, tính epsilon, alpha và phân phối mới. Khi chuyển sang đa lớp, kiểm tra ngưỡng lỗi và công thức SAMME thay vì giữ điều kiện epsilon<0.5 cho mọi K.

## 15. Nền tảng lý thuyết và logic hình thành

AdaBoost nhị phân có thể được nhìn qua việc giảm exponential loss của margin yF(x), với y và dự đoán cơ sở lấy giá trị −1/+1. Margin lớn cùng dấu nhãn làm loss nhỏ; mẫu có margin xấu nhận trọng số tương đối cao hơn.

Góc nhìn này giải thích liên hệ với additive modeling và gradient boosting, nhưng không làm mọi thuật toán boosting giống nhau. Exponential loss nhạy với mẫu rất sai; noise trong nhãn vì thế cần được kiểm tra bằng nghiên cứu độ bền.

## 16. Các thành phần cấu tạo

Các thành phần gồm phân phối trọng số mẫu, base estimator, bộ tính lỗi có trọng số, hệ số learner, cập nhật/chuẩn hóa trọng số, số vòng tối đa và quy tắc dừng. Pipeline còn có vectorizer và mapping nhãn.

Với stump, max_depth=1 giới hạn base learner; n_estimators giới hạn số vòng, không đảm bảo số learner cuối luôn đúng bằng giá trị yêu cầu vì có thể dừng sớm. Learning rate điều chỉnh đóng góp theo công thức biến thể.

## 17. Các loại và biến thể

Có AdaBoost nhị phân rời rạc, SAMME đa lớp và các biến thể khác trong tài liệu. AdaBoost cho hồi quy là bài toán/cập nhật khác; không áp dụng công thức lỗi nhị phân nguyên xi cho giá trị liên tục.

Ví dụ thư viện dùng AdaBoostClassifier của scikit-learn 1.8, triển khai SAMME rời rạc và không có tham số algorithm trong chữ ký. Nếu dùng phiên bản khác, ghi rõ thay đổi API và cơ chế thay vì âm thầm đổi biến thể.

## 18. Đầu vào và đầu ra

Đầu vào huấn luyện là X, nhãn y và có thể có sample_weight ban đầu. Nếu trọng số thể hiện thiết kế mẫu hoặc chi phí, cần nêu cách xây dựng, không đánh đồng với trọng số mà boosting tạo ở các vòng sau.

Đầu ra gồm nhãn, score/xác suất theo API, các learner đã học và thông tin lỗi/trọng số. Bộ vectorizer phải được lưu để transform dữ liệu mới cùng cột; các từ chưa gặp không được thêm vào vocabulary khi dự đoán.

## 19. Căn cứ lựa chọn

Đáng thử khi muốn đối chứng với learner đơn giản, có nhãn đáng tin và chấp nhận học tuần tự. Chọn cuối dựa vào hiệu quả ngoài mẫu và hành vi trên lỗi quan trọng, không phải mức hoàn hảo trên train.

Có thể khảo sát xem mô hình gặp khó do noise hay do biểu diễn bằng tập thách thức gán nhãn độc lập. Nếu chỉ nhìn trọng số cao rồi xóa mẫu, ta có thể vô tình xóa các cách diễn đạt hiếm nhưng hợp lệ.

## 20. Trường hợp không nên sử dụng

Không nên ưu tiên nếu nhãn nhiều lỗi chưa được rà soát hoặc base learner không hỗ trợ trọng số theo yêu cầu. Với dữ liệu cực ít, số vòng lớn có thể chỉ nhấn mạnh sự ngẫu nhiên của một vài mẫu.

Nếu cần giải thích chỉ bằng một quy tắc ngắn, ensemble nhiều stump vẫn có thể quá phức tạp. Nếu nhiệm vụ thực là phân tích cảm xúc từng khía cạnh, nhãn toàn văn nhị phân có thể sai mục đích dù classifier được huấn luyện đúng.

## 21. Các quyết định quan trọng khi thực hiện

Chốt base learner, độ sâu, số vòng, learning rate, trọng số ban đầu và metric lựa chọn. Kiểm tra điều kiện dừng và số learner thực tế. Với lớp hiếm, xem weighting ảnh hưởng cả lựa chọn learner và chất lượng xác suất như thế nào.

Nếu chọn threshold hoặc calibration, cần dữ liệu development phù hợp và quy trình tránh leakage. Không tự gọi số vòng là regularization duy nhất; độ phức tạp base learner và chất lượng biểu diễn cũng quan trọng.

## 22. Giải thích công thức và ký hiệu

Dạng nhị phân kinh điển, $y_i,h_m(x_i)\in\{-1,+1\}$, với $\sum_iD_i^{(m)}=1$:

$$\epsilon_m=\sum_iD_i^{(m)}\mathbf{1}[y_i\ne h_m(x_i)],\quad \alpha_m=\frac12\log\frac{1-\epsilon_m}{\epsilon_m},$$

$$D_i^{(m+1)}=\frac{D_i^{(m)}e^{-\alpha_my_ih_m(x_i)}}{Z_m},\quad H(x)=\operatorname{sign}\left(\sum_m\alpha_mh_m(x)\right).$$

Công thức hữu hạn trên xét $0<\epsilon_m<0.5$; $Z_m$ chuẩn hóa. Khi lỗi bằng 0 phải xử lý dừng, không tính log vô hạn.

SAMME với K lớp và learning rate $\eta$ dùng $\alpha_m^{S}=\eta[\log((1-\epsilon_m)/\epsilon_m)+\log(K-1)]$, tăng trọng số mẫu sai theo $e^{\alpha_m^{S}}$, với điều kiện lỗi dưới $1-1/K$. Khi K=2 và $\eta=1$, hệ số gấp đôi dạng nhị phân ở trên; phân phối cập nhật sau chuẩn hóa tương đương. Không so trực tiếp hệ số thô giữa hai quy ước.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Đầu tiên kiểm tra hai quy ước cho một mẫu sai trên bốn mẫu. Phần sau fit một pipeline thật bằng scikit-learn 1.8. Dữ liệu tự tạo chỉ kiểm tra thao tác, không ước lượng hiệu năng nghiên cứu.

```python
import numpy as np
from sklearn.pipeline import make_pipeline
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import AdaBoostClassifier

D = np.full(4, 0.25)
wrong = np.array([False, False, False, True])
epsilon = D[wrong].sum()
alpha = 0.5*np.log((1-epsilon)/epsilon)
yh = np.where(wrong, -1., 1.)
classic = D*np.exp(-alpha*yh)
classic /= classic.sum()
samme_alpha = np.log((1-epsilon)/epsilon)  # K=2, learning_rate=1
samme = D*np.exp(samme_alpha*wrong)
samme /= samme.sum()
assert np.allclose(classic, [1/6, 1/6, 1/6, 1/2])
assert np.allclose(classic, samme)

train = ["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
         "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
         "dịch vụ tệ bẩn", "khách sạn bẩn tệ"]
y = [1, 1, 1, 1, 0, 0, 0, 0]
test = ["phòng sạch zzholdout", "dịch vụ tốt", "phòng bẩn", "dịch vụ tệ"]
pipe = make_pipeline(
    CountVectorizer(ngram_range=(1, 2)),
    AdaBoostClassifier(
        estimator=DecisionTreeClassifier(max_depth=1, random_state=42),
        n_estimators=30, learning_rate=0.5, random_state=42))
pipe.fit(train, y)
prob = pipe.predict_proba(test)
assert "zzholdout" not in pipe[0].vocabulary_
assert np.isfinite(prob).all() and np.allclose(prob.sum(axis=1), 1)
assert 1 <= len(pipe[-1].estimators_) <= 30
print("Trọng số mẫu sau một vòng:", classic)
print("Nhãn dự đoán trên ví dụ:", pipe.predict(test))
```

Ví dụ không truyền algorithm hay base_estimator. Với bộ dữ liệu thật, bổ sung development/CV theo thiết kế trước khi chọn số vòng và learning rate.

## 24. Tình huống sai và phản ví dụ

Phản ví dụ: một review tiêu cực bị gán nhãn tích cực do lỗi nhập liệu. Nhiều learner dự đoán tiêu cực theo nội dung, khiến mẫu đó bị tăng trọng số. Tiếp tục boosting có thể ép mô hình học lỗi nhãn chứ không cải thiện hiểu ngôn ngữ.

Ngược lại, một câu mỉa mai hợp lệ khó học không nên bị xóa chỉ vì có trọng số cao. Cần rà soát nhãn độc lập, có hướng dẫn và ghi lý do; “mô hình không đồng ý” không phải bằng chứng đủ để sửa nhãn.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Kiểm tra learning curve trên development, lỗi theo lớp, mẫu khó lặp lại và sự ổn định theo seed/split. Đo ảnh hưởng của noise bằng protocol được định trước nếu đó là câu hỏi nghiên cứu, không tự bịa mức cải thiện.

Với dữ liệu phụ thuộc theo khách sạn, dùng split và khoảng bất định phù hợp đơn vị. Kiểm tra calibration nếu dùng xác suất. Xem số learner thực học cùng cảnh báo, thay vì giả định đã hoàn thành mọi vòng yêu cầu.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu yếu hơn baseline, kiểm tra nhãn, biểu diễn và khả năng base learner đạt điều kiện trước. Có thể giảm độ phức tạp hoặc số vòng khi overfit, hoặc tăng khả năng biểu diễn một cách có kiểm soát khi underfit.

Nếu lỗi do phủ định không được mã hóa, thêm n-gram/chỉ báo phù hợp trong train và đánh giá lại. Nếu trọng số tập trung vào nhãn lỗi, rà soát theo quy trình; không bỏ mẫu test khó. Khi chuyển phương pháp, giữ protocol đánh giá công bằng.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ là hiệu năng theo lớp và độ ổn định trên thiết kế đã chọn. Có thể kết luận AdaBoost hữu ích hơn stump đơn trong phạm vi đó nếu so sánh và độ bất định hỗ trợ.

Không kết luận mẫu có trọng số lớn là nhóm khách quan trọng nhất, hay feature trong nhiều stump là nguyên nhân gây bất mãn. Trọng số phục vụ tối ưu thuật toán, khác trọng số khảo sát và khác tầm quan trọng quản trị.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu sử dụng AdaBoostClassifier [phiên bản] với SAMME và base estimator là cây độ sâu [...]. Learning rate và số vòng được chọn trên [development/CV]. Biểu diễn văn bản được fit trong train của từng fold; test giữ lại theo [đơn vị].”

Báo số learner thực học, metric theo lớp, quy trình rà soát nhãn và cấu hình weighting nếu có. Khi giải thích toán, ghi rõ đang dùng dạng nhị phân hay SAMME để tránh hệ số và nhãn ký hiệu không nhất quán.

## 29. Khả năng tái lập và tính minh bạch

Lưu pipeline, nhãn, split ID, seed, phiên bản sklearn và NumPy, tham số base learner, learning rate/số vòng cùng số learner thực học. Ghi cả sample weight ban đầu nếu dùng và cách tạo nó.

Mã mục 23 đã được chạy trong môi trường sklearn 1.8.0. Điều đó xác nhận ví dụ thao tác được, không bảo đảm hiệu quả trên dữ liệu người dùng. Kết quả nghiên cứu thật phải đi cùng code, môi trường và dữ liệu được phép chia sẻ.

## 30. Đạo đức nghiên cứu liên quan

Không dùng trọng số cao để gắn nhãn người viết là spam, khó tính hoặc không đáng tin. Đó chỉ là trạng thái thuật toán trên một biểu diễn và bộ nhãn cụ thể. Rà soát phải dựa nội dung và quy tắc độc lập.

Bảo vệ thông tin cá nhân trong review, tránh công khai câu có thể truy ngược, và có người xem xét trước quyết định bất lợi cho nhân viên. Kiểm tra sai số theo nhóm khi có cơ sở dữ liệu hợp lệ; ghi giới hạn đại diện.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Vì sao lỗi ở vòng sau phải tính có trọng số?
2. Với epsilon=0.25, tính alpha nhị phân và alpha SAMME khi K=2, eta=1.
3. Ngưỡng điều kiện lỗi của SAMME khi K=3 là bao nhiêu?
4. Vì sao một mẫu trọng số cao chưa đủ căn cứ sửa nhãn?

Gợi ý: alpha lần lượt khoảng 0.5493 và 1.0986; ngưỡng ba lớp là 2/3. Bài mở rộng: mô phỏng một nhãn sai rồi theo dõi phân phối trọng số, ghi rõ đây là thí nghiệm tổng hợp và không suy thành tỷ lệ noise thật.

## 32. Những điểm còn tranh luận

Cần đánh giá thực nghiệm mức nhạy với noise, cách chọn base learner và giá trị của ensemble so với baseline trên từng biểu diễn tiếng Việt. Các kết quả lý thuyết không xóa yêu cầu kiểm tra điều kiện thực tế.

Một vấn đề khác là dùng trọng số mẫu để tìm lỗi nhãn mà không loại bỏ các biểu đạt thiểu số hợp lệ. Thiết kế rà soát cần chuyên môn và tính minh bạch; tối ưu điểm số không đủ làm tiêu chuẩn duy nhất cho chất lượng dữ liệu.

