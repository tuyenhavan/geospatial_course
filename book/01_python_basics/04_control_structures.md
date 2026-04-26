# Bài 4: Cấu trúc điều khiển và vòng lặp

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

### 4.2.1. Câu lệnh điều kiển với `if` và `if-else`

- **Câu lệnh điều khiển với `if`**


```python
# Câu lệnh if cơ bản
temperature = 25  # Celsius
if temperature > 30: # Nếu nhiệt độ lớn hơn 30 độ C, không thì không thoải mãn điều kiện
    weather_status = "hot" # thì gán thời tiết nóng
    print(f"Thời tiết {weather_status}") # Nếu nhiệt độ không lớn hơn 30 độ C, không có gì được in ra
```

- **Câu lệnh điều khiển `if-else`**


```python
    # Câu lệnh if-else
elevation = 2500  # mét trên mực nước biển

if elevation > 2000: # Nếu độ cao lớn hơn 2000 mét
    air_pressure = "low" # thì áp suất không khí thấp
else: # Ngược lại, nếu độ cao không lớn hơn 2000 mét
    air_pressure = "normal" # thì áp suất không khí bình thường
print(f"Độ cao: {elevation}m, Áp suất không khí: {air_pressure}")
```

    Độ cao: 2500m, Áp suất không khí: low
    

- **`if-else` viết trên một dòng**


```python
# Có thể sử dụng if-else để kiểm tra điều kiện
temperature = 25
weather_status = 'hot' if temperature > 30 else 'normal'
print(f"Thời tiết: {weather_status}")
# hoặc như này
a = 10 
result = a*2 if a > 0 else 0
print("Kết quả:", result)
```

    Thời tiết: normal
    Kết quả: 20
    

### 4.2.2. Câu lệnh điều khiển `if-elif-else`

Câu lệnh `if-elif-else` được sử dụng khi bạn cần kiểm tra nhiều điều kiện khác nhau và thực hiện các hành động tương ứng.
- `if`: kiểm tra điều kiện đầu tiên
- `elif`: kiểm tra các điều kiện tiếp theo nếu điều kiện trước sai
- `else`: chạy khi tất cả điều kiện trên đều sai


```python
# Khởi tạo nhiệt độ (ví dụ)
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

    Nhiệt độ: 25°C, Thời tiết: warm
    

### 4.2.3. Câu điều kiện lồng (nested conditions)


```python
# Khởi tạo độ cao
elevation = 2500 # Khởi tạo độ cao
# Điều kiện lồng nhau
# Giải thích 
""" 
Kiểm tra xem độ cao có trên 2000 mét không, nếu có gán áp suất không khí thấp.
Nếu độ cao cũng trên 3000 mét, gán áp suất không khí rất thấp.
Nếu độ cao không trên 2000 mét, gán áp suất không khí bình thường.
"""
if elevation > 2000:
    air_pressure = "low"
    if elevation > 3000:
        air_pressure = "very low"
else:
    air_pressure = "normal"
print(f"Độ cao: {elevation}m, Áp suất không khí: {air_pressure}")
```

    Độ cao: 2500m, Áp suất không khí: low
    

## 4.3. Toán tử so sánh và logic

Các toán tử này giúp chúng ta tạo điều kiện cho câu lệnh if. Thực ra bản chất cậu lệnh là so sánh xem có thoải mãn điều kiện mình đưa ra không, nếu thoải mãn, làm một nhiệm vụ nào đó, nếu không làm nhiệm vụ khác.

### 4.3.1. Toán tử so sánh

- **So sánh hai chuỗi**


```python
# So sánh chuỗi
language = "Python"
if language == "Python":
    print("Bạn đang sử dụng Python!")
else:
    print("Bạn đang sử dụng ngôn ngữ khác.")
```

    Bạn đang sử dụng Python!
    

- **So sánh hai số**


```python
# So sánh số
temperature = 25
threshold = 30
if temperature > threshold:
    print("Nhiệt độ cao hơn ngưỡng.")
