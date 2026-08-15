# swirl "R Programming" — Complete Cheat Sheet

Covers all 15 lessons of the swirl `R_Programming` course. Everything runnable in base R; no packages needed.

**Course lessons:** 1 Basic Building Blocks · 2 Workspace and Files · 3 Sequences of Numbers · 4 Vectors · 5 Missing Values · 6 Subsetting Vectors · 7 Matrices and Data Frames · 8 Logic · 9 Functions · 10 lapply and sapply · 11 vapply and tapply · 12 Looking at Data · 13 Simulation · 14 Dates and Times · 15 Base Graphics

Running it yourself:
```r
install.packages("swirl")
library(swirl)
install_course("R Programming")
swirl()          # bye() to exit, skip() to skip a question, play()/nxt() to experiment
```

---

## 1. Basic Building Blocks

R is an interactive calculator first, a language second.

```r
5 + 7            # 12
5 - 7; 5 * 7; 5 / 7; 5 ^ 2
```

### Assignment
```r
x <- 5 + 7       # <- is the idiomatic assignment operator
x                # printing = just type the name
x = 5            # works, but reserve = for function arguments
```

### Vectors with `c()`
`c()` = "concatenate". The vector is R's fundamental data structure — even a single number is a length-1 vector.

```r
z <- c(1.1, 9, 3.14)
z <- c(z, 555, z)     # c() flattens: 1.1 9 3.14 555 1.1 9 3.14
length(z)             # 7
```

### Vectorized arithmetic
Operations apply element-by-element, with no loop required.

```r
z * 2 + 100           # every element doubled, then +100
my_sqrt <- sqrt(z - 1)
sqrt(z); abs(z)
```

Other common arithmetic operators are `+`, `-`, `/`, and `^` (where x^2 means 'x squared'). To take the square root,
| use the sqrt() function and to take the absolute value, use the abs() function.




When dividing 2 vectors, the result is equal to the first element of z divided by the first element of the other, and so on...


### Recycling — the single most important gotcha here
When two vectors differ in length, the shorter one is **recycled** (repeated) to match the longer.

```r
c(1, 2, 3, 4) + c(0, 10)          # 1 12 3 14   (short vector recycles cleanly)
c(1, 2, 3, 4) + c(0, 10, 100)     # 1 12 103 4  + WARNING: not a multiple
```
R warns only when the longer length isn't a multiple of the shorter. Clean recycling is silent — that's what makes it dangerous.

For example: 
```r
c(1, 2, 3, 4) + c(0, 10, 100)
[1]   1  12 103   4
Warning message:
In c(1, 2, 3, 4) + c(0, 10, 100) :
  longer object length is not a multiple of shorter object length
```

### Getting help
```r
?c              # help page for a function
?`:`            # backticks for operators
help.start()    # full manual index
```

Press the **up arrow** to recall previous commands instead of retyping.
Press the **tab button** for variable autocomplete. 

---

## 2. Workspace and Files

R exposes one file API that works the same across Windows/macOS/Linux, so scripts stay portable.

| Task | Command |
|---|---|
| Current working directory | `getwd()`  |
| Change directory | `setwd("path")` <- if a variable, no quotes needed|
| Objects in workspace | `ls()` |
| Files in directory | `list.files()` or `dir()` |
| What arguments does a function take? | `args(list.files)`  <- no parentheses|
| Create directory | `dir.create("testdir")` |
| Create nested directories | `dir.create("a/b", recursive = TRUE)` |
| Create empty file | `file.create("mytest.R")` |
| Does it exist? | `file.exists("mytest.R")` |
| File metadata | `file.info("mytest.R")` |
| One metadata field | `file.info("mytest.R")$mode` |
| Rename | `file.rename("mytest.R", "mytest2.R")` |
| Copy | `file.copy("mytest2.R", "mytest3.R")` |
| Full path to a file | `file.path("mytest3.R")` |
| Build a portable path | `file.path("folder1", "folder2")`  <- inserts the path regardless of whether / or \ |
| Delete file | `file.remove("mytest.R")` |
| Delete directory + contents | `unlink("testdir2", recursive = TRUE)` |
| Wipe workspace | `rm(list = ls())` |

