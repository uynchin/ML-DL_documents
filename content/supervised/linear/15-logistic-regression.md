# Logistic Regression: giải thích đơn giản

Nhánh Machine Learning, phần học có giám sát tuyến tính. Bài này giữ cấu trúc 10 mục như file SVM và các file Lexicon Based, mục 4 là cách học và cách đọc điểm. Điểm khác lớn nhất so với SVM: Logistic Regression cho ra một con số xác suất theo mô hình, nên bài có thêm phần nói về cách đọc và kiểm tra con số đó.

**Mục lục**

- [1. Logistic Regression là gì](#muc-1)
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

**1. Logistic Regression là gì**

Logistic Regression, viết tắt LR, tiếng Việt là hồi quy logistic. Dù tên có chữ "regression" (hồi quy), trong bài này nó dùng để **phân loại**: học từ các đánh giá khách sạn đã có nhãn khen hoặc chê rồi đoán nhãn cho đánh giá mới [S1].

Có ba bước trong đầu của chương trình.

1. Đổi mỗi đánh giá thành một dãy số. Cách phổ biến là TF-IDF, giống như bài SVM: mỗi từ hoặc cụm từ có một con số.
2. Học một bộ trọng số, mỗi từ một trọng số, cộng thêm một số điều chỉnh chung gọi là b. Từ nghiêng về khen có trọng số dương, từ nghiêng về chê có trọng số âm.
3. Với đánh giá mới, nhân trọng số với dãy số rồi cộng lại thành điểm z. Sau đó đưa z qua một hàm uốn cong tên là sigmoid để ra **một số từ 0 đến 1**, gọi là xác suất theo mô hình. Số này lớn hơn ngưỡng (thường là 0.5) thì chọn nhãn khen.

Điểm khác với SVM nằm ở bước 3 và ở cách học. SVM chỉ quan tâm điểm nằm bên nào của số 0 và cách đường phân chia bao xa. LR đặt một mô hình xác suất, rồi học trọng số sao cho các nhãn người đã gán trở nên "dễ xảy ra" nhất theo mô hình đó [S1] [S2].

Khác với các cách Lexicon Based, LR không có từ điển điểm dựng sẵn. Trọng số của từng từ do máy học từ nhãn của người. Chất lượng nhãn quyết định chất lượng mô hình.

Có ba điều dễ nhầm.

1. Xác suất 0.8 không có nghĩa khách hài lòng 80 phần trăm, cũng không có nghĩa mô hình đúng 80 phần trăm. Đó là ước lượng của mô hình, chưa chắc đã chuẩn.
2. Odds và xác suất là hai thứ khác nhau. Odds là tỷ lệ "khen so với chê". Xác suất 0.8 ứng với odds bằng 4 (0.8 chia 0.2), không phải 0.8.
3. Trọng số dương của một từ không chứng minh từ đó gây ra sự hài lòng. Trọng số chỉ cho biết từ đó giúp phân biệt hai nhãn trong dữ liệu đã học.

<a id="muc-2"></a>

**2. So sánh với các cách trước**

Thông tin về các cách Lexicon Based lấy từ các file trước. Thông tin về SVM lấy từ file SVM. Thông tin về các cách học máy khác lấy từ tài liệu nguồn đã đối chiếu.

**So với các cách dùng từ điển.**

| Điểm so sánh | Các cách từ điển như VADER, AFINN, SentiStrength | Logistic Regression |
|---|---|---|
| Điểm của từ đến từ đâu | Có sẵn trong từ điển, do tác giả hoặc người đánh giá gán | Máy học từ nhãn của người trên dữ liệu của nghiên cứu |
| Cần nhãn để chạy không | Không, nhưng cần nhãn để kiểm tra | Có, cần nhãn để huấn luyện và thêm nhãn để kiểm tra |
| Từ ngoài từ điển | Bị bỏ qua, không có điểm | Từ chưa thấy khi học bị bỏ qua trong vector |
| Học từ ngữ riêng của khách sạn | Không, trừ khi người tự thêm | Có, nếu từ đó xuất hiện đủ trong dữ liệu học |
| Xử lý "not" | Tùy cách, VADER và SentiStrength có sẵn | Không có sẵn. Học được nếu dùng cụm hai từ như "not good" và cụm đó có trong dữ liệu học |
| Có luôn ra nhãn không | Có thể ra thiếu dữ liệu khi không khớp từ nào | Luôn ra một nhãn và một xác suất, kể cả khi câu không có từ cảm xúc |
| Đầu ra | Điểm cảm xúc hoặc nhãn | Xác suất theo mô hình, nhãn theo ngưỡng, trọng số từng từ |
| Rủi ro riêng | Từ điển khác miền | Học nhầm dấu hiệu không liên quan, như tên khách sạn. Xác suất có thể quá tự tin hoặc quá dè dặt |
| Tiếng Việt | Phần lớn chỉ tiếng Anh | Dùng được nếu có nhãn tiếng Việt và cách tách từ phù hợp |

**So với các cách học máy gần nhau.**

| Phương pháp | Điểm khác với Logistic Regression |
|---|---|
| SVM tuyến tính | Tối ưu biên rộng nhất, đầu ra là điểm quyết định, không phải xác suất. Muốn xác suất phải hiệu chuẩn thêm |
| Perceptron | Cập nhật trọng số khi gặp mẫu sai, không có mô hình xác suất, không tối ưu biên |
| Linear Regression | Dự đoán một số liên tục. Nếu dùng cho nhãn khen chê, kết quả có thể lọt ra ngoài khoảng 0 đến 1 |
| Multinomial Logistic | LR mở rộng cho nhiều nhãn loại trừ nhau (khen, chê, trung tính), dùng hàm softmax |
| Ordinal Logistic | Dành cho nhãn có thứ tự như 1 đến 5 sao, giả định riêng |

Khi một đánh giá có thể mang nhiều cảm xúc cùng lúc, cần thiết kế multi-label, không ép các nhãn loại trừ nhau.

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
| LR chỉ dùng từng từ | Giống SVM: "good" thường dương, dễ ra xác suất khen cao, sai |
| LR có thêm cụm hai từ | Giống SVM: đúng nếu cụm phủ định đã xuất hiện đủ trong dữ liệu học với nhãn chê. Khác ở chỗ ra thêm xác suất |

Điểm khác biệt lớn nhất so với từ điển: từ điển đã có điểm trước khi nhìn dữ liệu, còn LR tạo điểm từ dữ liệu. Điểm khác biệt lớn nhất so với SVM: hai cách này thường cho nhãn gần giống nhau trên cùng đặc trưng, nhưng LR cho sẵn xác suất theo mô hình còn SVM thì không.

<a id="muc-3"></a>

**3. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Quy định thế nào là khen, chê, trung tính, và xử lý đánh giá vừa khen vừa chê ra sao | Người |
| Gán nhãn cho tập đánh giá mẫu | Người |
| Chọn cách đổi văn bản thành số: từng từ, cụm hai từ, cách bỏ từ quá thông dụng | Người |
| Chọn các tham số như C, loại regularization, có cân bằng lớp không | Người đặt quy tắc, máy thử và đo |
| Chia dữ liệu thành phần học, phần chỉnh và phần chấm điểm cuối, theo khách sạn | Người |
| **Chọn ngưỡng đổi xác suất thành nhãn** | **Người, dựa trên chi phí sai và phần chỉnh** |
| Đổi văn bản thành vector | Máy |
| Học trọng số và số điều chỉnh b | Máy |
| Tính xác suất cho văn bản mới | Máy |
| Đổi xác suất thành nhãn theo ngưỡng | Máy |
| So nhãn của máy với đáp án, tính chỉ số | Máy tính, người đọc các câu sai |
| Kết luận trong bài viết | Người |

So với SVM, LR có thêm một việc của người: chọn ngưỡng. Ngưỡng 0.5 là mặc định tiện, không phải chân lý. Nếu bỏ sót một lời phàn nàn tốn kém hơn báo nhầm, người có thể hạ ngưỡng. Đổi ngưỡng chỉ làm nhãn thay đổi, xác suất của mô hình đã học vẫn giữ nguyên.

<a id="muc-4"></a>

**4. Cách học và cách đọc điểm**

Máy làm ba việc.

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Học từ vựng và TF-IDF trên phần học, đổi mỗi đánh giá thành vector | Mỗi đánh giá là một dãy số |
| 2 | Tìm trọng số w và số điều chỉnh b sao cho các nhãn thật trở nên "dễ xảy ra" nhất, có kìm trọng số không quá lớn | Mỗi từ hoặc cụm có một trọng số |
| 3 | Với đánh giá mới, tính điểm z, đưa qua sigmoid ra xác suất, rồi so với ngưỡng | Xác suất và nhãn |

Công thức dạng văn bản:

```text
z = (w1 nhân x1) cộng (w2 nhân x2) cộng ... cộng b
xác suất khen p = 1 chia (1 cộng e mũ trừ z)
```

| Ký hiệu | Ý nghĩa |
|---|---|
| x1, x2, ... | Các con số của đánh giá, ví dụ TF-IDF của từng từ |
| w1, w2, ... | Trọng số của từng từ, do máy học |
| b | Số điều chỉnh chung, do máy học |
| z | Điểm thô, có thể âm hoặc dương. Tên chính thức là log-odds |
| p | Xác suất khen theo mô hình, từ 0 đến 1 |

Đọc nhanh mối quan hệ giữa z, xác suất và odds:

| z | Xác suất khen p | Odds (khen so với chê) | Ý nghĩa |
|---|---|---|---|
| 0 | 0.50 | 1 | Mô hình không nghiêng bên nào |
| 0.7 | 0.67 | 2.01 | Nghiêng về khen |
| 2.2 | 0.90 | 9.03 | Nghiêng mạnh về khen |
| -0.7 | 0.33 | 0.50 | Nghiêng về chê |

Điểm quan trọng: mô hình tuyến tính ở z (log-odds), không tuyến tính ở xác suất. Vì vậy mỗi từ cộng hoặc trừ vào z, và tác động lên xác suất nhỏ dần khi p đã gần 0 hoặc gần 1.

Cách máy học trọng số, dạng giảng giải đơn giản:

```text
Giảm: mức phạt của tất cả đánh giá, cộng một khoản phạt cho trọng số lớn
Mức phạt của một đánh giá = trừ log của (xác suất mô hình gán cho nhãn thật)
```

| Mô hình gán xác suất cho nhãn thật | Mức phạt | Ý nghĩa |
|---|---|---|
| 0.9 | 0.11 | Đúng và tự tin, phạt rất nhẹ |
| 0.6 | 0.51 | Đúng nhưng dè dặt, vẫn bị phạt |
| 0.2 | 1.61 | Sai và tự tin, phạt nặng |

So với SVM: SVM phạt bằng 0 khi đánh giá đúng phía và nằm ngoài biên. LR luôn phạt một chút kể cả khi đúng, nên LR liên tục "đẩy" xác suất của nhãn đúng lên cao hơn. Cả hai đều phạt nặng khi sai phía.

Về C: trong scikit-learn, C nhỏ nghĩa là kìm trọng số mạnh hơn, mô hình đơn giản hơn và xác suất thường dè dặt hơn. C lớn nghĩa là kìm yếu hơn, mô hình bám dữ liệu học sát hơn, dễ bám vào mẫu nhiễu. Hướng này giống C của SVM [S2]. Giá trị C phải chọn trên phần chỉnh, không chọn trên phần chấm điểm cuối.

Về thứ tự lớp: cột xác suất của `predict_proba` đi theo thứ tự `classes_`. Phải kiểm tra cột nào là lớp khen, không mặc định cột thứ hai là khen [S2].

Về ngưỡng: ngưỡng không phải tham số học. Đổi ngưỡng không cần học lại. Đổi C hoặc đặc trưng thì phải học lại và xác suất có thể đổi theo.

Về solver: công thức trên chỉ để giảng giải. Thư viện dùng các thuật toán tối ưu như lbfgs, không cập nhật đơn giản từng mẫu một.

<a id="muc-5"></a>

**5. Ví dụ năm tình huống**

Các trọng số trong bảng là **giả định** để tính cho dễ, không phải mô hình học thật, và dùng chung với bài SVM để so sánh. Đặc trưng chỉ có giá trị 0 hoặc 1, nghĩa là từ có xuất hiện hay không. Kết quả thật của mô hình học được xem ở mục 9.

| Đặc trưng | Trọng số giả định |
|---|---|
| clean | 1.2 |
| friendly | 0.9 |
| dirty | -1.1 |
| rude | -1.0 |
| not clean | -2.0 |
| aurora | 1.5 |
| b, số điều chỉnh chung | 0.1 |

**Tình huống 1: tính xác suất cho một câu.** Câu: "The staff were friendly and the room was clean."

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Tìm đặc trưng có trong câu | friendly, clean |
| 2 | Cộng trọng số và b | 0.9 cộng 1.2 cộng 0.1 |
| 3 | Điểm z | z = 2.2 |
| 4 | Đưa qua sigmoid | p khoảng 0.90 |
| 5 | So với ngưỡng 0.5 | Khen |

SVM trên cùng trọng số cho điểm 2.2 và dừng ở đó. LR đi thêm một bước để ra 0.90.

**Tình huống 2: mức phạt trong lúc học.** Bốn đánh giá trong phần học, với xác suất khen do mô hình tạm thời đưa ra.

| Đánh giá và nhãn thật | Xác suất mô hình gán cho nhãn thật | Mức phạt |
|---|---|---|
| "Spotless room, lovely staff." Nhãn khen | 0.90 | 0.11, đúng và tự tin |
| "The room was fine." Nhãn khen | 0.60 | 0.51, đúng nhưng dè dặt |
| "The room was dirty." Nhãn chê | 0.70 | 0.36, đúng, khá tự tin |
| "The staff were rude." Nhãn chê, nhưng mô hình đang cho khen 0.80, tức chê chỉ 0.20 | 0.20 | 1.61, sai và tự tin nên phạt nặng |

Máy điều chỉnh trọng số để giảm tổng mức phạt này, đồng thời kìm trọng số không quá lớn.

**Tình huống 3: chữ "not".** Câu: "The bed was not clean."

| Cách làm | Tính | z | Xác suất khen | Kết quả |
|---|---|---|---|---|
| Chỉ dùng từng từ, đặc trưng có clean | 1.2 cộng 0.1 | 1.3 | 0.79 | Khen, sai |
| Thêm cụm hai từ, đặc trưng có clean và not clean | 1.2 trừ 2.0 cộng 0.1 | -0.7 | 0.33 | Chê, đúng |

Cụm "not clean" chỉ có trọng số âm nếu cụm này đã xuất hiện đủ trong phần học với nhãn chê. Trong lần chạy ở mục 9, ví dụ học có sẵn một câu chứa cụm này. Cần kiểm tra bằng đánh giá có phủ định ở phần chỉnh, không giả định.

**Tình huống 4: học nhầm tên khách sạn.** Giả sử phần học chỉ có các đánh giá khen của khách sạn Aurora, nên đặc trưng "aurora" có trọng số dương.

| Câu | Đặc trưng khớp | Tính | z | Xác suất khen | Kết quả |
|---|---|---|---|---|---|
| "Aurora was dirty." | aurora, dirty | 1.5 trừ 1.1 cộng 0.1 | 0.5 | 0.62 | Khen, sai |
| "Bella was dirty." khách sạn khác | dirty | -1.1 cộng 0.1 | -1.0 | 0.27 | Chê, đúng |

Chia ngẫu nhiên các câu, chung khách sạn ở cả hai phần, sẽ thấy điểm cao vì tên Aurora giúp đoán đúng. Sang khách sạn mới thì dấu hiệu đó biến mất. Vì vậy phải chia theo khách sạn. Với LR có thêm một cảnh báo: xác suất 0.62 trông như "khá chắc", nhưng nó dựa trên dấu hiệu sai.

**Tình huống 5: câu không có từ cảm xúc.** Câu: "The room was zzholdout." Từ cuối là chữ giả chưa xuất hiện khi học.

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Bỏ từ chưa thấy khi học | zzholdout không vào vector |
| 2 | Còn lại các từ thông dụng như the, room, was | Có trong từ vựng |
| 3 | Tính xác suất, lấy từ lần chạy thật ở mục 9 | p khoảng 0.54 |
| 4 | So với ngưỡng 0.5 | Khen |

Mô hình vẫn ra nhãn khen dù câu không chứa từ cảm xúc nào, vì các từ thông dụng như "the room was" xuất hiện nhiều hơn một chút ở các câu khen trong dữ liệu nhỏ này. Từ điển sẽ ghi thiếu dữ liệu, còn LR luôn trả một xác suất. Xác suất sát 0.5 là dấu hiệu mô hình gần như không có bằng chứng, nhưng nhãn vẫn được in ra. Vì vậy phải kiểm tra riêng tỷ lệ câu có ít nhất một từ có ích, và không coi mọi nhãn máy là bằng chứng.

<a id="muc-6"></a>

**6. Lỗi hay gặp**

1. Đọc xác suất 0.8 như "khách hài lòng 80 phần trăm" hoặc "mô hình đúng 80 phần trăm". Đó là ước lượng của mô hình, cần kiểm tra mức chuẩn.
2. Cho rằng có sigmoid là xác suất đã chuẩn. Sigmoid chỉ đưa điểm vào khoảng 0 đến 1. Xác suất có chuẩn hay không phải đo bằng dữ liệu riêng [S3].
3. Đọc trọng số như phần trăm đổi xác suất. Trọng số 0.7 nhân odds lên khoảng 2.01 lần, không phải tăng xác suất 70 điểm phần trăm. Mức đổi của xác suất phụ thuộc điểm xuất phát: từ p = 0.5 lên 0.67, nhưng từ p = 0.9 chỉ lên 0.95.
4. Nhầm odds với xác suất. Xác suất 0.8 ứng với odds bằng 4.
5. Học TF-IDF trên toàn bộ dữ liệu trước khi chia hoặc trước khi kiểm chứng chéo. Từ vựng và hệ số IDF cũng là thứ được học, nên làm vậy là rò rỉ. Phải đặt TF-IDF và LR trong cùng một quy trình, và học lại trong từng lượt [S4].
6. Chọn ngưỡng trên phần chấm điểm cuối. Ngưỡng cũng là một lựa chọn, phải chọn trên phần chỉnh.
7. Tin ngưỡng 0.5 là đúng cho mọi mục đích. Chi phí bỏ sót lời phàn nàn và chi phí báo nhầm có thể khác nhau.
8. Dùng điểm hay xác suất gần 0.5 làm nhãn trung tính. Gần 0.5 chỉ là khó phân biệt giữa hai lớp. Muốn có lớp trung tính phải gán nhãn và huấn luyện ba lớp.
9. Hiểu trọng số dương là từ đó gây ra sự hài lòng. Trọng số là tham số dự đoán, không phải tác động nhân quả.
10. Chia ngẫu nhiên khi dữ liệu có nhiều đánh giá của cùng khách sạn, hoặc có nhiều bản trùng. Kết quả đẹp hơn thực tế.
11. Để tên khách sạn, tên nhân viên, mã đặt phòng nằm trong đặc trưng. Mô hình có thể dùng chúng thay cho nội dung.
12. Dùng nhãn của VADER hay nhãn của từ điển làm đáp án để huấn luyện rồi báo độ đúng so với chính nhãn đó. Đó là độ giống từ điển, không phải độ đúng. Đáp án phải do người gán.
13. Không kiểm tra thứ tự lớp trong `classes_`, rồi lấy nhầm cột xác suất.
14. Bỏ qua cảnh báo chưa hội tụ. Cần kiểm tra thang đo, solver, số vòng lặp tối đa, thay vì chỉ tăng số vòng mãi.
15. Báo p-value hay khoảng tin cậy từ một bộ phân loại chỉ xuất hệ số. Nếu mục tiêu là suy luận thống kê thì phải dùng công cụ và giả định phù hợp, đó là bài toán khác với phân loại văn bản.
16. Dùng LR cho điểm 1 đến 5 sao như thể là nhãn rời rạc mà không nghĩ đến thứ tự. Điểm sao có thứ tự, cần cân nhắc cách thiết kế khác.

<a id="muc-7"></a>

**7. Làm sao biết kết quả đáng tin**

Kiểm tra theo ba bước, giống file SVM. LR có thêm phần kiểm tra xác suất.

Bước 1. Người tạo đáp án.

1. Chọn khoảng 100 câu trở lên từ nhiều khách sạn khác nhau. Càng nhiều càng tốt, nhưng cần đủ để chia ba phần.
2. Hai người gán nhãn khen, chê hoặc trung tính, mỗi người làm riêng.
3. Máy tính mức hai người đồng ý với nhau, gọi là hệ số kappa. Càng gần 1 thì càng đồng ý.
4. Chỗ hai người khác nhau thì bàn lại và chọn một nhãn. Quy định trước cách xử lý đánh giá vừa khen vừa chê. Nếu dùng số sao làm nhãn, phải ghi quy tắc đổi sao thành nhãn và cách xử lý đánh giá chữ mâu thuẫn với sao.

Bước 2. Chia đáp án làm ba phần, theo khách sạn.

| Phần | Dùng để làm gì |
|---|---|
| Phần học | Máy học trọng số. Học TF-IDF cũng chỉ dùng phần này |
| Phần chỉnh | Chọn C, chọn đặc trưng, cân bằng lớp, thử cụm hai từ, **chọn ngưỡng**, thử hiệu chuẩn xác suất |
| Phần chấm điểm cuối | Chỉ dùng một lần ở cuối, không sửa gì sau khi thấy kết quả |

Câu của cùng một khách sạn chỉ nằm ở một phần. Nên loại các bản trùng trước khi chia. Có thể dùng kiểm chứng chéo trên phần học để chọn tham số. Quy trình trong mỗi lượt phải gồm cả bước tạo vector và bước học [S4].

Bước 3. Máy so nhãn của LR với đáp án trên phần chấm điểm cuối, người báo các con số sau.

| Con số | Ý nghĩa đơn giản |
|---|---|
| Độ đúng trung bình của các nhãn (macro-F1) | Máy đúng bao nhiêu, tính riêng cho từng nhãn rồi lấy trung bình |
| Bảng nhầm lẫn | Máy hay nhầm nhãn nào thành nhãn nào |
| So với mốc nhãn phổ biến nhất | Mô hình có hơn cách luôn đoán nhãn nhiều nhất không |
| Tỷ lệ câu có ít nhất một từ có ích đã thấy khi học | Mô hình dựa vào nội dung được bao nhiêu phần |
| Kết quả theo nhóm độ dài và theo khách sạn | Câu ngắn, câu dài, khách sạn khác có đúng như nhau không |
| **Log loss và Brier score** | **Xác suất sai nhiều hay ít. Càng thấp càng tốt** |
| **Đường hiệu chuẩn** | **Các câu được gán khoảng 0.8 thì thật sự có khoảng 80 phần trăm là khen không** |

Với LR có thêm bốn kiểm tra đáng làm.

1. Đọc trọng số lớn nhất và nhỏ nhất xem mô hình dựa vào từ có nghĩa hay dấu hiệu ngẫu nhiên, như tên khách sạn.
2. Chạy lại với các giá trị C và đặc trưng khác để xem kết luận có vững không. Không lặp mãi sau khi đã có bằng chứng đủ.
3. Chạy trên khách sạn hoặc thời gian mới nếu mục tiêu là áp dụng sang dữ liệu khác, vì xác suất dễ lệch khi dữ liệu đổi.
4. Nếu xác suất quá tự tin hoặc quá dè dặt, thử hiệu chuẩn bằng dữ liệu không dùng để học mô hình [S3].

Lưu ý: AUC cao chỉ cho biết mô hình xếp khen cao hơn chê, không chứng minh xác suất đã chuẩn.

Để so LR với SVM và các cách từ điển cho công bằng, chạy tất cả trên cùng phần chấm điểm cuối rồi so macro-F1. LR và SVM cần phần học riêng, từ điển thì không. Hai bên vẫn so trên cùng phần chấm điểm cuối.

Khi viết kết luận, nhãn của máy không đủ để nói khách hài lòng nhiều hay ít. Nói "70 phần trăm mẫu được gán khen" chưa có nghĩa 70 phần trăm khách hài lòng, vì còn sai số của mô hình và cách lấy mẫu. Người để lại đánh giá cũng không nhất thiết đại diện cho toàn bộ khách.

<a id="muc-8"></a>

**8. Chọn cái nào**

| Tình huống | Cách hợp hơn | Lý do | Điểm yếu |
|---|---|---|---|
| Chưa có nhãn để huấn luyện | Các cách từ điển | Chạy ngay, không cần nhãn | Từ điển khác miền, phải kiểm tra |
| Có nhãn, cần baseline học máy cho văn bản nhiều chiều | LR hoặc SVM tuyến tính | Cả hai dùng tốt với ma trận thưa như TF-IDF, xem được trọng số | Không hiểu ngữ cảnh dài |
| Cần xác suất để xếp ưu tiên phản hồi hoặc đổi ngưỡng theo chi phí | Logistic Regression | Cho sẵn xác suất theo mô hình | Phải kiểm tra mức chuẩn của xác suất |
| Chỉ cần nhãn, không cần xác suất | LR hoặc SVM đều được | Thường cho kết quả gần nhau | Nên đo trên cùng dữ liệu |
| Muốn cách rất đơn giản | Perceptron | Cập nhật theo mẫu sai | Không có xác suất, không tối ưu biên |
| Ranh giới có thể cong | SVM có kernel | Phi tuyến trong đặc trưng gốc | Tốn tài nguyên hơn, nên thử tuyến tính trước |
| Nhiều nhãn loại trừ nhau (khen, chê, trung tính) | Multinomial Logistic | Cho xác suất cho từng nhãn | Cần nhãn đủ cho từng lớp |
| Nhãn có thứ tự 1 đến 5 sao | Ordinal Logistic hoặc thiết kế riêng | Tôn trọng thứ tự | Giả định riêng, cần kiểm tra |
| Nghiên cứu xác suất khách quay lại theo biến khảo sát | Công cụ hồi quy logistic có thống kê suy luận | Có sai số chuẩn, kiểm định | Đây là bài toán khác với phân loại văn bản, không dùng nguyên cách đọc hệ số TF-IDF |
| Ngữ cảnh dài, phủ định phức tạp | Mô hình ngữ cảnh | Hiểu câu tốt hơn từng từ | Chưa nằm trong bộ tài liệu này |
| Muốn kết hợp tri thức từ điển | LR kèm điểm của từ điển làm đặc trưng thêm | Vừa có tri thức có sẵn vừa học từ dữ liệu | Đây là đề xuất của người biên soạn, chưa được kiểm tra |
| Đúng hơn trên đánh giá khách sạn | Chưa biết | Chưa có bằng chứng | Phải đo macro-F1 trên cùng phần chấm điểm cuối |

Không có thuật toán luôn đứng đầu trên mọi tập dữ liệu. Kết quả gắn với cách chọn tham số, cách chia dữ liệu và thiết kế đánh giá. Nếu LR đạt gần bằng SVM và còn cho sẵn xác suất, đó là một lý do chọn LR, không nên chọn chỉ theo điểm trung bình cao hơn một chút.

<a id="muc-9"></a>

**9. Mã chạy thử**

Cần Python và scikit-learn. Mã đã được chạy thử với scikit-learn 1.9.1 vào ngày 30/10/2026. Dùng cùng dữ liệu tiếng Anh với bài SVM để so sánh. Dữ liệu cực nhỏ, chỉ minh họa quy trình, không dùng để tuyên bố hiệu năng. Nhãn 1 là khen, nhãn 0 là chê, không có lớp trung tính.

```python
import numpy as np
from sklearn.pipeline import Pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import f1_score, confusion_matrix, log_loss

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
    ("clf", LogisticRegression(C=1.0, solver="lbfgs", max_iter=2000)),
])
model.fit(train, y_train)

vec = model.named_steps["tfidf"]
clf = model.named_steps["clf"]
assert "zzholdout" not in vec.vocabulary_
print("Thứ tự lớp:", clf.classes_.tolist())

cot_khen = int(np.flatnonzero(clf.classes_ == 1).item())  # cột xác suất của lớp khen
prob = model.predict_proba(test)
pred = model.predict(test)
for text, p, l in zip(test, prob[:, cot_khen], pred):
    print(round(float(p), 3), int(l), text)
print("Macro-F1 minh họa:", f1_score(y_test, pred, average="macro"))
print(confusion_matrix(y_test, pred, labels=[0, 1]))
print("Log loss minh họa:", round(log_loss(y_test, prob, labels=clf.classes_), 4))

p2 = model.predict_proba([khong_co_tu_cam_xuc])[0, cot_khen]
print("Câu không có từ cảm xúc:", round(float(p2), 3),
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
| Thứ tự lớp | 0 rồi 1, nên cột xác suất của lớp khen là cột thứ hai |
| "Friendly staff and a clean room." | p khen khoảng 0.51, nhãn 1 |
| "Delicious breakfast and comfortable bed." | p khen khoảng 0.54, nhãn 1 |
| "Dirty room and terrible service." | p khen khoảng 0.44, nhãn 0 |
| "The bathroom was noisy and the staff were rude." | p khen khoảng 0.45, nhãn 0 |
| Macro-F1 minh họa | 1.0 |
| Log loss minh họa | khoảng 0.62 |
| Câu không có từ cảm xúc | p khen khoảng 0.54, nhãn 1 |
| Trọng số âm nhất | terrible, dirty, rude |
| Trọng số dương nhất | friendly, comfortable, were friendly |

Cách đọc kết quả này.

1. Nhãn đều đúng và macro-F1 bằng 1, nhưng chỉ vì dữ liệu thử rất dễ, không có nghĩa mô hình tốt.
2. Mọi xác suất đều sát 0.5. Với dữ liệu chỉ 8 câu và regularization mặc định, mô hình rất dè dặt. Log loss khoảng 0.62 cũng cho thấy điều đó, vì mô hình đoán đúng nhãn nhưng không tự tin. Đây là ví dụ tại sao nhãn đúng chưa đủ, cần nhìn cả chất lượng của xác suất.
3. Câu "The room was zzholdout" ra 0.54 và nhãn khen dù không có từ cảm xúc nào, như tình huống 5. Sự chênh lệch nhỏ so với các câu khen thật (0.51 và 0.54) cho thấy trên dữ liệu này xác suất 0.54 không phân biệt được "có bằng chứng khen" với "không có bằng chứng". Đó là lý do cần kiểm tra riêng tỷ lệ câu có từ có ích.
4. Dòng trọng số cho thấy mô hình học được các từ khen chê hợp lý trên dữ liệu nhỏ này.

Từ zzholdout là chữ giả để kiểm tra rằng dữ liệu thử không đi vào từ vựng. Pipeline không tự ngăn mọi loại rò rỉ. Vẫn phải chia đúng theo khách sạn, loại bản trùng và chọn tham số, ngưỡng ngoài phần chấm điểm cuối. Với tiếng Việt thật, cần kiểm tra cách tách từ, cách xử lý phủ định và chất lượng nhãn.

<a id="muc-10"></a>

**10. Tài liệu tham khảo**

- **[S1]** scikit-learn. [Linear Models: Logistic regression](https://scikit-learn.org/stable/modules/linear_model.html#logistic-regression).
- **[S2]** scikit-learn. [LogisticRegression API](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LogisticRegression.html).
- **[S3]** scikit-learn. [Probability calibration](https://scikit-learn.org/stable/modules/calibration.html).
- **[S4]** scikit-learn. [Common pitfalls and data leakage](https://scikit-learn.org/stable/common_pitfalls.html).

Các nguồn trên là tài liệu hướng dẫn của thư viện, không phải một công trình khai sinh duy nhất. Bài không gán một năm hay một tác giả cho toàn bộ lịch sử của hồi quy logistic. Nội dung khái niệm và quy trình lấy từ bài Logistic Regression dài đã gửi, đơn giản hóa theo định dạng file SVM. Phần so sánh với các cách Lexicon Based và SVM dựa trên các file trước. Ví dụ khách sạn, trọng số giả định, các xác suất tính tay trong mục 4 và 5, và kế hoạch kiểm tra là đề xuất của người biên soạn, chưa phải kết quả của một thí nghiệm thực tế. Các con số ở mục 9 đến từ lần chạy mã thật với scikit-learn 1.9.1, tài liệu trực tuyến có thể thuộc phiên bản khác.
