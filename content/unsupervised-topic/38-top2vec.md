# Top2Vec — Vector chủ đề trong không gian tài liệu và từ

> **Nhánh:** MACHINE LEARNING → Unsupervised và Topic. **Nguồn:** dòng 39 bảng Excel. **Đối chiếu:** 29/10/2026. Mã NumPy minh họa tâm topic và tìm từ gần trong không gian tự tạo; chưa chạy package top2vec, UMAP, HDBSCAN hoặc học embedding. Mọi ví dụ và dữ liệu trong bài đều tự tạo.

| Trường | Nội dung |
|---|---|
| Vai trò | Khám phá/biểu diễn không dùng nhãn cảm xúc trong ví dụ |
| Bối cảnh | Review khách sạn tiếng Việt; không có benchmark dữ liệu thật |
| Đọc kết quả | Phân biệt cấu trúc học được với nhãn/khái niệm được xác thực |

## 1. Định nghĩa học thuật

Top2Vec là phương pháp topic modeling và semantic search dựa trên các vector tài liệu, từ và topic trong một không gian tương thích. Cách tiếp cận gốc tìm các vùng tài liệu dày đặc, xây vector topic và lấy các từ gần vector đó để mô tả [S1]. Số topic được phát hiện qua pipeline, không cần cố định K ngay từ đầu như LDA chuẩn.

## 2. Giải thích đơn giản

Hình dung review và từ cùng nằm trên một bản đồ ngữ nghĩa. Nhiều review nằm gần nhau tạo một vùng nội dung; một điểm đại diện vùng đó giúp tìm các từ mô tả gần nhất. Điều kiện quan trọng là từ và review thật sự có tọa độ tương thích, không phải hai bộ vector độc lập chỉ tình cờ có cùng số chiều.

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu khách sạn, có thể dùng để khám phá nhóm nội dung, truy xuất review theo topic hoặc tìm tài liệu tương đồng. Cần kiểm tra embedding phù hợp tiếng Việt. Một topic về giá không tự cho sentiment; điểm tương đồng vector không phải xác suất hài lòng hoặc mức quan trọng trong quản trị.

## 4. Điều kiện và giả định sử dụng

Cần đủ dữ liệu cho cấu trúc mật độ/embedding, vocabulary và tokenizer phù hợp. Từ hiếm bị loại theo cấu hình có thể làm thiếu nội dung quan trọng. Tuyên bố không cần stopword list không có nghĩa không cần kiểm tra dữ liệu. Khả năng hoạt động với review ngắn phụ thuộc corpus, encoder và cấu hình, không là bảo đảm.

## 5. Cơ chế hoạt động bên trong

Quy trình gốc/điển hình tạo joint document-word embeddings, giảm chiều document vectors để tìm vùng mật độ, rồi lấy các document vectors liên quan trong không gian ngữ nghĩa gốc để xây topic vectors. Từ gần topic được dùng làm mô tả. Các bước gán tài liệu/tìm kiếm và giảm topic cần được đối chiếu triển khai [S2]. Không gọi phép trung bình tọa độ UMAP 2D là mặc định topic vector gốc.

## 6. Quy trình áp dụng trong nghiên cứu

1. Xác định mục tiêu topic/search và đơn vị tài liệu.
2. Loại trùng, kiểm soát dữ liệu cá nhân, chọn tokenizer/embedding tiếng Việt.
3. Chọn phạm vi fit và cấu hình giảm chiều/mật độ.
4. Fit pipeline native khi có môi trường, kiểm tra số/kích thước topic.
5. Đọc từ gần và review đại diện, kiểm tra nội dung hiếm.
6. Đánh giá ổn định, tìm kiếm và coverage theo mục tiêu.

Ví dụ mã bài chỉ minh họa phần vector với nhóm cho trước, không thực hiện các bước học đầy đủ.

## 7. Ví dụ minh họa xuyên suốt

