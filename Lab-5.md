---
title: "Lab 5"
author: "Juehan Wang"
date: "9/24/2021"
output:
    html_document:
      toc: yes 
      toc_float: yes 
      keep_md: yes
    github_document:
      keep_html: true
      html_preview: false
always_allow_html: true
---





## Setup in R

Load the data.table (and the dtplyr and dplyr packages if you plan to work with those).

Load the met data from https://raw.githubusercontent.com/USCbiostats/data-science-data/master/02_met/met_all.gz, and also the station data. For the later, you can use the code we used during lecture to pre-process the stations data:


```r
# Download the data
stations <- fread("ftp://ftp.ncdc.noaa.gov/pub/data/noaa/isd-history.csv")
stations[, USAF := as.integer(USAF)]
```

```
## Warning in eval(jsub, SDenv, parent.frame()): NAs introduced by coercion
```

```r
# Dealing with NAs and 999999
stations[, USAF   := fifelse(USAF == 999999, NA_integer_, USAF)]
stations[, CTRY   := fifelse(CTRY == "", NA_character_, CTRY)]
stations[, STATE  := fifelse(STATE == "", NA_character_, STATE)]

# Selecting the three relevant columns, and keeping unique records
stations <- unique(stations[, list(USAF, CTRY, STATE)])

# Dropping NAs
stations <- stations[!is.na(USAF)]

# Removing duplicates
stations[, n := 1:.N, by = .(USAF)]
stations <- stations[n == 1,][, n := NULL]
```


```r
if (!file.exists("met_all.gz"))
  download.file(
    url = "https://raw.githubusercontent.com/USCbiostats/data-science-data/master/02_met/met_all.gz",
    destfile = "met_all.gz",
    method   = "libcurl",
    timeout  = 60
    )
met <- data.table::fread("met_all.gz")
```


```r
met <- merge(
  x = met,
  y = stations,
  all.x = TRUE, all.y = FALSE,
  by.x = "USAFID", by.y = "USAF"
)
```


### Question 1: Representative station for the US

What is the median station in terms of temperature, wind speed, and atmospheric pressure? Look for the three weather stations that best represent continental US using the quantile() function. Do these three coincide?

Knit the document, commit your changes, and Save it on GitHub. Don’t forget to add README.md to the tree, the first time you render it.


```r
station_averages <- met[,.(
  temp = mean(temp, na.rm = TRUE),
  wind.sp = mean(wind.sp, na.rm = TRUE),
  atm.press = mean(atm.press, na.rm = TRUE)
), by = USAFID]
```


```r
medians <- station_averages[, .(
  temp_50 = quantile(temp, probs = .5, na.rm = TRUE),
  wind.sp_50 = quantile(wind.sp, probs = .5, na.rm = TRUE),
  atm.press_50 = quantile(atm.press, probs = .5, na.rm = TRUE)
)] ;medians
```

```
##     temp_50 wind.sp_50 atm.press_50
## 1: 23.68406   2.461838     1014.691
```

Now we can find stations that are the closest to these. (hint: `which.min()`).


```r
station_averages[, temp_dist := abs(temp - medians$temp_50)]
median_temp_station <- station_averages[order(temp_dist)][1]; median_temp_station
```

```
##    USAFID     temp  wind.sp atm.press   temp_dist
## 1: 720458 23.68173 1.209682       NaN 0.002328907
```

The median temperature station is 720458.


```r
station_averages[, wind.sp_dist := abs(temp - medians$temp_50)]
median_wind.sp_station <- station_averages[order(wind.sp_dist)][1]; median_wind.sp_station
```

```
##    USAFID     temp  wind.sp atm.press   temp_dist wind.sp_dist
## 1: 720458 23.68173 1.209682       NaN 0.002328907  0.002328907
```

The median wind speed station is 720458.


```r
station_averages[, atm.press_dist := abs(atm.press - medians$atm.press_50)]
median_atm.press_station <- station_averages[order(atm.press_dist)][1]; median_atm.press_station
```

```
##    USAFID     temp  wind.sp atm.press temp_dist wind.sp_dist atm.press_dist
## 1: 722238 26.13978 1.472656  1014.691  2.455719     2.455719   0.0005376377
```

The median atmospheric pressure station is 722238.

### Question 2: Representative station per state

