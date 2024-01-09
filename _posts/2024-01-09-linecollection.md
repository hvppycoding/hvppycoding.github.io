---
title: "LineCollection 예시"
excerpt: "Matplotlib"
date: 2024-01-09 13:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Python
---

## LineCollection

많은 Line segment를 그릴 때, `plot` 함수를 이용하면 매우 느리다.  
대신에 `LineCollection`을 이용하여 효율적으로 구현할 수 있다.  

아래는 `LineCollection`을 활용한 예시이다.  
`ListedColormap`과 `Normalize`를 이용하여 색상도 지정하였다.  

```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.collections import LineCollection
from matplotlib.colors import ListedColormap, Normalize

# Generate some example data (points for the line segments)
line0 = ((0, 1), (1, 0), (2, 1))
line1 = ((0, 0), (1, 1), (2, 0))
line2 = ((0, 0), (2, 1))
line3 = ((0, 1), (2, 0))

# Create the line segments
segments = (line0, line1, line2, line3)
values = [-1, 2, 0, 1]

# Create a custom colormap
cmap = ListedColormap(['blue', 'green', 'yellow', 'orange', 'red'])

# Normalize the values to match the colormap
norm = Normalize(-2.5, 2.5)

# Create the LineCollection
lc = LineCollection(segments, cmap=cmap, norm=norm)

# Set the values to assign colors to each line segment
lc.set_array(values)

# Create a figure and axes
fig, ax = plt.subplots()

ax.set_xlim(0, 2)
ax.set_ylim(0, 1)
# Add the LineCollection to the axes
ax.add_collection(lc)

# Add a colorbar with your custom colormap
cbar = plt.colorbar(lc, ax=ax, cmap=cmap, norm=norm)

# Set labels for the colorbar
cbar.set_label('Custom Colorbar Label')

# Add other plot elements or customize as needed
plt.xlabel('X-axis')
plt.ylabel('Y-axis')
plt.title('LineCollection with Custom Colorbar')

# Show the plot
plt.show()

```

![2024-01-09-linecollection]({{site.baseurl}}/assets/images/2024-01-09-linecollection.png){: .align-center}  
