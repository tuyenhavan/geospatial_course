# Bài 24: Tổng hợp ảnh theo giai đoạn (GEE)

Tổng hợp ảnh là kỹ thuật kết hợp nhiều ảnh thành một ảnh đại diện cho một giai đoạn (tháng, mùa, năm). Kĩ thuật này giúp giảm ảnh hưởng của mây và cung cấp dữ liệu đầu vào ổn định hơn cho các phân tích không gian.


## 24.1. Mục tiêu học tập

Sau bài này bạn có thể:

- Giải thích sự khác biệt giữa các phương pháp: **median**, **mean**, **mosaic**, **percentile**, **max NDVI**
- Tổng hợp ảnh theo tháng với dữ liệu Sentinel-2 SR và Sentinel-1
- Tổng hợp ảnh theo mùa với dữ liệu Landsat 9 LST và MODIS LST
- Tổng hợp ảnh theo năm với dữ liệu khí hậu ERA5


```python
import ee
import geemap

ee.Authenticate()
ee.Initialize()
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>




```python
# Trong bài học này, chúng ta sẽ sử dụng một bounding box nhỏ ở miền nam nước Đức để làm ví dụ. Bạn có thể thay đổi bbox này để phù hợp với khu vực bạn quan tâm.
bbox = [9.84375   , 47.5172007 , 10.1953125 , 47.75409798]
# Tạo một đối tượng hình chữ nhật từ bounding box
roi = ee.Geometry.Rectangle(bbox)
# Xác định khoảng thời gian cho bộ dữ liệu Sentinel-2
start_date = '2023-06-01'
end_date = '2025-09-30'
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>



## 24.2. Phương pháp tổng hợp ảnh theo thời gian
### 24.2.1. Các phương pháp tổng hợp ảnh phổ biến
Tổng hợp dữ liệu ảnh theo thời gian là phương pháp chia chuỗi ảnh thành các giai đoạn xác định (tháng, mùa hoặc một khoảng thời gian nghiên cứu cụ thể) và thu thập tất cả ảnh trong từng giai đoạn đó. Sau khi loại bỏ mây, các ảnh trong cùng một giai đoạn được tổng hợp thành một ảnh đại diện thông qua các phép thống kê như trung bình (mean), trung vị (median), giá trị lớn nhất (max) hoặc nhỏ nhất (min). Phương pháp này giúp giảm nhiễu, hạn chế ảnh hưởng của dữ liệu thiếu và phản ánh đặc trưng bề mặt ổn định hơn theo thời gian. Dưới đây là các phương pháp thống kê phổ biến cho tổng hợp ảnh theo thời gian có sẵn tỏng GEE. 
| Phương pháp | Hàm trong GEE | Ưu điểm | Nhược điểm / Lưu ý |
|-------------|-----------|---------|-------------------|
| **Median** | `.median()` | Loại outlier tốt, ít bị ảnh hưởng bởi mây sót | Cần ≥ 3 ảnh/pixel |
| **Mean** | `.mean()` | Đơn giản, phù hợp khi đã mask sạch | Nhạy với outlier |
| **Max NDVI** | `.qualityMosaic('NDVI')` | Ưu tiên pixel xanh nhất - tốt cho vegetation | Có thể lấy pixel mây |
| **Mosaic** | `.mosaic()` | Ghép ảnh theo thứ tự (ảnh đầu tiên ưu tiên) | Thường cần sort trước |
| **Percentile** | `.reduce(ee.Reducer.percentile([p]))` | Linh hoạt - p25 cho đất, p75 cho thực vật | Chậm hơn median |
| **Min** | `.min()` | Phát hiện nước, bóng mây | Dễ bị mây/shadow |
```


### 24.2.2. Ý tưởng thực hiện tổng hợp ảnh trong GEE

Trong Google Earth Engine, việc tổng hợp ảnh theo giai đoạn thời gian được thực hiện bằng cách xác định thời gian bắt đầu và kết thúc của bộ dữ liệu, sau đó tạo một danh sách các mốc thời gian bắt đầu theo bước nhảy phù hợp với giai đoạn mong muốn (ví dụ: 1 tháng, 1 năm hoặc 1 mùa). Mỗi mốc thời gian được sử dụng để xác định khoảng thời gian tổng hợp tương ứng, từ đó trích xuất và tổng hợp các ảnh trong khoảng đó bằng các phép thống kê như trung bình (mean), trung vị (median), giá trị lớn nhất (max) hoặc nhỏ nhất (min). Cuối cùng, các ảnh đại diện của từng giai đoạn được gộp thành một `ImageCollection` để phục vụ phân tích chuỗi thời gian.


```python
# Viết 1 hàm để lấy thời gian đầu và cuối cùng của một ImageCollection
def date_range_col(col):
    """Return the first and latest datetimes of image acquision in the collection

    Args:
        col (ee.ImageCollection): The input image collection.

    Returns:
        tuple: The ee.Date object of the first and latest datetimes of image acquision in the collection.
    """
    first_date = ee.Date(col.first().get("system:time_start"))
    latest_date = ee.Date(
        col.limit(1, "system:time_start", False).first().get("system:time_start")
    )
    return first_date, latest_date

