# ヨセミテ英語クエスト（yosemite-quiz）

移動中に家族（Toshiya / Natsuko / Yume / Yuki）で **最高スコアを競う** 英語クイズアプリ。
目標は **Yosemite 現地の掲示・標識・パンフレットを読破できる語彙力**。

- スマホHTML（単一 `index.html`・自己完結）
- 電波あり前提（自宅↔ホテル）＋ **圏外でもプレー可**。圏外中はトップ5スコアを端末保持し、接続時にランキング反映
- ランキングは **Google スプレッドシート（GAS Web App 裏持ち）**

公開URL（予定）: `https://superduperohiolife-lgtm.github.io/yosemite-quiz/`

---

## 遊び方

1. 自分の **名前**（Toshiya / Natsuko / Yume / Yuki）を入力＝あいことば（パスコード）
2. 「クエストに でる」で開始。**1回20問**
3. 画面の英文中、**下線の語・表現の意味**を選択肢から選ぶ
4. 回答と同時に **◯／✕** と解説を表示（連続正解で高得点）
5. 20問終了で結果＆称号。オンラインなら総合ランキングに反映

### 出題ルール（4ステップ・アダプティブ）

- 20問＝ **5・5・5・5** の4ステップ
- **ステップ1は必ず Lv1**。以降は直前ステップの正答率で難易度を見直す
  - 正答率が高い（**5問中3問以上＝60%以上**）→ レベルを1つ上げる（**1→2→3→3**・上限3）
  - 正答率が低い → **現状維持**（下げない。低いままなら Lv1 を維持）
- 難易度帯：**Lv1＝英検2級 / Lv2＝準1級 / Lv3＝1級・ネイティブ表現（掲示のリアル語）**
- 選択肢は運で当たらないよう **Lv連動で5〜7択**
- 各問の先頭に **出題カテゴリ（地形／名所／歴史／自然）** を色つきラベルで表示
- スコアは **難易度で重み付け**（難問ほど高得点）。子供と大人が同じ総合ランキングで公平に競える設計。満点の目安は約5700点

---

## 問題の追加方法

語彙データは **`data.js`（`window.QUIZ_DATA`）** に分離。同じ形で1行追加するだけ。

```js
{ s:"Bears *forage* for acorns, berries, and insects.", // *で囲った語=下線ターゲット
  m:"（餌を）あさる・探し回る",  // 正解の意味（日本語）
  kana:"フォリッジ",           // カタカナ発音（ヒント表示用）
  cat:"nat",                  // 分類（下記4種）
  lv:2,                       // 1=2級 / 2=準1級 / 3=1級・ネイティブ
  d:["冬眠する","縄張りを守る","獲物を追う"], // 近い誤答（任意）
  note:"NPS頻出語" },          // 一言解説（任意）
```

- 下線は英文中の `*...*`（**1文に1か所**）。単語でもイディオムでも可
- `cat`：`geo`(地形/地質) `spot`(名所/景観) `hist`(歴史/人物) `nat`(自然/動植物/生態)
- **禁則**：選択肢（`m`・`d`）に**ラテン文字**や**対象語のカタカナ読み**を入れない（答えが直結してしまうため）
- 選択肢のダミーは `d`→同カテゴリ→他カテゴリの順で自動補完。`m` は他と重複しない表現にする

現在の収録数：**500語**（geo107 / spot183 / hist51 / nat159｜Lv1=92 / Lv2=243 / Lv3=165）
※ 出典・題材は NPS Yosemite（nps.gov/yose）。**Places To Go**（各名所ページ）を主体に、地形・自然・景観語を厚く収録。歴史は基本語のみ

---

## ランキング（GAS Web App）のセットアップ

1. `drive.google.com` で新規スプレッドシート作成（例：`yosemite-quiz-ranking`）
2. 拡張機能 → **Apps Script** を開き、`gas/Code.gs` の内容を貼り付け
3. `Code.gs` の `SHEET_ID` にスプレッドシートのID（URLの `/d/★ID★/edit`）を設定
4. **デプロイ → 新しいデプロイ → ウェブアプリ**
   - 実行するユーザー：**自分**
   - アクセスできるユーザー：**全員（Anyone）**
5. 発行された **`/exec` URL** を `index.html` 冒頭の `RANKING_ENDPOINT` に貼る

```js
const RANKING_ENDPOINT = "https://script.google.com/macros/s/XXXX/exec";
```

- 空のままでも **オフライン（端末内ベストのみ）** で動作する
- APIキーは不要。クライアントに秘密情報は置かない方針
- Sheet列：`ts | date | player | score | title | correct | total | accuracy | level`（初回自動生成）

---

## 公開（GitHub Pages）

- リポジトリ：**public**、名称 `yosemite-quiz`、アカウント `superduperohiolife-lgtm`
- `index.html` は **リポジトリ直下** に配置
- `main` へ push すると `.github/workflows/deploy.yml`（GitHub Actions）で **Pages 自動デプロイ**
- 初回のみ GitHub の **Settings → Pages → Build and deployment → Source: GitHub Actions** を選択

### 想定フォルダ構成

```
yosemite-quiz/
├─ index.html                     ← アプリ本体（これがPagesで公開される）
├─ README.md
├─ gas/
│  └─ Code.gs                     ← ランキング用 GAS（Pagesには不要）
└─ .github/
   └─ workflows/
      └─ deploy.yml               ← Pages 自動デプロイ
```

---

## 技術メモ

- 文字コード **UTF-8** / 改行 **LF**
- 外部CDN・外部ライブラリなし（オフライン完全動作）。効果音は WebAudio 内蔵生成、発音は Web Speech API があれば使用（無ければ🔊非表示）
- スコア・ベスト・未送信分は `localStorage` 保持。圏外の未送信は**高得点トップ5**のみ保持し、接続時に自動再送
- ランキング通信は `fetch`（POSTは `text/plain` 既定でプリフォフライト回避、GAS側は `e.postData.contents` を読む）
