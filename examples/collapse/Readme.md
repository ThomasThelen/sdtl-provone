This example is the [collapse](https://gitlab.com/c2metadata/python-to-sdtl/-/blob/master/test/sdtl/collapse.json) test case.

It represents the following source code

```
import pandas as pd

scores = pd.read_csv("scores.csv")

mean = scores.mean()
desc = scores.describe()
median_age = scores.groupby("Age").median()
group_sum = scores.groupby(["Gender", "Age"]).sum()
group_des = scores.groupby("Gender").describe()
non_index = scores.groupby("Gender", as_index=False).mad()
single_agg = scores.groupby("Gender").agg("var")
multiple_agg = scores.groupby("Gender").agg(["quantile", "nunique", "skew"])
named_agg = scores.groupby("Gender").agg(
    youngest=pd.NamedAgg(column="Age", aggfunc="min"),
    oldest=pd.NamedAgg(column="Age", aggfunc="max"),
    mean_score=pd.NamedAgg(column="
```