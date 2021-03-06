# listenv: Environments Behaving (Almost) as Lists

## Creating list environments
List environments are created similarly to lists but also similarly to environments.  To create an empty list environment, use
```r
> x <- listenv()
> x
A 'listenv' vector with 0 elements.
```
This can later can be populated using named assignments,
```r
> x$a <- 1
> x
A 'listenv' vector with 1 element ('a').
```
comparable to how both lists and environments work.  Similarly to lists, they can also be populated using indices, e.g.
```r
> x[[2]] <- 2
> x$c <- 3
> x
A 'listenv' vector with 3 elements ('a', '', 'c').
```
Just as for lists, a list environment is expanded with `NULL` elements whenever a new element is added that is beyond the current length plus one, e.g.
```r
> x[[5]] <- 5
> x
A 'listenv' vector with 5 elements ('a', '', 'c', '', '').
> x[[4]]
NULL
```

As with lists, the above list environment can also be created from the start, e.g.
```r
> x <- listenv(a = 1, 3, c = 4, NULL, 5)
> x
A 'listenv' vector with 5 elements ('a', '', 'c', '', '').
```


As for lists, the length of a list environment can at any time be increased or decreased by assigning it a new length.
If decreased, elements are dropped, e.g.
```r
> x
A 'listenv' vector with 5 elements ('a', '', 'c', '', '').
> length(x) <- 2
> x
A 'listenv' vector with 2 elements ('a', '').
> x[[1]]
[1] 1
> x[[2]]
[1] 3
```
If increased, new elements are populated with unnamed elements of `NULL`, e.g.
```r
> length(x) <- 4
> x
A 'listenv' vector with 4 elements ('a', '', '', '').
> x[[3]]
NULL
> x[[4]]
NULL
```

To allocate an "empty" list environment (with all `NULL`:s) of a given length, do
```r
> x <- listenv()
> length(x) <- 4
> x
A 'listenv' vector with 4 unnamed elements.
```
_Note_: Unfortunately, it is _not_ possible to use `x <- vector("listenv", length=4)`; that construct is only supported for the basic data types.



## Iterating over elements

### Iterating over elements by names
Analogously to lists and plain environments, it is possible to iterate over elements of list environments by the element names.  For example,
```r
> x <- listenv(a = 1, b = 2, c = 3)
> for (name in names(x)) {
+     cat(sprintf("Element %s: %s\n", sQuote(name), x[[name]]))
+ }
Element 'a': 1
Element 'b': 2
Element 'c': 3
```

### Iterating over elements by indices
Analogously to lists, but contrary to plain environments, it is also possible to iterate over elements by their indices.  For example,
```r
> x <- listenv(a = 1, b = 2, c = 3)
> for (ii in seq_along(x)) {
+     cat(sprintf("Element %d: %s\n", ii, x[[ii]]))
+ }
Element 1: 1
Element 2: 2
Element 3: 3
```


## Coercion to and from list environments

### Coercing to lists and vectors

Coercing a list environment to a list:
```r
> x <- listenv(a = 2, b = 3, c = "hello")
> x
A 'listenv' vector with 3 elements ('a', 'b', 'c').
> y <- as.list(x)
> str(y)
List of 3
 $ a: num 2
 $ b: num 3
 $ c: chr "hello"
```

Coercing a list to a list environment:
```r
> z <- as.listenv(y)
> z
A 'listenv' vector with 3 elements ('a', 'b', 'c').
> identical(z, x)
[1] FALSE
> all.equal(z, x)
[1] TRUE
```

Unlisting:
```r
> unlist(x)
      a       b       c 
    "2"     "3" "hello" 
> unlist(x[-3])
a b 
2 3 
> unlist(x[1:2], use.names = FALSE)
[1] 2 3
```


## Multi-dimensional list environments

Analogously to lists, and contrary to plain environments, list environments can have dimensions with corresponding names.  For example,
```r
> x <- as.listenv(1:6)
> dim(x) <- c(2, 3)
> dimnames(x) <- list(c("a", "b"), c("A", "B", "C"))
> x
A 'listenv' matrix with 6 unnamed elements arranged in 2x3 rows ('a', 'b') and columns ('A', 'B', 'C').
```
An easy way to quickly get an overview is to coerce to a list, e.g.
```r
> as.list(x)
  A B C
a 1 3 5
b 2 4 6
```
Individual elements of a list environment can be accessed using standard subsetting syntax, e.g.
```r
> x[["a", "B"]]
[1] 3
> x[[1, 2]]
[1] 3
> x[[1, "B"]]
[1] 3
```
We can assign individual elements similarly, e.g.
```r
> x[["b", "B"]] <- -x[["b", "B"]]
> as.list(x)
  A B  C
a 1 3  5
b 2 -4 6
```
We can also assign multiple elements through dimensional subsetting, e.g.
```r
> x[2, -1] <- 98:99
> as.list(x)
  A B  C 
a 1 3  5 
b 2 98 99
> x["a", c(1, 3)] <- list(97, "foo")
> as.list(x)
  A  B  C    
a 97 3  "foo"
b 2  98 99   
> x[] <- 1:6
> as.list(x)
  A B C
a 1 3 5
b 2 4 6
```


