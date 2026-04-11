# Bài 17: RioXArray - Geospatial Extensions cho XArray

RioXArray là thư viện kết hợp sức mạnh của XArray và Rasterio, mang geospatial superpowers đến cho multi-dimensional arrays.

## 17.1. Mục tiêu học tập

Sau khi hoàn thành bài học này, bạn sẽ có thể:

- **Tạo geospatial DataArrays** với CRS và coordinate systems đầy đủ
- **Xử lý file I/O** cho nhiều raster formats (GeoTIFF, NetCDF, HDF5)
- **Quản lý coordinate reference systems** và thực hiện transformations
- **Thực hiện spatial operations** như clipping, masking, và buffering
- **Resampling và interpolation** cho grid data với spatial awareness
- **Visualize geospatial data** với built-in plotting capabilities  
- **Optimize performance** cho big geospatial datasets với chunking
- **Tích hợp với ecosystem** (geopandas, rasterio, matplotlib)


```python
# ==========================================
# SETUP VÀ IMPORT LIBRARIES
# ==========================================

import numpy as np
import pandas as pd
import xarray as xr
import rioxarray as rxr
import rasterio
from rasterio.crs import CRS
from rasterio.transform import from_bounds
import matplotlib.pyplot as plt
import time
import os
import warnings
warnings.filterwarnings('ignore')

# Optional: GeoPandas cho vector data
try:
    import geopandas as gpd
    from shapely.geometry import box, Point, Polygon
    GEOPANDAS_AVAILABLE = True
except ImportError:
    GEOPANDAS_AVAILABLE = False
    print("⚠️ GeoPandas not available - some examples will be skipped")

print("🚀 RioXArray Environment Setup Complete!")
print(f"   📦 RioXArray version: {rxr.__version__}")
print(f"   📦 XArray version: {xr.__version__}")
print(f"   📦 Rasterio version: {rasterio.__version__}")
print(f"   🗺️ GeoPandas available: {GEOPANDAS_AVAILABLE}")
```

## 17.2. Tạo Geospatial DataArrays

RioXArray cho phép tạo XArray DataArrays với thông tin geospatial đầy đủ. 

### 17.2.1. Tạo rioxarray dataarray và datasets


```python

```


```python
# ==========================================
# 1A. TẠO GEOSPATIAL DATAARRAY TỪ SYNTHETIC DATA
# ==========================================

print("🗺️ 1A. TẠO GEOSPATIAL DATAARRAY TỪ SYNTHETIC DATA:")

# Vietnam bounding box (approximate)
VIETNAM_BOUNDS = {
    'west': 102.14,   # Tây nhất
    'east': 109.46,   # Đông nhất  
    'south': 8.18,    # Nam nhất
    'north': 23.39    # Bắc nhất
}

# Tạo coordinate grids
print("   📐 Creating coordinate grids for Vietnam:")
lons = np.linspace(VIETNAM_BOUNDS['west'], VIETNAM_BOUNDS['east'], 100)
lats = np.linspace(VIETNAM_BOUNDS['south'], VIETNAM_BOUNDS['north'], 150)
lon_grid, lat_grid = np.meshgrid(lons, lats)

# Synthetic elevation data (DEM-like)
np.random.seed(42)
# Tạo pattern giống địa hình VN: cao ở Tây Bắc, thấp ở đồng bằng
elevation_data = (
    1000 * (lat_grid - VIETNAM_BOUNDS['south']) / (VIETNAM_BOUNDS['north'] - VIETNAM_BOUNDS['south']) +  # Gradient Bắc-Nam
    500 * (VIETNAM_BOUNDS['east'] - lon_grid) / (VIETNAM_BOUNDS['east'] - VIETNAM_BOUNDS['west']) +      # Gradient Đông-Tây
    200 * np.random.random(lat_grid.shape)  # Noise
)
elevation_data = np.maximum(0, elevation_data)  # Không có độ cao âm

print(f"   🏔️ Elevation range: {elevation_data.min():.1f}m - {elevation_data.max():.1f}m")
print(f"   📏 Grid shape: {elevation_data.shape} (lat x lon)")
print(f"   🌍 Spatial extent: {VIETNAM_BOUNDS}")
```


```python
# ==========================================
# 1B. CHUYỂN ĐỔI THÀNH GEOSPATIAL DATAARRAY
# ==========================================

print("\n🗺️ 1B. CHUYỂN ĐỔI THÀNH GEOSPATIAL DATAARRAY:")

# Tạo basic XArray DataArray
elevation_da = xr.DataArray(
    elevation_data,
    dims=['latitude', 'longitude'],
    coords={
        'latitude': lats,
        'longitude': lons
    },
    attrs={
        'long_name': 'Elevation above sea level',
        'units': 'meters',
        'description': 'Synthetic DEM for Vietnam',
        'source': 'Generated for tutorial'
    }
)

print("   📊 Basic DataArray created:")
print(f"   Shape: {elevation_da.shape}")
print(f"   Dimensions: {elevation_da.dims}")

# Thêm CRS information với RioXArray
print("\n   🌍 Adding CRS information:")
elevation_da = elevation_da.rio.write_crs("EPSG:4326")  # WGS84 Geographic
print(f"   CRS: {elevation_da.rio.crs}")
print(f"   Spatial dims: {elevation_da.rio.x_dim}, {elevation_da.rio.y_dim}")

# Kiểm tra geospatial properties
print("\n   📏 Geospatial properties:")
print(f"   Bounds: {elevation_da.rio.bounds()}")
print(f"   Shape: {elevation_da.rio.shape}")
print(f"   Resolution: {elevation_da.rio.resolution()}")
print(f"   Transform: {elevation_da.rio.transform()}")

print("   ✅ Geospatial DataArray ready for analysis!")
```

### 17.2.2. Các hàm cơ bản


