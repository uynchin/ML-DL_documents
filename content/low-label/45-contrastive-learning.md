# Contrastive Learning — Học biểu diễn qua cặp tương đồng và khác biệt

> **Nhánh:** MACHINE LEARNING → Ít nhãn. **Nguồn:** dòng 46 bảng Excel. **Đối chiếu:** 29/10/2026. Ví dụ NumPy kiểm tra contrastive loss và false negatives; không huấn luyện SimCLR, SimCSE hay transformer. Dữ liệu và tình huống khách sạn đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bối cảnh | Phân tích cảm xúc review khách sạn tiếng Việt |
| Nguồn tín hiệu | Phân biệt gold labels, pseudo/weak labels và target tự tạo trong bài |
| Đánh giá | Cần protocol và dữ liệu độc lập phù hợp, không có benchmark review thật ở ví dụ |

## 1. Định nghĩa học thuật

Contrastive learning học biểu diễn bằng cách khuyến khích các cặp dương tương đồng theo mục tiêu và phân biệt chúng với các mẫu âm trong một objective phù hợp. Nó có thể dùng cặp tự tạo không nhãn hoặc cặp có giám sát. Vì vậy không mặc định mọi contrastive learning đều không dùng nhãn hoặc đều là một thuật toán duy nhất.

## 2. Giải thích đơn giản

Bạn cho model hai cách nhìn được xem là cùng ý nghĩa cần giữ và yêu cầu đưa chúng gần nhau hơn các ví dụ khác. Nhưng “cùng” phải được xác định theo nhiệm vụ. Hai review cùng nói phòng chưa chắc cùng cảm xúc; xóa từ “không” để tạo view có thể làm đổi hoàn toàn nhãn cần giữ.

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu cảm xúc khách sạn, có thể học sentence embedding để phân loại, clustering hoặc retrieval. Mục tiêu cần nêu rõ: semantic similarity, cùng sentiment hay cùng khía cạnh. Các mục tiêu đó không đồng nhất. Đánh giá downstream phải kiểm tra đúng quan hệ cần cho nghiên cứu, không chỉ xem các cặp train gần hơn.

## 4. Điều kiện và giả định sử dụng

Cần cơ sở cho positive pairs, negative sampling và phép biến đổi không phá thuộc tính cần giữ. Embedding/temperature và batch ảnh hưởng objective. False negatives xảy ra khi mẫu được xem âm thực ra có quan hệ giống cần giữ. Với sentiment, nhiều review tích cực khác nhau có thể là negatives không mong muốn trong thiết kế instance-level.

## 5. Cơ chế hoạt động bên trong

Encode các view bằng encoder, có thể qua projection head, chuẩn hóa vector rồi tính similarity. Với InfoNCE/NT-Xent-style loss, tăng điểm positive tương đối các candidates khác. Backprop cập nhật weights khi huấn luyện native. SimCLR xuất phát từ hình ảnh; SimCSE có thiết kế sentence embeddings riêng, không lấy augmentation ảnh áp nguyên vào review [S1–S2].

## 6. Quy trình áp dụng trong nghiên cứu

1. Chốt quan hệ muốn học và downstream task.
2. Tách dữ liệu theo protocol, kiểm tra trùng/miền.
3. Tạo positive/negative hoặc views có cơ sở, audit sentiment preservation.
4. Chọn encoder/head, similarity, temperature và batch.
5. Train/tune trên dữ liệu phát triển phù hợp.
6. Đánh giá gold downstream, false negatives và chi phí.

Nếu dùng nhãn sentiment để chọn cặp, báo là supervised contrastive ở bước đó; không gọi toàn quy trình không nhãn.

## 7. Ví dụ minh họa xuyên suốt

