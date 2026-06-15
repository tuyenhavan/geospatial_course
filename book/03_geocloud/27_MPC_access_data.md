# Bài 27: Truy cập dữ liệu từ Microsoft Planetary Computer (MPC)

**Microsoft Planetary Computer (MPC)** là nền tảng lưu trữ dữ liệu địa không gian quy mô lớn của Microsoft, cung cấp hàng petabytes dữ liệu vệ tinh miễn phí thông qua giao thức **STAC** (SpatioTemporal Asset Catalog).

## 27.1. Mục tiêu học tập

Sau khi hoàn thành bài này, bạn có thể:

- Giải thích khái niệm **STAC** (Catalog → Collection → Item → Asset)
- Kết nối và liệt kê **Collections** trên MPC Catalog
- Tìm kiếm ảnh Sentinel-2 theo **bbox, thời gian, cloud cover**
- Khám phá **metadata** của STAC Items (band info, projection, properties)
- **Load dữ liệu** Sentinel-2 dưới dạng `xarray.Dataset` bằng `odc.stac`
- Truy cập dữ liệu **Landsat** và **Copernicus DEM** từ MPC

## 27.2 Giới thiệu MPC và STAC

### 27.2.1. Microsoft Planetary Computer (MPC)

MPC là nền tảng dữ liệu địa không gian của Microsoft, cung cấp:
- `Petabytes` dữ liệu vệ tinh (Sentinel-2, Landsat, MODIS, DEM, Sentinel-1,...)
- `STAC API` để tìm kiếm và truy cập dữ liệu theo chuẩn mở
- Miễn phí truy cập đọc - không cần tài khoản (chỉ cần sign URL)
- Tích hợp với `Azure Blob Storage` - data gần compute, không phí bandwidth

### 27.2.2. STAC - SpatioTemporal Asset Catalog

STAC là tiêu chuẩn mở để mô tả và tìm kiếm dữ liệu địa không gian theo không gian + thời gian:

```
Catalog
└── Collection  (nhóm dữ liệu cùng loại, ví dụ: sentinel-2-l2a)
    └── Item    (1 cảnh = 1 acquisition, có bbox + datetime + properties)
        └── Asset  (file thực tế: GeoTIFF, metadata JSON, thumbnail...)
```

| Thành phần | Ví dụ | Nội dung |
|-----------|-------|---------|
| **Catalog** | MPC STAC Catalog | Container cấp cao nhất |
| **Collection** | `sentinel-2-l2a` | Tất cả ảnh S2 trên toàn cầu |
| **Item** | `S2B_MSIL2A_20230715T...` | 1 cảnh ngày 15/7/2023, bbox, cloud% |
| **Asset** | `B04.tif`, `B08.tif`, `SCL.tif` | File GeoTIFF thực tế của từng band |

### 27.2.3. So sánh MPC với GEE

| Đặc điểm | MPC | GEE |
|----------|----------------------------------|------------------------|
| Mô hình | **Pull** - tải data về xử lý | **Push** - xử lý trên server |
| Ngôn ngữ | Python (`xarray`, `numpy`) | GEE API (Python/JS) |
| Dữ liệu | STAC → Azure Blob | GEE Image Catalog |
| Phí | Miễn phí đọc | Miễn phí (có giới hạn) |
| Khả năng custom | Rất cao (Python ecosystem) | Giới hạn theo GEE API |
| Phù hợp | Vùng nhỏ, xử lý local | Toàn cầu, no-code analysis |


```python
# pip install pystac-client planetary-computer odc-stac
import pystac_client
import planetary_computer as pc 

from odc.stac import stac_load
import xarray as xr
```


```python
# Bounding box cho vùng nghiên cứu ở Đức
bbox = [9.84375   , 47.5172007 , 10.1953125 , 47.75409798]
```

## 27.3 Kết nối MPC Catalog và khám phá dữ liệu

