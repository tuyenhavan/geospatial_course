# Bài 5: Hàm và lớp

`Hàm` (function) và `lớp` (class) là hai khái niệm cốt lõi của lập trình Python, giúp tổ chức code một cách có cấu trúc và tái sử dụng. Trong phân tích không gian địa lý, chúng giúp xây dựng các công cụ và workflows có thể tái sử dụng.

## 5.1. Mục tiêu học tập
- Định nghĩa và sử dụng functions với parameters và return values
- Hiểu về scope và lifetime của variables
- Tạo và sử dụng classes với attributes và methods
- Áp dụng OOP concepts cho bài toán geospatial
- Xây dựng reusable code cho data analysis

## 5.2. Hàm Cơ Bản (functions)

Functions là blocks của code có thể tái sử dụng để thực hiện một tác vụ cụ thể.

### 5.2.1. Functions có và không có tham số (parameters)

- **Hàm không có tham số**


```python
# Hàm đơn giản không có tham số (parameters)
def greet():
    print("Xin chào! Chào mừng đến với khóa học Python GIS")

# Gọi hàm
greet()
```

    Xin chào! Chào mừng đến với khóa học Python GIS
    

- **Hàm có tham số**


```python
# Hàm với tham số (parameters)
def greet_person(name):
    """Hàm chào hỏi một người cụ thể
    Args:
        name (str): Tên của người cần chào hỏi
    """
    print(f"Xin chào {name}! Chào mừng bạn đến với Python GIS")

greet_person("Minh")
```

    Xin chào Minh! Chào mừng bạn đến với Python GIS
    

### 5.2.2. Hàm với giá trị trả về

Functions có thể trả về giá trị để sử dụng trong code khác.

- **Hàm trả về một giá trị**


```python
# Hàm tính diện tích hình chữ nhật. Chữ mô tả trong hàm đặt trong 3 dấu nháy kép (""" """)
# được gọi là docstring, giúp giải thích mục đích và cách sử dụng của hàm.

def calculate_rectangle_area(length, width):
    """
    Tính diện tích hình chữ nhật
    Diện tích hình chữ nhật được tính bằng công thức: chiều dài * chiều rộng

    Args:
        length (float): Chiều dài của hình chữ nhật
        width (float): Chiều rộng của hình chữ nhật

    Returns:
        float: Diện tích của hình chữ nhật
    """
    return length * width
# Ví dụ sử dụng hàm
area = calculate_rectangle_area(5, 3)
print(f"Diện tích hình chữ nhật là: {area} m2")
```

    Diện tích hình chữ nhật là: 15 m2
    

- **Hàm trả về với nhiều giá trị**


```python
# Hàm trả về nhiều giá trị (multiple return values)

def calculate_areas(length, width):
    """
    Tính diện tích và chu vi hình chữ nhật

    Args:
        length (float): Chiều dài của hình chữ nhật
        width (float): Chiều rộng của hình chữ nhật

    Returns:
        tuple: Diện tích và chu vi của hình chữ nhật
    """
    area = length * width
    perimeter = 2 * (length + width)
    return area, perimeter # mặc định trả về là một tuple nếu có nhiều giá trị cách nhau bằng dấu phẩy.
# Ví dụ sử dụng hàm
area, perimeter = calculate_areas(5, 3)
print(f"Diện tích: {area} m2, Chu vi: {perimeter} m")
```

    Diện tích: 15 m2, Chu vi: 16 m
    


```python
# Hàm tính phương trình bậc 2
import math
def solve_quadratic(a, b, c):
    """
    Giải phương trình bậc 2 ax^2 + bx + c = 0

    Args:
        a (float): Hệ số a
        b (float): Hệ số b
        c (float): Hệ số c

    Returns:
        tuple: Nghiệm của phương trình (x1, x2) hoặc thông báo nếu không có nghiệm thực
    """
    if not a>0:
        return "Hệ số a phải lớn hơn 0"
    delta = b**2 - 4*a*c
    if delta > 0:
        x1 = (-b + math.sqrt(delta)) / (2*a)
        x2 = (-b - math.sqrt(delta)) / (2*a)
        return (x1, x2)
    elif delta == 0:
        x = -b / (2*a)
        return (x,)
    else:
        return "Phương trình vô nghiệm thực"
# Ví dụ sử dụng hàm
result = solve_quadratic(1, -3, 2)
print(f"Nghiệm của phương trình là: {result}")
```

    Nghiệm của phương trình là: (2.0, 1.0)
    

