# LDA — Latent Dirichlet Allocation cho mô hình chủ đề

> **Nhánh:** MACHINE LEARNING → Unsupervised và Topic. **Nguồn:** dòng 35 bảng Excel. **Đối chiếu:** 29/10/2026. LDA ở đây là Latent Dirichlet Allocation, không phải Linear Discriminant Analysis. Mã chạy sklearn 1.8.0. Mọi ví dụ và dữ liệu trong bài đều tự tạo.

| Trường | Nội dung |
|---|---|
| Vai trò | Khám phá/biểu diễn không dùng nhãn cảm xúc trong ví dụ |
| Bối cảnh | Review khách sạn tiếng Việt; không có benchmark dữ liệu thật |
| Đọc kết quả | Phân biệt cấu trúc học được với nhãn/khái niệm được xác thực |

## 1. Định nghĩa học thuật

Latent Dirichlet Allocation là mô hình xác suất sinh cho dữ liệu rời rạc, thường dùng mô tả mỗi tài liệu như hỗn hợp các topic và mỗi topic như phân bố trên vocabulary. Topic là biến tiềm ẩn học từ đồng xuất hiện từ, không phải nhãn cảm xúc được cung cấp trước [S1]. Một review có thể liên quan nhiều topic.

## 2. Giải thích đơn giản

Hãy tưởng tượng review được viết từ nhiều “túi từ” như phòng, nhân viên và giá. Mỗi review có tỷ lệ sử dụng các túi khác nhau; mỗi vị trí từ chọn một topic rồi chọn từ theo topic đó. Đây là giả định mô hình, không khẳng định khách thật đã viết theo đúng quy trình hoặc chỉ có một trải nghiệm.

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu khách sạn, LDA giúp khám phá nội dung thường được kể và xây các đặc trưng topic để phân tích tiếp. Có thể khảo sát sự khác nhau về topic giữa nhóm review, nhưng cần kiểm soát thiết kế lấy mẫu và độ bất định. Topic “phòng” không cho biết khen/chê; muốn cảm xúc theo khía cạnh cần thêm nhãn/quy trình.

## 4. Điều kiện và giả định sử dụng

Cần corpus đủ đồng xuất hiện, vocabulary hợp lý và ma trận count không âm phù hợp giả định từ rời rạc. Review quá ngắn có ít thông tin để suy hỗn hợp topic. Bag-of-words bỏ thứ tự từ, nên phủ định và ngữ cảnh không được xử lý đầy đủ chỉ bằng LDA. TF-IDF chạy được số học ở một số triển khai không đồng nghĩa đúng mô hình đếm gốc.

## 5. Cơ chế hoạt động bên trong

Mô hình giả định các phân bố topic-từ và document-topic với prior Dirichlet. Trong fit, thuật toán suy luận xấp xỉ các biến ẩn/tham số từ count; sklearn dùng variational inference. Khi transform tài liệu mới, vocabulary và topic-từ đã fit được giữ để suy phân bố topic của tài liệu. Không huấn luyện lại tất cả topic khi chỉ muốn suy cho một review mới.

## 6. Quy trình áp dụng trong nghiên cứu

1. Chọn đơn vị tài liệu, ngôn ngữ và mục tiêu khám phá/suy rộng.
2. Loại trùng, làm sạch có kiểm soát, xây count vocabulary.
3. Chọn phạm vi fit và dữ liệu phát triển nếu đánh giá ngoài mẫu.
4. Khảo sát K, priors, số vòng và seed.
5. Đọc top words cùng review đại diện, kiểm tra topic lặp hoặc khó hiểu.
6. Đánh giá coherence, ổn định và tiêu chí ngoài mẫu khi phù hợp.

Nếu topic làm feature cho classifier, fit vectorizer và LDA bên trong train từng fold.

## 7. Ví dụ minh họa xuyên suốt

Corpus tự tạo gồm review về phòng/giường, nhân viên/lễ tân và giá/chi phí. Mỗi nhóm có cả khen lẫn chê để tránh nhầm topic với polarity. Một review “phòng nhỏ nhưng nhân viên nhiệt tình” có thể có trọng số ở hơn một topic; trọng số không nhất thiết đúng hai nhóm trực giác nếu dữ liệu quá nhỏ hoặc biểu diễn không đủ.

