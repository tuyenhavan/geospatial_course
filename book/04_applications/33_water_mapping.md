# Bài 33: Theo dõi nước mặt sử dụng ảnh Sentinel-1 và Sentinel-2

Nước mặt (bao gồm sông, hồ, đầm, và các vùng ngập nước khác) đóng vai trò quan trọng trong hệ sinh thái, quản lý tài nguyên nước và phát triển kinh tế - xã hội. Việc theo dõi và lập bản đồ nước mặt một cách chính xác là cần thiết cho nhiều ứng dụng như quản lý tài nguyên nước, giám sát biến đổi khí hậu, quy hoạch đô thị và nông nghiệp. Công nghệ viễn thám vệ tinh, với khả năng quan sát liên tục và bao phủ diện rộng, đã chứng minh là công cụ hiệu quả trong giám sát nước mặt.

Trong bài học này, chúng ta sẽ sử dụng hai loại dữ liệu vệ tinh bổ trợ cho nhau: ảnh Sentinel-2 với khả năng quang học phát hiện nước qua chỉ số NDWI (Normalized Difference Water Index), và ảnh radar Sentinel-1 với ưu thế xuyên thấu qua mây và hoạt động cả ngày lẫn đêm. Sự kết hợp này cung cấp giải pháp toàn diện để lập bản đồ nước mặt, đặc biệt hiệu quả trong điều kiện thời tiết xấu khi mây che phủ thường xuyên.

> **Lưu ý**
> >
> Bạn có thể chạy trực tiếp notebook này bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/16UsUWaeWzraRVBDj1WhaxRYZ6GmfL4Pl?authuser=3) mà không cần cài đặt. Để tránh làm thay đổi nội dung gốc và thuận tiện cho việc lưu kết quả, hãy tạo một bản sao ( File → Save a copy in Drive ) trước khi chạy và chỉnh sửa mã nguồn trong notebook.

## 33.1. Mục tiêu bài học

Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Hiểu nguyên lý phát hiện mặt nước từ ảnh quang học và ảnh radar
- Tải và xử lý dữ liệu Sentinel-2 từ Google Earth Engine
- Tính toán chỉ số NDWI (Normalized Difference Water Index) để xác định vùng nước
- Tải và xử lý dữ liệu Sentinel-1 SAR cho lập bản đồ nước mặt
- Áp dụng phương pháp ngưỡng (thresholding) để trích xuất vùng nước
- So sánh và đánh giá ưu nhược điểm của hai phương pháp
- Trực quan hóa kết quả phát hiện nước mặt trên bản đồ tương tác


```python
import ee 
import geemap # Bạn có thể cài đặt geemap bằng pip install geemap
import geesat # Bạn có thể cài đặt geesat bằng pip install git+https://github.com/tuyenhavan/geesat.git
from geesat import geogee, geosen
ee.Authenticate()
ee.Initialize(project='geocourse-501706')
```

## 33.2. Xác định khu vực nghiên cứu


```python
# Định nghĩa bounding box (bbox) cho khu vực quan tâm. Trong ví dụ này, bbox là khu vực hồ Đá Bàn
bbox = [
    109.07678308345344,
    12.63164734584147,
    109.12381830073859,
    12.676032712162309
]
# Chuyển đổi bbox thành geometry để sử dụng trong các thao tác với Google Earth Engine
geometry = ee.Geometry.Rectangle(bbox)
```

Trong bài học này, chúng ta sẽ sử dụng khu vực hồ Đá Bàn (Gia Lai) làm ví dụ minh họa. Đây là khu vực có mặt nước rõ ràng, phù hợp để thực hành các kỹ thuật phát hiện nước từ ảnh vệ tinh. Các phương pháp được trình bày có thể áp dụng cho bất kỳ khu vực nào để theo dõi nước mặt hoặc giám sát biến đổi mặt nước theo thời gian.

## 33.3. Tính toán chỉ số nước từ ảnh Sentinel-2

Phương pháp đầu tiên chúng ta sẽ khám phá là sử dụng ảnh quang học Sentinel-2 với chỉ số NDWI. Đây là phương pháp phổ biến và dễ thực hiện, tận dụng sự khác biệt về phản xạ phổ giữa mặt nước và các bề mặt khác. Mặt nước có đặc tính hấp thụ mạnh ở vùng cận hồng ngoại (NIR) và phản xạ tương đối cao ở vùng xanh lá (Green), giúp phân biệt rõ ràng với thực vật và đất trống.