Just like the previous question, you are asked to identify what is the most representative, the median, station per state. This time, instead of looking at one variable at a time, look at the euclidean distance. If multiple stations show in the median, select the one located at the lowest latitude.

Knit the doc and save it on GitHub.


We first need to recover the state variable, by MERGING.


```r
station_averages[, temp_50 := quantile(temp, probs = .5, na.rm = TRUE)]
station_averages <- merge(
  x = station_averages, y = stations, 
  by.x = "USAFID", by.y = "USAF", 
  all.x = TRUE, all.y = FALSE
  ); head(station_averages)
```

```
##    USAFID     temp  wind.sp atm.press temp_dist wind.sp_dist atm.press_dist
## 1: 690150 33.18763 3.483560  1010.379 9.5035752    9.5035752       4.312471
## 2: 720110 31.22003 2.138348       NaN 7.5359677    7.5359677            NaN
## 3: 720113 23.29317 2.470298       NaN 0.3908894    0.3908894            NaN
## 4: 720120 27.01922 2.504692       NaN 3.3351568    3.3351568            NaN
## 5: 720137 21.88823 1.979335       NaN 1.7958292    1.7958292            NaN
## 6: 720151 27.57686 2.998428       NaN 3.8928051    3.8928051            NaN
##     temp_50 CTRY STATE
## 1: 23.68406   US    CA
## 2: 23.68406   US    TX
## 3: 23.68406   US    MI
## 4: 23.68406   US    SC
## 5: 23.68406   US    IL
## 6: 23.68406   US    TX
```

Now we can compute the median per state.


```r
station_averages[, temp_50 := quantile(temp, probs = .5, na.rm = TRUE), by = STATE]
station_averages[, wind.sp_50 := quantile(wind.sp, probs = .5, na.rm = TRUE), by = STATE]
station_averages[, atm.press_50 := quantile(atm.press, probs = .5, na.rm = TRUE), by = STATE]
head(station_averages)
```

```
##    USAFID     temp  wind.sp atm.press temp_dist wind.sp_dist atm.press_dist
## 1: 690150 33.18763 3.483560  1010.379 9.5035752    9.5035752       4.312471
## 2: 720110 31.22003 2.138348       NaN 7.5359677    7.5359677            NaN
## 3: 720113 23.29317 2.470298       NaN 0.3908894    0.3908894            NaN
## 4: 720120 27.01922 2.504692       NaN 3.3351568    3.3351568            NaN
## 5: 720137 21.88823 1.979335       NaN 1.7958292    1.7958292            NaN
## 6: 720151 27.57686 2.998428       NaN 3.8928051    3.8928051            NaN
##     temp_50 CTRY STATE wind.sp_50 atm.press_50
## 1: 22.66268   US    CA   2.565445     1012.557
## 2: 29.75188   US    TX   3.413737     1012.460
## 3: 20.51970   US    MI   2.273423     1014.927
## 4: 25.80545   US    SC   1.696119     1015.281
## 5: 22.43194   US    IL   2.237622     1014.760
## 6: 29.75188   US    TX   3.413737     1012.460
```

Now, the euclidean distance is calculated by $\sqrt{\sum_i(x_i - y_i)^2}$.


```r
station_averages[, eudist_temp_wind.sp := sqrt(
  (temp - temp_50)^2 + (wind.sp - wind.sp_50)^2
  )]; station_averages
```

