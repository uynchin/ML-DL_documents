# CRF — Trường ngẫu nhiên có điều kiện cho gán nhãn chuỗi

> **Nhánh:** MACHINE LEARNING → Supervised: Probabilistic. **Nguồn:** dòng 21 bảng Excel. **Phạm vi:** linear-chain CRF để trích khía cạnh bằng nhãn BIO; không đồng nhất mọi CRF với cấu trúc chuỗi. **Đối chiếu:** 28/10/2026. Ví dụ, điểm và ràng buộc minh họa tự đặt. Mã thực hiện suy luận từ điểm cố định, chưa huấn luyện trọng số từ corpus.

| Trường | Nội dung |
|---|---|
| Bản chất | Họ mô hình xác suất có điều kiện cho đầu ra có cấu trúc |
| Mốc | Lafferty, McCallum & Pereira, 2001; tutorial Sutton & McCallum, 2010 [S1–S2] |
| Đầu vào | Chuỗi quan sát, feature/điểm theo nhãn và dữ liệu chuỗi nhãn khi huấn luyện |
| Đầu ra | Chuỗi nhãn tốt nhất, xác suất chuỗi/marginal tùy thuật toán và API |
| Tiếng Việt | Cần tokenization, căn chỉnh span và nhãn phù hợp; token có thể là âm tiết/từ/subword theo thiết kế |
| Nhãn/tài nguyên | CRF có giám sát cần nhãn chuỗi; first-order linear-chain suy luận thường có chi phí $O(TK^2)$ cho $T$ token, $K$ nhãn |
| Phần mềm | CRFsuite/sklearn-crfsuite là lựa chọn tham khảo; neural-CRF cần thêm encoder. Demo dưới chỉ dùng NumPy/SciPy |

## 1. Định nghĩa học thuật

Conditional Random Field mô hình hóa phân bố có điều kiện của các biến đầu ra có cấu trúc khi đã biết quan sát. Linear-chain CRF liên kết các nhãn theo chuỗi, cho phép quyết định một nhãn phụ thuộc vào ngữ cảnh quan sát và quan hệ với nhãn lân cận [S1–S2].

Bài này dùng CRF để trích cụm khía cạnh, không gán sentiment toàn câu. Trích “nhân viên” là khía cạnh và xác định ý kiến về nhân viên tích cực hay tiêu cực là hai nhiệm vụ khác nhau.

## 2. Giải thích đơn giản

Thay vì chấm mỗi token riêng rồi ghép lại, CRF chọn một chuỗi nhãn có tổng điểm tốt. Nếu token thứ hai là phần bên trong một cụm, nhãn của token đầu phải phù hợp.

Trong “nhân viên tốt”, ta muốn “nhân” bắt đầu khía cạnh, “viên” tiếp tục khía cạnh và “tốt” ở ngoài cụm khía cạnh. Quy tắc nối nhãn giúp tránh chuỗi kỳ lạ như bắt đầu ngay bằng nhãn “bên trong”.

## 3. Mục đích và vấn đề giải quyết

Đề xuất nghiên cứu khách sạn: trích span khía cạnh như “phòng”, “nhân viên”, “bữa sáng” từ đánh giá, rồi đưa các span sang bước phân loại sentiment theo khía cạnh.

Cũng có thể thiết kế nhãn kết hợp khía cạnh–cực tính hoặc trích opinion, nhưng cần dữ liệu và quy ước riêng. Không mặc định CRF cơ bản trả cả bộ aspect–opinion–sentiment chỉ vì nó dự đoán được một chuỗi BIO.

## 4. Điều kiện và giả định sử dụng

Cần tokenization và nhãn chuỗi căn chỉnh đúng, hướng dẫn ranh giới rõ và tập dữ liệu có span thật. Với tiếng Việt, “nhân viên” có thể là hai đơn vị âm tiết hoặc một từ; phải giữ nhất quán từ train đến đánh giá.

First-order chain giới hạn phụ thuộc nhãn qua láng giềng trong score cục bộ, nhưng feature quan sát có thể dùng ngữ cảnh rộng hơn. Không nhầm giả định nhãn chuỗi với yêu cầu các từ quan sát độc lập như HMM đơn giản.

## 5. Cơ chế hoạt động bên trong

