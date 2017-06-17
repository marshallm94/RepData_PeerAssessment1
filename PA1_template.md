# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

The following code chunk will load and process the data for analysis. **This 
code chunk is assuming the zipped data file ("activity.zip") is in your current
working directory.**


```r
setwd('/Users/marsh/datasciencecoursera/reproducible_research/course_project_1/')
unzip('activity.zip')
df <- read.csv('activity.csv')
df$date <- as.Date(df$date, "%Y-%m-%d")
```

## What is mean total number of steps taken per day?

Code to:  
1. Calculate number of steps taken per day.  
2. Create a histogram of number of steps taken per day.  
3. Calulate and report the mean and median number of steps taken per day.  


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
df1 <- group_by(df, date) %>% summarize(sum(steps))
hist(df1$`sum(steps)`, main = 'Histogram of Number of Steps Taken per Day',
     xlab = 'Number of Steps', col = 'blue')
```

![](PA1_template_files/figure-html/steps_per_day, histogram, mean_median-1.png)<!-- -->

```r
mean(df1$`sum(steps)`, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(df1$`sum(steps)`, na.rm = TRUE)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

Code to calculate the average daily pattern (interval number against number of steps taken per day, averaged across all days in dataset) and to report the interval number with the highest number of steps taken (on average).  


```r
df2 <- group_by(df, interval) %>% summarize(sum(steps, na.rm = TRUE))
plot(df2$interval, df2$`sum(steps, na.rm = TRUE)`, type = 'l', col = 'blue',
     xlab = "Interval Number", ylab = "Number of Steps (Averaged Across All Days)",
     main = "What is the Average Daily Pattern?")
```

![](PA1_template_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

```r
df2 <- arrange(df2, desc(`sum(steps, na.rm = TRUE)`))
head(df2$interval, n = 1)
```

```
## [1] 835
```

## Imputing missing values

Code to:  
1. Calculate the total number of NA's in the dataset.  
2. Impute mean of steps by interval as replacement for NA's and save as a new column (imputed_steps) in a new data frame (df4).    
3. Create a histogram of steps taken per day of  **new** dataset and calculate the mean and median of **new** dataset.  


```r
sum(is.na(df$steps))
```

```
## [1] 2304
```

```r
df3 <- group_by(df, interval) %>% summarize(mean(steps, na.rm = TRUE))
no_nas <- filter(df , is.na(steps) == FALSE)
nas <- filter(df, is.na(steps) == TRUE)
nas$imputed_steps <- df3$`mean(steps, na.rm = TRUE)`
no_nas$imputed_steps <- no_nas$steps
df4 <- rbind(nas, no_nas)
df5 <- group_by(df4, date) %>% summarize(sum(imputed_steps))
hist(df5$`sum(imputed_steps)`, main = 'Histogram of Number of Steps (Imputed) Taken per Day',
     xlab = 'Number of Steps', col = 'blue')
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
mean(df5$`sum(imputed_steps)`)
```

```
## [1] 10766.19
```

```r
median(df5$`sum(imputed_steps)`)
```

```
## [1] 10766.19
```

These values are practically the exact same as the mean and median of the original data set. The only difference is that the median of the new data set equals the mean exactly, whereas in the original data set the median was slightly below the mean. This follows logic, seeing as by imputing the mean of steps by interval as a replacement for missing values, I am bringing the overall distribution of steps closer to the mean, therefore bringing the median to the mean.

## Are there differences in activity patterns between weekdays and weekends?


```r
df4 <- mutate(df4, day = weekdays(df4$date))
day_type <- ifelse(df4$day %in% c('Saturday','Sunday'), 'weekend', 'weekday')
df4$day_type <- day_type
df6 <- group_by(df4, interval, day_type) %>% summarize(sum(imputed_steps))
par(mfrow = c(2,1),
    cex = 0.5,
    cex.lab = 2,
    cex.main = 2.5,
    mar = c(4,5,3,1))
with(filter(df6, day_type == 'weekday'), plot(interval, `sum(imputed_steps)`,
                                              type = 'l', xlab = "",
                                              ylab = "Steps Taken",
                                              main = "Weekday"))
with(filter(df6, day_type == 'weekend'), plot(interval, `sum(imputed_steps)`,
                                              type = 'l', main = "Weekend",
                                              ylab = "Steps Taken", xlab = "Interval Number",
                                              ylim = as.numeric(range(df6$`sum(imputed_steps)`))))
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->


