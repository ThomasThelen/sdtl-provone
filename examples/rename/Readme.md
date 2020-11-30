This example is the [rename](https://gitlab.com/c2metadata/python-to-sdtl/-/blob/master/test/sdtl/rename.json) test case.

It represents the following source code

```
""Test the various methods of renaming columns."""

import pandas as pd

"""
    A B
--+----
0 | 1 4
1 | 2 5
2 | 3 6
"""
df = pd.read_csv("rename.csv")

df.rename(columns={ "A": "a", "B": "b" }, inplace=True)
df2 = df.rename(columns={ "a": "x", "b": "y" })
df3 = df.rename(index={ 0: "x", 1: "y", 2: "z" })   # this is unsupported


df.columns["a"] = "x"

# this highlights the necessity of keeping track of variable changes
# (so that we know what the old column names are when parsing such statements)
df.columns = ["one", "two"]

df.to_csv("rename.csv")

```