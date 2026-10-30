# Intensifier handling — Xử lý từ tăng và giảm cường độ

> **Nhánh:** LEXICON BASED → Xử lý quy tắc. **Nguồn đề mục:** dòng 12 trong bảng Excel. **Ngày đối chiếu:** 28/10/2026. Ví dụ, hệ số và mã được tự xây dựng để giảng giải; các hệ số không phải chuẩn tâm lý hoặc tham số chính thức của VADER/SO-CAL.

| Trường | Nội dung |
|---|---|
| Bản chất | Thành phần điều chỉnh mức độ của biểu thức đánh giá |
| Đầu vào | Từ/cụm cảm xúc, điểm gốc, modifier và quan hệ modifier–biểu thức |
| Đầu ra | Điểm điều chỉnh, kiểu modifier, biểu thức bị tác động và dấu vết quy tắc |
| Mốc tham khảo | Taboada và cộng sự, 2011; Hutto & Gilbert, 2014 [S1–S2]; đây là mốc công trình minh họa, không phải năm xuất hiện hiện tượng |
| Tiếng Việt | Cần xét cả từ đứng trước và sau, như “rất tốt”, “tốt lắm”, cùng các cấu trúc đặc thù |
| Nhãn | Luật không bắt buộc huấn luyện; học hệ số cần dữ liệu phát triển; kiểm định cần chuẩn độc lập |
| Tính toán | Luật cục bộ thường dùng CPU; mô hình nhận quan hệ cú pháp có chi phí khác |

## 1. Định nghĩa học thuật

Xử lý cường độ là mô hình hóa ảnh hưởng của biểu thức tăng/giảm mức độ đến sentiment của đơn vị mà nó bổ nghĩa. Có thể phân biệt **amplifier** — tăng mức — và **downtoner** — giảm mức. Hệ từ điển như SO-CAL và hệ quy tắc VADER đưa yếu tố mức độ vào quá trình chấm thay vì chỉ cộng điểm từ độc lập [S1–S2].

Trong bài này, tên đề mục “Intensifier handling” được dùng theo nghĩa thực hành rộng, gồm cả tăng và giảm cường độ.

## 2. Giải thích đơn giản

“Tốt”, “rất tốt” và “hơi tốt” thường không thể hiện cùng mức đánh giá. Tương tự, “rất tệ” thường mạnh hơn “tệ”: từ tăng mức làm lời chê mạnh hơn, không biến nó thành lời khen.

Bạn cần xác định từ nào bổ nghĩa cho nhận xét nào rồi mới điều chỉnh. Trong “phòng rất sạch nhưng dịch vụ chậm”, “rất” không nên tự tăng mức của “chậm” ở phần sau.

## 3. Mục đích và vấn đề giải quyết

Đề xuất nghiên cứu: kiểm tra hệ thống có giữ được khác biệt mức độ giữa các phản hồi khách sạn hay không. Nếu chỉ phân loại tích cực/tiêu cực, hai câu “tốt” và “rất tốt” có thể cùng nhãn; lợi ích cường độ cần đo bằng mục tiêu phù hợp chứ không chỉ đòi nhãn đổi.

Có thể dùng thành phần này để dự đoán điểm liên tục, xếp mức độ phàn nàn hoặc tạo đặc trưng. Những ứng dụng đó cần nhãn mức độ/thang đo và kiểm định riêng.

## 4. Điều kiện và giả định sử dụng

Cần biết điểm gốc thuộc thang nào và phép điều chỉnh có ý nghĩa trên thang đó không. Không nhân tùy ý điểm xác suất như điểm từ vựng. Cần giữ modifier, vị trí, phủ định và ranh giới câu.

Giả định của baseline cục bộ là modifier gần biểu thức thì có quan hệ bổ nghĩa. Điều này cần kiểm tra: “rất nhiều người thấy phòng sạch” không có nghĩa “rất” trực tiếp tăng mức “sạch”.

## 5. Cơ chế hoạt động bên trong

