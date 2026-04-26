# Bài 9: Trực quan hóa Dữ liệu

Trực quan hóa là chìa khóa để hiểu và truyền đạt dữ liệu. Trong bài học này, bạn sẽ sử dụng Matplotlib để tạo các biểu đồ cơ bản cho dữ liệu bảng và không gian địa lý.  

## 9.1. Mục tiêu học tập
- Tạo biểu đồ đường, cột, heatmap, và phân tán
- Tùy chỉnh giao diện biểu đồ
- Trực quan hóa dữ liệu không gian địa lý


```python
import matplotlib.pyplot as plt # Thư viện để vẽ đồ thị
import numpy as np # Thư viện để làm việc với mảng và tính toán số học
import matplotlib.cm as cm # Thư viện để sử dụng các colormap
import matplotlib.colors as colors # Thư viện để chuẩn hóa màu sắc
```

## 9.2. Giới thiệu về Matplotlib

Matplotlib là thư viện vẽ biểu đồ được sử dụng rộng rãi nhất trong Python.

**Các thành phần chính của 1 figure**

**`Figure`**: Container tổng thể chứa toàn bộ các thành phần của biểu đồ, hoạt động như canvas để vẽ visualization.

- `Axes`: Vùng trong figure nơi dữ liệu được vẽ; mỗi figure có thể chứa nhiều axes (subplots).

- `Axis`: Đại diện cho trục x và y, xác định giới hạn, vị trí tick và labels để diễn giải dữ liệu.

- `Lines và Markers`: Lines nối các điểm dữ liệu để hiển thị xu hướng, markers đánh dấu từng điểm dữ liệu trong các biểu đồ như scatter plot.

- `Title và Labels`: Title cung cấp ngữ cảnh cho biểu đồ, axis labels mô tả dữ liệu được biểu diễn trên mỗi trục.

- `Legend`: Giải thích ý nghĩa của các màu sắc, đường line khác nhau trong biểu đồ.

- `Ticks và Tick Labels`: Các vạch chia trên trục và nhãn số/text tương ứng để đọc giá trị chính xác.

- `Grid`: Lưới hỗ trợ giúp đọc và so sánh các giá trị trong biểu đồ dễ dàng hơn.

