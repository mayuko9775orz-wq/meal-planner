---
name: meal-planner-blog-add
description: 献立ラボ(meal-planner同梱ブログ)に新しい記事を追加する際に必ず使う。blog/posts/ へのHTML追加、SEOメタタグ・OGP・JSON-LD構造化データ・sitemap.xml・rss.xml・blog/index.html記事一覧・BLOG_ARTICLES定数の更新手順、引用ルールを含む。ユーザーから「記事追加」「ブログ」「記事を書く」「熱中症記事」「PINCH記事」等の指示があったら発動する。
---

# ブログ記事追加ガイド

## 新しい記事を追加する流れ

記事1本追加するごとに**5ヶ所**を更新する必要があります:

1. `blog/posts/[slug].html` — 記事HTML本体
2. `blog/index.html` — 記事一覧に追加
3. `blog/sitemap.xml` — URL追加
4. `blog/rss.xml` — 新着feed先頭に追加
5. `index.html` の `BLOG_ARTICLES` 配列先頭に追加(今日タブ横スクロールに反映)

## ファイル命名規則

- ファイル名: `NNN-english-slug.html`(例: `004-adhd-heat-exhaustion.html`)
- NNN: 3桁ゼロパディング、公開順
- slug: 英字小文字、ハイフン区切り、短く

## 記事HTMLテンプレート

既存の `blog/posts/002-adhd-and-cooking.html` を雛形にコピーし、以下の変数を置換:

```yaml
記事タイトル:      [長め・SEOキーワード含む]
記事descriptionメタ: [120字以内、検索結果に表示]
slug:              [例: 004-adhd-heat-exhaustion]
発行日(ISO 8601): 例: 2026-04-22T00:00:00+09:00
発行日(表示):     例: 2026年4月22日
読了時間:          例: 7分
タグ:              例: 研究, ADHD
リード文:          [ホック、1〜2段落]
見出しh2 複数:     [3〜5個。id付きで目次連動]
```

### 必須要素(コピペ順)

1. `<title>` - 32字以内推奨、キーワード前寄せ
2. `<meta name="description">` - 120字以内
3. `<link rel="canonical">` - 絶対URL
4. **OGP一式** (`og:type=article`, `og:title`, `og:description`, `og:url`, `og:image`, `og:locale=ja_JP`)
5. `<meta property="article:published_time">`
6. **Twitter Card** (`twitter:card=summary_large_image`)
7. **BlogPosting schema** (JSON-LD)
8. **BreadcrumbList schema** (JSON-LD)
9. `<link rel="stylesheet" href="../style.css">`
10. 本文
11. 記事末尾のCTA:
    - アプリ誘導(`.app-cta`)
    - 著者ボックス(`.author-box`)
    - 関連記事 (`.related`)

## 本文の書き方

### トーン(絶対)
- 「頑張れ」「気合い」は使わない
- 当事者を責めない・煽らない
- 一人称(私)を使う(当事者性の明示)
- 結論を押し付けない

### 構成
```
導入(共感ポイント)
 → 問題整理
 → 研究/エビデンス
 → 実践/応用
 → 優しいまとめ
 → アプリへの導線(自然に)
```

### 引用のルール
- 出典を**研究者名+媒体名**で明記(例: McEwen(アロスタティック負荷理論))
- 長い引用は避ける、要約で示す
- 断定を避ける:「〜と報告されている」「研究で示されている」
- 当事者経験と研究を明確に分ける

## sitemap.xml 追記

`blog/sitemap.xml` の `</urlset>` の直前に追加:

```xml
<url>
  <loc>https://mayuko9775orz-wq.github.io/meal-planner/blog/posts/[SLUG].html</loc>
  <lastmod>[YYYY-MM-DD]</lastmod>
  <changefreq>monthly</changefreq>
  <priority>0.8</priority>
</url>
```

## rss.xml 追記

`blog/rss.xml` の `<channel>` 直下、**既存 `<item>` より前**に:

```xml
<item>
  <title>[記事タイトル]</title>
  <link>https://mayuko9775orz-wq.github.io/meal-planner/blog/posts/[SLUG].html</link>
  <guid>https://mayuko9775orz-wq.github.io/meal-planner/blog/posts/[SLUG].html</guid>
  <description>[記事説明]</description>
  <pubDate>[RFC 822形式 例: Wed, 22 Apr 2026 00:00:00 +0900]</pubDate>
</item>
```

## blog/index.html 記事一覧追記

`<ul class="post-list">` の**先頭(既存 `<li>` より上)**に:

```html
<li class="post-card">
  <a href="./posts/[SLUG].html">
    <h2>[記事タイトル]</h2>
    <div class="post-meta">
      <time datetime="[YYYY-MM-DD]">[YYYY.MM.DD]</time>
      <span class="post-tag">[カテゴリ]</span>
      <span class="post-tag">[タグ2]</span>
    </div>
    <p class="post-excerpt">[要約]</p>
  </a>
</li>
```

## BLOG_ARTICLES(index.html) 先頭に追記

```js
const BLOG_ARTICLES = [
  { category:'[タグ]', title:'[タイトル]', excerpt:'[要約]', url:'./blog/posts/[SLUG].html', readTime:'[N分]' },
  // ...既存
];
```

アプリ今日タブの横スクロールに**先頭表示**される。

## 関連記事セクション

新記事を追加したら、既存の関連する記事の `<section class="related">` にも今回追加した記事への逆リンクを入れると、ブログ内の回遊性が上がる。

## ベストプラクティス

### タイトル
- 数字入り(「10選」「5つのトリガー」)でクリック率↑
- 疑問形・驚き系は抑える(当事者の期待を裏切りやすい)
- キーワードは前半に

### 記事内リンク
- アプリへ: **本文中に1〜2回、末尾にCTA**
- 関連記事へ: 末尾にまとめて
- 外部権威(研究論文等): 出典として本文中

### 構造化データ最重要
- BlogPosting schema は必ず入れる(検索結果のリッチ表示)
- BreadcrumbList も必ず(パンくず表示)
- 両方とも同じ cannonical URL を参照

### URL指定時の注意
本プロジェクトの本番URLは:
```
https://mayuko9775orz-wq.github.io/meal-planner/
```
OGP/canonical/sitemap/RSS/schema すべてでこのベースURL使用。変更時は全HTML+XMLを一括置換。

## 所要時間の目安

| 内容 | 所要 |
|---|---|
| 2,000字エッセイ(001型) | 30〜45分 |
| 3,000字研究記事(002型、引用あり) | 45〜60分 |
| 1,500字実用記事(003型、リスト) | 25〜35分 |
| 追加5ヶ所の編集 | 10分 |

## アンチパターン

- ❌ レシピや論文の原文コピペ(著作権)
- ❌ BLOG_ARTICLESを更新し忘れる → 今日タブに出ない
- ❌ sitemapを更新し忘れる → Googleにインデックスされない
- ❌ 絶対URLが間違っている → OGPシェアが壊れる
- ❌ canonicalが他記事と被る → SEOペナルティ

## 関連スキル
- `meal-planner-dev` — プロジェクト全体構造
- `.claude/research-notes.md` — ブログ執筆用の研究メモ
