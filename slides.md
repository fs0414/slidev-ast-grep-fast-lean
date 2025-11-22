---
theme: default
background: false
class: text-center
highlighter: shiki
lineNumbers: true
drawings:
  persist: false
transition: slide-left
title: 速習 ast-grep
mdc: true
fonts:
  sans: 'Roboto'
  serif: 'Roboto Slab'
  mono: 'Fira Code'
---

<CoverSlide
  title="速習 ast-grep"
  subtitle=""
  event="Terminal Night #1"
  author="fujitani sora"
/>

---

# 自己紹介

<TwoColumnLayout :gap="8">
  <template #left>

- **fujitani sora** / @_fs0414
- <EmojiText emoji="🏢">株式会社xxx・software engineer</EmojiText>
- <EmojiText emoji="🎤">xxx</EmojiText>
- <EmojiText emoji="💻">xxx</EmojiText>
- <EmojiText emoji="🌆">xxx</EmojiText>

<br> 

👋 

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

# 文字列ベースの検索

<TwoColumnLayout>
  <template #left>

```bash
$ grep "isNode" src/
```

<ul v-pre>
<li>フォーマットに依存</li>
<li>コメントや文字列リテラルも検出</li>
<li>コードの構造は考慮しない</li>
</ul>

  </template>
  <template #right>

```javascript
// すべてマッチする
isNode(node, ["type"])           // ← 検索対象
// isNode is deprecated          // ← コメント
const text = "isNode function"  // ← 文字列リテラル
```

  </template>
</TwoColumnLayout>

---

# 正規表現検索

<TwoColumnLayout>
  <template #left>

```bash
$ grep -E "isNode\(.*\[.*\]\)" src/
```

<ul v-pre>
<li>改行を跨ぐパターンには対応しづらい</li>
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

# ast-grep

<TwoColumnLayout>
  <template #left>

**AST (Abstract Syntax Tree) = 抽象構文木**

ast-grep はコードを構文木として扱う

  </template>
  <template #right>

```
isNode(node, ["type"])
         ↓
    ┌────┴────┐
 call_expr
    ├─ identifier: "isNode"
    ├─ arg[0]: identifier "node"
    └─ arg[1]: array
         └─ string: "type"
```

  </template>
</TwoColumnLayout>

---

# 検索方式の比較表

| 方式 | 速度 | 精度 | フォーマット非依存 | 構造理解 |
|------|------|------|-------------------|---------|
| **grep** | 高 | 低 | 非対応 | 非対応 |
| **正規表現** | 中 | 中 | 部分的 | 非対応 |
| **ast-grep** | 低 | 高 | 対応 | 対応 |

**ast-grepの用途:**
- リファクタリング
- コードパターンの検出
- 構造的な置換

---

# ast-grepの基本 - メタ変数

| メタ変数 | 説明 | 使用例 |
|---------|------|--------|
| `$VAR` | 単一ノードにマッチ | `$VAR.method()` |
| `$$$` | 0個以上のノードにマッチ | `func($$$)` |
| `$$MULTI` | 名前付き複数ノード | `func($$ARGS)` |

**例:**
```javascript
// パターン: console.$METHOD($$$)
console.log("hello")        // マッチ
console.error("error", e)   // マッチ
console.warn()              // マッチ
```

---

# 基本コマンド

**1. パターン検索**
```bash
ast-grep --lang js --pattern 'PATTERN' [ファイル]
```

**2. 置換（プレビュー）**
```bash
ast-grep --pattern 'OLD' --rewrite 'NEW' [ファイル]
```

**3. YAMLルールで検索**
```bash
ast-grep scan --rule rule.yml [ディレクトリ]
```

---

# 実践例1 - フォーマット非依存検索

<TwoColumnLayout>
  <template #left>

**prettierコードベースでの実例:**

構造的には同一

  </template>
  <template #right>

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

  </template>
</TwoColumnLayout>

---

# フォーマット非依存検索 - grepの場合

<TwoColumnLayout>
  <template #left>

```bash
$ grep "isNode.*\[" src/language-yaml/
```

<ul v-pre>
<li>改行を跨ぐパターンを検出できない</li>
<li>正規表現を複雑にしても限界がある</li>
</ul>

  </template>
  <template #right>

