# KNN — K láng giềng gần nhất cho phân loại cảm xúc

> **Nhánh:** MACHINE LEARNING → Supervised: Instance based. **Nguồn:** dòng 29, sheet “Công nghệ SA” của bảng Excel. **Đối chiếu:** 28/10/2026. Bài tập trung phân loại bằng k-nearest neighbors, phân biệt với k-means, KNN hồi quy và tìm kiếm tương đồng thuần túy. Các dữ liệu, tọa độ và tình huống khách sạn đều tự tạo. Mã minh họa được kiểm tra với scikit-learn 1.8.0.

| Trường | Nội dung |
|---|---|
| Bản chất | Phương pháp có giám sát dự đoán từ nhãn các mẫu train gần truy vấn trong không gian biểu diễn |
| Họ phương pháp | Instance-based learning; thường gọi là lazy learning |
| Mốc tham khảo | Cover & Hart, 1967: công trình nền tảng về nearest-neighbor classification [S1]; không coi đây là năm duy nhất khai sinh mọi biến thể |
| Đầu vào | Vector đặc trưng, nhãn, k, khoảng cách, quy tắc bỏ phiếu và dữ liệu tham chiếu train |
| Đầu ra | Nhãn, tỷ lệ phiếu hoặc tỷ lệ phiếu có trọng số; có thể truy xuất các láng giềng |
| Tiếng Việt | Cần TF-IDF, embedding hoặc biểu diễn khác; khoảng cách không tự nhận biết phủ định hay mỉa mai |
| Phần mềm | sklearn.neighbors.KNeighborsClassifier; ví dụ văn bản dùng cosine và brute force |
| Tài nguyên | Cần giữ dữ liệu tham chiếu; chi phí dự đoán và lưu trữ có thể đáng kể khi tập train lớn |

## 1. Định nghĩa học thuật

K-nearest neighbors classification là phương pháp dự đoán nhãn của một quan sát mới dựa trên nhãn của k quan sát gần nó nhất trong dữ liệu tham chiếu, theo biểu diễn và hàm khoảng cách đã xác định. Quy tắc tổng hợp thường là đa số phiếu hoặc phiếu có trọng số theo khoảng cách.

KNN thuộc nhóm có giám sát vì cần nhãn của các mẫu tham chiếu để phân loại. Việc tìm láng giềng tự nó có thể không dùng nhãn, nhưng tìm kiếm đó chưa phải toàn bộ bộ phân loại. KNN cũng thường được gọi là phi tham số: không giả định trước một dạng hàm hữu hạn như hồi quy tuyến tính, nhưng vẫn có siêu tham số và lựa chọn thiết kế.

## 2. Giải thích đơn giản

Khi gặp một đánh giá khách sạn mới, KNN tìm những đánh giá đã gán nhãn có vector giống nó nhất, rồi dùng nhãn của nhóm đó để dự đoán. Nếu trong ba láng giềng có hai tích cực và một tiêu cực, bỏ phiếu đồng đều sẽ chọn tích cực.

“Gần” là gần theo phép đo trên vector, không đồng nghĩa con người thấy hai câu có cùng ý nghĩa. “Phòng sạch” và “phòng không sạch” có nhiều từ chung nhưng cảm xúc trái nhau. Nếu biểu diễn không làm rõ phủ định, KNN có thể lấy nhầm nhóm tham chiếu dù việc tính khoảng cách hoàn toàn đúng.

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu cảm xúc review khách sạn, KNN có thể làm baseline dựa trên tương đồng hoặc kiểm tra mức độ phân tách cảm xúc của một biểu diễn. Nếu TF-IDF hoặc embedding đặt các câu cùng nhãn gần nhau, KNN có thể khai thác cấu trúc đó để dự đoán.

Một câu hỏi thực nghiệm phù hợp là: “Với cùng biểu diễn và split, KNN có dự đoán tốt các đánh giá ở khách sạn chưa gặp không?” Một câu hỏi khác là liệu việc đưa các review tham chiếu ra cho người rà soát có giúp tìm kiểu lỗi. Không mặc định truy xuất được ví dụ nghĩa là đã có lời giải thích nhân quả hay ngữ nghĩa đầy đủ.

## 4. Điều kiện và giả định sử dụng

