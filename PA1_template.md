# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

```r
unzip("activity.zip")
activity <- read.csv("activity.csv")
```


## What is mean total number of steps taken per day?

```r
# Make a histogram of the total number of steps taken each day
steps_per_day <- aggregate(steps ~ date, data = activity, FUN = sum)
with(steps_per_day, hist(steps, breaks = nrow(steps_per_day), main = "Total Number of Steps Per Day", 
    xlab = "Steps Per Day"))
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2.png) 

```r

# Calculate and report the mean and median total number of steps taken per
# day
mean_steps <- mean(steps_per_day$steps)
median_steps <- median(steps_per_day$steps)
```

The mean total number of steps taken per day is 10766

The median total number of steps taken per day is 10765

## What is the average daily activity pattern?

```r
# Make a time series plot (i.e. type = 'l') of the 5-minute interval
# (x-axis) and the average number of steps taken, averaged across all days
# (y-axis)
average_per_interval <- aggregate(steps ~ interval, data = activity, FUN = mean)
plot(average_per_interval, type = "l", xlab = "5-minute Interval", ylab = "Average # of Steps Taken", 
    main = "Average Daily Activity Pattern")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3.png) 

```r

# Which 5-minute interval, on average across all the days in the dataset,
# contains the maximum number of steps?
max_steps_interval <- average_per_interval[which.max(average_per_interval$steps), 
    "interval"]
```

The 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps is 835

## Imputing missing values

```r
# Calculate and report the total number of missing values in the dataset
# (i.e. the total number of rows with NAs)
total_missing <- sum(is.na(activity))
```

The total number of missing values in the dataset is 2304


```r
# Filling in all of the missing values in the dataset with the mean of the
# 5-minute interval averaged across all days
imputed <- numeric()
for (i in 1:nrow(activity)) {
    obs <- activity[i, ]
    if (is.na(obs$steps)) {
        steps <- subset(average_per_interval, interval == obs$interval)$steps
    } else {
        steps <- obs$steps
    }
    imputed <- c(imputed, steps)
}

# Create a new dataset that is equal to the original dataset but with the
# missing data filled in
imputed_activity <- activity
imputed_activity$steps <- imputed

# Make a histogram of the total number of steps taken each day and Calculate
# and report the mean and median total number of steps taken per day. Do
# these values differ from the estimates from the first part of the
# assignment? What is the impact of imputing missing data on the estimates
# of the total daily number of steps?
imputed_steps_per_day <- aggregate(steps ~ date, data = imputed_activity, FUN = sum)

hist(imputed_steps_per_day$steps, breaks = nrow(imputed_steps_per_day), xlab = "Steps Per Day", 
    main = "Total Number of Steps Per Day")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5.png) 

```r

imputed_mean_steps <- mean(imputed_steps_per_day$steps)
imputed_median_steps <- median(imputed_steps_per_day$steps)
```

The mean total number of steps taken per day is 10766

The median total number of steps taken per day is 10766

After imputing missing data, mean is unchanged, while median is a little bit increase.

## Are there differences in activity patterns between weekdays and weekends?

```r
# Create a new factor variable in the dataset with two levels – “weekday”
# and “weekend” indicating whether a given date is a weekday or weekend day.
wd <- strptime(imputed_activity$date, "%Y-%m-%d")
wd <- weekdays(wd)
l <- wd == "Sunday" | wd == "Saturday"
wd[l] <- "weekend"
wd[!l] <- "weekday"
wd <- as.factor(wd)
imputed_activity$daytype <- wd

# Make a panel plot containing a time series plot (i.e. type = 'l') of the
# 5-minute interval (x-axis) and the average number of steps taken, averaged
# across all weekday days or weekend days (y-axis).
require(plyr)
```

```
## Loading required package: plyr
```

```r
average_steps <- ddply(imputed_activity, .(interval, daytype), summarize, steps = mean(steps))

require(lattice)
```

```
## Loading required package: lattice
```

```r
xyplot(steps ~ interval | daytype, data = average_steps, layout = c(1, 2), type = "l", 
    xlab = "Interval", ylab = "Number of Steps")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6.png) 