```
✅ isNode(node, ["sequence", "mapping"])
✅ isNode(node,["type"])
❌ isNode(node, [     # 複数行は検出できない
```

  </template>
</TwoColumnLayout>

---

# フォーマット非依存検索 - ast-grepの場合

<TwoColumnLayout>
  <template #left>

```bash
$ ast-grep --lang js --pattern 'isNode($NODE, [$$$])' src/language-yaml/
```

**すべてのフォーマットを検出!**

  </template>
  <template #right>

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

  </template>
</TwoColumnLayout>

---

# 実際の検出結果

**prettierリポジトリでの実行結果**

```
src/language-yaml/print/misc.js:32:
    !isNode(node, [
      "documentHead",
      "documentBody",
      "flowMapping",
      "flowSequence",
    ])

src/language-yaml/printer-yaml.js:83:
    if (isNode(node, ["sequence", "mapping"]) && ...)

src/language-yaml/printer-yaml.js:115:
    if (... && !isNode(node, ["document", "documentHead"]))
```

**検出件数: 10箇所以上**

---

# 実践例2 - API使用パターンの検出

<TwoColumnLayout>
  <template #left>

**prettierでの実例: デバッグコードの検出**

**目的: このデバッグコードを一括検出したい**

  </template>
  <template #right>

```javascript
// src/language-yaml/printer-yaml.js より
const DEBUG = node.type === 'plain' && node.value === 'key';
if (DEBUG) console.error('\n🔵 Starting...');

if (node.type !== "mappingValue" && hasLeadingComments(node)) {
  if (DEBUG) console.error('  ➕ [領域1] leadingComments');
  parts.push(...);
}

// ... さらに10箇所以上続く
```

  </template>
</TwoColumnLayout>

---

# デバッグコード検出 - コマンド

```bash
$ ast-grep --pattern 'if ($DEBUG) console.$METHOD($$$)' \
    src/language-yaml/printer-yaml.js
```

**検出結果:**
```
✅ Line 84:  if (DEBUG) console.error('  ➕ [領域3+] ...');
✅ Line 87:  if (DEBUG) console.error('  ➕ [領域3+] ...');
✅ Line 93:  if (DEBUG) console.error('  ➕ [領域4] ...');
✅ Line 102: if (DEBUG) console.error('  ➕ [領域5] ...');
✅ Line 111: if (DEBUG) console.error('  ➕ [領域5] ...');
✅ Line 116: if (DEBUG) console.error('  ➕ [領域6] ...');
✅ Line 131: if (DEBUG) console.error('  ➕ [領域7] ...');
✅ Line 151: if (DEBUG) console.error('  ➕ [領域8] ...');
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

**prettierコードベースには両方が混在している!**

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

<TwoColumnLayout>
  <template #left>

**実行:**
```bash
$ ast-grep scan --rule modernize-array.yml src/language-yaml/utils.js
```

  </template>
  <template #right>

```yaml
id: modernize-array-access
language: js
rule:
  any:
    - pattern: $ARR[$ARR.length - 1]
    - pattern: $ARR.slice(-1)[0]
message: Consider using modern array.at(-1) syntax
```

  </template>
</TwoColumnLayout>

---

# 検出結果とリファクタリング

**prettier の utils.js での検出結果**

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

**3箇所のリファクタリング候補を発見!**

---

# 自動置換の実行

```bash
$ ast-grep --lang js \
  --pattern '$ARR[$ARR.length - 1]' \
  --rewrite '$ARR.at(-1)' \
  src/language-yaml/utils.js
```

**プレビュー結果:**
```diff
- lines[lines.length - 1] = [...lines.at(-1), ...words];
+ lines.at(-1) = [...lines.at(-1), ...words];

- words[words.length - 1] += " " + word;
+ words.at(-1) += " " + word;
```

**`--update-all` で実際に適用可能**

---

# 高度な使用例 - 条件の組み合わせ

<TwoColumnLayout>
  <template #left>

<ul v-pre>
<li><code>all</code>: すべての条件を満たす</li>
<li><code>any</code>: いずれかの条件を満たす</li>
<li><code>not</code>: 条件を満たさない</li>
<li><code>inside</code>: 特定のスコープ内</li>
</ul>

  </template>
  <template #right>

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

  </template>
</TwoColumnLayout>

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

# メタ変数の高度な活用

**パターン:**
```javascript
$ARR[$ARR.length - 1]
```

**重要:** `$ARR` が2回出現 = **同じ変数**である必要がある

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

  </template>
  <template #right>

```javascript
// コマンドインジェクション
exec('cat ' + userInput, cb);  // 危険

