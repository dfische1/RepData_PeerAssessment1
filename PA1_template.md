Reproducible Research Peer Assignment 1
========================================================


Before we can get started, we must load the data:


```r
activity <- read.csv('~/Documents/activity.csv')
```

And remove "NA" values:


```r
activity_nonnull <- na.omit(activity)
```

What is mean total number of steps taken per day?
------------------------------------------------------

### 1. Calculate the total number of steps taken per day

We will start by using the aggregate function to to sum the number of steps taken each day and assign appropriate column names:


```r
aggdata_day <- aggregate(activity_nonnull$steps, by = list(activity_nonnull$date),FUN =sum, na.rm = TRUE)
colnames(aggdata_day) <- c('date','total_steps')
```

### 2. Make a histogram of the total number of steps taken each day

Once aggregation happens, we can create a histogram to display the total number of steps taken each day:


```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.0.3
```

```r
qplot(date,total_steps,data=aggdata_day, geom="histogram",stat="identity") + theme(axis.text.x = element_text(angle = -90, hjust = 0, vjust = 0))
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

### 3. Calculate and report the mean and median of the total number of steps taken per day


```r
mean(aggdata_day$total_steps)
```

```
## [1] 10766.19
```

```r
median(aggdata_day$total_steps)
```

```
## [1] 10765
```

What is the average daily activity pattern?
-------------------------------------------------
### 1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

First, the data must be aggregated, appropriately:


```r
aggdata_int <- aggregate(activity_nonnull$steps, by = list(activity_nonnull$interval),FUN = mean, na.rm = TRUE)
colnames(aggdata_int) <- c('interval','avg_steps')
```

Then, we can create a time series plot:


```r
library(ggplot2)
qplot(interval,avg_steps,data=aggdata_int, geom = "line",stat='identity')
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7-1.png) 

### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

By looking at the output of the following code:


```r
subset(aggdata_int,aggdata_int$avg_steps == max(aggdata_int$avg_steps))
```

```
##     interval avg_steps
## 104      835  206.1698
```

We can see that interval 835, on average, had the maximum number of steps.

Imputing missing values
-------------------------

### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

By looking at the difference in the number of rows of datasets with and without "NA" values, we can see the total number of rows with NAs:


```r
nrow(activity) - nrow(activity_nonnull)
```

```
## [1] 2304
```

Thus, we can see there were 2304 rows with NAs.

### 2. Devise a strategy for filling in all of the missing values in the dataset. 

To keep things simple, my strategy is to fill-in missing values to be the overall mean.

### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
activity_2 <- activity
activity_2$steps[is.na(activity_2$steps)] = mean(activity_2$steps, na.rm=TRUE)
```

### 4. Make a histogram of the total number of steps taken each day

First, data must be aggregated appropriately:


```r
aggdata_day_2 <- aggregate(activity_2$steps, by = list(activity_2$date),FUN =sum, na.rm = TRUE)
colnames(aggdata_day_2) <- c('date','total_steps')
```

Then, we can create a histogram:


```r
library(ggplot2)
qplot(date,total_steps,data=aggdata_day_2, geom="histogram",stat="identity") + theme(axis.text.x = element_text(angle = -90, hjust = 0, vjust = 0))
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 

We can also calculate the mean and median to see how they changed:


```r
mean(aggdata_day_2$total_steps)
```

```
## [1] 10766.19
```

```r
median(aggdata_day_2$total_steps)
```

```
## [1] 10766.19
```

Based on the calculation, we can conclude that this method of imputing did not alter the mean and median.

Are there differences in activity patterns between weekdays and weekends?
---------------------------------------------------------------------------

### 1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

The timeDate library was used in order to distinguish weekdays from weekend days:


```r
install.packages(timeDate)
```

```
## Error in install.packages(timeDate): object 'timeDate' not found
```

```r
library(timeDate)
```

```
## Warning: package 'timeDate' was built under R version 3.0.3
```

```r
activity_2$weekday <-isWeekday(activity_2$date, wday=1:5)
```

### 2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

The data was first split into two subsets, one for weekdays only, one for weekend days only. We then conducted aggregations for each of the subsets, separately, re-assigned a weekday flag value, then appended the two subsets together to form a complete dataset:


```r
weekdays <- subset(activity_2, activity_2$weekday == TRUE)
weekends <- subset(activity_2, activity_2$weekday == FALSE)

aggdata_weekdays <- aggregate(weekdays$steps, by = list(weekdays$interval),FUN = mean, na.rm = TRUE)
colnames(aggdata_weekdays) <- c('interval','avg_steps')

aggdata_weekends <- aggregate(weekends$steps, by = list(weekends$interval),FUN = mean, na.rm = TRUE)
colnames(aggdata_weekends) <- c('interval','avg_steps')

aggdata_weekdays$weekday <- 'Weekday'
aggdata_weekends$weekday <- 'Weekend'

alldays <- rbind(aggdata_weekdays,aggdata_weekends)
```

Once this pre-processing was complete, we could create a panel graph:

```r
library(ggplot2)
qplot(interval,avg_steps,data=alldays, geom = "line",stat='identity')+facet_wrap(~weekday,nrow = 2)
```

![plot of chunk unnamed-chunk-16](figure/unnamed-chunk-16-1.png) 
