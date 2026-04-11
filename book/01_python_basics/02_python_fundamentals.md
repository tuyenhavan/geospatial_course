# Bài 2: Cơ bản về Python

Trong bài học này, chúng ta sẽ đề cập đến các khối xây dựng cơ bản của lập trình Python.

## 2.1. Mục tiêu học tập
Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Hiểu về biến Python và quy ước đặt tên
- Làm việc với các kiểu dữ liệu khác nhau (số, chuỗi, boolean)
- Thực hiện các phép toán cơ bản về toán học và chuỗi
- Sử dụng Python như một máy tính
- Hiểu về chú thích và tài liệu

## 2.2. Code Python đầu tiên của bạn

Hãy bắt đầu với chương trình "Hello, World!" truyền thống:


```python
# Chương trình Python đầu tiên của bạn!
print("Xin chào, thế giới!")
print("Chào mừng đến với Python cho phân tích không gian địa lý!")
```

**Điều gì vừa xảy ra?**
- `print()` là một hàm hiển thị văn bản trên màn hình
- Văn bản trong dấu ngoặc kép được gọi là "chuỗi" (string)
- Các dòng bắt đầu bằng `#` là chú thích (được Python bỏ qua)

## 2.3. Biến và phép gán

Biến giống như các thùng chứa lưu trữ giá trị dữ liệu. Trong Python, bạn tạo biến bằng cách gán giá trị cho tên. Tên biến thông thường được viết bằng tiếng Anh.

### 2.3.1. Tạo biến và hiển thị biến


```python
# Tạo biến
city_name = "Ha Noi" # Tạo biến tên thành phố và gán giá trị
population = 8587000 # Tạo biến dân số và gán giá trị
latitude = 21.0285 # Tạo biến vĩ độ và gán giá trị
longitude = 105.8542 # Tạo biến kinh độ và gán giá trị
is_capital = True # Tạo biến là thủ đô và gán giá trị

```

### 2.3.2. Quy tắc đặt tên biến

