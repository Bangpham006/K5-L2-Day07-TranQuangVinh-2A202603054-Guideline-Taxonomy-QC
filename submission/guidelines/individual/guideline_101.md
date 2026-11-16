# Guideline gán nhãn — v0.1-101 (bản cá nhân) — 101

- Phiên bản: v0.1-101 
- Ngày: 16-11-2026
- Người viết / duyệt: 101
- Phạm vi khảo sát: cả 40 ảnh trong `data/images/`, ưu tiên 12 ảnh pilot (★)
- Loại nhãn: bounding box chữ nhật 2D, công cụ CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

**Mô hình dùng nhãn:** mô hình phát hiện vật 2D (object detection) cho **camera giao thông / hỗ trợ lái xe**,
nhận ra **người và phương tiện thật đang có mặt trên đường hoặc lề đường**.

**Dùng để làm gì:** đếm và định vị người đi đường, người điều khiển phương tiện và các loại phương tiện để
cảnh báo va chạm, đo mật độ giao thông.

**Sai ở đâu thì nguy hiểm:**

| Loại sai | Hậu quả | Mức độ |
|---|---|---|
| Bỏ sót **người thật** (nhất là người bị che, người ngồi sau xe máy) | Mô hình học cách bỏ qua người → không cảnh báo | **Nghiêm trọng nhất** |
| Gán **vật giả** (tượng, ma-nơ-canh, ảnh in, phản chiếu) thành người/xe | Mô hình báo động giả, đếm sai mật độ | Cao |
| Sai lớp phương tiện (pickup ↔ car, van ↔ car) | Sai thống kê loại xe, ít ảnh hưởng an toàn | Trung bình |
| Box lệch / lỏng | Giảm độ chính xác định vị | Trung bình |

Nguyên tắc ưu tiên khi phân vân: **vật thật có thể tham gia giao thông thì gán; vật không thật thì không gán**.

---

## 2. Lớp và thuộc tính

### 2.1 Lớp

Mỗi vật thuộc **đúng một** lớp. Đề xuất thêm 3 lớp (`bus`, `bicycle`, `three_wheeler`) vì chúng xuất hiện ở
nhiều ảnh (009, 031, 032, 037, 040; 003, 006, 010, 023, 029, 039; 004, 016, 035, 040) mà v0 không có chỗ đặt.

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người thật nhìn thấy được: đi bộ, đứng, ngồi, điều khiển hoặc ngồi trên phương tiện (R-03, R-04). | img_014 (2 người đi bộ), img_026 ★ (2 người trên xe máy), img_034 | Tượng (img_017 ★), ma-nơ-canh (img_033 ★), người trên billboard (img_018), poster (img_030), tranh tường (img_026 ★) |
| `car` | Ô tô chở người cỡ nhỏ, thân có nắp capo và cabin: sedan, hatchback, SUV, crossover, taxi. | img_023 (ô tô đen), img_012 ★ (SUV bạc), img_040 ★ (taxi xanh, SUV đen) | Pickup → `truck` (R-10); xe thân liền một khối mái cao → `van`; xe mô hình (img_038) |
| `van` | Xe thân liền **một khối** (không tách cabin và khoang sau), mái cao, cửa trượt hoặc cửa sau lớn, dưới 16 chỗ. | img_005 ★ (van bạc phía sau), img_016 (van trắng) | SUV/MPV mái thấp → `car`; minibus từ 16 chỗ → `bus` |
| `truck` | Xe có **khoang hàng tách riêng sau cabin** (thùng hở, thùng kín, thùng bạt, nắp thùng), kể cả pickup, xe tải nhỏ, đầu kéo và xe chở xe. | img_002 ★ (xe chở ô tô), img_007, img_012 ★ (pickup), img_036 (xe tải nhỏ), img_037 (pickup có nắp thùng) | Van (khoang liền cabin) → `van`; tượng hình pickup (img_027 ★) → không gán |
| `bus` | Xe chở khách từ 16 chỗ trở lên: xe buýt, xe khách, minibus. | img_009 (minibus), img_031, img_037, img_040 ★ (xe buýt phía sau) | Van chở khách nhỏ → `van` |
| `motorcycle` | Xe máy, xe tay ga, mô tô hai bánh có động cơ; xe mô tô kèm thùng bên (sidecar) tính cả thùng. | img_006 ★, img_008 ★, img_022 (sidecar), img_026 ★ | Xe đạp → `bicycle`; tuk-tuk → `three_wheeler`; mô hình xe trang trí không chạy được → không gán |
| `bicycle` | Phương tiện chạy bằng sức người: xe đạp, xe đạp chở hàng, xích lô. | img_006 ★, img_023 (xích lô), img_029, img_010 (xe đạp chở hàng) | Xe máy điện có bàn đạp nhưng có động cơ → `motorcycle` |
| `three_wheeler` | Xe ba bánh có động cơ chở khách/hàng: tuk-tuk, xe lam, auto rickshaw. | img_004 ★, img_016, img_035, img_040 ★ | Xích lô (đạp chân) → `bicycle`; xe máy kéo rơ-moóc nhỏ (img_025) → `motorcycle` |
| `ignore_region` | Vùng chứa đám đông hoặc cụm vật nhỏ không tách được từng vật; vật thật bị bỏ sót bên trong không tính lỗi (R-05, R-06). | img_021 (dòng xe máy phía xa), img_019 (đám đông sau rào), img_030 (bãi xe phía sau) | Không dùng ignore để "né" vật lớn rõ ràng; không dùng cho vật giả (vật giả không gán gì cả) |

