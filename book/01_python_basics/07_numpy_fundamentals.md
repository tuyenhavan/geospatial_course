# Bài 7: Cơ bản về NumPy

NumPy là nền tảng của tính toán số trong Python. Trong phân tích không gian địa lý, nó được sử dụng cho các phép toán mảng hiệu quả, dữ liệu raster và tính toán toán học.

## 7.1. Mục tiêu học tập
- Hiểu về NumPy arrays và ưu điểm so với lists
- Tạo, lập chỉ mục và thao tác arrays
- Thực hiện các phép toán toán học trên arrays
- Áp dụng NumPy cho các bài toán dữ liệu không gian địa lý


```python
 # Khi muốn sử dụng thư viện NumPy, ta cần import nó trước tiên.
 # Nếu không có, ta sẽ cài đặt numpy. Xem lại notebook 01_installation.ipynb để biết cách cài đặt.
import numpy as np
print('Phiên bản NumPy:', np.__version__)
```

## 7.2. Giới thiệu về NumPy

NumPy cung cấp ndarray (mảng N-chiều) cho các phép toán số nhanh chóng và hiệu quả.

### 7.2.1. Tạo array từ list


```python
# Tạo NumPy array từ một list
data = [1, 2, 3, 4, 5]
arr = np.array(data)
print('Array:', arr)
print('Kiểu dữ liệu:', type(arr))
```

### 7.2.2. Tạo arrays bằng các hàm của numpy

NumPy có thể tạo arrays từ lists, hoặc tạo arrays với các giá trị cụ thể.


```python
# Các phương pháp tạo mảng phổ biến
zeros = np.zeros((3, 3)) # Mảng 3x3 toàn số 0
ones = np.ones((2, 4)) # Mảng 2x4 toàn số 1
full = np.full((2, 2), 7) # Mảng 2x2 toàn số 7
one_like = np.ones_like(full) # Mảng cùng kích thước với arr nhưng toàn số 1
zeros_like = np.zeros_like(full) # Mảng cùng kích thước với arr nhưng toàn số 0
arange = np.arange(0, 10, 2) # Mảng từ 0 đến 10 với bước nhảy 2
arange_float = np.arange(0, 1, 0.2) # Mảng từ 0 đến 1 với bước nhảy 0.2
linspace = np.linspace(0, 1, 5) # Mảng 5 phần tử từ 0 đến 1
random_array = np.random.rand(3, 2) # Mảng ngẫu nhiên 3x2 với giá trị từ 0 đến 1
random_int_array = np.random.randint(0, 10, size=(4, 4)) # Mảng ngẫu nhiên 4x4 với giá trị nguyên từ 0 đến 10
```


```python
# Reshape dữ liệu mảng từ 2D sang 1D và ngược lại
array_2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
array_1d = array_2d.reshape(-1)  # Chuyển từ 2D sang 1D
# Reshape array sau 
a = np.arange(12).reshape(3, 4)  # Tạo mảng 2D 3x4 từ 1D
# Merge (kết hợp) hai mảng
b = np.array([[10, 11, 12, 13], [14, 15, 16, 17], [18, 19, 20, 21]])
merged_array = np.concatenate((a, b), axis=0)  # Kết hợp
# Split (tách) mảng thành hai phần
split_arrays = np.array_split(merged_array, 2, axis=0)  # Tách mảng thành 2 phần dọc theo axis 0
# sử dụng where để lọc các giá trị trong mảng
array = np.array([10, 15, 20, 25, 30, 35])
filtered_array = array[np.where(array > 20)]  # Lọc các giá trị lớn hơn 20
```


```python
# kiem tra kich thuoc, hinh dang, so chieu, va kieu du lieu cua array
print('Kích thước (shape):', filtered_array.shape)
print('Số chiều (ndim):', filtered_array.ndim)
print('Kích thước (size):', filtered_array.size)
print('Kiểu dữ liệu (dtype):', filtered_array.dtype)
```


```python
# doi kieu du lieu cua array
array = np.random.randint(-10, 10, size=(3, 3))
array
```


