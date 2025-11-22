---
# テーマ設定
theme: apple-basic

# プレゼンテーションのタイトル
title: プレゼンテーションタイトル

# プレゼンテーション情報（メタデータ）
info: |
  ## プレゼンテーション概要
  ここにプレゼンテーションの概要を記載します。

# スライドトランジション
transition: slide-left

# シンタックスハイライター
highlighter: shiki

# 行番号表示
lineNumbers: false

# 描画機能
drawings:
  persist: false

# ダウンロード機能を有効化
download: true

# MDC構文を有効化
mdc: true
---

<CoverSlide
  title="プレゼンテーションタイトル"
  subtitle="サブタイトルや簡単な説明"
  event="イベント名"
  author="fujitani sora"
  :social="{ github: 'fs0414', twitter: '_fs0414' }"
/>

<!--
スピーカーノート: ここにプレゼンテーションノートを書きます
-->

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
class: slide-gradient-bg
---

# 見出しスライド

## サブ見出し

通常のテキストはこのように表示されます。oklch color spaceを使用した美しいカラーパレットが特徴です。

- <EmojiText emoji="✨">リスト項目1</EmojiText>
- <EmojiText emoji="🎨">リスト項目2</EmojiText>
  - ネストしたリスト
  - ネストしたリスト2
- <EmojiText emoji="🚀">リスト項目3</EmojiText>

> これは引用テキストです。
> 重要な情報を強調するのに便利です。

---

# コードハイライト

TypeScriptのコード例:

```ts {2-4|6-8|all}
// コードハイライトの例
interface User {
  name: string
  age: number
}

const user: User = {
  name: 'Taro',
  age: 25
}

console.log(user)
```

<v-click>

JetBrains Mono フォントでコードを見やすく表示

</v-click>

---
layout: two-cols
---

# 2カラムレイアウト

左側のコンテンツ

- <EmojiText emoji="📝">ポイント1</EmojiText>
- <EmojiText emoji="💡">ポイント2</EmojiText>
- <EmojiText emoji="🎯">ポイント3</EmojiText>

::right::

# 右側

右側のコンテンツ

```js
// コード例
const hello = 'world'
console.log(hello)
```

---

<GradientHeading :animated="true">
  グラデーション見出し
</GradientHeading>

アニメーション付きのグラデーション効果

<v-clicks>

- Purple to Blue のグラデーション
- アニメーションするテキストシャドウ
- oklch color space の活用

</v-clicks>

<br>

<div class="mt-8">

```css
.gradient-heading {
  background: linear-gradient(135deg,
    oklch(0.65 0.25 270) 0%,
    oklch(0.7 0.22 240) 50%,
    oklch(0.75 0.2 210) 100%
  );
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}
```

</div>

---

# 画像表示

<CenteredImage
  src="https://via.placeholder.com/800x400"
  alt="サンプル画像"
  width="800px"
  caption="画像のキャプション"
/>

---
layout: center
---

# View Transition対応

<v-click>

<div class="text-6xl emoji">
🎬
</div>

</v-click>

<v-click>

モダンブラウザのView Transition APIに対応しています

</v-click>

---
layout: center
class: text-center
---

<GradientHeading>
  ご清聴ありがとうございました
</GradientHeading>

<div class="mt-8">
  <SocialLinks
    github="fs0414"
    twitter="_fs0414"
  />
</div>
