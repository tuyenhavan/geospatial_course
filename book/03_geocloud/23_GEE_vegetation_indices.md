# Bài 23: Tính toán chỉ số thực vật (GEE)

Chỉ số thực vật (vegetation indices) được tính từ tổ hợp các bands phổ - đặc biệt là `NIR` và `Red` - phản ánh trạng thái sức khoẻ và mật độ thực vật. Bài này trình bày cách tính các chỉ số phổ biến nhất trên ba nguồn dữ liệu chính trong GEE như Sentinel-2, Landsat, MODIS.

## 23.1. Mục tiêu học tập

Sau bài này bạn có thể:

- Giải thích công thức và ý nghĩa của NDVI, EVI, SAVI, NDWI, NBR, BSI.
- Tính các chỉ số thực vật từ Sentinel-2 (đã cloud mask).
- Tính các chỉ số thực vật từ Landsat 8/9 Collection 2.
- Truy cập sản phẩm NDVI/EVI sẵn có của MODIS.


```python
import ee
import geemap

ee.Initialize()
ee.Authenticate()
```




    True



Trong bài học này, chúng ta sẽ chọn khu vực nghiên cứu theo bounding bên dưới và khoảng thời gian từ tháng 6 đến tháng 9 năm 2025. Bạn có thể thay đổi vị trí và thời gian phù hợp với yêu cầu của bạn.


```python
# Bounding box cho vùng nghiên cứu ở Đức
bbox = [9.84375   , 47.5172007 , 10.1953125 , 47.75409798]
# Tạo một đối tượng hình chữ nhật từ bounding box
roi = ee.Geometry.Rectangle(bbox)
# Xác định khoảng thời gian cho bộ dữ liệu Sentinel-2
start_date = '2025-06-01'
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



## 23.2. Tổng quan các chỉ số thực vật

Các chỉ số thực vật là các đại lượng được tính toán từ dữ liệu phản xạ phổ của ảnh viễn thám nhằm đánh giá đặc trưng và tình trạng của thảm thực vật. Chúng dựa trên sự khác biệt phản xạ giữa các kênh phổ, đặc biệt là vùng đỏ (Red) và cận hồng ngoại (NIR). Các chỉ số này được sử dụng rộng rãi trong giám sát sinh trưởng cây trồng, tài nguyên và môi trường. Nếu bạn quan tâm đến danh sách đầy đủ các chỉ số thực vật, bạn có thể tham khảo tại [đây](https://www.indexdatabase.de/db/i.php). Dưới đây là một số chỉ số phổ biến và ý nghĩa của chúng.

| Chỉ số | Công thức | Khoảng giá trị | Ứng dụng |
|--------|-----------|---------------|----------|
| **NDVI** | $\frac{NIR - Red}{NIR + Red}$ | −1 → 1 | Đánh giá mật độ và sức khỏe thực vật. Giá trị càng cao cho thấy thảm thực vật xanh tốt, phát triển mạnh; giá trị thấp phản ánh đất trống, nước hoặc thực vật suy giảm. |
| **EVI** | $2.5 \times \frac{NIR - Red}{NIR + 6 \cdot Red - 7.5 \cdot Blue + 1}$ | −1 → 1 | Phân tích khu vực có thực vật dày đặc, giảm ảnh hưởng khí quyển và hiện tượng bão hòa phổ. Giá trị cao thể hiện sinh khối và độ xanh lớn. |
| **SAVI** | $\frac{(NIR - Red)(1 + L)}{NIR + Red + L}$, $L=0.5$ | −1.5 → 1.5 | Thích hợp cho khu vực có thực vật thưa hoặc nền đất trống. Giá trị cao cho thấy mức độ che phủ thực vật tốt. |
| **NDWI** | $\frac{Green - NIR}{Green + NIR}$ | −1 → 1 | Xác định hàm lượng nước trong thực vật hoặc phát hiện mặt nước. Giá trị cao phản ánh độ ẩm lớn hoặc khu vực chứa nước. |

## 23.3. Chỉ số thực vật từ Sentinel-2

Ảnh Sentinel-2 cung cấp dữ liệu đa phổ có độ phân giải cao, cho phép tính toán hiệu quả các chỉ số liên quan đến thảm thực vật và hỗ trợ giám sát đặc điểm sinh trưởng, độ che phủ cũng như tình trạng của cây trồng.

### 23.3.1. Chuẩn bị dự liệu Sentinel-2

Đoạn mã sau thực hiện tiền xử lý ảnh Sentinel-2 trước khi tính toán các chỉ số thực vật. Quá trình này bao gồm loại bỏ mây và bóng mây dựa trên band SCL, chuyển giá trị ảnh về phản xạ bề mặt (reflectance), đồng thời đổi tên các kênh phổ để thuận tiện cho việc phân tích và tính toán sau này.


```python
# Hàm chuẩn bị dữ liệu Sentinel-2: mask mây, scale, đổi tên bands
def prepare_sen2data(image):
    """Mask mây bằng SCL, scale, đổi tên bands."""
    scl  = image.select('SCL')
    mask = scl.eq(3).Or(scl.eq(8)).Or(scl.eq(9)).Or(scl.eq(10)).Not()
    return (image
            .updateMask(mask) # Loại bỏ mây và bóng mây dựa trên SCL
            .divide(10000) # Sentinel-2 SR đã được scale sẵn, chỉ cần chia cho 10000 để về reflectance
            .select(['B2','B3','B4','B8','B8A','B11','B12'],
                    ['Blue','Green','Red','NIR','NIR8A','SWIR1','SWIR2'])
            .copyProperties(image, ['system:time_start']))
