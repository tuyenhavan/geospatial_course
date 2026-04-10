# Bài 2: PyProj - Hệ tọa độ tham chiếu và phép chiếu bản đồ

PyProj là giao diện Python cho thư viện PROJ (được sử dụng bởi hầu hết các phần mềm GIS chuyên nghiệp) để thực hiện các phép chiếu bản đồ và biến đổi tọa độ với độ chính xác cao. Đây là công cụ không thể thiếu khi làm việc với dữ liệu không gian địa lý từ nhiều nguồn khác nhau, và là nền tảng cho các thư viện khác như GeoPandas, Rasterio, Fiona.

## Mục tiêu học tập
Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Hiểu rõ về hệ tọa độ tham chiếu (CRS) và các loại phép chiếu bản đồ
- Thực hiện biến đổi tọa độ chính xác giữa các CRS khác nhau
- Làm việc với các hệ tọa độ phổ biến tại Việt Nam (VN-2000, UTM, WGS84)
- Thực hiện các tính toán geodesic (khoảng cách và góc trên mặt cầu)
- Xử lý các vấn đề về datum và ellipsoid
- Áp dụng PyProj trong quy trình xử lý dữ liệu không gian thực tế
- Tối ưu hóa hiệu suất khi xử lý large datasets

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

## 1. Hiểu về hệ tọa độ tham chiếu - CRS

Giới thiệu về khái niệm CRS và làm việc với các hệ tọa độ khác nhau, đặc biệt tại Việt Nam.


```python
# 1. WGS84 - Hệ tọa độ địa lý toàn cầu (GPS)
wgs84 = CRS.from_epsg(4326)  # EPSG:4326
print(f"WGS84: {wgs84.name}")
print(f"Loại: {wgs84.type_name}")
print(f"Ellipsoid: {wgs84.ellipsoid.name}")
print(f"Datum: {wgs84.datum.name}")
```


```python
# 2. VN-2000 - Hệ tọa độ quốc gia Việt Nam
vn2000 = CRS.from_epsg(4756)  # EPSG:4756
print(f"\nVN-2000: {vn2000.name}")
print(f"Loại: {vn2000.type_name}")
print(f"Ellipsoid: {vn2000.ellipsoid.name}")
```


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


```python
# 5. Tạo CRS từ PROJ string
vietnam_custom = CRS.from_proj4("+proj=utm +zone=48 +datum=WGS84 +units=m +no_defs")
print(f"\nCRS tùy chỉnh: {vietnam_custom.name}")
print(f"WGS84 WKT:\n{wgs84.to_wkt()[:200]}...")
print(f"\nVN-2000 PROJ4: {vn2000.to_proj4()}")
```


```python
# Các phương thức tạo crs sử dụng pyproj 
crs = CRS.from_epsg(4326) # Hệ tọa độ địa lý, sử dụng phổ biến
crs = CRS.from_proj4("+proj=longlat +datum=WGS84 +no_defs") # Tạo từ chuỗi PROJ4
crs = CRS.from_wkt('GEOGCS["WGS 84",DATUM["WGS_1984",SPHEROID["WGS 84",6378137,298.257223563]],PRIMEM["Greenwich",0],UNIT["degree",0.0174532925199433]]') # Tạo từ WKT
crs = CRS.from_user_input("EPSG:4326") # Tạo từ đầu vào người dùng
crs = CRS.from_dict({
    "proj": "utm",
    "zone": 48,
    "datum": "WGS84",
    "units": "m"
}) # Tạo từ dictionary
# Hiển thị các thông tin thuộc tính của crs 
print(f"CRS Name: {crs.name}")
print(f"CRS Type: {crs.type_name}")
print(f"CRS Area of Use: {crs.area_of_use}")
print(f"CRS Datum: {crs.datum.name}")
print(f"CRS Ellipsoid: {crs.ellipsoid.name}")
print(f"CRS Axis Info: {crs.axis_info}")
print(f"CRS to_proj4: {crs.to_proj4()}")
print(f"CRS to epsg code: {crs.to_epsg()}")
```

## 2. Biến đổi tọa độ

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


