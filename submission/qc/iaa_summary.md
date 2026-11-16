# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 3
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 230 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 230 | 0.814 | 91.3% | 51.7% | 92.6% | 157/134 | 0 | CHƯA ĐẠT |
| 111|222 | 13 | 68 | 0.797 | 91.2% | 54.4% | 92.6% | 39/84 | 0 | CHƯA ĐẠT |
| 111|333 | 13 | 106 | 0.822 | 96.2% | 50.9% | 90.6% | 34/25 | 0 | CHƯA ĐẠT |
| 222|333 | 14 | 56 | 0.819 | 82.1% | 50.0% | 96.4% | 84/25 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 619 ({'bo_sot_thua': 164, 'co_can_review': 44, 'iou_thap': 105, 'sai_thuoc_tinh': 119, 'bat_dong_ignore': 167, 'sai_lop': 20}), trên 34 ảnh cần calibration.