```
##       USAFID     temp  wind.sp atm.press temp_dist wind.sp_dist atm.press_dist
##    1: 690150 33.18763 3.483560  1010.379 9.5035752    9.5035752      4.3124708
##    2: 720110 31.22003 2.138348       NaN 7.5359677    7.5359677            NaN
##    3: 720113 23.29317 2.470298       NaN 0.3908894    0.3908894            NaN
##    4: 720120 27.01922 2.504692       NaN 3.3351568    3.3351568            NaN
##    5: 720137 21.88823 1.979335       NaN 1.7958292    1.7958292            NaN
##   ---                                                                         
## 1591: 726777 19.15492 4.673878  1014.299 4.5291393    4.5291393      0.3920955
## 1592: 726797 18.78980 2.858586  1014.902 4.8942607    4.8942607      0.2106085
## 1593: 726798 19.47014 4.445783  1014.072 4.2139153    4.2139153      0.6195467
## 1594: 726810 25.03549 3.039794  1011.730 1.3514356    1.3514356      2.9607085
## 1595: 726813 23.47809 2.435372  1012.315 0.2059716    0.2059716      2.3759601
##        temp_50 CTRY STATE wind.sp_50 atm.press_50 eudist_temp_wind.sp
##    1: 22.66268   US    CA   2.565445     1012.557          10.5649277
##    2: 29.75188   US    TX   3.413737     1012.460           1.9447578
##    3: 20.51970   US    MI   2.273423     1014.927           2.7804480
##    4: 25.80545   US    SC   1.696119     1015.281           1.4584280
##    5: 22.43194   US    IL   2.237622     1014.760           0.6019431
##   ---                                                                
## 1591: 19.15492   US    MT   4.151737     1014.185           0.5221409
## 1592: 19.15492   US    MT   4.151737     1014.185           1.3437090
## 1593: 19.15492   US    MT   4.151737     1014.185           0.4310791
## 1594: 20.56798   US    ID   2.568944     1012.855           4.4922623
## 1595: 20.56798   US    ID   2.568944     1012.855           2.9131751
```


```r
station_averages[, eudist_temp_atm.press := sqrt(
  (temp - temp_50)^2 + (atm.press - atm.press_50)^2
  )]; station_averages
```

```
##       USAFID     temp  wind.sp atm.press temp_dist wind.sp_dist atm.press_dist
##    1: 690150 33.18763 3.483560  1010.379 9.5035752    9.5035752      4.3124708
##    2: 720110 31.22003 2.138348       NaN 7.5359677    7.5359677            NaN
##    3: 720113 23.29317 2.470298       NaN 0.3908894    0.3908894            NaN
##    4: 720120 27.01922 2.504692       NaN 3.3351568    3.3351568            NaN
##    5: 720137 21.88823 1.979335       NaN 1.7958292    1.7958292            NaN
##   ---                                                                         
## 1591: 726777 19.15492 4.673878  1014.299 4.5291393    4.5291393      0.3920955
## 1592: 726797 18.78980 2.858586  1014.902 4.8942607    4.8942607      0.2106085
## 1593: 726798 19.47014 4.445783  1014.072 4.2139153    4.2139153      0.6195467
## 1594: 726810 25.03549 3.039794  1011.730 1.3514356    1.3514356      2.9607085
## 1595: 726813 23.47809 2.435372  1012.315 0.2059716    0.2059716      2.3759601
##        temp_50 CTRY STATE wind.sp_50 atm.press_50 eudist_temp_wind.sp
##    1: 22.66268   US    CA   2.565445     1012.557          10.5649277
##    2: 29.75188   US    TX   3.413737     1012.460           1.9447578
##    3: 20.51970   US    MI   2.273423     1014.927           2.7804480
##    4: 25.80545   US    SC   1.696119     1015.281           1.4584280
##    5: 22.43194   US    IL   2.237622     1014.760           0.6019431
##   ---                                                                
## 1591: 19.15492   US    MT   4.151737     1014.185           0.5221409
## 1592: 19.15492   US    MT   4.151737     1014.185           1.3437090
## 1593: 19.15492   US    MT   4.151737     1014.185           0.4310791
## 1594: 20.56798   US    ID   2.568944     1012.855           4.4922623
## 1595: 20.56798   US    ID   2.568944     1012.855           2.9131751
##       eudist_temp_atm.press
##    1:            10.7480686
##    2:                   NaN
##    3:                   NaN
##    4:                   NaN
##    5:                   NaN
##   ---                      
## 1591:             0.1137256
## 1592:             0.8041051
## 1593:             0.3351114
## 1594:             4.6068486
## 1595:             2.9597295
```


```r
station_averages[, eudist_wind.sp_atm.press := sqrt(
  (wind.sp - wind.sp_50)^2 + (atm.press - atm.press_50)^2
  )]; station_averages
```

