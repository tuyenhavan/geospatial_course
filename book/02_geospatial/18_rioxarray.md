# Bài 18: Mở rộng dữ liệu raster với Rioxarray

RioXArray là thư viện kết hợp sức mạnh của XArray và Rasterio, mang geospatial superpowers đến cho multi-dimensional arrays.

## 18.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

- Tạo DataArrays với CRS và coordinate systems đầy đủ
- Xử lý file I/O cho nhiều raster formats (GeoTIFF, NetCDF, HDF5)
- Quản lý hệ tọa độ và thực hiện transformations
- Thực hiện spatial operations như clipping, masking, và buffering
- Resampling và interpolation cho grid data với spatial awareness
- Optimize performance cho dữ liệu lớn với chunking
- Tích hợp với các thư viện khác như geopandas, rasterio, matplotlib.


```python
import numpy as np
import pandas as pd
import xarray as xr
import rioxarray as rxr
import rasterio
from rasterio.crs import CRS
from rasterio.transform import from_bounds
import matplotlib.pyplot as plt
import time
import os
```

## 18.2. Tạo DataArrays đơn giản với thông tin địa lý

Rioxarray DataArray là một lớp mở rộng của xarray.DataArray, được thiết kế để làm việc với dữ liệu địa lý. Nó cung cấp các phương thức và thuộc tính bổ sung để xử lý thông tin về hệ tọa độ (CRS), phép biến đổi tọa độ, và các thao tác không gian khác. Điều này giúp bạn dễ dàng tích hợp dữ liệu địa lý vào quy trình phân tích của mình mà không cần phải chuyển đổi giữa các định dạng dữ liệu khác nhau.

### 18.2.1. Tạo DataArray đơn giản


```python
# Tạo dữ liệu nhiệt độ cho 4 vị trí
lon = [105.8, 106.0, 106.2, 106.4]
lat = [21.0, 21.2, 21.4, 21.6]
temp = [30, 32, 28, 31]

temperature = xr.DataArray(
    temp,
    coords={"point": range(4), "lon": ("point", lon), "lat": ("point", lat)},
    dims=["point"],
    name="temperature"
)
# Thêm crs và transform cho DataArray
temperature.rio.set_crs("EPSG:4326", inplace=True)
```

### 18.2.2. Tạo DataArray 2 chiều


```python
# Tạo DataArray 2 chiều 
lon = [105.8, 106.0, 106.2, 106.4]
lat = [21.0, 21.2, 21.4, 21.6]
temp_grid = np.array([[30, 31, 29, 28],
                      [32, 33, 30, 29],
                      [28, 29, 27, 26],
                      [31, 32, 29, 28]])
temperature = xr.DataArray(
    temp_grid,
    coords={"lon": lon, "lat": lat},
    dims=["lon", "lat"],
    name="temperature"
)
# Thêm crs và transform cho DataArray
temperature.rio.write_crs("EPSG:4326", inplace=True)
# create transform 
transform = from_bounds(min(lon), min(lat), max(lon), max(lat), len(lon), len(lat))
temperature.rio.write_transform(transform, inplace=True)
print(f"CRS: {temperature.rio.crs}")
```

### 18.2.3. Tạo DataArray nhiều chiều


```python
# Tạo DataArray 3d có time, lon, lat
time = pd.date_range("2024-01-01", periods=4, freq="D")
data = np.random.rand(4, 4, 4) * 30 + 10  # Dữ liệu nhiệt độ ngẫu nhiên từ 10 đến 40 độ
lon = [105.8, 106.0, 106.2, 106.4]
lat = [21.0, 21.2, 21.4, 21.6]
temperature = xr.DataArray(
    data,
    coords={"time": time, "lon": lon, "lat": lat},
    dims=["time", "lon", "lat"],
    name="temperature"
)
# Thêm crs và transform cho DataArray
temperature.rio.write_crs("EPSG:4326", inplace=True)
transform = from_bounds(min(lon), min(lat), max(lon), max(lat), len(lon), len(lat))
temperature.rio.write_transform(transform, inplace=True)
print(f"CRS: {temperature.rio.crs}")
```