```python
# ==========================================
# 1C. DEMO CÁC FUNCTIONALITIES CƠ BẢN
# ==========================================

print("\n🔧 1C. DEMO CÁC FUNCTIONALITIES CƠ BẢN:")

# Demo 1: Kiểm tra geospatial properties
print("\n   📏 Demo 1: Geospatial Properties Inspection")
print(f"   CRS: {elevation_da.rio.crs}")
print(f"   Bounds: {elevation_da.rio.bounds()}")
print(f"   Width x Height: {elevation_da.rio.width} x {elevation_da.rio.height}")
print(f"   Resolution: {elevation_da.rio.resolution()}")
print(f"   Transform matrix:")
transform = elevation_da.rio.transform()
for i in range(2):
    print(f"     [{transform[i*3]:.6f}, {transform[i*3+1]:.6f}, {transform[i*3+2]:.2f}]")

# Demo 2: Coordinate manipulation
print("\n   🌍 Demo 2: Coordinate System Information")
print(f"   Spatial dimensions: x={elevation_da.rio.x_dim}, y={elevation_da.rio.y_dim}")
print(f"   X coordinates range: {elevation_da.rio.x_dim} from {elevation_da.coords[elevation_da.rio.x_dim].min().values:.2f} to {elevation_da.coords[elevation_da.rio.x_dim].max().values:.2f}")
print(f"   Y coordinates range: {elevation_da.rio.y_dim} from {elevation_da.coords[elevation_da.rio.y_dim].min().values:.2f} to {elevation_da.coords[elevation_da.rio.y_dim].max().values:.2f}")

# Demo 3: NoData handling
print("\n   🔍 Demo 3: NoData Handling")
# Add some NoData values for demonstration
elevation_with_nodata = elevation_da.copy()
# Set some high elevation areas as NoData
elevation_with_nodata = elevation_with_nodata.where(elevation_with_nodata < 1200)
nodata_count = elevation_with_nodata.isnull().sum().values
valid_count = elevation_with_nodata.count().values
print(f"   Total pixels: {elevation_with_nodata.size}")
print(f"   Valid pixels: {valid_count}")
print(f"   NoData pixels: {nodata_count}")
print(f"   NoData percentage: {(nodata_count/elevation_with_nodata.size)*100:.1f}%")

# Demo 4: Spatial indexing
print("\n   📍 Demo 4: Point and Area Sampling")
# Sample specific points (major Vietnamese cities)
sampling_points = {
    'Hanoi': (105.8342, 21.0285),
    'Ho_Chi_Minh': (106.6297, 10.8231),
    'Da_Nang': (108.2022, 16.0544),
    'Hue': (107.5909, 16.4637)
}

for city, (lon, lat) in sampling_points.items():
    # Sample elevation at city location
    city_elevation = elevation_da.sel(longitude=lon, latitude=lat, method='nearest')
    print(f"   {city.replace('_', ' ')}: Elevation = {city_elevation.values:.0f}m at ({lon}°E, {lat}°N)")

# Demo 5: Attribute management
print("\n   📋 Demo 5: Metadata and Attributes")
print(f"   Original attributes: {elevation_da.attrs}")

# Add more detailed attributes
elevation_da.attrs.update({
    'creation_date': '2023-11-11',
    'author': 'Vietnam GIS Tutorial',
    'data_type': 'elevation',
    'vertical_datum': 'Mean Sea Level',
    'horizontal_accuracy': '±30 meters',
    'vertical_accuracy': '±10 meters'
})

print(f"   Enhanced attributes count: {len(elevation_da.attrs)}")
for key, value in elevation_da.attrs.items():
    print(f"     {key}: {value}")

print("   ✅ Basic functionalities demonstrated!")
```


```python
# ==========================================\n# 1D. PRACTICAL DEMO - WORKING WITH REAL COORDINATES\n# ==========================================\n\nprint(\"🔧 1D. PRACTICAL DEMO - WORKING WITH REAL COORDINATES:\")\n\n# Create practical example with realistic Vietnam data\nprint(\"\\n   📍 Creating elevation profiles for major Vietnamese routes:\")\n\n# Define major transportation routes in Vietnam\nroutes = {\n    'North_South_Highway': {\n        'start': (105.85, 21.03),  # Hanoi\n        'end': (106.70, 10.82),    # Ho Chi Minh City\n        'description': 'Highway 1A - Main north-south route'\n    },\n    'East_West_Central': {\n        'start': (108.20, 16.05),  # Da Nang (coast)\n        'end': (106.00, 16.00),    # Towards Laos border\n        'description': 'Central Highland route'\n    },\n    'Mekong_Delta_Route': {\n        'start': (106.70, 10.82),  # Ho Chi Minh City\n        'end': (105.12, 9.18),     # Ca Mau (southernmost)\n        'description': 'Mekong Delta exploration route'\n    }\n}\n\n# Extract elevation profiles along routes\nroute_profiles = {}\nfor route_name, route_info in routes.items():\n    start_lon, start_lat = route_info['start']\n    end_lon, end_lat = route_info['end']\n    \n    # Create points along the route\n    num_points = 20\n    lons = np.linspace(start_lon, end_lon, num_points)\n    lats = np.linspace(start_lat, end_lat, num_points)\n    \n    # Sample elevation along the route\n    elevations = []\n    distances = []\n    \n    for i, (lon, lat) in enumerate(zip(lons, lats)):\n        # Get elevation at this point\n        try:\n            elev = elevation_da.sel(longitude=lon, latitude=lat, method='nearest').values\n            elevations.append(float(elev))\n            \n            # Calculate cumulative distance (approximate)\n            if i == 0:\n                distances.append(0)\n            else:\n                # Simple distance calculation (degrees to km approximation)\n                prev_lon, prev_lat = lons[i-1], lats[i-1]\n                dist_deg = np.sqrt((lon - prev_lon)**2 + (lat - prev_lat)**2)\n                dist_km = dist_deg * 111  # Rough conversion\n                distances.append(distances[-1] + dist_km)\n        except:\n            elevations.append(np.nan)\n            distances.append(distances[-1] if distances else 0)\n    \n    route_profiles[route_name] = {\n        'distances': distances,\n        'elevations': elevations,\n        'coordinates': list(zip(lons, lats)),\n        'info': route_info\n    }\n    \n    # Print route statistics\n    valid_elevs = [e for e in elevations if not np.isnan(e)]\n    if valid_elevs:\n        min_elev = min(valid_elevs)\n        max_elev = max(valid_elevs)\n        avg_elev = sum(valid_elevs) / len(valid_elevs)\n        total_distance = distances[-1]\n        \n        print(f\"\\n   🛣️ {route_name.replace('_', ' ')} ({route_info['description']}):\")\n        print(f\"     Route length: ~{total_distance:.0f} km\")\n        print(f\"     Elevation range: {min_elev:.0f}m to {max_elev:.0f}m\")\n        print(f\"     Average elevation: {avg_elev:.0f}m\")\n        print(f\"     Elevation gain: {max_elev - min_elev:.0f}m\")\n\n# Create elevation profile plot\nfig, axes = plt.subplots(len(routes), 1, figsize=(12, 8))\nif len(routes) == 1:\n    axes = [axes]\n\nfor i, (route_name, profile) in enumerate(route_profiles.items()):\n    ax = axes[i]\n    \n    # Plot elevation profile\n    valid_indices = [j for j, e in enumerate(profile['elevations']) if not np.isnan(e)]\n    valid_distances = [profile['distances'][j] for j in valid_indices]\n    valid_elevations = [profile['elevations'][j] for j in valid_indices]\n    \n    if valid_distances:\n        ax.plot(valid_distances, valid_elevations, 'b-o', linewidth=2, markersize=4)\n        ax.fill_between(valid_distances, 0, valid_elevations, alpha=0.3)\n        ax.set_title(f\"Elevation Profile: {route_name.replace('_', ' ')}\")\n        ax.set_xlabel('Distance (km)')\n        ax.set_ylabel('Elevation (m)')\n        ax.grid(True, alpha=0.3)\n        ax.set_ylim(0, max(valid_elevations) * 1.1 if valid_elevations else 1000)\n\nplt.tight_layout()\nplt.show()\n\n# Demonstrate spatial queries\nprint(\"\\n   🔍 Spatial Query Examples:\")\n\n# Find highest point in each region\nfor region_name, bounds in {\n    'Northern Mountains': (102, 104, 21, 23),\n    'Central Highlands': (107, 109, 14, 16), \n    'Mekong Delta': (104, 106, 9, 11)\n}.items():\n    west, east, south, north = bounds\n    \n    # Clip region\n    region_data = elevation_da.sel(\n        longitude=slice(west, east),\n        latitude=slice(south, north)\n    )\n    \n    if region_data.size > 0:\n        # Find maximum elevation\n        max_elev_data = region_data.where(region_data == region_data.max(), drop=True)\n        \n        if max_elev_data.size > 0:\n            max_coords = {\n                'lat': float(max_elev_data.latitude.values.flat[0]),\n                'lon': float(max_elev_data.longitude.values.flat[0]),\n                'elev': float(max_elev_data.values.flat[0])\n            }\n            \n            print(f\"     {region_name}: {max_coords['elev']:.0f}m at ({max_coords['lon']:.2f}°E, {max_coords['lat']:.2f}°N)\")\n\nprint(\"\\n   ✅ Practical coordinate operations completed!\")"
```

