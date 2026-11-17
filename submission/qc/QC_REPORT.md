# Báo cáo QC — G0001

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Nhãn bounding box 2D cho người và phương tiện trên ảnh đường phố, dùng để thực hành đặc tả và QC nhãn trong Day 07. Lô 40 ảnh được tuyển chọn để luyện ca biên (tượng, phản chiếu, xe chở xe, đám đông), không đại diện cho dữ liệu camera giao thông vận hành; không dùng để huấn luyện hay đánh giá mô hình thật nếu chưa kiểm thêm.
- Nguồn và phép sử dụng dữ liệu: 40 ảnh từ Wikimedia Commons, ghi công đủ trong `data/ATTRIBUTION.csv` (CC BY-SA 4.0: 15, CC BY 2.0: 7, CC BY-SA 2.0: 6, CC BY 4.0: 5, CC0: 3, CC BY-SA 3.0: 3, public domain: 1). Ảnh CC BY-SA khi chia sẻ lại phải giữ ghi công và cùng giấy phép; không sửa hay đổi tên ảnh.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json` (SHA-256 `d5821f32f05154a648cdc9f4be9b8799bba662fb75f8bd6046ce43217f315696`); nhãn `submission/qc/labels_calibrated.json` (SHA-256 `dcd737a85025afa244e8db0b94f252ec49f4dad19cfd00a7f7883b2e0117ab08`), 40 ảnh, 509 annotation (bản sau Rework vòng 1). Bản trước Rework có SHA-256 `964ebc45177c3ac7b1198a7a1392e9023ce629af257c659632eb65b1a4a13a14`, 508 annotation, được giữ trong `reviews/round-01/`. Nếu vòng 2 dẫn tới Rework tiếp thì hai checksum này đổi và phải cập nhật.
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán độc lập: `guideline_v1.0_agreed.md` + `taxonomy.json` v1.0 (9 lớp). Khi bàn giao: `guideline_v1.1_calibrated.md` + `taxonomy_v1.1.json` v1.1 (10 lớp, thêm `bus`). Thay đổi loại `semantic` (`submission/guideline_change.json`), cả lô 40 ảnh được rà lại. Nhóm đã sửa hàm `xyxy_to_xywh` trong `scripts/common.py` (lỗi làm tròn làm `consensus_draft.json` có box vượt ảnh 0,01 px ở img_001); IAA chạy lại cho kết quả không đổi, 20 bài kiểm thử của lab vẫn qua.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt (40 ảnh × 2 người). 111: 26 ảnh, 222: 27 ảnh, 333: 27 ảnh; chia bằng `split_assign.py` seed 42; cặp 111|222: 13 ảnh, 111|333: 13 ảnh, 222|333: 14 ảnh.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 230 cặp box ghép (ghép 1-1 không phân biệt lớp, IoU ≥ 0,5) là mẫu số của cả hai số đo. IoU TB 0,814 (ngưỡng 0,85), cùng lớp 91,3% (ngưỡng 95%): **chưa đạt**. Theo cặp: 111|222 0,797 / 91,2% (68 cặp); 111|333 0,822 / 96,2% (106 cặp); 222|333 0,819 / 82,1% (56 cặp). Cùng `occluded_level` 51,7%, cùng `truncated` 92,6%.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 157 / 134 box chưa ghép (người A / người B, cộng mọi cặp), không nằm trong mẫu số IoU và đồng thuận lớp, nên mức đồng thuận thật thấp hơn hai số trên. 0 box bị loại do nằm trong vùng ignore. 167 ca bất đồng vùng ignore, chủ yếu vì 222 vẽ box từng người trong đám đông còn 111 và 333 dùng `ignore_region` (img_019: khoảng 65 box). Tổng 619 ca bất thường trên 34 ảnh (bỏ sót/thừa 164, sai thuộc tính 119, IoU thấp 105, `can_review` 44, sai lớp 20).
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: CHỜ BỘ CA THAM CHIẾU CỦA COACH — nhóm chưa nhận bộ ca tham chiếu của coach. Sẽ đối chiếu các ảnh hai người gán giống nhau với tham chiếu và với guideline, ghi ảnh, điều khoản R-xx và cách kiểm.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 619/619 ca có một dòng trong `submission/qc/calibration_decisions.csv` (quyết định, rule R-04 / R-09 / R-10 / R-12 / mục 2.1 / mục 2.3, phiên bản v1.1, phạm vi: 192 ca `lo`, 427 ca `anh`). Người chốt: 111, theo quy tắc cả nhóm thống nhất. Bản sửa trước vòng QC: 40 ảnh, 508 box. Sau Rework vòng 1, bảng có thêm 24 dòng Q01–Q24 (643 dòng): Q01–Q10 sửa lỗi phiếu, Q11–Q24 bỏ cờ `can_review` ở 14 ảnh; bản sửa hiện tại `submission/qc/labels_calibrated_b8c74ff109d41c88.json` (40 ảnh, 509 box). Kiểm lại: cột `kiem_lai` ghi kết quả vòng 1 của 222 (img_001–020) và 333 (img_021–040); 222 còn ghi `chưa đạt` cho box chạm mép ở img_021 và hai box `can_review` chưa kết luận (img_023, img_037), đã xử lý ở Q08 và Q11–Q24. Kết quả kiểm trên bản sửa chờ vòng 2.
- Tác động v1.1 lên nhãn và ảnh: Phạm vi cả lô 40 ảnh. (1) Thêm lớp `bus`: mọi vùng `ignore_region` dùng cho xe buýt ở v1.0 được gán lại thành `bus` (10 box: img_006, 008, 009, 021, 031, 037, 040; thêm 1 box ở img_040 sau Rework, tổng 11). (2) R-10: mỗi người/xe thấy từ 30% thân và cao từ 20 px có box riêng, `ignore_region` chỉ cho lõi không tách được (img_032: 25 người ở trạm xe buýt được vẽ riêng). (3) R-04: box `motorcycle` lấy cả gương và tay lái; 26 box `rider` ôm cả người lẫn xe (của 222) bị bỏ và tách lại. (4) `occluded_level` lấy mức thấp hơn khi phân vân. Không cần mapping cho lớp cũ vì 9 lớp v1.0 giữ nguyên tên.
- Tham chiếu bằng chứng khi Rework: vòng 1 giữ nguyên tại `submission/qc/reviews/round-01/` (phiếu, mẫu, snapshot nhãn `964ebc45…`, `qc_report_auto.md`, `qc_result.json`). Vùng sửa: img_001, 003, 005, 008, 010, 018, 019, 021 (`truncated` theo R-03); img_005 (tách box rider gộp, R-04, R-10); img_029 (mở rộng hai box rider, R-14); img_040 (#506 đổi sang `bus`, #508 xóa vì trùng #504, R-12, R-01). Nhãn mới và vòng 2: `reviews/round-02/`. Chưa có thay đổi guideline mới nên v1.1 giữ nguyên.

## 4. Kết quả duyệt

- Vòng QC: 2 (bản nhãn bàn giao). Số liệu vòng 1 để đối chiếu bên dưới.
- Người review và ngày: vòng 1, ngày 17/11/2026: 222 duyệt img_001–img_020, 333 duyệt img_021–img_040. Vòng 2: CHỜ PHIẾU VÒNG 2 (dự kiến đổi vai: 222 duyệt img_021–img_040, 333 duyệt img_001–img_020).
- Đã duyệt / cả lô: vòng 1: 40/40 ảnh. Vòng 2: CHỜ PHIẾU VÒNG 2.
- Ảnh lỗi / ảnh đã duyệt: vòng 1: 9/40 = 22,5% (ngưỡng 2%; nhóm `ca_kho` 9/34, nhóm `con_lai` 0/6). Vòng 2: CHỜ PHIẾU VÒNG 2.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: vòng 1: 16/475 = 3,4% (không tính `ignore_region`). Vòng 2: CHỜ PHIẾU VÒNG 2.
- Vật bỏ sót, ảnh và vùng: vòng 1: 0. Vòng 2: CHỜ PHIẾU VÒNG 2.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: vòng 1: 10 lỗi `sai_thuoc_tinh` (đều là `truncated=no` ở box chạm mép, R-03: img_001 #3, img_003 #22 và #30, img_005 #42, img_008 #84, img_010 #134, img_018 #194, img_019 #209, #213, #214), 4 lỗi `sai_vi_tri_kich_thuoc` (img_005 #43 và #45, R-04; img_029 #346 và #347, R-14), 1 lỗi `sai_lop` (img_040 #506, R-12), 1 lỗi `thua` (img_040 #508, R-01). Lỗi `truncated` lặp từ 10/16 lỗi, nên nhóm rà thêm các box chạm mép và sửa img_021 #244, #256 ngoài phiếu. Nhãn cuối vòng 1 còn 17 box `can_review`, báo cáo tự động ghi HOLD; nhóm xem lại từng box và bỏ cờ ở Q11–Q24. Vòng 2: CHỜ PHIẾU VÒNG 2.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn được: box vượt mép ảnh 0,01 px ở img_001 trong `consensus_draft.json` (`build_final.py` dừng, nhóm sửa hàm làm tròn); và `qc_report.py` chặn nhãn cuối còn cờ `can_review`. Cần người đọc ảnh: box rider img_005 #45 gộp ba người trên xe máy, kích thước hợp lệ nên máy không bắt được; 222 phát hiện khi đọc ảnh..

## 5. Quyết định bàn giao

- Quyết định: CHỜ PHIẾU VÒNG 2 (vòng 1: HOLD theo `qc_report.py`, kèm điều kiện Rework vì 22,5% ảnh lỗi vượt ngưỡng 2%)
- Lý do, người nhận và phạm vi sử dụng: CHỜ PHIẾU VÒNG 2
- Nếu còn việc: CHỜ PHIẾU VÒNG 2. Việc đã biết: xin bộ ca tham chiếu của coach để điền mục 2; xác nhận với 333 các nhận xét img_012 (car hay truck, quy tắc pickup là truck) và img_001, img_002 (R-05 chỉ một box truck) vì chưa có trong phiếu.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: ví dụ vòng 1: img_040 #506 → dòng phiếu `sai_lop`, R-12 → `calibration_decisions.csv` Q10 → box `bus` trong `labels_calibrated.json`. Người ngoài nhóm kiểm một ca: CHỜ PHIẾU VÒNG 2
