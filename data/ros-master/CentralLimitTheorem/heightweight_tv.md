Regression and Other Stories: Heights and weights
================
Andrew Gelman, Jennifer Hill, Aki Vehtari
2021-04-20

-   [3 Some basic methods in mathematics and
    probability](#3-some-basic-methods-in-mathematics-and-probability)
    -   [3.5 Probability distributions](#35-probability-distributions)
        -   [Mean and standard deviation of a probability
            distribution](#mean-and-standard-deviation-of-a-probability-distribution)
        -   [Normal distribution; mean and standard
            deviation](#normal-distribution-mean-and-standard-deviation)
        -   [Lognormal distribution](#lognormal-distribution)

Tidyverse version by Bill Behrman.

Height and weight distributions of women and men illustrating central
limit theorem and normal distribution. See Chapter 3 in Regression and
Other Stories.

------------------------------------------------------------------------

``` r
# Packages
library(tidyverse)

# Parameters
  # Common code
file_common <- here::here("_common.R")

#===============================================================================

# Run common code
source(file_common)
```

# 3 Some basic methods in mathematics and probability

## 3.5 Probability distributions

### Mean and standard deviation of a probability distribution

Data

``` r
heights <- 
  tibble(
    height = 54:75,
    men = 
      c(
        0, 0, 0, 0, 0, 0, 0, 542, 668, 1221, 2175, 4213, 5535, 7980, 9566, 9578,
        8867, 6716, 5019, 2745, 1464, 1263
      ) * 9983 / 67552,
    women =
      c(
        80, 107, 296, 695, 1612, 2680, 4645, 8201, 9948, 11733, 10270, 9942, 
        6181, 3990, 2131, 1154, 245, 257, 0, 0, 0, 0
      ) * 10339 / 74167
  )
```

Normal approximations of heights.

``` r
height_men_mean <- weighted.mean(heights$height, heights$men)
height_men_sd <- sqrt(Hmisc::wtd.var(heights$height, heights$men))
height_women_mean <- weighted.mean(heights$height, heights$women)
height_women_sd <- sqrt(Hmisc::wtd.var(heights$height, heights$women))

norm_approx <- 
  tibble(
    x = seq_range(c(min(heights$height), max(heights$height))),
    y_men = 
      dnorm(x, mean = height_men_mean, sd = height_men_sd) * sum(heights$men),
    y_women = 
      dnorm(x, mean = height_women_mean, sd = height_women_sd) *
      sum(heights$women)
  )
```

Heights of women.

``` r
heights %>% 
  ggplot() +
  geom_blank(aes(height, pmax(men, women))) +
  geom_col(aes(height, women)) +
  geom_line(aes(x, y_women), data = norm_approx, color = "red") +
  labs(
    title = "Heights of women",
    subtitle = "With normal approximation in red",
    x = "Height (inches)",
    y = "Count"
  )
```

<img src="heightweight_tv_files/figure-gfm/unnamed-chunk-4-1.png" width="100%" />

Heights of men.

``` r
heights %>% 
  ggplot() +
  geom_blank(aes(height, pmax(men, women))) +
  geom_col(aes(height, men)) +
  geom_line(aes(x, y_men), data = norm_approx, color = "red") +
  labs(
    title = "Heights of men",
    subtitle = "With normal approximation in red",
    x = "Height (inches)",
    y = "Count"
  )
```

<img src="heightweight_tv_files/figure-gfm/unnamed-chunk-5-1.png" width="100%" />

### Normal distribution; mean and standard deviation

Heights of all adults.

``` r
heights %>% 
  ggplot() +
  geom_col(aes(height, men + women)) +
  geom_line(aes(x, y_men + y_women), data = norm_approx, color = "red") +
  labs(
    title = "Heights of all adults",
    subtitle = "Not a normal distribution",
    x = "Height (inches)",
    y = "Count"
  )
```

<img src="heightweight_tv_files/figure-gfm/unnamed-chunk-6-1.png" width="100%" />

Normal distribution with mean 0 and standard deviation 1.

``` r
v <- 
  tibble(
    x = seq(-4, 4, length.out = 641),
    y = dnorm(x),
    group = if_else(near(abs(x), 4), x - sign(x), trunc(x))
  )

labels <- 
  tibble(
    x = c(-1.5, 0, 1.5),
    y = c(0.3, 0.35, 0.3) * dnorm(x),
    label = c("13.6%", "68.3%", "13.6%")
  )

fill_colors <- 
  c(
    "0" = "grey70",
    "1" = "grey50",
    "2" = "grey30",
    "3" = "grey10"
  )

v %>% 
  ggplot(aes(x, y)) +
  geom_area(aes(fill = factor(abs(group)), group = group)) +
  geom_line() +
  geom_segment(
    aes(x = x, xend = x, y = 0, yend = y),
    data = v %>% filter(abs(x) %in% 1:3)
  ) +
  geom_text(aes(label = label), data = labels) +
  scale_x_continuous(breaks = scales::breaks_width(1)) +
  scale_y_continuous(breaks = 0) +
  scale_fill_manual(values = fill_colors) +
  theme(legend.position = "none") +
  labs(
    title = "Normal distribution with mean 0 and standard deviation 1",
    x = NULL,
    y = NULL
  )
```

<img src="heightweight_tv_files/figure-gfm/unnamed-chunk-7-1.png" width="100%" />

### Lognormal distribution

``` r
weight_men_meanlog <- 5.13
weight_men_sdlog <- 0.17
```

Normal approximation of log weights of men.

``` r
v <- 
  tibble(
    x = seq_range(weight_men_meanlog + c(-3, 3) * weight_men_sdlog),
    y = dnorm(x, mean = weight_men_meanlog, sd = weight_men_sdlog)
  )

v %>% 
  ggplot(aes(x, y)) +
  geom_line() +
  scale_x_continuous(breaks = scales::breaks_width(0.2)) +
  scale_y_continuous(breaks = 0) +
  labs(
    title = "Normal approximation of log weights of men",
    x = "Log of weight in pounds",
    y = NULL
  )
```

<img src="heightweight_tv_files/figure-gfm/unnamed-chunk-9-1.png" width="100%" />

Lognormal approximation of weights of men.

``` r
v <- 
  tibble(
    x = seq_range(exp(weight_men_meanlog + c(-3, 3) * weight_men_sdlog)),
    y = dlnorm(x, meanlog = weight_men_meanlog, sdlog = weight_men_sdlog)
  )

v %>% 
  ggplot(aes(x, y)) +
  geom_line() +
  scale_x_continuous(breaks = scales::breaks_width(20)) +
  scale_y_continuous(breaks = 0) +
  labs(
    title = "Lognormal approximation of weights of men",
    x = "Weight in pounds",
    y = NULL
  )
```

<img src="heightweight_tv_files/figure-gfm/unnamed-chunk-10-1.png" width="100%" />
