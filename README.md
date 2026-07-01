# Rutile プログラミング言語

**Rutile** は Ruby と Lua に影響を受けた、動的型付けのインタプリタ型スクリプト言語です。
C++17 で実装されたシングルファイルインタプリタで、C拡張による機能拡張をサポートします。

---

## 目次

1. [ビルドと実行](#ビルドと実行)
2. [変数と型](#変数と型)
3. [演算子](#演算子)
4. [文字列](#文字列)
5. [配列](#配列)
6. [Map](#map)
7. [制御フロー](#制御フロー)
8. [関数](#関数)
9. [クラスとオブジェクト指向](#クラスとオブジェクト指向)
10. [型注釈](#型注釈)
11. [例外処理](#例外処理)
12. [eval](#eval)
13. [標準メソッド](#標準メソッド)
14. [標準ライブラリ](#標準ライブラリ)
15. [import](#import)
16. [C拡張の作り方](#c拡張の作り方)
17. [コマンドラインオプション](#コマンドラインオプション)
18. [既知の制限](#既知の制限)

---

## ビルドと実行

### 必要なもの
- C++17 対応コンパイラ (g++ 8+, clang++ 7+, MSVC 2019+)
- Linux / macOS / Windows

### ビルド

```bash
# インタプリタ本体
g++ -std=c++17 -O2 rutile.cpp -o rutile -ldl

# 標準ライブラリ拡張
make libs           # または個別に:
g++ -std=c++17 -O2 -fPIC -shared io.cpp     -o io.so
g++ -std=c++17 -O2 -fPIC -shared math.cpp   -o math.so
g++ -std=c++17 -O2 -fPIC -shared http.cpp   -o http.so
g++ -std=c++17 -O2 -fPIC -shared narashino.cpp -o narashino.so
```

Windows(MinGW)では `.so` を `.dll` に変えてください。

### 実行

```bash
rutile script.rut              # 通常実行
rutile -warning script.rut    # 型不一致などを事前に警告
rutile -jit script.rut        # 整数演算関数をJITコンパイル(Linux/Windows x64)
rutile script.rut arg1 arg2   # コマンドライン引数を渡す
```

### 拡張ライブラリの探索パス
1. スクリプトと同じディレクトリ
2. カレントディレクトリ
3. `rutile` 実行ファイルと同じディレクトリ ← **ここに .so を置くと自動で見つかる**
4. 環境変数 `RUTILE_LIB_PATH` (コロン区切り)
5. `/usr/local/lib/rutile/`

---

## 変数と型

```rutile
let x = 42;           // Integer (int64_t)
let y = 3.14;         // Double  (double)
let s = "hello";      // String
let b = true;         // Bool
let n = null;         // Null
let a = [1, 2, 3];    // Array
let m = @{"key": 1};  // Map
```

### 変数修飾子

| 書き方 | 意味 |
|--------|------|
| `let x = v;` | ローカル変数 (再代入可) |
| `const let X = v;` | ローカル定数 (再代入エラー) |
| `global let x = v;` | グローバル変数 |
| `static let x = v;` | 静的変数 (初回のみ初期化) |
| `global const let X = v;` | グローバル定数 (組み合わせ可) |

### 型変換

| `to_i` | `to_f` | `to_s` | `type()` |
|--------|--------|--------|---------|
| 整数に変換 (失敗で0) | 浮動小数点に変換 | 文字列に変換 | 型名を返す |

---

## 演算子

```rutile
// 算術
x + y;  x - y;  x * y;  x / y;  x % y;

// 比較 (型が違っても暗黙の型変換で比較する)
x == y;  x != y;  x < y;  x > y;  x <= y;  x >= y;

// 論理
x && y;  x || y;

// ビット演算
x & y;  x | y;  x ^ y;  ~x;  x << n;  x >> n;

// 代入
x = v;  x += v;  x -= v;  x++;  x--;

// スワップ
x <=> y;

// 数値リテラル
0xFF;    // 16進数 → 255
0b1010;  // 2進数  → 10
0o17;    // 8進数  → 15
```

### 真偽値
`null` と `false` → **偽**、それ以外すべて → **真**

---

## 文字列

```rutile
let s1 = "改行\n タブ\t #{"埋め込み式"} が使える";
let s2 = 'シングルクォートは#{補間なし}の生文字列';

let name = "Rutile";
let msg = "Hello, #{name}!";   // => "Hello, Rutile!"
```

### 文字列メソッド (括弧省略可)

```rutile
"Hello".length          // => 5
"Hello".upcase          // => "HELLO"
"HELLO".downcase        // => "hello"
"hello world".capitalize// => "Hello world"
"  hi  ".trim           // => "hi"
"hello".reverse         // => "olleh"
"hello".first           // => "h"
"hello".last            // => "o"
"a,b,c".split(",")      // => ["a","b","c"]
"foo bar".gsub("foo","baz")   // => "baz bar"
"hello".include?("ell") // => true
"hello".index("ll")     // => 2
"42abc".to_i            // => 42
"3.14".to_f             // => 3.14
"hello\n".chomp         // => "hello"
"hello".to_s            // => "hello"
"hello".type()          // => "String"
```

---

## 配列

```rutile
let arr = [1, "two", 3.0, true, null];  // 型混在OK
let nested = [[1,2], [3,4]];            // 多次元

arr[0];       // => 1
arr[-1];      // => null  (末尾から)
arr[2][1];    // 多次元アクセス
```

### 配列メソッド

```rutile
arr.length            // 要素数
arr.first             // 最初の要素
arr.last              // 最後の要素
arr.sum               // 数値要素の合計
arr.sort              // ソート (数値のみ/文字列のみ)
arr.reverse           // 逆順
arr.uniq              // 重複削除
arr.empty?            // 空かどうか
arr.clear()           // 全要素削除
arr.push(v)           // 末尾に追加
arr.pop()             // 末尾から取り出す
arr.shift             // 先頭から取り出す
arr.unshift(v)        // 先頭に追加
arr.include?(v)       // 含むか
arr.index(v)          // 最初に見つかった位置
arr.delete(v)         // 値で削除
arr.delete_at(n)      // インデックスで削除
arr.join(",")         // 区切り文字で結合
arr.each(v){ ... }    // 各要素に処理
arr.map(v){ v*2 }     // 新しい配列を作る
arr.map!(v){ v*2 }    // 元の配列を変更
```

---

## Map

```rutile
let m = @{
    "name": "Taro",
    42: "integer key",    // 文字列以外のキーもOK
    true: "bool key",
    null: "null key"
};

m["name"];          // => "Taro"
m[42];              // => "integer key"
m["missing"];       // => null (エラーにならない)
m["new"] = "val";   // 動的追加
```

### Mapメソッド

```rutile
m.length            // エントリ数
m.empty?            // 空かどうか
m.keys()            // キー一覧 (Array)
m.values()          // 値一覧 (Array)
m.include?("name")  // キーの存在確認
m.clear()           // 全削除
```

---

## 制御フロー

### if式/文

```rutile
// 式 (値を返す)
let x = if (n > 0) { "positive" } else if (n < 0) { "negative" } else { "zero" };

// 文
if (cond) io.println("one line");  // 波括弧省略(1文のみ)
if (cond) { ... } else { ... }

// if は式なので文字列補間の中でも使える
io.println("x is #{if (x > 0) { "positive" } else { "negative" }}");
```

### while

```rutile
while (i < 10) i++;                // 括弧省略(1文)
while (cond) { ... }               // 通常
```

### for

```rutile
// 範囲ループ (終端値を含む・逆順もOK)
for (i in 1..10) { io.println(i); }
for (i in 10..1) { io.println(i); }  // 逆順

// 配列のforEach
for (idx, val in arr) { io.println("#{idx}: #{val}"); }

break;      // ループを抜ける
continue;   // 次のイテレーションへ
```

---

## 関数

```rutile
// 関数リテラル (letで名前をつける)
let add = fn(a, b) { a + b };         // ブロック
let sq  = fn(x) x * x;               // 単一式(波括弧省略)

// 戻り値: 最後の式の値 (セミコロン省略) か return
let fact = fn(n) {
    if (n <= 1) { return 1; }
    n * fact(n - 1)     // セミコロンなし = 戻り値
};

// 関数は値なのでコピーできる
let f = add;
f(3, 4);  // => 7

// グローバル関数・定数関数
global let greet = fn(name) { "Hello, #{name}!" };
const let PI_APPROX = fn() { 3.14159 };

// クロージャ
let make_counter = fn() {
    let n = 0;
    fn() { n = n + 1; n }
};
let c = make_counter();
c();  // => 1
c();  // => 2
```

---

## クラスとオブジェクト指向

```rutile
// クラス定義: let クラス名 = class() { ... };
let Animal = class() {
    let name = "noname";    // インスタンスフィールド (デフォルト値)
    let sound = "...";
    static let count = 0;  // クラス変数

    fn init(name) {         // コンストラクタ
        this.name = name;
        Animal.count = Animal.count + 1;
    }

    fn speak() {
        "#{this.name} says #{this.sound}!"
    }
};

// 継承: let 子クラス:親クラス = class() { ... };
let Dog:Animal = class() {
    fn init(name) {
        super.init(name);   // 親のコンストラクタを呼ぶ
        this.sound = "Woof";
    }
    fn fetch() { "#{this.name} fetches!" }
};

// インスタンス生成: ClassName.new(args)
let d = Dog.new("Rex");
d.speak();              // => "Rex says Woof!"
d.fetch();              // => "Rex fetches!"
d.tag = "good boy";     // フィールド動的追加もOK

// クラスメソッド/変数へのアクセス
Dog.count;              // => 1 (Animalから継承されたstatic)
Animal.count;           // => 1

// 型チェック
d.type();               // => "Dog"
d.class.name;           // => "Dog"
d.is_a?(Animal);        // => true
d.is_a?(Dog);           // => true

// static メソッド
let MathHelper = class() {
    static let PI = 3.14159;
    static fn square(x) { x * x }
};
MathHelper.PI;          // => 3.14159
MathHelper.square(5);   // => 25
```

---

## 型注釈

`let 変数名:型名 = 値;` の形で型を明示できます。

```rutile
let n: Integer = "42";   // => 42  (String → Integer に暗黙変換)
let s: String  = 100;    // => "100"
let f: Double  = "3.14"; // => 3.14

// クラス名を指定すると継承チェックになる
let pet: Animal = Dog.new("Rex");   // OK (DogはAnimalのサブクラス)
let pet: Animal = "not a pet";      // エラー

// -warning オプションで暗黙変換を警告表示
// rutile -warning script.rut
```

### 組み込み型名
`Integer`, `Double`, `String`, `Bool`, `Array`, `Map`, `Function`, `Null`

---

## 例外処理

```rutile
try {
    let arr = [1, 2, 3];
    arr[100];       // 範囲外参照 → エラー
} catch (e) {
    io.println("Error: #{e}");
}

// 1行の場合
try io.println(1/0) catch (e) io.println("caught: #{e}");

// 独自エラーを投げる
let check = fn(n) {
    if (n < 0) { throw RutileError("負の値は不正です: #{n}"); }
    n * 2
};
```

---

## eval

```rutile
eval("1 + 2 * 3");   // => 7

// 現在のスコープで実行される
let x = 10;
eval("x = x * 2;");
x;  // => 20
```

---

## 標準メソッド

ライブラリを `import` しなくても使えます。メソッドは括弧省略可 (引数なしのNative関数のみ)。

### 数値 (Integer / Double)

```rutile
(-5).abs          // => 5
4.even?           // => true
7.odd?            // => true
3.14159.round(2)  // => 3.14
3.7.round()       // => 4

5.times(i) { io.println(i); }   // 0..4 を繰り返す
3.times() { io.println("!"); }  // 変数名省略可

(-5).to_i         // => -5
3.7.to_f          // => 3.7
42.to_s           // => "42"
42.type()         // => "Integer"
```

### 文字列

*(前述の文字列メソッド参照)*

### 共通

```rutile
val.to_i        // Integer に変換
val.to_f        // Double に変換
val.to_s        // String に変換
val.type()      // 型名を返す
```

---

## 標準ライブラリ

### io

```rutile
import "io";

io.println(value);               // 標準出力に出力(改行あり)
io.print(value);                 // 標準出力に出力(改行なし)
let line = io.input();           // 標準入力から1行読む(改行含む)
let content = io.open("file.txt"); // ファイル全体を読む
io.write("out.txt", "text");     // ファイルに書き込む(上書き)
io.append("log.txt", "line\n");  // ファイルに追記
io.err("error message");         // 標準エラー出力に出力
```

### math

```rutile
import "math";

math.pow(2, 10)    // => 1024
math.sqrt(16)      // => 4.0
math.abs(-5)       // => 5
math.floor(3.9)    // => 3
math.ceil(3.1)     // => 4
math.round(3.5)    // => 4
math.max(3, 7)     // => 7
math.min(3, 7)     // => 3
math.PI            // => 3.14159265358979...
```

### http

```rutile
import "http";

// ルート登録
http.get("/", fn(req) {
    http.respond(200, "text/html", "<h1>Hello!</h1>")
});

http.post("/api/data", fn(req) {
    io.println("body: #{req["body"]}");
    http.respond_json(200, "{\"ok\": true}")
});

// サーバー起動
http.listen(8080);       // ブロッキング (Ctrl+C で停止)
http.listen(8080, 5);    // 5リクエスト処理して終了

// レスポンスヘルパー
http.respond(200, "text/html", "<p>Hello</p>");
http.respond_json(200, "{\"key\": \"value\"}");
http.respond_redirect("https://example.com");

// クエリ文字列をMapに変換
let params = http.parse_query(req["query"]);

// req (リクエストMap) のフィールド
// req["method"]  : "GET" / "POST" 等
// req["path"]    : "/api/hello"
// req["query"]   : "name=taro&age=30"
// req["body"]    : POSTボディ
// req["headers"] : ヘッダMap (キーは小文字)
// req["remote"]  : クライアントIPアドレス
```

### narashino (サンプル拡張)

```rutile
import "narashino";

narashino.population();  // => 176383
narashino.household();   // => 84830
```

---

## import

```rutile
import "io"          // C拡張 (io.so / io.dll)
import "math"        // C拡張 (math.so / math.dll)
import "utils.rut"   // 別の .rut ファイル

// .rut を import すると、そのファイルの変数・関数が
// ファイル名(拡張子なし).名前 で使える
// utils.rut に let add = fn(a,b){a+b}; があれば:
utils.add(1, 2);  // => 3
```

---

## C拡張の作り方

`extension_template.cpp` をコピーして使ってください。

```cpp
#include "rutile_api.h"
using namespace rutile;

namespace {
Value my_hello(std::vector<Value>& args) {
    if (args.size() != 1) throw RutileError("hello(name) : 引数を1つ取ります");
    return Value::Str("Hello, " + asString(args[0]) + "!");
}
}

RUTILE_EXTENSION_ENTRYPOINT {
    __rutile_mod.addFunction("hello", my_hello);
    __rutile_mod.addValue("VERSION", Value::Str("1.0"));
}
```

### ビルドして使う

```bash
g++ -std=c++17 -O2 -fPIC -shared mylib.cpp -o mylib.so
rutile script.rut  # script.rut と同じディレクトリに mylib.so を置く
```

### rutile_api.h の主な型とヘルパー

```cpp
Value::Null()           // null値
Value::Bool(true)       // bool値
Value::Int(42)          // Integer値
Value::Double(3.14)     // Double値
Value::Str("hello")     // String値
Value::Arr({v1, v2})    // Array値
Value::MapV(entries)    // Map値

asNumber(v)             // 数値に変換 (Int/Double → double)
asString(v)             // 文字列に変換 (String以外はエラー)
isNumber(v)             // 数値型かどうか
isValidMapKeyType(v)    // Mapのキーとして使える型か
isCallable(v)           // 呼び出し可能か

mapFind(entries, key)   // Mapからキーで検索 (Value* を返す)
mapSet(entries, key, v) // Mapにキーと値をセット

// 拡張からRutile関数を呼び出す (http.cppのように)
callRutileFunction(handler, args);
```

---

## コマンドラインオプション

```
rutile [オプション] <スクリプト.rut> [引数...]

オプション:
  -warning   実行前に型不一致などを静的解析して警告表示する
  -jit       整数演算・自己再帰関数をx64機械語にJITコンパイルする
             (Linux / Windows x64 のみ対応)

引数は ARGV 配列で取得できる:
  let first = ARGV[0];
```

---

## 既知の制限

| 制限事項 | 補足 |
|---------|------|
| ユーザー定義メソッドの括弧省略不可 | `obj.method()` のように`()`必須。`let f = obj.method;` で関数コピー可 |
| JITは整数演算+自己再帰のみ | Float/配列/OOP等を使う関数はツリーウォーク実行にフォールバック |
| httpは シングルスレッド | 高負荷には不向き |
| -warning の静的解析はリテラル同士のみ | 変数を介した型不一致は実行時に暗黙変換される |
| 文字列の正規表現はECMAScript構文 | `std::regex` を使用 |
| UTF-8は表示・長さ計算のみ対応 | `.split` / `.gsub` 等はバイト単位 |

---

## ライセンス

MIT License

---

*Rutile - Ruby と Lua に影響を受けた、シンプルで拡張可能なスクリプト言語*
