# Fixed and rolling temporal groupby

## Grouping by fixed windows with `groupby_dynamic`

We can calculate temporal statistics using `groupby_dynamic` to group rows into days/months/years etc.

### Annual average example

In following simple example we calculate the annual average closing price of Apple stock prices. We first load the data from CSV:

```python
{{#include ../../examples/time_series/parsing_dates.py:4:4}}
```

```text
{{#include ../../outputs/time_series/parse_dates_example_df.txt}}
```

> The dates are sorted in ascending order - if they are not sorted in this way the `groupby_dynamic` output will not be correct!

To get the annual average closing price we tell `groupby_dynamic` that we want to:

- group by the `Date` column on an annual (`1y`) basis
- take the mean values of the `Close` column for each year:

```python
{{#include ../../examples/time_series/parsing_dates.py:14:17}}
```

The annual average closing price is then:

```text
{{#include ../../outputs/time_series/parse_dates_annual_average_df.txt}}
```

### Parameters for `groupby_dynamic`

A dynamic window is defined by a:

- **every**: indicates the interval of the window
- **period**: indicates the duration of the window
- **offset**: can be used to offset the start of the windows

#### `every`, `period` and `offset` parameters to control temporal window size

The value for `every` sets how often the groups start. The time period values are flexible - for example we could take:

- the average over 2 year intervals by replacing `1y` with `2y`
- the average over 18 month periods by replacing `1y` with `1y6mo`

We can also use the `period` parameter to set how long the time period for each group is. For example, if we set the `every` parameter to be `1y` and the `period` parameter to be `2y` then we would get groups at one year intervals where each groups spanned two years.

If the `period` parameter is not specified then it is set equal to the `every` parameter so that if the `every` parameter is set to be `1y` then each group spans `1y` as well.

Because _**every**_ does not have to be equal to _**period**_, we can create many groups in a very flexible way. They may overlap
or leave boundaries between them.

Let's see how the windows for some parameter combinations would look. Let's start out boring. 🥱

>

- every: 1 day -> `"1d"`
- period: 1 day -> `"1d"`

```text
this creates adjacent windows of the same size
|--|
   |--|
      |--|
```

>

- every: 1 day -> `"1d"`
- period: 2 days -> `"2d"`

```text
these windows have an overlap of 1 day
|----|
   |----|
      |----|
```

>

- every: 2 days -> `"2d"`
- period: 1 day -> `"1d"`

```text
this would leave gaps between the windows
data points that in these gaps will not be a member of any group
|--|
       |--|
              |--|
```

See [the API pages](https://pola-rs.github.io/polars/py-polars/html/reference/api/polars.DataFrame.groupby_dynamic.html) for the full range of time periods.

#### `truncate` parameter to set the start date for each group

The `truncate` parameter is a Boolean variable that determines what datetime value is associated with each group in the output. In the example above the first data point is on 23rd February 1981. If `truncate = True` (the default) then the date for the first year in the annual average is 1st January 1981. However, if `truncate = False` then the date for the first year in the annual average is the date of the first data point on 23rd February 1981.

### Using expressions in `groupby_dynamic`

We aren't restricted to using simple aggregations like `mean` in a groupby operation - we can use the full range of expressions available in Polars.

In the snippet below we create a `date range` with every **day** (`"1d"`) in 2021 and turn this into a `DataFrame`.

Then in the `groupby_dynamic` we create dynamic windows that start every **month** (`"1mo"`) and have a window length of `1` month. The values that match these dynamic windows are then assigned to that group and can be aggregated with the powerful expression API.

Below we show an example where we use **groupby_dynamic** to compute:

- the number of days until the end of the month
- the number of days in a month

```python
{{#include ../../examples/time_series/days_month.py:4:}}
print(out)
```

```text
{{#include ../../outputs/time_series/days_month.txt}}
```

## Grouping by rolling windows with `groupby_rolling`

The rolling groupby is another entrance to the `groupby` context. But different from the `groupby_dynamic` the windows are
not fixed by a parameter `every` and `period`. In a rolling groupby the windows are not fixed at all! They are determined
by the values in the `index_column`.

So imagine having a time column with the values `{2021-01-01, 20210-01-05}` and a `period="5d"` this would create the following
windows:

```text

2021-01-01   2021-01-06
    |----------|

       2021-01-05   2021-01-10
             |----------|
```

Because the windows of a rolling groupby are always determined by the values in the `DataFrame` column, the number of
groups is always equal to the original `DataFrame`.

## Combining Groupby and Dynamic / Rolling

Rolling and dynamic groupby's can be combined with normal groupby operations.

Below is an example with a dynamic groupby.

```python
{{#include ../../examples/time_series/dynamic_ds.py:0:}}
print(out)
```

```text
{{#include ../../outputs/time_series/dyn_df.txt}}
```

```python
{{#include ../../examples/time_series/dynamic_groupby.py:4:}}
print(out)
```

```text
{{#include ../../outputs/time_series/dyn_gb.txt}}
```