# Lấy bộ sưu tập Sentinel-2, áp dụng hàm chuẩn bị dữ liệu
sen2col = (ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
          .filterBounds(roi)
          .filterDate(start_date, end_date)
          .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 25)))
sen2col = sen2col.map(prepare_sen2data)
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



### 23.3.2. Tính chỉ số NDVI Sentinel-2

Chỉ số NDVI là chỉ số dùng để đánh giá mức độ xanh và sức khỏe của thảm thực vật từ ảnh vệ tinh. Giá trị NDVI cao cho thấy cây cối phát triển tốt, trong khi giá trị thấp biểu thị đất trống, nước hoặc thực vật kém phát triển. Trong GEE, NDVI được tính từ kênh cận hồng ngoại và kênh đỏ. Có 2 hàm chính để tính chỉ số thực vật trong GEE là dùng `.normalizedDifference()` hoặc `.expression()`. 

#### 23.3.2.1. Tính chỉ số NDVI Sentinel-2 cho một bức ảnh

Trong GEE, tính chỉ số NDVI thường được thực hiện bằng cách sử dụng phương pháp `normalizedDifference`, vì nó đã được tối ưu hóa và dễ sử dụng. Tuy nhiên, bạn cũng có thể tính NDVI bằng cách sử dụng biểu thức `expression` nếu bạn muốn kiểm soát nhiều hơn về cách tính toán hoặc nếu bạn muốn kết hợp nhiều chỉ số khác nhau trong một biểu thức phức tạp hơn.


