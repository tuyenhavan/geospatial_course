# Bài 32: Sử dụng ảnh MODIS và ERA5-land theo dõi hạn hán

Hạn hán là một trong những thảm họa tự nhiên nghiêm trọng, ảnh hưởng trực tiếp đến sản xuất nông nghiệp, an ninh lương thực và đời sống người dân. Việc theo dõi và cảnh báo sớm hạn hán là điều cần thiết để có các biện pháp ứng phó kịp thời. Viễn thám vệ tinh với khả năng quan sát liên tục và bao phủ diện rộng đã trở thành công cụ hiệu quả trong giám sát hạn hán.

Trong bài học này, chúng ta sẽ sử dụng dữ liệu MODIS để tính toán chỉ số tình trạng thực vật (VCI - Vegetation Condition Index) và dữ liệu ERA5-Land để tính toán chỉ số bất thường lượng mưa (Precipitation Anomaly Index). Hai chỉ số này kết hợp với nhau cung cấp cái nhìn toàn diện về tình trạng hạn hán, từ góc độ tình trạng thực vật và lượng mưa, giúp đánh giá mức độ nghiêm trọng và phạm vi ảnh hưởng của hạn hán trên khu vực nghiên cứu.

> **Lưu ý**
> 
> Bạn có thể chạy trực tiếp notebook này bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/1COpwmlu8fIU1z2LObsOllYW5JKd3kn_x) mà không cần cài đặt. Để tránh làm thay đổi nội dung gốc và thuận tiện cho việc lưu kết quả, hãy tạo một bản sao ( File → Save a copy in Drive ) trước khi chạy và chỉnh sửa mã nguồn trong notebook.

## 32.1. Mục tiêu bài học

Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Hiểu khái niệm và ý nghĩa của các chỉ số hạn hán trong viễn thám
- Tải và xử lý dữ liệu MODIS EVI từ Google Earth Engine
- Tính toán chỉ số VCI (Vegetation Condition Index) từ dữ liệu thực vật
- Tải và xử lý dữ liệu ERA5-Land về lượng mưa
- Tính toán chỉ số bất thường lượng mưa (Precipitation Anomaly Index)
- Trực quan hóa các chỉ số hạn hán trên bản đồ tương tác

## 32.2. Chuẩn bị dữ liệu

