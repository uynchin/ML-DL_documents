# Self-supervised Learning — Học mục tiêu tự tạo từ dữ liệu

> **Nhánh:** MACHINE LEARNING → Ít nhãn. **Nguồn:** dòng 45 bảng Excel. **Đối chiếu:** 29/10/2026. Ví dụ học dự đoán token cuối từ ngữ cảnh bằng mô hình đơn giản, không huấn luyện BERT hoặc transformer. Mã chạy sklearn 1.8.0. Dữ liệu và tình huống khách sạn đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bối cảnh | Phân tích cảm xúc review khách sạn tiếng Việt |
| Nguồn tín hiệu | Phân biệt gold labels, pseudo/weak labels và target tự tạo trong bài |
| Đánh giá | Cần protocol và dữ liệu độc lập phù hợp, không có benchmark review thật ở ví dụ |

## 1. Định nghĩa học thuật

Self-supervised learning tạo tín hiệu mục tiêu từ chính cấu trúc dữ liệu để học biểu diễn hoặc dự đoán, không cần nhãn thủ công cho pretext task. Ví dụ là dự đoán token bị che, token tiếp theo, tái tạo dữ liệu nhiễu hoặc học quan hệ giữa các view. Điều đó không có nghĩa không cần dữ liệu hay không có target.

## 2. Giải thích đơn giản

Mô hình đọc nhiều review và luyện bài như “điền phần bị che” thay vì được cho ngay nhãn tích cực/tiêu cực. Đáp án bài luyện được lấy từ câu gốc. Sau đó biểu diễn có thể dùng cho phân loại cảm xúc, nhưng vẫn cần đánh giá riêng và thường có nhãn downstream để học/kiểm tra nhiệm vụ đó.

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu khách sạn, có thể tận dụng corpus chưa nhãn để học biểu diễn hoặc thích nghi miền trước classifier. Câu hỏi là biểu diễn đó giúp nhiệm vụ cảm xúc với cùng ngân sách nhãn và compute hay không. Pretext accuracy hoặc loss thấp không tự chứng minh sentiment tốt, nhất là khi câu hỏi phụ thuộc phủ định/khía cạnh.

## 4. Điều kiện và giả định sử dụng

Cần dữ liệu phù hợp quyền sử dụng, mục tiêu tự tạo có cấu trúc hữu ích và protocol pretraining/evaluation rõ. Tokenization tiếng Việt, giới hạn độ dài và domain shift ảnh hưởng nhiều. Không cần gold sentiment cho pretraining không đồng nghĩa không cần gold test để xác thực chất lượng sentiment.

## 5. Cơ chế hoạt động bên trong

Tạo input bị che/nhiễu hoặc các view từ câu gốc, lấy target từ dữ liệu trước biến đổi, fit model bằng loss phù hợp. Sau pretraining, dùng encoder cố định hoặc fine-tune cho nhiệm vụ khác. BERT có masked language modeling và next sentence prediction trong thiết kế gốc; không phải mọi mô hình self-supervised đều dùng cả hai [S1].

## 6. Quy trình áp dụng trong nghiên cứu

1. Chốt pretext task và mục tiêu downstream.
2. Tách corpus pretraining/development/test theo quyền truy cập.
3. Chọn tokenizer, kiến trúc và cách tạo target/view.
4. Huấn luyện/chọn cấu hình trên dữ liệu phù hợp.
5. Freeze encoder hoặc fine-tune với nhãn downstream theo protocol.
6. Đánh giá bằng gold test độc lập và baseline phù hợp.

Nếu pretraining gồm test text không nhãn, công bố việc đó; không báo như chưa từng dùng test input. Với pretrained ngoài, ghi nguồn/revision và giới hạn biết về overlap.

## 7. Ví dụ minh họa xuyên suốt

Từ “phòng sạch”, tạo input “phòng [MASK]” và target token “sạch”. Đây là target từ câu gốc, không phải gold sentiment tích cực. Câu “phòng bẩn” tạo target khác nhưng cả hai là cùng kiểu pretext. Ví dụ mã dùng vài câu hai/ba token và classifier đơn giản; không mô phỏng đầy đủ contextual encoder.

