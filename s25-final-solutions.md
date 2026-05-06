# CS 340 Spring 2025 Final Exam: Solution Guide

---

## Problem 1: Multiple Choice (30 points)

### Question 1 - Type Synonyms

```haskell
type Point3D = (Float, Float, Float)
distanceToOrigin :: Point3D -> Float
```

**Answer: B - `distanceToOrigin (1.0, 1.0, 1.0)`**

`Point3D` is a *type synonym* for the tuple `(Float, Float, Float)` - not a new
type constructor. The name `Point3D` cannot appear in expressions, only in type
signatures. A valid call simply passes a 3-tuple literal.

---

### Question 2 - Counting ADT Values

```haskell
data Foo = FooNull | FooBool Bool          -- 1 + 2 = 3 values
data Bar = BarNull | BarBool Bool | BarFoo Foo  -- 1 + 2 + 3 = 6 values
```

**Answer: C - 6**

---

### Question 3 - Legal ADT Values

**Answer: E - `BarFoo (FooBool True)`**

---

### Question 4 - Polymorphic ADT Values

```haskell
data Qux a b = QuxA a | QuxB b | QuxQ a (Qux b a)
```

Target type: `Qux Int Bool`

**Answer: D - `QuxQ 1 (QuxQ True (QuxQ 2 (QuxA False)))`**

---

### Question 5 - Kinds

```haskell
data Kun a b = Kun b (a b)
```

**Answer: B - `(* -> *) -> * -> *`**

The field `a b` constrains `a` to have kind `* -> *` (it is applied to `b`). The
field `b` alone means `b :: *`. So `Kun` takes a `(* -> *)` argument, then a `*`
argument, yielding a `*`:

---

### Question 6 - Applying a Higher-Kinded Type

`Kun :: (* -> *) -> * -> *`

**Answer: C - `Kun Maybe Int`**

---

### Question 7 - Type Class Concepts

**Answer: A - They must provide implementations of the required methods defined
in `Baz`**

This is the core contract of a type class instance. The other options describe
properties that may hold for *some* classes (e.g., `Functor` instances are
container-like) but are not universal requirements.

---

### Question 8 - Type Class Instance Evaluation

```haskell
instance Mungible Int where
  tweak n = (n * 2) `mod` 5
```

`munge [1..5]` applies `tweak` to each element:

| Input | `n * 2` | `mod 5` |
| ----- | ------- | ------- |
| 1     | 2       | 2       |
| 2     | 4       | 4       |
| 3     | 6       | 1       |
| 4     | 8       | 3       |
| 5     | 10      | 0       |

**Answer: D - `[2,4,1,3,0]`**

---

### Question 9 - Higher-Kinded Type Class Instance

```haskell
class Fanfare k where
  tada :: k a -> (a -> String) -> String
```

`k` must have kind `* -> *` (it is applied to `a`).

**Answer: D - `instance Fanfare Maybe`**

---

### Question 10 - Functor Concepts

**Answer: B - they can be "mapped" over using an appropriate function**

`fmap :: (a -> b) -> f a -> f b` is the defining operation of `Functor`. The
other options are either too narrow (A describes `Maybe`/`Either`) or simply
false (C: `Functor` does not imply `Monad`; many `Functor`s are not `Monad`s).

---

### Question 11 - Functor / Applicative Application

```haskell
ternary :: Int -> Maybe Int -> Int -> Bool
```

**Answer: A - `ternary <$> Just 5 <*> Just Nothing <*> Just 10`**

---

### Question 12 - Applicative Either Evaluation

```haskell
pure (++) <*> Left "Doo" <*> Right "Wop"
```

**Answer: E - `Left "Doo"`**

Step by step, using the provided instance:

```
pure (++) <*> Left "Doo"
  = Right (++) <*> Left "Doo"
  = Left "Doo"              -- rule: (Right f) <*> (Left x) = Left x

Left "Doo" <*> Right "Wop"
  = Left "Doo"              -- rule: (Left x) <*> _ = Left x
```

---

### Question 13 - `do`-notation Desugaring

