# Bài 26: Xuất dữ liệu từ GEE

Google Earth Engine cho phép **export dữ liệu** ở ba dạng chính:
- **Image → Drive/Asset**: ảnh raster dưới dạng GeoTIFF
- **Table → Drive/Asset**: bảng dữ liệu dưới dạng CSV / GeoJSON / SHP
- **getDownloadURL**: tải trực tiếp ảnh nhỏ về máy (không cần task)

## 26.1. Mục tiêu học tập

Sau khi hoàn thành bài này, bạn có thể:

- Export ảnh (Image) ra **Google Drive** dưới dạng GeoTIFF có thể dùng trong QGIS/ArcGIS.
- Export ảnh hàng loạt theo tháng/năm bằng vòng lặp Python
- Export ảnh lên `GEE AssetAsset` để tái sử dụng và chia sẻ


```python
import ee
import geemap

ee.Initialize()
```

Trong bài học này, chúng ta sẽ chọn khu vực nghiên cứu theo bounding bên dưới và khoảng thời gian như bên dưới. Bạn có thể thay đổi vị trí và thời gian phù hợp với yêu cầu của bạn.


```python
# Bounding box cho vùng nghiên cứu ở Đức
bbox = [9.84375, 47.5172007 , 10.1953125 , 47.75409798]
# Tạo một đối tượng hình chữ nhật từ bounding box
roi = ee.Geometry.Rectangle(bbox)
# Xác định khoảng thời gian cho bộ dữ liệu Sentinel-2
start_date = '2025-06-01'
end_date = '2025-07-30'
```

## 26.2 Tải ảnh về Google Drive

Export image là thao tác phổ biến nhất: xuất kết quả phân tích thành **file GeoTIFF** để dùng trong QGIS, ArcGIS, hoặc Python (rasterio).

Trước tiên tạo ảnh S2 median composite làm dữ liệu mẫu:


```python
# Lấy bộ sưu tập Sentinel-2, áp dụng hàm chuẩn bị dữ liệu
sen2col = (ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
          .filterBounds(roi)
          .filterDate(start_date, end_date)
          .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 25))
          .select(['B2', 'B3', 'B4', 'B8'])
          )
print('Số lượng ảnh Sentinel-2:', sen2col.size().getInfo())
```

### 26.2.1. Tải một ảnh về Google Drive


```python
# Lấy một ảnh Sentinel-2 đầu tiên để tải ảnh về Google Drive cho vùng nghiên cứu roi
image = sen2col.first()
task = ee.batch.Export.image.toDrive(
    image=image.toFloat(), # ảnh cần tải về. Đảm bảo các bands cùng một loại float32. Bạn có thể chọn dtype khác phụ thuộc vào loại dữ liệu.
    description='Sentinel2_Image_Export',
    folder='GEE_Exports',
    fileNamePrefix='sentinel2_image',
    region=roi, # vùng nghiên cứu
    scale=10, # độ phân giải (10m cho Sentinel-2)
    crs='EPSG:4326', # hệ tọa độ mong muốn
    maxPixels=1e9 # giới hạn số pixel để tránh lỗi khi xuất ảnh lớn
)
# task.start() # Bỏ comment dòng này để bắt đầu quá trình xuất ảnh
```

### 26.2.2. Tải hàng loạt ảnh về Google Drive


```python
# Tương tự như tải 1 ảnh, bạn có thể tải toàn bộ ImageCollection bằng cách loop qua từng ảnh và tạo task export cho mỗi ảnh. Tuy nhiên, hãy cẩn thận với số lượng ảnh và giới hạn tài nguyên của Google Drive.

for i in range(sen2col.size().getInfo()):
    image = ee.Image(sen2col.toList(sen2col.size()).get(i))
    task = ee.batch.Export.image.toDrive(
        image=image.toFloat(),
        description=f'Sentinel2_Image_Export_{i}',
        folder='GEE_Exports',
        fileNamePrefix=f'sentinel2_image_{i}',
        region=roi,
        scale=10,
        crs='EPSG:4326',
        maxPixels=1e9
    )
    # task.start() # Bỏ comment dòng này để bắt đầu tải về từng ảnh
```

## 26.3 Tải ảnh về GEE Asset

### 26.3.1. Tải một ảnh về GEE Asset


```python
# Lưu một ảnh vào Asset của GEE
image = sen2col.first()
task = ee.batch.Export.image.toAsset(
    image=image.toFloat(),
    description='Sentinel2_Image_Export_Asset',
    assetId='users/yourasset_id/sentinel2_image_asset',
    region=roi,
    scale=10,
    crs='EPSG:4326',
    maxPixels=1e9
)
# task.start() # Bỏ comment dòng này để bắt đầu tải về ảnh vào Asset
```

### 26.3.2. Tải nhiều ảnh về GEE Asset


```python
# Lưu nhiểu ảnh vào Asset bằng cách loop qua từng ảnh trong ImageCollection
for i in range(sen2col.size().getInfo()):
    image = ee.Image(sen2col.toList(sen2col.size()).get(i))
    task = ee.batch.Export.image.toAsset(
        image=image.toFloat(),
        description=f'Sentinel2_Image_Export_Asset_{i}',
        assetId=f'users/yourasset_id/sentinel2_image_asset_{i}',
        region=roi,
        scale=10,
        crs='EPSG:4326',
        maxPixels=1e9
    )
    # task.start() # Bỏ comment dòng này để bắt đầu tải về từng ảnh vào Asset
```

## 26.4. Hiển thị ảnh với Ipython


```python
from IPython.display import Image as IPImage, display
url_thumb = image.select(['B4', 'B3', 'B2']).getThumbURL({
    'min'       : 0,
    'max'       : 3000,
    'region'    : roi,
    'dimensions': 600        # chiều dài cạnh tối đa (px)
})
display(IPImage(url=url_thumb, width=500))
```

## Tóm tắt

Bạn đã hoàn thành Bài 26 và nắm vững kỹ thuật **xuất dữ liệu từ Google Earth Engine** — bước quan trọng để đưa kết quả phân tích từ đám mây về môi trường làm việc cục bộ hoặc lưu trữ lại trên GEE.

### Các khái niệm chính đã nắm vững:
- ✅ Export ảnh về **Google Drive** dưới dạng GeoTIFF bằng `ee.batch.Export.image.toDrive()`
- ✅ Export hàng loạt ảnh từ `ImageCollection` bằng vòng lặp Python và kiểm soát từng `task`
- ✅ Lưu ảnh lên **GEE Asset** bằng `ee.batch.Export.image.toAsset()` để tái sử dụng và chia sẻ
- ✅ Xem nhanh ảnh trực tiếp trong notebook bằng `getThumbURL()` và `IPython.display`

### Kỹ năng bạn có thể áp dụng:
- Export kết quả phân tích (composite, chỉ số thực vật, phân loại...) thành file GeoTIFF sẵn sàng dùng trong QGIS, ArcGIS hoặc rasterio
- Tự động hóa việc tải về hàng loạt ảnh theo tháng/năm bằng vòng lặp Python
- Lưu ảnh trung gian vào GEE Asset để tránh tính toán lại và chia sẻ với cộng tác viên
- Dùng `getThumbURL()` để xem nhanh kết quả mà không cần chạy task export tốn thời gian
- Kiểm soát các tham số export quan trọng: `scale`, `crs`, `region`, `maxPixels` để đảm bảo chất lượng đầu ra
