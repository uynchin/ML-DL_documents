# CatBoost — Boosting với xử lý biến phân loại và ordered learning

> **Nhánh:** MACHINE LEARNING → Supervised: Boosting. **Nguồn:** dòng 27 bảng Excel. **Đối chiếu:** 28/10/2026. Phân biệt ordered target statistics với ordered boosting. Ví dụ Python mô phỏng thống kê theo thứ tự, không triển khai đầy đủ CatBoost và chưa chạy package catboost. Dữ liệu và tình huống khách sạn trong ví dụ đều tự tạo.

| Trường | Nội dung |
|---|---|
| Bản chất | Hệ thống gradient boosting với cơ chế xử lý categorical và prediction shift |
| Mốc tham khảo | Prokhorenkova và cộng sự, NeurIPS 2018; preprint xuất hiện 2017 [S1] |
| Đầu vào | Đặc trưng số/categorical khai báo đúng; text cần cấu hình riêng |
| Đầu ra | Raw score, xác suất/nhãn và thông tin mô hình |
| Điểm cần tách | Ordered target statistics khác ordered boosting |
| Phần mềm | catboost; mã cơ chế minh họa dùng thư viện chuẩn Python |

## 1. Định nghĩa học thuật

CatBoost là một hệ thống gradient boosting nổi bật với các cơ chế xử lý biến phân loại và giảm một dạng prediction shift liên quan cách dùng nhãn trong quá trình học. Bài gốc trình bày ordered boosting và cách xử lý categorical dựa trên thứ tự [S1].

Không định nghĩa nó đơn thuần là “XGBoost có thêm category”. Cũng không đồng nhất ordered boosting với phép mã hóa category hoặc coi mọi lần chạy thư viện đều mặc định cùng một chế độ học.

## 2. Giải thích đơn giản

Giả sử muốn biến “loại phòng” thành một con số có ích. Nếu lấy cảm xúc trung bình của toàn bộ các mẫu cùng loại rồi dùng ngay để dự đoán từng mẫu trong đó, nhãn của chính mẫu đã lọt vào đầu vào.

Một ý tưởng ordered là với mẫu đang xét, chỉ dùng nhãn của những mẫu đứng trước nó trong một thứ tự xác định và thêm thông tin prior. Nhãn hiện tại không được tự trả lời câu hỏi của chính nó. Đây mới là phần thống kê, chưa mô tả toàn bộ ordered boosting.

## 3. Mục đích và vấn đề giải quyết

Có thể dùng khi dữ liệu kết hợp chỉ báo review với categorical có ý nghĩa như loại phòng hoặc kênh đặt phòng, và các biến đó thực sự có sẵn ở dự đoán. Nó là ứng viên để kiểm tra lợi ích của xử lý categorical trong protocol nghiên cứu.

Không nên biến mỗi câu review thành một category duy nhất để mong mô hình hiểu câu. Văn bản có cấu trúc ngôn ngữ; cần vector hóa hoặc cấu hình xử lý text riêng, và kiểm tra chất lượng với tiếng Việt.

## 4. Điều kiện và giả định sử dụng

Cần khai báo đúng feature categorical, xử lý giá trị thiếu/kiểu dữ liệu nhất quán và xác định mục tiêu dự đoán. Category phải giữ nghĩa giữa train và test; mã nhãn tùy ý không biến thành đại lượng thứ bậc.

Cơ chế ordered nội bộ không thay thế train/test split bên ngoài. Nếu dự đoán khách sạn mới hoặc tương lai, vẫn cần giữ nhóm/thời gian. Random permutation phục vụ học không đồng nghĩa thiết kế đánh giá có thứ tự thời gian hợp lệ.

## 5. Cơ chế hoạt động bên trong

Ở mức khái niệm, ordered target statistics tính một biểu diễn category từ các nhãn có trước mẫu trong permutation và prior. Thư viện thực tế có nhiều loại thống kê, lượng hóa và tổ hợp feature; ví dụ trung bình nhị phân trong bài chỉ minh họa nguyên tắc [S2].