```python
# Tạo lưới tọa độ bao phủ Việt Nam
lat_range = np.linspace(8.5, 23.5, 5)  # Từ Nam đến Bắc
lon_range = np.linspace(102, 110, 5)    # Từ Tây sang Đông
lon_grid, lat_grid = np.meshgrid(lon_range, lat_range)

# Flatten để biến đổi batch
lons_flat = lon_grid.flatten()
lats_flat = lat_grid.flatten()

# Biến đổi tất cả các điểm cùng lúc từ WGS84 to UTM 48N
transformer_to_utm48 = Transformer.from_crs("EPSG:4326", "EPSG:32648", always_xy=True)
xs, ys = transformer_to_utm48.transform(lons_flat, lats_flat)
# Tạo DataFrame để dễ xem
grid_data = pd.DataFrame({
    'Longitude': lons_flat,
    'Latitude': lats_flat,
    'UTM_X': xs,
    'UTM_Y': ys
})
```


```python
# Trực quan hóa biến đổi tọa độ
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(15, 6))

# Biểu đồ 1: Tọa độ WGS84 (địa lý)
for city, (lat, lon) in cities_wgs84.items():
    ax1.scatter(lon, lat, s=100, alpha=0.7)
    ax1.annotate(city, (lon, lat), xytext=(5, 5), textcoords='offset points', fontsize=10)

ax1.scatter(lon_grid, lat_grid, c='lightblue', s=20, alpha=0.5, label='Lưới tọa độ')
ax1.set_xlabel('Kinh độ (°)')
ax1.set_ylabel('Vĩ độ (°)')
ax1.set_title('Tọa độ WGS84 (Địa lý)')
ax1.grid(True, alpha=0.3)
ax1.legend()

# Biểu đồ 2: Tọa độ UTM 48N (chiếu)
for city, (x, y) in cities_utm48.items():
    ax2.scatter(x/1000, y/1000, s=100, alpha=0.7)  # Chia 1000 để hiển thị km
    ax2.annotate(city, (x/1000, y/1000), xytext=(5, 5), textcoords='offset points', fontsize=12)

# Reshape lại để vẽ lưới
xs_grid = xs.reshape(lat_grid.shape)
ys_grid = ys.reshape(lat_grid.shape)
ax2.scatter(xs_grid/1000, ys_grid/1000, c='lightblue', s=20, alpha=0.5, label='Lưới tọa độ')

ax2.set_xlabel('UTM X (km)')
ax2.set_ylabel('UTM Y (km)')
ax2.set_title('Tọa độ UTM 48N (Chiếu)')
ax2.grid(True, alpha=0.3)
ax2.legend()

plt.tight_layout()
plt.show()
```

## 3. Tính toán geodesic

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


```python
# Tạo ma trận khoảng cách giữa tất cả các thành phố
print("=== MA TRẬN KHOẢNG CÁCH GIỮA TẤT CẢ THÀNH PHỐ ===")

city_names = list(cities_wgs84.keys())
n_cities = len(city_names)

# Tạo ma trận rỗng
distance_matrix = np.zeros((n_cities, n_cities))

# Tính toán khoảng cách cho từng cặp thành phố
for i, city1 in enumerate(city_names):
    for j, city2 in enumerate(city_names):
        if i != j:
            coords1 = cities_wgs84[city1]
            coords2 = cities_wgs84[city2]
            _, _, distance = geod.inv(coords1[1], coords1[0], coords2[1], coords2[0])
            distance_matrix[i, j] = distance / 1000  # Chuyển sang km

# Tạo DataFrame để hiển thị đẹp
df_distances = pd.DataFrame(distance_matrix, 
                          index=city_names, 
                          columns=city_names)

print("📊 MA TRẬN KHOẢNG CÁCH (km):")
print(df_distances.round(1))

# Tìm cặp thành phố gần nhất và xa nhất (không tính chéo)
mask = np.triu(np.ones_like(distance_matrix, dtype=bool), k=1)
masked_distances = np.where(mask, distance_matrix, np.inf)

min_idx = np.unravel_index(np.argmin(masked_distances), masked_distances.shape)
max_idx = np.unravel_index(np.argmax(np.where(mask, distance_matrix, 0)), distance_matrix.shape)

print(f"\n🔍 PHÂN TÍCH MA TRẬN:")
print(f"Cặp gần nhất: {city_names[min_idx[0]]} - {city_names[min_idx[1]]} ({distance_matrix[min_idx]:.1f} km)")
print(f"Cặp xa nhất: {city_names[max_idx[0]]} - {city_names[max_idx[1]]} ({distance_matrix[max_idx]:.1f} km)")
print(f"Khoảng cách trung bình: {distance_matrix[distance_matrix > 0].mean():.1f} km")
```


