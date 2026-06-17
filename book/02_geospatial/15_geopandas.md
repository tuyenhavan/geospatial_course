# Bài 15: Phân tích Dữ liệu vector với GeoPandas

GeoPandas là thư viện mạnh mẽ nhất cho phân tích dữ liệu địa không gian trong Python, kết hợp sức mạnh của pandas và Shapely để mang đến trải nghiệm xử lý dữ liệu GIS hoàn hảo.

## 15.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

- Tạo và manipulate GeoDataFrames từ các nguồn dữ liệu đa dạng
- Đọc và ghi dữ liệu địa không gian ở nhiều format khác nhau (Shapefile, GeoJSON, etc.)
- Làm chủ hệ tọa độ (CRS) và thực hiện chuyển đổi projection chính xác
- Tạo visualizations và maps chuyên nghiệp với matplotlib integration
- Thực hiện spatial operations phức tạp như joins, overlays, và buffer operations
- Tích hợp GeoPandas với các thư viện GIS khác trong ecosystem Python


```python
# Nếu chưa có thư thư viện sau thì cài đặt pip install geopandas pandas shapely
import geopandas as gpd           # Thư viện chính cho phân tích địa không gian
import pandas as pd                # Xử lý dữ liệu bảng
from shapely.geometry import Point, Polygon
import os 
outpath = r'G:\My Drive\python\geocourse\data\vector'
```

## 15.2 Tạo và hiểu GeoDataFrames

GeoDataFrame là **pandas DataFrame đặc biệt** có cột geometry chứa các đối tượng hình học Shapely. Đây là nền tảng của mọi phân tích địa không gian trong GeoPandas. Cũng giống như pandas, geopandas có hai loại object chính là  `Geoseries` và `GeoDataFrame`.

### 15.2.1. Tạo GeoSeries

GeoSeries là một cột duy nhất chứa geometry với hệ tọa độ (CRS)


```python
geo_series = gpd.GeoSeries([
    Point(105.85, 21.02),  # Hà Nội
    Point(106.63, 10.77),  # TP.HCM
], crs='EPSG:4326')
print(f"Kiểu dữ liệu GeoSeries:\n{geo_series}\n")
```

    Kiểu dữ liệu GeoSeries:
    0    POINT (105.85 21.02)
    1    POINT (106.63 10.77)
    dtype: geometry
    
    

### 15.2.2. Tạo GeoDataFrame từ dictionary

Tạo GeoDataFrame từ DataFrame và cột geometry giúp chúng ta dễ dàng làm việc với dữ liệu địa lý trong Python. Bằng cách sử dụng geopandas, chúng ta có thể tận dụng các tính năng mạnh mẽ của thư viện này để phân tích và trực quan hóa dữ liệu không gian một cách hiệu quả.


```python
# Dữ liệu thành phố lớn Việt Nam (tọa độ mang tính tương đối). Dữ liệu này chỉ mang tính minh họa và nên được kiểm tra lại với dữ liệu chính thức.
vietnam_cities_data = {
    'city': ['Hà Nội', 'TP.HCM', 'Hải Phòng', 'Đà Nẵng', 'Cần Thơ', 'Biên Hòa', 'Huế', 'Nha Trang', 'Buôn Ma Thuột', 'Quy Nhon'],
    'province': ['Hà Nội', 'TP.HCM', 'Hải Phòng', 'Đà Nẵng', 'Cần Thơ', 'Đồng Nai', 'Thừa Thiên Huế', 'Khánh Hòa', 'Đắk Lắk', 'Bình Định'],
    'region': ['Miền Bắc', 'Miền Nam', 'Miền Bắc', 'Miền Trung', 'Miền Nam', 'Miền Nam', 'Miền Trung', 'Miền Trung', 'Miền Trung', 'Miền Trung'],
    'population': [8246600, 8993082, 2028514, 1134310, 1282937, 1104800, 455230, 423000, 340000, 284000],
    'longitude': [105.8542, 106.6297, 106.6881, 108.2022, 105.7469, 106.8439, 107.5905, 109.1967, 108.0373, 109.2189],
    'latitude': [21.0285, 10.8231, 20.8449, 16.0544, 10.0452, 10.9460, 16.4637, 12.2585, 12.6667, 13.7830],
    'is_port_city': [False, True, True, True, True, False, False, True, False, True],
}

# Tạo DataFrame thường trước
df = pd.DataFrame(vietnam_cities_data)
# tạo cột geometry từ longitude và latitude
geometry = [Point(xy) for xy in zip(df.longitude, df.latitude)]
# Tạo GeoDataFrame từ DataFrame và cột geometry
gdf = gpd.GeoDataFrame(
    df.drop(['longitude', 'latitude'], axis=1), 
    geometry=geometry, 
    crs='EPSG:4326'
)
gdf.head()
```




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
      <th>city</th>
      <th>province</th>
      <th>region</th>
      <th>population</th>
      <th>is_port_city</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hà Nội</td>
      <td>Hà Nội</td>
      <td>Miền Bắc</td>
      <td>8246600</td>
      <td>False</td>
      <td>POINT (105.8542 21.0285)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>TP.HCM</td>
      <td>TP.HCM</td>
      <td>Miền Nam</td>
      <td>8993082</td>
      <td>True</td>
      <td>POINT (106.6297 10.8231)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Hải Phòng</td>
      <td>Hải Phòng</td>
      <td>Miền Bắc</td>
      <td>2028514</td>
      <td>True</td>
      <td>POINT (106.6881 20.8449)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Đà Nẵng</td>
      <td>Đà Nẵng</td>
      <td>Miền Trung</td>
      <td>1134310</td>
      <td>True</td>
      <td>POINT (108.2022 16.0544)</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Cần Thơ</td>
      <td>Cần Thơ</td>
      <td>Miền Nam</td>
      <td>1282937</td>
      <td>True</td>
      <td>POINT (105.7469 10.0452)</td>
    </tr>
  </tbody>
