# Emoji và emoticon — Giữ và diễn giải tín hiệu biểu cảm

> **Nhánh:** LEXICON BASED → Xử lý quy tắc. **Nguồn đề mục:** dòng 13 trong bảng Excel. **Ngày đối chiếu:** 28/10/2026. Điểm và bảng biểu tượng trong ví dụ được tự đặt, không sao chép Emoji Sentiment Ranking và không phải bảng nghĩa phổ quát.

| Trường | Nội dung |
|---|---|
| Bản chất | Nhận diện, biểu diễn và xử lý tín hiệu biểu cảm trong văn bản |
| Đầu vào | Văn bản Unicode gốc, biểu tượng/cụm ký tự, ngữ cảnh và tài nguyên nếu dùng |
| Đầu ra | Các span nhận diện, dạng biểu tượng, đặc trưng/điểm và dấu vết xử lý |
| Mốc tham khảo | Kralj Novak và cộng sự, 2015 về lexicon sentiment emoji [S1]; không phải năm emoji ra đời |
| Tiếng Việt | Ký tự không phụ thuộc chữ viết, nhưng cách dùng và sắc thái cần kiểm định trên cộng đồng/ngôn ngữ mục tiêu |
| Nhãn | Tra từ điển không cần huấn luyện trên nhãn dự án; học nghĩa theo ngữ cảnh và kiểm định cần dữ liệu phù hợp |
| Tính toán | Tra cứu nhỏ dùng CPU; xử lý Unicode và mô hình ngữ cảnh có yêu cầu khác |

## 1. Định nghĩa học thuật

**Emoji** gồm các ký tự và chuỗi Unicode có thể được hiển thị dưới dạng biểu tượng; chuẩn Unicode quy định cấu trúc và các chuỗi liên quan [S2]. **Emoticon** là biểu thức tạo từ ký tự như `:)`, `:-(` hoặc các tổ hợp chữ/dấu khác.

Xử lý hai loại này trong sentiment gồm nhận diện đơn vị đúng, giữ hoặc chuyển sang biểu diễn phù hợp, rồi mô hình hóa quan hệ với nội dung. Chuẩn mã hóa không phải từ điển cảm xúc và tên mô tả biểu tượng không tự là nhãn sentiment.

## 2. Giải thích đơn giản

Một phản hồi “Khách sạn tuyệt vời 😍” có thêm tín hiệu ngoài chữ. Xóa biểu tượng làm mất một phần cách người viết bày tỏ. Nhưng “Chờ hai tiếng mới được phòng 🙂” có thể mang ý châm biếm, nên gán 🙂 là vui trong mọi câu cũng có nguy cơ sai.

Cần hai câu hỏi riêng: máy có đọc đúng biểu tượng không, và máy có diễn giải đúng vai trò trong câu không? Giải quyết câu hỏi đầu chưa bảo đảm giải quyết câu hỏi sau.

## 3. Mục đích và vấn đề giải quyết

Đề xuất nghiên cứu khách sạn: so chất lượng mô hình khi xóa, giữ nguyên hoặc chuyển emoji/emoticon thành đặc trưng. Có thể khảo sát thêm phản hồi chỉ có biểu tượng hoặc biểu tượng mâu thuẫn với chữ.

Mục tiêu không phải chứng minh giữ mọi biểu tượng luôn tốt hơn. Một mô hình đã học cách dùng chúng có thể khác mô hình từ điển; cần so đúng nguồn biểu diễn và nhiệm vụ cuối.

## 4. Điều kiện và giả định sử dụng

Cần đọc/ghi UTF-8, bảo toàn bản gốc và xác định phiên bản tài nguyên nhận diện Unicode nếu dùng. Một biểu tượng nhìn như một hình có thể gồm nhiều code point; tách từng code point tùy tiện có thể làm sai đơn vị [S2].

Nếu dùng lexicon sentiment, giả định điểm từ miền/ngôn ngữ nguồn còn phù hợp với đánh giá khách sạn cần được kiểm tra. Biểu tượng mới hoặc không có điểm phải được ghi thiếu bằng chứng, không tự gán trung tính.

## 5. Cơ chế hoạt động bên trong

