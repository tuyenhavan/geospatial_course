# Bài 12: Hệ tọa độ tham chiếu và phép chiếu bản đồ (Pyproj)

PyProj là giao diện Python cho thư viện PROJ (được sử dụng bởi hầu hết các phần mềm GIS chuyên nghiệp) để thực hiện các phép chiếu bản đồ và biến đổi tọa độ với độ chính xác cao. Đây là công cụ không thể thiếu khi làm việc với dữ liệu không gian địa lý từ nhiều nguồn khác nhau, và là nền tảng cho các thư viện khác như GeoPandas, Rasterio, Fiona.

## 12.1. Mục tiêu học tập
Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Hiểu rõ về hệ tọa độ tham chiếu (CRS) và các loại phép chiếu bản đồ
- Thực hiện biến đổi tọa độ chính xác giữa các CRS khác nhau
- Làm việc với các hệ tọa độ phổ biến tại Việt Nam (VN-2000, UTM, WGS84)
- Xử lý các vấn đề về datum và ellipsoid
- Áp dụng PyProj trong quy trình xử lý dữ liệu không gian thực tế

- **Import các thư viện cần thiết**


```python
# Import các thư viện cần thiết
import pyproj
from pyproj import CRS, Transformer, Geod, Proj
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from math import radians, degrees, sin, cos, asin, sqrt
import warnings
warnings.filterwarnings('ignore')
# Hiển thị phiên bản của PyProj và PROJ
print(f"📦 PyProj version: {pyproj.__version__}")
print(f"📦 PROJ version: {pyproj.proj_version_str}")
```

    📦 PyProj version: 3.7.2
    📦 PROJ version: 9.5.1
    

## 12.2. Hiểu về hệ tọa độ tham chiếu - CRS

Giới thiệu về khái niệm CRS và làm việc với các hệ tọa độ khác nhau. Sau đây là các cách phổ biến tạo hệ tọa độ trong `pyproj`.

- **Hệ tọa độ địa lý**

Hệ tọa độ địa lý (Geographic Coordinate System) là hệ tọa độ dùng kinh độ và vĩ độ (lat/lon) để xác định vị trí trên bề mặt Trái Đất, thường dựa trên mô hình ellipsoid như WGS84 và sử dụng đơn vị độ (degrees).


```python
# 1. WGS84 - Hệ tọa độ địa lý toàn cầu (GPS)
wgs84 = CRS.from_epsg(4326)  # EPSG:4326
print(f"WGS84: {wgs84.name}")
print(f"Loại: {wgs84.type_name}")
print(f"Ellipsoid: {wgs84.ellipsoid.name}")
print(f"Datum: {wgs84.datum.name}")
print(f"Mã EPSG: {wgs84.to_epsg()}")
```

    WGS84: WGS 84
    Loại: Geographic 2D CRS
    Ellipsoid: WGS 84
    Datum: World Geodetic System 1984 ensemble
    Mã EPSG: 4326
    

- **Hệ tọa độ VN-2000**


```python
# 2. VN-2000 - Hệ tọa độ quốc gia Việt Nam
vn2000 = CRS.from_epsg(4756)  # EPSG:4756
print(f"\nVN-2000: {vn2000.name}")
print(f"Loại: {vn2000.type_name}")
print(f"Ellipsoid: {vn2000.ellipsoid.name}")
```

    
    VN-2000: VN-2000
    Loại: Geographic 2D CRS
    Ellipsoid: WGS 84
    

- **Hệ tọa độ UTM**

Hệ tọa độ UTM (Universal Transverse Mercator) là hệ tọa độ chiếu phẳng trong Đại số tuyến tính và GIS, dùng phép chiếu Mercator ngang để chia Trái Đất thành 60 múi (zone), mỗi múi rộng 6° kinh độ, giúp biểu diễn vị trí bằng tọa độ x, y (mét) với độ chính xác cao cho từng khu vực nhỏ.


```python
# 3. UTM Zone 48N - Phù hợp cho miền Bắc và Trung Việt Nam
utm48n = CRS.from_epsg(32648)  # EPSG:32648
print(f"\nUTM 48N: {utm48n.name}")
print(f"Loại: {utm48n.type_name}")
print(f"Đơn vị: {utm48n.axis_info[0].unit_name}")

# 4. UTM Zone 49N - Phù hợp cho miền Nam Việt Nam
utm49n = CRS.from_epsg(32649)  # EPSG:32649
print(f"\nUTM 49N: {utm49n.name}")
```

    
    UTM 48N: WGS 84 / UTM zone 48N
    Loại: Projected CRS
    Đơn vị: metre
    
    UTM 49N: WGS 84 / UTM zone 49N
    

