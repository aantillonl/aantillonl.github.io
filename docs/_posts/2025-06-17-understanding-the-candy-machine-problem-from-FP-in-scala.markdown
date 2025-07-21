---
layout: post
title:  "Understanding the Candy Machine Problem"
date:   2025-06-17 11:00:00 -0500
---

This document is a transcript of [the story published on Medium](https://medium.com/@alejandroantillon/the-candy-machine-problem-a30b6b233517)

The aim of this article is to offer a little guidance for the Candy Machine problem from Functional Programming in Scala — Chapter 6 — and the suggested solution provided in the book’s official GitHub:
https://github.com/fpinscala/fpinscala/blob/second-edition/src/main/scala/fpinscala/answers/state/State.scala
And maybe, more importantly, to offer a bit of emotional support for anyone struggling with this book (I certainly am).

## 🔁 Let’s begin with the update function

This is the simplest part. The update function takes an Input value and a Machine value and returns another machine. For example, a Coin input and a locked machine will return an unlocked machine with one more coin.
One thing to notice is that its arguments are curried — it first takes an Input and returns a function that then takes a Machine and performs the update. This might seem unnecessary at first, but it’s important because the solution separates the definition of a computation from its execution.
By currying the arguments, we can build a transformation ahead of time, even before having a specific machine. Think of it as a function with the action (Coin or Turn) already “baked in,” just waiting for a machine:

```scala
val input = Coin
val preBakedUpdate = update(input)
```

## 🧠 From update functions to State actions

To move toward a fully functional solution, we turn this preBakedUpdate into a state action. A state action is a function that:

* Takes a state (in our case, a Machine)
* Does “something” with it
* Returns a value and a new state

In this case, we don’t care about the value — we just want the updated machine:

```scala
val stateAction = State[Machine, Unit](machine => ((), preBakedUpdate(machine)))
To simplify, we use State.modify:
val stateAction = State.modify(preBakedUpdate) // or State.modify(update(input))
```

The book’s syntax can be a bit obscure. I saw a helpful comment (that I can no longer find) suggesting this alternative implementation of modify, which I find cleaner:

```scala
def modify[S](f: S => S): State[S, Unit] = State(s => ((), f(s)))
```

## ✅ What have we done so far?

Defined update: Input → Machine → Machine (curried)
Created pre-baked update functions
Wrapped those functions as State actions

Now comes the final boss: combining multiple state actions into one.
🍬 Combining state actions
Let’s say we have:

```scala
val sCoin = State.modify(update(Coin))
val m0 = Machine(true, 10, 0)
val (_, m1) = sCoin.run(m0)
assertEquals(m0.locked, true)
assertEquals(m1.locked, false)
```

Now we want candy — so we process a Turn:

```scala
val sTurn = State.modify(update(Turn))
val (_, m2) = sTurn.run(m1)
assertEquals(m2.candies, m0.candies - 1)
assertEquals(m2.coins, m0.coins + 1)
```

This works, but we’ve just executed both actions manually. That goes against the spirit of the State monad — we should be composing computations without executing them until the end.

## ➕ Enter map2

Instead of running them step-by-step, we can combine the two State actions using map2:

```scala
val sCombined = sCoin.map2ViaFlatMap(sTurn)((_, _) => ())
val (_, m1) = sCombined.run(machine)
```

Now we’ve built a single composed computation without executing any partial result. This is the key idea: define the computation separately from its execution.

## 🔁 Handling a list of inputs

Now let’s tackle the full problem: processing a list of inputs.
We could use State.sequence, which is in my opinion a bit easier to follow than traverse:

```scala
State.sequence(inputs.map(update).map(State.modify))
```

The book’s solution uses State.traverse, which flips the flow:

```scala
State.traverse(inputs)(i => State.modify(update(i)))
```

It may look less readable at first, but it accomplishes the same thing. Here’s a breakdown:
*
 sequence: Input → update → modify → sequence
* traverse: Input → traverse + modify (in one go)

Either way, the result is a combined State action that can process a list of inputs once you provide a starting machine.

## 🎯 Getting the final result

The problem asks for a tuple (coins, candies) at the end. But how do we get it if all our state actions return Unit?
We use State.get, which returns the current state as both value and state, and then map it:

```scala
State.traverse(inputs)(i => State.modify(update(i)))
  .flatMap(_ => State.get.map(m => (m.coins, m.candies)))
```

Yes, this syntax is a little intense — but it makes state flow explicit.
Finally, the book uses a for comprehension to make it more readable:

```scala
for
  _ <- State.traverse(inputs)(i => State.modify(update(i)))
  s <- State.get
yield (s.coins, s.candies)
```

## 💬 Final thoughts

This solution is not easy to grasp at first — it took me a long time to understand it. That’s exactly why I wanted to write this post. If nothing else, I hope it offers a little clarity and some emotional support if you’re stuck on this chapter.
If you made it this far and found this helpful, I created a PR with some hints for this problem based on this article. Feel free to leave a comment there. Thank you!
Would you like a Markdown version ready to post to Medium, Dev.to, or GitHub? Or maybe a subtitle and cover image idea to go with it?
