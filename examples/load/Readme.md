This example is the [load](https://gitlab.com/c2metadata/python-to-sdtl/-/blob/master/test/sdtl/load.json) test case.

It represents the following source code

```
""Test the various methods of reading a file (and also integrating comments)"""

import pandas as pd

'''
read in the various files
'''# and make sure to test all four types of quotes

csv_df = pd.read_csv("data.csv")
xls_df = pd.read_excel('data.xlsx')
sas_df = pd.read_sas("""data.sas7bdat""")   # in this context, it's a regular string
stata_df = pd.read_stata('''data.dta''')

```