# Bài 15: Rasterio - Xử lý Dữ liệu Raster và Ảnh Vệ tinh

Rasterio là thư viện Python chuyên nghiệp cho việc đọc, ghi và xử lý dữ liệu raster địa lý - từ ảnh vệ tinh đến mô hình độ cao số.

## 15.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

- **Đọc và hiểu dữ liệu raster** với metadata và properties chi tiết
- **Xử lý coordinate systems** và transformations cho dữ liệu raster
- **Trực quan hóa dữ liệu raster** với matplotlib và các hàm từ rasterio
- **Thực hiện tính toán trên dữ liệu raster** và band operations phức tạp
- **Optimize memory usage** với windowed reading cho big datasets
- **Viết và reproject rasters** với các hàm nâng cao
- **Tích hợp Rasterio** với NumPy và các thư viện khác.


- **Import thư viện cần thiết**


```python
# Import thư viện cốt lõi
import rasterio
from rasterio.plot import show, show_hist
from rasterio.windows import Window
from rasterio.mask import mask
from rasterio.merge import merge
from rasterio.warp import calculate_default_transform, reproject
from rasterio.enums import Resampling
from rasterio.transform import from_bounds, from_origin, Affine
import numpy as np
from pyproj import CRS
import matplotlib.pyplot as plt
print(f"�️ Rasterio version: {rasterio.__version__}")
```

## 15.2. Tạo và lưu dữ liệu
Hiểu cách tạo dữ liệu và lưu dữ liệu dưới dạng GeoTIFF cơ bản. Trong phần này, chúng ta sẽ tìm hiểu

- **Tạo raster datasets** từ đầu
- **Thiết lập meta** cho dữ liệu raster
- **Lưu dữ liệu** ra file


```python
import numpy as np 
from rasterio.transform import from_bounds
from pyproj import CRS 
import rasterio
```

### 15.2.1. Tạo dữ liệu cho vùng nghiên cứu


```python
# Đầu tiên thiết lập bounding box cho khu vực nghiên cứu. Ví dụ đây là khu vực thành phố vĩnh yên, vĩnh phúc
bounds = (105.5491712751346, 21.265278731790968, 105.64026044494432, 21.34612710736172)
# Tạo lưới tọa độ
height, width = 300, 338  # Resolution hợp lý (~30m)
x = np.linspace(bounds[0], bounds[2], width) # Tọa độ X
y = np.linspace(bounds[1], bounds[3], height) # Tọa độ Y
X, Y = np.meshgrid(x, y) # Tạo lưới 2D
# Tạo dữ liệu raster mẫu (giá trị ngẫu nhiên từ 0 đến 255, trong thực tế thì sẽ là dữ liệu đo đạc)
data = np.random.randint(0, 255, (height, width)).astype('uint8')
```

### 15.2.2. Thiết lập meta data cho dữ liệu


```python
# Thiết lập transform
transform = from_bounds(*bounds, width=width, height=height)
# Meta data 
meta = {
    'driver': 'GTiff', # Lưu dữ liệu ra dạndạng GeoTIFF
    'dtype': 'uint8', # Loại dữ liệu lưu là dạng unsigned int 8 bit
    'nodata': None, # Không có giá trị nodata
    'width': width, # Chiều rộng raster
    'height': height, # Chiều cao raster
    'count': 1, # Số band
    'crs': CRS.from_epsg(4326), # Hệ tọa độ WGS84
    'transform': transform, # Biến đổi affine,
    'compress': 'lzw' # Nén dữ liệu để giảm dung lượng file
}
```

### 15.2.3. Lưu dữ liệu 


```python
# Ghi dữ liệu raster vào file GeoTIFF
outfile = r'G:\My Drive\python\python_course\data\outputs\random_raster.tif'
with rasterio.open(outfile, 'w', **meta) as dst:
    dst.write(data, 1)  
```

## 15.3. Đọc file raster và Khám phá Metadata

