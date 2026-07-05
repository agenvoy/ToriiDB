> [!NOTE]
> 此 README 由 [SKILL](https://github.com/pardnchiu/skill-readme-generate) 生成，英文版請參閱 [這裡](../README.md)。

***

<p align="center">
<strong>EMBEDDED JSON KV STORAGE WITH REDIS-LIKE COMMANDS AND VECTOR SEARCH</strong>
</p>

<p align="center">
<a href="https://pkg.go.dev/github.com/agenvoy/toriidb"><img src="https://img.shields.io/badge/GO-REFERENCE-blue?include_prereleases&style=for-the-badge" alt="Go Reference"></a>
<a href="Release"><img src="https://img.shields.io/github/v/tag/agenvoy/toriidb?include_prereleases&style=for-the-badge" alt="Release"></a>
<a href="https://github.com/agenvoy/toriidb/releases"><img src="https://img.shields.io/github/license/agenvoy/toriidb?include_prereleases&style=for-the-badge" alt="License"></a>
<a href="https://app.codecov.io/github/agenvoy/toriidb/tree/master"><img src="https://img.shields.io/codecov/c/github/agenvoy/toriidb/master?include_prereleases&style=for-the-badge" alt="Coverage"></a>
</p>

***

> Go 內嵌式資料庫，具備 Redis 風格指令、JSON 欄位操作與語意向量搜尋

## 目錄

- [功能特點](#功能特點)
- [架構](#架構)
- [授權](#授權)
- [Author](#author)
- [Stars](#stars)

## 功能特點

> `go get github.com/agenvoy/toriidb` · [完整文件](./doc.zh.md)

- **Redis 風格 REPL** — 透過單一指令路由器提供熟悉的互動式命令體驗，方便快速測試與操作資料。
- **JSON 欄位增修** — 使用點記法直接存取巢狀欄位，無需每次重寫整份 JSON 文件。
- **本地持久化** — 以記憶體維持操作速度，同時透過 AOF 與逐鍵 JSON 快取保留寫入結果。
- **內建向量搜尋** — 可為鍵值附加 embedding，並在同一套儲存模型中完成語意搜尋與相似度比較。

## 架構

> [完整架構](./architecture.zh.md)

```mermaid
graph TB
    A[REPL 用戶端] --> B[指令路由器]
    B --> C[鍵值命令]
    B --> D[JSON 欄位操作]
    B --> E[向量命令]
    C --> F[記憶體資料庫]
    D --> F
    E --> F
    F --> G[AOF 與 JSON 快取]
    E --> H[OpenAI 向量嵌入器]
```

## 授權

本專案採用 [MIT LICENSE](../LICENSE)。

## Author

<img src="https://github.com/pardnchiu.png" align="left" width="96" height="96" style="margin-right: 0.5rem;">

<h4 style="padding-top: 0">邱敬幃 Pardn Chiu</h4>

<a href="mailto:hi@pardn.io">hi@pardn.io</a><br>
<a href="https://www.linkedin.com/in/pardnchiu">https://www.linkedin.com/in/pardnchiu</a>

## Stars

[![Star](https://api.star-history.com/svg?repos=agenvoy/toriidb&type=Date)](https://www.star-history.com/#agenvoy/toriidb&Date)

***

©️ 2026 [邱敬幃 Pardn Chiu](https://www.linkedin.com/in/pardnchiu)
