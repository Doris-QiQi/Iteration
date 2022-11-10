Iteration and List Columns
================
Jingyi Yao
2022-11-09

## Lists

#### 1. vectors are limited to a single data class

``` r
vec_numeric = 5:8                           # all numeric
vec_char = c("My", "name", "is", "Jeff")    # all character
vec_logical = c(TRUE, TRUE, TRUE, FALSE)    # all logical
```

#### 2. lists can store vectors of different classes, including tables

``` r
l = list(
  vec_numeric = 5:8,
  mat         = matrix(1:8, 2, 4),
  vec_logical = c(TRUE, FALSE),
  summary     = summary(rnorm(1000)))    # a table can be stored in a list
l
```

    ## $vec_numeric
    ## [1] 5 6 7 8
    ## 
    ## $mat
    ##      [,1] [,2] [,3] [,4]
    ## [1,]    1    3    5    7
    ## [2,]    2    4    6    8
    ## 
    ## $vec_logical
    ## [1]  TRUE FALSE
    ## 
    ## $summary
    ##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
    ## -3.00805 -0.69737 -0.03532 -0.01165  0.68843  3.81028

#### 3. Lists can be accessed using names or indices

##### call by name

``` r
l$vec_numeric   # list$vector name
```

    ## [1] 5 6 7 8

##### call be index (use \[\[\]\])

``` r
l[[1]]         # list[[1]]  the first item in a list, use double [[]]
```

    ## [1] 5 6 7 8

##### call the index and then the range

``` r
l[[1]][1:3]      # the range follows the index in a double [[]]
```

    ## [1] 5 6 7

#### 4. dataframes are special lists – each item has the same length

## **FOR** loops

#### 1. create a list and verify it by `is.list(list)`, get T OR F

``` r
list_norms = 
  list(
    a = rnorm(20, 3, 1),
    b = rnorm(20, 0, 5),
    c = rnorm(20, 10, .2),
    d = rnorm(20, -3, 1)
  )

is.list(list_norms)
```

    ## [1] TRUE

#### 2. load the function `mean_and_sd`

##### the argument x is for vectors, not lists. Thus, we cannot

``` r
mean_and_sd = function(x) {
  
  if (!is.numeric(x)) {
    stop("Argument x should be numeric")
  } else if (length(x) == 1) {
    stop("Cannot be computed for length 1 vectors")
  }
  
  mean_x = mean(x)
  sd_x = sd(x)

  tibble(
    mean = mean_x, 
    sd = sd_x
  )
}
```

#### 3. apply the function to **each item** in list `list_norms`

``` r
mean_and_sd(list_norms[[1]])
```

    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  2.70  1.12

``` r
mean_and_sd(list_norms[[2]][3:4])
```

    ## # A tibble: 1 × 2
    ##    mean     sd
    ##   <dbl>  <dbl>
    ## 1  3.87 0.0769

``` r
mean_and_sd(list_norms[[3]][1:10])
```

    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  10.1 0.169

``` r
mean_and_sd(list_norms[[4]])
```

    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1 -3.43  1.18

#### 4. use FOR loop to apply the function in 1 time

##### first, create an output vector in the mode of a **list** to store the result

##### because the function is for vector, it cannot take list as the argument

``` r
output = vector("list", length = 4)    # vector(mode="list")

for (i in 1:4) {
  output[[i]] = mean_and_sd(list_norms[[i]])
}


typeof(output)     # the output is also a list
```

    ## [1] "list"

``` r
output[[1]]        # get the item in output using index
```

    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  2.70  1.12

## Map

##### for loop is somtimes too complex

##### `map` function in `purrr` package can make it clearer

#### 1. `map(list, function)` the first argument is the list/vector/df to iterate over

##### `mean_and _sd()` ’s output is a dataframe

