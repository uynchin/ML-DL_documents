# NMF — Phân rã ma trận không âm cho chủ đề văn bản

> **Nhánh:** MACHINE LEARNING → Unsupervised và Topic. **Nguồn:** dòng 37 bảng Excel. **Đối chiếu:** 29/10/2026. Tập trung NMF Frobenius trên TF-IDF, phân biệt trọng số thành phần với xác suất. Mã chạy sklearn 1.8.0. Mọi ví dụ và dữ liệu trong bài đều tự tạo.

| Trường | Nội dung |
|---|---|
| Vai trò | Khám phá/biểu diễn không dùng nhãn cảm xúc trong ví dụ |
| Bối cảnh | Review khách sạn tiếng Việt; không có benchmark dữ liệu thật |
| Đọc kết quả | Phân biệt cấu trúc học được với nhãn/khái niệm được xác thực |

## 1. Định nghĩa học thuật

Non-negative Matrix Factorization xấp xỉ một ma trận không âm X bằng tích hai ma trận không âm W và H có rank mục tiêu nhỏ. Với văn bản, W biểu diễn mức đóng góp thành phần cho tài liệu, H biểu diễn feature của từng thành phần. Không âm khuyến khích biểu diễn cộng, nhưng không tự tạo mô hình xác suất topic [S1–S2].

## 2. Giải thích đơn giản

Hình dung mỗi review được ghép từ vài “gói từ” với mức đóng góp khác nhau, và không được dùng hệ số âm để trừ một gói. Cách đọc cộng thường trực quan hơn trục có dấu của LSA. Tuy nhiên, tên “phòng”, “giá” vẫn do người đọc xác định từ từ khóa và review, không tự là nhãn thật.

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu khách sạn, NMF có thể khám phá nhóm từ đồng xuất hiện, tạo feature gọn hoặc hỗ trợ đọc nội dung theo khía cạnh. Có thể so với LDA/LSA bằng protocol chung. Nếu cần cảm xúc của khía cạnh, phải có cách nhận diện khen/chê riêng; thành phần nói về phòng có thể chứa cả hai.

## 4. Điều kiện và giả định sử dụng

X phải không âm; TF-IDF/count thường phù hợp điều kiện này. Vector đã center hoặc embedding có số âm không thể đưa trực tiếp vào NMF chuẩn. Cần chọn số thành phần, loss, regularization và dữ liệu đủ cấu trúc. Dữ liệu ít/trùng có thể cho thành phần dễ nhìn nhưng không bền.

## 5. Cơ chế hoạt động bên trong

Khởi tạo W,H không âm rồi tối ưu luân phiên hoặc bằng cập nhật phù hợp để giảm loss dưới ràng buộc không âm. Không lồi đồng thời theo cả W,H nên có thể có nghiệm cục bộ. Khi transform tài liệu mới, giữ H đã fit và tìm W mới không âm, thay vì fit lại toàn bộ thành phần.

## 6. Quy trình áp dụng trong nghiên cứu

1. Chọn đơn vị và mục tiêu mô tả/biểu diễn.
2. Loại trùng, xây ma trận không âm trong phạm vi fit.
3. Chọn rank, init, solver, loss và regularization bằng dữ liệu phát triển.
4. Đọc top words cùng tài liệu có trọng số cao/thấp và câu hỗn hợp.
5. Kiểm tra ổn định/tái tạo/nhiệm vụ downstream.

Nếu đưa W vào classifier, fit vectorizer và NMF trong train của từng fold; không fit toàn corpus rồi gọi đánh giá classifier là hoàn toàn ngoài mẫu.

## 7. Ví dụ minh họa xuyên suốt

Corpus tự tạo có nhóm từ phòng/giường, nhân viên/lễ tân, giá/chi phí. NMF với ba thành phần có thể gợi ba nhóm nội dung nhưng không bảo đảm đúng trực giác. Một câu có nhiều khía cạnh có thể có nhiều hệ số W khác 0. Tổng hàng W không mặc định bằng 1 và không là tỷ trọng cảm xúc.

