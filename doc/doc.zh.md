# toriidb - 技術文件

> 返回 [README](./README.zh.md)

## 前置需求

- Go 1.25 以上版本
- 可執行 `go build`、`go test` 與 `go run` 的本機環境
- 若要使用向量嵌入與語意搜尋功能，需提供 `OPENAI_API_KEY`

## 安裝

### 從原始碼建置

```bash
git clone https://github.com/agenvoy/toriidb.git
cd toriidb
go build ./...
```

### 啟動 REPL

```bash
go run cmd/test/main.go
```

### 執行單元測試

```bash
go test ./... -count=1
```

## 設定

### 環境變數

| 變數 | 必填 | 說明 |
|----------|----------|-------------|
| `OPENAI_API_KEY` | 否 | 啟用 `SET ... VECTOR`、`VSEARCH`、`VSIM` 等向量功能所需的 OpenAI API 金鑰 |
| `TORIIDB_EMBED_DIM` | 否 | 覆寫 embedding 維度，預設為 `256` |

### 設定檔

若採用環境變數方式，可先建立本機 `.env` 檔：

```bash
cat <<'EOF' > .env
OPENAI_API_KEY=your_api_key
TORIIDB_EMBED_DIM=256
EOF
```

## 使用方式

### Basic

啟動互動式命令列：

```bash
go run cmd/test/main.go
```

操作基本型別與數值：

```bash
SET counter 1
INCR counter
GET counter
TTL counter
```

使用點記法處理 JSON 欄位：

```bash
SET user '{"name":"Torii","profile":{"level":1}}'
GET user.name
SET user.profile.level 2
INCR user.profile.level
GET user.profile.level
```

### Advanced

使用過期時間與多資料庫切換：

```bash
SET session.token abc123 300
TTL session.token
SELECT 1
SET cache.key warm
KEYS *
```

在已設定 `OPENAI_API_KEY` 後使用向量命令：

```bash
SET article:1 'Redis-style embedded KV with vector search' VECTOR
SET article:2 'Document store with JSON field mutation' VECTOR
VSEARCH vector search LIMIT 2
VSIM article:1 article:2
VGET article:1
```

## 命令列參考

### Commands

| Command | Syntax | Description |
|---------|--------|-------------|
| `GET` | `GET <key>` | 讀取鍵值，或透過點記法讀取巢狀欄位 |
| `SET` | `SET <key> <value> [NX\|XX] [seconds] [VECTOR]` | 寫入值，並可搭配條件旗標、TTL 或向量嵌入 |
| `EXIST` | `EXIST <key>` | 檢查鍵值或巢狀欄位是否存在 |
| `TYPE` | `TYPE <key>` | 回傳儲存值的型別 |
| `DEL` | `DEL <key> [key2] ...` | 刪除鍵值，或在點記法下刪除巢狀欄位 |
| `INCR` | `INCR <key> [delta]` | 遞增數值鍵值或巢狀數值欄位 |
| `TTL` | `TTL <key>` | 取得剩餘存活時間 |
| `EXPIRE` | `EXPIRE <key> <seconds>` | 以秒數設定過期時間 |
| `EXPIREAT` | `EXPIREAT <key> <timestamp>` | 以 Unix 時間戳設定過期時間 |
| `PERSIST` | `PERSIST <key>` | 移除鍵值的過期設定 |
| `KEYS` | `KEYS <pattern>` | 以 glob 模式列出符合的鍵值 |
| `FIND` | `FIND <op> <value> [LIMIT <n>]` | 以比較運算子掃描資料 |
| `QUERY` | `QUERY <expression> [LIMIT <n>]` | 以中序表達式過濾 JSON 文件 |
| `VSEARCH` | `VSEARCH <text> [MATCH <pattern>] [LIMIT <n>]` | 找出最相近的向量化鍵值 |
| `VSIM` | `VSIM <key1> <key2>` | 計算兩個已儲存向量的餘弦相似度 |
| `VGET` | `VGET <key>` | 取得已儲存的 embedding 向量 |
| `SELECT` | `SELECT <db>` | 在 `0` 到 `15` 之間切換目前資料庫 |

### Command Notes

- 點記法會把 `key.field.nested` 拆成頂層鍵與 JSON 子鍵路徑。
- `SET ... VECTOR` 會以 value 文字作為 embedding 輸入，並把向量直接存入 entry。
- `QUERY`、欄位讀取與 JSON 增修會共用解析後的 JSON 快取以減少重複解碼。
- `__torii:*` 內部向量快取鍵值不會出現在掃描型命令結果中。

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

- `New` 會建立 store，並初始化最多 16 個延遲載入的資料庫。
- `Session` 會回傳共享儲存狀態與 embedding 設定的工作階段 handle。
- `Exec` 透過單一解析與分派入口處理 REPL 指令。
- `SetVector`、`VSearch` 與 `VSim` 在核心鍵值儲存之上提供向量搜尋流程。

***

©️ 2026 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
