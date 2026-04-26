# Bài 3: Cấu trúc dữ liệu

Trong bài học này, bạn sẽ học về các cấu trúc dữ liệu tích hợp sẵn của Python, bao gồm danh sách (lists), bộ giá trị (tuples), từ điển (dictionaries), và tập hợp (sets). Chúng rất quan trọng để tổ chức và thao tác dữ liệu không gian địa lý.

## 3.1. Mục tiêu học tập
- Hiểu và sử dụng lists, tuples, dictionaries, và sets
- Thực hiện các thao tác phổ biến: lập chỉ mục, cắt lát, thêm, xóa, tìm kiếm
- Chọn đúng cấu trúc dữ liệu cho nhiệm vụ của bạn
- Áp dụng cấu trúc dữ liệu vào các bài toán không gian địa lý

## 3.2. Danh sách (Lists)
Danh sách là tập hợp có thứ tự và có thể thay đổi. Chúng được sử dụng để lưu chuỗi các phần tử và được tạo ra sử dụng hai phương pháp chính là dùng dấu ngoặc vuông ([]) hoặc là dùng từ khóa `list()`.

### 3.2.1. Tạo danh sách (list)

- **Tạo danh sách sử dụng dấu ngoặc vuông**


```python
# Tạo danh sách tên thành phố sử dụng dấu ngoặc vuông
cities = ['Hà Nội', 'TP.HCM', 'Đà Nẵng', 'Cần Thơ', 'Hải Phòng'] # Danh sách bắt đầu bằng dấu ngoặc vuông và các phần tử cách nhau bằng dấu phẩy.
print(f"Danh sách các thành phố: {cities}")
```

    Danh sách các thành phố: ['Hà Nội', 'TP.HCM', 'Đà Nẵng', 'Cần Thơ', 'Hải Phòng']
    

- **Tạo danh sách với dữ liệu hỗn hợp**


```python
# Tạo danh sách với loại dữ liệu hỗn hợp
mixed_list = ['Hà Nội', 1010, True, 3.14, None]
print(f"Danh sách hỗn hợp: {mixed_list}")
```

    Danh sách hỗn hợp: ['Hà Nội', 1010, True, 3.14, None]
    

- **Tạo danh sách dùng từ khóa `list` với `range`**


```python
# Tạo danh sách dùng list 
numbers = list(range(1, 11)) # Tạo danh sách các số từ 1 đến 10
print(f"Danh sách các số từ 1 đến 10: {numbers}")
```

    Danh sách các số từ 1 đến 10: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    

### 3.2.2. Lập chỉ mục và cắt lát (slicing and indexing)

Lập chỉ mục danh sách thực hiện theo cách truy cập phần tử thông qua vị trí (chỉ số) của nó trong danh sách, bắt đầu từ 0. Cắt lát (slicing) cho phép lấy ra một phần của danh sách dựa trên khoảng chỉ số như sau: `list[start : stop : step]`


```python
# Lập chỉ mục và cắt lát
print('Thành phố đầu tiên:', cities[0]) # Lập chỉ mục bắt đầu từ 0
print('Thành phố cuối cùng:', cities[-1]) # Lập chỉ mục âm bắt đầu từ -1
print('Ba thành phố đầu tiên:', cities[:3]) # Cắt lát từ đầu đến chỉ mục 3 (không bao gồm chỉ mục 3)
print('Hai thành phố cuối cùng:', cities[-2:]) # Cắt lát từ chỉ mục -2 đến cuối
```

    Thành phố đầu tiên: Hà Nội
    Thành phố cuối cùng: Hải Phòng
    Ba thành phố đầu tiên: ['Hà Nội', 'TP.HCM', 'Đà Nẵng']
    Hai thành phố cuối cùng: ['Cần Thơ', 'Hải Phòng']
    

### 3.2.3. Thêm và xóa phần tử

- **Thêm phần tử hoặc thay đổi phần tử trong `list`**


