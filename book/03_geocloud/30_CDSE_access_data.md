# Bài 30: Truy cập dữ liệu Copernicus Data Space Ecosystem (CDSE)

Copernicus Data Space Ecosystem (CDSE) là nền tảng cloud mới thay thế cho Copernicus Open Access Hub, cung cấp truy cập miễn phí đến toàn bộ dữ liệu Sentinel và các sản phẩm dữ liệu khác của Copernicus.

## 30.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

- Thiết lập xác thực với CDSE S3 credentials
- Tìm kiếm và truy cập dữ liệu Sentinel qua STAC API
- Tải dữ liệu trực tiếp từ cloud storage với odc-stac
- Lọc và xử lý dữ liệu ảnh vệ tinh theo khu vực quan tâm
- Làm việc với xarray cho phân tích dữ liệu time-series


## 30.2. Thiết lập xác thực CDSE

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

- **Chọn khu vực nghiên cứu**


```python
# define a bounding box or area of interest
bbox = [11.439263980173, 47.81384831137465, 11.475186504136818, 47.83147903246192] # this location is in Germany
```

## 30.3. Kết nối và khám phá CDSE STAC Catalog

### 30.3.1. Kết nối STAC Catalog

Để kết nối với STAC Catalog của CDSE, chúng ta sẽ sử dụng thư viện `pystac_client`. Nếu chưa biết CDSE đang cung cấp những bộ dữ liệu nào, bạn có thể lấy danh sách toàn bộ collection, xuất ra file Excel để khám phá các nguồn dữ liệu hiện có trước khi thực hiện truy vấn chi tiết.


```python
from pystac_client import Client
import pandas as pd
from odc.stac import stac_load
```


```python
# Connect to the Copernicus Data Space STAC catalog
catalog = Client.open("https://catalogue.dataspace.copernicus.eu/stac")
# you can see all collections in the catalog
collections = catalog.get_all_collections()
# Tạo một bảng để lưu thông tin về các collection
dlist = []

for collection in collections:
    title = collection.title
    description = collection.description
    id = collection.id
    dlist.append({"collection_id": id, "title": title, "description": description})

df_collections = pd.DataFrame(dlist)
df_collections.head() # Các bạn có thể xuất toàn bộ bảng cho dễ nhìn.
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>collection_id</th>
      <th>title</th>
      <th>description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ccm-optical</td>
      <td>Copernicus Contributing Missions Optical</td>
      <td>Optical data from the Copernicus Contributing ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ccm-sar</td>
      <td>Copernicus Contributing Missions SAR</td>
      <td>Synthetic Aperture Radar (SAR) data from the C...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>clms_ba_global_300m_daily_v3_cog</td>
      <td>CLMS BA Global 300m daily V3 (COG)</td>
      <td>Maps burn scars, surfaces which have been suff...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>clms_ba_global_300m_daily_v3_nc</td>
      <td>CLMS BA Global 300m daily V3 (NetCDF)</td>
      <td>Maps burn scars, surfaces which have been suff...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>clms_ba_global_300m_daily_v4_cog</td>
      <td>CLMS BA Global 300m daily V4 (COG)</td>
      <td>Maps burn scars, surfaces which have been suff...</td>
    </tr>
  </tbody>
</table>
</div>



### 30.3.2. Tìm kiếm dữ liệu Sentinel-2

Để tìm kiếm dữ liệu Sentinel-2, chúng ta sử dụng phương thức `catalog.search()` với các tham số về khu vực nghiên cứu và thời gian quan tâm. Khu vực nghiên cứu cần được khai báo dưới dạng bounding box (bbox) theo hệ tọa độ địa lý WGS84 (kinh độ, vĩ độ), giúp giới hạn phạm vi và tăng hiệu quả truy vấn dữ liệu.


```python
# search for items in sentinel-2-l2a collection
items = catalog.search(
    collections=["sentinel-2-l2a"], # Đây là collection id của Sentinel-2 L2A, bạn có thể thay bằng collection id khác nếu muốn. Xem bảng df_collections để biết collection id nào có dữ liệu bạn cần.
    bbox=bbox,
    datetime="2023-05-01/2023-09-30",
    query={"eo:cloud_cover": {"lt": 20}},
).item_collection()
print(f"Found {len(items)} items")
```

### 30.3.3. Xem thông tin thuộc tính của bức ảnh

Thông tin thuộc tính cho phép người dùng có thể `query` và lọc dữ liệu theo nhu cầu như giới hạn tỷ lệ mây.


```python
# get the first image id as example 
# print the properties of the first item
list(items)[:1][0].properties # you can use this info to filter queries
```

## 30.4. Đọc dữ liệu từ CDSE với odc-stac

Sau khi có `items`, ta dùng thư viện `odc-stac` để đọc dữ liệu ảnh từ các STAC items và trả về dạng xarray.Dataset. Lưu ý, hàm sẽ tự động gộp (merge) dữ liệu nếu các ảnh có cùng thời gian quan sát.

### 30.4.1. Đọc tất cả dữ liệu theo `AOI`


```python
# read the image data from the STAC items
data = stac_load(
    list(items),
    chunks={"x": 1024, "y": 1024},
    resolution=10,
    bbox=bbox, # you can also use selected band names here. here we load all bands
) # always return xarray.Dataset
data
```

### 30.4.2.  Đọc dữ liệu theo  `image id`

Bạn cũng có thể tìm kiếm ảnh thông qua mã `id` của chúng. Cách này có thể được sử dụng trong trường hợp bạn muốn đọc một ảnh cụ thể nào đó.


```python
img_id = list(items)[:1][0].id
search = catalog.search(collections=["sentinel-2-l2a"], ids=[img_id])
items = search.item_collection()
data = stac_load(
    list(items),
    chunks={"x": 1024, "y": 1024},
    resolution=10
) # always return xarray.Dataset
```

## Tóm tắt

Trong bài học này, chúng ta đã học cách:

- ✅ **Thiết lập xác thực** với CDSE S3 credentials
- ✅ **Kết nối và khám phá** STAC Catalog của Copernicus Data Space
- ✅ **Tìm kiếm dữ liệu** Sentinel-2 theo khu vực và thời gian
- ✅ **Tải dữ liệu** trực tiếp từ cloud với odc-stac và chunking
- ✅ **Xử lý và trực quan hóa** các band ảnh vệ tinh
- ✅ **Tính toán chỉ số NDVI** từ dữ liệu Sentinel-2
- ✅ **Tạo RGB composite** cho hiển thị màu tự nhiên

