---
title: "Reproducible Research: Peer Assessment 1"
output: 
html_document:
keep_md: true
---


## Loading and preprocessing the data

1. Unpackage data file from within the *activity.zip* file. Read the data file *activity.csv* into a data.frame. In addition, I convert the date values into Date objects.

NOTE: I assumed only the data file *activity.csv* is within the zip file. Hence, I my routine does not check for multiple files, file names etc.

```r
unzip("activity.zip")
stepData <- read.csv("activity.csv")
stepData$date <- as.Date(stepData$date)
```


## What is mean total number of steps taken per day?
1. Aggregate the steps by date, using sum as the function. Plotted a Histogram of aggregate step data.

```r
aggdata <- aggregate(steps ~ date, data=stepData, sum)
hist(aggdata$steps, breaks = 25, xlab = "Steps", main = "Histogram of Steps")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 

2. Calculate the _Mean_ and _Medium_

```r
aggdataMean <- mean(aggdata$steps)
aggdataMedian <-median(aggdata$steps)
print(paste("The mean is", round(aggdataMean, 2)))
```

```
## [1] "The mean is 10766.19"
```

```r
print(paste("The medium is", aggdataMedian))
```

```
## [1] "The medium is 10765"
```



## What is the average daily activity pattern?
1. Aggregate the steps by interval period, using mean as the function. Produced a line plot, with interval as the x-axis and steps for the y-axis

```r
aggdata2 <- aggregate(steps ~ interval, data=stepData, mean)
plot(aggdata2, type = "l", main = "Line Plot of Steps versus 5-Minute Interval Times")
```

![plot of chunk unnamed-chunk-4](figure/unnamed-chunk-4-1.png) 

2. Ordered the aggregated step data, and then selected the first instance to show the highest step date in the set.

```r
orderAggData2 <- order(aggdata2$steps, decreasing = TRUE)
maxAggData2 <- aggdata2[orderAggData2[1],]
print(paste("The maximum 5-minute interval average across all days is", row.names(maxAggData2)))
```

```
## [1] "The maximum 5-minute interval average across all days is 104"
```

```r
# NOTE: The are only 288 5-minute intervals in a day.
# 288 = 24 * (60/5)
```



## Imputing missing values
1. Report the total number of rows with missing data (i.e. NA)

```r
sumNaDateData <- sum(is.na(stepData$date))
sumNaIntervalData <- sum(is.na(stepData$interval))
sumNaStepData <- sum(is.na(stepData$steps))

if(sumNaDateData != 0) {
    print(paste("The total number of rows with missing date data is", sumNaDateData))
    }

if(sumNaIntervalData != 0) {
    print(paste("The total number of rows with missing interval data is", sumNaIntervalData))
    }

if(sumNaStepData != 0) {
    print(paste("The total number of rows with missing step data is", sumNaStepData))
    }
```

```
## [1] "The total number of rows with missing step data is 2304"
```

2. The methodolgy I've used for filling missing data (i.e. NA values) is to iterate through the dataset and checking the step value for each row. For rows with a steps value equal to NA, I determine the interval and then create a subet of all rows for that time interval. From the subet I calculate the mean steps (omiting NA values), round the value to zero decimal places and then write the value back into the dataset.

NOTE: These operations are all performed on a cloned version of the original dataset.

3. Below is the code used to create the new dataset with the missing values filled in 

```r
tidyData <- stepData

n <- nrow(tidyData)

for(i in 1:n) {
    if(is.na(tidyData[i,1])) {
        subsetTidyData <- subset(tidyData, interval == tidyData[i,3])
        subsetTidyStepData <- na.omit(subsetTidyData$steps)
        tidyData[i,1] <- round(mean(subsetTidyStepData))
        }
    }
```

4. Plot a Histogram of the aggregate step for the new tidy dataset.

```r
aggTidyData <- aggregate(steps ~ date, data=tidyData, sum)
hist(aggTidyData$steps, breaks = 25, xlab = "Steps", main = "Histogram of Steps")
```

![plot of chunk unnamed-chunk-8](figure/unnamed-chunk-8-1.png) 

Calculate the _Mean_ and _Medium_ for the new tidy dataset.

```r
aggTidyDataMean <- mean(aggTidyData$steps)
aggTidyDataMedian <-median(aggTidyData$steps)

print(paste("The mean is", round(aggTidyDataMean, 2)))
```

```
## [1] "The mean is 10765.64"
```

```r
print(paste("The medium is", aggTidyDataMedian))
```

```
## [1] "The medium is 10762"
```


By overlying the histogram of the original dataset and the new tidy dataset the difference is clearly illustrated. The method used to fill the missing data has only increased the frequency of the mean, leaving all other bins unchanged.

```r
hist(aggTidyData$steps, breaks = 25, xlab = "Steps", main = "Overly of both Histograms", col="deepskyblue", ylim = c(0, 20))
par(new = TRUE)
hist(aggdata$steps, breaks = 25, xlab = "Steps", main = "", col="royalblue", ylim = c(0, 20))
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

## Are there differences in activity patterns between weekdays and weekends?
1. Create a character vector containing the day of the week, from the date values in the tidyData dataset and using the weekdays() funcation.

Then create the 'workDays' factor vector based on the day of the week, which indicates if the day is a "weekday" or "weekend". The factor vextor is then bound to the tidyData dataset.


```r
dayOfWeek <- weekdays(tidyData$date)
workDay <- factor(levels=c("weekend", "weekday"))

for(i in 1:n) {
    if(dayOfWeek[i] == "Saturday") {
        workDay[i] <- "weekend"
        }
    if(dayOfWeek[i] == "Sunday") {
        workDay[i] <- "weekend"
        }
    else {
        workDay[i] <- "weekday"
        }
    }    

tidyData <- cbind(tidyData, workDay)
```


2. Create seperate subet datasets for weekday and weekend data, and then aggregate the new datasets. The aggregation is steps by time interval.

The resulting aggregate data is then plotted using the base plot system.

```r
weekdayData <- subset(tidyData, workDay == "weekday", select = c(interval, steps, workDay))
weekendData <- subset(tidyData, workDay == "weekend", select = c(interval, steps, workDay))
aggdata3 <- aggregate(steps ~ interval, data=weekdayData, mean)
aggdata4 <- aggregate(steps ~ interval, data=weekendData, mean)

par(mfrow=c(2,1), mai = c(0.75, 1.5, 0.5, 1.5))
plot(aggdata3, type = "l", main = "Weekday" )
plot(aggdata4, type = "l", main = "Weekend")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 

