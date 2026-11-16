# Guideline gán nhãn — bản cá nhân Pha 1 — mã <mã>

- Phiên bản: v0.1-cá-nhân
- Ngày: 2026-11-16
- Người viết: 111
- Loại nhãn: bounding box trên ảnh đường phố, CVAT, xuất COCO 1.0

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện vật 2D (người và phương tiện) cho camera giao thông và xe tự lái.
Lỗi nguy hiểm nhất là **bỏ sót người và xe thật** (nhất là người đi xe máy ở đường đông). Lỗi thứ hai là **báo nhầm vật không thật** (tượng, ma-nơ-canh, hình in, phản chiếu, đồ chơi) thành người hoặc xe. Vì vậy mọi quy tắc dưới đây ưu tiên: vật thật nhìn thấy được thì phải có box; vật không thật thì không có box.

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con từ 9 chỗ trở xuống, thân liền, không có thùng hàng hở: sedan, hatchback, SUV, MPV, taxi 4 bánh. | img_023 (sedan đen), img_015 (hai xe bị cắt mép), img_012 (SUV xám) | Bán tải là `truck` (img_012, xe trắng bên phải); xe đồ chơi (img_038) và xe tạc bằng đá (img_027) không phải xe thật. |
| `truck` | Xe có khoang chở hàng tách khỏi cabin hoặc để hở: tải nhỏ, tải thùng, bán tải (kể cả có nắp thùng), đầu kéo kèm rơ-moóc. | img_007 (tải Isuzu), img_036 (tải nhỏ phủ bạt), img_001 (tải thùng hở), img_037 (bán tải nắp cao), img_002 (xe chở ô tô) | Xe thân liền khối kín là `van` (img_005). |
| `van` | Xe thân liền khối kín cỡ nhỏ (cabin và khoang chở liền nhau), chở hàng hoặc chở khách tối đa 16 chỗ. | img_005 (xe khách nhỏ phía sau), img_016 (xe khách nhỏ bên phải) | Xe khách lớn hơn 16 chỗ là xe buýt, không thuộc lớp nào (img_009); SUV là `car`. |
| `motorcycle` | Xe hai bánh có động cơ: xe máy, xe tay ga, mô tô; xe có thùng phụ (sidecar) tính là một xe; xe máy điện dạng tay ga cũng vào đây. | img_006, img_008, img_026, img_022 (có sidecar) | Xe đạp, xe đạp thồ, xích lô không phải `motorcycle` (img_006 phải, img_003, img_023). |
| `person` | Người thật nhìn thấy trực tiếp (không qua kính, không qua gương): đi bộ, đi xe máy, xe đạp, xích lô, ngồi ở chỗ hở (thùng xe, tuk-tuk). | img_014, img_034, img_005, img_010 | Ma-nơ-canh (img_033), tượng (img_011, img_017), chân dung trên biển quảng cáo (img_018), người trong gương (img_020), người trong khoang xe kín nhìn qua kính. |
| `ignore_region` | Vùng không gán từng vật, bỏ sót ở đây không tính là lỗi. Chỉ dùng cho ba loại: (a) vật thật có động cơ nhưng ngoài taxonomy (xe buýt, tuk-tuk, xe ba bánh); (b) cụm vật thật quá nhỏ hoặc quá dày; (c) vệt nhoè không tách được từng vật. | img_009 (xe buýt nhỏ), img_004 (tuk-tuk), img_032 (đám đông xa), img_014 (vệt nhoè) | Vật không thật không được đưa vào đây (xem R-06); không dùng để né vật khó mà vẫn thuộc một lớp. |

**Vật không thuộc lớp nào.** Mỗi loại chọn đúng một cách:

