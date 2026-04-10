# Bài 3: Cấu trúc dữ liệu

Trong bài học này, bạn sẽ học về các cấu trúc dữ liệu tích hợp sẵn của Python: danh sách (lists), bộ giá trị (tuples), từ điển (dictionaries), và tập hợp (sets). Chúng rất quan trọng để tổ chức và thao tác dữ liệu không gian địa lý.

## 3.1. Mục tiêu học tập
- Hiểu và sử dụng lists, tuples, dictionaries, và sets
- Thực hiện các thao tác phổ biến: lập chỉ mục, cắt lát, thêm, xóa, tìm kiếm
- Chọn đúng cấu trúc dữ liệu cho nhiệm vụ của bạn
- Áp dụng cấu trúc dữ liệu vào các bài toán không gian địa lý

## 3.2. Danh sách (Lists)
Danh sách là tập hợp có thứ tự và có thể thay đổi. Sử dụng chúng cho chuỗi các phần tử, như tọa độ hoặc tên thành phố.

### 3.2.1. Tạo danh sách (list)


```python
# Tạo danh sách tên thành phố
cities = ['Hà Nội', 'TP.HCM', 'Đà Nẵng', 'Cần Thơ', 'Hải Phòng'] # Danh sách bắt đầu bằng dấu ngoặc vuông và các phần tử cách nhau bằng dấu phẩy.
print(cities)
```


```python
# Tạo danh sách với loại dữ liệu hỗn hợp
mixed_list = ['Hà Nội', 1010, True, 3.14, None]
print(mixed_list)
```

### 3.2.2. Lập chỉ mục và cắt lát (slicing and indexing)


```python
# Lập chỉ mục và cắt lát
print('Thành phố đầu tiên:', cities[0]) # Lập chỉ mục bắt đầu từ 0
print('Thành phố cuối cùng:', cities[-1]) # Lập chỉ mục âm bắt đầu từ -1
print('Ba thành phố đầu tiên:', cities[:3]) # Cắt lát từ đầu đến chỉ mục 3 (không bao gồm chỉ mục 3)
print('Hai thành phố cuối cùng:', cities[-2:]) # Cắt lát từ chỉ mục -2 đến cuối
```

### 3.2.3. Thêm và xóa phần tử


```python
# Thêm và xóa phần tử
cities.append('Nha Trang') # Thêm 'Nha Trang' vào cuối danh sách
print('Sau khi thêm:', cities)
cities.remove('Đà Nẵng') # Xóa 'Đà Nẵng' khỏi danh sách
print('Sau khi xóa:', cities)

# Độ dài danh sách
print('Số lượng thành phố:', len(cities)) # Hàm len() trả về số lượng phần tử trong danh sách
# Xóa toàn bộ danh sách
cities.clear() 
# Xóa list khỏi bộ nhớ
del cities

```

## 3.3. Bộ giá trị (Tuples)
Bộ giá trị là tập hợp có thứ tự và không thể thay đổi. Sử dụng chúng cho dữ liệu cố định, như tọa độ.

### 3.3.1. Tạo tuple


```python
# Tạo và sử dụng bộ giá trị với tọa độ chính xác
hanoi_coords = (21.0285, 105.8542)  # Tọa độ trung tâm Hà Nội (Hồ Hoàn Kiếm)
print('Tọa độ Hà Nội:', hanoi_coords)

# Giải nén bộ giá trị (unpacking)
lat, lon = hanoi_coords
print(f'Vĩ độ: {lat}, Kinh độ: {lon}')
```

### 3.3.2. Thêm giá trị vào tuple


```python
# Thêm thành phố mới
points = (20, 30, 40)
# Thêm điểm mới vào bộ giá trị, nhưng points hiện tại là một tuple mới.
points += (50,)
print('Bộ giá trị sau khi thêm điểm mới:', points)
```

## 3.4. Từ điển (dictionary)
Từ điển lưu trữ các cặp khóa-giá trị. Sử dụng chúng cho dữ liệu có cấu trúc, như thuộc tính của thành phố.

### 3.4.1. Tạo dictionary 