**Vật không thuộc lớp nào** (ngựa thật/tượng ngựa, chim, cọc giao thông, xe đẩy tay, biển quảng cáo…):
**không gán**, không vẽ ignore. Vật **có thể là người/phương tiện thật nhưng không chắc** → vẽ box theo lớp gần
nhất và tick `can_review`; nếu không đủ chắc để vẽ box → ghi vấn đề cấp ảnh (ảnh, vùng, lý do).

### 2.2 Thuộc tính

Thuộc tính chỉ gán cho box đối tượng, **không** gán cho `ignore_region`.

**`occluded_level`** — mức **bị vật khác che** (không tính phần bị mép ảnh cắt; phần đó thuộc `truncated`).
Ước lượng tỷ lệ diện tích vật bị che so với toàn bộ vật nếu nhìn thấy đầy đủ:

| Mức | Ngưỡng | Ví dụ |
|---|---|---|
| `0` | Bị che **dưới 10%** | img_023 (ô tô đen), img_014 (2 người đi bộ) |
| `1` | Bị che **10% đến dưới 40%** | Người lái xe máy bị xe phía trước che chân |
| `2` | Bị che **40% đến dưới 80%** | Ô tô tầng dưới trên xe chở xe bị khung che (img_002 ★); xe máy bị người ngồi che gần hết thân (img_026 ★) |
| `3` | Bị che **từ 80% trở lên** nhưng **vẫn nhận ra được lớp** | Người phía sau trong cụm img_005 ★ chỉ thấy mũ và một phần mặt |

- Không nhận ra được lớp → không gán (hoặc gom vào `ignore_region` nếu là cụm, R-06).
- Hàng rào song thưa vẫn nhìn rõ vật phía sau (img_018) → tối đa mức `1`.
- Kính xe trong suốt: người thấy qua kính tính phần khung cửa che, không tính lớp kính.

**`truncated`** — vật bị **mép ảnh** cắt:

| Giá trị | Khi nào |
|---|---|
| `yes` | Box chạm mép ảnh **và** ước lượng phần vật nằm ngoài ảnh **từ 5% trở lên** (img_015 ô tô đỏ, img_040 ★ người đội mũ và taxi, img_001 xe tải bên phải) |
| `no` | Vật nằm trọn trong ảnh, hoặc chỉ chạm mép mà phần mất dưới 5% |

**`can_review`** (checkbox) — tick khi **đã vẽ box nhưng chưa chắc** về: (a) vật thật hay giả, (b) lớp giữa hai
lựa chọn, (c) biên box do mờ/tối. Ghi kèm lý do trong nhật ký cá nhân. Mọi box `can_review` phải được nhóm
chốt ở calibration; bản nhãn cuối không còn box nào `can_review`.

