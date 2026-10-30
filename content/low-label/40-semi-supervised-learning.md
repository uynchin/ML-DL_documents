# Semi-supervised Learning — Học bán giám sát với ít nhãn

> **Nhánh:** MACHINE LEARNING → Ít nhãn. **Nguồn:** dòng 41 bảng Excel. **Đối chiếu:** 29/10/2026. Đây là một họ phương pháp; ví dụ dùng LabelSpreading trên train pool L+U, không dùng test trong fit. Mã chạy sklearn 1.8.0. Dữ liệu và tình huống khách sạn đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bối cảnh | Phân tích cảm xúc review khách sạn tiếng Việt |
| Nguồn tín hiệu | Phân biệt gold labels, pseudo/weak labels và target tự tạo trong bài |
| Đánh giá | Cần protocol và dữ liệu độc lập phù hợp, không có benchmark review thật ở ví dụ |

## 1. Định nghĩa học thuật

Semi-supervised learning kết hợp dữ liệu có nhãn L với dữ liệu chưa nhãn U để học một nhiệm vụ có giám sát, như phân loại cảm xúc. Nhãn ít nhưng chất lượng cao định nghĩa mục tiêu; U cung cấp thêm cấu trúc phân bố đầu vào. Đây là một họ phương pháp, không phải tên riêng của LabelSpreading hay self-training [S1].

## 2. Giải thích đơn giản

Bạn có một ít review khách sạn được đọc và gán nhãn, cùng nhiều review chưa được đọc. Mô hình cố tận dụng cách các review liên quan nhau để học tốt hơn. Nhưng nhiều review chưa nhãn không tự chứa đáp án đúng; nếu quan hệ “gần nhau” phản ánh chủ đề thay vì cảm xúc, việc tận dụng chúng có thể làm sai thêm.

## 3. Mục đích và vấn đề giải quyết

Mục tiêu là giảm nhu cầu nhãn cho một mức chất lượng dự đoán chấp nhận được. Trong nghiên cứu khách sạn, có thể kiểm tra liệu thêm U giúp vượt baseline chỉ dùng L trên test độc lập. Cần giữ ngân sách nhãn, biểu diễn và quyền truy cập dữ liệu tương đương khi so sánh; không coi nhãn giả là bằng chứng mới được con người xác nhận.

## 4. Điều kiện và giả định sử dụng

Cần L đủ đại diện các lớp, U có liên hệ phù hợp với nhiệm vụ và thiết kế đánh giá rõ. Các giả định thường dùng gồm tính trơn, cấu trúc cụm hoặc manifold: mẫu gần/ở cùng vùng được kỳ vọng liên quan nhãn. Chúng không tự đúng với review cùng chủ đề nhưng trái cảm xúc. U lệch miền có thể gây hại.

## 5. Cơ chế hoạt động bên trong

Tùy phương pháp, mô hình truyền thông tin nhãn trên đồ thị, tự sinh nhãn giả, ép nhất quán giữa biến đổi hoặc kết hợp loss có/không nhãn. Ví dụ LabelSpreading xây quan hệ giữa mẫu và làm trơn phân bố nhãn qua đồ thị, có soft clamping. Đây không phải chỉ chạy clustering rồi gán tên cụm bằng trực giác [S1–S2].

## 6. Quy trình áp dụng trong nghiên cứu

1. Chốt nhiệm vụ, codebook và tập nhãn thủ công tin cậy.
2. Tách L, U-train, development và test theo nhóm/thời gian.
3. Công bố U nào tham gia học biểu diễn và thuật toán.
4. Chạy baseline chỉ dùng L.
5. Chọn giả định/cấu hình bằng development, đo cả khi U lệch miền.
6. Khóa pipeline rồi đánh giá test.

Ví dụ được phép fit vocabulary trên L+U-train, không gồm test. Nếu cho test không nhãn tham gia đồ thị/biểu diễn, phải báo thiết kế transductive, không trình bày như inductive hoàn toàn.

## 7. Ví dụ minh họa xuyên suốt

