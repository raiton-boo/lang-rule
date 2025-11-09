# Python 命名規則ガイド（PEP8 + Docstring と typing — 実務向け）

![Language](https://img.shields.io/badge/Language-Python-blue.svg)
![Docs](https://img.shields.io/badge/Docs-Naming%20Guide-blue.svg)
![Updated](https://img.shields.io/badge/Updated-2025--11--09-yellow.svg)

このドキュメントは PEP8 を基礎に、docstring（Google / NumPy スタイル）、型ヒント（typing）、デコレータ・ラムダ等の機能の実務的な使い方、テスト、例外、自動生成コードの扱いについてまとめた実務向けガイドです。

目次

- [Quick Summary](#quick-summary)
- [モジュール / パッケージ名](#モジュール--パッケージ名)
- [ファイル名](#ファイル名)
- [クラス名](#クラス名)
- [関数 / メソッド名](#関数--メソッド名)
- [変数 / 定数](#変数--定数)
- [非公開 / 名前修飾 (\_ / \_\_)](#非公開--名前修飾-_--__)
- [Docstrings（書式と例）](#docstrings書式と例)
- [型ヒント（typing）——詳解](#型ヒントtyping——詳解)
- [デコレータ（decorator）](#デコレータdecorator)
- [ラムダ関数（lambda）](#ラムダ関数lambda)
- [例外（Exception）命名](#例外exception命名)
- [テスト命名（pytest）と fixture](#テスト命名pytestと-fixture)
- [ロギング / フィールド名](#ロギング--フィールド名)
- [環境変数 / 設定](#環境変数--設定)
- [自動生成コードの扱い（プロトコル / OpenAPI）](#自動生成コードの扱いプロトコル--openapi)
- [アンチパターン](#アンチパターン)
- [参考リンク](#参考リンク)

---

## Quick Summary

- モジュール/パッケージ: 小文字、必要ならアンダースコア（`my_module`）。
- クラス: CapWords（UpperCamelCase）。
- 関数/変数: snake_case。
- 定数: UPPER_SNAKE_CASE。
- Docstrings: Google または NumPy スタイルをプロジェクトで一貫して採用。

---

## モジュール / パッケージ名

- 小文字のみ。アンダースコアは必要最小限に。
- 名前は短く、責務が分かるように（例: `mypkg.auth`）。

良い例:

```
package: mypkg
module: my_module.py
```

悪い例:

```
package: MyPkg  # NG（大文字）
module: my-module.py  # NG（ハイフン）
```

## ファイル名

- 小文字・スネークケース。テストは `test_*.py`。
- CLI エントリーポイントは `__main__.py` や `cli.py` を使う。

## クラス名

- CapWords（UpperCamelCase）。
- 抽象クラスには `Base`、Mixin には `Mixin` をサフィックスにする。

例:

```python
class UserService:
    pass

class BaseRepository:
    pass
```

## 関数 / メソッド名

- snake*case。副作用がある場合は動詞（例: `save_user`）。真偽値を返す関数は `is*`/`has\_` をプレフィックスに使う。

- 公開 API には必ず明確な docstring と型ヒントを付ける。

例:

```python
def fetch_user(user_id: str) -> "User":
    """Fetch a user by id.

    Args:
        user_id: Unique identifier for the user.

    Returns:
        User instance if found.
    """
    ...
```

## 変数 / 定数

- 変数: snake_case
- 定数: UPPER_SNAKE_CASE

例:

```python
DEFAULT_TIMEOUT = 30
user_count = 0
```

## 非公開 / 名前修飾 (\_ / \_\_)

- `_name`: 慣習的に非公開。モジュール単位の非公開 API を示す際に使う。
- `__name`: name-mangling（クラス名修飾）を発生させるため、通常は避ける。主に継承時の衝突回避が必要な特殊ケースでのみ利用する。

例:

```python
class Foo:
    def _helper(self):
        pass
```

---

## Docstrings（書式と例）

- プロジェクトで Google スタイルか NumPy スタイルを選び、一貫して使う。
- フォーマット: 1 行要約 → 空行 → 詳細（Args/Parameters, Returns, Raises, Examples）
- 型ヒントがある場合、Args に型注記を繰り返す必要はない（説明は残す）。

Google スタイル例:

```python
def fetch_user(user_id: str) -> "User":
    """Fetch a user by id.

    Args:
        user_id: Unique identifier for the user.

    Returns:
        User instance if found.

    Raises:
        ValueError: If user_id is invalid.
    """
    ...
```

NumPy スタイル例:

```python
def add(a: int, b: int) -> int:
    """Add two numbers.

    Parameters
    ----------
    a : int
        First operand.
    b : int
        Second operand.

    Returns
    -------
    int
        Sum of a and b.
    """
    return a + b
```

---

## 型ヒント（typing）——詳解

概要／契約:

- 目的: 型ドキュメントとしての役割を果たし、静的検査（mypy/ruff）でバグを早期検出する。
- 入出力: 関数の引数と戻り値に対して具体的な型を付ける。公開 API では曖昧な `Any` を避ける。

主要トピックと実例:

1. 基本

- 基本型: `str`, `int`, `float`, `bool`。
- 3.9+/3.10+ の構文を推奨: `list[str]`, `dict[str, int]`, `str | None`。

```python
from typing import Optional

def get_user(id: str) -> Optional[User]:
    ...

def find_users(name: str) -> list[User]:
    ...
```

2. TypeVar / Generic

- 再利用可能なジェネリクスには `TypeVar` を使う。ボンド（bound）や共変/反変も指定可能。

```python
from typing import TypeVar, Generic

T = TypeVar('T')

class Repository(Generic[T]):
    def save(self, obj: T) -> None: ...
```

3. Callable / Protocol

- 単純な関数シグネチャには `Callable`、より構造的なインタフェースには `Protocol` を使う。

```python
from typing import Callable, Protocol

Handler = Callable[[str, int], bool]

class Serializer(Protocol):
    def serialize(self, obj: object) -> str: ...
```

Protocol は `@runtime_checkable` を併用するとランタイム isinstance チェックが可能です（注意: 制約あり）。

4. TypedDict / dataclass

- `TypedDict` は JSON レスポンスや外部スキーマを型で表現するのに便利です。ランタイムでの検証が必要な場合は `pydantic` や `attrs` などと組み合わせることを推奨します。

基本例:

```python
from typing import TypedDict

class UserDict(TypedDict):
    id: str
    name: str
```

optional キーや部分的なスキーマ（`total=False`）:

```python
from typing import TypedDict

class PartialUser(TypedDict, total=False):
    id: str
    name: str

# total=False にするとキーが揃っていない辞書も型として扱える
```

PEP 655 の NotRequired（Python のバージョン差異）:

```python
from typing_extensions import NotRequired, TypedDict

class UserV2(TypedDict):
    id: str
    name: NotRequired[str]

# 注: 古い Python では typing_extensions を使う
```

ランタイム検証／変換の実務例:

1. TypedDict を受け取り内部の dataclass / domain object にマッピングする（生成コードや外部 API のアダプタ層）

```python
from dataclasses import dataclass
from typing import TypedDict

class UserDict(TypedDict):
    id: str
    name: str

@dataclass
class User:
    id: str
    name: str

def user_from_dict(d: UserDict) -> User:
    # 最低限のバリデーション
    return User(id=d['id'], name=d.get('name', ''))
```

2. 厳密なランタイム検証が必要な場合は `pydantic` などを使う（型安全かつ検証済みオブジェクトを得られる）:

```python
from pydantic import BaseModel

class UserModel(BaseModel):
    id: str
    name: str | None = None

data = {'id': 'u1', 'name': 'Alice'}
user = UserModel(**data)
```

運用上のガイドライン:

- 型は静的検査のために用い、ランタイムでの検証が必要なら `pydantic` / `marshmallow` / 手続き的チェックを組み合わせる。
- `TypedDict` を API レスポンスの型として使いつつ、内部では dataclass/pydantic に変換して境界を明確にするパターンが安全で保守的です。

5. ParamSpec / Concatenate（デコレータ向け）

- デコレータで呼び出しシグネチャを保つには `ParamSpec` と `TypeVar` を組み合わせる。

基本例（関数に使う簡易な preserve_signature）:

```python
from typing import TypeVar, ParamSpec, Callable
from functools import wraps

P = ParamSpec('P')
R = TypeVar('R')

def preserve_signature(func: Callable[P, R]) -> Callable[P, R]:
    @wraps(func)
    def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
        # 前後処理
        return func(*args, **kwargs)

    return wrapper

@preserve_signature
def greet(name: str) -> str:
    return f"hello {name}"
```

実務で多いパターンは「メソッド」向けのデコレータです。メソッドは先頭に `self`（あるいは `cls`）が来るため、`Concatenate` を使って先頭引数を明示すると型情報を正確に保てます。

メソッド向けの実践例 — 認可チェックデコレータ（`Concatenate` を使用）:

```python
from typing import TypeVar, ParamSpec, Callable, Concatenate
from functools import wraps
import inspect

P = ParamSpec('P')
R = TypeVar('R')

def require_role(role: str) -> Callable[[Callable[Concatenate["Self", P], R]], Callable[Concatenate["Self", P], R]]:
    # 注: 型注釈内の "Self" は実際の self 型名に置き換わるものではなく、可読性のために示しています。
    def decorator(func: Callable[Concatenate["Self", P], R]) -> Callable[Concatenate["Self", P], R]:
        @wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            # args[0] は self に相当することを期待
            self = args[0]
            # 例: self.current_user.roles をチェック
            user = getattr(self, 'current_user', None)
            if user is None or role not in getattr(user, 'roles', []):
                raise PermissionError("missing role")
            return func(*args, **kwargs)

        return wrapper

    return decorator

class Service:
    def __init__(self, user):
        self.current_user = user

    @require_role('admin')
    def sensitive_operation(self, data: dict) -> bool:
        """管理者のみが実行できる処理"""
        return True

# 実行側は通常どおり呼べる
user = type('U', (), {'roles': ['admin']})()
svc = Service(user)
svc.sensitive_operation({'x': 1})
```

注記:

- `Concatenate` を使うと、`self` を含むメソッドのシグネチャを型レベルで表現できます。ただし、現状の mypy 等の挙動はバージョン依存のため CI で検証してください。
- 型チェックが厳しい場合は `# type: ignore` を使って意図を明示するか、デコレータをファクトリ関数にして明確に型注釈を与えるパターンにするのも現実的です。

6. 実務上の注意

- `Any` を安易に使わない。必要がある場合は `# type: ignore` と理由をコメントする。
- 公開 API は型注釈を丁寧に書き、mypy/ruff で CI チェックを行うことを推奨。

---

## デコレータ（decorator）

ポイント:

- デコレータは振る舞いの拡張に便利だが、シグネチャやドキュメントを失いやすい。
- `functools.wraps` と `ParamSpec` を使い、型とメタ情報（**name**, **doc**）を保持する。

実用例 — ログ出力付きデコレータ:

```python
from typing import TypeVar, ParamSpec, Callable
from functools import wraps
import logging

P = ParamSpec('P')
R = TypeVar('R')

def log_call(level: int = logging.INFO) -> Callable[[Callable[P, R]], Callable[P, R]]:
    def decorator(func: Callable[P, R]) -> Callable[P, R]:
        @wraps(func)
        def wrapper(*args: P.args, **kwargs: P.kwargs) -> R:
            logging.log(level, "calling %s", func.__name__)
            return func(*args, **kwargs)

        return wrapper

    return decorator

@log_call()
def add(a: int, b: int) -> int:
    return a + b
```

注意点:

- デコレータは副作用（例: キャッシュ、ログ、認可）に使うが、過剰なロジックを入れるとテストしづらくなる。
- 型チェックをパスさせるためには `ParamSpec` を使うのが現実的。

---

## ラムダ関数（lambda）

使いどころ:

- 単純な一時関数（キー関数、短い変換）にはラムダで可。
- 複雑な処理や再利用するロジックは命名関数（`def`）にしてテストとドキュメントを付ける。

例:

```python
# 簡潔なキー関数
sorted_users = sorted(users, key=lambda u: u.name)

# 一時的な短い式
add = lambda x, y: x + y
```

アンチパターン:

- 長い式や複数行のロジックをラムダに詰め込む。可読性が著しく低下する。
- デバッグメッセージや複雑な分岐がある処理をラムダにするのは避ける。

代替:

- 名前を付けた関数にして docstring を書く。

```python
def user_sort_key(u: User) -> str:
    """Return the sort key for a user."""
    return u.name

sorted_users = sorted(users, key=user_sort_key)
```

---

## 例外（Exception）命名

- 例外クラスは PascalCase で末尾に `Error` や `Exception` を付ける（例: `NotFoundError`）。

```python
class NotFoundError(Exception):
    """Raised when an entity is not found."""
    pass
```

---

## テスト命名（pytest）と fixture

- テスト関数は `test_` で始める。ファイルは `test_*.py`。
- fixture は snake_case、共通 fixture は `conftest.py` に配置。

例:

```python
def test_fetch_user_returns_user():
    assert fetch_user("u1") is not None
```

---

## ロギング / フィールド名

- ログのキーは一貫した snake_case（例: `user_id`, `request_id`）。

例:

```python
logger.info("user created", extra={"user_id": user.id})
```

## 環境変数 / 設定

- 環境変数は UPPER_SNAKE_CASE（例: `APP_PORT`, `DATABASE_URL`）。

## 自動生成コードの扱い（プロトコル / OpenAPI）

- 生成コードは直接編集せず、アダプター層で内部ドメイン型に変換する。生成物は差異を最小限にし、変換ロジックを明確に保つ。

---

## アンチパターン

- 命名規則の混在（snake_case と camelCase）。
- 大きな public API に型ヒントを付けないこと。
- 生成コードを手で修正すること。

---

## 望ましい linter / 型チェック設定（例）

以下はプロジェクトで型品質を保つための推奨設定例です。実際の設定はチームの Python バージョンや要件に合わせて調整してください。

- TypedDict のような PEP 655（NotRequired）を利用する場合、古い Python では `typing_extensions` を使う必要があります。
- 型チェックは `mypy`（または `pyright`）で行い、`ruff` や `flake8` はスタイルと簡易な検査に使うワークフローを推奨します。

例: `pyproject.toml` に ruff 設定を置く（抜粋）

```toml
[tool.ruff]
line-length = 88
select = ["E", "F", "W", "C", "B", "I", "N"]
exclude = [".venv", "build", "dist"]

[tool.ruff.per-file-ignores]
"tests/*" = ["D", "S101"]
```

例: `mypy.ini`（または `pyproject.toml` の `[tool.mypy]`）

```ini
[mypy]
python_version = 3.10
ignore_missing_imports = True
strict_optional = True
disallow_untyped_defs = True
warn_unused_ignores = True
plugins =

```

TypedDict の扱いの注意:

- `TypedDict` を定義する際はスキーマ的な用途で使い、ランタイム検証が必要な場合は `pydantic` や `attrs` などを併用すると安全です。
- PEP 655 の `NotRequired` を利用する場合、ユーザーの Python バージョンでネイティブにサポートされていないときは `typing_extensions.NotRequired` をインポートしてください。

例:

```python
from typing_extensions import NotRequired, TypedDict

class UserDict(TypedDict):
    id: str
    name: NotRequired[str]
```

推奨ワークフロー:

1. CI で `ruff` を先に実行してスタイル問題を早期に検出。
2. 続けて `mypy`（または `pyright`）で型チェック。
3. 重要な API には `mypy` の strict 設定を段階的に導入する（`disallow_untyped_defs` 等）。

## 参考リンク

- PEP8: https://peps.python.org/pep-0008/
- PEP257 (Docstring): https://peps.python.org/pep-0257/
- typing: https://docs.python.org/3/library/typing.html

---