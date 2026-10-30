# Chuẩn hóa tiếng lóng, viết tắt và dạng viết không chuẩn

> **Nhánh:** LEXICON BASED → Xử lý quy tắc. **Nguồn đề mục:** dòng 14 trong bảng Excel, “Chuẩn hóa tiếng lóng”. **Ngày đối chiếu:** 28/10/2026. Bài mở rộng phạm vi thực hành sang viết tắt/biến thể chữ; không đồng nhất mọi tiếng lóng với lỗi chính tả. Ví dụ, mapping và số liệu minh họa được tự xây dựng.

| Trường | Nội dung |
|---|---|
| Bản chất | Thành phần tiền xử lý hoặc mô hình chuẩn hóa từ vựng; không phải bộ phân loại sentiment |
| Đầu vào | Văn bản gốc, ngữ cảnh, mapping/mô hình và chính sách giữ nguyên |
| Đầu ra | Bản chuẩn hóa, bản gốc, nhật ký thay đổi và trạng thái không chắc |
| Mốc minh họa | ViLexNorm, 2024; ViSoLex, 2025 [S1–S3]; không coi đây là năm xuất hiện chuẩn hóa từ vựng |
| Tiếng Việt | Là trọng tâm bài; cần xét dấu, từ nhiều âm tiết, từ trộn và cách dùng cộng đồng |
| Nhãn | Mapping luật có thể không cần huấn luyện; mô hình học cần cặp dữ liệu hoặc nguồn giám sát; kiểm định cần chuẩn độc lập |
| Tính toán | Tra mapping nhỏ thường dùng CPU; mô hình sinh/chuẩn hóa theo ngữ cảnh có chi phí riêng |
| Dữ liệu tham khảo | ViLexNorm; cần đọc điều kiện sử dụng và kiểm tra chuyển miền sang đánh giá khách sạn |

## 1. Định nghĩa học thuật

**Lexical normalization** là chuyển dạng từ vựng về dạng chuẩn theo quy ước của nhiệm vụ. ViLexNorm nghiên cứu nhiệm vụ này cho văn bản mạng xã hội tiếng Việt [S1]. Trong ứng dụng sentiment, chuẩn hóa nhằm làm biểu thức dễ được tài nguyên/mô hình nhận diện trong khi bảo toàn ý nghĩa cần phân tích.

Tiếng lóng có thể mang nghĩa xã hội hoặc sắc thái riêng; không phải mọi biểu thức khác văn viết trang trọng đều sai. Quyết định chuẩn hóa phải dựa trên mục tiêu nghiên cứu và ý nghĩa trong ngữ cảnh.

## 2. Giải thích đơn giản

Nếu từ điển biết “không” nhưng không biết “ko”, hệ thống có thể bỏ lỡ phủ định trong “phòng ko sạch”. Chuẩn hóa đúng giúp chuyển thành “phòng không sạch”.

Nhưng “k” có thể là phủ định hoặc đơn vị nghìn trong “500k”. Thay mọi “k” thành “không” sẽ phá nghĩa. Công việc không phải làm câu đẹp hơn bằng mọi giá, mà là đổi có căn cứ và giữ nguyên khi chưa chắc.

## 3. Mục đích và vấn đề giải quyết

Đề xuất nghiên cứu khách sạn: kiểm tra việc chuẩn hóa biến thể có tăng coverage từ điển và giảm lỗi sentiment không. Ví dụ “nv ko nhiệt tình” cần nhận ra “nhân viên” và phủ định trước khi phân tích thái độ.

Chuẩn hóa cũng có thể tạo dữ liệu thống nhất để phân tích chủ đề/khía cạnh. Tuy nhiên, nếu nghiên cứu phong cách khách hàng hoặc mức nhấn mạnh, một số dạng không chuẩn chính là thông tin cần giữ, không chỉ nhiễu cần xóa.

## 4. Điều kiện và giả định sử dụng

Cần quy định chuẩn đích: dạng nào được đổi, dạng nào giữ và xử lý khi nhiều cách hiểu. Mapping phải phù hợp ngôn ngữ/miền, không chỉ danh sách truyền miệng. Với mô hình học, cặp đầu vào–đích cần hướng dẫn gán nhãn và căn chỉnh đủ rõ.

