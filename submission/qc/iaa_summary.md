# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **12**; số người gán: 2
- Lượt gán đủ **24/24**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 47 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 12 | 47 | 0.824 | 97.9% | 72.3% | 83.0% | 29/8 | 0 | CHƯA ĐẠT |
| 101|102 | 12 | 47 | 0.824 | 97.9% | 72.3% | 83.0% | 29/8 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 93 ({'bo_sot_thua': 25, 'iou_thap': 24, 'sai_thuoc_tinh': 18, 'co_can_review': 7, 'bat_dong_ignore': 18, 'sai_lop': 1}), trên 8 ảnh cần calibration.