Kết nối tới STAC API của MPC, sau đó liệt kê các collections dữ liệu phổ biến cho remote sensing. Tham số `modifier=planetary_computer.sign_inplace` giúp tự động ký URL khi truy cập asset.


```python
# ── Kết nối MPC STAC API ────────────────────────────────────────────────────
mpc_url = "https://planetarycomputer.microsoft.com/api/stac/v1"

catalog = pystac_client.Client.open(
    mpc_url,
    modifier = pc.sign_inplace  # tự động ký URL asset
)
# Các bộ sưu tập có sẵn trên MPC
collections = [col.id for col in catalog.get_collections()]
print(f"Tổng số bộ sưu tập trên MPC: {len(collections)}")
print("Các bộ sưu tập có sẵn trên MPC:")
collections[:5]  # hiển thị 5 bộ sưu tập đầu tiên
```

    Tổng số bộ sưu tập trên MPC: 134
    Các bộ sưu tập có sẵn trên MPC:
    




    ['daymet-annual-pr',
     'daymet-daily-hi',
     '3dep-seamless',
     '3dep-lidar-dsm',
     'fia']




```python
# Kiểm tra xem có bộ sưu tập Sentinel nào không
sentinel_collections = [col for col in collections if "sentinel" in col.lower()]
print(f"Số bộ sưu tập liên quan đến Sentinel: {len(sentinel_collections)}")
print("Các bộ sưu tập Sentinel trên MPC:")
sentinel_collections[:5]  # hiển thị 5 bộ sưu tập Sentinel đầu tiên
```

    Số bộ sưu tập liên quan đến Sentinel: 16
    Các bộ sưu tập Sentinel trên MPC:
    




    ['sentinel-1-rtc',
     'sentinel-2-l2a',
     'sentinel-1-grd',
     'sentinel-5p-l2-netcdf',
     'sentinel-3-olci-wfr-l2-netcdf']



## 27.4 Tìm kiếm dữ liệu theo bộ sưu tập

Trong phần này, chúng ta sẽ viết một hàm cho phép tìm kiếm ảnh trong một bộ sưu tập theo các điều kiện sau:

- `collections` - tên collection STAC
- `bbox` - vùng quan tâm `[W, S, E, N]`
- `datetime` - khoảng thời gian `'YYYY-MM-DD/YYYY-MM-DD'`
- `query` - filter theo properties (cloud cover, platform, ...)


```python
def search_items_by_bbox(collection_id, bbox, start_date, end_date, max_cloud=20):
    """Tìm kiếm các item trong bộ sưu tập dựa trên bbox và khoảng thời gian.
    
    Args:
        collection_id (str): ID của bộ sưu tập.
        bbox (list): Danh sách các tọa độ [minx, miny, maxx, maxy].
        start_date (str): Ngày bắt đầu theo định dạng 'YYYY-MM-DD'.
        end_date (str): Ngày kết thúc theo định dạng 'YYYY-MM-DD'.
        max_cloud (int): Mức độ che phủ mây tối đa (%).

    Returns:
        list: Danh sách các item tìm được.
    """
    search = catalog.search(
        collections=[collection_id],
        bbox=bbox,
        datetime=f"{start_date}/{end_date}",
        query={"eo:cloud_cover": {"lt": max_cloud}}  # Tìm kiếm các item có độ che phủ mây < max_cloud%
    )
    items = list(search.item_collection())
    return items

def load_data(bbox,
            collection_id=None,
            start_date='2020-05-01',
            end_date='2020-05-31', 
            resolution=10,
            chunks = 512,
            bands = None,
            max_cloud=20):
    """Tải dữ liệu từ MPC dựa trên AOI, bộ sưu tập, khoảng thời gian và mức độ che phủ mây.
    Lưu ý dữ liệu sau ngày 25.01.2022 thay đổi baseline từ Sentinel-2 L1C sang L2A, nên cần phải điều chỉnh (adjust) lại dữ liệu cho phù hợp https://planetarycomputer.microsoft.com/dataset/sentinel-2-l2a#Baseline-Change.
    
    Args:
        bbox (list): Danh sách các tọa độ [minx, miny, maxx, maxy].
        collection_id (str): ID của bộ sưu tập. Nếu None, sẽ sử dụng bộ sưu tập Sentinel-2 mặc định.
        start_date (str): Ngày bắt đầu theo định dạng 'YYYY-MM-DD'.
        end_date (str): Ngày kết thúc theo định dạng 'YYYY-MM-DD'.
        max_cloud (int): Mức độ che phủ mây tối đa (%).

    Returns:
        xarray.Dataset: Dữ liệu đã tải về dưới dạng xarray Dataset.
    """
    if collection_id is None:
        collection_id = "sentinel-2-l2a"  # Sử dụng bộ sưu tập Sentinel-2 mặc định
    
    items = search_items_by_bbox(collection_id, bbox, start_date, end_date, max_cloud)
    
    if not items:
        print("Không tìm thấy item nào phù hợp với tiêu chí.")
        return None
    chunks = {"x": chunks, "y": chunks}
    # Tải dữ liệu sử dụng stac_load
    data = stac_load(
        items=items,
        patch_url=pc.sign,
        resolution=resolution,
        chunks=chunks,
        bands=bands,
        bbox=bbox,
    ).squeeze()
    
    return data
```

