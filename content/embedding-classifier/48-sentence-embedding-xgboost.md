# Sentence embedding + XGBoost — Boosting cây trên vector câu

> **Nhánh:** MACHINE LEARNING → Embedding + bộ phân loại. **Nguồn:** dòng 49 bảng Excel. **Đối chiếu:** 29/10/2026. Ví dụ NumPy kiểm tra một split bậc hai trên vector tự tạo; mã sentence encoder + XGBClassifier chỉ kiểm tra cú pháp, chưa thực thi. Dữ liệu/vector/tình huống minh họa đều tự tạo.

| Trường | Nội dung |
|---|---|
| Thiết kế chính | Encoder đóng băng + classifier có giám sát |
| Nhiệm vụ | Phân loại cảm xúc review khách sạn tiếng Việt |
| Giới hạn mã | Tách ví dụ đã chạy với mẫu API chưa thực thi; không có benchmark embeddings thật |

## 1. Định nghĩa học thuật

Pipeline biến mỗi câu/review thành sentence embedding rồi dùng XGBoost tree booster học nhãn sentiment. Encoder có thể đóng băng hoặc fine-tune theo một protocol riêng; bài tập trung đóng băng. XGBoost nhận ma trận số, không trực tiếp hiểu token. Cây học các phép chia trên tọa độ embedding và cộng cập nhật qua boosting [S1–S3].

## 2. Giải thích đơn giản

Encoder nén review thành một dãy số, còn XGBoost học các câu hỏi như tọa độ j lớn hơn ngưỡng nào, rồi kết hợp nhiều cây. Các tọa độ không có sẵn tên “độ sạch” hay “dịch vụ”. Cây phi tuyến có thể khai thác tương tác nhưng không tự khắc phục embedding làm mất phủ định hoặc cắt mất phần chê.

## 3. Mục đích và vấn đề giải quyết

Có thể dùng làm ứng viên phi tuyến để so với LR/SVM trên cùng embedding. Trong nghiên cứu khách sạn, cần kiểm tra mức tăng ngoài mẫu có ổn định và đáng chi phí không. Không mặc định embedding dense cộng XGBoost sẽ vượt TF-IDF hoặc classifier tuyến tính; kết quả phụ thuộc corpus, encoder và tuning.

## 4. Điều kiện và giả định sử dụng

Cần encoder/ngôn ngữ phù hợp, nhãn rõ, ma trận hữu hạn và schema nhất quán. Embedding có giá trị âm hoàn toàn hợp lệ với cây. Dense/sparse và missing/zero phải có nghĩa rõ; không biến tọa độ 0 thành missing tùy ý. Feature nhiều chiều cùng nhãn ít có thể làm cây overfit hoặc học shortcut.

## 5. Cơ chế hoạt động bên trong

Encode train bằng model cố định. Trong mỗi vòng boosting, tính gradient/Hessian của loss theo margin, tìm split và giá trị lá theo objective/regularization, cộng cây với learning rate. Trong nhị phân logistic, cuối cùng dùng sigmoid để ra probability. Cấu trúc chia theo tọa độ có thể nhạy với cách tổ chức không gian embedding, không chỉ các khoảng cách semantic.

## 6. Quy trình áp dụng trong nghiên cứu

1. Chốt nhãn/đơn vị và split theo khách sạn/thời gian.
2. Chọn encoder/revision/pooling, tách quyền truy cập.
3. Tạo embeddings và kiểm tra ID/shape/finite.
4. Chọn tree_method, objective và ngân sách tuning.
5. Dùng development chọn depth/leaves, min_child_weight, regularization, sampling và số vòng.
6. Nếu early stopping, eval_set là development đã biến đổi đúng; không dùng test.
7. Khóa cả encoder/classifier/threshold trước gold test.

Nếu có PCA/feature selection được học, fit trong train từng fold; nếu encoder fine-tune, cũng phải tôn trọng phạm vi đó.

## 7. Ví dụ minh họa xuyên suốt

Bốn vector giả định có tọa độ đầu dương ở hai mẫu tích cực và âm ở hai mẫu tiêu cực. Một split tách chúng dễ dàng, dùng kiểm tra công thức gain. Trong embedding thật, sentiment có thể phân bố qua nhiều chiều hoặc không rõ; các vector minh họa không phải sản phẩm của encoder hay dữ liệu khách sạn đã thu thập.

## 8. Kết quả và cách diễn giải

