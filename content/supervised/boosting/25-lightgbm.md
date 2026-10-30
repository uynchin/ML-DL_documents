# LightGBM — Boosting theo histogram và phát triển cây theo lá

> **Nhánh:** MACHINE LEARNING → Supervised: Boosting. **Nguồn:** dòng 26 bảng Excel. **Đối chiếu:** 28/10/2026. Tập trung GBDT với histogram, leaf-wise, GOSS và EFB. Mã NumPy kiểm tra cơ chế đơn giản hóa; chưa chạy huấn luyện bằng gói lightgbm. Dữ liệu và tình huống khách sạn trong ví dụ đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bản chất | Hệ thống gradient boosting với các tối ưu học cây |
| Mốc tham khảo | Ke và cộng sự, NeurIPS 2017 [S1] |
| Đầu vào | Đặc trưng đã biểu diễn, nhãn và cấu hình cây |
| Đầu ra | Raw score, xác suất hoặc nhãn theo objective |
| Điểm phân biệt | Histogram, leaf-wise; GOSS là lựa chọn cấu hình, không luôn bật |
| Phần mềm | lightgbm; ví dụ số chỉ dùng NumPy và thư viện chuẩn |

## 1. Định nghĩa học thuật

LightGBM là một hệ thống gradient boosting có các kỹ thuật tổ chức dữ liệu và học cây nhằm giảm chi phí tính toán. Bài gốc giới thiệu GOSS và EFB như hai kỹ thuật tăng hiệu quả trong bối cảnh nghiên cứu của tác giả [S1].

Cần phân biệt thư viện hiện hành với cấu hình được nghiên cứu ở bài gốc. Trong bài này, GBDT là mô hình chính; không khẳng định mỗi lần chạy LightGBM đều dùng tất cả kỹ thuật, cùng thiết bị hoặc cùng chiến lược lấy mẫu.

## 2. Giải thích đơn giản

Thay vì thử mọi giá trị số riêng lẻ, mô hình gom giá trị feature vào các khoảng gọi là bins và cộng thống kê trong từng khoảng. Khi phát triển cây, nó ưu tiên lá có cơ hội giảm loss nhiều nhất theo tiêu chí và ràng buộc.

Ví như phân loại đánh giá khách sạn thành các nhóm rồi chọn nhóm đáng chia tiếp nhất. Một nhánh có thể sâu hơn nhánh khác; vì vậy cần giới hạn để mô hình không bám quá sát các nhóm ít mẫu.

## 3. Mục đích và vấn đề giải quyết

Có thể dùng để phân loại cảm xúc từ các chỉ báo văn bản kết hợp dữ liệu cấu trúc, hoặc làm đối chứng khi chi phí huấn luyện là mối quan tâm. Câu hỏi nghiên cứu nên bao gồm cả hiệu quả ngoài mẫu và tài nguyên sử dụng.

Với review khách sạn, cần kiểm tra liệu mô hình có giữ chất lượng khi chuyển sang khách sạn/thời gian khác. Thuật toán nhanh trên một bảng đặc trưng không tự bảo đảm giải quyết được phủ định, mỉa mai hoặc thay đổi cách viết tiếng Việt.

## 4. Điều kiện và giả định sử dụng

Cần nhãn rõ ràng, feature phù hợp, đủ mẫu cho các lá và một protocol đánh giá độc lập. Histogram làm xấp xỉ các ứng viên split; số bins là lựa chọn ảnh hưởng độ chi tiết và chi phí, không phải bước hoàn toàn không mất thông tin.

Nếu dùng categorical, khai báo và mã hóa theo yêu cầu thư viện, không coi mã nguyên là thứ bậc tự nhiên. Với missing/zero, ghi rõ cấu hình. Mặc định zero_as_missing trong tài liệu hiện hành là false; không biến mọi giá trị 0 thành thiếu dữ liệu theo suy đoán.

## 5. Cơ chế hoạt động bên trong

