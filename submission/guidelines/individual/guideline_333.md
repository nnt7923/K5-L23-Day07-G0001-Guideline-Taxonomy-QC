# Guideline gán nhãn — v0.2-draft — 333

> **Trạng thái:** Bản nháp guideline cá nhân sau khi rà 40 ảnh `img_001`–`img_040`.
> Người gán phải tự kiểm lại từng quyết định trước khi dùng. Đây chưa phải guideline nhóm v1.0
> và không thay thế bước họp thống nhất của nhóm.

- Phiên bản: `v0.2-draft`
- Ngày: 2026-11-16
- Người viết: `333`
- Người duyệt: Chưa duyệt
- Loại nhãn: Bounding box trên phần nhìn thấy của vật
- Định dạng xuất: COCO 1.0

## 1. Mục tiêu

Tạo dữ liệu để huấn luyện và đánh giá mô hình phát hiện **người** và một số **phương tiện đường bộ**
(`car`, `truck`, `van`, `motorcycle`, `bicycle`, `three_wheeler`, `bus`) trong ảnh đường phố 2D.
Ba lớp `bicycle`, `three_wheeler`, `bus` là đề xuất mở rộng của guideline cá nhân này; nhóm phải chốt
và thêm chúng vào `submission/taxonomy.json` trước khi sinh cấu hình CVAT. Kết quả có thể được dùng cho quan sát
giao thông hoặc hỗ trợ nhận biết vật cản, vì vậy các lỗi bỏ sót người/phương tiện, gán nhầm lớp,
hoặc vẽ box sai đáng kể đều có thể làm mô hình học sai vị trí và loại đối tượng.

Mục tiêu của guideline là để hai người độc lập nhìn cùng một ảnh và đưa ra cùng quyết định về:

1. Vật nào nằm trong phạm vi gán nhãn.
2. Mỗi vật nhận lớp nào và nhận đúng một box hay không.
3. Box bao phần nào của vật.
4. Cách đặt `occluded_level`, `truncated`, `can_review`.
5. Khi nào phải bỏ qua, dùng `ignore_region`, hoặc ghi vấn đề để reviewer chốt.

Đơn vị gán nhãn là **một vật thể vật lý có thật, nhìn thấy trực tiếp trong cảnh**. Hình in, ảnh trên
biển quảng cáo, phản chiếu, tượng, ma-nơ-canh và đồ chơi không phải vật thể mục tiêu.

## 2. Lớp và thuộc tính

### 2.1. Lớp

