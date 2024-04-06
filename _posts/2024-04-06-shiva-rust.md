---
layout: post
title:  "How to Easily Switch from Java to Rust: Tips and Tricks"
date:   2023-07-26 14:30:59 +0500
categories: java, rust
---
After working on two commercial projects in Rust, I got a good practical experience in this language. These were backend services for web applications, where Rust was used for the main business logic and working with databases.

In addition, I created three open source Rust libraries that I published on GitHub. This allowed me to learn more about idiomatic Rust, working with asynchrony, etc.

In general, after working on these projects, as a Java developer, I have accumulated an interesting experience that I would like to share for those who are just starting to learn Rust, coming from the Java world. Here are some helpful tips to help you transition to Rust.

**Do not use &str when writing the first version of the code, it is better to immediately String**

Rust has two types of strings — &str and String. The first is a reference to the string, the second is the owning string. At first glance, &str seems to be the Java equivalent of String, and is best used by default.

However, in practice, it’s easier to start with String. This type is easier to use and behaves like a normal string in Java. And &str requires you to keep track of ownership and references.

Therefore, I recommend using String in the first version of the code, and then optimize bottlenecks with &str.

```rust
fn main() {
    let name = String::from("Alice");
    print_greeting(name);
}

fn print_greeting(name: String) {
    println!("Hello, {}!", name);
}
```

**Easier debugging with derive(Debug)**

In Java, we have debugging tools, and Rust has its own. However, to make debugging easier, you can use the **derive(Debug)** attribute on the structures you want to parse. This will allow you to automatically generate an implementation of the fmt::Debug method that can be used to print the state of an object using the println! function.
```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u32,
}

fn main() {
    let person = Person {
        name: String::from("Bob"),
        age: 30,
    };

    println!("Person: {:?}", person);
}
```
**Don’t use references in function parameters**

Rust doesn’t have garbage collection, so it’s important to manage memory explicitly. It is often recommended to use references like &T to pass data to functions. But at first it is better to avoid this.

Instead, create copies of objects using the clone() method:

```rust
#[derive(Clone)] 
struct MyData {
   field: i32
}

fn process(data: MyData) {
   // работаем с data 
}

fn main() {
   let data = MyData { field: 42 };
   process(data.clone()); 
}
```

This will avoid many common mistakes. Then, when the code works, you can optimize bottlenecks by switching to links.

**Use structs with functions instead of classes**

It’s common in Java to create classes to organize code, and Rust uses structs for the same purpose. Instead of defining methods inside structs, as we do in classes, we define functions and implement them for concrete structs.

Instead of a constructor, they implement the new function. For asynchronous code, it is better to return Arc<MyClass>, and for synchronous code — Rc<MyClass>. This will mimic the semantics of object references in Java.

```rust
use std::sync::Arc;

struct MyClass {
  pub field: i32
}

impl MyClass {

  fn new() -> Arc<MyClass> {
    let obj = MyClass { field: 42 };
    Arc::new(obj)
  }

}

#[tokio::main]
async fn main() {
  
  let obj1 = MyClass::new();

  let obj2 = obj1.clone();

  // objects obj1 and obj2 point to the same data

  // thanks to Arc<T>

}
```
**Using Result<T, E> instead of exceptions**

In Java, errors are handled with exceptions. There are no exceptions in Rust. Instead, the Result<T, E> type is used, which is a pair: the value T and the error E.

Here is an example of a function in Java that can return a value or an error:

```java
public String readFile(String filename) throws IOException {
    File file = new File(filename);
    if (!file.exists()) {
        throw new IOException("File does not exist");
    }

    FileInputStream inputStream = new FileInputStream(file);
    byte[] bytes = new byte[(int) file.length()];
    inputStream.read(bytes);
    inputStream.close();

    return new String(bytes);
}
```
This function may return an IOException if the file does not exist. To handle this error, we can use try-catch.

In Rust, we can write a function using the Result<T, E> type:

```rust
fn divide(a: i32, b: i32) -> Result<i32, String> {
    if b == 0 {
        Err("Division by zero".to_string())
    } else {
        Ok(a / b)
    }
}
```

**Usage ? instead of unwrap() for error escalation**

In Java, to escalate an error through the code above, we can use the throw statement, or in the function definition, we can simply specify the type of exception that might occur there. Rust uses the ? operator for this.

