# Names and values {#names-values}



## Introduction

In R, it is important to understand the distinction between an object and its name. A correct mental model is important because it will help you:

* More accurately predict performance and memory usage of R code. 
* Write faster code because accidental copies are a major cause of slow code. 
* Better understand R's functional programming tools.

The goal of this chapter is to help you understand the distinction between names and values, and when R will copy an object.

### Quiz {-}

Answer the following questions to see if you can safely skip this chapter. You can find the answers at the end of the chapter in Section \@ref(names-values-answers).

1.  Given the following data frame, how do I create a new column called "3"
    that contains the sum of `1` and `2`? You may only use `$`, not `[[`.
    What makes `1`, `2`, and `3` challenging as variable names?

    
    ```r
    df <- data.frame(runif(3), runif(3))
    names(df) <- c(1, 2)
    ```

1.  In the following code, how much memory does `y` occupy?
   
    
    ```r
    x <- runif(1e6)
    y <- list(x, x, x)
    ```

1.  On which line does `a` get copied in the following example?

    
    ```r
    a <- c(1, 5, 3, 2)
    b <- a
    b[[1]] <- 10
    ```

### Outline {-}

* Section \@ref(binding-basics) introduces you to the distinction between
  names and values, and discusses how `<-` creates a binding, or reference,
  between a name and a value. 

* Section \@ref(copy-on-modify) describes when R makes a copy; whenever you
  modify vector, you're almost always actually create a new, modified vector.
  You'll learn how to use `tracemem()` to figure out when a copy actually
  occurs, and then explore the implications as they apply to function calls, 
  lists, data frames, and character vectors. 

* Section \@ref(object-size) explores the implications of the previous two
  sections on how much memory an object occupies. You'll learn to use 
  `lobstr::obj_size()` as your intuition may be profoundly wrong, and the
  base `object.size()` is unfortunately inaccurate.

* Section \@ref(modify-in-place) describes the two important exceptions to
  copy-on-modify: values with a single name, and environments. In these two
  special cases, objects are actually modified in place.

* Section \@ref(gc) closes out the chapter with a discussion of the 
  garbage collector, which frees up memory used by objects that are no longer
  referenced by a name.

### Prerequisites {-}

We'll use the development version of [lobstr](https://github.com/r-lib/lobstr) to dig into the internal representation of R objects.


```r
# devtools::install_github("r-lib/lobstr")
library(lobstr)
```

### Sources {-}

The details of R's memory management are not documented in a single place. Much of the information in this chapter was gleaned from a close reading of the documentation (particularly `?Memory` and `?gc`), the [memory profiling](http://cran.r-project.org/doc/manuals/R-exts.html#Profiling-R-code-for-memory-use) section of "Writing R extensions" [@r-exts], and the [SEXPs](http://cran.r-project.org/doc/manuals/R-ints.html#SEXPs) section of "R internals" [@r-ints]. The rest I figured out by reading the C source code, performing small experiments, and asking questions on R-devel. Any mistakes are entirely mine.

## Binding basics
\index{bindings} \index{assignment}

Take this code: 


```r
x <- c(1, 2, 3)
```

It's easy to read it as: "create an object named 'x', containing the values 1, 2, and 3". Unfortunately, that's a simplification that will lead to you make inaccurate predictions about what R is actually doing behind the scenes. It's more accurate to think about this code as doing two things:

* Creating an object, a vector of values, `c(1, 2, 3)`.
* Binding the object to a name, `x`.

Note that the object, or value, doesn't have a name; it's the name that has a value. To make that distinction more clear, I'll draw diagrams like this: 

<img src="diagrams/name-value/binding-1.png" style="display: block; margin: auto;" />

The name, `x`, is drawn with a rounded rectangle, and it has an arrow that points to, binds, or references, the value, the vector `1:3`. Note that the arrow points in opposite direction to the assignment arrow: `<-` creates a binding from the name on the left-hand side to the object on the right-hand side.

You can think of a name as a reference to a value. For example, if you run this code, you don't get another copy of the value `1:3`, you get another binding to the existing object:


```r
y <- x
```
<img src="diagrams/name-value/binding-2.png" style="display: block; margin: auto;" />

