# Bài 10. Phát triển và đóng gói Python

Mục tiêu của đóng gói Python là biến mã nguồn thành một sản phẩm hoàn chỉnh, dễ cài đặt, chia sẻ và tái sử dụng. Việc này giúp quản lý phụ thuộc rõ ràng, đảm bảo chương trình chạy ổn định trên nhiều môi trường, đồng thời hỗ trợ phân phối thư viện hoặc ứng dụng cho người khác một cách thuận tiện. Trong bài học này, chúng ta sẽ sử dụng `poetry` cho việc tạo gói python.

## 10.1. Mục tiêu học tập
- Hiểu cách tổ chức và cấu trúc một dự án Python
- Quản lý thư viện và môi trường làm việc (pip, venv)
- Đóng gói dự án thành thư viện hoặc ứng dụng có thể cài đặt
- Phân phối và chia sẻ package Python cho người khác
- Sử dụng poetry cho việc tạo và quản lý gói python.

## 10.2. Tạo Dự án Mới với `Poetry`

Chúng ta sẽ tạo một gói ví dụ tên `geomath` - một thư viện đơn giản cho việc tính toán khoảng cách giữa hai điểm sử dụng công thức haversine.

### 10.2.1. Tạo môi trường ảo 

Trước khi tạo dự án Python, chúng ta nên tạo ra một môi trường ảo cho dự án.

```bash
conda create -n geomath python=3.12 -y
```
Sau khi tạo ra môi trường ảo `geomath`, chúng ta kích hoạt nó.

```bash
conda activate geomath
```

### 10.2.2. Khởi tạo dự án

Tạo dự án/gói python bằng `poetry` bằng câu lệnh `poetry new tengoi`

```bash
# Tạo dự án mới với Poetry (chạy trong Terminal)
# cc tới folder nơi chúng ta muốn tạo gói geomath, sau đó thực hiện câu lệnh như sau.
poetry new geomath
# khởi tạo trong thư mục có sẵn
poetry init
```

Poetry sẽ tạo ra cấu trúc thư mục như sau:

```
geomath/
├── pyproject.toml       ← File cấu hình chính (tên, version, dependencies)
├── README.md            ← Tài liệu dự án
├── src/geomath/            ← Thư mục chứa mã nguồn (package)
│   └── __init__.py
└── tests/               ← Thư mục chứa tests
    └── __init__.py
```

### 10.2.3. Tìm hiểu file `pyproject.toml`

Đây là file quan trọng nhất, thay thế cho `setup.py`, `setup.cfg`, `requirements.txt`, và `MANIFEST.in` của cách cũ. Bạn có thể xem file này trong folder `geomath`.

```
[project]
name = "geomath"
version = "0.1.0"
description = "Bạn nên viết mô tả gói tại đây"
authors = [
    {name = "tên của bạn",email = "youremail@gmail.com"}
]
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
]

[tool.poetry]
packages = [{include = "geomath", from = "src"}]

[build-system]
requires = ["poetry-core>=2.0.0,<3.0.0"]
build-backend = "poetry.core.masonry.api"
```

**Giải thích các phần chính:**

- **`[tool.poetry]`**: Metadata của gói (tên, phiên bản, mô tả, tác giả...)
- **`[tool.poetry.dependencies]`**: Các thư viện cần thiết để **chạy** gói

## 10.5. Quản lý Dependencies với Poetry

Poetry giải quyết một vấn đề lớn của Python: **xung đột dependency**. Nó tự động tính toán tổ hợp phiên bản tương thích cho tất cả các thư viện.


```python
# Tổng hợp các lệnh quản lý dependency thường dùng (chạy trong Terminal)
dependency_commands = {
    "Thêm thư viện vào dự án":                   "poetry add numpy shapely",
    "Thêm thư viện chỉ dùng khi dev":            "poetry add --group dev pytest black",
    "Xóa thư viện":                              "poetry remove shapely",
    "Cài tất cả dependencies từ pyproject.toml":  "poetry install",
    "Cài chỉ dependencies production (bỏ dev)":  "poetry install --without dev",
    "Cập nhật tất cả lên phiên bản mới nhất":    "poetry update",
}

for action, command in dependency_commands.items():
    print(f"# {action}")
    print(f"  $ {command}\n")
```

    # Thêm thư viện vào dự án
      $ poetry add numpy shapely
    
    # Thêm thư viện chỉ dùng khi dev
      $ poetry add --group dev pytest black
    
    # Xóa thư viện
      $ poetry remove shapely
    
    # Cài tất cả dependencies từ pyproject.toml
      $ poetry install
    
    # Cài chỉ dependencies production (bỏ dev)
      $ poetry install --without dev
    
    # Cập nhật tất cả lên phiên bản mới nhất
      $ poetry update
    
    

