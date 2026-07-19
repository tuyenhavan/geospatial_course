# Bài 2: Cơ bản về Python

Trong bài học này, chúng ta sẽ tìm hiểu các khối xây dựng cơ bản trong lập trình Python. Bài học này có [video](https://www.youtube.com/watch?v=aFI6suhmjhQ&t=6s) kèm theo. 

>> **Lưu ý**: Bạn có thể chạy trực tiếp notebook này bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/1ymxXpoIDpsnhi45-g_9Rges2XAsZAlcB) mà không cần cài đặt Python.

## 2.1. Mục tiêu học tập
Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Hiểu về biến Python và quy ước đặt tên
- Làm việc với các kiểu dữ liệu khác nhau (số, chuỗi, boolean)
- Thực hiện các phép toán cơ bản về toán học và chuỗi
- Hiểu về chú thích và tài liệu

## 2.2. Code Python đầu tiên

Hãy bắt đầu với chương trình "Hello, World!" truyền thống:


```python
# Chương trình Python đầu tiên của bạn!
print("Xin chào, thế giới!")
print("Chào mừng đến với Python cho phân tích không gian địa lý!")
```

    Xin chào, thế giới!
    Chào mừng đến với Python cho phân tích không gian địa lý!
    

**Điều gì vừa xảy ra?**
- `print()` là một hàm hiển thị văn bản trên màn hình
- Văn bản trong dấu ngoặc kép được gọi là "chuỗi" (string)
- Các dòng bắt đầu bằng `#` là chú thích (được Python bỏ qua)

## 2.3. Biến và phép gán

Biến giống như các thùng chứa lưu trữ giá trị dữ liệu. Trong Python, bạn tạo biến bằng cách gán giá trị cho tên. Tên biến thông thường được viết bằng tiếng Anh.

### 2.3.1. Tạo biến

