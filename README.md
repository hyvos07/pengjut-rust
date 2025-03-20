# 🦀 Advanced Programming - Concurrency

**Nama**  : Daniel Liman <br>
**NPM**   : 2306220753 <br>
**Kelas** : Pemrograman Lanjut A


## Commit (1) Reflection

### What is Inside the `handle_connection` Method?

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    println!("Request: {http_request:#?}");
}
```

There are **three main parts** inside the `handle_connection` method.

**1. Creating a buffered reader**
```rust
let buf_reader = BufReader::new(&stream);
```
The program will create a `BufReader` instance by wrapping a reference to the `stream`, which is symbolized with `&stream`. This wraps the TCP stream in a `BufReader` to efficiently read data line by line. `BufReader` is a wrapper that provides efficient reading capabilities by buffering data in memory, reducing the number of low-level system calls.


**2. Read HTTP Request Lines**
```rust
let http_request: Vec<_> = buf_reader
    .lines()
    .map(|result| result.unwrap())
    .take_while(|line| !line.is_empty())
    .collect();
```
This part of code will:

- `buf_reader.lines()`

    Returns an iterator over the lines of input from the `buf_reader`. Each line is wrapped in a `Result<String, std::io::Error>`, because reading from a stream could lead to a failure.

- `.map(|result| result.unwrap())`
   
    Maps (transform) each `Result` to a `String` by calling `.unwrap()`. `unwrap()` will extract the value if `Ok`, but panic if there is an `Err`. This means if an error occurs (e.g., invalid UTF-8 data or a stream reading error), the program will crash.

- `.take_while(|line| !line.is_empty())`
    
    Takes lines while they are not empty (`!line.is_empty()`), which marks the end of HTTP headers. This is necessary because an HTTP request ends with a blank line (`""`), which signals the request is complete.

- `.collect()`

    Collects the remaining lines into a `Vec<_>`. `Vec` is a dynamic array type in Rust that can hold multiple values. The `_` allows the compiler to infer the item type (in this case, it's `String`).


**3. Print the HTTP Request**
```rust
println!("Request: {http_request:#?}");
```
This part will prints the `http_request` vector to the console using the `println!` macro. This output will display the HTTP request with each line in a separate row, making it easier to read.