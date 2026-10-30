# Sentence embedding + Logistic Regression — Phân loại cảm xúc trên vector câu

> **Nhánh:** MACHINE LEARNING → Embedding + bộ phân loại. **Nguồn:** dòng 47 bảng Excel. **Đối chiếu:** 29/10/2026. Tập trung encoder đóng băng và classifier có giám sát. Mã classifier trên vector tự tạo đã chạy; mã tải sentence encoder chỉ kiểm tra cú pháp, chưa thực thi. Dữ liệu/vector/tình huống minh họa đều tự tạo.

| Trường | Nội dung |
|---|---|
| Thiết kế chính | Encoder đóng băng + classifier có giám sát |
| Nhiệm vụ | Phân loại cảm xúc review khách sạn tiếng Việt |
| Giới hạn mã | Tách ví dụ đã chạy với mẫu API chưa thực thi; không có benchmark embeddings thật |

## 1. Định nghĩa học thuật

Phương pháp gồm hai phần: một sentence encoder biến câu/review x thành vector e(x), rồi Logistic Regression học xác suất nhãn từ vector đó. Với encoder đóng băng, chỉ classifier được fit bằng nhãn sentiment trong giai đoạn này. Đây không phải Logistic Regression trên số từ TF-IDF và không mặc định fine-tune toàn encoder [S1–S3].

## 2. Giải thích đơn giản

Hình dung encoder viết một bản tóm tắt bằng d con số cho mỗi review. Logistic Regression học cách kết hợp các con số để phân biệt tích cực/tiêu cực. Encoder có thể đã học từ corpus khác, nhưng khả năng gần nghĩa không tự bảo đảm giữ polarity. “Phòng sạch” và “phòng không sạch” vẫn cần được kiểm tra bằng nhãn và tập thách thức.

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu khách sạn, đây là baseline để kiểm tra embedding pretrained có giúp dự đoán sentiment khi nhãn hạn chế hay không. Nó còn giúp tách giá trị biểu diễn khỏi việc tăng độ phức tạp classifier. Cần so với TF-IDF+LR và cùng embedding+SVM/XGBoost trên cùng split, ngân sách tuning và quyền truy cập dữ liệu.

## 4. Điều kiện và giả định sử dụng

Cần sentence encoder phù hợp tiếng Việt/miền, nhãn rõ và vector cùng schema. Mapping review-vector phải chính xác. Giữ nguyên tokenizer, pooling, truncation, normalization và model revision giữa train/deploy. Một model hỗ trợ đa ngôn ngữ không tự bảo đảm chất lượng sentiment khách sạn; phải kiểm tra ngoài mẫu thay vì suy từ model card.

## 5. Cơ chế hoạt động bên trong

Encode văn bản bằng encoder đã chọn, thu ma trận n×d. LR tối ưu loss nhãn với regularization trên vector; ở dự đoán, encode câu mới theo đúng cách rồi áp linear score và sigmoid/softmax. Nếu chỉ encode độc lập bằng encoder cố định, tính vector test không tự là học từ test. Nhưng fine-tuning, PCA hoặc scaler học tham số vẫn phải nằm trong train.

## 6. Quy trình áp dụng trong nghiên cứu

1. Chốt đơn vị review/câu, codebook và lớp trung tính/hỗn hợp.
2. Loại trùng, chia train/development/test theo khách sạn/thời gian.
3. Chọn encoder và cấu hình bằng dữ liệu phát triển; xác định đóng băng hay fine-tune.
4. Encode theo mapping ID, kiểm tra shape/finite/truncation.
5. Fit mọi biến đổi được học và LR trong train, chọn C/weighting bằng development/CV.
6. Khóa pipeline, đánh giá gold test và calibration khi cần.

Nếu fine-tune encoder bằng nhãn, thực hiện lại trong từng fold tương ứng; không fine-tune trên toàn dữ liệu rồi chỉ cross-validate LR.

## 7. Ví dụ minh họa xuyên suốt

Một review “phòng sạch nhưng nhân viên hỗ trợ chậm” có nhiều khía cạnh. Nhãn toàn văn phải theo codebook, không tự quyết định bằng từ sạch. Encoder tạo một vector chung có thể làm mất thông tin của một đoạn. Các vector ba chiều ở mã kiểm tra là tọa độ tự tạo, không phải output thật từ một model tiếng Việt hay thang đo ba khía cạnh.