# 18.3. Tạo Dataset với thông tin địa lý

Rioxarray Dataset là một cấu trúc dữ liệu cho phép bạn lưu trữ và quản lý dữ liệu địa lý một cách hiệu quả. Bằng cách sử dụng Rioxarray, bạn có thể dễ dàng thêm thông tin về hệ tọa độ (CRS) và phép biến đổi (transform) vào Dataset của mình, giúp đảm bảo rằng dữ liệu của bạn được định vị chính xác trên bản đồ và có thể được sử dụng trong các ứng dụng GIS và phân tích không gian một cách dễ dàng.

### 18.3.1. Tạo Dataset 1 chiều


```python
# Tạo dữ liệu nhiệt độ cho 4 vị trí
lon = [105.8, 106.0, 106.2, 106.4]
lat = [21.0, 21.2, 21.4, 21.6]
temp = [30, 32, 28, 31]

temperature = xr.Dataset(
    {"temperature": ("point", temp)},
    coords={"point": range(4), "lon": ("point", lon), "lat": ("point", lat)}
)
# Thêm crs và transform cho Dataset
temperature.rio.set_crs("EPSG:4326", inplace=True)
# create transform
transform = from_bounds(min(lon), min(lat), max(lon), max(lat), len(lon), len(lat))
temperature.rio.write_transform(transform, inplace=True)
```

### 18.3.2. Tạo Dataset 2 chiều



```python
# Tạo DataArray 2 chiều 
lon = [105.8, 106.0, 106.2, 106.4]
lat = [21.0, 21.2, 21.4, 21.6]
temp_grid = np.array([[30, 31, 29, 28],
                      [32, 33, 30, 29],
                      [28, 29, 27, 26],
                      [31, 32, 29, 28]])
temperature = xr.Dataset(
    {"temperature": (["lon", "lat"], temp_grid)},
    coords={"lon": lon, "lat": lat}
)
# Thêm crs và transform cho Dataset
temperature.rio.write_crs("EPSG:4326", inplace=True)
# create transform 
transform = from_bounds(min(lon), min(lat), max(lon), max(lat), len(lon), len(lat))
temperature.rio.write_transform(transform, inplace=True)
# Bạn có thể thêm các thông tin metadata khác nếu cần
temperature.attrs["description"] = "Nhiệt độ tại các điểm"
# Thêm đơn vị cho biến nhiệt độ
temperature["temperature"].attrs["units"] = "°C"
temperature
```

### 18.3.3. Tạo Dataset n chiều


```python
# Tạo DataArray 3d có time, lon, lat
time = pd.date_range("2024-01-01", periods=4, freq="D")
temp = np.random.rand(4, 4, 4) * 30 + 10  # Dữ liệu nhiệt độ ngẫu nhiên từ 10 đến 40 độ
rainfall = np.random.rand(4, 4, 4) * 100  # Dữ liệu lượng mưa ngẫu nhiên từ 0 đến 100 mm
lon = [105.8, 106.0, 106.2, 106.4]
lat = [21.0, 21.2, 21.4, 21.6]
climate = xr.Dataset(
    {"temperature": (["time", "lon", "lat"], temp),
     "rainfall": (["time", "lon", "lat"], rainfall)},
    coords={"time": time, "lon": lon, "lat": lat}
)
# Thêm crs và transform cho Dataset
climate.rio.set_crs("EPSG:4326", inplace=True)
# create transform
transform = from_bounds(min(lon), min(lat), max(lon), max(lat), len(lon), len(lat))
climate.rio.write_transform(transform, inplace=True)
```

## 18.4. Đọc và viết file

RioXArray hỗ trợ đa dạng định dạng raster như `GeoTIFF`, `NetCDF`, `COG`, và `Zarr` và cho phép chunk với dữ liệu lớn (big data).

### 18.3.1. Đọc dữ liệu 


```python
# Đọc dữ liệu raster từ URL và hiển thị thông tin cơ bản
raw_url = "https://raw.githubusercontent.com/tuyenhavan/geospatial_course/main/data/raster/era5_temp_2020_2024_vietnam.tif"
temp = rxr.open_rasterio(f"/vsicurl/{raw_url}")
temp.attrs = {} # Xóa metadata để tránh lỗi khi hiển thị
```