Giả định L có “phòng sạch”/“nhân viên tốt” tích cực và “phòng bẩn”/“nhân viên tệ” tiêu cực. U gồm câu có từ quen và câu diễn đạt khác. Tính gần có thể giúp một số trường hợp, nhưng “phòng không sạch” có thể gần câu tích cực nếu biểu diễn bỏ qua phủ định. Ví dụ nhỏ chỉ minh họa cơ chế, không đo hiệu quả thực tế.

## 8. Kết quả và cách diễn giải

Đầu ra cuối là dự đoán nhãn/xác suất theo thuật toán. Với graph methods, phân bố nhãn của các hàng trong pool là kết quả suy luận của mô hình, không phải gold labels. Cần phân biệt dự đoán cho U đã tham gia fit và dự đoán cho mẫu mới độc lập; metric trên hai tình huống trả lời các câu hỏi khác nhau.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là khai thác U khi nhãn đắt và có thể giúp học ranh giới/biểu diễn. Hạn chế là phụ thuộc giả định, chất lượng L và mức tương thích miền; có thể củng cố sai lệch hoặc làm lớp hiếm kém hơn. Có U nhiều không bảo đảm cải thiện và còn tăng chi phí, nhất là đồ thị lớn.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| Supervised | Chỉ khai thác nhãn/đầu vào train theo thiết kế không có SSL |
| Self-training | Một cách bán giám sát bằng nhãn giả |
| Self-supervised | Target được tạo từ dữ liệu cho nhiệm vụ học biểu diễn, không nhất thiết dùng nhãn sentiment |
| Active learning | Chọn mẫu để hỏi nhãn thật |
| Weak supervision | Nguồn nhãn có thể không chính xác/không đầy đủ |

Các cách có thể kết hợp, không là những nhóm loại trừ tuyệt đối.

## 11. Sai lầm thường gặp và cách khắc phục

Không gọi -1 là lớp cảm xúc thứ ba trong estimator dùng -1 để đánh dấu chưa nhãn. Không đánh giá bằng chính pseudo-labels rồi báo accuracy. Không cho U chứa bản sao test trong một thiết kế inductive. Không kết luận thất bại do “ít nhãn” nếu L chưa có một lớp hoặc codebook không nhất quán.

## 12. Tài liệu tham khảo

