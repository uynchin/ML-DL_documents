# Hierarchical Clustering — Phân cụm phân cấp đánh giá

> **Nhánh:** MACHINE LEARNING → Unsupervised và Topic. **Nguồn:** dòng 34 bảng Excel. **Đối chiếu:** 29/10/2026. Tập trung agglomerative clustering và Ward; mã chạy SciPy và sklearn 1.8.0. Mọi ví dụ và dữ liệu trong bài đều tự tạo.

| Trường | Nội dung |
|---|---|
| Vai trò | Khám phá/biểu diễn không dùng nhãn cảm xúc trong ví dụ |
| Bối cảnh | Review khách sạn tiếng Việt; không có benchmark dữ liệu thật |
| Đọc kết quả | Phân biệt cấu trúc học được với nhãn/khái niệm được xác thực |

## 1. Định nghĩa học thuật

Hierarchical clustering tạo một cấu trúc phân cấp các nhóm lồng nhau. Agglomerative bắt đầu từ từng mẫu rồi gộp dần; divisive bắt đầu từ nhóm lớn rồi chia. Bài tập trung agglomerative, nơi linkage xác định cặp nhóm được gộp. Cây phân cấp không phải cây quyết định có giám sát và không trực tiếp tạo nhãn cảm xúc.

## 2. Giải thích đơn giản

Hình dung gom những review gần nhau thành nhóm nhỏ, rồi gom các nhóm nhỏ thành nhóm lớn. Ở mức chi tiết có thể thấy “giường” và “cách âm”; ở mức rộng hơn có thể đặt tên “phòng”. Tuy nhiên, tên các mức do người đọc xác định sau kiểm tra, không tự là một hệ thống khái niệm chuẩn.

## 3. Mục đích và vấn đề giải quyết

Dùng để khám phá cấu trúc nội dung ở nhiều mức và hỗ trợ xây taxonomy review. Trong nghiên cứu khách sạn, có thể khảo sát liệu các nhóm trải nghiệm nhỏ kết hợp thành nhóm rộng dễ diễn giải. Nếu cần tỷ lệ cảm xúc, cần gán/kiểm tra nhãn riêng; một nhánh dendrogram không mặc định là tích cực.

## 4. Điều kiện và giả định sử dụng

Cần khoảng cách và linkage tương thích, số mẫu đủ nhưng phù hợp tài nguyên. Ward yêu cầu hình học Euclid; không dùng Ward với ma trận cosine rồi giữ nguyên diễn giải phương sai. Nếu tiền xử lý thay đổi khoảng cách, cấu trúc cây cũng thay đổi. Trùng review và feature chi phối có thể tạo nhánh giả ổn định.

## 5. Cơ chế hoạt động bên trong

Tạo khoảng cách giữa các nhóm ban đầu; tìm cặp tốt nhất theo linkage, gộp, cập nhật khoảng cách và tiếp tục. Các phép gộp tham lam thường không được sửa ngược trong quá trình đó. Có thể cắt cây theo số cụm hoặc mức khoảng cách để lấy một phân hoạch. Quy tắc cắt là lựa chọn nghiên cứu cần công bố.

## 6. Quy trình áp dụng trong nghiên cứu

1. Chọn đơn vị và biểu diễn.
2. Loại trùng, xác định phạm vi corpus và mục tiêu mô tả/suy rộng.
3. Chọn metric-linkage hợp lệ.
4. Xây cây và khảo sát mức cắt trên dữ liệu phát triển.
5. Đọc các nhánh, mẫu đại diện lẫn phản ví dụ.
6. Đánh giá độ ổn định và chi phí.

Nếu chỉ vẽ dendrogram cho một mẫu con, ghi rõ cách lấy mẫu; không coi cây của mẫu con đại diện đầy đủ mọi review.

## 7. Ví dụ minh họa xuyên suốt

Xét bốn điểm (0,0),(0,2),(8,0),(8,2). Hai cặp cùng phía gần nhau với Euclid bằng 2. Ward gộp từng cặp rồi gộp hai nhóm lớn. Tọa độ tự tạo, không mang nhãn cảm xúc. Cùng dữ liệu có thể có độ cao gộp khác nếu dùng single/complete/average linkage; không so độ cao như cùng một đại lượng.

