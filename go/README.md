# Go 命名規則ガイド（詳細 + 実例）

![Language](https://img.shields.io/badge/Language-Go-lightblue.svg)
![Docs](https://img.shields.io/badge/Docs-Naming%20Guide-blue.svg)
![Updated](https://img.shields.io/badge/Updated-2025--11--09-yellow.svg)

このファイルは各セクションに実務で使える具体的なコード例を追加した `go/README.md` の拡張版です。

## 目次

- [Quick Summary](#quick-summary)
- [パッケージ / ファイル](#パッケージ--ファイル)
- [型、構造体、インタフェース（具体例）](#型構造体インタフェース具体例)
- [変数・定数・列挙（具体例）](#変数定数列挙具体例)
- [関数・メソッド・レシーバ（具体例）](#関数メソッドレシーバ具体例)
- [Initialisms（頭字語）](#initialisms頭字語)
- [エラーと sentinel 値 / 比較（具体例）](#エラーと-sentinel-値--比較具体例)
- [コンストラクタ / ファクトリ / Builder（具体例）](#コンストラクタ--ファクトリ--builder具体例)
- [ジェネリクス（具体例）](#ジェネリクス具体例)
- [テスト命名とテスト設計（具体例）](#テスト命名とテスト設計具体例)
- [Concurrency / 同期（具体例）](#concurrency--同期具体例)
- [Context とパラメータ順序（具体例）](#context-とパラメータ順序具体例)
- [Package comment / godoc（具体例）](#package-comment--godoc具体例)
- [Logger / structured logging のフィールド命名（具体例）](#logger--structured-loggingのフィールド命名具体例)
- [Config / 環境変数命名（具体例）](#config--環境変数命名具体例)
- [テストヘルパー / フィクスチャ命名（具体例）](#テストヘルパー--フィクスチャ命名具体例)
- [Exported vs Unexported（具体例）](#exported-vs-unexported具体例)
- [自動生成コードの命名取り扱い（具体例）](#自動生成コードの命名取り扱い具体例)
- [よくあるアンチパターン（収束）](#よくあるアンチパターン収束)
- [付録: 推奨頭字語一覧 / 参照リンク](#付録-推奨頭字語一覧--参照リンク)

---

## Quick Summary

（要点は既存の Quick Summary を参照）

---

## パッケージ / ファイル

具体例: 典型的プロジェクト構成

```
/myrepo
  /cmd
    /myapp
      main.go        // package main
  /pkg
    /billing
      billing.go     // package billing
      client.go
      client_test.go // package billing
  /internal
    /store
      store.go       // package store (internal)
```

package の命名ルールに従うことで import が自然になり、`billing.NewClient` のように読める API を作れます。

---

## 型、構造体、インタフェース（具体例）

- 小さく役割のはっきりしたインタフェースを設計する。

良い例（小さいインタフェースと実装）:

```go
// package store

type Reader interface {
    Read(ctx context.Context, id string) (*Record, error)
}

type Writer interface {
    Write(ctx context.Context, r *Record) error
}

// 組み合わせて使う
type Store interface {
    Reader
    Writer
}
```

悪い例（巨大なインタフェース）:

```go
// package store

// BigStore は多すぎる責務を持つ
type BigStore interface {
    Create(...)
    Update(...)
    Delete(...)
    Query(...)
    Migrate(...)
    // ...
}
```

---

## 変数・定数・列挙（具体例）

良い例: iota を使った列挙とパッケージスコープ定数

```go
package order

type Status int

const (
    StatusUnknown Status = iota
    StatusPending
    StatusPaid
    StatusCancelled
)

const DefaultPageSize = 20
```

悪い例: グローバルの可変マップ

```go
var cache = map[string]*User{} // NG: レースやテストの原因
```

---

## 関数・メソッド・レシーバ（具体例）

- レシーバは型の先頭文字 `s *Server` など。ポインタ vs 値の判断も示す。

良い例:

```go
type Server struct{
    addr string
}

func (s *Server) Start(ctx context.Context) error {
    // s を変更する可能性があるためポインタレシーバ
    return nil
}

func (s Server) Addr() string { // 値レシーバでも ok（不変）
    return s.addr
}
```

悪い例:

```go
func (serverImplementation *ServerImplementation) Start() {} // 冗長な名前
```

---

## Initialisms（頭字語）

具体例:

```go
type HTTPClient struct{}
func (c *HTTPClient) Do(req *Request) {}

type URLBuilder struct{}
```

プロジェクト内で `HTTP` か `Http` のどちらかに統一する（推奨: `HTTP`）。

---

## エラーと sentinel 値 / 比較（具体例）

良い例（公開 sentinel と errors.Is）:

```go
var ErrNotFound = errors.New("not found")

func Get(ctx context.Context, id string) (*Item, error) {
    if id == "" { return nil, ErrNotFound }
    return &Item{ID: id}, nil
}

if errors.Is(err, ErrNotFound) {
    // handle not found
}
```

悪い例（文字列比較）:

```go
if err.Error() == "not found" { // fragile
    // ...
}
```

ラップされたエラーのパターン:

```go
if err := do(); err != nil {
    return fmt.Errorf("do failed: %w", err)
}
```

---

## コンストラクタ / ファクトリ / Builder（具体例）

良い例:

```go
func NewClient(baseURL string) *Client {
    return &Client{BaseURL: baseURL}
}

func NewClientWithTimeout(baseURL string, timeout time.Duration) *Client {
    return &Client{BaseURL: baseURL, Timeout: timeout}
}
```

悪い例:

```go
func NewC(b string) *C { return &C{} } // 読めない
```

---

## ジェネリクス（具体例）

```go
func Map[T any](in []T, f func(T) T) []T {
    out := make([]T, len(in))
    for i, v := range in { out[i] = f(v) }
    return out
}

// 制約名をわかりやすくする
type Number interface{ ~int | ~float64 }
func Sum[T Number](xs []T) T { /*...*/ }
```

---

## テスト命名とテスト設計（具体例）

良い example: テーブル駆動テスト + t.Run

```go
func TestAdd(t *testing.T) {
    cases := []struct{ name string; a, b, want int }{
        {"both positive", 1,2,3},
        {"zero", 0,0,0},
    }
    for _, c := range cases {
        t.Run(c.name, func(t *testing.T) {
            if got := Add(c.a,c.b); got != c.want { t.Fatalf("want %v got %v", c.want, got) }
        })
    }
}
```

アンチパターン: 外部 DB に直接繋ぐテスト。→ モック化/DB sandbox を使う。

---

## Concurrency / 同期（具体例）

良い例: WaitGroup + error チャネルパターン

```go
var wg sync.WaitGroup
errCh := make(chan error, 1)
for _, job := range jobs {
    wg.Add(1)
    go func(j Job) {
        defer wg.Done()
        if err := j.Run(); err != nil { select { case errCh <- err: default: } }
    }(job)
}
wg.Wait()
select { case err := <-errCh: return err; default: }
```

悪い例: グローバル変数で状態を共有しロックを忘れる

---

## Context とパラメータ順序（具体例）

良い:

```go
func Fetch(ctx context.Context, id string) (*Thing, error)
```

悪い:

```go
func Fetch(id string, ctx context.Context) (*Thing, error) // NG
```

---

## Package comment / godoc（具体例）

良い例:

```go
// Package billing provides clients and services for billing operations.
package billing
```

---

## Logger / structured logging のフィールド命名（具体例）

良い:

```go
logger.Info("order processed", zap.String("order_id", order.ID), zap.Int("item_count", len(order.Items)))
```

悪い:

```go
logger.Info("order processed", "OrderId", order.ID) // 一貫性がない
```

---

## Config / 環境変数命名（具体例）

- 環境変数: `APP_PORT`, `DATABASE_URL`
- コード変数: `dbURL`, `appPort`

---

## テストヘルパー / フィクスチャ命名（具体例）

```go
func newTestDB(t *testing.T) *sql.DB { db, _ := sql.Open("sqlite3", ":memory:"); return db }
```

---

## Exported vs Unexported（具体例）

良い:

```go
// exported for testing
var nowFunc = time.Now
```

悪い（公開しすぎ）:

```go
// package api
type internalConfig struct{} // exported accidentally if capitalized
```

---

## 自動生成コードの命名取り扱い（具体例）

- protoc で生成される型は `pb.` prefix を使ってパッケージ分離する（例: `pb.User`）。
- 生成コードは編集せず、変換関数で内部型にマッピングする。

```go
func ToDomain(u *pb.User) *domain.User { return &domain.User{ID:u.Id, Name:u.Name} }
```

---

## よくあるアンチパターン（収束）

短くまとめると:

- API をエクスポートしすぎない
- 受信者名を冗長にしない
- エラー比較は errors.Is を使う
- グローバル可変状態を避ける

---

## 付録: 推奨頭字語一覧 / 参照リンク

- 推奨頭字語: API, HTTP, HTTPS, URL, URI, ID, UUID, SQL, JSON, XML, HTML, TLS, DB, IO, CPU

参考:

- Effective Go: https://go.dev/doc/effective_go
- Go Code Review Comments: https://github.com/golang/go/wiki/CodeReviewComments
- improver-tech 記事（日本語まとめ）: https://improver-tech.com/go%E8%A8%80%E8%AA%9E%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8B%E5%91%BD%E5%90%8D%E8%A6%8F%E5%89%87%E3%81%BE%E3%81%A8%E3%82%81/

---
