# Guideline gán nhãn dữ liệu 2D

Phiên bản: v0.1 cá nhân
Thành viên thực hiện: 102
Ngày cập nhật: 16-11-2026

## 1. Mục tiêu bộ dữ liệu

Dữ liệu gán nhãn hộp chữ nhật 2D này phục vụ trực tiếp cho mô hình thị giác máy tính trên xe tự hành và camera giao thông thông minh. Mục tiêu cao nhất là phát hiện chính xác các phương tiện và người thực tế đang tham gia lưu thông để đảm bảo an toàn.

Nếu gán sai nhãn, hậu quả trong thực tế rất nghiêm trọng:
- Bỏ sót người đi bộ hoặc người đi xe máy có thể dẫn đến việc hệ thống không nhận biết chướng ngại vật và gây va chạm nguy hiểm.
- Nhận nhầm các vật thể tĩnh như tượng nghệ thuật, hình ảnh trên biển quảng cáo hoặc bóng phản chiếu trong gương chiếu hậu sẽ khiến xe tự hành phanh gấp bất ngờ, gây nguy cơ tai nạn liên hoàn cho các xe đi phía sau.
- Vẽ hộp không ôm sát hoặc gộp chung xe với người làm sai lệch việc ước tính khoảng cách trong không gian thực tế.

## 2. Đặc tả lớp và thuộc tính

### Các lớp đối tượng chính

Lớp car:
- Định nghĩa: Ô tô con từ bốn đến chín chỗ, bao gồm xe sedan, hatchback, SUV và crossover.
- Ví dụ: Xe hơi gia đình, xe taxi thông thường.
- Phản ví dụ: Không gán lớp này cho xe bán tải gắn nắp thùng hàng kín cỡ lớn hoặc xe tải nhỏ.

Lớp truck:
- Định nghĩa: Xe tải chở hàng có buồng lái và thùng hàng tách rời, hoặc xe đầu kéo chuyên dụng.
- Ví dụ: Xe chở ô tô hai tầng, xe tải chở vật liệu xây dựng.
- Phản ví dụ: Không gán cho xe van thùng kín hoặc xe bán tải chở người.

Lớp van:
- Định nghĩa: Xe thùng kín cỡ vừa và nhỏ liền khối dùng chở người hoặc chở hàng bưu chính.
- Ví dụ: Xe van giao hàng, xe cứu thương cỡ nhỏ.
- Phản ví dụ: Không gán cho xe buýt cỡ lớn hoặc xe SUV năm chỗ.

Lớp motorcycle:
- Định nghĩa: Xe hai bánh hoặc ba bánh có động cơ, bao gồm xe máy phổ thông, xe tay ga và mô tô phân khối lớn.
- Ví dụ: Xe máy lưu thông trên đường, xe mô tô độ khung sườn.
- Phản ví dụ: Không gán cho xe đạp hoặc xe xích lô không có động cơ.

Lớp person:
- Định nghĩa: Con người thật ở trạng thái sống, đang đứng, đi bộ, ngồi hoặc đang lái xe.
- Ví dụ: Người đi bộ qua đường, người điều khiển xe máy, hành khách ngồi sau xe.
- Phản ví dụ: Không gán cho tượng điêu khắc, ma nơ canh trong tủ kính thời trang hoặc hình người in trên tranh ảnh quảng cáo.

Lớp ignore_region:
- Định nghĩa: Vùng đánh dấu khu vực quá đông đúc, nhòe mờ hoặc quá xa khiến việc tách riêng từng vật không còn chính xác.
- Ví dụ: Cụm đám đông đi bộ ở xa phía hậu cảnh, cụm xe cộ kẹt dính chặt vào nhau ở đường chân trời.

### Xử lý các đối tượng chưa có trong danh mục v0

Trong quá trình khảo sát ảnh thực tế, có một số phương tiện xuất hiện nhiều lần nhưng tài liệu gốc chưa có lớp tương ứng:
- Xe ba bánh tuk tuk: Hiện tại tạm thời tick cờ can_review để đánh dấu xem lại. Tôi đề xuất cả nhóm trong buổi họp thống nhất sẽ tạo thêm lớp tuk_tuk riêng vào danh mục nhãn.
- Xe buýt chở khách cỡ lớn: Tạm thời đánh dấu cờ can_review. Tôi đề xuất nhóm bổ sung thêm lớp bus vào danh mục vì kích thước xe buýt khác hoàn toàn so với ô tô con và xe van.
- Phương tiện thô sơ như xích lô và xe đạp: Không gán nhãn vì không thuộc nhóm phương tiện cơ giới theo mục tiêu bài toán.