## 8. Kết quả và cách diễn giải

Document-topic distribution trong transform thường được chuẩn hóa tổng 1, nhưng không phải tỷ lệ câu thật nói về từng khía cạnh đã được gán nhãn thủ công. components_ của sklearn là tham số biến phân/pseudocount, cần chuẩn hóa theo hàng nếu muốn đọc như phân bố topic-từ [S2]. Mã số topic có thể đổi giữa seed; tên topic là bước diễn giải.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là có mô hình hỗn hợp tài liệu, phân bố topic-từ để đọc và cách suy topic cho tài liệu mới. Hạn chế gồm phải chọn K, nhạy tiền xử lý/prior, khó với corpus ít hoặc quá ngắn, và bỏ nhiều ngữ cảnh/thứ tự. Topic dễ đọc không tự chứng minh giả định xác suất đúng hoặc phép đo quản trị hợp lệ.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| Linear Discriminant Analysis | Phương pháp có giám sát khác, dù cùng viết LDA |
| K-means | Thường gán một cụm; LDA dùng hỗn hợp topic |
| LSA | Phân rã tuyến tính có dấu, không cùng mô hình Dirichlet |
| NMF | Phân rã không âm, không mặc định là phân bố xác suất |
| BERTopic | Pipeline embedding-clustering và biểu diễn topic khác |

## 11. Sai lầm thường gặp và cách khắc phục

Không gọi K là số cảm xúc. Không gọi components_ trực tiếp là probability nếu chưa chuẩn hóa. Không fit vocabulary/LDA trên test rồi báo như dự đoán chưa thấy corpus. Đừng chọn K theo coherence duy nhất và bỏ đọc review: metric có thể ưu tiên topic nhìn rõ từ khóa nhưng không phù hợp câu hỏi nghiên cứu.

## 12. Tài liệu tham khảo

