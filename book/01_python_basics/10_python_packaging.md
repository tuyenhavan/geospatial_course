```python
import numpy as np
from itertools import product
```


```python
x = np.arange(1, 5)
b = np.array([1, 2, 3, 4])
x**b
```




    array([  1,   4,  27, 256])




```python
np.power(x, b)
```




    array([  1,   4,  27, 256])




```python
np.nanargmax([1, 2, 3, np.nan, 4])

```




    np.int64(4)




```python

```