## 8. Kết quả và cách diễn giải

Đầu ra pretraining có thể là phân bố token, code hoặc embedding; chúng khác xác suất sentiment. Loss giảm nghĩa mô hình làm tốt hơn mục tiêu đã chọn trên dữ liệu đó. Khả năng biểu diễn tốt cần đánh giá downstream, độ bền và chi phí. Không dùng từ “tự học” để suy mô hình tự xác thực mọi kết luận.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là khai thác lượng văn bản không có nhãn nhiệm vụ và có thể học feature tái sử dụng. Hạn chế là tốn compute/data, dễ học shortcut/ghi nhớ, mục tiêu có thể không giữ thông tin sentiment cần thiết và khó biết contamination ngoài. Nhiều dữ liệu không tự đảm bảo đúng ngôn ngữ/miền hoặc công bằng.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| Semi-supervised | Kết hợp gold task labels với U để học nhiệm vụ cụ thể |
| Self-training | Tạo nhãn giả task từ classifier |
| Weak supervision | Nhãn từ nguồn gián tiếp/quy tắc có sai số |
| Contrastive learning | Một họ objective quan hệ view/pair, có thể self-supervised hoặc supervised |
| Autoencoder | Một kiến trúc/mục tiêu tái tạo có thể thuộc cách học tự tạo target |

## 11. Sai lầm thường gặp và cách khắc phục

Không nhầm self-supervised với tự gán sentiment bằng dự đoán của chính model. Không cho target token xuất hiện nguyên trong input rồi báo học che từ. Không đánh giá pretext trên bản sao train rồi suy generalization. Không gọi mô hình pretrained là không cần nhãn trong cả chuỗi nghiên cứu nếu downstream đã fine-tune bằng nhãn.

## 12. Tài liệu tham khảo

