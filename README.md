# Rust-Cpp-Benchmarks
An Evaluation of Rust and C++ as System Level Languages for their Security and Performance. 

## Type Safety
## Concurrency
## Buffer Overflow
A common security vulnerability is the undefined behavior when a buffer overflow occurs.
In Rust:
```
fn buffer_overflow() {
    let mut buffer = [0_u8; 3];
    unsafe {
        buffer[10] = 42;
    }
}
```

This generates an error at compile time:
```
error: this operation will panic at runtime
 --> src/main.rs:7:9
  |
7 |         buffer[10] = 42;
  |         ^^^^^^^^^^ index out of bounds: the length is 3 but the index is 10
  |
  = note: `#[deny(unconditional_panic)]` on by default

warning: `rs-benchmarks` (bin "rs-benchmarks") generated 1 warning
error: could not compile `rs-benchmarks` (bin "rs-benchmarks") due to previous error; 1 warning emitted

```
