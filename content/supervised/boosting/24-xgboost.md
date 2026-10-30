# XGBoost — Boosting cây với tối ưu bậc hai và điều chuẩn

> **Nhánh:** MACHINE LEARNING → Supervised: Boosting. **Nguồn:** dòng 25 bảng Excel. **Đối chiếu:** 28/10/2026. Tập trung tree booster cho phân loại cảm xúc. Ví dụ số mô phỏng một bước tính gain; không phải triển khai đầy đủ hay benchmark của gói XGBoost. Dữ liệu và tình huống khách sạn trong ví dụ đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bản chất | Thư viện/hệ thống gradient boosting; bài tập trung cây |
| Mốc tham khảo | Chen & Guestrin, KDD 2016 [S1] |
| Đầu vào | Đặc trưng số, nhãn, cấu hình objective và cây |
| Đầu ra | Margin, xác suất/nhãn tùy objective và hàm dự đoán |
| Đặc điểm cần hiểu | Gradient, Hessian, regularization, gain, sparse/missing |
| Phần mềm | xgboost; ví dụ cơ chế chỉ cần NumPy |

## 1. Định nghĩa học thuật

XGBoost là một hệ thống học máy triển khai gradient boosting với nhiều lựa chọn mô hình và tối ưu tính toán. Với tree booster, nó xây dần các cây và tối ưu mục tiêu kết hợp loss với phạt độ phức tạp [S1].

Bài này tập trung cách xấp xỉ loss bậc hai để tính trọng số lá và lợi ích của split. XGBoost cũng có booster khác; không định nghĩa toàn bộ thư viện chỉ bằng một cấu hình cây hoặc một thuật toán tìm split.

## 2. Giải thích đơn giản

Hình dung mỗi cây đề xuất các điều chỉnh điểm cảm xúc. XGBoost hỏi: điều chỉnh này giảm sai số bao nhiêu, và có đáng tạo thêm nhánh/lá không? Một nhánh chia nhóm rất nhỏ có thể bị hạn chế vì phạt độ phức tạp và các ràng buộc.

Tốc độ và hiệu năng không đến từ một “mẹo” duy nhất. Chúng phụ thuộc biểu diễn dữ liệu, thuật toán tìm split, phần cứng và cấu hình. Không có bảo đảm nó thắng mọi mô hình chỉ vì tên gọi có chữ extreme.

## 3. Mục đích và vấn đề giải quyết

Dùng làm ứng viên phân loại đánh giá khách sạn khi muốn mô hình hóa tương tác giữa chỉ báo cảm xúc và biến có cấu trúc. Nó cũng là đối chứng thực nghiệm với GBDT thông thường, mô hình tuyến tính và các thư viện boosting khác.

Một câu hỏi rõ là liệu các chỉ báo phủ định, cường độ và độ dài khi kết hợp phi tuyến có cải thiện macro-F1 trên miền mới. Cần tách hiệu quả của feature engineering khỏi lợi ích của riêng XGBoost.

## 4. Điều kiện và giả định sử dụng

Cần xác định objective đúng nhãn, cung cấp đặc trưng hợp lệ và thiết kế chia dữ liệu phù hợp. Công thức bậc hai trong bài giả định loss trơn và Hessian hữu ích cho tối ưu; không áp dụng nguyên xi cho mọi custom objective.

Văn bản cần vector hóa hoặc biểu diễn thích hợp. Missing value, giá trị 0 và phần tử vắng trong sparse matrix có thể có ngữ nghĩa khác trong đường xử lý; cần kiểm tra hành vi phiên bản/định dạng thay vì chuyển dense sang sparse mà mặc định hoàn toàn tương đương [S2].

## 5. Cơ chế hoạt động bên trong

Ở mỗi vòng, tính gradient g và Hessian h của loss theo margin hiện tại trên train. Các ứng viên split gom tổng G và H vào các lá; tiêu chuẩn gain ước lượng mức giảm mục tiêu sau điều chuẩn. Các ràng buộc quyết định split có được chấp nhận không.

Sau khi chọn cấu trúc, tính giá trị lá và cộng cây vào margin với learning rate. Các triển khai exact/approx/hist tổ chức tìm ngưỡng khác nhau. Dự đoán nhị phân với objective logistic chuyển margin qua sigmoid; margin không phải xác suất.

## 6. Quy trình áp dụng trong nghiên cứu

