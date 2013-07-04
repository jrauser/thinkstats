Notes on Think Stats Chapter 1
------------------------------
Author: John Rauser  
Date: 2013-05-11


I could not agree more with the thesis of the book, that "if you know how to program, you can use that skill to help you understand probability and statistics."  Statistics is most frequently taught as mathematical statistics, rather than practical applied statistics.  This is a hold-over from one hundred years ago, when mathematical analysis were the only way to access statistical ideas.  

Statistics, in some sense, is the study of what happens when you repeat a random process over and over again.  The most important and deep ideas in statistics are difficult to access via mathematical analysis, but with a few lines of code we can do statistics directly.  The computer does the repetitions and collects the results, and the simple beauty of statistics is revealed.


Exercise 1-1
============

The study isn't logitudinal because it doesn't track the same people over time.  Each of the six samples include a different set of respondents.

Exercise 1-2
============

Here's R code that implements `survey.py`.


```r
fem_resp_raw <- readLines("2002FemResp.dat")
fem_resp <- transform(data.frame(record = fem_resp_raw), caseid = as.integer(substr(record, 
    1, 12)))

fem_resp$record <- NULL
nrow(fem_resp)
```

```
## [1] 7643
```



```r
fem_preg_raw <- readLines("2002FemPreg.dat")

# Normally we'd use read.fwf() here, but I don't want need all the fields,
# and I don't want to type in all that metadata.
fem_preg <- transform(data.frame(record = fem_preg_raw), caseid = as.integer(substr(record, 
    1, 12)), nbrnaliv = as.integer(substr(record, 22, 22)), babysex = factor(substr(record, 
    56, 56)), birthwgt_lb = as.integer(substr(record, 57, 58)), birthwgt_oz = as.integer(substr(record, 
    59, 60)), prglength = as.integer(substr(record, 275, 276)), outcome = factor(substr(record, 
    277, 277)), birthord = as.integer(substr(record, 278, 279)), agepreg = as.integer(substr(record, 
    284, 287))/100, finalwgt = as.numeric(substr(record, 423, 440)))

fem_preg <- transform(fem_preg, totalwgt_oz = ifelse(birthwgt_lb < 20 & birthwgt_oz <= 
    16, birthwgt_lb * 16 + birthwgt_oz, NA))

fem_preg$record <- NULL
nrow(fem_preg)
```

```
## [1] 13593
```



Exercise 1-3
============

Part 2: There were 9,148 live births.  Below is the count of each outcome.


```r
table(fem_preg$outcome)
```

```
## 
##    1    2    3    4    5    6 
## 9148 1862  120 1921  190  352
```


This is consistent with [the documentation for `prgoutcome`](http://www.icpsr.umich.edu/nsfg6/Controller?displayPage=labelDetails&fileCode=PREG&section=&subSec=8014&srtLabel=611785).

Part 3: 

Weird. It looks like outcome is recorded for first births only.  The data is at odds with [the documentation for `prgorder`](http://www.icpsr.umich.edu/nsfg6/Controller?displayPage=labelDetails&fileCode=PREG&section=&subSec=8014&srtLabel=611775).


```r
table(fem_preg$birthord, fem_preg$outcome)
```

```
##     
##         1    2    3    4    5    6
##   1  4413    0    0    0    0    0
##   2  2874    0    0    0    0    0
##   3  1234    0    0    0    0    0
##   4   421    0    0    0    0    0
##   5   126    0    0    0    0    0
##   6    50    0    0    0    0    0
##   7    20    0    0    0    0    0
##   8     7    0    0    0    0    0
##   9     2    0    0    0    0    0
##   10    1    0    0    0    0    0
```

```r
sum(table(fem_preg$birthord))
```

```
## [1] 9148
```


Part 4:



```r
# Make an is_first indicator variable
fem_preg <- transform(fem_preg, is_first = (birthord == 1))
```



```r
library(plyr)
ddply(fem_preg, .(is_first), summarize, n = length(prglength), mean_preglength = mean(prglength))
```

```
##   is_first    n mean_preglength
## 1    FALSE 4735           38.52
## 2     TRUE 4413           38.60
## 3       NA 4445           10.95
```

```r
(38.6 - 38.523) * (24 * 7)
```

```
## [1] 12.94
```


Below is a plot of the mean pregnancy length by birth-order, live births only.  Each subsequent baby comes earlier on average?


```r
library(ggplot2)
birthord_summary<-ddply(subset(fem_preg, !is.na(birthord)), .(birthord), 
                        summarize, mean_preglength=mean(prglength))
ggplot(birthord_summary, aes(factor(birthord),mean_preglength))+
  geom_point()+
	xlab("Pregnancy length (weeks)") + ylab("Birth order")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7.png) 





```r
save(fem_preg, fem_resp, file = "thinkstats.RData")
```


