# Naive Bayes — Phân loại cảm xúc bằng xác suất có điều kiện

> **Nhánh:** MACHINE LEARNING → Supervised: Probabilistic. **Nguồn:** dòng 18 bảng Excel. **Đối chiếu:** 28/10/2026. Bài tập trung Multinomial Naive Bayes cho văn bản, có phân biệt các biến thể. Số liệu và phản hồi khách sạn tự soạn; không phải benchmark thực. Mã được kiểm tra với scikit-learn 1.8.0.

| Trường | Nội dung |
|---|---|
| Bản chất | Họ bộ phân loại có giám sát dùng định lý Bayes và giả định độc lập có điều kiện |
| Mốc tham khảo | McCallum & Nigam, 1998 về các event model cho văn bản [S2]; không coi đó là năm ra đời định lý Bayes hay toàn bộ họ NB |
| Đầu vào | Vector đặc trưng theo biến thể và nhãn huấn luyện |
| Đầu ra | Nhãn, log-score và xác suất theo mô hình; không bảo đảm hiệu chuẩn |
| Tiếng Việt | Có thể dùng với biểu diễn phù hợp; cần kiểm tra phủ định, cụm và chuẩn hóa |
| Nhãn/tài nguyên | Cần nhãn. Đếm và suy luận sparse thường có thể dùng CPU |
| Phần mềm | `MultinomialNB`, `BernoulliNB`, `ComplementNB`, `GaussianNB` trong scikit-learn [S1] |

## 1. Định nghĩa học thuật

Naive Bayes dùng quy tắc Bayes để suy xác suất lớp từ các đặc trưng và giả định các đặc trưng độc lập có điều kiện khi biết lớp. “Naive” nói đến giả định mô hình, không phải kết luận các từ trong ngôn ngữ thật độc lập.

Với MultinomialNB trong văn bản, ta học phân bố xác suất mục từ theo lớp và kết hợp với prior lớp. Các biến thể NB khác đặt mô hình khác cho dữ liệu đặc trưng [S1–S2]. Không được lấy công thức Multinomial áp nguyên cho Bernoulli hoặc Gaussian.

## 2. Giải thích đơn giản

Bạn xem trong các đánh giá đã gán nhãn, những từ nào hay xuất hiện ở nhóm khen và nhóm chê. Khi gặp đánh giá mới, bạn kết hợp bằng chứng của các từ với mức phổ biến ban đầu của mỗi lớp.

Điểm đơn giản hóa là coi các bằng chứng riêng độc lập khi đã biết lớp. Hai từ thường đi cùng có thể khiến mô hình đếm thông tin tương tự nhiều lần. Dù giả định không đúng hoàn toàn, bộ phân loại vẫn có thể hữu ích; phải đo hiệu quả thay vì suy từ tên gọi.

## 3. Mục đích và vấn đề giải quyết

Đề xuất nghiên cứu khách sạn: xây baseline phân loại tích cực/tiêu cực bằng count unigram hoặc n-gram, so với SVM và Logistic Regression trên cùng dữ liệu. NB cũng có thể giúp kiểm tra nhanh chất lượng nhãn và biểu diễn.

Nếu cần lớp trung tính, phải có dữ liệu/định nghĩa cho lớp đó. Không suy từ xác suất hai lớp gần nhau rằng câu chắc chắn trung tính; nó có thể là ý kiến hỗn hợp, thiếu từ đã biết hoặc mô hình chưa phân biệt được.

## 4. Điều kiện và giả định sử dụng

Cần nhãn và chọn biến thể phù hợp dạng feature. MultinomialNB dùng đặc trưng không âm; count là lựa chọn tự nhiên. TF-IDF cũng được dùng thực hành nhưng không phải literal multinomial count [S1].

Giả định độc lập có điều kiện là một xấp xỉ cần xem hệ quả. Nếu dùng cả unigram và bigram, feature có thể chồng thông tin; không có nghĩa không được dùng, nhưng xác suất đầu ra đặc biệt cần kiểm định. Train và test phải được chia theo mục tiêu khái quát hóa.

## 5. Cơ chế hoạt động bên trong

Với thiết kế đếm: fit vocabulary trên train → đếm mục từ theo lớp → ước lượng prior và likelihood có smoothing → cộng log-likelihood theo số đếm của mẫu mới → chuẩn hóa nếu cần xác suất → chọn lớp.

Không có bước bắt buộc phải học từng epoch như Perceptron. Phép cộng log giúp tránh nhân nhiều số rất nhỏ. Từ ngoài vocabulary không được tự động có điểm cảm xúc; trong biểu diễn cố định, chúng thường không đóng góp feature, nên cần kiểm tra lượng thông tin bị bỏ qua.