Hiểu cách mở file raster và kiểm tra các thuộc tính cơ bản là nền tảng để làm việc với dữ liệu raster. Trong phần này, chúng ta sẽ học cách:

- **Đọc raster datasets** và kiểm tra thông tin cơ bản
- **Khám phá metadata**: CRS, transform, resolution, bands
- **Hiểu raster properties**: nodata values, data types, compression
- **Kiểm tra spatial extent** và geographic coverage
- **Phân tích band statistics** và data distribution

### 15.3.1. Đọc dữ liệu file raster


```python
# Đọc dữ liệu ảnh Landsat RBG + NIR (30m resolution)
with rasterio.open(r'G:\My Drive\python\python_course\data\raster\landsat_rgbn.tif') as src:
    img = src.read() # Dữ liệu ở định dạng numpy array. Ta có thể xử lý tương tự như các mảng numpy khác.
    meta = src.meta # dictionary chứa các thông tin thuộc tính như kiểu dữ liệu, chiều cao, rộng và số band,etc.
```

### 15.3.2. Kiểm tra thông tin thuộc tính


```python
# Ta có thể kiểm tra thông tinh thuộc tính khác
print(f'Shape của dữ liệu: {img.shape}')
print(f'Thuộc tính meta:\n {meta}')
res = src.res # Độ phân giải không gian
crs = src.crs # Hệ tọa độ
bounds = src.bounds # Giới hạn không gian
print(f'Độ phân giải không gian: {res}')
print(f'Hệ tọa độ: {crs}')
print(f'Giới hạn không gian: {bounds}')
```


```python
# Tính các chỉ số thống kê cơ bản
min_val = np.nanmin(img, axis=(1,2)) # Tính giá trị nhỏ nhất cho từng band
max_val = np.nanmax(img, axis=(1,2)) # Tính giá trị lớn nhất cho từng band
std_val = np.nanstd(img, axis=(1,2)) # Tính độ lệch chuẩn cho từng band
mean_val = np.nanmean(img, axis=(1,2)) # Tính giá trị trung bình cho từng band
print(f'Min values per band: {min_val}')
print(f'Max values per band: {max_val}')
print(f'Std values per band: {std_val}')
print(f'Mean values per band: {mean_val}')
```

### 15.3.3. Trực quan hóa dữ liệu


```python
import matplotlib.pyplot as plt
# Ví dụ như ta tính toán chỉ số NDVI 
ndvi = (img[3] - img[2]) / (img[3] + img[2])
# Hiển thị bản đồ NDVI
fig, ax = plt.subplots(1,1, figsize=(10,10))
plot = ax.imshow(ndvi, vmin=-0.6, vmax=0.6)
ax.set_xticks([])
ax.set_yticks([])
ax.set_title('NDVI Map')
cbar = fig.colorbar(plot, ax=ax, pad=0.05, shrink=0.5, extend='both', 
                    orientation='horizontal', ticks=np.arange(-0.6,0.61, 0.2))
cbar.ax.set_title('NDVI')
plt.show()
```

### 15.3.4. Lưu dữ liệu


```python
# Thêm ndvi band vào với img và lưu file
img = np.concatenate([img, ndvi[np.newaxis, ...]], axis=0)
outfile = r'G:\My Drive\python\python_course\data\raster\landsat_rgbn_ndvi.tif'
# Cập nhật meta vì có thêm band mới là ndvi
meta.update({
    'count': len(img)
})
band_name = ['blue', 'green', 'red', 'nir', 'ndvi']
with rasterio.open(outfile, 'w', **meta) as dst:
    for i in range(1, len(img)+1):
        dst.write(img[i-1], i) # index bắt đầu từ 1 
        dst.set_band_description(i, band_name[i-1]) # ta có thể thêm mô tả cho mỗi band 
```

## 15.4. Transform và CRS Manipulation 

