#  Bài 20: Trực Quan hóa Dữ liệu Địa không gian

Trực quan hóa dữ liệu địa không gian là bước quan trọng giúp hiểu sâu và truyền đạt thông tin từ dữ liệu bản đồ, ảnh vệ tinh, và các lớp địa lý khác. Các thư viện Python như GeoPandas, Rasterio, Matplotlib, và Xarray cung cấp giải pháp mạnh mẽ để hiển thị, phân tích và so sánh dữ liệu vector, raster, chuỗi thời gian, cũng như kết hợp nhiều lớp dữ liệu trên cùng một biểu đồ.

> **Lưu Ý**
> 
> Bạn có thể chạy trực tiếp notebook này bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/1yCTKGP-y3sXb0fZ1W3COiyChisrXJRIL?authuser=3) mà không cần cài đặt Python. Để tránh làm thay đổi nội dung gốc và thuận tiện cho việc lưu kết quả, hãy tạo một bản sao ( File → Save a copy in Drive ) trước khi chạy và chỉnh sửa mã nguồn trong notebook.

## 20.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Hiển thị dữ liệu vector (điểm, đường, đa giác) bằng GeoPandas và Matplotlib
- Hiển thị dữ liệu raster bằng Rasterio và Matplotlib
- Kết hợp vector và raster trên cùng một biểu đồ
- Tạo nhiều subplot để so sánh các lớp dữ liệu địa lý
- Trực quan hóa chuỗi thời gian raster với Xarray và Matplotlib


```python
import geopandas as gpd
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
```

## 20.2. Hiển thị dữ liệu vector

Bước đầu tiên trong trực quan hóa dữ liệu địa không gian là học cách hiển thị một lớp dữ liệu vector đơn giản. Trong phần này, chúng ta sẽ vẽ bản đồ ranh giới tỉnh thành của Việt Nam bằng GeoPandas và Matplotlib. Đây là nền tảng cơ bản nhất - chỉ một lớp dữ liệu vector với các tùy chỉnh màu sắc, đường viền, lưới tọa độ và nhãn trục. Dữ liệu vector được lấy từ GADM (Database of Global Administrative Areas) - nguồn dữ liệu ranh giới hành chính miễn phí và có chất lượng cao.


```python
# Đọc dữ liệu vector từ URL và tạo bản đồ
vector_path = 'https://geodata.ucdavis.edu/gadm/gadm4.1/json/gadm41_SGP_0.json'
gdf = gpd.read_file(vector_path)
# Tạo figure và axes
fig, ax = plt.subplots(figsize=(12, 20))
# Vẽ bản đồ với màu sắc và đường viền tùy chỉnh
gdf.plot(ax=ax, edgecolor='black', color='lightblue', legend=False)
# Thêm tiêu đề
ax.set_title('Singapore Provinces')
# Tùy chỉnh trục và lưới
ax.grid(True, color='gray', linestyle='--', linewidth=0.5)
ax.set_xlabel('Longitude', fontsize=12) # Thêm nhãn cho trục x
ax.set_ylabel('Latitude', fontsize=12) # Thêm nhãn cho trục y
# Thiết lập khoảng cách giữa các nhãn trục x và y
ax.xaxis.set_major_locator(ticker.MultipleLocator(0.1)) # Khoảng cách 0.2 độ giữa các nhãn trục x
ax.yaxis.set_major_locator(ticker.MultipleLocator(0.1)) # Khoảng cách 0.2 độ giữa các nhãn trục y
# # format the x/y-axis to show longitude and latitude values with degree symbol
ax.xaxis.set_major_formatter(ticker.FuncFormatter(lambda x, pos: f'{x:.1f}°')) # Format trục x để hiển thị giá trị kinh độ với ký hiệu độ
ax.yaxis.set_major_formatter(ticker.FuncFormatter(lambda y, pos: f'{y:.1f}°')) # Format trục y để hiển thị giá trị vĩ độ với ký hiệu độ
ax.tick_params(axis='both', which='major', labelsize=12) # Tùy chỉnh kích thước chữ trục
plt.show()
```


    
![png](output_3_0.png)
    