| Loại | Cách xử lý | Lý do |
|---|---|---|
| Xe buýt / xe khách lớn, tuk-tuk, xe ba bánh có động cơ | (b) một `ignore_region` ôm cả xe | Là xe thật đang lưu thông; để trống thì mô hình bị phạt khi nhận ra nó, nên cần vùng bỏ qua. Đề xuất nhóm xem có thêm lớp riêng (hỏi ở Pha 2). |
| Xe đạp, xích lô, xe đạp thồ, xe đẩy tay | (a) không gán | Không có động cơ, không có lớp tương ứng; người ngồi trên vẫn gán `person`. |
| Tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu | (a) không gán, không dùng `ignore_region` | Đây là mẫu "không phải vật" mà mô hình phải học bỏ qua. |

Phép thử của lớp: "mô hình có phải đoán ra điều này khi suy luận không?" Có → lớp; không → thuộc tính. Ví dụ: "bị che" mô hình không cần đoán ra → thuộc tính.

### Thuộc tính

**`occluded_level`** — % diện tích của **cả vật** (ước lượng cả phần khuất) bị vật khác che.

| Mức | Ngưỡng | Ví dụ ảnh |
|---|---|---|
| `0` | che ≤ 5% (coi như thấy hết) | img_023 (sedan), img_034 (người đi bộ) |
| `1` | 6–30% | img_040 (taxi xanh bị người che khoảng 30%) |
| `2` | 31–70% | img_005 (người ngồi sau), img_001 (người trong quán sau bàn) |
| `3` | trên 70% nhưng vẫn nhận ra chắc chắn lớp | img_021 (xe máy phía sau trong dòng xe), img_030 |

- Dưới 10% diện tích còn thấy (chỉ lộ gương, bánh, tay lái) thì **không gán** (R-02).
- Không tính vào mức che: (i) bị mép ảnh cắt (đó là `truncated`); (ii) người đang ngồi trên xe che xe đó (xem R-04); (iii) vật tự che chính nó.
- Tính vào mức che: bị vật cùng loại che (người trong đám đông), bị hàng rào, cột, kính che (img_018 xe sau hàng rào: mức 1–2).

**`truncated`** — `yes` khi **từ 10% thân vật trở lên** nằm ngoài khung ảnh (ước lượng); dưới 10% hoặc chỉ chạm mép là `no`. Ví dụ `yes`: img_015 (hai xe bị cắt ngang), img_040 (người đi xe máy cắt dưới), img_012 (thùng bán tải cắt mép phải).

**`can_review`** — tick khi:
1. Chưa chắc giữa hai lớp (xe thùng kín giữa `van` và `truck`; xe khách 14–18 chỗ).
2. Mức che nằm sát ngưỡng 30% hoặc 70%.
3. Thuộc nhóm ca "hỏi reviewer" ở mục 4 (img_002, img_019, img_022, img_024).

**Không** tick khi: box lệch vài px; vật nhỏ nhưng rõ ràng; ca đã có quy tắc R-xx giải quyết (tượng, tuk-tuk, xe đạp…).

## 3. Quy tắc vẽ box

Mỗi quy tắc dùng mã R-xx để dẫn lại ở mục 4 và trong nhật ký QC.