| Lớp | Định nghĩa thao tác | Ví dụ trong bộ ảnh | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Xe cơ giới chở người, thân kín, thường là sedan, hatchback, wagon, SUV, MPV hoặc taxi; xe thật vẫn gán khi đang đỗ hoặc trưng bày. | Sedan/SUV ở `img_012`, `img_015`; taxi ở `img_040`; các ô tô thật trong bảo tàng ở `img_024`. | Pickup có thùng rời là `truck`; van thương mại là `van`; xe chở khách nhiều chỗ là `bus`; tuk-tuk là `three_wheeler`; xe đồ chơi và tượng xe không gán. |
| `truck` | Xe cơ giới có chức năng chở hàng rõ ràng, gồm pickup thùng hở, xe tải thùng rời, xe tải nhẹ và đầu xe kèm thân/rơ-moóc đang gắn. | Pickup ở `img_001`, `img_012`, `img_028`, `img_037`; xe chở ô tô ở `img_002`; xe tải nhẹ ở `img_036`. | SUV/MPV thân kín là `car`; panel van liền khối là `van`; tượng pickup `img_027` không gán. |
| `van` | Xe thân kín cao, khoang người/hàng liền với cabin và có hình dáng van/panel van thương mại cỡ nhỏ hoặc vừa. | Van trắng ở `img_016`; panel/service van nếu nhìn thấy rõ ở `img_037`. | Pickup có nắp thùng vẫn là `truck`; SUV/minivan chở người có dáng ô tô là `car`; xe chở khách nhiều hàng ghế là `bus`. |
| `motorcycle` | Xe cơ giới hai bánh, gồm mô tô, scooter và moped; xe máy gắn sidecar được coi là một xe và box gồm cả sidecar. | Scooter/xe máy ở `img_003`, `img_005`, `img_006`, `img_008`, `img_010`, `img_026`, `img_029`; xe sidecar ở `img_022`. | Xe đạp/xích lô đạp là `bicycle`; tuk-tuk/auto-rickshaw là `three_wheeler`; xe máy đồ chơi không gán. |
| `bicycle` | Phương tiện chạy chủ yếu bằng bàn đạp, có hai hoặc ba bánh; gồm xe đạp thường và xích lô/cyclo đạp ba bánh. Box xích lô bao toàn bộ khung xe, bánh và khoang ghế gắn liền. | Xe đạp ở `img_003`, `img_006`, `img_029`, `img_034`, `img_039`; xích lô ở `img_023`. | Xe máy/scooter là `motorcycle`; tuk-tuk có động cơ và cabin là `three_wheeler`; người và hàng trên xe có box riêng hoặc không thuộc box xe. |
| `three_wheeler` | Phương tiện cơ giới ba bánh có cabin/mui và khoang chở người hoặc hàng, gồm tuk-tuk và auto-rickshaw. | Tuk-tuk ở `img_004`, `img_016`, `img_035`, `img_040`. | Xích lô đạp là `bicycle`; xe máy gắn sidecar là `motorcycle`; ô tô bốn bánh là `car`. |
| `bus` | Xe cơ giới được thiết kế chở nhiều hành khách, có thân dài/cao và nhiều hàng ghế; gồm minibus, city bus và coach khi nhận diện được chắc chắn. | Minibus ở `img_009`; bus ở `img_037`, `img_040`. | Van nhỏ/ít hàng ghế là `van`; MPV/SUV là `car`; xe tải chở hàng là `truck`. |
| `person` | Một người thật nhìn thấy trực tiếp, bất kể đang đi bộ, đứng, ngồi, lái xe hay ngồi trên phương tiện. Mỗi người một box. | Người đi bộ `img_014`, người trên xe máy `img_026`, tài xế xe lạ mắt `img_019`, đám đông `img_032`. | Tượng `img_017`, ma-nơ-canh `img_033`, người trên billboard `img_018`, mascot/đồ chơi `img_019` và `img_038`, người chỉ xuất hiện trong phản chiếu không gán. |
| `ignore_region` | Một **vùng ảnh**, không phải một vật. Chỉ dùng khi vùng chứa nhiều vật mục tiêu chồng lấp/dày đặc đến mức không thể tách từng instance một cách đáng tin cậy, hoặc vùng quá mờ khiến mọi ranh giới instance đều không xác định được. | Có thể dùng cho phần lõi không thể tách của dòng xe/người rất dày ở `img_021` hoặc bãi xe `img_030`, sau khi đã gán riêng các vật tách được. | Không dùng thay cho một `bus`, `three_wheeler`, `bicycle` hay vật khó; không dùng để che lỗi; không bao các vật đã có box riêng. |

### 2.2. Quy tắc chọn lớp

- **R-01 — Một vật, một lớp:** mỗi vật mục tiêu nhận đúng một lớp và một box. Không gán đồng thời
  `car` và `truck` cho cùng xe.
- **R-02 — Car và truck:** xe có thùng hàng hở hoặc thân hàng tách rõ khỏi cabin là `truck`; xe chở
  người thân kín kiểu sedan/SUV/wagon/MPV là `car`.
- **R-03 — Truck và van:** thân hàng/cabin liền khối, kín và có dáng panel van là `van`; pickup hoặc
  cabin + thùng hàng tách rõ là `truck`, kể cả khi pickup có nắp che thùng.
- **R-04 — Người và xe:** người lái/ngồi trên xe vẫn có box `person` riêng; phương tiện có box riêng.
  Các box được phép chồng nhau.
- **R-05 — Bus, xe đạp và xe cơ giới ba bánh:** gán minibus/city bus/coach là `bus`; xe đạp hai bánh
  và xích lô đạp ba bánh là `bicycle`; tuk-tuk/auto-rickshaw có động cơ là `three_wheeler`. Người thật
  lái hoặc ngồi trên các phương tiện này vẫn nhận box `person` riêng. Xe ngựa và loài vật ngoài taxonomy
  nên không gán box.