### 10.5.1. File `poetry.lock`

Sau khi chạy `poetry install` hoặc `poetry add`, Poetry tạo ra file `poetry.lock` — lưu **chính xác phiên bản** của mọi thư viện (kể cả phụ thuộc của phụ thuộc).

```
geomath/
├── pyproject.toml   ← Khai báo yêu cầu (ví dụ: numpy ^1.24)
├── poetry.lock      ← Phiên bản chính xác đã khóa (ví dụ: numpy 1.26.4)
└── ...
```

**Quy tắc quan trọng:**
- ✅ Luôn **commit** `poetry.lock` vào Git để đảm bảo mọi thành viên nhóm dùng cùng phiên bản
- ✅ Dùng `poetry install` để cài từ lock file, không dùng `pip install` trực tiếp
- ✅ Dùng `poetry update` khi muốn nâng cấp lên phiên bản mới hơn

## 10.6. Cấu trúc Mã nguồn và Viết Code

Hãy xây dựng thực tế gói `geomath` với một số chức năng hữu ích.

### 10.6.1. Cấu trúc thư mục mở rộng

Trong ví dụ này, thư mục bao gồm các tệp được tạo tự động theo cấu trúc của `poetry`, cùng với các tệp do người dùng bổ sung như `coordinates.py`, `distance.py`, `utils.py`, v.v. Những tệp do người dùng tạo này được gọi là các module, tức là các đơn vị mã nguồn đảm nhiệm một chức năng cụ thể. Trong vị dụ này, mình tạo ra 3 files trong geomath, bạn có thể tạo ra nhiều files khác theo mục đích của bạn. Hiện tại, `tests` folder có thể bỏ qua.

```
geomath/
├── pyproject.toml
├── README.md
├── poetry.lock
├── .gitignore # tạo thêm .gitignore file để bỏ qua các files cho giai đoạn sau khi dùng git.
├── geomath/                    ← Package chính
│   ├── __init__.py              ← Khai báo public API
│   ├── coordinates.py           ← Module xử lý tọa độ # Tạo thêm file `coordinates.py`
│   ├── distance.py              ← Module tính khoảng cách # Tạo file `distance.py`
│   └── utils.py                 ← Hàm tiện ích # Tạo file `utils.py`. 
└── tests/                       ← Tests
    ├── __init__.py
    ├── test_coordinates.py
    └── test_distance.py
```


```python
# Nội dung file: geoutils/utils.py
utils = '''
"""
Module chứa các hàm tiện ích chung cho geomath.
"""
def is_valid_lat_lon(lat, lon):
    """
    Kiểm tra xem giá trị latitude và longitude có hợp lệ không.
    Latitude hợp lệ: -90 <= lat <= 90
    Longitude hợp lệ: -180 <= lon <= 180
    Args:
        lat (float): Giá trị latitude cần kiểm tra.
        lon (float): Giá trị longitude cần kiểm tra.
    Returns:
        bool: True nếu cả latitude và longitude đều hợp lệ, False nếu không.
    """
    try:
        lat = float(lat)
        lon = float(lon)
    except (TypeError, ValueError):
        return False
    return -90 <= lat <= 90 and -180 <= lon <= 180
'''
```


```python
# Nội dung file: geomath/coordinates.py
coordinates_py = '''
"""Module xử lý và chuyển đổi tọa độ địa lý."""

import geomath.utils as utils

class Point:
    """class biểu diễn một điểm trong tọa độ địa lý (kinh độ và vĩ độ)."""

    def __init__(self, longitude: float, latitude: float):
        if not utils.is_valid_lat_lon(latitude, longitude):
            raise ValueError(f"Latitude hoặc longitude không hợp lệ: {latitude}, {longitude}")
        self.longitude = longitude
        self.latitude = latitude

    def __repr__(self):
        return f"Point(longitude={self.longitude}, latitude={self.latitude})"

    @classmethod
    def from_lon_lat(cls, longitude: float, latitude: float):
        """Tạo một instance của Point từ kinh độ và vĩ độ."""
        return cls(longitude, latitude)

'''
```


