# SentiStrength: giải thích đơn giản

**Mục lục**

- [1. SentiStrength là gì](#muc-1)
- [2. So sánh với các cách trước](#muc-2)
- [3. Ai làm gì](#muc-3)
- [4. Cách tính điểm](#muc-4)
- [5. Ví dụ sáu tình huống](#muc-5)
- [6. Lỗi hay gặp](#muc-6)
- [7. Làm sao biết kết quả đáng tin](#muc-7)
- [8. Chọn cái nào trong mười cái](#muc-8)
- [9. Mã chạy thử](#muc-9)
- [10. Tài liệu tham khảo](#muc-10)

<a id="muc-1"></a>

**1. SentiStrength là gì**

SentiStrength là một chương trình chấm văn bản ngắn, không trang trọng, như bình luận và bài đăng trên mạng xã hội. Chương trình do nhóm của Mike Thelwall xây dựng, bài giới thiệu đầu tiên công bố năm 2010 [S1].

Điểm khác biệt đầu tiên nằm ở kết quả. Mỗi văn bản nhận hai điểm cùng lúc, không phải một điểm [S1] [S2].

| Điểm | Khoảng giá trị | Ý nghĩa |
|---|---|---|
| Điểm khen, ký hiệu P | Từ 1 đến 5 | 1 là không có khen. 5 là khen rất mạnh |
| Điểm chê, ký hiệu N | Từ -1 đến -5 | -1 là không có chê. -5 là chê rất mạnh |

Lý do thiết kế hai điểm là một người có thể vừa vui vừa không vui trong cùng một đoạn viết. Vì vậy khen và chê không bù trừ nhau.

Có ba điều dễ nhầm.

1. Điểm 1 ở thang khen không phải là khen yếu. Điểm 1 nghĩa là không thấy lời khen nào. Điểm -1 ở thang chê cũng vậy.
2. Văn bản không có từ cảm xúc nào cho kết quả P bằng 1 và N bằng -1, được hiểu là trung tính.
3. Hai điểm không phải là hai xác suất. Chúng là mức mạnh của từ cảm xúc mạnh nhất tìm thấy.

Ngoài kết quả hai điểm, chương trình còn có thể xuất kết quả gộp thành một con số. Một cách là cộng P với N, cho giá trị từ -4 đến 4 [S2] [S3]. Cách khác là đổi thành nhãn ba loại khen, chê, trung tính [S2]. Hai cách này làm mất một phần thông tin, xem mục 6.

Bên trong chương trình có một danh sách từ và gốc từ kèm điểm mạnh yếu. Gốc từ viết kèm dấu sao, ví dụ amaz* khớp với amazed và amazing [S2]. Chương trình còn có danh sách biểu tượng cảm xúc kèm điểm, danh sách thành ngữ có điểm riêng, và các quy tắc về từ tăng giảm mức độ, phủ định, chữ kéo dài [S1] [S2].

Chương trình chạy bằng Java, có thể chấm nhiều văn bản mỗi lần, và có tùy chọn explain để in ra lý do cho điểm [S5]. Tùy chọn này giúp người nghiên cứu kiểm tra từng câu bằng tay.

<a id="muc-2"></a>

**2. So sánh với các cách trước**

Thông tin về chín cách trước lấy từ các file trước.

| Điểm so sánh | VADER | AFINN | Opinion Lexicon | MPQA | SentiStrength |
|---|---|---|---|---|---|
| Kết quả cho văn bản | Điểm compound, kèm pos, neu, neg là tỷ lệ | Tổng điểm | Số từ khen và số từ chê | Nhãn, người tự gộp | Hai điểm cùng lúc, P và N |
| Cách gộp từ trong câu | Cộng rồi chuẩn hóa | Cộng | Đếm | Người tự đặt | Lấy từ mạnh nhất, không cộng |
| Khen và chê có bù trừ nhau không | Có trong compound | Có | Có nếu dùng hiệu số | Tùy cách gộp | Không, giữ riêng hai điểm |
| Điểm tổng phụ thuộc độ dài | Ít | Nhiều | Nhiều nếu dùng hiệu số | Tùy cách gộp | Ít, vì lấy từ mạnh nhất |
| Có mức mạnh yếu của từ | Có | Có | Không | Hai mức | Có, năm mức mỗi chiều |
| Xử lý "not", "very" | Có sẵn | Không có sẵn | Không có sẵn | Không có sẵn | Có sẵn |
| Xử lý biểu tượng cảm xúc, chữ kéo dài | Có xử lý một số dấu hiệu viết | Không | Không | Không | Có, nằm trong thiết kế |
| Xem được lý do cho điểm | Xem điểm từng từ | Tự cộng bằng tay | Tự đếm bằng tay | Xem nhãn từng mục | Có tùy chọn explain |
| Thiết kế cho loại văn bản nào | Văn bản mạng xã hội | Tin nhắn ngắn trên mạng | Đánh giá sản phẩm | Nghiên cứu gốc dùng văn bản tin tức | Văn bản ngắn, không trang trọng |

Điểm khác biệt lớn nhất của SentiStrength: giữ khen và chê thành hai điểm riêng, và mỗi điểm lấy từ mạnh nhất chứ không cộng dồn. Hai việc này làm điểm ít phụ thuộc vào độ dài văn bản, nhưng cũng làm mất thông tin về số lượng từ.

NRC, LIWC và SenticNet có mục đích khác với SentiStrength. NRC đếm mười nhóm cảm xúc, LIWC đếm nhiều nhóm ngôn ngữ, SenticNet tra khái niệm nhiều từ. Vì vậy chúng không đứng cạnh SentiStrength trong cùng một bảng đặc điểm, nhưng vẫn đứng cạnh trong bảng câu mẫu dưới đây.

Đặt các cách cạnh nhau trên cùng câu: "The room was clean, but the service was terrible."

| Cách | Điều gì xảy ra |
|---|---|
| SentiWordNet | Cộng đơn giản thì khen và chê bù nhau, ra gần 0 |
| VADER | Chữ "but" làm vế sau nặng hơn, nên điểm nghiêng về chê |
| TextBlob | Lấy trung bình hai ý, polarity gần 0, che mất ý khen và ý chê |
| AFINN | Cộng điểm hai từ, ra tổng âm. Không cho biết điểm nào đến từ phòng, điểm nào từ dịch vụ |
| NRC | Đếm từng từ vào các nhóm có trong từ điển. Không có mức mạnh yếu |
| LIWC | Đếm từ theo các ngăn, không chấm khen chê |
| SenticNet | Tùy các khái niệm trích được từ câu |
| MPQA | Đếm thì hỗn hợp, tính trọng số thì nghiêng chê |
| Opinion Lexicon | P bằng 1 và N bằng 1, hỗn hợp, không phân biệt mức chê |
| SentiStrength | Hai điểm: P khoảng 2 và N khoảng -4 (giả định). Khen và chê đều còn nguyên, và mức chê mạnh hơn mức khen |

Cả mười cách đều chưa nói được khen cái gì, chê cái gì. SentiStrength chỉ cho biết văn bản vừa có khen vừa có chê, chưa gắn mỗi điểm với phòng hay dịch vụ. Muốn gắn phải chấm riêng từng khía cạnh.

<a id="muc-3"></a>

**3. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Tải chương trình và tệp dữ liệu, ghi tên và phiên bản | Người |
| Chọn chấm từng câu hay cả bài đánh giá | Người |
| Chọn kiểu kết quả: hai điểm, một điểm cộng, hay nhãn ba loại | Người |
| Chọn chạy không giám sát hay có giám sát | Người |
| Gán nhãn hoặc điểm khen chê cho một tập câu mẫu làm đáp án | Người |
| Tách câu, tách từ, nhận biểu tượng cảm xúc và dấu câu | Máy |
| Khớp từ và gốc từ với danh sách, tra điểm | Máy |
| Áp quy tắc về từ tăng giảm, phủ định, chữ kéo dài, thành ngữ | Máy |
| Lấy từ mạnh nhất cho điểm khen và điểm chê | Máy |
| Đổi sang một điểm hoặc nhãn ba loại | Máy làm, người chọn cách |
| So kết quả của máy với đáp án, tính chỉ số | Máy tính, người đọc các câu sai |
| Kết luận trong bài viết | Người |

Điểm mạnh yếu của từng từ trong danh sách đã có sẵn, người dùng không chấm lại. Theo các tài liệu của tác giả, danh sách được xây dựng từ nhiều nguồn từ vựng, rồi điều chỉnh bằng các văn bản do người gán nhãn [S1] [S2]. Cần đọc các bài này để biết chính xác ai gán điểm cho từng từ trước khi viết vào bài nghiên cứu.

Chương trình có hai chế độ [S2]. Ở chế độ không giám sát, chương trình dùng nguyên các điểm có sẵn. Ở chế độ có giám sát, người đưa vào một tập văn bản đã gán nhãn để chương trình điều chỉnh điểm của từ cho phù hợp với đề tài. Chế độ có giám sát nhờ vậy phụ thuộc vào đáp án do người làm, nên cần cẩn thận khi chia dữ liệu, xem mục 7.

<a id="muc-4"></a>

**4. Cách tính điểm**

Máy làm các bước sau. Các bước được viết rút gọn từ các tài liệu của tác giả [S1] [S2]. Chi tiết chính xác từng quy tắc cần xem bằng tùy chọn explain, vì có thể thay đổi theo phiên bản.

| Bước | Máy làm |
|---|---|
| 1 | Tách văn bản thành câu, rồi thành từ. Tách riêng biểu tượng cảm xúc và dấu câu |
| 2 | Khớp mỗi từ với danh sách. Từ khen có điểm từ 2 đến 5, từ chê có điểm từ -2 đến -5 |
| 3 | Chỉnh điểm của từ theo quy tắc: từ tăng mức độ đứng trước làm điểm mạnh hơn, từ giảm mức độ làm yếu đi, phủ định làm đổi chiều, chữ kéo dài làm mạnh hơn |
| 4 | Biểu tượng cảm xúc và thành ngữ có điểm riêng, thành ngữ thay cho điểm của các từ bên trong |
| 5 | Trong mỗi câu, lấy từ khen mạnh nhất làm điểm khen của câu, và từ chê mạnh nhất làm điểm chê của câu |
| 6 | Trong cả văn bản, lấy điểm khen lớn nhất và điểm chê mạnh nhất của các câu |

Viết thành công thức dạng văn bản:

```text
P câu = điểm khen mạnh nhất trong câu. Nếu không có thì bằng 1
N câu = điểm chê mạnh nhất trong câu. Nếu không có thì bằng -1
P = giá trị lớn nhất của P câu qua các câu
N = giá trị nhỏ nhất của N câu qua các câu
Điểm cộng = P cộng N
```

| Ký hiệu | Ý nghĩa |
|---|---|
| P câu, N câu | Điểm khen và điểm chê của một câu |
| P, N | Điểm khen và điểm chê của cả văn bản |
| Điểm cộng | Một con số gộp, nằm từ -4 đến 4, vì P từ 1 đến 5 và N từ -5 đến -1 [S2] |

Có hai điểm cần chú ý.

Thứ nhất, các từ không được cộng với nhau mà chỉ có từ mạnh nhất được giữ. Vì vậy một câu có mười từ khen nhẹ và một câu có một từ khen nhẹ cho cùng điểm khen. Điều này làm điểm ít phụ thuộc độ dài, nhưng cũng làm mất thông tin về số lượng.

Thứ hai, nhãn ba loại lấy từ so sánh P với độ lớn của N. Theo ví dụ trong một bài nghiên cứu, khi P lớn hơn độ lớn của N thì kết quả là khen, và khi P bằng 1 và N bằng -1 thì kết quả là trung tính [S7]. Cách xử lý khi hai bên bằng nhau nhưng lớn hơn 1 cần đọc hướng dẫn của phiên bản đang dùng.

Ngưỡng và cách gộp: chương trình có sẵn các kiểu kết quả, nên người nghiên cứu không phải tự đặt ngưỡng như với nhiều cách trước. Tuy vậy vẫn cần chọn một kiểu trước khi chạy dữ liệu, và ghi rõ vào bài.

<a id="muc-5"></a>

**5. Ví dụ sáu tình huống**

Từ điển nhỏ và các quy tắc dưới đây là **giả định** để tính cho dễ, không phải trích từ chương trình thật. Điểm thật phải chạy ở mục 9.

| Từ | Điểm giả định |
|---|---|
| friendly | 2 |
| clean | 2 |
| good | 2 |
| delicious | 3 |
| slow | -2 |
| terrible | -4 |

Quy tắc giả định: từ "very" đứng trước làm độ lớn tăng thêm 1. Phủ định "not" đổi dấu và giữ nguyên độ lớn. Chữ kéo dài làm độ lớn tăng thêm 1.

**Tình huống 1: câu chỉ có lời khen.** Câu: "The staff were friendly and the breakfast was delicious."

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Khớp từ với danh sách | friendly, delicious |
| 2 | Lấy điểm | 2 và 3 |
| 3 | Chọn khen mạnh nhất | P = 3 |
| 4 | Không có từ chê | N = -1 |
| 5 | Kết quả hai điểm | P = 3, N = -1 |

So với AFINN, tổng sẽ là 2 cộng 3 bằng 5. SentiStrength chỉ giữ từ mạnh nhất, nên cho 3.

**Tình huống 2: câu vừa khen vừa chê.** Câu: "The room was clean, but the service was terrible."

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Khớp từ | clean, terrible |
| 2 | Lấy điểm | 2 và -4 |
| 3 | Điểm khen | P = 2 |
| 4 | Điểm chê | N = -4 |
| 5 | Điểm cộng | 2 cộng -4 bằng -2 |

Hai điểm P bằng 2 và N bằng -4 giữ nguyên cả ý khen lẫn ý chê và cho biết ý chê mạnh hơn. Chỉ riêng điểm cộng bằng -2 thì không thấy điều này. Đây là lợi thế chính của hai điểm so với một điểm.

**Tình huống 3: từ tăng mức độ và chữ kéo dài.**

Câu A: "The service was slow."
Câu B: "The service was very slow."
Câu C: "The breakfast was deliiicious."
Câu D: "The breakfast was delicious."

| Câu | Từ khớp | Điều chỉnh giả định | Hai điểm |
|---|---|---|---|
| A | slow, -2 | Không có | P = 1, N = -2 |
| B | slow, -2 | very làm tăng thêm 1, thành -3 | P = 1, N = -3 |
| C | delicious, 3 | Chữ kéo dài tăng thêm 1, thành 4 | P = 4, N = -1 |
| D | delicious, 3 | Không có | P = 3, N = -1 |

Nếu xóa chữ kéo dài hoặc bỏ từ "very" trước khi chạy, hai cặp câu sẽ cho cùng điểm và mất thông tin về mức nhấn mạnh. Văn bản gốc nên được giữ nguyên.

**Tình huống 4: có chữ "not".** Câu: "The bed was not good."

| Cách làm | Kết quả | Đúng hay sai |
|---|---|---|
| Chỉ khớp từng từ, bỏ qua "not" | Khớp "good", P = 2, N = -1, khen | Sai |
| Áp quy tắc phủ định giả định | "good" bị đổi dấu thành -2, P = 1, N = -2, chê | Đúng |

Cách SentiStrength xử lý phủ định có thể khác với giả định ở đây. Một số trường hợp phủ định chỉ làm yếu đi, không đổi hẳn dấu. Cần xem bằng tùy chọn explain trên câu thật.

**Tình huống 5: hai đánh giá cùng khen nhưng khác độ dài.**

Đánh giá A: "Good room." (2 từ)
Đánh giá B: "The room was good and the bed was good and the breakfast was good." (14 từ, ba lần "good")

| Cách | Đánh giá A | Đánh giá B |
|---|---|---|
| Hai điểm của SentiStrength | P = 2, N = -1 | P = 2, N = -1 |
| Tổng của AFINN, theo file trước | 3 | 9 |

SentiStrength cho hai đánh giá cùng kết quả, vì chỉ lấy từ mạnh nhất. Điểm không bị kéo theo độ dài. Đổi lại, đánh giá B khen ba lần mà điểm không thể hiện điều đó.

**Tình huống 6: một đoạn nhiều câu.** Đoạn: "The staff were friendly. The room was clean. The service was terrible."

| Câu | P câu | N câu |
|---|---|---|
| The staff were friendly. | 2 | -1 |
| The room was clean. | 2 | -1 |
| The service was terrible. | 1 | -4 |
| Cả đoạn | P = 2, lớn nhất | N = -4, nhỏ nhất |

Đoạn có hai câu khen và một câu chê, nhưng điểm chê vẫn là -4, mạnh hơn nhiều so với điểm khen bằng 2. Một đoạn có mười câu khen nhẹ và một câu chê nặng cho cùng kết quả. Vì vậy SentiStrength nhạy với từ mạnh nhất hơn với số lượng, và cần đặt điều này vào câu hỏi nghiên cứu.

<a id="muc-6"></a>

**6. Lỗi hay gặp**

1. Coi hai điểm là kết quả loại trừ nhau. Một văn bản có thể có P bằng 4 và N bằng -4 cùng lúc.
2. Hiểu điểm 1 hoặc -1 là điểm yếu. Điểm 1 ở thang khen nghĩa là không có khen. Cần ghi rõ trong bài.
3. Chỉ dùng điểm cộng. Hai văn bản P = 4, N = -4 và P = 1, N = -1 đều có điểm cộng bằng 0, nhưng một văn bản có cảm xúc mạnh cả hai chiều, còn văn bản kia gần như không có cảm xúc. Nên báo hai điểm, hoặc ít nhất ghi lý do chọn điểm cộng.
4. Coi điểm cao là nhiều lời khen. Điểm chỉ phản ánh từ mạnh nhất, không phản ánh số lượng.
5. Xóa biểu tượng cảm xúc, dấu câu, chữ kéo dài hoặc đổi sang chữ thường trước khi chạy. Chương trình dùng các dấu hiệu này, nên xóa đi là mất thông tin. Giữ nguyên văn bản gốc.
6. Bỏ chữ "not" khi lọc các từ phổ biến. Cần giữ lại "not", "no", "never".
7. Dùng cho tiếng Việt. Trang chính thức liệt kê các ngôn ngữ đã có bản riêng, trong đó chưa thấy tiếng Việt [S4]. Tài liệu của tác giả nói chương trình có thể được chuyển sang ngôn ngữ mới bằng cách dịch danh sách từ và các tài nguyên khác, nhưng việc đó phải được kiểm chứng riêng. Nếu dịch văn bản sang tiếng Anh rồi chấm thì lỗi dịch cộng thêm vào lỗi chấm.
8. Không ghi phiên bản. Có bản Windows và bản Java, các bản có khả năng khác nhau, trong đó có bản không làm các kiểu kết quả nhị phân, ba loại, cần xem kỹ trang tải [S4]. Cần ghi tên bản, ngày tải, tệp dữ liệu và kiểu kết quả đã chọn.
9. Dùng chế độ có giám sát trên cả phần chấm điểm cuối. Chế độ này điều chỉnh điểm theo đáp án, nên nếu điều chỉnh trên phần chấm điểm cuối thì kết quả đẹp hơn thực tế.
10. Dùng kết quả của chương trình làm đáp án. Đáp án phải do người gán.
11. Đăng tệp chương trình hoặc dữ liệu lên GitHub khi chưa đọc điều kiện. Trang chính thức ghi chương trình miễn phí cho nghiên cứu học thuật và cần giấy phép thương mại cho mục đích thương mại [S4]. Nên chỉ đăng mã tự soạn, kèm đường dẫn tải và điều kiện sử dụng.

<a id="muc-7"></a>

**7. Làm sao biết kết quả đáng tin**

Kiểm tra theo ba bước, giống các file trước.

Bước 1. Người tạo đáp án.

1. Chọn khoảng 100 câu từ nhiều khách sạn khác nhau.
2. Hai người gán nhãn khen, chê hoặc trung tính, mỗi người làm riêng.
3. Máy tính mức hai người đồng ý với nhau, gọi là hệ số kappa. Càng gần 1 thì càng đồng ý.
4. Chỗ hai người khác nhau thì bàn lại và chọn một nhãn.

Vì SentiStrength cho hai điểm, nên nên để người gán thêm hai điểm cho mỗi câu, một điểm khen từ 1 đến 5 và một điểm chê từ -1 đến -5, dùng chính hai thang của chương trình. Sau đó cho hai người đối chiếu điểm, tính chênh lệch trung bình. Nhãn ba loại có thể suy ra từ hai điểm này.

Bước 2. Chia đáp án làm hai phần.

| Phần | Dùng để làm gì |
|---|---|
| Phần thử nghiệm | Chọn kiểu kết quả, thử quy tắc, nếu dùng chế độ có giám sát thì điều chỉnh điểm của từ ở phần này. Được sửa thoải mái |
| Phần chấm điểm cuối | Chỉ dùng một lần ở cuối, không sửa gì sau khi thấy kết quả. Không dùng để điều chỉnh điểm |

Chia theo khách sạn, để câu của cùng một khách sạn chỉ nằm ở một phần.

Bước 3. Máy so kết quả với đáp án trên phần chấm điểm cuối, người báo các con số sau.

| Con số | Ý nghĩa đơn giản |
|---|---|
| Độ đúng trung bình của ba nhãn (macro-F1) | Máy đúng bao nhiêu, tính riêng cho khen, chê, trung tính rồi lấy trung bình |
| Bảng nhầm lẫn | Máy hay nhầm nhãn nào thành nhãn nào |
| Độ lệch trung bình của điểm khen và của điểm chê so với người | Mức chênh lệch giữa điểm của máy và điểm của người trên từng thang |
| Tỷ lệ câu có ít nhất một từ cảm xúc (coverage) | Máy hiểu được bao nhiêu phần dữ liệu, tức P lớn hơn 1 hoặc N nhỏ hơn -1 |
| Kết quả theo nhóm độ dài | Câu ngắn và câu dài có đúng như nhau không |

Với SentiStrength có thêm ba kiểm tra đáng làm.

1. Chạy ba kiểu kết quả là hai điểm, điểm cộng và nhãn ba loại, xem kết luận có đổi không. Nếu đổi mạnh thì phải báo.
2. Đọc tùy chọn explain của khoảng 50 câu sai, xem lỗi nằm ở từ nào, quy tắc nào.
3. Nếu dùng chế độ có giám sát, so kết quả có điều chỉnh và không điều chỉnh trên cùng phần chấm điểm cuối. Nếu điều chỉnh không cải thiện thì chưa cần dùng.

Để so các cách cho công bằng, chạy tất cả trên cùng phần chấm điểm cuối rồi so macro-F1. Nếu VADER hoặc AFINN không kém SentiStrength thì các quy tắc về chữ kéo dài và biểu tượng cảm xúc có thể không cần với dữ liệu đánh giá khách sạn, vốn thường viết nghiêm túc hơn bình luận mạng xã hội.

Khi viết kết luận, kết quả từ vựng chỉ cho biết chữ viết nghiêng về khen hay chê. Muốn nói khách hài lòng hơn thì cần thêm bằng chứng, ví dụ đối chiếu với số sao nếu có.

<a id="muc-8"></a>

**8. Chọn cái nào trong mười cái**

| Nhu cầu | Cách hợp hơn | Lý do | Điểm yếu |
|---|---|---|---|
| Biết văn bản vừa có khen vừa có chê và mức mạnh của từng chiều | SentiStrength | Hai điểm riêng, không bù trừ | Chỉ lấy từ mạnh nhất, mất thông tin về số lượng |
| Văn bản ngắn có biểu tượng cảm xúc, chữ kéo dài | SentiStrength | Có quy tắc riêng cho các dấu hiệu này | Đánh giá khách sạn có thể ít các dấu hiệu này |
| Xem lý do vì sao ra điểm | SentiStrength | Tùy chọn explain | Phải chạy bản Java hoặc dùng thư viện bọc |
| Kiểm tra bằng tay dễ nhất, làm mốc thấp nhất | AFINN hoặc Opinion Lexicon | Chỉ tra và cộng hoặc đếm | Không có quy tắc về ngữ cảnh |
| Chữ hoa, dấu chấm than, "but", "not" | VADER | Có sẵn các quy tắc này | Ra một điểm compound |
| Tách câu nêu ý kiến với câu nêu sự việc | MPQA | Có nhãn chủ quan | Không có điểm số, không xử lý "not" |
| Biết câu thiên về ý kiến bằng một con số | TextBlob | Có subjectivity | Chỉ là trung bình điểm chủ quan |
| Phân biệt các nghĩa khác nhau của cùng một từ | SentiWordNet | Điểm theo từng nghĩa | Phải tự chọn nghĩa |
| Hồ sơ nhiều nhóm cảm xúc | NRC | Mười nhóm cùng lúc | Không có mức mạnh yếu |
| Nhiều nhóm ngôn ngữ, giả thuyết về phong cách | LIWC | Có tài liệu kiểm định | Không phải bộ phân loại khen chê, cần quyền dùng |
| Hiểu cả cụm nhiều từ | SenticNet | Đơn vị tra là khái niệm | Thêm bước trích khái niệm, khó kiểm tra |
| Đúng hơn trên đánh giá khách sạn | Chưa biết | Chưa có bằng chứng | Phải đo macro-F1 trên cùng phần chấm điểm cuối |

Các cách đều chủ yếu dùng cho tiếng Anh và đều chưa tách từng khía cạnh như phòng, dịch vụ, bữa sáng. Mỗi cách mạnh ở một chỗ, không có cách nào hơn tuyệt đối.

<a id="muc-9"></a>

**9. Mã chạy thử**

Có hai cách chạy thử. Cả hai đều cần cài Java.

Cách một là chạy chương trình gốc. Tải chương trình và thư mục dữ liệu từ trang chính thức [S4], rồi chạy theo hướng dẫn Java [S5]. Trong hướng dẫn, khoảng trắng của câu được thay bằng dấu cộng.

```text
java -jar SentiStrength.jar sentidata D:/senti/SentiStrength_Data/ text the+room+was+clean+but+the+service+was+terrible explain
java -jar SentiStrength.jar sentidata D:/senti/SentiStrength_Data/ input danhgia.txt
```

Dòng đầu chấm một câu và in lý do nhờ tùy chọn explain. Dòng sau chấm từng dòng của tệp danhgia.txt và tạo tệp mới có thêm kết quả ở cuối mỗi dòng [S5]. Đường dẫn cần đổi theo máy.

Cách hai là dùng thư viện bọc cho Python, có sẵn tệp chương trình bên trong [S6]. Cài bằng lệnh `pip install pysenti`, rồi chạy:

```python
import pysenti

texts = [
    "The staff were friendly and the breakfast was delicious.",
    "The room was clean, but the service was terrible.",
    "The service was very slow.",
    "The breakfast was deliiicious.",
    "The bed was not good.",
    "Good room.",
]
for text in texts:
    result = pysenti.get_senti(text)
    print(text, result, result.scale())
```

Mã này in kết quả thật để đối chiếu với từ điển giả định ở mục 5. Theo trang mô tả thư viện, kết quả gồm hai điểm khen và chê, và phương thức scale cho điểm cộng gộp [S6]. Mã chưa được chạy thử trong tài liệu này. Thư viện này bọc cùng chương trình gốc, nên cùng điều kiện giấy phép: dùng cho mục đích thương mại cần xin phép tác giả [S4] [S6].

<a id="muc-10"></a>

**10. Tài liệu tham khảo**

- **[S1]** Thelwall, M., Buckley, K., Paltoglou, G., Cai, D., & Kappas, A. (2010). *Sentiment strength detection in short informal text*. Journal of the American Society for Information Science and Technology, 61(12), 2544-2558. [Bản thảo](https://mi-linux.wlv.ac.uk/~cm1993/papers/SentiStrengthPreprint.doc).
- **[S2]** Thelwall, M. *The Heart and Soul of the Web? Sentiment Strength Detection in the Social Web with SentiStrength*. [Chương sách](https://mi-linux.wlv.ac.uk/~cm1993/sentistrength/documentation/SentiStrengthChapter.pdf).
- **[S3]** Thelwall, M., & Buckley, K. *Topic-Based Sentiment Analysis for the Social Web: The Role of Mood and Issue-Related Words*. [Bản thảo](https://mi-linux.wlv.ac.uk/~cm1993/papers/SentiStrength3Preprintx.pdf).
- **[S4]** Thelwall, M. [Trang chính thức của SentiStrength](https://mi-linux.wlv.ac.uk/~cm1993/sentistrength), gồm tải về, ngôn ngữ và giấy phép.
- **[S5]** Thelwall, M. [Hướng dẫn sử dụng bản Java](https://mi-linux.wlv.ac.uk/~cm1993/sentistrength/documentation/SentiStrengthJavaManual.doc).
- **[S6]** [Thư viện PySentiStrength trên PyPI](https://pypi.org/project/PySentiStrength) và [kho PySenti](https://github.com/hykilpikonna/PySenti).
- **[S7]** *Sentiment Polarity Detection for Software Development*. [arXiv:1709.02984](https://arxiv.org/abs/1709.02984). Dùng làm ví dụ về cách chuyển hai điểm thành nhãn ba loại.

Các nguồn trên giới thiệu chương trình và gói phần mềm. Phần so sánh với chín cách trước dựa trên các file trước. Ví dụ khách sạn, từ điển nhỏ giả định, các quy tắc giả định về từ tăng mức độ, phủ định và chữ kéo dài, và kế hoạch kiểm tra ở trên là đề xuất của người biên soạn, chưa phải kết quả của một thí nghiệm thực tế.
