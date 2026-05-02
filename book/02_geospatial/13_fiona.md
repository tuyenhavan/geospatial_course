# Bài 13: Đọc và ghi dữ liệu vector (Fiona)

Fiona cung cấp giao diện Python đơn giản, đáng tin cậy và hiệu quả để làm việc với các tệp dữ liệu không gian địa lý. Được xây dựng trên nền tảng OGR (một phần của GDAL), Fiona tập trung hoàn toàn vào việc xử lý dữ liệu vector và là thư viện I/O nền tảng cho GeoPandas.

## 13.1. Mục tiêu học tập
Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Đọc và ghi các định dạng file vector địa không gian phổ biến (Shapefile, GeoJSON)
- Làm việc hiệu quả với collections, features và attributes
- Xử lý hệ tọa độ tham chiếu (CRS) trong quá trình I/O dữ liệu
- Thực hiện lọc không gian và truy vấn dữ liệu theo điều kiện
- Xử lý encoding và các vấn đề về định dạng dữ liệu

- **Import thư viện cần thiết**


```python
# Import các thư viện cần thiết
import fiona
from fiona.crs import from_epsg
from shapely.geometry import Point, mapping, shape
import warnings
warnings.filterwarnings('ignore')

print(f"📦 Fiona version: {fiona.__version__}")

# Kiểm tra các driver có sẵn trong Fiona
supported_drivers = fiona.supported_drivers
print(f"Có {len(supported_drivers)} driver được hỗ trợ trong Fiona.")
```

    📦 Fiona version: 1.10.1
    Có 20 driver được hỗ trợ trong Fiona.
    

## 13.2. Viết và đọc file dữ liệu vector với `fiona`
Mở và khám phá các định dạng file vector khác nhau.

### 13.2.1. Viết dữ liệu vào file

- **Tạo dữ liệu minh họa**


```python
# Tạo dữ liệu mẫu về các tỉnh/thành phố Việt Nam. Dữ liệu này mang tính minh họa và nên kiểm tra lại trước khi sử dụng cho mục đích chính thức.
vietnam_cities_data = [
    {
        'name': 'Hà Nội',
        'type': 'Thành phố trực thuộc TW',
        'population': 8435700,
        'area_km2': 3358.59,
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


```python
# 1. Tạo GeoJSON file
geojson_file = r"G:\My Drive\python\geocourse\data\vector\vietnam_cities.geojson"

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


```python
# Đọc file GeoJSON
with fiona.open(geojson_file, 'r') as src:
    print(f"📄 Tên file: {src.name}")
    print(f"🔧 Driver: {src.driver}")
    print(f"🗺️ CRS: {src.crs}")
    print(f"📊 Số lượng features: {len(src)}")
    print(f"📏 Bounds: {src.bounds}")
    
    # Schema (cấu trúc dữ liệu)
    print(f"\n📋 SCHEMA:")
    print(f"  Geometry type: {src.schema['geometry']}")
    print(f"  Properties:")
    for prop, dtype in src.schema['properties'].items():
        print(f"    {prop}: {dtype}")
    
    print(f"\n🔍 MẪU DỮ LIỆU (1 FEATURE ĐẦU):")
    for i, feature in enumerate(src):
        if i < 1:
            props = feature['properties']
            geom = feature['geometry']
            coords = geom['coordinates']
            print(f"\n  Feature {i+1}:")
            print(f"    Tên: {props['name']}")
            print(f"    Loại: {props['type']}")
            print(f"    Dân số: {props['population']:,}")
            print(f"    Tọa độ: ({coords[1]:.4f}, {coords[0]:.4f})")
            print(f"    Mật độ: {props['density']} người/km²")
        else:
            break
```

## 13.3. Lọc thông tin và chuyển đổi format

Đọc hiệu quả tập con dữ liệu dựa trên tiêu chí không gian và thuộc tính.

### 13.3.1. Lọc thông tin


```python
# Tìm các thành phố có dân số > 5 triệu
large_cities = []
with fiona.open(geojson_file, 'r') as src:
    for feature in src:
        props = feature['properties']
        if props['population'] > 5000000:
            large_cities.append(props['name'])
            print(f"   🏙️  {props['name']}: {props['population']:,} người")

print(f"   → Tìm thấy {len(large_cities)} thành phố lớn")
```

### 13.3.2. Đổi từ `geojson` sang `shapefile`


```python
outfile = r"G:\My Drive\python\geocourse\data\vector\vietnam_cities.shp"
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
