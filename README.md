# Rutile プログラミング言語

**Rutile** は Ruby/Lua に影響を受けた、動的型付けのインタプリタ型スクリプト言語です。  
拡張子は `.rt`。C++17 単一ファイル実装、C 拡張、x64 JIT を搭載します。

---

## ビルド方法

### Linux / macOS

```bash
# インタプリタ本体
g++ -std=c++17 -O2 -Wl,-export-dynamic rutile.cpp -o rutile -ldl

# 標準ライブラリ拡張 (全部まとめて)
make libs

# 個別ビルド
g++ -std=c++17 -O2 -fPIC -shared io.cpp      -o io.so
g++ -std=c++17 -O2 -fPIC -shared math.cpp    -o math.so
g++ -std=c++17 -O2 -fPIC -shared http.cpp    -o http.so
g++ -std=c++17 -O2 -fPIC -shared async.cpp   -o async.so  -lpthread
g++ -std=c++17 -O2 -fPIC -shared zip.cpp     -o zip.so    -lz
g++ -std=c++17 -O2 -fPIC -shared sqlite.cpp  -o sqlite.so -ldl
g++ -std=c++17 -O2 -fPIC -shared narashino.cpp -o narashino.so
```

### Windows — MinGW (推奨)

```bash
# インタプリタ本体
g++ -std=c++17 -O2 rutile.cpp -o rutile.exe

# 拡張ライブラリ
g++ -std=c++17 -O2 -shared io.cpp       -o io.dll
g++ -std=c++17 -O2 -shared math.cpp     -o math.dll
g++ -std=c++17 -O2 -shared http.cpp     -o http.dll  -lws2_32
g++ -std=c++17 -O2 -shared async.cpp    -o async.dll
g++ -std=c++17 -O2 -shared zip.cpp      -o zip.dll   -lz
g++ -std=c++17 -O2 -shared sqlite.cpp   -o sqlite.dll
g++ -std=c++17 -O2 -shared narashino.cpp -o narashino.dll
```

### Windows — MSVC (Visual Studio 2019/2022)

> スタートメニューから **「Developer Command Prompt for VS」** を開いて実行してください。

```bat
REM インタプリタ本体
cl /std:c++17 /O2 /EHsc rutile.cpp /Fe:rutile.exe /link

REM 標準ライブラリ (拡張DLL)
cl /std:c++17 /O2 /EHsc /LD io.cpp       /Fe:io.dll
cl /std:c++17 /O2 /EHsc /LD math.cpp     /Fe:math.dll
cl /std:c++17 /O2 /EHsc /LD http.cpp     /Fe:http.dll   /link ws2_32.lib
cl /std:c++17 /O2 /EHsc /LD async.cpp    /Fe:async.dll
cl /std:c++17 /O2 /EHsc /LD zip.cpp      /Fe:zip.dll    /link zlib.lib
cl /std:c++17 -O2 /EHsc /LD sqlite.cpp   /Fe:sqlite.dll
cl /std=c++17 /O2 /EHsc /LD narashino.cpp /Fe:narashino.dll
```