- **Tạo CRS từ Proj string**


```python
# Tạo CRS từ PROJ string
vietnam_custom = CRS.from_proj4("+proj=utm +zone=48 +datum=WGS84 +units=m +no_defs")
print(f"\nCRS tùy chỉnh: {vietnam_custom.name}") # Không có tên chính thức nên sẽ trả về Unknown. Bạn có thể in ra chuỗi PROJ để xác nhận.
print(f"\nVN-2000 PROJ4: {vn2000.to_proj4()}")
```

    
    CRS tùy chỉnh: unknown
    
    VN-2000 PROJ4: +proj=longlat +ellps=WGS84 +no_defs +type=crs
    


```python
# Tạo CRS từ chuỗi PROJ4, hệ tọa độ địa lý WGS84 nếu như bạn không có EPSG code cụ thể nào:
crs = CRS.from_proj4("+proj=longlat +datum=WGS84 +no_defs") # Tạo từ chuỗi PROJ4
```

- **Tạo CRS từ mã `EPSG` và `dictionary`**


```python
crs = CRS.from_user_input("EPSG:4326") # Tạo từ đầu vào người dùng
crs = CRS.from_dict({
    "proj": "utm",
    "zone": 48,
    "datum": "WGS84",
    "units": "m"
}) # Tạo từ dictionary
```

## 12.3. Biến đổi tọa độ

Chuyển đổi tọa độ giữa các CRS khác nhau sử dụng đối tượng Transformer.


```python
# Tọa độ các thành phố lớn (WGS84 - độ thập phân)
cities_wgs84 = {
    'Hà Nội': (21.0285, 105.8542),
    'TP.HCM': (10.8231, 106.6297),
    'Đà Nẵng': (16.0471, 108.2068),
    'Cần Thơ': (10.0452, 105.7469),
    'Hải Phòng': (20.8449, 106.6881),
    'Huế': (16.4637, 107.5909)
}

# 1. Biến đổi từ WGS84 sang UTM 48N
transformer_to_utm48 = Transformer.from_crs("EPSG:4326", "EPSG:32648", always_xy=True)

print("🗺️ CHUYỂN ĐỔI WGS84 → UTM 48N:")
cities_utm48 = {}
for city, (lat, lon) in cities_wgs84.items():
    x, y = transformer_to_utm48.transform(lon, lat)  # always_xy=True: lon, lat
    cities_utm48[city] = (x, y)
    print(f"{city:10}: ({lat:7.4f}°, {lon:8.4f}°) → ({x:9.0f}m, {y:10.0f}m)")
```

    🗺️ CHUYỂN ĐỔI WGS84 → UTM 48N:
    Hà Nội    : (21.0285°, 105.8542°) → (   588762m,    2325539m)
    TP.HCM    : (10.8231°, 106.6297°) → (   678162m,    1196896m)
    Đà Nẵng   : (16.0471°, 108.2068°) → (   843173m,    1776802m)
    Cần Thơ   : (10.0452°, 105.7469°) → (   581848m,    1110503m)
    Hải Phòng : (20.8449°, 106.6881°) → (   675642m,    2305903m)
    Huế       : (16.4637°, 107.5909°) → (   776636m,    1822002m)
    

## 12.4. Tính toán geodesic

Thực hiện tính toán khoảng cách và diện tích chính xác trên bề mặt Trái đất.


```python
# Tạo đối tượng Geod cho ellipsoid WGS84
geod = Geod(ellps='WGS84')

# Tính khoảng cách giữa Hà Nội và các thành phố khác
hanoi = cities_wgs84['Hà Nội']

distances_from_hanoi = {}
for city, coords in cities_wgs84.items():
    if city != 'Hà Nội':
        # inverse() trả về: azimuth1, azimuth2, distance
        az1, az2, distance = geod.inv(hanoi[1], hanoi[0], coords[1], coords[0])
        distances_from_hanoi[city] = {
            'distance_km': distance / 1000,
            'azimuth_from_hanoi': az1,
            'azimuth_to_hanoi': az2
        }
        print(f"  → {city:10}: {distance/1000:7.1f} km (hướng: {az1:6.1f}°)")

# Tìm thành phố gần và xa nhất
closest_city = min(distances_from_hanoi.items(), key=lambda x: x[1]['distance_km'])
farthest_city = max(distances_from_hanoi.items(), key=lambda x: x[1]['distance_km'])

print(f"\n🏃 Gần nhất: {closest_city[0]} ({closest_city[1]['distance_km']:.1f} km)")
print(f"🚗 Xa nhất: {farthest_city[0]} ({farthest_city[1]['distance_km']:.1f} km)")
```

      → TP.HCM    :  1132.4 km (hướng:  175.7°)
      → Đà Nẵng   :   604.7 km (hướng:  155.4°)
      → Cần Thơ   :  1215.4 km (hướng: -179.4°)
      → Hải Phòng :    89.1 km (hướng:  103.0°)
      → Huế       :   537.4 km (hướng:  159.8°)
    
    🏃 Gần nhất: Hải Phòng (89.1 km)
    🚗 Xa nhất: Cần Thơ (1215.4 km)
    