- **R-06 — Vật thật và hình biểu diễn:** không gán tượng, ma-nơ-canh, đồ chơi/mô hình, hình in, màn hình,
  billboard, tranh hoặc phản chiếu. Xe thật đang đỗ/trưng bày vẫn gán; vật chỉ có hình dáng xe nhưng là
  tượng/đồ trang trí thì không gán.
- **R-07 — Xe chở xe:** xe tải chở xe nhận một box `truck` bao toàn bộ tổ hợp xe tải/thân chở đang gắn;
  mỗi ô tô thật được chở nhận một box `car` riêng nếu phần nhìn thấy cao ít nhất 20 px.

### 2.3. Thuộc tính

Mọi box trừ `ignore_region` phải có đủ ba thuộc tính dưới đây.

#### `occluded_level`

Ước lượng tỷ lệ diện tích của **toàn vật** bị vật thể khác che trong phạm vi ảnh. Mép ảnh cắt mất vật
không tính là occlusion mà dùng `truncated`. Mờ do chuyển động, bóng tối hoặc phản xạ không tự động tính
là che khuất.

| Giá trị | Ngưỡng phần vật bị che | Cách hiểu |
|---|---:|---|
| `0` | 0–10% | Gần như thấy trọn vật. |
| `1` | >10–35% | Che ít; lớp và biên phần lớn còn rõ. |
| `2` | >35–70% | Che nhiều; vẫn nhận ra chắc chắn vật và lớp. |
| `3` | >70% | Chỉ còn một phần nhỏ nhưng vẫn đủ bằng chứng nhận ra instance; thường cần cân nhắc `can_review`. |

Nếu không đủ bằng chứng khẳng định có một vật mục tiêu riêng biệt thì không tạo box chỉ dựa trên phỏng đoán.

#### `truncated`

| Giá trị | Điều kiện |
|---|---|
| `no` | Toàn bộ vật nằm trong khung ảnh, dù có thể bị vật khác che. |
| `yes` | Vật tiếp tục ra ngoài một hay nhiều mép ảnh; phần thấy được chạm/cắt bởi biên ảnh. |

Một vật nằm sát mép nhưng vẫn nhìn thấy hết là `no`. `truncated=yes` có thể đi cùng mọi mức occlusion.

#### `can_review`

- Không tick khi quyết định đã rõ theo guideline.
- Tick khi đã vẽ được box nhưng còn một bất định thực sự về lớp, ranh giới instance, phần trăm che khuất,
  hoặc vật là thật/đồ trưng bày.
- Không dùng `can_review` để thay cho việc đọc guideline.
- Nếu chưa thể vẽ một box có căn cứ, không tạo box giả; ghi `image`, mã người gán, vùng và lý do vào
  `submission/qc/image_issues.csv` sau khi khóa nhãn độc lập.

## 3. Quy tắc vẽ box

### 3.1. Box cơ bản

- **R-08 — Phần nhìn thấy:** box chữ nhật nhỏ nhất bao hết các pixel nhìn thấy của một vật, không suy diễn
  tới kích thước đầy đủ phía sau vật che hoặc ra ngoài mép ảnh.
- **R-09 — Nhiều mảnh nhìn thấy:** nếu biết chắc các mảnh rời thuộc cùng một vật, dùng một box nhỏ nhất
  bao tất cả các mảnh; khoảng giữa do vật che vẫn nằm trong box.
- **R-10 — Không gộp instance:** hai người hoặc hai xe sát nhau vẫn là hai box nếu phân biệt được. Không
  gộp người lái với xe, không gộp nhiều xe đỗ thành một box.
- **R-11 — Kích thước tối thiểu:** chỉ gán khi chiều cao box của phần nhìn thấy đạt **ít nhất 20 px**.
  Dưới 20 px thì bỏ qua. Ngưỡng lấy từ `taxonomy.json.rules.min_box_height_px`.
