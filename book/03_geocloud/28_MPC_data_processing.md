# Bài 28: Tính toán chỉ số thực vật với MPC

Trong bài  học này, chúng ta sẽ tập trung vào **xử lý dữ liệu** đã load từ MPC: tính toán các chỉ số thực vật, tổng hợp ảnh theo giai đoạn. 

> **Yêu cầu:** `pip install pystac-client planetary-computer odc-stac geopandas`

## 28.1. Mục tiêu học tập

Sau khi hoàn thành bài này, bạn có thể:

- Tìm kiếm và đọc dữ liệu Sentinel-2
- Tính toán chỉ số thực vật từ spectral band Sentinel-2
- Tổng hợp ảnh theo thời gian


```python
import pystac_client
import planetary_computer as pc
from odc.stac import stac_load
```

- **Chuẩn bị số hàm viết sẵn cho tìm kiếm và đọc dữ liệu**


```python
# Hàm này để tìm kiếm và tải dữ liệu từ Microsoft Planetary Computer (MPC) dựa trên bounding box, khoảng thời gian và mức độ che phủ mây.
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
    mpc_url = "https://planetarycomputer.microsoft.com/api/stac/v1"

    catalog = pystac_client.Client.open(
        mpc_url,
        modifier = pc.sign_inplace  # tự động ký URL asset
    )
    search = catalog.search(
        collections=[collection_id],
        bbox=bbox,
        datetime=f"{start_date}/{end_date}",
        query={"eo:cloud_cover": {"lt": max_cloud}}  # Tìm kiếm các item có độ che phủ mây < max_cloud%
    )
    items = list(search.item_collection())
    return items
# Hàm này để tải dữ liệu từ MPC dựa trên AOI, bộ sưu tập, khoảng thời gian và mức độ che phủ mây.
def load_data(bbox,
            collection_id=None,
            start_date='2020-05-01',
            end_date='2020-06-30', 
            resolution=10,
            chunks = 512,
            bands = None,
            max_cloud=20):
    """Tải dữ liệu từ MPC dựa trên AOI, bộ sưu tập, khoảng thời gian và mức độ che phủ mây.
    
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

## 28.2. Tìm kiếm và đọc dữ liệu Sentinel-2

Sentinel-2 là hệ thống vệ tinh quan sát Trái Đất phát triển và quản lý bởi ESA cung cấp ảnh đa phổ độ phân giải cao, phục vụ theo dõi môi trường, nông nghiệp và biến động bề mặt đất trên phạm vi toàn cầu. Thông tin chi tiết về số lượng kênh màu, độ phân giải, và các phiên bản khác nhau tại [link](https://developers.google.com/earth-engine/datasets/catalog/sentinel-2). Bên dưới ta xác định khu vực nghiên cứu theo `bbox` như bên dưới.


```python
# Bounding box cho vùng nghiên cứu ở Đức
bbox = [9.84375   , 47.5172007 , 10.1953125 , 47.75409798]
# Tìm kiếm các item trong catalog của Microsoft Planetary Computer
sen2data = load_data(
    bbox=bbox,
    collection_id="sentinel-2-l2a"
)
print(f"Số lượng ảnh Sentinel-2 tìm thấy cho khu vực nghiên cứu: {len(sen2data.time)}")
```

    Số lượng ảnh Sentinel-2 tìm thấy cho khu vực nghiên cứu: 3
    

## 28.3. Tính toán chỉ số thực vật từ ảnh Sentinel-2

Sentinel-2 cung cấp các dải phổ NIR và red-edge có độ phân giải cao, rất phù hợp để tính toán các chỉ số thực vật như NDVI, EVI, SAVI và theo dõi sức khỏe cây trồng theo thời gian. Trong phần này, chúng ta sẽ tính các chỉ số: NDVI, EVI, SAVI.

### 28.3.1. Chỉ số NDVI

NDVI là chỉ số thực vật được tính từ phản xạ của dải đỏ (Red, B04) và cận hồng ngoại (NIR, B08), dùng để đánh giá mức độ xanh, sức khỏe và mật độ thảm thực vật. Giá trị NDVI dao động từ -1 đến 1, trong đó giá trị càng cao cho thấy thực vật càng phát triển khỏe mạnh và có hoạt động quang hợp mạnh. NDVI được ứng dụng rộng rãi trong giám sát cây trồng, rừng và biến động thảm thực vật.


```python
# Tính NDVI cho dữ liệu đọc được
ndvi = (sen2data["B08"] - sen2data["B04"]) / (sen2data["B08"] + sen2data["B04"])
# Lấy dữ liệu NDVI cho ngày đầu tiên
first_ndvi = ndvi.isel(time=0)
print(f'Giá trị NDVI trung bình cho vùng nghiên cứu vào ngày đầu tiên: {first_ndvi.mean().compute().item():.4f}')
```

    Giá trị NDVI trung bình cho vùng nghiên cứu vào ngày đầu tiên: 0.6590
    

### 28.3.2. Tính toán chỉ số EVI

EVI là chỉ số thực vật được phát triển nhằm cải thiện khả năng theo dõi thảm thực vật trong các khu vực có mật độ cây xanh cao. EVI sử dụng thêm dải xanh lam (Blue) để giảm ảnh hưởng của khí quyển và hạn chế hiện tượng bão hòa tín hiệu thường gặp ở NDVI. Nhờ đó, EVI phản ánh chính xác hơn tình trạng sinh trưởng, sức khỏe và biến động của thảm thực vật, đặc biệt trong các khu rừng hoặc vùng có sinh khối lớn.


```python
evi = 2.5 * (sen2data["B08"] - sen2data["B04"]) / (sen2data["B08"] + 6 * sen2data["B04"] - 7.5 * sen2data["B02"] + 1)
first_evi = evi.isel(time=0)
print(f'Giá trị EVI trung bình cho vùng nghiên cứu vào ngày đầu tiên: {first_evi.mean().compute().item():.4f}')
```

### 28.3.3. Tính toán chỉ số SAVI

SAVI là chỉ số thực vật được thiết kế để giảm ảnh hưởng của nền đất trong quá trình đánh giá thảm thực vật. SAVI đặc biệt hiệu quả ở những khu vực có mật độ thực vật thưa hoặc đất trống chiếm tỷ lệ lớn, giúp phản ánh chính xác hơn tình trạng sinh trưởng và độ che phủ của cây xanh so với NDVI.


```python
savi = (sen2data['B08']-sen2data['B04'])*(1+0.5)/(sen2data['B08']+sen2data['B04']+0.5)
print(f"Giá trị SAVI trung bình cho vùng nghiên cứu cho ngày đầu tiên: {savi.isel(time=0).mean().compute().item():04f}")
```

    Giá trị SAVI trung bình cho vùng nghiên cứu cho ngày đầu tiên: 0.988296
    

## 28.4. Tổng hợp ảnh theo giai đoạn

Sau khi đọc dữ liệu, chúng ta có thể sử dụng kĩ năng từ xarray để tính toán chỉ số thực vật hoặc tổng hợp ảnh theo giai đoạn thời gian, ví dụ như tạo composite theo mùa hoặc theo năm. Dưới đây là một số ví dụ về cách thực hiện điều này.

### 28.4.1. Tổng hợp spectral bands theo tháng


```python
sen2col = load_data(
    bbox=bbox,
    start_date='2020-05-01',
    end_date='2020-10-31'
)
print(f"Số lượng ảnh khu vực nghiên cứu {len(sen2col.time)}")
print(f"Các bands trong sen2col {list(sen2col.data_vars)}.")
```

    Số lượng ảnh khu vực nghiên cứu 9
    Các bands trong sen2col ['AOT', 'B01', 'B02', 'B03', 'B04', 'B05', 'B06', 'B07', 'B08', 'B09', 'B11', 'B12', 'B8A', 'SCL', 'WVP', 'visual'].
    


```python
# Chọn bands cụ thể 
sen2col = sen2col[['B04', 'B08']]
print(f"Các bands trong sen2col {list(sen2col.data_vars)}")
# Tính median cho dữ liệu
monthly_median = sen2col.resample(time="1MS").median(dim="time")
```

    Các bands trong sen2col ['B04', 'B08']
    

### 28.4.2. Tính NDVI và tổng hợp theo tháng


```python
# Đọc dữ liệu theo AOI và khoảng thời gian
sen2col = load_data(
    bbox=bbox,
    start_date='2020-05-01',
    end_date='2020-10-31'
)
# Tính NDVI cho bộ dữ liệu đã đọc
ndvi = (sen2col['B08']-sen2col['B04'])/(sen2col['B08']+sen2col['B04'])
# Tổng hợp NDVi theo tháng
monthly_ndvi = ndvi.resample(time="1MS").median(dim="time")
print(f"Số tháng NDVI {len(monthly_ndvi.time)}")
```

    Số tháng NDVI 5
    

## Tóm tắt

Bạn đã hoàn thành Bài 28 và nắm vững cách **tính toán chỉ số thực vật và tổng hợp ảnh theo thời gian từ dữ liệu Sentinel-2 trên (MPC)**.

### Các khái niệm chính đã nắm vững:
- ✅ Viết hàm tái sử dụng **`search_items_by_bbox`** và **`load_data`** để tìm kiếm và tải dữ liệu từ MPC theo bbox, khoảng thời gian và mức độ che phủ mây
- ✅ Tải dữ liệu **Sentinel-2 L2A** dưới dạng `xarray.Dataset` với lazy loading (Dask chunks) qua `odc.stac.stac_load`
- ✅ Tính chỉ số **NDVI** (Normalized Difference Vegetation Index) từ band `B08` (NIR) và `B04` (Red)
- ✅ Tính chỉ số **EVI** (Enhanced Vegetation Index) sử dụng thêm band `B02` (Blue) để giảm ảnh hưởng khí quyển
- ✅ Tính chỉ số **SAVI** (Soil-Adjusted Vegetation Index) để giảm ảnh hưởng của nền đất trong vùng thực vật thưa
- ✅ Tổng hợp spectral bands và chỉ số thực vật theo tháng bằng **`resample(time="1MS").median()`**

### Kỹ năng bạn có thể áp dụng:
- Xây dựng pipeline tái sử dụng để tải và xử lý dữ liệu cho bất kỳ STAC collection nào trên MPC
- Tính toán các chỉ số thực vật (NDVI, EVI, SAVI) cho bất kỳ vùng nghiên cứu nào từ ảnh Sentinel-2
- Tổng hợp ảnh theo chu kỳ thời gian (tháng, quý, năm) bằng `resample()` với các hàm thống kê (median, mean, max)
- Kết hợp `xarray`, `odc.stac`, `Dask` để xử lý dữ liệu vệ tinh quy mô lớn mà không bị tràn bộ nhớ
- Áp dụng kiến thức này vào các bài toán thực tế như phân loại lớp phủ, phát hiện hạn hán, lập bản đồ ngập lụt ở các bài tiếp theo
