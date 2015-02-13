# Reproducible Research: Peer Assessment 1


***

## Loading and preprocessing the data

Setting the global options for the document


```r
library(knitr)
library(ggplot2)
library(lubridate)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
## 
## The following objects are masked from 'package:lubridate':
## 
##     intersect, setdiff, union
## 
## The following object is masked from 'package:stats':
## 
##     filter
## 
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
knitr::opts_chunk$set(echo = TRUE)
```

Extracting Data, Converting the resultant data frame to tbl_df


```r
unzip("activity.zip")
dat <- read.csv("activity.csv")
dat <- tbl_df(dat)
```


***

## What is mean total number of steps taken per day?

1. Calculate the total number of steps taken per day


```r
by_day <- group_by(dat, date)
by_day <- summarize(by_day, total.steps = sum(steps, na.rm = TRUE))
```

2. Make a histogram of the total number of steps taken each day


```r
ggplot(by_day, aes(total.steps)) + 
        geom_histogram(binwidth = 1000) +
        labs(title = "Histogram of Total Steps taken per day",
             x = "Total Steps Taken",
             y = "Frequency")
```

![](PA1_template_files/figure-html/histogram-1.png) 

3. Calculate and report the mean and median of the total number of steps taken per day


```r
(median.steps <- median(by_day$total.steps))
```

```
## [1] 10395
```

```r
(mean.steps <- mean(by_day$total.steps))
```

```
## [1] 9354.23
```

Mean number of steps = 9354.2295082

Median number of steps = 10395


***


## What is the average daily activity pattern?


1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
## Group the data by time intervals
by_int <- group_by(dat, interval)
by_int <- summarise(by_int, average.steps = mean(steps, na.rm = TRUE))
```


```r
## Plot Time Series graph, using ggplot
ggplot(by_int, aes(interval, average.steps)) +
        geom_line() +
        labs(title = "Time Series Plot",
             x = "Time Interval of the day",
             y = "Average steps taken (across all days)")
```

![](PA1_template_files/figure-html/unnamed-chunk-1-1.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
(max.steps <- paste(by_int[which.max(by_int$average.steps), 1]))
```

```
## [1] "835"
```

Answer: 835

## Imputing missing values

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
indices <- is.na(dat$steps)
total.nas <- sum(indices)
```

Answer: 2304

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


Answer: We will be filling missing values in the dataset with the mean for the respective 5-minute interval. We already computed a logical vector containing indices in the original dataset for which `steps == NA` in the previous question.

*       The tbl_df object `by_int` will come handy in making substitutions.
*       We can use the logical vector `indices` to get the row numbers corresponding the `steps == NA`
*       Running a `for` loop with the row numbers, we can make substitute the NA values in 

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.



```r
a <- which(indices == TRUE)
newdat <- dat
for (i in a){
        xt <- which(by_int$interval == newdat[i, ]$interval)
        newdat[i, ]$steps <- by_int[xt,]$average.steps
}
newdat
```

```
## Source: local data frame [17,568 x 3]
## 
##        steps       date interval
## 1  1.7169811 2012-10-01        0
## 2  0.3396226 2012-10-01        5
## 3  0.1320755 2012-10-01       10
## 4  0.1509434 2012-10-01       15
## 5  0.0754717 2012-10-01       20
## 6  2.0943396 2012-10-01       25
## 7  0.5283019 2012-10-01       30
## 8  0.8679245 2012-10-01       35
## 9  0.0000000 2012-10-01       40
## 10 1.4716981 2012-10-01       45
## ..       ...        ...      ...
```



## Are there differences in activity patterns between weekdays and weekends?
