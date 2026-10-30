# LSA — Phân tích ngữ nghĩa tiềm ẩn bằng SVD

> **Nhánh:** MACHINE LEARNING → Unsupervised và Topic. **Nguồn:** dòng 36 bảng Excel. **Đối chiếu:** 29/10/2026. Tập trung TF-IDF và truncated SVD không centering. Không nhầm LSA với Linear Discriminant Analysis. Mã chạy sklearn 1.8.0. Mọi ví dụ và dữ liệu trong bài đều tự tạo.

| Trường | Nội dung |
|---|---|
| Vai trò | Khám phá/biểu diễn không dùng nhãn cảm xúc trong ví dụ |
| Bối cảnh | Review khách sạn tiếng Việt; không có benchmark dữ liệu thật |
| Đọc kết quả | Phân biệt cấu trúc học được với nhãn/khái niệm được xác thực |

## 1. Định nghĩa học thuật

Latent Semantic Analysis biểu diễn quan hệ tài liệu-từ trong không gian hạng thấp, thường dùng SVD cắt ngọn của ma trận count/TF-IDF. Các chiều tiềm ẩn là tổ hợp tuyến tính của feature, có thể mang dấu âm/dương. Chúng không phải phân bố topic xác suất và không được học từ nhãn cảm xúc [S1–S2].

## 2. Giải thích đơn giản

Hình dung nén một bảng rất nhiều cột từ/cụm từ thành ít trục hơn, giữ các cấu trúc biến thiên quan trọng theo mục tiêu phân rã. Review có từ khác nhau nhưng đồng xuất hiện theo cấu trúc có thể gần hơn trong không gian mới. Tuy nhiên, một trục có dấu âm không đồng nghĩa tiêu cực; đó là tọa độ số học.

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu khách sạn, LSA có thể giảm chiều TF-IDF để tìm kiếm tương đồng, phân cụm hoặc làm feature cho classifier. Câu hỏi cần kiểm tra là phép nén có giữ thông tin hữu ích cho nhiệm vụ không. Giảm chiều không tự tạo nhãn hoặc xác nhận các trục là khái niệm quản trị chuẩn.

## 4. Điều kiện và giả định sử dụng

Cần ma trận số phù hợp, chọn rank không vượt cấu hình dữ liệu và xử lý vocabulary. Trong sklearn, TruncatedSVD không center dữ liệu, phù hợp ma trận sparse; không đồng nhất với PCA đã trừ trung bình. Review quá ít hoặc feature nghèo có thể tạo trục khó giải thích. Mục tiêu nén phương sai/năng lượng không ưu tiên tín hiệu cảm xúc hiếm.

## 5. Cơ chế hoạt động bên trong

Xấp xỉ X bằng các singular vectors/values lớn nhất; dùng phép chiếu lên không gian thành phần để lấy tọa độ tài liệu. Dữ liệu mới transform bằng vocabulary và các trục đã fit. Randomized SVD là cách tính gần đúng có seed và tham số, không phải một mô hình topic xác suất khác.

## 6. Quy trình áp dụng trong nghiên cứu

1. Chốt mục tiêu giảm chiều và đơn vị tài liệu.
2. Loại trùng, xây count/TF-IDF trong phạm vi fit.
3. Chọn rank và cấu hình SVD bằng dữ liệu phát triển.
4. Đọc loading cả hai dấu, xem review có tọa độ cực trị.
5. Đánh giá nhiệm vụ sau giảm chiều, ổn định và tài nguyên.

Nếu classifier dùng tọa độ LSA, vectorizer và SVD phải fit trong train mỗi fold; giảm chiều không giám sát vẫn có thể rò rỉ thông tin corpus.

## 7. Ví dụ minh họa xuyên suốt

Trong corpus tự tạo, các từ phòng/giường cùng xuất hiện, nhân viên/lễ tân cùng xuất hiện và giá/chi phí cùng xuất hiện. SVD có thể giữ các cấu trúc đó trong ít chiều. Một review khen phòng và review chê phòng vẫn có thể gần nhau vì nội dung chung. Không đặt tên trục “tích cực” chỉ vì loading từ sạch cao.

