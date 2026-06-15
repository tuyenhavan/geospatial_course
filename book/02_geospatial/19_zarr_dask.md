# Bài 19: Xử lý dữ liệu lớn với Zarr và Dask

Zarr và Dask tạo thành bộ công cụ mạnh mẽ cho xử lý **big data địa không gian** hiện đại - từ terabyte satellite imagery đến dữ liệu khí hậu toàn cầu.

- **Zarr**: Định dạng lưu trữ mảng nhiều chiều (array) được tối ưu cho cloud, hỗ trợ chunking và compression mạnh mẽ
- **Dask**: Framework tính toán song song (parallel computing) với lazy evaluation, có thể scale từ laptop đến HPC cluster

## 19.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

- Sử dụng thư viện `zarr-python` để tạo, đọc, ghi mảng dữ liệu
- Áp dụng Dask Array cho parallel computing với lazy evaluation
- Xây dựng custom workflows song song với `dask.delayed`
- Kết hợp Zarr + Dask để xử lý large raster stacks và tính toán NDVI hiệu quả


```python
import numpy as np
import xarray as xr
import zarr
import os
import dask.array as da
```

## 29.2. Cơ bản về Dask

Dask là thư viện Python giúp xử lý dữ liệu lớn bằng cách chia nhỏ dữ liệu và chạy song song trên nhiều CPU hoặc máy tính. Nó mở rộng NumPy và xarray, cho phép làm việc với dữ liệu lớn hơn RAM nhờ tính toán lười (lazy execution).

### 29.2.1. Tạo dask array


```python
# Tạo một mảng dữ liệu lớn với NumPy (ví dụ 10000x10000) để mô phỏng dữ liệu vệ tinh lớn. Bạn có thể thay thế bằng dữ liệu thực tế nếu có.
np_array = np.random.rand(10000, 10000)

# Chuyển mảng NumPy thành Dask array với kích thước chunk phù hợp để xử lý hiệu quả
dask_array = da.from_array(np_array, chunks=(1000, 1000)) # Hiện tại dữ liệu này vẫn còn ở dạng lazy, chưa được tính toán. Khi bạn thực hiện các phép toán trên dask_array, Dask sẽ tự động quản lý việc tính toán và sử dụng bộ nhớ một cách hiệu quả bằng cách chia nhỏ dữ liệu thành các chunk và xử lý chúng từng phần một.
print(f"Đã tạo Dask array với shape: {dask_array.shape} và chunk size: {dask_array.chunksize}")
print(f"Đây là một Dask array với dtype: {dask_array.dtype} và có {dask_array.npartitions} partitions (chunks).")
```

    Đã tạo Dask array với shape: (10000, 10000) và chunk size: (1000, 1000)
    Đây là một Dask array với dtype: float64 và có 100 partitions (chunks).
    

### 29.2.2. Tính toán trên dask


```python
# Ví dụ ta tính giá trị trung bình của toàn bộ mảng Dask array. Lệnh này sẽ không thực sự tính toán ngay lập tức, mà sẽ tạo ra một biểu thức tính toán (task graph) mà Dask sẽ thực hiện khi bạn gọi .compute().
mean_value = dask_array.mean() # Đây vẫn là một Dask array, chưa được tính toán. Khi bạn gọi .compute(), Dask sẽ thực hiện tính toán trên tất cả các chunk và trả về kết quả cuối cùng.
# Kích hoạt tính toán và lấy kết quả
result = mean_value.compute() # Lệnh này sẽ thực sự tính toán giá trị trung bình của toàn bộ mảng, và có thể mất thời gian nếu dữ liệu lớn. Dask sẽ quản lý việc sử dụng bộ nhớ và có thể sử dụng nhiều lõi CPU để tăng tốc độ tính toán.
print(f"Giá trị trung bình của Dask array là: {result:.2f}")
```

    Giá trị trung bình của Dask array là: 0.50
    


```python
# Tạo dask array (20, 100, 100) với chunk (5, 100, 100)
dask_array = da.random.random((20, 100, 100), chunks=(5, 100, 100))
# Tính trung bình theo trục 0 (tức là tính trung bình của 20 phần tử trong mỗi chunk)
vmean = dask_array.mean(axis=0).compute() # Kích hoạt tính toán và lấy kết quả
print(f"Shape của vmean: {vmean.shape}, dtype: {vmean.dtype}")
```

    Shape của vmean: (100, 100), dtype: float64
    

### 29.2.3. Tính toán song song

Mặc định, khi gọi .compute() trong Dask, các phép tính được thực thi bởi scheduler đồng bộ (synchronous scheduler) trên một luồng duy nhất, nên không tận dụng được đa lõi CPU.