```
##       USAFID     temp  wind.sp atm.press temp_dist wind.sp_dist atm.press_dist
##    1: 690150 33.18763 3.483560  1010.379 9.5035752    9.5035752      4.3124708
##    2: 720110 31.22003 2.138348       NaN 7.5359677    7.5359677            NaN
##    3: 720113 23.29317 2.470298       NaN 0.3908894    0.3908894            NaN
##    4: 720120 27.01922 2.504692       NaN 3.3351568    3.3351568            NaN
##    5: 720137 21.88823 1.979335       NaN 1.7958292    1.7958292            NaN
##   ---                                                                         
## 1591: 726777 19.15492 4.673878  1014.299 4.5291393    4.5291393      0.3920955
## 1592: 726797 18.78980 2.858586  1014.902 4.8942607    4.8942607      0.2106085
## 1593: 726798 19.47014 4.445783  1014.072 4.2139153    4.2139153      0.6195467
## 1594: 726810 25.03549 3.039794  1011.730 1.3514356    1.3514356      2.9607085
## 1595: 726813 23.47809 2.435372  1012.315 0.2059716    0.2059716      2.3759601
##        temp_50 CTRY STATE wind.sp_50 atm.press_50 eudist_temp_wind.sp
##    1: 22.66268   US    CA   2.565445     1012.557          10.5649277
##    2: 29.75188   US    TX   3.413737     1012.460           1.9447578
##    3: 20.51970   US    MI   2.273423     1014.927           2.7804480
##    4: 25.80545   US    SC   1.696119     1015.281           1.4584280
##    5: 22.43194   US    IL   2.237622     1014.760           0.6019431
##   ---                                                                
## 1591: 19.15492   US    MT   4.151737     1014.185           0.5221409
## 1592: 19.15492   US    MT   4.151737     1014.185           1.3437090
## 1593: 19.15492   US    MT   4.151737     1014.185           0.4310791
## 1594: 20.56798   US    ID   2.568944     1012.855           4.4922623
## 1595: 20.56798   US    ID   2.568944     1012.855           2.9131751
##       eudist_temp_atm.press eudist_wind.sp_atm.press
##    1:            10.7480686                2.3641377
##    2:                   NaN                      NaN
##    3:                   NaN                      NaN
##    4:                   NaN                      NaN
##    5:                   NaN                      NaN
##   ---                                               
## 1591:             0.1137256                0.5343825
## 1592:             0.8041051                1.4783475
## 1593:             0.3351114                0.3152722
## 1594:             4.6068486                1.2190288
## 1595:             2.9597295                0.5559611
```

Then, we need to recover the lat variable, by MERGING.


```r
met_lat <- distinct(met,USAFID,lat)

station_averages_rep <- merge(
  x = station_averages,
  y = met_lat,
  all.x = TRUE, all.y = FALSE,
  by.x = "USAFID", by.y = "USAFID"
)
```

We decide the most representative station per state by calculating the sum of the euclidean distance between each two of temperature, wind speed, and atmospheric pressure.


```r
station_averages_rep <- station_averages_rep %>% mutate(eudist_sum = rowSums(cbind(eudist_temp_wind.sp, eudist_temp_atm.press, eudist_wind.sp_atm.press), na.rm = TRUE))

station_averages_rep <- station_averages_rep[order(eudist_sum,lat)]; station_averages_rep
```

```
##       USAFID     temp  wind.sp atm.press  temp_dist wind.sp_dist atm.press_dist
##    1: 720742      NaN      NaN       NaN        NaN          NaN            NaN
##    2: 720391 31.39394      NaN       NaN  7.7098802    7.7098802            NaN
##    3: 720712      NaN 1.426370       NaN        NaN          NaN            NaN
##    4: 720505 26.58794      NaN       NaN  2.9038827    2.9038827            NaN
##    5: 723825      NaN 3.482819       NaN        NaN          NaN            NaN
##   ---                                                                          
## 2636: 722868 35.62635 2.755172  1008.681 11.9422912   11.9422912       6.010369
## 2637: 722868 35.62635 2.755172  1008.681 11.9422912   11.9422912       6.010369
## 2638: 723805 37.62539 3.532935  1005.207 13.9413314   13.9413314       9.484025
## 2639: 723805 37.62539 3.532935  1005.207 13.9413314   13.9413314       9.484025
## 2640: 720413 24.48892 1.431936  1049.886  0.8048629    0.8048629      35.194606
##        temp_50 CTRY STATE wind.sp_50 atm.press_50 eudist_temp_wind.sp
##    1: 29.75188   US    TX   3.413737     1012.460                 NaN
##    2: 29.75188   US    TX   3.413737     1012.460                 NaN
##    3: 26.70404   US    GA   1.495596     1015.208                 NaN
##    4: 26.33664   US    AL   1.662132     1014.959                 NaN
##    5: 22.66268   US    CA   2.565445     1012.557                 NaN
##   ---                                                                
## 2636: 22.66268   US    CA   2.565445     1012.557            12.96506
## 2637: 22.66268   US    CA   2.565445     1012.557            12.96506
## 2638: 22.66268   US    CA   2.565445     1012.557            14.99396
## 2639: 22.66268   US    CA   2.565445     1012.557            14.99396
## 2640: 26.33664   US    AL   1.662132     1014.959             1.86200
##       eudist_temp_atm.press eudist_wind.sp_atm.press    lat eudist_sum
##    1:                   NaN                      NaN 28.209    0.00000
##    2:                   NaN                      NaN 28.350    0.00000
##    3:                   NaN                      NaN 31.429    0.00000
##    4:                   NaN                      NaN 33.902    0.00000
##    5:                   NaN                      NaN 34.583    0.00000
##   ---                                                                 
## 2636:              13.53085                 3.881119 33.822   30.37703
## 2637:              13.53085                 3.881119 33.830   30.37703
## 2638:              16.67055                 7.413536 34.766   39.07805
## 2639:              16.67055                 7.413536 34.768   39.07805
## 2640:              34.97541                34.927329 34.267   71.76474
```

