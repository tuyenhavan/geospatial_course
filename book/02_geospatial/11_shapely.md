# Bài 10: Cơ bản về Shapely

Chào mừng bạn đến với bài học đầu tiên về phân tích dữ liệu địa không gian với Python! Trong bài học này, chúng ta sẽ khám phá Shapely - một thư viện nền tảng để thao tác và phân tích các đối tượng hình học phẳng trong Python.

Shapely dựa trên thư viện GEOS được sử dụng rộng rãi (công cụ hình học của PostGIS) và cung cấp giao diện Python đơn giản, trực quan để làm việc với các hình dạng hình học 2D.

## 10.1. Mục tiêu học tập
Sau khi hoàn thành bài học này, bạn sẽ có thể:
- Tạo và thao tác các đối tượng hình học cơ bản (Point, LineString, Polygon)
- Hiểu các thuộc tính và phương thức của đối tượng hình học
- Thực hiện các phép toán không gian (buffer, intersection, union, difference)
- Kiểm tra mối quan hệ không gian (contains, intersects, touches)
- Áp dụng các phép biến đổi hình học (rotate, scale, translate)
- Làm việc với các đối tượng đa hình học phức tạp
- Tối ưu hóa hiệu suất cho dữ liệu không gian lớn

## 10.2. Cài đặt và import thư viện

- **Cài đặt thư viện**

Nếu bạn chưa cài đặt các thư viện cần thiết, hãy chạy:

```bash
pip install shapely
```

> **Lưu ý:** Khi bạn cài `geopandas` hoặc `rasterio`, thư viện **Shapely thường được cài tự động** vì chúng phụ thuộc vào Shapely.

- **Import thư viện shapely**


```python
from shapely.geometry import Point, LineString, Polygon, MultiPoint, MultiLineString, MultiPolygon
from shapely.geometry import GeometryCollection
from shapely.ops import unary_union, transform, nearest_points, split
from shapely import affinity
from shapely.validation import make_valid
# Thư viện cần thiết khác
import numpy as np
import matplotlib.pyplot as plt
import warnings
warnings.filterwarnings('ignore')
```

## 10.3. Các đối tượng hình học cơ bản

Shapely cung cấp các lớp để biểu diễn các đối tượng hình học. Hiểu những điều này là cần thiết:

- **Point (Điểm)**: Hình học 0 chiều (tọa độ x, y)
- **LineString (Đường)**: Hình học 1 chiều (chuỗi các điểm liên kết)
- **Polygon (Đa giác)**: Hình học 2 chiều (vùng khép kín có thể có lỗ)
- **Multi-geometries (Đa hình học)**: Tập hợp các hình học cùng loại

### 10.3.1. Tạo điểm và kiểm tra thuộc tính


```python
# Tạo điểm - các cách khác nhau
point1 = Point(0, 0)                    # Điểm cơ bản
point2 = Point([1, 1])                  # Từ list/tuple
point3 = Point(np.array([2, 2]))        # Từ mảng numpy
# Chúng ta có thể kiểm tra các thuộc tính và phương thức của điểm như sau:
print(f"Tọa độ điểm 1: ({point1.x}, {point1.y})")
print(f"Điểm 1 WKT: {point1.wkt}")
# Thuộc tính điểm
print(f"Giới hạn điểm (minx, miny, maxx, maxy): {point1.bounds}")
print(f"Điểm có trống: {point1.is_empty}")
print(f"Điểm hợp lệ: {point1.is_valid}")
print(f"Khoảng cách từ điểm 1 đến điểm 2: {point1.distance(point2)}")
```

### 10.3.2. Tạo đường và kiểm tra thuộc tính


```python
# Tạo đường
line1 = LineString([(0, 0), (1, 1), (2, 0), (3, 1)])
line2 = LineString([point1, point2, point3])
# Kiểm tra các thuộc tính của đường 
print(f"Tọa độ đường 1: {list(line1.coords)}")
print(f"Độ dài đường 1: {line1.length:.3f}")
print(f"Đường 1 khép kín: {line1.is_closed}")
print(f"Đường 1 đơn giản (không tự cắt): {line1.is_simple}")

# Truy cập các phần của đường
print(f"Điểm bắt đầu: {Point(line1.coords[0])}")
print(f"Điểm kết thúc: {Point(line1.coords[-1])}")
```