Điều kiện quan trọng là tính gần trong biểu diễn có liên hệ hữu ích với nhãn. Không cần mọi biến độc lập, nhưng feature vô nghĩa, thang đo lệch hoặc metric không phù hợp có thể làm nhóm láng giềng mất giá trị. Cần đủ mẫu tham chiếu cho các kiểu câu và lớp dự kiến gặp.

Phải có k không lớn hơn số mẫu tham chiếu trong từng lượt fit của quá trình đánh giá. Nhãn, thứ tự cột và biến đổi đầu vào phải nhất quán. Với cosine theo công thức thông thường, vector phải khác 0; phần mềm có thể có quy ước cho vector 0, nhưng quy ước số học không tạo ra thông tin ngôn ngữ bị thiếu.

## 5. Cơ chế hoạt động bên trong

Ở bước fit, KNN cơ bản lưu dữ liệu tham chiếu và nhãn, có thể xây cấu trúc chỉ mục tùy thuật toán tìm kiếm. Nó không học một chuỗi cây hoặc một vector hệ số phân loại theo cách các phương pháp đã học trước đó. Tuy nhiên, vectorizer, scaler hoặc embedding được huấn luyện có thể đã học thông tin ở các bước trước.

Khi dự đoán, pipeline biến đổi câu mới bằng bộ biến đổi đã khóa, tính hoặc tìm các khoảng cách đến train, chọn k láng giềng rồi tổng hợp nhãn. Brute force xét trực tiếp dữ liệu tham chiếu; các cấu trúc cây tìm kiếm có điều kiện hiệu quả riêng. Trong cấu hình văn bản của bài, dùng sparse TF-IDF với cosine và brute force, không giả định KD-tree tăng tốc được trường hợp này [S2–S3].

## 6. Quy trình áp dụng trong nghiên cứu

1. Quy định đơn vị phân tích, bộ nhãn và cách xử lý câu hỗn hợp/trung tính.
2. Loại review trùng/gần trùng và xác định phụ thuộc theo khách sạn, người viết hoặc thời gian.
3. Chia train/development/test theo đích suy rộng; giữ test độc lập.
4. Fit vocabulary, IDF, scaler hoặc bộ biến đổi được học chỉ trên train của từng fold.
5. Chọn biểu diễn, khoảng cách, k và weighting bằng development/CV phù hợp.
6. Kiểm tra các láng giềng của lỗi điển hình, câu vector 0, khoảng cách hòa và lớp hiếm.
7. Khóa cấu hình, đánh giá test và báo cả chất lượng, độ bất định, chi phí dự đoán.

Nếu muốn sử dụng toàn bộ văn bản không nhãn, gồm cả test, để học biểu diễn, phải công bố thiết kế transductive đó. Không trình bày như mô hình chỉ học từ train rồi đánh giá hoàn toàn trên dữ liệu chưa từng tham gia xây hệ thống.

## 7. Ví dụ minh họa xuyên suốt

Tình huống giả định: phân loại review khách sạn thành tích cực (1) hoặc tiêu cực (0). Phần văn bản ở mục 23 dùng những câu như “phòng sạch đẹp” và “dịch vụ tệ bẩn”. Ví dụ chưa có lớp trung tính; khi nghiên cứu thực tế phải quy định lớp này trước.

Để nhìn rõ phép tính, xét một truy vấn có tọa độ q=(2,1) trong không gian hai chiều tự tạo. Tọa độ chỉ phục vụ toán học, không phải số từ khen/chê hoặc kết quả embedding đo thật.

| Mẫu train | Tọa độ | Nhãn | Khoảng cách Euclid đến q |
|---|---|---:|---:|
| A | (2,2) | 1 | 1 |
| B | (1,1) | 1 | 1 |
| C | (2,3) | 0 | 2 |
| D | (5,1) | 0 | 3 |
| E | (2,5) | 0 | 4 |

Với k=3, chọn A, B, C. Hai mẫu tích cực thắng một mẫu tiêu cực nếu bỏ phiếu đồng đều. Hòa khoảng cách giữa A và B không làm thay đổi tập ba mẫu vì cả hai đều nằm trong tập được chọn; hòa tại ranh giới mẫu thứ k và k+1 mới đặt ra vấn đề chọn tập láng giềng.

## 8. Kết quả và cách diễn giải