Với histogram, thuật toán cộng gradient/Hessian theo bin, từ đó đánh giá ngưỡng giữa các bins. Chiến lược leaf-wise chọn lá có gain tốt nhất để chia tiếp trong các ứng viên hợp lệ, thay vì bắt buộc hoàn tất mọi nút ở cùng tầng.

GOSS giữ các mẫu có độ lớn gradient cao và lấy mẫu nhóm gradient thấp với hiệu chỉnh trọng số. EFB khai thác đặc trưng thưa có tính loại trừ để gộp biểu diễn trong các điều kiện kiểm soát xung đột. Chúng không đồng nghĩa gộp mọi feature tương quan hoặc loại bỏ tùy ý các câu khó.

## 6. Quy trình áp dụng trong nghiên cứu

1. Chốt nhãn, đơn vị mẫu và mục tiêu suy rộng.
2. Loại trùng và tách train/development/test theo nhóm/thời gian thích hợp.
3. Fit biểu diễn và các bước xử lý trong train từng fold.
4. Đặt objective, metric, num_leaves, max_depth, min_data_in_leaf và cấu hình histogram.
5. Tìm learning rate/số vòng bằng development; early stopping chỉ dùng development.
6. Nếu khảo sát GOSS/EFB, ghi chính xác cấu hình và làm so sánh có kiểm soát.
7. Báo điểm số, độ bất định, thời gian, bộ nhớ và lỗi theo nhóm.

Dữ liệu validation phải được transform bằng cùng bộ tiền xử lý đã fit trên train. Không lấy test để xây vocabulary, chọn bins bên ngoài mô hình hay chọn best iteration.

## 7. Ví dụ minh họa xuyên suốt

Bốn review giả định được biểu diễn bằng một chỉ báo cường độ khen x=[0.1,0.2,0.8,0.9], nhãn y=[0,0,1,1]. Một biên bin 0.5 tách hai nhóm. Ở p=0.5, gradient hai nhóm có dấu ngược nhau.

Ví dụ cho thấy cách cộng thống kê theo bins, không mô phỏng tiếng Việt từ văn bản thô. Trong nghiên cứu thật, cách xây chỉ báo phải được công bố, áp dụng nhất quán và không dùng nhãn test để quyết định biểu diễn.

## 8. Kết quả và cách diễn giải

Đầu ra phải đọc theo objective và lớp. Raw score của nhị phân được chuyển sang xác suất bằng hàm liên kết tương ứng; probability không phải mức hài lòng trên thang đo tâm lý đã được xác thực.

Báo macro-F1 và metric theo lớp nếu mục tiêu bao gồm lớp ít mẫu. Nếu so tốc độ, tính rõ phần vector hóa, tuning, training và prediction nào nằm trong phép đo. Thời gian huấn luyện một cấu hình không đại diện toàn bộ chi phí chọn mô hình.

## 9. Điểm mạnh, điểm yếu và giới hạn

Histogram và các tối ưu dữ liệu có thể giảm chi phí ở bài toán phù hợp. Leaf-wise phân bổ khả năng chia cây linh hoạt; thư viện có nhiều ràng buộc để kiểm soát số lá, độ sâu và kích thước vùng.

Hạn chế là lá nhỏ và nhánh sâu có thể overfit, bins có thể bỏ qua ngưỡng hữu ích, và cấu hình sai làm mô hình không chia được cây. Không bảo đảm nhanh nhất trên mọi cỡ dữ liệu; văn bản sparse cần so với tuyến tính thay vì chỉ so các hệ thống boosting.

## 10. So sánh với phương pháp gần nhau

| Khái niệm/phương pháp | Phân biệt |
|---|---|
| Level-wise | Mở rộng theo tầng; leaf-wise ưu tiên gain ở từng lá |
| XGBoost hist | Cũng có histogram; so sánh cần đúng cấu hình hiện hành |
| GOSS | Chiến lược lấy mẫu dựa độ lớn gradient, không phải tên toàn bộ LightGBM |
| EFB | Gộp feature theo tính loại trừ/thưa, khác chọn feature theo tương quan |
| CatBoost | Có cơ chế xử lý categorical/ordered riêng |