```python
cities = ['Hà Nội', 'TP.HCM', 'Đà Nẵng', 'Cần Thơ', 'Hải Phòng']
cities[1] = 'Sài Gòn' # Thay đổi phần tử tại chỉ mục 1
# Thêm phần tử mới vào danh sách
cities.append('Huế') # Thêm 'Huế' vào cuối danh sách
print(f"Danh sách các thành phố sau khi cập nhật: {cities}")    
```

    Danh sách các thành phố sau khi cập nhật: ['Hà Nội', 'Sài Gòn', 'Đà Nẵng', 'Cần Thơ', 'Hải Phòng', 'Huế']
    

- **Xóa phần tử trong `list`**


```python
cities = ['Hà Nội', 'TP.HCM', 'Đà Nẵng', 'Cần Thơ', 'Hải Phòng']
# Xóa phần tử khỏi danh sách
del cities[2] # Xóa phần tử tại chỉ mục 2 (Đà Nẵng)
print(f"Danh sách sau khi xóa phần tử tại chỉ mục 2: {cities}")
cities.remove('Cần Thơ') # Xóa phần tử có giá trị 'Cần Thơ'
print(f"Danh sách sau khi xóa phần tử 'Cần Thơ': {cities}")
cities.pop() # Xóa phần tử cuối cùng trong danh sách (Hải Phòng)
print(f"Danh sách sau khi xóa phần tử cuối cùng: {cities}")
# xóa toàn bộ danh sách
cities.clear() # Xóa tất cả phần tử trong danh sách, nhưng danh sách vẫn tồn tại
print(f"Danh sách sau khi xóa tất cả phần tử: {cities}")
# Xóa hoàn toàn danh sách khỏi bộ nhớ
del cities # Xóa hoàn toàn danh sách khỏi bộ nhớ
```

    Danh sách sau khi xóa phần tử tại chỉ mục 2: ['Hà Nội', 'TP.HCM', 'Cần Thơ', 'Hải Phòng']
    Danh sách sau khi xóa phần tử 'Cần Thơ': ['Hà Nội', 'TP.HCM', 'Hải Phòng']
    Danh sách sau khi xóa phần tử cuối cùng: ['Hà Nội', 'TP.HCM']
    Danh sách sau khi xóa tất cả phần tử: []
    

- **Độ dài danh sách**


```python
cities = ['Hà Nội', 'TP.HCM', 'Đà Nẵng', 'Cần Thơ', 'Hải Phòng']
print(f"Số lượng thành phố: {len(cities)}") # Sử dụng hàm len() để đếm số phần tử trong danh sách
```

    Số lượng thành phố: 5
    

## 3.3. Bộ giá trị (Tuples)
Bộ giá trị là tập hợp có thứ tự và không thể thay đổi. Sử dụng chúng cho dữ liệu cố định, như tọa độ, và được tạo ra dùng dấu ngoặc đơn `()` hoặc từ khóa `tuple()`.

### 3.3.1. Tạo tuple

- **Tạo `tuple` dùng dấu ngoặc đơn hoặc từ khóa `tuple`**


```python
# Tạo tuple với dấu ngoặc đơn
long_lat = (21.0285, 105.8542)  # Tọa độ trung tâm Hà Nội (Hồ Hoàn Kiếm)
print('Tọa độ Hà Nội:', long_lat)
# Tạo tuple với từ khóa tuple
long_lat = tuple([21.0285, 105.8542])  # Tạo tuple từ một danh sách
print('Tọa độ Hà Nội:', long_lat)
```

    Tọa độ Hà Nội: (21.0285, 105.8542)
    Tọa độ Hà Nội: (21.0285, 105.8542)
    

- **Tạo tuple một phần tử**


```python
# Tạo tuple một phần tử
single_tuple = (42,)  # Cần dấu phẩy để tạo tuple một phần tử
print('Tuple một phần tử:', single_tuple)
```

    Tuple một phần tử: (42,)
    

### 3.3.2. Thêm giá trị vào tuple


```python
# Thêm thành phố mới
points = (20, 30, 40)
# nguyên tắc là không thể thay đổi tuple, nhưng points hiện tại là một tuple mới.
points += (50,)
print('Bộ giá trị sau khi thêm điểm mới:', points)
```

    Bộ giá trị sau khi thêm điểm mới: (20, 30, 40, 50)
    

