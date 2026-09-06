### The Double-Ended Iterator Pattern

In Rust, a double-ended iterator can consume elements from both the front
(`next`) and the back (`next_back`). It is expressed with the
[`DoubleEndedIterator`](https://doc.rust-lang.org/std/iter/trait.DoubleEndedIterator.html)
trait, which extends the standard `Iterator` trait. This pattern enables
reversing iteration for free (`rev`) and interleaving consumption from both
ends, often halving the work for algorithms like palindrome checks.

### Requirements

Implementing `DoubleEndedIterator` requires that the type already implements
`Iterator` first:

* `Iterator` — implement `next()` to pop an item off the front.
* `DoubleEndedIterator` — implement `next_back()` to pop an item off the back.

The two fronts "meet in the middle"; once the front index passes the back index,
the iterator is exhausted and must return `None`.

### UML Diagrams

#### Trait Hierarchy

```
┌───────────────────────────────┐
│        «trait» Iterator       │
│───────────────────────────────│
│  type Item                    │
│───────────────────────────────│
│  next(&mut self) -> Option<I> │
└───────────────────────────────┘
              ▲
              │  extends (must be implemented
              │  before DoubleEndedIterator)
              │
┌───────────────────────────────┐
│    «trait» DoubleEndedIterator│
│───────────────────────────────│
│  next_back(&mut self)         │
│      -> Option<Self::Item>    │
└───────────────────────────────┘
```

#### Class Diagram

```
┌────────────────────────────────────────────┐
│              Countdown                     │
│────────────────────────────────────────────│
│  - low: i32                                │
│  - high: i32                               │
└────────┬────────────────────────────┬──────┘
         │ implements                 │ implements
         │ (required, first)          │ (second)
         ▼                            ▼
┌──────────────────┐       ┌─────────────────────────┐
│    «trait»       │       │  «trait»                │
│    Iterator      │       │  DoubleEndedIterator    │
│──────────────────│       │─────────────────────────│
│  type Item = i32 │       │  next_back()            │
│──────────────────│       │    -> Option<i32>       │
│  next()          │       │─────────────────────────│
│    low  →  high  │       │  consumes an item from  │
└──────────────────┘       │  the end (high side)    │
                           └─────────────────────────┘
```

#### Execution Trace — `main()`

```
┌──────────────────────────────────────────────┐
│  low=1    high=5                             │
│  next()      → Some(1)   low→2               │
│  next_back() → Some(5)   high→4              │
│  next()      → Some(2)   low→3               │
│  next_back() → Some(4)   high→3              │
│  next()      → Some(3)   low→4               │
│  now low(4) > high(3) → next() = None        │
└──────────────────────────────────────────────┘
```

### Benefits

* **Symmetric traversal**: iterate from either end with equal efficiency.
* **Free `rev()`**: a `DoubleEndedIterator` can be reversed with a single
  method call, no extra code.
* **Shortcuts for symmetric algorithms**: palindrome checks, two-pointer
  searches, and "meet in the middle" problems can stop early instead of
  scanning the entire sequence.
