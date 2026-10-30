# BERTopic — Chủ đề từ embedding, clustering và c-TF-IDF

> **Nhánh:** MACHINE LEARNING → Unsupervised và Topic. **Nguồn:** dòng 38 bảng Excel. **Đối chiếu:** 29/10/2026. Mã NumPy/sklearn minh họa c-TF-IDF trên nhóm cho trước; chưa chạy gói bertopic, UMAP hoặc HDBSCAN. Mọi ví dụ và dữ liệu trong bài đều tự tạo.

| Trường | Nội dung |
|---|---|
| Vai trò | Khám phá/biểu diễn không dùng nhãn cảm xúc trong ví dụ |
| Bối cảnh | Review khách sạn tiếng Việt; không có benchmark dữ liệu thật |
| Đọc kết quả | Phân biệt cấu trúc học được với nhãn/khái niệm được xác thực |

## 1. Định nghĩa học thuật

BERTopic là một khung topic modeling kết hợp biểu diễn tài liệu bằng embedding, phân cụm và biểu diễn topic bằng class-based TF-IDF. Pipeline phổ biến dùng giảm chiều rồi clustering theo mật độ; các thành phần có thể thay thế theo cấu hình [S1–S2]. Topic thu được không tự là nhãn cảm xúc và không nhất thiết mỗi review thật chỉ có một khía cạnh.

## 2. Giải thích đơn giản

Trước hết mô hình đặt review vào không gian ngữ nghĩa, gom các review tương tự rồi tìm những từ giúp mô tả mỗi nhóm. Embedding quyết định một phần “gần”; clustering quyết định nhóm; từ khóa là lớp mô tả. Việc đổi danh sách từ mô tả không luôn đổi các review thuộc nhóm, nên cần phân biệt hai lớp thao tác.

## 3. Mục đích và vấn đề giải quyết

Dùng để khám phá nội dung review có thể diễn đạt bằng từ khác nhau và hỗ trợ tổng hợp khía cạnh khách sạn. Cần kiểm tra embedding có phù hợp tiếng Việt/miền dữ liệu. Nếu muốn cảm xúc theo topic, cần thêm bước gán/đánh giá polarity; không gọi topic “phòng” là hài lòng chỉ vì có từ sạch.

## 4. Điều kiện và giả định sử dụng

Cần corpus đủ cấu trúc, embedding phù hợp ngôn ngữ/độ dài, cấu hình giảm chiều/clustering và văn bản cho bộ từ khóa. Không phải embedding nào cũng xử lý tiếng Việt tốt. Dữ liệu ít hoặc cấu hình min_cluster_size không phù hợp có thể cho rất ít topic hoặc nhiều noise; đó không tự là lỗi phần mềm.

## 5. Cơ chế hoạt động bên trong

Pipeline điển hình: encode review, giảm chiều embedding, tìm vùng mật độ để phân cụm, gộp văn bản trong từng nhóm thành class-document, tính c-TF-IDF để xếp từ/cụm mô tả. Có thể tinh chỉnh biểu diễn bằng mô hình khác. BERTopic linh hoạt nên không khẳng định mọi lần chạy bắt buộc đúng một encoder hoặc clustering mặc định [S2].

## 6. Quy trình áp dụng trong nghiên cứu

1. Chốt đơn vị và mục tiêu khám phá/suy rộng.
2. Loại trùng/nhận diện, chọn embedding tiếng Việt phù hợp.
3. Quy định phạm vi fit của reducer, clusterer và vectorizer.
4. Khảo sát cấu hình trên dữ liệu phát triển, ghi noise/topic sizes.
5. Đọc topic words cùng review, kiểm tra tên và trường hợp bất đồng.
6. Đánh giá ổn định, nội dung và cơ chế transform khi cần.

Nếu dùng nhãn/seed/guidance, mô tả đây là thiết kế có hướng dẫn, không gọi hoàn toàn không giám sát.

## 7. Ví dụ minh họa xuyên suốt

Review “giường êm nhưng phòng ồn” có thể vào nhóm phòng, còn “chi phí cao nhưng đáng tiền” vào nhóm giá. Một nhóm có thể chứa khen/chê và câu hỗn hợp. Ví dụ mã cung cấp nhóm theo nội dung đã biết để chỉ tính c-TF-IDF; không phải BERTopic đã tự phát hiện các nhóm đó.

