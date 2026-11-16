# Guideline gán nhãn — v1.1 (sau calibration, nhóm G0001)

> **Trạng thái:** bản sau calibration (Pha 5), dùng để sửa nhãn và duyệt QC. Thay đổi so với v1.0 là thay đổi `semantic`: cả lô 40 ảnh được rà lại. Bản v1.0 giữ nguyên ở `guideline_v1.0_agreed.md`.

- Phiên bản: v1.1
- Ngày: 2026-11-16
- Người viết / duyệt: 111, 222, 333 (điều phối: 111)
- Loại nhãn: bounding box trên phần nhìn thấy của vật; CVAT, xuất COCO 1.0
- Taxonomy đi kèm: `submission/taxonomy_v1.1.json` (version `v1.1`); bản khi gán độc lập: `submission/taxonomy.json` (v1.0)

## 0. Cách chốt các điểm ba bản cá nhân khác nhau

Ba bản giống nhau ở: bán tải là `truck`; xe có sidecar là một `motorcycle`; tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu không gán; người đi xe máy tách thành box xe và box từng người, được chồng nhau; ngưỡng 20 px; xe giả tưởng chạy được (img_019) tạm là `car` kèm `can_review`; người ngồi ở chỗ hở của tuk-tuk vẫn được gán nhãn người (nay là `rider`). Chỗ khác nhau:

| # | Chủ đề | 111 | 222 | 333 | **Chốt v1.0** | Lý do |
|---|---|---|---|---|---|---|
| 1 | Xe buýt, tuk-tuk, xe ba bánh | mỗi xe một `ignore_region` | giống 111 | không gán, **không** dùng ignore | Ban đầu: mỗi xe một `ignore_region`. **Điều chỉnh trước Pha 3:** xe buýt / xe khách > 16 chỗ vẫn là `ignore_region`; tuk-tuk và xe ba bánh có động cơ thành lớp **`three_wheeler`** | Điều phối 111 thêm lớp để mô hình học tuk-tuk thay vì bị bỏ qua; xe buýt vẫn ngoài taxonomy. |
| 2 | Xe được chở trên xe tải (img_002) | chỉ box `truck` | giống 111 | thêm một `car` cho mỗi xe chở | **Chỉ một box `truck`** | 2/3 bản; tránh 7–8 box bị che nặng, vốn là nguồn lệch IAA lớn. |
| 3 | Xe vàng trên nóc xe (img_024) | không box | gán nếu là xe thật | không gán (prop) | **Không box riêng** | cùng nguyên tắc xe được chở (R-05); 111 và 333 trùng kết quả. |
| 4 | Người nhìn qua kính xe kín | không gán | không gán | gán nếu thấy chắc, ≥ 20 px | **Không gán** (R-07) | 2/3 bản; biên người qua kính không đáng tin, dễ lệch. |
| 5 | Ngưỡng `occluded_level` | 5 / 30 / 70% | 5 / 30 / 70% | 10 / 35 / 70% | **≤5 / 6–30 / 31–70 / >70%** | 2/3 bản và khớp gợi ý của đề lab. |
| 6 | `truncated` | `yes` khi ≥ 10% ngoài khung | giống 111 | `yes` khi bị mép cắt (mọi mức) | **`yes` khi ≥ 10% thân ngoài khung** | 2/3 bản. Rủi ro: ước lượng 10% chủ quan; xem lại ở Pha 4 nếu lệch. |
| 7 | Vệt nhoè chuyển động | `ignore_region` khi chỉ còn vệt | giống 111 | không ignore; ghi `image_issues.csv` | **`ignore_region`** nếu chỉ còn vệt (R-11) | 2/3 bản; vẫn có đường ghi ca không vẽ được. |
| 8 | Từ bao nhiêu vật thì sang `ignore_region` | ≥ 6 vật chồng | ≥ 5 vật chồng | không nêu số | **≥ 5 vật cùng lớp chồng sát** (R-10) | lấy mốc thấp hơn của hai bản có số; sẽ thử lại ở Pha 4. |
| 9 | Người thấy từ bao nhiêu thì gán | ≥ 30% thân | "thấy rõ" | ≥ 20 px, chắc chắn | **cao ≥ 20 px và thấy ≥ 30% thân** (R-07, R-10) | có số đo cụ thể, không dựa cảm tính. |
| 10 | Cụm vật nhỏ | ≥ 3 vật nhỏ → một vùng | "có thể" gom | chỉ ignore khi dày | **≥ 3 vật cao < 20 px sát nhau → một `ignore_region`** (R-09) | 111 cụ thể nhất; đơn lẻ thì bỏ qua. |