```python
# Viết hàm tính NDVI cho một bức ảnh Sentinel-2 dùng normalizedDifference 
def calculate_ndvi(image):
    """Tính NDVI từ ảnh Sentinel-2 đã chuẩn bị."""
    ndvi = image.normalizedDifference(['NIR', 'Red']).rename('NDVI')
    return image.addBands(ndvi)

# Hàm tính NDVI cho một bức ảnh Sentinel-2 dùng expression
def calculate_ndvi_expression(image):
    """Tính NDVI từ ảnh Sentinel-2 đã chuẩn bị dùng expression."""
    ndvi = image.expression(
        '(NIR - Red) / (NIR + Red)',
        {
            'NIR': image.select('NIR'),
            'Red': image.select('Red')
        }
    ).rename('NDVI')
    return image.addBands(ndvi)
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
# Lấy một bức ảnh Sentinel-2 đầu tiên từ bộ sưu tập và tính NDVI bằng cả hai cách
ndvi = calculate_ndvi(sen2col.first())
print('Có NDVI đã được thêm vào bức ảnh:', ndvi.bandNames().getInfo())
# Dùng expression 
ndvi = calculate_ndvi_expression(sen2col.first())
print('Có NDVI đã được thêm vào bức ảnh:', ndvi.bandNames().getInfo())
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



    Có NDVI đã được thêm vào bức ảnh: ['Blue', 'Green', 'Red', 'NIR', 'NIR8A', 'SWIR1', 'SWIR2', 'NDVI']
    Có NDVI đã được thêm vào bức ảnh: ['Blue', 'Green', 'Red', 'NIR', 'NIR8A', 'SWIR1', 'SWIR2', 'NDVI']
    

#### 23.3.2.2. Tính chỉ số NDVI Sentinel-2 cho nhiều bức ảnh

Để tính chỉ số NDVI cho tất cả các bức ảnh trong `sen2col`, ta dùng hàm `.map()` để duyệt qua các bức ảnh và tính toán chỉ số NDVI và trả về một dữ liệu mới có thêm band NDVI.


```python
sen2col_ndvi = sen2col.map(calculate_ndvi)
print('Bộ sưu tập Sentinel-2 với NDVI có các bands:', sen2col_ndvi.first().bandNames().getInfo())
print(f"Số lượng ảnh trong bộ sưu tập sau khi tính NDVI: {sen2col_ndvi.size().getInfo()}")
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



    Bộ sưu tập Sentinel-2 với NDVI có các bands: ['Blue', 'Green', 'Red', 'NIR', 'NIR8A', 'SWIR1', 'SWIR2', 'NDVI']
    Số lượng ảnh trong bộ sưu tập sau khi tính NDVI: 7
    


```python
# Tương tự như vậy, ta có thể dùng calculate_ndvi_expression để tính NDVI cho toàn bộ bộ sưu tập nếu muốn.
sen2col_ndvi = sen2col.map(calculate_ndvi_expression)
print('Bộ sưu tập Sentinel-2 với NDVI (dùng expression) có các bands:', sen2col_ndvi.first().bandNames().getInfo())
print(f"Số lượng ảnh trong bộ sưu tập sau khi tính NDVI (dùng expression): {sen2col_ndvi.size().getInfo()}")
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



    Bộ sưu tập Sentinel-2 với NDVI (dùng expression) có các bands: ['Blue', 'Green', 'Red', 'NIR', 'NIR8A', 'SWIR1', 'SWIR2', 'NDVI']
    Số lượng ảnh trong bộ sưu tập sau khi tính NDVI (dùng expression): 7
    

### 23.3.3. Tính nhiều chỉ số với ảnh Sentinel-2

Tương tự như ví dụ trên, nhưng thay vì xây dựng một hàm cho từng chỉ số riêng lẻ, ta có thể tính đồng thời nhiều chỉ số thực vật cho mỗi ảnh trong cùng một hàm để tối ưu quy trình xử lý.


```python
def calculate_sen2indices(image):
    """
    Calculate multiple vegetation indices for a Sentinel-2 image.
    This function computes EVI, SAVI, and NDWI using the appropriate bands.
    Args:
        image (ee.Image): A Sentinel-2 image with bands named 'Red', 'Green', 'Blue', 'NIR', etc.
    Returns:
        ee.Image: The input image with added bands for EVI, SAVI, and NDWI.

    """
    # Tính EVI - Enhanced Vegetation Index
    # Phương pháp 2: .expression() — linh hoạt cho công thức phức tạp
    evi = image.expression(
        '2.5 * (NIR - RED) / (NIR + 6.0 * RED - 7.5 * BLUE + 1.0)',
        {'NIR': image.select('NIR'), 'RED': image.select('Red'),
         'BLUE': image.select('Blue')}
    ).rename('EVI')

    # SAVI - Soil Adjusted Vegetation Index (L=0.5)
    savi = image.expression(
        '((NIR - RED) / (NIR + RED + 0.5)) * 1.5',
        {'NIR': image.select('NIR'), 'RED': image.select('Red')}
    ).rename('SAVI')

    # NDWI - Normalized Difference Water Index (thực vật → hàm lượng nước)
    ndwi = image.normalizedDifference(['Green', 'NIR']).rename('NDWI')
    return image.addBands(evi).addBands(savi).addBands(ndwi)


