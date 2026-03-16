---
title: "Java 25 -- New Features Overview"
classes: wide
---

Java 25 (JDK 25) introduces improvements across the Java language,
runtime performance, concurrency model, security APIs, and observability
tooling.

This document summarizes the most important updates relevant for
developers.

------------------------------------------------------------------------

# 1. Language Features

## Primitive Types in Pattern Matching (Preview)

Pattern matching has been expanded to support primitive types in
`switch` and other pattern contexts.

Example:

``` java
switch (value) {
    case int i -> System.out.println("Integer: " + i);
    case double d -> System.out.println("Double: " + d);
}
```

Benefits:

-   More consistent pattern matching
-   Simplifies conditional logic
-   Better support for performance‑sensitive code

------------------------------------------------------------------------

## Module Import Declarations

Java now allows importing all exported packages from a module in a
single statement.

``` java
import module java.sql;
```

Advantages:

-   Reduces boilerplate imports
-   Makes modular libraries easier to use
-   Simplifies dependency usage

------------------------------------------------------------------------

## Compact Source Files and Instance Main Methods

This feature removes much of the boilerplate needed for small programs.

Traditional Java:

``` java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

Simplified version:

``` java
void main() {
    System.out.println("Hello World");
}
```

------------------------------------------------------------------------

# 2. Concurrency Improvements

## Structured Concurrency

Structured concurrency simplifies working with groups of concurrent
tasks.

Concept example:

``` java
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    Future<String> user = scope.fork(() -> findUser());
    Future<Integer> order = scope.fork(() -> fetchOrder());

    scope.join();
}
```

Benefits:

-   Simplified thread coordination
-   Improved error handling
-   Better cancellation support

------------------------------------------------------------------------

## Scoped Values

Scoped values provide a safe alternative to `ThreadLocal` variables.

Key characteristics:

-   Immutable data sharing
-   Lower memory overhead
-   Better integration with virtual threads

Example:

``` java
ScopedValue<String> USER = ScopedValue.newInstance();
```

------------------------------------------------------------------------

# 3. Performance Improvements

## Compact Object Headers

Java 25 introduces smaller object headers on 64‑bit architectures.

Benefits:

-   Reduced memory footprint
-   Improved cache efficiency
-   Higher application density

------------------------------------------------------------------------

## Vector API

The Vector API allows developers to write vectorized computations that
map to modern CPU vector instructions.

Use cases:

-   machine learning
-   financial simulations
-   scientific computing
-   AI inference workloads

------------------------------------------------------------------------

# 4. Security Enhancements

## PEM Encoding API

Java introduces a built-in API for reading and writing cryptographic
objects in PEM format.

Supported items include:

-   cryptographic keys
-   certificates
-   certificate revocation lists

------------------------------------------------------------------------

## Key Derivation Function API

Provides standard APIs for deriving secure cryptographic keys from
shared secrets.

Use cases:

-   encryption systems
-   password-based key generation
-   modern cryptographic protocols

------------------------------------------------------------------------

# 5. Observability Improvements

## Java Flight Recorder Enhancements

Java Flight Recorder receives improvements including:

-   CPU-time profiling
-   cooperative stack sampling
-   detailed method timing

These tools improve diagnostics for performance bottlenecks in
production systems.

------------------------------------------------------------------------

# 6. Platform Changes

## Removal of 32‑bit x86 Support

Java 25 removes support for legacy 32‑bit x86 systems, focusing
exclusively on modern 64‑bit architectures.

Benefits:

-   simplified JVM maintenance
-   better performance optimizations
-   reduced platform complexity

------------------------------------------------------------------------

# Conclusion

Java 25 continues improving the platform with stronger support for:

-   concurrency
-   performance optimization
-   cryptography
-   developer productivity
-   runtime observability

These improvements help keep Java competitive for enterprise systems,
cloud workloads, and high-performance computing.
