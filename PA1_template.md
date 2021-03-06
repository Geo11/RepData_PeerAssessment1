# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data


```r
setwd("C://Users/demyc 13/Documents/Geovanni/Coursera (Data Science)/5. Reproducible Research/Week 1/Project 1")
library(dplyr)
```

```
## Warning: package 'dplyr' was built under R version 3.1.1
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
library(lubridate)
```

```
## Warning: package 'lubridate' was built under R version 3.1.2
```

```r
library(lattice)
```

```
## Warning: package 'lattice' was built under R version 3.1.2
```

## What is mean total number of steps taken per day?


```r
# Select only the complete cases (without NA's)
data<-read.csv("activity.csv", header=TRUE, colClasses=c("numeric", "Date","numeric"))
cc<-data[complete.cases(data),]
# Calculate the total number steps per day
steps_day<-group_by(cc,date)%>% summarise(stepsday=sum(steps))
# Plot the histogram
hist(steps_day$stepsday,xlab="Steps per day",main="Total number of steps taken per day",col="red")
```

![plot1](Plot/plot1.png)<!-- -->

```r
# Calculate the media and the median of the total number of steps taken per day 
mean(steps_day$stepsday)
```

```
## [1] 10766.19
```

```r
median(steps_day$stepsday)
```

```
## [1] 10765
```

The mean total number of steps taken per day is 10,766 and the median is 10,765.

## What is the average daily activity pattern?

First calculate average steps for each interval for all days


```r
# Calculate average steps
steps_interval<-group_by(cc,interval)%>% summarise(stepsint=mean(steps))
```

Then plot the average steps by interval.


```r
plot(stepsint~interval,data=steps_interval,type="l",xlab="Interval",ylab="Average number of steps",main="Average daily activity pattern")
```

![plot2](Plot/plot2.png)<!-- -->

Now, find the interval with most steps.


```r
max_steps<-steps_interval[steps_interval$stepsint==max(steps_interval$stepsint),1]
max_steps
```

```
## [1] 835
```

The interval with most steps is 835

## Imputing missin values

Calculate the number missing values in the dataset.


```r
# Filter the rows with missing values
missing<-data[!complete.cases(data),]
nrow(missing)
```

```
## [1] 2304
```

The number of missing values in the dataset is 2304.

To replace the missing values, use the average of existing data. We create the dataset complete_data, with missing values replace.


```r
# Calculate the mean of total number steps per day for replace the missing data
steps_day<- aggregate(steps ~ interval, data = data, FUN = median)
r_data<-numeric()
for (i in 1:nrow(data)) {
    ind <- data[i, ]
    if (is.na(ind$steps)) {
        steps <- subset(steps_day, interval == ind$interval)$steps
    } else {
        steps <- ind$steps
    }
    r_data <- c(r_data, steps)
}
```

Create a new dataset with imputed data

```r
complete_data<-data
complete_data$steps<-r_data
```

Contruct the histogram with the new dataset and recalculate the mean and the median.


```r
idsteps_day<-group_by(complete_data,date)%>% summarise(stepsday=sum(steps))
hist(idsteps_day$stepsday,xlab="Steps per day",main="Total number of steps taken per day with imputed data",col="blue")
```

![plot3](Plot/plot3.png)<!-- -->

```r
# Calculate the media and the median of the total number of steps taken per day 
mean(idsteps_day$stepsday)
```

```
## [1] 9503.869
```

```r
median(idsteps_day$stepsday)
```

```
## [1] 10395
```

The mean of steps per day in the dataframe with imputed data is 9,503 and the median is 10,395, differs from previous results, where the mean was 10766 and the median was 10765. Thus, the effects of the imputation of data is large , because the values change drastically.

## Are there differences in activity patterns between weekdays and weekends?


```r
# Classifying days
days <- c("lunes", "martes","miÃ©rcoles", "jueves", "viernes")
complete_data$week <- as.factor(ifelse(is.element(weekdays(complete_data$date),days), "Weekday", "Weekend"))
# Calculate the average of steps by interval and type the day
steps_total<-group_by(complete_data,interval,week)%>% summarise(steps=mean(steps))
xyplot(steps_total$steps ~ steps_total$interval|steps_total$week, main="Average Steps per Day by Interval",xlab="Interval", ylab="Steps",layout=c(1,2), type="l")
```

![plot4](Plot/plot4.png)<!-- -->
