# Bài 16: Mảng N-Chiều cho Dữ liệu raster với xarray

XArray là thư viện Python mạnh mẽ cho việc xử lý dữ liệu mảng N-chiều có nhãn, đặc biệt thiết yếu cho climate science và geospatial analysis.

## 16.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

1. **Hiểu cấu trúc XArray** - Dataset, DataArray và coordinate systems
2. **Manipulate multi-dimensional data** từ NetCDF, Zarr và climate datasets  
3. **Thực hiện indexing và selection** theo dimensions địa lý và thời gian
4. **Áp dụng computations và statistics** cho dữ liệu đa chiều
5. **Sử dụng GroupBy operations** cho temporal analysis và seasonal patterns
6. **Tích hợp Advanced operations** với apply_ufunc và custom functions
7. **Visualize complex datasets** với built-in plotting capabilities
8. **Kết hợp với ecosystem** khác (rasterio, dask, geopandas).


```python
import xarray as xr
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import os
from datetime import datetime, timedelta
import warnings
warnings.filterwarnings('ignore')
```

## 16.2. Cấu trúc Dữ liệu XArray Cơ bản

XArray có **2 cấu trúc dữ liệu chính** cần hiểu rõ trước khi áp dụng:

### **DataArray - Mảng đa chiều có nhãn**
- **Khái niệm**: Như NumPy array nhưng có coordinates và labels
- **Thành phần**: `data` + `dimensions` + `coordinates` + `attributes`
- **Ứng dụng**: Biến đơn lẻ (nhiệt độ, mưa, NDVI) trong không gian-thời gian

### **Dataset - Tập hợp nhiều DataArrays**  
- **Khái niệm**: Như DataFrame của Pandas nhưng cho dữ liệu đa chiều
- **Thành phần**: Nhiều DataArrays có chung coordinates system
- **Ứng dụng**: Datasets khí hậu hoàn chỉnh (temp + rainfall + humidity + wind)

![image-2.png](image-2.png)


```python
# Tạo DataArray đơn giản nhất - chỉ có data
simple_data = np.array([25.5, 26.8, 24.2, 27.1, 23.9])
simple_da = xr.DataArray(simple_data)
type(simple_da)  # Output: xarray.core.dataarray.DataArray
```




    xarray.core.dataarray.DataArray




```python
# Thêm dimensions và coordinates vào DataArray
# Dữ liệu nhiệt độ cho 5 thành phố Việt Nam
cities = ['Hà Nội', 'TP.HCM', 'Đà Nẵng', 'Cần Thơ', 'Hải Phòng']
temperatures = [25.5, 28.8, 26.2, 27.8, 24.9]

# Tạo DataArray với dimension labels
temp_cities = xr.DataArray(
    temperatures,
    dims=['city'],  # Tên dimension
    coords={'city': cities}  # Labels cho dimension
)
temp_cities.dims, temp_cities.coords
```




    (('city',),
     Coordinates:
       * city     (city) <U9 180B 'Hà Nội' 'TP.HCM' 'Đà Nẵng' 'Cần Thơ' 'Hải Phòng')




```python
# Tạo DataArray hoàn chỉnh với attributes
temp_complete = xr.DataArray(
    temperatures,
    dims=['city'],
    coords={'city': cities},
    attrs={
        'long_name': 'Average Annual Temperature',
        'units': 'degrees_celsius', 
        'description': 'Annual average temperature for major Vietnamese cities',
        'country': 'Vietnam',
        'created_by': 'Ha Van Tuyen'
    }
)
```


```python
# Tạo dữ liệu nhiệt độ 2D random cho Việt Nam
lats = np.linspace(10.5, 21.0, 100)  # Vĩ độ từ 10.5°N đến 21.0°N
lons = np.linspace(102.0, 109.5, 100)  # Kinh độ từ 102.0°E đến 109.5°E

# Tạo temperature grid (100x100) với giá trị ngẫu nhiên
temp_grid = np.random.uniform(20.0, 35.0, size=(len(lats), len(lons)))

# Tạo 2D DataArray
temp_data = xr.DataArray(
    temp_grid,
    dims=['latitude', 'longitude'],
    coords={
        'latitude': (['latitude'], lats, {'units': 'degrees_north'}),
        'longitude': (['longitude'], lons, {'units': 'degrees_east'})
    },
    attrs={
        'long_name': 'Surface Air Temperature',
        'units': 'degrees_celsius',
        'country': 'Vietnam'
    }
)
print(f'dimensions: {temp_data.dims}')
print(f'coordinates: {temp_data.coords}')
print(f'attributes: {temp_data.attrs}')

```

    dimensions: ('latitude', 'longitude')
    coordinates: Coordinates:
      * latitude   (latitude) float64 800B 10.5 10.61 10.71 ... 20.79 20.89 21.0
      * longitude  (longitude) float64 800B 102.0 102.1 102.2 ... 109.3 109.4 109.5
    attributes: {'long_name': 'Surface Air Temperature', 'units': 'degrees_celsius', 'country': 'Vietnam'}
    


```python
# Đọc dữ liệu nhiệt độ hàng tháng Việt Nam từ file raster
file = r'G:\My Drive\python\python_course\data\raster\era5_temperature_vietnam.tif'
temp = xr.open_dataarray(file)
temp.attrs = ''
print(f'dimensions: {temp.dims}')
print(f'coordinates: {temp.coords}')
print(f'attributes: {temp.attrs}')
# band hien chua co thong tin thoi gian
time = pd.date_range(start='2020-01-01', periods=temp.sizes['band'], freq='M')
temp['band'] = time
temp = temp.rename({'band': 'time'})
print(f'Updated dimensions: {temp.dims}')
temp.time.values[-1]
```

    dimensions: ('band', 'y', 'x')
    coordinates: Coordinates:
      * band         (band) int64 12kB 1 2 3 4 5 6 ... 1435 1436 1437 1438 1439 1440
      * y            (y) float64 1kB 23.4 23.31 23.22 23.13 ... 8.579 8.489 8.399
      * x            (x) float64 656B 102.2 102.3 102.4 102.5 ... 109.3 109.4 109.5
        spatial_ref  int64 8B ...
    attributes: {}
    Updated dimensions: ('time', 'y', 'x')
    




    np.datetime64('2023-12-10T00:00:00.000000000')



## 2. Dataset - Kết hợp nhiều DataArrays

**Dataset** là cấu trúc dữ liệu chính của XArray để làm việc với nhiều biến cùng lúc:

### **Dataset vs DataArray**
- **DataArray**: 1 biến duy nhất (ví dụ: chỉ temperature)  
- **Dataset**: Nhiều biến có chung coordinates (temp + rainfall + humidity + wind)

### **Cách tạo Dataset**
- **From DataArrays**: Kết hợp các DataArrays có sẵn
- **From dict**: Tạo từ dictionary của arrays
- **From files**: Load từ NetCDF, Zarr, GRIB


```python

# Tạo thêm một biến: humidity (độ ẩm)
humidity_data = 70 + 10 * np.random.random((12, 3, 3))  # 70-80% humidity
humidity_3d = xr.DataArray(
    humidity_data,
    dims=['time', 'latitude', 'longitude'],
    coords=temp_3d.coords,  # Dùng chung coordinates
    attrs={
        'long_name': 'Relative Humidity',
        'units': 'percent',
        'description': 'Monthly relative humidity for Vietnam',
    }
)

# Tạo Dataset từ DataArrays
climate_ds = xr.Dataset({
    'temperature': temp_3d,
    'humidity': humidity_3d
})

print(f"Variables in dataset: {list(climate_ds.data_vars.keys())}")
print(f"Shared dimensions: {dict(climate_ds.dims)}")
print(f"Shared coordinates: {list(climate_ds.coords.keys())}")
climate_ds
```

    Variables in dataset: ['temperature', 'humidity']
    Shared dimensions: {'time': 12, 'latitude': 3, 'longitude': 3}
    Shared coordinates: ['time', 'latitude', 'longitude']
    




<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in notebooks */

:root {
  --xr-font-color0: var(
    --jp-content-font-color0,
    var(--pst-color-text-base rgba(0, 0, 0, 1))
  );
  --xr-font-color2: var(
    --jp-content-font-color2,
    var(--pst-color-text-base, rgba(0, 0, 0, 0.54))
  );
  --xr-font-color3: var(
    --jp-content-font-color3,
    var(--pst-color-text-base, rgba(0, 0, 0, 0.38))
  );
  --xr-border-color: var(
    --jp-border-color2,
    hsl(from var(--pst-color-on-background, white) h s calc(l - 10))
  );
  --xr-disabled-color: var(
    --jp-layout-color3,
    hsl(from var(--pst-color-on-background, white) h s calc(l - 40))
  );
  --xr-background-color: var(
    --jp-layout-color0,
    var(--pst-color-on-background, white)
  );
  --xr-background-color-row-even: var(
    --jp-layout-color1,
    hsl(from var(--pst-color-on-background, white) h s calc(l - 5))
  );
  --xr-background-color-row-odd: var(
    --jp-layout-color2,
    hsl(from var(--pst-color-on-background, white) h s calc(l - 15))
  );
}