### Quy định các thuộc tính đi kèm

Thuộc tính mức độ che khuất:
- Mức 0: Đối tượng thấy rõ hoàn toàn, không bị che khuất.
- Mức 1: Đối tượng bị che khuất nhẹ, diện tích bị che từ một phần trăm đến ba mươi phần trăm.
- Mức 2: Đối tượng bị che khuất vừa phải, diện tích bị che từ ba mươi mốt phần trăm đến bảy mươi phần trăm.
- Mức 3: Đối tượng bị che khuất rất nặng trên bảy mươi phần trăm nhưng mắt người vẫn nhận biết được dáng hình.

Thuộc tính cắt mép ảnh:
- Chọn no nếu toàn bộ đối tượng nằm trọn vẹn trong khung hình hoặc chỉ chạm mép ảnh không đáng kể.
- Chọn yes nếu thân xe hoặc cơ thể người bị đường viền mép ảnh cắt mất từ năm phần trăm diện tích trở lên.

Thuộc tính xem lại can_review:
- Bật cờ này khi người gán nhãn thấy trường hợp quá khó, hình dạng dị biệt hoặc chưa chắc chắn về ranh giới để người kiểm tra xem lại sau.

## 3. Quy tắc vẽ hộp bounding box

Nguyên tắc bao trọn phần nhìn thấy:
- Hộp vẽ phải ôm khít phần pixel thực sự nhìn thấy của đối tượng, không ước lượng vẽ bù vào phần bị che khuất dưới mặt đất hoặc sau vật cản khác.

Người điều khiển xe máy:
- Tách thành hai hộp độc lập gồm một hộp person ôm trọn người lái từ mũ bảo hiểm đến gác chân, và một hộp motorcycle ôm trọn thân xe từ bánh trước đến bánh sau.
- Hai hộp này được phép đè lên nhau ở khu vực yên xe.
- Nếu xe chở thêm người ngồi sau thì vẽ thêm một hộp person riêng biệt cho người ngồi sau.

Xe tải chở ô tô nhiều tầng:
- Vẽ một hộp truck bao toàn bộ thân xe tải kéo cùng sàn chở hàng.
- Đồng thời vẽ từng hộp car riêng cho mỗi chiếc ô tô con đang nằm trên các tầng sàn vì các xe con này đều có đầy đủ đặc trưng thị giác của ô tô thật.

Các trường hợp vật thể nhân tạo và hình ảnh ảo:
- Tượng điêu khắc bằng đá hoặc kim loại: Không gán nhãn.
- Hình người in trên tấm pano quảng cáo: Không gán nhãn.
- Ma nơ canh trưng bày thời trang: Không gán nhãn.
- Xe mô hình đồ chơi thu nhỏ trong tủ kính: Không gán nhãn.
- Hình ảnh phương tiện phản chiếu qua gương chiếu hậu: Không gán nhãn vì đây là ảnh ảo, không có vị trí vật lý thật tại tọa độ đó.

Xử lý hàng hóa cồng kềnh chở quá khổ:
- Đối với xe máy hoặc xe tuk tuk chở đống bao tải hàng hóa to gấp nhiều lần thân xe, hiện tại hộp vẽ sẽ ôm trọn thân xe và cả khối hàng để đảm bảo bao trùm chướng ngại vật thực tế.
- Tuy nhiên việc này làm méo mó tỷ lệ khung hình chuẩn của xe. Vì vậy tôi đề xuất trong buổi họp nhóm ở pha tới nên xem xét tạo thêm một lớp cargo để tách riêng khối hàng cồng kềnh với thân xe.

Ngưỡng chiều cao tối thiểu:
- Đối tượng có chiều cao dưới hai mươi pixel thì không vẽ hộp đơn lẻ. Nếu xuất hiện thành từng cụm phương tiện hoặc đám đông ở xa thì khoanh chung một hộp lớn thuộc lớp ignore_region.

## 4. Bảng các ca khó khảo sát trong bộ ảnh

Dưới đây là các tình huống thực tế khó xử lý được tôi rút ra sau khi kiểm tra toàn bộ bốn mươi ảnh:

