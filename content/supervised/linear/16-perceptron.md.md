# Perceptron: giải thích đơn giản

Nhánh Machine Learning, phần học có giám sát tuyến tính. Bài này giữ cấu trúc 10 mục như file SVM và Logistic Regression, mục 4 là cách học và cách đọc điểm. Điểm riêng của Perceptron: nó học bằng cách **sửa trọng số mỗi khi gặp một mẫu sai**, không có xác suất, không giữ biên, và kết quả có thể thay đổi theo thứ tự mẫu.

**Mục lục**

- [1. Perceptron là gì](#muc-1)
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

**1. Perceptron là gì**

Perceptron là một bộ phân loại tuyến tính học từ các ví dụ đã có nhãn rồi đoán nhãn cho văn bản mới. Tên này gắn với công trình của Rosenblatt năm 1958 [S1]. Bài này chỉ nói về perceptron tuyến tính cơ bản, không có tầng ẩn. **Multilayer Perceptron (MLP)** là mạng nhiều tầng, thuộc một họ mô hình khác, không phải perceptron chạy thêm vòng lặp.

Có ba bước trong đầu của chương trình.

1. Đổi mỗi đánh giá thành một dãy số. Cách phổ biến là TF-IDF, giống bài SVM và Logistic Regression.
2. Học một bộ trọng số, mỗi từ một trọng số, cộng thêm một số điều chỉnh chung gọi là b. Từ nghiêng về khen có trọng số dương, từ nghiêng về chê có trọng số âm.
3. Với đánh giá mới, nhân trọng số với dãy số rồi cộng lại. Kết quả gọi là điểm quyết định. Điểm nằm bên nào của số 0 thì chọn nhãn bên đó.

Cách tìm trọng số là phần đặc trưng. Chương trình bắt đầu với trọng số bằng 0, rồi duyệt từng đánh giá trong phần học. Nếu đánh giá đó bị chấm sai hoặc đúng điểm 0, trọng số được kéo theo hướng giúp đánh giá đó khá hơn. Nếu đã chấm đúng phía thì không đổi gì. Mỗi lượt duyệt hết phần học gọi là một **epoch**, và chương trình lặp nhiều epoch [S2].

Khác với các cách Lexicon Based, Perceptron không có từ điển điểm dựng sẵn. Trọng số do máy học từ nhãn của người, nên chất lượng nhãn quyết định chất lượng mô hình.

Có ba điều dễ nhầm.

1. Perceptron không phải MLP. Thêm epoch không biến nó thành mạng nhiều tầng và không làm nó giải được những bài toán một đường thẳng không tách nổi.
2. Điểm quyết định không phải xác suất. Điểm 0.9 không có nghĩa khách tích cực 90 phần trăm. API Perceptron của scikit-learn không có `predict_proba`.
3. Chạy đủ epoch không bảo đảm hết lỗi. Chỉ khi dữ liệu tách được bằng một đường thẳng và thỏa các điều kiện của định lý cổ điển thì mới có bảo đảm đó. Đánh giá khách sạn thật thường có nhiễu và ý kiến hỗn hợp.

<a id="muc-2"></a>

**2. So sánh với các cách trước**

Thông tin về các cách Lexicon Based lấy từ các file trước. Thông tin về SVM và Logistic Regression lấy từ hai file kế trước. Thông tin về các cách học máy khác lấy từ tài liệu nguồn đã đối chiếu.

**So với các cách dùng từ điển.**

| Điểm so sánh | Các cách từ điển như VADER, AFINN, SentiStrength | Perceptron |
|---|---|---|
| Điểm của từ đến từ đâu | Có sẵn trong từ điển, do tác giả hoặc người đánh giá gán | Máy học từ nhãn của người trên dữ liệu của nghiên cứu |
| Cần nhãn để chạy không | Không, nhưng cần nhãn để kiểm tra | Có, cần nhãn để huấn luyện và thêm nhãn để kiểm tra |
| Từ ngoài từ điển | Bị bỏ qua, không có điểm | Từ chưa thấy khi học bị bỏ qua trong vector |
| Học từ ngữ riêng của khách sạn | Không, trừ khi người tự thêm | Có, nếu từ đó xuất hiện đủ trong dữ liệu học |
| Xử lý "not" | Tùy cách, VADER và SentiStrength có sẵn | Không có sẵn. Học được nếu dùng cụm hai từ và cụm đó có trong dữ liệu học |
| Có luôn ra nhãn không | Có thể ra thiếu dữ liệu khi không khớp từ nào | Luôn ra một nhãn, kể cả khi câu không có từ cảm xúc |
| Kết quả chạy lại | Cùng đầu vào, cùng kết quả | Có thể đổi theo thứ tự mẫu khi học (xem mục 5 và mục 9) |
| Rủi ro riêng | Từ điển khác miền | Học nhầm dấu hiệu không liên quan, như tên khách sạn. Nhạy với thứ tự và nhiễu |
| Tiếng Việt | Phần lớn chỉ tiếng Anh | Dùng được nếu có nhãn tiếng Việt và cách tách từ phù hợp |

**So với các cách học máy gần nhau.**

| Phương pháp | Điểm khác với Perceptron |
|---|---|
| SVM tuyến tính | Tìm đường phân chia có biên rộng, đầu ra là điểm quyết định. Perceptron chỉ cần phân đúng phía, không quan tâm biên |
| Logistic Regression | Có mô hình xác suất, tối ưu log loss. Đúng rồi vẫn tiếp tục được "đẩy" nhẹ. Perceptron đúng phía là dừng cập nhật |
| SGDClassifier | Khung huấn luyện dùng được cho nhiều loss. Perceptron của scikit-learn là một trường hợp dùng loss perceptron [S2] [S4] |
| Averaged Perceptron | Lấy trung bình trọng số qua các lần cập nhật, thường ổn định hơn nhưng không bảo đảm luôn thắng |
| MLP | Mạng nhiều tầng, có thể phi tuyến nhờ tầng ẩn |

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
| SVM hoặc LR chỉ dùng từng từ | Trọng số của "good" thường dương, dễ ra khen, sai |
| SVM hoặc LR có thêm cụm hai từ | Đúng nếu cụm "not good" hoặc cụm tương tự đã xuất hiện đủ trong dữ liệu học với nhãn chê |
| Perceptron chỉ dùng từng từ | Giống trên: "good" thường dương, dễ ra khen, sai |
| Perceptron có thêm cụm hai từ | Đúng nếu cụm phủ định đã xuất hiện trong dữ liệu học với nhãn chê và được cập nhật đủ. Vì cập nhật chỉ xảy ra khi sai, cụm này phải "gây ra lỗi" lúc học thì mới được sửa |

Điểm khác biệt lớn nhất so với từ điển: từ điển đã có điểm trước khi nhìn dữ liệu, còn Perceptron tạo điểm từ lỗi của chính nó trên dữ liệu. Điểm khác biệt lớn nhất so với SVM và LR: Perceptron đơn giản nhất, nhưng kết quả nhạy với thứ tự mẫu, không có biên, không có xác suất.

<a id="muc-3"></a>

**3. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Quy định thế nào là khen, chê, trung tính, và xử lý đánh giá vừa khen vừa chê ra sao | Người |
| Gán nhãn cho tập đánh giá mẫu | Người |
| Chọn cách đổi văn bản thành số: từng từ, cụm hai từ, cách bỏ từ quá thông dụng | Người |
| Chọn số epoch, có xáo trộn thứ tự mẫu không, hạt giống ngẫu nhiên, có phạt trọng số không, quy tắc dừng | Người đặt quy tắc, máy thử và đo |
| Chia dữ liệu thành phần học, phần chỉnh và phần chấm điểm cuối, theo khách sạn | Người |
| Đổi văn bản thành vector | Máy |
| Duyệt mẫu, phát hiện mẫu sai, cập nhật trọng số và b | Máy |
| Tính điểm quyết định cho văn bản mới | Máy |
| Đổi điểm thành nhãn | Máy |
| So nhãn của máy với đáp án, tính chỉ số | Máy tính, người đọc các câu sai |
| Kết luận trong bài viết | Người |

Điểm của từng từ do máy học, không phải từ điển có sẵn. Người ảnh hưởng đến điểm gián tiếp qua nhãn, qua cách chọn đặc trưng, qua cách chia dữ liệu và qua các lựa chọn như thứ tự mẫu, hạt giống ngẫu nhiên. Vì vậy phải ghi lại các lựa chọn này khi báo kết quả.

<a id="muc-4"></a>

**4. Cách học và cách đọc điểm**

Máy làm ba việc.

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Học từ vựng và TF-IDF trên phần học, đổi mỗi đánh giá thành vector | Mỗi đánh giá là một dãy số |
| 2 | Bắt đầu với w bằng 0 và b bằng 0. Duyệt từng mẫu, sai thì cập nhật. Lặp nhiều epoch | Mỗi từ hoặc cụm có một trọng số |
| 3 | Với đánh giá mới, tính điểm quyết định rồi chọn nhãn theo dấu | Nhãn và điểm quyết định |

Điểm quyết định, giống SVM:

```text
f = (w1 nhân x1) cộng (w2 nhân x2) cộng ... cộng b
```

| Ký hiệu | Ý nghĩa |
|---|---|
| x1, x2, ... | Các con số của đánh giá, ví dụ TF-IDF của từng từ |
| w1, w2, ... | Trọng số của từng từ, do máy học |
| b | Số điều chỉnh chung, do máy học |
| f | Điểm quyết định. Dương nghĩa là nghiêng về một lớp, âm là nghiêng về lớp kia |

Luật cập nhật, khi viết nhãn thật là y bằng +1 (khen) hoặc -1 (chê):

```text
Nếu y nhân f nhỏ hơn hoặc bằng 0:
    w mới = w cũ cộng (tốc độ học nhân y nhân x)
    b mới = b cũ cộng (tốc độ học nhân y)
Ngược lại: giữ nguyên
```

| Ký hiệu | Ý nghĩa |
|---|---|
| y | Nhãn thật, +1 là khen, -1 là chê |
| y nhân f | Dương nghĩa là đúng phía. Nhỏ hơn hoặc bằng 0 nghĩa là sai phía hoặc đúng trên ranh giới |
| Tốc độ học | Số do người chọn, quyết định mỗi lần sửa mạnh đến đâu |

Đọc y nhân f:

| Giá trị y nhân f | Ý nghĩa | Máy làm |
|---|---|---|
| Lớn hơn 0 | Đúng phía | Không đổi gì, kể cả khi chỉ đúng sát ranh giới |
| Bằng 0 | Đúng trên ranh giới | Cập nhật |
| Âm | Sai phía | Cập nhật |

So với SVM: SVM còn phạt nhẹ khi mẫu đúng phía nhưng nằm trong vùng biên. Perceptron không có khái niệm biên, nên đúng sát ranh giới cũng bị coi là xong.

Về tốc độ học: với trọng số khởi tạo bằng 0 và không phạt trọng số, đổi tốc độ học chỉ nhân cả bộ trọng số lên một hệ số, không đổi nhãn dự đoán.

Về thứ tự: vì mỗi lần cập nhật chỉ nhìn một mẫu, đổi thứ tự mẫu có thể cho ra bộ trọng số khác. Cần ghi hạt giống ngẫu nhiên (random seed) và thử nhiều hạt giống.

Về mã nhãn: tính tay dùng +1 và -1. API scikit-learn nhận nhãn 0 và 1 và tự xử lý. Không đưa 0 trực tiếp vào công thức tay vì cập nhật của lớp đó sẽ bằng 0.

Về thứ tự lớp: trong bài toán hai lớp của scikit-learn, điểm dương hướng đến lớp thứ hai trong `classes_`. Điểm đúng bằng 0 được xếp vào lớp đầu trong lần chạy ở mục 9. Phải kiểm tra thứ tự lớp, không mặc định số dương là tích cực [S2].

Về xác suất: điểm quyết định không phải xác suất. Nếu cần xác suất, dùng mô hình khác như Logistic Regression, hoặc hiệu chuẩn riêng bằng dữ liệu tách biệt. Không áp sigmoid tùy ý rồi gọi là xác suất đã kiểm định.

<a id="muc-5"></a>

**5. Ví dụ năm tình huống**

Các trọng số trong tình huống 1, 3, 4 là **giả định** để tính cho dễ, giống bài SVM và Logistic Regression, không phải mô hình học thật. Đặc trưng chỉ có giá trị 0 hoặc 1, nghĩa là từ có xuất hiện hay không. Kết quả thật xem ở mục 9.

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

Đến đây Perceptron giống hệt SVM. Khác nhau nằm ở cách học trọng số, xem tình huống 2.

**Tình huống 2: một lượt học, từng bước.** Học với bốn đặc trưng friendly, clean, dirty, rude cộng b. Bắt đầu mọi trọng số bằng 0, tốc độ học 1, nhãn khen là +1 và chê là -1. Đây là phép tính tay minh họa, không phải kết quả thật của scikit-learn.

| Mẫu thứ tự | Nhãn y | f trước khi cập nhật | y nhân f | Máy làm | Trọng số sau |
|---|---|---|---|---|---|
| 1. "friendly, clean" | +1 | 0 | 0 | Cập nhật | friendly 1, clean 1, b 1 |
| 2. "dirty, rude" | -1 | 0 cộng b (1) bằng 1 | -1 | Cập nhật | dirty -1, rude -1, b 0 |
| 3. "friendly, clean" | +1 | 1 cộng 1 cộng 0 bằng 2 | 2 | Không đổi | Giữ nguyên |
| 4. "rude" | -1 | -1 cộng 0 bằng -1 | 1 | Không đổi | Giữ nguyên |
| 5. "friendly, dirty", nhãn khen | +1 | 1 trừ 1 cộng 0 bằng 0 | 0 | Cập nhật | friendly 2, dirty 0, b 1 |

Nhìn mẫu 5 và mẫu 2: sau khi sửa cho mẫu 5, trọng số của "dirty" về 0 và b lên 1. Giờ mẫu 2 "dirty, rude" có f bằng 0 cộng (-1) cộng 1, tức 0, nên lại thành mẫu cần sửa ở epoch sau. Một lần sửa cho mẫu này làm mẫu trước khó hơn. Đây là lý do Perceptron cần lặp nhiều epoch và vì sao nếu dữ liệu mâu thuẫn thì nó có thể dao động mãi, không phải do chưa chạy đủ lâu.

**Tình huống 3: chữ "not".** Câu: "The bed was not clean."

| Cách làm | Tính | Điểm f | Kết quả |
|---|---|---|---|
| Chỉ dùng từng từ, đặc trưng có clean | 1.2 cộng 0.1 | 1.3 | Khen, sai |
| Thêm cụm hai từ, đặc trưng có clean và not clean | 1.2 trừ 2.0 cộng 0.1 | -0.7 | Chê, đúng |

Cụm "not clean" chỉ có trọng số âm nếu cụm này có trong phần học với nhãn chê và từng bị cập nhật. Vì Perceptron chỉ học khi sai, nếu câu phủ định đã được chấm đúng nhờ các từ khác thì cụm này có thể không được kéo xuống. Cần kiểm tra bằng đánh giá có phủ định ở phần chỉnh, không giả định.

**Tình huống 4: học nhầm tên khách sạn.** Giả sử phần học chỉ có các đánh giá khen của khách sạn Aurora, nên đặc trưng "aurora" có trọng số dương.

| Câu | Đặc trưng khớp | Tính | Điểm f | Kết quả |
|---|---|---|---|---|
| "Aurora was dirty." | aurora, dirty | 1.5 trừ 1.1 cộng 0.1 | 0.5 | Khen, sai |
| "Bella was dirty." khách sạn khác | dirty | -1.1 cộng 0.1 | -1.0 | Chê, đúng |

Chia ngẫu nhiên các câu, chung khách sạn ở cả hai phần, sẽ thấy điểm cao vì tên Aurora giúp đoán đúng. Sang khách sạn mới thì dấu hiệu đó biến mất. Vì vậy phải chia theo khách sạn.

**Tình huống 5: câu không có từ cảm xúc, và chạy lại với hạt giống khác.** Câu: "The room was zzholdout." Từ cuối là chữ giả chưa xuất hiện khi học. Số liệu lấy từ lần chạy thật ở mục 9.

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Bỏ từ chưa thấy khi học | zzholdout không vào vector |
| 2 | Còn lại các từ thông dụng như the, room, was | Có trong từ vựng |
| 3 | Tính điểm quyết định, hạt giống 42 | f khoảng 0.34, nhãn khen |
| 4 | Chạy lại với hạt giống 1 | f bằng 0.0, nhãn chê |

Cùng dữ liệu, cùng cấu hình, chỉ đổi thứ tự mẫu mà nhãn của câu này đổi từ khen sang chê. Mô hình vẫn luôn trả một nhãn dù câu không có từ cảm xúc, còn từ điển sẽ ghi thiếu dữ liệu. Vì vậy phải kiểm tra riêng tỷ lệ câu có ít nhất một từ có ích, thử nhiều hạt giống, và không coi mọi nhãn máy là bằng chứng.

<a id="muc-6"></a>

**6. Lỗi hay gặp**

1. Nhầm Perceptron với MLP. Cần ghi rõ có hay không tầng ẩn.
2. Tin mọi dữ liệu đều hội tụ. Chỉ khi dữ liệu tách được bằng một đường thẳng mới có bảo đảm cổ điển. Dữ liệu thật có nhiễu và ý kiến hỗn hợp thường không thỏa.
3. Cho rằng tăng epoch sẽ giải quyết mọi thứ. Nếu hai mẫu có cùng vector nhưng nhãn trái nhau, hoặc quan hệ kiểu XOR, thì không có đường thẳng nào phân đúng hết, thêm epoch không đổi điều đó.
4. Đọc điểm quyết định như xác suất. Điểm 0.9 không phải 90 phần trăm. Cần hiệu chuẩn riêng nếu mục tiêu đòi hỏi xác suất.
5. Chỉ chạy một lần với một hạt giống rồi kết luận. Đổi thứ tự mẫu có thể đổi kết quả, xem mục 9.
6. Học TF-IDF trên toàn bộ dữ liệu trước khi chia hoặc trước khi kiểm chứng chéo. Từ vựng và hệ số IDF cũng là thứ được học, nên làm vậy là rò rỉ. Phải đặt TF-IDF và Perceptron trong cùng một quy trình, và học lại trong từng lượt [S5].
7. Dừng hoặc chọn số epoch bằng cách nhìn phần chấm điểm cuối. Số epoch cũng là một lựa chọn, phải chọn trên phần chỉnh.
8. Dùng điểm gần 0 làm nhãn trung tính. Điểm gần 0 chỉ là khó phân biệt giữa hai lớp. Muốn có lớp trung tính phải gán nhãn và huấn luyện ba lớp.
9. Hiểu trọng số dương là từ đó gây ra sự hài lòng. Trọng số là tham số dự đoán, không phải tác động nhân quả, và còn đổi theo thứ tự mẫu.
10. Chia ngẫu nhiên khi dữ liệu có nhiều đánh giá của cùng khách sạn, hoặc có nhiều bản trùng. Kết quả đẹp hơn thực tế.
11. Để tên khách sạn, tên nhân viên, mã đặt phòng nằm trong đặc trưng. Mô hình có thể dùng chúng thay cho nội dung.
12. Dùng nhãn của VADER hay nhãn của từ điển làm đáp án để huấn luyện rồi báo độ đúng so với chính nhãn đó. Đó là độ giống từ điển, không phải độ đúng.
13. Không kiểm tra thứ tự lớp. Số dương có thể hướng đến lớp chê nếu nhãn được mã hóa theo thứ tự khác.
14. Đổi từ vựng giữa các lần `partial_fit` khi học dần. Thay cột đặc trưng làm trọng số cũ mất nghĩa. Phải cố định không gian đặc trưng, ví dụ bằng hashing, và ghi rõ cấu hình.
15. Học dần bằng chính nhãn máy tự dự đoán như thể là nhãn chuẩn. Sai số có thể tự khuếch đại.

<a id="muc-7"></a>

**7. Làm sao biết kết quả đáng tin**

Kiểm tra theo ba bước, giống file SVM và Logistic Regression, có thêm kiểm tra độ nhạy với thứ tự mẫu.

Bước 1. Người tạo đáp án.

1. Chọn khoảng 100 câu trở lên từ nhiều khách sạn khác nhau. Với học máy, càng nhiều càng tốt, nhưng cần đủ để chia ba phần.
2. Hai người gán nhãn khen, chê hoặc trung tính, mỗi người làm riêng.
3. Máy tính mức hai người đồng ý với nhau, gọi là hệ số kappa. Càng gần 1 thì càng đồng ý.
4. Chỗ hai người khác nhau thì bàn lại và chọn một nhãn. Quy định trước cách xử lý đánh giá vừa khen vừa chê.

Bước 2. Chia đáp án làm ba phần, theo khách sạn.

| Phần | Dùng để làm gì |
|---|---|
| Phần học | Máy học trọng số. Học TF-IDF cũng chỉ dùng phần này |
| Phần chỉnh | Chọn số epoch, tốc độ học, phạt trọng số, cân bằng lớp, thử cụm hai từ, thử nhiều hạt giống |
| Phần chấm điểm cuối | Chỉ dùng một lần ở cuối, không sửa gì sau khi thấy kết quả |

Câu của cùng một khách sạn chỉ nằm ở một phần. Nên loại các bản trùng trước khi chia. Có thể dùng kiểm chứng chéo trên phần học để chọn tham số. Quy trình trong mỗi lượt phải gồm cả bước tạo vector và bước học [S5].

Bước 3. Máy so nhãn của Perceptron với đáp án trên phần chấm điểm cuối, người báo các con số sau.

| Con số | Ý nghĩa đơn giản |
|---|---|
| Độ đúng trung bình của các nhãn (macro-F1) | Máy đúng bao nhiêu, tính riêng cho từng nhãn rồi lấy trung bình |
| Bảng nhầm lẫn | Máy hay nhầm nhãn nào thành nhãn nào |
| So với mốc nhãn phổ biến nhất | Mô hình có hơn cách luôn đoán nhãn nhiều nhất không |
| Tỷ lệ câu có ít nhất một từ có ích đã thấy khi học | Mô hình dựa vào nội dung được bao nhiêu phần |
| Kết quả theo nhóm độ dài và theo khách sạn | Câu ngắn, câu dài, khách sạn khác có đúng như nhau không |
| **Kết quả qua nhiều hạt giống** | **Đổi thứ tự mẫu thì macro-F1 dao động bao nhiêu** |
| Số epoch thực chạy và cảnh báo | Mô hình có ổn định không |

Với Perceptron có thêm bốn kiểm tra đáng làm.

1. Đọc trọng số lớn nhất và nhỏ nhất xem mô hình dựa vào từ có nghĩa hay dấu hiệu ngẫu nhiên, như tên khách sạn.
2. Chạy lại với nhiều hạt giống và báo cả khoảng dao động, không chỉ báo lần đẹp nhất.
3. Thử Averaged Perceptron hoặc có phạt trọng số trên phần chỉnh, ghi rõ đó là thay đổi phương pháp.
4. Nếu học dần theo thời gian, đánh giá theo kiểu dự đoán trước, chấm khi nhãn có, rồi mới cập nhật. Không trộn dữ liệu tương lai vào việc học đặc trưng.

Để so Perceptron với SVM, LR và các cách từ điển cho công bằng, chạy tất cả trên cùng phần chấm điểm cuối rồi so macro-F1. Các cách học máy cần phần học riêng, từ điển thì không. Hai bên vẫn so trên cùng phần chấm điểm cuối.

Khi viết kết luận, nhãn của máy không đủ để nói khách hài lòng nhiều hay ít. Nói "70 phần trăm mẫu được gán khen" chưa có nghĩa 70 phần trăm khách hài lòng. Cần độ đúng của mô hình, cách lấy mẫu và sai số theo nhóm. Câu "Perceptron đã cập nhật n lần" chỉ nói về quá trình học, không có nghĩa n ví dụ đã được hiểu.

<a id="muc-8"></a>

**8. Chọn cái nào**

| Tình huống | Cách hợp hơn | Lý do | Điểm yếu |
|---|---|---|---|
| Chưa có nhãn để huấn luyện | Các cách từ điển | Chạy ngay, không cần nhãn | Từ điển khác miền, phải kiểm tra |
| Cần một baseline học máy rất đơn giản, dễ theo dõi từng bước cập nhật | Perceptron | Luật cập nhật ngắn, chạy nhanh trên ma trận thưa | Không có biên, không có xác suất, nhạy thứ tự |
| Cần baseline ổn định hơn cho văn bản nhiều chiều | SVM tuyến tính | Giữ biên, thường ổn định hơn | Không có xác suất trực tiếp |
| Cần xác suất để xếp ưu tiên phản hồi | Logistic Regression | Cho sẵn xác suất theo mô hình | Phải kiểm tra mức chuẩn của xác suất |
| Nhãn đến dần theo thời gian, cần cập nhật theo lô | Perceptron với `partial_fit`, hoặc SGDClassifier | Cập nhật không cần học lại từ đầu | Phải cố định không gian đặc trưng và thiết kế đánh giá theo thời gian |
| Muốn Perceptron ổn định hơn | Averaged Perceptron hoặc thêm phạt trọng số | Giảm nhạy thứ tự | Không bảo đảm luôn tốt hơn |
| Ranh giới có thể cong, hoặc kiểu XOR | SVM có kernel, thêm đặc trưng tương tác, hoặc MLP | Phi tuyến | Tốn tài nguyên hơn, nên thử tuyến tính trước |
| Ngữ cảnh dài, phủ định phức tạp | Mô hình ngữ cảnh | Hiểu câu tốt hơn từng từ | Chưa nằm trong bộ tài liệu này |
| Muốn kết hợp tri thức từ điển | Perceptron, SVM hoặc LR kèm điểm của từ điển làm đặc trưng thêm | Vừa có tri thức có sẵn vừa học từ dữ liệu | Đây là đề xuất của người biên soạn, chưa được kiểm tra |
| Đúng hơn trên đánh giá khách sạn | Chưa biết | Chưa có bằng chứng | Phải đo macro-F1 trên cùng phần chấm điểm cuối |

Không có thuật toán luôn đứng đầu trên mọi tập dữ liệu. Kết quả gắn với cách chọn tham số, cách chia dữ liệu và thiết kế đánh giá. Nếu Perceptron đạt gần bằng SVM hoặc LR nhưng dao động nhiều hơn giữa các hạt giống và không có xác suất, đó là lý do cân nhắc chọn cách ổn định hơn, không nên chọn chỉ vì luật cập nhật gọn.

<a id="muc-9"></a>

**9. Mã chạy thử**

Cần Python và scikit-learn. Mã đã được chạy thử với scikit-learn 1.9.1 vào ngày 30/10/2026. Dùng cùng dữ liệu tiếng Anh với hai bài SVM và Logistic Regression để so sánh. Dữ liệu cực nhỏ, chỉ minh họa quy trình, không dùng để tuyên bố hiệu năng. Nhãn 1 là khen, nhãn 0 là chê, không có lớp trung tính.

Mã gồm hai phần: phần đầu tính tay đúng hai bước cập nhật, phần sau chạy Perceptron của scikit-learn với nhiều hạt giống.

```python
import numpy as np
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import Perceptron
from sklearn.metrics import f1_score, confusion_matrix

# Phần 1: hai bước cập nhật tính tay, nhãn dùng +1 và -1
w, b = np.zeros(2), 0.0
for x, y in [(np.array([1.0, 0.0]), 1), (np.array([0.0, 1.0]), -1)]:
    if y * (w @ x + b) <= 0:
        w = w + y * x
        b = b + y
assert np.allclose(w, [1.0, -1.0]) and b == 0.0

# Phần 2: TF-IDF + Perceptron
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

for seed in [42, 0, 1, 2, 3]:
    model = Pipeline([
        ("tfidf", TfidfVectorizer(ngram_range=(1, 2), lowercase=True)),
        ("clf", Perceptron(penalty=None, eta0=1.0, max_iter=20,
                           tol=None, shuffle=True, random_state=seed)),
    ])
    model.fit(train, y_train)
    vec = model.named_steps["tfidf"]
    clf = model.named_steps["clf"]
    assert "zzholdout" not in vec.vocabulary_
    assert not hasattr(clf, "predict_proba")      # không có xác suất

    pred = model.predict(test)
    scores = model.decision_function(test)
    s2 = model.decision_function([khong_co_tu_cam_xuc])[0]
    print("seed", seed, "| thứ tự lớp:", clf.classes_.tolist())
    print("  điểm:", [round(float(s), 3) for s in scores], "nhãn:", pred.tolist())
    print("  Macro-F1 minh họa:", round(f1_score(y_test, pred, average="macro"), 3))
    print("  Câu không có từ cảm xúc:", round(float(s2), 3),
          int(model.predict([khong_co_tu_cam_xuc])[0]))

    if seed == 42:
        print(confusion_matrix(y_test, pred, labels=[0, 1]))
        names = np.array(vec.get_feature_names_out())
        wts = clf.coef_[0]
        order = np.argsort(wts)
        print("  Trọng số âm nhất:", [(names[i], round(float(wts[i]), 2)) for i in order[:3]])
        print("  Trọng số dương nhất:", [(names[i], round(float(wts[i]), 2)) for i in order[::-1][:3]])
```

Kết quả khi chạy thử, rút gọn. Hai bước cập nhật tính tay ra w bằng (1, -1) và b bằng 0, khớp phép kiểm tra trong mã.

| Hạt giống | Điểm bốn câu thử theo thứ tự | Nhãn | Macro-F1 | Câu không có từ cảm xúc |
|---|---|---|---|---|
| 42 | 0.065, 0.423, -0.570, -0.467 | 1, 1, 0, 0 | 1.0 | điểm 0.338, nhãn 1 |
| 0 | 0.376, 0.118, -0.090, -0.076 | 1, 1, 0, 0 | 1.0 | điểm 0.348, nhãn 1 |
| 1 | -0.152, 0.335, -0.331, -0.363 | 0, 1, 0, 0 | 0.733 | điểm 0.0, nhãn 0 |
| 2 | 0.369, 0.454, -0.383, -0.080 | 1, 1, 0, 0 | 1.0 | điểm 0.348, nhãn 1 |
| 3 | 0.376, 0.118, -0.090, -0.076 | 1, 1, 0, 0 | 1.0 | điểm 0.348, nhãn 1 |

Với hạt giống 42: thứ tự lớp là 0 rồi 1, nên điểm dương hướng đến nhãn 1, tức khen. Trọng số âm nhất là terrible, dirty, rude. Trọng số dương nhất là friendly, comfortable, very friendly.

Cách đọc kết quả này.

1. Bốn trong năm hạt giống cho macro-F1 bằng 1, nhưng hạt giống 1 sai câu "Friendly staff and a clean room." và macro-F1 tụt còn khoảng 0.73. Cùng dữ liệu, cùng cấu hình, chỉ đổi thứ tự mẫu. Đây là lý do phải báo nhiều hạt giống.
2. Macro-F1 bằng 1 ở bốn lần kia chỉ có nghĩa dữ liệu thử rất dễ, không có nghĩa mô hình tốt. Dữ liệu 8 câu học và 4 câu thử quá nhỏ để chọn mô hình thắng cuộc.
3. Điểm quyết định không có thang xác suất. Câu đầu tiên ở hạt giống 42 có điểm 0.065, rất gần 0, nhưng ở hạt giống 0 điểm là 0.376. Độ lớn của điểm không đọc được như mức tự tin.
4. Câu không có từ cảm xúc ra nhãn khen ở bốn hạt giống và chê ở một hạt giống (điểm đúng bằng 0). Nhãn luôn được trả ra, như tình huống 5.
5. Dòng trọng số cho thấy mô hình học được các từ khen chê hợp lý trên dữ liệu nhỏ này.

Từ zzholdout là chữ giả để kiểm tra rằng dữ liệu thử không đi vào từ vựng. Pipeline không tự ngăn mọi loại rò rỉ. Vẫn phải chia đúng theo khách sạn, loại bản trùng và chọn tham số ngoài phần chấm điểm cuối. Với tiếng Việt thật, cần kiểm tra cách tách từ, cách xử lý phủ định và chất lượng nhãn.

<a id="muc-10"></a>

**10. Tài liệu tham khảo**

- **[S1]** Rosenblatt, F. (1958). *The perceptron: A probabilistic model for information storage and organization in the brain*. Psychological Review, 65(6), 386 đến 408. [DOI công trình](https://doi.org/10.1037/h0042519).
- **[S2]** scikit-learn. [Perceptron API](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.Perceptron.html).
- **[S3]** scikit-learn. [Linear Models](https://scikit-learn.org/stable/modules/linear_model.html).
- **[S4]** scikit-learn. [Stochastic Gradient Descent](https://scikit-learn.org/stable/modules/sgd.html).
- **[S5]** scikit-learn. [Common pitfalls](https://scikit-learn.org/stable/common_pitfalls.html).

S1 là mốc lịch sử. Từ "probabilistic" trong tên bài gốc không có nghĩa API Perceptron hiện đại xuất xác suất. Nội dung khái niệm và quy trình lấy từ bài Perceptron dài đã gửi, đơn giản hóa theo định dạng file SVM. Phần so sánh với các cách Lexicon Based, SVM và Logistic Regression dựa trên các file trước. Ví dụ khách sạn, trọng số giả định, lượt học tính tay ở tình huống 2 và kế hoạch kiểm tra là đề xuất của người biên soạn, chưa phải kết quả của một thí nghiệm thực tế. Các con số ở mục 9 đến từ lần chạy mã thật với scikit-learn 1.9.1, tài liệu trực tuyến có thể thuộc phiên bản khác.
