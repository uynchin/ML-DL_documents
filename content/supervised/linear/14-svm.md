# SVM: giải thích đơn giản

Nhánh Machine Learning, phần học có giám sát tuyến tính. Bài này giữ cấu trúc 10 mục như các file Lexicon Based, đổi mục 4 từ cách tính điểm sang cách học và cách đọc điểm, vì SVM không có từ điển điểm sẵn.

**Mục lục**

- [1. SVM là gì](#muc-1)
- [2. So sánh với các cách trước](#muc-2)
- [3. Ai làm gì](#muc-3)
- [4. Cách học và cách đọc điểm](#muc-4)
- [5. Ví dụ năm tình huống](#muc-5)
- [6. Lỗi hay gặp](#muc-6)
- [7. Làm sao biết kết quả đáng tin](#muc-7)
- [8. Chọn cái nào](#muc-8)
- [9. Mã chạy thử](#muc-9)
- [10. Tài liệu tham khảo](#muc-10)

<a id="muc-1"></a>

**1. SVM là gì**

SVM, tên đầy đủ là máy vector hỗ trợ, là một chương trình học từ các ví dụ đã có nhãn rồi dùng điều đã học để đoán nhãn cho văn bản mới [S1] [S2]. Ví dụ có nhãn là các đánh giá khách sạn mà người đã ghi là khen hoặc chê.

Có ba bước trong đầu của chương trình.

1. Đổi mỗi đánh giá thành một dãy số. Cách phổ biến là TF-IDF: mỗi từ hoặc cụm từ có một con số, từ xuất hiện nhiều trong đánh giá này nhưng ít gặp ở các đánh giá khác thì có số lớn hơn.
2. Học một bộ trọng số, mỗi từ một trọng số, cộng thêm một số điều chỉnh chung gọi là b. Từ nghiêng về khen có trọng số dương, từ nghiêng về chê có trọng số âm.
3. Với đánh giá mới, nhân trọng số với dãy số của đánh giá rồi cộng lại. Kết quả gọi là điểm quyết định. Điểm nằm bên nào của số 0 thì chọn nhãn bên đó.

Phần tên gọi nói về cách chương trình tìm trọng số: không chỉ tìm đường phân chia đúng với dữ liệu học mà còn giữ một khoảng trống quanh đường đó, gọi là biên. Khi dữ liệu có nhiễu hoặc hai nhóm lẫn vào nhau, chương trình chấp nhận một số điểm vi phạm biên để tránh bám quá sát các mẫu khó [S1].

Khác với các cách Lexicon Based, SVM không có từ điển điểm dựng sẵn. Trọng số của từng từ do máy học từ nhãn của người. Nghĩa là chất lượng nhãn quyết định chất lượng mô hình.

Có ba điều dễ nhầm.

1. SVM không luôn tuyến tính. Dùng kernel thì ranh giới có thể cong. Bài này chỉ nói về SVM tuyến tính [S1] [S3].
2. Điểm quyết định không phải xác suất. Điểm 0.7 không có nghĩa chắc chắn 70 phần trăm.
3. Trọng số dương của một từ không chứng minh từ đó gây ra sự hài lòng. Trọng số chỉ cho biết từ đó giúp phân biệt hai nhãn trong dữ liệu đã học.

<a id="muc-2"></a>

**2. So sánh với các cách trước**

Thông tin về các cách Lexicon Based lấy từ các file trước. Thông tin về các cách học máy gần nhau lấy từ tài liệu nguồn đã đối chiếu.

**So với các cách dùng từ điển.**

| Điểm so sánh | Các cách từ điển như VADER, AFINN, SentiStrength | SVM tuyến tính |
|---|---|---|
| Điểm của từ đến từ đâu | Có sẵn trong từ điển, do tác giả hoặc người đánh giá gán | Máy học từ nhãn của người trên dữ liệu của nghiên cứu |
| Cần nhãn để chạy không | Không, nhưng cần nhãn để kiểm tra | Có, cần nhãn để huấn luyện và thêm nhãn để kiểm tra |
| Từ ngoài từ điển | Bị bỏ qua, không có điểm | Từ chưa thấy khi học bị bỏ qua trong vector |
| Học từ ngữ riêng của khách sạn | Không, trừ khi người tự thêm | Có, nếu từ đó xuất hiện đủ trong dữ liệu học |
| Xử lý "not" | Tùy cách, VADER và SentiStrength có sẵn | Không có sẵn. Học được nếu dùng cụm hai từ như "not good" và cụm đó có trong dữ liệu học |
| Có luôn ra nhãn không | Có thể ra thiếu dữ liệu khi không khớp từ nào | Luôn ra một nhãn, kể cả khi câu không có từ cảm xúc |
| Xem lý do | Xem điểm từng từ | Xem trọng số từng từ hoặc cụm sau khi học |
| Rủi ro riêng | Từ điển khác miền | Học nhầm dấu hiệu không liên quan, như tên khách sạn |
| Tiếng Việt | Phần lớn chỉ tiếng Anh | Dùng được nếu có nhãn tiếng Việt và cách tách từ phù hợp. Chất lượng phụ thuộc dữ liệu |

Điểm khác biệt lớn nhất: các cách từ điển đã có điểm trước khi nhìn dữ liệu, còn SVM tạo điểm từ dữ liệu. Vì vậy SVM tận dụng được từ ngữ riêng của dữ liệu nghiên cứu, nhưng đổi lại cần nhãn đủ nhiều và đủ tốt, và dễ học nhầm.

**So với các cách học máy gần nhau.**

| Phương pháp | Điểm khác với SVM tuyến tính |
|---|---|
| Logistic Regression | Tối ưu một hàm mất mát khác, xuất ước lượng xác suất theo mô hình |
| Perceptron | Cập nhật trọng số khi gặp mẫu sai, không tối ưu biên như SVM |
| SVM có kernel | Ranh giới có thể cong, chi phí và tham số khác |

Đặt các cách cạnh nhau trên cùng câu: "The bed was not good."

| Cách | Điều gì xảy ra |
|---|---|
| SentiWordNet | Tra "good" ra điểm khen. Cần quy tắc "not" do người viết |
| VADER | Tự nhận ra "not", ra điểm chê |
| TextBlob | Có xử lý một số dấu hiệu, cần chạy mã để biết |
| AFINN | Tra "good" ra điểm khen, sai, trừ khi danh sách có sẵn cụm phủ định |
| Opinion Lexicon | Khớp "good", khen, sai, trừ khi người tự viết quy tắc |
| SentiStrength | Phủ định đổi chiều, ra chê |
| NRC | Đếm "good" vào nhóm khen, "not" không đổi gì |
| SVM chỉ dùng từng từ | Trọng số của "good" thường dương, dễ ra khen, sai |
| SVM có thêm cụm hai từ | Đúng nếu cụm "not good" hoặc cụm tương tự đã xuất hiện đủ trong dữ liệu học với nhãn chê |

<a id="muc-3"></a>

**3. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Quy định thế nào là khen, chê, trung tính, và xử lý đánh giá vừa khen vừa chê ra sao | Người |
| Gán nhãn cho tập đánh giá mẫu | Người |
| Chọn cách đổi văn bản thành số: từng từ, cụm hai từ, cách bỏ từ quá thông dụng | Người |
| Chọn các tham số như C, dạng hàm phạt, có cân bằng lớp không | Người đặt quy tắc, máy thử và đo |
| Chia dữ liệu thành phần học, phần chỉnh và phần chấm điểm cuối, theo khách sạn | Người |
| Đổi văn bản thành vector | Máy |
| Học trọng số và số điều chỉnh b | Máy |
| Tính điểm quyết định cho văn bản mới | Máy |
| Đổi điểm thành nhãn | Máy |
| So nhãn của máy với đáp án, tính chỉ số | Máy tính, người đọc các câu sai |
| Kết luận trong bài viết | Người |

Điểm của từng từ trong SVM do máy học, không phải từ điển có sẵn. Người ảnh hưởng đến điểm gián tiếp qua nhãn, qua cách chọn đặc trưng và qua cách chia dữ liệu. Vì vậy không có câu hỏi ai gán điểm cho từ, mà là nhãn do ai gán và gán thế nào.

<a id="muc-4"></a>

**4. Cách học và cách đọc điểm**

Máy làm ba việc.

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Học từ vựng và TF-IDF trên phần học, đổi mỗi đánh giá thành vector | Mỗi đánh giá là một dãy số, mỗi vị trí ứng với một từ hoặc cụm |
| 2 | Tìm trọng số w và số điều chỉnh b sao cho phân đúng nhãn và giữ biên | Mỗi từ hoặc cụm có một trọng số |
| 3 | Với đánh giá mới, tính điểm quyết định rồi chọn nhãn theo dấu | Nhãn và điểm quyết định |

Điểm quyết định viết thành công thức dạng văn bản:

```text
f = (w1 nhân x1) cộng (w2 nhân x2) cộng ... cộng b
```

| Ký hiệu | Ý nghĩa |
|---|---|
| x1, x2, ... | Các con số của đánh giá, ví dụ TF-IDF của từng từ |
| w1, w2, ... | Trọng số của từng từ, do máy học |
| b | Số điều chỉnh chung, do máy học |
| f | Điểm quyết định. Dương nghĩa là nghiêng về một lớp, âm là nghiêng về lớp kia |

Cách máy học trọng số, dạng giảng giải đơn giản:

```text
Giảm: một nửa tổng bình phương các trọng số, cộng C nhân tổng mức phạt của từng đánh giá
Mức phạt của một đánh giá = lớn nhất của (0 và 1 trừ y nhân f)
```

| Ký hiệu | Ý nghĩa |
|---|---|
| y | Nhãn thật của đánh giá, ví dụ +1 là khen, -1 là chê |
| y nhân f | Cho biết đúng hay sai phía và còn cách biên bao xa |
| C | Số do người chọn, quyết định phạt các đánh giá vi phạm mạnh đến đâu |

Đọc y nhân f:

| Giá trị y nhân f | Ý nghĩa | Mức phạt |
|---|---|---|
| Từ 1 trở lên | Đúng phía và nằm ngoài biên | 0 |
| Từ 0 đến dưới 1 | Đúng phía nhưng nằm trong vùng biên | Phạt nhẹ |
| Âm | Sai phía | Phạt nặng |

Về C: C lớn nghĩa là phạt vi phạm mạnh hơn, mô hình cố khớp dữ liệu học sát hơn và dễ bám vào mẫu nhiễu. C nhỏ nghĩa là cho phép vi phạm nhiều hơn, mô hình đơn giản hơn. Giá trị C phải chọn trên phần chỉnh, không chọn trên phần chấm điểm cuối.

Về hàm phạt: công thức trên dùng phạt dạng hinge để giảng giải. Trong scikit-learn, LinearSVC mặc định dùng phạt bình phương của số này, nên công thức giảng giải không trùng hoàn toàn với hàm mục tiêu trong mã [S3] [S4].

Về thứ tự lớp: trong bài toán hai lớp của scikit-learn, điểm quyết định dương thường hướng đến lớp thứ hai trong danh sách classes_. Phải kiểm tra thứ tự lớp của mô hình, không mặc định số dương là tích cực.

Về xác suất: điểm quyết định chưa phải xác suất. Nếu cần xác suất thì phải hiệu chuẩn bằng quy trình và dữ liệu riêng [S6].

<a id="muc-5"></a>

**5. Ví dụ năm tình huống**

Các trọng số trong bảng là **giả định** để tính cho dễ, không phải mô hình học thật. Để đơn giản, đặc trưng chỉ có giá trị 0 hoặc 1, nghĩa là từ có xuất hiện hay không. Điểm thật của mô hình học được xem ở mục 9.

| Đặc trưng | Trọng số giả định |
|---|---|
| clean | 1.2 |
| friendly | 0.9 |
| dirty | -1.1 |
| rude | -1.0 |
| not clean | -2.0 |
| aurora | 1.5 |
| b, số điều chỉnh chung | 0.1 |

**Tình huống 1: tính điểm cho một câu.** Câu: "The staff were friendly and the room was clean."

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Tìm đặc trưng có trong câu | friendly, clean |
| 2 | Lấy trọng số | 0.9 và 1.2 |
| 3 | Cộng thêm b | 0.9 cộng 1.2 cộng 0.1 |
| 4 | Điểm quyết định | f = 2.2 |
| 5 | Chọn nhãn theo dấu, giả sử dương là khen | Khen |

**Tình huống 2: mức phạt trong lúc học.** Ba đánh giá khác nhau trong phần học, với điểm f do mô hình tạm thời đưa ra.

| Đánh giá và nhãn thật | y | f | y nhân f | Mức phạt |
|---|---|---|---|---|
| "Spotless room, lovely staff." Nhãn khen | +1 | 1.4 | 1.4 | 0 vì ngoài biên |
| "The room was fine." Nhãn khen | +1 | 0.7 | 0.7 | 1 trừ 0.7 bằng 0.3, đúng phía nhưng nằm trong vùng biên |
| "The room was dirty." Nhãn chê | -1 | -0.7 | 0.7 | 1 trừ 0.7 bằng 0.3, đúng phía nhưng nằm trong vùng biên |
| "The staff were rude." Nhãn chê, nhưng mô hình đang cho điểm dương | -1 | 0.5 | -0.5 | 1 trừ -0.5 bằng 1.5, sai phía nên phạt nặng |

Máy điều chỉnh trọng số để giảm tổng mức phạt này, đồng thời giữ trọng số không quá lớn. Với hàm phạt bình phương của LinearSVC, mức phạt 0.3 thành 0.09.

**Tình huống 3: chữ "not".** Câu: "The bed was not clean."

| Cách làm | Tính | Điểm f | Kết quả |
|---|---|---|---|
| Chỉ dùng từng từ, đặc trưng có clean | 1.2 cộng 0.1 | 1.3 | Khen, sai |
| Thêm cụm hai từ, đặc trưng có clean và not clean | 1.2 trừ 2.0 cộng 0.1 | -0.7 | Chê, đúng |

Cụm "not clean" chỉ có trọng số âm nếu cụm này đã xuất hiện đủ trong phần học với nhãn chê. Trong lần chạy ở mục 9, cụm "not clean" thật sự có trọng số âm, nhưng đó là vì ví dụ học có sẵn một câu chứa cụm này. Cần kiểm tra bằng đánh giá có phủ định ở phần chỉnh, không giả định.

**Tình huống 4: học nhầm tên khách sạn.** Giả sử phần học chỉ có các đánh giá khen của khách sạn Aurora, nên đặc trưng "aurora" có trọng số dương.

| Câu | Đặc trưng khớp | Tính | Điểm f | Kết quả |
|---|---|---|---|---|
| "Aurora was dirty." | aurora, dirty | 1.5 trừ 1.1 cộng 0.1 | 0.5 | Khen, sai |
| "Bella was dirty." khách sạn khác | dirty | -1.1 cộng 0.1 | -1.0 | Chê, đúng |

Chia ngẫu nhiên các câu, chung khách sạn ở cả hai phần, sẽ thấy điểm cao vì tên Aurora giúp đoán đúng. Sang khách sạn mới thì dấu hiệu đó biến mất. Vì vậy phải chia theo khách sạn.

**Tình huống 5: câu không có từ cảm xúc.** Câu: "The room was zzholdout." Từ cuối là chữ giả chưa xuất hiện khi học.

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Bỏ từ chưa thấy khi học | zzholdout không vào vector |
| 2 | Còn lại các từ thông dụng như the, room, was | Có trong từ vựng |
| 3 | Tính điểm quyết định, lấy từ lần chạy thật ở mục 9 | f bằng khoảng 0.32 |
| 4 | Chọn nhãn | Khen |

Mô hình vẫn ra nhãn khen dù câu không chứa từ cảm xúc nào. Từ điển sẽ ghi thiếu dữ liệu ở trường hợp này, còn SVM luôn trả một nhãn. Vì vậy phải kiểm tra riêng tỷ lệ câu có ít nhất một từ có ích, và không coi mọi nhãn máy là bằng chứng.

<a id="muc-6"></a>

**6. Lỗi hay gặp**

1. Đọc điểm quyết định 0.8 như xác suất 80 phần trăm. Cần hiệu chuẩn riêng nếu mục tiêu đòi hỏi xác suất.
2. Cho rằng tăng C là tăng mức đơn giản của mô hình. Với cách tham số thông dụng, C lớn phạt vi phạm mạnh hơn, mô hình bám dữ liệu học sát hơn.
3. Học TF-IDF trên toàn bộ dữ liệu trước khi chia hoặc trước khi kiểm chứng chéo. Từ vựng và hệ số IDF cũng là thứ được học, nên làm vậy là rò rỉ. Phải đặt TF-IDF và SVM trong cùng một quy trình, và học lại trong từng lượt [S5].
4. Nhầm SVC mặc định với SVM tuyến tính. SVC mặc định dùng kernel khác. Kiểm tra tên lớp và kernel [S3] [S4].
5. Dùng điểm gần 0 làm nhãn trung tính. Điểm gần 0 chỉ là khó phân biệt giữa hai lớp. Muốn có lớp trung tính phải gán nhãn và huấn luyện ba lớp.
6. Hiểu trọng số dương là từ đó gây ra sự hài lòng. Trọng số là tham số dự đoán, không phải tác động nhân quả.
7. Chia ngẫu nhiên khi dữ liệu có nhiều đánh giá của cùng khách sạn, hoặc có nhiều bản trùng. Kết quả đẹp hơn thực tế.
8. Để tên khách sạn, tên nhân viên, mã đặt phòng nằm trong đặc trưng. Mô hình có thể dùng chúng thay cho nội dung.
9. Dùng nhãn của VADER hay nhãn của từ điển làm đáp án để huấn luyện rồi báo độ đúng so với chính nhãn đó. Đó là độ giống từ điển, không phải độ đúng. Đáp án phải do người gán.
10. Không kiểm tra thứ tự lớp. Số dương có thể hướng đến lớp chê nếu nhãn được mã hóa theo thứ tự khác.
11. Bỏ qua cảnh báo chưa hội tụ. Cần kiểm tra thang đo, số vòng lặp tối đa, thay vì tắt cảnh báo.
12. Dùng kernel tốn kém cho dữ liệu lớn khi chưa thử SVM tuyến tính làm mốc.

<a id="muc-7"></a>

**7. Làm sao biết kết quả đáng tin**

Kiểm tra theo ba bước, giống các file trước, có thêm phần chỉnh vì SVM cần chọn tham số.

Bước 1. Người tạo đáp án.

1. Chọn khoảng 100 câu trở lên từ nhiều khách sạn khác nhau. Với học máy, càng nhiều càng tốt, nhưng cần đủ để chia ba phần.
2. Hai người gán nhãn khen, chê hoặc trung tính, mỗi người làm riêng.
3. Máy tính mức hai người đồng ý với nhau, gọi là hệ số kappa. Càng gần 1 thì càng đồng ý.
4. Chỗ hai người khác nhau thì bàn lại và chọn một nhãn. Quy định trước cách xử lý đánh giá vừa khen vừa chê.

Bước 2. Chia đáp án làm ba phần, theo khách sạn.

| Phần | Dùng để làm gì |
|---|---|
| Phần học | Máy học trọng số. Học TF-IDF cũng chỉ dùng phần này |
| Phần chỉnh | Chọn C, chọn đặc trưng, cân bằng lớp, thử cụm hai từ. Được sửa thoải mái |
| Phần chấm điểm cuối | Chỉ dùng một lần ở cuối, không sửa gì sau khi thấy kết quả |

Câu của cùng một khách sạn chỉ nằm ở một phần. Nên loại các bản trùng trước khi chia. Có thể dùng kiểm chứng chéo trên phần học để chọn tham số, và kiểm chứng chéo lồng nhau nếu vừa chọn tham số vừa cần ước lượng hiệu năng mà chưa có phần chấm điểm cuối riêng. Quy trình trong mỗi lượt phải gồm cả bước tạo vector và bước học [S5].

Bước 3. Máy so nhãn của SVM với đáp án trên phần chấm điểm cuối, người báo các con số sau.

| Con số | Ý nghĩa đơn giản |
|---|---|
| Độ đúng trung bình của các nhãn (macro-F1) | Máy đúng bao nhiêu, tính riêng cho từng nhãn rồi lấy trung bình |
| Bảng nhầm lẫn | Máy hay nhầm nhãn nào thành nhãn nào |
| So với mốc nhãn phổ biến nhất | Mô hình có hơn cách luôn đoán nhãn nhiều nhất không |
| Tỷ lệ câu có ít nhất một từ có ích đã thấy khi học | Mô hình dựa vào nội dung được bao nhiêu phần |
| Kết quả theo nhóm độ dài và theo khách sạn | Câu ngắn, câu dài, khách sạn khác có đúng như nhau không |

Với SVM có thêm bốn kiểm tra đáng làm.

1. Đọc trọng số lớn nhất và nhỏ nhất xem mô hình dựa vào từ có nghĩa hay dấu hiệu ngẫu nhiên, như tên khách sạn.
2. Chạy lại với các giá trị C và đặc trưng khác để xem kết luận có vững không. Không lặp mãi sau khi đã có bằng chứng đủ.
3. Chạy trên khách sạn hoặc thời gian mới nếu mục tiêu là áp dụng sang dữ liệu khác.
4. Nếu dùng xác suất, hiệu chuẩn bằng dữ liệu riêng và đo thêm mức sai của xác suất [S6].

Để so SVM với các cách từ điển cho công bằng, chạy tất cả trên cùng phần chấm điểm cuối rồi so macro-F1. SVM có được học từ nhãn nên cần dành phần học riêng. Từ điển thì không cần. Hai bên vẫn so trên cùng phần chấm điểm cuối.

Khi viết kết luận, nhãn của máy không đủ để nói khách hài lòng nhiều hay ít. Nói "70 phần trăm mẫu được gán khen" chưa có nghĩa 70 phần trăm khách hài lòng. Cần độ đúng của mô hình, cách lấy mẫu và sai số theo nhóm.

<a id="muc-8"></a>

**8. Chọn cái nào**

| Tình huống | Cách hợp hơn | Lý do | Điểm yếu |
|---|---|---|---|
| Chưa có nhãn để huấn luyện | Các cách từ điển | Chạy ngay, không cần nhãn | Từ điển khác miền, phải kiểm tra |
| Có nhãn, cần baseline học máy cho văn bản nhiều chiều | SVM tuyến tính | Dùng tốt với ma trận thưa như TF-IDF, xem được trọng số | Không có xác suất trực tiếp, không hiểu ngữ cảnh dài |
| Cần ước lượng xác suất | Logistic Regression, hoặc SVM kèm hiệu chuẩn | Xuất xác suất theo mô hình | Cần kiểm tra mức chuẩn của xác suất |
| Muốn cách rất đơn giản | Perceptron | Cập nhật theo mẫu sai | Không tối ưu biên |
| Ranh giới có thể cong | SVM có kernel | Phi tuyến trong đặc trưng gốc | Tốn tài nguyên hơn, nên thử tuyến tính trước |
| Ngữ cảnh dài, phủ định phức tạp | Mô hình ngữ cảnh | Hiểu câu tốt hơn từng từ | Chưa nằm trong bộ tài liệu này |
| Muốn kết hợp tri thức từ điển | SVM kèm điểm của từ điển làm đặc trưng thêm | Vừa có tri thức có sẵn vừa học từ dữ liệu | Đây là đề xuất của người biên soạn, chưa được kiểm tra |
| Đúng hơn trên đánh giá khách sạn | Chưa biết | Chưa có bằng chứng | Phải đo macro-F1 trên cùng phần chấm điểm cuối |

Không có thuật toán luôn đứng đầu trên mọi tập dữ liệu. Kết quả gắn với cách chọn tham số, cách chia dữ liệu và thiết kế đánh giá. Nếu Logistic Regression đạt gần bằng SVM và dễ đáp ứng yêu cầu xác suất hơn, đó là một lý do chọn, không nên chọn chỉ theo điểm trung bình cao hơn một chút.

<a id="muc-9"></a>

**9. Mã chạy thử**

Cần Python và scikit-learn. Mã đã được chạy thử với scikit-learn 1.8.0. Dữ liệu cực nhỏ, chỉ minh họa quy trình, không dùng để tuyên bố hiệu năng. Nhãn 1 là khen, nhãn 0 là chê, không có lớp trung tính.

```python
import numpy as np
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.svm import LinearSVC
from sklearn.metrics import f1_score, confusion_matrix

train = [
    "The room was clean and the staff were friendly.",
    "Lovely hotel, comfortable bed and delicious breakfast.",
    "Great location and very friendly service.",
    "The room was spotless and the bed was comfortable.",
    "The room was dirty and the service was terrible.",
    "Rude staff and a noisy, dirty bathroom.",
    "Terrible breakfast and slow service.",
    "The bed was not clean and the staff were rude.",
]
y_train = [1, 1, 1, 1, 0, 0, 0, 0]   # 1 là khen, 0 là chê

test = [
    "Friendly staff and a clean room.",
    "Delicious breakfast and comfortable bed.",
    "Dirty room and terrible service.",
    "The bathroom was noisy and the staff were rude.",
]
y_test = [1, 1, 0, 0]
khong_co_tu_cam_xuc = "The room was zzholdout."

model = Pipeline([
    ("tfidf", TfidfVectorizer(ngram_range=(1, 2), lowercase=True)),
    ("clf", LinearSVC(C=1.0, loss="squared_hinge", dual=True,
                      max_iter=10000, random_state=42)),
])
model.fit(train, y_train)

vec = model.named_steps["tfidf"]
clf = model.named_steps["clf"]
assert "zzholdout" not in vec.vocabulary_
print("Thứ tự lớp:", clf.classes_.tolist())

pred = model.predict(test)
scores = model.decision_function(test)
for text, s, p in zip(test, scores, pred):
    print(round(float(s), 3), int(p), text)
print("Macro-F1 minh họa:", f1_score(y_test, pred, average="macro"))
print(confusion_matrix(y_test, pred, labels=[0, 1]))

s2 = model.decision_function([khong_co_tu_cam_xuc])[0]
print("Câu không có từ cảm xúc:", round(float(s2), 3),
      int(model.predict([khong_co_tu_cam_xuc])[0]))

names = np.array(vec.get_feature_names_out())
w = clf.coef_[0]
order = np.argsort(w)
print("Trọng số âm nhất:", [(names[i], round(float(w[i]), 2)) for i in order[:3]])
print("Trọng số dương nhất:", [(names[i], round(float(w[i]), 2)) for i in order[::-1][:3]])
```

Kết quả khi chạy thử, rút gọn.

| Dòng | Giá trị |
|---|---|
| Thứ tự lớp | 0 rồi 1, nên điểm dương hướng đến nhãn 1, tức khen |
| "Friendly staff and a clean room." | f khoảng 0.15, nhãn 1 |
| "Delicious breakfast and comfortable bed." | f khoảng 0.30, nhãn 1 |
| "Dirty room and terrible service." | f khoảng -0.43, nhãn 0 |
| "The bathroom was noisy and the staff were rude." | f khoảng -0.37, nhãn 0 |
| Câu không có từ cảm xúc | f khoảng 0.32, nhãn 1 |
| Trọng số âm nhất | terrible, rude, dirty |
| Trọng số dương nhất | friendly, comfortable, were friendly |

Dòng trọng số cho thấy mô hình học được các từ khen chê hợp lý trên dữ liệu nhỏ này. Macro-F1 bằng 1 ở đây chỉ có nghĩa dữ liệu thử rất dễ, không có nghĩa mô hình tốt. Dòng câu không có từ cảm xúc cho thấy nhãn luôn được trả ra, như tình huống 5.

Từ zzholdout là chữ giả để kiểm tra rằng dữ liệu thử không đi vào từ vựng. Pipeline không tự ngăn mọi loại rò rỉ. Vẫn phải chia đúng theo khách sạn, loại bản trùng và chọn tham số ngoài phần chấm điểm cuối. Với tiếng Việt thật, cần kiểm tra cách tách từ, cách xử lý phủ định và chất lượng nhãn.

<a id="muc-10"></a>

**10. Tài liệu tham khảo**

- **[S1]** Cortes, C., & Vapnik, V. (1995). *Support-vector networks*. Machine Learning, 20, 273 đến 297. [Trang công trình](https://research.google/pubs/support-vector-networks/). [DOI](https://doi.org/10.1007/BF00994018).
- **[S2]** Joachims, T. (1998). *Text Categorization with Support Vector Machines: Learning with Many Relevant Features*. [Bản của tác giả](https://www.cs.cornell.edu/people/tj/publications/joachims_98a.pdf).
- **[S3]** scikit-learn. [Support Vector Machines](https://scikit-learn.org/stable/modules/svm.html).
- **[S4]** scikit-learn. [LinearSVC](https://scikit-learn.org/stable/modules/generated/sklearn.svm.LinearSVC.html).
- **[S5]** scikit-learn. [Common pitfalls: data leakage](https://scikit-learn.org/stable/common_pitfalls.html).
- **[S6]** scikit-learn. [Probability calibration](https://scikit-learn.org/stable/modules/calibration.html).

Các nguồn trên giới thiệu phương pháp và phần mềm. Nội dung khái niệm và quy trình lấy từ tài liệu nguồn dài đã gửi, đối chiếu với các nguồn trên. Phần so sánh với các cách Lexicon Based dựa trên các file trước. Ví dụ khách sạn, trọng số giả định, và kế hoạch kiểm tra là đề xuất của người biên soạn, chưa phải kết quả của một thí nghiệm thực tế. Mã đã chạy được với scikit-learn 1.8.0, tài liệu trực tuyến có thể thuộc phiên bản mới hơn.
