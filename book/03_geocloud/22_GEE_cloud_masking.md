# Bài 22: Loại bỏ mây trên ảnh (GEE)

Loại bỏ mây là bước tiền xử lý quan trọng trong mọi phân tích viễn thám. Trong bài này ta học cách đọc và áp dụng **QA bits**, thông tin chất lượng pixel được cung cấp sẵn bởi mỗi sản phẩm vệ tinh.

## 22.1. Mục tiêu học tập

Sau bài này bạn có thể:

- Viết hàm cloud mask cho Sentinel-2 sử dụng band `QA60`
- Viết hàm cloud mask cho Landsat 8/9 sử dụng band `QA_PIXEL`
- Viết hàm cloud mask cho MODIS sử dụng band `DetailedQA`
- Áp dụng cloud mask lên ImageCollection bằng `.map()`


```python
import ee
import geemap

ee.Authenticate()  # Chỉ cần chạy lần đầu để xác thực tài khoản GEE
ee.Initialize() # Khởi tạo GEE sau khi đã xác thực
```

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

## 22.2. Loại bỏ mây với dữ liệu Sentinel-2

Mỗi một bức ảnh vệ tinh chụp vào thời gian khác nhau, và chứa mây phủ hoặc chất lượng khác nhau. Do vậy, ý tưởng là chúng ta sẽ truy cập vào mỗi ảnh, lấy ra thông tin về chất lượng pixel của ảnh đó, và đưa chúng về dạng nhị phân 0 và 1 (1: không mây và 0: có mây) và sau đó sẽ xóa bỏ phần có mây nơi mà giá trị pixel là 0. Để thực hiện được ý tưởng này, thông thường ta sẽ viết 1 hàm để loại bỏ mây cho 1 bức ảnh, sau đó sẽ `map` hàm đó cho tất cả các bức ảnh trong bộ sưu tập.

Trong phần này, chúng ta sẽ loại bỏ mây trong ảnh Sentinel-2 dựa trên lớp QA60 và Scene Classification Layer (SCL). QA60 sử dụng các bit chất lượng để xác định pixel bị che phủ bởi mây và mây ti tầng (cirrus), trong khi SCL phân loại chi tiết các đối tượng trên bề mặt, bao gồm cả mây và bóng mây.

Để thuận lợi cho việc truy cập và loại bỏ mây, chúng ta có thể tạo ra các hàm để loại bỏ mây như bên dưới.



```python
# Phương pháp 1: QA60 bit masking (áp dụng cho S2 TOA lẫn S2 SR)
def mask_sen2cloud_qa60(image):
    """
    Mask cloud and cirrus based on QA60 band of Sentinel-2.
    QA60 bit 10 = opaque clouds, bit 11 = cirrus clouds.
    Args:     
        image: ee.Image, Sentinel-2 image with QA60 band.
    Returns: 
        ee.Image, masked and scaled image with original properties.

    """
    qa = image.select('QA60')
    cloud_bit_mask  = 1 << 10   # = 1024
    cirrus_bit_mask = 1 << 11   # = 2048

    # Pixel sạch: cả 2 bit đều = 0
    mask = (qa.bitwiseAnd(cloud_bit_mask).eq(0)
              .And(qa.bitwiseAnd(cirrus_bit_mask).eq(0)))

    return (image
            .updateMask(mask)
            .divide(10000)          # Scale factor S2 SR: ÷ 10000 → [0, 1]
            .copyProperties(image, ['system:time_start', 'system:index']))

# Phương pháp 2: SCL (Scene Classification Layer) — chỉ S2 SR
def mask_sen2cloud_scl(image):
    """
    Mask cloud and cloud shadow based on SCL (Scene Classification Layer) of Sentinel-2 SR.
    SCL values: 3=Cloud shadows, 8=Cloud medium probability, 9=Cloud high probability, 10=Thin cirrus.
    Args:
        image: ee.Image, Sentinel-2 SR image with SCL band.
    Returns:
        ee.Image, masked and scaled image with original properties.
    """
    scl = image.select('SCL')
    # Giữ lại: 4=Vegetation, 5=Bare soil, 6=Water, 2=Dark area
    bad_pixels = scl.eq(3).Or(scl.eq(8)).Or(scl.eq(9)).Or(scl.eq(10))
    mask = bad_pixels.Not()

    return (image
            .updateMask(mask)
            .divide(10000)
            .copyProperties(image, ['system:time_start', 'system:index']))
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
# Áp dụng phương pháp QA60 để lọc mây cho Sentinel-2 SR
sen2col_qa60 = (ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
             .filterBounds(roi)
                .filterDate(start_date, end_date)
                .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 30))
                .map(mask_sen2cloud_qa60)  # Áp dụng phương pháp QA60
)
print('Số lượng ảnh sau khi lọc QA60:', sen2col_qa60.size().getInfo())
# Tương tự như vậy, bạn có thể tạo một collection khác áp dụng phương pháp SCL nếu muốn so sánh.
sen2col_scl = (ee.ImageCollection('COPERNICUS/S2_SR_HARMONIZED')
             .filterBounds(roi)
                .filterDate(start_date, end_date)
                .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 30))
                .map(mask_sen2cloud_scl)  # Áp dụng phương pháp SCL
)
print('Số lượng ảnh sau khi lọc SCL:', sen2col_scl.size().getInfo())
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



    Số lượng ảnh sau khi lọc QA60: 8
    Số lượng ảnh sau khi lọc SCL: 8
    

## 22.3. Loại bỏ mây với dữ liệu Landsat 8/9

Việc loại bỏ mây trong dữ liệu Landsat 8/9 Collection 2 Level-2 được thực hiện bằng cách sử dụng các bit chất lượng trong băng `QA_PIXEL` để nhận diện và loại bỏ các pixel bị ảnh hưởng bởi mây, mây giãn nở (dilated cloud) và bóng mây. Đồng thời, băng `QA_RADSAT` được sử dụng để loại bỏ các pixel bị bão hòa tín hiệu. Sau khi áp dụng mặt nạ chất lượng bằng hàm `updateMask()`, các băng phản xạ bề mặt được hiệu chỉnh theo hệ số scale chính thức trước khi đưa vào phân tích.


```python
# Cloud mask cho Landsat 8/9 Collection 2 Level-2
def mask_landsat_clouds(image):
    """
        Mask clouds and cloud shadows for Landsat 8/9 Collection 2 Level-2.
        Args:
            image: ee.Image, Landsat 8/9 C2 L2 image with QA_PIXEL and QA_RADSAT bands.
        Returns:
            ee.Image, masked and scaled image with original properties.
    """
    qa = image.select('QA_PIXEL')

    # Mask: tất cả 3 loại cần = 0
    cloud_mask = (qa.bitwiseAnd(1 << 1).eq(0)   # Dilated cloud
                    .And(qa.bitwiseAnd(1 << 3).eq(0))   # Cloud
                    .And(qa.bitwiseAnd(1 << 4).eq(0)))   # Cloud shadow

    # Saturation mask — loại pixel bão hoà trong bất kỳ band quang học nào
    sat = image.select('QA_RADSAT')
    sat_mask = sat.bitwiseAnd(0b0111_1110).eq(0)  # bits 1-6 (bands 1-6)

    return (image
            .updateMask(cloud_mask.And(sat_mask))
            # Landsat C2 L2 Scale: SR = ×0.0000275 − 0.2
            .select(['SR_B2','SR_B3','SR_B4','SR_B5','SR_B6','SR_B7'],
                    ['Blue','Green','Red','NIR','SWIR1','SWIR2'])
            .multiply(0.0000275).add(-0.2)   # Scale factor chính thức
            .copyProperties(image, ['system:time_start', 'system:index']))