## 3.4. Từ điển (dictionary)
Từ điển lưu trữ các cặp khóa-giá trị. Sử dụng chúng cho dữ liệu có cấu trúc, và được tạo ra sử dụng dấu ngoặc nhọn `{}` hoặc từ khóa `dict()`.

### 3.4.1. Tạo dictionary 

- **Tạo từ điển từ dấu ngoặc nhọn `{}`**


```python
# Tạo từ điển với dấu ngoặc nhọn
city_data = {
    'name': 'Hà Nội',
    'population': 8435700,  # Dân số 2023
    'coordinates': (21.0285, 105.8542),  # Tọa độ trung tâm (Hồ Hoàn Kiếm)
    'is_coastal': False,
    'districts': ['Hoàn Kiếm', 'Đống Đa', 'Hai Bà Trưng', 'Ba Đình', 'Cầu Giấy'] # Ví dụ 5 quận trung tâm
}
print('Dữ liệu thành phố Hà Nội:', city_data)
```

    Dữ liệu thành phố Hà Nội: {'name': 'Hà Nội', 'population': 8435700, 'coordinates': (21.0285, 105.8542), 'is_coastal': False, 'districts': ['Hoàn Kiếm', 'Đống Đa', 'Hai Bà Trưng', 'Ba Đình', 'Cầu Giấy']}
    

- **Tạo từ điển với từ khóa `dict`**


```python
# Tạo từ điển với dict 
city_data = dict(name='Hà Nội', population=8435700, coordinates=(21.0285, 105.8542), is_coastal=False)
print('Dữ liệu thành phố Hà Nội:', city_data)
```

    Dữ liệu thành phố Hà Nội: {'name': 'Hà Nội', 'population': 8435700, 'coordinates': (21.0285, 105.8542), 'is_coastal': False}
    

- **Tạo từ điển lồng nhau**


```python
city = {
    'name': 'Hà Nội',
    'coordinates': (21.0285, 105.8542),
    'districts': ['Hoàn Kiếm', 'Đống Đa', 'Hai Bà Trưng', 'Ba Đình', 'Cầu Giấy'],
    'stats': {
        'population': 8435700,
        'area_km2': 3323.0,
        'is_coastal': False
    }
}
print('Dữ liệu thành phố Hà Nội:', city)
```

    Dữ liệu thành phố Hà Nội: {'name': 'Hà Nội', 'coordinates': (21.0285, 105.8542), 'districts': ['Hoàn Kiếm', 'Đống Đa', 'Hai Bà Trưng', 'Ba Đình', 'Cầu Giấy'], 'stats': {'population': 8435700, 'area_km2': 3323.0, 'is_coastal': False}}
    

### 3.4.2. Truy cập, cập nhật, và xóa giá trị

- **Truy cập dùng từ khóa `key`**


```python
# tạo từ điển
city = {
    'name': 'Hà Nội',
    'coordinates': (21.0285, 105.8542)
}
# truy cập 
print('Tên thành phố:', city['name'])
print('Tọa độ:', city['coordinates'])
```

    Tên thành phố: Hà Nội
    Tọa độ: (21.0285, 105.8542)
    

- **Truy cập giá trị dùng `get`**

Dùng `dict.get(key)` để lấy giá trị trong dictionary an toàn, không gây lỗi nếu key không tồn tại (trả về None hoặc giá trị mặc định nếu có).


```python
city = {
    'name': 'Hà Nội',
    'is_coastal': False
}
# Truy cập tên
print('Tên thành phố:', city.get('city_name', 'Không có khóa city_name'))  # Sử dụng get() để truy cập giá trị, trả về 'Không có dữ liệu' nếu khóa không tồn tại
```

    Tên thành phố: Không có khóa city_name
    

- **Cập nhật `dictionary` dùng `key`** 


```python
# tạo từ điển
city = {
    'name': 'Hà Nội',
    'is_coastal': False
}
# Cập nhật tên
city['name'] = 'Thủ đô Hà Nội'
```

