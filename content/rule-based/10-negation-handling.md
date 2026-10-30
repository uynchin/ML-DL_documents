# Negation handling — Xử lý phủ định trong phân tích cảm xúc

> **Nhánh:** LEXICON BASED → Xử lý quy tắc. **Nguồn đề mục:** dòng 11 trong bảng Excel. **Ngày đối chiếu:** 28/10/2026. Ví dụ khách sạn, điểm số và mã minh họa được tự xây dựng; không phải kết quả thực nghiệm của các công trình tham khảo.

| Trường | Nội dung |
|---|---|
| Bản chất | Thành phần xử lý ngữ cảnh; không phải từ điển hoặc bộ phân loại hoàn chỉnh |
| Đầu vào | Văn bản/token, điểm từ vựng, tín hiệu phủ định và cấu trúc câu nếu có |
| Đầu ra | Tín hiệu phủ định, phạm vi tác động, điểm/đặc trưng được điều chỉnh |
| Mốc tham khảo | Councill và cộng sự, 2010; Wiegand và cộng sự, 2010 [S1–S2]; không coi 2010 là năm phủ định được phát hiện |
| Tiếng Việt | Cần quy tắc/tài nguyên theo tiếng Việt, xét các cấu trúc như “không”, “chưa”, “chẳng”, “không phải…mà…” |
| Nhãn | Luật có thể chạy không cần huấn luyện; học nhận diện phạm vi cần nhãn hoặc nguồn giám sát phù hợp; đánh giá cần nhãn độc lập |
| Tính toán | Luật tuyến tính nhỏ thường dùng CPU; parser và mô hình theo ngữ cảnh có chi phí riêng |

## 1. Định nghĩa học thuật

Xử lý phủ định là nhận diện và mô hình hóa ảnh hưởng của phủ định đến ý nghĩa cần phân tích. Trong sentiment analysis, cần phân biệt **cue** — biểu thức báo hiệu phủ định — và **scope** — phần nội dung chịu tác động. Công trình về nhận diện scope cho sentiment cho thấy đây là một nhiệm vụ cần được xử lý riêng, không chỉ tìm từ “not” [S1].

Một hệ thống còn phải quyết định phủ định làm điểm thay đổi thế nào. Nhận diện phạm vi đúng và điều chỉnh sắc thái đúng là hai vấn đề khác nhau.

## 2. Giải thích đơn giản

Trong “phòng sạch”, “sạch” là lời khen. Thêm “không” tạo “phòng không sạch”, làm ý nghĩa thay đổi. Nhưng trong “phòng không sạch nhưng nhân viên tốt”, từ “không” không nên làm lời khen nhân viên thành lời chê.

Vì vậy, cần biết **có phủ định ở đâu, nó tác động vào đâu và tác động thế nào**. Chỉ đếm từ tốt/xấu hoặc đảo dấu mọi thứ sau “không” đều có thể sai.

## 3. Mục đích và vấn đề giải quyết

Đề xuất nghiên cứu khách sạn: kiểm tra xem thêm xử lý phủ định có giảm lỗi ở các câu như “không đáng tiền”, “chưa hài lòng”, “không quá tệ” hay không. Lợi ích nên được đo trên cùng bộ dữ liệu với hệ thống chưa thêm thành phần này.

Ngoài tăng độ đúng nhãn, xử lý phủ định có thể giúp nhận diện đúng khía cạnh bị phàn nàn. Một phản hồi có hai khía cạnh cần giữ sự khác biệt giữa phòng và nhân viên.

## 4. Điều kiện và giả định sử dụng

Cần giữ tín hiệu phủ định trong tiền xử lý và thống nhất đơn vị token. Văn bản mạng xã hội có thể viết “ko”, “k”, “khum”; việc chuẩn hóa những dạng đó cần kiểm tra ngữ cảnh, không phải thay mọi ký tự trùng.