Một trong những kỹ năng quan trọng nhất khi làm việc với raster là hiểu và manipulate **coordinate transforms** và **coordinate reference systems (CRS)**. Phần này bao gồm:

- **Hiểu Affine Transform**: Ma trận biến đổi pixel ↔ world coordinates
- **Tạo ra custom transforms**: Từ bounds, origin, resolution
- **CRS operations**: Chuyển đổi, so sánh
- **Georeferencing**: Gắn coordinates cho ungeoreferenced rasters  
- **Transform arithmetic**: Scaling, translation, rotation
- **Precision và accuracy**: Handling coordinate precision issues


```python
from rasterio.transform import from_bounds, from_origin, Affine
from rasterio.warp import transform_bounds
```

### 15.4.1. Tạo transform từ bounding box


```python
# Xác định bounding box và kích thước raster
bounds = (105.5491712751346, 21.265278731790968, 105.64026044494432, 21.34612710736172)
width, height = 338, 300
# Sử dụng from_bounds
transform = from_bounds(*bounds, width=width, height=height)
```

### 15.4.2. Tạo transform từ origin


```python
# Xác định kich thước pixel và upper left origin (west, north)
pixel_size_x = (bounds[2] - bounds[0]) / width
pixel_size_y = (bounds[3] - bounds[1]) / height
transform = from_origin(bounds[0], bounds[3], pixel_size_x, pixel_size_y)
```

### 15.4.3. Tạo transform dùng Affine


```python
# Xác định (west, north) và kích thước raster
transform = Affine(pixel_size_x, 0, bounds[0], 0, -pixel_size_y, bounds[3]) 
```

### 15.4.4. Chuyển bounding box từ CRS này sang CRS khác từ transform_bounds


```python
# Xác định bounding box và kích thước raster
src_crs = CRS.from_epsg(4326) # WGS84
dst_crs = CRS.from_epsg(3857) # Web Mercator
transformed_bounds = transform_bounds(src_crs, dst_crs, *bounds)
print(f'bounds ban đầu {bounds}')
print(f'Transformed bounds: {transformed_bounds}')
```

### 15.4.5. Chuyển đổi tọa độ pixels sang tọa độ thế giới (world coordinate) và ngược lại

- **Từ tọa độ pixel sang tọa độ thế giới**


```python
# Chuyển coordinate transformation từ pixel sang coordinate thế giới
# Lấy một vài pixel coordinates để test
test_pixels = [(0,0), (width//2, height//2), (width-1, height-1)] # Test pixel tại góc trên trái, giữa và góc dưới phải
for px, py in test_pixels:
    x_world, y_world = transform * (px, py)
    print(f"Pixel coordinates ({px}, {py}) -> World coordinates ({x_world:.6f}, {y_world:.6f})")

```

- **Từ tọa độ thế giới sang tọa độ pixel**


```python
# Chuyển từ World sang pixel coordinates
for col, row in test_pixels:
    world_x, world_y = transform * (col, row)
    # Sử dụng inverse transform
    inv_transform = ~transform  # Nghịch đảo transform
    back_col, back_row = inv_transform * (world_x, world_y)
    print(f"World({world_x:.4f}°, {world_y:.4f}°) → Pixel({back_col:.1f}, {back_row:.1f})")
```

### 15.4.6. Thông tin về CRS


```python
# Làm việc với CRS (Coordinate Reference Systems)
# Thông tin về CRS hiện tại
print(f"       CRS hiện tại: {crs}")
print(f"      • Authority: {crs.to_authority()}")
print(f"      • EPSG code: {crs.to_epsg()}")
print(f"      • Is geographic: {crs.is_geographic}")
print(f"      • Is projected: {crs.is_projected}")
print(f"      • Units: {crs.linear_units}")

# Tạo CRS mới (UTM Zone 48N cho miền Bắc Việt Nam)
utm48n = CRS.from_epsg(32648)
print(f"\n   UTM Zone 48N (EPSG:32648):")
print(f"      • Name: {utm48n.name}")
print(f"      • Authority: {utm48n.to_authority()}")
print(f"      • Is projected: {utm48n.is_projected}")

# VN-2000 CRS cho Việt Nam  
vn2000 = CRS.from_epsg(3405)
print(f"\n   🇻🇳 VN-2000 (EPSG:3405):")
print(f"      • Name: {vn2000.name}")
print(f"      • Authority: {vn2000.to_authority()}")
```

