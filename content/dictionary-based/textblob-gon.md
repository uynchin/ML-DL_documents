# TextBlob (PatternAnalyzer): giải thích đơn giản

**Mục lục**

- [1. TextBlob là gì](#muc-1)
- [2. So sánh với SentiWordNet và VADER](#muc-2)
- [3. Ai làm gì](#muc-3)
- [4. Hai con số: polarity và subjectivity](#muc-4)
- [5. Ví dụ bốn câu](#muc-5)
- [6. Lỗi hay gặp](#muc-6)
- [7. Làm sao biết kết quả đáng tin](#muc-7)
- [8. Chọn cái nào trong ba cái](#muc-8)
- [9. Mã chạy thử](#muc-9)
- [10. Tài liệu tham khảo](#muc-10)

<a id="muc-1"></a>

**1. TextBlob là gì**

TextBlob là một thư viện Python, giống một hộp công cụ có nhiều đầu máy. Ghi "dùng TextBlob" thì người đọc chưa biết đầu máy nào làm ra kết quả. Với cảm xúc, TextBlob có hơn một bộ phân tích, và file này chỉ nói về bộ tên **PatternAnalyzer**, dùng cho tiếng Anh [S2].

PatternAnalyzer cho hai con số cho mỗi văn bản [S1]:

| Con số | Khoảng giá trị | Ý nghĩa đơn giản |
|---|---|---|
| polarity | Từ -1 đến 1 | Nghiêng về chê (âm) hay khen (dương) |
| subjectivity | Từ 0 đến 1 | Câu nói về sự việc (gần 0) hay thể hiện ý kiến cá nhân (gần 1) |

Hai con số này đo hai thứ khác nhau. Một câu có thể rất chủ quan nhưng không nghiêng về khen hay chê, và ngược lại.

Điểm hay bị hiểu nhầm: subjectivity không phải độ tin cậy, cũng không cho biết câu đúng hay sai. Polarity cũng không phải xác suất.

<a id="muc-2"></a>

**2. So sánh với SentiWordNet và VADER**

Bảng dưới so ba cách làm. Thông tin về SentiWordNet và VADER lấy từ hai file trước.

| Điểm so sánh | SentiWordNet | VADER | TextBlob (PatternAnalyzer) |
|---|---|---|---|
| Điểm trong từ điển tính theo | Từng nghĩa của từ | Từng từ | Từ điển có sẵn của Pattern, xem [S3] [S4] |
| Có phải chọn nghĩa không | Có | Không | Không |
| Đầu ra | Người tự gộp điểm | Điểm tổng compound, cùng pos, neu, neg | polarity và subjectivity |
| Có đo mức chủ quan không | Không | Không | Có |
| Xử lý "not", "very", dấu câu | Người phải tự viết | Có sẵn, gồm cả chữ hoa và dấu chấm than | Có xử lý một số dấu hiệu, chi tiết cần đối chiếu mã nguồn [S3] vì có thể thay đổi theo phiên bản |
| Xem được vì sao ra điểm | Xem điểm từng nghĩa | Xem điểm từng từ | Xem được danh sách assessments, tức các từ hoặc cụm được chấm kèm điểm |

Đặt ba cách làm cạnh nhau trên cùng bốn câu. Với TextBlob các con số là giả định, điểm thật phải chạy mã ở mục 9.

| Câu | SentiWordNet | VADER | TextBlob |
|---|---|---|---|
| The hotel has 100 rooms. | Không có từ cảm xúc, ra thiếu dữ liệu hoặc gần 0 | Gần 0, trung tính | polarity gần 0 và subjectivity thấp, cho biết đây là câu nói về sự việc |
| I think the room is wonderful. | Có điểm khen | Có điểm khen | polarity dương và subjectivity cao, vì có từ "wonderful" là từ ý kiến (giải thích ở mục 4) |
| The room is wonderful but the staff are rude. | Cộng đơn giản thì khen chê bù nhau | Chữ "but" làm vế sau nặng hơn | Trung bình hai ý, nên polarity gần 0 và che mất ý khen ý chê |
| The bed was not comfortable. | Phải tự viết quy tắc đảo dấu | Tự nhận ra "not" | Có xử lý phủ định, cần chạy mã xem kết quả |

Điểm khác biệt chính là TextBlob thêm con số subjectivity, hai cách còn lại không có. Điểm giống nhau là cả ba đều cho một điểm cho cả câu, nên đều chưa nói được khen cái gì, chê cái gì.

<a id="muc-3"></a>

**3. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Ghi rõ dùng PatternAnalyzer, ghi phiên bản thư viện | Người |
| Quyết định chấm từng câu hay cả bài đánh giá | Người |
| Quyết định giữ hay bỏ các câu không có từ nào được chấm | Người |
| Gán nhãn khen, chê, trung tính cho một tập câu mẫu làm đáp án | Người |
| Tách câu, tra điểm trong từ điển | Máy |
| Tính polarity và subjectivity | Máy |
| Đổi polarity thành nhãn khen, chê, trung tính | Máy làm, người quyết định ngưỡng |
| So nhãn của máy với đáp án của người | Máy tính, người đọc các câu sai |
| Kết luận trong bài viết | Người |

Với TextBlob, TextBlob không tự đưa ra mức ngưỡng. Tài liệu chỉ cho hai con số, còn đổi sang khen, chê, trung tính là việc của người, chọn trên phần thử nghiệm ở mục 7.

<a id="muc-4"></a>

**4. Hai con số: polarity và subjectivity**

Một văn bản thường có vài từ hoặc cụm được chấm, mỗi mục gọi là một assessment. Máy gộp các assessment này lại. Cách hiểu đơn giản nhất là lấy trung bình. Cách này chỉ để minh họa, mã Pattern có thể xử lý chi tiết hơn [S3].

```text
polarity cả câu = trung bình polarity của các assessment
subjectivity cả câu = trung bình subjectivity của các assessment
```

| Ký hiệu | Ý nghĩa |
|---|---|
| assessment | Một từ hoặc cụm được chấm, kèm điểm polarity và subjectivity của nó |
| m | Số assessment tìm được trong văn bản. Nếu m bằng 0 thì ghi là thiếu dữ liệu |

Đổi polarity thành nhãn cần một ngưỡng do người chọn. Ví dụ giả định, ngưỡng là 0.1:

| Điều kiện | Nhãn |
|---|---|
| polarity lớn hơn 0.1 | Khen |
| polarity nhỏ hơn -0.1 | Chê |
| Còn lại | Trung tính |

Tên TextBlob không cho sẵn mức ngưỡng nào. Mức 0.1 ở đây chỉ để minh họa.

**Vì sao subjectivity cho biết "ý kiến" hay "sự việc"**

Máy không hiểu câu. Nó làm giống tra từ điển. Theo cách Pattern được mô tả [S3] [S4], mỗi từ đánh giá trong từ điển được lưu kèm hai điểm riêng: polarity (khen hay chê) và subjectivity (mức ý kiến cá nhân). Điểm subjectivity của cả câu là trung bình điểm subjectivity của các từ tìm thấy trong câu. Cách chắc nhất để thấy điều này là chạy mã ở mục 9 và đọc danh sách assessments.

Ví dụ ba từ trong từ điển, các điểm là giả định:

| Từ | polarity | subjectivity | Ý nghĩa |
|---|---|---|---|
| wonderful | 0.9 | 0.9 | Khen mạnh, thể hiện ý kiến cá nhân |
| rude | -0.6 | 0.9 | Chê, thể hiện ý kiến cá nhân |
| large | 0.0 | 0.4 | Nghiêng về mô tả, ít khen chê |

Từ đó suy ra cách máy "phân biệt" ý kiến và sự việc: câu có nhiều từ ý kiến thì subjectivity cao, câu không có từ nào trong từ điển thì subjectivity bằng 0 và được coi là sự việc.

| Câu | Từ máy tìm thấy trong từ điển | subjectivity (giả định) | Máy cho là | Thực tế |
|---|---|---|---|---|
| The hotel has 100 rooms. | Không có | 0 | Sự việc | Đúng là sự việc |
| The room is wonderful. | wonderful | Cao | Ý kiến | Đúng là ý kiến |
| The wifi did not work for 3 days. | Có thể không có từ nào | Thấp hoặc 0 | Sự việc | Là lời phàn nàn |
| I would never stay here again. | Có thể không có từ nào | Thấp hoặc 0 | Sự việc | Là ý kiến rất mạnh |

Hai dòng cuối cho thấy giới hạn. Máy chỉ nhìn vào từ có trong từ điển, nên ý kiến viết bằng những từ ngoài từ điển có thể bị coi là sự việc. Cụm "I think" cũng không tự làm câu thành ý kiến, trừ khi từ điển có mục đó. Cần chạy mã để xác nhận kết quả thật cho hai dòng cuối.

Nói gọn: subjectivity là ước lượng dựa trên từ vựng, không phải hiểu ý. Dùng để thăm dò thì được, chưa đủ để kết luận một câu là ý kiến hay sự việc.

<a id="muc-5"></a>

**5. Ví dụ bốn câu**

Các điểm là giả định để tính cho dễ. Điểm thật phải chạy mã ở mục 9.

**Câu 1: "The hotel has 100 rooms."** Không có từ ý kiến. Giả định polarity bằng 0 và subjectivity bằng 0. Nhãn là trung tính, và câu nói về sự việc.

**Câu 2: "I think the room is wonderful."** Giả định polarity bằng 0.9 và subjectivity bằng 0.9. Nhãn là khen. Subjectivity cao vì từ "wonderful" là từ ý kiến trong từ điển, không phải vì cụm "I think" (xem giải thích ở mục 4).

**Câu 3: "The room is wonderful but the staff are rude."** Tính từng bước.

Bước 1. Máy tìm hai assessment và điểm của chúng (giả định).

| Assessment | polarity | subjectivity |
|---|---|---|
| wonderful | 0.8 | 0.7 |
| rude | -0.6 | 0.9 |

Bước 2. Máy lấy trung bình.

| Con số | Phép tính | Kết quả |
|---|---|---|
| polarity cả câu | (0.8 cộng -0.6) chia 2 | 0.1 |
| subjectivity cả câu | (0.7 cộng 0.9) chia 2 | 0.8 |

Bước 3. So với ngưỡng 0.1 (giả định). Polarity 0.1 nằm trong khoảng trung tính nên nhãn là trung tính. Subjectivity 0.8 cho biết câu chủ quan nhiều. Nhưng câu thật ra vừa khen vừa chê, nên điểm trung bình che mất cả hai ý.

**Câu 4: "The bed was not comfortable."** Giả định polarity bằng -0.4 và subjectivity bằng 0.7. Nhãn là chê. Cần chạy mã để biết máy xử lý chữ "not" thế nào trong thực tế.

<a id="muc-6"></a>

**6. Lỗi hay gặp**

1. Gọi subjectivity là độ tin cậy. Ví dụ ghi "subjectivity bằng 0.9 nghĩa là máy chắc chắn 90%" là sai. Subjectivity chỉ cho biết câu thiên về ý kiến hay về sự việc.
2. Dùng subjectivity để kết luận đánh giá giả hoặc kém tin cậy. Điểm này không cho căn cứ nào để kết luận như vậy.
3. Loại các câu có subjectivity thấp cho gọn dữ liệu. Câu "The wifi did not work for 3 days" nói về sự việc nên subjectivity có thể thấp, nhưng thực chất là lời phàn nàn. Loại các câu này có thể làm mất những lời chê quan trọng. Nếu muốn lọc, thử cả khi có lọc và không lọc rồi so kết quả.
4. Áp ngưỡng của VADER (0.05) cho TextBlob mà không giải thích. Hai bộ có thang điểm riêng. Cần chọn ngưỡng cho TextBlob trên phần thử nghiệm.
5. Dùng cho tiếng Việt. Đây là bộ tiếng Anh. Nếu dịch trước rồi chấm thì phải lưu bản dịch, kiểm tra cả lỗi dịch lẫn lỗi chấm.
6. Dùng nhãn của máy làm đáp án cho mô hình khác. Đáp án phải do người gán.

<a id="muc-7"></a>

**7. Làm sao biết kết quả đáng tin**

Kiểm tra theo ba bước, giống hai file trước.

Bước 1. Người tạo đáp án.

1. Chọn khoảng 100 câu từ nhiều khách sạn khác nhau.
2. Hai người gán nhãn khen, chê hoặc trung tính, mỗi người làm riêng.
3. Máy tính mức hai người đồng ý với nhau, gọi là hệ số kappa. Càng gần 1 thì càng đồng ý.
4. Chỗ hai người khác nhau thì bàn lại và chọn một nhãn.

Bước 2. Chia đáp án làm hai phần.

| Phần | Dùng để làm gì |
|---|---|
| Phần thử nghiệm | Chọn ngưỡng cho polarity, thử có lọc theo subjectivity hay không. Được sửa thoải mái |
| Phần chấm điểm cuối | Chỉ dùng một lần ở cuối |

Chia theo khách sạn, để câu của cùng một khách sạn chỉ nằm ở một phần.

Bước 3. Máy so nhãn với đáp án trên phần chấm điểm cuối, rồi người báo độ đúng trung bình của ba nhãn (macro-F1) và bảng nhầm lẫn.

Nếu muốn dùng subjectivity, cần đáp án riêng cho nó. Người gán thêm một nhãn cho mỗi câu mẫu, là "ý kiến" hay "sự việc", rồi so với subjectivity của máy. Nhãn khen chê không đủ để kiểm tra subjectivity.

Để so ba cách làm cho công bằng, chạy cả SentiWordNet, VADER và TextBlob trên cùng phần chấm điểm cuối rồi so macro-F1.

Khi viết kết luận, polarity cao hơn ở nhóm A cho biết chữ viết nhóm A nghiêng về khen nhiều hơn. Muốn nói nhóm A hài lòng hơn cần thêm bằng chứng. Còn "nhóm A trung thực hơn vì subjectivity thấp" thì không có căn cứ từ hai con số này.

<a id="muc-8"></a>

**8. Chọn cái nào trong ba cái**

| Chọn | Khi nào |
|---|---|
| SentiWordNet | Muốn thấy điểm từng nghĩa, muốn tự kiểm soát cách chọn nghĩa và cách gộp điểm |
| VADER | Cần chạy nhanh, ít việc thêm, văn bản có dấu chấm than, chữ viết hoa, "not", "very" |
| TextBlob PatternAnalyzer | Cần thêm thông tin xem câu là ý kiến hay sự việc, và muốn xem danh sách assessments |
| Cả ba | Muốn so sánh trên cùng phần chấm điểm cuối |

**Hơn và thua giữa ba cách**

| Nhu cầu | Cách hợp hơn | Lý do | Điểm yếu |
|---|---|---|---|
| Cùng một từ có nghĩa khác nhau, cần phân biệt | SentiWordNet | Điểm tính theo từng nghĩa | Phải tự chọn nghĩa, chọn sai thì điểm sai |
| Văn bản có chữ viết hoa, dấu chấm than, "very", "but", "not" | VADER | Có sẵn các quy tắc này | Mỗi từ chỉ có một điểm, không phân biệt nghĩa |
| Biết câu thiên về ý kiến hay về sự việc | TextBlob | Là cách duy nhất trong ba cách có con số này | Chỉ ước lượng theo từ điển, dễ coi lời phàn nàn viết như sự việc là sự việc |
| Chạy ít việc nhất | VADER hoặc TextBlob | Có sẵn đầu ra, không cần tự chọn nghĩa | Vẫn cần kiểm tra bằng đáp án |
| Chấm khen chê đúng hơn trên đánh giá khách sạn | Chưa biết | Chưa có bằng chứng cho cách nào hơn | Phải đo bằng macro-F1 trên cùng phần chấm điểm cuối |

Vì vậy không có cách nào hơn tuyệt đối. Mỗi cách mạnh ở một chỗ và yếu ở một chỗ. Chỉ khi chạy cả ba trên cùng phần chấm điểm cuối thì mới biết cách nào đúng hơn trên dữ liệu của nghiên cứu.

Cả ba đều chỉ dùng cho tiếng Anh và đều cho một điểm cho cả câu, chưa tách từng khía cạnh. Nếu mục tiêu chỉ là phân loại khen chê, bằng chứng chính là độ đúng trên nhãn khen chê, không cần dùng mọi con số chỉ vì thư viện cung cấp.

<a id="muc-9"></a>

**9. Mã chạy thử**

Cài gói bằng lệnh `pip install textblob`, rồi chạy. Một số thao tác khác của TextBlob có thể cần tải thêm dữ liệu, nếu có thông báo thiếu thì tải theo thông báo.

```python
from textblob import TextBlob
from textblob.sentiments import PatternAnalyzer

texts = [
    "The hotel has 100 rooms.",
    "I think the room is wonderful.",
    "The room is wonderful but the staff are rude.",
    "The bed was not comfortable.",
]
for text in texts:
    blob = TextBlob(text, analyzer=PatternAnalyzer())
    print(text)
    print(blob.sentiment)
    print(blob.sentiment_assessments.assessments)
```

Mã này in ra polarity, subjectivity và các assessment thật của bốn câu ở mục 5, để đối chiếu với các điểm giả định. Đây là ví dụ cách gọi theo tài liệu [S1], chưa phải đánh giá chất lượng trên dữ liệu khách sạn.

<a id="muc-10"></a>

**10. Tài liệu tham khảo**

- **[S1]** TextBlob. [Quickstart: Sentiment Analysis](https://textblob.readthedocs.io/en/stable/quickstart.html).
- **[S2]** TextBlob. [Advanced Usage: Overriding Models](https://textblob.readthedocs.io/en/dev/advanced_usage.html).
- **[S3]** TextBlob. [Mã nguồn các sentiment analyzer](https://textblob.readthedocs.io/en/stable/_modules/textblob/en/sentiments.html).
- **[S4]** De Smedt, T., & Daelemans, W. (2012). *Pattern for Python*. [JMLR](https://www.jmlr.org/papers/v13/desmedt12a.html).

S4 giới thiệu Pattern, không phải công trình chứng minh TextBlob chấm tốt trên đánh giá khách sạn. Phần so sánh với SentiWordNet và VADER dựa trên hai file trước. Các ví dụ khách sạn, điểm giả định và kế hoạch kiểm tra ở trên là đề xuất của người biên soạn.
