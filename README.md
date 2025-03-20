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

- **Creating a buffered reader**

    ```rust
    let buf_reader = BufReader::new(&stream);
    ```
    The program will create a `BufReader` instance by wrapping a reference to the `stream`, which is symbolized with `&stream`. This wraps the TCP stream in a `BufReader` to efficiently read data line by line. `BufReader` is a wrapper that provides efficient reading capabilities by buffering data in memory, reducing the number of low-level system calls.


- **Read HTTP Request Lines**

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
    
        Maps (transform) each `Result` to a `String` by calling `.unwrap()`. `unwrap()` will extract the value if `Ok`, but `panic` if there is an `Err`. This means if an error occurs (e.g., invalid UTF-8 data or a stream reading error), the program will crash.

    - `.take_while(|line| !line.is_empty())`
        
        Takes lines while they are not empty (`!line.is_empty()`), which marks the end of HTTP headers. This is necessary because an HTTP request ends with a blank line (`""`), which signals the request is complete.

    - `.collect()`

        Collects the remaining lines into a `Vec<_>`. `Vec` is a dynamic array type in Rust that can hold multiple values. The `_` allows the compiler to infer the item type (in this case, it's `String`).


- **Print the HTTP Request**

    ```rust
    println!("Request: {http_request:#?}");
    ```
    This part will prints the `http_request` vector to the console using the `println!` macro. This output will display the HTTP request with each line in a separate row, making it easier to read.

<br>

## Commit (2) Reflection

Here is the image:

![Commit 2 screen capture](/assets/commit2.png)

### What is Inside the `handle_connection` Method Now?

Now that we got some new lines inside `handle_connection` method, there are some new functionalities that this method provides, which is:

- **Define HTTP Response Status Line**

    ```rust
    let status_line = "HTTP/1.1 200 OK";
    ```
    This part defines the `status_line` variable containing a standard HTTP 200 OK response. This code will be provided as the successful request status for our response later.

- **Read HTML File Content**

    ```rust
    let contents = fs::read_to_string("hello.html").unwrap();
    ```
    This part will reads the contents of `hello.html` from the project directory using the `fs::read_to_string` function. After that, the contents of the HTML file will be stored as a `String` with the **`.unwrap()`** method. The **`.unwrap()`** method will results in `panic` if the file cannot be read (e.g., if the file is missing).

- **Calculate Content Length**

    ```rust
    let length = contents.len();
    ```
    This line calculates the length (**in bytes**) of the `contents` string. The length is required in the HTTP response header (`Content-Length`) to inform the browser how much data to expect.

- **Format the HTTP Response**

    ```rust
    let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
    ```
    Inside of the `format!` macro, there is:
    1. **`{status_line}`**: Inserts the HTTP status line (`HTTP/1.1 200 OK`).
    2. **`\r\n`**: Carriage return + newline (CRLF), separating headers and body according to the HTTP protocol.
    3. **`Content-Length: {length}`**: Indicates the length of the content to be sent.
    4. **`\r\n\r\n`**: Two blank lines separating the HTTP headers from the body.
    5. **`{contents}`**: Inserts the actual HTML content.

    Example of what the **response** will be:
    ```
    HTTP/1.1 200 OK
    Content-Length: 75

    // The html content here
    ```

- **Write Response to Stream**

    ```rust
    stream.write_all(response.as_bytes()).unwrap();
    ```
    In this line, **`response.as_bytes()`** converts the formatted HTTP response from a `String` to a byte slice (`&[u8]`), which is required for network transmission. After that, **`.write_all()`** will writes all the bytes to the client via the `TcpStream` and **`.unwrap()`** ensures any write errors (e.g., if the connection is lost) will results in `panic` to the program.