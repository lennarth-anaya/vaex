# Vaex

Python / Vaex test to check if it's that fast and how it works with huge files (it wasn't so cool with 16 GB of RAM).

## Setup

1. Make sure you have Python 3.7 and [Anacodna](https://www.anaconda.com/products/individual)

2. Ubuntu 18 uses Python 2.6 for terminal and other apps, so you better create a test environment for Python 3.7:

```
conda create --name p3_7-env python=3.7
```

3. Activate your environment every time you want to run these tests (with command conda deactivate you will go back to default):

```
conda activate p3_7-env
```

4. Install Vaex (it could be with Anaconda itself or PIP):

```
conda install -c conda-forge vaex
#  or pip install vaex
```

## Run the examples

As per step 3, recall to activate your environment if you are re-running this examples.

5. Run the examples (jupyter allows to see plots, though Python interactive mode would also work in most of the cases):

```
cd {working folder where this repo was cloned to}
jupyter notebook
# python
```

6. Open vaex_test notebook, the content will be similar to code below (which you could also run Python interactive mode):

```
import vaex
import pandas as pd
import numpy as np
# n_rows = 1000000 <-- totally not working with a laptop with 
n_rows = 100000
n_cols = 1000
df = pd.DataFrame(np.random.randint(0, 100, size=(n_rows, n_cols)), columns=['col%d' % i for i in range(n_cols)])
df.head()

# check main-memory usage:
df.info(memory_usage='deep')

# save it to disk:
file_path = 'big_file.csv'
df.to_csv(file_path, index=False)

# If we cannot open a bigger file with pandas, because of memory constraints, we can covert it to HDF5 and process it with Vaex.
#   below line will create in disk several chunks of 100,000 rows (approx 800 MB in this example) and in the end concatenate all
#   of them in one single file, so we have to have twice the final HDF5 size available in disk.
#
#   This process will take several minutes.
dv = vaex.from_csv(file_path, convert=True, chunk_size=100_000)
	# if the number of chunks is 9 and program looks stale:
	# Ctrl+C
	# df = vaex.open(file_path + '*.hdf5')


type(dv)
# output must state a class related to Hdf5 in memory

# test the data frame
dv = vaex.open(file_path + '.hdf5')
suma = dv.col1.sum()
suma

# Plotting
dv.plot1d(dv.col2, figsize=(14, 7))

# column sum
dv['col1_plus_col2'] = dv.col1 + dv.col2
dv['col1_plus_col2']


# Filtering
dv_90 = dv[dv.col1 > 90]
dv_90

# Aggregations
dv['col1_50'] = dv.col1 >= 50
dv_group = dv.groupby(dv['col1_50'], agg=vaex.agg.sum(dv['col3']))
dv_group

# Joins
dv_join = dv.join(dv_group, on='col1_50')
dv_join
```