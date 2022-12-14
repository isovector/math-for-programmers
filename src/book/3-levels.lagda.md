# Levels

from https://nextjournal.com/zampino/russell-paradox

```agda
module 3-levels where

open import Level
postulate
  unimportant : {ℓ : Level} {A : Set ℓ} → A
```

Perhaps you have heard of Bertrand Russell's "barber paradox" --- if there is a
barber who shaves only barbers who do not shave themselves, does he shave
himself? The paradox is that the barber shaves himself in the case that he
doesn't, and doesn't if he does. The truth value of this proposition seems to
flip-flop back and forth, from yes to no to yes again, forever, never settling
down and converging to an answer.

Of course, Russell wasn't actually wondering about barbers; the question was
posed to underline a problem with the now-called "naive set theory" that was in
vogue at the time. We call it naive set theory these days because it allowed for
paradoxes like the one above, and paradoxes are anathema to mathematics. Once
you have a contradiction, the entire mathematical system falls apart and it's
possible to prove anything whatsoever. As you can imagine, there is great value
in contradictions to politicians, but thankfully they are not the ones in charge
of mathematics.

This exact problem can happen in Agda if we disable the so-called "universe
check," which is a limitation in Agda's type system to disallow contradictions
of exactly this sort. Before continuing with the history of the development of
set theory, let's show that we can express Russell's paradox in Agda, and thus
that this isn't just some moot point that computer scientists do not need to
worry about.

```agda
{-# NO_UNIVERSE_CHECK #-}
record M : Set where
  inductive
  pattern
  constructor m
  field
    bigger : Set
    refine : bigger → M

open import Data.Empty

⊘ : M
⊘ = m ⊥ ⊥-elim

open import Data.Unit

[⊘] : M
[⊘] = m ⊤ λ _ → ⊘

open import Data.Bool

[⊘,[⊘]] : M
[⊘,[⊘]] = m Bool λ { false → ⊘
                   ; true → [⊘]
                   }

open import Data.Product
open import Relation.Binary.PropositionalEquality

_∈_ : M → M → Set
a ∈ m I f = ∃[ i ] (a ≡ f i)

_ : ⊘ ∈ [⊘]
_ = tt , refl

open import Relation.Nullary

_∉_ : M → M → Set
x ∉ y = ¬ (x ∈ y)

⊘-empty : {s : M} → s ∉ ⊘
⊘-empty ()

russell : M
russell = m (Σ M λ s → s ∉ s) proj₁

lemma₁ : {X : M} → X ∈ X → X ∉ russell
lemma₁ x ((fst , snd₁) , refl) = snd₁ x

lemma₂ : {X : M} → X ∉ X → X ∈ russell
lemma₂ {X} x = (X , x) , refl

lemma₃ : russell ∉ russell
lemma₃ x = lemma₁ x x

paradox : ⊥
paradox = lemma₃ (lemma₂ lemma₃)
```

Russell's solution to the barber paradox was the realization that not all sets
are created equally. Some collections are "too big" to be sets. There is no "set
of all sets" because such a thing is too big. Therefore, the question of the
barber who doesn't cut his own hair is swept under the rug, much like the hair
at actual barbershops.

But this only punts on the problem. What is the corresponding mathematical
object of the "set of all sets?" The trick is to build a hierarchy of sets. The
everyday sets we talk about are `Set₀`. `Set₀` itself is contained by `Set₁`,
which is contained by `Set₂`, and so on and so forth. Mathematicians have built
an infinite hierarchy of ever-larger sets, in essence, putting a type-system on
top of sets to ensure no set is able to quantify over itself. Of course,
sometimes it is nice to be able to quantify over sets, and thus the hierarchy
allows us to quantify over sets of a given size by stepping up one tier in the
hierarchy.

In Agda, we take exactly the same solution. In every day typing contexts, you'll
usually be using `Set` (a synonym, for `Set₀`) directly, without any problems.
Where things begin to get a bit messy is when you need to quantify over types;
often when you try to put a `Set` inside of a record. But the record itself
needs to produce a set of some sort. If it produced a `Set` then you could put
the record inside of itself, and then some clever person would start to ask if
you could re-express Russell's paradox, which of course you could.

Agda gives us an infinite hierarchy of `Set` subscripts, everything from `Set₀`
to `Set₄₂` and beyond, but these are all individual identifiers, and thus, we
are unable to abstract over them. Instead, Agda also have special syntax,
allowing you to refer to `Set₂` as `Set 2`, and likewise for every other
subscripted universe. But this `2` is no ordinary number; it is a `Level`
(itself provided in a module of the same name.)

Levels have a minimal interface; we are provided with only three introduction
forms for working with them, and absolutely no elimination forms. We have `zero
: Level`, which corresponds to the lowest level of the universe hierarchy. We
also have `suc : Level → Level` which increments the hierarchy by one. And
finally, we have `_⊔_ : Level → Level → Level` which takes the maximum of its
two level arguments.

Thus you will often see code of the form:

```agda
open import Level

_ : {c ℓ : Level}
  → (A : Set c)
  → (_≈_ : A → A → Set ℓ)
  → Set (suc c ⊔ ℓ)
_ = unimportant
```

which binds two levels, `c` and `ℓ` (the latter symbol is commonly used for
abstract level variables), and then binds some sets parameterized by those
levels. Finally, this code returns a set which is some arithmetic over the
levels in order to get everything to typecheck. This is more something to watch
out for when you're reading code; you can safely ignore all the `Level` stuff,
that's merely there to ensure nobody has snuck any paradoxes in without our
noticing.

When you're writing Agda, you can start by putting everything in `Set`, and
generalizing if the typechecker ever gets mad at you for violating one `Level`
rule or another. When that happens, simply introduce a new implicit `Level` for
each `Set` you're binding, and then follow the type errors until everything
compiles again. Sometimes the errors might be incomplete, complaining that the
level you gave it is not the level it should be. Just make the change and try
again; sometimes Agda will further complain, giving you an even higher bound
that you must respect in your level algebra. It can be frustrating, but keep
playing along, and Agda will eventually stop complaining.

As you gain more proficiency in Agda, you'll often find yourself trying to do
interesting things with `Set`s, like putting them inside of data structures. If
you wrote the data structures naively over `Set`, this will invoke the ire of
the universe checker, and Agda will refuse to compile your program. After
running into this problem a few times, you will begin making all of your
programs universe-polymorphic in the same way that strongly-typed functional
programmers automatically make their programs type-polymorphic whenever
possible. The result is being able to reuse code you wrote to operate over
values when you later decide you also need to be able to operate over types. A
little discipline in advance goes a long way.


