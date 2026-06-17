# Bài 21: Thao tác cơ bản trên ảnh trong GEE

Bài này tập trung vào các thao tác **cơ bản** trực tiếp trên đối tượng ảnh `ee.Image` - những bước nền tảng cần nắm vững trước khi đi vào phân tích nâng cao. 

## 21.1. Mục tiêu bài học

Sau khi hoàn thành bài học này, bạn sẽ có thể:

- Load ảnh từ `ImageCollection` và đọc metadata
- Chọn và đổi tên bands với `.select()` / `.rename()`
- Clip ảnh theo AOI với `.clip()`
- Áp dụng scale factor để chuyển giá trị pixel về đơn vị vật lý
- Thực hiện phép toán pixel-wise với `.add()`, `.multiply()`, `.expression()`
- Tính thống kê vùng với `reduceRegion()`
- Lấy giá trị pixel tại các điểm mẫu với `sampleRegions()`
- Reproject/Resample ảnh sang CRS và scale khác


```python
import ee
import geemap
ee.Authenticate() # xác thực tài khoản Google Earth Engine
ee.Initialize(project='ee-tuyenrss') # khởi tạo thư viện Earth Engine
```

Trong bài học này, chúng ta sẽ chọn khu vực nghiên cứu là bounding box khu vực Hà Nội. Bạn có thể thay đổi bằng khu vực khác, nhưng kết quả trả về có thể khác vì mỗi nơi sẽ có mây phủ khác nhau vào các thời gian khác nhau.


```python
# AOI: Hà Nội cho toàn bộ phân tích
aoi = ee.Geometry.BBox(105.7, 20.9, 106.0, 21.2)
```

## 21.2. Đọc và lọc ảnh 

Trong Google Earth Engine, chúng ta thường làm việc với từng ảnh riêng lẻ bên trong `ImageCollection`. Vì vậy, bước đầu tiên là áp dụng các bộ lọc theo vị trí, thời gian và thuộc tính để chọn ra những ảnh phù hợp với mục đích phân tích.

Trong ví dụ dưới đây, ta mong muốn tìm kiếm tất cả các bức ảnh Sentinel-2 cho khu vực AOI (Hà Nội) trong khoảng thoài gian 2026 và có độ mây phủ dưới 20%. 

### 21.2.1. Đọc và lọc ảnh


```python
# Load Sentinel-2 Surface Reflectance cho Hà Nội năm 2026 
START_DATE = '2026-01-01'
END_DATE = '2026-12-31'
sen2col = (ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
          .filterDate(START_DATE, END_DATE)
          .filterBounds(aoi)
          .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20))
          .sort('CLOUDY_PIXEL_PERCENTAGE'))   # Ít mây lên đầu

n = sen2col.size().getInfo()
print(f"Số ảnh tìm được: {n}")
# Lấy bước ảnh ít mấy nhất 
cloudless = ee.Image(sen2col.first())
print(f"Ngày chụp ảnh ít mây nhất: {cloudless.date().format('YYYY-MM-dd').getInfo()}")
```

### 21.2.2. Hiển thị ảnh

Trong ví dụ này, ta sử dụng gói python geemap để hiển thị ảnh Sentinel-2 ít mây nhất trên bản đồ. Để hiển thị ảnh trên bản đồ, ta cần xác định các tham số hiển thị (vis_params) như băng sóng nào sẽ được sử dụng để hiển thị (trong trường hợp này là B4, B3, B2 tương ứng với RGB), giá trị tối thiểu và tối đa để hiển thị, và gamma để điều chỉnh độ sáng của ảnh. Sau đó, ta tạo một đối tượng Map và thêm lớp ảnh vào bản đồ với các tham số hiển thị đã định nghĩa. Cuối cùng, ta trung tâm hóa bản đồ vào khu vực Hà Nội với mức zoom phù hợp để có thể quan sát rõ ràng.


```python
vis_params = {
    'bands': ['B4', 'B3', 'B2'],  # RGB
    'min': -80,
    'max': 1400,
    'gamma': 1
}
Map = geemap.Map()
Map.addLayer(cloudless, vis_params, 'Sentinel-2 Cloudless')
Map.centerObject(aoi, 10)
Map
```

## 21.3. Thêm bands, chọn, và đổi tên bands

Trong GEE, `.select()` được dùng để chọn các band cần thiết, còn `.rename()` giúp đặt lại tên band dễ hiểu hơn. Trong trường hợp thêm band, ta dùng `.addBand()`.

### 21.3.1. Chọn và đổi tên

Khi làm việc với ảnh vệ tinh, chúng ta thường chỉ quan tâm đến một số band nhất định. Ví dụ, để tính toán chỉ số NDVI, chúng ta chỉ cần band Red và NIR. Do đó, việc chọn bands cụ thể và đặt tên thân thiện sẽ giúp chúng ta dễ dàng thao tác và hiểu rõ hơn về dữ liệu mà chúng ta đang sử dụng.

