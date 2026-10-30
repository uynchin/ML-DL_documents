# SentiWordNet — Từ điển cảm xúc theo nghĩa của từ

> Nhánh: LEXICON BASED → Dictionary Based. Tài liệu học và thiết kế nghiên cứu; đối chiếu nguồn ngày 28/10/2026. Ví dụ khách sạn và các số liệu ghi “giả định” do người biên soạn xây dựng, không phải kết quả thực nghiệm.

| Trường | Thông tin |
|---|---|
| Bản chất | Tài nguyên từ vựng theo synset, cần một quy trình bên ngoài để phân loại văn bản |
| Mốc lịch sử | Công bố ban đầu 2006; SentiWordNet 3.0 năm 2010 [S1–S2] |
| Đầu vào thực hành | Văn bản tiếng Anh; từ gốc, từ loại và nghĩa được chọn |
| Tiếng Việt | Không phải từ điển tiếng Việt trực tiếp; cần ánh xạ nghĩa hoặc dịch và kiểm định |
| Nhãn và tính toán | Tra cứu không cần huấn luyện trên nhãn của dự án; đánh giá vẫn cần nhãn chuẩn. CPU thường đủ cho tra cứu, bước chọn nghĩa có thể tốn hơn |

<details>
<summary>Mục lục 32 đề mục</summary>