You might have noticed the value `1:3` has a label: `0x74b`. While the vector doesn't have a name, I'll occasionally need to refer to objects independently of their bindings. To make that possible, I'll label values with a unique identifier. These unique identifers have a special form that looks like the object's memory "address", i.e. the location in memory in which the object is stored. It doesn't make sense to use the actual memory address because that changes every time the code is run.

You can access the address of an object with `lobstr::obj_addr()`. This allows us to see that `x` and `y` both point to the same location in memory:


```r
obj_addr(x)
#> [1] "0x7ff8453a8b68"
obj_addr(y)
#> [1] "0x7ff8453a8b68"
```

These identifiers are long, and change every time you restart R.

It takes some time to get your head around the distinction between names and values, but it's really helpful for functional programming when you start to work with functions that have different names in different contexts.

### Non-syntactic names {#non-syntactic}
\index{reserved names} 
\indexc{`} 
\index{non-syntactic names}

R has strict rules about what constitutes a valid name. A __syntactic__ name must consist of letters[^letters], digits, `.` and `_`, and can't begin with `_` or a digit. Additionally, it can not be one of a list of __reserved words__ like `TRUE`, `NULL`, `if`, and `function` (see the complete list in `?Reserved`). Names that don't follow these rules are called __non-syntactic__ names, and if you try to use them, you'll get an error:


```r
_abc <- 1
#> Error: unexpected input in "_"

if <- 10
#> Error: unexpected assignment in "if <-"
```

[^letters]: Surprisingly, what constitutes a letter is determined by your current locale. That means that the syntax of R code actually differs from computer to computer, and it's possible for a file that works on one computer to not even parse on another!

It's possible to override the usual rules and use a name with any sequence of characters by surrounding the name with backticks:


```r
`_abc` <- 1
`_abc`
#> [1] 1

`if` <- 10
`if`
#> [1] 10
```

Typically, you won't deliberately create such crazy names. Instead, you need to understand them because you'll be subjected to the crazy names created by others. This happens most commonly when you load data that has been created outside of R.

::: sidebar
You _can_ create non-syntactic bindings using single or double quotes (e.g. `"_abc" <- 1`) instead of backticks, but you shouldn't, because you'll have to use a different syntax to retrieve the values. The ability to use strings on the left hand side of the assignment arrow is a historical artefact, used before R supported backticks.
:::

### Exercises

1.  Explain the relationship between `a`, `b`, `c` and `d` in the following 
    code:

    
    ```r
    a <- 1:10
    b <- a
    c <- b
    d <- 1:10
    ```

1.  The following code accesses the mean function in multiple different ways.
    Do they all point to the same underlying function object? Verify with
    `lobstr::obj_addr()`.
    
    
    ```r
    mean
    base::mean
    get("mean")
    evalq(mean)
    match.fun("mean")
    ```
    
1.  By default, base R data import functions, like `read.csv()`, will automatically
    convert non-syntactic names to syntactic names. Why might this be 
    problematic? What option allows you to suppress this behaviour?
    
1.  What rules does `make.names()` use to convert non-syntactic names into
    syntactic names?

1.  I slightly simplified the rules that govern syntactic names. Why is `.123e1`
    not a syntactic name? Read `?make.names` for the full details.

## Copy-on-modify

Consider the following code, which binds `x` and `y` to the same underlying value, then modifies[^double-bracket] `y`.

[^double-bracket]: You may be surprised to see `[[` used with a numeric vector. We'll come back to this in Section \@ref(subset-single), but in brief, I think you should use `[[` whenever you are getting or setting a single element.


```r
x <- c(1, 2, 3)
y <- x

y[[3]] <- 4
x
#> [1] 1 2 3
```

Modifying `y` clearly doesn't modify `x`, so what happened to the shared binding? While the value associated with `y` changes, the original object does not. Instead, R creates a new object, `0xcd2`, a copy of `0x74b` with one value changed, then rebinds `y` to that object.

<img src="diagrams/name-value/binding-3.png" style="display: block; margin: auto;" />

This behaviour is called __copy-on-modify__, and understanding it makes your intuition for the performance of R code radically better. A related way to describe this phenomenon is to say that R objects are __immutable__, or unchangeable. However, I'll generally avoid that term because there are a couple of important exceptions to copy-on-modify that you'll learn about in Section \@ref(modify-in-place). 

### `tracemem()`

You can see when an object gets copied with the help of `base::tracemem()`. You call it with an object and it returns the current address of the object:


```r
x <- c(1, 2, 3)
cat(tracemem(x), "\n")
#> <0x7f80c0e0ffc8> 
```

Whenever that object is copied in the future, `tracemem()` will print out a message telling you which object was copied, what the new address is, and the sequence of calls that lead to the copy:


```r
y <- x
y[[3]] <- 4L
#> tracemem[0x7f80c0e0ffc8 -> 0x7f80c4427f40]: 
```

Note that if you modify `y` again, it doesn't get copied. That's because the new object now only has a single name binding to it, so R can apply a modify-in-place optimisation. We'll come back to that shortly.


```r
y[[3]] <- 5L

untracemem(y)
```

`untracemem()` is the opposite of `tracemem()`; it turns tracing off.

### Function calls

The same rules for copying also apply to function calls. Take this code:


```r
f <- function(a) {
  a
}

x <- c(1, 2, 3)
cat(tracemem(x), "\n")
#> <0x7ff840b48578>

z <- f(x)
# there's no copy here!

untracemem(x)
```

While `f()` is running, `a` inside the function will point to the same value as `x` does outside of it:

<img src="diagrams/name-value/binding-f1.png" style="display: block; margin: auto;" />

(You'll learn more about the conventions used in this diagram in [Execution environments].)

And once complete, `x` and `z` will point to the same object. `0x74b` never gets copied because it never gets modified. If `f()` did modify `x`, R would create a new copy, and then `z` would bind that object. 

<img src="diagrams/name-value/binding-f2.png" style="display: block; margin: auto;" />

### Lists {#list-references}

It's not just names (i.e. variables) that point to values; the elements of lists do too. Take this list, which superficially is very similar to the vector above:


```r
l1 <- list(1, 2, 3)
```

The internal representation of the list is actually quite different to that of a vector. A list is really a vector of references:

<img src="diagrams/name-value/list.png" style="display: block; margin: auto;" />

This is particularly important when we modify a list:


```r
l2 <- l1
```

<img src="diagrams/name-value/l-modify-1.png" style="display: block; margin: auto;" />


```r
l2[[3]] <- 4
```

<img src="diagrams/name-value/l-modify-2.png" style="display: block; margin: auto;" />

Like vectors, lists are copied-on-modify; the original list is left unchanged, and R creates a modified copy. This is a __shallow__ copy: the list object and its bindings are copied, but the values pointed to by the bindings are not. The oppposite of a shallow copy is a deep copy, where the contents of every reference are also copied. Prior to R 3.1.0, copies were always deep copies, .

You can use `lobstr::ref()` to see values that are shared across lists. `ref()` prints the memory address of each object, along with a local id so that you can easily cross-reference shared components.


```r
ref(l1, l2)
#> █ [1:0x7ff8454e4a48] <list> 
#> ├─[2:0x7ff845499d78] <dbl> 
#> ├─[3:0x7ff845499d40] <dbl> 
#> └─[4:0x7ff845499d08] <dbl> 
#>  
#> █ [5:0x7ff8471fd3f8] <list> 
#> ├─[2:0x7ff845499d78] 
#> ├─[3:0x7ff845499d40] 
#> └─[6:0x7ff8444ffb50] <dbl>
```

### Data frames {#df-modify}

Data frames are lists of vectors, so copy-on-modify has important consequences when you modify a data frame. Take this data frame as an example:


```r
d1 <- data.frame(x = c(1, 5, 6), y = c(2, 4, 3))
```
<img src="diagrams/name-value/dataframe.png" style="display: block; margin: auto;" />

If you modify a column, only that column needs to be modified; the others can continue to point to the same place:


```r
d2 <- d1
d2[, 2] <- d2[, 2] * 2
```
<img src="diagrams/name-value/d-modify-c.png" style="display: block; margin: auto;" />

However, if you modify a row, there is no way to share data with the previous version of the data frame, and every column must be copied-and-modified.


```r
d3 <- d1
d3[1, ] <- d3[1, ] * 3
```
<img src="diagrams/name-value/d-modify-r.png" style="display: block; margin: auto;" />

### Character vectors
\index{string pool}

The final place that R uses references is in character vectors. I usually draw character vectors like this:


```r
x <- c("a", "a", "abc", "d")
```
<img src="diagrams/name-value/character.png" style="display: block; margin: auto;" />

But this is a polite fiction, because R has a __global string pool__. Each element of a character vector is actually a pointer to a unique string in that pool:

<img src="diagrams/name-value/character-2.png" style="display: block; margin: auto;" />

You can request that `ref()` show these references:


```r
ref(x, character = TRUE)
#> █ [1:0x7ff840b3e898] <chr> 
#> ├─[2:0x7ff8411f7b98] <string: "a"> 
#> ├─[2:0x7ff8411f7b98] 
#> ├─[3:0x7ff84157edc0] <string: "abc"> 
#> └─[4:0x7ff840874200] <string: "d">
```

This has a profound impact on the amount of memory a character vector takes but, but is otherwise not generally important, so elsewhere in the book I'll draw character vectors as if the strings live inside the vector.

### Exercises

1.  Why is `tracemem(1:10)` not useful?

1.  Explain why `tracemem()` shows two copies when you run this code.
    Hint: carefully look at the difference between this code and the code 
    shown earlier in the section.
     
    
    ```r
    x <- c(1L, 2L, 3L)
    tracemem(x)
    
    x[[3]] <- 4
    ```

1.  Sketch out the relationship between the following objects:

    
    ```r
    a <- 1:10
    b <- list(a, a)
    c <- list(b, a, 1:10)
    ```

1.  What happens when you run this code?

    
    ```r
    x <- list(1:10)
    x[[2]] <- x
    ```
    
    Draw a picture.

## Object size
\indexc{object\_size} 
\indexc{obj\_size}

You can find out how much space an object occupies in memory with `lobstr::obj_size()`[^object.size]:

[^object.size]: Beware of the base `utils::object.size()` function. It does not correctly account for shared references and will return sizes that are too large.


```r
obj_size(letters)
#> 1,792 B
obj_size(ggplot2::diamonds)
#> 3,457,104 B
```

Since the elements of lists are references to values, the size of a list might be much smaller than you expect:


```r
x <- runif(1e6)
obj_size(x)
#> 8,000,048 B

y <- list(x, x, x)
obj_size(y)
#> 8,000,128 B
```

`y` is only 72 bytes[^32bit] bigger than `x`. That's the size of an empty list with three elements:


```r
obj_size(list(NULL, NULL, NULL))
#> 80 B
```

[^32bit]: If you're running 32-bit R you'll see slightly different sizes.

Similarly, the global string pool means that character vectors take up less memory than you might expect: repeating a string 1000 times does not make it take up 1000 times as much memory.


```r
banana <- "bananas bananas bananas"
obj_size(banana)
#> 272 B
obj_size(rep(banana, 100))
#> 1,064 B
```

References also make it challenging to think about the size of individual objects. `obj_size(x) + obj_size(y)` will only equal `obj_size(x, y)` if there are no shared values. Here, the combined size of `x` and `y` is the same as the size of `y`:


```r
obj_size(x, y)
#> 8,000,128 B
```

### Exercises

1.  In the following example, why are `object.size(y)` and `obj_size(y)`
    so radically different? Consult the documentation of `object.size()`.

    
    ```r
    y <- rep(list(runif(1e4)), 100)
    
    object.size(y)
    #> 8005648 bytes
    obj_size(y)
    #> 80,896 B
    ```

1.  Take the following list. Why is its size somewhat misleading?

    
    ```r
    x <- list(mean, sd, var)
    obj_size(x)
    #> 17,664 B
    ```

1.  Predict the output of the following code:

    
    ```r
    x <- runif(1e6)
    obj_size(x)
    
    y <- list(x, x)
    obj_size(y)
    obj_size(x, y)
    
    y[[1]][[1]] <- 10
    obj_size(y)
    obj_size(x, y)
    
    y[[2]][[1]] <- 10
    obj_size(y)
    obj_size(x, y)
    ```

## Modify-in-place

As we've seen above, modifying an R object will usually create a copy. There are two exceptions that we'll explore below:

* Objects with a single binding get a special performance optimisation.

* Environments are a special type of object that is always modified in place.

### Objects with a single binding {#single-binding}

If an object only has a single name that binds it, R will modify it in place:


```r
v <- c(1, 2, 3)
```

<img src="diagrams/name-value/v-inplace-1.png" style="display: block; margin: auto;" />


```r
v[[3]] <- 4
```

<img src="diagrams/name-value/v-inplace-2.png" style="display: block; margin: auto;" />

(Carefully note the object ids here: `v` continues to bind to the same object, `0x207`.)

It's challenging to predict exactly when R applies this optimisation because of two complications:

* When it comes to bindings, R can currently[^refcnt] only count 0, 1, 
  and many. That means if an object has two bindings, and one goes away,
  the reference count does not go back to 1 (because one less than many is 
  still many).
  
* Whenever you call any regular function, it will make a reference to the 
  object. The only exception are specially written C functions. These occur 
  mostly in the base package.

[^refcnt]: By the time you read this, that may have changed, as plans are afoot to improve reference counting: https://developer.r-project.org/Refcnt.html

Together, this makes it hard to predict whether or not a copy will occur. Instead, it's better to determine it empirically with `tracemem()`. Let's explore the subtleties with a case study using for loops. For loops have a reputation for being slow in R, but often that slowness is because every iteration of the loop is creating a copy. 

Consider the following code. It subtracts the median from each column of a large data frame: \index{loops!avoiding copies}


```r
x <- data.frame(matrix(runif(5 * 1e4), ncol = 5))
medians <- vapply(x, median, numeric(1))

for (i in seq_along(medians)) {
  x[[i]] <- x[[i]] - medians[[i]]
}
```

This loop is surprisingly slow because every iteration of the loop copies the data frame, as revealed by using `tracemem()`:


```r
cat(tracemem(x), "\n")
#> <0x7f80c429e020> 

for (i in 1:5) {
  x[[i]] <- x[[i]] - medians[[i]]
}
#> tracemem[0x7f80c429e020 -> 0x7f80c0c144d8]: 
#> tracemem[0x7f80c0c144d8 -> 0x7f80c0c14540]: [[<-.data.frame [[<- 
#> tracemem[0x7f80c0c14540 -> 0x7f80c0c145a8]: [[<-.data.frame [[<- 
#> tracemem[0x7f80c0c145a8 -> 0x7f80c0c14610]: 
#> tracemem[0x7f80c0c14610 -> 0x7f80c0c14678]: [[<-.data.frame [[<- 
#> tracemem[0x7f80c0c14678 -> 0x7f80c0c146e0]: [[<-.data.frame [[<- 
#> tracemem[0x7f80c0c146e0 -> 0x7f80c0c14748]: 
#> tracemem[0x7f80c0c14748 -> 0x7f80c0c147b0]: [[<-.data.frame [[<- 
#> tracemem[0x7f80c0c147b0 -> 0x7f80c0c14818]: [[<-.data.frame [[<- 
#> tracemem[0x7f80c0c14818 -> 0x7f80c0c14880]: 
#> tracemem[0x7f80c0c14880 -> 0x7f80c0c148e8]: [[<-.data.frame [[<- 
#> tracemem[0x7f80c0c148e8 -> 0x7f80c0c14950]: [[<-.data.frame [[<- 
#> tracemem[0x7f80c0c14950 -> 0x7f80c0c149b8]: 
#> tracemem[0x7f80c0c149b8 -> 0x7f80c0c14a20]: [[<-.data.frame [[<- 
#> tracemem[0x7f80c0c14a20 -> 0x7f80c0c14a88]: [[<-.data.frame [[<- 

untracemem(x)
```

In fact, each iteration copies the data frame not once, not twice, but three times! Two copies are made by `[[.data.frame`, and a further copy[^shallow-copy] it made because `[[.data.frame` is a regular function and hence increments the reference count of `x`. 

[^shallow-copy]: Note that these copies are shallow, and only copy the reference to each individual column, not the contents. This means the performance isn't terrible, but it's obviously not as good as it could be.

We can reduce the number of copies by using a list instead of a data frame. Modifying a list uses internal C code, so the refs are not incremented and only a single copy is made:


```r
y <- as.list(x)
cat(tracemem(y), "\n")
#> <0x7f80c5c3de20>
  
for (i in 1:5) {
  y[[i]] <- y[[i]] - medians[[i]]
}
#> tracemem[0x7f80c5c3de20 -> 0x7f80c48de210]: 
```

While it's not hard to determine when copies are made, it is hard to prevent them. If you find yourself resorting to exotic tricks to avoid copies, it may be time to rewrite your function in C++, as described in Chapter \@ref(rcpp).

### Environments {#env-modify}

You'll learn more about environments in Chapter \@ref(environments), but it's important to mention them here because they behave differently to other objects: environments are always modified in place. This property is sometimes described as __reference semantics__ because when you modify an environment all existing bindings to the environment continue to have the same reference.

Take this environment, which we bind to `e1` and `e2`:


```r
e1 <- rlang::env(a = 1, b = 2, c = 3)
e2 <- e1
```

<img src="diagrams/name-value/e-modify-1.png" style="display: block; margin: auto;" />

If we change a binding, the environment is modified in place:


```r
e1$c <- 4
e2$c
#> [1] 4
```
<img src="diagrams/name-value/e-modify-2.png" style="display: block; margin: auto;" />

This basic idea can be used to create functions that "remember" their previous state. See Section \@ref(stateful-funs) for more details.

One consequence of this is that environments can contain themselves:


```r
e <- rlang::env()
e$self <- e

ref(e)
#> █ [1:0x7ff84845cd98] <env> 
#> └─self = [1:0x7ff84845cd98]
```
<img src="diagrams/name-value/e-self.png" style="display: block; margin: auto;" />

This is a unique property of environments!

### Exercises

1.  Wrap the two methods for subtracting medians into two functions, then
    use the bench [@bench] package to carefully compare their speeds. How does
    performance change as the number of columns increase?

1.  What happens if you attempt to use `tracemem()` on an environment?

## Unbinding and the garbage collector {#gc}
\index{garbage collector} 
\indexc{rm()}
\indexc{gc()}

Consider this code:


```r
x <- 1:3
```
<img src="diagrams/name-value/unbinding-1.png" style="display: block; margin: auto;" />


```r
x <- 2:4
```
<img src="diagrams/name-value/unbinding-2.png" style="display: block; margin: auto;" />


```r
rm(x)
```
<img src="diagrams/name-value/unbinding-3.png" style="display: block; margin: auto;" />

We create two objects, but by the end of code neither object is bound to a name. How do these objects get deleted? That's the job of the __garbage collector__, or GC, for short. The GC creates more memory by deleting R objects that are no longer used, and if needed, requesting more memory from the operating system. 

R uses a __tracing__ GC. That means it traces every object reachable from the global[^callstack] environment, and all the objects reachable from those objects (i.e. the references in lists and environments are searched recursively). The garbage collector does not use the reference count used for the modify-in-place optimisation described above. The two ideas are closely related but the internal data structures have been optimised for different use cases.

[^callstack]: And every environment on the current call stack.

The garbage collector (GC) is run automatically whenever R needs more memory to create a new object. From the outside, it's basically impossible to predict when the GC will run, and indeed, you shouldn't try. Instead, if you want to find out when the GC runs, call `gcinfo(TRUE)`: the the GC will print a message to the console every time it runs. 

You can force the garbage collector to run by calling `gc()`. Despite what you might have read elsewhere, there's never any _need_ to call `gc()` yourself. You may _want_ to call `gc()` to ask R to return memory to your operating system, or for its side-effect of telling you how much memory is currently being used:  


```r
gc() 
#>           used (Mb) gc trigger  (Mb) limit (Mb) max used  (Mb)
#> Ncells  676622 36.2    1242004  66.4         NA  1242004  66.4
#> Vcells 3681258 28.1   17080385 130.4      16384 17076540 130.3
```

`lobstr::mem_used()` is a wrapper around `gc()` that just prints the total number of bytes used:


```r
mem_used()
#> 67,339,984 B
```

This number won't agree with the amount of memory reported by your operating system for three reasons:

1. It only includes objects created by R, not the R interpreter itself.

1. Both R and the operating system are lazy: they won't reclaim memory 
   until it's actually needed. R might be holding on to memory because 
   the OS hasn't yet asked for it back.

1. R counts the memory occupied by objects but there may be gaps due to 
   deleted objects. This problem is known as memory fragmentation.

## Answers {#names-values-answers}

1.  You must surround non-syntactic names in `` ` ``. The variables `1`, `2`,
    and `3` have non-syntactic names, so must always be quoted with backticks.

    
    ```r
    df <- data.frame(runif(3), runif(3))
    names(df) <- c(1, 2)
    
    df$`3` <- df$`1` + df$`2`
    ```


1.  It occupies about 4 MB.
   
    
    ```r
    x <- runif(1e6)
    y <- list(x, x, x)
    obj_size(y)
    #> 8,000,128 B
    ```

1.  `a` is copied when `b` is modified, `b[[1]] <- 10`.