Tuy nhiên, phương pháp này có hạn chế lớn là phụ thuộc vào điều kiện thời tiết. Khi có mây che phủ - điều thường xảy ra trong mùa mưa - ảnh quang học không thể xuyên qua để quan sát mặt đất. Đây là lý do chúng ta cần bổ sung thêm dữ liệu radar Sentinel-1 ở phần sau.

### 33.3.1. Tính toán chỉ số NDWI

**Chỉ số NDWI (Normalized Difference Water Index)** là chỉ số phổ biến nhất để xác định mặt nước từ ảnh vệ tinh quang học. Chỉ số này dựa trên nguyên lý mặt nước có phản xạ cao ở dải sóng xanh lá (Green) và hấp thụ mạnh ở dải cận hồng ngoại (NIR). NDWI được tính theo công thức:

$$
NDWI = \frac{Green - NIR}{Green + NIR}
$$

Trong đó:
- $Green$ là giá trị phản xạ tại dải sóng xanh lá (band B3 của Sentinel-2, 560 nm)
- $NIR$ là giá trị phản xạ tại dải cận hồng ngoại (band B8 của Sentinel-2, 842 nm)

Giá trị NDWI dao động từ -1 đến 1. Mặt nước thường có giá trị NDWI dương (thường > 0), trong khi thực vật và đất có giá trị âm. Ngưỡng phổ biến để phân tách nước là NDWI > 0 hoặc > 0.1, tuy nhiên giá trị này có thể điều chỉnh tùy theo điều kiện cụ thể của khu vực nghiên cứu.


```python
# Đọc dữ liệu Sentinel-2 từ Google Earth Engine, lọc theo khu vực quan tâm, khoảng thời gian và tỷ lệ mây dưới 5%, sau đó tính toán giá trị median của các ảnh trong bộ sưu tập
sen2col = ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED").filterBounds(geometry).filterDate('2025-03-01', '2026-03-28').filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 5)).median()
# Tính toán chỉ số NDWI (Normalized Difference Water Index) để xác định vùng nước
ndwi = sen2col.multiply(0.0001).normalizedDifference(['B3', 'B8']).rename('NDWI').clip(geometry)
```

### 33.3.2. Xác định vùng nước dựa vào chỉ số NDWI

Sau khi tính toán chỉ số NDWI, bước tiếp theo là áp dụng ngưỡng (threshold) để tách vùng nước khỏi các bề mặt khác. Trong ví dụ này, chúng ta sử dụng ngưỡng NDWI > 0.1 để tạo mặt nạ nhị phân (binary mask), trong đó pixel có giá trị 1 đại diện cho nước và 0 đại diện cho không phải nước.


```python
water_mask = ndwi.gt(0.1)  # Tạo mặt nạ nước bằng cách lọc các giá trị NDWI lớn hơn 0.1
water = water_mask.updateMask(water_mask)  # Cập nhật mặt nạ nước để chỉ hiển thị các pixel nước

Map = geemap.Map()
Map.centerObject(geometry, 12)
Map.addLayer(water, {'palette': ['blue']}, 'Water Mask')
Map
```

## 33.4. Tính toán chỉ số nước từ ảnh Sentinel-1

Trong khi ảnh quang học gặp khó khăn với mây che phủ, **ảnh radar vệ tinh (SAR - Synthetic Aperture Radar)** như Sentinel-1 có khả năng xuyên qua mây và hoạt động cả ngày lẫn đêm. Điều này làm cho SAR trở thành công cụ lý tưởng cho giám sát nước mặt liên tục, đặc biệt trong điều kiện thời tiết xấu.

Nguyên lý phát hiện nước từ ảnh SAR dựa trên đặc tính phản xạ gương (specular reflection) của mặt nước. Khi sóng radar chiếu xuống mặt nước phẳng, nó bị phản xạ theo hướng khác với ăng-ten thu, dẫn đến tín hiệu backscatter rất thấp (giá trị âm tính theo đơn vị dB). Ngược lại, thực vật và các bề mặt nhám tán xạ ngược lại nhiều năng lượng về ăng-ten, cho giá trị backscatter cao hơn.

Trong ví dụ này, các hàm trong gói `geesat` được sử dụng để thực hiện quy trình tiền xử lý dữ liệu Sentinel-1. Quy trình bao gồm hiệu chỉnh địa hình (terrain correction) và loại bỏ nhiễu đốm (speckle noise), giúp cải thiện chất lượng ảnh và tăng độ tin cậy của các kết quả phân tích.

### 33.4.1. Chuẩn bị dữ liệu Sentinel-1


