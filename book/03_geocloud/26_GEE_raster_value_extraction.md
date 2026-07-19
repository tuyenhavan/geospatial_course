# Bài 26: Trích xuất giá trị ảnh theo vị trí (GEE)

Trong bài học này, chúng ta sẽ học cách trích xuất giá trị raster theo theo vị trí điểm hoặc polygons sử dụng GEE Python API.

> **Lưu ý**: Bạn có thể chạy trực tiếp notebook bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/1aXJYL8YYIPwG7tUus-xwzdt8xDyh9bS7) mà không cần cài đặt Python. 

## 26.1. Mục tiêu học tập

Sau bài này bạn có thể:

- Trích xuất giá trị ảnh cho một hoặc nhiều điểm
- Trích xuất giá trị cho các vùng khác nhau
- Chuyển kết quả về pandas DataFrame và export ra file excel.


```python
import ee
import geemap
import geopandas as gpd
import pandas as pd

ee.Authenticate()
ee.Initialize(project='geocourse-501706')
```

## 26.2. Chuẩn bị dữ liệu Sentinel-2

Trong mục này, chúng ta sẽ trích xuất giá trị ảnh Sentinel-2 dựa trên dữ liệu điểm và polygons. Trước tiên, chúng ta xác định khu vực nghiên cứu và thời gian. Trong ví dụ này, ta lấy bản đồ của Singapore và sau đó tạo ra các điểm cho mỗi huyện hay địa điểm và bộ dữ liệu Sentinel-2 như bên dưới.


```python
# Ví dụ vùng nghiên cứu là Singapore 
roi = gpd.read_file('https://geodata.ucdavis.edu/gadm/gadm4.1/json/gadm41_SGP_1.json')
# Tạo thêm points vs crs=4326 
points = gpd.GeoDataFrame(geometry=[i.geometry.centroid for i in roi.itertuples()], crs='EPSG:4326')
start_date = '2023-06-01'
end_date = '2023-09-30'
# Chuyển đổi roi sang định dạng ee.Geometry
roi = geemap.geopandas_to_ee(roi) # Chuyển đổi GeoDataFrame sang ee.FeatureCollection
points = geemap.geopandas_to_ee(points) # Chuyển đổi GeoDataFrame sang ee.FeatureCollection

sen2col = (ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
          .filterBounds(roi)
          .filterDate(start_date, end_date)
          .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 25))
            .select(['B2', 'B3', 'B4', 'B8'])
          )
print('Số lượng ảnh Sentinel-2:', sen2col.size().getInfo())
```


```python
# Format giá trị trích xuất thành DataFrame
def format_values(values):
    features = values['features']
    data = []
    for feature in features:
        properties = feature['properties']
        data.append(properties)
    return pd.DataFrame(data)
```

## 26.3. Trích xuất giá trị ảnh với dữ liệu điểm

### 26.3.1. Trích xuất giá trị với một ảnh

Trong ví dụ này, chúng ta đã trích xuất giá trị pixel từ một ảnh Sentinel-2 tại các điểm vùng nghiên cứu Singapore. Kết quả được lưu trong một DataFrame của pandas, trong đó mỗi hàng tương ứng với một điểm và các cột chứa giá trị pixel của các band đã chọn (B2, B3, B4, B8). 


```python
# Chọn một ảnh Sentinel-2 đầu tiên để trích xuất giá trị
image = sen2col.first()
# Trích xuất giá trị pixel tại các điểm
values = image.sampleRegions(
            collection=ee.FeatureCollection(points), scale=10, geometries=True
        ).getInfo()
df = format_values(values)
df.head()
```

### 26.3.2. Trích xuất giá trị ảnh theo thời gian