Anchor “phòng không sạch” và một view giữ nguyên nội dung có thể là positive. Biến thành “phòng sạch” bằng xóa “không” không phải positive hợp lệ cho mục tiêu giữ sentiment. Hai câu “phòng bẩn” và “dịch vụ tệ” khác chủ đề nhưng có thể cùng lớp tiêu cực nếu mục tiêu là sentiment; thiết kế cặp phải phản ánh điều này.

## 8. Kết quả và cách diễn giải

Đầu ra thường là encoder/embedding, không phải nhãn hay xác suất cảm xúc. Cosine tăng cho positive trong batch không tự chứng minh chất lượng semantic hoặc class separation ngoài mẫu. Loss phụ thuộc số candidates/temperature, nên so raw loss giữa batch sizes khác nhau cần thận trọng.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là có thể học cấu trúc biểu diễn từ quan hệ/cặp và tận dụng dữ liệu chưa nhãn theo thiết kế. Hạn chế là chọn view/negative khó, false negatives, shortcut, compute và nguy cơ mismatch downstream. Tính gần ngữ nghĩa có thể che polarity; không có một embedding tối ưu cho mọi mục tiêu.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| Self-supervised | Họ mục tiêu tự tạo; contrastive có thể là một trường hợp |
| Supervised contrastive | Dùng nhãn để xác định quan hệ cặp |
| Autoencoder | Tái tạo đầu vào thay vì cùng contrastive objective |
| Metric learning | Họ rộng hơn, có overlap với contrastive |
| Classifier | Tối ưu trực tiếp dự đoán nhãn task |

Không đồng nhất supervised contrastive với không cần gold labels.

## 11. Sai lầm thường gặp và cách khắc phục

Không xóa phủ định hoặc đổi từ cảm xúc khi chưa kiểm tra view. Không gọi mọi câu khác trong batch là negative đúng về sentiment. Không đọc cosine âm là tiêu cực. Không đánh giá bằng chính cặp train rồi suy generalization. Không tuyên bố đã huấn luyện SimCSE từ đoạn NumPy chỉ tính loss.

## 12. Tài liệu tham khảo

