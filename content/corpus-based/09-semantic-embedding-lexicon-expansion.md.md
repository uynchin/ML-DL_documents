# Semantic: mở rộng từ điển cảm xúc bằng embedding (bản đơn giản)

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

Hình dung một tấm bản đồ, trong đó các từ được đặt gần nhau nếu cách dùng giống nhau. Đã biết vài từ nằm ở khu khen và vài từ nằm ở khu chê. Từ mới nằm gần khu nào thì có thể đoán là khen hoặc chê theo khu đó [S1] [S2].

Cách làm này giúp từ điển cũ có thêm từ mới, ví dụ "đáng tiền" hay "ổn áp" mà từ điển dựng sẵn chưa có.

Ba khái niệm cần nhớ:

| Khái niệm | Hiểu đơn giản |
|---|---|
| Embedding (vector) | Dãy số đại diện cho một từ, giống tọa độ của từ trên bản đồ |
| Độ gần (cosine) | Hai "mũi tên" chỉ cùng hướng đến mức nào: 1 là cùng hướng, 0 là vuông góc, -1 là ngược hướng |
| Từ mẫu (seed) | Vài từ khen và vài từ chê đã biết chắc, ví dụ "hài_lòng" và "thất_vọng" |

Cách chấm gồm ba việc:

1. Đo độ gần của từ cần chấm với các từ mẫu khen, lấy trung bình.
2. Đo độ gần của từ cần chấm với các từ mẫu chê, lấy trung bình.
3. Lấy số thứ nhất trừ số thứ hai. Dương là nghiêng về khen, âm là nghiêng về chê, gần 0 là chưa rõ.

Có một điều cần nhớ ngay từ đầu: gần nhau chưa chắc cùng cảm xúc. "Sạch" và "bẩn" cùng nói về vệ sinh nên có thể nằm sát nhau trên bản đồ, dù một từ khen và một từ chê. Vì vậy phải so với cả hai phía từ mẫu, và phải đọc lại kết quả.

Các vector trong bài là số tự đặt để tính tay, không phải vector thật từ Word2Vec, fastText hay SentProp.

<a id="muc-2"></a>

**2. Ai làm gì**

| Việc | Ai làm |
|---|---|
| Quyết định chấm từ đơn, cụm từ hay nghĩa theo ngữ cảnh | Người |
| Chọn nguồn vector (học từ đánh giá khách sạn hoặc dùng vector có sẵn) | Người |
| Chọn từ mẫu khen và chê, kiểm tra nghĩa trong câu thật | Người |
| Tách từ nhất quán (tiếng Việt: "hài_lòng" là một đơn vị, không phải "hài" và "lòng") | Máy, người kiểm tra thử |
| Tra vector, tính độ gần, tính điểm | Máy |
| Đặt ngưỡng để nhận một từ vào từ điển, và khoảng điểm coi là chưa rõ | Người |
| Đọc các từ mới được đề xuất, loại từ sai | Người |
| So từ điển gốc với từ điển mở rộng trên dữ liệu kiểm tra | Máy tính, người đọc các câu sai |
| Kết luận trong bài viết | Người |

Nhớ gọn: máy tra và tính, người chọn từ mẫu, đặt ngưỡng và duyệt từ mới.

<a id="muc-3"></a>

**3. Một từ, làm từ đầu đến cuối**

Mục tiêu: tìm điểm cho từ "đáng_tiền". Từ mẫu khen là "hài_lòng", từ mẫu chê là "thất_vọng". Để dễ tính, mỗi từ chỉ có hai số (tọa độ tự đặt):

| Từ | Vector |
|---|---|
| hài_lòng (mẫu khen) | (1, 0) |
| thất_vọng (mẫu chê) | (-1, 0) |
| đáng_tiền (cần chấm) | (0.8, 0.6) |

Bước 1. Máy kiểm tra cả ba từ đều có trong bảng vector. Từ nào thiếu thì ghi "chưa có vector", không cho điểm.

