# Intro to Exploratory Data Analysis (EDA)

```r
install.packages("UsingR")
library(UsingR)
x=father.son$fheight
round(sample(x, 20), digits = 1) # rounded to the first tenth decimal 
```


## Histogram! 
```r
hist(x, breaks=seq(floor(min(x)), ceiling(max(x))), 
     main = "Height Histogram", xlab = "Height in Inches")
```

1. breaks argument: Where to draw the lines/intervals and to record the number of individuals
- a vector giving the breakpoints between histogram cells,
- a function to compute the vector of breakpoints,
- a single number giving the number of cells for the histogram,
- a character string naming an algorithm to compute the number of cells (see ‘Details’),
- a function to compute the number of cells.

2. min(x) and max(x): Finds the lowest (62.4) and highest (71.8) values in your dataset.
3. floor(62.4) rounds down to 62.
4 .ceiling(71.8) rounds up to 72.
5. The seq() function generates a sequence of numbers between two points. Since you didn't specify a step size, it defaults to counting by 1s.


Histogram examples
```r
sum(age>=35 & age<45)
```




## Empirical cumulative distribution function (ECDF) 

Estimates the true cumulative distribution function of a sample. (probability that one gets a value equal to or less than that value). In other words, the percentage of of the sample that is at or below a given threshold.  

Although not as popular as the histogram for EDA, the empirical cumulative density function (CDF) shows us the same information and does not require us to define bins. For any number x the empirical CDF reports the proportion of numbers in our list smaller or equal to x. R provides a function that has as out the empirical CDF function. 

```r
myCDF <- ecdf(x) 
xs <- seq(floor(min(x)), ceiling(max(x)), 0.1)
plot(xs, ecdf(x)(xs), type = "l", 
    xlab="Height in inches", ylab  = "F(x)")
```
1-pnorm(72, mean(x), sd(x))


```r
mean(x>70) # proportion of individuals > 70 inches. Goes through the list to check if each value > 70 and then returns a bunch of TRUE or FALSE. Then calculates proportion by taking the mean (TRUE = 1, FALSE = 0). 
1-pnorm(70, mean(x), sd(x)) # Normal approximation. The pnorm() calculates the probability that a value is less than or equal to 70 in a perfect Normal distribution with the exact same mean and standard deviation (sd) as your data x. the 1-pnorm then calculates the probability that a randomly selected value is equal or LARGER on the distribution.   

```



## qq Plot - the Quantile-Quantile Plot

A quantile-quantile (Q-Q) plot is a visual graph used to check if a set of data follows a specific theoretical model, like a normal distribution, or to see if two data sets match each other. It plots the quantiles of your sample data against expected theoretical values.

In this scenario, we would check to see if our dataset `x=father.son$fheight` fits a normal distribution. 

```r
x=father.son$fheight # dataset 
ps <- seq(0.01, 0.99, 0.01) # seq makes a sequence of numbers (probabilities for each percentile) to use the quantile() function on. 
qs <- quantile(x, ps) # x, our dataset as a numeric vector, ps, a numeric vector of probabilities with values in [0,1].
normalqs <- qnorm(ps, mean(x), sd(x))
plot(normalqs, qs, xlab="Normal percentiles", ylab="Height Percentiles")
# makes a generic (x, y) plot which becomes our QQ plot  
abline(0, 1) # arguments: intercept, slope
```

ALTERNATIVELY: 
```r
qqnorm(x)
qqline(x)
```

Both do the same thing. ACK. 


---

### QQ Plot Exercises 

```r
load("skew.RData")
dim(dat) # Even though the object "dat" wasn't created in this script (like .csv files where it was assigned a variable in read(), with RDATA files it doesn't need it; they saved the actual R objects along with their names.)


# Using QQ-plots, compare the distribution of each column of the matrix to a normal. That is, use qqnorm() on each column. To accomplish this quickly, you can use the following line of code to set up a grid for 3x3=9 plots. (mfrow means we want a multifigure grid filled in row-by-row.Another choice is mfcol.)

par(mfrow = c(3,3))  


# par() sets graphical parameters — global settings for the base plotting system in R. Think of it as adjusting the canvas before you paint. Any parameter you set stays in effect for every subsequent plot in that session until you change it back.

# The command above carves the canvas into a 3×3 grid of nine cells. Now each call to qqnorm() doesn't wipe the canvas and start over; it drops into the next empty cell. After nine plots the grid is full, and the tenth plot clears everything and starts a fresh grid.

# par(mfrow = c(1, 1)) → a 1×1 grid, i.e. one plot filling the whole canvas. That's R's default, so this line is how you undo the change.

# mfrow stands for multi-figure by row. You give it c(rows, columns). mfrow fills the cells by rows, while mfcol fills by columns. 


# Then you can use a for loop, to loop through the columns, and display one qqnorm() plot at a time. 

for (i in 1:9) {
  qqnorm(dat[, i], main = paste("Column", i))
  qqline(dat[, i])
}


par(mfrow = c(1, 1)) # resets graphics settings to plot one at a time
```

