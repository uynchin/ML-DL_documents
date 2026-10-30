# Autoencoder — Học mã hóa và tái tạo biểu diễn review

> **Nhánh:** MACHINE LEARNING → Unsupervised và Topic. **Nguồn:** dòng 40 bảng Excel. **Đối chiếu:** 29/10/2026. Mã NumPy huấn luyện mạng nhỏ có bottleneck, không dùng nhãn cảm xúc. Đây không phải mô hình topic tự động hay mô hình sinh văn bản hoàn chỉnh. Mọi ví dụ và dữ liệu trong bài đều tự tạo.

| Trường | Nội dung |
|---|---|
| Vai trò | Khám phá/biểu diễn không dùng nhãn cảm xúc trong ví dụ |
| Bối cảnh | Review khách sạn tiếng Việt; không có benchmark dữ liệu thật |
| Đọc kết quả | Phân biệt cấu trúc học được với nhãn/khái niệm được xác thực |

## 1. Định nghĩa học thuật

Autoencoder học encoder ánh xạ đầu vào x thành code h và decoder tái tạo x̂ từ code, thường tối ưu reconstruction loss với ràng buộc/regularization. Nó có thể học feature không cần nhãn cảm xúc, nhưng mục tiêu tái tạo không mặc định tạo topic hoặc sentiment [S1]. Autoencoder thường được xếp không giám sát; target từ chính dữ liệu cũng liên hệ cách nhìn self-supervised.

## 2. Giải thích đơn giản

Hình dung nén một bảng chỉ báo review thành vài số rồi cố dựng lại bảng ban đầu. Nếu bộ nén có hạn chế phù hợp, nó phải giữ một số cấu trúc hữu ích. Nhưng giữ thứ cần tái tạo không đồng nghĩa giữ thứ quan trọng cho cảm xúc. Mạng có thể học độ dài/cách viết thay vì khen/chê.

## 3. Mục đích và vấn đề giải quyết

Trong nghiên cứu khách sạn, có thể thử giảm chiều, denoising, học biểu diễn hoặc phát hiện mẫu tái tạo kém theo giả định rõ. Sau đó có thể clustering/classifier trên code h. Cần đánh giá nhiệm vụ đó riêng; reconstruction tốt không tự xác nhận sentiment hoặc chỉ ra trải nghiệm bất thường thực tế.

## 4. Điều kiện và giả định sử dụng

Cần biểu diễn đầu vào, kiến trúc/loss phù hợp, dữ liệu đủ và train/development riêng khi chọn mô hình. Bottleneck hoặc regularization giúp tránh sao chép vô ích nhưng không bảo đảm feature có ý nghĩa. Với text tokens, decoder/loss khác tái tạo vector TF-IDF; không thay hai bài toán cho nhau.

## 5. Cơ chế hoạt động bên trong

Forward pass tính h=fθ(x), x̂=gψ(h), rồi loss so với target. Backprop tính gradient và optimizer cập nhật tham số encoder/decoder. Lặp trên train, dùng development cho lựa chọn/early stopping khi cần. Ở deploy, giữ weights và preprocessing cố định để encode/reconstruct; fit thêm với dữ liệu mới là cập nhật model khác.

## 6. Quy trình áp dụng trong nghiên cứu

1. Chọn mục tiêu giảm chiều/denoising/anomaly và biểu diễn.
2. Loại trùng, chia dữ liệu phù hợp nhóm/thời gian.
3. Fit scaler/vocabulary trong train.
4. Chọn kích thước code, activation, loss và regularization.
5. Huấn luyện theo protocol, chọn cấu hình bằng development.
6. Đánh giá reconstruction lẫn nhiệm vụ downstream, kiểm tra noise/lệch miền.

Nếu code phục vụ classifier, encoder cũng phải học trong phạm vi train từng fold trừ thiết kế pretrained/transductive đã công bố rõ.

## 7. Ví dụ minh họa xuyên suốt

Ví dụ tự tạo có sáu hàng, bốn feature số, được nén thành hai chiều. Đây là tọa độ minh họa của chỉ báo đã xử lý, không phải bốn khía cạnh khách sạn đã được xác thực. Một lớp ẩn tanh và decoder tuyến tính học giảm MSE. Không có nhãn tích cực/tiêu cực hoặc tên topic trong quá trình fit.

