---
layout: post
title: Mashing CDC and TSA Data
image: /images/cdc-dot-mashup.png
date: 2021-12-26 06:00
tag: Airline Travel Is Correlated with COVID Cases
categories: [data, statistics, covid, cdc, tsa, R]
---
[COVID spreads when infected people breathe, potentially spreading it to others sharing space with them.](https://www.cdc.gov/coronavirus/2019-ncov/prevent-getting-sick/how-covid-spreads.html) It shouldn't be a surprise that when infected people travel, they help to spread COVID. In this post, I take airline travel data from the [US Department of Transportation](https://data.transportation.gov/Aviation/Consumer-Airfare-Report-Table-6-Contiguous-State-C/yj5y-b2ir) and mash it with [COVID 19 case/death data](https://data.cdc.gov/Case-Surveillance/United-States-COVID-19-Cases-and-Deaths-by-State-o/9mfq-cb36) and [Vaccination data](https://data.cdc.gov/Vaccinations/COVID-19-Vaccinations-in-the-United-States-Jurisdi/unsk-b7fc) from the CDC to show that variations in state-to-state US air travel are correlated with COVID 19 case and death prevalence.

The analysis suggests that, all else equal, an infected passenger transiting from one state to another is correlated with roughly a 38 COVID case increase in the following quarter.

# DoT Consumer Airfare Data
The Department of Transportation provides a quarterly Consumer Airfare Report containing passenger counts for "Contiguous State City-Pair Markets That Average At Least 10 Passengers Per Day." You can access the DoT data export [here](https://data.transportation.gov/Aviation/Consumer-Airfare-Report-Table-6-Contiguous-State-C/yj5y-b2ir). I've cleaned this data and made it available [here](https://media.githubusercontent.com/media/JLospinoso/covid-airlines/main/travel-out.csv) as well.

# CDC Case/Death and Vaccination Data
The CDC provides [COVID 19 case/death data](https://data.cdc.gov/Case-Surveillance/United-States-COVID-19-Cases-and-Deaths-by-State-o/9mfq-cb36) as well as [Vaccination data](https://data.cdc.gov/Vaccinations/COVID-19-Vaccinations-in-the-United-States-Jurisdi/unsk-b7fc). I've massaged this into quarterly data with the same state abbreviation codes as the DoT Consumer Airfare Data. I've also differenced the case and death data by quarter, meaning that you can model increases rather than cumulative values. These transformed datasets are available [here](https://media.githubusercontent.com/media/JLospinoso/covid-airlines/main/cases-out.csv) and [here](https://media.githubusercontent.com/media/JLospinoso/covid-airlines/main/vaccinations-out.csv).

# Wikipedia State Populations
Finally, I've pulled state population data from [Wikipedia](https://simple.wikipedia.org/wiki/List_of_U.S._states_by_population). I used linear interpolation to model quarterly state-level populations in the data frame [here](https://github.com/JLospinoso/covid-airlines/blob/main/populations-out.csv).

# All Together and the Aggregate Inbound Statistic
Joining these datasets, I created a panel of the eight quarters in 2020 and 2021 for each state containing the following columns:

- year/quarter
- state code
- state population
- covid_cases/covid_deaths

I also compute two columns I call "Aggregate Inbound Cases/Deaths". These columns model infected passengers traveling between a particular state and all the other states in the dataset. Say we're looking at California's 2Q 2021. To compute Aggregate Inbound Cases, you can sum the following statistic for all *other* states' 1Q 2021 data:

```
(Passengers between CA and State_X) * (State_X change in COVID Cases) / (State_X population)
```

The idea is that, very roughly, `(State_X change in COVID Cases) / (State_X population)` should be correlated with the percent of COVID-infected people residing in State_X. If you multiply that by the amount of passengers traveling between California and State_X, you get rough measure of how many infected people traveled between these two states. You can think of this statistic as representing "how many infected passengers transit between these states per day."

This panel is available [here](https://media.githubusercontent.com/media/JLospinoso/covid-airlines/main/panel.csv).

# Summarizing the Panel

<svg>
  <circle style="fill: #69b3a2" stroke="black" cx=50 cy=50 r=40></circle>
</svg>

# Modeling the Aggregate Inbound Effect

Our task is to determine the impact of Aggregate Inbound Cases/Deaths on a state's next quarter COVID case/death count. [ARIMA](https://en.wikipedia.org/wiki/Autoregressive_integrated_moving_average) models are the go-to statistical tool for panel data. You can use them to determine the correlation between Aggregate Inbound Cases/Deaths on a state's next quarter COVID case/death count. You can get all set up in R:

```R
install.packages(c("plm", "zoo"))
library(zoo)
library(plm)
```

[plm](https://cran.r-project.org/web/packages/plm/plm.pdf) is a widely used package for generalized linear modeling in panel data, and [zoo](https://cran.r-project.org/web/packages/zoo/index.html) is another widely used package for totally ordered data (like our panel). Lets read our panel into a data frame `d`, and add a `ts` column corresponding to the year/quarter as a zoo `yearqtr`:

```R
d = read.csv("panel-out.csv")
d$ts = as.yearqtr(paste(d$year, d$quarter, sep='-'))
```

You can fit a linear model of new covid cases as dependent upon:

- aggregate inbound cases
- vaccinated population
- population

This will tell us how variations in COVID cases/vaccinations are correlated with new COVID cases.

You include population here as well, since states with larger populations will likely have more new COVID cases, all else equal. Let's also include [random effects](https://en.wikipedia.org/wiki/Random_effects_model) to control for peculiarities at the state and time level, as well as [time dummies]()

```R
summary(re.cases <- plm(diff(covid_cases) ~ aggregate_inbound_cases
                        + vaccinated + population + ts,
                  index=c("state", "ts"), data=d, model="random"))
```

Here are the results:

```
Oneway (individual) effect Random Effect Model
   (Swamy-Arora's transformation)

Call:
plm(formula = diff(covid_cases) ~ aggregate_inbound_cases + vaccinated +
    population + ts, data = d, model = "random", index = c("state",
    "ts"))

Balanced Panel: n = 49, T = 7, N = 343

Effects:
                    var   std.dev share
idiosyncratic 2.033e+13 4.508e+06     1
individual    0.000e+00 0.000e+00     0
theta: 0

Residuals:
     Min.   1st Qu.    Median   3rd Qu.      Max.
-18365898  -1921325     82615   1560244  31458011

Coefficients:
                           Estimate  Std. Error z-value  Pr(>|z|)    
(Intercept)             -3.0590e+06  6.7411e+05 -4.5378 5.684e-06 ***
aggregate_inbound_cases  3.7818e+01  1.6525e+01  2.2885   0.02211 *  
vaccinated              -7.3144e-01  9.8806e-02 -7.4028 1.334e-13 ***
population               6.3323e-01  4.4075e-02 14.3669 < 2.2e-16 ***
ts2020 Q3                1.4868e+06  8.5641e+05  1.7360   0.08256 .  
ts2020 Q4                5.1997e+06  8.5860e+05  6.0560 1.396e-09 ***
ts2021 Q1                6.7820e+06  8.7406e+05  7.7591 8.553e-15 ***
ts2021 Q2                2.1933e+06  9.6623e+05  2.2700   0.02321 *  
ts2021 Q3                6.7389e+06  9.4731e+05  7.1138 1.129e-12 ***
ts2021 Q4                3.1076e+05  9.7136e+05  0.3199   0.74903    
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Total Sum of Squares:    1.3908e+16
Residual Sum of Squares: 5.9807e+15
R-Squared:      0.56998
Adj. R-Squared: 0.55835
Chisq: 441.374 on 9 DF, p-value: < 2.22e-16
```

OK - let's interpret these results:

- The Aggregate Inbound Cases coefficient is 37.8: Each additional infected passenger transiting to this state per day is correlated with an additional 37.8 COVID case increase per quarter.

- The vaccinated coefficient is -0.731: Each additional vaccinated individual is correlated with a reduction of 0.731 COVID cases per quarter. (Interesting that this interpretation is ballpark similar to [COVID vaccine effectiveness](https://www.cdc.gov/mmwr/covid19_vaccine_safety.html).)

- The population coefficient is 0.633: Each additional state resident is correlated with a 0.633 COVID case increase per quarter.

- There are significant "time dummy" estimates, meaning there's substantial time-based variation. This is perhaps unsurprising as social distancing protocols/virus variants/COVID fatigue create unique circumstances.

*Note 1: There's a lot of work we'd need to do on this model before we could establish causality between air travel and increased COVID cases, even though the relationship seems intuitive. We'd also need to do some work on the linear model itself, as it suffers from [stationarity issues](https://en.wikipedia.org/wiki/Ljung–Box_test):*

```R
> Box.test(re.cases$residuals)

	Box-Pierce test

data:  re.cases$residuals
X-squared = 29.721, df = 1, p-value = 4.988e-08
```
* Note 2: I tried computing Eigenvector Centrality of each state using the weighted, undirected graph of passenger data per day. This statistic is highly colinear with state population. When removing population, the adjusted R squared drops from .56 to .31.

```R
> summary(re.cases.cent <- plm(diff(covid_cases) ~ eigenvector_centrality
+                         + aggregate_inbound_cases
+                         + vaccinated + ts,
+                         index=c("state", "ts"), data=d, model="random"))
Oneway (individual) effect Random Effect Model
   (Swamy-Arora's transformation)

Call:
plm(formula = diff(covid_cases) ~ eigenvector_centrality + aggregate_inbound_cases +
    vaccinated + ts, data = d, model = "random", index = c("state",
    "ts"))

Balanced Panel: n = 49, T = 7, N = 343

Effects:
                    var   std.dev share
idiosyncratic 1.987e+13 4.457e+06     1
individual    0.000e+00 0.000e+00     0
theta: 0

Residuals:
     Min.   1st Qu.    Median   3rd Qu.      Max.
-22626452  -2207206    -99294   1441738  43419127

Coefficients:
                           Estimate  Std. Error z-value  Pr(>|z|)    
(Intercept)              2.0707e+05  8.0160e+05  0.2583  0.796159    
eigenvector_centrality   1.3385e+07  3.6213e+06  3.6961  0.000219 ***
aggregate_inbound_cases  3.2926e+01  2.5923e+01  1.2702  0.204018    
vaccinated               2.0942e-01  9.4237e-02  2.2222  0.026268 *  
ts2020 Q3                1.4630e+06  1.0683e+06  1.3694  0.170873    
ts2020 Q4                5.2135e+06  1.0727e+06  4.8601 1.174e-06 ***
ts2021 Q1                5.8402e+06  1.0931e+06  5.3426 9.163e-08 ***
ts2021 Q2               -8.9834e+05  1.1897e+06 -0.7551  0.450192    
ts2021 Q3                1.9559e+06  1.1681e+06  1.6745  0.094036 .  
ts2021 Q4               -4.9816e+06  1.1866e+06 -4.1982 2.690e-05 ***
---
Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

Total Sum of Squares:    1.3908e+16
Residual Sum of Squares: 9.306e+15
R-Squared:      0.33088
Adj. R-Squared: 0.31279
Chisq: 164.668 on 9 DF, p-value: < 2.22e-16
```

# Conclusion

Putting CDC and Department of Transportation data together, we can demonstrate correlation between air travel and increased COVID cases. If you want to re-create/modify the data cleaning or analysis, you can check it out for yourself at [https://github.com/JLospinoso/covid-airlines](https://github.com/JLospinoso/covid-airlines).
