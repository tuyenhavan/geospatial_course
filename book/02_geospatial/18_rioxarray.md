# Bài 18: Mở rộng dữ liệu raster với Rioxarray

RioXArray là thư viện kết hợp sức mạnh của XArray và Rasterio, mang geospatial superpowers đến cho multi-dimensional arrays.

> **Lưu ý**: Bạn có thể chạy trực tiếp notebook này bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/10Slrsif0Y3zfI87ulUHP_gd8jV4QYVnR) mà không cần cài đặt Python.

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
from rasterio.crs import CRS
from rasterio.transform import from_bounds
import matplotlib.pyplot as plt
import os
import geopandas as gpd
```

## 18.2. Tạo DataArrays đơn giản với thông tin địa lý

Rioxarray DataArray là một lớp mở rộng của xarray.DataArray, được thiết kế để làm việc với dữ liệu địa lý. Nó cung cấp các phương thức và thuộc tính bổ sung để xử lý thông tin về hệ tọa độ (CRS), phép biến đổi tọa độ, và các thao tác không gian khác. Điều này giúp bạn dễ dàng tích hợp dữ liệu địa lý vào quy trình phân tích của mình mà không cần phải chuyển đổi giữa các định dạng dữ liệu khác nhau.

### 18.2.1. Tạo DataArray đơn giản

Trong ví dụ này, chúng ta tạo một DataArray với tên là `temperature` chứa dữ liệu nhiệt độ cho 4 điểm khác nhau, mỗi điểm có tọa độ kinh độ (lon) và vĩ độ (lat). Chúng ta đã sử dụng `rioxarray` để gán hệ tọa độ `EPSG:4326` cho DataArray này, giúp chúng ta có thể làm việc với dữ liệu địa lý một cách dễ dàng hơn trong các phân tích địa không gian.


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

Tương tự vậy như ví dụ trên, chúng ta cũng có thể tạo một DataArray 2 chiều với các giá trị nhiệt độ được tổ chức theo lưới kinh độ và vĩ độ. Sau khi tạo DataArray này, chúng ta cũng gán hệ tọa độ `EPSG:4326` và `transform` để xác định vị trí của các pixel trong không gian địa lý. Điều này giúp chúng ta có thể hiển thị dữ liệu trên bản đồ.


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

'Trong thực tế, dữ liệu được thu thập theo thời gian. Vì vậy, chúng ta có thể tạo một DataArray 3 chiều với các giá trị nhiệt độ được tổ chức theo thời gian (time), kinh độ (lon) và vĩ độ (lat). Sau khi tạo DataArray này, chúng ta cũng gán hệ tọa độ `EPSG:4326` và `transform` để xác định vị trí của các pixel trong không gian địa lý.


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

Giả sử chúng ta có một tập dữ liệu nhiệt độ được tổ chức theo các điểm với tọa độ kinh độ (lon) và vĩ độ (lat). Chúng ta muốn tạo một Dataset chứa biến "temperature" với các giá trị nhiệt độ tương ứng cho từng điểm. Sau đó, chúng ta sử dụng rioxarray để gán hệ tọa độ `EPSG:4326` và `transform` để xác định vị trí của các pixel trong không gian địa lý.


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

Thay vì tạo một Dataset với dữ liệu điểm, chúng ta có thể tạo một Dataset với dữ liệu được tổ chức theo lưới kinh độ và vĩ độ. Sau khi tạo Dataset, chúng ta cũng gán hệ tọa độ `EPSG:4326` và `transform` để xác định vị trí của các pixel trong không gian địa lý. Ngoài ra, chúng ta cũng có thể thêm các thông tin metadata khác như mô tả và đơn vị cho biến nhiệt độ để làm rõ ý nghĩa của dữ liệu.


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
```

### 18.3.3. Tạo Dataset n chiều

Trong ví dụ này, chúng ta tạo một Dataset với hai biến là `temperature` và `rainfall`, mỗi biến có dữ liệu được tổ chức theo thời gian (time), kinh độ (lon) và vĩ độ (lat). Sau khi tạo Dataset này, chúng ta cũng gán hệ tọa độ `EPSG:4326` và `transform` để xác định vị trí của các pixel trong không gian địa lý. Ngoài ra, chúng ta cũng có thể thêm các thông tin metadata khác như mô tả và đơn vị cho các biến để làm rõ ý nghĩa của dữ liệu.'


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

