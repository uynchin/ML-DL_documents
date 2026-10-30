# Sentence embedding + SVM — Học ranh giới margin trên vector câu

> **Nhánh:** MACHINE LEARNING → Embedding + bộ phân loại. **Nguồn:** dòng 48 bảng Excel. **Đối chiếu:** 29/10/2026. Bài phân biệt LinearSVC với SVC kernel. Mã LinearSVC trên vector tự tạo đã chạy; mẫu sentence encoder chưa thực thi. Dữ liệu/vector/tình huống minh họa đều tự tạo.

| Trường | Nội dung |
|---|---|
| Thiết kế chính | Encoder đóng băng + classifier có giám sát |
| Nhiệm vụ | Phân loại cảm xúc review khách sạn tiếng Việt |
| Giới hạn mã | Tách ví dụ đã chạy với mẫu API chưa thực thi; không có benchmark embeddings thật |

## 1. Định nghĩa học thuật

Pipeline dùng sentence encoder biến văn bản thành vector rồi Support Vector Machine học ranh giới phân loại từ vector và nhãn. Linear SVM tìm ranh giới tuyến tính có margin theo loss/regularization; kernel SVM cho phép cấu trúc khác qua kernel. Bài minh họa LinearSVC với squared hinge, không đồng nhất nó với SVC(kernel="linear") ở mọi chi tiết [S1–S2].

## 2. Giải thích đơn giản

Encoder biến review thành điểm trong không gian số. SVM học một đường/mặt phân chia, cân bằng độ rộng margin và việc vi phạm nhãn. Nó không trực tiếp đọc từ khóa. Nếu encoder đặt câu trái cảm xúc gần nhau, việc tìm mặt phân chia khó hơn; gọi nó là SVM mạnh không tự tạo thông tin bị mất.

## 3. Mục đích và vấn đề giải quyết

Có thể dùng như baseline margin classifier trên embeddings và so LR/XGBoost cùng encoder. Trong nghiên cứu khách sạn, câu hỏi là ranh giới này có cải thiện lớp tiêu cực hoặc độ bền chuyển khách sạn không. So với TF-IDF+SVM để tách giá trị của embedding khỏi giá trị của loss/regularization.

## 4. Điều kiện và giả định sử dụng

Cần vector số hữu hạn, nhãn rõ và biểu diễn thống nhất. Normalization/scaling ảnh hưởng margin và kernel; lựa chọn phải được kiểm tra trong development. Với kernel RBF, chi phí học/tìm tham số có thể cao khi n lớn. Mô hình hỗ trợ đa ngôn ngữ không tự bảo đảm encoder tách sentiment tốt ở tiếng Việt.

## 5. Cơ chế hoạt động bên trong

Với LinearSVC, tối ưu hệ số tuyến tính và bias theo squared hinge/L2 trong cấu hình ví dụ. Dự đoán dùng dấu/argmax decision scores theo mapping lớp. Với kernel SVC, hàm quyết định được xây từ support vectors và kernel. Dù cùng SVM, các implementation có solver, multiclass, loss và probability support khác nhau.

## 6. Quy trình áp dụng trong nghiên cứu

1. Chốt codebook, đơn vị và group/time split.
2. Chọn encoder và quyền truy cập/fine-tuning rõ ràng.
3. Encode train/development/test bằng cùng cơ sở, giữ ID alignment.
4. Fit normalization/scaler học tham số và SVM đúng phạm vi train.
5. Chọn C, kernel/gamma nếu có, weighting bằng development/CV.
6. Nếu cần xác suất, thiết kế calibration trong train/development phù hợp.
7. Khóa pipeline rồi đánh giá gold test.

Không fine-tune encoder trên toàn gold data trước CV của SVM.

## 7. Ví dụ minh họa xuyên suốt

Một review có thể vừa khen giường vừa chê dịch vụ. Nhãn toàn văn cần theo hướng dẫn gán, không do margin tự định nghĩa. Vector ba chiều của mã A là fixture tự tạo nhằm kiểm tra LinearSVC, không phải câu đã được encode thật hoặc ba khía cạnh khách sạn được xác thực.

## 8. Kết quả và cách diễn giải