```r
station_averages_rep <- station_averages_rep %>% group_by(STATE) %>% filter(row_number(STATE) == 1)
station_averages_rep_table <- station_averages_rep[-c(1:9,11:14), c("USAFID","STATE","eudist_sum")]; station_averages_rep_table
```

```
## # A tibble: 35 × 3
## # Groups:   STATE [35]
##    USAFID STATE eudist_sum
##     <int> <chr>      <dbl>
##  1 724180 DE        0     
##  2 720328 WV        0.0161
##  3 725464 IA        0.0449
##  4 720625 OK        0.0834
##  5 722076 IL        0.0882
##  6 726413 WI        0.0905
##  7 720864 NC        0.0955
##  8 722106 FL        0.108 
##  9 722041 LA        0.119 
## 10 723107 SC        0.181 
## # … with 25 more rows
```

Finally, we get the most representative, the median, station per state.

### Question 3: In the middle?

For each state, identify what is the station that is closest to the mid-point of the state. Combining these with the stations you identified in the previous question, use leaflet() to visualize all ~100 points in the same figure, applying different colors for those identified in this question.

Knit the doc and save it on GitHub.


```r
met[, lat_50 := quantile(lat, probs = .5, na.rm = TRUE), by = STATE]
met[, lon_50 := quantile(lon, probs = .5, na.rm = TRUE), by = STATE]
```


```r
met_lat_lon <- distinct(met,USAFID,STATE,lat,lon,lat_50,lon_50)

station_midpoint <- merge(
  x = station_averages,
  y = met_lat_lon,
  all.x = TRUE, all.y = FALSE,
  by = "USAFID"
)
names(station_midpoint)[names(station_midpoint) == 'STATE.x'] <- 'STATE'
```

Now, the euclidean distance is calculated by $\sqrt{\sum_i(x_i - y_i)^2}$.


```r
station_midpoint[, eudist_lat_lon := sqrt(
  (lat - lat_50)^2 + (lon - lon_50)^2
  )]; station_midpoint
```