### 18.4.1. Đọc dữ liệu từ `url`

RioXArray cho phép đọc dữ liệu raster trực tiếp từ URL mà không cần tải về máy tính, giúp tiết kiệm không gian lưu trữ và tăng tốc độ xử lý. Điều này đặc biệt hữu ích khi làm việc với dữ liệu lớn hoặc khi bạn muốn thử nghiệm nhanh mà không cần tải toàn bộ dữ liệu về máy.


```python
# Đọc dữ liệu raster từ URL và hiển thị thông tin cơ bản
raw_url = "https://raw.githubusercontent.com/tuyenhavan/geodata/main/raster/vinhphuc_temperature_2020.tif"
temp = rxr.open_rasterio(raw_url, masked=True)
```

### 18.4.2. Lưu dữ liệu trên máy

Sau khi xử lý dữ liệu raster, bạn có thể lưu kết quả về máy tính dưới nhiều định dạng khác nhau như GeoTIFF, NetCDF, COG (Cloud Optimized GeoTIFF), hoặc Zarr. Phương thức `rio.to_raster()` giúp lưu DataArray thành file GeoTIFF với đầy đủ thông tin CRS và transform. Bạn cũng có thể tùy chỉnh các thông số nén (compression), kiểu dữ liệu (dtype), và NoData value khi lưu file để tối ưu kích thước và chất lượng dữ liệu.


```python
# lưu dữ liệu 
temp.rio.to_raster(r"J:\My Drive\geocourse_data\outputs\temperature_2020.tif") 
```

## 18.4. Hệ tọa độ (CRS)

### 18.4.1. Thiết lập CRS cho dữ liệu

Có nhiều phương pháp để xác định kích thước pixel và xây dựng phép biến đổi affine (affine transform) cho dữ liệu raster. Trong phần này, chúng ta sẽ lần lượt tìm hiểu hai phương pháp phổ biến.

- **Sử dụng `from_bounds`**

`from_bounds()` được sử dụng khi biết tọa độ biên của raster (west, south, east, north) và kích thước ảnh (width, height). Hàm sẽ tự động tính kích thước pixel và tạo affine transform phù hợp cho raster.


```python
# Tạo dữ liệu raster ngẫu nhiên 100x100 pixel cho khu vực theo tọa độ lon, lat
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

`Reproject_match` là một phương pháp trong rioxarray cho phép bạn tái dự án một Dataset hoặc DataArray sao cho nó khớp với hệ tọa độ và phép biến đổi của một đối tượng tham chiếu khác. Điều này rất hữu ích khi bạn có nhiều nguồn dữ liệu với các hệ tọa độ khác nhau và muốn đảm bảo rằng chúng được căn chỉnh chính xác trên bản đồ như kích thước pixels hay tranform. 

Trong ví dụ này, ta sẽ sử dụng `reproject_match` để khớp dữ liệu 10m với dữ liệu 30m như bên dưới.


```python
# Dữ liệu Sentinel-2 10m 
raster_10m = rxr.open_rasterio('https://raw.githubusercontent.com/tuyenhavan/geodata/main/raster/sen2data.tif')
# Dữ liệu 30m 
raster_30m = rxr.open_rasterio('https://raw.githubusercontent.com/tuyenhavan/geodata/main/raster/sen2data_30m.tif')

