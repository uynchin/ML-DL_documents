# Decision Tree — Cây quyết định cho phân loại cảm xúc

> **Nhánh:** MACHINE LEARNING → Supervised: Tree. **Nguồn:** dòng 22 bảng Excel. **Đối chiếu:** 28/10/2026. Bài tập trung cây phân loại nhị phân kiểu CART, phân biệt với cây hồi quy và cây quyết định trong quản trị. Dữ liệu, số đếm và tình huống khách sạn tự soạn. Mã kiểm tra với scikit-learn 1.8.0.

| Trường | Nội dung |
|---|---|
| Bản chất | Mô hình có giám sát chia không gian đặc trưng thành các vùng bằng quy tắc phân nhánh |
| Mốc tham khảo | CART: Breiman, Friedman, Olshen & Stone, 1984, được tài liệu thư viện dẫn [S1]; không coi 1984 là năm xuất hiện mọi loại cây |
| Đầu vào | Đặc trưng số đã xử lý, nhãn và cấu hình phát triển/cắt tỉa |
| Đầu ra | Nhãn, phân bố lớp tại lá, đường đi và độ quan trọng đặc trưng theo tiêu chuẩn đã chọn |
| Tiếng Việt | Cần biểu diễn như count/TF-IDF hoặc chỉ báo trích xuất; không tự hiểu văn bản thô |
| Nhãn/tài nguyên | Cần nhãn; cây nhỏ thường chạy CPU, kích thước phụ thuộc dữ liệu/độ sâu/feature |
| Phần mềm | `sklearn.tree.DecisionTreeClassifier`; bài dùng tài liệu phiên bản 1.8 phù hợp môi trường kiểm tra |

## 1. Định nghĩa học thuật

Decision Tree học một tập quy tắc phân chia dữ liệu để dự đoán mục tiêu. Ở cây phân loại kiểu CART, một nút thường kiểm tra một feature và ngưỡng, đưa mẫu sang hai nhánh; các lá chứa thông tin lớp để dự đoán [S1–S2].

Đây là mô hình phi tham số theo nghĩa không cố định một dạng hàm với số tham số cố định trước mọi cỡ dữ liệu. Nó vẫn có nhiều tham số học và siêu tham số cần kiểm soát; “phi tham số” không có nghĩa không cần cấu hình.

## 2. Giải thích đơn giản

Hình dung một chuỗi câu hỏi: đánh giá có một biểu thức chê không, có phủ định không, điểm feature vượt ngưỡng nào? Câu trả lời dẫn mẫu đến một lá, nơi mô hình đưa ra nhãn/phân bố lớp.

Những câu hỏi được học từ nhãn và đặc trưng, không phải hiểu biết quản trị được tự xác nhận. Một cây có thể hỏi tên khách sạn nếu tên đó giúp chia nhãn trong train, dù đó không phải tín hiệu cảm xúc đáng tin ở khách sạn mới.

## 3. Mục đích và vấn đề giải quyết

Đề xuất nghiên cứu khách sạn: xây baseline phân loại phản hồi hoặc mô hình dự đoán từ các chỉ báo văn bản và biến có cấu trúc. Cây nhỏ có thể giúp đọc các quy tắc mô hình sử dụng và phát hiện shortcut.

Nếu nghiên cứu quan tâm tương tác, cây có thể biểu diễn điều kiện kết hợp qua nhiều nhánh. Nhưng việc học được quy tắc “nếu A và B thì dự đoán âm” không chứng minh A/B gây ra trải nghiệm tiêu cực.

## 4. Điều kiện và giả định sử dụng

Cần nhãn rõ, đặc trưng có sẵn lúc dự đoán, dữ liệu chia đúng mục tiêu và đủ mẫu cho các vùng/lá. Cây không đòi chuẩn hóa thang như nhiều mô hình khoảng cách, nhưng tiền xử lý, mã hóa biến và chất lượng dữ liệu vẫn quan trọng.

Phiên bản `DecisionTreeClassifier` trong bài nhận đầu vào số; không coi chuỗi tên loại là dữ liệu categorical được xử lý tự động. Với missing value, khả năng hỗ trợ phụ thuộc estimator/cấu hình/phiên bản; không khẳng định mọi cây luôn hỗ trợ hoặc luôn không hỗ trợ missing [S1–S2].

