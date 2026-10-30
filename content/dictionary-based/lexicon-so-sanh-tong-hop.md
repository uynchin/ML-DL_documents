# Bảng phân tích mười phương pháp Lexicon Based cho văn bản đánh giá khách sạn

**Mục lục**

- [1. Cách đọc tài liệu này](#muc-1)
- [2. Bảng điểm mạnh, điểm yếu và mức phù hợp](#muc-2)
- [3. Bảng kết hợp và lưu ý khi dùng](#muc-3)
- [4. Câu minh họa cho từng phương pháp](#muc-4)
- [5. Chọn cách nào cho văn bản đánh giá khách sạn](#muc-5)
- [6. Điều chưa biết và cách xử lý](#muc-6)

<a id="muc-1"></a>

**1. Cách đọc tài liệu này**

Tài liệu tổng hợp mười phương pháp đã viết: SentiWordNet, VADER, TextBlob, AFINN, Opinion Lexicon, MPQA, SentiStrength, NRC, LIWC và SenticNet. Mục tiêu là trả lời bốn câu hỏi cho từng cách: mạnh ở đâu, yếu ở đâu, hợp với mức nào của văn bản, và nên ghép với gì.

**Mức của văn bản.** Cùng một cách có thể hợp ở mức này mà không hợp ở mức khác.

| Mức | Ý nghĩa | Ví dụ |
|---|---|---|
| Từ | Xem từng từ có điểm gì | Tra "cold" trong danh sách |
| Câu | Chấm một câu | "The room was clean." |
| Bài đánh giá | Chấm cả bài gồm nhiều câu | Một đánh giá năm câu |
| Khía cạnh | Chấm riêng từng đối tượng như phòng, dịch vụ, bữa sáng | Phòng được khen, dịch vụ bị chê |
| Cả tập dữ liệu | So sánh nhóm, theo thời gian, theo khách sạn | Cảm xúc trung bình mỗi quý |

Không cách nào trong mười cách tự tách theo khía cạnh. Muốn chấm khía cạnh phải thêm một bước tách trước, ví dụ lập danh sách từ khóa cho từng khía cạnh rồi gắn từ cảm xúc với khía cạnh gần nhất.

**Quy ước.**

| Nhãn | Ý nghĩa |
|---|---|
| Phù hợp | Có thể dùng làm cách chính ở mức đó |
| Dùng được nếu chỉnh | Cần thêm quy tắc hoặc ghép với cách khác |
| Dùng bổ trợ | Không dùng một mình, nhưng thêm thông tin có ích |
| Không nên | Không hợp mục đích đó |

Các nhãn này là đánh giá của người biên soạn dựa trên cách mỗi phương pháp hoạt động. Chưa có phép đo nào trên dữ liệu đánh giá khách sạn thật, nên chưa thể nói cách nào đúng nhất. Điều đó chỉ biết sau khi chạy các cách trên cùng phần chấm điểm cuối và so macro-F1, như các file trước đã mô tả.

**Về các câu minh họa ở mục 4.** Kết quả ghi là kết quả dự kiến, suy ra từ quy tắc của từng phương pháp trong các file trước. Các con số là giả định. Không có câu nào đã được chạy thử, nên cần chạy mã để xác nhận trước khi viết vào bài.

<a id="muc-2"></a>

**2. Bảng điểm mạnh, điểm yếu và mức phù hợp**

| Phương pháp | Điểm mạnh | Điểm yếu | Mức phù hợp nhất |
|---|---|---|---|
| SentiWordNet | Điểm theo từng nghĩa, nên phân biệt được "cold" lạnh nhạt với "cold" lạnh nhiệt độ. Xem được điểm từng nghĩa | Phải chọn nghĩa, chọn sai thì điểm sai. Không xử lý "not". Người phải tự gộp điểm | Từ và câu |
| VADER | Có sẵn quy tắc cho "not", "very", chữ hoa, dấu chấm than, "but". Chạy nhanh. Cho sẵn điểm compound | Một điểm cho cả câu. Mỗi từ chỉ một điểm, không tách nghĩa. Dễ sai với mỉa mai | Câu và bài ngắn |
| TextBlob | Có thêm subjectivity cho biết câu thiên về ý kiến. Dễ dùng. Xem được các từ được chấm | Lấy trung bình nên che mất câu vừa khen vừa chê. Cách xử lý "not" và "very" phải đối chiếu mã nguồn | Câu |
| AFINN | Dễ kiểm tra nhất, cộng bằng tay được. Làm mốc thấp nhất | Điểm tổng tăng theo độ dài. Không có quy tắc nào về ngữ cảnh | Câu ngắn, làm mốc |
| Opinion Lexicon | Đơn giản nhất, hai danh sách khen và chê. Gắn với nghiên cứu về đánh giá sản phẩm | Không có mức mạnh yếu. Không xử lý "not". Không có nhãn trung tính | Câu, làm mốc |
| MPQA | Phân biệt câu nêu ý kiến với câu nêu sự việc. Có nhãn mạnh yếu và nhãn neutral | Không có điểm số. Không xử lý "not". Từ neutral đứng một mình là bằng chứng yếu. Giấy phép GPL | Câu, để lọc |
| SentiStrength | Hai điểm khen và chê riêng nên không bù trừ. Có quy tắc cho biểu tượng cảm xúc, chữ kéo dài. Có tùy chọn explain | Chỉ lấy từ mạnh nhất nên mất thông tin về số lượng. Cần Java và giấy phép khi dùng thương mại. Chưa có bản tiếng Việt sẵn | Câu và bài ngắn |
| NRC | Cho mười nhóm cùng lúc, gồm tám cảm xúc cộng khen và chê. Dễ đếm | Chỉ 0 hoặc 1, không có mức mạnh yếu. Không xử lý "not". Không sinh ra để phân loại khen chê | Cả tập dữ liệu, so nhóm |
| LIWC | Nhiều nhóm ngôn ngữ, không chỉ cảm xúc. Có tài liệu kiểm định | Không phải bộ phân loại khen chê. Từ điển thương mại. Tiếng Việt cần từ điển riêng | Bài dài, cả tập dữ liệu |
| SenticNet | Tra cả cụm nhiều từ chứ không chỉ từng từ | Thêm bước trích khái niệm nên có thêm chỗ sai. Nhiều phiên bản. Khó kiểm tra bằng tay | Câu có cụm |

Bảng dưới đánh giá mức phù hợp cho từng mức của văn bản.

| Phương pháp | Từ | Câu | Bài đánh giá | Khía cạnh | Cả tập dữ liệu |
|---|---|---|---|---|---|
| SentiWordNet | Phù hợp | Dùng được nếu chỉnh | Không nên, nếu chưa có cách gộp | Dùng được nếu chỉnh | Không nên |
| VADER | Không nên | Phù hợp | Dùng được nếu chỉnh | Dùng được nếu chỉnh | Dùng được nếu chỉnh |
| TextBlob | Không nên | Phù hợp | Không nên, vì trung bình che mất ý | Dùng được nếu chỉnh | Dùng được nếu chỉnh |
| AFINN | Phù hợp | Phù hợp, làm mốc | Dùng được nếu chuẩn hóa | Dùng được nếu chỉnh | Dùng được nếu dùng trung bình |
| Opinion Lexicon | Phù hợp | Phù hợp, làm mốc | Dùng được nếu chuẩn hóa | Dùng được nếu chỉnh | Dùng được nếu dùng tỷ lệ |
| MPQA | Phù hợp | Dùng bổ trợ, để lọc ý kiến | Dùng bổ trợ | Không nên một mình | Dùng bổ trợ |
| SentiStrength | Không nên | Phù hợp | Phù hợp với bài ngắn | Dùng được nếu chỉnh | Dùng được nếu chỉnh |
| NRC | Dùng bổ trợ | Không nên một mình | Dùng bổ trợ | Dùng bổ trợ | Phù hợp, để so nhóm |
| LIWC | Không nên | Không nên | Dùng bổ trợ | Không nên | Phù hợp, để so nhóm |
| SenticNet | Không nên | Dùng được nếu chỉnh | Dùng được nếu chỉnh | Dùng được nếu chỉnh | Dùng được nếu chỉnh |

<a id="muc-3"></a>

**3. Bảng kết hợp và lưu ý khi dùng**

Cột thứ hai cho biết điểm yếu của từng cách nên bù bằng gì. Cột thứ ba cho biết điểm mạnh của từng cách nên dùng để làm gì khi ghép với cách khác. Cột thứ tư là điều cần chú ý khi chạy.

| Phương pháp | Điểm yếu thì ghép với | Điểm mạnh thì dùng để | Khi dùng chú ý |
|---|---|---|---|
| SentiWordNet | Một bước chọn nghĩa. Quy tắc "not" do người viết. Hoặc ghép với VADER hay SentiStrength để lấy các quy tắc ngữ cảnh | Kiểm tra các từ nhiều nghĩa mà cách khác chấm một điểm duy nhất | Ghi phiên bản NLTK. Đọc thử khoảng 50 câu có từ nhiều nghĩa để đếm máy chọn đúng nghĩa bao nhiêu lần |
| VADER | Tách khía cạnh để biết khen gì chê gì. Người đọc riêng các câu mỉa mai. Ghép SentiStrength nếu cần biết câu vừa khen vừa chê | Làm bộ chấm chính cho câu tiếng Anh | Giữ nguyên dấu chấm than, chữ hoa, "not". Chỉ đổi ngưỡng 0.05 trên phần thử nghiệm. Không dùng nhãn VADER làm đáp án |
| TextBlob | Tách thành câu rồi mới chấm, không chấm cả bài. Đối chiếu với VADER | Dùng subjectivity để đoán câu ý kiến, rồi đối chiếu với MPQA | Xem danh sách các từ được chấm để biết vì sao ra điểm. Ghi phiên bản, vì cách xử lý "not" có thể thay đổi |
| AFINN | Quy tắc "not". Chuẩn hóa độ dài | Làm mốc thấp nhất để các cách phức tạp hơn đặt lên so | Ghi cách chuẩn hóa. Không ghi điểm 0 khi không khớp từ nào |
| Opinion Lexicon | Lấy mức mạnh yếu từ AFINN. Thêm quy tắc "not". Chuẩn hóa chữ viết sai trước khi khớp | Làm mốc đếm đơn giản, gần miền đánh giá sản phẩm | Giữ riêng P và N để phân biệt hỗn hợp với thiếu dữ liệu. Ghi nguồn tệp |
| MPQA | VADER hoặc SentiStrength để có điểm số. Quy tắc "not" | Lọc câu nêu ý kiến trước khi chấm khen chê | Chọn quy tắc nhận câu ý kiến trên phần thử nghiệm. Đọc điều kiện GPL trước khi đăng tệp |
| SentiStrength | Tách khía cạnh. Ghép với Opinion Lexicon hoặc AFINN để lấy số lượng từ khen chê mà SentiStrength bỏ | Phát hiện câu hoặc bài vừa khen vừa chê. Dùng explain để kiểm tra từng câu | Giữ biểu tượng cảm xúc và chữ kéo dài. Ghi bản và kiểu kết quả. Xem giấy phép |
| NRC | VADER hoặc SentiStrength để có mức mạnh yếu và xử lý "not" | Gắn tên cảm xúc cụ thể cho các câu đã biết là chê hoặc khen | Không dùng nhãn NRC làm đáp án. Kiểm tra bản dịch nếu dùng cho tiếng Việt |
| LIWC | Một bộ phân loại khen chê riêng, vì LIWC không thay được | So sánh phong cách viết giữa các nhóm, ví dụ đánh giá ít sao với nhiều sao | Cần quyền dùng. Không đưa nội dung từ điển lên GitHub. Chỉ dùng nhóm đã có bằng chứng kiểm định |
| SenticNet | Quy tắc "not" riêng. Kiểm tra bước trích khái niệm bằng tay | Nhận cụm nhiều từ mà cách tra từng từ bỏ lỡ | Ghi phiên bản và cách gọi. Kiểm tra cả bước trích khái niệm lẫn bước tra |

**Ba kiểu ghép hay dùng nhất.**

| Kiểu ghép | Cách làm | Khi nào |
|---|---|---|
| Lọc rồi chấm | MPQA lọc câu nêu ý kiến, rồi VADER hoặc SentiStrength chấm các câu đã lọc | Đánh giá có nhiều câu mô tả khách sạn xen với câu nhận xét |
| Chấm rồi giải thích | VADER hoặc SentiStrength xác định khen chê, rồi NRC gắn tên cảm xúc | Muốn biết đánh giá chê là vì giận, ghê tởm hay buồn |
| Mốc và máy chính | Opinion Lexicon hoặc AFINN làm mốc, VADER hoặc SentiStrength làm máy chính | Muốn biết các quy tắc phức tạp hơn có thật sự giúp không |

<a id="muc-4"></a>

**4. Câu minh họa cho từng phương pháp**

Với mỗi phương pháp, bảng cho biết câu, điều dự kiến xảy ra, và bài học rút ra. Mọi kết quả là dự kiến, cần chạy mã để xác nhận.

**4.1. SentiWordNet**

| Câu | Điều dự kiến xảy ra | Bài học |
|---|---|---|
| The receptionist was cold. | Nếu chọn nghĩa lạnh nhạt thì chê rõ. Nếu chọn nhầm nghĩa nhiệt độ thì gần trung tính | Chọn nghĩa quyết định kết quả |
| The soup was cold. | Chọn nghĩa nhiệt độ thì chê nhẹ. Đây là nghĩa phù hợp với câu | Cùng một chữ, hai câu, hai nghĩa khác nhau |
| The room was light and airy. | Từ "light" có nhiều nghĩa, như nhẹ cân và nhiều ánh sáng. Máy có thể chọn nghĩa không phải nghĩa trong câu | Từ nhiều nghĩa là nơi SentiWordNet dễ sai nhất |
| The breakfast was light. | Nghĩa ít calo hoặc nhẹ. Điểm khen chê của nghĩa này khác với nghĩa nhiều ánh sáng | Cần đọc thử các câu có từ nhiều nghĩa |
| The pool area was cool. | Có thể hiểu là mát về nhiệt độ hoặc là có phong cách. Hai nghĩa có điểm khác nhau | Nghĩa nào đúng phụ thuộc cả câu |
| The bed was not comfortable. | Chỉ tra "comfortable" thì ra khen, sai. Cần quy tắc đảo dấu do người viết | Không có quy tắc "not" sẵn |
| The room was clean, but the service was slow. | Nếu cộng đơn giản thì khen chê bù nhau, ra gần trung tính | Cần chấm riêng từng đối tượng |
| The hotel has 100 rooms. | Không có từ cảm xúc, ra thiếu dữ liệu hoặc gần 0 | Điểm 0 chưa chắc là trung tính |

**4.2. VADER**

| Câu | Điều dự kiến xảy ra | Bài học |
|---|---|---|
| The room was good. | Khen vừa phải | Mốc để so với các câu dưới |
| The room was VERY good!!! | Khen mạnh hơn câu trên, nhờ chữ hoa và dấu chấm than | Xóa dấu chấm than và chữ hoa trước khi chạy là mất thông tin |
| The room was not good. | Đảo sang chê và yếu đi một chút | Quy tắc "not" có sẵn |
| The room was good, but the service was terrible. | Nghiêng về chê, vì vế sau "but" được tính nặng hơn | Hợp lý, nhưng vẫn là một điểm cho cả câu |
| The room was terrible, but the staff were wonderful. | Nghiêng về khen, vì vế sau "but" được tính nặng hơn | Thứ tự hai vế làm đổi kết quả. Cần đọc xem có đúng ý khách không |
| Yeah, great, another hour waiting at check-in. | Dễ chấm khen vì có "great", trong khi ý là chê | Mỉa mai là điểm yếu chung của mọi cách tra từ điển |
| Not bad at all. | Dự kiến nghiêng khen nhẹ nhờ đảo dấu của "not". Cần chạy mã để xem mức | Cụm phủ định của từ chê |
| The bed was hard as a rock. | Có thể không thấy từ cảm xúc nào, nên ra gần 0 | Lời phàn nàn không dùng từ cảm xúc rõ thì bị bỏ sót |
| Phòng sạch sẽ, nhân viên thân thiện. | Không đáng tin, vì từ điển là tiếng Anh | Tiếng Việt cần dịch hoặc từ điển riêng, và phải kiểm tra |

**4.3. TextBlob**

| Câu | Điều dự kiến xảy ra | Bài học |
|---|---|---|
| The hotel has 100 rooms. | polarity gần 0, subjectivity thấp | Subjectivity thấp báo hiệu câu nêu sự việc |
| I think the room is wonderful. | polarity dương, subjectivity cao | Subjectivity cao báo hiệu câu nêu ý kiến |
| The room is wonderful but the staff are rude. | Trung bình hai ý, polarity gần 0 | Trung bình che mất cả khen lẫn chê |
| The room is wonderful. The staff are rude. | Chấm từng câu cho hai kết quả riêng, một khen một chê | Tách câu trước khi chấm giữ được thông tin |
| The bed was not comfortable. | Có xử lý phủ định, cần chạy mã xem kết quả | Đối chiếu mã nguồn theo phiên bản |
| Absolutely wonderful stay! | polarity dương mạnh, subjectivity cao | Một câu ý kiến rõ |
| We stayed three nights. | Không có từ chấm được, polarity gần 0, subjectivity thấp | Câu sự việc |
| Terrible, awful, dirty, noisy. | Điểm lấy trung bình, nên có thể không âm hơn một từ chê đơn lẻ | Trung bình làm mất số lượng lời chê |

**4.4. AFINN**

Từ điển giả định dùng cho các tính toán dưới đây: friendly 2, clean 2, good 3, terrible -4, awful -3, rude -2.

| Câu | Điều dự kiến xảy ra | Bài học |
|---|---|---|
| Good room. | Tổng bằng 3 | Mốc đơn giản |
| The room was good and the bed was good and the breakfast was good. | Tổng bằng 9, gấp ba câu trên chỉ vì dài hơn | Tổng tăng theo độ dài, không phải mức hài lòng |
| The staff were friendly, the room was clean, the breakfast was good, but the bed was terrible. | 2 cộng 2 cộng 3 trừ 4 bằng 3, ra khen | Ba lời khen nhẹ lấn át một lời chê nặng. Khách có thể không hài lòng |
| Terrible, awful, rude staff. | Tổng bằng -4 trừ 3 trừ 2, tức -9 | Nhiều từ chê làm tổng lớn, nhưng nhiều từ chưa có nghĩa là chê nặng hơn |
| The bed was not good. | Khớp "good", ra khen, sai | Không có quy tắc "not" |
| We have no complaints. | Nếu danh sách có từ "complaint" thì ra chê, sai | Từ phủ định "no" không được xử lý |
| Một tháng có 500 đánh giá, tháng khác có 100 đánh giá | Tổng tháng đầu lớn hơn chỉ vì nhiều đánh giá hơn | Dùng trung bình khi so sánh theo thời gian |
| The hotel is near the airport. | Không khớp mục nào, thiếu dữ liệu | Ghi riêng, không ghi là 0 |

**4.5. Opinion Lexicon**

Danh sách giả định: khen gồm friendly, clean, good, spacious, comfortable. Chê gồm terrible, slow, rude, dirty, noisy.

| Câu | Điều dự kiến xảy ra | Bài học |
|---|---|---|
| Spacious and comfortable room. | P = 2, N = 0, khen | Đếm đơn giản |
| The service was terrible. | P = 0, N = 1, chê | Mốc đơn giản |
| The service was slow. | P = 0, N = 1, chê | Cùng kết quả với câu trên dù "terrible" nặng hơn "slow" |
| Great location but dirty bathroom. | P = 1, N = 1, hỗn hợp | Giữ riêng P và N giúp thấy câu có cả khen lẫn chê |
| The hotel is located near the airport. | P = 0, N = 0, thiếu dữ liệu | Không ghi là trung tính |
| The bed was not comfortable. | Khớp "comfortable", ra khen, sai | Cần quy tắc "not" |
| The room was comfy and clene. | Từ viết sai chính tả như "clene" có thể không khớp | Chuẩn hóa chính tả hoặc kiểm tra danh sách có chứa lỗi chính tả phổ biến không |
| The room was good and the bed was good and the breakfast was good. | Hiệu số cao vì dài, tỷ lệ bằng 1 | Hiệu số, tỷ lệ và mật độ cho ba kết luận khác nhau |

**4.6. MPQA**

Nhãn giả định: wonderful là strongsubj positive, friendly và clean là weaksubj positive, terrible là strongsubj negative, think và feel là weaksubj neutral.

| Câu | Điều dự kiến xảy ra | Bài học |
|---|---|---|
| The hotel has 120 rooms and a pool. | Không thấy manh mối, câu có vẻ nêu sự việc | Lọc được câu mô tả |
| Check-in starts at 2 pm. | Không thấy manh mối, câu nêu sự việc | Câu thông tin không cần chấm khen chê |
| I think the staff were friendly. | Thấy "think" và "friendly", câu nêu ý kiến, nghiêng khen nhẹ | Từ neutral giúp nhận ra câu ý kiến |
| Absolutely wonderful stay! | Thấy "wonderful" mạnh, câu ý kiến, khen mạnh | Nhãn strong báo hiệu ý kiến mạnh |
| The room was clean, but the service was terrible. | Đếm thì hỗn hợp, có trọng số thì nghiêng chê | Cách gộp phải chọn trước |
| I think we arrived at 3 pm. | Chỉ thấy "think", dễ bị nhận nhầm là câu ý kiến | Từ neutral đứng một mình là bằng chứng yếu |
| The bathroom smelled of smoke. | Có thể không có manh mối nào, bị coi là sự việc | Lời chê không dùng từ cảm xúc bị bỏ sót |
| The bed was not good. | Khớp "good" ra khen, sai | MPQA không xử lý "not" |

**4.7. SentiStrength**

Các con số hai điểm dưới đây là giả định, cho biết cách đọc. Điểm thật cần chạy và xem tùy chọn explain.

| Câu | Điều dự kiến xảy ra | Bài học |
|---|---|---|
| The staff were friendly and the breakfast was delicious. | P khoảng 3, N bằng -1 | Chỉ lấy từ mạnh nhất, không cộng |
| The room was clean, but the service was terrible. | P khoảng 2, N khoảng -4 | Giữ cả khen lẫn chê, thấy chê mạnh hơn |
| Great location, awful noise at night. | P khen khá mạnh, N chê khá mạnh | Hai điểm cùng lớn báo hiệu cảm xúc hai chiều |
| The service was very slow. | N yếu hơn "slow" nhưng mạnh hơn khi không có "very" | Từ tăng mức độ có quy tắc |
| Loooved the breakfast. | Chữ kéo dài làm điểm khen mạnh hơn | Giữ nguyên chữ kéo dài |
| The room was clean :) | Biểu tượng cảm xúc cộng thêm vào điểm khen | Có danh sách biểu tượng cảm xúc riêng |
| The service was terrible :( | Biểu tượng cảm xúc xác nhận ý chê | Xóa biểu tượng trước khi chạy là mất thông tin |
| The bed was not good. | Phủ định đổi chiều, ra chê | Có sẵn quy tắc |
| The room was ok. | P và N đều gần 1, gần trung tính | Điểm 1 là không có, không phải khen yếu |
| Ten câu khen nhẹ và một câu chê nặng | N vẫn là chê nặng, P vẫn là khen nhẹ | Mạnh nhất quyết định, số lượng không có mặt |
| Phòng rất sạch, dịch vụ tệ | Không có bản tiếng Việt sẵn trên trang chính thức | Cần chuyển hoặc kiểm chứng riêng |

**4.8. NRC**

Nhóm: tám cảm xúc gồm anger, anticipation, disgust, fear, joy, sadness, surprise, trust, cộng positive và negative. Một từ có thể thuộc nhiều nhóm. Kết quả dưới đây chỉ đúng nếu từ có trong từ điển.

| Câu | Điều dự kiến xảy ra | Bài học |
|---|---|---|
| I was furious about the rude staff. | Đếm vào nhóm anger và negative nếu các từ có trong từ điển | Cho biết loại cảm xúc, không chỉ chê |
| The dirty bathroom was disgusting. | Có thể vào nhóm disgust và negative | Phân biệt chê vì ghê tởm với chê vì giận |
| We were very happy during our stay. | Có thể vào joy và positive | Cảm xúc vui |
| We were not happy during our stay. | Vẫn đếm "happy" vào joy và positive | Chữ "not" không đổi gì, yếu nhất trong mười cách |
| We are looking forward to coming back. | Có thể vào anticipation nếu từ có trong từ điển | Cảm xúc mong chờ |
| The staff were helpful and trustworthy. | Có thể vào trust và positive | Nhóm trust có ích với đánh giá dịch vụ |
| The room was clean. | Có thể chỉ vào positive, không vào nhóm cảm xúc nào | Không phải câu nào cũng có cảm xúc cụ thể |
| So-so. | Có thể không khớp từ nào | Thiếu dữ liệu |
| The service was terrible. The service was awful. | Đếm hai lần vào negative | NRC không có mức mạnh yếu, chỉ có đếm |

**4.9. LIWC**

LIWC đếm tỷ lệ phần trăm từ thuộc từng nhóm ngôn ngữ. Các nhóm dưới đây chỉ đúng nếu phiên bản đang dùng có nhóm đó. Từ điển là thương mại nên không đưa nội dung vào.

| Câu | Điều dự kiến xảy ra | Bài học |
|---|---|---|
| I think I should have booked another hotel. | Tăng tỷ lệ đại từ ngôi thứ nhất và nhóm quá trình nhận thức | Hợp với giả thuyết về phong cách viết |
| They did not care about us at all. | Tăng tỷ lệ nhóm phủ định và đại từ | Đếm từ, không hiểu câu |
| Maybe the room was fine, I guess. | Có thể tăng nhóm do dự, tức tentative | Cho biết mức do dự, không phải khen chê |
| We arrived at 5 pm and left the next morning. | Tăng tỷ lệ từ chỉ thời gian | Mô tả nội dung kể lại |
| You said you were angry, we understand. | Đếm từ giận ở nhóm cảm xúc âm nếu có | Không biết ai đang giận, đây là lời nhân viên trích lại lời khách |
| Giả thuyết: đánh giá ít sao dùng nhiều đại từ ngôi thứ nhất hơn đánh giá nhiều sao | Tính tỷ lệ nhóm đại từ cho từng nhóm đánh giá rồi so sánh | Là kiểu câu hỏi mà LIWC trả lời được. Đây là giả thuyết minh họa, chưa được kiểm chứng |
| The room was clean. | Có thể tăng nhóm cảm xúc dương rất nhẹ vì chỉ có một từ | Không dùng LIWC làm bộ chấm khen chê cho một câu |
| Phòng sạch, nhân viên thân thiện | Cần từ điển tiếng Việt đã được kiểm định | Không dùng bản tiếng Anh cho tiếng Việt |

**4.10. SenticNet**

Kết quả dưới đây chỉ đúng nếu cụm có trong tài nguyên và bước trích khái niệm tìm đúng cụm.

| Câu | Điều dự kiến xảy ra | Bài học |
|---|---|---|
| It was a waste of money. | Tra được cụm "waste of money", ra chê | Từng từ riêng có thể không chê, cụm thì chê |
| We will never come back. | Cụm có thể được nhận ra, dự kiến chê | Ý "không quay lại" khó bắt bằng cách tra từng từ |
| The room was out of order. | Cụm có thể được nhận ra là chê | Cụm làm nghĩa khác với từng từ |
| Breakfast was a joke. | Có thể bị hiểu theo nghĩa đen, bỏ lỡ ý chê | Thành ngữ không phải lúc nào cũng được tra đúng |
| It was not a waste of money. | Nếu chỉ tra mà không xử lý "not" thì vẫn ra chê, sai | Phải tự xử lý hoặc dùng bộ suy luận |
| Paid a fortune for a tiny room. | Có thể nhận ra các khái niệm như "tiny room", dự kiến chê | Câu không có từ cảm xúc rõ vẫn có thể chấm được |
| Check-in took forever. | Có thể nhận ra khái niệm chờ lâu | Cần kiểm tra bước trích khái niệm |
| Very close to the beach. | Cụm "close to the beach" dự kiến khen | Cụm vị trí có thể là điểm mạnh |

<a id="muc-5"></a>

**5. Chọn cách nào cho văn bản đánh giá khách sạn**

**Theo mục tiêu nghiên cứu.**

| Mục tiêu | Cách chính | Ghép thêm | Lý do |
|---|---|---|---|
| Phân loại khen, chê, trung tính cho đánh giá tiếng Anh | VADER hoặc SentiStrength | Opinion Lexicon và AFINN làm mốc | Hai cách này có sẵn quy tắc ngữ cảnh. Mốc cho biết quy tắc có giúp không |
| Phát hiện đánh giá vừa khen vừa chê | SentiStrength | Chấm theo khía cạnh | Hai điểm riêng không bù trừ |
| Bỏ câu mô tả, chỉ chấm câu nhận xét | MPQA lọc, rồi VADER hoặc SentiStrength chấm | TextBlob subjectivity để đối chiếu | Câu sự việc kéo điểm về gần 0 |
| Biết khách giận, ghê tởm hay buồn | NRC | VADER hoặc SentiStrength để biết mức | NRC cho nhóm cảm xúc, không cho mức mạnh |
| So sánh phong cách viết giữa các nhóm đánh giá | LIWC | Không cần ghép | Là việc LIWC sinh ra để làm, nếu có quyền dùng |
| Cụm thành ngữ quan trọng với dữ liệu | SenticNet | VADER | Nhận cụm nhiều từ |
| Từ nhiều nghĩa gây lỗi đáng kể | SentiWordNet | Một bước chọn nghĩa | Điểm theo nghĩa |
| Đánh giá tiếng Việt | Chưa có cách nào dùng thẳng | Từ điển tiếng Việt đã kiểm định, hoặc dịch rồi kiểm tra cả bước dịch | Mười cách đều chủ yếu cho tiếng Anh |

**Theo kiểu câu.**

| Kiểu câu | Cách hợp hơn | Lý do |
|---|---|---|
| Khen hoặc chê đơn giản, không phủ định | Mọi cách đều làm được | Không có khó khăn đặc biệt |
| Có "not", "never" | VADER, SentiStrength | Có sẵn quy tắc phủ định |
| Vừa khen vừa chê | SentiStrength, hoặc chấm theo khía cạnh | Giữ cả hai chiều |
| Mỉa mai | Không cách nào đáng tin | Người đọc riêng loại câu này |
| Cụm hoặc thành ngữ | SenticNet, SentiStrength | Nhận cụm |
| Từ nhiều nghĩa | SentiWordNet | Điểm theo nghĩa |
| Câu sự việc | MPQA | Nhận ra không có ý kiến |
| Biểu tượng cảm xúc, chữ kéo dài | SentiStrength | Có quy tắc riêng |
| Lời chê không dùng từ cảm xúc | Không cách nào bắt được | Cần phương pháp khác, ví dụ học máy hoặc khai thác chủ đề |

**Quy trình đề xuất.** Đây là đề xuất của người biên soạn, cần thử và điều chỉnh theo dữ liệu.

| Bước | Việc | Ai làm |
|---|---|---|
| 1 | Giữ nguyên văn bản gốc, không xóa dấu câu, chữ hoa, biểu tượng cảm xúc | Người |
| 2 | Tách thành câu, nếu cần tách theo khía cạnh bằng danh sách từ khóa | Máy theo quy tắc người đặt |
| 3 | Lọc câu nêu ý kiến bằng MPQA, nếu dữ liệu có nhiều câu mô tả | Máy, người chọn quy tắc |
| 4 | Chấm bằng hai cách trở lên, ví dụ VADER và SentiStrength, kèm Opinion Lexicon làm mốc | Máy |
| 5 | Nếu cần, gắn nhóm cảm xúc bằng NRC cho các câu đã biết khen chê | Máy |
| 6 | Hai người gán nhãn khoảng 100 câu, tính kappa, chia thử nghiệm và chấm điểm cuối theo khách sạn | Người |
| 7 | Chạy mọi cách trên phần chấm điểm cuối, so macro-F1, bảng nhầm lẫn, coverage | Máy tính, người đọc các câu sai |
| 8 | Chọn cách, ghi tên, phiên bản, tệp dữ liệu, quy tắc, ngưỡng vào bài | Người |

<a id="muc-6"></a>

**6. Điều chưa biết và cách xử lý**

| Điều chưa biết | Cách xử lý |
|---|---|
| Cách nào đúng nhất trên đánh giá khách sạn của nghiên cứu | Đo macro-F1 trên cùng phần chấm điểm cuối. Không dùng kết quả trong các bài gốc làm kết quả của nghiên cứu, vì các bài gốc đo trên dữ liệu khác |
| Các nhãn "phù hợp", "dùng được nếu chỉnh" ở bảng mục 2 có đúng với dữ liệu không | Đây là đánh giá của người biên soạn dựa trên cách hoạt động, cần kiểm chứng bằng thí nghiệm |
| Kết quả thật của từng câu minh họa ở mục 4 | Chạy mã trong từng file riêng, rồi thay các điểm giả định bằng điểm thật |
| Tiếng Việt | Chưa có cách nào dùng thẳng. Nếu dịch sang tiếng Anh rồi chấm, kiểm tra riêng bước dịch bằng đáp án |
| Lời chê không dùng từ cảm xúc, mỉa mai | Cần người đọc riêng hoặc phương pháp khác |

Tài liệu này dựa trên các file riêng của từng phương pháp, ở đó có nguồn tham khảo và các chỗ chưa đối chiếu. Mọi câu minh họa, từ điển giả định, kết quả dự kiến và quy trình đề xuất là của người biên soạn, chưa phải kết quả của một thí nghiệm thực tế.
