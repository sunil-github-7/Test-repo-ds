R Markdown
----------

This is an R Markdown document. Markdown is a simple formatting syntax
for authoring HTML, PDF, and MS Word documents. For more details on
using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that
includes both content as well as the output of any embedded R code
chunks within the document. You can embed an R code chunk like this:

    #Install packages and call their libraries
    library("ggplot2")
    library("data.table")
    library("gtools")

    library("Hmisc")

    ## Loading required package: lattice

    ## Loading required package: survival

    ## Loading required package: Formula

    ## 
    ## Attaching package: 'Hmisc'

    ## The following objects are masked from 'package:base':
    ## 
    ##     format.pval, units

    zip = "repdata%2Fdata%2Factivity.zip"
    link = "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
    dir = "data"

    # File download verification. If file does not exist, download to working directory.
    if(!file.exists(zip))
    {
      download.file(link, zip , mode = "wb") 
    }

    # File unzip verification. If the directory does not exist, unzip the downloaded file.
    if(!file.exists(dir))
    {
      unzip("repdata%2Fdata%2Factivity.zip", exdir="data")
      getwd()
    }

    setwd("D:/data")
    # File Read data file.
    activity<-read.csv("activity.csv", header=T)
    View(activity)
    names(activity)

    ## [1] "steps"    "date"     "interval"

    stepsday = tapply(activity$steps, activity$date, sum, na.rm=TRUE)
    View(stepsday)
    stepsdayDF=data.frame(stepsday)
    View(stepsdayDF)

Including Plots
---------------

Histogram

    ggplot(stepsdayDF, aes(x = stepsday)) +
        geom_histogram(fill = "blue", binwidth = 1000) +
        labs(title = "Daily Steps", x = "Steps", y = "Frequency")

![](Repro_Proj1_files/figure-markdown_strict/histogram-1.png)

Mean and median number of steps taken each day
----------------------------------------------

Mean and Median

    summary(stepsdayDF)

    ##     stepsday    
    ##  Min.   :    0  
    ##  1st Qu.: 6778  
    ##  Median :10395  
    ##  Mean   : 9354  
    ##  3rd Qu.:12811  
    ##  Max.   :21194

    #Mean
    mean(stepsdayDF$steps,na.rm=TRUE)

    ## [1] 9354.23

    # Media
    median(stepsdayDF$steps,na.rm=TRUE)

    ## [1] 10395

Time series plot of the average number of steps taken
-----------------------------------------------------

Time series

      #REmove NA
    stepstime = aggregate(activity$steps, by=list(activity$interval), mean, na.rm=TRUE)
    names(stepstime)[1] = "Interval"
    names(stepstime)[2] = "Steps"
    names(stepstime)

    ## [1] "Interval" "Steps"

    # Plot
    ggplot(stepstime, aes(x = Interval,y=Steps)) +
      labs(title = "Time series plot of the average number of steps taken", x = "interval", y = "steps")+
      geom_line(color="BLUE") 

![](Repro_Proj1_files/figure-markdown_strict/TimeSeries-1.png)

The 5-minute interval that, on average, contains the maximum number of steps
----------------------------------------------------------------------------

Maximum steps

      #Max steps 
    stepstime = aggregate(activity$steps, by=list(activity$interval), mean, na.rm=TRUE)
    names(stepstime)[1] = "Interval"
    names(stepstime)[2] = "Steps"
    names(stepstime)

    ## [1] "Interval" "Steps"

    summary(stepstime)

    ##     Interval          Steps        
    ##  Min.   :   0.0   Min.   :  0.000  
    ##  1st Qu.: 588.8   1st Qu.:  2.486  
    ##  Median :1177.5   Median : 34.113  
    ##  Mean   :1177.5   Mean   : 37.383  
    ##  3rd Qu.:1766.2   3rd Qu.: 52.835  
    ##  Max.   :2355.0   Max.   :206.170

    maxsteps = stepstime[which.max(stepstime$Steps),]
    maxsteps

    ##     Interval    Steps
    ## 104      835 206.1698

Code to describe and show a strategy for imputing missing data
--------------------------------------------------------------

Strategy to replace data

    # generate Mean Data2 by interval
    stepstime = aggregate(activity$steps, by=list(activity$interval), mean, na.rm=TRUE)
    names(stepstime)[1] ="interval"
    names(stepstime)[2] ="steps"

    meanint=mean(stepstime$interval)
    meanint

    ## [1] 1177.5

    meanstep=mean(stepstime$steps)
    meanstep

    ## [1] 37.3826

    round(meanstep)

    ## [1] 37

    meanstep

    ## [1] 37.3826

    activityMiss=activity



    activityMiss$steps <- impute(activityMiss$steps, round(meanstep))
    activityMiss$steps=round(activityMiss$steps,digits=0)
    View(activityMiss)

Including Plots
---------------

Histogram

    stepsdayMiss = tapply(activityMiss$steps, activity$date, sum, na.rm=TRUE)
    View(stepsdayMiss)
    stepsdayMissDF=data.frame(stepsdayMiss)
    View(stepsdayMissDF)


    ggplot(stepsdayMissDF, aes(x = stepsday)) +
        geom_histogram(fill = "blue", binwidth = 1000) +
        labs(title = "Daily Steps with no Missing values", x = "Steps", y = "Frequency")

![](Repro_Proj1_files/figure-markdown_strict/histogram-2-1.png)

Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends
---------------------------------------------------------------------------------------------------------

Panel plot

    activityMiss$day =  ifelse(as.POSIXlt(activityMiss$date)$wday %in% c(0,6), 'weekend', 'weekday')
    View(activityMiss)


    names(activityMiss)

    ## [1] "steps"    "date"     "interval" "day"

    ggplot(activityMiss , aes(x = interval , y = steps, color=day))  + facet_grid(day ~ . )  +  labs(title = "Avg steps", x = "interval", y = "steps")  + geom_line()

    ## Don't know how to automatically pick scale for object of type impute. Defaulting to continuous.

![](Repro_Proj1_files/figure-markdown_strict/panel-1.png)