- **R-12 — Box sát:** cạnh box đi qua cực trái/phải/trên/dưới của phần nhìn thấy; tránh lấy bóng đổ,
  khoảng trời/đường hoặc hàng hóa nhô ra không phải bộ phận xe.

### 3.2. Người và người trên phương tiện

- Mỗi người thật nhận một box `person`, kể cả người ngồi sau, người trong cabin hoặc người bị che, nếu
  vẫn nhận ra chắc chắn và chiều cao phần nhìn thấy đạt 20 px.
- Box người chỉ bao cơ thể và đồ mặc/đội gắn với người; không bao xe, hàng đang xách hoặc thanh kim loại
  đang mang nếu chúng nhô xa khỏi cơ thể.
- Người trên xe máy: một box cho xe máy và một box cho từng người. Ví dụ `img_026` có một xe máy và hai
  người thật; các box được chồng nhau.
- Người xuất hiện trên billboard, poster, màn hình, tranh, gương hoặc bề mặt phản chiếu không gán.

### 3.3. Phương tiện có tải, phụ kiện hoặc nhiều phần

- Xe tải: box gồm cabin, thân/thùng và rơ-moóc đang gắn thành cùng tổ hợp vận hành; không bao hàng hóa
  đặt trên xe nếu hàng nhô ra ngoài thân xe.
- Xe chở xe `img_002`: một `truck` cho xe vận chuyển; mỗi xe con thật trên giá là một `car`. Khung chở
  che xe con được phản ánh bằng `occluded_level` của từng `car`.
- Pickup chở hàng `img_012`: box `truck` ôm thân pickup, không kéo box theo các bao/thùng hàng vượt ra
  ngoài thùng xe.
- Pickup chở tượng ngựa `img_028`: pickup là `truck`; tượng ngựa không gán.
- Xe máy sidecar `img_022`: coi xe máy và sidecar gắn cố định là một instance `motorcycle`, một box chung.
- Giá chở đồ, thùng gắn cố định hoặc mui xe được tính vào phương tiện; tải rời, người và động vật không tính.

### 3.4. Lớp mở rộng, vật ngoài phạm vi và cảnh phản chiếu

- Tuk-tuk/auto-rickshaw ở `img_004`, `img_016`, `img_035`, `img_040` nhận một box
  `three_wheeler` cho mỗi xe. Box bao thân xe, cabin/mui, bánh và khoang chở gắn liền; không bao người
  hoặc hàng rời. Không gộp nhiều tuk-tuk vào một box.
- Minibus ở `img_009` và bus ở `img_037`, `img_040` nhận lớp `bus`. Mỗi xe một box, kể cả khi chỉ
  nhìn thấy một phần nhưng vẫn nhận diện chắc chắn và phần nhìn thấy cao ít nhất 20 px.
- Xe đạp ở `img_003`, `img_006`, `img_029`, `img_034`, `img_039` nhận lớp `bicycle`. Xích lô ở
  `img_023` cũng là `bicycle` vì vận hành bằng bàn đạp; box bao toàn bộ khung, bánh và khoang ghế gắn liền.
  Người thật lái/ngồi trên xe vẫn nhận box `person` riêng.
- Xe máy gắn sidecar ở `img_022` vẫn là `motorcycle`, không phải `three_wheeler`, vì cấu trúc chính là
  xe máy hai bánh với thùng phụ gắn bên cạnh.
- Tượng/người giả ở `img_011`, `img_017`, `img_027`, `img_028`, `img_033` không gán.
- Đồ chơi/mô hình xe ở `img_038` không gán.
- `img_020` chỉ cho thấy phương tiện qua gương chiếu hậu: không gán vật trong gương. Ở `img_031`, gán
  xe/người thật trong cảnh nhưng không gán bản sao phản chiếu trên mặt đường ướt.

### 3.5. Đám đông, che khuất và ảnh khó

- Gán riêng mọi instance đạt 20 px và tách được một cách có căn cứ, kể cả trong đám đông.
- Với vùng lõi dày đặc không thể tách instance, vẽ một `ignore_region` sát vùng không phân giải được;
  trước đó gán riêng các vật ở rìa còn tách được. Không để `ignore_region` chồng lên box đã gán riêng.