Bước 2. Máy đo độ gần của "đáng_tiền" với từ mẫu khen. Ba vector này đều có độ dài bằng 1 nên độ gần chính là phép nhân từng cặp số rồi cộng lại: 0.8 × 1 + 0.6 × 0 = **0.8**.

Bước 3. Máy đo độ gần với từ mẫu chê: 0.8 × (-1) + 0.6 × 0 = **-0.8**.

Bước 4. Máy lấy độ gần với phía khen trừ độ gần với phía chê: 0.8 trừ (-0.8) = **1.6**.

| Phép tính | Điểm của "đáng_tiền" |
|---|---:|
| 0.8 trừ (-0.8) | 1.6 |

Bước 5. Máy so với ngưỡng do người đặt. Giả sử ngưỡng nhận từ khen là điểm từ 0.5 trở lên, thì 1.6 vượt ngưỡng. Từ này được đề xuất vào từ điển với nhãn nghiêng về khen, chờ người đọc lại trong câu thật.

Điểm này nằm trong khoảng từ -2 đến 2, không phải xác suất và không phải mức hài lòng của khách. Một từ nằm vuông góc với cả hai phía, như "mơ_hồ" (0, 1), có điểm 0.

Đoạn mã dưới đây kiểm tra lại phép tính, chỉ cần Python, không cần cài thêm:

```python
from math import sqrt

# Vector tự đặt, không phải kết quả học từ dữ liệu thật.
vectors = {
    "hai_long": (1.0, 0.0),
    "that_vong": (-1.0, 0.0),
    "dang_tien": (0.8, 0.6),
}

def cosine(x, y):
    nx = sqrt(sum(v * v for v in x))
    ny = sqrt(sum(v * v for v in y))
    return sum(a * b for a, b in zip(x, y)) / (nx * ny)

def cham(tu, khen, che):
    if any(w not in vectors for w in [tu, *khen, *che]):
        return None  # chưa có vector, không phải trung tính
    gan_khen = sum(cosine(vectors[tu], vectors[s]) for s in khen) / len(khen)
    gan_che = sum(cosine(vectors[tu], vectors[s]) for s in che) / len(che)
    return gan_khen - gan_che

print(round(cham("dang_tien", ["hai_long"], ["that_vong"]), 3))
# 1.6
```

Mã này chỉ tính điểm cho một từ. Mã không học vector, không dựng đồ thị và không đo chất lượng từ điển.

<a id="muc-4"></a>

**4. Ba lỗi hay gặp và cách sửa**

**Lỗi 1: gần nhau nhưng khác chiều cảm xúc**

| Từ | Nằm gần | Vấn đề |
|---|---|---|
| sạch | bẩn | Cùng nói về vệ sinh nên vector giống nhau, nhưng một khen một chê |
| đắt | rẻ | Cùng nói về giá |

Nếu lấy mọi từ ở gần "sạch" rồi gán là khen, "bẩn" sẽ bị gán sai.

Cách sửa: luôn so với cả hai phía từ mẫu rồi lấy hiệu, không chỉ nhìn một phía. Người đọc lại các từ được đề xuất trong câu thật. Nếu hai phía có độ gần gần như bằng nhau, giữ trạng thái "chưa rõ", không ép phải gán nhãn.

**Lỗi 2: từ không có trong bảng vector hoặc tách từ không khớp**

| Tình huống | Hậu quả |
|---|---|
| Bảng vector lưu "hài_lòng" nhưng dữ liệu bị tách thành "hài" và "lòng" | Không tra được, hoặc tra nhầm hai từ khác |
| Từ hiếm không có trong bảng vector (gọi là OOV) | Không có điểm |
| Dùng vector kiểu subword, từ lạ vẫn có vector | Có số, nhưng chưa chắc đúng |

