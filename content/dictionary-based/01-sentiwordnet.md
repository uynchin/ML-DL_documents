# SentiWordNet: giải thích đơn giản

**Mục lục**

- [1. Ý chính trong vài câu](#muc-1)
- [2. Ai làm gì](#muc-2)
- [3. Một câu, làm từ đầu đến cuối](#muc-3)
- [4. Ba lỗi hay gặp và cách sửa](#muc-4)
- [5. Làm sao biết kết quả đáng tin](#muc-5)
- [6. Khi nào nên dùng](#muc-6)
- [7. Tài liệu tham khảo](#muc-7)

<a id="muc-1"></a>

**1. Ý chính trong vài câu**

SentiWordNet là một cuốn từ điển tiếng Anh có thêm điểm khen chê cho từng nghĩa của từ [S1] [S2].

Mỗi nghĩa có ba điểm cộng lại bằng 1: điểm khen, điểm chê và điểm trung lập. Điểm trung lập là phần không khen cũng không chê.

Điểm này gắn với nghĩa của từ, không gắn với mặt chữ, vì một từ có thể có nhiều nghĩa mà mỗi nghĩa khen chê khác nhau. Ví dụ với từ "cold" (điểm giả định):

| Nghĩa của "cold" | Câu ví dụ | Điểm khen | Điểm chê | Điểm trung lập |
|---|---|---|---|---|
| Lạnh về nhiệt độ | The soup was cold | 0.000 | 0.125 | 0.875 |
| Lạnh nhạt, thiếu thân thiện | The receptionist was cold | 0.000 | 0.625 | 0.375 |

Cùng chữ "cold" nhưng nghĩa thứ hai chê mạnh hơn nhiều. Vì vậy trước khi tra điểm phải biết từ đang mang nghĩa nào.

Cách dùng để chấm một câu rất gọn:

1. Với mỗi từ có cảm xúc, lấy điểm khen trừ điểm chê, gọi là điểm của từ.
2. Lấy trung bình điểm của các từ trong câu, gọi là điểm của câu.
3. Điểm dương nghĩa là câu nghiêng về khen, điểm âm nghĩa là nghiêng về chê, điểm gần 0 nghĩa là trung tính (hoặc chưa đủ thông tin).

SentiWordNet chỉ cung cấp bảng điểm. Các việc còn lại như tách câu, chọn nghĩa, xử lý chữ "not" là do người làm nghiên cứu tự thêm vào.

<a id="muc-2"></a>

**2. Ai làm gì**

Có hai bên: máy làm các việc lặp lại, người làm các việc cần phán đoán.

| Việc | Ai làm |
|---|---|
| Quyết định chấm theo từng câu, từng bài, hay từng khía cạnh (phòng, dịch vụ, bữa sáng) | Người |
| Đọc và gán nhãn khen, chê, trung tính cho một tập câu mẫu để làm đáp án | Người |
| Tách câu thành từng từ, đưa về dạng gốc (ví dụ "rooms" về "room") | Máy |
| Chọn nghĩa của từ (xem phần 4) | Máy làm, người kiểm tra một mẫu |
| Tra điểm khen chê của nghĩa đó | Máy |
| Viết quy tắc cho chữ "not" | Người viết, máy áp dụng |
| Tính điểm câu và đổi thành nhãn khen, chê, trung tính | Máy |
| Quyết định điểm gần 0 trong khoảng nào thì coi là trung tính | Người |
| So nhãn của máy với đáp án của người, tính tỷ lệ đúng | Máy tính, người đọc các câu sai |
| Kết luận trong bài viết | Người |

Có thể nhớ gọn như sau: máy tra và tính, người đặt luật và kiểm tra.

<a id="muc-3"></a>

**3. Một câu, làm từ đầu đến cuối**

Câu: "The staff were friendly and the breakfast was delicious."

Bước 1. Máy tách câu và tìm từ có cảm xúc: "friendly" và "delicious".

Bước 2. Máy tra điểm của hai từ (điểm giả định).

| Từ | Điểm khen | Điểm chê |
|---|---|---|
| friendly | 0.625 | 0.000 |
| delicious | 0.750 | 0.000 |

Bước 3. Máy lấy điểm khen trừ điểm chê cho từng từ.

| Từ | Phép tính | Điểm của từ |
|---|---|---|
| friendly | 0.625 trừ 0.000 | 0.625 |
| delicious | 0.750 trừ 0.000 | 0.750 |

Bước 4. Máy lấy trung bình hai điểm.

| Phép tính | Điểm của câu |
|---|---|
| (0.625 cộng 0.750) chia 2 | 0.6875 |

Bước 5. Máy so với khoảng trung tính do người đặt. Ở đây giả định khoảng trung tính là từ -0.1 đến 0.1. Điểm 0.6875 nằm ngoài khoảng và dương, nên nhãn là khen. Người đọc câu cũng thấy đó là lời khen, nên máy chấm đúng.

<a id="muc-4"></a>

**4. Ba lỗi hay gặp và cách sửa**

**Lỗi 1: câu vừa khen vừa chê bị bù trừ**

Câu: "The room was clean, but the service was slow."

| Từ | Điểm khen | Điểm chê | Điểm của từ (giả định) |
|---|---|---|---|
| clean | 0.625 | 0.125 | 0.500 |
| slow | 0.125 | 0.500 | -0.375 |

Trung bình hai điểm là 0.0625, nằm trong khoảng trung tính, nên máy kết luận trung tính. Nhưng câu khen phòng và chê dịch vụ, hai ý bù trừ nhau nên mất thông tin.

Cách sửa: chấm riêng từng đối tượng thay vì cả câu.

| Đối tượng | Từ cảm xúc | Điểm | Nhãn |
|---|---|---|---|
| room | clean | 0.500 | Khen |
| service | slow | -0.375 | Chê |

Người lập danh sách từ khóa cho từng đối tượng (room, service, breakfast), máy gắn mỗi từ cảm xúc với đối tượng gần nhất. Kết quả này cho biết cần cải thiện dịch vụ, còn một điểm trung bình thì không.

**Lỗi 2: chữ "not" làm đảo ý**

Câu: "The bed was not comfortable."

| Cách làm | Điểm của "comfortable" (giả định) | Nhãn |
|---|---|---|
| Chỉ tra từ, bỏ qua "not" | 0.375 | Khen (sai) |
| Có quy tắc đảo dấu khi từ đứng ngay sau "not" | -0.375 | Chê (đúng) |

Người viết quy tắc: nếu từ cảm xúc nằm trong 3 từ sau "not" thì đổi dấu điểm. Máy áp dụng. Quy tắc này vẫn có thể sai ở câu như "not only comfortable but also quiet", nên phải thử trên các câu mẫu trước khi dùng. Ngoài ra khi lọc bỏ các từ quá phổ biến (the, is, a), cần giữ lại "not", "no", "never", vì danh sách mặc định thường xóa chúng.

**Lỗi 3: chọn nhầm nghĩa**

Câu: "The receptionist was cold."

| Cách máy chọn nghĩa | Nghĩa được chọn | Điểm của từ (giả định) | Kết quả |
|---|---|---|---|
| Luôn lấy nghĩa đầu tiên trong từ điển | Lạnh về nhiệt độ | -0.125 | Gần như trung tính, bỏ sót lời phàn nàn |
| Chọn nghĩa theo cả câu | Lạnh nhạt, thiếu thân thiện | -0.625 | Chê rõ, đúng với câu |

Cách chọn nghĩa theo cả câu chạy chậm hơn và cũng có thể sai. Vì vậy người nên đọc thử khoảng 50 câu có từ nhiều nghĩa, đếm máy chọn đúng nghĩa bao nhiêu lần, rồi so hai cách chọn trên cùng tập câu.

Để xem một từ có những nghĩa nào và điểm của từng nghĩa, có thể chạy đoạn mã sau [S3]:

```python
import nltk
nltk.download("wordnet")
nltk.download("sentiwordnet")

from nltk.corpus import sentiwordnet as swn

for s in swn.senti_synsets("cold", "a"):
    print(s, s.pos_score(), s.neg_score(), s.obj_score())
```

Mã này chỉ liệt kê các nghĩa có sẵn, không tự chọn nghĩa đúng cho câu.

<a id="muc-5"></a>

**5. Làm sao biết kết quả đáng tin**

Chỉ chạy máy và đọc kết quả chưa đủ. Cần ba việc đơn giản, làm theo đúng thứ tự.

Bước 1. Người tạo đáp án.

| Việc | Cách làm |
|---|---|
| Chọn khoảng 100 câu mẫu | Lấy từ nhiều khách sạn khác nhau |
| Hai người gán nhãn độc lập | Mỗi người ghi khen, chê hoặc trung tính cho từng câu |
| Đo hai người đồng ý với nhau bao nhiêu | Máy tính hệ số kappa. Số càng gần 1 thì hai người càng đồng ý |
| Chỗ hai người khác nhau | Bàn lại và thống nhất một nhãn |

Nếu hai người còn không đồng ý với nhau thì đáp án chưa đủ chắc để chấm máy.

Bước 2. Chia đáp án làm hai phần.

| Phần | Dùng để làm gì |
|---|---|
| Phần thử nghiệm | Thử các cách chọn nghĩa, quy tắc "not", khoảng trung tính. Được sửa thoải mái |
| Phần chấm điểm cuối | Chỉ dùng đúng một lần ở cuối, không sửa gì sau khi thấy kết quả |

Nên chia theo khách sạn: câu của cùng một khách sạn chỉ nằm ở một phần. Nếu chia lẫn lộn thì kết quả sẽ đẹp hơn thực tế.

Bước 3. Máy so nhãn của máy với đáp án của người trên phần chấm điểm cuối, và người báo ba con số.

| Con số | Ý nghĩa đơn giản |
|---|---|
| Độ đúng trung bình của ba nhãn (macro-F1) | Máy đúng bao nhiêu, tính riêng cho khen, chê, trung tính rồi lấy trung bình, để nhãn ít câu không bị bỏ quên |
| Bảng nhầm lẫn | Máy hay nhầm nhãn nào thành nhãn nào |
| Tỷ lệ câu tra được ít nhất một từ (coverage) | Máy hiểu được bao nhiêu phần dữ liệu |

Khi thêm một thành phần mới, ví dụ quy tắc "not", hãy chạy lại trước và sau khi thêm trên phần thử nghiệm để biết nó thật sự giúp hay không.

Nếu dữ liệu có số sao của khách, có thể so điểm của máy với số sao. Điểm cao mà sao cũng cao thì thêm một dấu hiệu rằng máy đo đúng điều cần đo.

Một lưu ý khi viết kết luận: điểm của máy chỉ cho biết chữ viết nghiêng về khen hay chê. Muốn nói khách hài lòng hơn, hoặc việc khách sạn làm đã khiến khách hài lòng hơn, cần thêm bằng chứng ở mức tương ứng.

<a id="muc-6"></a>

**6. Khi nào nên dùng**

| Nên dùng | Không nên dùng |
|---|---|
| Đánh giá viết bằng tiếng Anh | Đánh giá tiếng Việt mà chưa kiểm tra riêng bước dịch |
| Cần cách chấm đơn giản, xem được từng từ đóng góp bao nhiêu | Văn bản nhiều mỉa mai, nhiều tiếng lóng, trộn ngôn ngữ |
| Cần một mốc so sánh trước khi thử mô hình phức tạp | Không có đáp án nào để kiểm tra |

Với tiếng Việt, nếu dịch sang tiếng Anh rồi mới chấm, lỗi dịch cộng thêm vào lỗi chấm, nên bước dịch cũng phải được kiểm tra bằng đáp án.

Khi viết vào bài, cần ghi rõ phiên bản SentiWordNet và NLTK, cách chọn nghĩa, quy tắc "not", khoảng trung tính, số câu trong phần chấm điểm cuối, kết quả ba con số ở phần 5 và hệ số kappa của hai người gán nhãn. Khi đưa lên GitHub, mã và ví dụ tự soạn chia sẻ được, còn dữ liệu và từ điển phải theo giấy phép của chúng.

<a id="muc-7"></a>

**7. Tài liệu tham khảo**

- **[S1]** Esuli, A., & Sebastiani, F. (2006). *SENTIWORDNET: A Publicly Available Lexical Resource for Opinion Mining*. [ACL Anthology](https://aclanthology.org/L06-1225/).
- **[S2]** Baccianella, S., Esuli, A., & Sebastiani, F. (2010). *SentiWordNet 3.0: An Enhanced Lexical Resource for Sentiment Analysis and Opinion Mining*. [ACL Anthology](https://aclanthology.org/L10-1531/).
- **[S3]** NLTK. *SentiWordNet interface*. [Tài liệu tra cứu](https://www.nltk.org/howto/sentiwordnet.html).

S1 và S2 giới thiệu tài nguyên, S3 là hướng dẫn dùng phần mềm. Các ví dụ khách sạn, cách phân công máy và người, và các bước kiểm tra ở trên là đề xuất của người biên soạn, không phải nghiên cứu của các tác giả này.