html[theme="dark"],
html[data-theme="dark"],
body[data-theme="dark"],
body.vscode-dark {
  --xr-font-color0: var(
    --jp-content-font-color0,
    var(--pst-color-text-base, rgba(255, 255, 255, 1))
  );
  --xr-font-color2: var(
    --jp-content-font-color2,
    var(--pst-color-text-base, rgba(255, 255, 255, 0.54))
  );
  --xr-font-color3: var(
    --jp-content-font-color3,
    var(--pst-color-text-base, rgba(255, 255, 255, 0.38))
  );
  --xr-border-color: var(
    --jp-border-color2,
    hsl(from var(--pst-color-on-background, #111111) h s calc(l + 10))
  );
  --xr-disabled-color: var(
    --jp-layout-color3,
    hsl(from var(--pst-color-on-background, #111111) h s calc(l + 40))
  );
  --xr-background-color: var(
    --jp-layout-color0,
    var(--pst-color-on-background, #111111)
  );
  --xr-background-color-row-even: var(
    --jp-layout-color1,
    hsl(from var(--pst-color-on-background, #111111) h s calc(l + 5))
  );
  --xr-background-color-row-odd: var(
    --jp-layout-color2,
    hsl(from var(--pst-color-on-background, #111111) h s calc(l + 15))
  );
}

.xr-wrap {
  display: block !important;
  min-width: 300px;
  max-width: 700px;
  line-height: 1.6;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-obj-name,
.xr-group-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-group-name::before {
  content: "📁";
  padding-right: 0.3em;
}

.xr-group-name,
.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 0 20px 0 20px;
  margin-block-start: 0;
  margin-block-end: 0;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: inline-block;
  opacity: 0;
  height: 0;
  margin: 0;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
  border: 2px solid transparent !important;
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:focus + label {
  border: 2px solid var(--xr-font-color0) !important;
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: "►";
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: "▼";
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-top: 4px;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-group-box {
  display: inline-grid;
  grid-template-columns: 0px 20px auto;
  width: 100%;
}

.xr-group-box-vline {
  grid-column-start: 1;
  border-right: 0.2em solid;
  border-color: var(--xr-border-color);
  width: 0px;
}

.xr-group-box-hline {
  grid-column-start: 2;
  grid-row-start: 1;
  height: 1em;
  width: 20px;
  border-bottom: 0.2em solid;
  border-color: var(--xr-border-color);
}

.xr-group-box-contents {
  grid-column-start: 3;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: "(";
}

.xr-dim-list:after {
  content: ")";
}

.xr-dim-list li:not(:last-child):after {
  content: ",";
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  border-color: var(--xr-background-color-row-odd);
  margin-bottom: 0;
  padding-top: 2px;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
  border-color: var(--xr-background-color-row-even);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-index-preview {
  grid-column: 2 / 5;
  color: var(--xr-font-color2);
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  display: none;
  border-top: 2px dotted var(--xr-background-color);
  padding-bottom: 20px !important;
  padding-top: 10px !important;
}

.xr-var-attrs-in + label,
.xr-var-data-in + label,
.xr-index-data-in + label {
  padding: 0 1px;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data,
.xr-index-data-in:checked ~ .xr-index-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-data > pre,
.xr-index-data > pre,
.xr-var-data > table > tbody > tr {
  background-color: transparent !important;
}

.xr-var-name span,
.xr-var-data,
.xr-index-name div,
.xr-index-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt,
.xr-attrs dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2,
.xr-no-icon {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}

.xr-var-attrs-in:checked + label > .xr-icon-file-text2,
.xr-var-data-in:checked + label > .xr-icon-database,
.xr-index-data-in:checked + label > .xr-icon-database {
  color: var(--xr-font-color0);
  filter: drop-shadow(1px 1px 5px var(--xr-font-color2));
  stroke-width: 0.8px;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.Dataset&gt; Size: 2kB
Dimensions:      (time: 12, latitude: 3, longitude: 3)
Coordinates:
  * time         (time) datetime64[ns] 96B 2023-01-01 2023-02-01 ... 2023-12-01
  * latitude     (latitude) float64 24B 21.0 16.0 10.5
  * longitude    (longitude) float64 24B 105.8 108.2 106.7
Data variables:
    temperature  (time, latitude, longitude) float64 864B 25.75 26.13 ... 28.09
    humidity     (time, latitude, longitude) float64 864B 73.64 79.72 ... 79.28</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.Dataset</div></div><ul class='xr-sections'><li class='xr-section-item'><input id='section-ac03b134-0aba-4b18-ae1b-af7e779f6b85' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-ac03b134-0aba-4b18-ae1b-af7e779f6b85' class='xr-section-summary'  title='Expand/collapse section'>Dimensions:</label><div class='xr-section-inline-details'><ul class='xr-dim-list'><li><span class='xr-has-index'>time</span>: 12</li><li><span class='xr-has-index'>latitude</span>: 3</li><li><span class='xr-has-index'>longitude</span>: 3</li></ul></div><div class='xr-section-details'></div></li><li class='xr-section-item'><input id='section-d9603e64-aa56-438c-83ad-f6096ccc01e4' class='xr-section-summary-in' type='checkbox'  checked><label for='section-d9603e64-aa56-438c-83ad-f6096ccc01e4' class='xr-section-summary' >Coordinates: <span>(3)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(time)</div><div class='xr-var-dtype'>datetime64[ns]</div><div class='xr-var-preview xr-preview'>2023-01-01 ... 2023-12-01</div><input id='attrs-414f4b08-e32c-4ccd-9890-59499f4f73ab' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-414f4b08-e32c-4ccd-9890-59499f4f73ab' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-2a8c8162-9866-40db-9ae1-923cbe59d30c' class='xr-var-data-in' type='checkbox'><label for='data-2a8c8162-9866-40db-9ae1-923cbe59d30c' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>Time</dd></dl></div><div class='xr-var-data'><pre>array([&#x27;2023-01-01T00:00:00.000000000&#x27;, &#x27;2023-02-01T00:00:00.000000000&#x27;,
       &#x27;2023-03-01T00:00:00.000000000&#x27;, &#x27;2023-04-01T00:00:00.000000000&#x27;,
       &#x27;2023-05-01T00:00:00.000000000&#x27;, &#x27;2023-06-01T00:00:00.000000000&#x27;,
       &#x27;2023-07-01T00:00:00.000000000&#x27;, &#x27;2023-08-01T00:00:00.000000000&#x27;,
       &#x27;2023-09-01T00:00:00.000000000&#x27;, &#x27;2023-10-01T00:00:00.000000000&#x27;,
       &#x27;2023-11-01T00:00:00.000000000&#x27;, &#x27;2023-12-01T00:00:00.000000000&#x27;],
      dtype=&#x27;datetime64[ns]&#x27;)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>latitude</span></div><div class='xr-var-dims'>(latitude)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>21.0 16.0 10.5</div><input id='attrs-9dca9b99-3361-4542-a7c4-eaad434c4e62' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-9dca9b99-3361-4542-a7c4-eaad434c4e62' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-0fa36783-7789-4a40-a7fd-348c3776ad9d' class='xr-var-data-in' type='checkbox'><label for='data-0fa36783-7789-4a40-a7fd-348c3776ad9d' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>degrees_north</dd></dl></div><div class='xr-var-data'><pre>array([21. , 16. , 10.5])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>longitude</span></div><div class='xr-var-dims'>(longitude)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>105.8 108.2 106.7</div><input id='attrs-a52775c9-5521-4421-be6e-3a2f7a86bc50' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-a52775c9-5521-4421-be6e-3a2f7a86bc50' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-4f702c49-20cf-45d9-9c8f-579f63093ca3' class='xr-var-data-in' type='checkbox'><label for='data-4f702c49-20cf-45d9-9c8f-579f63093ca3' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>degrees_east</dd></dl></div><div class='xr-var-data'><pre>array([105.8, 108.2, 106.7])</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-d76849b4-7a27-477b-8768-e4379495c64f' class='xr-section-summary-in' type='checkbox'  checked><label for='section-d76849b4-7a27-477b-8768-e4379495c64f' class='xr-section-summary' >Data variables: <span>(2)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>temperature</span></div><div class='xr-var-dims'>(time, latitude, longitude)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>25.75 26.13 27.42 ... 28.74 28.09</div><input id='attrs-c86187e5-7958-4590-9ba1-dfc108897755' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-c86187e5-7958-4590-9ba1-dfc108897755' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-08b08e54-e3dd-4909-96ca-5b3d4183b601' class='xr-var-data-in' type='checkbox'><label for='data-08b08e54-e3dd-4909-96ca-5b3d4183b601' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>Monthly Surface Air Temperature</dd><dt><span>units :</span></dt><dd>degrees_celsius</dd><dt><span>description :</span></dt><dd>2023 monthly temperature for Vietnam regions</dd><dt><span>country :</span></dt><dd>Vietnam</dd></dl></div><div class='xr-var-data'><pre>array([[[25.74835708, 26.13086785, 27.42384427],
        [27.56151493, 27.38292331, 28.08293152],
        [28.88960641, 29.18371736, 28.76526281]],

       [[26.77128002, 26.96829115, 27.86713512],
        [27.92098114, 27.54335988, 28.33754108],
        [28.81885624, 29.29358444, 30.15712367]],

       [[26.77803877, 27.22589896, 29.56487519],
        [28.41916266, 29.26581491, 29.21967671],
        [29.55985945, 30.5875121 , 30.15655402]],

       [[27.68784901, 27.89968066, 28.95415313],
        [28.49914669, 30.42613909, 30.19325139],
        [29.57114454, 31.21127246, 30.38957818]],

       [[27.33648261, 26.95221575, 28.16795778],
        [28.63048143, 29.6012841 , 30.01773495],
        [29.77422667, 30.38149896, 29.99278981]],

...

       [[23.90189669, 25.60626291, 26.77812001],
        [25.76399494, 27.00176645, 27.38081801],
        [26.77744012, 27.9806978 , 28.76901828]],

       [[23.75003617, 25.25027102, 24.05807664],
        [25.47890044, 25.81147273, 26.31844552],
        [26.41382958, 26.07416474, 27.15811325]],

       [[23.67855629, 24.93894702, 24.84086489],
        [24.3957532 , 25.24912148, 26.65770106],
        [26.26437555, 26.5351199 , 27.25663372]],

       [[23.81648797, 24.95227169, 25.01692265],
        [24.90411812, 25.57189512, 25.73619172],
        [26.51600933, 27.19847683, 27.27050592]],

       [[24.38270643, 24.49231463, 25.88967734],
        [25.62864274, 26.09886137, 27.11935714],
        [27.30202543, 28.74309295, 28.08728891]]])</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>humidity</span></div><div class='xr-var-dims'>(time, latitude, longitude)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>73.64 79.72 79.62 ... 70.15 79.28</div><input id='attrs-bfdb5a93-4715-4ae1-a695-4602eca037af' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-bfdb5a93-4715-4ae1-a695-4602eca037af' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-8b23df69-3d98-4998-bd55-667bf834f570' class='xr-var-data-in' type='checkbox'><label for='data-8b23df69-3d98-4998-bd55-667bf834f570' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>Relative Humidity</dd><dt><span>units :</span></dt><dd>percent</dd><dt><span>description :</span></dt><dd>Monthly relative humidity for Vietnam</dd></dl></div><div class='xr-var-data'><pre>array([[[73.63629602, 79.71782083, 79.62447295],
        [72.51782296, 74.97248506, 73.0087831 ],
        [72.84840494, 70.36886947, 76.09564334]],

       [[75.02679023, 70.51478751, 72.78646464],
        [79.08265886, 72.39561891, 71.44894872],
        [74.8945276 , 79.85650454, 72.42055272]],

       [[76.72135547, 77.61619615, 72.37637544],
        [77.28216349, 73.67783133, 76.32305831],
        [76.33529711, 75.35774684, 70.9028977 ]],

       [[78.35302496, 73.20780065, 71.8651851 ],
        [70.40775142, 75.90892943, 76.77564362],
        [70.16587829, 75.12093058, 72.26495775]],

       [[76.4517279 , 71.74366429, 76.90937738],
        [73.86735346, 79.36729989, 71.37520944],
        [73.41066351, 71.13473521, 79.24693618]],

...

       [[70.84139965, 71.61628714, 78.98554189],
        [76.0642906 , 70.09197052, 71.01471543],
        [76.63501769, 70.05061584, 71.60808051]],

       [[75.48733789, 76.91895198, 76.5196126 ],
        [72.24269309, 77.12179221, 72.37249087],
        [73.25399698, 77.46491405, 76.49632899]],

       [[78.4922341 , 76.57612892, 75.68308603],
        [70.93674768, 73.67715803, 72.65202368],
        [72.43989643, 79.73010555, 73.93097725]],

       [[78.92046555, 76.31138626, 77.94811304],
        [75.02637093, 75.76903885, 74.92517694],
        [71.95242988, 77.22452115, 72.80772362]],

       [[70.24315966, 76.45472296, 71.77110679],
        [79.40458584, 79.53928577, 79.1486439 ],
        [73.701587  , 70.15456617, 79.28318563]]])</pre></div></li></ul></div></li></ul></div></div>




```python
# Tạo Dataset từ dictionary
# Tạo dữ liệu wind speed
wind_data = 5 + 3 * np.random.random((12, 3, 3))  # 5-8 m/s

# Tạo Dataset trực tiếp từ dictionary
weather_ds = xr.Dataset(
    # Data variables
    data_vars={
        'temp': (['time', 'lat', 'lon'], temp_3d_data, {
            'long_name': 'Air Temperature',
            'units': 'celsius'
        }),
        'humidity': (['time', 'lat', 'lon'], humidity_data, {
            'long_name': 'Relative Humidity', 
            'units': 'percent'
        }),
        'wind_speed': (['time', 'lat', 'lon'], wind_data, {
            'long_name': 'Wind Speed',
            'units': 'm/s'
        })
    },
    # Coordinates
    coords={
        'time': (['time'], times),
        'lat': (['lat'], lats, {'units': 'degrees_north'}),
        'lon': (['lon'], lons, {'units': 'degrees_east'})
    },
    # Global attributes  
    attrs={
        'title': 'Vietnam Weather Dataset 2023',
        'institution': 'Vietnam Geospatial Course',
        'country': 'Vietnam'
    }
)
```


```python
# Access individual variables
print(f"      • temperature variable: {type(weather_ds['temp'])}")
print(f"      • Same as: {type(weather_ds.temp)}")  # Dot notation

# Dataset arithmetic - apply to all variables
weather_scaled = weather_ds * 1.1  # Scale all variables by 10%
print(f"      • Original temp mean: {weather_ds.temp.mean().values:.1f}°C")
print(f"      • Scaled temp mean: {weather_scaled.temp.mean().values:.1f}°C")

# Select subset of variables
temp_only = weather_ds[['temp', 'humidity']]  # Only temp and humidity
print(f"      • Original variables: {list(weather_ds.data_vars.keys())}")
print(f"      • Subset variables: {list(temp_only.data_vars.keys())}")

# Add new variable
weather_ds['heat_index'] = weather_ds.temp + (weather_ds.humidity - 50) * 0.1
print(f"      • Variables now: {list(weather_ds.data_vars.keys())}")
print(f"      • Heat index range: {weather_ds.heat_index.min().values:.1f} - {weather_ds.heat_index.max().values:.1f}")
```

          • temperature variable: <class 'xarray.core.dataarray.DataArray'>
          • Same as: <class 'xarray.core.dataarray.DataArray'>
          • Original temp mean: 27.4°C
          • Scaled temp mean: 30.2°C
          • Original variables: ['temp', 'humidity', 'wind_speed']
          • Subset variables: ['temp', 'humidity']
          • Variables now: ['temp', 'humidity', 'wind_speed', 'heat_index']
          • Heat index range: 26.0 - 33.7
    


```python
ds = xr.tutorial.load_dataset("air_temperature")
ds
```




<div><svg style="position: absolute; width: 0; height: 0; overflow: hidden">
<defs>
<symbol id="icon-database" viewBox="0 0 32 32">
<path d="M16 0c-8.837 0-16 2.239-16 5v4c0 2.761 7.163 5 16 5s16-2.239 16-5v-4c0-2.761-7.163-5-16-5z"></path>
<path d="M16 17c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
<path d="M16 26c-8.837 0-16-2.239-16-5v6c0 2.761 7.163 5 16 5s16-2.239 16-5v-6c0 2.761-7.163 5-16 5z"></path>
</symbol>
<symbol id="icon-file-text2" viewBox="0 0 32 32">
<path d="M28.681 7.159c-0.694-0.947-1.662-2.053-2.724-3.116s-2.169-2.030-3.116-2.724c-1.612-1.182-2.393-1.319-2.841-1.319h-15.5c-1.378 0-2.5 1.121-2.5 2.5v27c0 1.378 1.122 2.5 2.5 2.5h23c1.378 0 2.5-1.122 2.5-2.5v-19.5c0-0.448-0.137-1.23-1.319-2.841zM24.543 5.457c0.959 0.959 1.712 1.825 2.268 2.543h-4.811v-4.811c0.718 0.556 1.584 1.309 2.543 2.268zM28 29.5c0 0.271-0.229 0.5-0.5 0.5h-23c-0.271 0-0.5-0.229-0.5-0.5v-27c0-0.271 0.229-0.5 0.5-0.5 0 0 15.499-0 15.5 0v7c0 0.552 0.448 1 1 1h7v19.5z"></path>
<path d="M23 26h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 22h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
<path d="M23 18h-14c-0.552 0-1-0.448-1-1s0.448-1 1-1h14c0.552 0 1 0.448 1 1s-0.448 1-1 1z"></path>
</symbol>
</defs>
</svg>
<style>/* CSS stylesheet for displaying xarray objects in notebooks */

:root {
  --xr-font-color0: var(
    --jp-content-font-color0,
    var(--pst-color-text-base rgba(0, 0, 0, 1))
  );
  --xr-font-color2: var(
    --jp-content-font-color2,
    var(--pst-color-text-base, rgba(0, 0, 0, 0.54))
  );
  --xr-font-color3: var(
    --jp-content-font-color3,
    var(--pst-color-text-base, rgba(0, 0, 0, 0.38))
  );
  --xr-border-color: var(
    --jp-border-color2,
    hsl(from var(--pst-color-on-background, white) h s calc(l - 10))
  );
  --xr-disabled-color: var(
    --jp-layout-color3,
    hsl(from var(--pst-color-on-background, white) h s calc(l - 40))
  );
  --xr-background-color: var(
    --jp-layout-color0,
    var(--pst-color-on-background, white)
  );
  --xr-background-color-row-even: var(
    --jp-layout-color1,
    hsl(from var(--pst-color-on-background, white) h s calc(l - 5))
  );
  --xr-background-color-row-odd: var(
    --jp-layout-color2,
    hsl(from var(--pst-color-on-background, white) h s calc(l - 15))
  );
}

html[theme="dark"],
html[data-theme="dark"],
body[data-theme="dark"],
body.vscode-dark {
  --xr-font-color0: var(
    --jp-content-font-color0,
    var(--pst-color-text-base, rgba(255, 255, 255, 1))
  );
  --xr-font-color2: var(
    --jp-content-font-color2,
    var(--pst-color-text-base, rgba(255, 255, 255, 0.54))
  );
  --xr-font-color3: var(
    --jp-content-font-color3,
    var(--pst-color-text-base, rgba(255, 255, 255, 0.38))
  );
  --xr-border-color: var(
    --jp-border-color2,
    hsl(from var(--pst-color-on-background, #111111) h s calc(l + 10))
  );
  --xr-disabled-color: var(
    --jp-layout-color3,
    hsl(from var(--pst-color-on-background, #111111) h s calc(l + 40))
  );
  --xr-background-color: var(
    --jp-layout-color0,
    var(--pst-color-on-background, #111111)
  );
  --xr-background-color-row-even: var(
    --jp-layout-color1,
    hsl(from var(--pst-color-on-background, #111111) h s calc(l + 5))
  );
  --xr-background-color-row-odd: var(
    --jp-layout-color2,
    hsl(from var(--pst-color-on-background, #111111) h s calc(l + 15))
  );
}

.xr-wrap {
  display: block !important;
  min-width: 300px;
  max-width: 700px;
  line-height: 1.6;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
  margin-bottom: 4px;
  border-bottom: solid 1px var(--xr-border-color);
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-obj-name,
.xr-group-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-group-name::before {
  content: "📁";
  padding-right: 0.3em;
}

.xr-group-name,
.xr-obj-type {
  color: var(--xr-font-color2);
}

.xr-sections {
  padding-left: 0 !important;
  display: grid;
  grid-template-columns: 150px auto auto 1fr 0 20px 0 20px;
  margin-block-start: 0;
  margin-block-end: 0;
}

.xr-section-item {
  display: contents;
}

.xr-section-item input {
  display: inline-block;
  opacity: 0;
  height: 0;
  margin: 0;
}

.xr-section-item input + label {
  color: var(--xr-disabled-color);
  border: 2px solid transparent !important;
}

.xr-section-item input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item input:focus + label {
  border: 2px solid var(--xr-font-color0) !important;
}

.xr-section-item input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.5em;
}

.xr-section-summary-in:disabled + label {
  color: var(--xr-font-color2);
}

.xr-section-summary-in + label:before {
  display: inline-block;
  content: "►";
  font-size: 11px;
  width: 15px;
  text-align: center;
}

.xr-section-summary-in:disabled + label:before {
  color: var(--xr-disabled-color);
}

.xr-section-summary-in:checked + label:before {
  content: "▼";
}

.xr-section-summary-in:checked + label > span {
  display: none;
}

.xr-section-summary,
.xr-section-inline-details {
  padding-top: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  display: none;
  grid-column: 1 / -1;
  margin-top: 4px;
  margin-bottom: 5px;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-group-box {
  display: inline-grid;
  grid-template-columns: 0px 20px auto;
  width: 100%;
}

.xr-group-box-vline {
  grid-column-start: 1;
  border-right: 0.2em solid;
  border-color: var(--xr-border-color);
  width: 0px;
}

.xr-group-box-hline {
  grid-column-start: 2;
  grid-row-start: 1;
  height: 1em;
  width: 20px;
  border-bottom: 0.2em solid;
  border-color: var(--xr-border-color);
}

.xr-group-box-contents {
  grid-column-start: 3;
}

.xr-array-wrap {
  grid-column: 1 / -1;
  display: grid;
  grid-template-columns: 20px auto;
}

.xr-array-wrap > label {
  grid-column: 1;
  vertical-align: top;
}

.xr-preview {
  color: var(--xr-font-color3);
}

.xr-array-preview,
.xr-array-data {
  padding: 0 5px !important;
  grid-column: 2;
}

.xr-array-data,
.xr-array-in:checked ~ .xr-array-preview {
  display: none;
}

.xr-array-in:checked ~ .xr-array-data,
.xr-array-preview {
  display: inline-block;
}

.xr-dim-list {
  display: inline-block !important;
  list-style: none;
  padding: 0 !important;
  margin: 0;
}

.xr-dim-list li {
  display: inline-block;
  padding: 0;
  margin: 0;
}

.xr-dim-list:before {
  content: "(";
}

.xr-dim-list:after {
  content: ")";
}

.xr-dim-list li:not(:last-child):after {
  content: ",";
  padding-right: 5px;
}

.xr-has-index {
  font-weight: bold;
}

.xr-var-list,
.xr-var-item {
  display: contents;
}

.xr-var-item > div,
.xr-var-item label,
.xr-var-item > .xr-var-name span {
  background-color: var(--xr-background-color-row-even);
  border-color: var(--xr-background-color-row-odd);
  margin-bottom: 0;
  padding-top: 2px;
}

.xr-var-item > .xr-var-name:hover span {
  padding-right: 5px;
}

.xr-var-list > li:nth-child(odd) > div,
.xr-var-list > li:nth-child(odd) > label,
.xr-var-list > li:nth-child(odd) > .xr-var-name span {
  background-color: var(--xr-background-color-row-odd);
  border-color: var(--xr-background-color-row-even);
}

.xr-var-name {
  grid-column: 1;
}

.xr-var-dims {
  grid-column: 2;
}

.xr-var-dtype {
  grid-column: 3;
  text-align: right;
  color: var(--xr-font-color2);
}

.xr-var-preview {
  grid-column: 4;
}

.xr-index-preview {
  grid-column: 2 / 5;
  color: var(--xr-font-color2);
}

.xr-var-name,
.xr-var-dims,
.xr-var-dtype,
.xr-preview,
.xr-attrs dt {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-right: 10px;
}

.xr-var-name:hover,
.xr-var-dims:hover,
.xr-var-dtype:hover,
.xr-attrs dt:hover {
  overflow: visible;
  width: auto;
  z-index: 1;
}

.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  display: none;
  border-top: 2px dotted var(--xr-background-color);
  padding-bottom: 20px !important;
  padding-top: 10px !important;
}

.xr-var-attrs-in + label,
.xr-var-data-in + label,
.xr-index-data-in + label {
  padding: 0 1px;
}

.xr-var-attrs-in:checked ~ .xr-var-attrs,
.xr-var-data-in:checked ~ .xr-var-data,
.xr-index-data-in:checked ~ .xr-index-data {
  display: block;
}

.xr-var-data > table {
  float: right;
}

.xr-var-data > pre,
.xr-index-data > pre,
.xr-var-data > table > tbody > tr {
  background-color: transparent !important;
}

.xr-var-name span,
.xr-var-data,
.xr-index-name div,
.xr-index-data,
.xr-attrs {
  padding-left: 25px !important;
}

.xr-attrs,
.xr-var-attrs,
.xr-var-data,
.xr-index-data {
  grid-column: 1 / -1;
}

dl.xr-attrs {
  padding: 0;
  margin: 0;
  display: grid;
  grid-template-columns: 125px auto;
}

.xr-attrs dt,
.xr-attrs dd {
  padding: 0;
  margin: 0;
  float: left;
  padding-right: 10px;
  width: auto;
}

.xr-attrs dt {
  font-weight: normal;
  grid-column: 1;
}

.xr-attrs dt:hover span {
  display: inline-block;
  background: var(--xr-background-color);
  padding-right: 10px;
}

.xr-attrs dd {
  grid-column: 2;
  white-space: pre-wrap;
  word-break: break-all;
}

.xr-icon-database,
.xr-icon-file-text2,
.xr-no-icon {
  display: inline-block;
  vertical-align: middle;
  width: 1em;
  height: 1.5em !important;
  stroke-width: 0;
  stroke: currentColor;
  fill: currentColor;
}

.xr-var-attrs-in:checked + label > .xr-icon-file-text2,
.xr-var-data-in:checked + label > .xr-icon-database,
.xr-index-data-in:checked + label > .xr-icon-database {
  color: var(--xr-font-color0);
  filter: drop-shadow(1px 1px 5px var(--xr-font-color2));
  stroke-width: 0.8px;
}
</style><pre class='xr-text-repr-fallback'>&lt;xarray.Dataset&gt; Size: 31MB
Dimensions:  (time: 2920, lat: 25, lon: 53)
Coordinates:
  * time     (time) datetime64[ns] 23kB 2013-01-01 ... 2014-12-31T18:00:00
  * lat      (lat) float32 100B 75.0 72.5 70.0 67.5 65.0 ... 22.5 20.0 17.5 15.0
  * lon      (lon) float32 212B 200.0 202.5 205.0 207.5 ... 325.0 327.5 330.0
Data variables:
    air      (time, lat, lon) float64 31MB 241.2 242.5 243.5 ... 296.2 295.7
Attributes:
    Conventions:  COARDS
    title:        4x daily NMC reanalysis (1948)
    description:  Data is from NMC initialized reanalysis\n(4x/day).  These a...
    platform:     Model
    references:   http://www.esrl.noaa.gov/psd/data/gridded/data.ncep.reanaly...</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.Dataset</div></div><ul class='xr-sections'><li class='xr-section-item'><input id='section-09506522-7ab9-4385-beef-dd8f3441edad' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-09506522-7ab9-4385-beef-dd8f3441edad' class='xr-section-summary'  title='Expand/collapse section'>Dimensions:</label><div class='xr-section-inline-details'><ul class='xr-dim-list'><li><span class='xr-has-index'>time</span>: 2920</li><li><span class='xr-has-index'>lat</span>: 25</li><li><span class='xr-has-index'>lon</span>: 53</li></ul></div><div class='xr-section-details'></div></li><li class='xr-section-item'><input id='section-fe67093b-c527-4495-a876-9afc63d1c48b' class='xr-section-summary-in' type='checkbox'  checked><label for='section-fe67093b-c527-4495-a876-9afc63d1c48b' class='xr-section-summary' >Coordinates: <span>(3)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(time)</div><div class='xr-var-dtype'>datetime64[ns]</div><div class='xr-var-preview xr-preview'>2013-01-01 ... 2014-12-31T18:00:00</div><input id='attrs-85bfa2cc-2b68-4b45-a089-0f9aa52310de' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-85bfa2cc-2b68-4b45-a089-0f9aa52310de' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-8671853e-54f3-4c34-a6b2-cd0f60e024a8' class='xr-var-data-in' type='checkbox'><label for='data-8671853e-54f3-4c34-a6b2-cd0f60e024a8' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>standard_name :</span></dt><dd>time</dd><dt><span>long_name :</span></dt><dd>Time</dd></dl></div><div class='xr-var-data'><pre>array([&#x27;2013-01-01T00:00:00.000000000&#x27;, &#x27;2013-01-01T06:00:00.000000000&#x27;,
       &#x27;2013-01-01T12:00:00.000000000&#x27;, ..., &#x27;2014-12-31T06:00:00.000000000&#x27;,
       &#x27;2014-12-31T12:00:00.000000000&#x27;, &#x27;2014-12-31T18:00:00.000000000&#x27;],
      shape=(2920,), dtype=&#x27;datetime64[ns]&#x27;)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>lat</span></div><div class='xr-var-dims'>(lat)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>75.0 72.5 70.0 ... 20.0 17.5 15.0</div><input id='attrs-f71614de-4d1d-424d-a92b-b12109e3f17e' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-f71614de-4d1d-424d-a92b-b12109e3f17e' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-d2d4dd69-2dec-4804-aa5c-78480d5bd080' class='xr-var-data-in' type='checkbox'><label for='data-d2d4dd69-2dec-4804-aa5c-78480d5bd080' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>standard_name :</span></dt><dd>latitude</dd><dt><span>long_name :</span></dt><dd>Latitude</dd><dt><span>units :</span></dt><dd>degrees_north</dd><dt><span>axis :</span></dt><dd>Y</dd></dl></div><div class='xr-var-data'><pre>array([75. , 72.5, 70. , 67.5, 65. , 62.5, 60. , 57.5, 55. , 52.5, 50. , 47.5,
       45. , 42.5, 40. , 37.5, 35. , 32.5, 30. , 27.5, 25. , 22.5, 20. , 17.5,
       15. ], dtype=float32)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>lon</span></div><div class='xr-var-dims'>(lon)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>200.0 202.5 205.0 ... 327.5 330.0</div><input id='attrs-ec68632a-b80b-47f1-aa82-1f3532ab73ba' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-ec68632a-b80b-47f1-aa82-1f3532ab73ba' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-d871c5da-7394-413d-85f0-e46215b34c09' class='xr-var-data-in' type='checkbox'><label for='data-d871c5da-7394-413d-85f0-e46215b34c09' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>standard_name :</span></dt><dd>longitude</dd><dt><span>long_name :</span></dt><dd>Longitude</dd><dt><span>units :</span></dt><dd>degrees_east</dd><dt><span>axis :</span></dt><dd>X</dd></dl></div><div class='xr-var-data'><pre>array([200. , 202.5, 205. , 207.5, 210. , 212.5, 215. , 217.5, 220. , 222.5,
       225. , 227.5, 230. , 232.5, 235. , 237.5, 240. , 242.5, 245. , 247.5,
       250. , 252.5, 255. , 257.5, 260. , 262.5, 265. , 267.5, 270. , 272.5,
       275. , 277.5, 280. , 282.5, 285. , 287.5, 290. , 292.5, 295. , 297.5,
       300. , 302.5, 305. , 307.5, 310. , 312.5, 315. , 317.5, 320. , 322.5,
       325. , 327.5, 330. ], dtype=float32)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-5f08bf20-7923-45fe-ac4d-86d5b1c0f595' class='xr-section-summary-in' type='checkbox'  checked><label for='section-5f08bf20-7923-45fe-ac4d-86d5b1c0f595' class='xr-section-summary' >Data variables: <span>(1)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>air</span></div><div class='xr-var-dims'>(time, lat, lon)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>241.2 242.5 243.5 ... 296.2 295.7</div><input id='attrs-2bd58ae7-f7e1-4a55-b3de-7f63a3b5c6f0' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-2bd58ae7-f7e1-4a55-b3de-7f63a3b5c6f0' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-100ab19a-1609-4d58-b78b-f41cd3c6e1d5' class='xr-var-data-in' type='checkbox'><label for='data-100ab19a-1609-4d58-b78b-f41cd3c6e1d5' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>long_name :</span></dt><dd>4xDaily Air temperature at sigma level 995</dd><dt><span>units :</span></dt><dd>degK</dd><dt><span>precision :</span></dt><dd>2</dd><dt><span>GRIB_id :</span></dt><dd>11</dd><dt><span>GRIB_name :</span></dt><dd>TMP</dd><dt><span>var_desc :</span></dt><dd>Air temperature</dd><dt><span>dataset :</span></dt><dd>NMC Reanalysis</dd><dt><span>level_desc :</span></dt><dd>Surface</dd><dt><span>statistic :</span></dt><dd>Individual Obs</dd><dt><span>parent_stat :</span></dt><dd>Other</dd><dt><span>actual_range :</span></dt><dd>[185.16 322.1 ]</dd></dl></div><div class='xr-var-data'><pre>array([[[241.2 , 242.5 , 243.5 , ..., 232.8 , 235.5 , 238.6 ],
        [243.8 , 244.5 , 244.7 , ..., 232.8 , 235.3 , 239.3 ],
        [250.  , 249.8 , 248.89, ..., 233.2 , 236.39, 241.7 ],
        ...,
        [296.6 , 296.2 , 296.4 , ..., 295.4 , 295.1 , 294.7 ],
        [295.9 , 296.2 , 296.79, ..., 295.9 , 295.9 , 295.2 ],
        [296.29, 296.79, 297.1 , ..., 296.9 , 296.79, 296.6 ]],

       [[242.1 , 242.7 , 243.1 , ..., 232.  , 233.6 , 235.8 ],
        [243.6 , 244.1 , 244.2 , ..., 231.  , 232.5 , 235.7 ],
        [253.2 , 252.89, 252.1 , ..., 230.8 , 233.39, 238.5 ],
        ...,
        [296.4 , 295.9 , 296.2 , ..., 295.4 , 295.1 , 294.79],
        [296.2 , 296.7 , 296.79, ..., 295.6 , 295.5 , 295.1 ],
        [296.29, 297.2 , 297.4 , ..., 296.4 , 296.4 , 296.6 ]],

       [[242.3 , 242.2 , 242.3 , ..., 234.3 , 236.1 , 238.7 ],
        [244.6 , 244.39, 244.  , ..., 230.3 , 232.  , 235.7 ],
        [256.2 , 255.5 , 254.2 , ..., 231.2 , 233.2 , 238.2 ],
        ...,
...
        [294.79, 295.29, 297.49, ..., 295.49, 295.39, 294.69],
        [296.79, 297.89, 298.29, ..., 295.49, 295.49, 294.79],
        [298.19, 299.19, 298.79, ..., 296.09, 295.79, 295.79]],

       [[245.79, 244.79, 243.49, ..., 243.29, 243.99, 244.79],
        [249.89, 249.29, 248.49, ..., 241.29, 242.49, 244.29],
        [262.39, 261.79, 261.29, ..., 240.49, 243.09, 246.89],
        ...,
        [293.69, 293.89, 295.39, ..., 295.09, 294.69, 294.29],
        [296.29, 297.19, 297.59, ..., 295.29, 295.09, 294.39],
        [297.79, 298.39, 298.49, ..., 295.69, 295.49, 295.19]],

       [[245.09, 244.29, 243.29, ..., 241.69, 241.49, 241.79],
        [249.89, 249.29, 248.39, ..., 239.59, 240.29, 241.69],
        [262.99, 262.19, 261.39, ..., 239.89, 242.59, 246.29],
        ...,
        [293.79, 293.69, 295.09, ..., 295.29, 295.09, 294.69],
        [296.09, 296.89, 297.19, ..., 295.69, 295.69, 295.19],
        [297.69, 298.09, 298.09, ..., 296.49, 296.19, 295.69]]],
      shape=(2920, 25, 53))</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-28f3b622-4b05-46f1-88bf-a52a3a67fc7f' class='xr-section-summary-in' type='checkbox'  checked><label for='section-28f3b622-4b05-46f1-88bf-a52a3a67fc7f' class='xr-section-summary' >Attributes: <span>(5)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><dl class='xr-attrs'><dt><span>Conventions :</span></dt><dd>COARDS</dd><dt><span>title :</span></dt><dd>4x daily NMC reanalysis (1948)</dd><dt><span>description :</span></dt><dd>Data is from NMC initialized reanalysis
(4x/day).  These are the 0.9950 sigma level values.</dd><dt><span>platform :</span></dt><dd>Model</dd><dt><span>references :</span></dt><dd>http://www.esrl.noaa.gov/psd/data/gridded/data.ncep.reanalysis.html</dd></dl></div></li></ul></div></div>



## 3. Indexing và Selection

XArray cung cấp nhiều cách mạnh mẽ để truy cập và lọc dữ liệu đa chiều:

### **Các phương pháp Indexing**
- **`.sel()`**: Label-based selection (sử dụng coordinate values)  
- **`.isel()`**: Integer-based selection (như NumPy indexing)
- **`.where()`**: Boolean masking và conditional selection
- **`.query()`**: SQL-like querying cho complex conditions

### **Spatial và Temporal Selection** 
- **Geographic slicing**: Lấy dữ liệu theo bounding box
- **Time range selection**: Cắt dữ liệu theo khoảng thời gian
- **Nearest neighbor**: Tìm điểm gần nhất với tọa độ cho trước
- **Interpolation selection**: Nội suy tại tọa độ bất kỳ


```python
# Select theo coordinate values
hanoi_data = weather_ds.sel(lat=21.0, lon=105.8)
print(f"      • Temperature: {hanoi_data.temp.values[:3]} °C (first 3 months)")
print(f"      • Shape after selection: {hanoi_data.temp.shape}")

# Select multiple coordinates
central_vietnam = weather_ds.sel(lat=16.0)  # Entire Đà Nẵng latitude
print(f"      • Shape: {central_vietnam.temp.shape} (time x longitude)")
print(f"      • Avg temp: {central_vietnam.temp.mean().values:.1f}°C")

# Select time ranges
summer_2023 = weather_ds.sel(time=slice('2023-06', '2023-08'))
print(f"      • Time points: {len(summer_2023.time)}")
print(f"      • Summer avg temp: {summer_2023.temp.mean().values:.1f}°C")

# Multiple dimension selection
hanoi_summer = weather_ds.sel(lat=21.0, lon=105.8, time=slice('2023-06', '2023-08'))
```

          • Temperature: [25.74835708 26.77128002 26.77803877] °C (first 3 months)
          • Shape after selection: (12,)
          • Shape: (12, 3) (time x longitude)
          • Avg temp: 27.4°C
          • Time points: 3
          • Summer avg temp: 27.5°C
    


```python
# Boolean condition - nhiệt độ > 27°C
hot_weather = weather_ds.temp > 27
print(f"      • Boolean array shape: {hot_weather.shape}")
print(f"      • Hot days count: {hot_weather.sum().values} grid-days")

# .where() - giữ values thỏa điều kiện, NaN cho còn lại
hot_temps_only = weather_ds.temp.where(weather_ds.temp > 27)
print(f"      • Original data points: {weather_ds.temp.size}")
print(f"      • Hot temp data points: {(~np.isnan(hot_temps_only)).sum().values}")
print(f"      • Max hot temp: {hot_temps_only.max().values:.1f}°C")

# Multiple conditions - Hot AND high humidity
hot_humid = weather_ds.where((weather_ds.temp > 27) & (weather_ds.humidity > 75))
print(f"      • Valid data points: {(~np.isnan(hot_humid.temp)).sum().values}")
print(f"      • Avg temp in hot+humid: {hot_humid.temp.mean().values:.1f}°C")

# Geographical masking - Chỉ miền Nam (latitude < 12)
south_vietnam = weather_ds.where(weather_ds.lat < 12, drop=True)
print(f"      • Original lat points: {len(weather_ds.lat)}")
print(f"      • South Vietnam lat points: {len(south_vietnam.lat)}")
print(f"      • South avg temp: {south_vietnam.temp.mean().values:.1f}°C")
```

## 4. Computations và Statistics

XArray cung cấp powerful computing operations cho dữ liệu đa chiều:

### **Statistical Operations**
- **Aggregations**: `.mean()`, `.sum()`, `.std()`, `.max()`, `.min()`
- **Dimension-specific**: Compute along specific axes 
- **Rolling operations**: Moving windows và temporal smoothing
- **Cumulative operations**: `.cumsum()`, `.cumprod()`

### **Advanced Computations**
- **Mathematical functions**: Trigonometric, exponential, logarithmic
- **Element-wise operations**: Broadcasting và vectorized operations  
- **Custom functions**: `.apply_ufunc()` for user-defined functions
- **Coordinate arithmetic**: Operations involving coordinate values


```python
# ==========================================
# 4A. AGGREGATIONS THEO DIMENSIONS
# ==========================================

print("📊 4A. AGGREGATIONS VÀ STATISTICAL OPERATIONS:")

# Tính trung bình theo từng dimension
temp_annual_mean = weather_ds.temp.mean(dim='time')  # Trung bình năm
temp_spatial_mean = weather_ds.temp.mean(dim=['lat', 'lon'])  # Trung bình không gian
temp_overall_mean = weather_ds.temp.mean()  # Trung bình tổng thể

print(f"   📊 Temperature means:")
print(f"      • Annual mean shape: {temp_annual_mean.shape} (spatial only)")
print(f"      • Spatial mean shape: {temp_spatial_mean.shape} (time only)")  
print(f"      • Overall mean: {temp_overall_mean.values:.1f}°C")

# Multiple statistics
temp_stats = {
    'mean': weather_ds.temp.mean().values,
    'std': weather_ds.temp.std().values,
    'min': weather_ds.temp.min().values,
    'max': weather_ds.temp.max().values,
    'range': weather_ds.temp.max().values - weather_ds.temp.min().values
}

print(f"\n   📈 Temperature statistics:")
for stat, value in temp_stats.items():
    print(f"      • {stat.capitalize()}: {value:.2f}°C")

# Percentiles
temp_p25 = weather_ds.temp.quantile(0.25)
temp_p75 = weather_ds.temp.quantile(0.75)
print(f"\n   📊 Temperature quartiles:")
print(f"      • 25th percentile: {temp_p25.values:.1f}°C")
print(f"      • 75th percentile: {temp_p75.values:.1f}°C")
print(f"      • IQR: {temp_p75.values - temp_p25.values:.1f}°C")
```


```python
# ==========================================
# 4B. ROLLING OPERATIONS VÀ TEMPORAL ANALYSIS
# ==========================================

print("📈 4B. ROLLING WINDOWS VÀ TEMPORAL SMOOTHING:")

# Rolling mean - moving average 3-month
temp_rolling_3m = weather_ds.temp.rolling(time=3, center=True).mean()
print(f"   📊 3-month rolling mean:")
print(f"      • Original shape: {weather_ds.temp.shape}")
print(f"      • Rolling mean shape: {temp_rolling_3m.shape}")

# Rolling statistics
temp_rolling_max = weather_ds.temp.rolling(time=3).max()  # 3-month max
temp_rolling_std = weather_ds.temp.rolling(time=3).std()  # 3-month std

# Compare original vs smoothed for Hanoi
hanoi_original = weather_ds.sel(lat=21.0, lon=105.8).temp
hanoi_smoothed = temp_rolling_3m.sel(lat=21.0, lon=105.8)

print(f"\n   🏙️ Hà Nội temperature comparison:")
print(f"      • Original monthly variation: {hanoi_original.std().values:.2f}°C")
print(f"      • 3-month smoothed variation: {hanoi_smoothed.std().values:.2f}°C")
print(f"      • Smoothing reduces variation by: {(1 - hanoi_smoothed.std().values/hanoi_original.std().values)*100:.1f}%")

# Cumulative operations
temp_cumsum = weather_ds.temp.cumsum(dim='time')  # Cumulative sum over time
print(f"\n   📈 Cumulative temperature sum:")
print(f"      • Jan 2023: {temp_cumsum.isel(time=0).mean().values:.1f}")
print(f"      • Dec 2023: {temp_cumsum.isel(time=-1).mean().values:.1f}")

# Seasonal differences (diff operation)
temp_monthly_change = weather_ds.temp.diff(dim='time')
print(f"\n   🔄 Month-to-month temperature changes:")
print(f"      • Max monthly increase: {temp_monthly_change.max().values:.1f}°C")
print(f"      • Max monthly decrease: {temp_monthly_change.min().values:.1f}°C")
```

## 5. GroupBy Operations và Temporal Grouping

GroupBy là một trong những features mạnh nhất của XArray cho phân tích temporal:

### **Temporal Grouping**
- **Seasonal analysis**: Group theo months, seasons, years
- **Time-based statistics**: Monthly means, annual cycles  
- **Climate normals**: Long-term averages và anomalies
- **Custom time groupings**: Wet/dry seasons, custom periods

### **Advanced GroupBy**
- **Multi-level grouping**: Combine multiple grouping criteria
- **Custom grouping functions**: User-defined aggregation functions
- **GroupBy with coordinates**: Group by coordinate values
- **Resampling operations**: Temporal up/downsampling


```python
# ==========================================
# 5A. TEMPORAL GROUPBY - SEASONAL ANALYSIS
# ==========================================

print("🗂️ 5A. GROUPBY OPERATIONS CHO PHÂN TÍCH THEO MÙA:")

# Group by month
monthly_climate = weather_ds.groupby('time.month').mean()
print(f"   📅 Monthly climatology:")
print(f"      • Grouped dimensions: {dict(monthly_climate.dims)}")
print(f"      • Months: {monthly_climate.month.values}")

# Tìm tháng nóng nhất và lạnh nhất
hottest_month = monthly_climate.temp.argmax(dim='month')
coldest_month = monthly_climate.temp.argmin(dim='month')

print(f"\n   🌡️ Temperature by location:")
for i, (lat_val, lon_val) in enumerate(zip(weather_ds.lat.values, weather_ds.lon.values)):
    hot_month = int(hottest_month.sel(lat=lat_val, lon=lon_val).values) + 1  # +1 vì month index từ 0
    cold_month = int(coldest_month.sel(lat=lat_val, lon=lon_val).values) + 1
    hot_temp = float(monthly_climate.temp.sel(lat=lat_val, lon=lon_val).max().values)
    cold_temp = float(monthly_climate.temp.sel(lat=lat_val, lon=lon_val).min().values)
    
    print(f"      • Lat {lat_val}°: Hottest={hot_month:2d}月 ({hot_temp:.1f}°C), Coldest={cold_month:2d}月 ({cold_temp:.1f}°C)")

# Group by season (custom function)
def vietnam_season(month):
    """Define Vietnam seasons: Dry (Nov-Apr), Wet (May-Oct)"""
    return xr.where(month.isin([11, 12, 1, 2, 3, 4]), 'dry', 'wet')

seasonal_climate = weather_ds.groupby(vietnam_season(weather_ds['time.month'])).mean()
print(f"\n   🌦️ Seasonal analysis (Vietnam climate):")
print(f"      • Seasons: {list(seasonal_climate.vietnam_season.values)}")

dry_temp = seasonal_climate.temp.sel(vietnam_season='dry').mean().values
wet_temp = seasonal_climate.temp.sel(vietnam_season='wet').mean().values
print(f"      • Dry season avg temp: {dry_temp:.1f}°C")
print(f"      • Wet season avg temp: {wet_temp:.1f}°C")
print(f"      • Seasonal difference: {wet_temp - dry_temp:.1f}°C")
```

## 6. Advanced Operations với apply_ufunc

XArray cung cấp các công cụ mạnh mẽ để áp dụng functions tùy chỉnh:

### **apply_ufunc - Universal Functions**
- **Custom functions**: Áp dụng functions bất kỳ lên XArray data
- **NumPy integration**: Sử dụng NumPy functions với XArray
- **Broadcasting control**: Kiểm soát cách function được broadcast
- **Dask compatibility**: Parallel execution với Dask arrays

### **map_blocks - Block-wise Operations** 
- **Chunked processing**: Xử lý dữ liệu theo từng blocks
- **Memory efficiency**: Làm việc với datasets lớn
- **Custom block functions**: User-defined operations trên blocks
- **Parallel execution**: Automatic parallelization với Dask


```python
# ==========================================
# 5B1. APPLY_UFUNC VỚI CUSTOM FUNCTIONS
# ==========================================

print("🧮 5B1. SỬ DỤNG APPLY_UFUNC CHO CUSTOM FUNCTIONS:")

# Định nghĩa custom function - Heat Index calculation
def heat_index_formula(temp_c, humidity_pct):
    """Tính chỉ số Heat Index (cảm giác nhiệt độ thực tế)"""
    # Convert Celsius to Fahrenheit
    temp_f = temp_c * 9/5 + 32
    
    # Heat index formula (Rothfusz equation)
    if temp_f < 80:
        return temp_c  # Không áp dụng heat index khi mát
    
    hi_f = (0.5 * (temp_f + 61.0 + ((temp_f - 68.0) * 1.2) + (humidity_pct * 0.094)))
    
    # Convert back to Celsius
    hi_c = (hi_f - 32) * 5/9
    return hi_c

# Áp dụng function lên XArray data với apply_ufunc
heat_index = xr.apply_ufunc(
    heat_index_formula,
    weather_ds.temp, 
    weather_ds.humidity,
    input_core_dims=[[], []],  # No core dimensions
    output_dtypes=[float],     # Output data type
    dask="allowed",           # Allow dask arrays
    vectorize=True            # Vectorize the function
)

print(f"   🔥 Heat Index calculation:")
print(f"      • Input temp shape: {weather_ds.temp.shape}")
print(f"      • Input humidity shape: {weather_ds.humidity.shape}")
print(f"      • Output heat index shape: {heat_index.shape}")

# So sánh temperature thật vs cảm giác
hanoi_comparison = weather_ds.sel(lat=21.0, lon=105.8)
hanoi_heat_index = heat_index.sel(lat=21.0, lon=105.8)

print(f"\n   🏙️ Hà Nội - Temperature vs Heat Index:")
for i in range(3):  # First 3 months
    actual_temp = float(hanoi_comparison.temp.isel(time=i).values)
    feels_like = float(hanoi_heat_index.isel(time=i).values)
    humidity = float(hanoi_comparison.humidity.isel(time=i).values)
    month_name = hanoi_comparison.time.isel(time=i).dt.strftime('%B').values
    
    print(f"      • {month_name}: {actual_temp:.1f}°C → cảm giác {feels_like:.1f}°C (độ ẩm: {humidity:.0f}%)")

print(f"\n   💡 apply_ufunc cho phép sử dụng bất kỳ Python function nào với XArray!")
```


```python
# ==========================================
# 5B2. NUMPY FUNCTIONS VỚI APPLY_UFUNC
# ==========================================

print("🔢 5B2. SỬ DỤNG NUMPY FUNCTIONS VỚI APPLY_UFUNC:")

# Sử dụng NumPy functions phức tạp
def climate_comfort_index(temp, humidity, wind_speed):
    """Tính chỉ số thoải mái khí hậu cho Việt Nam"""
    # Normalized temperature (optimal around 25°C)
    temp_comfort = 1 - np.abs(temp - 25) / 15
    temp_comfort = np.clip(temp_comfort, 0, 1)
    
    # Normalized humidity (optimal around 60%)
    humidity_comfort = 1 - np.abs(humidity - 60) / 40
    humidity_comfort = np.clip(humidity_comfort, 0, 1)
    
    # Wind comfort (moderate wind is good)
    wind_comfort = np.minimum(wind_speed / 10, 1)  # Cap at 10 m/s
    
    # Combined comfort index
    comfort = (temp_comfort * 0.5 + humidity_comfort * 0.3 + wind_comfort * 0.2)
    return comfort * 100  # Scale to 0-100

# Apply với multiple inputs
comfort_index = xr.apply_ufunc(
    climate_comfort_index,
    weather_ds.temp,
    weather_ds.humidity, 
    weather_ds.wind_speed,
    input_core_dims=[[], [], []],
    output_dtypes=[float],
    dask="allowed",
    vectorize=True
)

# Thêm attributes
comfort_index.attrs = {
    'long_name': 'Climate Comfort Index',
    'units': 'index (0-100)',
    'description': 'Combined climate comfort index for Vietnam (higher = more comfortable)',
    'optimal_range': '70-85'
}

print(f"   🌤️ Climate Comfort Index:")
print(f"      • Shape: {comfort_index.shape}")
print(f"      • Range: {comfort_index.min().values:.1f} - {comfort_index.max().values:.1f}")
print(f"      • Mean comfort: {comfort_index.mean().values:.1f}")

# Tìm tháng thoải mái nhất cho từng địa điểm
most_comfortable = comfort_index.argmax(dim='time')
for i, (lat_val, lon_val) in enumerate(zip(weather_ds.lat.values[:3], weather_ds.lon.values[:3])):
    best_month_idx = int(most_comfortable.sel(lat=lat_val, lon=lon_val).values)
    best_comfort = float(comfort_index.sel(lat=lat_val, lon=lon_val).isel(time=best_month_idx).values)
    best_month = weather_ds.time.isel(time=best_month_idx).dt.strftime('%B').values
    
    print(f"      • Lat {lat_val}°: Tháng thoải mái nhất = {best_month} (chỉ số: {best_comfort:.1f})")

print(f"\n   ✅ apply_ufunc cho phép sử dụng complex NumPy operations!")
```

## 5C. Map Blocks và Chunked Operations

Cho datasets lớn, XArray cung cấp map_blocks để xử lý hiệu quả:

### **Map Blocks Benefits**
- **Memory efficiency**: Xử lý từng chunk thay vì toàn bộ dataset
- **Parallel processing**: Automatic parallelization với Dask
- **Custom block functions**: Functions được áp dụng trên từng block
- **Scalability**: Làm việc với datasets GB-TB scale

### **Dask Integration**
- **Lazy evaluation**: Computations chỉ chạy khi cần
- **Automatic chunking**: XArray tự động chia data thành chunks  
- **Distributed computing**: Scale lên multiple cores/machines
- **Memory management**: Intelligent caching và cleanup


```python
# ==========================================
# 5C1. MAP_BLOCKS CHO CHUNKED PROCESSING  
# ==========================================

print("📦 5C1. SỬ DỤNG MAP_BLOCKS CHO CHUNKED OPERATIONS:")

if DASK_AVAILABLE:
    # Convert sang Dask arrays để demo chunking
    weather_dask = weather_ds.chunk({'time': 6, 'lat': 2, 'lon': 2})  # 6 months, 2x2 spatial chunks
    
    print(f"   🧩 Chunked dataset:")
    print(f"      • Original shape: {weather_ds.temp.shape}")
    print(f"      • Chunk sizes: time={6}, lat={2}, lon={2}")
    print(f"      • Number of chunks: {weather_dask.temp.data.npartitions}")
    
    # Custom function để áp dụng trên từng block
    def normalize_block(block):
        """Normalize mỗi block về range 0-1"""
        block_min = np.nanmin(block)
        block_max = np.nanmax(block)
        if block_max > block_min:
            normalized = (block - block_min) / (block_max - block_min)
        else:
            normalized = np.zeros_like(block)
        return normalized
    
    # Apply function lên từng chunk
    temp_normalized = xr.map_blocks(
        normalize_block,
        weather_dask.temp,
        template=weather_dask.temp  # Template để preserve structure
    )
    
    print(f"\n   🔄 Block-wise normalization:")
    print(f"      • Function applied to each chunk independently")
    print(f"      • Lazy evaluation: {type(temp_normalized.data)}")
    
    # Compute kết quả (trigger actual computation)
    result = temp_normalized.compute()
    print(f"      • Computed result range: {result.min().values:.3f} - {result.max().values:.3f}")
    print(f"      • Each block normalized to its own 0-1 range")
    
    # So sánh memory usage
    original_size = weather_ds.temp.nbytes / 1024 / 1024
    dask_size = weather_dask.temp.nbytes / 1024 / 1024
    
    print(f"\n   💾 Memory comparison:")
    print(f"      • Original in-memory: {original_size:.2f} MB")
    print(f"      • Dask chunked: {dask_size:.2f} MB")
    print(f"      • Chunks processed on-demand, không load toàn bộ vào memory")
    
else:
    print("   ⚠️ Dask không available - map_blocks cần Dask để hoạt động tối ưu")
    
    # Demo với NumPy alternative
    def simple_block_processing(data_array, block_size=6):
        """Simulate block processing without Dask"""
        results = []
        time_chunks = [data_array.isel(time=slice(i, i+block_size)) 
                      for i in range(0, len(data_array.time), block_size)]
        
        for i, chunk in enumerate(time_chunks):
            # Process chunk
            chunk_normalized = (chunk - chunk.min()) / (chunk.max() - chunk.min())
            results.append(chunk_normalized)
            print(f"      • Processed chunk {i+1}/{len(time_chunks)}: {chunk.time.values[0]} to {chunk.time.values[-1]}")
        
        return xr.concat(results, dim='time')
    
    # Apply manual block processing  
    temp_normalized = simple_block_processing(weather_ds.temp)
    print(f"\n   📦 Manual block processing completed:")
    print(f"      • Result shape: {temp_normalized.shape}")

print(f"\n   ✅ Map blocks cho phép xử lý datasets lớn một cách hiệu quả!")
```

## Tóm tắt

Bạn đã hoàn thành Bài 7 và học được XArray - thư viện mạnh mẽ cho multi-dimensional array processing trong Python ecosystem.

### Các khái niệm chính đã nắm vững:
- ✅ **DataArray & Dataset**: Labeled arrays với coordinates, dimensions và attributes management
- ✅ **Indexing operations**: Label-based selection với `.sel()`, `.isel()` và boolean masking
- ✅ **Multi-dimensional computations**: Aggregations, rolling operations và statistical analysis
- ✅ **Temporal grouping**: Seasonal analysis, climate statistics và custom time groupings
- ✅ **Advanced operations**: apply_ufunc cho custom functions và map_blocks cho chunked processing
- ✅ **Visualization**: Built-in plotting, faceting và contour plots cho geospatial data
- ✅ **Ecosystem integration**: Seamless compatibility với pandas, matplotlib và geospatial libraries
- ✅ **Performance optimization**: Dask integration, lazy evaluation và memory-efficient workflows

### Kỹ năng bạn có thể áp dụng:
- Xử lý và phân tích climate datasets và scientific data một cách chuyên nghiệp
- Thực hiện temporal analysis cho weather patterns và seasonal variations ở Việt Nam
- Optimize multi-dimensional data processing cho production-scale scientific applications
- Tích hợp XArray với NumPy, pandas và geospatial stack cho advanced analysis workflows
- Chuẩn bị foundation expertise cho climate modeling và environmental monitoring projects