```
##       USAFID     temp  wind.sp atm.press temp_dist wind.sp_dist atm.press_dist
##    1: 690150 33.18763 3.483560  1010.379 9.5035752    9.5035752       4.312471
##    2: 690150 33.18763 3.483560  1010.379 9.5035752    9.5035752       4.312471
##    3: 720110 31.22003 2.138348       NaN 7.5359677    7.5359677            NaN
##    4: 720113 23.29317 2.470298       NaN 0.3908894    0.3908894            NaN
##    5: 720120 27.01922 2.504692       NaN 3.3351568    3.3351568            NaN
##   ---                                                                         
## 2880: 726810 25.03549 3.039794  1011.730 1.3514356    1.3514356       2.960708
## 2881: 726810 25.03549 3.039794  1011.730 1.3514356    1.3514356       2.960708
## 2882: 726810 25.03549 3.039794  1011.730 1.3514356    1.3514356       2.960708
## 2883: 726813 23.47809 2.435372  1012.315 0.2059716    0.2059716       2.375960
## 2884: 726813 23.47809 2.435372  1012.315 0.2059716    0.2059716       2.375960
##        temp_50 CTRY STATE wind.sp_50 atm.press_50 eudist_temp_wind.sp
##    1: 22.66268   US    CA   2.565445     1012.557           10.564928
##    2: 22.66268   US    CA   2.565445     1012.557           10.564928
##    3: 29.75188   US    TX   3.413737     1012.460            1.944758
##    4: 20.51970   US    MI   2.273423     1014.927            2.780448
##    5: 25.80545   US    SC   1.696119     1015.281            1.458428
##   ---                                                                
## 2880: 20.56798   US    ID   2.568944     1012.855            4.492262
## 2881: 20.56798   US    ID   2.568944     1012.855            4.492262
## 2882: 20.56798   US    ID   2.568944     1012.855            4.492262
## 2883: 20.56798   US    ID   2.568944     1012.855            2.913175
## 2884: 20.56798   US    ID   2.568944     1012.855            2.913175
##       eudist_temp_atm.press eudist_wind.sp_atm.press    lat      lon STATE.y
##    1:             10.748069                2.3641377 34.300 -116.166      CA
##    2:             10.748069                2.3641377 34.296 -116.162      CA
##    3:                   NaN                      NaN 30.784  -98.662      TX
##    4:                   NaN                      NaN 42.543  -83.178      MI
##    5:                   NaN                      NaN 32.224  -80.697      SC
##   ---                                                                       
## 2880:              4.606849                1.2190288 43.567 -116.240      ID
## 2881:              4.606849                1.2190288 43.567 -116.241      ID
## 2882:              4.606849                1.2190288 43.564 -116.223      ID
## 2883:              2.959729                0.5559611 43.650 -116.633      ID
## 2884:              2.959729                0.5559611 43.642 -116.636      ID
##       lat_50   lon_50 eudist_lat_lon
##    1: 36.780 -120.448     4.94832537
##    2: 36.780 -120.448     4.95379168
##    3: 31.178  -97.691     1.04789169
##    4: 43.067  -84.688     1.59833538
##    5: 34.181  -80.634     1.95801379
##   ---                               
## 2880: 43.650 -116.240     0.08300000
## 2881: 43.650 -116.240     0.08300602
## 2882: 43.650 -116.240     0.08766413
## 2883: 43.650 -116.240     0.39300000
## 2884: 43.650 -116.240     0.39608080
```


```r
station_midpoint <- station_midpoint[order(station_midpoint$eudist_lat_lon)]
station_midpoint <- station_midpoint %>% group_by(STATE) %>% filter(row_number(STATE) == 1); station_midpoint
```

```
## # A tibble: 48 × 21
## # Groups:   STATE [48]
##    USAFID  temp wind.sp atm.press temp_dist wind.sp_dist atm.press_dist temp_50
##     <int> <dbl>   <dbl>     <dbl>     <dbl>        <dbl>          <dbl>   <dbl>
##  1 724088  24.7   3.03      1015.     1.04         1.04           0.168    24.6
##  2 724750  27.4   6.71      1012.     3.76         3.76           2.99     24.4
##  3 725975  17.0   2.08      1014.     6.71         6.71           0.281    18.0
##  4 725074  24.5   4.71       NaN      0.774        0.774        NaN        22.5
##  5 725466  21.8   2.72       NaN      1.89         1.89         NaN        21.3
##  6 722570  29.8   3.54      1011.     6.10         6.10           3.26     29.8
##  7 722201  24.2   0.910      NaN      0.502        0.502        NaN        24.7
##  8 726114  17.5   1.17      1015.     6.21         6.21           0.101    18.6
##  9 720928  21.9   2.08       NaN      1.81         1.81         NaN        22.0
## 10 726050  19.9   1.73      1014.     3.82         3.82           0.204    19.6
## # … with 38 more rows, and 13 more variables: CTRY <chr>, STATE <chr>,
## #   wind.sp_50 <dbl>, atm.press_50 <dbl>, eudist_temp_wind.sp <dbl>,
## #   eudist_temp_atm.press <dbl>, eudist_wind.sp_atm.press <dbl>, lat <dbl>,
## #   lon <dbl>, STATE.y <chr>, lat_50 <dbl>, lon_50 <dbl>, eudist_lat_lon <dbl>
```