Các thư viện đã phát triển qua thời gian; không lặp lại so sánh mặc định cũ như đặc điểm vĩnh viễn của đối thủ.

## 11. Sai lầm thường gặp và cách khắc phục

Không viết “LightGBM mặc định luôn dùng GOSS”. Tài liệu hiện hành có data_sample_strategy mặc định bagging; GOSS là lựa chọn rõ ràng, và bagging chỉ thực sự lấy mẫu khi cấu hình điều kiện liên quan [S3].

Một lỗi khác là tăng num_leaves mà không xem min_data_in_leaf và độ sâu. Đặt max_depth chỉ giới hạn đường đi, không thay thế mọi kiểm soát số lá. Trên dữ liệu đồ chơi rất ít mẫu, ngưỡng lá mặc định có thể khiến cây không chia, khác với lỗi thư viện.

## 12. Tài liệu tham khảo

- [S1] Ke, G., và cộng sự (2017). *LightGBM: A Highly Efficient Gradient Boosting Decision Tree*. NeurIPS. [Bài gốc](https://proceedings.neurips.cc/paper_files/paper/2017/hash/6449f44a102fde848669bdd9eb6b76fa-Abstract.html).
- [S2] LightGBM. [Features](https://lightgbm.readthedocs.io/en/stable/Features.html).
- [S3] LightGBM. [Parameters](https://lightgbm.readthedocs.io/en/stable/Parameters.html).
- [S4] LightGBM. [Parameters Tuning](https://lightgbm.readthedocs.io/en/stable/Parameters-Tuning.html).

Bài gốc mô tả đóng góp nghiên cứu; tài liệu hiện hành xác định cấu hình phần mềm. Không dùng kết quả benchmark trong bài gốc làm điểm số dự kiến cho bộ review của mình.

## 13. Vị trí trong toàn bộ nghiên cứu

LightGBM là ứng viên ở khâu học dự đoán, sau thiết kế nghiên cứu, thu thập, gán nhãn và xây đặc trưng. Nếu nghiên cứu so hệ thống, việc đo tài nguyên và kiểm soát ngân sách tuning là một phần protocol.

Đầu ra dự đoán có thể được tổng hợp theo khách sạn, nhưng cần xem sai số đo lường và cơ chế chọn review. Một mẫu đánh giá trực tuyến không mặc nhiên đại diện toàn bộ khách lưu trú.

## 14. Kiến thức nền cần biết trước

Cần hiểu gradient/Hessian, cây hồi quy, histogram, sparse feature và regularization. Nên biết vì sao nhiều lá làm tăng khả năng biểu diễn, còn kích thước mẫu nhỏ trong lá làm ước lượng dễ biến động.

Để hiểu GOSS, cần thêm xác suất chọn mẫu và trọng số nghịch xác suất. Việc nhân trọng số cho nhóm được lấy mẫu nhằm sửa thiên lệch tổng thống kê theo thiết kế, không biến một tập mẫu ngẫu nhiên thành toàn bộ dữ liệu giống hệt trong mỗi lượt.

## 15. Nền tảng lý thuyết và logic hình thành

Chi phí tìm split có thể giảm khi dùng thống kê gộp thay vì xét mọi giá trị riêng. Leaf-wise tập trung bước chia tại nơi có lợi ích cục bộ cao; GOSS và EFB tiếp cận chi phí từ chiều mẫu và chiều feature.

Đó là các đánh đổi thuật toán: phép xấp xỉ, chọn mẫu và gộp biểu diễn cần điều kiện. Chúng không làm mất yêu cầu kiểm tra overfit hoặc bảo đảm tối ưu toàn cục. Thống kê được ước lượng tốt không đồng nghĩa cấu trúc cây cuối chắc chắn giống khi dùng toàn dữ liệu.

## 16. Các thành phần cấu tạo

Các bộ phận gồm bộ lượng hóa feature thành bins, thống kê gradient/Hessian, bộ tìm split, hàng đợi ứng viên lá, giá trị lá, regularization, sampling, learning rate và điều kiện dừng. Pipeline ngoài thư viện xử lý văn bản và schema.

Phân biệt max_bin với num_leaves: một tham số kiểm soát độ chi tiết xét giá trị feature, tham số kia kiểm soát số vùng cuối của cây. Thay một tham số không thay thế trực tiếp vai trò của tham số còn lại.

## 17. Các loại và biến thể

Thư viện có các chế độ như gbdt, dart và rf theo tài liệu. Trong GBDT, có thể thay chiến lược lấy mẫu và tùy chọn feature bundling. Bài này không coi mọi chế độ là cùng một thuật toán với các công thức hoàn toàn giống nhau.

Hỗ trợ CPU/GPU và categorical có điều kiện triển khai. Với đa lớp, một vòng có thể tạo nhiều cây tùy cách triển khai; không đồng nhất n_estimators với tổng số cây thực tế trong mọi bài toán.

## 18. Đầu vào và đầu ra

Đầu vào gồm X có schema ổn định, y đúng mục tiêu và có thể có weights/categorical declarations. Tại dự đoán, phải giữ đúng mapping category và không tự ý mã hóa lại theo thứ tự xuất hiện trong test.

Đầu ra có score/xác suất/nhãn và thông tin mô hình. Importance theo split count khác importance theo gain. Cần ghi loại, đơn vị và dữ liệu sử dụng khi đưa biểu đồ vào bài nghiên cứu.

## 19. Căn cứ lựa chọn

Nên thử khi feature có cấu trúc hoặc kết hợp, dữ liệu đủ cho phân chia phi tuyến, và có nhu cầu đo hiệu quả tính toán. Lựa chọn cuối dựa cả chất lượng ngoài mẫu và các lỗi tác động đến mục tiêu ứng dụng.

Nếu ưu tiên phát hiện phản hồi tiêu cực, macro-F1 tổng thể chưa đủ: cần recall/precision lớp tiêu cực và chi phí cảnh báo sai. Mức cải thiện cần xem độ ổn định theo nhóm, không chỉ một split ngẫu nhiên thuận lợi.

## 20. Trường hợp không nên sử dụng

Không nên ưu tiên khi dữ liệu quá nhỏ mà không có thiết kế kiểm soát overfit, khi chỉ muốn một quy tắc rất ngắn, hoặc khi thiếu nhãn đáng tin. Một cơ chế tính nhanh không thể thay thông tin cần cho học có giám sát.

Nếu đánh giá toàn văn có nhiều khía cạnh trái chiều, ép nhãn nhị phân có thể sai mục tiêu. Khi đó cần sửa định nghĩa nhiệm vụ hoặc gán nhãn trước, thay vì chỉ đổi thông số LightGBM.

## 21. Các quyết định quan trọng khi thực hiện

Chọn num_leaves cùng max_depth, min_data_in_leaf/min_child_samples, ràng buộc Hessian và learning rate/số vòng. Với giới hạn độ sâu d của cây nhị phân, số lá không vượt 2^d; đây chỉ là giới hạn cấu trúc, không phải công thức chọn num_leaves tối ưu.

Ghi max_bin, xử lý missing/zero, strategy lấy mẫu và có bật EFB không. Không dùng GOSS chỉ vì tên phương pháp; cần so với cấu hình chuẩn theo cùng ngân sách và kiểm tra tác động lên chất lượng.

## 22. Giải thích công thức và ký hiệu

Tại bin b, gom $G_b=\sum_{i\in b}g_i$, $H_b=\sum_{i\in b}h_i$. Từ tổng tích lũy theo bin có thể tính thống kê bên trái/phải và gain của ngưỡng theo objective đã chọn. Minh họa dùng gain có L2 như bài XGBoost, không khẳng định mọi chi tiết triển khai giống nhau.

Với GOSS đơn giản hóa, giữ nhóm A gồm aN mẫu gradient lớn, lấy ngẫu nhiên nhóm B gồm bN mẫu trong (1−a)N mẫu còn lại. Trọng số cho mẫu thấp được chọn:

$$c=\frac{1-a}{b},\qquad \widehat G=\sum_{i\in A}g_i+c\sum_{i\in B}g_i.$$

Ở đây b là tỷ lệ trên **toàn bộ N**, không phải trên nhóm còn lại. Nếu q là tỷ lệ chọn trong nhóm còn lại, hệ số là 1/q. Tổng được hiệu chỉnh đúng theo kỳ vọng với giả thiết chọn đều; một lượt chọn không nhất thiết bằng tổng thật.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mã dưới kiểm tra hai ý: gom histogram và trọng số lấy mẫu. Đây là mô hình số giản lược, không gọi lightgbm, không kiểm tra tốc độ hoặc thực thi đầy đủ GOSS/EFB.

```python
import numpy as np
from itertools import combinations

x = np.array([0.1, 0.2, 0.8, 0.9])
y = np.array([0., 0., 1., 1.])
p = np.full(4, 0.5)
g, h = p-y, p*(1-p)
bins = np.searchsorted([0.5], x)
G = np.bincount(bins, weights=g, minlength=2)
H = np.bincount(bins, weights=h, minlength=2)
lam = 1.
gain = 0.5 * (np.sum(G**2/(H+lam)) - G.sum()**2/(H.sum()+lam))
assert np.allclose(G, [1., -1.])
assert np.allclose(H, [0.5, 0.5])
assert np.isclose(gain, 2/3)

# 6 mẫu: giữ 2 gradient lớn; chọn 2 trong 4 gradient nhỏ.
high = np.array([2., -3.])
low = np.array([0.1, 0.2, 0.3, 0.4])
a, b = 2/6, 2/6
weight = (1-a)/b
estimates = [high.sum() + weight * low[list(pair)].sum()
             for pair in combinations(range(4), 2)]
assert np.isclose(weight, 2.)
assert np.isclose(np.mean(estimates), high.sum()+low.sum())
assert not np.allclose(estimates, high.sum()+low.sum())
print("Gain theo bins:", gain)
print("Các ước lượng tổng gradient:", np.round(estimates, 4))
```

Ví dụ liệt kê mọi cặp để kiểm tra kỳ vọng trong trường hợp nhỏ. Tính không chệch của một tổng không chứng minh gain phi tuyến hay cây được chọn cũng không chệch theo cùng nghĩa.

## 24. Tình huống sai và phản ví dụ

Phản ví dụ: hai feature thường cùng xuất hiện, như “phòng” và “sạch”, không thể được coi là loại trừ chỉ vì đều sparse. Gộp chúng bằng cộng số học tùy ý sẽ mất phân biệt và không tương đương EFB.

Phản ví dụ khác: một leaf có gain lớn trên ba review gần như trùng nhau. Chiến lược leaf-wise có thể muốn chia vùng này, nhưng lợi ích trên train không chứng minh quy tắc tổng quát; cần loại trùng và giới hạn lá.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đánh giá theo nhóm và thời gian, xem learning curve và phân bố kích thước lá. Thử độ nhạy hợp lý với num_leaves, min_data_in_leaf và max_bin bằng development, không thực hiện chuỗi thử đó trên test.

Với tuyên bố tiết kiệm tài nguyên, đo cùng phần cứng, luồng, dữ liệu và ngân sách tuning; báo cả chất lượng tương ứng. Một cấu hình nhanh vì không tạo được split không phải bằng chứng thuật toán đạt hiệu quả tốt.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu cây không chia, kiểm tra số mẫu, biến hằng, ràng buộc lá/Hessian và cảnh báo. Nếu overfit, giảm số lá/độ sâu, tăng yêu cầu ở lá hoặc điều chỉnh số vòng. Đừng tắt mọi ràng buộc chỉ để thấy cây lớn.

Nếu hiệu năng kém với GOSS hoặc EFB trong protocol của mình, so với cấu hình không dùng kỹ thuật đó trên development. Khi biểu diễn không nắm được phủ định, cần sửa feature hoặc nhiệm vụ; tuning cây không tự tạo thông tin bị mất.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ mô tả mức dự đoán và chi phí trên dữ liệu, split và cấu hình đã xác định. Có thể kết luận LightGBM phù hợp với một thiết lập cụ thể nếu bằng chứng chất lượng/tài nguyên đủ rõ.

Không suy ra “leaf-wise luôn tốt hơn level-wise” từ một lượt chạy, cũng không suy ra feature quan trọng gây cảm xúc. Tuyên bố về các kỹ thuật riêng cần ablation có kiểm soát; đổi cả biểu diễn và phần cứng không cô lập được nguyên nhân cải thiện.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu sử dụng LightGBM GBDT với [objective]. Số lá, độ sâu, kích thước lá và số vòng được chọn bằng [development/CV]. Cấu hình histogram, sampling và bundling được ghi đầy đủ; test giữ lại theo [đơn vị].”

Nếu so GOSS/EFB, mô tả điều gì được thay đổi và điều gì được giữ cố định. Báo phiên bản, thiết bị, số luồng, metric và cách tính thời gian. Chỉ đưa số đo thực tế; mục 23 không phải kết quả huấn luyện thư viện.

## 29. Khả năng tái lập và tính minh bạch

Lưu split ID, schema, mapping category, vectorizer, toàn bộ parameters, seed, phiên bản và môi trường phần cứng. Với benchmark tốc độ, ghi warm-up, số lần đo và phạm vi xử lý được tính vào thời gian.

Bài này đã kiểm tra mã số NumPy, không xác nhận API hoặc benchmark bằng package lightgbm. Khi tái lập huấn luyện thực tế, lưu script đầy đủ và cả cảnh báo thay vì chỉ file model cuối.

## 30. Đạo đức nghiên cứu liên quan

Tối ưu tốc độ không làm thay đổi nghĩa vụ bảo vệ dữ liệu review. Ẩn thông tin cá nhân, quản lý quyền truy cập và cân nhắc khả năng truy ngược từ câu trích dẫn. Không đưa ID người viết vào mô hình chỉ vì nó dự đoán tốt.

Kiểm tra lỗi theo nhóm ngôn ngữ/bối cảnh và ghi giới hạn đại diện. Nếu dùng để ưu tiên xử lý khiếu nại, cần người xem lại các trường hợp nhạy cảm; một score tự động không đủ xác định trách nhiệm nhân viên.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Histogram thay đổi tập ngưỡng split như thế nào?
2. Vì sao max_depth và num_leaves cần xem cùng nhau?
3. Với a=0.2, b=0.1 tính trọng số GOSS theo định nghĩa mục 22.
4. Vì sao gộp hai feature có tương quan không phải EFB?

Gợi ý: trọng số là 8; b được tính trên toàn tập. Bài mở rộng: thay biên bin 0.5 bằng 0.85 trong ví dụ số rồi tính lại tổng gradient; thảo luận thông tin phân chia đã thay đổi, không báo đó là benchmark.

## 32. Những điểm còn tranh luận

Mức lợi ích của GOSS/EFB và leaf-wise phụ thuộc phân bố feature, số mẫu và cấu hình. Cần kiểm tra khi dữ liệu ít, nhiều noise hoặc chuyển miền; benchmark gốc không thay thế đánh giá tại bài toán đích.

Một vấn đề khác là so sánh công bằng giữa hệ thống: cùng số vòng không có nghĩa cùng độ phức tạp hay chi phí. Nên mô tả ngân sách tìm kiếm và đường đánh đổi chất lượng/tài nguyên thay vì chỉ một bảng xếp hạng.