## 15.5. Trực quan hóa Raster Data

Trực quan hóa là bước quan trọng để hiểu và phân tích dữ liệu raster.

### 15.5.1. Trực quan hóa dùng hàm `show` từ rasterio


```python
from rasterio.plot import show
# Đọc dữ liệu ảnh landsat and hiển thị sử dụng show từ rasterio plot
file = r'G:\My Drive\python\python_course\data\raster\landsat_rgbn.tif'
with rasterio.open(file) as src:
    fig, ax = plt.subplots(1,1, figsize=(10,10))
    show(src.read([3, 2, 1]), transform=src.transform, adjust='linear', ax=ax)
    ax.set_xticks([])
    ax.set_yticks([])
```

### 15.5.2. Trực quan hóa dùng matplotlib


```python
# Normalize each band (optional: clip outliers)
def normalize(band):
    p2, p98 = np.nanpercentile(band, (5, 95))  # clip 5–95% range
    return np.clip((band - p2) / (p98 - p2), 0, 1)
# Đọc dữ liệu ảnh landsat và hiển thị sử dụng matplotlib
with rasterio.open(file) as src:
    img = src.read([3, 2, 1])  # RGB
    rgb = np.stack([normalize(b) for b in img], axis=0)
    rgb = np.transpose(rgb, (1, 2, 0))  # reshape for imshow
    fig, ax = plt.subplots(figsize=(10, 10))
    extent = rasterio.plot.plotting_extent(src)
    ax.imshow(rgb, extent=extent)
    ax.set_xticks([]) # Bỏ trục tọa độ
    ax.set_yticks([]) # Bỏ trục tọa độ
```


```python
# Đọc dữ liệu ảnh Landsat RBG + NIR (30m resolution)
with rasterio.open(r'G:\My Drive\python\python_course\data\raster\landsat_rgbn.tif') as src:
    img = src.read() # Dữ liệu ở định dạng numpy array. Ta có thể xử lý tương tự như các mảng numpy khác.
    meta = src.meta # dictionary chứa các thông tin thuộc tính như kiểu dữ liệu, chiều cao, rộng và số band,etc.
# Ví dụ như ta tính toán chỉ số NDVI 
ndvi = (img[3] - img[2]) / (img[3] + img[2])
# Hiển thị bản đồ NDVI
fig, ax = plt.subplots(1,1, figsize=(10,10))
plot = ax.imshow(ndvi, vmin=-0.6, vmax=0.6)
ax.set_xticks([])
ax.set_yticks([])
ax.set_title('NDVI Map')
cbar = fig.colorbar(plot, ax=ax, pad=0.05, shrink=0.5, extend='both', 
                    orientation='horizontal', ticks=np.arange(-0.6,0.61, 0.2))
cbar.ax.set_title('NDVI')
plt.show()
```

## 15.6. Tính toán và Biến đổi Raster

Raster calculations và transformations là trái tim của phân tích raster. Phần này bao gồm:

- **Band arithmetic**: Cộng, trừ, nhân, chia các bands
- **Spectral indices**: NDVI, NDWI, SAVI và các chỉ số khác
- **Reclassification**: Phân loại lại giá trị pixels
- **Mathematical functions**: Log, sqrt, trigonometric operations

### 15.6.1. Tính toán cơ bản


