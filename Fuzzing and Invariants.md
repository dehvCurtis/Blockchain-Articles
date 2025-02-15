## Testing the Unknown

Often, hacks result from scenarios you didn't anticipate or consider for testing. But what if you could write a test that checks for every possible scenario, not just one? Welcome to the world of fuzz testing.

## What Is Fuzz Testing?

Also known as fuzzing, this technique involves supplying random data to your system in an attempt to break it. Imagine your code is an indestructible balloon. Fuzzing involves applying random actions (like poking, squeezing, or even kicking) to the balloon with the sole intention of breaking it.

This makes fuzzing a valuable technique for unearthing unexpected application failures. This lesson will guide you through the concept and practical application of fuzz testing.

## The Fundamental Principle: Testing Invariants

Each system, from a function to an entire program, has an integral property, often referred to as the invariant. This property must always hold true. For instance, consider a function called `doStuff` that should always return zero, regardless of the value of the input. In such a case, returning zero would be the invariant of that function.

### Example:
```solidity
function doStuff(uint256 data) public {
    if (data == 2){
        shouldAlwaysBeZero = 1;
    }
    if(hiddenValue == 7) {
        shouldAlwaysBeZero = 1;
    }
    hiddenValue = data;
}
```

A unit test for this function would look something like this:
```solidity
function testIsAlwaysGetZero() public {
    uint256 data = 0;
    exampleContract.doStuff(data);
    assert(exampleContract.shouldAlwaysBeZero() == 0);
}
```

The above test will pass because, in that specific situation (where `data == 0`), our invariant isn’t broken.

However, what happens if our input is `2`? We get `shouldAlwaysBeZero` as `1`, which violates our invariant!

Of course, this is a simple example. But what if we have a function that is more complex? Writing a test case for every scenario could be tedious or impossible. We need a more programmatic approach to test these cases en masse.

## Introducing Fuzz Tests and Invariant Tests

There are two primary methodologies when dealing with edge cases: fuzz tests/invariant tests and symbolic execution (which we’ll cover another day).

> "Fuzz testing and invariant testing are great tools to assess the robustness of your code."

Let’s consider an example of a fuzz test in Foundry. Here, we set our data directly in the test parameter, allowing Foundry to automate the process of providing random input data during tests.

```solidity
function testIsAlwaysGetZeroFuzz(uint256 data) public {
    exampleContract.doStuff(data);
    assert(exampleContract.shouldAlwaysBeZero() == 0);
}
```

Foundry will automatically randomize `data` and use numerous examples to run through the test script. This test will be supplied random data from `0` to `uint256.max()`, as many times as you’ve configured runs.

> Reminder: You can configure the number of runs in your `foundry.toml` under the `[fuzz]` variable.

Notably, this pseudo-random mechanism is not exhaustive. It won’t provide a scenario for every possible data input. That’s why further understanding of how the fuzzer generates random data is crucial.

## Stateless Fuzzing versus Stateful Fuzzing

Fuzzing comes in different flavors, one of which is **stateless fuzzing**. Another important variant is **stateful fuzzing**.

### Stateless Fuzzing
Stateless fuzzing resets the contract state for each new run, treating every test as independent. This is useful when testing isolated functions.

### Stateful Fuzzing
Stateful fuzzing, instead of resetting the contract state for each run, uses the ending state of the previous run as the starting state for the next. This is important for situations where contract state affects subsequent executions.

A stateful fuzz test creates an interlocking sequence of function calls throughout a single run. Achieving this in Foundry requires using the `invariant` keyword and a bit of setup.

First, import `StdInvariant` from `forge-std` and inherit it in our test contract.

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {StdInvariant} from "forge-std/StdInvariant.sol";

contract MyContractTest is StdInvariant, Test {
    ...
}
```

Then, in the setup of our test contract, we need to tell Foundry which contract we’ll be calling random functions on.

```solidity
function setUp() public {
    exampleContract = new MyContract();
    targetContract(address(exampleContract));
}
```

Now, our stateful fuzz test will look like this:

```solidity
function invariant_testAlwaysReturnsZero() public {
    assert(exampleContract.shouldAlwaysBeZero() == 0);
}
```

With the above test, Foundry will call random functions on the `targetContract` (in our case, `doStuff` repeatedly). If other functions exist, they would be called in a random order, each receiving random data.

## Summary

Fuzz testing is about understanding your system’s invariants and writing tests that execute numerous scenarios. This can be done through:

1. **Stateless fuzzing** – Providing random data with each run independent of the last.
2. **Stateful fuzzing** – Allowing both random data and random function calls to interact sequentially within a contract.

This is becoming the new standard for Web3 security.

Going forward, aim to fully understand the invariants in the systems you're working on and write fuzz tests to ensure they are not broken.

> "Fuzz testing is a technique that some of the top protocols are yet to adopt, yet it can aid in discovering high-severity vulnerabilities in smart contracts." – Alex Rohn, Co-founder at Cyfrin.

### Next Lesson:
We’re going to discuss common Ethereum Improvement Proposals (EIPs)!

