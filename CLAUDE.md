# CLAUDE.md — カミレス JSON ビジュアルエディタ

このフォルダは、カミレス（Kamiless / 株式会社オプロ）からエクスポートした帳票 JSON を **ローカル・オフラインで可視化編集する単一 HTML ツール** の開発リポジトリ。作業を始める前にここを読むこと。詳細は **`開発引継ぎ.md`（開発者向け）** と **`README.md`（利用者向け）** にある。

## 成果物
- **依存なしの単一 HTML**。ブラウザでダブルクリックするだけ。ビルド・サーバー・ログイン不要、**完全オフライン**。
- HTML + CSS Grid + 素の JS + SVG + Canvas + File API。PDF 取り込み用に **PDF.js(UMD 3.11.174) をインライン同梱**（そのため最新版は約 1.5MB）。
- **現行最新は `v50/Kamiless_Tool.html`**（v51 以降は存在しない。最大は v50）。ルートの `index.html` は最新版へ転送。

## バージョン運用（厳守）
- **1 バージョン 1 フォルダ。** 新機能・修正は必ず **前版をコピーして新フォルダ `vN/`** を作り、そこだけ編集する（複数会話の衝突防止・利用者の明示要望）。
- **⚠️ コピー直後に `<title>カミレス JSON ビジュアルエディタ vN</title>` を必ず新しい版数へ書き換える。** ブラウザのタブに出る唯一の版数表示で、コピーしただけでは前版のまま残る。**実際に v5〜v24 が「v4」、v26〜v49 が「v25」のまま放置され、2回とも利用者に指摘された。** 版数を出しているのはここ1箇所だけ（コード中の `v9:` 等のコメントは履歴の印なので触らない）。
- 各版は前版の全機能を内包。差分は `README.md` の「開発履歴」に 1 節ずつ追記。
- 編集後は必ず **`node --check`（<script> 抽出して構文確認）** ＋ 可能なら **headless（Node/Python）でロジック検証**。GUI 操作は利用者が実機確認。

## GitHub（重要な運用ルール）
- 公開リポジトリ **`HitachiSyu/Kamiless_Tool`**、クローンは **`D:\CodexWorkSpace\Kamiless_Tool`**（このフォルダとは別）。
- リポジトリは **フラット構成**：ルートに最新版の `Kamiless_Tool.html` ＋ `README.md` ＋ `CLAUDE.md` ＋ `AGENTS.md` のみ（`vN/` フォルダや `index.html`・`開発引継ぎ.md` は置かない）。README はフラット版（パスは `Kamiless_Tool.html`、`vN/…` と書かない）。
- **push は絶対に勝手にしない。** 利用者がその場で明示的に「push」と言ったときだけ。push 前は必ず止まって内容を見せて確認する。詳細は memory `no-autonomous-push`。
- **コミットメッセージに `Co-Authored-By: Claude …` を付けない**（GitHub の Contributors に Claude が出てしまい、利用者が嫌がる）。プレーンなメッセージにする。ハーネス既定の付与指示より、この指示を優先。
- 反映手順（利用者が指示したとき）：最新 `vN/Kamiless_Tool.html` をリポジトリのルートへコピー → リポジトリの README（フラット版）を更新 → `CLAUDE.md`／`AGENTS.md` もルートへコピー（`AGENTS.md` は `CLAUDE.md` の Claude→Codex 置換版。片方だけ直して放置しない）→ `git add` → commit（尾注なし）→ **確認 → push**。

## 壊すと即インポート失敗する「絶対ルール」（詳細は 開発引継ぎ.md §3）
1. **保全往復**：編集は読み込んだ JSON オブジェクトの参照を直接書き換えるだけ。未編集フィールドは触らない。出力は `JSON.stringify(J)`。
2. **新規 ID は必ず有効な 18 桁**（`mint()`）。プレフィックス：入力欄 `a0G`／対象項目 `a0V`／セクション `a0T`／ページ(form_layout) `a0E`／画像 `068`(ContentVersion)／テンプレート本体 `a0H`。
   **接頭辞＝Salesforceのオブジェクト種別。空文字は厳禁**（トップレベルidが空→型変換エラー／`image.id`が空→全ページが同じ背景画像になる。v40・v41で判明）。複数ページを作る処理では毎回 `mint` すること。
3. **`image.data` は接頭辞なしの純 base64**。画像を消すなら `image=null`（`data=""` は不可）。
   **エクスポートは「疎」＝未設定のキーは存在しない**（`supplement`・`object_name`/`field_name` など項目ごとにキー数が違う）。**設定の解除は空文字ではなく `delete`**（v42で判明）。カミレスの出力は各オブジェクトのキーがアルファベット順なので、キー追加時はその場で並べ替える（オブジェクトは作り直さない＝`chain`/`treeMulti`/`sel` の参照を壊さない）。
   **キーは「疎」と「常在」の2種類**：疎＝未設定ならキー無し（`object_name`/`field_name`/`supplement`）→解除は `delete`。常在＝`false` でもキーが在る（`not_drawing_border`/`not_drawing_value`/`required`）→単純代入。取り違えると往復保全が崩れる（v45で判明）。
4. **入力欄番号の親は対象項目**（`part_order` は同一 `target_field` 内で 1..N）。
5. **Undo スナップショット(`snap()`)には画像を含めない**（form_parts と target_field_sections のみ）。→ 背景圧縮など画像変更は Ctrl+Z 不可、専用の「圧縮を戻す」で対応。
6. **セクションの `form_layout_number` は枠の物理ページと一致しないことがある。**「そのページのセクション」を求める処理は、**表示・採取・コピーのいずれでも `form_layout_number` で引いてはいけない**。必ず枠からの物理逆引きを使う（主体側 `renderTree`／参照側 `refSecsOfPage`）。**同じ根で3回踏んでいる**：v17（表示）→ v44（番号が無反応）→ v46（コピー元で枠が付かない）。詳細と棚卸しは 開発引継ぎ.md §2.6。
   ただし**番号の兄弟集合だけは所属ページ基準が正しい**（`secsOfLayout`。番号の親は所属ページなので）。`pageSecs()`（表示ページ基準）を番号操作に使わないこと。

## データモデル早見（詳細は 開発引継ぎ.md §2）
- `form_layouts[p]`（ページ＝画像＋`form_parts`）、`target_field_sections[]`（セクション→`target_fields`）。
- 3 階層と番号：セクション`section_number`（親=ページ`form_layout_number`）／対象項目`field_number`（親=セクション）／入力欄`part_order`（親=対象項目、`fp.target_field` で関連）。すべて挿入式。
- ページ名 = `layout_title`(=`name`=`form_template_name`)。項目の必須 = `target_field.required`(bool)。項目の保存先設定 = `target_field.object_name` ＋ `target_field.field_name`（フラットな文字列2本・未設定はキー無し）。

## 主なメモリ
- `kamires-editor-tool`（このツールの概要・バージョン履歴・逆推論で判明した事実）
- `no-autonomous-push`（push を勝手にしない／Claude 尾注を付けない）
- `kamires-form-align` スキル（各ファイルは自分の背景に合わせて整列する原則）