## 0.1. Quyết định calibration (v1.1)

Nguồn: `submission/qc/iaa_summary.md` (IoU TB 0,814; cùng lớp 91,3%; 619 ca trên 34 ảnh) và `submission/qc/calibration_decisions.csv`.

| # | Điểm lệch ở Pha 4 | Quyết định v1.1 | Quy tắc | Loại lệch |
|---|---|---|---|---|
| 1 | Xe buýt gán `ignore_region` nên mô hình không học được xe buýt | Thêm lớp **`bus`**; mỗi xe buýt / xe khách > 16 chỗ một box | R-12 | đổi taxonomy |
| 2 | Đám đông: 222 vẽ box từng người, 111 và 333 dùng `ignore_region` (167 ca) | **Box riêng cho từng người / xe còn thấy ≥ 30% thân và cao ≥ 20 px**; `ignore_region` chỉ cho phần lõi không tách được và được phép chồng lên box riêng | R-09, R-10 | guideline mơ hồ |
| 3 | 222 vẽ một box `rider` ôm cả người lẫn xe | Giữ R-04: **hai box tách** (xe và từng người) | R-04 | một người lệch |
| 4 | Box `motorcycle` có lấy gương, tay lái hay không (IoU thấp) | **Lấy cả gương và tay lái** | R-04 | guideline mơ hồ |
| 5 | `occluded_level` chỉ đồng thuận 51,7% | Khi hai người lệch một mức, lấy **mức thấp hơn**; người ngồi trên xe không tính là che xe; `truncated=yes` chỉ khi box chạm mép ảnh | mục 2.3 | guideline mơ hồ |
| 6 | Câu "Lớp mới (`bus`, `tricycle`, `bicycle`) không thêm" ở mục 2.2 mâu thuẫn với bảng lớp | Đính chính: `bicycle` và `three_wheeler` đã là lớp từ v1.0; `bus` thêm ở v1.1 | mục 2.2 | lỗi soạn thảo |

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** (`car`, `truck`, `van`, `bus`, `motorcycle`, `bicycle`, `three_wheeler`, `pedestrian`, `rider`) trên ảnh đường phố 2D, cho camera giao thông và hỗ trợ nhận biết vật cản. Nguy hiểm nhất là **bỏ sót người và xe thật**; thứ hai là báo nhầm **vật không thật** (tượng, ma-nơ-canh, hình in, phản chiếu, đồ chơi) thành người hoặc xe. Nguyên tắc chung: vật thật nhìn thấy trực tiếp thì có box; vật không thật thì không có box; chỗ nào không chắc thì tick `can_review` thay vì đoán.

Mục tiêu của guideline là để hai người độc lập nhìn cùng một ảnh vẫn ra cùng quyết định về vật nào gán, lớp nào, box bao phần nào, và ba thuộc tính.

## 2. Lớp và thuộc tính

### 2.1. Lớp (mỗi vật thuộc đúng một lớp, đúng một box)

