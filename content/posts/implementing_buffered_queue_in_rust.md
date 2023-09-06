+++
title = "Implementing a Naive Buffered Queue in Rust"
description = "Writing a custom implementation of a simple blocking, buffered queue in Rust"
date = "2023-09-06"
+++

## Introduction

O’Reilly’s `Programming Rust` book walks us through optimizing a part of a pipeline, in Chapter 19 `Concurrency`. It explains how a channel-based pipeline can encounter slowdowns and high memory usage if one of the consumer threads is much slower than one of the producer threads. The producer keeps adding tasks to the queue, but the consumer is unable to consume them at a satisfactory pace. The queue will have a large amount of unconsumed data causing high-memory usage. Defining fixed capacities will lower memory consumption in applications without affecting the latencies since the consumer already processes at its own fixed pace.

I had known about queues but had never thought about them in a larger scope, so I thought attempting a custom implementation would be a good way to learn more. I received a lot of help from the Rust community for this project, allowing me to better understand the concepts and improve my code :)  

## Overview

We will walk through the implementation of a simple multi-threaded, blocking, buffered queue. The Producer thread will push elements till the queue is at capacity, and block until the queue has space again. Similarly, the Consumer thread will consume elements till the queue is empty, and block until it has elements again. We do not persist the threads once the input stream is extinguished.
  
## Declaring our Types

We can create a new project with `cargo new buffered-queue-rs` and put our queue logic in `src/lib.rs`, marking all code inside the file as library code and making it accessible to the whole project by importing it with the project name specified in the `cargo new` command.

Add the following imports to the file:

```rust
use std::collections::VecDeque;
use std::sync::atomic::{AtomicBool, Ordering};
use std::sync::{Arc, Condvar, Mutex, MutexGuard};
```

Next, we will define the types for our buffered queue implementation:

```rust
pub struct Producer<T>(Arc<BufferedQueue<T>>);

pub struct Consumer<T>(Arc<BufferedQueue<T>>);

pub struct BufferedQueue<T> {
    data: Mutex<VecDeque<T>>,
    pub capacity: usize,
    pub is_full: Mutex<bool>,
    pub is_full_signal: Condvar,
    pub is_empty: Mutex<bool>,
    pub is_empty_signal: Condvar,
    pub elements_processed: AtomicBool,
}
```