## 17.3. File I/O và Format Support

RioXArray hỗ trợ đa dạng định dạng raster như `GeoTIFF`, `NetCDF`, `COG`, và `Zarr` và cho phép chunk với dữ liệu lớn (big data).

### 17.3.1. Đọc dữ liệu 


```python
# ==========================================
# 2A. LƯU VÀ ĐỌC RASTER FILES
# ==========================================

print("? 2A. LƯU VÀ ĐỌC RASTER FILES:")

# Lưu dưới các format khác nhau
print("   ? Saving elevation data in multiple formats:")

try:
    # 1. GeoTIFF format
    elevation_da.rio.to_raster("vietnam_elevation.tif")
    print("   ✅ Saved: vietnam_elevation.tif")
    
    # 2. NetCDF format với compression
    elevation_da.to_netcdf("vietnam_elevation.nc", 
                          encoding={'__xarray_dataarray_variable__': {'zlib': True, 'complevel': 6}})
    print("   ✅ Saved: vietnam_elevation.nc")
    
    # 3. Zarr format cho big data
    elevation_da.to_zarr("vietnam_elevation.zarr", mode='w')
    print("   ✅ Saved: vietnam_elevation.zarr")
    
except Exception as e:
    print(f"   ⚠️ Save error: {e}")

# Đọc trở lại và so sánh
print("\n   ? Reading back saved files:")

try:
    # Đọc GeoTIFF với rioxarray
    loaded_tif = rxr.open_rasterio("vietnam_elevation.tif").squeeze('band')
    print(f"   GeoTIFF - Shape: {loaded_tif.shape}, CRS: {loaded_tif.rio.crs}")
    
    # Đọc NetCDF với xarray
    loaded_nc = xr.open_dataarray("vietnam_elevation.nc")
    print(f"   NetCDF - Shape: {loaded_nc.shape}, Has CRS: {loaded_nc.rio.crs is not None}")
    
    # So sánh data integrity
    data_identical = np.allclose(elevation_da.values, loaded_tif.values, rtol=1e-10)
    print(f"   Data integrity: {data_identical}")
    
except Exception as e:
    print(f"   ⚠️ Read error: {e}")

print("   ✅ Multi-format I/O demonstrates RioXArray flexibility!")
```

### 17.3.2. Lưu dữ liệu 


```python

```

## 17.4. Coordinate Reference Systems (CRS)

CRS management là core functionality của RioXArray cho geospatial analysis.

### 17.4.1. Thiết lập CRS cho dữ liệu


```python

```

### 17.4.2. Chuyển đổi hệ tọa độ sử dụng `reproject`


```python

```

### 17.4.3. Chuyển đổi CRS và khớp lưới pixel sử dụng `reproject_match`


```python

```

## 17.5. Cắt raster theo vùng

RioXArray cung cấp powerful tools cho spatial operations.

### 17.5.1. Cắt raster theo vùng với `rio.clip`


```python

```

### 17.5.2. Cắt raster theo vùng với `rio.clip_box`


```python

```


```python
# ==========================================
# 3A. CRS TRANSFORMATIONS
# ==========================================

print("🌍 3A. CRS TRANSFORMATIONS:")

# Vietnam-specific CRS options
print("   📍 Vietnam CRS options:")
vietnam_crs_options = {
    'WGS84_Geographic': 'EPSG:4326',
    'VN2000_Geographic': 'EPSG:4756', 
    'UTM_48N': 'EPSG:32648',  # Covers most of Vietnam
    'UTM_49N': 'EPSG:32649',  # Eastern Vietnam
    'Web_Mercator': 'EPSG:3857'
}

for name, epsg in vietnam_crs_options.items():
    print(f"   {name}: {epsg}")

# Transform to different CRS
print("\n   🔄 Transforming to UTM 48N (projected coordinates):")

try:
    # Transform to UTM 48N for Vietnam
    utm_elevation = elevation_da.rio.reproject('EPSG:32648')
    
    print(f"   Original CRS: {elevation_da.rio.crs}")
    print(f"   Original bounds: {elevation_da.rio.bounds()}")
    print(f"   Original resolution: {elevation_da.rio.resolution()}")
    
    print(f"\n   UTM CRS: {utm_elevation.rio.crs}")
    print(f"   UTM bounds: {utm_elevation.rio.bounds()}")
    print(f"   UTM resolution: {utm_elevation.rio.resolution()}")
    print(f"   UTM shape: {utm_elevation.shape}")
    
    # Resolution comparison
    orig_res_deg = elevation_da.rio.resolution()[0]
    utm_res_m = utm_elevation.rio.resolution()[0]
    approx_deg_to_m = 111000  # Rough conversion at Vietnam latitude
    orig_res_m = orig_res_deg * approx_deg_to_m
    
    print(f"\n   Resolution comparison:")
    print(f"   Geographic: {orig_res_deg:.4f}° (~{orig_res_m:.0f}m)")
    print(f"   UTM: {utm_res_m:.0f}m")
    
except Exception as e:
    print(f"   ⚠️ CRS transformation failed: {e}")

print("   ✅ CRS transformations enable proper distance/area calculations!")
```


