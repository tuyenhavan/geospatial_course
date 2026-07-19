# Bài 14: Thao tác dữ liệu với GDAL

GDAL (Geospatial Data Abstraction Library) là thư viện mã nguồn mở mạnh mẽ nhất trong hệ sinh thái GIS, cung cấp nền tảng để đọc, ghi, chuyển đổi và xử lý dữ liệu địa không gian. Với hỗ trợ hơn **200 định dạng** dữ liệu khác nhau, GDAL là xương sống cho GeoPandas, Rasterio, Fiona và hầu hết các phần mềm GIS chuyên nghiệp như QGIS và ArcGIS.

GDAL bao gồm 3 thành phần chính:
- **GDAL**: Xử lý dữ liệu raster (ảnh vệ tinh, DEM, bản đồ số hóa)
- **OGR**: Xử lý dữ liệu vector (điểm, đường, vùng)
- **OSR**: Quản lý hệ tọa độ tham chiếu (CRS) và phép chiếu bản đồ

> **Lưu ý**: Bạn có thể chạy trực tiếp notebook này bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/1Vj7XgUfvuHVRH5AVFnx-6RTF_M3BJRmk) mà không cần cài đặt Python.

## 14.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

- Hiểu kiến trúc GDAL và vai trò trong hệ sinh thái GIS Python
- Thao tác dữ liệu raster - tạo, đọc, ghi, reproject, clip với GDAL
- Thao tác dữ liệu vector - đọc, ghi, tạo, truy vấn không gian với OGR
- Quản lý hệ tọa độ và chuyển đổi projection với OSR
- Chuyển đổi định dạng giữa hàng trăm format địa không gian


```python
# Import các thư viện cần thiết
from osgeo import gdal, ogr, osr   # GDAL/OGR/OSR
import numpy as np                  # Xử lý mảng số
import os                           # Thao tác file/directory
```

## 14.2. Xử lý dữ liệu Raster với GDAL

GDAL xử lý dữ liệu raster thông qua mô hình phân cấp `Dataset` → `Bands` → `Pixels`:
- **Dataset**: File raster hoàn chỉnh (tương tự "file container")
- **Band**: Một kênh dữ liệu trong dataset (kênh đỏ, xanh lá, NIR, elevation,…)
- **GeoTransform**: 6 tham số xác định vị trí và kích thước pixel trên bản đồ
- **Projection**: Hệ tọa độ tham chiếu (CRS) của dataset

### 14.2.1. Tạo raster từ đầu (Create Raster)

Trong ví dự sau, chúng ta sẽ tạo ra một dữ liệu độ cao (giả định) cho khu vực nghiên cứu. Mặc dù ví dụ này về DEM, nhưng các bước sau áp dụng cho bất kì một dữ liệu nào.

- Chọn driver định dạng raster cần tạo. 
- Tạo file raster với thông số xác định (số cột, số hàng, và số kênh)
- Thiết lập GeoTransform
- Thiết lập hệ tọa độ tham chiếu (CRS)
- Tạo dữ liệu DEM giả lập bằng NumPy (bạn nên dùng dữ liệu của bạn nếu có)
- Ghi dữ liệu vào band raster
- Thiết lập NoData và thống kê.
- Lưu và đóng file.