- [S1] Blei, D. M., Ng, A. Y., & Jordan, M. I. (2003). *Latent Dirichlet Allocation*. JMLR 3, 993–1022. [Bài gốc](https://jmlr.org/papers/v3/blei03a.html).
- [S2] scikit-learn. [LatentDirichletAllocation](https://scikit-learn.org/1.8/modules/generated/sklearn.decomposition.LatentDirichletAllocation.html).
- [S3] scikit-learn. [Decomposition](https://scikit-learn.org/1.8/modules/decomposition.html).

Các công thức là mô hình chuẩn đơn giản hóa; ví dụ khách sạn tự tạo, chưa có kết quả topic thật của người dùng.

## 13. Vị trí trong toàn bộ nghiên cứu

LDA nằm ở khám phá nội dung hoặc xây biến/feature tiềm ẩn. Nếu dùng topic để giải thích rating hoặc ý định quay lại, cần tách bước đo topic khỏi mô hình quan hệ và xem độ bất định. Topic không tự là construct được xác thực; việc đặt tên “chất lượng dịch vụ” cần chứng cứ nội dung và giới hạn rõ.

## 14. Kiến thức nền cần biết trước

Cần hiểu bag-of-words, multinomial/categorical, phân bố Dirichlet, biến ẩn và suy luận xấp xỉ. Với tiếng Việt, cần kiểm tra phân đoạn từ/cụm, giữ dấu và từ phủ định theo mục tiêu. Loại stopwords quá mạnh có thể xóa tín hiệu cảm xúc hoặc nghĩa quan hệ, dù top terms trông sạch hơn.

## 15. Nền tảng lý thuyết và logic hình thành

LDA cho phép nhiều tài liệu chia sẻ các topic nhưng có tỷ lệ riêng. Prior tạo cấu trúc cho hỗn hợp; dữ liệu đồng xuất hiện giúp suy ra phân bố. Quá trình sinh là định nghĩa xác suất của mô hình, không phải dữ liệu quan sát được. Suy luận thường xấp xỉ vì posterior không tính trực tiếp dễ dàng.

## 16. Các thành phần cấu tạo

Thành phần gồm vocabulary, K topic, prior document-topic α, prior topic-word η, các θ của tài liệu, các φ của topic và chỉ số topic ẩn theo vị trí từ. Triển khai còn có batch/online, learning schedule và tiêu chí hội tụ. Những lựa chọn này ảnh hưởng fit chứ không tự quyết định tên topic.

## 17. Các loại và biến thể

Có batch và online variational learning, Gibbs sampling ở triển khai khác, và nhiều mở rộng như topic có cấu trúc thời gian hoặc tương quan. Không gọi mọi phương pháp phát hiện chủ đề là LDA. Chọn inference/biến thể phải được mô tả vì cùng tên K và prior chưa chắc tạo kết quả tương đương.

## 18. Đầu vào và đầu ra

Đầu vào count matrix n×V và cấu hình. Đầu ra document-topic n×K cùng tham số topic-word K×V. Transform dữ liệu mới chỉ dùng vocabulary đã khóa; từ chưa gặp bị bỏ qua theo vectorizer. Với tài liệu toàn từ mới, kết quả có thể chủ yếu theo prior, không là bằng chứng review thật thuộc topic phổ biến.

## 19. Căn cứ lựa chọn

Nên thử khi cần mô hình chủ đề hỗn hợp và có corpus phù hợp bag-of-words. Chọn K bằng nhiều bằng chứng: khả năng diễn giải, ổn định, coherence và tiêu chí ngoài mẫu nếu mục tiêu cần. So với NMF/LSA hoặc embedding-based model trên cùng đơn vị và quy trình đánh giá, không chỉ tên thuật toán.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu dữ liệu quá ngắn/ít mà không có đủ đồng xuất hiện, hoặc nếu nhiệm vụ yêu cầu ngữ cảnh phủ định/mỉa mai chính xác. Không dùng phân bố topic như nhãn cảm xúc mà không xác thực. Nếu cần topic thay đổi theo thời gian, mô hình tĩnh toàn corpus có thể che sự dịch chuyển cần nghiên cứu.

## 21. Các quyết định quan trọng khi thực hiện

Chọn đơn vị tài liệu, vocabulary, min/max document frequency, K, priors, inference, seed và số vòng. Chốt tiêu chí đặt tên/ghép topic trước khi kể câu chuyện. Với nghiên cứu mô tả có thể fit toàn corpus nhưng công bố; với kiểm tra dữ liệu mới hoặc classifier, tách phạm vi fit và không dùng test cho lựa chọn.

## 22. Giải thích công thức và ký hiệu

Với tài liệu d, topic k và vị trí n:

$$\theta_d\sim Dir(\alpha),\quad \phi_k\sim Dir(\eta),\quad z_{dn}\sim Cat(\theta_d),\quad w_{dn}\sim Cat(\phi_{z_{dn}}).$$

Suy xác suất từ: $p(w=v\mid d)=\sum_k\theta_{dk}\phi_{kv}$. Nếu $\theta=(0.7,0.3)$ và xác suất từ “phòng” ở hai topic là $(0.4,0.1)$, giá trị là $0.7\times0.4+0.3\times0.1=0.31$. Các con số giả định chỉ để tính; đó không phải xác suất cảm xúc tích cực.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Ví dụ fit CountVectorizer+LDA thật; K=3 đặt trước để minh họa, không phải kết luận số topic tối ưu.

```python
import numpy as np
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.decomposition import LatentDirichletAllocation
docs = ["phòng sạch giường êm", "phòng bẩn giường cũ", "phòng rộng giường đẹp",
        "nhân viên nhiệt tình lễ tân", "nhân viên chậm lễ tân", "nhân viên thân thiện lễ tân",
        "giá cao chi phí đắt", "giá thấp chi phí rẻ", "giá hợp lý chi phí"]
vec = CountVectorizer(ngram_range=(1,2))
X = vec.fit_transform(docs)
lda = LatentDirichletAllocation(n_components=3, max_iter=30,
                               learning_method="batch", random_state=42)
Theta = lda.fit_transform(X)
Phi = lda.components_ / lda.components_.sum(axis=1, keepdims=True)
assert np.allclose(Theta.sum(axis=1), 1)
assert np.allclose(Phi.sum(axis=1), 1)
assert np.isfinite(Theta).all() and (Theta >= 0).all()
assert np.isclose(np.dot([0.7,0.3], [0.4,0.1]), 0.31)
terms = vec.get_feature_names_out()
for k, row in enumerate(Phi):
    print(k, terms[np.argsort(row)[-5:][::-1]].tolist())
new = vec.transform(["phòng giường zzholdout"])
assert "zzholdout" not in vec.vocabulary_ and new.nnz > 0
assert np.allclose(lda.transform(new).sum(axis=1), 1)
```

Corpus quá nhỏ để đánh giá coherence hay khả năng suy rộng thực tế. Không diễn giải trọng số thành tỷ lệ nội dung đã được người gán nhãn xác nhận.

## 24. Tình huống sai và phản ví dụ

Một topic có “sạch, bẩn, phòng” có thể là nội dung vệ sinh chứ không phải cảm xúc tích cực. Một topic có tên khách sạn và địa danh có thể phản ánh nguồn lấy mẫu. Một review chỉ gồm từ ngoài vocabulary vẫn được transform nhưng thiếu bằng chứng nội dung. Các trường hợp này cần được rà soát trước kết luận.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Kiểm tra top words, review đại diện và review bất đồng tên topic; lặp seed/lấy mẫu và căn chỉnh topic trước so sánh. Nếu báo perplexity, dùng dữ liệu/định nghĩa rõ và không coi thấp hơn tự động dễ hiểu hơn. Coherence phụ thuộc bộ tham chiếu/cấu hình; cần công bố cách tính và đánh giá con người bổ sung.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu topic lặp, kiểm tra K, từ phổ biến và corpus; nếu không ổn định, xem cỡ dữ liệu và priors. Với review ngắn có thể thay đơn vị theo cơ sở nghiên cứu hoặc thử phương pháp khác, nhưng ghép tài liệu làm thay đổi đơn vị phân tích và trọng số. Không chỉnh corpus theo nhãn test để tạo topic mong muốn.

## 27. Phân biệt kết quả với kết luận

Kết quả là phân bố topic và diễn giải được hỗ trợ trong corpus. Không kết luận topic là cấu trúc tâm lý khách hàng hoặc nguyên nhân rating chỉ từ top words. Nếu kiểm định liên hệ topic-rating, mô tả mô hình thống kê, điều kiện và phạm vi; mối liên hệ dự đoán khác với tác động nhân quả.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu dùng LDA với K=[...], count vocabulary [...], priors/inference [...]. Topic được đặt tên qua [...]; chất lượng đánh giá bằng [...].” Báo phạm vi fit, số tài liệu/từ, seed, số vòng và cách đọc tỷ trọng. Không gọi components_ chưa chuẩn hóa là xác suất từ.

## 29. Khả năng tái lập và tính minh bạch

Lưu corpus ID, vocabulary, cấu hình count, model/topic parameters, mapping tên, seed và phiên bản. Nếu dùng topic làm feature, lưu cả protocol split và phép fit trong từng fold. Mã mục 23 chạy sklearn 1.8.0; không có thực nghiệm trên corpus review thật của người dùng.

## 30. Đạo đức nghiên cứu liên quan

Top words hoặc review đại diện có thể lộ thông tin cá nhân và tên nhân viên. Loại nhận diện theo mục đích hợp lệ, kiểm soát quyền truy cập và không công khai nhóm quá nhỏ. Tránh đặt tên topic theo định kiến nhóm khách hoặc suy diễn thái độ cá nhân mà văn bản không đủ chứng minh.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. LDA trong bài khác Linear Discriminant Analysis thế nào?
2. Tính 0.7×0.4+0.3×0.1.
3. Vì sao count phù hợp mô hình sinh hơn TF-IDF?
4. Topic “phòng” có xác định cảm xúc không?

Gợi ý: xác suất từ là 0.31; cảm xúc cần quy trình khác. Bài mở rộng: đọc topic với review trái tên đặt để kiểm tra diễn giải.

## 32. Những điểm còn tranh luận

Các vấn đề mở gồm tính xác thực semantic của topic, ổn định với review ngắn và tác động của gộp tài liệu/tiền xử lý. Không có một K/prior tốt cho mọi miền. Nghiên cứu cần cân bằng predictive fit với khả năng diễn giải và không biến đầu ra mô hình thành sự thật về trải nghiệm khách.

