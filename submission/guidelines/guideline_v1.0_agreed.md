# Guideline gán nhãn — v1.0 (thống nhất nhóm G0001)

> **Trạng thái:** đã khóa cho lượt gán độc lập (Pha 3). Mọi thay đổi sau đây đi qua calibration (Pha 5) và tăng phiên bản.

- Phiên bản: v1.0
- Ngày: 2026-11-16
- Người viết / duyệt: 111, 222, 333 (điều phối: 111)
- Loại nhãn: bounding box trên phần nhìn thấy của vật; CVAT, xuất COCO 1.0
- Taxonomy đi kèm: `submission/taxonomy.json` (version `v1.0`)

## 0. Cách chốt các điểm ba bản cá nhân khác nhau

Ba bản giống nhau ở: bán tải là `truck`; xe có sidecar là một `motorcycle`; xe đạp / xích lô không gán; tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu không gán; người đi xe máy tách thành box xe và box từng người, được chồng nhau; ngưỡng 20 px; xe giả tưởng chạy được (img_019) tạm là `car` kèm `can_review`; người ngồi ở chỗ hở của tuk-tuk vẫn gán `person`. Chỗ khác nhau:

| # | Chủ đề | 111 | 222 | 333 | **Chốt v1.0** | Lý do |
|---|---|---|---|---|---|---|
| 1 | Xe buýt, tuk-tuk, xe ba bánh | mỗi xe một `ignore_region` | giống 111 | không gán, **không** dùng ignore | **Mỗi xe một `ignore_region`** | 2/3 bản; xe này là vật thật, để trống thì mô hình bị phạt khi nhận ra nó. Phần ba bản 333 lo `ignore_region` bị lạm dụng: đã siết ở R-12 và định nghĩa lớp. |
| 2 | Xe được chở trên xe tải (img_002) | chỉ box `truck` | giống 111 | thêm một `car` cho mỗi xe chở | **Chỉ một box `truck`** | 2/3 bản; tránh 7–8 box bị che nặng, vốn là nguồn lệch IAA lớn. |
| 3 | Xe vàng trên nóc xe (img_024) | không box | gán nếu là xe thật | không gán (prop) | **Không box riêng** | cùng nguyên tắc xe được chở (R-05); 111 và 333 trùng kết quả. |
| 4 | Người nhìn qua kính xe kín | không gán | không gán | gán nếu thấy chắc, ≥ 20 px | **Không gán** (R-07) | 2/3 bản; biên người qua kính không đáng tin, dễ lệch. |
| 5 | Ngưỡng `occluded_level` | 5 / 30 / 70% | 5 / 30 / 70% | 10 / 35 / 70% | **≤5 / 6–30 / 31–70 / >70%** | 2/3 bản và khớp gợi ý của đề lab. |
| 6 | `truncated` | `yes` khi ≥ 10% ngoài khung | giống 111 | `yes` khi bị mép cắt (mọi mức) | **`yes` khi ≥ 10% thân ngoài khung** | 2/3 bản. Rủi ro: ước lượng 10% chủ quan; xem lại ở Pha 4 nếu lệch. |
| 7 | Vệt nhoè chuyển động | `ignore_region` khi chỉ còn vệt | giống 111 | không ignore; ghi `image_issues.csv` | **`ignore_region`** nếu chỉ còn vệt (R-11) | 2/3 bản; vẫn có đường ghi ca không vẽ được. |
| 8 | Từ bao nhiêu vật thì sang `ignore_region` | ≥ 6 vật chồng | ≥ 5 vật chồng | không nêu số | **≥ 5 vật cùng lớp chồng sát** (R-10) | lấy mốc thấp hơn của hai bản có số; sẽ thử lại ở Pha 4. |
| 9 | Người thấy từ bao nhiêu thì gán | ≥ 30% thân | "thấy rõ" | ≥ 20 px, chắc chắn | **cao ≥ 20 px và thấy ≥ 30% thân** (R-07, R-10) | có số đo cụ thể, không dựa cảm tính. |
| 10 | Cụm vật nhỏ | ≥ 3 vật nhỏ → một vùng | "có thể" gom | chỉ ignore khi dày | **≥ 3 vật cao < 20 px sát nhau → một `ignore_region`** (R-09) | 111 cụ thể nhất; đơn lẻ thì bỏ qua. |

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** (`car`, `truck`, `van`, `motorcycle`, `person`) trên ảnh đường phố 2D, cho camera giao thông và hỗ trợ nhận biết vật cản. Nguy hiểm nhất là **bỏ sót người và xe thật**; thứ hai là báo nhầm **vật không thật** (tượng, ma-nơ-canh, hình in, phản chiếu, đồ chơi) thành người hoặc xe. Nguyên tắc chung: vật thật nhìn thấy trực tiếp thì có box; vật không thật thì không có box; chỗ nào không chắc thì tick `can_review` thay vì đoán.

