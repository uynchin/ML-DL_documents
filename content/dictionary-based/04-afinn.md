# AFINN: giải thích đơn giản

**Mục lục**

- [1. AFINN là gì](#muc-1)
- [2. So sánh với ba cách trước](#muc-2)
- [3. Ai làm gì](#muc-3)
- [4. Cách tính điểm và ba cách chuẩn hóa](#muc-4)
- [5. Ví dụ bốn tình huống](#muc-5)
- [6. Lỗi hay gặp](#muc-6)
- [7. Làm sao biết kết quả đáng tin](#muc-7)
- [8. Chọn cái nào trong bốn cái](#muc-8)
- [9. Mã chạy thử](#muc-9)
- [10. Tài liệu tham khảo](#muc-10)

<a id="muc-1"></a>

**1. AFINN là gì**

AFINN là một danh sách từ tiếng Anh, mỗi từ có một số nguyên từ -5 (chê rất mạnh) đến +5 (khen rất mạnh) [S1] [S2]. Ví dụ giả định: "good" là 3, "terrible" là -4.

Cách chấm rất đơn giản: tìm các từ có trong danh sách, rồi cộng điểm của chúng lại. Tổng là điểm của văn bản.

Có hai điều dễ nhầm.

1. Khoảng từ -5 đến +5 là điểm của một từ. Điểm của cả văn bản là tổng nên có thể lớn hơn nhiều. Một đánh giá có nhiều từ khen có thể ra 12 hoặc 20.
2. Điểm tổng 8 không có nghĩa "khen gấp đôi" điểm tổng 4. Tổng tăng có thể chỉ vì viết dài hơn.

Khác với VADER, AFINN không có quy tắc nào điều chỉnh điểm theo cách viết câu. Các chữ như "not", "very", dấu chấm than không tự làm điểm đổi, trừ khi danh sách có sẵn mục cho cụm đó hoặc người dùng tự thêm quy tắc [S2].

<a id="muc-2"></a>

**2. So sánh với ba cách trước**

Thông tin về ba cách trước lấy từ ba file trước.

| Điểm so sánh | SentiWordNet | VADER | TextBlob | AFINN |
|---|---|---|---|---|
| Điểm trong từ điển tính theo | Từng nghĩa của từ | Từng từ | Từng từ trong từ điển Pattern | Từng từ hoặc cụm |
| Thang điểm của từng mục | Ba điểm khen, chê, trung lập cộng lại bằng 1 | Từ -4 đến +4 | polarity từ -1 đến 1, subjectivity từ 0 đến 1 | Số nguyên từ -5 đến +5 |
| Phải chọn nghĩa | Có | Không | Không | Không |
| "not", "very", chữ hoa, dấu chấm than | Người tự viết quy tắc | Có sẵn | Có xử lý một số dấu hiệu | Không có sẵn, người tự thêm |
| Kết quả cuối | Người tự gộp điểm | Điểm compound từ -1 đến 1 | polarity và subjectivity | Tổng điểm, không bị giới hạn |
| Phụ thuộc độ dài văn bản | Tùy cách gộp | Ít, vì được chuẩn hóa | Ít, vì lấy trung bình | Nhiều, nếu dùng tổng |
| Dễ kiểm tra bằng tay | Trung bình | Trung bình | Trung bình | Dễ nhất |

Điểm khác biệt lớn nhất của AFINN: đây là cách đơn giản nhất trong bốn cách. Chỉ có từ điển và phép cộng, nên người đọc có thể tự cộng lại bằng tay để kiểm tra. Đổi lại, mọi việc về ngữ cảnh đều do người tự lo.

Đặt bốn cách cạnh nhau trên cùng một câu để thấy khác biệt. Câu: "The bed was not good." Các con số của AFINN là giả định.

| Cách | Điều gì xảy ra |
|---|---|
| SentiWordNet | Tra "good" ra điểm khen. Phải có quy tắc "not" do người viết thì mới ra chê |
| VADER | Tự nhận ra "not", ra điểm chê |
| TextBlob | Có xử lý một số dấu hiệu, cần chạy mã để biết |
| AFINN | Tra "good" ra +3, khen, sai, trừ khi danh sách có sẵn mục cụm "not good" hoặc người tự thêm quy tắc |

<a id="muc-3"></a>

**3. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Chọn đúng tệp danh sách (ghi tên và phiên bản) | Người |
| Chọn chấm từng câu hay cả bài đánh giá | Người |
| Viết quy tắc cho chữ "not" nếu cần | Người viết, máy áp dụng |
| Chọn cách chuẩn hóa: tổng, trung bình trên mục khớp, hay chia số từ | Người |
| Gán nhãn khen, chê, trung tính cho một tập câu mẫu làm đáp án | Người |
| Khớp từ hoặc cụm trong văn bản với danh sách | Máy |
| Tra điểm và cộng lại | Máy |
| Đổi điểm thành nhãn bằng ngưỡng | Máy làm, người quyết định ngưỡng |
| So nhãn của máy với đáp án, tính chỉ số | Máy tính, người đọc các câu sai |
| Kết luận trong bài viết | Người |

Điểm của từng từ trong danh sách đã có sẵn, người dùng không chấm lại. Theo bài giới thiệu của tác giả, các điểm này được gán thủ công, nhưng chi tiết cần đọc bài gốc [S1] [S3] để xác nhận trước khi viết vào bài nghiên cứu.

<a id="muc-4"></a>

**4. Cách tính điểm và ba cách chuẩn hóa**

Cách cơ bản là cộng điểm các mục khớp:

```text
S = tổng điểm của tất cả các mục khớp trong văn bản
```

Vì tổng tăng theo độ dài, người nghiên cứu thường thử thêm hai cách chuẩn hóa:

```text
S theo mục khớp = S chia cho m
S theo số từ = S chia cho N
```

| Ký hiệu | Ý nghĩa |
|---|---|
| S | Tổng điểm của các mục khớp |
| m | Số mục khớp. Nếu m bằng 0 thì ghi là thiếu dữ liệu, không ghi là 0 |
| N | Tổng số từ của văn bản. Phải ghi rõ cách đếm, ví dụ bỏ dấu câu hay không |

Ba cách này trả lời ba câu hỏi khác nhau.

| Cách | Trả lời câu hỏi nào | Điểm yếu |
|---|---|---|
| Tổng S | Văn bản có bao nhiêu bằng chứng khen chê | Văn bản dài thì tổng lớn hơn |
| S chia m | Trung bình mỗi từ cảm xúc mạnh khen chê cỡ nào | Không cho biết văn bản có nhiều hay ít từ cảm xúc |
| S chia N | Mật độ cảm xúc trong cả văn bản | Văn bản có nhiều từ không cảm xúc thì điểm bị kéo về gần 0 |

Không có cách nào đúng tuyệt đối. Người chọn một cách, ghi vào bài, rồi thử cả ba để xem kết luận có đổi không (xem mục 7).

Ngưỡng đổi điểm thành nhãn: AFINN không có sẵn mức ngưỡng chuẩn. Người chọn trên phần thử nghiệm. Ví dụ giả định, tổng lớn hơn 0 là khen, nhỏ hơn 0 là chê, bằng 0 là trung tính hoặc chưa đủ thông tin.

<a id="muc-5"></a>

**5. Ví dụ bốn tình huống**

Từ điển nhỏ dưới đây là **giả định** để tính cho dễ, không phải trích từ AFINN thật. Điểm thật phải chạy mã ở mục 9.

| Từ | Điểm giả định |
|---|---|
| good | 3 |
| friendly | 2 |
| delicious | 3 |
| clean | 2 |
| terrible | -4 |

**Tình huống 1: câu chỉ có lời khen.** Câu: "The staff were friendly and the breakfast was delicious."

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Khớp từ với danh sách | friendly, delicious |
| 2 | Tra điểm | 2 và 3 |
| 3 | Cộng lại | S = 5 |
| 4 | So với ngưỡng 0 | Khen |

**Tình huống 2: câu vừa khen vừa chê.** Câu: "The room was clean, but the service was terrible."

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Khớp từ | clean, terrible |
| 2 | Tra điểm | 2 và -4 |
| 3 | Cộng lại | S = -2 |
| 4 | Chia cho số mục khớp (m = 2) | -1 |
| 5 | Chia cho số từ (N = 9, bỏ dấu câu) | khoảng -0.22 |

Cả ba cách đều cho kết quả âm, nhưng điểm không cho biết phòng được khen còn dịch vụ bị chê. Muốn biết phải chấm riêng từng khía cạnh, giống các file trước.

**Tình huống 3: hai đánh giá cùng khen nhưng khác độ dài.**

Đánh giá A: "Good room." (2 từ, 1 mục khớp là "good").
Đánh giá B: "The room was good and the bed was good and the breakfast was good." (14 từ, 3 mục khớp, cả ba đều là "good").

| Cách tính | Đánh giá A | Đánh giá B |
|---|---|---|
| Tổng S | 3 | 9 |
| S chia m | 3 | 3 |
| S chia N | 1.5 | khoảng 0.64 |

Nếu chỉ nhìn tổng, B có vẻ khen gấp ba A. Nếu chia theo mục khớp thì hai đánh giá bằng nhau. Nếu chia theo số từ thì A lại khen mạnh hơn B. Ba cách cho ba kết luận khác nhau về cùng hai đánh giá, nên phải chọn cách phù hợp với câu hỏi nghiên cứu và nói rõ trong bài.

**Tình huống 4: có chữ "not".** Câu: "The bed was not good."

| Cách làm | Cách tính | Kết quả | Đúng hay sai |
|---|---|---|---|
| Chỉ khớp từng từ | Khớp "good", S = 3 | Khen | Sai |
| Có mục cụm "not good" trong danh sách (giả định điểm là -2), khớp cụm trước | Khớp "not good", S = -2, không cộng thêm "good" bên trong cụm | Chê | Đúng |

Ở dòng sau, nếu máy khớp cả cụm "not good" và từ "good" bên trong nó, điểm bị đếm lặp. Nên kiểm tra bằng vài câu ngắn mà người cộng tay được. Cũng cần kiểm tra xem danh sách đang dùng có thật sự chứa các cụm như vậy không, vì các bản AFINN có thể khác nhau [S2].

<a id="muc-6"></a>

**6. Lỗi hay gặp**

1. Cho rằng điểm cả văn bản nằm trong khoảng từ -5 đến +5. Khoảng đó chỉ áp dụng cho từng mục trong danh sách.
2. Coi tổng 8 là "tốt gấp đôi" tổng 4. Điểm tổng còn phụ thuộc độ dài và số lần từ xuất hiện.
3. Ghi điểm 0 khi không khớp được từ nào. Điểm 0 nghe như trung tính, trong khi thực tế là chưa có thông tin. Cần ghi riêng là thiếu dữ liệu.
4. Cộng điểm tổng của cả tháng để xem cảm xúc tăng giảm. Tổng có thể tăng chỉ vì tháng đó có nhiều đánh giá hơn. Nên dùng trung bình.
5. Bỏ chữ "not" khi lọc các từ phổ biến. Danh sách từ bị bỏ mặc định thường chứa "not". Cần giữ lại.
6. Dùng cho tiếng Việt. Danh sách là tiếng Anh, nên dùng thẳng cho tiếng Việt thì kết quả không đáng tin. Nếu dịch rồi chấm thì lỗi dịch cộng thêm vào lỗi chấm.
7. Coi mọi tệp AFINN như nhau. Cần ghi rõ tên và phiên bản tệp thật sự dùng.

<a id="muc-7"></a>

**7. Làm sao biết kết quả đáng tin**

Kiểm tra theo ba bước, giống các file trước.

Bước 1. Người tạo đáp án.

1. Chọn khoảng 100 câu từ nhiều khách sạn khác nhau.
2. Hai người gán nhãn khen, chê hoặc trung tính, mỗi người làm riêng.
3. Máy tính mức hai người đồng ý với nhau, gọi là hệ số kappa. Càng gần 1 thì càng đồng ý.
4. Chỗ hai người khác nhau thì bàn lại và chọn một nhãn.

Bước 2. Chia đáp án làm hai phần.

| Phần | Dùng để làm gì |
|---|---|
| Phần thử nghiệm | Chọn ngưỡng, thử ba cách chuẩn hóa, thử thêm quy tắc "not". Được sửa thoải mái |
| Phần chấm điểm cuối | Chỉ dùng một lần ở cuối |

Chia theo khách sạn, để câu của cùng một khách sạn chỉ nằm ở một phần.

Bước 3. Máy so nhãn với đáp án trên phần chấm điểm cuối, người báo các con số sau.

| Con số | Ý nghĩa đơn giản |
|---|---|
| Độ đúng trung bình của ba nhãn (macro-F1) | Máy đúng bao nhiêu, tính riêng cho khen, chê, trung tính rồi lấy trung bình |
| Bảng nhầm lẫn | Máy hay nhầm nhãn nào thành nhãn nào |
| Tỷ lệ câu khớp được ít nhất một mục | Máy hiểu được bao nhiêu phần dữ liệu |
| Kết quả theo nhóm độ dài | Câu ngắn và câu dài có đúng như nhau không |

Với AFINN, có thêm một kiểm tra đáng làm: chạy cả ba cách chuẩn hóa ở mục 4 và xem xếp hạng khách sạn hoặc kết luận có đổi không. Nếu đổi mạnh thì phải báo tính nhạy này trong bài, không chọn cách nào cho ra kết luận thuận lợi nhất.

Để so bốn cách cho công bằng, chạy cả bốn trên cùng phần chấm điểm cuối rồi so macro-F1. Nếu VADER không hơn AFINN thì các quy tắc của VADER chưa giúp được gì trên dữ liệu đó. AFINN là mốc đơn giản nhất để đặt các cách phức tạp hơn lên so.

Khi viết kết luận, điểm từ vựng âm không chứng minh toàn bộ trải nghiệm của khách là tiêu cực. Muốn nói khách hài lòng hơn hoặc nói về nguyên nhân cần thêm bằng chứng ở mức tương ứng.

<a id="muc-8"></a>

**8. Chọn cái nào trong bốn cái**

| Nhu cầu | Cách hợp hơn | Lý do | Điểm yếu |
|---|---|---|---|
| Kiểm tra bằng tay dễ nhất, làm mốc thấp nhất | AFINN | Chỉ có tra điểm và cộng | Không có quy tắc nào về ngữ cảnh, điểm tổng phụ thuộc độ dài |
| Phân biệt các nghĩa khác nhau của cùng một từ | SentiWordNet | Điểm theo từng nghĩa | Phải tự chọn nghĩa, chọn sai thì sai |
| Chữ hoa, dấu chấm than, "very", "but", "not" | VADER | Có sẵn các quy tắc này | Mỗi từ chỉ có một điểm, không phân nghĩa |
| Biết câu thiên về ý kiến hay sự việc | TextBlob | Có thêm con số subjectivity | Chỉ ước lượng theo từ điển |
| Đúng hơn trên đánh giá khách sạn | Chưa biết | Chưa có bằng chứng | Phải đo bằng macro-F1 trên cùng phần chấm điểm cuối |

Cả bốn đều chỉ dùng cho tiếng Anh và đều cho một điểm cho cả văn bản, chưa tách từng khía cạnh. Mỗi cách mạnh ở một chỗ, không có cách nào hơn tuyệt đối.

<a id="muc-9"></a>

**9. Mã chạy thử**

Cài gói bằng lệnh `pip install afinn`, rồi chạy:

```python
from afinn import Afinn

analyzer = Afinn(language="en")
texts = [
    "The staff were friendly and the breakfast was delicious.",
    "The room was clean, but the service was terrible.",
    "Good room.",
    "The bed was not good.",
]
for text in texts:
    print(text, analyzer.score(text))
```

Mã này in điểm tổng thật cho các câu, để đối chiếu với từ điển giả định ở mục 5. Kết quả chưa được chạy thử trong tài liệu này. Muốn chia theo số mục khớp hoặc số từ, người dùng tự đếm số mục khớp và số từ rồi chia, và ghi rõ cách đếm.

<a id="muc-10"></a>

**10. Tài liệu tham khảo**

- **[S1]** Nielsen, F. Å. (2011). *A new ANEW: Evaluation of a word list for sentiment analysis in microblogs*. [arXiv:1103.2903](https://arxiv.org/abs/1103.2903).
- **[S2]** Nielsen, F. Å. [AFINN: kho mã Python và hướng dẫn](https://github.com/fnielsen/afinn).
- **[S3]** Nielsen, F. Å. [Giới thiệu tài nguyên năm 2011](https://finnaarupnielsen.wordpress.com/2011/03/16/afinn-a-new-word-list-for-sentiment-analysis/).

Các nguồn trên giới thiệu tài nguyên và gói phần mềm. Phần so sánh với SentiWordNet, VADER, TextBlob dựa trên ba file trước. Ví dụ khách sạn, từ điển nhỏ giả định, các cách chuẩn hóa và kế hoạch kiểm tra ở trên là đề xuất của người biên soạn, chưa phải kết quả của một thí nghiệm thực tế.