Một quy trình gồm tính điểm nhãn tại token, tính điểm chuyển nhãn, cộng điểm toàn chuỗi rồi chuẩn hóa trên các chuỗi có thể. Huấn luyện tăng log-likelihood chuỗi đúng và kiểm soát trọng số theo regularization.

Forward tính partition function; forward–backward tính marginal; Viterbi tìm chuỗi MAP có điểm cao nhất. Ba nhiệm vụ liên quan nhưng không đồng nhất. Chọn nhãn marginal cao nhất ở từng token không luôn tạo chuỗi MAP hoặc chuỗi hợp lệ theo ràng buộc.

## 6. Quy trình áp dụng trong nghiên cứu

1. Chọn nhiệm vụ/span: aspect, opinion hoặc nhãn kết hợp có định nghĩa rõ.
2. Chuẩn hóa tokenization và hướng dẫn BIO/BIOES, xử lý câu không có span.
3. Gán nhãn độc lập, kiểm tra bất đồng và căn chỉnh offset.
4. Chia theo đánh giá/khách sạn, không rải token cùng câu qua train/test.
5. Thiết kế feature hoặc encoder và CRF; chọn regularization trên development.
6. Cố định tập chuỗi hợp lệ và decoder, kiểm tra train/inference nhất quán.
7. Đánh giá exact span F1 và lỗi biên, sau đó đánh giá pipeline sentiment nếu có.

Mã minh họa mục 23 kiểm tra suy luận. Muốn thành mô hình học thật phải bổ sung huấn luyện bằng chuỗi nhãn và kiểm định ngoài mẫu.

## 7. Ví dụ minh họa xuyên suốt

Với token `nhân | viên | tốt`, chọn bộ nhãn `B-ASP` bắt đầu khía cạnh, `I-ASP` tiếp tục, `O` ngoài khía cạnh. Chuỗi mong muốn là `B-ASP | I-ASP | O` theo quy ước này.

“Tốt” không được gán nhãn khía cạnh dù là từ cảm xúc. Nếu muốn trích opinion, phải có lớp/đầu ra khác. Việc trích “nhân viên” đúng chưa đủ biết ai được khen trong câu nhiều đối tượng hoặc có phủ định.

## 8. Kết quả và cách diễn giải

Đầu ra chính có thể là chuỗi nhãn và span quy đổi về văn bản gốc. Xác suất chuỗi là xác suất của toàn bộ chuỗi theo mô hình; marginal token là xác suất nhãn tại một vị trí sau khi tổng qua chuỗi khác.

Không lấy trung bình marginal rồi gọi là xác suất toàn span khi chưa có phép tính đúng. Xác suất cao cũng chưa chứng minh calibration hay ranh giới đúng ở dữ liệu mới. Cần đọc cùng tokenization và quy tắc chuyển nhãn thành span.

## 9. Điểm mạnh, điểm yếu và giới hạn

Điểm mạnh là mô hình hóa đầu ra có liên kết, dùng feature quan sát linh hoạt và suy luận hiệu quả ở chain. Ràng buộc/transition có thể giúp tránh tổ hợp nhãn không phù hợp.

Giới hạn là cần nhãn span tốn công, phụ thuộc feature/encoder và quy ước nhãn. CRF chain chuẩn không tự giải aspect ngầm, span lồng hoặc chồng lấn. Ràng buộc BIO cũng không bảo đảm span đúng nghĩa: chuỗi hợp lệ về cấu trúc vẫn có thể trích sai từ.

## 10. So sánh với phương pháp gần nhau

| Phương pháp | Khác biệt |
|---|---|
| Token classifier độc lập | Không có điểm chuyển nhãn toàn chuỗi nếu chưa thêm cấu trúc |
| HMM | Mô hình joint sinh chuỗi theo giả định khác; CRF trực tiếp điều kiện hóa vào quan sát |
| MEMM | Chuẩn hóa cục bộ ở bước chuyển, khác chuẩn hóa toàn chuỗi của CRF |
| MaxEnt classifier | Gán một nhãn/đơn vị; CRF mở sang đầu ra có cấu trúc |
| Encoder + CRF | Encoder tạo điểm, CRF là tầng cấu trúc; cải thiện cần tách đóng góp |