> **注意 (MSVC)**: `zlib` は [zlib.net](https://zlib.net) から入手するか、  
> `vcpkg install zlib` でインストールしてください。  
> SQLite は libsqlite3.so.0 を実行時に動的ロードするため、ビルド時依存なし。

### C拡張ヘッダの取得

```bash
rutile -api   # rutile_api.h をカレントディレクトリへコピー
```

---

## 実行

```bash
rutile script.rt              # 通常実行
rutile -warning script.rt    # 型不一致などを事前に警告
rutile -jit script.rt        # 整数演算関数を x64 JIT (Linux/Windows x64)
rutile -api                    # rutile_api.h をカレントディレクトリへコピー
rutile script.rt arg1 arg2   # コマンドライン引数 → ARGV 配列
```

### ライブラリ探索パス (優先順)
1. スクリプトと同じディレクトリ
2. カレントディレクトリ
3. rutile 実行ファイルと同じディレクトリ
4. `<各Dir>/ore/<名前>/` — 整理用サブフォルダ
5. `<各Dir>/../ore/<名前>/` — exe が src/ 等のサブディレクトリにある場合  
   例: exe が `rutile/src/` → `rutile/ore/io/io.so` も探す
6. 環境変数 `RUTILE_LIB_PATH` (`:` 区切り)
7. `/usr/local/lib/rutile/`

---

## 変数と型

```rutile
let x = 42;        // Integer (int64_t)
let y = 3.14;      // Double
let s = "hello";   // String
let b = true;      // Bool
let n = null;      // Null
let a = [1,2,3];   // Array
let m = map(){"k":1};  // Map
```

### 変数修飾子 (組み合わせ可)

| 書き方 | 意味 |
|---|---|
| `let x = v;` | ローカル変数 |
| `const let X = v;` | ローカル定数 |
| `global let x = v;` | グローバル変数 |
| `static let x = v;` | 静的変数 (初回のみ初期化) |
| `global const let X = v;` | グローバル定数 |

### 型注釈

```rutile
let n:Integer = "42abc";  // → 42 (暗黙変換)
let s:String  = 100;      // → "100"
// -warning オプションで暗黙変換時に警告表示
```

---

## 演算子

```rutile
+  -  *  /  %          // 算術
==  !=  <  >  <=  >=   // 比較 (型不一致でも暗黙変換して比較)
&&  ||                  // 論理
&  |  ^  ~  <<  >>     // ビット演算
=  +=  -=  x++  x--    // 代入
a <=> b                 // スワップ
0xFF  0b1010  0o17      // 数値リテラル (16進/2進/8進)
```

---

## 文字列

```rutile
let name = "Rutile";
let msg  = "Hello, #{name}!\n次の行";  // 補間 + エスケープ
let raw  = '補間なし #{text}';          // シングルクォート

s.length   s.upcase    s.downcase  s.capitalize
s.trim     s.reverse   s.chomp
s.first    s.last
s.include?("word")
s.index("word")
s.split(",")
s.gsub("foo","bar")
s.to_i   s.to_f   s.to_s   s.type()
```

---

## 配列

```rutile
let arr = [1, "two", 3.0];   // 型混在OK
arr[0]   // 先頭
arr[-1]  // 末尾 (-1=末尾, -2=末尾から2番目, ...)

// 多次元配列: [[],[]] 形式
let grid = [[1,2,3],[4,5,6],[7,8,9]];
grid[1][2];   // → 6

arr.length   arr.first    arr.last
arr.sum      arr.sort     arr.reverse
arr.uniq     arr.empty?   arr.clear()
arr.push(v)  arr.pop()
arr.shift    arr.unshift(v)
arr.include?(v)  arr.index(v)
arr.delete(v)    arr.delete_at(n)
arr.join(",")
arr.each(v){ ... }
arr.map(v){ v*2 }
arr.map!(v){ v*2 }  // 破壊的変更
```

---

## Map

```rutile
// 新構文 (推奨)
let m = map() {
    "name": "Taro",
    42:     "整数キー",    // Integer/Double/Bool/Null もキーに使える
    true:   "真偽キー"
};

// 旧構文 @{} (後方互換)
let m2 = @{"x": 10, "y": 20};

m["name"];          // 取得 (存在しなければ null)
m["new"] = "追加"; // 動的追加

m.length()   m.empty?   m.keys()   m.values()
m.include?("name")  m.clear()
```

---

## 制御フロー

```rutile
// if (式にも文にもなる)
let x = if (n > 0) { "正" } else { "負以下" };
if (cond) io.println("1行");        // 波括弧省略
if (cond) { ... } else if (cond) { ... } else { ... }

// while
while (i < 10) i++;
while (cond) { ... }

// for
for (i in 1..10) { io.println(i); }  // 範囲 (終端含む・逆順可)
for (idx, val in arr) { ... }         // forEach

break;   continue;
```

---

## 関数

```rutile
let add  = fn(a, b) { a + b };   // ブロック形式
let sq   = fn(x) x * x;          // 単一式 (波括弧省略)

// 戻り値: 最後の式 or return
let fact = fn(n) {
    if (n <= 1) { return 1; }
    n * fact(n - 1)
};

let f = add;       // 関数コピー
f(3, 4);           // → 7

// クロージャ
let make_counter = fn() {
    let n = 0;
    fn() { n = n + 1; n }
};
```

---

## クラスとOOP

```rutile
// クラス定義: let 名前 = class() { ... };
// メソッドは let name = fn(args){...}; スタイル
let Animal = class() {
    let name  = "noname";
    let sound = "...";
    static let count = 0;      // クラス変数

    let init = fn(n) {
        this.name = n;
        Animal.count = Animal.count + 1;
    };
    let speak = fn() { "#{this.name} says #{this.sound}!" };
};

// 継承: let 子:親 = class() { ... };
let Dog:Animal = class() {
    let init = fn(n) {
        super.init(n);         // 親コンストラクタ
        this.sound = "Woof";
    };
    let fetch = fn() { "#{this.name} fetches!" };
};

// インスタンス生成
let d = Dog.new("Rex");
d.speak();                 // → "Rex says Woof!"
d.tag = "good boy";       // フィールド動的追加
d.type();                  // → "Dog"
d.class.name;              // → "Dog"
d.is_a?(Animal);           // → true

// static メソッド
let Util = class() {
    static let PI = 3.14159;
    static let square = fn(x) { x * x };
};
Util.square(5);   // → 25
```

---

## 例外処理

```rutile
try {
    let arr = [1,2,3];
    arr[100];           // 範囲外 → エラー
} catch (e) {
    io.println("エラー: #{e}");
}
try risky() catch (e) io.println(e);  // 1行形式
```

---

## eval

```rutile
eval("1 + 2 * 3");    // → 7
let x = 10;
eval("x = x * 2;");
x;                     // → 20
```

---

## 標準メソッド (import 不要)

```rutile
// 数値
(-5).abs      // → 5
4.even?       // → true
7.odd?        // → true
3.14159.round(2)  // → 3.14
5.times(i) { io.println(i); }
3.times() { io.println("!"); }

// 全型共通
val.to_i   val.to_f   val.to_s   val.type()
```

---

## 標準ライブラリ

### io

```rutile
import "io";
io.println(value);            // 出力(改行あり)
io.print(value);              // 出力(改行なし)
let s = io.input();           // 1行入力(改行含む)
let c = io.open("file.txt");  // ファイル全読み
io.write("out.txt", "text");  // 上書き書き込み
io.append("log.txt", "line"); // 追記
io.err("error msg");          // 標準エラー出力
```

### math

```rutile
import "math";
math.pow(2,10)   math.sqrt(16)   math.abs(-5)
math.floor(3.9)  math.ceil(3.1)  math.round(3.5)
math.max(3,7)    math.min(3,7)
math.PI   // 3.14159265358979...
```

### http

```rutile
import "http";

http.get("/", fn(req) {
    http.respond(200, "text/html", "<h1>Hello!</h1>")
});
http.post("/api", fn(req) {
    http.respond_json(200, "{\"ok\":true}")
});
http.listen(8080);         // ブロッキング起動
http.listen(8080, 5);      // 5リクエストで終了(テスト用)
http.respond_redirect("/");

// req フィールド: "method" "path" "query" "body" "headers" "remote"
let params = http.parse_query(req["query"]);
```

### async (非同期)

```rutile
import "async";

async.sleep(1000);          // スリープ(ms)
let t = async.now();        // 現在時刻(ms)

// バックグラウンド実行 + 待機
let f = async.run(fn() { async.sleep(200); 42 });
let r = async.await(f);              // → 42
async.is_done(f);                    // → true

// 複数並列実行
let f1 = async.run(fn() { "A" });
let f2 = async.run(fn() { "B" });
let results = async.await_all([f1, f2]);

// 一括並列
let res = async.parallel([fn(){1}, fn(){2}, fn(){3}]);

// タイムアウト
let r = async.timeout(500, fn() { async.sleep(1000); "ok" }); // → null

// 定期実行
async.interval(5, 200, fn(i) { io.println("tick #{i}") });

// チャンネル
let ch = async.channel();
async.send(ch, "hello");
let msg = async.recv(ch);           // ブロッキング
let msg2 = async.recv_nowait(ch);   // ノンブロッキング (空→null)
async.close(ch);

// 計測
let m = async.measure(fn() { heavy() });
io.println("#{m["elapsed_ms"]}ms");
async.thread_count();   // スレッドプールのスレッド数
```

### zip (圧縮・アーカイブ)

```rutile
import "zip";

// gzip
let c = zip.gzip(data);
let d = zip.gunzip(c);
zip.gzip_file("in.txt", "out.gz");
zip.gunzip_file("out.gz", "restored.txt");

// deflate/inflate
let d2 = zip.deflate(data);
let r2 = zip.inflate(d2);

// ZIPアーカイブ作成
let arc = zip.create();
zip.add(arc, "hello.txt", "Hello!");  // 文字列データ
zip.add_file(arc, "src.rt");          // ファイルを追加
zip.save(arc, "archive.zip");
zip.close(arc);

// ZIPアーカイブ読み込み
let arc2 = zip.open("archive.zip");
let files = zip.list(arc2);           // → ["hello.txt", "src.rt"]
let content = zip.read(arc2, "hello.txt");
zip.extract_all(arc2, "./output/");
zip.close(arc2);

zip.compress_ratio(orig_size, comp_size);  // 圧縮率(%)
zip.size(data);                            // バイト数
```

### sqlite

```rutile
import "sqlite";

// ※ libsqlite3.so.0 (Linux) / sqlite3.dll (Windows) が実行時に必要
let db = sqlite.open(":memory:");
// let db = sqlite.open("mydb.sqlite");

sqlite.exec(db, "CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, age INTEGER)");
sqlite.exec(db, "INSERT INTO users (name,age) VALUES ('Alice',30)");

// パラメータバインド
let rows = sqlite.query(db, "SELECT * FROM users WHERE age > ?", [25]);
for (i, row in rows) {
    io.println("#{row["name"]} (#{row["age"]})");
};

sqlite.last_id(db);     // 最後の INSERT の ROWID
sqlite.changes(db);     // 直前操作の影響行数
sqlite.tables(db);      // テーブル一覧

// トランザクション
sqlite.transaction(db, fn() {
    sqlite.exec(db, "INSERT INTO users (name,age) VALUES ('Bob',25)");
});

sqlite.close(db);
```

### narashino (サンプル拡張)

```rutile
import "narashino";
narashino.population();  // → 176383
narashino.household();   // → 84830
```

---

## import

```rutile
import "io"          // C拡張 (io.so / io.dll)
import "utils.rt"    // 別の .rt ファイル → utils.関数名() で使用可
```

---

## C拡張の作り方

```bash
rutile -api   # rutile_api.h をカレントディレクトリへコピー
```

```cpp
// mylib.cpp
#include "rutile_api.h"
using namespace rutile;

namespace {
Value my_hello(std::vector<Value>& args) {
    if (args.size() != 1) throw RutileError("hello(name): 引数1つ");
    return Value::Str("Hello, " + asString(args[0]) + "!");
}
}

RUTILE_EXTENSION_ENTRYPOINT {
    __rutile_mod.addFunction("hello", my_hello);
    __rutile_mod.addValue("VERSION", Value::Str("1.0"));
}
```

```bash
# Linux
g++ -std=c++17 -O2 -fPIC -shared mylib.cpp -o mylib.so
# Windows MinGW
g++ -std=c++17 -O2 -shared mylib.cpp -o mylib.dll
# Windows MSVC
cl /std:c++17 /O2 /EHsc /LD mylib.cpp /Fe:mylib.dll
```

---

## コマンドラインオプション

| オプション | 説明 |
|---|---|
| `-warning` | 型不一致などを静的解析して実行前に警告 |
| `-jit` | 整数演算・自己再帰関数を x64 JIT コンパイル (Linux/Windows) |
| `-api` | `rutile_api.h` をカレントディレクトリへコピー |

---

## 既知の制限

| 制限 | 補足 |
|---|---|
| ユーザー定義メソッドは `()` 必須 | `let f = obj.method;` でコピー可 |
| JIT は整数演算+自己再帰のみ | それ以外はツリーウォークにフォールバック |
| http はシングルスレッド | 高負荷には不向き |
| async は Rutile 呼び出しをシリアル化 | I/O バウンドに有効、CPU バウンドには不向き |
| sqlite はランタイム動的ロード | libsqlite3.so.0 / sqlite3.dll が実行時に必要 |
| zip の日本語ファイル名 | UTF-8 として処理 (Shift-JIS 環境では文字化けの可能性あり) |

---

*Rutile — シンプルで拡張可能なスクリプト言語 (.rt)*