## 12.5. Làm việc với các phép chiếu khác nhau

Hiểu và áp dụng các phép chiếu bản đồ khác nhau.


```python
# 1. UTM (Universal Transverse Mercator) - Phổ biến nhất
utm_proj = Proj(proj='utm', zone=48, datum='WGS84')
print(f"UTM Zone 48N: {utm_proj.crs}")

# 2. Mercator - Cho navigation
mercator_proj = Proj(proj='merc', datum='WGS84')
print(f"Mercator: {mercator_proj.crs}")

# 3. Lambert Conformal Conic - Tốt cho khu vực rộng theo chiều đông-tây
lambert_proj = Proj(proj='lcc', lat_1=16, lat_2=20, lat_0=18, lon_0=106, datum='WGS84')
print(f"Lambert Conformal Conic: {lambert_proj.crs}")

# 4. Albers Equal Area - Bảo toàn diện tích
albers_proj = Proj(proj='aea', lat_1=10, lat_2=22, lat_0=16, lon_0=106, datum='WGS84')
print(f"Albers Equal Area: {albers_proj.crs}")

# 5. Sinusoidal - Phép chiếu bảo toàn diện tích
sinusoidal_proj = Proj(proj='sinu', lon_0=106, datum='WGS84')
print(f"Sinusoidal: {sinusoidal_proj.crs}")

# So sánh biến dạng ở các phép chiếu khác nhau
print(f"\n=== SO SÁNH BIẾN DẠNG TẠI CÁC THÀNH PHỐ ===")

# Tọa độ test: hình vuông 1° x 1° xung quanh mỗi thành phố
test_cities = ['Hà Nội', 'TP.HCM']

for city in test_cities:
    lat, lon = cities_wgs84[city]
    print(f"\n🏙️ {city} ({lat:.2f}°, {lon:.2f}°):")
    
    # Tạo hình vuông 1° x 1°
    square_corners = [
        (lon - 0.5, lat - 0.5),  # SW
        (lon + 0.5, lat - 0.5),  # SE
        (lon + 0.5, lat + 0.5),  # NE
        (lon - 0.5, lat + 0.5),  # NW
    ]
    
    # Tính diện tích trong các phép chiếu khác nhau
    projections = {
        'UTM 48N': utm_proj,
        'Mercator': mercator_proj,
        'Lambert CC': lambert_proj,
        'Albers EA': albers_proj
    }
    
    for proj_name, proj in projections.items():
        # Biến đổi các góc sang hệ chiếu
        projected_corners = []
        for corner_lon, corner_lat in square_corners:
            x, y = proj(corner_lon, corner_lat)
            projected_corners.append((x, y))
        
        # Tính diện tích bằng công thức Shoelace
        def shoelace_area(corners):
            n = len(corners)
            area = 0.0
            for i in range(n):
                j = (i + 1) % n
                area += corners[i][0] * corners[j][1]
                area -= corners[j][0] * corners[i][1]
            return abs(area) / 2.0
        
        area_proj = shoelace_area(projected_corners)
        
        # Diện tích thực tế của 1° x 1° (geodesic)
        # Tính bằng pyproj.Geod
        square_lons = [c[0] for c in square_corners] + [square_corners[0][0]]
        square_lats = [c[1] for c in square_corners] + [square_corners[0][1]]
        area_real, _ = geod.polygon_area_perimeter(square_lons, square_lats)
        area_real = abs(area_real)
        
        # Tính tỷ lệ biến dạng
        distortion = (area_proj - area_real) / area_real * 100
        
        print(f"  {proj_name:12}: {area_proj/1e6:8.1f} km² (biến dạng: {distortion:+5.1f}%)")
```

    UTM Zone 48N: +proj=utm +zone=48 +datum=WGS84 +type=crs
    Mercator: +proj=merc +datum=WGS84 +type=crs
    Lambert Conformal Conic: +proj=lcc +lat_1=16 +lat_2=20 +lat_0=18 +lon_0=106 +datum=WGS84 +type=crs
    Albers Equal Area: +proj=aea +lat_1=10 +lat_2=22 +lat_0=16 +lon_0=106 +datum=WGS84 +type=crs
    Sinusoidal: +proj=sinu +lon_0=106 +datum=WGS84 +type=crs
    
    === SO SÁNH BIẾN DẠNG TẠI CÁC THÀNH PHỐ ===
    
    🏙️ Hà Nội (21.03°, 105.85°):
      UTM 48N     :  11502.2 km² (biến dạng:  -0.1%)
      Mercator    :  13198.9 km² (biến dạng: +14.7%)
      Lambert CC  :  11527.4 km² (biến dạng:  +0.2%)
      Albers EA   :  11509.0 km² (biến dạng:  -0.0%)
    
    🏙️ TP.HCM (10.82°, 106.63°):
      UTM 48N     :  12095.8 km² (biến dạng:  -0.0%)
      Mercator    :  12535.1 km² (biến dạng:  +3.6%)
      Lambert CC  :  12269.3 km² (biến dạng:  +1.4%)
      Albers EA   :  12095.6 km² (biến dạng:  -0.0%)
    