### 5.2.3. Hàm với tham số mặc định hoặc kết hợp

Hàm có thể có một hoặc nhiều tham số mặc định (default parameters).

- **Hàm với tham số mặc định**


```python
# Hàm với các tham số mặc định (default parameters). Ta có thể có 1 hoặc nhiều default parameters trong hàm.
def calculate_areas(length, width=2):
    """
    Tính diện tích và chu vi hình chữ nhật với chiều rộng mặc định là 2

    Args:
        length (float): Chiều dài của hình chữ nhật
        width (float, optional): Chiều rộng của hình chữ nhật. Mặc định là 2.

    Returns:
        tuple: Diện tích và chu vi của hình chữ nhật
    """
    area = length * width
    perimeter = 2 * (length + width)
    return area, perimeter
# Ví dụ sử dụng hàm với chiều rộng mặc định
area, perimeter = calculate_areas(5)
print(f"Diện tích: {area} m2, Chu vi: {perimeter} m")
# Ví dụ sử dụng hàm với chiều rộng tùy chỉnh
area, perimeter = calculate_areas(5, 4)
print(f"Diện tích: {area} m2, Chu vi: {perimeter} m")
```

    Diện tích: 10 m2, Chu vi: 14 m
    Diện tích: 20 m2, Chu vi: 18 m
    

### 5.2.4. Hàm với variable Arguments (*args và **kwargs)

Hàm có thể nhận số lượng arguments linh hoạt.

- **Hàm với positional arguments `(*args)`**


```python
# Function với *args (variable positional arguments). Bản chất của *args là một tuple.
def display_number(*args):
    """
    Hiển thị các số được truyền vào dưới dạng một chuỗi

    Args:
        *args: Một số lượng không giới hạn các đối số vị trí (positional arguments)
    """
    numbers_str = ", ".join(str(num) for num in args)
    print(f"Các số đã nhập: {numbers_str}")
# Ví dụ sử dụng hàm với *args
display_number(1, 2, 3, 4, 5)
```

    Các số đã nhập: 1, 2, 3, 4, 5
    

- **Hàm với keyword arguments `(**kwargs)`**


```python
# Function với **kwargs (variable keyword arguments). Bản chất của **kwargs là một dictionary.
def create_location_info(**info):
    """Tạo thông tin về một địa điểm từ keyword arguments"""
    result = {}
    for key, value in info.items():
        if key not in result:
            result[key] = value
    return result

# Sử dụng với các keyword arguments khác nhau
hanoi_info = create_location_info(
    name="Hà Nội",
    latitude=21.0285,
    longitude=105.8542,
    population=8053663,
    country="Việt Nam",
    type="Thủ đô"
)
print(hanoi_info)
```

    {'name': 'Hà Nội', 'latitude': 21.0285, 'longitude': 105.8542, 'population': 8053663, 'country': 'Việt Nam', 'type': 'Thủ đô'}
    


```python
# Hoặc có thể sử dụng dictionary để truyền vào **kwargs
hanoi_info = create_location_info(**{
    "name": "Hà Nội",
    "latitude": 21.0285,
    "longitude": 105.8542,
    "population": 8053663,  
        "country": "Việt Nam",
        "type": "Thủ đô"
})
print(hanoi_info)
```

    {'name': 'Hà Nội', 'latitude': 21.0285, 'longitude': 105.8542, 'population': 8053663, 'country': 'Việt Nam', 'type': 'Thủ đô'}
    

### 5.2.5. Hàm Lambda
Hàm lambda là anonymous functions ngắn gọn cho các tác vụ đơn giản.

- **Hàm lambda đơn giản**


