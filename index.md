---
title       : Data Manipulation with R
author      : Kaushik Roy Chowdhury
framework   : deckjs
deckjs      :
  theme     : swiss
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : googlecode           # 
widgets     : [mathjax]            # {mathjax, quiz, bootstrap}
mode        : standalone # {standalone, draft}
knit        : slidify::knit2slides
---

<style>
code {
fontsize: 12pt;
}
pre {
font-size: -1em;
}
</style>

# Import data into R


---


## Importing data into R
Conforming to its' philosophy of freedom (of choice), reading data into R can be performed in various ways. 

### Reading data with Base R functions
* Widely used functions for reading data into R
* Available with base R distribution
* No packages required
* Can be slow and take up surprising amount of memory when reading large files
* `read.table()` and family


__Usage__: `read.table(file, header = FALSE, sep = "", colClasses = NA, stringsAsFactors = TRUE, ...)`

---


## Is there a fast and efficient way to read-in data?

- `data.table()` package provides an alternative.
- `fread()` _fast file reader function_ is a fast and efficient way to read in data into R


__Usage__: `fread(input, ...)` where `...` takes in same arguments as that of `read.table`.

---


## Timing `read.table()` and `fread()` with a 20MB .csv file

The file `flights.csv` can be downloaded from [here](http://bit.ly/1L4IFxB)


```r
# install packages if not present
#install.packages(c("data.table", "rbenchmark")) 

# load install packages
library(data.table); library(rbenchmark)

# file saved in windows default directory (~ = C:/Users/.../Documents)
read_base <- function(x) raw <- read.csv("~/flights.csv")
read_DT   <- function(x) rawDT <<- fread("~/flights.csv")

# reading a 20MB .csv file
benchmark(read_base(), read_DT(), replications = 1,
  columns = c("test", "elapsed")) 
```

```
##          test elapsed
## 1 read_base()    3.12
## 2   read_DT()    0.28
```

`fread()` is almost 5x as fast as `read.csv()`

---


## A look at the data

Having read the `flights.csv` data into R using `fread()` function, here are the first few rows


```
##    year month day dep_time dep_delay arr_time arr_delay carrier tailnum
## 1: 2013     1   1      517         2      830        11      UA  N14228
## 2: 2013     1   1      533         4      850        20      UA  N24211
## 3: 2013     1   1      542         2      923        33      AA  N619AA
## 4: 2013     1   1      544        -1     1004       -18      B6  N804JB
## 5: 2013     1   1      554        -6      812       -25      DL  N668DN
## 6: 2013     1   1      554        -4      740        12      UA  N39463
##    flight origin dest air_time distance hour minute
## 1:   1545    EWR  IAH      227     1400    5     17
## 2:   1714    LGA  IAH      227     1416    5     33
## 3:   1141    JFK  MIA      160     1089    5     42
## 4:    725    JFK  BQN      183     1576    5     44
## 5:    461    LGA  ATL      116      762    5     54
## 6:   1696    EWR  ORD      150      719    5     54
```

The data contains flights information of all planes that departed NYC (i.e. JFK, LGA or EWR airports) in 2013.

__Data Source__: [RITA, Bureau of transportation statistics](http://www.transtats.bts.gov/DL_SelectFields.asp?Table_ID=236)

---


# Data step of `data.table()`


---


## Deep-dive into `data.table()` package

`data.table()` provides __faster__ and __efficient__ manipulation of data stored in RAM

Perform the following operations:
  - Filtering relevant data/rows
  - Creating new variables, modify/delete columns
  - Group by operations
  - Summarize data
  - Ordered joins
  - Transpose data

__General form:__
> `DT[i, j, by]`: Take DT, subset rows using `i`, calculate `j` grouped by `by`


---

# Filter/Subset

---


## Filtering data 


`DT[i, j, by]`: Take DT, subset rows using `i`,<span style="opacity: 0.15;"> calculate `j`, grouped by `by` </span>

Rows can be filtered using column names satisfying conditions

- Select all flights departing from JFK airport and has a departure delay of more than 30 minutes


```r
head(rawDT[origin == "JFK" & dep_delay > 30])
```

```
##    year month day dep_time dep_delay arr_time arr_delay carrier tailnum
## 1: 2013     1   1      826        71     1136        51      AA  N3GVAA
## 2: 2013     1   1      848       853     1001       851      MQ  N942MQ
## 3: 2013     1   1      909        59     1331        16      AA  N5EXAA
## 4: 2013     1   1     1337        77     1649        78      B6  N636JB
## 5: 2013     1   1     1428        59     1803        83      B6  N635JB
## 6: 2013     1   1     1515        38     1834        52      B6  N178JB
##    flight origin dest air_time distance hour minute
## 1:    443    JFK  MIA      160     1089    8     26
## 2:   3944    JFK  BWI       41      184    8     48
## 3:    655    JFK  STT      184     1623    9      9
## 4:    673    JFK  LAX      352     2475   13     37
## 5:    355    JFK  BUR      371     2465   14     28
## 6:    347    JFK  SRQ      171     1041   15     15
```


---

# Manipulating columns (adding/updating)

---

## Create new variables (standalone)

`DT[i, j, by]`: Take DT, <span style="opacity: 0.2;"> subset rows using `i`, </span> calculate `j` <span style="opacity: 0.2;"> grouped by `by` </span>

New variables can be created in the `j` argument of data table operation

- Calculate air speed for each flight (= distance/air_time)


```r
head(rawDT[, .(air_speed = distance/air_time)])
```

```
##    air_speed
## 1:  6.167401
## 2:  6.237885
## 3:  6.806250
## 4:  8.612022
## 5:  6.568966
## 6:  4.793333
```

- `.()` is an alias for `list()` to perform multiple operations separated by ','
- If `.()` is not used, the result is a vector, else the result is a `data.table`

---

## Adding new variables to `data.table`

To add the `air_speed` variable in the `rawDT data.table`, use `:=` operator



```r
head(rawDT[, air_speed := distance/air_time])
```

```
##    year month day dep_time dep_delay arr_time arr_delay carrier tailnum
## 1: 2013     1   1      517         2      830        11      UA  N14228
## 2: 2013     1   1      533         4      850        20      UA  N24211
## 3: 2013     1   1      542         2      923        33      AA  N619AA
## 4: 2013     1   1      544        -1     1004       -18      B6  N804JB
## 5: 2013     1   1      554        -6      812       -25      DL  N668DN
## 6: 2013     1   1      554        -4      740        12      UA  N39463
##    flight origin dest air_time distance hour minute air_speed
## 1:   1545    EWR  IAH      227     1400    5     17  6.167401
## 2:   1714    LGA  IAH      227     1416    5     33  6.237885
## 3:   1141    JFK  MIA      160     1089    5     42  6.806250
## 4:    725    JFK  BQN      183     1576    5     44  8.612022
## 5:    461    LGA  ATL      116      762    5     54  6.568966
## 6:   1696    EWR  ORD      150      719    5     54  4.793333
```

---

## Adding multiple new variables to `data.table`

- To add multiple variables, `air_speed` and `total_delay (=dep_delay + arr_delay)` use a chained operation
- Chaining operations improves readability and avoids intermediate assignments


```r
# in rawDT[, create var1][, create var2][print rows 1:3]
rawDT[, air_speed := distance/air_time][, total_delay := dep_delay + arr_delay][1:3]
```

```
##    year month day dep_time dep_delay arr_time arr_delay carrier tailnum
## 1: 2013     1   1      517         2      830        11      UA  N14228
## 2: 2013     1   1      533         4      850        20      UA  N24211
## 3: 2013     1   1      542         2      923        33      AA  N619AA
##    flight origin dest air_time distance hour minute air_speed total_delay
## 1:   1545    EWR  IAH      227     1400    5     17  6.167401          13
## 2:   1714    LGA  IAH      227     1416    5     33  6.237885          24
## 3:   1141    JFK  MIA      160     1089    5     42  6.806250          35
```



---

# Group by operations

---

## Grouped Operations

`DT[i, j, by]`: Take DT, <span style="opacity: 0.2;"> subset rows using `i`, calculate `j` </span> grouped by `by`

- Calculate the average air speed for each carrier
- This can be achieved by calculating the air speed variable and take an average across all flights __grouped by__ carrier
- Make `na.rm = TRUE` which removes the `NA` (missing values) from the data when calculating the `mean`


```r
# calculate average air speed by carrier and print rows 1 to 5
rawDT[, .(avg_air_speed = mean(distance/air_time, na.rm = TRUE)), by = carrier][1:5]
```

```
##    carrier avg_air_speed
## 1:      UA      7.014730
## 2:      AA      6.957879
## 3:      B6      6.666191
## 4:      DL      6.974380
## 5:      EV      6.049060
```


---

# Summarize

---


## Data summary

Summarize data using necessary arguments of `i`, `j` and `by` of `data.table`

__Examples:__
- Calculate daily count of all flights departing from JFK airport (`origin == "JFK`)
- For all flights flying out of JFK airport (`origin == "JFK`), find the carrier  with maximum average departure delay 
- In which month of the year does flights departing from JFK airport has the maximum departure delay?




---




### Daily count of flights departing from JFK airport


```r
head(rawDT[origin == "JFK", .N, by = day])
```

```
##    day    N
## 1:   1 3663
## 2:   2 3661
## 3:   3 3696
## 4:   4 3638
## 5:   5 3608
## 6:   6 3640
```



---



### Carrier with maximum average departure delay for flights departing from JFK airport


```r
head(rawDT[origin == "JFK", .(avg_dep_delay = mean(dep_delay, na.rm = TRUE)), by = carrier][order(-avg_dep_delay)])
```

```
##    carrier avg_dep_delay
## 1:      9E      19.00152
## 2:      EV      18.52036
## 3:      VX      13.27944
## 4:      MQ      13.19997
## 5:      B6      12.75745
## 6:      AA      10.30216
```



---



### Month with maximum average departure delay for flights departing from JFK airport



```r
smryDT <- rawDT[origin == "JFK", .(avg_dep_delay = mean(dep_delay, na.rm = TRUE)), by = month]

# setorder works much faster than base::order from previous example
setorder(smryDT, -avg_dep_delay)
head(smryDT)
```

```
##    month avg_dep_delay
## 1:     7      23.76926
## 2:     6      20.49973
## 3:    12      14.78835
## 4:     8      12.91436
## 5:     5      12.51943
## 6:     4      12.24906
```


---

# Join/Merge

---


## Ordered Joins

Joins in `data.table` are performed using `merge.data.table()` function. However, `data.tables` need to be sorted by `key(s)` which are established using `setkey()` for an existing `data.table` or `key` argument while creating one

- Create two `data.tables` with same `key` value to join


```r
dt1 <- data.table(A = letters[1:10], X = 1:10, key = "A"); head(dt1)
```

```
##    A X
## 1: a 1
## 2: b 2
## 3: c 3
## 4: d 4
## 5: e 5
## 6: f 6
```

```r
dt2 <- data.table(A = letters[5:14], Y = 1:10, key = "A"); head(dt2)
```

```
##    A Y
## 1: e 1
## 2: f 2
## 3: g 3
## 4: h 4
## 5: i 5
## 6: j 6
```


---


### Left Outer Join


```r
# left outer join
merge.data.frame(x = dt1, y = dt2, all.x=TRUE)
```

```
##    A  X  Y
## 1  a  1 NA
## 2  b  2 NA
## 3  c  3 NA
## 4  d  4 NA
## 5  e  5  1
## 6  f  6  2
## 7  g  7  3
## 8  h  8  4
## 9  i  9  5
## 10 j 10  6
```

---

### Right Inner Join


```r
# right inner join
merge.data.frame(x = dt2, y = dt1, all.x=FALSE)
```

```
##   A Y  X
## 1 e 1  5
## 2 f 2  6
## 3 g 3  7
## 4 h 4  8
## 5 i 5  9
## 6 j 6 10
```

---

### Full Outer Join


```r
# full outer join
merge.data.frame(x = dt1, y = dt2, all = TRUE)
```

```
##    A  X  Y
## 1  a  1 NA
## 2  b  2 NA
## 3  c  3 NA
## 4  d  4 NA
## 5  e  5  1
## 6  f  6  2
## 7  g  7  3
## 8  h  8  4
## 9  i  9  5
## 10 j 10  6
## 11 k NA  7
## 12 l NA  8
## 13 m NA  9
## 14 n NA 10
```

---

# Transposing data

---

## Reshaping data `melt`

`data.tables` can be reshaped using the `melt` and `dcast` functions:

- __`melt`__: Wide-to-long (melting)

__Usage__: `melt(data, id.vars, measure.vars, variable.name = "variable", value.name = "value", ...)` 

where,

`data` A `data.table` to melt

`id.vars` vector of id variables; if missing, all non-id columns are assigned

`measure.vars` vector of measure variables; if missing, all non-id columns are assigned

`variable.name` name for the measured variable names column

`value.name` name for the molten data values column

`...` advanced argument for `melt` functions

---

### Example

Create the data to melt


```r
library(reshape2)
DT <- data.table(
      i1 = c(1:3, NA), 
      i2 = c(5, 6, 7, 8), 
      f1 = c("A", "C", "D", "Q"), 
      c1 = c("XY", "FE", "AA", "GG"))
DT
```

```
##    i1 i2 f1 c1
## 1:  1  5  A XY
## 2:  2  6  C FE
## 3:  3  7  D AA
## 4: NA  8  Q GG
```


---


### Melt the data


```r
(DT.melt1 <- melt(DT, id = c("i1", "i2"), measure = c("f1", "c1")))
```

```
##    i1 i2 variable value
## 1:  1  5       f1     A
## 2:  2  6       f1     C
## 3:  3  7       f1     D
## 4: NA  8       f1     Q
## 5:  1  5       c1    XY
## 6:  2  6       c1    FE
## 7:  3  7       c1    AA
## 8: NA  8       c1    GG
```

```r
#rename variable and value columns
(DT.melt2 <- melt(DT, id = c("i1", "i2"), measure = c("f1", "c1"), variable.name = "Factors", value.name = "data_value"))
```

```
##    i1 i2 Factors data_value
## 1:  1  5      f1          A
## 2:  2  6      f1          C
## 3:  3  7      f1          D
## 4: NA  8      f1          Q
## 5:  1  5      c1         XY
## 6:  2  6      c1         FE
## 7:  3  7      c1         AA
## 8: NA  8      c1         GG
```


---

## Reshaping data `dcast`

- __`dcast.data.table`__: Long-to-wide (casting)

__Usage__: `dcast.data.table(data, formula, fun.aggregate = NULL, ...)` 

where,

`data` A molten data.table

`formula` A formula of the form LHS ~ RHS to cast, eg: var1 + var2 ~ var3. The first varies slowest, and the last fastest.  "..." represents all other variables not used in the formula and "." represents no variable

`fun.aggregate` Aggregation function needed if variables do not identify a single observation for each output cell

`...` other advanced arguments

---

### `dcast` the molten `data.table` 


```r
(DT.dcast <- dcast.data.table(DT.melt2, i1+i2~Factors))
```

```
## Using 'data_value' as value column. Use 'value.var' to override
```

```
##    i1 i2 f1 c1
## 1: NA  8  Q GG
## 2:  1  5  A XY
## 3:  2  6  C FE
## 4:  3  7  D AA
```