</table>
</div>



### 15.2.3. Tạo GeoDataFrame từ list


```python
# Tạo GeoDataFrames từ polygons sử dụng shapely và gán vào geometry
# Tạo một số polygons ví dụ
geometry = [
    Polygon([(105, 20), (106, 20), (106, 21), (105, 21)]),
    Polygon([(106, 10), (107, 10), (107, 11), (106, 11)]),
    Polygon([(108, 16), (109, 16), (109, 17), (108, 17)]),
    Polygon([(109, 12), (110, 12), (110, 13), (109, 13)]),
    Polygon([(105, 10), (106, 10), (106, 11), (105, 11)]),
    Polygon([(106, 10), (107, 10), (107, 11), (106, 11)]),
    Polygon([(107, 16), (108, 16), (108, 17), (107, 17)]),
    Polygon([(109, 12), (110, 12), (110, 13), (109, 13)]),
    Polygon([(108, 12), (109, 12), (109, 13), (108, 13)]),
    Polygon([(109, 13), (110, 13), (110, 14), (109, 14)]),
]
polygon = gpd.GeoDataFrame(
    geometry=geometry,
    crs='EPSG:4326'
)
# Ta có thêm cột thuộc tính vào GeoDataFrame polygon
polygon["landcover"] = ["Urban", "Urban", "Rural", "Rural", "Urban", "Urban", "Rural", "Rural", "Rural", "Rural"]
polygon.head()
```




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
      <th>geometry</th>
      <th>landcover</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>POLYGON ((105 20, 106 20, 106 21, 105 21, 105 ...</td>
      <td>Urban</td>
    </tr>
    <tr>
      <th>1</th>
      <td>POLYGON ((106 10, 107 10, 107 11, 106 11, 106 ...</td>
      <td>Urban</td>
    </tr>
    <tr>
      <th>2</th>
      <td>POLYGON ((108 16, 109 16, 109 17, 108 17, 108 ...</td>
      <td>Rural</td>
    </tr>
    <tr>
      <th>3</th>
      <td>POLYGON ((109 12, 110 12, 110 13, 109 13, 109 ...</td>
      <td>Rural</td>
    </tr>
    <tr>
      <th>4</th>
      <td>POLYGON ((105 10, 106 10, 106 11, 105 11, 105 ...</td>
      <td>Urban</td>
    </tr>
  </tbody>
</table>
</div>



## 15.3. Đọc và Lưu Dữ liệu Địa không gian

GeoPandas có thể đọc và ghi hơn **20 định dạng** địa không gian khác nhau. Đây là kỹ năng thiết yếu cho công việc thực tế. Trong phần này, chúng ta sẽ khám phá đọc và ghi các loại dữ liệu chính trong GIS.

Dữ liệu sử dụng trong notebook này được tải từ [GADM](https://gadm.org/) và NDVI từ ảnh MODI.

### 15.3.1. Đọc dữ liệu

- **Đọc dữ liệu lưu trữ trên máy**


```python
# Đọc dữ liệu từ local machine
districts = gpd.read_file(r'G:\My Drive\python\geocourse\data\vector\Vietnam_districts.geojson')
# Chuyển crs từ 4326 sang 32648 (UTM 48N)
districts = districts.to_crs(epsg=32648)
# Thêm cột diện tích 
districts['area_km2'] = districts.geometry.area/1e6  # Chuyển từ m2 sang km2
districts.head()
```




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
      <th>codes</th>
      <th>NAME_1</th>
      <th>NAME_2</th>
      <th>NDVI</th>
      <th>geometry</th>
      <th>area_km2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>A01</td>
      <td>AnGiang</td>
      <td>AnPhú</td>
      <td>0.571832</td>
      <td>MULTIPOLYGON (((517117.408 1197043.213, 517502...</td>
      <td>226.849284</td>
    </tr>
    <tr>
      <th>1</th>
      <td>A02</td>
      <td>AnGiang</td>
      <td>ChâuĐốc</td>
      <td>0.596210</td>
      <td>MULTIPOLYGON (((511484.367 1175988.508, 508914...</td>
      <td>103.388808</td>
    </tr>
    <tr>
      <th>2</th>
      <td>A03</td>
      <td>AnGiang</td>
      <td>ChâuPhú</td>
      <td>0.583617</td>
      <td>MULTIPOLYGON (((520563.388 1156311.715, 519676...</td>
      <td>450.432027</td>
    </tr>
    <tr>
      <th>3</th>
      <td>A04</td>
      <td>AnGiang</td>
      <td>ChâuThành</td>
      <td>0.605679</td>
      <td>MULTIPOLYGON (((540603.418 1145482.678, 540308...</td>
      <td>354.959119</td>
    </tr>
    <tr>
      <th>4</th>
      <td>A05</td>
      <td>AnGiang</td>
      <td>ChợMới</td>
      <td>0.607893</td>
      <td>MULTIPOLYGON (((553305.107 1165103.948, 553435...</td>
      <td>368.509538</td>
    </tr>
  </tbody>
</table>
</div>



- **Đọc dữ liệu từ `url`**


```python
# Đọc dữ liệu từ url 
vietnam_data = gpd.read_file('https://geodata.ucdavis.edu/gadm/gadm4.1/json/gadm41_VNM_1.json')
vietnam_data.head()
```




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
      <th>GID_1</th>
      <th>GID_0</th>
      <th>COUNTRY</th>
      <th>NAME_1</th>
      <th>VARNAME_1</th>
      <th>NL_NAME_1</th>
      <th>TYPE_1</th>
      <th>ENGTYPE_1</th>
      <th>CC_1</th>
      <th>HASC_1</th>
      <th>ISO_1</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>VNM.1_1</td>
      <td>VNM</td>
      <td>Vietnam</td>
      <td>AnGiang</td>
      <td>AnGiang</td>
      <td>NA</td>
      <td>Tỉnh</td>
      <td>Province</td>
      <td>NA</td>
      <td>VN.AG</td>
      <td>VN-44</td>
      <td>MULTIPOLYGON (((105.5486 10.4295, 105.5495 10....</td>
    </tr>
    <tr>
      <th>1</th>
      <td>VNM.7_1</td>
      <td>VNM</td>
      <td>Vietnam</td>
      <td>BàRịa-VũngTàu</td>
      <td>BaRia-VungTau</td>
      <td>NA</td>
      <td>Tỉnh</td>
      <td>Province</td>
      <td>NA</td>
      <td>VN.BV</td>
      <td>NA</td>
      <td>MULTIPOLYGON (((107.0901 10.324, 107.0889 10.3...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>VNM.3_1</td>
      <td>VNM</td>
      <td>Vietnam</td>
      <td>BắcGiang</td>
      <td>BacGiang</td>
      <td>NA</td>
      <td>Tỉnh</td>
      <td>Province</td>
      <td>NA</td>
      <td>VN.BG</td>
      <td>NA</td>
      <td>MULTIPOLYGON (((106.2838 21.1323, 106.2734 21....</td>
    </tr>
    <tr>
      <th>3</th>
      <td>VNM.4_1</td>
      <td>VNM</td>
      <td>Vietnam</td>
      <td>BắcKạn</td>
      <td>BacKan</td>
      <td>NA</td>
      <td>Tỉnh</td>
      <td>Province</td>
      <td>NA</td>
      <td>VN.BK</td>
      <td>NA</td>
      <td>MULTIPOLYGON (((105.8724 21.8558, 105.8629 21....</td>
    </tr>
    <tr>
      <th>4</th>
      <td>VNM.2_1</td>
      <td>VNM</td>
      <td>Vietnam</td>
      <td>BạcLiêu</td>
      <td>BacLieu</td>
      <td>NA</td>
      <td>Tỉnh</td>
      <td>Province</td>
      <td>NA</td>
      <td>VN.BL</td>
      <td>NA</td>
      <td>MULTIPOLYGON (((105.4244 9.0213, 105.4164 9.01...</td>
    </tr>
  </tbody>
</table>
</div>



### 15.3.2. Viết dữ liệu

- **Lưu dữ liệu ra file `GeoJSON`**


```python
# Lưu dữ liệu ra file GeoJSON
vietnam_data.to_file(os.path.join(outpath, 'vietnam_provinces.geojson'), driver='GeoJSON')
```

- **Lưu dữ liệu ra `shapefile`**


```python
# Lưu dữ liệu ra file Shapefile
vietnam_data.to_file(os.path.join(outpath, 'Vietnam_provincess.shp'), driver='ESRI Shapefile')
```

- **Lưu dữ liệu ra `Parquet` file**


```python
# Lưu dữ liệu ra file Parquet
vietnam_data.to_file(os.path.join(outpath, 'Vietnam_provinces.parquet'), driver='Parquet')
```

- **Lưu dữ liệu ra `GeoPackage`**


```python
# Lưu dữ liệu ra file GeoPackage
vietnam_data.to_file(os.path.join(outpath, 'Vietnam_provinces.gpkg'), driver='GPKG')
```

## 15.4. Thao tác và Phân tích Không gian

Geopandas cho phép người dùng có thể dễ dàng thao tác với dữ liệu địa lý bằng cách sử dụng các phương thức và thuộc tính của GeoDataFrame. Bạn có thể chọn các cột cụ thể, đổi tên cột, lọc dữ liệu dựa trên điều kiện, và thực hiện nhiều thao tác địa lý khác.


```python
# Đọc dữ liệu từ url 
province = gpd.read_file('https://geodata.ucdavis.edu/gadm/gadm4.1/json/gadm41_VNM_1.json')
# Select two columns 
province = province[['VARNAME_1', 'geometry']]
# Rename the column
province = province.rename(columns={'VARNAME_1': 'province'})
province.head()
```




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
      <th>province</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>AnGiang</td>
      <td>MULTIPOLYGON (((105.5486 10.4295, 105.5495 10....</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BaRia-VungTau</td>
      <td>MULTIPOLYGON (((107.0901 10.324, 107.0889 10.3...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>BacGiang</td>
      <td>MULTIPOLYGON (((106.2838 21.1323, 106.2734 21....</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BacKan</td>
      <td>MULTIPOLYGON (((105.8724 21.8558, 105.8629 21....</td>
    </tr>
    <tr>
      <th>4</th>
      <td>BacLieu</td>
      <td>MULTIPOLYGON (((105.4244 9.0213, 105.4164 9.01...</td>
    </tr>
  </tbody>
</table>
</div>



### 15.4.1. Tạo buffer

Trước khi tạo buffer, chúng ta nên chuyển dữ liệu qua hệ tọa độ UTM cho độ chính xác cao hơn.


```python
# Chuyển hệ tọa độ từ WGS84 (EPSG:4326) sang UTM 48N (EPSG:32648)
province_utm = province.to_crs(epsg=32648)
# Chọn tỉnh vĩnh phúc
vinhphuc = province_utm[province_utm['province'] == 'VinhPhuc']
# Tạo 1km buffer quanh tỉnh vĩnh phúc
vinhphuc_buffer = vinhphuc.buffer(1000)  # Buffer 1000 mét (1 km)
# Chuyển từ GeoSeries sang GeoDataFrame để dễ dàng xử lý
vinhphuc_buffer_gdf = gpd.GeoDataFrame(geometry=vinhphuc_buffer, crs='EPSG:32648')
vinhphuc_buffer_gdf.head()
```




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
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>61</th>
      <td>POLYGON ((534128.869 2379245.172, 534157.041 2...</td>
    </tr>
  </tbody>
</table>
</div>



### 15.4.2. Sử dụng phép join giữa hai `GeoDataFrame`


```python
# Đọc dữ liệu districts từ url
districts = gpd.read_file('https://geodata.ucdavis.edu/gadm/gadm4.1/json/gadm41_VNM_2.json')
# Đảm bảo dữ liệu vinhphuc có cùng hệ tọa độ với districts trước khi join
vinhphuc = vinhphuc.to_crs(districts.crs)
# Join dữ liệu tỉnh vĩnh phúc với dữ liệu districts để lấy ra các huyện thuộc tỉnh vĩnh phúc
vinhphuc_districts = gpd.sjoin(vinhphuc, districts, how='inner', predicate='intersects') # ngoài intersects còn có within, contains, touches, crosses, covers, covered_by.
vinhphuc_districts.head()
```




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
      <th>province</th>
      <th>geometry</th>
      <th>index_right</th>
      <th>GID_2</th>
      <th>GID_0</th>
      <th>COUNTRY</th>
      <th>GID_1</th>
      <th>NAME_1</th>
      <th>NL_NAME_1</th>
      <th>NAME_2</th>
      <th>VARNAME_2</th>
      <th>NL_NAME_2</th>
      <th>TYPE_2</th>
      <th>ENGTYPE_2</th>
      <th>CC_2</th>
      <th>HASC_2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>61</th>
      <td>VinhPhuc</td>
      <td>MULTIPOLYGON (((105.5713 21.1615, 105.5336 21....</td>
      <td>250</td>
      <td>VNM.27.23_1</td>
      <td>VNM</td>
      <td>Vietnam</td>
      <td>VNM.27_1</td>
      <td>HàNội</td>
      <td>NA</td>
      <td>SócSơn</td>
      <td>SocSon</td>
      <td>NA</td>
      <td>Huyện</td>
      <td>District</td>
      <td>NA</td>
      <td>VN.NB.HL</td>
    </tr>
    <tr>
      <th>61</th>
      <td>VinhPhuc</td>
      <td>MULTIPOLYGON (((105.5713 21.1615, 105.5336 21....</td>
      <td>615</td>
      <td>VNM.56.4_1</td>
      <td>VNM</td>
      <td>Vietnam</td>
      <td>VNM.56_1</td>
      <td>TháiNguyên</td>
      <td>NA</td>
      <td>PhổYên</td>
      <td>PhoYen</td>
      <td>NA</td>
      <td>Thịxã</td>
      <td>Town</td>
      <td>NA</td>
      <td>VN.TV.TI</td>
    </tr>
    <tr>
      <th>61</th>
      <td>VinhPhuc</td>
      <td>MULTIPOLYGON (((105.5713 21.1615, 105.5336 21....</td>
      <td>230</td>
      <td>VNM.27.4_1</td>
      <td>VNM</td>
      <td>Vietnam</td>
      <td>VNM.27_1</td>
      <td>HàNội</td>
      <td>NA</td>
      <td>BaVì</td>
      <td>BaVi</td>
      <td>NA</td>
      <td>Huyện</td>
      <td>District</td>
      <td>NA</td>
      <td>VN.KG.HT</td>
    </tr>
    <tr>
      <th>61</th>
      <td>VinhPhuc</td>
      <td>MULTIPOLYGON (((105.5713 21.1615, 105.5336 21....</td>
      <td>251</td>
      <td>VNM.27.24_1</td>
      <td>VNM</td>
      <td>Vietnam</td>
      <td>VNM.27_1</td>
      <td>HàNội</td>
      <td>NA</td>
      <td>SơnTây</td>
      <td>SonTay</td>
      <td>NA</td>
      <td>Thịxã</td>
      <td>Town</td>
      <td>NA</td>
      <td>VN.TN.HT</td>
    </tr>
    <tr>
      <th>61</th>
      <td>VinhPhuc</td>
      <td>MULTIPOLYGON (((105.5713 21.1615, 105.5336 21....</td>
      <td>248</td>
      <td>VNM.27.21_1</td>
      <td>VNM</td>
      <td>Vietnam</td>
      <td>VNM.27_1</td>
      <td>HàNội</td>
      <td>NA</td>
      <td>PhúcThọ</td>
      <td>PhucTho</td>
      <td>NA</td>
      <td>Huyện</td>
      <td>District</td>
      <td>NA</td>
      <td>VN.HO.HB</td>
    </tr>
  </tbody>
</table>
</div>



## Tóm tắt

Bạn đã hoàn thành Bài 5 và học được GeoPandas - thư viện "con dao Thụy Sĩ" cho geospatial data analysis trong Python.

### Các khái niệm chính đã nắm vững:
- ✅ **GeoDataFrames**: Pandas DataFrames với geometry column cho spatial data
- ✅ **Spatial I/O**: Đọc/ghi Shapefile, GeoJSON, GPKG với read_file() và to_file()
- ✅ **CRS management**: Coordinate reference systems với to_crs() transformations
- ✅ **Spatial operations**: buffer(), intersects(), within() cho geometric analysis
- ✅ **Spatial joins**: Kết hợp datasets dựa trên spatial relationships

### Kỹ năng bạn có thể áp dụng:
- Thực hiện comprehensive spatial analysis với pandas-like syntax
- Xử lý và visualize complex geospatial datasets một cách hiệu quả
- Tích hợp spatial data với business intelligence và data science workflows
- Phát triển location-based applications và market analysis tools
- Chuẩn bị expertise foundation cho advanced GIS development và spatial data science