```python
# Đọc dữ liệu ảnh landsat
file = r'G:\My Drive\python\python_course\data\raster\landsat_rgbn.tif'
with rasterio.open(file) as src:
    data = src.read() 
    out_meta = src.meta
    data = np.sqrt(data) * 5 + 10 -30 # Một số biến đổi đơn giản trên raster data
```

### 15.6.2. Tính toán chỉ số thực vật


```python
with rasterio.open(file) as src:
    data = src.read() 
    out_meta = src.meta

# Chúng ta có thể tính toán trên các mảng numpy như bình thường
# Ví dụ tính chỉ số NDVI
ndvi = (data[3] - data[2]) / (data[3] + data[2])
# Tính toán chỉ số SAVI 
savi = ((data[3] - data[2]) / (data[3] + data[2] + 0.5)) * (1.0 + 0.5)
# Tính toán chỉ số NDWI
ndwi = (data[1] - data[3]) / (data[1] + data[3])   

# Ta có thể lưu dữ liệu ra file  sử dụng out_meta (cập nhật lại số band) như các phần bên trên.
# Ví dụ stack các chỉ số thực vật vào với nhau
indices = np.stack([ndvi, savi, ndwi], axis=0)
outfile = r'G:\My Drive\python\python_course\data\outputs\landsat_indices.tif'
out_meta.update({
    'count': indices.shape[0] # Cập nhật số band mới
})
with rasterio.open(outfile, 'w', **out_meta) as dst:
    for i in range(1, indices.shape[0]+1):
        dst.write(indices[i-1], i)
```

### 15.6.3. Phân loại sức khỏe thực vật


```python
# Vị dụ phân loại ndvi thành các lớp sức khỏe thực vật (healthy [0.6, 1], moderate [0.2, 0.6), unhealthy [-0.2, 0.2), barren [-1, -0.2))
def ndvi_reclass(a):
    if a >= 0.6:
        return 4  # healthy
    elif 0.2 <= a < 0.6:
        return 3  # moderate
    elif -0.2 <= a < 0.2:
        return 2  # unhealthy
    else:
        return 1  # barren
# Vectorize the function to apply it on numpy array
ndvi_classes = np.vectorize(ndvi_reclass)(ndvi)
# Lưu kết quả phân loại ra file
outfile = r'G:\My Drive\python\python_course\data\outputs\ndvi_classes.tif'
out_meta.update({
    'count': 1  # Chỉ có 1 band cho kết quả phân loại
})
with rasterio.open(outfile, 'w', **out_meta) as dst:
    dst.write(ndvi_classes, 1)
```

## 5. Windowed Reading và Quản lý Bộ nhớ

Làm việc hiệu quả với large raster datasets đòi hỏi kỹ thuật tối ưu để tránh memory overflow. Phần này bao gồm:

- **Windowed reading**: Đọc data theo chunks/tiles
- **Memory-efficient processing**: Xử lý từng phần thay vì load toàn bộ
- **Streaming operations**: Operations trên data streams
- **Block-based processing**: Sử dụng internal tiling structure
- **Out-of-core processing**: Xử lý datasets lớn hơn RAM
- **Performance optimization**: Benchmarking và profiling


```python
from rasterio.windows import Window
from rasterio.windows import from_bounds
```


```python
file = r'G:\My Drive\python\python_course\data\landsat_rgbn.tif'
with rasterio.open(file) as src:
    # Đọc window nhỏ
    window_size = 100
    window = Window(50, 50, window_size, window_size)  # col_off, row_off, width, height
    window_data = src.read(window=window) # Đọc tất cả các band trong window, hoặc đọc một số band cụ thể bằng cách thêm parameter indexes=[1,2,3]
    # Tính meta cho window
    out_meta = src.meta.copy()
    # Cập nhật lại chiều cao, rộng và transform cho window
    out_meta.update({
        "height": window.height,
        "width": window.width,
        "transform": src.window_transform(window)
    })
    # Từ window có thể tính toán bounding box
    window_bounds = rasterio.windows.bounds(window, src.transform)
# Từ out_meta và window_data ta có thể lưu window ra file mới nếu cần thiết
outfile = r'G:\My Drive\python\python_course\data\landsat_window.tif'
with rasterio.open(outfile, 'w', **out_meta) as dst:
    dst.write(window_data)
```


