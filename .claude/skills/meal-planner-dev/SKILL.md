---
name: meal-planner-dev
description: 献立プランナー(meal-planner)プロジェクトの開発ガイド。index.html(単一HTMLファイル、約3800行)のレシピ追加、機能実装、バグ修正、デザイン調整を行う際は必ずこのスキルを参照すること。「このアプリ」「献立アプリ」「kondate」「meal-planner」等のキーワード、または /Users/mayuko/Library/Mobile Documents/com~apple~CloudDocs/06_開発/献立アプリ 以下のファイルを編集する際に発動する。ファイル全体を読まなくても、このスキルを読めばアーキテクチャ・命名規則・行番号マップ・設計思想・アンチパターンが分かる。
---

# 献立プランナー 開発ガイド

## 1. プロジェクトの魂

### コンセプト(最重要)
**「段取りが苦手でも、料理がうまくいく」**

発達障害(ADHD/ASD)当事者や、共働きで段取りに疲れた人が、認知負荷を下げて料理を完遂できることを目的とする。すべての実装判断はこのコンセプトとの整合性で決める。

### 設計原則
1. **情報密度を下げる** — 選択肢は常に3つ以内を基本
2. **一つずつ見せる** — 「今これ」に集中できる視覚階層(NOWモード)
3. **優しいトーン** — 煽らない、責めない、讃えすぎない
4. **時間の見通しを示す** — あと何分、次に何
5. **落ち着いた配色** — 彩度抑えめ、鋭い対比なし
6. **感覚過敏配慮** — 派手なアニメ・音・赤警告を避ける

### やってはいけないこと
- 外部ライブラリ/フレームワーク追加(Vanilla JS厳守)
- 「時短!」「効率!」と煽る文言
- 派手なアニメーション・点滅・通知音
- ポップアップ広告・画面追従広告
- 「ADHDのあなたへ」等のラベリング訴求
- SNS的リアクション要素(いいね・ポイント)
- 決定的に重要: 課金の壁を設けない(フリー+アフィリエイトが基本)

## 2. 技術構成

- **単一HTMLファイル**: `index.html`(約3800行)
- **フレームワーク**: なし(Vanilla JS)
- **CSS**: インライン `<style>`、CSS変数でトークン管理
- **状態管理**: `localStorage`(キー prefix `v3_`)
- **ホスティング**: Vercel (`meal-planner-topaz-delta.vercel.app`)
- **デプロイ**: `main` ブランチ push で自動(現在の作業ブランチは `add-project-config`)

## 3. ファイル地図(index.html 行番号)

### HTML構造
| セクション | 行番号 |
|---|---|
| `<style>` CSS全般 | 〜550 |
| 今日の献立タブ | 598 |
| 週間献立タブ | 621 |
| レシピタブ | 668 |
| 調理タブ | 726 |
| 買物タブ | 767 |
| 栄養タブ | 791 |
| レシピモーダル | 813 |
| レシピ詳細モーダル | 845 |
| ガントブロック詳細モーダル | 853 |
| スロット追加モーダル | 861 |
| 設定モーダル | 883 |
| フッター/開示文 | 末尾 |

### JavaScript主要セクション
| 機能 | 開始行 | キー関数 |
|---|---|---|
| 旬食材マップ | 〜980 | `isSeasonalRecipe` |
| 基本レシピDB | 1009 | `basicRecipeDB` |
| 主食レシピDB | 1092 | `shushokuRecipeDB` |
| あかりレシピDB | 1117 | `akariRecipeDB` |
| ホットクックDB | 1141 | `hotcookRecipeDB` |
| 限界レシピDB | 1160 | `limitRecipeDB` |
| 挑戦レシピDB | 1174 | `challengeRecipeDB` |
| 外部サイトDB | 1184 | `siteRecipeDB` |
| レシピ集約 | 1260 | `getAllRecipes` |
| レベル判定 | 1280 | `getRecipeLevel` |
| 状態保存/読込 | 1315 | `saveState`/`loadState` |
| アフィリエイト | 1356 | `saveAffiliateSettings`/`getAffiliateLinks`/`renderMealKitCard` |
| モード切替 | 1482 | `setCookingMode` |
| 週間ボード描画 | 1559 | `renderWeekBoard` |
| 週間自動生成 | 1612 | `autoGenerateWeek` |
| スロットモーダル | 1695 | `openSlotModal` |
| 今日ボード描画 | 〜2040 | `renderTodayBoard` |
| 今日の自動生成 | 〜2130 | `autoGenerateToday` |
| レシピ一覧 | 〜2220 | `renderRecipeList` |
| レシピ詳細 | 〜2335 | `showRecipeDetail` |
| 並行調理タイムライン | 〜2500 | `generateTimeline`, `scheduleStepsV3` |
| 効率分析 | 〜2830 | `updateEfficiencySummary` |
| ガント描画 | 〜2880 | `renderGantt` |
| ガント詳細モーダル | 〜2920 | `showGanttDetail` |
| ステップリスト | 〜3000 | `renderStepList` |
| NOWモード | 〜3100 | `renderNowMode` |
| 調理タイマー | 〜3280 | `startTimer`/`stopTimer` |
| 買い物リスト | 〜3320 | `generateShoppingFromScope`/`updateShoppingDisplay` |
| 栄養バランス | 〜3450 | `updateNutrition`/`drawRadarChart` |
| Notion連携 | 〜3600 | `sendToNotion` |
| テーマ切替 | 〜3660 | `initTheme` |

