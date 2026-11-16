# Guideline gán nhãn — v0.1-cá nhân — mã 222

- Phiên bản: v0.1-cá-nhân
- Ngày: 2026-11-16
- Người viết: 222
- Loại nhãn: bounding box trên ảnh đường phố; CVAT, COCO 1.0
- Trạng thái: đề xuất cá nhân Pha 1, chưa phải quyết định của nhóm

## 1. Mục tiêu

Tạo nhãn nhất quán cho mô hình phát hiện người và phương tiện trên ảnh giao thông. Ưu tiên không bỏ sót người/phương tiện thật có thể nhận diện; đồng thời không gán nhãn cho tượng, đồ chơi, hình in hay phản chiếu. Chỉ gán theo những gì nhìn thấy trong ảnh, không tưởng tượng phần bị che.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô con, SUV và xe chở khách dạng thân liền; bán tải có khoang hàng riêng xếp vào `truck`. | Xe con trong img_015, SUV/xe con trong img_034. | Xe mô hình trong img_038; xe tải chở ô tô trong img_002. |
| `truck` | Xe có khoang hàng tách biệt hoặc thùng hàng hở; gồm xe tải và bán tải, kể cả bán tải gắn nắp thùng nếu vẫn nhận ra thùng hàng. | Xe trong img_001, img_007, img_012, img_028, img_036, img_037. | Xe khách thân liền khoang thuộc `van`. |
| `van` | Xe thân liền khoang, chở khách/hàng, cỡ nhỏ; minibus tối đa 16 chỗ. | Xe khách nhỏ trong img_016. | Xe buýt lớn trong img_009: ngoài taxonomy. |
| `motorcycle` | Phương tiện có động cơ dạng xe máy/mô-tô hai bánh; xe máy điện dạng scooter cũng tính. | Xe và người đi xe trong img_005, img_006, img_026. | Xe đạp, xích lô và tuk-tuk không xếp vào lớp này. |
| `person` | Người thật nhìn thấy trực tiếp, gồm người đi bộ và người đang lái/ngồi trên phương tiện. Mỗi người một box. | Người bán hàng trong img_001, người đi bộ img_014, người đi xe máy img_005. | Tượng img_017, manocanh img_033, người trong tranh/biển quảng cáo img_018. |
| `ignore_region` | Vùng bỏ qua cho phương tiện có động cơ thật nhưng ngoài taxonomy, hoặc cụm đối tượng thật không thể tách box ổn định. | Tuk-tuk img_004, xe buýt img_009, cụm xe máy đông img_021. | Không dùng cho vật giả, phản chiếu hay để né một vật thuộc lớp có thể nhận diện. |

**Vật ngoài taxonomy:** xe buýt lớn, tuk-tuk và xe ba bánh có động cơ: vẽ một `ignore_region` cho mỗi xe. Xe đạp, xích lô, xe đẩy tay và hàng hóa không có lớp riêng nên không gán box. Nếu có người trên/đẩy các vật này, vẫn gán `person` khi nhìn thấy đủ rõ.

### Thuộc tính

- **`occluded_level`** (ước tính phần trăm diện tích toàn vật bị vật khác che): `0` = 0–5%; `1` = 6–30%; `2` = 31–70%; `3` = trên 70% nhưng vẫn nhận diện chắc chắn. Nếu chỉ còn thấy dưới 10% vật hoặc không thể xác định lớp thì không gán box.
- **`truncated`**: `yes` nếu ước tính từ 10% thân vật trở lên nằm ngoài biên ảnh; ngược lại `no`. Box luôn dừng tại biên ảnh, không kéo ra ngoài.
- **`can_review`**: tick khi chưa chắc lớp, sát ngưỡng thuộc tính, hoặc gặp trường hợp chưa có quyết định chung. Không tick chỉ vì box có thể lệch vài pixel nếu vật đã rõ ràng.

## 3. Quy tắc vẽ box

