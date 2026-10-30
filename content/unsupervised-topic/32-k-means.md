# K-means — Phân cụm đánh giá theo tâm

> **Nhánh:** MACHINE LEARNING → Unsupervised và Topic. **Nguồn:** dòng 33 bảng Excel. **Đối chiếu:** 29/10/2026. Cụm là nhóm theo biểu diễn, không phải nhãn cảm xúc. Mã chạy sklearn 1.8.0. Mọi ví dụ và dữ liệu trong bài đều tự tạo.

| Trường | Nội dung |
|---|---|
| Vai trò | Khám phá/biểu diễn không dùng nhãn cảm xúc trong ví dụ |
| Bối cảnh | Review khách sạn tiếng Việt; không có benchmark dữ liệu thật |
| Đọc kết quả | Phân biệt cấu trúc học được với nhãn/khái niệm được xác thực |

## 1. Định nghĩa học thuật

K-means phân hoạch các vector thành K cụm bằng cách giảm tổng khoảng cách Euclid bình phương từ mẫu đến tâm cụm. Tâm là trung bình vector của các mẫu được gán vào cụm. Đây là phân cụm không giám sát; nhãn tích cực/tiêu cực không tham gia mục tiêu tối ưu. K là số cụm, khác số láng giềng của KNN.

## 2. Giải thích đơn giản

Hình dung xếp review thành các nhóm và đặt một điểm đại diện cho mỗi nhóm. Mỗi review về nhóm có tâm gần nhất, rồi tâm được tính lại. Việc lặp có thể tạo nhóm nói về phòng, nhân viên hoặc giá. Những nhóm này có thể chứa cả lời khen lẫn chê vì cùng chủ đề chưa phải cùng cảm xúc.

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu khách sạn, K-means có thể khám phá các nhóm nội dung, tạo phân đoạn review hoặc làm baseline kiểm tra biểu diễn. Câu hỏi là các nhóm có nhất quán, hữu ích và ổn định không. Nếu mục tiêu là đo tỷ lệ tiêu cực, cần một quy trình gán/kiểm định nhãn khác; không tự coi cụm số 0 là tiêu cực.

## 4. Điều kiện và giả định sử dụng

Cần vector số hữu hạn, chọn K hợp lệ và khoảng cách phù hợp biểu diễn. Mục tiêu ưu tiên các vùng tương đối gọn theo Euclid; dữ liệu hình cong, mật độ khác nhau hay nhiều outlier có thể không phù hợp. Feature số khác đơn vị cần xử lý thang đo trong phạm vi dữ liệu fit. Văn bản thô cần vector hóa.

## 5. Cơ chế hoạt động bên trong

Với Lloyd-style K-means: khởi tạo tâm, gán mỗi mẫu vào tâm gần, cập nhật từng tâm bằng trung bình, lặp đến điều kiện dừng. Tối ưu tham lam có thể đến nghiệm cục bộ, nên khởi tạo và số lượt chạy quan trọng. K-means++ hỗ trợ chọn tâm ban đầu nhưng không bảo đảm nghiệm tối ưu toàn cục [S1–S2].

## 6. Quy trình áp dụng trong nghiên cứu

1. Chốt đơn vị review/câu/khía cạnh và mục tiêu khám phá hay dự đoán dữ liệu mới.
2. Loại trùng, bảo vệ thông tin và xây biểu diễn.
3. Chọn phạm vi fit: toàn corpus để mô tả, hoặc train riêng nếu đánh giá suy rộng.
4. Khảo sát K và cấu hình trên dữ liệu phát triển.
5. Đọc từ đại diện, review gần tâm và các trường hợp ranh giới.
6. Kiểm tra ổn định và giới hạn trước khi đặt tên cụm.

Nếu dùng cụm làm feature cho classifier, fit toàn bộ clustering bên trong train từng fold.

## 7. Ví dụ minh họa xuyên suốt

Bốn điểm giả định (0,0),(0,2),(8,0),(8,2), K=2 có hai tâm (0,1),(8,1) ở cách chia tự nhiên. Mỗi điểm cách tâm bình phương 1, tổng bằng 4. Tọa độ chỉ để tính toán, không phải số từ khen/chê. Ví dụ văn bản mục 23 có ba nội dung phòng, nhân viên, giá và mỗi nội dung có cả khen/chê.