```python
# ==========================================
# 3B. DEMO ADVANCED CRS OPERATIONS
# ==========================================

print("\n🔧 3B. DEMO ADVANCED CRS OPERATIONS:")

# Demo 1: Multiple CRS transformations comparison
print("\n   🔄 Demo 1: Multiple CRS Transformations")

transformed_datasets = {}
for crs_name, crs_code in vietnam_crs_options.items():
    try:
        if crs_code != 'EPSG:4326':  # Skip original CRS
            print(f"\n     Transforming to {crs_name} ({crs_code}):")
            
            transformed = elevation_da.rio.reproject(crs_code)
            transformed_datasets[crs_name] = transformed
            
            bounds = transformed.rio.bounds()
            resolution = transformed.rio.resolution()
            
            print(f"       Shape: {transformed.shape}")
            print(f"       Bounds: ({bounds[0]:.0f}, {bounds[1]:.0f}) to ({bounds[2]:.0f}, {bounds[3]:.0f})")
            print(f"       Resolution: {resolution[0]:.0f} x {resolution[1]:.0f} units")
            print(f"       Data preserved: {not transformed.isnull().all().values}")
            
    except Exception as e:
        print(f"       ⚠️ Failed to transform to {crs_name}: {e}")

# Demo 2: Area calculations in different CRS
print("\n   📐 Demo 2: Area Calculations in Different CRS")

# Calculate pixel area for a sample region
sample_region = {
    'west': 105.0, 'east': 106.0,
    'south': 20.0, 'north': 21.0
}

print(f"\n     Sample region: {sample_region}")

for crs_name in ['WGS84_Geographic', 'UTM_48N']:
    if crs_name in transformed_datasets or crs_name == 'WGS84_Geographic':
        try:
            if crs_name == 'WGS84_Geographic':
                data = elevation_da
                unit_name = "degrees"
            else:
                data = transformed_datasets[crs_name]
                unit_name = "meters"
                
            # Get resolution
            res_x, res_y = data.rio.resolution()
            
            if crs_name == 'WGS84_Geographic':
                # Convert degrees to approximate area
                lat_center = (sample_region['north'] + sample_region['south']) / 2
                deg_to_m = 111000 * np.cos(np.radians(lat_center))
                pixel_area_m2 = abs(res_x * deg_to_m) * abs(res_y * 111000)
                pixel_area_km2 = pixel_area_m2 / 1e6
            else:
                # Direct area calculation in meters
                pixel_area_m2 = abs(res_x * res_y)
                pixel_area_km2 = pixel_area_m2 / 1e6
            
            print(f"\n     {crs_name}:")
            print(f"       Resolution: {abs(res_x):.6f} x {abs(res_y):.6f} {unit_name}")
            print(f"       Pixel area: {pixel_area_m2:.0f} m²")
            print(f"       Pixel area: {pixel_area_km2:.6f} km²")
                
        except Exception as e:
            print(f"       ⚠️ Area calculation failed for {crs_name}: {e}")

# Demo 3: Accuracy assessment across projections
print("\n   🎯 Demo 3: Transformation Accuracy Assessment")

# Create test points at Vietnam cities
test_points = {
    'Hanoi': (105.8342, 21.0285),
    'Ho_Chi_Minh': (106.6297, 10.8231),
    'Da_Nang': (108.2022, 16.0544)
}

print("\n     Coordinate transformation accuracy:")
print(f"     {'City':<12} {'Original (°)':<20} {'UTM 48N (m)':<25} {'Back to WGS84 (°)':<20}")
print(f"     {'-'*80}")

if 'UTM_48N' in transformed_datasets:
    utm_data = transformed_datasets['UTM_48N']
    
    for city, (orig_lon, orig_lat) in test_points.items():
        try:
            # Sample elevation at original coordinates
            orig_elev = elevation_da.sel(longitude=orig_lon, latitude=orig_lat, method='nearest')
            
            # Transform coordinates manually for verification
            from rasterio.warp import transform as rio_transform
            
            # Forward transform: WGS84 -> UTM
            utm_coords = rio_transform('EPSG:4326', 'EPSG:32648', [orig_lon], [orig_lat])
            utm_x, utm_y = utm_coords[0][0], utm_coords[1][0]
            
            # Sample at UTM coordinates
            utm_elev = utm_data.sel(x=utm_x, y=utm_y, method='nearest')
            
            # Reverse transform: UTM -> WGS84
            wgs_coords = rio_transform('EPSG:32648', 'EPSG:4326', [utm_x], [utm_y])
            back_lon, back_lat = wgs_coords[0][0], wgs_coords[1][0]
            
            # Calculate errors
            coord_error = np.sqrt((orig_lon - back_lon)**2 + (orig_lat - back_lat)**2) * 111000  # Convert to meters
            elev_error = abs(orig_elev.values - utm_elev.values)
            
            print(f"     {city:<12} ({orig_lon:.4f},{orig_lat:.4f})  ({utm_x:.0f},{utm_y:.0f})      ({back_lon:.4f},{back_lat:.4f})")
            print(f"     {'':>12} Coord error: {coord_error:.2f}m, Elev error: {elev_error:.1f}m")
            
        except Exception as e:
            print(f"     {city:<12} ⚠️ Accuracy test failed: {e}")

# Demo 4: CRS-aware operations
print("\n   🧮 Demo 4: CRS-aware Spatial Operations")

if 'UTM_48N' in transformed_datasets:
    utm_data = transformed_datasets['UTM_48N']
    
    # Calculate gradients (slope) in proper units
    print("\n     Slope calculation comparison:")
    
    # Geographic coordinates (degrees) - incorrect for slope
    grad_y_deg, grad_x_deg = np.gradient(elevation_da.values)
    # Convert to proper units (very approximate)
    grad_y_m = grad_y_deg / (elevation_da.rio.resolution()[1] * 111000)
    grad_x_m = grad_x_deg / (elevation_da.rio.resolution()[0] * 111000 * np.cos(np.radians(20)))
    slope_deg = np.degrees(np.arctan(np.sqrt(grad_x_m**2 + grad_y_m**2)))
    
    # UTM coordinates (meters) - correct for slope
    grad_y_utm, grad_x_utm = np.gradient(utm_data.values)
    grad_y_utm_proper = grad_y_utm / abs(utm_data.rio.resolution()[1])
    grad_x_utm_proper = grad_x_utm / abs(utm_data.rio.resolution()[0])
    slope_utm = np.degrees(np.arctan(np.sqrt(grad_x_utm_proper**2 + grad_y_utm_proper**2)))
    
    print(f"       Geographic (approximate): Mean slope = {np.nanmean(slope_deg):.2f}°")
    print(f"       UTM (proper): Mean slope = {np.nanmean(slope_utm):.2f}°")
    print(f"       Difference: {abs(np.nanmean(slope_deg) - np.nanmean(slope_utm)):.2f}°")

print("\n   ✅ Advanced CRS operations ensure spatial accuracy!")
```

