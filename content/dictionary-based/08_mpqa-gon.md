# MPQA Subjectivity Lexicon: giải thích đơn giản

**Mục lục**

- [1. MPQA Subjectivity Lexicon là gì](#muc-1)
- [2. So sánh với các cách trước](#muc-2)
- [3. Ai làm gì](#muc-3)
- [4. Cách tính điểm](#muc-4)
- [5. Ví dụ sáu tình huống](#muc-5)
- [6. Lỗi hay gặp](#muc-6)
- [7. Làm sao biết kết quả đáng tin](#muc-7)
- [8. Chọn cái nào trong chín cái](#muc-8)
- [9. Mã chạy thử](#muc-9)
- [10. Tài liệu tham khảo](#muc-10)

<a id="muc-1"></a>

**1. MPQA Subjectivity Lexicon là gì**

Đây là một danh sách các từ tiếng Anh thường xuất hiện khi người viết nêu ý kiến, cảm xúc hay đánh giá. Danh sách do nhóm Wilson, Wiebe và Hoffmann công bố cùng bài báo năm 2005 [S1]. Người ta gọi mỗi mục là một manh mối chủ quan, vì sự có mặt của từ đó gợi ý rằng câu đang nói về ý kiến chứ không chỉ nói về sự việc.

**Điểm khác biệt cốt lõi: MPQA trả lời hai câu hỏi riêng**

Các cách trước, như AFINN, VADER hay Opinion Lexicon, chỉ hỏi một câu: từ này khen hay chê. Từ nào không khen không chê thì không có trong danh sách, và máy coi như không thấy gì.

MPQA hỏi hai câu, mỗi câu có nhãn riêng.

| Câu hỏi | Nhãn trả lời | Ví dụ giả định |
|---|---|---|
| Câu hỏi 1. Từ này có cho thấy người viết đang nêu ý kiến không, và mạnh hay yếu | type: strongsubj hoặc weaksubj. Nếu từ không có trong danh sách thì không có manh mối | "wonderful" là strongsubj, "think" là weaksubj |
| Câu hỏi 2. Nếu có, từ nghiêng khen hay chê | priorpolarity: positive, negative, neutral hoặc both | "wonderful" là positive, "think" là neutral |

Vì có câu hỏi 1, danh sách chứa cả những từ không khen không chê nhưng báo hiệu người viết đang nói ý kiến, ví dụ "think", "feel". Các từ này mang nhãn neutral. Danh sách chỉ liệt kê từ khen và từ chê thường không chứa chúng.

Hai câu dưới đây cho thấy sự khác nhau. Nhãn là giả định.

| Câu | Danh sách chỉ có khen và chê | MPQA |
|---|---|---|
| The hotel has 120 rooms and a pool. | Không thấy từ nào | Không thấy manh mối nào. Câu có vẻ nêu sự việc |
| I think the room is wonderful. | Chỉ thấy "wonderful", khen. Từ "think" bị bỏ qua | Thấy "think" là weaksubj neutral và "wonderful" là strongsubj positive. Kết luận: câu nêu ý kiến và nghiêng khen |

Với câu thứ nhất, hai cách cho kết quả giống nhau. Với câu thứ hai, cả hai cùng thấy lời khen, nhưng chỉ MPQA ghi nhận thêm rằng đây là câu nêu ý kiến nhờ từ "think".

Nhờ vậy người nghiên cứu có thể làm hai việc theo thứ tự. Việc một là lọc ra câu nêu ý kiến, bỏ câu chỉ mô tả khách sạn. Việc hai là chấm khen chê trên các câu đã lọc.

Khác với các cách trước, mỗi mục không có con số. Mỗi mục là một dòng gồm các nhãn chữ [S2]. Dòng mẫu:

```text
type=weaksubj len=1 word1=abandoned pos1=adj stemmed1=n priorpolarity=negative
```

| Nhãn | Ý nghĩa |
|---|---|
| type | strongsubj là manh mối chủ quan mạnh, weaksubj là manh mối chủ quan yếu |
| len | Số từ của mục. Trong các dòng mẫu đều là 1 |
| word1 | Từ của mục |
| pos1 | Loại từ, ví dụ adj là tính từ, noun là danh từ, verb là động từ, anypos là loại từ nào cũng được |
| stemmed1 | Chữ y nghĩa là mục viết theo dạng gốc và áp dụng cho các dạng biến đổi của từ. Chữ n nghĩa là chỉ áp dụng cho đúng mặt chữ. Cần đọc README để xác nhận |
| priorpolarity | Khen chê vốn có của từ: positive, negative, neutral hoặc both |

Khen chê vốn có ở đây gọi là điểm gốc, tiếng Anh là prior polarity. Đó là khen chê của từ khi đứng riêng, chưa tính câu xung quanh.

Có hai điều dễ nhầm.

1. Nhãn neutral không có nghĩa là từ không đáng chú ý. Một số từ như "feel", "think" là manh mối chủ quan nhưng không nghiêng khen hay chê. Chúng giúp nhận ra câu nêu ý kiến.
2. Nhãn both nghĩa là từ có thể khen hoặc chê tùy cách dùng. Cách xử lý do người nghiên cứu quyết định, nên phải ghi vào bài.

Chính nhóm tác giả đã phân biệt khen chê vốn có của từ với khen chê trong ngữ cảnh, và dành cả một nghiên cứu để đo hai thứ này khác nhau bao nhiêu [S3]. Vì vậy danh sách này nên được coi là điểm xuất phát, không phải kết luận.

<a id="muc-2"></a>

**2. So sánh với các cách trước**

Thông tin về tám cách trước lấy từ các file trước.

| Điểm so sánh | SentiWordNet | VADER | TextBlob | AFINN | NRC | LIWC | SenticNet | MPQA |
|---|---|---|---|---|---|---|---|---|
| Dạng giá trị của mục | Ba điểm theo từng nghĩa | Điểm từ -4 đến +4 | Điểm trong từ điển Pattern | Số nguyên từ -5 đến +5 | Bảng 0 hoặc 1 cho mười nhóm | Nhiều nhóm ngôn ngữ | Thuộc tính của khái niệm | Nhãn chữ, không có số |
| Có mức mạnh yếu | Có, qua điểm | Có | Có | Có | Không | Không phải mục đích chính | Có | Chỉ hai mức, strong và weak |
| Phải chọn nghĩa | Có | Không | Không | Không | Không | Không | Có bước trích khái niệm | Không, nhưng có thể dùng loại từ |
| Tách được câu nêu ý kiến với câu nêu sự việc | Không | Không | Có, qua subjectivity | Không | Không | Không | Không | Có, nhờ nhãn chủ quan |
| Xử lý "not", "very", dấu câu | Người tự viết | Có sẵn | Một số dấu hiệu | Không có sẵn | Không | Không | Phải tự xử lý | Không có sẵn |
| Kết quả cuối | Người tự gộp | compound | polarity và subjectivity | Tổng điểm | Bảng đếm | Bảng tỷ lệ | Tùy cách tổng hợp | Người tự gộp từ các nhãn |

Điểm khác biệt lớn nhất của MPQA: đây là danh sách nhãn chữ, trong đó mỗi mục trả lời hai câu hỏi riêng. Câu hỏi thứ nhất là người viết có đang nêu ý kiến không, mạnh hay yếu. Câu hỏi thứ hai là nếu có thì nghiêng khen hay chê. Mọi cách gộp thành điểm đều do người tự đặt. Đổi lại, người đọc kiểm tra từng mục rất dễ.

Nếu mục tiêu chỉ là phân loại khen chê, MPQA gần với Opinion Lexicon có thêm nhãn mạnh yếu, nên khác biệt không lớn. MPQA có giá trị riêng khi cần tách câu nêu ý kiến khỏi câu nêu sự việc. TextBlob cũng làm việc này bằng con số subjectivity, nhưng đó là trung bình điểm chủ quan của các từ tìm thấy, còn MPQA cho nhãn chữ nên người đọc kiểm tra được từng từ.

Đặt các cách cạnh nhau trên hai câu khác loại. Các nhãn của MPQA là giả định, kết quả của các cách khác theo các file trước.

| Câu | AFINN | VADER | TextBlob | MPQA |
|---|---|---|---|---|
| The hotel has 120 rooms and a pool. | Không khớp từ nào, thiếu dữ liệu | Gần 0, trung tính | polarity gần 0 và subjectivity thấp | Không thấy manh mối. Câu nêu sự việc |
| I think the room is wonderful. | Tra "wonderful" ra điểm khen. Từ "think" không đóng góp | Điểm khen | polarity dương và subjectivity cao | "think" là neutral, "wonderful" là strongsubj positive. Câu nêu ý kiến, nghiêng khen |

Ba cách đầu cho biết câu có nghiêng khen chê hay không. TextBlob và MPQA đều cho biết thêm câu có thiên về ý kiến không. Khác nhau ở chỗ TextBlob đưa ra một con số, còn MPQA đưa ra danh sách các từ đã được nhận ra kèm nhãn.

Đặt các cách cạnh nhau trên cùng câu: "The bed was not good."

| Cách | Điều gì xảy ra |
|---|---|
| SentiWordNet | Tra "good" ra điểm khen. Cần quy tắc "not" do người viết |
| VADER | Tự nhận ra "not", ra điểm chê |
| TextBlob | Có xử lý một số dấu hiệu, cần chạy mã để biết |
| AFINN | Tra "good" ra điểm khen, sai, trừ khi danh sách có sẵn cụm phủ định |
| NRC | Đếm "good" vào nhóm khen. Chữ "not" không làm đổi gì |
| LIWC | Đếm từ theo các ngăn, không chấm khen chê |
| SenticNet | Nếu chỉ tra mà không xử lý "not" thì vẫn sai |
| MPQA | Khớp "good" ra nhãn positive, khen, sai. Chữ "not" không làm đổi gì, trừ khi người tự viết quy tắc |

<a id="muc-3"></a>

**3. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Tải danh sách, ghi tên tệp và phiên bản | Người |
| Chọn chấm từng câu hay cả bài đánh giá | Người |
| Quyết định cách xử lý nhãn both và neutral | Người |
| Đặt trọng số cho strong và weak nếu dùng trọng số | Người |
| Viết quy tắc cho chữ "not" nếu cần | Người viết, máy áp dụng |
| Gán nhãn khen, chê, trung tính cho một tập câu mẫu làm đáp án | Người |
| Tách từ, đưa về dạng gốc, khớp với danh sách | Máy |
| Đếm hoặc cộng các mục khớp | Máy |
| Đổi kết quả thành nhãn bằng ngưỡng | Máy làm, người quyết định ngưỡng |
| So nhãn của máy với đáp án, tính chỉ số | Máy tính, người đọc các câu sai |
| Kết luận trong bài viết | Người |

Nhãn trong danh sách đã có sẵn, người dùng không chấm lại. Theo trang của nhóm tác giả, các manh mối được gom từ nhiều nguồn, chi tiết nằm trong README đi kèm [S2]. Các manh mối chủ quan ban đầu đến từ nghiên cứu của Riloff và Wiebe năm 2003 [S4], còn nhãn khen chê vốn có do nhóm tác giả bổ sung [S1]. Cần đọc README để biết ai gán nhãn và gán thế nào trước khi viết vào bài nghiên cứu.

<a id="muc-4"></a>

**4. Cách tính điểm**

MPQA không có công thức chuẩn. Dưới đây là ba cách người nghiên cứu có thể chọn. Cả ba là đề xuất của người biên soạn.

```text
Cách A, chỉ đếm:
S đếm = P trừ N

Cách B, có trọng số: mỗi mục khớp có dấu (positive là +1, negative là -1) nhân trọng số w
S trọng số = tổng của (dấu nhân w)

Cách C, mật độ chủ quan:
U = M chia T
```

| Ký hiệu | Ý nghĩa |
|---|---|
| P | Số mục khớp có nhãn positive |
| N | Số mục khớp có nhãn negative |
| M | Số mục khớp ở mọi loại, kể cả neutral |
| T | Tổng số từ của văn bản. Phải ghi rõ cách đếm, ví dụ bỏ dấu câu hay không |
| w | Trọng số theo type. Ví dụ giả định, strongsubj là 2 và weaksubj là 1 |
| U | Tỷ lệ từ thuộc manh mối chủ quan trong văn bản |

| Cách | Trả lời câu hỏi nào | Điểm yếu |
|---|---|---|
| A | Có nhiều từ khen hơn từ chê không | Mọi từ nặng như nhau |
| B | Như A nhưng từ mạnh được tính nặng hơn | Trọng số 2 và 1 chỉ là lựa chọn của người, chưa có căn cứ từ danh sách |
| C | Câu nghiêng về nêu ý kiến hay nêu sự việc | Không cho biết khen hay chê |

Khi không khớp được mục nào, ghi là không thấy manh mối, không ghi là 0. Câu không có manh mối chưa chắc là câu khách quan, vì danh sách không chứa mọi từ.

**Quy tắc nhận câu nêu ý kiến**

Để dùng thế mạnh riêng của MPQA, người nghiên cứu cần một quy tắc quyết định câu nào là câu nêu ý kiến. Đây là các lựa chọn gợi ý, đều là đề xuất của người biên soạn.

| Quy tắc | Nội dung | Điểm yếu |
|---|---|---|
| Quy tắc 1 | Câu có ít nhất một mục khớp | Dễ nhận nhầm. Một từ neutral như "think" đứng một mình là bằng chứng yếu |
| Quy tắc 2 | Câu có ít nhất một mục strongsubj, hoặc ít nhất hai mục bất kỳ | Chặt hơn, nhưng bỏ sót câu ý kiến ngắn, ví dụ "Nice room." nếu "nice" chỉ là weaksubj |
| Quy tắc 3 | Câu có U lớn hơn một mức do người chọn | Phải thử mức trên phần thử nghiệm |

Chọn một quy tắc trên phần thử nghiệm, so với nhãn ý kiến hoặc sự việc do người gán, rồi ghi vào bài.

Ngưỡng đổi thành nhãn: danh sách không kèm ngưỡng chuẩn. Ví dụ giả định, S lớn hơn 0 là khen, nhỏ hơn 0 là chê. Nếu S bằng 0 nhưng có mục khớp thì ghi là hỗn hợp. Nếu không có mục khớp thì ghi là thiếu dữ liệu. Ngưỡng nên chọn trên phần thử nghiệm ở mục 7.

<a id="muc-5"></a>

**5. Ví dụ sáu tình huống**

Từ điển nhỏ dưới đây là **giả định** để tính cho dễ, không phải trích từ danh sách thật. Nhãn thật phải chạy mã ở mục 9.

| Từ | type giả định | priorpolarity giả định |
|---|---|---|
| friendly | weaksubj | positive |
| delicious | strongsubj | positive |
| clean | weaksubj | positive |
| terrible | strongsubj | negative |
| good | weaksubj | positive |
| think | weaksubj | neutral |

Trọng số giả định: strongsubj bằng 2, weaksubj bằng 1.

**Tình huống 1: câu chỉ có lời khen.** Câu: "The staff were friendly and the breakfast was delicious."

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Khớp từ với danh sách | friendly, delicious |
| 2 | Đếm | P = 2, N = 0, S đếm = 2 |
| 3 | Tính có trọng số | 1 cộng 2, S trọng số = 3 |
| 4 | So với ngưỡng 0 | Khen |

**Tình huống 2: câu vừa khen vừa chê.** Câu: "The room was clean, but the service was terrible."

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Khớp từ | clean, terrible |
| 2 | Đếm | P = 1, N = 1, S đếm = 0, nhãn hỗn hợp |
| 3 | Tính có trọng số | 1 trừ 2, S trọng số = -1 |
| 4 | So với ngưỡng 0 | Chê |

Hai cách tính cho hai nhãn khác nhau trên cùng một câu. Cách đếm coi hai mục nặng như nhau, cách có trọng số coi từ chê mạnh hơn từ khen yếu. Phải chọn một cách trước khi chạy dữ liệu và ghi vào bài. Cả hai đều không cho biết phòng được khen còn dịch vụ bị chê.

**Tình huống 3: câu nêu ý kiến và câu nêu sự việc.**

Câu A: "The hotel has 120 rooms and a pool." (8 từ)
Câu B: "I think the room is clean." (6 từ)

| Bước | Câu A | Câu B |
|---|---|---|
| Mục khớp | Không có | think, clean |
| Số mục khớp M | 0 | 2 |
| U, mật độ chủ quan | 0 | khoảng 0.33 |
| S đếm | Không thấy manh mối | P = 1, N = 0, S đếm = 1 |

Câu B có mật độ chủ quan cao hơn và nghiêng khen. Từ "think" là neutral nên không đổi S, nhưng giúp nhận ra câu nêu ý kiến. Câu A không có manh mối, nhưng chưa đủ để kết luận câu hoàn toàn khách quan.

**Tình huống 4: có chữ "not".** Câu: "The bed was not good."

| Cách làm | Kết quả | Đúng hay sai |
|---|---|---|
| Chỉ khớp từng từ | Khớp "good" positive, S đếm = 1, khen | Sai |
| Có quy tắc người viết: nếu từ cảm xúc nằm trong ba từ sau "not" thì đổi dấu | S đếm = -1, chê | Đúng |

Quy tắc đổi dấu này vẫn có thể sai ở các câu như "not only clean but also quiet". Nên thử trên các câu mẫu trước khi dùng. Khi lọc từ phổ biến, cần giữ lại "not", "no", "never".

**Tình huống 5: lọc câu nêu ý kiến trước khi chấm khen chê.**

Một đánh giá gồm năm câu. Dùng thêm hai từ giả định: "think" là weaksubj neutral, đã có ở bảng trên. Quy tắc nhận câu nêu ý kiến là quy tắc 1: câu có ít nhất một mục khớp.

| Câu | T | Mục khớp | M | U | Loại câu | S đếm |
|---|---|---|---|---|---|---|
| 1. The hotel has 120 rooms and a pool. | 8 | Không có | 0 | 0 | Sự việc | Không thấy manh mối |
| 2. I think the staff were friendly. | 6 | think, friendly | 2 | khoảng 0.33 | Ý kiến | 1 |
| 3. Check-in starts at 2 pm. | 5 | Không có | 0 | 0 | Sự việc | Không thấy manh mối |
| 4. The breakfast was delicious. | 4 | delicious | 1 | 0.25 | Ý kiến | 1 |
| 5. The service was terrible. | 4 | terrible | 1 | 0.25 | Ý kiến | -1 |

Máy làm các bước sau.

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Nhận câu nêu ý kiến theo quy tắc 1 | Ba câu: câu 2, 4 và 5 |
| 2 | Bỏ hai câu sự việc khi chấm khen chê | Câu 1 và câu 3 |
| 3 | Cộng S đếm của ba câu ý kiến | 1 cộng 1 trừ 1 bằng 1 |
| 4 | Chia cho số câu ý kiến | Trung bình khoảng 0.33, nghiêng khen nhẹ |

Nếu không lọc, và coi hai câu sự việc có S bằng 0 rồi chia cho cả năm câu, trung bình là 0.2. Hai câu chỉ mô tả khách sạn đã kéo kết quả về gần 0 mà không nói gì về cảm xúc của khách. Đây là lợi ích riêng của bước lọc. Đổi lại, kết quả 0.33 hay 0.2 đều chỉ là minh họa, còn ý nghĩa thật phải đo trên dữ liệu thật như mục 7.

**Tình huống 6: câu chỉ có từ neutral.** Câu: "I think we arrived at 3 pm." (7 từ)

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Khớp từ | think |
| 2 | Đếm | P = 0, N = 0, M = 1 |
| 3 | U | khoảng 0.14 |
| 4 | Quy tắc 1 | Câu nêu ý kiến, chưa biết khen hay chê |
| 5 | Quy tắc 2, cần ít nhất một mục strongsubj hoặc hai mục bất kỳ | Không đạt, nên coi là câu nêu sự việc |

Hai quy tắc cho hai kết luận khác nhau. Câu này nêu giờ đến, người nói chỉ dùng "I think" để nói nhẹ đi, không thật sự đánh giá khách sạn. Vì vậy quy tắc 1 dễ nhận nhầm, và nên thử cả hai quy tắc trên phần thử nghiệm. Với Opinion Lexicon, câu này đơn giản là thiếu dữ liệu, không phân biệt được hai trường hợp.

<a id="muc-6"></a>

**6. Lỗi hay gặp**

1. Coi điểm gốc của từ là khen chê của cả câu. Từ vốn khen vẫn có thể nằm trong câu mang ý khác, ví dụ câu phủ định hoặc câu hỏi.
2. Bỏ qua nhãn neutral và both. Hai nhãn này ảnh hưởng đến cách đếm, nên phải quyết định rõ và ghi vào bài.
3. Khớp sai dạng của từ. Mục có stemmed1 bằng y áp dụng cho các dạng biến đổi, mục có stemmed1 bằng n chỉ áp dụng cho đúng mặt chữ. Nếu bỏ qua trường này, máy có thể bỏ sót hoặc khớp thừa.
4. Bỏ qua loại từ. Cùng một mặt chữ có thể có nhiều dòng với loại từ và nhãn khác nhau. Nếu chỉ lấy dòng đầu tiên thì có thể lấy nhầm.
5. Ghi điểm 0 khi không khớp được mục nào. Điểm 0 nghe như trung tính, thực tế là chưa có thông tin.
6. Chọn trọng số theo ý muốn để ra kết quả đẹp. Trọng số phải đặt trước, thử trên phần thử nghiệm.
7. Dùng cho tiếng Việt. Danh sách là tiếng Anh, nên dùng thẳng cho tiếng Việt thì không đáng tin. Nếu dịch rồi chấm thì lỗi dịch cộng thêm vào lỗi chấm.
8. Dùng nhãn của danh sách làm đáp án. Đáp án phải do người gán trên câu thật.
9. Đưa tệp danh sách lên GitHub khi chưa đọc điều kiện sử dụng. Trang của nhóm tác giả ghi danh sách phát hành theo giấy phép GNU GPL và yêu cầu điền biểu mẫu để tải [S2]. Nên chỉ đăng mã tự soạn và hướng dẫn tải, cùng ghi rõ điều kiện.
10. Coi một từ neutral đứng một mình là đủ để kết luận câu nêu ý kiến. Các từ như "think" cũng xuất hiện trong câu nêu sự việc, nơi người viết chỉ nói nhẹ đi. Nên đặt quy tắc chặt hơn và thử trên phần thử nghiệm, như ở tình huống 6.
11. Coi câu không có manh mối là câu khách quan. Câu ý kiến dùng từ ngoài danh sách vẫn bị bỏ sót. Cần đo tỷ lệ bỏ sót trên câu do người gán nhãn.
12. Dùng MPQA như bộ chấm khen chê rồi bỏ qua câu hỏi thứ nhất. Khi đó MPQA chỉ khác Opinion Lexicon ở nhãn mạnh yếu, và chưa dùng đến phần mạnh nhất của danh sách.

<a id="muc-7"></a>

**7. Làm sao biết kết quả đáng tin**

Kiểm tra theo ba bước, giống các file trước.

Bước 1. Người tạo đáp án.

1. Chọn khoảng 100 câu từ nhiều khách sạn khác nhau.
2. Hai người gán nhãn khen, chê hoặc trung tính, mỗi người làm riêng.
3. Máy tính mức hai người đồng ý với nhau, gọi là hệ số kappa. Càng gần 1 thì càng đồng ý.
4. Chỗ hai người khác nhau thì bàn lại và chọn một nhãn.

Nếu mục tiêu có cả việc tách câu nêu ý kiến với câu nêu sự việc, hai người gán thêm nhãn này cho từng câu.

Bước 2. Chia đáp án làm hai phần.

| Phần | Dùng để làm gì |
|---|---|
| Phần thử nghiệm | Chọn cách A, B hoặc C, chọn trọng số, thử xử lý nhãn both, thử quy tắc "not", chọn ngưỡng. Được sửa thoải mái |
| Phần chấm điểm cuối | Chỉ dùng một lần ở cuối, không sửa gì sau khi thấy kết quả |

Chia theo khách sạn, để câu của cùng một khách sạn chỉ nằm ở một phần.

Bước 3. Máy so nhãn với đáp án trên phần chấm điểm cuối, người báo các con số sau.

| Con số | Ý nghĩa đơn giản |
|---|---|
| Độ đúng trung bình của ba nhãn (macro-F1) | Máy đúng bao nhiêu, tính riêng cho khen, chê, trung tính rồi lấy trung bình |
| Bảng nhầm lẫn | Máy hay nhầm nhãn nào thành nhãn nào |
| Tỷ lệ câu khớp được ít nhất một mục (coverage) | Máy hiểu được bao nhiêu phần dữ liệu |
| Kết quả theo nhóm độ dài | Câu ngắn và câu dài có đúng như nhau không |

Với MPQA có thêm ba kiểm tra đáng làm.

1. Chạy cả cách A và cách B, xem kết luận có đổi không.
2. Kiểm tra việc nhận câu nêu ý kiến, tức câu hỏi thứ nhất của MPQA. Hai người gán thêm nhãn ý kiến hoặc sự việc cho từng câu mẫu, riêng với nhãn khen chê. Máy nhận câu ý kiến theo quy tắc ở mục 4, rồi so với nhãn của người. Báo macro-F1 và bảng nhầm lẫn cho riêng việc này. Thử cả ba quy tắc trên phần thử nghiệm, chọn một quy tắc, rồi mới chạy phần chấm điểm cuối.
3. Kiểm tra xem bước lọc có thật sự giúp không. Chạy việc chấm khen chê hai lần trên phần thử nghiệm, một lần trên mọi câu và một lần chỉ trên câu đã được nhận là ý kiến. So macro-F1 của hai lần. Nếu lọc không cải thiện thì phần mạnh nhất của MPQA chưa có tác dụng trên dữ liệu đó, và phải nói rõ trong bài.

Để so các cách cho công bằng, chạy tất cả trên cùng phần chấm điểm cuối rồi so macro-F1. Nếu MPQA không hơn AFINN thì nhãn mạnh yếu chưa giúp được gì trên dữ liệu đó.

Khi viết kết luận, kết quả từ vựng chỉ cho biết chữ viết nghiêng về khen hay chê. Muốn nói khách hài lòng hơn thì cần thêm bằng chứng, ví dụ đối chiếu với số sao nếu có.

<a id="muc-8"></a>

**8. Chọn cái nào trong chín cái**

| Nhu cầu | Cách hợp hơn | Lý do | Điểm yếu |
|---|---|---|---|
| Tách câu nêu ý kiến với câu nêu sự việc theo nhãn chữ, tự quyết cách gộp | MPQA | Có nhãn mạnh yếu và nhãn trung tính cho manh mối chủ quan | Không có điểm số, không xử lý "not" |
| Kiểm tra bằng tay dễ nhất, làm mốc thấp nhất | AFINN hoặc Opinion Lexicon | Chỉ có tra và cộng hoặc đếm | Không có quy tắc về ngữ cảnh |
| Chữ hoa, dấu chấm than, "very", "but", "not" | VADER | Có sẵn các quy tắc này | Mỗi từ chỉ có một điểm |
| Biết câu thiên về ý kiến hay sự việc bằng một con số | TextBlob | Có subjectivity | Chỉ là trung bình điểm chủ quan của các từ tìm thấy |
| Phân biệt các nghĩa khác nhau của cùng một từ | SentiWordNet | Điểm theo từng nghĩa | Phải tự chọn nghĩa |
| Hồ sơ nhiều nhóm cảm xúc | NRC | Mười nhóm cùng lúc | Không có mức mạnh yếu |
| Nhiều nhóm ngôn ngữ, giả thuyết về phong cách | LIWC | Có tài liệu kiểm định | Không phải bộ phân loại khen chê, cần quyền dùng |
| Hiểu cả cụm nhiều từ | SenticNet | Đơn vị tra là khái niệm | Thêm bước trích khái niệm, khó kiểm tra |
| Đúng hơn trên đánh giá khách sạn | Chưa biết | Chưa có bằng chứng | Phải đo macro-F1 trên cùng phần chấm điểm cuối |

Các cách đều chỉ dùng cho tiếng Anh và đều chưa tách từng khía cạnh như phòng, dịch vụ, bữa sáng. Mỗi cách mạnh ở một chỗ, không có cách nào hơn tuyệt đối.

<a id="muc-9"></a>

**9. Mã chạy thử**

Tải danh sách từ trang của nhóm tác giả [S2], giải nén, rồi đặt đường dẫn tệp vào biến `path`. Tên tệp trong gói tải có thể khác theo phiên bản, cần xem lại trong gói.

```python
import re

path = "subjclueslen1-HLTEMNLP05.tff"   # kiểm tra lại tên tệp trong gói tải

clues = []
with open(path, encoding="utf-8") as f:
    for line in f:
        fields = dict(p.split("=", 1) for p in line.split() if "=" in p)
        if "word1" in fields:
            clues.append(fields)

WEIGHT = {"strongsubj": 2, "weaksubj": 1}   # giả định, người đặt
SIGN = {"positive": 1, "negative": -1}

exact = {}
stems = []
for c in clues:
    if c.get("stemmed1") == "y":
        stems.append(c)
    else:
        exact.setdefault(c["word1"], []).append(c)

def find(token):
    found = exact.get(token, [])
    for c in stems:
        if token.startswith(c["word1"]):
            found = found + [c]
    return found[:1]          # lấy mục đầu tiên để không đếm lặp

def score(text):
    tokens = re.findall(r"[a-z']+", text.lower())
    p = n = s = m = 0
    for t in tokens:
        for c in find(t):
            m += 1
            sign = SIGN.get(c["priorpolarity"], 0)
            p += sign > 0
            n += sign < 0
            s += sign * WEIGHT[c["type"]]
    return {"P": p, "N": n, "S_dem": p - n, "S_trong_so": s,
            "so_muc_khop": m, "U": m / len(tokens) if tokens else None,
            "cau_y_kien_quy_tac_1": m >= 1}

texts = [
    "The staff were friendly and the breakfast was delicious.",
    "The room was clean, but the service was terrible.",
    "The hotel has 120 rooms and a pool.",
    "I think the room is clean.",
    "I think we arrived at 3 pm.",
    "The bed was not good.",
]
for text in texts:
    print(text, score(text))
```

Mã này đọc từng dòng nhãn, khớp từ và in kết quả thật để đối chiếu với từ điển giả định ở mục 5, gồm cả câu nêu sự việc và câu chỉ có từ neutral. Dòng cau_y_kien_quy_tac_1 áp dụng quy tắc 1 ở mục 4. Muốn thử quy tắc 2, người dùng tự thêm điều kiện về số mục strongsubj. Mã chưa được chạy thử trong tài liệu này. Mã bỏ qua loại từ, nhãn both và neutral chỉ được tính vào số mục khớp, và chưa xử lý "not". Khớp theo gốc dùng so sánh phần đầu của từ, đây là cách làm gần đúng, cần kiểm tra trên câu thật.

<a id="muc-10"></a>

**10. Tài liệu tham khảo**

- **[S1]** Wilson, T., Wiebe, J., & Hoffmann, P. (2005). *Recognizing Contextual Polarity in Phrase-Level Sentiment Analysis*. Proceedings of HLT-EMNLP 2005.
- **[S2]** Nhóm MPQA, Đại học Pittsburgh. [Trang Subjectivity Lexicon](https://www.mpqa.cs.pitt.edu/lexicons/subj_lexicon/) và [trang các từ điển](https://mpqa.cs.pitt.edu/lexicons/).
- **[S3]** Wilson, T., Wiebe, J., & Hoffmann, P. (2009). *Recognizing Contextual Polarity: An Exploration of Features for Phrase-Level Sentiment Analysis*. Computational Linguistics. [ACL Anthology](https://aclanthology.org/J09-3003.pdf).
- **[S4]** Riloff, E., & Wiebe, J. (2003). *Learning extraction patterns for subjective expressions*. EMNLP 2003.

Các nguồn trên giới thiệu tài nguyên và nghiên cứu của nhóm tác giả. Phần so sánh với tám cách trước dựa trên các file trước. Ví dụ khách sạn, từ điển nhỏ giả định, các cách tính, trọng số và kế hoạch kiểm tra ở trên là đề xuất của người biên soạn, chưa phải kết quả của một thí nghiệm thực tế.
