# Bài 6: Làm việc với Files

Trong bài học này, bạn sẽ học cách đọc và ghi files trong Python, bao gồm làm việc với CSV files - một định dạng phổ biến cho dữ liệu không gian địa lý.

## 6.1. Mục tiêu học tập
- Mở, đọc và ghi text files
- Làm việc với CSV files sử dụng module `csv` tích hợp sẵn
- Hiểu về đường dẫn files và các phương pháp tốt nhất
- Áp dụng các thao tác file cho dữ liệu không gian địa lý

## 6.2. Đọc Text Files

Hãy đọc một text file đơn giản chứa tên các thành phố.

### 6.2.1. Tạo và viết dữ liệu vào file


```python
# # Tạo một text file mẫu (để minh họa)
cities_list = ["Hà Nội", "TP.HCM", "Đà Nẵng", "Hải Phòng", "Cần Thơ"]
file = r'G:\My Drive\python\geocourse\data\outputs\cities.txt' # đường dẫn file để lưu dữ liệu. Bạn thay thế bằng file của bạn.
with open(file, 'w') as f:
    for city in cities_list:
        f.write(city + '\n') # Ghi mỗi thành phố vào một dòng trong file. Dữ liệu sẽ được ghi dưới dạng string kể cả số.
```

### 6.2.2. Mở file và xem nội dung


```python
# # Đọc file và in nội dung
file = r'G:\My Drive\python\geocourse\data\outputs\cities.txt' # đường dẫn file để lưu dữ liệu. Bạn thay thế bằng file của bạn.
with open(file, 'r') as f:
    # content = f.read() # .read() sẽ đọc toàn bộ nội dung của file và trả về một chuỗi (string)
    content = f.readlines() # .readlines() sẽ đọc toàn bộ nội dung của file và trả về một danh sách (list) các dòng trong file
    print(f"Nội dung file: {content}")
```

    Nội dung file: ['Hà Nội\n', 'TP.HCM\n', 'Đà Nẵng\n', 'Hải Phòng\n', 'Cần Thơ\n']
    

## 6.3. Làm việc với CSV Files

CSV (Comma-Separated Values) files được sử dụng rộng rãi cho dữ liệu dạng bảng, bao gồm các bộ dữ liệu không gian địa lý. 

### 6.3.1. Viết và lưu dữ liệu vào file csv


```python
import csv # Muốn làm việc với file CSV, chúng ta cần import thư viện csv có sẵn trong Python.
```

- **Viết dữ liệu vào file `csv`**


```python
# Tạo một CSV file mẫu
outfile = r'G:\My Drive\python\geocourse\data\outputs\city_data.csv' # đường dẫn file để lưu dữ liệu. Bạn thay thế bằng file của bạn.
# Tạo dữ liệu mẫu 
city_data = [['City', 'Population', 'Latitude', 'Longitude'],
             ['Hà Nội', 8053663, 21.0285, 105.8542],
             ['TP.HCM', 9420000, 10.8231, 106.6297],
             ['Đà Nẵng', 1134000, 16.0544, 108.2022]]
with open(outfile, 'w', newline='') as csvfile:
    writer = csv.writer(csvfile)
    for row in city_data:
        writer.writerow(row)
```

- **Đọc dữ liệu từ file `csv`**


```python
# Đọc CSV file
with open(outfile, 'r') as csvfile:
    reader = csv.reader(csvfile)
    for row in reader:
        print(row)
```

    ['City', 'Population', 'Latitude', 'Longitude']
    ['Hà Nội', '8053663', '21.0285', '105.8542']
    ['TP.HCM', '9420000', '10.8231', '106.6297']
    ['Đà Nẵng', '1134000', '16.0544', '108.2022']
    

### 6.3.2. Đọc CSV dưới dạng dictionaries

Bạn cũng có thể đọc CSV files dưới dạng dictionaries để dễ dàng truy cập các cột theo tên.


```python
file = r'G:\My Drive\python\geocourse\data\outputs\city_data.csv'
with open(file, 'r') as csvfile:
    reader = csv.DictReader(csvfile)
    for row in reader:
        print(f"{row['City']} có dân số {row['Population']} và tọa độ ({row['Latitude']}, {row['Longitude']})")
```

    Hà Nội có dân số 8053663 và tọa độ (21.0285, 105.8542)
    TP.HCM có dân số 9420000 và tọa độ (10.8231, 106.6297)
    Đà Nẵng có dân số 1134000 và tọa độ (16.0544, 108.2022)
    

## Tóm tắt

Bạn đã hoàn thành Bài 5 và học được những kỹ năng quan trọng cho việc làm việc với dữ liệu trong Python.

### Các khái niệm chính đã nắm vững:
- ✅ **Đọc text files**: Mở và đọc nội dung files với open() và read()
- ✅ **Ghi text files**: Lưu dữ liệu vào files với write() và writelines()
- ✅ **Context managers**: Sử dụng with statement để quản lý files an toàn
- ✅ **CSV processing**: Làm việc với module csv để xử lý dữ liệu dạng bảng
- ✅ **CSV reader**: Đọc CSV files dưới dạng lists và dictionaries
- ✅ **CSV writer**: Ghi dữ liệu vào CSV files với headers
- ✅ **File paths**: Hiểu và sử dụng đường dẫn files đúng cách
- ✅ **Ứng dụng thực tế**: Xử lý dữ liệu thành phố với tọa độ địa lý

### Kỹ năng bạn có thể áp dụng:
- Đọc và xử lý datasets từ CSV files cho phân tích GIS
- Xuất kết quả phân tích dữ liệu ra files để chia sẻ
- Quản lý và tổ chức dữ liệu không gian địa lý
- Xử lý dữ liệu thô từ các nguồn khác nhau
- Chuẩn bị dữ liệu cho các thư viện geospatial như GeoPandas
