---
title: "Reproducible Research: Peer Assessment 1 - Sam Vennell"
output: html_document
keep_md: true
---





## Loading and preprocessing the data

### Initial set up: 

```r
library(knitr)
library(rmarkdown)
library(dplyr)
library(ggplot2)
```

###Loading in the data:

```r
activity<-read.csv("activity/activity.csv")
```


```r
### Aggregate and groom the data
activity_agg<-activity %>%
  group_by(date) %>%
  summarise(totalsteps=sum(steps))
```


```r
activity_alldays<-activity %>%
  group_by(interval) %>%
  summarise(meansteps=mean(steps, na.rm=TRUE))
```


## What is mean total number of steps taken per day?

### Histogram showing the total number of steps taken per day:


```r
hist(activity_agg$totalsteps, xlab = "Steps per Day", main="Steps per Day", col="blue", breaks = seq(0,25000,2500))
```

<img src="PA1_template_files/figure-html/hist1-1.png" title="" alt="" width="1152" />

### The mean and median total number of steps taken per day:


```r
mean(activity_agg$totalsteps, na.rm=TRUE)
```

```
## [1] 10766.19
```

```r
median(activity_agg$totalsteps, na.rm=TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

### The average number of steps taken in each 5-minute interval:


```r
ggplot(activity_alldays, aes(x=interval, y=meansteps)) +
    geom_line(color="blue") + 
    labs(x="5-minute interval", y="Mean number of steps", title= "Average Number of Steps taken in Each 5 Minute Interval")
```

<img src="PA1_template_files/figure-html/avg_nonimputed-1.png" title="" alt="" width="1152" />

### The 5-minute interval which, on average, contains the maximum number of steps


```r
with(activity_alldays, interval[which(meansteps==max(meansteps))])
```

```
## [1] 835
```

## Imputing missing values

### How many missing values are there in the dataset?


```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

### We have opted to perform a simple imputation by replacing missing values with the mean for its 5-minute interval.  This is implemented as follows:


```r
activity_imputed <- activity %>% 
                      left_join(activity_alldays, by="interval") %>%
                      mutate(steps=ifelse(is.na(steps), meansteps, steps)) %>%
                      select(-meansteps)
```

### A histogram of the total number steps per day based on the imputed data:


```r
activity_imputed_agg<-activity_imputed %>%
                        group_by(date) %>%
                        summarise(totalsteps=sum(steps))

hist(activity_imputed_agg$totalsteps, xlab = "Steps per Day", main="Steps per Day", col="blue", breaks = seq(0,25000,2500))
```

<img src="PA1_template_files/figure-html/hist2-1.png" title="" alt="" width="1152" />

### The mean and median total number of steps for the imputed data:


```r
mean(activity_imputed_agg$totalsteps, na.rm=TRUE)
```

```
## [1] 10766.19
```

```r
median(activity_imputed_agg$totalsteps, na.rm=TRUE)
```

```
## [1] 10766.19
```

Note that the mean calculated is exactly the same as that of the original data, within working precision.  The median also does not differ appreciably from the non-imputed data.

## Are there differences in activity patterns between weekdays and weekends?

### We create a aggregated data frame with a factor variable which distinguishes weekdays from weekends and contains the average number of steps taken in each 5-minute interval seperately for weekdays and for weekends:


```r
activity_imputed_weekdays <- mutate(activity_imputed,
                              weekday = factor(
                                ifelse(
                                  weekdays(as.Date(date)) %in% c("Saturday","Sunday"), 
                                  "weekend", "weekday"
                                  )
                                )
                            ) %>%
                                group_by(weekday,interval) %>%
                                summarise(meansteps=mean(steps))

levels(activity_imputed_weekdays$weekday)<-c("Weekday","Weekend")
```

### Using this data frame we draw a faceted plot showing the average number of steps taken in each 5-minute interval for weekdays and for weekends


```r
ggplot(activity_imputed_weekdays, aes(x=interval, y=meansteps, color=weekday)) +
    geom_line()+
    facet_wrap(~weekday)+
    labs(x="5-minute interval", y="Mean number of steps", 
         title= "Average Number of Steps taken in Each 5 Minute Interval")+
    theme(legend.position="none")
```

<img src="PA1_template_files/figure-html/meanstepsweekdays-1.png" title="" alt="" width="1152" />
