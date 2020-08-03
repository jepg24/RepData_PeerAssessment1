---

title: "Reproducible Research: Peer Assessment 1"
author: "Jesús Pérez"
date: "7/30/2020"
output: 
  html_document:
    keep_md: true

---

# Activity monitoring

## 1. Loading and opening the data


```r
## Loading the file as a zip file
path <- getwd()
url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(url, destfile = paste(path, "Files.zip", sep = "/"))
unzip(zipfile = "Files.zip")

## Open the data
data <- read.csv("activity.csv", header = TRUE)
str(data)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

```r
head(data)
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

Now we can see some parameters such as maximum, minimum, range and the NA's values of each variable using the **summary** function.


```r
summary(data)
```

```
##      steps                date          interval     
##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
##  Median :  0.00   2012-10-03:  288   Median :1177.5  
##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
##  NA's   :2304     (Other)   :15840
```

### 1.1 subsetting the data

Before plot the histogram we need to subset the data. For that purpose the **aggregate** function was used. The NA's values were desconsired.


```r
df <- aggregate(data$steps, by=list(date=data$date), FUN=sum)
df <- df[!is.na(df$x),]
summary(df)
```

```
##          date          x        
##  2012-10-02: 1   Min.   :   41  
##  2012-10-03: 1   1st Qu.: 8841  
##  2012-10-04: 1   Median :10765  
##  2012-10-05: 1   Mean   :10766  
##  2012-10-06: 1   3rd Qu.:13294  
##  2012-10-07: 1   Max.   :21194  
##  (Other)   :47
```

```r
head(df)
```

```
##         date     x
## 2 2012-10-02   126
## 3 2012-10-03 11352
## 4 2012-10-04 12116
## 5 2012-10-05 13294
## 6 2012-10-06 15420
## 7 2012-10-07 11015
```

## 2. Plotting the Total per day histogram

To plot the histogram the **ggplot2** package was used. This is the ***histogram*** and not the ***barplot***. A histogram is used to continuous variables and shows the frequency of that particularly variable.
 

```r
library(ggplot2)
p <- ggplot(df, aes(x=x))
p + geom_histogram(binwidth=1000, 
                   fill="#69b3a2", 
                   color="#e9ecef", 
                   alpha=0.7) + 
        labs(x= "Total steps per day", 
             y = "Frequency", 
             title = "Histogram of the total steps taken each day")
```

![](PA1_template_files/figure-html/histogram-1.png)<!-- -->

## 3. Mean and median 

The Mean and median number of steps taken each day can be obtained using the **summary** function (see section 7.1).


```r
summary(df$x)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##      41    8841   10765   10766   13294   21194
```
## 4. Time series plot of the average number of steps taken

For the second plot a new subsetting was made using the **aggregate** function, and then graphicated using **ggplot2**. 


```r
## Subsetting
df2 <- aggregate(data$steps, 
                 by=list(l=data$interval), 
                 FUN=mean, na.rm = TRUE)

## Calculating the max
maxx <- df2$l[which.max(df2$x)]

## Plotting 
p <-    ggplot(df2, aes(x = l, y = x)) +
        geom_line() + 
        labs(x= "Interval", y = "Average of steps taken", 
                title = "Average of steps taken by Time Interval") +
        geom_vline(xintercept= maxx, linetype="dashed", color = "red")
print(p)
```

![](PA1_template_files/figure-html/plot2-1.png)<!-- -->

## 5. Maximum number of steps by time interval

The 5-minute interval that, on average, contains the maximum number of steps was calculate using the **which.max** function (See plot 2) and the result was reported in hours.



```r
library(lubridate)
```

```
## 
## Attaching package: 'lubridate'
```

```
## The following objects are masked from 'package:base':
## 
##     date, intersect, setdiff, union
```

```r
## Calculating the max
maxx <- df2$l[which.max(df2$x)]

## Transform to hours
duration(maxx, "minutes")
```

```
## [1] "50100s (~13.92 hours)"
```

## 6. Imputing missing values

As there are days/intervals with missing values \textcolor{red}{NA} a new datasets was created. But, first the porcentage of \textcolor{red}{NA} was calculated. (Note that the total \textcolor{red}{NA} values were reported using the **summary** function in section **1**)