- [S1] Devlin, J., Chang, M.-W., Lee, K., & Toutanova, K. (2019). *BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding*. [Bài gốc](https://aclanthology.org/N19-1423/).
- [S2] Gao, T., Yao, X., & Chen, D. (2021). *SimCSE*. [Bài gốc](https://aclanthology.org/2021.emnlp-main.552/).
- [S3] Goodfellow, Bengio & Courville (2016). [Autoencoders](https://www.deeplearningbook.org/contents/autoencoders.html).

Đây là các cơ chế tham khảo, không gán một năm khởi đầu duy nhất cho toàn họ self-supervised learning.

## 13. Vị trí trong toàn bộ nghiên cứu

Self-supervised learning thường ở giai đoạn representation/pretraining trước phân loại hoặc tìm kiếm. Phải phân biệt corpus pretraining với gold task data và đánh giá cuối. Nếu dùng pretrained encoder như công cụ đo, cần xem tính phù hợp miền/ngôn ngữ, không coi pretraining tự xác nhận construct trải nghiệm khách hàng.

## 14. Kiến thức nền cần biết trước

Cần hiểu input/target, tokenizer, loss, pretraining/fine-tuning, frozen encoder và contamination. Biết một nhiệm vụ không dùng sentiment labels vẫn có thể dùng hàng triệu target tokens từ dữ liệu. Với tiếng Việt, token không nhất thiết bằng từ ngữ nghĩa; segmentation ảnh hưởng cách tạo và đánh giá mục tiêu.

## 15. Nền tảng lý thuyết và logic hình thành

Ý tưởng là cấu trúc nội tại dữ liệu có thể tạo các bài học giúp mô hình giữ thông tin tái sử dụng. Nhưng mục tiêu và kiến trúc quyết định cái gì được ưu tiên. Dự đoán token có thể dựa tần suất hoặc nguồn dữ liệu thay vì sentiment. Liên hệ giữa pretext và downstream phải được kiểm chứng, không chỉ suy từ loss.

## 16. Các thành phần cấu tạo

Thành phần gồm corpus, tokenizer, phép tạo target/view, encoder/decoder hoặc prediction head, loss, optimizer và đánh giá downstream. Tách head pretext khỏi head sentiment. Nếu chỉ dùng embedding đầu ra, ghi pooling/normalization; dùng cùng backbone nhưng pooling khác có thể cho biểu diễn khác.

## 17. Các loại và biến thể

Có masked prediction, autoregressive prediction, denoising/reconstruction và contrastive objectives. Các nhiệm vụ có quyền nhìn context khác nhau; masked bidirectional và next-token causal không cùng cơ chế. Một hệ thống có thể kết hợp nhiều loss, nhưng cần công bố thay vì gọi tất cả là “BERT tự học”.

## 18. Đầu vào và đầu ra

Đầu vào có thể là văn bản không nhãn task; target được trích từ dữ liệu theo phép biến đổi. Đầu ra gồm model weights/representations hoặc phân bố pretext. Downstream classifier cần schema và nhãn riêng. Dữ liệu mới phải cùng tokenizer/encoder revision, không tự thay vocabulary rồi dùng weights cũ.

## 19. Căn cứ lựa chọn

Đáng thử khi có corpus không nhãn có giá trị, pretrained phù hợp hoặc cơ sở cho domain adaptation. Chọn bằng downstream gold evaluation, compute và độ bền. So encoder gốc với encoder được thích nghi trong cùng budget/protocol để không quy lợi ích của nhiều tài nguyên thành lợi ích thuật toán đơn lẻ.

## 20. Trường hợp không nên sử dụng

Không ưu tiên huấn luyện từ đầu khi dữ liệu/tài nguyên quá ít và baseline phù hợp đã đáp ứng. Không dùng pretext score làm thay metric sentiment. Nếu dữ liệu có thông tin cá nhân không có quyền dùng, khả năng tự tạo target không làm việc huấn luyện trở nên phù hợp tự động.

## 21. Các quyết định quan trọng khi thực hiện

Chọn corpus, mục tiêu, masking/augmentation, tokenizer, mô hình/revision, learning schedule và cách đánh giá. Chốt freeze hay fine-tune và phạm vi pretraining. Nếu tiếp tục pretrain theo miền, không dùng test text mà không công bố; ghi rõ compute và nhãn downstream để so sánh công bằng.

## 22. Giải thích công thức và ký hiệu

Với tập vị trí bị che M, target token gốc $w_i$ và input đã biến đổi $\widetilde x$:

$$L_{mask}=-\frac1{|M|}\sum_{i\in M}\log p_\theta(w_i\mid\widetilde x).$$

Nếu model cho token đúng xác suất 0.7 thì loss một vị trí là $-\log0.7\approx0.3567$; nếu 0.2 thì khoảng 1.6094. Token loss không là sentiment loss. Target không được để lộ trực tiếp trong input như một shortcut vô nghĩa cho bài che từ.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Ví dụ tự tạo target token cuối, học bằng TF-IDF+Logistic Regression để thấy nguyên tắc. Không huấn luyện BERT/transformer hoặc một sentence encoder mới.

```python
import numpy as np
from sklearn.pipeline import make_pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import log_loss
sentences = ["phòng sạch", "phòng sạch", "phòng rộng",
             "dịch vụ tốt", "dịch vụ tệ", "dịch vụ tốt"]
contexts, targets = [], []
for sentence in sentences:
    tokens = sentence.split()
    targets.append(tokens[-1])
    contexts.append(" ".join(tokens[:-1] + ["[MASK]"]))
model = make_pipeline(TfidfVectorizer(),
                      LogisticRegression(C=10, max_iter=1000, random_state=42))
model.fit(contexts, targets)
P = model.predict_proba(contexts)
loss = log_loss(targets, P, labels=model[-1].classes_)
assert "sạch" not in model[0].vocabulary_
assert np.isfinite(loss) and loss < np.log(len(model[-1].classes_))
assert np.isclose(-np.log(0.7),0.3566749439)
print("Input tự tạo:", contexts[0], "target:", targets[0])
print("Loss train của pretext đơn giản:", loss)
print("Dự đoán token:", model.predict(["phòng [MASK]"]))
```

Lặp câu cố ý minh họa tần suất target, không phải thiết kế test. Đây là ví dụ pretext nhỏ, chưa đánh giá biểu diễn hoặc sentiment ngoài mẫu; cần dữ liệu và model phù hợp cho nghiên cứu thật.

## 24. Tình huống sai và phản ví dụ

Model có thể dự đoán token bằng template hoặc tên khách sạn mà không giữ cảm xúc. Một encoder có thể nén “phòng sạch” và “phòng không sạch” gần nhau dù pretext loss tốt. Contamination do review trùng giữa pretraining và test cũng làm kết quả lạc quan; cần xử lý và báo giới hạn.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Đánh giá pretext trên dữ liệu giữ lại khi phù hợp, rồi gold downstream theo protocol. Kiểm tra nhóm câu phủ định, hỗn hợp, ngôn ngữ và thời gian. So freeze/fine-tune và baseline cùng tài nguyên. Ghi nguồn pretraining và khả năng overlap, không khẳng định sạch tuyệt đối nếu không biết corpus ngoài.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu pretext tốt nhưng downstream kém, xem mục tiêu, pooling và miền dữ liệu; có thể dùng encoder khác hoặc baseline đơn giản. Nếu overfit, kiểm tra dữ liệu/trùng/regularization và development. Không tiếp tục tối ưu bằng test. Nếu phát hiện target bị lộ trong input, sửa nhiệm vụ và chạy lại thay vì giữ điểm cũ.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ là khả năng thực hiện pretext và/hoặc hiệu quả downstream được đo riêng. Không kết luận “hiểu khách hàng” từ token loss hoặc gọi pretraining không nhãn là toàn bộ nghiên cứu không dùng nhãn. Không suy latent feature tạo ra tác động nhân quả của trải nghiệm.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu dùng encoder [...revision...] pretrained bằng [...], tiếp tục thích nghi trên corpus [...nếu có]. Downstream [freeze/fine-tune] dùng [...] gold labels; test giữ lại [...].” Báo tokenizer, pooling, objective, compute và quyền truy cập corpus. Không đưa ví dụ token nhỏ thành benchmark encoder.

## 29. Khả năng tái lập và tính minh bạch

Lưu định danh/revision model, tokenizer, corpus IDs/quy tắc lọc, cấu hình target/augmentation, seed và downstream split. Nếu không biết corpus pretrained đầy đủ, ghi giới hạn. Mã minh họa đã chạy sklearn 1.8.0, không có thực nghiệm BERT hoặc pretrained transformer trong bài.

## 30. Đạo đức nghiên cứu liên quan

Văn bản không gán nhãn vẫn có thể chứa thông tin cá nhân/nhạy cảm; self-supervision không ẩn danh hóa. Cần quyền sử dụng, quản lý truy cập và xem nguy cơ ghi nhớ. Không suy đặc điểm người viết từ embedding hoặc dùng model tự động cho quyết định bất lợi mà thiếu đánh giá/người rà soát.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Self-supervised có target không?
2. Target token khác nhãn sentiment thế nào?
3. -log(0.7) bằng bao nhiêu?
4. Loss pretext thấp có bảo đảm downstream tốt?

Gợi ý: khoảng 0.3567; cần đánh giá sentiment riêng. Bài mở rộng: lập bảng dữ liệu nào dùng pretraining, fine-tuning, tuning và test.

## 32. Những điểm còn tranh luận

Các vấn đề cần kiểm chứng là liên hệ pretext-downstream, contamination, giới hạn corpus tiếng Việt và chi phí adaptation. Không có objective tốt nhất chung. Nghiên cứu cần tách rõ dùng nhãn ở giai đoạn nào và đo giá trị thực, thay vì chỉ dựa vào quy mô pretraining hoặc tên mô hình.