Concurrently with dimensional names it is possible to have names of the invidual elements just as for list environments without dimensions.  For example,
```r
> names(x) <- letters[seq_along(x)]
> x
A 'listenv' matrix with 6 elements ('a', 'b', 'c', ..., 'f') arranged in 2x3 rows ('a', 'b') and columns ('A', 'B', 'C').
> x[["a"]]
[1] 1
> x[["f"]]
[1] 6
> x[c("a", "f")]
A 'listenv' vector with 2 elements ('a', 'f').
> unlist(x)
a b c d e f 
1 2 3 4 5 6 
```
Contrary to lists, element names are preserved also with multi-dimensional subsetting, e.g.
```r
> x[1, 2]
A 'listenv' vector with 1 element ('c').
> x[1, 2, drop = FALSE]
A 'listenv' matrix with 1 element ('c') arranged in 1x1 rows ('a') and columns ('B').
> x[1:2, 2:1]
A 'listenv' matrix with 4 elements ('c', 'd', 'a', 'b') arranged in 2x2 rows ('a', 'b') and columns ('B', 'A').
> x[2, ]
A 'listenv' vector with 3 elements ('b', 'd', 'f').
> x[2, , drop = FALSE]
A 'listenv' matrix with 3 elements ('b', 'd', 'f') arranged in 1x3 rows ('b') and columns ('A', 'B', 'C').
> x["b", -2, drop = FALSE]
A 'listenv' matrix with 2 elements ('b', 'f') arranged in 1x2 rows ('b') and columns ('A', 'C').
```


Note, whenever dimensions are set using `dim(x) <- dims` both the dimensional names and the element names are removed, e.g.
```r
> dim(x) <- NULL
> names(x)
NULL
```
This behavior is by design, cf. `help("dim", package="base")`.



## Important about environments
List environments are as their name suggests _environments_.  Whenever working with environments in R, it is important to understand that _environments are mutable_ whereas all other of the basic data types in R are immutable.  For example, consider the following function that assigns zero to element `a` of object `x`:
```r
> setA <- function(x) {
+     x$a <- 0
+     x
+ }
```
If we pass a regular list to this function,
```r
> x <- list(a = 1)
> y <- setA(x)
> x$a
[1] 1
> y$a
[1] 0
```
we see that `x` is unaffected by the assignment.  This is because _lists are immutable_ in R.  However, if we pass an environment instead,
```r
> x <- new.env()
> x$a <- 1
> y <- setA(x)
> x$a
[1] 0
> y$a
[1] 0
```
we find that `x` was affected by the assignment.  This is because _environments are mutable_ in R.  Since list environments inherits from environments, this also goes for them, e.g.
```r
> x <- listenv(a = 1)
> y <- setA(x)
> x$a
[1] 0
> y$a
[1] 0
```

What is also important to understand is that it is not just the _content_ of an environment that is mutable but also its _attributes_.  For example,
```r
> x <- listenv(a = 1)
> y <- x
> attr(y, "foo") <- "Hello!"
> attr(x, "foo")
[1] "Hello!"
```


[listenv]: http://cran.r-project.org/package=listenv


## Installation
R package listenv is available on [CRAN](http://cran.r-project.org/package=listenv) and can be installed in R as:
```r
install.packages('listenv')
```




## Software status

| Resource:     | CRAN        | Travis CI     | Appveyor         |
| ------------- | ------------------- | ------------- | ---------------- |
| _Platforms:_  | _Multiple_          | _Linux_       | _Windows_        |
| R CMD check   | <a href="http://cran.r-project.org/web/checks/check_results_listenv.html"><img border="0" src="http://www.r-pkg.org/badges/version/listenv" alt="CRAN version"></a> | <a href="https://travis-ci.org/HenrikBengtsson/listenv"><img src="https://travis-ci.org/HenrikBengtsson/listenv.svg" alt="Build status"></a> | <a href="https://ci.appveyor.com/project/HenrikBengtsson/listenv"><img src="https://ci.appveyor.com/api/projects/status/github/HenrikBengtsson/listenv?svg=true" alt="Build status"></a> |
| Test coverage |                     | <a href="https://coveralls.io/r/HenrikBengtsson/listenv"><img src="https://coveralls.io/repos/HenrikBengtsson/listenv/badge.svg?branch=develop" alt="Coverage Status"/></a>   |                  |