```python
array = array.astype(np.uint8)  # Chuyển đổi kiểu dữ liệu sang uint8 (số nguyên không âm 8-bit  )
array
```

## 7.3. Lập chỉ mục và cắt Array (indexing and slicing)

Lập chỉ mục và cắt hoạt động tương tự như lists, nhưng mạnh mẽ hơn cho dữ liệu đa chiều.


```python
arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]]) # Mảng 2D
print('Mảng 2D:\n', arr2d)
print('Phần tử tại (0,1):', arr2d[0, 1])
print('Hàng đầu tiên:', arr2d[0])
print('Cột đầu tiên:', arr2d[:, 0])
```


```python
# Thay thế giá trị trong mảng
arr2d[1, 1] = 99
print('Mảng sau khi thay thế:\n', arr2d)
# Thay thế cả một cột
arr2d[:, 2] = [7, 8, 9]
print('Mảng sau khi thay thế cột 2:\n', arr2d)
```

## 4. Các phép toán Array

NumPy hỗ trợ các phép toán theo từng phần tử nhanh chóng và broadcasting.


```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
print('Phép cộng:', a + b)
print('Phép nhân:', a * b)
print('Giá trị trung bình:', np.mean(a))
print('Tổng:', np.sum(b))
```


```python
# Ví dụ: Lưới độ cao
elevation = np.array([[100, 110, 120], [90, 95, 105], [80, 85, 90]])
print('Lưới độ cao:\n', elevation)
print('Độ cao lớn nhất:', np.max(elevation))
print('Độ cao trung bình:', np.mean(elevation))
```


```python
# Tạo ra array nhiều chiều và thao tác với nó
array = np.array([[[1, 2], [3, 4]], [[5, 6], [7, 8]]]) # Mảng 3D
mean = np.mean(array, axis=0) # Trung bình theo trục 0
std = np.std(array, axis=1)   # Độ lệch chuẩn theo trục 1
median = np.nanmedian(array, axis=2) # Trung vị theo trục 2
sum = np.nansum(array)        # Tổng bỏ qua NaN
percentile = np.percentile(array, 50) # Phần trăm vị trí 50 (trung vị)
```

## 5. Ứng dụng numpy vào địa không gian


```python
# tạo mảng 2d array mô phỏng dữ liệu bản đồ đất đai với kích thước 200x200 và các giá trị từ 1 đến 10
land_use = np.random.randint(1, 11, size=(200, 200))
# Ví dụ ta muốn nhóm các loại đất thành 3 nhóm: Nhóm 1 (1-3), Nhóm 2 (4-7), Nhóm 3 (8-10)
def reclass_landuse(value):
    """Hàm phân loại lại giá trị đất đai thành 3 nhóm."""
    if 1 <= value <= 3:
        return 1
    elif 4 <= value <= 7:
        return 2
    else:
        return 3
new_land_use = np.vectorize(reclass_landuse)(land_use)
```


```python
# tạo mảng 3d array mô phỏng dữ liệu bản đồ đất đai với kích thước 200x200 và các giá trị từ 1 đến 10 from 2000 to 2020
land_use = np.random.randint(1, 11, size=(20, 200, 200))
# Ví dụ ta muốn nhóm các loại đất thành 3 nhóm: Nhóm 1 (1-3), Nhóm 2 (4-7), Nhóm 3 (8-10)
def reclass_landuse(value):
    """Hàm phân loại lại giá trị đất đai thành 3 nhóm."""
    if 1 <= value <= 3:
        return 1
    elif 4 <= value <= 7:
        return 2
    else:
        return 3
new_land_use = np.vectorize(reclass_landuse)(land_use)
```