# Áp dụng lên toàn collection
sen2col_indices = sen2col.map(calculate_sen2indices)

print('Bộ sưu tập Sentinel-2 với tất cả chỉ số có các bands:', sen2col_indices.first().bandNames().getInfo())
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



    Bộ sưu tập Sentinel-2 với tất cả chỉ số có các bands: ['Blue', 'Green', 'Red', 'NIR', 'NIR8A', 'SWIR1', 'SWIR2', 'EVI', 'SAVI', 'NDWI']
    

## 23.4. Tính chỉ số thực vật từ ảnh Landsat

### 23.4.1. Chuẩn bị dữ liệu ảnh Landsat

Trước khi tính toán các chỉ số từ ảnh Landsat, cần thực hiện bước tiền xử lý nhằm đảm bảo dữ liệu sạch và có độ tin cậy cao cho quá trình phân tích. Trong ví dụ này, dữ liệu được sử dụng là ảnh phản xạ bề mặt (Surface Reflectance – SR) từ vệ tinh Landsat 8/9. Đầu tiên, chúng ta tạo ra hàm để chuẩn bị dữ liệu bằng cách loại bỏ mây phủ.


```python
def prepare_landsat(image):
    """Cloud mask + scale factor + đổi tên bands cho Landsat 8/9 C2 L2."""
    
    qa  = image.select('QA_PIXEL')
    sat = image.select('QA_RADSAT')

    # Mask cloud, cloud shadow, cirrus, dilated cloud
    cloud_mask = (
        qa.bitwiseAnd(1 << 1).eq(0)  # Dilated cloud
        .And(qa.bitwiseAnd(1 << 2).eq(0))  # Cirrus
        .And(qa.bitwiseAnd(1 << 3).eq(0))  # Cloud
        .And(qa.bitwiseAnd(1 << 4).eq(0))  # Cloud shadow
    )

    # Mask saturated pixels
    sat_mask = sat.bitwiseAnd(0b01111110).eq(0)

    return (image
            .updateMask(cloud_mask.And(sat_mask))
            .multiply(0.0000275).add(-0.2)
            .select(['SR_B2','SR_B3','SR_B4','SR_B5','SR_B6','SR_B7'],
                    ['Blue','Green','Red','NIR','SWIR1','SWIR2'])
            .copyProperties(image, ['system:time_start']))
# Lấy bộ sưu tập Landsat 8/9, áp dụng hàm chuẩn bị dữ liệu
ls8col = (ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')
          .filterDate(start_date, end_date)
            .filterBounds(roi)
            .filter(ee.Filter.lt('CLOUD_COVER', 30)))

l9_col = (ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')
          .filterBounds(roi).filterDate(start_date, end_date)
          .filter(ee.Filter.lt('CLOUD_COVER', 30)))

landsat = ls8col.merge(l9_col).map(prepare_landsat) # Gộp 2 bộ sưu tập Landsat 8 và 9, áp dụng hàm chuẩn bị dữ liệu

print('Bộ sưu tập Landsat 8/9 đã chuẩn bị có các bands:', landsat.first().bandNames().getInfo())
print(f"Số lượng ảnh trong bộ sưu tập Landsat 8/9 sau khi chuẩn bị: {landsat.size().getInfo()}")
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



    Bộ sưu tập Landsat 8/9 đã chuẩn bị có các bands: ['Blue', 'Green', 'Red', 'NIR', 'SWIR1', 'SWIR2']
    Số lượng ảnh trong bộ sưu tập Landsat 8/9 sau khi chuẩn bị: 10
    

### 23.4.2. Tính chỉ số NDVI Landsat

Tương tự như với Sentinel-2, chúng ta xác định được `NIR` và `RED` bands trong bộ dữ liệu Landsat 8/9 và tính NDVI như bên dưới.

#### 23.4.2.1. Tính chỉ số NDVI Landsat cho một bức ảnh


```python
# Viết hàm tính NDVI cho một bức ảnh Landsat dùng normalizedDifference 
def calculate_ndvi(image):
    """Tính NDVI từ ảnh Landsat đã chuẩn bị."""
    ndvi = image.normalizedDifference(['NIR', 'Red']).rename('NDVI')
    return image.addBands(ndvi)
