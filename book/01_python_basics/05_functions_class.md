# Bài 5: Hàm và lớp (Functions và Classes)

Functions và Classes là hai khái niệm cốt lõi của lập trình Python, giúp tổ chức code một cách có cấu trúc và tái sử dụng. Trong phân tích không gian địa lý, chúng giúp xây dựng các công cụ và workflows có thể tái sử dụng.

## 5.1. Mục tiêu học tập
- Định nghĩa và sử dụng functions với parameters và return values
- Hiểu về scope và lifetime của variables
- Tạo và sử dụng classes với attributes và methods
- Áp dụng OOP concepts cho bài toán geospatial
- Xây dựng reusable code cho data analysis

## 5.2. Hàm Cơ Bản (functions)

Functions là blocks của code có thể tái sử dụng để thực hiện một tác vụ cụ thể.

### 5.2.1. Functions có và không có tham số (parameters)

- **Không có tham số**


```python
# Hàm đơn giản không có tham số (parameters)
def greet():
    print("Xin chào! Chào mừng đến với khóa học Python GIS")

# Gọi hàm
greet()
```

- **Có tham số**


```python
# Hàm với tham số (parameters)
def greet_person(name):
    print(f"Xin chào {name}! Chào mừng bạn đến với Python GIS")

greet_person("Minh")
```

### 5.2.2. Hàm với giá trị trả về

Functions có thể trả về giá trị để sử dụng trong code khác.

- **Hàm trả về một giá trị**


```python
# Hàm tính diện tích hình chữ nhật

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

### 5.2.3. Hàm với tham số mặc định và kết hợp

Functions có thể có default values và được gọi với keyword arguments.

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

### 5.2.4. Hàm với variable Arguments (*args và **kwargs)

Hàm có thể nhận số lượng arguments linh hoạt.

- **Hàm với positional arguments `(*args)`**


```python
# Function với *args (variable positional arguments). Bản chất của *args là một tuple.
def calculate_center(*coordinates):
    """
    Tính tọa độ trung tâm của nhiều điểm.
    """
    if not coordinates:
        return 0, 0
    
    total_lat = sum(coord[0] for coord in coordinates)
    total_lon = sum(coord[1] for coord in coordinates)
    count = len(coordinates)
    
    center_lat = total_lat / count
    center_lon = total_lon / count
    
    return center_lat, center_lon

# Sử dụng với số lượng coordinates khác nhau
vietnam_cities = [
    (21.0285, 105.8542),  # Hà Nội
    (10.8231, 106.6297),  # TP.HCM
    (16.0544, 108.2022),  # Đà Nẵng
    (20.8449, 106.6881),  # Hải Phòng
    (10.0452, 105.7469)   # Cần Thơ
]

center = calculate_center(*vietnam_cities)
print(f"Tọa độ trung tâm Việt Nam: {center[0]:.4f}, {center[1]:.4f}")
```

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

### 5.2.5. Hàm Lambda
Hàm lambda là anonymous functions ngắn gọn cho các tác vụ đơn giản.

- **Hàm lambda đơn giản**


```python
# Tạo danh sách 
numbers = [1, 2, 3, 4, 5]
# Bình phương các số trong danh sách khi chưa sử dụng lambda
def square(x):
    return x ** 2
square_numbers = list(map(square, numbers))
print(square_numbers)  # Output: [1, 4, 9, 16, 25]
```


```python
# Sử dụng hàm lambda để thay thế
square_numbers_lambda = list(map(lambda x: x ** 2, numbers))
print(square_numbers_lambda)  # Output: [1, 4, 9, 16, 25]
```

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
print(sorted_by_population)
```


```python
# Lọc thành phố có dân số > 5 triệu
large_cities = list(filter(lambda city: city['population'] > 5000000, cities_data))
```

### 5.2.6. list comprehension


```python
mlist = [1, 2, 3, 4, 5]
squared_list = [x**2 for x in mlist]
print(squared_list)  # Output: [1, 4, 9, 16, 25]
```


```python
# list comprehension với điều kiện
even_squared = [x**2 for x in mlist if x % 2 == 0]
print(even_squared)  # Output: [4, 16]
# list comprehension với nhiều vòng lặp
matrix = [[1, 2], [3, 4], [5, 6]]
flattened = [num for row in matrix for num in row]
print(flattened)  # Output: [1, 2, 3, 4, 5, 6]
```

## 5.3. Classes Cơ Bản