```python
# Tính toán đường đi (great circle) và diện tích
print("=== TÍNH TOÁN ĐƯỜNG ĐI VÀ DIỆN TÍCH ===")

# 1. Tạo đường đi từ Hà Nội đến TP.HCM
hanoi_coords = cities_wgs84['Hà Nội']
hcm_coords = cities_wgs84['TP.HCM']

# Tính toán intermediate points dọc theo đường đi
npoints = 20  # Số điểm trung gian
line_geod = geod.npts(hanoi_coords[1], hanoi_coords[0], 
                      hcm_coords[1], hcm_coords[0], 
                      npoints)

# Thêm điểm đầu và cuối
full_line = [(hanoi_coords[1], hanoi_coords[0])] + line_geod + [(hcm_coords[1], hcm_coords[0])]
line_lons, line_lats = zip(*full_line)

print(f"🛣️ Đường đi Hà Nội → TP.HCM:")
print(f"Số điểm trung gian: {len(line_geod)}")
print(f"Khoảng cách trực tuyến: {distances_from_hanoi['TP.HCM']['distance_km']:.1f} km")

# 2. Tính diện tích đa giác tạo bởi các thành phố miền Bắc
northern_cities = ['Hà Nội', 'Hải Phòng']
if len(northern_cities) >= 3:
    # Cần ít nhất 3 điểm để tạo đa giác
    pass
else:
    # Tạo tam giác với Hà Nội, Hải Phòng và một điểm ảo
    print("📐 Tính diện tích khu vực miền Bắc (tam giác ước tính):")
    
    # Tạo điểm thứ 3 để tạo thành tam giác
    # Sử dụng Huế làm điểm thứ 3 để tạo tam giác lớn
    triangle_cities = ['Hà Nội', 'Hải Phòng', 'Huế']
    triangle_coords = []
    
    for city in triangle_cities:
        coords = cities_wgs84[city]
        triangle_coords.extend([coords[1], coords[0]])  # lon, lat, lon, lat, ...
    
    # Tính diện tích bằng polygon_area_perimeter
    area, perimeter = geod.polygon_area_perimeter(triangle_coords[::2], triangle_coords[1::2])
    
    print(f"Diện tích tam giác {'-'.join(triangle_cities)}: {abs(area)/1e6:.0f} km²")
    print(f"Chu vi: {perimeter/1000:.0f} km")
```


```python
# Trực quan hóa tính toán geodesic
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(15, 6))

# Biểu đồ 1: Đường đi geodesic Hà Nội - TP.HCM
ax1.plot(line_lons, line_lats, 'b-', linewidth=2, alpha=0.7, label='Đường đi geodesic')
ax1.scatter([hanoi_coords[1]], [hanoi_coords[0]], c='red', s=100, label='Hà Nội', zorder=5)
ax1.scatter([hcm_coords[1]], [hcm_coords[0]], c='blue', s=100, label='TP.HCM', zorder=5)

# Vẽ tất cả thành phố
for city, (lat, lon) in cities_wgs84.items():
    if city not in ['Hà Nội', 'TP.HCM']:
        ax1.scatter(lon, lat, c='gray', s=50, alpha=0.7)
        ax1.annotate(city, (lon, lat), xytext=(3, 3), textcoords='offset points', fontsize=8)

ax1.annotate('Hà Nội', hanoi_coords[::-1], xytext=(5, 5), textcoords='offset points', fontsize=10)
ax1.annotate('TP.HCM', hcm_coords[::-1], xytext=(5, 5), textcoords='offset points', fontsize=10)
ax1.set_xlabel('Kinh độ (°)')
ax1.set_ylabel('Vĩ độ (°)')
ax1.set_title('Đường đi geodesic Hà Nội - TP.HCM')
ax1.grid(True, alpha=0.3)
ax1.legend()

# Biểu đồ 2: Heat map ma trận khoảng cách
im = ax2.imshow(distance_matrix, cmap='YlOrRd', aspect='auto')
ax2.set_xticks(range(n_cities))
ax2.set_yticks(range(n_cities))
ax2.set_xticklabels(city_names, rotation=45, ha='right')
ax2.set_yticklabels(city_names)
ax2.set_title('Ma trận khoảng cách (km)')

# Thêm text hiển thị số liệu
for i in range(n_cities):
    for j in range(n_cities):
        if i != j:
            text = ax2.text(j, i, f'{distance_matrix[i, j]:.0f}',
                           ha="center", va="center", color="white" if distance_matrix[i, j] > 500 else "black",
                           fontsize=8)

plt.colorbar(im, ax=ax2, label='Khoảng cách (km)')
plt.tight_layout()
plt.show()

print("📊 Trực quan hóa tính toán geodesic hoàn thành!")
```