## 8. Kết quả và cách diễn giải

predict_proba trả xác suất theo classes_, không phải mức hài lòng chuẩn hóa. decision_function là logit trong thiết kế nhị phân LR đang dùng. Hệ số gắn với chiều embedding, không trực tiếp với từ hoặc khía cạnh. Không gọi hệ số chiều 17 là ảnh hưởng của độ sạch nếu chưa có chứng cứ/thiết kế diễn giải riêng.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là classifier gọn, dễ đối chứng, thường rẻ để fit sau khi cache embedding và có probability API. Hạn chế là phụ thuộc encoder/pooling, có thể mất tín hiệu sentiment, cần nhãn và không mô hình hóa mọi ranh giới phi tuyến trong vector. Tổng chi phí phải tính cả encode, không chỉ fit LR; xác suất vẫn cần calibration audit.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| TF-IDF+LR | Dùng feature từ/cụm sparse; embedding dùng vector dense học trước |
| Encoder fine-tuning end-to-end | Cập nhật encoder theo nhãn, khác thiết kế đóng băng |
| Embedding+SVM | Học margin/loss khác, không tự cho probability như LR |
| Embedding+XGBoost | Học chia theo tọa độ/tương tác qua cây |

Đổi encoder lẫn classifier cùng lúc không cô lập được đóng góp của classifier.

## 11. Sai lầm thường gặp và cách khắc phục

Không dùng model chỉ tốt tiếng Anh rồi mặc định phù hợp tiếng Việt. Không gọi sigmoid của cosine là LR đã học. Không gán tên ý nghĩa cho từng chiều embedding theo hệ số lớn. Không học StandardScaler/PCA trên toàn dữ liệu trước CV; dù encoder cố định, các bước đó vẫn có thể leakage.

## 12. Tài liệu tham khảo