```haskell
do x <- m1 a
   y <- m2 c d
   return (x + y)
```

**Answer: D - `m1 a >>= \x -> m2 c d >>= \y -> return (x + y)`**

Each `<-` binding desugars to `>>=` with a lambda capturing the bound variable.
Sequencing without a bind (`>>`) is only used when the result is explicitly
discarded.

---

### Question 14 - State Monad Trace

```haskell
bip     = State $ \(c1:c2:cs) -> (c2, c1:cs)
bop s   = State $ \(c:cs)     -> (c,  s++cs)
sfun    = do c1 <- bop "abc"
             c2 <- bip
             c3 <- bop "def"
             return [c1,c2,c3]
```

Trace `runState sfun "ghi"`:

| Step | Action              | State before | Result                   | State after                   |
| ---- | ------------------- | ------------ | ------------------------ | ----------------------------- |
| 1    | `bop "abc"`         | `"ghi"`      | `c1 = 'g'`               | `"abc" ++ "hi"` = `"abchi"`   |
| 2    | `bip`               | `"abchi"`    | `c2 = 'b'` (second char) | `'a':"chi"` = `"achi"`        |
| 3    | `bop "def"`         | `"achi"`     | `c3 = 'a'`               | `"def" ++ "chi"` = `"defchi"` |
| 4    | `return [c1,c2,c3]` | `"defchi"`   | `"gba"`                  | `"defchi"`                    |

**Answer: D - `("gba", "defchi")`**

---

### Question 15 - StateT / Maybe Monad Stack Trace

```haskell
lose n = StateT $ \b -> if b < n then Nothing else Just (-n, b-n)
earn n = StateT $ \b -> Just (n, b+n)
trans  = do n1 <- lose 40
            n2 <- earn 20
            n3 <- lose 30
            return (n1,n2,n3)
```

Trace `runStateT trans 50`:

| Step | Action    | Balance before | Result                           | Balance after |
| ---- | --------- | -------------- | -------------------------------- | ------------- |
| 1    | `lose 40` | 50             | `50 < 40`? No → `Just (-40, 10)` | 10            |
| 2    | `earn 20` | 10             | `Just (20, 30)`                  | 30            |
| 3    | `lose 30` | 30             | `30 < 30`? No → `Just (-30, 0)`  | 0             |
| 4    | `return`  | 0              | `(-40, 20, -30)`                 | 0             |

**Answer: C - `Just ((-40, 20, -30), 0)`**

Note: `lose 30` with balance exactly 30 succeeds because the guard is strict:
`b < n` (not `b <= n`).

---

## Problem 2: Monadic Parsing (12 points)

The grammar (operators are parsed but *discarded* - not stored in the AST):

```
prefixExp = litExp <|> unaryExp <|> binExp
litExp    = Lit <$> token int
unaryExp  = (symbol "!" <|> symbol "-") >> prefixExp >>= \e -> return (Unary e)
binExp    = (symbol "+" <|> symbol "*") >> prefixExp >>= \e1 -> prefixExp >>= \e2 -> return (Binary e1 e2)
```

### Answers