- **R-01 Ôm sát.** Box ôm **phần nhìn thấy** của vật, mép box cách mép vật không quá 3 px (ảnh cạnh dài 1024 px). Không ước lượng phần bị che để kéo dài box. Hàng hóa, vật dài nhô ra khỏi thân (ống dài ở img_029, quang gánh ở img_003) **không** tính vào box.
- **R-02 Vật bị che.** Một vật bị che chia làm hai phần vẫn là **một box** bao cả hai phần. Chỉ lộ dưới 10% diện tích (chỉ thấy gương, tay lái, một bánh) thì không gán.
- **R-03 Cắt mép ảnh.** Box dừng ở mép ảnh, đặt `truncated` theo ngưỡng 10%. Vật còn chưa tới 10% trong khung thì không gán.
- **R-04 Người đi xe máy.** Hai loại box riêng: một box `motorcycle` ôm thân xe (từ bánh trước tới bánh sau và tay lái; chân người thò vào thì chấp nhận), và **một box `person` cho mỗi người** (người lái và từng người ngồi sau) ôm phần nhìn thấy của người đó, kể cả mũ bảo hiểm. Hai box được phép chồng lên nhau. Người ngồi trên xe **không** tính là che xe (mức che của xe chỉ tính vật khác). Thử trên img_005 (hai xe, bốn người), img_026 (một xe hai người), img_006 (nhiều xe), img_010 (xe đỗ có người ngồi).
- **R-05 Xe chở xe, hàng hóa.** Chỉ gán **xe chủ**; xe và hàng được chở không có box riêng. Box xe tải ôm cả đầu kéo, rơ-moóc, giàn chở (nên các xe chở nằm gọn trong box, img_002). Xe nhỏ đặt trên nóc xe (img_024) không có box riêng; box `car` ôm xe bên dưới. Hàng trong thùng bán tải (img_012) nằm trong box xe vì đã nằm trong khung xe.
- **R-06 Vật không thật.** Tượng, ma-nơ-canh, đồ chơi, xe mô hình, hình in trên biển, tranh tường, bia mộ hình xe, phản chiếu trong gương hay kính: **không vẽ box và không dùng `ignore_region`**. Ngoại lệ duy nhất: vật thật nằm cạnh hoặc chồng với vật giả vẫn được gán bình thường (xe bán tải thật chở tượng ngựa ở img_028 vẫn là `truck`).
- **R-07 Người trong khoang kín.** Người ngồi trong ô tô, xe tải, van, xe buýt nhìn qua kính thì **không gán** (img_040 tài xế taxi, img_007 tài xế). Người ở chỗ hở (xe máy, xe đạp, xích lô, tuk-tuk hở, thùng xe) thì gán `person` nếu còn thấy từ 30% thân trở lên.
- **R-08 Xe chủ (ego).** Phần của xe đang chụp (gương, nắp ca-pô, kính, tay lái) không gán. Ở img_020 cái gương chiếu hậu và mọi thứ trong gương đều không gán.
- **R-09 Vật quá nhỏ.** Vật thật cao dưới 20 px (ngưỡng `min_box_height_px` của lô, giữ nguyên) không vẽ box riêng. Nếu từ 3 vật nhỏ trở lên nằm gần nhau thì gom vào **một** `ignore_region` ôm cả cụm; vật nhỏ đứng lẻ thì bỏ qua và không tính là lỗi (img_025, img_013, img_039).
- **R-10 Đám đông và dãy xe dày.** Từ 6 vật cùng lớp trở lên chồng sát nhau, phần lớn ở mức che 3 hoặc quá nhỏ: vẽ box riêng cho các vật hàng trước còn thấy từ 30% thân trở lên, **phần còn lại gom vào `ignore_region`** (img_021, img_030, img_008, img_032, đám đông sau rào ở img_019).
- **R-11 Nhoè, quá tối.** Nhoè do chuyển động: nếu vẫn nhận ra lớp và biên vật thì gán, nếu chỉ còn vệt thì `ignore_region` (img_014). Ảnh tối (img_008, img_013, img_031): gán khi nhìn ra lớp chắc chắn; chỗ nào không chắc thì tick `can_review`, không đoán.
- **R-12 Xe có động cơ ngoài taxonomy.** Mỗi xe buýt, tuk-tuk, xe ba bánh một `ignore_region` ôm xe. Người ngồi ở chỗ hở của tuk-tuk vẫn gán `person` theo R-07 (img_004, img_016).

## 4. Ca khó

Cột "Loại ca": `quy tắc` = tôi tự tin theo định nghĩa; `hỏi reviewer` = cần họp ở Pha 2.