![image.png](geocourse/images/matplotlib_overview.png)
Hình ảnh từ [geeksforgeeks](https://www.geeksforgeeks.org/python/python-introduction-matplotlib/)

## 9.3. Biểu đồ đường, cột, và phân tán

Biểu đồ đường và cột hữu ích để xem xu hướng và so sánh.

### 9.3.1. Biểu đồ cột


```python
# Biểu đồ cột dân số thành phố
cities = ['Hà Nội', 'TP.HCM', 'Đà Nẵng']
populations = [8053663, 9420000, 1134000] # Các bạn kiểm tra lại số liệu hiện tại để cập nhật chính xác
# Đầu tiên tạo figure và axes
fig, ax = plt.subplots(nrows=1, ncols=1, figsize=(10, 5))
# Vẽ biểu đồ cột với màu sắc và đường viền
ax.bar(cities, populations, edgecolor="black", color=["darkred", "darkblue", "darkgreen"])
# Thêm tiêu đề
ax.set_title('Dân số các thành phố Việt Nam', fontsize=12)
# Thêm nhãn trục y và điều chỉnh font chữ
ax.set_ylabel('Dân số', fontsize=12)
# Thêm nhãn trục x và điều chỉnh font chữ
ax.set_xticks(cities)
# Thêm nhãn trục x với góc nghiêng và điều chỉnh font chữ
ax.set_xticklabels(cities, rotation=45, fontsize=12)
# Điều chỉnh kích thước font chữ của nhãn trục y
ax.tick_params(axis='y', which='major', labelsize=12)
# Thêm lưới với kiểu đường nét và độ mờ
ax.grid(linestyle='--', alpha=0.7, color="gray")
plt.tight_layout()  # Điều chỉnh layout
plt.show()
```


    
![png](output_7_0.png)
    


### 9.3.2. Biểu đồ đường


```python
# Biểu đồ đường 
# Tạo dữ liệu ngẫu nhiên
x = np.random.rand(50)
y = np.random.rand(50)
# Tạo figure và axes
fig, ax = plt.subplots(nrows=1, ncols=1, figsize=(10, 5))
# Vẽ biểu đồ đường đầu tiên
ax.plot(x, marker='o', linestyle='-', color='b')
# Vẽ biểu đồ đường thứ hai
ax.plot(y, marker='s', linestyle='--', color='r')
ax.set_xlabel('Trục X', fontsize=12) # Thêm nhãn trục x
ax.set_ylabel('Trục Y', fontsize=12) # Thêm nhãn trục y
ax.tick_params(axis='both', which='major', labelsize=12) # Tùy chỉnh kích thước chữ trục
ax.legend(['Dữ liệu X', 'Dữ liệu Y']) # Thêm chú thích
ax.grid(linestyle='--', alpha=0.7, color="gray") # Thêm lưới
plt.show()
```


    
![png](output_9_0.png)
    


### 9.3.3. Biểu đồ phân tán

Biểu đồ phân tán rất tốt để trực quan hóa mối quan hệ giữa các biến.


```python
# Biểu đồ phân tán tọa độ thành phố
latitudes = [21.0285, 10.8231, 16.0544, 20.8449, 10.0452]
longitudes = [105.8542, 106.6297, 108.2022, 106.6881, 105.7469]
city_names = ['Hà Nội', 'TP.HCM', 'Đà Nẵng', 'Hải Phòng', 'Cần Thơ']
# Tạo figure và axes
fig, ax = plt.subplots(nrows=1, ncols=1, figsize=(8, 6))
# Vẽ biểu đồ phân tán với màu sắc và độ mờ
ax.scatter(longitudes, latitudes, s=100, c='red', alpha=0.7)
ax.set_title('Vị trí các thành phố Việt Nam') # Thêm tiêu đề
ax.set_xlabel('Kinh độ (°E)') # Thêm nhãn trục x
ax.set_ylabel('Vĩ độ (°N)') # Thêm nhãn trục y

# Thêm tên thành phố
for i, city in enumerate(city_names):
    ax.annotate(city, (longitudes[i], latitudes[i]), xytext=(5, 5), 
                textcoords='offset points', fontsize=9)
ax.grid(True, alpha=0.3) # Thêm lưới
plt.show()
```


    
![png](output_11_0.png)
    


## 9.4. Kết hợp nhiều biều đồ 


```python
import pandas as pd
df = pd.read_csv('https://raw.githubusercontent.com/mwaskom/seaborn-data/master/iris.csv')
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
      <th>sepal_length</th>
      <th>sepal_width</th>
      <th>petal_length</th>
      <th>petal_width</th>
      <th>species</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>5.1</td>
      <td>3.5</td>
      <td>1.4</td>
      <td>0.2</td>
      <td>setosa</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4.9</td>
      <td>3.0</td>
      <td>1.4</td>
      <td>0.2</td>
      <td>setosa</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4.7</td>
      <td>3.2</td>
      <td>1.3</td>
      <td>0.2</td>
      <td>setosa</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4.6</td>
      <td>3.1</td>
      <td>1.5</td>
      <td>0.2</td>
      <td>setosa</td>
    </tr>
    <tr>
      <th>4</th>
      <td>5.0</td>
      <td>3.6</td>
      <td>1.4</td>
      <td>0.2</td>
      <td>setosa</td>
    </tr>
  </tbody>
</table>
</div>



### 9.4.1. Biểu đồ boxplot và cột trong cùng một figure


```python
# Hiển thị nhiều biểu đồ trong cùng một figure
# Tạo figure và axes với 1 hàng và 2 cột
fig, axes = plt.subplots(nrows=1, ncols=2, figsize=(12, 5)) # Tạo ra figure và 2 axis
note_list = list("ab") # Danh sách ghi chú cho từng biểu đồ
for i, ax in enumerate(axes.flatten()): # Duyệt qua từng axis
    if i ==0: # Vẽ biểu đồ hộp cho chiều dài đài hoa
        # Biểu đồ hộp (box plot)
        x = [df[df['species'] == species]['sepal_length'] for species in df['species'].unique()]
        y = df['species'].unique()
        ax.boxplot(x, tick_labels=y) # Vẽ boxplot với nhãn trục x là tên loài hoa
        ax.set_xlabel('Loài hoa', fontsize=12) # Thêm nhãn trục x
        ax.set_ylabel('Chiều dài đài hoa (cm)', fontsize=12) # Thêm nhãn trục y
        ax.tick_params(axis='both', which='major', labelsize=12) # Tùy chỉnh kích thước chữ trục
        ax.text(0.07, 0.9, f'({note_list[i]})', transform=ax.transAxes, 
                fontsize=12) # Thêm ghi chú vào góc trên bên trái của biểu đồ
        
    else:
        # Biểu đồ histogram
        ax.hist(df['sepal_width'], bins=15, color='skyblue', edgecolor='black')
        ax.set_xlabel('Chiều rộng đài hoa (cm)', fontsize=12) # Thêm nhãn trục x
        ax.set_ylabel('Tần suất', fontsize=12) # Thêm nhãn trục y
        ax.tick_params(axis='both', which='major', labelsize=12) # Tùy chỉnh kích thước chữ trục
        ax.text(0.07, 0.9, f'({note_list[i]})', transform=ax.transAxes, 
                fontsize=12) # Thêm ghi chú vào góc trên bên trái của biểu đồ
```


    
![png](output_15_0.png)
    


### 9.4.2. Biểu đồ phân tán và biểu đồ histogram


```python
# Biểu đồ phân tán với màu sắc theo loài hoa cho nhiều biến
# Tạo figure và axes với 1 hàng và 2 cột
fig, axes = plt.subplots(nrows=1, ncols=2, figsize=(12, 5), dpi=500)

species = df['species'].unique() # Lấy danh sách các loài hoa duy nhất
colors_list = ['red', 'blue', 'green'] # Màu sắc tương ứng cho từng loài hoa
for i, ax in enumerate(axes.flatten()): # Duyệt qua từng axis
    if i==0:
        for j, specie in enumerate(species):
            subset = df[df['species'] == specie]
            ax.scatter(subset['sepal_length'], subset['sepal_width'], 
                       color=colors_list[j], label=specie, alpha=0.7)
        ax.set_xlabel('Chiều dài đài hoa (cm)', fontsize=12)
        ax.set_ylabel('Chiều rộng đài hoa (cm)', fontsize=12) # Thêm nhãn trục y
        ax.legend(title='Loài hoa') # Thêm chú thích
        ax.tick_params(axis='both', which='major', labelsize=12) # Tùy chỉnh kích thước chữ trục
        ax.text(0.07, 0.9, '(a)', transform=ax.transAxes, fontsize=12) # Thêm ghi chú vào góc trên bên trái của biểu đồ
    else:
        # Biểu đồ histogram 
        ax.hist(df['sepal_width'], bins=15, color='lightgreen', edgecolor='black')
        ax.set_xlabel('Chiều rộng đài hoa (cm)', fontsize=12) # Thêm nhãn trục x
        ax.set_ylabel('Tần suất', fontsize=12) # Thêm nhãn trục y
        ax.tick_params(axis='both', which='major', labelsize=12) # Tùy chỉnh kích thước chữ trục
        ax.text(0.07, 0.9, '(b)', transform=ax.transAxes, fontsize=12) # Thêm ghi chú vào góc trên bên trái của biểu đồ
```


    
![png](output_17_0.png)
    


### 9.4.3. Sắp xếp điều chính kích thước của từng biểu đồ


```python
# Tạo 3 biểu đồ với 2 biểu đồ to và 1 biểu đồ nhỏ với dữ liệu ngẫu nhiên. Bạn có thể thay thế dữ liệu này bằng dữ liệu thực tế của bạn.
fig = plt.figure(figsize=(12, 8))
ax1 = plt.subplot2grid((2, 2), (0, 0), colspan=2)  # Biểu đồ to trên cùng
ax2 = plt.subplot2grid((2, 2), (1, 0))  # Biểu đồ nhỏ bên dưới bên trái
ax3 = plt.subplot2grid((2, 2), (1, 1))  # Biểu đồ nhỏ bên dưới bên phải
# Điều chỉnh khoảng cách giữa các biểu đồ
fig.subplots_adjust(hspace=0.4, wspace=0.3)
note_list = list("abc") # Danh sách ghi chú cho từng biểu đồ

for i, ax in enumerate([ax1, ax2, ax3]): # Duyệt qua từng axis
    if i == 0: # Vẽ biểu đồ thứ nhất 
        # Biểu đồ đường
        x = np.linspace(0, 10, 100)
        y = np.sin(x)
        ax.plot(x, y, color='purple')
        ax.set_xlabel('X Label', fontsize=12)
        ax.set_ylabel('Y Label', fontsize=12)
        ax.tick_params(axis='both', which='major', labelsize=12)
        ax.text(0.04, 0.9, f'({note_list[i]})', transform=ax.transAxes, 
                fontsize=12)
    elif i == 1: # Vẽ biểu đồ thứ hai
        # Biểu đồ cột
        categories = ['A', 'B', 'C', 'D']
        values = [23, 45, 56, 78]
        ax.bar(categories, values, color='orange', edgecolor='black')
        ax.set_xlabel('Danh mục', fontsize=12)
        ax.set_ylabel('Giá trị', fontsize=12)
        ax.tick_params(axis='both', which='major', labelsize=12)
        ax.text(0.07, 0.9, f'({note_list[i]})', transform=ax.transAxes, 
                fontsize=12)
    else: # Vẽ biểu đồ thứ ba
        # Biểu đồ phân tán
        x = np.random.rand(50)
        y = np.random.rand(50)
        ax.scatter(x, y, color='green', alpha=0.6)
        ax.set_xlabel('X-axis', fontsize=12)
        ax.set_ylabel('Y-axis', fontsize=12)
        ax.tick_params(axis='both', which='major', labelsize=12)
        ax.text(0.07, 0.9, f'({note_list[i]})', transform=ax.transAxes, 
                fontsize=12)
```


    
![png](output_19_0.png)
    


### 9.5. Biểu đồ heatmap và colorbar line plots

### 9.5.1. Biểu đồ heatmap


Biểu đồ heatmap về cơ bản là 1 bức ảnh. Do vậy, chúng ta phải đưa dữ liệu về dạng numpy 2d.


```python
df = pd.read_csv("https://raw.githubusercontent.com/thedownlohd/Measles/refs/heads/master/data/raw/measles_cases_USA_by_state_1928_to_2002.csv")
# Lựa chọn các cột cần thiết
df = df[['year', 'state', 'disease', 'cases']]
# Bỏ qua dữ liệu trống 
df = df.dropna()
# Giữ lại dữ liệu bệnh sởi
df = df[df['disease'] == 'MEASLES']
# Lựa chọn 10 bang có tổng số ca bệnh cao nhất
top_states = df.groupby('state')['cases'].sum().nlargest(10).index
top_states_data = df[df['state'].isin(top_states)]
```


```python
# Chuyển dữ liệu về dạng ma trận cho biểu đồ heatmap
heat = top_states_data.pivot_table(index='state', columns='year', values='cases', aggfunc='sum', fill_value=0)
vmin, vmax =  np.nanpercentile(heat.values, [1, 98])
# heat = np.clip(heat.values, vmin, vmax)
# Tạo figure và axes cho biểu đồ heatmap
fig, ax = plt.subplots(figsize=(12, 5.5))
# Vẽ biểu đồ heatmap với colormap và giới hạn giá trị
cax = ax.imshow(heat, aspect='auto', cmap='viridis',vmin=vmin, vmax=vmax)
# set xtick labels with 10 equal intervals
ax.set_xticks(np.linspace(0, len(heat.columns) - 1, 10)) # Thiết lập vị trí của các nhãn trục x với 10 khoảng cách đều nhau
ax.set_xticklabels(heat.columns[np.linspace(0, len(heat.columns) - 1, 10).astype(int)]) # Thiết lập nhãn trục x là các năm tương ứng với 10 khoảng cách đều nhau
ax.set_yticks(np.arange(len(heat.index))) # Thiết lập vị trí của các nhãn trục y
ax.set_yticklabels(heat.index) # Thiết lập nhãn trục y là tên các bang
ax.tick_params(axis='both', which='major', labelsize=12) # Tùy chỉnh kích thước chữ trục
ax.set_ylabel('Bang', fontsize=12) # Thêm nhãn cho trục y
# Thêm colorbar với nhãn và điều chỉnh kích thước chữ
cbar = fig.colorbar(cax, ax=ax, label='Số ca bệnh sởi', orientation='horizontal', pad=0.1, shrink=0.6,
             extend="both", ticks=np.linspace(vmin, vmax, num=7))
# Kích thước chữ nhãn colorbar
cbar.ax.tick_params(labelsize=12)
cbar.set_label('Số ca bệnh sởi', fontsize=12) # Thêm nhãn cho colorbar
plt.show()
```


    
![png](output_23_0.png)
    


### 9.5.2. Biểu đồ line plots with colorbars

Sử dụng khi cần vẽ nhiều biểu đồ đường với mỗi màu sắc cho một đường.


```python
# Tạo dữ liệu mẫu ngẫu nhiên
n_lines = 50
n_points = 100
x = np.linspace(400, 2500, n_points)
# Đường phản xạ (ngẫu nhiên)
y = 0.05 + 0.4 * np.exp(-((x[:, None] - 1000) / 800)**2) + 0.05 * np.random.rand(n_points, n_lines)

# Hàm lượng nước trong đất (%), một giá trị cho mỗi đường. Chỉ là ví dụ minh họa.
moisture = np.linspace(0, 25, n_lines)
# Chọn colormap và chuẩn hóa
cmap = cm.jet  # or 'viridis', 'plasma', etc.
norm = colors.Normalize(vmin=moisture.min(), vmax=moisture.max())
# Tạo figure và axes  
fig, ax = plt.subplots(figsize=(10, 5))

for i in range(n_lines): # Vẽ từng đường với màu sắc tương ứng với hàm lượng nước trong đất
    ax.plot(x, y[:, i], color=cmap(norm(moisture[i])))
# Thêm colorbar
sm = cm.ScalarMappable(cmap=cmap, norm=norm) # Tạo một ScalarMappable để tạo colorbar
sm.set_array([])  # only needed for colorbar
# Thêm colorbar với nhãn và điều chỉnh kích thước chữ
cbar = fig.colorbar(sm, ax=ax, extend="both", pad=0.02) 
cbar.set_label("Soil Moisture Content (%)", fontsize=12) 
ax.set_xlabel("Wavelength (nm)", fontsize=12)
ax.set_ylabel("Reflectance", fontsize=12) # Thêm nhãn cho trục y
ax.tick_params(axis='both', which='major', labelsize=12) # Tùy chỉnh kích thước chữ trục
ax.grid(linestyle='--', alpha=0.5, color="gray") # Thêm lưới với kiểu đường nét đứt và độ mờ
plt.show()
```


    
![png](output_25_0.png)
    


## Tóm tắt

Bạn đã hoàn thành Bài 9 và học được các kỹ năng trực quan hóa dữ liệu quan trọng.

### Các khái niệm chính đã nắm vững:
- ✅ **Matplotlib cơ bản**: Tạo và tùy chỉnh biểu đồ
- ✅ **Biểu đồ đường**: Hiển thị xu hướng theo thời gian (nhiệt độ hàng tháng)
- ✅ **Biểu đồ cột**: So sánh giá trị giữa các danh mục (dân số, lượng mưa)
- ✅ **Biểu đồ phân tán**: Visualize mối quan hệ và phân bố không gian
- ✅ **Heatmaps**: Trực quan hóa dữ liệu lưới và mật độ
- ✅ **Tùy chỉnh biểu đồ**: Colors, labels, annotations, layouts

### Kỹ năng bạn có thể áp dụng:
- Tạo biểu đồ chuyên nghiệp để trình bày dữ liệu
- Trực quan hóa dữ liệu khí hậu và địa lý Việt Nam
- Phân tích xu hướng thời gian và phân bố không gian
- Tùy chỉnh biểu đồ cho presentation và báo cáo
- Sử dụng màu sắc và layout hiệu quả