landsat_first_ndvi = calculate_ndvi(landsat.first())
print('Có NDVI đã được thêm vào bức ảnh Landsat:', landsat_first_ndvi.bandNames().getInfo())
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



    Có NDVI đã được thêm vào bức ảnh Landsat: ['Blue', 'Green', 'Red', 'NIR', 'SWIR1', 'SWIR2', 'NDVI']
    

#### 23.4.2.2. Tính chỉ số NDVI Landsat cho nhiều bức ảnh


```python
landsat_col_ndvi = landsat.map(calculate_ndvi)
print('Bộ sưu tập Landsat 8/9 với NDVI có các bands:', landsat_col_ndvi.first().bandNames().getInfo())
print(f"Số lượng ảnh trong bộ sưu tập Landsat 8/9 sau khi tính NDVI: {landsat_col_ndvi.size().getInfo()}")
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



    Bộ sưu tập Landsat 8/9 với NDVI có các bands: ['Blue', 'Green', 'Red', 'NIR', 'SWIR1', 'SWIR2', 'NDVI']
    Số lượng ảnh trong bộ sưu tập Landsat 8/9 sau khi tính NDVI: 10
    

### 23.4.3. Tính nhiều chỉ số với ảnh Landsat


```python
def calculate_landsat_indices(image):
    """
    Calculate multiple vegetation indices for a Landsat image.
    This function computes EVI, SAVI, and NDWI using the appropriate bands.
    Args:
        image (ee.Image): A Landsat image with bands named 'Red', 'Green', 'Blue', 'NIR', etc.
    Returns:
        ee.Image: The input image with added bands for EVI, SAVI, and NDWI.

    """
    # Tính EVI - Enhanced Vegetation Index
    # Phương pháp 2: .expression() — linh hoạt cho công thức phức tạp
    evi = image.expression(
        '2.5 * (NIR - RED) / (NIR + 6.0 * RED - 7.5 * BLUE + 1.0)',
        {'NIR': image.select('NIR'), 'RED': image.select('Red'),
         'BLUE': image.select('Blue')}
    ).rename('EVI')

    # SAVI - Soil Adjusted Vegetation Index (L=0.5)
    savi = image.expression(
        '((NIR - RED) / (NIR + RED + 0.5)) * 1.5',
        {'NIR': image.select('NIR'), 'RED': image.select('Red')}
    ).rename('SAVI')

    # NDWI - Normalized Difference Water Index (thực vật → hàm lượng nước)
    ndwi = image.normalizedDifference(['Green', 'NIR']).rename('NDWI')
    return image.addBands(evi).addBands(savi).addBands(ndwi)


# Áp dụng lên toàn collection
landsat_col_indices = landsat.map(calculate_landsat_indices).select(["EVI", "SAVI", "NDWI"])

print('Bộ sưu tập Landsat 8/9 với tất cả chỉ số có các bands:', landsat_col_indices.first().bandNames().getInfo())
print(f"Số lượng ảnh trong bộ sưu tập Landsat 8/9 sau khi tính tất cả chỉ số: {landsat_col_indices.size().getInfo()}")
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



    Bộ sưu tập Landsat 8/9 với tất cả chỉ số có các bands: ['EVI', 'SAVI', 'NDWI']
    Số lượng ảnh trong bộ sưu tập Landsat 8/9 sau khi tính tất cả chỉ số: 10
    

