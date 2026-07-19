# Bài 32: Phân loại đất sử dụng ảnh Sentinel-2 và máy học

Trong các bài toán viễn thám hiện nay, phân loại lớp phủ đất là một trong những hướng nghiên cứu quan trọng, cho phép xác định các loại bề mặt lớp phủ khác nhau như đất rừng, đất nông nghiệp, đất đô thị hay mặt nước từ dữ liệu ảnh vệ tinh. Với sự phát triển của máy học, thay vì xây dựng các ngưỡng thủ công, ta có thể huấn luyện mô hình dựa trên dữ liệu phổ để tự động học mối quan hệ giữa đặc trưng ảnh và các lớp đất.


> **Lưu ý**: Nếu bạn chưa muốn cài đặt Python trên máy tính, bạn cũng có thể chạy trực tiếp notebook bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/1VioqXc1Y0goh2aliaL-x7HLEJyKFEYln).
>
> Dữ liệu thực hành có thể tải tại [đây](https://drive.google.com/drive/folders/119C2B1pBKwvDx5OASvQRvGR1lOljgJNd?usp=sharing)

## 32.1. Mục tiêu bài học

- Chuẩn bị dữ liệu cho huấn luyện mô hình máy học random forest
- Xác định số kênh và loại đất phân loại
- Sử dụng mô hình Random forest để huấn luyện bài toán phân loại đất
- Sử dụng mô hình đã huấn luyện để dự đoán trên ảnh mới.

## 32.2. Chuẩn bị dữ liệu

Trong ví dụ này, ta sử dụng GEE để tải dữ liệu ảnh vệ tinh Sentinel-2 trong giai đoạn 2025-06-01 đến 2025-09-30 cho AOI (khu vực nghiên cứu), lựa chọn các kênh phổ quan trọng gồm B2, B3, B4, B5, B6, B7 và B8 để đại diện cho thông tin màu sắc và cấu trúc thực vật. Để giảm nhiễu do mây (giới hạn 5% mây) và sau đó tính giá trị median để tạo ra một ảnh đại diện ổn định cho toàn bộ giai đoạn nghiên cứu.

Từ tập hợp đặc trưng phổ này, ta xây dựng bộ dữ liệu huấn luyện để mô hình học máy nhận biết các loại lớp phủ đất với 5 loại đất chính (1: Nước, 2: Rừng, 3: Nông nghiệp, 4: Đô thị/đường, 5: Cây bụi). Sau khi huấn luyện, mô hình được áp dụng để dự đoán bản đồ phân loại cho toàn bộ vùng nghiên cứu, từ đó hỗ trợ phân tích hiện trạng sử dụng đất trong khu vực.

### 32.2.1. Xác định vùng nghiên cứu và ảnh Sentinel-2

Trong ví dụ này, ta chọn vùng nghiên cứu là một vùng nhỏ ở phía nam nước Đức như được xác định bởi bounding box bên dưới. 

> Bạn có thể chọn khu vực nghiên cứu khác và phương pháp và quy trình thực hiện hoàn toàn tương tự.


```python
import ee 
import rioxarray as rxr
ee.Authenticate()
ee.Initialize(project='geocourse-501706')
```


```python
# Bounding box cho vùng nghiên cứu ở Đức
bbox = [9.84375   , 47.5172007 , 10.1953125 , 47.75409798]
roi = ee.Geometry.Rectangle(bbox)
```


```python
sen2col = ee.ImageCollection("COPERNICUS/S2_SR_HARMONIZED").filterBounds(roi).filterDate("2025-06-01", "2025-09-30").filter(ee.Filter.lt("CLOUDY_PIXEL_PERCENTAGE", 5)).select(["B2", "B3", "B4", "B5", "B6", "B7", "B8"])
# Tính giá trị median cho mỗi pixel trong khoảng thời gian đã lọc
sen2_median = sen2col.median().multiply(0.0001) # Sentinel-2 Surface Reflectance cần nhân với 0.0001 để chuyển về giá trị phản xạ thực tế
```

### 32.2.2. Tải dữ liệu Sentinel-2 cho vùng nghiên cứu

Dữ liệu cho bài này đã được chuẩn bị trong tại [đây](https://drive.google.com/drive/folders/1jCC6j6MAGOk6ODUjbasE3tOGyyIH6U9Q?usp=sharing). Nếu bạn muốn thực hành việc tải dữ liệu, bạn có thể làm như bên dưới và tải dữ liệu về Google Drive của mình. Sau khi tải xong, bạn đưa dữ liệu vào QGIS và vẽ polygons cho các loại đất mong muốn. Sau đó trích xuất giá trị spectral bands cho các polygons loại đất tương ứng. Dữ liệu sau khi trích xuất sẽ là một bảng thể hiện các bands và loại đất.


```python
task = ee.batch.Export.image.toDrive(
    image=sen2_median.toFloat(), # ảnh cần tải về
    description='Sentinel2_Image_Export',
    folder='GEE_Exports',
    fileNamePrefix='sen2median_2025',
    region=roi, # vùng nghiên cứu
    scale=10, # độ phân giải (10m cho Sentinel-2)
    crs='EPSG:32632', # hệ tọa độ mong muốn
    maxPixels=1e9 # giới hạn số pixel để tránh lỗi khi xuất ảnh lớn
)
# task.start() # Bỏ comment dòng này để bắt đầu quá trình xuất ảnh
# Sau khi chạy task.start(), bạn có thể theo dõi tiến trình xuất ảnh trong Google Drive của mình. Khi hoàn thành, ảnh sẽ được lưu trong thư mục 'GEE_Exports' với tên 'sen2median_2025.tif'.
```

# 32.3. Xây dựng mô hình

Trong bài này, chúng ta sẽ sử dụng mô hình máy học Random Forest để phân loại lớp phủ đất. Random Forest là một thuật toán học máy giám sát, hoạt động bằng cách xây dựng nhiều cây quyết định từ các mẫu dữ liệu huấn luyện khác nhau. Kết quả phân loại cuối cùng được xác định dựa trên nguyên tắc bỏ phiếu của các cây, giúp tăng độ chính xác, giảm hiện tượng quá khớp (overfitting) và cho kết quả ổn định trên các bộ dữ liệu viễn thám.

Trong Python, Random Forest được tích hợp sẵn trong thư viện `scikit-learn`, vì vậy chúng ta chỉ cần import mô hình từ thư viện và sử dụng để huấn luyện cũng như dự đoán dữ liệu.


```python
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import classification_report, confusion_matrix, accuracy_score
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
```


```python
# Đọc dữ liệu huấn luyện từ file excel 
data = pd.read_csv(r"J:\My Drive\geocourse_data\excels\landcover_train_data_sample.csv")
# check classes 
data['classes'].value_counts() # Có vẻ class số 2 nhiều hơn, mất cân bằng nhưng vẫn chấp nhận được, có thể dùng SMOTE nếu muốn cân bằng dữ liệu
```

### 32.3.1. Huấn luyện mô hình

- **Chia dữ liệu thành tập train và tập test**

Để đánh giá hiệu suất của mô hình, chúng ta nên chia dữ liệu thành hai phần: tập huấn luyện (train set) dùng để huấn luyện mô hình và tập kiểm tra (test set) dùng để đánh giá độ chính xác. Ở đây ta sử dụng hàm `train_test_split` từ `scikit-learn` với tỷ lệ 80% dữ liệu cho huấn luyện và 20% cho kiểm tra.


```python
X = data.drop(columns=['classes']).values
y = data['classes'].values
# Chia dữ liệu thành tập huấn luyện và tập kiểm tra (20% dữ liệu dùng để kiểm tra)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
```

- **Huấn luyện mô hình**

Sau khi đã chia dữ liệu, chúng ta khởi tạo mô hình Random Forest với `n_estimators=500` (số lượng cây quyết định trong rừng) và `random_state=42` để đảm bảo tính tái lập của kết quả. Sau đó sử dụng phương thức `fit()` để huấn luyện mô hình trên tập dữ liệu huấn luyện.


```python
model = RandomForestClassifier(n_estimators=500, random_state=42)
model.fit(X_train, y_train)
```

### 32.3.2. Đánh giá mô hình

Sau khi huấn luyện mô hình, chúng ta cần đánh giá hiệu suất của nó trên tập dữ liệu kiểm tra để xem mô hình có thể tổng quát hóa tốt trên dữ liệu mới hay không. Các chỉ số đánh giá quan trọng bao gồm:

- **Accuracy (Độ chính xác)**: Tỷ lệ phần trăm mẫu được phân loại đúng trên tổng số mẫu
- **Precision, Recall, F1-score**: Đánh giá chi tiết độ chính xác, độ bao phủ và cân bằng giữa hai chỉ số này cho từng lớp
- **Confusion Matrix**: Ma trận nhầm lẫn giúp xem xét chi tiết các lớp nào bị nhầm lẫn với nhau, từ đó có thể cải thiện mô hình hoặc thu thập thêm dữ liệu huấn luyện cho các lớp này


```python
y_pred = model.predict(X_test)
# Độ chính xác của mô hình
accuracy = accuracy_score(y_test, y_pred)
print(f"Accuracy: {accuracy:.2f}")
# độ chính xác của từng class
print(classification_report(y_test, y_pred))
```


```python
# Bạn có thể in ra confusion matrix để xem chi tiết hơn về hiệu suất của mô hình
conf_matrix = confusion_matrix(y_test, y_pred)
# Chuyển dạng count sang dạng tỷ lệ phần trăm
conf_matrix_percentage = conf_matrix.astype('float') / conf_matrix.sum(axis=1)[:, np.newaxis]
conf_matrix_percentage =conf_matrix_percentage.round(2)
# Chuyển dạng sang DataFrame để dễ đọc hơn
conf_matrix_df = pd.DataFrame(conf_matrix_percentage, index=model.classes_, columns=model.classes_)
print("Confusion Matrix (Percentage):")
print(conf_matrix_df)
```

> **Nhận xét**: Nhìn chung mô hình có độ chính xác cao, đạt khoảng 98% trên tập kiểm tra. Trong số các loại đất, **cây bụi** có độ chính xác thấp nhấp, khoảng 61%, chủ yếu nhầm lẫn với **đô thị/đường**. Chú ý rằng, đây chỉ là một ví dụ minh họa, và để đánh giá khả năng tổng quát hóa của mô hình, cần kiểm định trên tập dữ liệu độc lập tại khu vực hoặc thời điểm khác nhằm đánh giá khả năng chuyển giao theo không gian và thời gian (spatial và temporal transferability) của mô hình.


```python
# Sau khi bạn hài lòng với mô hình, bạn có thể lưu mô hình để sử dụng sau này mà không cần huấn luyện lại
import joblib
# Lưu mô hình vào file
# joblib.dump(model, 'random_forest_model.pkl')
```

## 32.4. Sử dụng mô hình dự đoán

Sau khi mô hình đã được huấn luyện, chúng ta có thể sử dụng mô hình để dữ đoán cho ảnh trên toàn bộ khu vực nghiên cứu. Trong ví dụ này là ảnh `sen2median_2025.tif` đã chuẩn bị trong Google Drive link tại [đây](https://drive.google.com/drive/folders/17Nnmw5TSm3zxacZsHBiZNb5rUdrIiygH?usp=sharing).

Có nhiều cách để đưa dữ liệu ảnh về theo cấu trúc dữ liệu huấn luyện mô hình. Trong ví dụ này, chúng ta sử dụng 2 cách là dùng `rioxarray` và `rasterio`.

- **Đọc ảnh với xarray và dự đoán**

Phương pháp này sử dụng `rioxarray` để đọc toàn bộ ảnh vào bộ nhớ, sau đó chuyển đổi dữ liệu từ dạng đa chiều (bands, height, width) sang dạng phẳng (n_samples, n_features) để phù hợp với đầu vào của mô hình. Phương pháp này đơn giản nhưng yêu cầu bộ nhớ RAM lớn nếu ảnh có kích thước lớn.


```python
# path đến ảnh 
import rioxarray as rxr
path = r"J:\My Drive\geocourse_data\raster\sen2median_2025.tif"
input_raster = rxr.open_rasterio(path, masked=True).squeeze() # đọc ảnh và bỏ chiều channel nếu có
input_raster.attrs = {}
# flatten mỗi band thành 1D array và đưa vào numpy array
flattened_bands = np.array([band.values.flatten().reshape(-1, 1) for band in input_raster]).squeeze().T # chuyển sang dạng (n_samples, n_features)
```


```python
# Load mô hình đã lưu
# model = joblib.load('random_forest_model.pkl') # Nếu bạn đã lưu mô hình trước đó, bạn có thể tải lại mô hình từ file
y_pred = model.predict(flattened_bands) # dự đoán lớp cho từng pixel. Mất khoảng 5-6 phút, tùy vào số lượng pixel trong ảnh. Nếu ảnh quá lớn, bạn có thể chia nhỏ ảnh ra để dự đoán từng phần hoặc tham khảo cách bên dưới với rasterio.
# reshape lại kết quả dự đoán về hình dạng ban đầu của ảnh
output = input_raster[0].copy().astype(np.uint16) # tạo một bản sao của ảnh đầu vào để giữ metadata
output.data = y_pred.reshape(output.shape) # gán dữ liệu dự đoán vào ảnh đầu ra
# lưu ảnh kết quả dự đoán
output.rio.to_raster(r"J:\My Drive\geocourse_data\outputs\landcover_prediction.tif", dtype='uint16') # lưu ảnh kết quả dự đoán
```

- **Đọc với rasterio và dữ đoán**

Khi dữ liệu lớn, bạn có thể sử dụng rasterio và đọc từng window một vào để dự đoán và viết lưu vào file như bên dưới.


```python
import rasterio as rio 
from rasterio.windows import Window
with rio.open(path, 'r') as src:
    # Lấy metadata của ảnh gốc
    meta = src.meta.copy()
    # Cập nhật metadata cho ảnh kết quả dự đoán
    meta.update({
        'count': 1,  # số lượng band
        'dtype': 'uint8'  # kiểu dữ liệu của ảnh kết quả dự đoán
    })
    # Lưu ảnh kết quả dự đoán với metadata mới
    with rio.open(r"J:\My Drive\geocourse_data\outputs\landcover_prediction_rasterio.tif", 'w', **meta) as dst:
        for _, window in src.block_windows(1):
            # Đọc dữ liệu từ ảnh gốc
            block = src.read(window=window)
            bands, height, width = block.shape
            # Chuyển đổi block sang dạng (n_samples, n_features)
            X = block.reshape(bands, -1).T
            # Dự đoán lớp cho block
            y_pred_block = model.predict(X)
            # viết dữ liệu dự đoán vào ảnh kết quả dự đoán
            dst.write(y_pred_block.reshape(height, width).astype('uint8'), window=window, indexes=1)
```

## Tóm tắt

Trong bài học này, bạn đã học cách phân loại lớp phủ đất sử dụng ảnh Sentinel-2 và máy học Random Forest - kỹ năng quan trọng trong ứng dụng viễn thám hiện đại để xác định các loại bề mặt lớp phủ khác nhau như đất rừng, đất nông nghiệp, đất đô thị hay mặt nước.

### Các khái niệm chính đã nắm vững:
- ✅ **Chuẩn bị dữ liệu huấn luyện**: Thu thập và xử lý ảnh Sentinel-2 từ Google Earth Engine, lựa chọn các kênh phổ quan trọng (B2-B8) và tính toán giá trị median để giảm nhiễu
- ✅ **Xây dựng bộ dữ liệu phân loại**: Tạo dữ liệu huấn luyện cho 5 loại lớp phủ đất (Nước, Rừng, Nông nghiệp, Đô thị/đường, Cây bụi) với các đặc trưng phổ từ ảnh vệ tinh
- ✅ **Huấn luyện mô hình Random Forest**: Sử dụng scikit-learn để xây dựng và huấn luyện mô hình phân loại với việc chia tập train/test và điều chỉnh tham số
- ✅ **Đánh giá hiệu suất mô hình**: Sử dụng confusion matrix, classification report và accuracy score để đánh giá độ chính xác và hiệu suất của mô hình
- ✅ **Dự đoán trên ảnh lớn**: Áp dụng mô hình đã huấn luyện để phân loại toàn bộ vùng nghiên cứu sử dụng cả rioxarray và rasterio với xử lý theo window cho dữ liệu lớn

### Kỹ năng bạn có thể áp dụng:
- Xây dựng quy trình hoàn chỉnh từ thu thập dữ liệu ảnh vệ tinh đến phân loại lớp phủ đất tự động
- Sử dụng Google Earth Engine hoặc Planetary computer để tải và xử lý dữ liệu Sentinel-2 cho khu vực nghiên cứu
- Huấn luyện và tối ưu mô hình máy học Random Forest cho bài toán phân loại ảnh viễn thám
- Xử lý và dự đoán trên dữ liệu raster lớn một cách hiệu quả với rioxarray và rasterio
- Lưu và tái sử dụng mô hình đã huấn luyện cho các bài toán tương tự trong tương lai
- Áp dụng kỹ thuật phân loại lớp phủ đất để phân tích hiện trạng và biến động sử dụng đất trong các dự án viễn thám thực tế.