- **R-01 — Box theo phần nhìn thấy.** Vẽ box sát quanh phần vật nhìn thấy; không ước lượng phần nằm sau vật khác. Nếu vật bị che thành nhiều phần nhưng vẫn là một vật, dùng một box bao các phần nhìn thấy.
- **R-02 — Người và xe máy.** Vẽ một box `motorcycle` cho mỗi xe và một box `person` riêng cho từng người lái/người ngồi sau. Hai loại box được chồng lên nhau. Người ngồi trên xe không tính là vật che chiếc xe đó. Người trên xe đạp/xích lô chỉ có box `person`, không có box cho xe đạp/xích lô.
- **R-03 — Vật bị cắt bởi biên ảnh.** Chỉ gán nếu phần còn lại vẫn đủ nhận diện và box cao ít nhất 20 px. Gán `truncated=yes` khi phần nằm ngoài khung ước tính từ 10% trở lên; phần nằm ngoài khung không được vẽ thêm.
- **R-04 — Xe chở xe và hàng hóa.** Trên xe chuyên chở (img_002), gán một box cho xe vận chuyển; không gán riêng các xe đang được chở. Hàng hóa, ống dài, rau và vật trên thùng xe không có box riêng (img_001, img_003, img_012, img_029). Xe thật đang đỗ/trưng bày nhưng không phải hàng trên xe chuyên chở vẫn có thể là `car`/`truck`; xe mô hình và xe giả không gán.
- **R-05 — Phương tiện ngoài lớp.** Vẽ `ignore_region` riêng, gọn quanh từng xe buýt lớn, tuk-tuk hoặc xe ba bánh có động cơ. Người nhìn thấy trong khoang hở vẫn được gán `person` (img_004, img_009, img_016, img_035, img_040).
- **R-06 — Vật giả và phản chiếu.** Không gán tượng, manocanh, đồ chơi, xe mô hình, hình in/tranh vẽ, bia mộ hình xe hay phản chiếu trên gương/kính/đường ướt. Ví dụ: img_011, img_015, img_017, img_020, img_027, img_033, img_038. Phương tiện thật bên cạnh vật giả vẫn gán bình thường (img_028).
- **R-07 — Người trong xe kín.** Không gán người chỉ nhìn thấy qua cửa kính ô tô/xe tải/van; kính và phản chiếu làm khó xác định biên người. Người ở chỗ hở (xe máy, xe đạp, thùng xe mở, tuk-tuk mở) được gán riêng nếu nhìn thấy rõ.
- **R-08 — Xe chủ và gương.** Không gán phần của chính xe chụp ảnh (gương, mũi xe, tay lái) hoặc cảnh vật chỉ xuất hiện trong gương (img_020).
- **R-09 — Vật nhỏ và cụm đông.** Dùng ngưỡng `min_box_height_px = 20` trong taxonomy. Vật có box thấp hơn 20 px thì không gán riêng; nếu có ít nhất 3 vật nhỏ sát nhau, có thể gom cả cụm vào một `ignore_region`. Không dùng vùng ignore để gom các vật lớn, rõ ràng mà vẫn tách box được.
- **R-10 — Đám đông/xe dày và nhòe.** Gán box riêng cho các vật tiền cảnh nhìn rõ. Nếu từ 5 vật trở lên chen lấn khiến không thể tách ổn định, giữ box cho vật rõ và vẽ `ignore_region` gọn quanh phần còn lại (img_008, img_021, img_030, img_032). Với vệt nhòe, gán riêng nếu vẫn nhận diện được vật; nếu chỉ là vệt không tách được thì dùng `ignore_region` cho cụm xe (img_014).
- **R-11 — Xe có hình dạng lạ.** Nếu vật có kích thước thật, có bánh/thiết kế cho thấy có thể lưu thông và đang tham gia giao thông, tạm xếp vào lớp phù hợp; tick `can_review` nếu chưa chắc (img_019). Chuột bông/đồ trang trí trên xe không có box.
- **R-12 — Ảnh tối và vật bị che.** Chỉ gán khi đủ nhận diện lớp; mức che tính theo vật khác che, không tính phần bị cắt bởi ảnh. Nếu sát mốc 5/30/70% thì chọn mức gần nhất và tick `can_review`. Không suy diễn phần bị che từ hình dáng dự kiến.

## 4. Ca khó quan sát trong bộ ảnh

