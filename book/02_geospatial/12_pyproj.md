# Bài 12: Hệ tọa độ tham chiếu và phép chiếu bản đồ

PyProj là giao diện Python cho thư viện PROJ (được sử dụng bởi hầu hết các phần mềm GIS chuyên nghiệp) để thực hiện các phép chiếu bản đồ và biến đổi tọa độ với độ chính xác cao. Đây là công cụ không thể thiếu khi làm việc với dữ liệu không gian địa lý từ nhiều nguồn khác nhau, và là nền tảng cho các thư viện khác như GeoPandas, Rasterio, Fiona.

## 12.1. Mục tiêu học tập
Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Hiểu rõ về hệ tọa độ tham chiếu (CRS) và các loại phép chiếu bản đồ
- Thực hiện biến đổi tọa độ chính xác giữa các CRS khác nhau
- Làm việc với các hệ tọa độ phổ biến tại Việt Nam (VN-2000, UTM, WGS84)
- Xử lý các vấn đề về datum và ellipsoid
- Áp dụng PyProj trong quy trình xử lý dữ liệu không gian thực tế


```python
# Nếu bạn chưa cài đặt pyproj, hãy chạy: pip install pyproj hoặc xem lại Bài 1 về cách cài đặt thư viện.
import pyproj
```

## 12.2. Các phương thức cơ bản để khởi tạo hệ tọa độ

Giới thiệu về khái niệm CRS và làm việc với các hệ tọa độ khác nhau. Sau đây là các cách phổ biến tạo hệ tọa độ trong `pyproj`.

## 12.2.1. Hệ tọa độ địa lý

Hệ tọa độ địa lý (Geographic Coordinate System) là hệ tọa độ dùng kinh độ và vĩ độ (lat/lon) để xác định vị trí trên bề mặt Trái Đất, thường dựa trên mô hình ellipsoid như WGS84 và sử dụng đơn vị độ (degrees).


```python
# Thiết lập hệ tọa độ địa lý toàn cầu WGS84
wgs84 = pyproj.CRS.from_epsg(4326)  # EPSG:4326
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
    

### 12.2.2. Hệ tọa độ VN-2000

Hệ tọa độ VN-2000 là một hệ tọa độ địa lý được sử dụng rộng rãi tại Việt Nam, dựa trên hệ thống tọa độ UTM (Universal Transverse Mercator) và sử dụng elipsoid WGS84. Hệ tọa độ này được thiết kế để cung cấp độ chính xác cao cho các ứng dụng GIS và bản đồ tại Việt Nam, giúp người dùng dễ dàng chuyển đổi giữa hệ tọa độ địa lý toàn cầu và hệ tọa độ cục bộ của Việt Nam.


```python
# Thiết lập hệ tọa độ VN2000 (UTM Zone 48N)
vn2000 = pyproj.CRS.from_epsg(4756)  # EPSG:4756
print(f"\nHệ tọa độ: {vn2000.name}")
print(f"Loại: {vn2000.type_name}")
print(f"Ellipsoid: {vn2000.ellipsoid.name}")
```

    
    Hệ tọa độ: VN-2000
    Loại: Geographic 2D CRS
    Ellipsoid: WGS 84
    

### 12.2.3. Hệ tọa độ UTM

Hệ tọa độ UTM (Universal Transverse Mercator) là hệ tọa độ chiếu phẳng trong Đại số tuyến tính và GIS, dùng phép chiếu Mercator ngang để chia Trái Đất thành 60 múi (zone), mỗi múi rộng 6° kinh độ, giúp biểu diễn vị trí bằng tọa độ x, y (mét) với độ chính xác cao cho từng khu vực nhỏ.


```python
# Định nghĩa hệ tọa độ UTM Zone 48N - Phù hợp cho miền Bắc và Trung Việt Nam
utm48n = pyproj.CRS.from_epsg(32648)  # EPSG:32648
print(f"\nUTM 48N: {utm48n.name}")
print(f"Loại: {utm48n.type_name}")
print(f"Đơn vị: {utm48n.axis_info[0].unit_name}")