Trong ví dụ dưới đây, ta sẽ đặt một vài tên biến và gán giá trị cho chúng. Dân số lấy từ trang [thư viện pháp luật](https://thuvienphapluat.vn/chinh-sach-phap-luat-moi/vn/ho-tro-phap-luat/chinh-sach-moi/108590/xep-hang-dan-so-34-tinh-thanh-pho-moi-nhat-2026) năm 2025. 


```python
# Tạo biến
city_name = "Ha Noi" # city_name là tên biến và gán giá trị. Trong ví dụ này giá trị là Ha Noi
population = 8860000 # population là tên biến và gán giá trị.
latitude = 21.0285 # latitude là tên biến và gán giá trị. Trong ví dụ này giá trị là 21.0285
longitude = 105.8542 # longitude là tên biến và gán giá trị. Trong ví dụ này giá trị là 105.8542
is_capital = True # is_capital là tên biến và gán giá trị. Trong ví dụ này giá trị là True
print("Tên thành phố:", city_name)
print("Dân số:", population)
print("Vĩ độ:", latitude)
print("Kinh độ:", longitude)
print("Có phải là thủ đô không?", is_capital)
```

    Tên thành phố: Ha Noi
    Dân số: 8587000
    Vĩ độ: 21.0285
    Kinh độ: 105.8542
    Có phải là thủ đô không? True
    

### 2.3.2. Quy tắc đặt tên biến

Tên biến thường được viết chuẩn theo quy tắc snake_case, nghĩa là các từ được nối với nhau bằng dấu gạch dưới và tất cả đều viết thường. Ví dụ: city_name, population, latitude, longitude, is_capital.

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

Python có nhiều kiểu dữ liệu tích hợp sẵn. Trong phần này, chúng ta tìm hiểu một vài kiểu dữ liệu có sẵn trong Python.

### 2.4.1. Các kiểu dữ liệu Python sẵn có

- **Kiểu số nguyên**

Kiểu số nguyên là kiểu dữ liệu dùng để biểu diễn các số không có phần thập phân. Kiểu này thường được sử dụng để đếm, đánh số hoặc thực hiện các phép toán số học cơ bản.


```python
# Số nguyên (whole numbers)
number_of_cities = 50
year = 2024
print("Số lượng thành phố:", number_of_cities)
print("Năm:", year)
```

    Số lượng thành phố: 50
    Năm: 2024
    

- **Kiểu số thực**

Kiểu số thực là kiểu dữ liệu dùng để biểu diễn các số có phần thập phân. Kiểu này thường được sử dụng trong các phép tính cần độ chính xác cao như đo lường hoặc tính toán khoa học.


```python
# Số thực (decimal numbers)  
area_km2 = 783.8
elevation_m = 156.7
print("Diện tích (km²):", area_km2)
print("Độ cao (m):", elevation_m)
```

    Diện tích (km²): 783.8
    Độ cao (m): 156.7
    

- **Kiểu chuỗi**

Kiểu chuỗi là kiểu dữ liệu dùng để lưu trữ một dãy các ký tự, như chữ cái, số hoặc ký hiệu, được đặt trong dấu nháy (nháy đơn hoặc nháy kép). Chuỗi thường được dùng để biểu diễn văn bản trong chương trình.


```python
# Chuỗi (text)
region = 'Đông Nam Á'  # Dấu nháy đơn hoặc kép đều được
quote = "Đây là một chuỗi với dấu ngoặc kép bên trong: 'Hello!'"
print("Khu vực:", region)
print("Trích dẫn:", quote)
```

    Khu vực: Đông Nam Á
    Trích dẫn: Đây là một chuỗi với dấu ngoặc kép bên trong: 'Hello!'
    

- **Kiểu boolean**

Kiểu boolean là kiểu dữ liệu chỉ có hai giá trị: True (đúng) và False (sai). Kiểu này thường được sử dụng trong các phép so sánh và điều kiện để kiểm tra đúng/sai trong chương trình.

> **Chú ý**: Các giá trị như 0, None, hoặc chuỗi trống, danh sách trống đều là False và các giá trị như 1, 2, etc hoặc chuỗi không trống đều là True.


```python
# Boolean (True/False)
has_coastline = True
is_landlocked = False
print("Có đường bờ biển không?", has_coastline)
print("Có phải là quốc gia không giáp biển không?", is_landlocked)
```

    Có đường bờ biển không? True
    Có phải là quốc gia không giáp biển không? False
    

### 2.4.1. Chuyển đổi kiểu dữ liệu

Chuyển đổi kiểu dữ liệu là quá trình biến đổi một giá trị từ kiểu dữ liệu này sang một kiểu dữ liệu khác, nhằm phù hợp với mục đích xử lý hoặc tính toán trong chương trình.

- **Chuyển đổi từ số dạng chuỗi sang số nguyên**

Số nguyên dạng chuỗi có thể chuyển sang dạng số nguyên dùng hàm `int`.


```python
# Chuyển đổi giữa các kiểu dữ liệu chuỗi số
population_str = "8860000"  # Chuỗi số
population_int = int(population_str)  # Chuỗi sang số nguyên
print("Dân số (số nguyên):", population_int)
print(f'Nhiệt độ hiện tại là {int(23.7)}°C') # Chuyển đổi số thực sang số nguyên và in ra kết quả

```

    Dân số (số nguyên): 8860000
    Nhiệt độ hiện tại là 23°C
    

- **Chuyển đổi từ số dạng chuỗi sang số thực**

Số nguyên hoặc số thực dạng chuỗi có thể chuyển sang dạng số dùng hàm `float`.


```python
temperature = '23.7' # Số thực sang số nguyên và chuỗi
temperature_float = float(temperature)  # Chuỗi sang số thực
temperature_int = int(temperature_float)  # Số thực sang số nguyên
print("Nhiệt độ (số thực):", temperature_float)
```

    Nhiệt độ (số thực): 23.7
    

- **Chuyển đổi số sang dạng chuỗi**

Số có thể chuyển sang dạng chuỗi dùng hàm `str`.


```python
number = 42
number_str = str(number)  # Số nguyên sang chuỗi
print("Số nguyên dưới dạng chuỗi:", number_str)
```

    Số nguyên dưới dạng chuỗi: 42
    

- **Chuyển đổi boolean sang dạng số**

`boolean` chuyển sang dạng số nguyên dùng hàm `int`.


```python
is_true = True
bool_number = int(is_true)  # Boolean sang số nguyên (True=1, False=0)
print("Giá trị số nguyên của True:", bool_number)
```

    Giá trị số nguyên của True: 1
    

## 2.5. Phép toán

Python có thể được sử dụng như một máy tính. Hãy khám phá các phép toán với ví dụ liên quan đến phân tích không gian địa lý.

### 2.5.1. Phép toán cơ bản

Phép toán cơ bản bao gồm các phép tính toán cộng, trừ, nhân, chia.

- **Phép cộng**

Phép cộng hoặc phép trừ trong Python sử dụng toán tử `+` và `-` tương ứng.


```python
# Các phép toán cơ bản
length = 100  # mét
width = 100    # mét
# Phép cộng
total = length + width
print("Tổng chiều dài và chiều rộng:", total, "m")
```

    Tổng chiều dài và chiều rộng: 200 m
    

- **Phép nhân**

Phép nhân trong Python sử dụng toán tử `*`. 


```python
# Phép nhân
length = 100  # mét
width = 100    # mét
area = length * width
print("Diện tích hình chữ nhật:", area, "m²")
```

    Diện tích hình chữ nhật: 10000 m²
    

- **Phép chia**

Phép chia trong Python sử dụng toán tử `/`.


```python
# Phép chia
area = 10000  # mét vuông
area_hectares = area / 10000  # Chuyển sang hecta
print("Diện tích (hecta):", area_hectares, "ha")
```

    Diện tích (hecta): 1.0 ha
    

- **Phép chia lấy phần nguyên**

Chia lấy phần nguyên trong Python sử dụng toán tử `//`. Ví dụ, nếu bạn muốn biết có bao nhiêu lô đất có diện tích 100 km² có thể được tạo ra từ một khu vực có diện tích 783.8 km², bạn có thể sử dụng phép chia lấy phần nguyên như bên dưới.


```python
# Phép chia lấy phần nguyên
area = 783.8  # km²
plots = area // 100  # Luôn trả về số nguyên làm tròn xuống 
print("Số lô đất (100 km² mỗi lô):", plots)
```

    Số lô đất (100 km² mỗi lô): 7.0
    

- **Phép chia lấy phần dư**

Phép chia lấy phân dư trong Python sử dụng toán tử `%`. Ví dụ, nếu bạn muốn biết diện tích còn lại sau khi chia một khu vực thành các lô có diện tích 100 km², bạn có thể sử dụng phép chia lấy phần dư như bên dưới.


```python
# Phép chia lấy dư
area = 783.8  # km²
remaining_area = area % 100 
print("Diện tích còn lại sau khi chia thành các lô 100 km²:", remaining_area, "km²")
```

    Diện tích còn lại sau khi chia thành các lô 100 km²: 83.79999999999995 km²
    

- **Bình phương**

Phép bình phương trong Python sử dụng toán tử `**`. Ví dụ, nếu bạn muốn tính diện tích của một hình vuông có cạnh dài 20 mét, bạn có thể sử dụng phép bình phương như bên dưới.


```python
# Bình phương, ví dụ là tính diện tích hình vuông
side_length = 20
square_area = side_length ** 2
print("Diện tích hình vuông:", square_area, "m²")
```

    Diện tích hình vuông: 400 m²
    

- **Giá trị tuyệt đối và làm tròn**

Trong Python, giá trị tuyệt đối của một số có thể được tính bằng cách sử dụng hàm `abs()`. Hàm này sẽ trả về giá trị dương của số đó, bất kể nó ban đầu là dương hay âm. Để làm tròn trong Python, ta sử dụng hàm `round()`. Hàm này có thể được sử dụng để làm tròn một số đến một số chữ số thập phân nhất định. Các ví dụ bên dưới mô tả phép lấy giá trị tuyệt đối và làm tròn.


```python
result = 12.5 - 35.2 # Kết quả sẽ là -22.7
result = abs(result) # Lấy giá trị tuyệt đối của kết quả
print("Giá trị tuyệt đối của phép trừ:", result)
result = round(result, 2) # Làm tròn kết quả đến 2 chữ số thập phân
print("Kết quả của phép trừ sau khi lấy giá trị tuyệt đối và làm tròn:", result)
```

    Giá trị tuyệt đối của phép trừ: 22.700000000000003
    Kết quả của phép trừ sau khi lấy giá trị tuyệt đối và làm tròn: 22.7
    

- **Thứ tự phép toán**

Trong toán học và lập trình, các phép toán được thực hiện theo thứ tự sau: Ngoặc đơn, lũy thừa, nhân/chia, cộng/trừ.


```python
result = 3+2*(4**2-1) # Kết quả sẽ là 3 + 2*(16-1) = 3 + 2*15 = 3 + 30 = 33
print("Kết quả của biểu thức là:", result)
```

    Kết quả của biểu thức là: 33
    

### 2.5.2. Hàm toán học tích hợp sẵn

Trong Python, có nhiều hàm và công cụ toán học có sẵn giúp người dùng thực hiện các phép tính một cách nhanh chóng và chính xác. Trong phần này, chúng ta tạo ra một danh sách dữ liệu độ cao và tính toán các chỉ số thống kê cho danh sách đó như các ví dụ bên dưới.


```python
# Ví dụ cho một danh sách các giá trị độ cao và thực hiện các phép toán bên dưới.
elevations = [156.7, -45.2, 1234.5, 2890.1, 0.0]  # Đây là kiểu dữ liệu list (danh sách)
print(f"Danh sách độ cao:", elevations)
```

    Danh sách độ cao: [156.7, -45.2, 1234.5, 2890.1, 0.0]
    

- **Tìm giá trị lớn nhất trong danh sách `elevations`**

Trong Python, bạn có thể sử dụng hàm `max()` để tìm giá trị lớn nhất trong một danh sách. 


```python
vmax = max(elevations) # Tính giá trị lớn nhất từ danh sách
print("Độ cao lớn nhất:", vmax, "m")
```

    Độ cao lớn nhất: 2890.1 m
    

- **Tìm giá trị nhỏ nhất trong danh sách `elevations`**

Trong Python, bạn có thể sử dụng hàm `min()` để tìm giá trị nhỏ nhất trong một danh sách. 


```python
vmin = min(elevations) # Tính giá trị nhỏ nhất từ danh sách
print("Độ cao nhỏ nhất:", vmin, "m")
```

    Độ cao nhỏ nhất: -45.2 m
    

- **Tổng các giá trị trong danh sách `elevations`**

Trong Python, bạn có thể sử dụng hàm `sum()` để tìm tổng giá trị trong một danh sách. 


```python
total = sum(elevations) # Tính tổng giá trị từ danh sách
print("Tổng độ cao:", total, "m")
```

    Tổng độ cao: 4236.099999999999 m
    

- **Số lượng phần tử trong danh sách `elevations`**

Trong Python, bạn có thể sử dụng hàm `len()` để tính số lượng phần tử trong một danh sách. 


```python
counts = len(elevations) # Tính số lượng phần tử trong danh sách
print("Số lượng độ cao trong danh sách:", counts)
```

    Số lượng độ cao trong danh sách: 5
    

- **Giá trị trung bình `elevations`**

Để tính giá trị trung bình, ta lấy tổng chia cho số lượng phân tử trong danh sách như bên dưới.


```python
average = total / counts # Tính giá trị trung bình
print("Độ cao trung bình:", average, "m")
```

    Độ cao trung bình: 847.2199999999999 m
    

## 2.6. Thao tác chuỗi

Chuỗi là dãy các ký tự. Chúng được sử dụng để thể hiện các thông tin như tên địa điểm, địa chỉ và mô tả. Dưới đây chúng ta sẽ tìm hiểu về các thao tác cơ bản trên chuỗi.

### 2.6.1. Tạo chuỗi đơn giản

- **Tạo chuỗi**

Chuỗi được tạo ra bằng sử dụng dấu nháy đơn hoặc kép như ví dụ bên dưới.


```python
# Tạo chuỗi
city = "Hà Nội"
country = "Việt Nam"
print(f"{city} là thủ đô của {country}.") # Sử dụng f-string để chèn biến vào chuỗi
```

    Hà Nội là thủ đô của Việt Nam.
    

- **Nối hai hoặc nhiều chuỗi**

Có nhiều cách để nối chuỗi trong Python. Một cách phổ biến là sử dụng toán tử `+` để kết hợp các chuỗi lại với nhau.


```python
# Nối chuỗi (joining)
full_location = city + ", " + country
print("Vị trí đầy đủ:", full_location)
```

    Vị trí đầy đủ: Hà Nội, Việt Nam
    

### 2.6.2. Định dạng chuỗi sử dụng `f-string`

Định dạng chuỗi (`f-strings`) là một cách hiện đại và tiện lợi để chèn biến vào chuỗi. Bạn có thể sử dụng `f-string` bằng cách đặt chữ f trước dấu nháy mở của chuỗi và sau đó chèn các biến vào trong dấu ngoặc nhọn `{}`.


```python
# Định dạng chuỗi (f-strings) - Cách hiện đại của Python
population = 8860000
area_sq_km = 3359.59
country = "Việt Nam"
city = "Hà Nội"
# Tạo mô tả về thành phố sử dụng f-string
description = f"{city} là thành phố thuộc {country} với dân số {population:,} người và diện tích {area_sq_km} km²."
print(description)
```

    Hà Nội là thành phố thuộc Việt Nam với dân số 8,860,000 người và diện tích 3359.59 km².
    

### 2.6.3. Phương thức chuỗi

Có nhiều phương thức chuỗi hữu ích trong Python để thao tác với chuỗi. Ví dụ, bạn có thể sử dụng phương thức `upper()` để chuyển một chuỗi thành chữ hoa, `lower()` để chuyển thành chữ thường, và `title()` để chuyển thành dạng tiêu đề (mỗi từ viết hoa).

- **Chuyển chuỗi thành chữ hoa hoặc chữa thường**

Sau đây là các ví dụ về chuyển đổi chữa hoa và chữ thường trong chuỗi.


```python
city = "Hà Nội"
city_capital = city.upper() # Chuyển chuỗi thành chữ hoa
city_lower = city.lower() # Chuyển chuỗi thành chữ thường
city_title = city.title() # Chuyển chuỗi thành dạng tiêu đề (mỗi từ viết hoa)
print("Chuỗi chữ hoa:", city_capital)
print("Chuỗi chữ thường:", city_lower)
print("Chuỗi dạng tiêu đề:", city_title)
```

    Chuỗi chữ hoa: HÀ NỘI
    Chuỗi chữ thường: hà nội
    Chuỗi dạng tiêu đề: Hà Nội
    

- **Loại bỏ khoảng trắng ở đầu và cuối**

Chuỗi thường có thể chứa các thông tin khoảng trắng ở đầu câu hoặc cuối câu. Để loại bỏ khoảng trắng này, chúng ta dùng phương thức `strip()`.


```python
city = "  Hà Nội  "
city_stripped = city.strip()  # Loại bỏ khoảng trắng ở đầu và cuối chuỗi
print("Tên thành phố sau khi loại bỏ khoảng trắng:", city_stripped)
```

    Tên thành phố sau khi loại bỏ khoảng trắng: Hà Nội
    

- **Thay thế chuỗi bằng chuỗi khác**

Để thay thế một phần của chuỗi bằng một phần khác, bạn có thể sử dụng phương thức `replace()` như ví dụ bên dưới.


```python
city = "Hà Nội"
replaced_city = city.replace('Hà', 'Thủ đô Hà') # Thay thế 'Hà' bằng 'Thủ đô Hà' trong chuỗi
print("Chuỗi sau khi thay thế:", replaced_city)
```

    Chuỗi sau khi thay thế: Thủ đô Hà Nội
    

- **Tách chuỗi dùng hàm `split`**

Nếu ta muốn tách một chuỗi thành nhiều phần dựa trên một ký tự phân cách (ví dụ: dấu phẩy), ta có thể sử dụng phương thức split(). Giả sử trong ví dụ sau, ta muốn tách tọa độ thành kinh và vĩ độ tại vị trí dấu phẩy như bên dưới.


```python
# Tách và nối chuỗi
coordinates = "21.0285,105.8542"
lat_str, lon_str = coordinates.split(',') # Tách chuỗi thành hai phần dựa trên dấu phẩy
lat = float(lat_str) # Chuyển vĩ độ từ chuỗi sang số thực
lon = float(lon_str) # Chuyển kinh độ từ chuỗi sang số thực
print(f'Vĩ độ: {lat}, Kinh độ: {lon}')
```

    Vĩ độ: 21.0285, Kinh độ: 105.8542
    

- **Chuỗi nhiều dòng**

Chuỗi nhiều dòng là chuỗi chứa nhiều dòng văn bản, thường được viết trong cặp dấu nháy ba (''' hoặc """)


```python
# Chuỗi nhiều dòng
description = """
Đây là chuỗi nhiều dòng.
Nó có thể trải rộng trên nhiều dòng.
Hữu ích cho các mô tả dài hoặc docstring.
"""
print(description)
```

    
    Đây là chuỗi nhiều dòng.
    Nó có thể trải rộng trên nhiều dòng.
    Hữu ích cho các mô tả dài hoặc docstring.
    
    

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
