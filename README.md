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


### Buffer Overflow in Rust

**Rust Code:**
```rust
// Buffer Overflow in Rust
fn buffer_overflow(index: usize) {
    let mut buffer = [0u8; 3];
    unsafe {
        buffer[index] = 42;
    }
}

fn main() {
    // Attempt to trigger buffer overflow
    buffer_overflow(4);
}
```
Rust Documentation:
In Rust, buffer overflows are not undefined behavior; they are checked at runtime and result in a panic. The compiler injects bounds checks so that at runtime, overflows do not occur; instead, the panic handler is invoked.

thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 4', src/main.rs:7:9
### Null Pointer Dereferencing in Rust
Rust Code:

```
// Null Pointer Dereferencing in Rust
fn null_pointer_dereference(ptr: *const i32) {
    unsafe {
        let value = *ptr; // Dereference null pointer
        println!("Dereferenced Value: {}", value);
    }
}

fn main() {
    let null_ptr: *const i32 = std::ptr::null();
    // Attempt to dereference null pointer
    null_pointer_dereference(null_ptr);
}
```
Rust Documentation:
In Rust, dereferencing a null pointer is unsafe and can result in undefined behavior. The compiler does not automatically catch such issues, and it is the responsibility of the developer to ensure pointer validity.

error: attempt to dereference a null pointer: 0x0
### Memory Leak in Rust
Rust Code:

```
// Memory Leak in Rust
fn memory_leak() {
    let data = Box::new(42); // Allocate memory on the heap
    // Memory is not deallocated
}

fn main() {
    // Trigger memory leak
    memory_leak();
}
```
Rust Documentation:
In Rust, memory leaks are less common due to ownership and borrowing concepts. However, the developer must explicitly manage memory deallocation using mechanisms like std::mem::drop or smart pointers. The code above intentionally leaks memory by not deallocating the heap-allocated memory.

### Use After Free in Rust
Rust Code:
```
// Use After Free in Rust
fn use_after_free() {
    let data = Box::new(42); // Allocate memory on the heap
    // Memory is deallocated after this block

    // Attempt to use the memory after deallocation
    let value = *data; // Use after free
    println!("Value: {}", value);
}

fn main() {
    // Trigger use after free
    use_after_free();
}
```
Rust Documentation:
In Rust, attempting to use memory after it has been freed is unsafe and can result in undefined behavior. Rust ownership system helps prevent use after free issues, but it's crucial for developers to manage memory correctly.

error: use of possibly-dangling pointer: `data`
### Buffer Overflow in C++
C++ Code:
```
#include <iostream>
#include <vector>

// Buffer Overflow in C++
void buffer_overflow(int index) {
    std::vector<int> buffer(3, 0);
    buffer[index] = 42;
}

int main() {
    // Attempt to trigger buffer overflow
    buffer_overflow(4);
    return 0;
}
```
C++ Documentation:
In C++, buffer overflows are typically undefined behavior and can lead to memory corruption. The C++ compiler may not perform bounds checking by default, leaving it to the developer's responsibility. In the provided example, accessing an out-of-bounds index can result in unpredictable behavior.

### Null Pointer Dereferencing in C++
C++ Code:
```
#include <iostream>

// Null Pointer Dereferencing in C++
void null_pointer_dereference(int* ptr) {
    // Attempt to dereference null pointer
    int value = *ptr;
    std::cout << "Dereferenced Value: " << value << std::endl;
}

int main() {
    int* null_ptr = nullptr;
    // Attempt to dereference null pointer
    null_pointer_dereference(null_ptr);
    return 0;
}
```
C++ Documentation:
In C++, dereferencing a null pointer is undefined behavior. The compiler may not catch such issues, and it is the developer's responsibility to ensure the pointer's validity. In the provided example, attempting to dereference a null pointer can result in unpredictable behavior.

### Memory Leak in C++
C++ Code:

```
#include <iostream>

// Memory Leak in C++
void memory_leak() {
    int* data = new int(42); // Allocate memory on the heap
    // Memory is not deallocated
}

int main() {
    // Trigger memory leak
    memory_leak();
    return 0;
}
```
C++ Documentation:
In C++, managing memory is the developer's responsibility. The provided example intentionally leaks memory by not deallocating the heap-allocated memory. This can lead to long-running programs consuming more and more memory, potentially causing system instability.

### Use After Free in C++
```
#include <iostream>

// Use After Free in C++
void use_after_free() {
    int* data = new int(42); // Allocate memory on the heap
    delete data; // Deallocate memory

    // Attempt to use the memory after deallocation
    int value = *data; // Use after free
    std::cout << "Value: " << value << std::endl;
}

int main() {
    // Trigger use after free
    use_after_free();
    return 0;
}
```
###
In C++, attempting to use memory after it has been freed is undefined behavior. The example demonstrates using memory after it has been deallocated, which can lead to unpredictable program behavior. Proper memory management, including avoiding use after free, is crucial in C++ programming.