Quy trình đề xuất: nhận diện biểu thức cảm xúc → nhận modifier/cụm modifier → xác định đối tượng được bổ nghĩa → phân loại tăng/giảm hoặc cấu trúc đặc biệt → điều chỉnh → kết hợp phủ định → tổng hợp.

Việc ưu tiên cụm quan trọng: “không quá tốt” khác “quá tốt”, và “quá đắt” mang đánh giá về vượt mức chấp nhận. Một từ bề mặt không luôn có chức năng cố định trong mọi cấu trúc.

## 6. Quy trình áp dụng trong nghiên cứu

1. Chọn mục tiêu: phân lớp, mức cường độ hay xếp hạng.
2. Gán nhãn hoặc thẩm định các biểu thức có/không modifier.
3. Xây baseline giữ nguyên điểm gốc.
4. Lập từ điển modifier và quy tắc liên kết theo ngôn ngữ.
5. Ước lượng/chọn hệ số trên tập phát triển, ghi thang điểm rõ ràng.
6. Cố định cách kết hợp nhiều modifier, phủ định và giới hạn thang nếu có.
7. Đánh giá độc lập cùng các nhóm câu phản ví dụ.

Không chọn hệ số để làm các ví dụ đẹp rồi nhận đó là hệ số đã được kiểm định trên người dùng thật.

## 7. Ví dụ minh họa xuyên suốt

Giả định “sạch” có điểm +2, “tệ” có điểm −2. Đặt hệ số tự chọn `rất=1.5`, `hơi=0.5`. Khi đó “rất sạch” thành +3, “hơi tệ” thành −1 và “rất tệ” thành −3.

Ba kết quả thể hiện điều chỉnh độ lớn trong baseline. Với “phòng hơi nhỏ nhưng rất sạch”, còn cần xác định nhỏ có mang ý chê trong bối cảnh không và modifier đang gắn với đúng tính từ nào.

## 8. Kết quả và cách diễn giải

Đầu ra nên cho cả điểm gốc và điểm mới. “Tệ” từ −2 thành −3 nghĩa là quy tắc tăng mức lời chê, không phải xác suất tiêu cực tăng từ 2% lên 3%.

Nếu dùng ngưỡng phân lớp, điểm thay đổi chưa chắc nhãn thay đổi. Đây không tự là lỗi: một thành phần mức độ có thể giữ cùng nhãn nhưng cải thiện tương quan với mức độ đánh giá. Cần đánh giá đúng loại mục tiêu đã định nghĩa.

## 9. Điểm mạnh, điểm yếu và giới hạn

Luật mức độ dễ kiểm tra đóng góp, có thể giữ thông tin mà nhãn cực tính đơn giản bỏ qua và cho baseline rõ ràng. Tuy nhiên, hệ số chủ quan, phạm vi bổ nghĩa và tương tác cú pháp đều có thể tạo sai số.

Các biểu thức ngôn ngữ không tạo thang tỷ lệ tâm lý chỉ vì ta dùng phép nhân. “Rất tốt” không có nghĩa khách hài lòng gấp 1.5 lần “tốt”; đó chỉ là lựa chọn tính điểm trong ví dụ.

## 10. So sánh với phương pháp gần nhau

| Thành phần | Vai trò |
|---|---|
| Tăng cường độ | Tăng độ lớn của đánh giá theo quy tắc |
| Giảm cường độ | Giảm độ lớn, không mặc định xóa ý kiến |
| Phủ định | Có thể thay đổi chiều và ý nghĩa, không chỉ mức |
| Chữ hoa/dấu câu/lặp chữ | Có thể là tín hiệu nhấn mạnh nhưng cần quy tắc riêng |
| Điểm sao | Đánh giá do người dùng cung cấp, không đồng nhất điểm từ vựng |

Phải phân biệt từ có điểm sentiment mạnh sẵn với từ chỉ tăng mức: cộng điểm dương của “rất” vào “tệ” có thể làm lời chê yếu đi, trái mục tiêu.

