# Bài 25: Trích xuất giá trị ảnh theo vị trí (GEE)

## 25.1. Mục tiêu học tập

Sau bài này bạn có thể:

- Trích xuất giá trị ảnh cho một hoặc nhiều điểm
- Trích xuất giá trị cho các vùng khác nhau
- Chuyển kết quả về pandas DataFrame và export ra file excel.


```python
import ee
import geemap
import geopandas as gpd
import pandas as pd

ee.Initialize()
```

## 25.2. Chuẩn bị dữ liệu Sentinel-2

Trong mục này, chúng ta sẽ trích xuất giá trị ảnh Sentinel-2 dựa trên dữ liệu điểm và polygons.


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



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>



    Số lượng ảnh Sentinel-2: 2
    


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



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>



## 25.3. Trích xuất giá trị ảnh với dữ liệu điểm

### 25.3.1. Trích xuất giá trị với một ảnh


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



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>






<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>B2</th>
      <th>B3</th>
      <th>B4</th>
      <th>B8</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>794</td>
      <td>872</td>
      <td>848</td>
      <td>1160</td>
    </tr>
    <tr>
      <th>1</th>
      <td>833</td>
      <td>928</td>
      <td>730</td>
      <td>3280</td>
    </tr>
    <tr>
      <th>2</th>
      <td>231</td>
      <td>450</td>
      <td>201</td>
      <td>3086</td>
    </tr>
    <tr>
      <th>3</th>
      <td>612</td>
      <td>936</td>
      <td>800</td>
      <td>3176</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1310</td>
      <td>1518</td>
      <td>1620</td>
      <td>2082</td>
    </tr>
  </tbody>
</table>
</div>



### 25.3.2. Trích xuất giá trị ảnh theo thời gian


```python
from datetime import datetime
values = sen2col.getRegion(geometry=points, scale=10).getInfo()
df = pd.DataFrame(values[1:], columns=values[0])
df["time"] = [
        datetime.fromtimestamp(timestamp_ms / 1000) for timestamp_ms in df["time"]
    ]
df.head()
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>






<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>longitude</th>
      <th>latitude</th>
      <th>time</th>
      <th>B2</th>
      <th>B3</th>
      <th>B4</th>
      <th>B8</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>20230724T031529_20230724T033402_T48NUG</td>
      <td>103.792920</td>
      <td>1.415071</td>
      <td>2023-07-24 05:37:53.217</td>
      <td>231</td>
      <td>450</td>
      <td>201</td>
      <td>3086</td>
    </tr>
    <tr>
      <th>1</th>
      <td>20230927T031521_20230927T033258_T48NUG</td>
      <td>103.792920</td>
      <td>1.415071</td>
      <td>2023-09-27 05:37:49.482</td>
      <td>209</td>
      <td>428</td>
      <td>213</td>
      <td>3220</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20230724T031529_20230724T033402_T48NUG</td>
      <td>103.703538</td>
      <td>1.330180</td>
      <td>2023-07-24 05:37:53.217</td>
      <td>963</td>
      <td>1194</td>
      <td>1230</td>
      <td>2286</td>
    </tr>
    <tr>
      <th>3</th>
      <td>20230927T031521_20230927T033258_T48NUG</td>
      <td>103.703538</td>
      <td>1.330180</td>
      <td>2023-09-27 05:37:49.482</td>
      <td>1562</td>
      <td>1770</td>
      <td>1852</td>
      <td>2684</td>
    </tr>
    <tr>
      <th>4</th>
      <td>20230724T031529_20230724T033402_T48NUG</td>
      <td>103.831907</td>
      <td>1.306106</td>
      <td>2023-07-24 05:37:53.217</td>
      <td>794</td>
      <td>872</td>
      <td>848</td>
      <td>1160</td>
    </tr>
  </tbody>
</table>
</div>



## 25.4. Trích xuất giá trị ảnh với đối tượng đa giác

### 25.4.1. Trích xuất giá trị với một ảnh 


```python
# Trích xuất giá trị trung bình của ảnh đầu tiên cho toàn bộ roi
results = sen2col.first().reduceRegions(
            collection=roi, reducer=ee.Reducer.mean(), scale=10
        ).getInfo()
df = format_values(results)
df.head()
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>






