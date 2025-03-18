# Redirect:

- [Milestone 1](#milestone-1-single-threaded-web-server)

# Milestone 1: Single threaded web server

This is a simple single-threaded web server in Rust. The web server consists of two main functions:

1. `main()`: Sets up a TCP listener and handles incoming connections
2. `handle_connection()`: Processes each individual TCP stream (connection)

Next is the TCP Listener setup:

```rust
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
```

This code creates a TCP listener bound to localhost (127.0.0.1) on port 7878, and `unwrap()` method extracts the success value or panics if there's an error (e.g., if the port is already in use).

Then, I created the connection handling loop:

```rust
    for stream in listener.incoming() { 
        let stream = stream.unwrap();
        
        handle_connection(stream);
    }
```

`listener.incoming()` returns an iterator over connection attempts, and for each successful connection (`stream.unwrap()`), we call `handle_connection()`. This creates a simple loop that processes one connection at a time (single-threaded).

Next for the request processing inside `handle_connection()` function. 

```rust
    fn handle_connection(mut stream: TcpStream) {
        let buf_reader = BufReader::new(&mut stream);
        let http_request: Vec<_> = buf_reader
            .lines()
            .map(|result| result.unwrap())
            .take_while(|line| !line.is_empty())
            .collect();

        println!("Request: {:#?}", http_request);
    }
```

This function does the following:

1. `BufReader::new(&mut stream)` creates a buffered reader from the TCP stream, which is more efficient for reading line by line

2. The lines of the HTTP request are then processed through a chain of iterator operations:

- `.lines()` returns an iterator over the lines in the buffered reader
- `.map(|result| result.unwrap())` extracts the string from each Result (or panics if there's an error)
- `.take_while(|line| !line.is_empty())` continues taking lines until it encounters an empty line, which marks the end of the HTTP headers
- `.collect()` gathers all these lines into a vector

3. Finally, `println!("Request: {:#?}", http_request)` prints the HTTP request headers with pretty formatting (`{:#?}`)

At this stage, the server simply receives and prints HTTP requests without sending any responses, processes connections sequentially (single-threaded), has minimal error handling (uses `unwrap()` which will crash the program on errors), and doesn't parse or route requests based on paths or methods.