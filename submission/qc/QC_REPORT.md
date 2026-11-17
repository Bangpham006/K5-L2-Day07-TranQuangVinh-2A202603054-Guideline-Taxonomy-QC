# Báo cáo QC — Nhóm 101-102 (Trần Quang Vinh - Phạm Công Bằng)

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Huấn luyện và đánh giá mô hình thị giác máy tính phát hiện vật thể 2D cho hệ thống camera giao thông và xe tự hành; giới hạn tập 12 ảnh pilot dùng cho thực hành chuẩn hóa quy trình đặc tả, hiệu chuẩn độ đồng thuận và kiểm định chất lượng (QC), chưa đại diện cho phân bố giao thông toàn diện mọi điều kiện thời tiết.
- Nguồn và phép sử dụng dữ liệu: Dữ liệu ảnh thực hành Day 07 thu thập từ tập mở, thông tin bản quyền và giấy phép ghi nhận chi tiết tại data/ATTRIBUTION.csv.
- Manifest và checksum nhãn cuối: batch_manifest.json (SHA-256: e1de0089641edd53243ede009a99502a7db8929454bacafc9ac4a086d26381e7), labels_calibrated.json (SHA-256: bbb1d89ba907693c04778b77ed93600424d1fa5fcb0598a1ddf8b4f859dbaebd).
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán độc lập dùng guideline_v1.0_agreed.md và taxonomy.json (v1.0); khi bàn giao dùng guideline_v1.1_calibrated.md và taxonomy.json (v1.0, giữ nguyên cấu trúc lớp và thuộc tính, làm rõ chi tiết thực thi bounding box và vùng ignore).

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 24/24 lượt gán (đạt 100%, 2 thành viên 101 và 102 hoàn thành đủ 12/12 ảnh pilot).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 47 cặp box ghép (mẫu số), IoU trung bình 0.824, tỷ lệ đồng thuận phân lớp 97.9% (đạt ngưỡng >= 95% của lab).
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 29 box chưa ghép (101 thừa/102 thiếu hoặc ngược lại), 0 box bị loại do ignore, 18 ca bất đồng vùng ignore_region ở các ảnh đông xe xa (img_006, img_008).
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Ảnh img_017 tượng danh nhân cưỡi ngựa bằng đồng và img_027 bia đá khắc hình xe bán tải - cả hai thành viên có thể cùng nhận dạng nhầm thành person hoặc car/truck nếu không áp dụng quy tắc R-01 (chỉ gán đối tượng vật lý thực, loại trừ tượng nghệ thuật/bia đá). Cần kiểm tra kỹ danh sách annotation rỗng cho hai ảnh này.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Đã xử lý toàn bộ 93 ca trong calibration_candidates.csv (C001 đến C093), tham chiếu chi tiết các quy tắc R-02, R-03, R-06, R-07, R-08, R-10, R-14, R-16; đã ghi nhận đầy đủ quyết định và kiểm lại trong calibration_decisions.csv bởi 101,102.
- Tác động v1.1 lên nhãn và ảnh: Bản v1.1 là đính chính biên tập (editorial), không làm thay đổi ngữ nghĩa danh mục lớp; phạm vi hiệu chỉnh gồm 8 ảnh có ca cờ (img_002, img_004, img_005, img_006, img_008, img_012, img_026, img_040), trong đó ca nổi bật C080 trên img_012 chốt xe bán tải là truck theo R-10 và xóa bỏ hoàn toàn 7 cờ can_review.
- Tham chiếu bằng chứng khi Rework: Không phát sinh vòng Rework; toàn bộ 8 ảnh sau khi xuất consensus_flagged.json đã được kiểm tra, căn chỉnh và tích hợp vào labels_calibrated_4f92901627ec222c.json trước khi build bản cuối.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 102, 17-11-2026
- Đã duyệt / cả lô: 12/12 ảnh (100% lô pilot)
- Ảnh lỗi / ảnh đã duyệt: 0/12 = 0.0% (đạt ngưỡng <= 2% của taxonomy rules)
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/84 = 0.0%
- Vật bỏ sót, ảnh và vùng: 0 vật bỏ sót
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Đã duyệt 8/8 ảnh ca khó và 4/4 ảnh còn lại, 0 lỗi lặp, xem chi tiết tại submission/qc/reviews/round-01/qc_review.csv.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Lỗi máy chặn được là sai định dạng thuộc tính hoặc bbox âm/vượt kích thước ảnh (validate_labels.py); lỗi cần người đọc ảnh là phân biệt xe thật với tượng đá (img_027) hoặc tranh tường vẽ người (img_026) mà máy không tự phát hiện được nếu chỉ nhìn hình học box.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Lô nhãn đạt chất lượng 100% sau hiệu chuẩn, 0 lỗi trên 12 ảnh kiểm định vòng 1, đầy đủ manifest và checksum; bàn giao cho nhóm phát triển mô hình perception để huấn luyện và đánh giá trên tập dữ liệu pilot.
- Nếu còn việc: Không còn tồn đọng, toàn bộ 93 ca calibration đã được giải quyết dứt điểm và không còn cờ can_review nào.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca C080 (ảnh img_012.jpg, xe bán tải bên phải) áp dụng quy tắc R-10, đối tượng #72 trong labels_calibrated.json được gán đúng nhãn truck kèm truncated=yes, kiểm tra khớp hoàn toàn giữa ảnh, guideline v1.1 và file nhãn bàn giao.