## 8. Kết quả và cách diễn giải

Đầu ra Z là tọa độ có dấu; components_ là các trục theo feature, không phải xác suất từ. explained_variance_ratio_ trong API liên quan phương sai của phép chiếu theo quy ước, không phải accuracy hoặc tỷ lệ ý nghĩa được hiểu. Lật dấu cả trục và tọa độ tương ứng không thay tái tạo nên dấu riêng không có ý nghĩa cảm xúc cố định.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là phép nén tuyến tính rõ, hoạt động với dữ liệu sparse qua TruncatedSVD và hỗ trợ downstream models. Hạn chế gồm mất thông tin khi cắt rank, loading có dấu khó đọc như topic, nhạy tiền xử lý và không mô hình hóa ngữ cảnh đầy đủ. Tái tạo tốt bảng từ chưa bảo đảm nhiệm vụ cảm xúc tốt.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| PCA | PCA thường center; TruncatedSVD trong LSA ví dụ không center |
| LDA topic model | Có phân bố hỗn hợp xác suất; LSA là tọa độ tuyến tính |
| NMF | Giới hạn không âm, thường dễ đọc như thành phần cộng |
| Autoencoder | Có thể phi tuyến, cần kiến trúc/huấn luyện khác |

“Semantic” trong tên không bảo đảm hiểu ngữ nghĩa câu như con người.

## 11. Sai lầm thường gặp và cách khắc phục

Không normalize tọa độ LSA theo tổng hàng rồi gọi probability vì có thể có số âm. Không chỉ đọc top loading dương mà bỏ phần âm khi mô tả trục. Đừng so dấu trục giữa seed như ý nghĩa bất biến. Không chọn số chiều trên test của classifier rồi báo kết quả độc lập.

## 12. Tài liệu tham khảo

