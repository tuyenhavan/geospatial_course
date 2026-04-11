```python
import os 
import nbformat
from nbconvert import MarkdownExporter
from pathlib import Path
```


```python
outpath = r"G:\My Drive\python\geocourse\book"

path = r"G:\My Drive\python\geocourse\notebooks"
```


```python
def convert_notebook_to_markdown(notebook_path: str, output_path: str = None) -> str:
    """
    Convert a Jupyter notebook (.ipynb) to Markdown (.md).
    Handles both markdown cells and code cells.

    Args:
        notebook_path: Path to the .ipynb file
        output_path: Path to save the .md file (optional).
                     If None, saves in the same directory as the notebook.

    Returns:
        Path to the saved markdown file
    """
    nb_path = Path(notebook_path)

    if not nb_path.exists():
        raise FileNotFoundError(f"Notebook not found: {notebook_path}")
    if nb_path.suffix != ".ipynb":
        raise ValueError(f"Expected a .ipynb file, got: {nb_path.suffix}")

    # Load notebook
    try:
        with open(nb_path, "r", encoding="utf-8") as f:
            notebook = nbformat.read(f, as_version=4)
    except Exception as e:
        print(f"⚠️ Error reading notebook {nb_path.name}: {e}")
        return None
    # Check if notebook is empty
    # Check if notebook has no cells or all cells are blank
    if not notebook.cells or all(cell.source.strip() == "" for cell in notebook.cells):
        print(f"⚠️ Skipped (no content): {nb_path.name}")
        return None
    # Convert to markdown
    exporter = MarkdownExporter()
    markdown_output, _ = exporter.from_notebook_node(notebook)
    markdown_output = markdown_output.encode("utf-8", errors="replace").decode("utf-8")
    # Determine output path
    if output_path is None:
        out_path = nb_path.with_suffix(".md")
    else:
        out_path = Path(output_path)
        out_path.parent.mkdir(parents=True, exist_ok=True)

    # Save markdown file
    with open(out_path, "w", encoding="utf-8") as f:
        f.write(markdown_output)

    print(f"✅ Converted: {nb_path.name} → {out_path}")
    return str(out_path)
```


```python
for root, _, files in os.walk(path):
    for file in files:
        if file.endswith('.ipynb'):
            folder_path = os.path.join(outpath, os.path.relpath(root, path))
            os.makedirs(folder_path, exist_ok=True)
            file_path = os.path.join(root, file)
            outfile = os.path.join(folder_path, file.replace('.ipynb', '.md'))
            if os.path.exists(outfile):
                continue
            convert_notebook_to_markdown(file_path, outfile)
            # print(folder_path)
```


```python
import pandas as pd
import numpy as np
data = {
    "Petal_Length": [1.4, 1.3, 1.4, 4.0, 4.7, 3.6],
    "Petal_Width":  [0.2, 0.4, 0.3, 1.0, 1.4, 1.3],
    "Label":        [0,   0,   0,   1,   1,   1]
}

df = pd.DataFrame(data)

x_test = [2.4, 0.8]
```


```python
df['distance'] = np.sqrt((df['Petal_Length'] - x_test[0])**2 + (df['Petal_Width'] - x_test[1])**2)
df_sorted = df.sort_values(by='distance')
k = 3
nearest_neighbors = df_sorted.head(k)
predicted_label = nearest_neighbors['Label'].mode()[0]
```


```python
df
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
      <th>Petal_Length</th>
      <th>Petal_Width</th>
      <th>Label</th>
      <th>distance</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1.4</td>
      <td>0.2</td>
      <td>0</td>
      <td>1.166190</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1.3</td>
      <td>0.4</td>
      <td>0</td>
      <td>1.170470</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1.4</td>
      <td>0.3</td>
      <td>0</td>
      <td>1.118034</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4.0</td>
      <td>1.0</td>
      <td>1</td>
      <td>1.612452</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4.7</td>
      <td>1.4</td>
      <td>1</td>
      <td>2.376973</td>
    </tr>
    <tr>
      <th>5</th>
      <td>3.6</td>
      <td>1.3</td>
      <td>1</td>
      <td>1.300000</td>
    </tr>
  </tbody>
</table>
</div>




```python

```