## 23.5. NDVI và EVI từ ảnh MODIS và VIIRS 

MODIS là nguồn dữ liệu viễn thám quan sát trái đất với tần suất cập nhật cao, phù hợp cho nghiên cứu biến động thảm thực vật theo thời gian. Khác với Sentinel-2 hay Landsat, MODIS cung cấp sẵn các sản phẩm chỉ số thực vật như NDVI và EVI đã được tiền xử lý, giúp giảm thời gian tính toán và thuận tiện cho phân tích chuỗi thời gian trên phạm vi lớn. Các sản phẩm chính của MODIS và VIIRS bao gồm:

| Sản phẩm | Độ phân giải | Chu kỳ | Bands có sẵn |
|----------|-------------|--------|-------------|
| **MOD13Q1** (Terra) | 250 m | 16 ngày | NDVI, EVI |
| **MYD13Q1** (Aqua) | 250 m | 16 ngày | NDVI, EVI |
| **MOD13A1** (Terra) | 500 m | 16 ngày | NDVI, EVI |
| **MYD13A1** (Aqua) | 500 m | 16 ngày | NDVI, EVI |
| **MOD13A2** (Terra) | 1 km | 16 ngày | NDVI, EVI |
| **MYD13A2** (Aqua) | 1 km | 16 ngày | NDVI, EVI |
| **VNP13A1** (VIIRS) | 500 m | 16 ngày | NDVI, EVI |

### 23.5.1. Chuẩn bị dữ liệu ảnh MODIS và VIIRS

Trước khi sử dụng NDVI hoặc EVI từ MODIS và VIIRS, cần loại bỏ các pixel bị mây và nhiễu khí quyển để đảm bảo độ chính xác. Dữ liệu cũng cần được chọn đúng sản phẩm và áp dụng hệ số scale theo quy định. Điều này giúp tăng độ tin cậy khi phân tích biến động thảm thực vật theo thời gian. Chúng ta chuẩn bị một hàm cho việc loại mây cho ảnh MODIS như bên dưới.


```python

def mask_cloud(col, from_bit, to_bit, QA_band, threshold=1):
    """Extract bitmask and apply cloud masking to an ImageCollection.
    Hàm này có tham khảo từ https://spatialthoughts.com/2021/08/19/qa-bands-bitmasks-gee/
    Args:
        col (ee.ImageCollection): The input image collection.
        from_bit (int): The starting bit position for the mask.
        to_bit (int): The ending bit position for the mask.
        QA_band (str): The name of the QA band.
        threshold (int, optional): The threshold for masking. Defaults to 1.

    Returns:
        ee.ImageCollection: The masked image collection.
    """
    
    def img_mask(img):
        qa = img.select(QA_band)

        # Extract bits directly (combine both functions)
        mask_size = ee.Number(to_bit).add(1).subtract(from_bit)
        mask = ee.Number(1).leftShift(mask_size).subtract(1)

        bitmask = qa.rightShift(from_bit).bitwiseAnd(mask)

        return img.updateMask(bitmask.lte(threshold))

    return col.map(img_mask)

modis = ee.ImageCollection("MODIS/061/MOD13A2").filterBounds(roi).filterDate(start_date, end_date)
modis = mask_cloud(modis, from_bit=0, to_bit=1, QA_band='DetailedQA', threshold=0)
print('Bộ sưu tập MODIS sau khi mask mây có các bands:', modis.first().bandNames().getInfo())
print(f"Số lượng ảnh trong bộ sưu tập MODIS sau khi mask mây: {modis.size().getInfo()}")
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



    Bộ sưu tập MODIS sau khi mask mây có các bands: ['NDVI', 'EVI', 'DetailedQA', 'sur_refl_b01', 'sur_refl_b02', 'sur_refl_b03', 'sur_refl_b07', 'ViewZenith', 'SolarZenith', 'RelativeAzimuth', 'DayOfYear', 'SummaryQA']
    Số lượng ảnh trong bộ sưu tập MODIS sau khi mask mây: 7
    

### 23.5.2. Sử dụng chỉ số NDVI và EVI từ ảnh MODIS


```python
modis_indices = modis.select(["NDVI", "EVI"])
# scale data MODIS về khoảng 0-1
modis_indices = modis_indices.map(lambda img: img.multiply(0.0001).copyProperties(img, ['system:time_start']))
print('Bộ sưu tập MODIS với NDVI và EVI có các bands:', modis_indices.first().bandNames().getInfo())
print(f"Số lượng ảnh trong bộ sưu tập MODIS sau khi tính chỉ số: {modis_indices.size().getInfo()}")
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



    Bộ sưu tập MODIS với NDVI và EVI có các bands: ['NDVI', 'EVI']
    Số lượng ảnh trong bộ sưu tập MODIS sau khi tính chỉ số: 7
    

