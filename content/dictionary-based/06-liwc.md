# LIWC — Đo các nhóm ngôn ngữ trong nghiên cứu

> Nhánh: LEXICON BASED → Dictionary Based. Phạm vi: cơ chế phân tích từ điển truyền thống, với LIWC-22 làm mốc tài liệu. Không đồng nhất mọi mô-đun của ứng dụng với phép đếm đơn giản. Đối chiếu nguồn ngày 28/10/2026. Ví dụ tự soạn không sử dụng hay phân phối từ điển thương mại.

| Trường | Thông tin |
|---|---|
| Tên đầy đủ | Linguistic Inquiry and Word Count |
| Bản chất | Hệ công cụ và từ điển phân tích ngôn ngữ; phạm vi rộng hơn sentiment |
| Mốc dùng trong bài | LIWC-22, tài liệu phát triển năm 2022 [S1] |
| Đầu vào | Văn bản; ứng dụng có thể nhập nhiều định dạng, nhưng đối tượng phân tích vẫn là nội dung chữ |
| Tiếng Việt | Cần một từ điển tiếng Việt phù hợp và bằng chứng kiểm định; không suy ra từ khả năng nhập Unicode |
| Nhãn, tính toán | Chấm từ điển có sẵn không cần nhãn huấn luyện dự án; cần dữ liệu thẩm định. Phân tích truyền thống có thể dùng CPU |
| Truy cập | [Trang LIWC](https://www.liwc.app/); không giả định có kho GitHub công khai thay thế từ điển chính thức |

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

LIWC phân tích văn bản bằng các nhóm ngôn ngữ có ý nghĩa tâm lý/xã hội. Trong cơ chế truyền thống, từ được đối chiếu với từ điển và nhiều kết quả được tính bằng phần trăm từ thuộc nhóm. Tuy nhiên, word count, words per sentence và các chỉ số tổng hợp có cách tính khác [S2–S3]. LIWC vì thế không chỉ là một bộ phân loại tích cực/tiêu cực.

<a id="muc-2"></a>

## 2. Giải thích đơn giản

Hình dung một bảng phân loại từ: đại từ, biểu hiện cảm xúc, quan hệ xã hội và các nhóm khác. Đọc một đánh giá, hệ thống thống kê mỗi nhóm xuất hiện bao nhiêu. Đây giống một bảng thành phần ngôn ngữ. Từ bảng đó đến kết luận về tâm lý người viết là một bước suy luận cần bằng chứng, không phải sự thật tự hiện ra từ phép đếm.

<a id="muc-3"></a>

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu khách sạn, có thể hỏi đánh giá hài lòng và không hài lòng khác nhau về các nhóm ngôn ngữ nào, hoặc phản hồi của khách sạn thay đổi cách dùng từ sau chương trình đào tạo ra sao. Cần chọn nhóm biến dựa trên lý thuyết và giả thuyết, thay vì chạy hàng loạt rồi chỉ báo những nhóm có kết quả nổi bật.

<a id="muc-4"></a>

## 4. Điều kiện và giả định sử dụng

Phải có quyền dùng công cụ/từ điển, xác định ngôn ngữ và phiên bản. Giả định cần kiểm tra là nhóm từ phù hợp với cấu trúc bạn muốn đo trong bối cảnh nghiên cứu. Một nhóm từ xã hội không tự trở thành thang đo chất lượng tương tác. Văn bản ngắn tạo tỷ lệ dễ dao động; cần xem độ dài và đơn vị tổng hợp.

<a id="muc-5"></a>

## 5. Cơ chế hoạt động bên trong

Ở cấp truyền thống, LIWC đọc từ, đối chiếu các mục từ/thân từ/biểu thức trong từ điển và tính tỷ lệ theo nhóm [S2]. Một từ có thể đóng góp vào nhiều nhóm. Các chỉ số tổng hợp không được tái tạo chỉ bằng lấy tỷ lệ một nhóm từ. Bước ghép nhóm và các chỉ số phải được hiểu theo đúng tài liệu của phiên bản đang dùng.

<a id="muc-6"></a>

## 6. Quy trình áp dụng trong nghiên cứu

1. Xác định giả thuyết và nhóm biến phù hợp lý thuyết.
2. Chọn từ điển, ngôn ngữ, phiên bản và đơn vị văn bản.
3. Chuẩn bị dữ liệu có ID; kiểm tra trích xuất chữ từ tệp nguồn.
4. Cấu hình phân đoạn và các cột văn bản; chạy, xuất kết quả giữ ID.
5. Kiểm tra tỷ lệ khớp, độ dài và ví dụ đóng góp cho biến chính.
6. Thực hiện phân tích thống kê có kiểm soát thiết kế mẫu và nhiều phép so sánh.

<a id="muc-7"></a>

## 7. Ví dụ minh họa xuyên suốt

Nghiên cứu tự đề xuất: so sánh phản hồi khách sạn trước và sau đào tạo xử lý phàn nàn. Biến ngôn ngữ được chọn có thể liên quan cảm xúc và tương tác xã hội, nhưng phải đối chiếu tên nhóm chính thức. Cần kiểm tra độ dài, mức độ nghiêm trọng của phàn nàn và loại khách sạn; phản hồi dài hơn tự nó không chứng minh đồng cảm hơn.

<a id="muc-8"></a>

## 8. Kết quả và cách diễn giải

Nếu một nhóm tỷ lệ có giá trị 4.2, cách đọc thông thường là 4.2% từ trong văn bản thuộc nhóm đó, theo quy tắc của phiên bản [S3]. Nó không có nghĩa người viết mang 4.2% một đặc điểm tâm lý. Cũng không được diễn giải mọi cột đầu ra là phần trăm: phải tra định nghĩa từng biến, nhất là các chỉ số tổng hợp.

<a id="muc-9"></a>

## 9. Điểm mạnh, điểm yếu và giới hạn

Điểm mạnh thực hành là tạo được nhiều chỉ báo ngôn ngữ nhất quán và có tài liệu tâm trắc để tham khảo. Hạn chế là phép đo phụ thuộc từ điển, phiên bản và bối cảnh. Tỷ lệ từ cảm xúc cao có thể đến từ trích lại lời phàn nàn, không phải cảm xúc của người đang trả lời. Việc đo nhóm từ cần được tách khỏi việc gán trạng thái tâm lý.

<a id="muc-10"></a>

## 10. So sánh với phương pháp gần nhau

| Phương án | Khác biệt cần nhớ |
|---|---|
| NRC EmoLex | Tập trung liên tưởng cảm xúc/sentiment |
| VADER | Trọng tâm là sắc thái và các quy tắc kết hợp |
| LIWC truyền thống | Nhiều nhóm ngôn ngữ, không chỉ cảm xúc |
| Từ điển tự xây | Phải tự chứng minh chất lượng; không thừa hưởng kiểm định của LIWC |

Chạy từ điển riêng bằng ứng dụng LIWC không biến từ điển riêng thành thang đo chuẩn hóa chính thức.

<a id="muc-11"></a>

## 11. Sai lầm thường gặp và cách khắc phục

Không đọc tên như “Authenticity” thành kết luận người này nói thật. Không trộn các phiên bản vì cùng tên một cột. Không xem mọi nhóm là độc lập hay buộc tỷ lệ cộng thành 100%. Cần đọc tài liệu định nghĩa và báo phiên bản; khi nâng phiên bản, nên chạy cùng một mẫu để kiểm tra tính so sánh trước khi gộp dữ liệu.

<a id="muc-12"></a>

## 12. Tài liệu tham khảo

- **[S1]** Boyd, R. L., Ashokkumar, A., Seraj, S., & Pennebaker, J. W. (2022). *The Development and Psychometric Properties of LIWC-22*. [Manual chính thức](https://www.liwc.app/static/documents/LIWC-22%20Manual%20-%20Development%20and%20Psychometrics.pdf).
- **[S2]** LIWC. [How It Works](https://liwc.app/help/howitworks).
- **[S3]** LIWC. [LIWC Analysis: thao tác và diễn giải](https://liwc.app/help/liwc).
- **[S4]** LIWC. [Danh mục tài liệu tâm trắc](https://liwc.app/help/psychometrics-manuals).

S1 là nền tảng cần đọc sâu theo từng biến. Các ví dụ đào tạo nhân viên và kế hoạch kiểm định trong bài là đề xuất mới, không phải nghiên cứu đã được manual xác nhận.

<a id="muc-13"></a>

## 13. Vị trí trong toàn bộ nghiên cứu

LIWC có thể phục vụ vận hành hóa các đặc điểm ngôn ngữ trước khi kiểm định giả thuyết. Nếu so trước–sau đào tạo, cần tách câu hỏi “ngôn ngữ thay đổi không” khỏi “đào tạo có gây thay đổi không”. Bước đo chỉ cung cấp biến; thiết kế thực nghiệm hoặc chiến lược nhận diện mới quyết định sức mạnh suy luận nhân quả.

<a id="muc-14"></a>

## 14. Kiến thức nền cần biết trước

Nên hiểu tỷ lệ, phân bố, độ tin cậy, giá trị hội tụ và giá trị phân biệt. **Hội tụ** hỏi chỉ báo có tương thích với phép đo khác của cùng cấu trúc không; **phân biệt** hỏi nó có khác cấu trúc gần nhau không. Một tên nhóm có vẻ hợp lý chưa đủ thay thế hai loại bằng chứng này.

<a id="muc-15"></a>

## 15. Nền tảng lý thuyết và logic hình thành

Ngôn ngữ có thể cung cấp dấu hiệu về cách nghĩ, tương tác và trải nghiệm, nhưng quan hệ từ dấu hiệu đến cấu trúc tâm lý cần được nghiên cứu. Trong ứng dụng khách sạn, người viết còn bị ràng buộc bởi mẫu phản hồi và quy định thương hiệu. Vì vậy, phong cách văn bản có thể phản ánh tổ chức hoặc mẫu viết sẵn bên cạnh người soạn.

<a id="muc-16"></a>

## 16. Các thành phần cấu tạo

Một phân tích đầy đủ gồm từ điển, cấu hình nhóm, bước phân đoạn, cơ chế tính chỉ số và bảng kết quả ghép ID. Đối với nghiên cứu, bổ sung hướng dẫn chọn biến và kế hoạch thống kê. Nếu dùng custom dictionary, phải ghi ranh giới giữa biến tự xây và biến chuẩn; không giả định mọi summary measure hoạt động với mọi từ điển [S3].

<a id="muc-17"></a>

## 17. Các loại và biến thể

Phân biệt các phiên bản LIWC, từ điển theo ngôn ngữ, từ điển tùy chỉnh và các mô-đun phân tích khác trong ứng dụng. Bài này không hướng dẫn toàn bộ mô-đun. Có thể dùng LIWC-22 làm mốc đã xác định nhưng không gọi nó mặc nhiên là bản mới nhất về sau. Kết quả so sánh chỉ có ý nghĩa khi biết cả phiên bản công cụ và từ điển.

<a id="muc-18"></a>

## 18. Đầu vào và đầu ra

Đề xuất đầu vào một dòng mỗi văn bản, có `review_id`, `hotel_id`, thời gian và nội dung. Đầu ra giữ ID, word count, chỉ báo độ khớp và các nhóm đã chọn. Nếu văn bản lấy từ PDF, cần kiểm tra OCR hoặc lỗi trích xuất trước; hỗ trợ nhập PDF không có nghĩa LIWC đang hiểu hình ảnh như mô hình đa phương thức.

<a id="muc-19"></a>

## 19. Căn cứ lựa chọn

Cân nhắc LIWC khi nhóm ngôn ngữ của nó khớp giả thuyết và có căn cứ tài liệu về phép đo. Kiểm tra tính khả thi về truy cập, ngôn ngữ và độ dài. Nếu chỉ cần nhãn cực tính cơ bản, một công cụ khác có thể đủ; nếu cần phong cách ngôn ngữ hoặc nhóm tâm lý, phải chọn biến và kiểm định phù hợp, không chỉ chọn tên ứng dụng.

<a id="muc-20"></a>

## 20. Trường hợp không nên sử dụng

Không dùng một vài điểm để chẩn đoán cá nhân, xác định ai nói dối hoặc đánh giá năng lực nhân viên. Không áp từ điển không phù hợp ngôn ngữ để so sánh nhóm khách. Nếu không có tài nguyên đã kiểm định cho miền/ngôn ngữ, có thể nghiên cứu phát triển phép đo trước, thay vì trình bày kết quả như một thang đo đã sẵn độ giá trị.

<a id="muc-21"></a>

## 21. Các quyết định quan trọng khi thực hiện

Chọn biến trước hay sau khi xem kết quả, gộp các phản hồi cùng khách sạn hay giữ từng phản hồi, và xử lý văn bản ngắn là các quyết định chính. Dữ liệu lồng theo khách sạn/nhân viên cần phân tích tương ứng. Nếu chọn rất nhiều nhóm biến, nên phân biệt phân tích xác nhận với khám phá và điều chỉnh chiến lược kiểm định.

<a id="muc-22"></a>

## 22. Giải thích công thức và ký hiệu

Với nhóm từ $k$, phép đếm truyền thống được giải thích bằng:

$$P_k(d)=100\times\frac{n_k(d)}{N(d)}.$$

$n_k$ là số vị trí khớp nhóm, $N$ là số từ theo bộ phân tích; $P_k$ là tỷ lệ phần trăm. Vì một từ có thể thuộc nhiều nhóm, tổng các $P_k$ không bắt buộc bằng 100. Công thức này **không phải công thức của mọi summary measure**; muốn diễn giải các chỉ số đó phải đọc tài liệu riêng. Văn bản rỗng cần xử lý thiếu dữ liệu.

<a id="muc-23"></a>

## 23. Ví dụ tính toán hoặc thao tác từng bước

Giả định văn bản 100 từ có ba vị trí thuộc nhóm A và hai vị trí thuộc nhóm B; một vị trí thuộc cả hai. Kết quả A=3%, B=2%; có bốn vị trí riêng biệt thuộc ít nhất một nhóm, không phải năm. Đây là ví dụ toán học với nhóm giả định, không tiết lộ nội dung từ điển LIWC.

Trong ứng dụng: nhập bảng → chọn đúng cột văn bản → xác nhận từ điển/nhóm/phân đoạn → chạy → xuất giữ ID. Tên lựa chọn có thể thay đổi theo phiên bản; cần đối chiếu tài liệu thao tác [S3]. Không ghép nhiều phản hồi thành một dòng nếu đơn vị phân tích dự kiến là từng phản hồi.

<a id="muc-24"></a>

## 24. Tình huống sai và phản ví dụ

Một phản hồi viết “You said you were angry; we understand” có từ chỉ giận nhưng người phản hồi không nhất thiết đang giận. Nếu nhà nghiên cứu dùng tỷ lệ nhóm từ để kết luận trạng thái tâm lý của nhân viên, họ đã bỏ qua việc trích dẫn và vai giao tiếp. Có thể giữ kết luận ở cấp ngôn ngữ hoặc xây quy trình xác định người trải nghiệm cảm xúc.

<a id="muc-25"></a>

## 25. Kiểm tra chất lượng và độ vững

Đề xuất kiểm tra tỷ lệ khớp, độ dài và đọc mẫu đóng góp cho các biến chính. Nếu đo một cấu trúc như đồng cảm, đối chiếu đánh giá con người hoặc phép đo độc lập. Thử kết quả khi loại văn bản rất ngắn hoặc đổi cấp tổng hợp, với lý do rõ ràng. Không biến một ngưỡng độ dài tự chọn thành tiêu chuẩn phổ quát.

<a id="muc-26"></a>

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu độ khớp thấp, kiểm tra lỗi chữ và ngôn ngữ. Nếu từ điển không đo đúng miền, cân nhắc thích nghi và thẩm định riêng. Nếu các nhóm biến quá tương quan, xem quan hệ phân cấp/chồng lấn trước khi đưa cùng vào mô hình. Nếu kết luận đổi khi bỏ mẫu viết sẵn, cần báo vai trò của mẫu thay vì quy mọi biến động cho tâm lý người viết.

<a id="muc-27"></a>

## 27. Phân biệt kết quả với kết luận

“Tỷ lệ nhóm từ X tăng” là kết quả ngôn ngữ. “Nhân viên đồng cảm hơn” cần bằng chứng độ giá trị. “Đào tạo làm tăng đồng cảm” cần thêm nhận diện nhân quả. Trong phản hồi khách sạn, đồng thời thay đổi người quản lý, khách hàng và mẫu trả lời có thể giải thích khác biệt trước–sau mà không liên quan riêng chương trình đào tạo.

<a id="muc-28"></a>

## 28. Cách viết vào bài nghiên cứu

Nêu phiên bản công cụ/từ điển, tên nhóm chính xác, đơn vị, cách tiền xử lý và lý do chọn biến. Báo riêng nhóm tỷ lệ và chỉ số tổng hợp. Nếu dùng từ điển tự xây, mô tả quy trình xây, kiểm tra nội dung và kiểm định; không trích manual LIWC như bằng chứng trực tiếp rằng từ điển mới đã có cùng chất lượng.

<a id="muc-29"></a>

## 29. Khả năng tái lập và tính minh bạch

Lưu cấu hình xuất, biến được chọn, quy tắc phân đoạn, mã phân tích và ID liên kết. Tài nguyên có thể chịu điều kiện cấp phép; kho GitHub có thể cung cấp hướng dẫn, cấu hình và dữ liệu giả định mà không phân phối từ điển hạn chế. Ghi rõ người tái lập cần truy cập công cụ/tài nguyên nào, thay vì gọi toàn bộ quy trình là mã nguồn mở.

<a id="muc-30"></a>

## 30. Đạo đức nghiên cứu liên quan

Phân tích ngôn ngữ phản hồi có thể trở thành đánh giá nhân viên ngoài mục tiêu nghiên cứu ban đầu. Cần hạn chế nhận diện cá nhân và chỉ suy luận trong phạm vi được phép. Đặc biệt, chỉ số có tên gợi đặc điểm tâm lý không đủ làm căn cứ khen thưởng, kỷ luật hay kết luận về sức khỏe tâm thần của người viết.

<a id="muc-31"></a>

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Mọi đầu ra LIWC có phải phần trăm không? **Không.**
2. Từ điển tự xây chạy bằng LIWC có tự thừa hưởng kiểm định của LIWC không? **Không.**
3. Bài tập: dùng hai nhóm từ giả định chồng lấn, tính tỷ lệ trên hai phản hồi khác độ dài. So trung bình tỷ lệ từng phản hồi với tỷ lệ khi gộp token; giải thích trọng số của mỗi cách.

<a id="muc-32"></a>

## 32. Những điểm còn tranh luận

Một vấn đề là từ các nhóm từ hữu hạn có thể suy ra cấu trúc tâm lý đến mức nào trong bối cảnh mới. Một vấn đề khác là tính so sánh giữa ngôn ngữ, phiên bản và tập văn bản. Cách tiếp cận thuyết phục là dùng tài liệu tâm trắc như điểm xuất phát, rồi bổ sung kiểm định theo ngữ cảnh nghiên cứu; không coi việc sử dụng một công cụ nổi tiếng là đủ bảo đảm mọi diễn giải.