// eval使用
eval(getUserInput());  // 危険

// パストラバーサル
fs.readFile('./uploads/' + file, cb);  // 危険
```

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

# ast-grepの特徴

| 特徴 | 説明 |
|------|------|
| **高精度** | コードの構造を理解、誤検出が少ない |
| **フォーマット非依存** | インデント・改行に左右されない |
| **構造保持** | 構造を保ったまま変更可能 |
| **一括変更** | 大規模なリファクタリングに対応 |
| **多言語対応** | JS, TS, Rust, Python, Go等 |

---

# ast-grepの特性

**特性:**
- **学習コスト**: パターン構文の習得が必要
- **パフォーマンス**: 大規模コードベースでは処理時間がかかる
- **言語依存**: パーサーが必要（対応言語は限定的）
- **複雑な構造**: 非常に複雑なパターンは表現が難しい

**使い分け:**
- シンプルな文字列検索 → `grep`
- コード構造の検索・置換 → `ast-grep`

---

# 実用シーン別の使い分け

**ast-grepの用途:**
- API変更に伴うコード更新
- コーディング規約の検査
- 構文の移行
- 特定のパターンの一括検出

**grepの用途:**
- ファイル名の検索
- 文字列リテラルの検索
- 単純なキーワード検索
- ログの検索

---

# 実践的な活用フロー

```
1. パターンの特定
   ↓
2. ast-grepで検索
   ↓
3. 検出結果の確認
   ↓
4. ルールファイルの作成
   ↓
5. --rewriteでプレビュー
   ↓
6. テスト実行
   ↓
7. --update-all で適用
```

**注意: バージョン管理下での実行を推奨**

---

# prettierでの具体的な成果

| ケース | 検出数 | ファイル |
|--------|--------|---------|
| **デバッグコード** | 10+ | `printer-yaml.js` |
| **isNode呼び出し** | 10+ | `language-yaml/*` |
| **古い配列アクセス** | 3 | `utils.js` |
| **型チェックパターン** | 1 | `utils.js` |
| **switch文** | 1 | `printer-yaml.js` |

**すべて実際のprettierコードベースから抽出**

---

# インストールと環境構築

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

# 学習リソース

**公式リソース:**
- 📖 **公式ドキュメント**: https://ast-grep.github.io/
- 🎮 **Playground**: https://ast-grep.github.io/playground.html
- 📘 **パターン構文ガイド**: https://ast-grep.github.io/guide/pattern-syntax.html
- 💻 **GitHub**: https://github.com/ast-grep/ast-grep

**実践:**
- prettierリポジトリで試す
- 自分のプロジェクトで実験
- カスタムルールを作成

---

# 次のステップ

**1. 動作確認**
```bash
git clone https://github.com/prettier/prettier.git
cd prettier
ast-grep --lang js --pattern 'console.$METHOD($$$)' src/
```

**2. 自分のプロジェクトで**
- よく使うパターンを特定
- ルールファイルを作成
- CI/CDに組み込む

**3. チームで共有**
- プロジェクト共通のルール作成
- コードレビューに活用
- リファクタリング計画に活用

---

# まとめ

**キーポイント:**
- **AST (構文木) ベースの検索**
- **フォーマット非依存・高精度**
- **検索だけでなく置換も可能**
- **大規模リファクタリングに対応**

**基本コマンド:**
```bash
# 基本検索
ast-grep --pattern 'PATTERN' file

# 置換
ast-grep --pattern 'OLD' --rewrite 'NEW' file

# ルール検索
ast-grep scan --rule rule.yml dir
```

---

# 補足資料

**同梱ファイル:**
- `ast-grep-examples.sh` - 10個の実行可能な例
- `ast-grep-knowledge-base.md` - 詳細ドキュメント（元資料）

**実行方法:**
```bash
cd /path/to/prettier
./ast-grep-examples.sh
```

すべての例がそのまま動作します。

---
layout: center
class: text-center
---

# ご清聴ありがとうございました

**スライド作成日**: 2025-11-22
**ベース**: prettier main branch
**サンプルコード**: すべて実在のコードから抽出