- Ảnh mờ/chuyển động: nếu nhận ra chắc chắn lớp và biên phần nhìn thấy thì vẫn gán; nếu box có thể vẽ
  nhưng còn bất định thì tick `can_review`; nếu không đủ căn cứ thì ghi vấn đề cấp ảnh.
- Hàng rào, kính, cây, xe khác hoặc người khác là vật che và ảnh hưởng `occluded_level`. Độ tối hoặc blur
  không phải occlusion.

## 4. Ca khó đã rà trong 40 ảnh

| Ảnh | Tình huống | Rule ID | Quyết định draft | Lý do |
|---|---|---|---|---|
| `img_001` | Pickup và nhiều người, một số người bị xe che. | R-02, R-08 | Pickup=`truck`; mỗi người thật đủ 20 px một `person`; đặt occlusion riêng. | Thùng hàng tách cabin; người là instance độc lập. |
| `img_002` | Xe tải chở nhiều ô tô. | R-07 | Một `truck` cho tổ hợp vận chuyển; mỗi ô tô thật đủ 20 px một `car`. | Xe được chở vẫn là vật thật; khung chở chỉ gây che khuất. |
| `img_003` | Người, xe đạp và xe máy xen nhau. | R-04, R-05 | Gán từng `person`, `bicycle`, `motorcycle` đủ ngưỡng. | Ba loại là các instance riêng; box được phép chồng khi người ngồi trên xe. |
| `img_004` | Hai tuk-tuk và tài xế. | R-04, R-05 | Mỗi tuk-tuk một `three_wheeler`; mỗi người thật nhìn thấy rõ một `person`. | Xe cơ giới ba bánh có cabin thuộc lớp riêng, không ép vào motorcycle/car. |
| `img_009` | Minibus/bus và xe máy đỗ. | R-05 | Minibus=`bus`; gán thêm `motorcycle`/`person` đủ ngưỡng. | Minibus có thân và nhiều hàng ghế chở khách. |
| `img_011` | Tác phẩm điêu khắc cơ khí giống người/xe. | R-06 | Không gán tác phẩm; chỉ gán người thật ở hậu cảnh nếu đạt 20 px. | Vật biểu diễn không phải mục tiêu thật. |
| `img_012` | Pickup chở hàng, SUV và xe khác che nhau. | R-02, R-12 | Pickup=`truck`, SUV=`car`; box pickup không bao hàng nhô khỏi thân. | Phân lớp theo cấu tạo và mục đích thân xe. |
| `img_014` | Người rõ, xe máy nhòe do chuyển động. | R-04, R-08 | Gán hai người; gán xe máy nhòe nếu biên đủ xác định, nếu không ghi issue. | Blur không phải occlusion nhưng có thể làm biên không đáng tin. |
| `img_015` | Nhiều ô tô bị cắt bởi mép ảnh. | R-08 | Mỗi xe đủ ngưỡng một `car`, box phần nhìn thấy, `truncated=yes`. | Vật tiếp tục ra ngoài ảnh. |
| `img_016` | Tuk-tuk, van, pickup và scooter. | R-02, R-03, R-05 | Tuk-tuk=`three_wheeler`; van=`van`; pickup=`truck`; scooter=`motorcycle`; người riêng. | Phân lớp theo cấu trúc và nguồn truyền động của từng xe. |
| `img_017` | Tượng người cưỡi ngựa. | R-06 | Không gán `person`. | Tượng không phải người thật. |
| `img_018` | Người in trên billboard và ô tô thật sau hàng rào. | R-06, R-11 | Không gán người trên biển; gán xe thật đủ 20 px, với occlusion do hàng rào. | Phân biệt hình in với vật thật. |
| `img_019` | Xe tạo hình giả tưởng, tài xế thật và mascot chuột. | R-06 | Xe có biển bánh và đang vận hành=`car`; tài xế=`person`; mascot không gán. | Chức năng vật thật quyết định lớp, không phải hình thức trang trí. |
| `img_020` | Xe chỉ xuất hiện trong gương chiếu hậu. | R-06 | Không gán xe phản chiếu. | Không phải vật nhìn trực tiếp trong cảnh ảnh. |
| `img_021` | Dòng người và xe máy cực dày. | R-10, R-11 | Gán riêng instance gần/tách được; dùng `ignore_region` cho lõi không tách được; bỏ vật dưới 20 px. | Không được gộp tùy tiện nhưng cũng không đoán instance. |
| `img_022` | Xe máy gắn sidecar trong ảnh cũ. | R-01 | Một box `motorcycle` gồm xe và sidecar. | Sidecar là bộ phận gắn của một phương tiện. |
| `img_023` | Car, motorcycle, xích lô và người lái. | R-04, R-05 | Gán `car`, `motorcycle`, `bicycle` cho xích lô và từng `person` đủ ngưỡng. | Xích lô ba bánh nhưng vận hành bằng bàn đạp nên thuộc bicycle. |
| `img_024` | Xe thật trưng bày và vật hình xe màu vàng trên nóc. | R-06 | Gán các ô tô thật; vật vàng thiếu cabin/nội thất rõ ràng: không gán và ghi reviewer kiểm lại nếu còn nghi ngờ. | Xe thật vẫn là mục tiêu; mô hình/prop không phải mục tiêu. |
| `img_026` | Hai người trên một xe máy. | R-04 | Một `motorcycle` và hai `person`; box được chồng nhau. | Mỗi người là một instance độc lập. |
| `img_027` | Tượng/mộ tạo hình pickup. | R-06 | Không gán `truck`. | Tượng không phải phương tiện thật. |
| `img_028` | Pickup thật chở tượng ngựa, có car phía sau. | R-02, R-06 | Pickup=`truck`; car nền=`car`; tượng ngựa không gán. | Tách vật thật khỏi tải trang trí. |
| `img_029` | Người mang thanh dài, xe máy và xe đạp. | R-05, R-12 | Người, `motorcycle` và `bicycle` là các box riêng; box người không bao hết thanh dài. | Thanh là vật mang theo, không phải cơ thể. |
| `img_030` | Bãi xe máy dày đặc và chồng lấp. | R-10 | Gán từng motorcycle tách được; vùng lõi không tách được dùng `ignore_region`. | Tránh box suy đoán hoặc một box gộp nhiều xe. |
| `img_031` | Cảnh đêm, xe thật và phản chiếu trên đường ướt. | R-06 | Gán vật thật đủ ngưỡng; không gán ảnh phản chiếu trên mặt đường. | Phản chiếu không phải instance mới. |
| `img_032` | Đám đông ở trạm và car bị cắt mép. | R-10, R-11 | Gán từng người đủ 20 px; car mép trái `truncated=yes`; vùng người không tách được có thể ignore. | Dùng cùng ngưỡng kích thước và quy tắc mép ảnh. |
| `img_033` | Ma-nơ-canh, hình quảng cáo và phản chiếu xe. | R-06 | Không gán person/vehicle. | Tất cả mục tiêu giống thật đều là biểu diễn hoặc phản chiếu. |
| `img_035` | Tuk-tuk, motorcycle, car và người. | R-04, R-05 | Mỗi tuk-tuk=`three_wheeler`; gán thêm motorcycle, car và người đủ ngưỡng. | Xe cơ giới ba bánh có lớp riêng. |
| `img_036` | Xe tải nhẹ có cabin và thùng kín tách biệt. | R-03 | `truck`. | Thùng hàng tách cabin, không phải van liền khối. |
| `img_037` | Pickup có canopy, hai bus và người đi bộ. | R-02, R-05 | Pickup=`truck`; mỗi bus nhìn thấy được một box `bus`; người đủ ngưỡng=`person`. | Canopy không biến pickup thành van; bus là lớp mục tiêu đề xuất. |
| `img_038` | Tủ xe mô hình và đồ chơi. | R-06 | Không gán car/person. | Đồ chơi không phải vật thật theo mục tiêu. |
| `img_040` | Người tiền cảnh, taxi/car, tuk-tuk và bus. | R-04, R-05 | Gán `person`, `car`, `three_wheeler`, `bus` đủ ngưỡng; box người tiền cảnh theo phần thấy được. | Mỗi loại phương tiện nhận đúng lớp riêng; xử lý che khuất độc lập. |

