# Go Worker Pool with Graceful Shutdown & Observability

## Project Overview

This project is a demonstration of a robust worker pool pattern implemented in Go. It showcases best practices for concurrent programming, including:

*   **Graceful Shutdown:** Handling OS interrupt signals (like Ctrl+C) to allow workers to finish their current tasks before exiting.
*   **Concurrency Management:** Using a semaphore to limit the number of concurrently running goroutines (workers).
*   **Context Propagation:** Utilizing `context.Context` for cancellation and managing goroutine lifecycles.
*   **Error Handling:** Proper error propagation from goroutines.
*   **Observability:** Basic metrics collection using `sync/atomic` and exposure of `pprof` endpoints for performance profiling.

The main goal of this project is to illustrate how to build resilient and manageable concurrent applications in Go.

## Problem Solved

In many applications, there's a need to process a large number of tasks concurrently without overwhelming system resources or losing data during an unexpected shutdown. This project addresses:

1.  **Efficient Task Processing:** Distributing tasks among a pool of workers.
2.  **Resource Control:** Limiting the maximum number of active workers.
3.  **Data Integrity:** Ensuring that tasks are not abruptly terminated and data is handled.
4.  **Application Stability:** Allowing the application to shut down, cleaning up resources.
5.  **Performance Analysis:** Providing tools to profile and understand the application's behavior under load.

## How It Works

1.  The application launches a specified number of worker goroutines.
2.  A main goroutine generates tasks (e.g., numbers to be processed) and sends them into a task channel.
3.  Workers pick up tasks from the channel, process them (e.g., square a number), and send results to a result channel.
4.  The main goroutine collects results.
5.  A `sync.WaitGroup` is used to ensure all workers complete their tasks.
6.  OS interrupt signals (`SIGINT`, `SIGTERM`) are caught to trigger a graceful shutdown:
    *   A `context.WithCancel` is used. The `cancel()` function is called сигнал.
    *   Workers use `select` to listen to both the task channel and `ctx.Done()`.
    *   Upon context cancellation, workers stop accepting new tasks, finish current ones, and exit.
7.  `net/http/pprof` endpoints are exposed on a separate HTTP server for live profiling.
8.  Atomic counters track the number of processed tasks.

## Tech Stack

*   Go (Golang)
*   Standard Go libraries: `context`, `sync`, `os/signal`, `net/http/pprof`, `sync/atomic`, `fmt`, `log`, `time`, `math/rand` (for task generation).

## Getting Started

### Prerequisites

*   Go (version 1.18+ recommended for generics)

### Running the Application

1.  Clone the repository:
    ```bash
    git clone https://github.com/malformed-c/go-worker-pool.git
    cd go-worker-pool
    ```
2.  Run the application:
    ```bash
    go run main.go
    ```
    *(You can adjust the number of workers and tasks in `main.go` if needed)*

### Profiling

While the application is running, `pprof` data is available at `http://localhost:6060/debug/pprof/`.
You can use `go tool pprof` to analyze the data, for example:
```bash
# CPU profile for 30 seconds
go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30

# Heap profile
go tool pprof http://localhost:6060/debug/pprof/heap
```