---

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** | **Chỉ gán vật thật, hiện diện vật lý trong cảnh.** Không gán: tượng, mô hình, đồ chơi, ma-nơ-canh, người/xe trong ảnh in, poster, billboard, tranh tường, hình vẽ; không gán hình **phản chiếu** trong gương, kính, mặt đường ướt. Ảnh không có vật thật vẫn giữ trong bộ nhãn với 0 box. |
| **R-02** | **Box ôm sát phần nhìn thấy** (modal box): cạnh box chạm điểm ngoài cùng nhìn thấy của vật, sai lệch không quá ~2 px; **không đoán** phần bị che; không gồm bóng đổ. Vật đeo/mặc trên người (mũ, ba lô, túi đeo) thuộc box `person`. Hàng hoá **gắn chặt** trên xe thuộc box xe (img_035 hàng chất trên auto rickshaw). Vật **do người cầm/vác** vươn xa (ống dài img_029, quang gánh img_003) **không** tính vào box nào. |
| **R-03** | **Người trên xe hai/ba bánh:** mỗi người một box `person` (cả người lái và người ngồi sau), xe một box riêng theo lớp xe. Box xe chỉ gồm phần xe nhìn thấy (bánh, thân, tay lái, gương); phần xe bị người che không đoán. Ví dụ img_026 ★ = 2 `person` + 1 `motorcycle`. Áp dụng tương tự cho `bicycle` (img_029, img_023 xích lô) và `three_wheeler`. |
| **R-04** | **Người trong xe có khoang** (tài xế ô tô, tuk-tuk, xe diễu hành): gán `person` nếu nhìn thấy rõ ít nhất **đầu và vai** và box cao từ 20 px; `occluded_level` theo phần thân xe che. Nếu chỉ thấy bóng mờ qua kính phản quang → không gán. |
| **R-05** | **Vật nhỏ:** vật có chiều cao phần nhìn thấy **dưới 20 px** (ngưỡng `min_box_height_px` của lab) → không vẽ box riêng. Vật nhỏ đơn lẻ: bỏ qua. Từ **3 vật nhỏ trở lên đứng sát nhau** → vẽ một `ignore_region` bao cụm. |
| **R-06** | **Đám đông / bãi xe dày đặc:** khi từ **5 vật cùng lớp** trở lên chồng lấp đến mức không xác định được biên từng vật → vẽ một `ignore_region` bao phần không tách được. Vật ở **tiền cảnh** xác định được biên vẫn vẽ box riêng; `ignore_region` **không phủ** lên box đã vẽ riêng (vẽ ignore cho phần phía sau). |
| **R-07** | **Che khuất:** gán `occluded_level` theo bảng ở mục 2.2; che khuất không làm thay đổi cách vẽ box (vẫn theo R-02). |
| **R-08** | **Cắt mép ảnh:** box dừng ở mép ảnh, `truncated=yes` theo ngưỡng ở mục 2.2; không tính phần ngoài ảnh vào `occluded_level`. |
| **R-09** | **Xe chở xe:** xe chở (đầu kéo + rơ-moóc nối liền) là **một** box `truck` bao cả đầu kéo và khung chở. **Mỗi ô tô được chở** nhìn thấy là một box `car` riêng, `occluded_level` theo phần khung che (img_002 ★). Lý do: đó vẫn là ô tô thật; nhầm chúng thành vật nền làm mô hình bỏ sót ô tô bị che. |
| **R-10** | **Phân lớp xe bốn bánh:** khoang hàng tách khỏi cabin → `truck` (gồm **pickup**, kể cả pickup có nắp thùng); thân liền một khối mái cao → `van`; từ 16 chỗ → `bus`; còn lại (sedan, SUV, crossover, taxi) → `car`. |
| **R-11** | **Xe ba bánh:** tuk-tuk, xe lam, auto rickshaw → `three_wheeler`; xích lô (đạp chân) → `bicycle`. |
| **R-12** | **Xe của người chụp (ego) và gương:** không gán thân xe, gương, nắp capo của xe người chụp; mọi vật trong gương chiếu hậu là phản chiếu → không gán (R-01). Ví dụ img_020 ★: 0 box. |
| **R-13** | **Mờ chuyển động:** gán nếu nhận ra lớp và ước lượng được biên với sai số dưới khoảng 10% chiều cao vật; box ôm phần thân chính, không ôm vệt nhòe kéo dài. Không tách được từng vật → `ignore_region` (img_014 dòng xe phía sau). |
| **R-14** | **Xe đỗ / trưng bày:** xe thật đang đỗ (lề đường, bãi xe, garage, bảo tàng) **vẫn gán** như xe đang chạy (img_008 ★, img_018, img_024). |
| **R-15** | **Ảnh đen trắng / ảnh cũ:** áp dụng quy tắc như ảnh màu (img_022, img_029). |
| **R-16** | **Không chắc:** đã vẽ box → tick `can_review`; chưa vẽ box → ghi vấn đề cấp ảnh (ảnh, vùng, lý do). Không tự đặt quy tắc mới khi đang gán. |

