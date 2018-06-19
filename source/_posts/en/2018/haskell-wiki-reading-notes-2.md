---
title: Haskell wiki reading notes 2 - Elementary
date: 2018-06-18
tags:
- haskell
- notes
---
# Recursion

## Numeric recursion

```haskell
factorial 0 = 1
factorial n = n * factorial (n - 1)

-- declared in one line, use ; to seperate
let factorial 0 = 1; factorial n = n \* factorial (n - 1)
```

```haskell
factorial n = product [1..n]
```

Haskell decides which function definition to use by starting at the top. So, **always list multiple function definitions starting with the most specific**.

## Loops

Translate a loop into recursive form by making loop variables into an argument.

```haskell
factorial n = go n 1
  where
  go n res
    | n > 1 = go (n - 1) (res * n)
    | otherwise = res
```

## List based recursion

```haskell
length :: [a] -> Int
length [] = 0
length (x:xs) = 1 + length xs

(++) :: [a] -> [a] -> [a]
[] ++ ys     = ys
(x:xs) ++ ys = x : xs ++ ys
```

# List II

## Currying

All functions take only one argument.

## `map`

```haskell
map :: (a -> b) -> [a] -> [b]
map _ [] = []
map f (x:xs) = (f x) : map f xs
```

## Tricks

- Dot dot notation
```haskell
[1..4] --= [1,2,3,4]
[2,4..8] --= [2,4,6,8]
[5,4..1] --= [5,4,3,2,1]
[1,3..8] --= [1,3,5,7]
```

- Infinite lists
```haskell
[1..]
```

# Lists III
## Folds

- `foldr`
```haskell
foldr :: (a -> b -> b) -> b -> [a] -> b
foldr f acc [] = acc
foldr f acc (x:xs) = f x (foldr f acc xs)
```

```
  :                         f
 / \                       / \
a   :       foldr f acc   a   f
   / \    ------------->     / \
  b   :                     b   f
     / \                       / \
    c  []                     c   acc
```

- `foldl`
```haskell
foldl :: (a -> b -> a) -> a -> [b] -> a
foldl f acc []     =  acc
foldl f acc (x:xs) =  foldl f (f acc x) xs
```

```
  :                            f
 / \                          / \
a   :       foldl f acc      f   c
   / \    ------------->    / \
  b   :                    f   b
     / \                  / \
    c  []                acc a
```

> `foldl` is *tail-recursive*, the compiler will optimise it to a loop. Becase of laziness, the calls to `f` will be left unevaluated, thus causing running tou of memory.

- `foldl'`: forces the evaluation of `f` immediately at each step.

> Use `foldr` on lists that might be infinite or where the fold is building up a data structure. Use `foldl'` if the list is known to be finite and comes down to a single value.

- `foldr1` & `foldl1`: taking the first element as default accumulator.

   Types have to be the same, and an empty list is an error.


## Folds and Laziness

```haskell
echoes = foldr (\ x xs -> (replicate x x) ++ xs) []
take 10 (echoes [1..])     -- [1,2,2,3,3,3,4,4,4,4]

echoes = foldl (\ xs x -> xs ++ (replicate x x)) []
take 10 (echoes [1..])     -- not terminating
```

## Scans

Accumulates a values like a fold, and returns a list of all the intermediate values.

```haskell
scanl :: (a -> b -> a) -> a -> [b] -> [a]
```

## Filter

Generates a new list composed of elements that meet a condition.

```haskell
filter :: (a -> Bool) -> [a] -> [a]
```

## List comprehensions

```haskell
retainEven es = [n | n <- es, isEven n, ...condition]
{-
1. n <- es: take the list es and draw each element as value n.
2. isEven n: for each drawn n test the boolean condition.
3. n |: append n to the new list being created.
-}

firstForEvenSeconds ps = [x | (x, y) <- ps, isEven y]
allPairs = [(x, y) | x <- [1..4], y <- [5..8]]
```

# Type declarations

## `data` and constructor functions

```haskell
data Anniversary = Birthday String Int Int Int
  | Wedding String String Int Int Int
```

Type names and constructor functions must start with captial letters.

## Deconstructing types

```haskell
showAnniversary :: Anniversary -> String
showAnniversary (Birthday name year month day) =
   name ++ " born " ++ showDate year month day
showAnniversary (Wedding name1 name2 year month day) =
   name1 ++ " married " ++ name2 ++ " on " ++ showDate year month day
```

## Type synonyms

```haskell
type Name = String
```

# Pattern matching

In pattern matching, we attempte to **match** values against **patterns** and **bind** variables to successful matches.

- Constructors are allowed in patterns.
- `[]` & `(:)` are constructors of the list.
- tuple constructors: `(,)`, `(,,)`...
```haskell
(,) 5 3 --> (5,3)
(,,) 1 2 3 --> (1,2,3)
```

## Matching literal values

```haskell
g :: [Int] -> Bool
g (0:[]) = False
g (0:xs) = True
g _ = False
```

## Syntax tricks

### As-patterns

Binds a name to the whold value being matched: `var@pattern`.

```haskell
contrivedMap :: ([a] -> a -> b) -> [a] -> [b]
contrivedMap f [] = []
contrivedMap f list@(x:xs) = f list x : contrivedMap f xs
```

### Records

