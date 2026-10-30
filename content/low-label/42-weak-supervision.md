# Weak Supervision — Học từ các nguồn nhãn yếu

> **Nhánh:** MACHINE LEARNING → Ít nhãn. **Nguồn:** dòng 43 bảng Excel. **Đối chiếu:** 29/10/2026. Ví dụ chạy labeling functions và majority vote có abstain bằng Python/NumPy; không huấn luyện Snorkel label model. Dữ liệu và tình huống khách sạn đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bối cảnh | Phân tích cảm xúc review khách sạn tiếng Việt |
| Nguồn tín hiệu | Phân biệt gold labels, pseudo/weak labels và target tự tạo trong bài |
| Đánh giá | Cần protocol và dữ liệu độc lập phù hợp, không có benchmark review thật ở ví dụ |

## 1. Định nghĩa học thuật

Weak supervision sử dụng tín hiệu giám sát không hoàn hảo như quy tắc, metadata, distant supervision hoặc nguồn gán nhãn có sai số để hỗ trợ huấn luyện. Trong lập trình nhãn, nhiều labeling functions có thể gán nhãn hoặc abstain, sau đó được tổng hợp thành nhãn/soft labels. Đây là họ rộng hơn một thư viện hay một phép bỏ phiếu.

## 2. Giải thích đơn giản

Thay vì đọc từng review, bạn viết quy tắc như thấy “tệ” thì gợi ý tiêu cực, thấy “sạch” thì gợi ý tích cực. Các quy tắc có thể không biết, đồng ý hoặc mâu thuẫn. Chúng giúp mở rộng tín hiệu nhưng không biến gợi ý thành sự thật. “Không sạch” là ví dụ quy tắc đơn giản có thể xung đột.

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu khách sạn, có thể tạo tập huấn luyện sơ bộ khi gold labels tốn công. Cần kiểm tra model cuối trên gold data độc lập và so với baseline phù hợp. Không dùng quy tắc tạo nhãn rồi chỉ đánh giá bằng chính nhãn đó: mô hình có thể chỉ học lại quy tắc, chưa chứng minh hiểu cảm xúc.

## 4. Điều kiện và giả định sử dụng

Cần nguồn tín hiệu có cơ sở, mapping lớp thống nhất, khả năng abstain và dữ liệu audit/evaluation đáng tin. Nhiều nguồn tương quan không tương đương nhiều bằng chứng độc lập. Nếu mọi LF sao chép cùng một từ khóa, số phiếu lớn không phản ánh độ tin cậy cao hơn. Metadata có thể lệch so với mục tiêu văn bản.

## 5. Cơ chế hoạt động bên trong

Chạy các LF trên dữ liệu train để tạo ma trận nhãn, phân tích coverage/overlap/conflict, chọn cách tổng hợp. Có thể dùng majority vote hoặc label model dưới giả định cụ thể để ước lượng độ chính xác/phụ thuộc. Sau đó huấn luyện discriminative model từ nhãn cứng/mềm, kiểm tra bằng gold. Ví dụ chỉ thực thi LF+vote, không tái tạo toàn Snorkel [S1].

## 6. Quy trình áp dụng trong nghiên cứu

1. Chốt bộ nhãn và gold development/test độc lập.
2. Viết LF chỉ từ thông tin được phép và dữ liệu phát triển.
3. Chạy trên train, phân tích coverage, conflict và tương quan.
4. Audit LF bằng nhãn thật nếu có; chọn aggregator đúng giả định.
5. Huấn luyện classifier và kiểm tra theo lớp/miền.
6. Khóa quy tắc/model trước test.

Mọi lần sửa LF sau xem test đều dùng thông tin test; phải tránh hoặc có tập đánh giá mới phù hợp.

## 7. Ví dụ minh họa xuyên suốt

Ba LF tự tạo: gặp “sạch” gợi ý tích cực; gặp “tệ/bẩn” gợi ý tiêu cực; gặp “không sạch” gợi ý tiêu cực. Review “phòng không sạch” tạo phiếu 1 và 0, còn “bình thường” có thể không LF nào bỏ phiếu. Đây là các trường hợp cần abstain/giải quyết xung đột, không phải thêm một lớp sentiment -1.

