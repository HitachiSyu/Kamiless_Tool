# 個人ポートフォリオ & 技術ブログ

シンプルで美しい、純粋な HTML/CSS/JS で構築された静的ポートフォリオサイトです。AWS S3 にそのままアップロードしてホスティングできます。

## ディレクトリ構成

```
personal-site/
├── index.html          # メインページ
├── css/                # スタイルシート
│   ├── reset.css
│   ├── variables.css   # テーマ変数
│   ├── base.css        # 基本スタイル
│   ├── layout.css      # レイアウト
│   ├── components.css  # コンポーネント
│   ├── animations.css  # アニメーション
│   └── responsive.css  # レスポンシブ
├── js/                 # JavaScript
│   ├── theme.js        # ダーク/ライトモード
│   ├── data.js         # データ読み込み
│   ├── renderer.js     # レンダリング
│   ├── navigation.js   # ナビゲーション
│   └── main.js         # メインエントリ
├── data/
│   └── profile.json    # 全テキストデータ
├── images/
│   └── avatar.svg      # プロフィール画像（.jpg/.png/.webp も可）
└── README.md
```

## コンテンツの編集方法

### 全テキスト：data/profile.json

すべてのテキスト情報は `data/profile.json` で管理されます。

| 項目 | JSON パス |
|-----|----------|
| 名前 | `profile.name` |
| 肩書き | `profile.title` |
| 自己紹介文 | `about.content` |
| プロジェクト | `projects[]` |
| 資格 | `certifications[]` |
| ブログ記事 | `blog[]` |

### アバター画像

`images/avatar.svg` を自分の画像で置き換えてください。
対応フォーマット: `.jpg` `.png` `.webp` `.svg`

画像を変更した場合は `data/profile.json` の `profile.avatar` も更新してください。

```json
{
  "profile": {
    "avatar": "images/your-photo.png"
  }
}
```

## ローカルで確認

### 方法1: ファイルを直接開く
```bash
# Mac
open index.html

# Windows
start index.html
```

### 方法2: ローカルサーバー（推奨）
```bash
# Python
python -m http.server 8080

# Node.js (npx)
npx serve .

# PHP
php -S localhost:8080
```

ブラウザで `http://localhost:8080` を開きます。

## AWS S3 へのデプロイ

### 1. S3 バケットを作成
- バケット名: `your-domain.com` など
- リージョン: お好みのリージョン

### 2. 静的ウェブサイトホスティングを有効化
```
プロパティ → 静的ウェブサイトホスティング → 編集
- 有効化: On
- インデックスドキュメント: index.html
- エラードキュメント: index.html
```

### 3. バケットポリシーを設定
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "PublicReadGetObject",
    "Effect": "Allow",
    "Principal": "*",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::your-bucket-name/*"
  }]
}
```

### 4. ファイルをアップロード
```bash
# AWS CLI を使用
aws s3 sync . s3://your-bucket-name --exclude "README.md" --exclude ".git/*"
```

または AWS コンソールからドラッグ&ドロップでアップロード。

### 5. CloudFront でカスタムドメイン + HTTPS（オプション）
- CloudFront ディストリビューションを作成
- オリジン: S3 バケット
- ACM で SSL 証明書を発行
- Route 53 でカスタムドメインを設定

## テーマ（ダーク/ライトモード）

- 右上の 🌙/☀️ アイコンで切り替え
- 選択はブラウザの localStorage に保存されます
- 初回アクセス時はシステムの設定を自動検出

## カスタマイズ

### アクセントカラーの変更
`css/variables.css` の `--accent` と `--accent-hover` を変更:

```css
:root {
  --accent: #3b82f6;      /* デフォルト: blue */
  --accent-hover: #2563eb;
}

[data-theme="dark"] {
  --accent: #818cf8;      /* ダーク時: indigo */
  --accent-hover: #a5b4fc;
}
```

### フォントの変更
`css/base.css` の `@import` と `--font-sans` を変更:

```css
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+JP:wght@300;400;500;700&display=swap');

body {
  font-family: 'Noto Sans JP', sans-serif;
}
```

## ブログ記事の Markdown 記法

`data/profile.json` の `blog[].content` では以下の記法が使用できます:

| 記法 | 出力 |
|-----|-----|
| `# 見出し` | <h3>見出し</h3> |
| `**太字**` | <strong>太字</strong> |
| `*斜体*` | <em>斜体</em> |
| `` `コード` `` | <code>コード</code> |
| `[リンク](URL)` | <a href="URL">リンク</a> |
| 空行 | 段落区切り |

## 注意事項

- `file://` プロトコルでは JSON の fetch に制限がある場合があります。ローカル確認には HTTP サーバーの使用を推奨します。
- S3 にアップロードする際は、ファイルの Content-Type が正しく設定されていることを確認してください。
- ブログの Markdown レンダリングは簡易的なものです。複雑な表現が必要な場合は HTML を直接記述してください。