Giả định cần kiểm tra là thay thế bảo toàn sentiment và đối tượng. Một dạng chuẩn hơn về chính tả vẫn có thể làm mất mỉa mai, cường độ, mức lịch sự hoặc đặc trưng cộng đồng.

## 5. Cơ chế hoạt động bên trong

Quy trình đề xuất: lưu bản gốc → nhận diện ứng viên → lấy các cách chuẩn hóa → xét ngữ cảnh → chọn thay thế hoặc giữ → ghi span và lý do → đưa vào tác vụ sau.

Mapping đơn giản thay token đã biết. Phương án theo ngữ cảnh quyết định giữa nhiều đích, có thể xử lý một token thành nhiều token hoặc nhiều token thành một cụm. Sau thay thế, offset và token count có thể đổi; cần lưu liên kết với bản gốc để kiểm tra.

## 6. Quy trình áp dụng trong nghiên cứu

1. Xác định mục tiêu downstream và chuẩn đích.
2. Lấy mẫu đúng miền, lập taxonomy biến thể và hướng dẫn giữ/đổi.
3. Chia tập trước khi xây mapping/học mô hình.
4. Tạo baseline giữ nguyên văn bản.
5. Xây mapping hoặc mô hình trên dữ liệu cho phép; kiểm tra các trường hợp đa nghĩa.
6. Lưu raw/normalized text và nhật ký thay đổi.
7. Đánh giá độ đúng chuẩn hóa và hiệu quả tác vụ sau trên test độc lập.
8. Phân tích lỗi biến đổi nhầm, bỏ sót và mất sắc thái.

Không lấy toàn bộ test để bổ sung mapping rồi báo hiệu năng như chưa từng thấy các dạng đó.

## 7. Ví dụ minh họa xuyên suốt

Văn bản tự soạn: “Phòng ko sạch, nv thân thiện :)”. Trong bối cảnh đã xác nhận, có thể đổi `ko → không`, `nv → nhân viên`, giữ emoticon để bước tiếp theo xử lý. Kết quả là “Phòng không sạch, nhân viên thân thiện :)”.

Sau chuẩn hóa, mô hình vẫn phải xử lý phủ định và hai khía cạnh. Mapping đúng chỉ phục hồi tín hiệu đầu vào; nó không tự kết luận sentiment của toàn bộ đánh giá.

## 8. Kết quả và cách diễn giải

Đầu ra là văn bản theo chuẩn đích và danh sách thay đổi, không phải nhãn khen/chê. Tỷ lệ thay nhiều hơn không đồng nghĩa tốt hơn: hệ thống có thể đang sửa cả các từ đúng.

Cần phân biệt không phát hiện ứng viên, phát hiện nhưng chọn giữ, và đã sửa. Với một dạng đa nghĩa, giữ nguyên có thể là quyết định hợp lý nếu thiếu ngữ cảnh; không nên bắt hệ thống sửa để đạt số lượng thay thế cao.

## 9. Điểm mạnh, điểm yếu và giới hạn

Mapping nhỏ có thể minh bạch, dễ kiểm tra và giúp tài nguyên nhận lại những biểu thức thường gặp. Mô hình ngữ cảnh có thể xử lý biến thể không nằm trong bảng và trường hợp cần lựa chọn đích.

Hạn chế là sửa quá tay, mapping nhanh lỗi thời, khác cộng đồng và sai số mô hình sinh. Một bộ chuẩn hóa cũng có thể làm câu trôi chảy nhưng thêm/bỏ nội dung. Chất lượng phải được đo bằng ý nghĩa cần bảo toàn, không chỉ sự tự nhiên của câu mới.

## 10. So sánh với phương pháp gần nhau

| Tác vụ | Khác biệt |
|---|---|
| Sửa chính tả | Sửa dạng viết lỗi; không bao quát mọi tiếng lóng/viết tắt |
| Chuẩn hóa từ vựng | Chuyển về dạng đích theo quy ước nhiệm vụ |
| Tách từ | Xác định đơn vị; không tự quyết định “ko” thành “không” |
| Khôi phục dấu | Thêm dấu phù hợp; có thể đa nghĩa và cần ngữ cảnh |
| Dịch máy | Chuyển ngôn ngữ, khác mục tiêu chuẩn hóa trong cùng ngôn ngữ |
| Viết lại cho trang trọng | Có thể đổi phong cách nhiều hơn mức cần cho phân tích |