| Lớp | Định nghĩa (một câu) | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô con thân kín, từ 9 chỗ trở xuống: sedan, hatchback, wagon, SUV, MPV, taxi 4 bánh; xe thật đang đỗ hoặc trưng bày vẫn gán. | img_023, img_015, img_012 (SUV), img_040 (taxi), img_024 (xe thật trong showroom) | Bán tải là `truck`; xe đồ chơi (img_038), xe tạc đá (img_027) không phải xe. |
| `truck` | Xe có khoang chở hàng tách khỏi cabin hoặc để hở: tải nhỏ, tải thùng, bán tải (kể cả có nắp thùng), đầu kéo kèm rơ-moóc. | img_001, img_007, img_012, img_028, img_036, img_037, img_002 | Xe thân liền khối kín là `van` (img_016). |
| `van` | Xe thân liền khối kín (khoang người/hàng liền cabin), cỡ nhỏ: panel van hoặc xe khách tối đa 16 chỗ. | img_005, img_016 | Xe khách > 16 chỗ (img_009) là `bus`; SUV/MPV dáng ô tô là `car`. |
| `motorcycle` | Xe hai bánh có động cơ: xe máy, tay ga, moped, mô tô, xe máy điện dạng tay ga; xe gắn sidecar là **một** xe, một box. | img_006, img_008, img_026, img_022 | Xe đạp, xích lô là `bicycle` (img_003, img_023); tuk-tuk là `three_wheeler`. |
| `bicycle` | Xe hai bánh chạy bằng sức người (xe đạp, xe đạp thồ có giỏ hoặc hàng) và xích lô (xe đạp ba bánh chở khách). Box ôm thân xe; người đi là box riêng. | img_003, img_006, img_023 (xích lô), img_029, img_010 (xe đạp thồ rau) | Xe máy là `motorcycle`; xe đẩy tay, xe ngựa không gán. |
| `three_wheeler` | Xe ba bánh có động cơ: tuk-tuk, auto-rickshaw, xe ba gác máy. Box ôm thân xe. | img_004, img_016, img_035, img_040 | Xích lô đạp là `bicycle`; xe buýt là `bus`. |
| `bus` | Xe buýt, xe khách trên 16 chỗ, minibus chở khách; mỗi xe một box ôm phần nhìn thấy. | img_009, img_037, img_031, img_040, img_008 | Xe khách ≤ 16 chỗ là `van`; tuk-tuk chở khách là `three_wheeler`. |
| `pedestrian` | Người thật không ở trên phương tiện: đi bộ, đứng, ngồi, đẩy xe, gánh hay xách hàng. Box bao cả đồ người đó mang (R-14). | img_014, img_034, img_003 (người gánh hàng), img_010 (người bán đứng cạnh xe) | Tượng (img_011, img_017), ma-nơ-canh (img_033), hình trên biển (img_018), mascot (img_019), người trong gương (img_020), người nhìn qua kính xe kín (R-07). |
| `rider` | Người thật đang lái hoặc ngồi trên phương tiện hở: xe máy, xe đạp, xích lô, tuk-tuk, ô tô mui trần, thùng xe; mỗi người một box (người lái và từng người ngồi sau). Box bao cả đồ người đó mang (R-14). | img_005, img_006, img_026, img_023 (phu xích lô), img_019 (tài xế xe mui trần) | Người đứng cạnh xe là `pedestrian`; người trong khoang kín không gán (R-07). |
| `ignore_region` | **Vùng ảnh** (không phải vật): bỏ sót ở đây không tính lỗi. Chỉ dùng cho (a) phần lõi đám đông hoặc dãy xe không tách được từng vật; (b) cụm từ 3 vật thật cao < 20 px; (c) vệt nhoè không tách được vật. Được phép chồng lên box riêng. | img_021 / img_030 (lõi dày), img_019 (hàng sau đám đông), img_014 (vệt nhoè) | Không dùng cho vật không thật; không dùng để né vật khó mà vẫn thuộc lớp; không dùng cho vật tách được. |

### 2.2. Vật không thuộc lớp nào — một cách xử lý cho mỗi loại