Khi khởi tạo LocalCluster + Client, Dask chuyển sang distributed scheduler, cho phép chia task graph thành nhiều phần và thực thi song song trên nhiều worker, giúp tăng tốc đáng kể các phép toán trên dữ liệu lớn.


```python
import warnings 
warnings.filterwarnings("ignore") 
from dask.distributed import Client, LocalCluster
# Khởi tạo Dask client để quản lý các tác vụ tính toán phân tán. Bạn có thể điều chỉnh số lượng workers và threads tùy thuộc vào cấu hình máy tính của bạn.
cluster = LocalCluster(
    n_workers=2,
    threads_per_worker=2,
)
client = Client(cluster)
```


```python
# Tạo một mảng Dask array lớn để mô phỏng dữ liệu vệ tinh lớn hơn, ví dụ (20, 1000, 1000) với chunk (5, 1000, 1000)
x = da.random.random(
    (20, 1000, 1000),
    chunks=(5, 1000, 1000)
    ) # Giả sử đây là dữ liệu NDVI cho 20 tháng, 1000x1000 là kích thước không gian của dữ liệu vệ tinh. 
# Tính trung bình theo trục 0 (tức là tính trung bình của 20 phần tử trong mỗi chunk)
vmean = x.mean(axis=0).compute() # Kích hoạt tính toán và lấy kết quả. Giờ đây, Dask sẽ sử dụng cluster để phân phối công việc tính toán trên nhiều workers, giúp xử lý dữ liệu lớn một cách hiệu quả hơn.
print(f"Shape của vmean: {vmean.shape}, dtype: {vmean.dtype}")
```

    Shape của vmean: (1000, 1000), dtype: float64
    

## 19.3. Cơ bản về Zarr

Zarr là định dạng lưu trữ mảng nhiều chiều tương tự NumPy, nhưng hỗ trợ chia dữ liệu thành các khối (chunks) và nén hiệu quả. Nhờ đó, người dùng có thể đọc và ghi từng phần dữ liệu lớn một cách nhanh chóng mà không cần tải toàn bộ dữ liệu vào bộ nhớ.

Mảng `Zarr` có ba thuộc tính cơ bản: `shape`, `dtype`, và `thuộc tính`.

### 19.3.1. Tạo Zarr array và lưu dữ liệu 


```python
# đường dẫn đến thư mục lưu file zarr
outpath = r"G:\My Drive\python\geocourse\data\outputs"
```

- **Tạo zarr array với `zarr.zeros()`**


```python
# Tạo một mảng dữ liệu 100x100 với zarr.zeros và lưu vào file zarr
zeros = zarr.zeros(
    shape=(100, 100),
    chunks=(10, 10),
    dtype=np.float32,
    store =os.path.join(outpath, "zeros.zarr") # sẽ tạo file zeros.zarr trong thư mục outputs
)
print(zeros.info)
```

    Type               : Array
    Zarr format        : 3
    Data type          : Float32(endianness='little')
    Fill value         : 0.0
    Shape              : (100, 100)
    Chunk shape        : (10, 10)
    Order              : C
    Read-only          : False
    Store type         : LocalStore
    Filters            : ()
    Serializer         : BytesCodec(endian=<Endian.little: 'little'>)
    Compressors        : (ZstdCodec(level=0, checksum=False),)
    No. bytes          : 40000 (39.1K)
    

- **Chuyển NumPy array sang Zarr**


```python
# Tạo một mảng dữ liệu 3D (100, 200, 200) với numpy và lưu vào file zarr
data = np.random.rand(
    100,
    200,
    200
).astype("float32")

# Có thể lưu trực tiếp thành Zarr:

z = zarr.array(
    data,
    chunks=(10, 200, 200),
    store=os.path.join(outpath, "from_numpy.zarr") # sẽ tạo file from_numpy.zarr trong thư mục outputs
)
```

- **Tạo zarr array với `zarr.open()`**


```python
# Ví dụ tạo một mảng dữ liệu NDVI với 100 timestep, mỗi timestep là một ảnh NDVI 200x200 pixel sau đó lưu vào file zarr
z = zarr.open(
    os.path.join(outpath, "simulated_satellite_image.zarr"),
    mode="w",
    shape=(100, 200, 200),
    chunks=(10, 200, 200),  # one timestep per chunk
    dtype="float32",
)
# Điền dữ liệu giả vào Zarr array
for t in range(100):
    z[t, :, :] = np.random.rand(200, 200).astype("float32")  # Gán dữ liệu ngẫu nhiên cho mỗi timestep
```