Output XGBoost phụ thuộc objective/lời gọi: raw margin khác probability; nhãn cần mapping và quy tắc. Importance theo gain/split/cover có nghĩa khác nhau. Feature index 17 quan trọng không tự là yếu tố dịch vụ quan trọng. Độ chính xác hoặc xác suất tốt vẫn cần được kiểm tra theo lớp/miền và calibration nếu dùng cho quyết định.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là khai thác tương tác phi tuyến, có regularization và early stopping. Hạn chế gồm tuning/chi phí, overfit khi nhiều chiều và khó diễn giải feature semantic. Cây chia theo trục không bất biến chung với phép xoay embedding; hai cơ sở giữ cosine có thể có độ dễ học khác nhau cho cây. Không có bảo đảm XGBoost hợp mọi encoder.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| Embedding+LR/SVM linear | Ranh giới tuyến tính trong embedding, ít kiểu tương tác tự học hơn cây |
| TF-IDF+XGBoost | Feature từ/cụm sparse, khác dense latent coordinates |
| Encoder end-to-end | Cập nhật encoder theo task, không chỉ fit booster |
| XGBoost trên metadata | Cùng thuật toán nhưng schema/ý nghĩa feature khác |

Cần giữ encoder và split khi so classifier; nếu thêm metadata chỉ cho một model, phải báo khác biệt thông tin.

## 11. Sai lầm thường gặp và cách khắc phục

Không dùng test làm eval_set cho early stopping trong đánh giá cuối. Không hiểu min_child_weight là số mẫu tối thiểu: nó liên quan tổng Hessian. Không đặt tên ý nghĩa cho các chiều dựa gain. Không thay encoder/revision rồi tiếp tục dùng booster cũ chỉ vì embedding cùng chiều. Không coi mọi giá trị 0 là thiếu.

## 12. Tài liệu tham khảo

