# Bài 13: Đọc và ghi dữ liệu với Fiona

Fiona cung cấp giao diện Python đơn giản, đáng tin cậy và hiệu quả để làm việc với các tệp dữ liệu không gian địa lý. Được xây dựng trên nền tảng OGR (một phần của GDAL), Fiona tập trung hoàn toàn vào việc xử lý dữ liệu vector và là thư viện I/O nền tảng cho GeoPandas.

> **Lưu Ý**
> 
> Bạn có thể chạy trực tiếp notebook này bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/10AskmkiK2lrTNQCCDXs1uDk39Llytn21) mà không cần cài đặt Python. Để tránh làm thay đổi nội dung gốc và thuận tiện cho việc lưu kết quả, hãy tạo một bản sao ( File → Save a copy in Drive ) trước khi chạy và chỉnh sửa mã nguồn trong notebook.

## 13.1. Mục tiêu học tập
Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Đọc và ghi các định dạng file vector địa không gian phổ biến (Shapefile, GeoJSON)
- Làm việc hiệu quả với collections, features và attributes
- Xử lý hệ tọa độ tham chiếu (CRS) trong quá trình I/O dữ liệu
- Thực hiện lọc không gian và truy vấn dữ liệu theo điều kiện
- Xử lý encoding và các vấn đề về định dạng dữ liệu


```python
# nếu chưa có các thư viện sau thì cài đặt bằng pip install fiona shapely hoặc xem lại Bài 1.
import fiona
from fiona.crs import from_epsg
from shapely.geometry import Point, mapping, shape
```

## 13.2. Viết và đọc dữ liệu vector với `fiona`
Mở và khám phá các định dạng file vector khác nhau.

### 13.2.1. Viết dữ liệu vào file

Fiona cung cấp khả năng ghi dữ liệu vector vào các định dạng file khác nhau như GeoJSON, Shapefile, KML, GPKG. Quá trình ghi dữ liệu yêu cầu bạn định nghĩa schema (cấu trúc dữ liệu) bao gồm loại hình học (geometry type) và các thuộc tính (properties), cùng với hệ tọa độ (CRS). Fiona sử dụng context manager (`with` statement) để đảm bảo file được đóng đúng cách sau khi ghi.

- **Tạo dữ liệu minh họa**

Trước khi ghi dữ liệu vào file, chúng ta cần chuẩn bị dữ liệu dưới dạng cấu trúc Python. Trong ví dụ này, chúng ta tạo một danh sách các dictionary chứa thông tin về các tỉnh/thành phố Việt Nam, bao gồm tên, loại hình, dân số, diện tích, khu vực và tọa độ địa lý. Dữ liệu này sẽ được chuyển đổi thành các features không gian địa lý trong bước tiếp theo.


```python
# Tạo dữ liệu mẫu về các tỉnh/thành phố Việt Nam. Dữ liệu này mang tính minh họa và nên kiểm tra lại trước khi sử dụng cho mục đích chính thức.
vietnam_cities_data = [
    {
        'name': 'Hà Nội',
        'type': 'Thành phố trực thuộc TW',
        'population': 8860000,
        'area_km2': 3359.59,
        'region': 'Miền Bắc',
        'coordinates': (21.0285, 105.8542),
    },
    {
        'name': 'TP. Hồ Chí Minh', 
        'type': 'Thành phố trực thuộc TW',
        'population': 9077158,
        'area_km2': 2061.45,
        'region': 'Miền Nam', 
        'coordinates': (10.8231, 106.6297),
    },
    {
        'name': 'Đà Nẵng',
        'type': 'Thành phố trực thuộc TW',
        'population': 1230000,
        'area_km2': 1285.53,
        'region': 'Miền Trung',
        'coordinates': (16.0471, 108.2068),
    },
    {
        'name': 'Cần Thơ',
        'type': 'Thành phố trực thuộc TW',
        'population': 1235171,
        'area_km2': 1408.93,
        'region': 'Miền Nam',
        'coordinates': (10.0452, 105.7469),
    },
    {
        'name': 'Hải Phòng',
        'type': 'Thành phố trực thuộc TW',
        'population': 2028514,
        'area_km2': 1561.47,
        'region': 'Miền Bắc',
        'coordinates': (20.8449, 106.6881),
    },
    {
        'name': 'An Giang',
        'type': 'Tỉnh',
        'population': 1908900,
        'area_km2': 3536.67,
        'region': 'Miền Nam',
        'coordinates': (10.5111, 105.1268),
    }
]
```

- **Viết dữ liệu vào file**

Để viết dữ liệu địa lý vào một file GeoJSON, bạn cần xác định schema cho dữ liệu của mình, bao gồm loại hình học (geometry) và các thuộc tính (properties) mà bạn muốn lưu trữ. Sau đó, bạn có thể sử dụng thư viện Fiona để tạo file GeoJSON và ghi các feature vào đó. Trong ví dụ bên dưới, chúng ta đã tạo một file GeoJSON chứa thông tin về các tỉnh/thành phố ở Việt Nam, bao gồm tên, loại hình, dân số, diện tích, khu vực và mật độ dân số.


