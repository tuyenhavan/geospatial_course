# Bài 15: Phân tích Dữ liệu vector với GeoPandas

GeoPandas là thư viện mạnh mẽ nhất cho phân tích dữ liệu địa không gian trong Python, kết hợp sức mạnh của pandas và Shapely để mang đến trải nghiệm xử lý dữ liệu GIS hoàn hảo.

> **Lưu ý**: Bạn có thể chạy trực tiếp notebook này bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/1QJjw-5dmbrj6kO3Dm_XMYilZJpnjk22t) mà không cần cài đặt Python.

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
outpath = r'J:\My Drive\geocourse_data\outputs'
```

## 15.2 Tạo và hiểu GeoDataFrames

GeoDataFrame là **pandas DataFrame đặc biệt** có cột geometry chứa các đối tượng hình học Shapely. Đây là nền tảng của mọi phân tích địa không gian trong GeoPandas. Cũng giống như pandas, geopandas có hai loại object chính là  `Geoseries` và `GeoDataFrame`.

### 15.2.1. Tạo GeoSeries

GeoSeries là cấu trúc dữ liệu cơ bản nhất trong GeoPandas, đại diện cho một cột chứa các đối tượng geometry (như Point, LineString, Polygon) cùng với hệ tọa độ tham chiếu (CRS). GeoSeries tương tự như pandas Series nhưng được tối ưu hóa cho dữ liệu không gian, cho phép thực hiện các phép toán geometric và spatial indexing. Bạn tạo GeoSeries bằng cách truyền danh sách các đối tượng Shapely geometry và chỉ định CRS.


```python
geo_series = gpd.GeoSeries([
    Point(105.85, 21.02),  # Hà Nội
    Point(106.63, 10.77),  # TP.HCM
], crs='EPSG:4326')
print(f"Kiểu dữ liệu GeoSeries:\n{geo_series}\n")
```

### 15.2.2. Tạo GeoDataFrame từ dictionary

Tạo GeoDataFrame từ DataFrame và cột geometry giúp chúng ta dễ dàng làm việc với dữ liệu địa lý trong Python. Bằng cách sử dụng geopandas, chúng ta có thể tận dụng các tính năng mạnh mẽ của thư viện này để phân tích và trực quan hóa dữ liệu không gian một cách hiệu quả.


```python
# Dữ liệu thành phố lớn Việt Nam (tọa độ mang tính tương đối). Dữ liệu này chỉ mang tính minh họa và nên được kiểm tra lại với dữ liệu chính thức.
vietnam_cities_data = {
    'city': ['Hà Nội', 'TP.HCM', 'Hải Phòng', 'Đà Nẵng', 'Cần Thơ', 'Biên Hòa', 'Huế', 'Nha Trang', 'Buôn Ma Thuột', 'Quy Nhon'],
    'province': ['Hà Nội', 'TP.HCM', 'Hải Phòng', 'Đà Nẵng', 'Cần Thơ', 'Đồng Nai', 'Thừa Thiên Huế', 'Khánh Hòa', 'Đắk Lắk', 'Bình Định'],
    'region': ['Miền Bắc', 'Miền Nam', 'Miền Bắc', 'Miền Trung', 'Miền Nam', 'Miền Nam', 'Miền Trung', 'Miền Trung', 'Miền Trung', 'Miền Trung'],
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
gdf.head(2)
```

### 15.2.3. Tạo GeoDataFrame từ list

Ngoài việc tạo GeoDataFrame từ dictionary, bạn có thể tạo trực tiếp từ list các đối tượng geometry. Phương pháp này đặc biệt hữu ích khi bạn đã có sẵn các đối tượng Shapely geometry (như Polygon, LineString) và muốn chuyển chúng thành GeoDataFrame để phân tích. Bạn chỉ cần truyền list geometry vào tham số `geometry` và chỉ định CRS. Sau đó có thể thêm các cột thuộc tính khác bằng cách gán trực tiếp như pandas DataFrame.


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
polygon.head(2)
```

## 15.3. Đọc và Lưu Dữ liệu Địa không gian

GeoPandas có thể đọc và ghi hơn **20 định dạng** địa không gian khác nhau. Đây là kỹ năng thiết yếu cho công việc thực tế. Trong phần này, chúng ta sẽ khám phá đọc và ghi các loại dữ liệu chính trong GIS.

Dữ liệu sử dụng trong notebook này được tải từ [GADM](https://gadm.org/) và NDVI từ ảnh MODI.

### 15.3.1. Đọc dữ liệu

GeoPandas sử dụng `read_file()` để đọc dữ liệu địa không gian từ nhiều nguồn khác nhau. Hàm này tự động nhận diện định dạng file (Shapefile, GeoJSON, GPKG, KML...) và đọc vào GeoDataFrame. Bạn có thể đọc từ file local trên máy tính hoặc trực tiếp từ URL trên internet. Khi đọc xong, dữ liệu được tải vào bộ nhớ dưới dạng GeoDataFrame với đầy đủ geometry và attributes, cho phép thao tác như pandas DataFrame nhưng có thêm các phương thức spatial.

- **Đọc dữ liệu lưu trữ trên máy**

Khi làm việc với dữ liệu local, bạn cung cấp đường dẫn tuyệt đối hoặc tương đối đến file. GeoPandas sẽ tự động detect định dạng dựa trên phần mở rộng file (.shp, .geojson, .gpkg...). Sau khi đọc, bạn có thể thực hiện các thao tác như chuyển đổi CRS (coordinate reference system) với `to_crs()` để phù hợp với hệ tọa độ cần thiết cho phân tích. Ví dụ, chuyển từ WGS84 (EPSG:4326) sang UTM để tính toán diện tích chính xác hơn.


```python
# Đọc dữ liệu từ local machine
districts = gpd.read_file(r"J:\My Drive\geocourse_data\vector\vinhphuc_districts.geojson")
# Chuyển crs từ 4326 sang 32648 (UTM 48N)
districts = districts.to_crs(epsg=32648)
# Thêm cột diện tích 
districts['area_km2'] = districts.geometry.area/1e6  # Chuyển từ m2 sang km2
districts.head(2)
```

- **Đọc dữ liệu từ `url`**

GeoPandas cho phép đọc dữ liệu trực tiếp từ URL mà không cần tải về máy trước. Điều này rất tiện lợi khi làm việc với các data repositories công khai như GADM (Global Administrative Areas), Natural Earth, hoặc các API GIS. Dữ liệu được stream và parse trực tiếp vào GeoDataFrame. Phương pháp này giúp tiết kiệm không gian lưu trữ và đảm bảo bạn luôn làm việc với phiên bản dữ liệu mới nhất từ nguồn.


```python
# Đọc dữ liệu từ url 
vinhphuc_districts = gpd.read_file('https://raw.githubusercontent.com/tuyenhavan/geodata/refs/heads/main/vector/vinhphuc_districts.geojson')
vinhphuc_districts.head(2)
```

### 15.3.2. Viết dữ liệu

Sau khi xử lý và phân tích dữ liệu, bạn cần lưu kết quả ra file để chia sẻ hoặc sử dụng trong các công cụ GIS khác. GeoPandas sử dụng phương thức `to_file()` để ghi GeoDataFrame ra nhiều định dạng khác nhau. Bạn chỉ cần chỉ định đường dẫn output và driver (định dạng) mong muốn. Mỗi định dạng có ưu nhược điểm riêng: GeoJSON tốt cho web, Shapefile phổ biến trong desktop GIS, Parquet hiệu quả cho big data, GeoPackage là standard mới của OGC.

- **Lưu dữ liệu ra file `GeoJSON`**

GeoJSON là định dạng text-based, dễ đọc và được sử dụng rộng rãi trong web mapping và JavaScript libraries (Leaflet, Mapbox, Deck.gl). File GeoJSON có thể mở trực tiếp trong text editor để xem cấu trúc dữ liệu. Đây là định dạng lý tưởng khi cần chia sẻ dữ liệu qua web APIs hoặc tích hợp với web applications. Tuy nhiên, file size có thể lớn hơn các định dạng binary như Shapefile hay Parquet.


```python
# Lưu dữ liệu ra file GeoJSON
vinhphuc_districts.to_file(os.path.join(outpath, 'vinhphuc_districts.geojson'), driver='GeoJSON')
```

- **Lưu dữ liệu ra `shapefile`**

Shapefile là định dạng vector cổ điển và phổ biến nhất trong GIS, được hỗ trợ bởi hầu hết các phần mềm GIS như ArcGIS, QGIS. Lưu ý rằng một Shapefile thực chất gồm nhiều files (.shp, .shx, .dbf, .prj...) nên cần giữ chúng cùng nhau. Shapefile có giới hạn về độ dài tên trường (10 ký tự) và file size (2GB), nhưng vẫn là lựa chọn tốt cho tương thích với legacy systems và desktop GIS tools.


```python
# Lưu dữ liệu ra file Shapefile
vinhphuc_districts.to_file(os.path.join(outpath, 'vinhphuc_districts.shp'), driver='ESRI Shapefile')
```

- **Lưu dữ liệu ra `Parquet` file**

Parquet là định dạng columnar storage hiệu quả cao, được thiết kế cho big data analytics. File Parquet có kích thước nhỏ hơn nhiều so với GeoJSON hay Shapefile nhờ compression tốt, đồng thời đọc/ghi nhanh hơn đáng kể. Đây là lựa chọn lý tưởng khi làm việc với datasets lớn (hàng triệu features), trong data pipelines, hoặc khi cần tích hợp với các công cụ big data như Apache Spark, Dask. Tuy nhiên, Parquet ít được hỗ trợ trong traditional GIS software.


```python
# Lưu dữ liệu ra file Parquet
# vinhphuc_districts.to_file(os.path.join(outpath, 'vinhphuc_districts.parquet'), driver='Parquet')
```

- **Lưu dữ liệu ra `GeoPackage`**

GeoPackage (GPKG) là định dạng container dạng SQLite database, được OGC (Open Geospatial Consortium) công nhận là standard mới thay thế Shapefile. GPKG lưu toàn bộ dữ liệu trong một file duy nhất, không giới hạn file size hay độ dài tên trường, hỗ trợ nhiều layers trong cùng một file, và có thể chứa cả vector lẫn raster data. Đây là định dạng được khuyến nghị cho các projects GIS hiện đại và được hỗ trợ tốt bởi QGIS, ArcGIS Pro.


```python
# Lưu dữ liệu ra file GeoPackage
vinhphuc_districts.to_file(os.path.join(outpath, 'vinhphuc_districts.gpkg'), driver='GPKG')
```

## 15.4. Thao tác và Phân tích Không gian

Geopandas cho phép người dùng có thể dễ dàng thao tác với dữ liệu địa lý bằng cách sử dụng các phương thức và thuộc tính của GeoDataFrame. Bạn có thể chọn các cột cụ thể, đổi tên cột, lọc dữ liệu dựa trên điều kiện, và thực hiện nhiều thao tác địa lý khác.


```python
# Đọc dữ liệu từ url 
districts = gpd.read_file('https://raw.githubusercontent.com/tuyenhavan/geodata/refs/heads/main/vector/vinhphuc_districts.geojson')

districts.head(2)
```

### 15.4.1. Tạo buffer

Trước khi tạo buffer, chúng ta nên chuyển dữ liệu qua hệ tọa độ UTM cho độ chính xác cao hơn.


```python
# Chuyển hệ tọa độ từ WGS84 (EPSG:4326) sang UTM 48N (EPSG:32648)
districts = districts.to_crs(epsg=32648)
# Tạo 1km buffer quanh tỉnh vĩnh phúc
districts_buffer = districts.buffer(1000)  # Buffer 1000 mét (1 km)
# Chuyển từ GeoSeries sang GeoDataFrame để dễ dàng xử lý
districts_buffer = gpd.GeoDataFrame(geometry=districts_buffer, crs='EPSG:32648')
districts_buffer.head(2)
```

### 15.4.2. Sử dụng phép join giữa hai `GeoDataFrame`

Spatial join là phép toán kết hợp hai GeoDataFrames dựa trên mối quan hệ không gian giữa các geometries, không phải dựa vào key chung như database join thông thường. `gpd.sjoin()` (spatial join) cho phép bạn tìm các features từ GeoDataFrame này mà có quan hệ không gian với features từ GeoDataFrame kia. Các predicates phổ biến: `intersects` (giao nhau), `within` (nằm trong), `contains` (chứa), `touches` (chạm). Ví dụ, tìm tất cả các huyện nằm trong một tỉnh bằng cách join districts với province sử dụng predicate `intersects`.


```python
# Đảm bảo dữ liệu vinhphuc có cùng hệ tọa độ với districts trước khi join
vinhphuc = gpd.read_file('https://raw.githubusercontent.com/tuyenhavan/geodata/refs/heads/main/vector/vinhphuc_province.geojson').to_crs(districts.crs)
# Join dữ liệu tỉnh vĩnh phúc với dữ liệu districts để lấy ra các huyện thuộc tỉnh vĩnh phúc
vinhphuc_districts = gpd.sjoin(vinhphuc, districts, how='inner', predicate='intersects') # ngoài intersects còn có within, contains, touches, crosses, covers, covered_by.
vinhphuc_districts.head(2)
```

## Tóm tắt

Bạn đã hoàn thành Bài 15 và học được GeoPandas - thư viện "con dao Thụy Sĩ" cho geospatial data analysis trong Python.

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