## 17.6. Resampling và Interpolation

RioXArray cung cấp các phương pháp khác nhau cho việc resampling và nội suy.

### 🎯 **Resampling Methods**
- **Nearest**: Preserve original values, fast
- **Bilinear**: Smooth interpolation cho continuous data
- **Cubic**: Higher-order polynomial, smooth results
- **Average**: Mean value cho downsampling

### 📐 **Resolution Operations**
- **Upsampling**: Increase resolution (interpolation)
- **Downsampling**: Decrease resolution (aggregation)
- **Match target**: Align với existing raster grid
- **Custom grids**: Specify exact output coordinates

### 🇻🇳 **Applications**
- **Multi-scale analysis**: Combine data ở different resolutions
- **Performance optimization**: Coarsen data cho faster processing
- **Data integration**: Match resolutions từ different sources
- **Visualization**: Appropriate resolution cho display

### 17.6.1. Resampling dữ liệu raster 


```python

```

### 17.6.2. Nội suy dữ liệu trống (missing values)


```python

```

## 17.7. Tính toán các chỉ số thực vật và kết hợp

RioXArray cung cấp các công cụ làm việc với multi-band rasters như ảnh viễn thám.

### 17.7.1. Tính toán chỉ số thực vật


```python

```

### 17.7.2. Tính số tháng hạn hán


```python
# ==========================================
# 4A. SPATIAL OPERATIONS - CLIPPING VÀ MASKING
# ==========================================

print("✂️ 4A. SPATIAL OPERATIONS - CLIPPING VÀ MASKING:")

# Vietnam regions for clipping demonstration
regions = {
    'Northern_Mountains': {
        'bounds': (102.0, 21.0, 106.0, 24.0),
        'description': 'Hoàng Liên Sơn và Tây Bắc'
    },
    'Red_River_Delta': {
        'bounds': (105.0, 20.0, 107.0, 22.0),
        'description': 'Đồng bằng sông Hồng'
    },
    'Central_Coast': {
        'bounds': (107.0, 15.0, 109.5, 17.0),
        'description': 'Duyên hải miền Trung'
    },
    'Mekong_Delta': {
        'bounds': (104.5, 8.5, 107.0, 11.0),
        'description': 'Đồng bằng sông Cửu Long'
    }
}

# Clip elevation data for each region
print("   🗺️ Regional clipping demonstration:")

clipped_regions = {}
for region_name, region_info in regions.items():
    west, south, east, north = region_info['bounds']
    
    try:
        # Spatial clipping using bounds
        clipped = elevation_da.rio.clip_box(
            minx=west, miny=south, maxx=east, maxy=north
        )
        
        clipped_regions[region_name] = clipped
        
        # Calculate regional statistics
        stats = {
            'min': clipped.min().values,
            'max': clipped.max().values,
            'mean': clipped.mean().values,
            'std': clipped.std().values,
            'pixels': clipped.count().values
        }
        
        print(f"\n   📊 {region_name.replace('_', ' ')} ({region_info['description']}):")
        print(f"      Bounds: ({west}, {south}) to ({east}, {north})")
        print(f"      Shape: {clipped.shape}")
        print(f"      Elevation: {stats['min']:.0f}m - {stats['max']:.0f}m (avg: {stats['mean']:.0f}m)")
        print(f"      Valid pixels: {stats['pixels']:,}")
        
    except Exception as e:
        print(f"   ⚠️ Clipping failed for {region_name}: {e}")

# Masking demonstration
print("\n   🎭 Elevation-based masking:")

# Create elevation masks
elevation_classes = {
    'Lowlands': {'min': 0, 'max': 200, 'description': 'Đồng bằng và vùng thấp'},
    'Hills': {'min': 200, 'max': 500, 'description': 'Đồi núi thấp'},
    'Highlands': {'min': 500, 'max': 1000, 'description': 'Cao nguyên'},
    'Mountains': {'min': 1000, 'max': 2000, 'description': 'Núi cao'}
}

elevation_masks = {}
for class_name, class_info in elevation_classes.items():
    min_elev, max_elev = class_info['min'], class_info['max']
    
    # Create boolean mask
    mask = (elevation_da >= min_elev) & (elevation_da < max_elev)
    masked_data = elevation_da.where(mask)
    
    elevation_masks[class_name] = masked_data
    
    # Calculate area coverage
    valid_pixels = mask.sum().values
    total_pixels = mask.size
    coverage_percent = (valid_pixels / total_pixels) * 100
    
    print(f"\n   🏔️ {class_name} ({class_info['description']}):")
    print(f"      Elevation range: {min_elev}m - {max_elev}m")
    print(f"      Area coverage: {coverage_percent:.1f}% ({valid_pixels:,} pixels)")
    
    if valid_pixels > 0:
        mean_elev = masked_data.mean().values
        print(f"      Mean elevation in class: {mean_elev:.0f}m")

print("   ✅ Spatial clipping và masking enable focused analysis!")
```