Two ideas worth internalizing:
- **`file.path()` over string pasting.** It inserts the correct separator (`/` vs `\`) for the OS, so `file.path("folder1", "folder2")` beats `"folder1/folder2"`. 

- `recursive = TRUE` tells R to walk down the path and create each missing level along the way: first testdir2, then testdir3 inside it.
Note that `file.path()` just builds a string — it doesn't check whether the folder exists or create anything. That's `dir.create()`.

- **Read the "See Also" section** of any help page. That's how you discover the sibling functions (`unlink` from `file.remove`, `dir` from `list.files`).

---

## 3. Sequences of Numbers

### The colon operator
```r
1:20             # 1 2 3 ... 20
pi:10            # 3.141593 4.141593 ... 9.141593  (stops before exceeding 10)
15:1             # counts down
```

### `seq()` — when you need control
```r
seq(1, 20)                  # same as 1:20
seq(0, 10, by = 0.5)        # fixed step size
seq(5, 10, length = 30)     # exactly 30 evenly spaced values
```

### Sequences derived from another object's length
```r
my_seq <- seq(5, 10, length = 30)
length(my_seq)              # 30
1:length(my_seq)            # works
seq(along.with = my_seq)    # same, more explicit
seq_along(my_seq)           # idiomatic and fastest
seq_len(length(my_seq))     # also fine
```
Prefer `seq_along(x)` over `1:length(x)` — if `x` is empty, `1:length(x)` gives `1 0`, which silently breaks loops.


`1:length(my_seq)`, or `seq(1, length(my_seq))`, both do the same thing! 


### `rep()` — repeating values
```r
rep(0, times = 40)              # forty zeros
rep(c(0, 1, 2), times = 10)     # 0 1 2 0 1 2 ...  (whole vector repeated)
rep(c(0, 1, 2), each = 10)      # 0 0 0 ... 1 1 1 ... 2 2 2  (each element repeated)
```
`times` vs `each` is a favorite exam distinction.

---

## 4. Vectors

The simplest and most common data structure in R is the vector.

Two flavors:
- **Atomic vectors** — all elements the same class. Flat, 1-dimensional array
- **Lists** — can mix classes (covered in lesson 10).  Hierarchical / Recursive (can contain nested lists)

Six atomic classes: **numeric (double), logical, character, integer, complex, raw**.

### Logical vectors
Elements are `TRUE`, `FALSE`, or `NA`. Comparison operators return logical vectors, elementwise:

```r
num_vect <- c(0.5, 55, -10, 6)
num_vect < 1        # TRUE FALSE TRUE FALSE
num_vect >= 6       # FALSE TRUE FALSE TRUE
```

| Operator | Meaning |
|---|---|
| `<` `>` `<=` `>=` | comparison |
| `==` | equal to (never `=`) |
| `!=` | not equal |
| `!` | NOT |
| `&` | AND (both are true?|
| `\|` | OR (at least one is true?) |

Example: 

`((111 >= 111) | !(TRUE)) & ((4 + 1) == 5)`   # true or false? 

it's... true! 




### Character vectors
```r
my_char <- c("My", "name", "is")     # a length-3 character vector
paste(my_char, collapse = " ")       # "My name is"  — collapse joins into ONE string
my_name <- c(my_char, "Matthew")
paste(my_name, collapse = " ")       # "My name is Matthew"
paste("Hello", "world!", sep = " ")  # sep joins across vectors, element by element
```

**`sep` vs `collapse`:** `sep` glues corresponding elements of several vectors together; `collapse` flattens one vector into a single string.

```r
paste(1:3, c("X", "Y", "Z"), sep = "")   # "1X" "2Y" "3Z"
paste(LETTERS, 1:4, sep = "-")           # "A-1" "B-2" ... recycles 1:4 across 26 letters
```
Note that `paste()` **coerces** numbers to characters automatically.

---

## 5. Missing Values

`NA` is not a value — it's a placeholder for something unknown. Therefore any operation involving `NA` returns `NA`.

```r
x <- c(44, NA, 5, NA)
x * 3            # 132 NA 15 NA
NA > 0           # NA
NA == NA         # NA  (you can't know whether two unknowns are equal)
```

Because of that last line, **never test for missingness with `==`, `e.g. my_data == NA`**. Use `is.na()`:

```r
y <- rnorm(1000)                  # 1000 draws from standard normal
z <- rep(NA, 1000)
my_data <- sample(c(y, z), 100)   # 100 drawn from the 2000 combined
my_na <- is.na(my_data)           # logical vector: TRUE where NA... gives a vector with all NA's 
```



### Counting with logicals
`TRUE` coerces to 1 and `FALSE` to 0, so summing a logical vector counts the `TRUE`s:

```r
sum(my_na)                # how many NAs
mean(my_na)               # proportion of NAs
```
This trick — `sum()` on a condition — is used constantly throughout the rest of the course.

### `NaN`, `Inf`, `NULL`
```r
0 / 0            # NaN  — "not a number", an undefined result
1 / 0            # Inf
-1 / 0           # -Inf
Inf - Inf        # NaN
```

| Value | Means | Test with |
|---|---|---|
| `NA` | missing / unknown | `is.na()` |
| `NaN` | undefined numeric result | `is.nan()` |
| `Inf` | infinity | `is.infinite()` |
| `NULL` | the object doesn't exist / zero length | `is.null()` |

`is.na()` returns `TRUE` for `NaN` too; `is.nan()` does not return `TRUE` for `NA`.

---

## 6. Subsetting Vectors

Everything happens through `[ ]`. There are exactly **four** kinds of index vector. 

### (a) Logical index — "give me elements where TRUE"
```r
x[1:10]                  # elements 1 to 10 in the vector 
x[is.na(x)]              # all the NAs (useless, but instructive)
y <- x[!is.na(x)]        # drop the NAs — the standard cleanup idiom
y[y > 0]                 # positive values, NA-free
x[x > 0]                 # WRONG: keeps NAs, because NA > 0, meaning NAs are included
x[!is.na(x) & x > 0]     # RIGHT: positive values, in one step
x[c(FALSE, TRUE, TRUE, FALSE, FALSE)]  # logical: "keep where TRUE, drop where FALSE" -> 20 30
```
This is the lesson's central point: **subsetting with a condition on a vector containing `NA` leaks `NA`s into your result.** Guard with `!is.na()`. 

Using logic with an index (like `x[!c(2, 10)]`, for example) *first converts 0 to `FALSE`, every other number to `TRUE` before applying the logic.* Therefore the numbers lose all their meaning as positions and become "is it 0?"  `x[!c(2, 10)]` becomes `x[c(FALSE, FALSE)]` and the sequence of FALSE is recycled to the length of the vector x. 

### (b) Positive integer index
```r
x[1:10]                  # R is 1-INDEXED, not 0-indexed, meaning the first element is #1, not #0
x[c(3, 5, 7)]            # selects index numbers 3, 5, 7
x[1]                     
x[0]                     # returns numeric(0) — an empty vector, not an error
x[3000]                  # NA — out of bounds gives NA, not an error
```
Both `x[0]` and `x[3000]` are silent, so off-by-one bugs don't announce themselves.

### (c) Negative integer index — "everything except"
```r
x[c(-2, -10)]            # all but the 2nd and 10th
x[-c(2, 10)]             # identical, and cleaner
```
You cannot mix positive and negative indices in one call.

### (d) Character index (names)
```r
vect <- c(foo = 11, bar = 2, norf = NA)  # Printing vect returns the names (without quotes) and the values of each
names(vect)                              # "foo" "bar" "norf"

vect2 <- c(11, 2, NA)                    # equivalent, built in two steps
names(vect2) <- c("foo", "bar", "norf")  # step 2: adds the names back. 
identical(vect, vect2)                   # TRUE

vect["bar"]                              # calls second element "bar"
vect[c("foo", "bar")]                    # 11 and 2
```
`identical()` is the correct way to ask "are these two objects exactly the same?" It returns TRUE or FALSE.
Used in logic too! It is Case-sensitive

---

## 7. Matrices and Data Frames

Both are rectangular (two-dimensional). The difference:
- **Matrix** — every element must be the same class.
- **Data frame** — each *column* has its own class. This is what real datasets look like.

### A matrix is just a vector with a `dim` attribute

`dim()` allows us to set or get the dim attribute. 
```r
my_vector <- 1:20               # vector with numbers from 1 to 20 
dim(my_vector)                  # NULL — vectors have no dim
length(my_vector)               # 20

dim(my_vector) <- c(4, 5)       # give it dimensions -> it becomes a matrix 
dim(my_vector)                  # 4 5
attributes(my_vector)           # $dim 4 5 -> the "get dimensions"
class(my_vector)                # "matrix" "array"
my_matrix <- my_vector
```
Rows first, then columns: `c(4, 5)` = 4 rows, 5 columns. Values fill **column-wise** by default.


| Input | Output | Comments |
|---|---|---|
| `my_vector <- 1:20` |  | no output |
| `my_vector` | [1]  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 |
| `dim(my_vector)` <- c(4, 5) | no output | sets the vector to a 4 row x 5 column matrix |
| my_vector |  See below please :) | printing the new vector! |


| | [,1] | [,2] | [,3] | [,4] | [,5] |
|---|---|---|---|---|---|
| **[1,]** | 1 | 5 | 9 | 13 | 17 |
| **[2,]** | 2 | 6 | 10 | 14 | 18 |
| **[3,]** | 3 | 7 | 11 | 15 | 19 |
| **[4,]** | 4 | 8 | 12 | 16 | 20 |



### The direct route
```r
my_matrix2 <- matrix(1:20, nrow = 4, ncol = 5) # renders the above matrix
identical(my_matrix, my_matrix2)          # checks whether matrices are identical (TRUE)
matrix(1:20, nrow = 4, byrow = TRUE)      # fill row-wise instead
```

### The coercion trap
```r
patients <- c("Bill", "Gina", "Kelly", "Sean")
cbind(patients, my_matrix)      # everything becomes CHARACTER STRINGS
```
`cbind()` must produce a matrix, and a matrix can hold only one class — so the numbers get coerced to strings (note the quotes in the output). This is **implicit coercion**: R silently picks the class that can hold everything.


### Data frames solve it

Data frames are matrix-like structures whose columns may be of differing types (numeric, logical, factor and character and so on).
```r
my_data <- data.frame(patients, my_matrix)
class(my_data)                  # "data.frame"

cnames <- c("patient", "age", "weight", "bp", "rating", "test")
colnames(my_data) <- cnames     # sets the colnames attribute of the data frame to cnames, the vector 
```

The function data.frame() creates data frames, tightly coupled collections of variables which share many of the properties of matrices and of lists, used as the fundamental data structure by most of R's modeling software. ?data.frame for more info. 

```r
Using cbind()...

> cbind(patients, my_matrix)
     patients                       
[1,] "Bill"   "1" "5" "9"  "13" "17"
[2,] "Gina"   "2" "6" "10" "14" "18"
[3,] "Kelly"  "3" "7" "11" "15" "19"
[4,] "Sean"   "4" "8" "12" "16" "20"

With Data Frames...

> data.frame(patients, my_matrix)
  patients X1 X2 X3 X4 X5
1     Bill  1  5  9 13 17
2     Gina  2  6 10 14 18
3    Kelly  3  7 11 15 19
4     Sean  4  8 12 16 20
```

- `rownames()` works the same way. Columns keep their own types.

- Useful relatives: `rbind()` (stack rows), `nrow()`, `ncol()`, `t()` (transpose), `as.data.frame()`.

---

## 8. Logic

### Comparison
```r
TRUE == TRUE         # TRUE
(FALSE == TRUE) == FALSE   # TRUE
6 == 7               # FALSE
6 < 7                # TRUE
10 <= 10             # TRUE
5 != 7               # TRUE
```
Note `TRUE > FALSE` is `TRUE`, since they coerce to 1 and 0.

### NOT, AND, OR

- `&` and `&&`: If the right and left operands of AND are both TRUE the entire expression is TRUE, otherwise it is FALSE. 
- `|` and `||` The OR operator evaluates to TRUE if either operand is TRUE (at least 1 side). 

**ALL `AND` operaters evaluated before `OR` operators.**

```r
!(5 == 7)                    # TRUE
FALSE & FALSE                # FALSE
TRUE & c(TRUE, FALSE, FALSE) # TRUE FALSE FALSE  — vectorized, recycles the TRUE
TRUE && c(TRUE, FALSE, FALSE)# TRUE — evaluates only the FIRST element
TRUE | c(TRUE, FALSE, FALSE) # TRUE TRUE TRUE
TRUE || c(TRUE, FALSE, FALSE)# TRUE — first element only
 5 > 8 || 6 != 8 && 4 > 3.9  # TRUE — simplified: FALSE || TRUE && TRUE --> FALSE || TRUE --> TRUE
```

**Single vs double is the key distinction:**

| | Behavior | Use for |
|---|---|---|
| `&`  and `\|` | vectorized, elementwise | filtering vectors, subsetting |
| `&&` and `\|\|` | first element only, short-circuits | `if()` conditions, control flow |

Short-circuiting means `&&` stops as soon as it hits `FALSE` and `||` stops at the first `TRUE` — the rest is never evaluated. Handy for guards like `if (!is.null(x) && x > 0)`.

### Operator precedence
**AND is evaluated before OR.**
```r
5 > 8 || 6 != 8 && 4 > 3.9   # TRUE
```
Reads as `5 > 8 || (6 != 8 && 4 > 3.9)`. Use parentheses rather than trusting your memory.

### Logic helper functions
```r
isTRUE(6 > 4)                # TRUE — is the argument exactly TRUE?
identical('hello', 'Hello')  # FALSE — it is case-sensitive
xor(5 == 6, !FALSE)          # TRUE — exclusive OR: exactly one must be TRUE

ints <- sample(10)           # random permutation of 1:10
ints > 5                     # produces a vector of TRUEs and FALSEs 

which(ints > 7)              # takes logical vector and returns the INDICES where the condition is TRUE 
any(ints < 0)                # FALSE — is at least one TRUE?
all(ints > 0)                # TRUE  — are they all TRUE?
```
`which.min()` and `which.max()` give the index of the smallest/largest element.

Remember: `which()` returns **positions**, whereas `ints[ints > 7]` returns **values**.

---

## 9. Functions

**Functions** are the heart of R. Crucially, **functions are objects** — you can pass them around like any other value.

They are small pieces of reusable code that can be treated like any other R object. 
- Denoted as `function_name()`
- Arguments are put in the parentheses, separated by commas (they can be named or unnamed). They are variables for the function to operate on.
- Typing a function's name without parentheses prints its source code — a good way to learn from base R.

**Objects** — self-contained units that bundles data and behavior together (State/characteristics and behavior/actions, properties and methods). It acts as a digital package used to represent specific things like a user account, a button on a website, or a character in a game. 



### Anatomy
```r
function_name <- function(arg1, arg2) {    #assignment beginning
  # body
  # the LAST expression evaluated is returned automatically
}
```

```r
boring_function <- function(x) x        # one-liners need no braces
my_mean <- function(my_vector) sum(my_vector) / length(my_vector)
mean()   # default R function that does the same as the line above! 
```
`return(x)` works but is only idiomatic for early exits. 

### Default arguments
Used to set default values. 
```r
remainder <- function(num, divisor = 2) {
  num %% divisor
}
remainder(5)                    # 1  — divisor defaults to 2
remainder(11, 5)                # 1  — positional matching
remainder(divisor = 11, num = 5)# 5  — named matching, order irrelevant
remainder(4, div = 2)           # partial matching of argument names works
```
Matching order: exact name → partial name → position. Rely on names for anything past the second argument.

```r
args(remainder)                 # prints a function's arguments: function (num, divisor = 2) NULL
```

### Functions as arguments
```r
evaluate <- function(func, dat) {
  func(dat)
}
evaluate(sum, c(2, 4, 6))          # 12
evaluate(median, c(7, 40, 9))      # 9
```

### Anonymous functions
Defined inline, never named:
```r
evaluate(function(x) x + 1, 6)          # 7
evaluate(function(x) x[1], c(8, 4, 0))  # 8   — first element
evaluate(function(x) x[length(x)], c(8, 4, 0))  # 0  — last element
```
This is the pattern that makes `lapply`/`sapply` powerful in the next lesson.

### The ellipsis `...`
`...` captures any number of unspecified arguments and passes them on.
All arguments after an ellipses must have default values

```r
paste("Programming", "is", "fun!")      # paste uses ... itself

telegram <- function(...) {
  paste("START", ..., "STOP")
}
telegram("Hello", "there")              # prints out "START Hello there STOP"
```

Unpacking `...` into *named* pieces:
```r
mad_libs <- function(...) {
  args <- list(...)                     # capture the ellipsis into a list
  place <- args[["place"]]              # stores the named argument "place" from the list into a variable
  adjective <- args[["adjective"]]
  noun <- args[["noun"]]
  paste("News from", place, "today where", adjective, "students took to the streets in protest of the new", noun, "being installed on campus.")
}
mad_libs(place = "Vancouver", adjective = "unruly", noun = "cafeteria")
```

**Rule:** any argument that appears *after* `...` in the signature can only ever be supplied by full name. So put `...` last unless you deliberately want that.

### Custom binary operators
**Binary Operators** — things that take 2 inputs (left and right) and make 1 output. e.g. the +, -, *, and / symbols

Any function named `%something%` becomes an infix operator. The name must be quoted at definition.
```r
"%p%" <- function(left, right) {
  paste(left, right)
}
"I" %p% "love" %p% "R"          # "I love R"
```
Built-in examples: `%%` (modulo), `%/%` (integer division), `%in%` (membership), `%o%` (outer product).

```r
7 %% 4      # 3
7 %/% 4     # 1
3 %in% c(1, 2, 3)   # TRUE
```

---

## 10. lapply and sapply

The **`*apply` family replaces loops.** You give it a list/data frame and a function; it applies the function to each element.
It offers a concise and convenient means of implementing the Split-Apply-Combine strategy for data analysis.  See Hadley Wickham's Journal of Statistical Software paper titled 'The Split-Apply-Combine Strategy for Data Analysis' for more on this strategy.


`lapply(X, FUN, ...)` with arguments being dataset, function_name

Dataset used: `flags` — 194 rows (countries), 30 columns of flag attributes from the UCI Machine Learning Repository
http://archive.ics.uci.edu/ml/datasets/Flags

```r
dim(flags)          # 194  30 (in rows, columns)
head(flags)         # shows the first 6 lines 
str(flags)
?flags              # the help page documents every column
```

### `lapply` — always takes and returns a **L**ist
A data frame *is* a list of columns, so `lapply` iterates over columns:
```r
cls_list <- lapply(flags, class)   # applies the class() function to each column of flags and makes a list of 30 one-element character vectors
class(cls_list)                    # "list"
as.character(cls_list)             # manually flattens list to a character vector since every element of the list returned by lapply() is a character vector of length one
```

### `sapply` — **s**implifies the result when it can
```r
cls_vect <- sapply(flags, class)   # named character vector, not a list
class(cls_vect)                    # "character"
```

**Simplification rules:**

| Each call returns... | `sapply` gives you |
|---|---|
| a length-1 value | a vector |
| vectors of equal length > 1 | a matrix (one column per element) |
| results of varying length | a list (no simplification possible) |


### Worked examples
```r
sum(flags$orange)                  # because columns 11-17 are indicator variables (1 for yes, 0 for no) we can add up the 1s and 0s in the 'orange' column to see how many flags contain orange.  
flag_colors <- flags[, 11:17]      # extracts columns 11-17. Syntax: [rows (none), columns] red green blue gold white black orange (all 0/1)
sapply(flag_colors, sum)           # how many flags contain each colour
sapply(flag_colors, mean)          # PROPORTION of flags with each colour
```
This is the `sum`/`mean`-on-a-0/1-column idiom again — `mean()` of an indicator column is a proportion.

```r
flag_shapes <- flags[, 19:23]      # circles crosses saltires quarters sunstars
                                          # the range function below shows returns the maximum and minimum of first argument
shape_mat <- sapply(flag_shapes, range)   # 2x5 MATRIX: min in row 1, max in row 2
class(shape_mat)                          # "matrix" "array"
```
Each call to `range()` returns 2 numbers, all the same length → matrix.

```r
unique_vals <- lapply(flags, unique)      # stores all the unique values per column
sapply(unique_vals, length)               # how many distinct values per column

sapply(flags, unique)                     # returns a LIST — lengths differ, can't simplify
```

### With anonymous functions
```r
lapply(unique_vals, function(elem) elem[2])   # 2nd unique value from each column
# Note that our function takes one argument, elem, which is just a 'dummy variable' that takes on the value of each element of unique_vals, in turn.

```

Relatives to know: `mapply()` (multiple lists in parallel), `apply()` (over matrix rows/columns via `MARGIN`), `Map()`, `Reduce()`, `Filter()`.

---

## 11. vapply and tapply

### `vapply` — `sapply` with a safety net
You declare the expected return type and length up front.  If the result doesn't match the format you specify, vapply() will throw an error immediately, causing the operation to stop  rather than a silently wrong object

Fun fact: vapply() may perform faster than sapply() for large datasets.

```r
vapply(flags, class, character(1))      # "each call must return 1 character value"
vapply(flags, unique, numeric(1))       # ERROR — values must be length 1
sapply(unique_vals, length)             # works
vapply(unique_vals, length, numeric(1)) # same result, but type-guaranteed
```

Common templates: `character(1)`, `numeric(1)`, `logical(1)`, `integer(1)`, `numeric(2)`.

**Why it matters:** `sapply` guesses. If your data changes shape, `sapply` can quietly hand back a list where your code expected a vector, and the bug surfaces far downstream. `vapply` fails loudly and at the right place — prefer it in scripts and functions you'll rerun.

### `table()` — counting
`table(...)`: This function takes all the data in that column, finds every unique value, and counts exactly how many times each value appears. However, it can do all manner of other things too (2-way tables, counts to percentages, etc). Table() does ignore NAs, or missing data though. Use the `useNA = "ifany"` argument to bypass that. 


```r
table(flags$landmass)   # frequency of each of the 6 landmass codes
table(flags$animate)    # how many flags depict an animate image (0/1)
mean(flags$animate)     # ~0.4 — proportion, since the column is 0/1
```

`table(flags$landmass)`: from the dataset `flags` extract (with the 1-column extraction operator `$`) the column `landmass`, then table finds every unique value, and counts exactly how many times each value appears. 



### `tapply` — split, then apply
Think of it as "Table Apply". It takes the data in argument one, splits them into piles based on the value of some variable (argument 2), and applies a function to each individually. 

`tapply(X, INDEX, FUN)` splits vector `X` into groups defined by the factor `INDEX`, then applies `FUN` to each group. This is grouped summary statistics in one line.

```r
tapply(flags$animate, flags$landmass, mean)
# proportion of flags with an animate image, WITHIN each landmass group. Mean function used to calculate proportions!

tapply(flags$population, flags$red, summary)
# five-number summary of population, split by whether the flag contains red
# minimum, 1st quartile, median, mean, 3rd quartile, max. 

tapply(flags$population, flags$landmass, summary)
# population summary per landmass
```
Mnemonic: `t` for **t**able/split, so read it as "for each group of INDEX, do FUN to X".

---

## 12. Looking at Data

The discipline of this lesson: **before analyzing anything, characterize it.** Run this checklist on every new dataset.

- What is the format of the data? 
- What are the dimensions? 
- What are the variable names? 
- How are the variables stored? 
- Are there missing data? 
- Are there any flaws in the data?

Dataset used: `plants` — 5166 rows, 10 columns.
United States Department of Agriculture's PLANTS Database (http://plants.usda.gov/adv_search.html).

```r
ls()                     # what's in my workspace? List the variables
class(plants)            # What format/class? "data.frame"
dim(plants)              # What are the dimensions? 5166  10   (rows, columns)
nrow(plants)             # 5166
ncol(plants)             # 10
object.size(plants)      # memory footprint? 745944 bytes
names(plants)            # a character vector of column (i.e. variable) names
```

### Peeking at rows
```r
head(plants)             # first 6 rows (default)
head(plants, 10)         # first 10 rows
tail(plants, 15)         # last 15 — worth checking, junk often hides at the bottom
```

### Summarizing
```r
summary(plants)
```
`summary()` is class-aware: 

- For **numeric** columns it gives min, 1st quartile, median, mean, 3rd quartile, max, and an `NA's` count

- For **categorical** (factor/character) columns it acts like the table() function! It simply counts up how many times each word appears and gives you a tally (e.g., 3,031 Perennials and 682 Annuals), lumping excess categories (it stops at 7, unlike Table, which would never lump excess categories) into an (Other) group. Factors > characters because of custom sorting and a master list of allowed categories, known as Levels. 

- For **character columns** (plain text), it provides structural stats like the number of rows (Length) and character counts (Min.nchar, Max.nchar).

```r
table(plants$Active_Growth_Period)   # exact counts for one categorical column
```



```r
str(plants)                          # the single most useful function here
```

`str()` = **str**ucture. It combines class, dimensions, column names, column types, and a preview of the values. It works on *any* object, not just data frames — try `str(lm)` or `str(1:10)`.

**Recommended order for a fresh dataset:** `class` → `dim` → `names` → `head`/`tail` → `str` → `summary` → `table` on the categoricals.

---

## 13. Simulation

### `sample()` — the workhorse

`sample(x, size, replace = FALSE, prob = NULL)`
where x is 	either a vector of 1+ elements from which to choose, or a positive integer

```r
sample(1:6, 4, replace = TRUE)    # four rolls of a die (WITH replacement, meaning it can show up more than once)
sample(1:20, 10)                  # 10 distinct numbers from 1 to 20 (WITHOUT replacement, the default)
sample(LETTERS)                   # random permutation of all 26; sample n=26 w/o replacement
```
`LETTERS` is a built-in constant (also `letters`, `month.name`, `pi`).

### Unequal probabilities
```r
flips <- sample(c(0, 1), 100, replace = TRUE, prob = c(0.3, 0.7))
sum(flips)     # roughly 70 — a biased coin flipped 100 times (adding up all the heads, if heads = 1)
``` 

### Random draws from named distributions
The naming convention is a **prefix + distribution**:

| Prefix | Gives you | Example |
|---|---|---|
| `r` | **r**andom draws | `rnorm(10)` |
| `d` | **d**ensity / PMF | `dnorm(0)` |
| `p` | cumulative **p**robability | `pnorm(1.96)` |
| `q` | **q**uantile (inverse of `p`) | `qnorm(0.975)` |

| Distribution | Suffix | Notes |
|---|---|---|
| Normal | `norm` |  |
| Binomial | `binom` | A binomial random variable represents the number of 'successes' in a given number of independent 'trials'  |
| Poisson | `pois` | A poisson random variable counts the number of independent events that happen during a fixed block of time or space |
| Uniform | `unif` | flat rectangular graph |
| Exponential | `exp` |  |
| Chi-squared | `chisq` | the sum of the squared values of independent standard normal random variables |
| Gamma | `gamma` |  |

```r
rbinom(1, size = 100, prob = 0.7)     # ONE number: total successes in 100 trials 
rbinom(100, size = 1, prob = 0.7)     # 100 numbers: the individual trials

# both achieve the same effect as the 2 following commands: 
# sample(c(0, 1), 100, replace = TRUE, prob = c(0.3, 0.7)) 
# sum(flips)

```
Same underlying process, different granularity — a binomial is a sum of Bernoullis.

```r
rnorm(10)                   # 10 numbers from a standard normal: mean 0, sd 1 (the defaults) 
rnorm(10, 100, 25)          # mean 100, sd 25
rpois(5, 10)                # 5 Poisson draws with lambda = 10 (mean = 10, the average number of times an event happens in a block of time/space)
```

### `replicate()` — repeat an expression many times
```r
my_pois <- replicate(100, rpois(5, 10))   # 5x100 (rxc) matrix: 100 independent samples of size 5
cm <- colMeans(my_pois)                   # the sample mean of each column
hist(cm)                                  # approximately normal — the CLT, demonstrated
```
That three-line block is the standard shape of a Monte Carlo simulation: generate → summarize → visualize.

Also useful: `rowMeans()`, `colSums()`, `rowSums()`.

### Reproducibility
```r
set.seed(42)
rnorm(5)        # same five numbers on any machine, every time
```
Always `set.seed()` in any analysis or assignment involving randomness.

---

## 14. Dates and Times

R stores both as numbers under the hood, counting from the **Unix epoch, 1970-01-01**.

| Class | Stores | Counts |
|---|---|---|
| `Date` | dates only | days since 1970-01-01 |
| `POSIXct` | date-times as one number | seconds since 1970-01-01 |
| `POSIXlt` | date-times as a **list** of components | — |

`unclass()` strips the class attribute and reveals the raw number — the single most illuminating command in this lesson.

### Dates
```r
d1 <- Sys.Date()          # today, in YYYY/MM/DD
class(d1)                 # "Date"
unclass(d1)               # e.g. 20675 — days since the epoch (1970-01-01!)

d2 <- as.Date("1969-01-01") # What happens about dates before the epoch? 
unclass(d2)               # -365 — dates before 1970 are negative
```

### Times
```r
t1 <- Sys.time()          # contains "2026-08-15 11:10:32 PDT", class POSIXct
class(t1)                 # "POSIXct" "POSIXt" (output)
unclass(t1)               # 1786817432 — a large number of seconds

t2 <- as.POSIXlt(Sys.time()) # coercion of Sys.time() from class POSIXct to POSIXlt
class(t2)                 # "POSIXlt" "POSIXt" (output)
unclass(t2)               # a LIST of components, like all POSIXlt objects
str(unclass(t2))          # compact view
t2$min                    # extract just the minute
```
Other `POSIXlt` fields: `sec`, `hour`, `mday`, `mon` (0–11), `year` (years since 1900), `wday` (0 = Sunday), `yday`, `isdst`.

Use `POSIXct` for storage and arithmetic; `POSIXlt` when you need to pull out components.

### Extracting human-readable parts
```r
weekdays(d1)     # "Monday"
months(t1)       # "August"
quarters(t2)     # "Q3"
```
These work on any of the three classes (any date or time object).

### Parsing messy strings with `strptime()`
Converts character vectors to POSIXlt.

```r
t3 <- "October 17, 1986 08:24"            # a character string
t4 <- strptime(t3, "%B %d, %Y %H:%M")
t4                                        # "1986-10-17 08:24:00 PDT"
class(t4)                                 # "POSIXlt" "POSIXt"
```
`strptime()` needs a **format string** because it can't guess:

| Code | Means |
|---|---|
| `%Y` | 4-digit year |
| `%y` | 2-digit year |
| `%m` | month as number |
| `%B` | full month name |
| `%b` | abbreviated month |
| `%d` | day of month |
| `%H` `%M` `%S` | hour, minute, second |
| `%A` | full weekday name |
| `%j` | day of year |

`as.Date()` accepts a `format` argument too. Note `Sys.Date()`/`as.Date()` do NOT need a format for the ISO `"YYYY-MM-DD"` layout.

### Arithmetic
```r
Sys.time() > t1                              # TRUE — comparisons work
Sys.time() - t1                              # a difftime object
difftime(Sys.time(), t1, units = "days")     # Function to choose your own units for time difference. Prints:  Time difference of 0.01098802 days
```
Units available: `"secs"`, `"mins"`, `"hours"`, `"days"`, `"weeks"`.

The lesson closes by recommending the **`lubridate`** package (`ymd()`, `mdy()`, `hms()`, `year()`, `month()`) by Hadley Wickham for anything nontrivial.

---

## 15. Base Graphics

Base graphics = quick exploratory plots. Verbose for publication work, but instant.
lattice, ggplot2 and ggvis are not covered. 

```r
data(cars)      # load the built-in dataset
?cars
head(cars)      # two columns: speed, dist
```

### `plot()` — scatterplots
`plot()` is generic: it dispatches on the class of its argument.
```r
plot(cars)                              # with a 2-column data frame it plots col1 vs col2
plot(x = cars$speed, y = cars$dist)     # identical, but explicit, except arguments ($ signs included) become the axes for the graph
plot(x = cars$dist, y = cars$speed)     # axes swapped
```
`plot(x, y)` puts `x` on the horizontal axis. Convention: **x = predictor, y = outcome.**

- First, R notes that the data frame you have given it has just two columns, so it assumes that you want to plot one column versus the other.
- Second, since we do not provide labels for either axis, R uses the names of the columns. 
- Third, it creates axis tick marks at nice round numbers and labels them accordingly. 
- Fourth, it uses the other defaults supplied in plot().


### Common arguments
```r
plot(x = cars$speed, y = cars$dist, xlab = "Speed", ylab = "Stopping Distance") # labels for axes 
plot(cars, main = "My Plot")            # main title (top)
plot(cars, sub = "My Plot Subtitle")    # subtitle (bottom)
plot(cars, col = 2)                     # colour: 2 = red (or use "red")
plot(cars, xlim = c(10, 15))            # restrict the x-axis range
plot(cars, pch = 2)                     # plotting character: 2 = open triangles
```

| Argument | Controls |
|---|---|
| `xlab` `ylab` | axis labels |
| `main` `sub` | title, subtitle |
| `col` | colour (number or name) |
| `pch` | point symbol (0–25) |
| `xlim` `ylim` | axis ranges, as `c(min, max)` |
| `type` | `"p"` points, `"l"` lines, `"b"` both |
| `cex` | symbol size |
| `lty` `lwd` | line type, line width |

```r
?par            # the full catalogue of graphical parameters — the reference to bookmark
```

### Other plot types
```r
data(mtcars)                                  # loads the data frame
boxplot(formula = mpg ~ cyl, data = mtcars)   # mpg distribution BY cylinder count (mpg on y-axis)
hist(mtcars$mpg)                              # histogram of a single variable. 
```

**The formula syntax `y ~ x`** reads as "y as a function of x" (or "y by x"). It reappears everywhere in R — `lm()`, `t.test()`, `aov()`, `aggregate()` — so it's worth recognizing now.

Also in base: `barplot()`, `points()`, `lines()`, `abline()`, `text()`, `legend()`, `pairs()`.
Histograms `hist()` and `plot()` both work best with single vectors. 

The lesson ends by pointing to **`ggplot2`** (and `lattice`) for serious graphics.

If you want to explore other elements of base graphics, then this web page (http://www.ling.upenn.edu/~joseff/rstudy/week4.html) provides a useful overview.

---

# Appendix A: The gotchas the course is really teaching

1. **Recycling is silent** when lengths divide evenly. `c(1,2,3,4) + c(0,10)` gives no warning.
2. **`NA` propagates through everything**, including comparisons. `NA == NA` is `NA`. Use `is.na()`.
3. **Subsetting with a condition leaks `NA`s**: `x[x > 0]` keeps them. Write `x[!is.na(x) & x > 0]`.
4. **R is 1-indexed.** `x[0]` returns an empty vector, `x[3000]` returns `NA` — neither errors.
5. **`&` vs `&&`.** Vectorized filtering vs single-value control flow. Same for `|` vs `||`.
6. **AND binds tighter than OR.** Parenthesize.
7. **Implicit coercion in matrices/`cbind`.** Mixing types silently makes everything character. Use a data frame.
8. **`sapply` guesses; `vapply` verifies.** Use `vapply` in code you'll rerun.
9. **`times` vs `each`** in `rep()`, and **`sep` vs `collapse`** in `paste()`.
10. **`seq_along(x)` over `1:length(x)`** — the latter runs backwards on empty input.
11. **`set.seed()`** before anything random.
12. **`which()` returns positions; logical subsetting returns values.**
13. **R is case-sensitive.** `TRUE` is not `True`; `b` is not `B`.
14. `=` is for arguments, `<-` is for assignment, `==` is for comparison.

---

# Appendix B: Function index by lesson

| Lesson | Functions introduced |
|---|---|
| 1 Basic Building Blocks | `c` `sqrt` `abs` `length` `?` `help.start` |
| 2 Workspace and Files | `getwd` `setwd` `ls` `list.files` `dir` `args` `dir.create` `file.create` `file.exists` `file.info` `file.rename` `file.copy` `file.path` `file.remove` `unlink` `rm` |
| 3 Sequences | `:` `seq` `seq_along` `seq_len` `rep` `length` |
| 4 Vectors | `c` `paste` `LETTERS` |
| 5 Missing Values | `NA` `NaN` `Inf` `is.na` `is.nan` `rnorm` `rep` `sample` `sum` `mean` |
| 6 Subsetting | `[` `is.na` `names` `identical` |
| 7 Matrices/Data Frames | `dim` `attributes` `class` `matrix` `cbind` `rbind` `data.frame` `colnames` `rownames` `nrow` `ncol` `t` |
| 8 Logic | `==` `!=` `!` `&` `&&` `\|` `\|\|` `isTRUE` `xor` `which` `any` `all` `sample` |
| 9 Functions | `function` `return` `args` `paste` `...` `list` `%%` `%/%` `%in%` |
| 10 lapply/sapply | `lapply` `sapply` `head` `str` `dim` `unique` `range` `sum` `mean` `as.character` |
| 11 vapply/tapply | `vapply` `tapply` `table` `summary` `mean` |
| 12 Looking at Data | `ls` `class` `dim` `nrow` `ncol` `object.size` `names` `head` `tail` `summary` `table` `str` |
| 13 Simulation | `sample` `rbinom` `rnorm` `rpois` `replicate` `colMeans` `hist` `set.seed` `LETTERS` |
| 14 Dates and Times | `Sys.Date` `Sys.time` `as.Date` `as.POSIXct` `as.POSIXlt` `unclass` `weekdays` `months` `quarters` `strptime` `difftime` |
| 15 Base Graphics | `plot` `boxplot` `hist` `par` `data` `points` `lines` `abline` `legend` |

---

# Appendix C: Twenty-question self-test

Answer without running R, then check.

1. What does `c(1, 2, 3, 4) + c(0, 10)` return, and does it warn?
2. Difference between `rep(c(0,1), times = 3)` and `rep(c(0,1), each = 3)`?
3. What is `seq(5, 10, length = 6)`?
4. What does `paste(c("a","b"), collapse = "-")` give? And `paste(c("a","b"), 1:2, sep = "-")`?
5. What is `NA == NA`? Why?
6. How do you count the `NA`s in a vector `x`?
7. What does `x[x > 0]` return if `x` contains `NA`s?
8. What does `x[0]` return? What about `x[9999]`?
9. Difference between `x[-c(1,2)]` and `x[c(-1,-2)]`?
10. Why does `cbind(c("a","b"), 1:2)` produce quoted numbers?
11. What is `class(matrix(1:6, nrow = 2))`?
12. Evaluate `TRUE || stop("boom")`. Why doesn't it error?
13. How does `5 > 8 || 6 != 8 && 4 > 3.9` group?
14. Difference between `which(x > 5)` and `x[x > 5]`?
15. In `function(a, ..., b)`, how must `b` be supplied?
16. When does `sapply` return a matrix rather than a vector?
17. Write `vapply` to get the class of every column of `df`.
18. What does `tapply(df$y, df$g, mean)` compute?
19. What does `unclass(Sys.Date())` show, and what is the reference point?
20. What does `mpg ~ cyl` mean in `boxplot()`?

Answers, condensed: (1) `1 12 3 14`, no warning. (2) `0 1 0 1 0 1` vs `0 0 0 1 1 1`. (3) `5 6 7 8 9 10`. (4) `"a-b"`; `"a-1" "b-2"`. (5) `NA` — two unknowns can't be compared. (6) `sum(is.na(x))`. (7) The positives plus one `NA` per missing value. (8) `numeric(0)`; `NA`. (9) Identical. (10) A matrix holds one class, so numbers coerce to character. (11) `"matrix" "array"`. (12) `TRUE` — `||` short-circuits and never evaluates the right side. (13) `5 > 8 || (6 != 8 && 4 > 3.9)`; AND first. (14) Indices vs values. (15) By full name only. (16) When every call returns a vector of the same length > 1. (17) `vapply(df, class, character(1))`. (18) The mean of `y` within each group of `g`. (19) Days since 1970-01-01. (20) "mpg by cyl" — distribution of mpg split by cylinder count.
