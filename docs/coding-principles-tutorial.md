# Coding Principles: A Tutorial

> **Who this is for:** developers who are comfortable writing Python but haven't thought much about
> type design. Each section opens with a realistic bug, then shows how the principle eliminates it
> by construction. The [reference document](../CODING_PRINCIPLES.md) gives you the rules; this one
> teaches you why they exist.

## The one idea underneath everything

Two terms appear throughout this tutorial. Get them right and the rest follows.

An **invariant** is a rule a value must always obey. For a price: must be non-negative. For a
percentage: must be between 0 and 1. For a username: must be non-empty and match a format.
An invariant is a *domain* rule — it comes from the real world, not from the language.

A **naked primitive** is a built-in type (`float`, `int`, `str`, `list`) used to carry a domain
value without encoding the domain's rules. `price: float` is a naked primitive: the type `float`
admits `-3.14`, `inf`, and `nan`, none of which are valid prices.

The rule underneath all four principles:

> **A type should be exactly as wide as its set of valid values — no wider.**

When a type is wider than its domain, invalid values can exist and circulate silently. Narrowing
the type makes those values *inexpressible*: the defect is gone by construction, not by vigilance.

---

## Rule 1 — Give every constrained value a type

### The bug this rule prevents

You're building an e-commerce system. A price is a non-negative number. You represent it as `float`:

```python
def add_prices(a: float, b: float) -> float:
    return a + b
```

Three months later, a bug in a discount function passes `-5.0` where a price was expected.
`add_prices(-5.0, 10.0)` happily returns `5.0`. The invalid value circulated silently through five
functions before manifesting as a wrong total on an invoice. The type said "any float" and you got
any float.

### The fix

Give `Price` its own type that enforces the invariant:

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Price:
    amount: float

    def __post_init__(self) -> None:
        if self.amount < 0:
            raise ValueError("Price must not be negative.")
```

What each part does:

- `@dataclass` generates `__init__`, `__repr__`, and `__eq__` — no boilerplate.
- `frozen=True` makes the instance immutable. Once constructed, `price.amount` cannot change.
  (More on why that matters in Rule 3.)
- `__post_init__` runs *after* `__init__` populates `self.amount`. It is the single place where
  the invariant is enforced.

Your function signature is now honest:

```python
def add_prices(a: Price, b: Price) -> Price:
    return Price(a.amount + b.amount)
```

`Price(-5.0)` raises immediately at the construction site — not five functions downstream. The
invalid value never gets to circulate.

### The same idea in PHP

```php
final class Price {
    public readonly float $amount;

    public function __construct(float $amount) {
        if ($amount < 0) {
            throw new InvalidArgumentException('Price must not be negative.');
        }
        $this->amount = $amount;
    }
}
```

`readonly` gives you the same immutability guarantee as `frozen=True`. `final` prevents a subclass
from weakening the invariant by overriding the constructor.

### The same idea in C++

```cpp
template <typename T>
class Price {
    static_assert(std::is_same<T, float>::value || std::is_same<T, double>::value,
                  "Price requires float or double");
public:
    explicit Price(T amount) : amount_(amount) {
        if (amount < 0) throw std::invalid_argument("Price must not be negative.");
    }
    T amount() const { return amount_; }
private:
    T amount_;
};
```

In C++, this wrapper compiles to the same machine code as a raw `float` — **zero runtime cost**.
There is no reason not to use it.

### When not to

A throwaway script that reads a CSV, computes a sum, and exits doesn't need `Price`. The invariant
is trivial and the code runs once. The machinery adds noise without adding safety.

---

## Rule 2 — Enforce the invariant once, at construction

### The bug this rule prevents

You have several functions operating on prices as `float`. Each one defensively validates its input:

```python
def apply_discount(price: float, discount: float) -> float:
    if price < 0:
        raise ValueError("price must be non-negative")
    if not (0 <= discount <= 1):
        raise ValueError("discount must be between 0 and 1")
    return price * (1 - discount)

def apply_tax(price: float, rate: float) -> float:
    if price < 0:
        raise ValueError("price must be non-negative")
    return price * (1 + rate)

def format_price(price: float, currency: str) -> str:
    if price < 0:
        raise ValueError("price must be non-negative")
    return f"{currency}{price:.2f}"
```

Three problems:

1. The same check is copy-pasted everywhere. Miss it in one new function and you have a hole.
2. The real logic is buried under guard clauses — harder to read and review.
3. Any code that constructs a `float` without going through one of these functions bypasses all
   checks.

### The fix

Validate at construction — once — and trust downstream:

```python
@dataclass(frozen=True)
class Price:
    amount: float
    def __post_init__(self) -> None:
        if self.amount < 0:
            raise ValueError("Price must not be negative.")

