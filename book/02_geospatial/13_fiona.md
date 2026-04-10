# Bài 3: Fiona - Đọc và ghi dữ liệu vector không gian địa lý

Fiona cung cấp giao diện Python đơn giản, đáng tin cậy và hiệu quả để làm việc với các tệp dữ liệu không gian địa lý. Được xây dựng trên nền tảng OGR (một phần của GDAL), Fiona tập trung hoàn toàn vào việc xử lý dữ liệu vector và là thư viện I/O nền tảng cho GeoPandas.

## Mục tiêu học tập
Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Đọc và ghi các định dạng file vector địa không gian phổ biến (Shapefile, GeoJSON, KML, GPKG)
- Làm việc hiệu quả với collections, features và attributes
- Xử lý hệ tọa độ tham chiếu (CRS) trong quá trình I/O dữ liệu
- Thực hiện lọc không gian và truy vấn dữ liệu theo điều kiện
- Hiểu mối quan hệ và tích hợp giữa Fiona với GeoPandas và Shapely
- Tối ưu hóa hiệu suất khi xử lý file dữ liệu lớn
- Xử lý encoding và các vấn đề về định dạng dữ liệu

- **Import thư viện cần thiết**


```python
# Import các thư viện cần thiết
import fiona
from fiona import collection, transform
from fiona.crs import from_epsg, from_string
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from shapely.geometry import Point, LineString, Polygon, mapping, shape
import json
import tempfile
import os
from pathlib import Path
import warnings
warnings.filterwarnings('ignore')

print(f"📦 Fiona version: {fiona.__version__}")

# Kiểm tra các driver có sẵn trong Fiona
supported_drivers = fiona.supported_drivers
print(f"Có {len(supported_drivers)} driver được hỗ trợ trong Fiona.")
```

    📦 Fiona version: 1.10.1
    Có 20 driver được hỗ trợ trong Fiona.
    

## 1. Đọc file dữ liệu vector (Reading Vector Data Files)

Mở và khám phá các định dạng file vector khác nhau.


```python
# Tạo dữ liệu mẫu về các tỉnh/thành phố Việt Nam
vietnam_cities_data = [
    {
        'name': 'Hà Nội',
        'type': 'Thành phố trực thuộc TW',
        'population': 8435700,
        'area_km2': 3358.59,
        'region': 'Miền Bắc',
        'coordinates': (21.0285, 105.8542),
        'established': 1010
    },
    {
        'name': 'TP. Hồ Chí Minh', 
        'type': 'Thành phố trực thuộc TW',
        'population': 9077158,
        'area_km2': 2061.45,
        'region': 'Miền Nam', 
        'coordinates': (10.8231, 106.6297),
        'established': 1698
    },
    {
        'name': 'Đà Nẵng',
        'type': 'Thành phố trực thuộc TW',
        'population': 1230000,
        'area_km2': 1285.53,
        'region': 'Miền Trung',
        'coordinates': (16.0471, 108.2068),
        'established': 1888
    },
    {
        'name': 'Cần Thơ',
        'type': 'Thành phố trực thuộc TW',
        'population': 1235171,
        'area_km2': 1408.93,
        'region': 'Miền Nam',
        'coordinates': (10.0452, 105.7469),
        'established': 1876
    },
    {
        'name': 'Hải Phòng',
        'type': 'Thành phố trực thuộc TW',
        'population': 2028514,
        'area_km2': 1561.47,
        'region': 'Miền Bắc',
        'coordinates': (20.8449, 106.6881),
        'established': 1888
    },
    {
        'name': 'An Giang',
        'type': 'Tỉnh',
        'population': 1908900,
        'area_km2': 3536.67,
        'region': 'Miền Nam',
        'coordinates': (10.5111, 105.1268),
        'established': 1832
    }
]
```


```python
# 1. Tạo GeoJSON file
geojson_file = r"G:\My Drive\python\python_course\book\data\vietnam_cities.geojson"

# Schema cho GeoJSON
schema = {
    'geometry': 'Point',
    'properties': {
        'name': 'str:50',
        'type': 'str:30', 
        'population': 'int',
        'area_km2': 'float',
        'region': 'str:20',
        'established': 'int',
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
            'geometry': mapping(point),
            'properties': {
                'name': city_data['name'],
                'type': city_data['type'],
                'population': city_data['population'],
                'area_km2': city_data['area_km2'],
                'region': city_data['region'],
                'established': city_data['established'],
                'density': round(density, 2)
            }
        }
        
        output.write(feature)
```


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
    
    print(f"\n🔍 MẪU DỮ LIỆU (3 FEATURE ĐẦU):")
    for i, feature in enumerate(src):
        if i < 3:
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

    📄 Tên file: vietnam_cities
    🔧 Driver: GeoJSON
    🗺️ CRS: EPSG:4326
    📊 Số lượng features: 6
    📏 Bounds: (105.1268, 10.0452, 108.2068, 21.0285)
    
    📋 SCHEMA:
      Geometry type: Point
      Properties:
        name: str
        type: str
        population: int32
        area_km2: float
        region: str
        established: int32
        density: float
    
    🔍 MẪU DỮ LIỆU (3 FEATURE ĐẦU):
    
      Feature 1:
        Tên: Hà Nội
        Loại: Thành phố trực thuộc TW
        Dân số: 8,435,700
        Tọa độ: (21.0285, 105.8542)
        Mật độ: 2511.68 người/km²
    
      Feature 2:
        Tên: TP. Hồ Chí Minh
        Loại: Thành phố trực thuộc TW
        Dân số: 9,077,158
        Tọa độ: (10.8231, 106.6297)
        Mật độ: 4403.29 người/km²
    
      Feature 3:
        Tên: Đà Nẵng
        Loại: Thành phố trực thuộc TW
        Dân số: 1,230,000
        Tọa độ: (16.0471, 108.2068)
        Mật độ: 956.8 người/km²
    