```r
station_midpoint_table <- station_midpoint[, c("USAFID","STATE","eudist_lat_lon")]; station_midpoint_table
```

```
## # A tibble: 48 × 3
## # Groups:   STATE [48]
##    USAFID STATE eudist_lat_lon
##     <int> <chr>          <dbl>
##  1 724088 DE            0     
##  2 724750 UT            0     
##  3 725975 OR            0     
##  4 725074 RI            0.0210
##  5 725466 IA            0.0260
##  6 722570 TX            0.0382
##  7 722201 NC            0.0490
##  8 726114 VT            0.0611
##  9 720928 OH            0.0646
## 10 726050 NH            0.0730
## # … with 38 more rows
```

Therefore, we get the station that is closest to the mid-point of the state.


```r
# get identified stations ID in the three questions
stations_q1 <- unique(c(median_temp_station$USAFID,median_wind.sp_station$USAFID,median_atm.press_station$USAFID))
stations_q2 <- unique(station_averages_rep_table$USAFID)
stations_q3 <- unique(station_midpoint_table$USAFID)

# get latitude and longitude of the identified stations
met_stations_q1 <- distinct(subset(met, USAFID %in% c(stations_q1))[,c("USAFID","lat","lon")])
  met_stations_q1 <- met_stations_q1[order(USAFID,lat)]
  met_stations_q1 <- met_stations_q1 %>% group_by(USAFID) %>% filter(row_number(USAFID) == 1)
  met_stations_q1$q <- "1"
met_stations_q2 <- distinct(subset(met, USAFID %in% c(stations_q2))[,c("USAFID","lat","lon")])
  met_stations_q2 <- met_stations_q2[order(USAFID,lat)]
  met_stations_q2 <- met_stations_q2 %>% group_by(USAFID) %>% filter(row_number(USAFID) == 1)
  met_stations_q2$q <- "2"
met_stations_q3 <- distinct(subset(met, USAFID %in% c(stations_q3))[,c("USAFID","lat","lon")])
  met_stations_q3 <- met_stations_q3[order(USAFID,lat)]
  met_stations_q3 <- met_stations_q3 %>% group_by(USAFID) %>% filter(row_number(USAFID) == 1)
  met_stations_q3$q <- "3"
met_stations <- rbind(met_stations_q1,met_stations_q2,met_stations_q3)
```


```r
if (knitr::is_html_output()) {
  leaflet(met_stations) %>%
    addProviderTiles('CartoDB.Positron') %>% 
    addCircles(
      data = subset(met_stations, q == 1),
      lat = ~ subset(met_stations, q == 1)$lat, lng = ~ subset(met_stations, q == 1)$lon,
      popup = "median stations", opacity = 1, fillOpacity = 1, radius = 400, color = "red"
      ) %>%
    addCircles(
      data = subset(met_stations, q == 2),
      lat = ~ subset(met_stations, q == 2)$lat, lng = ~ subset(met_stations, q == 2)$lon,
      popup = "average stations", opacity = 1, fillOpacity = 1, radius = 400, color = "blue"
      ) %>%
    addCircles(
      data = subset(met_stations, q == 3),
      lat = ~ subset(met_stations, q == 3)$lat, lng = ~ subset(met_stations, q == 3)$lon,
      popup = "mid point stations", opacity= 1 , fillOpacity = 1, radius = 400, color = "green"
      )
} else {
  message("Sorry! No HTML.")
}
```

