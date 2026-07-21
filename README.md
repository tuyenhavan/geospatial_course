# Dữ liệu Địa không gian với Python

Chuỗi bài hướng dẫn toàn diện về lập trình Python ứng dụng trong phân tích dữ liệu địa không gian, viễn thám và các nền tảng địa không gian đám mây.

## Giới thiệu

Khóa học này được thiết kế dành cho những ai muốn xây dựng kỹ năng lập trình cho phân tích dữ liệu địa không gian với Python - từ người mới bắt đầu cho đến người dùng trung cấp muốn mở rộng kiến thức về xử lý dữ liệu không gian, trực quan hóa và điện toán đám mây. Các bài hướng dẫn được trình bày dưới dạng Jupyter Notebook, cho phép người học thực hành trực tiếp song song với các giải thích lý thuyết.

## Cấu trúc khóa học

### Phần 1 - Nền tảng Python cơ bản

Xây dựng nền tảng vững chắc về lập trình Python trước khi đi vào các công cụ địa không gian.

| Bài số | Nội dung                                         |
| ------ | ------------------------------------------------ |
| 01     | Cài đặt và Thiết lập môi trường Python           |
| 02     | Các kiểu dữ liệu cơ bản Python                   |
| 03     | Cấu trúc dữ liệu                                 |
| 04     | Cấu trúc điều khiển và vòng lặp                  |
| 05     | Hàm & Lớp (function & class)                     |
| 06     | Làm việc với tệp tin                             |
| 07     | NumPy cơ bản                                     |
| 08     | Pandas cơ bản                                    |
| 09     | Trực quan hóa dữ liệu cơ bản                     |
| 10     | Phát triển và Đóng gói Python (Python packaging) |

---

### Phần 2 - Python Địa không gian

Các thư viện địa không gian cốt lõi để xử lý dữ liệu địa không gian.

| Bài số | Nội dung                                                     |
| ------ | ------------------------------------------------------------ |
| 11     | Shapely - Đối tượng hình học và các phép toán không gian     |
| 12     | Pyproj - Hệ tọa độ và phép chiếu bản đồ                      |
| 13     | Fiona - Đọc và ghi dữ liệu vector                            |
| 14     | GDAL - Xử lý dữ liệu raster và vector                        |
| 15     | GeoPandas - Xử lý dữ liệu vector với GeoPandas               |
| 16     | Rasterio - Đọc, xử lý, và ghi dữ liệu raster                 |
| 17     | Xarray - Mảng nhiều chiều có nhãn                            |
| 18     | Rioxarray - Tích hợp Rasterio với Xarray                     |
| 19     | Zarr & Dask - Lưu trữ mảng quy mô lớn và tính toán song song |
| 20     | Matplotlib - Trực quan hóa dữ liệu địa không gian            |

---

### Phần 3 - Nền tảng Địa không gian Đám mây

Truy cập và xử lý dữ liệu địa không gian quy mô lớn trên các nền tảng đám mây.

| Bài số | Nội dung                                                    |
| ------ | ----------------------------------------------------------- |
| 21     | Truy cập và khám phá dữ liệu (Google Earth Engine)          |
| 22     | Các thao tác cơ bản (Google Earth Engine)                   |
| 23     | Loại bỏ mây (Google Earth Engine)                           |
| 24     | Tính toán các chỉ số thực vật (Google Earth Engine)         |
| 25     | Tổng hợp ảnh theo giai đoạn (Google Earth Engine)           |
| 26     | Trích xuất giá trị raster theo vị trí (Google Earth Engine) |
| 27     | Xuất dữ liệu (Google Earth Engine)                          |
| 28     | Truy cập dữ liệu (Microsoft Planetary Computer)             |
| 29     | Xử lý dữ liệu (Microsoft Planetary Computer)                |
| 30     | Xử lý dữ liệu lớn (Microsoft Planetary Computer)            |
| 31     | Truy cập dữ liệu Copernicus (CDSE)                 |

---

### Phần 4 - Ứng dụng

Ứng dụng các kỹ thuật phân tích và học máy vào dữ liệu không gian và viễn thám.

| Bài số | Nội dung                                                                                                                     |
| ------ | ---------------------------------------------------------------------------------------------------------------------------- |
| 32     | Phân loại lớp phủ mặt đất (Land Cover Classification) - Random Forest trên ảnh vệ tinh Sentinel-2                                 |
| 33     | Phát hiện hạn hán (Drought Detection) - Phân tích chỉ số hạn hán với dữ liệu MODIS và ERA5                                   |
| 34     | Giám sát biến động nước bề mặt (Surface water monitoring) - Kết hợp SAR và ảnh quang học                                                             |
| 35     | Giám sát phân đoạn trang trại điện mặt trời (solarfarm segmentation) - Sử dụng ảnh Sentinel-2 và mô hình học sâu (Deep learning)                         |


## Dữ liệu

Các tập dữ liệu mẫu sử dụng trong khóa học có thể tải về tại [đây](https://drive.google.com/drive/folders/119C2B1pBKwvDx5OASvQRvGR1lOljgJNd?usp=sharing) hoặc trong mỗi phần của bài học. 

> **Chú ý**: Dữ liệu trong khóa học này chỉ phục vụ mục đích minh họa. Để đảm bảo tính chính xác và cập nhật, vui lòng đối chiếu với các nguồn thông tin chính thống.

## Cài đặt

Để có trải nghiệm học tập tốt nhất và hạn chế các lỗi phát sinh trong quá trình thực hành, bạn nên chuẩn bị môi trường làm việc trước khi bắt đầu khóa học.

Nếu chưa cài đặt Python và Miniconda, hãy đọc Bài 1, trong đó hướng dẫn chi tiết cách cài đặt và thiết lập đầy đủ các công cụ cần thiết.
Nếu bạn đã cài đặt Python và Miniconda, bạn chỉ cần chạy câu lệnh sau sẽ tạo ra môi trường ảo `geobook` và cài đặt các thư viện cần thiết trong `environment.yml` file.

```bash 
conda env create -f https://raw.githubusercontent.com/tuyenhavan/geospatial_course/main/environment.yml
```

> **Chú ý**: Mỗi bài học đều hỗ trợ chạy notebook trên Google Colab, giúp bạn thực hành ngay mà không cần cài đặt Python.

## Giấy phép

Dự án này là mã nguồn mở, được phép sử dụng miễn phí cho mục đích học tập và nghiên cứu cá nhân. Mọi hình thức sử dụng cho giảng dạy, đào tạo hoặc mục đích thương mại cần có sự đồng ý của tác giả.

## Miễn trừ trách nhiệm

Nội dung, mã nguồn và dữ liệu trong dự án này chỉ nhằm mục đích học tập, nghiên cứu và minh họa. Tác giả không bảo đảm tính đầy đủ, chính xác hoặc cập nhật của thông tin và không chịu trách nhiệm đối với bất kỳ thiệt hại nào phát sinh từ việc sử dụng nội dung của dự án. Người sử dụng có trách nhiệm tự kiểm chứng thông tin trước khi áp dụng vào thực tế.

## Kết nối

Trong quá trình học, nếu bạn có bất kỳ câu hỏi nào hoặc góp ý, hãy tham gia và trao đổi trên nhóm Facebook chính thức của khóa học: [Open Learning Geolab](https://www.facebook.com/groups/4141192679432266). Đây là nơi bạn có thể đặt câu hỏi, thảo luận và đóng góp giúp cải thiện khóa học.