Cách sửa: dùng đúng một cách tách từ cho cả bảng vector lẫn dữ liệu cần chấm. Từ không có vector thì ghi riêng một trạng thái, không gọi là trung tính. Từ có vector nhờ cách ghép chữ cái vẫn phải được người đọc kiểm tra, vì có vector chỉ là điều kiện kỹ thuật, không chứng minh từ đã được hiểu đúng.

**Lỗi 3: mở rộng nhiều vòng làm lỗi lan rộng**

| Cách làm | Hậu quả |
|---|---|
| Từ mới được nhận thì đưa luôn làm từ mẫu cho vòng sau | Một từ nhận sai ở vòng đầu kéo thêm nhiều từ sai ở các vòng sau, nghĩa là lệch dần khỏi mục tiêu |
| Luôn giữ từ mẫu gốc, giới hạn số vòng, người duyệt giữa các vòng | Lỗi dễ phát hiện và dừng kịp |

Cách sửa: giữ nguyên từ mẫu gốc, ít vòng mở rộng, và duyệt thủ công một mẫu ở mỗi vòng. Số từ trong từ điển tăng lên chưa phải là thành công, cần xem các từ thêm vào có đúng không.

<a id="muc-5"></a>

**5. Làm sao biết kết quả đáng tin**

Cần kiểm tra hai chỗ: từ mới thêm vào có đúng không, và từ điển mở rộng có giúp chấm văn bản tốt hơn từ điển gốc không.

Bước 1. Chia dữ liệu trước khi làm.

| Phần | Dùng để làm gì |
|---|---|
| Phần xây | Học vector (nếu tự học), chọn từ mẫu, tính điểm |
| Phần thử nghiệm | Thử đổi từ mẫu, ngưỡng, số láng giềng. Được sửa thoải mái |
| Phần chấm điểm cuối | Chỉ dùng một lần ở cuối, không sửa gì sau khi đã thấy kết quả |

Nên chia theo khách sạn: câu của cùng một khách sạn chỉ nằm ở một phần. Từ mẫu không được lẫn vào phần dùng để chấm từ mới, vì khi đó kết quả đẹp mà không chứng minh được khả năng đoán từ chưa biết.

Bước 2. Kiểm tra từ mới. Người lấy khoảng 20 từ có điểm cao nhất, 20 từ có điểm gần 0, đọc trong câu thật rồi ghi: đúng chiều, sai chiều, hay chỉ là từ chỉ chủ đề. Với nhóm gần 0, ghi thêm lý do: từ đa nghĩa, gần cả hai phía, xa cả hai phía hay vector kém. Mẫu nhỏ này dùng để thử nghiệm ban đầu, không đại diện cho toàn bộ từ vựng. Thử thêm việc đổi từ mẫu và đổi nguồn vector xem điểm có thay đổi nhiều không.

Bước 3. Kiểm tra việc chấm văn bản.

| Việc | Cách làm |
|---|---|
| Tạo đáp án | Khoảng 100 câu từ nhiều khách sạn, hai người gán nhãn độc lập, đo mức đồng ý (kappa), bàn lại chỗ khác nhau |
| So hai từ điển | Chấm cùng tập câu bằng từ điển gốc và từ điển mở rộng, cùng cách chấm, cùng ngưỡng |
| Báo ba con số | Độ đúng trung bình của ba nhãn (macro-F1), bảng nhầm lẫn, tỷ lệ câu tra được ít nhất một từ (coverage) |

Nếu tỷ lệ tra được tăng mà macro-F1 giảm, từ mới đang gây thêm lỗi, cần xem lại ngưỡng. Không được xóa từ gây lỗi sau khi xem tập chấm điểm cuối rồi báo lại tập đó như bằng chứng độc lập.