- **Cập nhật `dictionary` dùng `update`**

Ngoài cách cập nhật giá trị sử dụng `key`, bạn cũng có thể sử dụng `update` để cập nhật giá trị của `key` trong `dictionary`.


```python
# tạo từ điển
city = {
    'name': 'Hà Nội',
    'is_coastal': False
}
# Cập nhật tên và tạo thêm thuộc tính mới. Đầu vào là một từ điển với key:value.
city.update({'name': 'Thanh Hóa',  'river': 'Tô Lịch'})  # Cập nhật bằng cách sử dụng update(), cập nhật xảy ra inplace (có nghĩa là city được cập nhật trực tiếp mà không cần gán lại)
print('Dữ liệu sau khi cập nhật:', city)
```

    Dữ liệu sau khi cập nhật: {'name': 'Thanh Hóa', 'is_coastal': False, 'river': 'Tô Lịch'}
    

- **Thêm `key` mới vào `dictionary`**


```python
# Tạo từ điển
city_data = {
    'name': 'Hà Nội',
    'population': 8435700,
    'is_coastal': False
}
# Thêm khóa mới tên sông
city_data['river'] = 'Tô Lịch'  # Tên sông chảy qua thành phố Hà Nội
print('Dữ liệu sau khi thêm khóa mới:', city_data)
```

    Dữ liệu sau khi thêm khóa mới: {'name': 'Hà Nội', 'population': 8435700, 'is_coastal': False, 'river': 'Tô Lịch'}
    

- **Liệt kê `keys` và giá trị hoặc cả hai**


```python
# Tạo ra từ điển
city_data = {
    'name': 'Hà Nội',
    'population': 8435700,
    'is_coastal': False
}
# Liệt kê tất cả khóa
print('Khóa trong từ điển:', list(city_data.keys())) # Chuyển đổi keys thành danh sách [key1, key2, ...] 
# Liệt kê tất cả giá trị
print('Giá trị trong từ điển:', list(city_data.values())) # Chuyển đổi values thành danh sách [value1, value2, ...] 
# Liệt kê tất cả cặp khóa-giá trị
print('Cặp khóa-giá trị trong từ điển:', list(city_data.items())) # Chuyển đổi items thành danh sách [(key, value), ...] để hiển thị
```

    Khóa trong từ điển: ['name', 'population', 'is_coastal']
    Giá trị trong từ điển: ['Hà Nội', 8435700, False]
    Cặp khóa-giá trị trong từ điển: [('name', 'Hà Nội'), ('population', 8435700), ('is_coastal', False)]
    

- **Xóa từ điển**


```python
# Tạo ra từ điển
city_data = {
    'name': 'Hà Nội',
    'population': 8435700,
    'is_coastal': False
}
# Xóa khóa name
del city_data['name']
print('Dữ liệu sau khi xóa khóa name:', city_data)
# Xóa toàn bộ từ điển
city_data.clear()
# Xóa từ điển khỏi bộ nhớ
del city_data
```

    Dữ liệu sau khi xóa khóa name: {'population': 8435700, 'is_coastal': False}
    

## 3.5. Tập hợp (Sets)
Tập hợp là tập hợp không có thứ tự các phần tử duy nhất. Sử dụng chúng để loại bỏ trùng lặp hoặc kiểm tra thành viên. Chúng được tạo ra sử dụng dấu ngoặc nhọn `{}` hoặc từ khóa `set()`.

### 3.5.1. Tạo tập hợp (set)

- **Tạo `set` bằng dấu ngoặc nhọn**


```python
# Tạo tập hợp với dấu ngoặc nhọn
visited_cities = {'Hà Nội', 'Đà Nẵng', 'Cần Thơ', 'Hà Nội'}
print('Thành phố đã thăm:', visited_cities) # Tập hợp tự động loại bỏ phần tử trùng lặp
```

    Thành phố đã thăm: {'Hà Nội', 'Đà Nẵng', 'Cần Thơ'}
    