```python
# ==========================================
# 4B. DEMO ADVANCED SPATIAL OPERATIONS
# ==========================================

print("\n? 4B. DEMO ADVANCED SPATIAL OPERATIONS:")

# Demo 1: Buffer operations around cities
print("\n   📍 Demo 1: Buffer Operations Around Cities")

if GEOPANDAS_AVAILABLE:
    # Create city points
    cities_data = {
        'name': ['Hanoi', 'Ho_Chi_Minh', 'Da_Nang', 'Can_Tho', 'Hai_Phong'],
        'longitude': [105.8342, 106.6297, 108.2022, 105.7667, 106.6881],
        'latitude': [21.0285, 10.8231, 16.0544, 10.0452, 20.8648]
    }
    
    cities_gdf = gpd.GeoDataFrame(
        cities_data,
        geometry=gpd.points_from_xy(cities_data['longitude'], cities_data['latitude']),
        crs='EPSG:4326'
    )
    
    print(f"     Created {len(cities_gdf)} city points")
    
    # Create buffers around cities (in geographic degrees, approximate)
    buffer_sizes_deg = [0.2, 0.5, 1.0]  # Approximate buffer sizes in degrees
    
    for buffer_size in buffer_sizes_deg:
        buffer_km = buffer_size * 111  # Rough conversion to km
        print(f"\n     Buffer size: {buffer_size}° (~{buffer_km:.0f}km)")
        
        # Create buffers
        city_buffers = cities_gdf.copy()
        city_buffers['geometry'] = cities_gdf.geometry.buffer(buffer_size)
        
        # Extract elevation statistics within buffers
        for idx, city_row in city_buffers.iterrows():
            city_name = city_row['name']
            city_geom = city_row['geometry']
            
            try:
                # Convert geometry to bounds for clipping
                bounds = city_geom.bounds  # (minx, miny, maxx, maxy)
                
                # Clip elevation data to buffer bounds (approximate)
                city_elevation = elevation_da.sel(
                    longitude=slice(bounds[0], bounds[2]),
                    latitude=slice(bounds[1], bounds[3])
                )
                
                if city_elevation.size > 0:
                    stats = {
                        'mean': city_elevation.mean().values,
                        'std': city_elevation.std().values,
                        'min': city_elevation.min().values,
                        'max': city_elevation.max().values
                    }
                    
                    print(f"       {city_name}: {stats['min']:.0f}-{stats['max']:.0f}m (avg: {stats['mean']:.0f}m ± {stats['std']:.0f}m)")
                    
            except Exception as e:
                print(f"       {city_name}: ⚠️ Buffer analysis failed: {e}")

else:
    print("     ⚠️ GeoPandas not available - skipping buffer operations")

# Demo 2: Watershed/drainage analysis simulation
print("\n   💧 Demo 2: Drainage Pattern Analysis")

try:
    # Find local minima (potential drainage points)
    from scipy import ndimage
    
    # Smooth elevation for better drainage analysis
    smoothed_elev = ndimage.gaussian_filter(elevation_da.values, sigma=2)
    
    # Find local minima
    local_minima = ndimage.minimum_filter(smoothed_elev, size=5) == smoothed_elev
    
    # Get coordinates of minima
    minima_coords = np.where(local_minima)
    num_minima = len(minima_coords[0])
    
    print(f"     Found {num_minima} potential drainage points")
    
    # Analyze elevation around drainage points
    if num_minima > 0:
        # Sample some drainage points for analysis
        sample_indices = np.random.choice(num_minima, min(5, num_minima), replace=False)
        
        print("     Sample drainage point analysis:")
        for i, idx in enumerate(sample_indices):
            lat_idx, lon_idx = minima_coords[0][idx], minima_coords[1][idx]
            
            # Get coordinates
            lat_val = elevation_da.latitude.values[lat_idx]
            lon_val = elevation_da.longitude.values[lon_idx]
            elev_val = elevation_da.values[lat_idx, lon_idx]
            
            # Analyze surrounding area
            window = 3  # 3x3 window around point
            lat_start = max(0, lat_idx - window)
            lat_end = min(elevation_da.shape[0], lat_idx + window + 1)
            lon_start = max(0, lon_idx - window)
            lon_end = min(elevation_da.shape[1], lon_idx + window + 1)
            
            surrounding = elevation_da.values[lat_start:lat_end, lon_start:lon_end]
            elev_diff = surrounding.mean() - elev_val
            
            print(f"       Point {i+1}: ({lon_val:.2f}°E, {lat_val:.2f}°N)")
            print(f"         Elevation: {elev_val:.0f}m")
            print(f"         Depression: {elev_diff:.0f}m below surroundings")
            
except ImportError:
    print("     ⚠️ SciPy not available - skipping drainage analysis")
except Exception as e:
    print(f"     ⚠️ Drainage analysis failed: {e}")

# Demo 3: Distance calculations
print("\n   📏 Demo 3: Distance Calculations và Proximity Analysis")

# Calculate distance from coastline (approximate using southern boundary)
print("     Distance to southern coastline (approximate):")

# Define approximate coastline as southern edge
coastline_lat = VIETNAM_BOUNDS['south'] + 0.5  # Approximate coastline

# Calculate distance to coastline for each pixel
distances_to_coast = []
sample_points = [
    (105.8, 21.0, 'Hanoi'),
    (108.2, 16.0, 'Da_Nang'), 
    (106.6, 10.8, 'Ho_Chi_Minh')
]

for lon, lat, city_name in sample_points:
    # Simple distance calculation (degrees to km)
    distance_deg = abs(lat - coastline_lat)
    distance_km = distance_deg * 111  # Rough conversion
    
    # Get elevation at this point
    city_elev = elevation_da.sel(longitude=lon, latitude=lat, method='nearest').values
    
    print(f"     {city_name}: {distance_km:.0f}km from coast, {city_elev:.0f}m elevation")

# Demo 4: Elevation profile analysis
print("\n   ? Demo 4: Elevation Profile Analysis")

# Create elevation profiles along major axes
profiles = {
    'North_South': {
        'start': (106.0, 23.0),  # North
        'end': (106.0, 9.0),     # South
        'description': 'North-South transect through center'
    },
    'East_West': {
        'start': (102.5, 16.0),  # West
        'end': (109.0, 16.0),    # East  
        'description': 'East-West transect through center'
    }
}

for profile_name, profile_info in profiles.items():
    start_lon, start_lat = profile_info['start']
    end_lon, end_lat = profile_info['end']
    
    # Create points along the profile
    num_points = 50
    if profile_name == 'North_South':
        lons = np.full(num_points, start_lon)
        lats = np.linspace(start_lat, end_lat, num_points)
    else:  # East_West
        lons = np.linspace(start_lon, end_lon, num_points)
        lats = np.full(num_points, start_lat)
    
    # Extract elevations along profile
    elevations = []
    distances = []
    
    for i, (lon, lat) in enumerate(zip(lons, lats)):
        try:
            elev = elevation_da.sel(longitude=lon, latitude=lat, method='nearest').values
            elevations.append(float(elev))
            
            # Calculate distance along profile
            if i == 0:
                distances.append(0)
            else:
                if profile_name == 'North_South':
                    dist = abs(lat - lats[0]) * 111  # Latitude distance in km
                else:
                    dist = abs(lon - lons[0]) * 111 * np.cos(np.radians(lat))  # Longitude distance
                distances.append(dist)
                
        except:
            elevations.append(np.nan)
            distances.append(distances[-1] if distances else 0)
    
    # Profile statistics
    valid_elevs = [e for e in elevations if not np.isnan(e)]
    if valid_elevs:
        profile_stats = {
            'length': distances[-1],
            'min_elev': min(valid_elevs),
            'max_elev': max(valid_elevs),
            'mean_elev': sum(valid_elevs) / len(valid_elevs),
            'elev_range': max(valid_elevs) - min(valid_elevs)
        }
        
        print(f"\n     {profile_name.replace('_', '-')} Profile ({profile_info['description']}):")
        print(f"       Length: {profile_stats['length']:.0f}km")
        print(f"       Elevation: {profile_stats['min_elev']:.0f}m - {profile_stats['max_elev']:.0f}m")
        print(f"       Average: {profile_stats['mean_elev']:.0f}m")
        print(f"       Relief: {profile_stats['elev_range']:.0f}m")

print("\n   ✅ Advanced spatial operations provide deep geographic insights!")
```


