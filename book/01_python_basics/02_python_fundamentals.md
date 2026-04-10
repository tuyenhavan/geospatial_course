# Bài 2: Cơ bản về Python

Chào mừng đến với bài học Python thực hành đầu tiên! Trong bài học này, chúng ta sẽ đề cập đến các khối xây dựng cơ bản của lập trình Python.

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


```python
# Ví dụ về đặt tên biến tốt cho dữ liệu không gian địa lý
building_height = 100.5  # mét 
road_length_km = 15.7  # km
temperature_celsius = 23.5  # °C
wind_speed_mps = 8.2  # m/s
precipitation_mm = 45.0  # mm
```

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
country = "Việt Nam"
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

temperature = 23.7
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
perimeter = 2 * (length + width)

# Phép nhân
area = length * width

# Phép chia
area_hectares = area / 10000  # Chuyển sang hecta

# Tính thể tích
volume = length * width * 3  # Giả sử chiều cao 3m

# Phép chia lấy phần nguyên
plots = area // 100  # Bao nhiêu lô 100 m² có thể chia?

# Phép chia lấy dư
remaining_area = area % 100
# Bình phương, ví dụ là tính diện tích hình vuông
side_length = 20
square_area = side_length ** 2
print(plots)
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
distance_km
```

### 2.5.2. Hàm toán học tích hợp sẵn


```python
# Hàm toán học tích hợp sẵn
elevations = [156.7, -45.2, 1234.5, 2890.1, 0.0]  # Đây là kiểu dữ liệu list (danh sách)

vmax = max(elevations) # Giá trị lớn nhất
print(vmax)
vmin = min(elevations) # Giá trị nhỏ nhất
total = sum(elevations) # Tổng giá trị
counts = len(elevations) # Số lượng phần tử
print(counts)
average = total / counts # Tính trung bình
# Giá trị tuyệt đối
temperature_anomaly = -3.5
abs_temperature_anomaly = abs(temperature_anomaly) # Giá trị tuyệt đối

# Làm tròn
precise_area = 1234.56789
rounded_area = round(precise_area, 2) # Làm tròn đến 2 chữ số thập phân
rounded_area
```

## 2.6. Thao tác chuỗi

Chuỗi là dãy các ký tự. Chúng rất quan trọng để xử lý dữ liệu văn bản như tên địa điểm, địa chỉ và mô tả.

### 2.6.1. Tạo chuỗi đơn giản


```python
# Tạo chuỗi
city = "Hà Nội"
province = "Hà Nội"
country = "Việt Nam"

# Nối chuỗi (joining)
full_location = city + ", " + province + ", " + country
full_location
```




    'Hà Nội, Hà Nội, Việt Nam'



### 2.6.2. Định dạng chuỗi sử dụng `f-string`


```python
# Định dạng chuỗi (f-strings) - Cách hiện đại của Python
population = 8587000
area_sq_km = 3358.59
# Tạo mô tả về thành phố sử dụng f-string
description = f"{city} là thành phố thuộc {province}, {country} với dân số {population:,} người và diện tích {area_sq_km} km²."

# Nhiều cách định dạng chuỗi
population_density = population / area_sq_km 
print(f"Mật độ dân số: {population_density:.1f} người/km²")
print("Mật độ dân số: {:.1f} người/km²".format(population_density))
print("Mật độ dân số: %.1f người/km²" % (population_density))
print(description)
```

### 2.6.3. Phương thức chuỗi


```python
# Phương thức chuỗi
location = "  Phú Thọ  "
counter = len(location)  # Đếm số ký tự trong chuỗi
print(f"Địa điểm: '{location}'")
print(f"Độ dài: {counter} ký tự")
print(f"Chữ hoa: {location.upper()}")
print(f"Chữ thường: {location.lower()}")
print(f"Chữ cái đầu hoa: {location.title()}")
print(f"Loại bỏ khoảng trắng: '{location.strip()}'")
print(f"Thay thế: {location.replace('Phú Thọ', 'Vĩnh Phúc')}")
```

    Địa điểm: '  Phú Thọ  '
    Độ dài: 11 ký tự
    Chữ hoa:   PHÚ THỌ  
    Chữ thường:   phú thọ  
    Chữ cái đầu hoa:   Phú Thọ  
    Loại bỏ khoảng trắng: 'Phú Thọ'
    Thay thế:   Vĩnh Phúc  
    


```python
# Tách và nối chuỗi
coordinates = "21.0285,105.8542"
lat_str, lon_str = coordinates.split(",")
latitude = float(lat_str)
longitude = float(lon_str)

print(f"Chuỗi gốc: {coordinates}")
print(f"Vĩ độ: {latitude}")
print(f"Kinh độ: {longitude}")

# Nối chuỗi với ký tự phân tách
location_parts = ["Hà Nội", "Việt Nam", "Châu Á"]
full_location = ", ".join(location_parts)

# Chuỗi nhiều dòng
description = """
Đây là chuỗi nhiều dòng.
Nó có thể trải rộng trên nhiều dòng.
Hữu ích cho các mô tả dài hoặc tài liệu.
"""
# print(description.strip())
full_location
```

    Chuỗi gốc: 21.0285,105.8542
    Vĩ độ: 21.0285
    Kinh độ: 105.8542
    




    'Hà Nội, Việt Nam, Châu Á'



## 2.7. Chú thích và Tài liệu

Chú thích rất quan trọng để làm cho code của bạn dễ đọc và dễ bảo trì:


```python
# Chú thích một dòng

# Tính mật độ dân số
population =  8685607  # Dân số Hà Nội
area_sq_km = 3345  # Diện tích tính bằng km vuông

"""
Chú thích nhiều dòng (thực chất là chuỗi không được gán cho biến)
Sử dụng cho các giải thích dài hơn.
Thường được dùng cho tài liệu hàm.
"""

density = population / area_sq_km  # Người trên km vuông
print(f"Mật độ dân số: {density:.1f} người/km²")
```

    Mật độ dân số: 2596.6 người/km²
    

> **Lưu ý:**

>✅ **Chú thích tốt:**
>- Giải thích TẠI SAO, không chỉ LÀM GÌ
>- Mô tả logic phức tạp
>- Ghi chú các giả định hoặc hạn chế
>- Bao gồm đơn vị đo lường

>❌ **Tránh:**
>- Chú thích hiển nhiên (`x = 5  # Gán x bằng 5`)
>- Chú thích lỗi thời
>- Chú thích lặp lại code

## Tóm tắt

Bạn đã hoàn thành bài học cơ bản về Python. Đây là những gì bạn đã học:

### Các khái niệm chính đã đề cập:
- ✅ Biến và quy ước đặt tên
- ✅ Kiểu dữ liệu: số nguyên, số thực, chuỗi, boolean
- ✅ Chuyển đổi kiểu dữ liệu
- ✅ Phép toán và module math
- ✅ Thao tác và định dạng chuỗi
- ✅ Thực hành tốt nhất cho chú thích và tài liệu

### Kỹ năng bạn có thể áp dụng:
- Tạo và thao tác biến cho dữ liệu không gian địa lý
- Thực hiện tính toán với tọa độ và đo lường
- Định dạng và làm sạch dữ liệu địa điểm
- Chuyển đổi giữa các đơn vị khác nhau
- Viết code rõ ràng, có tài liệu