## 4. Làm việc với các phép chiếu khác nhau

Hiểu và áp dụng các phép chiếu bản đồ khác nhau phù hợp với Việt Nam.


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

### Lựa chọn phép chiếu phù hợp cho Việt Nam

**UTM (Universal Transverse Mercator)**: 
- ✅ **Tốt nhất cho đo đạc và bản đồ địa hình**
- ✅ Biến dạng tối thiểu trong phạm vi múi chiếu
- ✅ Được sử dụng rộng rãi trong GIS Việt Nam

**Mercator**:
- ✅ Tốt cho navigation và bản đồ web
- ❌ Biến dạng diện tích lớn ở vĩ độ cao

**Lambert Conformal Conic**:
- ✅ Tốt cho bản đồ quốc gia (kéo dài theo đông-tây)
- ✅ Bảo toàn góc và hình dạng tương đối tốt

**Albers Equal Area**:
- ✅ Bảo toàn diện tích chính xác
- ✅ Tốt cho phân tích thống kê không gian

## 5. Ứng dụng thực tế

Áp dụng PyProj trong các tình huống thực tế tại Việt Nam.


```python
# Kịch bản: Tính toán tuyến đường bộ tối ưu
# Giả sử cần xây dựng đường cao tốc nối các thành phố

# 1. Phân tích khoảng cách và thời gian di chuyển
speed_kmh = 80  # Tốc độ trung bình trên cao tốc (km/h)

print("🛣️ PHÂN TÍCH TUYẾN ĐƯỜNG CAO TỐC:")
print(f"Tốc độ trung bình: {speed_kmh} km/h")
print("-" * 50)

# Tính thời gian di chuyển giữa các thành phố
travel_times = {}
for city1 in city_names:
    travel_times[city1] = {}
    for city2 in city_names:
        if city1 != city2:
            distance_km = df_distances.loc[city1, city2]
            time_hours = distance_km / speed_kmh
            travel_times[city1][city2] = time_hours
            print(f"{city1} → {city2}: {distance_km:6.1f}km ({time_hours:.1f}h)")

# 2. Tính toán vùng phủ sóng dịch vụ
print(f"\n📍 VÙNG PHỦ SÓNG DỊCH VỤ (bán kính 100km):")

service_radius_km = 100
coverage_analysis = {}

for city, (lat, lon) in cities_wgs84.items():
    # Tìm các thành phố khác trong bán kính phục vụ
    nearby_cities = []
    for other_city, other_coords in cities_wgs84.items():
        if city != other_city:
            distance = df_distances.loc[city, other_city]
            if distance <= service_radius_km:
                nearby_cities.append((other_city, distance))
    
    coverage_analysis[city] = nearby_cities
    print(f"{city}: {len(nearby_cities)} thành phố trong bán kính {service_radius_km}km")
    for nearby, dist in nearby_cities:
        print(f"  - {nearby}: {dist:.1f}km")

# 3. Tính toán trung tâm hình học của các thành phố
all_lats = [coords[0] for coords in cities_wgs84.values()]
all_lons = [coords[1] for coords in cities_wgs84.values()]

geographic_center = (np.mean(all_lats), np.mean(all_lons))
print(f"\n🎯 TRUNG TÂM HÌNH HỌC: ({geographic_center[0]:.4f}°, {geographic_center[1]:.4f}°)")

# Tìm thành phố gần trung tâm nhất
center_distances = {}
for city, (lat, lon) in cities_wgs84.items():
    _, _, distance = geod.inv(geographic_center[1], geographic_center[0], lon, lat)
    center_distances[city] = distance / 1000

closest_to_center = min(center_distances.items(), key=lambda x: x[1])
print(f"Thành phố gần trung tâm nhất: {closest_to_center[0]} ({closest_to_center[1]:.1f}km)")
```


