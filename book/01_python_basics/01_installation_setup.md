# Bài 1: Cài đặt và thiết lập môi trường Python

Trong bài học đầu tiên này, chúng ta sẽ cài đặt các công cụ cần thiết cho việc sử dụng python trong lĩnh vực địa không gian. Bài học này có kèm theo [video](https://www.youtube.com/watch?v=msZteBOpblc&t=29s) hướng dẫn cài đặt.

## 1.1. Mục tiêu học tập
Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Cài đặt và cấu hình VS Code làm môi trường làm việc
- Cài đặt Python thông qua Miniconda
- Cài đặt và thiết lập Git để quản lý mã nguồn
- Tạo và quản lý môi trường ảo bằng Miniconda
- Cài đặt các thư viện cần thiết vào môi trường ảo
- Sử dụng Jupyter Notebook trong VS Code

## 1.2. Tải và cài đặt VS Code

Visual Studio Code (VS Code) là một trình soạn thảo mã nguồn mở miễn phí và mạnh mẽ, hoàn hảo cho việc soạn thảo code Python.

### 1.2.1. Tải VS code

Để tải VS code, truy cập vào [vs code website](https://code.visualstudio.com/) và tiến hành tải về phiên bản phù hợp với hệ điều hành đang sử dụng. Trong quá trình tải, hãy lựa chọn đúng phiên bản cài đặt tương ứng:
- Windows: `.exe` file
- macOS: `.dmg` file
- Linux: `.deb` hoặc `.rpm` file

### 1.2.2. Cài đặt VS code trên Windows

Dưới đây là các bước cài đặt `vs code` trên Windows:

- ✅ Chạy file `.exe`, trong quá trình cài đặt
- ✅ Chọn "Add 'Open with Cod' action to Windows Explorer file context menu".
- ✅ Chọn "Add 'Open with Code' action to Windows Explorer directory context menu".
- ✅ Tất cả các thông số còn lại có thể chọn mặc định. 

### 1.2.3. Cài đặt VS code trên Lunix

Trên hệ điều hành Linux, Visual Studio Code có thể được cài đặt thông qua trình quản lý gói (package manager) hoặc bằng cách sử dụng tệp cài đặt đã tải về từ trang chính thức.

Người dùng có thể lựa chọn phương pháp phù hợp với bản phân phối đang sử dụng (Ubuntu, Debian, Fedora, v.v.) để đảm bảo quá trình cài đặt diễn ra thuận lợi. Hướng dẫn chi tiết cho từng hệ thống Linux có thể tham khảo tại trang tài liệu chính thức tại [đây](https://code.visualstudio.com/docs/setup/linux). 

## 1.3. Cài đặt tiện ích trong VS code

### 1.3.1. Cài đặt các tiện ích chính

VS Code cung cấp một hệ sinh thái tiện ích (extensions) phong phú, cho phép người dùng dễ dàng mở rộng chức năng theo nhu cầu công việc. Trong phần này, chúng ta sẽ tập trung lựa chọn và cài đặt các tiện ích thiết yếu, hỗ trợ hiệu quả cho quá trình phân tích dữ liệu không gian.

Để cài đặt tiện ích trong VS code, bạn có thể dùng Ctrl+Shift+X.

- **Python** (Microsoft): Extension chính để phát triển Python, cung cấp khả năng chạy và debug code Python, quản lý interpreter, syntax highlighting, và tích hợp terminal Python. Đây là extension bắt buộc cho mọi dự án Python.

- **Jupyter** (Microsoft): Tích hợp hoàn chỉnh Jupyter Notebook vào VS Code, cho phép tạo, chỉnh sửa và chạy các notebook (.ipynb) trực tiếp trong editor. Hỗ trợ nhiều kernel khác nhau và hiển thị output trực quan.

- **Pylance** (Microsoft): Language server tiên tiến cho Python, cung cấp IntelliSense thông minh, kiểm tra lỗi thời gian thực, type checking, auto-completion, và phân tích code static. Kế thừa từ Pyright và là công cụ phân tích code Python tốt nhất hiện tại.

Cài đặt tiện ích khuyên dùng

- **autoDocstring** (Nils Werner): Tự động tạo docstring cho hàm và class theo chuẩn Google, NumPy hoặc Sphinx. Giúp viết tài liệu code nhanh chóng và chuẩn mực.
- **Black Formatter** (Microsoft): Công cụ format code Python tự động theo chuẩn PEP 8. Giúp code nhất quán, dễ đọc và tuân thủ quy tắc coding style của Python.
- **vscode-icons** (VSCode Icons Team): Thêm biểu tượng đẹp cho các loại file khác nhau trong VS Code, giúp dễ dàng nhận diện và điều hướng trong project. 
- **GitHub Copilot Chat** (Microsoft): Trợ lý lập trình được tích hợp trực tiếp trong VS Code.

### 1.3.2. Cấu hình cơ bản VS code

bạn có thể điều chỉnh và thêm các cài đặt cơ bản cho VS code trong phần cài đặt (ctr+).

```json
   {
     "python.defaultInterpreterPath": "python",
     "python.formatting.provider": "black",
     "editor.formatOnSave": true,
     "files.autoSave": "afterDelay"
   }
```

## 1.4. Cài đặt Git

Git là hệ thống quản lý phiên bản phân tán, rất cần thiết cho việc phát triển phần mềm và quản lý dự án.

## 1.4.1. Tải và cài đặt Git cho Windows

Để cài đặt Git, bạn truy cập trang web chính thức tại [đây](https://git-scm.com/install/windows) và tải về phiên bản phù hợp với hệ điều hành Windows đang sử dụng (thường là phiên bản 64-bit mới nhất). Sau khi tải xong, tiến hành cài đặt theo các bước hướng dẫn để hoàn tất quá trình thiết lập Git trên máy tính.

Sau khi tải xong, các bước cài đặt như sau:

- ✅ Chạy file `.exe` đã tải và chọn
- ✅ "Use Visual Studio Code as Git's default editor"
- ✅ Các thông số khác chọn mặc định.

### 1.4.2. Cài đặt Git cho Macs và Lunix

**Cài đặt Git cho macOS**

```bash
brew install git
```

**Cài đặt Git cho Linux (Ubuntu/Debian)**
```bash
sudo apt update
sudo apt install git
```

## 1.4.3. Thiết lập cấu hình cho Git

Khi sử dụng Git lần đầu tiên, bạn cần thiết lập một số thông tin cấu hình cơ bản để đảm bảo các commit được ghi nhận đúng người dùng và môi trường làm việc được đồng bộ. Mở Terminal và thực hiện các lệnh sau: 

```bash
# Thiết lập tên và email (nên dùng thông tin GitHub của bạn)
git config --global user.name "yourname" 
git config --global user.email "youremail@gmail.com"

# Thiết lập trình soạn thảo mặc định (Visual Studio Code)
git config --global core.editor "code --wait"

# Thiết lập nhánh mặc định khi khởi tạo repository
git config --global init.defaultBranch main
```


## 1.5. Tải và Cài đặt Miniconda (Khuyến nghị)

Miniconda là phiên bản tối giản của Anaconda, cung cấp Python và conda package manager mà không có các gói thư viện phụ. Miniconda miễn phí cho người dùng.

### 1.5.1. Tải Miniconda

Để tải Miniconda, truy cập vào [miniconda website](https://repo.anaconda.com/miniconda/) và tiến hành tải về phiên bản phù hợp với hệ điều hành đang sử dụng. Trong quá trình tải, hãy lựa chọn đúng phiên bản cài đặt tương ứng:
- Windows: `.exe` file
- macOS:  `.pkg` hoặc `.sh`
- Linux: `.sh` file

### 1.5.2. Cài đặt Miniconda trên Windows

Dưới đây là các bước cài đặt `Miniconda` trên Windows:

- ✅ Chạy file `.exe`, trong quá trình cài đặt chọn
- ✅ "Just Me (recommended)" 
- ❌ Không chọn "Add Miniconda3 to PATH" (dùng Anaconda Prompt)
- ✅ Tất cả các thông số cài đặt khác chọn mặc định (khuyến nghị).

### 1.5.3. Cài đặt Miniconda trên macOS và Lunix

#### 1.5.3.1. Cài đặt trên macOS

```bash
# Nếu tải file .sh
bash Miniconda3-latest-MacOSX-x86_64.sh
```

#### 1.5.3.2. Cài đặt trên Lunix

```bash
# Tải và cài đặt
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

## 1.6. Kiểm tra cài đặt 

Sau khi hoàn tất quá trình cài đặt các phần mềm cần thiết, bạn nên kiểm tra lại hệ thống để đảm bảo mọi công cụ đã được cài đặt thành công và hoạt động bình thường. Thực hiện kiểm tra bằng cách chạy các lệnh sau trong Terminal:

```bash
git --version
python --version
conda --version
```
Nếu các lệnh trên trả về thông tin phiên bản tương ứng của từng phần mềm, điều đó có nghĩa là quá trình cài đặt đã được thực hiện thành công.

## 1.7. Thiết lập môi trường ảo

Môi trường ảo giúp cách ly các dự án và tránh xung đột giữa các phiên bản thư viện khác nhau. Chúng ta sẽ dùng miniconda để tạo và quản lý môi trường ảo và các gói python cài đặt.

### 1.7.1. Tạo môi trường ảo bằng Miniconda

Để tạo môi trường ảo, ta cần mở `Anaconda Promt` lên và thực hiện như sau:

```bash
# Tạo môi trường cho khóa học này. Ví dụ tạo môi trường tên `geotest`
conda create -n geotest python=3.12 -y # Lần đầu tạo môi trường ảo sẽ hiển thị các điều khoản. Làm theo hướng dẫn hiển thị nếu có.
```
Trong câu lệnh trên, `geotest` là tên môi trường. Mình có thể đặt tên khác nếu mong muốn.

### 1.7.2. Kích hoạt môi trường ảo

Sau khi tạo môi trường ảo, ta có thể kích hoạt tên môi trường đó để sử dụng như sau:

```bash
# Kích hoạt môi trường
conda activate geotest
```

Trong trường hợp muốn xóa, bạn có thể làm như sau:

```bash
conda env remove -n geotest --all  # Bạn có thể thay ptest bằng tên môi trường bạn đã tạo.
```

## 1.8. Cài đặt các gói thư viện cần thiết 

Đảm bảo môi trường `geotest` đã được kích hoạt trước khi cài đặt thư viện. Nếu chưa kích hoạt, ta có thể mở `Annaconda Prompt` và kích hoạt môi trường như bên dưới. Sau đó, cài đặt thư viện theo một trong những phương pháp bên dưới.

### 1.8.1. Cải đặt thư viện với `pip`

```bash
# Đảm bảo môi trường đúng
conda activate geotest

# Cài đặt gói python bằng pip
pip install requests ipykernal poetry rioxarray geopandas
```

### 1.8.2. Cài đặt thư viện với `conda`

Cú pháp cài đặt thư viện bằng conda như sau: `conda install -c conda-forge ten_goi`

```bash
conda install -c conda-forge geopandas rioxarray ee planetary-computer odc-stac -y
```

### 1.8.3. Cài đặt thư viện từ GitHub Repo

Ví dụ như cài đặt gói từ pymapee repo như sau:

```bash
pip install git+https://github.com/tuyenhavan/pymapee.git 
```

### 18.3.4. Cài đặt thư viện từ file requirements

Trong nhiều dự án, danh sách các thư viện cần thiết thường được tổng hợp sẵn trong một tệp requirements.txt. Thay vì cài đặt thủ công từng thư viện, bạn có thể cài đặt toàn bộ dependencies của dự án một cách nhanh chóng và đồng bộ thông qua tệp này.

```bash
pip install -r requirements.txt # Nếu có sẵn file này.
```

## 1.9. Các lỗi thường gặp và cách khắc phục

Cuối cùng kiểm tra xem nếu mọi thứ đã được cài đặt thành công. Sau đây là một vài lỗi phổ biến và cách khắc phục: 

### 1.9.1. Lỗi liên quan đến conda

Nếu như lỗi `conda: not found`, thì có thể Anaconda không có trong PATH của bạn.

**Giải pháp**: 
- Windows: Tìm đến path của conda và thêm chúng vào trong Edit Environment Variables. Thường path sẽ nằm trong thư mục này `C:\Users\yourname\miniconda3\Scripts`.
- Mac/Linux: Thêm vào shell profile hoặc sử dụng Anaconda Navigator

### 1.9.2. Lỗi liên quan đến cài đặt thư viện

Nếu bạn gặp lỗi khi cài đặt thư viện, bạn có thể thử cài đăntj bằng `pip` hoặc `conda`.

```bash
conda install -c conda-forge tên_gói
# hoặc
pip install tên_gói
```

### 1.9.3. Không có Jupyter Notebook

Trong trường hợp này, bạn cần phải cài đặt `ipykernal` và `jupyter extension` trong VS code.

```bash
conda activate geotest
conda install ipykernel jupyter
```