```{=html}
<div id="htmlwidget-0e7e8b7b08898cfac751" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-0e7e8b7b08898cfac751">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addProviderTiles","args":["CartoDB.Positron",null,null,{"errorTileUrl":"","noWrap":false,"detectRetina":false}]},{"method":"addCircles","args":[[37.751,31.346],[-82.637,-85.654],400,null,null,{"interactive":true,"className":"","stroke":true,"color":"red","weight":5,"opacity":1,"fill":true,"fillColor":"red","fillOpacity":1},"median stations",null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null,null]},{"method":"addCircles","args":[[45.417,39,37.578,44.567,41.384,36.75,40.225,41.066,35.937,46.941,35.178,29.445,40.199,26.585,43.433,32.633,32.166,33.587,40.033,39.674,39.05,33.761,41.909,41.532,42.637,40.767,41.674,40.789,41.117,40.599,43.2,44.45,43.417,45.443,45.698],[-123.817,-80.274,-84.77,-72.017,-72.506,-97.35,-83.352,-86.182,-77.547,-98.018,-86.066,-90.261,-87.596,-81.861,-83.867,-108.166,-110.883,-80.209,-74.35,-75.606,-96.767,-90.758,-70.729,-71.282,-77.053,-80.4,-93.022,-99.771,-111.966,-116.874,-71.5,-68.367,-88.133,-98.413,-110.44],400,null,null,{"interactive":true,"className":"","stroke":true,"color":"blue","weight":5,"opacity":1,"fill":true,"fillColor":"blue","fillOpacity":1},"average stations",null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null,null]},{"method":"addCircles","args":[[39,47.1,37.578,30.558,37.4,41.384,34.283,48.39,40.28,40.711,35.38,28.228,39.467,35.582,32.564,33.177,31.133,33.45,35.258,35.415,34.257,39.173,39.13,40.033,40.477,38.704,38.058,38.417,39.6,37.285,33.761,41.876,41.597,40.217,42.2,43.322,41.691,40.893,42.6,43.2,44.534,44.316,44.778,43.766,45.097,43.064,45.698,43.564],[-80.274,-122.283,-84.77,-92.099,-77.517,-72.506,-80.567,-100.024,-83.115,-86.375,-86.246,-82.156,-106.15,-79.101,-82.985,-86.783,-97.717,-105.516,-93.095,-97.387,-111.339,-76.684,-75.466,-74.35,-88.916,-93.183,-97.275,-113.017,-116.01,-120.512,-90.758,-71.021,-71.412,-76.851,-75.983,-84.688,-93.566,-97.997,-123.364,-71.5,-72.614,-69.797,-89.667,-99.321,-94.507,-108.458,-110.44,-116.223],400,null,null,{"interactive":true,"className":"","stroke":true,"color":"green","weight":5,"opacity":1,"fill":true,"fillColor":"green","fillOpacity":1},"mid point stations",null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null,null]}],"limits":{"lat":[26.585,48.39],"lng":[-123.817,-68.367]}},"evals":[],"jsHooks":[]}</script>
```

### Question 4: Means of means

Using the quantile() function, generate a summary table that shows the number of states included, average temperature, wind-speed, and atmospheric pressure by the variable “average temperature level,” which you’ll need to create.

Start by computing the states’ average temperature. Use that measurement to classify them according to the following criteria:

low: temp < 20
Mid: temp >= 20 and temp < 25
High: temp >= 25
Once you are done with that, you can compute the following:

Number of entries (records),
Number of NA entries,
Number of stations,
Number of states included, and
Mean temperature, wind-speed, and atmospheric pressure.
All by the levels described before.


```r
met[, state_temp := mean(temp, na.rm = TRUE), by = STATE]
met[, temp_cat := fifelse(
  state_temp < 20, "low-temp",
  fifelse(state_temp < 25, "mid-temp","high-temp")
)]
```

Let's make sure that we don't have NAs.


```r
table(met$temp_cat, useNA = "always")
```

```
## 
## high-temp  low-temp  mid-temp      <NA> 
##    811126    430794   1135423         0
```

Now, let's summarize.


```r
tab <- met[, .(
  N_entries = .N,
  N_entries_NA = sum(is.na(temp_cat)),
  N_stations = length(unique(USAFID)),
  N_states = length(unique(STATE)),
  Mean_temp = mean(temp, na.rm = TRUE),
  Mean_wind.sp = mean(wind.sp, na.rm = TRUE),
  Mean_atm.press = mean(atm.press, na.rm = TRUE)
), by = temp_cat]

knitr::kable(tab)
```



|temp_cat  | N_entries| N_entries_NA| N_stations| N_states| Mean_temp| Mean_wind.sp| Mean_atm.press|
|:---------|---------:|------------:|----------:|--------:|---------:|------------:|--------------:|
|mid-temp  |   1135423|            0|        781|       25|  22.39909|     2.352712|       1014.383|
|high-temp |    811126|            0|        555|       12|  27.75066|     2.514644|       1013.738|
|low-temp  |    430794|            0|        259|       11|  18.96446|     2.637410|       1014.366|