### 18.3.2. Lưu dữ liệu


```python
# lưu dữ liệu 
temp.rio.to_raster("output_temperature.tif") 
```

## 18.4. Hệ tọa độ (CRS)

### 18.4.1. Thiết lập CRS cho dữ liệu

Có nhiều phương pháp để xác định kích thước pixel và xây dựng phép biến đổi affine (affine transform) cho dữ liệu raster. Trong phần này, chúng ta sẽ lần lượt tìm hiểu hai phương pháp phổ biến.

- **Sử dụng `from_bounds`**

`from_bounds()` được sử dụng khi biết tọa độ biên của raster (west, south, east, north) và kích thước ảnh (width, height). Hàm sẽ tự động tính kích thước pixel và tạo affine transform phù hợp cho raster.


```python
# Tạo dữ liệu raster ngẫu nhiên 100x100 pixel
lon = np.linspace(105.8, 106.4, 100)
lat = np.linspace(21.0, 21.6, 100)
data = np.random.rand(100, 100) * 30 + 10  # Dữ liệu nhiệt độ ngẫu nhiên từ 10 đến 40 độ
temperature = xr.DataArray(
    data,
    coords={"x": lon, "y": lat},
    dims=["y", "x"],
    name="temperature"
)
# Thêm crs và transform cho DataArray
temperature.rio.write_crs("EPSG:4326", inplace=True)
transform = from_bounds(min(lon), min(lat), max(lon), max(lat), len(lon), len(lat))
temperature.rio.write_transform(transform, inplace=True)
```

- **Sử dụng from_origin**

`from_origin()` được sử dụng khi biết tọa độ góc trên bên trái của raster (west, north) và kích thước pixel (xsize, ysize). Hàm sẽ tạo affine transform dựa trên vị trí gốc và độ phân giải của ảnh.


```python
from rasterio.transform import from_origin
from pyproj import transformer
# giả sử biết tạo độ góc cho raster đo bằng GPS và dùng from_origin để tạo transform
lon_start, lat_start = 105.8, 21.01
# Ta cần chuyển đổi tọa độ từ EPSG:4326 (WGS 84) sang EPSG:32648 (UTM zone 48N) để tính toán transform chính xác với pixel size = 10m
x_start, y_start = transformer.Transformer.from_crs("EPSG:4326", "EPSG:32648").transform(lat_start, lon_start)
# Tạo dữ liệu raster giả lập với pixel size =10 m
data = np.random.rand(100, 150) * 30 + 10  # Dữ liệu nhiệt độ ngẫu nhiên từ 10 đến 40 độ
# Tạo x_range và y_range dựa trên transform và pixel size =10m 
x_range = np.linspace(x_start, x_start + 150 * 10, 150)
y_range = np.linspace(y_start, y_start + 100 * 10, 100)
# Tạo transform từ x_start, y_start với pixel size =10m
transform = from_origin(x_start, y_start, 10, 10)
temperature = xr.DataArray(
    data,
    coords={"x": x_range, "y": y_range},
    dims=["y", "x"],
    name="temperature"
)
# Thêm crs và transform cho DataArray
temperature.rio.write_crs("EPSG:32648", inplace=True)
temperature.rio.write_transform(transform, inplace=True)
```

### 18.4.2. Chuyển đổi hệ tọa độ sử dụng `reproject`

Reproject là quá trình chuyển đổi dữ liệu địa lý từ một hệ tọa độ này sang một hệ tọa độ khác. Điều này rất quan trọng trong GIS và phân tích không gian, vì dữ liệu có thể được thu thập hoặc lưu trữ ở các hệ tọa độ khác nhau. Việc reproject đảm bảo rằng dữ liệu của bạn được định vị thống nhất trên cùng một hệ tọa độ.


```python
# Chuyển temperature sang CRS khác (ví dụ EPSG:4326)
temperature_4326 = temperature.rio.reproject("EPSG:4326")
print(f"Original CRS: {temperature.rio.crs}")
print(f"Reprojected CRS: {temperature_4326.rio.crs}")
```