## 8. Kết quả và cách diễn giải

Code h là vector latent, không phải nhãn hay probability. x̂ là tái tạo theo loss; decoder tuyến tính có thể trả số ngoài [0,1] dù dữ liệu train nằm trong khoảng đó. Reconstruction error đo khác biệt số học, không tự là mức bất mãn, mức gian lận hoặc độ chắc chắn của cảm xúc.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là có thể học biểu diễn phi tuyến và thêm ràng buộc phù hợp bài toán. Hạn chế gồm cần tuning/compute, dễ học identity hoặc noise, latent khó diễn giải và có thể mất tín hiệu hiếm. Càng sâu không mặc định tốt hơn; một baseline PCA/LSA hoặc dữ liệu nguyên bản vẫn cần được so.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| PCA/LSA | Tuyến tính theo objective cụ thể; autoencoder có thể phi tuyến |
| NMF | Ràng buộc không âm cho phân rã, khác kiến trúc mạng |
| Classifier | Tối ưu nhãn mục tiêu, không chỉ tái tạo input |
| VAE | Có mô hình latent xác suất và objective bổ sung, không đồng nhất mọi autoencoder |

Một AE tuyến tính trong điều kiện thích hợp có quan hệ với không gian PCA, nhưng không áp kết luận đó cho mọi kiến trúc/loss.

## 11. Sai lầm thường gặp và cách khắc phục

Không gọi error cao là tiêu cực hoặc spam nếu chưa có giả định/ngoại kiểm. Không dùng hidden dimension lớn rồi tái tạo gần hoàn hảo như bằng chứng feature tốt. Không học scaler/encoder trên test rồi báo như pipeline inductive. Đừng dùng binary cross-entropy cho target ngoài phạm vi hoặc decoder không phù hợp mà không kiểm tra.

## 12. Tài liệu tham khảo