```python
# Tạo danh sách 
numbers = [1, 2, 3, 4, 5]
# Sử dụng hàm lambda để thay thế
square_numbers_lambda = list(map(lambda x: x ** 2, numbers))
print(f"Square numbers: {square_numbers_lambda}")  # Output: [1, 4, 9, 16, 25]
```

    Square numbers: [1, 4, 9, 16, 25]
    

- **Hàm lambda với built-in functions**


```python
# Sử dụng lambda với built-in functions
cities_data = [
    {'name': 'Hà Nội', 'population': 8053663},
    {'name': 'TP.HCM', 'population': 9420000},
    {'name': 'Đà Nẵng', 'population': 1134000},
    {'name': 'Hải Phòng', 'population': 2028220}
]

# Sắp xếp theo dân số sử dụng lambda
sorted_by_population = sorted(cities_data, key=lambda city: city['population'], reverse=True)
print(f"Cities sorted by population: {sorted_by_population}")
```

    Cities sorted by population: [{'name': 'TP.HCM', 'population': 9420000}, {'name': 'Hà Nội', 'population': 8053663}, {'name': 'Hải Phòng', 'population': 2028220}, {'name': 'Đà Nẵng', 'population': 1134000}]
    


```python
# Lọc thành phố có dân số > 5 triệu
large_cities = list(filter(lambda city: city['population'] > 5000000, cities_data))
```

### 5.2.6. List comprehension

- **Bình phương với `list comprehension`**


```python
mlist = [1, 2, 3, 4, 5]
squared_list = [x**2 for x in mlist]
print(squared_list)  # Output: [1, 4, 9, 16, 25]
```

    [1, 4, 9, 16, 25]
    

- **`list comprehension` với điều kiện**


```python
# list comprehension với điều kiện
even_squared = [x**2 for x in mlist if x % 2 == 0] # Chỉ bình phương các số chẵn
print(f"Số chẵn bình phương: {even_squared}")  # Output: [4, 16]
# list comprehension với nhiều vòng lặp
matrix = [[1, 2], [3, 4], [5, 6]]
flattened = [num for row in matrix for num in row] # Làm phẳng ma trận
print(f"Ma trận làm phẳng: {flattened}")  # Output: [1, 2, 3, 4, 5, 6]
```

    Số chẵn bình phương: [4, 16]
    Ma trận làm phẳng: [1, 2, 3, 4, 5, 6]
    

## 5.3. Classes Cơ Bản

Class là một khuân mẫu, được sử dụng khi bạn muốn tổ chức, quản lý và tái sử dụng logic xoay quanh một đối tượng thay vì viết các hàm rời rạc. Mỗi đối tượng (object) được tạo từ class sẽ bao gồm thuộc tính (attributes) để lưu trữ dữ liệu và phương thức (methods) để thực hiện các thao tác liên quan đến dữ liệu đó.

### 5.3.1. Tạo class với phương thức (method) và thuộc tính (attributes) cơ bản


```python
class Square:
    """ Lớp đại diện cho hình vuông với các phương thức tính diện tích và chu vi"""
    def __init__(self, side_length):
        self.side_length = side_length
    
    def area(self):
        """ Tính diện tích hình vuông"""
        return self.side_length ** 2
    
    def perimeter(self):
        """ Tính chu vi hình vuông"""
        return 4 * self.side_length
    def show_info(self):
        return f"Hình vuông có cạnh dài {self.side_length} m, diện tích {self.area()} m2, chu vi {self.perimeter()} m"
square = Square(4)
print(f"Diện tích hình vuông: {square.area()} m2")
print(f"Chu vi hình vuông: {square.perimeter()} m")
print(square.show_info())
```

    Diện tích hình vuông: 16 m2
    Chu vi hình vuông: 16 m
    Hình vuông có cạnh dài 4 m, diện tích 16 m2, chu vi 16 m
    

## 5.3.2. Tạo class với phương thức nâng cao và tính kế thừa

Class trong Python không chỉ chứa thuộc tính và phương thức, mà còn hỗ trợ kế thừa để tái sử dụng code và các phương thức nâng cao như @staticmethod, giúp thiết kế chương trình linh hoạt và rõ ràng hơn. Dưới đây ta xem xét một ví dụ về class giải phương trình bậc nhất và bậc hai.

