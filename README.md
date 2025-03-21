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
    The program will create a `BufReader` instance by wrapping a reference to the `stream`, which is symbolized with `&stream`. This wraps the TCP stream in a `BufReader` to efficiently read data line by line.


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
        
        Takes lines while they are not empty (`!line.is_empty()`), which marks the end of HTTP headers. This is necessary because an HTTP request ends with a blank line (`""`), which indicate that the request is complete.

    - `.collect()`

        Collects the remaining lines into a `Vec<_>`. `Vec` is a dynamic array type in Rust that can hold multiple values. The `_` allows the compiler to infer the item type (in this case, it's `String`).


- **Print the HTTP Request**

    ```rust
    println!("Request: {http_request:#?}");
    ```
    This part will prints the `http_request` vector to the console using the `println!` macro. This output will display the HTTP request with each line in a separate row.

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
    - **`{status_line}`**: Inserts the HTTP status line (`HTTP/1.1 200 OK`).
    - **`\r\n`**: Carriage return + newline (CRLF), separating headers and body according to the HTTP protocol.
    - **`Content-Length: {length}`**: Indicates the length of the content to be sent.
    - **`\r\n\r\n`**: Two blank lines separating the HTTP headers from the body.
    - **`{contents}`**: Inserts the actual HTML content.

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

<br>

## Commit (3) Reflection

Here is the image:

![Commit 2 screen capture](/assets/commit3.png)

### What is Inside the `handle_connection` Method Now?

After further addition and refactor to the `handle_connection` method, now the method has a split condition on how to handle between difference response. 
```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
}
```
With this code, the program can now **distinguish between valid and invalid requests** by checking the content of the incoming HTTP request. This changes ensure that if the `request_line` matches `"GET / HTTP/1.1"`, it means the client is requesting the home page (`/`), which indicate that the server should responds with `HTTP/1.1 200 OK` status and HTML from `hello.html` as a success GET response.

If the `request_line` does not match `"GET / HTTP/1.1"`, the server assumes the client requested an invalid page (e.g., `/foo`, `/bad`, etc.). In that case, the server will responds with `HTTP/1.1 404 NOT FOUND` status and HTML from `404.html` as a `404 Not Found` error.

### Is There Something You Can Improve?

After implementing the splitting method, I realized that the `if` and `else` blocks have a lot of repetition. They’re both reading files and writing the contents of the files to the stream. The only differences are the status line and the filename, which lead to code duplicate. To refactor this, I've simplified the branching into this:

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```
Now the `if` and `else` blocks only return the appropriate values for the status line and filename. After receiving the right status and HTML content related to the situation, the program will assign these two values to status_line and filename using a pattern in the `let` statement and use them to return the correct response to the request. Refactoring this branch also makes it easier to see the difference between the two cases, while also leaving only one place to update the code if we want to change how the file reading and response writing work later.

Before the request conditions splitted into different cases, I also removed this part of the `handle_connection` method.
```rust
let http_request: Vec<_> = buf_reader
    .lines()
    .map(|result| result.unwrap())
    .take_while(|line| !line.is_empty())
    .collect();
```
The removal of this part of the code based on its relevance to the method's purposes. Since we don't need to show the HTTP request in our terminal (where we returns an HTML response instead now), Keeping this part will be unnecessary. This part also shows a warning for `use of moved value`, which make the refactor even more essential.

<br>

## Commit (4) Reflection

In this version of `handle_connection()`, some lines were added and modified such as shown below:

```rust
let (status_line, filename) = match &request_line[..] {
    "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
    "GET /sleep HTTP/1.1" => {
        thread::sleep(Duration::from_secs(5));
        ("HTTP/1.1 200 OK", "hello.html")
    }
    _ => ("HTTP/1.1 404 NOT FOUND", "404.html"),
};
```

There is another endpoint added to the request handling, which is the `/sleep` endpoint. This endpoint will make the current thread "sleep" for 10 seconds. This simulates a thread when that thread is busy processing a request. If we opened two browser windows, where we access the endpoint of `/sleep` first and then trying to load the `/` endpoint, we'll see that `/` waits until `/sleep` has slept for its full 5 seconds before loading.

This simulation shows that if another user tries to access our web application during this time, they will be forced to wait because our server only rely on a single thread. When this thread is busy, all other requests cannot be processed and must wait until the current request is completed. To resolve this issue, there are multiple techniques we could use as a preventing action; the one that I’ll implement is a **thread pool**, which will be explained in the next commit.

<br>

## Commit (5) Reflection

To use a thread pool for handling requests with multiple threads, I've implemented its functionality in `lib.rs`.

```rust
pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

type Job = Box<dyn FnOnce() + Send + 'static>;

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();
        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));   // spawn new workers
        }

        ThreadPool { workers, sender }
    }
    // --snip--
}
```
A new thread pool is created using the `new()` method, which initializes a set of `Workers` based on the given `size` parameter and push them in a vector. The method also create a new `sender` and `receiver` to facilitate communication between the threads.

The `Worker` instances perform the tasks given by the thread pool through asynchronous channel communication that previously initialized in the `new()` method. At runtime, each worker will continuously loops while waiting for the thread pool to send them a `Job`, executing it upon receiving it, and then returning to the loop to wait for a new job.
```rust
struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || loop {
            let job = receiver.lock().unwrap().recv().unwrap();

            println!("Worker {id} got a job; executing.");

            job();
        });

        Worker { id, thread }
    }
}
```

To send a `Job` to the workers, the program must call the `execute()` method, which will dispatches the job to one of the workers in the vector. There's also a **mutex lock** ensures that only one worker can access the job at a time, allowing it to accept it without worries.
```rust
// src/lib.rs
pub fn execute<F>(&self, f: F)
where
    F: FnOnce() + Send + 'static,
{
    let job = Box::new(f);

    self.sender.send(job).unwrap();
}


// src/main.rs
fn main() {
    // --snip--
    for stream in listener.incoming() {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }
}
```


<br>

## Commit (Bonus) Reflection

Instead of using the `assert!` macro, I created a new method `build()` that works similarly to the `new()` method but with a slight modification at the start of the process.
```rust
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
    
    // Validate the size; it must be greater than 0
    if size <= 0 {
        return Err("The size of the thread pool must be greater than 0");
    }

    let (sender, receiver) = mpsc::channel();
    let receiver = Arc::new(Mutex::new(receiver));

    let mut workers = Vec::with_capacity(size);

    for id in 0..size {
        workers.push(Worker::new(id, Arc::clone(&receiver)));
    }

    Ok(ThreadPool { workers, sender })
}
```

The `build()` method provides a better approach by ensuring that an invalid size (zero or negative) is caught early, preventing the creation of an empty or negative-sized thread pool. The return type of this method is `Result`, which also make any error that occurs during the execution of `build()` will cause the `main` function to unwrap the `Result`, leading to a system panic if an error is encountered.