## 5. Cơ chế hoạt động bên trong

Tại một nút, thuật toán xét các split được phép, tính độ giảm impurity và chọn phương án theo chiến lược. Nó lặp ở các nút con đến khi đạt điều kiện dừng. Có thể hạn chế trước khi cây lớn hoặc cắt tỉa sau bằng tiêu chuẩn phù hợp.

Đây thường là tìm kiếm tham lam cục bộ, không bảo đảm cây tối ưu toàn cục. Một split có lợi nhất ngay lúc đó có thể không dẫn đến cấu trúc tốt nhất cuối cùng. Dự đoán chỉ đi theo nhánh học được, không cần tìm lại split.

## 6. Quy trình áp dụng trong nghiên cứu

1. Quy định nhãn, đơn vị và xử lý phản hồi hỗn hợp/trung tính.
2. Loại trùng, chia train/development/test theo thời gian/khách sạn khi cần.
3. Fit biểu diễn trong pipeline trên train; không học vocabulary/chọn feature bằng test.
4. Xây baseline đa số và cây với mức phức tạp được quy định.
5. Chọn depth, leaf size, pruning và weighting qua development/CV.
6. Đọc cây, kiểm tra lá ít mẫu và feature có nguy cơ rò rỉ.
7. Đánh giá ngoài mẫu, xác suất và độ ổn định khi phù hợp.

Nếu dùng cây để tìm quy tắc nghiên cứu, nên xác nhận các quy tắc trên dữ liệu độc lập thay vì chỉ chọn quy tắc nhìn hợp lý trong train.

## 7. Ví dụ minh họa xuyên suốt

Đặt một feature nhị phân tự tạo $x$: có/không một tín hiệu chê đã nhận diện. Sáu mẫu giả định có nhãn:

| Giá trị feature | Tích cực | Tiêu cực | Tổng |
|---|---:|---:|---:|
| $x=0$ | 2 | 0 | 2 |
| $x=1$ | 1 | 3 | 4 |
| Toàn bộ | 3 | 3 | 6 |

Split $x\le0.5$ tạo một lá toàn tích cực và một lá có 1/4 mẫu tích cực. Đây là bảng giả định cho phép tính Gini; không khẳng định mọi đánh giá chứa từ chê đều mang cùng nhãn toàn văn.

## 8. Kết quả và cách diễn giải

Một mẫu đến lá nào sẽ nhận phân bố lớp của lá đó theo ước lượng của cây. Nếu không dùng trọng số, tỷ lệ lớp trong lá là cơ sở trực tiếp; khi dùng sample/class weight cần đọc theo số đếm có trọng số.

Probability bằng 1 ở lá chỉ có một mẫu không chứng minh mô hình chắc chắn đúng ngoài mẫu. Đường đi giải thích phép quyết định của cây cho mẫu, không phải lời giải thích nhân quả hoặc toàn bộ ý nghĩa của văn bản.

## 9. Điểm mạnh, điểm yếu và giới hạn

Cây nhỏ có quy tắc dễ đọc, biểu diễn tương tác và không bắt buộc quan hệ tuyến tính theo feature gốc. Đầu ra có thể truy được nút/lá để kiểm tra lỗi.

Hạn chế gồm dễ overfit, nhạy biến động mẫu và nhiều feature, xác suất ở lá nhỏ không ổn định. Cây sâu có thể quá lớn để “dễ giải thích” trên thực tế. Trong văn bản sparse rất nhiều chiều, cần so với baseline tuyến tính thay vì mặc định cây phi tuyến sẽ tốt hơn.

## 10. So sánh với phương pháp gần nhau

| Mô hình/khái niệm | Khác biệt |
|---|---|
| Linear classifier | Ranh giới tuyến tính; cây kết hợp các phép chia cục bộ |
| Random Forest | Tập nhiều cây ngẫu nhiên hóa và tổng hợp; không cùng khả năng đọc như một cây |
| Gradient Boosting | Học nhiều cây theo trình tự mục tiêu sửa sai/gradient riêng |
| Cây quyết định quản trị | Thường mô tả phương án, kết quả và utility; không tự là cây học từ nhãn |
| DecisionTreeRegressor | Dự đoán mục tiêu số liên tục, tiêu chuẩn lá/split khác classifier |

