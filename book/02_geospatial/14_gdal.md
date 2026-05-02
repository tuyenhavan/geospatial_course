# Bài 14: Thảo tác dữ liệu với GDAL

Trong bài học này, chúng ta sẽ khám phá thư viện GDAL (Geospatial Data Abstraction Library) - "trái tim" của hầu hết các ứng dụng GIS trên thế giới.

GDAL là thư viện mã nguồn mở mạnh mẽ nhất trong hệ sinh thái GIS, cung cấp nền tảng để đọc, ghi, chuyển đổi và xử lý dữ liệu địa không gian. Với hỗ trợ hơn 200 định dạng dữ liệu khác nhau, GDAL là xương sống cho các thư viện như GeoPandas, Rasterio, Fiona và hầu hết các phần mềm GIS chuyên nghiệp.

## 14.1. Mục tiêu học tập
Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Hiểu rõ kiến trúc và vai trò của GDAL trong hệ sinh thái GIS
- Làm việc trực tiếp với dữ liệu raster (GDAL) và vector (OGR)
- Xử lý hệ tọa độ tham chiếu với OSR (OGR Spatial Reference)
- Thực hiện chuyển đổi định dạng dữ liệu giữa hàng trăm format
- Tối ưu hóa hiệu suất khi xử lý dữ liệu địa không gian lớn


```python
# Import các thư viện cần thiết
from osgeo import gdal, ogr, osr  # Thư viện GDAL chính
import numpy as np                # Xử lý mảng số
import matplotlib.pyplot as plt   # Vẽ biểu đồ
import warnings                  # Quản lý cảnh báo
warnings.filterwarnings('ignore', category=FutureWarning)
```

## 14.2. Xử lý dữ liệu Raster với GDAL

Làm việc với dữ liệu raster (ảnh vệ tinh, bản đồ số) sử dụng giao diện GDAL gốc.

## 2. Xử lý dữ liệu Vector với OGR

Làm việc với dữ liệu vector (điểm, đường, vùng) sử dụng thành phần OGR của GDAL.

## 3. Hệ tọa độ tham chiếu với OSR 

Làm việc với hệ tọa độ tham chiếu không gian sử dụng thành phần OSR của GDAL.

## Tóm tắt

Bạn đã hoàn thành Bài 4 và học được GDAL - thư viện nền tảng mạnh mẽ nhất trong hệ sinh thái GIS toàn cầu.

### Các khái niệm chính đã nắm vững:
- ✅ **GDAL Architecture**: Raster (GDAL), Vector (OGR), và Spatial Reference (OSR) systems
- ✅ **Raster operations**: Tạo, đọc, ghi và manipulate raster datasets với drivers
- ✅ **Vector operations**: Shapefile, GeoJSON processing với OGR layers và features
- ✅ **Coordinate systems**: CRS transformations và reprojection với OSR
- ✅ **Format conversions**: Chuyển đổi giữa 200+ định dạng raster và vector
- ✅ **Spatial operations**: Clipping, merging, buffering và geometric analysis
- ✅ **Performance optimization**: Memory management và chunked processing
- ✅ **Industry integration**: Nền tảng cho GeoPandas, Rasterio, Fiona và major GIS software

### Kỹ năng bạn có thể áp dụng:
- Xử lý và chuyển đổi dữ liệu geospatial ở level thấp với performance cao
- Làm việc trực tiếp với GDAL C++ API từ Python cho maximum control
- Debug và troubleshoot issues trong các thư viện GIS Python khác
- Thực hiện batch processing và automation cho large-scale geospatial workflows
- Hiểu sâu về geospatial data structures để tối ưu hóa analysis pipelines