## 8. Kết quả và cách diễn giải

labels_ là mã cụm tùy ý; hoán đổi số 0 và 1 không đổi chất lượng phân hoạch. cluster_centers_ là tâm trong không gian vector, không nhất thiết tương ứng một review có thật. inertia_ là tổng sai số nội cụm của dữ liệu fit, không phải accuracy và không phải thước đo đầy đủ của tính hợp lý quản trị.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là quy tắc rõ, có tâm phục vụ mô tả và có thể gán dữ liệu mới theo tâm đã học. Hạn chế gồm cần K trước, nhạy khởi tạo/thang đo/outlier và không diễn tả tốt mọi hình dạng cụm. Tâm không phải một câu ngôn ngữ tự nhiên; top terms chỉ hỗ trợ diễn giải, không xác nhận tính hợp lệ của tên cụm.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| KNN | KNN dùng nhãn láng giềng để phân loại |
| Hierarchical | Xây cấu trúc gộp/tách nhiều mức, không chỉ một phân hoạch tâm |
| LDA | Mỗi tài liệu có phân bố topic, không bắt buộc chỉ một cụm |
| Spherical K-means | Dùng hình học chuẩn hóa/cosine với cập nhật riêng |

K-means thường trên TF-IDF chuẩn hóa L2 không tự biến thành spherical K-means: tâm trung bình không mặc định được chuẩn hóa lại.

## 11. Sai lầm thường gặp và cách khắc phục

Không chọn K=3 chỉ vì có ba nhãn cảm xúc mong muốn rồi báo phát hiện ba cảm xúc. Không chọn K bằng inertia nhỏ nhất đơn thuần: inertia thường giảm khi tăng K. Đừng fit TF-IDF trên test trong một đánh giá inductive. Đừng so trực tiếp nhãn cụm giữa seed bằng equality; cần chỉ số bất biến hoán vị hoặc căn chỉnh cụm.

## 12. Tài liệu tham khảo

