# Bài 1: Cài đặt và Thiết lập Môi trường Python cho Phân tích dữ liệu không gian

Chào mừng bạn đến với chuỗi bài hướng dẫn sử dụng Python cơ bản! Trong bài học đầu tiên này, chúng ta sẽ cài đặt các công cụ cần thiết cho việc sử dụng python trong lĩnh vực địa không gian.

## 1.1. Mục tiêu học tập
Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Cài đặt và cấu hình VS Code làm môi trường làm việc
- Thiết lập Git để quản lý mã nguồn
- Cài đặt Python thông qua Miniconda
- Tạo và quản lý môi trường ảo bằng conda
- Cài đặt các thư viện cần thiết vào môi trường ảo
- Sử dụng Jupyter Notebook trong VS Code

## 1.2. Tải và cài đặt VS Code

Visual Studio Code (VS Code) là một trình soạn thảo mã nguồn mở miễn phí và mạnh mẽ, hoàn hảo cho việc soạn thảo code Python.

- **Bước 1:** Tải và cài đặt VS Code

1. **Tải VS Code**:
   - Truy cập: https://code.visualstudio.com/
   - Chọn phiên bản phù hợp với hệ điều hành của bạn:
     - Windows: `.exe` file
     - macOS: `.dmg` file
     - Linux: `.deb` hoặc `.rpm` file