Trong ví dụ dưới đây, chúng ta sẽ chọn ra 6 bands từ hơn 10 bands ảnh Sentinel-2 và đặt tên lại cho chúng tương ứng.


```python
# Bands Sentinel-2 gốc → tên thân thiện
S2_BANDS  = ['B2',   'B3',    'B4',  'B8',  'B11',   'B12'  ]
NEW_NAMES = ['Blue', 'Green', 'Red', 'NIR', 'SWIR1', 'SWIR2']
print(f"Các bands gốc: {cloudless.bandNames().getInfo()}")
# Chọn các band cần thiết và rename trong một bước
img_selected_band = cloudless.select(S2_BANDS).rename(NEW_NAMES)
print(f"\nSau select + rename: {img_selected_band.bandNames().getInfo()}")
```

### 21.3.2. Thêm bands vào ảnh

Nhiều khi một chúng ta tạo ra một band mới như NDVI, chúng ta có thể thêm chúng vào ảnh và đặt tên là NDVI như ví dụ bên dưới.


```python
# Tính NDVI
ndvi = img_selected_band.normalizedDifference(['NIR', 'Red']).rename('NDVI')
img_selected_band = img_selected_band.addBands(ndvi)
print(f"\nSau khi thêm band NDVI: {img_selected_band.bandNames().getInfo()}")
```

# 21.4. Clip ảnh theo vùng nghiên cứu

### 21.4.1. Clip ảnh theo vùng nghiên cứu

Đa số chúng ta sẽ làm việc với một vùng nghiên cứu cụ thể thay vì toàn bộ thế giới. Do đó, việc cắt (clip) ảnh về vùng nghiên cứu sẽ giúp giảm kích thước dữ liệu và tăng tốc độ xử lý. Trong ví dụ này, chúng ta đã cắt ảnh Sentinel-2 về khu vực Hà Nội để tập trung vào phân tích trong khu vực này. Để cắt ảnh theo vùng nghiên cứu, chúng ta sử dụng `.clip(geometry)` và hiển thị chúng. 


```python
# Clip ảnh đã select/rename về AOI Hà Nội
img_clip = cloudless.clip(aoi)

Map = geemap.Map()
Map.addLayer(img_clip, vis_params, 'Sentinel-2 Clipped')
Map.centerObject(aoi, 10)
Map
```