```python
# Khám phá metadata và thuộc tính file
print("=== PHÂN TÍCH METADATA CHI TIẾT ===")

def analyze_vector_file(file_path, file_type):
    """Hàm phân tích chi tiết file vector"""
    print(f"\n🔍 PHÂN TÍCH {file_type.upper()}:")
    
    with fiona.open(file_path, 'r') as src:
        # Thông tin cơ bản
        print(f"  📁 Đường dẫn: {src.name}")
        print(f"  🔧 Driver: {src.driver}")
        print(f"  🌍 CRS: {src.crs}")
        print(f"  📊 Số features: {len(src)}")
        
        # Bounds (giới hạn không gian)
        bounds = src.bounds
        print(f"  📐 Bounds:")
        print(f"    Tây (minx): {bounds[0]:.6f}°")
        print(f"    Nam (miny): {bounds[1]:.6f}°") 
        print(f"    Đông (maxx): {bounds[2]:.6f}°")
        print(f"    Bắc (maxy): {bounds[3]:.6f}°")
        
        # Phạm vi địa lý
        width = bounds[2] - bounds[0]
        height = bounds[3] - bounds[1]
        print(f"    Chiều rộng: {width:.4f}° ({width*111:.1f} km)")
        print(f"    Chiều cao: {height:.4f}° ({height*111:.1f} km)")
        
        # Thống kê thuộc tính
        print(f"  📈 THỐNG KÊ THUỘC TÍNH:")
        populations = []
        areas = []
        densities = []
        
        for feature in src:
            props = feature['properties']
            populations.append(props['population'])
            areas.append(props['area_km2'])
            densities.append(props['density'])
        
        print(f"    Dân số - Min: {min(populations):,}, Max: {max(populations):,}, TB: {np.mean(populations):,.0f}")
        print(f"    Diện tích - Min: {min(areas):.1f}, Max: {max(areas):.1f}, TB: {np.mean(areas):.1f} km²")
        print(f"    Mật độ - Min: {min(densities):.1f}, Max: {max(densities):.1f}, TB: {np.mean(densities):.1f} người/km²")

# Phân tích cả hai file
analyze_vector_file(geojson_file, "GeoJSON")
analyze_vector_file(shapefile_path, "Shapefile")

# So sánh kích thước file
json_size = geojson_file.stat().st_size
shp_size = sum(f.stat().st_size for f in temp_dir.glob("vietnam_cities.*"))

print(f"\n💾 SO SÁNH KÍCH THƯỚC FILE:")
print(f"  GeoJSON: {json_size:,} bytes ({json_size/1024:.1f} KB)")
print(f"  Shapefile (tất cả): {shp_size:,} bytes ({shp_size/1024:.1f} KB)")
print(f"  Tỷ lệ: Shapefile {'lớn' if shp_size > json_size else 'nhỏ'} hơn {abs(shp_size-json_size)/min(shp_size,json_size)*100:.1f}%")
```

## 2. Làm việc với Features và Attributes (Working with Features and Attributes)

Hiểu về mô hình feature-attribute trong dữ liệu vector.


```python
# ==========================================
# THAO TÁC VỚI FEATURES VÀ ATTRIBUTES
# ==========================================

print("=== THAO TÁC VỚI FEATURES VÀ ATTRIBUTES ===")

# Đọc và xử lý từng feature
with fiona.open(geojson_file, 'r') as src:
    print(f"🔍 XỬ LÝ TỪNG FEATURE:")
    
    for i, feature in enumerate(src):
        # Feature structure
        geometry = feature['geometry']  # Phần hình học
        properties = feature['properties']  # Phần thuộc tính
        
        print(f"\n  📍 Feature {i+1}: {properties['name']}")
        print(f"    Geometry type: {geometry['type']}")
        print(f"    Coordinates: {geometry['coordinates']}")
        print(f"    Properties: {len(properties)} thuộc tính")
        
        # Truy cập các thuộc tính
        print(f"    Dân số: {properties['population']:,} người")
        print(f"    Diện tích: {properties['area_km2']:,.2f} km²")
        print(f"    Loại: {properties['type']}")
        print(f"    Khu vực: {properties['region']}")
        
        # Tính toán từ thuộc tính
        if properties['area_km2'] > 0:
            density = properties['population'] / properties['area_km2']
            print(f"    Mật độ tính toán: {density:.2f} người/km²")
        
        # Chỉ hiển thị 3 feature đầu
        if i >= 2:
            break

print(f"\n=== LỌC VÀ TÌM KIẾM FEATURES ===")

# 1. Lọc theo thuộc tính
def filter_by_region(file_path, target_region):
    """Lọc các thành phố theo khu vực"""
    results = []
    with fiona.open(file_path, 'r') as src:
        for feature in src:
            if feature['properties']['region'] == target_region:
                results.append(feature)
    return results

# Lọc thành phố miền Bắc
mien_bac_cities = filter_by_region(geojson_file, 'Miền Bắc')
print(f"🏛️ Thành phố miền Bắc: {len(mien_bac_cities)} thành phố")
for city in mien_bac_cities:
    name = city['properties']['name']
    pop = city['properties']['population']
    print(f"  - {name}: {pop:,} dân")

# 2. Lọc theo dân số
def filter_by_population(file_path, min_population):
    """Lọc các thành phố có dân số trên ngưỡng"""
    results = []
    with fiona.open(file_path, 'r') as src:
        for feature in src:
            if feature['properties']['population'] >= min_population:
                results.append(feature)
    return results

large_cities = filter_by_population(geojson_file, 2000000)
print(f"\n🏙️ Thành phố lớn (>2M dân): {len(large_cities)} thành phố")
for city in large_cities:
    name = city['properties']['name'] 
    pop = city['properties']['population']
    area = city['properties']['area_km2']
    print(f"  - {name}: {pop:,} dân, {area:,.1f} km²")

# 3. Sắp xếp theo thuộc tính
print(f"\n📊 SẮP XẾP THEO DÂN SỐ (giảm dần):")
all_features = []
with fiona.open(geojson_file, 'r') as src:
    all_features = list(src)

# Sắp xếp theo dân số
sorted_by_pop = sorted(all_features, 
                      key=lambda x: x['properties']['population'], 
                      reverse=True)

for i, feature in enumerate(sorted_by_pop[:5]):  # Top 5
    props = feature['properties']
    print(f"  {i+1}. {props['name']}: {props['population']:,} dân")
```


