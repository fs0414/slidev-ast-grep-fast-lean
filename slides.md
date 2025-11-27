---
theme: default
background: false
class: text-center
highlighter: shiki
lineNumbers: true
shikiConfig:
  theme: 'nord'
drawings:
  persist: false
transition: slide-left
title: fast lean, ast-grep
mdc: true
fonts:
  sans: 'Roboto'
  serif: 'Roboto Slab'
  mono: 'Fira Code'
---

<CoverSlide
  title="fast lean, ast-grep"
  subtitle=""
  event="Terminal Night #1"
  author="fujitani sora"
/>

---

# 自己紹介

<TwoColumnLayout :gap="8">
  <template #left>

- **fujitani sora** / @_fs0414
- <EmojiText emoji="👤">2001（24）</EmojiText>
- <EmojiText emoji="🏢">株式会社トリドリ・software engineer</EmojiText>
- <EmojiText emoji="🎤">技育CAMPの公式メンター</EmojiText>
- <EmojiText emoji="💪">TSKaigiの運営</EmojiText>
- <EmojiText emoji="💻">Terminal Use : NeoVim, WebTerm, ClaudeCode, </EmojiText>
- <EmojiText emoji="📱">Photo, Post 👌</EmojiText>

<p>最近のTips</p>
CodeFormatterに凝っています。<br/>
Prettierのバグを直したり、自作のRust製Ruby Code Formatterを公開したり<br/>
<div>
<a href="https://github.com/fs0414/rfmt">https://github.com/fs0414/rfmt</a>
</div>
  </template>
  <template #right>

<CenteredImage
  src="https://raw.githubusercontent.com/fs0414/imgs/main/fs0414_dot_image.png"
  alt="プロフィール画像"
  width="320px"
/>

  </template>
</TwoColumnLayout>

---

# アジェンダ

1. 従来の検索手法
2. ast-grepとは何か
3. 基本的な使い方
4. 実践例1: フォーマット非依存検索
5. 実践例2: API使用パターン検出
6. 実践例3: リファクタリング支援
7. 高度な活用法
8. まとめと次のステップ

---

# コードベースから特定のパターンを検索したい
<br/>

**ex: Prettierのコードから、isNode関数の呼び出しを全て見つけたい**

<ul v-pre>
<li>文字列で検索する</li>
<li>正規表現で検索する</li>
</ul>

🤔

---

# 文字列ベースの検索

<!-- <TwoColumnLayout> -->
<!--   <template #left> -->

```bash
$ grep "isNode" src/
```

<ul v-pre>
<li>フォーマットに依存</li>
<li>コメントや文字列リテラルも検出</li>
<li>コードの構造は考慮しない</li>
</ul>

  <!-- </template> -->
  <!-- <template #right> -->

```javascript
// すべてマッチする
isNode(node, ["type"])           // ← 検索対象
// isNode is deprecated          // ← コメント
const text = "isNode function"  // ← 文字列リテラル
```

<!--   </template> -->
<!-- </TwoColumnLayout> -->

---

# 正規表現検索

<TwoColumnLayout>
  <template #left>

```bash
$ grep -E "isNode\(.*\[.*\]\)" src/
```

<ul v-pre>
<li>改行を跨ぐパターンには対応しづらい<br/>
    grep -Pzo などのoptionを使えば可能ではある</li>
<li>ネストした構造の表現が複雑</li>
<li>パターンが長くなる傾向</li>
</ul>

  </template>
  <template #right>

```javascript
// マッチする
isNode(node, ["type1", "type2"])

// マッチしない
isNode(node, [
  "type1",
  "type2"
])
```

  </template>
</TwoColumnLayout>

---

# 👀 コードをASTに変換しての検索
<br/>

**AST (Abstract Syntax Tree) = 抽象構文木**

コードをASTにParse<br/>
**→ ASTから「関数呼び出し」のNodeみを検索可能**


```javascript
// コードをASTとして解析
isNode(node, ["type"])
```

```yaml
# AST表現（簡略化）
CallExpression:
  callee:
    name: "isNode"
  arguments:
    - Identifier: "node"
    - ArrayExpression:
        - "type"
```
<!---->
<!--   </template> -->
<!-- </TwoColumnLayout> -->

---

# 検索方式の比較表

| 方式 | 速度 | 精度 | フォーマット非依存 | 構造理解 |
|------|------|------|-------------------|---------|
| **grep** | 高 | 低 | 非対応 | 非対応 |
| **正規表現** | 中 | 中 | 部分的 | 非対応 |
| **ASTを利用** | 低 | 高 | 対応 | 対応 |