```python
# Sử dụng block-based reading để xử lý raster lớn. Ví dụ này minh họa và raster không quá lớn.
file = r'G:\My Drive\python\python_course\data\landsat_rgbn.tif'
with rasterio.open(file) as src:
    # Lấy kích thước block
    block_shapes = src.block_shapes
    # Duyệt qua từng block
    for ji, window in src.block_windows(1):  # Chỉ đọc band 1
        block_data = src.read(1, window=window)  # Đọc dữ liệu block
        # Thực hiện các phép tính trên block_data ở đây
        # Ví dụ tính giá trị trung bình của block
        block_mean = np.nanmean(block_data)
        print(f'Block {ji} mean value: {block_mean}')
```


```python
def block_process(file, block_size=100, outfile=None):
    """ Hàm xử lý raster theo block để tiết kiệm bộ nhớ.
    Args:
        file (str): Đường dẫn đến file raster đầu vào.
        block_size (int): Kích thước block để đọc và xử lý.
        outfile (str, optional): Đường dẫn đến file raster đầu ra. Nếu None, không lưu file.
    Returns:
        np.ndarray: Mảng dữ liệu raster đã được xử lý.
    """
    if not isinstance(file, str):
        raise ValueError("File path must be a string.")
    with rasterio.open(file) as src:
        height = src.height
        width = src.width
        out_meta = src.meta.copy()
        new_data = np.zeros((height, width), dtype=src.dtypes[0])
        for row in range(0, height, block_size):
            for col in range(0, width, block_size):
                window = Window(col, row, min(block_size, width - col), min(block_size, height - row))
                block_data = src.read(1, window=window)  # Đọc band 1
                # Thực hiện các phép tính trên block_data ở đây
                block_data = block_data * 2  # Ví dụ: nhân đôi giá trị pixel
                new_data[row:row + window.height, col:col + window.width] = block_data
    out_meta.update({
        'count': 1,
    })
    if outfile:
        with rasterio.open(outfile, 'w', **out_meta) as dest:
            dest.write(new_data, 1)
    return new_data
# Sử dụng hàm block_process
test = block_process(r'G:\My Drive\python\python_course\data\landsat_rgbn.tif', block_size=200)
test.shape
```


```python
# Đọc dữ liệu dựa trên bounding box tọa độ địa lý
bbox = (105.56264600439638, 21.305702919576344, 105.58959546291997, 21.332652378099926)

with rasterio.open(file) as src:
    # Chuyển geographic bounds thành pixel window
    window = from_bounds(*bbox, transform=src.transform)
    data = src.read(window=window) # Đọc tất cả các band trong window
    # Tính meta cho window
    out_meta = src.meta.copy()
    out_meta.update({
        "height": window.height,
        "width": window.width,
        "transform": src.window_transform(window)
    })
```

## 6. Writing, Reprojection và Advanced Operations

Phần cuối này bao gồm các kỹ thuật nâng cao để xuất, biến đổi và tối ưu hóa raster data:

- **Writing optimized rasters**: COG, tiling, compression strategies
- **Coordinate system transformation**: Reprojection với resampling
- **Raster warping và georeferencing**: Geometric corrections
- **Mosaicing và merging**: Kết hợp multiple rasters
- **Clipping và masking**: Cắt theo vector boundaries
- **Performance benchmarking**: So sánh hiệu suất các operations