``` r
output = map(list_norms, mean_and_sd)

output
```

    ## $a
    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  2.70  1.12
    ## 
    ## $b
    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1 0.416  4.08
    ## 
    ## $c
    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  10.1 0.191
    ## 
    ## $d
    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1 -3.43  1.18

#### 2. use `.x`to specify the input list, and `~` to specify the function on that list

``` r
output = map(.x = list_norms, ~ mean_and_sd(.x)) # use function as an argument in another function
output
```

    ## $a
    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  2.70  1.12
    ## 
    ## $b
    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1 0.416  4.08
    ## 
    ## $c
    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  10.1 0.191
    ## 
    ## $d
    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1 -3.43  1.18

``` r
output = map(list_norms, median)
# output = map(.x = list_norms, ~ median(.x))

output
```

    ## $a
    ## [1] 2.621376
    ## 
    ## $b
    ## [1] 0.7210996
    ## 
    ## $c
    ## [1] 10.05016
    ## 
    ## $d
    ## [1] -3.521665

## Map Variants

#### 1. `map_dbl(list,function,.id="input"` when the output is double

##### the output of `median()` is double, so we can use `map_dbl`

##### becasue the output of median is not integer or logical, we cannot use `map_int` or \`map_lgl

##### `id="input"` makes sure that we maintain the original list name in the output

``` r
output = map_dbl(list_norms, median, .id = "input")

output
```

    ##          a          b          c          d 
    ##  2.6213757  0.7210996 10.0501641 -3.5216649

#### 2. `map_dfr()` is used when the output is data frame

##### this output result is more tidy than used in a FOR Loop

##### the FOR Loop will generate 4 separate df for the 4 items in a list

##### `map_dfr` can generate a single df to store the 4 items’ result in 4 rows

``` r
output = map_dfr(list_norms, mean_and_sd, .id = "input")

output
```

    ## # A tibble: 4 × 3
    ##   input   mean    sd
    ##   <chr>  <dbl> <dbl>
    ## 1 a      2.70  1.12 
    ## 2 b      0.416 4.08 
    ## 3 c     10.1   0.191
    ## 4 d     -3.43  1.18

#### 3. `map_df()` is used in a pipeline

#### 4. map2() and map2_dbl(), map2_int(), map2_dfr() are for function with 2 arguments

``` r
# output = map2(.x = input_1, .y = input_2, ~func(arg_1 = .x, arg_2 = .y))
```

## List Columns and Operations

#### 1. How we generate a list

##### list(item1 = vector1, item2 = vector2)

##### items are the names and the vectors are the content

##### the vectors can be from different classes

``` r
list_norms = 
  list(
    a = rnorm(20, 3, 1),
    b = rnorm(20, 0, 5),
    c = rnorm(20, 10, .2),
    d = rnorm(20, -3, 1)
  )
```

#### 2. create a df with a column of list

##### name is a character vector, samp is a list

##### tibble() can use vector and list to create a df

##### the column as a list is called the list column

``` r
listcol_df = 
  tibble(
    name = c("a", "b", "c", "d"),
    samp = list_norms
  )

listcol_df
```

    ## # A tibble: 4 × 2
    ##   name  samp        
    ##   <chr> <named list>
    ## 1 a     <dbl [20]>  
    ## 2 b     <dbl [20]>  
    ## 3 c     <dbl [20]>  
    ## 4 d     <dbl [20]>

#### 2. `pull(column name)` from the df

``` r
listcol_df %>% pull(name)   # pulled out a vector
```

    ## [1] "a" "b" "c" "d"

