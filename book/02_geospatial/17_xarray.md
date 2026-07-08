# Bài 17: Mảng N-chiều cho dữ liệu raster với xarray

XArray là thư viện Python mạnh mẽ cho việc xử lý dữ liệu mảng N-chiều có nhãn, đặc biệt thiết yếu cho phân tích không gian và khoa học khí hậu.

> **Lưu Ý**
> 
> Bạn có thể chạy trực tiếp notebook này bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/1rTVx0BhRIu_5-P9eU8czyaNEjzeCl7va?authuser=3) mà không cần cài đặt Python. Để tránh làm thay đổi nội dung gốc và thuận tiện cho việc lưu kết quả, hãy tạo một bản sao ( File → Save a copy in Drive ) trước khi chạy và chỉnh sửa mã nguồn trong notebook.

## 17.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

- Hiểu cấu trúc XArray - Dataset, DataArray và coordinate systems
- Thao tác trên dữ liệu nhiều chiều
- Thực hiện indexing và lựa chọn theo chiều địa lý và thời gian
- Áp dụng tính toán và thống kê cho dữ liệu đa chiều
- Sử dụng các phương thức nhóm cho dữ liệu theo thời gian 
- Tích hợp các hàm nâng cao `apply_ufunc` hoặc các hàm thủ công.


```python
import xarray as xr
import numpy as np
import pandas as pd
```

## 17.2. Cấu trúc Dữ liệu XArray Cơ bản


Trong thực tế, dữ liệu khoa học thường không chỉ bao gồm các giá trị số mà còn đi kèm với thông tin mô tả như thời gian, vị trí hoặc các chiều dữ liệu khác. Xarray được thiết kế để quản lý loại dữ liệu này một cách trực quan thông qua các chiều (dimensions) và tọa độ (coordinates).

XArray có **2 cấu trúc dữ liệu chính** là `DataArray` và `Dataset`:

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

DataArray là một trong hai kiểu dữ liệu cốt lõi của thư viện xarray. Về bản chất dữ liệu vẫn được lưu dưới dạng mảng NumPy, nhưng được gắn thêm tên các chiều (dimensions), tọa độ (coordinates) và thuộc tính (attributes), giúp dữ liệu trực quan và dễ xử lý hơn.

- **Tạo DataArray 1 chiều**

Để làm quen với cách tổ chức dữ liệu của xarray, trước tiên ta sẽ tạo một DataArray một chiều, trong đó mỗi giá trị được gắn với một tọa độ tương ứng.


```python
# Tạo DataArray đơn giản nhất - chỉ có data
simple_data = np.array([25.5, 26.8, 24.2, 27.1, 23.9])
simple_da = xr.DataArray(simple_data)
print(f"Loại dữ liệu: {type(simple_da)}")
# print chiều của DataArray 
print(f"Chiều của DataArray: {simple_da.shape}")
```


```python
# Dữ liệu nhiệt độ cho 5 thành phố Việt Nam
cities = ['Hà Nội', 'TP.HCM', 'Đà Nẵng', 'Cần Thơ', 'Hải Phòng']
temperatures = [25.5, 28.8, 26.2, 27.8, 24.9]

# Tạo DataArray với dimension labels
temp_cities = xr.DataArray(
    temperatures,
    dims=['city'],  # Tên dimension
    coords={'city': cities}  # Labels cho dimension
)
print(f"DataArray với dimension labels:\n{temp_cities.dims}")
print(f"Chiều của DataArray: {temp_cities.shape} và coordinates {temp_cities.coords}")
```

- **Tạo DataArray 2 chiều**

Trong ví dụ dưới đây, chúng ta sẽ tạo ra một DataArray 2 chiều mô phỏng nhiệt độ theo thời gian và thành phố. 


```python
# Tao 2-d DataArray cho nhiệt độ theo ngày và thành phố
dates = pd.date_range(start='2024-01-01', periods=3, freq='D')
cities = ['Hà Nội', 'TP.HCM']

temp_data = np.random.rand(3, 2) * 10 + 20  # Nhiệt độ ngẫu nhiên từ 20 đến 30 độ C
temp_2d = xr.DataArray(
    temp_data,
    dims=['time', 'city'],
    coords={'time': dates, 'city': cities}
)
# Dữ liệu nhiệt độ cho 2 thành phố qua 3 ngày. Dimension 'time' và 'city' giúp ta dễ dàng truy cập theo ngày hoặc theo thành phố.
```

