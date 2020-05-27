
<!-- README.md is generated from README.Rmd. Please edit that file -->

# Q7

<!-- badges: start -->

<!-- [![Lifecycle: experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental) -->

<!-- badges: end -->

`Q7` enables *Compositional Object Oriented Programming*, a fluid,
powerful and *postmodern* paradigm, featuring:

#### Smart Objects

  - functions within an object know:
      - Where am I? What are my neighbors?
  - extensible
      - Make variants of an object

#### No Magic

  - Mechanism decomposes into basic R constructs
      - A *type* is a function
      - A *feature* is a function
      - An *instance*, created by *type* and *feature*, is an
        environment
  - Same great R syntax & semantics
      - Perform any action on or within an object
      - Native scoping rules, almost no NSE

#### Compositional Manner of Construction

  - Freely add, change or delete members, ad or post hoc
  - Focuses on “has a”, rather than than “is a” relationships
  - Objects can contain other objects

#### Unlocked Objects

  - Object instances are unlocked environments
  - Feel free to shoot your feet, if so desired
  - Want safety? Lock’em yerself

## Installation

``` r
# install.packages("devtools")
devtools::install_github("iqis/Q7")
```

## Examples

Let’s make a `String` type and re-pack some base R functions into it,
giving the object a distinct API, which some users may find familar
with.

``` r
require(Q7)
#> Loading required package: Q7
#> Loading required package: magrittr
String <- type(function(string){
  if (length(string) != 1 || !is.character(string)) { 
    # we only want this object to hold a 1-length character vector
    stop("string must be a vector of length 1")
  }
  .my$string <- string
  
  .my$charAt <- function(index){
    unlist(strsplit(.my$string, ""))[index + 1]
  }
  
  .my$concat <- function(str){
    stopifnot(inherits(str, 
                       "String"))
    
    .my$string <- paste0(.my$string, 
                         str$string)
    .my
  }
  
  .my$length <- function(){
    nchar(.my$string)
  }
  
  .my$isEmpty <- function(){
    nchar(.my$string) == 0
  }
  
  .my$matches <- function(pattern){
    .my$string <- grepl(pattern, .my$string)
    .my
  }
  
  .my$replaceFirst <- function(regex, replacement){
    .my$string <- sub(regex, 
                      replacement, 
                      .my$string)
    .my
  }
}, s3 = "String")

print.String <- function(String){
  if (inherits(String, "Q7instance")) {
    print(String$string)
  } else if (inherits(String, "Q7type")) {
    print(String)
  }
}
```

We have defined a type named *String*, which will produce instances with
a S3 class *String*, as well as a S3 method for `print()`. The `$string`
member is the vector that holds the data; All other members are bound
functions (methods) of the object. They understand that they live inside
the `String` object. The `print.String()` function lives in the global
scope, and simply reaches into the `String` object and print the
`$string$` vector.

`.my` object is a special object that refers to the instance itself.
Although it can be safely excluded in most cases, with some precaution
in place, the use of `.my` is highly recommended to avoid scope leak.

``` r
motto <- String("Carpe")$concat(String(" "))$concat(String("Diem"))

motto$length()
#> [1] 10
motto$isEmpty()
#> [1] FALSE
motto$charAt(5)
#> [1] " "
motto$replaceFirst("\\s", "-")$charAt(5)
#> [1] "-"
motto
#> [1] "Carpe-Diem"
```

### API

  - `type()`
      - Defines a *type*. (like *class*)
      - Takes a function or expression as constructor
      - When invoked, the constructor’s closure becomes an *instance*,
        which is an environment
          - Contains every binding inside the closure, except for the
            arguments
          - The arguments are not accessible outside of the object,
            making them private
          - Also contains `.my`, which refers to the instance itself
  - `feature()`
      - Defines a *feature*
      - Takes an expression
      - Appends the expression to the object
          - Ad hoc: A *feature* can be implemented on a *type*
          - Post hoc: Can also be implemented on an *instance*
  - `implement()`
      - Takes
          - object, a *type* or *instance*
          - any expression (including *features*, but more importantly,
            any arbitrary expression)
      - Appends the expresseion to the object

`Q7` users can leave behind the grand narrative of the classical OOP
orthodoxy, and exploit the benefits of objects simply as a unit of code,
and as an instrument for namespace resolution.

Make a type:

``` r
TypeOne <- type(function(arg1, arg2){
  
  
})
```

`type()` takes a function, which is used to construct an instance.

Everything defined within the function’s closure becomes members of the
object. Arguments supplied to the function are accesible to the closure,
but not become members of the object themselves.

``` r
type_one <- TypeOne(1, 2)
type_one$...
#> NULL
```

The object can be modified post-hoc.

``` r
type_one %>% implement({
  
  
})
```

The features implemented can be packaged with `feature()`.

``` r
hasFeatureOne <- feature({
  x <- 1
  y <- function(n){
    x + n
  }
})
```

``` r
hasFeatureTwo <- feature({
  x <- 2 # Overrides x from hasFeatureOne
  old_y <- y
  y <- function(n){
    old_y(n) + 1
  }
})
```

#### Private members

``` r

Counter <- type(function(count = 0){
    add_one <- function(){
      count <<- count + 1
    }
    
    get_count <- function(){
      count
    }
})
```

``` r
counter <- Counter()
ls(counter)
#> [1] "add_one"   "get_count"
counter$get_count()
#> [1] 0
counter$add_one()
counter$add_one()
counter$get_count()
#> [1] 2
```

##### Get Access to Private Environment

The private environment is upstream to (parent of) the public
environment. The following code allows direct outside access to the
`count` object.

``` r
exposePrivateEnv <- feature({
  .pvt <- parent.env(.my)
})

counter %>% exposePrivateEnv()
counter$.pvt
#> <environment: 0x000000001c896d40>
counter$.pvt$count
#> [1] 2
```

#### Why is there no parameterized features?

*feature* is subordinate to and dependent on *type*. It is encouraged to
put all data members in a *type* definition, while *feature* mainly
contain functions. If you feel significant need to parameterize a
feature, think if it’s better to create a nested object or to formally
extend a type. You can always re-define something in a feature post hoc.

``` r
Word <- type(function(word){})
hasRepeat <- feature({
  N_TIMES <- 2
  repeat_word <- function(){
    cat(rep(word, N_TIMES))
  }
})
```

``` r
apple <- Word("apple") %>% hasRepeat()
apple$repeat_word()
#> apple apple
```

``` r
pear <- Word("pear") %>% 
  implement({
    hasRepeat()
    N_TIMES <- 5
  })
pear$repeat_word()
#> pear pear pear pear pear
```

``` r
repeatWordNTimes <- function(word, times){
  localize(Word)(word) %>% 
    hasRepeat() %>% 
    implement({
      N_TIMES <- times
    })
}

orange <- repeatWordNTimes("orange", 7)
orange$repeat_word()
#> orange orange orange orange orange orange orange
```
