# Bài 5: Phân tích Dữ liệu vector với GeoPandas

GeoPandas là thư viện mạnh mẽ nhất cho phân tích dữ liệu địa không gian trong Python, kết hợp sức mạnh của pandas và Shapely để mang đến trải nghiệm xử lý dữ liệu GIS hoàn hảo.

## Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

1. **Tạo và manipulate GeoDataFrames** từ các nguồn dữ liệu đa dạng
2. **Đọc và ghi dữ liệu địa không gian** ở nhiều format khác nhau (Shapefile, GeoJSON, etc.)
3. **Làm chủ hệ tọa độ (CRS)** và thực hiện chuyển đổi projection chính xác
4. **Tạo visualizations và maps** chuyên nghiệp với matplotlib integration
5. **Thực hiện spatial operations** phức tạp như joins, overlays, và buffer operations
6. **Tích hợp GeoPandas** với các thư viện GIS khác trong ecosystem Python

- **Import thư viện cần thiết**


```python
# Import các thư viện chính
import geopandas as gpd           # Thư viện chính cho phân tích địa không gian
import pandas as pd              # Xử lý dữ liệu bảng
import numpy as np               # Tính toán số học
import matplotlib.pyplot as plt  # Vẽ biểu đồ và bản đồ
import matplotlib.ticker as ticker  # Quản lý thang đo trục
from shapely.geometry import Point, LineString, Polygon
import os 
import warnings                  # Quản lý cảnh báo
warnings.filterwarnings('ignore')  # Ẩn cảnh báo không cần thiết
outpath = r'G:\My Drive\python\python_course\data'
```

## 1. Tạo và hiểu GeoDataFrames

GeoDataFrame là **pandas DataFrame đặc biệt** có cột geometry chứa các đối tượng hình học Shapely. Đây là nền tảng của mọi phân tích địa không gian trong GeoPandas.

- **Tạo GeoDataFrame từ dictionary**


```python
# Dữ liệu thành phố lớn Việt Nam (tọa độ mang tính tương đối)
vietnam_cities_data = {
    'city': ['Hà Nội', 'TP.HCM', 'Hải Phòng', 'Đà Nẵng', 'Cần Thơ', 'Biên Hòa', 'Huế', 'Nha Trang', 'Buôn Ma Thuột', 'Quy Nhon'],
    'province': ['Hà Nội', 'TP.HCM', 'Hải Phòng', 'Đà Nẵng', 'Cần Thơ', 'Đồng Nai', 'Thừa Thiên Huế', 'Khánh Hòa', 'Đắk Lắk', 'Bình Định'],
    'region': ['Miền Bắc', 'Miền Nam', 'Miền Bắc', 'Miền Trung', 'Miền Nam', 'Miền Nam', 'Miền Trung', 'Miền Trung', 'Miền Trung', 'Miền Trung'],
    'population': [8246600, 8993082, 2028514, 1134310, 1282937, 1104800, 455230, 423000, 340000, 284000],
    'longitude': [105.8542, 106.6297, 106.6881, 108.2022, 105.7469, 106.8439, 107.5905, 109.1967, 108.0373, 109.2189],
    'latitude': [21.0285, 10.8231, 20.8449, 16.0544, 10.0452, 10.9460, 16.4637, 12.2585, 12.6667, 13.7830],
    'is_port_city': [False, True, True, True, True, False, False, True, False, True],
}

# Tạo DataFrame thường trước
df = pd.DataFrame(vietnam_cities_data)
# Chuyển đổi thành GeoDataFrame
geometry = [Point(xy) for xy in zip(df.longitude, df.latitude)]
gdf = gpd.GeoDataFrame(
    df.drop(['longitude', 'latitude'], axis=1), 
    geometry=geometry, 
    crs='EPSG:4326'
)
gdf.head()
```


```python
# Thông tin về total_bounds
total_bounds = gdf.total_bounds
print("Total Bounds:", total_bounds)
# Retrieve x and y 
x = gdf.geometry.x
y = gdf.geometry.y
```

- **Tạo GeoDataFrame từ list**


