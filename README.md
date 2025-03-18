# Redirect:

- [Milestone 1](#milestone-1-single-threaded-web-server)
- [Milestone 2](#milestone-2-returning-html)

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

# Milestone 2: Returning HTML

![Commit 2 screen capture](/assets/images/commit2.png)

At this milestone, the updated `handle_connection()` function now creates an HTTP response with a status line, headers, and HTML content. Then the server constructs a proper HTTP response to the client with:

1. Status line `(HTTP/1.1 200 OK)`
2. `Content-Length` header to indicate the size of the response body
3. Two CRLF sequences `(\r\n\r\n)` to separate headers from body
4. HTML content from a file

The server now reads HTML content from a file named "hello.html" using `fs::read_to_string()` and then the `stream.write_all()` method sends the complete response back to the client.

At this stage, the server always returns a 200 OK response regardless of the request, only serves a single HTML file ("hello.html"), and still processes connections sequentially (single-threaded).

# Milestone 3: Validating request and selectively responding

![Commit 3 screen capture](/assets/images/commit3.png)

In this milestone, I've enhanced the web server to validate incoming requests and respond selectively with appropriate content. The server now also returns proper 404 error pages for invalid routes.

The updated `handle_connection()` function now extracts only the first line of the request (the request line), then uses pattern matching to determine the appropriate response. It makes the server serves different HTML files based on the request path.

For improvements, I've done several things:

1. Instead of collecting all HTTP headers, the code now extract only the request line using `buf_reader.lines().next()` which contains the HTTP method, path, and version.
2. Using Rust's match expression to determine the response based on the request path:
- For requests to the root path (`GET / HTTP/1.1`), we return a 200 OK with "hello.html"
- For all other requests, we return a 404 NOT FOUND with "404.html"
3. Different HTML files are served based on the request, providing appropriate user feedback


About refactoring, the `match` expression used in this update is a significant refactoring from the traditional if-else approach. Here's why this refactoring is beneficial:

1. Without the refactoring, we would need to repeat similar code blocks for each condition:

```rust
    if request_line == "GET / HTTP/1.1" {
        status_line = "HTTP/1.1 200 OK";
        filename = "hello.html";
    } else {
        status_line = "HTTP/1.1 404 NOT FOUND";
        filename = "404.html";
    }
```

2. The match expression ensures all variables are properly initialized for all possible cases, which the Rust compiler can verify

3. Using a tuple to return multiple values from the match expression is more elegant and reduces the chance of inconsistency between related variables

4. Adding new routes and responses becomes cleaner and requires less code as we only need to add new match arms without duplicating the response generation logic

5. This approach allows us to use immutable variables (`let` without `mut`), which is preferred in Rust for safer concurrency and clearer code intention

At this stage, the server now listens for connections on localhost:7878, parses the HTTP request line, responds with "hello.html" and 200 OK for requests to "/", responds with "404.html" and 404 NOT FOUND for all other paths, but still maintains a single-threaded processing model.