- **Tạo DataArray n-chiều**


```python
# Tạo DataArray 3D cho nhiệt độ theo ngày, thành phố và giờ
cities = ['Hà Nội', 'TP.HCM', 'Đà Nẵng', 'Cần Thơ', 'Hải Phòng']
hours = np.arange(0, 24, 3)  # Mỗi 3 giờ một lần
dates = pd.date_range(start='2024-01-01', periods=10, freq='D')
temp_3d_data = np.random.rand(10, 5, len(hours)) * 10 + 20  # Nhiệt độ ngẫu nhiên từ 20 đến 30 độ C
temp_3d = xr.DataArray(
    temp_3d_data,
    dims=['time', 'city', 'hour'],
    coords={'time': dates, 'city': cities, 'hour': hours}
) # Dữ liệu nhiệt độ theo ngày, thành phố và giờ. Dimension 'time', 'city' và 'hour' giúp ta dễ dàng truy cập theo từng yếu tố.
print(f"Số chiều của dữ liệu: {len(temp_3d.dims)}")
```


```python
# Tạo dữ liệu nhiệt độ theo ngày và long và lat 
lats = np.linspace(-10, 10, 5)  # 5 điểm từ -10 đến 10 độ vĩ
lons = np.linspace(100, 120, 5)  # 5 điểm từ 100 đến 120 độ kinh
temp_grid_data = np.random.rand(10, 5, 5) * 10 + 20  # Nhiệt độ ngẫu nhiên từ 20 đến 30 độ C
temp_grid = xr.DataArray(
    temp_grid_data,
    dims=['time', 'lat', 'lon'],
    coords={'time': dates, 'lat': lats, 'lon': lons}
) # Dữ liệu nhiệt độ theo ngày và vị trí địa lý (long, lat). Dimension 'time', 'lat' và 'lon' giúp ta dễ dàng truy cập theo thời gian hoặc vị trí.
temp_grid.dims, temp_grid.coords
```

### 17.2.2. Tạo `DataSet` xarray

**Dataset** là cấu trúc dữ liệu chính của xarray để làm việc với nhiều biến cùng lúc. 

- **Tạo Dataset 1 chiều `DataArray`**

Cách đơn giản nhất để tạo Dataset là chuyển đổi từ một DataArray đã có bằng phương thức `.to_dataset()`. Bạn cần đặt tên cho biến (variable name) thông qua tham số `name`. Phương pháp này hữu ích khi bạn có một DataArray đơn lẻ và muốn mở rộng thành Dataset để sau này thêm các biến khác hoặc tích hợp với dữ liệu từ nguồn khác.


```python
data = [10, 20, 30, 40, 50]
da = xr.DataArray(data, dims=["time"], coords={"time": [0,1,2,3,4]})
ds = da.to_dataset(name="temperature")
print(f"Các biến dữ liệu {ds.variables}")
```

- **Tạo Dataset sử dụng `dictionary`**

Dataset thường được tạo từ dictionary trong đó mỗi key là tên biến và value là DataArray tương ứng. Tất cả các DataArrays phải có cùng dimensions và coordinates để có thể kết hợp trong một Dataset. Phương pháp này rất tiện lợi khi bạn có nhiều biến khí hậu (nhiệt độ, độ ẩm, áp suất...) cùng lưới không gian-thời gian và muốn quản lý chúng trong một cấu trúc thống nhất.


```python
dataset = xr.Dataset(
    {
        'temperature': temp_grid,
        'humidity': temp_grid - 5  # Giả sử độ ẩm thấp hơn nhiệt độ 5 độ
    }
)   
# print các biến dữ liệu trong dataset
print(f"Các biến dữ liệu trong dataset: {dataset.data_vars}")
```

## 17.3. Indexing và Selection

XArray cung cấp nhiều cách để truy cập và lọc dữ liệu đa chiều. Hai cách phổ biến nhất là dừng theo index hoặc theo nhãn (label).

### 17.3.1. Chọn theo index sử dung `.isel()`

