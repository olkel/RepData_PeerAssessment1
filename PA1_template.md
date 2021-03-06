# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

Loading the data:
```{r, echo=T}
setwd("Z:/Dokumente/Coursera/Reproducible Research/Assignment1/RepData_PeerAssessment1")
studis <- read.csv("activity.csv", header=T)
```


## What is mean total number of steps taken per day?

Rearrangin the data, aggregated by days and plot a histogram:
```{r, echo=T}
stepsPday <- tapply(studis$steps, studis$date, sum)
stepsPday <- stack(stepsPday)
hist(stepsPday$values, main="Steps per day", xlab="number of steps per day")
```

Statistics of the steps per day (mean, median):
```{r}
mean(stepsPday$values, na.rm=T)
median(stepsPday$values, na.rm=T)
```

## What is the average daily activity pattern?

Time series plot:
```{r}
timeList <- stack(tapply(studis$steps, studis$interval, mean, na.rm=T, simplify=T))
plot(timeList$values~timeList$ind, type="l", main="Steps over an Average Day", xlab="Daytime (n-th 5 minute interval)", ylab="Average steps in 5 minutes")
```

Interval with maximum average steps:
```{r}
timeList[which.max(timeList$values),]
```

## Imputing missing values

Total number of NA's:
```{r}
countNA <- 0
for (step in studis$steps) {
      if (is.na(step)){
            countNA <- countNA + 1
      }
}
countNA
```

Filling out the NA's:
```{r}
intBYdate <- reshape(studis, timevar="interval" , idvar="date", direction="wide")
aveDays <- stack(lapply(intBYdate[,-1], mean, na.rm=T))
aveDays[,2] <- substring(aveDays$ind,7)

studisWoNa <- studis
studisWoNa <- merge(studisWoNa, aveDays, by.x = "interval", by.y = "ind")

studisWoNa$steps[is.na(studisWoNa$steps)] <- studisWoNa$values[is.na(studisWoNa$steps)]
studisWoNa <- studisWoNa[,1:3]

```

Show first lines of result witout NA's:
```{r}
studisWoNa[1:20,]
```

Histogram without the NA's:
```{r}
stepsPdayWoNa <- tapply(studisWoNa$steps, studisWoNa$date, sum)
stepsPdayWoNa <- stack(stepsPdayWoNa)
hist(stepsPdayWoNa$values, main="Steps per day", xlab="number of steps per day")
```

Impact of imputing NA's:
```{r}
mean(stepsPday$values, na.rm=T)- mean(stepsPdayWoNa$values)
```

## Are there differences in activity patterns between weekdays and weekends?

Creating weekday column:
```{r}
daycat <- function(x){
      if (x == "Samstag" | x == "Sonntag"){
            "weekend"
      }else{
            "weekday"
      }
}
studisWoNa[,"daytype"]<-  stack(sapply(weekdays(as.Date(studisWoNa$date)),daycat))[,1]
```

Plotting:
```{r}
library("lattice")

timeListWeek <- aggregate(steps~interval+daytype, data = studisWoNa, mean)
xyplot(steps ~ interval | daytype, data = timeListWeek, layout = c(1,2), type= "l")
```