---

# ast-grep : ASTベースの検索ツール

<TwoColumnLayout>
  <template #left>

**ast-grepの特徴**

<ul>
    <li>コードの構造検索・置換を行う</li>
    <li>20種類以上のプログラミング言語に対応 </li>
</ul>
<div>
    <blockquote cite="https://www.huxley.net/bnw/four.html">
        <p>
          構文解析機能を備えたgrep/sedのようなものと考えてください。<br/>
          AST（抽象構文木）に基づいてコードを検索・修正するためのパターンを記述でき、数千ものファイルに対してインタラクティブに操作を行うことが可能です。
        </p>
    </blockquote>
    <a href="https://ast-grep.github.io/">https://ast-grep.github.io/</a>
</div>

</template>
<template #right>

**ast-grepが理解するコード構造**

```javascript
// 検索対象コード
isNode(node, ["type1", "type2"])
```

```yaml
# ast-grepのパターンマッチング
pattern: isNode($NODE, $TYPES)

# マッチする箇所
$NODE  → node
$TYPES → ["type1", "type2"]
```

  </template>
</TwoColumnLayout>

---

# ast-grepの基本 - メタ変数

| メタ変数 | 説明 | 使用例 |
|---------|------|--------|
| $VAR | 単一ノードにマッチ | $VAR.method() |
| $$$ | 0個以上のノードにマッチ | func($$$) |
| $$MULTI | 名前付き複数ノード | func($$ARGS) |

**例:**
```javascript
// パターン: console.$METHOD($$$)
console.log("hello")        // マッチ
console.error("error", e)   // マッチ
console.warn()              // マッチ
```

---

<!-- # 基本コマンド -->
<!---->
<!-- **1. パターン検索** -->
<!-- ```bash -->
<!-- ast-grep --lang js --pattern 'PATTERN' [ファイル] -->
<!-- ``` -->
<!---->
<!-- **2. 置換（プレビュー）** -->
<!-- ```bash -->
<!-- ast-grep --pattern 'OLD' --rewrite 'NEW' [ファイル] -->
<!-- ``` -->
<!---->
<!-- **3. YAMLルールで検索** -->
<!-- ```bash -->
<!-- ast-grep scan --rule rule.yml [ディレクトリ] -->
<!-- ``` -->
<!---->
<!-- --- -->

# 実践例1 - フォーマット非依存検索
<br/>

**ref : github.com/prettier/prettier**

Parse結果としての構文木は同一<br/>
WhiteSpaceなどの情報はParserの字句解析時点で除去される

```javascript
// パターン1: 1行
isNode(node, ["sequence", "mapping"])

// パターン2: 複数行・インデント
isNode(node, [
  "documentHead",
  "documentBody",
  "flowMapping",
  "flowSequence",
])

// パターン3: スペースなし
isNode(node,["type"])
```
---

# フォーマット非依存検索 - grepの場合

<TwoColumnLayout>
  <template #left>

```bash
$ grep "isNode.*\[" src/language-yaml/
```

<ul v-pre>
<li>Defaultで改行を跨ぐパターンを検出できない</li>
<li>複雑な正規表現は認知負荷が高い</li>
</ul>

  </template>
  <template #right>

```
✅ isNode(node, ["sequence", "mapping"])
✅ isNode(node,["type"])
❌ isNode(node, [\n     // 複数行は検出できない
```

  </template>
</TwoColumnLayout>

---

# フォーマット非依存検索 - ast-grepの場合
<!---->
<!-- <TwoColumnLayout> -->
<!--   <template #left> -->

```bash
$ ast-grep --lang js --pattern 'isNode($NODE, [$$$])' src/language-yaml/
```

**すべてのフォーマットを検出**

  <!-- </template> -->
  <!-- <template #right> -->

**結果:**
```
✅ isNode(node, ["sequence", "mapping"])
✅ isNode(node, [
     "documentHead",
     "documentBody",
     ...
   ])
✅ isNode(node,["type"])
```

<!--   </template> -->
<!-- </TwoColumnLayout> -->

---

# 実際の検出結果

```
// src/language-yaml/print/misc.js:32:
    !isNode(node, [
      "documentHead",
      "documentBody",
      "flowMapping",
      "flowSequence",
    ])

// src/language-yaml/printer-yaml.js:83:
    if (isNode(node, ["sequence", "mapping"]) && ...)

src/language-yaml/printer-yaml.js:115:
    if (... && !isNode(node, ["document", "documentHead"]))
```

**検出件数: 10箇所以上**

---

# 実践例2 - 空配列チェックパターンの検出