Phương thức `.isel()` trong xarray được dùng để chọn dữ liệu theo vị trí chỉ số của các chiều (dimensions), tương tự như indexing trong NumPy. Điều này hữu ích khi muốn truy cập một phần dữ liệu dựa trên thứ tự phần tử thay vì giá trị tọa độ.


```python
# Lựa chọn index theo dimension 'city' cho thành phố 'Hà Nội' cho DataArray
subset = temp_2d.isel(city=0)
```


```python
# Lựa chọn index đầu tiên của dimension 'time'
subset = dataset.isel(time=0)
```

### 17.3.2. Chọn theo nhãn sử dụng `.sel()`

Phương thức `.sel()` trong xarray được dùng để chọn dữ liệu theo giá trị tọa độ (coordinate labels) của các chiều, thay vì theo vị trí chỉ số. Điều này giúp truy cập dữ liệu trực quan hơn khi các chiều có nhãn như thời gian, kinh độ hoặc vĩ độ.


```python
# Lựa chọn label theo dimension 'city' cho thành phố 'Hà Nội' cho DataArray
subset = temp_2d.sel(city='Hà Nội')
```


```python
# Lựa chọn ngày cụ thể theo dimension 'time' cho ngày '2024-01-01' cho Dataset
subset = dataset.sel(time='2024-01-01')
```

# 17.4. Đọc và viết dữ liệu

###  17.4.1. Đọc dữ liệu

Xarray hỗ trợ nhiều định dạng phổ biến như NetCDF, GRIB, HDF5, Zarr,… thông qua các hàm như `open_dataset()` hoặc `open_dataarray()`.


```python
url = 'http://dapds00.nci.org.au/thredds/dodsC/rr3/CMIP5/output1/CSIRO-BOM/ACCESS1-3/historical/mon/atmos/Amon/r1i1p1/latest/tas/tas_Amon_ACCESS1-3_historical_r1i1p1_185001-200512.nc'
ds = xr.open_dataset(url)
```

### 17.4.2. Viết dữ liệu

Sau khi xử lý và phân tích, bạn có thể lưu Dataset hoặc DataArray ra file bằng phương thức `.to_netcdf()`. NetCDF là định dạng chuẩn cho dữ liệu khoa học đa chiều, được sử dụng rộng rãi trong khí hậu học, hải dương học và viễn thám. File NetCDF lưu trữ cả dữ liệu và metadata (dimensions, coordinates, attributes), đảm bảo tính tái sử dụng và khả năng chia sẻ dữ liệu giữa các nhà nghiên cứu. Xarray cũng hỗ trợ xuất sang Zarr format cho big data.


```python
outfile = r"yourpath\data\raster\data_netcdf.nc"
# ds.to_netcdf(outfile) # Lưu dataset vào file NetCDF. 
```

## 17.5. Tính toán và thống kê

XArray cung cấp nhiều hàm tính toán cho dữ liệu đa chiều.

### 17.5.1. Các phép tổng hợp

XArray cung cấp các phép toán tổng hợp (aggregation) mạnh mẽ để tính toán thống kê trên dữ liệu đa chiều. Các hàm như `mean()`, `median()`, `sum()`, `min()`, `max()`, `std()` có thể áp dụng trên toàn bộ mảng hoặc chỉ theo một/nhiều dimensions cụ thể thông qua tham số `dim`. Ví dụ, tính mean theo dimension 'time' sẽ tạo climatology (khí hậu trung bình), còn tính mean theo 'x' và 'y' sẽ tạo time series trung bình không gian. Khả năng chỉ định dimensions giúp phân tích linh hoạt theo nhiều khía cạnh khác nhau.


```python
# Đọc file tif và trả về Dataset. Đây là dữ liệu về nhiệt độ
data = xr.open_dataset(r"G:\My Drive\python\geocourse\data\raster\Vietnam_Temperature_2020.tif")
# Chọn dimension 'band' để lấy dữ liệu nhiệt độ
temp = data['band_data']
temp.attrs = ""
# Tạo ra một danh sách hàng tháng cho 2020 
time = pd.date_range(start='2020-01-01', periods=len(temp), freq='ME')
temp['band'] = time  # Gán dimension 'band' thành 'time' với giá trị là danh sách thời gian hàng tháng
temp = temp.rename({
    'band': 'time'
}) # Đổi tên dimension 'band' thành 'time' để dễ hiểu hơn. Bây giờ ta có một DataArray với dimension 'time' và dữ liệu nhiệt độ tương ứng cho mỗi tháng từ 2020 đến 2024.
```