These are [generic](https://doc.rust-lang.org/book/ch10-01-syntax.html "https://doc.rust-lang.org/book/ch10-01-syntax.html") types, signified by the type parameter `<T>`, and can be used with any type as we have not defined any constraints on the type `T`.

`Producer` and `Consumer` follow the [NewType](https://rust-unofficial.github.io/patterns/patterns/behavioural/newtype.html "https://rust-unofficial.github.io/patterns/patterns/behavioural/newtype.html") pattern, allowing us to specify special behaviour on the wrapped type. It will help us separate producer and consumer concerns.

All the defined types use an [Arc](https://doc.rust-lang.org/std/sync/struct.Arc.html "https://doc.rust-lang.org/std/sync/struct.Arc.html"), a special pointer type that enables cheap shared access to data. It also allows sharing its pointer values across threads, even though the wrapped value might not be shareable. It maintains a reference counter for each reference active in memory, similar to Python objects.

Our internal queue implementation `data` is a double-ended queue, held by a mutex to prevent data inconsistencies and enforce exclusive data access. `capacity` is the user-defined maximum capacity for our queue. `usize` data type ensures that the value cannot be negative. `is_full` and `is_empty` indicate the queue’s current state. They will be used by the `is_full_signal` and `is_empty_signal` [Condvars](https://doc.rust-lang.org/std/sync/struct.Condvar.html "https://doc.rust-lang.org/std/sync/struct.Condvar.html") to allow the producer and consumer threads to wait until the queue is in their desired state. `elements_processed` is an [AtomicBool](https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html "https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html") and is thread-safe.  

The `Operation` enum type will signal the queue’s state updates to listening threads. It maps to the queue’s push and pop operations:

```rust
enum Operation<'a> {
    Push { is_full_flag: MutexGuard<'a, bool> },
    Pop { is_empty_flag: MutexGuard<'a, bool> },
}
```

Acquiring the lock on a mutex returns a [MutexGuard](https://doc.rust-lang.org/std/sync/struct.MutexGuard.html "https://doc.rust-lang.org/std/sync/struct.MutexGuard.html"), a thin wrapper around the value held by the mutex. The [lifetime specifier](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html "https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html") `<’a>`  in the type definition indicates how long the boolean flags are going to stay in memory. They are now associated with the enum variants and their held locks will be unlocked when the enum variants go out of scope.

We can see Rust’s powerful enums here, as we can add data on individual variants like we would do with a struct.

## Defining Producer and Consumer Logic

Producer and consumer have a similar logical flow. Both have 2 methods, the `len` method is common to both types and wraps a call to `BufferedQueue`‘s `len` method.  

### Producer

Producer’s implementation is:

```rust
impl<T> Producer<T> {
    pub fn push(&self, value: T) {
        let mut queue_is_full = self.0.is_full.lock().unwrap();
        while *queue_is_full {
            queue_is_full = self.0.is_full_signal.wait(queue_is_full).unwrap();
        }

        let mut queue = self.0.data.lock().unwrap();
        queue.push_back(value);
        println!("pushed element");

        self.0.signal_queue_changes(
            queue,
            Operation::Push {
                is_full_flag: queue_is_full,
            },
        );
    }

    pub fn len(&self) -> usize {
        self.0.len()
    }
}
```

`self.0` accesses the Producer’s first value in the tuple – the buffered queue Arc, to access its fields and methods.

We first get the `queue_is_full` boolean value and check whether the queue is full. Code execution will be paused until the queue has space and `queue_is_full` equals `false`. The `wait` method takes a MutexGuard and atomically releases the lock. This enables other threads to update its value. It re-acquires the lock before returning.

We access the internal queue if there is space, push the new element and call the `signal_queue_changes` method that we will define on `BufferedQueue` later.

We will also implement the [Drop](https://doc.rust-lang.org/rust-by-example/trait/drop.html "https://doc.rust-lang.org/rust-by-example/trait/drop.html") trait, which will perform cleanup after our producer is out of scope:

```rust
impl<T> Drop for Producer<T> {
    fn drop(&mut self) {
        self.0.elements_processed.store(true, Ordering::SeqCst);
    }
}
```

We set `elements_processed` value to `true`, indicating that the producer has processed all its elements and is going out of scope. The `Drop` trait ensures that this implementation detail remains associated with the producer.

The `store` method requires a memory ordering, which defines how the memory is organized and ensures that our code avoids race conditions and improper data access across threads. We use the strongest possible ordering, `SeqCst`.

### Consumer

Consumer’s methods are as follows:

```rust
impl<T> Consumer<T> {
    pub fn pop(&self) -> Option<T> {
        let mut queue_is_empty = self.0.is_empty.lock().unwrap();
        while *queue_is_empty {
            if self.0.elements_processed.load(Ordering::SeqCst) {
                return None;
            }
            queue_is_empty = self.0.is_empty_signal.wait(queue_is_empty).unwrap();
        }

        let mut queue = self.0.data.lock().unwrap();
        let popped_element = queue.pop_front();
        println!("popped element");

        self.0.signal_queue_changes(
            queue,
            Operation::Pop {
                is_empty_flag: queue_is_empty,
            },
        );
        popped_element
    }

    pub fn len(&self) -> usize {
        self.0.len()
    }
}
```

`pop` returns an `Option<T>` meaning it will return an enum variant `Some(T)` from the front of the queue, or `None` if the queue is empty. We wait for the producer to add elements if the queue is currently empty.

Our implementation guarantees that the queue will only pop an element from front of the queue if there is at least one element. We only return `None` once `elements_processed` is `true`, signalling that we can finish our execution.

## Defining BufferedQueue Logic

We will first write a function to create a new buffered queue:

```rust
pub fn buffered_queue<T>(mut capacity: usize) -> (Producer<T>, Consumer<T>) {
    if capacity < 1 {
        eprintln!("capacity cannot be lower than 1, defaulting to 1...");
        capacity = 1
    }

    let buffered_queue = BufferedQueue {
        data: Mutex::new(VecDeque::with_capacity(capacity)),
        capacity,
        is_full: Mutex::new(false),
        is_empty: Mutex::new(true),
        is_full_signal: Condvar::new(),
        is_empty_signal: Condvar::new(),
        elements_processed: AtomicBool::new(false),
    };

    let data = Arc::new(buffered_queue);
    let producer = Producer(data.clone());
    let consumer = Consumer(data);

    (producer, consumer)
}
```

`buffered_queue` takes a capacity and returns a tuple of Producer and Consumer types. It uses 1 as default if the capacity is 0, wraps the buffered queue value in Arc for cheap referencing and thread-safety, makes a reference copy and passes the Arc instances to Producer and Consumer.

Now we will implement its methods:

```rust
impl<T> BufferedQueue<T> {
    fn len(&self) -> usize {
        let queue = self.data.lock().unwrap();
        queue.len()
    }

    fn signal_queue_changes(&self, queue: MutexGuard<'_, VecDeque<T>>, operation: Operation) {
        let is_empty = queue.len() == 0;
        let is_full = queue.len() == self.capacity;

        match operation {
            Operation::Push { mut is_full_flag } => {
                let mut is_empty_flag = self.is_empty.lock().unwrap();
                if *is_empty_flag {
                    *is_empty_flag = false;
                    println!("set is_empty to false");
                    self.is_empty_signal.notify_all();
                }

                if is_full {
                    *is_full_flag = true;
                    self.is_full_signal.notify_all();
                    println!("set is_full to true");
                }
            }

            Operation::Pop { mut is_empty_flag } => {
                let mut is_full_flag = self.is_full.lock().unwrap();
                if *is_full_flag {
                    *is_full_flag = false;
                    println!("set is_full to false");
                    self.is_full_signal.notify_all();
                }

                if is_empty {
                    *is_empty_flag = true;
                    self.is_empty_signal.notify_all();
                    println!("set is_empty to true");
                }
            }
        }
    }
}
```

This method accepts the internal queue and operation enum types. `queue` defines the double-ended queue value after acquiring its mutex lock.

We match the operation value and define the associated boolean values as mutable. Rust allows us to shorthand values if the variable name matches the field name, so we can write `{ mut is_full_flag: is_full_flag }` as  `{ mut is_full_flag }` and so on.

The method checks whether the queue’s state has changed: after an element `Push`, whether the queue is now full and whether it was empty earlier, after an element `Pop`, whether the queue is now empty and whether it was full before. It notifies waiting threads on the state changes if these conditions match, by calling the Condvars’ `notify_all` method.

### Testing Things Out

We can now test the functionality by creating a small simulation.

Add the following imports to the top of the `src/main.rs` file:

```rust
use buffered_queue_rs::buffered_queue;
use std::thread::{self, sleep};
use std::time::Duration;
```

Write the following code in the `src/main.rs` file and replace the existing `main` function:

```rust
fn main() {
    let (producer, consumer) = buffered_queue(3);
    let mut output = Vec::new();

    let producer_handle = thread::spawn(move || {
        println!("initializing producer thread...");

        for num in 1..=5 {
            let processed_num = num * num * num;

            // mock processing behaviour
            sleep(Duration::from_millis(250));
            producer.push(processed_num);
        }
    });

    let consumer_handle = thread::spawn(move || {
        println!("initializing consumer thread...");

        loop {
            let Some(num) = consumer.pop() else {
                    println!("exhausted queue, terminating consumer!\n");
                    return;
            };

            // mock processing behaviour
            sleep(Duration::from_millis(400));
            output.push(num);

            println!(
                "pushed to output num: {}; output_vec len: {}",
                num,
                output.len()
            );
        }
    });

    producer_handle.join().unwrap();
    consumer_handle.join().unwrap();
}
```

We initialize our `producer` and `consumer` values by calling `buffered_queue`, and create a vector for the output produced by the consumer thread.

Then we mark our threads with `move`, meaning they will take ownership of any values used inside them. We use closures to write the thread logic inside the `spawn` blocks.

The producer thread iterates over a range of numbers, mocking input processing flow and pushes values to the queue. Meanwhile, the consumer thread processes values received from the `pop` function, stopping when it receives `None`, which is the signal to terminate execution.  

Finally, we receive return values of type [JoinHandle](https://doc.rust-lang.org/std/thread/struct.JoinHandle.html "https://doc.rust-lang.org/std/thread/struct.JoinHandle.html") from the spawned threads and call `join` on them in the main thread. This ensures that it waits for the other threads to finish before exiting. The `unwrap` call will propagate any runtime errors in these threads to the main thread.

Running `cargo run` will output the following:

```plaintext
initializing consumer thread...
initializing producer thread...
pushed element
set is_empty to false
popped element
set is_empty to true
pushed element
set is_empty to false
pushed to output num: 1; output_vec len: 1
popped element
set is_empty to true
pushed element
set is_empty to false
pushed element
pushed to output num: 8; output_vec len: 2
popped element
pushed element
pushed to output num: 27; output_vec len: 3
popped element
pushed to output num: 64; output_vec len: 4
popped element
set is_empty to true
pushed to output num: 125; output_vec len: 5
exhausted queue, terminating consumer!
```
  
## Conclusion

This was a rewarding exercise for me, as it helped me get more familiar with Rust and concurrency concepts in general. You can find the full code for the exercise [here](https://github.com/dhruv-ahuja/buffered-queue-rs "https://github.com/dhruv-ahuja/buffered-queue-rs"), there are some differences in the code shown here and in the repo.

Thanks for reading my post, any feedback or advice would be appreciated! You can write to me at [dhruvahuja2k@gmail.com](mailto:dhruvahuja2k@gmail.com "mailto:dhruvahuja2k@gmail.com").
