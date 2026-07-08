# Bài 7: Cơ bản về NumPy

NumPy là nền tảng của tính toán số trong Python. Trong phân tích không gian địa lý, nó được sử dụng cho các phép toán mảng hiệu quả, dữ liệu raster và tính toán toán học.

> **Lưu Ý**: Bạn có thể chạy trực tiếp notebook này bằng **Google Colab** thông qua [liên kết này](https://colab.research.google.com/drive/1J2vFtvV5vV7kjuWnC03HQqiDdltyeBxz) mà không cần cài đặt Python. Trong trường hợp bạn muốn tạo bản sao của notebook này, bạn có thể làm như sau: `File → Save a copy in Drive`.

## 7.1. Mục tiêu học tập
- Hiểu về NumPy arrays và ưu điểm so với lists
- Tạo, lập chỉ mục và thao tác arrays
- Thực hiện các phép toán toán học trên arrays
- Áp dụng NumPy cho các bài toán dữ liệu không gian địa lý


```python
import numpy as np # Nếu bạn chưa cài đặt numpy, hãy chạy: pip install numpy hoặc xem lại Bài 1.
```

## 7.2. Giới thiệu về NumPy

NumPy cung cấp ndarray (mảng N-chiều) cho các phép toán số nhanh chóng và hiệu quả.

### 7.2.1. Tạo array từ list

Cách đơn giản nhất để tạo NumPy array là chuyển đổi từ Python list bằng hàm `np.array()`. Điều này giúp chúng ta dễ dàng làm việc với dữ liệu có sẵn dưới dạng list và tận dụng được các phép toán hiệu quả của NumPy.


```python
# Tạo NumPy array từ một list
data = [1, 2, 3, 4, 5]
arr = np.array(data)
print('Array:', arr)
print('Kiểu dữ liệu:', type(arr))
```

    Array: [1 2 3 4 5]
    Kiểu dữ liệu: <class 'numpy.ndarray'>
    

### 7.2.2. Tạo arrays bằng các hàm của numpy

NumPy có thể tạo arrays từ lists, hoặc tạo arrays với các giá trị cụ thể. Ghi chú, mảng numpy array chỉ chứa 1 kiểu loại dữ liệu cho toàn mảng. Ví dụ là số thực thì toàn mảng là số thực.

- **Tạo mảng toàn số 0 với hàm `np.zeros`**

Hàm `np.zeros()` tạo một mảng mới với tất cả các phần tử có giá trị 0. Bạn cần chỉ định kích thước mảng mong muốn thông qua một tuple. Điều này rất hữu ích khi khởi tạo mảng để lưu trữ kết quả tính toán.


```python
# Các phương pháp tạo mảng phổ biến
zeros = np.zeros((3, 3)) # Mảng 3x3 toàn số 0
print('Mảng zeros:\n', zeros)
```

    Mảng zeros:
     [[0. 0. 0.]
     [0. 0. 0.]
     [0. 0. 0.]]
    

- **Tạo mảng toàn số 0 với hàm `np.zeros_like`**

Hàm `np.zeros_like()` tạo một mảng mới với cùng kích thước và kiểu dữ liệu với mảng đầu vào, nhưng tất cả giá trị đều là 0. Điều này tiện lợi khi bạn muốn tạo một mảng kết quả có cùng cấu trúc với mảng gốc.


```python
zero_like = np.zeros_like(zeros) # Mảng cùng kích thước với zeros nhưng toàn số 0
print('Mảng zeros_like:\n', zero_like)
```

    Mảng zeros_like:
     [[0. 0. 0.]
     [0. 0. 0.]
     [0. 0. 0.]]
    

- **Tạo mảng toàn số 1 với hàm `np.ones`**

Hàm `np.ones()` hoạt động tương tự như `np.zeros()` nhưng tạo mảng với tất cả các phần tử có giá trị 1. Hàm này thường được sử dụng để khởi tạo ma trận đơn vị hoặc tạo mask cho các phép toán.


```python
ones = np.ones((2, 4)) # Mảng 2x4 toàn số 1
print('Mảng ones:\n', ones)
```

    Mảng ones:
     [[1. 1. 1. 1.]
     [1. 1. 1. 1.]]
    

- **Tạo mảng toàn số 1 với hàm `np.ones_like`**

Hàm `np.ones_like()` tạo một mảng với cùng kích thước và kiểu dữ liệu với mảng đầu vào, nhưng tất cả giá trị đều là 1. Đây là cách nhanh chóng để khởi tạo một mảng có cùng cấu trúc với dữ liệu gốc.


```python
one_like = np.ones_like(zeros) # Mảng cùng kích thước với zeros nhưng toàn số 1
print('Mảng ones_like:\n', one_like)
```

    Mảng ones_like:
     [[1. 1. 1.]
     [1. 1. 1.]
     [1. 1. 1.]]
    

- **Tạo mảng toàn một số cụ thể với hàm `np.full`**

Hàm `np.full()` cho phép bạn tạo một mảng với tất cả các phần tử có cùng một giá trị tùy chỉnh. Bạn chỉ định kích thước mảng và giá trị muốn điền vào. Điều này hữu ích khi khởi tạo mảng với một giá trị mặc định cụ thể.


```python
full = np.full((2, 2), 7) # Mảng 2x2 toàn số 7
print('Mảng full:\n', full)
```

    Mảng full:
     [[7 7]
     [7 7]]
    

- **Thay đổi `shape` của mảng**

Phương thức `reshape()` cho phép thay đổi hình dạng của mảng mà không thay đổi dữ liệu. Điều quan trọng là tổng số phần tử phải giữ nguyên. Sử dụng `-1` trong reshape sẽ tự động tính toán kích thước cho chiều đó. 


```python
# Reshape dữ liệu mảng từ 2D sang 1D và ngược lại
array_2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
print('Mảng 2D:\n', array_2d)
array_1d = array_2d.reshape(-1)  # Chuyển từ 2D sang 1D
print('Mảng 1D:', array_1d)
```

    Mảng 2D:
     [[1 2 3]
     [4 5 6]
     [7 8 9]]
    Mảng 1D: [1 2 3 4 5 6 7 8 9]
    


```python
# Reshape array sau 
array_2d = np.arange(12).reshape(3, 4)  # Tạo mảng 2D 3x4 từ 1D
print('Mảng 2D sau reshape:\n', array_2d)
```

    Mảng 2D sau reshape:
     [[ 0  1  2  3]
     [ 4  5  6  7]
     [ 8  9 10 11]]
    

- **Concatenate hai hoặc nhiều mảng**

Hàm `np.concatenate()` kết hợp nhiều mảng thành một mảng duy nhất theo một trục cụ thể. Tham số `axis` xác định chiều kết hợp: `axis=0` kết hợp theo hàng (dọc), `axis=1` kết hợp theo cột (ngang). Các mảng phải có cùng kích thước ở các chiều khác.


```python
# Reshape array sau 
a = np.arange(12).reshape(3, 4)  # Tạo mảng 2D 3x4 từ 1D
print('Mảng a:\n', a)
# Merge (kết hợp) hai mảng
b = np.array([[10, 11, 12, 13], [14, 15, 16, 17], [18, 19, 20, 21]])
print('Mảng b:\n', b)
merged_array = np.concatenate((a, b), axis=0)  # Kết hợp theo chiều dọc (theo hàng)
print('Mảng sau khi merge:\n', merged_array)
```

    Mảng a:
     [[ 0  1  2  3]
     [ 4  5  6  7]
     [ 8  9 10 11]]
    Mảng b:
     [[10 11 12 13]
     [14 15 16 17]
     [18 19 20 21]]
    Mảng sau khi merge:
     [[ 0  1  2  3]
     [ 4  5  6  7]
     [ 8  9 10 11]
     [10 11 12 13]
     [14 15 16 17]
     [18 19 20 21]]
    

- **Tách mảng thành 2 phần**

Hàm `np.array_split()` chia một mảng thành nhiều mảng con theo số phần chỉ định. Tham số `axis` xác định chiều tách. Nếu không thể chia đều, các phần sẽ có kích thước gần bằng nhau. Điều này hữu ích khi chia dữ liệu thành các batch hoặc fold cho cross-validation.


```python
# Split (tách) mảng thành hai phần
split_arrays = np.array_split(merged_array, 2, axis=0)  # Tách mảng thành 2 phần dọc theo axis 0
print('Mảng sau khi split:')
for i, arr in enumerate(split_arrays):
    print(f'Phần {i+1}:\n', arr)
```

    Mảng sau khi split:
    Phần 1:
     [[ 0  1  2  3]
     [ 4  5  6  7]
     [ 8  9 10 11]]
    Phần 2:
     [[10 11 12 13]
     [14 15 16 17]
     [18 19 20 21]]
    

- **Lọc giá trị với `np.where`**

Hàm `np.where()` trả về các chỉ số của các phần tử thỏa mãn điều kiện. Kết hợp với indexing, bạn có thể lọc và lấy ra các giá trị cần thiết từ mảng. Đây là công cụ mạnh mẽ để thao tác dữ liệu dựa trên điều kiện logic.


```python
# sử dụng where để lọc các giá trị trong mảng
array = np.array([10, 15, 20, 25, 30, 35])
filtered_array = array[np.where(array > 20)]  # Lọc các giá trị lớn hơn 20
print('Mảng sau khi lọc:', filtered_array)
```

    Mảng sau khi lọc: [25 30 35]
    

- **Tạo mảng ngẫu nhiên với `np.random`**

Module `np.random` cung cấp nhiều hàm để tạo mảng với các giá trị ngẫu nhiên. `randint()` tạo số nguyên ngẫu nhiên trong một khoảng, `rand()` tạo số thực từ 0 đến 1, `randn()` tạo số theo phân phối chuẩn. Điều này hữu ích cho việc khởi tạo dữ liệu test hoặc simulation.


```python
# doi kieu du lieu cua array
array = np.random.randint(-10, 10, size=(3, 3)) # Tạo mảng 3x3 với các số nguyên ngẫu nhiên từ -10 đến 9
print('Mảng ban đầu:\n', array)
```

    Mảng ban đầu:
     [[ 4  3  4]
     [ 4  9  0]
     [ 7  2 -8]]
    

- **Chuyển đổi kiểu dữ liệu cho mảng**

Phương thức `astype()` cho phép chuyển đổi kiểu dữ liệu của mảng sang kiểu khác như int, float, bool. Việc chọn đúng kiểu dữ liệu giúp tiết kiệm bộ nhớ và tăng hiệu suất tính toán.


```python
array = np.random.randint(-10, 10, size=(3, 3))
print('Mảng sau khi tạo mới với kiểu dữ liệu:', array.dtype, '\n', array)
array_float = array.astype(float) # Chuyển đổi kiểu dữ liệu sang float
print('Mảng sau khi đổi kiểu dữ liệu sang float:\n', array_float)
```

    Mảng sau khi tạo mới với kiểu dữ liệu: int32 
     [[-7 -1 -9]
     [ 5  1 -3]
     [ 0  7 -8]]
    Mảng sau khi đổi kiểu dữ liệu sang float:
     [[-7. -1. -9.]
     [ 5.  1. -3.]
     [ 0.  7. -8.]]
    

- **Kiểm tra thuộc tính liên quan đến mảng**

NumPy arrays có nhiều thuộc tính hữu ích: `shape` cho biết kích thước mỗi chiều, `ndim` cho biết số chiều, `size` cho biết tổng số phần tử, và `dtype` cho biết kiểu dữ liệu. Hiểu rõ các thuộc tính này giúp bạn debug và xử lý dữ liệu hiệu quả hơn.


```python
array = np.array([10, 15, 20, 25, 30, 35])
# kiem tra kich thuoc, hinh dang, so chieu, va kieu du lieu cua array
print('Kích thước (shape):', array.shape)
print('Số chiều (ndim):', array.ndim)
print('Kích thước (size):', array.size)
print('Kiểu dữ liệu (dtype):', array.dtype)
```

    Kích thước (shape): (6,)
    Số chiều (ndim): 1
    Kích thước (size): 6
    Kiểu dữ liệu (dtype): int64
    

## 7.3. Lập chỉ mục và cắt lát Array

Lập chỉ mục và cắt lát (indexing and slicing) hoạt động tương tự như lists, nhưng mạnh mẽ hơn cho dữ liệu đa chiều.

- **Lập chỉ mục**

Lập chỉ mục (indexing) cho phép truy cập các phần tử cụ thể trong mảng. Đối với mảng đa chiều, sử dụng dấu phẩy để phân tách các chỉ số cho mỗi chiều. Chỉ số bắt đầu từ 0 và có thể sử dụng chỉ số âm để đếm từ cuối mảng.


```python
arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]]) # Mảng 2D
print('Mảng 2D:\n', arr2d)
print('Phần tử tại hàng 0, cột 1:', arr2d[0, 1])
```

    Mảng 2D:
     [[1 2 3]
     [4 5 6]
     [7 8 9]]
    Phần tử tại hàng 0, cột 1: 2
    

- **Cắt lát**

Cắt lát theo quy tắc sau `array[start:stop:step]` 


```python
# Khởi tạo mảng 2D
arr2d = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
print('Mảng 2D:\n', arr2d)
print('Tất cả phần tử của cột 1:', arr2d[:, 1])
print('Tất cả phần tử của hàng 0:', arr2d[0, :])
print('Lấy phần tử tại hàng 1 với bước nhảy 2:', arr2d[1, ::2])
```

    Mảng 2D:
     [[1 2 3]
     [4 5 6]
     [7 8 9]]
    Tất cả phần tử của cột 1: [2 5 8]
    Tất cả phần tử của hàng 0: [1 2 3]
    Lấy phần tử tại hàng 1 với bước nhảy 2: [4 6]
    

- **Thay thế cập nhật giá trị trong mảng**

Bạn có thể gán giá trị mới cho một phần tử hoặc một nhóm phần tử trong mảng bằng cách sử dụng indexing hoặc slicing. Điều này cho phép cập nhật dữ liệu một cách linh hoạt, từ thay đổi một giá trị đơn lẻ đến cập nhật toàn bộ một hàng hay cột.


```python
# Thay thế giá trị trong mảng
arr2d[1, 1] = 99
print('Mảng sau khi thay thế:\n', arr2d)
# Thay thế cả một cột
arr2d[:, 2] = [7, 8, 9]
print('Mảng sau khi thay thế cột 2:\n', arr2d)
```

    Mảng sau khi thay thế:
     [[ 1  2  3]
     [ 4 99  6]
     [ 7  8  9]]
    Mảng sau khi thay thế cột 2:
     [[ 1  2  7]
     [ 4 99  8]
     [ 7  8  9]]
    

## 7.4. Các phép toán Array

NumPy hỗ trợ các phép toán theo từng phần tử nhanh chóng và broadcasting.

- **Phép toán cơ bản**

NumPy hỗ trợ các phép toán số học cơ bản (+, -, *, /) được thực hiện theo từng phần tử (element-wise). Ngoài ra còn có các hàm thống kê như `mean()`, `sum()`, `max()`, `min()`, `std()` để tính toán nhanh chóng trên toàn bộ mảng hoặc theo từng trục.


```python
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
print('Phép cộng:', a + b)
print('Phép nhân:', a * b)
print('Giá trị trung bình:', np.mean(a))
print('Tổng:', np.sum(b))
```

    Phép cộng: [5 7 9]
    Phép nhân: [ 4 10 18]
    Giá trị trung bình: 2.0
    Tổng: 15
    


```python
# Ví dụ: Lưới độ cao
elevation = np.array([[100, 110, 120], [90, 95, 105], [80, 85, 90]])
print('Lưới độ cao:\n', elevation)
print('Độ cao lớn nhất:', np.max(elevation))
print('Độ cao trung bình:', np.mean(elevation))
```

    Lưới độ cao:
     [[100 110 120]
     [ 90  95 105]
     [ 80  85  90]]
    Độ cao lớn nhất: 120
    Độ cao trung bình: 97.22222222222223
    


```python
# Tạo ra array nhiều chiều và thao tác với nó
array = np.array([[[1, 2], [3, 4]], [[5, 6], [7, 8]]]) # Mảng 3D
mean = np.mean(array, axis=0) # Trung bình theo trục 0
std = np.std(array, axis=1)   # Độ lệch chuẩn theo trục 1
median = np.nanmedian(array, axis=2) # Trung vị theo trục 2
sum = np.nansum(array)        # Tổng bỏ qua NaN
percentile = np.percentile(array, 50) # Phần trăm vị trí 50 (trung vị)
print('Trung bình theo trục 0:\n', mean)
```

    Trung bình theo trục 0:
     [[3. 4.]
     [5. 6.]]
    

- **Phép toán tích vô hướng giữa 2 vectors**

Tích vô hướng (dot product) của hai vectors là tổng của tích các phần tử tương ứng. Trong NumPy, sử dụng `np.dot()` để tính tích vô hướng. Phép toán này rất quan trọng trong đại số tuyến tính và machine learning, đo lường mức độ tương đồng giữa hai vectors.


```python
# Tính tích vô hướng 2 vector a và b
a = np.array([1, 2, 3])
b = np.array([4, 5, 6])
dot_product = np.dot(a, b)
print('Tích vô hướng của a và b:', dot_product)
```

    Tích vô hướng của a và b: 32
    

- **Phép toán tích vô hướng giữa ma trận với vector**

Khi nhân ma trận với vector, mỗi hàng của ma trận được nhân tích vô hướng với vector, tạo ra một vector kết quả. Điều kiện là số cột của ma trận phải bằng số phần tử của vector. Phép toán này được sử dụng rộng rãi trong các mô hình hồi quy tuyến tính và neural networks.


```python
# Tích vô hướng giữa một ma trận A và một vector x. Điều kiện là số cột của A phải bằng số phần tử của x.
# Kết quả sẽ là một vector mới có số phần tử bằng số hàng của A.
A = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]]) # Ma trận 3x3
x = np.array([1, 0, -1]) # Vector 3 phần tử
dot_product_matrix_vector = np.dot(A, x)
print('Tích vô hướng giữa ma trận A và vector x:\n', dot_product_matrix_vector)
```

    Tích vô hướng giữa ma trận A và vector x:
     [-2 -2 -2]
    

- **Phép toán tích vô hướng giữa vector với ma trận**

Nhân vector với ma trận tương tự như nhân ma trận với vector nhưng theo thứ tự ngược lại. Vector được nhân với từng cột của ma trận. Điều kiện là số phần tử của vector phải bằng số hàng của ma trận. Kết quả là một vector có số phần tử bằng số cột của ma trận.


```python
# Tích vô hướng giữa 1 vector a và một ma trận B. Điều kiện là số phần tử của a phải bằng số hàng của B.
# Kết quả sẽ là một vector mới có số phần tử bằng số cột của B
a = np.array([1, 2, 3]) # Vector 3 phần tử
B = np.array([[4, 5, 6], [7, 8, 9], [10, 11, 12]]) # Ma trận 3x3
dot_product_vector_matrix = np.dot(a, B)    
print('Tích vô hướng giữa vector a và ma trận B:\n', dot_product_vector_matrix)
```

    Tích vô hướng giữa vector a và ma trận B:
     [48 54 60]
    

- **Phép toán nhân 2 ma trận**

Phép nhân ma trận (matrix multiplication) là phép toán cơ bản trong đại số tuyến tính. Phần tử ở hàng i, cột j của ma trận kết quả là tích vô hướng của hàng i của ma trận thứ nhất với cột j của ma trận thứ hai. Điều kiện là số cột của ma trận đầu phải bằng số hàng của ma trận sau.


```python
# Tích vô hướng giữa hai ma trận A và B. Điều kiện là số cột của A phải bằng số hàng của B.
# Kết quả sẽ là một ma trận mới có số hàng bằng số hàng của A và số cột bằng số cột của B
A = np.array([[1, 2], [3, 4], [5, 6]]) # Ma trận 3x2
B = np.array([[7, 8, 9], [10, 11, 12]]) # Ma trận 2x3
dot_product_matrix_matrix = np.dot(A, B)
print('Tích vô hướng giữa ma trận A và ma trận B:\n', dot_product_matrix_matrix)
```

    Tích vô hướng giữa ma trận A và ma trận B:
     [[ 27  30  33]
     [ 61  68  75]
     [ 95 106 117]]
    

## 7.5. Ứng dụng numpy vào địa không gian

### 7.5.1. Ứng dụng numpy vectorize

Hàm `vectorize` là một công cụ mạnh mẽ trong NumPy cho phép bạn áp dụng một hàm Python thông thường lên từng phần tử của một mảng mà không cần phải viết vòng lặp thủ công. Điều này giúp code của bạn trở nên ngắn gọn và hiệu quả hơn, đồng thời tận dụng được sức mạnh của các phép toán mảng trong NumPy để xử lý dữ liệu nhanh hơn.

- **Nhóm loại đất với mảng 2 chiều**

Ví dụ bạn có một mảng 2D (bản đồ) mô phỏng dữ liệu bản đồ đất đai với các giá trị từ 1 đến 10, bạn muốn phân loại lại các giá trị này thành 3 nhóm: Nhóm 1 (1-3), Nhóm 2 (4-7), và Nhóm 3 (8-10). Bạn có thể sử dụng hàm np.vectorize để áp dụng hàm phân loại lại cho từng phần tử trong mảng thành một trong ba nhóm như bên dưới.


```python
# tạo mảng 2d array mô phỏng dữ liệu bản đồ đất đai với kích thước 200x200 và các giá trị từ 1 đến 10
land_use = np.random.randint(1, 11, size=(200, 200))
# Ví dụ ta muốn nhóm các loại đất thành 3 nhóm: Nhóm 1 (1-3), Nhóm 2 (4-7), Nhóm 3 (8-10)
def reclass_landuse(value):
    """Hàm phân loại lại giá trị đất đai thành 3 nhóm."""
    if 1 <= value <= 3:
        return 1
    elif 4 <= value <= 7:
        return 2
    else:
        return 3
new_land_use = np.vectorize(reclass_landuse)(land_use)
print(f"Shpae của mảng sau khi phân loại lại: {new_land_use.shape}")
```

    Shpae của mảng sau khi phân loại lại: (200, 200)
    

- **Nhóm loại đất với mảng 3 chiều**

Tương tự như vậy, bạn có thể áp dụng hàm np.vectorize để phân loại lại dữ liệu bản đồ đất đai cho mảng 3D (ví dụ như bản đồ đất theo năm), giúp bạn dễ dàng xử lý và phân tích dữ liệu theo thời gian hoặc theo các lớp khác nhau.


```python
# tạo mảng 3d array mô phỏng dữ liệu bản đồ đất đai với kích thước 200x200 và các giá trị từ 1 đến 10 from 2000 to 2020
land_use = np.random.randint(1, 11, size=(20, 200, 200))
# Ví dụ ta muốn nhóm các loại đất thành 3 nhóm: Nhóm 1 (1-3), Nhóm 2 (4-7), Nhóm 3 (8-10)
def reclass_landuse(value):
    """Hàm phân loại lại giá trị đất đai thành 3 nhóm."""
    if 1 <= value <= 3:
        return 1
    elif 4 <= value <= 7:
        return 2
    else:
        return 3
new_land_use = np.vectorize(reclass_landuse)(land_use)
print(f"Shpae của mảng sau khi phân loại lại: {new_land_use.shape}")
```

    Shpae của mảng sau khi phân loại lại: (20, 200, 200)
    

### 7.5.2. Tính giá trị theo trục

Tính toán theo trục trong NumPy cho phép bạn thực hiện các phép toán như trung bình, tổng, hoặc đếm số phần tử theo một chiều cụ thể của mảng. Điều này rất hữu ích khi bạn muốn phân tích dữ liệu theo thời gian, không gian, hoặc bất kỳ chiều nào khác mà dữ liệu của bạn có.


```python
# tạo ra mảng 3 chiều mô phỏng dữ liệu NDVI theo thời gian với kích thước 12x100x100 (12 tháng, mỗi tháng là một bản đồ 100x100)
ndvi = np.random.rand(12, 100, 100) * 2 - 1  # Giá trị NDVI từ -1 đến 1
# Tính giá trị NDVI trung bình hàng năm cho mỗi pixel
annual_ndvi = np.mean(ndvi, axis=0)
print(f"Shape của mảng NDVI trung bình hàng năm: {annual_ndvi.shape}")
# Tính giá trị NDVI cao nhất trong năm cho mỗi pixel
max_ndvi = np.max(ndvi, axis=0)
print(f"Shape của mảng NDVI cao nhất trong năm: {max_ndvi.shape}")
# Xác định số tháng có NDVI > 0.5 cho mỗi pixel
ndvi_above_threshold = np.sum(ndvi > 0.5, axis=0)
print(f"Shape của mảng số tháng có NDVI > 0.5: {ndvi_above_threshold.shape}")
```

    Shape của mảng NDVI trung bình hàng năm: (100, 100)
    Shape của mảng NDVI cao nhất trong năm: (100, 100)
    Shape của mảng số tháng có NDVI > 0.5: (100, 100)
    

### 7.5.3. Clip giá trị

Hàm `clip` trong NumPy được sử dụng để giới hạn giá trị của một mảng trong một khoảng nhất định. Điều này rất hữu ích khi bạn muốn đảm bảo rằng các giá trị của mảng không vượt quá một ngưỡng cụ thể, chẳng hạn như khi làm việc với dữ liệu NDVI, nơi giá trị phải nằm trong khoảng từ -1 đến 1.


```python
# Ham clip (giới hạn giá trị) để đảm bảo NDVI nằm trong khoảng -1 đến 1
clipped_ndvi = np.clip(ndvi, -1, 1)
print(f"Giá trị lớn nhất trong mảng NDVI sau khi clip: {np.max(clipped_ndvi)}")
print(f"Giá trị nhỏ nhất trong mảng NDVI sau khi clip: {np.min(clipped_ndvi)}")
```

    Giá trị lớn nhất trong mảng NDVI sau khi clip: 0.9999879149782394
    Giá trị nhỏ nhất trong mảng NDVI sau khi clip: -0.9999742130704479
    

### 7.5.4. Sử dụng hàm `np.apply_along_axis`

Hàm `np.apply_along_axis` được sử dụng để áp dụng một hàm tùy chỉnh dọc theo một trục cụ thể của mảng. Điều này rất hữu ích khi bạn muốn thực hiện một phép biến đổi hoặc xử lý dữ liệu theo từng phần tử dọc theo một chiều nhất định, chẳng hạn như làm mượt dữ liệu NDVI theo trục thời gian.


```python
# sử dụng median filter để làm mịn dữ liệu NDVI hàng năm
from scipy.ndimage import median_filter
ndvi_filter = np.apply_along_axis(median_filter, 0, ndvi, size=3) # Áp dụng median filter với kích thước 3 theo axis 0 (thời gian).
# Tương tự vậy, ta có thể sử dụng savitzky_golay filter để làm mượt dữ liệu NDVI
from scipy.signal import savgol_filter
# Tạo một vài giá trị thiếu trong dữ liệu NDVI để minh họa
ndvi[3, 20:30, 20:30] = np.nan
ndvi_smooth = np.apply_along_axis(savgol_filter, 0, ndvi, window_length=3, polyorder=2) # Áp dụng savitzky_golay filter với cửa sổ dài 5 và bậc đa thức 2 theo axis 0 (thời gian).
# Mình cũng có thể viết 1 hàm tùy chỉnh để xử lý dữ liệu NDVI theo ý muốn
def ndvi_filter_func(a, window_size=3, poly_order=2):
    """Hàm làm mịn dữ liệu NDVI sử dụng savitzky_golay filter."""
    if np.isnan(a).all():
        return a    
    else:
        return savgol_filter(a, window_length=window_size, polyorder=poly_order)

ndvi_smooth = np.apply_along_axis(ndvi_filter_func, 0, ndvi, window_size=3, poly_order=2)
```

### 7.5.5. Chuyển vị của một mảng

Chuyển vị được sử dụng để thay đổi thứ tự của các trục trong một mảng. Điều này hữu ích khi bạn muốn thay đổi cách dữ liệu được tổ chức hoặc khi bạn cần chuẩn bị dữ liệu cho một thuật toán yêu cầu một thứ tự trục cụ thể.


```python
# transpose (chuyển vị) của một mảng 3D 
data = np.random.rand(2, 3, 4) # Mảng 3D với kích thước (2, 3, 4)
transposed_data = np.transpose(data, (1, 0, 2)) # Chuyển vị mảng theo thứ tự trục mới (1, 0, 2)
print('Dữ liệu gốc:\n', data.shape)
print('Dữ liệu sau khi chuyển vị:\n', transposed_data.shape)
```

    Dữ liệu gốc:
     (2, 3, 4)
    Dữ liệu sau khi chuyển vị:
     (3, 2, 4)
    

## Tóm tắt

Bạn đã hoàn thành Bài 6 và học được nền tảng NumPy - thư viện cốt lõi cho tính toán khoa học trong Python.

### Các khái niệm chính đã nắm vững:
- ✅ **NumPy arrays**: Cấu trúc dữ liệu mảng hiệu quả thay thế cho Python lists
- ✅ **Tạo arrays**: Sử dụng zeros(), ones(), arange(), linspace(), random functions
- ✅ **Indexing và slicing**: Truy cập và thao tác dữ liệu mảng đa chiều
- ✅ **Array operations**: Phép toán vectorized nhanh chóng trên toàn bộ mảng
- ✅ **Broadcasting**: Thực hiện phép toán giữa các mảng có kích thước khác nhau
- ✅ **Statistical functions**: mean(), max(), min(), sum() cho phân tích dữ liệu
- ✅ **Advanced operations**: reshape(), concatenate(), where(), apply_along_axis()

### Kỹ năng bạn có thể áp dụng:
- Xử lý dữ liệu raster và ảnh vệ tinh một cách hiệu quả
- Thực hiện tính toán khoa học và phân tích thống kê trên datasets lớn
- Manipulate dữ liệu đa chiều như NDVI time series và climate data
- Áp dụng các bộ lọc và thuật toán xử lý tín hiệu cho dữ liệu geospatial
- Chuẩn bị nền tảng cho các thư viện advanced như pandas, scikit-learn, và rasterio