```python
# Trực quan hóa các phép chiếu
fig, axes = plt.subplots(2, 3, figsize=(18, 12))
axes = axes.flatten()

# Tạo lưới tọa độ cho Việt Nam
lat_range = np.linspace(8.5, 23.5, 8)
lon_range = np.linspace(102, 110, 8)
lon_grid, lat_grid = np.meshgrid(lon_range, lat_range)

# Các phép chiếu để so sánh
projections_viz = {
    'Gốc (WGS84)': None,
    'UTM Zone 48N': utm_proj,
    'Mercator': mercator_proj,
    'Lambert CC': lambert_proj,
    'Albers Equal Area': albers_proj,
    'Sinusoidal': sinusoidal_proj
}

for idx, (proj_name, proj) in enumerate(projections_viz.items()):
    ax = axes[idx]
    
    if proj is None:
        # Hiển thị tọa độ gốc
        x_plot, y_plot = lon_grid, lat_grid
        ax.set_xlabel('Kinh độ (°)')
        ax.set_ylabel('Vĩ độ (°)')
    else:
        # Biến đổi tọa độ
        x_proj, y_proj = proj(lon_grid, lat_grid)
        x_plot, y_plot = x_proj/1000, y_proj/1000  # Chuyển sang km
        ax.set_xlabel('X (km)')
        ax.set_ylabel('Y (km)')
    
    # Vẽ lưới
    ax.plot(x_plot, y_plot, 'b-', alpha=0.3)
    ax.plot(x_plot.T, y_plot.T, 'b-', alpha=0.3)
    
    # Vẽ các thành phố
    for city, (lat, lon) in cities_wgs84.items():
        if proj is None:
            city_x, city_y = lon, lat
        else:
            city_x, city_y = proj(lon, lat)
            city_x, city_y = city_x/1000, city_y/1000
        
        ax.scatter(city_x, city_y, c='red', s=50, alpha=0.8)
        ax.annotate(city, (city_x, city_y), xytext=(2, 2), 
                   textcoords='offset points', fontsize=8)
    
    ax.set_title(f'{proj_name}')
    ax.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```


    
![png](output_21_0.png)
    


## Tóm tắt

Bạn đã hoàn thành Bài 12 và học được PyProj - thư viện chuyên nghiệp cho coordinate systems và map projections.

### Các khái niệm chính đã nắm vững:
- ✅ **Coordinate Reference Systems**: WGS84, VN-2000, UTM và cách lựa chọn CRS phù hợp
- ✅ **Coordinate transformations**: Chuyển đổi chính xác giữa các hệ tọa độ khác nhau
- ✅ **Geodesic calculations**: Tính khoảng cách, diện tích và azimuth trên mặt cầu
- ✅ **Map projections**: UTM, Mercator, Lambert với ưu nhược điểm từng loại
- ✅ **CRS operations**: from_epsg(), from_string(), to_wkt() cho CRS management
- ✅ **Transformer class**: Hiệu suất cao cho batch coordinate transformations
- ✅ **Geod class**: Tính toán geodesic chính xác cho navigation và surveying
- ✅ **Ứng dụng Việt Nam**: VN-2000 coordinates, UTM zones 48N/49N cho địa phương

### Kỹ năng bạn có thể áp dụng:
- Xử lý dữ liệu GPS và chuyển đổi sang coordinate systems địa phương
- Thực hiện tính toán geodesic chính xác cho navigation và land surveying
- Lựa chọn map projections tối ưu cho từng vùng địa lý và ứng dụng
- Tích hợp coordinate transformations vào geospatial workflows
- Chuẩn bị dữ liệu coordinate cho GeoPandas và các GIS libraries nâng cao