else:
    print("Nhiệt độ không cao hơn ngưỡng.")
```

    Nhiệt độ không cao hơn ngưỡng.
    

### 4.3.2. Toán tử logic

- **Toán tử logic và (`and`)**


```python
# Toán tử logic: and, or, not
temperature = 22
humidity = 65
precipitation = 0
# Toán tử AND
comfortable_weather = temperature >= 20 and precipitation <= 25 and humidity < 70
print(f"Điều kiện thời tiết dễ chịu: {comfortable_weather}")
```

    Điều kiện thời tiết dễ chịu: True
    

- **Toán tử logic hoặc (`or`)**


```python
# Toán tử OR
extreme_weather = temperature > 35 or temperature < 0 or humidity > 90
print(f"Điều kiện thời tiết cực đoan: {extreme_weather}")
```

    Điều kiện thời tiết cực đoan: False
    

- **Toán tử logic `not`**


```python
# Khởi tạo một giá trị để kiểm tra toán tử NOT
small = False
# Toán tử NOT
not_small = not small
print(f"Không nhỏ: {not_small}")
```

    Không nhỏ: True
    

- **Kết hợp nhiều toán tử logic**


```python
# Khởi tạo một số để kiểm tra điều kiện phức hợp
temperature = 22
humidity = 65
precipitation = 0
# Điều kiện phức hợp
if (temperature >= 15 and not temperature > 28 and 
    precipitation == 0 and humidity < 80):
    print("Ngày hoàn hảo để leo núi!")
```

    Ngày hoàn hảo để leo núi!
    

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

    Đà Nẵng là thành phố lớn của Việt Nam
    

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
```

    Đọc nhiệt độ trong 5 ngày:
    Ngày 1: 22°C
    Ngày 2: 24°C
    Ngày 3: 26°C
    Ngày 4: 28°C
    Ngày 5: 30°C
    

- **Vòng lặp `for` với `list`**


```python
# Vòng lặp for với danh sách - dữ liệu tương đối 2023
cities = ["Hà Nội", "TP.HCM", "Đà Nẵng", "Cần Thơ"]
for city in cities:
    print(f"- {city}")
```

    - Hà Nội
    - TP.HCM
    - Đà Nẵng
    - Cần Thơ
    


```python
populations = [8435700, 9077158, 1230000, 1235171]  # Dân số cập nhật 2023
areas = [3358.59, 2061.45, 1285.53, 1408.93]  # Diện tích (km²)
# Vòng lặp và lấy dữ liệu từ hai danh sách cùng lúc sử dụng chỉ số (index)
for i in range(len(cities)):
    print(f"{cities[i]}: {populations[i]:,} người, diện tích: {areas[i]:,.2f} km²")
# Có thể dùng zip để gọn hơn
# for city, population, area in zip(cities, populations, areas):
#     print(f"{city}: {population:,} người, diện tích: {area:,.2f} km²")
```

    Hà Nội: 8,435,700 người, diện tích: 3,358.59 km²
    TP.HCM: 9,077,158 người, diện tích: 2,061.45 km²
    Đà Nẵng: 1,230,000 người, diện tích: 1,285.53 km²
    Cần Thơ: 1,235,171 người, diện tích: 1,408.93 km²
    

- **Dùng vòng lặp `for` và `enumerate`**


```python
# Sử dụng enumerate để có cả chỉ số và giá trị
for index, city in enumerate(cities):
    print(f"{index + 1}. {city}")
```

    1. Hà Nội
    2. TP.HCM
    3. Đà Nẵng
    4. Cần Thơ
    

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
# Bạn cũng có thể lặp qua chỉ các keys hoặc chỉ các values nếu muốn.
```

    Hà Nội: Nhiệt độ 30°C, Độ ẩm 70%
    TP.HCM: Nhiệt độ 32°C, Độ ẩm 75%
    Đà Nẵng: Nhiệt độ 28°C, Độ ẩm 65%
    

### 4.4.2. Vòng lặp `while`

Vòng lặp `while` trong Python được dùng khi bạn muốn lặp lại một đoạn mã miễn là điều kiện còn đúng. Vòng lặp này rất hữu ích cho các tình huống không biết trước số lần lặp và dừng lại chỉ khi điều kiện trở thành False.

- **Vòng lặp while**


```python
# Ví dụ về vòng lặp while
count = 0 # Khởi tạo biến đếm
while count < 5: # Vòng lặp sẽ tiếp tục chạy miễn là count nhỏ hơn 5, khi count đạt 5, điều kiện không còn đúng và vòng lặp sẽ dừng lại
    print(f"Đếm: {count}")
    count += 1 # Tăng biến đếm lên 1 mỗi lần lặp