```python
# ==========================================
# THỐNG KÊ VÀ PHÂN TÍCH ATTRIBUTES
# ==========================================

print("=== THỐNG KÊ VÀ PHÂN TÍCH CHI TIẾT ===")

# Đọc tất cả dữ liệu vào DataFrame để phân tích
features_data = []
with fiona.open(geojson_file, 'r') as src:
    for feature in src:
        props = feature['properties'].copy()
        geom = feature['geometry']
        
        # Thêm tọa độ
        props['latitude'] = geom['coordinates'][1]
        props['longitude'] = geom['coordinates'][0]
        
        features_data.append(props)

df = pd.DataFrame(features_data)
print(f"📊 ĐÃ CHUYỂN ĐỔI THÀNH DATAFRAME:")
print(df.head())

# Thống kê mô tả
print(f"\n📈 THỐNG KÊ MÔ TẢ:")
numeric_stats = df[['population', 'area_km2', 'density']].describe()
print(numeric_stats)

# Phân tích theo khu vực
print(f"\n🗺️ PHÂN TÍCH THEO KHU VỰC:")
region_stats = df.groupby('region').agg({
    'population': ['count', 'sum', 'mean'],
    'area_km2': ['sum', 'mean'],
    'density': ['mean', 'max', 'min']
}).round(2)

print(region_stats)

# Tìm các giá trị đặc biệt
print(f"\n🏆 CÁC GIÁ TRỊ ĐẶC BIỆT:")

# Thành phố đông dân nhất
max_pop_idx = df['population'].idxmax()
max_pop_city = df.loc[max_pop_idx]
print(f"  Đông dân nhất: {max_pop_city['name']} ({max_pop_city['population']:,} dân)")

# Thành phố rộng nhất
max_area_idx = df['area_km2'].idxmax()
max_area_city = df.loc[max_area_idx]
print(f"  Diện tích lớn nhất: {max_area_city['name']} ({max_area_city['area_km2']:,.1f} km²)")

# Mật độ cao nhất
max_density_idx = df['density'].idxmax()
max_density_city = df.loc[max_density_idx]
print(f"  Mật độ cao nhất: {max_density_city['name']} ({max_density_city['density']:,.0f} người/km²)")

# Thành phố cổ nhất
min_established_idx = df['established'].idxmin()
oldest_city = df.loc[min_established_idx]
print(f"  Thành lập sớm nhất: {oldest_city['name']} (năm {oldest_city['established']})")

# Tương quan giữa các biến
print(f"\n🔗 TƯƠNG QUAN GIỮA CÁC BIẾN:")
correlation_matrix = df[['population', 'area_km2', 'density', 'established']].corr()
print(correlation_matrix.round(3))
```


```python
# Trực quan hóa dữ liệu features và attributes
fig, axes = plt.subplots(2, 2, figsize=(15, 12))

# Biểu đồ 1: Bản đồ phân bố địa lý
ax1 = axes[0, 0]
colors = {'Miền Bắc': 'red', 'Miền Trung': 'blue', 'Miền Nam': 'green'}
for region in df['region'].unique():
    region_data = df[df['region'] == region]
    ax1.scatter(region_data['longitude'], region_data['latitude'], 
               c=colors[region], s=region_data['population']/50000,
               alpha=0.7, label=region)

for _, city in df.iterrows():
    ax1.annotate(city['name'], 
                (city['longitude'], city['latitude']),
                xytext=(2, 2), textcoords='offset points', fontsize=8)

ax1.set_xlabel('Kinh độ (°)')
ax1.set_ylabel('Vĩ độ (°)')
ax1.set_title('Phân bố địa lý các tỉnh/thành phố')
ax1.legend()
ax1.grid(True, alpha=0.3)

# Biểu đồ 2: Dân số theo khu vực
ax2 = axes[0, 1]
region_pop = df.groupby('region')['population'].sum()
bars = ax2.bar(region_pop.index, region_pop.values, 
               color=[colors[region] for region in region_pop.index])
ax2.set_title('Tổng dân số theo khu vực')
ax2.set_ylabel('Dân số')
ax2.tick_params(axis='x', rotation=45)

# Thêm nhãn trên cột
for bar, value in zip(bars, region_pop.values):
    height = bar.get_height()
    ax2.text(bar.get_x() + bar.get_width()/2., height + height*0.01,
             f'{value/1e6:.1f}M', ha='center', va='bottom')

# Biểu đồ 3: Mối quan hệ diện tích - dân số
ax3 = axes[1, 0]
scatter = ax3.scatter(df['area_km2'], df['population'], 
                     c=df['density'], cmap='viridis', 
                     s=100, alpha=0.7)
ax3.set_xlabel('Diện tích (km²)')
ax3.set_ylabel('Dân số')
ax3.set_title('Mối quan hệ diện tích - dân số')

# Thêm colorbar
cbar = plt.colorbar(scatter, ax=ax3)
cbar.set_label('Mật độ (người/km²)')

# Thêm tên thành phố
for _, city in df.iterrows():
    ax3.annotate(city['name'], 
                (city['area_km2'], city['population']),
                xytext=(3, 3), textcoords='offset points', fontsize=8)

# Biểu đồ 4: Phân bố mật độ dân số
ax4 = axes[1, 1]
ax4.hist(df['density'], bins=8, alpha=0.7, color='orange', edgecolor='black')
ax4.set_xlabel('Mật độ (người/km²)')
ax4.set_ylabel('Số lượng tỉnh/thành phố')
ax4.set_title('Phân bố mật độ dân số')
ax4.grid(True, alpha=0.3)

plt.tight_layout()
plt.show()

print("📊 Trực quan hóa dữ liệu features và attributes hoàn thành!")
```

