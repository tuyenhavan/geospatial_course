# Bài 27: Xuất dữ liệu từ GEE

Google Earth Engine cho phép **export dữ liệu** ở ba dạng chính:
- **Image → Drive/Asset**: ảnh raster dưới dạng GeoTIFF
- **Table → Drive/Asset**: bảng dữ liệu dưới dạng CSV / GeoJSON / SHP
- **getDownloadURL**: tải trực tiếp ảnh nhỏ về máy (không cần task)

> **Lưu Ý**: Bạn có thể chạy trực tiếp notebook bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/15W5vdI05KGvSMtKbiBiVd1h30bDFJ5gQ) mà không cần cài đặt Python. Trong trường hợp bạn muốn tạo bản sao của notebook này, bạn có thể làm như sau: `File → Save a copy in Drive`.

## 27.1. Mục tiêu học tập

Sau khi hoàn thành bài này, bạn có thể:

- Export ảnh (Image) ra **Google Drive** dưới dạng GeoTIFF có thể dùng trong QGIS/ArcGIS.
- Export ảnh hàng loạt theo tháng/năm bằng vòng lặp Python
- Export ảnh lên `GEE AssetAsset` để tái sử dụng và chia sẻ


```python
import ee
import geemap
ee.Authenticate()
ee.Initialize(project='geocourse-501706')
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

## 27.2 Tải ảnh về Google Drive

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

### 27.2.1. Tải một ảnh về Google Drive

Để xuất một ảnh từ GEE về Google Drive, sử dụng `ee.batch.Export.image.toDrive()`. Phương thức này tạo một task chạy trên server của GEE và lưu kết quả vào thư mục được chỉ định trên Google Drive của bạn. Các tham số quan trọng bao gồm: `image` (ảnh cần xuất - nên chuyển sang `.toFloat()` để đảm bảo tính nhất quán kiểu dữ liệu), `region` (vùng xuất - thường là AOI), `scale` (độ phân giải pixel tính bằng mét), `crs` (hệ tọa độ), và `maxPixels` (giới hạn số pixel tối đa để tránh lỗi khi xuất ảnh quá lớn). Sau khi tạo task, cần gọi `.start()` để bắt đầu quá trình export. Bạn có thể theo dõi tiến độ tại tab Tasks trong Code Editor hoặc qua GEE console.


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

### 27.2.2. Tải hàng loạt ảnh về Google Drive

Để xuất toàn bộ ImageCollection (nhiều ảnh), cần sử dụng vòng lặp Python để tạo task export riêng cho từng ảnh. Phương pháp này hữu ích khi bạn muốn tải về chuỗi thời gian ảnh theo tháng, năm hoặc tất cả ảnh trong một collection đã được xử lý. Lưu ý rằng mỗi task là độc lập và chạy song song trên GEE server, do đó việc tạo nhiều tasks không làm chậm quá trình export. Tuy nhiên, cần cẩn thận với giới hạn số lượng tasks đồng thời (thường là 3000 tasks pending) và dung lượng Google Drive. Nên đặt tên file có ý nghĩa (ví dụ thêm index hoặc ngày tháng) để dễ quản lý sau khi tải về.


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

## 27.3 Tải ảnh về GEE Asset

### 27.3.1. Tải một ảnh về GEE Asset

GEE Asset là kho lưu trữ riêng trên GEE cho phép bạn lưu kết quả xử lý để tái sử dụng mà không cần tính toán lại. Khác với export ra Drive, ảnh lưu trong Asset có thể được load lại ngay lập tức bằng `ee.Image('users/yourname/assetname')` mà không cần tải về máy. Điều này rất hữu ích khi bạn có các bước tiền xử lý tốn thời gian (cloud masking, tổng hợp composite...) và muốn sử dụng kết quả trong nhiều phân tích khác nhau. Asset cũng giúp chia sẻ dữ liệu với đồng nghiệp bằng cách cấp quyền truy cập. Lưu ý rằng mỗi tài khoản GEE có giới hạn dung lượng Asset (thường 250GB cho tài khoản miễn phí).


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

### 27.3.2. Tải nhiều ảnh về GEE Asset

Tương tự như export hàng loạt ra Drive, bạn có thể lưu toàn bộ `ImageCollection` vào GEE Asset bằng vòng lặp. Mỗi ảnh sẽ được lưu thành một Asset riêng với đường dẫn `assetId` duy nhất. Kỹ thuật này rất hữu ích khi bạn có workflow phức tạp: ví dụ tạo monthly composites từ Sentinel-2, lưu vào Asset, sau đó các phân tích tiếp theo chỉ cần load Asset mà không phải chạy lại toàn bộ pipeline. Sau khi tất cả tasks hoàn thành, bạn có thể tạo một ImageCollection mới từ danh sách Assets bằng cách load từng Asset và merge chúng lại. Nhớ quản lý dung lượng Asset thường xuyên để tránh vượt quá giới hạn.


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

## 27.4. Hiển thị ảnh với Ipython

Phương thức `.getThumbURL()` cho phép xem nhanh ảnh trực tiếp trong Jupyter notebook mà không cần export. GEE sẽ render ảnh thành PNG/JPEG nhỏ theo các tham số visualization (min, max, bands, dimensions) và trả về URL để hiển thị. Kỹ thuật này rất hữu ích để kiểm tra kết quả nhanh chóng trước khi quyết định export toàn bộ ảnh (có thể mất nhiều thời gian).


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
