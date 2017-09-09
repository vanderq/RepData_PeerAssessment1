### Introduction

This R Markdown document describes the Data Analysis performed for the
Assignment of week 2 of the Coursera Course "Reproducible Research".

Step 1 is loading the data into a data-frame, which is put into the
variable stepsData.

    stepsData <- read.csv("activity.csv", na.strings="NA")

### Number of steps per day

First we have a look at the number of steps per day. We calculate it
(using the aggregate function), and display it in a barplot. As we need
to plot the total number of steps for each day (2 variables), we need a
barplot instead of a histogram.

    stepsPerDay <- aggregate(stepsData$steps, list(stepsData$date), sum, na.rm=TRUE)
    colnames(stepsPerDay) <- c("Date", "Steps")
    barplot(stepsPerDay$Steps, names.arg=stepsPerDay$Date, xlab="Date", ylab="Steps per day", main="Total number of steps per day")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-2-1.png)

The mean and the median values:

    mean(stepsPerDay$Steps)

    ## [1] 9354.23

    median(stepsPerDay$Steps)

    ## [1] 10395

### Average daily pattern

The average daily pattern of the steps taken on a day can be obtained by
looking at the same interval across multiple days. The plot below shows
the average number of steps for each interval over all days in the data
set.

    avgStepsPerInterval <- aggregate(stepsData$steps, list(stepsData$interval), mean, na.rm=TRUE)
    colnames(avgStepsPerInterval) <- c("Interval", "Steps")
    plot(avgStepsPerInterval$Interval, avgStepsPerInterval$Steps, type="l", xlab="Interval (starting minute)", ylab="Average number of Steps", main="Average number of steps per interval of 5 minutes across various days" )

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-4-1.png)

The interval with the highest number of average steps over all days is:

    avgStepsPerInterval[avgStepsPerInterval$Steps == max(avgStepsPerInterval$Steps),]

    ##     Interval    Steps
    ## 104      835 206.1698

### Insert missing values

The dataset unfortunately has missing values. The number of missing
values, and the percentage:

    sum(is.na(stepsData$steps))

    ## [1] 2304

    mean(is.na(stepsData$steps))

    ## [1] 0.1311475

To deal with the missing values, we will set the average value of each
interval across all days (excluding missing values) into the dataSet.

    stepsDataMerged <- merge(stepsData, avgStepsPerInterval, by.x = "interval", by.y = "Interval", all.x = TRUE)
    colnames(stepsDataMerged) <- c("interval", "steps", "date", "avgSteps")
    stepsDataMerged$newSteps <- ifelse(is.na(stepsDataMerged$steps), stepsDataMerged$avgSteps, stepsDataMerged$steps)

The new overview of steps per day:

    stepsPerDayWithNA <- aggregate(stepsDataMerged$newSteps, list(stepsDataMerged$date), sum)
    colnames(stepsPerDayWithNA) <- c("Date", "Steps")
    barplot(stepsPerDayWithNA$Steps, names.arg=stepsPerDayWithNA$Date, xlab="Date", ylab="Steps per day", main="Total number of steps per day")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-8-1.png)

The mean and the median values:

    mean(stepsPerDayWithNA$Steps)

    ## [1] 10766.19

    median(stepsPerDayWithNA$Steps)

    ## [1] 10766.19

As can be seen, by imputing the missing values the average and the
median values have increased. This is probably because there were days
without steps (sum = 0), which have now a value of the average. This is
also the reason why the median is now equal to the mean. Several days
have been imputed by the avarege value (because we have imputed the NAs
by the average of all days for each interval).

### Difference between weekdays and weekend days

To be able to check whether there is a difference between weekdays and
weekend days we create a multi-panel plot showing for weekdays and
weekend days the average number of steps for each interval over all days
in the data set.  
First we need to add a variable which makes the distinction between a
weekday and a weekend day.

    stepsDataMerged$date <- as.Date(stepsDataMerged$date)
    stepsDataMerged$DayOfWeek <- weekdays(stepsDataMerged$date)
    stepsDataMerged$dayType <- ifelse(stepsDataMerged$DayOfWeek %in% c("Saturday", "Sunday"), "Weekend", "Weekday")

With the datType variable showing the distinction between a weekday and
a weekday, we can calculate the averages per interval for both values,
and create a multi-panel plot

    stepsDataWeekend <- stepsDataMerged[stepsDataMerged$dayType == 'Weekend',]
    avgStepsPerIntervalWeekend <- aggregate(stepsDataWeekend$newSteps, list(stepsDataWeekend$interval), mean)
    stepsDataWeekday <- stepsDataMerged[stepsDataMerged$dayType == 'Weekday',]
    avgStepsPerIntervalWeekday <- aggregate(stepsDataWeekday$newSteps, list(stepsDataWeekday$interval), mean)
    colnames(avgStepsPerIntervalWeekend) <- c("Interval", "Steps")
    colnames(avgStepsPerIntervalWeekday) <- c("Interval", "Steps")

    rng <- range(avgStepsPerIntervalWeekday$Steps, avgStepsPerIntervalWeekend$Steps)

    par(mfrow=c(2,1), mar=c(5,4,2,1), oma=c(0,0,3,0))
    plot(avgStepsPerIntervalWeekend$Interval, avgStepsPerIntervalWeekend$Steps, type="l", xlab="Interval (starting minute)", ylab="Avg number of Steps", main="Weekend Days", ylim = rng)

    plot(avgStepsPerIntervalWeekday$Interval, avgStepsPerIntervalWeekday$Steps, type="l", xlab="Interval (starting minute)", ylab="Avg number of Steps", main="Week Days", ylim = rng)

    mtext("Comparison between average steps per interval between weekend days and weekdays", side=3, outer=T, cex=1)

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-11-1.png)