- **Tính giá trị trung bình theo `dimension` xác định**

Phương thức `.mean(dim='time')` tính giá trị trung bình dọc theo dimension 'time', thu gọn dimension đó và trả về DataArray có số chiều ít hơn. Kết quả là một bản đồ climatology - giá trị nhiệt độ trung bình theo thời gian tại mỗi điểm không gian. Đây là phép toán phổ biến nhất trong phân tích khí hậu để xác định mức chuẩn (baseline) hoặc trung bình dài hạn. Bạn cũng có thể tính mean theo nhiều dimensions cùng lúc, ví dụ `dim=['time', 'x']`.


```python
mean_temp = temp.mean(dim='time') # Tính giá trị trung bình của nhiệt độ qua tất cả các tháng. Kết quả sẽ là một DataArray mới với giá trị là trung bình của nhiệt độ qua thời gian.
```

- **Tính giá trị trung vị theo `dimension` xác định**

Phương thức `.median(dim='time')` tính giá trị trung vị thay vì trung bình, giúp loại bỏ ảnh hưởng của outliers (giá trị ngoại lệ). Trung vị ít bị ảnh hưởng bởi các giá trị cực đoan như sóng nhiệt hay đợt lạnh bất thường, do đó có thể phản ánh xu hướng trung tâm đáng tin cậy hơn trong một số trường hợp. Phép toán này đặc biệt hữu ích khi dữ liệu có phân phối không đối xứng hoặc chứa nhiễu cao.


```python
median = temp.median(dim='time') # Tính giá trị trung vị của nhiệt độ qua tất cả các tháng. Kết quả sẽ là một DataArray mới với giá trị là trung vị của nhiệt độ qua thời gian.
```

- **Tính tổng theo `dimension` xác định**

Phương thức `.sum(dim='time')` tính tổng các giá trị dọc theo dimension 'time'. Phép toán này hữu ích khi bạn muốn tích lũy giá trị theo thời gian, ví dụ tổng lượng mưa trong năm, tổng bức xạ mặt trời, hoặc tổng carbon flux. Đối với các biến tích lũy như lượng mưa, tổng giá trị có ý nghĩa thực tế quan trọng hơn trung bình. Lưu ý rằng kết quả phụ thuộc vào đơn vị và time step của dữ liệu gốc.


```python
total = temp.sum(dim='time') # Tính tổng của nhiệt độ qua tất cả các tháng. Kết quả sẽ là một DataArray mới với giá trị là tổng của nhiệt độ qua thời gian.
```

- **Tính giá trị min, max, std theo `dimension` xác định**

Ba phương thức `.min()`, `.max()`, `.std()` tính các thống kê cực trị và độ biến động. Min/max cho biết nhiệt độ thấp nhất/cao nhất đã quan sát được, hữu ích để xác định records hoặc extreme events. Standard deviation (std) đo lường độ dao động xung quanh giá trị trung bình - std cao chỉ ra biến động lớn theo thời gian hoặc không gian. Các thống kê này thường được dùng để phát hiện xu hướng biến đổi khí hậu và đánh giá rủi ro cực đoan.


```python
min_temp = temp.min(dim='time') # Tính giá trị nhỏ nhất của nhiệt độ qua tất cả các tháng. Kết quả sẽ là một DataArray mới với giá trị là nhiệt độ thấp nhất qua thời gian.
max_temp = temp.max(dim='time') # Tính giá trị lớn nhất của nhiệt độ qua tất cả các tháng. Kết quả sẽ là một DataArray mới với giá trị là nhiệt độ cao nhất qua thời gian.
std_temp = temp.std(dim='time') # Tính độ lệch chuẩn của nhiệt độ qua tất cả các tháng. Kết quả sẽ là một DataArray mới với giá trị là độ lệch chuẩn của nhiệt độ qua thời gian.
```

### 17.5.2. Phép tính đơn giản khác