## 27.5. Tìm kiếm và đọc dữ liệu Sentinel-2

### 27.5.1. Tìm hiểu về thông tin thuộc tính của dữ liệu Sentinel-2


```python
# Bộ sưu tập Sentinel-2 mặc định trên MPC là "sentinel-2-l2a"
start_date = '2020-06-01'
end_date = '2020-07-30'
max_cloud = 20
items = search_items_by_bbox("sentinel-2-l2a", bbox, start_date, end_date, max_cloud)
# Kiểm tra số lượng item tìm được
print(f"Số lượng item tìm được: {len(items)}")
# kiểm tra thông tin thuộc tính của item đầu tiên
properties = items[0].properties # Ta  có thể dùng thông tin này để lọc thêm nếu cần thiết, ví dụ: "eo:cloud_cover", "datetime", v.v.
print(f"Phần trăm che phủ mây của item đầu tiên: {properties.get('eo:cloud_cover', 'Không có thông tin')} %")

```

    Số lượng item tìm được: 3
    Phần trăm che phủ mây của item đầu tiên: 8.31416 %
    

### 27.5.2. Đọc dữ liệu Sentinel-2


```python
sen2col = load_data(bbox, collection_id="sentinel-2-l2a", start_date='2020-06-01', end_date='2020-07-30', max_cloud=25)
print(f"Số lượng ảnh Sentinel-2 tìm thấy cho khu vực nghiên cứu: {len(sen2col.time)}")
```

    Số lượng ảnh Sentinel-2 tìm thấy cho khu vực nghiên cứu: 3
    

### 27.5.3. Loại bỏ mây ảnh Sentinel-2


```python
def generate_sen2cloud_mask(data, cloud_band="SCL"):
    """
    Generate a cloud mask layer from Sentinel-2 image data.
    This function creates a cloud mask layer by identifying cloud and cloud shadow pixels
    using the Scene Classification Layer (SCL) band from Sentinel-2 data.

    Args:
        data (xarray.DataArray): The input data containing the SCL band.

    Returns:
        xarray.DataArray: A binary mask where cloud and cloud shadow pixels are marked as 0, and other pixels as 1.
    """
    import numpy as np
    if isinstance(data, xr.DataArray):
        if cloud_band not in data.coords["band"].values:
            raise ValueError(f"Cloud band '{cloud_band}' not found in data")
        scl = data.sel(band=cloud_band)
        data = data.drop_sel(band=cloud_band)
        cloud_mask = xr.where(scl.isin([2, 3, 7, 8, 9]), np.nan, 1)
        data.data = data * cloud_mask
    elif isinstance(data, xr.Dataset):
        scl = data[cloud_band]
        data = data.drop_vars(cloud_band)
        cloud_mask = xr.where(scl.isin([2, 3, 7, 8, 9]), np.nan, 1)
        for var in data.data_vars:
            data[var].data = data[var] * cloud_mask
    return data
```


