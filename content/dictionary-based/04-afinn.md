# AFINN — Từ điển điểm sắc thái và phép cộng minh bạch

> Nhánh: LEXICON BASED → Dictionary Based. Đối chiếu nguồn ngày 28/10/2026. Ví dụ khách sạn được tự soạn. Từ điển nhỏ dùng trong phép tính bên dưới là giả định để giảng giải, không phải trích xuất chính thức từ AFINN.

| Trường | Thông tin |
|---|---|
| Bản chất | Danh sách từ/cụm có điểm sắc thái; gói Python là công cụ sử dụng tài nguyên |
| Công trình | Finn Årup Nielsen, 2011 [S1] |
| Đầu vào | Văn bản; bài này tập trung tài nguyên tiếng Anh |
| Tiếng Việt | Không mặc định gói/tài nguyên tiếng Anh hỗ trợ trực tiếp |
| Nhãn | Không cần nhãn huấn luyện của dự án cho phép tra cứu; vẫn cần nhãn đánh giá |
| Tính toán | Chấm từ vựng cơ bản thường phù hợp CPU |
| GitHub | [fnielsen/afinn](https://github.com/fnielsen/afinn) |

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

AFINN là tài nguyên từ vựng gán điểm valence cho từ/cụm, gắn với nghiên cứu của Nielsen về phân tích cảm xúc trên microblog. Bản danh sách tiếng Anh quen thuộc sử dụng điểm nguyên từ −5 đến +5 [S1–S2]. Cần phân biệt điểm của **mục từ** với điểm của **văn bản**: tổng điểm văn bản không nhất thiết nằm trong khoảng đó.

<a id="muc-2"></a>

## 2. Giải thích đơn giản

Hãy coi mỗi biểu thức là một phiếu khen hoặc chê có trọng số. Đọc một đánh giá, hệ thống tìm các phiếu xuất hiện rồi cộng lại. Phương án này dễ hiểu và dễ kiểm tra, nhưng nếu không có bước xử lý ngữ cảnh phù hợp, “good” trong “not good” vẫn có nguy cơ đóng góp theo nghĩa khen.

<a id="muc-3"></a>

## 3. Mục đích và vấn đề giải quyết

Có thể dùng AFINN làm baseline để hỏi: chỉ bằng bằng chứng từ vựng, ta phân loại đánh giá khách sạn được đến đâu? Nó cũng hữu ích để kiểm tra lợi ích tăng thêm của mô hình phức tạp. Nếu mô hình mới tốt hơn, cần so trên cùng dữ liệu và thước đo; không suy từ độ phức tạp rằng nó chắc chắn tốt hơn.

<a id="muc-4"></a>

## 4. Điều kiện và giả định sử dụng

Đề xuất xác nhận phiên bản danh sách, ngôn ngữ, cách khớp từ/cụm và cách xử lý dấu câu. Giả định cần kiểm tra là điểm mục từ phù hợp miền khách sạn và phép cộng đủ phản ánh mục tiêu. Nếu so văn bản khác độ dài, phải xem điểm tổng có đang phản ánh lượng từ được viết hơn là chiều hướng trải nghiệm hay không.

<a id="muc-5"></a>

## 5. Cơ chế hoạt động bên trong

Với một thiết kế cơ bản: chuẩn hóa phù hợp → nhận diện mục từ/cụm → tra điểm → cộng điểm. Gói `afinn` cung cấp giao diện như `Afinn().score(text)`; các lựa chọn như ngôn ngữ và emoticon phụ thuộc gói [S2]. Không được gán toàn bộ khả năng xử lý ngữ cảnh của một biến thể cho mọi bản AFINN. Khi cần phủ định, phải xem tài nguyên và mã cụ thể.

<a id="muc-6"></a>

## 6. Quy trình áp dụng trong nghiên cứu

1. Chọn bài toán cực tính, đơn vị câu/đánh giá và ngôn ngữ.
2. Chuẩn bị mẫu gán nhãn và chia tập độc lập.
3. Cố định tài nguyên, cách khớp cụm và chế độ emoticon.
4. Lưu điểm tổng, số lần khớp, độ dài và trạng thái thiếu bằng chứng.
5. Thử các quy tắc chuẩn hóa/ngưỡng trên tập phát triển.
6. Đánh giá trên tập kiểm tra, báo lỗi phủ định và khác biệt theo độ dài.

<a id="muc-7"></a>

## 7. Ví dụ minh họa xuyên suốt

Xét “The room was clean, but the service was terrible.” Nếu cả lời khen và lời chê có điểm, tổng có thể gần 0. Cần giữ hai thành phần thay vì gọi đây chắc chắn là đánh giá trung tính. Nếu câu tiếp theo chỉ mô tả giờ nhận phòng và không khớp mục từ nào, cùng điểm 0 đó lại có nguyên nhân hoàn toàn khác.

<a id="muc-8"></a>

## 8. Kết quả và cách diễn giải

Điểm dương cho biết tổng đóng góp dương vượt tổng âm trong cách khớp đang dùng. Điểm âm là chiều ngược lại. Độ lớn còn phụ thuộc số lần xuất hiện và độ dài. Vì vậy, tổng 8 không có nghĩa “tốt gấp đôi” tổng 4 theo thang hài lòng; muốn diễn giải cường độ trải nghiệm cần kiểm định đo lường riêng.

<a id="muc-9"></a>

## 9. Điểm mạnh, điểm yếu và giới hạn

Mạnh ở tính minh bạch, ít yêu cầu tính toán và dễ kiểm tra từng đóng góp. Yếu khi phép cộng bỏ qua phạm vi phủ định, đối tượng và nghĩa chuyên ngành. Với “a cold room”, “cold” có thể tiêu cực; trong “cold drinks available”, sắc thái có thể khác. Một điểm mục từ không tự biết đối tượng và tiêu chuẩn đánh giá đang được dùng.

<a id="muc-10"></a>

## 10. So sánh với phương pháp gần nhau

| Phương án | Khác biệt quan trọng |
|---|---|
| VADER | Kết hợp lexicon với hệ quy tắc chấm riêng |
| SentiWordNet | Điểm theo nghĩa WordNet, cần quyết định chọn nghĩa |
| NRC Emotion Lexicon | Quan hệ với nhiều nhóm cảm xúc, không phải cùng thang điểm cường độ |

Khi chọn công cụ, hãy bắt đầu từ biến cần đo: cực tính, liên tưởng cảm xúc hay nghĩa từ. Các tài nguyên này không thay thế nhau hoàn toàn.

<a id="muc-11"></a>

## 11. Sai lầm thường gặp và cách khắc phục

Không giới hạn tổng văn bản vào −5 đến +5 chỉ vì điểm mục từ có khoảng đó. Không gộp mục không khớp với mục có điểm trung tính mà không lưu cờ. Nếu thêm luật phủ định, cần kiểm tra cả câu có và không có phủ định; một luật đảo dấu quá rộng dễ sửa đúng câu này nhưng làm sai câu khác.

<a id="muc-12"></a>

## 12. Tài liệu tham khảo

- **[S1]** Nielsen, F. Å. (2011). *A new ANEW: Evaluation of a word list for sentiment analysis in microblogs*. [arXiv:1103.2903](https://arxiv.org/abs/1103.2903).
- **[S2]** Nielsen, F. Å. [AFINN: kho mã Python và hướng dẫn](https://github.com/fnielsen/afinn).
- **[S3]** Nielsen, F. Å. [Giới thiệu tài nguyên năm 2011](https://finnaarupnielsen.wordpress.com/2011/03/16/afinn-a-new-word-list-for-sentiment-analysis/).

Các nguồn trên giới thiệu tài nguyên và triển khai; ví dụ khách sạn, công thức chuẩn hóa đề xuất và kế hoạch đánh giá bên dưới là nội dung hướng dẫn, chưa phải kết quả của một thí nghiệm thực tế.

<a id="muc-13"></a>

## 13. Vị trí trong toàn bộ nghiên cứu

AFINN có thể dùng ở giai đoạn tạo đặc trưng hoặc làm đối chứng trong đánh giá phương pháp. Nếu dùng điểm tổng để nghiên cứu biến động cảm xúc theo tháng, cần kiểm soát việc lượng và độ dài đánh giá thay đổi giữa các tháng. Tổng điểm của cả tháng có thể tăng đơn giản vì có nhiều đánh giá hơn.

<a id="muc-14"></a>

## 14. Kiến thức nền cần biết trước

Cần hiểu mục từ, cụm từ, tần suất, tổng và trung bình. Phân biệt **tần suất token** với **số loại từ**: một từ xuất hiện ba lần có thể đóng góp ba lần, tùy quy tắc. Ngoài ra, cần hiểu mô hình cơ sở là mốc so sánh có quy trình rõ ràng, không phải kết quả kém được cố tình thiết kế để mô hình mới dễ thắng.

<a id="muc-15"></a>

## 15. Nền tảng lý thuyết và logic hình thành

Logic thực hành là tín hiệu valence của nhiều mục từ có thể tích lũy thành tín hiệu văn bản. Phần cần thận trọng là tính cộng được: thêm một từ khen không luôn tăng cảm xúc tương ứng trong mọi ngữ cảnh. Phép cộng là một phép xấp xỉ hữu ích để thử nghiệm, không phải quy luật tâm lý được chứng minh chỉ bằng tài nguyên từ vựng.

<a id="muc-16"></a>

## 16. Các thành phần cấu tạo

Hệ thống nghiên cứu gồm tài nguyên điểm, bộ khớp từ/cụm, bộ tổng hợp và quy tắc phân loại. Các thành phần tùy chọn có thể là xử lý phủ định, chuẩn hóa độ dài hoặc gán điểm theo khía cạnh. Cần báo phần nào có sẵn, phần nào nhóm tự thêm để người đọc tái lập đúng biến thể.

<a id="muc-17"></a>

## 17. Các loại và biến thể

Không đồng nhất tên danh sách như AFINN-111 với toàn bộ các bản tài nguyên nằm trong một gói phát hành. Cần ghi tệp thực dùng. Có thể nghiên cứu bản gốc, bản mở rộng theo miền và bản kết hợp luật; hai bản sau là cấu hình nghiên cứu riêng. Chế độ emoticon cũng nên được xem là một yếu tố cấu hình khi so sánh.

<a id="muc-18"></a>

## 18. Đầu vào và đầu ra

Đề xuất lưu `review_id`, `text`, `raw_score`, `n_tokens`, `n_matches`, `positive_sum`, `negative_sum`, `label`. Nếu dùng cụm, lưu cả vị trí span để phát hiện đếm lặp. Khi không có mục khớp, `n_matches=0` cần được giữ rõ; nhãn cuối có thể là “chưa đủ bằng chứng” nếu thiết kế cho phép.

<a id="muc-19"></a>

## 19. Căn cứ lựa chọn

Chọn thử khi muốn một baseline đơn giản có thể kiểm tra bằng tay và dữ liệu có ngôn ngữ phù hợp. Sau pilot, xem coverage, lỗi lớp âm và tính nhạy với độ dài. Nếu bài nghiên cứu nhấn mạnh tính giải thích, khả năng chỉ rõ đóng góp từ là lợi thế, nhưng vẫn phải chứng minh độ chính xác đủ cho mục tiêu.

<a id="muc-20"></a>

## 20. Trường hợp không nên sử dụng

Không dùng cộng điểm thuần túy như phương án duy nhất cho câu mỉa mai hoặc gắn ý kiến theo khía cạnh. Không xem từ điển tiếng Anh là bộ đo tiếng Việt. Khi các câu rất dài và nhiều ý trái chiều, có thể phân tích câu/khía cạnh hoặc dùng mô hình ngữ cảnh; quyết định cần dựa vào bộ nhãn và phân tích lỗi.

<a id="muc-21"></a>

## 21. Các quyết định quan trọng khi thực hiện

Khớp cụm hay từng từ, có đếm lặp không, dùng tổng hay trung bình và đặt ngưỡng ở đâu là các quyết định ảnh hưởng trực tiếp đầu ra. Nếu cụm “not good” được khớp, không nên vô tình cộng tiếp “good” bên trong cùng span nếu thiết kế không cho phép. Cần kiểm tra chồng lấn bằng các câu ngắn có thể tính tay.

<a id="muc-22"></a>

## 22. Giải thích công thức và ký hiệu

Với các đơn vị khớp $u_i$ và điểm từ điển $v(u_i)$:

$$S(d)=\sum_{i=1}^{m}v(u_i).$$

Có thể đề xuất thêm $S_{match}=S/m$ khi $m>0$ hoặc $S_{length}=S/N$ với $N$ là tổng số token. Đây là các quy tắc chuẩn hóa do nhà nghiên cứu lựa chọn, không phải các thang đo tương đương. $S/m$ đo cường độ trên phần khớp; $S/N$ còn phản ánh mật độ bằng chứng trong toàn văn bản.

<a id="muc-23"></a>

## 23. Ví dụ tính toán hoặc thao tác từng bước

Giả định từ điển học tập có `clean=2`, `terrible=-4`; các từ khác không khớp. Câu ở mục 7 cho $S=-2$, $m=2$, $S_{match}=-1$. Với 11 token theo cách tách từ được chọn, $S_{length}=-2/11$. Các con số là giả định và phụ thuộc định nghĩa token.

Gọi gói thật cần cài `afinn` trước:

```python
from afinn import Afinn

analyzer = Afinn(language="en")
text = "The room was clean, but the service was terrible."
print(analyzer.score(text))
```

Không dùng kết quả từ từ điển giả định để khẳng định đầu ra số của đoạn mã này.

<a id="muc-24"></a>

## 24. Tình huống sai và phản ví dụ

Hai đánh giá đều chứa “good” ba lần có thể rất khác: một đánh giá khen trực tiếp, đánh giá kia liên tục nói “not good”. Nếu quy trình chỉ cộng điểm “good”, nó không thể phân biệt đúng. Cách sửa không phải xóa từ phủ định, mà là xây/kiểm tra quy tắc phạm vi hoặc dùng một hệ thống có xử lý ngữ cảnh phù hợp hơn.

<a id="muc-25"></a>

## 25. Kiểm tra chất lượng và độ vững

Đề xuất macro-F1, precision/recall từng lớp và coverage; chia thêm kết quả theo nhóm độ dài. So sánh tổng, trung bình trên mục khớp và mật độ trên toàn token. Nếu xếp hạng khách sạn thay đổi mạnh giữa các cách chuẩn hóa, cần báo tính nhạy này thay vì chọn cách cho ra kết luận thuận lợi nhất.

<a id="muc-26"></a>

## 26. Xử lý khi không đáp ứng yêu cầu

Coverage thấp: kiểm tra tách từ, dấu câu, ngôn ngữ và cụm trước. Lỗi phủ định: đánh giá phương án thêm luật trên tập phát triển. Chênh lệch do độ dài: thử chuẩn hóa và xem độ giá trị. Nếu nhiều lỗi ngữ cảnh không thể sửa ổn định, giữ AFINN làm baseline và chuyển phương pháp chính; không cần ép một tài nguyên giải mọi vấn đề.

<a id="muc-27"></a>

## 27. Phân biệt kết quả với kết luận

Điểm từ vựng âm không chứng minh toàn bộ trải nghiệm của khách là tiêu cực. Tỷ lệ đánh giá bị gán âm còn phụ thuộc cách khách diễn đạt và phạm vi từ điển. Khi đưa vào phân tích quản trị khách sạn, cần kết hợp đọc mẫu phản hồi và thông tin khía cạnh để tránh biến một điểm tổng thành đề xuất quản trị không đúng trọng tâm.

<a id="muc-28"></a>

## 28. Cách viết vào bài nghiên cứu

Ghi tài nguyên/tệp, phiên bản gói, ngôn ngữ, cơ chế khớp, cách tổng hợp và ngưỡng. Mẫu: “Điểm được tính bằng [tổng/trung bình] trên [đơn vị khớp], với [quy tắc phủ định]. Chúng tôi xác định ngưỡng từ [tập] và đánh giá bằng [chỉ số].” Nếu mở rộng từ điển, báo số mục thêm và cách chọn mà không dùng tập kiểm tra.

<a id="muc-29"></a>

## 29. Khả năng tái lập và tính minh bạch

Lưu tệp tài nguyên hoặc đường dẫn/phiên bản và checksum theo quyền phân phối; giữ mã và nhật ký chỉnh sửa. Mã cần thể hiện rõ tổng và mẫu số chuẩn hóa. Với GitHub, nên cung cấp ví dụ tự soạn và hướng dẫn tải tài nguyên từ nguồn chính thức; không mặc định giấy phép của mã tự động áp dụng cho mọi dữ liệu phụ thuộc.

<a id="muc-30"></a>

## 30. Đạo đức nghiên cứu liên quan

Không dùng kết quả tự động để gán tính cách hoặc động cơ cho khách phàn nàn. Khi so sánh nhóm khách, xem liệu cách viết hoặc từ vựng của một nhóm có ít được tài nguyên nhận diện hơn không. Việc coverage không đồng đều có thể tạo chênh lệch điểm giả và dẫn tới đề xuất quản trị thiên lệch.

<a id="muc-31"></a>

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Điểm văn bản có bị giới hạn ở −5 đến +5 không? **Không nhất thiết.**
2. $S=0$ có đủ kết luận trung tính không? **Không, cần xem số khớp và hai thành phần.**
3. Bài tập: tạo hai đánh giá cùng chiều cảm xúc nhưng khác độ dài; tính tổng và hai dạng chuẩn hóa. Giải thích mỗi phép tính giữ lại và làm mất thông tin gì.

<a id="muc-32"></a>

## 32. Những điểm còn tranh luận

Một điểm cần cân nhắc là nên giữ cường độ tích lũy hay loại ảnh hưởng độ dài. Không có đáp án chung vì phụ thuộc cấu trúc cần đo. Một vấn đề khác là mở rộng từ điển đến mức nào trước khi nó trở thành tài nguyên theo miền riêng. Nghiên cứu tốt cần công bố lựa chọn và kiểm tra độ nhạy, thay vì coi một cách cộng điểm là mặc định đúng cho mọi bối cảnh.