Ordered boosting xử lý vấn đề khác: xây các ước lượng phục vụ gradient sao cho mẫu không bị dùng một cách gây lệch như trong cơ chế mà bài gốc phân tích. Đây không phải chỉ tính target mean rồi chạy một bộ GBDT thông thường. Cây đối xứng là lựa chọn mặc định phổ biến, nhưng có grow policy khác.

## 6. Quy trình áp dụng trong nghiên cứu

1. Định nghĩa nhiệm vụ và biến có sẵn tại thời điểm dự đoán.
2. Loại trùng và chia train/development/test theo đơn vị suy rộng.
3. Định nghĩa schema số/categorical/text; giữ pipeline tiền xử lý trong train.
4. Chọn objective, chế độ boosting, grow policy và cấu hình categorical/text có lý do.
5. Tuning số vòng, learning rate, depth và regularization bằng development.
6. Dùng development cho early stopping, khóa model trước đánh giá test.
7. Kiểm tra category chưa gặp, độ bền chuyển miền và lỗi theo nhóm.

Nếu tự mã hóa target statistics bên ngoài thư viện, phải làm đúng trong từng fold bằng cách cross-fit/ordered phù hợp. Không tính thống kê nhãn trên toàn bộ dữ liệu rồi chia fold.

## 7. Ví dụ minh họa xuyên suốt

Dùng category tự tạo A, A, B, A theo thứ tự bốn review; nhãn lần lượt 1, 0, 1, 1. Với prior 0.5 và hệ số làm trơn a=1, thống kê trước khi đọc nhãn hiện tại là 0.5, 0.75, 0.5, 0.5.

A có thể tượng trưng một loại phòng, không phải ID người viết. Dãy số minh họa việc cùng category có thể có biểu diễn khác nhau tùy phần lịch sử đứng trước trong train. Nó không phải xác suất đầu ra của CatBoost.

## 8. Kết quả và cách diễn giải

Xác suất đầu ra dự đoán nhãn được định nghĩa trong nghiên cứu, không phải xác suất khách hàng thực sự hài lòng theo một cấu trúc tiềm ẩn đã xác thực. Cần kiểm tra calibration nếu dùng score để ra quyết định.

Importance hoặc giải thích cục bộ mô tả cách model sử dụng biểu diễn. Một category quan trọng có thể phản ánh đặc điểm chọn mẫu, không phải lý do gây cảm xúc. Tách tác dụng dự đoán khỏi nhận định quản trị.

## 9. Điểm mạnh, điểm yếu và giới hạn

Điểm mạnh là có cơ chế tích hợp cho categorical và các lựa chọn giảm lệch trong học boosting, giúp tránh một số cách target encoding ngây thơ. Khi schema phù hợp, có thể giảm khối lượng mã hóa thủ công.

Giới hạn là vẫn cần nhãn tốt, split đúng, cấu hình và ngân sách. High-cardinality category có thể trở thành shortcut về danh tính/miền. CatBoost không tự sửa leakage do điểm sao tạo nhãn, review trùng giữa train/test hoặc feature chỉ có sau thời điểm dự đoán.

## 10. So sánh với phương pháp gần nhau

| Khái niệm/phương pháp | Phân biệt |
|---|---|
| One-hot | Biểu diễn danh tính category, không lấy trung bình nhãn |
| Target encoding toàn bộ train | Có thể chứa nhãn chính mẫu nếu làm ngây thơ |
| Ordered target statistics | Dùng phần đứng trước và prior cho phép mã hóa minh họa |
| Ordered boosting | Cơ chế học boosting, khác bước mã hóa category |
| XGBoost/LightGBM | Cũng có hỗ trợ categorical theo triển khai; không coi CatBoost là lựa chọn duy nhất |