```python
# Ứng dụng trong nông nghiệp: Tính toán diện tích canh tác
print("=== ỨNG DỤNG NÔNG NGHIỆP: TÍNH DIỆN TÍCH CANH TÁC ===")

# Mô phỏng các khu vực canh tác (polygon) 
crop_areas = {
    'Đồng bằng sông Hồng': [
        (105.5, 20.5), (106.5, 20.5), (106.5, 21.5), 
        (106.0, 22.0), (105.5, 21.5), (105.5, 20.5)
    ],
    'Đồng bằng sông Cửu Long': [
        (105.0, 9.5), (106.5, 9.5), (107.0, 10.5),
        (106.0, 11.0), (105.0, 10.5), (105.0, 9.5)
    ]
}

total_area = 0
for region_name, coords in crop_areas.items():
    # Tách lon và lat
    lons = [coord[0] for coord in coords]
    lats = [coord[1] for coord in coords]
    
    # Tính diện tích bằng geodesic
    area, perimeter = geod.polygon_area_perimeter(lons, lats)
    area_km2 = abs(area) / 1e6  # Chuyển sang km²
    perimeter_km = perimeter / 1000
    
    print(f"🌾 {region_name}:")
    print(f"   Diện tích: {area_km2:,.0f} km²")
    print(f"   Chu vi: {perimeter_km:,.0f} km")
    
    total_area += area_km2

print(f"\n📊 TỔNG DIỆN TÍCH CANH TÁC: {total_area:,.0f} km²")

# So sánh với diện tích thực tế Việt Nam
vietnam_total_area = 331690  # km²
crop_percentage = (total_area / vietnam_total_area) * 100
print(f"Tỷ lệ so với diện tích Việt Nam: {crop_percentage:.1f}%")
```


```python
# Trực quan hóa ứng dụng thực tế
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(15, 6))

# Biểu đồ 1: Mạng lưới giao thông và vùng phủ sóng
for city, (lat, lon) in cities_wgs84.items():
    ax1.scatter(lon, lat, s=100, alpha=0.7, zorder=5)
    ax1.annotate(city, (lon, lat), xytext=(3, 3), textcoords='offset points', fontsize=9)
    
    # Vẽ vùng phủ sóng (đơn giản hóa thành hình tròn)
    circle = plt.Circle((lon, lat), service_radius_km/111, # Chuyển đổi gần đúng km sang độ
                       fill=False, alpha=0.3, linestyle='--')
    ax1.add_patch(circle)

# Vẽ trung tâm hình học
ax1.scatter(geographic_center[1], geographic_center[0], 
           c='red', s=200, marker='*', label='Trung tâm hình học', zorder=6)

# Vẽ các tuyến đường chính (ví dụ)
major_routes = [
    ['Hà Nội', 'Hải Phòng'],
    ['Hà Nội', 'Huế'],
    ['Huế', 'Đà Nẵng'],
    ['Đà Nẵng', 'TP.HCM'],
    ['TP.HCM', 'Cần Thơ']
]

for route in major_routes:
    city1, city2 = route
    lat1, lon1 = cities_wgs84[city1]
    lat2, lon2 = cities_wgs84[city2]
    ax1.plot([lon1, lon2], [lat1, lat2], 'b-', alpha=0.6, linewidth=1.5)

ax1.set_xlabel('Kinh độ (°)')
ax1.set_ylabel('Vĩ độ (°)')
ax1.set_title('Mạng lưới giao thông và vùng phủ sóng')
ax1.grid(True, alpha=0.3)
ax1.legend()

# Biểu đồ 2: Khu vực canh tác
colors = ['lightgreen', 'lightcoral']
for i, (region_name, coords) in enumerate(crop_areas.items()):
    lons = [coord[0] for coord in coords]
    lats = [coord[1] for coord in coords]
    ax2.fill(lons, lats, alpha=0.6, color=colors[i], label=region_name)
    ax2.plot(lons, lats, 'k-', linewidth=1)

# Vẽ các thành phố lên bản đồ nông nghiệp
for city, (lat, lon) in cities_wgs84.items():
    ax2.scatter(lon, lat, s=80, c='black', alpha=0.8, zorder=5)
    ax2.annotate(city, (lon, lat), xytext=(2, 2), textcoords='offset points', fontsize=8)

ax2.set_xlabel('Kinh độ (°)')
ax2.set_ylabel('Vĩ độ (°)')
ax2.set_title('Khu vực canh tác chính')
ax2.grid(True, alpha=0.3)
ax2.legend()

plt.tight_layout()
plt.show()
```


    
![png](output_25_0.png)
    


## Tóm tắt

Bạn đã hoàn thành Bài 2 và học được PyProj - thư viện chuyên nghiệp cho coordinate systems và map projections.

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
