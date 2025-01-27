---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---

```r
# Load packages

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
library(ggplot2)
```

## Loading and preprocessing the data

Show any code that is needed to

1. Load the data (i.e. read.csv())

2. Process/transform the data (if necessary) into a format suitable for your analysis


```r
# Load file into data
data <- read.csv(unzip("activity.zip")) 

# Check structure of data 
str(data)                                                   
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
# Cast date column to class Date
data$date <- as.Date(data$date)                         
```

## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day

2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

3. Calculate and report the mean and median of the total number of steps taken per day  

- **The mean of the total number of steps per day is 10766 and the median is 10765.**


```r
# Calculate the total of steps taken per day
steps_per_day <- aggregate(steps ~ date, data, sum) 

# Report of mean and median of steps per day
summary(steps_per_day$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8841   10765   10766   13294   21194
```

```r
# Histogram of number of steps per day
hist(steps_per_day$steps, 
     col = 2, 
     main = "Histogram of total number of steps per day", 
     xlab = "Steps per day")
```

![](PA1_template_files/figure-html/steps_per_day-1.png)<!-- -->

## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = “l”) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?  

- **The max number of steps of an interval is 206.1698 and corresponds to interval #835**


```r
# Calculate the average of steps per interval
steps_per_interval <- aggregate(steps ~ interval, data, mean)

# Create a time series plot 
plot(steps_per_interval$interval, steps_per_interval$steps, 
     type='l',
     col = 2,
     main="Average number of steps per interval over all days", 
     xlab="Interval", 
     ylab="Average number of steps")
```

![](PA1_template_files/figure-html/average_daily-1.png)<!-- -->

```r
# Show interval with max number of steps
steps_per_interval[which.max(steps_per_interval$steps), ]
```

```
##     interval    steps
## 104      835 206.1698
```

## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)  

- **The total number of missing values in the dataset is 2304.**

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

- **The strategy selected for replacing NA’s was the mean for that 5-minute interval.**

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?  
- **The new dataset's summary is median: 10766 and Mean: 10766. The old dataset's numbers were Median: 10765 and Mean:10766. Therefore, there was a slight change on the median with the original dataset   **



```r
#Calculate the total number of missing values
sum(is.na(data))
```

```
## [1] 2304
```

```r
#Create a dataset with the replacement of NA's with the mean for 5-minute interval
data_without_NAs <- data

for (i in 1 : nrow(data_without_NAs)) {
        
        if (is.na(data_without_NAs$steps[i])) {
                interval <- data_without_NAs$interval[i]
                steps_interval <- steps_per_interval[steps_per_interval$interval == interval,]
                data_without_NAs$steps[i] <- steps_interval$steps
        }
}

# calculate the total number of steps taken each day new dataset without NA's
agg_data_without_NAs <- aggregate(steps ~ date, data_without_NAs, sum)

#Histogram of step per day of the new dataset without NA's 
hist(agg_data_without_NAs$steps, 
     col = 2, 
     main = "Histogram of total number of steps per day without NA's", 
     xlab = "Steps per day")
```

![](PA1_template_files/figure-html/imputing_na-1.png)<!-- -->

```r
# Report of mean and median of steps per day
summary(agg_data_without_NAs$steps)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    9819   10766   10766   12811   21194
```

## Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
#New variable added with two levels “weekday” and “weekend”
data_without_NAs$weekdayType <- ifelse(weekdays(data_without_NAs$date) %in% c("Satuday", "Sunday"), 
    "weekend", "weekday")

# calculate the average number of steps taken each day
agg_avg_data <- aggregate(steps ~ interval + weekdayType, data_without_NAs, mean)


# Plot Differences in activity patterns between weekdays and weekends
ggplot(agg_avg_data, aes(x=interval, y=steps, color=weekdayType)) + 
  facet_grid(weekdayType~.) +
  geom_line() + 
  labs(title="Differences in activity patterns between weekdays and weekends", 
       y="Average Number of Steps", 
       x="5-min Interval")
```

![](PA1_template_files/figure-html/differences_weekday_weekends-1.png)<!-- -->