Tương tự như vậy, chúng ta có thể trích xuất giá trị pixel từ ảnh Sentinel-2 tại các vị trí theo thời gian của ảnh. Kết quả được lưu trong một DataFrame của pandas, trong đó mỗi hàng tương ứng với một điểm và các cột chứa giá trị pixel của các band đã chọn (B2, B3, B4, B8). Cột "time" chứa thông tin về thời gian của ảnh, được chuyển đổi từ định dạng timestamp của GEE sang định dạng datetime của Python.


```python
from datetime import datetime
values = sen2col.getRegion(geometry=points, scale=10).getInfo()
df = pd.DataFrame(values[1:], columns=values[0])
df["time"] = [
        datetime.fromtimestamp(timestamp_ms / 1000) for timestamp_ms in df["time"]
    ]
df.head()
```

## 26.4. Trích xuất giá trị ảnh với đối tượng đa giác

### 26.4.1. Trích xuất giá trị với một ảnh 

Trong ví dụ này, chúng ta trích dẫn giá trị pixel ảnh Sentinel-2 cho các polygons trong vùng nghiên cứu. Kết quả được lưu trong một DataFrame của pandas, trong đó mỗi hàng tương ứng với một polygon và các cột chứa giá trị pixel trung bình của các band đã chọn (B2, B3, B4, B8) cùng với thông tin thuộc tính từ GeoDataFrame


```python
# Trích xuất giá trị trung bình của ảnh đầu tiên cho toàn bộ roi
results = sen2col.first().reduceRegions(
            collection=roi, reducer=ee.Reducer.mean(), scale=10
        ).getInfo()
df = format_values(results)
df.head()
```

### 26.4.2. Trích xuất giá trị ảnh theo thời gian

Tương tự, chúng ta cũng có thể trích xuất giá trị raster theo polygons và theo thời gian. Kết quả lưu ra là một DataFrame với thông tin giá trị được trích xuất và cột thời gian.


```python
def extract_values(image):
        # Extract the date from the image
        date = image.date().format("YYYY-MM-dd")
        # Reduce the image by the polygons
        stats = image.reduceRegions(
            collection=roi, reducer=ee.Reducer.mean(), scale=10
        ).map(lambda feature: feature.set("date", date))
        return stats
# Áp dụng hàm extract_values cho toàn bộ ImageCollection
results = sen2col.map(extract_values).flatten().getInfo()
df = format_values(results)
df.head() # Bạn có thể lọc bỏ những cột không cần thiết nếu muốn.
```

## Tóm tắt

Bạn đã hoàn thành Bài 26 và nắm vững kỹ thuật **trích xuất giá trị ảnh theo vị trí** - kỹ năng cốt lõi để kết nối dữ liệu viễn thám với dữ liệu thực địa trên Google Earth Engine.

### Các khái niệm chính đã nắm vững:
- ✅ Trích xuất giá trị pixel tại **dữ liệu điểm** với một ảnh đơn lẻ bằng `sampleRegions()`
- ✅ Trích xuất giá trị theo **chuỗi thời gian** tại nhiều điểm bằng `getRegion()`
- ✅ Tính thống kê vùng (mean, ...) theo **đa giác** với `reduceRegions()`
- ✅ Trích xuất giá trị theo thời gian cho **đa giác** bằng cách kết hợp `map()` + `reduceRegions()` + `flatten()`
- ✅ Chuyển kết quả GEE về **pandas DataFrame** để phân tích và xuất file

### Kỹ năng bạn có thể áp dụng:
- Trích xuất giá trị phổ Sentinel-2 tại các điểm khảo sát thực địa để xây dựng tập dữ liệu huấn luyện
- Tính toán thống kê theo vùng (trung bình, tổng, độ lệch chuẩn) cho nhiều polygon cùng lúc
- Xây dựng time-series giá trị phổ cho từng vùng địa lý phục vụ phân tích xu hướng
- Kết hợp dữ liệu điểm từ GPS với ảnh vệ tinh để kiểm chứng kết quả phân loại
- Xuất kết quả ra Excel/CSV để chia sẻ và phân tích ngoài GEE

