# Statistical — Xây từ điển cảm xúc bằng PMI và đồng xuất hiện

> **Nhánh:** LEXICON BASED → Corpus Based. **Vị trí nguồn:** dòng 9 trong bảng Excel, tên phương pháp “Statistical (PMI, co-occurrence)”. **Ngày đối chiếu nguồn:** 28/10/2026.
>
> Đây là một họ phương pháp. Bài trình bày nền tảng đồng xuất hiện và một thiết kế SO-PMI minh họa; không coi mọi phương pháp corpus-based là một thuật toán duy nhất. Ví dụ khách sạn, bảng đếm và chương trình nhỏ đều do người biên soạn tự xây dựng.

| Trường thông tin | Nội dung |
|---|---|
| Bản chất | Suy ra điểm từ/cụm từ các quan hệ thống kê trong ngữ liệu |
| Mốc học thuật | Church & Hanks, 1990 về liên kết từ; Turney, 2002 và Turney & Littman, 2003 về suy ra định hướng cảm xúc [S1–S3] |
| Đầu vào | Ngữ liệu văn bản, đơn vị ngữ cảnh, tập từ hạt giống và cấu hình đếm |
| Đầu ra | Từ điển từ/cụm kèm điểm, số đếm hỗ trợ và trạng thái đủ/thiếu bằng chứng |
| Tiếng Việt | Có thể thiết kế trực tiếp bằng ngữ liệu và seed tiếng Việt; cần xử lý từ nhiều âm tiết |
| Dữ liệu có nhãn | Không nhất thiết cần nhãn từng văn bản để ước lượng; seed vẫn đưa thông tin cảm xúc vào hệ thống. Đánh giá cần chuẩn độc lập |
| Tính toán | Đếm cơ bản có thể dùng CPU; bộ nhớ và thời gian tăng theo ngữ liệu, cửa sổ và số cặp lưu |
| Bộ dữ liệu thử | Ưu tiên tập đánh giá khách sạn đúng miền, được thu thập hợp lệ và tách riêng phần xây từ điển với phần kiểm tra |

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
- [25. Kiểm tra chất lượng và độ vững của kết quả](#muc-25)
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

**Co-occurrence** là sự cùng xuất hiện của hai đơn vị ngôn ngữ trong một ngữ cảnh đã quy định. **PMI — Pointwise Mutual Information** đo mức liên kết của một cặp sự kiện bằng cách so xác suất cùng xảy ra với tích xác suất riêng của chúng [S1]. Trong xây từ điển cảm xúc, có thể so liên kết của từ ứng viên với seed tích cực và tiêu cực để suy ra **semantic orientation**, tức định hướng đánh giá [S2–S3].

PMI tự nó không phải cảm xúc. Chỉ khi gắn nó với nguồn định hướng như các seed, ta mới tạo ra một quy tắc ước lượng sắc thái.

<a id="muc-2"></a>

## 2. Giải thích đơn giản

Giả sử bạn chưa biết cụm “đáng tiền” thường là khen hay chê. Bạn đọc nhiều đánh giá và xem cụm đó hay xuất hiện gần các lời khen hay lời chê. Nếu liên kết với nhóm khen mạnh hơn, bạn có cơ sở đề xuất điểm dương.

Tuy nhiên, chỉ đếm số lần đi cùng chưa đủ: một từ rất phổ biến có thể xuất hiện cạnh hầu hết mọi thứ. PMI điều chỉnh theo mức phổ biến riêng để hỏi liệu sự đi cùng đó có nhiều hơn mức dự kiến nếu hai sự kiện độc lập hay không.

<a id="muc-3"></a>

## 3. Mục đích và vấn đề giải quyết

Một từ điển chung có thể bỏ sót cách diễn đạt của khách du lịch. Đề xuất nghiên cứu: xây từ điển từ đánh giá khách sạn rồi kiểm tra nó có nhận diện tốt hơn các biểu thức theo miền hay không.

Sản phẩm có thể dùng để khám phá từ ứng viên, tạo đặc trưng cho mô hình khác hoặc xây baseline phân loại. Mỗi mục tiêu cần đánh giá riêng: tìm đúng từ cảm xúc khác với phân loại đúng toàn bộ đánh giá.

<a id="muc-4"></a>

## 4. Điều kiện và giả định sử dụng

Ngữ liệu cần có đủ lần xuất hiện của ứng viên và seed để ước lượng quan hệ. Không có một số lượng đánh giá tối thiểu dùng chung cho mọi miền; cần xem phân bố tần suất, độ ổn định và độ phủ thực tế.

Giả định cốt lõi cần kiểm tra là liên kết với seed phản ánh sắc thái, thay vì chỉ phản ánh chủ đề. Ví dụ “phòng” xuất hiện nhiều trong cả khen và chê. Seed cũng phải có nghĩa phù hợp miền, hiện diện đủ và không bị một mẫu viết lặp lại chi phối.

<a id="muc-5"></a>

## 5. Cơ chế hoạt động bên trong

Quy trình minh họa gồm: xác định đơn vị ngữ cảnh → lập tập từ/cụm mỗi ngữ cảnh → đếm riêng và đếm chung → ước lượng PMI → so hai phía seed → lọc mục thiếu bằng chứng → tạo từ điển.

Trong ví dụ của bài, ngữ cảnh là **một câu**, mỗi từ chỉ tính có/không trong câu đó. Vì vậy, lặp một từ ba lần trong cùng câu vẫn chỉ tăng một đơn vị ở số đếm câu. Nếu dùng cặp vị trí token hoặc cửa sổ trượt, phải thay đổi mô hình đếm và mẫu số cho nhất quán.

<a id="muc-6"></a>

## 6. Quy trình áp dụng trong nghiên cứu

1. Định nghĩa cấu trúc cần đo: cực tính từ, câu hay khía cạnh.
2. Thu thập ngữ liệu hợp lệ, loại trùng và lưu mã khách sạn/người viết khi có thể.
3. Chia phần xây tài nguyên, phần phát triển và phần kiểm tra trước khi tối ưu.
4. Chọn seed bằng lý do ngôn ngữ và kiểm tra chúng trong ngữ cảnh thật.
5. Cố định tách từ, cụm, ngữ cảnh, số đếm và xử lý số 0.
6. Tính điểm, lưu số đếm hỗ trợ; thẩm định từ ứng viên trên mẫu phát triển.
7. Đóng băng từ điển và ngưỡng trước đánh giá độc lập.
8. Báo cáo cả chất lượng từ điển lẫn chất lượng ứng dụng cuối.

Nếu dự kiến áp dụng sang khách sạn mới, chia theo khách sạn giúp kiểm tra chuyển miền sát mục tiêu hơn chia ngẫu nhiên các câu cùng một khách sạn.

<a id="muc-7"></a>

## 7. Ví dụ minh họa xuyên suốt

Giả sử cần suy ra sắc thái của “cozy” từ 100 câu tiếng Anh về khách sạn. Chọn hai seed minh họa `excellent` và `poor`; bảng số liệu tự đặt như sau:

| Sự kiện | Số câu |
|---|---:|
| Có `cozy` | 30 |
| Có `excellent` | 20 |
| Có `poor` | 10 |
| Có cả `cozy` và `excellent` | 12 |
| Có cả `cozy` và `poor` | 2 |

Ta sẽ tính mức liên kết với mỗi seed ở mục 22–23. Ví dụ chỉ cho thấy cách tính; không khẳng định đây là thống kê thật hoặc nghĩa phổ quát của “cozy”.

<a id="muc-8"></a>

## 8. Kết quả và cách diễn giải

PMI dương nghĩa là cặp xảy ra nhiều hơn dự kiến theo giả định độc lập; PMI âm là ít hơn dự kiến. Đây là quan hệ thống kê của cặp, chưa phải nhãn khen/chê.

Điểm SO-PMI dương trong thiết kế này nghĩa là ứng viên liên kết tương đối mạnh hơn với phía tích cực. Điểm không phải xác suất và không có sẵn thang hài lòng. Cần đọc cùng số đếm hỗ trợ: một điểm lớn từ hai lần xuất hiện không tương đương bằng chứng ổn định từ hàng nghìn lần.

<a id="muc-9"></a>

## 9. Điểm mạnh, điểm yếu và giới hạn

**Điểm mạnh:** có thể học tín hiệu theo miền từ văn bản chưa gán nhãn từng bản ghi; tính toán có thể truy về bảng đếm; dễ tạo baseline minh bạch.

**Điểm yếu:** từ hiếm dễ tạo tỷ số cực đoan; lựa chọn seed và ngữ cảnh ảnh hưởng kết quả; đồng xuất hiện có thể chỉ phản ánh chủ đề. Câu có nhiều khía cạnh còn làm lời khen và lời chê đi cùng nhau dù không cùng đối tượng.

Vì vậy, không dùng độ lớn điểm thay cho đánh giá độ chắc chắn hoặc chứng minh hiểu ngữ cảnh.

<a id="muc-10"></a>

## 10. So sánh với phương pháp gần nhau

| Phương án | Đại lượng và khác biệt |
|---|---|
| Số đếm đồng xuất hiện | Cho lượng đi cùng; chưa điều chỉnh xác suất riêng |
| PMI | Liên kết một cặp sự kiện; chưa có định hướng cảm xúc tự thân |
| SO-PMI | So liên kết với seed dương và âm để tạo điểm định hướng |
| Embedding-based induction | Dùng biểu diễn vector/đồ thị thay vì chỉ liên kết trực tiếp với seed |
| Từ điển dựng sẵn | Tra tài nguyên có sẵn; không tự học điểm mới từ corpus dự án |

Ranh giới “statistical” và “semantic” trong bảng là cách tổ chức để học: embedding cũng có thể được học từ thống kê đồng xuất hiện.

<a id="muc-11"></a>

## 11. Sai lầm thường gặp và cách khắc phục

- **Trộn mẫu số:** lấy đồng xuất hiện theo câu nhưng tần suất riêng theo token. Cần đưa mọi xác suất về cùng không gian sự kiện.
- **Gọi từ không xuất hiện là trung tính:** phải ghi thiếu bằng chứng.
- **Chọn seed sau khi xem nhãn test:** dùng tập phát triển và đóng băng lựa chọn.
- **Xóa phủ định:** giữ “không”, “not” và kiểm tra phạm vi; cùng xuất hiện với “good” không luôn là khen.
- **Dùng số lượt tìm kiếm web như số đếm cố định:** nếu theo hướng PMI-IR lịch sử, phải mô tả nguồn và khả năng tái lập; đếm corpus cục bộ là một thiết kế khác.

<a id="muc-12"></a>

## 12. Tài liệu tham khảo

- **[S1]** Church, K. W., & Hanks, P. (1990). *Word Association Norms, Mutual Information, and Lexicography*. Computational Linguistics, 16(1), 22–29. [ACL Anthology](https://aclanthology.org/J90-1003/).
- **[S2]** Turney, P. (2002). *Thumbs Up or Thumbs Down? Semantic Orientation Applied to Unsupervised Classification of Reviews*. ACL, 417–424. [Bài báo](https://aclanthology.org/P02-1053/). [DOI](https://doi.org/10.3115/1073083.1073153).
- **[S3]** Turney, P. D., & Littman, M. L. (2003). *Measuring Praise and Criticism: Inference of Semantic Orientation from Association*. ACM TOIS, 21(4), 315–346. [Bản thảo](https://arxiv.org/abs/cs/0309034). [DOI](https://doi.org/10.1145/944012.944013).

S1 là nền tảng liên kết từ; S2–S3 minh họa suy ra định hướng bằng quan hệ với từ tham chiếu. Quy trình đếm câu, ngữ liệu giả định và mã trong bài này là thiết kế giảng giải riêng, không phải bản tái lập nguyên vẹn PMI-IR của Turney.

<a id="muc-13"></a>

## 13. Vị trí trong toàn bộ nghiên cứu

Phương pháp nằm ở khâu tạo tài nguyên hoặc vận hành hóa văn bản thành chỉ báo. Sau khi xây từ điển, vẫn cần bộ chấm câu/văn bản và một bước kiểm định.

Nếu điểm cuối được dùng làm biến phụ thuộc trong nghiên cứu chất lượng dịch vụ, sai số từ điển sẽ truyền sang mô hình thống kê. Nên thẩm định phép đo trước khi diễn giải các hệ số thành phát hiện quản trị.

<a id="muc-14"></a>

## 14. Kiến thức nền cần biết trước

Cần hiểu xác suất biên, xác suất đồng thời, độc lập và logarit. Cần biết **seed** là đơn vị được cấp định hướng ban đầu; **candidate** là từ/cụm đang được suy ra điểm.

Phân biệt **token frequency** và **document/sentence frequency**: số lần từ xuất hiện khác số ngữ cảnh có từ đó. Trong bài này, xác suất “có từ trong một câu ngẫu nhiên” dùng tần suất câu.

<a id="muc-15"></a>

## 15. Nền tảng lý thuyết và logic hình thành

Định hướng suy ra dựa trên ý tưởng rằng quan hệ sử dụng trong ngôn ngữ cung cấp bằng chứng về ý nghĩa đánh giá. Công trình Turney & Littman thử cách suy ra từ liên kết với các nhóm từ tham chiếu [S3].

Trong ứng dụng khách sạn, cần bổ sung kiểm tra xem bằng chứng đang phản ánh cảm xúc, chủ đề hay cấu trúc thể loại. Đây là câu hỏi thực nghiệm; không thể giải bằng cách chỉ đổi tên biến thành “sentiment”.

<a id="muc-16"></a>

## 16. Các thành phần cấu tạo

Một hệ hoàn chỉnh cần: corpus; bộ tách từ/cụm; định nghĩa ngữ cảnh; seed hai phía; bộ đếm; phép ước lượng liên kết; quy tắc lọc; từ điển đầu ra; bộ chấm ứng dụng.

Nên lưu tách từng thành phần. Nhờ đó, khi chất lượng đổi, có thể biết nguyên nhân do corpus mới, seed mới hay cách chấm văn bản mới thay vì quy tất cả cho PMI.

<a id="muc-17"></a>

## 17. Các loại và biến thể

Có thể dùng ngữ cảnh theo câu, văn bản, cửa sổ token hoặc quan hệ cú pháp. Có thể dùng một hay nhiều seed mỗi phía. **PPMI** đặt PMI âm về 0; thao tác này làm mất thông tin âm của liên kết, nên không tự thay thế PMI trong mọi công thức SO-PMI.

Các lựa chọn như ngưỡng tần suất, làm trơn và trọng số khoảng cách là biến thể thiết kế cần báo cáo. Corpus-based còn gồm những phương pháp khác; không nên gộp tất cả vào SO-PMI.

<a id="muc-18"></a>

## 18. Đầu vào và đầu ra

Đầu vào nên giữ `review_id`, `hotel_id`, `sentence_id`, thời gian, ngôn ngữ và token/cụm đã xử lý. Đầu ra từ điển nên gồm `term`, `count`, các số đếm liên kết seed, `score`, `support_status`, phiên bản corpus và cấu hình.

Khi ứng dụng, lưu riêng điểm đánh giá và tỷ lệ khớp. Một từ điển nhiều mục nhưng đa số mục rất hiếm có thể không hữu ích hơn một từ điển nhỏ đã được thẩm định.

<a id="muc-19"></a>

## 19. Căn cứ lựa chọn

Nên thử khi có corpus đúng miền, ít nhãn văn bản, có seed đáng tin và cần một baseline truy được số đếm. Pilot cần trả lời: bao nhiêu ứng viên có hỗ trợ đủ; điểm ổn định thế nào khi đổi mẫu; từ xếp cao có đúng chiều đánh giá không?

Không chọn chỉ vì “không cần nhãn”: thời gian kiểm tra seed, xử lý dữ liệu và thẩm định từ điển vẫn là chi phí nghiên cứu.

<a id="muc-20"></a>

## 20. Trường hợp không nên sử dụng

Không nên dùng đơn độc khi corpus quá thưa với phần lớn từ mục tiêu, seed gần như không xuất hiện hoặc nhiệm vụ cần hiểu mỉa mai và từng đối tượng chính xác. Có thể dùng từ điển sẵn, embedding thích hợp, mô hình có giám sát hoặc gán nhãn thủ công, tùy dữ liệu và mục tiêu.

Nếu chỉ có vài chục phản hồi, ví dụ PMI vẫn hữu ích để học phép tính nhưng chưa đủ cơ sở khẳng định đã xây được một từ điển miền đáng tin.

<a id="muc-21"></a>

## 21. Các quyết định quan trọng khi thực hiện

Ngữ cảnh rộng giúp tăng số đếm nhưng dễ trộn khía cạnh; ngữ cảnh hẹp giảm nhiễu nhưng có thể tăng độ thưa. Lọc từ hiếm làm giảm mục không ổn định nhưng cũng có thể bỏ cách nói quan trọng của nhóm ít xuất hiện.

Cần quyết định xử lý seed không có dữ liệu, giá trị 0 và câu hỗn hợp trước khi xem test. Nếu dùng test không nhãn để học từ điển, phải báo thiết kế **transductive**; không trình bày như đánh giá hoàn toàn trên dữ liệu chưa từng thấy.

<a id="muc-22"></a>

## 22. Giải thích công thức và ký hiệu

Với $N$ câu, gọi $c(w)$ là số câu có từ $w$, $c(s)$ là số câu có seed $s$, và $c(w,s)$ là số câu có cả hai:

$$\operatorname{PMI}(w,s)=\log_2\frac{P(w,s)}{P(w)P(s)}
=\log_2\frac{N\,c(w,s)}{c(w)c(s)}.$$

Thiết kế hai seed của bài dùng:

$$SO(w)=\operatorname{PMI}(w,s_+)-\operatorname{PMI}(w,s_-).$$

Nếu mọi số đếm cần thiết dương, có thể rút gọn thành:

$$SO(w)=\log_2\frac{c(w,s_+)c(s_-)}{c(w,s_-)c(s_+)}.$$

Với nhiều seed, một lựa chọn riêng là hiệu hai **trung bình** PMI; phải quy định cách xử lý seed thiếu dữ liệu và không nhầm với mọi công thức trong tài liệu gốc.

Nếu $c(w,s)=0$, PMI quan sát có log 0; chương trình minh họa trả `None` để tránh xử lý tùy tiện. Làm trơn là lựa chọn khác: với bảng hiện diện/vắng mặt $2\times2$, có thể cộng cùng $\alpha$ vào cả bốn ô, rồi tính lại biên và tổng $N+4\alpha$. Không chỉ cộng vào tử rồi giữ các xác suất còn lại như cũ và coi đó là cùng một mô hình xác suất.

<a id="muc-23"></a>

## 23. Ví dụ tính toán hoặc thao tác từng bước

Từ bảng mục 7:

$$PMI(cozy,excellent)=\log_2\frac{100\times12}{30\times20}=1.$$

$$PMI(cozy,poor)=\log_2\frac{100\times2}{30\times10}
\approx-0.585.$$

Do đó $SO(cozy)\approx1.585$: trong ngữ liệu giả định, bằng chứng nghiêng về phía seed tích cực. Điều đó chưa cho biết nhãn của mọi câu chứa “cozy”.

Mã dưới đây tạo đúng bảng số đếm giả định, chỉ cần thư viện chuẩn Python. Các tập là **sự kiện từ đã xử lý**, không phải câu văn tự nhiên để huấn luyện:

```python
from math import log2, isclose

contexts = (
    [{"cozy", "excellent"} for _ in range(12)]
    + [{"cozy", "poor"} for _ in range(2)]
    + [{"cozy"} for _ in range(16)]
    + [{"excellent"} for _ in range(8)]
    + [{"poor"} for _ in range(8)]
    + [set() for _ in range(54)]
)

def pmi(a, b, corpus):
    n = len(corpus)
    ca = sum(a in context for context in corpus)
    cb = sum(b in context for context in corpus)
    cab = sum(a in context and b in context for context in corpus)
    if not n or not ca or not cb or not cab:
        return None
    return log2(n * cab / (ca * cb))

positive = pmi("cozy", "excellent", contexts)
negative = pmi("cozy", "poor", contexts)
score = None if positive is None or negative is None else positive - negative
assert len(contexts) == 100
assert isclose(positive, 1.0)
assert isclose(score, log2(3))
assert pmi("missing", "excellent", contexts) is None
print(round(positive, 4), round(negative, 4), round(score, 4))
# 1.0 -0.585 1.585
```

Trên dữ liệu thật, cần thay phần tạo sự kiện bằng quy trình tách câu/từ đã kiểm tra, loại trùng và kiểm soát chia tập. Không dùng 100 sự kiện giả lập này để đánh giá độ chính xác nghiên cứu.

<a id="muc-24"></a>

## 24. Tình huống sai và phản ví dụ

“The room was excellent but the service was poor” làm cả hai seed đi cùng các từ chỉ khía cạnh. Nếu lấy cả câu làm ngữ cảnh, từ “room” có thể được liên kết với cả hai phía dù lời chê hướng vào dịch vụ.

Một câu khác, “The room was not excellent”, có seed dương nhưng ý nghĩa không phải lời khen trực tiếp. Có thể thử phân mệnh đề, khía cạnh hoặc xử lý phủ định; mỗi cách cần đánh giá lại thay vì mặc định sửa được mọi trường hợp.

<a id="muc-25"></a>

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đề xuất hai tầng kiểm tra. **Tầng từ điển:** người thẩm định đọc từ/cụm trong ngữ cảnh, chấm chiều cảm xúc; đo precision của các ứng viên được chọn hoặc tương quan với điểm chuẩn nếu có thang liên tục. **Tầng ứng dụng:** đo macro-F1, từng lớp và coverage trên nhãn văn bản độc lập.

Kiểm tra độ nhạy theo seed, tần suất và cửa sổ. Bootstrap theo khách sạn hoặc người viết khi dữ liệu lồng theo các đơn vị đó; việc lấy mẫu lại từng câu như độc lập có thể đánh giá thiếu biến thiên. Không coi độ ổn định cao là bằng chứng duy nhất của tính đúng.

<a id="muc-26"></a>

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu từ hiếm có điểm cực đoan, thử tăng hỗ trợ tối thiểu, làm trơn nhất quán hoặc cho phép từ chối chấm. Nếu seed quá chung/đa nghĩa, kiểm tra và thay trên tập phát triển. Nếu điểm bám chủ đề, xem lại loại ứng viên và phạm vi ngữ cảnh.

Nếu khác biệt giữa các nhóm đến từ corpus mất cân bằng, thử lấy mẫu có kiểm soát và báo hệ quả. Khi kết luận không ổn định qua các cấu hình hợp lý, phải trình bày giới hạn thay vì chỉ chọn cấu hình cho kết quả thuận lợi.

<a id="muc-27"></a>

## 27. Phân biệt kết quả với kết luận

“Từ X liên kết mạnh hơn với seed tích cực” là kết quả thống kê. “Từ X luôn tích cực” là kết luận vượt dữ liệu. “Khách sạn có nhiều từ X nên làm khách hài lòng hơn” còn thêm vấn đề giá trị đo lường và nhân quả.

Muốn nghiên cứu tác động của một dịch vụ, cần thiết kế nhận diện phù hợp và biến kiểm soát; một từ điển tự học từ đánh giá không tự tạo bằng chứng nhân quả cho dịch vụ đó.

<a id="muc-28"></a>

## 28. Cách viết vào bài nghiên cứu

Mẫu có chỗ trống: “Chúng tôi suy ra điểm từ/cụm bằng SO-PMI trên [corpus, thời gian], với ngữ cảnh là [đơn vị]. Seed gồm [danh sách và lý do]. Ứng viên cần [hỗ trợ tối thiểu]; số 0 được xử lý bằng [cách]. Từ điển và ngưỡng được cố định trước khi đánh giá trên [tập độc lập].”

Báo số mục từ trước/sau lọc, độ phủ và kết quả thẩm định. Ghi rõ đây là biến thể đếm corpus nếu không dùng cơ chế truy vấn của công trình PMI-IR gốc. Không chép độ chính xác bài gốc làm kết quả dự án.

<a id="muc-29"></a>

## 29. Khả năng tái lập và tính minh bạch

Lưu seed, từ vựng, số đếm, cách chia tập, quy tắc token/cụm, phiên bản ngữ liệu và mã tính điểm. Từ điển cần có phiên bản gắn với corpus cụ thể; không chỉ lưu cột điểm cuối.

GitHub có thể chứa mã, cấu hình và ví dụ giả định. Với văn bản thật, chỉ chia sẻ khi quyền sử dụng cho phép; nếu không, cung cấp hướng dẫn tạo dữ liệu hoặc các sản phẩm tổng hợp được phép công bố. Không hứa tái lập đầy đủ khi nguồn dữ liệu không thể truy cập.

<a id="muc-30"></a>

## 30. Đạo đức nghiên cứu liên quan

Seed và thành phần corpus có thể ưu tiên một cách diễn đạt hoặc nhóm khách. Kiểm tra chất lượng theo nhóm ngôn ngữ/kiểu viết trước khi diễn giải khác biệt. Tên địa danh, quốc tịch và tên người có thể đi cùng lời chê vì hoàn cảnh thu thập; không biến liên kết đó thành nhãn bản chất của nhóm người.

Ẩn thông tin nhận diện và tránh dùng điểm tự động để quy trách nhiệm cho nhân viên mà không có kiểm tra con người và căn cứ phù hợp.

<a id="muc-31"></a>

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **PMI dương có nghĩa từ mang cảm xúc tích cực không?** Không; đó là liên kết của một cặp. Cần bước so với định hướng seed.
2. **Một từ không xuất hiện có thể được gọi là trung tính không?** Không; trường hợp này thiếu bằng chứng.
3. **Vì sao phải cùng đơn vị mẫu số?** Vì xác suất chung và biên phải mô tả cùng mô hình sự kiện.
4. **Bài tập:** thay $c(cozy,poor)$ từ 2 lên 6, giữ các biên và tổng hợp lệ. Kết quả SO-PMI bằng 0 vì hai mức liên kết với seed bằng nhau. Giải thích vì sao điều này chưa chứng minh từ hoàn toàn không mang cảm xúc.
5. **Bài tập thiết kế:** so đếm theo câu và mệnh đề trên 20 câu hỗn hợp tự soạn; ghi những quan hệ đồng xuất hiện biến mất khi tách mệnh đề.

<a id="muc-32"></a>

## 32. Những điểm còn tranh luận

Các vấn đề cần cân nhắc gồm: phân biệt cảm xúc với chủ đề, mức phụ thuộc seed, xử lý từ hiếm và tính so sánh giữa corpus. Hai từ điển tạo từ hai thời kỳ có điểm khác nhau có thể do ngôn ngữ đổi, nhưng cũng có thể do nền tảng, mẫu khách hoặc tần suất seed đổi.

Nên trình bày thang điểm như một phép vận hành hóa có điều kiện. Muốn kết luận thay đổi cảm xúc theo thời gian, cần kiểm tra độ ổn định phép đo và thành phần ngữ liệu, thay vì coi mọi chênh lệch SO-PMI là thay đổi tâm lý thực.
