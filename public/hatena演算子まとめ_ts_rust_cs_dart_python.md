---
title: 「?」演算子まとめ（TypeScript,C#,Dart,Rust,Python）
tags:
  - 'TypeScript'
  - 'C#'
  - 'Dart'
  - 'Rust'
  - 'Python'
private: false
updated_at: ''
id: null
organization_url_name: null
slide: false
ignorePublish: false
---

記号`?`は、プログラミング言語ごとに多様な役割で使われます。
複数の言語を扱っていると混乱するので、表にまとめました。
表題の通り、本記事では、TypeScript, C#, Dart, Rust, Pythonについて記載します。

| 役割                                                 | プログラミング言語   | 例                                                                       |
| ---------------------------------------------------- | -------------------- | ------------------------------------------------------------------------ |
| **三項演算子（? :）**                                | TypeScript, C#, Dart | `let isAdult = age >= 18 ? "Yes" : "No";` (TypeScript)                   |
|                                                      | （Rust, Python）     | `let is_adult = if age >= 18 { "Yes" } else { "No" };` (Rust)            |
| **オプショナルチェイニング（null条件演算子）（.?）** | TypeScript, C#, Dart | `let user1Name = user1?.name;` (TypeScript)                              |
|                                                      | （Rust, Python）     | `user1_name = getattr(user1, 'name', None)` (Python)                     |
| **null合体演算子（??）**                             | TypeScript, C#, Dart | `let name = userName ?? "Guest";` (TypeScript)                           |
|                                                      | （Rust, Python）     | `let default_name = name.unwrap_or_else(\|\| "Guest".to_string());` (Rust) |
| **エラー・None委譲（?）**                            | Rust                 | `let content = std::fs::read_to_string(file_path)?;`                     |


## 各役割の説明

### 三項演算子

#### TypeScript
```typescript
let age = 18;
let isAdult = age >= 18 ? "Yes" : "No";
console.log(isAdult); // Output: Yes
```

#### C#
```csharp
int age = 18;
string isAdult = age >= 18 ? "Yes" : "No";
Console.WriteLine(isAdult); // Output: Yes
```

#### Dart
```dart
int age = 18;
String isAdult = age >= 18 ? "Yes" : "No";
print(isAdult); // Output: Yes
```

#### （Rust）
Rustには直接の三項演算子はありませんが、`if`文を使って同じことができます。
```rust
let age = 18;
let is_adult = if age >= 18 { "Yes" } else { "No" };
println!("{}", is_adult); // Output: Yes
```

#### （Python）
Pythonには三項演算子はありませんが、`if`文を使って同じことができます。
```python
age = 18
is_adult = "Yes" if age >= 18 else "No"
print(is_adult) # Output: Yes
```

### オプショナルチェイニング（null条件演算子）
オプショナルチェイニング演算子`?.`は、オブジェクトのプロパティやメソッドにアクセスする際に、オブジェクトの存在（`null`値や`undefined`値）チェックを行い、存在しなければ`null`や`undefined`を返します。

#### TypeScript

```typescript
let user1 = {
    name: "Alice"
};
let user2 = undefined
let user1Name = user1?.name; // "Alice"
let user2Name = user2?.name; // undefined
```

#### C#
C#では、`?.`はnull条件演算子と呼ばれますが、他のオプショナルチェイニングと同様です。

```csharp
class User
{
    public string Name { get; set; }
}

class Program
{
    static void Main()
    {
        User user1 = new User { Name = "Alice" };
        User user2 = null;

        string user1Name = user1?.Name; // "Alice"
        string user2Name = user2?.Name; // null
    }
}
```

#### Dart
```dart
class User {
  String? name;

  User(this.name);
}

void main() {
  var user1 = User("Alice");
  User? user2;

  String? user1Name = user1.name; // "Alice"
  String? user2Name = user2?.name; // null
}

```

#### （Rust）
Rustにはオプショナルチェイニング演算子の直接的なサポートはありません。
Option型と`and_then`関数を使用して同様の機能を実現できます。

```rust
fn main() {
    struct User {
        name: Option<String>,
    }

    let user1: Option<User> = Some(User {
        name: Some("Alice".to_string()),
    });
    let user2: Option<User> = None;

    let user1_name = user1.as_ref().and_then(|u| u.name.as_deref()); // Some("Alice")
    let user2_name = user2.as_ref().and_then(|u| u.name.as_deref()); // None
}
```

#### （Python）
Pythonにはオプショナルチェイニング演算子の直接的なサポートはありません。
`getattr`関数を使って同様の機能を実現できます。

```python
class User:
    def __init__(self, name=None):
        self.name = name

user1 = User(name="Alice")
user2 = None

user1_name = getattr(user1, 'name', None)  # "Alice"
user2_name = getattr(user2, 'name', None)  # None
```

### null合体演算子
null合体演算子`??`は、`null`や`undefined`の可能性がある値に対して、デフォルト値を設定します。

#### TypeScript

```typescript
let name = null;
let defaultName = name ?? "Guest"; // "Guest"（nameがnullなので）
```

#### C#

```csharp
string name = null;
string defaultName = name ?? "Guest"; // "Guest"（nameがnullなので）
```

#### Dart

```dart
String? name = null;
String defaultName = name ?? "Guest"; // "Guest"（nameがnullなので）
```

#### Rust
Rustではnull合体演算子が直接サポートされていません。
Option型と`unwrap_or_else`関数を使って、同等の機能を実現できます。

```rust
let name: Option<String> = None;
let default_name = name.unwrap_or_else(|| "Guest".to_string()); // "Guest"（nameがNoneなので）
```

#### Python
Pythonではnull合体演算子が直接サポートされていません。
`or`演算子で同等の機能を実現できます。

```python
name = None
default_name = name or "Guest"  # "Guest"（nameがNoneなので）
```

### エラー・None委譲
#### Rust
Rustの`?`演算子は、`Result<T, E>`型のエラーや`Option<T>`型の`None`を、関数の呼び出し元に委譲します。

**`Result`型の使用例**

```rust
fn read_file(file_path: &str) -> Result<String, std::io::Error> {
    let content = std::fs::read_to_string(file_path)?;
    Ok(content)
}
```

`std::fs::read_to_string`関数が`Result<String, std::io::Error>`を返します。`?`演算子を使うことで、エラーが発生した場合は即座に`read_file`関数の呼び出し元へエラーを返します。

**`Option`型の使用例**

```rust
fn get_length(s: Option<&str>) -> Option<usize> {
    let s = s?;
    Some(s.len())
}
```
`Option`型の変数`s`に`?`演算子を使うことで、値が`None`の場合は即座に`get_length`関数の呼び出し元へ`None`を返します。
