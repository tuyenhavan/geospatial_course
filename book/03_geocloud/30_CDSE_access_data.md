# Bài 29: Truy cập dữ liệu Copernicus Data Space Ecosystem (CDSE)

Copernicus Data Space Ecosystem (CDSE) là nền tảng cloud mới thay thế cho Copernicus Open Access Hub, cung cấp truy cập miễn phí đến toàn bộ dữ liệu Sentinel và các sản phẩm Copernicus khác.

## 29.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

- **Thiết lập xác thực** với CDSE S3 credentials
- **Tìm kiếm và truy cập** dữ liệu Sentinel qua STAC API
- **Tải dữ liệu** trực tiếp từ cloud storage với odc-stac
- **Lọc và xử lý** dữ liệu ảnh vệ tinh theo khu vực quan tâm
- **Làm việc với xarray** cho phân tích dữ liệu time-series


## 29.2. Thiết lập xác thực CDSE

**Bước 1:** Tạo tài khoản tại [Copernicus Data Space](https://dataspace.copernicus.eu/) nếu chưa có.

**Bước 2:** Đăng nhập và tạo S3 credentials từ [S3 Key Manager](https://eodata-s3keysmanager.dataspace.copernicus.eu/panel/s3-credentials).

**Bước 3:** Copy Access Key và Secret Key để sử dụng trong code.

Bạn có thể làm theo 1 trong 2 cách sau để authenticate và truy cập vào CDSE.

- `Cách 1`: Có thể thiết lập environment variables với conda:
```bash
conda env config vars set GDAL_HTTP_TCP_KEEPALIVE=YES \
  AWS_S3_ENDPOINT=eodata.dataspace.copernicus.eu \
  AWS_ACCESS_KEY_ID=your_access_key \
  AWS_SECRET_ACCESS_KEY=your_secret_key \
  AWS_HTTPS=YES \
  AWS_VIRTUAL_HOSTING=FALSE \
  GDAL_HTTP_UNSAFESSL=YES
```

- `Cách 2`: Authenticate trực tiếp trong notebook như sau.



```python
# Thay thế bằng keys thực tế của bạn
import os 
access_key = 'your_actual_access_key'  # Thay bằng Access Key từ CDSE
secret_key = 'your_actual_secret_key'  # Thay bằng Secret Key từ CDSE

# Cấu hình environment variables cho GDAL và AWS S3
os.environ["GDAL_HTTP_TCP_KEEPALIVE"] = "YES"
os.environ["AWS_S3_ENDPOINT"] = "eodata.dataspace.copernicus.eu"
os.environ["AWS_HTTPS"] = "YES"
os.environ["AWS_VIRTUAL_HOSTING"] = "FALSE"
os.environ["GDAL_HTTP_UNSAFESSL"] = "YES"

# Uncomment và sử dụng keys của bạn
# os.environ["AWS_ACCESS_KEY_ID"] = access_key
# os.environ["AWS_SECRET_ACCESS_KEY"] = secret_key
```

## 29.3. Kết nối và khám phá STAC Catalog

### 29.3.1. Kết nối STAC Catalog


```python
from pystac_client import Client
```


```python
# Connect to the Copernicus Data Space STAC catalog
catalog = Client.open("https://catalogue.dataspace.copernicus.eu/stac")
# you can see all collections in the catalog
collections = catalog.get_all_collections()
collections_ids = [col.id for col in collections]
print(collections_ids[10:15])  # print some collection ids
```

    ['cop-dem-glo-90-dged-cog', 'sentinel-3-olci-2-wrr-nrt', 'sentinel-2-l2a', 'sentinel-1-slc', 'sentinel-2-gri-l1c-gcp']
    

### 29.3.2. Tìm kiếm dữ liệu Sentinel-2


```python
# define a bounding box or area of interest
bbox = [11.439263980173, 47.81384831137465, 11.475186504136818, 47.83147903246192] # this location is in Germany
# search for items in sentinel-2-l2a collection
items = catalog.search(
    collections=["sentinel-2-l2a"],
    bbox=bbox,
    datetime="2023-05-01/2023-09-30",
    query={"eo:cloud_cover": {"lt": 20}},
).item_collection()
print(f"Found {len(items)} items")
```

    Found 31 items
    

### 29.3.3. Xem thông tin thuộc tính của bức ảnh


```python
# get the first image id as example 
# print the properties of the first item
list(items)[:1][0].properties # you can use this info to filter queries
```




    {'gsd': 10,
     'created': '2025-03-21T17:41:10.043000Z',
     'updated': '2025-04-21T00:16:33.282435Z',
     'datetime': '2023-09-28T10:17:19.024Z',
     'platform': 'sentinel-2b',
     'grid:code': 'MGRS-32UPU',
     'published': '2025-04-21T00:16:33.282435Z',
     'statistics': {'water': 1.483443,
      'nodata': 1.7e-05,
      'dark_area': 0.003301,
      'vegetation': 80.213308,
      'thin_cirrus': 0.805717,
      'cloud_shadow': 0.0,
      'unclassified': 0.000269,
      'not_vegetated': 17.480843,
      'high_proba_clouds': 0.002183,
      'medium_proba_clouds': 0.010677,
      'saturated_defective': 0.0},
     'instruments': ['msi'],
     'auth:schemes': {'s3': {'type': 's3'},
      'oidc': {'type': 'openIdConnect',
       'openIdConnectUrl': 'https://identity.dataspace.copernicus.eu/auth/realms/CDSE/.well-known/openid-configuration'}},
     'end_datetime': '2023-09-28T10:17:19.024Z',
     'product:type': 'S2MSI2A',
     'view:azimuth': 269.6725837191976,
     'constellation': 'sentinel-2',
     'eo:snow_cover': 0.000255,
     'eo:cloud_cover': 0.818577,
     'start_datetime': '2023-09-28T10:17:19.024Z',
     'sat:orbit_state': 'descending',
     'storage:schemes': {'cdse-s3': {'type': 'custom-s3',
       'title': 'Copernicus Data Space Ecosystem S3',
       'platform': 'https://eodata.dataspace.copernicus.eu',
       'description': 'This endpoint provides access to EO data which is stored on the object storage of both CloudFerro Cloud and OpenTelekom Cloud (OTC). See the [documentation](https://documentation.dataspace.copernicus.eu/APIs/S3.html) for more information, including how to get credentials.',
       'requester_pays': False},
      'creodias-s3': {'type': 'custom-s3',
       'title': 'CREODIAS S3',
       'platform': 'https://eodata.cloudferro.com',
       'description': 'Comprehensive Earth Observation Data (EODATA) archive offered by CREODIAS as a commercial part of CDSE, designed to provide users with access to a vast repository of satellite data without predefined quota limits.',
       'requester_pays': True}},
     'eopf:datatake_id': 'GS2B_20230928T101719_034268_N05.10',
     'processing:level': 'L2',
     'view:sun_azimuth': 167.450831069454,
     'eopf:datastrip_id': 'S2B_OPER_MSI_L2A_DS_S2RP_20241026T142623_S20230928T102123_N05.10',
     'processing:version': '05.10',
     'product:timeliness': 'PT24H',
     'sat:absolute_orbit': 34268,
     'sat:relative_orbit': 65,
     'view:sun_elevation': 39.0424772062148,
     'processing:datetime': '2024-10-26T14:26:23.000000Z',
     'processing:facility': 'ESA',
     'eopf:instrument_mode': 'INS-NOBS',
     'eopf:origin_datetime': '2025-03-21T17:41:10.043000Z',
     'view:incidence_angle': 4.842546212382268,
     'product:timeliness_category': 'NRT',
     'sat:platform_international_designator': '2017-013A'}



## 29.4. Tải dữ liệu với odc-stac

### 29.4.1. Đọc tất cả dữ liệu theo `AOI` và `datetime`


```python
from odc.stac import stac_load
# read the image data from the STAC items
data = stac_load(
    list(items),
    chunks={"x": 1024, "y": 1024},
    resolution=10,
    bbox=bbox, # you can also use selected band names here. here we load all bands
) # always return xarray.Dataset
data
```

### 29.4.2.  Đọc dữ liệu theo  `image id`


```python
img_id = list(items)[:1][0].id
search = catalog.search(collections=["sentinel-2-l2a"], ids=[img_id])
items = search.item_collection()
data = stac_load(
    list(items),
    chunks={"x": 1024, "y": 1024},
    resolution=10
) # always return xarray.Dataset
data
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
</style><pre class='xr-text-repr-fallback'>&lt;xarray.Dataset&gt; Size: 17GB
Dimensions:      (y: 10980, x: 10980, time: 1)
Coordinates:
  * y            (y) float64 88kB 5.4e+06 5.4e+06 5.4e+06 ... 5.29e+06 5.29e+06
  * x            (x) float64 88kB 6e+05 6e+05 6e+05 ... 7.098e+05 7.098e+05
  * time         (time) datetime64[ns] 8B 2023-09-28T10:17:19.024000
    spatial_ref  int32 4B 32632
Data variables: (12/36)
    AOT_10m      (time, y, x) float32 482MB dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;
    AOT_20m      (time, y, x) float32 482MB dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;
    AOT_60m      (time, y, x) float32 482MB dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;
    B01_20m      (time, y, x) float32 482MB dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;
    B01_60m      (time, y, x) float32 482MB dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;
    B02_10m      (time, y, x) float32 482MB dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;
    ...           ...
    TCI_10m      (time, y, x) float32 482MB dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;
    TCI_20m      (time, y, x) float32 482MB dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;
    TCI_60m      (time, y, x) float32 482MB dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;
    WVP_10m      (time, y, x) float32 482MB dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;
    WVP_20m      (time, y, x) float32 482MB dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;
    WVP_60m      (time, y, x) float32 482MB dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</pre><div class='xr-wrap' style='display:none'><div class='xr-header'><div class='xr-obj-type'>xarray.Dataset</div></div><ul class='xr-sections'><li class='xr-section-item'><input id='section-4fb2e7a4-39b4-4bcb-bb24-a1934bc63b8b' class='xr-section-summary-in' type='checkbox' disabled ><label for='section-4fb2e7a4-39b4-4bcb-bb24-a1934bc63b8b' class='xr-section-summary'  title='Expand/collapse section'>Dimensions:</label><div class='xr-section-inline-details'><ul class='xr-dim-list'><li><span class='xr-has-index'>y</span>: 10980</li><li><span class='xr-has-index'>x</span>: 10980</li><li><span class='xr-has-index'>time</span>: 1</li></ul></div><div class='xr-section-details'></div></li><li class='xr-section-item'><input id='section-6e155119-e7e7-4b52-be9a-905f6f648fad' class='xr-section-summary-in' type='checkbox'  checked><label for='section-6e155119-e7e7-4b52-be9a-905f6f648fad' class='xr-section-summary' >Coordinates: <span>(4)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>y</span></div><div class='xr-var-dims'>(y)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>5.4e+06 5.4e+06 ... 5.29e+06</div><input id='attrs-afa8d74e-548b-4d26-a203-1380f0b9e1b7' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-afa8d74e-548b-4d26-a203-1380f0b9e1b7' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-94598fbd-37f0-4d6c-85f5-80eaa05295ec' class='xr-var-data-in' type='checkbox'><label for='data-94598fbd-37f0-4d6c-85f5-80eaa05295ec' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>metre</dd><dt><span>resolution :</span></dt><dd>-10.0</dd><dt><span>crs :</span></dt><dd>EPSG:32632</dd></dl></div><div class='xr-var-data'><pre>array([5399995., 5399985., 5399975., ..., 5290225., 5290215., 5290205.],
      shape=(10980,))</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>x</span></div><div class='xr-var-dims'>(x)</div><div class='xr-var-dtype'>float64</div><div class='xr-var-preview xr-preview'>6e+05 6e+05 ... 7.098e+05 7.098e+05</div><input id='attrs-691b2275-ff64-4c9e-8c68-6d8bbd7e8cf2' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-691b2275-ff64-4c9e-8c68-6d8bbd7e8cf2' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-48289573-b87c-4cce-a92e-add961b98d72' class='xr-var-data-in' type='checkbox'><label for='data-48289573-b87c-4cce-a92e-add961b98d72' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>units :</span></dt><dd>metre</dd><dt><span>resolution :</span></dt><dd>10.0</dd><dt><span>crs :</span></dt><dd>EPSG:32632</dd></dl></div><div class='xr-var-data'><pre>array([600005., 600015., 600025., ..., 709775., 709785., 709795.],
      shape=(10980,))</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span class='xr-has-index'>time</span></div><div class='xr-var-dims'>(time)</div><div class='xr-var-dtype'>datetime64[ns]</div><div class='xr-var-preview xr-preview'>2023-09-28T10:17:19.024000</div><input id='attrs-06454135-77c5-417b-8302-1346c48e0d90' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-06454135-77c5-417b-8302-1346c48e0d90' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-f1812520-9c51-475b-be32-8f02df0474da' class='xr-var-data-in' type='checkbox'><label for='data-f1812520-9c51-475b-be32-8f02df0474da' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><pre>array([&#x27;2023-09-28T10:17:19.024000000&#x27;], dtype=&#x27;datetime64[ns]&#x27;)</pre></div></li><li class='xr-var-item'><div class='xr-var-name'><span>spatial_ref</span></div><div class='xr-var-dims'>()</div><div class='xr-var-dtype'>int32</div><div class='xr-var-preview xr-preview'>32632</div><input id='attrs-ae880f96-eb55-49b5-a5dd-46159724f3ba' class='xr-var-attrs-in' type='checkbox' ><label for='attrs-ae880f96-eb55-49b5-a5dd-46159724f3ba' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-562e298a-2851-427d-851e-a73dd6af6bb2' class='xr-var-data-in' type='checkbox'><label for='data-562e298a-2851-427d-851e-a73dd6af6bb2' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'><dt><span>spatial_ref :</span></dt><dd>PROJCRS[&quot;WGS 84 / UTM zone 32N&quot;,BASEGEOGCRS[&quot;WGS 84&quot;,ENSEMBLE[&quot;World Geodetic System 1984 ensemble&quot;,MEMBER[&quot;World Geodetic System 1984 (Transit)&quot;],MEMBER[&quot;World Geodetic System 1984 (G730)&quot;],MEMBER[&quot;World Geodetic System 1984 (G873)&quot;],MEMBER[&quot;World Geodetic System 1984 (G1150)&quot;],MEMBER[&quot;World Geodetic System 1984 (G1674)&quot;],MEMBER[&quot;World Geodetic System 1984 (G1762)&quot;],MEMBER[&quot;World Geodetic System 1984 (G2139)&quot;],MEMBER[&quot;World Geodetic System 1984 (G2296)&quot;],ELLIPSOID[&quot;WGS 84&quot;,6378137,298.257223563,LENGTHUNIT[&quot;metre&quot;,1]],ENSEMBLEACCURACY[2.0]],PRIMEM[&quot;Greenwich&quot;,0,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433]],ID[&quot;EPSG&quot;,4326]],CONVERSION[&quot;UTM zone 32N&quot;,METHOD[&quot;Transverse Mercator&quot;,ID[&quot;EPSG&quot;,9807]],PARAMETER[&quot;Latitude of natural origin&quot;,0,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433],ID[&quot;EPSG&quot;,8801]],PARAMETER[&quot;Longitude of natural origin&quot;,9,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433],ID[&quot;EPSG&quot;,8802]],PARAMETER[&quot;Scale factor at natural origin&quot;,0.9996,SCALEUNIT[&quot;unity&quot;,1],ID[&quot;EPSG&quot;,8805]],PARAMETER[&quot;False easting&quot;,500000,LENGTHUNIT[&quot;metre&quot;,1],ID[&quot;EPSG&quot;,8806]],PARAMETER[&quot;False northing&quot;,0,LENGTHUNIT[&quot;metre&quot;,1],ID[&quot;EPSG&quot;,8807]]],CS[Cartesian,2],AXIS[&quot;(E)&quot;,east,ORDER[1],LENGTHUNIT[&quot;metre&quot;,1]],AXIS[&quot;(N)&quot;,north,ORDER[2],LENGTHUNIT[&quot;metre&quot;,1]],USAGE[SCOPE[&quot;Navigation and medium accuracy spatial referencing.&quot;],AREA[&quot;Between 6°E and 12°E, northern hemisphere between equator and 84°N, onshore and offshore. Algeria. Austria. Cameroon. Denmark. Equatorial Guinea. France. Gabon. Germany. Italy. Libya. Liechtenstein. Monaco. Netherlands. Niger. Nigeria. Norway. Sao Tome and Principe. Svalbard. Sweden. Switzerland. Tunisia. Vatican City State.&quot;],BBOX[0,6,84,12]],ID[&quot;EPSG&quot;,32632]]</dd><dt><span>crs_wkt :</span></dt><dd>PROJCRS[&quot;WGS 84 / UTM zone 32N&quot;,BASEGEOGCRS[&quot;WGS 84&quot;,ENSEMBLE[&quot;World Geodetic System 1984 ensemble&quot;,MEMBER[&quot;World Geodetic System 1984 (Transit)&quot;],MEMBER[&quot;World Geodetic System 1984 (G730)&quot;],MEMBER[&quot;World Geodetic System 1984 (G873)&quot;],MEMBER[&quot;World Geodetic System 1984 (G1150)&quot;],MEMBER[&quot;World Geodetic System 1984 (G1674)&quot;],MEMBER[&quot;World Geodetic System 1984 (G1762)&quot;],MEMBER[&quot;World Geodetic System 1984 (G2139)&quot;],MEMBER[&quot;World Geodetic System 1984 (G2296)&quot;],ELLIPSOID[&quot;WGS 84&quot;,6378137,298.257223563,LENGTHUNIT[&quot;metre&quot;,1]],ENSEMBLEACCURACY[2.0]],PRIMEM[&quot;Greenwich&quot;,0,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433]],ID[&quot;EPSG&quot;,4326]],CONVERSION[&quot;UTM zone 32N&quot;,METHOD[&quot;Transverse Mercator&quot;,ID[&quot;EPSG&quot;,9807]],PARAMETER[&quot;Latitude of natural origin&quot;,0,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433],ID[&quot;EPSG&quot;,8801]],PARAMETER[&quot;Longitude of natural origin&quot;,9,ANGLEUNIT[&quot;degree&quot;,0.0174532925199433],ID[&quot;EPSG&quot;,8802]],PARAMETER[&quot;Scale factor at natural origin&quot;,0.9996,SCALEUNIT[&quot;unity&quot;,1],ID[&quot;EPSG&quot;,8805]],PARAMETER[&quot;False easting&quot;,500000,LENGTHUNIT[&quot;metre&quot;,1],ID[&quot;EPSG&quot;,8806]],PARAMETER[&quot;False northing&quot;,0,LENGTHUNIT[&quot;metre&quot;,1],ID[&quot;EPSG&quot;,8807]]],CS[Cartesian,2],AXIS[&quot;(E)&quot;,east,ORDER[1],LENGTHUNIT[&quot;metre&quot;,1]],AXIS[&quot;(N)&quot;,north,ORDER[2],LENGTHUNIT[&quot;metre&quot;,1]],USAGE[SCOPE[&quot;Navigation and medium accuracy spatial referencing.&quot;],AREA[&quot;Between 6°E and 12°E, northern hemisphere between equator and 84°N, onshore and offshore. Algeria. Austria. Cameroon. Denmark. Equatorial Guinea. France. Gabon. Germany. Italy. Libya. Liechtenstein. Monaco. Netherlands. Niger. Nigeria. Norway. Sao Tome and Principe. Svalbard. Sweden. Switzerland. Tunisia. Vatican City State.&quot;],BBOX[0,6,84,12]],ID[&quot;EPSG&quot;,32632]]</dd><dt><span>semi_major_axis :</span></dt><dd>6378137.0</dd><dt><span>semi_minor_axis :</span></dt><dd>6356752.314245179</dd><dt><span>inverse_flattening :</span></dt><dd>298.257223563</dd><dt><span>reference_ellipsoid_name :</span></dt><dd>WGS 84</dd><dt><span>longitude_of_prime_meridian :</span></dt><dd>0.0</dd><dt><span>prime_meridian_name :</span></dt><dd>Greenwich</dd><dt><span>geographic_crs_name :</span></dt><dd>WGS 84</dd><dt><span>horizontal_datum_name :</span></dt><dd>World Geodetic System 1984 ensemble</dd><dt><span>projected_crs_name :</span></dt><dd>WGS 84 / UTM zone 32N</dd><dt><span>grid_mapping_name :</span></dt><dd>transverse_mercator</dd><dt><span>latitude_of_projection_origin :</span></dt><dd>0.0</dd><dt><span>longitude_of_central_meridian :</span></dt><dd>9.0</dd><dt><span>false_easting :</span></dt><dd>500000.0</dd><dt><span>false_northing :</span></dt><dd>0.0</dd><dt><span>scale_factor_at_central_meridian :</span></dt><dd>0.9996</dd><dt><span>GeoTransform :</span></dt><dd>600000 10 0 5400000 0 -10</dd></dl></div><div class='xr-var-data'><pre>array(32632, dtype=int32)</pre></div></li></ul></div></li><li class='xr-section-item'><input id='section-f7c01bdb-d152-4f54-a379-b8cd379c3910' class='xr-section-summary-in' type='checkbox'  ><label for='section-f7c01bdb-d152-4f54-a379-b8cd379c3910' class='xr-section-summary' >Data variables: <span>(36)</span></label><div class='xr-section-inline-details'></div><div class='xr-section-details'><ul class='xr-var-list'><li class='xr-var-item'><div class='xr-var-name'><span>AOT_10m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-e7967f0b-2ad6-4492-9175-d8f1e2d3a19c' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-e7967f0b-2ad6-4492-9175-d8f1e2d3a19c' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-204cf9ec-166d-4852-b163-64cf1618ff5c' class='xr-var-data-in' type='checkbox'><label for='data-204cf9ec-166d-4852-b163-64cf1618ff5c' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>AOT_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-2b77788f-5c48-4b19-8f90-a18a9d65cb49' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-2b77788f-5c48-4b19-8f90-a18a9d65cb49' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-10226756-1f11-44dd-b9bf-5c43209f94ab' class='xr-var-data-in' type='checkbox'><label for='data-10226756-1f11-44dd-b9bf-5c43209f94ab' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>AOT_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-09312304-c0aa-4721-832a-5b144bee2f0e' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-09312304-c0aa-4721-832a-5b144bee2f0e' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-3437bd9e-e521-4e03-8388-e38095f261fc' class='xr-var-data-in' type='checkbox'><label for='data-3437bd9e-e521-4e03-8388-e38095f261fc' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B01_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-49d28a68-7752-4833-ae23-78ffda760109' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-49d28a68-7752-4833-ae23-78ffda760109' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-46a536a7-ed6c-4319-9411-a6464ec21d85' class='xr-var-data-in' type='checkbox'><label for='data-46a536a7-ed6c-4319-9411-a6464ec21d85' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B01_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-4c53dc0e-535d-4783-ae58-0e30c74f7bd5' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-4c53dc0e-535d-4783-ae58-0e30c74f7bd5' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-419d54b5-8a82-48b4-896f-cc0d5860c5f9' class='xr-var-data-in' type='checkbox'><label for='data-419d54b5-8a82-48b4-896f-cc0d5860c5f9' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B02_10m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-af289543-df47-4a55-854a-b9a11de002c0' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-af289543-df47-4a55-854a-b9a11de002c0' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-c0c53dbc-1bf0-43d6-99f1-3f25e46d0d19' class='xr-var-data-in' type='checkbox'><label for='data-c0c53dbc-1bf0-43d6-99f1-3f25e46d0d19' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B02_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-5357dc61-a4da-4b52-9cfc-08ea499e8c5f' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-5357dc61-a4da-4b52-9cfc-08ea499e8c5f' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-f3a446a3-ac23-4390-b960-e6a7a7f5221d' class='xr-var-data-in' type='checkbox'><label for='data-f3a446a3-ac23-4390-b960-e6a7a7f5221d' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B02_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-af286e4e-d3b2-4ba6-8404-1e1018d9b8f9' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-af286e4e-d3b2-4ba6-8404-1e1018d9b8f9' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-bdba60b4-156a-4511-875c-ab648b514429' class='xr-var-data-in' type='checkbox'><label for='data-bdba60b4-156a-4511-875c-ab648b514429' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B03_10m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-ef860005-bdb5-4976-aeac-d096cc85a5ed' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-ef860005-bdb5-4976-aeac-d096cc85a5ed' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-8900b60a-65f0-472a-8de4-04908c964357' class='xr-var-data-in' type='checkbox'><label for='data-8900b60a-65f0-472a-8de4-04908c964357' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B03_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-70c30117-3a96-49fd-bb0b-0fdc715a6062' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-70c30117-3a96-49fd-bb0b-0fdc715a6062' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-902b2038-dfa7-4b67-9f3c-1258e4e16560' class='xr-var-data-in' type='checkbox'><label for='data-902b2038-dfa7-4b67-9f3c-1258e4e16560' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B03_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-86df4b18-b937-4b6a-8454-48af712de036' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-86df4b18-b937-4b6a-8454-48af712de036' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b63ea738-3ab8-4b25-a561-d90cf7681f4d' class='xr-var-data-in' type='checkbox'><label for='data-b63ea738-3ab8-4b25-a561-d90cf7681f4d' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B04_10m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-5f767e2a-0b5d-423a-920a-6a5e30901edd' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-5f767e2a-0b5d-423a-920a-6a5e30901edd' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-809d07ba-ea3e-4b44-bd0d-40d3ebf36bcf' class='xr-var-data-in' type='checkbox'><label for='data-809d07ba-ea3e-4b44-bd0d-40d3ebf36bcf' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B04_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-c5aed435-83ea-4eea-9964-238f4bd2ad44' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-c5aed435-83ea-4eea-9964-238f4bd2ad44' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-57e08e17-9969-4421-b8c6-37456e3ec84a' class='xr-var-data-in' type='checkbox'><label for='data-57e08e17-9969-4421-b8c6-37456e3ec84a' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B04_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-3af6eec5-24a4-45d4-9046-15567effa4f2' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-3af6eec5-24a4-45d4-9046-15567effa4f2' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-4eca8d43-58c5-4f3c-9e11-97dc72811c10' class='xr-var-data-in' type='checkbox'><label for='data-4eca8d43-58c5-4f3c-9e11-97dc72811c10' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B05_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-22899ab3-b45f-4ae9-9f0c-8eb17844b906' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-22899ab3-b45f-4ae9-9f0c-8eb17844b906' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-7ef18483-17da-4a4c-afdd-0cbe2172fe3e' class='xr-var-data-in' type='checkbox'><label for='data-7ef18483-17da-4a4c-afdd-0cbe2172fe3e' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B05_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-da1e4f54-67d0-4682-8d08-d884d38c935a' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-da1e4f54-67d0-4682-8d08-d884d38c935a' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b02b26b5-90a7-4296-8543-63284bb67daa' class='xr-var-data-in' type='checkbox'><label for='data-b02b26b5-90a7-4296-8543-63284bb67daa' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B06_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-140b2e82-f348-4db1-8344-6536dd3ed5b7' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-140b2e82-f348-4db1-8344-6536dd3ed5b7' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-eeb1eff6-ba5d-49df-b06a-7a2823855500' class='xr-var-data-in' type='checkbox'><label for='data-eeb1eff6-ba5d-49df-b06a-7a2823855500' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B06_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-96044df4-0178-4c13-a6f6-090feb4fd1c7' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-96044df4-0178-4c13-a6f6-090feb4fd1c7' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-de5afb1d-4618-4ead-a2cb-f639ca75719f' class='xr-var-data-in' type='checkbox'><label for='data-de5afb1d-4618-4ead-a2cb-f639ca75719f' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B07_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-14120cc3-fe03-4a2b-9922-f9293415dcad' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-14120cc3-fe03-4a2b-9922-f9293415dcad' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b704ee88-e03e-4438-b545-53c18f4a618c' class='xr-var-data-in' type='checkbox'><label for='data-b704ee88-e03e-4438-b545-53c18f4a618c' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B07_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-675e82c8-d814-46da-8cba-03f1c93e72cd' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-675e82c8-d814-46da-8cba-03f1c93e72cd' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-9697cc5c-03e4-4eec-b057-fa72b58bad3a' class='xr-var-data-in' type='checkbox'><label for='data-9697cc5c-03e4-4eec-b057-fa72b58bad3a' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B08_10m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-cc1d1dbb-7576-4214-81c5-66dce1bed92a' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-cc1d1dbb-7576-4214-81c5-66dce1bed92a' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-262e6fc0-7a8b-46e2-a8c2-687fb0de258b' class='xr-var-data-in' type='checkbox'><label for='data-262e6fc0-7a8b-46e2-a8c2-687fb0de258b' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B09_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-6fa7199e-eccb-479d-b2cd-37595760ae2c' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-6fa7199e-eccb-479d-b2cd-37595760ae2c' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-979cafe8-7097-4500-8331-bb925acbd237' class='xr-var-data-in' type='checkbox'><label for='data-979cafe8-7097-4500-8331-bb925acbd237' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B11_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-5da5169b-b9c3-4ed8-a816-edb366f915a7' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-5da5169b-b9c3-4ed8-a816-edb366f915a7' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-890659fa-57f4-4fbb-a893-08b2c6bb0117' class='xr-var-data-in' type='checkbox'><label for='data-890659fa-57f4-4fbb-a893-08b2c6bb0117' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B11_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-1aa4d4d5-92d2-4562-9889-52e174a839e3' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-1aa4d4d5-92d2-4562-9889-52e174a839e3' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-27140378-b11e-44bd-b2be-9cee7660125e' class='xr-var-data-in' type='checkbox'><label for='data-27140378-b11e-44bd-b2be-9cee7660125e' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B12_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-56f369e9-afcc-40ee-a8fc-cf7805bebed8' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-56f369e9-afcc-40ee-a8fc-cf7805bebed8' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-0203995c-4ea4-4a85-9f6c-4b7c9efbea37' class='xr-var-data-in' type='checkbox'><label for='data-0203995c-4ea4-4a85-9f6c-4b7c9efbea37' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B12_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-e9dffde4-d55a-4404-bb19-0f8956faaab1' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-e9dffde4-d55a-4404-bb19-0f8956faaab1' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-f99eedd0-fafe-45a2-9c29-f75fe75ba3f4' class='xr-var-data-in' type='checkbox'><label for='data-f99eedd0-fafe-45a2-9c29-f75fe75ba3f4' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B8A_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-6fc5fc22-c740-4a51-84cc-0534dddf919c' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-6fc5fc22-c740-4a51-84cc-0534dddf919c' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b2da2b56-397d-4a94-a49c-b0c713affe93' class='xr-var-data-in' type='checkbox'><label for='data-b2da2b56-397d-4a94-a49c-b0c713affe93' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>B8A_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-48c32851-c2db-4f19-8067-2089c35b461c' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-48c32851-c2db-4f19-8067-2089c35b461c' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-10f5a783-d329-4297-83d6-8a567ee542a1' class='xr-var-data-in' type='checkbox'><label for='data-10f5a783-d329-4297-83d6-8a567ee542a1' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>SCL_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-a653e681-6c77-4f49-9eb4-2f41c8e8558c' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-a653e681-6c77-4f49-9eb4-2f41c8e8558c' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-b0697615-a217-47ec-be53-f5d239f54243' class='xr-var-data-in' type='checkbox'><label for='data-b0697615-a217-47ec-be53-f5d239f54243' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>SCL_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-056d404a-3c71-4c4f-81bc-9ae16385ab16' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-056d404a-3c71-4c4f-81bc-9ae16385ab16' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-2caaa692-6f98-46e4-8b27-55641929d9b9' class='xr-var-data-in' type='checkbox'><label for='data-2caaa692-6f98-46e4-8b27-55641929d9b9' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>TCI_10m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-a6e9ee76-2cd1-47f4-b5a1-369a0573701f' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-a6e9ee76-2cd1-47f4-b5a1-369a0573701f' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-cdae8dd2-870c-4f84-8ba6-998c6f59c258' class='xr-var-data-in' type='checkbox'><label for='data-cdae8dd2-870c-4f84-8ba6-998c6f59c258' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>TCI_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-2f4942cd-a053-478a-b03f-f53ca738ac47' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-2f4942cd-a053-478a-b03f-f53ca738ac47' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-4d861d25-6cfc-46b2-9008-b1209edea0c2' class='xr-var-data-in' type='checkbox'><label for='data-4d861d25-6cfc-46b2-9008-b1209edea0c2' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>TCI_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-25e45042-6b7f-4416-bc24-8d31a59d3384' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-25e45042-6b7f-4416-bc24-8d31a59d3384' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-ed906d17-9428-4df5-b46a-880f8ee767e1' class='xr-var-data-in' type='checkbox'><label for='data-ed906d17-9428-4df5-b46a-880f8ee767e1' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>WVP_10m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-ec90a47c-7f0b-4bf4-b89a-23f9daa73bcb' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-ec90a47c-7f0b-4bf4-b89a-23f9daa73bcb' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-e94cc4d2-4a9c-49bc-a129-2a757fb6638e' class='xr-var-data-in' type='checkbox'><label for='data-e94cc4d2-4a9c-49bc-a129-2a757fb6638e' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>WVP_20m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-4fdafbc6-d822-41a9-bcda-56d7faf106bd' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-4fdafbc6-d822-41a9-bcda-56d7faf106bd' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-ddc790ba-4cd4-49a5-b41f-3df2176976be' class='xr-var-data-in' type='checkbox'><label for='data-ddc790ba-4cd4-49a5-b41f-3df2176976be' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li><li class='xr-var-item'><div class='xr-var-name'><span>WVP_60m</span></div><div class='xr-var-dims'>(time, y, x)</div><div class='xr-var-dtype'>float32</div><div class='xr-var-preview xr-preview'>dask.array&lt;chunksize=(1, 1024, 1024), meta=np.ndarray&gt;</div><input id='attrs-5694872e-f4c2-4328-85d7-84c820269d8e' class='xr-var-attrs-in' type='checkbox' disabled><label for='attrs-5694872e-f4c2-4328-85d7-84c820269d8e' title='Show/Hide attributes'><svg class='icon xr-icon-file-text2'><use xlink:href='#icon-file-text2'></use></svg></label><input id='data-5368a5d6-7af9-4b6c-994f-919b8ead50a4' class='xr-var-data-in' type='checkbox'><label for='data-5368a5d6-7af9-4b6c-994f-919b8ead50a4' title='Show/Hide data repr'><svg class='icon xr-icon-database'><use xlink:href='#icon-database'></use></svg></label><div class='xr-var-attrs'><dl class='xr-attrs'></dl></div><div class='xr-var-data'><table>
    <tr>
        <td>
            <table style="border-collapse: collapse;">
                <thead>
                    <tr>
                        <td> </td>
                        <th> Array </th>
                        <th> Chunk </th>
                    </tr>
                </thead>
                <tbody>

                    <tr>
                        <th> Bytes </th>
                        <td> 459.90 MiB </td>
                        <td> 4.00 MiB </td>
                    </tr>

                    <tr>
                        <th> Shape </th>
                        <td> (1, 10980, 10980) </td>
                        <td> (1, 1024, 1024) </td>
                    </tr>
                    <tr>
                        <th> Dask graph </th>
                        <td colspan="2"> 121 chunks in 3 graph layers </td>
                    </tr>
                    <tr>
                        <th> Data type </th>
                        <td colspan="2"> float32 numpy.ndarray </td>
                    </tr>
                </tbody>
            </table>
        </td>
        <td>
        <svg width="194" height="184" style="stroke:rgb(0,0,0);stroke-width:1" >

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="10" y1="11" x2="24" y2="26" />
  <line x1="10" y1="22" x2="24" y2="37" />
  <line x1="10" y1="33" x2="24" y2="48" />
  <line x1="10" y1="44" x2="24" y2="59" />
  <line x1="10" y1="55" x2="24" y2="70" />
  <line x1="10" y1="67" x2="24" y2="82" />
  <line x1="10" y1="78" x2="24" y2="93" />
  <line x1="10" y1="89" x2="24" y2="104" />
  <line x1="10" y1="100" x2="24" y2="115" />
  <line x1="10" y1="111" x2="24" y2="126" />
  <line x1="10" y1="120" x2="24" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="10" y2="120" style="stroke-width:2" />
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 24.9485979497544,14.948597949754403 24.9485979497544,134.9485979497544 10.0,120.0" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="10" y1="0" x2="130" y2="0" style="stroke-width:2" />
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="10" y1="0" x2="24" y2="14" style="stroke-width:2" />
  <line x1="21" y1="0" x2="36" y2="14" />
  <line x1="32" y1="0" x2="47" y2="14" />
  <line x1="43" y1="0" x2="58" y2="14" />
  <line x1="54" y1="0" x2="69" y2="14" />
  <line x1="65" y1="0" x2="80" y2="14" />
  <line x1="77" y1="0" x2="92" y2="14" />
  <line x1="88" y1="0" x2="103" y2="14" />
  <line x1="99" y1="0" x2="114" y2="14" />
  <line x1="110" y1="0" x2="125" y2="14" />
  <line x1="121" y1="0" x2="136" y2="14" />
  <line x1="130" y1="0" x2="144" y2="14" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="10.0,0.0 130.0,0.0 144.9485979497544,14.948597949754403 24.9485979497544,14.948597949754403" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Horizontal lines -->
  <line x1="24" y1="14" x2="144" y2="14" style="stroke-width:2" />
  <line x1="24" y1="26" x2="144" y2="26" />
  <line x1="24" y1="37" x2="144" y2="37" />
  <line x1="24" y1="48" x2="144" y2="48" />
  <line x1="24" y1="59" x2="144" y2="59" />
  <line x1="24" y1="70" x2="144" y2="70" />
  <line x1="24" y1="82" x2="144" y2="82" />
  <line x1="24" y1="93" x2="144" y2="93" />
  <line x1="24" y1="104" x2="144" y2="104" />
  <line x1="24" y1="115" x2="144" y2="115" />
  <line x1="24" y1="126" x2="144" y2="126" />
  <line x1="24" y1="134" x2="144" y2="134" style="stroke-width:2" />

  <!-- Vertical lines -->
  <line x1="24" y1="14" x2="24" y2="134" style="stroke-width:2" />
  <line x1="36" y1="14" x2="36" y2="134" />
  <line x1="47" y1="14" x2="47" y2="134" />
  <line x1="58" y1="14" x2="58" y2="134" />
  <line x1="69" y1="14" x2="69" y2="134" />
  <line x1="80" y1="14" x2="80" y2="134" />
  <line x1="92" y1="14" x2="92" y2="134" />
  <line x1="103" y1="14" x2="103" y2="134" />
  <line x1="114" y1="14" x2="114" y2="134" />
  <line x1="125" y1="14" x2="125" y2="134" />
  <line x1="136" y1="14" x2="136" y2="134" />
  <line x1="144" y1="14" x2="144" y2="134" style="stroke-width:2" />

  <!-- Colored Rectangle -->
  <polygon points="24.9485979497544,14.948597949754403 144.9485979497544,14.948597949754403 144.9485979497544,134.9485979497544 24.9485979497544,134.9485979497544" style="fill:#ECB172A0;stroke-width:0"/>

  <!-- Text -->
  <text x="84.948598" y="154.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" >10980</text>
  <text x="164.948598" y="74.948598" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(-90,164.948598,74.948598)">10980</text>
  <text x="7.474299" y="147.474299" font-size="1.0rem" font-weight="100" text-anchor="middle" transform="rotate(45,7.474299,147.474299)">1</text>
</svg>
        </td>
    </tr>
</table></div></li></ul></div></li></ul></div></div>



## 29.7. Xử lý và trực quan hóa dữ liệu

### 29.7.1. Trích xuất và xử lý band đơn lẻ


```python
# Trích xuất band Blue (B02) với độ phân giải 10m
blue = data['B02_10m'].squeeze()  # Loại bỏ dimensions đơn lẻ
blue = blue.compute()  # Tải dữ liệu vào memory

print(f"🔵 Band Blue (B02):")
print(f"   Kích thước: {blue.shape}")
print(f"   Kiểu dữ liệu: {blue.dtype}")
print(f"   Giá trị min: {blue.min().values}")
print(f"   Giá trị max: {blue.max().values}")
print(f"   Giá trị trung bình: {blue.mean().values:.2f}")
```

### 29.7.2. Trực quan hóa dữ liệu


```python
# Tạo figure với subplots
fig, axes = plt.subplots(1, 2, figsize=(15, 6))

# Plot 1: Band Blue
im1 = axes[0].imshow(blue, cmap='Blues', vmin=0, vmax=4000)
axes[0].set_title('Sentinel-2 Band Blue (B02)', fontsize=12, fontweight='bold')
axes[0].set_xlabel('Pixel X')
axes[0].set_ylabel('Pixel Y')
plt.colorbar(im1, ax=axes[0], label='Reflectance')

# Plot 2: Histogram
blue_flat = blue.values.flatten()
blue_flat = blue_flat[~np.isnan(blue_flat)]  # Loại bỏ NaN values
axes[1].hist(blue_flat, bins=50, alpha=0.7, color='blue', edgecolor='black')
axes[1].set_title('Histogram - Band Blue Distribution', fontsize=12, fontweight='bold')
axes[1].set_xlabel('Reflectance Value')
axes[1].set_ylabel('Frequency')
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.show()

print("📊 Đã hoàn thành trực quan hóa band Blue!")
```

### 29.7.3. Xử lý nhiều band và tính chỉ số thực vật


```python
# Trích xuất các band cần thiết cho tính NDVI
red = data['B04_10m'].squeeze().compute()    # Band Red
nir = data['B08_10m'].squeeze().compute()    # Band NIR

# Tính NDVI (Normalized Difference Vegetation Index)
# NDVI = (NIR - Red) / (NIR + Red)
ndvi = (nir - red) / (nir + red)

print(f"🌱 NDVI Statistics:")
print(f"   Min: {ndvi.min().values:.3f}")
print(f"   Max: {ndvi.max().values:.3f}")
print(f"   Mean: {ndvi.mean().values:.3f}")
print(f"   Std: {ndvi.std().values:.3f}")

# Trực quan hóa NDVI
fig, ax = plt.subplots(figsize=(10, 8))
im = ax.imshow(ndvi, cmap='RdYlGn', vmin=-0.5, vmax=1.0)
ax.set_title('NDVI - Chỉ số thực vật chuẩn hóa', fontsize=14, fontweight='bold')
ax.set_xlabel('Pixel X')
ax.set_ylabel('Pixel Y')
cbar = plt.colorbar(im, ax=ax, label='NDVI Value')

# Thêm legend cho NDVI
ax.text(0.02, 0.98, 'NDVI Interpretation:\n• < 0: Water/Snow\n• 0-0.2: Bare soil\n• 0.2-0.5: Low vegetation\n• > 0.5: Dense vegetation', 
        transform=ax.transAxes, fontsize=10, verticalalignment='top',
        bbox=dict(boxstyle='round', facecolor='white', alpha=0.8))

plt.tight_layout()
plt.show()

print("🎯 Đã tính và trực quan hóa NDVI thành công!")
```

### 29.7.4. Tạo composite RGB


```python
# Trích xuất các band RGB
green = data['B03_10m'].squeeze().compute()  # Band Green
# red và blue đã có từ trước

# Chuẩn hóa giá trị về 0-1 cho hiển thị RGB
def normalize_band(band, percentile_range=(2, 98)):
    """Chuẩn hóa band sử dụng percentile stretch"""
    p_low, p_high = np.percentile(band.values[~np.isnan(band.values)], percentile_range)
    band_norm = (band - p_low) / (p_high - p_low)
    return np.clip(band_norm, 0, 1)

# Chuẩn hóa các band
red_norm = normalize_band(red)
green_norm = normalize_band(green)
blue_norm = normalize_band(blue)

# Tạo RGB composite
rgb = np.stack([red_norm, green_norm, blue_norm], axis=-1)

# Hiển thị RGB composite
fig, axes = plt.subplots(1, 2, figsize=(16, 8))

# Plot RGB
axes[0].imshow(rgb)
axes[0].set_title('Sentinel-2 True Color (RGB)', fontsize=12, fontweight='bold')
axes[0].set_xlabel('Pixel X')
axes[0].set_ylabel('Pixel Y')

# Plot NDVI để so sánh
im2 = axes[1].imshow(ndvi, cmap='RdYlGn', vmin=-0.5, vmax=1.0)
axes[1].set_title('NDVI Comparison', fontsize=12, fontweight='bold')
axes[1].set_xlabel('Pixel X')
axes[1].set_ylabel('Pixel Y')
plt.colorbar(im2, ax=axes[1], label='NDVI Value')

plt.tight_layout()
plt.show()

print("🌈 Đã tạo và hiển thị RGB composite thành công!")
```

## 29.8. Tóm tắt

Trong bài học này, chúng ta đã học cách:

- ✅ **Thiết lập xác thực** với CDSE S3 credentials
- ✅ **Kết nối và khám phá** STAC Catalog của Copernicus Data Space
- ✅ **Tìm kiếm dữ liệu** Sentinel-2 theo khu vực và thời gian
- ✅ **Tải dữ liệu** trực tiếp từ cloud với odc-stac và chunking
- ✅ **Xử lý và trực quan hóa** các band ảnh vệ tinh
- ✅ **Tính toán chỉ số NDVI** từ dữ liệu Sentinel-2
- ✅ **Tạo RGB composite** cho hiển thị màu tự nhiên

### Ưu điểm của CDSE:
- 🚀 Truy cập nhanh chóng đến dữ liệu Copernicus
- ☁️ Không cần download, xử lý trực tiếp từ cloud
- 🔄 Cập nhật dữ liệu real-time
- 💾 Tiết kiệm băng thông và storage
- 🎯 STAC API chuẩn hóa cho interoperability

### Ứng dụng thực tế:
- 🌾 Monitoring nông nghiệp và crop health
- 🏙️ Theo dõi phát triển đô thị
- 🌊 Giám sát tài nguyên nước
- 🌳 Phân tích biến đổi rừng
- 🌡️ Nghiên cứu biến đổi khí hậu


### 29.5.2. Tìm kiếm ảnh Sentinel-2 Level-2A


```python
# Tìm kiếm ảnh trong collection sentinel-2-l2a
items = catalog.search(
    collections=["sentinel-2-l2a"],
    bbox=bbox,
    datetime="2023-05-01/2023-09-30",  # Thời gian từ tháng 5 đến tháng 9/2023
    query={"eo:cloud_cover": {"lt": 20}},  # Độ phủ mây < 20%
).item_collection()

print(f"🔍 Đã tìm kiếm xong với các tiêu chí:")
print(f"   - Collection: sentinel-2-l2a")
print(f"   - Thời gian: 2023-05-01 đến 2023-09-30")
print(f"   - Độ phủ mây: < 20%")
```


```python
access_key = 'example_access_key' # Please replace with your actual access key
secret_key = 'example_secret_key' # Please replace with your actual secret key
os.environ["GDAL_HTTP_TCP_KEEPALIVE"] = "YES"
os.environ["AWS_S3_ENDPOINT"] = "eodata.dataspace.copernicus.eu"
os.environ["AWS_HTTPS"] = "YES"
os.environ["AWS_VIRTUAL_HOSTING"] = "FALSE"
os.environ["GDAL_HTTP_UNSAFESSL"] = "YES"
# os.environ["AWS_ACCESS_KEY_ID"] = access_key
# os.environ["AWS_SECRET_ACCESS_KEY"] = secret_key
```


```python
# Connect to the Copernicus Data Space STAC catalog
catalog = Client.open("https://catalogue.dataspace.copernicus.eu/stac")
# you can see all collections in the catalog
collections = catalog.get_all_collections()
collections_ids = [col.id for col in collections]
print(collections_ids)  # print all collection ids
```

    ['sentinel-3-sl-2-aod-nrt', 'sentinel-1-global-mosaics', 'sentinel-3-olci-2-wfr-nrt', 'cop-dem-glo-30-dged-cog', 'sentinel-1-slc-wv', 'sentinel-2-gri-l1c', 'sentinel-2-l1c', 'sentinel-2-global-mosaics', 'sentinel-3-sl-2-wst-nrt', 'sentinel-3-olci-1-efr-nrt', 'cop-dem-glo-90-dged-cog', 'sentinel-3-olci-2-wrr-nrt', 'sentinel-2-l2a', 'sentinel-1-slc', 'sentinel-2-gri-l1c-gcp', 'sentinel-3-sl-2-lst-ntc', 'sentinel-3-olci-1-efr-ntc', 'sentinel-3-olci-2-wfr-ntc', 'sentinel-3-olci-2-lfr-nrt', 'sentinel-3-sl-2-wst-ntc', 'sentinel-3-olci-1-err-nrt', 'sentinel-3-olci-2-lfr-ntc', 'sentinel-3-sl-2-frp-ntc', 'sentinel-3-olci-1-err-ntc', 'sentinel-3-sl-1-rbt-ntc', 'sentinel-3-olci-2-lrr-ntc', 'sentinel-3-sl-2-lst-nrt', 'sentinel-3-sl-1-rbt-nrt', 'sentinel-3-olci-2-lrr-nrt', 'sentinel-3-sl-2-frp-nrt', 'sentinel-6-p4-1b-nrt', 'sentinel-5p-l1-ra-bd2-offl', 'sentinel-3-sr-1-sra-a-nrt', 'sentinel-6-p4-1b-ntc', 'sentinel-5p-l1-ra-bd2-nrti', 'sentinel-3-sr-1-sra-a-stc', 'sentinel-6-p4-1b-stc', 'sentinel-5p-l1-ra-bd1-rpro', 'sentinel-3-sr-1-sra-a-ntc', 'sentinel-6-p4-2-nrt', 'sentinel-5p-l1-ra-bd5-nrti', 'sentinel-1-grd', 'sentinel-6-p4-2-ntc', 'sentinel-5p-l1-ra-bd6-offl', 'sentinel-6-p4-2-stc', 'ccm-dem', 'sentinel-5p-l1-ra-bd4-nrti', 'sentinel-3-olci-2-wrr-ntc', 'sentinel-3-sr-1-sra-stc', 'ccm-sar', 'sentinel-5p-l1-ra-bd1-nrti', 'sentinel-3-sr-1-sra-ntc', 'ccm-optical', 'sentinel-5p-l1-ra-bd3-nrti', 'sentinel-3-sr-1-sra-nrt', 'sentinel-5p-l1-ra-bd6-rpro', 'sentinel-3-sr-2-lan-hy-nrt', 'opengeohub-landsat-bimonthly-mosaic-v1.0.1', 'sentinel-5p-l1-ra-bd5-rpro', 'sentinel-3-sr-2-lan-hy-stc', 'sentinel-6-amr-c-nrt', 'sentinel-5p-l1-ra-bd8-rpro', 'sentinel-3-sr-2-lan-hy-ntc', 'sentinel-6-amr-c-ntc', 'sentinel-5p-l1-ra-bd3-rpro', 'sentinel-3-sr-2-lan-li-ntc', 'sentinel-6-amr-c-stc', 'sentinel-5p-l1-ra-bd7-offl', 'sentinel-3-sr-2-lan-li-stc', 'sentinel-5p-l1-ra-bd6-nrti', 'sentinel-3-sr-2-lan-li-nrt', 'sentinel-5p-l1-ra-bd5-offl', 'sentinel-3-sr-2-lan-si-stc', 'sentinel-5p-l1-ra-bd8-offl', 'sentinel-3-sr-2-lan-si-nrt', 'sentinel-5p-l1-ra-bd2-rpro', 'sentinel-3-sr-2-lan-si-ntc', 'sentinel-5p-l1-ra-bd4-offl', 'sentinel-3-sr-2-lan-nrt', 'sentinel-5p-l1-ra-bd7-rpro', 'sentinel-3-sr-2-lan-stc', 'sentinel-5p-l1-ra-bd8-nrti', 'sentinel-3-sr-2-lan-ntc', 'sentinel-5p-l1-ra-bd4-rpro', 'sentinel-3-sr-2-wat-stc', 'sentinel-5p-l1-ra-bd7-nrti', 'sentinel-3-sr-2-wat-nrt', 'sentinel-5p-l1-ra-bd3-offl', 'sentinel-3-sr-2-wat-ntc', 'sentinel-5p-l1-ra-bd1-offl', 'sentinel-3-syn-2-aod-ntc', 'sentinel-5p-l2-aer-ai-nrti', 'sentinel-3-syn-2-syn-stc', 'sentinel-5p-l2-aer-ai-offl', 'sentinel-3-syn-2-syn-ntc', 'sentinel-5p-l2-aer-ai-rpro', 'sentinel-3-syn-2-v10-ntc', 'sentinel-5p-l2-aer-lh-nrti', 'sentinel-3-syn-2-v10-stc', 'sentinel-5p-l2-aer-lh-offl', 'sentinel-3-syn-2-vg1-ntc', 'sentinel-5p-l2-aer-lh-rpro', 'sentinel-3-syn-2-vg1-stc', 'sentinel-5p-l2-ch4-offl', 'sentinel-3-syn-2-vgp-stc', 'sentinel-5p-l2-ch4-rpro', 'sentinel-3-syn-2-vgp-ntc', 'sentinel-5p-l2-cloud-nrti', 'sentinel-5p-l2-cloud-offl', 'sentinel-5p-l2-cloud-rpro', 'sentinel-5p-l2-co-rpro', 'sentinel-5p-l2-co-nrti', 'sentinel-5p-l2-co-offl', 'sentinel-5p-l2-hcho-nrti', 'sentinel-5p-l2-hcho-rpro', 'sentinel-5p-l2-hcho-offl', 'sentinel-5p-l2-no2-offl', 'sentinel-5p-l2-no2-rpro', 'sentinel-5p-l2-no2-nrti', 'sentinel-5p-l2-np-bd3-rpro', 'sentinel-5p-l2-np-bd3-offl', 'sentinel-5p-l2-np-bd6-rpro', 'sentinel-5p-l2-np-bd6-offl', 'sentinel-5p-l2-np-bd7-rpro', 'sentinel-5p-l2-np-bd7-offl', 'sentinel-5p-l2-o3-pr-nrti', 'sentinel-5p-l2-o3-pr-rpro', 'sentinel-5p-l2-o3-pr-offl', 'sentinel-5p-l2-o3-tcl-offl', 'sentinel-5p-l2-o3-tcl-nrti', 'sentinel-5p-l2-o3-tcl-rpro', 'sentinel-5p-l2-o3-rpro', 'sentinel-5p-l2-o3-nrti', 'sentinel-5p-l2-o3-offl', 'sentinel-5p-l2-so2-offl', 'sentinel-5p-l2-so2-nrti', 'sentinel-5p-l2-so2-rpro']
    


```python
# define a bounding box or area of interest
bbox = [11.439263980173, 47.81384831137465, 11.475186504136818, 47.83147903246192] # this location is in Germany
# search for items in sentinel-2-l2a collection
items = catalog.search(
    collections=["sentinel-2-l2a"],
    bbox=bbox,
    datetime="2023-05-01/2023-09-30",
    query={"eo:cloud_cover": {"lt": 20}},
).item_collection()
```


```python
# check how many items found
print(f'Number of items found: {len(items)}')
# get the first image id as example
items = list(items)[:1]
# print the properties of the first item
items[0].properties # you can use this info to filter queries
```


```python
# read the image data from the STAC items
data = stac_load(
    items,
    chunks={"x": 1024, "y": 1024},
    resolution=10,
    bbox=bbox, # you can also use selected band names here. here we load all bands
) # always return xarray.Dataset
data
```


```python
# load B02_10m band only
blue = data['B02_10m'].squeeze()
blue = blue.compute() # load data into memory
```