## 8. Kết quả và cách diễn giải

Topic IDs và danh sách từ mô tả không phải tên khái niệm đã được xác thực. Với cấu hình HDBSCAN thường có nhãn noise -1; không coi -1 là cảm xúc tiêu cực hoặc topic quản trị rõ nghĩa. Probability/membership tùy cấu hình không tương đương θ của LDA hoặc tỷ trọng khía cạnh đã gán thủ công.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là tận dụng biểu diễn ngữ cảnh và tách clustering với mô tả từ khóa; dễ thử các thành phần khác nhau. Hạn chế là nhiều cấu hình/phiên bản, chi phí embedding, nhạy reducer/clusterer và khó giải thích toàn bộ. Khả năng đặt tên trôi chảy, nhất là dùng mô hình sinh, không đảm bảo tên phản ánh đúng tài liệu.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| LDA | Mô hình sinh xác suất hỗn hợp topic |
| NMF | Phân rã bảng không âm, không cùng pipeline mật độ |
| Top2Vec | Biểu diễn từ/topic trong không gian chung; cách tạo từ đại diện khác |
| K-means | Có thể là clusterer thay thế, không toàn bộ BERTopic |

Embedding và c-TF-IDF có vai trò khác; c-TF-IDF không phải bản thân bước encode ngữ nghĩa.

## 11. Sai lầm thường gặp và cách khắc phục

Không gọi BERTopic luôn là BERT duy nhất. Không bỏ toàn bộ noise khỏi báo cáo rồi báo chất lượng chỉ trên phần dễ mà không nêu coverage. Không dùng topic probability như độ chắc chắn cảm xúc. Không gộp/relabel topic bằng nhìn rating rồi báo khám phá độc lập với rating. Phải ghi cả các bước can thiệp sau fit.

## 12. Tài liệu tham khảo