Ngoài tính toán theo một dimension, bạn có thể tính mean theo nhiều dimensions cùng lúc, ví dụ `.mean(dim=('x', 'y'))` để tính trung bình không gian cho mỗi time step. Điều này tạo ra time series đại diện cho giá trị trung bình của toàn vùng nghiên cứu, rất hữu ích để theo dõi xu hướng tổng thể theo thời gian mà không quan tâm đến biến động không gian. Kỹ thuật này thường dùng trong phân tích regional climate hoặc monitoring các chỉ số tổng hợp.


```python
temporal_mean = temp.mean(dim=('x', 'y')) # Tính giá trị trung bình của nhiệt độ qua tất cả các điểm không gian (x và y) cho mỗi thời điểm. Kết quả sẽ là một DataArray mới với giá trị là trung bình của nhiệt độ qua không gian cho mỗi thời điểm.
```

## 17.6. Tính toán dựa theo nhóm

`GroupBy` là một trong những tính năng mạnh nhất của XArray cho phân tích theo thời gian.

### 17.6.1. Tính toán theo nhóm

GroupBy là một trong những tính năng mạnh mẽ nhất của XArray, cho phép nhóm dữ liệu theo các thuộc tính thời gian (month, year, season, dayofyear...) và áp dụng các phép tính tổng hợp. `.groupby('time.month')` nhóm tất cả các time steps có cùng tháng (January, February,...) rồi tính toán trên từng nhóm. Điều này giúp trích xuất patterns theo mùa vụ, chu kỳ hàng năm hay biến động seasonal. GroupBy tương tự như trong pandas nhưng được tối ưu cho dữ liệu đa chiều và xử lý coordinates tự động.

- **Tính trung bình theo tháng**

`.groupby('time.month').mean()` tính climatology hàng tháng - nhiệt độ trung bình cho mỗi tháng trong lịch (January = tháng 1, February = tháng 2,...) dựa trên toàn bộ dữ liệu lịch sử. Kết quả là 12 giá trị đại diện cho chu kỳ mùa vụ điển hình. Đây là công cụ cơ bản để hiểu climate normal và so sánh anomalies. Ví dụ, bạn có thể thấy tháng 7-8 luôn nóng nhất, tháng 12-1 lạnh nhất ở miền Bắc Việt Nam.


```python
month_mean = temp.groupby('time.month').mean() # Tính giá trị trung bình của nhiệt độ theo từng tháng trong giai đoạn quan sát. Kết quả sẽ là một DataArray mới với giá trị là trung bình của nhiệt độ cho mỗi tháng (từ 1 đến 12).
# Tương tự như vậy, ta có thể tính min, max, std theo từng tháng bằng cách thay mean() bằng min(), max(), std() trong câu lệnh trên.
```

- **Tính trung bình theo năm**

`.groupby('time.year').mean()` tính giá trị trung bình cho mỗi năm trong dataset, tạo ra annual time series. Phép toán này loại bỏ biến động seasonal và giúp nhận diện xu hướng dài hạn như global warming hay climate variability. So sánh annual means giữa các năm cho thấy năm nào nóng/lạnh bất thường hoặc có climate extremes. Đây là bước đầu tiên trong phân tích xu hướng và trend detection.


```python
yearly_mean = temp.groupby('time.year').mean() # Tính giá trị trung bình của nhiệt độ theo từng năm trong giai đoạn quan sát. Kết quả sẽ là một DataArray mới với giá trị là trung bình của nhiệt độ cho mỗi năm.
```

- **Tính trung bình theo mùa**

`.groupby('time.season').mean()` nhóm dữ liệu theo 4 mùa meteorological chuẩn: DJF (Dec-Jan-Feb = Winter), MAM (Mar-Apr-May = Spring), JJA (Jun-Jul-Aug = Summer), SON (Sep-Oct-Nov = Autumn). Kết quả là 4 giá trị đại diện cho nhiệt độ trung bình mùa. Phân tích seasonal rất quan trọng để hiểu climate patterns, ảnh hưởng monsoon, và planning cho nông nghiệp. Lưu ý rằng definition của mùa có thể khác nhau giữa Northern và Southern Hemisphere.


```python
seasonal_mean = temp.groupby('time.season').mean() # Tính giá trị trung bình của nhiệt độ theo từng mùa trong giai đoạn quan sát. Kết quả sẽ là một DataArray mới với giá trị là trung bình của nhiệt độ cho mỗi mùa (Spring, Summer, Autumn, Winter).
```