| #    | Input         | Answer | Reasoning                                                                                                                                                                                               |
| ---- | ------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| i    | `"3 - 4"`     | **B**  | `litExp` succeeds: `Just (Lit 3, "- 4")`                                                                                                                                                                |
| ii   | `"! 3 - 4"`   | **C**  | `unaryExp` parses `!`, then `litExp` on `"3 - 4"` → `Lit 3` rem `"- 4"`. Returns `Just (Unary (Lit 3), "- 4")`                                                                                          |
| iii  | `"- + 3 4"`   | **E**  | `unaryExp` parses `-`, then `binExp` on `"+ 3 4"` → `Binary (Lit 3) (Lit 4)` rem `""`. Returns `Just (Unary (Binary (Lit 3) (Lit 4)), "")`                                                              |
| iv   | `"+ 3 - 4"`   | **F**  | `binExp` parses `+`, first prefixExp → `Lit 3`, second prefixExp on `"- 4"`: `unaryExp` → `Unary (Lit 4)`. Returns `Just (Binary (Lit 3) (Unary (Lit 4)), "")`                                          |
| v    | `"+ + 3 4"`   | **A**  | `binExp` parses `+`, first inner prefixExp consumes all of `"+ 3 4"` as `Binary (Lit 3) (Lit 4)` leaving `""`, then second prefixExp on `""` fails → `Nothing`                                          |
| vi   | `"! - 3 4"`   | **D**  | `unaryExp` parses `!`, inner `unaryExp` parses `-`, innermost `litExp` → `Lit 3` rem `"4"`. Returns `Just (Unary (Unary (Lit 3)), "4")`                                                                 |
| vii  | `"+ * 3 4"`   | **A**  | Same as v: `binExp` parses `+`, first inner prefixExp consumes all of `"* 3 4"` as `Binary (Lit 3) (Lit 4)`, second prefixExp on `""` fails → `Nothing`                                                 |
| viii | `"! + 3 - 4"` | **H**  | `unaryExp` parses `!`, inner `binExp` parses `+` then `Lit 3` and `Unary (Lit 4)` from `"- 4"` → `Binary (Lit 3) (Unary (Lit 4))` rem `""`. Returns `Just (Unary (Binary (Lit 3) (Unary (Lit 4))), "")` |

---

## Problem 3: Data Types and Type Classes (12 points)

```haskell
data T a b = TV1 b (T a b) | TV2 a
```

### WP1(a) - Legal value of `T String Int` with ≥ 2 `TV1` constructors (2 pts)

```haskell
TV1 1 (TV1 2 (TV2 "hello")) :: T String Int
```

Here `b = Int` (goes in `TV1`) and `a = String` (goes in `TV2`).

---

### WP1(b) - Implement `tmap` (5 pts)

```haskell
tmap :: (a -> c) -> (b -> d) -> T a b -> T c d
tmap f g (TV1 y t) = TV1 (g y) (tmap f g t)
tmap f g (TV2 x)   = TV2 (f x)
```

- `TV1` holds a `b` and a recursive `T a b`; apply `g` to the `b` value and
  recurse.
- `TV2` holds an `a`; apply `f` to it.

---

### WP1(c) - `Eq` instance for `T` (5 pts)

```haskell
instance (Eq a, Eq b) => Eq (T a b) where
  (==) :: T a b -> T a b -> Bool
  (TV2 x)   == (TV2 y)   = x == y
  (TV1 y t) == (TV1 z u) = y == z && t == u
  _         == _         = False
```

Two values are equal only if they share the same constructor. Mixed constructors
are immediately `False`. Structural equality recurses on the tail of `TV1`.

---

## Problem 4: Functors, Applicatives, and Monads (12 points)

```haskell
data Annotated a = Annotated String a
```

Key behavior from the examples:

- The `String` annotation is **preserved** by `fmap` (only the value changes).
- `pure` produces an **empty** annotation `""`.
- `<*>` and `>>=` **concatenate** the annotations from left to right.

---

### WP2(a) - `Functor` instance (4 pts)

```haskell
instance Functor Annotated where
  fmap :: (a -> b) -> Annotated a -> Annotated b
  fmap f (Annotated s x) = Annotated s (f x)
```

The annotation is unchanged; the function is applied to the contained value.

---

### WP2(b) - `Applicative` instance (4 pts)

```haskell
instance Applicative Annotated where
  pure :: a -> Annotated a
  pure x = Annotated "" x

  (<*>) :: Annotated (a -> b) -> Annotated a -> Annotated b
  (Annotated sf f) <*> (Annotated sx x) = Annotated (sf ++ sx) (f x)
```

- `pure` wraps a value with an empty annotation.
- `<*>` concatenates the two annotations and applies the function.

---

### WP2(c) - `Monad` instance (4 pts)

```haskell
instance Monad Annotated where
  (>>=) :: Annotated a -> (a -> Annotated b) -> Annotated b
  (Annotated s x) >>= f = let Annotated s' y = f x
                           in Annotated (s ++ s') y
```

- Extract the value `x` and run `f` on it to get a new `Annotated s' y`.
- Prepend the original annotation to the new one.