- [S1] scikit-learn. [Semi-supervised learning](https://scikit-learn.org/1.8/modules/semi_supervised.html).
- [S2] scikit-learn. [LabelSpreading](https://scikit-learn.org/1.8/modules/generated/sklearn.semi_supervised.LabelSpreading.html).
- [S3] scikit-learn. [SelfTrainingClassifier](https://scikit-learn.org/1.8/modules/generated/sklearn.semi_supervised.SelfTrainingClassifier.html).

Nguồn [S1] cho các giả định và họ phương pháp; mã/hướng dẫn nghiên cứu dưới tự soạn, không có benchmark dữ liệu người dùng.

## 13. Vị trí trong toàn bộ nghiên cứu

Học bán giám sát nằm ở giai đoạn huấn luyện, sau thiết kế mẫu và gán một phần nhãn. Nó không thay bước xác định cảm xúc hay kiểm định độ nhất quán người gán nhãn. Nếu dự đoán được tổng hợp theo khách sạn, phải đánh giá sai số và đại diện mẫu trước suy luận về toàn bộ khách lưu trú.

## 14. Kiến thức nền cần biết trước

Cần hiểu dữ liệu có/chưa nhãn, leakage, validation/test, probability và giả định lân cận. Với graph methods, hiểu similarity kernel và chi phí ma trận quan hệ. Với tiếng Việt, biết n-gram/embedding có thể giữ hoặc làm mất phủ định; thuật toán không tự sửa thông tin đã bị tiền xử lý loại bỏ.

## 15. Nền tảng lý thuyết và logic hình thành

U chỉ giúp khi cấu trúc p(x) cung cấp thông tin hữu ích cho quan hệ nhãn p(y|x) dưới giả định của phương pháp. Không có nguyên tắc chung rằng thêm mẫu chưa nhãn luôn giảm lỗi. Nếu cấu trúc chủ đề khác cấu trúc sentiment, ranh giới được làm trơn theo cụm có thể đi sai hướng.

## 16. Các thành phần cấu tạo

Thành phần gồm L với gold labels, U-train, mô hình cơ sở/đồ thị, thành phần loss không nhãn, hệ số cân bằng, tiêu chí dừng và protocol đánh giá. Phân biệt U-train với test giấu nhãn: đều không đưa y vào fit nhưng có vai trò và quyền truy cập khác nhau.

## 17. Các loại và biến thể

Có graph-based, self-training, consistency regularization và các phương pháp khác. Ví dụ LabelPropagation/LabelSpreading có cách giữ nhãn ban đầu khác nhau. Không suy mọi graph method nhận sparse TF-IDF hoặc có cùng predict API; kiểm tra estimator/cấu hình. Ví dụ bài chuyển dense chỉ vì số mẫu rất nhỏ.

## 18. Đầu vào và đầu ra

Đầu vào gồm X của L+U, vector y với sentinel -1 trong ví dụ và mapping gold labels 0/1. Đầu ra có các phân bố/gán nhãn của pool và predict cho dữ liệu mới theo API. Không đưa -1 vào một classifier supervised thông thường rồi mặc định nó hiểu là chưa nhãn.

## 19. Căn cứ lựa chọn

Đáng thử khi có U cùng miền, L có chất lượng và giả định về cấu trúc có lý do. Căn cứ chọn là lợi ích trên gold test với cùng ngân sách nhãn và độ bền. Nên làm đường learning curve theo số nhãn và so baseline supervised, thay vì chỉ báo một cấu hình SSL tốt nhất.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu U rất lệch miền, nhãn định nghĩa mơ hồ hoặc không thể dành gold development/test. Không dùng SSL để hợp thức hóa nhãn giả như dữ liệu khảo sát thật. Nếu nguồn review bị chọn lệch theo nền tảng/ngôn ngữ, việc thêm U không tự làm mẫu đại diện.

## 21. Các quyết định quan trọng khi thực hiện

Chọn L/U theo đơn vị nhóm/thời gian, cách lấy seed labels, biểu diễn, kernel/loss, weighting và tiêu chí dừng. Chốt inductive hay transductive trước. Giữ tỷ lệ lớp và nội dung hiếm trong gold evaluation; không chỉ gán nhãn những câu dễ rồi xem đó là chuẩn chất lượng toàn corpus.

## 22. Giải thích công thức và ký hiệu

Dạng mục tiêu minh họa cho một số phương pháp:

$$L(\theta)=\frac1{|L|}\sum_{i\in L}\ell(y_i,f_\theta(x_i))+\lambda\frac1{|U|}\sum_{j\in U}R(x_j;\theta).$$

$R$ phụ thuộc thiết kế; không phải mọi graph estimator tối ưu đúng công thức này. Với đồ thị có trọng số a, tính trơn có thể đo bằng $\sum_{ij}a_{ij}\|F_i-F_j\|^2$. Tối thiểu hóa trơn cần liên hệ đúng giữa gần và nhãn; trơn theo chủ đề không bảo đảm trơn theo cảm xúc.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Ví dụ fit vocabulary trên L+U-train; không có test tham gia. Dense chỉ dùng cho tám mẫu tự tạo.

```python
import numpy as np
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.semi_supervised import LabelSpreading
texts = ["phòng sạch", "nhân viên tốt", "phòng sạch đẹp", "nhân viên thân thiện",
         "phòng bẩn", "nhân viên tệ", "phòng bẩn tệ", "nhân viên chậm"]
y = np.array([1,1,-1,-1,0,0,-1,-1])
vec = TfidfVectorizer(ngram_range=(1,2))
X = vec.fit_transform(texts).toarray()
model = LabelSpreading(kernel="rbf", gamma=5, alpha=0.2,
                       max_iter=1000, tol=1e-5).fit(X,y)
assert np.array_equal(model.classes_, [0,1])
assert np.allclose(model.label_distributions_.sum(axis=1),1)
assert np.isfinite(model.label_distributions_).all()
new = vec.transform(["phòng sạch zzholdout"]).toarray()
assert "zzholdout" not in vec.vocabulary_
assert np.allclose(model.predict_proba(new).sum(axis=1),1)
print("Nhãn mô hình suy cho pool:", model.transduction_)
print("Dự đoán cho câu mới:", model.predict(new))
```

Không dùng nhãn suy cho pool để tính độ chính xác nếu chưa có gold labels độc lập. Mã không chứng minh SSL tốt hơn supervised; đó là câu hỏi cần thực nghiệm riêng.

## 24. Tình huống sai và phản ví dụ

Một đồ thị có thể nối “phòng sạch” với “phòng không sạch” mạnh hơn với “dịch vụ tốt”. Nhãn được truyền đúng theo đồ thị nhưng sai về sentiment. Nếu U còn chứa tên khách sạn lặp lại, đồ thị có thể nhóm theo thương hiệu. Các phản ví dụ cần kiểm tra trước khi kết luận dữ liệu chưa nhãn hữu ích.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đo macro-F1 và metric theo lớp trên gold test, độ bền theo cách chọn L và seed. Kiểm tra U cùng/lệch miền, tỷ lệ unlabeled và các nhóm câu khó. Với thiết kế transductive, ghi rõ test input đã tham gia. Không dùng gold test để chọn kernel, số vòng hoặc lượng U.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu SSL giảm chất lượng, kiểm tra giả định/miền của U, độ đại diện L và biểu diễn. Có thể giảm weighting U, thay phương pháp hoặc dùng supervised baseline. Đừng thêm U vô hạn khi sai lệch nằm ở cấu trúc. Khi phát hiện leakage, phải chạy lại cả lựa chọn mô hình với split đúng.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ là mức thay đổi chất lượng theo số gold labels và U trong protocol cụ thể. Không kết luận nhãn giả là sự thật hoặc tiết kiệm một tỷ lệ chi phí cố định cho mọi corpus. Không suy tương quan nhãn dự đoán-rating thành quan hệ nhân quả về trải nghiệm khách.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu dùng L=[...] nhãn thủ công và U=[...] review chưa nhãn trong train. Phương pháp bán giám sát [...], cấu hình chọn trên gold development. Test [nhóm/thời gian] không tham gia fit trong thiết kế inductive.” Nếu transductive, thay mô tả đúng quyền truy cập. Báo codebook, ngân sách nhãn và baseline.

## 29. Khả năng tái lập và tính minh bạch

Lưu IDs của L/U/development/test, nguồn nhãn, seed chọn L, biểu diễn và tham số. Giữ riêng gold với nhãn mô hình suy ra. Mã minh họa chạy sklearn 1.8.0; nghiên cứu thật cần protocol và dữ liệu được phép chia sẻ, không chỉ file transduction_ cuối.

## 30. Đạo đức nghiên cứu liên quan

Không gọi pseudo-label là ý kiến trực tiếp của khách đã được xác nhận. Bảo vệ thông tin review và kiểm tra lỗi theo bối cảnh có dữ liệu hợp lệ. Nếu dùng dự đoán ưu tiên khiếu nại, có cơ chế người xem lại. Ít nhãn không miễn trách nhiệm đo sai số ở nhóm hiếm.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Vì sao U không luôn giúp?
2. -1 trong ví dụ có là sentiment không?
3. Phân biệt inductive/transductive theo quyền truy cập.
4. Cần tập nhãn nào để đánh giá?

Gợi ý: gold test độc lập theo protocol. Bài mở rộng: vẽ learning curve theo số nhãn với baseline chỉ dùng L và cùng cách lấy seed.

## 32. Những điểm còn tranh luận

Các vấn đề còn cần kiểm chứng là mức tương thích U, nhãn tối thiểu cho nhóm hiếm và độ ổn định khi chuyển miền. Không có giả định SSL đúng cho mọi review. Nghiên cứu nên công bố quyền truy cập dữ liệu và cả trường hợp thêm U gây hại, thay vì chỉ báo cấu hình thuận lợi.