### 18.4.3. Chuyển đổi CRS và khớp lưới pixel sử dụng `reproject_match`

`Reproject_match` là một phương pháp trong rioxarray cho phép bạn tái dự án một Dataset hoặc DataArray sao cho nó khớp với hệ tọa độ và phép biến đổi của một đối tượng tham chiếu khác. Điều này rất hữu ích khi bạn có nhiều nguồn dữ liệu với các hệ tọa độ khác nhau và muốn đảm bảo rằng chúng được căn chỉnh chính xác trên bản đồ như kích thước pixels hay trasnform.


```python
# Dữ liệu raster nhiệt độ hàng năm của Việt Nam từ 2020-2024
raw_url = "https://raw.githubusercontent.com/tuyenhavan/geospatial_course/main/data/raster/era5_temp_2020_2024_vietnam.tif"
temp = rxr.open_rasterio(f"/vsicurl/{raw_url}")
temp.attrs = {} # Xóa metadata để tránh lỗi khi hiển thị
# Dữ liệu raster Landsat RGBN khu vực Vĩnh Yên, Vĩnh Phúc
landsat = rxr.open_rasterio(f"/vsicurl/https://raw.githubusercontent.com/tuyenhavan/geospatial_course/main/data/raster/landsat_rgbn.tif")
```


```python
# Ví dụ ta cắt dữ liệu raster nhiệt độ theo khu vực Vĩnh Yên, Vĩnh Phúc
bbox = landsat.rio.bounds()
temp_clip = temp.rio.clip_box(*bbox)[0] # Chỉ lấy band đầu tiên nếu có nhiều band
temp_reproject_match = temp_clip.rio.reproject_match(landsat) # Reproject và resample để khớp với raster Landsat.
# In shape trước và sau khi cắt và reprojection
print(f"Clipped shape: {temp_clip.shape} và Landsat shape: {landsat.shape}")
print(f"Reprojected and resampled shape: {temp_reproject_match.shape}")
```

## 18.5. Clip raster theo vùng

RioXArray cung cấp powerful tools cho spatial operations.

### 18.5.1. Cắt raster theo vùng với `rio.clip`

Clip là kĩ thuật cắt một raster theo một hình học vector (như đa giác) để chỉ giữ lại phần raster nằm trong hình học đó. Điều này rất hữu ích khi bạn muốn tập trung vào một khu vực cụ thể trong dữ liệu raster của mình, chẳng hạn như một tỉnh hoặc thành phố, và loại bỏ phần dữ liệu không liên quan đến khu vực đó. Clip raster giúp giảm kích thước dữ liệu và tăng hiệu quả khi phân tích không gian hoặc hiển thị bản đồ.


```python
# Đọc dữ liệu bản đồ Việt Nam
import geopandas as gpd
vector_path = 'https://geodata.ucdavis.edu/gadm/gadm4.1/json/gadm41_VNM_1.json'
vietnam_provinces = gpd.read_file(vector_path)
vinhphuc = vietnam_provinces[vietnam_provinces['VARNAME_1'] == 'VinhPhuc']
# Đọc dữ liệu nhiệt độ raster từ URL
raw_url = "https://raw.githubusercontent.com/tuyenhavan/geospatial_course/main/data/raster/era5_temp_2020_2024_vietnam.tif"
temp = rxr.open_rasterio(f"/vsicurl/{raw_url}")
temp.attrs = {} # Xóa metadata để tránh lỗi khi hiển thị
# Clip raster theo ranh giới Vĩnh Phúc
temp_vinhphuc = temp.rio.clip(vinhphuc.geometry, vinhphuc.crs)
print(f"Shape sau khi clip theo Vĩnh Phúc: {temp_vinhphuc.shape}")
```

### 18.5.2. Cắt raster theo vùng với `rio.clip_box`