### 10.3.3. Tạo Polygon và kiểm tra thuộc tính


```python
# Đa giác đơn giản (tam giác)
triangle = Polygon([(0, 0), (1, 0), (0.5, 1), (0, 0)])

# Đa giác vuông
square_coords = [(0, 0), (2, 0), (2, 2), (0, 2), (0, 0)]
square = Polygon(square_coords)

# Đa giác có lỗ (hình donut)
exterior = [(0, 0), (4, 0), (4, 4), (0, 4), (0, 0)]
interior = [(1, 1), (3, 1), (3, 3), (1, 3), (1, 1)]  # lỗ
donut = Polygon(exterior, [interior])
# Hiển thị các thông tin thuộc tính
print(f"Diện tích tam giác: {triangle.area:.3f}")
print(f"Diện tích hình vuông: {square.area}")
print(f"Chu vi hình vuông: {square.length}")
print(f"Diện tích donut (có lỗ): {donut.area}")
print(f"Donut có {len(donut.interiors)} lỗ bên trong")

# Các phần của đa giác
print(f"Đường biên ngoài hình vuông: {list(square.exterior.coords)}")
print(f"Tâm hình vuông: {square.centroid}")
print(f"Hình bao hình vuông (hộp giới hạn): {square.envelope}")
```

### 10.3.4. Trực quan hóa điểm, đường và đa giác


```python
# Trực quan hóa các hình học cơ bản
fig, axes = plt.subplots(2, 2, figsize=(12, 10))

# Biểu đồ 1: Điểm
axes[0, 0].scatter([point1.x, point2.x, point3.x], 
                   [point1.y, point2.y, point3.y], 
                   c=['red', 'blue', 'green'], s=100, alpha=0.7)
axes[0, 0].set_title('Điểm')
axes[0, 0].grid(True, alpha=0.3)

# Biểu đồ 2: Đường
x, y = line1.xy
axes[0, 1].plot(x, y, 'b-', linewidth=2, alpha=0.7)
axes[0, 1].scatter(x, y, c='red', s=50)
axes[0, 1].set_title('Đường')
axes[0, 1].grid(True, alpha=0.3)

# Biểu đồ 3: Đa giác đơn giản
x, y = square.exterior.xy
axes[1, 0].fill(x, y, alpha=0.3, fc='lightblue', ec='blue', linewidth=2)
axes[1, 0].set_title('Đa giác đơn giản (Hình vuông)')
axes[1, 0].grid(True, alpha=0.3)

# Biểu đồ 4: Đa giác có lỗ
x_ext, y_ext = donut.exterior.xy
x_int, y_int = donut.interiors[0].xy
axes[1, 1].fill(x_ext, y_ext, alpha=0.3, fc='lightgreen', ec='green', linewidth=2)
axes[1, 1].fill(x_int, y_int, alpha=1, fc='white', ec='red', linewidth=2)
axes[1, 1].set_title('Đa giác có lỗ (Donut)')
axes[1, 1].grid(True, alpha=0.3)

plt.tight_layout()
plt.show()
```

## 10.4. Mối quan hệ không gian

Kiểm tra mối quan hệ không gian giữa các đối tượng hình học.


```python
# Tạo hình học thử nghiệm cho mối quan hệ không gian
circle = Point(0, 0).buffer(1)  # Hình tròn bán kính 1
square_small = Polygon([(-0.5, -0.5), (0.5, -0.5), (0.5, 0.5), (-0.5, 0.5)])
line_through = LineString([(-2, 0), (2, 0)])  # Đường qua tâm
line_outside = LineString([(-2, 2), (2, 2)])   # Đường bên ngoài
point_inside = Point(0, 0)
point_outside = Point(2, 2)
point_on_boundary = Point(1, 0)
```

### 10.4.1. Mối quan hệ chứa