1. Định nghĩa nhãn và loại biến được dùng tại dự đoán.
2. Loại trùng, chia dữ liệu theo khách sạn/thời gian và khóa test.
3. Fit biểu diễn trong train; thống nhất cách biểu diễn missing/zero.
4. Chọn objective, metric theo dõi và ngân sách tìm cấu hình.
5. Tìm learning rate, số vòng, depth, min_child_weight, regularization và sampling bằng development/CV.
6. Nếu early stopping, cung cấp development đã biến đổi bởi bộ tiền xử lý fit trên train.
7. Đánh giá test sau khi khóa cả số vòng/threshold và báo độ bất định.

Trong giao diện sklearn, eval_set không tự được biến đổi bởi vectorizer chỉ vì estimator nằm cuối một pipeline thông thường. Phải bảo đảm đúng luồng dữ liệu và đối chiếu API phiên bản [S3].

## 7. Ví dụ minh họa xuyên suốt

Dùng bốn phản hồi giả định: hai tích cực, hai tiêu cực. Một feature đã trích xuất tách hai nhóm; ở margin 0, tất cả có p=0.5. Vì vậy hai mẫu tích cực có g=−0.5 và hai mẫu tiêu cực g=+0.5.

Với lambda=1 và gamma=0, nhánh tích cực có giá trị lá +2/3, nhánh tiêu cực −2/3. Đây là tình huống thiết kế để kiểm tra phép tính, không chứng minh feature này xuất hiện trong dữ liệu thật hay tạo split hoàn hảo ngoài mẫu.

## 8. Kết quả và cách diễn giải

Đọc xác suất theo mapping nhãn và objective. Khi bật weighting lớp, chất lượng xác suất cần được kiểm tra riêng; tăng recall sau đổi trọng số không có nghĩa xác suất đã phản ánh tỷ lệ ngoài thực tế.

Feature importance có nhiều định nghĩa như gain, số lần dùng hoặc cover. Luôn nêu loại đang dùng. Các thứ hạng này mô tả hoạt động mô hình trên feature, không phải hệ số quan hệ hay tác động quản trị.

## 9. Điểm mạnh, điểm yếu và giới hạn

Điểm mạnh là nhiều công cụ điều chuẩn, hỗ trợ các chiến lược tìm split và có thể khai thác dữ liệu sparse. Các tùy chọn sampling và giới hạn cây giúp điều chỉnh đánh đổi độ phức tạp với chi phí.

Điểm yếu là không gian cấu hình lớn, nguy cơ overfit và shortcut, cùng chi phí diễn giải một ensemble. Khả năng chạy nhanh hoặc xử lý dữ liệu lớn phải được đo trong môi trường cụ thể. Nó không tự hiểu mỉa mai, miền tiếng Việt hoặc ý định của người viết.

## 10. So sánh với phương pháp gần nhau

| So sánh | Điểm cần phân biệt |
|---|---|
| GBDT tổng quát | XGBoost là hệ thống triển khai với objective/regularization và kỹ thuật cụ thể |
| LightGBM | Cả hai có thể dùng histogram; không xem hist là độc quyền của một thư viện |
| CatBoost | Có cơ chế và hỗ trợ xử lý categorical riêng; cần so trên cùng đầu vào/mục tiêu |
| Random Forest | Không xây cùng chuỗi cập nhật gradient theo cách tree boosting |
| Logistic Regression | Cần ít tương tác thiết kế hơn để có baseline văn bản sparse hữu ích |

Không khẳng định thuật toán bậc hai là đặc quyền duy nhất của XGBoost; nhiều triển khai khác cũng sử dụng thông tin độ cong.

## 11. Sai lầm thường gặp và cách khắc phục

Đừng hiểu min_child_weight là số mẫu tối thiểu trong lá: với tree booster, nó liên quan tổng Hessian theo objective. Chẳng hạn h=p(1−p) của logistic thay đổi theo dự đoán, nên cùng số mẫu có thể có tổng H khác nhau.

Không dùng tập test trong eval_set để chọn best iteration. Đừng báo gain feature là tỷ lệ đóng góp nhân quả. Nếu đổi sparse/dense, hãy có kiểm tra ngữ nghĩa missing/zero trên vài dòng biết trước.

## 12. Tài liệu tham khảo