- **Tính trung bình theo giai đoạn**

Bạn có thể tự định nghĩa giai đoạn custom bằng cách lọc dữ liệu trước khi groupby. Ví dụ, chọn các tháng mùa hè (4-8) với `.isin(summer_months)`, sau đó group theo năm để tính summer mean cho mỗi năm. Kỹ thuật này cho phép phân tích các periods đặc biệt như: mùa mưa (May-Oct), mùa khô (Nov-Apr), hoặc growing season của cây trồng. Sự linh hoạt này rất quan trọng cho regional climate analysis phù hợp với đặc thù địa phương.


```python
# Calculate mean cho tháng 4,5,6,7,8 mỗi năm 
summer_months = [4, 5, 6, 7, 8]
# Lọc dữ liệu cho các tháng sau 
summer_data = temp.sel(time=temp['time.month'].isin(summer_months))
# Tính giá trị trung bình cho các tháng mùa hè theo năm 
summer_mean = summer_data.groupby('time.year').mean()

```

### 17.6.2. Tính theo rolling 

`rolling` trong Xarray dùng để tính toán trên cửa sổ trượt (moving window) dọc theo một chiều (thường là time). Ví dụ như bạn muốn tính trung bình nhiệt độ theo thời gian với một cửa sổ 3 tháng, giúp làm mượt dữ liệu theo thời gian. Tuy nhiên, lưu ý rằng các giá trị ở đầu và cuối của chuỗi thời gian sẽ có giá trị NaN do không đủ dữ liệu để tính toán trong cửa sổ 3 tháng.


```python
# Tính rolling mean với cửa sổ 3 tháng để làm mượt dữ liệu nhiệt độ theo thời gian
rolling_mean = temp.rolling(time=3, center=True).mean() # Tính rolling mean với cửa sổ 3 tháng. Kết quả sẽ là một DataArray mới với giá trị là trung bình của nhiệt độ trong cửa sổ 3 tháng, giúp làm mượt dữ liệu theo thời gian. Tuy nhiên, tháng đầu và tháng cuối sẽ có giá trị NaN do không đủ dữ liệu để tính trung bình trong cửa sổ 3 tháng.
# Tương tự vậy, bạn có thể tính rolling min, max, std bằng cách thay mean() bằng min(), max(), std() trong câu lệnh trên và theo time step khác nhau (ví dụ: time=6 cho rolling 6 tháng, time=12 cho rolling 1 năm, v.v.).
```

### 17.6.3. Tính theo resampling

Resample là kỹ thuật tái lấy mẫu (resampling) dữ liệu theo tần suất thời gian mới. `.resample(time='1Y').mean()` gom tất cả monthly data trong mỗi năm và tính trung bình, tạo annual time series. Khác với groupby('time.year'), resample giữ nguyên datetime index và có thể handle các time frequencies phức tạp (quarters, weeks, custom periods). Resample hỗ trợ cả upsampling (tăng tần suất với interpolation) và downsampling (giảm tần suất với aggregation), là công cụ chuẩn cho time series analysis và data harmonization.


```python
# Tính giá trị trung bình nhiệt độ sử dụng resample để tính theo năm
yearly_mean_resample = temp.resample(time='1Y').mean() # Tính giá trị trung bình của nhiệt độ theo năm sử dụng resample. Kết quả sẽ là một DataArray mới với giá trị là trung bình của nhiệt độ cho mỗi năm, được tính bằng cách lấy trung bình của tất cả các tháng trong mỗi năm.
```

## 17.7. Tính toán nâng cao

XArray cung cấp các công cụ mạnh mẽ để áp dụng các hàm tùy chỉnh.

### 17.7.1. Tính toán sử dụng hàm `apply_ufunc`

`apply_ufunc()` trong xarray được dùng để áp dụng các hàm NumPy hoặc hàm tùy chỉnh lên dữ liệu của `DataArray/Dataset` theo cách tương thích với dimensions và coordinates của xarray. Mục tiêu chính là giúp vector hóa phép tính, tự động broadcast dữ liệu theo chiều và hỗ trợ xử lý song song với `Dask` khi làm việc với dữ liệu lớn.

- **Tính sự bất thường nhiệt độ (anomaly)**