### 23.5.3. Sử dụng chỉ số NDVI và EVI từ ảnh VIIRS


```python
# Ảnh có độ phân giải 500m và 16 ngày
viirs = ee.ImageCollection("NASA/VIIRS/002/VNP13A1").filterDate(start_date, end_date).filterBounds(roi)
# Mask mây cho VIIRS nếu cần thiết (tùy thuộc vào QA band của VIIRS)
viirs = mask_cloud(viirs, from_bit=0, to_bit=1, QA_band='VI_Quality', threshold=0)
print('Bộ sưu tập VIIRS sau khi mask mây có các bands:', viirs.first().bandNames().getInfo())
print(f"Số lượng ảnh trong bộ sưu tập VIIRS sau khi mask mây: {viirs.size().getInfo()}")
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



    Bộ sưu tập VIIRS sau khi mask mây có các bands: ['EVI', 'EVI2', 'NDVI', 'NIR_reflectance', 'SWIR1_reflectance', 'SWIR2_reflectance', 'SWIR3_reflectance', 'VI_Quality', 'red_reflectance', 'green_reflectance', 'blue_reflectance', 'composite_day_of_the_year', 'pixel_reliability', 'relative_azimuth_angle', 'sun_zenith_angle', 'view_zenith_angle']
    Số lượng ảnh trong bộ sưu tập VIIRS sau khi mask mây: 15
    


```python
# Chọn chỉ số NDVI và EVI từ VIIRS. Giá trị không cần scale vì VIIRS đã được scale sẵn về khoảng 0-1.
viirs_indices = viirs.select(["NDVI", "EVI"])
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



## Tóm tắt

Bạn đã hoàn thành Bài 23 và nắm vững cách **tính toán các chỉ số thực vật** từ nhiều nguồn dữ liệu vệ tinh trên Google Earth Engine.

### Các khái niệm chính đã nắm vững:
- ✅ Ý nghĩa và công thức của các chỉ số phổ biến: NDVI, EVI, SAVI, NDWI
- ✅ Tính chỉ số bằng `.normalizedDifference()` và `.expression()` cho công thức phức tạp hơn
- ✅ Thêm chỉ số vào ảnh với `.addBands()` và áp dụng lên toàn bộ dữ liệu (collection) với `.map()`
- ✅ Sử dụng sản phẩm NDVI/EVI có sẵn từ MODIS (MOD13A2) và VIIRS (VNP13A1)
- ✅ Áp dụng scale factor cho MODIS (×0.0001) và cloud mask bằng `DetailedQA` / `VI_Quality`.

### Kỹ năng bạn có thể áp dụng:
- Tính NDVI, EVI, SAVI, NDWI từ Sentinel-2 và Landsat 8/9 cho bất kỳ khu vực và thời gian nào
- Viết hàm tính nhiều chỉ số cùng lúc và áp dụng lên toàn `ImageCollection`
- Chọn đúng nguồn dữ liệu phù hợp với bài toán: Sentinel-2 (chi tiết 10m), Landsat (lịch sử dài), MODIS/VIIRS (tần suất cao, diện tích lớn)
- Đặt nền tảng để phân tích chuỗi thời gian thực vật, phát hiện hạn hán và giám sát biến động đất đai ở các bài tiếp theo
