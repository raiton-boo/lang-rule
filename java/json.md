# JSON / シリアライズ（詳細）

このドキュメントは `java/java.md` の「JSON / シリアライズ」節の詳細補足です。
主に Jackson を想定した実用的な注意と、DTO ⇄ ドメインのマッピング方針をまとめます。

## 目的

- 外部 API との契約（snake_case など）と内部モデル（lowerCamelCase）のギャップを明確化する。
- 破壊的な変更を避けるためのマイグレーション方針を提示する。

## 命名とアノテーション

- 内部モデルは Java の命名規則（lowerCamelCase）を維持する。
- 外部契約に合わせる場合は DTO を用意し、Jackson の `@JsonProperty` でマッピングする。

例:

```java
public class UserDto {
  @JsonProperty("user_id")
  private String userId;

  @JsonProperty("created_at")
  private String createdAt; // ISO 8601 などを想定
}
```

- Jackson の PropertyNamingStrategy（例: `PropertyNamingStrategies.SNAKE_CASE`）を使うと DTO 側でのアノテーションを減らせるが、API で複雑な変換がある場合は明示的な `@JsonProperty` を推奨します。

## 日付・時刻の扱い

- API で使うフォーマット（ISO 8601 など）をチームで決める。
- DTO では文字列で受け取り、マッピング層で `OffsetDateTime` などに変換する方が安全。

## マッピング層

- DTO ⇄ ドメインの変換は MapStruct のようなライブラリ、または明示的な変換ユーティリティを用いる。
- 直接ドメインを JSON に晒すべきではない（将来的なフィールド追加やシリアライズ制御のため）。

MapStruct 例（概念）:

```java
@Mapper
public interface UserMapper {
  User toDomain(UserDto dto);
  UserDto toDto(User user);
}
```

## バージョニングと互換性

- API にバージョンを付ける（例: `/api/v1/users`）ことで、後方互換性を保ちやすくする。
- フィールド追加は互換性を壊さない範囲で行い、削除／意味変更はメジャーバージョンで行う。

## Null と Optional の方針

- JSON レスポンスで値が存在しない場合と null を区別する必要がある場合は DTO 側で `Optional` を使うか、明示的なフィールド設計を検討する。

## テストと契約

- 外部契約（JSON スキーマ）については契約テストを用意する（例: Pact や契約テストの自動化）。

## まとめ（推奨ワークフロー）

1. 外部 API の契約定義 → 2. DTO を定義（必要なら @JsonProperty）→ 3. MapStruct 等でドメインに変換 → 4. ドメインで処理 → 5. 必要に応じて DTO に変換して返却

---
