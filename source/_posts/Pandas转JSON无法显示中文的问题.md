---
title: Pandas转JSON无法显示中文的问题
date: 2024-09-18 11:24:46
tags:
---

**问题解释：**

> 在使用pandas转换DataFrame到JSON格式时，如果DataFrame中含有中文字符，可能会遇到无法正确显示中文的问题。这通常是因为默认情况下，pandas会将字符串编码为Unicode Escape形式，这不是JSON的标准格式，而是Python内部的表现形式。

**解决方法：**

1. 在转换为JSON之前，确保DataFrame的编码是正确的，通常使用UTF-8编码。

2. 使用to_json方法时，设置参数ensure_ascii=False，这样可以避免ASCII编码的问题，正确显示中文。

**示例代码：**

```python
import pandas as pd

# 创建一个包含中文的DataFrame

df = pd.DataFrame({'data': ['中文内容', '更多中文', '最后一条']})

# 转换为JSON，确保使用utf-8编码，并且不使用ASCII编码

json_str = df.to_json(orient='records', force_ascii=False)

print(json_str)
```

这段代码会创建一个包含中文的DataFrame，并将其转换为JSON格式，正确显示中文字符。