```python
# Tạo GeoDataFrames từ polygons sử dụng shapely và gán vào geometry
# Tạo một số polygons ví dụ
geometry = [
    Polygon([(105, 20), (106, 20), (106, 21), (105, 21)]),
    Polygon([(106, 10), (107, 10), (107, 11), (106, 11)]),
    Polygon([(108, 16), (109, 16), (109, 17), (108, 17)]),
    Polygon([(109, 12), (110, 12), (110, 13), (109, 13)]),
    Polygon([(105, 10), (106, 10), (106, 11), (105, 11)]),
    Polygon([(106, 10), (107, 10), (107, 11), (106, 11)]),
    Polygon([(107, 16), (108, 16), (108, 17), (107, 17)]),
    Polygon([(109, 12), (110, 12), (110, 13), (109, 13)]),
    Polygon([(108, 12), (109, 12), (109, 13), (108, 13)]),
    Polygon([(109, 13), (110, 13), (110, 14), (109, 14)]),
]
polygon = gpd.GeoDataFrame(
    geometry=geometry,
    crs='EPSG:4326'
)
```


```python
# Tạo GeoDataFrame từ lines sử dụng shapely và gán vào geometry
# Tạo một số lines ví dụ
geometry = [
    LineString([(105, 20), (106, 21)]),
    LineString([(106, 10), (107, 11)]),
    LineString([(108, 16), (109, 17)]),
    LineString([(109, 12), (110, 13)]),
    LineString([(105, 10), (106, 11)]),
    LineString([(106, 10), (107, 11)]),
    LineString([(107, 16), (108, 17)]),
    LineString([(109, 12), (110, 13)]),
    LineString([(108, 12), (109, 13)]),
    LineString([(109, 13), (110, 14)]),
]
lines = gpd.GeoDataFrame(
    geometry=geometry,
    crs='EPSG:4326'
)
```

## 2. Đọc và Lưu Dữ liệu Địa không gian

GeoPandas có thể đọc và ghi hơn **20 định dạng** địa không gian khác nhau. Đây là kỹ năng thiết yếu cho công việc thực tế.

