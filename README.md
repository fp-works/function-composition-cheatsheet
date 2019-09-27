# Composition of Functions

## Why Another Cheatsheet?

FP is about composition of functions. And it is surprisingly hard to find a cheatsheet just for composition of functions in Haskell, so let's try to start with a simple one and iterate on it over time. PRs more than welcome :)

## Change Logs:

| Date | Contributors | Description |
| ---  | ---          | ---
|2019-09-24 | [Daniel Deng](https://github.com/sinogermany) | Initial version, Function as Functor, Applicative and Monad, Kleisli Arrow Composition

## Special Thanks To:

- [Shine Li](https://github.com/shineli1984) who helped me along the way while I worked on the initial version.

## GHC Source Code

<details>
<summary>Function as Functor, Applicative and Monad</summary>

```haskell
-- | @since 2.01
instance Functor ((->) r) where
    fmap = (.)

-- | @since 2.01
instance Applicative ((->) a) where
    pure = const
    (<*>) f g x = f x (g x)
    liftA2 q f g x = q (f x) (g x)

-- | @since 2.01
instance Monad ((->) r) where
    f >>= k = \ r -> k (f r) r
```

</details>

<details><summary>"Komposition" of Functions (Kleisli Arrows)</summary>

```haskell
-- | Left-to-right composition of Kleisli arrows.
(>=>)       :: Monad m => (a -> m b) -> (b -> m c) -> (a -> m c)
f >=> g     = \x -> f x >>= g

-- | Right-to-left composition of Kleisli arrows. @('>=>')@, with the arguments
-- flipped.
--
-- Note how this operator resembles function composition @('.')@:
--
-- > (.)   ::            (b ->   c) -> (a ->   b) -> a ->   c
-- > (<=<) :: Monad m => (b -> m c) -> (a -> m b) -> a -> m c
(<=<)       :: Monad m => (b -> m c) -> (a -> m b) -> (a -> m c)
(<=<)       = flip (>=>)

```

</details>

## Cheatsheet

<details>
<summary>Function as Functor (Function Composition)</summary>

```haskell
f x = f1 (f2 (f3 (fn x)))

-- composition
f = f1 . f2 . f3 . fn

-- GHC.Base.Functor
-- Usually we just use (.) though.
f = f1 `fmap` f2 `fmap` f3 `fmap` fn
f = f1 <$> f2 <$> f3 <$> fn

-- Control.Arrow
f = f1 <<< f2 <<< f3 <<< fn
f = fn >>> f3 >>> f2 >>> f1

-- Flow
f = f1 <. f2 <. f3 <. fn
f = fn .> f3 .> f2 .> f1
```

```haskell
-- Functor Law
f <$> id == f
id <$> f == f
```

</details>

<details><summary>Function as Applicative</summary>

```haskell
f x = f1 x (f2 x)
f = f1 <*> f2
```

```haskell
f x = g (f1 x) (f2 x) (f3 x) (fn x)

-- Applicative Style
f = g <$> f1 <*> f2 <*> f3 <*> fn
f = g . f1 <*> f2 <*> f3 <*> fn
```

```haskell
f x = g (f1 x) (f2 x) (f3 x) (fn x) x

-- Applicative Style
f = g <$> f1 <*> f2 <*> f3 <*> fn <*> id
```

```haskell
f x = g x (f1 x) (f2 x) (f3 x) (fn x)

-- Applicative Style
f = g <$> id <*> f1 <*> f2 <*> f3 <*> fn
f = g <*> f1 <*> f2 <*> f3 <*> fn
```

</details>

<details><summary>Function as Monad</summary>

```haskell
f x = f1 (f2 x) x
f = f1 =<< f2
f = f2 >>= f1
```

</details>

<details><summary>"Komposition" of Functions (Kleisli Arrows)</summary>

```haskell
-- Definition of an Kleisli Arrow:
f :: Monad m => a -> m b
```

```haskell
-- `f1` and `f2` must return the same Monad instance type.
-- If `f1` returns `[b]`, `f2` must return `[c]`.
-- If `f1` returns `Maybe b`, `f2` must return `Maybe c`.
-- If `f1` returns `IO b`, `f2` must return `IO c`.
f1 :: Monad m => a -> m b
f2 :: Monad m => b -> m c

-- >>= is the monadic binding
f a = f1 a >>= f2
f = f1 >=> f2

f a = f2 =<< f1 a
f = f2 <=< f1
```

```haskell
findUser :: UserID -> Either Err User
getDepartment :: User -> Either Err Department
getManager :: Department -> Either Err User

getManagerByUserID :: UserID -> Either Err User
getManagerByUserID = findUser >=> getDepartment >=> getManager

-- caveat: to use 'fish operator' all monadic functions need to return the same type of monad.
-- In this case above for instance, all functions must return `Either Err <SomeType>`
```

</details>
