# Lenses

https://www.reddit.com/r/haskell/comments/9ded97/is_learning_how_to_use_the_lens_library_worth_it/

## Lens
`Lens' s a`: given a type `s` (the whole) that ALWAYS has an `a` in it by definition of a product type, a `Lens' s a` is a way of getting and setting that `a` (part) inside of `s` (whole). `Lens s t a b` is just for providing polymorphic updates
of changing the type of the part `a` you're setting within the whole `s`.

## Prism
`Prism' s a`: given a type `s` (the whole) that MIGHT have an `a` in it by definition of a sum type, a `Prism' s a` is a way of extracting the `a` (branch in a sum type) if it exists, and being able to create an `s` given an `a`.

## Traversal
`Traversal' s a`: target many `a`s which may or may not exist inside of an `s`.

## Iso
`Iso' s a`: says that `s` and `a` are different representations of the same type
All Lenses are Traversals and all Prisms are also Traversals. Not all Traversals are Lenses or Prisms though.

The canonical operations for these things are as follows
Note: You can use the function name or there are infix operators

`&` is the opposite of `$`, it will be applied last.

# Lens operators
https://github.com/ekmett/lens/wiki/operators

For lens you'll use `view,set,over`.

```
view _2 (1,2,3) == (1,2,3) ^. _2
```

### set

```
set _2 23 (1,2,3) == (1,2,3) & _2 .~ 23
```

### over

lets you run a function to change the value.
mnemonic % implies mod eg: modifying something.

```
over _2 (+50) (1,2,3) == (1,2,3) & _2 %~ (+50)

```

the "usual" infix operators: (+~), (-~), (*~), (<>~) etc -- add/subtract/multiply/mappend a value to the existing one

```
person1 & age +~ 1
person1 & age *~ 5
```


# Prism operators

Something that may or may not exist

### preview

`preview (^?)`: maybe get a value out of a structure. mnemonic: previewing is getting a sneak peek

```
Right 5 ^? _Left == preview _Left $ Right 5
=> Nothing

Right 5 ^? _Right == preview _Right $ Right 5
=> Just 5
```

### review
`review (#)`: construct an `s` (the whole) out of an `a` (the part).
I suppose its like applicative `pure`.

```
review _Left "hello" == _Left # "hello"
=> Left "hello"

review _Just "hello" == _Just # "hello"
=> Just "hello"
```




https://stackoverflow.com/questions/50915526/what-are-prisms

Lenses characterise the has-a relationship;
Prisms characterise the is-a relationship.

Lenses "focus" in on some part of a structure (product type) that ALWAYS exists.
Prims address some part of a structure (sum type) that may exist

A `Lens s a` says "s has an a"; it has methods to get exactly
one `a` from an `s` and to overwrite exactly one `a` in an `s`.

Lenses are for focusing in on different slots in a product type that ALWAYS exist by definition.
A `Person` consists of a `String` AND `Int` AND `Double`

```
data Person = Person String Int Double
```

The `Lens s t a b` is for polymorphic updates. Its possible to change the type of the part you're setting within the whole.

```
s = The whole
a = The part within the whole

Lens s a
```

Can also think of it like this

```
data Lens s a = Lens {
  get :: s -> a,
  set :: a -> s -> s
}
              s      a
struct Lens<Whole, Part> {
  get: Whole -> Part
  set: Part -> Whole -> Whole
}
```

# Prism

A `Prism s a` says "a is an s"; it has methods to upcast
an `a` to an `s` and to (attempt to) downcast an `s` to an `a`

When its a sum type `or`, you don't know which case (branch) you're on. This is what a `Prism` is for.

up injects an a into s (without adding any information), and down tests whether the s is an a.

In lens, up is spelled review and down is preview.
There’s no Prism constructor; you use the prism' smart constructor.

```
data Prism s a = Prism {
    up :: a -> s,
    down :: s -> Maybe a
}

struct Prism<Whole, Part> {
  inject: Part -> Whole            -- Horse is Animal
  tryGet: Whole -> Maybe Part      -- The Animal can be an of the 4 cases, don't know which one!
}

data Animal
  = Horse Int
  | Goat Text
  | Cat Int
  | Dog [Int]
  deriving (Show, Eq)

_HorseP :: Prism' Animal Int
_HorseP =
  prism'
    Horse
    (\a -> case a of
      Horse i -> Just i
      _ -> Nothing)
```

You cant write a lens for an `Either`, you don't know if its a left or right.
Lenses don't support this - you can't write a Lens (Either a b) a because you can't implement get :: Either a b -> a

```
_Left :: Prism (Either a b) a
_Left = Prism {
    up = Left,
    down = either Just (const Nothing)
}
_Right :: Prism (Either a b) b
_Right = Prism {
    up = Right,
    down = either (const Nothing) Just
}
-}
```

This is how you create a prism using smart constructor

```
rightPrism :: Prism' (Either a b) b
rightPrism = prism' Right (either (const Nothing) Just)
```

by convention, template haskell creates prisms which begin with _underscore

Why would you want to use a `Prism`?
Perfect for writing generic set functions for example that will go into some structure and update it if its there
otherwise it will just do nothing without crashing like you would in java with a NPE.

compose a prism with a lens. go into the persons contact sum type and if its mobile update their mobile number
else do nothing.

`person1 & contact . _Mobile .~ "34459"`


# Traversals
Traversals are like prisms, but traversals may address many things that may or may not exist
