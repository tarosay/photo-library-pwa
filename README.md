# 画像アップロード（EXIF必須・手動POST）PWA

このリポジトリは、EXIF 情報を検証してから JPEG 画像のみを任意のエンドポイントへ手動で POST 送信するためのプログレッシブ・ウェブアプリ（PWA）です。iPhone 等のモバイル端末からブラウザでアクセスできるよう、QR コードを同梱し、HTTPS 経由でのカメラ撮影にも対応しています。

## 主な機能
- **EXIF 必須チェック**：ISO / F 値 / シャッター速度のいずれかが含まれていない画像はアップロードを拒否します。
- **JPEG 限定運用**：ライブラリ・カメラのどちらから取り込んだ場合でも `image/jpeg` 以外はエラーになります（HEIC は送信不可）。
- **ライブラリ選択とブラウザ撮影**：`<input type="file">` によるライブラリ選択と、`ImageCapture` API を用いたブラウザからの直接撮影に対応。撮影時は端末の設定により HEIC が返る場合があるため、互換性優先（JPEG）に変更するようガイドが表示されます。
- **送信前プレビュー**：ファイル名・ MIME タイプ・サイズ・EXIF 概要を UI 上で確認できます。
- **手動 POST ボタン**：EXIF と形式の検証に合格した場合のみ POST ボタンが活性化し、`FormData` に画像を添付して指定エンドポイントへ送信します。
- **クリア機能**：現在の選択状態をリセットし、カメラストリームも停止します。

## 画面構成
- 画像選択・撮影ボタン（ライブラリ / ブラウザカメラ）
- ファイル情報表示（ファイル名・ MIME・サイズ・EXIF 判定）
- POST / クリアボタン
- ステータスメッセージ表示領域
- iPhone などで開くための QR コード画像（`QR_942906.png`）

## 利用手順
1. アプリを HTTPS で配信し、対応ブラウザ（Safari 18.4+ 推奨）でアクセスします。
2. 「ライブラリから選ぶ」または「カメラで撮影（ブラウザ）」のいずれかで画像を取得します。
   - カメラ起動には `navigator.mediaDevices.getUserMedia` および `ImageCapture` API が必要です。
3. 画像が JPEG かつ必要な EXIF を満たすと、ステータスに「EXIF OK」が表示され、POST ボタンが有効化されます。
4. 「POST送信」を押すと、`FormData` フィールド名 `image` でエンドポイントへ送信します。
5. 応答結果はステータスメッセージに表示されます。エラーの場合もメッセージで理由を確認できます。
6. 状態をリセットしたい場合は「クリア」ボタンを押します。

## EXIF ポリシー
- ISO・FNumber・ExposureTime の 3 要素のうち **1 つ以上** が含まれていれば送信可能です。
- 判定は [exifr](https://github.com/MikeKovarik/exifr) の CDN 版（`full.esm.js`）を用いて行っています。
- EXIF 解析に失敗した場合はエラーとして扱われ、送信できません。

## 設定値のカスタマイズ
`index.html` 内のモジュールスクリプト先頭で以下を変更できます。

```js
const ENDPOINT = "https://ik1-127-70116.vs.sakura.ne.jp/markAnomaly"; // 送信先
const FIELD_NAME = "image";                                           // FormData フィールド名
const REQUIRE_JPEG_ONLY = true;                                       // JPEGのみ送信
const ANY_ONE_OF_THREE_OK = true;                                     // EXIF 必須ポリシー
```

用途に応じて送信先や必須条件を調整してください。

## セットアップとホスティングのヒント
- 静的ホスティングサービス（Netlify、Vercel、GitHub Pages など）に配置するだけで利用できます。
- PWA としての配信には `manifest.webmanifest` と `service-worker.js` を併用してください。
- カメラ撮影機能は HTTPS が必須です。ローカルで試す場合は `localhost` を利用してください。

## ライセンス
MIT License（`LICENSE` を参照）。