### Checklist rà đủ 40 ảnh

Bảng này không thay cho nhãn COCO và không khẳng định số lượng box. Nó ghi điểm cần chú ý để người gán
không bỏ qua ca biên khi mở từng ảnh ở kích thước gốc.

| Ảnh | Điểm cần kiểm khi gán |
|---|---|
| `img_001` | Pickup=`truck`; nhiều người bị che. |
| `img_002` | Xe chở xe: truck và từng car riêng. |
| `img_003` | Gán riêng `person`, `motorcycle`, `bicycle` đủ ngưỡng. |
| `img_004` | Mỗi tuk-tuk=`three_wheeler`; người thật gán riêng. |
| `img_005` | Nhiều người, motorcycle và car chồng lấp; tách instance. |
| `img_006` | Dòng motorcycle/person/bicycle; gán từng instance và chú ý mép ảnh. |
| `img_007` | Phân biệt truck, pickup và car trong gara; che khuất cao. |
| `img_008` | Nhiều motorcycle/person, một số bị cắt hoặc che. |
| `img_009` | Minibus=`bus`; motorcycle/person đủ ngưỡng vẫn gán riêng. |
| `img_010` | Motorcycle/person và hàng hóa; box không bao tải rời. |
| `img_011` | Điêu khắc không gán; người thật hậu cảnh theo ngưỡng 20 px. |
| `img_012` | Pickup=`truck`; hàng không tính vào box; car/SUV riêng. |
| `img_013` | Cảnh đêm: chỉ gán vật đủ 20 px và nhận ra chắc chắn. |
| `img_014` | Người thật rõ; motorcycle nhòe cần đánh giá biên box. |
| `img_015` | Nhiều car bị cắt mép: `truncated=yes`. |
| `img_016` | Tuk-tuk=`three_wheeler`; van/truck/motorcycle/person phân lớp riêng. |
| `img_017` | Tượng người cưỡi ngựa không gán. |
| `img_018` | Billboard không gán; xe thật sau hàng rào có thể gán. |
| `img_019` | Xe giả tưởng nhưng vận hành thật=`car`; người thật gán; mascot không. |
| `img_020` | Không gán vật chỉ thấy trong gương. |
| `img_021` | Đám đông cực dày; individual trước, ignore phần không tách được. |
| `img_022` | Xe máy sidecar là một `motorcycle`. |
| `img_023` | Xích lô=`bicycle`; car/motorcycle/person gán thành các instance riêng. |
| `img_024` | Xe thật trưng bày có gán; prop/mô hình không gán. |
| `img_025` | Người và xe ở xa: áp dụng đúng ngưỡng 20 px. |
| `img_026` | Một motorcycle, hai person; các box chồng nhau. |
| `img_027` | Tượng pickup không gán. |
| `img_028` | Pickup thật=`truck`, tượng ngựa không, car nền có gán. |
| `img_029` | Motorcycle/person/bicycle có gán; thanh dài ngoài box người. |
| `img_030` | Xe máy đỗ dày: tách instance hoặc ignore lõi không phân giải. |
| `img_031` | Gán vật thật, không gán phản chiếu trên đường ướt. |
| `img_032` | Đám đông và car cắt mép; ngưỡng 20 px. |
| `img_033` | Ma-nơ-canh, quảng cáo, phản chiếu: không gán. |
| `img_034` | Person/motorcycle/car/bicycle có gán; vật xa theo ngưỡng. |
| `img_035` | Tuk-tuk=`three_wheeler`; motorcycle/car/person gán riêng. |
| `img_036` | Xe tải nhẹ thùng tách cabin=`truck`. |
| `img_037` | Pickup canopy=`truck`; mỗi bus=`bus`; person đủ ngưỡng có gán. |
| `img_038` | Toàn bộ xe/người đồ chơi không gán. |
| `img_039` | Motorcycle/person/bicycle đủ ngưỡng có gán. |
| `img_040` | Gán person/car/three_wheeler/bus đủ ngưỡng thành các box riêng. |

