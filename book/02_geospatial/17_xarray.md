# Bài 17: Mảng N-Chiều cho Dữ liệu raster với xarray

XArray là thư viện Python mạnh mẽ cho việc xử lý dữ liệu mảng N-chiều có nhãn, đặc biệt thiết yếu cho climate science và geospatial analysis.

## 17.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

- **Hiểu cấu trúc XArray** - Dataset, DataArray và coordinate systems
- **Manipulate multi-dimensional data** từ NetCDF, Zarr và climate datasets  
- **Thực hiện indexing và selection** theo dimensions địa lý và thời gian
- **Áp dụng computations và statistics** cho dữ liệu đa chiều
- **Sử dụng GroupBy operations** cho temporal analysis và seasonal patterns
- **Tích hợp Advanced operations** với apply_ufunc và custom functions


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

## 17.2. Cấu trúc Dữ liệu XArray Cơ bản

XArray có **2 cấu trúc dữ liệu chính** cần hiểu rõ trước khi áp dụng:

**DataArray - Mảng đa chiều có nhãn**
- **Khái niệm**: Như NumPy array nhưng có coordinates và labels
- **Thành phần**: `data` + `dimensions` + `coordinates` + `attributes`
- **Ứng dụng**: Biến đơn lẻ (nhiệt độ, mưa, NDVI) trong không gian-thời gian

**Dataset - Tập hợp nhiều DataArrays**  
- **Khái niệm**: Như DataFrame của Pandas nhưng cho dữ liệu đa chiều
- **Thành phần**: Nhiều DataArrays có chung coordinates system
- **Ứng dụng**: Datasets khí hậu hoàn chỉnh (temp + rainfall + humidity + wind)

![image-2.png](image-2.png)

### 17.2.1. Tạo `DataArray` đơn giản

- **Tạo DataArray 1 chiều**


```python
# Tạo DataArray đơn giản nhất - chỉ có data
simple_data = np.array([25.5, 26.8, 24.2, 27.1, 23.9])
simple_da = xr.DataArray(simple_data)
print(f"Loại dữ liệu: {type(simple_da)}")
simple_da
```

    Loại dữ liệu: <class 'xarray.core.dataarray.DataArray'>
    




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
  padding-bottom: 4px;
}

.xr-text-repr-fallback {
  /* fallback to plain text repr when CSS is not injected (untrusted notebook) */
  display: none;
}

.xr-header {
  padding-top: 6px;
  padding-bottom: 6px;
}

.xr-header {
  border-bottom: solid 1px var(--xr-border-color);
  margin-bottom: 4px;
}

.xr-header > div,
.xr-header > ul {
  display: inline;
  margin-top: 0;
  margin-bottom: 0;
}

.xr-obj-type,
.xr-obj-name {
  margin-left: 2px;
  margin-right: 10px;
}