Mục tiêu của guideline là để hai người độc lập nhìn cùng một ảnh vẫn ra cùng quyết định về vật nào gán, lớp nào, box bao phần nào, và ba thuộc tính.

## 2. Lớp và thuộc tính

### 2.1. Lớp (mỗi vật thuộc đúng một lớp, đúng một box)

| Lớp | Định nghĩa (một câu) | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô con thân kín, từ 9 chỗ trở xuống: sedan, hatchback, wagon, SUV, MPV, taxi 4 bánh; xe thật đang đỗ hoặc trưng bày vẫn gán. | img_023, img_015, img_012 (SUV), img_040 (taxi), img_024 (xe thật trong showroom) | Bán tải là `truck`; xe đồ chơi (img_038), xe tạc đá (img_027) không phải xe. |
| `truck` | Xe có khoang chở hàng tách khỏi cabin hoặc để hở: tải nhỏ, tải thùng, bán tải (kể cả có nắp thùng), đầu kéo kèm rơ-moóc. | img_001, img_007, img_012, img_028, img_036, img_037, img_002 | Xe thân liền khối kín là `van` (img_016). |
| `van` | Xe thân liền khối kín (khoang người/hàng liền cabin), cỡ nhỏ: panel van hoặc xe khách tối đa 16 chỗ. | img_005, img_016 | Xe khách > 16 chỗ (img_009) ngoài taxonomy; SUV/MPV dáng ô tô là `car`. |
| `motorcycle` | Xe hai bánh có động cơ: xe máy, tay ga, moped, mô tô, xe máy điện dạng tay ga; xe gắn sidecar là **một** xe, một box. | img_006, img_008, img_026, img_022 | Xe đạp, xe đạp thồ, xích lô (img_003, img_023); tuk-tuk. |
| `person` | Người thật nhìn thấy trực tiếp: đi bộ, đứng, ngồi, đi xe máy/xe đạp/xích lô, ngồi ở chỗ hở (thùng xe, tuk-tuk). | img_014, img_034, img_005, img_010, img_026 | Tượng (img_011, img_017), ma-nơ-canh (img_033), hình trên biển (img_018), mascot (img_019), người trong gương (img_020), người nhìn qua kính xe kín (R-07). |
| `ignore_region` | **Vùng ảnh** (không phải vật): bỏ sót ở đây không tính lỗi. Chỉ dùng cho (a) xe có động cơ ngoài taxonomy, mỗi xe một vùng; (b) cụm vật thật quá nhỏ hoặc quá dày không tách được; (c) vệt nhoè không tách được vật. | img_009 (buýt nhỏ), img_004 (tuk-tuk), img_021 / img_030 (lõi dày), img_032 (đám đông xa), img_014 (vệt nhoè) | Không dùng cho vật không thật; không dùng để né vật khó mà vẫn thuộc lớp; không dùng cho vật tách được. |

### 2.2. Vật không thuộc lớp nào — một cách xử lý cho mỗi loại

| Loại | Cách xử lý |
|---|---|
| Xe buýt / xe khách > 16 chỗ, tuk-tuk, xe ba bánh có động cơ | Mỗi xe **một `ignore_region`** ôm xe (chỉ khi xe cao ≥ 20 px). Người ngồi ở chỗ hở vẫn gán `person`, box người nằm chồng lên vùng này. |
| Xe đạp, xích lô, xe đạp thồ, xe đẩy tay, xe ngựa | **Không gán.** Người ngồi/đi vẫn gán `person`. |
| Tượng, ma-nơ-canh, đồ chơi, xe mô hình, hình in, tranh tường, bia mộ hình xe, phản chiếu | **Không gán, không dùng `ignore_region`** (đây là mẫu âm cho mô hình). |
| Hàng hóa, quang gánh, vật dài nhô ra | Không có box riêng, không đưa vào box của người hay xe nếu nhô ra khỏi thân. |

Lớp mới (`bus`, `tricycle`, `bicycle`) **không thêm** ở v1.0; nếu cần, đổi ở calibration kèm rà lại cả lô.

### 2.3. Thuộc tính (đủ ba thuộc tính cho mọi box trừ `ignore_region`)

**`occluded_level`** — % diện tích **cả vật** (ước lượng cả phần khuất) bị **vật khác** che.