| Ảnh | Tình huống | Rule | Quyết định | Lý do | Loại ca |
|---|---|---|---|---|---|
| img_002 | xe tải chở 7–8 ô tô trên giàn | R-05 | 1 box `truck` ôm cả đầu kéo, rơ-moóc và giàn; không box `car` cho xe được chở | xe chở không phải phương tiện tham gia giao thông độc lập; box tải đã gồm chúng | hỏi reviewer |
| img_024 | xe vàng nhỏ đặt trên nóc ô tô trong showroom | R-05, R-10 | 1 box `car` cho xe đen; xe vàng không box; dãy xe xa phía sau: xe rõ có box, phần chồng nhiều thì `ignore_region` | như img_002 | hỏi reviewer |
| img_004, img_016, img_035, img_040 | tuk-tuk, xe ba bánh (img_035 chở hàng cao) | R-12 | mỗi xe một `ignore_region`; người ngồi/lái ở chỗ hở thấy rõ: `person` | không có lớp, nhưng là xe thật | quy tắc |
| img_009, img_037, img_031 | xe buýt / xe khách lớn | R-12 | `ignore_region` ôm xe | > 16 chỗ, ngoài taxonomy | quy tắc |
| img_037 | bán tải gắn nắp thùng cao | lớp `truck` | `truck` | có thùng hàng riêng (nắp là thùng) | quy tắc |
| img_001, img_007, img_012, img_028, img_036 | bán tải và tải nhỏ | lớp `truck` | `truck` (kể cả img_036 phủ bạt) | định nghĩa `truck` | quy tắc |
| img_005, img_016, img_009 | xe khách nhỏ và minibus | lớp `van` | img_005 và img_016: `van`; img_009: `ignore_region` | tách theo 16 chỗ | quy tắc |
| img_011 | tượng sinh vật lớn và xe tạo hình từ kim loại | R-06 | không box (không `ignore_region`); người xa phía sau lấy theo R-09/R-10 | vật không thật | quy tắc |
| img_017 | tượng người cưỡi ngựa, đội nón giao thông | R-06 | không box | tượng; nón giao thông cũng không có lớp | quy tắc |
| img_027 | xe tải tạc bằng đá trên mộ | R-06 | không box | bia mộ, không phải xe | quy tắc |
| img_028 | tượng ngựa đặt trên thùng bán tải thật | R-06, R-05 | `truck` cho bán tải; ngựa không box | xe thật, ngựa giả | quy tắc |
| img_033 | ma-nơ-canh sau kính, có phản chiếu xe | R-06 | không box gì | giả và phản chiếu | quy tắc |
| img_038 | tủ kính đầy xe và thú đồ chơi | R-06 | không box | đồ chơi | quy tắc |
| img_018 | chân dung trên biển quảng cáo; xe thật đỗ sau hàng rào | R-06, R-02 | không box chân dung; xe sau rào: `car`, `occluded_level` 1–2 | hình in không phải người | quy tắc |
| img_020 | cảnh trong gương chiếu hậu | R-06, R-08 | không box gì | phản chiếu và bộ phận xe chủ | quy tắc |
| img_019 | xe giả tưởng màu vàng có chuột bông, tài xế thật | R-06, R-07 | `car` cho xe (tick `can_review`); `person` cho tài xế thấy rõ; chuột bông không box; đám đông sau rào: `ignore_region` | xe chạy được trên đường, nhưng hình dạng lạ | hỏi reviewer |
| img_006, img_003, img_023, img_029 | xe đạp, xích lô, người gánh hàng, người chở ống dài | lớp, R-01 | xe đạp/xích lô không box; người vẫn `person`; ống và quang gánh không vào box | ngoài taxonomy, và R-01 | quy tắc |
| img_005 | 4 người trên 2 xe máy, cận cảnh, che nhau nhiều | R-04, R-02 | mỗi người một `person` (mức che 1–2), xe máy lộ dưới 10% thì không gán | tránh gán xe không nhìn thấy | quy tắc |
| img_040 | người đi xe máy cận cảnh bị cắt dưới, chỉ lộ gương xe; taxi và tuk-tuk phía sau | R-02, R-03, R-07, R-12 | `person` có `truncated=yes`; xe máy không gán; taxi xanh `car` mức 1; tuk-tuk `ignore_region`; tài xế taxi không gán | đúng các quy tắc nêu | quy tắc |
| img_021, img_030, img_008 | dòng xe máy dày đặc, xe đỗ chồng chất | R-10 | box cho xe và người hàng trước; phần sau `ignore_region` | quá chồng chéo | quy tắc |
| img_032 | đám đông đứng xa sau dải phân cách | R-09, R-10 | `ignore_region` | người nhỏ, sát nhau | quy tắc |
| img_014 | hai người đi bộ rõ, phía sau xe chạy nhoè thành vệt | R-11 | 2 `person`; vệt xe `ignore_region` | tách theo R-11 | quy tắc |
| img_015 | hai ô tô bị cắt mép ảnh, phản chiếu trên mặt đường ướt | R-03, R-06 | `car` `truncated=yes` cho cả hai; phản chiếu không gán | cắt hơn 10%; phản chiếu không thật | quy tắc |
| img_022 | ảnh đen trắng, mô tô có sidecar ở xa | lớp, R-04 | 1 `motorcycle` ôm cả sidecar; `person` nếu thấy rõ; tick `can_review` | sidecar là một phần xe | hỏi reviewer |
| img_026 | hai người trên một xe máy, trước tranh tường cảng | R-04, R-06 | 1 `motorcycle` và 2 `person`; tranh tường (tàu, cảng) không gán | tranh là hình in | quy tắc |
| img_010 | xe đạp thồ rau, người bán, hai xe tay ga có người ngồi | lớp, R-04 | xe đạp thồ không gán; mỗi người 1 `person`; mỗi xe tay ga 1 `motorcycle` | theo lớp và R-04 | quy tắc |
| img_025, img_013, img_039 | người và xe rất xa, cao dưới hoặc sát 20 px | R-09 | dưới 20 px: không box lẻ, gom cụm vào `ignore_region`; từ 20 px trở lên: gán bình thường | ngưỡng lô | quy tắc |

