# LIWC: giải thích đơn giản

**Mục lục**

- [1. LIWC là gì](#muc-1)
- [2. So sánh với năm cách trước](#muc-2)
- [3. Ai làm gì](#muc-3)
- [4. Cách tính tỷ lệ](#muc-4)
- [5. Ví dụ bốn tình huống](#muc-5)
- [6. Lỗi hay gặp](#muc-6)
- [7. Làm sao biết kết quả đáng tin](#muc-7)
- [8. Chọn cái nào trong sáu cái](#muc-8)
- [9. Cách thao tác và mã tính thử](#muc-9)
- [10. Tài liệu tham khảo](#muc-10)

<a id="muc-1"></a>

**1. LIWC là gì**

LIWC là viết tắt của Linguistic Inquiry and Word Count. Đây là một công cụ kèm từ điển, dùng để đo xem văn bản dùng những nhóm từ nào. File này lấy bản LIWC-22 làm mốc [S1].

LIWC khác các cách trước ở chỗ nó không chỉ nói về khen chê hay cảm xúc. Từ điển của nó chia từ thành nhiều nhóm ngôn ngữ có ý nghĩa tâm lý hoặc xã hội, ví dụ nhóm về đại từ, nhóm về quan hệ xã hội, nhóm về cảm xúc. Tên và số nhóm cụ thể phụ thuộc phiên bản, nên cần đọc tài liệu chính thức của phiên bản đang dùng [S2] [S3].

**Hình dung đơn giản: một cái tủ nhiều ngăn kéo**

Mỗi ngăn kéo có một cái tên, ví dụ ngăn "từ chỉ chúng tôi" và ngăn "từ chỉ cảm xúc". LIWC đọc từng từ của văn bản rồi bỏ vào đúng ngăn. Đọc xong, nó đếm mỗi ngăn chiếm bao nhiêu phần trăm số từ.

LIWC chỉ đếm, không hiểu ý và không chấm khen chê. Các cách trước như VADER hay AFINN trả lời "câu này khen hay chê". LIWC trả lời "văn bản này dùng nhiều từ thuộc ngăn nào".

Ví dụ nhỏ, các ngăn và từ đều là giả định. Câu: "We love our room" (4 từ).

| Bước | Việc | Kết quả |
|---|---|---|
| 1 | Bỏ từng từ vào ngăn | "we" vào ngăn chúng tôi, "love" vào ngăn cảm xúc, "our" vào ngăn chúng tôi, "room" không vào ngăn nào |
| 2 | Đếm ngăn chúng tôi | 2 từ, bằng 2 chia 4, tức 50% |
| 3 | Đếm ngăn cảm xúc | 1 từ, bằng 1 chia 4, tức 25% |
| 4 | Đọc kết quả | 50% và 25% cộng lại là 75%, không phải 100%. Điều này bình thường vì "room" không thuộc ngăn nào |

Nếu có từ nằm trong hai ngăn cùng lúc thì cộng các ngăn lại còn có thể vượt 100%.

**LIWC dùng để làm gì**

Để so sánh hai nhóm văn bản xem cách dùng từ khác nhau ở ngăn nào. Ví dụ so phản hồi của nhân viên trước và sau khi được đào tạo, xem tỷ lệ ở ngăn cảm xúc có thay đổi không. Ví dụ có số liệu ở mục 5, tình huống 5.

**Vì sao không được kết luận về tâm lý**

Con số 50% chỉ nói rằng 50% số từ nằm trong ngăn đó. Nó không nói người viết đang nghĩ hay cảm thấy gì. Câu "You said you were angry; we understand." có từ "angry", nhưng nhân viên không giận, chỉ nhắc lại lời khách. Máy vẫn bỏ từ "angry" vào ngăn giận. Ví dụ đầy đủ ở mục 5, tình huống 3.

**Các bước máy làm theo cơ chế truyền thống**


1. Máy đối chiếu từng từ của văn bản với từ điển.
2. Với mỗi nhóm, máy tính xem bao nhiêu phần trăm số từ thuộc nhóm đó.
3. Kết quả là một bảng, mỗi nhóm một cột.

Cần nhớ ba điều.

1. Đây là bảng thành phần ngôn ngữ, không phải bảng tâm lý của người viết. Từ bảng đến kết luận về tâm lý là một bước suy luận cần bằng chứng.
2. Một từ có thể thuộc nhiều nhóm, nên các tỷ lệ cộng lại không bằng 100%.
3. Không phải cột nào cũng là phần trăm. Các chỉ số tổng hợp có cách tính riêng, phải tra định nghĩa từng cột [S1] [S3].

<a id="muc-2"></a>

**2. So sánh với năm cách trước**

Thông tin về năm cách trước (SentiWordNet, VADER, TextBlob, AFINN, NRC EmoLex) lấy từ các file trước.

| Điểm so sánh | Năm cách trước | LIWC |
|---|---|---|
| Mục đích chính | Khen chê (NRC thêm nhiều nhóm cảm xúc) | Nhiều nhóm ngôn ngữ, không chỉ cảm xúc |
| Đầu ra cho văn bản | Một điểm, hoặc bảng đếm với NRC | Bảng tỷ lệ phần trăm theo nhóm, cộng thêm một số chỉ số tổng hợp |
| Một từ thuộc mấy nhóm | Một điểm, riêng NRC thuộc nhiều nhóm | Có thể thuộc nhiều nhóm |
| Xử lý "not", "very", dấu câu | VADER có sẵn nhiều nhất | Ở cơ chế đếm truyền thống, không xử lý như một bộ chấm khen chê |
| Có tài liệu kiểm định về phép đo | Khác nhau tùy cách | Có manual về phát triển và tâm trắc [S1] [S4] |
| Điều kiện dùng | Mỗi nơi có điều kiện riêng | Cần có quyền dùng công cụ và từ điển, xem điều kiện trên trang LIWC |
| Tiếng Việt | Không dùng trực tiếp | Cần một từ điển tiếng Việt phù hợp và bằng chứng kiểm định |
| Dùng làm bộ phân loại khen chê | Có | Không phải mục đích chính |

Điểm khác biệt lớn nhất: LIWC trả lời "văn bản có bao nhiêu phần trăm từ thuộc từng nhóm ngôn ngữ", dùng cho giả thuyết về phong cách, tương tác, tâm lý. Nó không sinh ra để thay một bộ phân loại khen chê.

Để thấy điểm chung của cả sáu cách, thử với câu: "You said you were angry; we understand."

| Cách | Điều gì xảy ra |
|---|---|
| SentiWordNet, VADER, TextBlob, AFINN | Có từ "angry", điểm nghiêng về chê |
| NRC EmoLex | Đếm "angry" vào nhóm anger và negative |
| LIWC | Tính tỷ lệ phần trăm từ thuộc nhóm từ giận (nếu phiên bản có nhóm này) |

Cả sáu cách đều không biết ai đang giận. Câu này là lời nhân viên trích lại lời khách, nhưng máy vẫn thấy từ giận.

<a id="muc-3"></a>

**3. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Đặt giả thuyết và chọn nhóm biến, trước khi xem kết quả | Người |
| Chọn phiên bản, ngôn ngữ và từ điển | Người |
| Chuẩn bị dữ liệu mỗi dòng một văn bản, có mã số | Người |
| Quyết định đơn vị phân tích: từng phản hồi hay gộp | Người |
| Đối chiếu từ với từ điển | Máy (LIWC) |
| Tính tỷ lệ theo nhóm và các chỉ số tổng hợp | Máy (LIWC) |
| Xuất bảng kết quả giữ mã số | Máy (LIWC) |
| Đọc các từ đóng góp nhiều nhất để kiểm tra | Người |
| Phân tích thống kê và kết luận | Người |

Các nhóm trong từ điển do nhà phát triển LIWC xây sẵn, người dùng không gán lại. Nếu người dùng tự xây từ điển riêng để chạy bằng ứng dụng LIWC, thì từ điển đó không tự có chất lượng đã kiểm định của LIWC, và người xây phải tự chứng minh [S1].

<a id="muc-4"></a>

**4. Cách tính tỷ lệ**

Với một nhóm k và một văn bản d:

```text
P = 100 nhân (n chia cho N)
```

| Ký hiệu | Ý nghĩa |
|---|---|
| k | Một nhóm từ trong từ điển |
| n | Số vị trí trong văn bản khớp với nhóm k |
| N | Tổng số từ của văn bản theo cách đếm của công cụ |
| P | Tỷ lệ phần trăm từ thuộc nhóm k |

Ví dụ cách đọc: P bằng 4.2 nghĩa là 4.2% số từ của văn bản thuộc nhóm đó theo quy tắc của phiên bản. Không có nghĩa người viết mang 4.2% một đặc điểm tâm lý.

Hai lưu ý.

1. Vì một từ có thể thuộc nhiều nhóm, tổng các P của các nhóm có thể lớn hơn 100.
2. Công thức trên chỉ đúng cho tỷ lệ nhóm theo cơ chế đếm truyền thống. Các chỉ số tổng hợp có cách tính khác, không tự tính lại được bằng cách lấy tỷ lệ một nhóm [S1] [S2].

Văn bản rỗng thì N bằng 0, cần ghi là thiếu dữ liệu.

Áp dụng công thức vào câu "We love our room" ở mục 1, với ngăn "chúng tôi" (nhóm giả định):

| Ký hiệu | Giá trị trong ví dụ | Giải thích |
|---|---|---|
| k | Ngăn chúng tôi | Nhóm đang tính |
| n | 2 | Hai từ "we" và "our" thuộc ngăn này |
| N | 4 | Câu có 4 từ |
| P | 100 nhân (2 chia 4), bằng 50 | 50% số từ thuộc ngăn chúng tôi |

Làm tương tự với ngăn cảm xúc: n bằng 1, N bằng 4, P bằng 25. Mỗi ngăn tính riêng, nên hai kết quả không cần cộng thành 100.

<a id="muc-5"></a>

**5. Ví dụ bốn tình huống**

Các nhóm và số liệu dưới đây là **giả định** để tính cho dễ, không phải nội dung từ điển LIWC. Từ điển LIWC thuộc về nhà phát triển và không được phân phối trong tài liệu này.

**Tình huống 1: một từ thuộc hai nhóm.** Văn bản 100 từ. Nhóm A có 3 vị trí khớp, nhóm B có 2 vị trí khớp, trong đó có 1 vị trí thuộc cả hai nhóm.

| Bước | Việc | Kết quả |
|---|---|---|
| 1 | Tính tỷ lệ nhóm A | 3 chia 100 nhân 100, bằng 3% |
| 2 | Tính tỷ lệ nhóm B | 2 chia 100 nhân 100, bằng 2% |
| 3 | Đếm số vị trí khác nhau thuộc ít nhất một nhóm | 3 cộng 2 trừ 1, bằng 4 |
| 4 | Đọc kết quả | Hai tỷ lệ cộng lại 5%, nhưng chỉ có 4% số từ thực sự thuộc nhóm nào đó |

Nếu cộng tỷ lệ các nhóm mà nghĩ rằng có 5% số từ được phân loại thì sai.

**Tình huống 2: trung bình từng phản hồi và gộp tất cả từ.** Hai phản hồi cùng thuộc nhóm A.

| Phản hồi | Số từ | Số vị trí khớp nhóm A | Tỷ lệ |
|---|---|---|---|
| 1 | 20 | 2 | 10% |
| 2 | 200 | 4 | 2% |

| Cách tính | Phép tính | Kết quả |
|---|---|---|
| Trung bình tỷ lệ từng phản hồi | (10 cộng 2) chia 2 | 6% |
| Gộp tất cả từ | (2 cộng 4) chia (20 cộng 200) | khoảng 2.73% |

Hai cách cho hai con số khác nhau vì phản hồi ngắn được tính nặng ngang phản hồi dài ở cách đầu. Phải chọn một cách theo câu hỏi nghiên cứu và nói rõ trong bài.

**Tình huống 3: từ giận trong lời người khác.** Câu của nhân viên: "You said you were angry; we understand." (7 từ). Giả định nhóm B là nhóm từ giận.

| Bước | Việc | Kết quả |
|---|---|---|
| 1 | Máy khớp từ | angry thuộc nhóm B |
| 2 | Tính tỷ lệ nhóm B | 1 chia 7 nhân 100, khoảng 14.3% |
| 3 | Đọc kết quả | Tỷ lệ cao, nhưng nhân viên không giận, chỉ trích lại lời khách |

Nếu dùng tỷ lệ này để kết luận về trạng thái tâm lý của nhân viên thì đã bỏ qua việc trích dẫn và vai giao tiếp. Có thể giữ kết luận ở cấp ngôn ngữ ("câu có nhiều từ nhóm B") hoặc xây thêm cách xác định ai là người trải nghiệm cảm xúc.

**Tình huống 4: văn bản ngắn.**

| Văn bản | Số từ | Số vị trí khớp nhóm A | Tỷ lệ |
|---|---|---|---|
| Rất ngắn | 5 | 1 | 20% |
| Dài | 100 | 1 | 1% |

Cùng một vị trí khớp mà tỷ lệ khác nhau rất xa. Văn bản ngắn làm tỷ lệ dao động mạnh, nên cần xem độ dài và cân nhắc cách tổng hợp trước khi so sánh.

**Tình huống 5: so sánh phản hồi trước và sau khi đào tạo.** Giả định ngăn cảm xúc, ba phản hồi mỗi giai đoạn. Các con số là giả định.

Bước 1. Máy tính tỷ lệ ngăn cảm xúc cho từng phản hồi.

| Giai đoạn | Phản hồi 1 | Phản hồi 2 | Phản hồi 3 |
|---|---|---|---|
| Trước đào tạo | 2% | 4% | 3% |
| Sau đào tạo | 5% | 6% | 4% |

Bước 2. Người lấy trung bình từng giai đoạn.

| Giai đoạn | Phép tính | Trung bình |
|---|---|---|
| Trước đào tạo | (2 cộng 4 cộng 3) chia 3 | 3% |
| Sau đào tạo | (5 cộng 6 cộng 4) chia 3 | 5% |

Bước 3. Người đọc kết quả ở ba mức.

| Mức | Nói được không | Lý do |
|---|---|---|
| "Tỷ lệ từ cảm xúc tăng 2 điểm phần trăm" | Được, nếu phép đếm đáng tin | Đây chỉ là kết quả của phép đếm |
| "Nhân viên đồng cảm hơn" | Chưa | Cần bằng chứng rằng ngăn cảm xúc đo đúng sự đồng cảm, ví dụ hai người đánh giá độc lập |
| "Đào tạo làm tăng sự đồng cảm" | Chưa | Còn có thể do thay đổi người quản lý, loại khách hay mẫu viết sẵn. Chỉ ba phản hồi mỗi giai đoạn cũng quá ít |

Ví dụ này cho thấy máy chỉ cho phần đầu tiên, hai phần sau là việc của người và của thiết kế nghiên cứu.

<a id="muc-6"></a>

**6. Lỗi hay gặp**

1. Đọc tên nhóm thành kết luận. Ví dụ thấy cột tên "Authenticity" rồi kết luận người này nói thật. Tên cột chỉ là tên của chỉ số, cần đọc định nghĩa trong tài liệu của phiên bản.
2. Coi mọi cột là phần trăm hoặc buộc các tỷ lệ cộng thành 100%.
3. Trộn dữ liệu chạy bằng các phiên bản khác nhau chỉ vì cùng tên cột. Khi nâng phiên bản, chạy lại cùng một mẫu để kiểm tra có so sánh được không trước khi gộp.
4. Chạy hàng loạt các nhóm rồi chỉ báo nhóm có kết quả nổi bật. Cần chọn nhóm theo giả thuyết trước khi xem dữ liệu.
5. Cho rằng từ điển tự xây chạy bằng LIWC thì có chất lượng như LIWC.
6. Gộp nhiều phản hồi vào một dòng trong khi đơn vị phân tích dự kiến là từng phản hồi.
7. Dùng kết quả để đánh giá nhân viên hoặc chẩn đoán tâm lý. Chỉ số có tên gợi đặc điểm tâm lý không đủ làm căn cứ khen thưởng, kỷ luật hay kết luận về sức khỏe tâm thần của người viết.
8. Đưa tệp PDF vào và cho rằng công cụ hiểu hình ảnh. Công cụ chỉ phân tích chữ đã được trích ra, nên cần kiểm tra lỗi trích xuất chữ trước.
9. Phân phối từ điển lên GitHub. Từ điển có thể chịu điều kiện cấp phép. Nên đăng hướng dẫn, cấu hình, mã phân tích và dữ liệu giả định, và ghi rõ người tái lập cần có quyền dùng những gì [S2].

<a id="muc-7"></a>

**7. Làm sao biết kết quả đáng tin**

Với LIWC, câu hỏi kiểm chứng phụ thuộc vào điều muốn kết luận. Bảng dưới chia theo ba mức.

| Muốn kết luận | Bằng chứng cần có | Cách làm |
|---|---|---|
| "Ngôn ngữ của nhóm A dùng nhiều từ nhóm X hơn" | Phép đếm đáng tin trên dữ liệu này | Người đọc các từ đóng góp nhiều nhất xem có đúng nghĩa trong khách sạn không. Kiểm tra tỷ lệ khớp và độ dài. Thử bỏ văn bản rất ngắn hoặc đổi cách tổng hợp, và nêu lý do |
| "Nhân viên đồng cảm hơn" (một cấu trúc tâm lý) | Bằng chứng rằng nhóm từ đo đúng cấu trúc đó | Hai người đánh giá độc lập mức đồng cảm trên một mẫu phản hồi, máy tính mức hai người đồng ý với nhau (hệ số kappa, càng gần 1 càng đồng ý), rồi so với tỷ lệ nhóm từ |
| "Chương trình đào tạo làm tăng sự đồng cảm" | Thiết kế cho phép nói về nguyên nhân | So sánh trước và sau chưa đủ, vì còn thay đổi người quản lý, thành phần khách và mẫu trả lời. Cần nhóm đối chứng hoặc cách nhận diện nguyên nhân phù hợp |

Có hai khái niệm giúp kiểm tra cấu trúc tâm lý.

| Khái niệm | Câu hỏi |
|---|---|
| Giá trị hội tụ | Chỉ số có đi cùng chiều với một phép đo khác của cùng một cấu trúc không |
| Giá trị phân biệt | Chỉ số có khác với các cấu trúc gần nó không |

Tên nhóm nghe hợp lý chưa thay thế được hai loại bằng chứng này. Manual của LIWC có bằng chứng về tâm trắc, nhưng đó là ở dữ liệu của nghiên cứu khác, không tự thay thế bằng chứng trong miền khách sạn [S1] [S4].

Thêm hai điều cần làm khi phân tích thống kê.

1. Dữ liệu lồng theo khách sạn hoặc nhân viên thì phải phân tích theo cách tương ứng, không coi mọi phản hồi là độc lập.
2. Nếu dùng nhiều nhóm biến, tách rõ phân tích xác nhận (giả thuyết đặt trước) với phân tích khám phá, và điều chỉnh cách kiểm định cho nhiều phép so sánh.

Nếu muốn so với năm cách trước trên bài toán khen chê, cần quy ra một nhãn khen chê từ nhóm cảm xúc của phiên bản đang dùng, rồi thử trên phần thử nghiệm. Tên nhóm cần đối chiếu với tài liệu. Cách này là đề xuất của người biên soạn, và LIWC không sinh ra để làm bộ phân loại khen chê.

<a id="muc-8"></a>

**8. Chọn cái nào trong sáu cái**

| Nhu cầu | Cách hợp hơn | Điểm yếu |
|---|---|---|
| Mốc khen chê đơn giản nhất | AFINN | Không có quy tắc nào về ngữ cảnh, tổng phụ thuộc độ dài |
| Phân biệt các nghĩa khác nhau của cùng một từ | SentiWordNet | Phải tự chọn nghĩa |
| Có chữ hoa, dấu chấm than, "very", "but", "not" | VADER | Mỗi từ chỉ có một điểm |
| Biết câu thiên về ý kiến hay sự việc | TextBlob | Chỉ ước lượng theo từ điển |
| Hồ sơ nhiều nhóm cảm xúc như giận, sợ, vui, tin tưởng | NRC EmoLex | Không có cường độ, không xử lý "not" |
| Nhiều chỉ báo ngôn ngữ cho giả thuyết về phong cách và tương tác | LIWC | Cần quyền dùng, cần thiết kế nghiên cứu chặt, tiếng Việt cần từ điển riêng |
| Đúng hơn trên đánh giá khách sạn | Chưa biết | Phải đo trên dữ liệu của nghiên cứu |

Chọn LIWC khi nhóm ngôn ngữ của nó khớp giả thuyết và có căn cứ tài liệu cho phép đo. Nếu chỉ cần nhãn khen chê cơ bản thì một trong năm cách kia có thể đủ. Quyết định không nên dựa vào tên ứng dụng nổi tiếng mà dựa vào việc biến đó có đo đúng điều cần đo không.

<a id="muc-9"></a>

**9. Cách thao tác và mã tính thử**

Thao tác trong ứng dụng, theo các bước chung. Tên lựa chọn có thể thay đổi theo phiên bản nên cần đối chiếu với hướng dẫn [S3].

1. Chuẩn bị bảng, mỗi dòng một văn bản, có cột mã số, cột khách sạn, cột thời gian và cột nội dung.
2. Nhập bảng vào ứng dụng.
3. Chọn đúng cột chứa nội dung chữ.
4. Xác nhận từ điển, nhóm biến và cách phân đoạn.
5. Chạy và xuất kết quả, giữ cột mã số để ghép lại với dữ liệu gốc.

Đoạn mã dưới đây chỉ tính thử phép toán của mục 5 với nhóm giả định. Nó không dùng và không tái tạo từ điển LIWC.

```python
# Nhóm giả định, không phải từ điển LIWC.
groups = {
    "A": {"you", "we"},
    "B": {"angry"},
}

def ty_le(text):
    tokens = text.lower().replace(";", "").replace(".", "").split()
    n = len(tokens)
    result = {}
    for name, words in groups.items():
        count = sum(w in words for w in tokens)
        result[name] = 100 * count / n
    return result

print(ty_le("You said you were angry; we understand."))
```

Hàm này in tỷ lệ phần trăm cho từng nhóm giả định. Cách tách `.split()` chỉ phù hợp tiếng Anh ngắn, không phải cách tách từ tiếng Việt. Mã chưa được chạy thử trong tài liệu này.

<a id="muc-10"></a>

**10. Tài liệu tham khảo**

- **[S1]** Boyd, R. L., Ashokkumar, A., Seraj, S., & Pennebaker, J. W. (2022). *The Development and Psychometric Properties of LIWC-22*. [Manual chính thức](https://www.liwc.app/static/documents/LIWC-22%20Manual%20-%20Development%20and%20Psychometrics.pdf).
- **[S2]** LIWC. [How It Works](https://liwc.app/help/howitworks).
- **[S3]** LIWC. [LIWC Analysis: thao tác và diễn giải](https://liwc.app/help/liwc).
- **[S4]** LIWC. [Danh mục tài liệu tâm trắc](https://liwc.app/help/psychometrics-manuals).

S1 là nền tảng cần đọc sâu theo từng biến. Phần so sánh với năm cách trước dựa trên các file trước. Các nhóm giả định, ví dụ khách sạn, kế hoạch kiểm định và các ví dụ về đào tạo nhân viên ở trên là đề xuất của người biên soạn, không phải nghiên cứu đã được manual xác nhận.