| Mức | Ngưỡng | Ví dụ |
|---|---|---|
| `0` | che ≤ 5% | img_023, img_034 |
| `1` | 6–30% | img_040 (taxi bị người che ≈ 30%) |
| `2` | 31–70% | img_005 (người ngồi sau), img_001 |
| `3` | > 70% nhưng vẫn nhận ra chắc chắn lớp | img_021, img_030 |

- Dưới 10% diện tích còn thấy thì **không gán** (R-02); không tạo box dựa trên phỏng đoán.
- Không tính là che: mép ảnh cắt (dùng `truncated`); người ngồi trên xe che xe đó (R-04); vật tự che chính nó; blur, tối hoặc phản xạ.
- Tính là che: vật cùng loại che nhau, hàng rào, cột, kính, cây (img_018 xe sau hàng rào: mức 1–2).

**`truncated`** — `yes` khi **từ 10% thân vật nằm ngoài khung ảnh** (ước lượng); dưới 10% hoặc chỉ chạm mép là `no`. Box dừng ở mép ảnh. Ví dụ `yes`: img_015, img_040, img_012.

**`can_review`** — tick khi **đã vẽ được box** nhưng còn bất định thật sự về: lớp (xe thùng kín giữa `van` và `truck`; xe khách 14–18 chỗ), mức che sát ngưỡng 30% hoặc 70%, vật thật hay trưng bày, hoặc thuộc ca đã đánh dấu "hỏi" ở mục 4 (img_019, img_022). **Không** tick khi box chỉ lệch vài px, vật nhỏ nhưng rõ, hoặc ca đã có quy tắc R-xx. Nếu không thể vẽ box có căn cứ: không tạo box giả, ghi `image`, mã người gán, vùng, lý do vào `image_issues.csv` sau khi khóa nhãn độc lập.

## 3. Quy tắc vẽ box

- **R-01 Ôm sát phần nhìn thấy.** Box chữ nhật nhỏ nhất bao hết pixel nhìn thấy của **một** vật, mép cách mép vật không quá 3 px (ảnh cạnh dài 1024 px). Không ước lượng phần khuất, không lấy bóng đổ, khoảng trời/đường, hàng hóa hay vật dài nhô ra (ống ở img_029, quang gánh ở img_003). Hai người / hai xe sát nhau vẫn là hai box nếu phân biệt được; không gộp người với xe, không gộp nhiều xe đỗ thành một box.
- **R-02 Vật bị che.** Vật bị che chia làm nhiều mảnh nhưng chắc chắn là một vật thì **một box** bao tất cả các mảnh. Dưới 10% diện tích còn thấy (chỉ lộ gương, tay lái, một bánh) thì không gán.
- **R-03 Cắt mép ảnh.** Box dừng ở mép; `truncated` theo ngưỡng 10%. Vật còn dưới 10% trong khung, hoặc cao < 20 px, thì không gán.
- **R-04 Người đi xe máy.** Một box `motorcycle` ôm thân xe (bánh trước tới bánh sau và tay lái) và **một box `person` cho mỗi người** (người lái và từng người ngồi sau), ôm phần nhìn thấy của người đó kể cả mũ. Hai loại box được chồng nhau. Người ngồi trên xe **không** tính là che xe. Người trên xe đạp/xích lô chỉ có box `person`. Ví dụ: img_005, img_026, img_006, img_010.
- **R-05 Xe chở xe, hàng hóa.** Chỉ gán **xe chủ**; xe và hàng được chở không có box riêng. Box xe tải ôm cả đầu kéo, rơ-moóc và giàn chở (nên xe được chở nằm trong box, img_002). Xe nhỏ đặt trên nóc xe khác (img_024) không có box riêng; `car` ôm xe bên dưới. Hàng trong thùng bán tải (img_012) nằm sẵn trong khung xe; hàng nhô ra ngoài thân thì không tính.
- **R-06 Vật không thật.** Không box và không `ignore_region` cho tượng, ma-nơ-canh, đồ chơi, xe mô hình, hình in/billboard/tranh, bia mộ hình xe, phản chiếu (gương, kính, mặt đường ướt). Vật thật cạnh hoặc chồng với vật giả vẫn gán bình thường (img_028: bán tải là `truck`, tượng ngựa thì không). Xe thật đỗ/trưng bày (img_024) vẫn gán.
- **R-07 Người trong khoang kín.** Người chỉ thấy qua kính ô tô, xe tải, van, xe buýt **không gán**. Người ở chỗ hở (xe máy, xe đạp, xích lô, tuk-tuk hở, thùng xe) gán `person` khi cao ≥ 20 px và còn thấy ≥ 30% thân.
- **R-08 Xe chủ (ego) và gương.** Không gán phần của xe đang chụp (gương, nắp ca-pô, kính, tay lái) và mọi thứ chỉ thấy trong gương chiếu hậu (img_020).
- **R-09 Vật nhỏ.** Vật thật cao < 20 px (`min_box_height_px`, giữ nguyên) không vẽ box riêng. Từ 3 vật nhỏ sát nhau thì gom vào **một** `ignore_region`; vật nhỏ đứng lẻ thì bỏ qua, không tính lỗi (img_025, img_013, img_039).
- **R-10 Đám đông, dãy xe dày.** Từ 5 vật cùng lớp trở lên chồng sát nhau: vẽ box riêng cho vật còn tách được (thấy ≥ 30% thân, cao ≥ 20 px), phần lõi không tách được gom vào một `ignore_region`. Vùng này **không bao** box đã gán riêng của vật khác, ngoại trừ box `person` của người trong tuk-tuk/xe hở theo R-12 (img_021, img_030, img_008, img_032, đám đông sau rào ở img_019).
- **R-11 Nhoè, quá tối.** Nhoè chuyển động: nếu vẫn nhận ra lớp và biên thì gán; nếu chỉ còn vệt thì `ignore_region` (img_014). Ảnh tối (img_008, img_013, img_031): gán khi nhận ra lớp chắc chắn, chỗ không chắc thì tick `can_review`; không gán quầng đèn như một xe.
- **R-12 Xe có động cơ ngoài taxonomy.** Mỗi xe buýt lớn, tuk-tuk, xe ba bánh một `ignore_region` ôm xe (img_004, img_016, img_035, img_040, img_009, img_037, img_031). Không dùng ignore cho vật tách được thuộc lớp có sẵn.
- **R-13 Xe hình dạng lạ.** Vật có kích thước thật, có bánh, đang chạy trên đường thì xếp vào lớp phù hợp và tick `can_review` (img_019: `car`; chuột bông và đồ trang trí không box).