Clip theo bounding box sẽ giữ lại tất cả các pixel nằm trong hộp giới hạn, bao gồm cả những pixel nằm ngoài ranh giới chính xác của AOI, trong khi clip theo ranh giới sẽ chỉ giữ lại các pixel thực sự nằm trong ranh giới của AOI. Do đó, shape sau khi clip theo bounding box thường sẽ lớn hơn hoặc bằng shape sau khi clip theo ranh giới. Kĩ thuật này có một lợi thế là tốc độ sử lý nhanh hơn.


```python
bbox = vinhphuc.total_bounds
temp_vinhphuc_bbox = temp.rio.clip_box(*bbox)
print(f"Shape sau khi clip theo bounding box của Vĩnh Phúc: {temp_vinhphuc_bbox.shape}")
```

## 18.6. Resampling và Interpolation

RioXArray cung cấp các phương pháp khác nhau cho việc resampling và nội suy.

### 18.6.1. Temporal resampling dữ liệu raster 

Tổng hợp dữ liệu theo giai đoạn thời gian có thể giúp chúng ta hiểu rõ hơn về xu hướng và biến động của các yếu tố môi trường như nhiệt độ, lượng mưa, và chỉ số thực vật (NDVI) trong một khu vực cụ thể. Bằng cách sử dụng rioxarray để xử lý dữ liệu raster và xarray để quản lý dữ liệu đa chiều, chúng ta có thể dễ dàng thực hiện các phép tính tổng hợp như trung bình, tổng, hoặc đếm số lần vượt ngưỡng trong các khoảng thời gian khác nhau, từ đó cung cấp thông tin quan trọng cho việc phân tích không gian và dự báo môi trường.


```python
from datetime import datetime

# Đọc dữ liệu NDVI từ 2015 đến 2024. Dữ liệu 16 ngày một lần, nên có nhiều band tương ứng với các thời điểm khác nhau.
modis = rxr.open_rasterio(f"/vsicurl/https://raw.githubusercontent.com/tuyenhavan/geospatial_course/main/data/raster/MODIS_NDVI_2015_2024.tif")
# Datetime của dữ liệu được chứa trong metadata của raster, bạn có thể trích xuất và đưa chúng thành python datetime objects.
time = [datetime.strptime(":".join(i.split("_")[1:]), '%d:%m:%Y') for i in modis.attrs['long_name']]
modis.attrs = {} # Xóa metadata để tránh lỗi khi hiển thị
modis['band'] = time
# đổi tên dimension band thành time để dễ hiểu hơn
modis = modis.rename({"band": "time"})
print(f"MODIS NDVI shape: {modis.shape}")
```

    MODIS NDVI shape: (230, 10, 11)
    


```python
# Resampling dữ liệu NDVI 16 ngày theo tháng bằng phương pháp mean
modis_monthly = modis.resample(time="1ME").mean()
print(f"Shape sau khi resample theo tháng: {modis_monthly.shape}")
```

    Shape sau khi resample theo tháng: (120, 10, 11)
    

### 18.6.2. Nội suy dữ liệu trống (missing values)

Nội suy dữ liệu trống (missing values) là kĩ thuật trong xử lý dữ liệu, đặc biệt là khi làm việc với dữ liệu raster có thể có các pixel bị thiếu do nhiều nguyên nhân như mây che phủ, lỗi cảm biến, hoặc khu vực không được quan sát. Việc nội suy giúp chúng ta ước lượng giá trị tại các điểm bị thiếu dựa trên các giá trị xung quanh, từ đó tạo ra một bức tranh hoàn chỉnh hơn về dữ liệu và cải thiện chất lượng phân tích không gian.


```python
from datetime import datetime

# Đọc dữ liệu NDVI từ 2015 đến 2024. Dữ liệu 16 ngày một lần, nên có nhiều band tương ứng với các thời điểm khác nhau.
modis = rxr.open_rasterio(f"/vsicurl/https://raw.githubusercontent.com/tuyenhavan/geospatial_course/main/data/raster/MODIS_NDVI_2015_2024.tif")
# Datetime của dữ liệu được chứa trong metadata của raster, bạn có thể trích xuất và đưa chúng thành python datetime objects.
time = [datetime.strptime(":".join(i.split("_")[1:]), '%d:%m:%Y') for i in modis.attrs['long_name']]
modis.attrs = {} # Xóa metadata để tránh lỗi khi hiển thị
modis['band'] = time
# đổi tên dimension band thành time để dễ hiểu hơn
modis = modis.rename({"band": "time"})
```