Giả định của luật đơn giản là phạm vi phủ định có thể xấp xỉ bằng cửa sổ hoặc ranh giới mệnh đề. Giả định này dễ sai khi có câu dài, chủ ngữ khác, cấu trúc lồng hoặc thiếu dấu câu. Phải đánh giá mức phù hợp với dữ liệu thực tế.

## 5. Cơ chế hoạt động bên trong

Một quy trình minh họa: nhận diện cue → loại ngoại lệ → xác định span chịu tác động → gắn span với biểu thức ý kiến → điều chỉnh điểm → tổng hợp theo đối tượng. Nếu bỏ bước liên kết đối tượng, hệ thống vẫn có thể đảo sai một ý kiến ở mệnh đề khác.

Ví dụ “không chỉ sạch mà còn yên tĩnh” chứa chữ “không” nhưng không phải phủ định thông thường của “sạch”. Nên nhận diện cấu trúc nhiều từ trước khi áp dụng luật một từ để tránh kích hoạt sai.

## 6. Quy trình áp dụng trong nghiên cứu

1. Quy định nhiệm vụ và đơn vị: cue, scope, cực tính biểu thức hay nhãn câu.
2. Lập hướng dẫn gán nhãn, gồm ngoại lệ và câu không chắc.
3. Chia tập phát triển/kiểm tra; giữ mã đánh giá và khách sạn.
4. Xây baseline không xử lý phủ định.
5. Thêm một phương án rõ ràng: cửa sổ, mệnh đề, dependency hoặc mô hình học.
6. Chọn tham số trên tập phát triển, đóng băng trước test.
7. Đánh giá thành phần và chất lượng nhãn cuối, phân tích trường hợp sửa đúng/làm sai mới.

Không chỉ kiểm tra các câu có “không”; cần cả câu không phủ định để đo tác hại khi kích hoạt nhầm.

## 7. Ví dụ minh họa xuyên suốt

Xét “Phòng không sạch, nhưng nhân viên tốt.” Giả định điểm từ vựng “sạch” là +2 và “tốt” là +2. Luật mong muốn nhận “không” tác động lên “sạch”, dừng trước phần nói về nhân viên.

Sau điều chỉnh minh họa, phòng mang điểm −2 và nhân viên giữ +2. Tổng bằng 0 không có nghĩa cả hai khía cạnh trung tính; nó là kết quả của hai ý kiến trái chiều. Phân tích theo khía cạnh giữ được thông tin quản trị hơn điểm toàn câu.

## 8. Kết quả và cách diễn giải

Đầu ra trung gian có thể là cue, chỉ số bắt đầu/kết thúc scope và các biểu thức ý kiến bị điều chỉnh. Đầu ra cuối là điểm hoặc nhãn theo bộ chấm đã chọn.

Một cờ `negated=True` không đồng nghĩa “câu tiêu cực”. “Không tệ” và “không tốt” cùng có phủ định nhưng có ý nghĩa đánh giá khác nhau. Cần đọc cờ cùng điểm gốc, loại cấu trúc và quy tắc điều chỉnh.

## 9. Điểm mạnh, điểm yếu và giới hạn

Luật minh bạch có thể sửa một nhóm lỗi có nguyên nhân rõ và dễ truy lại biểu thức bị tác động. Tuy nhiên, luật hẹp bỏ sót cấu trúc phức tạp; luật rộng dễ lan sai qua mệnh đề. Parser có thể cung cấp cấu trúc nhưng lỗi parser cũng truyền xuống.

Điểm hạn chế quan trọng là phủ định không luôn tạo một cực đối xứng. “Không xuất sắc” không nhất thiết mạnh tương đương “rất tệ”; việc đảo dấu là phép xấp xỉ cần thẩm định.

## 10. So sánh với phương pháp gần nhau

| Thành phần | Khác biệt |
|---|---|
| Phủ định | Thay đổi hiệu lực/ý nghĩa của một nội dung; có thể ảnh hưởng cực tính |
| Tăng/giảm cường độ | Điều chỉnh mức độ; không mặc định đổi chiều |
| Liên từ tương phản | Tổ chức hai vế, không tự là phủ định |
| Subjectivity | Mức biểu hiện chủ quan, không phải cờ phủ định |
| Mô hình ngữ cảnh | Có thể học tương tác nhưng vẫn cần kiểm tra nhóm câu phủ định |

