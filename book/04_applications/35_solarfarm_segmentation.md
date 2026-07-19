# Bài 35: Ứng dụng học sâu và dữ liệu Sentinel-2 cho phát hiện trang trại điện mặt trời

Năng lượng mặt trời đang ngày càng đóng vai trò quan trọng trong chuyển đổi năng lượng sạch toàn cầu. Việc theo dõi và lập bản đồ các trang trại điện mặt trời là cần thiết cho quy hoạch năng lượng, đánh giá tiềm năng năng lượng tái tạo và giám sát phát triển bền vững. Tuy nhiên, việc xác định và phân loại thủ công các trang trại điện mặt trời từ ảnh vệ tinh là công việc tốn kém thời gian và nguồn lực.

Trong bài học này, chúng ta sẽ áp dụng kỹ thuật học sâu (deep learning) với kiến trúc `UNet`, một trong những mô hình được sử dụng phổ biến cho bài toán phân đoạn ảnh (image segmentation), kết hợp với dữ liệu ảnh vệ tinh Sentinel-2 để tự động phát hiện và vẽ ranh giới các trang trại điện mặt trời. Phương pháp này không chỉ tiết kiệm thời gian mà còn có khả năng xử lý diện tích lớn, mở ra tiềm năng ứng dụng trong giám sát và quản lý năng lượng tái tạo quy mô quốc gia.