```python
# Filled missing values (nếu có) bằng phương pháp linear interpolation
modis = modis.interpolate_na(dim="time", method="linear")
print(f"Shape sau khi filled missing values: {modis.shape}")
```

    Shape sau khi filled missing values: (230, 10, 11)
    

## 18.7. Tính toán các chỉ số thực vật
RioXArray cung cấp các công cụ làm việc với multi-band rasters như ảnh viễn thám.

### 18.7.1. Tính toán chỉ số thực vật


```python
# Dữ liệu raster Landsat RGBN khu vực Vĩnh Yên, Vĩnh Phúc
landsat = rxr.open_rasterio(f"/vsicurl/https://raw.githubusercontent.com/tuyenhavan/geospatial_course/main/data/raster/landsat_rgbn.tif")
print(f"Landsat shape: {landsat.shape} và CRS: {landsat.rio.crs}")
# Tính NDVI. Trong ví dụ này, red band là band 3 và NIR band là band 4 (theo thứ tự trong file raster)
ndvi = (landsat[3] - landsat[2]) / (landsat[3] + landsat[2])
print(f"NDVI shape: {ndvi.shape}")
```

    Landsat shape: (4, 300, 338) và CRS: EPSG:4326
    NDVI shape: (300, 338)
    

### 18.7.2. Tính chỉ số nước


```python
# Tính chỉ số normalized difference water index (NDWI) sử dụng green band (band 2) và NIR band (band 4)
ndwi = (landsat[2] - landsat[3]) / (landsat[2] + landsat[3])
print(f"NDWI shape: {ndwi.shape}")
# Bạn có thể thêm ndvi, ndwi vào dataset 
dataset = xr.concat(
    [landsat, ndvi.expand_dims("band").assign_coords(band=["NDVI"]), ndwi.expand_dims("band").assign_coords(band=["NDWI"])],
    dim="band"
)
print(f"Dataset shape sau khi thêm NDVI và NDWI: {dataset.shape}")
```

    NDWI shape: (300, 338)
    Dataset shape sau khi thêm NDVI và NDWI: (6, 300, 338)
    

## Tóm tắt

Bạn đã hoàn thành Bài 18 và học được RioXArray - thư viện kết hợp sức mạnh của XArray và Rasterio cho phân tích dữ liệu địa không gian.

### Các khái niệm chính đã nắm vững:
- ✅ **DataArray địa không gian**: Mảng có nhận thức CRS với hệ tọa độ tham chiếu và metadata không gian
- ✅ **Đọc/ghi dữ liệu raster**: Xử lý GeoTIFF, NetCDF và HDF5 với bảo toàn thuộc tính địa không gian
- ✅ **Quản lý CRS**: Chuyển đổi tọa độ, chiếu lại và xử lý hệ tham chiếu không gian
- ✅ **Thao tác không gian**: Cắt (clipping), che phủ (masking), vùng đệm (buffering) và biến đổi hình học trên dữ liệu raster
- ✅ **Quy trình resampling**: Nội suy lưới, tổng hợp không gian và điều chỉnh độ phân giải
- ✅ **Khả năng tích hợp**: Quy trình làm việc liền mạch với geopandas, rasterio và matplotlib

### Kỹ năng bạn có thể áp dụng:
- Xử lý và phân tích ảnh vệ tinh và tập dữ liệu raster một cách chuyên nghiệp với nhận thức không gian
- Thực hiện các quy trình địa không gian cho giám sát môi trường và phân tích khí hậu tại Việt Nam
- Tối ưu hiệu suất xử lý raster cho các ứng dụng địa không gian quy mô lớn với hiệu quả bộ nhớ
- Tích hợp RioXArray với bộ công cụ Python khoa học cho phân tích không gian-thời gian nâng cao
- Xây dựng nền tảng chuyên sâu cho các ứng dụng viễn thám và GIS trong nghiên cứu và công nghiệp