decision_function của LinearSVC là score theo hàm quyết định, không phải xác suất đúng hoặc probability tích cực. Trong nhị phân, dấu tương ứng classes_ theo API; độ lớn score chưa là khoảng cách hình học chuẩn hóa nếu chưa chia norm w. Không dùng sigmoid(score) tùy ý rồi gọi đó là calibrated probability.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là margin classifier gọn với embedding cố định, có regularization và baseline rõ. Hạn chế gồm nhạy biểu diễn/scaling/C, không tự có predict_proba ở LinearSVC và không giải thích từng từ từ hệ số tọa độ. Kernel có thể linh hoạt hơn nhưng tốn tuning/compute và vẫn không bảo đảm thắng tuyến tính.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| Embedding+LR | Loss log và probability link khác hinge/margin |
| LinearSVC | Triển khai linear theo liblinear, ví dụ squared hinge |
| SVC RBF | Kernel phi tuyến, có gamma và scaling sensitivity |
| Embedding+XGBoost | Cây chia theo từng tọa độ, không cùng geometry margin |

Cùng encoder/split mới giúp cô lập classifier; đổi cả hai làm đối chứng khác câu hỏi.

## 11. Sai lầm thường gặp và cách khắc phục

Không gọi mọi SVM có predict_proba. Không truyền probability=True vào LinearSVC vì đó không phải API của nó. Không nói C lớn luôn tốt hơn; nó giảm tương đối regularization và có thể overfit. Không đọc chiều embedding có hệ số lớn như từ/khía cạnh tác động mạnh, hoặc dùng SVM như bằng chứng nhân quả.

## 12. Tài liệu tham khảo

