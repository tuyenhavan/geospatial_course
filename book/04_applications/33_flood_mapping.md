# Bài 33: Theo dõi lũ lụt sử dụng ảnh Sentinel-1

## 33.1. Mục tiêu bài học


```python
import ee 
ee.Authenticate()
ee.Initialize()
```

## 33.2. Chuẩn bị dữ liệu


```python
bbox = [
    109.07678308345344,
    12.63164734584147,
    109.12381830073859,
    12.676032712162309
]
# Chuyển đổi bbox thành đối tượng Geometry
geometry = ee.Geometry.Rectangle(bbox)
```


```python
sen1col = ee.ImageCollection("COPERNICUS/S1_GRD")
```

## 33.3. Tính toán chỉ số nước

### 33.3.1. Tính toán chỉ số NDWI


```python

```

### 33.3.2. Xác định vùng nước dựa vào chỉ số NDWI