2. **Cài đặt VS Code**:
   - **Windows**: Chạy file `.exe`, trong quá trình cài đặt, chọn :
     - ✅ Chọn "Add 'Open with Cod' action to Windows Explorer file context menu".
     - ✅ Chọn "Add 'Open with Code' action to Windows Explorer directory context menu".
     - ✅ Tất cả các thông số còn lại có thể chọn mặc định. 
   - **macOS**: Mở file `.dmg` và kéo VS Code vào thư mục Applications
   - **Linux**: Sử dụng package manager hoặc cài đặt từ file đã tải. Tham khảo thêm ở [đây](https://code.visualstudio.com/docs/setup/linux). 

- **Bước 2:** Cài đặt các Extensions cần thiết trong VS code

Mở VS Code và cài đặt các extensions sau (Ctrl+Shift+X để mở Extensions):

- **`Extensions cần thiết`**
1. **Python** (Microsoft): Extension chính để phát triển Python, cung cấp khả năng chạy và debug code Python, quản lý interpreter, syntax highlighting, và tích hợp terminal Python. Đây là extension bắt buộc cho mọi dự án Python.

2. **Jupyter** (Microsoft): Tích hợp hoàn chỉnh Jupyter Notebook vào VS Code, cho phép tạo, chỉnh sửa và chạy các notebook (.ipynb) trực tiếp trong editor. Hỗ trợ nhiều kernel khác nhau và hiển thị output trực quan.

3. **Pylance** (Microsoft): Language server tiên tiến cho Python, cung cấp IntelliSense thông minh, kiểm tra lỗi thời gian thực, type checking, auto-completion, và phân tích code static. Kế thừa từ Pyright và là công cụ phân tích code Python tốt nhất hiện tại.

- **`Extensions khuyên dùng`**
4. **autoDocstring** (Nils Werner): Tự động tạo docstring cho hàm và class theo chuẩn Google, NumPy hoặc Sphinx. Giúp viết tài liệu code nhanh chóng và chuẩn mực.
5. **Black Formatter** (Microsoft): Công cụ format code Python tự động theo chuẩn PEP 8. Giúp code nhất quán, dễ đọc và tuân thủ quy tắc coding style của Python.
6. **vscode-icons** (VSCode Icons Team): Thêm biểu tượng đẹp cho các loại file khác nhau trong VS Code, giúp dễ dàng nhận diện và điều hướng trong project. 
7. **GitHub Copilot Chat** (Microsoft): Trợ lý lập trình được tích hợp trực tiếp trong VS Code.

- **Bước 3:** Cấu hình cơ bản VS Code

1. **Mở Settings** (Ctrl+,):
   ```json
   {
     "python.defaultInterpreterPath": "python",
     "python.formatting.provider": "black",
     "editor.formatOnSave": true,
     "files.autoSave": "afterDelay"
   }
   ```

2. **Thiết lập phím tắt hữu ích**:
   - Ctrl+Shift+P: Command Palette
   - Ctrl+`: Mở Terminal
   - Shift+Enter: Chạy Python code trong file


## 1.3. Cài đặt Git

Git là hệ thống quản lý phiên bản phân tán, rất cần thiết cho việc phát triển phần mềm và quản lý dự án.

**Bước 1:** Tải và cài đặt Git
#### Windows
**Tải Git**:
   - Truy cập: https://git-scm.com/install/windows
   - Tải phiên bản 64-bit mới nhất

**Cài đặt Git**:
   - Chạy file `.exe` đã tải
   - Trong quá trình cài đặt, chọn các tùy chọn sau:
   - ✅ "Use Visual Studio Code as Git's default editor"
   - ✅ Các thông số khác chọn mặc định.
#### macOS:
```bash
# Sử dụng Homebrew (khuyến nghị)
brew install git
```
#### Linux (Ubuntu/Debian):
```bash
sudo apt update
sudo apt install git
```

**Bước 2:** Cấu hình Git lần đầu
Mở Terminal và chạy các lệnh sau:

```bash
# Thiết lập tên và email. Bạn Nên nhập tên GITHUB và Email đăng nhập GITHUB vào bên dưới.
git config --global user.name "yourname" # Ví dụ như tên đăng nhập github yourusername
git config --global user.email "youremail@gmail.com" # Dùng Email của bạn dùng để đăng nhập github.

# Thiết lập editor mặc định
git config --global core.editor "code --wait"

# Thiết lập branch mặc định
git config --global init.defaultBranch main

# Xem cấu hình
git config --list
```

**Bước 3:** Xác minh cài đặt Git

```bash
git --version
```

Bạn nên thấy output tương tự: `git version 2.x.x`

## 1.4. Cài đặt Miniconda (Khuyến nghị)

Miniconda là phiên bản tối giản của Anaconda, cung cấp Python và conda package manager mà không có các gói thư viện phụ. Miniconda miễn phí cho người dùng.

**Bước 1:** Tải Miniconda

#### Tất cả hệ điều hành:
- Truy cập: https://repo.anaconda.com/miniconda/
- Chọn phiên bản Python 3.12 (khuyến nghị vào 01.2026) cho hệ điều hành của bạn. Bạn nên chọn version thấp hơn 1 bậc so với phiên bản mới nhất để đảm bảo sự ổn định.

#### Lưu ý quan trọng:
- **Windows**: Tải file `.exe` 64-bit
- **macOS**: Tải file `.pkg` hoặc `.sh`
- **Linux**: Tải file `.sh`

**Bước 2:** Cài đặt Miniconda

#### Windows:
1. Chạy file `.exe` đã tải
2. Trong quá trình cài đặt:
   - ✅ "Just Me (recommended)" 
   - ❌ **KHÔNG** chọn "Add Miniconda3 to PATH" (dùng Anaconda Prompt)
   - ✅ Tất cả các thông số cài đặt khác chọn mặc định (khuyến nghị).

#### macOS:
```bash
# Nếu tải file .sh
bash Miniconda3-latest-MacOSX-x86_64.sh

# Hoặc double-click file .pkg và làm theo hướng dẫn
```

#### Linux:
```bash
# Tải và cài đặt
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

**Bước 3:** Xác minh cài đặt

Mở **Anaconda Prompt** (Windows) hoặc Terminal (macOS/Linux):

```bash
# Kiểm tra conda
conda --version
conda info

# Kiểm tra Python
python --version

```


## 1.5. Thiết lập môi trường ảo (Set Up Virtual Environment)

Môi trường ảo giúp cách ly các dự án và tránh xung đột giữa các phiên bản thư viện khác nhau. Chúng ta sẽ dùng miniconda để tạo và quản lý môi trường ảo và các gói python cài đặt.

**Bước 1:** Tạo môi trường mới

Mở **Anaconda Prompt** (Windows) hoặc Terminal và chạy:

```bash
# Tạo môi trường cho khóa học này. Ví dụ tạo môi trường tên `geotest`
conda create -n geotest python=3.12 -y # Lần đầu tạo môi trường ảo sẽ hiển thị các điều khoản. Làm theo hướng dẫn hiển thị nếu có.

# Liệt kê các môi trường
conda env list
```

**Bước 2:** Kích hoạt môi trường

```bash
# Kích hoạt môi trường
conda activate geotest

# Xác nhận môi trường đang hoạt động
conda info --envs
python --version
```

**Bước 3:** Thiết lập môi trường mặc định

#### Cách 1: Kích hoạt tự động (Windows)
Tạo file batch để tự động kích hoạt:
```batch
@echo off
call conda activate geotest
cmd /k
```

#### Cách 2: Thêm vào shell profile
```bash
# Thêm vào ~/.bashrc hoặc ~/.zshrc (macOS/Linux)
echo "conda activate geotest" >> ~/.bashrc
source ~/.bashrc
```

**Bước 4:** Quản lý môi trường

```bash
# Xem thông tin môi trường hiện tại
conda info

# Xuất danh sách packages
conda list --export > requirements.txt

# Xóa môi trường
conda env remove -n geotest --all  # Bạn có thể thay ptest bằng tên môi trường bạn đã tạo.
```

### Tại sao sử dụng môi trường ảo?

1. **Cách ly dự án**: Mỗi dự án có thư viện riêng biệt
2. **Quản lý phiên bản**: Tránh xung đột giữa các phiên bản thư viện
3. **Tái tạo được**: Dễ dàng chia sẻ và tái tạo môi trường
4. **An toàn**: Không ảnh hưởng đến Python system
5. **Dễ dọn dẹp**: Xóa toàn bộ môi trường khi không cần.


## 1.6. Cài đặt các gói thư viện cần thiết 


Đảm bảo môi trường `geotest` đã được kích hoạt trước khi cài đặt. Nếu chưa kích hoạt, ta có thể mở `Annaconda Prompt` và kích hoạt môi trường như bên dưới. Sau đó, cài đặt gói có thể theo một số phương pháp sau.
- **Phương pháp 1:** Cài đặt thư viện python bằng `pip` hoặc `conda` hoặc `mamba`

```bash
# Đảm bảo môi trường đúng
conda activate geotest

# Cài đặt thư viện sử dụng conda
conda install -c conda-forge ipykernel

# Cài đặt gói python bằng pip
pip install requests 

# Có thể dùng mamba để tăng tốc độ cài đặt thư viện
conda install -n base mamba -c conda-forge
# Các gói thư viện cơ bản
mamba install -c conda-forge numpy pandas matplotlib seaborn poetry openpyxl -y

# Các gói không gian địa lý chính
mamba install -c conda-forge geopandas rioxarray ee planetary-computer odc-stac -y
```

- **Phương pháp 2:** Cài đặt thư viện từ repo GITHUB hoặc từ requirements.txt file

```bash
# Cài đặt thư viện từ GITHUB
pip install git+https://github.com/tuyenhavan/pymapee.git 
# Cài đặt thư viện từ file requirements.txt 
pip install -r requirements.txt # Nếu có sẵn file này.
```
- **Phương pháp 3:** Cài đặt thư viện từ gói đang phát triển trên máy cá nhân (Không bắt buộc)
```bash
# Đầu tiên tạo ra 1 gói/thư viện tên là geocom
poetry new geocom # Câu lệnh này sẽ tạo ra package tên là geocom (bạn có thể đặt tên gì bạn muốn)
cd geocom # Di chuyển vào folder geocom vừa tạo ra có chứa file pyproject.toml
poetry install # Sẽ cài đặt gói geocom vào môi trường ảo của bạn.
```

#### Xác minh cài đặt

```bash
# Kiểm tra danh sách packages đã cài
conda list
```
#### Tổng quan thư viện:

##### Khoa học dữ liệu cốt lõi:
- **numpy**: Tính toán số học hiệu suất cao
- **pandas**: Thao tác và phân tích dữ liệu
- **matplotlib**: Trực quan hóa dữ liệu cơ bản
- **seaborn**: Trực quan hóa dữ liệu, tương tự như matplotlib.
- **scikit-learn**: Machine learning

##### Không gian địa lý:
- **geopandas**: Xử lý dữ liệu vector GIS
- **rioxarray**: Xử lý dữ liệu raster. 
- **planetary-computer**: Truy cập dữ liệu của Planetary Computer.
- **ee**: Try cập vào sử dụng dữ liệu của Google Earth Engine.
- **odc-stac**: Đọc và xử lý dữ liệu từ Planerary Computer hoặc từ STAC.


## 1.7. Cấu hình VS Code cho Python

Tích hợp VS Code với môi trường Python và Jupyter để có trải nghiệm phát triển tốt nhất.

### Chọn Python Interpreter

1. **Mở VS Code**
2. **Tạo hoặc mở file jupyter notebook** với `.ipynb` extension
2. **Mở Command Palette** (Ctrl+Shift+P)
3. **Gõ**: `Python: Select Interpreter`
4. **Chọn**: Interpreter từ môi trường `geotest` (Ví dụ môi trường là geotest, bạn nên chọn tên môi trường bạn đã tạo).

### Shortcuts hữu ích trong VS Code

#### Jupyter Notebook:
- `Shift + Enter`: Chạy cell và chuyển đến cell tiếp theo
- `Ctrl + Enter`: Chạy cell hiện tại
- `Alt + Enter`: Chạy cell và tạo cell mới bên dưới
- `A`: Thêm cell phía trên (command mode)
- `B`: Thêm cell phía dưới (command mode)
- `DD`: Xóa cell (command mode)

#### Python Development:
- `F5`: Start debugging
- `Ctrl + F5`: Run without debugging
- `F9`: Toggle breakpoint
- `Ctrl + Shift + P`: Command palette


## 1.8. Kiểm tra xác minh

Cuối cùng kiểm tra và nếu mọi thứ chạy thành công. Xin chúc mừng bạn đã thiết lập xong môi trường python cho phân tích dữ liệu địa không gian. Tuy nhiên, trong quá trình cài đặt, một số lỗi sau có thể xảy ra và cách khắc phục: 

#### Vấn đề 1: "conda: command not found"
**Giải pháp**: Anaconda không có trong PATH của bạn
- Windows: Tìm đến path của conda và thêm chúng vào trong Edit Environment Variables. Thường path sẽ nằm trong thư mục này `C:\Users\yourname\miniconda3\Scripts`.
- Mac/Linux: Thêm vào shell profile hoặc sử dụng Anaconda Navigator

#### Vấn đề 2: Cài đặt gói thư viện thất bại
**Giải pháp**: Thử các kênh thay thế
```bash
conda install -c conda-forge tên_gói
# hoặc
pip install tên_gói
```

#### Vấn đề 3: Jupyter không khởi động
**Giải pháp**: Kiểm tra kích hoạt môi trường
```bash
conda activate geotest
conda install jupyter
jupyter notebook
```

#### Vấn đề 4: Lỗi quyền truy cập (Windows)
**Giải pháp**: Chạy với quyền quản trị hoặc cài đặt cho người dùng
```bash
pip install --user tên_gói
```

Nếu bạn thử các bước trên và không còn lỗi nứa. Chúc mừng! Bạn đã thiết lập thành công môi trường Python cho phân tích không gian địa lý.

## 1.9. Bonus: Cài đặt QGIS

QGIS là phần mềm GIS mã nguồn mở mạnh mẽ, bổ sung cho Python trong phân tích không gian địa lý. Mình thường sử dụng QGIS cho việc kiểm tra và trực quan hóa dữ liệu địa không gian.

### Tải và cài đặt QGIS

#### Windows:
1. **Tải QGIS**:
   - Truy cập: https://www.qgis.org/en/site/forusers/download.html
   - Chọn "Long term release" (LTR) để ổn định

2. **Cài đặt**:
   - Chạy file `.msi` đã tải
   - Chọn "Complete installation"

#### macOS:
```bash
# Sử dụng Homebrew
brew install --cask qgis

# Hoặc tải từ trang chính thức
```

#### Linux (Ubuntu):
```bash
# Thêm repository
sudo apt update
sudo apt install qgis qgis-plugin-grass
```
