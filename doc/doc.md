# toriidb - Documentation

> Back to [README](../README.md)

## Prerequisites

- Go 1.25 or higher
- A local environment that can run `go build`, `go test`, and `go run`
- `OPENAI_API_KEY` if you want to use vector embedding and semantic search features

## Installation

### From Source

```bash
git clone https://github.com/agenvoy/toriidb.git
cd toriidb
go build ./...
```

### Run the REPL

```bash
go run cmd/test/main.go
```

### Run Unit Tests

```bash
go test ./... -count=1
```

## Configuration

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OPENAI_API_KEY` | No | Enables embedding generation for `SET ... VECTOR`, `VSEARCH`, `VSIM`, and related vector features |
| `TORIIDB_EMBED_DIM` | No | Overrides the embedding dimension; defaults to `256` |

### Config File

If you use environment-based setup, place the variables in a local `.env` file before running the REPL:

```bash
cat <<'EOF' > .env
OPENAI_API_KEY=your_api_key
TORIIDB_EMBED_DIM=256
EOF
```

## Usage

### Basic

Start the interactive shell:

```bash
go run cmd/test/main.go
```

Query and mutate primitive values:

```bash
SET counter 1
INCR counter
GET counter
TTL counter
```

Work with JSON fields through dot notation:

```bash
SET user '{"name":"Torii","profile":{"level":1}}'
GET user.name
SET user.profile.level 2
INCR user.profile.level
GET user.profile.level
```

### Advanced

Use expiration and database selection:

```bash
SET session.token abc123 300
TTL session.token
SELECT 1
SET cache.key warm
KEYS *
```

Use vector-enabled commands after configuring `OPENAI_API_KEY`:

```bash
SET article:1 'Redis-style embedded KV with vector search' VECTOR
SET article:2 'Document store with JSON field mutation' VECTOR
VSEARCH vector search LIMIT 2
VSIM article:1 article:2
VGET article:1
```

## CLI Reference

### Commands

| Command | Syntax | Description |
|---------|--------|-------------|
| `GET` | `GET <key>` | Return the raw value for a key or a nested field via dot notation |
| `SET` | `SET <key> <value> [NX\|XX] [seconds] [VECTOR]` | Write a value, optionally with conditional flags, TTL, or vector embedding |
| `EXIST` | `EXIST <key>` | Check whether a key or nested field exists |
| `TYPE` | `TYPE <key>` | Return the stored value type |
| `DEL` | `DEL <key> [key2] ...` | Delete keys or a nested field when dot notation is used |
| `INCR` | `INCR <key> [delta]` | Increment a numeric key or nested numeric field |
| `TTL` | `TTL <key>` | Return remaining TTL |
| `EXPIRE` | `EXPIRE <key> <seconds>` | Set TTL in seconds |
| `EXPIREAT` | `EXPIREAT <key> <timestamp>` | Set expiry as a Unix timestamp |
| `PERSIST` | `PERSIST <key>` | Remove expiry from a key |
| `KEYS` | `KEYS <pattern>` | List keys matching a glob pattern |
| `FIND` | `FIND <op> <value> [LIMIT <n>]` | Scan keys by comparison operator |
| `QUERY` | `QUERY <expression> [LIMIT <n>]` | Filter JSON documents with infix expressions |
| `VSEARCH` | `VSEARCH <text> [MATCH <pattern>] [LIMIT <n>]` | Return the most similar vectorized keys |
| `VSIM` | `VSIM <key1> <key2>` | Compute cosine similarity between two stored vectors |
| `VGET` | `VGET <key>` | Return the stored embedding vector |
| `SELECT` | `SELECT <db>` | Switch active database between `0` and `15` |

### Command Notes

- Dot notation splits `key.field.nested` into a top-level key plus JSON subkeys.
- `SET ... VECTOR` uses the value text as embedding input and stores the vector inline with the entry.
- `QUERY`, `GET field`, and JSON mutations rely on a parsed JSON cache to avoid repeated decoding.
- Internal vector cache keys under `__torii:*` are hidden from scan-style commands.

### Public Go API

```go
func New(path ...string) (*Store, error)
func (s *Store) Session() *Session
func (s *Store) Close() error
func (c *core) Exec(input string) string
func (c *core) Set(key, value string, flag SetFlag, expireAt *int64) error
func (c *core) SetVector(ctx context.Context, key, value string, flag SetFlag, expireAt *int64) error
func (c *core) Get(key string) (*Entry, bool)
func (c *core) VSearch(ctx context.Context, text, pattern string, k int) ([]string, error)
func (c *core) VSim(key1, key2 string) (float64, error)
```

- `New` creates the store and initializes up to 16 lazy-loaded databases.
- `Session` returns a session-scoped handle that shares storage and embedding settings.
- `Exec` routes REPL commands through a single parser and dispatcher.
- `SetVector`, `VSearch`, and `VSim` expose vector-search workflows on top of the core key-value engine.

***

©️ 2026 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
