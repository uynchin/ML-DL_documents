# Active Learning — Chọn review để hỏi nhãn thật

> **Nhánh:** MACHINE LEARNING → Ít nhãn. **Nguồn:** dòng 44 bảng Excel. **Đối chiếu:** 29/10/2026. Tập trung pool-based uncertainty sampling. Mã mô phỏng một lượt hỏi oracle bằng nhãn tự tạo, không gửi dữ liệu hay câu hỏi cho người khác. Dữ liệu và tình huống khách sạn đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bối cảnh | Phân tích cảm xúc review khách sạn tiếng Việt |
| Nguồn tín hiệu | Phân biệt gold labels, pseudo/weak labels và target tự tạo trong bài |
| Đánh giá | Cần protocol và dữ liệu độc lập phù hợp, không có benchmark review thật ở ví dụ |

## 1. Định nghĩa học thuật

Active learning là quy trình chọn mẫu cần thêm nhãn dựa trên trạng thái mô hình hoặc tiêu chí thiết kế, với mục tiêu học hiệu quả hơn trong ngân sách gán nhãn. Trong pool-based setting, chọn từ kho U rồi hỏi oracle, thường là người gán nhãn [S1]. Đây không phải tự gán pseudo-label cho mẫu khó.

## 2. Giải thích đơn giản

Thay vì đọc review ngẫu nhiên, bạn ưu tiên một số câu mô hình chưa chắc hoặc có thể bổ sung thông tin. Người gán nhãn trả lời theo codebook, rồi mô hình học lại. Câu khó không luôn là câu hữu ích nhất: có thể là spam, ngoài miền hoặc mơ hồ ngay cả với con người.

## 3. Mục đích và vấn đề giải quyết

Mục tiêu là đạt chất lượng tốt với ít nhãn/chi phí hơn một chiến lược đối chứng phù hợp. Trong review khách sạn, cần so với random sampling cùng ngân sách, cùng seed labels và cùng test. Không chỉ đếm số lượt chọn; thời gian đọc câu dài, giải quyết bất đồng và kiểm tra chất lượng cũng có chi phí.

## 4. Điều kiện và giả định sử dụng

Cần pool hợp lệ, seed labels đủ khởi tạo, oracle/protocol gán nhãn và gold test độc lập. Uncertainty cần có ý nghĩa đủ tốt để xếp mẫu; model có thể tự tin sai về nhóm hiếm. Người gán nhãn không phải nguồn hoàn hảo, nên cần hướng dẫn, abstain và adjudication khi phù hợp.

## 5. Cơ chế hoạt động bên trong

Fit model trên L; tính query score cho U; chọn mẫu/batch theo budget; lấy nhãn oracle; chuyển mẫu khỏi U vào L; refit và lặp. Có thể kết hợp uncertainty với diversity hoặc representativeness. Mẫu test không thuộc pool query, và nhãn chưa được hỏi không được lén dùng để xếp hoặc chọn cấu hình.

## 6. Quy trình áp dụng trong nghiên cứu

1. Chốt codebook, ngân sách và cách tính chi phí.
2. Tách seed L, query pool, development và test.
3. Chọn chiến lược/model theo protocol.
4. Score U, chọn batch và lấy nhãn thật theo quy trình.
5. Kiểm tra nhãn, cập nhật L/U, huấn luyện lại.
6. Theo dõi learning curve theo nhãn/chi phí và so random baseline.

Trong mô phỏng, giấu nhãn pool khỏi selector và chỉ mở nhãn các mẫu được chọn. Không dùng gold test để quyết định câu nào cần hỏi.

## 7. Ví dụ minh họa xuyên suốt

Giả định model có p(positive) của ba câu lần lượt 0.5, 0.9, 0.1. Binary entropy cao nhất ở 0.5 nên uncertainty sampling sẽ ưu tiên câu đầu. Nhãn oracle của câu này có thể là tích cực hoặc tiêu cực; không lấy chính argmax model làm đáp án. Ví dụ chỉ minh họa một lượt, chưa đo tiết kiệm chi phí.

