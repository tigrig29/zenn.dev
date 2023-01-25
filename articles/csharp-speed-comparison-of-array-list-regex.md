---
title: '【C#】カンマ区切りの文字列から値を検索する速度比較（配列、List）'
emoji: '🦊'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: ['csharp', 'linq', '正規表現']
published: true
---

:::details 本検証の実施理由

- Naninovel でノベルゲーム制作しております → https://ci-en.net/creator/2349/article/585093
- キャラクター表示に [LayeredCharacter](https://naninovel.com/guide/characters.html#layered-characters) を使っている
- ある Character における『現在表示中のポーズ名』を抜き取りたい
- Character の『現在表示中の情報』（ポーズが〇〇で、表情が ×× で……という情報）は `IActor.Appearance` から抜き取れた
- 抜き取ったテキストは以下のような内容
  ```plaintext
  body+default-0,body-front-0,body-front-1,face/default-ai-1,face/default-ai-2,face/default-ai-3,face/default-ai-4,face/default+def-1,face/default-def-2,face/default-do-1,face/default-do-2,face/default-ki-1,face/default-ki-2,face/default-ki-3,face/default-ki-4,face/default-odo-1,face/default-odo-2,face/default-odo-3,face/default-raku-1,face/default-raku-2,face/default-tokusyu1-1,face/default-tokusyu1-2,face/default-tokusyu2-1,face/default-tokusyu2-2,face/default-tokusyu2-3,face/default-tokusyu2-4,face/default-tokusyu3-1,face/default-tokusyu3-2,face/front-do-1,face/front-do-2,face/front-raku-1,face/front-raku-2,face/front-tokusyu1-1,face/front-tokusyu1-2,face/front-tokusyu2-1,face/front-tokusyu2-2,face/front-tokusyu3-1,face/front-tokusyu3-2,face/front-tokusyu4-1,face/front-tokusyu4-3,face/front-tokusyu5-1,face/front-tokusyu5-2,face/front-tokusyu5-3,face/front-tokusyu5-4
  ```
- この中から『現在表示中のポーズ名』を示すのは `body+` で始まる値
- つまり上記テキストから `body+default-0` を抜き取りたい
  - 正規表現で抽出？
  - 配列化して Find/First/Where メソッドあたりで検索？
  - → 速度比較して見よう！

:::

結果はこちら → [速度比較結果](#速度比較結果)

# 前提

- .NET 6.0.10
- C# 10.0

# 検索処理の詳細

```plaintext
body+default-0,body-front-0,body-front-1,face/default-ai-1,face/default-ai-2,face/default-ai-3,face/default-ai-4,face/default+def-1,face/default-def-2,face/default-do-1,face/default-do-2,face/default-ki-1,face/default-ki-2,face/default-ki-3,face/default-ki-4,face/default-odo-1,face/default-odo-2,face/default-odo-3,face/default-raku-1,face/default-raku-2,face/default-tokusyu1-1,face/default-tokusyu1-2,face/default-tokusyu2-1,face/default-tokusyu2-2,face/default-tokusyu2-3,face/default-tokusyu2-4,face/default-tokusyu3-1,face/default-tokusyu3-2,face/front-do-1,face/front-do-2,face/front-raku-1,face/front-raku-2,face/front-tokusyu1-1,face/front-tokusyu1-2,face/front-tokusyu2-1,face/front-tokusyu2-2,face/front-tokusyu3-1,face/front-tokusyu3-2,face/front-tokusyu4-1,face/front-tokusyu4-3,face/front-tokusyu5-1,face/front-tokusyu5-2,face/front-tokusyu5-3,face/front-tokusyu5-4
```

このようなカンマ区切りのテキスト中から、 `body+` という表記を含む値を抜き出す。

- 条件
  - カンマ区切りの各値に重複はない
  - `body+` の表記を含む値は１つだけ

→ 正解の値は `body+default-0` 。

# 比較対象

1. **正規表現**でマッチする値を抜き出す
2. カンマ区切りで **配列** or **List** 作成 → **`foreach`** でループして検索
3. 同上 → **`Find`** メソッドで検索
4. 同上 → **`LINQ First`** メソッドで検索
5. 同上 → **`LINQ Where`** メソッドで検索

## 比較テーブル

| 検証処理        | 処理にかかった時間（ミリ秒） |
| --------------- | ---------------------------- |
| 正規表現        | 後述                         |
| foreach/配列    | 後述                         |
| foreach/List    | 後述                         |
| Find/配列       | 後述                         |
| Find/List       | 後述                         |
| LINQ First/配列 | 後述                         |
| LINQ First/List | 後述                         |
| LINQ Where/配列 | 後述                         |
| LINQ Where/List | 後述                         |

# 検証コード

コード全体は GitHub を御覧ください。

https://github.com/tigrig29/csharp-speed-comparison-of-array-list-regex

※下記は検証対象処理の具体的な処理部分のみを抜き出しております。

:::details 正規表現

```cs
var match = Regex.Match(targetText, "(body\\+.+?(\\d+,|\\d+$))");
var result =  match.Value.Replace(",", "");
```

:::

:::details foreach/配列

```cs
var array = targetText.Split(",");
string result = string.Empty;
foreach (var text in array)
{
    if (text.Contains("body+"))
    {
        result = text;
        break;
    }
}
```

:::

:::details foreach/List

```cs
var list = targetText.Split(",").ToList();
string result = string.Empty;
foreach (var text in list)
{
    if (text.Contains("body+"))
    {
        result = text;
        break;
    }
}
```

:::

:::details Find/配列

```cs
var array = targetText.Split(",");
var result = Array.Find(array, (text) => text.Contains("body+")) ?? string.Empty;
```

:::

:::details Find/List

```cs
var list = targetText.Split(",").ToList();
var result = list.Find((text) => text.Contains("body+")) ?? string.Empty;
```

:::

:::details LINQ First/配列

```cs
var array = targetText.Split(",");
var result = array.First((text) => text.Contains("body+"));
```

:::

:::details LINQ First/List

```cs
var list = targetText.Split(",").ToList();
var result = list.First((text) => text.Contains("body+"));
```

:::

:::details LINQ Where/配列

```cs
var array = targetText.Split(",");
var result = array.Where((text) => text.Contains("body+")).First();
```

:::

:::details LINQ Where/List

```cs
var list = targetText.Split(",").ToList();
var result = list.Where((text) => text.Contains("body+")).First();
```

:::

# 速度比較結果

:::details プログラム実行結果

```bash
$ dotnet run
===================================================
■ 正規表現
result: body+default-0
処理時間: 8.9825 ms
===================================================
■ foreach/配列
result: body+default-0
処理時間: 0.1151 ms
===================================================
■ foreach/List
result: body+default-0
処理時間: 0.2386 ms
===================================================
■ Find/配列
result: body+default-0
処理時間: 0.2551 ms
===================================================
■ Find/List
result: body+default-0
処理時間: 0.2026 ms
===================================================
■ LINQ First/配列
result: body+default-0
処理時間: 0.233 ms
===================================================
■ LINQ First/List
result: body+default-0
処理時間: 0.2466 ms
===================================================
■ LINQ Where/配列
result: body+default-0
処理時間: 0.5936 ms
===================================================
■ LINQ Where/List
result: body+default-0
処理時間: 0.2876 ms
```

:::

| 検証処理        | 処理にかかった時間（ミリ秒） |
| --------------- | ---------------------------- |
| 正規表現        | 8.9825 ms                    |
| foreach/配列    | 0.1151 ms                    |
| foreach/List    | 0.2386 ms                    |
| Find/配列       | 0.2551 ms                    |
| Find/List       | 0.2026 ms                    |
| LINQ First/配列 | 0.233 ms                     |
| LINQ First/List | 0.2466 ms                    |
| LINQ Where/配列 | 0.5936 ms                    |
| LINQ Where/List | 0.2876 ms                    |

# 結論

- **正規表現** → **結構遅い**
- **`foreach`** → **速い**
  - 配列が高速
  - List も十分速いが配列には劣る
- **`Find`** メソッド → **十分速い**
  - 配列も List も大体同じくらいの速度
  - ※少し List のほうが高速かも（何度か実行してみて List のほうが高速な結果の方が多かったため、テストデータによっては明確な差が生じるかも）
- **`LINQ First`** メソッド → **十分速い**
  - 配列も List も大体同じくらいの速度
  - ※少し List のほうが高速かも（ ↑ と同じ理由）
- **`LINQ Where`** メソッド → **ちょっと遅いかも**
  - 特に配列が遅い

**`foreach`（配列）が一番高速**でした。
ただ他の処理よりコードが少し長くなってしまう（読みにくい）という欠点もあります。

→ **コードの可読性も考慮するならば `LINQ First` メソッドが良い**かな……と思いました。
（`Where` はちょっと遅いし、`Find` は戻り値の型が `string?` で null の可能性を考慮したコードを書くのが面倒 ⇒ `First` または `FirstOrDefault` が良さそう）

:::message
あくまで本検証の条件（特定の値を含む要素検索、検索結果は１件である想定、など）に限った結果です。
またテストデータも１件しか用意しておらず正確性が少し低いかと思うので、その点はご注意ください。
:::

# その他気づいたこと

- LINQ メソッドは初回実行時とその後の実行時で速度が違う？
  - 例として、今回検証した `Find/List` の処理を 5 回行うと……
    ```bash
    $ dotnet run
    ===================================================
    ■ Find/List
    result: body+default-0
    処理時間: 1.3224 ms
    ===================================================
    ■ Find/List
    result: body+default-0
    処理時間: 0.0387 ms
    ===================================================
    ■ Find/List
    result: body+default-0
    処理時間: 0.0047 ms
    ===================================================
    ■ Find/List
    result: body+default-0
    処理時間: 0.0048 ms
    ===================================================
    ■ Find/List
    result: body+default-0
    処理時間: 0.0049 ms
    ```
  - 初回だけ極端に遅い（理由よくわかっていません 😭）

# あとがき

最後までお読み下さりありがとうございます。

こういった検証は突き詰めると結構時間がかかりそうで、今回はさらっとテストデータ１件でやってしまいました。
少し正確性には欠きますが、参考になれば幸いです。

間違っている点や気になる点などございましたら、コメント頂けますと幸いです！
