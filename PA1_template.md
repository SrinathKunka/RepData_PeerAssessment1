**Reproducible Research - Week 2 Assignment**

Setting the working directory


```r
setwd("C:/Srinath/Study/DataScientist/Course - 5/Week2/Assignment")
```

Declaring the required libraries

```r
library(ggplot2)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

**Loading the data**

```r
csvdata <- read.csv("activity.csv")
```

Ignoring the missing data

```r
dataWithoutNA <- na.omit(csvdata)
```

***Processing the data for the required format***

**What is mean total number of steps taken per day?**

1. Calculate the total number of steps taken per day

```r
stepsPerDay <- group_by(dataWithoutNA, date)
stepsPerDay <- summarise(stepsPerDay, steps=sum(steps))
```

2. Chart for the total number of steps taken each day

```r
qplot(steps, data = stepsPerDay, bins = 25,
      xlab="Total Number of Steps in Each Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

3. Mean and median of the total number of steps taken per day

```r
mean(stepsPerDay$steps)
```

```
## [1] 10766.19
```

```r
median(stepsPerDay$steps)
```

```
## [1] 10765
```

**What is the average daily activity pattern?**

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
stepsPerDayInterval <- group_by(dataWithoutNA, interval)
stepsPerDayInterval <- summarise(stepsPerDayInterval, steps=mean(steps))
```

Plotting the average daily agains the interval
ggplot(stepsPerDayInterval, aes(interval, steps)) + geom_line()


2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
stepsPerDayInterval[stepsPerDayInterval$steps == max(stepsPerDayInterval$steps),]
```

```
## # A tibble: 1 × 2
##   interval    steps
##      <int>    <dbl>
## 1      835 206.1698
```

**Imputing missing values**  
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
missingValueRows <- nrow(csvdata)-nrow(dataWithoutNA)
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.


```r
names(stepsPerDayInterval)[2] <- "StepsMean"
missingValueData <- merge(csvdata, stepsPerDayInterval)
```

3, Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
missingValueData$steps[is.na(missingValueData$steps)] <- missingValueData$StepsMean[is.na(missingValueData$steps)]
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
impactedStepsPerDay <- group_by(missingValueData, date)
impactedStepsPerDay <- summarise(impactedStepsPerDay, steps=sum(steps))

qplot(steps, data=impactedStepsPerDay,bins = 25)
```

![](PA1_template_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

```r
mean(impactedStepsPerDay$steps)
```

```
## [1] 10766.19
```

```r
median(impactedStepsPerDay$steps)
```

```
## [1] 10766.19
```

**Are there differences in activity patterns between weekdays and weekends?**

 1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
missingValueData$weekDay <- weekdays(as.Date(missingValueData$date))
missingValueData$weekEnd <- as.factor(missingValueData$weekDay=="Saturday" | missingValueData$weekDay == "Sunday")
levels(missingValueData$weekEnd) <- c("Weekday", "Weekend")
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
weekDayData <- missingValueData[missingValueData$weekEnd=="Weekday",]
weekEndData <- missingValueData[missingValueData$weekEnd=="Weekend",]

weekDayIntervalData <- group_by(weekDayData,interval)
weekDayIntervalData <- summarise(weekDayIntervalData, steps=mean(steps))
weekDayIntervalData$weekEnd <- "Weekday"

weekEndIntervalData <- group_by(weekEndData,interval)
weekEndIntervalData <- summarise(weekEndIntervalData, steps=mean(steps))
weekEndIntervalData$weekEnd <- "Weekend"
```


Two data frames, to differentiate two time series


```r
stepsPerDayInterval <- rbind(weekDayIntervalData, weekEndIntervalData)
stepsPerDayInterval$weekEnd <- as.factor(stepsPerDayInterval$weekEnd)
ggplot(stepsPerDayInterval, aes(interval,steps)) + geom_line() + facet_grid(weekEnd ~ .)
```

![](PA1_template_files/figure-html/unnamed-chunk-16-1.png)<!-- -->