def monthly_datetime_list(first_date, latest_date):
    """Return a list of monthly datetime objects.

    Args:
        first_date(ee.date): The first date of collection.
        latest_date(ee.Date): The latest date of collection.

    Returns:
        ee.List: The list of monthly datetime objects.
    """
    m = ee.Number.parse(first_date.format("MM"))
    y = ee.Number.parse(first_date.format("YYYY"))
    month_count = latest_date.difference(first_date, "month").round()
    month_list = ee.List.sequence(0, month_count)

    def month_step(month):
        first_month = ee.Date.fromYMD(y, m, 1)
        next_month = first_month.advance(month, "month")
        return next_month.millis()

    monthly_list = month_list.map(month_step)
    return monthly_list

def yearly_datetime_list(first_date, latest_date):
    """Return a list of yearly datetime objects.

    Args:
        first_date(ee.date): The first date of collection.
        latest_date(ee.Date): The latest date of collection.

    Returns:
        ee.List: The list of yearly datetime objects.
    """
    y = ee.Number.parse(first_date.format("YYYY"))
    year_count = latest_date.difference(first_date, "year").round()
    year_list = ee.List.sequence(0, year_count)

    def year_step(year):
        first_year = ee.Date.fromYMD(y, 1, 1)
        next_year = first_year.advance(year, "year")
        return next_year.millis()

    yearly_list = year_list.map(year_step)
    return yearly_list
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>



## 24.3. Tổng hợp ảnh Sentinel-2 theo thời gian

Trong mục này, chúng ta sẽ học cách tổng hợp ảnh Sentinel-2 theo các thời gian, giai đoạn khác nhau. Để tiết kiệm thời gian, chúng ta sẽ không loại bỏ mây. Nếu bạn muốn loại bỏ mây, bạn có thể xem lại Bài 22, và sau đó tổng hợp ảnh bình thường như nhưng phương pháp bên dưới. Các bạn có thể mổ rộng code sau cho các bộ dữ liệu khác như Landsat hoặc MODIS.


```python
sen2col = (ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
          .filterBounds(roi)
          .filterDate(start_date, end_date)
          .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 25)))
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>



### 24.3.1. Tổng hợp ảnh Sentinel-2 theo tháng


```python
# Tổng hợp ảnh theo tháng bằng cách lấy giá trị trung bình. Bạn có thể thay đổi sử dụng các phương pháp tổng hợp khác như median, min, max,... tùy theo mục đích phân tích của bạn.
def generate_monthly_composite(col):
    """Generate monthly composite by taking the mean value of images in each month.

    Args:
        col (ee.ImageCollection): The input image collection.

    Returns:
        ee.ImageCollection: The monthly composite image collection.
    """
    first_date, latest_date = date_range_col(col)
    monthly_list = monthly_datetime_list(first_date, latest_date)

    def monthly_composite(date):
        date = ee.Date(date)
        start = date
        end = date.advance(1, 'month')
        monthly_col = col.filterDate(start, end)
        composite = monthly_col.mean().set('system:time_start', start.millis())
        return composite

    monthly_composite_col = ee.ImageCollection(monthly_list.map(monthly_composite))
    return monthly_composite_col
sen2col_monthly_composite = generate_monthly_composite(sen2col)
print('Số lượng ảnh sau khi tổng hợp theo tháng:', sen2col_monthly_composite.size().getInfo())
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>



    Số lượng ảnh sau khi tổng hợp theo tháng: 27
    

### 24.3.2. Tổng hợp ảnh Sentinel-2 theo mùa