Instead of the well-loved unwrap(), which can cause a panic in case of an error, we can use the ? operator to simply pass the error up. This avoids panics and gives you more control over error handling.

```rust
fn read_file(filename: &str) -> Result<(), std::io::Error> {
    let mut file = File::open(filename)?;
    let mut bytes = vec![];

    file.read_to_end(&mut bytes)?;

    Ok(())
}
```
In this example, if the file does not exist, the function will return an error of type Error. This error will be propagated to the calling code.

**Use the thiserror library to declare errors**

Java uses Exception to declare errors. There is no such thing in Rust. Instead, errors are declared using types.

Using the thiserror library makes declaring errors in Rust very convenient and easy. With this library, you can use macros to declare bugs that automatically generate code for their release.

```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum MyError {
    #[error("Connection problem error")]
    ConnectionError,
    #[error("Access deny error")]
    PermissionError,
}
```
Using the thiserror library also allows you to automatically convert third-party library errors to your error.

```rust
#[derive(Error, Debug)]
pub enum DataStoreError {
    #[error("data store disconnected")]
    Disconnect(#[from] std::io::Error),
}
```
**Use the log + env_logger libraries instead of println!**

In Rust, I also recommend avoiding using println directly! for logging. Instead, I include the log library and use macros like info!, warn! or debug!.

And then in main, initialize env_logger, which makes it easy to configure the output of logs to stdout or a file.
```rust
fn main() {
  env_logger::init();
  
  log::debug!("Application launched");
  
  // ...
}
```
This approach makes the code much cleaner and allows flexible control over application logging. In Java, I often used this pattern, so I was pleased to find a similar solution in the Rust ecosystem.

**Using Mutex for Asynchronous Code**

In Java, when you want a variable to be readable by multiple threads, you simply pass a reference. In Rust, the analogue of these structures is Arc. Arc allows you to safely pass a reference to a variable between threads, but does not allow you to change the variable.

In Java, when you want a variable to be available across multiple threads for modification, you use AtomicReference or ConcurrentHashMap.

In Rust, if you want a variable to be available across multiple threads and changeable, you need to use a Mutex. A mutex allows you to acquire an exclusive lock on a variable, which allows other threads to wait until the thread that has gained access through the lock releases it.

By the way, it’s better to use futures::lock::Mutex right away, and not std::sync::Mutex, since the second one is designed for non-asynchronous code.

Here is an example of how to use Mutex for asynchronous code:
```rust
use futures::lock::Mutex;

fn main() {
    let mut counter = Mutex::new(0);

    let handle1 = spawn(async {
        let mut lock = counter.lock().await;
        *lock += 1;
    });

    let handle2 = spawn(async {
        let mut lock = counter.lock().await;
        *lock += 1;
    });

    handle1.join().unwrap();
    handle2.join().unwrap();

    println!("{}", counter.lock().unwrap()); // 2
}
```
In this example, we are creating a counter variable of type Mutex. We then start two asynchronous threads that increment the counter by one. At the end of the program, we print the value of the counter, which should be 2.

**Overcoming limitations when using async functions in traits**

Another interesting aspect of moving from Java to Rust is the use of traits. In Rust, you can’t declare asynchronous functions inside traits directly. However, there is a library called “async_trait” that allows you to get around this limitation.

```rust
use async_trait::async_trait;

#[async_trait]
trait Worker {
    async fn do_work(&self);
}

struct MyWorker;

#[async_trait]
impl Worker for MyWorker {
    async fn do_work(&self) {
        println!("Working asynchronously");
    }
}

#[tokio::main]
async fn main() {
    let worker = MyWorker;
    worker.do_work().await;
}
```

Thanks to the “async_trait” library, we can use async functions inside traits.

**What conclusions do we have?**

1. When migrating from Java to Rust, it’s best to start with simple types like String and avoid complex ones like &str. This will allow you to write a working version of the code faster.
2. Using structs and associated functions instead of classes is in keeping with idiomatic Rust.
3. Error handling in Rust uses the Result type instead of exceptions as in Java.
4. Libraries like thiserror, log, and async_trait make it easy to write idiomatic code in Rust.

**What do you want to say in conclusion?**

Switching from Java to Rust can be tricky due to the differences in approach between the two languages. The main thing is to start with simple constructions, then gradually move on to idiomatic code.
