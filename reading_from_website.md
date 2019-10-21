read from website
================

## get some data

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"

# we want to pull out something we care about, for example, table!
drug_use_xml = 
  read_html(url)

drug_use_xml %>% 
  html_nodes(css = "table")
```

    ## {xml_nodeset (15)}
    ##  [1] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [2] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [3] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [4] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [5] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [6] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [7] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [8] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [9] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [10] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [11] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [12] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [13] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [14] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [15] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...

``` r
## everything before the pipe gonna appear at the dot.
table_marj = 
  (drug_use_xml %>% html_nodes(css = "table")) %>% 
  .[[1]] %>% 
  html_table() %>% 
  slice(-1) %>% 
  as_tibble()
```

## get Hery Potter data

``` r
hpsaga_html = 
  read_html("https://www.imdb.com/list/ls000630791/")

hp_movie_names = 
  hpsaga_html %>% 
  html_nodes(".lister-item-header a") %>% 
  html_text()

hp_movie_runtime = 
  hpsaga_html %>% 
  html_nodes(".runtime") %>% 
  html_text()

hp_movie_money = 
  hpsaga_html %>% 
  html_nodes(".text-small:nth-child(7) span:nth-child(5)") %>% 
  html_text()

hp_df = 
  tibble(
    title = hp_movie_names,
    runtime = hp_movie_runtime,
    money = hp_movie_money
  )
hp_df
```

    ## # A tibble: 8 x 3
    ##   title                                        runtime money   
    ##   <chr>                                        <chr>   <chr>   
    ## 1 Harry Potter and the Sorcerer's Stone        152 min $317.58M
    ## 2 Harry Potter and the Chamber of Secrets      161 min $261.99M
    ## 3 Harry Potter and the Prisoner of Azkaban     142 min $249.36M
    ## 4 Harry Potter and the Goblet of Fire          157 min $290.01M
    ## 5 Harry Potter and the Order of the Phoenix    138 min $292.00M
    ## 6 Harry Potter and the Half-Blood Prince       153 min $301.96M
    ## 7 Harry Potter and the Deathly Hallows: Part 1 146 min $295.98M
    ## 8 Harry Potter and the Deathly Hallows: Part 2 130 min $381.01M

## get napoleon

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
  html_text()

review_text = 
  dynamite_html %>%
  html_nodes(".review-text-content span") %>%
  html_text()

reviews = tibble(
  title = review_titles,
  stars = review_stars,
  text = review_text
)
reviews
```

    ## # A tibble: 10 x 3
    ##    title             stars        text                                     
    ##    <chr>             <chr>        <chr>                                    
    ##  1 Great video       5.0 out of … Product as described.  Great transaction…
    ##  2 Give me some of … 5.0 out of … This movie will always be my favorite mo…
    ##  3 Nostalgic         5.0 out of … One of the best nostalgic movies of my g…
    ##  4 Make you giggle … 5.0 out of … "I love, love, love this movie.  It make…
    ##  5 This movie is so… 5.0 out of … No, really.  It's so stupid.  Your IQ dr…
    ##  6 Hilarious         5.0 out of … Hilarious                                
    ##  7 Waste of money    1.0 out of … Terrible movie! Please don’t waste your …
    ##  8 Good movie        5.0 out of … Funny                                    
    ##  9 A classic         5.0 out of … I like your sleeves. They're real big.   
    ## 10 FRIKKEN SWEET MO… 5.0 out of … It’s Napolean Dynamite. It’s charming, i…

This is the right way of selecting stars from amazon. Don’t use
selectgadget to select stars. `#cm_cr-review_list .review-rating`

## another way to get contents from the web, API

``` r
nyc_water = 
  GET("https://data.cityofnewyork.us/resource/waf7-5gvc.csv") %>% 
  # this code just simply view the reading url as reading a csv file and turn it into a useful format
  content("parsed")
```

    ## Parsed with column specification:
    ## cols(
    ##   year = col_double(),
    ##   new_york_city_population = col_double(),
    ##   nyc_consumption_million_gallons_per_day = col_double(),
    ##   per_capita_gallons_per_person_per_day = col_double()
    ## )

We can also import this dataset as a JSON file. This takes a bit more
work, but it’s still doable.

``` r
nyc_water = 
  GET("https://data.cityofnewyork.us/resource/waf7-5gvc.json") %>% 
  content("text") %>%
  jsonlite::fromJSON() %>%
  as_tibble()
```

``` r
brfss_smart2010 = 
  GET("https://data.cdc.gov/api/views/waxm-p5qv/rows.csv?accessType=DOWNLOAD") %>% 
  content("parsed")
```

    ## Parsed with column specification:
    ## cols(
    ##   .default = col_character(),
    ##   Year = col_double(),
    ##   Locationabbr = col_double(),
    ##   Sample_Size = col_double(),
    ##   Data_value = col_double(),
    ##   Confidence_limit_Low = col_double(),
    ##   Confidence_limit_High = col_double(),
    ##   Display_order = col_double(),
    ##   LocationID = col_double()
    ## )

    ## See spec(...) for full column specifications.

``` r
poke = 
  GET("http://pokeapi.co/api/v2/pokemon/1") %>%
  content()

poke$name
```

    ## [1] "bulbasaur"

``` r
poke$abilities
```

    ## [[1]]
    ## [[1]]$ability
    ## [[1]]$ability$name
    ## [1] "chlorophyll"
    ## 
    ## [[1]]$ability$url
    ## [1] "https://pokeapi.co/api/v2/ability/34/"
    ## 
    ## 
    ## [[1]]$is_hidden
    ## [1] TRUE
    ## 
    ## [[1]]$slot
    ## [1] 3
    ## 
    ## 
    ## [[2]]
    ## [[2]]$ability
    ## [[2]]$ability$name
    ## [1] "overgrow"
    ## 
    ## [[2]]$ability$url
    ## [1] "https://pokeapi.co/api/v2/ability/65/"
    ## 
    ## 
    ## [[2]]$is_hidden
    ## [1] FALSE
    ## 
    ## [[2]]$slot
    ## [1] 1
