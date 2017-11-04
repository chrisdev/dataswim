# Dataswim

Utilities to swim in a data lake

## Dependencies

[Dataset](https://dataset.readthedocs.io/en/latest/): to work with databases
[Pandas](https://github.com/pandas-dev/pandas) and 
[Pandas profiling](https://github.com/JosPolfliet/pandas-profiling) to work with data

   ```
   pip install pandas pandas_profiling dataset
   ```

## Usage

Read the [api documentation](http://dataswim.readthedocs.io/en/latest/api.html#database-operations)

## Example

A Jupyter notebook is available as example. It uses Django user data loaded from csv or directly from a Django database.

To run the demo [Holoviews](https://github.com/ioam/holoviews/) is required for data visualization:

   ```
   conda install -c ioam holoviews bokeh
   # or
   pip install holoviews
   ```

Run the [example notebook](notebooks/django_users.ipynb):

```python
from dataswim import ds

#ds.connect('postgresql://dbuser:dbpassword@localhost/dbname')
ds.connect('sqlite:////home/me/my/path/to/a/django/db/db.sqlite3')
# load a table
ds.load("auth_user")
# for a full report:
#ds.report()
# for a data description:
#ds.describe()
# for a quick look:
ds.look()
```

    221 rows
    Fields: id, password, last_login, is_superuser, first_name, last_name, email, is_staff, is_active, date_joined, username


### Clean and format the data


```python
# drop null values
ds.drop()
# format date fields
ds.date(["last_login", "date_joined"])
# keep only the relevant data
ds.reduce(["username", "date_joined"])
# print data
ds.show()
```

    221 rows
    Fields: username, date_joined





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>username</th>
      <th>date_joined</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>bob</td>
      <td>2017-10-31 09:24:10</td>
    </tr>
    <tr>
      <th>1</th>
      <td>denise40</td>
      <td>1974-03-15 13:13:54</td>
    </tr>
    <tr>
      <th>2</th>
      <td>hannah55</td>
      <td>2008-03-05 13:25:32</td>
    </tr>
    <tr>
      <th>3</th>
      <td>dbaker</td>
      <td>2017-04-10 19:12:24</td>
    </tr>
    <tr>
      <th>4</th>
      <td>youngnatasha</td>
      <td>1975-01-15 08:02:34</td>
    </tr>
  </tbody>
</table>
</div>



### Transform the data


```python
# Add a num field
ds.add("Logins", 1)
# Create a datetime index
ds.index("date_joined", "Date")
ds.show()
```

    221 rows
    Fields: username, date_joined, Logins





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>username</th>
      <th>date_joined</th>
      <th>Logins</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2017-10-31 09:24:10</th>
      <td>bob</td>
      <td>2017-10-31 09:24:10</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1974-03-15 13:13:54</th>
      <td>denise40</td>
      <td>1974-03-15 13:13:54</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2008-03-05 13:25:32</th>
      <td>hannah55</td>
      <td>2008-03-05 13:25:32</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2017-04-10 19:12:24</th>
      <td>dbaker</td>
      <td>2017-04-10 19:12:24</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1975-01-15 08:02:34</th>
      <td>youngnatasha</td>
      <td>1975-01-15 08:02:34</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Resample data by one year
# see Pandas frequencies for units: 
# https://github.com/pandas-dev/pandas/blob/master/pandas/tseries/frequencies.py#L98
df = ds.resample("1A", "date").sum()
```


```python
# Set the new data as the main dataset
ds.set(df)
# set nulls to 0
ds.fill("Logins")
# convert floats
ds.to_int("Logins")
# Add a date column from index
ds.date_field("Date")
# keep the original data for later
odf = ds.df
```


```python
# check it out
ds.show()
```

    48 rows
    Fields: Logins, Date





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Logins</th>
      <th>Date</th>
    </tr>
    <tr>
      <th>Date</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1970-12-31</th>
      <td>7</td>
      <td>1970-12-31</td>
    </tr>
    <tr>
      <th>1971-12-31</th>
      <td>4</td>
      <td>1971-12-31</td>
    </tr>
    <tr>
      <th>1972-12-31</th>
      <td>6</td>
      <td>1972-12-31</td>
    </tr>
    <tr>
      <th>1973-12-31</th>
      <td>4</td>
      <td>1973-12-31</td>
    </tr>
    <tr>
      <th>1974-12-31</th>
      <td>6</td>
      <td>1974-12-31</td>
    </tr>
  </tbody>
</table>
</div>



### Draw charts


```python
import holoviews as hv
hv.extension('bokeh')
```

```python
macro = hv.Curve(ds.df, kdims=["Date"], vdims=["Logins"])
plot_opts = dict(show_legend=True, width=940)
style = dict(color='navy')
macro(plot=plot_opts, style=style)
```

![Users chart](https://github.com/synw/dataswim/blob/master/docs/img/users_date_joined.png)