Giả sử ta muốn đánh giá liệu nhiệt độ tại một thời điểm có cao hay thấp hơn mức bình thường hay không. Một cách tiếp cận phổ biến là tính sự bất thường nhiệt độ (temperature anomaly), được xác định bằng độ chênh lệch giữa nhiệt độ quan sát và nhiệt độ trung bình của giai đoạn tham chiếu. Giá trị dương biểu thị nhiệt độ cao hơn trung bình, trong khi giá trị âm cho thấy nhiệt độ thấp hơn trung bình.


```python
def calculate_annomaly(x):
    return x-np.mean(x)
# Tính anomaly bằng cách trừ đi giá trị trung bình của toàn bộ dữ liệu
anomaly = xr.apply_ufunc(
    calculate_annomaly,
    temp,
    input_core_dims=[['time']],  # Chỉ định dimension 'time' là dimension cốt lõi để áp dụng hàm
    output_core_dims=[['time']],  # Kết quả cũng sẽ có dimension 'time'
    vectorize=True  # Cho phép áp dụng hàm cho từng phần tử trong DataArray
)
```

- **Smooth dữ liệu theo thời gian**
  
  Giả sử ta muốn theo dõi sự thay đổi của thảm thực vật trong suốt một mùa sinh trưởng thông qua chỉ số NDVI. Tuy nhiên, chuỗi thời gian NDVI thường bị nhiễu bởi mây hoặc các điều kiện quan trắc khác, dẫn đến các giá trị tăng giảm bất thường. Trong ví dụ này, ta sẽ áp dụng kỹ thuật `savgol_filter` làm mượt chuỗi thời gian NDVI để loại bỏ nhiễu và thu được đường cong biến động ổn định hơn, phản ánh chính xác hơn trạng thái của thảm thực vật.


```python
# Ví dụ tạo ra dữ liệu NDVI hàng tháng từ 2000 đến 2025 cho một vùng cụ thể với kích thước 10x10 điểm lưới
dates = pd.date_range(start='2000-01-01', end='2025-12-31', freq='ME')  # Tạo danh sách ngày hàng tháng từ 2000 đến 2025
lats = np.linspace(-10, 10, 10)  # 10 điểm vĩ từ -10 đến 10 độ
lons = np.linspace(100, 120, 10)  # 10 điểm kinh từ 100 đến 120 độ
ndvi_data = np.random.rand(len(dates), len(lats), len(lons))  # Dữ liệu NDVI ngẫu nhiên từ 0 đến 1
ndvi = xr.DataArray(
    ndvi_data,
    dims=['time', 'lat', 'lon'],
    coords={'time': dates, 'lat': lats, 'lon': lons}
) # Dữ liệu NDVI hàng tháng từ 2000 đến 2025 cho một vùng cụ thể với kích thước 10x10 điểm lưới. Dimension 'time', 'lat' và 'lon' giúp ta dễ dàng truy cập theo thời gian hoặc vị trí.
# Dùng hàm savgo filter để làm mượt dữ liệu NDVI theo thời gian với cửa sổ 5 tháng và độ mượt 2
from scipy.signal import savgol_filter
def smooth_ndvi(x, window_length=5, polyorder=2):
    return savgol_filter(x, window_length=window_length, polyorder=polyorder, mode='nearest')
smoothed_ndvi = xr.apply_ufunc(
    smooth_ndvi,
    ndvi,
    input_core_dims=[['time']],  # Chỉ định dimension 'time' là dimension
    output_core_dims=[['time']],  # Kết quả cũng sẽ có dimension 'time'
    vectorize=True,  # Cho phép áp dụng hàm cho từng phần tử trong DataArray
    kwargs={'window_length': 3, 'polyorder': 2}  # Truyền tham số cho hàm smooth_ndvi
)
# chuyển đổi thứ tự dimension về ban đầu như ndvi.
smoothed_ndvi = smoothed_ndvi.transpose('time', 'lat', 'lon')
```

### 17.7.2. Tính toán sử dụng hàm `map_blocks`

`map_blocks()` trong xarray được dùng để áp dụng một hàm lên từng block (chunk) của dữ liệu lớn thay vì xử lý toàn bộ mảng cùng lúc. Mục tiêu chính là giúp xử lý dữ liệu theo từng phần nhỏ để tiết kiệm bộ nhớ, hỗ trợ tính toán song song và tăng hiệu năng khi làm việc với dữ liệu lớn thông qua `Dask`.