### 21.4.2. Chuyển đổi giá trị Pixel
Trong Google Earth Engine (GEE), giá trị pixel thường được lưu dưới dạng số nguyên (integer) để tối ưu dung lượng lưu trữ và hiệu năng xử lý. Để chuyển từ dữ liệu số nguyên, ta thường phải nhân với một hệ số nào đó. Bảng dưới đây là ví dụ cho ba bộ sưu tập và hệ số (scaling factor) thường được tìm thấy trong thông tin bands của dữ liệu (ví dụ [Sentinel-2](https://developers.google.com/earth-engine/datasets/catalog/COPERNICUS_S2_SR_HARMONIZED)).

| Collection | Scale | Offset | Range thực |
|---|---|---|---|
| Sentinel-2 SR `S2_SR_HARMONIZED` | × 0.0001 | 0 | 0.0 – 1.0 |
| Landsat 8/9 SR `LANDSAT/LC09/C02/T1_L2` | × 0.0000275 | −0.2 | −0.2 – 1.6 |
| MODIS `MOD09GA` | × 0.0001 | 0 | 0.0 – 1.0 |


```python
sen2band = cloudless.select(['B2', 'B3', 'B4', 'B8', 'B11', 'B12'])
# Áp dụng scale factor: nhân tất cả bands lựa chọn × 0.0001
sen2scale = sen2band.multiply(0.0001)
```

## 21.5. Toán học cơ bản trên ảnh

GEE hỗ trợ phép tính **pixel-wise** trực tiếp trên `ee.Image`. Có hai cách thường được sử dụng:
- **Methods**: `.add()`, `.subtract()`, `.multiply()`, `.divide()`, `.abs()`, `.sqrt()`, `.log()`, `.gt()`, `.lt()`, `.where()`
  
- **`.expression()`**: Viết công thức dạng chuỗi - linh hoạt hơn cho công thức phức tạp
- Tính toán hoàn toàn dựa vào server-side hoặc hàm đã được GEE team tạo ra. Không được sử dụng cả hàm python local với server-side.


```python
# Chọn các bands sau đây
blue  = sen2scale.select('B2')
green = sen2scale.select('B3')
red   = sen2scale.select('B4')
nir   = sen2scale.select('B8')
swir1 = sen2scale.select('B11')
```

### 21.5.1. Phép cộng

Trong GEE, các tính toán phải được thực hiện trên server-side, nghĩa là chúng ta không thể lấy giá trị pixel về máy tính để tính toán như bình thường. Thay vào đó, chúng ta phải sử dụng các phương thức của GEE để thực hiện các phép toán trên đối tượng ảnh. Trong ví dụ này, chúng ta tính toán độ sáng (Brightness) bằng cách lấy trung bình của ba bands nhìn thấy (Blue, Green, Red) và sử dụng phương thức add và divide để thực hiện phép tính trên server-side.


```python
# Brightness = trung bình 3 bands nhìn thấy
brightness = blue.add(green).add(red).divide(3).rename('Brightness')
```

### 21.5.2. Phép chia

Tương tự vậy, chúng ta có thể thực hiện các phép toán học khác nhau trên các bands để tạo ra các chỉ số mới. Ví dụ, chúng ta tính band ratio giữa NIR và Red.


```python
# Band ratio đơn giản
nir_red_ratio = nir.divide(red).rename('NIR_Red_Ratio') 
```

### 21.5.3. Phép nhân


```python
# Nhân band Red với band Green, sau đó nhân tiếp với 2
multiply = red.multiply(green).multiply(2)
```

### 21.5.4. Phép trừ


```python
# Trừ band Green từ band Red, kết quả đổi tên thành Red_minus_Green
subtract = red.subtract(green).rename('Red_minus_Green')
```

### 21.5.5. Phép so sánh

Phép toán so sánh sẽ trả về một ảnh nhị phân (binary image) với giá trị 1 ở những pixel thoải mãn điều kiện và 0 ở những pixel không thoả mãn điều kiện. Chúng rất hữu ích trong nhiều trường hợp khác nhau, chúng ta sẽ tìm hiểu một vài ví dụ dưới đây.

- **So sánh lớn hơn**

Ví dụ ta muốn phân loại NDVI thành hai lớp: cây xanh và không phải cây xanh. Giả sử chúng ta xác định NDVI > 0.3 được coi là cây xanh, còn lại là không phải cây xanh. Ta có thể sử dụng toán tử so sánh để tạo ra một mask nhị phân như bên dưới.


```python
# Toán tử so sánh (.gt, .lt, .gte, .lte, .eq) → trả về 0/1 
ndvi = nir.subtract(red).divide(nir.add(red)).rename('NDVI')
ndvi_mask = ndvi.gt(0.3)  # NDVI > 0.3 → có thực vật
```

- **So sánh bằng**

Giả sử chúng ta muốn xác định những khu vực là vùng nước mặt từ bản đồ sử dụng đất. Nếu chúng ta có một ảnh phân loại sử dụng đất, trong đó các pixel có giá trị 1 đại diện cho nước và các giá trị khác đại diện cho các loại đất khác, chúng ta có thể tạo một mask để chỉ giữ lại những pixel là nước mặt.


```python
# Bằng thì trả về 1, ngược lại 0
# Nếu bạn muốn biết từng loại đất, tham khảo thêm ở đây https://developers.google.com/earth-engine/datasets/catalog/MODIS_061_MCD12Q1
modis_landcover = ee.ImageCollection("MODIS/061/MCD12Q1").filterBounds(aoi).select('LC_Type1').first()
water_mask = modis_landcover.eq(17).clip(aoi)  # LC_Type1 = 17 → nước. Đọc thêm về các loại đất ở đây https://developers.google.com/earth-engine/datasets/catalog/MODIS_061_MCD12Q1#bands

Map = geemap.Map()
Map.centerObject(aoi, 10)
Map.addLayer(water_mask)
Map
```

- **So sánh nhỏ hơn**

Giả sử chúng ta muốn tạo ra một mask nơi mà tất cả khu vực không phải là nước sẽ có giá trị 1, còn khu vực nước sẽ có giá trị 0. Chúng ta có thể sử dụng dữ liệu MODIS Land Cover để tạo ra mask này bằng cách sử dụng toán tử so sánh lte để kiểm tra xem loại đất có phải là nước hay không.


```python
# Nhỏ hơn hoặc bằng 0, trả về 1 nếu thỏa, ngược lại 0
non_water_mask = modis_landcover.lte(16).rename('non_water_mask')  # LC_Type1 <= 16 → không phải nước
```

### 21.5.6. Sử dụng `expression`

Trong một số trường hợp, bạn có thể cần sử dụng biểu thức tùy chỉnh để tính toán các chỉ số phức tạp hơn. GEE cung cấp phương thức expression cho phép bạn viết công thức toán học dưới dạng chuỗi, sử dụng tên band đã chọn. Ví dụ, chúng ta đã tính chỉ số nước MNDWI bằng cách sử dụng biểu thức với các band Green và SWIR1


```python
# Chí số nước MNDWI = (Green - SWIR1) / (Green + SWIR1)
mndwi = cloudless.expression(
    '(GREEN - SWIR1) / (GREEN + SWIR1)',
    {'GREEN': cloudless.select('B3'), 'SWIR1': cloudless.select('B11')}
).rename('MNDWI')
print(f"Band MNDWI: {mndwi.bandNames().getInfo()}")
```

## 21.6. Reproject, Resample và xem Thumbnail ảnh nhanh

- **`.reproject(crs, scale)`**: Chuyển đổi CRS và/hoặc scale (độ phân giải pixel)
  
- **`.resample(method)`**: Thay đổi thuật toán nội suy khi reproject - `'bilinear'` cho dữ liệu liên tục, `'near'` cho categorical (classification maps)

### 21.6.1. Reproject ảnh và resample

Dữ liệu vệ tinh thường có độ phân giải khác nhau giữa các bands, vì vậy khi làm việc với nhiều bands mà có độ phân giải khác nhau, bạn cần chú ý đến việc resample hoặc reproject để đảm bảo các bands có cùng độ phân giải và hệ tọa độ trước khi thực hiện các phép toán giữa chúng.


```python
# Trong Sentinel-2, có một vài bands có độ phân giải 20m (ví dụ B11, B12).
print(f"Độ phân giải của band B11: {cloudless.select('B11').projection().nominalScale().getInfo()} mét")
# Hiện thị cả crs và transform của band B11
print(f"CRS của band B11: {cloudless.select('B11').projection().crs().getInfo()}")
print(f"Transform của band B11: {cloudless.select('B11').projection().transform().getInfo()}")
```


```python
# Resample band 11 về 10m để khớp với các bands khác
resampled_b11 = cloudless.select('B11').resample('bilinear').reproject(
    crs=cloudless.select('B4').projection()
).rename('B11_resampled')
print(f"Độ phân giải của band B11 sau khi resample: {resampled_b11.projection().nominalScale().getInfo()} mét")
```

### 21.6.2. Xem Thumbnail ảnh nhanh

`.getThumbURL()` trả về URL ảnh thu nhỏ — hữu ích để kiểm tra nhanh kết quả mà không cần render bản đồ đầy đủ.


```python
import IPython.display as ipd

def show_thumbnail(image, vis_params, title='', width=450):
    """Hiển thị thumbnail GEE image trong notebook."""
    params = {**vis_params, 'dimensions': 512, 'region': aoi, 'format': 'png'}
    url = image.getThumbURL(params)
    # print(f"{title}  →  {url[:70]}...")
    return ipd.Image(url=url, width=width)

# True Color thumbnail
plot = show_thumbnail(cloudless, {'bands': ['B4', 'B3', 'B2'], 'min': 0.0, 'max': 1300, 'gamma': 1.4},
               title='True Color')
# Lưu thumbnail vào file
# plot.save('sentinel2_true_color_thumbnail.png')
```

## Tóm tắt

Bạn đã nắm vững tất cả thao tác cơ bản trên `ee.Image` - nền tảng cho mọi phân tích nâng cao trong GEE.

### Các thao tác đã học:

| Thao tác | Method | Ghi chú |
|---|---|---|
| Load ảnh tốt nhất | `.sort().first()` | Lọc theo % mây |
| Chọn bands | `.select(bands).rename(names)` | Chuẩn bị trước tính toán |
| Clip theo AOI | `.clip(geometry)` | Luôn clip để tăng tốc (vùng nhỏ) |
| Scale factor | `.multiply(0.0001)` | S2: ×0.0001, L8: ×0.0000275−0.2 |
| Toán học pixel-wise | `.add()`, `.divide()`, `.expression()` | Lazy, server-side |
| Reproject | `.reproject(crs, scale)` | UTM cho tính toán metric |

### Kỹ năng bạn có thể áp dụng:
- Tải và lọc ảnh Sentinel-2 (hoặc Landsat, MODIS) chất lượng cao cho bất kỳ khu vực và khoảng thời gian nào
- Chuẩn hóa dữ liệu đầu vào: chọn đúng bands, đổi tên và áp dụng scale factor trước khi tính toán
- Tính các chỉ số viễn thám tùy chỉnh (NDVI, MNDWI, ...) bằng `.expression()` hoặc phép toán trực tiếp
- Đồng nhất độ phân giải giữa các bands (ví dụ: đưa band 20m về 10m) để tránh lỗi khi kết hợp bands
- Kiểm tra nhanh kết quả xử lý ảnh bằng thumbnail mà không cần render bản đồ interactive
- Đặt nền tảng vững chắc để tiếp tục với cloud masking, tính chỉ số thực vật và tổng hợp ảnh composite ở các bài tiếp theo
