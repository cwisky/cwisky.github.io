---
title: "Jupyter notebook 파일(*.md)을 웹페이지로 보기"
keywords: sample homepage
tags: [getting_started]
sidebar: ai_sidebar
permalink: sample_ipynb.html
summary: These brief instructions will help you get started quickly with the theme. The other topics in this help provide additional information and detail about working with other aspects of this theme and Jekyll.
---


```python
import numpy as np
import matplotlib.pyplot as plt

# evenly sampled time at 200ms intervals
t = np.arange(0., 5., 0.2)

# red dashes, blue squares and green triangles
plt.plot(t, t, 'r--', t, t**2, 'bs', t, t**3, 'g^')
plt.show()
```


    
![png](output_0_0.png)
    