## 8. Kết quả và cách diễn giải

Coverage của các LF là tỷ lệ hàng có ít nhất một nguồn bỏ phiếu; tỷ lệ aggregator trả nhãn có thể thấp hơn vì tie. Conflict là có các nhãn không abstain khác nhau. Các chỉ số này không đo accuracy; một bộ quy tắc có coverage 100% vẫn có thể sai toàn bộ. Cần gold audit để đọc chất lượng.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là tận dụng tri thức chuyên môn và nguồn dữ liệu hiện có, mở rộng labeling nhanh hơn từng hàng trong một số bối cảnh. Hạn chế là noise có cấu trúc, dependency, coverage lệch lớp và rủi ro đánh giá vòng tròn. Chi phí viết/bảo trì LF và audit cũng là chi phí, không gọi là nhãn hoàn toàn miễn phí.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| Rule-based inference | Dùng luật trực tiếp dự đoán, không nhất thiết tạo dữ liệu train cho model khác |
| Self-training | Nguồn pseudo-label là classifier hiện tại |
| Active learning | Hỏi nhãn thật cho mẫu chọn |
| Gold annotation | Nhãn được gán/kiểm tra theo protocol chuẩn |
| Distant supervision | Một kiểu lấy nhãn gián tiếp, có thể thuộc weak supervision |

## 11. Sai lầm thường gặp và cách khắc phục

Không gọi abstain=-1 là tiêu cực khi tiêu cực mã 0. Không coi LF đồng ý là gold. Không tính accuracy model với nhãn yếu rồi dùng đó làm chất lượng thật. Không đưa điểm sao vào input khi nhãn yếu được định nghĩa từ chính điểm sao rồi diễn giải mô hình phân tích sentiment văn bản độc lập.

## 12. Tài liệu tham khảo