@dataclass(frozen=True)
class Discount:
    rate: float
    def __post_init__(self) -> None:
        if not (0 <= self.rate <= 1):
            raise ValueError("Discount rate must be between 0 and 1.")

def apply_discount(price: Price, discount: Discount) -> Price:
    return Price(price.amount * (1 - discount.rate))

def apply_tax(price: Price, rate: float) -> Price:
    return Price(price.amount * (1 + rate))

def format_price(price: Price, currency: str) -> str:
    return f"{currency}{price.amount:.2f}"
```

Every consuming function now holds *evidence* that the value is valid. If `apply_discount` receives
a `Price`, that price was validated at construction. There is nothing to re-check.

### "Parse, don't validate"

This pattern has a name. **Parsing** means converting untrusted input — a raw `float`, a form
string, a CLI argument — into a trusted type at the boundary of your system. Once parsed, the value
carries its own proof of validity everywhere it goes.

**Validating**, by contrast, means re-checking a raw value inside consuming functions. It works,
but it's fragile: you have to remember to do it every time, and every unchecked call path is a
vulnerability.

Parse at the boundary — user input, API responses, database reads. Trust everywhere inside.

---

## Rule 3 — Make invalid values structurally impossible

Rule 2 said: validate once, at construction. Rule 3 says: pick the *strongest* form of that
available to you. There are two.

### Truly structural: the language forbids the invalid state

Some constraints can be expressed in the type system itself, leaving nothing to check at runtime.

An **enum** is the clearest example. If sort direction is either ascending or descending, make it
an enum:

```python
from enum import Enum

class SortDirection(Enum):
    ASCENDING = "ascending"
    DESCENDING = "descending"
```

A `SortDirection` variable *cannot* hold any value other than `ASCENDING` or `DESCENDING`. There
is no check because there is nothing to check: the invalid state is unrepresentable.

Non-nullable types work the same way. In a language where `int` cannot be `None`, there is no
null-check needed — the type system already eliminated that case.

### Single-gate runtime: one constructor, one check, immutable value

Some invariants can't be expressed in the type system. "This `float` is non-negative" is something
most type systems can't say. So you do the next best thing:

```python
@dataclass(frozen=True)
class Price:
    amount: float
    def __post_init__(self) -> None:
        if self.amount < 0:
            raise ValueError("Price must not be negative.")
```

What makes this *structural*:

1. **One gate.** The check happens once, in one place. There is no other way to construct a `Price`
   without going through `__post_init__`.
2. **Immutable.** `frozen=True` means `price.amount` cannot change after construction.
   *Possessing a `Price` instance is proof the check passed.*

That second point is why immutability is load-bearing, not decorative. Without `frozen=True`:

```python
@dataclass   # no frozen=True
class Price:
    amount: float
    def __post_init__(self) -> None:
        if self.amount < 0:
            raise ValueError("Price must not be negative.")

price = Price(10.0)   # check passes
price.amount = -5.0   # no error — the guarantee is silently discarded
```

The constructor's guarantee is erased on the next line. The single gate is no longer enough.

### The payoff

Both mechanisms — truly structural and single-gate runtime — give downstream code the same thing:
evidence. Not "I think this is probably valid" — evidence: possessing the instance proves the
invariant holds. That is what Rule 2's consuming functions are trusting when they stop re-checking.

---

## Rule 4 — No naked primitives; make functions hard to misuse

This rule has two beats. The first is about distinct domain concepts; the second is about functions
and collections.

### Beat 1: Distinct concepts get distinct types

Consider an order-placement function:

```python
def place_order(user_id: int, product_id: int, quantity: int) -> None:
    ...
```

Call it wrong:

```python
place_order(product_id, user_id, quantity)   # compiles, runs, silently wrong
```

No error. No warning. Both IDs are `int`, so they're interchangeable as far as the type system is
concerned. The bug surfaces when you find that order #7 shows user_id 42 and product_id 7 — swapped.

The fix:

```python
@dataclass(frozen=True)
class UserId:
    value: int

@dataclass(frozen=True)
class ProductId:
    value: int

@dataclass(frozen=True)
class Quantity:
    value: int
    def __post_init__(self) -> None:
        if self.value <= 0:
            raise ValueError("Quantity must be positive.")

def place_order(user_id: UserId, product_id: ProductId, quantity: Quantity) -> None:
    ...
```

Now `place_order(ProductId(42), UserId(7), Quantity(3))` is a type error. The transposition is
caught before the function runs. The signature is also self-documenting: you don't need a docstring
to know which argument is which.

### Beat 2: Replace naked callables and bare collections

A `callable` parameter is a naked primitive wearing a function's clothes:

```python
def sort_prices(prices: list[float], comparator: callable) -> list[float]:
    ...