```python
# Tổng hợp ảnh mùa hè bằng cách lấy giá trị trung bình. Bạn có thể thay đổi sử dụng các phương pháp tổng hợp khác như median, min, max,... tùy theo mục đích phân tích của bạn.
def generate_summer_composite(col):
    """Generate summer composite by taking the mean value of images in June, July, and August.

    Args:
        col (ee.ImageCollection): The input image collection.
    Returns:
        ee.ImageCollection: The summer composite image collection.
    """
    first_date, latest_date = date_range_col(col) # Lấy thời gian đầu và cuối của collection
    yearly_list = yearly_datetime_list(first_date, latest_date) # Tạo danh sách các mốc thời gian đầu của mỗi năm
    def summer_composite(date):
        date = ee.Date(date)
        start = date
        end = date.advance(1, 'year')
        yearly_col = col.filterDate(start, end)
        summer_col = yearly_col.filter(ee.Filter.calendarRange(6, 8, 'month')) # Lọc ảnh trong tháng 6, 7, 8
        composite = summer_col.mean().set('system:time_start', start.millis()) # Tính giá trị trung bình và gán thời gian là mốc thời gian đầu của năm
        return composite
    summer_composite_col = ee.ImageCollection(yearly_list.map(summer_composite)) # Tạo ImageCollection từ danh sách các mốc thời gian đầu của mỗi năm
    return summer_composite_col

sen2col_summer_composite = generate_summer_composite(sen2col)
print('Số lượng ảnh sau khi tổng hợp theo mùa hè:', sen2col_summer_composite.size().getInfo())
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>



    Số lượng ảnh sau khi tổng hợp theo mùa hè: 3
    

### 24.3.3. Tổng hợp ảnh Sentinel-2 theo năm


```python
# Tổng hợp ảnh theo năm bằng cách lấy giá trị trung bình. Bạn có thể thay đổi sử dụng các phương pháp tổng hợp khác như median, min, max,... tùy theo mục đích phân tích của bạn.
def generate_yearly_composite(col):
    """Generate yearly composite by taking the mean value of images in each year.

    Args:
        col (ee.ImageCollection): The input image collection.

    Returns:
        ee.ImageCollection: The yearly composite image collection.
    """
    first_date, latest_date = date_range_col(col)
    yearly_list = yearly_datetime_list(first_date, latest_date)

    def yearly_composite(date):
        date = ee.Date(date)
        start = date
        end = date.advance(1, 'year')
        yearly_col = col.filterDate(start, end)
        composite = yearly_col.mean().set('system:time_start', start.millis())
        return composite

    yearly_composite_col = ee.ImageCollection(yearly_list.map(yearly_composite))
    return yearly_composite_col

sen2col_yearly_composite = generate_yearly_composite(sen2col)
print('Số lượng ảnh sau khi tổng hợp theo năm:', sen2col_yearly_composite.size().getInfo())
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>



    Số lượng ảnh sau khi tổng hợp theo năm: 3
    

## 24.4. Tổng hợp ảnh theo thời gian với dữ liệu Landsat

Trong phần này, chúng ta sẽ tổng hợp ảnh theo thời gian sử dụng `.qualityMosaic()` để tạo ra ảnh đại diện có chất lượng cho từng giai đoạn. Phương pháp này hoạt động bằng cách lựa chọn, tại mỗi pixel, giá trị từ ảnh có giá trị cao nhất của một band được chỉ định (ví dụ: NDVI, EVI hoặc một chỉ số chất lượng khác). Sau khi xác định ảnh có giá trị cao nhất tại từng vị trí, tất cả các bands còn lại của pixel đó sẽ được lấy từ cùng một ảnh, giúp tạo ra ảnh tổng hợp giữ được tính nhất quán phổ và hạn chế ảnh hưởng của mây, nhiễu hoặc dữ liệu chất lượng thấp.


### 24.4.1. Tính chỉ số NDVI Landsat


