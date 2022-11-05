Writing Functions
================
Jingyi Yao
2022-11-05

## Basic Function

#### 1. calculate z score

``` r
x_vec = rnorm(25, mean = 5, sd = 3)  # sampled from a normal distribution

(x_vec - mean(x_vec)) / sd(x_vec)    # calculate the z score for the sample 
```

    ##  [1] -0.83687228  0.01576465 -1.05703126  1.50152998  0.16928872 -1.04107494
    ##  [7]  0.33550276  0.59957343  0.42849461 -0.49894708  1.41364561  0.23279252
    ## [13] -0.83138529 -2.50852027  1.00648110 -0.22481531 -0.19456260  0.81587675
    ## [19]  0.68682298  0.44756609  0.78971253  0.64568566 -0.09904161 -2.27133861
    ## [25]  0.47485186

#### 2. writing a `z_score function(vector)` function to calculate z score

``` r
z_scores = function(x) {      # function(argument) 
  
  z = (x - mean(x)) / sd(x)   # operation
  z                           # output
  
}

z_scores(x_vec)
```

    ##  [1] -0.83687228  0.01576465 -1.05703126  1.50152998  0.16928872 -1.04107494
    ##  [7]  0.33550276  0.59957343  0.42849461 -0.49894708  1.41364561  0.23279252
    ## [13] -0.83138529 -2.50852027  1.00648110 -0.22481531 -0.19456260  0.81587675
    ## [19]  0.68682298  0.44756609  0.78971253  0.64568566 -0.09904161 -2.27133861
    ## [25]  0.47485186

#### 3. sample from the vector for sample size times

`sample(vector,sample size,replace)`

``` r
z_scores(sample(c(TRUE, FALSE), 25, replace = TRUE))
```

    ##  [1] -0.7348469  1.3063945 -0.7348469 -0.7348469  1.3063945  1.3063945
    ##  [7] -0.7348469 -0.7348469 -0.7348469  1.3063945  1.3063945 -0.7348469
    ## [13] -0.7348469 -0.7348469 -0.7348469 -0.7348469 -0.7348469  1.3063945
    ## [19] -0.7348469 -0.7348469 -0.7348469 -0.7348469  1.3063945  1.3063945
    ## [25]  1.3063945

#### 4. identify the data type and data length before operation

`is.numeric(vector)` judge if the vector is numeric, output is True or
False

`length(x)==1` if the argument is a single value

`stop("warning message")` when the data type is not satisfied, show the
message

``` r
z_scores = function(x) {
  
  if (!is.numeric(x)) {  
    stop("Argument x should be numeric")
  } 
  else if (length(x) == 1) {
    stop("Z scores cannot be computed for length 1 vectors")
  }
  
  z = x - mean(x) / sd(x)    # if -- else if -- else, else can be omitted
  
  z
}
```

## Multiple Outputs

#### 1. use list to show output

`list(output name 1 = output1, output name 2 = output 2)`

``` r
mean_and_sd = function(x) {
  
  if (!is.numeric(x)) {
    stop("Argument x should be numeric")
  } else if (length(x) == 1) {
    stop("Cannot be computed for length 1 vectors")
  }
  
  mean_x = mean(x)
  sd_x = sd(x)

  list(mean = mean_x,    # output the multiple outputs in a list
       sd = sd_x)
}

mean_and_sd(x_vec)
```

    ## $mean
    ## [1] 5.505996
    ## 
    ## $sd
    ## [1] 2.850324

#### 2. use df (tibble) to show the output

`tibble(output name 1 = output 1, output name 2 = output 2)`

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
    mean = mean_x,      # the out put is a tibble (data frame)
    sd = sd_x
  )
}
```

## Multiple inputs

#### 1. 3 arguments and with defaults

``` r
sim_mean_sd = function(n, mu = 2, sigma = 3) {
  
  sim_data = tibble(
    x = rnorm(n, mean = mu, sd = sigma),  # use the arguments here
  )
  
  sim_data %>% 
    summarize(
      mu_hat = mean(x),
      sigma_hat = sd(x)     
    )                     # the result of this operation is the output
}

sim_mean_sd(25,10,2)
```

    ## # A tibble: 1 × 2
    ##   mu_hat sigma_hat
    ##    <dbl>     <dbl>
    ## 1   9.84      2.15

## Example : Scraping Amazon

``` r
url = "https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber=1"

dynamite_html = read_html(url)

review_titles = 
  dynamite_html %>%
  html_nodes(".a-text-bold span") %>%
  html_text()

review_stars = 
  dynamite_html %>%
  html_nodes("#cm_cr-review_list .review-rating") %>%
  html_text() %>%
  str_extract("^\\d") %>%
  as.numeric()

review_text = 
  dynamite_html %>%
  html_nodes(".review-text-content span") %>%
  html_text() %>% 
  str_replace_all("\n", "") %>% 
  str_trim()

reviews = tibble(
  title = review_titles,
  stars = review_stars,
  text = review_text
)
```

``` r
read_page_reviews <- function(url) {
  
  html = read_html(url)
  
  review_titles = 
    html %>%
    html_nodes(".a-text-bold span") %>%
    html_text()
  
  review_stars = 
    html %>%
    html_nodes("#cm_cr-review_list .review-rating") %>%
    html_text() %>%
    str_extract("^\\d") %>%
    as.numeric()
  
  review_text = 
    html %>%
    html_nodes(".review-text-content span") %>%
    html_text() %>% 
    str_replace_all("\n", "") %>% 
    str_trim() %>% 
    str_subset("The media could not be loaded.", negate = TRUE) %>% 
    str_subset("^$", negate = TRUE)
  
  tibble(
    title = review_titles,
    stars = review_stars,
    text = review_text
  )
}
```

``` r
url_base = "https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber="
vec_urls = str_c(url_base, 1:5)

dynamite_reviews = bind_rows(
  read_page_reviews(vec_urls[1]),
  read_page_reviews(vec_urls[2]),
  read_page_reviews(vec_urls[3]),
  read_page_reviews(vec_urls[4]),
  read_page_reviews(vec_urls[5])
)

dynamite_reviews
```

    ## # A tibble: 50 × 3
    ##    title                                      stars text                        
    ##    <chr>                                      <dbl> <chr>                       
    ##  1 Lol hey it’s Napoleon. What’s not to love…     5 Vote for Pedro              
    ##  2 Still the best                                 5 Completely stupid, absolute…
    ##  3 70’s and 80’s Schtick Comedy                   5 …especially funny if you ha…
    ##  4 Amazon Censorship                              5 I hope Amazon does not cens…
    ##  5 Watch to say you did                           3 I know it's supposed to be …
    ##  6 Best Movie Ever!                               5 We just love this movie and…
    ##  7 Quirky                                         5 Good family film            
    ##  8 Funny movie - can't play it !                  1 Sony 4k player won't even r…
    ##  9 A brilliant story about teenage life           5 Napoleon Dynamite delivers …
    ## 10 HUHYAH                                         5 Spicy                       
    ## # … with 40 more rows

## Functions as arguments

#### when to use : the inner function is used for many different objects

``` r
x_vec = rnorm(25, 0, 1)

my_summary = function(x, summ_func) {   # summ_func should be filled with a function
  summ_func(x)                          # summ_func is just the argument name
}
```

``` r
my_summary(x_vec, sd)   # sd the the function as the argument
```

    ## [1] 0.7139041

``` r
my_summary(x_vec, var)
```

    ## [1] 0.5096591

``` r
f = function(x) {
  z = x + y
  z
}

x = 1
y = 2

f(x = y)
```

    ## [1] 4