- [S1] Deerwester, S., Dumais, S. T., Furnas, G. W., Landauer, T. K., & Harshman, R. (1990). *Indexing by Latent Semantic Analysis*. JASIS 41, 391–407. [Trang nhà xuất bản](https://asistdl.onlinelibrary.wiley.com/doi/10.1002/%28SICI%291097-4571%28199009%2941%3A6%3C391%3A%3AAID-ASI1%3E3.0.CO%3B2-9).
- [S2] scikit-learn. [TruncatedSVD](https://scikit-learn.org/1.8/modules/generated/sklearn.decomposition.TruncatedSVD.html).
- [S3] scikit-learn. [Decomposition](https://scikit-learn.org/1.8/modules/decomposition.html).

## 13. Vị trí trong toàn bộ nghiên cứu

LSA thường là bước biểu diễn trước tìm kiếm, clustering hoặc supervised learning. Khi là bước của một pipeline dự đoán, phải đánh giá toàn pipeline thay vì riêng explained variance. Nếu dùng trục để đặt construct, cần xác thực ý nghĩa và tính bền trên dữ liệu khác, không coi phép phân rã là định nghĩa lý thuyết.

## 14. Kiến thức nền cần biết trước

Cần hiểu ma trận, rank, vector trực giao, singular values và phép chiếu. Phân biệt norm/năng lượng của X với phương sai đã center. Với văn bản, biết TF-IDF và sparse representation. Không cần nghĩ mỗi singular vector là một “chủ đề thật”; đó là đối tượng toán học phục vụ xấp xỉ.

## 15. Nền tảng lý thuyết và logic hình thành

Trong SVD chính xác, giữ các singular values lớn nhất tạo xấp xỉ rank-r tối ưu theo Frobenius norm dưới điều kiện chuẩn. Điều này tối ưu tái tạo ma trận, không tối ưu sẵn macro-F1, coherence hoặc quan hệ lý thuyết. Randomized approximation và cấu hình số học cần được phân biệt với kết quả lý thuyết phân rã chính xác.

## 16. Các thành phần cấu tạo

Thành phần gồm X, rank r, U/S/V, thuật toán SVD, số vòng của randomized method, phép chiếu và các bước chuẩn hóa sau đó. Nếu chuẩn hóa Z theo L2 để tính cosine, ghi rõ vì nó thay hình học downstream, không coi đó là một phần bắt buộc giống nhau trong mọi LSA.

## 17. Các loại và biến thể

Có LSA trên count hoặc TF-IDF, SVD chính xác/randomized và biến thể incremental theo triển khai. Latent Semantic Indexing thường nhấn ứng dụng truy xuất. Đừng đánh đồng mọi embedding nén với LSA hoặc gọi SVD trên feature có nhãn mục tiêu là cùng một thiết kế không giám sát thuần túy.

## 18. Đầu vào và đầu ra

X có n tài liệu, V feature; Z có n×r, components có r×V. Inverse transform tạo xấp xỉ trong không gian feature, có thể âm dù X ban đầu không âm. Dữ liệu mới dùng cùng vocabulary và V; hàng toàn 0 không có thông tin để suy ngữ nghĩa đáng tin.

## 19. Căn cứ lựa chọn

Nên thử khi cần baseline giảm chiều văn bản rẻ và có cấu trúc tuyến tính hữu ích. Chọn r theo nhiệm vụ, độ ổn định, tài nguyên và đánh giá ngoài mẫu khi cần. So với TF-IDF nguyên bản để biết phép nén giúp hay làm mất tín hiệu, thay vì giả định ít chiều luôn tốt hơn.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu cần các thành phần chắc chắn không âm hoặc một phân bố topic xác suất. Không dùng r cao gần chiều gốc chỉ để đạt tái tạo gần hoàn hảo rồi gọi đó là phát hiện nội dung mới. Với nhiệm vụ phụ thuộc từ hiếm/khía cạnh nhỏ, cắt rank có thể bỏ mất tín hiệu cần thiết.

## 21. Các quyết định quan trọng khi thực hiện

Chọn weighting, r, số vòng/seed, có normalize Z không và thước đo downstream. Chốt phạm vi fit và cách căn chỉnh không gian giữa các lượt trước so sánh thời gian. Fit riêng hai LSA trên hai năm không tự cho trục cùng ý nghĩa; cần căn chỉnh hoặc một cơ sở chung có thiết kế rõ.

## 22. Giải thích công thức và ký hiệu

Với $X\in\mathbb R^{n\times V}$:

$$X\approx U_r\Sigma_rV_r^\top,\qquad Z=XV_r\approx U_r\Sigma_r,\qquad \widehat X=ZV_r^\top.$$

Nếu $X=diag(3,1)$ và giữ rank 1, xấp xỉ là $diag(3,0)$, sai số Frobenius bình phương bằng 1. Đồng thời đổi dấu một cột $U_r$ và cột tương ứng $V_r$ không đổi tích. Vì thế dấu không thể được gán trực tiếp là cảm xúc.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mã kiểm tra tái tạo rank 1 và fit LSA thật trên TF-IDF.

```python
import numpy as np
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.decomposition import TruncatedSVD
A = np.diag([3.,1.])
U, s, Vt = np.linalg.svd(A, full_matrices=False)
A1 = (U[:,:1]*s[:1]) @ Vt[:1]
assert np.isclose(np.sum((A-A1)**2), 1.)
docs = ["phòng sạch giường êm", "phòng bẩn giường cũ", "phòng rộng giường đẹp",
        "nhân viên nhiệt tình lễ tân", "nhân viên chậm lễ tân", "nhân viên thân thiện lễ tân",
        "giá cao chi phí đắt", "giá thấp chi phí rẻ", "giá hợp lý chi phí"]
vec = TfidfVectorizer(ngram_range=(1,2))
X = vec.fit_transform(docs)
svd = TruncatedSVD(n_components=3, n_iter=10, random_state=42)
Z = svd.fit_transform(X)
assert Z.shape == (len(docs),3) and np.isfinite(Z).all()
terms = vec.get_feature_names_out()
for k, loading in enumerate(svd.components_):
    print(k, "dương:", terms[np.argsort(loading)[-4:][::-1]].tolist(),
          "âm:", terms[np.argsort(loading)[:4]].tolist())
new = vec.transform(["phòng giường zzholdout"])
assert "zzholdout" not in vec.vocabulary_ and new.nnz > 0
assert svd.transform(new).shape == (1,3)
```

Loading âm là giá trị hệ số, không gán nhãn tiêu cực. Ví dụ chưa có classifier hoặc kiểm định ý nghĩa các chiều.

## 24. Tình huống sai và phản ví dụ

Hai review “phòng sạch” và “phòng bẩn” cùng chủ đề có thể gần trong không gian nén. Một từ chê hiếm có thể bị loại khỏi các thành phần lớn dù cần cho phân loại. Cả hai là ví dụ mục tiêu nén và mục tiêu cảm xúc không hoàn toàn trùng, cần đối chứng downstream.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Kiểm tra sai số tái tạo, explained variance theo định nghĩa và chất lượng nhiệm vụ thực. Với diễn giải, đọc cả hai dấu và review cực trị, đánh giá ổn định không gian thay vì chỉ dấu trục. Biểu đồ 2D có thể mất cấu trúc; không dùng nó làm bằng chứng duy nhất rằng lớp/cụm thật tách biệt.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu mất tín hiệu, thử rank khác trong development, xem lại vocabulary hoặc giữ TF-IDF nguyên bản. Nếu trục khó đọc, có thể dùng NMF/topic model cho câu hỏi mô tả khác, nhưng cần đánh giá lại phù hợp. Không xoay/đổi tên trục chỉ để khớp giả thuyết mà không công bố phép biến đổi.

## 27. Phân biệt kết quả với kết luận

Kết quả là biểu diễn hạng thấp và hiệu quả theo tiêu chí được đo. Không kết luận trục là cấu trúc tâm lý hoặc tác động nhân quả. Mức phương sai giữ lại không bằng mức hiểu cảm xúc. Nếu classifier trên Z tốt hơn, kết luận về lợi ích dự đoán có điều kiện trên protocol đó.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu dùng TF-IDF và TruncatedSVD không centering với r=[...], thuật toán/seed=[...]. Toàn bộ biểu diễn fit trong [phạm vi]. Hiệu quả được đánh giá bằng [...].” Nêu có normalize tọa độ không và phân biệt explained variance với metric classifier. Điền số thật, kèm baseline không giảm chiều.

## 29. Khả năng tái lập và tính minh bạch

Lưu vocabulary/IDF, components, singular values, chuẩn hóa và schema cột; ghi phiên bản/seed. Giữ cùng không gian cho train/deploy. Mã chạy sklearn 1.8.0 trên dữ liệu tự tạo; khi chia sẻ thực nghiệm, công bố split và pipeline để người khác không vô tình fit SVD ngoài fold.

## 30. Đạo đức nghiên cứu liên quan

Không coi nén vector là ẩn danh được bảo đảm; biểu diễn vẫn có thể giữ thông tin nhạy cảm. Bảo vệ review và mô hình theo quyền truy cập phù hợp. Tránh suy đoán nhóm người từ các trục hoặc công khai review cực trị có thể truy ngược danh tính.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Vì sao LSA có dấu âm nhưng không mang nhãn cảm xúc âm?
2. Tính sai số rank 1 của diag(3,1).
3. TruncatedSVD khác PCA center ra sao?
4. Vì sao cần fit trong fold?

Gợi ý: sai số bình phương bằng 1. Bài mở rộng: so classifier trước/sau giảm chiều với cùng split và ngân sách tuning.

## 32. Những điểm còn tranh luận

Các vấn đề còn cần kiểm chứng là khả năng giữ tín hiệu hiếm, độ bền khi chuyển miền và diễn giải trục tuyến tính trong review tiếng Việt. Không có rank tối ưu chung. Nên báo đánh đổi nén, tài nguyên và nhiệm vụ thay vì dùng “ngữ nghĩa tiềm ẩn” như bảo đảm hiểu văn bản.