```python
# tạo ra mảng 3 chiều mô phỏng dữ liệu NDVI theo thời gian với kích thước 12x100x100 (12 tháng, mỗi tháng là một bản đồ 100x100)
ndvi = np.random.rand(12, 100, 100) * 2 - 1  # Giá trị NDVI từ -1 đến 1
# Tính giá trị NDVI trung bình hàng năm cho mỗi pixel
annual_ndvi = np.mean(ndvi, axis=0)
# Tính giá trí NDVI cao nhất trong năm cho mỗi pixel
max_ndvi = np.max(ndvi, axis=0)
# Xác định số tháng có NDVI > 0.5 cho mỗi pixel
ndvi_above_threshold = np.sum(ndvi > 0.5, axis=0)
```


```python
# Ham clip (giới hạn giá trị) để đảm bảo NDVI nằm trong khoảng -1 đến 1
clipped_ndvi = np.clip(ndvi, -1, 1)
```


```python
# sử dụng median filter để làm mịn dữ liệu NDVI hàng năm
from scipy.ndimage import median_filter
ndvi_filter = np.apply_along_axis(median_filter, 0, ndvi, size=3) # Áp dụng median filter với kích thước 3 theo axis 0 (thời gian).
# Tương tự vậy, ta có thể sử dụng savitzky_golay filter để lấp đầy các giá trị thiếu trong dữ liệu NDVI
from scipy.signal import savgol_filter
# Tạo một vài giá trị thiếu trong dữ liệu NDVI để minh họa
ndvi[3, 20:30, 20:30] = np.nan
ndvi_smooth = np.apply_along_axis(savgol_filter, 0, ndvi, window_length=3, polyorder=2) # Áp dụng savitzky_golay filter với cửa sổ dài 5 và bậc đa thức 2 theo axis 0 (thời gian).
# Mình cũng có thể viết 1 hàm tùy chỉnh để xử lý dữ liệu NDVI theo ý muốn
def ndvi_filter_func(a, window_size=3, poly_order=2):
    """Hàm làm mịn dữ liệu NDVI sử dụng savitzky_golay filter."""
    if np.isnan(a).all():
        return a    
    else:
        return savgol_filter(a, window_length=window_size, polyorder=poly_order)

ndvi_smooth = np.apply_along_axis(ndvi_filter_func, 0, ndvi, window_size=3, poly_order=2)
```


```python
# transpose (chuyển vị) của một mảng 3D 
data = np.random.rand(2, 3, 4) # Mảng 3D với kích thước (2, 3, 4)
transposed_data = np.transpose(data, (1, 0, 2)) # Chuyển vị mảng theo thứ tự trục mới (1, 0, 2)
print('Dữ liệu gốc:\n', data.shape)
print('Dữ liệu sau khi chuyển vị:\n', transposed_data.shape)
```

## Tóm tắt

Bạn đã hoàn thành Bài 6 và học được nền tảng NumPy - thư viện cốt lõi cho tính toán khoa học trong Python.

### Các khái niệm chính đã nắm vững:
- ✅ **NumPy arrays**: Cấu trúc dữ liệu mảng hiệu quả thay thế cho Python lists
- ✅ **Tạo arrays**: Sử dụng zeros(), ones(), arange(), linspace(), random functions
- ✅ **Indexing và slicing**: Truy cập và thao tác dữ liệu mảng đa chiều
- ✅ **Array operations**: Phép toán vectorized nhanh chóng trên toàn bộ mảng
- ✅ **Broadcasting**: Thực hiện phép toán giữa các mảng có kích thước khác nhau
- ✅ **Statistical functions**: mean(), max(), min(), sum() cho phân tích dữ liệu
- ✅ **Advanced operations**: reshape(), concatenate(), where(), apply_along_axis()
- ✅ **Ứng dụng geospatial**: Xử lý dữ liệu raster, NDVI time series, land use classification

### Kỹ năng bạn có thể áp dụng:
- Xử lý dữ liệu raster và ảnh vệ tinh một cách hiệu quả
- Thực hiện tính toán khoa học và phân tích thống kê trên datasets lớn
- Manipulate dữ liệu đa chiều như NDVI time series và climate data
- Áp dụng các bộ lọc và thuật toán xử lý tín hiệu cho dữ liệu geospatial
- Chuẩn bị nền tảng cho các thư viện advanced như pandas, scikit-learn, và rasterio
