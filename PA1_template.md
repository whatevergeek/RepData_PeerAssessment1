---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

## Setting Global Options

```r
opts_chunk$set(echo=TRUE)
```

## Loading and preprocessing the data
* Loading the data

```r
movement_data <- read.csv(unzip("activity.zip"), header=TRUE, colClasses=c("integer", "Date", "integer"))
```
* Additional processing for analysis

```r
movement_data$month <- format(movement_data$date, format="%B")
mdata <- na.omit(movement_data)
options(scipen=999) #Turn off scientific notation
library(ggplot2)
```

## What is mean total number of steps taken per day?
* Make a histogram of the total number of steps taken each day

```r
ggplot(mdata, aes(date, steps)) + 
  geom_bar(stat = "identity", colour = "black", fill = "black", width = 0.8) + 
  facet_grid(. ~ month, scales = "free") + 
  labs(x = "Date", y = "Total Steps", title = "Histogram of the total number of steps taken each day")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

* Calculate and report the mean and median total number of steps taken per day

```r
steps <- aggregate(mdata$steps, list(Date = mdata$date), FUN = "sum")$x
orig_steps_mean <- mean(steps)
orig_steps_median <- median(steps)
```
Mean is 10766.1886792  
Median is 10765

## What is the average daily activity pattern?
* Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
steps_mean <- aggregate(mdata$steps, list(interval = as.numeric(as.character(mdata$interval))), FUN = "mean")
names(steps_mean)[2] <- "stepsmean"

ggplot(steps_mean, aes(interval, stepsmean)) + 
  geom_line(color = "black", size = 0.8) + 
  labs(x = "5-minute interval", 
    y = "Average number of steps taken",
    title = "Time Series Plot of Daily Activity")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 
* Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
max_steps <- max(steps_mean$stepsmean)
max_interval <- steps_mean[steps_mean$stepsmean == max_steps, ]$interval
```
Interval 835 contains the maximum number of steps (206.1698113)

## Imputing missing values
* Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
total_na_rows <- sum(is.na(movement_data))
```
Total number of rows with NAs: 2304

* Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

I'll be using mean for the 5-minute interval to fill in the missing values.

* Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
new_mdata <- movement_data
for (i in 1:nrow(new_mdata)) {
    if (is.na(new_mdata$steps[i])) {
        new_mdata$steps[i] <- steps_mean[which(new_mdata$interval[i] == steps_mean$interval), ]$stepsmean
    }
}

#check for NA
total_na_rows_new <- sum(is.na(new_mdata))
```

Total number of rows with NAs in the new data: 0

* Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
ggplot(new_mdata, aes(date, steps)) + 
  geom_bar(stat = "identity", colour = "black", fill = "black", width = 0.8) + 
  facet_grid(. ~ month, scales = "free") + 
  labs(x = "Date", y = "Total Steps", title = "Histogram of the total number of steps taken each day (NAs=0)")
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png) 

```r
new_steps <- aggregate(new_mdata$steps, list(Date = new_mdata$date), FUN = "sum")$x
new_steps_mean <- mean(new_steps)
new_steps_median <- median(new_steps)

mean_diff <- new_steps_mean - orig_steps_mean
median_diff <- new_steps_median - orig_steps_median
```
Mean is 10766.1886792  
Median is 10766.1886792

Difference between old and new mean: 0  
Difference between old and new median: 1.1886792

The mean stayed the same while there's a change in the median.
Imputing missing data affects the median but not necessarily the mean.


## Are there differences in activity patterns between weekdays and weekends?

* Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
new_mdata$daytype <- factor(format(new_mdata$date, format="%A"))
levels(new_mdata$daytype) <- list(weekday = c("Monday", 
                                              "Tuesday",
                                             "Wednesday", 
                                             "Thursday", 
                                              "Friday"),
                                 weekend = c("Saturday", 
                                             "Sunday"))
```

* Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

```r
steps_mean <- aggregate(new_mdata$steps, 
                      list(interval = as.numeric(as.character(new_mdata$interval)), 
                           daytype = new_mdata$daytype),
                      FUN = "mean")
names(steps_mean)[3] <- "stepsmean"

library(lattice)
xyplot(steps_mean$stepsmean ~ steps_mean$interval | steps_mean$daytype, 
       layout = c(1, 2), type = "l", 
       xlab = "5-Minute Interval", 
       ylab = "Average Steps Taken")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png) 

