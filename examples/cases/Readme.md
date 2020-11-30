This example is the [cases](https://gitlab.com/c2metadata/python-to-sdtl/-/blob/master/test/sdtl/cases.json) test case.

It represents the following source code

```
import pandas as pd

df = pd.read_csv("df.csv")

case = df[df["A"] > 2]
case2 = df[(df["A"] < 4) & (~(df["B"] == 3))]
case3 = df[df.isin([2, 3]).all(axis=1)]
case4 = df[df.isna().any(axis=1)]
case5 = df.dropna(how="all")
```