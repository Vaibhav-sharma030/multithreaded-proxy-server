# Multithreaded HTTP Proxy Server

A multithreaded HTTP proxy server built in **C** using POSIX sockets, pthreads, semaphores, mutexes, DNS resolution, and an in-memory LRU cache.

The proxy acts as an intermediary between a client and a remote web server. It receives HTTP requests from clients, forwards them to the destination server, returns the server response to the client, and caches responses to reduce repeated network requests.

---

## Features

- HTTP GET request handling
- TCP socket-based communication
- Multithreaded client handling using POSIX threads
- Semaphore-based concurrency control
- Thread-safe shared cache using mutex synchronization
- DNS hostname resolution
- In-memory response caching
- LRU (Least Recently Used) cache eviction
- Configurable proxy port
- Chunk-based response forwarding
- HTTP request parsing
- Basic HTTP error handling
- Support for HTTP/1.0 and HTTP/1.1 requests

---

## Architecture

```text
                         ┌─────────────────────┐
                         │       Client        │
                         │   Browser / curl     │
                         └──────────┬──────────┘
                                    │
                                    │ HTTP Request
                                    ▼
                         ┌─────────────────────┐
                         │    Proxy Server     │
                         │                     │
                         │  ┌───────────────┐  │
                         │  │ Request Parser│  │
                         │  └───────┬───────┘  │
                         │          │          │
                         │  ┌───────▼───────┐  │
                         │  │  Cache Lookup  │  │
                         │  └───────┬───────┘  │
                         │          │          │
                         │     ┌────┴────┐     │
                         │     │         │     │
                         │    HIT       MISS   │
                         │     │         │     │
                         └─────┼─────────┼─────┘
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
                         ┌─────────────────────┐
                         │       Client        │
                         └─────────────────────┘



## Tech Stack

- **Language:** C
- **Networking:** TCP Sockets
- **Concurrency:** POSIX Threads (`pthread`)
- **Synchronization:** Mutexes & Semaphores
- **Environment:** Linux / WSL
- **Build:** Makefile

## Project Structure

```text
proxy-web-server/
├── proxy_server_with_cache.c   # Main proxy server
├── proxy_parse.c               # HTTP request parsing
├── proxy_parse.h               # Parser definitions
├── Makefile                    # Build configuration
└── README.md


## How It Works
Client
  |
  | HTTP Request
  v
Proxy Server
  |
  |-- Check Cache
  |      |
  |      └── Cache Hit → Return cached response
  |
  |-- Cache Miss
  |      |
  |      v
  |   Remote Server
  |      |
  |      v
  |   Response
  |      |
  |      ├── Send to Client
  |      └── Store in Cache
  |
  v
Client


## Concurrency
              Proxy Server
                   │
        ┌──────────┼──────────┐
        ▼          ▼          ▼
     Thread 1   Thread 2   Thread 3
        │          │          │
     Client 1   Client 2   Client 3


## Caching
Request
   │
   ▼
Cache Lookup
   │
   ├──────────── HIT ────────────► Cached Response
   │
   └──────────── MISS
                    │
                    ▼
              Remote Server
                    │
                    ▼
                Response
                 /     \
                /       \
               ▼         ▼
            Client     Cache