---

## 4. Ca khó

Cột **Loại**: `Quy tắc` = quyết định suy ra trực tiếp từ một rule; `Hỏi reviewer` = guideline chưa đủ để chốt,
cần nhóm/reviewer quyết ở Pha 2 hoặc calibration.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do | Loại |
|---|---|---|---|---|---|
| img_017 ★ | Tượng người cưỡi ngựa đội nón giao thông | R-01 | **Không gán** (0 box). Ảnh vẫn có trong COCO | Không phải người thật; gán `person` sẽ dạy mô hình báo động giả. Đây là ca hai người có thể **cùng sai** mà IAA không phát hiện | Quy tắc |
| img_027 ★ | Tượng đá hình xe pickup trong nghĩa trang | R-01 | **Không gán** (0 box) | Không phải phương tiện thật, không tham gia giao thông | Quy tắc |
| img_033 ★ | Ma-nơ-canh, ảnh in người đen trắng trên kính; bên phải có thân và gương của một chiếc xe | R-01, R-16 | Ma-nơ-canh và ảnh in: **không gán**. Chiếc xe bên phải: nếu là phản chiếu trên kính → không gán; nếu là xe thật đứng trước tủ kính → gán theo R-10 | Ảnh đơn sắc, khó phân biệt xe thật và phản chiếu | Hỏi reviewer (phần xe) |
| img_020 ★ | Gương chiếu hậu: trong gương có đường, một ô tô ở xa và thân xe của chính mình | R-01, R-12 | **Không gán** (0 box) | Toàn bộ là phản chiếu và xe ego | Quy tắc |
| img_026 ★ | Hai người trên một xe máy trong hầm; phía sau là tranh tường in hình bến cảng | R-03, R-01, R-07 | 2 `person` + 1 `motorcycle`; xe máy `occluded_level` 1–2 tuỳ phần chân người che; tranh tường: không gán | Người ngồi sau vẫn là người thật cần phát hiện; tranh tường là ảnh in | Quy tắc |
| img_002 ★ | Xe đầu kéo chở 7–8 ô tô trên hai tầng; một người nhỏ phía sau | R-09, R-07, R-05 | 1 `truck` bao đầu kéo + khung; mỗi ô tô nhìn thấy 1 `car` (tầng dưới thường `occluded_level` 2); người phía sau gán nếu cao từ 20 px | Ô tô được chở vẫn là ô tô thật; định nghĩa thống nhất dễ đếm | Quy tắc (cần nhóm xác nhận R-09) |
| img_005 ★ | 4–5 người trên xe máy chen chúc, che nhau mạnh, cắt mép trái/dưới; van bạc và ô tô phía sau | R-03, R-07, R-08, R-10 | Mỗi người nhận ra được 1 `person` (người phía sau `occluded_level` 2–3); xe máy chỉ thấy tay lái/gương mà không nhận ra được xe → không gán; van bạc → `van`; ô tô xám cắt mép → `car`, `truncated=yes` | Ưu tiên không bỏ sót người; không đoán xe khi không nhìn thấy | Quy tắc |
| img_006 ★ | Dòng xe máy và xe đạp nhìn từ phía sau; taxi cắt mép trái; nhiều xe nhỏ ở xa | R-03, R-11, R-05, R-06 | Người + xe tiền cảnh vẽ riêng (xe chở 2 người = 2 `person`); xe đạp → `bicycle`; taxi → `car`, `truncated=yes`; cụm xe nhỏ phía xa → `ignore_region` | Tiền cảnh đủ rõ để tách; phía xa dưới 20 px | Quy tắc |
| img_008 ★ | Bãi xe máy đỗ dày đặc ban đêm; 2 người áo cam (1 đứng, 1 ngồi trên xe) | R-06, R-03, R-14 | Xe máy tiền cảnh tách được biên → box riêng; phần bãi xe chồng lấp phía sau → `ignore_region`; 2 người áo cam → `person` | Xe đỗ vẫn là xe thật (R-14); không thể tách từng xe ở phía sau | Quy tắc |
| img_004 ★ | Hai tuk-tuk đỗ ban đêm, có tài xế ngồi trong | R-11, R-04 | 2 `three_wheeler`; tài xế thấy rõ đầu vai → `person`, `occluded_level` 2 | Tuk-tuk là phương tiện thật, không thuộc car/motorcycle | Quy tắc (nếu nhóm không thêm `three_wheeler` → hỏi reviewer) |
| img_012 ★ | Pickup chở hàng cắt mép phải/dưới; SUV bạc; xe tải phía xa bị che | R-10, R-08, R-02 | Pickup → `truck`, `truncated=yes`, hàng trong thùng thuộc box; SUV → `car`; xe tải xa → `truck` nếu cao từ 20 px | Pickup có khoang hàng tách cabin | Quy tắc |
| img_040 ★ | Người đội mũ bảo hiểm cận cảnh (thấy gương xe máy), taxi xanh cắt mép, tuk-tuk, SUV, xe buýt phía sau | R-03, R-08, R-10, R-11 | Người → `person`, `truncated=yes`; xe máy chỉ thấy gương → không gán (không nhận ra xe); taxi → `car`, `truncated=yes`; tuk-tuk → `three_wheeler`; SUV → `car`; xe buýt → `bus` | Gương xe máy không đủ để vẽ box xe | Quy tắc |
| img_011 | Mô hình nhân vật rất lớn cận cảnh; xe máy trang trí kim loại; đám đông nhỏ ở xa | R-01, R-06, R-16 | Nhân vật: kích thước lớn hơn người thật → coi là mô hình, **không gán**; xe máy trang trí: nếu là xe chạy được → `motorcycle` + `can_review`; đám đông xa → `ignore_region` | Chưa chắc nhân vật là tượng hay người hoá trang, xe là xe thật hay tác phẩm trưng bày | Hỏi reviewer |
| img_019 | Xe diễu hành hình xe thể thao có biển số, người lái thật và nhân vật mascot; đám đông sau rào | R-04, R-06, R-16 | Xe → `car` (có biển số, chạy trên đường); người lái → `person`; mascot → `person` + `can_review` (có thể là người mặc trang phục); đám đông → `ignore_region` | Xe có biển số là phương tiện thật; mascot chưa chắc có người bên trong | Hỏi reviewer (mascot) |
| img_024 | Bảo tàng xe: xe đen phía trước, thân xe vàng đặt trên nóc, nhiều xe trưng bày khác | R-14, R-09, R-16 | Xe trưng bày thật → `car`; thân xe vàng trên nóc → `car` + `can_review` | Chưa rõ thân xe vàng là xe thật hay mô hình | Hỏi reviewer |
| img_028 | Pickup cổ chở tượng ngựa trắng; SUV đỏ phía sau | R-10, R-01, R-07 | Pickup → `truck`; tượng ngựa: không gán (không thuộc lớp nào); SUV đỏ → `car`, `occluded_level` 1 | Tượng ngựa không phải lớp mục tiêu | Quy tắc |
| img_013 | Phố đêm; tượng ngựa vàng cạnh người đi bộ; xe nhỏ ở xa | R-01, R-05 | Người đi bộ cao từ 20 px → `person`; tượng ngựa không gán; xe xa dưới 20 px bỏ qua | Tượng không phải lớp mục tiêu | Quy tắc |
| img_018 | Billboard chân dung người; ô tô đỗ sau hàng rào sắt | R-01, R-07, R-14 | Billboard: không gán; ô tô sau rào → `car`, `occluded_level` 1 (rào thưa) | Người trên billboard là ảnh in | Quy tắc |
| img_030 | Bãi xe máy dày đặc cắt mép; người đồng phục; poster người | R-06, R-01, R-08 | Xe tiền cảnh tách được → `motorcycle`, `truncated` theo mép; phía sau → `ignore_region`; người đồng phục → `person`; poster: không gán | Quá nhiều xe chồng lấp phía sau | Quy tắc |
| img_021 | Ngã tư rất nhiều xe máy và người | R-06, R-03 | Vài hàng tiền cảnh: vẽ người + xe riêng; phần dày đặc phía sau → `ignore_region` | Không thể tách biên từng xe ở xa | Quy tắc |
| img_014 | Hai người đi bộ rõ; xe máy và người phía sau nhòe chuyển động | R-13 | 2 `person` rõ nét; vùng nhòe không xác định được biên → `ignore_region` | Không thể vẽ box chính xác cho vật nhòe | Quy tắc |
| img_015 | Ô tô đỏ cắt mép trái, xe bạc cắt mép phải; xe xa nhòe; phản chiếu trên đường ướt | R-08, R-13, R-01 | 2 `car` `truncated=yes`; xe xa nhận ra được → `car`, nhòe quá → ignore; phản chiếu: không gán | Bóng xe trên mặt đường là phản chiếu | Quy tắc |
| img_031 | Phố đêm mưa: ô tô, xe buýt, xe máy, người nhỏ; phản chiếu ánh đèn; hình vẽ nhân vật trên tường | R-10, R-05, R-01 | Gán xe/người từ 20 px; xe buýt → `bus`; hình vẽ và phản chiếu: không gán | Hình vẽ không phải người | Quy tắc |
| img_009 | Minibus trắng; xe máy nhỏ | R-10, R-05 | Minibus → `bus`; xe máy từ 20 px → `motorcycle` | Xe khách từ 16 chỗ thuộc `bus` | Quy tắc (cần đếm chỗ ước lượng) |
| img_016 | Hai tuk-tuk, người đi xe tay ga, van trắng, phụ nữ cận cảnh cắt mép | R-11, R-03, R-10, R-08 | `three_wheeler` ×2 (một chiếc `truncated=yes`); người + `motorcycle`; van → `van`; phụ nữ → `person`, `truncated=yes` | Theo định nghĩa lớp | Quy tắc |
| img_035 | Auto rickshaw chở hàng chất cao; người đi mô tô; ô tô trắng | R-11, R-02, R-03 | Auto rickshaw → `three_wheeler`, box gồm cả hàng chất trên xe; người + `motorcycle`; ô tô → `car` | Hàng gắn chặt trên xe là một phần chiếm chỗ của xe | Quy tắc |
| img_023 | Ô tô đen; xích lô có người đạp | R-11, R-03 | Ô tô → `car`; xích lô → `bicycle` + người đạp → `person` | Xích lô chạy bằng sức người | Quy tắc |
| img_029 | Người đi xe máy chở ống dài; người đi xe đạp đỡ ống | R-03, R-02 | 2 `person` + 1 `motorcycle` + 1 `bicycle`; ống dài **không** tính vào box nào | Ống do người vác, không gắn với một xe | Quy tắc |
| img_022 | Ảnh đen trắng cũ: mô tô có thùng bên | R-15, lớp `motorcycle` | 1 `motorcycle` bao cả thùng bên | Sidecar gắn liền với xe máy | Quy tắc |
| img_036 | Xe tải nhỏ thùng bạt; hàng xe máy đỗ phía sau | R-10, R-06 | Xe tải nhỏ → `truck`; xe máy phía sau tách được → box riêng, không tách được → `ignore_region` | Có khoang hàng tách cabin | Quy tắc |
| img_037 | Pickup có nắp thùng; xe buýt; người đi bộ | R-10 | Pickup → `truck`; xe buýt → `bus`; người → `person` | Nắp thùng không biến xe thành van | Quy tắc |
| img_038 | Tủ kính xe mô hình và thú nhồi bông | R-01 | **Không gán** (0 box) | Toàn bộ là đồ chơi | Quy tắc |
| img_032 | Hàng dài người chờ xe buýt chồng lấp; ô tô cắt mép trái | R-06, R-08 | Người tiền cảnh tách được → `person`; phần chồng lấp → `ignore_region`; ô tô → `car`, `truncated=yes` | Đám đông không tách được biên | Quy tắc |

