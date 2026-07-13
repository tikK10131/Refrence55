# files API設計

---

## 概要
ファイル（files）の管理API。添付ファイルアップロードおよび添付ファイル取得を行う。

## エンドポイント一覧
| 操作       | メソッド | URL例                    | 概要                   |
|------------|----------|--------------------------|------------------------|
| アップロード | POST     | /files               | 添付ファイルアップロード |
| 取得       | GET      | /files/{fileId}      | 添付ファイル取得       |

## パラメータ定義
| パラメータ名     | 型             | 必須 | 備考                            |
|------------------|----------------|------|---------------------------------|
| file_id          | BIGINT         | ○    | ファイルID。自動採番            |
| file_name        | VARCHAR(256)   | ○    | 元ファイル名                    |
| file_path        | VARCHAR(1024)  |      | 保存パス                        |
| file_type        | VARCHAR(32)    |      | ファイル種別                    |
| file_size        | BIGINT         | ○    | ファイルサイズ（Byte）          |
| storage_type     | VARCHAR(32)    |      | ストレージ種別                  |
| external_file_id | VARCHAR(256)   |      | 外部ファイルID                  |
| external_url     | VARCHAR(512)   |      | 外部URL                         |
| external_bucket  | VARCHAR(256)   |      | 外部バケット                    |
| created_at       | DATETIME       |      | 作成日時（`yyyy/MM/dd HH:mm:ss` JST） |
| updated_at       | DATETIME       |      | 更新日時（`yyyy/MM/dd HH:mm:ss` JST） |
| deleted_at       | DATETIME       |      | 論理削除日時（`yyyy/MM/dd HH:mm:ss` JST） |

## リクエスト

### アップロード
- `multipart/form-data` を使用する。
- ファイル本体は `file` 項目で送信する。
- 必要に応じて関連情報を追加パラメータとして送信してよい。

### アップロード例
    POST /files
    Content-Type: multipart/form-data

    file=<binary>

## レスポンス例

### アップロード
    {
      "success": true,
      "error_code": null,
      "message": "ok",
      "data": {
        "file_id": 10001,
        "file_name": "manual.pdf",
        "file_type": "application/pdf",
        "file_size": 245760,
        "storage_type": "local",
        "external_file_id": null,
        "external_url": null,
        "external_bucket": null,
        "created_at": "2026/03/27 10:00:00",
        "updated_at": "2026/03/27 10:00:00"
      }
    }

### 取得
- 本APIは JSON ではなくファイル本体を返却する。
- ブラウザ表示可能なファイルは別タブ表示またはブラウザ表示とする。
- ブラウザ表示に適さないファイルはダウンロードとする。

    GET /files/10001

## エラーレスポンス例
    {
      "success": false,
      "error_code": "validation_error",
      "message": "アップロード対象ファイルが指定されていません",
      "errors": {
        "file": ["この項目は必須です"]
      },
      "data": null
    }

## 認可
- ログイン済ユーザのみ利用可
- アップロードは付箋登録、付箋更新、施設編集等の対象画面で操作可能なユーザのみ可
- 取得は対象ファイルに紐づく施設、付箋、設備等に対する参照権限を持つユーザのみ可
- 施設単位の権限判定は `facility_role_permission` と `permission_master` に基づいて行う

---

## 備考
- 本APIは付箋添付ファイル、施設画像ファイル、その他関連ファイルのアップロードおよび取得に利用する。
- `GET /files/{fileId}` はファイル本体返却APIであり、JSONレスポンスは返さない。
- 表示またはダウンロードの挙動は `Content-Type` および `Content-Disposition` により制御する。
- `content_type`、`uploaded_by`、`uploaded_at` をAPIで返却する場合は、DB項目との差異を吸収するマッピングを行う。
- 既存ファイルの差替時も `POST /files` により新規 `file_id` を採番してアップロードする。
- 旧ファイルの削除判定は files API ではなく、付箋更新API・施設更新APIなど関連API側で、更新後のファイル一覧との差分判定により実施する。
- 差分で不要となった旧ファイルは `files.deleted_at` を更新して論理削除し、物理削除が必要な場合は運用バッチ等で実施してよい。