- **Tạo ra lớp cơ sở (base class)**

Base class dùng để định nghĩa chuẩn chung (interface) cho các class con, đảm bảo chúng phải triển khai các phương thức cần thiết (như solve()), giúp code dễ mở rộng và sử dụng thống nhất.


```python
import math
class Equation:
    """ Lớp cơ sở cho các phương trình, có phương thức solve() chưa được triển khai (abstract method)"""
    def __init__(self):
        pass
    def solve(self):
        raise NotImplementedError("Phương thức solve() chưa được triển khai")
```

- **Viết class giải phương trình bậc nhất và kế thừa từ lớp cơ sở**


```python
class LinearSolver(Equation):
    """ Lớp giải phương trình bậc nhất ax + b = 0"""
    def __init__(self, a, b):
        super().__init__()
        self.a = a
        self.b = b
    
    def solve(self):
        """ Giải phương trình bậc nhất ax + b = 0"""
        if self.a == 0:
            if self.b == 0:
                return {"message": "Phương trình vô số nghiệm"}
            else:
                return {"message": "Phương trình vô nghiệm"}
        else:
            return {"x": -self.b / self.a}
# Ví dụ sử dụng LinearSolver
linear_solver = LinearSolver(2, -4)
solution = linear_solver.solve()
print(f"Nghiệm của phương trình bậc nhất: {solution}")
```

    Nghiệm của phương trình bậc nhất: {'x': 2.0}
    

- **Tạo lớp giải phương trình bậc hai có `staticmethod` và thừa kế từ `Equation`**


```python
class QuadraticSolver(Equation):
    """ Lớp giải phương trình bậc 2 ax^2 + bx + c = 0"""
    def __init__(self, a, b, c):
        super().__init__()
        self.a = a
        self.b = b
        self.c = c
    
    @staticmethod
    def delta(a, b, c):
        """ Tính discriminant (delta) của phương trình bậc 2"""
        return b**2 - 4*a*c

    def solve(self):
        if self.a == 0:
            return LinearSolver(self.b, self.c).solve()
        
        delta_value = self.delta(self.a, self.b, self.c)
        if delta_value > 0:
            x1 = (-self.b + math.sqrt(delta_value)) / (2*self.a)
            x2 = (-self.b - math.sqrt(delta_value)) / (2*self.a)
            return {
                "x1": x1,
                "x2": x2
            }
        elif delta_value == 0:
            x = -self.b / (2*self.a)
            return {
                "x": x
            }
        else:
            return {"message": "Phương trình vô nghiệm thực"}
# Ví dụ sử dụng QuadraticSolver
quadratic_solver = QuadraticSolver(1, -3, 2)
result = quadratic_solver.solve()
print(f"Nghiệm của phương trình bậc 2: {result}")
```

    Nghiệm của phương trình bậc 2: {'x1': 2.0, 'x2': 1.0}
    

## Tóm tắt

Bạn đã hoàn thành Bài 5 và học được Functions và Classes - hai khái niệm quan trọng nhất trong lập trình Python có cấu trúc.

### Các khái niệm chính đã nắm vững:
- ✅ **Functions**: Định nghĩa, parameters, return values, và function scope
- ✅ **Default parameters**: Sử dụng giá trị mặc định và keyword arguments
- ✅ **Variable arguments**: *args và **kwargs cho flexibility
- ✅ **Lambda functions**: Anonymous functions cho các tác vụ ngắn gọn
- ✅ **Classes**: Định nghĩa classes với __init__, attributes và methods
- ✅ **Inheritance**: Kế thừa để tái sử dụng và mở rộng functionality

### Kỹ năng bạn có thể áp dụng:
- Xây dựng reusable functions cho các tính toán geospatial phức tạp
- Tạo classes để mô hình hóa các entities trong GIS (điểm, đường, polygon)
- Sử dụng OOP để tổ chức code phân tích dữ liệu một cách có cấu trúc
- Xây dựng frameworks và libraries tùy chỉnh cho geospatial workflows
- Tích hợp với các thư viện GIS lớn như GeoPandas, Shapely một cách hiệu quả