---

## 5. QC và phiên bản

**Ngưỡng QC áp dụng** (hằng số của lab trong `taxonomy.json`, không đổi):

| Chỉ số | Ngưỡng |
|---|---|
| Ghép box khi so IAA | IoU từ 0.5 |
| IoU trung bình trên cặp box ghép | Từ 0.85 |
| Đồng thuận lớp trên cặp box ghép | Từ 95% |
| IoU theo từng box | Box cao từ 50 px dùng 0.85, nhỏ hơn dùng 0.7 |
| Chiều cao box tối thiểu | 20 px (R-05) |
| Tỷ lệ ảnh lỗi để Release | Tối đa 2%, duyệt toàn bộ lô |

**Lịch sử thay đổi:**

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v0.1-101 | Thêm mục tiêu và mức nguy hiểm; định nghĩa và phản ví dụ cho từng lớp; đề xuất lớp `bus`, `bicycle`, `three_wheeler`; ngưỡng % cho `occluded_level` và `truncated`; 16 quy tắc R-01…R-16; 33 ca khó có mã ảnh | v0 không trả lời được: người trên xe máy (img_026), tượng (img_017, img_027), phản chiếu (img_020), xe chở xe (img_002), tuk-tuk (img_004), pickup (img_012), đám đông (img_008, img_021) |