```python
# Mối quan hệ chứa
print(f"Hình tròn chứa hình vuông: {circle.contains(square_small)}")
print(f"Hình tròn chứa điểm bên trong: {circle.contains(point_inside)}")
print(f"Hình tròn chứa điểm bên ngoài: {circle.contains(point_outside)}")
print(f"Hình vuông nằm trong hình tròn: {square_small.within(circle)}")
```

### 10.4.2. Mối quan hệ giao nhau


```python
# Mối quan hệ giao nhau
print(f"Hình tròn giao với đường qua tâm: {circle.intersects(line_through)}")
print(f"Hình tròn giao với đường bên ngoài: {circle.intersects(line_outside)}")
print(f"Hình tròn giao với hình vuông: {circle.intersects(square_small)}")
```

### 10.4.3. Mối quan hệ biên


```python
# Mối quan hệ biên
print(f"Điểm trên biên chạm hình tròn: {point_on_boundary.touches(circle)}")
print(f"Điểm bên trong chạm hình tròn: {point_inside.touches(circle)}")
print(f"Đường qua tâm cắt hình tròn: {line_through.crosses(circle)}")
```

### 10.4.4. Mối quan hệ khoảng cách


```python
# Mối quan hệ khoảng cách
print(f"Khoảng cách từ điểm ngoài đến hình tròn: {point_outside.distance(circle):.3f}")
print(f"Khoảng cách từ điểm trong đến hình tròn: {point_inside.distance(circle):.3f}")
print(f"Khoảng cách giữa hình tròn và hình vuông: {circle.distance(square_small):.3f}")
```

### 10.4.5. Mối quan hệ topo


```python
# Mối quan hệ topo (mô hình DE-9IM)
print(f"Hình tròn bằng chính nó: {circle.equals(circle)}")
print(f"Các hình học tách rời (không chạm): {circle.disjoint(point_outside)}")
print(f"Hình vuông bao phủ điểm bên trong: {square_small.covers(Point(0, 0))}")
```

### 10.4.6. Mối quan hệ chồng lấp


```python
# Kiểm tra mối quan hệ chồng lấp
overlap_square = Polygon([(0.5, -0.5), (1.5, -0.5), (1.5, 0.5), (0.5, 0.5)])
print(f"Hình tròn chồng lấp với hình vuông dịch chuyển: {circle.overlaps(overlap_square)}")
```

## 10.5. Phép toán hình học

Thực hiện các phép toán như đệm, giao, hợp và hiệu.


