# Guideline gán nhãn dữ liệu 2D — v1.0 (Bản đồng thuận nhóm)

- Phiên bản: v1.0
- Ngày thống nhất: 16-11-2026
- Nhóm thực hiện: 101 (Trần Quang Vinh - Leader), 102 (Phạm Công Bằng - Reviewer)
- Phạm vi áp dụng: Toàn bộ 40 ảnh trong tập dữ liệu `data/images/`, trọng tâm là 12 ảnh thuộc tập pilot
- Công cụ sử dụng: CVAT, xuất dữ liệu định dạng COCO 1.0

---

## 1. Mục tiêu bộ dữ liệu

Dữ liệu gán nhãn hộp chữ nhật 2D phục vụ trực tiếp cho việc huấn luyện mô hình thị giác máy tính phát hiện đối tượng trên xe tự hành và hệ thống camera giám sát giao thông thông minh. Mục tiêu là định vị và phân loại chính xác các tác nhân sống cùng phương tiện cơ giới, thô sơ đang thực tế hiện diện trong luồng lưu thông hoặc trên lề đường.

Mức độ nguy hiểm khi xảy ra lỗi gán nhãn:

| Loại lỗi | Hậu quả thực tế | Mức độ rủi ro |
|---|---|---|
| Bỏ sót người thật, đặc biệt là người đi bộ, người bị che khuất một phần hoặc người ngồi sau xe máy | Hệ thống tự hành không phát hiện chướng ngại vật sống, dẫn tới nguy cơ va chạm chết người | Nghiêm trọng nhất |
| Gán nhầm vật thể tĩnh nhân tạo như tượng đài, tranh ảnh quảng cáo, ma nơ canh hoặc bóng phản chiếu qua gương thành người hoặc xe thật | Hệ thống phanh gấp ngoài ý muốn hoặc đánh lái bất ngờ, gây tai nạn liên hoàn cho các phương tiện phía sau | Rất cao |
| Sai phân lớp phương tiện giữa xe con, xe bán tải, xe tải và xe van | Ảnh hưởng đến việc thống kê lưu lượng và phân luồng làn đường | Trung bình |
| Vẽ hộp lỏng lẻo, lệch tâm hoặc bao trùm sai khoảng nhìn thấy | Làm sai lệch thuật toán ước tính khoảng cách và độ sâu trong không gian ba chiều | Trung bình |

Nguyên tắc cốt lõi: Vật thể thật có khả năng tham gia vào luồng giao thông thì gán nhãn; hình ảnh mô phỏng tĩnh và ảnh ảo quang học thì không gán nhãn.

---

## 2. Đặc tả lớp và thuộc tính

### 2.1 Danh mục lớp đối tượng

Mỗi đối tượng hợp lệ trong ảnh thuộc về đúng một lớp duy nhất. Nhóm đã thống nhất mở rộng danh mục ban đầu v0 bằng việc bổ sung thêm các lớp `bus`, `bicycle`, `three_wheeler` và `cargo` để phản ánh đúng hiện trường giao thông thực tế.