- [S1] Ratner, A., Bach, S. H., Ehrenberg, H., Fries, J., Wu, S., & Ré, C. *Snorkel: Rapid Training Data Creation with Weak Supervision*. [Preprint 2017](https://arxiv.org/abs/1711.10160); bài PVLDB có [DOI](https://doi.org/10.14778/3157794.3157797).
- [S2] Stanford DAWN. [Trang công trình Snorkel](https://dawn.cs.stanford.edu/publications/snorkel/snorkel-rapid-training-data-creation-weak-supervision).

Mốc/triển khai Snorkel không định nghĩa toàn bộ weak supervision. Ví dụ dưới không chạy package Snorkel hoặc label model xác suất.

## 13. Vị trí trong toàn bộ nghiên cứu

Weak supervision thường ở khâu tạo tín hiệu huấn luyện trước classifier. Nhãn yếu không thay gold evaluation hoặc phép đo nghiên cứu đã xác thực. Nếu dùng sentiment dự đoán làm biến cho phân tích rating/hành vi, cần báo nguồn nhãn và kiểm tra sai số để tránh lập luận vòng tròn.

## 14. Kiến thức nền cần biết trước

Cần hiểu labeling function, abstain, label matrix, class mapping, coverage và nguồn sai số phụ thuộc. Biết từ khóa/metadata có thể không tương ứng nhãn toàn văn. Với tiếng Việt, phủ định, mỉa mai và câu hỗn hợp cần codebook; việc LF không quyết định không đồng nghĩa review trung tính.

## 15. Nền tảng lý thuyết và logic hình thành

Ý tưởng là chuyển tri thức/nguồn tín hiệu thành giám sát có thể tổng hợp và học, thay vì đòi nhãn sạch cho mọi hàng. Label model cần giả định nhận diện và chất lượng nguồn; quan sát đồng ý/xung đột không luôn đủ biết nguồn nào đúng. Không có bảo đảm majority vote hoặc label model luôn thắng từng LF.

## 16. Các thành phần cấu tạo

Thành phần gồm nguồn dữ liệu, LF, ma trận λ, bộ phân tích coverage/dependency, aggregator, downstream classifier và gold validation/test. Phân biệt label model với classifier: model thứ nhất kết hợp nguồn nhãn, model thứ hai học từ feature để dự đoán. Ghi rõ bước nào được fit bằng gold nếu có.

## 17. Các loại và biến thể

Có rule/lexicon-based labeling, distant labels từ rating/knowledge base, nguồn người/LLM chưa audit và multi-source label modeling. Các loại noise khác nhau cần xử lý khác. Nếu LLM được dùng làm nguồn, cần báo prompt/model/revision, chi phí và audit, không xem đầu ra tự động là gold chỉ vì diễn giải trôi chảy.

## 18. Đầu vào và đầu ra

Đầu vào label matrix có n hàng, m LF, giá trị trong tập lớp và abstain. Đầu ra aggregator là nhãn hoặc soft labels cùng coverage. Downstream input có thể là văn bản khác cấu trúc LF. Giữ nguồn/provenance theo hàng để kiểm tra, không chỉ lưu một cột nhãn cuối không biết từ đâu.

## 19. Căn cứ lựa chọn

Đáng thử khi có tri thức/metadata hữu ích nhưng khó gán từng hàng và có khả năng audit. Chọn theo chất lượng model cuối với gold evaluation cùng chi phí tổng thể. So với một tập gold nhỏ, các LF trực tiếp và baseline khác để biết bước học thêm có đem lợi ích hay không.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu mọi nguồn đều rất lệch hoặc cùng lỗi mà không có cách xác thực, nếu mục tiêu yêu cầu nhãn từng review được người xác nhận, hoặc nếu gold test không thể tách. Không dùng weak labels giả làm dữ liệu khảo sát do khách trực tiếp trả lời.

## 21. Các quyết định quan trọng khi thực hiện

Chọn LF nào, khi abstain, xử lý conflict/tie, có model phụ thuộc nguồn không và cách huấn luyện từ soft labels. Chốt split trước viết/sửa LF. Nếu dùng rating như distant labels, nói rõ nhiệm vụ dự đoán rating hay sentiment thủ công, và tránh dùng cùng biến để tự chứng minh nhãn đúng.

## 22. Giải thích công thức và ký hiệu

LF j cho $\lambda_j(x)\in\{-1,0,1\}$, trong đó -1 là abstain. Với hàng i, số phiếu lớp c:

$$v_{ic}=\sum_j\mathbf1[\lambda_j(x_i)=c].$$

Ví dụ majority vote chỉ trả nhãn nếu có phiếu và một lớp thắng rõ; không có phiếu hoặc tie thì abstain. Coverage LF $=n^{-1}\sum_i\mathbf1[\exists j:\lambda_j(x_i)\ne-1]$. Conflict là tỷ lệ hàng có hơn một nhãn không abstain khác nhau. Các tỷ lệ này không là xác suất đúng.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mã chỉ chạy LF và vote minh họa, không fit Snorkel hoặc classifier.

```python
import numpy as np
ABSTAIN = -1
texts = ["phòng sạch", "phòng tệ", "phòng không sạch",
         "dịch vụ bình thường", "phòng sạch nhưng tệ", "giá cao"]
def lf_clean(t):
    return 1 if "sạch" in t else ABSTAIN
def lf_bad(t):
    return 0 if ("tệ" in t or "bẩn" in t) else ABSTAIN
def lf_negation(t):
    return 0 if "không sạch" in t else ABSTAIN
LFs = [lf_clean, lf_bad, lf_negation]
L = np.array([[lf(t) for lf in LFs] for t in texts])
counts = np.column_stack([(L == c).sum(axis=1) for c in [0,1]])
has_vote = (L != ABSTAIN).any(axis=1)
conflict = (counts > 0).sum(axis=1) > 1
tie = counts[:,0] == counts[:,1]
labels = np.full(len(texts), ABSTAIN)
valid = has_vote & ~tie
labels[valid] = np.argmax(counts[valid], axis=1)
assert np.isclose(has_vote.mean(), 4/6)
assert np.isclose(conflict.mean(), 2/6)
assert np.isclose(valid.mean(), 2/6)
assert labels[2] == ABSTAIN and labels[3] == ABSTAIN
print("Ma trận LF:", L, sep="\n")
print("Nhãn vote:", labels)
print("Coverage LF / nhãn cuối:", has_vote.mean(), valid.mean())
```

Hai nguyên nhân abstain ở review “không sạch” và “bình thường” khác nhau: một bên conflict/tie, một bên không có phiếu. Không dùng cả hai để tự tạo gold class trung tính.

## 24. Tình huống sai và phản ví dụ

Nếu ba LF sao chép cùng điều kiện “sạch”, chúng có thể lấn một LF đúng về “không sạch”. Tăng số LF tương quan làm phiếu lệch chứ không tăng bằng chứng độc lập. Rating một sao cũng có thể do giá/đợi lâu trong khi câu được phân tích chỉ khen phòng; distant label không luôn đúng đơn vị.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Audit coverage theo lớp/nhóm và gold accuracy của từng LF/aggregator trên dữ liệu phù hợp. Kiểm tra dependencies, conflict và những hàng bị bỏ lại. Đánh giá classifier cuối trên gold test; làm ablation nguồn. Không suy chất lượng từ coverage hay mức đồng ý mà thiếu nhãn thật hoặc giả định nhận diện hợp lệ.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu coverage thấp, bổ sung nguồn có cơ sở thay vì nới luật để đoán mọi câu. Nếu conflict nhiều, kiểm tra codebook/đơn vị và nguồn phụ thuộc. Nếu noise cao, dùng gold audit, abstain hoặc bỏ nguồn theo development. Không sửa LF dựa test rồi giữ điểm test như độc lập.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ là độ phủ/độ đúng theo audit và chất lượng downstream trên gold evaluation. Không kết luận đã tạo nhãn thật cho toàn corpus hoặc thuật toán loại hết noise. Không suy quan hệ rating-sentiment khi sentiment được định nghĩa bằng chính rating mà không có ngoại kiểm độc lập.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu xây m LF từ [...], có abstain. Nhãn tổng hợp bằng [...], quy tắc tie [...]; coverage/conflict tính theo [...]. Model cuối đánh giá trên gold test [...].” Công bố LF/provenance, số lần chỉnh trên development, chi phí và nhãn yếu/gold tách riêng.

## 29. Khả năng tái lập và tính minh bạch

Lưu mã LF, phiên bản nguồn, label matrix, mapping, cấu hình aggregator, gold splits và downstream pipeline. Với LLM/nguồn ngoài, lưu prompt và revision theo quyền sử dụng. Mã mục 23 chỉ kiểm tra LF/vote NumPy; chưa chạy label model xác suất hoặc benchmark dữ liệu thật.

## 30. Đạo đức nghiên cứu liên quan

Không trình bày weak labels như xác nhận của khách hoặc gold annotator. Các nguồn metadata có thể tạo proxy thiên lệch nhóm; cần kiểm tra theo bối cảnh có dữ liệu hợp lệ. Bảo vệ dữ liệu cá nhân trong luật/đầu vào và có người rà soát quyết định tác động nhân viên hoặc khách.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Coverage cao có nghĩa accuracy cao không?
2. Abstain khác lớp tiêu cực thế nào?
3. Vì sao LF trùng điều kiện nguy hiểm?
4. Vì sao coverage LF và nhãn cuối khác nhau?

Gợi ý: tie làm aggregator abstain dù có phiếu. Bài mở rộng: thiết kế gold audit và đếm chi phí viết/sửa LF trong ngân sách.

## 32. Những điểm còn tranh luận

Các vấn đề mở gồm nhận diện độ tin cậy khi nguồn phụ thuộc, so sánh chi phí nhãn công bằng và kiểm toán nguồn LLM/metadata. Không có aggregator luôn loại được noise. Nghiên cứu cần công bố assumptions, provenance và gold evaluation, không chỉ nhãn cuối hoặc độ phủ đẹp.

