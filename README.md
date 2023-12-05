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

### Race Conditions
Rust Code:
```
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));

    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```
Rust Documentation:
In Rust, the code demonstrates a race condition where multiple threads concurrently access and modify a shared counter without proper synchronization. This can lead to unpredictable results, and in this case, the final count may not be what's expected due to the race condition. In Rust, the code demonstrates a race condition where multiple threads concurrently access and modify a shared counter without proper synchronization. Rust's ownership and borrowing system ensures memory safety but does not prevent race conditions. The expected behavior includes unpredictable results in the final count due to the race condition.

C++ Code:
```
#include <iostream>
#include <thread>
#include <mutex>

int main() {
    std::mutex counterMutex;
    int counter = 0;

    std::vector<std::thread> threads;

    for (int i = 0; i < 10; ++i) {
        threads.emplace_back([&counter, &counterMutex]() {
            std::lock_guard<std::mutex> lock(counterMutex);
            ++counter;
        });
    }

    for (auto& thread : threads) {
        thread.join();
    }

    std::cout << "Result: " << counter << std::endl;

    return 0;
}
```
C++ Documentation:
In C++, the code demonstrates a similar race condition scenario where multiple threads concurrently modify a shared counter without proper synchronization. The final count may not be as expected due to the race condition. In C++, the code also illustrates a race condition scenario where multiple threads concurrently modify a shared counter without proper synchronization. C++ does not inherently prevent race conditions, and the expected behavior includes unpredictable results in the final count due to the race condition.

### Deadlock
Rust Code:
```
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    let resource1 = Arc::new(Mutex::new(()));
    let resource2 = Arc::new(Mutex::new(()));

    let resource1_clone = Arc::clone(&resource1);
    let resource2_clone = Arc::clone(&resource2);

    let handle1 = thread::spawn(move || {
        let _lock1 = resource1_clone.lock().unwrap();
        // Simulate some work
        thread::sleep(std::time::Duration::from_millis(100));
        let _lock2 = resource2_clone.lock().unwrap();
        println!("Thread 1 acquired both resources");
    });

    let handle2 = thread::spawn(move || {
        let _lock1 = resource2.lock().unwrap();
        // Simulate some work
        thread::sleep(std::time::Duration::from_millis(100));
        let _lock2 = resource1.lock().unwrap();
        println!("Thread 2 acquired both resources");
    });

    handle1.join().unwrap();
    handle2.join().unwrap();
}
```
Rust Documentation:
The Rust code demonstrates a potential deadlock scenario where two threads attempt to acquire two mutex-protected resources in a different order. Each thread holds one resource and attempts to acquire the other, resulting in a deadlock where neither thread can proceed. The Rust code showcases a potential deadlock scenario where two threads attempt to acquire two mutex-protected resources in different orders. Rust's ownership system ensures memory safety, but it doesn't inherently prevent deadlocks. The expected behavior involves the program becoming deadlocked, with neither thread able to proceed.

C++ Code:
```
#include <iostream>
#include <thread>
#include <mutex>

int main() {
    std::mutex resource1Mutex;
    std::mutex resource2Mutex;

    std::thread thread1([&resource1Mutex, &resource2Mutex]() {
        std::lock_guard<std::mutex> lock1(resource1Mutex);
        // Simulate some work
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        std::lock_guard<std::mutex> lock2(resource2Mutex);
        std::cout << "Thread 1 acquired both resources" << std::endl;
    });

    std::thread thread2([&resource1Mutex, &resource2Mutex]() {
        std::lock_guard<std::mutex> lock2(resource2Mutex);
        // Simulate some work
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
        std::lock_guard<std::mutex> lock1(resource1Mutex);
        std::cout << "Thread 2 acquired both resources" << std::endl;
    });

    thread1.join();
    thread2.join();

    return 0;
}
```
C++ Documentation:
The C++ code demonstrates a similar deadlock scenario where two threads attempt to acquire two mutex-protected resources in a different order. This can lead to a situation where both threads are waiting for each other, resulting in a deadlock. The C++ code demonstrates a potential deadlock scenario where two threads attempt to acquire two mutex-protected resources in different orders. C++ does not automatically detect or prevent deadlocks. The expected behavior involves the program becoming deadlocked, with neither thread able to proceed.



