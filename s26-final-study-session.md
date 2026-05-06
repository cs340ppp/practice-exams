# CS 340 Final Exam Study Session

---

## Practice Problem 1 - State Monad

Consider the following functions:

```haskell
bumpFst :: State (Int, Int) Int
bumpFst = State $ \(a, b) -> (a, (a+1, b))

doubleSnd :: State (Int, Int) ()
doubleSnd = State $ \(a, b) -> ((), (a, b*2))

swap :: State (Int, Int) ()
swap = State $ \(a, b) -> ((), (b, a))
```

Match each expression to its value when evaluated with initial state `(3, 5)`
(i.e., `runState expr (3, 5)`). Some answers may not be used.

**Expressions:**

| #    | Expression                                        |
| ---- | ------------------------------------------------- |
| i    | `bumpFst`                                         |
| ii   | `fmap (*2) bumpFst`                               |
| iii  | `bumpFst >> bumpFst`                              |
| iv   | `bumpFst >>= \x -> return (x+10)`                 |
| v    | `do { x <- bumpFst; doubleSnd; return x }`        |
| vi   | `do { swap; x <- bumpFst; return x }`             |
| vii  | `do { x <- bumpFst; y <- bumpFst; return (x+y) }` |
| viii | `fmap (*2) (swap >> bumpFst)`                     |

**Answer pool:**

|   |                |
| - | -------------- |
| A | `(3, (4, 5))`  |
| B | `(3, (4, 10))` |
| C | `(4, (4, 5))`  |
| D | `(4, (5, 5))`  |
| E | `(5, (5, 5))`  |
| F | `(5, (6, 3))`  |
| G | `(6, (4, 5))`  |
| H | `(7, (5, 5))`  |
| I | `(10, (6, 3))` |
| J | `(13, (4, 5))` |

<details>
<summary>Answer key</summary>

i→A, ii→G, iii→D, iv→J, v→B, vi→F, vii→H, viii→I

Decoys: C `(4, (4, 5))` and E `(5, (5, 5))` - both arise from the mistake of
thinking `bumpFst` returns the *incremented* value `a+1` rather than the current
value `a`.

</details>

---

## Practice Problem 2 - Data Types and Type Classes

Consider the following mutually recursive type definitions:

```haskell
data Expr     = Lit Int | Add Expr Expr | App String ExprList
data ExprList = NoArgs | ArgCons Expr ExprList
```

An `Expr` is either an integer literal, an addition of two sub-expressions, or a
named function application with an `ExprList` of arguments. An `ExprList` is
either empty (`NoArgs`) or an `Expr` followed by another `ExprList`.

### PP2(a) - 2 points

Provide a legal value of type `Expr` that uses at least two `App` constructors.

<details>
<summary>Sample answer</summary>

```haskell
App "f" (ArgCons (App "g" NoArgs) NoArgs)
```

</details>

### PP2(b) - 5 points

Implement the following pair of mutually recursive functions, which count the
number of `Lit` constructors in an expression or expression list:

```haskell
countLits  :: Expr     -> Int
countLitsL :: ExprList -> Int
```

<details>
<summary>Solution</summary>

```haskell
countLits :: Expr -> Int
countLits (Lit _)     = 1
countLits (Add e1 e2) = countLits e1 + countLits e2
countLits (App _ es)  = countLitsL es

countLitsL :: ExprList -> Int
countLitsL NoArgs          = 0
countLitsL (ArgCons e es)  = countLits e + countLitsL es
```

</details>

### PP2(c) - 5 points

Implement `Eq` instances for both `Expr` and `ExprList`. Two values are equal if
they have the same constructor and all sub-terms are pairwise equal.

```haskell
class Eq a where
  (==) :: a -> a -> Bool
  (/=) :: a -> a -> Bool
  x /= y = not (x == y)
```

<details>
<summary>Solution</summary>

```haskell
instance Eq Expr where
  Lit n1    == Lit n2    = n1 == n2
  Add e1 e2 == Add e3 e4 = e1 == e3 && e2 == e4
  App s1 l1 == App s2 l2 = s1 == s2 && l1 == l2
  _         == _         = False

instance Eq ExprList where
  NoArgs        == NoArgs        = True
  ArgCons e1 l1 == ArgCons e2 l2 = e1 == e2 && l1 == l2
  _             == _             = False
```

</details>

---

## Practice Problem 3 - Functors, Applicatives, and Monads

Consider the following type, which represents a computation that either fails
with a message or succeeds with a value:

```haskell
data Fallible a = Fail String | Succeed a
```

### PP3(a–c) - 4 points each

Implement the `Functor`, `Applicative`, and `Monad` instances for `Fallible`.
Their behavior is demonstrated by the following examples:

```haskell
s1 = Succeed 5        :: Fallible Int
e1 = Fail "too big"   :: Fallible Int
e2 = Fail "too small" :: Fallible Int
sf = Succeed (*2)     :: Fallible (Int -> Int)
ff = Fail "bad op"    :: Fallible (Int -> Int)

-- Functor
fmap (*2) s1               -->  Succeed 10
fmap (*2) e1               -->  Fail "too big"

-- Applicative
pure 7                     -->  Succeed 7
sf  <*> s1                 -->  Succeed 10
sf  <*> e1                 -->  Fail "too big"
ff  <*> s1                 -->  Fail "bad op"
ff  <*> e2                 -->  Fail "bad op"

-- Monad
s1  >>= \x -> Succeed (x+1)    -->  Succeed 6
s1  >>= \x -> Fail "negative"  -->  Fail "negative"
e1  >>= \x -> Fail "negative"  -->  Fail "too big"
```

<details>
<summary>Solution</summary>

```haskell
instance Functor Fallible where
  fmap _ (Fail s)    = Fail s
  fmap f (Succeed x) = Succeed (f x)

instance Applicative Fallible where
  pure = Succeed

  Fail s    <*> _         = Fail s
  _         <*> Fail s    = Fail s
  Succeed f <*> Succeed x = Succeed (f x)

instance Monad Fallible where
  Fail s    >>= _ = Fail s
  Succeed x >>= f = f x
```

</details>