| Tên ảnh | Tình huống phát hiện | Quy tắc áp dụng | Quyết định gán nhãn | Lý do quyết định |
|---|---|---|---|---|
| img_002.jpg | Xe tải chở tám ô tô con trên hai tầng khung sàn | R-03 | Gán một hộp truck cho xe tải và gán tám hộp car cho tám xe con | Các xe con trên sàn có đầy đủ đặc trưng ô tô thật, mô hình cần nhận dạng để tránh va quẹt |
| img_004.jpg | Hai chiếc xe tuk tuk ba bánh màu xanh đỗ ven đường | R-04 | Tạm tick cờ can_review và đề xuất nhóm tạo thêm lớp tuk_tuk | Tài liệu v0 thiếu lớp xe ba bánh, xếp vào ô tô hay xe máy đều không chuẩn |
| img_005.jpg | Dòng người đi xe máy rất đông đúc tại giao lộ | R-02 | Tách riêng từng người lớp person và từng xe lớp motorcycle | Giúp mô hình phân biệt rõ ràng giữa người đi bộ với người đang điều khiển phương tiện |
| img_006.jpg | Đoàn xích lô du lịch chở khách trên đường phố Huế | R-05 | Gán người đạp và khách vào lớp person, không gán hộp cho thân xe xích lô | Xích lô là phương tiện thô sơ không có động cơ, không nằm trong mục tiêu phương tiện cơ giới |
| img_011.jpg | Tượng quái vật khổng lồ bên cạnh dàn xe máy độ khung xương thép | R-06 | Không gán tượng quái vật, còn các xe máy độ dị thường vẫn gán motorcycle kèm cờ can_review | Tượng quái vật là mô hình nhân tạo tĩnh, còn xe độ dù hình thù kỳ dị nhưng bản chất cơ học vẫn là xe máy |
| img_017.jpg | Tượng đồng công tước cưỡi ngựa giữa quảng trường | R-01 | Không gán nhãn person cho tượng người và không gán cho ngựa | Đây là công trình điêu khắc nghệ thuật bất động, không tham gia vào luồng giao thông |
| img_018.jpg | Tấm biển quảng cáo tranh cử khổ lớn in hình chân dung người | R-01 | Không gán nhãn person | Hình ảnh in phẳng trên mặt biển bạt, không phải người thật đang đi lại |
| img_020.jpg | Hình ảnh ô tô và cầu vồng phản chiếu qua gương chiếu hậu | R-01 | Không gán nhãn ô tô phản chiếu trong mặt gương | Đây là hình ảnh phản xạ quang học, không có vị trí thực trên mặt đường |
| img_027.jpg | Khối đá điêu khắc tạc hình xe bán tải trong nghĩa trang | R-01 | Không gán nhãn car hay truck | Là bia mộ bằng đá tĩnh, không có động cơ và không có khả năng di chuyển |
| img_033.jpg | Ma nơ canh hình người đứng trong tủ kính cửa hàng thời trang | R-01 | Không gán nhãn person | Là vật thể búp bê trưng bày hàng hóa, không phải tác nhân sống |
| img_035.jpg | Xe tuk tuk chở núi bao tải hàng hóa cao gấp ba lần thân xe | R-07 | Tạm vẽ hộp bao trùm và đề xuất nhóm tạo thêm lớp hàng hóa riêng | Tránh để khối hàng khổng lồ làm sai lệch hình thái kích thước chuẩn của phương tiện |
| img_037.jpg | Xe bán tải cá nhân gắn nắp thùng cao kín phía sau | R-08 | Tạm gán lớp car và tick cờ can_review để nhóm cùng thảo luận | Dáng xe nửa giống xe bán tải du lịch, nửa giống xe van chở hàng cỡ nhỏ |
| img_038.jpg | Tủ kính bày bán các mô hình xe đồ chơi thu nhỏ | R-01 | Không gán bất kỳ hộp nhãn nào | Đồ chơi trẻ em kích thước nhỏ, không phải phương tiện giao thông ngoài đường |

## 5. Kiểm soát chất lượng và lịch sử phiên bản

- Việc giải quyết các trường hợp nghi ngờ: Người gán nhãn bật cờ can_review khi gặp các tình huống phức tạp. Tất cả các trường hợp này sẽ được đưa ra buổi họp nhóm ở pha tiếp theo để hai thành viên cùng chốt hướng giải quyết thống nhất.
- Lịch sử cập nhật:
  - Bản v0 ban đầu: Định nghĩa sơ khai, thiếu nhiều tình huống thực tế và mức che khuất còn cảm tính.
  - Bản v0.1 cá nhân của thành viên 102: Xác lập số liệu phần trăm che khuất đo được, đưa ra quy tắc tách người với xe máy, phân định rõ tượng và ảnh ảo, đồng thời bổ sung phân tích chi tiết mười ba ca khó trong bộ ảnh.