```r
NAt <- sum(is.na(data$steps))
t <- nrow(data)
NAp <- (NAt/t)*100
NAp
```

```
## [1] 13.11475
```

### 6.1 Filling in the missing values

The \textcolor{red}{NA} values were filled with the mean values for their respective 5-minute interval using the **mutate** function of the **dyplr** package. A new dataset identically to the original but with the missing data filled in was created.


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:lubridate':
## 
##     intersect, setdiff, union
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
df3 <- cbind(df2$x, data)
names(df3)[names(df3) == "df2$x"] <- "MeanSteps"
df4 <- df3 %>% mutate(MissingSteps = ifelse(is.na(steps), MeanSteps, steps))
df4 <- df4[, 2:5]
head(df4)
```

```
##   steps       date interval MissingSteps
## 1    NA 2012-10-01        0    1.7169811
## 2    NA 2012-10-01        5    0.3396226
## 3    NA 2012-10-01       10    0.1320755
## 4    NA 2012-10-01       15    0.1509434
## 5    NA 2012-10-01       20    0.0754717
## 6    NA 2012-10-01       25    2.0943396
```

## 7 Histogram of the total number of steps taken each day after missing values

The new histogram was created with the new dataset. The results show that are no evident differences between plot 1 and 3, that means the ausence of \textcolor{red}{NA} values did not affect the analysis.


```r
df5 <- aggregate(df4$MissingSteps, by=list(date=df4$date), FUN=sum)
p <- ggplot(df5, aes(x=x))
p + geom_histogram(binwidth=1000, 
                   fill="#404080", 
                   color="#e9ecef", 
                   alpha=0.6) + 
        labs(x= "Total steps per day", 
             y = "Frequency", 
             title = "Histogram of the total steps taken each day without NA values")
```

![](PA1_template_files/figure-html/plot3-1.png)<!-- -->

### 7.1 New Mean and Median

To show the mean and median of the new dataset a table was created. The analysis shows that there were not differences between the original dataset and the dataset imputed with \textcolor{red}{NA} values. So, imputing missing data on the estimates of the total daily number of steps did not alterate the results.


```r
Mean <- c(mean(df$x), mean(df5$x))
Median <- c(median(df$x), median(df5$x))
Comparisson <- cbind(Mean, Median)
Comparisson
```

```
##          Mean   Median
## [1,] 10766.19 10765.00
## [2,] 10766.19 10766.19
```

## 8. Activity patterns between weekdays and weekends

To analyse the differences in the activity patterns between weekdays and weekends a new dataset was created. For this reason, the **weekdays** function was used, first converting the date to Date objects and then calculating the DayType as a factor. 


```r
df4$date <- as.Date(df4$date)
day <- as.factor(weekdays(df4$date))
df6 <- cbind(df4, day)
weekend <- c("Satruday", "Sunday")
df6 <- df6 %>% mutate(DayType = ifelse(day == weekend, "Weekend", "Weekday"))
df6$DayType <- as.factor(df6$DayType)
head(df6)
```

```
##   steps       date interval MissingSteps    day DayType
## 1    NA 2012-10-01        0    1.7169811 Monday Weekday
## 2    NA 2012-10-01        5    0.3396226 Monday Weekday
## 3    NA 2012-10-01       10    0.1320755 Monday Weekday
## 4    NA 2012-10-01       15    0.1509434 Monday Weekday
## 5    NA 2012-10-01       20    0.0754717 Monday Weekday
## 6    NA 2012-10-01       25    2.0943396 Monday Weekday
```

### 8.1 Panel plot

With the new dataset the results were plot using **ggplot2**. These plots clrearly show an important difference between the Activity patterns between weekdays and weekends.


```r
df7 <- df6 %>% group_by(interval,DayType) %>% summarise(AverageSteps=mean(MissingSteps))
p <- df7 %>% ggplot(aes(x = interval, y = AverageSteps)) +
        geom_line() + 
        facet_grid(DayType ~ .) +
        labs(x= "Interval", y = "Average of steps taken by Time Interval", 
             title = "Average of steps taken by Time Interval and Day Type")
print(p)
```

![](PA1_template_files/figure-html/plot4-1.png)<!-- -->