```python
# ==========================================
# 5A. RESOLUTION RESAMPLING
# ==========================================

print("📏 5A. RESOLUTION RESAMPLING:")

# Current resolution info
original_res = elevation_da.rio.resolution()
print(f"   📐 Original resolution: {original_res[0]:.4f}° x {original_res[1]:.4f}°")
print(f"   📐 Original shape: {elevation_da.shape}")

# Downsampling - reduce resolution (faster processing)
print("\n   ⬇️ Downsampling (coarsening):")

# Method 1: Simple coarsen (factor-based)
coarse_elevation = elevation_da.coarsen(latitude=3, longitude=3, boundary='trim').mean()
print(f"   Coarsened shape: {coarse_elevation.shape}")
print(f"   Coarsened resolution: ~{original_res[0]*3:.4f}° x {original_res[1]*3:.4f}°")

# Method 2: Reproject to specific resolution 
downsampled = elevation_da.rio.reproject(
    elevation_da.rio.crs,
    resolution=(original_res[0] * 2, original_res[1] * 2),  # 2x coarser
    resampling=rasterio.enums.Resampling.bilinear
)
print(f"   Reproject downsampled shape: {downsampled.shape}")

# Upsampling - increase resolution (interpolation)
print("\n   ⬆️ Upsampling (refinement):")

# Upsample with different methods
resampling_methods = [
    ('nearest', rasterio.enums.Resampling.nearest),
    ('bilinear', rasterio.enums.Resampling.bilinear),
    ('cubic', rasterio.enums.Resampling.cubic)
]

upsampled_results = {}
for method_name, method in resampling_methods:
    upsampled = elevation_da.rio.reproject(
        elevation_da.rio.crs,
        resolution=(original_res[0] / 2, original_res[1] / 2),  # 2x finer
        resampling=method
    )
    upsampled_results[method_name] = upsampled
    print(f"   {method_name.capitalize()}: {upsampled.shape}")

# Quality comparison
print("\n   📊 Resampling quality metrics:")
for method_name, resampled in upsampled_results.items():
    # Sample a region for comparison
    sample_region = resampled.sel(
        longitude=slice(105.5, 106.5),
        latitude=slice(20.5, 21.5)
    )
    
    smoothness = sample_region.std().values  # Lower std = smoother
    print(f"   {method_name.capitalize()}: Smoothness (std) = {smoothness:.2f}m")

print("   ✅ Different resampling methods suit different use cases!")
```


