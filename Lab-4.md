Lab4
================
Juehan Wang
9/17/2021

## 2. Prepare the data

``` r
#Remove temperatures less than -17C
met <- met[met$temp >= -17]
#Make sure there are no missing data in the key variables coded as 9999, 999, etc
met[,range(temp,na.rm = TRUE)]
```

    ## [1] -17  56

``` r
met[,range(rh,na.rm = TRUE)]
```

    ## [1]   0.8334298 100.0000000

``` r
met[,range(wind.sp,na.rm = TRUE)]
```

    ## [1]  0 36

``` r
met[,range(vis.dist,na.rm = TRUE)]
```

    ## [1]      0 160000

``` r
met[,range(dew.point,na.rm = TRUE)]
```

    ## [1] -37.2  36.0

``` r
met[,range(lat,na.rm = TRUE)]
```

    ## [1] 24.550 48.941

``` r
met[,range(lon,na.rm = TRUE)]
```

    ## [1] -124.290  -68.313

``` r
met[,range(elev,na.rm = TRUE)]
```

    ## [1]  -13 9999

``` r
met[elev == 9999.0, elev :=NA]

#Generate a date variable using the functions as.Date() (hint: You will need the following to create a date paste(year, month, day, sep = "-")).
met[, ymd := as.Date(paste(met$year, met$month, met$day, sep = "-"))]

met[,table(week(ymd))]
```

    ## 
    ##     31     32     33     34     35 
    ## 297260 521605 527924 523847 446576

``` r
#Using the data.table::week function, keep the observations of the first week of the month.
met <- met[week(ymd) ==31]

#Compute the mean by station of the variables temp, rh, wind.sp, vis.dist, dew.point, lat, lon, and elev.
met_avg <- met[,.(
  temp      = mean(temp,na.rm=TRUE),
  rh        = mean(rh,na.rm=TRUE),
  wind.sp   = mean(wind.sp,na.rm=TRUE),
  dew.point = mean(dew.point,na.rm=TRUE),
  vis.dist  = mean(vis.dist,na.rm=TRUE),
  lat       = mean(lat),
  lon       = mean(lon), 
  elev      = mean(elev,na.rm=TRUE)
), by=c("USAFID")]

#Create a region variable for NW, SW, NE, SE based on lon = -98.00 and lat = 39.71 degrees


#Create a categorical variable for elevation as in the lecture slides
```