<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>B2</th>
      <th>B3</th>
      <th>B4</th>
      <th>B8</th>
      <th>CC_1</th>
      <th>COUNTRY</th>
      <th>ENGTYPE_1</th>
      <th>GID_0</th>
      <th>GID_1</th>
      <th>HASC_1</th>
      <th>ISO_1</th>
      <th>NAME_1</th>
      <th>NL_NAME_1</th>
      <th>TYPE_1</th>
      <th>VARNAME_1</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892.041324</td>
      <td>1032.100549</td>
      <td>1018.428051</td>
      <td>2217.619864</td>
      <td>NA</td>
      <td>Singapore</td>
      <td>Region</td>
      <td>SGP</td>
      <td>SGP.1_1</td>
      <td>NA</td>
      <td>NA</td>
      <td>Central</td>
      <td>NA</td>
      <td>Region</td>
      <td>NA</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1007.188920</td>
      <td>1191.552757</td>
      <td>1174.162151</td>
      <td>2502.056017</td>
      <td>NA</td>
      <td>Singapore</td>
      <td>Region</td>
      <td>SGP</td>
      <td>SGP.2_1</td>
      <td>NA</td>
      <td>NA</td>
      <td>East</td>
      <td>NA</td>
      <td>Region</td>
      <td>NA</td>
    </tr>
    <tr>
      <th>2</th>
      <td>644.554584</td>
      <td>828.279536</td>
      <td>724.837611</td>
      <td>2520.811121</td>
      <td>NA</td>
      <td>Singapore</td>
      <td>Region</td>
      <td>SGP</td>
      <td>SGP.3_1</td>
      <td>NA</td>
      <td>NA</td>
      <td>North</td>
      <td>NA</td>
      <td>Region</td>
      <td>NA</td>
    </tr>
    <tr>
      <th>3</th>
      <td>695.144399</td>
      <td>859.685397</td>
      <td>789.526614</td>
      <td>2484.610876</td>
      <td>NA</td>
      <td>Singapore</td>
      <td>Region</td>
      <td>SGP</td>
      <td>SGP.4_1</td>
      <td>NA</td>
      <td>NA</td>
      <td>North-East</td>
      <td>NA</td>
      <td>Region</td>
      <td>NA</td>
    </tr>
    <tr>
      <th>4</th>
      <td>856.337325</td>
      <td>1033.909113</td>
      <td>1002.770418</td>
      <td>2360.996355</td>
      <td>NA</td>
      <td>Singapore</td>
      <td>Region</td>
      <td>SGP</td>
      <td>SGP.5_1</td>
      <td>NA</td>
      <td>NA</td>
      <td>West</td>
      <td>NA</td>
      <td>Region</td>
      <td>NA</td>
    </tr>
  </tbody>
</table>
</div>



### 25.4.2. Trích xuất giá trị ảnh theo thời gian


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



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>






<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>B2</th>
      <th>B3</th>
      <th>B4</th>
      <th>B8</th>
      <th>CC_1</th>
      <th>COUNTRY</th>
      <th>ENGTYPE_1</th>
      <th>GID_0</th>
      <th>GID_1</th>
      <th>HASC_1</th>
      <th>ISO_1</th>
      <th>NAME_1</th>
      <th>NL_NAME_1</th>
      <th>TYPE_1</th>
      <th>VARNAME_1</th>
      <th>date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892.041324</td>
      <td>1032.100549</td>
      <td>1018.428051</td>
      <td>2217.619864</td>
      <td>NA</td>
      <td>Singapore</td>
      <td>Region</td>
      <td>SGP</td>
      <td>SGP.1_1</td>
      <td>NA</td>
      <td>NA</td>
      <td>Central</td>
      <td>NA</td>
      <td>Region</td>
      <td>NA</td>
      <td>2023-07-24</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1007.188920</td>
      <td>1191.552757</td>
      <td>1174.162151</td>
      <td>2502.056017</td>
      <td>NA</td>
      <td>Singapore</td>
      <td>Region</td>
      <td>SGP</td>
      <td>SGP.2_1</td>
      <td>NA</td>
      <td>NA</td>
      <td>East</td>
      <td>NA</td>
      <td>Region</td>
      <td>NA</td>
      <td>2023-07-24</td>
    </tr>
    <tr>
      <th>2</th>
      <td>644.554584</td>
      <td>828.279536</td>
      <td>724.837611</td>
      <td>2520.811121</td>
      <td>NA</td>
      <td>Singapore</td>
      <td>Region</td>
      <td>SGP</td>
      <td>SGP.3_1</td>
      <td>NA</td>
      <td>NA</td>
      <td>North</td>
      <td>NA</td>
      <td>Region</td>
      <td>NA</td>
      <td>2023-07-24</td>
    </tr>
    <tr>
      <th>3</th>
      <td>695.144399</td>
      <td>859.685397</td>
      <td>789.526614</td>
      <td>2484.610876</td>
      <td>NA</td>
      <td>Singapore</td>
      <td>Region</td>
      <td>SGP</td>
      <td>SGP.4_1</td>
      <td>NA</td>
      <td>NA</td>
      <td>North-East</td>
      <td>NA</td>
      <td>Region</td>
      <td>NA</td>
      <td>2023-07-24</td>
    </tr>
    <tr>
      <th>4</th>
      <td>856.337325</td>
      <td>1033.909113</td>
      <td>1002.770418</td>
      <td>2360.996355</td>
      <td>NA</td>
      <td>Singapore</td>
      <td>Region</td>
      <td>SGP</td>
      <td>SGP.5_1</td>
      <td>NA</td>
      <td>NA</td>
      <td>West</td>
      <td>NA</td>
      <td>Region</td>
      <td>NA</td>
      <td>2023-07-24</td>
    </tr>
  </tbody>
</table>
</div>



## Tóm tắt

Bạn đã hoàn thành Bài 25 và nắm vững kỹ thuật **trích xuất giá trị ảnh theo vị trí** - kỹ năng cốt lõi để kết nối dữ liệu viễn thám với dữ liệu thực địa trên Google Earth Engine.

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