| Tên lớp | Định nghĩa chuẩn | Ví dụ tiêu biểu | Phản ví dụ và điểm dễ nhầm lẫn |
|---|---|---|---|
| `person` | Con người thật ở trạng thái sống, đang đứng, đi bộ, ngồi trên ghế ven đường hoặc đang điều khiển, ngồi sau các phương tiện giao thông. | Hai người đi bộ qua đường ở img_014, hai người ngồi trên xe máy ở img_026. | Không gán cho tượng điêu khắc đá ở img_017, ma nơ canh ở img_033, người in trên biển quảng cáo ở img_018 hay tranh vẽ ở img_026. |
| `car` | Ô tô con chở người từ 4 đến 9 chỗ ngồi, thân xe có nắp ca-pô và ca-bin tách biệt gồm các dòng sedan, hatchback, SUV, crossover và xe taxi. | SUV màu bạc trong img_012, taxi xanh và SUV đen trong img_040. | Xe bán tải chở hàng xếp vào lớp `truck`; xe thân liền khối trần cao xếp vào lớp `van`; mô hình đồ chơi trong tủ kính không gán. |
| `truck` | Phương tiện có khoang hàng hoặc sàn chở hàng tách riêng biệt phía sau ca-bin, bao gồm xe tải thông thường, xe đầu kéo chuyên dụng, xe chở xe và toàn bộ xe bán tải pickup kể cả loại có nắp thùng hàng. | Xe tải chở ô tô hai tầng ở img_002, xe bán tải ở img_012, xe bán tải có nắp thùng kín ở img_037, xe tải nhỏ ở img_036. | Xe van có khoang hàng liền khối với khoang lái xếp vào `van`; tượng đá hình xe bán tải ở img_027 không gán. |
| `van` | Xe thân liền một khối kín từ đầu đến đuôi xe, không tách rời khoang lái và khoang hàng, trần xe cao, thường có cửa trượt bên hông hoặc cửa đôi phía sau, sức chứa dưới 16 chỗ. | Xe van chở hàng màu bạc ở img_005, xe van màu trắng ở img_016. | Các dòng xe SUV hoặc MPV gầm thấp xếp vào `car`; xe chở khách cỡ lớn từ 16 chỗ trở lên xếp vào `bus`. |
| `bus` | Xe chở khách cỡ lớn có sức chứa từ 16 chỗ ngồi trở lên, bao gồm xe buýt nội đô, xe khách liên tỉnh và xe minibus. | Minibus màu trắng ở img_009, xe buýt màu vàng ở img_031, xe buýt phía hậu cảnh ở img_040. | Xe van chở khách cỡ nhỏ dưới 16 chỗ xếp vào `van`. |
| `motorcycle` | Phương tiện hai bánh có trang bị động cơ đốt trong hoặc động cơ điện, bao gồm xe máy số, xe tay ga, mô tô phân khối lớn và mô tô có gắn thùng bên sidecar. | Dòng xe máy lưu thông ở img_005, mô tô gắn thùng bên ở img_022, xe máy số ở img_026. | Xe đạp cơ học xếp vào `bicycle`; xe ba bánh tuk tuk xếp vào `three_wheeler`. |
| `bicycle` | Phương tiện thô sơ di chuyển bằng sức người đạp, bao gồm xe đạp hai bánh, xe đạp chở hàng và xe xích lô du lịch. | Xe xích lô chở khách ở img_006 và img_023, xe đạp chở hàng ở img_010. | Xe máy điện hoặc xe đạp điện có tay ga chạy bằng động cơ xếp vào `motorcycle`. |
| `three_wheeler` | Phương tiện cơ giới ba bánh có động cơ dùng để chở khách hoặc chở hàng, bao gồm xe tuk tuk, xe lam, xe lôi máy và xe auto rickshaw. | Đôi xe tuk tuk đỗ ven đường ở img_004, xe tuk tuk ở img_016 và img_035. | Xe xích lô ba bánh chạy bằng sức người đạp xếp vào `bicycle`. |
| `cargo` | Khối hàng hóa cồng kềnh, quá khổ chất trên phương tiện giao thông được tách thành một hộp riêng biệt để không làm méo mó hình thái chuẩn của xe. | Đống bao tải hàng hóa khổng lồ chất cao trên nóc xe tuk tuk ở img_035. | Hàng hóa nằm gọn hoàn toàn bên trong lòng thùng xe tải hoặc giỏ xe máy thông thường thì không tách riêng lớp này. |
| `ignore_region` | Vùng đánh dấu khu vực chứa cụm đám đông hoặc cụm xe cộ kẹt dính ở khoảng cách xa không thể phân tách chính xác ranh giới từng cá thể. | Cụm xe máy đông đúc ở hậu cảnh ngã tư img_021, đám đông người xem đứng sau hàng rào xa ở img_019. | Không dùng vùng ignore để bỏ qua các đối tượng ở cự ly gần nhìn rõ; không dùng cho vật thể nhân tạo hoặc ảnh ảo. |