`Error in plot.new() : figure margins too large `

This simply means that you need to expand the size of the plot window. 

Which columns are skewed? 
```r
    hist(dat[,4]) # Positive skew (tail to the right)
    hist(dat[,9]) # Negative skew (long tail to the left)
```




## Boxplots

What if the data isn't even *close* to being normally distributed? 

Consider a dataset of the salaries of people in a company (within UsingR). 

```r
hist(exec.pay)
qqnorm(exec.pay)
qqline(exec.pay)
boxplot(exec.pay, ylab="10000s of dollars", ylim=c(0, 400))
```
The boxplot shows the median, the 25th and 75th percentiles in the box, the maximum and minimum ranges (without outliers), and the outliers as defined by R: anything 1.5 times the Interquartile Range (IQR) beyond the upper or lower quartiles. 


### Boxplot exercises - featuring split() 

head(InsectSprays)

Use split and formula to do the same thing. 

```r
par(mfrow = c(3,2))
tapply(InsectSprays$count, InsectSprays$spray, boxplot)
```
Makes 6 different boxplots on 6 different axes, but each with its own axes. 

```r
par(mfrow = c(1, 1))

# Both commands make boxplots by plotting the count statistics for each spray type

# 1) split()
boxplot(split(InsectSprays$count, InsectSprays$spray))

# 2) formula — reads as "count as a function of spray"
boxplot(count ~ spray, data = InsectSprays) #also...
boxplot(InsectSprays$count ~ InsectSprays$spray)

# Then, you can use your eye, or use 
tapply(InsectSprays$count, InsectSprays$spray, median)
#... to list out each median value
```

`split()` takes a vector and a grouping variable of the same length, then hands you back a named list — one element per group, holding all the values that belonged to that group. It's a gathering operation: scattered values get collected into buckets.


Example: 

```r
split(c(5, 8, 3, 9, 4, 7), c("a", "b", "a", "b", "a", "b"))
# $a
# [1] 5 3 4

# $b
# [1] 8 9 7
```

```r

# Let's consider a random sample of finishers from the New York City Marathon in 2002. This dataset can be found in the UsingR package. Load the library and then load the nym.2002 dataset.

library(dplyr)
data(nym.2002, package="UsingR")

# Use boxplots and histograms to compare the finishing times of males and females. Which of the following best describes the difference?


nym2002dim <- dim(nym.2002)
head(nym.2002)
class(nym.2002)
par(mfrow = c(2, 1))


males <- filter(nym.2002, gender=="Male")
hist(males$time, main="time")
boxplot(males$time)

females <- filter(nym.2002, gender=="Female")
hist(females$time, main="time")
boxplot(females$time)

par(mfrow = c(1, 1))
boxplot(time ~ gender, data = nym.2002)

boxplot(filter(nym.2002, gender=="Male") %>% dplyr::select(time))



##Model Solution
par(mfrow = c(1, 3))
males <- filter(nym.2002, gender=="Male") %>% dplyr::select(time) %>% unlist
females <- filter(nym.2002, gender=="Female") %>% dplyr::select(time) %>% unlist
boxplot(females, males)
hist(females,xlim=c(range( nym.2002$time)))
hist(males,xlim=c(range( nym.2002$time)))


## Gem's solution
par(mfrow = c(1, 3))
# Extract the time vector directly using base R subsetting
males <- nym.2002$time[nym.2002$gender == "Male"]
females <- nym.2002$time[nym.2002$gender == "Female"]

boxplot(females, males)
hist(females, xlim=c(range(nym.2002$time)))
hist(males, xlim=c(range(nym.2002$time)))
```

 Interesting note about the model solution
The model solution doesn't work (the wrong `select()` function.) without the `dplyr::` before the `select()`.  You'd get "Error in select(., time) : unused argument (time)"

This usually occurs when the MASS package is loaded after the dplyr package. The MASS::select() function overwrites (masks) the dplyr::select() function, and since the MASS version doesn't know what to do with column names like time, it throws the unused argument error. Therefore, force R to do it with `dplyr::select()`. 