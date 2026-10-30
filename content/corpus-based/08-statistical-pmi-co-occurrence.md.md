# Statistical: tự tính điểm khen chê của từ bằng PMI (bản đơn giản)

**Mục lục**

- [1. Ý chính trong vài câu](#muc-1)
- [2. Ai làm gì](#muc-2)
- [3. Một từ, làm từ đầu đến cuối](#muc-3)
- [4. Ba lỗi hay gặp và cách sửa](#muc-4)
- [5. Làm sao biết kết quả đáng tin](#muc-5)
- [6. Khi nào nên dùng](#muc-6)
- [7. Tài liệu tham khảo](#muc-7)

<a id="muc-1"></a>

**1. Ý chính trong vài câu**

Có một nguyên tắc quen thuộc: nói cho biết bạn của ai, sẽ đoán được người đó thế nào. Với từ cũng vậy. Từ nào hay đứng cạnh lời khen thì có lẽ là từ khen. Từ nào hay đứng cạnh lời chê thì có lẽ là từ chê [S2] [S3].

Cách làm là lấy một đống đánh giá của khách, rồi đếm xem một từ lạ hay "chơi" với nhóm khen hay nhóm chê. Không cần ai đọc và chấm điểm từng câu.

Ba từ cần nhớ:

| Từ | Nghĩa đơn giản |
|---|---|
| Từ mẫu (seed) | Vài từ khen và vài từ chê đã biết chắc, ví dụ "excellent" (khen), "poor" (chê) |
| Từ cần chấm | Từ chưa biết khen hay chê, ví dụ "cozy" |
| PMI | Con số cho biết hai từ đi cùng nhau nhiều hơn bình thường hay ít hơn bình thường [S1] |

Cách chấm gồm hai việc:

1. Xem từ cần chấm đi cùng từ mẫu khen nhiều đến đâu, và đi cùng từ mẫu chê nhiều đến đâu.
2. Lấy mức đi cùng với từ khen trừ mức đi cùng với từ chê. Kết quả dương là nghiêng về khen, âm là nghiêng về chê, gần 0 là chưa rõ.

Hai điều cần nhớ:

- PMI chỉ đo mức "hay đứng cạnh nhau". Phải so với từ mẫu mới ra được khen hay chê.
- Từ mẫu do người chọn, nên người vẫn là người đưa ý khen chê vào hệ thống. Chọn từ mẫu cẩn thận là việc quan trọng nhất.

<a id="muc-2"></a>

**2. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Quyết định chấm theo từ, theo câu, hay theo từng khía cạnh | Người |
| Thu thập đánh giá hợp lệ, bỏ câu trùng | Người |
| Chọn từ mẫu khen và từ mẫu chê | Người |
| Tách câu thành từ (tiếng Việt cần ghép cụm như "đáng tiền") | Máy, người kiểm tra thử |
| Đếm số câu có từ, số câu có cả hai từ | Máy |
| Tính PMI và điểm của từ | Máy |
| Đặt số câu tối thiểu để một từ được chấm | Người |
| Đọc kết quả, xem từ nào máy chấm đúng, từ nào sai | Người |
| Kết luận trong bài viết | Người |

Nhớ gọn: máy đếm và tính, người chọn từ mẫu, đặt ngưỡng và kiểm tra.

<a id="muc-3"></a>

**3. Một từ, làm từ đầu đến cuối**

Muốn biết "cozy" là khen hay chê. Có 100 câu đánh giá khách sạn (số liệu tự đặt):

| Số câu | Bao nhiêu |
|---|---:|
| Có "cozy" | 30 |
| Có "excellent" (khen) | 20 |
| Có "poor" (chê) | 10 |
| Có cả "cozy" và "excellent" | 12 |
| Có cả "cozy" và "poor" | 2 |

Bước 1. Đoán xem nếu hai từ chẳng liên quan gì nhau thì sẽ có bao nhiêu câu chứa cả hai. Lấy số câu của từ thứ nhất nhân số câu của từ thứ hai, chia cho 100.

| Cặp từ | Phép tính | Dự kiến | Thực tế |
|---|---|---:|---:|
| cozy và excellent | 30 × 20 chia 100 | 6 | 12 |
| cozy và poor | 30 × 10 chia 100 | 3 | 2 |

Bước 2. So thực tế với dự kiến. "cozy" đi cùng "excellent" gấp đôi dự kiến (12 so với 6). "cozy" đi cùng "poor" ít hơn dự kiến (2 so với 3).

Bước 3. Đổi sự so sánh thành con số PMI (lấy logarit cơ số 2 của thực tế chia dự kiến). Chỉ cần nhớ cách đọc: gấp đôi thì được 1, bằng nhau thì được 0, ít hơn thì ra số âm.

| Cặp từ | PMI |
|---|---:|
| cozy và excellent | 1.000 |
| cozy và poor | -0.585 |

Bước 4. Lấy PMI với từ khen trừ PMI với từ chê: 1.000 trừ (-0.585) bằng **1.585**.

Bước 5. Kiểm tra số câu có từ. "cozy" có mặt trong 30 câu, giả sử mức tối thiểu người đặt là 20 câu, nên đủ tin. Điểm dương và khá xa 0, vậy ghi vào từ điển là nghiêng về khen.

Đoạn mã ngắn kiểm tra lại phép tính (chỉ cần Python, không cần cài thêm):

```python
from math import log2

def pmi(n, ca, cb, cab):
    if not (n and ca and cb and cab):
        return None  # thiếu bằng chứng, không phải trung tính
    return log2(n * cab / (ca * cb))

khen = pmi(100, 30, 20, 12)
che = pmi(100, 30, 10, 2)
print(round(khen, 3), round(che, 3), round(khen - che, 3))
# 1.0 -0.585 1.585
```

Lưu ý: điểm này không có thang cố định như -1 đến 1. Chỉ dùng để so sánh các từ với nhau trong cùng một bộ đánh giá và cùng bộ từ mẫu.

<a id="muc-4"></a>

**4. Ba lỗi hay gặp và cách sửa**

**Lỗi 1: từ quá hiếm**

Từ "snug" chỉ có trong 2 câu, cả hai đều có "excellent", không câu nào có "poor". Khi đó số câu có cả "snug" và "poor" bằng 0, phép tính không làm được.

| Từ | Số câu | Cùng "excellent" | Cùng "poor" | Kết quả |
|---|---:|---:|---:|---|
| cozy | 30 | 12 | 2 | Tính được, 1.585 |
| snug | 2 | 2 | 0 | Không tính được |

Sai lầm hay gặp: cho điểm 0 rồi gọi là trung tính. Thực ra "snug" chỉ là chưa đủ dữ liệu.

Cách sửa: đặt số câu tối thiểu. Từ nào ít hơn thì ghi "thiếu bằng chứng", không cho điểm. Luôn ghi số câu bên cạnh điểm, vì điểm cao từ 2 câu không đáng tin bằng điểm vừa phải từ hàng nghìn câu.

**Lỗi 2: câu vừa khen vừa chê**

Câu: "The room was excellent but the service was poor."

Câu này có cả "excellent" lẫn "poor". Nếu coi cả câu là một nhóm thì "room" và "service" đều bị tính là đi cùng cả hai bên, dù lời chê chỉ nhắm vào dịch vụ. Điểm sẽ phản ánh chủ đề nhiều hơn cảm xúc.

Cách sửa: thử tách câu thành từng ý nhỏ (cắt ở "but" và dấu phẩy) rồi đếm theo ý. Thử trên tập câu mẫu xem cách nào cho kết quả đúng hơn, vì cách này không sửa được mọi câu.

Với câu phủ định như "The room was not excellent", cần giữ chữ "not", "không" khi lọc từ, vì "excellent" ở đây không còn là lời khen.

**Lỗi 3: đếm lẫn đơn vị**

| Cách làm | Hậu quả |
|---|---|
| Số câu có cả hai từ thì đếm theo câu, nhưng số lần xuất hiện của từng từ lại đếm theo từng lần lặp | Các số không cùng loại, điểm bị lệch |
| Tất cả đều đếm theo câu: câu có từ, câu có cả hai, tổng số câu | Đúng, dùng được |

Cách sửa: chọn một đơn vị đếm (câu, ý nhỏ hoặc cụm vài từ) rồi dùng đúng đơn vị đó cho mọi con số, kể cả tổng số. Đổi đơn vị thì tính lại từ đầu.

<a id="muc-5"></a>

**5. Làm sao biết kết quả đáng tin**

Cần kiểm tra hai chỗ: từ điển có đúng không, và dùng từ điển chấm câu có đúng không.

Bước 1. Chia dữ liệu thành ba phần trước khi làm.

| Phần | Dùng để làm gì |
|---|---|
| Phần xây từ điển | Đếm và tính điểm |
| Phần thử nghiệm | Thử đổi từ mẫu, số câu tối thiểu, cách đếm. Được sửa thoải mái |
| Phần chấm điểm cuối | Chỉ dùng một lần ở cuối, không sửa gì sau khi đã thấy kết quả |

Nên chia theo khách sạn: câu của cùng một khách sạn chỉ nằm ở một phần. Chia lẫn lộn thì kết quả đẹp hơn thực tế.

Bước 2. Kiểm tra từ điển. Người đọc khoảng 50 đến 100 từ có điểm cao nhất hoặc thấp nhất, xem trong câu thật rồi ghi từ đó khen, chê hay chỉ là từ chỉ chủ đề. Càng nhiều từ người đồng ý thì từ điển càng tốt. Thử thêm việc đổi từ mẫu hoặc đổi số câu tối thiểu: nếu điểm thay đổi quá nhiều thì kết quả chưa vững.

Bước 3. Kiểm tra việc chấm câu.

| Việc | Cách làm |
|---|---|
| Tạo đáp án | Khoảng 100 câu từ nhiều khách sạn, hai người gán nhãn độc lập, đo mức đồng ý (kappa), bàn lại chỗ khác nhau |
| Chấm bằng máy | Dùng từ điển đã chốt để chấm phần chấm điểm cuối |
| Báo ba con số | Độ đúng trung bình của ba nhãn (macro-F1), bảng nhầm lẫn, tỷ lệ câu tra được ít nhất một từ (coverage) |

Một lưu ý khi viết kết luận: "từ X đi cùng từ khen nhiều hơn" là kết quả đo được. "Từ X luôn là lời khen" hay "khách sạn có nhiều từ X thì khách hài lòng hơn" là kết luận vượt quá dữ liệu.

<a id="muc-6"></a>

**6. Khi nào nên dùng**

| Nên dùng | Không nên dùng |
|---|---|
| Có nhiều đánh giá đúng loại khách sạn, để từ cần chấm xuất hiện nhiều lần | Chỉ có vài chục đánh giá (vẫn học được cách tính, nhưng chưa đủ gọi là từ điển) |
| Ít câu có nhãn nhưng chọn được từ mẫu đáng tin | Từ mẫu hầu như không xuất hiện trong dữ liệu |
| Muốn từ điển theo đúng cách khách hay nói, truy lại được từng số đếm | Văn bản nhiều mỉa mai, cần chấm chính xác từng đối tượng |
| Cần một mốc so sánh trước khi thử cách phức tạp hơn | Không có đáp án nào để kiểm tra |

Với tiếng Việt, có thể làm trực tiếp trên đánh giá tiếng Việt, không cần dịch, nhưng phải ghép cụm nhiều âm tiết (như "thân thiện", "đáng tiền") trước khi đếm. Không nên chọn cách này chỉ vì "không cần gán nhãn từng câu": chọn từ mẫu, xử lý dữ liệu và kiểm tra từ điển vẫn tốn công.

Khi viết vào bài, cần ghi rõ: nguồn đánh giá và thời gian thu thập, đơn vị đếm, danh sách từ mẫu và lý do chọn, số câu tối thiểu, cách xử lý trường hợp bằng 0, số từ trước và sau khi lọc, số câu ở phần chấm điểm cuối, ba con số ở phần 5 và hệ số kappa. Đây là cách đếm đơn giản trên dữ liệu của nghiên cứu, không phải bản làm lại nguyên vẹn của Turney, nên không chép độ chính xác của bài gốc làm kết quả của dự án.

<a id="muc-7"></a>

**7. Tài liệu tham khảo**

- **[S1]** Church, K. W., & Hanks, P. (1990). *Word Association Norms, Mutual Information, and Lexicography*. Computational Linguistics, 16(1), 22–29. [ACL Anthology](https://aclanthology.org/J90-1003/).
- **[S2]** Turney, P. (2002). *Thumbs Up or Thumbs Down? Semantic Orientation Applied to Unsupervised Classification of Reviews*. ACL, 417–424. [ACL Anthology](https://aclanthology.org/P02-1053/).
- **[S3]** Turney, P. D., & Littman, M. L. (2003). *Measuring Praise and Criticism: Inference of Semantic Orientation from Association*. ACM TOIS, 21(4), 315–346. [arXiv](https://arxiv.org/abs/cs/0309034).

S1 là nền tảng về việc đo mức đi cùng nhau của hai từ, S2 và S3 là công trình suy ra khen chê từ mức đi cùng với từ mẫu. Các ví dụ khách sạn, bảng số, cách phân công máy và người, các bước kiểm tra và đoạn mã ở trên là đề xuất của người biên soạn, không phải nghiên cứu của các tác giả này.
