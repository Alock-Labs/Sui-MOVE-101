# Sui Move 101
*This tutorial is presented by [Alock Labs](https://www.alock.club). Under active development.*

Welcome to Sui Move 101! This repository is designed for beginners who want to learn Move, the smart contract language used on the Sui blockchain. Reference: [The Move Book](https://move-book.com/index.html). 

While The Move Book is a good source to learn Sui Move, it can be a bit overwhelming for beginners. This repository aims to provide a more structured and beginner-friendly guide to help you get started with Sui Move.

## 0. Basic Concepts
Sui Move is a fork of the Rust programming language for developing dApps on the Sui blockchain. It has an **object-centric data structures**. Note that while other blockchains like Aptos also uses Move, the Move language used on Sui is different, and this guide is specific to Sui Move. 

If you are familiar with Solidity, you will find that Sui Move is quite different. Sui doesn't have the concept of a **contract**. As a developer, you don't deploy a contract to the blockchain. Instead, you publish a **package**. A package is a collection of **modules**, which define a number of **structs and functions**.

![Sui smart contract architecture](https://github.com/Alock-Labs/Sui-MOVE-101/blob/main/images/Sui-smart-contract-architecture.png?raw=true)

### Objects
An **object** is an instance of a **struct** (that has the `key` ability, but you can ignore this for now). In a **function**, when you create an instance of a struct, you create an object. An object has its owner. 

<ins>Object ownership rule in the simplest words</ins>: When an object is created inside a function, the function temporarily owns the object. When the function **returns** the object, the object is owned by the function caller. The owner can again pass the object to another function, while the function executes, the object is again owned by the function and it can do anything with the object.

An object has two categories of use cases:
1. To actually represent something with value in the blockchain (e.g. a weapon in a game, a number of tokens, etc.)
2. To use as an authentication pass to perform some operations. Think of it as an ID card that can be used to access certain functions. 

The second category above is particularly important because it represents the access control mechanism of Sui Move. When a function requires an object as an argument, it means that only the owner of the object can call the function.

## 1. Struct and ability
**struct** is the most basic element in Sui Move, therefore it is the first thing we should learn. A struct is used to define data structures that can contain multiple fields.

In Sui Move, each struct can have its abilities. Abilities define what you can do with a struct. There are four abilities in Sui Move, which are **key**, **store**, **copy**, and **drop**.

An example struct named `Foo` with abilities key and store is shown below:

```rust
public struct Foo has key, store { // key and store are abilities
    id: UID,
    s: String
}
```

Let's have a look at the four abilities:

### `key` ability
The first field of a struct with the `key` ability must be named `id` and of type `UID`. `UID` is a unique identifier that is used to identify each instance of the struct (i.e., each object) across the Sui blockchain.
Having a `UID` meaning that the blockchain can identify this struct instance, which is an essential requirement to become an `object`. If your struct instances are objects that can be owned and transferred (e.g. player A creates and owns a weapon object, but later it can be transferred to another player B), you should give the struct the key ability. 

### `store` ability
It simply means that the struct can be stored as a field of another struct. See example below:

```rust
public struct Foo has store {
    x: u64,
    y: u64
}

public struct Bar has key {
    id: UID,
    foo: Foo  // Foo can be stored as a field of Bar because Foo has the store ability
}
```

### `copy` ability
It means that when an object reference is passed into a function, the object can be dereferenced and the value can be copied into a new object. 

<ins>The `copy` ability cannot be used with the `key` ability.</ins> Because having the key ability means that the object has a `UID` field, which is unique and cannot be copied.

### `drop` ability
Objects from structs with the `drop` ability can be destroyed implicitly when it is owned by a function and the function returns.

For objects without the `drop` ability, if a function wants to destroy it, the function has to explicitly unpack it. See example below:

```rust
public struct Foo {
    x: u64,
    y: u64
}

public fun destroy_foo(foo: Foo) {
    let Foo { x, y } = foo; // unpack the object by declaring x and y
}
```

But for objects with the `drop` ability, you don't have to unpack it, and if the function does not return the object, it will be destroyed automatically.

### Special Struct: One Time Witness
One time witness is a special struct that cannot be manually created and can only be used once. It is used as an argument in a function to ensure that the function can only be called once. A one time witness must be a struct with only the `drop` ability, has no fields, and is named after the module with all uppercase letters. To get an instance of a one time witness, you need to add it as the first argument to the module init function.

```rust
module Alock::foo;

public struct FOO has drop {} // FOO is a one time witness because it satisfies the requirements

public fun init(foo: FOO) {  // bar can only be called once because it requires a one time witness
    // do something
}
```


## 2. Object State Changes
In the last section we assumed that an object usually owned by a single account. But an object also be shared and frozen.

### Shared Object
An object can be in a shared state in the Sui blockchain. **If an object is shared, any account can access the object and use the object's mutable reference as an argument in functions.**

To create a shared object, you need to use `transfer::share_object(your_object);` **in the function that creates the object**. You don't need to return the shared object in the function. Example below:

```rust
public fun create_shared_object() {
    let obj = Foo { id: UID::new(), s: "Hello".to_string() };
    let shared_obj = transfer::share_object(obj); // create a shared object
}
```

### Frozen Object
An object can be in a frozen state in the Sui blockchain. If an object is frozen, the object is publicly readable but cannot be modified. Freezing an object can be done by using `transfer::freeze_object(your_object);`, and it doesn't have to be in the function that creates the object. You don't need to return the frozen object in the function. Example below:

```rust
public fun freeze_gift(gift: Gift) {
    transfer::freeze_object(gift);
}
```

Frozen objects can be used as an **immutable** reference in a function argument.

## 3. Function

### Function Visibility
In Sui Move, functions can be either:
- **public**: The function can be called from outside the module and the package. 
```rust
public fun foo() {
    // public function is with the public keyword
}
```
- **package**: The function can be called from all modules in the same package.
```rust
public(package) fun foo() {
    // package function is with the public keyword along with the package modifier
}
```
- **private**: The function can only be called from within the module.
```rust
fun foo() {
    // private function is without the public keyword
}
```