## 5. QC và phiên bản

- Ngưỡng của lô (giữ nguyên, lấy từ `taxonomy.json`): IoU ≥ 0.85 với vật cao từ 50 px, ≥ 0.70 với vật nhỏ hơn; đồng thuận lớp ≥ 95%; tối đa 2% ảnh lỗi.
- Ca mơ hồ: người gán tick `can_review` ở box (hoặc ghi vấn đề cấp ảnh nếu chưa vẽ được); ở Pha 5 nhóm chốt từng ca, ghi mã quy tắc vào `calibration_decisions.csv`.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 → v0.1-cá-nhân | Thêm định nghĩa và phản ví dụ cho cả 5 lớp | v0 không phân `truck`, `van`, bán tải (img_001, img_037, img_005) |
| | Quy định xe/vật ngoài taxonomy: buýt, tuk-tuk → `ignore_region`; xe đạp, xích lô → không gán | img_004, img_009, img_006, img_023 |
| | Ngưỡng `occluded_level` (5/30/70%) và `truncated` (10%) | v0 chỉ ghi "ít/nhiều" |
| | R-04: tách box người và xe máy, người ngồi không tính là che xe | img_005, img_026 |
| | R-05, R-06: xe chở xe, vật không thật không gán | img_002, img_011, img_017, img_027, img_033, img_038 |
| | R-07 đến R-12: người trong khoang kín, xe chủ, vật nhỏ, đám đông, nhoè, tuk-tuk | img_040, img_020, img_025, img_032, img_014 |

## Câu chưa chốt, cần họp Pha 2

1. Có thêm lớp `bus` / `tricycle` thay vì dùng `ignore_region` cho xe buýt và tuk-tuk không (img_009, img_004)?
2. Xe được chở trên xe khác (img_002, img_024) có cần box riêng không?
3. Bán tải xếp `truck` hay `car` (img_001, img_012, img_037)? Tôi chọn `truck` theo định nghĩa v0 "có thùng hàng riêng".
4. Xe giả tưởng chạy được trên đường (img_019) tính là `car` hay là vật trưng bày?
5. Xe đạp có cần lớp riêng không, hay vẫn bỏ (img_006, img_023)?
6. Ngưỡng 30% thân để gán `person` trong xe hở và đám đông có hợp lý với người ngồi sau xe máy không (img_005)?
