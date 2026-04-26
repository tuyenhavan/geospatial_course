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
    


```python
# Ví dụ tính diện tích hình vuông từ 1 danh sách cạnh
side_lengths = [2, 3, 4, 5]
areas = [Square(side).area() for side in side_lengths]
print(f"Diện tích các hình vuông: {areas} m2")
```

    Diện tích các hình vuông: [4, 9, 16, 25] m2
    

## 5.3.2. Tạo class với phương thức nâng cao

Classes có thể có properties, class methods, và static methods.


```python
class Location:
    def __init__(self, name, latitude, longitude):
        self.name = name
        self.latitude = latitude
        self.longitude = longitude
    
    def __str__(self):
        return f"{self.name}: ({self.latitude}, {self.longitude})"
    def __call__(self):
        return f"{self.name}: ({self.latitude}, {self.longitude})"
    def coordinate_point(self):
        return self.latitude, self.longitude
    
location = Location("Hà Nội", 21.0285, 105.8542)
print(location)  # Output: Hà Nội: (21.0285, 105.8542)
print(location.coordinate_point())  # Output: (21.0285, 105.8542)
print(location())  # Output: Hà Nội: (21.0285, 105.8542)
```

    Hà Nội: (21.0285, 105.8542)
    (21.0285, 105.8542)
    Hà Nội: (21.0285, 105.8542)
    


```python
class Location:
    def __init__(self, name, latitude, longitude):
        self.name = name
        self.latitude = latitude
        self.longitude = longitude
    
    def __str__(self):
        return f"{self.name}: ({self.latitude}, {self.longitude})"
    
    def __call__(self):
        return f"{self.name}: ({self.latitude}, {self.longitude})"
    
    def coordinate_point(self):
        return self.latitude, self.longitude
    def __add__(self, other):
        if isinstance(other, Location):
            other_lat, other_lon = other.coordinate_point()
            new_lat = (self.latitude + other_lat) / 2
            new_lon = (self.longitude + other_lon) / 2
            return Location(f"Midpoint of {self.name} and {other.name}", new_lat, new_lon)
        return NotImplemented
location1 = Location("Hà Nội", 21.0285, 105.8542)
location2 = Location("TP.HCM", 10.8231, 106.6297)
midpoint = location1 + location2
print(midpoint)  # Output: Midpoint of Hà Nội and TP.HCM: (
```

    Midpoint of Hà Nội and TP.HCM: (15.9258, 106.24195)
    

### 5.3.3 Inheritance (Kế Thừa)

Classes có thể kế thừa từ classes khác để tái sử dụng và mở rộng functionality.


```python
# Base class
class Location:
    """Base class cho các địa điểm"""
    
    def __init__(self, name, latitude, longitude):
        self.name = name
        self.latitude = latitude
        self.longitude = longitude
    
    def display_coordinates(self):
        print(f"{self.name}: ({self.latitude}, {self.longitude})")


# Derived class - City inherit từ Location
class City(Location):
    """Class City kế thừa từ Location"""
    
    def __init__(self, name, latitude, longitude, population, country):
        super().__init__(name, latitude, longitude)  # Gọi constructor của parent
        self.population = population
        self.country = country
    
    def display_info(self):
        """Override method từ parent và thêm thông tin"""
        super().display_coordinates()  # Gọi method từ parent
        print(f"Dân số: {self.population:,}")
        print(f"Quốc gia: {self.country}")
    
    def population_density_info(self):
        """Method riêng của City"""
        if self.population > 5000000:
            return "Siêu đô thị"
        elif self.population > 1000000:
            return "Đô thị lớn"
        else:
            return "Đô thị trung bình"

# Derived class - NaturalSite inherit từ Location
class NaturalSite(Location):
    """Class cho các địa điểm tự nhiên"""
    
    def __init__(self, name, latitude, longitude, site_type, protected_status=False):
        super().__init__(name, latitude, longitude)
        self.site_type = site_type
        self.protected_status = protected_status
    
    def display_info(self):
        super().display_coordinates()
        print(f"Loại: {self.site_type}")
        print(f"Được bảo vệ: {'Có' if self.protected_status else 'Không'}")
    
    def conservation_level(self):
        return "Vùng bảo tồn" if self.protected_status else "Không được bảo vệ"
```


```python
# Sử dụng inheritance
hanoi = City("Hà Nội", 21.0285, 105.8542, 8053663, "Việt Nam")
halong = NaturalSite("Vịnh Hạ Long", 20.9101, 107.1839, "Vịnh biển", True)

print("=== THÔNG TIN THÀNH PHỐ ===")
hanoi.display_info()
print(f"Phân loại: {hanoi.population_density_info()}")

print("\n=== THÔNG TIN ĐỊA ĐIỂM TỰ NHIÊN ===")
halong.display_info()
print(f"Tình trạng bảo tồn: {halong.conservation_level()}")

# Kiểm tra inheritance
print(f"\nKiểm tra inheritance:")
print(f"hanoi là instance của City: {isinstance(hanoi, City)}")
print(f"hanoi là instance của Location: {isinstance(hanoi, Location)}")
print(f"City là subclass của Location: {issubclass(City, Location)}")
```

    === THÔNG TIN THÀNH PHỐ ===
    Hà Nội: (21.0285, 105.8542)
    Dân số: 8,053,663
    Quốc gia: Việt Nam
    Phân loại: Siêu đô thị
    
    === THÔNG TIN ĐỊA ĐIỂM TỰ NHIÊN ===
    Vịnh Hạ Long: (20.9101, 107.1839)
    Loại: Vịnh biển
    Được bảo vệ: Có
    Tình trạng bảo tồn: Vùng bảo tồn
    
    Kiểm tra inheritance:
    hanoi là instance của City: True
    hanoi là instance của Location: True
    City là subclass của Location: True
    

## Tóm tắt

Bạn đã hoàn thành Bài 5 và học được Functions và Classes - hai khái niệm quan trọng nhất trong lập trình Python có cấu trúc.

### Các khái niệm chính đã nắm vững:
- ✅ **Functions**: Định nghĩa, parameters, return values, và function scope
- ✅ **Default parameters**: Sử dụng giá trị mặc định và keyword arguments
- ✅ **Variable arguments**: *args và **kwargs cho flexibility
- ✅ **Lambda functions**: Anonymous functions cho các tác vụ ngắn gọn
- ✅ **Classes**: Định nghĩa classes với __init__, attributes và methods
- ✅ **Properties**: Getters, setters và validation với @property decorator
- ✅ **Special methods**: __str__, __repr__, static methods và class methods
- ✅ **Inheritance**: Kế thừa để tái sử dụng và mở rộng functionality

### Kỹ năng bạn có thể áp dụng:
- Xây dựng reusable functions cho các tính toán geospatial phức tạp
- Tạo classes để mô hình hóa các entities trong GIS (điểm, đường, polygon)
- Sử dụng OOP để tổ chức code phân tích dữ liệu một cách có cấu trúc
- Xây dựng frameworks và libraries tùy chỉnh cho geospatial workflows
- Tích hợp với các thư viện GIS lớn như GeoPandas, Shapely một cách hiệu quả