.xr-obj-type,
.xr-group-box-contents > label {
  color: var(--xr-font-color2);
  display: block;
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

.xr-section-item > input,
.xr-group-box-contents > input,
.xr-array-wrap > input {
  display: block;
  opacity: 0;
  height: 0;
  margin: 0;
}

.xr-section-item > input + label,
.xr-var-item > input + label {
  color: var(--xr-disabled-color);
}

.xr-section-item > input:enabled + label,
.xr-var-item > input:enabled + label,
.xr-array-wrap > input:enabled + label,
.xr-group-box-contents > input:enabled + label {
  cursor: pointer;
  color: var(--xr-font-color2);
}

.xr-section-item > input:focus-visible + label,
.xr-var-item > input:focus-visible + label,
.xr-array-wrap > input:focus-visible + label,
.xr-group-box-contents > input:focus-visible + label {
  outline: auto;
}

.xr-section-item > input:enabled + label:hover,
.xr-var-item > input:enabled + label:hover,
.xr-array-wrap > input:enabled + label:hover,
.xr-group-box-contents > input:enabled + label:hover {
  color: var(--xr-font-color0);
}

.xr-section-summary {
  grid-column: 1;
  color: var(--xr-font-color2);
  font-weight: 500;
  white-space: nowrap;
}

.xr-section-summary > em {
  font-weight: normal;
}

.xr-span-grid {
  grid-column-end: -1;
}

.xr-section-summary > span {
  display: inline-block;
  padding-left: 0.3em;
}

.xr-group-box-contents > input:checked + label > span {
  display: inline-block;
  padding-left: 0.6em;
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
.xr-section-inline-details,
.xr-group-box-contents > label {
  padding-top: 4px;
}

.xr-section-inline-details {
  grid-column: 2 / -1;
}

.xr-section-details {
  grid-column: 1 / -1;
  margin-top: 4px;
  margin-bottom: 5px;
}

.xr-section-summary-in ~ .xr-section-details {
  display: none;
}

.xr-section-summary-in:checked ~ .xr-section-details {
  display: contents;
}

.xr-children {
  display: inline-grid;
  grid-template-columns: 100%;
  grid-column: 1 / -1;
  padding-top: 4px;
}

.xr-group-box {
  display: inline-grid;
  grid-template-columns: 0px 30px auto;
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
  width: 26px;
  border-bottom: 0.2em solid;
  border-color: var(--xr-border-color);
}

.xr-group-box-contents {
  grid-column-start: 3;
  padding-bottom: 4px;
}

.xr-group-box-contents > label::before {
  content: "📂";
  padding-right: 0.3em;
}

.xr-group-box-contents > input:checked + label::before {
  content: "📁";
}

.xr-group-box-contents > input:checked + label {
  padding-bottom: 0px;
}

.xr-group-box-contents > input:checked ~ .xr-sections {
  display: none;
}

.xr-group-box-contents > input + label > span {
  display: none;
}

.xr-group-box-ellipsis {
  font-size: 1.4em;
  font-weight: 900;
  color: var(--xr-font-color2);
  letter-spacing: 0.15em;
  cursor: default;
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
</style><pre class='xr-text-repr-fallback'>&lt;xarray.DataArray (dim_0: 5)&gt; Size: 40B
array([25.5, 26.8, 24.2, 27.1, 23.9])
Dimensions without coordinates: dim_0</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.DataArray</div><div class='xr-obj-name'></div><ul class='xr-dim-list'><li><span>dim_0</span>: 5</li></ul></div><ul class='xr-sections'><li class='xr-section-item'><div class='xr-array-wrap'><input id='section-9278b821-d21e-4355-b9ac-b627d5870518' class='xr-array-in' type='checkbox' checked><label for='section-9278b821-d21e-4355-b9ac-b627d5870518' title='Show/hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-array-preview xr-preview'><span>25.5 26.8 24.2 27.1 23.9</span></div><div class='xr-array-data'><pre>array([25.5, 26.8, 24.2, 27.1, 23.9])</pre></div></div></li></ul></div></div>



- **Tạo DataArray 2 chiều**


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



- **Tạo DataArray n-chiều**


```python

```

### 17.2.2. Tạo `DataSet` đơn giản

**Dataset** là cấu trúc dữ liệu chính của XArray để làm việc với nhiều biến cùng lúc. Cách cách tạo `DataSet` phổ biển: 
`From DataArrays`: Kết hợp các DataArrays có sẵn,  
`From dictdict`: Tạo từ dictionary của arrays,  
 `From files`: Load từ NetCDF, Zarr, GRIB

- **Tạo Dataset 1 chiều**


```python

```

- **Tạo Dataset 2 chiều**


```python

```

- **Tạo Dataset n-chiều**


```python

```

## 17.3. Indexing và Selection

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

## 17.4. Computations và Statistics

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

## 17.5. GroupBy Operations và Temporal Grouping

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

## 17.6. Advanced Operations với apply_ufunc

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

## 17.7. Map Blocks và Chunked Operations

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
