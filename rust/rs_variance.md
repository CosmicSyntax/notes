### Subtyping and Variance in Rust

## Subtyping

trait Animal {}
trait Cat: Animal {}
trait Dog: Animal {}

Cat is a subtype of Animal, and Animal is supertype to Cat.
Within safe Rust, the compiler will take care of corner cases,
but in unsafe code, you can get a meowing dog (i.e., UB).

Applying subtyping in a naive way using "find and replace"...

fn evil_feeder(pet: &mut Animal) {
    let spike: Dog = ...;

    // `pet` is an Animal, and Dog is a subtype of Animal,
    // so this should be fine, right..?
    *pet = spike;
}

fn main() {
    let mut mr_snuggles: Cat = ...;
    evil_feeder(&mut mr_snuggles);  // Replaces mr_snuggles with a Dog
    mr_snuggles.meow();             // OH NO, MEOWING DOG!
}

We need something more robust... Variance.

## Subtyping in Rust

Subtyping occurs in lifetimes in Rust.

'big: 'small (big outlives small / big contains small)
    'big is a subtype fo 'small
        Think of: Cat is an Animal and more... 'big is 'small and more.
If someone wants a reference for 'small, they mean a reference that lives
at least 'small. So, it's ok to forget somethings lives 'big and just
remember it lives 'small. Meowing dog happens when 'small is used when
'big, creating a danglig reference.

Lifetime is not a type... so to apply liftime subtyping...

## Variance

1. Type constructor (F<T>) -> any generic type with unbound arguments.
    Vec: type T
    & and &mut: lifetime and pointer to type

2. Three variance in Rust

    Sub is subtype of Super
    Sub: Super
    'big: 'small

    1. covariance
    2. contravariance
    3. invariance