```

This tells the caller nothing about what `comparator` should do. A single-argument key function, a
function that returns a string, and a function that mutates global state all satisfy `callable`. A
wrong-arity function silently produces wrong results; the others may crash or corrupt data.

When the valid choices are a **closed set** — ascending or descending, never anything else — model
them as an enum that *produces* the function:

```python
from enum import Enum
from typing import Callable

class SortDirection(Enum):
    ASCENDING = "ascending"
    DESCENDING = "descending"

    def key(self) -> Callable[["Price"], float]:
        if self == SortDirection.ASCENDING:
            return lambda p: p.amount
        return lambda p: -p.amount

def sort_prices(prices: list[Price], direction: SortDirection) -> list[Price]:
    return sorted(prices, key=direction.key())
```

The caller cannot pass an invalid comparator because the caller doesn't pass a comparator at all —
they pass a `SortDirection`, and the enum is responsible for producing the valid function. You still
get first-class functions; you just guarantee it is one of the valid ones.

The enum is also the right type when the choice is **user-selectable**: a user passes a string
(`"ascending"`), not a function. Something must map that string to behavior at a closed boundary.
The enum *is* that mapping, done once, safely. A naked `callable` can't be user-selected at all —
you'd have to rebuild the same mapping behind it, less safely.

**The same principle applies to collections.** A bare `list[float]` is wider than "a list of
prices":

```python
prices: list[float] = [10.0, -5.0, 3.0]   # invalid entry sneaks in undetected
```

Wrap it in a type that can only hold the right elements:

```python
class PriceList:
    def __init__(self, *prices: Price) -> None:
        self._prices = list(prices)

    def sorted(self, direction: SortDirection) -> "PriceList":
        return PriceList(*sorted(self._prices, key=direction.key()))

    def amounts(self) -> list[float]:
        return [p.amount for p in self._prices]
```

The variadic `*prices: Price` annotation means Python raises `TypeError` if you pass a non-`Price`.
The invalid state is blocked at the boundary, not caught by a validation loop inside.

### PHP version — the full pattern together

PHP's `sort_prices.php` shows all three types working together. `SortDirection` is a native enum;
`PriceList` uses variadic typed parameters to block non-`Price` entries at construction:

```php
enum SortDirection {
    case Ascending;
    case Descending;

    public function comparator(): callable {
        return match ($this) {
            self::Ascending  => static fn(Price $a, Price $b): int => $a->amount <=> $b->amount,
            self::Descending => static fn(Price $a, Price $b): int => $b->amount <=> $a->amount,
        };
    }
}

final class PriceList {
    private readonly array $prices;

    public function __construct(Price ...$prices) {
        $this->prices = $prices;
    }

    public function sorted(SortDirection $direction): self {
        $prices = $this->prices;
        usort($prices, $direction->comparator());
        return new self(...$prices);
    }

    public function amounts(): array {
        return array_map(static fn(Price $p): float => $p->amount, $this->prices);
    }
}
```

`SortDirection::Ascending` and `SortDirection::Descending` are the only legal values. `PriceList`
can only hold `Price` instances. Both guarantees are structural — enforced by the type, not by
downstream vigilance.

---

## Cost and judgment

These principles apply to **production-quality code**. For throwaway scripts, prototypes, and
one-off glue, skip them — the machinery adds noise without value when the code runs once and is
discarded.

When you're writing maintainable code, the cost question has two inputs:

**Language cost.** In C++, wrapping a primitive in a type compiles away entirely — zero runtime
overhead. In Python and PHP, there is a small allocation and dispatch cost. Pay it for values that
carry real invariants or that are easy to mix up. Don't pay it for every `int` in a utility script.

**Context cost.** Production code serves real users, runs at scale, and lives for years. A `Price`
that can be negative is a financial correctness bug. A `UserId` and `ProductId` that are
interchangeable is a data integrity bug. The machinery cost is small compared to those bugs. A
throwaway script has neither the risk nor the longevity — the trade-off flips.

### Smells that suggest a type is too wide

- A naked `float` or `int` carrying money, a rate, or a ratio
- A `str` carrying a format-constrained value (email, URL, ISO date) passed around raw
- A `callable` parameter when the valid choices are a finite, enumerable set
- A `list` or `dict` where you mean "a collection of X"
- Two or more parameters of the same primitive type in the same function signature —
  high risk of transposition bugs like the `UserId`/`ProductId` example above

### The question worth asking every time

> Does this type admit values the domain forbids?

If yes, that is the defect. Whether you fix it right now depends on cost, context, and stakes. But
the question itself costs nothing, and asking it habitually is how you build the instinct.

---

*This tutorial covers the same ground as [`CODING_PRINCIPLES.md`](../CODING_PRINCIPLES.md), which
is the authoritative reference. Read that for the concise rules; read this for the reasoning behind
them.*