Decision Tree có thể biểu diễn XOR bằng nhiều nhánh trong feature gốc, nhưng thuật toán tham lam không được bảo đảm khám phá mọi cấu trúc tương tác khó với mọi quy tắc dừng.

## 11. Sai lầm thường gặp và cách khắc phục

- Xem cây fit hoàn hảo train là tốt: đánh giá ngoài mẫu và kiểm soát complexity.
- Tin cây không cần tiền xử lý: xử lý kiểu dữ liệu, nhãn, missing và văn bản đúng cách.
- Đọc importance cao thành tác động mạnh: xác định metric importance và giới hạn.
- Đọc threshold TF-IDF như số lần từ: kiểm tra thang feature thật.
- Dùng cây lớn nhưng gọi “dễ diễn giải”: báo depth, số lá và độ dài đường đi.

## 12. Tài liệu tham khảo

- **[S1]** scikit-learn 1.8. [Decision Trees: cơ chế, tiêu chuẩn và cắt tỉa](https://scikit-learn.org/1.8/modules/tree.html). Tài liệu có dẫn công trình CART của Breiman, Friedman, Olshen & Stone (1984).
- **[S2]** scikit-learn 1.8. [DecisionTreeClassifier API](https://scikit-learn.org/1.8/modules/generated/sklearn.tree.DecisionTreeClassifier.html).
- **[S3]** scikit-learn. [Permutation feature importance](https://scikit-learn.org/stable/modules/permutation_importance.html).
- **[S4]** scikit-learn. [Common pitfalls](https://scikit-learn.org/stable/common_pitfalls.html).
- **[S5]** scikit-learn. [Probability calibration](https://scikit-learn.org/stable/modules/calibration.html).

Nguồn giải thích phương pháp/triển khai; các ví dụ khách sạn và phép tính trong bài là nội dung giảng giải tự xây, không phải kết quả thí nghiệm công bố trong các nguồn này.

## 13. Vị trí trong toàn bộ nghiên cứu

Cây nằm ở bước dự đoán sau biểu diễn hoặc ở bước khám phá quy tắc có kiểm định. Nếu nhãn máy dùng để mô tả trải nghiệm khách, phải kiểm tra độ đúng và tính đại diện trước.

Nếu dùng cây để chọn nhóm biến rồi suy luận trong mô hình tiếp, cần xem việc lựa chọn đã dựa vào cùng dữ liệu kết quả hay chưa. Quy tắc tìm sau khi xem dữ liệu là kết quả khám phá, không tự là giả thuyết đã được xác nhận độc lập.

## 14. Kiến thức nền cần biết trước

Cần hiểu root, internal node, split, branch, leaf, depth, impurity và pruning. Impurity mô tả mức trộn nhãn trong nút, không phải độ bẩn của dữ liệu hay cảm xúc tiêu cực.

Phân biệt entropy dùng cho tiêu chuẩn split với nguyên lý Maximum Entropy của bài trước. Dùng entropy trong cây không biến cây thành MaxEnt classifier. Cũng cần hiểu leaf size và xác suất thực nghiệm tại lá.

## 15. Nền tảng lý thuyết và logic hình thành

Logic là chia không gian feature thành các vùng có nhãn đồng nhất hơn, rồi dùng dự đoán đơn giản trong từng vùng. Cấu trúc nhiều tầng tạo các tương tác điều kiện mà một điểm tuyến tính đơn giản có thể không biểu diễn trực tiếp.

Độ thuần của train tăng khi chia thêm thường đi cùng nguy cơ cây học nhiễu. Do đó, cải thiện impurity cục bộ phải được đặt cạnh mức phức tạp và chất lượng ngoài mẫu, không coi mọi split làm train thuần hơn là có ích cho nghiên cứu.

## 16. Các thành phần cấu tạo

Hệ gồm vectorizer/biểu diễn, tiêu chuẩn impurity, chiến lược tìm split, điều kiện dừng, pruning, dự đoán tại lá và đánh giá. Nếu dùng class weight, cả cách chọn split và phân bố lớp tại lá có thể bị ảnh hưởng.

Nên giữ feature names, node IDs và leaf IDs để kiểm tra. Một đường đi chỉ có giá trị nếu biết mỗi cột feature được tạo ra thế nào và threshold thuộc đơn vị nào.

## 17. Các loại và biến thể

CART, ID3/C4.5 và các biến thể có khác biệt về split, tiêu chuẩn và xử lý dữ liệu. scikit-learn dùng triển khai CART tối ưu hóa [S1]; không gọi `DecisionTreeClassifier` là C4.5.

Có classification/regression tree, splitter best/random, cây hạn chế trước và cây cắt tỉa sau. Bài không mô tả mọi loại là cùng thuật toán. Chi tiết categorical/missing cần đối chiếu từng hệ, không suy từ một mô tả chung của họ cây.

## 18. Đầu vào và đầu ra

Đầu vào $X$ có các cột số và $y$ là nhãn. Với text có thể dùng count/TF-IDF, feature ngôn ngữ hoặc embedding; mỗi biểu diễn tạo ý nghĩa threshold khác nhau.

Đầu ra nên giữ `review_id`, nhãn thật, nhãn dự đoán, xác suất, leaf ID, đường đi, split và model version. Nếu văn bản thành vector 0 vì OOV, cây vẫn dẫn qua các threshold; không vì thế mà xem nó đã hiểu nội dung.

## 19. Căn cứ lựa chọn

Nên thử khi cần kiểm tra quy tắc cục bộ hoặc tương tác và có nhãn phù hợp. Đo depth/số lá, hiệu năng và độ ổn định cùng nhau; một cây nhỏ kém hơn chút có thể hữu ích cho minh họa nhưng phải công bố đánh đổi.

Với corpus text nhiều chiều, nên so cây với LR/SVM trên cùng split. Không chọn cây chỉ vì trực quan, hoặc bỏ cây chỉ vì mô hình đơn giản, trước khi có bằng chứng đúng nhiệm vụ.

## 20. Trường hợp không nên sử dụng

Không dùng một cây sâu học từ ít mẫu để cung cấp quy tắc quản trị chắc chắn. Không dùng ID cá nhân/khách sạn như feature mặc định nếu mục tiêu là nội dung và áp dụng cho đối tượng mới.

Nếu cần xác suất đáng tin, xác suất lá cần kiểm định; cây nhỏ lá thuần không tự hiệu chuẩn tốt. Nếu cần dự đoán ngoài khoảng cho mục tiêu liên tục, cây hồi quy có giới hạn khác; không mượn kết luận classifier cho nhiệm vụ regression.

## 21. Các quyết định quan trọng khi thực hiện

Chọn `criterion`, `max_depth`, `min_samples_split`, `min_samples_leaf`, `max_leaf_nodes`, `ccp_alpha` và trọng số. Các tham số có thể tương tác, nên đặt một quy trình phát triển rõ thay vì tinh chỉnh theo test.

Chọn tỷ lệ/tần suất tối thiểu ở lá theo dữ liệu, không biến một giá trị tùy chọn thành tiêu chuẩn chung. Nếu đọc importance để chọn lại feature, lựa chọn đó thuộc giai đoạn development và cần đánh giá cuối riêng.

## 22. Giải thích công thức và ký hiệu

Với tỷ lệ lớp $p_k$ trong nút $t$, Gini là:

$$G(t)=1-\sum_kp_k^2.$$

Với split tạo hai nút trái/phải, giảm impurity cục bộ là:

$$\Delta G=G(t)-\frac{n_L}{n_t}G(L)-\frac{n_R}{n_t}G(R).$$

$n$ là số mẫu, hoặc tổng trọng số khi dùng weighting. Một số tiêu chuẩn triển khai xét thêm trọng số $n_t/N$ đối với toàn cây; không nhầm với độ giảm cục bộ vừa tính.

Entropy $H=-\sum_kp_k\log p_k$ là tiêu chuẩn khác, với quy ước $0\log0=0$. Cost-complexity pruning có dạng $R_\alpha(T)=R(T)+\alpha|L(T)|$, trong đó thư viện dùng rủi ro impurity có trọng số và số lá theo quy ước [S1]. $\alpha$ lớn phạt cây phức tạp mạnh hơn; chọn bằng dữ liệu phát triển, không bằng test.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Với bảng mục 7: gốc có Gini $1-0.5^2-0.5^2=0.5$. Lá trái thuần nên Gini 0; lá phải có tỷ lệ (0.25,0.75), Gini 0.375. Sau split, impurity có trọng số là $2/6\times0+4/6\times0.375=0.25$, giảm 0.25.

Mã kiểm tra stump (cây một split), rồi chạy pipeline text tự soạn. Hai ví dụ độc lập về dữ liệu; không dùng số đếm stump như kết quả học corpus text.

```python
import numpy as np
from sklearn.tree import DecisionTreeClassifier
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.metrics import f1_score

# Một split đúng theo bảng giả định.
stump = DecisionTreeClassifier(max_depth=1, criterion="gini", random_state=42)
stump.fit([[0], [0], [1], [1], [1], [1]], [1, 1, 1, 0, 0, 0])
positive = np.flatnonzero(stump.classes_ == 1).item()
assert np.allclose(stump.predict_proba([[0], [1]])[:, positive], [1.0, 0.25])
assert np.isclose(stump.tree_.impurity[0], 0.5)
assert np.isclose(0.5 - (2 / 6) * 0 - (4 / 6) * 0.375, 0.25)

train = ["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
         "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
         "dịch vụ tệ bẩn", "khách sạn bẩn tệ"]
y_train = [1, 1, 1, 1, 0, 0, 0, 0]
test = ["phòng sạch zzholdout", "dịch vụ tốt", "phòng bẩn", "dịch vụ tệ"]
y_test = [1, 1, 0, 0]
model = Pipeline([
    ("count", CountVectorizer(ngram_range=(1, 2))),
    ("tree", DecisionTreeClassifier(criterion="gini", max_depth=3,
                                    min_samples_leaf=1, random_state=42)),
])
model.fit(train, y_train)
pred = model.predict(test)
prob = model.predict_proba(test)
X_test = model.named_steps["count"].transform(test)
tree = model.named_steps["tree"]
assert "zzholdout" not in model.named_steps["count"].vocabulary_
assert np.isfinite(prob).all() and np.allclose(prob.sum(axis=1), 1.0)
assert tree.get_depth() <= 3 and tree.apply(X_test).shape == (4,)
print("Pred:", pred.tolist())
print("Macro-F1 minh họa:", f1_score(y_test, pred, average="macro"))
print("Depth / số lá:", tree.get_depth(), tree.get_n_leaves())
```

Mã cần NumPy/scikit-learn. Corpus nhỏ và đơn giản chỉ kiểm tra cách chạy; không đủ đo hiệu năng thật, chọn depth hoặc kết luận cây hơn/kém mô hình khác. Nhãn demo 0/1 không có trung tính.

## 24. Tình huống sai và phản ví dụ

Một cây có thể tạo lá chỉ chứa một đánh giá của khách sạn A và dự đoán xác suất 1.0 theo nhãn đó. Điều này là quy tắc khớp train, không chứng minh A luôn được khen hoặc cây có độ chắc chắn hoàn hảo.

Với “phòng không sạch”, một split theo feature “sạch có xuất hiện” có thể phân sai nếu không có feature/nhãn để phân biệt phủ định. Cây có khả năng biểu diễn điều kiện kết hợp không đồng nghĩa nó tự phát hiện đúng mọi ngữ cảnh.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đề xuất macro-F1, từng lớp, confusion matrix và baseline đa số, cùng depth/số lá/leaf support. Nếu cần probability, kiểm tra log loss, Brier và calibration, không chỉ accuracy.

Kiểm tra split/feature/độ ổn định cây khi thay mẫu train. Cùng mức dự đoán có thể có nhiều cây khác nhau nên một đường đi đơn lẻ không tự là quy tắc ổn định. Permutation importance có thể hỗ trợ kiểm tra mô hình trên held-out data, nhưng feature tương quan và mẫu nhỏ vẫn gây khó diễn giải [S3].

## 26. Xử lý khi không đáp ứng yêu cầu

Overfit: thử hạn chế depth, tăng leaf size hoặc pruning trên development. Underfit: kiểm tra feature, nhãn và mức hạn chế, không chỉ tăng depth vô hạn. Lỗi hiếm/nhóm: xem hỗ trợ mẫu và class weighting.

Importance không ổn định: xem feature tương quan và thử kiểm tra trên dữ liệu độc lập đủ lớn. Nếu lỗi đến từ rò rỉ, phải sửa pipeline và đánh giá lại; giảm depth không biến một test đã bị dùng thành độc lập.

## 27. Phân biệt kết quả với kết luận

“Cây chia đầu tiên theo từ bẩn” là mô tả cấu trúc mô hình. “Vệ sinh là yếu tố quan trọng nhất với mọi khách” là kết luận vượt bằng chứng nếu chỉ dựa vào split đó.

Importance không có dấu tác động và không phải phần trăm hài lòng được giải thích. Tỷ lệ nhãn máy cần được đặt trong sai số và mẫu nghiên cứu; quy tắc dự đoán không thay chiến lược nhận diện nhân quả.

## 28. Cách viết vào bài nghiên cứu

Ghi estimator/version, feature, criterion, depth/leaf/pruning, weighting và cách chọn. Mẫu: “Count n-gram được fit trong từng fold; cây được chọn theo [metric] qua [CV/development], sau đó kiểm tra trên [test].”

Nếu trình bày cây, báo số mẫu/tỷ lệ lớp ở lá và phạm vi áp dụng. Nếu chỉ xuất một cây rút gọn để giải thích, nêu đó là mô hình thật hay bản surrogate; không tráo hai loại trong báo cáo.

## 29. Khả năng tái lập và tính minh bạch

Lưu pipeline, mapping lớp, feature names, cây, cấu hình, seed và phép chia. Nếu dữ liệu có weighting, lưu cách tạo trọng số để tái lập tỷ lệ và split.

Kho GitHub có thể chứa ví dụ tự soạn, mô tả lá và mã export/plot khi cần. Dữ liệu thật cần quyền chia sẻ và ẩn danh; cây có thể tiết lộ các vùng chứa rất ít cá nhân nên cũng cần xem nội dung được công khai.

## 30. Đạo đức nghiên cứu liên quan

Một quy tắc dễ đọc có thể khiến người đọc tin quá mức, đặc biệt khi lá ít mẫu. Không dùng cây chưa kiểm định để tự động đánh giá nhân viên hay quy kết nhóm khách có thái độ tiêu cực.

Kiểm tra lỗi theo ngôn ngữ/kiểu viết và sự hiện diện tên riêng. Nếu feature là nhóm xã hội hoặc proxy, cần lý do phù hợp mục tiêu và đánh giá hệ quả, không dựa riêng vào việc thuật toán đã chọn nó.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **Impurity 0 có chứng minh đúng ngoài mẫu không?** Không; đó là độ thuần trong nút theo dữ liệu học.
2. **Cây không cần scale có nghĩa không cần tiền xử lý không?** Không.
3. **Bài tập tính:** nút có tỷ lệ lớp (0.8,0.2) có Gini 0.32. So với nút (0.5,0.5), giải thích nút nào trộn hơn.
4. **Bài tập thiết kế:** thử vài depth/leaf size trên development, ghi metric và số lá; giữ test ngoài lựa chọn.
5. **Bài tập phản biện:** vì sao feature tên khách sạn ở gốc không đủ chứng minh đặc điểm khách sạn gây sentiment?

## 32. Những điểm còn tranh luận

Các lựa chọn là độ phức tạp, độ ổn định quy tắc, thang feature và cách diễn giải importance. Cây nhỏ có thể dễ kiểm tra nhưng bỏ quan hệ; cây lớn có thể fit tốt nhưng khó đọc và bất ổn.

Nghiên cứu cần nói rõ đang ưu tiên dự đoán, khám phá hay giải thích. Không dùng cùng từ “giải thích được” để gộp đường đi kỹ thuật của mô hình với bằng chứng khoa học về nguyên nhân trải nghiệm khách hàng.