## 20.3. Hiển thị dữ liệu raster

Sau khi đã hiểu cách hiển thị dữ liệu vector, bước tiếp theo là làm việc với dữ liệu raster - dạng dữ liệu lưới (grid) như ảnh vệ tinh, dữ liệu nhiệt độ, độ cao, v.v. Trong phần này, chúng ta sẽ hiển thị một lớp dữ liệu raster đơn giản (nhiệt độ từ ERA5) sử dụng RioXarray và Matplotlib. Bạn sẽ học cách chọn băng dữ liệu, áp dụng colormap (bảng màu) phù hợp, và tùy chỉnh legend (thanh màu) để dễ dàng diễn giải giá trị dữ liệu.


```python
import rioxarray as rxr
```


```python
# Đọc dữ liệu raster từ URL và hiển thị thông tin cơ bản
raw_url = "https://raw.githubusercontent.com/tuyenhavan/geospatial_course/main/data/raster/era5_temp_2020_2024_vietnam.tif"
temp = rxr.open_rasterio(f"/vsicurl/{raw_url}")
temp.attrs = {}
```


```python
# Tạo figure và axes
fig, ax = plt.subplots(figsize=(10, 10))
# Hiển thị raster với colormap tùy chỉnh
first = temp[0]  # Chọn băng đầu tiên để hiển thị
first.plot(ax=ax, cmap='viridis')  # Sử dụng colormap 'viridis' để hiển thị raster
# Thêm tiêu đề và nhãn trục
ax.set_title('MODIS NDVI (2015-2024)', fontsize=14)
ax.set_xlabel('Longitude', fontsize=12) # Thêm nhãn cho trục x
ax.set_ylabel('Latitude', fontsize=12) # Thêm nhãn cho trục y
ax.tick_params(axis='both', which='major', labelsize=12) # Tùy chỉnh kích thước chữ trục
plt.show()
```


    
![png](output_7_0.png)
    


## 20.4. Hiển thị đồng thời vector và raster trên cùng một biểu đồ

Khi đã nắm vững cách hiển thị từng loại dữ liệu riêng lẻ, bước tiếp theo là kết hợp chúng lại. Phần này minh họa cách chồng lớp dữ liệu vector (ranh giới tỉnh) lên trên lớp dữ liệu raster (nhiệt độ) trong cùng một biểu đồ. Đây là kỹ thuật quan trọng trong phân tích không gian, giúp bạn hiểu được sự phân bố của dữ liệu raster trong bối cảnh địa lý cụ thể. Bạn sẽ thấy rõ nhiệt độ thay đổi như thế nào qua các tỉnh thành khác nhau của Việt Nam.


```python
# Tạo figure và axes
fig, ax = plt.subplots(figsize=(10, 10))
# Hiển thị raster với colormap tùy chỉnh
first.plot(ax=ax, cmap='viridis')  # Sử dụng colormap 'viridis' để hiển thị raster
# Vẽ đường viền của các tỉnh lên trên raster
gdf.boundary.plot(ax=ax, edgecolor='black', linewidth=0.5)
# Thêm tiêu đề và nhãn trục
ax.set_title('MODIS NDVI (2015-2024) with Vietnam Provinces', fontsize=14)
ax.set_xlabel('Longitude', fontsize=12) # Thêm nhãn cho trục x
ax.set_ylabel('Latitude', fontsize=12) # Thêm nhãn cho trục y
ax.tick_params(axis='both', which='major', labelsize=12) # Tùy chỉnh kích thước chữ trục
plt.show()
```


    
![png](output_9_0.png)
    


## 20.5. Hiển thị nhiều subplot