```python
# Tạo hình học cơ sở để thao tác
poly1 = Polygon([(0, 0), (2, 0), (2, 2), (0, 2)])  # Hình vuông
poly2 = Polygon([(1, 1), (3, 1), (3, 3), (1, 3)])  # Hình vuông chồng lấp
line = LineString([(0.5, 0.5), (2.5, 2.5)])        # Đường chéo

# 1. PHÉP TOÁN ĐỆM
point_buffer = Point(0, 0).buffer(1)
line_buffer = line.buffer(0.2)
polygon_buffer_out = poly1.buffer(0.5)    # Đệm ra ngoài
polygon_buffer_in = poly1.buffer(-0.2)    # Đệm vào trong (xói mòn)

print(f"Diện tích đệm điểm: {point_buffer.area:.3f}")
print(f"Diện tích đệm đường: {line_buffer.area:.3f}")
print(f"Diện tích đa giác gốc: {poly1.area}")
print(f"Diện tích đa giác sau đệm: {polygon_buffer_out.area:.3f}")
print(f"Diện tích đa giác sau xói mòn: {polygon_buffer_in.area:.3f}")

# 2. PHÉP TOÁN TẬP HỢP
intersection = poly1.intersection(poly2)
union = poly1.union(poly2)
difference = poly1.difference(poly2)
symmetric_difference = poly1.symmetric_difference(poly2)

print(f"Diện tích đa giác 1: {poly1.area}")
print(f"Diện tích đa giác 2: {poly2.area}")
print(f"Diện tích phần giao: {intersection.area}")
print(f"Diện tích phần hợp: {union.area}")
print(f"Diện tích hiệu (đa giác1 - đa giác2): {difference.area}")
print(f"Diện tích hiệu đối xứng: {symmetric_difference.area}")

# 3. BAO LỒI
# Tạo một số điểm ngẫu nhiên
np.random.seed(42)
random_points = [Point(x, y) for x, y in np.random.random((10, 2)) * 4 - 2]
multipoint = MultiPoint(random_points)
convex_hull = multipoint.convex_hull

print(f"Số điểm ngẫu nhiên: {len(random_points)}")
print(f"Diện tích bao lồi: {convex_hull.area:.3f}")
print(f"Loại bao lồi: {convex_hull.geom_type}")

# 4. ĐƠN GIẢN HÓA
# Tạo đa giác phức tạp với nhiều điểm
angles = np.linspace(0, 2*np.pi, 50)
noise = np.random.random(50) * 0.1  # Thêm một chút nhiễu
complex_coords = [(np.cos(a) + n, np.sin(a) + n) for a, n in zip(angles, noise)]
complex_polygon = Polygon(complex_coords)

simplified = complex_polygon.simplify(0.1, preserve_topology=True)
print(f"Số điểm đa giác gốc: {len(complex_polygon.exterior.coords)}")
print(f"Số điểm đa giác đơn giản: {len(simplified.exterior.coords)}")
print(f"Diện tích gốc: {complex_polygon.area:.3f}")
print(f"Diện tích đơn giản: {simplified.area:.3f}")

# 5. HÌNH BAO VÀ HÌNH HỘP BÀO TỐI THIỂU
envelope = multipoint.envelope
minimum_rotated_rectangle = multipoint.minimum_rotated_rectangle
print(f"Hình bao (hộp thẳng hàng trục): {envelope.geom_type}")
print(f"Diện tích hình bao: {envelope.area:.3f}")
print(f"Diện tích hình chữ nhật xoay tối thiểu: {minimum_rotated_rectangle.area:.3f}")
```

## 10.6. Phép biến đổi hình học

Áp dụng các phép biến đổi như xoay, co giãn và tịnh tiến cho hình học.


```python
# Tạo hình học cơ sở để biến đổi
original_polygon = Polygon([(0, 0), (2, 0), (2, 1), (0, 1)])  # Hình chữ nhật
```

### 10.6.1. Phép tịnh tiến


```python
# TỊNH TIẾN (Translation)
translated = affinity.translate(original_polygon, xoff=3, yoff=2)
print(f"Tịnh tiến theo x=3, y=2")
print(f"Tâm hình gốc: ({original_polygon.centroid.x:.1f}, {original_polygon.centroid.y:.1f})")
print(f"Tâm sau tịnh tiến: ({translated.centroid.x:.1f}, {translated.centroid.y:.1f})")
```

### 10.6.2. Phép xoay


```python
# XOAY (Rotation)
rotated_45 = affinity.rotate(original_polygon, 45, origin='center')  # Xoay 45 độ quanh tâm
rotated_90 = affinity.rotate(original_polygon, 90, origin=(0, 0))   # Xoay 90 độ quanh gốc
```

### 10.6.3. Phép scaling 


```python
# CO GIÃN (Scaling)
scaled_up = affinity.scale(original_polygon, xfact=2, yfact=1.5)  # Phóng to x2, y1.5
scaled_down = affinity.scale(original_polygon, xfact=0.5, yfact=0.5) # Thu nhỏ 0.5
print(f"Diện tích gốc: {original_polygon.area}")
print(f"Diện tích phóng to (2x, 1.5y): {scaled_up.area}")
print(f"Diện tích thu nhỏ (0.5x, 0.5y): {scaled_down.area}")
```

### 10.6.4. Phép biến dạng (Skew/shear)


```python
# BIẾN DẠNG (Skew/Shear)
skewed = affinity.skew(original_polygon, xs=15, ys=0)  # Nghiêng 15 độ theo trục x
print(f"Biến dạng nghiêng 15° theo trục x")
print(f"Diện tích sau biến dạng: {skewed.area:.3f}")
```