## 3. Ghi dữ liệu vector (Writing Vector Data)

Tạo và xuất các tập dữ liệu vector sang nhiều định dạng khác nhau.


```python
# ==========================================
# TẠO VÀ GHI DỮ LIỆU VECTOR MỚI
# ==========================================

print("=== TẠO DỮ LIỆU VECTOR MỚI ===")

# 1. Tạo dữ liệu về các con sông lớn của Việt Nam
vietnam_rivers = [
    {
        'name': 'Sông Hồng',
        'length_km': 1200,
        'basin_area': 169000,
        'source': 'Trung Quốc',
        'mouth': 'Biển Đông',
        'provinces': ['Hà Nội', 'Hải Phòng', 'Nam Định'],
        'coordinates': [  # Tọa độ từ nguồn đến cửa
            [105.8542, 21.0285],  # Hà Nội
            [105.9178, 20.9614],  # Đoạn qua Hà Nội
            [106.6881, 20.8449],  # Hải Phòng
            [106.7321, 20.4590]   # Cửa sông
        ]
    },
    {
        'name': 'Sông Mekong',
        'length_km': 4350,
        'basin_area': 795000,
        'source': 'Tây Tạng',
        'mouth': 'Biển Đông',
        'provinces': ['An Giang', 'Cần Thơ', 'TP.HCM'],
        'coordinates': [
            [105.1268, 10.5111],  # An Giang
            [105.7469, 10.0452],  # Cần Thơ
            [106.6297, 10.8231],  # TP.HCM
            [106.8000, 10.4000]   # Cửa sông
        ]
    },
    {
        'name': 'Sông Đà',
        'length_km': 1200,
        'basin_area': 52900,
        'source': 'Trung Quốc',
        'mouth': 'Sông Hồng',
        'provinces': ['Lai Châu', 'Điện Biên', 'Sơn La'],
        'coordinates': [
            [103.2000, 22.3000],  # Lai Châu
            [103.5000, 21.8000],  # Sơn La
            [104.5000, 21.2000],  # Hòa Bình
            [105.8542, 21.0285]   # Hà Nội (nhập sông Hồng)
        ]
    }
]

# Schema cho dữ liệu sông (LineString)
rivers_schema = {
    'geometry': 'LineString',
    'properties': {
        'name': 'str:50',
        'length_km': 'float',
        'basin_area': 'float',
        'source': 'str:30',
        'mouth': 'str:30',
        'province_count': 'int',
        'avg_width_est': 'float'
    }
}

# Tạo file GeoJSON cho sông
rivers_file = temp_dir / "vietnam_rivers.geojson"

with fiona.open(rivers_file, 'w', driver='GeoJSON', schema=rivers_schema, crs=crs) as output:
    for river_data in vietnam_rivers:
        # Tạo LineString geometry
        line = LineString(river_data['coordinates'])
        
        # Ước tính độ rộng trung bình (dựa trên lưu vực)
        avg_width = (river_data['basin_area'] / river_data['length_km']) ** 0.5 / 10
        
        feature = {
            'geometry': mapping(line),
            'properties': {
                'name': river_data['name'],
                'length_km': river_data['length_km'],
                'basin_area': river_data['basin_area'],
                'source': river_data['source'],
                'mouth': river_data['mouth'],
                'province_count': len(river_data['provinces']),
                'avg_width_est': round(avg_width, 1)
            }
        }
        
        output.write(feature)

print(f"✅ Đã tạo file sông: {rivers_file}")
```