## 11. Sai lầm thường gặp và cách khắc phục

- Cộng một số dương vào cả từ khen và chê: thử phép điều chỉnh bảo toàn chiều phù hợp thay vì mặc định cộng.
- Tăng mọi tính từ sau “rất”: xác định phạm vi và đối tượng.
- Nhân tiếp nhiều hệ số không giới hạn: quy định cách kết hợp và kiểm tra bão hòa.
- Áp luật cường độ sau bộ chấm đã xử lý cùng tín hiệu: kiểm tra tránh tác động hai lần.
- Xóa từ lặp, dấu chấm than trước khi nghiên cứu cường độ: giữ bản gốc và đánh giá từng biến thể xử lý.

## 12. Tài liệu tham khảo

- **[S1]** Taboada, M., Brooke, J., Tofiloski, M., Voll, K., & Stede, M. (2011). *Lexicon-Based Methods for Sentiment Analysis*. [ACL Anthology](https://aclanthology.org/J11-2001/).
- **[S2]** Hutto, C. J., & Gilbert, E. (2014). *VADER: A Parsimonious Rule-Based Model for Sentiment Analysis of Social Media Text*. [ICWSM](https://ojs.aaai.org/index.php/ICWSM/article/view/14550).
- **[S3]** Tác giả VADER. [Mã triển khai](https://github.com/cjhutto/vaderSentiment/blob/master/vaderSentiment/vaderSentiment.py).

Nguồn dùng để tham khảo vai trò xử lý mức độ trong các hệ cụ thể. Công thức và hệ số minh họa của bài là thiết kế riêng; không phải bảng hệ số chính thức hay hướng dẫn tái lập toàn bộ SO-CAL/VADER.

## 13. Vị trí trong toàn bộ nghiên cứu

Thành phần thường nằm trong bước chấm biểu thức trước khi tổng hợp câu/khía cạnh. Chuẩn hóa tiếng lóng có thể cần chạy trước để nhận “rấttt”, nhưng chuẩn hóa quá mạnh có thể làm mất tín hiệu nhấn mạnh.

Trong mô hình quản trị, nên dùng điểm sau kiểm định và nói rõ đơn vị. Không lấy điểm luật chưa xác nhận để so mức nghiêm trọng phàn nàn giữa khách sạn như một thang khách quan.

## 14. Kiến thức nền cần biết trước

Cần hiểu cực tính là chiều dương/âm, cường độ là mức mạnh/yếu và modifier là biểu thức bổ nghĩa. Phân biệt **thang thứ bậc** với **thang tỷ lệ**: xếp “rất tốt” cao hơn “tốt” không chứng minh khoảng cách đều hoặc có thể nói gấp bao nhiêu lần.

Cần hiểu điểm gốc không có bằng chứng khác điểm trung tính: modifier không tạo sentiment khi chưa xác định được nội dung nó đang bổ nghĩa.

## 15. Nền tảng lý thuyết và logic hình thành

Logic là sentiment trong tổ hợp chịu ảnh hưởng của cách biểu thức được điều chỉnh mức độ. Một hệ tính điểm cố gắng xấp xỉ tác động đó bằng hàm số.

Nhưng chức năng mức độ còn phụ thuộc ngữ cảnh và chuẩn kỳ vọng. “Phòng quá nhỏ” diễn đạt vượt mức chấp nhận, trong khi “phòng rất nhỏ” có thể là mô tả được chấp nhận ở một loại lưu trú. Điểm cần được kiểm tra theo mục tiêu đánh giá chứ không chỉ từ mặt chữ.

## 16. Các thành phần cấu tạo

Hệ cần có lexicon điểm gốc, danh sách modifier/cụm, hướng và phạm vi bổ nghĩa, hệ số hoặc hàm điều chỉnh, cách kết hợp và bộ tổng hợp. Đặc biệt cần biết điểm đầu vào đã được chuẩn hóa hoặc xử lý ngữ cảnh chưa.

Nên lưu một bản ghi cho mỗi modifier–opinion, gồm vị trí và quy tắc. Nhờ vậy có thể đọc được “rất” đã tác động vào “sạch” hay nhầm sang “chậm”.

## 17. Các loại và biến thể

Có thể dùng phép nhân hệ số, thay đổi mức trên thang rời rạc hoặc mô hình học từ nhãn. Có thể liên kết modifier bằng khoảng cách, mẫu từ loại hoặc dependency.

Các tín hiệu chữ hoa, dấu câu, kéo dài chữ và lặp từ có thể được xử lý như các nguồn nhấn mạnh bổ sung. Không nên gọi tất cả là từ tăng cường độ theo cùng nghĩa ngữ pháp; cần đặt tên và kiểm định từng nhóm rõ ràng.

## 18. Đầu vào và đầu ra

Đề xuất lưu `modifier_span`, `modifier_type`, `opinion_span`, `aspect_id`, `base_score`, `factor`, `adjusted_score`, `rule_id`, cùng mã câu gốc. Nếu nhiều modifier cùng tác động, giữ danh sách thứ tự thay vì chỉ hệ số tổng.

Với biểu thức không có điểm gốc, ghi `missing_base_score`. Nhận thấy “rất” không đủ để suy ra câu tích cực hoặc tiêu cực nếu chưa biết nội dung theo sau.

## 19. Căn cứ lựa chọn

Trước hết xem trong dữ liệu có nhiều biểu thức mức độ không và baseline sai ở đâu. Nếu mục tiêu là dự đoán mức độ, cần nhãn đủ phân biệt và độ đồng thuận giữa người gán.

Có thể bắt đầu bằng một tập modifier nhỏ được thẩm định, rồi mở rộng theo lỗi phát triển. Đánh giá lợi ích bằng kiểm định cặp hoặc xếp hạng khi thích hợp, không chỉ đòi accuracy ba lớp tăng.

## 20. Trường hợp không nên sử dụng

Không nhân hệ số vào xác suất đã hiệu chuẩn rồi coi kết quả vẫn là xác suất. Không bổ sung tác động lần hai khi mô hình đã sử dụng cùng modifier mà chưa có thiết kế kết hợp.

Không dùng luật “từ đứng trước” để xử lý toàn bộ tiếng Việt vì nhiều biểu thức đứng sau như “tốt lắm”. Với các tổ hợp có phủ định, cần kiểm tra cả cấu trúc thay vì nhân hệ số bất kể thứ tự.

## 21. Các quyết định quan trọng khi thực hiện

Phải chọn hệ số, hướng bổ nghĩa, phạm vi và cách xử lý nhiều modifier. Nếu đặt trần, cần công bố trần và tác dụng: nó có thể tránh điểm cực đoan nhưng làm nhiều biểu thức khác nhau có cùng điểm.

Thứ tự với phủ định và cách phân tích cú pháp phải được ghi. Trong thực tế, “không quá tốt” có thể hàm ý khen hạn chế/chê nhẹ, không nhất thiết là “rất tệ”. Một phép nhân và đổi dấu đơn giản không tự nắm được khác biệt đó.

## 22. Giải thích công thức và ký hiệu

Baseline của bài dùng điểm gốc $v$ và hệ số dương $m$:

$$v'=m\,v.$$

$m>1$ tăng độ lớn; $0<m<1$ giảm độ lớn; $m=1$ giữ nguyên. Phép nhân bảo toàn dấu với $m>0$. Hệ số này là lựa chọn của hệ tính điểm, không phải tỷ lệ cảm xúc tâm lý thật.

Nếu kết hợp nhiều modifier bằng tích, một thiết kế có thể là $m_{total}=\prod_jm_j$. Đây là giả định bổ sung cần kiểm tra, không quy tắc ngôn ngữ phổ quát. Nếu thang giới hạn trong $[-B,B]$, có thể dùng phép chặn $\max(-B,\min(B,mv))$, nhưng phải nói rõ mất thông tin do chặn. Không cần chặn khi thang đã được định nghĩa không giới hạn.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Đặt “sạch”=+2, “tệ”=−2; “rất”=1.5, “hơi”=0.5. “Rất tệ” cho −3, “hơi tệ” cho −1. Các con số là tự đặt; không phải đầu ra đã đo trên đánh giá khách sạn.

Mã chỉ nhận modifier **ngay trước** một từ có điểm, không có xử lý phủ định hoặc cú pháp. Giới hạn được thể hiện bằng ví dụ “không rất sạch” ở cuối.

```python
import re
from math import isclose

lexicon = {"sạch": 2.0, "tệ": -2.0}
factors = {"rất": 1.5, "hơi": 0.5}

def intensity_demo(text):
    tokens = re.findall(r"\w+|[^\w\s]", text.casefold())
    records = []
    for i, token in enumerate(tokens):
        if token not in lexicon:
            continue
        previous = tokens[i - 1] if i > 0 else None
        factor = factors.get(previous, 1.0)
        records.append({"term": token, "factor": factor,
                        "base": lexicon[token],
                        "adjusted": lexicon[token] * factor})
    return records

assert isclose(intensity_demo("Phòng rất sạch.")[0]["adjusted"], 3.0)
assert isclose(intensity_demo("Dịch vụ rất tệ.")[0]["adjusted"], -3.0)
assert isclose(intensity_demo("Dịch vụ hơi tệ.")[0]["adjusted"], -1.0)
assert intensity_demo("Rất gần biển.") == []
# Luật này chưa hiểu phủ định; kết quả sau không phải nhãn nghĩa đúng:
assert intensity_demo("Phòng không rất sạch.")[0]["adjusted"] == 3.0
print([r["adjusted"] for r in intensity_demo("Phòng rất sạch, dịch vụ hơi tệ.")])
# [3.0, -1.0]
```

Nếu muốn dùng cho nghiên cứu, phải bổ sung và kiểm định quan hệ bổ nghĩa, ngoại lệ và tương tác. Không đánh giá chỉ bằng chính các câu đã dùng để viết luật.

## 24. Tình huống sai và phản ví dụ

“Rất nhiều khách nói phòng sạch” chứa “rất” nhưng đối tượng mức độ là số lượng khách, không nhất thiết mức sạch. “Nhân viên tốt lắm” có modifier sau tính từ nên luật chỉ nhìn trước sẽ bỏ sót.

“Hơi bị đẹp” có thể mang tác dụng nhấn mạnh trong khẩu ngữ, trái việc luôn xem “hơi” là giảm mức. Cần xử lý cụm và cộng đồng/ngữ cảnh, thay vì áp một nghĩa cố định cho mọi lần xuất hiện.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đề xuất kiểm tra liên kết modifier–opinion bằng nhãn quan hệ hoặc span phù hợp. Nếu dự đoán cường độ liên tục, có thể dùng MAE và tương quan; nếu xếp thứ bậc, dùng metric xếp hạng phù hợp và giải thích loại nhãn.

Đánh giá các cặp tối thiểu “tệ/rất tệ/hơi tệ”, cả dương và âm. So cấu hình có/không modifier trên dữ liệu tự nhiên; kiểm tra nhạy với hệ số và trần. Không coi điểm tăng đúng hướng trên ví dụ tự soạn là bằng chứng hiệu năng tổng quát.

## 26. Xử lý khi không đáp ứng yêu cầu

Hướng đúng nhưng mức quá mạnh: xem hệ số và thang. Sai đối tượng: sửa liên kết thay vì chỉ giảm hệ số. Sai cụm khẩu ngữ: nhận biểu thức nhiều từ và giữ ngữ cảnh. Không cải thiện nhãn ba lớp: kiểm tra mục tiêu có nhạy với mức độ không trước khi kết luận thành phần vô ích.

Nếu hệ số không ổn định giữa nhóm khách, có thể cần thang/biểu diễn thích hợp hơn; không tự động dùng hệ số của nhóm đa số cho mọi nhóm mà không báo giới hạn.

## 27. Phân biệt kết quả với kết luận

“Điểm luật tăng từ +2 lên +3” không chứng minh khách hài lòng tăng 50%. Nó chỉ mô tả phép biến đổi số đã chọn. Muốn nói thay đổi trải nghiệm thực, cần phép đo độc lập và thiết kế nghiên cứu phù hợp.

Tương tự, lời phàn nàn dùng nhiều từ nhấn mạnh không tự chứng minh sự cố nghiêm trọng hơn; nó còn có thể phản ánh phong cách viết. Không dùng cường độ ngôn ngữ đơn độc để xếp mức thiệt hại thực tế.

## 28. Cách viết vào bài nghiên cứu

Nêu danh sách/cách xây modifier, cách liên kết, hệ số, thang điểm và tương tác với phủ định. Khung: “Điểm từ vựng được điều chỉnh bằng [hàm], modifier gắn bằng [quy tắc]. Hệ số được xác lập trên [tập], đánh giá bằng [metric phù hợp].”

Báo riêng kiểm tra cường độ và phân loại cuối. Nếu dùng hệ số tự thiết kế, gọi rõ là hệ số của nghiên cứu; không mượn tên một công cụ nổi tiếng để làm chúng có vẻ đã được chuẩn hóa.

## 29. Khả năng tái lập và tính minh bạch

Lưu bảng hệ số, phạm vi, thứ tự kết hợp, trần, phiên bản từ điển và ví dụ trước/sau. Nếu học hệ số từ dữ liệu, lưu cách chia tập và hàm mục tiêu.

Kho GitHub có thể chứa cấu hình và các cặp câu tự soạn. Không cần cài một gói NLP lớn để tái lập phép tính minh họa; nhưng phương án thật phải ghi các phụ thuộc nếu dùng parser hoặc mô hình học.

## 30. Đạo đức nghiên cứu liên quan

Phong cách nhấn mạnh có thể khác giữa nhóm tuổi, ngôn ngữ và nền tảng. Không xem người viết nhiều từ “rất” là phản ứng thái quá hoặc thiếu tin cậy. Cần tránh biến khác biệt phong cách thành đánh giá phẩm chất khách hàng.

Nếu dùng điểm để ưu tiên xử lý phản hồi, nên xem thêm nội dung sự cố và đánh giá con người; khách viết nhẹ vẫn có thể gặp vấn đề nghiêm trọng.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **“Rất tệ” có nên tăng theo hướng dương không?** Không trong baseline tăng độ lớn; điểm âm thường có độ lớn tăng.
2. **Nhân 1.5 có nghĩa hài lòng gấp 1.5 lần không?** Không.
3. **Bài tập tính:** với $v=-4$ và $m=0.5$, điểm là −2. Nếu có trần $B=3$, “rất tệ” với $m=1.5$ và $v=-4$ bị chặn thành −3; giải thích thông tin bị mất.
4. **Bài tập thiết kế:** tạo câu có modifier trước, sau, bổ nghĩa số lượng và khẩu ngữ. Chỉ ra loại nào luật minh họa nhận sai/bỏ sót.

## 32. Những điểm còn tranh luận

Các vấn đề là phép toán nào phù hợp với mức độ ngôn ngữ, hệ số có chuyển được giữa miền và cách kết hợp phủ định/cường độ. Điểm số thuận tiện cho tính toán không tự trở thành thang tâm lý có khoảng cách đều.

Nghiên cứu cần xác định mình muốn bảo toàn thứ tự, dự đoán nhãn hay ước lượng cường độ; sau đó dùng bằng chứng phù hợp. Một phương án có thể tốt về thứ tự nhưng chưa tốt về sai số tuyệt đối, nên không nên gộp hai kết luận.