## 5. QC và phiên bản

### 5.1. Ngưỡng QC cố định

Các ngưỡng dưới đây lấy nguyên từ `submission/taxonomy.json.rules`; không tự ý sửa trong guideline nhóm:

| Tên | Giá trị | Ý nghĩa thao tác |
|---|---:|---|
| `min_box_height_px` | 20 | Không gán box có chiều cao phần nhìn thấy dưới 20 px. |
| `match_iou` | 0.50 | Ngưỡng ghép hai box để so sánh. |
| `large_px` | 50 | Mốc phân biệt vật lớn/nhỏ khi kiểm IoU. |
| `iou_thr_large` | 0.85 | Ngưỡng IoU mong đợi cho vật lớn. |
| `iou_thr_small` | 0.70 | Ngưỡng IoU mong đợi cho vật nhỏ. |
| `iaa_iou` | 0.85 | Mức IAA IoU mục tiêu. |
| `iaa_class` | 0.95 | Mức đồng thuận lớp mục tiêu. |
| `error_rate_img_max` | 0.02 | Tỷ lệ ảnh lỗi tối đa trước khi cần Rework theo cây QC. |

Script kiểm được tên lớp, thuộc tính, hình học box, kích thước tối thiểu và độ lệch giữa người gán. Người
review vẫn phải kiểm bằng mắt để phát hiện box không ôm sát, gán sai thực tế, vật mà mọi người cùng bỏ sót,
hoặc mọi người cùng hiểu sai guideline.