- [S1] Grootendorst, M. (2022). *BERTopic: Neural Topic Modeling with a Class-based TF-IDF Procedure*. [Bài gốc](https://arxiv.org/abs/2203.05794).
- [S2] BERTopic. [The Algorithm](https://maartengr.github.io/BERTopic/algorithm/algorithm.html).
- [S3] BERTopic. [c-TF-IDF](https://maartengr.github.io/BERTopic/getting_started/ctfidf/ctfidf.html).

Bài gốc và tài liệu triển khai cần đọc riêng. Mã dưới chỉ kiểm tra một phép tính thành phần với nhóm tự đặt, không phải benchmark package.

## 13. Vị trí trong toàn bộ nghiên cứu

BERTopic nằm ở khám phá topic hoặc xây feature cho bước nghiên cứu khác. Việc nhóm, đặt tên, gộp và liên hệ topic với rating là những bước riêng cần công bố. Không dùng tên topic do người/LLM đặt như bằng chứng độc lập xác nhận khung lý thuyết ban đầu.

## 14. Kiến thức nền cần biết trước

Cần hiểu embedding, cosine/Euclid, giảm chiều, density clustering, noise và bag-of-words. Biết encoder/reducer/vectorizer có thể được học ở phạm vi khác nhau. Chuyển sang biểu đồ 2D có thể làm méo khoảng cách; không coi vị trí trên biểu đồ là toàn bộ bằng chứng phân tách topic.

## 15. Nền tảng lý thuyết và logic hình thành

Ý tưởng là dùng embedding để nhóm diễn đạt tương đồng, rồi dùng thống kê từ theo nhóm để mô tả. Tách hai việc cho phép biểu diễn từ khóa dễ đọc mà không cần từ khóa quyết định duy nhất các nhóm. Tuy nhiên, sai lệch ở embedding hoặc clustering vẫn truyền sang mô tả; từ khóa đẹp không sửa nhóm sai.

## 16. Các thành phần cấu tạo

Các thành phần gồm encoder, reducer, clusterer, vectorizer, c-TF-IDF transformer và representation model tùy chọn. Tên/phiên bản encoder, tokenizer và giới hạn độ dài là một phần cấu hình. Chọn model ngôn ngữ mặc định không đủ mô tả thực nghiệm tiếng Việt; cần lưu định danh/revision và cách xử lý câu dài.

## 17. Các loại và biến thể

Có pipeline unsupervised, guided, supervised/semi-supervised và các phân tích dynamic/hierarchical tùy thiết kế. Các biến thể không có cùng phạm vi thông tin. Nếu rating/nhãn được dùng dẫn hướng clustering, kết quả liên hệ với rating không còn là ngoại kiểm hoàn toàn độc lập; cần thiết kế xác nhận riêng.

## 18. Đầu vào và đầu ra

Đầu vào là văn bản, có thể kèm embeddings đã tạo đúng thứ tự. Đầu ra thường gồm topic assignment, topic words, số/kích thước nhóm và các giá trị bổ sung tùy cấu hình. Nếu cung cấp embeddings bên ngoài, phải bảo đảm hàng khớp review. Transform mẫu mới phụ thuộc các submodels hỗ trợ, không phải fit lại tất cả corpus.

## 19. Căn cứ lựa chọn

Đáng thử khi embedding ngữ cảnh có lý do giúp dữ liệu tiếng Việt và có tài nguyên kiểm tra toàn pipeline. Chọn bằng nội dung, ổn định, coverage/noise và mục tiêu nghiên cứu. So với baseline đơn giản bằng cùng đơn vị và protocol; không mặc định neural topic model luôn vượt LDA/NMF.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu corpus quá nhỏ, không có embedding phù hợp hoặc không đủ nguồn lực ghi/quản lý cấu hình. Không dùng để xác nhận sẵn một taxonomy mà không có kiểm tra độc lập. Nếu mỗi review chứa nhiều khía cạnh, một cluster assignment đơn có thể không phản ánh đủ yêu cầu phân tích.

## 21. Các quyết định quan trọng khi thực hiện

Chọn encoder, chiều reducer, n_neighbors/min_dist nếu dùng UMAP, tham số mật độ nếu dùng HDBSCAN, vocabulary và cách xử lý noise. Chọn mức giảm/gộp topic và quy tắc đặt tên trước diễn giải kết quả. Ghi rõ có cập nhật topic words hay gán lại tài liệu; hai thao tác không tự tương đương.

## 22. Giải thích công thức và ký hiệu

Minh họa một dạng c-TF-IDF không có các tùy chọn BM25/sqrt:

$$s_{ct}=\frac{n_{ct}}{L_c}\log\left(1+\frac{A}{f_t}\right),\quad L_c=\sum_tn_{ct},\quad f_t=\sum_cn_{ct}.$$

$n_{ct}$ là số lần từ t trong văn bản gộp nhóm c; A là độ dài trung bình các class-documents theo quy ước. $f_t$ ở đây là tổng tần suất, không phải số nhóm chứa từ. Điểm này mô tả từ nổi bật, không là xác suất topic hoặc phép tính sentiment. Chi tiết làm tròn/tùy chọn phải đối chiếu phiên bản [S3].

## 23. Ví dụ tính toán hoặc thao tác từng bước

Nhóm ở đây được cho trước, không gọi package bertopic. Mã kiểm tra phép cộng count và xếp từ theo công thức minh họa; không chạy embedding, UMAP hoặc HDBSCAN.

```python
import numpy as np
from sklearn.feature_extraction.text import CountVectorizer
docs = ["phòng sạch giường êm", "phòng bẩn giường cũ", "phòng rộng giường đẹp",
        "nhân viên nhiệt tình lễ tân", "nhân viên chậm lễ tân", "nhân viên thân thiện lễ tân",
        "giá cao chi phí đắt", "giá thấp chi phí rẻ", "giá hợp lý chi phí"]
labels = np.array([0,0,0,1,1,1,2,2,2])  # nhóm tự đặt cho minh họa
vec = CountVectorizer(ngram_range=(1,2))
X = vec.fit_transform(docs)
counts = np.vstack([np.asarray(X[labels == k].sum(axis=0)).ravel()
                    for k in range(3)])
lengths = counts.sum(axis=1)
frequency = counts.sum(axis=0)
assert np.all(lengths > 0) and np.all(frequency > 0)
A = lengths.mean()
scores = (counts/lengths[:,None]) * np.log1p(A/frequency)[None,:]
assert scores.shape == counts.shape
assert np.isfinite(scores).all() and (scores >= 0).all()
assert np.allclose((counts/lengths[:,None]).sum(axis=1), 1)
terms = vec.get_feature_names_out()
for k, row in enumerate(scores):
    print(k, terms[np.argsort(row)[-5:][::-1]].tolist())
```

Không tuyên bố đã fit BERTopic hoặc đo coherence từ ví dụ này. Thực nghiệm native cần cài đúng gói, chọn model phù hợp và lưu môi trường/cấu hình; không có kết quả chạy package trong tài liệu này.

## 24. Tình huống sai và phản ví dụ

Một embedding có thể nhóm review theo ngôn ngữ thay vì trải nghiệm. Một nhóm noise có thể chứa nội dung hiếm quan trọng như sự cố an toàn. Nếu bỏ nhóm đó, báo cáo topic đẹp nhưng bỏ mất vấn đề cần nghiên cứu. Cần đọc và báo noise/coverage, không xem mọi điểm ngoài cụm là dữ liệu vô giá trị.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Kiểm tra ổn định seed/lấy mẫu, kích thước topic, noise và độ nhất quán nội dung qua người đọc. Báo coherence/diversity với cấu hình rõ nếu dùng. Tách đánh giá clustering, từ khóa và tên topic. Nếu dùng representation sinh, kiểm tra groundedness bằng review thật và không để tên vượt bằng chứng.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu nhiều noise, xem embedding/đơn vị văn bản và cấu hình mật độ trên development; không ép gán hết chỉ để đạt coverage đẹp. Nếu top words kém, sửa vectorizer/biểu diễn từ mô tả chưa chắc cần cluster lại. Mỗi thay đổi sau fit phải được ghi và đánh giá tác động, kể cả gộp topic thủ công.

## 27. Phân biệt kết quả với kết luận

Kết quả là các nhóm và mô tả có điều kiện cấu hình/corpus. Không kết luận topic là nguyên nhân rating hoặc tỷ lệ noise bằng tỷ lệ review tiêu cực. Tên do LLM đặt không là nguồn xác nhận độc lập. Nếu dùng topic phân tích khác, công bố toàn bộ quá trình khám phá và kiểm định.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu dùng BERTopic với encoder [...], reducer [...], clusterer [...], vectorizer/c-TF-IDF [...]. Corpus fit gồm [...]; noise được [...]. Topic được đặt/kiểm tra tên bằng [...].” Báo phiên bản, seed, model revision, gộp/gán lại và coverage. Chỉ điền kết quả thực nghiệm native đã chạy.

## 29. Khả năng tái lập và tính minh bạch

Lưu thứ tự review, embeddings khi được phép, định danh encoder, submodels, topic mapping, cấu hình reducer/clusterer/vectorizer và các bước chỉnh sau fit. Mã mục 23 đã chạy NumPy/sklearn, chưa chạy bertopic. Đừng chỉ lưu danh sách từ khóa rồi cho rằng đủ tái tạo toàn hệ thống.

## 30. Đạo đức nghiên cứu liên quan

Embedding dịch vụ ngoài có thể liên quan chuyển dữ liệu; cần chọn cách xử lý phù hợp quyền sử dụng. Không công khai review đại diện chứa thông tin nhận diện. Đừng đặt tên topic mang định kiến nhóm khách hoặc dùng noise để kết luận khách viết không đáng tin. Có người rà soát cho quyết định tác động.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Embedding, clustering và c-TF-IDF có vai trò gì?
2. Noise -1 có nghĩa tiêu cực không?
3. f_t trong công thức là count hay số nhóm?
4. Đổi tên topic có đổi assignment không?

Gợi ý: f_t là tổng count ở công thức bài. Bài mở rộng: thiết kế kiểm tra tên topic bằng mẫu đại diện lẫn review phản ví dụ.

## 32. Những điểm còn tranh luận

Các vấn đề mở là độ bền topic với encoder/reducer, đánh giá nội dung đa ngôn ngữ và sự phụ thuộc vào bước gộp/đặt tên. Không có cấu hình neural bảo đảm đúng khái niệm. Cần phân biệt cảm giác dễ đọc, tính ổn định và giá trị kiểm định của topic trong nghiên cứu.