| Loại | Cách xử lý |
|---|---|
| Xe buýt / xe khách > 16 chỗ | Từ v1.1 là lớp **`bus`** (xem bảng 2.1), không còn dùng `ignore_region`. Người ngồi trong khoang kín của xe không gán (R-07). |
| Xe đẩy tay, xe ngựa | **Không gán.** Người đi/ngồi vẫn gán `rider` hoặc `pedestrian`. (Xe đạp, xích lô, tuk-tuk nay là lớp `bicycle`, `three_wheeler`.) |
| Tượng, ma-nơ-canh, đồ chơi, xe mô hình, hình in, tranh tường, bia mộ hình xe, phản chiếu | **Không gán, không dùng `ignore_region`** (đây là mẫu âm cho mô hình). |
| Hàng hóa chất trên xe, vật nhô khỏi thân xe | Không có box riêng, không đưa vào box xe nếu nhô ra khỏi thân. Đồ **do người trực tiếp mang** thì vào box người (R-14). |

**Đính chính v1.1:** câu ở v1.0 "Lớp mới (`bus`, `tricycle`, `bicycle`) không thêm" là sai. `bicycle` và `three_wheeler` đã là lớp từ v1.0; `bus` được thêm ở v1.1 kèm rà lại cả lô.

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
- Khi phân vân giữa hai mức liền nhau, chọn **mức thấp hơn** (quyết định calibration v1.1).
- Tính là che: vật cùng loại che nhau, hàng rào, cột, kính, cây (img_018 xe sau hàng rào: mức 1–2).

**`truncated`** — `yes` khi **từ 10% thân vật nằm ngoài khung ảnh** (ước lượng); dưới 10% hoặc chỉ chạm mép là `no`; box không chạm mép ảnh thì luôn là `no`. Box dừng ở mép ảnh. Ví dụ `yes`: img_015, img_040, img_012.

**`can_review`** — tick khi **đã vẽ được box** nhưng còn bất định thật sự về: lớp (xe thùng kín giữa `van` và `truck`; xe khách 14–18 chỗ), mức che sát ngưỡng 30% hoặc 70%, vật thật hay trưng bày, hoặc thuộc ca đã đánh dấu "hỏi" ở mục 4 (img_019, img_022). **Không** tick khi box chỉ lệch vài px, vật nhỏ nhưng rõ, hoặc ca đã có quy tắc R-xx. Nếu không thể vẽ box có căn cứ: không tạo box giả, ghi `image`, mã người gán, vùng, lý do vào `image_issues.csv` sau khi khóa nhãn độc lập.

## 3. Quy tắc vẽ box

