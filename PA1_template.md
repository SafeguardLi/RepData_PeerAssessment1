Loading and preprocessing the data
----------------------------------

At the very begining, we need to check the existence of files in a
certain directory. Here come the codes.

    wd <- getwd()
     if(!file.exists("./data")){
    +     dir.create("./data")  
    }
    if(!file.exists("./data/repdata_data_activity.zip")){
    +     url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
    +     download.file(url, destfile="./data/repdata_data_activity.zip", method="curl")
    }
     if(!file.exists("./data/activity.csv")){
    +     zipF<-file.choose() 
    +     outDir<-"./data" 
    +     unzip(zipF,exdir=outDir)
    } 

Now, let's load the data!

    setwd("./data")
    rawdata <- read.csv("activity.csv")

To preprocess the data set, here we only need to change the format of
date *from factor to date*.

    #Take a glance first
    summary(rawdata)

    ##      steps                date          interval     
    ##  Min.   :  0.00   2012-10-01:  288   Min.   :   0.0  
    ##  1st Qu.:  0.00   2012-10-02:  288   1st Qu.: 588.8  
    ##  Median :  0.00   2012-10-03:  288   Median :1177.5  
    ##  Mean   : 37.38   2012-10-04:  288   Mean   :1177.5  
    ##  3rd Qu.: 12.00   2012-10-05:  288   3rd Qu.:1766.2  
    ##  Max.   :806.00   2012-10-06:  288   Max.   :2355.0  
    ##  NA's   :2304     (Other)   :15840

    str(rawdata)

    ## 'data.frame':    17568 obs. of  3 variables:
    ##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

    #preprocess it
    mydata <- rawdata
    mydata$date <- as.Date(mydata$date)

What is mean total number of steps taken per day?
-------------------------------------------------

Before following calculations, we need to drop all the missing values in
the data set.

    nona <- mydata[complete.cases(mydata),]
    str(nona)

    ## 'data.frame':    15264 obs. of  3 variables:
    ##  $ steps   : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ date    : Date, format: "2012-10-02" "2012-10-02" ...
    ##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...

Now, calculate the total number of steps taken per day.

    #Create a new dataframe to store the total number of steps each day
    totalsteps <- aggregate(nona$steps, by=list(Date=nona$date), FUN=sum)
    names(totalsteps)<-c("Date","Steps")
    totalsteps

    ##          Date Steps
    ## 1  2012-10-02   126
    ## 2  2012-10-03 11352
    ## 3  2012-10-04 12116
    ## 4  2012-10-05 13294
    ## 5  2012-10-06 15420
    ## 6  2012-10-07 11015
    ## 7  2012-10-09 12811
    ## 8  2012-10-10  9900
    ## 9  2012-10-11 10304
    ## 10 2012-10-12 17382
    ## 11 2012-10-13 12426
    ## 12 2012-10-14 15098
    ## 13 2012-10-15 10139
    ## 14 2012-10-16 15084
    ## 15 2012-10-17 13452
    ## 16 2012-10-18 10056
    ## 17 2012-10-19 11829
    ## 18 2012-10-20 10395
    ## 19 2012-10-21  8821
    ## 20 2012-10-22 13460
    ## 21 2012-10-23  8918
    ## 22 2012-10-24  8355
    ## 23 2012-10-25  2492
    ## 24 2012-10-26  6778
    ## 25 2012-10-27 10119
    ## 26 2012-10-28 11458
    ## 27 2012-10-29  5018
    ## 28 2012-10-30  9819
    ## 29 2012-10-31 15414
    ## 30 2012-11-02 10600
    ## 31 2012-11-03 10571
    ## 32 2012-11-05 10439
    ## 33 2012-11-06  8334
    ## 34 2012-11-07 12883
    ## 35 2012-11-08  3219
    ## 36 2012-11-11 12608
    ## 37 2012-11-12 10765
    ## 38 2012-11-13  7336
    ## 39 2012-11-15    41
    ## 40 2012-11-16  5441
    ## 41 2012-11-17 14339
    ## 42 2012-11-18 15110
    ## 43 2012-11-19  8841
    ## 44 2012-11-20  4472
    ## 45 2012-11-21 12787
    ## 46 2012-11-22 20427
    ## 47 2012-11-23 21194
    ## 48 2012-11-24 14478
    ## 49 2012-11-25 11834
    ## 50 2012-11-26 11162
    ## 51 2012-11-27 13646
    ## 52 2012-11-28 10183
    ## 53 2012-11-29  7047