## 6. Quy trình áp dụng trong nghiên cứu

1. Định nghĩa nhãn và đơn vị phân tích, xử lý trung tính/hỗn hợp.
2. Loại trùng, chia train/development/test theo thời gian hoặc khách sạn khi cần.
3. Đặt vectorizer và NB trong pipeline, fit trong từng fold huấn luyện.
4. Thử count, biến thể n-gram và NB phù hợp trên development.
5. Chọn smoothing, prior và ngưỡng nếu có, không dùng nhãn test.
6. Đánh giá từng lớp và xác suất nếu mục tiêu cần xác suất.
7. Đọc lỗi theo phủ định, OOV, độ dài và nguồn phản hồi.

Nếu chọn feature dựa trên nhãn, bước đó cũng phải nằm trong phần huấn luyện của từng fold, không chọn trước trên toàn dữ liệu.

## 7. Ví dụ minh họa xuyên suốt

Giả định vocabulary chỉ có `sạch` và `bẩn`. Ở lớp tích cực, số đếm là (3,1); ở lớp tiêu cực là (1,3). Hai lớp có prior bằng nhau và dùng add-one smoothing.

Mẫu mới có một `sạch` cho bằng chứng nghiêng về lớp tích cực. Đây là ví dụ toán học cố ý đơn giản; “phòng không sạch” không thể được hiểu đúng chỉ bằng feature `sạch` nếu bỏ tín hiệu phủ định.

## 8. Kết quả và cách diễn giải

Posterior lớp là kết quả theo mô hình và giả định đã chọn, không phải độ hài lòng thực của khách. Cột xác suất phải đọc theo `classes_`. Xác suất cộng thành 1 chưa chứng minh calibration tốt.

Với dữ liệu nhiều feature tương quan, NB có thể tự tin quá mức dù nhãn đúng tương đối tốt. Vì vậy, phân biệt hiệu năng phân lớp và chất lượng xác suất; tài liệu thư viện cũng lưu ý hai mặt này [S1]. Nếu có nhu cầu xác suất, dùng kiểm định và hiệu chuẩn phù hợp.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm thực hành là đếm/ước lượng đơn giản, baseline dễ chạy và có thể kiểm tra bằng chứng mục từ theo lớp. Một số biến thể hỗ trợ cập nhật tăng dần khi giữ không gian feature nhất quán.

Hạn chế gồm giả định độc lập, nhạy biểu diễn, prior lệch miền và lỗi ngữ cảnh. “Không tốt” với chỉ unigram không được bảo đảm xử lý đúng. Smoothing giúp tránh xác suất 0 do thiếu số đếm, không tự chữa mỉa mai hoặc nhãn mâu thuẫn.

## 10. So sánh với phương pháp gần nhau

| Phương pháp | Khác biệt |
|---|---|
| MultinomialNB | Tính theo số đếm; sự lặp từ góp thêm bằng chứng |
| BernoulliNB | Tính hiện diện/vắng mặt nhị phân; cả không xuất hiện cũng tham gia mô hình |
| ComplementNB | Dùng thống kê phần bù theo cách thiết kế riêng, thường được thử khi lệch lớp |
| Logistic/MaxEnt | Mô hình trực tiếp phân bố lớp có điều kiện, không cần giả định NB về likelihood |
| Bayesian Network | Họ đồ thị xác suất rộng hơn; NB là cấu trúc phân loại hạn chế có liên hệ với BN |

GaussianNB đặt giả định liên tục khác; không chọn chỉ vì tên “Bayes” giống nhau.

## 11. Sai lầm thường gặp và cách khắc phục

- Nhầm độc lập có điều kiện với độc lập vô điều kiện: viết đúng giả định.
- Dùng feature âm cho MultinomialNB: chọn biểu diễn/biến thể phù hợp, không dịch điểm tùy tiện để ép chạy.
- Fit vocabulary trên toàn dữ liệu: fit trong train/CV pipeline.
- Gọi smoothing là “thêm nhãn”: nó làm trơn ước lượng, không tạo quan sát nhãn thật.
- Xem probability cao là chắc chắn: đánh giá calibration và shift.

## 12. Tài liệu tham khảo