| Ảnh | Tình huống | Rule | Đề xuất xử lý | Lý do / cần hỏi |
|---|---|---|---|---|
| img_001 | Bán tải và người xung quanh | R-02, R-04 | Bán tải `truck`; người nhìn thấy gán `person`; vật trên thùng không gán riêng | Phân biệt xe và hàng trên xe. |
| img_002 | Xe chuyên chở nhiều ô tô | R-04 | Một `truck`; xe trên giàn là hàng đang vận chuyển, không gán box riêng | Cần nhóm xác nhận quy tắc với xe chuyên chở. |
| img_003 | Xe đạp, người gánh hàng/bán hàng | R-02, R-04 | Chỉ gán người nhìn thấy; không gán xe đạp, hàng hóa hay quang gánh | Không có lớp xe đạp/hàng hóa. |
| img_004, img_016, img_035 | Tuk-tuk và xe ba bánh | R-05 | Mỗi xe một `ignore_region`; gán người ở chỗ hở nếu rõ | Tuk-tuk không có lớp riêng. |
| img_005, img_006, img_026 | Người đi xe máy, có xe bị che | R-02, R-12 | Tách box xe và từng người; không gán xe nếu chỉ còn phần quá nhỏ, không nhận diện được | Box người/xe có thể chồng lên nhau. |
| img_007, img_009, img_037 | Xe tải, xe khách nhỏ/lớn, bán tải nắp cao | R-05 | Xe tải/bán tải `truck`; minibus tới 16 chỗ `van`; buýt lớn `ignore_region` | Nếu không rõ số chỗ/loại thân xe thì tick `can_review`. |
| img_008, img_021, img_030, img_032 | Dòng xe máy/người đông đặc | R-09, R-10 | Box riêng cho đối tượng tiền cảnh rõ; phần quá nhỏ hoặc không tách được gom `ignore_region` | Tránh vẽ một box không phân biệt được từng vật. |
| img_011, img_017, img_027, img_033, img_038 | Tượng, xe đá, manocanh, đồ chơi | R-06 | Không gán box cho vật giả | Đây là vật trưng bày/nhân tạo, không phải đối tượng thật. |
| img_014, img_031 | Vật nhòe/tối và đèn xe | R-10, R-12 | Gán người/phương tiện rõ; vệt nhòe không tách được thì bỏ qua/ignore; không lấy quầng đèn làm box xe | Chỉ gán khi có đủ bằng chứng nhận diện. |
| img_015, img_020 | Ô tô cắt biên, mặt đường ướt và gương | R-03, R-06, R-08 | Ô tô cắt biên gán `truncated` theo ngưỡng; không gán phản chiếu hay vật trong gương | Phản chiếu không phải vật thật trong cảnh. |
| img_018 | Biển quảng cáo và xe sau hàng rào | R-01, R-06 | Không gán người trên biển; gán xe thật nếu đủ nhìn thấy và box cao từ 20 px | Hàng rào là vật che, không phải lý do bỏ qua xe. |
| img_019 | Xe màu vàng đi cùng đám đông và đồ trang trí | R-06, R-11 | Tạm gán xe là `car`, người thật là `person`; không gán chuột bông; tick `can_review` | Cần nhóm xác nhận xe biểu diễn có được tính là xe tham gia giao thông. |
| img_022 | Mô-tô có sidecar trong ảnh đen trắng | R-02, R-12 | Một `motorcycle` gồm cả sidecar; gán `person` nếu thấy rõ | Nếu không chắc đây là xe có động cơ hay xe kéo, tick `can_review`. |
| img_024 | Nhiều xe thật trưng bày, một xe đặt trên nóc xe khác | R-04, R-11 | Gán xe thật có kích thước đầy đủ; xe trên nóc cũng gán nếu là xe thật, không phải mô hình | Ca xe chồng lên nhau cần nhóm chốt cách vẽ box. |
| img_028 | Xe tải/bán tải thật chở tượng ngựa | R-04, R-06 | Gán xe `truck`; không gán tượng ngựa | Vật giả không làm mất nhãn của phương tiện thật. |
| img_029 | Người chở ống dài, xe đạp và xe máy | R-02, R-04 | Gán người và xe máy rõ; không gán xe đạp/ống dài; box người không bao gồm ống | Hàng hóa không thuộc taxonomy. |
| img_034, img_039 | Người/xe xa trong ảnh đường phố | R-01, R-09 | Gán đối tượng rõ, đủ kích thước; vật xa có box dưới 20 px thì bỏ qua | Áp dụng cùng một ngưỡng, không đoán thêm vật quá nhỏ. |
| img_040 | Người sát máy ảnh, taxi và tuk-tuk phía sau | R-03, R-05, R-07 | Gán `person` và xe máy nếu nhận diện được; taxi `car`; tuk-tuk `ignore_region`; không gán tài xế chỉ thấy qua kính | Xe/người bị cắt phải dừng box tại biên ảnh. |

## 5. QC và phiên bản

- Giữ nguyên các ngưỡng trong `submission/taxonomy.json`: kích thước box tối thiểu 20 px, IoU, đồng thuận lớp và tỷ lệ lỗi ảnh. Không tự sửa `rules` khi calibration.
- Dùng các mức `occluded_level` và ngưỡng cắt biên đã nêu để mọi người áp dụng nhất quán; nếu nhóm chọn mốc khác, cập nhật guideline và taxonomy theo quy trình Pha 2.
- Box chưa chắc cần có `can_review`; lỗi cấp ảnh không vẽ được box thì ghi ảnh, vùng và lý do vào `image_issues.csv`.
- Người kiểm QC nên khác người sửa. Kiểm cả box, vật bị bỏ sót và vật gán nhầm; IAA cao không chứng minh cả nhóm đã hiểu đúng quy tắc.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0 → v0.1-cá-nhân | Bổ sung quy tắc cho hàng trên xe, phương tiện ngoài taxonomy, xe chồng lên nhau, vật giả và cụm đông | Các tình huống quan sát được trong img_001–040 chưa được v0 quy định cụ thể |

### Các điểm đề nghị nhóm chốt ở Pha 2

1. Xe trên xe chuyên chở (img_002) là hàng hay cũng gán nhãn `car`?
2. Xe thật trưng bày hoặc xe đặt trên nóc xe khác (img_024) có gán box riêng không?
3. Tuk-tuk/xe ba bánh/buýt lớn dùng `ignore_region` hay cần bổ sung lớp? Không tự thêm lớp vào taxonomy trước khi cả nhóm đồng ý.
4. Xe biểu diễn có hình dạng lạ trong img_019 có tính là `car` nếu có kích thước thật và đang chạy?
5. Ngưỡng gộp cụm đông người/xe cần được thử trên ảnh thực tế để tránh `ignore_region` quá rộng.
