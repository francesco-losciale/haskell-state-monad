# Monad definition 

Notes collected watching [Bartosz Milewski's lecture](https://www.youtube.com/watch?v=gHiyzctYqZ0) about Monad.

Monads are algebraic structures used to simulate imperative programming  and execute side-effects, still mantaining the language functionally pure (state management is an example of side effect).

## Problem

Compose two pure functions so that an additional action or effect is executed at the same time. 

Given the functions `f` and `g`, we can't use the usual `f . g` composition if we want to execute an effect after `f`. In order to use `.` composition operator, the type of the output of `f` should match the type of the input of `g`.  

What we want to achieve is this: 

Given two pure functions `f` and `g` below, we want a new pure function `f >=> g` that would behave as the composition of `f` and `g`. 


```haskell
(>=>) :: (a -> m b) -> (b -> m c) -> (a -> m c)
(>=>) ::      f     ->      g     ->   f >=> g     
```
Note: `f >=> g = (a -> m c)` where `a` is the input of `f` and `m c` is the output of of `g`. 


Again, since the `.` operator is used to compose functions with the type of the output of `f` that matches the type of the input
of `g`, ...
the `(>=>)` operator is used to match an "embellished" output (for example, with logging) to the input of another function. 

## Solution

```haskell
(>=>) :: (a -> m b) -> (b -> m c) -> (a -> m c)
(>=>) ::      f     ->      g     ->   f >=> g     

f (>=>) g = λa -> let m b = f a 
                  in m b >>= g
```

In this step we have applied `f` on `a` and received `f a` (ie. `m b`). A new operator `>>=` has been introduced.

Let's define `>>=` :

```haskell
(>>=) :: m b -> (b -> m c) -> m c
(>>=) :: f a ->      g     -> m c
```

After renaming `b` to `a` and `c` to `b`, we have the generic...

```haskell
(>>=) :: m a -> (a -> m b) -> m b
                     f  
```

Note that `f` is a function on `a`, that is the element contained in the `m` structure of the first argument.

If `m` was a Functor - and a Monad is by definition a Functor -, we could apply `f` to `m a`, we could "get insisde" `m a`, we could lift `f` over `m a`, we could `fmap` over `m a `.

```haskell
(>>=) :: m a -> (a -> m b) -> m b
                     f  
m a >>= f =      fmap f m a
m a >>= f =      fmap (a -> m b) m a
m a >>= f =      m (m b)
``` 
You can see `b` is wrapped twice in `m`, this is not matching our expectation `m b`. 

We can simply fix this introducing the function `join`:

```haskell
join :: m (m a) -> m a 
join = ...
```
The implementation depends on the kind of Monad itself - see [1]

So, the final definition of `(>>=)` is ...

```haskell
(>>=) :: m a -> (a -> m b) -> m b
m a >>= f = join fmap f m a
m a >>= f = join fmap (a -> m b) m a
m a >>= f = join m (m b)
m a >>= f = m b
```

[1] - Monad join function - https://stackoverflow.com/questions/3382210/monad-join-function