```haskell
data Foo2 = Bar2 | Baz2 {bazNumber::Int, bazName::String}

h :: Foo2 -> Int
h Baz2 {bazName=name} = length name
h Bar2 {} = 0

x = Baz2 1 "Haskell"     -- construct by declaration order, try ":t Baz2" in GHCi
y = Baz2 {bazName = "Curry", bazNumber = 2} -- construct by name

h x -- 7
h y -- 5
```

Also, the `{}` pattern can be used for matching a constructor regardless of the datatype elements

```haskell
data Foo = Bar | Baz Int
g :: Foo -> Bool
g Bar {} = True
g Baz {} = False
```

## Where pattern matching can be used

- Equations
- `let` & `where`
- Lambda asbstractions
- List comprehensions
```hakell
data Maybe a = Nothing | Just a

catMaybes :: [Maybe a] -> [a]
catMaybes ms = [ x | Just x <- ms ]
```
- `do` blocks

# Control structures

## `if`
```haskell
if <condition> then <true-value> else <false-value>
```

`if` is an expression, and `else`is mandatory.

`if` can be embedding:
```haskell
g x y = (if x == 0 then 1 else sin x / x) * y
```

## Guards
```haskell
f (pattern1) | predicate1 = w
             | predicate2 = x
             | otherwise = z
```

`otherwise` is jst an alias to `True`.


## `case`

```hasekll
f x =
    case x of
        0 -> 18
        1 -> 15
        2 -> 12
        _ -> 12 - x
```

## Controlling actions

```haskell
doGuessing num = do
   putStrLn "Enter your guess:"
   guess <- getLine
   if (read guess) < num
     then do putStrLn "Too low!"
             doGuessing num
     else if (read guess) > num
            then do putStrLn "Too high!"
                    doGuessing num
            else do putStrLn "You Win!"

doGuessing num = do
  putStrLn "Enter your guess:"
  guess <- getLine
  case compare (read guess) num of
    LT -> do putStrLn "Too low!"
             doGuessing num
    GT -> do putStrLn "Too high!"
             doGuessing num
    -- the dos after the ->s are necessary because of sequencing actions.
    EQ -> putStrLn "You Win!"
```

> A note about `return`
>
> `return` is not equivalent to other language, `return ()` evaluates to an action which does nothing.

# More on functions

## `let` and `where`

```haskell
addStr :: Float -> String -> Float
addStr x str = x + read str

sumStr :: [String] -> Float
sumStr = foldl addStr 0.0

-- with let
sumStr =
   let addStr x str = x + read str
   in foldl addStr 0.0

-- with where
sumStr = foldl addStr 0.0
   where addStr x str = x + read str
```

`let` is an expression, `where` are like guards and are not expressions. `let` bindings can be used within complex expressions.

`where` can be incorporated into `case` expressions and guards:
```haskell
describeColour c =
   "This colour "
   ++ case c of
          Black -> "is black"
          White -> "is white"
          RGB red green blue -> " has an average of the components of " ++ show av
             -- only available for RGB
             where av = (red + green + blue) `div` 3
   ++ ", yeah?"

doStuff :: Int -> String
doStuff x
  | x < 3     = report "less than three"
  | otherwise = report "normal"
  where
    report y = "the input is " ++ y
```

## Lambdas

```haskell
sumStr = foldl (\ x str -> x + read str) 0.0
```

## Operators

All function that takes two arguments and has a name consisting entirely of non-alphanumeric characters is considered an operator.

```haskell
2 + 4
(+) 2 4
```

```haskell
(\\) :: (Eq a) => [a] -> [a] -> [a]
xs \\ ys = foldl (\zs y -> delete y zs) xs ys

(\\) xs ys = foldl (\zs y -> delete y zs) xs ys
```

### Sections

```haskell
(2+) 4
(+4) 2
```

### "normal" prefix

Use function as a operator:
```haskell
elem :: (Eq a) => a -> [a] -> Bool
x `elem` xs = any (==x) xs

1 `elem` [1..4]
```

# Higher-order functions

## `Ord`

Almost all basic data types are members of the `Ord` class， which is for ordering tests.

```haskell
compare :: (Ord a) => a -> a -> Ordering
Ordering = LT | EQ | GT
```

- `->` is right-associative.

## Function manipulation

1. Flipping arguments: `flip`
```haskell
flip :: (a -> b -> c) -> b -> a -> c
```

2. Composition： `(.)`
```haskell
(.) :: (b -> c) -> (a -> b) -> a ->c
```

```haskell
-- myInits [1,2,3] -> [[], [1,2], [1,2,3]]
myInits :: [a] -> [[a]]
myInits = map reverse . scanl (flip (:)) []
```

3. Application: `($)`
```haskell
($) :: (a -> b) -> a -> b
```

- `$` has very low precedence.

```haskell
map ($ 2) [(2*), (4*), (8*)]
```

4. `curry` & `uncurry`
```haskell
curry :: ((a, b) -> c) -> a -> b -> c
uncurry :: (a -> b -> c) -> (a, b) -> c

-- the first element of a paire is applied to the second
uncurry ($) (reverse, "stressed")
```

5. `id` & `const`

```haskell
-- return its argument unchanged
id :: a -> a
-- discards the second and return the first
const :: a -> b -> a
```

# Using GHCi effectively

## Tab completion

## `:commands`

- `:h[elp]`
- `:l[oad]`
- `:r[eload]`
- `:t[ype]`
- `:m[odule]`
- `:b[rowse]`

## Timing funcitions

The basic way to measure how much a function takes to run. Run `:set + s` then run the functions that are tested.

## Multi-line input

Surrounded code with `:{` `:}`.