```python
# Tương tự vậy ta có thể tạo một mảng dữ liệu 4D (time, band, y, x) với 100 timestep, 3 band (ví dụ RGB), mỗi band là một ảnh 200x200 pixel sau đó lưu vào file zarr
z = zarr.open(
    os.path.join(outpath, "simulated_satellite_image_rgb.zarr"),
    mode="w",
    shape=(100, 3, 200, 200),  # (time, band, y, x)
    chunks=(10, 3, 200, 200),   # 10 timestep per chunk
    dtype="float32",
)
for t in range(100):
    for b in range(3):
        z[t, b, :, :] = np.random.rand(200, 200).astype("float32")  # Gán dữ liệu ngẫu nhiên cho mỗi band
```

### 19.3.2. Đọc dữ liệu zarr

- **Đọc dữ liệu dùng `zarr.open()`**


```python
# Mở Zarr array với zarr.open để kiểm tra thông tin về shape và dtype
ds = zarr.open(os.path.join(outpath, "simulated_satellite_image.zarr"), mode="r")
print(f"Zarr array shape: {ds.shape}, dtype: {ds.dtype}")
```

    Zarr array shape: (100, 200, 200), dtype: float32
    

- **Đọc zarr array với `dask` và `xarray`**


```python
# simulated_satellite_image.zarr is a raw zarr array (not a group),
# so xr.open_zarr() won't work directly — use dask.array.from_zarr instead
zarr_path = os.path.join(outpath, "simulated_satellite_image.zarr")
dask_arr = da.from_zarr(zarr_path)
print(f"Dask array shape: {dask_arr.shape}, dtype: {dask_arr.dtype}, chunks: {dask_arr.chunks}")
```

    Dask array shape: (100, 200, 200), dtype: float32, chunks: ((10, 10, 10, 10, 10, 10, 10, 10, 10, 10), (200,), (200,))
    


```python
ds = xr.DataArray(dask_arr, dims=["time", "y", "x"], name="reflectance").to_dataset()
print(f"Đọc thành công dataset với xarray: {ds}.")
```

    Đọc thành công dataset với xarray: <xarray.Dataset> Size: 16MB
    Dimensions:      (time: 100, y: 200, x: 200)
    Dimensions without coordinates: time, y, x
    Data variables:
        reflectance  (time, y, x) float32 16MB dask.array<chunksize=(10, 200, 200), meta=np.ndarray>.
    

## 19.4. Tổ chức dữ liệu phân cấp với Zarr

Zarr Group (tổ chức dữ liệu phân cấp) là cấu trúc dạng “thư mục” cho phép tổ chức nhiều mảng dữ liệu (arrays) liên quan trong cùng một dataset. Bạn có thể tạo và truy cập từng array bên trong group như các file riêng biệt nhưng vẫn nằm trong một hệ thống thống nhất. Điều này giúp quản lý dữ liệu đa biến rõ ràng hơn, dễ mở rộng và phù hợp cho các bài toán lớn như time series ảnh vệ tinh.

### 19.4.1. Tạo Zarr Group

Trong ví dụ này, chúng ta sẽ mô phỏng việc lưu trữ chuỗi ảnh Sentinel-2 theo thời gian bằng `Zarr Group`. Giả sử có 100 ảnh Sentinel-2 của cùng một khu vực, mỗi ảnh gồm 4 band phổ: B02 (Blue), B03 (Green), B04 (Red) và B08 (NIR).

Thay vì tạo một file cho từng ngày chụp hoặc từng band, ta sẽ tạo một Zarr Group mỗi band được lưu thành một Zarr Array riêng với kích thước (time, y, x) = (100, 200, 200), trong đó 100 là số thời điểm quan sát. Cấu trúc dữ liệu được tổ chức như sau:
```bash
sentinel2.zarr
├── B02
├── B03
├── B04
└── B08
```


```python
# Tạo một Zarr group để lưu nhiều sentinel-2 bands cùng một lúc, mỗi band là một Zarr array riêng biệt bên trong group đó.
root = zarr.open_group(
    os.path.join(outpath, "sentinel2.zarr"),
    mode="w"
)
# Thêm các band vào group
for band in ["B02", "B03", "B04", "B08"]:
    arr = root.create_array(
        name=band,
        shape=(100, 200, 200),
        chunks=(10, 200, 200),
        dtype="float32",
    )

    arr[:] = np.random.rand(100, 200, 200).astype("float32")
    # thêm metadata cho mỗi band
    root[band].attrs["description"] = f"Simulated {band} band data"
    root[band].attrs["units"] = "reflectance"
```

