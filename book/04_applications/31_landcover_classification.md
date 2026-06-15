# Bài 31: Phân loại đất sử dụng ảnh Sentinel-2 và máy học

Trong các bài toán viễn thám hiện nay, phân loại lớp phủ đất là một trong những hướng nghiên cứu quan trọng, cho phép xác định các loại bề mặt lớp phủ khác nhau như đất rừng, đất nông nghiệp, đất đô thị hay mặt nước từ dữ liệu ảnh vệ tinh. Với sự phát triển của máy học, thay vì xây dựng các ngưỡng thủ công, ta có thể huấn luyện mô hình dựa trên dữ liệu phổ để tự động học mối quan hệ giữa đặc trưng ảnh và các lớp đất.

## 31.1. Mục tiêu bài học

- Chuẩn bị dữ liệu cho huấn luyện mô hình máy học random forest
- Xác định số kênh và loại đất phân loại
- Sử dụng mô hình Random forest để huấn luyện bài toán phân loại đất
- Sử dụng mô hình đã huấn luyện để dự đoán trên ảnh mới.


```python
import ee 
import rioxarray as rxr
ee.Authenticate()
ee.Initialize()
```

## 31.2. Chuẩn bị dữ liệu

Trong ví dụ này, ta sử dụng GEE để tải dữ liệu ảnh vệ tinh Sentinel-2 trong giai đoạn 2025-06-01 đến 2025-09-30 cho AOI (khu vực nghiên cứu), lựa chọn các kênh phổ quan trọng gồm B2, B3, B4, B5, B6, B7 và B8 để đại diện cho thông tin màu sắc và cấu trúc thực vật. Để giảm nhiễu do mây (giới hạn 5% mây) và biến động ngắn hạn, giá trị median theo thời gian được sử dụng nhằm tạo ra một ảnh đại diện ổn định cho toàn bộ giai đoạn nghiên cứu.

Từ tập hợp đặc trưng phổ này, ta xây dựng bộ dữ liệu huấn luyện để mô hình học máy nhận biết các loại lớp phủ đất với 5 loại đất chính (1: Nước, 2: Rừng, 3: Nông nghiệp, 4: Đô thị/đường, 5: Cây bụi). Sau khi huấn luyện, mô hình được áp dụng để dự đoán bản đồ phân loại cho toàn bộ vùng nghiên cứu, từ đó hỗ trợ phân tích hiện trạng và biến động sử dụng đất trong khu vực.

### 31.2.1. Xác định vùng nghiên cứu và ảnh Sentinel-2

Trong ví dụ này, ta xác định vùng nghiên cứu là một vùng nhỏ ở phía Nam nước Đức như bên dưới, nhưng phương pháp và quy trình thực hiện hoàn toàn có thể mở rộng và áp dụng cho bất kỳ khu vực nghiên cứu nào khác.


```python
# Bounding box cho vùng nghiên cứu ở Đức
bbox = [9.84375   , 47.5172007 , 10.1953125 , 47.75409798]
roi = ee.Geometry.Rectangle(bbox)
```


```python
sen2col = ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED").filterBounds(roi).filterDate("2025-06-01", "2025-09-30").filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 5)).select(["B2", "B3", "B4", "B5", "B6", "B7", "B8"])
# Tính giá trị median cho mỗi pixel trong khoảng thời gian đã lọc
sen2_median = sen2col.median().multiply(0.0001) # Sentinel-2 Surface Reflectance cần nhân với 0.0001 để chuyển về giá trị phản xạ thực tế
```

### 31.2.2. Tải dữ liệu Sentinel-2 cho vùng nghiên cứu

Dữ liệu cho bài này được chuẩn bị trong folder `data/sen2median_2025.tif`. Nếu bạn muốn thực hành việc tải dữ liệu, bạn có thể làm như bên dưới và tải dữ liệu về Google Drive của mình. Giờ bạn có dữ liệu, bạn cho vào QGIS và vẽ polygons cho các loại đất mong muốn. Sau đó trích xuất giá trị spectral bands cho các polygons loại đất tương ứng. Dữ liệu sau khi trích xuất sẽ là một bảng thể hiện các bands và loại đất.


```python
task = ee.batch.Export.image.toDrive(
    image=sen2_median.toFloat(), # ảnh cần tải về
    description='Sentinel2_Image_Export',
    folder='GEE_Exports',
    fileNamePrefix='sen2median_2025',
    region=roi, # vùng nghiên cứu
    scale=10, # độ phân giải (10m cho Sentinel-2)
    crs='EPSG:32632', # hệ tọa độ mong muốn
    maxPixels=1e9 # giới hạn số pixel để tránh lỗi khi xuất ảnh lớn
)
# task.start() # Bỏ comment dòng này để bắt đầu quá trình xuất ảnh
# Sau khi chạy task.start(), bạn có thể theo dõi tiến trình xuất ảnh trong Google Drive của mình. Khi hoàn thành, ảnh sẽ được lưu trong thư mục 'GEE_Exports' với tên 'sen2median_2025.tif'.
```

# 31.3. Xây dựng mô hình


```python
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, confusion_matrix
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
```

### 31.3.1. Huấn luyện mô hình


```python

```

### 31.3.2. Đánh giá mô hình


```python

```

## 31.4. Sử dụng mô hình dự đoán


```python

```