## 4. Ca khó và quyết định

`quy tắc` = nhóm đã chốt; `hỏi` = tick `can_review` và xem lại ở Pha 4–5.

| Ảnh | Tình huống | Rule | Quyết định | Lý do | Loại |
|---|---|---|---|---|---|
| img_002 | xe tải chở 7–8 ô tô | R-05 | 1 `truck` ôm cả tổ hợp; không `car` cho xe chở | xe chở là hàng; tránh box che nặng | hỏi |
| img_024 | xe vàng nhỏ trên nóc xe; showroom | R-05, R-06, R-10 | `car` cho xe đen; xe vàng không box; xe thật xa phía sau: box nếu rõ, `ignore_region` nếu chồng nhiều | như img_002 | hỏi |
| img_004, img_016, img_035, img_040 | tuk-tuk, xe ba bánh | R-12 | mỗi xe một `ignore_region`; người ở chỗ hở: `person` | không có lớp, xe thật | quy tắc |
| img_009, img_037, img_031 | xe buýt, minibus > 16 chỗ | R-12 | `ignore_region` | ngoài taxonomy | quy tắc |
| img_037 | bán tải nắp thùng cao | lớp | `truck` | canopy không biến bán tải thành van | quy tắc |
| img_001, img_007, img_012, img_028, img_036 | bán tải, tải nhỏ, tải thùng | lớp | `truck` (img_036 thùng phủ bạt vẫn là `truck`) | khoang hàng tách cabin | quy tắc |
| img_005, img_016 | xe khách nhỏ | lớp | `van` | liền khối, ≤ 16 chỗ | quy tắc |
| img_011 | tượng sinh vật lớn, xe tạo hình kim loại | R-06 | không box; người thật xa theo R-09/R-10 | vật không thật | quy tắc |
| img_017 | tượng người cưỡi ngựa đội nón giao thông | R-06 | không box | tượng | quy tắc |
| img_027 | bia mộ hình xe tải | R-06 | không box | không phải xe | quy tắc |
| img_028 | bán tải thật chở tượng ngựa | R-06 | `truck`; ngựa không box | tách thật/giả | quy tắc |
| img_033, img_038 | ma-nơ-canh, phản chiếu; đồ chơi | R-06 | không box | giả / phản chiếu | quy tắc |
| img_018 | chân dung trên biển; xe thật sau hàng rào | R-06, R-02 | không box chân dung; xe sau rào `car` mức 1–2 | hình in; hàng rào là vật che | quy tắc |
| img_020 | cảnh trong gương chiếu hậu | R-06, R-08 | không box gì | phản chiếu, xe chủ | quy tắc |
| img_019 | xe giả tưởng, tài xế thật, mascot, đám đông | R-13, R-06, R-10 | `car` (`can_review`); tài xế `person`; mascot không; đám đông sau rào `ignore_region` | xe chạy được | hỏi |
| img_003, img_006, img_023, img_029 | xe đạp, xích lô, quang gánh, ống dài | lớp, R-01 | xe đạp/xích lô không box; người `person`; vật nhô ra không vào box | ngoài taxonomy | quy tắc |
| img_005 | 4 người trên 2 xe máy cận cảnh, che nhau | R-04, R-02 | mỗi người một `person` (mức 1–2); xe máy lộ < 10% thì không gán | tránh gán vật không thấy | quy tắc |
| img_040 | người cận cảnh bị cắt dưới, chỉ lộ gương xe; taxi; tuk-tuk; buýt | R-02, R-03, R-07, R-12 | `person` `truncated=yes`; xe máy không gán; taxi `car` mức 1; tuk-tuk và buýt `ignore_region`; tài xế taxi không gán | đúng các quy tắc | quy tắc |
| img_021, img_030, img_008 | dòng xe máy dày, xe đỗ chồng | R-10 | box cho vật tách được; lõi `ignore_region` | không đoán instance | quy tắc |
| img_032 | đám đông xa sau dải phân cách; xe cắt mép trái | R-09, R-10, R-03 | đám đông `ignore_region`; xe `truncated=yes` | người nhỏ, sát nhau | quy tắc |
| img_014 | hai người rõ; xe nhoè thành vệt | R-11 | 2 `person`; vệt `ignore_region` | nhoè | quy tắc |
| img_015 | hai ô tô bị cắt mép; phản chiếu đường ướt | R-03, R-06 | `car` `truncated=yes`; phản chiếu không gán | cắt > 10% | quy tắc |
| img_022 | ảnh đen trắng, mô tô có sidecar | R-04, R-13 | 1 `motorcycle` ôm cả sidecar; `person` nếu rõ; `can_review` | sidecar là bộ phận xe | hỏi |
| img_026 | hai người một xe máy, trước tranh tường cảng | R-04, R-06 | 1 `motorcycle` + 2 `person`; tranh không gán | tranh là hình in | quy tắc |
| img_010 | xe đạp thồ rau, người bán, xe tay ga có người ngồi | lớp, R-04 | xe đạp thồ không gán; mỗi người `person`; mỗi tay ga `motorcycle` | theo lớp | quy tắc |
| img_025, img_013, img_039 | người, xe rất xa | R-09 | < 20 px không box lẻ, cụm ≥ 3 vào `ignore_region`; ≥ 20 px gán thường | ngưỡng lô | quy tắc |

