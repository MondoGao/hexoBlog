---
title: A introduction to R reading notes
date: 2018-06-27
tags:
- R
- R lang
- notes
---
## 1 Introduction and preliminaries

- Exit repl: `q()`

### Help

- `help(solve)` || `?solve` || `help("[[")`
- Lanuch help: `help.start()`
- `help.search()` || `??solve``
- `example(topic)`

### Basic commands

- Elementary commands: **expressions** & **assignments**
- Commands are separated by `;` or newline. Commands can be grouped by `{ }`. Comments start with `#`.
- Executing file: `source("<file>")`
- output to file: `sink("<file>")`, restore: `sink()`

### Data permanency

The entities that R creates and manipulates are **objects**.

Use `objects()` or `ls()` to list. The collection of objects currently stored is called the **workspace**.

Use `rm()` to remove.

Objects are stored in `.RData`, historys are stored in `.Rhistory`.

## 2 Simple manipulations; numbers and vectors

### Vectors and assignment

**vector**: a ordered array of numbers.

```R
# an assignment useing the function c()
x <- c(1, 2, 3, 4)
c(1, 2, 3, 4) -> x
assign("x", c(1, 2, 3, 4))
```

In most contexts the `=` operator can be used as an alternative to `<-`.

If an expression is used as a complete command, the value is preinted and **lost**.

### Vector arithmetic

Shorter vectors are **recycled** when occurring in the same expression. In particular a constant is simply repeated.

- operators: `+`, `-`, `*`, `/`, `^`
- functions
  - `log`, `exp`, `sin`, `cos`, `tan`, `sqrt`
  - `max` & `min`

    select in all their arguments, use `pmax` to select parallel.

  - `length`
  - `sum` & `prod`
  - `mean`, `var`
  - `sort`, `order`, `sort.list`

To work with complex numbers, supply an explicit complex part: `sqrt(-17+0i)`

### Generating regular sequences

```R
1:10
c(1,2,...10)
seq(1,10)
seq(from=1, to=10, by=1, length=10, along)
```

Arguments can be given in named form and mixing with normal form

```R
# replicate an object
rep(1, times=5)
```

### Logical vectors

Values: `TRUE`, `FALSE`, `NA` (not available)

Operators: `<`, `<=`, `>`, `>=`, `==`, `!=`

Logical operators: `&` (and), `|` (or)

```R
temp <- x > 13
```

In ordinary arithmetic, `TRUE` are coerced `1`, `FALSE` becomes `0`.

### Missing values

- `is.na(x)`: compare elements with `NA` & `NaN` and return a new logical vector

`x == NA` is different from `is.na(x)`, former one will compare the `x` itself with `NA`.

`NaN` (not a number) values:
```R
0/0 # NaN
Inf - Inf # NaN
```
### Character vectors

