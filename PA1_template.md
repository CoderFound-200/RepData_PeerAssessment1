---
title: "PA1_template.Rmd"
author: "JK"
date: "5/11/2021"
output: 
  html_document: 
    keep_md: yes
---


This is the Coursera Data Science course "Reproducible Research" assignment for week 2.  
The following packages are used in the analysis.  


```r
library (dplyr) #Load dplyr for data summarizing.
library(tidyverse) #Load tidyverse for data wrangling.
library(ggplot2) #Load ggplot for making graphs.
```

Data was read into R (assumes that zip file is in working directory. For the first part of the assignment "NA" values were omitted.


```r
activity<-read.csv(unzip("repdata_data_activity.zip")) #Read in data and create dataframe.
str(activity) #Look at data structure.
activity$date<-as.Date(activity$date) #Reformat date column into date format.
activity_noNA <-na.omit(activity) #Remove steps=NA Rows.
```

What is the mean total number of steps taken per day?  
1. Calculate Total Number of Steps per day.


```r
TotalSteps <- activity_noNA %>% group_by(date) %>% summarize(TotalSteps=sum(steps)) 
```

2. Make a histogram of the total number of steps taken each day.


```r
hist(TotalSteps$TotalSteps, main="Total Number of Steps per Day", xlab="Total Steps per day", col=5)
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->


3.Calculate and report the mean and median of the total number of steps taken per day.


```r
meanStep <-mean(TotalSteps$TotalSteps)
medianStep <-median(TotalSteps$TotalSteps)
```

The mean number of total steps is **10766.1886792453** and the median number of total steps is **10765**.

What is the average daily activity pattern?  

1. Make a time series plot (i.e.  of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days ().

```r
AvgSteps <-activity_noNA %>% group_by(interval) %>% summarize(AvgSteps=mean(steps))
plot(AvgSteps$interval, AvgSteps$AvgSteps, type="l", main="Average Steps per 5 Minute Interval", xlab="Interval", ylab="Average Steps", col=4)
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
MAX <- AvgSteps$interval[which.max(AvgSteps$AvgSteps)]
```
The interval with the maximum number of steps is **835**.

Imputing missing values

1. Calculate and report the total number of missing values in the dataset.


```r
NAs<-sum(is.na(activity))
```
The number of NAs in the dataset is **2304**.

2. /3. Devise a strategy for filling in all of the missing values in the dataset.Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
activity_imp <- activity %>% group_by(interval) %>% mutate(steps_imp = ifelse(is.na(steps), mean(steps, na.rm=TRUE), steps)) #Using mean over interval to impute NAs.
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.


```r
TotalSteps_Imp <- activity_imp %>% group_by(date) %>% summarize(TotalSteps=sum(steps_imp))
```


```r
hist(TotalSteps_Imp$TotalSteps, main="Total Number of Steps per Day (Imputed NA with mean per interval)", xlab="Total Steps per day", col=5)
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->


```r
meanStep_Imp<-mean(TotalSteps_Imp$TotalSteps)
medianStep_Imp <-median(TotalSteps_Imp$TotalSteps)
```

After imputing the NAs using the mean across intervals the mean of total steps is **10766.1886792453** and the median is **10766.1886792453**. There is no difference in the mean following imputation and the median only changes slightly and is now the same as the mean.

Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.



```r
activity_imp_WD <-activity_imp %>% mutate(WD = weekdays(date)) #Adds column with weekday.
#Create Helpertable for weekdays and weekends.
WD<-c("Monday","Tuesday","Wednesday","Thursday", "Friday", "Saturday","Sunday")
daylevel<-c("Weekday", "Weekday" , "Weekday" , "Weekday" , "Weekday" , "Weekend" , "Weekend")
help_day<-tibble(WD,daylevel)
activity_imp_WD2 <-full_join(activity_imp_WD, help_day, by="WD") #Join Tables.
AvgSteps_imp_WD <-activity_imp_WD2 %>% group_by(interval,WD) %>% summarize(AvgSteps=mean(steps))
```

Create Panel Plot with two levels (Weekday, Weekend) for time series of 5 minute interval vs average steps.


```r
AvgSteps_imp_WD <-activity_imp_WD2 %>% group_by(interval, daylevel) %>% summarize(AvgSteps=mean(steps_imp))
ggplot(data = AvgSteps_imp_WD, aes(x=interval, y=AvgSteps)) + geom_line() + facet_wrap(~daylevel, nrow=2) + ggtitle("Average Steps per Interval") + ylab("Average Steps") #Create Plot.
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png)<!-- -->