## 5. QC và phiên bản

- Ngưỡng của lô giữ nguyên từ `submission/taxonomy.json` (`rules`): `min_box_height_px` 20; `match_iou` 0.5; `large_px` 50; IoU 0.85 (vật cao ≥ 50 px) / 0.70 (nhỏ hơn); IAA IoU 0.85; đồng thuận lớp 0.95; tối đa 2% ảnh lỗi. Không tự sửa mục `rules`.
- Script kiểm được lớp, thuộc tính, hình học, kích thước tối thiểu, độ lệch giữa người gán. **Không** kiểm được box có ôm sát không, hay cả nhóm cùng hiểu sai quy tắc: người review kiểm bằng mắt.
- Ca mơ hồ: tick `can_review` (hoặc ghi `image_issues.csv`); ở Pha 5 nhóm chốt từng ca, mỗi ca một dòng trong `calibration_decisions.csv` kèm mã R-xx. Người chốt dùng mã ba chữ số; người kiểm lại khác người sửa.
- Quyết định làm đổi tập vật, lớp, box hoặc thuộc tính là thay đổi `semantic`: tăng phiên bản guideline và rà lại phạm vi bị ảnh hưởng.
- Điểm dễ lệch cần theo dõi ở Pha 4: ước lượng 10% của `truncated`, mốc 5 vật của R-10, ranh giới 16 chỗ giữa `van` và xe buýt, độ rộng `ignore_region` quanh tuk-tuk.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Guideline starter của giảng viên | chưa xử lý ca biên |
| v0.1-cá-nhân (111, 222, 333) | ba bản độc lập ở Pha 1 | nguồn của v1.0 |
| **v1.0** | Hợp nhất ba bản, chốt 10 điểm bất đồng (mục 0), thêm R-13, khóa định nghĩa lớp trong `taxonomy.json` | img_002, img_024, img_004, img_009, img_014, img_021, img_040 |