Để thuận tiện, chúng ta sẽ dùng một gói python `geesat` đơn giản, cung cấp các hàm cho việc xử lý và tính toán các chỉ số hạn hán. Để sử dụng thư viện `geesat`, ta có thể cài đặt như sau `pip install git+https://github.com/tuyenhavan/geesat.git`. Nếu bạn muốn biết thêm về thư viện này, bạn có thể tham khảo thêm tại [đây](https://github.com/tuyenhavan/geesat). 


```python
from geesat import geogee
import geesat
import ee 
import geemap 
import geopandas as gpd
ee.Authenticate()
ee.Initialize(project='geocourse-501706')
```

## 32.3. Tính toán chỉ số hạn hán

Có nhiều chỉ số khác nhau để đánh giá hạn hán, mỗi chỉ số dựa trên các yếu tố khác nhau như lượng mưa, nhiệt độ, độ ẩm đất, hay tình trạng thực vật. Trong bài học này, chúng ta tập trung vào hai chỉ số chính:

- **VCI (Vegetation Condition Index)**: Đánh giá tình trạng sức khỏe thực vật dựa trên chỉ số thực vật EVI/NDVI từ ảnh vệ tinh MODIS
- **Precipitation Anomaly Index**: Đánh giá mức độ bất thường của lượng mưa so với giá trị trung bình lịch sử từ dữ liệu ERA5-Land

Việc kết hợp nhiều chỉ số giúp đánh giá toàn diện hơn về tình trạng hạn hán, vì hạn hán là hiện tượng phức tạp phụ thuộc vào nhiều yếu tố khí tượng và sinh học.

### 32.3.1. Tính toán chỉ số VCI

**Chỉ số VCI (Vegetation Condition Index)** là một chỉ số quan trọng trong giám sát hạn hán, phản ánh tình trạng sức khỏe thực vật qua việc so sánh giá trị EVI/NDVI hiện tại với phạm vi giá trị lịch sử (min-max) tại cùng một thời điểm trong năm. VCI được tính theo công thức:

$$
VCI = \frac{EVI - EVI_{min}}{EVI_{max} - EVI_{min}} \times 100
$$

Trong đó:
- $EVI$ là giá trị chỉ số thực vật tại thời điểm hiện tại
- $EVI_{min}$ và $EVI_{max}$ là giá trị nhỏ nhất và lớn nhất của EVI trong chuỗi dữ liệu lịch sử tại cùng tháng

VCI dao động từ 0 đến 100, với giá trị thấp (< 35) chỉ ra điều kiện thực vật kém, có khả năng xảy ra hạn hán, trong khi giá trị cao (> 65) thể hiện điều kiện thực vật tốt. Chỉ số này đặc biệt hữu ích trong nông nghiệp vì phản ánh trực tiếp tác động của hạn hán lên cây trồng.

- **Đọc và chuẩn bị dữ liệu**

Bước đầu tiên trong tính toán VCI là chuẩn bị dữ liệu MODIS EVI chất lượng cao. Chúng ta kết hợp dữ liệu từ cả hai vệ tinh Terra (MOD13A2) và Aqua (MYD13A2) để tăng tần suất quan sát và giảm khoảng trống do mây. Sau đó áp dụng cloud masking dựa trên band DetailedQA để loại bỏ các pixel bị ảnh hưởng bởi mây, bóng mây và tuyết. Dữ liệu được scale về khoảng 0-1 (nhân với 0.0001) theo tài liệu của NASA. Cuối cùng, tổng hợp ảnh theo tháng bằng median để tạo ra chuỗi thời gian EVI hàng tháng ổn định, loại bỏ nhiễu và cung cấp đầu vào tin cậy cho tính toán VCI.


```python
# Tải dữ liệu MODIS EVI từ Terra và Aqua
terra = ee.ImageCollection("MODIS/061/MOD13A2")
aqua = ee.ImageCollection("MODIS/061/MYD13A2")
# Kết hợp dữ liệu từ Terra và Aqua, sắp xếp theo thời gian giảm dần
modis = terra.merge(aqua).sort('system:time_start').filterDate('2000', '2026')
# Bỏ những ô pixel chất lượng kém
modis = geogee.generate_modis_cloud_mask(
    col=modis,
    from_bit=0,
    to_bit=1,
    qa_band='DetailedQA',
    threshold=1
).select(['EVI'])
# Chuyển đổi giá trị pixel về 0 đến 1
modis = geogee.generate_scaled_data(
    modis, scale_factor=0.0001
)
# Tổng hợp ảnh theo tháng
modis = geogee.generate_monthly_composite(
    col=modis,
    aggregate_method='median',
)
```

- **Tính chỉ số VCI**

Hàm `generate_monthly_vci()` từ gói geesat tự động tính toán VCI cho toàn bộ chuỗi thời gian MODIS EVI. Hàm này thực hiện các bước: (1) Nhóm dữ liệu theo tháng trong năm (tháng 1, 2, 3,...), (2) Tính giá trị min và max của EVI cho mỗi tháng từ toàn bộ dữ liệu lịch sử, (3) Chuẩn hóa giá trị EVI hiện tại theo công thức VCI để tạo ra giá trị từ 0-100. Kết quả là một ImageCollection chứa VCI hàng tháng, trong đó mỗi ảnh phản ánh tình trạng thực vật tại thời điểm đó so với lịch sử, giúp xác định các giai đoạn và khu vực bị ảnh hưởng bởi hạn hán.


```python
# Tính chỉ số thực vật VCI hàng tháng
vci = geogee.generate_monthly_vci(col=modis)
```

- **Trực quan ảnh**

Để kiểm tra kết quả VCI, chúng ta sử dụng geemap để hiển thị ảnh VCI đầu tiên trong collection trên bản đồ tương tác. Bảng màu được chọn từ màu đỏ sẫm (darkred) cho giá trị thấp (hạn hán nghiêm trọng) đến màu xanh đậm (darkgreen) cho giá trị cao (thực vật khỏe mạnh), với các mức trung gian là cam (orange), xám nhạt (lightgray) và xanh nhạt (lightgreen). Phạm vi giá trị từ 0-100 phản ánh đầy đủ điều kiện thực vật từ tồi tệ nhất đến tốt nhất. Bản đồ này cho phép quan sát không gian phân bố hạn hán và xác định các khu vực cần can thiệp.


```python
# Hiển thị bản đồ chỉ số thực vật VCI cho ảnh đầu tiên trong tập hợp ảnh
Map = geemap.Map()
Map.addLayer(vci.first(), {'min': 0, 'max': 100, 'palette': ['darkred', 'red', 'orange', 'lightgray', 'lightgreen', 'green', 'darkgreen']}, 'VCI')
Map
```

### 32.3.2. Tính toán chỉ số hạn dựa trên dữ liệu mưa

**Chỉ số bất thường lượng mưa (Precipitation Anomaly Index)** đánh giá mức độ chênh lệch của lượng mưa hiện tại so với giá trị trung bình lịch sử, được chuẩn hóa bởi độ lệch chuẩn. Chỉ số này giúp xác định các giai đoạn có lượng mưa bất thường thấp (hạn hán) hoặc cao (lũ lụt). Công thức tính:

$$
PAI = \frac{P - \bar{P}}{\sigma}
$$

Trong đó:
- $P$ là lượng mưa tại thời điểm hiện tại
- $\bar{P}$ là lượng mưa trung bình lịch sử tại cùng tháng
- $\sigma$ là độ lệch chuẩn của lượng mưa lịch sử

Giá trị PAI âm chỉ ra lượng mưa thấp hơn bình thường (nguy cơ hạn hán), với các mức độ:
- PAI < -2: Hạn hán nghiêm trọng
- -2 ≤ PAI < -1: Hạn hán vừa
- -1 ≤ PAI < 0: Thiếu mưa nhẹ

Ngược lại, giá trị PAI dương cao cho thấy lượng mưa dồi dào, có thể dẫn đến ngập lụt. Chỉ số này bổ sung cho VCI bằng cách cung cấp thông tin trực tiếp về nguồn nước từ mưa, yếu tố quan trọng quyết định tình trạng hạn hán.


```python
# Đọc dữ liệu lượng mưa từ ERA5-Land
era5land = ee.ImageCollection("ECMWF/ERA5_LAND/MONTHLY_AGGR").select(['total_precipitation_sum']).filterDate('2000', '2026')
# Tính chỉ số bất thường lượng mưa hàng tháng
pai = geogee.generate_monthly_anomaly_index(
    col=era5land,
    scaling_factor=1000 # Chuyển đổi từ m sang mm
)
```


```python
# Hiển thị bản đồ chỉ số bất thường lượng mưa cho ảnh đầu tiên trong tập hợp ảnh
Map = geemap.Map()
Map.addLayer(pai.first(), {'min': -3, 'max': 3, 'palette': ['darkred', 'red', 'orange', 'lightgray', 'lightgreen', 'green', 'darkblue']}, 'Precipitation')
Map
```

## Tóm tắt

Bạn đã hoàn thành Bài 32 và học được cách sử dụng dữ liệu viễn thám MODIS và ERA5-Land để theo dõi và đánh giá tình trạng hạn hán - kỹ năng quan trọng trong ứng dụng viễn thám phục vụ nông nghiệp, quản lý tài nguyên nước và cảnh báo thiên tai.

### Các khái niệm chính đã nắm vững:
- ✅ **Chỉ số VCI (Vegetation Condition Index)**: Tính toán và hiểu ý nghĩa của chỉ số tình trạng thực vật dựa trên dữ liệu MODIS EVI, phản ánh sức khỏe cây trồng và mức độ hạn hán sinh học
- ✅ **Chỉ số bất thường lượng mưa (Precipitation Anomaly Index)**: Đánh giá mức độ thiếu hụt hoặc dư thừa lượng mưa so với giá trị trung bình lịch sử từ dữ liệu ERA5-Land
- ✅ **Xử lý dữ liệu MODIS**: Kết hợp dữ liệu từ Terra và Aqua, lọc mây, chuẩn hóa dữ liệu và tổng hợp theo tháng với Google Earth Engine
- ✅ **Xử lý dữ liệu ERA5-Land**: Truy xuất và tính toán các chỉ số bất thường từ dữ liệu khí tượng tái phân tích toàn cầu
- ✅ **Trực quan hóa chỉ số hạn hán**: Sử dụng geemap để hiển thị các chỉ số hạn hán trên bản đồ tương tác với bảng màu phù hợp

### Kỹ năng bạn có thể áp dụng:
- Xây dựng hệ thống giám sát hạn hán tự động sử dụng dữ liệu viễn thám từ Google Earth Engine
- Tính toán và phân tích các chỉ số hạn hán khác nhau để đánh giá toàn diện tình trạng hạn hán
- Kết hợp nhiều nguồn dữ liệu (MODIS, ERA5-Land) để có cái nhìn đa chiều về hiện tượng hạn hán
- Tạo cảnh báo sớm hạn hán cho nông nghiệp và quản lý tài nguyên nước
- Phân tích xu hướng và biến động hạn hán theo thời gian phục vụ nghiên cứu biến đổi khí hậu
- Mở rộng phương pháp cho các chỉ số hạn hán khác như TCI (Temperature Condition Index), VHI (Vegetation Health Index) hoặc SPI (Standardized Precipitation Index)
