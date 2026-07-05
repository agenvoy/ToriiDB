# toriidb - 架構

> 返回 [README](./README.zh.md)

## 概覽

```mermaid
graph TB
    A[REPL 用戶端] --> B[Store 工作階段]
    B --> C[指令路由器]
    C --> D[鍵值操作]
    C --> E[JSON 欄位操作]
    C --> F[向量操作]
    D --> G[資料庫 0-15]
    E --> G
    F --> G
    G --> H[AOF 重播與追加]
    G --> I[逐鍵 JSON 檔案]
    F --> J[OpenAI 嵌入器]
```

## Module: REPL Entry

命令列測試客戶端負責讀取使用者輸入、顯示提示字元，並把每一條指令送到 store session。

```mermaid
graph TB
    subgraph REPL 入口
        A[stdin 讀取器] --> B[修剪與驗證]
        B --> C[離開處理]
        B --> D[Store.Exec]
        D --> E[stdout 輸出器]
    end
    F[作業系統訊號] --> C
```

## Module: Command Router

路由器會解析指令 token，並分派到鍵值、JSON、TTL、查詢與向量處理流程。

```mermaid
graph TB
    subgraph 指令路由器
        A[輸入字串] --> B[strings.Fields]
        B --> C[指令分派]
        C --> D[GET SET DEL INCR]
        C --> E[TTL EXPIRE PERSIST SELECT]
        C --> F[FIND QUERY KEYS]
        C --> G[VSEARCH VSIM VGET]
        C --> H[點記法切分]
    end
```

## Module: Storage Core

儲存核心維護 16 個資料庫，每個資料庫都有自己的鎖、記憶體 map、AOF 檔案與延遲載入生命週期。

```mermaid
graph TB
    subgraph 儲存核心
        A[Store] --> B[allDBs 0-15]
        B --> C[db 結構]
        C --> D["data map[string]*Entry"]
        C --> E[sync.RWMutex]
        C --> F[record.aof]
        C --> G[延遲載入 once]
    end
```

## Module: JSON Document Engine

文件輔助流程會同步維持原始字串值與解析後的 JSON 快取，並支援巢狀欄位增修。

```mermaid
graph TB
    subgraph JSON 文件引擎
        A[Entry.value] --> B[setValue]
        A --> C[setParsed]
        C --> D[parsed 快取]
        D --> E[GetField]
        D --> F[Query]
        C --> G[SetField IncrField DelField]
        H[WalkKeys] --> E
    end
```

## Module: Vector Search Pipeline

向量功能會嵌入文字、把查詢向量快取在內部，並以餘弦相似度為候選 entry 排名。

```mermaid
graph TB
    subgraph 向量搜尋流程
        A[SET ... VECTOR 或 VSEARCH] --> B[resolveQueryVector]
        B --> C[內部向量快取]
        B --> D[OpenAI Client]
        D --> E[Embedding]
        E --> F[Entry.Vector]
        F --> G[scanTopK]
        G --> H[餘弦排名]
        H --> I[Top-K 結果鍵值]
    end
```

## 資料流

```mermaid
sequenceDiagram
    participant User as 使用者
    participant REPL as REPL
    participant Router as 指令路由器
    participant DB as 目前資料庫
    participant Disk as AOF/JSON 快取
    User->>REPL: 輸入指令
    REPL->>Router: Exec(input)
    Router->>DB: 呼叫處理器
    DB->>Disk: 追加 AOF / 更新檔案
    DB-->>Router: 回傳結果
    Router-->>REPL: 組合輸出
    REPL-->>User: 顯示回應
```

## 狀態機

```mermaid
stateDiagram-v2
    [*] --> 閒置
    閒置 --> 解析中: 收到指令
    解析中 --> 執行中: 指令有效
    解析中 --> 閒置: 輸入為空
    執行中 --> 持久化中: 寫入操作
    執行中 --> 閒置: 唯讀結果
    持久化中 --> 閒置: 成功
    持久化中 --> 錯誤: 失敗
    錯誤 --> 閒置: 下一個指令
```

***

©️ 2026 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
