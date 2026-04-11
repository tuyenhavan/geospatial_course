# Bài 4: Cấu trúc điều khiển - Đưa ra quyết định và lặp lại hành động

Trong bài học này, chúng ta sẽ học cách làm cho chương trình thông minh bằng cách thêm khả năng đưa ra quyết định và lặp lại.

## 4.1. Mục tiêu học tập
Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Sử dụng câu lệnh điều kiện để đưa ra quyết định trong code
- Triển khai vòng lặp để lặp lại hành động một cách hiệu quả
- Tạo và sử dụng hàm để tổ chức code
- Xử lý lỗi cơ bản trong chương trình
- Áp dụng những khái niệm này vào xử lý dữ liệu không gian địa lý

## 4.2. Câu lệnh điều kiện

Câu lệnh điều kiện cho phép chương trình của bạn đưa ra quyết định dựa trên các điều kiện khác nhau. Trong Python, câu lệnh điều kiện gồm `if, elif, else`. 

### 4.2.1. Câu lệnh điều kiển với `if` và `if else`


```python
# Câu lệnh if cơ bản
temperature = 25  # Celsius

if temperature > 30: # Nếu nhiệt độ lớn hơn 30 độ C
    weather_status = "hot" # thì gán thời tiết nóng
    print(f"Thời tiết {weather_status}") # Nếu nhiệt độ không lớn hơn 30 độ C, không có gì được in ra

# Câu lệnh if-else
elevation = 2500  # mét trên mực nước biển

if elevation > 2000: # Nếu độ cao lớn hơn 2000 mét
    air_pressure = "low" # thì áp suất không khí thấp
else:
    air_pressure = "normal" # ngược lại, áp suất không khí bình thường
print(f"Độ cao: {elevation}m, Áp suất không khí: {air_pressure}")
```


```python
# Có thể sử dụng if -else để kiểm tra điều kiện
temperature = 25
weather_status = 'hot' if temperature > 30 else 'normal'
# hoặc như này
a = 10 
result = a*2 if a > 0 else 0
print(result)
```

### 4.2.2. Câu lệnh điều khiển `if elif else`


```python
# if-elif-else cho nhiều điều kiện. Có thể có một hoặc nhiều khối elif 
temperature = 25
# Giải thích
""" 
Kiểm tra xem nhiệt độ có trên 30 độ không, nếu có gán thời tiết nóng. 
Nếu không, kiểm tra xem nhiệt độ có trên 20 độ không, nếu có gán thời tiết ấm áp.
Nếu không, kiểm tra xem nhiệt độ có trên 10 độ không, nếu có gán thời tiết mát mẻ.
Nếu không, gán thời tiết lạnh. 
"""
if temperature > 30:
    weather_status = 'hot'
elif temperature > 20:
    weather_status = 'warm'
elif temperature > 10:
    weather_status = 'cool'
else:
    weather_status = 'cold'
print(f"Nhiệt độ: {temperature}°C, Thời tiết: {weather_status}")
```

### 4.2.3. Câu điều kiện lồng (nested conditions)


```python
# Điều kiện lồng nhau
# Giải thích 
""" 
Kiểm tra xem độ cao có trên 2000 mét không, nếu có gán áp suất không khí thấp.
Nếu độ cao cũng trên 3000 mét, gán áp suất không khí rất thấp.
Nếu độ cao không trên 2000 mét, gán áp suất không khí bình thường.
"""
elevation = 2500
if elevation > 2000:
    air_pressure = "low"
    if elevation > 3000:
        air_pressure = "very low"
else:
    air_pressure = "normal"
print(f"Độ cao: {elevation}m, Áp suất không khí: {air_pressure}")
```

## 4.3. Toán tử so sánh và logic

Các toán tử này giúp chúng ta tạo điều kiện cho câu lệnh if.

### 4.3.1. Toán tử so sánh


```python
# So sánh chuỗi
language = "Python"
if language == "Python":
    print("Bạn đang sử dụng Python!")
else:
    print("Bạn đang sử dụng ngôn ngữ khác.")

# So sánh số
temperature = 25
threshold = 30
if temperature > threshold:
    print("Nhiệt độ cao hơn ngưỡng.")
else:
    print("Nhiệt độ không cao hơn ngưỡng.")
```

### 4.3.2. Toán tử logic


```python
# Toán tử logic: and, or, not
temperature = 22
humidity = 65
precipitation = 0

# Toán tử AND
comfortable_weather = temperature >= 20 and precipitation <= 25 and humidity < 70
print(f"Điều kiện thời tiết dễ chịu: {comfortable_weather}")
```


```python
# Toán tử OR
extreme_weather = temperature > 35 or temperature < 0 or humidity > 90
print(f"Điều kiện thời tiết cực đoan: {extreme_weather}")
```