- [S1] Chen, T., Kornblith, S., Norouzi, M., & Hinton, G. (2020). *A Simple Framework for Contrastive Learning of Visual Representations*. [SimCLR](https://arxiv.org/abs/2002.05709).
- [S2] Gao, T., Yao, X., & Chen, D. (2021). *SimCSE: Simple Contrastive Learning of Sentence Embeddings*. [Bài gốc](https://aclanthology.org/2021.emnlp-main.552/).

Hai công trình là cơ chế tham khảo khác nhau, không định nghĩa mọi contrastive learning. Ví dụ loss dưới tự soạn, không có native training.

## 13. Vị trí trong toàn bộ nghiên cứu

Contrastive learning thường ở học biểu diễn trước classifier/search hoặc là loss kết hợp trong fine-tuning. Phải xác định nhãn/quan hệ dùng ở bước này trước diễn giải downstream. Nếu dùng cùng sentiment labels tạo cặp và classifier, tính chúng trong ngân sách và tránh dùng test labels để tạo positives.

## 14. Kiến thức nền cần biết trước

Cần hiểu vector normalization, cosine, softmax/log-sum-exp, positive/negative pairs và augmentation. Biết dropout-based views khác sửa nội dung câu. Với tiếng Việt, xóa dấu, tách từ, thay đồng nghĩa hoặc cắt câu có thể đổi nghĩa; cần audit thay vì xem mọi biến đổi bề mặt là invariance an toàn.

## 15. Nền tảng lý thuyết và logic hình thành

Objective so sánh tương đối khiến encoder giữ các thuộc tính cần phân biệt positive với candidates. Nó không tự biết thuộc tính nghiên cứu nào quan trọng; cách chọn cặp quyết định nhiều. Instance-level positive có thể hữu ích cho semantic representation nhưng không bảo đảm gom toàn bộ review cùng sentiment vào một vùng.

## 16. Các thành phần cấu tạo

Các thành phần gồm corpus, view/pair generator, encoder, projection head tùy chọn, similarity, temperature, batch/negative pool, loss và optimizer. Phân biệt representation dùng cho objective với representation chọn downstream. Các bước normalize/pooling/head phải được lưu, không chỉ tên backbone.

## 17. Các loại và biến thể

Có instance discrimination, supervised contrastive, triplet-based objectives và nhiều biến thể sampling. SimCSE không giám sát dùng dropout tạo view cùng câu, bản có giám sát dùng dữ liệu NLI theo thiết kế [S2]. Không khẳng định mọi phương pháp contrastive đều cần explicit negatives hoặc có cùng denominator như ví dụ bài.

## 18. Đầu vào và đầu ra

Đầu vào là cặp/view cùng quan hệ và có thể có nhãn dùng tạo cặp. Đầu ra là embeddings/weights, loss và các thông tin training. Bộ phân loại downstream là thành phần riêng nếu cần sentiment labels. Không dùng số chiều embedding hoặc cosine làm tỷ lệ cảm xúc.

## 19. Căn cứ lựa chọn

Đáng thử khi có lý do quan hệ cặp tạo biểu diễn phù hợp và đủ dữ liệu/compute. Chọn dựa trên gold downstream, retrieval relevance hoặc tiêu chí đúng nhiệm vụ, kèm baseline frozen/pretrained. Kiểm tra cả câu phủ định/cùng chủ đề trái sentiment để biết model giữ thuộc tính nào.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu không xác định được positive hợp lệ hoặc pool negatives có noise nặng mà không kiểm tra. Không học từ đầu chỉ vì contrastive đang phổ biến nếu baseline đáp ứng và tài nguyên hạn chế. Nếu nhiệm vụ cần exact sentiment, embedding similarity riêng không thay classifier/nhãn chuẩn.

## 21. Các quyết định quan trọng khi thực hiện

Chọn quan hệ positive, augmentations, hard/false negatives, temperature, batch, encoder/head và evaluation. Với supervised pairs, bảo đảm chỉ dùng nhãn train. Nếu dùng mining từ toàn corpus gồm test input, phải công bố thiết kế; không báo hoàn toàn inductive. Ghi tổng compute và quyền truy cập dữ liệu.

## 22. Giải thích công thức và ký hiệu

Một dạng InfoNCE minh họa cho anchor i, positive p(i), tập candidates A(i) không gồm chính anchor:

$$\ell_i=-\log\frac{\exp(s(z_i,z_{p(i)})/\tau)}{\sum_{a\in A(i)}\exp(s(z_i,z_a)/\tau)}.$$

$\tau>0$ là temperature; denominator có positive và negatives trong ví dụ. Nếu chỉ có positive giống hệt anchor và một negative trực giao, similarity là 1 và 0, $\tau=1$ thì loss $=\log(1+e^{-1})\approx0.3133$. Đây là xác suất lựa chọn trong objective, không là xác suất cảm xúc.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mã NumPy chỉ tính loss từ vector tự tạo; không cập nhật encoder hoặc huấn luyện SimCLR/SimCSE.

```python
import numpy as np

def unit(v):
    norm = np.linalg.norm(v)
    assert norm > 0
    return v/norm

def contrastive_loss(anchor, positive, negatives, tau=1.0):
    assert tau > 0
    a = unit(np.asarray(anchor,dtype=float))
    candidates = np.vstack([unit(np.asarray(positive,dtype=float))]
                           + [unit(np.asarray(n,dtype=float)) for n in negatives])
    logits = (candidates@a)/tau
    log_denom = np.logaddexp.reduce(logits)
    return float(log_denom-logits[0])

clean = contrastive_loss([1,0],[1,0],[[0,1]],tau=1)
false_negative = contrastive_loss([1,0],[1,0],[[0,1],[1,0]],tau=1)
assert np.isclose(clean,np.log1p(np.exp(-1)))
assert false_negative > clean
print("Loss với negative trực giao:", clean)
print("Loss khi thêm false negative trùng positive:", false_negative)
```

Loss tăng khi thêm candidate rất giống positive minh họa áp lực của false negatives trong objective này, không chứng minh mọi chiến lược thêm negatives đều gây hại. Huấn luyện native cần pipeline và dữ liệu kiểm tra riêng.

## 24. Tình huống sai và phản ví dụ

Hai câu cùng tích cực khác chủ đề có thể bị xem negatives trong instance contrastive. Nếu mục tiêu muốn gom sentiment, điều này cần phân tích chứ không tự đúng. Một augmentation xóa “không” có thể kéo hai sentiment trái nhau gần. Một model có thể học dấu nguồn/độ dài làm shortcut thay vì thuộc tính cần giữ.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Kiểm tra pairs/views bằng audit nội dung, false negatives và gold downstream theo nhóm. Lặp seed và báo compute. So embedding gốc với sau fine-tuning trong protocol cùng quyền truy cập. Với retrieval, đánh giá relevance độc lập; không chỉ đọc vài nearest neighbors thuận câu chuyện hoặc chỉ so loss train.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu sentiment kém, xem pair objective có khớp nhiệm vụ không, kiểm tra phủ định và false negatives. Có thể thay views/mining hoặc dùng supervised contrastive khi có nhãn hợp lệ, nhưng phải công bố thay đổi. Nếu không có lợi ích ngoài mẫu, giữ baseline; không tối ưu temperature/batch theo test cuối.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ là chất lượng embedding/downstream trong thiết kế cặp và evaluation cụ thể. Không suy cosine là sentiment hoặc loss nhỏ chứng minh hiểu ngữ nghĩa. Nếu dùng nhãn tạo cặp, không kết luận toàn quy trình không cần nhãn. Liên hệ dự đoán khác bằng chứng nhân quả về trải nghiệm khách.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu học embedding với [objective], positives [...], negatives/mining [...], encoder/head [...], temperature/batch [...]. Quyền truy cập corpus/nhãn [...]. Downstream đánh giá trên gold test [...].” Báo pooling/normalization, compute, model revision và audit augmentation. Chỉ điền kết quả native đã thực hiện.

## 29. Khả năng tái lập và tính minh bạch

Lưu corpus/pair IDs hoặc thuật toán tạo view, seed, encoder/revision, head, loss/temperature, optimizer và downstream split. Với mining động, lưu cấu hình và checkpoints cần tái lập. Mã mục 23 chỉ chạy loss NumPy; chưa huấn luyện transformer hoặc package SimCSE trong bài.

## 30. Đạo đức nghiên cứu liên quan

Tạo cặp/search có thể làm lộ liên hệ giữa review hoặc thông tin người viết. Kiểm soát dữ liệu cá nhân và không suy cùng tác giả/gian lận chỉ từ similarity. Augmentation cần tránh xóa tín hiệu ngôn ngữ của nhóm ít đại diện. Quyết định tác động người thật cần đánh giá/người rà soát.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Contrastive có luôn không dùng nhãn không?
2. Xóa “không” có là view hợp lệ cho sentiment?
3. False negative nghĩa là gì?
4. Loss ví dụ positive=1, negative=0, τ=1 bao nhiêu?

Gợi ý: khoảng 0.3133. Bài mở rộng: audit positives theo cả khía cạnh lẫn polarity và thiết kế gold evaluation phù hợp.

## 32. Những điểm còn tranh luận

Các vấn đề cần kiểm chứng là positive semantics, false negatives trong review, hiệu quả tiếng Việt và đánh đổi compute. Không có augmentation/temperature tốt nhất chung. Nghiên cứu cần nói rõ thuộc tính nào được giữ và kiểm tra bằng nhiệm vụ thật, thay vì coi contrastive loss giảm là đủ.

