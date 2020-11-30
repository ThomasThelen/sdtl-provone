This example is the [compute](https://gitlab.com/c2metadata/python-to-sdtl/-/blob/master/test/sdtl/compute.json) test case.

It represents the following source code

```python
import pandas as pd

"""
    A B
--+----
0 | 1 4
1 | 2 5
2 | 3 6
"""
df = pd.read_csv("df.csv")

df["A"] = 3
df["B"] = df["B"] + 6.5
df["C"] = df["A"] - df["B"]

"""
     fahrenheit
--+------------
0 |  45
1 |  69
2 |  12
3 | -40
"""
temps = pd.read_csv("temps.csv")

temps_c = temps.assign(celsius=((temps["fahrenheit"] - 32) * 5 / 9))
temps_c = temps.assign(celsius=lambda x: (x.fahrenheit - 32) * 5 / 9)

temps_k = temps.assign(celsius=lambda x: ((x["fahrenheit"] - 32) * 5 / 9), 
                       kelvin=lambda x : (x["celsius"] + 273))

temps.to_csv("temps.csv")

```