``` r
listcol_df %>% pull(samp)   # pulled out a list
```

    ## $a
    ##  [1] 3.7720863 2.8591606 3.3930939 3.2242186 3.0235420 2.3770373 4.2620094
    ##  [8] 2.5942260 3.6667638 3.1646392 4.7815245 3.7112140 2.6623088 2.9908510
    ## [15] 2.8746908 0.9091539 4.6973939 4.0638812 2.2333834 3.3820076
    ## 
    ## $b
    ##  [1]  1.2094795 -5.6637971  7.4495371 -1.2412355  0.9179185  2.0243550
    ##  [7] -4.9706223 -5.4271466 -0.2427128  2.8804280  0.3691527  3.5297279
    ## [13]  1.6749005  2.7269390 -7.0145295  3.3852695 -3.9490022 -2.3286444
    ## [19] -0.5242603 -8.2392554
    ## 
    ## $c
    ##  [1]  9.980093  9.912028  9.856298  9.889080 10.249098  9.748216  9.956923
    ##  [8]  9.505608  9.865166  9.899741 10.308465  9.807596  9.825564  9.720474
    ## [15] 10.035961 10.230818  9.760293  9.914855 10.273262  9.863141
    ## 
    ## $d
    ##  [1] -2.314488 -2.610496 -4.305396 -1.783112 -2.204826 -3.488203 -3.903993
    ##  [8] -3.390418 -2.185937 -3.564249 -4.874205 -3.142905 -2.228096 -4.159118
    ## [15] -3.237916 -4.222193 -2.883194 -3.132500 -3.033685 -3.622326

#### 3. the list column has all the properties as a list

##### the items in a list column can be called by index

``` r
listcol_df$samp[[1]]
```

    ##  [1] 3.7720863 2.8591606 3.3930939 3.2242186 3.0235420 2.3770373 4.2620094
    ##  [8] 2.5942260 3.6667638 3.1646392 4.7815245 3.7112140 2.6623088 2.9908510
    ## [15] 2.8746908 0.9091539 4.6973939 4.0638812 2.2333834 3.3820076

#### 4. map() can be applied to list columns as list

``` r
map(listcol_df$samp, mean_and_sd)
```

    ## $a
    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  3.23 0.897
    ## 
    ## $b
    ## # A tibble: 1 × 2
    ##     mean    sd
    ##    <dbl> <dbl>
    ## 1 -0.672  4.11
    ## 
    ## $c
    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1  9.93 0.205
    ## 
    ## $d
    ## # A tibble: 1 × 2
    ##    mean    sd
    ##   <dbl> <dbl>
    ## 1 -3.21 0.832

#### 5. because the map() returns a list, it can be stored as a list column again.

##### this is often achieved by mutate(new_list = map(list_col,function))

``` r
listcol_df = 
  listcol_df %>% 
  mutate(summary = map(samp, mean_and_sd))  # create a summary list to store the list

listcol_df
```

    ## # A tibble: 4 × 3
    ##   name  samp         summary         
    ##   <chr> <named list> <named list>    
    ## 1 a     <dbl [20]>   <tibble [1 × 2]>
    ## 2 b     <dbl [20]>   <tibble [1 × 2]>
    ## 3 c     <dbl [20]>   <tibble [1 × 2]>
    ## 4 d     <dbl [20]>   <tibble [1 × 2]>

## Nested Data

``` r
weather_df = 
  rnoaa::meteo_pull_monitors(
    c("USW00094728", "USC00519397", "USS0023B17S"),
    var = c("PRCP", "TMIN", "TMAX"), 
    date_min = "2017-01-01",
    date_max = "2017-12-31") %>%
  mutate(
    name = recode(
      id, 
      USW00094728 = "CentralPark_NY", 
      USC00519397 = "Waikiki_HA",
      USS0023B17S = "Waterhole_WA"),
    tmin = tmin / 10,
    tmax = tmax / 10) %>%
  select(name, id, everything())
```

    ## Registered S3 method overwritten by 'hoardr':
    ##   method           from
    ##   print.cache_info httr

    ## using cached file: ~/Library/Caches/R/noaa_ghcnd/USW00094728.dly

    ## date created (size, mb): 2022-09-06 10:41:07 (8.397)

    ## file min/max dates: 1869-01-01 / 2022-09-30

    ## using cached file: ~/Library/Caches/R/noaa_ghcnd/USC00519397.dly

    ## date created (size, mb): 2022-09-06 10:41:10 (1.697)

    ## file min/max dates: 1965-01-01 / 2020-02-29

    ## using cached file: ~/Library/Caches/R/noaa_ghcnd/USS0023B17S.dly

    ## date created (size, mb): 2022-09-06 10:41:12 (0.949)

    ## file min/max dates: 1999-09-01 / 2022-09-30