正確な行番号が必要なときは `Grep` で関数名検索。コミット後にずれる可能性があるため目安として使うこと。

## 4. データモデル

### レシピ構造
```js
{
  title: 'レシピ名',
  category: '主菜' | '副菜' | '汁物' | 'サラダ' | '主食' | 'ごはん・麺',
  desc: '短い説明(1文)',
  ingredients: ['材料名/分量', ...],  // 'キャベツ/1/4個' 形式(スラッシュ区切り)
  tags: ['タグ1', 'タグ2'],
  flavors: ['味噌', '醤油', ...],  // 味付け分類
  cookTime: 30,  // 分
  steps: [
    {text:'手順文', duration:5, resource:'手作業'|'コンロ1'|'コンロ2'|'オーブン'|'レンジ'|'炊飯器'|'ホットクック', passive:false}
  ],
  nutrition: {protein:3, vegetable:2, carb:1, other:0},  // 0〜4程度

  // 拡張: 難易度レベル
  limitLevel: 'limit' | 'normal' | 'challenge',  // 明示したい時のみ
  limitTags: ['まな板不要','冷凍OK','一鍋','放置系','レンジのみ','調味料3種以内'],  // limit時
  challengeTags: ['本格','時間必要','複数工程','特別な食材']  // challenge時
}
```

`limitLevel` を省略した場合、`getRecipeLevel()` が `cookTime` と `steps.length` から自動推測する(`cookTime ≤ 15 && steps ≤ 5` → `easy`、`cookTime ≥ 90 || steps ≥ 9` → `challenge`、他は `normal`)。

### ID規則
- `b_タイトル` — 基本レシピ(basic + shushoku)
- `a_タイトル` — あかりレシピ
- `h_タイトル` — ホットクック
- `l_タイトル` — 限界
- `c_タイトル` — 挑戦
- `s_サイト_タイトル` — 外部サイト
- `m_uuid` — マイレシピ

### 週間献立構造
```js
weekPlan[dayIdx] = {
  shushoku: [{id, title}, ...],
  shusai: [...],
  fukusai: [...],
  shirumono: [...],
  locked: false
}
```
`DAYS = ['月','火','水','木','金','土','日']`、`dayIdx` は月曜=0。今日のindexは `(new Date().getDay() + 6) % 7`。

### 調理モード
`cookingMode: 'normal' | 'limit' | 'challenge'` - `localStorage v3_mode` に永続化。`setCookingMode(mode)` で切替。

### localStorage キー
- `v3_week` - 週間献立
- `v3_my` - マイレシピ
- `v3_shop` - 買い物リスト
- `v3_ng` - NGレシピID
- `v3_fav` - お気に入りID
- `v3_history` - 使用履歴
- `v3_mode` - 調理モード
- `v3_affiliate` - アフィリエイト設定
- `v3_notion` - Notion設定
- `v3_theme` - テーマ(light/dark)

## 5. 命名規則とパターン

| 動詞 | 用途 |
|---|---|
| `get*` | 純粋関数(値を返す) |
| `render*` | DOM描画 |
| `update*` | 既存DOMの更新 |
| `save*`/`load*` | 永続化 |
| `open*Modal`/`close*Modal` | モーダル開閉 |
| `toggle*` | 状態切替 |
| `handle*`/`on*` | イベントハンドラ(少ない) |

CSS変数(`:root` にあり):
- `--primary`, `--primary-light`, `--primary-dark` - 落ち着いた緑系
- `--accent`, `--accent-light` - くすんだ金色
- `--bg`, `--card`, `--text`, `--text-light`, `--border` - 基本色
- `--success`, `--seasonal`, `--kagome`, `--ajinomoto` 等 - 意味付き色
- ダークモード対応: `[data-theme="dark"]` で上書き

## 6. 実装パターン(よくある追加作業)

### レシピを追加する
1. 適切なDB(`basicRecipeDB`/`limitRecipeDB`等)の配列末尾に追加
2. `cookTime`, `steps` の `resource` と `duration` は並行調理計算に影響するため正確に
3. 放置時間(煮込み等)は `passive: true` を明示。これで料理人稼働率に含まれなくなる
4. 下ごしらえタグ(解凍/浸水等)がある場合は `preSteps` に別途記述する既存構造を参考に
5. `nutrition` は0〜4の目安で4分類すべて埋める(栄養チェックのため)

