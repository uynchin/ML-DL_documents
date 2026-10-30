# Semantic — Mở rộng từ điển cảm xúc bằng embedding

> **Nhánh:** LEXICON BASED → Corpus Based. **Vị trí nguồn:** dòng 10 trong bảng Excel, tên phương pháp “Semantic (mở rộng từ điển bằng embedding)”. **Ngày đối chiếu nguồn:** 28/10/2026.
>
> Đây là một họ cách tiếp cận. Bài dùng baseline so độ gần với seed để tính tay, rồi giới thiệu logic lan truyền trên đồ thị. Ví dụ vector là số tự đặt, không phải vector đo từ Word2Vec, fastText hay SentProp và không tái lập nguyên vẹn một công trình cụ thể.

| Trường thông tin | Nội dung |
|---|---|
| Bản chất | Suy ra điểm hoặc đề xuất mục từ mới từ cấu trúc không gian biểu diễn |
| Đầu vào | Từ điển/seed ban đầu, vector từ/cụm, corpus hoặc nguồn vector và quy tắc chọn ứng viên |
| Đầu ra | Từ điển mở rộng, điểm, nguồn suy ra, hỗ trợ tần suất và các chỉ báo kiểm định |
| Mốc minh họa | Word2Vec, 2013; SentProp, 2016; mô hình subword của fastText, bài TACL 2017 [S1–S3] |
| Tiếng Việt | Có thể làm với corpus/vector tiếng Việt và seed phù hợp; cần nhất quán cách tách từ nhiều âm tiết |
| Nhãn | Có seed hoặc từ điển có định hướng; không nhất thiết cần nhãn từng văn bản để học vector. Đánh giá vẫn cần chuẩn độc lập |
| Tính toán | Tùy nguồn vector và thuật toán. Baseline nhỏ có thể dùng CPU; đồ thị dày và tìm láng giềng toàn bộ có thể tốn nhiều bộ nhớ |
| Mã tham khảo | [SocialSent của tác giả SentProp](https://github.com/williamleif/socialsent); không đồng nhất mã minh họa của bài với gói này |

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

**Embedding** ánh xạ một đơn vị ngôn ngữ thành vector số trong không gian nhiều chiều. Mở rộng từ điển cảm xúc bằng embedding dùng quan hệ giữa các vector và một nguồn định hướng ban đầu để đề xuất hoặc gán điểm cho từ/cụm mới.

Ví dụ học thuật là SentProp: kết hợp word embedding theo miền với lan truyền nhãn để suy ra lexicon từ seed nhỏ [S2]. Tuy nhiên, không phải mọi phép lấy từ gần seed đều là SentProp, và không phải mọi embedding tự chứa một trục cảm xúc đã được xác định.

<a id="muc-2"></a>

## 2. Giải thích đơn giản

Hình dung một bản đồ mà từ ngữ được đặt gần nhau nếu cách sử dụng của chúng tương tự. Bạn đã biết vài địa điểm thuộc phía khen và vài địa điểm thuộc phía chê. Bạn dùng những mốc đó để đánh giá các địa điểm mới.

Điểm khó là “gần” có nhiều nghĩa. Hai từ có thể gần vì cùng nói về dịch vụ, dù một từ khen và một từ chê. Vì thế, lấy hàng xóm của “tốt” rồi gán tất cả là tích cực có thể tạo lỗi.

<a id="muc-3"></a>

## 3. Mục đích và vấn đề giải quyết

Một từ điển ban đầu có thể không có “đáng tiền”, “ổn áp” hoặc cách nói theo miền. Đề xuất nghiên cứu là dùng vector phù hợp để tìm ứng viên mới, thẩm định rồi kiểm tra chất lượng tăng thêm khi phân tích phản hồi khách sạn.

Mục tiêu có thể là tăng độ phủ, giảm lỗi từ miền hoặc khám phá khác biệt cách dùng từ giữa phân khúc. Tăng số lượng mục từ không tự là thành công: cần xem những mục thêm có đúng và có giúp nhiệm vụ cuối hay không.

<a id="muc-4"></a>

## 4. Điều kiện và giả định sử dụng

Cần seed đáng tin, vector của ứng viên/seed có thể so sánh trong cùng không gian và dữ liệu đủ phù hợp miền. Với tiếng Việt, nếu vector lưu `hài_lòng` nhưng dữ liệu đưa vào là hai token `hài`, `lòng`, quy trình tra cứu cần được điều chỉnh nhất quán.

Giả định cần kiểm tra là cấu trúc láng giềng chứa thông tin hữu ích cho sắc thái. Không được mặc định cùng ngữ cảnh đồng nghĩa cùng cực tính. Nếu ghép vector từ hai mô hình độc lập, phải có phương án căn chỉnh hợp lệ trước khi so trực tiếp.

<a id="muc-5"></a>

## 5. Cơ chế hoạt động bên trong

Một quy trình có thể gồm: học/nạp vector → kiểm tra token và seed → chuẩn hóa vector → tính độ gần → suy ra điểm → chọn ứng viên → thẩm định → đóng băng từ điển.

Baseline đơn giản so độ gần trung bình với hai phía seed. Biến thể đồ thị nối các từ theo láng giềng rồi truyền tín hiệu từ seed qua các cạnh. Hệ thống dùng đồ thị cần thêm quyết định về cạnh, trọng số, chuẩn hóa và nút cô lập; không chỉ thay cosine bằng một tên thuật toán khác.

<a id="muc-6"></a>

## 6. Quy trình áp dụng trong nghiên cứu

1. Xác định đơn vị mở rộng: từ đơn, cụm hay nghĩa theo ngữ cảnh.
2. Chuẩn bị corpus hợp lệ và chia tập theo mục tiêu khái quát hóa.
3. Chọn nguồn vector hoặc học vector chỉ từ phần dữ liệu được phép dùng.
4. Chọn seed dương/âm, kiểm tra nghĩa trong miền và tình trạng có vector.
5. Xây baseline và, nếu cần, phương án đồ thị.
6. Chọn cấu hình và ngưỡng trên tập phát triển; thẩm định ứng viên.
7. Đóng băng lexicon và bộ chấm văn bản.
8. So từ điển gốc với bản mở rộng trên cùng tập kiểm tra độc lập.

Nếu đưa nhãn sao hoặc emoji vào cách học điểm, cần khai báo nguồn giám sát đó; không gọi toàn bộ quy trình là hoàn toàn không có thông tin nhãn.

<a id="muc-7"></a>

## 7. Ví dụ minh họa xuyên suốt

Giả sử có seed tích cực “hài_lòng”, tiêu cực “thất_vọng” và ứng viên “đáng_tiền”. Cần xem “đáng_tiền” gần phía nào hơn trong vector phù hợp miền khách sạn.

Sau đó, thử câu “Phòng không đáng tiền nhưng nhân viên rất tận tình”. Dù từ điển có điểm đúng cho hai biểu thức, bộ chấm vẫn cần xử lý phủ định và gắn ý kiến với từng khía cạnh. Mở rộng từ điển giải bài toán thiếu mục từ, không tự giải toàn bộ nghĩa của câu.

<a id="muc-8"></a>

## 8. Kết quả và cách diễn giải

Đầu ra thường gồm ứng viên, điểm theo quy tắc suy ra, các seed/láng giềng liên quan và trạng thái chấp nhận. Điểm cosine là độ tương tự hình học; hiệu hai phía seed là điểm tương đối của thiết kế, không phải xác suất hoặc độ hài lòng của khách.

Điểm gần 0 có thể do gần cả hai phía, xa cả hai phía hoặc vector không hữu ích cho phân biệt cảm xúc. Cần lưu từng thành phần thay vì chỉ điểm chênh lệch.

<a id="muc-9"></a>

## 9. Điểm mạnh, điểm yếu và giới hạn

**Điểm mạnh:** có thể tìm ứng viên theo cấu trúc phân bố, hỗ trợ thích nghi miền và mở rộng từ một nguồn định hướng nhỏ. Dấu vết seed/láng giềng giúp kiểm tra các mục được đề xuất.

**Điểm yếu:** từ trái nghĩa có thể gần nhau; một vector tĩnh có thể gộp nhiều nghĩa; sai seed dễ lan ra ứng viên. Corpus lệch hoặc nhỏ có thể tạo vector không ổn định. Khả năng tạo vector cho từ mới cũng không bảo đảm điểm cảm xúc của từ đó chính xác.

<a id="muc-10"></a>

## 10. So sánh với phương pháp gần nhau

| Phương án | Điểm khác cần hiểu |
|---|---|
| SO-PMI trực tiếp | So liên kết thống kê trực tiếp với seed trong định nghĩa ngữ cảnh đã chọn |
| Cosine với seed | So vector; có thể dùng tín hiệu gián tiếp từ cấu trúc phân bố |
| Lan truyền trên đồ thị | Dùng thêm đường liên kết qua các từ trung gian |
| Bộ phân loại dùng embedding | Dự đoán nhãn văn bản; không nhất thiết xuất ra từ điển |
| Tra đồng nghĩa WordNet | Dùng quan hệ từ vựng biên soạn sẵn thay vì chỉ hình học vector |

“Semantic” và “statistical” không tách biệt tuyệt đối. Vector có thể được xây từ ma trận đồng xuất hiện; thay đổi nằm ở biểu diễn và cách suy ra điểm.

<a id="muc-11"></a>

## 11. Sai lầm thường gặp và cách khắc phục

- Gán cùng cực tính cho mọi láng giềng: kiểm tra đối chứng với seed âm và đọc ngữ cảnh.
- So vector khác không gian: dùng cùng mô hình hoặc căn chỉnh có kiểm định.
- Dùng từ trong tập đánh giá lexicon làm seed: loại seed khỏi đánh giá ứng viên mới và công bố phần chồng lấn.
- Tự nhận mở rộng tốt vì coverage tăng: phải đo độ đúng và nhiệm vụ cuối.
- Dùng cùng ký hiệu “seed” cho từ hạt giống và random seed mà không phân biệt: ghi rõ hai loại trong cấu hình.

<a id="muc-12"></a>

## 12. Tài liệu tham khảo

- **[S1]** Mikolov, T., Chen, K., Corrado, G., & Dean, J. (2013). *Efficient Estimation of Word Representations in Vector Space*. [arXiv:1301.3781](https://arxiv.org/abs/1301.3781).
- **[S2]** Hamilton, W. L., Clark, K., Leskovec, J., & Jurafsky, D. (2016). *Inducing Domain-Specific Sentiment Lexicons from Unlabeled Corpora*. EMNLP, 595–605. [Bài báo](https://aclanthology.org/D16-1057/). [DOI](https://doi.org/10.18653/v1/D16-1057).
- **[S3]** Bojanowski, P., Grave, E., Joulin, A., & Mikolov, T. (2017). *Enriching Word Vectors with Subword Information*. TACL, 5, 135–146. [ACL Anthology](https://aclanthology.org/Q17-1010/).
- **[S4]** Hamilton và cộng sự. [SocialSent: mã và hướng dẫn của dự án](https://github.com/williamleif/socialsent).

S1 và S3 giới thiệu cách biểu diễn; S2 minh họa suy ra lexicon theo miền; S4 là mã nghiên cứu tham khảo. Baseline cosine, ví dụ khách sạn và sơ đồ tính tổng quát trong bài này không được trình bày như bản tái lập đầy đủ SentProp.

<a id="muc-13"></a>

## 13. Vị trí trong toàn bộ nghiên cứu

Phương pháp nằm ở giai đoạn xây tài nguyên hoặc tạo đặc trưng. Sau đó có thể dùng lexicon trong phân loại, mô tả hay mô hình giải thích.

Nếu cùng corpus được dùng để xây từ điển và mô tả chính corpus đó, cần nói rõ tính phụ thuộc. Muốn chứng minh khả năng áp dụng sang dữ liệu mới, nên dành dữ liệu thời gian/khách sạn khác cho bước đánh giá cuối.

<a id="muc-14"></a>

## 14. Kiến thức nền cần biết trước

Cần hiểu vector, tích vô hướng, chuẩn vector, cosine, láng giềng gần và seed. Với đồ thị, cần biết nút là từ/cụm, cạnh là quan hệ được chọn, trọng số biểu diễn mức liên kết.

Phân biệt **embedding tĩnh** — thường một vector cho một mục từ — và **embedding theo ngữ cảnh** — biểu diễn thay đổi theo lần xuất hiện. Nếu dùng loại theo ngữ cảnh để xây lexicon, phải thêm quy tắc chọn và tổng hợp các lần xuất hiện.

<a id="muc-15"></a>

## 15. Nền tảng lý thuyết và logic hình thành

Logic thực hành là cách một từ được sử dụng có thể cung cấp thông tin về quan hệ ngữ nghĩa, trong đó có phần liên quan sắc thái. Seed định hướng vùng không gian để ta suy ra một chỉ báo cảm xúc.

Nhưng tương tự ngữ nghĩa rộng hơn cùng cảm xúc. “Sạch” và “bẩn” cùng nói về vệ sinh và có thể đi vào các khung câu tương tự. Vì vậy, giả định láng giềng có nhãn tương tự cần được kiểm tra theo mô hình/miền, không coi là một định luật cho mọi vector.

<a id="muc-16"></a>

## 16. Các thành phần cấu tạo

Một hệ gồm nguồn văn bản/vector, vocabulary, seed, phép tương tự, phép suy ra điểm, quy tắc chọn mục mới và bộ kiểm định. Biến thể đồ thị thêm cơ chế dựng cạnh và lan truyền.

Nên lưu `candidate → seed/neighbor → component_score → final_score` để kiểm tra. Nếu hệ thống chỉ xuất một số cuối mà không giữ nguồn, sẽ khó phân biệt ứng viên sai do vector, seed hay cách kết hợp.

<a id="muc-17"></a>

## 17. Các loại và biến thể

Có thể dùng Word2Vec, vector ma trận rút gọn, fastText hoặc biểu diễn theo ngữ cảnh; chúng là lựa chọn biểu diễn, không cùng một thuật toán suy ra cảm xúc. Mô hình subword của fastText dùng thông tin character n-gram để biểu diễn từ [S3].

Về suy ra điểm, có thể dùng trung bình cosine với seed, so centroid hoặc lan truyền đồ thị. Công trình SentProp có thiết kế riêng, bao gồm vector/đồ thị và cơ chế lan truyền [S2]; không được dùng công thức minh họa bên dưới để khẳng định đã tái lập bài đó.

<a id="muc-18"></a>

## 18. Đầu vào và đầu ra

Đầu vào cần lưu ngôn ngữ, đơn vị token, phiên bản mô hình, vector dimension, seed hai phía và tập ứng viên. Đầu ra nên gồm `term`, `frequency`, `score_positive`, `score_negative`, `score_difference`, `accepted`, `source`, `review_status`.

Với OOV — mục ngoài vocabulary — ghi riêng trạng thái. Nếu mô hình subword tạo được vector cho OOV, vẫn cần ghi nguồn và đánh giá chất lượng; có vector là điều kiện kỹ thuật, không phải bằng chứng từ đã được hiểu đúng.

<a id="muc-19"></a>

## 19. Căn cứ lựa chọn

Nên thử khi lexicon ban đầu thiếu từ miền và có vector phù hợp hoặc đủ corpus để xây. Trước khi mở rộng hàng nghìn mục, kiểm tra một mẫu: seed có nghĩa đúng không, láng giềng có cùng chủ đề hay cùng cực tính, điểm có ổn định khi đổi seed không?

Nếu cần hệ thống minh bạch, baseline cosine có thể là bước xuất phát dễ kiểm tra. Đồ thị chỉ đáng thêm khi lợi ích đo được phù hợp chi phí và mục tiêu.

<a id="muc-20"></a>

## 20. Trường hợp không nên sử dụng

Không nên gán điểm tự động khi seed cực ít, đa nghĩa mạnh hoặc ứng viên chủ yếu là tên người/địa danh. Nếu nguồn vector khác hẳn miền, cần pilot trước; không mặc định vector lớn hơn luôn tốt hơn.

Với nhiệm vụ câu cần hiểu phủ định, mỉa mai và đối tượng, lexicon mở rộng vẫn chỉ là một thành phần. Có thể chuyển sang mô hình ngữ cảnh hoặc kết hợp luật, nhưng phải so trên cùng tập kiểm tra và mô tả đúng nhiệm vụ.

<a id="muc-21"></a>

## 21. Các quyết định quan trọng khi thực hiện

Chọn vector theo miền hay pretrained; lọc tần suất thế nào; tính trung bình trên seed hay dùng centroid; chọn bao nhiêu láng giềng; duyệt thủ công phần nào đều ảnh hưởng sản phẩm.

**Trung bình cosine từng seed không luôn bằng cosine với centroid** sau chuẩn hóa. Nếu đổi phép tính, phải xem như đổi phương pháp. Với mở rộng nhiều vòng, có nên đưa từ mới thành seed không là quyết định quan trọng: làm vậy có thể tăng bao phủ và đồng thời khuếch đại lỗi.

<a id="muc-22"></a>

## 22. Giải thích công thức và ký hiệu

Với hai vector khác 0:

$$\cos(\mathbf{x},\mathbf{y})=
\frac{\mathbf{x}^{\mathsf T}\mathbf{y}}{\|\mathbf{x}\|_2\|\mathbf{y}\|_2}.$$

Baseline của bài với seed dương $S_+$ và âm $S_-$ là:

$$a_+(w)=\frac{1}{|S_+|}\sum_{s\in S_+}\cos(\mathbf{v}_w,\mathbf{v}_s),$$

$$a_-(w)=\frac{1}{|S_-|}\sum_{s\in S_-}\cos(\mathbf{v}_w,\mathbf{v}_s),
\qquad score(w)=a_+(w)-a_-(w).$$

Trong thiết kế này, điểm nằm trong $[-2,2]$; không tự chuyển thành xác suất. Hai tập seed phải không rỗng và các vector dùng phải hợp lệ. Có thể dùng ngưỡng $\tau$ để chọn ứng viên dương/âm và vùng chưa quyết định, nhưng $\tau$ phải được chọn trên tập phát triển.

Để hiểu **một cơ chế lan truyền điểm tổng quát**, có thể xét:

$$\mathbf{f}^{(t+1)}=(1-\alpha)\mathbf{y}+\alpha P\mathbf{f}^{(t)},
\qquad 0\le\alpha<1.$$

$P$ là ma trận trọng số không âm, tổng mỗi hàng bằng 1; $\mathbf{y}$ chứa tín hiệu seed cố định. Với quy ước này, mỗi nút nhận trung bình có trọng số của điểm hàng xóm và được kéo về tín hiệu gốc. Nút không có cạnh cần quy tắc riêng, chẳng hạn self-loop. Đây là phương trình giảng giải chung, **không phải khẳng định công thức đầy đủ của SentProp**. Không đưa cosine âm trực tiếp vào $P$ rồi gọi nó là ma trận xác suất.

<a id="muc-23"></a>

## 23. Ví dụ tính toán hoặc thao tác từng bước

Giả định vector hai chiều: seed dương $(1,0)$, seed âm $(-1,0)$, ứng viên “đáng_tiền” $(0.8,0.6)$. Các chuẩn đều bằng 1, nên hai cosine là $0.8$ và $-0.8$, điểm chênh lệch bằng $1.6$.

Đây là hình học tự đặt để hiểu công thức, không khẳng định cảm xúc thật có trục hai chiều như vậy. Chương trình chỉ dùng thư viện chuẩn Python:

```python
from math import sqrt, isclose

# Vector giả định; không phải kết quả học từ corpus.
vectors = {
    "hai_long": (1.0, 0.0),
    "that_vong": (-1.0, 0.0),
    "dang_tien": (0.8, 0.6),
    "mo_ho": (0.0, 1.0),
}

def cosine(x, y):
    if len(x) != len(y):
        raise ValueError("Khác số chiều")
    nx = sqrt(sum(v * v for v in x))
    ny = sqrt(sum(v * v for v in y))
    if nx == 0 or ny == 0:
        raise ValueError("Vector 0 không có cosine hợp lệ")
    return sum(a * b for a, b in zip(x, y)) / (nx * ny)

def infer(term, positive, negative):
    if not positive or not negative:
        raise ValueError("Cần seed cho cả hai phía")
    required = [term, *positive, *negative]
    if any(w not in vectors for w in required):
        return None
    vp = sum(cosine(vectors[term], vectors[s]) for s in positive) / len(positive)
    vn = sum(cosine(vectors[term], vectors[s]) for s in negative) / len(negative)
    return {"positive": vp, "negative": vn, "score": vp - vn}

result = infer("dang_tien", ["hai_long"], ["that_vong"])
assert isclose(result["score"], 1.6)
assert isclose(infer("mo_ho", ["hai_long"], ["that_vong"])["score"], 0.0)
assert infer("khong_co", ["hai_long"], ["that_vong"]) is None
print({k: round(v, 4) for k, v in result.items()})
# {'positive': 0.8, 'negative': -0.8, 'score': 1.6}
```

Để dùng thật, cần nạp vector có nguồn rõ, kiểm tra seed, tần suất và xử lý OOV. Mã trên không huấn luyện Word2Vec, không xây đồ thị và không đo chất lượng từ điển.

<a id="muc-24"></a>

## 24. Tình huống sai và phản ví dụ

Nếu “sạch” và “bẩn” gần nhau vì cùng xuất hiện trong câu mô tả phòng, lấy mọi láng giềng “sạch” làm từ tích cực sẽ sai. Một cách kiểm tra là so ứng viên với cả hai phía seed và đọc câu chứa nó.

Nhưng hiệu hai phía cũng có thể không phân biệt được nếu vector biểu diễn hai phía quá giống nhau. Khi đó cần thay biểu diễn, thêm thông tin cảm xúc hoặc giữ trạng thái chưa quyết định. Không nên ép mọi ứng viên có nhãn chỉ để làm lexicon lớn hơn.

<a id="muc-25"></a>

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đề xuất kiểm tra **mục từ mới** bằng thẩm định có ngữ cảnh, precision@k hoặc F1 theo bộ nhãn được thiết kế. Không để seed trong tập kiểm tra rồi nhận kết quả dễ dự đoán đó là khả năng khái quát.

Ở **nhiệm vụ cuối**, so lexicon gốc và mở rộng bằng cùng bộ chấm, ngưỡng được lựa chọn công bằng và cùng tập test; báo macro-F1, từng lớp và coverage. Với vector được huấn luyện ngẫu nhiên, lặp các random seed hợp lý; đồng thời đổi mẫu corpus hoặc tập từ hạt giống để kiểm tra các nguồn bất định khác. Không gộp chúng thành một loại sai số duy nhất.

<a id="muc-26"></a>

## 26. Xử lý khi không đáp ứng yêu cầu

OOV cao: xem tách từ và nguồn vector trước. Ứng viên sai cực tính: kiểm tra các seed và láng giềng. Lỗi tăng theo vòng mở rộng: giữ seed gốc, thêm thẩm định hoặc giới hạn số vòng để kiểm tra semantic drift — sự lệch dần khỏi mục tiêu ban đầu.

Nếu coverage tăng mà F1 giảm, xem các mục mới tạo false positive ở đâu; có thể dùng ngưỡng chọn chặt hơn hoặc trọng số khác trên tập phát triển. Không xóa các từ gây lỗi sau khi xem test rồi báo lại test đó như bằng chứng độc lập.

<a id="muc-27"></a>

## 27. Phân biệt kết quả với kết luận

“Ứng viên gần seed dương hơn seed âm” là kết quả hình học theo mô hình. “Ứng viên tích cực trong mọi ngữ cảnh” chưa được chứng minh. “Nhóm khách này hài lòng hơn” còn đòi hỏi xác nhận phép đo và mẫu so sánh.

Nếu so điểm từ điển giữa thời kỳ, không so trực tiếp tọa độ vector từ hai mô hình độc lập. Dù đã có phép căn chỉnh, khác biệt điểm vẫn có thể do dữ liệu, seed và cấu hình thay vì thay đổi tâm lý thật.

<a id="muc-28"></a>

## 28. Cách viết vào bài nghiên cứu

Khung báo cáo: “Chúng tôi mở rộng lexicon [nguồn] bằng vector [mô hình, corpus, số chiều]. Seed gồm [danh sách/cách chọn]. Điểm ứng viên tính bằng [công thức], chọn với [ngưỡng/hỗ trợ]. Các quyết định được xác lập trên [tập phát triển], sau đó đánh giá trên [tập kiểm tra].”

Với đồ thị, báo thêm số láng giềng, cách đối xứng hóa nếu có, trọng số, chuẩn hóa, xử lý nút cô lập, điều kiện dừng và nguồn mã. Nếu chỉ dùng baseline cosine, gọi đúng tên baseline, không nhận đã tái lập SentProp.

<a id="muc-29"></a>

## 29. Khả năng tái lập và tính minh bạch

Lưu mô hình vector hoặc đường dẫn/version/checksum hợp lệ; corpus, seed, vocabulary và cấu hình suy ra điểm. Ghi rõ thư viện, random seed, số luồng và các nguồn biến thiên không kiểm soát được nếu huấn luyện.

Kho GitHub nên có mã, ví dụ nhỏ, hướng dẫn lấy tài nguyên và nhật ký các mục thêm. Mã SocialSent là tài liệu nghiên cứu tham khảo [S4]; cần kiểm tra môi trường tương thích trước khi dùng, không hứa chạy ngay chỉ từ việc kho tồn tại. Không phân phối lại corpus/vector khi chưa có quyền tương ứng.

<a id="muc-30"></a>

## 30. Đạo đức nghiên cứu liên quan

Vector học từ ngôn ngữ có thể mang liên kết thiên lệch với giới, vùng miền và nhóm xã hội. Không gán điểm tiêu cực cho tên một cộng đồng rồi diễn giải đó là thuộc tính cộng đồng. Cần xem ứng viên nào nên bị loại khỏi lexicon cảm xúc vì chúng là đối tượng được đánh giá, không phải biểu thức ý kiến.

Trong phản hồi khách sạn, bảo vệ thông tin nhận diện và kiểm tra sai số theo kiểu diễn đạt của các nhóm khách. Việc dùng phép tính tự động không loại được trách nhiệm lựa chọn và diễn giải dữ liệu.

<a id="muc-31"></a>

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **Cosine cao có chắc cùng chiều cảm xúc không?** Không; có thể chỉ cùng chủ đề/ngữ cảnh.
2. **Điểm 1.6 của ví dụ có phải xác suất 160% không?** Không; đó là hiệu hai cosine trong thang $[-2,2]$.
3. **Có vector cho OOV có bảo đảm từ được chấm đúng không?** Không; vẫn cần thẩm định.
4. **Bài tập tính:** thay vector ứng viên bằng $(0.6,0.8)$. Với seed giữ nguyên, hai cosine là $0.6,-0.6$, hiệu bằng $1.2$.
5. **Bài tập thiết kế:** lấy 20 ứng viên có điểm cao và 20 gần 0, đọc ngữ cảnh rồi phân loại nguyên nhân không chắc: đa nghĩa, gần hai phía, xa hai phía, vector kém hoặc thiếu hỗ trợ. Mẫu này dùng để pilot, không đại diện cho toàn bộ từ vựng.

<a id="muc-32"></a>

## 32. Những điểm còn tranh luận

Các vấn đề chính là quan hệ giữa tương tự ngữ nghĩa và tương tự cảm xúc, mức phụ thuộc seed, lựa chọn vector theo miền và tính so sánh giữa nhóm/thời gian. Lexicon có thể minh bạch ở bước chấm nhưng nguồn vector vẫn phức tạp và khó giải thích.

Mở rộng nhiều hơn, có nhiều láng giềng hơn hoặc dùng vector lớn hơn không tự bảo đảm tốt hơn. Cần xác định trước chất lượng nào quan trọng: độ đúng mục mới, coverage, hiệu năng nhiệm vụ cuối hay khả năng diễn giải; rồi báo riêng các tiêu chí và sự đánh đổi.