## 8. Kết quả và cách diễn giải

Đầu ra gồm cấu trúc gộp, khoảng cách/độ cao theo quy ước và nhãn ở mức cắt. Vị trí trái/phải của lá trên hình không có thứ tự tâm lý cố định; đổi cách vẽ có thể đổi thứ tự mà không đổi phân cấp. Màu cụm và mã nhãn tùy ý, cần mapping rõ khi đặt tên.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là cho nhiều mức nhóm và không cần chọn một K duy nhất trước khi xây cây đầy đủ. Nhược điểm gồm nhạy metric-linkage, chi phí khoảng cách/cây và khó xử lý sai gộp sớm. Single linkage dễ chaining; complete có thể nhạy điểm cực đoan. Không có linkage tốt nhất cho mọi biểu diễn review.

## 10. So sánh với phương pháp gần nhau

| Phương pháp | Khác biệt |
|---|---|
| K-means | Tối ưu quanh tâm ở một K, không xây cùng cây gộp |
| Ward | Một linkage trong họ phân cấp, không phải tên mọi hierarchical clustering |
| Decision Tree | Học split từ mục tiêu, khác cây khoảng cách |
| BERTopic hierarchy | Có thể phân cấp các topic đã có, khác trực tiếp phân cấp review |

Cây nhánh không tự là bằng chứng mô hình quản trị nhiều tầng.

## 11. Sai lầm thường gặp và cách khắc phục

Không dùng tham số affinity cũ nếu API cần metric. Không coi Ward-height luôn bằng tăng SSE: có quy ước hệ số trong phần mềm. Đừng tạo ma trận n×n dense rất lớn mà bỏ qua bộ nhớ. AgglomerativeClustering sklearn không có predict tiêu chuẩn cho mẫu mới; không gọi fit_predict trên dữ liệu mới như phép gán vào cây đã khóa.

## 12. Tài liệu tham khảo