### 2.2 Quy định thuộc tính đối tượng

Các thuộc tính được áp dụng cho toàn bộ các hộp đối tượng thông thường, không áp dụng cho nhãn `ignore_region`.

#### Thuộc tính mức độ che khuất `occluded_level`
Đo lường tỷ lệ phần trăm diện tích bề mặt của vật thể bị vật thể khác che chắn trước ống kính camera:
- Mức `0`: Vật thể nhìn thấy gần như hoàn toàn, tỷ lệ bị che khuất dưới 10 phần trăm diện tích.
- Mức `1`: Vật thể bị che khuất nhẹ, tỷ lệ bị che từ 10 phần trăm đến dưới 40 phần trăm diện tích.
- Mức `2`: Vật thể bị che khuất trung bình đến nặng, tỷ lệ bị che từ 40 phần trăm đến dưới 80 phần trăm diện tích.
- Mức `3`: Vật thể bị che khuất cực nặng từ 80 phần trăm diện tích trở lên, nhưng mắt người vẫn nhận diện chắc chắn được hình bóng và phân lớp đối tượng.

Lưu ý:
- Nếu vật thể bị che quá nặng đến mức không thể nhận biết được lớp thì không vẽ hộp riêng, hoặc gom vào `ignore_region` nếu nằm trong một cụm.
- Hàng rào song sắt thưa nhìn xuyên thấu chỉ tính mức độ che khuất tối đa là mức 1.
- Kính chắn gió trong suốt không tính là bề mặt che khuất; người ngồi trong xe chỉ tính phần che khuất bởi khung cửa kim loại.

#### Thuộc tính cắt mép ảnh `truncated`
Đánh giá mức độ đối tượng bị giới hạn khung hình cắt ngang:
- Giá trị `no`: Đối tượng nằm trọn vẹn trong ảnh, hoặc có chạm vào mép ảnh nhưng phần thân xe hoặc cơ thể bị khuất ngoài ảnh ước tính dưới 5 phần trăm diện tích.
- Giá trị `yes`: Hộp chạm mép ảnh và phần thân vật thể bị cắt mất nằm ngoài khung hình từ 5 phần trăm diện tích trở lên.

#### Thuộc tính nghi ngờ cần xem lại `can_review`
- Đặt giá trị `true` khi người gán nhãn đã vẽ hộp nhưng còn phân vân về việc phân lớp hoặc ranh giới điểm mù để đưa ra thảo luận tại buổi hiệu chuẩn.
- Mọi hộp đánh dấu `can_review` phải được thống nhất giải quyết dứt điểm ở pha hiệu chuẩn; bộ dữ liệu bàn giao cuối cùng không còn hộp nào tồn tại cờ này.

---

## 3. Hệ thống quy tắc vẽ hộp bounding box