```

    Đếm: 0
    Đếm: 1
    Đếm: 2
    Đếm: 3
    Đếm: 4
    

- **Điều khiển vòng lặp với `break`**

`break` dùng để thoát khỏi vòng lặp ngay lập tức khi thỏa mãn điều kiện.


```python
counter = 0 
while True: # Vòng lặp vô hạn, nếu không có câu lệnh break bên trong. 
    print(f"Đếm: {counter}")
    counter += 1
    if counter >= 5: # Khi biến đếm đạt 5, điều kiện này sẽ trở thành True và câu lệnh break sẽ được thực thi, 
        # làm cho vòng lặp dừng lại.
        break
```

    Đếm: 0
    Đếm: 1
    Đếm: 2
    Đếm: 3
    Đếm: 4
    

- **Điều khiển vòng lặp với `continue`**

`continue` dùng để bỏ qua phần còn lại của vòng lặp hiện tại và chuyển sang lần lặp tiếp theo khi thỏa mãn điều kiện.


```python
counter = 0 

while counter<5:
    if counter in [3,4]:
        counter += 1
        print(f"Đếm: {counter} - Bỏ qua số này")
        continue 
    print(f"Đếm: {counter}")
    counter += 1
```

    Đếm: 0
    Đếm: 1
    Đếm: 2
    Đếm: 4 - Bỏ qua số này
    Đếm: 5 - Bỏ qua số này
    

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

    Giá trị hiện tại: 1
    Giá trị hiện tại: 2
    Bỏ qua số 3 (continue)
    Giá trị hiện tại: 4
    Giá trị hiện tại: 5
    Giá trị hiện tại: 6
    Giá trị hiện tại: 7
    Dừng vòng lặp tại số 8 (break)
    

## 4.5. Xử lý lỗi

Xử lý lỗi giúp cho debug chương trình dễ dàng hơn khi gặp phải tình huống không mong muốn.

### 4.5.1. Xử lý lỗi với `try-except`

`try-except` được dùng khi bạn dự đoán có thể xảy ra lỗi và muốn xử lý lỗi để chương trình không bị dừng đột ngột. Dưới đây là một vài ví dụ cụ thể.

- **Xử lý lỗi khi thực hiện phép chia**


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

    Lỗi: Không thể chia cho 0!
    

- **Xử lý lỗi không hợp lệ**


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

    Lỗi xác thực tọa độ: Kinh độ không hợp lệ: 200
    

- **Xử lý lỗi khi đọc `file`**


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

    Dữ liệu tệp đã được đọc thành công.
    

### 4.5.2. Xử lý lỗi với `assert`

Sử dụng assert khi muốn bắt lỗi xảy ra. Cấu trức như sau `assert condition, message`. Khi `condition` là `True` chương trình sẽ bỏ qua (thành công), nhưng khi `condition` là False thì lỗi (`error`) sẽ hiển thị.


```python
# sử dụng assert để bắt lỗi 
human_age = -5  # Tuổi không hợp lệ
try:
    assert human_age >= 0, "Tuổi phải là số không âm"
    print(f"Tuổi hợp lệ: {human_age}")
except AssertionError as ae:
    print(f"Lỗi xác thực tuổi: {ae}")
```

    Lỗi xác thực tuổi: Tuổi phải là số không âm
    

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
