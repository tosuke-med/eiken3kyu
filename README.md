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
├── data/                          # 問題データのみを保管するJSファイル群
│   ├── vol01_narabi_meishi.js
│   ├── vol02_choice_doushi.js
│   └── ...
│
├── audio/                         # 音声ファイル（mp3）
│   ├── n001.mp3
│   └── v006.mp3
│
├── images/                        # 画像ファイル
│   └── cat.png
│
├── published/                     # テンプレ + data を合体した配布用HTML
│   ├── quiz_vol01.html
│   └── quiz_vol02.html
│
└── README.md
```

---

## テンプレートの構造

`quiz_3kyuu_template.html` は **ZONE A（データ）** と **ZONE B（エンジン）** に分かれている。

- **ZONE A**：問題データ（`QUIZ_META` と `questions`）を定義する。ここだけを編集する。
- **ZONE B**：UIとJSエンジン。触らない。

```html
<!-- ZONE A: ここだけ編集する -->
<script>
const QUIZ_META = { ... };
const questions = [ ... ];
</script>

<!-- UI HTML -->

<!-- ZONE B: 触らない -->
<script>
  // エンジンロジック
</script>
```

---

## dataファイルの仕様

各 `.js` ファイルには **ZONE A の中身のみ**（`QUIZ_META` と `questions`）を保存する。

```javascript
// vol01_narabi_meishi.js
// カテゴリ: ならびかえ／名詞トピック
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

## published HTMLの作り方

1. `template/quiz_3kyuu_template.html` をコピーして `published/quiz_vol○○.html` にリネーム
2. ZONE A のサンプルデータを削除
3. 対応する `data/vol○○_xxx.js` の中身を貼り付けて保存

---

## 問題のtype

### choice（4択）

空欄に入る語句を4択から選ぶ。

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

単語タイルをタップして正しい語順に並べる。

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
- 単語数は4〜7語が推奨（8語以上は避ける）
- `words` 配列はあらかじめシャッフルした状態で記述する

---

## IDの命名規則

カテゴリごとにプレフィックスを持ち、3桁の連番を付ける。**カテゴリ内で通し番号を管理する。**

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

**IDはリポジトリ全体で重複させない。**
各カテゴリの現在の最大番号を把握し、続きの番号から採番すること。

---

## LLMへの問題生成指示テンプレ

以下のフォーマットで指示することで、保管用データとインタラクティブクイズの両方を生成できる。

```
【カテゴリ】動詞
【テーマ】学校
【type】choice
【問題数】5問
【IDの開始番号】v006から

① 保管用コード（ZONE A貼り付け用）
② アーティファクトで遊べるクイズ
の両方を出して。
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
- **英検の過去問の流用禁止**。文・文脈はすべてオリジナルで作成する
- 「英検3級レベル相当」という難易度の目安表現はOK
- `jp_hint` はひらがなを基本とする（小学生以下が読める水準）

---

## ローカルでの動作確認

**音声・画像なし：** `published/quiz_vol○○.html` をブラウザで直接開く。

**音声・画像あり：** CORSの制限があるためローカルサーバーが必要。

```bash
# Python
cd eiken3-quiz
python -m http.server 8000
# → http://localhost:8000/published/quiz_vol01.html

# Node.js
npx serve .
```

VS Codeの場合は「Live Server」拡張を使うと簡単。