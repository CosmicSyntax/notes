# Subtyping and Variance in Rust

## Subtyping

```rust
// This is psuedo-rust code
trait Animal {}
trait Cat: Animal {}
trait Dog: Animal {}
```

Cat is a subtype of Animal, and Animal is supertype to Cat.
Within safe Rust, the compiler will take care of corner cases,
but in unsafe code, you can get a meowing dog (i.e., UB).

Applying subtyping in a naive way using "find and replace"...

```rust
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
```

We need something more robust... Variance.

## Subtyping in Rust

Subtyping occurs in lifetimes in Rust.

'big: 'small (big outlives small / big contains small)

    1. 'big is a subtype for 'small
    2. Think of: Cat is an Animal and more... 'big is 'small and more.

If someone wants a reference for 'small, they mean a reference that lives
at least 'small. So, it's ok to forget somethings lives 'big and just
remember it lives 'small. Meowing dog happens when 'small is used when
'big actually needed, creating a dangling reference.

Lifetime is not a type... so to apply liftime subtyping...

## Variance

A property that type constructors have with respect to their arguments.

Type constructor (F<T>) -> any generic type with unbound arguments. Examples:

    * Vec: type T
    * & and &mut: lifetime and pointer to type

A type contructor F's variance is how the subtyping of its inputs affect the subtyping of its outputs.

Give two types: Sub and Super, where...

    * Sub is subtype of Super
    * Sub: Super
    * 'big: 'small
    * Sub outlives Super

Three kinds variances in Rust

    1. covariance
        F<Sub>: F<Super> (subtyping "passes through")
    2. contravariance
        F<Super>: F<Sub> (subtyping is "inverted")
    3. invariance
        No subtyping relationship

Table of important variances in Rust

F\<T\>          | 'a        | T             | U
--------------- | --------- | ------------- | --------
&'a T           | covariant | covariant     |	
&'a mut T       | covariant | invariant     |	
Box\<T\>        |           | covariant     |	
Vec\<T\>        |           | covariant     |	
UnsafeCell\<T\> |           | invariant     |	
Cell\<T\>       |           | invariant     |
fn(T) -> U      |           | contravariant | covariant
*const T         |           | covariant     |
*mut T           |           | invariant     |	