| Mã hiệu | Quy tắc chi tiết |
|---|---|
| **R-01** | **Chỉ gán nhãn đối tượng vật lý có thật:** Tuyệt đối không gán hộp cho tượng điêu khắc kim loại hoặc đá, mô hình nhân tạo, búp bê ma nơ canh, ảnh chân dung trên biển bạt quảng cáo, tranh vẽ trang trí tường và hình ảnh phản chiếu qua gương phẳng hay mặt đường ướt. Ảnh không chứa vật thể thật vẫn phải được lưu giữ trong tệp nhãn COCO với danh sách không có hộp nào. |
| **R-02** | **Bao trọn vùng nhìn thấy thực tế:** Hộp vẽ phải tiếp xúc sát ranh giới ngoài cùng nhìn thấy của đối tượng với sai số không quá hai pixel. Không ước lượng vẽ bù vào phần bị che khuất sau chướng ngại vật; không gom phần bóng đổ trên mặt đường vào hộp. Đồ vật gắn trên người như ba lô, mũ bảo hiểm tính vào hộp `person`. |
| **R-03** | **Tách biệt người và phương tiện hai hoặc ba bánh:** Mỗi người trên xe được vẽ một hộp `person` riêng biệt bao gồm cả người lái và người ngồi sau. Thân xe được vẽ một hộp riêng theo đúng lớp của xe (`motorcycle`, `bicycle` hoặc `three_wheeler`). Hộp thân xe chỉ bao trùm phần cơ khí nhìn thấy được, không vẽ bù phần bị thân người che chắn. |
| **R-04** | **Người ngồi trong ca-bin có khoang:** Chỉ vẽ hộp `person` cho tài xế hoặc hành khách ngồi trong ô tô, xe tải hoặc xe tuk tuk khi nhìn thấy rõ ràng ít nhất từ phần đầu và vai trở lên và chiều cao hộp đạt tối thiểu 20 pixel. |
| **R-05** | **Ngưỡng kích thước tối thiểu:** Đối tượng có chiều cao vùng nhìn thấy dưới 20 pixel không vẽ hộp riêng lẻ. Nếu các đối tượng nhỏ này đứng đơn lẻ thì bỏ qua; nếu có từ 3 đối tượng nhỏ trở lên tập trung sát nhau thì khoanh một hộp `ignore_region` bao quanh cụm. |
| **R-06** | **Xử lý cụm đám đông và bãi xe dày đặc:** Khi có từ 5 đối tượng trở lên cùng nhóm xếp chồng lấn phức tạp đến mức không thể xác định ranh giới tách biệt từng cá thể thì vẽ một hộp `ignore_region` bao trùm vùng phía sau. Các đối tượng ở tiền cảnh nhìn rõ biên vẫn vẽ hộp riêng; hộp `ignore_region` không được vẽ đè lên các hộp đã tách riêng. |
| **R-07** | **Gán thuộc tính che khuất chuẩn xác:** Đánh giá `occluded_level` theo đúng 4 mức quy định tại mục 2.2. Mức độ che khuất không làm thay đổi nguyên tắc vẽ hộp ôm sát phần nhìn thấy theo R-02. |
| **R-08** | **Xử lý đối tượng chạm mép ảnh:** Hộp vẽ dừng lại ngay sát viền cạnh ảnh và đánh dấu `truncated=yes` nếu phần cơ thể hoặc thân xe bị mất ra ngoài mép ảnh đạt từ 5 phần trăm diện tích trở lên. Không tính phần diện tích ngoài mép ảnh vào thuộc tính `occluded_level`. |
| **R-09** | **Xe chuyên dụng chở ô tô nhiều tầng:** Vẽ một hộp `truck` bao quát toàn bộ xe tải kéo và khung giàn chở hàng hai tầng. Đồng thời vẽ từng hộp `car` độc lập cho mỗi chiếc ô tô con đang đặt trên sàn vì các xe con này đều có đầy đủ đặc trưng thị giác của xe thật. |
| **R-10** | **Quy tắc phân định xe bốn bánh:** Mọi phương tiện có khoang chở hàng hoặc thùng hàng tách rời khỏi ca-bin lái đều xếp vào lớp `truck`, bao gồm toàn bộ các dòng xe bán tải pickup kể cả khi có lắp thêm nắp thùng cao kín phía sau. Xe thân kín liền khối trần cao xếp vào `van`; xe chở khách từ 16 chỗ xếp vào `bus`; các dòng xe con còn lại xếp vào `car`. |
| **R-11** | **Quy tắc phân định phương tiện ba bánh:** Phương tiện ba bánh gắn động cơ dùng chở khách hoặc chở hàng như xe tuk tuk, xe lam xếp vào `three_wheeler`. Phương tiện ba bánh thô sơ vận hành bằng sức người đạp như xe xích lô xếp vào `bicycle`. |
| **R-12** | **Xử lý góc chụp xe chủ và gương chiếu hậu:** Không gán nhãn cho gương chiếu hậu, nắp ca-pô hoặc thân xe của chính xe gắn camera chụp ảnh. Toàn bộ hình ảnh xe cộ hoặc người nhìn qua mặt gương chiếu hậu là ảnh phản chiếu ảo, áp dụng R-01 không gán nhãn. |
| **R-13** | **Đối tượng bị nhòe mờ do chuyển động:** Vẽ hộp ôm phần thân chính nhìn rõ của vật thể nếu vẫn phân biệt được lớp và ước tính sai số biên dưới 10 phần trăm; không ôm theo vệt bóng nhòe kéo dài. Trường hợp vệt nhòe quá nặng không thể nhận dạng ranh giới thì khoanh vùng `ignore_region`. |
| **R-14** | **Phương tiện đang đỗ hoặc trưng bày:** Toàn bộ xe cơ giới thật đang đỗ bên lề đường, trong bãi trông giữ hoặc trưng bày trong phòng trưng bày, bảo tàng xe vẫn được gán nhãn bình thường như xe đang lưu thông. |
| **R-15** | **Tách biệt khối hàng cồng kềnh quá khổ:** Đối với phương tiện chở khối hàng khổng lồ chất cao gấp nhiều lần thân xe thì vẽ một hộp xe ôm trọn thân xe và vẽ thêm một hộp `cargo` riêng ôm trọn khối hàng cồng kềnh đó để giữ nguyên chuẩn thị giác cho thân xe. |
| **R-16** | **Xử lý các tình huống chưa chắc chắn:** Nếu đã vẽ hộp nhưng còn phân vân thì giữ nguyên hộp và tích chọn `can_review=true`. Nếu chưa đủ căn cứ vẽ hộp thì ghi lại tên ảnh, tọa độ vùng và lý do để đưa ra phiên họp hiệu chuẩn nhóm. Tuyệt đối không tự ý đặt quy tắc ngoài thỏa thuận khi đang gán nhãn. |