### 10.6.5. Phép đối xứng


```python
# phép đối xứng qua trục y (scale với xfact=-1)
reflected = affinity.scale(original_polygon, xfact=-1, yfact=1, origin=(1, 0))
print(f"Đối xứng qua trục y")
print(f"Tâm sau đối xứng: ({reflected.centroid.x:.1f}, {reflected.centroid.y:.1f})")
```


```python
# Trực quan hóa các phép biến đổi
fig, axes = plt.subplots(2, 4, figsize=(16, 8))

def plot_polygon(ax, poly, title, color='blue', alpha=0.5):
    """Hàm phụ trợ để vẽ đa giác"""
    x, y = poly.exterior.xy
    ax.fill(x, y, alpha=alpha, fc=color, ec='black', linewidth=1)
    ax.set_title(title)
    ax.grid(True, alpha=0.3)

# Vẽ tất cả các phép biến đổi
geometries = [
    (original_polygon, 'Hình gốc', 'blue'),
    (translated, 'Tịnh tiến', 'red'),
    (rotated_45, 'Xoay 45°', 'green'),
    (scaled_up, 'Phóng to', 'orange'),
    (scaled_down, 'Thu nhỏ', 'purple'),
    (skewed, 'Biến dạng', 'brown'),
    (reflected, 'Đối xứng Y', 'pink'),
]

for i, (geom, title, color) in enumerate(geometries):
    ax = axes[i // 4, i % 4]
    plot_polygon(ax, geom, title, color)
    
    # Thiết lập giới hạn trục phù hợp cho mỗi biểu đồ
    bounds = geom.bounds
    margin = 0.5
    ax.set_xlim(bounds[0] - margin, bounds[2] + margin)
    ax.set_ylim(bounds[1] - margin, bounds[3] + margin)

plt.tight_layout()
plt.show()
```

## 10.7. Đa hình học và tập hợp

Làm việc với các tập hợp đối tượng hình học.

### 10.7.1. Nhiều điểm (MULTIPOINT)


```python
# Tạo tập hợp nhiều điểm
points_list = [Point(0, 0), Point(1, 1), Point(2, 0), Point(1, -1)]
multipoint = MultiPoint(points_list)
# cach khachhác nhau để tạo MultiPoint
# multipoint = MultiPoint([(0, 0), (1, 1), (2, 0), (1, -1)])
print(f"Số điểm trong MultiPoint: {len(multipoint.geoms)}")
print(f"Giới hạn MultiPoint: {multipoint.bounds}")
print(f"Diện tích bao lồi: {multipoint.convex_hull.area:.3f}")

# Truy cập từng điểm
for i, point in enumerate(multipoint.geoms):
    print(f"Điểm {i+1}: ({point.x}, {point.y})")
```

    Số điểm trong MultiPoint: 4
    Giới hạn MultiPoint: (0.0, -1.0, 2.0, 1.0)
    Diện tích bao lồi: 2.000
    Điểm 1: (0.0, 0.0)
    Điểm 2: (1.0, 1.0)
    Điểm 3: (2.0, 0.0)
    Điểm 4: (1.0, -1.0)
    


```python
# Trực quan hóa các đa hình học
fig, ax = plt.subplots(1, 1, figsize=(10, 6))

# 1. MultiPoint
x_coords = [p.x for p in multipoint.geoms]
y_coords = [p.y for p in multipoint.geoms]
ax.scatter(x_coords, y_coords, c='red', s=100, alpha=0.7)
# Vẽ bao lồi
hull_x, hull_y = multipoint.convex_hull.exterior.xy
ax.plot(hull_x, hull_y, 'r--', alpha=0.5)
ax.set_title('MultiPoint với bao lồi')
ax.grid(True, alpha=0.3)
```


    
![png](output_46_0.png)
    


### 10.7.2. Nhiều đường (MultiLineString)