Ở mức độ nâng cao nhất, chúng ta sẽ tạo nhiều subplot (biểu đồ con) để so sánh nhiều lớp dữ liệu cùng lúc. Phần này minh họa cách hiển thị nhiều băng dữ liệu raster khác nhau (ví dụ: nhiệt độ của 5 năm liên tiếp) trong một lưới subplot, đồng thời chồng lớp vector (ranh giới tỉnh) lên từng subplot. Kỹ thuật này rất hữu ích khi bạn cần so sánh sự thay đổi theo thời gian hoặc giữa các kịch bản khác nhau. Bạn sẽ học cách đồng bộ các trục tọa độ, tạo colorbar chung cho tất cả subplot, và tối ưu bố cục để dễ so sánh.


```python
# Tạo figure và axes
fig, axes = plt.subplots(1, 5, figsize=(20, 10), sharex =True, sharey=True)
# Lấy 4 băng đầu tiên và hiển thị chúng
for i in range(5):
    band = temp[i]
    plot = band.plot(ax=axes[i], cmap='viridis', add_colorbar=False)  # Hiển thị băng mà không thêm colorbar
    axes[i].set_title('')
    axes[i].set_xlabel('Longitude', fontsize=10)
    axes[i].set_ylabel('Latitude', fontsize=10)
    axes[i].tick_params(axis='both', which='major', labelsize=8)
    # Vẽ đường viền của các tỉnh lên trên raster
    gdf.boundary.plot(ax=axes[i], edgecolor='black', linewidth=0.5)
    if i>0:
        axes[i].set_ylabel('') # Ẩn nhãn trục y cho các subplot bên phải
    axes[i].tick_params(axis='both', which='major', labelsize=10) # Tùy chỉnh kích thước chữ trục x
    # Các bạn có thể thêm điều chỉnh khác như lưới, nhãn trục, v.v. tùy ý
# Tạo colorbar chung cho tất cả các subplot
cbar = fig.colorbar(plot, 
                    ax=axes, orientation='horizontal', # Đặt colorbar nằm ngang dưới các subplot
                    pad=0.1, extend='both', shrink=0.6) # shrink để điều chỉnh kích thước colorbar

cbar.set_label('NDVI Value', fontsize=12) # Thêm nhãn cho colorbar
plt.show()
```


    
![png](output_11_0.png)
    


## Tóm tắt

Bạn đã hoàn thành Bài 20 và học được cách trực quan hóa dữ liệu địa không gian - kỹ năng quan trọng giúp hiểu sâu và truyền đạt thông tin từ dữ liệu bản đồ, ảnh vệ tinh, và các lớp địa lý khác.

### Các khái niệm chính đã nắm vững:
- ✅ **Hiển thị dữ liệu vector**: Vẽ dữ liệu điểm, đường, đa giác bằng GeoPandas và Matplotlib với tùy chỉnh màu sắc, lưới và nhãn trục
- ✅ **Hiển thị dữ liệu raster**: Đọc và hiển thị ảnh raster bằng RioXarray và Matplotlib với colormap linh hoạt
- ✅ **Kết hợp vector và raster**: Chồng dữ liệu vector lên raster để trực quan hóa đồng thời nhiều lớp địa lý trên cùng một biểu đồ
- ✅ **Nhiều subplot so sánh**: Tạo lưới subplot để so sánh nhiều băng dữ liệu raster cùng lúc với colorbar chung
- ✅ **Tùy chỉnh trục và lưới**: Định dạng kinh độ/vĩ độ với ký hiệu độ, điều chỉnh khoảng cách nhãn và kích thước chữ

### Kỹ năng bạn có thể áp dụng:
- Tạo bản đồ chuyên nghiệp từ dữ liệu vector địa lý với GeoPandas và Matplotlib
- Hiển thị và phân tích ảnh vệ tinh, dữ liệu raster nhiều băng một cách trực quan
- Kết hợp nhiều lớp dữ liệu địa không gian (vector + raster) trên cùng một biểu đồ
- Xây dựng bố cục subplot để so sánh chuỗi thời gian raster hoặc nhiều kịch bản dữ liệu
- Tích hợp kỹ năng trực quan hóa vào quy trình phân tích địa không gian hoàn chỉnh với Python