```python
# 2. Tạo dữ liệu về các khu bảo tồn (Polygon)
vietnam_parks = [
    {
        'name': 'Vườn Quốc gia Ba Vì',
        'type': 'Vườn Quốc gia',
        'area_km2': 107.58,
        'established': 1991,
        'province': 'Hà Nội',
        'center': [105.3667, 21.1167],
        'radius_km': 6  # Để tạo polygon đơn giản
    },
    {
        'name': 'Vườn Quốc gia Cát Bà',
        'type': 'Vườn Quốc gia',
        'area_km2': 263.0,
        'established': 1986,
        'province': 'Hải Phòng',
        'center': [107.0167, 20.8167],
        'radius_km': 9
    },
    {
        'name': 'Vườn Quốc gia Phong Nha-Kẻ Bàng',
        'type': 'Vườn Quốc gia',
        'area_km2': 857.54,
        'established': 2001,
        'province': 'Quảng Bình',
        'center': [106.2000, 17.5500],
        'radius_km': 16
    },
    {
        'name': 'Khu Bảo tồn U Minh Hạ',
        'type': 'Khu Bảo tồn',
        'area_km2': 82.48,
        'established': 2006,
        'province': 'Cà Mau',
        'center': [105.0500, 9.4500],
        'radius_km': 5
    }
]

# Schema cho khu bảo tồn (Polygon)
parks_schema = {
    'geometry': 'Polygon',
    'properties': {
        'name': 'str:50',
        'type': 'str:20',
        'area_km2': 'float',
        'established': 'int',
        'province': 'str:20',
        'biodiversity_index': 'float',
        'visitor_capacity': 'int'
    }
}

# Tạo file cho khu bảo tồn
parks_file = temp_dir / "vietnam_parks.geojson"

def create_circle_polygon(center_lon, center_lat, radius_km, num_points=20):
    """Tạo polygon hình tròn đơn giản"""
    points = []
    earth_radius = 6371.0  # km
    
    for i in range(num_points + 1):
        angle = 2 * np.pi * i / num_points
        
        # Tính toán offset
        lat_offset = (radius_km / earth_radius) * (180 / np.pi)
        lon_offset = (radius_km / (earth_radius * np.cos(np.pi * center_lat / 180))) * (180 / np.pi)
        
        lat = center_lat + lat_offset * np.cos(angle)
        lon = center_lon + lon_offset * np.sin(angle)
        
        points.append([lon, lat])
    
    return points

with fiona.open(parks_file, 'w', driver='GeoJSON', schema=parks_schema, crs=crs) as output:
    for park_data in vietnam_parks:
        # Tạo polygon hình tròn đơn giản
        center_lon, center_lat = park_data['center']
        polygon_coords = create_circle_polygon(center_lon, center_lat, park_data['radius_km'])
        
        polygon = Polygon(polygon_coords)
        
        # Tính toán các chỉ số
        biodiversity_index = min(9.5, park_data['area_km2'] / 100 + np.random.uniform(5, 9))
        visitor_capacity = int(park_data['area_km2'] * 50)  # 50 người/km²
        
        feature = {
            'geometry': mapping(polygon),
            'properties': {
                'name': park_data['name'],
                'type': park_data['type'],
                'area_km2': park_data['area_km2'],
                'established': park_data['established'],
                'province': park_data['province'],
                'biodiversity_index': round(biodiversity_index, 1),
                'visitor_capacity': visitor_capacity
            }
        }
        
        output.write(feature)

print(f"✅ Đã tạo file khu bảo tồn: {parks_file}")

# 3. Ghi dữ liệu sang nhiều định dạng khác nhau
print(f"\n=== GHI SANG NHIỀU ĐỊNH DẠNG ===")

# Chuyển đổi sang KML (Google Earth)
kml_file = temp_dir / "vietnam_cities.kml"
with fiona.open(geojson_file, 'r') as src:
    with fiona.open(kml_file, 'w', driver='KML', schema=src.schema, crs=src.crs) as dst:
        for feature in src:
            dst.write(feature)

print(f"✅ Đã chuyển đổi sang KML: {kml_file}")

# Ghi ra CSV với tọa độ (cho Excel/Google Sheets)
csv_file = temp_dir / "vietnam_cities.csv"
csv_data = []

with fiona.open(geojson_file, 'r') as src:
    for feature in src:
        props = feature['properties'].copy()
        coords = feature['geometry']['coordinates']
        props['longitude'] = coords[0]
        props['latitude'] = coords[1]
        csv_data.append(props)

df_csv = pd.DataFrame(csv_data)
df_csv.to_csv(csv_file, index=False, encoding='utf-8')
print(f"✅ Đã xuất CSV: {csv_file}")

# Liệt kê tất cả file đã tạo
print(f"\n📂 TẤT CẢ FILE ĐÃ TẠO:")
all_files = list(temp_dir.glob("*"))
for file_path in sorted(all_files):
    size_kb = file_path.stat().st_size / 1024
    print(f"  {file_path.name}: {size_kb:.1f} KB")
```