```python
raster_path = r"J:\My Drive\geocourse_data\outputs"
# B1: Chọn driver để tạo file GeoTIFF
driver = gdal.GetDriverByName('GTiff')

# B2: Tạo file GeoTIFF mới với kích thước 100x100 pixel, 1 band, kiểu dữ liệu float32
output_tif = os.path.join(raster_path, 'dem_sample.tif')
cols, rows, n_bands = 100, 100, 1
dataset = driver.Create(output_tif, cols, rows, n_bands, gdal.GDT_Float32)

# B3: Thiết lập GeoTransform: (x_min, pixel_width, rotation_x, y_max, rotation_y, -pixel_height)
x_min, y_max = 105.8, 21.5 # Ví dụ tọa độ 
pixel_size = 0.01   # 0.01 độ ≈ ~1 km
dataset.SetGeoTransform((x_min, pixel_size, 0, y_max, 0, -pixel_size))

# B4: Thiết lập hệ tọa độ WGS84 (EPSG:4326)
srs = osr.SpatialReference()
srs.ImportFromEPSG(4326)
dataset.SetProjection(srs.ExportToWkt())

# B5: Tạo dữ liệu elevation giả lập với numpy (mô phỏng địa hình)
np.random.seed(42)
base = np.random.uniform(5, 300, (rows, cols)).astype(np.float32)
# Thêm gradient: phía Tây cao hơn (núi), phía Đông thấp hơn (đồng bằng)
x_grad = np.linspace(200, 0, cols)
y_grad = np.linspace(50, 150, rows)
elevation = base + np.meshgrid(x_grad, y_grad)[0] + np.meshgrid(x_grad, y_grad)[1]

# B6: Ghi dữ liệu vào band 1
band = dataset.GetRasterBand(1)
band.WriteArray(elevation)
# B7: Thiết lập NoData value và tính thống kê
band.SetNoDataValue(-9999)        # Giá trị đại diện cho "không có dữ liệu"
band.ComputeStatistics(False)     # Tính min/max/mean/std

# B8: Flush và đóng dataset để lưu file
dataset.FlushCache()
dataset = None   # None = đóng file và giải phóng bộ nhớ
```

### 14.2.2. Đọc raster và xem thông tin metadata

Sau khi tạo file raster, bước tiếp theo là đọc và kiểm tra thông tin metadata của nó. GDAL cung cấp các phương thức để truy cập thông tin cơ bản về dataset như số lượng band, kích thước (số cột và hàng), driver được sử dụng, hệ tọa độ và GeoTransform. Việc hiểu metadata giúp bạn xác định cấu trúc dữ liệu trước khi xử lý.


```python
# Mở raster file đã tạo
ds = gdal.Open(output_tif, gdal.GA_ReadOnly)  # GA_ReadOnly hoặc GA_Update

# --- Thông tin cơ bản của Dataset ---
print(f"Số band    : {ds.RasterCount}")
print(f"Kích thước : {ds.RasterXSize} cols x {ds.RasterYSize} rows")
print(f"Driver     : {ds.GetDriver().ShortName}")

ds = None  # Đóng dataset
```

### 14.2.3. Đọc và ghi dữ liệu nhiều kênh

Dữ liệu raster thường có nhiều band (kênh), đặc biệt là ảnh vệ tinh đa phổ với các kênh như Red, Green, Blue, NIR, SWIR. Mỗi band lưu trữ một ma trận 2D có thể đọc thành numpy array. GDAL cho phép đọc toàn bộ band hoặc chỉ một vùng nhỏ (window) để tối ưu bộ nhớ. Khi tạo raster đa band, bạn cần ghi dữ liệu vào từng band và có thể thiết lập color interpretation để chỉ định ý nghĩa của mỗi band.


```python
# --- Đọc band dữ liệu thành numpy array ---
ds = gdal.Open(output_tif)
band = ds.GetRasterBand(1)

# ReadAsArray(): đọc toàn bộ band
data = band.ReadAsArray()

# ReadAsArray(xoff, yoff, xsize, ysize): đọc một vùng (window) cụ thể
# Đọc cửa sổ 20x20 pixel từ vị trí (10, 10)
window = band.ReadAsArray(10, 10, 20, 20)
print(f"\nCửa sổ 20x20 tại (10,10): shape={window.shape}, mean={window.mean():.2f}")

ds = None

# --- Tạo raster đa band (RGB) ---
output_rgb = os.path.join(raster_path, 'rgb_sample.tif')
driver = gdal.GetDriverByName('GTiff')
ds_rgb = driver.Create(output_rgb, 50, 50, 3, gdal.GDT_Byte)  # 3 bands, 8-bit

# Thiết lập georeference
ds_rgb.SetGeoTransform((105.8, 0.01, 0, 21.5, 0, -0.01))
srs = osr.SpatialReference()
srs.ImportFromEPSG(4326)
ds_rgb.SetProjection(srs.ExportToWkt())

# Ghi từng band (R, G, B) với dữ liệu khác nhau
np.random.seed(0)
for i, color in enumerate(['Red', 'Green', 'Blue'], start=1):
    arr = np.random.randint(0, 256, (50, 50), dtype=np.uint8)
    ds_rgb.GetRasterBand(i).WriteArray(arr)
    ds_rgb.GetRasterBand(i).SetColorInterpretation(
        [gdal.GCI_RedBand, gdal.GCI_GreenBand, gdal.GCI_BlueBand][i - 1]
    )

ds_rgb.FlushCache()
ds_rgb = None
```