```python
# Tạo một DataArray 2D với dimension 'x' và 'y' và dữ liệu từ 0 đến 15 được sắp như sau
data = xr.DataArray(
    np.arange(16).reshape(4, 4),  # Dữ liệu từ 0 đến 15 được sắp xếp thành ma trận 4x4
    dims=["x", "y"]
)
# Chunk dữ liệu thành các block 2x2
data = data.chunk({'x': 2, 'y': 2})

def add(block):
    return block + 10
# Áp dụng hàm add cho từng block 2x2 trong DataArray. Kết quả sẽ là một DataArray mới với giá trị được cộng thêm 10 cho mỗi block 2x2.
result = xr.map_blocks(add, data, template=data).compute()
```

### 17.7.3. Tính toán sử dụng `map_overlap`

`map_overlap()` là hàm của Dask dùng để áp dụng một hàm lên từng chunk nhưng **cho phép mỗi chunk mượn thêm dữ liệu từ chunk lân cận** (vùng overlap), tránh hiệu ứng biên. Thường dùng cho các phép tính không gian như spatial filtering, convolution, hay tính gradient - những phép tính mà kết quả tại một điểm phụ thuộc vào các điểm xung quanh nó.



```python
from scipy.ndimage import uniform_filter
import dask.array as da
def smooth_block(block):
    return uniform_filter(block, size=3)  # Làm mượt block 3x3 bằng cách lấy trung bình của các giá trị trong block
data = xr.DataArray(
    np.arange(10000).reshape(100, 100),  # Dữ liệu ngẫu nhiên cho DataArray 2D
    dims=["x", "y"]
).chunk({'x': 10, 'y': 10})  # Chunk dữ liệu thành các block 10x10

# Áp dụng map_overlap với depth=1 (mỗi chunk mượn thêm 1 hàng/cột từ chunk lân cận)
# depth=1 đảm bảo tính đúng tại vùng biên của từng chunk
smoothed = da.map_overlap(
    smooth_block,
    data.data,       # Dask array từ DataArray
    depth=1,             # Số hàng/cột chồng lấp với chunk lân cận
    boundary='reflect',  # Cách xử lý biên ngoài cùng: phản chiếu dữ liệu or None để không mượn dữ liệu
    dtype=data.dtype,
)
```

## Tóm tắt

Bạn đã hoàn thành Bài 17 và học được XArray - thư viện mạnh mẽ cho xử lý mảng đa chiều trong hệ sinh thái Python.

### Các khái niệm chính đã nắm vững:
- ✅ **DataArray & Dataset**: Mảng có nhãn với tọa độ, chiều dữ liệu và quản lý thuộc tính
- ✅ **Indexing**: Lựa chọn theo nhãn với `.sel()`, `.isel()` và lọc theo điều kiện boolean
- ✅ **Tính toán đa chiều**: Tổng hợp thống kê, rolling operations và phân tích thống kê
- ✅ **Nhóm theo thời gian**: Phân tích theo mùa, thống kê khí hậu và nhóm thời gian tùy chỉnh
- ✅ **Tính toán nâng cao**: apply_ufunc cho hàm tùy chỉnh và map_blocks cho xử lý theo khối
- ✅ **Tích hợp hệ sinh thái**: Tương thích liền mạch với pandas, matplotlib và các thư viện địa không gian
- ✅ **Tối ưu hiệu suất**: Tích hợp Dask, lazy evaluation và xử lý dữ liệu tiết kiệm bộ nhớ

### Kỹ năng bạn có thể áp dụng:
- Xử lý và phân tích tập dữ liệu khí hậu và dữ liệu khoa học một cách chuyên nghiệp
- Thực hiện phân tích theo thời gian cho các mẫu thời tiết và biến thiên theo mùa tại Việt Nam
- Tối ưu xử lý dữ liệu đa chiều cho các ứng dụng khoa học quy mô lớn
- Tích hợp XArray với NumPy, pandas và bộ công cụ địa không gian cho các quy trình phân tích nâng cao
- Xây dựng nền tảng chuyên sâu cho các dự án mô hình hóa khí hậu và giám sát môi trường

