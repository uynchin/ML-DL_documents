# SenticNet: giải thích đơn giản

**Mục lục**

- [1. SenticNet là gì](#muc-1)
- [2. So sánh với sáu cách trước](#muc-2)
- [3. Ai làm gì](#muc-3)
- [4. Cách tính điểm baseline](#muc-4)
- [5. Ví dụ bốn tình huống](#muc-5)
- [6. Lỗi hay gặp](#muc-6)
- [7. Làm sao biết kết quả đáng tin](#muc-7)
- [8. Chọn cái nào trong bảy cái](#muc-8)
- [9. Mã tính thử](#muc-9)
- [10. Tài liệu tham khảo](#muc-10)

<a id="muc-1"></a>

**1. SenticNet là gì**

SenticNet là một kho tri thức về cảm xúc, nhưng đơn vị tra không phải từng từ mà là **khái niệm**, tức là cả cụm có nhiều từ. Ví dụ "a waste of money" được coi là một khái niệm: một nhận xét về giá trị, chứ không phải cộng nghĩa của "waste" và nghĩa của "money" [S1] [S2].

Hình dung như một cuốn sổ ghi ý nghĩa của cả cụm. Các cách trước phần lớn chỉ tra từng từ, còn SenticNet tìm cả cụm trước.

Có ba thứ khác nhau cùng mang tên SenticNet, dễ lẫn nhau.

| Thứ | Là gì |
|---|---|
| Tài nguyên tải về | Kho khái niệm và thuộc tính để tra cứu, có nhiều phiên bản |
| Sentic API | Dịch vụ làm các nhiệm vụ như trích khái niệm và phân loại cực tính [S3] |
| Hệ neurosymbolic | Hướng nghiên cứu kết hợp tri thức tường minh với học máy, ví dụ SenticNet 7 [S2] |

Tra cứu tài nguyên không có nghĩa là đã dùng toàn bộ hệ suy luận. Nếu chỉ lấy trung bình điểm các khái niệm tìm được thì đó là một baseline dựa trên SenticNet, không phải thuật toán đầy đủ.

Nguồn chính thức ghi các phiên bản SenticNet 1 năm 2010, SenticNet 7 năm 2022, SenticNet 8 năm 2024 và SenticNet 9 năm 2026 [S1]. Cách xây tài nguyên thay đổi qua các phiên bản, nên không lấy mô tả của một phiên bản áp cho phiên bản khác.

Một điều quan trọng: tìm được cụm vẫn chưa đủ để biết cụm đó đang được khẳng định, phủ định hay trích dẫn.

<a id="muc-2"></a>

**2. So sánh với sáu cách trước**

Thông tin về sáu cách trước (SentiWordNet, VADER, TextBlob, AFINN, NRC EmoLex, LIWC) lấy từ các file trước.

| Điểm so sánh | Sáu cách trước | SenticNet |
|---|---|---|
| Đơn vị tra | Phần lớn là từng từ, một số có cụm | Khái niệm, gồm cụm nhiều từ |
| Có bước trích khái niệm trước khi tra | Không | Có, nên có thêm một chỗ có thể sai |
| Đầu ra | Điểm hoặc bảng đếm | Thuộc tính của khái niệm, rồi tùy cách tổng hợp. Dịch vụ API có thể trả nhãn hoặc điểm khác |
| Xử lý "not" | Khác nhau tùy cách | Phải tự xử lý hoặc dùng bộ suy luận |
| Phiên bản | Ít thay đổi | Nhiều phiên bản, cần ghi rõ |
| Dễ kiểm tra bằng tay | AFINN dễ nhất | Khó hơn, vì có thêm bước trích khái niệm |
| Tiếng Việt | Không dùng trực tiếp | Cần xác nhận ngôn ngữ và tài nguyên cụ thể đang dùng, hỗ trợ nhiều ngôn ngữ không bảo đảm chính xác như nhau |

Điểm khác biệt chính: SenticNet cố hiểu cả cụm. Đổi lại, lỗi ở bước trích khái niệm sẽ truyền xuống mọi bước sau.

Đặt bảy cách cạnh nhau trên cùng câu: "It was not a waste of money."

| Cách | Điều gì xảy ra |
|---|---|
| SentiWordNet | Tra "waste" ra chê. Cần quy tắc "not" do người viết |
| VADER | Tự nhận ra "not", đảo ý |
| TextBlob | Có xử lý một số dấu hiệu, cần chạy mã để biết |
| AFINN | Tra "waste" ra chê, sai, trừ khi danh sách có sẵn cụm phù hợp |
| NRC EmoLex | Đếm "waste" vào nhóm chê nếu có trong từ điển. Chữ "not" không làm đổi gì |
| LIWC | Đếm từ theo các ngăn, không chấm khen chê |
| SenticNet | Tra được cả cụm "waste of money" ra chê. Nếu chỉ tra mà không xử lý "not" thì vẫn sai |

Cả bảy đều có cùng một điểm yếu: tìm thấy cụm hoặc từ chưa cho biết nó đang được khẳng định hay phủ định.

<a id="muc-3"></a>

**3. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Quyết định chấm cả câu hay theo từng đối tượng (giá trị, nhân viên) | Người |
| Chọn phiên bản tài nguyên và bộ trích khái niệm | Người |
| Viết quy tắc phủ định | Người viết, máy áp dụng |
| Chọn cách xử lý cụm chồng lấn, ví dụ ưu tiên cụm dài nhất | Người |
| Chọn trọng số cho từng khái niệm và ngưỡng khen chê | Người |
| Gán nhãn đáp án cho một tập câu mẫu | Người |
| Trích khái niệm từ văn bản | Máy |
| Ánh xạ khái niệm vào tài nguyên và tra thuộc tính | Máy |
| Tổng hợp điểm, đổi thành nhãn | Máy |
| Kiểm tra một mẫu xem khái niệm máy tìm có đúng ý không | Người |
| So kết quả với đáp án, tính chỉ số | Máy tính, người đọc các câu sai |
| Kết luận trong bài viết | Người |

Thuộc tính của từng khái niệm trong tài nguyên do nhóm SenticNet xây sẵn, người dùng không gán lại. Cách xây thay đổi theo phiên bản, đọc ở [S1] và [S2].

<a id="muc-4"></a>

**4. Cách tính điểm baseline**

Một baseline do người nghiên cứu tự thiết kế là trung bình có trọng số điểm các khái niệm tìm được:

```text
S = (w1 nhân p1 cộng w2 nhân p2 cộng ... cộng wm nhân pm) chia (w1 cộng w2 cộng ... cộng wm)
```

| Ký hiệu | Ý nghĩa |
|---|---|
| p1, p2, ..., pm | Điểm của từng khái niệm tìm được |
| w1, w2, ..., wm | Trọng số của từng khái niệm, do người đặt, lớn hơn 0 |
| m | Số khái niệm tìm được. Nếu m bằng 0 thì ghi là thiếu bằng chứng, không ghi là trung tính |
| S | Điểm của cả văn bản |

Công thức này không phải công thức chính thức của SenticNet hay của Sentic API. Nếu mọi trọng số bằng nhau thì S là trung bình thường. Trọng số cần có lý do và phải chọn trước khi nhìn tập kiểm tra.

Điểm polarity của khái niệm không phải xác suất khách hài lòng. Điểm của API khác cũng không chắc cùng thang đo với điểm trong tệp tài nguyên, nên phải đọc cấu trúc kết quả trước khi diễn giải.

<a id="muc-5"></a>

**5. Ví dụ bốn tình huống**

Từ điển nhỏ và các điểm dưới đây là **giả định**, không phải dữ liệu thật của SenticNet. Số liệu ở các tình huống 1, 2 và 4 đã được chạy bằng đoạn mã ở mục 9.

| Khái niệm hoặc từ | Điểm giả định |
|---|---|
| waste of money | -0.8 |
| waste | -0.5 |
| friendly | 0.6 |

**Tình huống 1: tra cả cụm khác với tra từng từ.** Câu: "It was a waste of money."

| Cách | Cách tìm | Điểm |
|---|---|---|
| Tra từng từ | Chỉ có "waste" nằm trong danh sách, "money" không có điểm | -0.5 |
| Tra cụm | Tìm thấy "waste of money" | -0.8 |

Cách tra cụm cho điểm chê mạnh hơn, đúng với ý của cụm. Nhưng điều này chỉ đúng khi bộ trích khái niệm nhận ra cụm. Nếu bộ trích cắt sai cụm thì kết quả rơi về tra từng từ, và lợi thế của tra cụm mất.

**Tình huống 2: một câu có hai ý trái chiều.** Câu: "The hotel was a waste of money, although the staff were friendly."

| Bước | Máy làm | Kết quả |
|---|---|---|
| 1 | Tìm khái niệm | "waste of money" và "friendly" |
| 2 | Tra điểm | -0.8 và 0.6 |
| 3 | Trung bình, trọng số bằng nhau | (-0.8 cộng 0.6) chia 2, bằng -0.1 |
| 4 | So với khoảng trung tính giả định từ -0.2 đến 0.2 | Trung bình |

Điểm -0.1 nghe như trung tính, nhưng câu nói rõ khách chê giá trị và khen nhân viên. Có hai cách xử lý.

| Cách | Cách làm | Kết quả |
|---|---|---|
| Đổi trọng số | Người đặt trọng số 2 cho nhận xét về giá trị, 1 cho nhân viên | (2 nhân -0.8 cộng 1 nhân 0.6) chia 3, khoảng -0.33, nghiêng về chê |
| Tách theo đối tượng | Báo riêng giá trị là -0.8 và nhân viên là 0.6 | Hai nhận xét rõ ràng, không bù trừ |

Đổi trọng số làm điểm nghiêng về chê, nhưng trọng số 2 và 1 phải có lý do, không được chọn sau khi xem tập kiểm tra. Cách tách theo đối tượng giữ được thông tin quản trị, như các file trước.

**Tình huống 3: có chữ "not".** Câu: "It was not a waste of money."

| Cách | Cách làm | Kết quả |
|---|---|---|
| Chỉ tra cụm | Tìm "waste of money" ra -0.8 | Chê, sai |
| Có quy tắc phủ định | Đảo dấu điểm của cụm nằm trong phạm vi "not", cho +0.8 | Khen, đúng |

Đảo dấu +0.8 là giả định, đảo ý không phải lúc nào cũng cho đúng độ mạnh đối xứng. Đảo dấu mọi khái niệm sau "not" cũng có thể sai khi qua ranh giới của một mệnh đề. Vì vậy cần kiểm tra bằng các cặp câu có và không có "not".

**Tình huống 4: đếm đôi khi cụm chồng lấn.** Câu: "The hotel was a waste of money."

Cụm "waste of money" và từ "waste" bên trong nó đều có trong từ điển.

| Cách | Cách làm | Tổng điểm |
|---|---|---|
| Cộng cả cụm lẫn từ bên trong | -0.8 cộng -0.5 | -1.3 |
| Ưu tiên cụm dài nhất, bỏ từ bên trong | Chỉ lấy -0.8 | -0.8 |

Cách đầu đếm một bằng chứng hai lần. Ưu tiên cụm dài nhất là một cách xử lý hợp lý, nhưng chỉ là heuristic, tức một quy tắc đoán, cần thử và không bảo đảm đúng cho mọi biểu thức.

<a id="muc-6"></a>

**6. Lỗi hay gặp**

1. Gọi phép lấy trung bình điểm khái niệm là toàn bộ thuật toán SenticNet. Đó chỉ là baseline do người nghiên cứu tự thiết kế.
2. Coi cụm không tìm thấy là trung tính. Cần ghi riêng là thiếu bằng chứng.
3. Trộn kết quả từ nhiều phiên bản tài nguyên mà không lưu phiên bản.
4. Không tách các nguồn lỗi. Lỗi có thể nằm ở bộ trích khái niệm, ở bước ánh xạ vào tài nguyên, hoặc ở bước tổng hợp. Nếu không tách thì không biết sửa chỗ nào.
5. Coi tri thức phong phú là bảo đảm kết quả tốt hơn. Tri thức chung có thể không hợp một phân khúc khách sạn, và sai số trích khái niệm truyền xuống các bước sau.
6. Tin vào lời giải thích nghe hợp lý. Việc nhìn thấy khái niệm trung gian giúp kiểm tra, nhưng chưa chứng minh lời giải thích trung thành với mọi phép tính của hệ thống.
7. Đưa khóa API hoặc dữ liệu khách có thông tin nhận diện lên GitHub công khai.
8. Gửi văn bản ra dịch vụ ngoài mà chưa xem điều kiện sử dụng dữ liệu nghiên cứu.
9. Dùng nhãn tự động để đánh giá nhân viên hoặc đoán tâm lý một cá nhân.

<a id="muc-7"></a>

**7. Làm sao biết kết quả đáng tin**

SenticNet có nhiều bước nối tiếp, nên cần kiểm tra từng bước, không chỉ kiểm tra kết quả cuối.

Bước 1. Kiểm tra bước trích và ánh xạ khái niệm.

1. Chọn khoảng 100 câu mẫu từ nhiều khách sạn khác nhau.
2. Người đọc từng câu, ghi khái niệm nào đúng ra phải được tìm thấy.
3. So với khái niệm máy tìm được, tính tỷ lệ máy tìm đúng.
4. Tính tỷ lệ câu máy tìm được ít nhất một khái niệm (coverage).

Bước 2. Tạo đáp án khen chê.

1. Hai người gán nhãn khen, chê hoặc trung tính cho cùng các câu, mỗi người làm riêng.
2. Máy tính mức hai người đồng ý với nhau, gọi là hệ số kappa. Càng gần 1 thì càng đồng ý.
3. Chỗ hai người khác nhau thì bàn lại và chọn một nhãn.
4. Chia đáp án thành phần thử nghiệm và phần chấm điểm cuối, chia theo khách sạn. Phần chấm điểm cuối chỉ dùng một lần.

Bước 3. Đo kết quả cuối trên phần chấm điểm cuối.

| Con số | Ý nghĩa đơn giản |
|---|---|
| Độ đúng trung bình của ba nhãn (macro-F1) | Máy đúng bao nhiêu, tính riêng cho khen, chê, trung tính rồi lấy trung bình |
| Bảng nhầm lẫn | Máy hay nhầm nhãn nào thành nhãn nào |
| Coverage | Máy hiểu được bao nhiêu phần dữ liệu |

Bước 4. So sánh để biết tri thức khái niệm có giúp không. Làm kiểu thêm hoặc bớt từng thành phần rồi đo lại, gọi là ablation, trên cùng cách chia dữ liệu.

| Phiên bản thử | Cách làm |
|---|---|
| Chỉ từ đơn | Tra từng từ, không dùng khái niệm |
| Khái niệm | Tra cả cụm |
| Khái niệm cộng xử lý ngữ cảnh | Tra cụm và có quy tắc phủ định |

Nếu bản khái niệm không hơn bản từ đơn thì tri thức khái niệm chưa giúp được gì trên dữ liệu này. Nếu mục tiêu là áp dụng sang khách sạn hoặc giai đoạn mới, thử thêm trên dữ liệu của khách sạn hoặc thời gian chưa dùng khi chỉnh.

Khi viết kết luận, điểm theo khái niệm cho biết hệ thống đánh giá biểu thức ra sao. Nó chưa đo trải nghiệm toàn diện hay ý định quay lại. Một ví dụ giải thích nghe hợp lý cũng chưa phải bằng chứng thống kê về hiệu năng của cả mô hình.

<a id="muc-8"></a>

**8. Chọn cái nào trong bảy cái**

| Nhu cầu | Cách hợp hơn | Điểm yếu |
|---|---|---|
| Mốc khen chê đơn giản nhất, kiểm tra bằng tay | AFINN | Không có quy tắc nào về ngữ cảnh |
| Phân biệt các nghĩa khác nhau của cùng một từ | SentiWordNet | Phải tự chọn nghĩa |
| Có chữ hoa, dấu chấm than, "very", "but", "not" | VADER | Mỗi từ chỉ có một điểm |
| Biết câu thiên về ý kiến hay sự việc | TextBlob | Chỉ ước lượng theo từ điển |
| Hồ sơ nhiều nhóm cảm xúc | NRC EmoLex | Không có cường độ, không xử lý "not" |
| Nhiều chỉ báo ngôn ngữ cho giả thuyết về phong cách | LIWC | Cần quyền dùng, cần thiết kế nghiên cứu chặt |
| Dữ liệu có nhiều cụm nhiều từ, muốn kiểm tra vai trò của tri thức khái niệm | SenticNet | Lỗi bộ trích khái niệm truyền xuống, cần ghi phiên bản, khó tái lập hơn nếu dùng API |
| Đúng hơn trên đánh giá khách sạn | Chưa biết | Phải đo trên dữ liệu của nghiên cứu |

Nên thử SenticNet khi dữ liệu có nhiều biểu thức nhiều từ và nghiên cứu cần kiểm tra vai trò của tri thức khái niệm. Làm một đợt thử nhỏ để đo mức ánh xạ đúng, độ bao phủ và lỗi cuối. Nếu bộ trích thường cắt sai cụm thì tài nguyên giàu tri thức cũng không phát huy được.

Không nên dùng tra cứu đơn thuần để giải quyết mỉa mai, ngữ cảnh dài hay cảm xúc cá nhân.

<a id="muc-9"></a>

**9. Mã tính thử**

Đoạn mã dưới đây thử ý tưởng ở mục 5 với từ điển khái niệm giả định. Nó không dùng dữ liệu hay API của SenticNet. Bộ khớp ưu tiên cụm dài nhất, giống cách xử lý ở tình huống 4. Đoạn mã này đã được chạy và cho đúng các số -0.8, -0.1 và -0.333 ở mục 5.

```python
# Từ điển khái niệm giả định, không phải dữ liệu SenticNet.
concepts = {
    "waste of money": -0.8,
    "waste": -0.5,
    "friendly": 0.6,
}
max_len = 3

def match_concepts(text):
    tokens = text.lower().replace(",", "").replace(".", "").split()
    found = []
    i = 0
    while i < len(tokens):
        for length in range(max_len, 0, -1):
            phrase = " ".join(tokens[i:i + length])
            if phrase in concepts:
                found.append((phrase, concepts[phrase]))
                i += length
                break
        else:
            i += 1
    return found

def score(found, weights=None):
    if not found:
        return None   # thiếu bằng chứng, không phải trung tính
    if weights is None:
        weights = [1] * len(found)
    total = sum(w * p for w, (_, p) in zip(weights, found))
    return total / sum(weights)

text = "The hotel was a waste of money, although the staff were friendly."
found = match_concepts(text)
print(found, round(score(found), 3))
print(round(score(found, [2, 1]), 3))
```

Cách tách `.split()` chỉ phù hợp tiếng Anh ngắn, không phải cách tách từ tiếng Việt. Khi làm với dữ liệu thật, thay từ điển giả định bằng tài nguyên lấy từ nguồn chính thức [S1], hoặc dùng API theo cấu trúc hiện hành ở [S3]. Cần đối chiếu quyền truy cập trước khi gọi API thật, và ghi lại thông tin phiên bản.

<a id="muc-10"></a>

**10. Tài liệu tham khảo**

- **[S1]** SenticNet. [Downloads và lịch sử phiên bản](https://sentic.net/downloads/).
- **[S2]** Cambria, E., Liu, Q., Decherchi, S., Xing, F., & Kwok, K. (2022). *SenticNet 7: A Commonsense-based Neurosymbolic AI Framework for Explainable Sentiment Analysis*. [LREC](https://aclanthology.org/2022.lrec-1.408/).
- **[S3]** SenticNet. [Mô tả các API](https://sentic.net/api/).
- **[S4]** SenticNet. [Tổ chức GitHub chính thức](https://github.com/senticnet).

Phần so sánh với sáu cách trước dựa trên các file trước. Ví dụ khách sạn, từ điển giả định, công thức baseline và kế hoạch kiểm tra ở trên là đề xuất của người biên soạn, không phải kết quả thí nghiệm khách sạn trong các nguồn này.
