# Multithreaded HTTP Proxy Server

A multithreaded HTTP proxy server built in **C** using POSIX sockets, pthreads, semaphores, and mutex-based synchronization. The proxy forwards HTTP GET requests to remote web servers, returns responses to clients, and maintains an in-memory cache with **LRU (Least Recently Used) eviction**.

## Features

- HTTP GET request handling
- TCP socket-based client-server communication
- DNS resolution using `gethostbyname()`
- Multithreaded client handling using POSIX threads
- Semaphore-based concurrency control
- Thread-safe shared cache using mutex synchronization
- In-memory response caching
- LRU-based cache eviction
- Configurable proxy port
- HTTP response forwarding
- Error handling for invalid requests and connection failures

## Architecture

                    ┌──────────────────────┐
                    │      Web Client      │
                    │   (Browser / curl)   │
                    └──────────┬───────────┘
                               │
                               │ HTTP Request
                               ▼
                    ┌──────────────────────┐
                    │    Proxy Server      │
                    │                      │
                    │  ┌────────────────┐  │
                    │  │ Request Parser │  │
                    │  └───────┬────────┘  │
                    │          │           │
                    │  ┌───────▼────────┐  │
                    │  │  Cache Lookup  │  │
                    │  └───────┬────────┘  │
                    │          │           │
                    │     ┌────┴────┐      │
                    │     │         │      │
                    │    HIT       MISS    │
                    │     │         │      │
                    └─────┼─────────┼──────┘
                          │         │
                          │         ▼
                          │  ┌───────────────┐
                          │  │ Remote Server │
                          │  └───────┬───────┘
                          │          │
                          │    HTTP Response
                          │          │
                          └────┬─────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │       Client         │
                    └──────────────────────┘

How It Works
1. Client Connection

The proxy creates a TCP listening socket and waits for incoming client connections.

Each accepted client connection is handled by a separate POSIX thread.

Client → Proxy Listening Socket → Client Socket → Worker Thread
2. Request Parsing

The worker thread receives the HTTP request and parses:

HTTP method
Host
Path
HTTP version
Headers
Port number

Currently, the proxy supports HTTP GET requests.

3. Cache Lookup

Before contacting the remote server, the proxy checks whether the requested URL already exists in its cache.

Request
   │
   ▼
Cache Lookup
   │
   ├── HIT  ──→ Return cached response
   │
   └── MISS ─→ Contact remote server

A cache hit avoids another network request and allows the response to be returned directly from memory.

4. Remote Server Connection

For a cache miss, the proxy:

Resolves the hostname using DNS.
Creates a TCP socket.
Connects to the remote web server.
Constructs and sends an HTTP GET request.
Receives the response.
5. Response Forwarding

The response is forwarded to the client in chunks instead of assuming that the complete response fits inside a single buffer.

The response is also stored in the cache for future requests.

6. LRU Cache

The proxy maintains an in-memory linked-list cache.

Each cache entry stores:

Response data
Response size
Requested URL
Last access timestamp
Pointer to the next cache entry

When the cache reaches its maximum capacity, the entry with the oldest access time is removed.

Most Recently Used
       ↓
┌──────┐    ┌──────┐    ┌──────┐
│ URL1 │ →  │ URL2 │ →  │ URL3 │ → NULL
└──────┘    └──────┘    └──────┘
                         ↑
                    Least Recently Used
                         │
                         └── Evicted first
7. Thread Synchronization

Since multiple worker threads access the shared cache simultaneously, a POSIX mutex is used to protect cache operations.

A semaphore limits the number of concurrently active client threads.

                    ┌───────────────┐
                    │   Semaphore   │
                    │  MAX_CLIENTS  │
                    └───────┬───────┘
                            │
              ┌─────────────┼─────────────┐
              ▼             ▼             ▼
           Thread 1      Thread 2      Thread N
              │             │             │
              └─────────────┼─────────────┘
                            │
                     Shared Cache
                            │
                       ┌────▼────┐
                       │  Mutex  │
                       └─────────┘



Technologies Used :-
C
POSIX TCP Sockets
pthreads
POSIX Semaphores
POSIX Mutexes
DNS / gethostbyname()
HTTP/1.0 and HTTP/1.1 request handling
Linux / WSL


Project Structure
|
├── Makefile
├── proxy_server_with_cache.c
├── proxy_parse.c
├── proxy_parse.h
├── .gitignore
└── README.md
