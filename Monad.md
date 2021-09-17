# Monad definition 

These notes have been collected while watching [Bartosz Milewski's lecture](https://www.youtube.com/watch?v=gHiyzctYqZ0) about Monad.

Monads are algebraic structures that enable compositions of functions that produce side effects and keep the language functionally pure at the same time. For example, we can use them to simulate imperative style in a pure functional language that for its nature does not allow sequential statements. They also help to have a clear distinction, within the language, between pure functions and impure functions. 

To the compiler, a function that returns a type `b` it's not the same thing of a function that returns a type `b` that also produces a side effect `m`. 

```
b != m b
```

This distinction would be useful to check referential transparency at compile time, leading to "more correct" programs. In a language like Haskell, we can leverage the type system (and the Monad type class) to check that impure functions call pure functions but not the other way round. 



## Problem

Compose two pure functions so that, when the composition is executed, an additional action or side effect happens at the same time. 

Given the functions `f` and `g`, we can't use the simple `f . g` composition when we want to execute an additional side effect. In order to use `.` composition operator, the types have to match - ie. the type of the output of `f` must match the type of the input of `g`.  

In other words, we want to achieve this: 

Given two pure functions `f` and `g` defined below, we want the new pure function `f >=> g` that would behave as the composition of `f` and `g`. 

`f` is a function that returns `b` plus a side effect `m`. We can write this result as `m b`. As you can see, the output of `f` doesn't match the input of `g` - that's why we can't use `.`.

```haskell
(>=>) :: (a -> m b) -> (b -> m c) -> (a -> m c)
(>=>) ::      f     ->      g     ->   f >=> g     
```
Note: `f >=> g = (a -> m c)` where `a` is the input of `f` and `m c` is the output of of `g`. 


Again, where the `.` operator is used to compose functions with the type of the output of `f` that matches the type of the input
of `g`, ...
the `(>=>)` operator is used to match an "embellished" output (for example, with logging side effects) to the input of a different type for another function. 

## Solution

```haskell
(>=>) :: (a -> m b) -> (b -> m c) -> (a -> m c)
(>=>) ::      f     ->      g     ->   f >=> g     

f (>=>) g = λa -> let m b = f a 
                  in m b >>= g
```

In this step we have applied `f` on `a` and received `f a` (ie. `m b`). A new operator `>>=` has been introduced.

If we can define `>>=`, we've found a way to make this work:

```haskell
(>>=) :: m b -> (b -> m c) -> m c
(>>=) :: f a ->      g     -> m c
```

Note that `g` is a function on `b`, that is the element contained in the `m` structure of the first argument.

If `m` was a Functor - and a Monad is by definition a Functor -, we could apply `g` to `m b`, we could "get inside" `m b` to apply `(b -> m c)`, we could lift `g` over `m b`, we could `fmap` over `m b `.

```haskell
(>>=) :: m b -> (b -> m c) -> m c
                     g  
m b >>= g =      fmap g m b
m b >>= g =      fmap (b -> m c) m b
m b >>= g =      m (m c)
``` 
You can see `c` is wrapped twice in `m`, this is not matching our expected `m c`. 

We can simply fix this introducing the function `join`:

```haskell
join :: m (m b) -> m b 
join = ...
```
The implementation depends on the kind of Monad itself - see [1]

So, the final definition of `(>>=)` is ...

```haskell
(>>=) :: m b -> (b -> m c) -> m c
m b >>= g = join fmap g m b
m b >>= g = join fmap (b -> m c) m b
m b >>= g = join m (m c)
m b >>= g = m c
```
We have defined `(>>=)` and we can use it to compose monadic functions (function with side effects). 
In Haskell books the definition above is described as below, it's the same thing just after renaming `b` to `a` and `c` to `b`.
We need to add another function that, given `a` can return a monad `m a` - ie. `a` with an `m` side effect. This last function would be used as a first step of the composition chaining.

```haskell
(>>=) :: m a -> (a -> m b) -> m b 
(return) :: a -> m a
```


[1] - Monad join function - https://stackoverflow.com/questions/3382210/monad-join-function