```python
# Trực quan hóa dữ liệu đã tạo
print("=== TRỰC QUAN HÓA DỮ LIỆU ĐÃ TẠO ===")

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(16, 8))

# Biểu đồ 1: Tổng hợp tất cả dữ liệu
# Vẽ thành phố (Points)
with fiona.open(geojson_file, 'r') as src:
    for feature in src:
        coords = feature['geometry']['coordinates']
        props = feature['properties']
        ax1.scatter(coords[0], coords[1], c='red', s=props['population']/100000, 
                   alpha=0.7, label='Thành phố' if 'Thành phố' not in ax1.get_legend_handles_labels()[1] else "")
        ax1.annotate(props['name'], (coords[0], coords[1]), 
                    xytext=(2, 2), textcoords='offset points', fontsize=8)

# Vẽ sông (LineStrings)
with fiona.open(rivers_file, 'r') as src:
    for feature in src:
        coords = feature['geometry']['coordinates']
        props = feature['properties']
        lons, lats = zip(*coords)
        ax1.plot(lons, lats, 'b-', linewidth=props['length_km']/1000, 
                alpha=0.7, label='Sông' if 'Sông' not in ax1.get_legend_handles_labels()[1] else "")
        
        # Thêm tên sông ở giữa
        mid_idx = len(coords) // 2
        mid_coord = coords[mid_idx]
        ax1.annotate(props['name'], mid_coord, 
                    xytext=(0, -10), textcoords='offset points', 
                    fontsize=9, color='blue', ha='center')

# Vẽ khu bảo tồn (Polygons)
with fiona.open(parks_file, 'r') as src:
    for i, feature in enumerate(src):
        coords = feature['geometry']['coordinates'][0]  # Exterior ring
        props = feature['properties']
        lons, lats = zip(*coords)
        ax1.fill(lons, lats, alpha=0.3, color='green', 
                label='Khu bảo tồn' if 'Khu bảo tồn' not in ax1.get_legend_handles_labels()[1] else "")
        
        # Tên khu bảo tồn ở trung tâm
        center_lon = sum(lons) / len(lons)
        center_lat = sum(lats) / len(lats)
        ax1.annotate(props['name'], (center_lon, center_lat), 
                    fontsize=8, color='darkgreen', ha='center', va='center')

ax1.set_xlabel('Kinh độ (°)')
ax1.set_ylabel('Vĩ độ (°)')
ax1.set_title('Bản đồ tổng hợp Việt Nam')
ax1.legend()
ax1.grid(True, alpha=0.3)

# Biểu đồ 2: So sánh diện tích khu bảo tồn
park_names = []
park_areas = []
with fiona.open(parks_file, 'r') as src:
    for feature in src:
        props = feature['properties']
        park_names.append(props['name'].replace('Vườn Quốc gia ', '').replace('Khu Bảo tồn ', ''))
        park_areas.append(props['area_km2'])

bars = ax2.bar(range(len(park_names)), park_areas, color='green', alpha=0.7)
ax2.set_xlabel('Khu bảo tồn')
ax2.set_ylabel('Diện tích (km²)')
ax2.set_title('Diện tích các khu bảo tồn')
ax2.set_xticks(range(len(park_names)))
ax2.set_xticklabels(park_names, rotation=45, ha='right')

# Thêm nhãn trên cột
for bar, area in zip(bars, park_areas):
    height = bar.get_height()
    ax2.text(bar.get_x() + bar.get_width()/2., height + height*0.01,
             f'{area:.1f}', ha='center', va='bottom')

plt.tight_layout()
plt.show()

print("📊 Trực quan hóa dữ liệu vector mới hoàn thành!")
```

## 4. Lọc không gian và truy vấn (Spatial Filtering and Queries)

Đọc hiệu quả tập con dữ liệu dựa trên tiêu chí không gian và thuộc tính.


```python
# ==========================================
# LỌC KHÔNG GIAN VÀ TRUY VẤN DỮ LIỆU
# ==========================================

print("=== LỌC DỮ LIỆU THEO TIÊU CHÍ ===")

# 1. Lọc theo thuộc tính (Attribute filtering)
print("1. LỌC THEO THUỘC TÍNH:")

# Tìm các thành phố có dân số > 5 triệu
large_cities = []
with fiona.open(geojson_file, 'r') as src:
    for feature in src:
        props = feature['properties']
        if props['population'] > 5000000:
            large_cities.append(props['name'])
            print(f"   🏙️  {props['name']}: {props['population']:,} người")

print(f"   → Tìm thấy {len(large_cities)} thành phố lớn")

# Tìm khu bảo tồn được thành lập trước năm 2000
print("\n2. KHU BẢO TỒN CỔ:")
old_parks = []
with fiona.open(parks_file, 'r') as src:
    for feature in src:
        props = feature['properties']
        if props['established'] < 2000:
            old_parks.append({
                'name': props['name'],
                'established': props['established'],
                'area': props['area_km2']
            })
            print(f"   🌲  {props['name']}: thành lập {props['established']}, {props['area_km2']} km²")

# 2. Lọc không gian (Spatial filtering)
print(f"\n=== LỌC KHÔNG GIAN ===")

# Định nghĩa vùng quan tâm (miền Bắc Việt Nam)
north_vietnam_bbox = [102.0, 20.0, 110.0, 24.0]  # [min_lon, min_lat, max_lon, max_lat]
north_polygon = box(*north_vietnam_bbox)

print("3. LỌC THEO VÙNG ĐỊA LÝ (MIỀN BẮC):")

# Tìm thành phố trong miền Bắc
north_cities = []
with fiona.open(geojson_file, 'r') as src:
    for feature in src:
        point = shape(feature['geometry'])
        props = feature['properties']
        
        if north_polygon.contains(point):
            north_cities.append({
                'name': props['name'],
                'coords': feature['geometry']['coordinates'],
                'population': props['population']
            })
            print(f"   🌆  {props['name']}: {props['population']:,} người")

# 3. Lọc theo khoảng cách (Distance filtering)
print(f"\n4. LỌC THEO KHOẢNG CÁCH:")

# Chọn Hà Nội làm điểm trung tâm
hanoi_coords = [105.8542, 21.0285]
hanoi_point = Point(hanoi_coords)
search_radius_km = 200

print(f"   📍 Tìm địa điểm trong bán kính {search_radius_km}km từ Hà Nội:")

# Chuyển đổi radius từ km sang độ (ước tính)
radius_degrees = search_radius_km / 111.0  # 1 độ ≈ 111km

# Tạo vùng buffer
search_area = hanoi_point.buffer(radius_degrees)

# Tìm thành phố gần Hà Nội
nearby_cities = []
with fiona.open(geojson_file, 'r') as src:
    for feature in src:
        point = shape(feature['geometry'])
        props = feature['properties']
        
        if props['name'] != 'Hà Nội' and search_area.contains(point):
            # Tính khoảng cách chính xác
            distance = hanoi_point.distance(point) * 111  # chuyển sang km
            nearby_cities.append({
                'name': props['name'],
                'distance_km': round(distance, 1),
                'population': props['population']
            })

# Sắp xếp theo khoảng cách
nearby_cities.sort(key=lambda x: x['distance_km'])

for city in nearby_cities:
    print(f"   🚗  {city['name']}: {city['distance_km']} km, {city['population']:,} người")
```