Character strings are entered using `"` or `'`. Escaping uses `\`. 

- `?Quotes`

  ```R
  # concatenates arguments one by one into character strings
  paste(c("x", "y"), 1:6, sep=" ") #> c("x1", "y2", "x3",...)
  # paster(..., collapses=<str>) join arguments into a single string.
  ```

### Index vectors; selecting and modifying subsets of a data set

Index can be:

1. Logical vector: work like a filter

```R
x <- c(1:3, NA)
x[!is.na(x) & x>0] #> c(1,2,3)
```

2. A vector of positive integral quantities
```R
x[1:10]
```

3. A vector of negative integral quantities: specifies the values to be **excluded**

```R
x[-(1:5)] # gives all but the first five
```

4. A vector of character strings: applied where an object has a **names** attribute:
```R
fruit <- c(5, 10, 1)
names(fruit) <- c("orange", "banana", "apple")
fruit[c("apple", "orange")]
```

An indexed expression can also appear on the rhs of a vector, when the assignment is performed only on those eles of the vector.

```R
# same
y[y < 0] <- -y[y < 0]
y <- abs(y)
```

### Other types of objects

- **matrices**
- **factors**
- **lists**: a vector whose eles needn't be of the same type
- **data frames**: matrix-like
- **functions**

## 3 Objects, their modes and attributes

- Intrinsic attributes: mode and length

Type, or **mode**, namely *numeric*, *complex*, *logical*, *caharacter* and *raw*.

Use `mode(object)`, `length(object)`, `attributes(object)` to examine.

- Changing the length of an object

```R
e <- numeric()
e[3] <- 17 # length to 3
# or
length(e) <- 3
```

- Getting and setting attributes

```R
# attr(object, name)
attr(z, "dim") <- c(10, 10)
```
- The class of an object
 
  All objects in R have a *class*, for simple vectors this is *mode*, but "matrix, " array" are other possible values.

  - `class()`
  - `unclass()`
 
## 4 Ordered and unordered factors

A factor of a vector is a group of unique elements.

- Use `factor(<vector>)` to create a factor.
- A *factor* has a *class* attr containing all unique eles.
- Use `tapply(<values>, <levels>, <func>)` to apply function to each levels.
  
  `tapply` can be used to handle complicated indexing of a vector by multiple categories.

```R
state <- c("tas", "sa", "qld", "nsw", "nsw", "nt", "wa", "wa", "qld", "vic", "nsw", "vic", "qld", "qld", "sa", "tas", "sa", "nt", "wa", "vic", "qld", "nsw", "nsw", "wa", "sa", "act", "nsw", "vic", "vic", "act")
statef <- factor(state)
incomes <- c(60, 49, 40, 61, 64, 60, 59, 54, 62, 69, 70, 42, 56, 61, 61, 61, 58, 51, 48, 65, 49, 49, 41, 48, 52, 46, 59, 46, 58, 43)
tapply(incomes, statef, mean)
```

The `ordered()` creates ordered factors but is otherwise identical to `factor`.

## 5 Arrays and matrices

A vector can be an array only if it has a dimension vector as its *dim* attr:

- *dimension vector*: a vector of non-negative integers.

```R
dim(z) <- c(3,5,100)
z[3,5,99]
z[,5,99] #> c(z[1,5,99], z[2,5,99], z[3,5,99])
```

Matrices are two dimension arrays.

If any index position is given an empty index vector, then the full range of that subscript is taken:

```R
a[2,,]
```

### Index matrices

Use a matrice in index to extract an irregular collection as a vector.

```R
x <- array(1:20, dim=c(4,5))
i <- array(c(1:3, 3:1), dim=c(3,2))
x[i] #> 9,6,3
```

- Negative indeices are not allowd in index matrices.
- Rows containing `NA` produce `NA`.
- Rows containing a zero is ignored.
- `cbind`
- `table`

### `array()`

```R
array(<dataVector>, <dimVector>)
```

### Mixed vector and array arithmetic. The recycling rule

- The expression is scanned from left to right.
- Any short vector operands are extended by cetycling their values until they match the size of any other operands.
- As long as short vectors and arrays only are encountered, the arrays must all have the same `dim` attribute or an error results.
- Any vector operand longer than a matrix or array operand generates an error.
- If array structures are present and no error or coercion to vector has been precipitated, the result is an array structure with the common `dim` attribute of its array operands.

### The outer product of two arrays

The result's dimentsion of a outer product is obatined by concatenating their two dimension, and whose data vector is got by forming all possible products of a with those of b.

```R
ab <- a %o% b
# Do function(product) to all a, b's combination.
ab <- outer(a,b,"*")
```

### Generalied transpose of an array

- `aperm(a, perm)`: Switch dimension in a. Used to permute an array.
- `t(a)` & `aperm(a, c(2,1))`: transpose an array.

### Matrix facilities

- `t(X)` is hte matrix transpose function
- `nrow(A)` & `ncol(A)`

#### Multiplication

```R
A * B #> matrix of product of each slot
A %*% B #> matrix product
```

- `crossprod(X, y)` is the same as `t(X) %*% y`
- `diag(v)`
  - When v is a vector, gives a diagonal matrix.
  - When v is a matrix, gives the vector of diagonal entries of v.
  - When v is a single value, gives the identity matrix.

#### Linear equations and inversion

```R
b <- A %*% x
solve(A,b) #> x #> A^-1 b
solve(A) #> A^-1
```

#### Eigenvalues and eigenvectors

```R
# calculates the eigenvalues and eigenvectors of a symmetric matrix
ev <- eigen(Sm)
ev$vec
ev$val
```

#### Singular value decomposition and determinants

- `svd(M)`: calculates the singular value decomposition of M
- `prod()`
- `absdet()`
- `det()`

#### Least squares fitting and the QR decomposition

- `lsfit()`: returns a list giving results of a least squares fitting procedure.
- `ls.diag()`
- `lm(.)`
- `qr()`

### Forming partitioned matrices

- `cbind()`: form matrices by binding matrices horizontally.
- `rbind()`

They respect `dim` attribute.

They are the simplest ways explicitly to allow the vector to be treated as a column or row matrix.

### The concatenatoin function, `c()`, with arrays

```R
# Corece an array back to a simple vector
as.vector()
c(<array>)
```

### Frequency tables from factors

`table` allows frequency tables to be calculated from equal length factors.

Equivalent to `tapply(statef, statef, length)`.

```R
incomef <- factor(cut(incomes, breaks = 35+10*(0:7))
table(incomef, statef)
```

## 6 Lists and data frames

### Lists

*list* is an object consisting of an ordered collection of objects knows as its *components*.

*components* can be diffirent types and are always *numbered*.

```R
Lst <- list(name="Mondo", bf="ziyang", no.children=0, child.ages=c(101))

List[[1]]
# same as
List$name
List[["name"]]

# name can be a variable
x <- "name"
List[[x]]

# names may be abbreviated
List$na

# modify
List[5] <- list(matrix=Mat)
List$name <- value
```

`[[...]]` is used to select a single element, `[...]` selects a *sublist* of the *list*, if list is a named list, the names are transferred to the sublit.

### Data frames

A *data frame* is a list with class `data.frame`, may for many purposes be regarded as a matrix with columns possibly of differing modes and attributes.

Restrictions:
- The components must be vectors, factors, numeric matrices, lists, or other data frames.
- Matrices, lists, and data frames provide as many variables to the new data frame as they have columns, elements, or variables.
- Numeric vectors, logicals and factors are included as is, and by default character vectors are coerced to be factors.
- Vector structures appearing as variables of the data frame must all have the same length, and matrix structures must all have the same row size.

```R
# create
accountants <- data.frame(home=statf, loot=incomes, shot=incomef)
read.table() # read an entire from an external file
as.data.frame() # coerce into a data frame

# attach an data frame to search path, but remains unchanged
attach()
detach()
```

Data frames can use as a workspace.

`attach` allows not only directories and data frames to be attached, but other classes of object as well.

### Managing the search path

- `search()` shows the current search path.
- `ls(<search_level>)` examine the contents of any position on the search path.

## 7 Reading data from files

### `read.table()`

The external file will normally have a special form:
- The first line of the file should have a *name* for each variable.
- Each additional line has  as its first item a *row label* and the values for each variable.

- `read.table(header=TRUE)`: specifies that the first line is a line of headings.

### `scan`

```R
# The second arg is a dummy list that establishes the mode of the three vectors of data to be read.
scan("input.dat", list(id="",0,0))
```

### Accessing builtin datasets

- `data()`: see the list of builtin datasets.
- `data(<data_name>[, package="datasets"])`: load datasets.

### Editing data

```R
xnew <- edit(xold)

fix(xold)
# same as
xold <- edit(xold)

# enter new data via sheet interface
xnew <- edit(data.frame())
```

## 8 Probability distributions

### R as a set of statistical tables

Distribution|R name|additional arguments
---|---|---
beta|`beta`|shape1, shape2, ncp
binomial|`binom`|size, prob
Cauchy|`cauchy`|location, scale
chi-squared|`chisq`|df, ncp
exponential|`exp`|rate
F|`f`|df1, df2, ncp
gamma|`gamma`|shape, scale
geometric|`geom`|prob
hypergeometric|`hyper`|m, n, k
log-normal|`lnorm`|meanlog, sdlog
logistic|`logis`|location, scale
negative binomial|`nbinom`|size, prob
normal|`norm`|mean, sd
Poisson|`pois`|lambda
signed rank|`signrank`|n
Student's t|`t`|df, ncp
uniform|`unif`|min, max
Weibull|`weibull`|shape, scale
Wilcoxon|`wilcox`|m, n

Prefix by `d` for the density, `p` for the CDF, `q` for the quantile function, `r` for simualtion.

- `ptukey` & `qtukey`
- `dmultinom` & `rmultinom`

### Examining the distribution of a set of data

`summary` & `fivenum` & `stem`

```R
# examine the numbers
attach(faithful)
summary(eruptions)
fivenum(eruptions)
```

```R
# make steam-and-leaf plot and histogram
stem(eruptions)
hist(eruptions)

# make the bins smaller, make a plot of density
hist(eruptions, seq(1.6, 5.2, 0.2), prob=TRUE)
lines(density(eruptions, bw=0.1))
rug(eruptions) # show the actual data points
```

```R
# plot the empirical cumulative distribution function
plot(ecdf(eruptions), do.points=FALSE, verticals=TRUE)

# eruptions of longer than 3min, fix a normal distribution
lnog <- eruptions[eruptions > 3]
plot(ecdf(long), do.points=FALSE, verticals=TRUE)
x <- seq(3, 5.4, 0.01)
lines(x, pnorm(x, mean=mean(long), sd=sqrt(var(long))), lty=3)
```

```R
# Quantile-quantile plots
par(pty="s")
qqnorm(long)
qqline(long)

# Shapiro-Wilk test
shapiro.test(long)

# Kolmogorov-Smirnov test
ks.test(long, "pnorm", mean=mean(long), sd=sqrt(var(long)))
```

### One- and two-sample tests

All "classical" tests are in package **stats** which is normally loaded.

```
A <- scan()
79.98 80.04 80.02 80.04 80.03 80.03 80.04 79.97 80.05 80.03 80.02 80.00 80.02
B <- scan()
80.02 79.94 79.98 79.97 79.97 80.03 79.95 79.97
boxplot(A, B)

t.test(A, B)

var.test(A, B)
t.test(A, B, var.equal=TRUE)

wilcox.test(A, B)

ks.test(A, B)
```

## 9 Grouping, loops and conditional execution

### Grouped expressions

Commands may be grouped together in braces, `{expr_1; ...; expr_m}`, in which case the value of the group is the result of the last expression.

### Control statement

#### Conditional execution: `if` statements
```R
# expr_1 must evaluate to single logical value
if (<expr_1>) expr_2 else expr_3
```
 
 `&&` and `||` are "short-circuit" operators, whereas `&` and `|` apply element-wise to vectors, `&&` and `||` apply to cetors of length one.

Vectorized version of `if`: `ifelse(condition, a, b)` returns a vectorof the same length as condition, with elements a[i] if conditions[i] is true.

#### Repetitive execution: `for` loops, `repeat` and `while`

- `for`

  ```R
  # `expr_1` is a vector expression
  for (name in expr_1) expr_2
  ```

  - `split()`
- `repeat expr`
- `while (condition) expr`
- `break`: terminate any loop. The only way to terminate `repeat` loopes.
- `next`

## 10 Writing your own functions

```R
# caculate two samples t test
add <- function(X, y = 0) {
  X + y
}

add(1,2)
add(X=1, y=2)
add(X=1)
add(1)

# define a new binary operator
%+% <- function(X, y) {
  add(X, y)
}
```

### `...` arg

```R
fun1 <- function(data, ...) {
  # pass settings to another function
  par(...)
  # '..n' returning the n'th unmatched argument
  par(..1)
  # evaluates all such arguments and returns them in a named list
  list(...)
}
```

### Assignments within functions

Any ordinary assignments done within the function are local and temporary and are lost after exit from the function.

- evaluation *frame*
- global assignments with superassignment operator: `<<-` or `assign`

### Scope

Thef symbold which occur in the body of the function can be divides into:
- formal parameters: occurring in the arg list, their values are determined by *binding* the actual function arguments to the formal parameters.
- local variables: whose values are determined by the evaluation of expressoins in the body of the function.
- free variables: not formal or local variables. Free variables become local variables if they are assigned to.

```R
f <- function(x) {
  y <- 2*x
  print(x) # formal parameter
  print(y) # local variable
  print(z) # free variable
}
```

The free variable bindings are resolve in which the function was created, this is called *lexical scope*.

`<<-` can change the value of free variables.

```R
open.account <- function(total) {
  list(
    deposit = function(amount) {
      total <<- total + amount
      cat(amount, "deposited.")
   }
  )
}
```

### Customizing the environment


Env profiles:
- Rprofile.site: in**etc**, or **R_PROFILE**
- .Rprofile: in work dir or home dir, or **R_PROFILE_USER**

Any function named `.First()` in either of the profiles or in the *.RData* image is automatically performed at the begginning of an session.

`.Last()` executed at the end of session.

### Classes, generic functions and object orientation

The class of an object determines behavior with *generic functions*.

- `class` attribute
- `methods(class="data.frame")`
- `methods(plot)`

`UseMethod` indicates this is a generic function.

## 11 Statistical models in R

The `~` operator is used to define a *model formula*. The form for an ordinary linear model is:
```R
response ~ op_1 term_1 op_2 term_2 ...
```

- *response*: a vector or matrix defining the response variable(s)
- *op_i*: an operator, either `+` or `-`, implying the inclusion or exclusion of a term.
- *term_i*: is either:
  - a vector or matrix expression, or 1
  - a factor
  - a formula expression consisting of factors, vectors or matrices connected by formula operators

The modal formulae specify the *columns* of the model matrix.

### Notations

```R
Y ~ M # Y is modeled as M
M_1 + M_2 # Include M_1 and M_2
M_1 - M_2 # include M_1 leaving out terms of M_2
M_1 : M_2 # the tensor product of M_1 and M_2. If both terms are factors, then the "subclasses" factor
M_1 %in% M_2 # Similar to :, but with a dirrerent coding
M_1 * M_2 # M_1 + M_2 + M_1:M_2
M_1 / M_2 # M_1 + M_2 %in% M_1
M^n # All term in M together with "interactions" up to order n
I(M) # Insulate M. Inside M all operators have their normal arithmetic meaning, and that term appears in the model matrix.
```


### Examples

```R
# Simple linear regression model of y on x. The first has an implicit intercept term.
y ~ x
y ~ 1 + x 

# Simple linear regression of y on x through the origin
y ~ 0 + x
y ~ -1 + x
y ~ x - 1

# Mutiple regression of the transformed variable
log(y) ~ x1 + x2

# polynomial regression of y on x of degree 2
y ~ poly(x,2)
y ~ 1 + x + I(X^2)
```

### Contrasts

### LInear models

```R
fitted.model <- lm(formula, data = data.frame)

# operates at the simplest level in a very similar way to lm(), aloows an analysis of models with multiple error strata
aov(formula, data = data.frame)

# allows a model to be fitted that differs from one previously
new.moddel <- update(old.model, new.formula)

fm05 <- lm(y ~ x1 + x2 + x3 + x4 + x5, data=production)
# . can be used to stand for the corresponding part of the old model formula
fm6 <- update(fm05, . ~ . + x6)
smf6 <- update(fm6, sqrt(.) ~ .)

# . can also used in lm(), represent for all other variables in the data frame
fmfull <- lm(y ~ ., data = production)
```

### Generic functions for extracting model info

```R
anova(obj_1, obj_2)
coef(obj) # Extract the regression coefficient (matrix).
deviance(obj)
formula(obj)
plot(obj)
predict(obj, newdata=data.frame)
print(obj)
residuals(obj)
step(obj)
summary(obj)
vcov(obj)
```

## 12 Graphical procedures

### High-level plotting commands

Designed to genrate a complete plot of the data, alwats start a new plot, erasing the current plot.

#### `plot()`

```R
# If x and y are vectors, produces a scatterplot
plot(x, y)
plot(xy) 

# If x is a time series, produces a time-series plot. If x is a numeric vector, produces a plot of the calues in the vector against their index.
plot(x)

# f is a factor object, y is a numeric vector. The first generates a bar plot of f; the second form produces boxplots of y for each level of f
plot(f)
plot(f, y)

plot(df)
plot(~ expr)
plot(y ~ expr)
```
#### Displaying multivariate data

- `pairs(X)`
- `coplot()`

#### Display graphics

```R
# Distribution-comparison plots.
qqnorm(x)
qqline(x)
qqplot(x, y)

# Histogram
hist(x)
hist(, nclass=n)
hist(x, breaks=b, ...)

# Dotchart
dotchart(x, ...)

# Plots of three variables
image(x, y, z, ...)
contour(x, y, z, ...)
persp(x, y, z, ...)

#### Arguments to high-level plotting functions

```R
add
axes
log
type
xlab
ylab
main
```

### Low-level plotting commands

```R
points(x, y)
lines(x, y)
text(x, y, labels, ...)

abline(a, b)
abline(h=y)
abline(v=x)
abline(lm.obj)

polygon(x, y, ...)

legend(x, y, legend)

title(main, sub)

axis(side, ...)
```

#### Mathematical annotation

```R
help(plotmath)
example(plotmath)
demo(plotmath)
```

### Interacting with graphics

```R
locator(n, type)

identify(x, y, labels)
```

### Using graphics parameters

#### `par()`

Used to access and modify the list of graphics parameters for the current graphics device.

```R
par() # Return a list of all graphics paras.
par(c("col", "lty")) # Returns only the named graphics parameters.
par(col=4, lty=2) # Sets the values of the named paras.
```

### Graphics parameters list

#### Graphical elements

```R
# Character to be used for plotting points
pch="+"
pch=4

# Line types
lty=2

# Line widths
lwd=2

# Colors
col = 2 # see ?palette
col.axis
col.lab
col.main
col.sub

font = 2
font.asix...

# Justification of text relative tot the plotting position.
adj=-0.1 

# Character expansion
cex=1.5
cex.asix...
```

#### Axes and tick marks

Axes have three main components:
- axis line
- tick marks
- tick labels

```R
lab=c(5,7,12) # The desired number of tick intervals on the x and y axes and length of axis labels.
las=1 # Orientation of axis labels.
mgp=c(3,1,0) # Positions of axis components.
tck=0.01 # Length of tick marks.

# Axis styles for x and y axes, "i"(internal), "r"(default)
xaxs="r"
yaxs="i"
```

#### Figure margins

A single plot is known as a **figure** and comprises a *plot region* surrounded by margins.

![plot margin](_v_images/_plotmargin_1530088595_399663081.png)

```R
mai=c(1, 0.5, 0.5, 0) # Widths of the bottom, left, top, right margins

mar # Similar to mai, except the measurement unit is text lines
```

#### Multiple figure environment

```R
# Set the size of a multiple figure array
mfcol=c(3,2)
mfrow=c(2,4)

# Position of the current figure
mfg=c(2,2,3,2)

# Position of the current figure on the page
fig=c(4,9,1,4)/10

# Size of outer margins
oma
omi
```

### Device drivers

```R
X11() # For unix
windows()
quartz() # For mac
postscript()
pdf()
png()
jpeg()

dev.off()
```

#### Multiple graphics devices

```R
quartz()
pdf()
# Each new call to a device driver function opens a new graphics device

dev.list()
dev.next()
dev.prev()
dev.set(which=k)
dev.off(k)
```

## 13 Packages

- `library()`: see which packages are installed.
- `library(name)`: load package.
- `install/update.packages()`
- `search()`: see which packages are currently loaded.
- `loadedNamespaces()`: display packages may be loaded but not available on the search list

### Namespaces

Packages have *naespaces*:
- allow to hide functions and data that are meant only for internal use.
- prevent functions from name clashes.
- provide a way to refer to an object within a particular package.

`t()` is the same as `base::t()`

`:::` allows access to hidden obejcts. Users are more likely to use `getAnywhere()`.