> **Lưu ý**: Bạn có thể chạy trực tiếp notebook này bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/1ikx8wnQpOg6GAeQwjHMIdN5p9qs3GmcV) mà không cần cài đặt thự viện hay Python. 
>
> Dữ liệu thực hành có thể tải tại [đây](https://drive.google.com/drive/folders/119C2B1pBKwvDx5OASvQRvGR1lOljgJNd?usp=sharing)

## 35.1. Mục tiêu bài học

Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Hiểu kiến trúc `UNet` và ứng dụng của nó trong bài toán phân đoạn ảnh vệ tinh
- Xây dựng dataset tùy chỉnh (`CustomDataset`) để đọc và xử lý ảnh huấn luyện
- Xây dựng mô hình `UNet` từ đầu với `PyTorch` bao gồm `encoder, bottleneck và decoder
- Hiểu và áp dụng các thành phần chính: convolution, max pooling, upsampling và skip connections
- Thiết lập quy trình huấn luyện và validation cho mô hình phân đoạn
- Sử dụng các kỹ thuật tối ưu hóa và hàm loss phù hợp cho bài toán binary segmentation
- Áp dụng mô hình học sâu vào bài toán thực tế phát hiện trang trại điện mặt trời từ ảnh Sentinel-2


```python
import os
import torch # Nếu chưa cài đặt torch thì cài đặt từ trang chính thức https://pytorch.org/get-started/locally/
import torch.nn as nn
import torch.nn.functional as TF
import rasterio as rio
# import albumentations as A # if not installed, install it using pip install albumentations
import numpy as np
from torch.utils.data import Dataset, DataLoader
from tqdm import tqdm
from sklearn.model_selection import train_test_split
import xarray as xr
```

## 35.2. Chuẩn bị dữ liệu huấn luyện

Dữ liệu huấn luyện cho mô hình học sâu đóng vai trò quyết định đến chất lượng của mô hình. Trong bài toán phân đoạn ảnh, chúng ta cần hai loại dữ liệu:

- **Ảnh gốc (images)**: Ảnh vệ tinh Sentinel-2 với các kênh màu lựa chọn. Trong bài này, chúng ta sẽ chọn hết các kênh màu Sentinel-2, trừ B1 và tính thêm NDVI.
- **Nhãn (masks)**: Ảnh nhị phân đánh dấu vị trí các trang trại điện mặt trời (pixel = 1) và các vùng khác (pixel = 0)

Quy trình chuẩn bị dữ liệu bao gồm các bước sau:
  1. Xác định khu vực nghiên cứu và khoảng thời gian. Trong bài học này, chúng ta chọn khu vực nghiên cứu theo bounding box `[107.82491814, 12.54327821, 107.97316082, 12.64476045]` và thời gian từ `20.03.2026` đến `28.03.2026`. Khu vực này chứa nhiều trang trại điện mặt trời, và trong thời gian này có một vài ảnh không có mây.
  2. Truy cập và tải dữ liệu Sentinel-2 từ Microsoft Planetary Computer. Giới hạn ảnh dưới 5% mây cho nghiên cứu này.
  3. Sau load ảnh, chúng ta sẽ offset ảnh 1000 và chuyển dữ liệu về reflectance bằng cách chia cho 10000. Xem ở ví dụ gốc tại [đây](https://planetarycomputer.microsoft.com/dataset/sentinel-2-l2a#Baseline-Change). 
  4. Tính NDVI và tạo median composite và tải ảnh về. Trong bài này ta sẽ lấy tất các band (trừ B1) và NDVI. 
  5. Tính giá trị percentile 2% và 98% và giới hạn dữ liệu trong khoảng này. Chuẩn hóa dữ liệu về trong khoảng 0 và 1.
  6. Sử dụng QGIS để vẽ polygon đánh dấu trang trại điện mặt trời. Có thể chọn 3 bands RGB cho việc vẽ polygons này.
  7. Rasterize các polygon thành mask nhị phân, nên cùng kích thước với ảnh Sentinel-2 đã tải.
  8. Tạo các patch nhỏ (128x128 pixels) từ ảnh và mask

Từ những kiến thức từ các bài trước, bạn có thể thực hiện được các bước chuẩn bị dữ liệu cho huấn luyện một mô hình học sâu theo các bước bên trên. Trong trường hợp bạn không thể thực hiện được các bước bên trên, bạn có thể tham khảo và tải dữ liệu huấn luyện đã chuẩn bị tại [đây](https://drive.google.com/drive/folders/1lTQ-2_k92ZiZbb1KuxGPLuBZqcMYqJfm?usp=sharing).

## 35.3. Xây dựng hàm đọc dữ liệu

Trước khi huấn luyện mô hình, chúng ta cần xây dựng một lớp Dataset tùy chỉnh để đọc và xử lý dữ liệu. `CustomDataset` kế thừa từ `torch.utils.data.Dataset` và thực hiện các nhiệm vụ:

- **Quản lý file paths**: Lưu trữ đường dẫn đến ảnh gốc và mask tương ứng
- **Đọc dữ liệu**: Sử dụng rasterio để đọc file GeoTIFF
- **Data augmentation**: Hỗ trợ các phép biến đổi tăng cường dữ liệu (flip, rotate, crop, v.v.) thông qua thư viện albumentations
- **Chuyển đổi sang tensor**: Biến đổi numpy array thành PyTorch tensor
- **Xử lý giá trị đặc biệt**: Thay thế NaN và Inf bằng giá trị 0 nếu có.

Dataset này sẽ được sử dụng với DataLoader để tạo batches dữ liệu trong quá trình huấn luyện.


```python
class CustomDataset(Dataset):
    """A custom dataset class for loading images and masks from file paths.
    
    Args:
        img_file_paths (list): List of file paths for the images.
        mask_file_paths (list): List of file paths for the masks.
        transform (albumentations.Compose, optional): Optional transform to be applied on a sample.
    Returns:
        tuple: (image, mask) where image is a tensor of shape (C, H, W) and mask is a tensor of shape (C, H, W).
    """
    def __init__(self, img_file_paths, mask_file_paths, transform=None):
        self.img_file_paths = img_file_paths
        self.mask_file_paths = mask_file_paths
        self.transform = transform
        assert len(self.img_file_paths) == len(self.mask_file_paths), "Image and mask file paths must have the same length."

    def __len__(self):
        return len(self.img_file_paths)

    def __getitem__(self, idx):
        image = self._read_image(self.img_file_paths[idx])
        mask = self._read_image(self.mask_file_paths[idx])

        if self.transform:
            if isinstance(self.transform, A.Compose):
                image = np.transpose(image, (1, 2, 0))  # Convert from (C, H, W) to (H, W, C)
                mask = np.transpose(mask, (1, 2, 0))  # Convert from (C, H, W) to (H, W, C)
                augmented = self.transform(image=image, mask=mask)
                image = np.transpose(augmented['image'], (2, 0, 1))  # Convert back to (C, H, W)
                mask = np.transpose(augmented['mask'], (2, 0, 1))  # Convert back to (C, H, W)
            else:
                raise ValueError("Transform should be an instance of albumentations.Compose")

        image = torch.from_numpy(image).float()
        image = image.nan_to_num(nan=0.0, posinf=0.0, neginf=0.0)  # Replace NaN and Inf values with 0
        mask = torch.from_numpy(mask).float()
        mask = mask.nan_to_num(nan=0.0, posinf=0.0, neginf=0.0)  # Replace NaN and Inf values with 0
        return image, mask

    def _read_image(self, file_path):
        with rio.open(file_path) as src:
            img = src.read()
        return img
```

## 35.4. Xây dựng mô hình UNet

**UNet** là kiến trúc mạng neural tích chập (CNN) được phát triển bởi [Olaf Ronneberger và cộng sự](https://arxiv.org/abs/1505.04597?utm_source=chatgpt.com) - thiết kế đặc biệt cho bài toán phân đoạn ảnh, đặc biệt hiệu quả khi dữ liệu huấn luyện hạn chế. Kiến trúc UNet có hình chữ U đặc trưng với các thành phần chính sau:

- **Encoder (Contracting Path)**: Thu nhỏ kích thước ảnh và trích xuất đặc trưng ở nhiều mức độ
- **Bottleneck**: Lớp nối giữa encoder và decoder, chứa đặc trưng ở mức độ trừu tượng cao nhất
- **Decoder (Expanding Path)**: Phục hồi kích thước ảnh ban đầu và tạo mask phân đoạn
- **Skip Connections**: Kết nối trực tiếp từ encoder sang decoder để giữ lại thông tin chi tiết

Cấu trúc này cho phép mô hình vừa học được ngữ cảnh tổng thể (qua encoder), vừa giữ lại chi tiết không gian chính xác (qua skip connections), điều quan trọng cho phân đoạn ảnh chính xác.  

### 35.4.1. Cấu trúc và các lớp Encoder

**Encoder** có nhiệm vụ giảm dần kích thước không gian của ảnh đầu vào đồng thời tăng số lượng kênh (channels), cho phép mô hình học các đặc trưng từ thấp đến cao:

- **DoubleConv**: Module cơ bản thực hiện hai lớp convolution liên tiếp, mỗi lớp theo sau bởi batch normalization và ReLU activation. Cấu trúc này giúp mô hình học được các đặc trưng phức tạp hơn.

- **EncoderBlock**: Kết hợp DoubleConv với max pooling để giảm kích thước ảnh xuống 1/2. Output gồm hai phần:
  - **Skip**: Kết quả sau DoubleConv, sẽ được truyền sang decoder (skip connection)
  - **Down**: Kết quả sau max pooling, truyền xuống layer tiếp theo

Quá trình này được lặp lại nhiều lần (thường 4-5 lần), mỗi lần giảm kích thước không gian và tăng số kênh.


```python
class DoubleConv(nn.Module):
    """A helper module that consists of two convolution layers, each followed by a batch normalization and a ReLU activation."""

    def __init__(self, in_channels, out_channels):
        super(DoubleConv, self).__init__()
        self.conv = nn.Sequential(
            nn.Conv2d(
                in_channels=in_channels,
                out_channels=out_channels,
                kernel_size=3,
                stride=1,
                padding=1,
            ),
            nn.BatchNorm2d(out_channels),
            nn.ReLU(),
            nn.Conv2d(
                in_channels=out_channels,
                out_channels=out_channels,
                kernel_size=3,
                stride=1,
                padding=1,
            ),
            nn.BatchNorm2d(out_channels),
            nn.ReLU(),
        )

    def forward(self, x):
        """Returns the output of the double convolution block."""
        return self.conv(x)

class EncoderBlock(nn.Module):
    """An encoder block consists of a double convolution followed by a max pooling layer."""

    def __init__(self, in_channels, out_channels):
        super(EncoderBlock, self).__init__()
        self.double_conv = DoubleConv(in_channels, out_channels)
        self.max_pool = nn.MaxPool2d(kernel_size=2, stride=2)

    def forward(self, x):
        """Returns the output of the double convolution (for skip connection) and the output of the max pooling (for downsampling)."""
        skip = self.double_conv(x)
        down = self.max_pool(skip)
        return skip, down
```

### 35.4.2. Lớp Bottleneck

**Bottleneck** là lớp nằm ở điểm sâu nhất của mạng UNet, nơi mà kích thước không gian đạt giá trị nhỏ nhất nhưng số kênh đạt giá trị lớn nhất. Lớp này đóng vai trò là cầu nối giữa encoder và decoder, chứa thông tin đặc trưng trừu tượng cấp cao nhất về nội dung của ảnh. Trong cài đặt của chúng ta, bottleneck đơn giản là một DoubleConv block với số kênh lớn nhất (thường là 1024).


```python
class Bottleneck(nn.Module):
    """The bottleneck block consists of a double convolution."""

    def __init__(self, in_channels, out_channels):
        super(Bottleneck, self).__init__()
        self.double_conv = DoubleConv(in_channels, out_channels)

    def forward(self, x):
        return self.double_conv(x)
```

### 35.4.3. Module cho decoder

**Decoder** có nhiệm vụ ngược lại với encoder: tăng dần kích thước không gian của ảnh về kích thước ban đầu để tạo ra mask phân đoạn:

- **Upsampling**: Sử dụng transposed convolution (còn gọi là deconvolution) để tăng kích thước ảnh lên gấp đôi

- **Skip Connection**: Kết nối với output tương ứng từ encoder thông qua phép concatenation. Điều này giúp decoder có thể truy cập trực tiếp vào thông tin chi tiết từ các lớp encoder, giải quyết vấn đề mất thông tin không gian trong quá trình downsampling.

- **DoubleConv**: Tinh chỉnh các đặc trưng sau khi kết hợp thông tin từ encoder và decoder

Quá trình này được lặp lại đối xứng với encoder, cuối cùng tạo ra output có cùng kích thước với ảnh đầu vào.


```python
class DecoderBlock(nn.Module):
    """A decoder block consists of a transposed convolution followed by a double convolution."""

    def __init__(self, in_channels, out_channels):
        super(DecoderBlock, self).__init__()
        self.up_conv = nn.ConvTranspose2d(
            in_channels=in_channels, out_channels=out_channels, kernel_size=2, stride=2
        )
        self.double_conv = DoubleConv(in_channels, out_channels)

    def forward(self, x, skip):
        """Returns the output of the double convolution after concatenating the upsampled input and the skip connection."""
        x = self.up_conv(x)
        if x.shape[2:] != skip.shape[2:]:
            x = TF.interpolate(
                x, size=skip.shape[2:], mode="bilinear", align_corners=False
            )
        x = torch.cat((skip, x), dim=1)
        return self.double_conv(x)
```

### 35.4.4. Ghép các khối theo cấu trúc UNet

Bây giờ chúng ta sẽ lắp ráp tất cả các thành phần đã xây dựng thành một mô hình UNet hoàn chỉnh. Mô hình này bao gồm:

- **4 EncoderBlocks**: Giảm kích thước từ (H, W) → (H/2, W/2) → (H/4, W/4) → (H/8, W/8) → (H/16, W/16)
- **1 Bottleneck**: Xử lý ở mức độ trừu tượng cao nhất
- **4 DecoderBlocks**: Tăng kích thước ngược lại từ (H/16, W/16) → (H/8, W/8) → (H/4, W/4) → (H/2, W/2) → (H, W)
- **1 Output Layer**: Convolution 1×1 để tạo mask cuối cùng với số kênh bằng số classes (1 cho binary segmentation)

Mô hình nhận đầu vào là ảnh với số kênh tùy chọn (ví dụ 5 kênh cho Sentinel-2: R, G, B, NIR, SWIR) và trả về mask phân đoạn với giá trị logits cho mỗi pixel.


```python

class UnetSegmentation(nn.Module):
    """The UNET architecture consists of an encoder, a bottleneck, and a decoder.
    Args:
    in_channels: The number of input channels (e.g., 3 for RGB images).
    out_classes: The number of output classes (e.g., 1 for binary segmentation).
    Returns:
    A tensor of shape (batch_size, out_classes, height, width) containing the predicted segmentation masks.
    """

    def __init__(self, in_channels=3, out_classes=1):
        super(UnetSegmentation, self).__init__()
        self.enc1 = EncoderBlock(in_channels, 64)
        self.enc2 = EncoderBlock(64, 128)
        self.enc3 = EncoderBlock(128, 256)
        self.enc4 = EncoderBlock(256, 512)
        self.bottleneck = Bottleneck(512, 1024)
        self.dec4 = DecoderBlock(1024, 512)
        self.dec3 = DecoderBlock(512, 256)
        self.dec2 = DecoderBlock(256, 128)
        self.dec1 = DecoderBlock(128, 64)
        self.outclass = nn.Conv2d(
            in_channels=64, out_channels=out_classes, kernel_size=1
        )

    def forward(self, x):
        """Returns the predicted segmentation masks for the input images."""
        skip1, down1 = self.enc1(x)
        skip2, down2 = self.enc2(down1)
        skip3, down3 = self.enc3(down2)
        skip4, down4 = self.enc4(down3)
        bottleneck = self.bottleneck(down4)
        up4 = self.dec4(bottleneck, skip4)
        up3 = self.dec3(up4, skip3)
        up2 = self.dec2(up3, skip2)
        up1 = self.dec1(up2, skip1)
        return self.outclass(up1)
```

## 35.5. Huấn luyện mô hình

Sau khi đã xây dựng xong mô hình UNet và dataset, bước tiếp theo là huấn luyện mô hình trên dữ liệu thực tế. Quá trình huấn luyện bao gồm các bước:

1. **Forward pass**: Đưa batch ảnh qua mô hình để dự đoán mask
2. **Tính loss**: So sánh mask dự đoán với mask thật sử dụng Binary Cross Entropy Loss
3. **Backward pass**: Tính gradient của loss theo các tham số mô hình
4. **Update weights**: Cập nhật tham số mô hình sử dụng optimizer (Adam)
5. **Validation**: Đánh giá mô hình trên tập validation để theo dõi quá trình học

Chúng ta sẽ chia dữ liệu thành training set (80%) và validation set (20%), sau đó huấn luyện mô hình qua nhiều epochs cho đến khi loss hội tụ.

### 35.5.1. Xác định thông số 

Trước khi bắt đầu huấn luyện, chúng ta cần thiết lập các thông số (hyperparameters) quan trọng:

- **Model**: Khởi tạo UNet với số kênh đầu vào (11 trong bài học này) và số classes đầu ra (1 cho binary segmentation)
- **Loss function**: BCEWithLogitsLoss - kết hợp sigmoid activation và binary cross entropy, hiệu quả cho bài toán phân loại nhị phân
- **Optimizer**: Adam với learning rate = 1e-4, một lựa chọn phổ biến cho deep learning
- **Device**: Sử dụng GPU nếu có (cuda) để tăng tốc độ huấn luyện, nếu không sẽ dùng CPU
- **Number of epochs**: Số lần duyệt qua toàn bộ dữ liệu huấn luyện (ví dụ 10 epochs)

Các thông số này có thể được điều chỉnh tùy theo dữ liệu và tài nguyên tính toán có sẵn.


```python
model = UnetSegmentation(in_channels=11, out_classes=1)
criterion = nn.BCEWithLogitsLoss()  # Binary Cross Entropy Loss with Logits
learning_rate = 1e-4
optimizer = torch.optim.Adam(model.parameters(), lr=learning_rate)
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model = model.to(device)
num_epochs = 10
batch_size = 4  # Adjust based on your GPU memory
```

### 35.5.2. Huấn luyện một epoch

Chúng ta xây dựng hai hàm chính cho quá trình huấn luyện:

**train_one_epoch()**: Thực hiện một epoch huấn luyện
- Chế độ `model.train()`: Bật dropout và batch normalization
- Duyệt qua tất cả batches trong training set
- Với mỗi batch: forward pass → tính loss → backward pass → update weights
- Trả về average loss của epoch

**validate_one_epoch()**: Đánh giá mô hình trên validation set
- Chế độ `model.eval()`: Tắt dropout và batch normalization
- Sử dụng `torch.inference_mode()` để tắt gradient computation (tiết kiệm bộ nhớ)
- Duyệt qua validation set và tính average loss
- Loss validation giúp phát hiện overfitting (khi train loss giảm nhưng val loss tăng)

Hai hàm này sẽ được gọi lặp lại qua nhiều epochs để huấn luyện mô hình.


```python
def train_one_epoch(model, train_data, optimizer, criterion, device):
    """Train the model for one epoch.
    
    Args:
        model (nn.Module): The model to be trained.
        train_data (DataLoader): The training data loader.
        optimizer (torch.optim.Optimizer): The optimizer for updating model parameters.
        criterion (nn.Module): The loss function.
        device (torch.device): The device to run the training on (CPU or GPU).
    Returns:
        float: The average loss for the epoch.
    """
    model.train()
    running_loss = 0.0
    for images, masks in train_data:
        images, masks = images.to(device), masks.to(device)
        optimizer.zero_grad()
        outputs = model(images)
        loss = criterion(outputs, masks)
        loss.backward()
        optimizer.step()
        running_loss += loss.item() * images.size(0)
    epoch_loss = running_loss / len(train_data.dataset)
    return epoch_loss

def validate_one_epoch(model, val_data, criterion, device):
    """Validate the model for one epoch.
    
    Args:
        model (nn.Module): The model to be validated.
        val_data (DataLoader): The validation data loader.
        criterion (nn.Module): The loss function.
        device (torch.device): The device to run the validation on (CPU or GPU).
    Returns:
        float: The average loss for the epoch.
    """
    model.eval()
    running_loss = 0.0
    with torch.inference_mode():
        for images, masks in val_data:
            images, masks = images.to(device), masks.to(device)
            outputs = model(images)
            loss = criterion(outputs, masks)
            running_loss += loss.item() * images.size(0)
    epoch_loss = running_loss / len(val_data.dataset)
    return epoch_loss
```

### 35.5.3. Huấn luyện nhiều epochs

Bây giờ chúng ta sẽ kết hợp tất cả các thành phần để huấn luyện mô hình qua nhiều epochs:

1. **Chuẩn bị dữ liệu**: Liệt kê tất cả file ảnh và mask, sau đó chia thành training (80%) và validation (20%) sets
2. **Tạo DataLoader**: Wrap datasets với DataLoader để có thể lấy dữ liệu theo batch, với batch_size = 4
3. **Training loop**: Lặp qua mỗi epoch:
   - Huấn luyện mô hình trên training set
   - Đánh giá mô hình trên validation set
   - In ra train loss và validation loss
   - Theo dõi quá trình học qua progress bar

Trong quá trình huấn luyện, chúng ta mong đợi cả train loss và validation loss đều giảm dần. Nếu validation loss bắt đầu tăng trong khi train loss vẫn giảm, đây là dấu hiệu của overfitting và chúng ta nên dừng huấn luyện sớm (early stopping).

- **Hàm liệt kê đường dẫn files**

Hàm liệt kê tất cả file paths từ một `directory` có kết thúc bằng `.tif` files. Đảm bảo tên các files ảnh và file mask nên giống nhau và tương ứng về mặt không gian khớp với nhau.


```python
def list_files(path):
    """List all files in a directory and its subdirectories.
    
    Args:
        path (str): The root directory to search for files.
    Returns:
        list: A list of file paths.
    """
    flist = []
    for root, dirs, files in os.walk(path):
        for file in files:
            if file.endswith('.tif'):
                flist.append(os.path.join(root, file))
    return sorted(flist)
```

- **Xây dựng Dataset và DataLoader**

Từ các hàm đã xây dựng bên trên, chúng ta sẽ liệt kê danh sách các files (img_files) từ folder chứa ảnh và ảnh nhãn (mask_files) từ thư mục dữ liệu. Sau đó, dữ liệu được chia thành hai phần: 80% cho huấn luyện (training) và 20% cho kiểm tra (validation) bằng hàm train_test_split().

Tiếp theo, các tập dữ liệu được tạo bằng lớp CustomDataset, có nhiệm vụ đọc và chuẩn bị ảnh cùng nhãn tương ứng cho mô hình.

Cuối cùng, DataLoader được sử dụng để nạp dữ liệu theo từng lô (batch). Tập huấn luyện sử dụng shuffle=True để xáo trộn dữ liệu, giúp mô hình học tốt hơn, trong khi tập validation sử dụng shuffle=False để giữ nguyên thứ tự dữ liệu khi đánh giá.


```python
img_files = list_files(r"G:\My Drive\python\geounet\dataset\images")
mask_files = list_files(r"G:\My Drive\python\geounet\dataset\masks")
# Split the dataset into training and validation sets
train_img_files, val_img_files, train_mask_files, val_mask_files = train_test_split(img_files, mask_files, test_size=0.2, random_state=42)

train_data = CustomDataset(train_img_files, train_mask_files)
val_data = CustomDataset(val_img_files, val_mask_files)
train_data = DataLoader(train_data, batch_size=batch_size, shuffle=True)
val_data = DataLoader(val_data, batch_size=batch_size, shuffle=False)
```

- **Huấn luyện mô hình**

Sau khi đã chuẩn bị các hàm chuẩn bị dữ liệu, mô hình, và huấn luyện một epoch. Giờ chúng ta huấn luyện nhiều epochs và theo dõi sai số của mô hình như bên dưới.


```python
progess_bar = tqdm(range(num_epochs), desc="Training Progress", unit="epoch")

for epoch in progess_bar:
    train_loss = train_one_epoch(model, train_data, optimizer, criterion, device)
    val_loss = validate_one_epoch(model, val_data, criterion, device)
    progess_bar.set_postfix({"Train Loss": train_loss, "Val Loss": val_loss})
```

### 35.5.4. Lưu lại mô hình đã huấn luyện

Sau khi hoàn thành huấn luyện, mô hình cần được **lưu lại** để có thể sử dụng sau này mà không phải train lại từ đầu. PyTorch cung cấp hai cách lưu mô hình:

- **Lưu toàn bộ mô hình** (`torch.save(model, path)`): Lưu cả kiến trúc lẫn trọng số vào một file duy nhất. Cách này phụ thuộc vào cấu trúc class Python khi tải lại, nên kém linh hoạt hơn.
- **Lưu chỉ trọng số** (`torch.save(model.state_dict(), path)`): Phương pháp được khuyến nghị - chỉ lưu các tham số (weights và biases) dưới dạng dictionary. Nhẹ hơn, linh hoạt hơn và không bị ràng buộc bởi cấu trúc file Python.


```python
# Lưu mô hình đã huấn luyện
torch.save(model.state_dict(), "G:\\My Drive\\python\\geocourse\\data\\unet_model.pth")
```

## 35.6. Sử dụng mô hình đã huấn luyện để dự đoán vào dữ liệu thực tế

Sau khi hoàn thành quá trình huấn luyện, mô hình UNet đã học được cách phân biệt trang trại điện mặt trời từ các đặc trưng phổ và không gian của ảnh Sentinel-2. Bước tiếp theo là đưa mô hình vào thực tế - giai đoạn này gọi là **inference** (suy luận). Không giống như huấn luyện, inference chỉ cần forward pass nên tương đối nhanh và có thể chạy trên cả CPU.

Quy trình inference trong bài toán này gồm bốn bước chính:

1. **Đọc và tiền xử lý ảnh**: Tải ảnh Sentinel-2 mới từ Planetary Computer, áp dụng đúng quy trình chuẩn hóa như khi tạo dữ liệu huấn luyện
2. **Dự đoán theo patch**: Chia ảnh lớn thành các patch 128×128, chạy mô hình trên từng patch rồi ghép lại thành mask toàn cục
3. **Trực quan hóa và lưu kết quả**: Hiển thị mask dự đoán chồng lên ảnh RGB (bước này chỉ để kiểm tra)

### 35.6.1. Tải mô hình đã huấn luyện

Khi **tải lại**, ta khởi tạo mô hình với **cùng kiến trúc và cùng số kênh đầu vào**, sau đó dùng `load_state_dict()` để nạp trọng số đã lưu. Một số lưu ý quan trọng:
- Tham số `map_location` cho phép tải mô hình được huấn luyện trên GPU về CPU (hoặc ngược lại) một cách linh hoạt.
- Tham số `weights_only=True` (PyTorch ≥ 2.0) được khuyến nghị để chỉ tải trọng số, tránh thực thi mã tùy ý từ file pickle.
- Sau khi tải, phải gọi `model.eval()` để chuyển sang chế độ inference (tắt dropout và batch normalization).  


```python
# --- Lưu trọng số mô hình sau huấn luyện ---
model_save_path = "G:\\My Drive\\python\\geocourse\\data\\unet_model.pth"

# --- Tải lại mô hình để inference ---
# (Có thể thực hiện trong một session mới hoặc sau khi khởi động lại kernel)
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
model = UnetSegmentation(in_channels=11, out_classes=1)
model.load_state_dict(torch.load(model_save_path, map_location=device, weights_only=True))
model = model.to(device)
print("Model loaded successfully for inference.")
```

### 35.6.2. Hàm tải và tiền xử lý ảnh Sentinel-2

Để dự đoán trên dữ liệu mới, ta xây dựng hàm `load_sen2_data()` thực hiện toàn bộ pipeline tải và tiền xử lý ảnh. Điều quan trọng nhất là **phải áp dụng cùng quy trình tiền xử lý** như khi chuẩn bị dữ liệu huấn luyện. Nếu dữ liệu inference có phân phối khác so với dữ liệu huấn luyện (gọi là **distribution shift** hay **data mismatch**), chất lượng dự đoán sẽ giảm đáng kể.

Hàm thực hiện pipeline đầy đủ qua các bước:

| Bước | Mô tả |
|------|-------|
| **Tìm kiếm ảnh** | Kết nối Planetary Computer STAC API, lọc theo bbox, thời gian và tỷ lệ mây (`< 5%`) |
| **Xử lý offset** | Ảnh Sentinel-2 sau ngày 2022-01-25 có thêm offset 1000 - cần trừ đi trước khi tính reflectance |
| **Tính reflectance** | Chia cho 10000 để chuyển giá trị DN về reflectance trong khoảng [0, 1] |
| **Tính NDVI** | Thêm kênh NDVI = (B08 − B04) / (B08 + B04) vào dataset |
| **Tạo median composite** | Lấy giá trị median theo trục thời gian để loại bỏ mây còn sót và giảm nhiễu |
| **Chuẩn hóa** | Clip về khoảng percentile 2%–98% rồi normalize về [0, 1] để giảm ảnh hưởng outlier |

Khu vực dự đoán (`bbox`) có thể **khác hoàn toàn** so với khu vực dữ liệu huấn luyện, miễn là cùng loại cảnh quan và ảnh đạt tiêu chuẩn chất lượng.


```python
def load_sen2_data(bbox, start_date='2026-03-20', end_date='2026-03-28', max_cloud_cover=5):
    """Load Sentinel-2 data for a given bounding box and date range.
    
    Args:
        bbox (list): A list of four coordinates [min_lon, min_lat, max_lon, max_lat] defining the bounding box.
        start_date (str): The start date for the data in 'YYYY-MM-DD' format
        end_date (str): The end date for the data in 'YYYY-MM-DD' format
        max_cloud_cover (int): The maximum cloud cover percentage for filtering images.
    Returns:
        xarray.DataArray: A data array containing the loaded Sentinel-2 data, with bands normalized to 0-1 range and NDVI calculated.
    """
    import planetary_computer as pc
    import pystac_client
    from odc.stac import stac_load
    import xarray as xr
    # Connect to the STAC API
    catalog = pystac_client.Client.open("https://planetarycomputer.microsoft.com/api/stac/v1")
    # Search for Sentinel-2 data
    search = catalog.search(
        collections=["sentinel-2-l2a"],
        bbox=bbox,
        datetime=f"{start_date}/{end_date}",
        query={"eo:cloud_cover": {"lt": max_cloud_cover}}
    )
    items = list(search.items())
    if not items:
        raise ValueError("No Sentinel-2 images found for the given parameters.")
    data = stac_load(
        items,
        bands=["B02", "B03", "B04", "B05", "B06", "B07", "B08", "B8A", "B11", "B12"],
        chunks={"x": 512, "y": 512},
        bbox=bbox,
        resolution=10,
        patch_url=pc.sign
    )
    # Handle the offset for images after 2022-01-25
    offset_value = 1000
    shift_date = np.datetime64("2022-01-25")
    time = data.time.values
    is_true = np.any(time > shift_date)
    if is_true:
        old_time = time[time < shift_date]
        new_time = time[time >= shift_date]
        old_dataset = data.sel(time=old_time)
        new_dataset = (
            data.sel(time=new_time).clip(offset_value) - offset_value
        )
        data= xr.concat([old_dataset, new_dataset], dim="time")
    data = data/10000 # convert to reflectance
    data['ndvi'] = (data['B08'] - data['B04']) / (data['B08'] + data['B04'])
    data = data.median(dim='time').to_dataarray(dim='band')
    # get 2% and 98 percentiles for each band
    p2 = data.quantile(0.02, dim=['x', 'y'])
    p98 = data.quantile(0.98, dim=['x', 'y'])
    # clip the data to the 2% and 98% percentiles
    data = data.clip(p2, p98)
    # normalize the data to 0-1 range
    data = (data - p2) / (p98 - p2)
    return data
```

### 35.6.3. Dự đoán mask phân đoạn theo từng patch

Vì mô hình UNet được huấn luyện trên các patch nhỏ (128×128 pixel), ta không thể đưa ảnh toàn cảnh lớn vào mô hình trực tiếp do giới hạn bộ nhớ GPU/RAM. Thay vào đó, chiến lược **dự đoán theo patch (patch-based inference)** chia ảnh thành các patch nhỏ, xử lý từng patch riêng lẻ rồi ghép kết quả lại.

Hai hàm được xây dựng cho pipeline này:

**`predict_binary_mask_patch_by_pytorch()`** - xử lý **một patch đơn lẻ**:
- Nhận vào mảng numpy hoặc DataArray có kích thước `(band, H, W)`
- Đưa qua mô hình để lấy logits, sau đó áp dụng `torch.sigmoid()` để chuyển về xác suất ∈ [0, 1]
- Ngưỡng hóa (thresholding): pixel có xác suất > `threshold` (mặc định 0.5) → được phân loại là trang trại điện mặt trời (giá trị = 1)

**`generate_predicted_mask()`** - xử lý **toàn bộ ảnh** bằng cơ chế lazy:
- Chia DataArray thành các chunk 128×128 bằng Dask để xử lý song song, tiết kiệm bộ nhớ
- Dùng `dask.array.map_overlap()` để áp dụng hàm prediction lên từng chunk, tự động xử lý các chunk biên
- Tham số `overlap`: thêm vùng đệm (pixels) tại ranh giới giữa các patch để giảm **boundary artifacts** — hiệu ứng sai lệch tại các đường cắt patch
- Ghép tất cả kết quả lại thành một DataArray duy nhất với đầy đủ tọa độ địa lý (y, x)

> **Lưu ý kỹ thuật**: `chunk_size` khi inference **nên bằng** `patch_size` lúc huấn luyện (128). Nếu gặp lỗi `CUDA out of memory`, giảm cả `chunk_size` và `batch_size` xuống. Sau khi gọi `generate_predicted_mask()`, nhớ gọi `.compute()` để trigger tính toán thực sự từ Dask lazy graph.

- **Hàm dự đoán trên patch và toàn bộ ảnh**


```python
def predict_binary_mask_patch_by_pytorch(
    model,
    data,
    threshold=0.5,
    device=None,
):
    """
    Predict a binary mask from a single image patch.

    Parameters
    ----------
    model : torch.nn.Module
    data : np.ndarray or xr.DataArray
        Shape: (band, y, x) or (y, x)
    threshold : float
    device : torch.device

    Returns
    -------
    np.ndarray
        Binary mask of shape (y, x)
    """
    if isinstance(data, xr.DataArray):
        data = data.values

    data = np.nan_to_num(data).astype(np.float32)

    # (y,x) -> (1,y,x)
    if data.ndim == 2:
        data = data[np.newaxis, ...] # add a new axis to make it (1, y, x)

    # (band,y,x) -> (1,band,y,x)
    if data.ndim == 3:
        data = data[np.newaxis, ...] # add a new axis to make it (1, band, y, x)

    x = torch.from_numpy(data).to(device)

    model.eval()
    with torch.inference_mode():
        pred = model(x)

    pred = torch.sigmoid(pred)

    pred = pred.squeeze().cpu().numpy()

    return (pred > threshold).astype(np.uint8)

def generate_predicted_mask(
    model,
    data,
    threshold=0.5,
    overlap=0,
    chunk_size=128,
    device=None,
):
    """
    Predict an entire raster using Dask map_overlap.

    Parameters
    ----------
    data : xr.DataArray
        Dimensions must be ('band', 'y', 'x')
    """

    if device is None:
        device = torch.device(
            "cuda" if torch.cuda.is_available() else "cpu"
        )

    model = model.to(device)

    data = data.chunk(
        {
            "band": -1,
            "y": chunk_size,
            "x": chunk_size,
        }
    )

    pred = data.data.map_overlap(
        lambda block: predict_binary_mask_patch_by_pytorch(
            model=model,
            data=block,
            threshold=threshold,
            device=device,
        ),
        depth={1: overlap, 2: overlap},
        boundary="reflect",
        trim=True,
        drop_axis=0,          # remove band dimension
        dtype=np.uint8,
    )

    return xr.DataArray(
        pred,
        dims=("y", "x"),
        coords={
            "y": data.y,
            "x": data.x,
        },
        name="prediction",
    )
```

- **Dự đoán cho khu vực lựa chọn**


```python
# Xác định khu vực nghiên cứu (bounding box: [min_lon, min_lat, max_lon, max_lat])
bbox = [107.82491814, 12.54327821, 107.91316082, 12.62476045]

# Tải và tiền xử lý ảnh Sentinel-2 từ Planetary Computer
data = load_sen2_data(
    bbox,
    start_date='2026-03-20',
    end_date='2026-03-28',
    max_cloud_cover=5
).astype(np.float32)
```


```python
# Chạy dự đoán trên toàn bộ ảnh theo từng patch 128×128
pred_mask = generate_predicted_mask(
    model, 
    data,
    threshold=0.5,
    device=device,
    chunk_size=256,
    overlap=10
)

# Trigger tính toán thực sự (Dask lazy → eager computation)
pred_mask = pred_mask.compute()
```

### 35.6.4. Trực quan hóa và lưu kết quả

Bước cuối cùng là trực quan hóa để kiểm tra chất lượng dự đoán và lưu kết quả ra file GeoTIFF. Ngoài ra, chúng ta có thể thực hiện thêm các tính toán thống kê từ ảnh dự đoán nếu muốn, như diện tích, phân bố, etc.


```python
pred_mask.plot()
```


```python
import rioxarray  # cần import để kích hoạt accessor .rio trên xarray DataArray

# Đặt CRS từ dữ liệu gốc sang mask dự đoán (cần thiết để lưu đúng thông tin địa lý)
pred_mask = pred_mask.rio.write_crs(data.rio.crs)

# Lưu mask dự đoán ra file GeoTIFF với tọa độ địa lý
output_path = "predicted_solar_farms.tif"
pred_mask.rio.to_raster(output_path, compress='LZW')
```

# Tóm tắt

Bạn đã hoàn thành Bài 35 và học được cách áp dụng học sâu với kiến trúc UNet để giải quyết bài toán phân đoạn ảnh vệ tinh - một kỹ năng quan trọng trong ứng dụng AI cho viễn thám và giám sát môi trường. Đây là nền tảng để bạn có thể mở rộng sang các bài toán phân đoạn phức tạp hơn như phát hiện nhiều lớp đối tượng, theo dõi biến đổi đất đai, hoặc giám sát thiên tai.

### Các khái niệm chính đã nắm vững:
- ✅ **Kiến trúc UNet**: Hiểu rõ cấu trúc encoder-decoder với skip connections, tại sao UNet hiệu quả cho phân đoạn ảnh
- ✅ **Xây dựng Dataset tùy chỉnh**: Tạo CustomDataset để đọc và xử lý ảnh GeoTIFF, hỗ trợ data augmentation
- ✅ **Các thành phần của CNN**: Convolution, batch normalization, ReLU, max pooling, transposed convolution
- ✅ **Encoder và Decoder**: Cài đặt encoder để trích xuất đặc trưng và decoder để phục hồi kích thước ảnh
- ✅ **Skip Connections**: Hiểu vai trò của skip connections trong việc giữ lại thông tin không gian chi tiết
- ✅ **Quy trình huấn luyện**: Forward pass, loss computation, backpropagation, weight update
- ✅ **Binary Segmentation**: Sử dụng BCEWithLogitsLoss và optimizer Adam cho bài toán phân đoạn nhị phân
- ✅ **Training và Validation**: Chia dữ liệu, huấn luyện qua nhiều epochs và theo dõi validation loss để phát hiện overfitting
- ✅ **Lưu và tải mô hình**: Lưu state_dict và tải lại mô hình an toàn với `weights_only=True`
- ✅ **Patch-based Inference**: Dự đoán trên ảnh lớn bằng chiến lược chia patch, sử dụng Dask để xử lý song song
- ✅ **PyTorch Framework**: Làm quen với PyTorch để xây dựng và huấn luyện mô hình deep learning

### Điểm mạnh của phương pháp Deep Learning cho phân đoạn ảnh vệ tinh:

| Ưu điểm | Giải thích |
|---------|------------|
| **Tự động trích xuất đặc trưng** | Không cần thiết kế đặc trưng thủ công, mô hình tự học từ dữ liệu |
| **Xử lý dữ liệu đa kênh** | Dễ dàng tích hợp nhiều kênh phổ (RGB, NIR, SWIR, NDVI) trong một mô hình |
| **Độ chính xác cao** | Với đủ dữ liệu huấn luyện, deep learning vượt trội so với phương pháp truyền thống |
| **Phát hiện pattern phức tạp** | Có thể học các mẫu hình học và phổ phức tạp khó định nghĩa bằng quy tắc |
| **Khả năng tổng quát hóa** | Mô hình được huấn luyện tốt có thể áp dụng cho nhiều khu vực khác nhau |

### Lưu ý khi áp dụng và mở rộng:
- Cần dữ liệu huấn luyện có nhãn chất lượng cao (ảnh + mask).Mở rộng khu vực nghiên cứu và thu thập thêm dữ liệu huấn luyện.
- Yêu cầu tài nguyên tính toán đáng kể (GPU) cho huấn luyện
- Cân nhắc data augmentation để tăng cường dữ liệu khi dataset nhỏ
- Theo dõi overfitting và áp dụng regularization nếu cần
- Đảm bảo tiền xử lý ảnh inference **nhất quán** với dữ liệu huấn luyện (tránh distribution shift)
- Sử dụng mô hình cho các vùng nghiên cứu khác và đánh giá độ chính xác.
- Mô hình UNet có thể cải thiện thêm bằng cách xây dựng thêm các khối tích chập hoặc bớt đi.
- Có thể mở rộng sang kiến trúc nâng cao hơn như UNet++, DeepLabV3+, Attention UNet, hoặc Mask R-CNN.

Với kiến thức này, bạn có thể tùy chỉnh mô hình UNet cho các bài toán phân đoạn khác như phát hiện rừng, phân loại cây trồng, theo dõi nước mặt, hoặc giám sát đô thị hóa từ ảnh vệ tinh.