Classes cho phép tạo objects với attributes và methods riêng.

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
print(areas)
```

    [4, 9, 16, 25]
    


```python
class Location:
    def __init__(self, name, latitude, longitude):
        self.name = name
        self.latitude = latitude
        self.longitude = longitude
    
    def show_location(self):
        return f"{self.name}: ({self.latitude}, {self.longitude})"
    def coordinates(self):
        return self.latitude, self.longitude
    
location = Location("Hà Nội", 21.0285, 105.8542)
print(location.show_location())  # Output: Hà Nội: (21.0285, 105.8542)
print(location.coordinates())  # Output: (21.0285, 105.8542)
```

    Hà Nội: (21.0285, 105.8542)
    (21.0285, 105.8542)
    

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
    


```python

```


```python
class GeoLocation:
    """Class nâng cao cho địa điểm với properties và methods đặc biệt"""
    
    # Class variable
    earth_radius_km = 6371
    
    def __init__(self, name, latitude, longitude, elevation=0):
        self._name = name
        self._latitude = latitude
        self._longitude = longitude
        self._elevation = elevation
    
    # Property getter và setter
    @property
    def name(self):
        return self._name
    
    @name.setter
    def name(self, value):
        if isinstance(value, str) and value.strip():
            self._name = value.strip()
        else:
            raise ValueError("Tên phải là string không rỗng")
    
    @property
    def latitude(self):
        return self._latitude
    
    @latitude.setter
    def latitude(self, value):
        if -90 <= value <= 90:
            self._latitude = value
        else:
            raise ValueError("Latitude phải trong khoảng -90 đến 90")
    
    @property
    def longitude(self):
        return self._longitude
    
    @longitude.setter
    def longitude(self, value):
        if -180 <= value <= 180:
            self._longitude = value
        else:
            raise ValueError("Longitude phải trong khoảng -180 đến 180")
    
    @property
    def coordinates(self):
        """Property chỉ đọc trả về tuple tọa độ"""
        return (self._latitude, self._longitude)
    
    # Instance method
    def distance_to(self, other):
        """Tính khoảng cách đến địa điểm khác"""
        return self.haversine_distance(
            self._latitude, self._longitude,
            other._latitude, other._longitude
        )
    
    # Static method
    @staticmethod
    def haversine_distance(lat1, lon1, lat2, lon2):
        """Static method tính khoảng cách Haversine"""
        import math 
        lat1_rad = math.radians(lat1)
        lon1_rad = math.radians(lon1)
        lat2_rad = math.radians(lat2)
        lon2_rad = math.radians(lon2)
        
        dlat = lat2_rad - lat1_rad
        dlon = lon2_rad - lon1_rad
        
        a = (math.sin(dlat/2)**2 + 
             math.cos(lat1_rad) * math.cos(lat2_rad) * math.sin(dlon/2)**2)
        c = 2 * math.atan2(math.sqrt(a), math.sqrt(1-a))
        
        return GeoLocation.earth_radius_km * c
    
    # Class method
    @classmethod
    def from_string(cls, location_string):
        """Class method tạo object từ string format 'name,lat,lon,elevation'"""
        parts = location_string.split(',')
        if len(parts) == 4:
            name = parts[0].strip()
            lat = float(parts[1].strip())
            lon = float(parts[2].strip())
            elev = float(parts[3].strip())
            return cls(name, lat, lon, elev)
        else:
            raise ValueError("Format phải là 'name,lat,lon,elevation'")
    
    def __str__(self):
        """String representation của object"""
        return f"{self._name} ({self._latitude}, {self._longitude})"
    
    def __repr__(self):
        """Developer-friendly representation"""
        return f"GeoLocation('{self._name}', {self._latitude}, {self._longitude}, {self._elevation})"

# Sử dụng class nâng cao
loc1 = GeoLocation("Hà Nội", 21.0285, 105.8542, 12)
print(f"Location: {loc1}")
print(f"Coordinates: {loc1.coordinates}")

# Sử dụng class method
loc2 = GeoLocation.from_string("TP.HCM,10.8231,106.6297,19")
print(f"From string: {loc2}")

# Sử dụng properties với validation
try:
    loc1.latitude = 95  # Sẽ raise error
except ValueError as e:
    print(f"Error: {e}")

# Sử dụng static method
distance = GeoLocation.haversine_distance(21.0285, 105.8542, 10.8231, 106.6297)
print(f"Distance (static method): {distance:.2f} km")

# Distance giữa objects
distance_obj = loc1.distance_to(loc2)
print(f"Distance (instance method): {distance_obj:.2f} km")
```

### 5.3.3 Inheritance (Kế Thừa)

Classes có thể inherit từ classes khác để tái sử dụng và mở rộng functionality.


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
