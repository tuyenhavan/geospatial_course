# Python cho Phân tích Dữ liệu Địa không gian

Chuỗi bài hướng dẫn toàn diện về lập trình Python ứng dụng trong phân tích dữ liệu địa không gian, viễn thám và các nền tảng địa không gian đám mây.

## Giới thiệu

Khóa học này được thiết kế dành cho những ai muốn xây dựng kỹ năng thực hành trong lập trình địa không gian với Python — từ người mới bắt đầu cho đến người dùng trung cấp muốn mở rộng kiến thức về xử lý dữ liệu không gian, trực quan hóa và điện toán đám mây. Các bài hướng dẫn được trình bày dưới dạng Jupyter Notebook, cho phép thực hành trực tiếp song song với các giải thích lý thuyết.

## Cấu trúc khóa học

### Phần 1 - Nền tảng Python cơ bản

Xây dựng nền tảng vững chắc về lập trình Python trước khi đi vào các công cụ địa không gian.

| Notebook | Nội dung |
|----------|----------|
| 01 | Cài đặt và Thiết lập môi trường |
| 02 | Các khái niệm cơ bản Python |
| 03 | Cấu trúc dữ liệu |
| 04 | Cấu trúc điều khiển và vòng lặp |
| 05 | Hàm & Lớp (function & class) |
| 06 | Làm việc với tệp tin |
| 07 | NumPy cơ bản |
| 08 | Pandas cơ bản |
| 09 | Trực quan hóa dữ liệu cơ bản |
| 10 | Phát triển và Đóng gói Python (Python Packaging) |

---

### Phần 2 - Python Địa không gian

Các thư viện địa không gian cốt lõi để xử lý dữ liệu vector và raster trên máy cục bộ.

| Notebook | Nội dung |
|----------|----------|
| 11 | Shapely - Đối tượng hình học và các phép toán không gian |
| 12 | Pyproj - Hệ tọa độ và phép chiếu bản đồ |
| 13 | Fiona - Đọc và ghi dữ liệu vector |
| 14 | GDAL - Xử lý dữ liệu raster và vector |
| 15 | GeoPandas - Spatial DataFrame và phân tích vector |
| 16 | Rasterio - Đọc/ghi và xử lý dữ liệu raster |
| 17 | Xarray - Mảng nhiều chiều có nhãn |
| 18 | Rioxarray - Tích hợp Rasterio với Xarray |
| 19 | Zarr & Dask - Lưu trữ mảng quy mô lớn và tính toán song song |
| 20 | Matplotlib - Trực quan hóa dữ liệu địa không gian |

---

### Phần 3 — Nền tảng Địa không gian Đám mây

Truy cập và xử lý dữ liệu địa không gian quy mô lớn trên các nền tảng đám mây.

| Notebook | Nội dung |
|----------|----------|
| 20 | Truy cập và khám phá dữ liệu (Google Earth Engine) |
| 21 | Các thao tác cơ bản (Google Earth Engine) |
| 22 | Loại bỏ mây (Google Earth Engine) |
| 23 | Tính toán các chỉ số thực vật (Google Earth Engine) |
| 24 | Tổng hợp ảnh theo giai đoạn (Google Earth Engine) |
| 25 | Trích xuất giá trị raster theo vị trí (Google Earth Engine) |
| 26 | Xuất dữ liệu (Google Earth Engine) |
| 27 | Truy cập dữ liệu (Microsoft Planetary Computer) |
| 28 | Xử lý dữ liệu (Microsoft Planetary Computer) |
| 29 | Xử lý dữ liệu lớn (Microsoft Planetary Computer) |
| 30 | Truy cập dữ liệu Copernicus (CDSE)|

---

### Phần 4 — Ứng dụng

Ứng dụng các kỹ thuật phân tích và học máy vào dữ liệu không gian và viễn thám *(sắp ra mắt)*.

| Bài số | Nội dung |
|----------|----------|
| 31 | Phân loại lớp phủ mặt đất (Land Cover Classification) — Random Forest & SVM trên ảnh vệ tinh |
| 32 | Phát hiện hạn hán (Drought Detection) — Phân tích chỉ số hạn hán với dữ liệu MODIS và ERA5 |
| 33 | Giám sát ngập lụt (Flood Mapping) — Kết hợp SAR và ảnh quang học |
| 34 | Ước tính chiều cao tán cây (Vegetation Canopy Height) — Xử lý ảnh Sentinel-2 kết hợp mô hình hồi quy học sâu (Deep learning)|
| 35 | Phân vùng mặt nước (Water Body Segmentation) — Deep Learning với dữ liệu viễn thám đa phổ |


## Dữ liệu

Các tập dữ liệu mẫu sử dụng trong khóa học (bao gồm ranh giới hành chính Việt Nam và dữ liệu raster) được lưu trong thư mục `data/`.

## Giấy phép

Dự án này là mã nguồn mở và được sử dụng tự do cho mục đích học tập.

## Ghi chú
Một phần nội dung của tài liệu này được hỗ trợ tạo bởi các công cụ AI. Tất cả nội dung đã được tác giả xem xét, chỉnh sửa, bổ sung và kiểm chứng nhằm đảm bảo tính chính xác và phù hợp.
