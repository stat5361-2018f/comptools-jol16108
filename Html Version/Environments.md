# Environments {#environments}
\index{environments}



## Introduction

The environment is the data structure that powers scoping. This chapter dives deep into environments, describing their structure in depth, and using them to improve your understanding of the four scoping rules described in [lexical scoping](#lexical-scoping). 
Understanding environments is not necessary for day-to-day use of R. But they are important to understand because they power many important R features like lexical scoping, namespaces, and R6 classes, and interact with evaluation to give you powerful tools for making domain specific languages, like dplyr and ggplot2.

### Quiz {-}

If you can answer the following questions correctly, you already know the most important topics in this chapter. You can find the answers at the end of the chapter in [answers](#env-answers).

1.  List at least three ways that an environment is different to a list.

1.  What is the parent of the global environment? What is the only 
    environment that doesn't have a parent?
    
1.  What is the enclosing environment of a function? Why is it 
    important?

1.  How do you determine the environment from which a function was called?

1.  How are `<-` and `<<-` different?

### Outline {-}

* [Environment basics](#env-basics) introduces you to the basic properties
  of an environment and shows you how to create your own.
  
* [Recursing over environments](#env-recursion) provides a function template
  for computing with environments, illustrating the idea with a useful
  function.
  
* [Explicit environments](#explicit-envs) briefly discusses three places where
  environments are useful data structures for solving other problems.

### Prerequisites {-}

This chapter will use rlang functions for working with environments, because it allows us to focus on the essence of environments, rather than the incidental details. 


```r
library(rlang)

# Some API changes that haven't made it in rlang yet
search_envs <- function() {
  rlang:::new_environments(c(
    list(global_env()),
    head(env_parents(global_env()), -1)
  ))
}
```

Note that the `env_` functions in rlang are designed to work with the pipe: all take an environment as the first argument, and many also return an environment. I won't use the pipe in this chapter in the interest of keeping the code as simple as possible, but you should consider it for your own code.

## Environment basics {#env-basics}

Generally, an environment is similar to a named list, with four important exceptions:

*   Every name must be unique.

*   The names in an environment are not ordered (i.e. it doesn't make sense 
    to ask what the first element of an environment is).

*   An environment has a parent. 

*   Environments are not copied when modified.

Let's explore these ideas with code and pictures. 

### Basics
\index{environments!creating}

To create an environment, use `rlang::env()`. It works like `list()`, taking a set of name-value pairs:


```r
e1 <- env(
  a = FALSE,
  b = "a",
  c = 2.3,
  d = 1:3,
)
```

::: base
Use `new.env()` to creates a new environment. Ignore the `hash` and `size` parameters; they are not needed. Note that you can not simultaneously create and define values; use `$<-`, as shown below.
:::

The job of an environment is to associate, or __bind__, a set of names to a set of values. You can think of an environment as a bag of names, with no implied order (i.e. it doesn't make sense to ask which is the first element in an environment). For that reason, we'll draw the environment as so:

<img src="diagrams/environments/bindings.png" style="display: block; margin: auto;" />

As discussed in [names and values](#env-modify), environments have reference semantics: unlike most R objects, when you modify them, you modify them in place, and don't create a copy. One important implication is that environments can contain themselves. This means that environments go one step further in their level of recursion than lists: an enviroment can contain any object, including itself!


```r
e1$d <- e1
```

<img src="diagrams/environments/loop.png" style="display: block; margin: auto;" />

Printing an evironment just displays its memory address, which is not terribly useful:


```r
e1
#> <environment: 0x7fa5b2321440>
```

Instead, we'll use `env_print()` which gives us a little more information:


```r
env_print(e1)
#> <environment: 0x7fa5b2321440>
#>   parent: <environment: global>
#>   bindings:
#>    * a: <lgl>
#>    * b: <chr>
#>    * c: <dbl>
#>    * d: <env>
```

You can use `env_names()` to get a character vector giving the current bindings


```r
env_names(e1)
#> [1] "a" "b" "c" "d"
```

::: base
In R 3.2.0 and greater, use `names()` to list the bindings in an environment. If your code needs to work with R 3.1.0 or earlier, use `ls()`, but note that the default value of `all.names` is `FALSE` so you don't see any bindings that start with `.`.
:::

### Important environments

We'll talk in detail about special environments in [Special environments], but for now we need to mention two. The current environment, or `current_env()` is the environment in which code is currently executing. When you're experimenting interactively, that's usually the global environment, or `global_env()`. The global environment is sometimes called your "workspace", as it's where all interactive (i.e. outside of a function) computation takes place.

Note that to compare environments, you need to use `identical()` and not `==`:


```r
identical(global_env(), current_env())
#> [1] TRUE

global_env() == current_env()
#> Error in global_env() == current_env():
#>   comparison (1) is possible only for atomic and list types
```

:::base
Access the global environment with `globalenv()` and the current environment with `environment()`. The global environment is printed as `Rf_GlobalEnv` and `.GlobalEnv`.
:::

### Parents

Every environment has a __parent__, another environment. In diagrams, the parent is shown as a small pale blue circle and arrow that points to another environment. The parent is what's used to implement lexical scoping: if a name is not found in an environment, then R will look in its parent (and so on). 

You can set the parent environment by supplying an unnamed argument to `env()`. If you don't supply it, it defaults to the current environment.


```r
e2a <- env(d = 4, e = 5)
e2b <- env(e2a, a = 1, b = 2, c = 3)
```
<img src="diagrams/environments/parents.png" style="display: block; margin: auto;" />

We use the metaphor of a family to name environments relative to one another. The grandparent of an environment is the parent's parent, and the ancestors include all parent environments up to the empty environment. To save space, I typically won't draw all the ancestors; just remember whenever you see a pale blue circle, there's a parent environment somewhere.

You can find the parent of an environment with `env_parent()`:


```r
env_parent(e2b)
#> <environment: 0x7fa5b787ef60>
env_parent(e2a)
#> <environment: R_GlobalEnv>
```

\index{environments!empty}
Only one environment doesn't have a parent: the __empty__ environment. I draw the empty environment with a hollow parent environment, and where space allows I'll label it with `R_EmptyEnv`, the name R uses.


```r
e2c <- env(empty_env(), d = 4, e = 5)
e2d <- env(e2c, a = 1, b = 2, c = 3)
```
<img src="diagrams/environments/parents-empty.png" style="display: block; margin: auto;" />

You'll get an error if you try to find the parent of the empty environment:


```r
env_parent(empty_env())
#> Error: The empty environment has no parent
```

You can list all ancestors of an environment with `env_parents()`:


```r
env_parents(e2b)
#> [[1]]   <env: 0x7fa5b787ef60>
#> [[2]] $ <env: global>

env_parents(e2d)
#> [[1]]   <env: 0x7fa5b8d43d38>
#> [[2]] $ <env: empty>
```

By default, `env_parents()` continues until it hits either the global environment or the empty environment. You can control this behaviour with the `last` environment.

::: base 
Use `parent.env()` to find the parent of an environment. No base function returns all ancestors.
:::

### Getting and setting

You can get and set elements of an environment with `$` and `[[` in the same way as a list:


```r
e3 <- env(x = 1, y = 2)
e3$x
#> [1] 1
e3$z <- 3
e3[["z"]]
#> [1] 3
```

But you can't use `[[` with numeric indices, and you can't use `[`:


```r
e3[[1]]
#> Error in e3[[1]]:
#>   wrong arguments for subsetting an environment

e3[c("x", "y")]
#> Error in e3[c("x", "y")]:
#>   object of type 'environment' is not subsettable
```

`$` and `[[` will return `NULL` if the binding doesn't exist. Use `env_get()` if you want an error:


```r
e3$xyz
#> NULL

env_get(e3, "xyz")
#> Error in env_get(e3, "xyz"):
#>   object 'xyz' not found
```

If you want to use a default value if the binding doesn't exist, you can use the `default` argument.


```r
env_get(e3, "xyz", default = NA)
#> [1] NA
```

There are two other ways to add bindings to an environment: 

*   `env_poke()`[^poke] takes a name (as string) and a value:

    
    ```r
    env_poke(e3, "a", 100)
    e3$a
    #> [1] 100
    ```

    [^poke]: You might wonder why rlang has `env_poke()` instead of `env_set()`. 
    This is for consistency: `_set()` functions return a modified copy; 
    `_poke()` functions modify in place.

*   `env_bind()` allows you to bind multiple values: 

    
    ```r
    env_bind(e3, a = 10, b = 20)
    env_names(e3)
    #> [1] "x" "y" "z" "a" "b"
    ```

You can determine if an environment has a binding with `env_has()`:


```r
env_has(e3, "a")
#>    a 
#> TRUE
```

Unlike lists, setting an element to `NULL` does not remove it. Instead, use `env_unbind()`:


```r
e3$a <- NULL
env_has(e3, "a")
#>    a 
#> TRUE

env_unbind(e3, "a")
env_has(e3, "a")
#>     a 
#> FALSE
```

Unbinding a name doesn't delete the object. That's the job of the garbage collector, which automatically removes objects with no names binding to them. This process is described in more detail in [GC](#gc).

::: base
\indexc{rm()}\indexc{assign()}\indexc{get()}\indexc{exists()}
See `get()`, `assign()`, `exists()`, and `rm()`. These are designed interactively for use with the current environment, so working with other environments is a little clunky. Also beware the `inherits` argument: it defaults to `TRUE` meaning that the base equivalents will inspect the supplied environment and all its ancestors.
:::

### Finalisers

[Add something once rlang has an API. Also mention in data structures below]{.todo}

### Advanced bindings
\index{bindings!delayed} \index{bindings!active}

There are two more exotic variants of `env_bind()`:

*   `env_bind_exprs()` creates __delayed bindings__, which are evaluated the
    first time they are accessed. Behind the scenes, delayed bindings create 
    promises, so behave in the same way as function arguments.

    
    ```r
    env_bind_exprs(current_env(), b = {Sys.sleep(1); 1})
    
    system.time(print(b))
    #> [1] 1
    #>    user  system elapsed 
    #>       0       0       1
    system.time(print(b))
    #> [1] 1
    #>    user  system elapsed 
    #>   0.001   0.000   0.000
    ```

    Delayed bindings are used to implement `autoload()`, which makes R behave 
    as if the package data is in memory, even though it's only loaded from 
    disk when you ask for it.
  
*   `env_bind_fns()` creates __active bindings__ which are re-computed every 
    time they're accessed:

    
    ```r
    env_bind_fns(current_env(), z1 = function(val) runif(1))
    
    z1
    #> [1] 0.0808
    z1
    #> [1] 0.834
    ```

    The argument to the function allows you to also override behaviour when
    the variable is set:
    
    
    ```r
    env_bind_fns(current_env(), z2 = function(val) {
      if (missing(val)) {
        2
      } else {
         stop("Don't touch z2!", call. = FALSE)
      }
    })
    
    z2
    #> [1] 2
    z2 <- 3
    #> Error: Don't touch z2!
    ```

::: base
See  `?delayedAssign()` and `?makeActiveBinding()`.
:::

### Exercises

1.  List three ways in which an environment differs from a list.

1.  Create an environment as illustrated by this picture.

    <img src="diagrams/environments/recursive-1.png" style="display: block; margin: auto;" />

1.  Create a pair of environments as illustrated by this picture.

    <img src="diagrams/environments/recursive-2.png" style="display: block; margin: auto;" />

1.  Explain why `e[[1]]` and `e[c("a", "b")]` don't make sense when `e` is
    an environment.

1.  Create a version of `env_poke()` that will only bind new names, never 
    re-bind old names. Some programming languages only do this, and are known 
    as [single assignment languages][single assignment].

## Recursing over environments {#env-recursion}
\index{recursion!over environments}

If you want to operate on every ancestor of an environment, it's often convenient to write a recursive function. This section shows you how, applying your new knowledge of environments to write a function that given a name, finds the environment `where()` that name is defined, using R's regular scoping rules. 

The definition of `where()` is straightforward. It has two arguments: the name to look for (as a string), and the environment in which to start the search. (We'll learn why `caller_env()` is a good default in [calling environments](#calling-environments).)


```r
where <- function(name, env = caller_env()) {
  if (identical(env, empty_env())) {
    # Base case
    stop("Can't find ", name, call. = FALSE)
  } else if (env_has(env, name)) {
    # Success case
    env
  } else {
    # Recursive case
    where(name, env_parent(env))
  }
}
```

There are three cases:

* The base case: we've reached the empty environment and haven't found the
  binding. We can't go any further, so we throw an error. 

* The successful case: the name exists in this environment, so we return the
  environment.

* The recursive case: the name was not found in this environment, so try the 
  parent.

These three cases are illustrated with these three examples:


```r
where("yyy")
#> Error: Can't find yyy

x <- 5
where("x")
#> <environment: R_GlobalEnv>

where("mean")
#> <environment: base>
```

It might help to see a picture. Imagine you have two environments, as in the following code and diagram:


```r
e4a <- env(empty_env(), a = 1, b = 2)
e4b <- env(e4a, x = 10, a = 11)
```
<img src="diagrams/environments/where-ex.png" style="display: block; margin: auto;" />

* `where(a, e4a)` will find `a` in `e4a`.

* `where("b", e4a)` doesn't find `b` in `e4a`, so it looks in its parent, `e4b`,
  and finds it there.

* `where("c", e4a)` looks in `e4a`, then `e4b`, then hits the empty environment
  and throws an error.

It's natural to work with environments recursively, so `where()` provides a useful template. Removing the specifics of `where()` shows the structure more clearly:


```r
f <- function(..., env = caller_env()) {
  if (identical(env, empty_env())) {
    # base case
  } else if (success) {
    # success case
  } else {
    # recursive case
    f(..., env = env_parent(env))
  }
}
```

::: sidebar
### Iteration vs recursion {-}

It's possible to use a loop instead of recursion. I think it's harder to understand than the recursive version, but I include it because you might find it easier to see what's happening if you haven't written many recursive functions.


```r
f2 <- function(..., env = caller_env()) {
  while (!identical(env, empty_env())) {
    if (success) {
      # success case
      return()
    }
    # inspect parent
    env <- env_parent(env)
  }

  # base case
}
```
:::

### Exercises

1.  Modify `where()` to return _all_ environments that contain a binding for
    `name`. Carefully think through what type of object the function will
    need to return.

1.  Write a function called `fget()` that finds only function objects. It 
    should have two arguments, `name` and `env`, and should obey the regular 
    scoping rules for functions: if there's an object with a matching name 
    that's not a function, look in the parent. For an added challenge, also 
    add an `inherits` argument which controls whether the function recurses up 
    the parents or only looks in one environment.

## Special environments {#function-envs}
\index{functions!environments}
 
Most environments are not created by you (e.g. with `env()`) but are instead created by R. In this section, you'll learn about the most important environments, starting with the package environments. You'll then learn about the function environment bound to the function when it is created, and the (usually) ephemeral execution environment created every time the function is called. Finally, you'll see how the package and function environments interact to support namespaces, which ensure that a package always behaves the same way, regardless of what other packages the user has loaded.

### Package environments and the search path
\indexc{search()} \index{search path}

Each package attached by `library()` or `require()` becomes one of the parents of the global environment. The immediate parent of the global environment is the last package you attached[^attach]:

[^attach]: Note the difference between attached and loaded. A package is loaded automatically if you access one of its functions using `::`; it is only __attached__ to the search path by `library()` or `require()`.


```r
env_parent(global_env())
#> <environment: package:rlang>
#> attr(,"name")
#> [1] "package:rlang"
#> attr(,"path")
#> [1] "/Library/Frameworks/R.framework/Versions/3.5/Resources/library/rlang"
```

And the parent of that package is the second to last package you attached:


```r
env_parent(env_parent(global_env()))
#> <environment: package:stats>
#> attr(,"name")
#> [1] "package:stats"
#> attr(,"path")
#> [1] "/Library/Frameworks/R.framework/Versions/3.5/Resources/library/stats"
```

If you follow all the parents back, you see the order in which every package has been attached. This is known as the __search path__ because all objects in these environments can be found from the top-level interactive workspace. 


```r
search_envs()
#>  [[1]] $ <env: global>
#>  [[2]] $ <env: package:rlang>
#>  [[3]] $ <env: package:stats>
#>  [[4]] $ <env: package:graphics>
#>  [[5]] $ <env: package:grDevices>
#>  [[6]] $ <env: package:utils>
#>  [[7]] $ <env: package:datasets>
#>  [[8]] $ <env: package:methods>
#>  [[9]] $ <env: Autoloads>
#> [[10]] $ <env: base>
```

:::base 
You can access the names of the environments on the search path with `search()`
:::

The last two environments on the search path are always the same:

* The `Autoloads` environment uses delayed bindings to save memory by only 
  loading package objects (like big datasets) when needed. 
  
* The base environment, `package:base` or sometimes just `base`, is the
  environment of the base package. It is special because it has to be able 
  to bootstrap the loading of all other packages. You can access it directly 
  with `base_env()`.

Graphically, the search path looks like this:

<img src="diagrams/environments/search-path.png" style="display: block; margin: auto;" />

When you attach another package with `library()`, the parent environment of the global environment changes:

<img src="diagrams/environments/search-path-2.png" style="display: block; margin: auto;" />

### The function environment
\index{environments!function}

A function binds the current environment when it is created. This is called the __function environment__, and is used for lexical scoping. Across computer languages, functions that capture their environments are called __closures__, which is why this term is often used interchangeably with function in R's documentation.

You can get the function environment with `fn_env()`: 


```r
y <- 1
f <- function(x) x + y
fn_env(f)
#> <environment: R_GlobalEnv>
```

::: base 
Use `environment(f)` to access the environment of function `f`.
:::

In diagrams, I'll depict functions as rectangles with a rounded end that binds an environment. 

<img src="diagrams/environments/binding.png" style="display: block; margin: auto;" />

In this case, `f()` binds the environment that binds the name `f` to the function. But that's not always the case: in the following example `g` is bound in a new environment `e`, but `g()` binds the global environment. The distinction between binding and being bound by is subtle but important; the difference is how we find `g` vs. how `g` finds its variables.


```r
e <- env()
e$g <- function() 1
```

<img src="diagrams/environments/binding-2.png" style="display: block; margin: auto;" />


### Namespaces
\index{namespaces}

In the diagram above, you saw that the parent environment of a package varies based on what other packages have been loaded. This seems worrying: doesn't that mean that the package will find different functions if packages are loaded in a different order? The goal of __namespaces__ is to make sure that this does not happen, and that every package works the same way regardless of what packages are attached by the user. 

For example, take `sd()`:


```r
sd
#> function (x, na.rm = FALSE) 
#> sqrt(var(if (is.vector(x) || is.factor(x)) x else as.double(x), 
#>     na.rm = na.rm))
#> <bytecode: 0x7fa5b425bff8>
#> <environment: namespace:stats>
```

`sd()` is defined in terms of `var()`, so you might worry that the result of `sd()` would be affected by any function called `var()` either in the global environment, or in one of the other attached packages. R avoids this problem by taking advantage of the function vs. binding environment described above. Every function in a package is associated with a pair of environments: the package environment, which you learned about earlier, and the __namespace__ environment. 

*   The package environment is the external interface to the package. It's how 
    you, the R user, find a function in an attached package or with `::`. Its 
    parent is determined by search path, i.e. the order in which packages have 
    been attached. 

*   The namespace environment is the internal interface to the package. The 
    package environment controls how we find the function; the namespace 
    controls how the function finds its variables. 

Every binding in the package environment is also found in the namespace environment; this ensures every function can use every other function in the package. But some bindings only occur in the namespace environment. These are known as internal or non-exported objects, which make it possible to hide internal implementation details from the user.

<img src="diagrams/environments/namespace-bind.png" style="display: block; margin: auto;" />

Every namespace environment has the same set of ancestors:

* Each namespace has an __imports__ environment that contains bindings to all 
  the functions used by the package. The imports environment is controlled by 
  the package developer with the `NAMESPACE` file.

* Explicitly importing every base function would be tiresome, so the parent
  of the imports environment is the base __namespace__. The base namespace 
  contains the same bindings as the base environment, but it has different
  parent.
  
* The parent of the base namespace is the global environment. This means that 
  if a binding isn't defined in the imports environment the package will look
  for it in the usual way. This is usually a bad idea (because it makes code
  depend on other loaded packages), so `R CMD check` automatically warns about
  such code. It is needed primarily for historical reasons, particularly due 
  to how S3 method dispatch works.

<img src="diagrams/environments/namespace-env.png" style="display: block; margin: auto;" />

Putting all these diagrams together we get:

<img src="diagrams/environments/namespace.png" style="display: block; margin: auto;" />

So when `sd()` looks for the value of `var` it always finds it in a sequence of environments determined by the package developer, but not by the package user. This ensures that package code always works the same way regardless of what packages have been attached by the user.

Note that there's no direct link between the package and namespace environments; the link is defined by the function environments.

### Execution environments

The last important topic we need to cover is the __execution__ environment. What will the following function return the first time it's run? What about the second? \index{environments!execution}


```r
g <- function(x) {
  if (!env_has(current_env(), "a")) {
    message("Defining a")
    a <- 1
  } else {
    a <- a + 1
  }
  a
}
```

Think about it for a moment before you read on.


```r
g(10)
#> Defining a
#> [1] 1
g(10)
#> Defining a
#> [1] 1
```

This function returns the same value every time because of the fresh start principle, described in [a fresh start](#fresh-start). Each time a function is called, a new environment is created to host execution. This is called the execution environment, and its parent is the function environment. Let's illustrate that process with a simpler function. I'll draw execution environments with an indirect parent; the parent environment is found via the function environment.


```r
h <- function(x) {
  # 1.
  a <- 2 # 2.
  x + a
}
y <- h(1) # 3.
```

<img src="diagrams/environments/execution.png" style="display: block; margin: auto;" />

An execution environment is usually ephemeral; once the function has completed, the environment will be GC'd. There are several ways to make it stay around for longer. The first is to explicitly return it:


```r
h2 <- function(x) {
  a <- x * 2
  current_env()
}

e <- h2(x = 10)
env_print(e)
#> <environment: 0x7fa5b9cfcf10>
#>   parent: <environment: global>
#>   bindings:
#>    * a: <dbl>
#>    * x: <dbl>
fn_env(h2)
#> <environment: R_GlobalEnv>
```
\index{closures!environment}

Another way to capture it is to return an object with a binding to that environment, like a function. The following example illustrates that idea with a function factory, `plus()`. We use that factory to create a function called `plus_one()`. 

There's a lot going on in the diagram because the enclosing environment of `plus_one()` is the execution environment of `plus()`. 


```r
plus <- function(x) {
  function(y) x + y
}

plus_one <- plus(1)
plus_one
#> function(y) x + y
#> <environment: 0x7fa5b9e28e00>
```

<img src="diagrams/environments/closure.png" style="display: block; margin: auto;" />

What happens when we call `plus_one()`? Its execution environment will have the captured execution environment of `plus()` as its parent:


```r
plus_one(2)
#> [1] 3
```

<img src="diagrams/environments/closure-call.png" style="display: block; margin: auto;" />

You'll learn more about function factories in [functional programming](#functional-programming).

### Exercises

1.  How is `search_envs()` different to `env_parents(global_env())`?

1.  Draw a diagram that shows the enclosing environments of this function:
    
    
    ```r
    f1 <- function(x1) {
      f2 <- function(x2) {
        f3 <- function(x3) {
          x1 + x2 + x3
        }
        f3(3)
      }
      f2(2)
    }
    f1(1)
    ```

1.  Write an enhanced version of `str()` that provides more information 
    about functions. Show where the function was found and what environment 
    it was defined in.

## The call stack
\index{environments!calling}
\index{scoping!dynamic} 
\index{dynamic scoping}

There is one last environment we need to explain, the __caller__ environment, accessed with `rlang::caller_env()`. This provides the environment from which the function was called, and hence varies based on how the function is called, not how the function was created. As we saw above this is a useful default whenever you write a function that takes an environment as an argument. 

::: base
`parent.frame()` is equivalent to `caller_env()`; just note that it returns an environment, not a frame.
::: 

To fully understand the caller environment we need to discuss two related concepts: the __call stack__, which is made up of __frames__. Executing a function creates two types of context. You've learned about one already: the execution environment is a child of the function environment, which is determined by where the function was created. There's another type of context created by where the function was called: this is called the call stack.

There are also a couple of small wrinkles when it comes to custom evaluation. See [environments vs. frames](#eval-frame) for more details.

### Simple call stacks

Let's illustrate this with a simple sequence of calls: `f()` calls `g()` calls `h()`.


```r
f <- function(x) {
  g(x = 2)
}
g <- function(x) {
  h(x = 3)
}
h <- function(x) {
  stop()
}
```

The way you most commonly see a call stack in R is by looking at the `traceback()` after an error has occured:


```r
f(x = 1)
#> Error:
traceback()
#> 4: stop()
#> 3: h(x = 3) 
#> 2: g(x = 2)
#> 1: f(x = 1)
```

Instead of `stop()` + `traceback()` to understand the call stack, we're going to use `lobstr::cst()` to print out the **c**all **s**tack **t**ree:


```r
h <- function(x) {
  lobstr::cst()
}
f(x = 1)
#> █
#> └─f(x = 1)
#>   └─g(x = 2)
#>     └─h(x = 3)
#>       └─lobstr::cst()
```

This shows us that `cst()` was called from `h()`, which was called from `g()`, which was called from `f()`. Note that the order is the opposite from `traceback()`. As the call stacks get more compliated, I think it's easier to understand the sequence of calls if you start from the beginning, rather than the end (i.e. `f()` calls `g()`; rather than `g()` was called by `f()`).

### Lazy evaluation

The call stack above is simple - while you get a hint that there's some tree-like structure involved, everything happens on a single branch. This is typical of a call stack when all arguments are eagerly evaluated. 

Let's create a more complicated example that involves some lazy evaluation. We'll create a sequence of functions, `a()`, `b()`, `c()`, that pass along an argument `x`.


```r
a <- function(x) b(x)
b <- function(x) c(x)
c <- function(x) x

a(f())
#> █
#> ├─a(f())
#> │ └─b(x)
#> │   └─c(x)
#> └─f()
#>   └─g(x = 2)
#>     └─h(x = 3)
#>       └─lobstr::cst()
```

`x` is lazily evaluated so this tree gets two branches. In the first branch `a()` calls `b()`, then `b()` calls `c()`. The second branch starts when `c()` evaluates its argument `x`. This argument is evaluated in a new branch because the environment in which it is evaluated is the global environment, not the environment of `c()`.

### Frames

Each element of the call stack is a __frame__[^frame], also known as an evaluation context.
The frame is an extremely important internal data structure, and R code can only access a small part of the data structure because it's so critical. A frame has three main components that are accessible from R:

* An expression (labelled with `expr`) giving the function call. This is
  what `traceback()` prints out.

* An environment (labelled with `env`), which is typically the execution 
  environment of a function. There are two main exceptions: the environment of 
  the global frame is the global environment, and calling `eval()` also 
  generates frames, where the environment can be anything.

* A parent, the previous call in the call stack (shown by a grey arrow). 

[^frame]: NB: `?environment` uses frame in a different sense: "Environments consist of a _frame_, or collection of named objects, and a pointer to an enclosing environment.". We avoid this sense of frame, which comes from S, because it's very specific and not widely used in base R. For example, the "frame" in `parent.frame()` is an execution context, not a collection of named objects.

<img src="diagrams/environments/calling.png" style="display: block; margin: auto;" />

(To focus on the calling environments, I have omitted the bindings in the global environment from `f`, `g`, and `h` to the respective function objects.)

The frame also holds exit handlers created with `on.exit()`, restarts and handlers for the condition system, and which context to `return()` to when a function completes. These are important for the internal operation of R, but are not directly accessible.

### Dynamic scope

Looking up variables in the calling stack rather than in the enclosing environment is called __dynamic scoping__. Few languages implement dynamic scoping (Emacs Lisp is a [notable exception](http://www.gnu.org/software/emacs/emacs-paper.html#SEC15).) This is because dynamic scoping makes it much harder to reason about how a function operates: not only do you need to know how it was defined, you also need to know the context in which it was called. Dynamic scoping is primarily useful for developing functions that aid interactive data analysis. It is one of the topics discussed in [non-standard evaluation](#nse). 

### Exercises

1.  Write a function that lists all the variables defined in the environment
    in which it was called. It should return the same results as `ls()`.

## As data structures {#explicit-envs}

As well as powering scoping, environments are also useful data structures in their own right because they have reference semantics.  There are three common problems that they can help solve:

*   __Avoiding copies of large data__. Since environments have reference semantics, 
    you'll never accidentally create a copy. This makes it a useful vessel for 
    large objects. Bare environments are not that pleasant to work with;
    I recommend using R6 objects instead. Learn more in [R6].
  
*   __Managing state within a package__. Explicit environments are useful in 
    packages because they allow you to maintain state across function calls. 
    Normally, objects in a package are locked, so you can't modify them 
    directly. Instead, you can do something like this:

    
    ```r
    my_env <- new.env(parent = emptyenv())
    my_env$a <- 1
    
    get_a <- function() {
      my_env$a
    }
    set_a <- function(value) {
      old <- my_env$a
      my_env$a <- value
      invisible(old)
    }
    ```

    Returning the old value from setter functions is a good pattern because 
    it makes it easier to reset the previous value in conjunction with 
    `on.exit()` (see more in [on exit](#on-exit)).

*   __As a hashmap__. A hashmap is a data structure that takes constant, O(1), 
    time to find an object based on its name. Environments provide this 
    behaviour by default, so can be used to simulate a hashmap. See the 
    CRAN package hash for a complete development of this idea. 
    \index{hashmaps} \index{dictionaries}

## `<<-`

The ancestors of an environment have an important relationship to `<<-`. The regular assignment arrow, `<-`, always creates a variable in the current environment. The deep assignment arrow, `<<-`, never creates a variable in the current environment, but instead modifies an existing variable found by walking up the parent environments. 


```r
x <- 0
f <- function() {
  x <<- 1
}
f()
x
#> [1] 1
```

If `<<-` doesn't find an existing variable, it will create one in the global environment. This is usually undesirable, because global variables introduce non-obvious dependencies between functions. `<<-` is most often used in conjunction with a closure, as described in [Closures](#closures).

### Exercises

1.  What does this function do? How does it differ from `<<-` and why
    might you prefer it?
    
    
    ```r
    rebind <- function(name, value, env = caller_env()) {
      if (identical(env, empty_env())) {
        stop("Can't find `", name, "`", call. = FALSE)
      } else if (env_has(env, name)) {
        env_poke(env, name, value)
      } else {
        rebind(name, value, env_parent(env))
      }
    }
    rebind("a", 10)
    #> Error: Can't find `a`
    a <- 5
    rebind("a", 10)
    a
    #> [1] 10
    ```


## Quiz answers {#env-answers}

1.  There are four ways: every object in an environment must have a name;
    order doesn't matter; environments have parents; environments have
    reference semantics.
   
1.  The parent of the global environment is the last package that you
    loaded. The only environment that doesn't have a parent is the empty
    environment.
    
1.  The enclosing environment of a function is the environment where it
    was created. It determines where a function looks for variables.
    
1.  Use `caller_env()` or `parent.frame()`.

1.  `<-` always creates a binding in the current environment; `<<-`
    rebinds an existing name in a parent of the current environment.

[single assignment]:http://en.wikipedia.org/wiki/Assignment_(computer_science)#Single_assignment