Các review về phòng có thể tạo vùng gần nhau dù cách viết khác. Vector topic của vùng đó có thể gần “phòng”, “giường”. Nhóm về giá có thể gần “giá”, “chi_phí”. Từ đại diện cần được đối chiếu review vì embedding có thể phản ánh chủ đề nền hoặc ngôn ngữ, không nhất thiết nội dung người nghiên cứu muốn đo.

## 8. Kết quả và cách diễn giải

Topic ID, topic size và cosine score có nghĩa theo API/quy trình gán. Cosine không phải posterior của LDA và không cần tổng trên topic bằng 1. Nếu đếm tài liệu sau gán lại, nêu rõ denominator/quy tắc; đừng mặc định số thành viên cụm mật độ ban đầu bằng số tài liệu được gán cuối cùng.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là gắn topic với semantic search và dùng từ gần trong không gian chung để mô tả. Hạn chế gồm phụ thuộc embedding/mật độ, cỡ corpus và vocabulary, chi phí huấn luyện/tìm kiếm cùng khó xác thực các chủ đề hiếm. “Tự tìm số topic” không có nghĩa không cần hyperparameter hoặc không có quyết định của nhà nghiên cứu.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| BERTopic | Thường mô tả topic bằng c-TF-IDF; Top2Vec gắn với joint word/document/topic vectors |
| LDA | Hỗn hợp topic xác suất theo mô hình sinh |
| K-means | Yêu cầu K và objective tâm khác pipeline mật độ |
| Semantic search | Có thể dùng vectors mà không có bước topic discovery |

So phương pháp cần công bố encoder và mọi lựa chọn, không quy chênh lệch do encoder cho riêng cách tạo topic.

## 11. Sai lầm thường gặp và cách khắc phục

Không ghép embedding từ và tài liệu được học độc lập rồi tính cosine như cùng không gian. Không coi score âm là sentiment tiêu cực. Không nghĩ số topic “tự động” là một hằng số khách quan của corpus. Không bỏ qua tài liệu ban đầu bị xem noise vì bước gán cuối có thể khác; báo từng quy tắc đang sử dụng.

## 12. Tài liệu tham khảo