```python
sen1col = geosen.prepare_sentinel1_collection(
    roi=geometry,
    orbit_pass='DESCENDING',
    start_date='2025-03-01',
    end_date='2026-03-28',
).select(['VV']).median().clip(geometry)
```

### 33.4.2. Xác định vùng nước dựa vào ngưỡng giá trị

Tương tự như phương pháp với Sentinel-2, chúng ta áp dụng ngưỡng để tách vùng nước. Với dữ liệu Sentinel-1 VV, mặt nước thường có giá trị backscatter dưới -15 dB. Giá trị ngưỡng này có thể thay đổi tùy thuộc vào điều kiện gió (làm gợn sóng mặt nước, tăng backscatter) và loại mặt nước (nước tĩnh vs nước chảy).

Ưu điểm của phương pháp SAR là khả năng hoạt động trong mọi điều kiện thời tiết, tuy nhiên nó cũng có thể bị nhầm lẫn với các bề mặt phẳng khác như đường nhựa hoặc sân bay. Do đó, trong thực tế thường kết hợp cả hai phương pháp để đạt kết quả chính xác nhất.


```python
water_mask = sen1col.lt(-15)  # Tạo mặt nạ nước bằng cách lọc các giá trị VV nhỏ hơn -15 dB
water = water_mask.updateMask(water_mask)  # Cập nhật mặt nạ nước để chỉ hiển thị các pixel nước
# Hiển thị bản đồ với lớp Sentinel-1 VV và lớp mặt nạ nước
Map = geemap.Map()
Map.centerObject(geometry, 12)
Map.addLayer(sen1col, {'min': -20, 'max': 0}, 'Sentinel-1 VV')
Map.addLayer(water, {'palette': ['blue']}, 'Water')
Map
```

## Tóm tắt

Bạn đã hoàn thành Bài 33 và học được cách sử dụng hai công nghệ vệ tinh bổ trợ, Sentinel-2 quang học và Sentinel-1 radar, để phát hiện và theo dõi nước mặt. Đây là kỹ năng thiết yếu trong ứng dụng viễn thám phục vụ quản lý tài nguyên nước, giám sát biến đổi khí hậu, quy hoạch và phát triển bền vững.

### Các khái niệm chính đã nắm vững:
- ✅ **Chỉ số NDWI (Normalized Difference Water Index)**: Tính toán và áp dụng chỉ số phổ từ ảnh Sentinel-2 để phân tách mặt nước dựa trên phản xạ phổ
- ✅ **Phát hiện nước từ SAR**: Sử dụng đặc tính backscatter thấp của mặt nước trong ảnh radar Sentinel-1 để lập bản đồ nước mặt không phụ thuộc thời tiết
- ✅ **Xử lý ảnh Sentinel-2**: Lọc mây, tổng hợp ảnh và tính toán các chỉ số quang phổ với Google Earth Engine
- ✅ **Xử lý ảnh Sentinel-1**: Tiền xử lý dữ liệu SAR, lọc nhiễu speckle và trích xuất thông tin mặt nước từ phân cực VV
- ✅ **Phương pháp ngưỡng (Thresholding)**: Áp dụng ngưỡng phù hợp để tạo mặt nạ nhị phân phân tách vùng nước và không phải nước
- ✅ **So sánh hai phương pháp**: Hiểu được ưu nhược điểm của ảnh quang học (phụ thuộc thời tiết) và ảnh radar (độc lập thời tiết) trong ứng dụng giám sát nước mặt
- ✅ **Trực quan hóa kết quả**: Sử dụng geemap để hiển thị các lớp dữ liệu và kết quả phân tích trên bản đồ tương tác

### Điểm khác biệt chính giữa hai phương pháp:

| Đặc điểm | Sentinel-2 (NDWI) | Sentinel-1 (SAR) |
|----------|-------------------|------------------|
| **Độc lập thời tiết** | ❌ Bị ảnh hưởng bởi mây | ✅ Hoạt động qua mây |
| **Thời gian quan sát** | ☀️ Chỉ ban ngày | 🌙 Cả ngày và đêm |
| **Độ phân giải không gian** | 10m | 10m |
| **Độ chính xác** | Cao khi trời quang | Tốt, có thể nhầm với bề mặt phẳng |
| **Dễ xử lý** | ✅ Đơn giản, trực quan | ⚠️ Phức tạp hơn, cần tiền xử lý |
| **Ứng dụng lý tưởng** | Giám sát thường xuyên, điều kiện tốt | Giám sát liên tục, mọi điều kiện thời tiết |
