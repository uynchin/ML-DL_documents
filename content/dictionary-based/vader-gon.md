# VADER: giải thích đơn giản

**Mục lục**

- [1. VADER là gì](#muc-1)
- [2. So sánh VADER với SentiWordNet](#muc-2)
- [3. Ai làm gì](#muc-3)
- [4. VADER chấm điểm thế nào](#muc-4)
- [5. Ví dụ bốn câu](#muc-5)
- [6. Lỗi hay gặp](#muc-6)
- [7. Làm sao biết kết quả đáng tin](#muc-7)
- [8. Chọn VADER hay SentiWordNet](#muc-8)
- [9. Mã chạy thử](#muc-9)
- [10. Tài liệu tham khảo](#muc-10)

<a id="muc-1"></a>

**1. VADER là gì**

VADER là một chương trình chấm văn bản tiếng Anh xem nghiêng về khen hay chê [S1].

Bên trong có hai thứ. Thứ nhất là một từ điển, mỗi từ đã có sẵn một điểm khen chê. Thứ hai là một vài quy tắc nhìn cách viết câu, ví dụ có chữ "not", có chữ "very", có dấu chấm than hay không.

Nhờ các quy tắc này, "good", "VERY good!!!" và "not good" cho ba điểm khác nhau, dù cùng có chữ "good".

Chạy xong, máy cho sẵn bốn con số. Người dùng chủ yếu nhìn một con số.

| Con số | Ý nghĩa |
|---|---|
| compound | Điểm tổng, từ -1 (chê hết mức) đến 1 (khen hết mức). Đây là con số dùng để quyết định khen hay chê |
| pos | Phần khen trong câu |
| neg | Phần chê trong câu |
| neu | Phần trung lập trong câu |

Ba con số pos, neg, neu chỉ là tỷ lệ, không phải xác suất. Điểm compound 0.8 không có nghĩa 80% khách hài lòng.

<a id="muc-2"></a>

**2. So sánh VADER với SentiWordNet**

Cả hai đều dùng từ điển điểm cho tiếng Anh, nhưng cách làm khác nhau. Bảng sau so sánh theo từng điểm.

| Điểm so sánh | SentiWordNet | VADER |
|---|---|---|
| Điểm trong từ điển tính theo | Từng nghĩa của từ. Một từ có nhiều điểm, mỗi nghĩa một điểm | Từng từ. Một từ chỉ có một điểm |
| Ai cho điểm trong từ điển | Chương trình của tác giả, tính tự động [S4] | Theo bài gốc, nhiều người đánh giá từng từ rồi lấy trung bình [S1] |
| Có phải chọn nghĩa không | Có, và chọn sai thì điểm sai | Không cần |
| Chữ "not" | Không xử lý. Người phải tự viết quy tắc | Có sẵn |
| Chữ viết hoa, dấu chấm than, "very", "but" | Không xử lý | Có sẵn |
| Kết quả cuối | Người tự quyết định cách gộp điểm | Máy ra sẵn điểm tổng compound |
| Việc người phải làm thêm | Nhiều | Ít hơn |

Để thấy rõ khác biệt, thử cùng một câu với hai cách làm. Các con số trong bảng dưới là giả định.

| Câu | SentiWordNet | VADER |
|---|---|---|
| The bed was not comfortable. | Nếu chỉ tra "comfortable" thì ra điểm khen, sai. Người phải viết thêm quy tắc đảo dấu sau "not" thì mới ra chê | Tự nhận ra "not", ra điểm chê, không cần viết thêm |
| The room was VERY good!!! | Chỉ thấy chữ "good", không phân biệt với "good" thường | Điểm cao hơn "good" thường vì có "VERY" và dấu chấm than |
| The room was clean, but the service was slow. | Nếu cộng đơn giản thì khen và chê bù nhau, ra gần 0 (ví dụ 0.06) | Chữ "but" làm vế sau quan trọng hơn, nên điểm nghiêng về chê |
| The receptionist was cold. | Chọn đúng nghĩa "lạnh nhạt" thì ra điểm chê mạnh (ví dụ -0.625). Chọn nhầm nghĩa nhiệt độ thì ra gần trung tính (ví dụ -0.125) | Chữ "cold" chỉ có một điểm, nên không phải chọn nghĩa, nhưng cũng không tách được hai nghĩa. Cần chạy mã xem ra bao nhiêu |

Điểm chung: cả hai đều cho một điểm cho cả câu, nên đều chưa nói được khen cái gì, chê cái gì. Muốn biết phòng được khen còn dịch vụ bị chê, vẫn phải chấm riêng từng khía cạnh.

<a id="muc-3"></a>

**3. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Quyết định chấm từng câu hay cả bài đánh giá | Người |
| Giữ nguyên dấu chấm than, chữ viết hoa và chữ "not" trong văn bản đưa vào | Người |
| Gán nhãn khen, chê, trung tính cho một tập câu mẫu làm đáp án | Người |
| Tra điểm từng từ | Máy |
| Áp các quy tắc về "not", "very", dấu chấm than, chữ hoa, "but" | Máy |
| Tính điểm tổng compound | Máy |
| Đổi điểm tổng thành nhãn khen, chê, trung tính | Máy làm, người quyết định mức ngưỡng |
| So nhãn của máy với đáp án của người | Máy tính, người đọc các câu sai |
| Kết luận trong bài viết | Người |

Nhớ gọn: máy chấm điểm, người giữ nguyên văn bản, đặt ngưỡng và kiểm tra máy.

<a id="muc-4"></a>

**4. VADER chấm điểm thế nào**

Máy làm ba việc.

1. Tra điểm của từng từ trong từ điển.
2. Điều chỉnh điểm theo cách viết, theo bảng dưới.
3. Cộng lại và đổi thành điểm tổng compound nằm trong khoảng từ -1 đến 1 [S3].

| Cách viết | Ví dụ | Điểm thay đổi thế nào |
|---|---|---|
| Từ tăng mức độ | "very good" | Mạnh hơn "good" |
| Từ giảm mức độ | "slightly good" | Yếu hơn "good" |
| Viết hoa toàn bộ | "VERY good" | Mạnh hơn |
| Dấu chấm than | "good!!!" | Mạnh hơn |
| Có "not" ngay trước | "not good" | Đảo sang chê và yếu đi một chút |
| Có "but" | "good, but terrible" | Vế sau "but" được tính nặng hơn vế trước |

Sau đó đổi điểm tổng thành nhãn bằng ngưỡng. README của tác giả đề nghị mức sau để bắt đầu [S2].

| Điểm tổng | Nhãn |
|---|---|
| Từ 0.05 trở lên | Khen |
| Từ -0.05 trở xuống | Chê |
| Ở giữa | Trung tính |

Mức 0.05 chỉ là điểm bắt đầu, chưa chắc hợp với dữ liệu khách sạn. Nếu muốn đổi thì chỉ đổi khi thử trên phần thử nghiệm ở mục 7.

<a id="muc-5"></a>

**5. Ví dụ bốn câu**

Các điểm trong bảng là giả định để minh họa. Điểm thật phải chạy mã ở mục 9.

| Câu | Điều gì xảy ra | Điểm tổng (giả định) | Nhãn |
|---|---|---|---|
| The room was good. | Chỉ có từ "good", không có quy tắc nào | 0.44 | Khen |
| The room was VERY good!!! | "VERY" viết hoa và ba dấu chấm than làm điểm mạnh lên | 0.59 | Khen, mạnh hơn câu trên |
| The room was not good. | "not" đảo điểm của "good" sang chê | -0.35 | Chê |
| The room was good, but the service was terrible. | "terrible" ở sau "but" nên được tính nặng hơn "good" | -0.49 | Chê |

Câu cuối cho kết quả chê hợp lý, nhưng vẫn là một điểm cho cả câu. Phòng được khen thì điểm này không cho biết.

<a id="muc-6"></a>

**6. Lỗi hay gặp**

1. Xóa dấu chấm than và đổi sang chữ thường trước khi chạy. VADER dùng các dấu hiệu này, nên xóa đi là mất thông tin. Giữ nguyên văn bản gốc.
2. Mỉa mai. Câu "A wonderful place if you enjoy sleepless nights" có chữ "wonderful" nên máy dễ chấm khen, trong khi ý là chê. Máy chỉ làm theo quy tắc, không hiểu tình huống. Cách xử lý là người đọc riêng loại câu này và đếm máy sai bao nhiêu câu.
3. Dùng cho tiếng Việt. Từ điển là tiếng Anh, nên đưa tiếng Việt vào thì kết quả không đáng tin. Nếu dịch sang tiếng Anh rồi mới chấm thì lỗi dịch cộng thêm vào lỗi chấm, nên phải kiểm tra cả hai.
4. Dùng nhãn VADER làm đáp án. Nếu một mô hình khác học theo nhãn VADER rồi báo độ đúng so với nhãn VADER, thì đó là độ giống VADER, không phải độ đúng. Đáp án phải do người gán.

<a id="muc-7"></a>

**7. Làm sao biết kết quả đáng tin**

Kiểm tra theo ba bước.

Bước 1. Người tạo đáp án.

1. Chọn khoảng 100 câu từ nhiều khách sạn khác nhau.
2. Hai người gán nhãn khen, chê hoặc trung tính, mỗi người làm riêng.
3. Máy tính mức hai người đồng ý với nhau, gọi là hệ số kappa. Càng gần 1 thì càng đồng ý.
4. Chỗ hai người khác nhau thì bàn lại và chọn một nhãn.

Bước 2. Chia đáp án làm hai phần.

| Phần | Dùng để làm gì |
|---|---|
| Phần thử nghiệm | Thử ngưỡng khác 0.05, thử giữ hay bỏ dấu câu. Được sửa thoải mái |
| Phần chấm điểm cuối | Chỉ dùng một lần ở cuối, không sửa gì sau khi thấy kết quả |

Chia theo khách sạn: câu của cùng một khách sạn chỉ nằm ở một phần.

Bước 3. Máy so nhãn của VADER với đáp án của người trên phần chấm điểm cuối, rồi người báo kết quả.

| Con số | Ý nghĩa đơn giản |
|---|---|
| Độ đúng trung bình của ba nhãn (macro-F1) | Máy đúng bao nhiêu, tính riêng cho khen, chê, trung tính rồi lấy trung bình |
| Bảng nhầm lẫn | Máy hay nhầm nhãn nào thành nhãn nào |

Để so VADER với SentiWordNet cho công bằng, chạy cả hai trên cùng phần chấm điểm cuối và so hai con số macro-F1. Không dùng kết quả trong bài gốc làm kết quả của nghiên cứu, vì bài gốc đo trên văn bản mạng xã hội [S1], còn khách sạn phải tự đo lại.

Khi viết kết luận, nhãn của máy chỉ cho biết chữ viết nghiêng về khen hay chê. Muốn nói khách hài lòng hơn thì cần thêm bằng chứng, ví dụ đối chiếu với số sao của khách nếu dữ liệu có.

<a id="muc-8"></a>

**8. Chọn VADER hay SentiWordNet**

| Chọn | Khi nào |
|---|---|
| VADER | Cần chạy nhanh, ít việc thêm. Văn bản có dấu chấm than, chữ viết hoa, "not", "very" |
| SentiWordNet | Muốn thấy rõ điểm của từng nghĩa. Muốn tự kiểm soát cách chọn nghĩa và cách gộp điểm |
| Cả hai | Muốn so sánh hai cách làm trên cùng phần chấm điểm cuối |

Cả hai đều chỉ dùng cho tiếng Anh và đều chưa chấm riêng từng khía cạnh.

<a id="muc-9"></a>

**9. Mã chạy thử**

Cài gói bằng lệnh `pip install vaderSentiment`, rồi chạy:

```python
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer

analyzer = SentimentIntensityAnalyzer()
texts = [
    "The room was good.",
    "The room was VERY good!!!",
    "The room was not good.",
    "The room was good, but the service was terrible.",
]
for text in texts:
    scores = analyzer.polarity_scores(text)
    c = scores["compound"]
    label = "khen" if c >= 0.05 else "chê" if c <= -0.05 else "trung tính"
    print(text, scores, label)
```

Mã này in ra điểm thật của VADER cho bốn câu ở mục 5, để đối chiếu với các điểm giả định.

<a id="muc-10"></a>

**10. Tài liệu tham khảo**

- **[S1]** Hutto, C. J., & Gilbert, E. (2014). *VADER: A Parsimonious Rule-Based Model for Sentiment Analysis of Social Media Text*. [Bài báo ICWSM](https://ojs.aaai.org/index.php/ICWSM/article/view/14550).
- **[S2]** Tác giả VADER. [Kho mã và README](https://github.com/cjhutto/vaderSentiment).
- **[S3]** Tác giả VADER. [Mã triển khai](https://github.com/cjhutto/vaderSentiment/blob/master/vaderSentiment/vaderSentiment.py).
- **[S4]** Baccianella, S., Esuli, A., & Sebastiani, F. (2010). *SentiWordNet 3.0*. [ACL Anthology](https://aclanthology.org/L10-1531/). Dùng cho phần so sánh ở mục 2, tài liệu SentiWordNet đi kèm có ghi chi tiết.

Các ví dụ khách sạn, điểm giả định và kế hoạch kiểm tra ở trên là đề xuất của người biên soạn.
