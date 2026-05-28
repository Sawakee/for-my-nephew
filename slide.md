---
marp: true
theme: default
paginate: true
backgroundColor: "#1a0a2e"
color: white
style: |
  section {
    font-family: 'Segoe UI', 'Noto Sans JP', sans-serif;
    background: linear-gradient(135deg, #1a0a2e 0%, #0a1a2e 100%);
    color: white;
  }
  h1 {
    background: linear-gradient(90deg, #ff6b6b, #ffd93d, #6bcb77, #74c0fc, #c77dff);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
    font-size: 2rem;
  }
  h2 { color: #ffd93d; border-bottom: 2px solid #ffd93d33; padding-bottom: 0.3rem; }
  h3 { color: #6bcb77; }
  code { background: rgba(255,255,255,0.1); padding: 0.1em 0.4em; border-radius: 4px; color: #ffd93d; }
  pre { background: rgba(255,255,255,0.08); border: 1px solid rgba(255,255,255,0.15); border-radius: 8px; }
  a { color: #74c0fc; }
  ul li { margin: 0.3em 0; }
  strong { color: #ffd93d; }
  table { border-collapse: collapse; width: 100%; }
  th { background: rgba(255,255,255,0.15); padding: 0.4em 0.8em; }
  td { border: 1px solid rgba(255,255,255,0.1); padding: 0.4em 0.8em; }
---

# 🎮 for-my-nephew
## ブラウザゲーム開発まとめ

Rust製CLIゲーム → ブラウザゲーム移植 & GitHub Pages 公開

🔗 https://sawakee.github.io/for-my-nephew/

---

## 📦 プロジェクト概要

- **目的**: 甥っ子が遊べるブラウザゲームを作って公開する
- **元ネタ**: 3つのRust製CLIゲーム（moji-hanabi / kioku-match / emoji-awase）
- **方針**: 単一HTMLファイル完結・ビルド不要・GitHub Pages で無料公開

```
for-my-nephew/
├── index.html          # ランディングページ
├── moji-hanabi/index.html   # 文字花火ゲーム
├── kioku-match/index.html   # 神経衰弱ゲーム
└── emoji-awase/index.html   # スライドパズル
```

---

## 🎆 もじはなび

**元ゲーム**: キー入力で文字が飛び跳ねるパーティクルゲーム

| 機能 | 実装 |
|------|------|
| 描画 | Canvas API + requestAnimationFrame |
| 物理 | 重力・壁バウンド（減衰0.75）|
| パーティクル | 最大700個、回転・フェード |
| キー | Space=★ / Enter=❤ / BackSpace=全消去 |
| **モバイル** | タップで花火・スワイプで軌跡・ボタンバー |

自動アイドルパーティクルで放置中も画面が動く

---

## 🃏 きおくあわせ

**元ゲーム**: 4×2の絵文字神経衰弱

| 機能 | 実装 |
|------|------|
| 描画 | CSS Grid + CSS 3D Transform |
| アニメ | `perspective(600px) rotateY(180deg)` |
| マッチ | バウンスアニメ + レインボーグロウ |
| ミス | シェイクアニメ + 1.2秒後に伏せる |
| クリア | Canvas 紙吹雪（rect/circle/star）|
| 操作 | 矢印キー + Enter/Space + クリック/タップ |

🐶🐱🐭🐹 の4種×2枚 = 計8枚

---

## 🧩 えもじパズル

**元ゲーム**: 3×3スライドパズル

| 機能 | 実装 |
|------|------|
| 状態 | `board[9]` 配列（0=空白, 1-4=絵文字種）|
| シャッフル | ゴール状態から200手ランダム合法手 |
| ゴール | `[1,1,2,2,3,3,4,4,0]` |
| タイル色 | 🐶=オレンジ 🐱=黄 🐭=青 🐹=ピンク |
| ヒント | 右側にゴール配置を常時表示 |
| クリア | Canvas 紙吹雪 |

シャッフルは合法手のみなので必ず解ける

---

## 🎨 ビジュアル強化（funsy化）

### ランディングページ
- レインボーアニメーション背景（`background-position` 移動）
- 空を漂う絵文字 ⭐🌈✨🎉（JS で動的生成・CSSアニメ）
- カードがグラデーション化 + ホバーで虹色グロウボーダー
- タイトルがバウンス・カード絵文字がゆらゆら揺れる

### 各ゲーム共通
- アニメーション背景グラデーション
- タイトルにレインボーカラーサイクル
- クリア時に Canvas 紙吹雪

---

## 📱 モバイル対応

### もじはなびのモバイル問題
- キーボード前提のゲームなので **タッチ操作が成立しない**
- 解決策: **3つのモバイルUI**を追加
  1. **タップ** → タップした場所に花火が出る
  2. **スワイプ** → 指の軌跡に粒子が流れる
  3. **ボタンバー** → 画面下部に ★ ❤ ✿ ♪ けす ボタン

### 全ゲーム共通
- `@media (pointer: coarse)` でタッチデバイス向けヒントに切り替え
- えもじパズルは 500px 以下で縦スタック配置

---

## 🐛 バグ修正: カードが下にずれる

### 現象
きおくあわせでカードを選択すると **一段下に視覚的にズレる**

### 原因
`perspective: 800px` を **親要素**（`.card-wrap`）に設定していたため、
グリッド内のカードの位置によって消失点がズレ、3D回転時に視覚的なシフトが発生

### 修正
```css
/* Before: 親に perspective → グリッド位置依存 */
.card-wrap { perspective: 800px; }
.card-wrap.face-up .card-inner { transform: rotateY(180deg); }

/* After: transform に perspective → 各カードが自分の中心で回転 */
.card-wrap.face-up .card-inner {
  transform: perspective(600px) rotateY(180deg);
}
```

---

## 🐛 バグ修正: render の layout shift

### 現象
カード操作のたびにページ全体が一瞬ずれる

### 原因
`render()` が毎回 `board.innerHTML = ''` で全カードを破棄→再生成していた
→ 一瞬ボードの高さが 0 になり `justify-content: center` が再計算

### 修正
```javascript
function render(shakeIdxs) {
  const existing = board.querySelectorAll('.card-wrap');
  if (existing.length === 8) {
    // クラスのみ更新 → DOM破棄なし → layout shift なし
    existing.forEach((w, i) => { w.className = cardClass(i, shakeIdxs); });
  } else {
    // 初回のみフル生成
    ...
  }
}
```

---

## 🚀 デプロイ

```bash
# 1. Git リポジトリ初期化 & コミット
git init && git add . && git commit -m "Add browser games"

# 2. GitHub パブリックリポジトリ作成 & プッシュ（一発）
gh repo create for-my-nephew --public --source=. --remote=origin --push

# 3. GitHub Pages 有効化
gh api repos/Sawakee/for-my-nephew/pages \
  --method POST \
  -f "source[branch]=main" \
  -f "source[path]=/"
```

- **リポジトリ**: https://github.com/Sawakee/for-my-nephew
- **公開URL**: https://sawakee.github.io/for-my-nephew/

---

## 📝 技術まとめ

| 項目 | 選択 | 理由 |
|------|------|------|
| フレームワーク | なし（バニラJS）| 子供向け小規模・ビルド不要 |
| ファイル構成 | 単一HTML完結 | GitHub Pages で即動作 |
| アニメ | CSS transform + requestAnimationFrame | 依存ゼロ |
| 3Dカードフリップ | `perspective()` in transform | グリッド位置非依存 |
| モバイル判定 | `@media (pointer: coarse)` + `navigator.maxTouchPoints` | タッチデバイス判定 |
| シャッフル | ゴール状態から逆算合法手 | 必ず解ける保証 |

---

## ✅ 完了 / 🔲 次のタスク

### 完了
- ✅ 3ゲーム + ランディングページ実装
- ✅ GitHub Pages 公開
- ✅ ビジュアル強化（funsy化）
- ✅ モバイル対応
- ✅ バグ修正（layout shift / カードずれ）

### 次にやること
- 🔲 実機（iPhone/Android）での動作確認
- 🔲 きおくあわせの難易度追加（4×4 = 8種16枚）
- 🔲 えもじパズルのヒント機能・星評価
- 🔲 BGMや効果音の追加（Web Audio API）