```python
# Nội dung file: geoutils/distance.py
distance_py = '''
"""Module tính khoảng cách giữa các điểm địa lý."""

from math import atan2, cos, radians, sin, sqrt

def haversine_distance(point1, point2):
    """
    Tính khoảng cách Haversine giữa hai điểm trên bề mặt Trái Đất.

    Args:
        point1 (Point): Điểm thứ nhất, có thuộc tính latitude và longitude (đơn vị độ).
        point2 (Point): Điểm thứ hai, có thuộc tính latitude và longitude (đơn vị độ).

    Returns:
        float: Khoảng cách giữa hai điểm (đơn vị mét).
    """
    # Bán kính Trái Đất (mét)
    R = 6371000
    lat1 = radians(point1.latitude)
    lon1 = radians(point1.longitude)
    lat2 = radians(point2.latitude)
    lon2 = radians(point2.longitude)

    dlat = lat2 - lat1
    dlon = lon2 - lon1

    a = sin(dlat / 2) ** 2 + cos(lat1) * cos(lat2) * sin(dlon / 2) ** 2
    c = 2 * atan2(sqrt(a), sqrt(1 - a))

    distance = R * c
    return round(distance, 2)

'''
```


```python
# Nội dung file: geoutils/__init__.py — định nghĩa Public API của gói
init_py = '''
"""
geomath: Thư viện tính khoảng cách địa lý sử dụng các công thức haversine.

Ví dụ sử dụng:
    from geomath import Point
    from geomath import distance

    # Khởi tạo hai điểm đại diện cho Hà Nội và Thành phố Hồ Chí Minh
    hanoi = Point.from_lon_lat(105.8342, 21.0278)
    hcmc  = Point.from_lon_lat(106.6602, 10.7769)
    # Tính khoảng cách giữa hai điểm sử dụng công thức Haversine
    print(distance.haversine_distance(hanoi, hcmc))  # 1143254.24 (meters)
"""
# Import các hàm thường dùng để người dùng có thể gọi trực tiếp từ geomath
from .coordinates import *
from .utils import *
'''
```

### 10.6.2. Cài đặt gói trên máy và kiểm tra 

Dưới đây là cách cài đặt gói `geomath` vào máy và kiểm tra chạy thử xem có lỗi gì không.

- **Cài đặt gói trên máy local (chạy trong Terminal)**
```
$ cd path/to/geomath
$ poetry install
```

- **Kiểm tra và chạy thử gói**


```python
from geomath import Point
from geomath import distance

# Khởi tạo hai điểm đại diện cho Hà Nội và Thành phố Hồ Chí Minh
hanoi = Point.from_lon_lat(105.8342, 21.0278)
hcmc  = Point.from_lon_lat(106.6602, 10.7769)
# Tính khoảng cách giữa hai điểm sử dụng công thức Haversine
print(distance.haversine_distance(hanoi, hcmc))  # 1143254.24 (meters)
```

## 10.7. Quản lý Phiên bản (Semantic Versioning)

Semantic Versioning (SemVer) là tiêu chuẩn đặt tên phiên bản: `MAJOR.MINOR.PATCH`

| Phần | Khi nào tăng | Ví dụ |
|---|---|---|
| **MAJOR** | Thay đổi không tương thích ngược (breaking change) | `1.0.0` → `2.0.0` |
| **MINOR** | Thêm tính năng mới, tương thích ngược | `1.0.0` → `1.1.0` |
| **PATCH** | Sửa lỗi, không thêm tính năng | `1.0.0` → `1.0.1` |

**Ví dụ thực tế:**
- `0.1.0`: Phiên bản alpha đầu tiên, đang phát triển
- `1.0.0`: Phiên bản ổn định đầu tiên, sẵn sàng dùng production
- `1.1.0`: Thêm hàm `nearest_city` mới
- `1.1.1`: Sửa lỗi tính toán trong `haversine`
- `2.0.0`: Đổi tên hàm, không tương thích với v1.x


```python
# Poetry quản lý version tự động — các lệnh bump version (chạy trong Terminal)
version_commands = {
    "Xem phiên bản hiện tại":              "poetry version",
    "Tăng PATCH (1.0.0 → 1.0.1)":         "poetry version patch",
    "Tăng MINOR (1.0.0 → 1.1.0)":         "poetry version minor",
    "Tăng MAJOR (1.0.0 → 2.0.0)":         "poetry version major",
    "Đặt phiên bản cụ thể":               "poetry version 1.2.3",
    "Phiên bản pre-release (alpha)":       "poetry version prerelease",   # 1.0.0 → 1.0.1a0
    "Phiên bản release candidate":         "poetry version 2.0.0rc1",
}

for action, command in version_commands.items():
    print(f"# {action}")
    print(f"  $ {command}\n")
```

    # Xem phiên bản hiện tại
      $ poetry version
    
    # Tăng PATCH (1.0.0 → 1.0.1)
      $ poetry version patch
    
    # Tăng MINOR (1.0.0 → 1.1.0)
      $ poetry version minor
    
    # Tăng MAJOR (1.0.0 → 2.0.0)
      $ poetry version major
    
    # Đặt phiên bản cụ thể
      $ poetry version 1.2.3
    
    # Phiên bản pre-release (alpha)
      $ poetry version prerelease
    
    # Phiên bản release candidate
      $ poetry version 2.0.0rc1
    
    