- **R-01 Ôm sát phần nhìn thấy.** Box chữ nhật nhỏ nhất bao hết pixel nhìn thấy của **một** vật, mép cách mép vật không quá 3 px (ảnh cạnh dài 1024 px). Không ước lượng phần khuất, không lấy bóng đổ, khoảng trời/đường, hàng chất trên xe nhô ra khỏi thân xe (đồ do người trực tiếp mang thì tính vào box người, xem R-14). Hai người / hai xe sát nhau vẫn là hai box nếu phân biệt được; không gộp người với xe, không gộp nhiều xe đỗ thành một box.
- **R-02 Vật bị che.** Vật bị che chia làm nhiều mảnh nhưng chắc chắn là một vật thì **một box** bao tất cả các mảnh. Dưới 10% diện tích còn thấy (chỉ lộ gương, tay lái, một bánh) thì không gán.
- **R-03 Cắt mép ảnh.** Box dừng ở mép; `truncated` theo ngưỡng 10%. Vật còn dưới 10% trong khung, hoặc cao < 20 px, thì không gán.
- **R-04 Người đi xe máy, xe đạp, xích lô.** Một box cho xe (`motorcycle`, `bicycle` hoặc `three_wheeler`, ôm thân xe từ bánh trước tới bánh sau, **kể cả gương và tay lái**) và **một box `rider` cho mỗi người** (người lái và từng người ngồi sau), ôm phần nhìn thấy của người đó kể cả mũ. Hai loại box được chồng nhau. Người ngồi trên xe **không** tính là che xe. **Không** vẽ một box `rider` ôm cả người lẫn xe. Ví dụ: img_005, img_026, img_006, img_010, img_023.
- **R-05 Xe chở xe, hàng hóa.** Chỉ gán **xe chủ**; xe và hàng được chở không có box riêng. Box xe tải ôm cả đầu kéo, rơ-moóc và giàn chở (nên xe được chở nằm trong box, img_002). Xe nhỏ đặt trên nóc xe khác (img_024) không có box riêng; `car` ôm xe bên dưới. Hàng trong thùng bán tải (img_012) nằm sẵn trong khung xe; hàng nhô ra ngoài thân thì không tính.
- **R-06 Vật không thật.** Không box và không `ignore_region` cho tượng, ma-nơ-canh, đồ chơi, xe mô hình, hình in/billboard/tranh, bia mộ hình xe, phản chiếu (gương, kính, mặt đường ướt). Vật thật cạnh hoặc chồng với vật giả vẫn gán bình thường (img_028: bán tải là `truck`, tượng ngựa thì không). Xe thật đỗ/trưng bày (img_024) vẫn gán.
- **R-07 Người trong khoang kín.** Người chỉ thấy qua kính ô tô, xe tải, van, xe buýt **không gán**. Người ở chỗ hở (xe máy, xe đạp, xích lô, tuk-tuk hở, ô tô mui trần, thùng xe) gán `rider` khi cao ≥ 20 px và còn thấy ≥ 30% thân.
- **R-08 Xe chủ (ego) và gương.** Không gán phần của xe đang chụp (gương, nắp ca-pô, kính, tay lái) và mọi thứ chỉ thấy trong gương chiếu hậu (img_020).
- **R-09 Vật nhỏ.** Vật thật cao < 20 px (`min_box_height_px`, giữ nguyên) không vẽ box riêng. Từ 3 vật nhỏ sát nhau thì gom vào **một** `ignore_region`; vật nhỏ đứng lẻ thì bỏ qua, không tính lỗi. Vật cao ≥ 20 px và nhận ra chắc chắn lớp thì **có box riêng**, kể cả khi ở xa (img_025, img_013, img_039, img_011).
- **R-10 Đám đông, dãy xe dày (sửa ở v1.1).** **Mỗi người / xe còn thấy ≥ 30% thân và cao ≥ 20 px đều có box riêng**, dù đứng trong đám đông (img_032: từng người ở trạm xe buýt; img_019: các hàng trước của đám đông sau rào; img_021: từng người đi xe máy). Chỉ phần lõi không tách được (chỉ lộ đầu, thấy < 30% thân) mới gom vào một `ignore_region`; vùng này **được phép chồng lên** box riêng, và box riêng nằm trong vùng vẫn có hiệu lực.
- **R-11 Nhoè, quá tối.** Nhoè chuyển động: nếu vẫn nhận ra lớp và biên thì gán; nếu chỉ còn vệt thì `ignore_region` (img_014). Ảnh tối (img_008, img_013, img_031): gán khi nhận ra lớp chắc chắn, chỗ không chắc thì tick `can_review`; không gán quầng đèn như một xe.
- **R-12 Tuk-tuk, xe ba bánh và xe ngoài taxonomy.** Tuk-tuk, auto-rickshaw, xe ba bánh có động cơ gán `three_wheeler` (img_004, img_016, img_035, img_040). Xe buýt lớn / xe khách > 16 chỗ, minibus chở khách gán **`bus`**, mỗi xe một box (img_009, img_037, img_031, img_040, img_008). Không dùng ignore cho vật tách được thuộc lớp có sẵn.
- **R-13 Xe hình dạng lạ.** Vật có kích thước thật, có bánh, đang chạy trên đường thì xếp vào lớp phù hợp và tick `can_review` (img_019: `car`; chuột bông và đồ trang trí không box).
- **R-14 Người mang nhiều đồ.** Người gánh, vác, xách, đội, ôm hàng cồng kềnh (quang gánh, bao, ống dài…) thì **một box bao cả người lẫn đồ họ đang mang**, dù đồ nhô xa khỏi thân (img_003 người gánh hai giỏ; img_029 người đỡ ống dài). Áp dụng cho `pedestrian` và `rider`. Đồ chất trên xe thì không tính (R-01). Khi một món đồ dài do hai người cùng đỡ (img_029), mỗi người một box bao phần mình đỡ và tick `can_review`.