Một từ như “nhưng” có thể là ranh giới hữu ích cho luật scope, nhưng không có nghĩa nó là cue phủ định.

## 11. Sai lầm thường gặp và cách khắc phục

- Xóa “không” như stopword: bảo toàn hoặc biểu diễn nó trong pipeline.
- Đảo mọi điểm sau cue đến hết văn bản: thêm ranh giới và kiểm tra câu nhiều đối tượng.
- Coi hai phủ định luôn triệt tiêu như đại số: kiểm tra ngữ dụng và cấu trúc.
- Không tách “không chỉ…mà còn…”: thêm nhận diện ngoại lệ trước luật chung.
- Báo mọi câu phủ định là tiêu cực: đánh giá cực tính sau khi xử lý nội dung chịu tác động.

## 12. Tài liệu tham khảo

- **[S1]** Councill, I., McDonald, R., & Velikovich, L. (2010). *What’s great and what’s not: learning to classify the scope of negation for improved sentiment analysis*. [ACL Anthology](https://aclanthology.org/W10-3110/).
- **[S2]** Wiegand, M., Balahur, A., Roth, B., Klakow, D., & Montoyo, A. (2010). *A survey on the role of negation in sentiment analysis*. [ACL Anthology](https://aclanthology.org/W10-3111/).
- **[S3]** Taboada, M., Brooke, J., Tofiloski, M., Voll, K., & Stede, M. (2011). *Lexicon-Based Methods for Sentiment Analysis*. [Bài báo](https://aclanthology.org/J11-2001/).

S1–S2 phục vụ phân biệt cue/scope và các vấn đề phủ định; S3 đặt thành phần trong bối cảnh hệ từ điển. Luật, hệ số và ví dụ tiếng Việt bên dưới là thiết kế giảng giải của người biên soạn, không gán cho các tác giả trên.

## 13. Vị trí trong toàn bộ nghiên cứu

Thành phần có thể nằm sau chuẩn hóa chữ viết và trước tổng hợp sentiment. Nếu dùng mô hình có giám sát, scope có thể là đặc trưng đầu vào hoặc tác vụ phụ, thay vì bắt buộc sửa văn bản thành câu khác.

Cần ghi thứ tự pipeline. Nếu chuẩn hóa làm mất cue trước khi thành phần phủ định chạy, sửa công thức điều chỉnh cũng không cứu được thông tin đã bị xóa.

## 14. Kiến thức nền cần biết trước

Cần hiểu token, span, mệnh đề, cực tính từ vựng và cực tính trong ngữ cảnh. **Cue** không phải toàn bộ **scope**; “không” là cue, còn phần bị phủ định có thể gồm nhiều từ.

Phân biệt phủ định ngôn ngữ với dấu âm của điểm toán học. Một biểu thức phủ định có thể truyền đạt lời khen nhẹ, lời chê, sự từ chối hoặc mô tả sự kiện, tùy nội dung và hoàn cảnh.

## 15. Nền tảng lý thuyết và logic hình thành

Sentiment của tổ hợp không luôn bằng tổng sentiment của từng từ độc lập. Phủ định là một ví dụ cho việc các đơn vị kết hợp làm thay đổi cách diễn giải.

Trong thiết kế đo lường, có thể xem một hàm điều chỉnh là phép xấp xỉ hiện tượng này. Điều cần kiểm định là phép xấp xỉ giữ được phán đoán con người đến mức nào, đặc biệt khi từ gốc có cường độ khác nhau hoặc khi phủ định mang tính nói giảm.

## 16. Các thành phần cấu tạo

Hệ thống nên tách danh sách cue; mẫu ngoại lệ; bộ xác định scope; liên kết biểu thức cảm xúc; hàm điều chỉnh; quy tắc tổng hợp và bộ kiểm tra. Mỗi lần đổi danh sách cue nên được xem như một phiên bản cấu hình.

Nhật ký cần cho biết từ nào đổi điểm, cue nào gây đổi và ranh giới nào kết thúc scope. Chỉ lưu nhãn cuối sẽ khó phát hiện lỗi lan sang ý kiến không liên quan.

## 17. Các loại và biến thể

Có thể dùng cửa sổ cố định, luật đến dấu câu/liên từ, phân tích phụ thuộc hoặc mô hình gán nhãn chuỗi. Về điều chỉnh, có thể đảo dấu, giảm/đổi mức điểm hoặc học một phép biến đổi theo dữ liệu.

Đây là các lựa chọn khác nhau, không phải các chế độ tương đương. Việc phủ định biểu hiện qua cấu trúc như “thiếu sự sạch sẽ” cũng khác việc tìm cue đứng ngay trước một tính từ. Cần xác định phạm vi nhiệm vụ đang bao phủ.

## 18. Đầu vào và đầu ra

Đề xuất giữ `review_id`, `sentence_id`, token/offset gốc, `cue_span`, `scope_span`, `opinion_span`, `aspect_id`, `base_score`, `adjusted_score`, `rule_id`.

Nếu một ý kiến không được từ điển nhận diện, không nên tự coi điểm 0 là trung tính. Ghi thiếu bằng chứng từ vựng riêng với trạng thái scope. Hệ thống có thể nhận đúng phủ định nhưng chưa biết sắc thái của phần bị phủ định.

## 19. Căn cứ lựa chọn

Bắt đầu bằng phân tích lỗi của baseline: bao nhiêu lỗi liên quan phủ định, cue và cấu trúc nào phổ biến? Nếu lỗi chủ yếu là các mẫu ngắn ổn định, luật đơn giản đáng thử. Nếu câu dài và lồng nhiều tầng, mô hình scope có thể phù hợp hơn, nhưng chi phí gán nhãn và triển khai cao hơn.

Không chọn parser chỉ vì nó phức tạp; phải chứng minh chất lượng tăng đủ cho mục tiêu thực tế.

## 20. Trường hợp không nên sử dụng

Không áp luật đảo dấu lần nữa lên điểm đã được một bộ chấm như VADER xử lý phủ định nếu chưa kiểm tra cơ chế tương tác. Làm vậy có thể đếm tác động hai lần.

Không dùng luật minh họa như giải pháp hoàn chỉnh cho phủ định kép, câu hỏi tu từ, mỉa mai hoặc cấu trúc đa ngôn ngữ. Với dữ liệu chưa rõ, giữ cờ cần kiểm tra hoặc dùng phương án phù hợp thay vì ép kết luận chắc chắn.

## 21. Các quyết định quan trọng khi thực hiện

Phải quyết định scope đi theo số token hay cấu trúc; có dừng ở dấu phẩy không; xử lý nhiều cue ra sao; ngoại lệ nào có ưu tiên. Dấu phẩy có thể ngăn mệnh đề nhưng cũng có thể chỉ ngắt phần xen vào, nên không phải ranh giới ngôn ngữ tuyệt đối.

Khi kết hợp cường độ, cần xác định thứ tự. “Không rất tốt” và “rất không tốt” không nên mặc nhiên được xem là một cấu trúc giống nhau chỉ vì có cùng các token.

## 22. Giải thích công thức và ký hiệu

Một baseline **do bài này lựa chọn** đặt $z_i=1$ nếu biểu thức ý kiến $i$ được xác định nằm trong scope phù hợp, ngược lại $z_i=0$. Với điểm gốc $v_i$:

$$v_i'=(1-2z_i)v_i.$$

Nếu $z_i=1$, điểm đảo dấu; nếu $z_i=0$, điểm giữ nguyên. Đây là quy tắc đại số đơn giản, không phải quy luật ngôn ngữ phổ quát. Ví dụ “không tệ” không bảo đảm có mức khen đối xứng với mức chê của “tệ”.

Có thể thử hàm khác như $v_i'=-\lambda v_i$ với $\lambda>0$ cho trường hợp phủ định, nhưng hệ số cần căn cứ phát triển/kiểm định. Đừng gộp xác suất nhận diện scope với điểm sentiment thành cùng một đại lượng.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Với ví dụ mục 7: $(v_1,v_2)=(2,2)$ và $(z_1,z_2)=(1,0)$, kết quả là $(-2,2)$. Lưu hai biểu thức riêng cho biết phòng bị chê và nhân viên được khen.

Mã Python dưới đây là **bộ luật giảng giải rất hẹp**: scope kéo dài sau “không” đến dấu câu hoặc “nhưng”; chưa xử lý ngoại lệ, phủ định kép, cú pháp và mỉa mai. Không dùng nó như bộ phân tích tiếng Việt hoàn chỉnh.

```python
import re

lexicon = {"sạch": 2.0, "tốt": 2.0, "tệ": -2.0}
boundaries = {",", ".", ";", "!", "?", "nhưng"}

def score_demo(text):
    tokens = re.findall(r"\w+|[^\w\s]", text.casefold())
    active = False
    records = []
    for index, token in enumerate(tokens):
        if token in boundaries:
            active = False
        elif token == "không":
            active = True
        elif token in lexicon:
            base = lexicon[token]
            records.append({"index": index, "term": token,
                            "negated": active, "base": base,
                            "adjusted": -base if active else base})
    return records

result = score_demo("Phòng không sạch, nhưng nhân viên tốt.")
assert [r["adjusted"] for r in result] == [-2.0, 2.0]
assert score_demo("Phòng sạch.")[0]["adjusted"] == 2.0
# Phản ví dụ cho thấy giới hạn của luật:
assert score_demo("Phòng không chỉ sạch mà còn tốt.")[0]["adjusted"] == -2.0
print([(r["term"], r["adjusted"]) for r in result])
# [('sạch', -2.0), ('tốt', 2.0)]
```

Kiểm tra cuối cố ý xác nhận một đầu ra sai nghĩa để chỉ ra giới hạn. Muốn sửa, phải nhận mẫu ngoại lệ trước khi kích hoạt cue; sau sửa cần kiểm tra cả các câu khác.

## 24. Tình huống sai và phản ví dụ

“Không chỉ phòng sạch mà nhân viên còn rất tốt” dễ bị đảo sai nếu thấy “không” là kích hoạt ngay. “Tôi không nói phòng tốt” lại phủ định hành động phát biểu, không nhất thiết khẳng định phòng tệ.

Với “không phải là không tốt”, việc nhân hai dấu âm có thể cho một đầu ra dương nhưng chưa thể hiện sắc thái dè dặt. Những ví dụ này nên thành nhóm kiểm tra riêng, có nhãn và hướng dẫn rõ ràng.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đề xuất đo cue precision/recall/F1, scope theo token hoặc khớp toàn span nếu có nhãn. Ở tầng cuối, báo macro-F1 và ma trận nhầm lẫn. Nêu rõ metric scope có tính cue hay không và cách ghép nhiều cue.

So baseline không luật với bản có luật trên cùng dữ liệu. Báo riêng các câu có cue, ngoại lệ và không cue; cải thiện trên một nhóm nhỏ có thể đi kèm giảm chất lượng toàn bộ. Dùng các cặp câu tối thiểu để chẩn đoán, không thay thế test tự nhiên.

## 26. Xử lý khi không đáp ứng yêu cầu

Cue bỏ sót: xem chuẩn hóa và danh sách biểu thức. Scope tràn: xem ranh giới, đối tượng và parser. Scope đúng nhưng polarity sai: thay hàm điều chỉnh hoặc kiểm tra điểm gốc. Ngoại lệ sai: bổ sung mẫu trên tập phát triển và kiểm tra tác dụng phụ.

Không sửa luật theo nhãn test rồi tiếp tục gọi kết quả trên test đó là đánh giá độc lập. Nếu dữ liệu quá ít, giới hạn kết luận ở pilot và tiếp tục thu thập mẫu kiểm định.

## 27. Phân biệt kết quả với kết luận

“Luật đổi điểm từ +2 thành −2” chỉ mô tả phép tính. “Câu được hiểu đúng” cần nhãn ngữ nghĩa phù hợp. “Khách không hài lòng về toàn bộ khách sạn” còn đòi hỏi xác định đối tượng, cấp phân tích và tính đại diện mẫu.

Một ý kiến phủ định về phòng không đủ để quy kết nhân viên kém hay chứng minh nguyên nhân khách không quay lại. Nên giới hạn suy luận theo thông tin văn bản và thiết kế nghiên cứu.

## 28. Cách viết vào bài nghiên cứu

Mẫu cần điền bằng dữ liệu thật: “Chúng tôi nhận diện cue bằng [nguồn], scope bằng [phương pháp], điều chỉnh điểm theo [công thức]. Các ngoại lệ gồm [mẫu]. Tham số được chọn trên [tập phát triển], đánh giá bằng [metric] trên [tập độc lập].”

Phần kết quả cần báo số câu bị tác động, lỗi được sửa và lỗi mới phát sinh. Nếu chỉ triển khai luật đơn giản, gọi đúng là heuristic phủ định, không nhận đã tái lập mô hình học scope trong S1.

## 29. Khả năng tái lập và tính minh bạch

Lưu phiên bản danh sách cue, ngoại lệ, tokenization, luật scope, hàm đổi điểm và thứ tự kết hợp với thành phần khác. Tạo nhật ký trước/sau cho mẫu có thể kiểm tra.

Kho GitHub nên có ví dụ tự soạn và bộ kiểm tra ngoại lệ. Nếu có nhãn scope thật, chỉ chia sẻ trong giới hạn quyền dữ liệu; lưu hướng dẫn gán nhãn và cách xử lý bất đồng để người khác hiểu chuẩn đánh giá.

## 30. Đạo đức nghiên cứu liên quan

Cách phủ định khác nhau giữa nhóm tuổi, vùng miền và phong cách viết. Không xem lời chê nhẹ hoặc lối nói vòng là thiếu trung thực. Kiểm tra tỷ lệ lỗi theo nhóm biểu đạt trước khi dùng kết quả để so khách hàng.

Ví dụ công khai cần ẩn tên người và mã đặt phòng. Nếu kết quả liên quan đánh giá nhân viên, cần tránh quyết định cá nhân chỉ từ một luật cảm xúc chưa được kiểm định đúng bối cảnh.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **Cue đúng có bảo đảm scope đúng không?** Không; đây là hai tầng khác nhau.
2. **Phủ định có luôn đảo thành mức đối xứng không?** Không; đảo dấu chỉ là một baseline.
3. **Bài tập tính:** cho $v=-3$ và $z=1$, baseline cho $v'=3$. Nêu tình huống mà mức khen +3 này có thể quá mạnh.
4. **Bài tập thiết kế:** viết 12 cặp đánh giá chỉ khác cue, ranh giới hoặc khía cạnh. Đánh dấu cue/scope, rồi so luật cửa sổ với luật mệnh đề.
5. **Bài tập phản biện:** giải thích lỗi của mã minh họa trên “không chỉ sạch mà còn tốt”, đề xuất thứ tự ưu tiên ngoại lệ và một câu mới có thể làm luật sửa bị sai.

## 32. Những điểm còn tranh luận

Các vấn đề chính là xác định scope theo cú pháp hay ngữ nghĩa, mô hình hóa sức mạnh phủ định và chuyển luật giữa ngôn ngữ/miền. Nhãn scope cũng có thể phụ thuộc quy ước gán nhãn nên cần công bố hướng dẫn.

Trong nghiên cứu cụ thể, mục tiêu không phải có nhiều luật nhất mà là phép xử lý phù hợp và có bằng chứng. Có thể một luật làm metric scope tăng nhưng nhãn sentiment cuối chưa cải thiện; phải đánh giá hai tầng thay vì coi chúng tương đương.
