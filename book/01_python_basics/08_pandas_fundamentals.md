# Bài 8: Cơ bản về Pandas

Pandas là thư viện chính cho việc thao tác và phân tích dữ liệu trong Python. Nó rất quan trọng để làm việc với dữ liệu không gian địa lý dạng bảng.

## 8.1. Mục tiêu học tập
- Tạo và sử dụng DataFrames
- Đọc và ghi CSV hoặc excel files
- Lọc, sắp xếp và tóm tắt dữ liệu
- Các phương thức nâng cao với DataFrames
- Áp dụng Pandas cho bộ dữ liệu không gian địa lý


```python
import pandas as pd
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
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Population</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hà Nội</td>
      <td>8053663</td>
      <td>21.0285</td>
      <td>105.8542</td>
    </tr>
    <tr>
      <th>1</th>
      <td>TP.HCM</td>
      <td>9420000</td>
      <td>10.8231</td>
      <td>106.6297</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Đà Nẵng</td>
      <td>1134000</td>
      <td>16.0544</td>
      <td>108.2022</td>
    </tr>
  </tbody>
</table>
</div>



### 8.2.2. Tạo DataFrame từ danh sách của từ điển (list of dictionary) hoặc danh sách lồng (list of lists)


```python
# Từ một list các dictionaries
cities = [
    {'City': 'Hải Phòng', 'Population': 2028220, 'Latitude': 20.8449, 'Longitude': 106.6881},
    {'City': 'Cần Thơ', 'Population': 1235000, 'Latitude': 10.0452, 'Longitude': 105.7469}
]
df = pd.DataFrame(cities)
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Population</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hải Phòng</td>
      <td>2028220</td>
      <td>20.8449</td>
      <td>106.6881</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Cần Thơ</td>
      <td>1235000</td>
      <td>10.0452</td>
      <td>105.7469</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Tạo DataFrame từ list các lists với chỉ mục và tên cột tùy chỉnh
data = [
    ['Hà Nội', 8053663, 21.0285, 105.8542],
    ['TP.HCM', 9420000, 10.8231, 106.6297],
    ['Đà Nẵng', 1134000, 16.0544, 108.2022]
]
df = pd.DataFrame(data, columns=['City', 'Population', 'Latitude', 'Longitude']) # Thêm cột tương ứng
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Population</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hà Nội</td>
      <td>8053663</td>
      <td>21.0285</td>
      <td>105.8542</td>
    </tr>
    <tr>
      <th>1</th>
      <td>TP.HCM</td>
      <td>9420000</td>
      <td>10.8231</td>
      <td>106.6297</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Đà Nẵng</td>
      <td>1134000</td>
      <td>16.0544</td>
      <td>108.2022</td>
    </tr>
  </tbody>
</table>
</div>



## 8.3. Đọc và ghi CSV hoặc excel Files

CSV và excel là định dạng phổ biến nhất cho dữ liệu dạng bảng.

### 8.3.1. Đọc và ghi ra file csv


