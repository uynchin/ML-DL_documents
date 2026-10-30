# NRC Emotion Lexicon — Từ ngữ liên tưởng đến cảm xúc

> Nhánh: LEXICON BASED → Dictionary Based. Bài này nói về **NRC Word–Emotion Association Lexicon / EmoLex**, không gộp với NRC VAD hay NRC Emotion Intensity. Đối chiếu nguồn ngày 28/10/2026. Ví dụ khách sạn và từ điển nhỏ minh họa được tự xây dựng.

| Trường | Thông tin |
|---|---|
| Bản chất | Tài nguyên liên tưởng giữa từ và nhóm cảm xúc, không phải bộ hiểu đầy đủ cảm xúc người viết |
| Mốc | Bài tiền thân 2010; bản phát hành công khai 2011; bài crowdsourcing 2013 [S1–S2] |
| Đầu vào | Từ/văn bản sau tách từ phù hợp ngôn ngữ |
| Tiếng Việt | Có bản dịch tự động trong tài nguyên đa ngôn ngữ; cần kiểm định bản địa [S1] |
| Nhãn, tính toán | Không cần nhãn huấn luyện dự án để đếm; cần nhãn đánh giá. Tra cứu/đếm thông thường dùng CPU |
| Nguồn tải | [Trang tác giả](https://saifmohammad.com/WebPages/NRC-Emotion-Lexicon.htm); không cần một kho GitHub không chính thức để thay thế nguồn gốc |

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

NRC Emotion Lexicon liên kết từ với tám nhóm cảm xúc — anger, anticipation, disgust, fear, joy, sadness, surprise, trust — và hai chiều sentiment positive/negative. Trong biểu diễn từ–nhãn phổ biến, quan hệ được mã hóa 0 hoặc 1 và một từ có thể thuộc nhiều nhóm [S1]. Đây là **liên tưởng của từ**, không tự động là xác suất cảm xúc của người viết.

<a id="muc-2"></a>

## 2. Giải thích đơn giản

Hãy tưởng tượng mỗi từ có thể được gắn nhiều nhãn màu. Khi đọc một văn bản, ta đếm xem những màu nào xuất hiện nhiều. Một từ có thể có hai màu, nên tổng số nhãn có thể lớn hơn số từ. Cách này cho một hồ sơ ngôn ngữ, nhưng chưa đủ để nói chắc người viết thực sự cảm thấy điều gì.

<a id="muc-3"></a>

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu khách sạn, có thể khảo sát phản hồi về an toàn thường chứa từ liên tưởng sợ hãi hay không, hoặc ngôn ngữ liên tưởng niềm vui xuất hiện thế nào trong trải nghiệm nghỉ dưỡng. Câu hỏi nên được đặt ở cấp **biểu hiện ngôn ngữ** trước; muốn kết luận trạng thái cảm xúc cần thiết kế kiểm định bổ sung.

<a id="muc-4"></a>

## 4. Điều kiện và giả định sử dụng

Đề xuất kiểm tra từ điển có bao phủ cách diễn đạt của tập khách đang xét không. Với tiếng Việt, phải xác định cách tách từ nhiều âm tiết và kiểm tra bản dịch. Không nên coi bản dịch tự động là thang đo đã được xác nhận về văn hóa. Từ liên tưởng “fear” xuất hiện trong câu phủ định cũng không bảo đảm câu đang biểu lộ sợ hãi.

<a id="muc-5"></a>

## 5. Cơ chế hoạt động bên trong

Tài nguyên gốc được xây bằng gán nhãn cộng đồng; biểu diễn chuẩn cho phép tra quan hệ từ–nhãn [S1–S2]. Quy trình ứng dụng do người nghiên cứu xây thường là: tách đơn vị → khớp từ → lấy vector nhãn → cộng/chuẩn hóa → diễn giải. Đếm từ không tự giải quyết phủ định, trích dẫn hoặc người nào đang được nói đến.

<a id="muc-6"></a>

## 6. Quy trình áp dụng trong nghiên cứu

1. Chọn rõ đang đo liên tưởng từ hay biểu hiện cảm xúc của câu.
2. Chọn phiên bản, ngôn ngữ và kiểm tra quyền sử dụng.
3. Chuẩn bị quy tắc tách từ; gán nhãn độc lập nếu đánh giá cảm xúc văn bản.
4. Tính vector đếm, độ bao phủ và mẫu số chuẩn hóa.
5. Kiểm tra bằng tay các từ đóng góp nhiều nhất và các câu phủ định.
6. So sánh nhóm với kiểm soát độ dài, cấu trúc mẫu và nhiều phép kiểm định.

<a id="muc-7"></a>

## 7. Ví dụ minh họa xuyên suốt

Xét “We felt safe and happy during our stay” và “We were not happy during our stay”. Cả hai có thể chứa một từ liên tưởng joy, nhưng ý nghĩa cảm xúc toàn câu không giống nhau. Đây là lý do phải phân biệt việc đếm từ thuộc nhóm joy với việc phân loại câu thể hiện niềm vui. Một bộ nhãn câu cần quy định phủ định và cảm xúc hỗn hợp.

<a id="muc-8"></a>

## 8. Kết quả và cách diễn giải

Đầu ra có thể là số hoặc tỷ lệ từ gắn với từng nhóm. “Joy = 4 trên 100 token” nghĩa là có bốn vị trí được quy tắc khớp với nhãn đó, không phải người viết vui 4% hay 4% khách vui. Các tỷ lệ nhóm không buộc cộng thành 100% vì nhãn chồng lấn và có từ không thuộc nhóm nào.

<a id="muc-9"></a>

## 9. Điểm mạnh, điểm yếu và giới hạn

Điểm mạnh thực hành là phân tích được nhiều chiều cảm xúc và truy về từ góp điểm. Hạn chế quan trọng là liên tưởng tĩnh có thể khác ý nghĩa trong ngữ cảnh; bản dịch có thể thêm sai số. Với “The staff prevented a frightening situation”, từ gợi sợ hãi có mặt trong một lời ghi nhận tích cực. Đọc chỉ một nhóm từ sẽ làm mất cấu trúc này.

<a id="muc-10"></a>

## 10. So sánh với phương pháp gần nhau

| Tài nguyên | Đại lượng khác nhau |
|---|---|
| NRC EmoLex | Quan hệ từ với nhóm cảm xúc/sentiment |
| NRC Emotion Intensity | Tài nguyên cường độ, không phải cùng bảng nhị phân |
| NRC VAD | Các chiều valence, arousal, dominance |
| AFINN | Điểm valence mục từ; không cung cấp cùng hồ sơ tám cảm xúc |

Không được gọi mọi tài nguyên mang tên NRC là một phương pháp duy nhất hoặc dùng công thức của tài nguyên này để diễn giải tài nguyên kia.

<a id="muc-11"></a>

## 11. Sai lầm thường gặp và cách khắc phục

Không gọi vector đếm là phân phối xác suất. Không ép mỗi từ hoặc câu vào duy nhất một cảm xúc khi thiết kế cho phép nhiều nhãn. Không suy từ “trust words” sang mức tin tưởng thương hiệu mà chưa kiểm định cấu trúc. Hãy đặt tên biến cụ thể như “tỷ lệ token khớp nhóm trust” thay vì tên khái niệm rộng hơn khả năng phép đo.

<a id="muc-12"></a>

## 12. Tài liệu tham khảo

- **[S1]** Mohammad, S. M., & Turney, P. D. [NRC Emotion Lexicon: mô tả, tải và điều kiện sử dụng](https://saifmohammad.com/WebPages/NRC-Emotion-Lexicon.htm).
- **[S2]** Mohammad, S. M., & Turney, P. D. (2013). *Crowdsourcing a Word–Emotion Association Lexicon*. Computational Intelligence, 29(3), 436–465. [Bản thảo được trang tác giả dẫn](https://arxiv.org/abs/1308.6297).
- **[S3]** Mohammad, S. (2023). *Best Practices in the Creation and Use of Emotion Lexicons*. [ACL Anthology](https://aclanthology.org/2023.findings-eacl.136/).

S1–S2 cho thông tin tài nguyên; S3 thảo luận thực hành và đạo đức. Ví dụ và quy trình khách sạn trong tài liệu này là đề xuất ứng dụng, không phải kết quả của ba nguồn.

<a id="muc-13"></a>

## 13. Vị trí trong toàn bộ nghiên cứu

Tài nguyên nằm ở khâu tạo chỉ báo từ văn bản. Các chỉ báo có thể phục vụ mô tả hoặc làm đặc trưng của mô hình phân loại. Nếu mục tiêu cuối là giải thích sự hài lòng, cần chứng minh mối liên hệ giữa chỉ báo ngôn ngữ và cấu trúc đó trước khi diễn giải hệ số thống kê thành hành vi/cảm xúc của khách.

<a id="muc-14"></a>

## 14. Kiến thức nền cần biết trước

Cần hiểu multi-label là nhiều nhãn cùng đúng, khác multi-class một nhãn trong nhiều lớp. Hiểu token, mẫu số, tỷ lệ và coverage. Trong cảm xúc học, việc chọn tập nhóm cảm xúc là một quyết định lý thuyết; việc một tài nguyên có tám nhóm không chứng minh mọi trải nghiệm con người chỉ gồm đúng tám nhóm đó.

<a id="muc-15"></a>

## 15. Nền tảng lý thuyết và logic hình thành

Ứng dụng dựa trên giả định từ ngữ mang các liên tưởng cảm xúc có thể nhận diện và tổng hợp. Nhưng liên tưởng có thể gợi cảm xúc cho người đọc, mô tả cảm xúc người khác hoặc phủ định cảm xúc của bản thân. Nghiên cứu cần nói rõ đang quan sát loại nào, thay vì gộp ba cơ chế thành “cảm xúc khách hàng”.

<a id="muc-16"></a>

## 16. Các thành phần cấu tạo

Thành phần cốt lõi là bảng từ–nhãn. Hệ thống đầy đủ cần thêm xử lý ngôn ngữ, quy tắc khớp, xử lý từ không tìm thấy, tổng hợp và diễn giải. Với tiếng Việt, nên có bảng đối chiếu những mục dịch hoặc cụm nhiều âm tiết được sửa; ghi cả lý do và người thẩm định để kiểm tra tính nhất quán.

<a id="muc-17"></a>

## 17. Các loại và biến thể

Phân biệt bản tiếng Anh gốc, các bản dịch và bản thích nghi do nhóm tự xây. Ngoài ra, NRC VAD và NRC Emotion Intensity là tài nguyên khác, không phải chỉ bật một chế độ mới trong EmoLex [S1]. Nếu thêm quy tắc phủ định, cần báo đó là hệ thống EmoLex kết hợp quy tắc và so sánh với bản đếm thuần túy.

<a id="muc-18"></a>

## 18. Đầu vào và đầu ra

Đề xuất lưu `review_id`, `language`, `n_tokens`, `n_lexicon_matches`, mười cột đếm nhãn và mười cột tỷ lệ nếu cần. Phân biệt từ **có trong từ điển nhưng mọi nhãn bằng 0** với từ **không có trong từ điển**. Hai trường hợp có thể cùng không tăng tổng cảm xúc nhưng khác nhau về độ bao phủ tài nguyên.

<a id="muc-19"></a>

## 19. Căn cứ lựa chọn

Chọn thử khi câu hỏi yêu cầu hồ sơ nhiều chiều liên tưởng cảm xúc và bạn có thể kiểm tra từ đóng góp. Với nghiên cứu tiếng Việt, nên dành pilot cho chất lượng dịch/tách từ trước khi mở rộng mẫu. Nếu mục tiêu là cường độ giận hoặc xác suất câu biểu lộ sợ, cần phép đo khác hoặc một mô hình được kiểm định đúng mục tiêu.

<a id="muc-20"></a>

## 20. Trường hợp không nên sử dụng

Không dùng đếm từ để chẩn đoán tâm lý, nhận diện trạng thái thực của từng cá nhân hoặc suy ra khách có ý định quay lại. Không nên áp dụng bản dịch chưa kiểm tra để so trực tiếp các quốc tịch. Có thể kết hợp bảng hỏi hoặc nhãn con người để kiểm tra độ giá trị; phương pháp bổ sung cần phù hợp câu hỏi và sự đồng thuận của người tham gia.

<a id="muc-21"></a>

## 21. Các quyết định quan trọng khi thực hiện

Chọn mẫu số là tổng token hay token khớp sẽ đổi ý nghĩa chỉ số. Chọn trung bình mỗi đánh giá hay gộp tất cả token cũng đổi trọng số giữa người viết ngắn và dài. Nếu so mười nhóm nhãn trên nhiều nhóm khách, cần xác định trước giả thuyết chính và cách xử lý nhiều phép so sánh để hạn chế chọn kết quả thuận lợi sau khi xem dữ liệu.

<a id="muc-22"></a>

## 22. Giải thích công thức và ký hiệu

Gọi $a(w,e)\in\{0,1\}$ là quan hệ giữa từ $w$ và nhãn $e$. Với văn bản có $N$ token:

$$C_e(d)=\sum_{i=1}^{N}a(w_i,e),\qquad R_e(d)=C_e(d)/N.$$

Đây là công thức **đếm đơn giản được chọn cho minh họa**, không phải bộ phân loại câu chính thức. Từ không khớp không tăng $C_e$, nhưng phải được ghi nhận riêng trong coverage. Vì một từ có nhiều nhãn, $\sum_e R_e$ có thể lớn hơn 1. Nếu $N=0$, tỷ lệ không xác định.

<a id="muc-23"></a>

## 23. Ví dụ tính toán hoặc thao tác từng bước

Từ điển giả định chỉ để học: `happy → {joy, positive}`, `safe → {trust, positive}`. Với năm token `we feel happy and safe`, đếm được joy=1, trust=1, positive=2. Tỷ lệ tương ứng là 0.2, 0.2, 0.4; không được xem chúng là ba xác suất loại trừ nhau.

```python
# Từ điển giả định, không phải bản sao NRC EmoLex.
lexicon = {"happy": {"joy", "positive"},
           "safe": {"trust", "positive"}}
tokens = "we feel happy and safe".split()
labels = ["joy", "trust", "positive"]
counts = {e: sum(e in lexicon.get(w, set()) for w in tokens)
          for e in labels}
ratios = {e: counts[e] / len(tokens) for e in labels}
print(counts, ratios)
```

Khi dùng dữ liệu thật, thay bảng giả định bằng tài nguyên được tải hợp lệ và bộ tách từ phù hợp; không coi `.split()` là giải pháp tách từ tiếng Việt đầy đủ.

<a id="muc-24"></a>

## 24. Tình huống sai và phản ví dụ

“I was not afraid” vẫn có từ gợi fear nếu đếm thuần túy. Kết luận “khách sợ hãi” từ đó là vượt quá đầu ra. Có hai hướng hợp lệ: giữ mục tiêu là đếm từ liên tưởng fear và nói rõ giới hạn, hoặc bổ sung mô hình xử lý biểu hiện cảm xúc trong ngữ cảnh rồi đánh giá trên nhãn câu.

<a id="muc-25"></a>

## 25. Kiểm tra chất lượng và độ vững

Đề xuất kiểm tra coverage và đọc các mục góp nhiều điểm nhất. Nếu phân loại nhiều nhãn, đánh giá precision/recall/F1 theo từng nhãn và trung bình phù hợp. Nếu chỉ nghiên cứu tỷ lệ từ, kiểm tra độ nhạy theo tách từ, mẫu số và từ điển dịch. Không đánh giá bằng một bộ nhãn cực tính rồi kết luận chất lượng của toàn bộ tám cảm xúc.

<a id="muc-26"></a>

## 26. Xử lý khi không đáp ứng yêu cầu

Sai do dịch: thẩm định các mục gây lỗi và lưu bản sửa riêng. Sai do tách từ: xử lý đơn vị nhiều âm tiết trước khi đổi nhãn từ. Sai do ngữ cảnh: đổi mục tiêu hoặc thêm phương pháp phù hợp. Không tự thêm nhiều nhãn sau khi xem tập kiểm tra rồi dùng chính tập đó làm bằng chứng cải thiện độc lập.

<a id="muc-27"></a>

## 27. Phân biệt kết quả với kết luận

“Nhóm A có tỷ lệ từ liên tưởng trust cao hơn” không đồng nghĩa “nhóm A tin thương hiệu hơn”. Cần loại trừ khác biệt chủ đề, độ dài và cách diễn đạt. Muốn nghiên cứu trust như một cấu trúc lý thuyết, nên kiểm tra hội tụ với thang đo độc lập và phân biệt với các cấu trúc gần nhau.

<a id="muc-28"></a>

## 28. Cách viết vào bài nghiên cứu

Ghi tên đầy đủ tài nguyên, phiên bản, ngôn ngữ, nguồn dịch, cách tách từ và mẫu số. Cách diễn đạt phù hợp: “Chúng tôi đo tỷ lệ token khớp các nhóm liên tưởng cảm xúc bằng NRC EmoLex.” Chỉ viết “phân loại cảm xúc người viết” khi đã có quy trình và kiểm định đúng cấp suy luận đó.

<a id="muc-29"></a>

## 29. Khả năng tái lập và tính minh bạch

Lưu cấu hình, mã tính tỷ lệ, bảng chỉnh sửa và nhãn kiểm định được phép công khai. Nguồn chính thức nêu điều kiện riêng cho sử dụng phi thương mại và thương mại [S1]; cần xem điều kiện đi kèm bản tải trước khi phân phối lại. Kho GitHub học tập có thể chứa hướng dẫn tải, mã và dữ liệu giả định thay vì đăng toàn bộ tài nguyên.

<a id="muc-30"></a>

## 30. Đạo đức nghiên cứu liên quan

Các suy luận cảm xúc có thể gây hại khi áp vào cá nhân; nguồn thực hành tốt nhấn mạnh việc cân nhắc cách tạo, dùng và diễn giải lexicon [S3]. Trong nghiên cứu khách sạn, nên giới hạn kết luận ở cấp dữ liệu phù hợp và tránh gán cảm xúc thật cho người viết từ vài từ khóa. Việc công khai phản hồi cần bảo vệ thông tin nhận diện.

<a id="muc-31"></a>

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Tám tỷ lệ cảm xúc có cần cộng thành 100% không? **Không.**
2. Bản tiếng Việt tự động dịch có tự động bảo đảm giá trị đo lường không? **Không.**
3. Bài tập: viết ba câu cùng có “afraid” nhưng mang nghĩa khẳng định, phủ định và trích lời người khác. So kết quả đếm với nhãn cảm xúc toàn câu; giải thích tại sao hai nhiệm vụ khác nhau.

<a id="muc-32"></a>

## 32. Những điểm còn tranh luận

Vấn đề trọng tâm là khoảng cách từ liên tưởng của từ đến cảm xúc của người viết. Việc chọn một tập nhóm cảm xúc cũng là lựa chọn lý thuyết, có thể chịu ảnh hưởng ngôn ngữ và văn hóa. Nghiên cứu nên trình bày đây là cách vận hành hóa có giới hạn, kiểm tra các cách đo khác khi khả thi và tránh gọi đầu ra là sự thật trực tiếp về nội tâm con người.