```python
# Tạo nhiều đường
line1 = LineString([(0, 0), (1, 1), (2, 1)])
line2 = LineString([(0, 2), (1, 3), (2, 2)])
line3 = LineString([(3, 0), (4, 1), (5, 0)])
multiline = MultiLineString([line1, line2, line3])

print(f"Số đường trong MultiLineString: {len(multiline.geoms)}")
total_length = sum(line.length for line in multiline.geoms)
print(f"Tổng độ dài các đường: {total_length:.3f}")
print(f"Độ dài từ thuộc tính: {multiline.length:.3f}")
```

    Số đường trong MultiLineString: 3
    Tổng độ dài các đường: 8.071
    Độ dài từ thuộc tính: 8.071
    


```python
# Trực quan hóa các đa hình học
fig, ax = plt.subplots(1, 1, figsize=(10, 6))
colors = ['blue', 'green', 'orange']
for i, line in enumerate(multiline.geoms):
    x, y = line.xy
    ax.plot(x, y, color=colors[i], linewidth=2, alpha=0.7, label=f'Đường {i+1}')
ax.set_title('MultiLineString')
ax.legend()
ax.grid(True, alpha=0.3)
```


    
![png](output_49_0.png)
    


### 10.7.3. Nhiều đa giác (MULTIPOLYGON)


```python
# Tạo nhiều đa giác
poly1 = Polygon([(0, 0), (1, 0), (1, 1), (0, 1)])
poly2 = Polygon([(2, 0), (3, 0), (3, 1), (2, 1)])
poly3 = Polygon([(0, 2), (2, 2), (2, 4), (0, 4)])
multipolygon = MultiPolygon([poly1, poly2, poly3])

print(f"Số đa giác trong MultiPolygon: {len(multipolygon.geoms)}")
total_area = sum(poly.area for poly in multipolygon.geoms)
print(f"Tổng diện tích các đa giác: {total_area}")
print(f"Diện tích từ thuộc tính: {multipolygon.area}")
```

    Số đa giác trong MultiPolygon: 3
    Tổng diện tích các đa giác: 6.0
    Diện tích từ thuộc tính: 6.0
    


```python
fig, ax = plt.subplots(1, 1, figsize=(10, 6))
colors = ['lightblue', 'lightcoral', 'lightgreen']
for i, poly in enumerate(multipolygon.geoms):
    x, y = poly.exterior.xy
    ax.fill(x, y, alpha=0.5, fc=colors[i], ec='black', linewidth=1, label=f'Đa giác {i+1}')
ax.set_title('MultiPolygon')
ax.legend()
ax.grid(True, alpha=0.3)
```


    
![png](output_52_0.png)
    


### 10.7.4. Tập hợp geometry hỗn hợp (GEOMETRYCOLLECTION)


```python
# Tạo tập hợp các loại hình học khác nhau
mixed_geoms = [
    Point(0, 0),
    LineString([(1, 0), (2, 1), (3, 0)]),
    Polygon([(4, 0), (5, 0), (5, 1), (4, 1)])
]

geom_collection = GeometryCollection(mixed_geoms)

print(f"Số hình học trong GeometryCollection: {len(geom_collection.geoms)}")
for i, geom in enumerate(geom_collection.geoms):
    print(f"Hình học {i+1}: {geom.geom_type}")
```

    Số hình học trong GeometryCollection: 3
    Hình học 1: Point
    Hình học 2: LineString
    Hình học 3: Polygon
    Loại kết quả sau union: MultiPolygon
    Buffer MultiPoint tạo ra: MultiPolygon
    Diện tích buffer: 3.137
    


```python
# Trực quan hóa các đa hình học
fig, ax = plt.subplots(1, 1, figsize=(10, 6))
for i, geom in enumerate(geom_collection.geoms):
    if geom.geom_type == 'Point':
        ax.scatter(geom.x, geom.y, c='red', s=100, label='Point')
    elif geom.geom_type == 'LineString':
        x, y = geom.xy
        ax.plot(x, y, 'b-', linewidth=2, label='LineString')
    elif geom.geom_type == 'Polygon':
        x, y = geom.exterior.xy
        ax.fill(x, y, alpha=0.3, fc='lightgreen', ec='green', label='Polygon')
ax.set_title('GeometryCollection')
ax.legend()
ax.grid(True, alpha=0.3)
```


    
![png](output_55_0.png)
    