### 5.2. Quy trình xử lý ca mơ hồ

1. Người gán áp dụng Rule ID gần nhất và làm độc lập.
2. Nếu vẽ được box nhưng còn bất định, tick `can_review`; nếu không vẽ được, ghi `image_issues.csv`.
3. Không hỏi/xem nhãn của người khác trước khi tất cả nộp bản độc lập.
4. Sau khi chạy IAA, nhóm mở ảnh gốc và overlay, ghi mỗi ca vào `calibration_decisions.csv`.
5. Người chốt phải dùng mã ba chữ số; người kiểm lại nên khác người sửa.
6. Quyết định làm thay đổi tập vật/lớp/box/thuộc tính là thay đổi semantic và phải tăng phiên bản guideline,
   rà lại phạm vi bị ảnh hưởng.

### 5.3. Các điểm bắt buộc nhóm chốt trước v1.0

- Có chấp nhận ba lớp đề xuất `bicycle`, `three_wheeler`, `bus` và cập nhật đồng bộ `taxonomy.json`,
  cấu hình CVAT hay không. Nếu nhóm không chấp nhận thì phải chốt lại cách xử lý trước khi gán độc lập.
- `truck` có bao cả rơ-moóc đang gắn hay chỉ cabin; draft này chọn toàn tổ hợp.
- Xe thật trưng bày và vật hình xe dạng prop ở `img_024` phân biệt theo dấu hiệu nào.
- Khi nào đám đông chuyển từ box riêng sang `ignore_region`; reviewer cần xem `img_021`, `img_030`, `img_032`.
- Ngưỡng `occluded_level` 10% / 35% / 70% có được cả nhóm chấp nhận.
- Người trong cabin/kính xe có gán khi đủ 20 px hay không; draft này chọn có gán nếu nhận ra chắc chắn.

### 5.4. Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do / ca liên quan |
|---|---|---|
| `v0` | Guideline starter do giảng viên cung cấp. | Chưa xử lý ca biên. |
| `v0.1-draft` | Định nghĩa lớp, ngưỡng thuộc tính, quy tắc box, ngoài taxonomy và rà 40 ảnh. | Chuẩn bị guideline cá nhân của mã `333`; cần người viết tự duyệt. |
| `v0.2-draft` | Đề xuất thêm `bicycle`, `three_wheeler`, `bus`; sửa quyết định cho xe đạp, xích lô, tuk-tuk và bus. | Rà lại `img_003`, `img_004`, `img_006`, `img_009`, `img_016`, `img_023`, `img_029`, `img_034`, `img_035`, `img_037`, `img_039`, `img_040`. |

