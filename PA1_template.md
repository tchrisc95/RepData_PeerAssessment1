---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data
Begin by reading in the csv file and view the first couple lines of the data.

```r
activity<-read.csv("activity.csv")
head(activity)
```

```
##   steps       date interval
## 1    NA 2012-10-01        0
## 2    NA 2012-10-01        5
## 3    NA 2012-10-01       10
## 4    NA 2012-10-01       15
## 5    NA 2012-10-01       20
## 6    NA 2012-10-01       25
```
Convert the date column to the Date data type. 

```r
activity$date<-as.Date(activity$date)
```

## What is the mean total number of steps taken per day?
Calculate the number of steps taken per day and display in a histogram:

```r
stepsPerDay<-aggregate(activity["steps"],by=activity["date"],sum)
hist(stepsPerDay$steps, main="Frequency of Number of Steps per Day",xlab="Steps per Day",ylab="Count of Days")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

The mean number of steps taken in a day is given by

```r
mean(stepsPerDay$steps,na.rm=TRUE)
```

```
## [1] 10766.19
```
and the median number of steps taken in a day is given by

```r
median(stepsPerDay$steps,na.rm=TRUE)
```

```
## [1] 10765
```


## What is the average daily activity pattern?
Calculate the average number of steps taken at each time interval across all days:

```r
stepsPerInterval<-aggregate(activity["steps"],by=activity["interval"],mean,na.rm=TRUE)
```

Create a time series plot based on the average steps per each time interval:


```r
plot(stepsPerInterval$interval,stepsPerInterval$steps,type="l",ylab="Number of Steps",xlab="Time Interval",main="Average Steps Per Time Interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

The interval with the maximal number of average steps is 

```r
stepsPerInterval[which.max(stepsPerInterval$steps),"interval"]
```

```
## [1] 835
```


## Imputing missing values
The total number of rows with missing values is:

```r
sum(!complete.cases(activity))
```

```
## [1] 2304
```

Fill in the missing values using the mean number of steps taken for the given interval. Note that the package dplyr must be installed.

```r
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

```r
completeActivity<-data.frame(activity)

completeActivity$steps <- with(completeActivity, ave(steps, interval,
    FUN = function(x) replace(x, is.na(x), mean(x, na.rm = TRUE))))
```

Display a histogram of the number of steps taken per day with the imputed data.


```r
completeStepsPerDay<-aggregate(completeActivity["steps"],by=completeActivity["date"],sum)
hist(completeStepsPerDay$steps, main="Frequency of Number of Steps per Day",xlab="Steps per Day",ylab="Count of Days")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

The new mean number of steps per day is

```r
mean(completeStepsPerDay$steps)
```

```
## [1] 10766.19
```
and the new median number of steps per day is

```r
median(completeStepsPerDay$steps)
```

```
## [1] 10766.19
```
While the mean stays the same, the median shifts slightly with the na's removed.

## Are there differences in activity patterns between weekdays and weekends?

Create a factor variable to split the data by weekday vs weekend

```r
weekdays1<-c("Monday","Tuesday","Wednesday","Thursday","Friday")

completeActivity$wDay<-factor((weekdays(completeActivity$date) %in% weekdays1), 
         levels=c(FALSE, TRUE), labels=c('weekend', 'weekday') )
```

Create a line plot comparing the average steps per time interval split by weeday vs weekend.


```r
weekdaySteps<-completeActivity[completeActivity$wDay=="weekday",]
weekendSteps<-completeActivity[completeActivity$wDay=="weekend",]
  
  
stepsPerIntervalWeekday<-aggregate(weekdaySteps["steps"],by=weekdaySteps["interval"],mean,na.rm=TRUE)
stepsPerIntervalWeekend<-aggregate(weekendSteps["steps"],by=weekendSteps["interval"],mean,na.rm=TRUE)


x<-stepsPerIntervalWeekday$interval
y1<-stepsPerIntervalWeekday$steps
y2<-stepsPerIntervalWeekend$steps


plot(x,y1,type="l",ylab="Number of Steps",xlab="Time Interval",main="Steps per Time Interval")
lines(x,y2,col="Red")
legend("topright", legend=c("Weekday","Weekend"),col=c("black", "red"),lty=c(1,1,1),cex=1)
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

From the plot it appears that the general pattern of steps taken per day and weekend is similar. However, there appear to be fewer steps taken in the morning on a weekend and a bigger spike between time interval 500 and 1000 for weekdays. There are also slightly more steps taken throughout the day on a weekend.
