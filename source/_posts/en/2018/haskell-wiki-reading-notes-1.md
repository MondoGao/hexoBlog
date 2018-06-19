---
title: Haskell wiki reading notes 1 - Basic
date: 2018-04-29
tags:
- haskell
- notes
---
# Variables and functions

## ghci

- `:quit`

- `:load <file>`

## variables

- `<name> = <value>`

  `r = 5`

- Use `let var = <value>` in ghci.

- Variables are immutable and can't be define twice.

- Variables can be defined in any order because of immutable.

## comment

- `--`

- `{- ... -}`

## functions

- `<name> <parameters…> =`

```haskell
area r = pi * r ^ 2
```

- Parentheses are omit.

```haskell
area 5 * 3 -- (area 5) * 3 area (5 * 3) -- area 15
```

- Functions take precedence over all other operators

- `where`: define function-local variables

```haskell
heron a b c = sqrt (s * (s - a) * (s - b) * (s - c))
  where s = (a + b + c) / 2
```

# Truth values

## Comparions

`>\<\>=\<=`

- equal: `==`

- not equal: `/=`

## Boolean

- Bool: `True/False`

- operations

  - and: `(&&)`

  - or: `(||)`

  - `not`

## Infix operators

- Haskell allows two-argument functions to be written as infix operators placed between their arguments.

```haskell
4 + 9 == 13 --True
(==) (4 + 9) 13 -- True
```

## Guards

```haskell
absolute x
  | x < 0 = 0 - x
  | otherwise = x
```

- catch-all gurad: `otherwise = True`

- `where`

```haskell
numOfRealSolutions a b c
  | disc > 0 = 2
  | disc == 0 = 1
  | otherwise = 0
    where
    disc = b^2 - 4*a*c
```

# Type basics

## `:type` in ghci

## Strings and chars

- `'h' :: Char`

- `"hello" :: [Char]`

- `"hello" ++ "world"`

## Function types

- `not :: Bool -> Bool`

- `chr :: Int -> Char` `ord :: Char -> Int`

  conversion char to its numeric form in :module Data.Char

- `xor :: Bool -\> Bool -\> Bool`

```haskell
xor p q = (p || q) && not(p && q)`
```

## Type annotations

```haskell
uppercase, lowercase :: String -> String

uppercase = map toUpper
lowercase = map toLower
```

# Lists and tuples

## Lists

- `[1, 2, 3]`

- Elements in a list must have the same type.

- **consing**: `<newEl>:<list>`

  - Can only cons(`:`) something onto a list, not onto an element.

  - `<array> ++ <array>`

## Tuples

- Tuples have a fixed number of elements.

- Elements in tuple can have different types.

- **pairs**(2-tuples)

- **triples**(3-tuples)

## Retrieving values

- `fst :: (a, b) -> a`

- `snd :: (a, b) -> b`

- `head :: [a] -> a`

  - Calling this with a empty list will case a error, use less.

  - `a` is a **type variable**.

- `tail :: [a] -> a`

# Type basics II

## Num class

- `(+) :: (Num a) => a -> a -> a`

- Num is a **type class**: a group of types which includes all number types.

- `(Num a) =>` restricts a to a number types (instances of Num).

## Numeric types

- `Int`

- `Integer`

  supports arbitrarily large values --- at the cost of some efficiency

- `Double/Float`

## Monomorphic trouble

```haskell
(/) :: (Fractional a) => a -> a -> a
length :: (Foldable t) => t a -> Int

4 / length [1,2,3] -- throw error, Int is not a Fractional

fromIntegral :: (Integral a, Num b) => a -> b --- transform Integral to Num
4 / fromIntegral (length [1,2,3])
```

## Other classes

- `Foldable`: [] 's upper class

- `Eq`: the class of types of values can be compared for equality.

```haskell
(==) :: (Eq a) => a -> a -> Bool
```

# Next Steps

## `if/then/else`

```haskell
mySignum x =
  if x < 0
    then -1
    else if x > 0
      then 1
      else 0

-- rewrite with gurards
mySignum x
  | x < 0 = -1
  | x > 0 = 1
  | otherwise = 0
```

- Haskell requires both a then and an else clause.

## Pattern matching

```haskell
pts :: Int -> Int
pts 1 = 10
pts 2 = 6
pts 3 = 4
pts 4 = 3
pts 5 = 2
pts 6 = 1
pts _ = 0

-- rewrite: mixing style
pts :: Int -> Int
pts 1 = 10
pts 2 = 6
pts x
  | x <= 6 = 7 - x
  | otherwise = 0
```

- The argument is match against the patterns on the left side.

- `_` is the wildcard meaning whatever.

- The named parameter is like `_`, the only difference is it have a name which can be used.

```haskell
-- example: (||)
(||) :: Bool -> Bool -> Bool
False || False = False
_ || _ = True

(||) :: Bool -> Bool -> Bool
True || _ = True
False || y = y
```

## Tuple and list patterns

```haskell
fst' :: (a, b) -> a
fst' (x, _) = x
```

```haskell
head :: [a] -> a
head (x:_) = x
head [] = error "Prelude.head: empty list"

tail :: [a] -> [a]
tail (_:xs) = xs
tail [] = error "Prelude.tail: empty list"
```
## let binding

```haskell
roots a b c =
  let sdisc = sqrt (b * b - 4 * a * c)
    twice_a = 2 * a
  in ((-b + sdisc) / twice_a,
    (-b - sdisc) / twice_a)
```

# Building voabulary

## Composing functions with `(.)`.

```haskell
squareOfF x = (square . f) x -- or squareOfF = square . f
```

## Libraries

- Prelude

  the core library loaded by default in every Haskell program. 

- Import modules

  - `import Data.List` `testPermutations = permutations "Prelude"`

  - `:m +Data.List`

# Simple input and output

## Type

- `printLn :: String -> IO ()`

- `IO ()`

  an action which returns `()`

- **unit type**: `()`

## do: an suger for `>>=`

```haskell
main = do
  putStrLn "Please enter your name:"
  name <- getLine
  putStrLn ("Hello," ++ name ++ ", how are you?")
```

- do assume the block return a action.

- `<-` can't be used in the last line

- `if`'s two branches have the same type

```haskell
if condition
  then do ...
  else ...
```

- let bindings in do block can omit `in`

```haskell
main = do
  name <- getLine
  let loudName = makeLoud name
  putStrLn ("Hello" ++ loudName ++ "!")
  putStrLn ("Oh boy! Am I excited to meet you," ++ loudName)
```

- `return :: Monad m => a -> m a`

  turn a non-action return into action

- Can omit the do keyword if there's only one IO action following it.

