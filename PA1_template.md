---
title: "project1"
output:
    html_document:
        keep_md: true
    
---



## Code for reading in the dataset and/or processing the data

The following code is used to preprocess and prepare the data.


```r
library(ggplot2)
download.file(url = 'https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip', 
              destfile = paste(getwd(), 'activity.zip', sep = '/'))
unzip('activity.zip')
activity = data.table::fread('activity.csv')
```

## Histogram of the total number of steps taken each day

Following is the code to generate the histogram with the plot itself as well.


```r
total_steps = activity[, lapply(.SD, sum, na.rm=F), .SDcols = c('steps'), by = date] 
ggplot(total_steps) +
    geom_histogram(aes(x=steps), binwidth = 1000) +
    labs(title='Num of steps taken each day') + 
    labs(x='Steps', y= 'Frequency')
```

```
## Warning: Removed 8 rows containing non-finite values (stat_bin).
```

![](project1_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

## Mean and median number of steps taken each day
Following is the code to calculate the mean and median of the steps taken each day.


```r
total_steps[, .(Mean_Steps = mean(steps, na.rm = TRUE), Median_Steps = median(steps, na.rm = TRUE))]
```

```
##    Mean_Steps Median_Steps
## 1:   10766.19        10765
```

## Time series plot of the average number of steps taken
Following is the code for the time series plot of steps taken each day.


```r
time_series = activity[, lapply(.SD, mean, na.rm=T), .SDcols = c('steps'), by = 'interval']

ggplot(data = time_series) + 
    geom_line(aes(x = interval, y = steps)) + 
    xlab("")
```

![](project1_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

## The 5-minute interval that, on average, contains the maximum number of steps
Following is the code for determining which interval has the max number of steps.

```r
time_series[which.max(time_series$steps)]
```

```
##    interval    steps
## 1:      835 206.1698
```

## Code to describe and show a strategy for imputing missing data
Here I will replace the missing values with the mean.


```r
activity[is.na(steps), 'steps'] = activity[, lapply(.SD, mean, na.rm=T), .SDcols=c('steps')]
```

```
## Warning in `[<-.data.table`(`*tmp*`, is.na(steps), "steps", value =
## structure(list(: 37.382600 (type 'double') at RHS position 1 truncated
## (precision lost) when assigning to type 'integer' (column 1 named 'steps')
```

## Histogram of the total number of steps taken each day after missing values are imputed
Here is the histogram of the steps after replacing missing values.

```r
total_steps = activity[, lapply(.SD, sum, na.rm=F), .SDcols = c('steps'), by = date] 
ggplot(total_steps) +
    geom_histogram(aes(x=steps), binwidth = 1000)+
    labs(title='Num of steps taken each day') + 
    labs(x='Steps', y= 'Frequency')
```

![](project1_files/figure-html/unnamed-chunk-7-1.png)<!-- -->


## Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends
Now lets compare the activity across weekdays and weekends.


```r
#Reading a clean version of the data
activity = data.table::fread('activity.csv')
activity[, isWeekend := chron::is.weekend(activity$date)]
activity[is.na(steps), 'steps'] = activity[, lapply(.SD, mean, na.rm=T), .SDcols = c('steps')]
```

```
## Warning in `[<-.data.table`(`*tmp*`, is.na(steps), "steps", value =
## structure(list(: 37.382600 (type 'double') at RHS position 1 truncated
## (precision lost) when assigning to type 'integer' (column 1 named 'steps')
```

```r
interval = activity[,lapply(.SD, mean, na.rm=T), .SDcols=c('steps'), by=.(interval, isWeekend)]
```
Now to create the plot

```r
ggplot(data=interval) +
    geom_line(aes(x=interval,y=steps,color=isWeekend)) +
    labs(x='Interval', y='Steps') +
    labs(main='Comparing activity over weekends and weekdays') +
    facet_wrap(~'isWeekend', ncol=1, nrow=2)
```

![](project1_files/figure-html/unnamed-chunk-9-1.png)<!-- -->