Quy trình đề xuất: đọc văn bản gốc → nhận chuỗi emoji và emoticon → lưu span → chuẩn hóa có kiểm soát nếu cần → tạo đặc trưng/tra điểm → xét ngữ cảnh → kết hợp với chữ.

Một bộ nhận diện cần tránh đếm vừa một chuỗi hoàn chỉnh vừa các thành phần bên trong nếu thiết kế không cho phép. Nhận diện emoticon cũng cần ngữ cảnh: `:)` trong phản hồi khác ký tự tương tự xuất hiện trong mã, đường dẫn hoặc dữ liệu kỹ thuật.

## 6. Quy trình áp dụng trong nghiên cứu

1. Xác định cần sentiment, cảm xúc cụ thể hay đặc trưng hình thức.
2. Giữ raw text, kiểm tra mã hóa và lỗi trích xuất.
3. Chọn bộ nhận diện/tài nguyên và phiên bản; liệt kê loại được bao phủ.
4. Gán nhãn mẫu có biểu tượng, không có biểu tượng và biểu tượng mâu thuẫn chữ.
5. Tạo các biến thể xử lý trên cùng phép chia dữ liệu.
6. Chọn cách kết hợp, trọng số và xử lý biểu tượng không biết trên tập phát triển.
7. Đánh giá nhận diện lẫn nhiệm vụ cuối, kiểm tra lỗi theo biểu tượng và nền tảng.

Không để emoji làm nhãn huấn luyện đồng thời xuất hiện nguyên xi trong test rồi diễn giải kết quả như hệ đã hiểu sentiment độc lập với nhãn đó.

## 7. Ví dụ minh họa xuyên suốt

So ba phản hồi tự soạn: “Phòng đẹp 🙂”, “Chờ hai tiếng 🙂” và “🙂”. Biểu tượng giống nhau nhưng vai trò có thể khác: bổ sung lời khen, đánh dấu mỉa mai, hoặc là toàn bộ thông điệp thiếu ngữ cảnh.

Cần giữ nội dung chữ và trạng thái ngữ cảnh. Không thể kết luận tâm trạng thực của khách từ biểu tượng duy nhất, đặc biệt khi người gán nhãn cũng không đủ thông tin.

## 8. Kết quả và cách diễn giải

Đầu ra kỹ thuật có thể là vị trí, chuỗi gốc, loại và số lần xuất hiện. Đầu ra sentiment có thể là điểm lexicon hoặc nhãn theo ngữ cảnh, tùy hệ thống.

“Có hai emoji được chấm dương” không có nghĩa người viết vui gấp đôi người dùng một emoji. Cần phân biệt **không có biểu tượng**, **có biểu tượng chưa nhận diện**, **đã nhận diện nhưng chưa có điểm**, và **có điểm 0**. Bốn trạng thái không thay thế nhau.

## 9. Điểm mạnh, điểm yếu và giới hạn

Giữ biểu tượng giúp bảo toàn tín hiệu mà chữ không thể hiện đầy đủ, nhất là phản hồi ngắn. Đặc trưng số lần/vị trí cũng có thể kiểm tra bằng tay.

Hạn chế là biểu tượng đa nghĩa, mỉa mai, khác cách hiển thị hoặc khác cách dùng theo cộng đồng. Một lexicon cố định khó mô tả toàn bộ vai trò. Ngược lại, dùng mô hình ngữ cảnh có thể tốn dữ liệu và vẫn sai ở các biểu tượng/kiểu dùng mới.

## 10. So sánh với phương pháp gần nhau

| Phương án | Tác động cần kiểm tra |
|---|---|
| Xóa toàn bộ | Đơn giản nhưng có thể mất thông tin |
| Giữ chuỗi gốc | Bảo toàn biểu đạt; mô hình cần xử lý được |
| Đổi sang tên mô tả | Dễ đưa vào pipeline chữ, nhưng có thể áp đặt nghĩa và tăng token |
| Tra lexicon | Minh bạch, giới hạn ở phạm vi và ngữ cảnh tài nguyên |
| Mô hình theo ngữ cảnh | Có thể học tương tác chữ–biểu tượng; không bảo đảm đúng tự động |