<!-- <TwoColumnLayout> -->
<!--   <template #left> -->
<br/>

**配列の存在と要素チェック**

**目的: 空でない配列をチェックする様々なパターンを検出**

  <!-- </template> -->
  <!-- <template #right> -->

```javascript
// prettierで見られるパターン
// パターン1: 長さチェック
if (array && array.length > 0)

// パターン2: 論理AND
if (array && array.length)

// パターン3: ユーティリティ関数
if (isNonEmptyArray(array))

// パターン4: Optional chaining
if (array?.length > 0)
```

<!--   </template> -->
<!-- </TwoColumnLayout> -->

---

# 空配列チェックパターンの検出

```bash
# パターン1: array.length > 0 を検出
$ ast-grep --pattern 'if ($ARRAY && $ARRAY.length > 0)' src/

# パターン2: isNonEmptyArray関数の使用箇所を検出
$ ast-grep --pattern 'isNonEmptyArray($ARG)' src/
```

**検出結果:**
```
✅ src/language-js/print.js:89    if (node.decorators && node.decorators.length > 0)
✅ src/language-js/utils.js:234   if (comments && comments.length > 0)
✅ src/common/util.js:45          if (isNonEmptyArray(node.properties))
✅ src/language-html/print.js:156 if (isNonEmptyArray(node.children))
✅ src/document/doc-printer.js:78 if (parts && parts.length > 0)

検出件数: 50箇所以上（統一の余地あり）
```

---

# YAMLルールで高度な検索

<TwoColumnLayout>
  <template #left>

```bash
$ ast-grep scan --rule debug-rule.yml src/
```

<ul v-pre>
<li>メッセージをカスタマイズ</li>
<li>重要度を設定</li>
<li>複数ルールを管理しやすい</li>
</ul>

  </template>
  <template #right>

```yaml
# debug-rule.yml
id: find-debug-console
language: js
rule:
  pattern: if ($DEBUG) console.$METHOD($$$)
message: Debug console statement found
severity: warning
```

  </template>
</TwoColumnLayout>

---

# 実践例3 - リファクタリング支援

<TwoColumnLayout>
  <template #left>

**配列の最後の要素へのアクセス**

**prettierコードベースには両方が混在している**

  </template>
  <template #right>

```javascript
// 古いスタイル（ES5）
arr[arr.length - 1]
items[items.length - 1]

// 新しいスタイル（ES2022）
arr.at(-1)
items.at(-1)
```

  </template>
</TwoColumnLayout>

---

# リファクタリング候補の検出

**実行:**
```bash
$ ast-grep scan --rule modernize-array.yml src/language-yaml/utils.js
```

<br/>
```yaml
id: modernize-array-access
language: js
rule:
  any:
    - pattern: $ARR[$ARR.length - 1]
    - pattern: $ARR.slice(-1)[0]
message: Consider using modern array.at(-1) syntax
```

---

# 検出結果とリファクタリング

**utils.js での検出結果**

```
help[modernize-array-access]:
    ┌─ src/language-yaml/utils.js:190:7
    │
190 │ lines[lines.length - 1] = [...lines.at(-1), ...words];
    │ ^^^^^^^^^^^^^^^^^^^^^^^

help[modernize-array-access]:
    ┌─ src/language-yaml/utils.js:250:7
    │
250 │ lines[lines.length - 1] = [...lines.at(-1), ...words];
    │ ^^^^^^^^^^^^^^^^^^^^^^^

help[modernize-array-access]:
    ┌─ src/language-yaml/utils.js:261:9
    │
261 │ words[words.length - 1] += " " + word;
    │ ^^^^^^^^^^^^^^^^^^^^^^^
```

**3箇所のリファクタリング候補を発見**

---

# 自動置換の実行

<TwoColumnLayout>
  <template #left>

**基本の置換コマンド:**
```bash
$ ast-grep --lang js \
  --pattern '$ARR[$ARR.length - 1]' \
  --rewrite '$ARR.at(-1)' \
  src/language-yaml/utils.js
```

**重要なオプション:**
<ul v-pre>
<li>--pattern: 検索パターン</li>
<li>--rewrite: 置換パターン</li>
<li>--update-all: プレビューではなく実際に更新</li>
<li>--interactive: 対話的に確認しながら置換</li>
</ul>

  </template>
  <template #right>

**実際の置換結果（--update-all適用後）:**

<div class="diff-block">

```js
// before
const lastLine = lines[lines.length - 1];
return arr[arr.length - 1] || defaultValue;
const last = words[words.length - 1];
```

```js
// after
const lastLine = lines.at(-1);
return arr.at(-1) || defaultValue;
const last = words.at(-1);
```