- [S1] Chen, T., & Guestrin, C. (2016). *XGBoost: A Scalable Tree Boosting System*. KDD. [Bài gốc](https://arxiv.org/abs/1603.02754), [DOI](https://doi.org/10.1145/2939672.2939785).
- [S2] XGBoost. [Introduction to Boosted Trees](https://xgboost.readthedocs.io/en/stable/tutorials/model.html) và [FAQ](https://xgboost.readthedocs.io/en/stable/faq.html).
- [S3] XGBoost. [Scikit-Learn Estimator Interface](https://xgboost.readthedocs.io/en/stable/python/sklearn_estimator.html).
- [S4] XGBoost. [Parameters](https://xgboost.readthedocs.io/en/stable/parameter.html).

Nguồn thuật toán và hướng dẫn phần mềm cần đọc riêng. Tài liệu stable thay đổi theo thời gian; khi áp dụng phải ghi phiên bản cài đặt thật.

## 13. Vị trí trong toàn bộ nghiên cứu

XGBoost nằm trong phần xây và so sánh mô hình có giám sát. Khâu gán nhãn, lấy mẫu và chọn đơn vị nghiên cứu phải có trước; phần kiểm tra độ hợp lệ và suy rộng đi sau.

Nếu dùng dự đoán để tạo thang đo cảm xúc tổng hợp, cần xác nhận độ tin cậy của phép đo đó độc lập với việc chọn một classifier mạnh. Bài toán dự đoán review và bài toán kiểm định lý thuyết quản trị có yêu cầu bằng chứng khác nhau.

## 14. Kiến thức nền cần biết trước

Nền tảng cần có: cây hồi quy, additive model, log loss, sigmoid, đạo hàm bậc một/bậc hai và điều chuẩn L2. Cũng cần hiểu split theo nhóm và sparse matrix trong xử lý văn bản.

Cách học tốt là tự tính g, h, G, H cho bốn mẫu ở mục 7 rồi tính gain. Khi đọc mã, xác định đang dùng raw margin hay xác suất; lấy đạo hàm với sai đại lượng sẽ làm toàn bộ trọng số lá sai.

## 15. Nền tảng lý thuyết và logic hình thành

Xấp xỉ Taylor bậc hai biến thay đổi loss khi thêm cây thành biểu thức theo gradient, Hessian và giá trị lá. Cộng một hình phạt cho số lá và độ lớn giá trị lá tạo tiêu chí so sánh cấu trúc dễ tính hơn.

Đây là tối ưu từng bước với tìm kiếm cấu trúc tham lam, không chứng minh đạt tối ưu toàn cục. Chất lượng xấp xỉ, loss, ràng buộc và learning rate đều ảnh hưởng kết quả; Hessian không phải thước đo độ chắc chắn về nhãn của người gán.

## 16. Các thành phần cấu tạo

Thành phần gồm objective, booster, phương pháp tìm split, gradient/Hessian, hàm phạt, quy tắc sampling, learning rate, số vòng và cơ chế dừng. Pipeline còn có bộ mã hóa/biểu diễn và mapping nhãn.

Với cây, cần phân biệt cấu trúc split với giá trị lá. Hai cấu hình có cùng max_depth vẫn có thể học số lá và giá trị khác nhau vì ràng buộc Hessian, gain tối thiểu hoặc sampling.

## 17. Các loại và biến thể

Có tree booster, DART và linear booster; bài này không chuyển công thức gain lá sang linear booster. Với tree booster, exact, approximate và histogram là các lựa chọn tổ chức tìm split tùy hỗ trợ phiên bản.

Nhị phân thường dùng binary:logistic, còn đa lớp có objective và số lớp tương ứng. CPU/GPU, chế độ categorical và custom objective cần kiểm tra theo phiên bản, không suy từ tên phương pháp hoặc một ví dụ trên mạng.

## 18. Đầu vào và đầu ra

X_train có n hàng, d cột; y tương ứng từng hàng. Development/test phải áp dụng đúng bộ biến đổi và thứ tự cột. Có thể có sample weight, nhưng phải giải thích chúng là trọng số thiết kế, chi phí hay cân bằng lớp.

Đầu ra phụ thuộc objective và lời gọi: raw margin khác predict_proba; nhãn cần quy tắc. Hãy lưu schema đầu vào cùng model để một cột độ dài không vô tình bị thay bằng cột số từ chê ở triển khai.

## 19. Căn cứ lựa chọn

Đáng thử khi có nhãn đáng tin, đặc trưng có cấu trúc hoặc kết hợp, và đủ ngân sách lựa chọn cấu hình. Có thể chọn nếu cải thiện ngoài mẫu ổn định, chi phí chấp nhận được và lỗi quan trọng được kiểm soát.

Căn cứ không phải số lần thuật toán được dùng trong bài báo khác. So sánh cần cùng split, cùng quyền truy cập feature và ngân sách tuning minh bạch. Nếu lợi ích chỉ xuất hiện khi test bị dùng để chọn vòng, kết luận không đáng tin.

## 20. Trường hợp không nên sử dụng

Không nên ưu tiên khi mục tiêu chỉ cần quy tắc cực ngắn, dữ liệu/nhãn quá ít hoặc biến đầu vào không có tại thời điểm dự đoán. Không dùng khả năng xử lý missing để bỏ qua phân tích cơ chế thiếu dữ liệu.

Trong văn bản sparse rất nhiều chiều, một baseline tuyến tính có thể rẻ hơn đáng kể. Nếu câu hỏi yêu cầu hiểu khía cạnh, ý định hay suy luận ngữ cảnh sâu, đổi classifier trên biểu diễn nghèo có thể không giải quyết nút thắt.

## 21. Các quyết định quan trọng khi thực hiện

Các quyết định chính: objective; mapping nhãn; tree_method; learning rate/số vòng; độ sâu hoặc số lá theo grow policy; min_child_weight; lambda/alpha/gamma; subsample và colsample. Không cần tìm mọi tham số cùng lúc: đặt không gian có lý do và lưu cấu hình đã thử.

Với mất cân bằng, chọn weighting và metric theo mục tiêu, rồi đánh giá lại calibration/threshold trên development. Ghi cách chọn best iteration khi early stopping và kiểm tra cách predict sử dụng nó trong API đã cài.

## 22. Giải thích công thức và ký hiệu

Với loss theo raw margin $F$, đặt $g_i=\partial L/\partial F_i$, $h_i=\partial^2L/\partial F_i^2$. Trường hợp logistic: $g_i=p_i-y_i$, $h_i=p_i(1-p_i)$.

Với một lá $j$, $G_j=\sum_{i\in j}g_i$, $H_j=\sum_{i\in j}h_i$. Chọn hình phạt $\Omega=\gamma T+\frac{\lambda}{2}\sum_jw_j^2$, không có L1 trong ví dụ:

$$w_j^*=-\frac{G_j}{H_j+\lambda}.$$

$$Gain=\frac12\left[\frac{G_L^2}{H_L+\lambda}+\frac{G_R^2}{H_R+\lambda}-\frac{(G_L+G_R)^2}{H_L+H_R+\lambda}\right]-\gamma.$$

$T$ là số lá, $w_j$ là cập nhật ở lá trước shrinkage. Công thức này dùng một quy ước objective cụ thể; thêm L1 hoặc ràng buộc khác sẽ thay phép tính. Cần $H_j+\lambda>0$.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mã sau chạy bằng NumPy, kiểm tra công thức một split. Nó không gọi gói xgboost và không mô phỏng toàn bộ quá trình tìm cây, sampling hay xử lý missing.

```python
import numpy as np

y = np.array([1., 1., 0., 0.])
p = np.full(4, 0.5)
g, h = p - y, p * (1 - p)
lam, gamma, eta = 1., 0., 0.1
GL, HL = g[:2].sum(), h[:2].sum()
GR, HR = g[2:].sum(), h[2:].sum()
wL, wR = -GL / (HL + lam), -GR / (HR + lam)
gain = 0.5 * (GL**2 / (HL + lam) + GR**2 / (HR + lam)
              - (GL + GR)**2 / (HL + HR + lam)) - gamma
assert np.allclose([wL, wR, gain], [2/3, -2/3, 2/3])
new_margin = eta * np.array([wL, wL, wR, wR])
new_p = 1 / (1 + np.exp(-new_margin))
assert np.all(new_p[:2] > 0.5) and np.all(new_p[2:] < 0.5)
print("Giá trị lá:", wL, wR, "gain:", gain)
print("Xác suất sau cập nhật:", new_p.round(6))
```

Ta được xác suất khoảng 0.516660 ở nhánh trái, 0.483340 ở nhánh phải. Gain 2/3 là ước lượng giảm objective bậc hai trước shrinkage theo quy ước trên, không phải macro-F1 tăng 2/3 hay giảm log loss thực tế đúng 2/3.

## 24. Tình huống sai và phản ví dụ

Giả sử tên nền tảng gần như xác định nhãn vì cách lấy mẫu: mô hình học nền tảng thay cảm xúc và đạt test ngẫu nhiên cao. Khi đánh giá trên nền tảng mới, nó có thể thất bại dù feature importance có vẻ rõ ràng.

Một phản ví dụ kỹ thuật: đặt min_child_weight=10 và nghĩ mỗi lá chỉ cần 10 câu. Với logistic ở p=0.5, mỗi mẫu có h=0.25, nên tổng H của 10 mẫu chỉ là 2.5. Ràng buộc đó không tương đương đếm mẫu.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Kiểm tra đường learning curve, confusion matrix, recall lớp hiếm và hiệu năng theo khách sạn/thời gian. Theo dõi sự ổn định khi thay seed và split; không chỉ seed một lần rồi kết luận ổn định.

Với importance, so sánh nhiều định nghĩa và đánh giá permutation trên tập giữ lại phù hợp, chú ý feature tương quan có thể chia sẻ tín hiệu. Với deployment, kiểm tra schema, missing, từ mới và calibration; test phải phản ánh dạng thay đổi dự kiến.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu overfit, kiểm tra leakage rồi tăng regularization, giảm độ sâu, điều chỉnh sampling hoặc dừng sớm trên development. Nếu không học được, kiểm tra objective/nhãn, tổng Hessian và ràng buộc có khiến cây không thể split không.

Nếu chi phí không hợp lý, giảm không gian feature hoặc thử baseline tuyến tính. Khi phát hiện xử lý zero/missing không đúng, sửa schema và chạy lại toàn bộ lựa chọn mô hình; không chỉ sửa test để điểm số đẹp hơn.

## 27. Phân biệt kết quả với kết luận

Kết quả mô tả hiệu năng và hành vi trên thiết kế đánh giá cụ thể. Kết luận hợp lệ có điều kiện: mô hình hữu ích trong miền và cơ chế lấy mẫu đã khảo sát, kèm sai số và giới hạn suy rộng.

Không viết “XGBoost chứng minh độ sạch là yếu tố quyết định sự hài lòng” chỉ vì feature độ sạch có gain cao. Nó có thể dự đoán nhãn được gán từ cùng câu chữ; điều đó khác với bằng chứng can thiệp hoặc cơ chế nhân quả.

## 28. Cách viết vào bài nghiên cứu

Mẫu phương pháp: “Nghiên cứu huấn luyện XGBoost tree booster với [objective], [tree_method] và [schema]. Siêu tham số được chọn trên [development/CV theo nhóm]. Early stopping, nếu có, chỉ theo dõi development. Số vòng và threshold được khóa trước đánh giá test.”

Bổ sung phiên bản, phần cứng, số luồng, ngân sách tuning, weighting và metric. Khi trình bày importance, ghi đúng loại gain/weight/cover, không gọi chung là mức ảnh hưởng của biến.

## 29. Khả năng tái lập và tính minh bạch

Lưu model, pipeline/vectorizer, tên và thứ tự feature, nhãn, seed, phiên bản, split ID và toàn bộ tham số. Ghi cả tree_method, thiết bị và số luồng vì lựa chọn triển khai ảnh hưởng thời gian và có thể ảnh hưởng tái lập số học.

Ví dụ mục 23 chỉ cần NumPy và kiểm tra toán; chưa xác nhận chạy huấn luyện bằng thư viện xgboost trong môi trường này. Khi bổ sung thực nghiệm, hãy lưu script và môi trường để người khác phân biệt minh họa với kết quả đo thật.

## 30. Đạo đức nghiên cứu liên quan

Cần bảo vệ danh tính người đánh giá và nhân viên, không công khai văn bản có thể truy ngược khi chưa có cơ sở phù hợp. Các ví dụ trong bài là tự tạo để tránh mang thông tin cá nhân vào tài liệu hướng dẫn.

Kiểm tra sai số giữa các nhóm ngôn ngữ/bối cảnh và có người rà soát quyết định có tác động. Gain hoặc giải thích cục bộ không cấp quyền kết luận động cơ khách hàng hay chất lượng cá nhân nhân viên.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Tính h của logistic khi p=0.5 và khi p=0.99.
2. Vì sao min_child_weight không bằng số mẫu tối thiểu?
3. Từ bốn mẫu ở mục 7, tính giá trị lá khi lambda tăng từ 1 lên 2.
4. Thiết kế eval_set không làm rò rỉ vocabulary.

Gợi ý: h lần lượt 0.25 và 0.0099; lá trái khi lambda=2 là 0.4. Development phải được transform bằng bộ fit trên train, không fit riêng vocabulary hoặc dùng test để dừng sớm.

## 32. Những điểm còn tranh luận

Các câu hỏi còn cần kiểm chứng gồm lợi ích thực tế so với baseline trong văn bản tiếng Việt, độ ổn định của giải thích và cách phân bổ ngân sách tuning công bằng. Không nên lấy thứ hạng một bộ dữ liệu làm chân lý cho mọi miền.

Đánh đổi tốc độ, bộ nhớ, độ chính xác và tính tái lập phụ thuộc cấu hình/phần cứng. Một so sánh có ích nên ghi rõ toàn bộ protocol và phạm vi, thay vì khẳng định một thư viện luôn nhanh hoặc luôn chính xác nhất.