# Định nghĩa hệ tọa độ UTM Zone 49N - Phù hợp cho miền Nam Việt Nam
utm49n = pyproj.CRS.from_epsg(32649)  # EPSG:32649
print(f"\nUTM 49N: {utm49n.name}")
print(f"Loại: {utm49n.type_name}")
print(f"Đơn vị: {utm49n.axis_info[0].unit_name}")
```

    
    UTM 48N: WGS 84 / UTM zone 48N
    Loại: Projected CRS
    Đơn vị: metre
    
    UTM 49N: WGS 84 / UTM zone 49N
    Loại: Projected CRS
    Đơn vị: metre
    

### 12.2.4. Tạo CRS từ Proj string

Pyproj hỗ trợ tạo hệ tọa độ tham chiếu (CRS) từ nhiều định dạng khác nhau, bao gồm mã EPSG, chuỗi PROJ và WKT. Trong mục này, chúng ta sẽ tìm hiểu cách khởi tạo CRS từ PROJ string - một định dạng mô tả hệ tọa độ thông qua tập hợp các tham số chiếu. Phương pháp này đặc biệt hữu ích khi cần định nghĩa các hệ tọa độ tùy chỉnh hoặc làm việc với dữ liệu không có mã EPSG tương ứng.


```python
# Tạo CRS từ PROJ string
vietnam_custom = pyproj.CRS.from_proj4("+proj=utm +zone=48 +datum=WGS84 +units=m +no_defs")
print(f"\nCRS tùy chỉnh: {vietnam_custom.name}") # Không có tên chính thức nên sẽ trả về Unknown. Bạn có thể in ra chuỗi PROJ để xác nhận.
```

    
    CRS tùy chỉnh: unknown
    


```python
# Tạo CRS từ chuỗi PROJ4, hệ tọa độ địa lý WGS84 nếu như bạn không có EPSG code cụ thể nào:
crs = pyproj.CRS.from_proj4("+proj=longlat +datum=WGS84 +no_defs") # Tạo từ chuỗi PROJ4
```

### 12.2.5. Tạo CRS từ mã `EPSG` và `dictionary`

Chúng ta có thể khởi tạo CRS từ mã code `EPSG` hoặc từ một `dictionary` như ví dụ bên dưới.


```python
crs = pyproj.CRS.from_user_input("EPSG:4326") # Tạo từ đầu vào người dùng
crs = pyproj.CRS.from_dict({
    "proj": "utm",
    "zone": 48,
    "datum": "WGS84",
    "units": "m"
}) # Tạo từ dictionary
```

## 12.3. Chuyển đổi hệ tọa độ

Việc chuyển đổi tọa độ giữa các hệ quy chiếu tọa độ (CRS) khác nhau được thực hiện thông qua đối tượng Transformer trong thư viện pyproj. Ví dụ dưới đây minh họa cách chuyển đổi tọa độ từ hệ tọa độ địa lý WGS84 sang hệ tọa độ phẳng UTM Zone 48N.


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
transformer_to_utm48 = pyproj.Transformer.from_crs("EPSG:4326", "EPSG:32648", always_xy=True)

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
    

## Tóm tắt

Trong bài học này, chúng ta đã học.

### Các khái niệm chính đã nắm vững:
- ✅ **Coordinate Reference Systems**: WGS84, VN-2000, UTM và cách lựa chọn CRS phù hợp
- ✅ **Coordinate transformations**: Chuyển đổi chính xác giữa các hệ tọa độ khác nhau
- ✅ **Map projections**: UTM, Mercator, Lambert với ưu nhược điểm từng loại
- ✅ **CRS operations**: from_epsg(), from_string(), to_wkt() cho CRS management

### Kỹ năng bạn có thể áp dụng:
- Xử lý dữ liệu GPS và chuyển đổi sang coordinate systems địa phương
- Thực hiện tính toán geodesic chính xác cho navigation và land surveying
- Lựa chọn map projections tối ưu cho từng vùng địa lý và ứng dụng
- Tích hợp coordinate transformations vào geospatial workflows
- Chuẩn bị dữ liệu coordinate cho GeoPandas và các GIS libraries nâng cao
