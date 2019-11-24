R Quickstart
================
Jesse Cambon
23 November, 2019

Goal: Simple, minimal code for getting started with R.

## Todo

  - Clearly label before and after datasets to illustrate data
    transformations
  - basic minimal ggplots (histogram, point, bar, line)
  - basic modeling - caret, lm, glm

## Setup

``` r
library(tidyverse)
library(ggplot2)

# Set default ggplot theme
theme_set(theme_bw()+
  theme(legend.position = "top",
            plot.subtitle= element_text(face="bold",hjust=0.5),
            plot.title = element_text(lineheight=1, face="bold",hjust = 0.5)))
```

## Data Manipulation

### Long to Wide

Initial Data:

| GEOID | NAME    | variable | estimate | moe |
| :---- | :------ | :------- | -------: | --: |
| 01    | Alabama | income   |    24476 | 136 |
| 01    | Alabama | rent     |      747 |   3 |
| 02    | Alaska  | income   |    32940 | 508 |
| 02    | Alaska  | rent     |     1200 |  13 |

  - pivot\_wider
      - names\_from: column containing values that we will use for our
        new column names

<!-- end list -->

``` r
col_ratio <- us_rent_income %>%
  select(-GEOID,-moe) %>%
  pivot_wider(names_from = variable, values_from = estimate) %>% 
  drop_na() %>%   # drop missing values
  mutate(income_rent_ratio = income / (12*rent))
```

Income and Rent are now in separate columns:

| NAME     | income | rent | income\_rent\_ratio |
| :------- | -----: | ---: | ------------------: |
| Alabama  |  24476 |  747 |            2.730478 |
| Alaska   |  32940 | 1200 |            2.287500 |
| Arizona  |  27517 |  972 |            2.359139 |
| Arkansas |  23789 |  709 |            2.796074 |

### Wide to Long

Initial Data:

| country | indicator   |         2000 |         2001 |         2002 |        2003 |         2004 |         2005 |          2006 |           2007 |           2008 |           2009 |           2010 |         2011 |         2012 |        2013 |         2014 |         2015 |        2016 |         2017 |
| :------ | :---------- | -----------: | -----------: | -----------: | ----------: | -----------: | -----------: | ------------: | -------------: | -------------: | -------------: | -------------: | -----------: | -----------: | ----------: | -----------: | -----------: | ----------: | -----------: |
| ABW     | SP.URB.TOTL | 42444.000000 | 43048.000000 | 43670.000000 | 44246.00000 | 4.466900e+04 | 4.488900e+04 |   4.48810e+04 |   4.468600e+04 |   4.437500e+04 |   4.405200e+04 |   4.377800e+04 | 4.382200e+04 | 4.406400e+04 | 4.43600e+04 | 4.467400e+04 | 4.497900e+04 | 4.52750e+04 | 4.557200e+04 |
| ABW     | SP.URB.GROW |     1.182632 |     1.413021 |     1.434559 |     1.31036 | 9.514777e-01 | 4.913027e-01 | \-1.78233e-02 | \-4.354289e-01 | \-6.984006e-01 | \-7.305493e-01 | \-6.239346e-01 | 1.004566e-01 | 5.507148e-01 | 6.69504e-01 | 7.053514e-01 | 6.804037e-01 | 6.55929e-01 | 6.538489e-01 |
| ABW     | SP.POP.TOTL | 90853.000000 | 92898.000000 | 94992.000000 | 97017.00000 | 9.873700e+04 | 1.000310e+05 |   1.00832e+05 |   1.012200e+05 |   1.013530e+05 |   1.014530e+05 |   1.016690e+05 | 1.020530e+05 | 1.025770e+05 | 1.03187e+05 | 1.037950e+05 | 1.043410e+05 | 1.04822e+05 | 1.052640e+05 |

  - pivot\_longer
      - cols (1st arg): what columns do we want to pivot? (ie. subtract
        ones we don’t want to)
      - names\_to : the name of new column holding the column names as
        values
      - values\_to : name of new column containing values
  - seq(start, stop, increment) -\> generates sequence

<!-- end list -->

``` r
wb_pop <- world_bank_pop %>%
  pivot_longer(c(-country,-indicator), names_to = "year", values_to = "value") %>%
  filter(year %in% seq(2000,2015,5))
```

After:

| country | indicator   | year | value |
| :------ | :---------- | :--- | ----: |
| ABW     | SP.URB.TOTL | 2000 | 42444 |
| ABW     | SP.URB.TOTL | 2005 | 44889 |
| ABW     | SP.URB.TOTL | 2010 | 43778 |

Initial ‘mpg’ Dataset:

| manufacturer | model | displ | year | cyl | trans      | drv | cty | hwy | fl | class   |
| :----------- | :---- | ----: | ---: | --: | :--------- | :-- | --: | --: | :- | :------ |
| audi         | a4    |   1.8 | 1999 |   4 | auto(l5)   | f   |  18 |  29 | p  | compact |
| audi         | a4    |   1.8 | 1999 |   4 | manual(m5) | f   |  21 |  29 | p  | compact |
| audi         | a4    |   2.0 | 2008 |   4 | manual(m6) | f   |  20 |  31 | p  | compact |