## 8. Kết quả và cách diễn giải

Đầu ra mỗi lượt là ID được hỏi, nhãn mới được xác nhận và model cập nhật. Query score không phải nhãn hay chất lượng khách sạn. Curve chất lượng theo budget cần được đánh giá trên test/protocol cố định, không chỉ độ đúng trên các câu được chọn vì chúng có phân bố lệch theo chiến lược.

## 9. Điểm mạnh, điểm yếu và giới hạn

Ưu điểm là tập trung ngân sách nhãn vào mẫu có tiềm năng bổ sung. Hạn chế gồm selection bias, model khởi đầu kém, batch trùng thông tin, noise/outlier và chi phí phối hợp người gán nhãn. Không có bảo đảm uncertainty sampling thắng random ở mọi giai đoạn hoặc miền.

## 10. So sánh với phương pháp gần nhau

| So với | Khác biệt |
|---|---|
| Self-training | Tự sinh nhãn; active learning hỏi nhãn từ oracle |
| Semi-supervised | Dùng U trong mục tiêu học; active learning chủ yếu quyết định hỏi mẫu nào |
| Random sampling | Đối chứng chọn ngẫu nhiên trong cùng pool/budget |
| Weak supervision | Nguồn gợi ý có thể là luật/metadata thay vì nhãn oracle |

Các cách có thể kết hợp nhưng cần báo tách nguồn nhãn và chi phí.

## 11. Sai lầm thường gặp và cách khắc phục

Không dùng oracle_labels của toàn pool khi chọn mẫu trong mô phỏng. Không so với random có seed labels tệ hơn hoặc budget khác. Không gọi câu khó là câu quan trọng nhất cho quản trị. Không tính metric chỉ trên tập queried rồi suy toàn population; sampling chủ động đã thay phân bố được gán nhãn.

## 12. Tài liệu tham khảo

