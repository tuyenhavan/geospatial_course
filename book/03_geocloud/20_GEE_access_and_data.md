# Bài 20: Truy cập và Lọc dữ liệu trên Google Earth Engine

Google Earth Engine (GEE) là một trong những nền tảng điện toán đám mây mạnh mẽ nhất thế giới dành cho phân tích dữ liệu địa không gian, cung cấp quyền truy cập vào hàng nghìn bộ dữ liệu vệ tinh, khí hậu và địa lý, miễn phí cho nghiên cứu và giáo dục theo một cách thống nhất. Trước khi học bày này, bạn cần phải đăng kí tài khoản GEE nếu chưa có. Bạn có thể tham khảo hướng dẫn đăng kí theo video [link](https://www.youtube.com/watch?v=O9iyjs4w-8I) này.

## 20.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

- Xác thực và khởi tạo GEE Python API
- Hiểu 5 đối tượng cơ bản của GEE: `ee.Image`, `ee.ImageCollection`, `ee.Feature`, `ee.FeatureCollection`
- Lọc `ImageCollection` theo thời gian, không gian và thuộc tính metadata
- Kết hợp nhiều bộ lọc để lấy đúng dữ liệu cần thiết
- Tra cứu và khám phá các tập dữ liệu phổ biến trong GEE Data Catalog


```python
import ee # chưa có thì cài đặt bằng pip install earthengine-api
import geopandas as gpd # chưa có thì cài đặt bằng pip install geopandas
from shapely.geometry import mapping
ee.Authenticate()
ee.Initialize()
```

## 20.2. Các đối tượng (Objects) chính trong GEE

Khi làm việc với GEE, các đối tượng trong GEE đều được xử lý theo cơ chế server-side. Vì vậy, chúng ta cần sử dụng các hàm và phương thức có sẵn của GEE như `map()`, `filter()`, `reduce()`, thay vì thao tác trực tiếp bằng Python như với dữ liệu local (client-side). Chỉ khi dùng`getInfo()` thì dữ liệu mới được tải từ server về client.

Tổng quan dữ liệu trên GEE có thể tham khảo tại [đây](https://developers.google.com/earth-engine/datasets). Nếu bạn chưa có tài khoản, bạn có thể đăng kí theo hướng dẫn tại video [link](https://www.youtube.com/watch?v=O9iyjs4w-8I) này.

### 20.2.1. Tổng quan 4 đối tượng cơ bản

| Đối tượng | Mô tả | Ví dụ thực tế |
|---|---|---|
| `ee.Image` | Ảnh raster đơn lẻ (nhiều band) | DEM, một bức ảnh Sentinel-2 |
| `ee.ImageCollection` | Tập hợp nhiều ảnh (time-series) | Toàn bộ archive Landsat |
| `ee.Feature` | Đối tượng vector = geometry + properties | Ranh giới một tỉnh |
| `ee.FeatureCollection` | Tập hợp nhiều Feature | Ranh giới 63 tỉnh VN |

### 20.2.2. Truy cập dữ liệu 

- **`ee.Image`**
 
Ảnh raster đơn lẻ gồm một hoặc nhiều **band** (kênh phổ). Mỗi pixel lưu một giá trị số như DEM hoặc một ảnh Sentinel-2.

Mỗi ảnh thường kèm theo các thuộc tính ví dụ như ngày chụp, nguồn gốc dữ liệu, và các thông tin liên quan khác. Điều này cho phép chúng ta có thể lọc và sử dụng dữ liệu một cách hiệu quả hơn trong các phân tích địa lý và môi trường.


```python
# ee.Image: SRTM DEM (độ cao địa hình toàn cầu, 30m)
srtm = ee.Image('USGS/SRTMGL1_003')
print(srtm.bandNames().getInfo()) # hiển thị tên các band dữ liệu trong ảnh
print(f"Các thuộc tính của ảnh: {srtm.propertyNames().getInfo()[:5]}") # hiển thị tên các thuộc tính của ảnh
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



    ['elevation']
    Các thuộc tính của ảnh: ['system:visualization_0_min', 'type_name', 'keywords', 'thumb', 'description']
    

- **`ee.ImageCollection`** 
  
Bộ sưu tập ảnh (`ee.ImageCollection`) là tập hợp nhiều ảnh `ee.Image`, thường là dữ liệu thu thập theo thời gian (time-series) cho cùng một sensor. Đây là loại đối tượng bạn sẽ làm việc nhiều nhất trong GEE. Trong ví dụ dưới đây, ta sẽ truy cập vào bộ sưu tập `Sentinel-2` và tìm hiểu các thông tin thuộc tính của ảnh này. 


```python
# Truy cập ee.ImageCollection: Sentinel-2 Surface Reflectance
s2col = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
# Lấy ảnh Sentinel-2 đầu tiên trong bộ sưu tập
s2img = s2col.first()
print(s2img.bandNames().getInfo()[:5]) # hiển thị tên các band dữ liệu trong ảnh
print(f"Các thuộc tính của ảnh: {s2img.propertyNames().getInfo()[:4]}") # hiển thị tên các thuộc tính của ảnh
print(f"Ngày chụp ảnh: {s2img.date().format('YYYY-MM-dd').getInfo()}") # hiển thị ngày chụp ảnh
print(f"Tên vệ tinh: {s2img.get('SPACECRAFT_NAME').getInfo()}") # hiển thị tên vệ tinh
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



    ['B1', 'B2', 'B3', 'B4', 'B5']
    Các thuộc tính của ảnh: ['system:version', 'system:id', 'SPACECRAFT_NAME', 'SATURATED_DEFECTIVE_PIXEL_PERCENTAGE']
    Ngày chụp ảnh: 2015-07-04
    Tên vệ tinh: Sentinel-2A
    

- **`ee.Feature`** 
 
`ee.Feature` được xây dựng chứa một đối tượng `ee.Geometry` và một `dictionary` các thuộc tính. Trong ví dụ bên dưới, `ee.Feature` đại diện cho một điểm tại Hà Nội với các thuộc tính như tên, dân số và thông tin về việc có phải là thủ đô hay không.


```python
# ee.Feature: Tạo điểm đại diện cho Hà Nội
hanoi = ee.Feature(
    ee.Geometry.Point([105.8542, 21.0285]),
    {
        'name':       'Hà Nội',
        'population': 8246600,
        'is_capital': True,
    }
)
hanoi.getInfo()
print(f"Dân số Hà Nội: {hanoi.get('population').getInfo()} người") # hiển thị dân số Hà Nội
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



    Dân số Hà Nội: 8246600 người
    

- **`ee.FeatureCollection`** 
 
`FeatureCollection` được tạo ra từ tập hợp nhiều `ee.Feature`. GEE cung cấp sẵn nhiều dữ liệu vector công khai như ranh giới hành chính (GAUL, GADM), đường bờ biển, điểm quan trắc. Ngoài những dữ liệu có sẵn trên GEE, chúng ta có thể chuyển một `shapefile` thành `FeatureCollection` như thay vì bạn phải tải chúng lên GEE Assets.


```python
# Tạo feature collection từ dữ liệu geopandas dataframe 
germany = gpd.read_file('https://geodata.ucdavis.edu/gadm/gadm4.1/json/gadm41_DEU_1.json')

ee_features = []

for _, row in germany.iterrows():
    geom = row.geometry
    if geom is None:
        continue
    # Convert shapely geometry to GeoJSON mapping
    geojson = mapping(geom)
    ee_geom = ee.Geometry(geojson)

    # Convert row properties to a dictionary (excluding geometry)
    properties = row.drop(labels="geometry").to_dict()

    # Create ee.Feature
    ee_feature = ee.Feature(ee_geom, properties)
    ee_features.append(ee_feature)
fcol = ee.FeatureCollection(ee_features)
print(f"Số lượng feature trong collection: {fcol.size().getInfo()}")
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



    Số lượng feature trong collection: 16
    

## 20.3. Lọc dữ liệu với Filters

GEE cung cấp các phương pháp lọc `ImageCollection` để lấy đúng dữ liệu cần thiết. Ba phương thức cốt lõi:

| Phương thức | Mục đích |
|---|---|
| `.filterDate(start, end)` | Lọc theo khoảng thời gian |
| `.filterBounds(geometry)` | Lọc ảnh giao với vùng AOI |
| `.filter(ee.Filter.xxx)` | Lọc theo thuộc tính metadata |

Các phương thức này có thể **chain** (nối tiếp nhau) và được thực thi lazy trên server GEE. Ví dụ dưới đây sử dụng bộ dữ liệu Sentinel-2 và sẽ thực hành các phép lọc trong GEE. Bạn có thể đọc thêm về ảnh Sentinel-2 trên [website](https://developers.google.com/earth-engine/datasets/catalog/sentinel-2).

### 20.3.1. Lọc theo vị trí 

Trong ví dụ này, chúng ta sử dụng dữ liệu Sentinel-2 để minh họa cách lọc dữ liệu theo vị trí. Có nghĩa là bất cứ bức ảnh nào có chứa toàn bộ hoặc một phần của khu vực AOI sẽ được lấy ra sau khi áp dụng `filterBounds(aoi)`. Các bộ dữ liệu khác cũng có thể được thực hiện tương tự.


```python
# AOI: Hà Nội
aoi = ee.Geometry.BBox(105.7, 20.9, 106.0, 21.2)
# Đọc dữ liệu Sentinel-2 SR từ GEE và lọc theo lấy ảnh khu vực AOI (Hà Nội)
sen2col = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED').filterBounds(aoi)
print(sen2col.size().getInfo()) # hiển thị số lượng ảnh Sentinel-2 SR có trong AOI Hà Nội
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



    1134
    

### 20.3.2. Lọc theo ngày tháng

Để lọc ảnh theo thời gian, chúng ta sử dụng cú pháp `.filterDate(start, end)` nhằm chọn các ảnh nằm trong khoảng thời gian mong muốn. Thời gian được định dạng theo chuẩn `YYYY-MM-DD`. Trong ví dụ dưới đây, chúng ta muốn tìm tất cả các bức ảnh Sentinel-2 trong năm 2026 trên phạm vi toàn cầu.


```python
# Ví dụ ta lọc dữ liệu Sentinel-2 trong năm 2026
sen2col = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
sen2_2026 = sen2col.filterDate('2026-01-01', '2026-12-31')
print(f"Số lượng ảnh Sentinel-2 SR trong năm 2026: {sen2_2026.size().getInfo()}")
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



    Số lượng ảnh Sentinel-2 SR trong năm 2026: 1991274
    

### 20.3.3. Lọc theo thuộc tính

Ngoài ra, chúng ta có thể lọc ảnh theo thuộc tính (meta data). Cú pháp pháp cơ bản để lọc ảnh theo thuộc tính thường dùng là `.filter(ee.Filter.xxx())`. Các filter phổ biến: `lt` (less than), `gt`, `eq`, `lte`, `gte`, `inList`, `stringContains`. Các ví dụ bên dưới sẽ mô tả cách sử dụng lọc theo thuộc tính.

- **Lọc ảnh Sentinel-2 có độ mây bao phủ dưới 5%**

Để lọc ảnh theo thuộc tính, đầu tiên chúng ta cần phải biết ảnh có những thông tin thuộc tính nào. Để biết được điều đó, chúng ta có thể lấy một ảnh đầu tiên trong bộ sưu tập và xem các thuộc tính của nó. Sau đó, chúng ta có thể sử dụng phương thức filter để lọc ảnh dựa trên giá trị của một thuộc tính cụ thể. Ví dụ bên dưới giúp chúng ta xem thông tin thuộc tính của ảnh và lọc ảnh với mấy bao phủ dưới 5%.


```python
sen2col = ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
# Đầu tiên phải xem thuộc tính của Collection.
print(f"Các thuộc tính của Sentinel-2 SR: {sen2col.first().propertyNames().getInfo()[:10]}") # hiển thị 10 thuộc tính đầu tiên của ảnh Sentinel-2 SR
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



    Các thuộc tính của Sentinel-2 SR: ['system:version', 'system:id', 'SPACECRAFT_NAME', 'SATURATED_DEFECTIVE_PIXEL_PERCENTAGE', 'BOA_ADD_OFFSET_B12', 'CLOUD_SHADOW_PERCENTAGE', 'system:footprint', 'SENSOR_QUALITY', 'GENERATION_TIME', 'CLOUDY_PIXEL_OVER_LAND_PERCENTAGE']
    


```python
# Cách 1 dùng ee.Filter.xx để lọc ảnh có giá trị thuộc tính CLOUDY_PIXEL_PERCENTAGE < 5
sen2col_cloudless = sen2col.filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 5))
print(f"Số lượng ảnh Sentinel-2 SR có độ che phủ mây < 5%: {sen2col_cloudless.size().getInfo()}")
# Cách 2 dùng 
sen2col_cloudless = sen2col.filterMetadata('CLOUDY_PIXEL_PERCENTAGE', 'less_than', 5)
print(f"Số lượng ảnh Sentinel-2 SR có độ che phủ mây < 5%: {sen2col_cloudless.size().getInfo()}")
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



    Số lượng ảnh Sentinel-2 SR có độ che phủ mây < 5%: 77
    Số lượng ảnh Sentinel-2 SR có độ che phủ mây < 5%: 77
    

### 20.3.4. Kết hợp nhiều Filters

'Trong thực tế, khi làm việc với dữ liệu vệ tinh, chúng ta thường mong muốn lọc ảnh dựa trên nhiều tiêu chí khác nhau như thời gian, khu vực, độ che phủ mây, v.v. Việc sử dụng các phương thức `filter` khác nhau giúp chúng ta dễ dàng truy cập và lọc dữ liệu theo các thuộc tính mong muốn. Thông thường chúng ta sẽ lọc theo thứ tự sau thời gian (`date`) → khu vực nghiên cứu (`bounds`) → thông tin thuộc tính (`metadata`) để Google Earth Engine tối ưu hiệu suất truy vấn. 

Trong ví dụ bên dưới, giả sử chúng ta mong muốn tìm kiếm tất cả các ảnh Sentinel-2 trong năm 2026, chỉ lấy bức ảnh có chứa toàn bộ hoặc một phần của vùng nghiên cứu AOI và có độ mây/bóng mây bao phủ dưới 5%. Cuối cùng sẽ sắp xếp tất cả bức ảnh theo độ mây che phủ tăng dần, có nghĩa ảnh ít mấy sẽ đứng đầu trong bộ sưu tập.


```python
# Ví dụ lọc ảnh Sentinel-2 trong năm 2026 khu vực Hà Nội (AOI) và có độ che phủ mây < 5% và shadow < 5%
sen2col = (ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
           .filterDate('2026-01-01', '2026-12-31')
           .filterBounds(aoi)
           .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 5))
           .filter(ee.Filter.lt('CLOUD_SHADOW_PERCENTAGE', 5))
           .sort('CLOUDY_PIXEL_PERCENTAGE', True) # sắp xếp theo độ che phủ mây tăng dần, ảnh ít mây nhất sẽ đứng đầu collection
           )
print(f"Số lượng ảnh Sentinel-2 SR trong năm 2026 khu vực Hà Nội có độ che phủ mây < 5% và shadow < 5%: {sen2col.size().getInfo()}")
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



    Số lượng ảnh Sentinel-2 SR trong năm 2026 khu vực Hà Nội có độ che phủ mây < 5% và shadow < 5%: 4
    

## 20.4. Các bộ dữ liệu phổ biến trong GEE

GEE Data Catalog chứa hàng nghìn datasets. Dưới đây là các bộ dữ liệu thường dùng nhất trong nghiên cứu và ứng dụng địa không gian.

### 20.4.1. Dữ liệu Landsat

Landsat là chương trình vệ tinh quan sát Trái Đất lâu đời do NASA và USGS phối hợp vận hành, cung cấp ảnh quang học độ phân giải trung bình (30 m) với chuỗi dữ liệu liên tục từ năm 1972 đến nay. Trong Google Earth Engine, dữ liệu Landsat được sử dụng rộng rãi để nghiên cứu biến động lớp phủ đất, đô thị hóa, tài nguyên nước, nông nghiệp và môi trường theo thời gian. Bạn có thể tham khảo tất cả các bộ sưu tập Landsat tại [đây](https://developers.google.com/earth-engine/datasets/catalog/landsat).

Ví dụ dưới mô tả truy cập vào bộ dữ liệu Landsat 9 (Surface Reflectance) như sau

```bash 
landsat = ee.ImageCollection("LANDSAT/LC09/C02/T1_L2")
```


```python
# Truy cập bộ sưu tập Landsat 9 Surface Reflectance trên GEE. Bộ sưu tập này có tên là "LANDSAT/LC09/C02/T1_L2" và chứa ảnh Landsat 9 đã được xử lý để có giá trị phản xạ bề mặt (Surface Reflectance). Ta có thể lọc bộ sưu tập này theo ngày, khu vực, hoặc các thuộc tính khác tương tự như cách đã làm với Sentinel-2.
landsat = ee.ImageCollection("LANDSAT/LC09/C02/T1_L2")
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



- **Kiểm tra thông tin thuộc tính**

Mỗi bộ sưu tập ảnh sẽ có các thông tin thuộc tính khác nhau tùy thuộc vào loại ảnh và bộ sưu tập. Ví dụ, ảnh Landsat 9 SR có thể có các thuộc tính như `CLOUDY_PIXEL_PERCENTAGE`, trong khi ảnh Landsat 9 L2 có thể có các thuộc tính như `RADIOMETRIC_QUALITY`. Việc hiểu rõ các thuộc tính này sẽ giúp bạn lọc và chọn lựa ảnh phù hợp cho phân tích không gian của mình. Để kiểm tra thông tin thuộc tính cho bộ sưu tập ảnh, bạn có thể làm như ví dụ bên dưới.


```python
print(f"Các thuộc tính của ảnh Landsat 9 L2: {landsat.first().propertyNames().getInfo()[:10]}") # hiển thị 10 thuộc tính đầu tiên của ảnh Landsat 9 L2
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



    Các thuộc tính của ảnh Landsat 9 L2: ['DATA_SOURCE_ELEVATION', 'WRS_TYPE', 'system:id', 'REFLECTANCE_ADD_BAND_1', 'REFLECTANCE_ADD_BAND_2', 'DATUM', 'REFLECTANCE_ADD_BAND_3', 'REFLECTANCE_ADD_BAND_4', 'REFLECTANCE_ADD_BAND_5', 'REFLECTANCE_ADD_BAND_6']
    

- **Kiểm tra các bands**

Mỗi ảnh có thể có một hoặc nhiều bands. Để kiểm tra số lượng bands trong một ảnh, bạn có thể sử dụng phương thức `bandNames()` của đối tượng `ee.Image`. Ví dụ, nếu bạn muốn kiểm tra tên các bands trong ảnh Landsat-9, bạn có thể làm như sau:


```python
print(f"Các bands của ảnh Landsat 9 L2: {landsat.first().bandNames().getInfo()[:10]}") # hiển thị 10 bands đầu tiên của ảnh Landsat 9 L2
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



    Các bands của ảnh Landsat 9 L2: ['SR_B1', 'SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B6', 'SR_B7', 'SR_QA_AEROSOL', 'ST_B10', 'ST_ATRAN']
    

### 20.4.2. Ảnh Sentinel-1 

Sentinel-1 là bộ dữ liệu ảnh radar khẩu độ tổng hợp (SAR) được phát triển bởi ESA và cung cấp trong Google Earth Engine (GEE) từ vệ tinh Sentinel-1. Khác với ảnh quang học, Sentinel-1 sử dụng sóng vi ba nên có khả năng thu nhận dữ liệu cả ngày lẫn đêm và xuyên qua mây, giúp quan sát bề mặt Trái Đất liên tục trong mọi điều kiện thời tiết.

Để truy cập bộ dữ liệu này, ta làm như sau:

```bash
sen1col = ee.ImageCollection("COPERNICUS/S1_GRD")
```


```python
# Để truy cập ảnh Sentinel-1, ta có thể sử dụng ee.ImageCollection("COPERNICUS/S1_GRD") để lấy toàn bộ bộ sưu tập Sentinel-1 GRD (Ground Range Detected). Sau đó, ta có thể lọc theo ngày, khu vực, hoặc các thuộc tính khác tương tự như cách đã làm với Sentinel-2 và Landsat.
sen1col = ee.ImageCollection("COPERNICUS/S1_GRD")
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



- **Kiểm tra thuộc tính của ảnh**
  
Tương tự như ảnh Landsat hay Sentinel-2. Chúng ta có thể truy cập vào thông tin thuộc tính ảnh Sentinel-1 như bên dưới.


```python
print(f"Các thuộc tính của ảnh Sentinel-1 GRD: {sen1col.first().propertyNames().getInfo()[:10]}") # hiển thị 10 thuộc tính đầu tiên của ảnh Sentinel-1 GRD
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



    Các thuộc tính của ảnh Sentinel-1 GRD: ['system:version', 'system:id', 'SNAP_Graph_Processing_Framework_GPF_vers', 'SLC_Processing_facility_org', 'SLC_Processing_facility_country', 'GRD_Post_Processing_facility_org', 'transmitterReceiverPolarisation', 'GRD_Post_Processing_start', 'sliceNumber', 'GRD_Post_Processing_facility_name']
    

- **Kiểm tra bands của ảnh**

Tương tự như vậy, ta có thể kiểm tra các bands trong ảnh Sentinel-1.


```python
print(f"Các bands trong ảnh Sentinel-1 GRD: {sen1col.first().bandNames().getInfo()}") # hiển thị tên các band dữ liệu trong ảnh Sentinel-1 GRD
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



    Các bands trong ảnh Sentinel-1 GRD: ['HH', 'HV', 'angle']
    

### 20.4.3. Ảnh MODIS 

MODIS (Moderate Resolution Imaging Spectroradiometer) là cảm biến quang học trên các vệ tinh Terra và Aqua, cung cấp dữ liệu quan sát Trái Đất với tần suất cao và phạm vi phủ toàn cầu. Trong Google Earth Engine, MODIS được sử dụng rộng rãi để theo dõi thảm thực vật, nhiệt độ bề mặt, lớp phủ đất và các biến động môi trường thông qua các chuỗi dữ liệu dài hạn có độ phân giải từ 250 m đến 1 km.

Để truy cập vào dữ liệu MODIS, ta có thể làm như sau:

```bash
terra = ee.ImageCollection("MODIS/061/MOD13A2")
aqua = ee.ImageCollection("MODIS/061/MYD13A2")
```


```python
# Truy cập bộ sưu tập MODIS NDVI (Terra + Aqua) trên GEE. MODIS có 2 bộ sưu tập NDVI riêng biệt cho 2 vệ tinh Terra và Aqua, ta sẽ gộp 2 bộ sưu tập này lại với nhau để có nhiều ảnh NDVI hơn. Sau đó, ta sẽ sắp xếp theo ngày chụp ảnh tăng dần để dễ dàng truy cập ảnh mới nhất.
terra = ee.ImageCollection("MODIS/061/MOD13A2")
aqua = ee.ImageCollection("MODIS/061/MYD13A2")
modis = terra.merge(aqua).sort('system:time_start') # gộp 2 bộ sưu tập MODIS Terra và Aqua lại với nhau và sắp xếp theo ngày chụp ảnh tăng dần
print(f"Số lượng ảnh MODIS NDVI (Terra + Aqua): {modis.size().getInfo()}")
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



    Số lượng ảnh MODIS NDVI (Terra + Aqua): 1153
    

- **Kiểm tra thông tin thuộc tính**

Kiểm tra thuộc tính với ảnh `MODIS` tương tự như các ảnh khác.


```python
print(f"Các thuộc tính của ảnh Modis: {modis.first().propertyNames().getInfo()[:10]}") # hiển thị 10 thuộc tính đầu tiên của ảnh MODIS
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



    Các thuộc tính của ảnh Modis: ['system:index', 'system:time_start', 'google:max_source_file_timestamp', 'num_tiles', 'system:footprint', 'system:time_end', 'system:version', 'system:id', 'system:asset_size', 'system:bands']
    

- **Kiểm tra bands trong ảnh**

Kiểm tra bands với ảnh `MOIDIS` cũng tương tự như với các ảnh khác.


```python
print(f"Các bands trong ảnh MODIS: {modis.first().bandNames().getInfo()}") # hiển thị tên các band dữ liệu trong ảnh MODIS
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



    Các bands trong ảnh MODIS: ['NDVI', 'EVI', 'DetailedQA', 'sur_refl_b01', 'sur_refl_b02', 'sur_refl_b03', 'sur_refl_b07', 'ViewZenith', 'SolarZenith', 'RelativeAzimuth', 'DayOfYear', 'SummaryQA']
    

## Tóm tắt

### Các khái niệm chính đã nắm vững:

| Đối tượng / Method | Mục đích |
|---|---|
| `ee.Image` | Ảnh raster đơn — đọc bands, thống kê vùng |
| `ee.ImageCollection` | Time-series — nền tảng để lọc và tính toán |
| `ee.Feature / FeatureCollection` | Vector data — ranh giới hành chính, điểm quan trắc |
| `.filterDate()` | Lọc theo khoảng thời gian |
| `.filterBounds()` | Lọc theo vùng địa lý |
| `.filter(ee.Filter.lt/gt/eq)` | Lọc theo metadata (cloud, orbit,...) |

### Kỹ năng bạn có thể áp dụng:
- Xác thực và kết nối GEE Python API trong môi trường Jupyter Notebook
- Truy cập và kiểm tra thông tin ảnh vệ tinh từ GEE Data Catalog (bands, properties, ngày chụp)
- Lọc chính xác `ImageCollection` theo thời gian, vùng không gian và chất lượng ảnh
- Tạo `FeatureCollection` từ dữ liệu GeoDataFrame (GeoPandas) để dùng làm AOI
- Kết hợp nhiều bộ lọc hiệu quả để chuẩn bị dữ liệu đầu vào cho các bài toán viễn thám
- Đặt nền tảng vững chắc cho các bài tiếp theo: cloud masking, tính chỉ số thực vật, tổng hợp ảnh composite