### 14.2.4. Reproject raster sang hệ tọa độ mới

Reprojection là quá trình chuyển đổi dữ liệu raster từ hệ tọa độ này sang hệ tọa độ khác. `gdal.Warp()` là công cụ mạnh mẽ cho phép reproject, resample, clip và merge raster trong một lệnh. Khi reproject, bạn cần chọn thuật toán nội suy (resampling) phù hợp: Nearest Neighbour cho dữ liệu rời rạc (landcover), Bilinear/Cubic cho dữ liệu liên tục (DEM, nhiệt độ). Việc chuyển đổi sang hệ tọa độ như UTM giúp tính toán diện tích và khoảng cách chính xác hơn.


```python
# Reproject DEM từ WGS84 (EPSG:4326) sang UTM Zone 48N (EPSG:32648)
# UTM phù hợp hơn khi cần tính toán diện tích, khoảng cách chính xác

output_utm = os.path.join(raster_path, 'dem_utm48n.tif')

# gdal.Warp: công cụ all-in-one cho reproject, resample, clip raster
result = gdal.Warp(
    output_utm,
    output_tif,
    dstSRS='EPSG:32648',          # Hệ tọa độ đích
    resampleAlg=gdal.GRA_Bilinear, # Thuật toán nội suy (Bilinear cho dữ liệu liên tục)
    # resampleAlg options: GRA_NearestNeighbour, GRA_Bilinear, GRA_Cubic, GRA_Average
    format='GTiff'
)
result = None  # Đóng dataset
```

### 14.2.5. Clip raster theo bounding box

Clip (cắt) raster là thao tác cắt bỏ phần dữ liệu nằm ngoài vùng quan tâm (Area of Interest - AOI), chỉ giữ lại phần cần thiết. Điều này giúp giảm kích thước file, tăng tốc độ xử lý và tiết kiệm bộ nhớ. `gdal.Translate()` với tham số `projWin` cho phép clip theo bounding box được định nghĩa bởi tọa độ (xmin, ymax, xmax, ymin). Phép clip đặc biệt hữu ích khi làm việc với ảnh vệ tinh phủ vùng lớn nhưng bạn chỉ cần phân tích một khu vực nhỏ.


```python
# Clip raster theo bounding box (projectedExtents)
# Cắt lấy vùng nhỏ: góc Tây-Nam của DEM mẫu
output_clip = os.path.join(raster_path, 'dem_clipped.tif')

clip_box = [105.8, 21.7, 106.5, 21.6]  # (xmin, ymax, xmax, ymin) dạng projWin

result = gdal.Translate(
    output_clip,
    output_tif,
    projWin=clip_box,   # Bounding box clip
    format='GTiff'
)
result = None

# Kiểm tra kết quả
ds_orig = gdal.Open(output_tif)
ds_clip = gdal.Open(output_clip)
print("=== SO SÁNH TRƯỚC VÀ SAU CLIP ===")
print(f"Gốc   : {ds_orig.RasterXSize} x {ds_orig.RasterYSize} pixels")
print(f"Clipped: {ds_clip.RasterXSize} x {ds_clip.RasterYSize} pixels")

ds_orig = None
ds_clip = None
```

### 14.3.1. Đọc vector và xem thông tin layer

OGR (phần vector của GDAL) cho phép đọc dữ liệu vector từ nhiều định dạng khác nhau như Shapefile, GeoJSON, GeoPackage. Khi mở một DataSource, bạn có thể truy cập các layer chứa features. Mỗi layer có thông tin về số lượng features, kiểu geometry (Point, LineString, Polygon) và cấu trúc thuộc tính (fields). Việc kiểm tra metadata của layer giúp bạn hiểu cấu trúc dữ liệu trước khi xử lý hoặc truy vấn.