- [S1] Angelov, D. (2020). *Top2Vec: Distributed Representations of Topics*. [Bài gốc](https://arxiv.org/abs/2008.09470).
- [S2] Top2Vec. [Tài liệu thuật toán và API](https://top2vec.readthedocs.io/en/stable/Top2Vec.html).

Nguồn gốc và API hiện hành không đồng nhất mọi tùy chọn. Mã trong bài không gọi Top2Vec; kiểm tra vector tự tạo không phải benchmark hoặc xác nhận khả năng xử lý tiếng Việt của package.

## 13. Vị trí trong toàn bộ nghiên cứu

Top2Vec nằm ở khám phá nội dung/truy xuất hoặc tạo biến cho phân tích sau. Nếu dùng topic để giải thích rating, cần tách bước khám phá và kiểm định, ghi toàn bộ lựa chọn. Không lấy từ khóa ngữ nghĩa làm định nghĩa construct tự động hoặc coi corpus trực tuyến đại diện toàn bộ khách lưu trú.

## 14. Kiến thức nền cần biết trước

Cần hiểu joint embedding, cosine, chuẩn hóa vector, density clustering và giảm chiều. Phân biệt không gian phát hiện cụm với không gian diễn giải/tìm từ. Biết tokenization tiếng Việt và tác động ngưỡng tần suất; từ đa âm tiết bị tách có thể làm mô tả không giống khái niệm người đọc mong muốn.

## 15. Nền tảng lý thuyết và logic hình thành

Giả định vận hành là các tài liệu tương đồng ngữ nghĩa tập trung thành vùng có thể liên quan topic. Topic vector tóm tắt các tài liệu liên quan và từ gần nó cung cấp mô tả. Một vector trung bình vẫn có thể che câu hỗn hợp hoặc nội dung đối lập; tìm cụm và đặt tên cần bằng chứng ngoài hình học.

## 16. Các thành phần cấu tạo

Thành phần gồm encoder/tokenizer, document vectors, word vectors, reducer, clusterer, topic vectors, vocabulary và index tìm kiếm. Các lựa chọn embedding theo triển khai có thể khác Doc2Vec gốc. Cần quản lý ID/revision và thứ tự tài liệu để không ghép nhầm vector với review.

## 17. Các loại và biến thể

Có các backend embedding khác nhau và tùy chọn giảm/gộp topic theo triển khai. Topic reduction tạo mức tổng hợp, không chứng minh số topic ban đầu sai. Các tính năng contextual/chunking trong phiên bản khác cần kiểm tra và mô tả riêng, không lấy mô tả thuật toán gốc áp cho mọi tùy chọn mới.

## 18. Đầu vào và đầu ra

Đầu vào là văn bản và cấu hình; nếu dùng embedding ngoài, cần bảo đảm tương thích cho từ/tài liệu theo yêu cầu phương pháp. Đầu ra gồm topic vectors, từ đại diện, kích thước và kết quả search/assignment tùy lời gọi. Không căn chỉnh bằng cách chỉ kiểm tra hai ma trận có cùng số cột.

## 19. Căn cứ lựa chọn

Đáng thử khi cần liên kết topic discovery với tìm kiếm semantic và có dữ liệu/embedding phù hợp. Chọn dựa trên nội dung, độ bền, chất lượng truy xuất và tài nguyên. So với baseline đơn giản và BERTopic theo protocol rõ, ghi cả phần encoder để đánh giá công bằng.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu corpus quá nhỏ để tìm cấu trúc mật độ, embedding ngôn ngữ không phù hợp hoặc các từ/tài liệu không thể đưa vào không gian chung đúng cách. Không dùng cosine để ước lượng tỷ lệ hài lòng. Nếu yêu cầu là multi-aspect có nhãn chuẩn, topic discovery chưa đủ trả lời.

## 21. Các quyết định quan trọng khi thực hiện

Chọn encoder, tokenizer, min_count/vocabulary, tham số reducer/mật độ và quy tắc gán/reduction. Chốt cách đếm topic size và xử lý tài liệu ngoài cụm. Nếu đánh giá tương lai, fit phạm vi lịch sử và thiết kế transform/update rõ; fit lại toàn corpus là thay mô hình và cần đánh giá lại.

## 22. Giải thích công thức và ký hiệu

Minh họa với document vectors gốc $d_i$ và nhóm đã có $C_k$:

$$u_k=\frac1{|C_k|}\sum_{i\in C_k}d_i,\quad t_k=\frac{u_k}{\|u_k\|_2},\quad s(t_k,v_w)=\frac{t_k^\top v_w}{\|t_k\|\|v_w\|}.$$

Tâm trung bình/chuẩn hóa ở đây là minh họa nguyên tắc, không toàn bộ pipeline. Cần norm dương. $s$ là cosine similarity trong không gian tương thích, không là probability hay polarity. Đầu ra từ gần phải được đọc cùng tài liệu thay vì chỉ xem danh sách từ.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Tọa độ và nhãn nhóm đều tự tạo. Mã không học embedding, không khám phá mật độ và không gọi top2vec.

```python
import numpy as np

def unit_rows(X):
    norms = np.linalg.norm(X, axis=1, keepdims=True)
    assert np.all(norms > 0)
    return X/norms

doc_vectors = unit_rows(np.array([[1.,0.],[0.9,0.1],[0.,1.],[0.1,0.9]]))
groups = np.array([0,0,1,1])  # giả định đã có nhóm
topic_vectors = unit_rows(np.vstack([doc_vectors[groups == k].mean(axis=0)
                                     for k in range(2)]))
words = np.array(["phòng", "giường", "giá", "chi_phí"])
word_vectors = unit_rows(np.array([[1.,0.],[0.8,0.2],[0.,1.],[0.2,0.8]]))
sim = topic_vectors @ word_vectors.T
assert np.isfinite(sim).all() and np.all(np.abs(sim) <= 1+1e-12)
nearest = np.argsort(-sim, axis=1)[:,:2]
assert set(words[nearest[0]]) == {"phòng", "giường"}
assert set(words[nearest[1]]) == {"giá", "chi_phí"}
for k, indices in enumerate(nearest):
    print(k, words[indices].tolist(), sim[k,indices])
query = unit_rows(np.array([[0.95,0.05]]))
assert int(np.argmax(query @ topic_vectors.T)) == 0
```

Kết quả đúng vì không gian được thiết kế đơn giản; không chứng minh embedding thật sẽ có cùng cấu trúc. Thực nghiệm package native cần môi trường và corpus riêng, chưa được chạy trong bài.

## 24. Tình huống sai và phản ví dụ

Hai vùng tài liệu có thể chia theo tiếng Việt/tiếng Anh thay vì phòng/giá. Một từ gần topic có thể là tên nền tảng, không trải nghiệm. Hai không gian embedding độc lập có thể cho cosine số học hợp lệ nhưng vô nghĩa semantic. Những phản ví dụ này cho thấy cần kiểm tra cả nguồn vector và nội dung.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Kiểm tra ổn định theo seed/lấy mẫu, topic sizes và review đại diện/trái tên. Với semantic search, có đánh giá relevance độc lập thay vì chỉ đọc vài kết quả thuận lợi. Báo vocabulary, từ bị lọc và quy tắc gán tài liệu. Nếu topic reduction, đánh giá cả mất thông tin sau gộp.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu topic không hợp lý, xem encoder/tokenization trước khi ép số topic. Nếu mất nội dung hiếm, kiểm tra ngưỡng tần suất/mật độ trên dữ liệu phát triển. Khi đổi backend, đánh giá lại toàn bộ không gian, không tái sử dụng word vectors cũ tùy ý. Không lấy test/rating làm hướng chỉnh mà vẫn gọi khám phá độc lập.

## 27. Phân biệt kết quả với kết luận

Kết quả là chủ đề/search theo cấu hình và corpus, không phải bộ khía cạnh phổ quát hoặc nhãn cảm xúc. Cosine cao không chứng minh quan hệ nhân quả hay cùng tác giả. Kết luận về liên hệ topic-rating cần một phân tích xác nhận khác với kiểm soát thiết kế và độ bất định.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu dùng Top2Vec với backend [...], tokenizer/vocabulary [...], reducer/clusterer [...]. Topic được kiểm tra qua [...]; quy tắc gán tài liệu/reduction là [...].” Báo model revision, seed, số topic và cách đếm size. Không báo ví dụ vector tự tạo như kết quả huấn luyện Top2Vec.

## 29. Khả năng tái lập và tính minh bạch

Lưu văn bản ID, thứ tự, encoder/revision, vectors/index theo quyền sử dụng, cấu hình clustering và mapping topic/tên. Nếu cập nhật model, giữ lịch sử vì số/ID topic có thể thay. Mã mục 23 chỉ chạy NumPy, chưa kiểm tra package Top2Vec; tái lập native cần script và môi trường thật.

## 30. Đạo đức nghiên cứu liên quan

Tìm kiếm semantic có thể đưa ra review chứa tên hoặc dữ liệu nhạy cảm. Kiểm soát truy cập và tránh công khai kết quả có thể truy ngược. Không dùng topic/vector để gán danh tính, gian lận hoặc đặc điểm nhạy cảm người viết. Review đại diện cần ẩn danh và đánh giá quyền sử dụng.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Vì sao word/document vectors cần không gian tương thích?
2. Topic vector nên được phân biệt với điểm UMAP 2D thế nào?
3. Cosine có tổng topic bằng 1 không?
4. Số topic tự tìm có phụ thuộc cấu hình không?

Gợi ý: có, số topic phụ thuộc pipeline. Bài mở rộng: lập protocol kiểm tra relevance của search và nội dung topic độc lập.

## 32. Những điểm còn tranh luận

Các vấn đề cần thực nghiệm gồm tính bền số topic, biểu diễn từ tiếng Việt và cách đếm thành viên sau gán/reduction. Không có kết quả tự động miễn diễn giải. Cần phân biệt chất lượng vector, chất lượng phân cụm và giá trị của topic đối với câu hỏi nghiên cứu.