Phân biệt các tác vụ giúp biết thành phần nào gây lỗi thay vì gom mọi tiền xử lý vào một thao tác “làm sạch”.

## 11. Sai lầm thường gặp và cách khắc phục

- Thay chuỗi con toàn văn: dùng ranh giới và ngữ cảnh, tránh biến `500k` thành nội dung vô nghĩa.
- Tự suy mọi từ lạ là lỗi: bảo vệ tên riêng, địa danh, thương hiệu và từ vay mượn.
- Xóa kéo dài chữ mà không lưu: giữ cờ nhấn mạnh hoặc bản gốc.
- Dịch mọi tiếng lóng sang một từ mạnh hơn: thẩm định mức độ và ý nghĩa.
- Chỉ giữ bản chuẩn hóa: lưu raw text và mapping span để truy lại.

## 12. Tài liệu tham khảo

- **[S1]** Nguyen, T.-N., Le, T.-P., & Nguyen, K. (2024). *ViLexNorm: A Lexical Normalization Corpus for Vietnamese Social Media Text*. [EACL](https://aclanthology.org/2024.eacl-long.85/).
- **[S2]** Nhóm ViLexNorm. [Kho dữ liệu và hướng dẫn](https://github.com/ngxtnhi/ViLexNorm).
- **[S3]** *ViSoLex: An Open-Source Repository for Vietnamese Social Media Lexical Normalization* (2025). [Bài báo công bố hệ thống](https://aclanthology.org/2025.coling-demos.18/).

S1–S2 là nguồn nhiệm vụ và dữ liệu; S3 là hệ thống liên quan. Không dùng kết quả công bố ở các nguồn này làm hiệu năng cho dữ liệu khách sạn của dự án. Mapping “ko/nv” và mã của bài được tự xây, không trích xuất từ ViLexNorm.

## 13. Vị trí trong toàn bộ nghiên cứu

Thành phần thường nằm sau đọc/giữ dữ liệu gốc, trước tách biểu thức và xử lý phủ định/cường độ. Thứ tự cụ thể phụ thuộc mô hình: có thể cần nhận cụm trước chuẩn hóa hoặc tách từ lại sau thay thế.

Nếu mô hình ngữ cảnh được huấn luyện với dạng viết tự nhiên, chuẩn hóa không bảo đảm giúp nó. Nên xem chuẩn hóa là giả thuyết cần thử trong pipeline, không là bước bắt buộc cho mọi bài toán.

## 14. Kiến thức nền cần biết trước

Cần hiểu dạng chuẩn/dạng biến thể, OOV, token, span và căn chỉnh. Trong tiếng Việt, một đơn vị từ có thể chứa nhiều âm tiết cách nhau bằng khoảng trắng; thay `nv` thành `nhân viên` làm số đơn vị theo `.split()` tăng nhưng chưa quyết định cách tách từ đúng.

Phân biệt tên riêng với tiếng lóng và phân biệt **có trong từ điển** với **đúng trong ngữ cảnh**. Một từ đúng mặt chữ vẫn có thể là lỗi dùng nhầm, còn từ không có trong từ điển vẫn có thể hợp lệ.

## 15. Nền tảng lý thuyết và logic hình thành

Logic là nhiều hình thức bề mặt có thể biểu đạt nội dung gần nhau, nên đưa chúng về dạng nhất quán có thể giảm phân mảnh dữ liệu. Nhưng quan hệ không luôn một–một: một viết tắt có nhiều nghĩa, và một tiếng lóng có thể mang sắc thái không được một từ chuẩn thay thế đầy đủ.

Nghiên cứu cần xem dạng chuẩn như một quy ước vận hành hóa có mục tiêu. Không thể suy rằng văn bản chuẩn hóa tự động khách quan hoặc đúng hơn lời người viết trong mọi nghĩa.

## 16. Các thành phần cấu tạo

Một hệ gồm chính sách chuẩn đích, bộ nhận ứng viên, từ điển/model sinh ứng viên, cơ chế chọn theo ngữ cảnh, danh sách bảo vệ và log thay đổi. Tác vụ sau cần biết những thông tin như phủ định/cường độ có bị biến đổi hay không.

Với mô hình sinh, có thể cần kiểm tra giới hạn chỉ thay phần được phép, phát hiện thêm nội dung và giữ dạng khi không chắc. Không coi văn bản mới trôi chảy là đủ tiêu chuẩn.

## 17. Các loại và biến thể

Có thể dùng mapping thủ công, luật ký tự, chọn ứng viên từ khoảng cách chỉnh sửa, mô hình theo ngữ cảnh hoặc kết hợp. Thay thế có thể một–một, một–nhiều hoặc nhiều–một theo định nghĩa đơn vị.

Khôi phục dấu, giảm lặp chữ và chuẩn hóa viết tắt là các nhóm có thể thử riêng. Không gộp tất cả thành một cấu hình duy nhất nếu cần biết nguồn cải thiện/tác hại. Việc chuẩn hóa emoji thuộc chính sách riêng cần phối hợp với bài trước.

## 18. Đầu vào và đầu ra

Đề xuất lưu `raw_text`, `normalized_text`, `edit_start`, `edit_end`, `source_form`, `target_form`, `rule_id`, `decision`, `uncertainty`, cùng ID câu. Offset phải ghi thuộc bản gốc hay bản đã chuẩn hóa Unicode.

Nếu chỉ lưu văn bản sau sửa, rất khó tìm lại “ko” nào đã bị đổi. Nếu thay đổi làm dài/ngắn văn bản, các span nhiệm vụ sau cần mapping, không tái dùng offset cũ như thể chuỗi không đổi.

## 19. Căn cứ lựa chọn

Đo lỗi downstream có liên quan biến thể không và từ điển/model hiện tại có thật sự thiếu chúng không. Pilot nên lấy cả câu chuẩn và không chuẩn để biết hệ có phá dữ liệu vốn đúng không.

Chọn mapping khi tập biến thể ổn định, ít đa nghĩa và có thể thẩm định. Chọn mô hình theo ngữ cảnh khi cần phân biệt đích, nhưng phải có dữ liệu kiểm định và chi phí phù hợp. Không mặc định mô hình lớn hơn luôn bảo toàn ý nghĩa tốt hơn.

## 20. Trường hợp không nên sử dụng

Không chuẩn hóa mạnh khi nghiên cứu phong cách viết, bản sắc cộng đồng hoặc chính sự xuất hiện tiếng lóng, trừ khi giữ một nhánh dữ liệu gốc để phân tích. Không ép chuyển tên thương hiệu/địa danh lạ thành từ phổ biến.

Không dùng mapping mặc định cho các dạng như “k” khi thiếu ngữ cảnh. Nếu chỉ có từ đơn ngoài câu, có thể phải ghi nhiều ứng viên hoặc chưa quyết định thay vì đoán chắc.

## 21. Các quyết định quan trọng khi thực hiện

Phải chọn chuẩn đích, điều kiện sửa, ngưỡng giữ nguyên, danh sách bảo vệ và chính sách lặp chữ/dấu. “Đẹppp” có thể chuẩn hóa thành “đẹp” cho tra từ nhưng nên giữ thông tin kéo dài nếu nghiên cứu mức độ.

Quyết định chuẩn hóa khi huấn luyện và khi suy luận phải nhất quán hoặc được mô tả như một thí nghiệm riêng. Nếu chỉ sửa test mà mô hình được huấn luyện trên dạng khác, kết quả đổi có thể do lệch phân bố chứ không chỉ do độ đúng chuẩn hóa.

## 22. Giải thích công thức và ký hiệu

Gọi $x$ là văn bản gốc, $g$ là bộ chuẩn hóa, $f$ là hệ thống tác vụ sau. Đầu ra pipeline là:

$$\hat y=f(g(x)).$$

Baseline giữ nguyên là $g_{LAI}(x)=x$ — **Leave-As-Is**. So sánh hữu ích là chất lượng $f(g(x))$ với $f(x)$ theo giao thức rõ ràng. Nếu huấn luyện lại $f$ cho từng điều kiện, hãy báo đó là so sánh hai pipeline được huấn luyện riêng, khác thử chỉ thay đầu vào của một mô hình cố định.

Một tỷ lệ giảm lỗi minh họa tổng quát có thể là:

$$R=\frac{E_{LAI}-E_g}{E_{LAI}},\qquad E_{LAI}>0.$$

$E$ phải được đếm bằng **cùng chuẩn căn chỉnh và đơn vị đánh giá**. $R<0$ nghĩa lỗi tăng, $R=1$ là hết lỗi theo metric đó. Công thức này chỉ minh họa ý tưởng giảm lỗi; khi báo **ERR của ViLexNorm**, phải theo chính xác giao thức/implementation của benchmark, không thay bằng tùy ý một loại khoảng cách chữ.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Giả định baseline có 20 lỗi theo một giao thức đã cố định, bộ chuẩn hóa còn 8 lỗi. Tỷ lệ giảm lỗi minh họa là $(20-8)/20=0.6$. Nó không có nghĩa accuracy sentiment là 60%; đó là hai nhiệm vụ khác nhau.

Mã minh họa thay **token nguyên vẹn** trong mapping nhỏ theo giả định miền đã xác nhận. Nó giữ nguyên khoảng trắng, dấu câu và các token ngoài mapping. Không xử lý mọi tiếng lóng hoặc tự suy ngữ cảnh.

```python
import re

# Mapping tự đặt cho demo đánh giá khách sạn, chưa kiểm định corpus thật.
mapping = {"ko": "không", "nv": "nhân viên"}

def normalize_demo(text):
    pieces, edits, cursor = [], [], 0
    for match in re.finditer(r"\w+", text):
        source = match.group(0)
        target = mapping.get(source.casefold())
        if target is None:
            continue
        pieces.append(text[cursor:match.start()])
        pieces.append(target)
        edits.append({"start": match.start(), "end": match.end(),
                      "source": source, "target": target})
        cursor = match.end()
    pieces.append(text[cursor:])
    return "".join(pieces), edits

raw = "Phòng ko sạch, nv thân thiện :)"
normalized, edits = normalize_demo(raw)
assert normalized == "Phòng không sạch, nhân viên thân thiện :)"
assert raw == "Phòng ko sạch, nv thân thiện :)"
assert normalize_demo("Giá 500k; k rõ.")[0] == "Giá 500k; k rõ."
assert normalize_demo("Phòng đẹppp!")[0] == "Phòng đẹppp!"
assert normalize_demo(normalized)[0] == normalized
assert all(raw[e["start"]:e["end"]] == e["source"] for e in edits)
print(normalized)
# Phòng không sạch, nhân viên thân thiện :)
```

Các offset của demo thuộc raw string Python. Chúng không phải offset của chuỗi sau thay thế. Mapping thật cần điều kiện ngữ cảnh và danh sách bảo vệ; `nv` không luôn có một nghĩa trong mọi miền.

## 24. Tình huống sai và phản ví dụ

Trong “phòng 500k”, `k` chỉ nghìn; trong “k sạch”, nó có thể là phủ định. Một phép thay chuỗi con `k → không` sẽ phá cả giá tiền và nhiều từ không liên quan.

“Đỉnh” có thể là lời khen hoặc chỉ vị trí địa lý; “cháy” có thể là tiếng lóng hoặc mô tả nguy cơ thật. Không thay những từ này bằng “tốt” chỉ vì có trong danh sách mạng xã hội. Cần xét ngữ cảnh, đặc biệt với nội dung an toàn khách sạn.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Tầng chuẩn hóa: dùng cặp gốc–đích và chuẩn căn chỉnh phù hợp, đo đúng số sửa cần/sửa sai/bỏ sót; báo nhóm nhiều–một hoặc một–nhiều nếu liên quan. Tầng downstream: macro-F1 hoặc metric đúng mục tiêu, so cùng test và kiểm soát cấu hình mô hình.

Đo trên cả câu cần sửa và không cần sửa. Kiểm tra tỷ lệ phủ định bị mất, cường độ đổi và tên riêng sửa nhầm. Tăng coverage không đủ chứng minh tăng độ đúng; có thể lexicon nhận được nhiều từ hơn vì nghĩa đã bị sửa sai.

## 26. Xử lý khi không đáp ứng yêu cầu

Sửa nhầm nhiều: tăng điều kiện/giữ nguyên khi không chắc, bổ sung bảo vệ hoặc dùng ngữ cảnh. Bỏ sót nhiều: kiểm tra corpus, nguồn ứng viên và tách đơn vị. Mất cường độ: giữ cờ dạng kéo dài hoặc hai nhánh raw/normalized.

Nếu chuẩn hóa tốt theo benchmark nhưng sentiment giảm, xem mục tiêu, sự tương thích mô hình và các tín hiệu mất đi. Có thể chỉ áp một tập quy tắc chọn lọc; không cần ép mọi bước chuẩn hóa vào pipeline cuối.

## 27. Phân biệt kết quả với kết luận

“Có 10% token được thay” là kết quả thao tác. “Bản chuẩn hóa đúng hơn” cần đối chiếu chuẩn đích và ý nghĩa. “Sentiment chính xác hơn” cần đánh giá downstream độc lập.

Việc khách dùng viết tắt không nói lên mức học vấn, độ tin cậy hoặc chất lượng trải nghiệm. Tránh biến một bước hỗ trợ mô hình thành kết luận về con người hoặc cách giao tiếp của họ.

## 28. Cách viết vào bài nghiên cứu

Nêu chuẩn đích, phạm vi thay, nguồn mapping/dữ liệu, điều kiện ngữ cảnh, chính sách giữ nguyên và thứ tự pipeline. Báo tỷ lệ câu/token đổi, lỗi sửa nhầm và tác dụng downstream.

Khung: “Chúng tôi chuẩn hóa [nhóm dạng] bằng [phương án], giữ nguyên [trường hợp], và lưu mapping từ raw sang normalized. Tài nguyên được xây từ [phần dữ liệu], cấu hình chọn trên [development], kiểm định bằng [metric và protocol].” Nếu dùng ViLexNorm, nêu cách đánh giá chuyển miền sang dữ liệu khách sạn thay vì mặc định tương đương.

## 29. Khả năng tái lập và tính minh bạch

Lưu mapping/model, version, danh sách bảo vệ, ngưỡng, tokenizer, raw/normalized và log. Với mô hình sinh, ghi cấu hình suy luận và các thành phần không quyết định nếu có. Bảo đảm người kiểm tra thấy được chỗ thay, không chỉ bản văn trôi chảy cuối.

Kho GitHub có thể chứa code, mapping tự tạo có quyền chia sẻ và ví dụ giả định. Điều kiện dữ liệu ViLexNorm cần theo nguồn chính thức [S1–S2]; không mặc định bộ dữ liệu nghiên cứu được phép phân phối lại cho mọi mục đích.

## 30. Đạo đức nghiên cứu liên quan

Tiếng lóng và phương ngữ có giá trị giao tiếp. Không gọi cách viết của cộng đồng là kém chuẩn theo nghĩa phán xét con người chỉ vì mô hình khó xử lý. Giải thích chuẩn hóa là lựa chọn kỹ thuật phục vụ nhiệm vụ.

Không viết lại lời phàn nàn theo hướng nhẹ đi/mạnh lên rồi trích như nguyên văn. Nếu công bố bản chuẩn hóa, ghi rõ đã xử lý và bảo vệ thông tin nhận diện, nhất là dữ liệu liên quan sự cố khách sạn.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **Mọi token lạ có phải lỗi cần sửa không?** Không.
2. **Chuẩn hóa đúng có bảo đảm sentiment tăng không?** Không; cần đánh giá pipeline cuối.
3. **Bài tập tính:** baseline 20 lỗi, mô hình mới 24 lỗi cho $R=-0.2$. Giải thích vì sao báo “giảm lỗi 20%” là sai.
4. **Bài tập dữ liệu:** chọn 20 phản hồi, đánh dấu ứng viên chắc/đa nghĩa, đề xuất đích và phần cần giữ. Nhờ hai người đánh giá độc lập theo cùng hướng dẫn.
5. **Bài tập phản ví dụ:** tìm ba cách hiểu của một viết tắt trong các miền khác nhau, rồi thiết kế quy tắc giữ nguyên khi thiếu ngữ cảnh.

## 32. Những điểm còn tranh luận

Các vấn đề chính là định nghĩa chuẩn đích, mức chấp nhận viết lại, bảo toàn sắc thái và khác biệt giữa cộng đồng. Một bản đích có thể hợp lý cho tra từ nhưng không thích hợp cho nghiên cứu phong cách.

Vì vậy, không đánh giá chuẩn hóa chỉ bằng việc câu nhìn trang trọng hơn. Cần công bố mục tiêu và đo cả độ đúng biến đổi lẫn tác dụng nghiên cứu sau đó. Khi nhiều dạng đích hợp lệ, chuẩn đánh giá cũng cần tránh phạt một cách viết đúng chỉ vì khác duy nhất một bản tham chiếu.
