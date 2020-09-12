# Haskell Exercises - State Monad implementations

- To learn Monads I've followed these steps 

    1. Implement an alias of a state transformer function - see `SimpleState`.
    2. After that, replace the alias with a constructor and add the accessor function to apply the state transformer out of the monad - see `State`. Only `runState` needs to be exported now.
    3. Finally, create the Monad instance for State, reusing the bind and the return functions. This State monad can be used in a do block - see `State`.StateMonad`
    4. Implement a state monad with a transactional state inside that provides commit/rollback actions on the state - `see TransactionalStateMonad`.

- IMPORTANT: Monad formal definition available [here](./Monad.md)

# How to run

Run:

- Compile and build executables: `stack build` 

- Compile and run all tests: `stack test` 

- List the tests you can run individually: `stack test --dry-run` 

- Run an individual test: `stack test --test-arguments "--match=SimpleState"` 