- [S1] Settles, B. *Active Learning Literature Survey*. University of Wisconsin–Madison, Computer Sciences Technical Report 1648; bản PDF cập nhật 26/01/2010. [Bản từ tác giả](https://burrsettles.com/pub/settles.activelearning.pdf).
- [S2] scikit-learn. [LogisticRegression](https://scikit-learn.org/1.8/modules/generated/sklearn.linear_model.LogisticRegression.html), cho classifier minh họa.

Entropy/query loop dưới tự soạn. Không có khảo sát hay truy vấn thật tới người khác trong ví dụ.

## 13. Vị trí trong toàn bộ nghiên cứu

Active learning nằm ở giao điểm thu thập nhãn và huấn luyện. Nó ảnh hưởng dữ liệu nào được đọc, nên cần ghi thiết kế lấy mẫu và không coi bộ nhãn cuối là mẫu ngẫu nhiên dân số. Nếu muốn ước lượng tỷ lệ cảm xúc toàn corpus, cần đánh giá/thiết kế suy rộng riêng phù hợp.

## 14. Kiến thức nền cần biết trước

Cần hiểu probability, entropy/margin, seed set, oracle, sampling bias và ngân sách. Phân biệt test cố định với pool phát triển. Nếu cùng khách sạn có nhiều câu gần trùng, query strategy có thể tiêu tốn nhiều nhãn vào cùng thông tin; cần xử lý trùng/nhóm và cân nhắc diversity.

## 15. Nền tảng lý thuyết và logic hình thành

Cơ chế dựa trên ý tưởng các nhãn không có giá trị học như nhau ở từng thời điểm. Uncertainty chỉ là một proxy về giá trị thông tin; không chắc tối ưu cải thiện generalization. Các câu mơ hồ không thể gán nhất quán có thể được chọn nhiều, nên tiêu chí và oracle protocol cần làm việc cùng nhau.

## 16. Các thành phần cấu tạo

Thành phần gồm seed L, pool U, model, query strategy, batch size/budget, oracle, xử lý nhãn và quy tắc dừng. Ghi nguồn/độ tin cậy nhãn sau hỏi; có thể cần adjudication. Cập nhật IDs đúng để không hỏi lặp ngoài chủ đích hoặc đánh rơi review chưa được chọn.

## 17. Các loại và biến thể

Có pool-based, stream-based và query synthesis; tiêu chí có uncertainty, query-by-committee, diversity hoặc expected information. Batch mode cần xem dư thừa. Không chuyển công thức entropy đơn lẻ thành lời khẳng định về mọi active learning. Query synthesis còn tạo đầu vào mới nên có yêu cầu hợp lệ khác.

## 18. Đầu vào và đầu ra

Đầu vào gồm L có nhãn, U không được lộ nhãn cho selector và budget. Đầu ra vòng lặp là IDs/nhãn queried cùng model/version. Mô phỏng có một bảng oracle labels giấu trong mã đánh giá, nhưng quyền truy cập selector phải giống tình huống triển khai. Nhãn ngoài pool không được đưa vào training.

## 19. Căn cứ lựa chọn

Đáng thử khi nhãn đắt, có tương tác oracle và đủ khả năng theo dõi quy trình. Căn cứ chọn là learning curve ổn định so random, chất lượng nhãn và chi phí thực. Nếu nhãn có chi phí khác nhau, so theo giờ/tiền phù hợp chứ không chỉ số review.

## 20. Trường hợp không nên sử dụng

Không ưu tiên nếu không thể hỏi thêm nhãn hoặc không có protocol gán đáng tin. Nếu pool quá lệch miền so test, uncertainty không tự sửa nguồn mẫu. Nếu cần corpus gán nhãn đại diện cho ước lượng dân số, phải cân nhắc selection bias thay vì dùng tập queried như mẫu ngẫu nhiên.

## 21. Các quyết định quan trọng khi thực hiện

Chọn seed sampling, query strategy, batch size, có diversity, tie rule, budget và stopping criterion. Chốt khi nào tính cost cho abstain/nhãn bất đồng. Nếu chọn chiến lược sau nhiều lượt nhìn test, đánh giá cuối bị tối ưu gián tiếp; cần development/outer protocol phù hợp.

## 22. Giải thích công thức và ký hiệu

Với classifier có K lớp, uncertainty entropy:

$$H(x)=-\sum_{c=1}^{K}p(c\mid x)\log p(c\mid x),\quad x^*=\arg\max_{x\in U}H(x).$$

Binary p=0.5 cho $H=\log2\approx0.6931$, p=0.9 hoặc 0.1 cho khoảng 0.3251 (log tự nhiên). Sau chọn, hỏi $y^*=oracle(x^*)$; không thay bằng nhãn argmax. Entropy không đo trực tiếp lợi ích kỳ vọng của nhãn hoặc độ mơ hồ của con người.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mô phỏng một lượt hỏi. Nhãn oracle tự tạo chỉ được lấy sau khi chọn; không có test trong pool này.

```python
import numpy as np
from sklearn.pipeline import make_pipeline
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.linear_model import LogisticRegression
seed = ["phòng sạch đẹp", "nhân viên tốt", "phòng bẩn tệ", "nhân viên tệ"]
y_seed = np.array([1,1,0,0])
pool = np.array(["phòng sạch", "phòng bẩn", "phòng không sạch", "nhân viên nhiệt tình"])
oracle_labels = np.array([1,0,0,1])  # chỉ mô phỏng người gán nhãn
pipe = make_pipeline(TfidfVectorizer(ngram_range=(1,2)),
                     LogisticRegression(max_iter=1000, random_state=42))
pipe.fit(seed,y_seed)
P = np.clip(pipe.predict_proba(pool),1e-12,1)
entropy = -(P*np.log(P)).sum(axis=1)
chosen = int(np.argmax(entropy))  # chọn trước khi lấy nhãn oracle
queried_y = int(oracle_labels[chosen])
remaining_ids = np.delete(np.arange(len(pool)), chosen)
pipe.fit(seed + [str(pool[chosen])], np.append(y_seed,queried_y))
assert chosen not in remaining_ids and len(remaining_ids)==len(pool)-1
assert pipe[-1].classes_.tolist()==[0,1]
print("ID được hỏi:", chosen, "câu:", pool[chosen])
print("Entropy:", entropy[chosen], "nhãn oracle:", queried_y)
p = np.array([0.5,0.9,0.1])
H = -p*np.log(p)-(1-p)*np.log(1-p)
assert int(np.argmax(H))==0 and np.isclose(H[0],np.log(2))
```

Chưa có learning curve hoặc bằng chứng tiết kiệm nhãn. Vocabulary ví dụ được fit trên seed rồi fit lại trên L mở rộng; nếu dùng pool để học biểu diễn, phải công bố thiết kế đó.

## 24. Tình huống sai và phản ví dụ

Model có thể rất tự tin sai với một kiểu mỉa mai hiếm nên uncertainty không bao giờ hỏi các câu đó. Ngược lại, nó có thể liên tục chọn câu spam/không đủ ngữ cảnh. Hỏi nhiều câu khó chưa chắc giúp; cần diversity, kiểm tra miền và protocol để oracle abstain hợp lệ.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Lặp seed/pool split và random baseline cùng budget; báo mean/uncertainty learning curve. Kiểm tra nhãn theo nhóm, chi phí và mức bất đồng oracle. Không chọn chiến lược theo đường test tốt nhất rồi chỉ báo đường đó. Nếu so thời gian, tính cả việc tạo score và chờ/kiểm tra nhãn.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu không vượt random, xem seed quá nghèo, uncertainty kém hoặc pool dư thừa. Có thể thêm exploration/diversity hoặc chỉnh batch theo development. Nếu câu không gán được, dùng quy trình abstain/adjudication, không ép oracle chọn cho đủ số nhãn. Giữ test độc lập sau mọi thay đổi.

## 27. Phân biệt kết quả với kết luận

Kết quả là chất lượng theo budget trong thiết kế cụ thể. Có thể kết luận tiết kiệm nhãn khi đạt cùng mục tiêu với ít chi phí hơn đối chứng và có độ bền. Không suy queried set đại diện tự nhiên cho dân số hoặc câu được hỏi là vấn đề khách sạn quan trọng nhất.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu dùng pool-based active learning với seed [...], query [...], batch/budget [...]. Nhãn được gán bằng [protocol], có xử lý bất đồng [...]. So với random cùng budget trên test giữ lại [...].” Báo số lần chạy, cách tính cost và stopping. Không đưa số nhãn mô phỏng thành số người tham gia thật.

## 29. Khả năng tái lập và tính minh bạch

Lưu thứ tự query, IDs pool, scores, nhãn oracle, thời gian/chi phí và model từng giai đoạn cần thiết. Giữ mapping giữa nhãn được hỏi và nhãn giấu trong mô phỏng. Mã đã chạy sklearn 1.8.0 trên dữ liệu tự tạo; không có truy vấn thật tới người khác.

## 30. Đạo đức nghiên cứu liên quan

Người gán nhãn cần hướng dẫn và điều kiện làm việc phù hợp; câu khó/nhạy cảm có thể tăng gánh nặng. Bảo vệ dữ liệu cá nhân của review, tránh xuất nguyên văn nhạy cảm cho nguồn không được phép. Selection bias có thể bỏ qua nhóm thiểu số nên cần audit phù hợp.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Vì sao entropy cao không bảo đảm câu hữu ích nhất?
2. Active learning lấy nhãn mới từ đâu?
3. Vì sao test không được là query pool?
4. Tại p=0.5 entropy bao nhiêu?

Gợi ý: ln2 với log tự nhiên. Bài mở rộng: thiết kế so random và uncertainty cùng budget/seed, tính cả chi phí nhãn bất đồng.

## 32. Những điểm còn tranh luận

Các vấn đề mở gồm cost-sensitive querying, nhóm hiếm, model tự tin sai và tái lập oracle thực tế. Không có chiến lược query luôn thắng. Nghiên cứu cần báo đường cong theo ngân sách và chất lượng gán nhãn, thay vì chỉ số cuối sau một lần chọn seed thuận lợi.