### 19.4.2. Đọc dữ liệu theo Zarr Group

Có nhiều cách đọc dữ liệu từ `Zarr Group`, trong phần này chúng ta sẽ thử nghiệm một vài cách chính.


```python
group = zarr.open_group(r"G:\My Drive\python\geocourse\data\outputs\sentinel2.zarr", mode="r")

print(group.tree())
```

    /
    ├── B02 (100, 200, 200) float32
    ├── B03 (100, 200, 200) float32
    ├── B04 (100, 200, 200) float32
    └── B08 (100, 200, 200) float32
    
    

- **Đọc một dataset trong `group`**


```python
b2 = group['B02'][:] # Đọc toàn bộ dữ liệu của band B02 vào bộ nhớ. Nếu dữ liệu quá lớn, điều này có thể gây ra lỗi thiếu bộ nhớ (out of memory) vì kích thước dữ liệu quá lớn để xử lý trên một máy tính cá nhân. Để tránh điều này, bạn có thể đọc dữ liệu theo từng chunk hoặc sử dụng Dask để quản lý việc đọc dữ liệu một cách hiệu quả hơn.
print(f"Đã đọc band B02 với shape: {b2.shape} và dtype: {b2.dtype}.")
```

    Đã đọc band B02 với shape: (100, 200, 200) và dtype: float32.
    

- **Đọc dữ liệu với `dask`**


```python
b2 = da.from_zarr(r"G:\My Drive\python\geocourse\data\outputs\sentinel2.zarr/B02") # Đọc dữ liệu của band B02 dưới dạng Dask array. Điều này cho phép bạn xử lý dữ liệu lớn một cách hiệu quả hơn bằng cách sử dụng tính năng lazy evaluation của Dask, tránh việc tải toàn bộ dữ liệu vào bộ nhớ cùng một lúc.
print(f"Đã đọc band B02 dưới dạng Dask array với shape: {b2.shape}, dtype: {b2.dtype}, chunks: {b2.chunks}.")
```

    Đã đọc band B02 dưới dạng Dask array với shape: (100, 200, 200), dtype: float32, chunks: ((10, 10, 10, 10, 10, 10, 10, 10, 10, 10), (200,), (200,)).
    

## Tóm tắt

Bạn đã hoàn thành Bài 19 và học được Zarr và Dask - bộ công cụ nền tảng cho xử lý **big data địa không gian** hiện đại trên máy cá nhân lẫn hệ thống phân tán.

### Các khái niệm chính đã nắm vững:
- ✅ **Dask Array**: Mảng tính toán lười (lazy) mở rộng NumPy, chia dữ liệu thành chunks và chỉ tính khi gọi `.compute()`
- ✅ **LocalCluster + Client**: Khởi tạo cluster đa worker để kích hoạt tính toán song song thực sự, tận dụng đa lõi CPU
- ✅ **Zarr Array**: Định dạng lưu trữ mảng nhiều chiều tối ưu cho cloud với chunking và nén dữ liệu mạnh mẽ
- ✅ **Tạo và ghi Zarr**: Ba cách tạo Zarr array — `zarr.zeros()`, `zarr.array()` từ NumPy, và `zarr.open()` để ghi dữ liệu theo từng phần
- ✅ **Đọc Zarr với Dask và XArray**: Dùng `da.from_zarr()` để đọc dữ liệu lớn dưới dạng Dask array, sau đó bọc thành `xr.DataArray` để phân tích không gian-thời gian
- ✅ **Zarr Group**: Cấu trúc phân cấp kiểu "thư mục" để tổ chức nhiều arrays liên quan (ví dụ: nhiều band Sentinel-2) trong một dataset thống nhất

### Kỹ năng bạn có thể áp dụng:
- Lưu trữ và truy xuất hiệu quả stack ảnh vệ tinh nhiều chiều (time, band, y, x) với Zarr mà không bị giới hạn bởi RAM
- Xây dựng pipeline tính toán song song với `LocalCluster` + Dask để tăng tốc xử lý dữ liệu raster quy mô lớn
- Tổ chức dữ liệu đa biến (nhiều band, nhiều biến khí hậu) theo cấu trúc Zarr Group rõ ràng và dễ mở rộng
- Kết hợp Zarr + Dask + XArray để xây dựng các workflow phân tích chuỗi thời gian vệ tinh (NDVI, LST, v.v.) cho toàn bộ lãnh thổ
- Đặt nền tảng để chuyển sang xử lý phân tán trên cloud (AWS S3, Azure Blob) vì Zarr hỗ trợ cloud storage trực tiếp

