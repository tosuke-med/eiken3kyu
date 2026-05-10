# eiken3-quiz

英検3級レベルを対象にした子ども向け英語クイズ教材の素材リポジトリ。
LLM（Claude等）と協力して問題を生成・蓄積し、HTMLファイルとして配布することを目的とする。

---

## フォルダ構造

```
eiken3-quiz/
├── template/
│   └── quiz_3kyuu_template.html   # ベーステンプレート。原則触らない
│
├── data/                          # カテゴリ別・ID範囲別の問題データ
│   ├── sort_001-005.js
│   ├── sort_006-010.js
│   ├── v_001-005.js
│   ├── n_001-005.js
│   └── phr_001-005.js
│
├── master/
│   └── all_questions.js           # 全問題を結合したマスターデータ（ランダム出題用）
│
├── audio/                         # 音声ファイル（mp3）
│   └── n001.mp3
│
├── images/                        # 画像ファイル
│   └── cat.png
│
├── published/                     # 配布用HTML
│   ├── quiz_sort_001-005.html     # 特定セットの配布用
│   ├── quiz_sort_006-010.html
│   └── quiz_random.html           # masterから毎回ランダム出題する版
│
└── README.md
```

---

## ファイル命名規則

### data/

```
{カテゴリプレフィックス}_{開始ID}-{終了ID}.js
```

例：`sort_001-005.js` `v_006-010.js` `phr_001-005.js`

ファイル名にID範囲を含めることで、**どのファイルにどの問題が入っているかが一目でわかり、IDの重複も防ぎやすい。**

### published/

```
quiz_{カテゴリプレフィックス}_{開始ID}-{終了ID}.html   # 特定セット配布用
quiz_random.html                                        # ランダム出題用（固定名）
```

---

## カテゴリとIDプレフィックス

カテゴリごとに独立した連番を持つ。**リポジトリ全体でIDを重複させない。**

| カテゴリ | プレフィックス | 例 |
|---|---|---|
| 名詞 | `n` | n001, n002 |
| 動詞 | `v` | v001, v002 |
| 形容詞 | `adj` | adj001, adj002 |
| 副詞 | `adv` | adv001, adv002 |
| 熟語・イディオム | `phr` | phr001, phr002 |
| 会話文 | `conv` | conv001, conv002 |
| 文法 | `gr` | gr001, gr002 |
| ならびかえ | `sort` | sort001, sort002 |

新しい問題を追加するときは、そのカテゴリの**現在の最大番号の続き**から採番する。

---

## dataファイルの仕様

各ファイルには `QUIZ_META` と `questions` のみを記述する。

```javascript
// data/sort_001-005.js
// カテゴリ: ならびかえ
// テーマ: 名詞トピック
// 作成日: YYYY-MM-DD
// ID範囲: sort001〜sort005

const QUIZ_META = {
  title: "えいけん3きゅうレベル ならびかえクイズ vol.1",
  level: "3",
  category: "ならびかえ"
};

const questions = [
  { ... },
  { ... }
];
```

---

## masterファイルの仕様

`master/all_questions.js` は全dataファイルの `questions` を結合したもの。
ランダム出題（`quiz_random.html`）はこのファイルだけを参照する。

```javascript
// master/all_questions.js
// ※ このファイルは手動で編集しない。Claudeに追記差分を出力させて貼り付ける。

const ALL_QUESTIONS = [
  // --- sort_001-005.js ---
  { id: "sort001", ... },
  { id: "sort002", ... },

  // --- v_001-005.js ---
  { id: "v001", ... },

  // --- phr_001-005.js ---
  { id: "phr001", ... },
];
```

---

## 問題のtype

### choice（4択）

```javascript
{
  id: "v001",
  type: "choice",        // 省略可（デフォルトがchoice）
  text: "She ( ) to school every day.",
  jp_hint: "かのじょはまいにちがっこうへいきます。",
  image: null,           // 画像パス or null
  audio: null,           // 音声パス or null
  choices: ["walk", "walks", "walked", "walking"],
  answer: "walks"        // choicesのどれかと完全一致
}
```

### sort（ならびかえ）

```javascript
{
  id: "sort001",
  type: "sort",          // 必須
  text: "ならびかえてみよう！",
  jp_hint: "わたしはまいにちがっこうへいきます。",
  words: ["day", "I", "school", "to", "go", "every"],
  answer: "I go to school every day",
  image: null,
  audio: null
}
```

**sortの重要ルール：**
- `answer` は `words` 内の要素をスペースでつないだ文字列と完全一致すること
- 余分なピリオドや大文字の不一致はNG
- 単語数は4〜7語を推奨（8語以上は避ける）
- `words` 配列はあらかじめシャッフルした状態で記述する

---

## テンプレートの構造

`quiz_3kyuu_template.html` は **ZONE A（データ）** と **ZONE B（エンジン）** に分かれている。

- **ZONE A**：`QUIZ_META` と `questions` を定義する。ここだけ編集する。
- **ZONE B**：UIとJSエンジン。触らない。

---

## published HTMLの作り方

**特定セット配布用**
1. `template/quiz_3kyuu_template.html` をコピーして `published/quiz_sort_001-005.html` にリネーム
2. ZONE A のサンプルデータを削除
3. 対応する `data/sort_001-005.js` の中身を貼り付けて保存

**ランダム出題用（quiz_random.html）**
- ZONE A の `questions` を `ALL_QUESTIONS` に差し替え、`master/all_questions.js` を読み込む構成にする
- カテゴリ・typeでフィルタリングする機能を追加することも可能

---

## LLMへの問題生成指示テンプレ

```
【カテゴリ】動詞
【テーマ】学校
【type】choice
【問題数】5問
【IDの開始番号】v006から

① 保管用コード（data/v_006-010.js として保存する用）
② master/all_questions.js への追記差分
③ アーティファクトで遊べるクイズ
の3つを出して。
```

### カテゴリの選択肢
`名詞` `動詞` `形容詞` `副詞` `熟語` `会話文` `文法` `ならびかえ`

### テーマの選択肢
`学校` `家族` `食べ物` `買い物` `旅行` `自然` `趣味` `スポーツ` `町・場所` `日常生活`

### typeの選択肢
`choice`（4択のみ） `sort`（ならびかえのみ） `まじり`（両方混在）

---

## 英検3級レベルのガイドライン

- 中学3年生修了レベル（約1,200〜1,500語）の語彙
- 日常トピックを中心とした平易な文
- **英検の過去問の流用禁止。** 文・文脈はすべてオリジナルで作成する
- 「英検3級レベル相当」という難易度の目安表現はOK
- `jp_hint` はひらがなを基本とする（小学生以下が読める水準）

---

## ローカルでの動作確認

**音声・画像なし：** `published/` の `.html` をブラウザで直接開く。

**音声・画像あり：** CORSの制限があるためローカルサーバーが必要。

```bash
# Python
cd eiken3-quiz
python -m http.server 8000
# → http://localhost:8000/published/quiz_sort_001-005.html

# Node.js
npx serve .
```

VS Codeの場合は「Live Server」拡張を使うと簡単。