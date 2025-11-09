# Java 命名規則ガイド（詳細版）

![Language](https://img.shields.io/badge/Language-Java-red.svg)
![Docs](https://img.shields.io/badge/Docs-Naming%20Guide-blue.svg)
![Updated](https://img.shields.io/badge/Updated-2025--11--05-yellow.svg)

---

## 目次

- [Quick Summary（要点）](#quick-summary要点)
- [パッケージ名](#パッケージ名)
- [ファイル名と公開クラス](#ファイル名と公開クラス)
- [クラス / インタフェース /列挙型](#クラス--インタフェース--列挙型)
- [定数（補強例）](#定数補強例)
- [変数・フィールド](#変数・フィールド)
- [メソッド](#メソッド)
- [ジェネリクス（補強例）](#ジェネリクス補強例)
- [テスト命名（補強例）](#テスト命名補強例)
- [ラムダ / ストリーム内の変数名](#ラムダ--ストリーム内の変数名)
- [頭字語（Acronym）方針 — CamelCase（推奨）](#頭字語acronym方針--camelcase推奨)
- [良い例 / 悪い例（対比表）](#良い例--悪い例対比表)
- [JSON / シリアライズ（簡潔な注意）](#json--シリアライズ簡潔な注意)
- [エッジケースと移行方針（簡潔）](#エッジケースと移行方針簡潔)
- [参考リンク](#参考リンク)
- [更新履歴](#更新履歴)

---

## Quick Summary（要点）

- パッケージ: 小文字、ドメイン逆順、機能単位で区切る。
- ファイル: public クラスとファイル名は一致させる。
- クラス/インタフェース: UpperCamelCase（意味のある名詞）。
- メソッド/変数: lowerCamelCase。boolean は is/has/can を優先。
- 定数: UPPER_SNAKE_CASE。単位（\_MS / \_SECONDS）を含める。

---

## パッケージ名

- すべて小文字。ドメイン逆順で始める（例: `com.example.project`）。
- 機能別にサブパッケージを切る（`service`, `repository`, `api`, `dto`, `internal` など）。
- 「module」「feature」など曖昧な命名は避け、責務を表現する。

良い例:

```
com.example.billing.service
com.example.billing.repository
com.example.billing.api.v1
```

悪い例:

```
com.example.Billing_Service   // 大文字やアンダースコアを含む
com.example.module1          // 何をする module1 かわからない
```

メモ: テストパッケージは通常ソースと同じパッケージ構造にする（例: `src/test/java/com/example/...`）。

## ファイル名と公開クラス

- public クラスはファイル名と一致させる（`UserService.java` に `public class UserService`）。
- 1 ファイルに複数の public クラスを置かない。

例:

```java
// UserService.java
public class UserService { ... }
```

## クラス / インタフェース /列挙型

- UpperCamelCase。クラス名は意味のある名詞（`OrderService`, `PaymentProcessor`）。
- インタフェースは名詞でもよい。`*able` は適切に使う（`Serializable` 等）。
- 列挙型はカテゴリ名（`OrderStatus`）。列挙子は UPPER_SNAKE_CASE を推奨。

良い例:

```java
public class OrderService { }
public interface PaymentProvider { }
public enum OrderStatus { NEW, PAID, CANCELLED }
```

悪い例:

```java
public class order_service { }   // 小文字／アンダースコアは NG
```

ネストクラス: 小さくて外部に公開しないユーティリティはネスト可能。公開するものはトップレベルに。

## 定数（補強例）

- static final は UPPER_SNAKE_CASE。単位を含める（例: `_MS`, `_SECONDS`）。

良い例:

```java
// public API に晒す定数（説明コメントを添える）
public static final String DEFAULT_DATE_FORMAT = "yyyy-MM-dd";
public static final int MAX_RETRIES = 3;

// 単位を含める例
private static final long CACHE_TTL_SECONDS = 60L;
private static final int TIMEOUT_MS = 5000;
```

悪い例:

```java
public static final int maxRetries = 3; // lowerCamelCase は NG
```

## 変数・フィールド

- インスタンス・ローカル変数は lowerCamelCase。
- フィールドプレフィックス（`m` や `_`）は使用しない方針（混在が問題になるため）。
- final フィールドは不変を示す命名／修飾子を使う。

例:

```java
private final String userName;
int retryCount;
```

ローカル変数:

- ループインデックスは `i,j,k` を許容。役割が明確なら `index`, `offset` を使う。

## メソッド

- lowerCamelCase。動詞または動詞句で命名する（`calculateTax`, `findUserById`）。
- boolean を返すメソッドは `is`/`has`/`can` で始める（`isEnabled`, `hasNext`）。
- 副作用があるか否かを名前で表す（`save`, `create`, `delete` は副作用あり）。

ファクトリ / ビルダ系:

- 静的ファクトリは `of`, `from`, `valueOf`, `create` などを使う。
- ビルダは `builder()` / `newBuilder()`。

例:

```java
public User findById(String id) { ... }
public boolean isActive() { ... }
public static Order of(OrderRequest req) { ... }
```

悪い例:

```java
public void data() { ... } // 何をするか不明
```

## ジェネリクス（補強例）

- 単一文字の慣例: `T, E, K, V, R`。
- 境界付き型やワイルドカードの実用例を示す。

例:

```java
// 境界付きジェネリクス
public interface Repository<T extends Entity> {
    Optional<T> findById(String id);
}

// ワイルドカードの例（読み取り専用）
public void processAll(Collection<? extends Number> numbers) { ... }
```

## テスト命名（補強例）

- 推奨パターン: `shouldX_whenY`, `givenX_whenY_thenZ`, `method_condition_expected` のいずれかをチームで統一。

例:

```java
@Test
void shouldReturnEmpty_whenCartIsEmpty() { ... }

@Test
void calculateTotal_givenEmptyCart_returnsZero() { ... }

@Test
void givenValidId_whenFindById_thenReturnUser() { ... }
```

## ラムダ / ストリーム内の変数名

- 短くても意味が分かる名前を優先。`s -> s.length()` より `str -> str.length()` の方が読みやすい。

例:

```java
list.stream()
    .map(user -> user.getName())
    .filter(name -> !name.isEmpty())
    .collect(Collectors.toList());
```

## 頭字語 (Acronym) 方針 — CamelCase（推奨）

- 方針: 頭字語は先頭のみ大文字にする（例: `HttpClient`, `XmlParser`, `FtpServer`）。
- 理由: Java の命名慣習に沿い、可読性と一貫性を保ちやすい。

具体例:

```
Good: HttpClient, XmlReader, FtpServer
Bad: HTTPClient, xmlReader
```

注: 定数は UPPER_SNAKE_CASE のルールに従うため、略語は大文字で表記される（例: `API_V1_PATH`）。

## 良い例 / 悪い例（対比表）

| 要素         | 良い例                       | 悪い例                               |
| ------------ | ---------------------------- | ------------------------------------ |
| クラス名     | `UserManager`                | `userManager`                        |
| 定数         | `MAX_SIZE`                   | `MaxSize`                            |
| メソッド     | `isEnabled()`                | `getEnabledFlag()`                   |
| ジェネリクス | `Repository<T, ID>`          | `Repository<Object, String>`（曖昧） |
| テスト       | `shouldReturn400WhenInvalid` | `test1`                              |

## JSON / シリアライズ

- 内部ドメインは Java 命名規則（lowerCamelCase）に従う。外部 API の契約（snake_case など）がある場合は `@JsonProperty` で明示する。DTO と内部モデルは分け、マッピング層で変換することを推奨。

例:

```java
public class UserDto {
  @JsonProperty("user_id")
  private String userId;
}
```

詳細は `java/json.md` を参照。

## エッジケースと移行方針

- 既存レガシーとの衝突: 新規コードは新ルールに従う。API 変更が伴う場合は段階的移行を検討。
- 自動生成コード: 生成物は編集しない。必要なら変換レイヤーを用意する（詳細は `java/autogen.md` を参照）。

## 参考リンク

- Qiita — 命名規則まとめ（rkonno）: https://qiita.com/rkonno/items/1b30daf83854fecbb814
- Google Java Style Guide: https://google.github.io/styleguide/javaguide.html
- Effective Java (Joshua Bloch): https://www.oreilly.co.jp/books/9784873115658/ (日本語版紹介)

---