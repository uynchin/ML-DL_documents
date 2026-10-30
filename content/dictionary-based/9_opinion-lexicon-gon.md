# Opinion Lexicon của Hu và Liu: giải thích đơn giản

**Mục lục**

- [1. Opinion Lexicon là gì](#muc-1)
- [2. So sánh với các cách trước](#muc-2)
- [3. Ai làm gì](#muc-3)
- [4. Cách tính điểm](#muc-4)
- [5. Ví dụ bốn tình huống](#muc-5)
- [6. Lỗi hay gặp](#muc-6)
- [7. Làm sao biết kết quả đáng tin](#muc-7)
- [8. Chọn cái nào trong chín cái](#muc-8)
- [9. Mã chạy thử](#muc-9)
- [10. Tài liệu tham khảo](#muc-10)

<a id="muc-1"></a>

**1. Opinion Lexicon là gì**

Đây là hai danh sách từ tiếng Anh: một danh sách từ khen và một danh sách từ chê. Danh sách gắn với bài của Minqing Hu và Bing Liu năm 2004 về tóm tắt đánh giá của khách hàng [S1]. Trong gói tải về, mỗi danh sách là một tệp chữ, mỗi dòng một từ [S2].

Đây là cách đơn giản nhất trong chín cách. Mỗi từ chỉ có một nhãn: khen hoặc chê. Không có điểm số, không có mức mạnh yếu, không có loại từ, không có nghĩa.

Số lượng thường được ghi là khoảng 6786 từ, trong đó 2006 từ khen và 4783 từ chê [S3]. Hai con số thành phần cộng lại là 6789, khác với 6786, nên cần đếm lại trên tệp thật trước khi viết vào bài.

Có hai điều dễ nhầm.

1. Danh sách chỉ nói từ đó khen hay chê. Từ "terrible" và từ "slow" đều là chê và nặng như nhau khi đếm. Mức độ không có trong danh sách.
2. Không có trong danh sách không có nghĩa là trung tính. Từ có thể thuộc loại danh sách chưa chứa.

Cách chấm câu trong bài gốc gần với cách đếm: câu có nhiều từ khen hơn thì nghiêng khen, nhiều từ chê hơn thì nghiêng chê. Khi số từ khen và chê bằng nhau, bài gốc dùng thêm một số dấu hiệu khác để quyết định [S1]. Với nghiên cứu này, cần quy tắc riêng và phải ghi rõ.

<a id="muc-2"></a>

**2. So sánh với các cách trước**

Thông tin về tám cách trước lấy từ các file trước.

| Điểm so sánh | SentiWordNet | VADER | AFINN | NRC | MPQA | Opinion Lexicon |
|---|---|---|---|---|---|---|
| Dạng giá trị của mục | Ba điểm theo từng nghĩa | Điểm từ -4 đến +4 | Số nguyên từ -5 đến +5 | Bảng 0 hoặc 1 cho mười nhóm | Nhãn chữ kèm mạnh yếu | Chỉ khen hoặc chê |
| Có mức mạnh yếu | Có | Có | Có | Không | Hai mức | Không |
| Phải chọn nghĩa | Có | Không | Không | Không | Không | Không |
| Có nhãn trung tính | Qua điểm trung lập | Qua ngưỡng | Qua ngưỡng | Không có nhóm trung tính riêng | Có | Không có |
| Xử lý "not", "very", dấu câu | Người tự viết | Có sẵn | Không có sẵn | Không | Không có sẵn | Không có sẵn |
| Kết quả cuối | Người tự gộp | compound | Tổng điểm | Bảng đếm | Người tự gộp | Người tự đếm |
| Dễ kiểm tra bằng tay | Trung bình | Trung bình | Dễ | Dễ | Dễ | Dễ nhất |

Điểm khác biệt lớn nhất: Opinion Lexicon bỏ hết thông tin ngoài việc khen hay chê. Đổi lại, mỗi từ hiện ra thành hai dòng ngắn trong hai tệp, và người đọc kiểm tra từng từ rất nhanh.

Đặt các cách cạnh nhau trên cùng câu: "The staff were not friendly."

| Cách | Điều gì xảy ra |
|---|---|
| SentiWordNet | Tra "friendly" ra điểm khen. Cần quy tắc "not" do người viết |
| VADER | Tự nhận ra "not", ra điểm chê |
| TextBlob | Có xử lý một số dấu hiệu, cần chạy mã để biết |
| AFINN | Tra "friendly" ra điểm khen, sai, trừ khi danh sách có sẵn cụm phủ định |
| NRC | Đếm "friendly" vào nhóm khen. Chữ "not" không làm đổi gì |
| LIWC | Đếm từ theo các ngăn, không chấm khen chê |
| SenticNet | Nếu chỉ tra mà không xử lý "not" thì vẫn sai |
| MPQA | Khớp "friendly" ra nhãn khen nếu có trong danh sách, sai |
| Opinion Lexicon | Khớp "friendly" trong danh sách khen, P = 1, khen, sai. Chữ "not" không làm đổi gì, trừ khi người tự viết quy tắc |

Cách nào cũng có chung một điểm yếu với chữ "not". Khác nhau ở chỗ VADER có sẵn quy tắc, còn các cách còn lại thì người phải tự lo.

<a id="muc-3"></a>

**3. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Tải hai tệp, ghi tên tệp và ngày tải | Người |
| Chọn chấm từng câu hay cả bài đánh giá | Người |
| Chọn cách tính: hiệu, tỷ lệ, hay chia số từ | Người |
| Viết quy tắc cho chữ "not" nếu cần | Người viết, máy áp dụng |
| Gán nhãn khen, chê, trung tính cho một tập câu mẫu làm đáp án | Người |
| Tách từ, khớp với hai danh sách | Máy |
| Đếm số từ khen và số từ chê | Máy |
| Đổi kết quả thành nhãn bằng ngưỡng | Máy làm, người quyết định ngưỡng |
| So nhãn của máy với đáp án, tính chỉ số | Máy tính, người đọc các câu sai |
| Kết luận trong bài viết | Người |

Nhãn khen và chê của từng từ đã có sẵn, người dùng không gán lại. Theo bài gốc, hướng làm là bắt đầu từ một ít tính từ mẫu, rồi mở rộng bằng từ đồng nghĩa và trái nghĩa trong WordNet, một cuốn từ điển tiếng Anh có quan hệ giữa các từ [S1]. Danh sách phát hành sau này có thể đã được mở rộng thêm, và cần đọc README đi kèm để biết người hay máy đã làm từng phần trước khi viết vào bài nghiên cứu [S4].

<a id="muc-4"></a>

**4. Cách tính điểm**

Danh sách không có công thức chuẩn. Dưới đây là ba cách người nghiên cứu có thể chọn, đều là đề xuất của người biên soạn.

```text
S = P trừ N
R = (P trừ N) chia (P cộng N)
D = (P trừ N) chia T
```

| Ký hiệu | Ý nghĩa |
|---|---|
| P | Số từ của văn bản có trong danh sách khen |
| N | Số từ của văn bản có trong danh sách chê |
| T | Tổng số từ của văn bản. Phải ghi rõ cách đếm, ví dụ bỏ dấu câu hay không |
| S | Hiệu số, có thể lớn khi văn bản dài |
| R | Tỷ lệ, nằm trong khoảng từ -1 đến 1. Nếu P cộng N bằng 0 thì ghi là thiếu dữ liệu |
| D | Mật độ, cho biết trong cả văn bản nghiêng về khen hay chê cỡ nào |

| Cách | Trả lời câu hỏi nào | Điểm yếu |
|---|---|---|
| S | Có nhiều từ khen hơn từ chê bao nhiêu | Văn bản dài thì S lớn hơn |
| R | Trong các từ có cảm xúc, bên nào chiếm nhiều hơn | Không cho biết văn bản có nhiều hay ít từ cảm xúc |
| D | Mật độ khen chê trong cả văn bản | Văn bản nhiều từ không cảm xúc thì D bị kéo về gần 0 |

Ngưỡng đổi thành nhãn: danh sách không kèm ngưỡng chuẩn. Ví dụ giả định, S lớn hơn 0 là khen, nhỏ hơn 0 là chê. Nếu S bằng 0 và P cộng N lớn hơn 0 thì ghi là hỗn hợp. Nếu P cộng N bằng 0 thì ghi là thiếu dữ liệu. Giữ P và N riêng giúp phân biệt hai trường hợp này, điều mà một con số duy nhất không làm được.

<a id="muc-5"></a>

**5. Ví dụ bốn tình huống**

Danh sách nhỏ dưới đây là **giả định** để tính cho dễ, không phải trích từ tệp thật. Kết quả thật phải chạy mã ở mục 9.

| Danh sách giả định | Các từ |
|---|---|
| Khen | friendly, delicious, clean, good |
| Chê | terrible, slow |

**Tình huống 1: câu chỉ có lời khen.** Câu: "The staff were friendly and the breakfast was delicious."

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Khớp từ với hai danh sách | friendly, delicious |
| 2 | Đếm | P = 2, N = 0 |
| 3 | Tính hiệu | S = 2 |
| 4 | So với ngưỡng 0 | Khen |

**Tình huống 2: hai câu chê khác mức độ.**

Câu A: "The room was clean, but the service was terrible."
Câu B: "The room was clean, but the service was slow."

| Bước | Câu A | Câu B |
|---|---|---|
| Từ khớp | clean, terrible | clean, slow |
| P và N | P = 1, N = 1 | P = 1, N = 1 |
| S | 0 | 0 |
| R | 0 | 0 |
| Nhãn | Hỗn hợp | Hỗn hợp |

Hai câu cho cùng kết quả dù "terrible" nặng hơn "slow" nhiều. Danh sách không có mức mạnh yếu nên không phân biệt được. Muốn phân biệt phải dùng cách có điểm số như AFINN hoặc VADER. Hai câu cũng không cho biết phòng được khen còn dịch vụ bị chê, muốn biết phải chấm riêng từng khía cạnh.

**Tình huống 3: hai đánh giá cùng khen nhưng khác độ dài.**

Đánh giá A: "Good room." (2 từ, P = 1)
Đánh giá B: "The room was good and the bed was good and the breakfast was good." (14 từ, P = 3)

| Cách tính | Đánh giá A | Đánh giá B |
|---|---|---|
| S | 1 | 3 |
| R | 1 | 1 |
| D | 0.5 | khoảng 0.21 |

Nếu chỉ nhìn S, B có vẻ khen gấp ba A. Nếu dùng R thì hai đánh giá bằng nhau. Nếu dùng D thì A lại khen mạnh hơn B. Ba cách cho ba kết luận khác nhau, nên phải chọn cách phù hợp với câu hỏi nghiên cứu và ghi vào bài.

**Tình huống 4: có chữ "not".** Câu: "The bed was not good."

| Cách làm | Cách tính | Kết quả | Đúng hay sai |
|---|---|---|---|
| Chỉ khớp từng từ | P = 1, S = 1 | Khen | Sai |
| Có quy tắc người viết: nếu từ cảm xúc nằm trong ba từ sau "not" thì chuyển sang danh sách bên kia | P = 0, N = 1, S = -1 | Chê | Đúng |

Quy tắc vẫn có thể sai ở các câu như "not only clean but also quiet". Nên thử trên các câu mẫu trước khi dùng. Khi lọc từ phổ biến, cần giữ lại "not", "no", "never".

<a id="muc-6"></a>

**6. Lỗi hay gặp**

1. Coi hiệu số S là mức độ hài lòng. S chỉ đếm số từ, mọi từ nặng như nhau và S tăng theo độ dài.
2. Ghi điểm 0 khi không khớp được từ nào. Điểm 0 nghe như trung tính, thực tế là thiếu dữ liệu. Cần ghi riêng.
3. Nhầm hỗn hợp với thiếu dữ liệu. P bằng N khác với P bằng N bằng 0.
4. Bỏ chữ "not" khi lọc các từ phổ biến. Danh sách từ bị bỏ mặc định thường chứa "not". Cần giữ lại.
5. Từ có thể đổi nghĩa theo ngữ cảnh. Danh sách không phân nghĩa, nên một từ đứng trong danh sách khen có thể dùng ở nghĩa khác trong câu. Nên đọc thử khoảng 50 câu chứa các từ hay gặp nhất để đếm máy sai bao nhiêu.
6. Dùng cho tiếng Việt. Danh sách là tiếng Anh, nên dùng thẳng cho tiếng Việt thì không đáng tin. Nếu dịch rồi chấm thì lỗi dịch cộng thêm vào lỗi chấm.
7. Cộng S của cả tháng để xem xu hướng. Tổng tăng có thể chỉ vì tháng đó có nhiều đánh giá hơn. Nên dùng R hoặc D rồi lấy trung bình.
8. Không ghi tệp thật sự dùng. Có các bản chia sẻ khác nhau, ví dụ bản trong thư viện NLTK và bản trong gói R tidytext, và gói R đã loại các từ có ký tự ngoài bảng chữ cái tiếng Anh [S2] [S3]. Cần ghi nguồn và ngày tải.
9. Dùng nhãn của danh sách làm đáp án. Đáp án phải do người gán trên câu thật.

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
| Phần thử nghiệm | Chọn S, R hoặc D, chọn ngưỡng, thử quy tắc "not". Được sửa thoải mái |
| Phần chấm điểm cuối | Chỉ dùng một lần ở cuối, không sửa gì sau khi thấy kết quả |

Chia theo khách sạn, để câu của cùng một khách sạn chỉ nằm ở một phần.

Bước 3. Máy so nhãn với đáp án trên phần chấm điểm cuối, người báo các con số sau.

| Con số | Ý nghĩa đơn giản |
|---|---|
| Độ đúng trung bình của ba nhãn (macro-F1) | Máy đúng bao nhiêu, tính riêng cho khen, chê, trung tính rồi lấy trung bình |
| Bảng nhầm lẫn | Máy hay nhầm nhãn nào thành nhãn nào |
| Tỷ lệ câu khớp được ít nhất một từ (coverage) | Máy hiểu được bao nhiêu phần dữ liệu |
| Kết quả theo nhóm độ dài | Câu ngắn và câu dài có đúng như nhau không |

Với Opinion Lexicon có thêm một kiểm tra đáng làm: chạy cả S, R và D, xem xếp hạng khách sạn hoặc kết luận có đổi không. Nếu đổi mạnh thì phải báo tính nhạy này trong bài, không chọn cách nào cho ra kết luận thuận lợi nhất.

Danh sách này là mốc đơn giản nhất để đặt các cách phức tạp hơn lên so. Chạy cả chín cách trên cùng phần chấm điểm cuối rồi so macro-F1. Nếu VADER không hơn Opinion Lexicon thì các quy tắc của VADER chưa giúp được gì trên dữ liệu đó.

Khi viết kết luận, kết quả từ vựng chỉ cho biết chữ viết nghiêng về khen hay chê. Muốn nói khách hài lòng hơn thì cần thêm bằng chứng, ví dụ đối chiếu với số sao nếu có.

<a id="muc-8"></a>

**8. Chọn cái nào trong chín cái**

| Nhu cầu | Cách hợp hơn | Lý do | Điểm yếu |
|---|---|---|---|
| Kiểm tra bằng tay dễ nhất, làm mốc thấp nhất, chỉ cần khen hay chê | Opinion Lexicon | Hai tệp, mỗi dòng một từ | Không có mức mạnh yếu, không xử lý "not" |
| Kiểm tra bằng tay dễ nhưng cần mức mạnh yếu | AFINN | Điểm nguyên từ -5 đến +5 | Điểm tổng phụ thuộc độ dài |
| Có nhãn chủ quan mạnh yếu và nhãn trung tính | MPQA | Mỗi mục có type và priorpolarity | Không có điểm số |
| Chữ hoa, dấu chấm than, "very", "but", "not" | VADER | Có sẵn các quy tắc này | Mỗi từ chỉ có một điểm |
| Biết câu thiên về ý kiến hay sự việc bằng một con số | TextBlob | Có subjectivity | Chỉ là trung bình điểm chủ quan |
| Phân biệt các nghĩa khác nhau của cùng một từ | SentiWordNet | Điểm theo từng nghĩa | Phải tự chọn nghĩa |
| Hồ sơ nhiều nhóm cảm xúc | NRC | Mười nhóm cùng lúc | Không có mức mạnh yếu |
| Nhiều nhóm ngôn ngữ, giả thuyết về phong cách | LIWC | Có tài liệu kiểm định | Không phải bộ phân loại khen chê, cần quyền dùng |
| Hiểu cả cụm nhiều từ | SenticNet | Đơn vị tra là khái niệm | Thêm bước trích khái niệm, khó kiểm tra |
| Đúng hơn trên đánh giá khách sạn | Chưa biết | Chưa có bằng chứng | Phải đo macro-F1 trên cùng phần chấm điểm cuối |

Các cách đều chỉ dùng cho tiếng Anh và đều chưa tách từng khía cạnh. Mỗi cách mạnh ở một chỗ, không có cách nào hơn tuyệt đối.

<a id="muc-9"></a>

**9. Mã chạy thử**

Thư viện NLTK có sẵn bộ đọc cho danh sách này [S2]. Cài gói bằng lệnh `pip install nltk`, rồi chạy:

```python
import re
import nltk
nltk.download("opinion_lexicon")
from nltk.corpus import opinion_lexicon

positive = set(opinion_lexicon.positive())
negative = set(opinion_lexicon.negative())

def score(text):
    tokens = re.findall(r"[a-z']+", text.lower())
    p = sum(t in positive for t in tokens)
    n = sum(t in negative for t in tokens)
    t_total = len(tokens)
    if p + n == 0:
        return {"P": p, "N": n, "S": 0, "R": None, "D": 0.0, "nhan": "thiếu dữ liệu"}
    s = p - n
    nhan = "khen" if s > 0 else "chê" if s < 0 else "hỗn hợp"
    return {"P": p, "N": n, "S": s, "R": s / (p + n), "D": s / t_total, "nhan": nhan}

texts = [
    "The staff were friendly and the breakfast was delicious.",
    "The room was clean, but the service was terrible.",
    "The room was clean, but the service was slow.",
    "Good room.",
    "The bed was not good.",
]
for text in texts:
    print(text, score(text))
```

Mã này in kết quả thật cho các câu, để đối chiếu với danh sách giả định ở mục 5. Mã chưa được chạy thử trong tài liệu này. Mã chưa xử lý "not". Tách từ bằng biểu thức đơn giản, nên các từ có dấu gạch nối hoặc chữ số trong danh sách có thể không khớp, cần kiểm tra khi dùng thật.

<a id="muc-10"></a>

**10. Tài liệu tham khảo**

- **[S1]** Hu, M., & Liu, B. (2004). *Mining and Summarizing Customer Reviews*. Proceedings of ACM SIGKDD 2004. [DOI](https://doi.org/10.1145/1014052.1014073).
- **[S2]** NLTK. [Bộ đọc Opinion Lexicon](https://www.nltk.org/_modules/nltk/corpus/reader/opinion_lexicon.html).
- **[S3]** Gói tidytext của R. [Trang mô tả dữ liệu sentiments](https://www.rdocumentation.org/packages/tidytext/topics/sentiments).
- **[S4]** Bing Liu. [Trang về phân tích ý kiến](https://www.cs.uic.edu/~liub/FBS/sentiment-analysis.html).

Các nguồn trên giới thiệu tài nguyên và gói phần mềm. Phần so sánh với tám cách trước dựa trên các file trước. Ví dụ khách sạn, danh sách nhỏ giả định, các cách tính và kế hoạch kiểm tra ở trên là đề xuất của người biên soạn, chưa phải kết quả của một thí nghiệm thực tế.