✅ **Tên biến tốt:**
- `city_name`, `population`, `latitude`, `longitude`
- Bắt đầu bằng chữ cái hoặc dấu gạch dưới
- Sử dụng chữ thường với dấu gạch dưới (snake_case)
- Mô tả rõ ràng và có ý nghĩa
- Không sử dụng các từ khóa trong python như `while, for, class`. Danh sách đầy đủ ở [đây](https://realpython.com/lessons/reserved-keywords/)

❌ **Tên biến tệ:**
- `2city` (bắt đầu bằng số)
- `city-name` (chứa dấu gạch ngang)
- `class` (từ khóa của Python)
- `x`, `data` (không mô tả rõ)

## 2.4. Kiểu dữ liệu

Python có nhiều kiểu dữ liệu tích hợp sẵn. Hãy khám phá những kiểu phổ biến nhất.

### 2.4.1. Các kiểu dữ liệu Python sẵn có


```python
# Số nguyên (whole numbers)
number_of_cities = 50
year = 2024

# Số thực (decimal numbers)  
area_km2 = 783.8
elevation_m = 156.7

# Chuỗi (text)
region = 'Đông Nam Á'  # Dấu nháy đơn hoặc kép đều được
quote = "Đây là một chuỗi với dấu ngoặc kép bên trong: 'Hello!'"

# Boolean (True/False)
has_coastline = True
is_landlocked = False
```

### 2.4.1. Chuyển đổi kiểu dữ liệu
Đôi khi bạn cần chuyển đổi giữa các kiểu dữ liệu khác nhau:


```python
# Chuyển đổi giữa các kiểu dữ liệu chuỗi số
population_str = "8419000"
population_int = int(population_str)  # Chuỗi sang số nguyên
population_float = float(population_str)  # Chuỗi sang số thực

temperature = 23.7 # Số thực sang số nguyên và chuỗi
temperature_str = str(temperature)  # Số thực sang chuỗi
temperature_int = int(temperature)  # Số thực sang số nguyên (làm tròn xuống)
```

## 2.5. Phép toán

Python có thể được sử dụng như một máy tính mạnh mẽ. Hãy khám phá các phép toán với ví dụ liên quan đến phân tích không gian địa lý:

### 2.5.1. Phép toán cơ bản


```python
# Các phép toán cơ bản
length = 100  # mét
width = 100    # mét

# Phép cộng
total = length + width

# Phép nhân
area = length * width

# Phép chia
area_hectares = area / 10000  # Chuyển sang hecta

# Tính thể tích
volume = length * width * 3  # Giả sử chiều cao 3m

# Phép chia lấy phần nguyên
plots = area // 100  # Luôn trả về số nguyên làm tròn xuống 

# Phép chia lấy dư
remaining_area = area % 100

# Bình phương, ví dụ là tính diện tích hình vuông
side_length = 20
square_area = side_length ** 2
```


```python
# Thứ tự các phép toán (PEMDAS)
# Ngoặc đơn, Lũy thừa, Nhân/Chia, Cộng/Trừ

# Tính khoảng cách sử dụng độ chênh lệch tọa độ (đơn giản hóa)
lat1, lon1 = 21.0285, 105.8542  # Hà Nội
lat2, lon2 = 10.76231, 106.66297  # TP.HCM

# Tính toán khoảng cách đơn giản (không chính xác, chỉ để minh họa)
lat_diff = lat2 - lat1
lon_diff = lon2 - lon1
distance_degrees = (lat_diff**2 + lon_diff**2)**0.5

# Chuyển đổi sang kilometer (ước tính)
distance_km = distance_degrees * 111  # Chuyển đổi ước lượng
```

### 2.5.2. Hàm toán học tích hợp sẵn


```python
# Hàm toán học tích hợp sẵn
elevations = [156.7, -45.2, 1234.5, 2890.1, 0.0]  # Đây là kiểu dữ liệu list (danh sách)

vmax = max(elevations) # Tính giá trị lớn nhất từ danh sách
vmin = min(elevations) # Tính giá trị nhỏ nhất từ danh sách
total = sum(elevations) # Tính tổng giá trị từ danh sách
counts = len(elevations) # Tính số lượng phần tử trong danh sách
average = total / counts # Tính giá trị trung bình

# Giá trị tuyệt đối
temperature = -3.5
abs_temperature = abs(temperature) # Giá trị tuyệt đối

# Làm tròn
precise_area = 1234.56789
rounded_area = round(precise_area, 2) # Làm tròn đến 2 chữ số thập phân
```

## 2.6. Thao tác chuỗi

Chuỗi là dãy các ký tự. Chúng rất quan trọng để xử lý dữ liệu văn bản như tên địa điểm, địa chỉ và mô tả.

### 2.6.1. Tạo chuỗi đơn giản


```python
# Tạo chuỗi
city = "Hà Nội"
country = "Việt Nam"

# Nối chuỗi (joining)
full_location = city + ", " + country
```

### 2.6.2. Định dạng chuỗi sử dụng `f-string`


```python
# Định dạng chuỗi (f-strings) - Cách hiện đại của Python
population = 8587000
area_sq_km = 3358.59
country = "Việt Nam"
city = "Hà Nội"
# Tạo mô tả về thành phố sử dụng f-string
description = f"{city} là thành phố thuộc {country} với dân số {population:,} người và diện tích {area_sq_km} km²."
```

### 2.6.3. Phương thức chuỗi


```python
# Phương thức chuỗi
location = "  Phú Thọ  "
counter = len(location)  # Đếm số ký tự trong chuỗi
print(f"Địa điểm: '{location}'")
print(f"Độ dài: {counter} ký tự")
print(f"Chữ hoa: {location.upper()}") # Chuyển đổi chuỗi thành chữ hoa
print(f"Chữ thường: {location.lower()}") # Chuyển đổi chuỗi thành chữ thường
print(f"Chữ cái đầu hoa: {location.title()}") # Chuyển đổi chữ cái đầu của mỗi từ thành chữ hoa
print(f"Loại bỏ khoảng trắng: '{location.strip()}'") # Loại bỏ khoảng trắng ở đầu và cuối chuỗi
print(f"Thay thế: {location.replace('Phú Thọ', 'Vĩnh Phúc')}") # Thay thế chuỗi con bằng chuỗi khác
```


```python
# Tách và nối chuỗi
coordinates = "21.0285,105.8542"
lat_str, lon_str = coordinates.split(",") # Tách chuỗi thành hai phần dựa trên dấu phẩy
latitude = float(lat_str) # Chuyển đổi chuỗi thành số thực
longitude = float(lon_str) # Chuyển đổi chuỗi thành số thực

print(f"Chuỗi gốc: {coordinates}")
print(f"Vĩ độ: {latitude}")
print(f"Kinh độ: {longitude}")

# Nối chuỗi với ký tự phân tách
location_parts = ["Hà Nội", "Việt Nam", "Châu Á"]
full_location = ", ".join(location_parts) # Nối các phần tử trong danh sách thành một chuỗi với dấu phẩy và khoảng trắng làm phân tách
print(f"Địa điểm đầy đủ: {full_location}")

# Chuỗi nhiều dòng
description = """
Đây là chuỗi nhiều dòng.
Nó có thể trải rộng trên nhiều dòng.
Hữu ích cho các mô tả dài hoặc docstring.
"""
```

## Tóm tắt

Bạn đã hoàn thành bài học cơ bản về Python. Đây là những gì bạn đã học:

### Các khái niệm chính đã đề cập:
- ✅ Biến và quy ước đặt tên
- ✅ Kiểu dữ liệu: số nguyên, số thực, chuỗi, boolean
- ✅ Chuyển đổi kiểu dữ liệu
- ✅ Phép toán và module math
- ✅ Thao tác và định dạng chuỗi

### Kỹ năng bạn có thể áp dụng:
- Tạo và thao tác biến cho dữ liệu không gian địa lý
- Thực hiện tính toán với tọa độ và đo lường
- Định dạng và làm sạch dữ liệu địa điểm
- Chuyển đổi giữa các đơn vị khác nhau
- Viết code rõ ràng, có tài liệu