Một lưu ý khi viết kết luận: "từ X gần từ mẫu khen hơn từ mẫu chê" là kết quả hình học theo mô hình vector. "Từ X luôn là lời khen" hay "nhóm khách này hài lòng hơn" là kết luận vượt quá dữ liệu. Khi so điểm giữa hai thời kỳ, không so thẳng vector của hai mô hình học riêng, vì hai bản đồ khác nhau chưa chắc đặt cùng một gốc.

<a id="muc-6"></a>

**6. Khi nào nên dùng**

| Nên dùng | Không nên dùng |
|---|---|
| Từ điển có sẵn thiếu nhiều từ theo cách khách hay nói | Từ mẫu quá ít, nhiều nghĩa hoặc gần như không có trong dữ liệu |
| Có vector phù hợp với miền khách sạn, hoặc có đủ đánh giá để tự học vector | Nguồn vector khác hẳn miền, chưa có thử nghiệm nhỏ |
| Muốn mỗi từ mới truy được từ mẫu nào và từ láng giềng nào đã gợi ra nó | Ứng viên chủ yếu là tên người, địa danh |
| Cần tăng độ phủ nhưng vẫn có người duyệt | Cần hiểu phủ định, mỉa mai, từng đối tượng trong câu (cần thêm quy tắc hoặc mô hình khác) |

Với tiếng Việt, có thể làm trực tiếp trên vector và từ mẫu tiếng Việt, không cần dịch, nhưng phải ghép cụm nhiều âm tiết nhất quán trước khi tra.

Nên bắt đầu từ cách đơn giản nhất ở bài này (so độ gần với từ mẫu), vì dễ kiểm tra. Cách lan truyền trên đồ thị chỉ đáng thử khi lợi ích đo được và phù hợp chi phí. Trước khi mở rộng hàng nghìn từ, kiểm tra thử một mẫu nhỏ.

Khi viết vào bài, cần ghi rõ: mô hình vector, nguồn dữ liệu và số chiều, danh sách từ mẫu và lý do chọn, công thức tính điểm, ngưỡng, số vòng mở rộng, số từ thêm vào, cách xử lý từ không có vector, số câu ở phần chấm điểm cuối, ba con số ở phần 5 và hệ số kappa. Nếu chỉ dùng cách so độ gần với từ mẫu thì gọi đúng là baseline, không nhận là đã làm lại SentProp. Khi đưa lên GitHub, mã, cấu hình và ví dụ tự soạn chia sẻ được, còn dữ liệu và vector phải theo quyền sử dụng của nguồn. Mã SocialSent [S4] là tài liệu tham khảo, cần kiểm tra môi trường chạy trước khi dùng.

<a id="muc-7"></a>

**7. Tài liệu tham khảo**

- **[S1]** Mikolov, T., Chen, K., Corrado, G., & Dean, J. (2013). *Efficient Estimation of Word Representations in Vector Space*. [arXiv:1301.3781](https://arxiv.org/abs/1301.3781).
- **[S2]** Hamilton, W. L., Clark, K., Leskovec, J., & Jurafsky, D. (2016). *Inducing Domain-Specific Sentiment Lexicons from Unlabeled Corpora*. EMNLP, 595–605. [ACL Anthology](https://aclanthology.org/D16-1057/).
- **[S3]** Bojanowski, P., Grave, E., Joulin, A., & Mikolov, T. (2017). *Enriching Word Vectors with Subword Information*. TACL, 5, 135–146. [ACL Anthology](https://aclanthology.org/Q17-1010/).
- **[S4]** Hamilton và cộng sự. [SocialSent: mã và hướng dẫn của dự án](https://github.com/williamleif/socialsent).

S1 và S3 giới thiệu cách biểu diễn từ bằng vector, S2 là công trình suy ra từ điển cảm xúc theo miền, S4 là mã nghiên cứu tham khảo. Các ví dụ khách sạn, vector số tự đặt, cách phân công máy và người, các bước kiểm tra và đoạn mã ở trên là đề xuất của người biên soạn, không phải nghiên cứu của các tác giả này.