```python
# Ghi DataFrame ra CSV
df.to_csv(r'G:\My Drive\python\geocourse\data\outputs\cities.csv', index=False)
# Đọc DataFrame từ CSV
df_read = pd.read_csv(r'G:\My Drive\python\geocourse\data\outputs\cities.csv')
df_read.head()  
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>City</th>
      <th>Population</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Hà Nội</td>
      <td>8053663</td>
      <td>21.0285</td>
      <td>105.8542</td>
    </tr>
    <tr>
      <th>1</th>
      <td>TP.HCM</td>
      <td>9420000</td>
      <td>10.8231</td>
      <td>106.6297</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Đà Nẵng</td>
      <td>1134000</td>
      <td>16.0544</td>
      <td>108.2022</td>
    </tr>
  </tbody>
</table>
</div>



### 8.3.2. Đọc và ghi ra file excel


```python
# Đọc DataFrame từ Excel
df_excel = pd.read_excel(r'G:\My Drive\python\geocourse\data\outputs\cities.xlsx')  # Giả sử bạn có file cities.xlsx
# Ghi DataFrame ra Excel
df.to_excel(r'G:\My Drive\python\geocourse\data\outputs\cities_output.xlsx', index=False)
```

### 8.3.3. Đọc dữ liệu từ url


```python
# Đọc dữ liệu từ URL
url = 'https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv'
df = pd.read_csv(url)
# Lưu DataFrame ra file CSV
df.to_csv(r'G:\My Drive\python\geocourse\data\outputs\iris_data.csv', index=False)
# Lưu DataFrame ra file Excel
df.to_excel(r'G:\My Drive\python\geocourse\data\outputs\iris_data.xlsx', index=False)
```

## 8.4. Lọc và sắp xếp

Bạn có thể lọc các hàng và sắp xếp dữ liệu một cách dễ dàng.

### 8.4.1. Lọc và nhóm


```python
# Dữ liệu Olympiad toán 2018 từ github bên dưới
df = pd.read_csv('https://raw.githubusercontent.com/leomtz/apmowebsite/refs/heads/master/data/data_clean/scoretable-2018-clean.csv')
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>code</th>
      <th>country</th>
      <th>rank</th>
      <th>last</th>
      <th>first</th>
      <th>sex</th>
      <th>p1</th>
      <th>p2</th>
      <th>p3</th>
      <th>p4</th>
      <th>p5</th>
      <th>total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ARG</td>
      <td>Argentina</td>
      <td>1</td>
      <td>MASLIAH</td>
      <td>JULIÁN</td>
      <td>M</td>
      <td>7</td>
      <td>6</td>
      <td>1</td>
      <td>5</td>
      <td>0</td>
      <td>19</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ARG</td>
      <td>Argentina</td>
      <td>2</td>
      <td>FLESCHLER</td>
      <td>IAN</td>
      <td>M</td>
      <td>7</td>
      <td>7</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>18</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ARG</td>
      <td>Argentina</td>
      <td>3</td>
      <td>SOTO</td>
      <td>CARLOS MIGUEL</td>
      <td>M</td>
      <td>7</td>
      <td>0</td>
      <td>4</td>
      <td>5</td>
      <td>0</td>
      <td>16</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ARG</td>
      <td>Argentina</td>
      <td>4</td>
      <td>DI SANZO</td>
      <td>BRUNO</td>
      <td>M</td>
      <td>7</td>
      <td>3</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
      <td>15</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ARG</td>
      <td>Argentina</td>
      <td>5</td>
      <td>CASSIA</td>
      <td>NICOLÁS</td>
      <td>M</td>
      <td>7</td>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>13</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Lọc những thí sinh có rank dưới 3
ranked_df = df[df['rank'] < 3]
# Sắp xếp theo điểm tổng (Total Score) giảm dần
sorted_df = df.sort_values(by='total', ascending=False)
sorted_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>code</th>
      <th>country</th>
      <th>rank</th>
      <th>last</th>
      <th>first</th>
      <th>sex</th>
      <th>p1</th>
      <th>p2</th>
      <th>p3</th>
      <th>p4</th>
      <th>p5</th>
      <th>total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>342</th>
      <td>USA</td>
      <td>United States of America</td>
      <td>1</td>
      <td>Gu</td>
      <td>Andrew</td>
      <td>M</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>35</td>
    </tr>
    <tr>
      <th>343</th>
      <td>USA</td>
      <td>United States of America</td>
      <td>2</td>
      <td>Wan</td>
      <td>Edward</td>
      <td>M</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>35</td>
    </tr>
    <tr>
      <th>146</th>
      <td>KOR</td>
      <td>Republic of Korea</td>
      <td>2</td>
      <td>Kim</td>
      <td>Ji Min</td>
      <td>M</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>35</td>
    </tr>
    <tr>
      <th>147</th>
      <td>KOR</td>
      <td>Republic of Korea</td>
      <td>3</td>
      <td>Kim</td>
      <td>Dain</td>
      <td>F</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>35</td>
    </tr>
    <tr>
      <th>145</th>
      <td>KOR</td>
      <td>Republic of Korea</td>
      <td>1</td>
      <td>Kwon</td>
      <td>Sunghyun</td>
      <td>M</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>7</td>
      <td>35</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Tóm tắt điểm trung bình theo quốc gia
mean_scores = df.groupby('country')['total'].mean().reset_index().sort_values(by='total', ascending=False)
mean_scores.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>country</th>
      <th>total</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>27</th>
      <td>Republic of Korea</td>
      <td>32.0</td>
    </tr>
    <tr>
      <th>38</th>
      <td>United States of America</td>
      <td>30.6</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Japan</td>
      <td>25.4</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Singapore</td>
      <td>23.1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Canada</td>
      <td>22.0</td>
    </tr>
  </tbody>
</table>
</div>



