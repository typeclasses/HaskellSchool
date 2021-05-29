---
version: 1.3.1
title: Containers
---

Tuples, Lists, Assoc Lists, Sets, Maps/Hashmaps, and Vectors.

{% include toc.html %}

## When Should I Use Which Container?

- Quickly group several pieces of data? Consider a [Tuple](#tuples)
- Fast prepend? Consider a [List](#lists)
- Unique and sorted elements? Consider a [Set](#sets)
- Lookup a key in a Key-Value mapping? Consider a [Map](#maps)
- Fast indexing? Consider a [Vector](#vectors)

## Using Container Modules

A very nice feature of the containers ecosystem in Haskell is that all the
modules expose the same api. This allows us to leverage our intuition and feel
familiar with a wide set of tools very quickly.

It also means that we are bound to run into namespace collisions. It is common
practice to import container modules using qualified imports. If you intend on
following along with your own `ghci` session, you should enter the below code to
make sure that all the modules are available.

```haskell
ghci> import qualified Data.Set as Set
ghci> import qualified Data.Map as Map
ghci> :t Map.empty
Map.empty :: Map k a
ghci> :t Set.empty
Set.empty :: Set.Set a
```

Just so you are familiar with the error when it comes up, this is what happens
when you use multiple container modules unqualified.

```haskell
ghci> import Data.List
ghci> import Data.Set
ghci> import Data.Map
ghci> lookup

<interactive>:4:1: error:
    Ambiguous occurrence ‘lookup’
    It could refer to
       either ‘Data.Map.lookup’,
              imported from ‘Data.Map’
              (and originally defined in ‘Data.Map.Internal’)
           or ‘Prelude.lookup’,
              imported from ‘Prelude’ (and originally defined in ‘GHC.List’)
ghci> empty

<interactive>:5:1: error:
    Ambiguous occurrence ‘empty’
    It could refer to
       either ‘Data.Map.empty’,
              imported from ‘Data.Map’
              (and originally defined in ‘Data.Map.Internal’)
           or ‘Data.Set.empty’,
              imported from ‘Data.Set’
              (and originally defined in ‘Data.Set.Internal’)
```

## Tuples

Tuples are the first container you are introduced to in Haskell. They are
simple, primitive, and have a terse, flexible, built in syntax. The fields of a
tuple are referenced them by position in the structure. Although tuples can
theoretically contain an arbitrary number of items, the reality is that the
Haskell Report only requires up to 15-tuples and GHC comes with support for
62-tuples. We will focus on the 2-tuple since Haskell has good built in support
for them. Here is an example of a tuple with 8 fields.

```haskell
ghci> :t ('0', '1', '2', '3', '4', '5', '6', "8-tuple")
('0', '1', '2', '3', '4', '5', '6', "8-tuple")
  :: (Char, Char, Char, Char, Char, Char, Char, [Char])
```
__Note__: `:t` in the ghci repl shows the type of the value, it will print out a
message of the form `term :: Type`

### When to Use

Tuples are useful for associating data; saying "I required both of these
things". They have a nice built-in syntax, which makes them easy to use.
Tuples focus on the _structure_ of data, and not on the _meaning_, so they
are sometimes discouraged. If the data is going to be used in multiple places
it is often better to define a record, but if the use is local then a tuple
can be expedient!

### Constructing Tuples

We can construct tuples using parentheses with the elements separated by a
comma `(a, b)`.

```haskell
ghci> myTuple = (True, "hello")
ghci> :t myTuple
myTuple :: (Bool, [Char])
```

### Consuming Tuples

We can consume tuples by breaking them apart. The most common approach is to
pattern match on the structure of the tuple, and then access its contents. This
technique is nice because it generalizes to tuples of any size.

```haskell
ghci> (\(a, b) -> not a) myTuple
False
ghci> (\(a, b) -> b <> " world") myTuple
"hello world"
```

With a 2-tuple there are the functions `fst` and `snd`, and they are available
in the standard library. 2-tuples are the only arity of tuple that comes with
this kind of built in support, so if you need anything larger be prepared to
write your own accessor functions.

```haskell
ghci> :t fst
fst :: (a, b) -> a
ghci> fst myTuple
True
ghci> :t snd
snd :: (a, b) -> b
ghci> snd myTuple
"hello"
```

### Limitations of Tuples

The main limitation with tuples in Haskell is that each different length of
tuple is a distinct data type. That means there is no general function to add an
additional element to any tuple. Rather these kinds of functions need to be
defined ad hoc.

Here is an example of increasing the tuple's length.

```haskell
ghci> twoTupleToThreeTuple c (a, b) = (a, b, c)
ghci> twoTupleToThreeTuple () myTuple
(1, "world", ())
```

Trying to call our function on a tuple of the incorrect length results in a type
error telling us just as much; the function expected a 2-tuple but we gave it a
3-tuple.

```haskell
ghci> twoTupleToThreeTuple True (1, 2, 3)

<interactive>:19:27: error:
    • Couldn't match expected type: (a, b)
                  with actual type: (a0, b0, c0)
                  ...
```
## Lists

From a usability perspective, Lists solve the problem of extensibility that
tuples face, but they can only contain one type (in other words they are
homogeneous). Lists also have a special built in syntax.

```haskell
ghci> [1,2,3,4]
[1,2,3,4]
ghci> ["hello", True]

<interactive>:22:11: error:
    • Couldn't match type ‘Bool’ with ‘[Char]’
      Expected: String
        Actual: Bool
    ...
```

### Inductive Types

Lists are also a first introduction to an "inductive" or "recursive" type. Here
is an example that is identical to the Haskell implementation without the
syntactic sugar (we use backticks to add infix syntax).

```haskell
data List a = Nil | Cons a (List a)
```

You can see that this type is recursive. `Nil` is the base case, and the `Cons`
constructor joins an `a` and a recursive call to `List a`. You can also see why
lists can only contain 1 type, because the `a` is threaded through the entire
structure. In our definition we can replace `Nil` with the empty list `[]` and
`Cons` with `:` to get back to the built-in syntax.

In that case our original example breaks down like so:

```haskell
ghci> [1,2,3,4]
[1,2,3,4]
ghci> 1 : 2 : 3 : 4 : []
[1,2,3,4]
ghci> Cons 1 (Cons 2 (Cons 3 (Cons 4 Nil)))
Cons 1 (Cons 2 (Cons 3 (Cons 4 Nil)))
```

### List Performance

Haskell implements lists as linked lists. The cons cells (the operator : is
called cons, short for constructor) act as the links. This dictates which
operations can be done quickly and which can be slow:

Prepending a value to a list is easy and fast - all we have to do is create a
new cons cell with the new value and point it to the existing list.

```haskell
prepend value list = value : list
```

On the other hand, since the list data type (as shown above) can be either
empty (Nil) or a cons cell that will point to the rest of the list, it does not
contain information about the length of the list, or a reference to the
end of the list.

Because of that, in order to retrieve the length of a list we must walk
each cons cell and count until we reach the end of the list, and to find the
value in a specific index we need to traverse the list until we reach it.

Similarly, in order to append a list to an existing list, we need to go to the
end of the existing list, and add a cons cell that points to the list we are
appending:

```haskell
append originalList newList =
    case originalList of
        [] -> newList
        x : xs -> x : append xs newList
```

This means many list operations run in linear time (`𝛰(n)`), which in some
cases can be significantly slower than some of the alternative containers.
This is a great reason to be familiar with the different kinds of containers
and their tradeoffs.

### When to Use

Linked lists are an extremely common data structure in functional programming,
this means you are going to see them everywhere in Haskell. They are generally
the first container a Haskell programmer reaches for.  Because of slow append
and relatively slow indexed access (𝛰(n) where n is the index) they are
generally used in situations where you know you are going to have to iterate
over the entire data set, or you want to preserve the order of elements.

A great example of an abstract data structure for which a linked list is a great
concrete implementation are stacks. This is because pushing and popping are
𝛰(1).

A bad use case would be for a queue, where either enqueue or dequeue would be
𝛰(n) depending on which side of the linked list you decide to insert into.

A common real world example of where lists are used is database queries. A
database query could return no results `[]`, or some `[entity..]`, and there is
potentially an ordering to these results. The database library doesn't really
care about indexed access, and so it leaves that consideration to the caller of
the function.

### List Concatenation

List concatenation uses the `++` operator:

```haskell
ghci> [1, 2] ++ [3, 4, 1]
[1, 2, 3, 4, 1]
```

### Head / Tail

When using lists, it is common to work with a list's head and tail. The head is
the list's first element, while the tail is a list containing the remaining
elements.

Haskell provides two helpful functions, `head` and `tail`, for working with
these parts:

```haskell
ghci> head ["Orange", "Banana", "Apple"]
"Orange"
ghci> tail ["Orange", "Banana", "Apple"]
["Banana","Apple"]
```

Unfortunately these functions reveal an ugly part of the language's base
library; they may raise an exception, even when given an argument with the
appropriate type. The cause of these exceptions is that they do not cover the
full domain of possible inputs, so there is room for undefined behaviour. This
means that they do not cover the full domain of possible inputs.

```haskell
ghci> head []
*** Exception: Prelude.head: empty list
ghci> tail []
*** Exception: Prelude.tail: empty list
```

We can use a common idiom in Haskell for covering partial functions in a safe
way, the `Maybe` type. This allows us to say that unhandled inputs return a
`Nothing`. Now the caller of this maybe-returning-function must handle the
`Nothing` case, but in return they are not faced with a nasty runtime exception.

```haskell
ghci> :i Maybe
data Maybe a = Nothing | Just a 	-- Defined in ‘GHC.Maybe’
...
```

__Note__: `:i` in ghci will give you some information about the type, the first
line is the implementation.

Now we can define a total head and tail function using pattern matching!

```haskell
ghci> :{
| safeHead :: [a] -> Maybe a
| safeHead [] = Nothing
| safeHead (x:xs) = Just x
|
| safeTail :: [a] -> Maybe [a]
| safeTail [] = Nothing
| safeTail (x:xs) = Just xs
| :}
ghci> safeHead ["Orange", "Banana", "Apple"]
Just "Orange"
ghci> safeHead []
Nothing
ghci> safeTail ["Orange", "Banana", "Apple"]
Just ["Banana","Apple"]
ghci> safeTail []
Nothing
```

__Note__: `:{` and `:}` allow you to write multiline definitions in ghci.

Hooray! No more exceptions.

Another container that is the cousin of the list, and ensures that `head` and
`tail` are safe is the nonempty list.

```haskell
ghci> import qualified Data.List.NonEmpty as NE
ghci> :i NE.NonEmpty
type NE.NonEmpty :: * -> *
data NE.NonEmpty a = a NE.:| [a]
  	-- Defined in ‘GHC.Base’
```

From its definition we can see that `NonEmpty` requires the first element to be
present. This symbol `:|` is a constructor just like `:`, in fact the definition of
`NonEmpty` is identical to that of lists, except it omits the `[]` (Nil) case.

This handles the partiality problem the exact opposite way as the
`Maybe` solution. Instead of forcing the caller of the function to guard against
the ill-defined case when handling the result of the function, it forces them to
construct a valid input up front (when calling the function).

```haskell
ghci> :t NE.head
NE.head :: NE.NonEmpty a -> a
ghci> NE.head (1 NE.:| [])
1
ghci> NE.head []

<interactive>:11:9: error:
    • Couldn't match expected type: NE.NonEmpty a
                  with actual type: [a0]
    ...
```

Notice that this time the error is not a runtime exception but a type error, the
compiler is telling us that we tried to use a (potentially empty) list rather
than the required `NonEmpty` type.

## Assoc lists

So far we have only been able to access values via position indexed lookup, or
pattern matching. However, one of the most common use cases for containers is
acting as a key value store. Assoc(iation) lists provide this by combining
2-tuples and regular lists. Since assoc lists are really just the combination
of two existing data types, the only thing we need is a lookup function, which
is provided in the `Data.List` module in base.

```haskell
ghci> assoc = [("foo", True), ("bar", False)]
ghci> :t assoc
assoc :: [(String, Bool)]
ghci> :t lookup
lookup :: Eq a => a -> [(a, b)] -> Maybe b
ghci> lookup "foo" assoc
Just True
ghci> lookup "bar" assoc
Just False
ghci> lookup "baz" assoc
Nothing
```

We can see the pattern, again, where no result is encoded using the `Maybe`
type. This allows us to have complete functions with errors at compile time
rather than runtime.

It is also interesting to note the `Eq a` constraint on the "key" which allows
the lookup function to do an equality comparison on them.

While assoc lists are a nice introduction to key value container that builds
on the previous types we learned about, they are not particularly useful. A list
simply isn't a very good data structure for lookup, as it provides worst case
`𝛰(n)` asymptotics. Assoc lists are usually an intermediate data structure
which Haskell programmers will usually convert into a `Map`. Although this
conversion is itself an `𝛰(n*log n)` operation `Map` provides an `O(log n)`
lookup, so the conversion cost is amortised very quickly.

## Sets

Sets are a very interesting container, the core concept of a set is membership.
It is common to build up sets, and then test to see if an element exists in that
set.

A set can be constructed exclusively by inserting elements into the empty set

```haskell
ghci> :t Set.empty
Set.empty :: Set.Set a
ghci> Set.empty
fromList []
ghci> Set.insert 1 (Set.insert 2 (Set.insert 3 Set.empty))
fromList [1,2,3]
```

Or by creating a set from a list

```haskell
ghci> Set.fromList [4,3,2,1]
fromList [1,2,3,4]
```

You might notice that the elements are sorted after turning them into a `Set`.
Internally the `Set` data type depends on its contents being orderable. The
concrete implementation of sets in Haskell are binary trees, which depend on an
ordering. We can see the ordering requirement from the constraints on the
`insert` and `fromList` (and other) functions.

```haskell
ghci> :t Set.insert
Set.insert :: Ord a => a -> Set.Set a -> Set.Set a
ghci> :t Set.fromList
Set.fromList :: Ord a => [a] -> Set.Set a
ghci> :t Set.member
Set.member :: Ord a => a -> Set.Set a -> Bool
```

Sets have a very useful property, they cannot contain duplicates.

```haskell
ghci> insert1 = Set.insert 1
ghci> insert1 Set.empty
fromList [1]
ghci> insert1 (insert1 Set.empty)
fromList [1]
ghci> insert1 (insert1 (insert1 Set.empty))
fromList [1]
```

Alright, let's see an almost-practical use case for sets.

```haskell
ghci> evens = Set.fromList [0,2..1000000]
ghci> Set.member 7 evens
False
ghci> Set.member 200012 evens
True
ghci> isEven n = Set.member n evens
ghci> isEven 7
False
ghci> isEven 8
True
```

You might say "hmmm a `1000000` limit for even numbers seems incorrect", and you
would be right! This highlights a property of sets in haskell, they are finite
due to the strictness in the underlying data structure. This contrasts lists,
which are lazy and therefore potentially infinite.

### Difference

Set difference is a good way to break apart a set based on the membership of its
elements in another set. It will return a set with all the elements of the first
argument less the elements of its second argument.

```haskell
ghci> set1 = Set.fromList ["a", "b", "c", "1", "2", "3"]
ghci> letters = Set.fromList ["a", "b", "c"]
ghci> nums = Set.fromList ["1", "2", "3"]
ghci> Set.difference set1 letters
fromList ["1","2","3"]
ghci> Set.difference set1 nums
fromList ["a","b","c"]
```

This brings up a critical point of order! The `difference` function subtracts
its second argument from the first, so mixing up your sets can lead to undesired
behaviour.

```haskell
ghci> Set.difference nums set1
fromList []
```

### Union

Union's are useful for when we want to build up a new set from the combination
of two other sets. Union will create a new set with the members of each set and,
of course, no duplicates.

```haskell
ghci> nums
fromList ["1","2","3"]
ghci> letters
fromList ["a","b","c"]
ghci> Set.union nums letters
fromList ["1","2","3","a","b","c"]
ghci> set1
fromList ["1","2","3","a","b","c"]
ghci> Set.union nums set1
fromList ["1","2","3","a","b","c"]
```

### Intersection

Intersection allows us to find elements that sets have in common.

```haskell
ghci> Set.intersection nums letters
fromList []
ghci> Set.intersection nums set1
fromList ["1","2","3"]
ghci> Set.intersection letters set1
fromList ["a","b","c"]
ghci> Set.intersection (fromList [1, 2]) (fromList [2, 3])
fromList [2]
```

### Subsets

Subsets allow us to detect if all the elements of a set are contained within
another set. The order of the arguments matters here, we are asking if the first
argument is a subset of the second argument.

```haskell
ghci> Set.isSubsetOf letters set1
True
ghci> Set.isSubsetOf nums set1
True
ghci> Set.isSubsetOf nums letters
False
ghci> Set.isSubsetOf set1 nums
False
```

Every set is a subset of itself.

```haskell
ghci> Set.isSubsetOf nums nums
True
ghci> Set.isSubsetOf set1 set1
True
ghci> Set.isSubsetOf letters letters
True
```

### Cartesian Products

Cartesian products, sometimes referred to as cross multiplication, allow us to
get all the possible combinations of the elements of two sets.

A classic example of a Cartesian product is a chess board, which is the cross
multiplication of the letters a - h and the numbers 1 - 8.

```haskell
ghci> ranks = Set.fromList [1..8]
ghci> files = Set.fromList ['a'..'h']
ghci> Set.cartesianProduct files ranks
fromList
  [('a',1),('a',2),('a',3),('a',4),('a',5),('a',6),('a',7),('a',8)
  ,('b',1),('b',2),('b',3),('b',4),('b',5),('b',6),('b',7),('b',8)
  ,('c',1),('c',2),('c',3),('c',4),('c',5),('c',6),('c',7),('c',8)
  ,('d',1),('d',2),('d',3),('d',4),('d',5),('d',6),('d',7),('d',8)
  ,('e',1),('e',2),('e',3),('e',4),('e',5),('e',6),('e',7),('e',8)
  ,('f',1),('f',2),('f',3),('f',4),('f',5),('f',6),('f',7),('f',8)
  ,('g',1),('g',2),('g',3),('g',4),('g',5),('g',6),('g',7),('g',8)
  ,('h',1),('h',2),('h',3),('h',4),('h',5),('h',6),('h',7),('h',8)
  ]
```

You're can thank me later when you ace the n-queens question on your next
technical interview.

## Maps

In Haskell, maps are the "go-to" key-value store, sometimes referred to as a
dictionary.

A `Map` requires that its key has an ordering (`Ord k`), in the same way that
`Set` required an ordering on the value. This is because the internal
implementation of the `Map` type in Haskell is a size balanced binary tree. This
is actually the same data structure as is used for `Set`, sets just assign a
number for the key based on the order of the value.

The `Map` data type and the functions for interacting with it are exported from
`Data.Map` which is a module in the `containers` package. Don't worry about
dependencies though, containers is a core library and ships with ghci.

Like sets, a map can be constructed by inserting key value pairs into an empty map

```haskell
ghci> :t Map.empty
Map.empty :: Map.Map k a
ghci> Map.empty
fromList []
ghci> Map.insert "a" 'a' (Map.insert "b" 'b' (Map.insert "c" 'c' Map.empty))
fromList [("a",'a'),("b",'b'),("c",'c')]
```

Or by creating it from a list

```haskell
ghci> Map.fromList [(4, '4'), (3, '3'), (2, '2'), (1,'1')]
fromList [(1,'1'),(2,'2'),(3,'3'),(4,'4')]
```

Luckily `containers` exposes very uniform api's. So we get all of our set
operations on maps, and they are just as performant! Lists also have the set
operations but they are much slower due to the different performance
characteristics of linked lists and binary trees.

## Updating Values to a Map

A useful function to be aware of is `adjust`. This lets us update a value at a
specified key, only if it exists, if not the old map is returned.

```haskell
ghci> Map.adjust (+2) "first" oneItem
fromList [("first",3)]
ghci> :t Mapadjust
Map.adjust :: Ord k => (a -> a) -> k -> Map.Map k a -> Map.Map k a
ghci> Map.adjust (+2) "second" oneItem
fromList [("first",1)]
```

The observant reader will notice that `adjust` doesn't actually update the map.
The second invocation of adjust returns the original `oneItem` map, this makes
sense when you consider that all data in Haskell is immutable!

## When to use Maps

Maps are really great for in memory persistence of state that will need to be
retrieved by a key of some arbitrary type. This is is because maps have great
lookup asymptotics (`𝛰(log n)`) due to the ordering on the key values. The
example that immediately springs to mind is session storage on the server in a
web application. The session state can be indexed by an id that is stored in
the cookie, and the session state can be retrieved from an in memory `Map` on
every request. This solution doesn't scale infinitely, but you would be
surprised how well it works!

## HashMaps

There are some cases where we do not have an ordering on our type, but still
want to use it as a key to index a map. In this case we probably want to reach
for a hashmap. A hashmap simply hashes the key, et voilà, we have an ordering!
This does require that the key type is hashable though.

The module that exports the `HashMap` data type and functionality is called
`Data.HashMap.Strict`, it lives in the `unordered-containers` package. The api
is identical to `Map` aside from a `Hashable k` constraint instead of an `Ord k`
on the key.

## Vectors

Sometimes we want a list like container where we can have performant indexed
access. In fact arrays are often the primitive sequential data structure in lots
of popular programming langauges (rhymes with guava equipped). Well Haskell has
those kinds of data structures too, and the most popular implementation is the
`Vector` type from the `vector` library.

One of the coolest things about the `Vector` type is that it offers `𝛰(1)`
indexed access. The `vector` library offers both a safe and unsafe index
accessor operator.

Here is an example where we want to create a container that allows us to
retrieve ascii chars by their character code.

__Note__: You can fetch the `vector` library using `cabal repl --build-depends "vector"`
or `stack exec --package vector -- ghci`

```haskell
ghci> import qualified Data.Vector as V
ghci> asciiChars = V.fromList ['\NUL'..'\DEL']
ghci> asciiChars ! 48
'0'
ghci> asciiChars ! 97
'a'
ghci> asciiChars ! 65
'A'
ghci> asciiChars !? 65
Just 'A'
ghci> asciiChars !? 128
Nothing
ghci> asciiChars !? 127
Just '\DEL'
```

Vectors also offer efficient slicing `𝛰(1)`, so they are really good for
creating sub-vectors. The slice function takes the index to start at, the
length of the slice, and the vector to slice, and returns a new vector. Slicing
is unsafe, if the starting index plus the length of the slice exceeds the vector
you will get a runtime error.

```haskell
ghci> :t V.slice
V.slice :: Int -> Int -> V.Vector a -> V.Vector a
ghci> lowerCase = V.slice 97 26 asciiChars
ghci> lowerCase
"abcdefghijklmnopqrstuvwxyz"
ghci> upperCase = V.slice 65 26 asciiChars
ghci> upperCase
"ABCDEFGHIJKLMNOPQRSTUVWXYZ"
ghci> nums = V.slice 48 10 asciiChars
ghci> nums
"0123456789"
 ghci> V.slice 97 92 asciiChars
"*** Exception: ./Data/Vector/Generic.hs:408 (slice): invalid slice (97,92,128)
CallStack (from HasCallStack):
  error, called at ./Data/Vector/Internal/Check.hs:87:5 in vector-0.12.3.0-8cc976946fcdbc43a65d82e2ca0ef40a7bb90b17e6cc65c288a8b694f5ac3127:Data.Vector.Internal.Check
```

## Overloaded Lists

You may have noticed that we frequently use the `fromList` function to
construct our containers. There is a language extension that allows us to
simply use the list syntax to construct these containers. The drawback of
overloading the list syntax is that we can get confusing type errors if we are
not explicit about our types.

```haskell
ghci> :set -XOverloadedLists -- This is the language extension
ghci> [1,2,3] :: Set.Set Int
fromList [1,2,3]
ghci> [1,2,3] :: V.Vector Int
[1,2,3]
ghci> [('a', 1),('b', 2),('c', 3)] :: Map.Map Char Int
fromList [('a',1),('b',2),('c',3)]
```

However, if we aren't explicit about the type...

```haskell
ghci> ["1", "2", "3"]

<interactive>:11:1: error:
    • Illegal equational constraint GHC.Exts.Item l ~ [Char]
      (Use GADTs or TypeFamilies to permit this)
    • When checking the inferred type
        it :: forall {l}.
              (GHC.Exts.IsList l, GHC.Exts.Item l ~ [Char]) =>
              l
```

This error is a bit of a doozey, which is why this language extension is
typically not enabled by default. It is still good to be aware of it!