### 新機能を追加するチェックリスト
- [ ] HTML: 該当タブまたは新モーダルに UI追加
- [ ] CSS: `<style>` 内に必要なクラス追加(CSS変数を使う)
- [ ] 状態変数(`let fooState`)追加
- [ ] `saveState()` に localStorage 書き込み追加
- [ ] `loadState()` に読込追加
- [ ] 初期化ロジック(必要なら `DOMContentLoaded` のコールバック)
- [ ] ダークモード確認(対応済み CSS変数を使えば自動対応)
- [ ] モバイル幅 375px で破綻しないか確認
- [ ] `prefers-reduced-motion` 対応(アニメーションがあれば)

### モーダルを追加する
既存の `.modal` クラスパターンを使う。例:
```html
<div id="foo-modal" class="modal" onclick="if(event.target===this)closeFooModal()">
  <div class="modal-content">
    <button class="modal-close" onclick="closeFooModal()">✕</button>
    <div id="foo-content"></div>
  </div>
</div>
```
JS側:
```js
function openFooModal() { document.getElementById('foo-modal').classList.add('show'); }
function closeFooModal() { document.getElementById('foo-modal').classList.remove('show'); }
```

### アフィリエイトリンクを追加する
- `getAffiliateLinks(name)` - 楽天/Amazon検索URL配列を返す(設定無効なら `null`)
- `renderMealKitCard(context)` - `'limit'|'empty'|'shopping'` で宅配提案カード生成
- どちらも `affiliateSettings.disabled` や `mealkitDisabled` をチェック済み
- 新規の提案カードを追加する際は必ず `rel="noopener noreferrer sponsored"` と「PR」表示を入れる

### タブを追加する
1. `<button class="tab-btn" onclick="switchTab('name',this)">` を既存タブボタン群の末尾に
2. `<div id="tab-name" class="tab-content">` を既存タブコンテンツの末尾に
3. 初期表示するには `renderFoo()` を `DOMContentLoaded` に追加

## 7. 並行調理タイムライン(最も複雑な部分)

- リソース: `手作業`/`コンロ1`/`コンロ2`/`オーブン`/`レンジ`/`炊飯器`/`ホットクック`
- `scheduleStepsV3` がリソース競合を避けてステップを配置
- `passive: true` のステップは料理人稼働率に含めない(放置時間)
- 下ごしらえ(`isPreStep: true`)は煮込み等の放置時間に挿入される

ガント描画は `renderGantt()`、ステップリストは `renderStepList()`、NOWモードは `renderNowMode()`。3ビューを `toggleTimelineView(view, btn)` で切替。

表示モードの切替時もタイムラインデータ(`cookingTimeline`)は保持される。データは `generateTimeline()` で生成。

## 8. 収益化の構造(現状)

**モデル**: 基本無料 + アフィリエイト(主婦層+発達当事者両対応)

### 実装済みの収益ポイント
1. レシピ材料に楽天/Amazon検索リンク(文脈関連性◎)
2. 買い物リストに楽天/Amazon検索リンク
3. 「がんばらない日」モード時のナッシュ等宅配提案カード
4. 献立が空の日の提案カード
5. 買い物リスト下部の宅配提案

### 避けるべき広告パターン
- 美容系(文脈外れ)
- 療育教材(レッテル感)
- 「ADHD向け」明示サプリ
- ポップアップ・追従・巨大バナー

### ターゲット層
- 第1: 発達障害(ADHD/ASD)当事者および家族
- 第2: 共働き時短ニーズ主婦層
- 共通の刺さるもの: 冷凍食品/ミールキット/宅配弁当/時短家電/便利グッズ

## 9. デプロイとブランチ戦略

- 現在の作業ブランチ: `add-project-config`
- デプロイはこのブランチから Vercel が自動 build
- コミットメッセージは日本語OK、`Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>` を付ける
- ユーザーが明示しない限り force push・hook skip しない

## 10. 開発のときに Claude がやるべきこと

1. **このスキルを最初に読む** — ファイル全体を読む前にまずここ
2. **変更箇所だけ `Grep` で特定** — 全ファイル読みは避ける
3. **小さくコミット** — 機能単位で分ける
4. **ユーザーに確認を取る** — 大きな変更・新機能は事前にプランを提示
5. **既存パターンを踏襲** — 独自の新パターンを勝手に導入しない
6. **プレビューパネル確認** — Edit tool 使用後はプレビューに表示される旨を伝える
7. **コンセプトとの整合性を確認** — 認知負荷を上げていないか、煽っていないか

## 11. 参考ファイル

- `.claude/plan.md` - v3の初期設計プラン(古い部分あり、参考程度)
- `.claude/launch.json` - ローカル起動設定
- `.claude/settings.local.json` - プロジェクトローカル設定
- `.gitignore` - 基本設定

## 12. アンチパターン集(過去にやりそうになったこと)

- ❌ `title="..."` 属性だけに頼ったツールチップ(モバイル非対応・遅延あり) → モーダル推奨
- ❌ `'-20分'` のような誤解を招く記号使い → `▲` 等の文脈明確な記号を
- ❌ Canvas 描画でラベル固定`textAlign:'center'` → 角度に応じて `left`/`right` 動的切替
- ❌ 「X が不足しています」だけの警告 → 具体的な補食提案まで含める
- ❌ 週全体のみの買い物リスト → 今日/明日/3日分等のスコープ選択を