## 8. Kết quả và cách diễn giải

Hàng H chứa trọng số từ/cụm theo thành phần; hàng W chứa hệ số tài liệu. Chuẩn hóa hàng W để dễ vẽ chỉ tạo một quy ước tỷ trọng tương đối, không biến NMF thành posterior LDA. Phân rã có bất định thang: tăng W một cột và giảm H hàng tương ứng có thể giữ tích WH.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là thành phần cộng dễ đọc, nhận ma trận văn bản không âm và cho phép transform tài liệu mới. Hạn chế gồm cần rank, nhạy khởi tạo/cấu hình, có thể tạo topic lặp và không nắm đầy đủ thứ tự/ngữ cảnh. Nonnegative không có nghĩa mọi thành phần đúng nghĩa chuyên môn hoặc vô thiên lệch.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| LSA | Hệ số có thể âm; NMF ràng buộc không âm |
| LDA | Mô hình xác suất với Dirichlet; NMF không tự có posterior |
| K-means | Thường một cụm; W cho nhiều mức đóng góp |
| Autoencoder | Có encoder/decoder, có thể phi tuyến |

So chủ đề cần giữ cùng đơn vị và cách đánh giá; không lấy reconstruction loss của NMF so trực tiếp với perplexity LDA như cùng thang.

## 11. Sai lầm thường gặp và cách khắc phục

Không đưa embedding có số âm vào NMF rồi tùy ý cộng hằng mà không giải thích thay đổi hình học. Không coi W/H là probability tự nhiên. Không đặt init/solver/loss không tương thích theo ví dụ cũ. Nếu dùng multiplicative updates với khởi tạo có nhiều 0, cần hiểu vấn đề zero-locking theo cấu hình [S2].

## 12. Tài liệu tham khảo