```python
# Lưu dữ liệu ra file với các compression options để giảm dung lượng file
# Một vài options phổ biến: 'lzw', 'deflate', 'jpeg'
outfile = r'G:\My Drive\python\python_course\data\landsat_copressed.tif'
# Ví dụ đọc và lưu lại với nén LZW cho 1 band 
file = r'G:\My Drive\python\python_course\data\landsat_rgbn.tif'
with rasterio.open(file) as src:
    data = src.read(1)  # Đọc band 1
    out_meta = src.meta.copy()
    out_meta.update({
        'compress': 'lzw'  # Thêm option nén LZW
    })
    with rasterio.open(outfile, 'w', **out_meta) as dest:
        dest.write(data, 1)
```


```python
from rasterio.warp import calculate_default_transform, reproject, Resampling
# Reproject raster từ WGS84 sang UTM Zone 48N
src_crs = CRS.from_epsg(4326)  # WGS84
dst_crs = CRS.from_epsg(32648)  # UTM Zone 48N
file = r'G:\My Drive\python\python_course\data\landsat_rgbn.tif'
outfile = r'G:\My Drive\python\python_course\data\landsat_rgbn_utm48n.tif'
with rasterio.open(file) as src:
    transform, width, height = calculate_default_transform(
        src.crs, dst_crs, src.width, src.height, *src.bounds)
    out_meta = src.meta.copy()
    out_meta.update({
        'crs': dst_crs,
        'transform': transform,
        'width': width,
        'height': height,
        "nodata": src.nodata if src.nodata is not None else 0  # Set nodata if not present
    })
    with rasterio.open(outfile, 'w', **out_meta) as dest:
        for i in range(1, src.count + 1):
            reproject(
                source=rasterio.band(src, i),
                destination=rasterio.band(dest, i),
                src_transform=src.transform,
                src_crs=src.crs,
                dst_transform=transform,
                dst_crs=dst_crs,
                resampling=Resampling.nearest)         
```


```python
from rasterio.mask import mask
import geopandas as gpd
poly = gpd.read_file(r"G:\My Drive\python\python_course\data\subset_polygon.geojson")
# Clip raster sử dụng poly 
file = r'G:\My Drive\python\python_course\data\landsat_rgbn.tif'
outfile = r'G:\My Drive\python\python_course\data\landsat_rgbn_clipped.tif'
with rasterio.open(file) as src:
    out_img, out_transform = mask(src, poly.geometry.to_list(), crop=True)
    out_meta = src.meta.copy()
    out_meta.update({
        "height": out_img.shape[1],
        "width": out_img.shape[2],
        "transform": out_transform
    })
    with rasterio.open(outfile, "w", **out_meta) as dest:
        dest.write(out_img)
```

## Tóm tắt

Bạn đã hoàn thành Bài 6 và học được Rasterio - thư viện chuyên nghiệp cho raster data processing trong Python ecosystem.

### Các khái niệm chính đã nắm vững:
- ✅ **Raster I/O**: Đọc và ghi raster datasets với metadata và profile management
- ✅ **Coordinate systems**: Transform matrices, CRS operations và reprojection workflows
- ✅ **Visualization**: Advanced plotting với matplotlib integration và custom colormaps
- ✅ **Band operations**: Mathematical calculations, spectral indices (NDVI, NDWI, SAVI)
- ✅ **Memory optimization**: Windowed reading và block-based processing cho big data
- ✅ **Advanced operations**: Clipping, masking, warping và geometric transformations
- ✅ **Format optimization**: Compression strategies, tiling và cloud-optimized workflows
- ✅ **Remote sensing applications**: DEM analysis, satellite imagery processing cho Việt Nam

### Kỹ năng bạn có thể áp dụng:
- Xử lý và phân tích satellite imagery và aerial photography một cách chuyên nghiệp
- Thực hiện remote sensing workflows cho environmental monitoring và agriculture
- Optimize raster processing performance cho production-scale geospatial applications
- Tích hợp Rasterio với NumPy và scientific Python stack cho advanced analysis
- Chuẩn bị foundation expertise cho computer vision và machine learning trên geospatial data
