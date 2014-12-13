---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

## Loading and preprocessing the data
1. Load the data. First of all, the data. I don't know why read.table works better than read.csv with a csv file, do you?

   
   ```r
   data<-read.table("activity.csv", sep = ",", header = TRUE)
   ```

2. Process/transform the data. I tapply by date / by interval
   
   ```r
    totalStepsPerDay<-tapply(data$steps, data$date, sum) 
    avgStepsPerInterval<-tapply(data$steps, data$interval, mean, na.rm=TRUE) 
   ```
   
## What is mean total number of steps taken per day?

1. Make a histogram of the total number of steps taken each day
   
   ```r
    histTotalStepsPerDay<-hist(totalStepsPerDay, xlab="Histogram 1")
   ```
   
   ![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 


2. Calculate and report the mean and median total number of steps taken per day
   
   ```r
   meanOfStepsPerDay<-mean(totalStepsPerDay, na.rm=TRUE)
   medianOfStepsPerDay<-median(totalStepsPerDay, na.rm=TRUE)
   meanOfStepsPerDay
   ```
   
   ```
   ## [1] 10766.19
   ```
   
   ```r
   medianOfStepsPerDay
   ```
   
   ```
   ## [1] 10765
   ```
   The mean is 1.0766189 &times; 10<sup>4</sup> and the median is 10765

## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
   
   ```r
    plot(names(avgStepsPerInterval), avgStepsPerInterval, type="l", xlab = "Intervals - Plot 1", ylab = "Avg Steps")
   ```
   
   ![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
   
   ```r
    maxStepAvgInterval<-avgStepsPerInterval[avgStepsPerInterval==max(avgStepsPerInterval)]
    maxStepAvgInterval
   ```
   
   ```
   ##      835 
   ## 206.1698
   ```
  The max average reaches at 8:35


## Imputing missing values
1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
   
   ```r
     totalNAs<-sum(is.na(data$steps))
     totalNAs
   ```
   
   ```
   ## [1] 2304
   ```
   There are  2304 rows with steps = NA

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

  I apply the interval mean on each NA: I need some calculations...
   
   ```r
   data$avgInt<-avgStepsPerInterval
   data$stepsRmNa<-data$steps
   data$stepsRmNa[is.na(data$steps)]<-0
   data$stepsOK<-is.na(data$steps) * data$avgInt + (!is.na(data$steps)) * data$stepsRmNa
   ```
3. Create a new dataset that is equal to the original dataset but with the missing data filled in.   

   
   ```r
   newData<-data.frame(data$stepsOK, data$date, data$interval)
   names(newData)<-names(data)[1:3]
   head(newData)
   ```
   
   ```
   ##       steps       date interval
   ## 1 1.7169811 2012-10-01        0
   ## 2 0.3396226 2012-10-01        5
   ## 3 0.1320755 2012-10-01       10
   ## 4 0.1509434 2012-10-01       15
   ## 5 0.0754717 2012-10-01       20
   ## 6 2.0943396 2012-10-01       25
   ```
   
   ```r
   str(newData)
   ```
   
   ```
   ## 'data.frame':	17568 obs. of  3 variables:
   ##  $ steps   : num  1.717 0.3396 0.1321 0.1509 0.0755 ...
   ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
   ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
   ```
4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

   
   ```r
    totalStepsPerDayNoNA<-tapply(newData$steps, newData$date, sum) 
    histTotalStepsPerDayNoNA<-hist(totalStepsPerDayNoNA, xlab="Histogram 2")
   ```
   
   ![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 
   
   ```r
   meanOfStepsPerDayNoNA<-mean(totalStepsPerDayNoNA, na.rm=TRUE)
   medianOfStepsPerDayNoNA<-median(totalStepsPerDayNoNA, na.rm=TRUE)
   meanOfStepsPerDayNoNA
   ```
   
   ```
   ## [1] 10766.19
   ```
   
   ```r
   medianOfStepsPerDayNoNA
   ```
   
   ```
   ## [1] 10766.19
   ```
   The mean is 1.0766189 &times; 10<sup>4</sup> and the median is 1.0766189 &times; 10<sup>4</sup>
    
    Mean value is the same as previous, because there are only whole days with NAs. When I fill the NA's, I fill entire days, then the avg per day remains the same value. By the same reason, median has changed, because I add the mean value several times to the avg per day series.
    
## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.
   
   ```r
    week<-data.frame(n=1:7, s=c("weekend", rep("weekday", 5), "weekend")) #Weekend are days 1 and 7
    wd<-merge(x=data.frame(d=wday(newData$date)), y=week, by.x="d", by.y="n")
    newData$wd<-wd$s
    str(newData)
   ```
   
   ```
   ## 'data.frame':	17568 obs. of  4 variables:
   ##  $ steps   : num  1.717 0.3396 0.1321 0.1509 0.0755 ...
   ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
   ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
   ##  $ wd      : Factor w/ 2 levels "weekday","weekend": 2 2 2 2 2 2 2 2 2 2 ...
   ```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).
   
   ```r
    newDataWeekend<-newData[newData$w=="weekend",]
    newDataWeekday<-newData[newData$w=="weekday",]
    avgStepsPerIntervalWeekend<-tapply(newDataWeekend$steps, newDataWeekend$interval, mean, na.rm=TRUE) 
    avgStepsPerIntervalWeekday<-tapply(newDataWeekday$steps, newDataWeekday$interval, mean, na.rm=TRUE) 
    plot(names(avgStepsPerIntervalWeekend), avgStepsPerIntervalWeekend, type="l", xlab = "Intervals - Plot 2", ylab = "Avg Steps")
   ```
   
   ![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 
   
   ```r
    plot(names(avgStepsPerIntervalWeekday), avgStepsPerIntervalWeekday, type="l", xlab = "Intervals - Plot 3", ylab = "Avg Steps")
   ```
   
   ![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-2.png) 

  Weekdays starts at 5am, later at weekends, and average are high on weekends. 