Với bỏ phiếu đồng đều, tỷ lệ lớp tích cực ở ví dụ là 2/3. Với trọng số nghịch khoảng cách, tỷ lệ là (1+1)/(1+1+1/2)=0.8. Cả hai cùng chọn tích cực nhưng không đưa cùng con số. Phải ghi cách weighting khi giải thích đầu ra.

Các tỷ lệ này là ước lượng từ nhóm láng giềng theo quy tắc đã chọn, không phải bảo đảm xác suất đúng 66.7% hoặc 80% ngoài thực tế. Ba láng giềng đều tích cực vẫn có thể cùng bị lệch miền hoặc dùng chung một mẫu câu sai. Nếu sử dụng xác suất cho quyết định, cần đánh giá calibration và chất lượng ngoài mẫu riêng.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là nguyên tắc dễ hiểu, không bắt buộc ranh giới tuyến tính và có thể xem trực tiếp các mẫu tham chiếu. Một bộ phân loại đơn giản trên embedding cố định cũng giúp đánh giá biểu diễn mà không thêm một classifier quá phức tạp.

Nhược điểm là nhạy với metric, thang đo, nhãn nhiễu, lớp mất cân bằng và cấu trúc mật độ. Dữ liệu nhiều chiều có thể làm tương đồng kém phân biệt; việc lưu train và tìm kiếm khi dự đoán có thể tốn tài nguyên. Tính đơn giản của quy tắc không có nghĩa toàn bộ hệ thống rẻ, dễ diễn giải hoặc bảo vệ riêng tư tự động.

Với dữ liệu dense, brute force tính khoảng cách từ một truy vấn đến n mẫu d chiều thường cần công việc cỡ O(nd), chưa kể chọn láng giềng và các chi tiết triển khai. Đây không phải chi phí O(n²d) của việc tính tất cả cặp trong n mẫu. Dữ liệu sparse, batching và phần cứng ảnh hưởng chi phí thực; cần đo khi triển khai.

## 10. So sánh với phương pháp gần nhau

| Phương pháp/khái niệm | Điểm khác với KNN phân loại |
|---|---|
| K-means | Phân cụm không giám sát quanh các tâm; k là số cụm, không phải số phiếu láng giềng |
| Nearest centroid | Đại diện mỗi lớp bằng tâm; không giữ nguyên cơ chế bỏ phiếu của k mẫu gần |
| Logistic Regression | Học tham số ranh giới theo feature thay vì dự đoán chủ yếu từ lân cận từng truy vấn |
| SVM | Học hàm quyết định dựa trên tối ưu margin; không đồng nhất support vectors với k láng giềng |
| Radius neighbors | Lấy các mẫu trong bán kính; số láng giềng thay đổi và có thể bằng 0 |
| Truy xuất tương đồng | Trả về các mẫu gần; chưa có nhãn/quy tắc tổng hợp thì chưa thành classifier |

Approximate nearest-neighbor search là cách tìm kiếm xấp xỉ, không tự tạo một thuật toán phân loại mới. Nó có thể thay đổi tập mẫu được chọn và vì vậy cần đánh giá tác động lên kết quả.

## 11. Sai lầm thường gặp và cách khắc phục

Một lỗi là chọn k theo một quy tắc như căn bậc hai số mẫu rồi coi đó là tối ưu. Đó chỉ có thể là điểm khởi đầu; cần chọn bằng development/CV, bảo đảm mọi giá trị k hợp lệ trong từng fold. K lẻ cũng không loại mọi trường hợp hòa, nhất là đa lớp hoặc có trọng số.

Lỗi khác gồm fit TF-IDF trên cả test, chuẩn hóa mọi dữ liệu trước split, đánh giá train có chính mẫu làm láng giềng, hoặc để review gần trùng nằm ở hai tập. Cách khắc phục là khóa phạm vi fit, kiểm soát phụ thuộc mẫu và đánh giá trên quan sát giữ lại. Kiểm tra câu vector 0 và chính sách xử lý trước khi đọc điểm test.

## 12. Tài liệu tham khảo