```python
# 4. Truy vấn phức tạp (Complex queries)
print(f"\n=== TRUY VẤN PHỨC TẠP ===")

# Tìm sông chảy qua nhiều tỉnh
print("5. SÔNG CHẢY QUA NHIỀU TỈNH:")
multi_province_rivers = []
with fiona.open(rivers_file, 'r') as src:
    for feature in src:
        props = feature['properties']
        if props['province_count'] >= 3:
            multi_province_rivers.append({
                'name': props['name'],
                'provinces': props['province_count'],
                'length': props['length_km'],
                'basin': props['basin_area']
            })
            print(f"   🌊  {props['name']}: {props['province_count']} tỉnh, "
                  f"dài {props['length_km']:,}km, lưu vực {props['basin_area']:,}km²")

# Phân tích mối quan hệ không gian
print(f"\n6. PHÂN TÍCH MỐI QUAN HỆ KHÔNG GIAN:")

# Tìm khu bảo tồn gần thành phố lớn
print("   Khu bảo tồn gần thành phố lớn (< 100km):")

large_city_coords = []
with fiona.open(geojson_file, 'r') as src:
    for feature in src:
        props = feature['properties']
        if props['population'] > 1000000:  # Trên 1 triệu người
            large_city_coords.append({
                'name': props['name'],
                'point': shape(feature['geometry']),
                'population': props['population']
            })

park_city_relations = []
with fiona.open(parks_file, 'r') as src:
    for park_feature in src:
        park_props = park_feature['properties']
        park_shape = shape(park_feature['geometry'])
        park_center = park_shape.centroid
        
        # Tìm thành phố gần nhất
        min_distance = float('inf')
        nearest_city = None
        
        for city_data in large_city_coords:
            distance = park_center.distance(city_data['point']) * 111  # km
            if distance < min_distance:
                min_distance = distance
                nearest_city = city_data['name']
        
        if min_distance < 100:  # Trong vòng 100km
            park_city_relations.append({
                'park': park_props['name'],
                'city': nearest_city,
                'distance_km': round(min_distance, 1),
                'park_area': park_props['area_km2'],
                'biodiversity': park_props['biodiversity_index']
            })

for relation in sorted(park_city_relations, key=lambda x: x['distance_km']):
    print(f"   🏞️  {relation['park']}")
    print(f"      → Gần {relation['city']}: {relation['distance_km']}km")
    print(f"      → Diện tích: {relation['park_area']}km², Đa dạng sinh học: {relation['biodiversity']}")

# 5. Xuất kết quả lọc thành file mới
print(f"\n=== XUẤT KẾT QUẢ LỌC ===")

# Tạo file cho thành phố miền Bắc
north_cities_file = temp_dir / "north_vietnam_cities.geojson"

# Schema giống như file gốc
with fiona.open(geojson_file, 'r') as src:
    original_schema = src.schema
    original_crs = src.crs

with fiona.open(north_cities_file, 'w', driver='GeoJSON', 
               schema=original_schema, crs=original_crs) as output:
    
    with fiona.open(geojson_file, 'r') as src:
        for feature in src:
            point = shape(feature['geometry'])
            if north_polygon.contains(point):
                output.write(feature)

print(f"✅ Đã tạo file thành phố miền Bắc: {north_cities_file}")

# Tạo thống kê tổng hợp
print(f"\n📊 THỐNG KÊ TỔNG HỢP:")
print(f"   • Tổng số thành phố: {len([c for c in north_cities])} (miền Bắc)")
print(f"   • Thành phố lớn (>5M dân): {len(large_cities)}")
print(f"   • Khu bảo tồn cổ (<2000): {len(old_parks)}")
print(f"   • Sông đa tỉnh (≥3 tỉnh): {len(multi_province_rivers)}")
print(f"   • Khu bảo tồn gần TP lớn: {len(park_city_relations)}")
```


