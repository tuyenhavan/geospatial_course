# Bài 35: Phân vùng mặt nước sử dụng học sâu và ảnh Sentinel-2


```python
import torch 
import torch.nn as nn
import torch.nn.functional as TF
```


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


class Bottleneck(nn.Module):
    """The bottleneck block consists of a double convolution."""

    def __init__(self, in_channels, out_channels):
        super(Bottleneck, self).__init__()
        self.double_conv = DoubleConv(in_channels, out_channels)

    def forward(self, x):
        return self.double_conv(x)


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


```python
image = torch.randn(1, 3, 128, 128)  # Example input image
up_conv = nn.ConvTranspose2d(
            in_channels=3, out_channels=64, kernel_size=2, stride=2
        )
output = up_conv(image)
print(output.shape)
```

    torch.Size([1, 64, 256, 256])
    

Comming soon
