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
The Rust compiler can statically determine here that an out of bounds access will occur, 
which generates an error at compile time:
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
Now if we specify the index of the buffer as an argument, as:
```
fn buffer_overflow(index: usize) {
    let mut buffer = [0u8; 3];
    unsafe {
        buffer[index] = 42;
    }
}
```

The compiler can no longer determine statically that this code results in a buffer overflow, therefore it compiles.
Passing in an oversized index to this function:
```
buffer_overflow(4);
```
In Rust, buffer overflows are not undefined behavior; they are checked at runtime and result in a panic. The compiler injects bounds checks so that at runtime, overflows do not occur, rather: the panic handler is invoked.
```
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 4', src/main.rs:7:9
```