``` r
weather_df
```

    ## # A tibble: 1,095 × 6
    ##    name           id          date        prcp  tmax  tmin
    ##    <chr>          <chr>       <date>     <dbl> <dbl> <dbl>
    ##  1 CentralPark_NY USW00094728 2017-01-01     0   8.9   4.4
    ##  2 CentralPark_NY USW00094728 2017-01-02    53   5     2.8
    ##  3 CentralPark_NY USW00094728 2017-01-03   147   6.1   3.9
    ##  4 CentralPark_NY USW00094728 2017-01-04     0  11.1   1.1
    ##  5 CentralPark_NY USW00094728 2017-01-05     0   1.1  -2.7
    ##  6 CentralPark_NY USW00094728 2017-01-06    13   0.6  -3.8
    ##  7 CentralPark_NY USW00094728 2017-01-07    81  -3.2  -6.6
    ##  8 CentralPark_NY USW00094728 2017-01-08     0  -3.8  -8.8
    ##  9 CentralPark_NY USW00094728 2017-01-09     0  -4.9  -9.9
    ## 10 CentralPark_NY USW00094728 2017-01-10     0   7.8  -6  
    ## # … with 1,085 more rows

#### 1. `nest(df,list_name = col_i : col_j)` create a nested df

##### the list is created by combining the original columns

##### the list is a new column in the nested dataframe.

##### the list items(tibbles) are belong to the 3 names respectively

``` r
weather_nest = 
  nest(weather_df, data = date:tmin)

weather_nest
```

    ## # A tibble: 3 × 3
    ##   name           id          data              
    ##   <chr>          <chr>       <list>            
    ## 1 CentralPark_NY USW00094728 <tibble [365 × 4]>
    ## 2 Waikiki_HA     USC00519397 <tibble [365 × 4]>
    ## 3 Waterhole_WA   USS0023B17S <tibble [365 × 4]>

``` r
weather_nest %>% pull(name)
```

    ## [1] "CentralPark_NY" "Waikiki_HA"     "Waterhole_WA"

#### 2. `pull(new list)` get 3 tibbles corresponding to the 3 names

``` r
weather_nest %>% pull(data)
```

    ## [[1]]
    ## # A tibble: 365 × 4
    ##    date        prcp  tmax  tmin
    ##    <date>     <dbl> <dbl> <dbl>
    ##  1 2017-01-01     0   8.9   4.4
    ##  2 2017-01-02    53   5     2.8
    ##  3 2017-01-03   147   6.1   3.9
    ##  4 2017-01-04     0  11.1   1.1
    ##  5 2017-01-05     0   1.1  -2.7
    ##  6 2017-01-06    13   0.6  -3.8
    ##  7 2017-01-07    81  -3.2  -6.6
    ##  8 2017-01-08     0  -3.8  -8.8
    ##  9 2017-01-09     0  -4.9  -9.9
    ## 10 2017-01-10     0   7.8  -6  
    ## # … with 355 more rows
    ## 
    ## [[2]]
    ## # A tibble: 365 × 4
    ##    date        prcp  tmax  tmin
    ##    <date>     <dbl> <dbl> <dbl>
    ##  1 2017-01-01     0  26.7  16.7
    ##  2 2017-01-02     0  27.2  16.7
    ##  3 2017-01-03     0  27.8  17.2
    ##  4 2017-01-04     0  27.2  16.7
    ##  5 2017-01-05     0  27.8  16.7
    ##  6 2017-01-06     0  27.2  16.7
    ##  7 2017-01-07     0  27.2  16.7
    ##  8 2017-01-08     0  25.6  15  
    ##  9 2017-01-09     0  27.2  15.6
    ## 10 2017-01-10     0  28.3  17.2
    ## # … with 355 more rows
    ## 
    ## [[3]]
    ## # A tibble: 365 × 4
    ##    date        prcp  tmax  tmin
    ##    <date>     <dbl> <dbl> <dbl>
    ##  1 2017-01-01   432  -6.8 -10.7
    ##  2 2017-01-02    25 -10.5 -12.4
    ##  3 2017-01-03     0  -8.9 -15.9
    ##  4 2017-01-04     0  -9.9 -15.5
    ##  5 2017-01-05     0  -5.9 -14.2
    ##  6 2017-01-06     0  -4.4 -11.3
    ##  7 2017-01-07    51   0.6 -11.5
    ##  8 2017-01-08    76   2.3  -1.2
    ##  9 2017-01-09    51  -1.2  -7  
    ## 10 2017-01-10     0  -5   -14.2
    ## # … with 355 more rows