- [S1] Reimers, N., & Gurevych, I. (2019). [Sentence-BERT](https://aclanthology.org/D19-1410/).
- [S2] scikit-learn. [LinearSVC 1.8](https://scikit-learn.org/1.8/modules/generated/sklearn.svm.LinearSVC.html).
- [S3] scikit-learn. [SVC 1.8](https://scikit-learn.org/1.8/modules/generated/sklearn.svm.SVC.html).
- [S4] scikit-learn. [CalibratedClassifierCV](https://scikit-learn.org/1.8/modules/generated/sklearn.calibration.CalibratedClassifierCV.html).
- [S5] Model card [encoder ví dụ](https://huggingface.co/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2).

Nguồn API cần đọc theo đúng estimator; model ví dụ chưa được benchmark tiếng Việt trong bài.

## 13. Vị trí trong toàn bộ nghiên cứu

Pipeline nằm ở dự đoán sau bước dữ liệu/nhãn. Nếu dùng margin để xếp review cần rà soát, đó là quy tắc vận hành phải đánh giá riêng. Margin không phải thang đo tâm lý hoặc mức bất mãn. Tổng hợp dự đoán theo khách sạn cần kiểm tra sai số, nhóm phụ thuộc và tính đại diện.

## 14. Kiến thức nền cần biết trước

Cần hiểu embeddings, margin, hinge loss, C, kernel, scaling và calibration. Phân biệt norm từng hàng với scaler theo cột. Hiểu frozen encoder có thể được cache nhưng fine-tuning/change revision làm cache thay nghĩa. Với tiếng Việt, biết tác động pooling/truncation lên câu phủ định và review dài.

## 15. Nền tảng lý thuyết và logic hình thành

Ranh giới tuyến tính hoạt động trên một biểu diễn đã được encoder học, nên toàn pipeline vẫn có thể phi tuyến theo văn bản gốc. Margin tối ưu là đặc tính trong không gian vector cụ thể; đổi scaling đổi geometry/regularization. Nếu embedding không chứa polarity rõ, SVM không có bảo đảm khôi phục từ score đơn thuần.

## 16. Các thành phần cấu tạo

Thành phần gồm encoder/tokenizer/revision, pooling/normalize, SVM estimator, C/loss/kernel và mapping lớp. Calibration nếu có là một thành phần được học thêm. Cần lưu cả scaler và quy trình fit, không chỉ coef_ của classifier. Với kernel, lưu support vectors/model theo định dạng phù hợp.

## 17. Các loại và biến thể

Có linear SVM, kernel SVM, class-weighted và probability calibration. Multiclass strategy khác theo estimator. Trong ví dụ dùng LinearSVC(loss="squared_hinge", dual="auto"); không lấy công thức hinge bậc một để khẳng định loss code giống hệt. CalibratedClassifierCV có thể bao một pipeline, nhưng phải định nghĩa folds/groups đúng nhiệm vụ.

## 18. Đầu vào và đầu ra

Etrain n×d cùng y, Enew cùng d/model basis. Giá trị âm hợp lệ. Đầu ra LinearSVC gồm nhãn và decision scores, không predict_proba trực tiếp. Nếu thêm calibration, các cột probability cần đọc theo classes_. Không tráo vector giữa encoder khác nhau dù shape giống nhau.

## 19. Căn cứ lựa chọn

Đáng thử khi embedding có cấu trúc tách nhãn hữu ích và cần baseline margin đáng kiểm tra. Chọn bằng metric ngoài mẫu, độ bền phủ định/miền, thời gian encode/fit và calibration cost nếu cần. So LR với cùng encoder để xem lựa chọn loss/classifier có lợi ích thật hay chỉ do tuning khác ngân sách.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu cần probability ngay mà không có dữ liệu/thiết kế calibration, nếu kernel quá tốn tài nguyên hoặc encoder không phù hợp ngôn ngữ. Không dùng để thay định nghĩa nhãn toàn văn/khía cạnh chưa rõ. Score cao không giải quyết nhãn gán mâu thuẫn hoặc dữ liệu trùng.

## 21. Các quyết định quan trọng khi thực hiện

Chọn encoder, normalize/scaler, LinearSVC hay SVC, C, class_weight và kernel/gamma nếu có. Chốt có calibration không, dữ liệu nào fit nó và chi phí nhãn tương ứng. Threshold trên margin nếu điều chỉnh cũng phải chọn bằng development. Không chọn encoder bằng test rồi chỉ coi C là hyperparameter cần kiểm soát.

## 22. Giải thích công thức và ký hiệu

Ví dụ nhị phân dùng $y_i\in\{-1,+1\}$ và $s_i=w^\top e_i+b$. Một dạng mục tiêu squared-hinge:

$$\frac12\|w\|^2+C\sum_i\max(0,1-y_is_i)^2.$$

Cách xử lý intercept/scale của solver cần đọc theo triển khai. Nếu y=+1, s=0.6, phần squared hinge là 0.16; nếu s=1.2 thì bằng 0. Nhãn dự đoán đúng chưa có nghĩa hinge bằng 0. Score s không là xác suất; khoảng cách có dấu đến mặt phẳng trong geometry đó là $s/\|w\|$ khi $w\ne0$.

## 23. Ví dụ tính toán hoặc thao tác từng bước

**A. Đã chạy kiểm tra:** LinearSVC trên vector tự tạo, không phải embedding thật.

```python
# DA_KIEM_TRA
import numpy as np
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import Normalizer
from sklearn.svm import LinearSVC
Xtrain = np.array([[1,.8,.1],[.9,.7,0],[1,.6,-.1],[.8,.9,.2],
                   [-1,-.8,.1],[-.9,-.7,0],[-1,-.6,-.1],[-.8,-.9,.2]], dtype=float)
ytrain = np.array([1,1,1,1,0,0,0,0])
Xnew = np.array([[.95,.75,0],[-.95,-.75,0]], dtype=float)
pipe = make_pipeline(Normalizer(norm="l2"),
                     LinearSVC(C=1.0, loss="squared_hinge", dual="auto",
                               max_iter=10000, random_state=42))
pipe.fit(Xtrain,ytrain)
scores = pipe.decision_function(Xnew)
pred = pipe.predict(Xnew)
assert np.isfinite(scores).all()
manual = pipe[-1].classes_[(scores > 0).astype(int)]
assert np.array_equal(pred,manual)
assert not hasattr(pipe[-1],"predict_proba")
assert np.isclose(max(0,1-0.6)**2,0.16)
print("Score minh họa:",scores,"nhãn:",pred)
```

**B. Mẫu API chưa thực thi:** cần sentence-transformers/weights; không có tuning, calibration hoặc gold test trong ví dụ này.

```python
# MINH_HOA_API_CHUA_CHAY
import numpy as np
from sentence_transformers import SentenceTransformer
from sklearn.svm import LinearSVC
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
clf = LinearSVC(C=1.0, loss="squared_hinge", dual="auto",
                max_iter=10000, random_state=42)
clf.fit(Etrain,ytrain)
print(clf.predict(Enew))
print(clf.decision_function(Enew))
```

Không thêm sigmoid để báo probability từ score. Khi áp dụng, ghi model revision và kiểm tra chính xác loại SVM/cấu hình thư viện đã cài.

## 24. Tình huống sai và phản ví dụ

Một review ngoài miền có thể nằm xa mặt phẳng nhưng vẫn bị dự đoán sai tự tin; margin lớn chưa chứng minh đúng. Câu khẳng định/phủ định gần nhau trong embedding có thể ở cùng phía. Nếu test có review trùng train, chất lượng cao không phản ánh khả năng suy rộng sang khách sạn mới.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đánh giá gold test theo lớp/nhóm, lỗi phủ định, mixed sentiment và độ dài. Kiểm tra convergence/finite scores. Chọn C/kernel/gamma trên development và đo độ ổn định. Nếu calibration, đánh giá probability bằng metric phù hợp riêng; không chỉ accuracy rồi kết luận calibration tốt.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu không hội tụ, xem scaling, solver/max_iter và dữ liệu thay vì bỏ cảnh báo. Nếu kém baseline, kiểm tra encoder/pooling/truncation và nhãn trước tăng kernel complexity. Khi đổi encoder, tạo lại embedding/refit. Nếu cần probability, thêm calibration đúng protocol hoặc chọn model khác theo mục tiêu, không biến score tùy ý.

## 27. Phân biệt kết quả với kết luận

Kết quả là hiệu năng và margin behavior trên protocol cụ thể. Không suy score thành mức hài lòng hoặc coefficient thành khía cạnh gây tác động. Không kết luận SVM luôn tốt hơn LR từ một split. Phân biệt lợi ích encoder với classifier và nêu phạm vi miền/ngôn ngữ được kiểm tra.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Review được encode bằng [...revision...] với [...]. Encoder [đóng băng/fine-tune]. Classifier là [LinearSVC/SVC], loss/kernel [...], C/gamma [...] chọn trên [...]. Test giữ [...].” Báo normalize/scaling, weighting, calibration nếu có, convergence và ngân sách. Chỉ báo probability nếu thực sự có cơ chế hợp lệ.

## 29. Khả năng tái lập và tính minh bạch

Lưu encoder/tokenizer/revision, preprocessing, vector-ID mapping, classifier, classes_, seeds/splits và môi trường. Nếu có calibration, lưu toàn bộ pipeline đó. Mã A chạy sklearn 1.8.0; mã B chỉ kiểm tra cú pháp, chưa tải encoder. Không có benchmark sentence embedding native trong tài liệu này.

## 30. Đạo đức nghiên cứu liên quan

Vector/margin không ẩn danh hóa khách; bảo vệ text/caches và quyền dùng model/dữ liệu. Không dùng score tự động để xác định động cơ khách hoặc phạt nhân viên. Kiểm tra sai số ở bối cảnh có dữ liệu hợp lệ và có người rà soát quyết định tác động.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. LinearSVC có predict_proba không?
2. Score và khoảng cách chuẩn hóa khác ra sao?
3. Với y=+1,s=0.6, squared hinge bao nhiêu?
4. Vì sao C lớn chưa chắc tốt?

Gợi ý: 0.16; score không là probability. Bài mở rộng: so LR và SVM cùng embedding/split, báo metric và chi phí.

## 32. Những điểm còn tranh luận

Các vấn đề cần kiểm chứng gồm tác động normalization, lợi ích kernel, calibration và chuyển miền. Không có C/kernel tối ưu chung. Nghiên cứu nên báo đúng implementation và phân biệt score/margin/probability, thay vì dùng tên SVM như một cơ chế thống nhất cho mọi cấu hình.