```python
# Đọc file GeoJSON tỉnh/thành Việt Nam
vector_path = r"J:\My Drive\geocourse_data\vector"
geojson_path = os.path.join(vector_path, 'vinhphuc_districts.geojson')
ds = ogr.Open(geojson_path, 0)  # 0 = read-only, 1 = read-write

if ds is None:
    raise FileNotFoundError(f"Không thể mở file: {geojson_path}")

# --- Thông tin DataSource ---
print(f"Driver     : {ds.GetDriver().GetName()}")
print(f"Số layer   : {ds.GetLayerCount()}")

# --- Thông tin Layer ---
layer = ds.GetLayer(0)  # Lấy layer đầu tiên
print(f"Tên layer       : {layer.GetName()}")
print(f"Số feature      : {layer.GetFeatureCount()}")
print(f"Kiểu geometry   : {ogr.GeometryTypeToName(layer.GetGeomType())}")

# --- Danh sách các trường thuộc tính ---
layer_defn = layer.GetLayerDefn()
print(f"\n=== Các trường thuộc tính ({layer_defn.GetFieldCount()} fields) ===")
for i in range(layer_defn.GetFieldCount()):
    field = layer_defn.GetFieldDefn(i)
    print(f"  [{i}] {field.GetName():<25} | {field.GetFieldTypeName(field.GetType())}")
ds = None
```

### 14.3.2. Duyệt qua các features và thuộc tính

Mỗi Feature trong OGR bao gồm hai thành phần chính: geometry (hình học không gian) và attributes (thuộc tính mô tả). Bạn có thể duyệt qua từng feature trong layer để đọc thông tin, trích xuất geometry, tính toán các giá trị như diện tích, chu vi, hoặc lọc dữ liệu theo điều kiện. `GetGeometryRef()` trả về geometry của feature, `GetField()` trả về giá trị của một trường thuộc tính. Việc duyệt tuần tự (iterator) tiết kiệm bộ nhớ hơn so với đọc toàn bộ dữ liệu vào memory.


```python
# Duyệt qua features và trích xuất thông tin
ds = ogr.Open(geojson_path)
layer = ds.GetLayer(0)

print(f"{'ID':<5} {'Tên tỉnh':<30} {'Geometry Type':<15} {'Diện tích xấp xỉ':>18}")
print("-" * 72)

areas = []
layer.ResetReading()  # Đặt lại con trỏ về đầu layer
for feature in layer:
    geom = feature.GetGeometryRef()
    
    # Lấy thuộc tính theo tên field
    # Thử các tên field phổ biến
    name = (feature.GetField('NAME_1') or 
            feature.GetField('VARNAME_1') or 
            feature.GetField('name') or 
            f"Feature {feature.GetFID()}")
    
    geom_type = ogr.GeometryTypeToName(geom.GetGeometryType())
    
    # Tính diện tích trong CRS gốc (degrees) — chỉ để minh họa
    area = geom.Area()
    areas.append((name, area))
    
print(f"  Đã đọc {len(areas)} features")
print(f"\n  Top 5 tỉnh lớn nhất (theo diện tích độ):")
for name, area in sorted(areas, key=lambda x: x[1], reverse=True)[:5]:
    print(f"    {name:<30} {area:.4f} sq.deg")

ds = None
```

### 14.3.3. Tạo vector file mới từ đầu

OGR cho phép tạo file vector mới từ đầu với bất kỳ định dạng nào được hỗ trợ. Quy trình bao gồm: (1) tạo DataSource với driver tương ứng, (2) tạo layer với geometry type và CRS, (3) định nghĩa schema (các trường thuộc tính), (4) tạo từng feature với geometry và attributes, (5) lưu file. Khi tạo geometry, bạn sử dụng các lớp như `ogr.Geometry(ogr.wkbPoint)` và thêm tọa độ với `AddPoint()`. Việc tạo vector programmatically hữu ích khi chuyển đổi dữ liệu từ CSV, database hoặc kết quả phân tích.