So sánh cần nêu biểu diễn và cấu hình cho mỗi mô hình, không gán lợi ích của feature mới cho riêng thuật toán.

## 11. Sai lầm thường gặp và cách khắc phục

Sai lầm là coi Ordered luôn là mặc định cho mọi CPU/GPU và mọi cỡ dữ liệu. Tài liệu có điều kiện mặc định theo thiết bị/bài toán; ghi boosting_type thực dùng thay vì suy từ tên CatBoost [S3].

Đừng cho ID người viết/khách sạn vào mô hình rồi chỉ test ngẫu nhiên nếu mục tiêu là nhóm chưa gặp. Đừng gọi một công thức target mean là triển khai đầy đủ CTR của thư viện; đó chỉ là minh họa một nguyên lý.

## 12. Tài liệu tham khảo

- [S1] Prokhorenkova, L., Gusev, G., Vorobev, A., Dorogush, A. V., & Gulin, A. (2018). *CatBoost: unbiased boosting with categorical features*. NeurIPS. [Bài hội nghị](https://proceedings.neurips.cc/paper_files/paper/2018/hash/14491b756b3a51daac41c24863285549-Abstract.html), [preprint](https://arxiv.org/abs/1706.09516).
- [S2] CatBoost. [Transforming categorical features to numerical features](https://catboost.ai/docs/en/concepts/algorithm-main-stages_cat-to-numberic).
- [S3] CatBoost. [Common parameters](https://catboost.ai/docs/en/references/training-parameters/common).
- [S4] CatBoost. [Text processing parameters](https://catboost.ai/docs/en/references/training-parameters/text-processing).

Tách mốc preprint 2017 với bài hội nghị 2018. Từ “unbiased” trong tiêu đề không phải bảo đảm mọi thống kê đầu ra không chệch hoặc mọi kết luận xã hội công bằng.

## 13. Vị trí trong toàn bộ nghiên cứu

CatBoost nằm ở khâu học mô hình dự đoán sau khi xác định câu hỏi, nguồn review, nhãn và schema. Nếu categorical là nội dung nghiên cứu, cần giải thích lý do đưa từng biến vào và khả năng có sẵn khi áp dụng.

Việc tổng hợp dự đoán thành chỉ số khách sạn đòi hỏi đánh giá sai số và đại diện mẫu. Cơ chế xử lý categorical không xác nhận tính hợp lệ của thang đo hay lý thuyết về sự hài lòng.

## 14. Kiến thức nền cần biết trước

Cần hiểu cây boosting, categorical so với ordinal, target encoding, smoothing prior và leakage. Kiến thức permutation và cross-fitting giúp phân biệt ý nghĩa thứ tự học với thời gian thực của dữ liệu.

Trước khi học công thức, hãy tự hỏi một feature có sử dụng nhãn chính mẫu hoặc nhãn test không. Với nghiên cứu dự đoán tương lai, còn phải kiểm tra có dùng dữ liệu tương lai để xây một feature “lịch sử” hay không.

## 15. Nền tảng lý thuyết và logic hình thành

Ý tưởng trọng tâm là kiểm soát sự phụ thuộc giữa giá trị học từ nhãn và mẫu đang được đánh giá trong quá trình học. Một thống kê có chứa nhãn chính mẫu có thể có hành vi khác lúc train và lúc gặp mẫu mới.

Dùng các phần dữ liệu đứng trước trong thứ tự là cách tổ chức thông tin để giảm vấn đề này theo cơ chế của phương pháp. Tuy nhiên, prior, số mẫu mỗi category và cách chia dữ liệu bên ngoài vẫn ảnh hưởng sai số và khả năng suy rộng.

## 16. Các thành phần cấu tạo

Các thành phần gồm schema dữ liệu, biến số/categorical/text, cơ chế thống kê categorical, permutation khi áp dụng, chế độ boosting, cấu trúc cây, loss, regularization và tiêu chí dừng. Các tổ hợp category có thể làm biểu diễn phức tạp hơn.

Cây đối xứng dùng cùng phép chia ở các nút của cùng độ sâu, tạo cấu trúc đều. Không từ đó suy ra model chỉ có một quy tắc hoặc dễ đọc hoàn toàn; nhiều cây và thống kê nội bộ vẫn có thể khó diễn giải.

## 17. Các loại và biến thể

Có Plain và Ordered boosting theo hỗ trợ/cấu hình; grow policy có thể là SymmetricTree hoặc các lựa chọn khác. CPU/GPU không hoàn toàn có cùng mặc định và giới hạn. Vì vậy cần báo tham số đã chọn và phiên bản.

Categorical có thể được xử lý bằng one-hot trong một số điều kiện hoặc thống kê khác. Text là loại feature riêng với tokenizer, dictionary và feature calcers; khả năng hỗ trợ text không có nghĩa mặc định đã phù hợp tiếng Việt.

## 18. Đầu vào và đầu ra

Đầu vào là bảng với kiểu cột được xác định rõ và nhãn; có thể dùng Pool để quản lý metadata. Tên category cần chuẩn hóa nhất quán, không gộp nhầm khác biệt quan trọng hoặc để cùng một loại thành nhiều cách viết không chủ đích.

Đầu ra gồm raw score/xác suất/nhãn tùy lời gọi và objective. Biểu diễn thống kê category nội bộ không phải biến kết quả mới của nghiên cứu. Category chưa gặp cần được kiểm tra qua cơ chế fallback thực tế của cấu hình.

## 19. Căn cứ lựa chọn

Nên thử khi có categorical hữu ích, có yêu cầu so sánh cách biểu diễn và đủ nhãn để đánh giá. Lựa chọn cuối dựa trên kết quả ngoài mẫu, lỗi quan trọng, chi phí và tính minh bạch của schema.

Một thiết kế có ích so CatBoost với baseline mã hóa categorical rõ ràng, cùng split/feature được phép. Nếu chỉ một mô hình được thêm metadata giàu thông tin, không thể quy mọi chênh lệch cho thuật toán.

## 20. Trường hợp không nên sử dụng

Không nên dùng tên CatBoost làm lý do duy nhất khi dữ liệu chỉ là văn bản thô và chưa có kế hoạch biểu diễn. Không ưu tiên categorical định danh cá nhân hoặc proxy miền nếu mục tiêu là suy rộng sang nhóm mới mà không có kiểm tra thích hợp.

Nếu nhãn không ổn định hoặc định nghĩa cảm xúc mơ hồ, ordered learning không sửa được. Nếu cần ước lượng nhân quả, classifier cùng categorical không tạo ra thiết kế nhận diện tác động.

## 21. Các quyết định quan trọng khi thực hiện

Quyết định quan trọng gồm dùng category nào, xử lý missing/nhãn hiếm, prior/cấu hình thống kê khi can thiệp, chế độ boosting, grow policy, depth, learning rate/số vòng và regularization. Chọn metric theo mục tiêu lớp hiếm nếu cần.

Với dữ liệu có thời gian, phân biệt thứ tự phục vụ học với giới hạn thông tin ở thời điểm dự đoán. Những tùy chọn như has_time phải được hiểu theo tài liệu; không thay thế đánh giá ngoài thời gian hoặc phép xây feature lịch sử đúng.

## 22. Giải thích công thức và ký hiệu

Ví dụ target statistic nhị phân đơn giản, không phải toàn bộ CTR của CatBoost:

$$TS_i=\frac{\sum_{j\prec i,\ c_j=c_i}y_j+a p_0}{n_{i,c_i}^{prev}+a}.$$

$c_i$ là category; $j\prec i$ là đứng trước trong thứ tự; $n^{prev}$ là số mẫu cùng category đứng trước; $p_0$ là prior; $a>0$ là độ mạnh làm trơn. Không có nhãn hiện tại $y_i$ trong tử số.

Với A,A,B,A; y=1,0,1,1; $p_0=0.5,a=1$: lần lượt $0.5,\ (1+0.5)/2=0.75,\ 0.5,\ (1+0+0.5)/3=0.5$. Prior ở đây đặt trước để minh họa. Nếu ước lượng prior từ dữ liệu, cũng phải xác định phạm vi thông tin và protocol phù hợp.

## 23. Ví dụ tính toán hoặc thao tác từng bước

Mã chạy bằng Python chuẩn, kiểm tra “mã hóa trước, cập nhật nhãn sau”. Không gọi catboost và không kiểm chứng ordered boosting, text processing hoặc CTR thực tế của thư viện.

```python
from collections import defaultdict
from math import isclose

def ordered_mean(cats, labels, prior=0.5, strength=1.0):
    assert strength > 0 and len(cats) == len(labels)
    sums, counts = defaultdict(float), defaultdict(int)
    result = []
    for c, y in zip(cats, labels):
        result.append((sums[c] + strength*prior) / (counts[c]+strength))
        sums[c] += y
        counts[c] += 1
    return result

cats = ["A", "A", "B", "A"]
y = [1, 0, 1, 1]
encoded = ordered_mean(cats, y)
expected = [0.5, 0.75, 0.5, 0.5]
assert all(isclose(a, b) for a, b in zip(encoded, expected))
changed = ordered_mean(cats, [1, 1, 1, 1])
assert changed[1] == encoded[1]  # đổi nhãn hiện tại không đổi TS hiện tại
assert isclose(changed[3], 2.5/3)  # nhưng đổi lịch sử của mẫu sau
assert changed[3] != encoded[3]
print("Thống kê ordered minh họa:", encoded)
print("Sau đổi nhãn mẫu thứ hai:", changed)
```

Kết quả cho thấy một thuộc tính của công thức với prior cố định, không chứng minh toàn bộ pipeline không leakage. Còn phải kiểm tra cách tạo category, prior, split, dữ liệu trùng và các feature khác.

## 24. Tình huống sai và phản ví dụ

Phản ví dụ: mỗi ID review xuất hiện đúng một lần và dùng target mean trên toàn train không smoothing. Feature thu được bằng chính nhãn mẫu, nên mô hình train gần hoàn hảo nhưng không có thông tin tương ứng khi gặp ID mới.

Phản ví dụ khác: giữ nguyên người viết ở cả train/test dù đích ứng dụng là người mới. Cơ chế ordered nội bộ không loại bỏ sự phụ thuộc giữa hai tập; split theo người viết có thể cần thiết nếu được phép dùng và có dữ liệu phù hợp.

## 25. Kiểm tra chất lượng và độ vững của kết quả

Kiểm tra nhóm category hiếm, category chưa gặp, các giá trị thiếu và đổi miền/thời gian. Đo hiệu năng khi loại các category có nguy cơ làm shortcut; mọi quyết định lựa chọn dựa trên development.

Nếu nghiên cứu lợi ích ordered boosting, so cấu hình Plain/Ordered trong điều kiện có kiểm soát, không thay đồng thời schema và độ sâu. Kiểm tra calibration và mức biến động seed/split; đừng dùng tên thuật toán thay bằng chứng ổn định.

## 26. Xử lý khi không đáp ứng yêu cầu

Nếu lỗi tập trung ở category hiếm, kiểm tra chuẩn hóa tên, schema và độ phụ thuộc vào category. Có thể điều chỉnh cấu hình, gộp nhóm theo cơ sở chuyên môn hoặc bỏ feature định danh, nhưng không gộp bằng cách nhìn nhãn test.

Nếu văn bản mới là nguồn thông tin chính, cải thiện biểu diễn và kiểm tra phủ định/khía cạnh. Nếu train tốt mà test miền mới kém, xem shortcut và thiết kế split trước khi tăng số cây. Chuyển mô hình cần được đánh giá lại đúng protocol.

## 27. Phân biệt kết quả với kết luận

Kết quả hợp lệ: “Mô hình với schema X đạt metric Y trên nhóm giữ lại Z.” Có thể nói categorical bổ sung có giá trị dự đoán khi đối chứng/ablation phù hợp, kèm độ bất định.

Không nói một loại phòng “gây” cảm xúc tiêu cực từ importance. Không nói CatBoost bảo đảm không thiên lệch xã hội vì tiêu đề bài gốc có chữ unbiased. Khái niệm kỹ thuật về prediction shift khác tính công bằng giữa các nhóm người.

## 28. Cách viết vào bài nghiên cứu

Mẫu viết: “Nghiên cứu dùng CatBoost với [loss], [boosting_type], [grow_policy]. Các cột categorical gồm [...], được xác định trước. Cấu hình được chọn bằng [development/CV theo nhóm], test giữ lại [đơn vị].”

Nếu dùng text native, báo tokenizer, dictionary, feature calcers và kiểm tra tiếng Việt. Nếu dùng vector bên ngoài, báo pipeline đó. Ghi rõ thiết bị/phiên bản vì mặc định thay đổi theo bối cảnh; không chỉ ghi ‘dùng tham số mặc định’ mà bỏ cấu hình.

## 29. Khả năng tái lập và tính minh bạch

Lưu schema cột và kiểu, mapping nhãn, split ID, cấu hình/category transformations, seed, phiên bản, thiết bị và model. Nếu có biến đổi ngoài thư viện, lưu cả bộ biến đổi, không chỉ file model CatBoost.

Mục 23 kiểm tra nguyên lý thống kê bằng Python chuẩn; chưa chạy package catboost. Thực nghiệm bổ sung cần script đầy đủ, điều kiện môi trường và kết quả thật. Dữ liệu công khai phải được ẩn danh/cho phép chia sẻ.

## 30. Đạo đức nghiên cứu liên quan

Category có thể là proxy cho danh tính hoặc nhóm nhạy cảm. Cần cân nhắc mục đích, quyền sử dụng, khả năng tái nhận diện và ảnh hưởng của sai phân loại, đặc biệt khi dữ liệu dùng để đánh giá nhân viên.

Không công khai bảng category quá nhỏ kèm review có thể truy ngược. Đánh giá lỗi theo nhóm chỉ khi có cơ sở thu thập và dùng dữ liệu phù hợp; không suy đoán đặc điểm nhạy cảm của người viết từ văn bản.

## 31. Câu hỏi tự kiểm tra và bài tập vận dụng

1. Vì sao target mean có thể lộ nhãn chính mẫu?
2. Tính thống kê của mẫu A thứ hai nếu prior=0.5, a=2 và nhãn A trước đó là 1.
3. Ordered target statistics khác ordered boosting ở điểm nào?
4. Thiết kế test cho khách sạn chưa gặp và category chưa gặp.

Gợi ý câu 2: (1+2×0.5)/(1+2)=2/3. Bài mở rộng: thay thứ tự mẫu trong mã và giải thích vì sao biểu diễn train thay đổi; điều đó không đồng nghĩa trật tự thời gian thật đã được tôn trọng.

## 32. Những điểm còn tranh luận

Các điểm cần kiểm chứng là mức lợi ích của cơ chế ordered trong từng cỡ dữ liệu, vai trò của categorical so với văn bản và khả năng chuyển sang category/miền mới. Không thể kết luận từ duy nhất công thức minh họa.

Một đánh đổi khác là tiện lợi của xử lý tích hợp với khả năng kiểm toán các biến đổi nội bộ. Nghiên cứu nên báo đủ schema và cấu hình để người đọc hiểu nguồn thông tin, nguy cơ shortcut và giới hạn của giải thích.