Then, make a histgram of the total number of steps taken each day.

    hist(totalsteps$Steps, main="Total Number of Steps taken Each Day", xlab="Steps", ylim= c(0,30))

![](PA1_template_files/figure-markdown_strict/histgram-1.png)<!-- -->

Calculate and report the mean and median of the total number of steps
taken each day.

    #Calculate mean and median
    mean <- mean(totalsteps$Steps)
    median <- median(totalsteps$Steps)

So the mean of the total number of steps is 1.076618910^{4} and the
median of that is 10765

What is the average daily activity pattern?
-------------------------------------------

To understand the average daily activity pattern, we coudl first make a
time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and
the average number of steps taken, averaged across all days (y-axis).

    #calculate the average number of steps taken
    pattern <- aggregate(nona$steps, by=list(Intervals = nona$interval),FUN=mean)
    names(pattern) <- c("Intervals","AvgSteps")
    #make a plot
    with(pattern, plot(Intervals, AvgSteps, type="n", ylab="Average Number of Steps taken", xlab="Intervals",main="Time Series Plot"))
    with(pattern, lines(Intervals, AvgSteps, type="l",col="red"))

![](PA1_template_files/figure-markdown_strict/timeplot-1.png)<!-- -->

Also, we could tell which 5-minute interval, on average across all the
days in the dataset, contains the maximum number of steps.

    maxium <- pattern$Intervals[which(pattern$AvgSteps==max(pattern$AvgSteps))]

The 5-minute interval which contains the maximum number of steps is 835.

Imputing missing values
-----------------------

Before imputing missing values, we could calculate and report the total
number of missing values in the dataset (i.e. the total number of rows
with NA)

    totalNA <- sum(is.na(mydata$steps))

The total number of missing values in the dataset is 2304.

Then, we could create a newdataset **newdata** with all of the missing
values filled in. Here we use **the mean for that 5-minute interval**.

    newdata <- mydata
    na <- is.na(newdata$steps)
    newdata[na, ]$steps <- mean(pattern$AvgSteps)

After that, we could make the histogram of the total number of steps
taken each day and the mean and median total number of steps taken per
day; then, compare these values with the first part of this assignment
to see the difference.

    #make a new histgram
    totalsteps2 <- aggregate(newdata$steps, by=list(Date=newdata$date), FUN=sum)
    names(totalsteps2)<-c("Date","Steps")
    par(mfrow=c(1,2),mar=c(4,4,2,1),cex.main=0.6)
    hist(totalsteps$Steps, main="Total Number of Steps taken Each Day", xlab="Steps", ylim= c(0,40))
    hist(totalsteps2$Steps, main="Total Number of Steps taken Each Day(Revised)", xlab="Steps", ylim= c(0,40))

![](PA1_template_files/figure-markdown_strict/compare-1.png)<!-- -->

    #recalculate the mean and the median
    mean2 <- mean(totalsteps2$Steps)
    median2 <- median(totalsteps2$Steps)

The **new** mean of the total number of steps is 1.076618910^{4} and the
**new** median of that is 1.076618910^{4}, while the **former** mean is
1.076618910^{4} and the **former** median is 10765.

By filling in missing values with mean for that 5-minute interval, we
could see clearly that **the frequency of total number of steps around
10,000 increases while other parts remain the same**, and that **the
mean remains the same while the meidan change from 10765 to the
1.076618910^{4}, exactly the mean**.

Are there differences in activity patterns between weekdays and weekends?
-------------------------------------------------------------------------

Create a new factor variable in the dataset with two levels - “weekday”
and “weekend” indicating whether a given date is a weekday or weekend
day.

    Sys.setlocale("LC_TIME","US")
    weekdata <- newdata
    weekdata$weekday <- weekdays(weekdata$date)
    weekdata$type <- with(weekdata, ifelse(weekday=="Saturday"|weekday=="Sunday","weekend","weekday"))
    weekdata$type <- as.factor(weekdata$type)

Finally, make a panel plot containing a time series plot of the 5-minute
interval (x-axis) and the average number of steps taken, averaged across
all weekday days or weekend days (y-axis).

    pattern2 <- aggregate(weekdata$steps, by=list(Intervals = weekdata$interval, Type=weekdata$type),FUN=mean)
    names(pattern2) <- c("Intervals","Type","AvgSteps")
    library(lattice)
    xyplot(AvgSteps~Intervals|Type,data=pattern2,layout=c(1,2),type="l",ylab="Number of Steps")

![](PA1_template_files/figure-markdown_strict/plot2-1.png)<!-- -->

END