# Đọc bộ sưu tập Landsat 8/9, áp dụng mask và gộp chung
ls8col = (ee.ImageCollection('LANDSAT/LC08/C02/T1_L2')
          .filterBounds(roi)
          .filterDate(start_date, end_date)
          .filter(ee.Filter.lt('CLOUD_COVER', 40)))
ls9col = (ee.ImageCollection('LANDSAT/LC09/C02/T1_L2')
          .filterBounds(roi)
          .filterDate(start_date, end_date)
          .filter(ee.Filter.lt('CLOUD_COVER', 40)))
# Gộp L8 + L9 và sắp xếp theo thời gian
landsat_col = ls8col.merge(ls9col).sort('system:time_start')
# Áp dụng mask cho toàn bộ collection
landsat_cloudmasked = landsat_col.map(mask_landsat_clouds)
print('Số lượng ảnh Landsat sau khi lọc:', landsat_cloudmasked.size().getInfo())
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



    Số lượng ảnh Landsat sau khi lọc: 11
    

## 22.4. Loại bỏ mây với dữ liệu MODIS

Việc loại bỏ mây trong dữ liệu MODIS được thực hiện bằng cách trích xuất các bit chất lượng từ băng QA để tạo mặt nạ nhị phân (binary mask). Các pixel bị ảnh hưởng bởi mây hoặc không đạt chất lượng được đánh dấu trong mặt nạ và loại bỏ thông qua hàm updateMask(). Kết quả là chỉ các pixel có chất lượng tốt được giữ lại cho các bước phân tích tiếp theo.


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
modis = ee.ImageCollection("MODIS/061/MOD13A2").filterBounds(roi).filterDate(start_date, end_date)
modis_masked = mask_cloud(modis, from_bit=0, to_bit=1, QA_band='state_1km', threshold=0)
print('Số lượng ảnh MODIS sau khi lọc:', modis_masked.size().getInfo())
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



    Số lượng ảnh MODIS sau khi lọc: 7
    

## Tóm tắt

Bạn đã hoàn thành Bài 22 và nắm vững kỹ thuật loại bỏ mây (cloud masking) - bước tiền xử lý bắt buộc trong mọi phân tích viễn thám quang học trên Google Earth Engine.

### Các khái niệm chính đã nắm vững:
- ✅ Lọc mây sử dụng các bộ dữ liệu khác nhau với `QA` band khác nhau.
- ✅ Áp dụng hàm `mask` lên toàn bộ `ImageCollection` theo cơ chế server-side
- ✅ Chuẩn hóa giá trị (scaling factors)
- ✅ Gộp hai collection để tăng tần suất ảnh trong cùng một khu vực và thời gian

### Kỹ năng bạn có thể áp dụng:
- Viết hàm cloud mask tùy chỉnh cho bất kỳ sản phẩm vệ tinh nào dựa trên tài liệu QA band của nhà cung cấp
- Áp dụng cloud mask cho Sentinel-2 bằng cả hai phương pháp (QA60 và SCL) và chọn phương pháp phù hợp với từng bài toán
- Xử lý đồng thời Landsat 8 và Landsat 9 để tăng mật độ ảnh time-series
- Xây dựng pipeline tiền xử lý hoàn chỉnh: filter → cloud mask → scale factor → sẵn sàng cho phân tích
- Đặt nền tảng sạch dữ liệu để tính chỉ số thực vật (NDVI), phát hiện hạn hán, lập bản đồ ngập lụt ở các bài tiếp theo