- [S1] Cover, T. M., & Hart, P. E. (1967). *Nearest Neighbor Pattern Classification*. IEEE Transactions on Information Theory, 13(1), 21–27. [DOI](https://doi.org/10.1109/TIT.1967.1053964), [bản từ trang tác giả tại Stanford](https://isl.stanford.edu/~cover/papers/transIT/0021cove.pdf).
- [S2] scikit-learn. [KNeighborsClassifier, phiên bản 1.8](https://scikit-learn.org/1.8/modules/generated/sklearn.neighbors.KNeighborsClassifier.html).
- [S3] scikit-learn. [Nearest Neighbors, hướng dẫn phiên bản 1.8](https://scikit-learn.org/1.8/modules/neighbors.html).
- [S4] scikit-learn. [TfidfVectorizer, phiên bản 1.8](https://scikit-learn.org/1.8/modules/generated/sklearn.feature_extraction.text.TfidfVectorizer.html).

[S1] là tài liệu nền tảng lý thuyết, không phải cam kết hiệu năng cho dữ liệu hữu hạn của người dùng. [S2–S4] phục vụ đối chiếu phần mềm và biểu diễn. Những ví dụ số, mã, tình huống và gợi ý thiết kế nghiên cứu trong bài được tự soạn; chưa có kết quả thực nghiệm trên review thật của người dùng.

## 13. Vị trí trong toàn bộ nghiên cứu

KNN xuất hiện ở khâu học và đánh giá mô hình sau khi xác định câu hỏi, nguồn dữ liệu, nhãn và biểu diễn. Khi dùng nó để so sánh embedding, cần giữ protocol tương đương để không trộn hiệu quả biểu diễn với hiệu quả lấy mẫu hoặc tuning.

Nếu đầu ra được tổng hợp thành tỷ lệ phản hồi tiêu cực theo khách sạn, cần phân tích sai số phân loại và tính đại diện của review. Một bộ phân loại tốt không tự chứng minh tỷ lệ dự đoán là thước đo hợp lệ của sự hài lòng toàn bộ khách lưu trú.

## 14. Kiến thức nền cần biết trước

Nên hiểu vector, chuẩn vector, khoảng cách Euclid, cosine, phép bỏ phiếu và train/development/test. Với văn bản, cần biết vocabulary, TF-IDF, n-gram, sparse matrix và từ ngoài từ điển. Với biến cấu trúc, cần hiểu thang đo và mã hóa categorical.

Một điểm dễ bỏ qua: TfidfVectorizer mặc định tách theo token, không phải một bộ phân đoạn từ tiếng Việt theo ngữ nghĩa. Một từ đa âm tiết có thể bị tách thành nhiều token. Việc dùng n-gram có thể giữ một số cụm, nhưng không đồng nghĩa đã giải quyết đầy đủ phân đoạn từ, phủ định hoặc ngữ cảnh.

## 15. Nền tảng lý thuyết và logic hình thành

Logic của phương pháp là dùng lân cận trong biểu diễn để suy ra nhãn cục bộ: quan sát gần nhau được kỳ vọng cung cấp thông tin về phân bố nhãn quanh truy vấn. Khi k nhỏ, dự đoán dễ nhạy với vài mẫu; khi k lớn, lân cận rộng hơn có thể làm mờ các cấu trúc cục bộ hữu ích.

Kết quả tiệm cận trong lý thuyết nearest-neighbor có điều kiện và xét những chế độ dữ liệu cụ thể [S1]. Không diễn dịch thành “KNN hữu hạn luôn gần tối ưu”, “1-NN luôn đạt Bayes error” hoặc “k càng lớn càng tốt”. Khả năng suy rộng thực tế còn phụ thuộc biểu diễn, kích thước mẫu, nhiễu và thay đổi miền.

## 16. Các thành phần cấu tạo

Một hệ thống KNN phân loại gồm: bộ biểu diễn; tập vector/nhãn tham chiếu; hàm khoảng cách; k; thuật toán tìm kiếm; trọng số phiếu; quy tắc xử lý hòa; và chính sách xử lý đầu vào không đủ thông tin. Pipeline nghiên cứu còn có split, metric và bộ kiểm tra rò rỉ.

Phân biệt k với số feature, số lớp và số cụm. Phân biệt weighting theo khoảng cách với class weighting: cấu hình weights='distance' không tự cân bằng lớp hiếm. Một lớp đông có thể chiếm nhiều vị trí trong lân cận nếu biểu diễn không tách được nó.

## 17. Các loại và biến thể

Các lựa chọn thường gặp là 1-NN, k-NN với phiếu đồng đều, k-NN có trọng số và radius-neighbor classification. KNN hồi quy tổng hợp giá trị liên tục thay vì bỏ phiếu nhãn; không gọi tỷ lệ phiếu nhị phân là toàn bộ cơ chế hồi quy.

Có thể dùng metric được thiết kế hoặc học, giảm chiều trước tìm kiếm, hay tìm láng giềng xấp xỉ. Nếu các bước đó học từ dữ liệu, chúng thuộc quy trình cần tránh leakage. Cấu hình khác nhau có thể thay đổi ý nghĩa của “gần”; không so điểm số mà bỏ qua biểu diễn và metric.

## 18. Đầu vào và đầu ra

Train có X kích thước n×d và nhãn y gồm n phần tử. Truy vấn có cùng d cột và cùng phép biến đổi. Với precomputed distances, cần tuân thủ định dạng và quan hệ hàng/cột của API; ma trận khoảng cách không phải ma trận feature thông thường.

Đầu ra nhãn có thể kèm tỷ lệ phiếu và ID/khoảng cách láng giềng. Các cột predict_proba tuân theo classes_, không theo suy đoán rằng cột đầu luôn tích cực. Khi trình bày review tham chiếu, cần quản lý quyền truy cập vì mô hình đang sử dụng dữ liệu train ở mức từng mẫu.

## 19. Căn cứ lựa chọn

KNN đáng thử khi tính tương đồng của biểu diễn có lý do phù hợp, tập train có thể lưu/tìm kiếm với chi phí chấp nhận được và việc xem các mẫu gần hữu ích cho rà soát. Nó cũng phù hợp làm một baseline kiểm tra chất lượng hình học của biểu diễn.

Lựa chọn cuối cần dựa trên chất lượng ngoài mẫu, mức ổn định, lỗi lớp quan trọng và tài nguyên. Có thể so TF-IDF+KNN với TF-IDF+Logistic Regression, rồi so thêm embedding+KNN. Thiết kế này giúp tách giá trị của biểu diễn khỏi giá trị của bộ phân loại.

## 20. Trường hợp không nên sử dụng

Không nên ưu tiên nếu khoảng cách chủ yếu phản ánh chủ đề khách sạn hoặc độ dài thay vì cảm xúc; nếu dữ liệu tham chiếu không thể được lưu an toàn; hoặc nếu yêu cầu dự đoán với độ trễ/bộ nhớ rất chặt mà chưa có giải pháp được kiểm tra.

Không nên tiếp tục tuning KNN để che đi vấn đề gán nhãn hoặc mục tiêu sai. Nếu câu chứa nhiều khía cạnh khen/chê và mục tiêu là từng khía cạnh, bỏ phiếu cho một nhãn toàn văn có thể không trả lời câu hỏi nghiên cứu dù chương trình chạy đúng.

## 21. Các quyết định quan trọng khi thực hiện

Cần chọn biểu diễn, metric, k, weighting, thuật toán tìm kiếm và cách xử lý hòa. Với feature số kết hợp như giá phòng và số từ chê, phải cân nhắc thang đo để một biến không chi phối khoảng cách chỉ vì đơn vị lớn. Fit scaler trong train; không mặc định StandardScaler cần áp dụng giống nhau cho sparse TF-IDF đã chuẩn hóa L2.

Cần chốt chính sách cho vector 0, câu gần nhất vẫn rất xa, category/miền mới và dữ liệu trùng. Nếu dùng ngưỡng từ chối dự đoán, chọn bằng development và báo tỷ lệ được dự đoán cùng hiệu năng trên phần đó. Không chỉ bỏ câu khó khỏi test rồi báo điểm của phần còn lại như toàn bộ hệ thống.

## 22. Giải thích công thức và ký hiệu

Với hai vector x,z có d chiều:

$$d_E(x,z)=\sqrt{\sum_{j=1}^{d}(x_j-z_j)^2}.$$

Với vector khác 0, khoảng cách cosine dùng trong bài:

$$d_{cos}(x,z)=1-\frac{x^\top z}{\|x\|_2\|z\|_2}.$$

Ký hiệu N_k(x) là tập chỉ số k mẫu được chọn. Tỷ lệ phiếu cho lớp c:

$$\widehat p(c\mid x)=\frac{\sum_{i\in N_k(x)}w_i\mathbf{1}[y_i=c]}{\sum_{i\in N_k(x)}w_i},\qquad \widehat y=\operatorname*{argmax}_c\widehat p(c\mid x).$$

Bỏ phiếu đồng đều lấy w_i=1. Trọng số nghịch khoảng cách w_i=1/d(x,x_i) chỉ dùng trực tiếp khi khoảng cách dương; phải có quy tắc riêng cho khoảng cách 0. Trong sklearn 1.8 với weights='distance', nếu có láng giềng khoảng cách 0, các mẫu đó nhận toàn bộ phần phiếu trong nhóm được xét, thay vì tính 1/0 [S2].

Với vector đơn vị khác 0, $\|x-z\|_2^2=2d_{cos}(x,z)$. Vì vậy thứ tự gần theo hai phép đo này tương ứng, nhưng trọng số 1/d không bằng nhau sau đổi metric. Cùng tập láng giềng không bảo đảm cùng tỷ lệ phiếu có trọng số.

## 23. Ví dụ tính toán hoặc thao tác từng bước

**A. Tính trên tọa độ ở mục 7.** Mã kiểm tra khoảng cách và hai quy tắc phiếu. Có hòa A/B nhưng không có hòa tại ranh giới k=3.

```python
import numpy as np
from sklearn.neighbors import KNeighborsClassifier

X = np.array([[2., 2.], [1., 1.], [2., 3.], [5., 1.], [2., 5.]])
y = np.array([1, 1, 0, 0, 0])
q = np.array([[2., 1.]])
distances = np.linalg.norm(X - q[0], axis=1)
assert np.allclose(distances, [1, 1, 2, 3, 4])

for weights, expected in [("uniform", 2/3), ("distance", 0.8)]:
    knn = KNeighborsClassifier(n_neighbors=3, metric="euclidean",
                              algorithm="brute", weights=weights)
    knn.fit(X, y)
    dist, idx = knn.kneighbors(q)
    assert set(idx[0]) == {0, 1, 2}
    positive_col = list(knn.classes_).index(1)
    p = knn.predict_proba(q)[0, positive_col]
    assert np.isclose(p, expected)
    assert knn.predict(q)[0] == 1
    print(weights, "khoảng cách:", dist[0], "tỷ lệ phiếu lớp 1:", p)
```

**B. Pipeline văn bản tiếng Việt.** Dùng TF-IDF trên train rồi KNN cosine, k=3. Tám câu train và bốn câu kiểm tra đều tự tạo; chưa có tập development nên k được đặt để minh họa, không phải lựa chọn tối ưu từ một nghiên cứu.

```python
import numpy as np
from sklearn.pipeline import make_pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import f1_score

train = ["phòng sạch đẹp", "nhân viên tốt", "dịch vụ tốt sạch",
         "khách sạn đẹp sạch", "phòng bẩn tệ", "nhân viên tệ",
         "dịch vụ tệ bẩn", "khách sạn bẩn tệ"]
y = np.array([1, 1, 1, 1, 0, 0, 0, 0])
test = ["phòng sạch zzholdout", "dịch vụ tốt", "phòng bẩn", "dịch vụ tệ"]
yt = np.array([1, 1, 0, 0])
pipe = make_pipeline(
    TfidfVectorizer(ngram_range=(1, 2), norm="l2"),
    KNeighborsClassifier(n_neighbors=3, metric="cosine",
                         algorithm="brute", weights="uniform"))
pipe.fit(train, y)
vec, knn = pipe[0], pipe[1]
Xt = vec.transform(test)
assert "zzholdout" not in vec.vocabulary_
assert np.all(Xt.getnnz(axis=1) > 0)
prob = pipe.predict_proba(test)
assert np.isfinite(prob).all() and np.allclose(prob.sum(axis=1), 1)

# Kiểm tra xác suất chính là tỷ lệ phiếu đồng đều.
dist, idx = knn.kneighbors(Xt)
positive_col = list(knn.classes_).index(1)
manual = (y[idx] == 1).mean(axis=1)
assert np.allclose(prob[:, positive_col], manual)
print("Macro-F1 trên dữ liệu đồ chơi:",
      f1_score(yt, pipe.predict(test), average="macro"))
print("ID láng giềng của câu đầu:", idx[0])
print("Khoảng cách cosine:", dist[0])

# Câu hoàn toàn ngoài vocabulary: phát hiện trước khi dự đoán.
unseen = vec.transform(["zzunknown"])
needs_review = unseen.getnnz(axis=1) == 0
assert needs_review[0]
if needs_review[0]:
    print("Chưa đủ đặc trưng: chuyển rà soát theo chính sách minh họa.")
else:
    print(knn.predict(unseen))
```

Mã không fit lại vocabulary trên test và không tự thêm từ mới khi predict. Kiểm tra vector 0 chỉ xử lý một trường hợp thiếu thông tin; câu có vài token quen nhưng rất khác miền vẫn có thể cần từ chối theo tiêu chí khác. Không suy từ điểm số của bốn câu thành hiệu năng thực tế.

## 24. Tình huống sai và phản ví dụ

**Phủ định:** “Phòng sạch” và “phòng không sạch” có thể gần nhau nếu biểu diễn nhấn mạnh từ chung. KNN đúng về phép tính nhưng sai về nhãn. Cần đánh giá n-gram, chỉ báo phủ định hoặc biểu diễn ngữ cảnh bằng tập thách thức độc lập.

**Trùng review:** bản sao một đánh giá được đưa vào train và test tạo láng giềng gần như trùng tuyệt đối, làm kết quả có vẻ rất tốt. Nếu đổi sang khách sạn hoặc thời gian mới, lợi thế ghi nhớ đó có thể biến mất.

**Câu không có feature:** với vector toàn 0, thư viện vẫn có thể trả về khoảng cách và nhãn theo quy ước. Kết quả đó không chứng minh câu có cảm xúc trung tính hoặc giống các review được trả về. Cần phát hiện và áp dụng chính sách được định trước.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Kiểm tra macro-F1, precision/recall từng lớp, confusion matrix và số mẫu trong từng nhóm. Đánh giá riêng phủ định, câu hỗn hợp, câu dài/ngắn, khách sạn chưa gặp và thời gian giữ lại. Kiểm tra tỷ lệ vector 0 và phân bố khoảng cách đến láng giềng gần nhất.

Đo độ nhạy với k, metric, weighting và biểu diễn trên development. Kiểm tra mức hòa khoảng cách tại ranh giới vì thứ tự train có thể ảnh hưởng tập được chọn [S2]. Khi báo khoảng bất định, giữ đơn vị phụ thuộc của thiết kế, chẳng hạn bootstrap theo khách sạn khi phù hợp, thay vì mặc định mọi review độc lập.

Nếu dùng tìm kiếm xấp xỉ, kiểm tra cả mức tìm lại láng giềng so với exact search trên mẫu đối chứng và metric phân loại cuối. Nếu có từ chối dự đoán, báo coverage, lỗi và chi phí xử lý phần bị từ chối để tránh báo cáo chọn lọc.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu train cao mà dữ liệu giữ lại thấp, kiểm tra trùng/lệch miền trước, rồi xem k quá nhỏ, noise và lựa chọn biểu diễn. Nếu mọi cấu hình đều kém, xem lại liệu khoảng cách đang phản ánh chủ đề thay vì cảm xúc; tăng k không sửa được mọi biểu diễn sai.

Nếu lớp hiếm bị bỏ sót, kiểm tra lân cận, nhãn, cách lấy mẫu và metric lựa chọn. Bất kỳ resampling hoặc học metric nào cũng chỉ thực hiện trong train từng fold. Với chi phí dự đoán lớn, cân nhắc giảm chiều, cấu trúc tìm kiếm hoặc mô hình khác; các bước được học cần đánh giá lại đúng protocol và không dùng test để chọn phương án.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ: “TF-IDF+KNN đạt macro-F1 X trên các khách sạn giữ lại, với k=Y và metric Z đã chọn bằng development.” Nếu có đối chứng, có thể kết luận mức hữu ích dự đoán trong phạm vi thiết kế đó, kèm độ bất định.

Không suy ra hai review gần nhau có cùng nguyên nhân cảm xúc, cùng tác giả hoặc cùng trải nghiệm thật. Không kết luận một thuộc tính khách sạn gây bất mãn chỉ vì nhiều láng giềng tiêu cực nhắc tới nó. Cũng không suy rộng tỷ lệ nhãn trong tập review thuận tiện thành tỷ lệ hài lòng toàn dân số khách du lịch.

## 28. Cách viết vào bài nghiên cứu

Mẫu phương pháp: “Nghiên cứu sử dụng k-nearest neighbors để phân loại [bộ nhãn]. Biểu diễn [TF-IDF/embedding] được xây dựng theo [phạm vi fit]. K, khoảng cách và weighting được chọn trên [development/CV theo nhóm]. Test gồm [đơn vị giữ lại] và chỉ được dùng sau khi khóa cấu hình.”

Bổ sung k cuối, metric, weighting, algorithm, quy tắc xử lý hòa/vector 0, phiên bản và tài nguyên. Nếu embedding được học ngoài tập nghiên cứu, mô tả nguồn và giới hạn thông tin đã biết. Bảng kết quả cần theo lớp, coverage khi có từ chối và độ trễ nếu liên quan; chỉ điền số thực nghiệm thật.

## 29. Khả năng tái lập và tính minh bạch

Lưu codebook nhãn, split ID, quy tắc loại trùng, thứ tự mẫu train, vectorizer/IDF hoặc embedding, mapping nhãn, k, metric, weighting, phiên bản và schema. KNN cần dữ liệu tham chiếu để predict; chỉ lưu k không đủ tái tạo mô hình.

Nếu có giảm chiều/học metric hoặc tìm kiếm xấp xỉ, lưu cả tham số, chỉ mục và seed tương ứng. Thay dữ liệu tham chiếu là thay hệ thống dự đoán, cần ghi phiên bản và đánh giá lại. Mã mục 23 kiểm tra được thao tác và công thức, không thay kết quả thực nghiệm trên bộ dữ liệu thật của người dùng.

## 30. Đạo đức nghiên cứu liên quan

KNN gắn với các mẫu tham chiếu cụ thể nên nguy cơ lộ review train cần được quản lý cẩn thận. Không công khai hàng xóm gần nhất có tên, số liên hệ hoặc chi tiết có thể truy ngược nếu không có quyền phù hợp. Cân nhắc quyền lưu dữ liệu và quy trình xử lý yêu cầu xóa.

Không dùng tính giống nhau của vector để quy kết người viết sao chép, giả mạo hoặc có cùng danh tính khi chưa có bằng chứng độc lập. Với ứng dụng ưu tiên khiếu nại, cần đánh giá lỗi theo nhóm/bối cảnh có dữ liệu hợp lệ và có người rà soát trước các quyết định gây bất lợi cho khách hoặc nhân viên.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. K trong KNN khác k trong k-means như thế nào?
2. Từ bảng mục 7, tính tỷ lệ phiếu lớp 1 với k=3, đồng đều và nghịch khoảng cách.
3. Vì sao k lẻ không bảo đảm hết mọi trường hợp hòa?
4. Nếu cả câu mới ngoài vocabulary, vì sao nhãn thư viện trả về chưa đủ cơ sở tin cậy?
5. Thiết kế split để dự đoán review của khách sạn chưa có trong train.
6. Vì sao Euclid và cosine có cùng thứ tự trên vector đơn vị nhưng weights='distance' có thể cho tỷ lệ phiếu khác?

Gợi ý: câu 2 cho 2/3 và 0.8; câu 5 cần giữ nhóm khách sạn. Với câu 6, kiểm tra quan hệ bình phương khoảng cách ở mục 22. Bài mở rộng: so KNN với Logistic Regression trên cùng biểu diễn và split, xác định ngân sách tuning trước; báo cả chất lượng và chi phí, không lấy dữ liệu đồ chơi làm benchmark.

## 32. Những điểm còn tranh luận

Các vấn đề cần kiểm chứng thực nghiệm gồm cách xây khoảng cách phù hợp cảm xúc tiếng Việt, vai trò biểu diễn so với classifier và ảnh hưởng của mật độ/lớp hiếm trong không gian nhiều chiều. Không có một metric hoặc k tối ưu chung cho mọi nguồn review.

Một đánh đổi khác là khả năng xem ví dụ tham chiếu với nguy cơ diễn giải quá mức và lộ dữ liệu. Tìm kiếm xấp xỉ, giảm chiều và cơ chế từ chối cũng tạo đánh đổi giữa chất lượng, tài nguyên và coverage. Nghiên cứu nên công bố điều kiện đánh giá để người đọc hiểu khi nào phương pháp hữu ích và khi nào bằng chứng còn thiếu.