---

## 4. Bảng hướng dẫn xử lý các ca khó trong bộ ảnh

| Mã ảnh | Tình huống phát hiện | Quy tắc áp dụng | Quyết định gán nhãn của nhóm | Căn cứ và lý giải kỹ thuật |
|---|---|---|---|---|
| img_002.jpg ★ | Xe đầu kéo chuyên dụng chở 8 ô tô con trên giàn khung hai tầng | R-09, R-07, R-05 | Vẽ 1 hộp `truck` bao toàn bộ đầu kéo và giàn khung chở hàng; vẽ riêng 8 hộp `car` cho 8 xe con trên sàn với mức che khuất tương ứng | Xe con trên sàn mang đầy đủ đặc trưng thị giác ô tô thật, cần nhận dạng để tránh nguy cơ va chạm quẹt thành xe |
| img_004.jpg ★ | Hai chiếc xe tuk tuk ba bánh màu xanh đỗ bên lề đường ban đêm | R-11, R-04 | Vẽ 2 hộp `three_wheeler` cho thân xe; tài xế nhìn rõ đầu và vai vẽ thêm hộp `person` với mức che khuất 2 | Phân lớp chuẩn xác xe ba bánh có động cơ, không gộp nhầm vào ô tô hay xe máy |
| img_005.jpg ★ | Dòng người đi xe máy chen chúc dày đặc tại giao lộ, che khuất nhau nặng | R-03, R-07, R-08, R-10 | Vẽ từng hộp `person` độc lập cho người nhìn thấy; xe máy nhìn rõ thân vẽ `motorcycle`; xe van bạc phía sau vẽ `van`; ô tô xám chạm mép vẽ `car` kèm `truncated=yes` | Ưu tiên cao nhất là bảo vệ người sống; không đoán vẽ thân xe máy khi hoàn toàn bị che khuất |
| img_006.jpg ★ | Đoàn xe xích lô du lịch và người đi xe máy nhìn từ phía sau | R-03, R-11, R-05, R-06 | Vẽ hộp `bicycle` cho thân xe xích lô; người đạp và khách ngồi vẽ hộp `person`; dòng xe máy vẽ `motorcycle` và `person`; cụm nhỏ xa vẽ `ignore_region` | Thống nhất nhóm đưa xích lô vào lớp `bicycle` và tách người đạp thành `person` |
| img_008.jpg ★ | Bãi xe máy dày đặc ban đêm với hai người bảo vệ mặc áo phản quang cam | R-06, R-03, R-14 | Xe máy tiền cảnh nhìn rõ biên vẽ hộp `motorcycle`; hai người bảo vệ vẽ `person`; phần bãi xe chồng chéo phía sau khoanh `ignore_region` | Xe thật đỗ bãi vẫn được gán nhãn; các xe dính liền không tách được chuyển sang vùng ignore |
| img_011.jpg | Tượng quái vật khổng lồ đứng cạnh dàn xe mô tô độ khung sườn thép | R-01, R-16 | Tượng quái vật không gán nhãn; dàn xe mô tô độ vẽ hộp `motorcycle` kèm cờ `can_review=true` | Tượng là vật thể điêu khắc nghệ thuật; xe máy dù khung sườn dị thường nhưng vẫn là phương tiện cơ học |
| img_012.jpg ★ | Xe bán tải chở hàng cắt mép phải và chiếc SUV màu bạc | R-10, R-08, R-02 | Xe bán tải vẽ hộp `truck` kèm `truncated=yes`; SUV vẽ hộp `car`; xe tải nhỏ ở xa vẽ `truck` nếu đạt chiều cao từ 20 pixel | Xe bán tải có khoang thùng hàng tách biệt ca-bin nên thuộc lớp `truck` |
| img_017.jpg ★ | Tượng đồng vị tướng cưỡi ngựa đặt giữa quảng trường lớn | R-01 | Không gán bất kỳ hộp nhãn nào (0 box). Ảnh vẫn lưu trong tệp COCO | Tượng điêu khắc bất động, không tham gia lưu thông, tuyệt đối không gán `person` |
| img_018.jpg | Tấm pano biển quảng cáo chân dung lớn và ô tô sau hàng rào sắt | R-01, R-07, R-14 | Người trên biển pano không gán; ô tô sau hàng rào thưa vẽ `car` với mức che khuất 1 | Hình ảnh in phẳng trên biển bạt không phải người thật; hàng rào thưa chỉ che khuất nhẹ |
| img_020.jpg ★ | Gương chiếu hậu ô tô phản chiếu dòng xe cộ và cầu vồng phía sau | R-01, R-12 | Không gán bất kỳ hộp nhãn nào (0 box). Giữ ảnh trong tệp COCO | Toàn bộ hình ảnh trong gương là ảnh ảo quang học và xe của chính người chụp |
| img_026.jpg ★ | Hai thanh niên ngồi trên một xe máy trong hầm, phía sau có tranh tường | R-03, R-01, R-07 | Vẽ 2 hộp `person` cho hai người; vẽ 1 hộp `motorcycle` cho thân xe máy nhìn thấy; tranh vẽ phong cảnh không gán | Đảm bảo phát hiện cả người điều khiển lẫn hành khách; không gán tranh vẽ |
| img_027.jpg ★ | Khối đá điêu khắc tạc hình xe bán tải đặt trong khu nghĩa trang | R-01 | Không gán bất kỳ hộp nhãn nào (0 box) | Bia đá mỹ nghệ tĩnh, không có động cơ và không có khả năng di chuyển |
| img_033.jpg ★ | Ma nơ canh mặc trang phục đứng trong tủ kính thời trang | R-01 | Không gán hộp nhãn cho ma nơ canh | Ma nơ canh là hình nộm bằng nhựa trưng bày, không phải người sống |
| img_035.jpg | Xe tuk tuk chở khối bao tải hàng hóa cồng kềnh khổng lồ | R-11, R-15, R-04 | Thân xe vẽ hộp `three_wheeler`; khối bao tải hàng hóa chất cao vẽ hộp `cargo` riêng; tài xế nhìn rõ vẽ `person` | Tách riêng khối hàng cồng kềnh để giữ nguyên tỷ lệ khung hình học chuẩn của xe ba bánh |
| img_037.jpg | Xe bán tải có nắp thùng cao kín phía sau, xe buýt và người đi bộ | R-10, R-03 | Xe bán tải có nắp thùng vẽ hộp `truck`; xe buýt vẽ hộp `bus`; người đi bộ vẽ `person` | Nắp thùng gắn thêm không làm thay đổi bản chất kết cấu xe bán tải chở hàng |
| img_038.jpg | Tủ kính bày bán các mô hình xe đồ chơi thu nhỏ nhiều màu | R-01 | Không gán bất kỳ hộp nhãn nào (0 box) | Toàn bộ là đồ chơi kích thước thu nhỏ, không tham gia giao thông thực tế |
| img_040.jpg ★ | Người đội mũ bảo hiểm cận cảnh, xe taxi xanh, xe tuk tuk và xe buýt | R-03, R-08, R-10, R-11 | Người cận cảnh vẽ `person` kèm `truncated=yes`; taxi vẽ `car`; tuk tuk vẽ `three_wheeler`; xe buýt xa vẽ `bus` | Gương xe máy ở góc không đủ căn cứ nhận diện thân xe nên không vẽ hộp xe máy |

