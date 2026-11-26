# VOTRUST - 改ざんできない透明キャラ投票

**Solana Devnet上に直接記録される、運営も操作不可能なリアルタイム投票**

VOTRUST は、3 体のキャラクターから推しを選び、**0.01 SOL（Devnet）を送金することで 1 票を投じる**投票アプリです。
`index.html` 1 ファイルだけで動作し、外部サーバー・データベース・スマートコントラクトは一切使いません。
投票はすべて Solana Devnet 上の通常の送金トランザクションとして記録され、集計もブロックチェーン上の残高から直接計算されます。

> ⚠ **Devnet 専用です。** 接続先 RPC の genesis hash を照合し、Devnet 以外（Mainnet / Testnet など）のエンドポイントには接続しません。使うのは無価値なテスト用 SOL だけです。

---

## しくみ（なぜ改ざんできないのか）

| 項目 | 内容 |
| --- | --- |
| 投票 | 各キャラに割り当てた **投票先アドレス** へ 0.01 SOL を送金 = 1 票 |
| 集計 | 投票先アドレスの現在残高を Devnet から直接取得し、`残高 ÷ 0.01 SOL`（小数点以下切り捨て）を得票数として表示 |
| 記録 | 投票はすべてオンチェーンのトランザクション。アプリ側にデータベースがないため、運営が票を書き換える場所がありません |
| 検証 | 各キャラの「Explorerで検証」リンクから、誰でも Solana Explorer（Devnet）で残高と全取引を直接確認できます |

> 📝 **公平性についての注意**
> - 得票数は「残高」から計算するため、0.01 SOL 以外の金額（例: 1 SOL = 100 票）や、このアプリを経由しない送金も得票として数えられます。
> - 投票先アドレスの**秘密鍵の持ち主は SOL を引き出せる**ため、その分の票が減ってしまいます。本番運用で「誰にも操作できない」状態にするには、秘密鍵を誰も持たないアドレス（生成直後に秘密鍵を破棄したアドレスなど）を投票先にしてください。

---

## 主な機能

| 機能 | 内容 |
| --- | --- |
| ウォレット接続 | ヘッダーの「Phantom / ウォレット接続」から Phantom / Solflare / Backpack などの拡張ウォレット、またはブラウザ内で生成する「ローカルテストウォレット」に接続。接続中はヘッダーにアドレスと残高を表示 |
| キャラクター投票 | 3 体のキャラをカードで表示し、各カードの「0.01 SOLで投票する」ボタンで投票。現在 1 位のキャラには 👑 が付きます |
| リアルタイム集計 | 得票数・得票率をアニメーション付きの棒グラフで表示。「最新結果を取得」ボタンのほか、投票完了時、および投票先アドレスの残高変化（WebSocket 購読）で自動更新 |
| 投票処理フローの可視化 | **構築 → 署名 → 送信 → Processed → Confirmed → Finalized** の各段階を、経過時間付きのステップ表示とログでリアルタイムに表示 |
| 限定デジタル応援カード | 投票が Confirmed になると、キャラのアイコン・名前・お礼メッセージ、「公式オンチェーン投票証明」バッジ、投票者アドレス、投票日時、トランザクション署名（コピー可）、Explorer リンクを載せたカードを紙吹雪付きで表示。Finalized になると状態表示も更新されます |
| 投票・送受信履歴 | 接続中ウォレットのオンチェーン履歴（日時・種別・送信元・送信先・数量・状態）を表示。投票先アドレスへの送金は「🗳️ 投票: キャラ名」と表示され、「投票」タブで絞り込めます |
| 安全対策 | レント免除額チェック、公開 RPC の 429（レート制限）に対する指数バックオフ付きリトライ、確定済みの投票が一時的な通信エラーで「失敗」と表示されない処理 |

### レート制限（429）対策
- 3 キャラの残高は `getMultipleAccountsInfo` で **1 リクエストにまとめて**取得しています。
- 投票先アドレスの残高変化は WebSocket（`accountSubscribe`）で受け取るため、HTTP リクエストはほとんど増えません。取りこぼし対策の再集計は 30 秒ごと（タブ表示中のみ）です。
- 履歴の詳細は少数ずつ間隔を空けて取得し、失敗した RPC 呼び出しは時間を空けてリトライします。