Dữ liệu sử dụng trong notebook này được tải từ [GADM](https://gadm.org/) và NDVI từ ảnh MODI.


```python
# Đọc dữ liệu từ local machine
districts = gpd.read_file(r'G:\My Drive\python\python_course\data\Vietnam_districts.geojson')
# Chuyển crs từ 4326 sang 32648 (UTM 48N)
districts = districts.to_crs(epsg=32648)
# Thêm cột diện tích 
districts['area_km2'] = districts.geometry.area/1e6  # Chuyển từ m2 sang km2
# Vietnam map 
provinces = gpd.read_file(r'G:\My Drive\python\python_course\data\Vietnam_provinces.geojson')
# Đổi tên cột 
provinces = provinces[['VARNAME_1', 'geometry']]
provinces.columns = ['province', 'geometry']
```


```python
# Lọc dữ liệu theo điều kiện
subset = districts[districts['area_km2'] > 500]  # Lọc các huyện có diện tích lớn hơn 500 km2
subset = subset.reset_index(drop=True)
# Giữ lại các cột cần thiết
subset = subset[['NAME_1','NAME_2', 'area_km2', 'geometry']]
# Đổi tên cột cho dễ hiểu
subset = subset.rename(columns={'NAME_1': 'province', 'NAME_2': 'district'})
subset.head()
```


```python
# # Lưu dữ liệu ra file GeoJSON
# subset.to_file(os.path.join(outpath, 'Vietnam_districts.geojson'), driver='GeoJSON')
# # Lưu dữ liệu ra file Shapefile
# subset.to_file(os.path.join(outpath, 'Vietnam_districts.shp'), driver='ESRI Shapefile')
# # Lưu dữ liệu ra file Parquet
# subset.to_file(os.path.join(outpath, 'Vietnam_districts.parquet'), driver='Parquet')
# # Lưu dữ liệu ra file GeoPackage
# subset.to_file(os.path.join(outpath, 'Vietnam_districts.gpkg'), driver='GPKG')
```


```python
# Đọc dữ liệu từ url 
vietnam_data = gpd.read_file('https://geodata.ucdavis.edu/gadm/gadm4.1/json/gadm41_VNM_0.json')
vietnam_data.head()
```

## 3. Trực quan hóa dữ liệu địa không gian

GeoPandas tích hợp matplotlib để tạo bản đồ đẹp một cách dễ dàng.


```python
# Trực quan hóa dữ liệu địa không gian 
from matplotlib.colors import Normalize
from matplotlib.cm import ScalarMappable
# Chuyển qua hệ tọa độ địa lý EPSG:4326 để vẽ bản đồ
districts = districts.to_crs(epsg=4326)
# Biểu đồ có 3 subplots theo hàng
fig, axes = plt.subplots(1, 3, figsize=(12,12), sharex=True, sharey=True)
# Ví dụ thêm ghi chú (a), (b), (c) vào từng subplot
notes = list('abc')
# Loop qua từng subplot để vẽ bản đồ
for i, ax in enumerate(axes.ravel()):
    if i ==0:
        provinces.boundary.plot(ax=ax, linewidth=0.5, color='black')
        ax.set_title('Vietnam map', fontsize=13)
    elif i==1:
        vmin , vmax = districts['area_km2'].min(), districts['area_km2'].max()
        cmap = plt.cm.OrRd
        districts.plot(column='area_km2', ax=ax,  cmap=cmap, edgecolor='black', legend=False, vmin=vmin, vmax=vmax, linewidth=0.5)
        ax.set_title('District areas (km²)', fontsize=13)
        sm = ScalarMappable(norm=Normalize(vmin=vmin, vmax=vmax), cmap=cmap)
        cbar = fig.colorbar(sm, ax=ax, orientation='horizontal', pad=0.04, shrink=0.8, extend='both')
        cbar.set_label('Area (km²)', fontsize=10)
        ticks = np.linspace(vmin, vmax, 4)
        cbar.set_ticks(ticks)
        cbar_ticks = ticker.FormatStrFormatter('%.0f')
    else:
        vmin, vmax = 0, 1
        cmap = plt.cm.Spectral
        districts.plot(column='NDVI', ax=ax, cmap=cmap, edgecolor='black', vmin=0, vmax=1, legend=False, linewidth=0.5)
        ax.set_title('NDVI districts', fontsize=13)
        sm = ScalarMappable(norm=Normalize(vmin=vmin, vmax=vmax), cmap=cmap)
        cbar = fig.colorbar(sm, ax=ax, orientation='horizontal', pad=0.04, shrink=0.8, extend='both')
        cbar.set_label('NDVI', fontsize=10)

    ax.set_xticks(np.arange(103, 109.1, 3))
    ax.set_yticks(np.arange(9, 26, 3))
    ax.set_xticklabels([f"{lon}°E" for lon in ax.get_xticks()])
    ax.set_yticklabels([f"{lat}°N" for lat in ax.get_yticks()])
    ax.tick_params(axis='both', which='major', labelsize=10)
    ax.text(0.05, 0.95, f'({notes[i]})', transform=ax.transAxes, fontsize=10)
    ax.grid(True, linestyle='--', alpha=0.5)
```

## 4. Thao tác và Phân tích Không gian

Đây là phần mạnh nhất của GeoPandas - các thao tác không gian cho phép phân tích mối quan hệ địa lý phức tạp.


```python
# Create points for each provinces
province_centroids = provinces.copy()
province_centroids['geometry'] = province_centroids.centroid
# Tạo buffers cho mỗi điểm 50km 
province_centroids = province_centroids.to_crs(epsg=32648) # Chuyển sang UTM 48N để tính khoảng cách đúng
province_centroids['buffer_50km'] = province_centroids.buffer(50000)  # Buffer 50km
# Visualization 
fig, ax = plt.subplots(1, 1, figsize=(8,8))
points = province_centroids.to_crs(4326)
buffers = province_centroids[['buffer_50km', 'province']].set_geometry('buffer_50km').to_crs(4326)
points.plot(ax=ax, color='black', linewidth=1, label='Province Boundaries')
buffers.plot(ax=ax, color='None', edgecolor='darkred', facecolor='None')
ax.set_xticks(np.arange(103, 109.1, 3))
ax.set_yticks(np.arange(9, 26, 3))
ax.set_xticklabels([f"{lon}°E" for lon in ax.get_xticks()])
ax.set_yticklabels([f"{lat}°N" for lat in ax.get_yticks()])
ax.tick_params(axis='both', which='major', labelsize=10)
```


```python
# Tìm tất cả các polygons districts có intersect với points 
points = province_centroids.to_crs(epsg=4326) # cùng crs
districts = districts.to_crs(epsg=4326) # cùng crs 
intersected_districts = gpd.sjoin(districts, points[['province', 'geometry']], how='inner', predicate='intersects')
intersected_districts = intersected_districts[['province', 'NAME_2', 'geometry']]
```


```python
# Spatial join: Tìm điểm du lịch gần thành phố nào nhất
# Tạo thêm dữ liệu điểm du lịch
tourism_points_data = {
    'attraction': ['Hồ Hoàn Kiếm', 'Chợ Bến Thành', 'Cầu Rồng', 'Hội An Ancient Town', 
                   'Chùa Một Cột', 'Dinh Độc Lập', 'Bảo tàng Dân tộc học', 'Phố đi bộ Nguyễn Huệ'],
    'type': ['Hồ', 'Chợ', 'Cầu', 'Phố cổ', 'Chùa', 'Di tích', 'Bảo tàng', 'Phố đi bộ'],
    'city_nearby': ['Hà Nội', 'TP.HCM', 'Đà Nẵng', 'Hội An', 'Hà Nội', 'TP.HCM', 'Hà Nội', 'TP.HCM'],
    'longitude': [105.8551, 106.6980, 108.2277, 108.3269, 105.8341, 106.6958, 105.8364, 106.7009],
    'latitude': [21.0285, 10.8231, 16.0613, 15.8801, 21.0369, 10.7769, 21.0368, 10.7796],
    'rating': [4.5, 4.2, 4.7, 4.8, 4.3, 4.1, 4.0, 4.4]
}
# Tạo DataFrame thường trước
tourism_df = pd.DataFrame(tourism_points_data)
# Chuyển đổi thành GeoDataFrame
tourism_geometry = [Point(xy) for xy in zip(tourism_df.longitude, tourism_df.latitude)]
tourism_gdf = gpd.GeoDataFrame(
    tourism_df.drop(['longitude', 'latitude'], axis=1), geometry=tourism_geometry)
tourism_gdf.set_crs(epsg=4326, inplace=True)
# Thực hiện spatial join để tìm thành phố gần nhất cho mỗi điểm du lịch
joined = gpd.sjoin_nearest(tourism_gdf, provinces[['province', 'geometry']], how='left', distance_col='distance_to_city')
joined = joined[['attraction', 'type', 'city_nearby', 'province', 'distance_to_city', 'geometry']]
```


```python
# Spatial join với buffer
point_buffer = points[['province', 'buffer_50km']].rename(columns={'buffer_50km': 'geometry'})
point_buffer = point_buffer.set_geometry('geometry')
joined_buffer = gpd.sjoin(tourism_gdf, districts, how='left', predicate='within')
```


```python
# Các phương thức khác
province_centroids = province_centroids.to_crs(epsg=32648) # Chuyển sang UTM 48N để tính khoảng cách đúng
buffer1 = province_centroids.sample(n=5).buffer(10000) # Buffer 100km 
buffer2 = province_centroids.sample(10).buffer(20000) # Buffer 200km 
# Union 
union_result = buffer1.union(buffer2)
# # Dissolve theo region (hợp nhất các subregion thành region)
combine = gpd.pd.concat([gpd.GeoDataFrame(geometry=buffer1), gpd.GeoDataFrame(geometry=buffer2)]).dissolve()
```


```python
# Chuyển đổi hệ tọa độ
crs_3405 = districts.to_crs(3405) # Chuyển đổi từ 4326 sang EPSG:3405 (VN-2000)
# Chuyển qua UTM 48N 
crs_utm = districts.to_crs(32648) # EPSG:32648 là UTM zone 48N
```

## Tóm tắt

Bạn đã hoàn thành Bài 5 và học được GeoPandas - thư viện "con dao Thụy Sĩ" cho geospatial data analysis trong Python.

### Các khái niệm chính đã nắm vững:
- ✅ **GeoDataFrames**: Pandas DataFrames với geometry column cho spatial data
- ✅ **Spatial I/O**: Đọc/ghi Shapefile, GeoJSON, GPKG với read_file() và to_file()
- ✅ **CRS management**: Coordinate reference systems với to_crs() transformations
- ✅ **Spatial operations**: buffer(), intersects(), within() cho geometric analysis
- ✅ **Spatial joins**: Kết hợp datasets dựa trên spatial relationships
- ✅ **Plotting integration**: Native matplotlib support với .plot() method
- ✅ **Performance optimization**: Spatial indexing và efficient operations cho big data

### Kỹ năng bạn có thể áp dụng:
- Thực hiện comprehensive spatial analysis với pandas-like syntax
- Xử lý và visualize complex geospatial datasets một cách hiệu quả
- Tích hợp spatial data với business intelligence và data science workflows
- Phát triển location-based applications và market analysis tools
- Chuẩn bị expertise foundation cho advanced GIS development và spatial data science