- [S1] Reimers, N., & Gurevych, I. (2019). *Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks*. [Bài gốc](https://aclanthology.org/D19-1410/).
- [S2] Sentence Transformers. [Semantic Textual Similarity](https://sbert.net/docs/sentence_transformer/usage/semantic_textual_similarity.html).
- [S3] scikit-learn. [LogisticRegression 1.8](https://scikit-learn.org/1.8/modules/generated/sklearn.linear_model.LogisticRegression.html).
- [S4] Model card của [paraphrase-multilingual-MiniLM-L12-v2](https://huggingface.co/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2).

Model [S4] chỉ làm ví dụ API; bài không xác nhận nó tối ưu cho sentiment tiếng Việt.

## 13. Vị trí trong toàn bộ nghiên cứu

Đây là pipeline dự đoán sau gán nhãn và chọn nguồn dữ liệu. Embedding là công cụ biểu diễn, không thay xây thang đo hoặc thiết kế lấy mẫu. Nếu dùng nhãn dự đoán tính tỷ lệ tiêu cực theo khách sạn, cần đánh giá sai số theo nhóm và tính đại diện của review trước diễn giải quản trị.

## 14. Kiến thức nền cần biết trước

Cần hiểu encoder pretrained, pooling, dense vectors, LR, regularization, calibration và split. Phân biệt normalize từng vector theo norm với StandardScaler học mean/std theo cột. Cách normalize không nhãn không đồng nghĩa mọi biến đổi đều miễn fit. Biết subword token/truncation có thể ảnh hưởng review dài và phủ định.

## 15. Nền tảng lý thuyết và logic hình thành

Encoder đóng băng chuyển văn bản sang một không gian đã học; LR tìm ranh giới tuyến tính trong không gian đó. Toàn hệ thống không tuyến tính theo văn bản gốc vì encoder có thể phi tuyến. Khả năng tách nhãn tùy thông tin còn trong e(x); classifier không phục hồi được tín hiệu đã mất do cắt câu hoặc pooling.

## 16. Các thành phần cấu tạo

Thành phần gồm văn bản/ID, tokenizer, encoder/revision, pooling, normalization, biến đổi tùy chọn, LR và mapping nhãn. Cache cần gắn với hash văn bản cùng cấu hình encoder. Nếu thay một cấu hình, không tái sử dụng cache cũ chỉ vì số chiều vẫn bằng nhau.

## 17. Các loại và biến thể

Có frozen embeddings, domain-adapted encoder và supervised fine-tuning; có LR nhị phân/đa lớp, weighting và regularization khác. Bài chính dùng frozen encoder và LR nhị phân. Nếu encoder được huấn luyện bằng nhãn sentiment toàn tập thì không còn thiết kế chỉ fit LR và cần báo phạm vi nhãn đó.

## 18. Đầu vào và đầu ra

Train gồm E n×d và y n phần tử; câu mới thành Enew cùng d và cùng cơ sở vector. Có thể có giá trị âm; LR chấp nhận feature số có dấu. Đầu ra là nhãn/score/probability. Không đưa vector từ hai encoder khác nhau vào cùng model chỉ vì đều có d chiều.

## 19. Căn cứ lựa chọn

Đáng thử khi cần baseline classifier đơn giản trên pretrained representation và có nhãn/development đủ kiểm tra. Chọn bằng metric ngoài mẫu, độ bền phủ định/miền và tài nguyên. Cần so TF-IDF+LR để biết lợi ích embedding, không chỉ so những classifier cùng dùng một embedding có thể chưa phù hợp.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu encoder không giữ tốt ngôn ngữ/khía cạnh, nếu dữ liệu bị truncation mất đoạn chê hoặc nếu cần giải thích trực tiếp bằng feature từ. Với mục tiêu nhân quả, LR trên embedding không tự thành mô hình nhận diện tác động dù tên có chữ regression.

## 21. Các quyết định quan trọng khi thực hiện

Chọn encoder/revision, độ dài, đơn vị câu/review, pooling và normalize. Chọn C, class_weight, solver và threshold bằng development. Với review dài, chia đoạn/gộp vector thay đổi đơn vị và weighting, cần báo rõ. Nếu cache embeddings, xác định quy trình đảm bảo train/deploy cùng encoder và thứ tự mẫu.

## 22. Giải thích công thức và ký hiệu

Với $e(x)\in\mathbb R^d$:

$$s=w^\top e(x)+b,\qquad p(y=1\mid x)=\sigma(s)=\frac1{1+e^{-s}}.$$

Loss nhị phân là cross-entropy cộng regularization theo quy ước solver. Trong sklearn, C điều chỉnh nghịch độ mạnh regularization; không nhầm C là trọng số lớp. Nếu $w=(1,-0.5)$, e=(0.8,0.2), b=-0.1 thì s=0.6, p≈0.6457. Hai tọa độ ví dụ không được đặt tên khía cạnh từ phép tính này.

## 23. Ví dụ tính toán hoặc thao tác từng bước

**A. Đã chạy kiểm tra:** classifier thật trên vector tự tạo, không phải sentence embeddings trích xuất thật.

```python
# DA_KIEM_TRA
import numpy as np
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import Normalizer
from sklearn.linear_model import LogisticRegression
Xtrain = np.array([[1,.8,.1],[.9,.7,0],[1,.6,-.1],[.8,.9,.2],
                   [-1,-.8,.1],[-.9,-.7,0],[-1,-.6,-.1],[-.8,-.9,.2]], dtype=float)
ytrain = np.array([1,1,1,1,0,0,0,0])
Xnew = np.array([[.95,.75,0],[-.95,-.75,0]], dtype=float)
pipe = make_pipeline(Normalizer(norm="l2"),
                     LogisticRegression(C=1.0, max_iter=1000, random_state=42))
pipe.fit(Xtrain,ytrain)
P = pipe.predict_proba(Xnew)
scores = pipe.decision_function(Xnew)
positive_col = list(pipe[-1].classes_).index(1)
assert np.isfinite(P).all() and np.allclose(P.sum(axis=1),1)
assert np.allclose(P[:,positive_col],1/(1+np.exp(-scores)))
assert np.isclose(1/(1+np.exp(-0.6)),0.6456563062)
print("Nhãn trên vector minh họa:", pipe.predict(Xnew))
```

**B. Mẫu API chưa thực thi:** cần cài sentence-transformers và tải weights; model ví dụ không là khuyến nghị hiệu năng. Chưa có bước tuning hoặc gold test trong đoạn này.

```python
# MINH_HOA_API_CHUA_CHAY
import numpy as np
from sentence_transformers import SentenceTransformer
from sklearn.linear_model import LogisticRegression
train_texts = ["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
               "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
               "dịch vụ tệ bẩn", "khách sạn bẩn tệ"]
ytrain = np.array([1,1,1,1,0,0,0,0])
new_texts = ["phòng không sạch", "nhân viên hỗ trợ nhiệt tình"]
encoder = SentenceTransformer("sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2")
Etrain = encoder.encode(train_texts, convert_to_numpy=True,
                       normalize_embeddings=True, show_progress_bar=False)
Enew = encoder.encode(new_texts, convert_to_numpy=True,
                     normalize_embeddings=True, show_progress_bar=False)
clf = LogisticRegression(C=1.0, max_iter=1000, random_state=42)
clf.fit(Etrain,ytrain)
print(clf.predict(Enew))
```

Khi triển khai thật, ghi model revision và kiểm tra license/requirements theo phiên bản. Không đưa chuỗi thô vào clf cuối: nó nhận vector E. Ví dụ B không fine-tune encoder và chưa đo chất lượng sentiment.

## 24. Tình huống sai và phản ví dụ

Một encoder semantic có thể đặt câu phủ định gần câu khẳng định; LR không bảo đảm tách nếu tín hiệu yếu. Hai review cùng khách sạn có thể giống vì tên riêng và làm test ngẫu nhiên lạc quan. Một review dài bị cắt mất đoạn chê tạo lỗi không thể chữa chỉ bằng C.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đánh giá macro-F1 và metric từng lớp trên gold test, thêm calibration nếu dùng xác suất. Kiểm tra phủ định, câu hỗn hợp, review dài, nhóm khách sạn và thời gian. So encoder/normalization bằng development, lặp split phù hợp. Không dùng test để chọn model ID hoặc template chỉ vì không fit classifier ở bước chọn đó.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu kém baseline, kiểm tra encoder/độ dài/nhãn trước tuning LR. Có thể dùng biểu diễn khác, giữ đoạn văn hoặc fine-tune đúng phạm vi train nếu dữ liệu cho phép. Nếu đổi encoder, phải tạo lại cache và refit classifier. Nếu xác suất lệch, xem calibration/weighting trên development, không sửa threshold bằng test.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ mô tả chất lượng pipeline trên tập giữ lại và điều kiện encoder. Không suy hệ số vector là nguyên nhân hài lòng hay một khía cạnh cụ thể. Không suy pretrained multilingual nghĩa là không có thiên lệch tiếng Việt. Tính dự đoán tốt khác với phép đo construct hoặc quan hệ nhân quả.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Review được encode bằng [...revision...] với [pooling/normalization/length]. Encoder được [đóng băng/fine-tune theo protocol]. LR với C=[...] được chọn bằng [...]; test giữ theo [...].” Báo số nhãn, weighting, threshold, nguồn pretrained và đối chứng TF-IDF. Chỉ điền kết quả thực đo, không dùng vector giả làm embedding benchmark.

## 29. Khả năng tái lập và tính minh bạch

Lưu IDs/hash văn bản, encoder/revision, tokenizer/pooling, cache manifest, preprocessing và classifier. Lưu mapping lớp, seed, split và phiên bản. Mã A chạy sklearn 1.8.0; mã B chỉ kiểm tra cú pháp vì môi trường chưa có sentence_transformers/weights. Không tuyên bố đã chạy trích xuất embedding thật.

## 30. Đạo đức nghiên cứu liên quan

Embedding không tự ẩn danh hóa review; dịch vụ encode ngoài có thể liên quan chuyển dữ liệu. Chọn quyền sử dụng và truy cập phù hợp, bảo vệ văn bản/caches. Không dùng output tự động làm căn cứ duy nhất để đánh giá nhân viên; kiểm tra lỗi theo bối cảnh có dữ liệu hợp lệ và có người rà soát.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Encoder đóng băng khác fine-tuning thế nào?
2. Vì sao hệ số embedding không là hệ số từ?
3. Khi s=0.6, p bao nhiêu?
4. Encode test bằng encoder cố định khác fit PCA trên test ra sao?

Gợi ý: p≈0.6457. Bài mở rộng: so TF-IDF+LR và embedding+LR cùng split, báo compute encode lẫn classifier.

## 32. Những điểm còn tranh luận

Các vấn đề cần kiểm chứng là tính phù hợp semantic embedding cho sentiment, truncation, domain shift và calibration. Không có encoder/C tối ưu chung. Nghiên cứu nên tách lợi ích biểu diễn với classifier và báo giới hạn nguồn pretrained thay vì dùng tên model như bảo đảm chất lượng.