```python
# Dữ liệu minh họa các thành phố lớn Việt Nam. Cần kiểm tra và cập nhật số liệu dân số chính xác từ nguồn đáng tin cậy khi sử dụng thực tế.
cities_data = [
    {'name': 'Hà Nội',    'pop': 8246600, 'lon': 105.8542, 'lat': 21.0285},
    {'name': 'TP.HCM',    'pop': 8993082, 'lon': 106.6297, 'lat': 10.8231},
    {'name': 'Hải Phòng', 'pop': 2028514, 'lon': 106.6881, 'lat': 20.8449},
    {'name': 'Đà Nẵng',   'pop': 1134310, 'lon': 108.2022, 'lat': 16.0544},
    {'name': 'Cần Thơ',   'pop': 1282937, 'lon': 105.7469, 'lat': 10.0452},
    {'name': 'Huế',       'pop':  455230, 'lon': 107.5905, 'lat': 16.4637},
    {'name': 'Nha Trang', 'pop':  423000, 'lon': 109.1967, 'lat': 12.2585},
]

# Tạo GeoJSON output
output_cities = os.path.join(vector_path, 'cities_ogr.geojson')

# Xóa file cũ nếu tồn tại (OGR không tự ghi đè)
if os.path.exists(output_cities):
    driver_del = ogr.GetDriverByName('GeoJSON')
    driver_del.DeleteDataSource(output_cities)

# --- Khởi tạo DataSource và Layer ---
driver = ogr.GetDriverByName('GeoJSON')
out_ds = driver.CreateDataSource(output_cities)

# Thiết lập CRS
srs = osr.SpatialReference()
srs.ImportFromEPSG(4326)

# Tạo layer với geometry type = wkbPoint
out_layer = out_ds.CreateLayer('cities', srs=srs, geom_type=ogr.wkbPoint)

# --- Định nghĩa các trường thuộc tính ---
out_layer.CreateField(ogr.FieldDefn('name', ogr.OFTString))         # String
out_layer.CreateField(ogr.FieldDefn('population', ogr.OFTInteger))  # Integer

# --- Tạo từng Feature ---
layer_defn = out_layer.GetLayerDefn()
for city in cities_data:
    feature = ogr.Feature(layer_defn)
    
    # Tạo geometry Point
    point = ogr.Geometry(ogr.wkbPoint)
    point.AddPoint(city['lon'], city['lat'])
    feature.SetGeometry(point)
    
    # Gán thuộc tính
    feature.SetField('name', city['name'])
    feature.SetField('population', city['pop'])
    
    out_layer.CreateFeature(feature)
    feature = None  # Giải phóng bộ nhớ

out_ds.FlushCache()
out_ds = None
```

### 14.3.4. Spatial filter và Attribute filter

OGR cung cấp hai loại filter mạnh mẽ để truy vấn dữ liệu hiệu quả mà không cần load toàn bộ features vào memory. **Attribute filter** (`SetAttributeFilter`) giống SQL WHERE clause, cho phép lọc features dựa trên giá trị thuộc tính (ví dụ: population > 1000000). **Spatial filter** (`SetSpatialFilter`) lọc features nằm trong một vùng không gian xác định (bounding box hoặc geometry phức tạp). Kết hợp hai loại filter giúp truy vấn chính xác và nhanh chóng trên datasets lớn, đặc biệt hữu ích cho web GIS và spatial analysis.


