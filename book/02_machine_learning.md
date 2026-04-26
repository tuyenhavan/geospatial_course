```python
import numpy as np
import pandas as pd
```

# 1. Phương pháp K-Means

- **Tính K-Means với dữ liệu một cột (feature)**


```python
# example data
df = pd.DataFrame(
    {
        'length': [1.4, 1.3, 1.4, 4.0, 4.7, 3.6]
    }
)
X = df.values
# initial centroids (randomly selected from the data points) with k = 2
k=3
centroids = np.array(X[:k]).reshape(k,-1)  # Using the first k data points as initial centroids

n = 10 # number of iterations
for i in range(n):
    # calculate distances from data points to centroids
    distances = np.sqrt(np.array(np.abs(X[:, np.newaxis] - centroids)**2).sum(axis=2))
    
    # assign clusters based on closest centroid
    labels = np.argmin(distances, axis=1)
    # update centroids by calculating the mean of assigned points
    temp_centroids = np.copy(centroids)
    for j in range(k):
        subset = X[labels == j]
        if len(subset) > 0:  # Avoid division by zero
            centroids[j] = subset.mean(axis=0)
    # Check for convergence (if centroids do not change)
    if np.all(centroids == temp_centroids):
        print(f"Converged after {i+1} iterations.")
        break
print("Final centroids:\n", centroids)
```

- **Phương pháp K-Mean với nhiều cột (features)**


```python
def calculate_distances(X, centroids):
    return np.sqrt(np.array(np.abs(X[:, np.newaxis] - centroids)**2).sum(axis=2))
def assign_clusters(distances):
    return np.argmin(distances, axis=1)
def update_centroids(X, labels, k):
    centroids = np.zeros((k, X.shape[1]))
    for j in range(k):
        subset = X[labels == j]
        if len(subset) > 0:  # Avoid division by zero
            centroids[j] = subset.mean(axis=0)
    return centroids
```

 Tường minh từng bước


```python
df = pd.DataFrame(
    {
        'length': [1.4, 1.3, 1.4, 4.0, 4.7, 3.6],
        'width':  [0.2, 0.4, 0.3, 1.0, 1.4, 1.3],
    }
)
X = df.values
# initial centroids (randomly selected from the data points) with k = 2
k=2 
centroids = np.array(X[:k]).reshape(k,-1)  # Using the first k data points as initial centroids

n = 100 # number of iterations
for i in range(n):
    # calculate distances from data points to centroids
    distances = np.sqrt(np.array(np.abs(X[:, np.newaxis] - centroids)**2).sum(axis=2))
    # distances = np.linalg.norm(X[:, np.newaxis] - centroids, axis=2)  # Using Euclidean distance, the same results as above
    # assign clusters based on closest centroid
    labels = np.argmin(distances, axis=1)
    # update centroids by calculating the mean of assigned points
    temp_centroids = np.copy(centroids)
    for j in range(k):
        subset = X[labels == j]
        if len(subset) > 0:  # Avoid division by zero
            centroids[j] = subset.mean(axis=0)
    # Check for convergence (if centroids do not change)
    if np.all(centroids == temp_centroids):
        print(f"Converged after {i+1} iterations.")
        break
```

Viết hàm rùi tính như trên


```python
X = df.values
# initial centroids (randomly selected from the data points) with k = 3
k=3
centroids = np.array(X[:k]).reshape(k,-1)  # Using the first k data points as initial centroids
centroids.shape
```


```python
for i in range(n):
    distances = calculate_distances(X, centroids)
    labels = assign_clusters(distances)
    temp_centroids = np.copy(centroids)
    # update centroids by calculating the mean of assigned points
    centroids = update_centroids(X, labels, k)
    # Check for convergence (if centroids do not change)
    if np.all(centroids == temp_centroids):
        print(f"Converged after {i+1} iterations.")
        break
```

- **Inference**


```python
# Inference: Assigning new data points to clusters
new_points = np.array([[1.5, 0.3], [4.5, 1.2]])
new_distances = calculate_distances(new_points, centroids) # centroids from training
new_labels = assign_clusters(new_distances)
print("Assigned clusters:\n", new_labels)
```