- [S1] Lee, D. D., & Seung, H. S. (1999). *Learning the Parts of Objects by Non-negative Matrix Factorization*. Nature 401, 788–791. [Bài gốc](https://www.nature.com/articles/44565).
- [S2] scikit-learn. [NMF](https://scikit-learn.org/1.8/modules/generated/sklearn.decomposition.NMF.html).
- [S3] scikit-learn. [Decomposition](https://scikit-learn.org/1.8/modules/decomposition.html).

Mốc 1999 là công trình tham khảo quan trọng, không khẳng định mọi ý tưởng phân rã không âm bắt đầu ở năm đó.

## 13. Vị trí trong toàn bộ nghiên cứu

NMF thường ở khám phá hoặc xây feature. Khi dùng thành phần như khía cạnh nghiên cứu, cần quy trình đặt tên, ngoại kiểm và đánh giá mức ổn định. Không thay bước xây thang đo lý thuyết chỉ bằng top words. Phân tích quan hệ thành phần-rating là bước thống kê khác, cần điều kiện và giới hạn.

## 14. Kiến thức nền cần biết trước

Cần hiểu tích ma trận, rank, Frobenius norm, tối ưu ràng buộc và weighting văn bản. Phân biệt chỉ số thành phần với nhãn nội dung; thành phần số 0 không giữ tên cố định giữa mọi lượt. Biết lý do TF-IDF không âm phù hợp số học nhưng không đảm bảo chủ đề có ý nghĩa.

## 15. Nền tảng lý thuyết và logic hình thành

Không âm giới hạn phép xấp xỉ vào những thành phần cộng, giảm khả năng triệt tiêu bằng hệ số dấu ngược. Loss chọn loại sai khác cần giảm; regularization kiểm soát độ lớn/thưa. Nghiệm không nhất thiết duy nhất hoặc tối ưu toàn cục. Tính dễ đọc có thể đến từ ràng buộc chứ không là chứng minh mô hình sinh đúng.

## 16. Các thành phần cấu tạo

Thành phần gồm X, W, H, rank r, init, solver, loss, regularization và điều kiện dừng. Với X n×V, W n×r và H r×V trong quy ước bài. Thư viện/tài liệu khác có thể đảo tên W/H, nên kiểm tra kích thước thay vì suy từ chữ cái.

## 17. Các loại và biến thể

Có NMF theo Frobenius, generalized KL và beta-divergence khác, cùng biến thể sparse/regularized/mini-batch. Điều kiện dữ liệu và solver phụ thuộc loss; không khẳng định mọi beta-loss chấp nhận mọi ma trận có số 0. Đổi loss thay mục tiêu, cần công bố thay vì chỉ ghi “NMF mặc định”.

## 18. Đầu vào và đầu ra

Đầu vào là ma trận không âm cùng schema. Đầu ra W và H, sai số tái tạo theo triển khai và số vòng. Transform tài liệu toàn 0 có thể tạo code không cung cấp nội dung; phát hiện riêng. Không áp H của vocabulary cũ lên cột vocabulary mới hoặc đảo thứ tự từ.

## 19. Căn cứ lựa chọn

Đáng thử khi muốn thành phần cộng dễ đọc và có count/TF-IDF hợp lý. Chọn rank bằng khả năng diễn giải, ổn định và nhiệm vụ thực, không chỉ giảm loss. Giữ baseline LSA/LDA phù hợp để biết giá trị của ràng buộc không âm trong corpus của mình.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu dữ liệu đầu vào có dấu mà không có mô hình biến đổi hợp lý, nếu cần posterior xác suất rõ ràng hoặc nếu kỳ vọng xử lý mỉa mai từ bag-of-words. Nếu thành phần không bền qua mẫu, không nên dùng tên của chúng như biến đo khách quan cố định.

## 21. Các quyết định quan trọng khi thực hiện

Chọn count hay TF-IDF, rank, init, solver/loss, regularization và số vòng. Ghi quy tắc chuẩn hóa khi vẽ W/H và xử lý thành phần có tổng 0. Nếu có nhiều lượt chọn topic đẹp, công bố protocol/ngân sách thay vì chỉ giữ một lượt thuận câu chuyện.

## 22. Giải thích công thức và ký hiệu

Minh họa không regularization:

$$\min_{W,H\ge0}\frac12\|X-WH\|_F^2.$$

Với W có các hàng (1,0), (0,1), (1,1) và H có các hàng (2,0), (0,3), tích WH có các hàng (2,0), (0,3), (2,3). Nếu D là ma trận đường chéo dương thì $(WD)(D^{-1}H)=WH$, cho thấy bất định thang. Chuẩn hóa để hiển thị phải được ghi rõ, không gọi các hệ số thô là xác suất.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mã kiểm tra bất định thang và fit NMF thật trên TF-IDF. Không có nhãn cảm xúc.

```python
import numpy as np
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.decomposition import NMF
W0 = np.array([[1.,0.],[0.,1.],[1.,1.]])
H0 = np.array([[2.,0.],[0.,3.]])
scale = np.diag([2.,0.5])
assert np.allclose(W0@H0, (W0@scale)@(np.linalg.inv(scale)@H0))
docs = ["phòng sạch giường êm", "phòng bẩn giường cũ", "phòng rộng giường đẹp",
        "nhân viên nhiệt tình lễ tân", "nhân viên chậm lễ tân", "nhân viên thân thiện lễ tân",
        "giá cao chi phí đắt", "giá thấp chi phí rẻ", "giá hợp lý chi phí"]
vec = TfidfVectorizer(ngram_range=(1,2))
X = vec.fit_transform(docs)
nmf = NMF(n_components=3, init="nndsvda", solver="cd",
          beta_loss="frobenius", max_iter=2000, random_state=42)
W = nmf.fit_transform(X)
H = nmf.components_
assert W.shape == (len(docs),3) and (W >= 0).all() and (H >= 0).all()
assert np.isfinite(W).all() and np.isfinite(H).all()
terms = vec.get_feature_names_out()
for k, row in enumerate(H):
    print(k, terms[np.argsort(row)[-5:][::-1]].tolist())
new = vec.transform(["phòng giường zzholdout"])
assert "zzholdout" not in vec.vocabulary_ and new.nnz > 0
assert nmf.transform(new).shape == (1,3)
```

Kết quả in ra là thành phần học trên ví dụ nhỏ. Không khẳng định có đúng ba topic thật hoặc W là tỷ lệ nội dung đã xác thực.

## 24. Tình huống sai và phản ví dụ

Một thành phần có “phòng, sạch, bẩn” không phải sentiment tích cực. Một thành phần bị chi phối tên khách sạn có thể phản ánh nguồn dữ liệu. Hai khởi tạo có thể cho thứ tự/thang khác nhưng tái tạo tương tự. So hệ số thô giữa các lượt mà không căn chỉnh có thể dẫn kết luận sai.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Kiểm tra hội tụ, reconstruction, sparsity theo mục tiêu, top words và review đại diện. Lặp seed/lấy mẫu và căn chỉnh thành phần để đo ổn định. Nếu dùng thành phần cho dự đoán, đánh giá downstream ngoài mẫu. Không chỉ báo một loss thấp mà bỏ topic trùng, khó đọc hoặc bỏ sót nội dung hiếm.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu không hội tụ, xem max_iter, tol, dữ liệu và init/solver tương thích. Nếu topic lặp, xem rank và từ chung; nếu lỗi ngữ nghĩa, xem biểu diễn. Đổi phương pháp hoặc số thành phần cần đánh giá lại protocol. Không xóa review khó để topic trông đẹp mà không có tiêu chí định trước.

## 27. Phân biệt kết quả với kết luận

Kết quả là xấp xỉ và các thành phần theo cấu hình cụ thể. Có thể kết luận chúng hỗ trợ tóm tắt corpus nếu nội dung/độ bền được xác thực. Không suy hệ số thành phần là tác động nhân quả hoặc tỷ trọng trải nghiệm thật. NMF không tự xác định khách hàng hài lòng hay không.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu dùng NMF trên [TF-IDF/count] với rank [...], init [...], solver/loss [...], regularization [...]. Topic đặt tên theo [...] và đánh giá ổn định bằng [...].” Báo phạm vi fit, hội tụ, chuẩn hóa hiển thị và phản ví dụ. Không gọi W/H là probability nếu chỉ chuẩn hóa phục vụ mô tả.

## 29. Khả năng tái lập và tính minh bạch

Lưu vocabulary/IDF, H, quy tắc transform/chuẩn hóa, seed, phiên bản và mapping tên. Nếu lưu W, giữ thứ tự ID tài liệu. Mã mục 23 đã chạy trên sklearn 1.8.0; thực nghiệm thật cần đủ corpus và quyền chia sẻ. Không giữ ảnh từ khóa thay cho cấu hình tái tạo.

## 30. Đạo đức nghiên cứu liên quan

Kiểm tra top terms có tên/thông tin cá nhân và tránh công khai review đại diện dễ truy ngược. Không coi thành phần không âm là thang đo không thiên lệch. Cần đánh giá nhóm/bối cảnh khi có dữ liệu hợp lệ, bảo vệ quyền người viết và tránh quy kết nhân viên từ phân tích tự động.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Vì sao W không tự có tổng hàng bằng 1?
2. Bất định thang WD/D⁻¹H nghĩa là gì?
3. Embedding âm có phù hợp trực tiếp NMF chuẩn không?
4. Thành phần phòng có phải tích cực?

Gợi ý: tích giữ nguyên dù thang thành phần đổi. Bài mở rộng: so độ bền top words qua seed và đọc review trái tên topic.

## 32. Những điểm còn tranh luận

Các vấn đề cần kiểm chứng gồm tính duy nhất/ổn định, rank phù hợp và mức dễ đọc có phản ánh giá trị nghiên cứu không. Không có loss/init tốt nhất cho mọi corpus. Nghiên cứu cần phân biệt tiện lợi diễn giải với xác thực khái niệm và xem cả nội dung hiếm bị bỏ qua.