## Counting

``` r
count_cyl <- mpg %>%
  count(cyl)
```

| cyl |  n |
| --: | -: |
|   4 | 81 |
|   5 |  4 |
|   6 | 79 |
|   8 | 70 |

## Calculate Summary Stats

``` r
mpg_stats <- mpg %>% select(class,hwy) %>%
  mutate(class_c=case_when(class %in% c("2seater","compact") ~ "compact",
                               TRUE ~ class)) %>%
  group_by(class_c) %>%
  summarize(count=n(),
            max_hwy=max(hwy),
            min_hwy=min(hwy),
            median_hwy=median(hwy),
            mean_hwy=mean(hwy)) %>%
  ungroup() %>%
  arrange(desc(count)) # sort dataset
```

A new class variable that combines ‘2seater’ and ‘compact’ is created

| class\_c   | count | max\_hwy | min\_hwy | median\_hwy | mean\_hwy |
| :--------- | ----: | -------: | -------: | ----------: | --------: |
| suv        |    62 |       27 |       12 |        17.5 |  18.12903 |
| compact    |    52 |       44 |       23 |        27.0 |  27.96154 |
| midsize    |    41 |       32 |       23 |        27.0 |  27.29268 |
| subcompact |    35 |       44 |       20 |        26.0 |  28.14286 |
| pickup     |    33 |       22 |       12 |        17.0 |  16.87879 |
| minivan    |    11 |       24 |       17 |        23.0 |  22.36364 |

``` r
## Drop missing height and weight values for scatter plot
# Also drop Jabba and Yoda because they are outliers
starwars_ht_wt <- starwars_jac %>% drop_na(c(height,mass,gender)) %>%
  filter(!str_detect(name,'Jabba|Yoda')) %>% 
  filter(num_films >= 3) 

### Crime Data

murder_rates <- USArrests %>% 
  rownames_to_column('State') %>%
  as_tibble() %>%
  arrange(desc(UrbanPop)) %>%
  head(15) %>%
  arrange(desc(Murder)) %>% 
  mutate(State=factor(State,levels=rev(State)))

### Stock Data
eu_stock <- EuStockMarkets %>% 
  as_tibble() %>%
  gather(Index,Price) %>%
  mutate(Year=rep(time(EuStockMarkets),4)) 
```

``` r
# Histogram with autobinning based on gender
ggplot(starwars_jac %>% replace_na(list(gender='None')), aes(height)) + #scale_fill_manual(values = wes_palette('Moonrise2')) +
  geom_histogram(aes(fill=gender), 
                   binwidth = 10, 
                   col="black") +
            #       size=.1) +  # change binwidth
  # remove bottom inner margin with expand
scale_y_continuous(expand = c(0,0,0.08,0)) + 
  labs(title="Height Distribution of Star Wars Characters", 
       caption="Han Shot First") +
xlab('Height (cm)') +
ylab('Count') +
guides(fill = guide_legend(title='Gender'))
```

## Lollipop

``` r
  ggplot(data=murder_rates, aes(x=State, y=Murder) ) +
    geom_segment( aes(x=State ,xend=State, y=0, yend=Murder), color="grey") +
    geom_point(size=3, color="navy") +
   theme_minimal() +
  theme(    plot.subtitle= element_text(face="bold",hjust=0.5),
            plot.title = element_text(lineheight=1, face="bold",hjust = 0.5),
      panel.grid.minor.y = element_blank(),
      panel.grid.major.y = element_blank(),
      panel.grid.minor.x = element_blank(),
      legend.position="none"
    ) +
  coord_flip() +
    # expand gets rid of space between labels stem of lollipops
    scale_y_continuous(expand = c(0, .15)) + 
    labs(title='Murder Rates of Selected States - 1975',
        caption='Data: World Almanac and Book of facts 1975. (Crime rates)') +
    xlab("") +
    ylab('Murders Per 100,000 Residents')
```

## Line

``` r
# Start and end for the breaks on the horizontal axis
eu_plot_lims <- c(ceiling(min(eu_stock$Year)),floor(max(eu_stock$Year)))

# Performance of EU Stock Indexes
ggplot(eu_stock,
          aes(x=Year,y=Price,color = fct_rev(Index))) +
geom_line() +
scale_x_continuous(breaks=eu_plot_lims[1]:eu_plot_lims[2],
                   expand=c(0,0,0.02,0)) +
scale_y_continuous(labels=scales::comma) + 
#scale_color_manual(values=wes_palette('GrandBudapest2')) +
labs(title='EU Stock Indexes',
     caption='Data provided by Erste Bank AG, Vienna, Austria') +
theme(legend.title = element_blank(),
      panel.grid.minor.x = element_blank(),
      legend.text=element_text(size=10),
      legend.position='right') +
xlab('Year') +
ylab('Value') +
# make legend lines bigger
guides(colour = guide_legend(override.aes = list(size=2.5))) 
```