# Bài 18: Mở rộng dữ liệu raster với Rioxarray

RioXArray là thư viện kết hợp sức mạnh của XArray và Rasterio, mang geospatial superpowers đến cho multi-dimensional arrays.

## 18.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

- **Tạo geospatial DataArrays** với CRS và coordinate systems đầy đủ
- **Xử lý file I/O** cho nhiều raster formats (GeoTIFF, NetCDF, HDF5)
- **Quản lý coordinate reference systems** và thực hiện transformations
- **Thực hiện spatial operations** như clipping, masking, và buffering
- **Resampling và interpolation** cho grid data với spatial awareness
- **Visualize geospatial data** với built-in plotting capabilities  
- **Optimize performance** cho big geospatial datasets với chunking
- **Tích hợp với ecosystem** (geopandas, rasterio, matplotlib)


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
import warnings
warnings.filterwarnings('ignore')

# Optional: GeoPandas cho vector data
try:
    import geopandas as gpd
    from shapely.geometry import box, Point, Polygon
    GEOPANDAS_AVAILABLE = True
except ImportError:
    GEOPANDAS_AVAILABLE = False
    print("⚠️ GeoPandas not available - some examples will be skipped")
```

## 18.2. Tạo DataArrays đơn giản với thông tin địa lý

### 18.2.1. Tạo xarray dataarray


```python
# Tạo dữ liệu raster dem minh họa
dem = np.random.rand(100, 100) * 1000  # Dữ liệu độ cao ngẫu nhiên
dem.shape
```

### 17.2.2. Các hàm cơ bản


```python

```

## 17.3. File I/O và Format Support

RioXArray hỗ trợ đa dạng định dạng raster như `GeoTIFF`, `NetCDF`, `COG`, và `Zarr` và cho phép chunk với dữ liệu lớn (big data).

### 17.3.1. Đọc dữ liệu 

### 17.3.2. Lưu dữ liệu 


```python

```

## 17.4. Coordinate Reference Systems (CRS)

CRS management là core functionality của RioXArray cho geospatial analysis.

### 17.4.1. Thiết lập CRS cho dữ liệu


```python

```

### 17.4.2. Chuyển đổi hệ tọa độ sử dụng `reproject`


```python

```

### 17.4.3. Chuyển đổi CRS và khớp lưới pixel sử dụng `reproject_match`


```python

```

## 17.5. Cắt raster theo vùng

RioXArray cung cấp powerful tools cho spatial operations.

### 17.5.1. Cắt raster theo vùng với `rio.clip`


```python

```

### 17.5.2. Cắt raster theo vùng với `rio.clip_box`


```python

```

## 17.6. Resampling và Interpolation

RioXArray cung cấp các phương pháp khác nhau cho việc resampling và nội suy.

### 🎯 **Resampling Methods**
- **Nearest**: Preserve original values, fast
- **Bilinear**: Smooth interpolation cho continuous data
- **Cubic**: Higher-order polynomial, smooth results
- **Average**: Mean value cho downsampling

### 📐 **Resolution Operations**
- **Upsampling**: Increase resolution (interpolation)
- **Downsampling**: Decrease resolution (aggregation)
- **Match target**: Align với existing raster grid
- **Custom grids**: Specify exact output coordinates

### 🇻🇳 **Applications**
- **Multi-scale analysis**: Combine data ở different resolutions
- **Performance optimization**: Coarsen data cho faster processing
- **Data integration**: Match resolutions từ different sources
- **Visualization**: Appropriate resolution cho display

### 17.6.1. Resampling dữ liệu raster 


```python

```

### 17.6.2. Nội suy dữ liệu trống (missing values)


```python

```

## 17.7. Tính toán các chỉ số thực vật và kết hợp

RioXArray cung cấp các công cụ làm việc với multi-band rasters như ảnh viễn thám.

### 17.7.1. Tính toán chỉ số thực vật


```python

```

### 17.7.2. Tính số tháng hạn hán

## Tóm tắt

Bạn đã hoàn thành Bài 8 và học được RioXArray - thư viện kết hợp sức mạnh XArray và Rasterio cho geospatial analysis.

### Các khái niệm chính đã nắm vững:
- ✅ **Geospatial DataArrays**: CRS-aware arrays với coordinate reference systems và spatial metadata
- ✅ **Raster I/O operations**: Đọc/ghi GeoTIFF, NetCDF và HDF5 với geospatial attributes preservation
- ✅ **CRS management**: Coordinate transformations, reprojection và spatial reference handling
- ✅ **Spatial operations**: Clipping, masking, buffering và geometric transformations trên raster data
- ✅ **Resampling workflows**: Grid interpolation, spatial aggregation và resolution adjustments
- ✅ **Integration capabilities**: Seamless workflow với geopandas, rasterio và matplotlib ecosystem
- ✅ **Performance optimization**: Chunking strategies, lazy evaluation cho large geospatial datasets
- ✅ **Real-world applications**: DEM analysis, satellite imagery processing và environmental monitoring

### Kỹ năng bạn có thể áp dụng:
- Xử lý và phân tích satellite imagery và raster datasets một cách chuyên nghiệp với spatial awareness
- Thực hiện geospatial workflows cho environmental monitoring và climate analysis ở Việt Nam
- Optimize raster processing performance cho production-scale geospatial applications với memory efficiency
- Tích hợp RioXArray với scientific Python stack cho advanced spatial-temporal analysis
- Chuẩn bị foundation expertise cho remote sensing và GIS applications trong research và industry