```python
sen2mask = generate_sen2cloud_mask(sen2col, cloud_band="SCL")
print(f"Các bands trong dữ liệu sau khi áp dụng cloud mask: {list(sen2mask.data_vars)}")
```

    Các bands trong dữ liệu sau khi áp dụng cloud mask: ['AOT', 'B01', 'B02', 'B03', 'B04', 'B05', 'B06', 'B07', 'B08', 'B09', 'B11', 'B12', 'B8A', 'WVP', 'visual']
    

## 27.6 Tìm kiếm và đọc dữ liệu Copernicus DEM 30m

Collection `cop-dem-glo-30` cung cấp **Digital Elevation Model (DEM) toàn cầu 30m** từ TanDEM-X. Không cần lọc theo thời gian — DEM là dữ liệu tĩnh.


```python
catalog = pystac_client.Client.open(
        "https://planetarycomputer.microsoft.com/api/stac/v1",
        modifier=pc.sign_inplace,
    )
search = catalog.search(
    collections=["cop-dem-glo-30"],
    bbox=bbox,
)
items = search.item_collection()
print(f"Số lượng ảnh DEM tìm thấy cho khu vực nghiên cứu: {len(items)}")
```

    Số lượng ảnh DEM tìm thấy cho khu vực nghiên cứu: 2
    


```python
# Đọc dữ liệu DEM từ item đầu tiên (nếu có)
dem = (
    (
        stac_load(
            list(items),
            patch_url=pc.sign,
            chunks={"x": 512, "y": 512},
            bbox=bbox
        )
    )
    .squeeze()
    .to_dataarray(dim="band")
    .squeeze()
)
```

## Tóm tắt

Bạn đã hoàn thành Bài 27 và nắm vững cách **truy cập và đọc dữ liệu địa không gian từ Microsoft Planetary Computer (MPC)** - nền tảng dữ liệu vệ tinh quy mô lớn sử dụng giao thức STAC.

### Các khái niệm chính đã nắm vững:
- ✅ Kiến trúc **STAC** (Catalog → Collection → Item → Asset) và cách MPC tổ chức petabytes dữ liệu vệ tinh
- ✅ Kết nối MPC STAC API bằng `pystac_client` với `modifier=pc.sign_inplace` để tự động ký URL
- ✅ Tìm kiếm dữ liệu theo **bbox, khoảng thời gian, và cloud cover** sử dụng `catalog.search()`
- ✅ Tải dữ liệu về dạng `xarray.Dataset` bằng `odc.stac.stac_load` với lazy loading (Dask chunks)
- ✅ Loại bỏ mây ảnh **Sentinel-2** sử dụng band **SCL** (Scene Classification Layer)
- ✅ Truy cập dữ liệu **Copernicus DEM 30m** (`cop-dem-glo-30`) - dữ liệu tĩnh không cần lọc thời gian

### Kỹ năng bạn có thể áp dụng:
- Khám phá và lọc các collections trên MPC (Sentinel-2, Landsat, MODIS, DEM, Sentinel-1)
- Viết hàm tìm kiếm và tải dữ liệu tái sử dụng cho bất kỳ STAC collection nào
- Xây dựng cloud mask tùy chỉnh cho Sentinel-2 dựa trên các lớp SCL (shadow, cloud low/medium/high confidence)
- Tải dữ liệu theo chunks với Dask để xử lý vùng rộng mà không bị tràn bộ nhớ
- Kết hợp MPC với `xarray`, `numpy`, `rioxarray` để xây dựng pipeline phân tích hoàn chỉnh cho các bài toán phân loại lớp phủ, phát hiện hạn hán, lập bản đồ ngập lụt ở các bài tiếp theo

