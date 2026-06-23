# Coding Principles

These apply to production-quality code. **Context matters:** for throwaway scripts,
prototypes, and one-off glue, this level of rigor is unnecessary — don't impose it
where it adds noise without value. The rest of the time, default to it. AI-assisted
programming makes generating rigorous, near-provably-correct code cheap rather than
tedious, so the bar for "worth the effort" is much lower than it used to be.

**The rule underneath all four:** a type should be exactly as wide as its set of valid
values — no wider. Every principle below is a corollary. A type that admits values the
domain forbids is the defect; narrowing it deletes whole classes of bug by construction.
This cuts both ways — sometimes it means *adding* a type (wrap a naked `float` in `Price`),
sometimes *replacing* a permissive built-in with a tighter one (an enum instead of a
`callable`, a `PriceList` instead of a bare `array`). The question is never "more machinery
or less?" — it's "does this type admit values the domain forbids?"

## 1. Give every constrained value a type

Any value that carries an invariant — non-negative, non-empty, bounded range, valid
format, a unit of measure — gets its own type that enforces that invariant. Don't pass
it around as a naked primitive guarded by ad-hoc checks.

```php
// Not this:
function addPrices(float $a, float $b): float { ... }

// This:
final class Price {
    public readonly float $amount;
    public function __construct(float $amount) {
        if ($amount < 0) throw new InvalidArgumentException('Price must not be negative.');
        $this->amount = $amount;
    }
}
function addPrices(Price $a, Price $b): Price { ... }
```

## 2. Enforce the invariant once, at construction

The right place to validate is where the value comes into existence — the constructor of
its type — not inside every function that consumes it. Validating in a consuming function
only protects that one call path; any other code that builds the raw value unchecked is
still a landmine. Once the type guarantees the invariant, every function that takes the
type can simply trust it. ("Parse, don't validate.")

## 3. Make invalid values structurally impossible

Prefer designs where an illegal state cannot be represented at all over designs that
detect illegal states at runtime. Two mechanisms get you there, and the distinction matters:

- **Truly structural** — a sum type / enum cannot hold a case you didn't declare, a
  non-nullable type cannot be null. There is no check because there is nothing to check.
- **Single-gate runtime** — some invariants ("this `float` is non-negative") can't be
  expressed in the type system, so the constructor performs a one-time runtime check. What
  makes *that* structural is that the constructor is the *sole* gate and the value is
  *immutable*: once past it, possessing the instance is itself proof the check passed. This
  is why immutability is load-bearing, not decorative — a `Price` you could mutate after
  construction would throw the guarantee away on the next line.

Either way the payoff is the one principle 2 depends on: downstream code holds *evidence*
the invariant is true, not a promise to re-verify it.

## 4. Avoid naked primitives; make functions hard to misuse

Distinct domain concepts should be distinct types, even when they share an underlying
representation. `Price`, `Quantity`, and `UserId` should not all be interchangeable
`float`s/`int`s/`string`s — separate types stop you from passing the wrong one and make
signatures self-documenting.

This applies to **function types** as much as scalars. A `callable` parameter is a naked
primitive wearing a function's clothes: it guarantees nothing about arity, argument types,
return value, or whether the behavior is even valid (a malformed comparator is undefined
behavior in many sort routines). When the valid choices are a closed set — e.g. sort
ascending or descending — model them as an enum and let it *produce* the function, rather
than accepting an arbitrary `callable`. You still get first-class functions; you just
guarantee it is one of the valid ones. And when the choice is *user-selectable*, the enum
is not optional polish: the user hands you a token — a CLI flag, a form value — not a
function, so something must map that token to behavior at a closed boundary. The enum *is*
that parsed form of the choice; a naked `callable` can't be user-selected at all, so it
doesn't even serve the requirement — you would just rebuild the enum behind it, less safely.
The same goes for collections: a bare `array` / `list` is wider than "a list of `Price`",
so wrap it in a type that can only hold the right elements where the language won't do it
for you.

## On cost: the *what* is universal, the *whether* is local

Asking "is this type wider than its domain?" is always worth doing. Whether it's worth
*fixing* depends on cost — and cost has two inputs. Neither changes the answer to that
question, only whether you act on it:

- **Language.** In C++, wrapping a primitive in a type is effectively **zero-cost**
  (compiled away) — there is no runtime reason not to. In Python and PHP there is a small
  runtime/verbosity cost; pay it for values with real invariants or that are easy to mix up
  — a frozen dataclass (Python), a `final` class with a `readonly` property (PHP).
- **Context.** The caveat at the top is this same axis: a throwaway script doesn't earn the
  machinery a production system does. Same question, lower stakes — so the cost has to clear
  a higher bar before it's worth paying.