#### 3. the reverse operation is to `unnest(nest_df, cols = list)`

##### in this way we may restore the original data frame.

``` r
unnest(weather_nest, cols = data)
```

    ## # A tibble: 1,095 × 6
    ##    name           id          date        prcp  tmax  tmin
    ##    <chr>          <chr>       <date>     <dbl> <dbl> <dbl>
    ##  1 CentralPark_NY USW00094728 2017-01-01     0   8.9   4.4
    ##  2 CentralPark_NY USW00094728 2017-01-02    53   5     2.8
    ##  3 CentralPark_NY USW00094728 2017-01-03   147   6.1   3.9
    ##  4 CentralPark_NY USW00094728 2017-01-04     0  11.1   1.1
    ##  5 CentralPark_NY USW00094728 2017-01-05     0   1.1  -2.7
    ##  6 CentralPark_NY USW00094728 2017-01-06    13   0.6  -3.8
    ##  7 CentralPark_NY USW00094728 2017-01-07    81  -3.2  -6.6
    ##  8 CentralPark_NY USW00094728 2017-01-08     0  -3.8  -8.8
    ##  9 CentralPark_NY USW00094728 2017-01-09     0  -4.9  -9.9
    ## 10 CentralPark_NY USW00094728 2017-01-10     0   7.8  -6  
    ## # … with 1,085 more rows

#### 4. define a function using df as the argument

``` r
weather_lm = function(df) {
  lm(tmax ~ tmin, data = df)
}
```

``` r
weather_lm(weather_nest$data[[1]])
```

    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = df)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##       7.209        1.039

#### 5. `map(list,function)` list here is the listcol in a df

``` r
map(weather_nest$data, weather_lm)
```

    ## [[1]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = df)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##       7.209        1.039  
    ## 
    ## 
    ## [[2]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = df)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##     20.0966       0.4509  
    ## 
    ## 
    ## [[3]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = df)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##       7.499        1.221

``` r
map(weather_nest$data, ~lm(tmax ~ tmin, data = .x))
```

    ## [[1]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = .x)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##       7.209        1.039  
    ## 
    ## 
    ## [[2]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = .x)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##     20.0966       0.4509  
    ## 
    ## 
    ## [[3]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = .x)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##       7.499        1.221

#### 6. `mutate(new_list = map(list,function))` create a new listcol taking value from map

``` r
weather_nest = 
  weather_nest %>% 
  mutate(models = map(data, weather_lm))

weather_nest$models
```

    ## [[1]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = df)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##       7.209        1.039  
    ## 
    ## 
    ## [[2]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = df)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##     20.0966       0.4509  
    ## 
    ## 
    ## [[3]]
    ## 
    ## Call:
    ## lm(formula = tmax ~ tmin, data = df)
    ## 
    ## Coefficients:
    ## (Intercept)         tmin  
    ##       7.499        1.221