```python
# Toán tử NOT
no_rain = not (precipitation > 0)
print(f"Không mưa: {no_rain}")

# Điều kiện phức hợp
good_hiking_weather = (temperature >= 15 and temperature <= 28 and 
                      precipitation == 0 and humidity < 80)
print(f"Thời tiết tốt để leo núi: {good_hiking_weather}")

if good_hiking_weather:
    print("Ngày hoàn hảo để leo núi!")
else:
    print("Có lẽ nên ở trong nhà hôm nay.")
```

### 4.3.3. Toán tử thành viên (membership)


```python
# Toán tử thành viên: in, not in
major_cities = ["Hà Nội", "TP.HCM", "Đà Nẵng", "Cần Thơ", "Hải Phòng"]
user_city = "Đà Nẵng"

if user_city in major_cities:
    print(f"{user_city} là thành phố lớn của Việt Nam")
else:
    print(f"{user_city} không có trong danh sách thành phố lớn của chúng ta")
```

## 4.4. Vòng lặp - For và While

Vòng lặp cho phép chúng ta lặp lại hành động một cách hiệu quả, rất cần thiết để xử lý nhiều điểm dữ liệu.

### 4.4.1. Vòng lặp `for`

Vòng lặp for sử dụng khi bạn muốn lặp qua từng phần tử trong một tập hợp (list, array, DataFrame, range, v.v.) và biết trước số lần lặp hoặc có thể duyệt được.

- **Vòng lặp `for` với `range()`**


```python
# Vòng lặp for với range(start, end, step)
print("Đọc nhiệt độ trong 5 ngày:")
for day in range(1, 6):  # 1 đến 5, step=1 mặc định
    temperature = 20 + day * 2  # Nhiệt độ tăng dần (mô phỏng)
    print(f"Ngày {day}: {temperature}°C")

print("\nĐếm ngược từ 10:")
for i in range(10, 0, -1):  # Bắt đầu từ 10, dừng trước 0, bước -1
    print(i)
print("Phóng!")
```

- **Vòng lặp `for` với `list`**


```python
# Vòng lặp for với danh sách - dữ liệu tương đối 2023
cities = ["Hà Nội", "TP.HCM", "Đà Nẵng", "Cần Thơ"]
for city in cities:
    print(f"- {city}")
```


```python
populations = [8435700, 9077158, 1230000, 1235171]  # Dân số cập nhật 2023
areas = [3358.59, 2061.45, 1285.53, 1408.93]  # Diện tích (km²)
# Vòng lặp và lấy dữ liệu từ hai danh sách cùng lúc sử dụng chỉ số (index)
for i in range(len(cities)):
    print(f"{cities[i]}: {populations[i]:,} người")
```


```python
# Cách tốt hơn sử dụng zip với dữ liệu thực. zip tạo các bộ từ các danh sách tương ứng
for city, population, area in zip(cities, populations, areas):
    density = population / area
    print(f"{city}: {population:,} người")
    print(f"  Diện tích: {area:,.2f} km²")
    print(f"  Mật độ thực tế: {density:.0f} người/km²")
```


```python
# su dung enumerate de co ca chi so va gia tri
for index, city in enumerate(cities):
    print(f"{index + 1}. {city}")
```

- **Vòng lặp `for` với `dictionary`**


```python
# Vòng lặp for cho dictionary 
weather_data = {
    "Hà Nội": {"temperature": 30, "humidity": 70},
    "TP.HCM": {"temperature": 32, "humidity": 75},
    "Đà Nẵng": {"temperature": 28, "humidity": 65},
}
# Ta có thể lặp qua các mục (key-value pairs) trong dictionary sử dụng .items()
for city, data in weather_data.items():
    temp = data["temperature"]
    hum = data["humidity"]
    print(f"{city}: Nhiệt độ {temp}°C, Độ ẩm {hum}%")
```

### 4.4.2. Vòng lặp `while`

Vòng lặp `while` trong Python được dùng khi bạn muốn lặp lại một đoạn mã miễn là điều kiện còn đúng. Vòng lặp này rất hữu ích cho các tình huống không biết trước số lần lặp và dừng lại chỉ khi điều kiện trở thành False.

- **Vòng lặp while**