```python
# 1. Tạo GeoJSON file
geojson_file = r"J:\My Drive\geocourse_data\outputs\vietnam_cities.geojson"

# Schema cho GeoJSON
schema = {
    'geometry': 'Point',
    'properties': {
        'name': 'str:50',
        'type': 'str:30', 
        'population': 'int',
        'area_km2': 'float',
        'region': 'str:20',
        'density': 'float'
    }
}

# CRS cho Việt Nam (WGS84)
crs = from_epsg(4326)

# Tạo GeoJSON file, bạn có thể thay đổi sang shapefile hoặc các định dạng khác bằng cách thay đổi driver
with fiona.open(geojson_file, 'w', driver='GeoJSON', schema=schema, crs=crs) as output:
    for city_data in vietnam_cities_data:
        # Tạo Point geometry
        point = Point(city_data['coordinates'][::-1])  # lon, lat
        
        # Tính mật độ dân số
        density = city_data['population'] / city_data['area_km2']
        
        # Tạo feature
        feature = {
            'geometry': mapping(point), # Chuyển đổi Point thành GeoJSON geometry
            'properties': {
                'name': city_data['name'],
                'type': city_data['type'],
                'population': city_data['population'],
                'area_km2': city_data['area_km2'],
                'region': city_data['region'],
                'density': round(density, 2)
            }
        }
        
        output.write(feature)
```

### 13.2.2. Đọc dữ liệu từ file `geojson`

Fiona cho phép người dùng đọc file GeoJSON và truy cập vào các thuộc tính và hình học của từng feature trong file. Bạn có thể kiểm tra schema để biết được các thuộc tính nào có trong dữ liệu và kiểu dữ liệu của chúng, sau đó truy cập vào các thuộc tính và hình học của từng feature để sử dụng trong phân tích hoặc trực quan hóa.


```python
# Đọc file GeoJSON
geojson_file = r"J:\My Drive\geocourse_data\outputs\vietnam_cities.geojson"

with fiona.open(geojson_file, 'r') as src:
    for prop, dtype in src.schema['properties'].items():
        print(f"{prop}: {dtype}") 
    for i, feature in enumerate(src):
        if i < 1:
            props = feature['properties'] 
            geom = feature['geometry']
            coords = geom['coordinates']
        else:
            break
```

    name: str
    type: str
    population: int32
    area_km2: float
    region: str
    density: float
    

## 13.3. Lọc thông tin và chuyển đổi format

Đọc hiệu quả tập con dữ liệu dựa trên tiêu chí không gian và thuộc tính.

### 13.3.1. Lọc thông tin

Fiona cho phép bạn duyệt qua các features và lọc chúng dựa trên các thuộc tính (attributes). Thay vì đọc toàn bộ dữ liệu vào bộ nhớ, bạn có thể xử lý từng feature một và chỉ giữ lại những feature thỏa mãn điều kiện. Điều này rất hiệu quả khi làm việc với datasets lớn, giúp tiết kiệm bộ nhớ và tăng tốc độ xử lý.


```python
# Tìm các thành phố có dân số > 5 triệu
large_cities = []
with fiona.open(geojson_file, 'r') as src:
    for feature in src:
        props = feature['properties']
        if props['population'] > 5000000:
            large_cities.append(props['name'])
            print(f"🏙️ {props['name']}: {props['population']:,} người")

print(f"→ Tìm thấy {len(large_cities)} thành phố lớn")
```

    🏙️ Hà Nội: 8,435,700 người
    🏙️ TP. Hồ Chí Minh: 9,077,158 người
    → Tìm thấy 2 thành phố lớn
    

### 13.3.2. Đổi từ `geojson` sang `shapefile`

Fiona cho phép bạn dễ dàng chuyển đổi giữa các định dạng dữ liệu địa lý khác nhau, chẳng hạn như từ GeoJSON sang Shapefile, giúp bạn linh hoạt trong việc sử dụng dữ liệu địa lý cho các mục đích khác nhau trong GIS.


```python
# Bạn nên thay đổi đường dẫn và tên file theo yêu cầu của bạn.
outfile = r"J:\My Drive\geocourse_data\outputs\vietnam_cities.shp"
with fiona.open(geojson_file, 'r') as src:
    # Đọc tất cả features vào bộ nhớ
    features = list(src)
    # Viết ra shapefile mới
    with fiona.open(outfile, 'w', driver='ESRI Shapefile', schema=src.schema, crs=src.crs) as dst:
        for feature in features:
            dst.write(feature)
```

## Tóm tắt

Bạn đã hoàn thành Bài 3 và học được Fiona - thư viện chuyên nghiệp cho vector data I/O trong Python GIS ecosystem.

### Các khái niệm chính đã nắm vững:
- ✅ **Vector data I/O**: Đọc và ghi Shapefile, GeoJSON, KML, GPKG với fiona.open()
- ✅ **Collections và Features**: Làm việc với feature collections và individual features
- ✅ **Schema management**: Properties, geometry types và data validation
- ✅ **CRS handling**: Coordinate reference systems trong I/O operations
- ✅ **Format conversions**: Chuyển đổi giữa các vector formats khác nhau
- ✅ **Integration với Shapely**: Seamless geometry conversion với shape() và mapping()

### Kỹ năng bạn có thể áp dụng:
- Xử lý và chuyển đổi dữ liệu vector từ nhiều nguồn khác nhau
- Thực hiện spatial queries và filtering hiệu quả trên large datasets
- Tích hợp Fiona với Shapely và GeoPandas trong geospatial workflows
- Validate và clean vector data với proper schema management
- Chuẩn bị data pipeline foundations cho advanced GIS analysis và visualization