- [S1] Goodfellow, I., Bengio, Y., & Courville, A. (2016). *Deep Learning*, Chapter 14: Autoencoders. [Chương từ trang tác giả](https://www.deeplearningbook.org/contents/autoencoders.html).
- [S2] scikit-learn. [Common pitfalls](https://scikit-learn.org/1.8/common_pitfalls.html), đối chiếu phạm vi tiền xử lý.

Nguồn [S1] giải thích họ autoencoder; không gán một năm phát minh duy nhất. Mã NumPy dưới tự soạn cho mạng nhỏ và không phải triển khai một package AE đầy đủ.

## 13. Vị trí trong toàn bộ nghiên cứu

Autoencoder thường là bước representation learning trước mô hình khác. Nếu dùng anomaly score, cần bước xác định đối tượng bất thường và kiểm định độc lập. Nếu dùng latent để xây topic, cần thêm clustering/diễn giải; code h riêng không tự là một bộ topic hoặc thang đo trải nghiệm.

## 14. Kiến thức nền cần biết trước

Cần hiểu mạng feedforward, activation, gradient, backprop, optimizer, MSE và tách dữ liệu. Biết lựa chọn loss theo đầu vào liên tục/count/token. Với văn bản sparse, chuyển sang dense có thể tốn bộ nhớ; không áp trực tiếp ví dụ sáu hàng cho corpus rất lớn mà bỏ qua tài nguyên.

## 15. Nền tảng lý thuyết và logic hình thành

Mục tiêu tái tạo khuyến khích giữ thông tin cần cho decoder. Ràng buộc bottleneck/regularization tạo áp lực học cấu trúc thay vì identity, nhưng cấu trúc đó do loss và dữ liệu quyết định. Denoising buộc tái tạo đầu vào sạch từ bản nhiễu; không phải tự động sửa nhãn cảm xúc sai.

## 16. Các thành phần cấu tạo

Thành phần gồm preprocessing, encoder, latent code, decoder, reconstruction loss, regularization, optimizer và protocol dừng. Bottleneck dimension nhỏ hơn input là undercomplete, nhưng không phải ràng buộc duy nhất. Bias, activation và normalization cũng là cấu hình cần lưu để transform dữ liệu mới đúng.

## 17. Các loại và biến thể

Có undercomplete, sparse, denoising, contractive, sequence autoencoder và VAE với objective riêng. Những biến thể không có cùng loss/diễn giải anomaly. Một sequence AE tái tạo token có khác biệt về tokenizer, teacher forcing và decoder so với AE tái tạo bốn feature số trong ví dụ.

## 18. Đầu vào và đầu ra

Đầu vào có n×d feature hoặc cấu trúc chuỗi phù hợp kiến trúc. Đầu ra gồm code n×r, reconstruction n×d và error theo định nghĩa. Nếu dùng trọng số feature/loss khác nhau, error thay ý nghĩa; phải công bố. Không đọc latent index 0 như topic phòng nếu chưa xác thực.

## 19. Căn cứ lựa chọn

Đáng thử khi có lý do cấu trúc phi tuyến hữu ích và đủ dữ liệu/tài nguyên để chọn mô hình. Chọn bằng downstream performance, reconstruction ngoài mẫu và độ bền theo mục tiêu. So với baseline đơn giản để biết mạng có giá trị thêm, không chỉ vì loss train giảm qua epoch.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu dữ liệu quá ít để đánh giá, cần diễn giải trực tiếp vài feature hoặc không có cơ sở dùng reconstruction cho anomaly. Một AE có thể tái tạo tốt cả trường hợp bất thường; không có bảo đảm error tách hai nhóm. Nếu mục tiêu là sentiment, học tái tạo không thay bộ nhãn hợp lệ.

## 21. Các quyết định quan trọng khi thực hiện

Chọn input scaling, code dimension, activation/decoder, loss, regularization, optimizer, số vòng và early stopping. Chốt dữ liệu dùng chọn threshold nếu anomaly và metric có nhãn ngoại kiểm khi có. Không dùng test để tìm threshold rồi báo đó là khả năng nhận diện độc lập.

## 22. Giải thích công thức và ký hiệu

Với hàng dữ liệu x:

$$h=\tanh(xW_1+b_1),\quad \widehat x=hW_2+b_2,\quad L=\frac1{nd}\sum_{i,j}(x_{ij}-\widehat x_{ij})^2.$$

Ở ví dụ $d=4,r=2$, $W_1$ có kích thước 4×2, $W_2$ 2×4. Gradient loss theo reconstruction là $2(\widehat X-X)/(nd)$. Backprop qua tanh dùng đạo hàm $1-h^2$. Loss giảm chỉ xác nhận tối ưu mục tiêu tái tạo, không chứng minh latent chứa nhãn cảm xúc.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mạng NumPy huấn luyện thật bằng full-batch gradient descent trên feature tự tạo. Số vòng đặt trước để minh họa, chưa có model selection hoặc đánh giá nghiên cứu.

```python
import numpy as np
rng = np.random.default_rng(42)
X = np.array([[1,1,0,0],[0.8,1,0,0.1],[1,0.8,0.1,0],
              [0,0,1,1],[0,0.1,0.8,1],[0.1,0,1,0.8]], dtype=float)
W1 = rng.normal(0,0.1,(4,2)); b1 = np.zeros(2)
W2 = rng.normal(0,0.1,(2,4)); b2 = np.zeros(4)
def forward(A):
    H = np.tanh(A@W1+b1)
    return H, H@W2+b2
initial = np.mean((forward(X)[1]-X)**2)
for epoch in range(2000):
    H, reconstruction = forward(X)
    G = 2*(reconstruction-X)/X.size
    dW2, db2 = H.T@G, G.sum(axis=0)
    GH = (G@W2.T)*(1-H**2)
    dW1, db1 = X.T@GH, GH.sum(axis=0)
    # Tính mọi gradient trước khi cập nhật tham số.
    W1 -= 0.2*dW1; b1 -= 0.2*db1
    W2 -= 0.2*dW2; b2 -= 0.2*db2
H, reconstruction = forward(X)
final = np.mean((reconstruction-X)**2)
assert H.shape == (6,2) and reconstruction.shape == X.shape
assert np.isfinite(reconstruction).all() and final < initial
Xnew = np.array([[0.9,0.9,0,0.1]])
assert forward(Xnew)[0].shape == (1,2)
print("MSE train trước/sau:", initial, final)
print("Code cho mẫu mới:", forward(Xnew)[0])
```

Mã không dùng deep-learning package hoặc nhãn sentiment. Không kết luận hiệu quả giảm chiều/cảm xúc từ loss train; cần train/development/test và đánh giá nhiệm vụ thật khi áp dụng.

## 24. Tình huống sai và phản ví dụ

Một mạng quá lớn có thể sao chép từng review và cho loss thấp nhưng feature kém suy rộng. Một review dài hoặc nhiều từ mới có error cao chỉ vì lệch biểu diễn, không phải khách bất mãn. Một review gian lận gần mẫu train có thể có error thấp. Cần ngoại kiểm thay vì suy ý nghĩa tự động.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Theo dõi loss train/development, gradient/finite values, reconstruction theo feature và độ bền seed. Đo downstream bằng protocol không leakage. Nếu dùng anomaly, kiểm tra threshold trên dữ liệu phát triển và báo precision/recall/coverage phù hợp; không coi error histogram tách đẹp là bằng chứng đầy đủ.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu identity/overfit, xem bottleneck, regularization và dữ liệu; nếu underfit, xem loss/scaling/kiến trúc. Nếu gradient không ổn, kiểm tra learning rate và numerical issues. Khi reconstruction tốt nhưng classifier kém, xem mục tiêu có giữ đúng thông tin cần không; có thể dùng baseline khác thay vì thêm tầng vô hạn.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ là mức tái tạo hoặc chất lượng downstream trên protocol đã nêu. Không suy loss thấp chứng minh hiểu văn bản hoặc anomaly score là sentiment. Không suy latent cause tạo trải nghiệm. Bất kỳ kết luận về khách/nhân viên cần bằng chứng ngoài reconstruction và kiểm soát mục tiêu nghiên cứu.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu huấn luyện AE trên [biểu diễn] với encoder/decoder [...], code dimension [...], loss [...], optimizer [...]. Cấu hình chọn bằng development; encoder được dùng cho [nhiệm vụ] và đánh giá trên [...].” Báo scaling, regularization, early stopping/epoch và baseline. Chỉ đưa số đo thật.

## 29. Khả năng tái lập và tính minh bạch

Lưu scaler/vectorizer, architecture, weights/bias, seed, optimizer/protocol, split ID và phiên bản. Nếu tiếp tục training, lưu checkpoint/trạng thái cần thiết. Mã mục 23 chạy bằng NumPy trên dữ liệu tự tạo; không có đánh giá corpus review thật. Giữ code encode/reconstruct để tránh dùng nhầm latent/decoder.

## 30. Đạo đức nghiên cứu liên quan

Nén không tự ẩn danh; latent/reconstruction có thể giữ thông tin nhạy cảm hoặc ghi nhớ train. Quản lý model và dữ liệu theo quyền truy cập. Không dùng reconstruction error để gán gian lận hoặc phạt nhân viên khi chưa kiểm định và có người xem xét. Tránh công bố mẫu tái tạo có thông tin nhận diện.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Encoder và decoder tối ưu mục tiêu nào?
2. Bottleneck 2 chiều từ input 4 chiều có bảo đảm sentiment không?
3. Vì sao error cao chưa là review tiêu cực?
4. Gradient theo reconstruction của MSE bài là gì?

Gợi ý: 2(X̂−X)/(nd). Bài mở rộng: thiết kế đối chứng PCA/LSA và đánh giá classifier trên latent đúng phạm vi train.

## 32. Những điểm còn tranh luận

Các vấn đề mở gồm sự phù hợp giữa reconstruction và nhiệm vụ, tính diễn giải latent, độ bền anomaly score và nguy cơ ghi nhớ. Không có code dimension/loss tối ưu chung. Nghiên cứu cần báo cả giới hạn và baseline, không dùng tính phi tuyến như bảo đảm chất lượng cao.