</div>

  </template>
</TwoColumnLayout>

---

# 高度な使用例 - 条件の組み合わせ

<ul v-pre>
<li>all : すべての条件を満たす</li>
<li>any : いずれかの条件を満たす</li>
<li>not : 条件を満たさない</li>
<li>inside : 特定のスコープ内</li>
</ul>

```yaml
id: complex-pattern
language: js
rule:
  all:
    - pattern: if ($COND) console.$METHOD($$$)
    - not:
        pattern: if (process.env.DEBUG) console.log($$$)
  any:
    - inside:
        pattern: function $FUNC($$$) { $$$ }
message: Non-production console statement found
```

---

# 高度な使用例 - スコープ検索

<TwoColumnLayout>
  <template #left>

<ul v-pre>
<li>特定のswitch caseの処理を検索</li>
<li>関数内の特定パターンのみ検出</li>
<li>クラスメソッド内の処理を検索</li>
</ul>

  </template>
  <template #right>

```yaml
id: find-in-switch
language: js
rule:
  pattern: |
    switch ($EXPR) {
      $$$
      case "root": { $$$BODY }
      $$$
    }
```

  </template>
</TwoColumnLayout>

---

# メタ変数の高度な使用法

**パターン:**
```javascript
$ARR[$ARR.length - 1]
```

**重要:** $ARR が2回出現 = **同じ変数**である必要がある

**マッチする:**
```javascript
lines[lines.length - 1]  // ✅ lines が2回
words[words.length - 1]  // ✅ words が2回
```

**マッチしない:**
```javascript
lines[words.length - 1]  // ❌ 異なる変数
```

---

# セキュリティチェックの例

<TwoColumnLayout>
  <template #left>

**Node.jsでの典型的な脆弱性:**

```javascript
// コマンドインジェクション
exec('cat ' + userInput, cb);  // 危険

// eval使用
eval(getUserInput());  // 危険

// パストラバーサル
fs.readFile('./uploads/' + file, cb);  // 危険
```

  </template>
  <template #right>

**検出ルール例:**
```yaml
id: detect-command-injection
language: js
rule:
  pattern: exec($STR + $VAR, $$$)
message: Potential command injection
severity: error
```

  </template>
</TwoColumnLayout>

---

<!-- # ast-grepの特徴 -->
<!---->
<!-- | 特徴 | 説明 | -->
<!-- |------|------| -->
<!-- | **高精度** | コードの構造を理解、誤検出が少ない | -->
<!-- | **フォーマット非依存** | インデント・改行に左右されない | -->
<!-- | **構造保持** | 構造を保ったまま変更可能 | -->
<!-- | **一括変更** | 大規模なリファクタリングに対応 | -->
<!-- | **多言語対応** | JS, TS, Rust, Python, Go等 | -->
<!---->
<!-- --- -->
<!---->
<!-- # ast-grepの特性 -->
<!---->
<!-- **特性:** -->
<!-- - **学習コスト**: パターン構文の習得が必要 -->
<!-- - **パフォーマンス**: 大規模コードベースでは処理時間がかかる -->
<!-- - **言語依存**: パーサーが必要（対応言語は限定的） -->
<!-- - **複雑な構造**: 非常に複雑なパターンは表現が難しい -->
<!---->
<!-- **使い分け:** -->
<!-- - シンプルな文字列検索 → `grep` -->
<!-- - コード構造の検索・置換 → `ast-grep` -->
<!---->
<!-- --- -->

# 総じて好きなところ

- 検索パターンをymlで管理, 共有できる
  - 情報が永続化して読み取り可能であることはAgenticCodgingにとって重要だと思っている
- 構文木で一致するコードの一括置換が可能である
- 特定言語に依存しない汎用ツールである
  - 構文解析可能であれば
  - 個別のCustomLintを理解するよりも汎用的な仕組みかな？と思う
- 使い手次第で色々できる拡張性
- astに関する知識の隠蔽, toolとしてのinterfaceが優れているなと思う

---

# install

**インストール:**
```bash
# macOS (Homebrew)
brew install ast-grep

# bun

bun install @ast-grep/cli

# cargo
cargo install ast-grep
```

**動作確認:**
```bash
ast-grep --version
```

---

# Reference

- **公式ドキュメント**: https://ast-grep.github.io/
- **Playground**: https://ast-grep.github.io/playground.html
- **パターン構文ガイド**: https://ast-grep.github.io/guide/pattern-syntax.html
- **GitHub**: https://github.com/ast-grep/ast-grep

---
layout: center
---

# see you later

👋