### 10.7.5. Hợp các đa giác thành một


```python
# Hợp các đa giác riêng lẻ thành một
union_result = unary_union([poly1, poly2, poly3])
print(f"Loại kết quả sau union: {union_result.geom_type}")
```

    Loại kết quả sau union: MultiPolygon
    


```python
fig, ax = plt.subplots(1, 1, figsize=(10, 6))

if union_result.geom_type == 'Polygon':
    x, y = union_result.exterior.xy
    axes[1, 1].fill(x, y, alpha=0.5, fc='yellow', ec='black', linewidth=2)
elif union_result.geom_type == 'MultiPolygon':
    for poly in union_result.geoms:
        x, y = poly.exterior.xy
        ax.fill(x, y, alpha=0.5, fc='yellow', ec='black', linewidth=2)
ax.set_title('Union của MultiPolygon')
ax.grid(True, alpha=0.3)
```


    
![png](output_58_0.png)
    


### 10.7.6. Tạo buffer cho nhiều điểm 


```python
# Tạo buffer cho MultiPoint
buffered_multipoint = multipoint.buffer(0.5)
print(f"Buffer MultiPoint tạo ra: {buffered_multipoint.geom_type}")
print(f"Diện tích buffer: {buffered_multipoint.area:.3f}")
```


```python
# Trực quan hóa buffer
fig, ax = plt.subplots(1, 1, figsize=(10, 6))
if buffered_multipoint.geom_type == 'Polygon':
    x, y = buffered_multipoint.exterior.xy
    ax.fill(x, y, alpha=0.3, fc='purple', ec='black', linewidth=2)
elif buffered_multipoint.geom_type == 'MultiPolygon':
    for poly in buffered_multipoint.geoms:
        x, y = poly.exterior.xy
        ax.fill(x, y, alpha=0.3, fc='purple', ec='black', linewidth=2)
# Vẽ các điểm gốc
ax.scatter(x_coords, y_coords, c='red', s=50, zorder=5)
ax.set_title('Buffer của MultiPoint')
ax.grid(True, alpha=0.3)
```


    
![png](output_61_0.png)
    


## 10.8. Ứng dụng: Phân tích vị trí cửa hàng