- [1. Định nghĩa học thuật](#muc-1)
- [2. Giải thích đơn giản](#muc-2)
- [3. Mục đích và vấn đề giải quyết](#muc-3)
- [4. Điều kiện và giả định sử dụng](#muc-4)
- [5. Cơ chế hoạt động bên trong](#muc-5)
- [6. Quy trình áp dụng trong nghiên cứu](#muc-6)
- [7. Ví dụ minh họa xuyên suốt](#muc-7)
- [8. Kết quả và cách diễn giải](#muc-8)
- [9. Điểm mạnh, điểm yếu và giới hạn](#muc-9)
- [10. So sánh với phương pháp gần nhau](#muc-10)
- [11. Sai lầm thường gặp và cách khắc phục](#muc-11)
- [12. Tài liệu tham khảo](#muc-12)
- [13. Vị trí trong toàn bộ nghiên cứu](#muc-13)
- [14. Kiến thức nền cần biết trước](#muc-14)
- [15. Nền tảng lý thuyết và logic hình thành](#muc-15)
- [16. Các thành phần cấu tạo](#muc-16)
- [17. Các loại và biến thể](#muc-17)
- [18. Đầu vào và đầu ra](#muc-18)
- [19. Căn cứ lựa chọn](#muc-19)
- [20. Trường hợp không nên sử dụng](#muc-20)
- [21. Các quyết định quan trọng khi thực hiện](#muc-21)
- [22. Giải thích công thức và ký hiệu](#muc-22)
- [23. Ví dụ tính toán hoặc thao tác từng bước](#muc-23)
- [24. Tình huống sai và phản ví dụ](#muc-24)
- [25. Kiểm tra chất lượng và độ vững](#muc-25)
- [26. Xử lý khi không đáp ứng yêu cầu](#muc-26)
- [27. Phân biệt kết quả với kết luận](#muc-27)
- [28. Cách viết vào bài nghiên cứu](#muc-28)
- [29. Khả năng tái lập và tính minh bạch](#muc-29)
- [30. Đạo đức nghiên cứu liên quan](#muc-30)
- [31. Câu hỏi tự kiểm tra và bài tập vận dụng](#muc-31)
- [32. Những điểm còn tranh luận](#muc-32)

</details>

<a id="muc-1"></a>

## 1. Định nghĩa học thuật

SentiWordNet gắn ba điểm tích cực, tiêu cực và khách quan cho các **synset** của WordNet. Synset là nhóm từ đồng nghĩa biểu đạt một nghĩa cụ thể, không phải toàn bộ các nghĩa của một từ. SentiWordNet 3.0 gắn với WordNet 3.0 [S2]. Vì vậy, nói “mỗi từ có một điểm cảm xúc cố định” là mô tả chưa đúng tài nguyên này.

<a id="muc-2"></a>

## 2. Giải thích đơn giản

Hãy hình dung một từ điển mà mỗi mục giải nghĩa có thêm ba thanh đo. Khi gặp một từ, bạn phải mở đúng nghĩa đang được dùng rồi mới đọc thanh đo. Một từ có thể biểu đạt sự đánh giá trong câu này nhưng mang nghĩa kỹ thuật ở câu khác; chỉ nhìn mặt chữ sẽ dễ chọn nhầm.

<a id="muc-3"></a>

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu đánh giá khách sạn, có thể dùng tài nguyên này để xây dựng một mô hình cơ sở: nội dung có xu hướng khen hay chê theo một quy tắc từ vựng minh bạch? Một ứng dụng khác là tạo đặc trưng cảm xúc cho mô hình học máy. Đây là công cụ đo biểu hiện trong ngôn ngữ; việc đo sự hài lòng của khách còn cần căn cứ về giá trị đo lường.

<a id="muc-4"></a>

## 4. Điều kiện và giả định sử dụng

Điều kiện quan trọng là nhận diện đúng từ, từ loại và nghĩa. Đề xuất kiểm tra trước một mẫu đánh giá tiếng Anh thuộc đúng phân khúc khách sạn. Giả định cần đánh giá là sắc thái của nghĩa trong từ điển còn phù hợp với miền dữ liệu và quy tắc tổng hợp giữ được ý kiến của câu. Không có mức bao phủ tối thiểu phổ quát bảo đảm hiệu quả.

<a id="muc-5"></a>

## 5. Cơ chế hoạt động bên trong

Cần tách **xây tài nguyên** và **dùng tài nguyên**. Công trình 3.0 mô tả gán điểm tự động bằng bước học bán giám sát và tinh chỉnh qua random walk [S2]. Khi ứng dụng, quy trình thường là: tách từ → xác định từ loại → đưa về từ gốc → chọn nghĩa → tra điểm → xử lý ngữ cảnh → tổng hợp. Các bước sau tra cứu là lựa chọn của hệ thống nghiên cứu, không phải toàn bộ đều có sẵn trong SentiWordNet.

<a id="muc-6"></a>

## 6. Quy trình áp dụng trong nghiên cứu

1. Định nghĩa đơn vị phân tích: câu, đánh giá hay cặp khía cạnh–ý kiến.
2. Lấy mẫu, loại trùng và giữ mã khách sạn để nhận diện dữ liệu phụ thuộc.
3. Xây hướng dẫn gán nhãn; tách tập phát triển và tập kiểm tra.
4. Cố định cách chọn nghĩa, xử lý phủ định và tổng hợp điểm.
5. Chọn ngưỡng trên tập phát triển; đánh giá một lần trên tập kiểm tra.
6. Báo cáo lỗi theo từ đa nghĩa, câu hỗn hợp và mức bao phủ.

Nếu mục tiêu là áp dụng cho khách sạn mới, nên chia dữ liệu theo khách sạn thay vì chỉ chia ngẫu nhiên từng câu.

<a id="muc-7"></a>

## 7. Ví dụ minh họa xuyên suốt

Xét câu tự soạn: “The room was clean, but the service was slow.” Phòng và dịch vụ là hai đối tượng được đánh giá khác nhau. Ta cần xác định nghĩa của “clean”, “slow”, rồi giữ liên kết với từng khía cạnh. Nếu chỉ cộng thành một điểm toàn câu, lời khen và lời chê có thể triệt tiêu nhau. Bản thân việc tra từ điển chưa thực hiện bước liên kết khía cạnh này.

<a id="muc-8"></a>

## 8. Kết quả và cách diễn giải

Đầu ra trung gian là điểm của từng nghĩa; đầu ra cuối phụ thuộc cách bạn xây bộ chấm điểm. Điểm tổng hợp dương nghĩa là quy tắc đang dùng ghi nhận nhiều bằng chứng tích cực hơn tiêu cực. Nó không phải xác suất khách hài lòng. Một điểm gần 0 cần đọc kèm độ bao phủ và hai thành phần dương–âm để phân biệt trung tính, hỗn hợp và thiếu bằng chứng.

<a id="muc-9"></a>

## 9. Điểm mạnh, điểm yếu và giới hạn

**Mạnh:** có cấu trúc theo nghĩa, dễ truy ngược đóng góp và không đòi hỏi huấn luyện bộ phân loại trên dữ liệu dự án. **Yếu:** chọn nghĩa sai sẽ truyền sai số xuống toàn quy trình; cộng điểm có thể bỏ qua cấu trúc câu. Với nghiên cứu khách sạn, “small” có thể là lời chê phòng hoặc mô tả trung tính. Cần kiểm tra ngữ cảnh thay vì mặc định một sắc thái cho mọi lần xuất hiện.

<a id="muc-10"></a>

## 10. So sánh với phương pháp gần nhau

| Đối chiếu | Khác biệt cần nhớ |
|---|---|
| WordNet | Tổ chức nghĩa và quan hệ từ vựng; SentiWordNet bổ sung điểm cảm xúc |
| AFINN | Thường tra điểm theo từ/cụm; SentiWordNet nhấn vào nghĩa của từ |
| VADER | Có hệ quy tắc chấm văn bản; SentiWordNet chủ yếu cung cấp tài nguyên |

Chọn SentiWordNet khi việc xử lý đa nghĩa là một phần câu hỏi hoặc thiết kế phương pháp, không chỉ vì nó có nhiều mục từ.

<a id="muc-11"></a>

## 11. Sai lầm thường gặp và cách khắc phục

Lỗi dễ gặp là lấy nghĩa đầu tiên như thể luôn đúng. Hãy gọi đó là **heuristic nghĩa đầu tiên** và so sánh với cách khác. Một lỗi nữa là xóa “not” khi bỏ stopword: nên bảo toàn phủ định. Ngoài ra, không ghi điểm 0 cho mọi mục không tìm thấy; cần một trạng thái thiếu dữ liệu riêng để tránh gọi văn bản chưa được hiểu là trung tính.

<a id="muc-12"></a>

## 12. Tài liệu tham khảo

- **[S1]** Esuli, A., & Sebastiani, F. (2006). *SENTIWORDNET: A Publicly Available Lexical Resource for Opinion Mining*. [ACL Anthology](https://aclanthology.org/L06-1225/).
- **[S2]** Baccianella, S., Esuli, A., & Sebastiani, F. (2010). *SentiWordNet 3.0: An Enhanced Lexical Resource for Sentiment Analysis and Opinion Mining*. [ACL Anthology](https://aclanthology.org/L10-1531/).
- **[S3]** NLTK. *SentiWordNet interface*. [Tài liệu tra cứu](https://www.nltk.org/howto/sentiwordnet.html).

S1–S2 là công trình giới thiệu tài nguyên; S3 là hướng dẫn phần mềm. Các quy trình kiểm định và ví dụ khách sạn trong tài liệu này là đề xuất thiết kế, không phải nghiên cứu khách sạn đã được các tác giả trên thực hiện.

<a id="muc-13"></a>

## 13. Vị trí trong toàn bộ nghiên cứu

SentiWordNet nằm ở giai đoạn biến văn bản thành biến quan sát được. Đầu ra của nó có thể đi vào thống kê mô tả hoặc mô hình giải thích, nhưng trước đó phải được kiểm tra chất lượng. Ví dụ, nếu dùng điểm cảm xúc làm biến phụ thuộc của mô hình chất lượng dịch vụ, sai số chọn nghĩa có thể ảnh hưởng kết luận của bước phân tích tiếp theo.

<a id="muc-14"></a>

## 14. Kiến thức nền cần biết trước

Nên hiểu token là đơn vị do bộ tách từ tạo ra; lemma là dạng từ gốc; POS là từ loại; synset là một nghĩa được WordNet định danh; WSD là chọn nghĩa theo ngữ cảnh. Cũng cần phân biệt **cực tính** với **cảm xúc cụ thể**: dương/âm không tự cho biết người viết đang vui, giận hay sợ.

<a id="muc-15"></a>

## 15. Nền tảng lý thuyết và logic hình thành

Logic ứng dụng là: nghĩa từ cung cấp tín hiệu đánh giá, nên có thể dùng tập hợp các tín hiệu để ước lượng sắc thái văn bản. Tuy nhiên, từ có sắc thái sẵn và câu có ý nghĩa cảm xúc là hai cấp khác nhau. Nghiên cứu cần kiểm chứng cầu nối giữa chúng; cấu trúc “không hề tốt” cho thấy điểm từ đơn không đủ.

<a id="muc-16"></a>

## 16. Các thành phần cấu tạo

Một hệ thống thực hành cần: tài nguyên nghĩa, tài nguyên điểm, bộ tiền xử lý, bộ chọn nghĩa, bộ xử lý ngữ cảnh, bộ tổng hợp và bộ chuyển điểm thành nhãn. Tách các phần này trong báo cáo giúp xác định cải thiện đến từ tài nguyên hay từ quy tắc do nhóm nghiên cứu thêm vào. NLTK cung cấp giao diện lấy các thành phần điểm [S3].

<a id="muc-17"></a>

## 17. Các loại và biến thể

Có thể thiết kế ba phương án ứng dụng: dùng nghĩa đầu tiên; trung bình điểm các nghĩa phù hợp từ loại; hoặc chọn nghĩa theo ngữ cảnh. Đây là ba **cách dùng** khác nhau, không phải ba phiên bản chính thức của SentiWordNet. Khi so sánh, cần giữ dữ liệu và quy tắc tổng hợp giống nhau để tách ảnh hưởng của bước chọn nghĩa.

<a id="muc-18"></a>

## 18. Đầu vào và đầu ra

Đề xuất lưu mỗi đơn vị bằng các trường `review_id`, `sentence_id`, `token`, `lemma`, `pos`, `synset`, `pos_score`, `neg_score`, `matched`. Khi có phân tích khía cạnh, thêm `aspect_id`. Đầu ra báo cáo nên giữ cả điểm và mức bao phủ; chỉ lưu nhãn cuối khiến việc kiểm tra lỗi chọn nghĩa khó hơn.

<a id="muc-19"></a>

## 19. Căn cứ lựa chọn

Cân nhắc ba câu hỏi: dữ liệu có đủ khớp tài nguyên không; bạn có thể kiểm tra cách chọn nghĩa không; mục tiêu có cần một mô hình dễ giải thích không? Thử trên cùng mẫu gán nhãn với một mô hình cơ sở khác. Nếu chi phí chọn nghĩa tăng nhưng chất lượng không cải thiện, nên cân nhắc dùng cách đơn giản hơn trong nghiên cứu chính.

<a id="muc-20"></a>

## 20. Trường hợp không nên sử dụng

Không nên dùng tra cứu tiếng Anh trực tiếp để kết luận về đánh giá tiếng Việt. Với mỉa mai dày đặc, ngôn ngữ trộn hoặc yêu cầu hiểu chính xác từng khía cạnh, tra từ điển đơn thuần khó đáp ứng. Có thể thử mô hình ngữ cảnh hoặc thiết kế lai, nhưng vẫn phải đo chất lượng trên đúng dữ liệu mục tiêu.

<a id="muc-21"></a>

## 21. Các quyết định quan trọng khi thực hiện

Phải quyết định cách xử lý nghĩa không chắc, token không khớp, câu nhiều ý kiến và điểm bằng 0. Chẳng hạn, loại câu không khớp khỏi phân tích sẽ làm thay đổi quần thể đang mô tả. Vì vậy cần báo tỷ lệ bị loại, đặc điểm nhóm bị loại và mức thay đổi kết luận khi giữ chúng như nhóm “chưa đủ bằng chứng”.

<a id="muc-22"></a>

## 22. Giải thích công thức và ký hiệu

Với synset được chọn cho đơn vị thứ $i$:

$$p_i+n_i+o_i=1,\qquad v_i=p_i-n_i.$$

$p_i,n_i,o_i$ lần lượt là ba điểm của nghĩa; $v_i$ là điểm chênh lệch do ta suy ra. Một quy tắc tổng hợp **đề xuất**, không phải công thức phân loại bắt buộc của tài nguyên, là:

$$S(d)=\frac{1}{m}\sum_{i\in M(d)}v_i.$$

$M(d)$ là các đơn vị khớp được giữ lại, $m=|M(d)|$. Nếu $m=0$, trả về thiếu bằng chứng. Nhãn trung tính có thể dùng khoảng $[-\tau,\tau]$; $\tau$ cần lựa chọn trên tập phát triển.

<a id="muc-23"></a>

## 23. Ví dụ tính toán hoặc thao tác từng bước

Giả định hai nghĩa đã chọn có $(p,n,o)$ lần lượt là $(0.6,0.1,0.3)$ và $(0.1,0.5,0.4)$. Khi đó $v_1=0.5$, $v_2=-0.4$, điểm trung bình là $0.05$. Đây là số tự đặt để học công thức, không phải điểm thực của “clean” hay “slow”. Điểm thấp này che khuất hai ý kiến trái chiều.

Ví dụ API theo NLTK [S3], cần cài NLTK và tải corpora `wordnet`, `sentiwordnet` trước khi chạy:

```python
from nltk.corpus import sentiwordnet as swn
s = swn.senti_synset("breakdown.n.03")
print(s.pos_score(), s.neg_score(), s.obj_score())
```

Đoạn này chỉ tra một nghĩa định danh sẵn; không phải bộ chọn nghĩa tự động cho câu.

<a id="muc-24"></a>

## 24. Tình huống sai và phản ví dụ

“The hotel is not good” có thể vẫn bị chấm dương nếu chỉ tra “good”. Đảo dấu trong phạm vi phủ định là một phương án sửa cần kiểm định; không nên đảo mọi từ phía sau “not”. Cấu trúc “not only good but also affordable” cho thấy phạm vi phủ định và liên từ có thể làm một quy tắc quá đơn giản tạo lỗi mới.

<a id="muc-25"></a>

## 25. Kiểm tra chất lượng và độ vững

Đề xuất đo macro-F1 cho phân loại ba lớp, ma trận nhầm lẫn và coverage. Kiểm tra độ nhạy khi đổi cách chọn nghĩa và cách tổng hợp. Nếu có nhiều câu từ cùng khách sạn, bootstrap theo khách sạn phù hợp hơn việc xem mọi câu là độc lập. Một lần chạy quyết định thường không cần lặp nhiều seed; cần biến thiên do lấy mẫu và thiết kế.

<a id="muc-26"></a>

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu coverage thấp, kiểm tra bộ tách từ và lemma trước khi bổ sung từ. Nếu lỗi tập trung ở đa nghĩa, xem lại WSD. Nếu lỗi ở ý kiến hỗn hợp, chuyển đơn vị phân tích sang câu hoặc khía cạnh. Nếu mở rộng từ điển theo miền, thực hiện trên tập phát triển và công bố phần mở rộng riêng; không sửa theo lỗi của tập kiểm tra rồi báo lại như kết quả độc lập.

<a id="muc-27"></a>

## 27. Phân biệt kết quả với kết luận

“Điểm từ vựng trung bình tăng” là kết quả của một phép đo. “Khách hài lòng hơn” cần bằng chứng đo lường bổ sung. “Can thiệp của khách sạn làm khách hài lòng hơn” còn cần thiết kế nhận diện nhân quả. So sánh trước–sau đơn thuần có thể bị ảnh hưởng bởi mùa du lịch, thành phần khách và thay đổi nền tảng đánh giá.

<a id="muc-28"></a>

## 28. Cách viết vào bài nghiên cứu

Mẫu cần điền bằng thông tin thật: “Chúng tôi dùng SentiWordNet [phiên bản], chọn nghĩa bằng [cách làm], tổng hợp tại cấp [đơn vị]. Ngưỡng được xác định trên [tập phát triển]. Chất lượng đo được trên tập kiểm tra độc lập gồm [n] đơn vị là [chỉ số], với coverage [x].” Phần thảo luận cần nêu giới hạn từ đa nghĩa và ngữ cảnh, thay vì chỉ liệt kê tên tài nguyên.

<a id="muc-29"></a>

## 29. Khả năng tái lập và tính minh bạch

Lưu phiên bản WordNet/SentiWordNet, NLTK, mã chọn nghĩa, quy tắc phủ định, ngưỡng, mã chia tập và checksum dữ liệu được phép lưu. Khi đưa lên GitHub, có thể chia sẻ mã và ví dụ tự soạn; việc phân phối lại dữ liệu/tài nguyên phải theo giấy phép tương ứng. Tài liệu này không kèm bản sao từ điển.

<a id="muc-30"></a>

## 30. Đạo đức nghiên cứu liên quan

Ẩn thông tin nhận diện của khách và nhân viên trong ví dụ công khai. Điểm chê về dịch vụ không đủ để kết luận một nhân viên có năng lực kém. Nếu so sánh nhóm ngôn ngữ, cần kiểm tra chất lượng riêng; khác biệt tỷ lệ lỗi có thể tạo ra khác biệt điểm cảm xúc giả giữa các nhóm.

<a id="muc-31"></a>

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Vì sao một từ có nhiều synset? **Gợi ý:** khác nghĩa hoặc từ loại.
2. Hai câu có cùng điểm 0 có chắc cùng trạng thái không? **Không:** có thể hỗn hợp hoặc không khớp tài nguyên.
3. Bài tập: so sánh hai cách chọn nghĩa trên cùng 30 câu tự gán nhãn, ghi từng lỗi và giải thích chúng. Mẫu này để học và thử quy trình, chưa đủ để khẳng định hiệu quả tổng quát.

<a id="muc-32"></a>

## 32. Những điểm còn tranh luận

Vấn đề cần cân nhắc là lợi ích của phân biệt nghĩa có bù được sai số và chi phí WSD hay không. Ngoài ra, phép cộng các sắc thái từ vựng có thể giải thích tốt đóng góp của từ nhưng chưa giải thích đầy đủ ý nghĩa của cả phát ngôn. Cách giải quyết trong một bài nghiên cứu là nêu rõ mức diễn giải đang hướng tới và kiểm tra bằng dữ liệu, thay vì coi tính minh bạch của từng điểm là bảo đảm cho kết luận cuối.