```python
# --- Attribute Filter: lọc theo dân số ---
ds = ogr.Open(output_cities)
layer = ds.GetLayer(0)

# SQL-like WHERE clause
layer.SetAttributeFilter("population > 1000000")
print(f"Số thành phố > 1 triệu dân: {layer.GetFeatureCount()}")
for feat in layer:
    print(f"  {feat.GetField('name'):<15} | dân số: {feat.GetField('population'):,}")

# Xóa filter
layer.SetAttributeFilter(None)

# Tạo geometry bounding box
bbox_north = ogr.Geometry(ogr.wkbPolygon)
ring = ogr.Geometry(ogr.wkbLinearRing)
ring.AddPoint(102.0, 19.0)
ring.AddPoint(110.0, 19.0)
ring.AddPoint(110.0, 24.0)
ring.AddPoint(102.0, 24.0)
ring.AddPoint(102.0, 19.0)
bbox_north.AddGeometry(ring)

layer.SetSpatialFilter(bbox_north)
print(f"\nSố thành phố trong Miền Bắc (lat > 19°N): {layer.GetFeatureCount()}")
for feat in layer:
    geom = feat.GetGeometryRef()
    print(f"  {feat.GetField('name'):<15} | lat={geom.GetY():.4f}°N")

layer.SetSpatialFilter(None)
ds = None
```

### 14.3.5. Chuyển đổi định dạng vector (Format Conversion)

OGR hỗ trợ chuyển đổi linh hoạt giữa hơn 50 định dạng vector khác nhau như GeoJSON, Shapefile, GeoPackage, KML, CSV, và thậm chí databases như PostGIS. `CopyDataSource()` là phương thức đơn giản nhất để copy toàn bộ DataSource sang format mới, giữ nguyên geometry, attributes và CRS. Việc chuyển đổi định dạng hữu ích khi tích hợp dữ liệu từ nhiều nguồn, tối ưu storage (GeoPackage nhỏ hơn Shapefile), hoặc tương thích với các công cụ GIS khác nhau.


```python
# Chuyển đổi GeoJSON → Shapefile và GeoPackage

def convert_vector(src_path, dst_path, dst_format):
    """Chuyển đổi vector file từ format này sang format khác."""
    src_ds = ogr.Open(src_path)
    src_layer = src_ds.GetLayer(0)
    
    dst_driver = ogr.GetDriverByName(dst_format)
    
    # Xóa file đích nếu đã tồn tại
    if os.path.exists(dst_path):
        dst_driver.DeleteDataSource(dst_path)
    
    # Dùng CopyDataSource để copy toàn bộ
    dst_ds = dst_driver.CopyDataSource(src_ds, dst_path)
    dst_ds.FlushCache()
    dst_ds = None
    src_ds = None
    return dst_path

# GeoJSON → Shapefile
shp_path = os.path.join(vector_path, 'cities_ogr.shp')
convert_vector(output_cities, shp_path, 'ESRI Shapefile')
print(f"✅ GeoJSON → Shapefile: {shp_path}")

# GeoJSON → GeoPackage
gpkg_path = os.path.join(vector_path, 'cities_ogr.gpkg')
convert_vector(output_cities, gpkg_path, 'GPKG')
print(f"✅ GeoJSON → GeoPackage: {gpkg_path}")
```

## Tóm tắt

Bạn đã hoàn thành Bài 14 và học được GDAL - thư viện nền tảng mạnh mẽ nhất trong hệ sinh thái GIS toàn cầu.

### Các khái niệm chính đã nắm vững:

| Thành phần | Khái niệm | API chính |
|---|---|---|
| **GDAL** | Raster: Dataset → Band → Array | `gdal.Open()`, `gdal.Warp()`, `gdal.Translate()` |
| **OGR** | Vector: DataSource → Layer → Feature | `ogr.Open()`, `CreateLayer()`, `CreateFeature()` |

### Kỹ năng bạn có thể áp dụng:
- ✅ **Tạo raster từ numpy array** và ghi ra GeoTIFF với GeoTransform, Projection
- ✅ **Đọc metadata raster** — kích thước, GeoTransform, CRS, thống kê band
- ✅ **Reproject & Clip raster** với `gdal.Warp()` và `gdal.Translate()`
- ✅ **Đọc/ghi vector** — GeoJSON, Shapefile, GeoPackage với OGR
- ✅ **Tạo features** từ đầu với geometry và thuộc tính
- ✅ **Lọc không gian** (SetSpatialFilter) và **lọc thuộc tính** (SetAttributeFilter)
- ✅ **Chuyển đổi định dạng** vector giữa 30+ formats
- ✅ **Transform tọa độ** điểm và reproject toàn bộ dataset với OSR