---

## 5. Kiểm soát chất lượng và Lịch sử phiên bản

### 5.1 Các chỉ số kiểm soát chất lượng kỹ thuật
Toàn bộ quá trình đánh giá độ đồng thuận giữa hai thành viên và nghiệm thu thành phẩm tuân thủ các ngưỡng đo lường chuẩn mực của bài lab trong tệp `taxonomy.json`:
- Ngưỡng ghép cặp hộp bounding box khi so sánh độ đồng thuận (IoU match threshold): Tối thiểu 0.50.
- Độ đồng thuận trung bình về diện tích giao nhau (Average IoU): Đạt tối thiểu 0.85 trên toàn bộ các cặp hộp ghép được.
- Tỷ lệ đồng thuận phân loại nhãn lớp (Class agreement): Đạt tối thiểu 95 phần trăm trên các cặp hộp ghép được.
- Chiều cao hộp nhỏ nhất được phép gán đơn lẻ: 20 pixel.
- Tỷ lệ sai số ảnh tối đa cho phép để nghiệm thu bàn giao: Dưới 2 phần trăm trên toàn bộ tập ảnh.

### 5.2 Lịch sử tiến hóa tài liệu
- **Bản v0:** Khung khởi tạo ban đầu, định nghĩa còn sơ khai và thiếu các quy tắc thực tế cho người đi xe máy, xe ba bánh, xe chở xe và các loại ảnh phản chiếu.
- **Bản v0.1 cá nhân:** Thành viên 101 và 102 xây dựng độc lập, xác lập chi tiết các ca khó, nhận diện nhu cầu tách lớp người và xe máy cũng như loại trừ các đối tượng giả.
- **Bản v1.0 đồng thuận nhóm (bản hiện tại):**
  - Đồng thuận thống nhất tên lớp xe ba bánh là `three_wheeler`.
  - Bổ sung lớp `bicycle` và quy định vẽ hộp cho thân xe xích lô cùng người đạp.
  - Bổ sung lớp `cargo` để tách biệt khối hàng hóa quá khổ trên xe.
  - Khóa thang đo mức độ che khuất 4 mức chuẩn theo tỷ lệ đo lường phần trăm diện tích cụ thể.
  - Thống nhất toàn bộ xe bán tải pickup kể cả có nắp thùng kín thuộc lớp `truck`.
  - Chuẩn hóa hệ thống 16 quy tắc cốt lõi từ R-01 đến R-16 làm kim chỉ nam cho quá trình gán nhãn độc lập ở Pha 3.
