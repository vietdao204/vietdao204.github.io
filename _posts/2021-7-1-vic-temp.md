---
layout: post
title: Predict air temperatures in Victoria with seasonal ARIMA
---

****
View the [code](https://github.com/vietdao204/vic-temp/blob/main/main.Rmd){:target='_blank'} and [data](https://github.com/vietdao204/vic-temp/blob/main/data/victemp.csv){:target='_blank'}

****

## Intro
Air temperature is an environmental measure that has been widely studied because it reflects the balance of ecosystems. Therefore, air temperature is deemed a prominent indicator of climate change. Many studies suggest that global temperatures are changing but the scale and impact of this change is different depending on the location of interest [1]. I aim to see whether there has been a change in temperatures in my city of Victoria, British Columbia using time series analysis in the last two decades and conveniently make some future predictions.

The data are retrieved from the National Centers for Environment Information through their Climate Data Online Search [2] and contains the average monthly air temperatures recorded at the University of Victoria station from December 1992 to February 2021. Three missing temperatures for October 2003, January 2020, and February 2020 are filled by values from the Esquimalt Harbour station in Victoria for October 2013 and the James Bay station in Victoria for January and February 2020. Esquimalt station's data is also provided by NOAA and James Bay station's data is from the Government of Canada's Historical Climate data portal [3]. The assumption here is that the temperature readings at these two stations for the months aforementioned would be approximately the same as those at the University of Victoria station, if had been available.

The average monthly temperature is computed by adding the unrounded monthly maximum and minimum temperatures and dividing by 2.

## Analysis
The data is plotted in Figure 1. Temperatures fluctuate depending on the time of year of course but don't look different year over year.
```
library(astsa)
# create time series, graph it
vicTemp <- ts(vicTemp$TAVG, start=c(1992,12), end=c(2021,2), frequency = 12) 
plot.ts(vicTemp, ylab = 'Celcius', main = 'Figure 1. Average monthly air temperatures in Victoria') # Fig1
```
![Fig1.png]({{ site.baseurl }}/images/vic-temp/Fig1.png "Fig1.png")
To make the series stationary, apply the first difference and the first seasonal difference. Figure 2 displays the transformed series.
```
# transform the series (take the first difference and first seasonal difference)
tsplot(diff(diff(vicTemp),12), main = "Figure 2. Transformed series", 
       ylab = "Celcius")  # Fig2
abline(h=0, col=2)
```
![Fig2.png]({{ site.baseurl }}/images/vic-temp/Fig2.png "Fig2.png")

## Model
Figure 3 presents the ACF and PACF plots of the transformed series, showing that the ACF cuts off after lag 1 at seasonal lags (Q=1) and the PACF tails off at seasonal lags (P=0).
```
acf2(diff(diff(vicTemp),12), 100, main = "Figure 3. ACF and PACF of transformed series") # Fig3
```
![Fig3.png]({{ site.baseurl }}/images/vic-temp/Fig3.png "Fig3.png")
d = 1, P = 0, D = 1, Q = 1, s = 12 are reasonable parameters, telling from the transformation, ACF, and PACF plots. However, it is not clear what p and q are so after trying various values for them in the fitted models and examine the residual analysis, the final resulted model is (with p=q=1): \\[ ARIMA(1, 1, 1)\times(0,1,1)_{12} \\]
The residuals plots for this model are presented in Figure 4. The residuals are normally distributed and uncorrelated. There are a few outliers. The coefficient estimates are within the unit circle and significant (p-values \\(\approx\\) 0), indicating the model is causal and can be used for forecasting. 
```
# p = 1, d = 1, q = 1, P = 0,  D = 1, Q = 1, s = 12
sarima(vicTemp, 1,1,1, 0,1,1, 12) # Fig4
```
**R Output**
![arima-output.png]({{ site.baseurl }}/images/vic-temp/arima-output.png "arima-output.png")

<div style="text-align:center"><b>Figure 4. Residuals plots</b></div>
![Fig4.png]({{ site.baseurl }}/images/vic-temp/Fig4.png "Fig4.png")

## Predictions
```
fore <- sarima.for(vicTemp,10, 1,1,1, 0,1,1, 12) #Fig5 & Table1
title(main="Figure 5. Predictions from March 2021 to December 2021", 
      ylab="                            (Celcius)")
```
![Fig5.png]({{ site.baseurl }}/images/vic-temp/Fig5.png "Fig5.png")

<table class=" lightable-material lightable-striped lightable-hover" style="margin-left: auto; margin-right: auto;border-bottom: 0;">
                <thead>
                    <tr>
                        <th style="text-align:left;">
                        Months
                        </th>
                        <th style="text-align:right;">
                        Predictions
                        </th>
                        <th style="text-align:right;">
                        Lower.Bound
                        </th>
                        <th style="text-align:right;">
                        Upper.Bound
                        </th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <td style="text-align:left;">
                        03/2021
                        </td>
                        <td style="text-align:right;">
                        6.905727
                        </td>
                        <td style="text-align:right;">
                        4.855140
                        </td>
                        <td style="text-align:right;">
                        8.956313
                        </td>
                    </tr>
                    <tr>
                        <td style="text-align:left;">
                        04/2021
                        </td>
                        <td style="text-align:right;">
                        9.378582
                        </td>
                        <td style="text-align:right;">
                        7.239189
                        </td>
                        <td style="text-align:right;">
                        11.517974
                        </td>
                    </tr>
                    <tr>
                        <td style="text-align:left;">
                        05/2021
                        </td>
                        <td style="text-align:right;">
                        12.566957
                        </td>
                        <td style="text-align:right;">
                        10.399623
                        </td>
                        <td style="text-align:right;">
                        14.734291
                        </td>
                    </tr>
                    <tr>
                        <td style="text-align:left;">
                        06/2021
                        </td>
                        <td style="text-align:right;">
                        15.101879
                        </td>
                        <td style="text-align:right;">
                        12.914040
                        </td>
                        <td style="text-align:right;">
                        17.289718
                        </td>
                    </tr>
                    <tr>
                        <td style="text-align:left;">
                        07/2021
                        </td>
                        <td style="text-align:right;">
                        17.388745
                        </td>
                        <td style="text-align:right;">
                        15.181762
                        </td>
                        <td style="text-align:right;">
                        19.595728
                        </td>
                    </tr>
                    <tr>
                        <td style="text-align:left;">
                        08/2021
                        </td>
                        <td style="text-align:right;">
                        17.328202
                        </td>
                        <td style="text-align:right;">
                        15.102447
                        </td>
                        <td style="text-align:right;">
                        19.553957
                        </td>
                    </tr>
                    <tr>
                        <td style="text-align:left;">
                        09/2021
                        </td>
                        <td style="text-align:right;">
                        14.685160
                        </td>
                        <td style="text-align:right;">
                        12.440826
                        </td>
                        <td style="text-align:right;">
                        16.929495
                        </td>
                    </tr>
                    <tr>
                        <td style="text-align:left;">
                        10/2021
                        </td>
                        <td style="text-align:right;">
                        10.287518
                        </td>
                        <td style="text-align:right;">
                        8.024749
                        </td>
                        <td style="text-align:right;">
                        12.550287
                        </td>
                    </tr>
                    <tr>
                        <td style="text-align:left;">
                        11/2021
                        </td>
                        <td style="text-align:right;">
                        7.205826
                        </td>
                        <td style="text-align:right;">
                        4.924722
                        </td>
                        <td style="text-align:right;">
                        9.486930
                        </td>
                    </tr>
                    <tr>
                        <td style="text-align:left;">
                        12/2021
                        </td>
                        <td style="text-align:right;">
                        5.204867
                        </td>
                        <td style="text-align:right;">
                        2.905928
                        </td>
                        <td style="text-align:right;">
                        7.503805
                        </td>
                    </tr>
                </tbody>
                <tfoot>
                    <tr>
                        <td style="padding: 0; " colspan="100%">
                            <sup></sup>
                             <b>Table 1. Predicted average temperatures and their 95% confidence intervals for the next 10 monhts.</b>
                        </td>
                    </tr>
                </tfoot>
            </table>

The recorded average temperature of March 2021 is 6.2 degree Celcius [Government of Canada, 2021], which is close to 6.9 and within the (4.9, 9.0) confidence interval as predicted. I intend to come back later to compare these predictions with the acutal recorded values. It is important to note the incompleteness of the original University of Victoria data. Backfilling using data from other sources (Esquimalt Harbour and James Bay stations) may interfere with data accuracy because temperature readings from different stations, while in the same city, are not necessarily be the same. With that aside however, the data shows that average temperatures over the years are fairly stable, which is definitely a good sign. 

## References
[1] Kennedy, C. (2020, October 29). Does "global warming" mean it's warming everywhere? Retrieved April 10, 2021, from <https://www.climate.gov/news-features/climate-qa/does-global-warming-mean-it%E2%80%99s-warming-everywhere>{:target='_blank'}

[2] NOAA (2021). Global Summary of the Month Station Details [Data set]. <https://www.ncdc.noaa.gov/cdo-web/datasets/GSOM/stations/GHCND:CA001018598/detail>{:target='_blank'}

[3] Government of Canada (2021). Historical Data [Data set]. <https://climate.weather.gc.ca/historical_data/search_historic_data_e.html>{:target='_blank'}

****
