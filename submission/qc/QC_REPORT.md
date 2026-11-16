# Báo cáo QC — G0001

> **BẢN NHÁP.** Các dòng ghi "CHỜ PHIẾU VÒNG 1" sẽ điền sau khi 222 và 333 nộp `reviews/round-01/qc_review.csv` và chạy `qc_report.py --round 1`. Xóa khối này trước khi nộp.

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Nhãn bounding box 2D cho người và phương tiện trên ảnh đường phố, dùng để thực hành đặc tả và QC nhãn trong Day 07. Lô 40 ảnh được tuyển chọn để luyện ca biên (tượng, phản chiếu, xe chở xe, đám đông), không đại diện cho dữ liệu camera giao thông vận hành; không dùng để huấn luyện hay đánh giá mô hình thật nếu chưa kiểm thêm.
- Nguồn và phép sử dụng dữ liệu: 40 ảnh từ Wikimedia Commons, ghi công đủ trong `data/ATTRIBUTION.csv` (CC BY-SA 4.0: 15, CC BY 2.0: 7, CC BY-SA 2.0: 6, CC BY 4.0: 5, CC0: 3, CC BY-SA 3.0: 3, public domain: 1). Ảnh CC BY-SA khi chia sẻ lại phải giữ ghi công và cùng giấy phép; không sửa hay đổi tên ảnh.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json` (SHA-256 `cf2705b4f8001cfd51ff49907a57169fa09eb7dee59f9ec82a5f29171364d8eb`); nhãn `submission/qc/labels_calibrated.json` (SHA-256 `964ebc45177c3ac7b1198a7a1392e9023ce629af257c659632eb65b1a4a13a14`), 40 ảnh, 508 annotation. Nếu vòng QC dẫn tới Rework thì hai checksum này đổi và phải cập nhật.
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán độc lập: `guideline_v1.0_agreed.md` + `taxonomy.json` v1.0 (9 lớp). Khi bàn giao: `guideline_v1.1_calibrated.md` + `taxonomy_v1.1.json` v1.1 (10 lớp, thêm `bus`). Thay đổi loại `semantic` (`submission/guideline_change.json`), cả lô 40 ảnh được rà lại. Nhóm đã sửa hàm `xyxy_to_xywh` trong `scripts/common.py` (lỗi làm tròn làm `consensus_draft.json` có box vượt ảnh 0,01 px ở img_001); IAA chạy lại cho kết quả không đổi, 20 bài kiểm thử của lab vẫn qua.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt (40 ảnh × 2 người). 111: 26 ảnh, 222: 27 ảnh, 333: 27 ảnh; chia bằng `split_assign.py` seed 42; cặp 111|222: 13 ảnh, 111|333: 13 ảnh, 222|333: 14 ảnh.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 230 cặp box ghép (ghép 1-1 không phân biệt lớp, IoU ≥ 0,5) là mẫu số của cả hai số đo. IoU TB 0,814 (ngưỡng 0,85), cùng lớp 91,3% (ngưỡng 95%): **chưa đạt**. Theo cặp: 111|222 0,797 / 91,2% (68 cặp); 111|333 0,822 / 96,2% (106 cặp); 222|333 0,819 / 82,1% (56 cặp). Cùng `occluded_level` 51,7%, cùng `truncated` 92,6%.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 157 / 134 box chưa ghép (người A / người B, cộng mọi cặp), không nằm trong mẫu số IoU và đồng thuận lớp, nên mức đồng thuận thật thấp hơn hai số trên. 0 box bị loại do nằm trong vùng ignore. 167 ca bất đồng vùng ignore, chủ yếu vì 222 vẽ box từng người trong đám đông còn 111 và 333 dùng `ignore_region` (img_019: khoảng 65 box). Tổng 619 ca bất thường trên 34 ảnh (bỏ sót/thừa 164, sai thuộc tính 119, IoU thấp 105, `can_review` 44, sai lớp 20).
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: CHỜ PHIẾU VÒNG 1 — nhóm chưa nhận bộ ca tham chiếu của coach. Sẽ đối chiếu các ảnh hai người gán giống nhau với tham chiếu và với guideline, ghi ảnh, điều khoản R-xx và cách kiểm.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 619/619 ca có một dòng trong `submission/qc/calibration_decisions.csv` (quyết định, rule R-04 / R-09 / R-10 / R-12 / mục 2.1 / mục 2.3, phiên bản v1.1, phạm vi: 192 ca `lo`, 427 ca `anh`). Người chốt: 111, theo quy tắc cả nhóm thống nhất. Bản sửa: `submission/qc/labels_calibrated_15eca0fbc896b4e9.json` (40 ảnh, 508 box). Kiểm lại: CHỜ PHIẾU VÒNG 1 — cột `kiem_lai` đang ghi người được phân công (222 hoặc 333), chưa có kết quả.
- Tác động v1.1 lên nhãn và ảnh: Phạm vi cả lô 40 ảnh. (1) Thêm lớp `bus`: mọi vùng `ignore_region` dùng cho xe buýt ở v1.0 được gán lại thành `bus` (10 box: img_006, 008, 009, 021, 031, 037, 040). (2) R-10: mỗi người/xe thấy từ 30% thân và cao từ 20 px có box riêng, `ignore_region` chỉ cho lõi không tách được (img_032: 25 người ở trạm xe buýt được vẽ riêng). (3) R-04: box `motorcycle` lấy cả gương và tay lái; 26 box `rider` ôm cả người lẫn xe (của 222) bị bỏ và tách lại. (4) `occluded_level` lấy mức thấp hơn khi phân vân. Không cần mapping cho lớp cũ vì 9 lớp v1.0 giữ nguyên tên.
- Tham chiếu bằng chứng khi Rework: CHỜ PHIẾU VÒNG 1.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: CHỜ PHIẾU VÒNG 1 (dự kiến 222: img_001–img_020; 333: img_021–img_040)
- Đã duyệt / cả lô: CHỜ PHIẾU VÒNG 1 (gói duyệt sinh cho 40/40 ảnh)
- Ảnh lỗi / ảnh đã duyệt: CHỜ PHIẾU VÒNG 1
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: CHỜ PHIẾU VÒNG 1
- Vật bỏ sót, ảnh và vùng: CHỜ PHIẾU VÒNG 1
- Ca khó / phần còn lại, lỗi lặp và ví dụ: CHỜ PHIẾU VÒNG 1
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn được: box vượt mép ảnh 0,01 px ở img_001 trong `consensus_draft.json` (`build_final.py` dừng, nhóm sửa hàm làm tròn). Cần người đọc ảnh: CHỜ PHIẾU VÒNG 1.

## 5. Quyết định bàn giao

- Quyết định: CHỜ PHIẾU VÒNG 1
- Lý do, người nhận và phạm vi sử dụng: CHỜ PHIẾU VÒNG 1
- Nếu còn việc: CHỜ PHIẾU VÒNG 1
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: CHỜ PHIẾU VÒNG 1