Label bias gắn với cấu trúc chuẩn hóa cục bộ là một động cơ nghiên cứu CRF; điều đó không nghĩa CRF miễn mọi loại thiên lệch dữ liệu [S1–S2].

## 11. Sai lầm thường gặp và cách khắc phục

- Báo token accuracy cao khi đa số là O: thêm metric span và từng loại.
- Lấy argmax từng token rồi gọi là Viterbi: dùng decoder đúng.
- Tin CRF luôn cấm BIO sai: ràng buộc phải có hoặc được triển khai; transition học không tự là lệnh cấm.
- Đánh giá trên tokenization khác gold mà không quy về span: xác định quy tắc căn chỉnh.
- Dùng aspect trích được như sentiment đã xác định: tách nhiệm vụ và đánh giá pipeline.

## 12. Tài liệu tham khảo

- **[S1]** Lafferty, J., McCallum, A., & Pereira, F. (2001). *Conditional Random Fields: Probabilistic Models for Segmenting and Labeling Sequence Data*. ICML, 282–289. [Bản công trình lưu tại đại học](https://home.cs.colorado.edu/~mozer/Teaching/syllabi/6622/papers/LaffertyMcCallumPereira2001.pdf).
- **[S2]** Sutton, C., & McCallum, A. (2010, bản arXiv). *An Introduction to Conditional Random Fields*. [Bản tác giả](https://arxiv.org/abs/1011.4088).
- **[S3]** sklearn-crfsuite. [Tutorial về feature, fit và dự đoán](https://sklearn-crfsuite.readthedocs.io/en/latest/tutorial.html).

S3 là hướng dẫn triển khai tham khảo, có thể chứa API cũ; cần kiểm tra môi trường thay vì sao chép nguyên mọi import. Ví dụ trong bài này là suy luận từ điểm tự đặt, không tái lập benchmark hoặc huấn luyện của S1–S3.

## 13. Vị trí trong toàn bộ nghiên cứu

CRF trích khía cạnh thường nằm sau tokenization/feature và trước liên kết opinion hoặc phân loại cực tính. Kết quả là biến trung gian của pipeline ABSA.

Cần đánh giá cả bước trích và pipeline hoàn chỉnh. Nếu chỉ đánh giá polarity trên khía cạnh gold, đó là đánh giá có điều kiện với đầu vào đúng, chưa phản ánh lỗi khi khía cạnh được máy trích. Hai thiết kế đều có thể hữu ích nhưng phải gọi đúng.

## 14. Kiến thức nền cần biết trước

Cần hiểu token, span, nhãn BIO, sequence score, transition, partition function, marginal và MAP. `B`/`I` mô tả biên cụm, không phải mức tích cực/tiêu cực.

Phân biệt thuật toán huấn luyện và thuật toán giải mã. Viterbi chọn chuỗi tốt nhất với trọng số đã có; nó không tự học trọng số từ gold labels. Forward tính tổng, Viterbi tính cực đại; thay tổng bằng max làm đổi đại lượng.

## 15. Nền tảng lý thuyết và logic hình thành

Đầu ra như chuỗi nhãn có cấu trúc, nên quyết định ở một vị trí có thể liên quan quyết định ở vị trí khác. CRF mô hình hóa liên hệ đó trong một phân bố có điều kiện được chuẩn hóa toàn cục.

Việc dùng full observation cho feature không có nghĩa mọi phụ thuộc giữa nhãn đều được giữ. First-order chain là lựa chọn giới hạn để suy luận khả thi; nếu cần span chồng lấn hoặc quan hệ xa, có thể cần cấu trúc/biểu diễn khác với chi phí khác.

## 16. Các thành phần cấu tạo

Hệ gồm tokenizer/căn chỉnh, feature hoặc encoder, điểm nhãn token, transition, start/end, mask ràng buộc nếu có, thuật toán normalization, decoder và metric span.

Trong CRF truyền thống, điểm có thể đến từ feature log-linear; trong neural-CRF, điểm đến từ encoder học được. Không gọi hai hệ là chỉ khác decoder nếu encoder cũng đổi. Một ablation phù hợp cần giữ các thành phần còn lại giống nhau khi đo giá trị của tầng CRF.

## 17. Các loại và biến thể

Có linear-chain, CRF bậc cao hơn, semi-Markov cho span và các cấu trúc đồ thị khác. Chỉ một số cấu trúc cho phép suy luận exact hiệu quả như chain bậc một.

BIO, BIOES và nhãn ghép sentiment là các lựa chọn biểu diễn đầu ra. Bộ nhãn lớn làm chi phí tăng và cần thêm dữ liệu cho transition. Không nhận demo một loại ASP như hệ có thể tự xử lý mọi loại aspect/opinion hoặc span lồng.

## 18. Đầu vào và đầu ra

Đầu vào của chain là chuỗi dài $T$, mỗi vị trí có feature/score. Đầu ra lưu token gốc, offset, nhãn gold/pred, span chuyển đổi, loại và score/probability nếu tính.

Nếu dùng subword, cần chính sách gộp về word/span. Nếu chuẩn hóa thay độ dài chuỗi, phải giữ mapping sang raw text. Số nhãn và số token cần khớp, không lặng lẽ cắt chuỗi thừa để mã chạy được.

## 19. Căn cứ lựa chọn

Nên thử khi cần trích các cụm có ranh giới và nhãn lân cận mang thông tin. Đánh giá bằng đối chứng token classifier với cùng feature/encoder để biết lợi ích tầng cấu trúc.

Nếu nhiệm vụ chỉ một nhãn cho cả đánh giá, CRF chuỗi có thể không cần thiết. Nếu dữ liệu chủ yếu aspect ngầm, trích token rõ mặt chữ không đáp ứng đầy đủ; cần điều chỉnh bài toán và chuẩn đánh giá.

## 20. Trường hợp không nên sử dụng

Không kỳ vọng một chuỗi BIO đơn chuẩn diễn đạt đầy đủ nhiều span lồng/chồng lấn trên cùng token. Không dùng accuracy gồm O làm bằng chứng duy nhất cho chất lượng trích aspect.

Nếu chưa có nhãn span, không nhận chuỗi tạo bằng lexicon là gold tự nhiên mà không kiểm tra. Có thể nghiên cứu weak supervision nhưng phải có test người gán độc lập và công bố nguồn nhãn yếu.

## 21. Các quyết định quan trọng khi thực hiện

Chọn tokenization, loại span, BIO variant, feature, regularization và decoder. Hard mask cấm transition phải nhất quán với gold và normalization; cấm một chuỗi gold sẽ làm likelihood không hợp lệ hoặc không thể tối ưu như dự kiến.

Start/end và câu rỗng cũng cần chính sách. Với batch padding, mask vị trí padding khác mask cấm transition. Không dùng cùng tên “mask” để bỏ qua sự khác nhau đó khi mô tả triển khai.

## 22. Giải thích công thức và ký hiệu

Một chain bậc một có thể dùng score:

$$s(x,y)=a_{y_1}+\sum_{t=1}^{T}e_t(y_t)
+\sum_{t=2}^{T}A_{y_{t-1},y_t}+b_{y_T}.$$

$e_t$ là điểm theo nhãn tại token, $A$ là điểm chuyển, $a,b$ là start/end. Các điểm là log-potential chưa chuẩn hóa, không nhất thiết là log xác suất cục bộ.

$$P(y\mid x)=\frac{e^{s(x,y)}}{Z(x)},\qquad
Z(x)=\sum_{y'\in\mathcal Y(x)}e^{s(x,y')}.$$

$\mathcal Y(x)$ là tập chuỗi cho phép. Huấn luyện có thể tối thiểu hóa $-s(x,y^{gold})+\log Z(x)$ cộng regularization. Với feature tuyến tính cố định, mục tiêu negative conditional log-likelihood có tính lồi; với neural encoder cùng học, không áp kết luận đó cho toàn bộ hệ.

Forward log-space dùng log-sum-exp qua trạng thái trước; Viterbi thay phép tổng đó bằng max và lưu backpointer. Marginal cần tổng qua các chuỗi, không phải chọn từng nhãn có emission lớn nhất. Với hard mask, score cấm đặt $-\infty$ và bị loại khỏi tổng/max.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Đặt thứ tự nhãn `[O, B-ASP, I-ASP]`. Ba hàng emission cho “nhân | viên | tốt” là $(0,2,3)$, $(0,0,2)$, $(2,0,0)$. Chọn riêng emission sẽ bắt đầu bằng I-ASP, sai quy ước BIO của demo.

Đặt điểm start của I-ASP là $-\infty$, cấm O→I-ASP và các transition còn lại bằng 0, end bằng 0. Viterbi chọn B-ASP→I-ASP→O với score $2+2+2=6$. Đây là điểm tự đặt, không phải điểm học từ câu thật.

Mã NumPy/SciPy kiểm tra Viterbi và forward bằng cách đối chiếu liệt kê mọi chuỗi của ví dụ ngắn. Không cần CRFsuite; **không có bước fit trọng số** trong mã này.

```python
import itertools
import numpy as np
from scipy.special import logsumexp

labels = ["O", "B-ASP", "I-ASP"]
emission = np.array([[0., 2., 3.], [0., 0., 2.], [2., 0., 0.]])
transition = np.zeros((3, 3))  # [nhãn trước, nhãn sau]
transition[0, 2] = -np.inf
start = np.array([0., 0., -np.inf])
end = np.zeros(3)
T, K = emission.shape

# Forward: cộng xác suất trong log-space.
alpha = start + emission[0]
for t in range(1, T):
    alpha = emission[t] + logsumexp(alpha[:, None] + transition, axis=0)
log_z = logsumexp(alpha + end)

# Viterbi: cực đại và truy ngược.
dp = start + emission[0]
backpointers = []
for t in range(1, T):
    candidates = dp[:, None] + transition
    backpointers.append(candidates.argmax(axis=0))
    dp = candidates.max(axis=0) + emission[t]
last = int((dp + end).argmax())
best_score = float((dp + end)[last])
path = [last]
for back in reversed(backpointers):
    path.append(int(back[path[-1]]))
path = list(reversed(path))

def sequence_score(seq):
    value = start[seq[0]] + end[seq[-1]]
    value += sum(emission[t, seq[t]] for t in range(T))
    value += sum(transition[seq[t - 1], seq[t]] for t in range(1, T))
    return float(value)

sequences = list(itertools.product(range(K), repeat=T))
all_scores = np.array([sequence_score(seq) for seq in sequences])
assert np.isclose(log_z, logsumexp(all_scores))
assert np.isclose(np.exp(all_scores - log_z).sum(), 1.0)
assert tuple(path) == sequences[int(all_scores.argmax())]
assert path == [1, 2, 0] and np.isclose(best_score, 6.0)
assert int(emission[0].argmax()) == 2  # argmax riêng tạo I đầu câu
print("Chuỗi MAP:", [labels[i] for i in path])
print("P(chuỗi MAP | x), giả định:", round(float(np.exp(best_score - log_z)), 4))
```

Liệt kê $K^T$ chỉ dùng kiểm tra ví dụ rất ngắn. Trong nghiên cứu thật, huấn luyện emission/transition bằng nhãn chuỗi, dùng thuật toán động và đo trên test độc lập. Hard constraints ở đây là lựa chọn rõ ràng, không khẳng định mọi thư viện CRF tự cấm BIO sai.

## 24. Tình huống sai và phản ví dụ

Chuỗi O→I-ASP có thể xuất hiện nếu giải mã độc lập hoặc CRF không có mask và transition học chưa đủ mạnh. Ngay cả chuỗi B-ASP→I-ASP hợp lệ vẫn có thể trích “viên tốt” thay vì “nhân viên” nếu emission sai.

Một câu “Không có gì để chê” không có aspect tường minh theo mọi quy ước. Không ép CRF trích một cụm chỉ vì hệ downstream muốn có đầu vào; cần hỗ trợ câu không có aspect hoặc một nhiệm vụ khác cho aspect ngầm.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Ưu tiên exact span precision/recall/F1 theo loại, quy định ranh giới và cách xử lý nhãn sai. Token accuracy/F1 là chỉ báo phụ, không thay span F1. Báo số câu không aspect và lớp O để người đọc hiểu độ lệch.

Kiểm tra biên đầu/cuối, từ nhiều âm tiết, OOV và chuyển miền. Với encoder+CRF, ablation giữ encoder để đo tầng CRF. Với pipeline ABSA, báo cả gold-aspect polarity và end-to-end khi phù hợp, không gộp hai thí nghiệm.

## 26. Xử lý khi không đáp ứng yêu cầu

Lỗi căn chỉnh: sửa token/offset trước. Lỗi biên: đọc gold guidelines và feature ngữ cảnh. Chuỗi không hợp lệ: xem decoder/mask và thống nhất quy ước nhãn. Nhãn ít: thêm dữ liệu hoặc đơn giản hóa mục tiêu.

Nếu span lồng/chồng lấn phổ biến, đổi biểu diễn thay vì cố ép BIO một chuỗi. Nếu aspect đúng nhưng polarity sai, sửa bước sentiment; không đổ mọi lỗi downstream cho CRF trích span.

## 27. Phân biệt kết quả với kết luận

“CRF trích đúng span nhân viên” chưa có nghĩa mô hình hiểu thái độ đối với nhân viên. “Chuỗi MAP có probability 0.8” không chứng minh xác suất đúng thực 80% hoặc trạng thái tâm lý của khách.

Nếu dùng tỷ lệ aspect bị phàn nàn để đề xuất quản trị, cần tính cả recall trích và độ đúng polarity. Chỉ phân tích các span máy tìm được có thể bỏ sót có hệ thống những phản hồi gián tiếp.

## 28. Cách viết vào bài nghiên cứu

Nêu cấu trúc chain, label scheme, đơn vị token, feature/encoder, transition/start/end, mask, objective, regularization và decoder. Ghi rõ loại metric span và quy tắc xử lý BIO sai.

Khung: “Chúng tôi gán nhãn [BIO] cho [loại span], học CRF bằng [cấu hình] trên [train], chọn theo [development], đánh giá exact span F1 trên [test].” Nếu chỉ chạy suy luận điểm tự đặt như demo, gọi đúng là minh họa thuật toán, không báo là mô hình đã huấn luyện.

## 29. Khả năng tái lập và tính minh bạch

Lưu hướng dẫn nhãn, tokenizer, offset mapping, label IDs, feature, mask, start/end, model weights, split và version. Số ID nhãn đảo thứ tự có thể làm giải mã sai dù ma trận kích thước vẫn khớp.

GitHub có thể chứa câu tự soạn và code kiểm tra động–liệt kê. Với corpus thật, cần quyền công bố và ẩn danh. Nếu dùng tutorial cũ, cập nhật API theo môi trường và ghi phiên bản đã chạy, không nhận các import lịch sử còn dùng được nguyên trạng.

## 30. Đạo đức nghiên cứu liên quan

Khía cạnh máy trích có thể bỏ sót cách nói gián tiếp, phương ngữ hoặc nhóm ít đại diện. Kiểm tra sai số trước khi kết luận nhóm khách nào quan tâm ít hơn đến một dịch vụ.

Span có thể chứa tên nhân viên hoặc thông tin cá nhân. Khi công bố nhãn/ví dụ, chỉ giữ nội dung cần thiết và bảo vệ nhận diện. Không dùng một span tự động như bằng chứng đủ để quy trách nhiệm cá nhân.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. **Viterbi có học trọng số không?** Không; nó giải mã với trọng số đã có.
2. **Marginal argmax có luôn là chuỗi MAP không?** Không.
3. **CRF có tự cấm BIO sai không?** Không phải mọi cấu hình; phải kiểm tra constraints/decoder.
4. **Bài tập tính:** trong demo, chuỗi B-ASP→B-ASP→O có score 4, kém chuỗi MAP score 6; giải thích vì sao xác suất còn cần $Z$.
5. **Bài tập thiết kế:** quy định aspect “bữa sáng” theo token âm tiết và token từ, nêu cách quy đổi cả hai về cùng span raw để so công bằng.

## 32. Những điểm còn tranh luận

Các lựa chọn chính là cấu trúc nhãn, lợi ích CRF trên encoder mạnh, ràng buộc cứng so với transition học và đánh giá span ngoài miền. Không có bảo đảm thêm CRF luôn cải thiện mọi encoder/corpus.

Nên tách cải thiện do tokenization, feature, encoder và cấu trúc thay vì gán hết cho một tên mô hình. Một hệ có chuỗi hợp lệ vẫn có thể đo sai khía cạnh, nên đánh giá cấu trúc và giá trị nhiệm vụ phải đi cùng nhau.
