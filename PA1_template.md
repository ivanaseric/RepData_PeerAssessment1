---
title: "Reproducible Research: Peer Assessment 1"
author: "Ivana Seric"
date: "August 14, 2015"
output: 
html_document:
keep_md: true
fig_caption: yes
---


This is the first project for "Repsoducibe Research" course, fifth course in the Coursera "Data Science" specialization. 


```r
setwd('~/R/Reproducible_research/RepData_PeerAssessment1/')
library(plyr)
library(lubridate)
library(lattice)
options(scipen = 1, digits = 2)
```

## Loading and preprocessing the data

Load the data.


```r
data_download <- mdy("August 9, 2015")
myData <- read.csv("activity.csv")
```
The data was downloaded from the course website on 2015-08-09.  

Process/transform the data (if necessary) into a format suitable for your 
analysis.  


```r
# change date factor into "POSIXct", for easier date maniplation
myData$date <- ymd(as.character(myData$date))
# data set without NA's
myDataNAOmit <- na.omit(myData)  
```


## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the 
dataset.  

Calculate the total number of steps taken per day.  


```r
totalsByDay <- ddply(myDataNAOmit, "date", numcolwise(sum))$steps
```

Make a histogram of the total number of steps taken each day.  


```r
hist(totalsByDay, col = "blue", ann = FALSE, breaks = 10)
title(xlab = "Number of steps", ylab = "Frequency", 
      main = "Total number of steps per day")
box()
```

![plot of chunk histogram](figure/histogram-1.png) 

Calculate and report the mean and median of the total number of steps taken 
per day.  


```r
stepsMean <- mean(totalsByDay, na.rm = TRUE)
stepsMedian <- median(totalsByDay, na.rm = TRUE)
```

The mean of the total number of steps taken per day is 10766.19, 
and the median is 10765.  

## What is the average daily activity pattern?

Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) 
and the average number of steps taken, averaged across all days (y-axis).  


```r
sumByInterval <- ddply(myDataNAOmit, "interval", na.rm = TRUE, numcolwise(sum))
# check that each interval has same number of values (days):
summary(table(myDataNAOmit$interval)) 
```

```
## Number of cases in table: 15264 
## Number of factors: 1
```

```r
numOfdays <- length(totalsByDay)
avStepsInterval <- sumByInterval$steps/numOfdays
plot(sumByInterval$interval, avStepsInterval,
     type = "l", col = "blue", lwd = 2, ann = FALSE)
title(xlab = "Interval", ylab = "Av. Steps",
      main = "Average steps for intervals")
box()
```

![plot of chunk average by interval](figure/average by interval-1.png) 

Which 5-minute interval, on average across all the days in the dataset, 
contains the maximum number of steps?  


```r
maxIndex <- which(max(avStepsInterval, na.rm = TRUE) == avStepsInterval,
                  arr.ind = TRUE)
maxInterval <- sumByInterval$interval[maxIndex]
timeHour <- maxInterval %/% 100
timeMinute <- maxInterval %% 100
```

Time interval  8:35 - 8:40 
contains the maximum number of steps.  


## Imputing missing values

Calculate and report the total number of missing values in the dataset (i.e.
the total number of rows with NAs).  


```r
numNAs <- sum(is.na(myData$steps))
```

The total number of missing values is 2304.  

Devise a strategy for filling in all of the missing values in the dataset.
The strategy does not need to be sophisticated. For example, you could use the 
mean/median for that day, or the mean for that 5-minute interval, etc.  

My approach: Replace NAs with the mean for that interval.  

Create a new dataset that is equal to the original dataset but with the 
missing data filled in.  


```r
# Replace NAs with the mean for that interval
dataLength <- dim(myData)[1]
noNAs <- rep(0, dataLength)

for (i in 1:dataLength) {
    if (is.na(myData$steps[i])) {
        intervalIndex <- which(sumByInterval$interval == myData$interval[i], 
                               arr.ind = TRUE)
        noNAs[i] <- avStepsInterval[intervalIndex]
    } else {
        noNAs[i] <- myData$steps[i]
    }
}

myDataNew <- data.frame(noNAs, myData$date, myData$interval)
colnames(myDataNew) <- c("steps", "date", "interval")
```

Make a histogram of the total number of steps taken each day and calculate 
and report the mean and median total number of steps taken per day. Do these 
values differ from the estimates from the first part of the assignment? What 
is the impact of imputing missing data on the estimates of the total daily 
number of steps?  


```r
totalsByDayNew <- ddply(myDataNew, "date", numcolwise(sum))$steps
hist(totalsByDayNew, col = "blue", ann = FALSE, breaks = 10)
title(xlab = "Number of steps", ylab = "Frequency", 
      main = "Total number of steps per day without NAs")
box()
```

![plot of chunk new historgam](figure/new historgam-1.png) 


```r
stepsMeanNew <- mean(totalsByDayNew)
stepsMedianNew <- median(totalsByDayNew)
```

The new mean of the total number of steps taken per day is 10766.19, 
and the new median is 10766.19.  


```r
compareMeans <- data.frame(c(stepsMedian, stepsMedianNew), c(stepsMean, stepsMeanNew) )
colnames(compareMeans) <- c("Median", "Mean")
rownames(compareMeans) <- c("Old", "New")
compareMeans
```

```
##     Median  Mean
## Old  10765 10766
## New  10766 10766
```

The new mean and median are larger than the old one. This is expected since we 
are adding more values to the total number of steps for each day. 

## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset
with the filled-in missing values for this part.  

Create a new factor variable in the dataset with two levels – “weekday” and 
“weekend” indicating whether a given date is a weekday or weekend day.  


```r
dataWeekday <- weekdays(myData$date, abbreviate = TRUE)
weekdayFactor <- as.factor(ifelse(dataWeekday == c("Sun", "Sat"),
                                  "weekend", "weekday" ))
myDataNew <- cbind(myDataNew, weekdayFactor)
```

Make a panel plot containing a time series plot (i.e. type = "l") of the 
5-minute interval (x-axis) and the average number of steps taken, averaged 
across all weekday days or weekend days (y-axis). See the README file in the 
GitHub repository to see an example of what this plot should look like using 
simulated data.  


```r
sumByIntervalNew <- ddply(myDataNew,.(interval, weekdayFactor), numcolwise(sum))
wdays <- data.frame(table(myDataNew$weekdayFactor))
totalWeekends <- wdays$Freq[which(wdays$Var1 == "weekend")]
totalWeekdays <- wdays$Freq[which(wdays$Var1 == "weekday")]
totalDaysPerFactor <- ifelse(sumByIntervalNew$weekdayFactor == "weekend",
                             totalWeekends, totalWeekdays )
avStepsIntervalNew <- sumByIntervalNew$steps/totalDaysPerFactor

xyplot(avStepsIntervalNew ~ sumByIntervalNew$interval | 
           sumByIntervalNew$weekdayFactor, type = "l", col = "blue", lwd = 2, 
           main = "Average steps for intervals", xlab = "Interval", ylab = 
           "Average steps")
```

![plot of chunk panel plot](figure/panel plot-1.png) 

The plot above shows comparison of average steps per interval for weekdays and
weekends. We can see increased activity during weekends.