```python
# Vòng lặp while
print("Tìm kích thước mẫu tối ưu:")
sample_size = 10
target_accuracy = 95

while sample_size < 1000:  # Ngăn vòng lặp vô hạn, nếu không đạt được độ chính xác mục tiêu
    # Tính toán độ chính xác đơn giản
    accuracy = 50 + (sample_size / 20) 
    
    print(f"Kích thước mẫu: {sample_size}, Độ chính xác: {accuracy:.1f}%")
    
    if accuracy >= target_accuracy: # Nếu đạt độ chính xác mục tiêu, dừng vòng lặp
        print(f"Đạt độ chính xác mục tiêu với {sample_size} mẫu!")
        break # Dừng vòng lặp nếu đạt mục tiêu
    sample_size += 10 # Tăng kích thước mẫu lên 10 cho lần thử tiếp theo
```

- **Điều khiển vòng lặp với `break` và `continue`**

`break` dùng để thoát khỏi vòng lặp ngay lập tức, trong khi `continue` để bỏ qua phần còn lại của vòng lặp hiện tại và chuyển sang lần lặp tiếp theo.


```python
# Demo sử dụng break và continue trong vòng lặp while
counter = 0
while counter < 10:
    counter += 1
    if counter == 3:
        print("Bỏ qua số 3 (continue)")
        continue  # Bỏ qua phần còn lại, chuyển sang lần lặp tiếp theo
    if counter == 8:
        print("Dừng vòng lặp tại số 8 (break)")
        break  # Thoát khỏi vòng lặp ngay lập tức
    print(f"Giá trị hiện tại: {counter}")
```

## 4.5. Xử lý lỗi

Xử lý lỗi giúp cho debug chương trình dễ dàng hơn khi gặp phải tình huống không mong muốn.

### 4.5.1. Xử lý lỗi với `try-except`


```python
# try-except cơ bản
a = 10
b = 0
try:
    result = a / b 
    print(f"Kết quả: {result}")
except ZeroDivisionError: # Bắt lỗi chia cho 0
    print("Lỗi: Không thể chia cho 0!")
```


```python
# Bắt lỗi với thông báo chi tiết
long = 200  # Kinh độ không hợp lệ
lat = 45   # Vĩ độ hợp lệ
try:
    if not (-90 <= lat <= 90):
        raise ValueError(f"Vĩ độ không hợp lệ: {lat}")
    if not (-180 <= long <= 180):
        raise ValueError(f"Kinh độ không hợp lệ: {long}")
    print(f"Tọa độ hợp lệ: ({lat}, {long})")
except ValueError as ve:
    print(f"Lỗi xác thực tọa độ: {ve}")
```


```python
# File handling with error management
file_path = r"G:\My Drive\python\geocourse\data\excels\data.txt"
try:
    with open(file_path, 'r') as file:
        data = file.read()
        print("Dữ liệu tệp đã được đọc thành công.")
except FileNotFoundError:
    print(f"Lỗi: Tệp không tìm thấy tại đường dẫn {file_path}")
except IOError:
    print(f"Lỗi: Không thể đọc tệp tại đường dẫn {file_path}")
```

### 4.5.2. Xử lý lỗi với `assert`

Sử dụng assert khi muốn bắt lỗi xảy ra. Cấu trức như sau
assert condition, message 

condition True will ignore but False will raise error


```python
# sử dụng assert để bắt lỗi 
human_age = -5  # Tuổi không hợp lệ
try:
    assert human_age >= 0, "Tuổi phải là số không âm"
    print(f"Tuổi hợp lệ: {human_age}")
except AssertionError as ae:
    print(f"Lỗi xác thực tuổi: {ae}")
```

## Tóm tắt

Bạn đã hoàn thành Bài 4 và học được những khái niệm lập trình mạnh mẽ tạo nên nền tảng cho việc xử lý dữ liệu.

### Các khái niệm chính đã nắm vững:
- ✅ **Câu lệnh điều kiện**: Đưa ra quyết định với if/elif/else
- ✅ **Toán tử so sánh**: Kiểm tra mối quan hệ giữa các giá trị
- ✅ **Toán tử logic**: Kết hợp điều kiện với and/or/not
- ✅ **Vòng lặp For**: Lặp qua các chuỗi và phạm vi
- ✅ **Vòng lặp While**: Lặp lại hành động dựa trên điều kiện
- ✅ **Hàm**: Tổ chức code thành các khối có thể tái sử dụng
- ✅ **Xử lý lỗi**: Quản lý các tình huống bất ngờ một cách khéo léo
- ✅ **Ứng dụng thực tế**: Phân tích thời tiết, tính toán khoảng cách, chất lượng dữ liệu

### Kỹ năng bạn có thể áp dụng:
- Xử lý và xác thực bộ dữ liệu không gian địa lý
- Tạo logic điều kiện cho phân tích dữ liệu
- Xây dựng hàm có thể tái sử dụng cho các tính toán phổ biến
- Xử lý lỗi và các trường hợp ngoại lệ trong xử lý dữ liệu
- Triển khai các thuật toán phức tạp như công thức Haversine