- [S1] Lloyd, S. P. (1982). *Least Squares Quantization in PCM*. IEEE Transactions on Information Theory. [Bài gốc](https://web.stanford.edu/class/ee398a/handouts/papers/Lloyd%20-%20Least%20Squares%20Q%20in%20PCM.pdf).
- [S2] scikit-learn. [Clustering](https://scikit-learn.org/1.8/modules/clustering.html).
- [S3] scikit-learn. [KMeans](https://scikit-learn.org/1.8/modules/generated/sklearn.cluster.KMeans.html).

Đây là các mốc tham khảo, không gán mọi ý tưởng K-means cho duy nhất một năm. Ví dụ và gợi ý protocol trong bài tự soạn.

## 13. Vị trí trong toàn bộ nghiên cứu

K-means thường nằm ở khám phá dữ liệu hoặc xây biểu diễn. Nếu cụm được dùng làm biến cho phân tích sau, cần giải thích cách tạo, độ ổn định và phạm vi dữ liệu đã tham gia. Không coi các cụm như những nhóm khách tồn tại khách quan trước phép phân tích chỉ vì thuật toán trả đúng K nhóm.

## 14. Kiến thức nền cần biết trước

Cần hiểu vector, trung bình, khoảng cách Euclid, bình phương sai số và ảnh hưởng chuẩn hóa. Với review tiếng Việt, biết vocabulary/TF-IDF và phân đoạn từ. TfidfVectorizer mặc định không phải bộ tách từ ngữ nghĩa tiếng Việt; giữ dấu, phủ định và cụm từ cần lựa chọn có chủ đích, đánh giá trên nội dung thật.

## 15. Nền tảng lý thuyết và logic hình thành

K-means xen kẽ hai bước tối ưu: khi tâm cố định, gán gần nhất giảm mục tiêu; khi gán cố định, trung bình giảm tổng bình phương khoảng cách. Điều đó không cho phép tìm mọi phân hoạch tối ưu toàn cục. Vì thế lời giải hội tụ có thể khác theo seed, và một lời giải số học tốt có thể khó diễn giải về nội dung.

## 16. Các thành phần cấu tạo

Thành phần gồm bộ biểu diễn, K, cách khởi tạo, metric/objective, số khởi tạo, tiêu chí hội tụ, tâm và quy tắc gán. Phân biệt max_iter của một lượt với n_init là số lượt khởi tạo. Nếu thêm giảm chiều, nó trở thành bước học cần được kiểm soát phạm vi fit và lưu cùng mô hình.

## 17. Các loại và biến thể

Có batch K-means, MiniBatchKMeans và các biến thể với hình học/ràng buộc khác. Mini-batch sử dụng các lô để cập nhật, đánh đổi chi phí với độ chính xác/ổn định. K-medoids dùng điểm đại diện thật và objective khác; không gọi hai phương pháp tương đương chỉ vì đều có đại diện cụm.

## 18. Đầu vào và đầu ra

Đầu vào X có n hàng và d feature cùng schema. Đầu ra gồm mã cụm, tâm và khoảng cách đến tâm theo API. predict trên câu mới giữ tâm cố định; fit lại trên dữ liệu mới sẽ đổi hệ thống. Vector toàn 0 có thể bị gán theo quy tắc số học dù không có thông tin nội dung, cần phát hiện riêng.

## 19. Căn cứ lựa chọn

Nên thử khi cần một baseline phân hoạch gọn, dữ liệu có thể biểu diễn phù hợp và có lý do chọn mức chi tiết K. Chọn bằng kết hợp độ gọn, ổn định và kiểm tra chuyên gia, không chỉ một biểu đồ elbow đẹp. So các biểu diễn/cấu hình trong protocol đã định để tránh chọn cụm hợp câu chuyện mong muốn.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu cần mỗi review có nhiều topic đồng thời, nếu hình dạng dữ liệu rất không đều hoặc nếu yêu cầu phát hiện noise riêng. K-means cơ bản ép mọi điểm vào một cụm. Nếu câu hỏi là tác động nhân quả của chất lượng phòng, một phân hoạch review không tạo ra thiết kế nhận diện nhân quả.

## 21. Các quyết định quan trọng khi thực hiện

Chọn đơn vị, biểu diễn, K, n_init, seed, chuẩn hóa và xử lý outlier. Chốt trước tên cụm do người đọc đặt theo bằng chứng nào. Nếu mục tiêu là mô tả corpus hiện có, có thể fit toàn corpus nhưng công bố rõ; nếu kiểm tra dữ liệu mới, giữ train và đánh giá riêng thay vì thay mục tiêu sau khi xem kết quả.

## 22. Giải thích công thức và ký hiệu

Với các cụm $C_k$ và tâm $\mu_k$:

$$J=\sum_{k=1}^{K}\sum_{i\in C_k}\|x_i-\mu_k\|_2^2,\qquad \mu_k=\frac1{|C_k|}\sum_{i\in C_k}x_i.$$

Gán mới $c_i=\arg\min_k\|x_i-\mu_k\|^2$. Với bốn điểm mục 7, $J=4$. Không biến khoảng cách thành xác suất bằng cách chuẩn hóa tùy ý rồi gọi đó là posterior của K-means. Thuật toán không định nghĩa mô hình xác suất như vậy.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mã chạy hai phép minh họa, không có nhãn cảm xúc để fit hoặc chọn cụm.

```python
import numpy as np
from sklearn.cluster import KMeans
from sklearn.feature_extraction.text import TfidfVectorizer
Xsmall = np.array([[0.,0.],[0.,2.],[8.,0.],[8.,2.]])
small = KMeans(n_clusters=2, n_init=20, random_state=42).fit(Xsmall)
assert np.isclose(small.inertia_, 4.)
assert np.allclose(np.sort(small.cluster_centers_, axis=0), [[0,1],[8,1]])
docs = ["phòng sạch giường êm", "phòng bẩn giường cũ", "phòng rộng giường đẹp",
        "nhân viên nhiệt tình lễ tân", "nhân viên chậm lễ tân", "nhân viên thân thiện lễ tân",
        "giá cao chi phí đắt", "giá thấp chi phí rẻ", "giá hợp lý chi phí"]
vec = TfidfVectorizer(ngram_range=(1,2))
X = vec.fit_transform(docs)
model = KMeans(n_clusters=3, n_init=20, random_state=42).fit(X)
assert len(model.labels_) == len(docs)
terms = vec.get_feature_names_out()
for k, center in enumerate(model.cluster_centers_):
    print(k, terms[np.argsort(center)[-5:][::-1]].tolist())
new = vec.transform(["phòng giường zzholdout"])
assert "zzholdout" not in vec.vocabulary_
assert new.nnz > 0
print("Cụm cho câu mới:", model.predict(new))
```

Mã cụm in ra cần được đọc cùng review gốc. Không có kiểm định trong ví dụ rằng ba nhóm tự động là ba khái niệm nghiên cứu hợp lệ.

## 24. Tình huống sai và phản ví dụ

Phản ví dụ: tên khách sạn xuất hiện dày và làm các review chia theo khách sạn, không theo trải nghiệm. Một cụm có nhiều từ “phòng” có thể chứa “sạch” lẫn “bẩn”. Đặt tên nó là “hài lòng” sẽ sai nội dung. Rà soát mẫu xa tâm và trường hợp trái tên cụm, không chỉ chọn câu minh họa thuận lợi.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Kiểm tra ổn định theo seed/lấy mẫu, silhouette khi số cụm/số mẫu hợp lệ, phân bố kích thước và đọc mẫu theo cụm. Nếu có nhãn ngoài dùng làm tiêu chí, ghi nó là ngoại kiểm và tránh dùng cùng nhãn để chọn rồi đánh giá. Biểu đồ 2D chỉ là phép chiếu, không chứng minh cấu trúc cao chiều tách biệt.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu cụm bị chi phối tên riêng hoặc từ phổ biến, kiểm tra biểu diễn và mục tiêu trước khi tăng K. Nếu không ổn định, thử nhiều khởi tạo, dữ liệu đủ lớn hoặc phương pháp khác trên dữ liệu phát triển. Nếu không có cấu trúc đáng tin, báo kết quả hạn chế thay vì ép tên cụm theo lý thuyết.

## 27. Phân biệt kết quả với kết luận

Kết quả là một phân hoạch theo dữ liệu, biểu diễn, K và cấu hình cụ thể. Kết luận có thể là những nhóm nội dung hữu ích trong corpus khảo sát. Không kết luận phát hiện ba loại khách phổ quát hoặc tỷ lệ hài lòng nếu chưa có quy trình ánh xạ/kiểm định độc lập và đánh giá tính đại diện.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu phân cụm [đơn vị] bằng K-means trên [biểu diễn]. K được chọn theo [tiêu chí kết hợp], n_init=[...], seed=[...]. Tên cụm được đặt sau khi đọc [quy trình] và kiểm tra độ ổn định.” Báo phạm vi fit, kích thước cụm, từ/review đại diện và ví dụ bất đồng; điền số thật.

## 29. Khả năng tái lập và tính minh bạch

Lưu corpus ID, quy tắc làm sạch, vocabulary/IDF, giảm chiều nếu có, K, tâm, nhãn gán, seed và phiên bản. Giữ mapping giữa mã cụm và tên người đặt, vì số cụm có thể đổi giữa lượt. Mã mục 23 đã chạy trên dữ liệu tự tạo; tái lập nghiên cứu cần dữ liệu được phép chia sẻ.

## 30. Đạo đức nghiên cứu liên quan

Không gán tên mang định kiến cho nhóm review hoặc suy đoán danh tính người viết từ cụm. Bảo vệ thông tin nhận diện trong review đại diện và cân nhắc khả năng truy ngược câu trích. Những cụm nhỏ có thể làm lộ trường hợp cá nhân; công bố đủ bằng chứng nội dung nhưng kiểm soát quyền truy cập.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Vì sao KNN và K-means khác nhiệm vụ?
2. Tính tâm và J cho bốn điểm mục 7.
3. Vì sao inertia nhỏ hơn chưa đủ chọn K?
4. Cụm “phòng” có thể gồm cả cảm xúc nào?

Gợi ý: J=4; inertia giảm khi tăng số cụm. Bài mở rộng: đánh giá ổn định theo seed bằng ARI thay vì so trực tiếp mã cụm.

## 32. Những điểm còn tranh luận

Các vấn đề cần kiểm chứng là mức ổn định của cụm trên biểu diễn tiếng Việt, vai trò chuẩn hóa/giảm chiều và cách xác thực tên cụm. Không có K tối ưu chung. Tính dễ đọc của top terms và tính hợp lệ của kết luận nghiên cứu là hai yêu cầu riêng, cần bằng chứng ở cả hai.