### 10.7.1. Git Tagging theo phiên bản

```bash
# Sau khi bump version và commit, tạo git tag tương ứng
git add pyproject.toml geomath/__init__.py
git commit -m "chore: bump version to 1.1.0"
git tag v1.1.0
git push origin main --tags
```

**Convention commit message (Angular convention — khuyến nghị):**

| Loại | Ý nghĩa | Ảnh hưởng version |
|---|---|---|
| `feat:` | Tính năng mới | MINOR tăng |
| `fix:` | Sửa lỗi | PATCH tăng |
| `BREAKING CHANGE:` | Thay đổi lớn | MAJOR tăng |
| `docs:` | Cập nhật tài liệu | Không tăng |
| `chore:` | Tác vụ bảo trì | Không tăng |
| `test:` | Thêm/sửa tests | Không tăng |

## 10.8. Xuất bản lên GitHub

GitHub là nơi lưu trữ mã nguồn và quản lý dự án. Dưới đây là quy trình đầy đủ từ đầu.

### 10.8.1. Chuẩn bị `.gitignore`


```python
# Nội dung file .gitignore chuẩn cho dự án Python/Poetry
gitignore_content = """
# Môi trường ảo Poetry
.venv/

# Bytecode Python
__pycache__/
*.py[cod]
*.pyo

# Build artifacts
dist/
build/
*.egg-info/

# Coverage reports
.coverage
htmlcov/
.pytest_cache/

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db

# Biến môi trường bí mật (API keys, tokens — KHÔNG BAO GIỜ commit)
.env
.env.local
"""
```


```python
# Ví dụ README.md chuyên nghiệp cho gói Python
readme_content = """
# geomath

[![PyPI version](https://badge.fury.io/py/geomath.svg)](https://badge.fury.io/py/geomath)
[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Tests](https://github.com/nguyenvana/geomath/actions/workflows/tests.yml/badge.svg)](https://github.com/nguyenvana/geomath/actions)

Thư viện tiện ích xử lý dữ liệu địa lý cho Việt Nam.

## Cài đặt

```bash
pip install geomath
```
## Sử dụng nhanh

```python
    from geomath import Point
    from geomath import distance

# Tính khoảng cách Hà Nội - TP.HCM
    hanoi = Point.from_lon_lat(105.8342, 21.0278)
    hcmc  = Point.from_lon_lat(106.6602, 10.7769)
    dist = distance.haversine_distance(hanoi, hcmc)  # 1143254.24 (meters)
```

## License
MIT
"""
```

### 10.8.2. Đẩy code lên GitHub

```bash
# 1. Khởi tạo Git repository (chạy một lần duy nhất)
cd geomath
git init
git add .
git commit -m "feat: initial release v0.1.0"

# 2. Tạo repository trên github.com, sau đó kết nối
git remote add origin https://github.com/your-username/geomath.git
git branch -M main
git push -u origin main

# 3. Đánh tag phiên bản đầu tiên
git tag v0.1.0
git push origin v0.1.0
```

## Tóm tắt

Bạn đã hoàn thành Bài 10 và nắm vững toàn bộ quy trình phát triển và đóng gói Python chuyên nghiệp.

### Các khái niệm chính đã nắm vững:
- ✅ **Poetry**: Công cụ quản lý dự án, dependency, build và publish toàn diện
- ✅ **pyproject.toml**: File cấu hình duy nhất thay thế setup.py và requirements.txt
- ✅ **Cấu trúc dự án**: Tổ chức mã nguồn, tests theo chuẩn Python
- ✅ **pytest**: Viết và chạy tests, đo độ phủ code coverage
- ✅ **Semantic Versioning**: Quy tắc đặt tên phiên bản MAJOR.MINOR.PATCH
- ✅ **GitHub**: Lưu trữ mã nguồn, quản lý phiên bản bằng git tag và Release

### Kỹ năng bạn có thể áp dụng:
- Tạo và publish thư viện địa lý / phân tích dữ liệu của riêng bạn
- Đóng gói các công cụ xử lý dữ liệu để chia sẻ với đồng nghiệp
- Đóng góp vào các dự án open-source
- Thiết lập pipeline CI/CD tự động cho dự án nhóm
- Quản lý nhiều phiên bản gói trong môi trường production