- [S1] scikit-learn. [Hierarchical clustering](https://scikit-learn.org/1.8/modules/clustering.html).
- [S2] scikit-learn. [AgglomerativeClustering](https://scikit-learn.org/1.8/modules/generated/sklearn.cluster.AgglomerativeClustering.html).
- [S3] SciPy. [linkage](https://docs.scipy.org/doc/scipy/reference/generated/scipy.cluster.hierarchy.linkage.html).

Nguồn mô tả thuật toán/API và quy ước linkage. Ví dụ số và cách áp dụng khách sạn trong bài tự soạn; không gán một năm phát minh duy nhất cho toàn bộ họ phương pháp.

## 13. Vị trí trong toàn bộ nghiên cứu

Phương pháp nằm ở khám phá cấu trúc hoặc hỗ trợ xây nhóm mã nội dung. Các nhóm cần được xác thực bằng đọc review và đánh giá chuyên môn. Nếu dùng mức phân cấp làm biến phân tích tiếp, phải lưu cách cắt/đặt tên và phân biệt giai đoạn khám phá với giai đoạn xác nhận trên dữ liệu khác.

## 14. Kiến thức nền cần biết trước

Cần hiểu khoảng cách giữa mẫu, khoảng cách giữa tập, linkage, trung bình và SSE. Biết dendrogram thể hiện trình tự/độ cao gộp chứ không phải xác suất. Với văn bản, hiểu vector hóa và chi phí sparse/dense; chuyển một ma trận rất thưa sang dense có thể làm bộ nhớ tăng mạnh.

## 15. Nền tảng lý thuyết và logic hình thành

Linkage là định nghĩa “hai nhóm gần nhau” nên quyết định kiểu cấu trúc tìm được. Single dùng cặp gần nhất, complete dùng cặp xa nhất, average dùng trung bình cặp. Ward chọn cách tăng sai số nội cụm ít nhất trong Euclid. Mỗi lựa chọn ưu tiên hình dạng khác nhau; đây là thiết kế, không chỉ thao tác vẽ hình.

## 16. Các thành phần cấu tạo

Các thành phần gồm dữ liệu/metric, linkage, cấu trúc gộp, quy tắc tie, mức cắt và công cụ hiển thị. Connectivity graph, nếu có, giới hạn các phép gộp và có thể thay đổi kết quả. Không coi nó chỉ tăng tốc vô hại; phải giải thích cách xây đồ thị và phạm vi fit.

## 17. Các loại và biến thể

Có agglomerative/divisive và các linkage khác nhau. Có thể phân cấp mẫu hoặc phân cấp feature, hai câu hỏi khác nhau. Optimal leaf ordering thay thứ tự lá để dễ đọc, không tự thay quy tắc gộp thành một mô hình cảm xúc. Cắt cây lấy K nhóm không biến nó thành thuật toán K-means.

## 18. Đầu vào và đầu ra

Đầu vào có thể là vector hoặc ma trận khoảng cách hợp lệ tùy API. SciPy linkage với precomputed distances cần dạng condensed phù hợp; truyền nhầm ma trận vuông như feature có thể đổi bài toán. Đầu ra linkage matrix có các chỉ số gộp, độ cao và số lá; label ở mức cắt không phải nhãn cảm xúc.

## 19. Căn cứ lựa chọn

Đáng thử khi cấu trúc nhiều mức hữu ích và cỡ dữ liệu phù hợp. Chọn metric/linkage theo biểu diễn và mục đích; đánh giá tính đọc được và độ bền, không chỉ hình cây đẹp. Có thể làm baseline trên mẫu đại diện rồi kiểm tra xem kết luận có ổn khi đổi mẫu hay không.

## 20. Trường hợp không nên sử dụng

Không ưu tiên khi cần predict nhanh theo một cây đã khóa cho lượng review mới liên tục mà chưa có quy tắc bổ sung. Không dùng dendrogram quá nhiều lá như bằng chứng dễ hiểu. Nếu dữ liệu thay đổi lớn theo thời gian, cây toàn corpus có thể che dịch chuyển; cần thiết kế so sánh riêng.

## 21. Các quyết định quan trọng khi thực hiện

Chọn metric/linkage, mức cắt, xử lý tie, có connectivity không và cách lấy mẫu để vẽ. Nếu muốn gán review mới bằng tâm/medoid sau cắt, đó là quy tắc bổ sung cần định nghĩa và kiểm tra riêng. Fit lại cây với review mới thay đổi phân hoạch, không cùng thao tác predict giữ mô hình.

## 22. Giải thích công thức và ký hiệu

Ward chọn tăng SSE nhỏ:

$$\Delta(A,B)=\frac{|A||B|}{|A|+|B|}\|\mu_A-\mu_B\|_2^2.$$

Hai singleton cách 2 cho $\Delta=2$. Sau khi gộp hai cặp mục 7, mỗi nhóm có 2 mẫu, tâm cách 8 nên $\Delta=64$. Với quy ước SciPy Ward, độ cao tương ứng $\sqrt{2\Delta}$: lần gộp cuối là $\sqrt{128}$, không phải 64. Công thức khác linkage cần được đọc theo định nghĩa riêng.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mã kiểm tra Ward trên tọa độ, rồi average-cosine trên review. Ma trận dense chỉ dùng vì ví dụ rất nhỏ.

```python
import numpy as np
from scipy.cluster.hierarchy import linkage
from sklearn.cluster import AgglomerativeClustering
from sklearn.feature_extraction.text import TfidfVectorizer
points = np.array([[0.,0.],[0.,2.],[8.,0.],[8.,2.]])
Z = linkage(points, method="ward")
assert np.allclose(Z[:2, 2], [2.,2.])
assert np.isclose(Z[-1, 2], np.sqrt(128))
docs = ["phòng sạch giường êm", "phòng bẩn giường cũ", "phòng rộng giường đẹp",
        "nhân viên nhiệt tình lễ tân", "nhân viên chậm lễ tân", "nhân viên thân thiện lễ tân",
        "giá cao chi phí đắt", "giá thấp chi phí rẻ", "giá hợp lý chi phí"]
X = TfidfVectorizer(ngram_range=(1,2)).fit_transform(docs).toarray()
assert np.all(np.linalg.norm(X, axis=1) > 0)
model = AgglomerativeClustering(n_clusters=3, metric="cosine", linkage="average")
labels = model.fit_predict(X)
assert len(labels) == len(docs) and len(set(labels)) == 3
print("Ward heights:", Z[:,2])
print("Mã nhóm review:", labels)
```

Không có kiểm định rằng nhãn 0/1/2 là cảm xúc hoặc taxonomy chuẩn. Không dùng Ward với cosine trong ví dụ; hai minh họa dùng hai cấu hình được ghi rõ.

## 24. Tình huống sai và phản ví dụ

Một chuỗi review nối tiếp từng cặp hơi giống có thể khiến single linkage gom thành nhóm lớn dù hai đầu rất khác. Một review spam lặp nhiều lần có thể tạo nhánh cực chặt. Cả hai đều có cấu trúc số học rõ nhưng chưa mang ý nghĩa trải nghiệm khách sạn; cần kiểm tra trùng và mẫu ở các nhánh.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Thử độ nhạy linkage/metric và mức cắt trong protocol phát triển, kiểm tra kích thước nhóm và ổn định khi resample. So phân hoạch bằng chỉ số bất biến hoán vị. Cophenetic correlation nếu dùng chỉ nói về mức bảo toàn một cấu trúc khoảng cách, không chứng minh topic có ý nghĩa hay quan hệ nhân quả.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu cây khó đọc, có thể hiển thị rút gọn nhưng giữ báo cáo đầy đủ về mẫu/cắt. Nếu kết quả thay mạnh theo linkage, báo tính nhạy và kiểm tra biểu diễn thay vì chọn cây hợp giả thuyết nhất. Khi bộ nhớ không đủ, cân nhắc cách lấy mẫu/đại diện hoặc phương pháp khác và ghi giới hạn suy rộng.

## 27. Phân biệt kết quả với kết luận

Kết quả là cây và một hoặc nhiều phân hoạch theo lựa chọn cụ thể. Kết luận hợp lệ mô tả cấu trúc nội dung quan sát trong corpus. Không suy cây là thứ bậc nhân quả của trải nghiệm, hoặc các nhánh tương ứng nhóm khách tự nhiên cố định. Đặt tên nhánh là bước diễn giải cần bằng chứng.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu dùng agglomerative clustering với [metric], [linkage] trên [biểu diễn]. Mức cắt chọn bằng [...]; taxonomy được đặt tên qua [quy trình đọc].” Báo số mẫu, cách xử lý trùng, connectivity nếu có, kích thước nhóm và độ nhạy. Ghi dendrogram là toàn bộ hay mẫu con.

## 29. Khả năng tái lập và tính minh bạch

Lưu schema, vectorizer, ma trận/linkage nếu phù hợp tài nguyên, thứ tự mẫu, mức cắt và mapping tên nhóm. Thứ tự hàng ảnh hưởng diễn giải chỉ số gộp; không lưu ảnh cây mà bỏ mất dữ liệu cấu trúc. Mã đã chạy trên ví dụ tự tạo; chia sẻ review thật theo quyền sử dụng.

## 30. Đạo đức nghiên cứu liên quan

Không đặt tên nhóm xúc phạm hoặc gán động cơ người viết. Nhánh ít mẫu có thể làm lộ review cụ thể qua trích dẫn, nên ẩn danh và kiểm soát công bố. Không dùng cây nhóm để xác định cá nhân gian lận hay nhóm khách có đặc điểm nhạy cảm khi chưa có bằng chứng phù hợp.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Ward khác average linkage ở tiêu chí nào?
2. Với hai nhóm 2 mẫu có tâm cách 8, tính Δ.
3. Vì sao Ward-height SciPy không bằng Δ?
4. Fit_predict lại có phải predict giữ cây không?

Gợi ý: Δ=64, height=√128. Bài mở rộng: so mức cắt bằng đọc nội dung và độ bền, không chỉ hình vẽ.

## 32. Những điểm còn tranh luận

Các vấn đề cần nghiên cứu là lựa chọn mức phân cấp hữu ích, độ bền taxonomy khi thêm dữ liệu và giới hạn diễn giải dendrogram. Không có một cây duy nhất hoàn toàn khách quan. Các quyết định về biểu diễn, linkage và mức cắt cần minh bạch để người đọc đánh giá bằng chứng.