```python
# Tạo dữ liệu mô phỏng: các cửa hàng trong thành phố
np.random.seed(123)

# Cụm 1: Khu vực trung tâm
center_stores = [Point(x, y) for x, y in np.random.normal([5, 5], [1, 1], (8, 2))]

# Cụm 2: Khu vực ngoại ô
suburb_stores = [Point(x, y) for x, y in np.random.normal([10, 2], [1.5, 0.8], (6, 2))]

# Cửa hàng riêng lẻ
isolated_stores = [Point(2, 8), Point(8, 9), Point(12, 7)]

# Tổng hợp tất cả cửa hàng
all_stores = center_stores + suburb_stores + isolated_stores
multipoint_stores = MultiPoint(all_stores)

print(f"Tổng số cửa hàng: {len(all_stores)}")
print(f"Giới hạn khu vực: {multipoint_stores.bounds}")

# Phân tích khu vực phủ sóng
coverage_area = multipoint_stores.buffer(1.5)  # Bán kính phục vụ 1.5km
service_hull = multipoint_stores.convex_hull      # Vùng bao phủ tổng thể

print(f"Diện tích phủ sóng (buffer): {coverage_area.area:.2f} km²")
print(f"Diện tích bao lồi: {service_hull.area:.2f} km²")

# Tìm trung tâm hình học
centroid = multipoint_stores.centroid
print(f"Trung tâm hình học: ({centroid.x:.2f}, {centroid.y:.2f})")

# Phân tích mật độ
density = len(all_stores) / service_hull.area
print(f"Mật độ cửa hàng: {density:.2f} cửa hàng/km²")

# Tìm cặp cửa hàng gần nhất
from itertools import combinations
min_distance = float('inf')
closest_pair = None

for store1, store2 in combinations(all_stores, 2):
    distance = store1.distance(store2)
    if distance < min_distance:
        min_distance = distance
        closest_pair = (store1, store2)

print(f"Khoảng cách gần nhất giữa hai cửa hàng: {min_distance:.2f} km")

# Trực quan hóa phân tích
fig, ax = plt.subplots(1, 1, figsize=(12, 8))

# Vẽ vùng phủ sóng
if coverage_area.geom_type == 'Polygon':
    x, y = coverage_area.exterior.xy
    ax.fill(x, y, alpha=0.2, fc='lightblue', ec='blue', linewidth=1, label='Vùng phủ sóng')
elif coverage_area.geom_type == 'MultiPolygon':
    for poly in coverage_area.geoms:
        x, y = poly.exterior.xy
        ax.fill(x, y, alpha=0.2, fc='lightblue', ec='blue', linewidth=1)

# Vẽ bao lồi
hull_x, hull_y = service_hull.exterior.xy
ax.plot(hull_x, hull_y, 'r--', linewidth=2, alpha=0.7, label='Bao lồi')

# Vẽ các cửa hàng theo cụm
center_x = [p.x for p in center_stores]
center_y = [p.y for p in center_stores]
suburb_x = [p.x for p in suburb_stores]
suburb_y = [p.y for p in suburb_stores]
isolated_x = [p.x for p in isolated_stores]
isolated_y = [p.y for p in isolated_stores]

ax.scatter(center_x, center_y, c='red', s=100, alpha=0.8, label='Cụm trung tâm')
ax.scatter(suburb_x, suburb_y, c='green', s=100, alpha=0.8, label='Cụm ngoại ô')
ax.scatter(isolated_x, isolated_y, c='orange', s=100, alpha=0.8, label='Cửa hàng riêng lẻ')

# Vẽ trung tâm hình học
ax.scatter(centroid.x, centroid.y, c='purple', s=200, marker='*', 
          label=f'Trung tâm ({centroid.x:.1f}, {centroid.y:.1f})')

# Vẽ cặp gần nhất
if closest_pair:
    ax.plot([closest_pair[0].x, closest_pair[1].x], 
           [closest_pair[0].y, closest_pair[1].y], 
           'k-', linewidth=3, alpha=0.7, label=f'Gần nhất: {min_distance:.2f}km')

ax.set_title('Phân tích không gian cửa hàng')
ax.legend()
ax.grid(True, alpha=0.3)
ax.set_xlabel('Khoảng cách (km)')
ax.set_ylabel('Khoảng cách (km)')

plt.show()
```

## Tóm tắt

Bạn đã hoàn thành Bài 1 và học được Shapely - thư viện nền tảng cho các phép toán hình học trong Python GIS.

### Các khái niệm chính đã nắm vững:
- ✅ **Geometric objects**: Point, LineString, Polygon và các đối tượng hình học cơ bản
- ✅ **Geometric properties**: area, length, bounds, centroid và các thuộc tính không gian
- ✅ **Spatial relationships**: contains(), intersects(), touches(), within() cho phân tích quan hệ
- ✅ **Geometric operations**: buffer(), intersection(), union(), difference() cho xử lý không gian
- ✅ **Transformations**: rotate(), scale(), translate() cho biến đổi hình học
- ✅ **Multi-geometries**: MultiPoint, MultiLineString, MultiPolygon cho dữ liệu phức tạp
- ✅ **Ứng dụng thực tế**: Phân tích vị trí cửa hàng, tính toán vùng phủ sóng, clustering

### Kỹ năng bạn có thể áp dụng:
- Tạo và thao tác các đối tượng hình học cho bài toán GIS thực tế
- Thực hiện phân tích không gian như buffer zones, intersection analysis
- Kiểm tra mối quan hệ địa lý giữa các features trong datasets
- Tối ưu hóa performance cho việc xử lý big geospatial data
- Chuẩn bị nền tảng vững chắc cho GeoPandas, Fiona và các thư viện GIS nâng cao