## 4. Ca khó và quyết định

`quy tắc` = nhóm đã chốt; `hỏi` = tick `can_review` và xem lại ở Pha 4–5.

| Ảnh | Tình huống | Rule | Quyết định | Lý do | Loại |
|---|---|---|---|---|---|
| img_002 | xe tải chở 7–8 ô tô | R-05 | 1 `truck` ôm cả tổ hợp; không `car` cho xe chở | xe chở là hàng; tránh box che nặng | hỏi |
| img_024 | xe vàng nhỏ trên nóc xe; showroom | R-05, R-06, R-10 | `car` cho xe đen; xe vàng không box; xe thật xa phía sau: box nếu rõ, `ignore_region` nếu chồng nhiều | như img_002 | hỏi |
| img_004, img_016, img_035, img_040 | tuk-tuk, xe ba bánh (img_035 chở hàng cao) | R-12, R-01 | mỗi xe một `three_wheeler` ôm thân xe, hàng chất cao ngoài thân không tính; người lái/ngồi ở chỗ hở: `rider` | lớp mới thêm trước Pha 3 | quy tắc |
| img_009, img_037, img_031, img_008 | xe buýt, minibus > 16 chỗ | R-12 | `bus`, mỗi xe một box (img_008: xe chở khách phía xa, `can_review`) | lớp thêm ở v1.1 | quy tắc |
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
| img_019 | xe giả tưởng, tài xế thật, mascot, đám đông | R-13, R-06, R-10 | `car` (`can_review`); tài xế `rider`; mascot không; đám đông sau rào: box `pedestrian` cho người thấy ≥ 30% thân, phần còn lại `ignore_region` | xe mui trần chạy được | hỏi |
| img_003, img_006, img_023 | xe đạp, xích lô, người gánh hàng | R-04, R-14 | xe đạp và xích lô: `bicycle`; người đi: `rider`; người gánh: `pedestrian`, box bao cả quang gánh và hàng | gánh hàng là một phần của người | quy tắc |
| img_029 | người đỡ ống dài, một xe máy một xe đạp | R-04, R-14 | `motorcycle`, `bicycle`; hai `rider`, mỗi box bao cả phần ống người đó đỡ; `can_review` | ống dài vắt qua hai người | hỏi |
| img_005 | 4 người trên 2 xe máy cận cảnh, che nhau | R-04, R-02 | mỗi người một `rider` (mức 1–2); xe máy lộ < 10% thì không gán | tránh gán vật không thấy | quy tắc |
| img_040 | người cận cảnh bị cắt dưới, chỉ lộ gương xe; taxi; tuk-tuk; buýt | R-02, R-03, R-07, R-12 | `rider` `truncated=yes`; xe máy không gán; taxi `car` mức 1; tuk-tuk `three_wheeler`; buýt `bus`; tài xế taxi không gán | đúng các quy tắc | quy tắc |
| img_021, img_030, img_008 | dòng xe máy dày, xe đỗ chồng | R-10 | box riêng cho từng người / xe thấy ≥ 30% thân; chỉ lõi không tách được mới `ignore_region` | không đoán instance | quy tắc |
| img_032 | đám đông xa sau dải phân cách; xe cắt mép trái | R-09, R-10, R-03 | từng người ở hàng trước một box `pedestrian` (cao ≈ 90 px, thấy rõ); phần khuất sau biển báo `ignore_region`; xe `truncated=yes` | sửa theo R-10 v1.1 | quy tắc |
| img_014 | hai người đi bộ rõ; xe nhoè thành vệt | R-11 | 2 `pedestrian`; vệt `ignore_region` | nhoè | quy tắc |
| img_015 | hai ô tô bị cắt mép; phản chiếu đường ướt | R-03, R-06 | `car` `truncated=yes`; phản chiếu không gán | cắt > 10% | quy tắc |
| img_022 | ảnh đen trắng, mô tô có sidecar | R-04, R-13 | 1 `motorcycle` ôm cả sidecar; `rider` nếu rõ; `can_review` | sidecar là bộ phận xe | hỏi |
| img_026 | hai người một xe máy, trước tranh tường cảng | R-04, R-06 | 1 `motorcycle` + 2 `rider`; tranh không gán | tranh là hình in | quy tắc |
| img_010 | xe đạp thồ rau, người bán, xe tay ga có người ngồi | R-04 | xe đạp thồ: `bicycle`; người bán đứng cạnh: `pedestrian`; người ngồi trên xe tay ga: `rider`; mỗi tay ga `motorcycle` | theo lớp | quy tắc |
| img_025, img_013, img_039 | người, xe rất xa | R-09 | < 20 px không box lẻ, cụm ≥ 3 vào `ignore_region`; ≥ 20 px gán thường | ngưỡng lô | quy tắc |

