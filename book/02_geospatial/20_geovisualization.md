#  Bài 19: Trực Quan hóa Dữ liệu Địa không gian

Trực quan hóa dữ liệu địa không gian là bước quan trọng giúp hiểu sâu và truyền đạt thông tin từ dữ liệu bản đồ, ảnh vệ tinh, và các lớp địa lý khác. Các thư viện Python như GeoPandas, Rasterio, Matplotlib, và Xarray cung cấp giải pháp mạnh mẽ để hiển thị, phân tích và so sánh dữ liệu vector, raster, chuỗi thời gian, cũng như kết hợp nhiều lớp dữ liệu trên cùng một biểu đồ.

## 19.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:
- **Hiển thị dữ liệu vector** (điểm, đường, đa giác) bằng GeoPandas và Matplotlib
- **Hiển thị dữ liệu raster** bằng Rasterio và Matplotlib
- **Kết hợp vector và raster** trên cùng một biểu đồ
- **Tạo nhiều subplot** để so sánh các lớp dữ liệu địa lý
- **Trực quan hóa chuỗi thời gian raster** với Xarray và Matplotlib

## 19.2. Hiển thị dữ liệu vector
Phần này minh họa cách vẽ dữ liệu vector (điểm, đường, đa giác) bằng GeoPandas và Matplotlib.


```python
import geopandas as gpd
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
vector_path = 'https://geodata.ucdavis.edu/gadm/gadm4.1/json/gadm41_VNM_1.json'
gdf = gpd.read_file(vector_path)
fig, ax = plt.subplots(figsize=(12, 20))
gdf.plot(ax=ax, edgecolor='black', color='lightblue', legend=False)
ax.set_title('Vietnam Provinces')
ax.grid(True, color='gray', linestyle='--', linewidth=0.5)
ax.set_xlabel('Longitude', fontsize=12)
ax.set_ylabel('Latitude', fontsize=12)
# set multiplelocators for x and y axes to show ticks every 1 degree
ax.xaxis.set_major_locator(ticker.MultipleLocator(2))
ax.yaxis.set_major_locator(ticker.MultipleLocator(3))
# format the x/y-axis to show longitude and latitude values with degree symbol
ax.xaxis.set_major_formatter(ticker.FuncFormatter(lambda x, pos: f'{x:.1f}°'))
ax.yaxis.set_major_formatter(ticker.FuncFormatter(lambda y, pos: f'{y:.1f}°'))
ax.tick_params(axis='both', which='major', labelsize=12)
plt.show()
```


    
![png](output_2_0.png)
    


## 19.3. Hiển thị dữ liệu raster
Phần này minh họa cách hiển thị dữ liệu raster bằng Rasterio và Matplotlib.


```python
import rioxarray as rxr

raw_url = "https://raw.githubusercontent.com/tuyenhavan/geospatial_course/main/data/raster/MODIS_NDVI_2015_2024.tif"
test = rxr.open_rasterio(f"/vsicurl/{raw_url}")
```


```python
import rasterio
from rasterio.plot import show
raster_path = r"G:\My Drive\python\python_course\data\raster\era5_precip_2020_2024_vietnam.tif" # Replace with your raster file
with rasterio.open(raster_path) as src:
    fig, ax = plt.subplots(figsize=(8, 8))
    show(src, ax=ax, title='Raster Example')
plt.show()
```


    
![png](output_5_0.png)
    


## 19.4. Hiển thị đồng thời vector và raster trên cùng một biểu đồ
Phần này minh họa cách chồng dữ liệu vector lên raster để trực quan hóa kết hợp.


```python
with rasterio.open(raster_path) as src:
    fig, ax = plt.subplots(figsize=(12, 20))
    show(src, ax=ax, title='Raster with Vector Overlay')
    gdf.plot(ax=ax, facecolor='none', edgecolor='red', linewidth=0.5)
plt.show()
```

## 19.5. Hiển thị nhiều subplot
Phần này minh họa cách tạo nhiều subplot để so sánh các lớp dữ liệu địa lý khác nhau.


```python
fig, axes = plt.subplots(1, 2, figsize=(16, 8))
gdf.plot(ax=axes[0], edgecolor='black', column='NAME_1', legend=False)
axes[0].set_title('Vector: Vietnam Provinces')
with rasterio.open(raster_path) as src:
    show(src, ax=axes[1], title='Raster Example')
plt.tight_layout()
plt.show()
```

## 19.6. Hiển thị chuỗi thời gian dữ liệu raster
Phần này minh họa cách trực quan hóa chuỗi ảnh raster theo thời gian bằng xarray và matplotlib.


```python
import rioxarray as rxr
import pandas as pd
data_name = "era5_precip_2020_2024_vietnam.tif"
url = f"/vsicurl/https://raw.githubusercontent.com/tuyenhavan/geospatial_course/main/data/raster/{data_name}"
ds = rxr.open_rasterio(url)
```


```python
ds.attrs = {}  # Clear attributes for simplicity
time_list = pd.date_range('2020-01-01', periods=ds.shape[0], freq='ME')
ds['band'] = time_list
ds = ds.rename({'band': 'time'})
ds = ds.groupby('time.year').mean('time')
fig, axes = plt.subplots(ncols=5, nrows=1, figsize=(10, 6))
for i in range(5):
    ds.sel(year=2020 + i).plot(ax=axes[i], cmap='viridis')
    axes[i].set_title(f'Average Temp {2020 + i}')
plt.tight_layout()
plt.show()
```


    
![png](output_12_0.png)
    



```python

```
