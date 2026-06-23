# Coding Principles

These apply to production-quality code. **Context matters:** for throwaway scripts,
prototypes, and one-off glue, this level of rigor is unnecessary — don't impose it
where it adds noise without value. The rest of the time, default to it. AI-assisted
programming makes generating rigorous, near-provably-correct code cheap rather than
tedious, so the bar for "worth the effort" is much lower than it used to be.

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
detect illegal states at runtime. A `Price` that cannot hold a negative number is better
than a `float` that you remember to check. Lean on the type system, sum types / enums,
non-nullable types, and immutability to close off bad states by construction.

## 4. Avoid naked primitives; make functions hard to misuse

Distinct domain concepts should be distinct types, even when they share an underlying
representation. `Price`, `Quantity`, and `UserId` should not all be interchangeable
`float`s/`int`s/`string`s — separate types stop you from passing the wrong one and make
signatures self-documenting.

- **In C++**, wrapping a primitive in a type is effectively **zero-cost** (compiled away),
  so there is no runtime reason not to.
- **In Python and PHP**, there is a small runtime/verbosity cost. It is still worth paying
  for values with real invariants or that are easy to mix up — use a frozen dataclass
  (Python) or a `final` class with a `readonly` property (PHP).