### 使用ライブラリ（CDN から読み込み）
- [`@solana/web3.js` 1.95.3](https://www.npmjs.com/package/@solana/web3.js)（unpkg）
- [`buffer` 6.0.3](https://www.npmjs.com/package/buffer)（jsDelivr / ブラウザ用 Buffer ポリフィル）

画像ファイルは使っておらず、キャラクターは絵文字と CSS だけで描いています。

---

## 遊び方

1. 下記「ローカルでの起動方法」のいずれかでアプリを開く
2. ウォレットのネットワークを **Devnet** に切り替える
   - Phantom: 設定 → 開発者設定 → **テストネットモード** を ON → Solana のネットワークを **Devnet** に
   - Solflare: 設定 → ネットワーク → **Devnet**
3. ヘッダーの「👛 Phantom / ウォレット接続」を押して、使うウォレットを選ぶ
4. テスト用 SOL を入手する
   - <https://faucet.solana.com/> に接続中のアドレスを入力して取得（着金は自動で画面に反映されます）
   - ※ 公開 Devnet RPC の `requestAirdrop` は閉鎖・レート制限されていることが多いため、「マイウォレット」欄の「エアドロップを試す」は失敗する場合があります
5. 推しキャラのカードで「🗳️ 0.01 SOLで投票する」を押し、ウォレットで署名を承認する
6. 「投票処理フロー」で進行状況を確認。Confirmed になると **限定デジタル応援カード** が表示されます
7. 集計グラフで票が増えたことを確認し、「Explorerで検証」から Solana Explorer 上でも直接確かめてみましょう
8. 画面下部の「投票・送受信履歴」で、自分の投票記録（署名・状態）を確認できます

> 💡 拡張ウォレットがない場合は「ローカルテストウォレット」を利用できます。秘密鍵はブラウザの `localStorage` に**平文で**保存されるため、**Devnet のテスト以外には絶対に使用しないでください。**

> 💡 ウォレットでは**署名だけ**を行い、署名済みトランザクションはアプリが Devnet RPC へ直接送信します。そのため、ウォレット側のネットワーク設定にかかわらず Devnet にしか送信されません（ただし残高表示やシミュレーションを正しく行うため、ウォレット側も Devnet に設定してください）。

---

## キャラクターのカスタマイズ

`index.html` の `<script>` の先頭にある **`CHARACTERS` 配列**を書き換えるだけで、名前・アイコン・色・説明・お礼メッセージ・投票先アドレスを変更できます。

```js
const CHARACTERS = [
  {
    id: 'soranyan',
    name: 'ソラにゃん（仮）',
    icon: '🐱',
    color: '#a35bff',   // テーマカラー
    description: 'スピード重視の元気な看板猫',
    thanks: '投票ありがとうにゃ！キミの1票はブロックチェーンにずっと残るよ！',
    address: '8jDVrbgUF2V2h6KZLPfxEMm1jJg9QkXfFNQyTjRuqtVU', // 投票先アドレス
  },
  // ...
];
const VOTE_SOL = 0.01; // 1 票あたりの送金額 (SOL)
```

| キャラ | アイコン | テーマカラー | 投票先アドレス |
| --- | --- | --- | --- |
| ソラにゃん（仮） | 🐱 | 紫 | `8jDVrbgUF2V2h6KZLPfxEMm1jJg9QkXfFNQyTjRuqtVU` |
| チェイン犬（仮） | 🐶 | 青 | `6XzScszLJyQW18D7i5kbc4Xg2YgaN7rHfk1irL3vFN2Z` |
| ブロックま（仮） | 🐻 | 緑 | `EdyNSxnnfQorivmyqgeBQdYHPToFcepri9ezLXcaNPDP` |

- アドレスが Base58 の公開鍵として正しくない場合、そのキャラのカードには警告が出て投票ボタンが無効になります。
- 新しいアドレスを使うと、最初の得票は 0 票から始まります（既に残高があるアドレスは、その残高も得票として数えられます）。

---

## ローカルでの起動方法

ブラウザ拡張ウォレットは `file://` で開いたページには接続できません。**必ずローカル HTTP サーバー経由で開いてください。**
以下はどちらか一方で OK です。

### A. Node.js で起動

#### 1. Node.js がインストールされているか確認

```bash
node -v
npm -v
```

バージョンが表示されれば 2. へ進みます。`command not found` などと表示される場合は、以下でインストールしてください（LTS 版を推奨）。

<details>
<summary><b>Node.js のインストール手順</b></summary>

- **Windows**
  - 公式インストーラー: <https://nodejs.org/ja> から「LTS」をダウンロードして実行
  - または PowerShell で winget を使用:
    ```powershell
    winget install OpenJS.NodeJS.LTS
    ```
- **macOS**
  - 公式インストーラー: <https://nodejs.org/ja>
  - または Homebrew を使用:
    ```bash
    brew install node
    ```
- **Linux (Ubuntu / Debian)**
  ```bash
  curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
  sudo apt-get install -y nodejs
  ```

インストール後、**ターミナルを開き直して** `node -v` で確認してください。
</details>

#### 2. サーバーを起動

```bash
cd solana_test
npx serve -l 8080 .
```

初回は `Need to install the following packages: serve ... Ok to proceed? (y)` と聞かれるので `y` を入力します。
（別の方法: `npx http-server -p 8080` でも可）

#### 3. ブラウザで開く

<http://localhost:8080/> （`serve` の場合）または <http://localhost:8080/index.html>

停止するにはターミナルで `Ctrl + C` を押します。

---

### B. Python で起動

#### 1. Python がインストールされているか確認

```bash
python --version     # Windows
python3 --version    # macOS / Linux
```

`Python 3.x.x` と表示されれば 2. へ進みます。表示されない場合は以下でインストールしてください。

<details>
<summary><b>Python のインストール手順</b></summary>

- **Windows**
  - 公式インストーラー: <https://www.python.org/downloads/> からダウンロードし、インストール時に **「Add python.exe to PATH」にチェック** を入れて実行
  - または PowerShell で winget を使用:
    ```powershell
    winget install Python.Python.3.12
    ```
  - ※ `python` と入力して Microsoft Store が開く場合は、上記の方法でインストールするか、「設定 → アプリ → アプリ実行エイリアス」で `python.exe` のエイリアスを OFF にしてください
- **macOS**
  ```bash
  brew install python
  ```
- **Linux (Ubuntu / Debian)**
  ```bash
  sudo apt update
  sudo apt install -y python3
  ```

インストール後、**ターミナルを開き直して** バージョンを確認してください。
</details>

#### 2. サーバーを起動

```bash
cd solana_test
python -m http.server 8000     # Windows
python3 -m http.server 8000    # macOS / Linux
```

> Windows で `python` が見つからない場合は、Python ランチャーを使って `py -m http.server 8000` でも起動できます。

#### 3. ブラウザで開く

<http://localhost:8000/>

停止するにはターミナルで `Ctrl + C` を押します。

---

## GitHub Pages での公開手順

`index.html` 1 ファイルのみで動作するため、そのまま GitHub Pages で公開できます（HTTPS で配信されるため拡張ウォレットも利用可能です）。

### 1. GitHub にリポジトリを作成

1. <https://github.com/new> を開く
2. Repository name に任意の名前（例: `VOTRUST`）を入力
3. **Public** を選択（無料プランで GitHub Pages を使う場合は Public が必要）
4. 「Create repository」をクリック

### 2. ファイルをプッシュ

Git がない場合は <https://git-scm.com/downloads> からインストールしてください（Windows は `winget install Git.Git` でも可）。

```bash
cd solana_test
git init
git add index.html README.md
git commit -m "Add VOTRUST (Solana Devnet character voting app)"
git branch -M main
git remote add origin https://github.com/<ユーザー名>/<リポジトリ名>.git
git push -u origin main
```

> Git を使わない場合は、作成したリポジトリの画面で「uploading an existing file」から `index.html` と `README.md` をドラッグ＆ドロップしてコミットしても構いません。

### 3. GitHub Pages を有効化

1. リポジトリの **Settings** → 左メニューの **Pages** を開く
2. **Build and deployment** の **Source** で **Deploy from a branch** を選択
3. **Branch** で `main`、フォルダで `/ (root)` を選び **Save**
4. 1〜2 分待つと、ページ上部に公開 URL が表示されます

```
https://<ユーザー名>.github.io/<リポジトリ名>/
```

デプロイの進行状況はリポジトリの **Actions** タブ（`pages build and deployment`）で確認できます。
以降は `main` ブランチに push するたびに自動で再公開されます。

---

## トラブルシューティング

| 症状 | 対処 |
| --- | --- |
| ウォレットが一覧に出ない | `file://` ではなく `http://localhost` / `https://` で開いているか確認。拡張機能を有効にしてページを再読み込み |
| 「Devnet に接続できません」 | ネットワーク接続を確認。「マイウォレット」→「RPC 設定」で別の Devnet RPC（例: Helius / QuickNode などの Devnet エンドポイント）を指定 |
| 集計・履歴の読み込みが遅い・`429` エラー | 公開 Devnet RPC はレート制限が厳しいため、自動でリトライしています。時間をおくか、専用の Devnet RPC を設定してください |
| 投票ボタンが押せない（残高不足） | 1 票に 0.01 SOL + 手数料が必要です。<https://faucet.solana.com/> でテスト用 SOL を取得してください |
| 署名時にウォレットで警告・残高 0 と表示 | ウォレット側のネットワークが Devnet になっているか確認 |
| 「レント免除額」に関するエラー | Solana では、残高が 0 より大きく約 0.00089 SOL（レント免除額）未満のアカウントは存在できません。投票後の残高がこの範囲になる場合は、faucet で SOL を追加してから投票してください |
| 「blockhash の有効期限が切れました」 | 署名の承認に時間がかかりすぎた場合に発生します。もう一度投票してください |
| 応援カードを閉じてしまった | 「投票・送受信履歴」の Tx リンク、または「投票処理フロー」の署名リンクから Explorer で投票記録を確認できます |
