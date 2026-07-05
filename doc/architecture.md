# toriidb - Architecture

> Back to [README](../README.md)

## Overview

```mermaid
graph TB
    A[REPL Client] --> B[Store Session]
    B --> C[Exec Router]
    C --> D[KV Operations]
    C --> E[JSON Field Operations]
    C --> F[Vector Operations]
    D --> G[DB 0-15]
    E --> G
    F --> G
    G --> H[AOF Replay and Append]
    G --> I[Per-Key JSON Files]
    F --> J[OpenAI Embedder]
```

## Module: REPL Entry

The command-line test client reads user input, prints prompts, and forwards each command to the store session.

```mermaid
graph TB
    subgraph REPL Entry
        A[stdin Reader] --> B[Trim and Validate]
        B --> C[Exit Handling]
        B --> D[Store.Exec]
        D --> E[stdout Printer]
    end
    F[OS Signals] --> C
```

## Module: Command Router

The router parses command tokens and dispatches them to key-value, JSON, TTL, query, and vector handlers.

```mermaid
graph TB
    subgraph Command Router
        A[Input String] --> B[strings.Fields]
        B --> C[Command Switch]
        C --> D[GET SET DEL INCR]
        C --> E[TTL EXPIRE PERSIST SELECT]
        C --> F[FIND QUERY KEYS]
        C --> G[VSEARCH VSIM VGET]
        C --> H[Dot-Notation Split]
    end
```

## Module: Storage Core

The storage core maintains sixteen databases, each with its own lock, in-memory map, AOF file, and lazy-loading lifecycle.

```mermaid
graph TB
    subgraph Storage Core
        A[Store] --> B[allDBs 0-15]
        B --> C[db struct]
        C --> D["data map[string]*Entry"]
        C --> E[sync.RWMutex]
        C --> F[record.aof]
        C --> G[lazy load once]
    end
```

## Module: JSON Document Engine

Document helpers keep raw string values and parsed JSON cache synchronized while supporting nested field mutation.

```mermaid
graph TB
    subgraph JSON Document Engine
        A[Entry.value] --> B[setValue]
        A --> C[setParsed]
        C --> D[parsed cache]
        D --> E[GetField]
        D --> F[Query]
        C --> G[SetField IncrField DelField]
        H[WalkKeys] --> E
    end
```

## Module: Vector Search Pipeline

Vector features embed text, cache query vectors internally, and score candidate entries with cosine similarity.

```mermaid
graph TB
    subgraph Vector Search Pipeline
        A[SET ... VECTOR or VSEARCH] --> B[resolveQueryVector]
        B --> C[Internal Vector Cache]
        B --> D[OpenAI Client]
        D --> E[Embedding]
        E --> F[Entry.Vector]
        F --> G[scanTopK]
        G --> H[Cosine Ranking]
        H --> I[Top-K Result Keys]
    end
```

## Data Flow

```mermaid
sequenceDiagram
    participant User
    participant REPL
    participant Router as Exec Router
    participant DB as Active DB
    participant Disk as AOF/JSON Cache
    User->>REPL: Enter command
    REPL->>Router: Exec(input)
    Router->>DB: Call handler
    DB->>Disk: Append AOF / update file
    DB-->>Router: Result
    Router-->>REPL: Render output
    REPL-->>User: Print response
```

## State Machine

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Parsing: Receive command
    Parsing --> Executing: Valid command
    Parsing --> Idle: Empty input
    Executing --> Persisting: Write operation
    Executing --> Idle: Read-only result
    Persisting --> Idle: Success
    Persisting --> Error: Failure
    Error --> Idle: Next command
```

***

©️ 2026 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