```python
# Lấy bộ sưu tập Landsat 8, áp dụng hàm chuẩn bị dữ liệu
ls8col = (ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')
          .filterDate(start_date, end_date)
            .filterBounds(roi)
            .filter(ee.Filter.lt('CLOUD_COVER', 30)))

def calculate_ndvi(image):
    """Calculate NDVI for a given image.

    Args:
        image (ee.Image): The input image.
    Returns:
        ee.Image: The NDVI image.
    """
    # chuyển đổi giá trị pixel 0.0000275	-0.2
    img = image.select(['SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B6', 'SR_B7'])
    img = img.multiply(0.0000275).add(-0.2)
    ndvi = img.normalizedDifference(['SR_B5', 'SR_B4']).rename('NDVI')
    return image.addBands(ndvi)
ls8col_ndvi = ls8col.map(calculate_ndvi)
print('Số lượng ảnh Landsat 8 sau khi tính NDVI:', ls8col_ndvi.size().getInfo())
print(f"Thông tin bands của ảnh Landsat 8 sau khi tính NDVI: {ls8col_ndvi.first().bandNames().getInfo()}")
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>



    Số lượng ảnh Landsat 8 sau khi tính NDVI: 36
    Thông tin bands của ảnh Landsat 8 sau khi tính NDVI: ['SR_B1', 'SR_B2', 'SR_B3', 'SR_B4', 'SR_B5', 'SR_B6', 'SR_B7', 'SR_QA_AEROSOL', 'ST_B10', 'ST_ATRAN', 'ST_CDIST', 'ST_DRAD', 'ST_EMIS', 'ST_EMSD', 'ST_QA', 'ST_TRAD', 'ST_URAD', 'QA_PIXEL', 'QA_RADSAT', 'NDVI']
    

### 24.4.2. Tổng hợp ảnh Landsat sử dụng `qualityMosaic`

Tổng hợp ảnh theo năm bằng `qualityMosaic` là một phương pháp để tạo ra composite hàng năm bằng cách chọn pixel có giá trị NDVI (hoặc một chỉ số khác) cao nhất trong mỗi năm hoặc giai đoạn. Điều này giúp đảm bảo rằng composite của bạn sẽ đại diện tốt hơn cho điều kiện thực tế của cây trồng hoặc bề mặt đất trong suốt cả năm, đặc biệt là khi có nhiều ảnh bị che phủ bởi mây hoặc có chất lượng kém.


```python
def generate_yearly_qualitymosaic_composite(col):
    """Generate yearly composite by taking the mean value of images in each year.

    Args:
        col (ee.ImageCollection): The input image collection.

    Returns:
        ee.ImageCollection: The yearly composite image collection.
    """
    first_date, latest_date = date_range_col(col)
    yearly_list = yearly_datetime_list(first_date, latest_date)

    def yearly_composite(date):
        date = ee.Date(date)
        start = date
        end = date.advance(1, 'year')
        yearly_col = col.filterDate(start, end)
        composite = yearly_col.qualityMosaic('NDVI').set('system:time_start', start.millis())
        return composite

    yearly_composite_col = ee.ImageCollection(yearly_list.map(yearly_composite))
    return yearly_composite_col
ls8col_yearly_qualitymosaic_composite = generate_yearly_qualitymosaic_composite(ls8col_ndvi)
print('Số lượng ảnh sau khi tổng hợp theo năm bằng qualityMosaic:', ls8col_yearly_qualitymosaic_composite.size().getInfo())
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>



    Số lượng ảnh sau khi tổng hợp theo năm bằng qualityMosaic: 3
    

## 24.5. Tổng hợp ảnh ERA-5 Land theo thời gian