```python
# Tạo từ điển với dict 
city_data = dict(name='Hà Nội', population=8435700, coordinates=(21.0285, 105.8542), is_coastal=False)
# Tạo từ điển với dữ liệu về Hà Nội
city_data = {
    'name': 'Hà Nội',
    'population': 8435700,  # Dân số 2023
    'coordinates': (21.0285, 105.8542),  # Tọa độ trung tâm (Hồ Hoàn Kiếm)
    'is_coastal': False,
    'districts': ['Hoàn Kiếm', 'Đống Đa', 'Hai Bà Trưng', 'Ba Đình', 'Cầu Giấy'] # Ví dụ 5 quận trung tâm
}
print(city_data)
```

### 3.4.2. Truy cập, cập nhật, và xóa giá trị


```python
# truy cập 
print('Tên thành phố:', city_data['name'])
print('Dân số:', city_data['population'])
```


```python
# Cập nhật dự báo dân số
city_data['population'] = 8500000  
print('Dân số cập nhật:', city_data['population'])
```


```python
# Cập nhật dùng update()
city_data.update({'gdp': 150})  # Giả định
print('Dữ liệu sau khi cập nhật GDP:', city_data)
```


```python
# Thêm khóa mới với diện tích chính xác
city_data['area_km2'] = 3358.59  # Diện tích thành phố Hà Nội
print(city_data)
# Xóa khóa
del city_data['is_coastal']
# Xóa khóa với pop()
population = city_data.pop('population')
# Xóa toàn bộ từ điển
city_data.clear()
# Xóa từ điển khỏi bộ nhớ
del city_data
```


```python
# Liệt kê tất cả khóa
print('Khóa trong từ điển:', list(city_data.keys()))
# Liệt kê tất cả giá trị
print('Giá trị trong từ điển:', list(city_data.values()))
# Liệt kê tất cả cặp khóa-giá trị
print('Cặp khóa-giá trị trong từ điển:', list(city_data.items()))
```


```python
# Truy cập với giá trị mặc định nếu khóa (key) không tồn tại
city = city_data.get('name', 'Không xác định') 
print('Tên thành phố (sử dụng get):', city)
```

## 3.5. Tập hợp (Sets)
Tập hợp là tập hợp không có thứ tự các phần tử duy nhất. Sử dụng chúng để loại bỏ trùng lặp hoặc kiểm tra thành viên.

### 3.5.1. Tạo tập hợp


```python
# Tạo tập hợp bằng set 
cities = set(['Hà Nội', 'TP.HCM', 'Đà Nẵng', 'Cần Thơ', 'Hải Phòng'])
cities 
```


```python
# Tạo và sử dụng tập hợp
visited_cities = {'Hà Nội', 'Đà Nẵng', 'Cần Thơ'}
print('Thành phố đã thăm:', visited_cities)
```


```python
# Tạo tập hợp với phần tử trùng lặp
visited_cities = {'Hà Nội', 'Đà Nẵng', 'Cần Thơ', 'Hà Nội'}
print('Thành phố đã thăm:', visited_cities)  # Trùng lặp đã được loại bỏ
```

### 3.5.2. Các phép toán tập hợp


```python
# Các phép toán tập hợp
northern_cities = {'Hà Nội', 'Hải Phòng', 'Nam Định'}
print('Thành phố miền Bắc đã thăm:', visited_cities & northern_cities) # Giao của hai tập hợp hoặc intersection
print('Tất cả thành phố (hợp):', visited_cities | northern_cities) # Hợp của hai tập hợp hoặc union
print('Thành phố đã thăm nhưng không ở miền Bắc:', visited_cities - northern_cities) # Hiệu của hai tập hợp hoặc difference
```

### 3.5.3. Thêm và xóa tập hợp


```python
# Thêm thành phố mới
visited_cities.add('TP.HCM')
print('Cập nhật thành phố đã thăm:', visited_cities)
# Xóa thành phố
visited_cities.remove('Cần Thơ')
print('Sau khi xóa:', visited_cities)
```


```python
# Tạo tập hợp trống sau đó thêm vào 
mset = set()  # Tạo tập hợp trống
for city in ['Hà Nội', 'Đà Nẵng', 'Cần Thơ', 'Hà Nội']:
    mset.add(city)
print('Tập hợp sau khi thêm:', mset)
```

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