- **[S1]** scikit-learn. [Naive Bayes User Guide](https://scikit-learn.org/stable/modules/naive_bayes.html).
- **[S2]** McCallum, A., & Nigam, K. (1998). *A Comparison of Event Models for Naive Bayes Text Classification*. [Bản công trình](https://www.cmi.ac.in/~madhavan/courses/dmml2020jan/literature/mccallum-nigam.pdf).
- **[S3]** scikit-learn. [MultinomialNB API](https://scikit-learn.org/stable/modules/generated/sklearn.naive_bayes.MultinomialNB.html).
- **[S4]** scikit-learn. [Probability calibration](https://scikit-learn.org/stable/modules/calibration.html).
- **[S5]** scikit-learn. [Common pitfalls](https://scikit-learn.org/stable/common_pitfalls.html).

Nguồn giải thích nền tảng/triển khai. Corpus khách sạn và số đếm trong bài là minh họa tự xây, không phải kết quả được công bố ở các nguồn này.

## 13. Vị trí trong toàn bộ nghiên cứu

NB nằm sau biểu diễn và trước đánh giá hoặc sử dụng nhãn máy. Nếu dùng tỷ lệ nhãn máy để mô tả thị trường khách sạn, cần tính đến sai số phân loại và mẫu không đại diện.

Nếu dùng NB tạo nhãn yếu cho mô hình sau, chất lượng so với nhãn NB không phải chất lượng so với người gán độc lập. Cần một chuẩn kiểm tra không được tạo bởi chính hệ thống đang đánh giá.

## 14. Kiến thức nền cần biết trước

Cần hiểu prior $P(c)$, likelihood $P(x\mid c)$, posterior $P(c\mid x)$, normalization và logarit. Phân biệt số văn bản theo lớp với tổng số token theo lớp: chúng dùng cho hai phần ước lượng khác nhau.

Cũng cần phân biệt vocabulary cố định và OOV. Smoothing cho mục đã có trong vocabulary nhưng chưa thấy ở một lớp khác việc tự tạo feature cho mọi từ mới xuất hiện ngoài vocabulary.

## 15. Nền tảng lý thuyết và logic hình thành

Bayes chuyển từ một mô hình sinh đặc trưng theo lớp sang suy luận lớp từ đặc trưng. Giả định đơn giản hóa cho phép ước lượng các phần nhỏ thay vì một phân bố đồng thời rất lớn.

Đó là sự đánh đổi giữa độ đơn giản và việc mô tả phụ thuộc thật. Một mô hình xác suất sai chi tiết vẫn có thể xếp lớp đúng ở nhiều mẫu; vì vậy, phải kiểm tra cả quyết định và xác suất thay vì coi chúng luôn tốt/kém cùng nhau.

## 16. Các thành phần cấu tạo

Hệ gồm vocabulary, số đếm theo lớp, prior, likelihood có smoothing, phép cộng log và quy tắc quyết định. Tùy biến thể có thành phần vắng mặt hoặc phần bù khác.

Nên lưu nguồn nhãn, mapping lớp và cách xây vocabulary cùng mô hình. Chỉ lưu `feature_log_prob_` mà không biết cột là từ nào sẽ không đủ tái lập hoặc giải thích đóng góp.

## 17. Các loại và biến thể

Các loại cần phân biệt gồm Multinomial, Bernoulli, Complement, Gaussian và Categorical NB [S1]. Chúng có giả định dữ liệu/ước lượng riêng.

Có thể thử count hoặc TF-IDF, unigram hoặc n-gram. Đây là thay đổi biểu diễn, không đồng nhất với đổi họ NB. Không mặc định Complement luôn thắng trên mọi dữ liệu lệch lớp; dùng nó như phương án cần so công bằng.

## 18. Đầu vào và đầu ra

Đầu vào $X$ là ma trận feature phù hợp và $y$ là nhãn. Đầu ra lưu nhãn, posterior/log-score, `classes_`, split, phiên bản và mã bản ghi.

Nếu một mẫu thành vector 0 trong MultinomialNB, phần bằng chứng token biến mất; posterior thường trở về prior theo mô hình. Cần ghi trạng thái này khi diễn giải, thay vì trình bày như hệ đã đọc hiểu đánh giá không có từ nhận diện được.

## 19. Căn cứ lựa chọn

Nên thử khi cần baseline phân loại văn bản có nhãn, feature không âm và ngân sách tính toán nhỏ. So với mô hình đa số và các baseline tuyến tính giúp xác định ích lợi.

Nếu mục tiêu đòi xác suất tin cậy hoặc ngữ cảnh phức tạp, xem metric tương ứng trước quyết định. “Chạy nhanh” và “đủ tốt cho nghiên cứu” là hai tiêu chí cần đánh giá riêng.

## 20. Trường hợp không nên sử dụng

Không dùng MultinomialNB với vector embedding chứa giá trị âm mà không thiết kế biến thể phù hợp. Không dùng prior từ mẫu cân bằng nhân tạo để suy thẳng tỷ lệ tự nhiên của khách hàng.

Nếu nhiệm vụ là trích span khía cạnh, NB phân loại toàn văn không phải giải pháp tương đương CRF. Có thể xây phương án token độc lập làm đối chứng, nhưng phải đánh giá đúng nhãn/span và giới hạn liên kết chuỗi.

## 21. Các quyết định quan trọng khi thực hiện

Chọn biến thể, smoothing $\alpha$, prior, n-gram, `min_df`, chuẩn hóa và ngưỡng. $\alpha$ quá lớn có thể làm các phân bố lớp giống nhau hơn; quá nhỏ có thể tạo tỷ số cực đoan khi số đếm ít.

Nếu dùng cả từ và cụm chồng lấn, cần hiểu chúng không độc lập thật. Có thể vẫn thử vì hiệu quả dự đoán, nhưng không suy xác suất đã trở nên đúng về mặt sinh ngôn ngữ chỉ vì F1 tăng.

## 22. Giải thích công thức và ký hiệu

Với Multinomial NB và vector số đếm $x_j$, score theo lớp là:

$$s_c=\log\pi_c+\sum_{j=1}^{V}x_j\log\theta_{cj},
\quad\theta_{cj}=\frac{N_{cj}+\alpha}{\sum_{k=1}^{V}N_{ck}+\alpha V}.$$

$\pi_c$ là prior lớp; $N_{cj}$ là số token feature $j$ ở lớp $c$; $V$ là kích thước vocabulary; $\alpha>0$ là additive smoothing trong công thức này. Prior có thể ước lượng từ số văn bản hoặc được quy định; không dùng cùng mẫu số token cho prior một cách nhầm lẫn.

Posterior được chuẩn hóa từ $e^{s_c}$ bằng tính toán ổn định log-sum-exp. Hệ số multinomial chỉ phụ thuộc $x$ được bỏ khi so lớp. Với Bernoulli, công thức có cả hiện diện và vắng mặt; không tự dùng công thức count này.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Với số đếm mục 7 và $\alpha=1,V=2$, xác suất token `sạch` ở lớp tích cực là $4/6$, ở lớp tiêu cực $2/6$. Prior mỗi lớp 0.5. Với một token `sạch`, posterior tích cực là $2/3$. Đây không phải số đo sự hài lòng thực tế.

Mã cần NumPy và scikit-learn. Phần đầu kiểm tra phép tính, phần sau minh họa pipeline; dữ liệu cực nhỏ không là thí nghiệm đánh giá mô hình thực.

```python
import numpy as np
from sklearn.naive_bayes import MultinomialNB
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import CountVectorizer
from sklearn.metrics import f1_score

toy = MultinomialNB(alpha=1.0).fit([[3, 1], [1, 3]], [1, 0])
column = np.flatnonzero(toy.classes_ == 1).item()
assert np.isclose(toy.predict_proba([[1, 0]])[0, column], 2 / 3)

train = ["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
         "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
         "dịch vụ tệ bẩn", "khách sạn bẩn tệ"]
y_train = [1, 1, 1, 1, 0, 0, 0, 0]
test = ["phòng sạch zzholdout", "dịch vụ tốt", "phòng bẩn", "dịch vụ tệ"]
y_test = [1, 1, 0, 0]
model = Pipeline([
    ("count", CountVectorizer(ngram_range=(1, 2))),
    ("nb", MultinomialNB(alpha=1.0)),
])
model.fit(train, y_train)
prob = model.predict_proba(test)
pred = model.predict(test)
assert "zzholdout" not in model.named_steps["count"].vocabulary_
assert np.isfinite(prob).all() and np.allclose(prob.sum(axis=1), 1.0)
print("Pred:", pred.tolist())
print("Macro-F1 minh họa:", f1_score(y_test, pred, average="macro"))
```

Token test riêng chỉ kiểm tra một dạng rò rỉ vocabulary; pipeline không tự xử lý trùng, chia nhóm hoặc tối ưu theo test. Nhãn demo 0/1 không có trung tính.

## 24. Tình huống sai và phản ví dụ

Nếu `tuyệt` và `vời` thường đi cùng, hai feature có thể mang thông tin tương quan. NB vẫn cộng riêng log bằng chứng; posterior có thể quá mạnh so với mức độc lập thật.

“Không tốt” và “tốt” có thể bị dự đoán giống nhau nếu phần biểu diễn/học không phân biệt đủ. Tăng smoothing không tự hiểu phủ định; cần xem n-gram, nhãn và cấu trúc câu.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đề xuất macro-F1, precision/recall từng lớp, confusion matrix, baseline đa số và kiểm tra nhóm OOV/phủ định. Nếu quan tâm xác suất, thêm log loss, Brier và calibration trên dữ liệu phù hợp.

Thử độ nhạy theo $\alpha$, prior và feature bằng development/CV đúng pipeline. Kiểm tra chuyển khách sạn/thời gian khi đó là mục tiêu. Không dùng độ chính xác train hoặc dữ liệu tự soạn rất dễ để khẳng định khả năng khái quát.

## 26. Xử lý khi không đáp ứng yêu cầu

Prior lấn át: kiểm tra phân bố lớp, sampling và metric, không chỉ ép cân bằng. Sai token hiếm: thử smoothing và lọc feature trên train. Xác suất quá tự tin: kiểm tra tương quan feature và calibration.

Nếu lỗi là khía cạnh hoặc ngữ cảnh dài, đổi biểu diễn/mô hình cần thiết hơn sửa số đếm. Mọi thay đổi chọn từ development; nếu test đã dùng để sửa, cần đánh giá độc lập mới.

## 27. Phân biệt kết quả với kết luận

“NB gán tích cực với posterior 0.9” là đầu ra mô hình, chưa phải mức chắc chắn thực 90%. “Từ sạch có likelihood cao ở lớp tích cực” không chứng minh vệ sinh gây hài lòng.

Nếu muốn mô tả tỷ lệ phản hồi, phải xem sai số mô hình và mẫu. Nhãn máy có tính xác suất không loại bỏ nhu cầu kiểm tra giá trị đo lường hoặc thiên lệch thu thập.

## 28. Cách viết vào bài nghiên cứu

Nêu biến thể NB, feature, smoothing, prior, chuẩn hóa và chia tập. Mẫu: “Count n-gram được fit trong từng fold; MultinomialNB với [alpha/prior] chọn theo [metric] trên [development/CV], test giữ riêng.”

Nếu dùng TF-IDF, ghi rõ đây là lựa chọn thực hành chứ không mô tả nó như count nguyên. Nếu báo xác suất, thêm kết quả hiệu chuẩn/giới hạn, không chỉ xuất `predict_proba` rồi gọi đó là confidence đã kiểm chứng.

## 29. Khả năng tái lập và tính minh bạch

Lưu pipeline, vocabulary, thứ tự lớp, prior, số đếm/ước lượng, phiên bản và split. Với `partial_fit`, lưu các lớp được khai báo, thứ tự lô và biểu diễn feature cố định.

Kho GitHub có thể chứa code và dữ liệu giả định; corpus thật cần quyền chia sẻ và ẩn danh. Báo các quyết định tiền xử lý để người khác không vô tình bỏ phủ định hoặc đổi n-gram khi tái lập.

## 30. Đạo đức nghiên cứu liên quan

Tần suất từ theo lớp có thể phản ánh nhóm khách, tên địa danh hoặc kiểu viết ngoài mục tiêu. Kiểm tra lỗi theo các nhóm liên quan và tránh gán đặc điểm tiêu cực cho cộng đồng từ thống kê từ vựng.

Không dùng nhãn máy đơn độc để xử lý cá nhân. Xác suất rất cao của một hệ chưa hiệu chuẩn có thể tạo cảm giác chắc chắn sai, nên báo ý nghĩa và giới hạn đầu ra khi chia sẻ kết quả.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **NB giả định từ độc lập hoàn toàn không?** Giả định độc lập có điều kiện theo lớp, theo event model được chọn.
2. **Bernoulli có bỏ qua từ vắng mặt không?** Không trong mô hình Bernoulli chuẩn.
3. **Bài tập tính:** với ví dụ hai token, thay mẫu mới thành một `bẩn`; posterior tích cực là $1/3$.
4. **Bài tập thiết kế:** so count unigram, n-gram và TF-IDF trên cùng split; kiểm tra cả nhãn và calibration.
5. **Bài tập phản biện:** giải thích vì sao tăng feature chồng lấn có thể tăng F1 nhưng làm xác suất kém tin hơn.

## 32. Những điểm còn tranh luận

Các lựa chọn quan trọng là event model, tác dụng của vi phạm độc lập, smoothing và cách dùng xác suất. Mô hình đơn giản có thể là baseline tốt nhưng không tự đủ cho mọi mục tiêu nghiên cứu.

Một kết quả phân lớp tốt không chứng minh mô hình sinh ngôn ngữ đúng hoặc posterior được hiệu chuẩn. Nên báo riêng chất lượng quyết định và đại lượng xác suất, gắn với corpus và biểu diễn đã kiểm tra.