ERA5 là tập dữ liệu **tái phân tích khí hậu** (reanalysis) của ECMWF - không phải quan trắc trực tiếp mà là kết hợp mô hình khí hậu + quan trắc thực địa. Có rất phiên bản sản phẩm khác nhau của ERA-5, bạn có tham khảo tại [ECMWF](https://cds.climate.copernicus.eu/datasets/reanalysis-era5-land?tab=overview) và [GEE](https://developers.google.com/earth-engine/datasets/tags/era5-land).

### 24.5.1. Tổng hợp ảnh ERA5-Land theo năm


```python
# Ảnh era5-land theo giờ từ 1950 đến nay
era5land = ee.ImageCollection("ECMWF/ERA5_LAND/HOURLY").filterBounds(roi).filterDate(start_date, end_date).select(['temperature_2m'])
print('Số lượng ảnh ERA5 Land:', era5land.size().getInfo())
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>



    Số lượng ảnh ERA5 Land: 20448
    


```python
# Tổng hợp ảnh theo công thức bên trên
era5land_yearly_composite = generate_yearly_composite(era5land)
print(f"Số lượng ảnh theo năm {era5land_yearly_composite.size().getInfo()}")
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>



    Số lượng ảnh theo năm 3
    

### 24.5.2. Tổng hợp ảnh ERA5-Land theo tháng


```python
# Ta cũng có thể sử dụng bằng cách loop qua từng tháng và tính monthly composite. Tuy nhiên, cách này sẽ chậm hơn so với sử dụng hàm map ở trên. Ta có thể thêm một thuộc tính 'valid' để đếm số lượng ảnh gốc trong mỗi composite, sau đó lọc ra những composite có ít nhất 1 ảnh gốc để so sánh với kết quả ở trên.
import calendar 
mlist = []

for y in range(2023, 2026):
    for m in range(1, 13):
        start = f"{y}-{m:02}-01"
        ndays = calendar.monthrange(y, m)[1]
        end = f"{y}-{m:02}-{ndays}"
        subcol = era5land.filterDate(start, end)
        mean_img = subcol.mean().set({'system:time_start': ee.Date(start).millis(), 'valid': subcol.size()}) # Thêm thuộc tính 'valid' để đếm số lượng ảnh gốc trong mỗi composite
        mlist.append(mean_img)
# Tạo ImageCollection từ danh sách các composite theo tháng
era5land_monthly_composite_loop = ee.ImageCollection(mlist)
# giữ lại những composite có ít nhất 1 ảnh gốc vì có thể có những tháng không có ảnh nào trong bộ sưu tập gốc, dẫn đến composite có giá trị null. Việc lọc ra những composite này sẽ giúp chúng ta so sánh chính xác hơn với kết quả sử dụng hàm map ở trên.
era5land_monthly_composite_loop = era5land_monthly_composite_loop.filter(ee.Filter.gt('valid', 0))
print(f"Số lượng ảnh theo tháng (loop): {era5land_monthly_composite_loop.size().getInfo()}")
```



<style>
    .geemap-dark {
        --jp-widgets-color: white;
        --jp-widgets-label-color: white;
        --jp-ui-font-color1: white;
        --jp-layout-color2: #454545;
        background-color: #383838;
    }

    .geemap-dark .jupyter-button {
        --jp-layout-color3: #383838;
    }

    .geemap-colab {
        background-color: var(--colab-primary-surface-color, white);
    }

    .geemap-colab .jupyter-button {
        --jp-layout-color3: var(--colab-primary-surface-color, white);
    }
</style>



    Số lượng ảnh theo tháng (loop): 28
    

## Tóm tắt

Bạn đã hoàn thành Bài 24 và nắm vững kỹ thuật **tổng hợp ảnh theo giai đoạn thời gian (temporal compositing)** - bước quan trọng giúp giảm nhiễu, hạn chế ảnh hưởng của mây và tạo ra dữ liệu đầu vào ổn định cho các phân tích không gian trên Google Earth Engine.

### Các khái niệm chính đã nắm vững:
- ✅ Phân biệt các phương pháp tổng hợp: **median**, **mean**, **mosaic**, **percentile**, **qualityMosaic** và trường hợp sử dụng phù hợp
- ✅ Xây dựng danh sách mốc thời gian theo tháng và năm với `ee.List.sequence` và `ee.Date.advance`
- ✅ Tổng hợp ảnh Sentinel-2 theo **tháng**, **mùa** và **năm** sử dụng `.mean()`
- ✅ Tổng hợp ảnh Landsat theo năm sử dụng `.qualityMosaic('NDVI')` để ưu tiên pixel có thực vật khỏe mạnh nhất
- ✅ Tổng hợp ảnh ERA5-Land theo năm và tháng bằng cả hai cách: `map` (server-side) và `loop` (client-side)

### Kỹ năng bạn có thể áp dụng:
- Viết hàm tổng hợp ảnh theo tháng, mùa, năm tái sử dụng được cho bất kỳ `ImageCollection` nào
- Lựa chọn phương pháp tổng hợp phù hợp: `median` cho dữ liệu quang học còn mây, `qualityMosaic` để ưu tiên pixel chất lượng cao, `mean` cho dữ liệu đã làm sạch
- Lọc ảnh theo mùa với `ee.Filter.calendarRange` để trích xuất đặc trưng mùa vụ
- Thêm thuộc tính `valid` để kiểm tra và lọc các composite không có ảnh gốc
- Áp dụng pipeline tổng hợp này làm đầu vào cho phân tích chuỗi thời gian, phát hiện biến động đất đai và tính chỉ số thực vật ở các bài tiếp theo