- [S1] Reimers, N., & Gurevych, I. (2019). [Sentence-BERT](https://aclanthology.org/D19-1410/).
- [S2] Chen, T., & Guestrin, C. (2016). *XGBoost: A Scalable Tree Boosting System*. [Bài gốc](https://arxiv.org/abs/1603.02754).
- [S3] XGBoost. [Introduction to Boosted Trees](https://xgboost.readthedocs.io/en/stable/tutorials/model.html).
- [S4] XGBoost. [Sklearn Estimator Interface](https://xgboost.readthedocs.io/en/stable/python/sklearn_estimator.html).
- [S5] Model card [encoder ví dụ](https://huggingface.co/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2).

Mã A chỉ kiểm tra phép tính, mã B chưa chạy các package native.

## 13. Vị trí trong toàn bộ nghiên cứu

Pipeline thuộc khâu dự đoán có giám sát. Nếu muốn giải thích rating/hành vi từ sentiment dự đoán, cần thêm bước kiểm tra sai số đo và thiết kế phân tích. Importance latent coordinates không thay kiểm định lý thuyết về thuộc tính khách sạn hoặc bằng chứng nhân quả.

## 14. Kiến thức nền cần biết trước

Cần hiểu embeddings, frozen/fine-tuning, tree boosting, logistic margin, gradient/Hessian và regularization. Phân biệt tọa độ vector với một biến đo có tên. Hiểu normalization từng hàng có thể thay thứ tự giá trị theo cột; không đồng nhất nó với phép đổi đơn vị đơn điệu theo từng feature mà cây thường ít phụ thuộc.

## 15. Nền tảng lý thuyết và logic hình thành

Encoder cung cấp biểu diễn; booster xấp xỉ quan hệ nhãn bằng tổng cây tham lam. Học bậc hai giúp tính cập nhật/lợi ích split nhưng không chứng minh tối ưu toàn cục. Nếu thông tin sentiment nằm trong hướng xiên nhiều chiều, cây cần cấu trúc khác so với linear separator. Phi tuyến không mặc định phù hợp hơn biểu diễn.

## 16. Các thành phần cấu tạo

Thành phần gồm encoder/tokenizer/revision, pooling/normalize, schema embeddings, objective, booster/tree_method, cấu hình cây, sampling, regularization và tiêu chí dừng. Development vector phải cùng biến đổi với train; eval_set không tự được vector hóa chỉ vì classifier ở cuối một pipeline bình thường.

## 17. Các loại và biến thể

Có frozen embeddings với tree booster, fine-tuned embeddings rồi booster, hoặc feature fusion với metadata. Có các chế độ booster/tree method khác nhau nhưng bài tập trung hist tree classifier nhị phân. Đổi sang linear booster là phương pháp khác cấu hình với công thức cây không còn áp nguyên. Multiclass có objective/cấu trúc riêng.

## 18. Đầu vào và đầu ra

Etrain n×d có thể dense có dấu; y là nhãn số hợp lệ cho objective/API. Development/test có cùng encoder, pooling và thứ tự feature. Đầu ra probability/nhãn/margin và booster. Cache phải gắn văn bản/model configuration; số chiều bằng nhau chưa đảm bảo hai batch ở cùng không gian.

## 19. Căn cứ lựa chọn

Đáng thử khi có lý do quan hệ phi tuyến hữu ích, đủ nhãn/development và ngân sách. Chọn bằng quality ngoài mẫu, calibration nếu cần và tổng compute. So cùng embedding+LR/SVM và baseline văn bản phù hợp; không chỉ so điểm train hoặc lượt tuning tốt nhất có ngân sách lớn hơn.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu nhãn quá ít, encoder không phù hợp ngôn ngữ hoặc yêu cầu giải thích từng từ rất trực tiếp. Nếu split bị trùng/nhóm phụ thuộc, thêm cây không sửa tính độc lập. Nếu cần đo tác động của khía cạnh, prediction và gain không tự là thiết kế nhận diện nhân quả.

## 21. Các quyết định quan trọng khi thực hiện

Chọn encoder/length/pooling, có normalize/PCA không, tree_method, learning rate, số vòng, depth, min_child_weight và penalties. Chọn weighting/threshold theo mục tiêu lớp hiếm. Nếu early stopping, lưu best iteration và cách API dùng khi predict. Các lựa chọn encoder cũng là model selection, phải nằm ngoài test cuối.

## 22. Giải thích công thức và ký hiệu

Với $p_i=\sigma(F_i)$, log loss nhị phân cho $g_i=p_i-y_i$, $h_i=p_i(1-p_i)$. Với lá j: $G_j=\sum g_i$, $H_j=\sum h_i$. Một dạng L2 không L1:

$$w_j^*=-\frac{G_j}{H_j+\lambda},$$

$$Gain=\frac12\left[\frac{G_L^2}{H_L+\lambda}+\frac{G_R^2}{H_R+\lambda}-\frac{(G_L+G_R)^2}{H_L+H_R+\lambda}\right]-\gamma.$$

Ở bốn mẫu, p=0.5, hai nhánh thuần lớp, λ=1, γ=0: giá trị lá ±2/3 và gain 2/3 theo xấp xỉ. Gain không phải macro-F1 tăng 2/3 hoặc ý nghĩa của một khía cạnh.

## 23. Ví dụ tính toán hoặc thao tác từng bước

**A. Đã chạy kiểm tra:** một split trên vector tự tạo, chỉ dùng NumPy; không gọi xgboost hoặc sentence encoder.

```python
# DA_KIEM_TRA
import numpy as np
E = np.array([[1.,0.],[1.,2.],[-1.,0.],[-1.,2.]])
y = np.array([1.,1.,0.,0.])
p = np.full(4,0.5)
g, h = p-y, p*(1-p)
left = E[:,0] > 0
GL, HL = g[left].sum(), h[left].sum()
GR, HR = g[~left].sum(), h[~left].sum()
lam, gamma = 1.,0.
wL, wR = -GL/(HL+lam), -GR/(HR+lam)
gain = 0.5*(GL**2/(HL+lam)+GR**2/(HR+lam)
             -(GL+GR)**2/(HL+HR+lam))-gamma
assert np.allclose([wL,wR,gain],[2/3,-2/3,2/3])
margin = 0.1*np.where(left,wL,wR)
new_p = 1/(1+np.exp(-margin))
assert np.isfinite(new_p).all() and np.all((new_p>0)&(new_p<1))
print("Giá trị lá/gain:",wL,wR,gain)
```

**B. Mẫu API chưa thực thi:** cần sentence-transformers, xgboost, scikit-learn và tải weights. Development chỉ để minh họa early stopping; chưa có đánh giá gold test độc lập trong đoạn này.

```python
# MINH_HOA_API_CHUA_CHAY
import numpy as np
from sentence_transformers import SentenceTransformer
from xgboost import XGBClassifier
train_texts = ["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
               "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
               "dịch vụ tệ bẩn", "khách sạn bẩn tệ"]
ytrain = np.array([1,1,1,1,0,0,0,0])
new_texts = ["phòng không sạch", "nhân viên hỗ trợ nhiệt tình"]
dev_texts = ["phòng đẹp", "dịch vụ tốt", "phòng tệ", "dịch vụ bẩn"]
ydev = np.array([1,1,0,0])
encoder = SentenceTransformer("sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2")
def encode(texts):
    return encoder.encode(texts, convert_to_numpy=True,
                          normalize_embeddings=True, show_progress_bar=False)
Etrain, Edev, Enew = encode(train_texts), encode(dev_texts), encode(new_texts)
clf = XGBClassifier(objective="binary:logistic", tree_method="hist",
                    n_estimators=100, max_depth=2, learning_rate=0.05,
                    min_child_weight=1, reg_lambda=1, random_state=42,
                    n_jobs=1, eval_metric="logloss", early_stopping_rounds=10)
clf.fit(Etrain,ytrain,eval_set=[(Edev,ydev)],verbose=False)
print(clf.predict(Enew))
```

Mẫu B không xác nhận package chạy trong môi trường hiện tại. Khi làm nghiên cứu, thay bằng đủ dữ liệu phát triển/test và ghi model/package revisions; không dùng bốn câu dev như bằng chứng lựa chọn bền vững.

## 24. Tình huống sai và phản ví dụ

Một tọa độ có gain cao có thể liên quan ngôn ngữ/khách sạn hoặc hướng latent không có tên semantic. Một phép xoay trực giao giữ khoảng cách/cosine nhưng đổi các split theo trục, nên không suy chất lượng cây bất biến. Review dài bị cắt mất đoạn chê không được phục hồi tự động bằng boosting.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Kiểm tra learning curves train/development, convergence/warnings, lớp hiếm và độ bền group/time split. Làm đối chứng cùng encoder và ngân sách; xem độ nhạy seed/tuning. Nếu báo importance, ghi loại gain/weight/cover và giới hạn latent coordinates. Đánh giá probability riêng khi dùng cho quyết định.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu overfit, kiểm tra leakage/nhãn rồi giảm độ phức tạp, tăng regularization hoặc chọn số vòng đúng development. Nếu biểu diễn kém, xem encoder/pooling/length thay vì chỉ thêm cây. Nếu đổi feature transform, fit lại booster và cache tương ứng. Khi không cải thiện ổn định so LR/SVM, có thể giữ mô hình đơn giản hơn.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ là chất lượng của pipeline đã khóa trên đánh giá độc lập. Không kết luận coordinate quan trọng là nguyên nhân hài lòng hoặc XGBoost luôn khai thác embedding tốt hơn tuyến tính. Phân biệt hạn chế của encoder, classifier và thiết kế dữ liệu trước giải thích quản trị.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Sentence encoder [...revision...] được dùng với [pooling/normalize/length], [đóng băng/fine-tune]. XGBoost tree booster [objective/tree_method] được chọn bằng [...]. Early stopping chỉ theo dõi development; test giữ theo [...].” Báo số vòng, penalties/sampling, môi trường, compute và đối chứng cùng embedding. Không dùng số gain minh họa như kết quả thực nghiệm.

## 29. Khả năng tái lập và tính minh bạch

Lưu văn bản-ID mapping, model revision/tokenizer, embeddings manifest, transforms, booster/config, best iteration, seeds và splits. Mã A chỉ chạy NumPy; mã B chỉ kiểm tra cú pháp do thiếu các package/weights. Không tuyên bố đã huấn luyện XGBoost với embeddings thật hoặc đo hiệu năng native trong bài.

## 30. Đạo đức nghiên cứu liên quan

Embeddings/caches vẫn có nguy cơ thông tin nhạy cảm. Quản lý quyền encode, lưu trữ và chia sẻ phù hợp; không gửi review sang dịch vụ ngoài mặc định. Không dùng probability/importance tự động để quy kết khách hoặc nhân viên. Đánh giá lỗi theo bối cảnh hợp lệ và có người rà soát.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. XGBoost có tự hiểu tên từng chiều embedding không?
2. min_child_weight có bằng số mẫu không?
3. Dùng tập nào cho early stopping?
4. Vì sao xoay embedding có thể đổi độ dễ học của cây?

Gợi ý: split theo trục thay đổi dù cosine có thể giữ. Bài mở rộng: so ba classifier trên cùng encoder/split, báo cả encode time và tuning budget.

## 32. Những điểm còn tranh luận

Các vấn đề còn cần kiểm chứng là sự phù hợp axis-aligned trees với embedding, mức lợi ích phi tuyến và độ bền miền/ngôn ngữ. Không có cấu hình tốt nhất chung. Nghiên cứu nên tách encoder với classifier, công bố mọi lựa chọn và giữ diễn giải latent importance ở đúng phạm vi.