### 8.4.2. Tạo cột mới và gộp các DataFrames


```python
# Tạo ra cột mới fullname bằng cách ghép first_name và last_name
df['fullname'] = df['first'] + ' ' + df['last']
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>code</th>
      <th>country</th>
      <th>rank</th>
      <th>last</th>
      <th>first</th>
      <th>sex</th>
      <th>p1</th>
      <th>p2</th>
      <th>p3</th>
      <th>p4</th>
      <th>p5</th>
      <th>total</th>
      <th>fullname</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ARG</td>
      <td>Argentina</td>
      <td>1</td>
      <td>MASLIAH</td>
      <td>JULIÁN</td>
      <td>M</td>
      <td>7</td>
      <td>6</td>
      <td>1</td>
      <td>5</td>
      <td>0</td>
      <td>19</td>
      <td>JULIÁN MASLIAH</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ARG</td>
      <td>Argentina</td>
      <td>2</td>
      <td>FLESCHLER</td>
      <td>IAN</td>
      <td>M</td>
      <td>7</td>
      <td>7</td>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>18</td>
      <td>IAN FLESCHLER</td>
    </tr>
    <tr>
      <th>2</th>
      <td>ARG</td>
      <td>Argentina</td>
      <td>3</td>
      <td>SOTO</td>
      <td>CARLOS MIGUEL</td>
      <td>M</td>
      <td>7</td>
      <td>0</td>
      <td>4</td>
      <td>5</td>
      <td>0</td>
      <td>16</td>
      <td>CARLOS MIGUEL SOTO</td>
    </tr>
    <tr>
      <th>3</th>
      <td>ARG</td>
      <td>Argentina</td>
      <td>4</td>
      <td>DI SANZO</td>
      <td>BRUNO</td>
      <td>M</td>
      <td>7</td>
      <td>3</td>
      <td>0</td>
      <td>5</td>
      <td>0</td>
      <td>15</td>
      <td>BRUNO DI SANZO</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ARG</td>
      <td>Argentina</td>
      <td>5</td>
      <td>CASSIA</td>
      <td>NICOLÁS</td>
      <td>M</td>
      <td>7</td>
      <td>3</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>13</td>
      <td>NICOLÁS CASSIA</td>
    </tr>
  </tbody>
</table>
</div>




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
merged_df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>id</th>
      <th>value1</th>
      <th>value2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2</td>
      <td>B</td>
      <td>D</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3</td>
      <td>C</td>
      <td>E</td>
    </tr>
  </tbody>
</table>
</div>



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

    C:\Users\tuyen\AppData\Local\Temp\ipykernel_12788\2741901675.py:6: Pandas4Warning: For backward compatibility, 'str' dtypes are included by select_dtypes when 'object' dtype is specified. This behavior is deprecated and will be removed in a future version. Explicitly pass 'str' to `include` to select them, or to `exclude` to remove them and silence this warning.
    See https://pandas.pydata.org/docs/user_guide/migration-3-strings.html#string-migration-select-dtypes for details on how to write code that works with pandas 2 and 3.
      categorical_summary = df.describe(include=['object'])
    


```python
# Tóm tắt dữ liệu nhiều thông số 
stats = df.groupby('country').agg({
    'total': ['mean', 'max', 'min'],
    'rank': ['mean', 'max', 'min']
})
stats = stats.reset_index()
stats.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }

    .dataframe thead tr:last-of-type th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th colspan="3" halign="left">total</th>
      <th colspan="3" halign="left">rank</th>
    </tr>
    <tr>
      <th></th>
      <th>mean</th>
      <th>max</th>
      <th>min</th>
      <th>mean</th>
      <th>max</th>
      <th>min</th>
    </tr>
    <tr>
      <th>country</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Argentina</th>
      <td>12.2</td>
      <td>19</td>
      <td>7</td>
      <td>5.5</td>
      <td>10</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Australia</th>
      <td>16.5</td>
      <td>21</td>
      <td>12</td>
      <td>5.5</td>
      <td>10</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Bangladesh</th>
      <td>13.5</td>
      <td>21</td>
      <td>9</td>
      <td>5.5</td>
      <td>10</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Bolivia</th>
      <td>13.2</td>
      <td>21</td>
      <td>3</td>
      <td>3.0</td>
      <td>5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>Brazil</th>
      <td>14.8</td>
      <td>20</td>
      <td>12</td>
      <td>5.5</td>
      <td>10</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



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
