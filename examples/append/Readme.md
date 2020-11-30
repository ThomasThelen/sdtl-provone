This example is the [append](https://gitlab.com/c2metadata/python-to-sdtl/-/blob/master/test/sdtl/append.json) test case.

It represents the following source code

```
import pandas as pd

top = pd.read_csv("top.csv")
bottom = pd.read_csv("bottom.csv")

appended = top.append(bottom)
concatenated = pd.concat([top, bottom])
concat_inner = pd.concat([top, bottom], join="inner")

#horizontal = pd.concat([top, bottom[["C"]]], axis=1)
```