print(f"Raster 10m shape: {raster_10m.shape}, raster 30m: {raster_30m.shape}")
```


```python
# Khớp dữ liệu 10m với dữ liệu 30m bằng cách reprojection và resampling. Đảm bảo crs của hai raster khớp nhau trước khi reprojection. Nếu không, bạn cần reproject một trong hai raster sang crs của raster còn lại trước khi reprojection.
reproject_data = raster_10m.rio.reproject_match(raster_30m) # Reproject và resample để khớp với raster 30m.
# Shape của raster sau khi reprojection và resampling
print(f"Reprojected and resampled shape: {reproject_data.shape}")
```

## 18.5. Clip raster theo vùng

RioXArray cung cấp powerful tools cho spatial operations.

### 18.5.1. Cắt raster theo vùng với `rio.clip`

Clip là kĩ thuật cắt một raster theo một hình học vector (như đa giác) để chỉ giữ lại phần raster nằm trong hình học đó. Điều này rất hữu ích khi bạn muốn tập trung vào một khu vực cụ thể trong dữ liệu raster của mình, chẳng hạn như một tỉnh hoặc thành phố, và loại bỏ phần dữ liệu không liên quan đến khu vực đó. Clip raster giúp giảm kích thước dữ liệu và tăng hiệu quả khi phân tích không gian hoặc hiển thị bản đồ.


```python
# Đọc dữ liệu cấp huyện Vĩnh Phúc từ file vector
districts = gpd.read_file('https://raw.githubusercontent.com/tuyenhavan/geodata/refs/heads/main/vector/vinhphuc_districts.geojson')
# Đọc dữ liệu MODIS EVI Vĩnh Phúc từ file raster
evi = rxr.open_rasterio('https://raw.githubusercontent.com/tuyenhavan/geodata/main/raster/modis_monthly_evi_vinhphuc_2020_2025.tif')
# Chọn một huyện nào đó, ví dụ huyện Vĩnh Yên
vinh_yen = districts[districts['districts'] == 'Vinh Yen']
# Cắt dữ liệu raster EVI theo khu vực  Vĩnh Yên
evi_clip = evi.rio.clip(vinh_yen.geometry, vinh_yen.crs, drop=True)
print(f"Original EVI shape: {evi.shape}, Clipped EVI shape: {evi_clip.shape}")
```

    Original EVI shape: (72, 48, 53), Clipped EVI shape: (72, 8, 10)
    

### 18.5.2. Cắt raster theo vùng với `rio.clip_box`

Clip theo bounding box sẽ giữ lại tất cả các pixel nằm trong hộp giới hạn, bao gồm cả những pixel nằm ngoài ranh giới chính xác của AOI, trong khi clip theo ranh giới sẽ chỉ giữ lại các pixel thực sự nằm trong ranh giới của AOI. Do đó, shape sau khi clip theo bounding box thường sẽ lớn hơn hoặc bằng shape sau khi clip theo ranh giới. Kĩ thuật này có một lợi thế là tốc độ sử lý nhanh hơn.


```python
bbox = vinh_yen.total_bounds
evi_bbox = evi.rio.clip_box(*bbox)
print(f"Shape sau khi clip theo bounding box của Vĩnh Phúc: {evi_bbox.shape}")
```

    Shape sau khi clip theo bounding box của Vĩnh Phúc: (72, 10, 11)
    

## 18.6. Resampling và Interpolation

RioXArray cung cấp các phương pháp khác nhau cho việc resampling và nội suy.

### 18.6.1. Temporal resampling dữ liệu raster 

Tổng hợp dữ liệu theo giai đoạn thời gian có thể giúp chúng ta hiểu rõ hơn về xu hướng và biến động của các yếu tố môi trường như nhiệt độ, lượng mưa, và chỉ số thực vật (NDVI) trong một khu vực cụ thể. Bằng cách sử dụng rioxarray để xử lý dữ liệu raster và xarray để quản lý dữ liệu đa chiều, chúng ta có thể dễ dàng thực hiện các phép tính tổng hợp như trung bình, tổng, hoặc đếm số lần vượt ngưỡng trong các khoảng thời gian khác nhau, từ đó cung cấp thông tin quan trọng cho việc phân tích không gian và dự báo môi trường.

- **Gán chiều thời gian cho dữ liệu**


```python
# Đọc dữ liệu EVI từ 2020 đến 2025 tỉnh Vĩnh Phúc. Dữ liệu theo tháng.
evi = rxr.open_rasterio('https://raw.githubusercontent.com/tuyenhavan/geodata/main/raster/modis_monthly_evi_vinhphuc_2020_2025.tif')
time = pd.date_range("2020-01-01", periods=evi.shape[0], freq="ME")
# Gán thời gian vào dimension band của raster EVI
evi['band'] = time
# Đổi tên dimension band thành time để dễ hiểu hơn
evi = evi.rename({"band": "time"})
```

- **Tổng hợp dữ liệu EVI theo năm**


```python
# Resampling dữ liệu EVI hàng tháng theo năm bằng phương pháp mean
evi_monthly = evi.resample(time="1YE").mean()
print(f"Shape sau khi resample theo tháng: {evi_monthly.shape}")
```

    Shape sau khi resample theo tháng: (6, 48, 53)
    

### 18.6.2. Nội suy dữ liệu trống (missing values)

Nội suy dữ liệu trống (missing values) là kĩ thuật trong xử lý dữ liệu, đặc biệt là khi làm việc với dữ liệu raster có thể có các pixel bị thiếu do nhiều nguyên nhân như mây che phủ, lỗi cảm biến, hoặc khu vực không được quan sát. Việc nội suy giúp chúng ta ước lượng giá trị tại các điểm bị thiếu dựa trên các giá trị xung quanh, từ đó tạo ra một bức tranh hoàn chỉnh hơn về dữ liệu và cải thiện chất lượng phân tích không gian.


```python
# Filled missing values (nếu có) bằng phương pháp linear interpolation
evi_filled = evi.interpolate_na(dim="time", method="linear")
print(f"Shape sau khi filled missing values: {evi.shape}")
```

    Shape sau khi filled missing values: (72, 48, 53)
    

## 18.7. Tính toán các chỉ số thực vật
RioXArray cung cấp các công cụ làm việc với multi-band rasters như ảnh viễn thám.

### 18.7.1. Tính toán chỉ số thực vật

Chỉ số thực vật NDVI (Normalized Difference Vegetation Index) là một trong những chỉ số quan trọng nhất trong viễn thám để đánh giá sức khỏe và mật độ thực vật. NDVI được tính bằng công thức (NIR - Red) / (NIR + Red), với giá trị dao động từ -1 đến 1. Giá trị NDVI cao (gần 1) cho thấy thực vật xanh tốt và khỏe mạnh, trong khi giá trị thấp (gần 0 hoặc âm) thường là đất trống, nước, hoặc thực vật chết. RioXArray giúp tính toán NDVI một cách dễ dàng từ dữ liệu ảnh vệ tinh đa băng tần như Sentinel-2 hoặc Landsat.


```python
# Dữ liệu raster Landsat RGBN khu vực Vĩnh Yên, Vĩnh Phúc
sen2data = rxr.open_rasterio('https://raw.githubusercontent.com/tuyenhavan/geodata/main/raster/sen2data.tif')
# Tính NDVI. Trong ví dụ này, red band là band 3 và NIR band là band 4 (theo thứ tự trong file raster)
ndvi = (sen2data[3] - sen2data[2]) / (sen2data[3] + sen2data[2])
print(f"NDVI shape: {ndvi.shape}")
```

    NDVI shape: (500, 500)
    

### 18.7.2. Tính chỉ số nước

Chỉ số nước NDWI (Normalized Difference Water Index) được sử dụng để phát hiện và theo dõi các vùng nước như sông, hồ, ao, và vùng ngập lụt. NDWI được tính bằng công thức (Green - NIR) / (Green + NIR), trong đó giá trị dương cao thường chỉ ra sự hiện diện của nước, trong khi giá trị âm thường là đất hoặc thực vật. Chỉ số này rất hữu ích trong giám sát tài nguyên nước, phát hiện lũ lụt, và quản lý môi trường. Bằng cách kết hợp NDVI và NDWI, chúng ta có thể phân tích toàn diện hơn về môi trường và các đối tượng trên bề mặt đất.


```python
# Tính chỉ số normalized difference water index (NDWI) sử dụng green band (band 2) và NIR band (band 4)
ndwi = (sen2data[2] - sen2data[3]) / (sen2data[2] + sen2data[3])
print(f"NDWI shape: {ndwi.shape}")
# Bạn có thể thêm ndvi, ndwi vào dataset 
dataset = xr.concat(
    [sen2data, ndvi.expand_dims("band").assign_coords(band=["NDVI"]), ndwi.expand_dims("band").assign_coords(band=["NDWI"])],
    dim="band"
)
print(f"Dataset shape sau khi thêm NDVI và NDWI: {dataset.shape}")
```

    NDWI shape: (500, 500)
    Dataset shape sau khi thêm NDVI và NDWI: (6, 500, 500)
    

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

