# NRC Emotion Lexicon (EmoLex): giải thích đơn giản

**Mục lục**

- [1. NRC EmoLex là gì](#muc-1)
- [2. So sánh với bốn cách trước](#muc-2)
- [3. Ai làm gì](#muc-3)
- [4. Cách đếm và cách tính tỷ lệ](#muc-4)
- [5. Ví dụ bốn tình huống](#muc-5)
- [6. Lỗi hay gặp](#muc-6)
- [7. Làm sao biết kết quả đáng tin](#muc-7)
- [8. Chọn cái nào trong năm cái](#muc-8)
- [9. Mã chạy thử](#muc-9)
- [10. Tài liệu tham khảo](#muc-10)

<a id="muc-1"></a>

**1. NRC EmoLex là gì**

File này chỉ nói về NRC Word-Emotion Association Lexicon, còn gọi là EmoLex. Các tài nguyên khác mang tên NRC, như NRC VAD hay NRC Emotion Intensity, là tài nguyên khác và không gộp vào đây [S1].

EmoLex là một bảng cho biết mỗi từ tiếng Anh gợi đến những nhóm cảm xúc nào. Có mười nhóm: tám nhóm cảm xúc và hai nhóm khen chê.

| Nhóm | Nghĩa |
|---|---|
| anger | giận |
| anticipation | mong đợi |
| disgust | ghê tởm |
| fear | sợ |
| joy | vui |
| sadness | buồn |
| surprise | ngạc nhiên |
| trust | tin tưởng |
| positive | khen |
| negative | chê |

Mỗi ô trong bảng chỉ có hai giá trị: 1 nếu từ đó gợi đến nhóm ấy, 0 nếu không. Một từ có thể gợi đến nhiều nhóm cùng lúc. Ví dụ giả định: "happy" gợi joy và positive [S1].

Cách chấm một văn bản rất đơn giản: với mỗi nhóm, đếm xem có bao nhiêu từ của văn bản gợi đến nhóm đó. Kết quả là một bảng đếm cho mười nhóm, không phải một điểm.

Cần nhớ rõ hai điều.

1. Đây là từ gợi đến cảm xúc, không phải người viết đang cảm thấy cảm xúc đó. Câu "I was not afraid" vẫn có từ "afraid" gợi fear.
2. Các con số đếm không phải xác suất và không cần cộng thành 100%.

<a id="muc-2"></a>

**2. So sánh với bốn cách trước**

Thông tin về bốn cách trước lấy từ các file trước.

| Điểm so sánh | SentiWordNet, VADER, TextBlob, AFINN | NRC EmoLex |
|---|---|---|
| Cho biết gì | Mức khen chê, một chiều | Mười nhóm cùng lúc: tám cảm xúc cộng khen và chê |
| Dạng giá trị của từ | Con số có mức mạnh yếu | Chỉ có 1 hoặc 0, không có mức mạnh yếu |
| Đầu ra cho văn bản | Một điểm (riêng TextBlob có thêm subjectivity) | Bảng đếm mười nhóm |
| Xử lý "not", "very", dấu câu | Tùy cách, VADER có sẵn nhiều nhất | Không có |
| Một từ thuộc mấy nhóm | Một điểm hoặc một thang | Có thể thuộc nhiều nhóm |
| Ai gán nhãn trong từ điển | Khác nhau tùy cách | Theo bài gốc, nhiều người gán qua nền tảng cộng đồng [S2] |
| Tiếng Việt | Không dùng trực tiếp | Có bản dịch tự động, cần người kiểm tra lại [S1] |

Điểm khác biệt lớn nhất: bốn cách trước trả lời "khen hay chê, mạnh cỡ nào", còn NRC trả lời "văn bản dùng những từ gợi đến cảm xúc nào". Vì vậy NRC cho hồ sơ nhiều chiều nhưng không cho cường độ.

Đặt các cách cạnh nhau trên cùng một câu: "We were not happy during our stay."

| Cách | Điều gì xảy ra |
|---|---|
| SentiWordNet | Tra "happy" ra điểm khen. Cần quy tắc "not" do người viết |
| VADER | Tự nhận ra "not", ra điểm chê |
| TextBlob | Có xử lý một số dấu hiệu, cần chạy mã để biết |
| AFINN | Tra "happy" ra điểm khen, trừ khi danh sách có sẵn cụm phủ định |
| NRC EmoLex | Đếm "happy" vào joy và positive. Chữ "not" không làm đổi gì cả |

NRC là cách yếu nhất về chữ "not", vì nó chỉ đếm từ.

<a id="muc-3"></a>

**3. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Quyết định xem nhóm cảm xúc nào (ví dụ chỉ fear và trust) | Người |
| Chọn mẫu số: chia cho tổng số từ hay chia cho số từ khớp | Người |
| Tách từ, đặc biệt với tiếng Việt có từ nhiều âm tiết | Người chọn cách, máy làm |
| Kiểm tra bản dịch tiếng Việt, sửa các mục sai | Người |
| Viết quy tắc cho chữ "not" nếu cần | Người viết, máy áp dụng |
| Gán nhãn đáp án cho một tập câu mẫu nếu muốn kiểm tra cảm xúc của câu | Người |
| Khớp từ và tra nhãn trong từ điển | Máy |
| Đếm và tính tỷ lệ cho từng nhóm | Máy |
| So kết quả với đáp án, tính chỉ số | Máy tính, người đọc các câu sai |
| Kết luận trong bài viết | Người |

Nhãn của từng từ trong từ điển đã có sẵn do người gán từ trước, người dùng không gán lại. Việc người dùng cần lo là chọn cách đếm và kiểm tra xem cách đếm có hợp với câu hỏi nghiên cứu không.

<a id="muc-4"></a>

**4. Cách đếm và cách tính tỷ lệ**

Với mỗi nhóm cảm xúc e, máy làm hai việc:

```text
C = số từ của văn bản có nhãn e bằng 1
R = C chia cho N
```

| Ký hiệu | Ý nghĩa |
|---|---|
| e | Một nhóm trong mười nhóm, ví dụ joy |
| C | Số từ của văn bản gợi đến nhóm e |
| N | Tổng số từ của văn bản. Phải ghi rõ cách đếm. Nếu N bằng 0 thì không tính |
| R | Tỷ lệ từ gợi đến nhóm e |

Ví dụ cách đọc: joy bằng 4 trên 100 từ nghĩa là có bốn vị trí khớp với nhóm joy. Không có nghĩa người viết vui 4%, cũng không có nghĩa 4% khách vui.

Hai điểm khác với các cách trước.

1. Một từ có thể thuộc nhiều nhóm, nên tổng các tỷ lệ của mười nhóm có thể lớn hơn 1. Điều này bình thường.
2. Cần phân biệt hai trường hợp. Từ có trong từ điển nhưng mọi nhãn bằng 0, và từ không có trong từ điển. Cả hai đều không làm tăng số đếm, nhưng khác nhau ở độ bao phủ của từ điển. Cần ghi riêng.

Chọn mẫu số cũng đổi ý nghĩa của con số.

| Mẫu số | Trả lời câu hỏi nào |
|---|---|
| Tổng số từ N | Trong cả văn bản, bao nhiêu phần trăm từ gợi đến nhóm này |
| Số từ khớp từ điển | Trong những từ có trong từ điển, bao nhiêu phần trăm gợi đến nhóm này |

Hai cách cho kết luận khác nhau khi văn bản dài ngắn khác nhau, nên cần chọn một cách, ghi vào bài, và thử cả hai ở bước kiểm tra.

<a id="muc-5"></a>

**5. Ví dụ bốn tình huống**

Từ điển nhỏ dưới đây là **giả định** để tính cho dễ, không phải bản sao của NRC EmoLex. Muốn dùng số thật thì tải tài nguyên từ trang tác giả.

| Từ | Nhóm gợi đến (giả định) |
|---|---|
| happy | joy, positive |
| safe | trust, positive |
| frightening | fear, negative |

**Tình huống 1: câu có hai từ cảm xúc.** Câu: "we feel happy and safe" (5 từ).

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Khớp từ với từ điển | happy, safe |
| 2 | Đếm cho từng nhóm | joy = 1, trust = 1, positive = 2 |
| 3 | Chia cho N = 5 | joy = 0.2, trust = 0.2, positive = 0.4 |

Ba tỷ lệ này không loại trừ nhau và không phải xác suất.

**Tình huống 2: câu có phủ định.** Câu: "We were not happy during our stay."

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Khớp từ | happy |
| 2 | Đếm | joy = 1, positive = 1 |
| 3 | Đọc kết quả | Hai nhóm vui và khen vẫn được đếm |

Câu thật ra đang chê, nhưng bảng đếm vẫn cho joy và positive. Chữ "not" không được xử lý. Đây là lý do không nên diễn giải bảng đếm thành "khách cảm thấy vui".

**Tình huống 3: từ gợi sợ nhưng câu khen.** Câu: "The staff prevented a frightening situation."

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Khớp từ | frightening |
| 2 | Đếm | fear = 1, negative = 1 |
| 3 | Đọc kết quả | Câu thật ra ghi nhận nhân viên làm tốt, nhưng bảng đếm nghiêng về sợ và chê |

Nhóm từ gợi sợ có mặt trong một lời khen. Chỉ đọc bảng đếm sẽ làm mất cấu trúc này.

**Tình huống 4: tổng tỷ lệ lớn hơn 1.** Câu: "happy safe" (2 từ).

| Nhóm | Số đếm | Tỷ lệ (chia cho 2) |
|---|---|---|
| joy | 1 | 0.5 |
| trust | 1 | 0.5 |
| positive | 2 | 1.0 |

Cộng các tỷ lệ ra 2.0. Điều này bình thường vì mỗi từ thuộc hai nhóm. Nếu ai đó cố làm các tỷ lệ cộng thành 100% thì đã hiểu sai bản chất của bảng này.

<a id="muc-6"></a>

**6. Lỗi hay gặp**

1. Coi bảng đếm là xác suất, hoặc ép mỗi từ hay mỗi câu vào đúng một cảm xúc.
2. Diễn giải số đếm thành cảm xúc của người viết. Ví dụ "tỷ lệ trust cao" không có nghĩa "khách tin thương hiệu".
3. Không xử lý phủ định, như ở tình huống 2.
4. Dùng bản dịch tiếng Việt mà chưa kiểm tra. Bản dịch tự động có thể sai nghĩa hoặc sai văn hóa [S1]. Với tiếng Việt cũng cần xử lý từ nhiều âm tiết, không thể chỉ tách theo dấu cách.
5. So mười nhóm giữa nhiều nhóm khách rồi chỉ báo cáo nhóm nào có khác biệt. Với mười nhóm, dễ tìm ra khác biệt chỉ do may rủi. Cần chọn trước một đến hai nhóm chính trước khi nhìn dữ liệu.
6. Không tính đến độ dài và chủ đề. Nhóm viết dài hơn hoặc nói về chủ đề khác có thể có tỷ lệ khác chỉ vì lý do đó.
7. Đặt tên biến quá rộng. Nên đặt tên là "tỷ lệ từ gợi nhóm trust" thay vì "mức tin tưởng".
8. Dùng để chẩn đoán tâm lý của từng khách. Không dùng đếm từ để suy ra trạng thái thật của một người hay ý định quay lại.
9. Đăng toàn bộ từ điển lên GitHub mà chưa xem điều kiện sử dụng. Nguồn chính thức có điều kiện riêng cho dùng phi thương mại và thương mại [S1]. Nên đăng hướng dẫn tải, mã và từ điển giả định thay vì bản sao tài nguyên.

<a id="muc-7"></a>

**7. Làm sao biết kết quả đáng tin**

Cách kiểm tra phụ thuộc vào câu hỏi nghiên cứu đang hỏi gì.

| Câu hỏi nghiên cứu | Có cần đáp án do người gán không | Cách kiểm tra |
|---|---|---|
| "Văn bản dùng nhiều từ gợi sợ hay vui hơn?" (chỉ đếm từ) | Không bắt buộc | Người đọc các từ đóng góp nhiều nhất xem có đúng nghĩa trong khách sạn không, rồi thử đổi cách tách từ, mẫu số, bản dịch xem kết luận có đổi không |
| "Câu này biểu lộ sợ hay vui?" (phân loại câu) | Có | Làm theo ba bước dưới đây |

Nếu mục tiêu là phân loại câu, làm ba bước giống các file trước, có chỉnh cho nhiều nhãn.

Bước 1. Người tạo đáp án.

1. Chọn khoảng 100 câu từ nhiều khách sạn khác nhau.
2. Hai người gán nhãn độc lập. Mỗi câu có thể có nhiều nhãn cùng đúng, ví dụ vừa joy vừa trust.
3. Máy tính mức hai người đồng ý với nhau, gọi là hệ số kappa, tính riêng cho từng nhãn. Càng gần 1 thì càng đồng ý.
4. Chỗ hai người khác nhau thì bàn lại và chọn một nhãn.

Bước 2. Chia đáp án làm hai phần, một phần để thử và sửa, một phần chấm điểm cuối chỉ dùng một lần. Chia theo khách sạn.

Bước 3. Máy so kết quả với đáp án trên phần chấm điểm cuối. Vì có nhiều nhãn nên tính độ chính xác riêng cho từng nhãn rồi báo từng nhãn, không chỉ báo một con số trung bình. Không dùng một bộ nhãn khen chê rồi kết luận về chất lượng của cả tám cảm xúc.

Để so NRC với bốn cách trước trên bài toán khen chê, cần quy ra một nhãn khen chê. Một đề xuất là lấy số từ positive trừ số từ negative, rồi đặt ngưỡng trên phần thử nghiệm. Đây là cách do người biên soạn đề xuất, không phải cách chuẩn của tài nguyên. Khi đó chạy cả năm cách trên cùng phần chấm điểm cuối rồi so độ đúng trung bình của ba nhãn khen, chê, trung tính. Nhưng cần nhớ NRC không sinh ra để chấm khen chê, nên đây không phải bài toán mạnh nhất của nó.

Khi viết kết luận, "nhóm A có tỷ lệ từ gợi trust cao hơn" không có nghĩa "nhóm A tin thương hiệu hơn". Muốn nói về trust như một khái niệm cần thêm thang đo độc lập và loại trừ khác biệt về chủ đề và độ dài.

<a id="muc-8"></a>

**8. Chọn cái nào trong năm cái**

| Nhu cầu | Cách hợp hơn | Điểm yếu |
|---|---|---|
| Chỉ cần biết khen hay chê, cần mốc đơn giản nhất | AFINN | Không có quy tắc nào về ngữ cảnh, tổng phụ thuộc độ dài |
| Phân biệt các nghĩa khác nhau của cùng một từ | SentiWordNet | Phải tự chọn nghĩa |
| Có chữ hoa, dấu chấm than, "very", "but", "not" | VADER | Mỗi từ chỉ có một điểm |
| Biết câu thiên về ý kiến hay sự việc | TextBlob | Chỉ ước lượng theo từ điển |
| Xem hồ sơ nhiều nhóm cảm xúc như giận, sợ, vui, tin tưởng | NRC EmoLex | Không có cường độ, không xử lý "not", tiếng Việt phải qua dịch |
| Đúng hơn trên đánh giá khách sạn | Chưa biết | Phải đo trên cùng phần chấm điểm cuối |

Với nhiều nhóm cảm xúc, NRC là cách duy nhất trong năm cách có thông tin này. Đổi lại nó không cho biết mạnh hay yếu. Nếu cần cường độ giận hay sợ thì cần một phép đo khác, không phải bảng nhị phân này.

Cả năm đều cho kết quả ở cấp cả văn bản, chưa tách từng khía cạnh như phòng hay dịch vụ.

<a id="muc-9"></a>

**9. Mã chạy thử**

Đoạn mã dưới đây dùng từ điển giả định ở mục 5, không phải bản sao NRC EmoLex. Khi làm với dữ liệu thật, thay bằng tài nguyên tải hợp lệ từ trang tác giả [S1].

```python
# Từ điển giả định, không phải bản sao NRC EmoLex.
lexicon = {
    "happy": {"joy", "positive"},
    "safe": {"trust", "positive"},
    "frightening": {"fear", "negative"},
}
labels = ["joy", "trust", "fear", "positive", "negative"]

def count_emotions(text):
    tokens = text.lower().split()
    counts = {e: sum(e in lexicon.get(w, set()) for w in tokens)
              for e in labels}
    ratios = {e: counts[e] / len(tokens) for e in labels}
    matched = sum(w in lexicon for w in tokens)
    return counts, ratios, matched, len(tokens)

print(count_emotions("we feel happy and safe"))
print(count_emotions("We were not happy during our stay"))
```

Hàm này trả về số đếm, tỷ lệ, số từ khớp từ điển và tổng số từ, để phân biệt độ bao phủ với số đếm. Cách tách `.split()` chỉ dùng cho tiếng Anh ngắn, không phải cách tách từ tiếng Việt đầy đủ. Mã chưa được chạy thử trong tài liệu này.

<a id="muc-10"></a>

**10. Tài liệu tham khảo**

- **[S1]** Mohammad, S. M., & Turney, P. D. [NRC Emotion Lexicon: mô tả, tải và điều kiện sử dụng](https://saifmohammad.com/WebPages/NRC-Emotion-Lexicon.htm).
- **[S2]** Mohammad, S. M., & Turney, P. D. (2013). *Crowdsourcing a Word-Emotion Association Lexicon*. Computational Intelligence, 29(3), 436-465. [Bản thảo được trang tác giả dẫn](https://arxiv.org/abs/1308.6297).
- **[S3]** Mohammad, S. (2023). *Best Practices in the Creation and Use of Emotion Lexicons*. [ACL Anthology](https://aclanthology.org/2023.findings-eacl.136/).

S1 và S2 cho thông tin tài nguyên, S3 thảo luận thực hành và đạo đức khi dùng từ điển cảm xúc. Phần so sánh với bốn cách trước dựa trên các file trước. Ví dụ khách sạn, từ điển nhỏ giả định, đề xuất quy ra khen chê và kế hoạch kiểm tra ở trên là đề xuất của người biên soạn, không phải kết quả của ba nguồn này.