## 5. QC và phiên bản

- Ngưỡng của lô giữ nguyên từ `submission/taxonomy.json` (`rules`): `min_box_height_px` 20; `match_iou` 0.5; `large_px` 50; IoU 0.85 (vật cao ≥ 50 px) / 0.70 (nhỏ hơn); IAA IoU 0.85; đồng thuận lớp 0.95; tối đa 2% ảnh lỗi. Không tự sửa mục `rules`.
- Script kiểm được lớp, thuộc tính, hình học, kích thước tối thiểu, độ lệch giữa người gán. **Không** kiểm được box có ôm sát không, hay cả nhóm cùng hiểu sai quy tắc: người review kiểm bằng mắt.
- Ca mơ hồ: tick `can_review` (hoặc ghi `image_issues.csv`); ở Pha 5 nhóm chốt từng ca, mỗi ca một dòng trong `calibration_decisions.csv` kèm mã R-xx. Người chốt dùng mã ba chữ số; người kiểm lại khác người sửa.
- Quyết định làm đổi tập vật, lớp, box hoặc thuộc tính là thay đổi `semantic`: tăng phiên bản guideline và rà lại phạm vi bị ảnh hưởng.
- Điểm dễ lệch cần theo dõi ở Pha 4: ước lượng 10% của `truncated`, mốc 5 vật của R-10, ranh giới 16 chỗ giữa `van` và `bus`, ranh giới `three_wheeler` với `motorcycle` và `car`, và việc có đưa hàng chất cao trên xe vào box hay không (img_035).

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Guideline starter của giảng viên | chưa xử lý ca biên |
| v0.1-cá-nhân (111, 222, 333) | ba bản độc lập ở Pha 1 | nguồn của v1.0 |
| **v1.0** | Hợp nhất ba bản, chốt 10 điểm bất đồng (mục 0), thêm R-13, khóa định nghĩa lớp trong `taxonomy.json` | img_002, img_024, img_004, img_009, img_014, img_021, img_040 |
| **v1.0 (điều chỉnh trước Pha 3)** | Tách `person` thành `pedestrian` và `rider`; thêm `bicycle` và `three_wheeler`; xích lô và xe đạp được gán; thêm R-14 (người mang nhiều đồ thì box bao cả người và đồ); sửa R-04, R-07, R-12 | điều phối 111, chưa ai gán nhãn; img_003, img_029, img_004, img_035, img_023 |
| **v1.1** | Thêm lớp `bus` (thay `ignore_region` cho xe buýt); R-10 và R-09: box riêng cho từng người / xe thấy rõ, `ignore_region` chỉ cho lõi không tách được và được chồng lên box; R-04: box xe máy lấy cả gương, tay lái, không gộp người với xe; `occluded_level` lấy mức thấp hơn khi phân vân; đính chính câu sai ở mục 2.2 | Pha 4: 619 ca trên 34 ảnh; img_019, img_021, img_032, img_034, img_009, img_037, img_006 |
