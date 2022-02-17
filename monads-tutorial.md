---
label: Monads Tutorial
icon: #file
author:
  name: #Antonio Hernandez
  email: #contacto@antoniohernandez.mx
order: -3
---

# Monads tutorial

As part of Lecture 4, Lars gave a pretty good introduction to the concept of
*Monads*.

## First examples

Consider

```haskell
run :: IO ()
run = putStrLn "Hello, world!"
```

`run` is of type IO taking `()` (read as "unit") as argument.  Think of `run`
as being of type IO *over* ().  What this means is that

- it returns *unit* (basically *nothing of value*)
- before returning *unit*, it executes an IO (*input-output*) operation.

The input-ouput operation is a "side-effect".  Ironically, while this manner
of speaking suggests that the IO operation is of lesser importance, it is via
side effects that Haskell performs useful operations.

In this case, `run` performs the side-effect of printing "Hello, world!" on
the monitor.  Never the less, the result of the operation is `()`.

It turns out that `IO` is our first example of *Monad*.  We will see later
what this means.  The type signature of `putStrLn` is

`putStrLn :: String -> IO ()`

which means that putStrLn is a *function that takes a string and returns a
Monad over unit*.

Another function that returns an "IO over something" is `getLine`.  Its type
signature is

`getLine :: IO String`

What `getLine` does,is it performs an IO operation and then returns a string.
In fact, the IO operation it performs is to wait for the user to type
something and then construct a string based on this typing.

## Input-Output (IO)

Asking the REPL for `:i IO` we see that IO is an Applicative, a Functor, a
Monad, and other things.  Asking REPL what a functor is,

```console
Prelude> :i Functor
type Functor :: (* -> *) -> Constraint
class Functor f where
  fmap :: (a -> b) -> f a -> f b
```

Here `f` represents some structure (could be a Monad, but not necessarily) and 
`f a`, which I will read "f over a", represents a structure `f` made out of
expressions of type `a`.  The structure `f` is a functor if has instances of
two laws, `fmap` and `(<$)`. `fmap` is a generalization of `map`, but now
works on more general structures than lists:  `fmap` applies the function `(a
-> b)` to every element inside the structure `f`.

To illustrate, let us compose `map`

```console
Prelude Data.Char> :t map
map :: (a -> b) -> [a] -> [b]
```

with `toUpper`,

```console
Prelude> import Data.Char
Prelude Data.Char> :t toUpper
toUpper :: Char -> Char
Prelude Data.Char> toUpper 'q'
'Q'
```

to define the function `map toUpper`

```console
Prelude Data.Char> :t map toUpper
map toUpper :: [Char] -> [Char]
```

We now construct an example where the functor `f` is `getLine`

```console
Prelude Data.Char> :t getLine
getLine :: IO String
```

so the type signature of `fmap (a -> b) -> f a` is

```console
Prelude Data.Char> :t fmap (map toUpper) getLine
fmap (map toUpper) getLine :: IO [Char]
```

That is to say, `f b` is `IO String`.

In practical terms, `fmap (map toUpper)` takes the string that the IO
opperation performed by `getLine` gets from the user at the console,
and returns the whole thing as if the user had typed her input in all caps.

```console
Prelude Data.Char> fmap (map toUpper) getLine
Haskell is nice
"HASKELL IS NICE"
```

[TO BE CONTINUED...]