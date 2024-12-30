# Hello Server

A simple multi-threaded web server implemented in Rust using a custom thread pool. This project demonstrates how to handle multiple incoming HTTP requests concurrently by leveraging Rust's concurrency primitives.

## Table of Contents

- [Features](#features)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
- [Project Structure](#project-structure)
- [Concurrency Details](#concurrency-details)

## Features

- **Thread Pool Implementation**: Efficiently manages a pool of worker threads to handle incoming HTTP requests.
- **HTTP Request Handling**: Supports basic routing for:
    - `GET /` - Serves `hello.html` with a 200 OK response.
    - `GET /sleep` - Simulates a long-running request by sleeping for 5 seconds before responding.
    - All other routes respond with a 404 Not Found.
- **Graceful Shutdown**: Ensures all worker threads are properly shut down when the server stops.

## Prerequisites

- **Rust**: Ensure you have Rust installed. You can install Rust using [rustup](https://rustup.rs/).

## Installation

1. **Clone the Repository**

   ```bash
   git clone https://github.com/yourusername/hello-server.git
   cd hello-server
   ```

2. **Build the Project**

   ```bash
   cargo build --release
   ```

## Usage

1. **Run the Server**

   ```bash
   cargo run
   ```

   The server listens on `127.0.0.1:7878` and initializes a thread pool with 18 worker threads.

2. **Access the Server**

    - Open your browser and navigate to `http://127.0.0.1:7878/` to view the `hello.html` page.
    - Navigate to `http://127.0.0.1:7878/sleep` to simulate a slow request.

3. **Shutdown**

   The server is configured to handle only two incoming connections (`listener.incoming().take(2)`). After processing two requests, it will print `Shutting down.` and terminate gracefully.

## Project Structure

```
hello-server/
├── src/
│   ├── main.rs
│   └── lib.rs
├── hello.html
├── 404.html
├── Cargo.toml
└── README.md
```

- **src/main.rs**: Contains the main function that sets up the TCP listener and thread pool.
- **src/lib.rs**: Implements the `ThreadPool` and `Worker` structs.
- **hello.html**: The HTML file served for the root route.
- **404.html**: The HTML file served for undefined routes.
- **Cargo.toml**: Rust's package configuration file.

## Concurrency Details

The server utilizes a custom thread pool to handle multiple requests concurrently. Here's a brief overview of how it works:

1. **ThreadPool Struct**: Manages a vector of `Worker` threads and a sending channel to dispatch jobs.
2. **Worker Struct**: Each worker thread listens for incoming jobs and executes them.
3. **Job Dispatching**: When a new connection is received, a job is created and sent to the thread pool's sender channel. Workers pick up these jobs and handle the requests.
4. **Graceful Shutdown**: Implemented via the `Drop` trait for `ThreadPool`, ensuring all threads are properly joined before the server exits.

### Why Only One Worker Handles Jobs Initially

If you observe that only `Worker 0` is handling all jobs, it's likely because requests are being handled sequentially or complete too quickly for other workers to be utilized. To see multiple workers in action:

- **Simulate Concurrent Requests**: Use tools like `ab` (ApacheBench) or `wrk` to send multiple simultaneous requests.

  ```bash
  ab -n 100 -c 18 http://127.0.0.1:7878/
  ```

- **Long-Running Tasks**: Access the `/sleep` route to keep a worker busy and allow other workers to handle additional requests.