```python
# Trực quan hóa kết quả lọc và truy vấn
print("=== TRỰC QUAN HÓA KẾT QUẢ LỌC ===")

fig, ((ax1, ax2), (ax3, ax4)) = plt.subplots(2, 2, figsize=(16, 12))

# Biểu đồ 1: Lọc không gian - miền Bắc
ax1.add_patch(plt.Rectangle((north_vietnam_bbox[0], north_vietnam_bbox[1]),
                           north_vietnam_bbox[2]-north_vietnam_bbox[0],
                           north_vietnam_bbox[3]-north_vietnam_bbox[1],
                           fill=False, edgecolor='red', linewidth=2, linestyle='--'))

with fiona.open(geojson_file, 'r') as src:
    for feature in src:
        coords = feature['geometry']['coordinates']
        props = feature['properties']
        point = shape(feature['geometry'])
        
        if north_polygon.contains(point):
            ax1.scatter(coords[0], coords[1], c='red', s=100, alpha=0.8, label='Miền Bắc')
        else:
            ax1.scatter(coords[0], coords[1], c='lightblue', s=50, alpha=0.5, label='Khác')
        
        ax1.annotate(props['name'], coords, xytext=(2, 2), 
                    textcoords='offset points', fontsize=8)

ax1.set_title('Lọc không gian: Thành phố miền Bắc')
ax1.set_xlabel('Kinh độ (°)')
ax1.set_ylabel('Vĩ độ (°)')
ax1.legend()
ax1.grid(True, alpha=0.3)

# Biểu đồ 2: Lọc theo dân số
populations = []
city_names_pop = []
colors = []

with fiona.open(geojson_file, 'r') as src:
    for feature in src:
        props = feature['properties']
        populations.append(props['population'])
        city_names_pop.append(props['name'])
        if props['population'] > 5000000:
            colors.append('red')
        elif props['population'] > 1000000:
            colors.append('orange')
        else:
            colors.append('lightblue')

bars = ax2.bar(range(len(populations)), [p/1000000 for p in populations], color=colors, alpha=0.7)
ax2.set_title('Phân loại theo dân số')
ax2.set_xlabel('Thành phố')
ax2.set_ylabel('Dân số (triệu người)')
ax2.set_xticks(range(len(city_names_pop)))
ax2.set_xticklabels(city_names_pop, rotation=45, ha='right')

# Thêm ngưỡng
ax2.axhline(y=5, color='red', linestyle='--', alpha=0.7, label='5M người')
ax2.axhline(y=1, color='orange', linestyle='--', alpha=0.7, label='1M người')
ax2.legend()

# Biểu đồ 3: Khoảng cách từ Hà Nội
if nearby_cities:
    distances = [city['distance_km'] for city in nearby_cities]
    names = [city['name'] for city in nearby_cities]
    
    ax3.barh(range(len(distances)), distances, color='green', alpha=0.7)
    ax3.set_title(f'Khoảng cách từ Hà Nội (< {search_radius_km}km)')
    ax3.set_xlabel('Khoảng cách (km)')
    ax3.set_yticks(range(len(names)))
    ax3.set_yticklabels(names)
    ax3.grid(True, axis='x', alpha=0.3)
    
    # Vẽ vòng tròn khoảng cách trên map nhỏ
    circle = plt.Circle(hanoi_coords, radius_degrees, fill=False, color='green', linestyle='--')
    ax3_inset = ax3.inset_axes([0.6, 0.6, 0.35, 0.35])
    ax3_inset.add_patch(circle)
    ax3_inset.scatter(hanoi_coords[0], hanoi_coords[1], c='red', s=100, marker='*')
    
    for city in nearby_cities:
        with fiona.open(geojson_file, 'r') as src:
            for feature in src:
                if feature['properties']['name'] == city['name']:
                    coords = feature['geometry']['coordinates']
                    ax3_inset.scatter(coords[0], coords[1], c='green', s=50)
                    break
    
    ax3_inset.set_xlim(hanoi_coords[0]-radius_degrees*1.2, hanoi_coords[0]+radius_degrees*1.2)
    ax3_inset.set_ylim(hanoi_coords[1]-radius_degrees*1.2, hanoi_coords[1]+radius_degrees*1.2)
    ax3_inset.set_title('Vùng tìm kiếm', fontsize=8)

# Biểu đồ 4: Thống kê tổng hợp
categories = ['Tất cả TP', 'TP lớn\n(>5M)', 'TP miền Bắc', 'KBT cổ\n(<2000)', 'Sông đa tỉnh\n(≥3)']
counts = [
    len([c for c in north_cities]) + 2,  # Ước tính tổng
    len(large_cities),
    len([c for c in north_cities]),
    len(old_parks),
    len(multi_province_rivers)
]

colors_stat = ['lightblue', 'red', 'orange', 'green', 'blue']
bars = ax4.bar(categories, counts, color=colors_stat, alpha=0.7)
ax4.set_title('Thống kê kết quả lọc')
ax4.set_ylabel('Số lượng')

# Thêm số trên cột
for bar, count in zip(bars, counts):
    height = bar.get_height()
    ax4.text(bar.get_x() + bar.get_width()/2., height + 0.05,
             f'{count}', ha='center', va='bottom', fontweight='bold')

plt.tight_layout()
plt.show()

print("🎯 Hoàn thành phân tích lọc và truy vấn dữ liệu không gian!")
```

## Tóm tắt

Bạn đã hoàn thành Bài 3 và học được Fiona - thư viện chuyên nghiệp cho vector data I/O trong Python GIS ecosystem.

### Các khái niệm chính đã nắm vững:
- ✅ **Vector data I/O**: Đọc và ghi Shapefile, GeoJSON, KML, GPKG với fiona.open()
- ✅ **Collections và Features**: Làm việc với feature collections và individual features
- ✅ **Schema management**: Properties, geometry types và data validation
- ✅ **CRS handling**: Coordinate reference systems trong I/O operations
- ✅ **Spatial filtering**: Bbox queries và geometric filtering cho large datasets
- ✅ **Format conversions**: Chuyển đổi giữa các vector formats khác nhau
- ✅ **Integration với Shapely**: Seamless geometry conversion với shape() và mapping()
- ✅ **Performance optimization**: Efficient reading patterns cho big geospatial data

### Kỹ năng bạn có thể áp dụng:
- Xử lý và chuyển đổi dữ liệu vector từ nhiều nguồn khác nhau
- Thực hiện spatial queries và filtering hiệu quả trên large datasets
- Tích hợp Fiona với Shapely và GeoPandas trong geospatial workflows
- Validate và clean vector data với proper schema management
- Chuẩn bị data pipeline foundations cho advanced GIS analysis và visualization