- **Tạo `set` bằng từ khóa `set` với `list`**


```python
# Tạo tập hợp bằng set 
cities = set(['Hà Nội', 'TP.HCM', 'Đà Nẵng', 'Cần Thơ', 'Hải Phòng'])
print(f"Danh sách thành phố {cities}")
```

    Danh sách thành phố {'Hà Nội', 'Hải Phòng', 'TP.HCM', 'Cần Thơ', 'Đà Nẵng'}
    

### 3.5.2. Các phép toán tập hợp

- **Giao của hai tập hợp**


```python
# Tạo tập hợp
cities = {'Hà Nội', 'Hải Phòng', 'Nam Định'} 
visited_cities = {"Hà Nội", "Vĩnh Phúc", "Nam Định"}
# Giao của hai tập hợp
intersection = cities & visited_cities
print('Các thành phố đã được thăm và có trong danh sách:', intersection)
```

    Các thành phố đã được thăm và có trong danh sách: {'Nam Định', 'Hà Nội'}
    

- **Hợp của hai tập hợp (`union`)**


```python
# Tạo tập hợp
cities = {'Hà Nội', 'Hải Phòng', 'Nam Định'} 
visited_cities = {"Hà Nội", "Vĩnh Phúc", "Nam Định"}
# Hợp của hai tập hợp
union = cities | visited_cities
print('Các thành phố trong danh sách hoặc đã được thăm:', union)
```

    Các thành phố trong danh sách hoặc đã được thăm: {'Hải Phòng', 'Vĩnh Phúc', 'Nam Định', 'Hà Nội'}
    

- **Hiệu của hai tập hợp (`difference`)**


```python
# Tạo tập hợp
cities = {'Hà Nội', 'Hải Phòng', 'Nam Định'} 
visited_cities = {"Hà Nội", "Vĩnh Phúc", "Nam Định"}
# Hiệu của hai tập hợp
difference = cities - visited_cities
print('Các thành phố trong danh sách nhưng chưa được thăm:', difference)
```

    Các thành phố trong danh sách nhưng chưa được thăm: {'Hải Phòng'}
    

### 3.5.3. Thêm và xóa tập hợp

- **Thêm phần tử mới**


```python
# Tạo tập hợp
cities = {"Hà Nội", "Phú Thọ"}
# Thêm Vĩnh Phúc vào cities 
cities.add('Vĩnh Phúc')
print(f"Tập hợp: {cities}")
# Hoặc tạo ra 1 set trống và thêm phần tử vào đó
new_cities = set()  # Tạo set trống
new_cities.add('Huế')
print(f"Tập hợp new_cities: {new_cities}")
```

    Tập hợp: {'Phú Thọ', 'Hà Nội', 'Vĩnh Phúc'}
    Tập hợp new_cities: {'Huế'}
    

- **Xóa tập hợp**


```python
# Tạo tập hợp
cities = {"Hà Nội", "Phú Thọ"}
# Xóa thành phố
cities.remove('Hà Nội')
print('Sau khi xóa:', cities)
# Xóa toàn bộ khỏi bộ nhớ
del cities
```

    Sau khi xóa: {'Phú Thọ'}
    

## Tóm tắt

Bạn đã học xong các cấu trúc dữ liệu cơ bản của Python:

### Kiến thức đã học:
- ✅ **Lists**: Danh sách có thứ tự, có thể thay đổi - dùng cho chuỗi dữ liệu
- ✅ **Tuples**: Bộ giá trị có thứ tự, không thể thay đổi - dùng cho dữ liệu cố định
- ✅ **Dictionaries**: Từ điển lưu cặp khóa-giá trị - dùng cho dữ liệu có cấu trúc
- ✅ **Sets**: Tập hợp không có thứ tự, phần tử duy nhất - dùng để loại trùng lặp

### Kỹ năng thực hành:
- Chọn đúng cấu trúc dữ liệu cho từng bài toán
- Thao tác cơ bản: thêm, xóa, tìm kiếm, lặp qua các phần tử
- Xử lý dữ liệu địa lý với các cấu trúc khác nhau