```python
# ==========================================
# 5B. DEMO ADVANCED RESAMPLING TECHNIQUES
# ==========================================

print("\n🔧 5B. DEMO ADVANCED RESAMPLING TECHNIQUES:")

# Demo 1: Custom resolution resampling
print("\n   📐 Demo 1: Custom Resolution Targets")

# Define various target resolutions
target_resolutions = [
    (0.01, 0.01, "High Resolution (~1km)"),
    (0.05, 0.05, "Medium Resolution (~5km)"),
    (0.1, 0.1, "Low Resolution (~10km)"),
    (0.2, 0.2, "Very Low Resolution (~20km)")
]

resampled_datasets = {}
for res_x, res_y, description in target_resolutions:
    try:
        print(f"\n     Resampling to {description}:")
        
        # Resample using bilinear interpolation
        resampled = elevation_da.rio.reproject(
            elevation_da.rio.crs,
            resolution=(res_x, res_y),
            resampling=rasterio.enums.Resampling.bilinear
        )
        
        resampled_datasets[description] = resampled
        
        print(f"       Original: {elevation_da.shape} pixels")
        print(f"       Resampled: {resampled.shape} pixels")
        print(f"       Compression ratio: {(elevation_da.size / resampled.size):.1f}x")
        print(f"       File size reduction: {(1 - resampled.size/elevation_da.size)*100:.1f}%")
        
        # Statistics comparison
        orig_mean = elevation_da.mean().values
        resamp_mean = resampled.mean().values
        print(f"       Mean elevation: Orig={orig_mean:.1f}m, Resampled={resamp_mean:.1f}m (Δ={abs(orig_mean-resamp_mean):.1f}m)")
        
    except Exception as e:
        print(f"       ⚠️ Resampling failed: {e}")

# Demo 2: Resampling method comparison
print("\n   🎯 Demo 2: Resampling Method Quality Comparison")

# Test different resampling methods
resampling_methods = [
    (rasterio.enums.Resampling.nearest, "Nearest Neighbor"),
    (rasterio.enums.Resampling.bilinear, "Bilinear"),
    (rasterio.enums.Resampling.cubic, "Cubic Convolution"),
    (rasterio.enums.Resampling.average, "Average"),
    (rasterio.enums.Resampling.lanczos, "Lanczos")
]

# Upsample to higher resolution for quality testing
target_res = (0.02, 0.02)  # 2x finer resolution

method_results = {}
for method, method_name in resampling_methods:
    try:
        print(f"\n     Testing {method_name}:")
        
        start_time = time.time()
        upsampled = elevation_da.rio.reproject(
            elevation_da.rio.crs,
            resolution=target_res,
            resampling=method
        )
        processing_time = time.time() - start_time
        
        # Quality metrics
        orig_stats = {
            'mean': elevation_da.mean().values,
            'std': elevation_da.std().values,
            'min': elevation_da.min().values,
            'max': elevation_da.max().values
        }
        
        new_stats = {
            'mean': upsampled.mean().values,
            'std': upsampled.std().values,
            'min': upsampled.min().values,
            'max': upsampled.max().values
        }
        
        # Calculate differences
        mean_diff = abs(orig_stats['mean'] - new_stats['mean'])
        std_diff = abs(orig_stats['std'] - new_stats['std'])
        
        method_results[method_name] = {
            'mean_diff': mean_diff,
            'std_diff': std_diff,
            'processing_time': processing_time,
            'min_val': new_stats['min'],
            'max_val': new_stats['max']
        }
        
        print(f"       Processing time: {processing_time:.3f}s")
        print(f"       Mean difference: {mean_diff:.2f}m")
        print(f"       Std difference: {std_diff:.2f}m")
        print(f"       Value range: {new_stats['min']:.1f}m to {new_stats['max']:.1f}m")
        
    except Exception as e:
        print(f"       ⚠️ {method_name} failed: {e}")

# Demo 3: Performance vs Quality Trade-offs
print("\n   ⚡ Demo 3: Performance vs Quality Trade-offs")

print("     Method Performance Summary:")
if method_results:
    # Sort by processing time
    sorted_methods = sorted(method_results.items(), key=lambda x: x[1]['processing_time'])
    
    print(f"     {'Method':<20} {'Time(s)':<8} {'Mean Δ':<8} {'Std Δ':<8} {'Quality Score':<12}")
    print(f"     {'-'*60}")
    
    for method_name, results in sorted_methods:
        # Simple quality score (lower differences = higher quality)
        quality_score = 100 - (results['mean_diff'] + results['std_diff'])
        quality_score = max(0, quality_score)  # Ensure non-negative
        
        print(f"     {method_name:<20} {results['processing_time']:<8.3f} {results['mean_diff']:<8.2f} {results['std_diff']:<8.2f} {quality_score:<12.1f}")

# Demo 4: Resampling with different target grids
print("\n   🗺️ Demo 4: Target Grid Matching")

if 'UTM_48N' in globals() and 'transformed_datasets' in globals() and 'UTM_48N' in transformed_datasets:
    utm_data = transformed_datasets['UTM_48N']
    
    # Create a coarser target grid
    print("\n     Creating target grid:")
    target_bounds = utm_data.rio.bounds()
    target_width = 50  # Much coarser grid
    target_height = 75
    
    # Calculate target resolution
    target_res_x = (target_bounds[2] - target_bounds[0]) / target_width
    target_res_y = (target_bounds[3] - target_bounds[1]) / target_height
    
    print(f"       Target grid: {target_width} x {target_height}")
    print(f"       Target resolution: {target_res_x:.0f}m x {target_res_y:.0f}m")
    
    # Resample to target grid
    try:
        target_resampled = utm_data.rio.reproject(
            utm_data.rio.crs,
            shape=(target_height, target_width),
            resampling=rasterio.enums.Resampling.bilinear
        )
        
        print(f"       Resampled shape: {target_resampled.shape}")
        print(f"       Actual resolution: {target_resampled.rio.resolution()}")
        
        # Validate that we hit our target
        actual_width = target_resampled.rio.width
        actual_height = target_resampled.rio.height
        print(f"       Grid match: {actual_width == target_width and actual_height == target_height}")
        
    except Exception as e:
        print(f"       ⚠️ Target grid resampling failed: {e}")
else:
    print("\n     ⚠️ UTM data not available - skipping target grid demo")

# Demo 5: Memory-efficient resampling for large datasets
print("\n   💾 Demo 5: Memory-Efficient Resampling")

try:
    # Create chunked version of elevation data
    chunked_elev = elevation_da.chunk({'latitude': 50, 'longitude': 50})
    
    print(f"     Original chunks: {elevation_da.chunks if hasattr(elevation_da, 'chunks') else 'Not chunked'}")
    print(f"     Chunked version: {chunked_elev.chunks}")
    
    # Resample chunked data
    start_time = time.time()
    chunked_resampled = chunked_elev.rio.reproject(
        chunked_elev.rio.crs,
        resolution=(0.1, 0.1),  # 10km resolution
        resampling=rasterio.enums.Resampling.bilinear
    )
    chunked_time = time.time() - start_time
    
    # Compare with non-chunked
    start_time = time.time()
    regular_resampled = elevation_da.rio.reproject(
        elevation_da.rio.crs,
        resolution=(0.1, 0.1),
        resampling=rasterio.enums.Resampling.bilinear
    )
    regular_time = time.time() - start_time
    
    print(f"     Chunked resampling time: {chunked_time:.3f}s")
    print(f"     Regular resampling time: {regular_time:.3f}s")
    print(f"     Performance difference: {((regular_time - chunked_time) / regular_time * 100):+.1f}%")
    
except Exception as e:
    print(f"     ⚠️ Memory-efficient resampling demo failed: {e}")

print("\n   ✅ Advanced resampling techniques demonstrated!")
```

## Tóm tắt

Bạn đã hoàn thành Bài 8 và học được RioXArray - thư viện kết hợp sức mạnh XArray và Rasterio cho geospatial analysis.

### Các khái niệm chính đã nắm vững:
- ✅ **Geospatial DataArrays**: CRS-aware arrays với coordinate reference systems và spatial metadata
- ✅ **Raster I/O operations**: Đọc/ghi GeoTIFF, NetCDF và HDF5 với geospatial attributes preservation
- ✅ **CRS management**: Coordinate transformations, reprojection và spatial reference handling
- ✅ **Spatial operations**: Clipping, masking, buffering và geometric transformations trên raster data
- ✅ **Resampling workflows**: Grid interpolation, spatial aggregation và resolution adjustments
- ✅ **Integration capabilities**: Seamless workflow với geopandas, rasterio và matplotlib ecosystem
- ✅ **Performance optimization**: Chunking strategies, lazy evaluation cho large geospatial datasets
- ✅ **Real-world applications**: DEM analysis, satellite imagery processing và environmental monitoring

### Kỹ năng bạn có thể áp dụng:
- Xử lý và phân tích satellite imagery và raster datasets một cách chuyên nghiệp với spatial awareness
- Thực hiện geospatial workflows cho environmental monitoring và climate analysis ở Việt Nam
- Optimize raster processing performance cho production-scale geospatial applications với memory efficiency
- Tích hợp RioXArray với scientific Python stack cho advanced spatial-temporal analysis
- Chuẩn bị foundation expertise cho remote sensing và GIS applications trong research và industry
