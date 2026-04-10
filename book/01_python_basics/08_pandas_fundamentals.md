# Bài 8: Pandas cho Dữ liệu

Pandas là thư viện chính cho việc thao tác và phân tích dữ liệu trong Python. Nó rất quan trọng để làm việc với dữ liệu không gian địa lý dạng bảng.

## 8.1. Mục tiêu học tập
- Tạo và sử dụng DataFrames
- Đọc và ghi CSV hoặc excel files
- Lọc, sắp xếp và tóm tắt dữ liệu
- Các phương thức nâng cao với DataFrames
- Áp dụng Pandas cho bộ dữ liệu không gian địa lý


```python
import pandas as pd
print('Phiên bản Pandas:', pd.__version__)
```

## 8.2. Giới thiệu về Pandas

Pandas cung cấp DataFrame cho việc phân tích dữ liệu mạnh mẽ.

### 8.2.1. Tạo DataFrame từ dictionary


```python
# Tạo DataFrame từ một dictionary
data = {
    'City': ['Hà Nội', 'TP.HCM', 'Đà Nẵng'],
    'Population': [8053663, 9420000, 1134000],
    'Latitude': [21.0285, 10.8231, 16.0544],
    'Longitude': [105.8542, 106.6297, 108.2022]
}
df = pd.DataFrame(data)
```

### 8.2.2. Tạo DataFrame từ danh sách của từ điển (list of dictionary) hoặc danh sách lồng (list of lists)


```python
# Từ một list các dictionaries
cities = [
    {'City': 'Hải Phòng', 'Population': 2028220, 'Latitude': 20.8449, 'Longitude': 106.6881},
    {'City': 'Cần Thơ', 'Population': 1235000, 'Latitude': 10.0452, 'Longitude': 105.7469}
]
df = pd.DataFrame(cities)
df
```


```python
# Tạo DataFrame từ list các lists với chỉ mục và tên cột tùy chỉnh
data = [
    ['Hà Nội', 8053663, 21.0285, 105.8542],
    ['TP.HCM', 9420000, 10.8231, 106.6297],
    ['Đà Nẵng', 1134000, 16.0544, 108.2022]
]
df = pd.DataFrame(data, columns=['City', 'Population', 'Latitude', 'Longitude'])
df
```

## 8.3. Đọc và ghi CSV hoặc excel Files

CSV và excel là định dạng phổ biến nhất cho dữ liệu dạng bảng.

### 8.3.1. Đọc và ghi ra file csv


```python
# Ghi DataFrame ra CSV
df.to_csv('cities.csv', index=False)
# Đọc DataFrame từ CSV
df_read = pd.read_csv('cities.csv')
```

### 8.3.2. Đọc và ghi ra file excel


```python
# Đọc DataFrame từ Excel
df_excel = pd.read_excel('cities.xlsx')  # Giả sử bạn có file cities.xlsx
# Ghi DataFrame ra Excel
df.to_excel('cities_output.xlsx', index=False)
```

### 8.3.3. Đọc dữ liệu online


```python
# Đọc dữ liệu từ URL
url = 'https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv'
df = pd.read_csv(url)
# Lưu DataFrame ra file CSV
df.to_csv('iris_data.csv', index=False)
# Lưu DataFrame ra file Excel
df.to_excel('iris_data.xlsx', index=False)
```

## 8.4. Lọc và sắp xếp

Bạn có thể lọc các hàng và sắp xếp dữ liệu một cách dễ dàng.

### 8.4.1. Lọc và nhóm


```python
# Dữ liệu Olympiad toán 2018 từ github bên dưới
df = pd.read_csv('https://raw.githubusercontent.com/leomtz/apmowebsite/refs/heads/master/data/data_clean/scoretable-2018-clean.csv')
df.head()
```


```python
# Lọc những thí sinh có rank dưới 3
ranked_df = df[df['rank'] < 3]
# Sắp xếp theo điểm tổng (Total Score) giảm dần
sorted_df = df.sort_values(by='total', ascending=False)
```


```python
# Tóm tắt điểm trung bình theo quốc gia
mean_scores = df.groupby('country')['total'].mean().reset_index().sort_values(by='total', ascending=False)
```

### 8.4.2. Tạo cột mới và gộp các DataFrames


```python
# Tạo ra cột mới fullname bằng cách ghép first_name và last_name
df['fullname'] = df['first'] + ' ' + df['last']
df.head()
```


```python
# Gộp 2 dataframes theo cột chung 'id'
df1 = pd.DataFrame({
    'id': [1, 2, 3],
    'value1': ['A', 'B', 'C']
})
df2 = pd.DataFrame({
    'id': [2, 3, 4],
    'value2': ['D', 'E', 'F']
})
merged_df = pd.merge(df1, df2, on='id', how='inner') # Thay 'inner' bằng 'outer', 'left', hoặc 'right' để thay đổi kiểu gộp
merged_df
```

## 8.5. Tóm tắt dữ liệu

Pandas giúp dễ dàng lấy thống kê và tóm tắt.


```python
# tóm tắt dữ liệu df 
summary = df.describe()
# Tóm tắt chỉ cột số liệu
numeric_summary = df.describe(include=['number'])
# Tóm tắt chỉ cột đối tượng
categorical_summary = df.describe(include=['object'])
```


```python
# Tóm tắt dữ liệu nhiều thông số 
stats = df.groupby('country').agg({
    'total': ['mean', 'max', 'min'],
    'rank': ['mean', 'max', 'min']
})
stats.reset_index()
```

## Tóm tắt

Bạn đã hoàn thành Bài 8 và học được Pandas - thư viện quan trọng nhất cho data analysis trong Python.

### Các khái niệm chính đã nắm vững:
- ✅ **DataFrames**: Cấu trúc dữ liệu 2D mạnh mẽ cho việc xử lý dữ liệu tabular
- ✅ **Tạo DataFrames**: Từ dictionaries, lists, và các nguồn dữ liệu khác nhau
- ✅ **File I/O**: Đọc và ghi CSV, Excel files với read_csv(), to_csv(), read_excel()
- ✅ **Data filtering**: Lọc dữ liệu với boolean indexing và điều kiện logic
- ✅ **Sorting**: Sắp xếp dữ liệu với sort_values() theo các columns khác nhau
- ✅ **Statistical analysis**: Tính toán thống kê với mean(), count(), describe()
- ✅ **Data exploration**: Khám phá và hiểu cấu trúc của datasets
- ✅ **Ứng dụng geospatial**: Xử lý dữ liệu thành phố với coordinates và attributes

### Kỹ năng bạn có thể áp dụng:
- Xử lý và phân tích datasets geospatial dạng tabular một cách hiệu quả
- Import/export dữ liệu từ nhiều nguồn khác nhau (CSV, Excel, web APIs)
- Thực hiện exploratory data analysis (EDA) cho các bộ dữ liệu địa lý
- Lọc và truy vấn dữ liệu based on spatial và non-spatial attributes
- Chuẩn bị dữ liệu sạch để sử dụng với GeoPandas và các GIS libraries