Không coi tên như “smiling face” là bằng chứng rằng câu chứa biểu tượng chắc chắn tích cực.

## 11. Sai lầm thường gặp và cách khắc phục

- Xóa mọi ký tự ngoài chữ/số: kiểm tra trước thông tin bị mất.
- Tách theo từng code point và gọi mỗi phần là một emoji: dùng bộ nhận chuỗi phù hợp.
- Bỏ modifier màu da hoặc ký tự nối mà không lưu bản gốc: bảo toàn và chỉ chuẩn hóa khi có lý do.
- Đếm cùng biểu tượng hai lần qua nguyên bản và tên mô tả: xác định một chính sách biểu diễn/đếm.
- Gán nghĩa cố định cho biểu tượng trong mỉa mai: thêm đánh giá ngữ cảnh và cho phép chưa chắc.

## 12. Tài liệu tham khảo

- **[S1]** Kralj Novak, P., Smailović, J., Sluban, B., & Mozetič, I. (2015). *Sentiment of Emojis*. PLOS ONE, 10(12), e0144296. [Bài báo](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0144296). [DOI](https://doi.org/10.1371/journal.pone.0144296).
- **[S2]** Unicode Consortium. *Unicode Technical Standard #51: Unicode Emoji*. [Chuẩn chính thức](https://www.unicode.org/reports/tr51/).
- **[S3]** Hutto, C. J., & Gilbert, E. (2014). *VADER: A Parsimonious Rule-Based Model for Sentiment Analysis of Social Media Text*. [ICWSM](https://ojs.aaai.org/index.php/ICWSM/article/view/14550).

S1 minh họa nghiên cứu lexicon cảm xúc; S2 quy định cấu trúc Unicode, không gán điểm tâm lý; S3 là ví dụ hệ xử lý văn bản mạng xã hội. Các thiết kế và hệ số giảng giải trong bài là nội dung tự xây, không phải tái lập ba nguồn này.

## 13. Vị trí trong toàn bộ nghiên cứu

Thành phần cần bắt đầu ngay khi đọc dữ liệu để tránh mất chuỗi trước khi mô hình chạy. Sau đó, biểu tượng có thể được giữ làm token, đặc trưng riêng hoặc đưa vào mô hình kết hợp.

Nếu bước thu thập đã thay biểu tượng bằng dấu hỏi, điều chỉnh sentiment ở cuối không khôi phục được bản gốc. Cần kiểm tra đường truyền dữ liệu từ thu thập đến xuất kết quả và phân biệt lỗi dữ liệu với lỗi mô hình.

## 14. Kiến thức nền cần biết trước

Cần phân biệt byte, code point, grapheme/đơn vị hiển thị và token của mô hình. `len()` của chuỗi Python không phải lúc nào cũng là số hình mà người đọc nhìn thấy.

Một emoji có thể có modifier hoặc chuỗi nối ZWJ; emoticon lại là chuỗi ký tự văn bản. Bộ tokenizer chung không nhất thiết nhận diện cả hai đúng. Cần xác định offset đang tính theo chuỗi Python hay đơn vị khác khi ghép span với dữ liệu gốc.

## 15. Nền tảng lý thuyết và logic hình thành

Biểu tượng có thể biểu đạt, nhấn mạnh, điều chỉnh hoặc tạo quan hệ với nội dung chữ. Vai trò không giới hạn ở đặt một nhãn cảm xúc lên câu.

Trong nghiên cứu, có thể xem tín hiệu biểu tượng và chữ là hai nguồn thông tin cần kết hợp. Khi chúng mâu thuẫn, đó là hiện tượng cần phân tích chứ không tự động chọn phía nào thắng. Mỉa mai chỉ là một khả năng; còn có trích dẫn, thói quen viết hoặc thiếu ngữ cảnh.

## 16. Các thành phần cấu tạo

Một hệ cần bộ đọc mã hóa, nhận diện emoji/emoticon, tài nguyên biểu diễn, luật vị trí/lặp, bộ kết hợp chữ và bộ kiểm định. Nếu dùng lexicon, lưu nguồn điểm và phạm vi của nó.

Với thay thế sang token tên, cần một lớp riêng để biết từ nào là chữ người viết và từ nào hệ thống chèn. Nếu không, từ chèn có thể bị chấm sentiment như từ gốc rồi lại cộng thêm điểm biểu tượng, tạo đếm trùng.

## 17. Các loại và biến thể

Có thể phân biệt emoji ký tự đơn, chuỗi presentation/modifier, chuỗi ZWJ và các dạng khác theo nguồn nhận diện; bên cạnh đó là emoticon như `:)` và các dạng kaomoji. Phạm vi bộ nhận diện phải được ghi rõ.

Về sentiment, có tra điểm tĩnh, thống kê từ corpus và mô hình theo ngữ cảnh. Một tài nguyên từ giai đoạn trước không tự bao phủ các biểu tượng được thêm sau đó. Nên kiểm tra coverage theo phiên bản thay vì gọi chung là “hỗ trợ emoji”.

## 18. Đầu vào và đầu ra

Đề xuất lưu `raw_text`, `symbol_span`, `raw_symbol`, `symbol_type`, `normalized_symbol`, `lexicon_score`, `score_available`, `detector_version`, cùng ID câu và khía cạnh nếu có.

Lưu số lần và thứ tự để phân biệt “🙂🙂🙂” với một lần xuất hiện; nhưng cách chuyển số lần thành cường độ là lựa chọn riêng. Không suy ra điểm cảm xúc từ màu da hoặc giới của biểu tượng chỉ vì chúng là modifier kỹ thuật.

## 19. Căn cứ lựa chọn

Đo tỷ lệ phản hồi chứa biểu tượng, loại phổ biến và tần suất lỗi do xóa/không nhận diện. Nếu đa số dữ liệu không có biểu tượng, vẫn cần xem chúng tập trung ở nhóm phản hồi quan trọng nào.

Chọn biểu diễn dựa trên mô hình cuối: một mô hình từ điển có thể cần tra/đổi token, trong khi một mô hình ngữ cảnh có thể đã được huấn luyện với chuỗi gốc. So sánh bằng thí nghiệm, không mặc định bước chuẩn hóa nào cũng có lợi.

## 20. Trường hợp không nên sử dụng

Không dùng điểm emoji đơn độc để xác định tâm lý, ý định hoặc sự trung thực của khách. Không xóa toàn bộ biểu tượng mặc định nếu mục tiêu là nghiên cứu cách biểu đạt.

Không dùng bộ nhận diện hạn chế vài ký tự như ví dụ trong bài để tuyên bố bao phủ Unicode. Nếu phải xử lý toàn bộ kho dữ liệu, cần thư viện/tài nguyên phù hợp và kiểm tra phiên bản, chuỗi mới, các trường hợp không nhận diện.

## 21. Các quyết định quan trọng khi thực hiện

Quyết định giữ hay đổi biểu tượng, có gộp biến thể không, xử lý lặp ra sao và kết hợp với chữ thế nào. Nếu gộp, giữ mapping để quay lại bản gốc. Nếu tính theo số lần, kiểm tra các chuỗi lặp dài có lấn át chữ không.

Vị trí cuối câu có thể liên quan vai trò biểu tượng nhưng không phải luật phổ quát. Khi emoji nằm giữa hai mệnh đề hoặc trong trích dẫn, cần tránh gắn nó với đối tượng sai.

## 22. Giải thích công thức và ký hiệu

Một baseline tự chọn với các biểu tượng **có điểm** $E(d)$ là:

$$S_e(d)=\frac{1}{|E(d)|}\sum_{e\in E(d)}v(e),\qquad |E(d)|>0.$$

$v(e)$ là điểm lexicon, đếm theo lần xuất hiện trong thiết kế này. Nếu không có biểu tượng có điểm, trả trạng thái thiếu chứ không tự khẳng định trung tính. Một thiết kế kết hợp có thể là:

$$S(d)=\lambda S_t(d)+(1-\lambda)S_e(d),\qquad0\le\lambda\le1.$$

Chỉ dùng phép kết hợp này khi hai điểm đã được đưa về thang có ý nghĩa so sánh và có chính sách cho thành phần thiếu. $\lambda$ cần chọn trên tập phát triển. Đây không phải công thức bắt buộc của Unicode hay của S1; nó cũng chưa xử lý mỉa mai hoặc liên kết khía cạnh.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Giả định `🙂=+1`, `:(=−1`. Chuỗi “🙂 :(” có điểm trung bình 0 nhưng chứa hai tín hiệu trái chiều, không giống không có biểu tượng. Nếu $S_t=-0.6$, $S_e=1$ và $\lambda=0.75$, điểm kết hợp minh họa là −0.2; không suy đây là cách hiểu đúng một lời mỉa mai.

Mã dưới đây chỉ dò **bảng chuỗi nhỏ đã khai báo**, ưu tiên chuỗi dài nhất. Nó không phải bộ tách mọi emoji Unicode; văn bản/biểu tượng ngoài bảng sẽ được bỏ qua trong demo và chưa có bộ đếm toàn bộ biểu tượng chưa biết.

```python
from math import isclose

# Điểm giả định; None nghĩa là nhận chuỗi nhưng chưa cấp điểm.
table = {"👩‍👩‍👧‍👦": None, "👍🏽": 1.0, "👍": 1.0,
         "🙂": 1.0, ":)": 1.0, ":(": -1.0}
keys = sorted(table, key=len, reverse=True)

def scan_demo(text):
    result, i = [], 0
    while i < len(text):
        found = next((key for key in keys if text.startswith(key, i)), None)
        if found is None:
            i += 1
            continue
        result.append({"start": i, "end": i + len(found),
                       "symbol": found, "score": table[found]})
        i += len(found)
    return result

def mean_score(records):
    known = [r["score"] for r in records if r["score"] is not None]
    return sum(known) / len(known) if known else None

assert [r["symbol"] for r in scan_demo("Ổn 👍🏽")] == ["👍🏽"]
assert len(scan_demo("👩‍👩‍👧‍👦")) == 1
assert mean_score(scan_demo("👩‍👩‍👧‍👦")) is None
assert mean_score(scan_demo("🙂 :(")) == 0.0
assert mean_score(scan_demo("Chỉ có chữ")) is None
assert isclose(0.75 * (-0.6) + 0.25 * 1.0, -0.2)
print([r["symbol"] for r in scan_demo("Phòng ổn 👍🏽 :)")])
# ['👍🏽', ':)']
```

Trong hệ thật, cần kiểm tra tokenization chuỗi đầy đủ và những trường hợp ngoài bảng. Nhận chuỗi đúng trong demo không chứng minh diễn giải sentiment theo ngữ cảnh đúng.

## 24. Tình huống sai và phản ví dụ

“Lại mất nước nữa rồi 🙂” có thể bị chấm dương nếu chỉ nhìn biểu tượng. “Tuyệt vời :(” có thể là chữ và biểu tượng mâu thuẫn, lỗi gõ hoặc bối cảnh thiếu. Không nên tự động đảo sentiment chỉ vì có mâu thuẫn.

Một lỗi kỹ thuật khác là tách `👍🏽` thành hai đơn vị rồi cộng hai điểm. Cần kiểm tra nhận diện trước khi sửa mô hình ngữ nghĩa; nếu đầu vào đã sai đơn vị, thay trọng số chỉ che một phần vấn đề.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Tầng nhận diện: kiểm tra precision/recall theo span và loại chuỗi, gồm modifier/ZWJ và emoticon trong mã/URL. Tầng sentiment: macro-F1 hoặc metric đúng bài toán, báo riêng dữ liệu có/không biểu tượng và nhóm mâu thuẫn chữ.

So xóa, giữ, đổi tên và tra điểm trên cùng test. Kiểm tra độ nhạy theo lexicon, phiên bản và số lần lặp. Không dùng nhãn được tạo từ chính emoji để chứng minh mô hình hiểu cảm xúc do con người gán độc lập.

## 26. Xử lý khi không đáp ứng yêu cầu

Chuỗi bị tách: sửa bộ nhận diện/phiên bản. Coverage điểm thấp: bổ sung tài nguyên hoặc giữ trạng thái thiếu. Mỉa mai sai: đánh giá mô hình ngữ cảnh và mẫu gán nhãn, không chỉ đổi điểm biểu tượng toàn cục.

Nếu đổi biểu tượng thành chữ làm hiệu năng giảm, kiểm tra đếm trùng và mất ngữ cảnh. Nếu kết quả không ổn định theo nền tảng, giới hạn phạm vi áp dụng và báo sự khác biệt thay vì gộp một điểm chung.

## 27. Phân biệt kết quả với kết luận

“Hệ nhận diện ba emoji” là kết quả kỹ thuật. “Lexicon cho điểm +1” là kết quả tài nguyên. “Khách đang vui” là suy luận về con người cần bằng chứng khác.

Trong nghiên cứu du lịch, biểu tượng có thể liên quan phong cách giao tiếp, không chỉ trải nghiệm. So mức sử dụng emoji giữa quốc tịch không đủ kết luận nhóm nào hài lòng hơn hoặc thân thiện hơn.

## 28. Cách viết vào bài nghiên cứu

Nêu bộ nhận diện, phiên bản Unicode/tài nguyên, chính sách giữ/gộp/thay, nguồn điểm và cách kết hợp với chữ. Báo tỷ lệ biểu tượng ngoài phạm vi và phản hồi chỉ có biểu tượng.

Khung: “Chúng tôi giữ raw text, nhận diện [loại] bằng [công cụ], xử lý biến thể bằng [quy tắc]. Sentiment được suy ra bằng [phương án], tham số chọn trên [tập], đánh giá trên [nhãn độc lập].” Không gọi tra lexicon là mô hình đã hiểu mỉa mai.

## 29. Khả năng tái lập và tính minh bạch

Lưu phiên bản mapping, raw sequence, offset và bộ nhận diện. Dữ liệu UTF-8 phải đi qua đọc/ghi không làm mất ký tự. Nếu chỉ chia sẻ bản đã thay tên, người khác không kiểm tra lại mọi quyết định được nên cần mô tả giới hạn.

GitHub có thể chứa các ví dụ tự soạn và bảng giả định. Tài nguyên điểm thật cần nguồn và quyền phân phối tương ứng; không mặc định giấy phép mã bao phủ mọi bảng dữ liệu đi kèm.

## 30. Đạo đức nghiên cứu liên quan

Không gán đặc điểm tâm lý cho cá nhân từ một biểu tượng. Không suy sentiment theo màu da, giới hoặc nhóm xã hội của hình. Khi phân tích cách dùng của cộng đồng, cần tách hiệu ứng dữ liệu, nền tảng và ngôn ngữ.

Phản hồi công khai vẫn có thể có thông tin nhận diện. Ví dụ nghiên cứu nên ẩn tên/mã đặt phòng và tránh trích chuỗi đặc thù khiến người viết dễ bị truy ra khi không cần thiết.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **Một hình hiển thị có luôn là một code point không?** Không.
2. **Điểm biểu tượng bằng 0 có giống không có dữ liệu không?** Không.
3. **Bài tập tính:** $S_t=0.4$, $S_e=-1$, $\lambda=0.5$ cho −0.3. Giải thích tại sao con số chưa quyết định cách hiểu câu nếu chưa kiểm định kết hợp.
4. **Bài tập kỹ thuật:** kiểm tra `👍`, `👍🏽`, chuỗi gia đình và `:)`; xác định span và tránh đếm lặp.
5. **Bài tập ngữ nghĩa:** viết ba câu cùng emoji nhưng dùng để khen, mỉa mai và chưa đủ ngữ cảnh. So nhãn biểu tượng tĩnh với nhãn toàn câu.

## 32. Những điểm còn tranh luận

Các vấn đề là mức phụ thuộc ngữ cảnh/văn hóa của nghĩa biểu tượng, khả năng chuyển lexicon giữa nền tảng và độ đúng khi kết hợp với chữ. Cần phân biệt chuẩn kỹ thuật hiển thị với bằng chứng cảm xúc.

Một phương án có thể nhận diện chuỗi rất tốt nhưng suy sentiment kém; một phương án khác có hiệu năng trung bình cao mà bỏ qua nhóm biểu tượng hiếm. Nghiên cứu nên báo hai tầng và những nhóm giới hạn, không dùng một chỉ số chung để che sự khác biệt.
