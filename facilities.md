# facilities API設計

---

## 概要
施設（facilities）の管理API。施設の一覧取得、詳細取得、登録、更新、削除、移動、コピー、およびVRView表示情報取得を行う。
現時点のDB設計では、施設本体はツリー構造と公開・属性情報を保持し、画像・地図・地図上位置・権限・設備・付箋は関連テーブルを参照して取得する。

## エンドポイント一覧
| 操作         | メソッド | URL例                                | 概要                       |
|--------------|----------|--------------------------------------|----------------------------|
| 一覧取得     | GET      | /facilities                          | 条件に一致する施設一覧取得 |
| 詳細取得     | GET      | /facilities/{facilityId}             | 施設詳細取得               |
| 表示用HTML取得 | GET    | /facilities/{facilityId}/view-html   | 表示用HTML取得             |
| 新規作成     | POST     | /facilities                          | 施設登録                   |
| 更新         | PUT      | /facilities/{facilityId}             | 施設更新                   |
| 削除         | DELETE   | /facilities/{facilityId}             | 施設削除                   |
| 移動         | POST     | /facilities/{facilityId}/move        | 施設移動                   |
| コピー       | POST     | /facilities/{facilityId}/copy        | 施設コピー                 |
| VRView取得   | GET      | /facilities/{facilityId}/vr-view     | VRView表示情報取得         |

## パラメータ定義

### 施設基本情報
| パラメータ名   | 型            | 必須 | 備考                                      |
|----------------|---------------|------|-------------------------------------------|
| facility_id    | BIGINT        | ○    | 施設ID。自動採番                          |
| parent_id      | BIGINT        |      | 親施設ID。ルートはNULL                    |
| facility_name  | VARCHAR(256)  | ○    | 施設名称                                  |
| tree_level     | INT           |      | 階層レベル                                |
| is_leaf        | BOOLEAN       |      | 末端フラグ                                |
| publish_mode   | VARCHAR(32)   |      | 公開モード。例: `private`, `public_self`, `public_children` |
| is_outdoor     | BOOLEAN       |      | 屋外フラグ                                |
| detail         | TEXT          |      | ノード詳細                                |
| has_permission | BOOLEAN       |      | 施設管理画面の「個別に権限を設定する」の設定有無（UI補助） |
| created_at     | DATETIME      |      | 作成日時（`yyyy/MM/dd HH:mm:ss` JST）     |
| updated_at     | DATETIME      |      | 更新日時（`yyyy/MM/dd HH:mm:ss` JST）     |
| deleted_at     | DATETIME      |      | 削除日時（`yyyy/MM/dd HH:mm:ss` JST）     |

### 関連参照項目（結合で返却可能）
| パラメータ名         | 型      | 必須 | 備考                                                 |
|----------------------|---------|------|------------------------------------------------------|
| images               | ARRAY   |      | 施設配下画像。images テーブルを参照                 |
| maps                 | ARRAY   |      | 施設配下地図。maps テーブルを参照                   |
| map_points           | ARRAY   |      | 地図上位置。map_points テーブルを参照               |
| roles                | ARRAY   |      | 施設権限。facility_role_permissions テーブルを参照   |
| equipments           | ARRAY   |      | 設備一覧。equipment / equipment_masters テーブルを参照 |
| annotations          | ARRAY   |      | 付箋一覧。annotations テーブルを参照                 |
| has_vr_image         | BOOLEAN |      | 表示補助情報。関連データの存在有無から導出         |
| has_emergency_annotation | BOOLEAN |   | 表示補助情報。関連データの存在有無から導出         |
| has_normal_annotation    | BOOLEAN |   | 表示補助情報。関連データの存在有無から導出         |
| has_equipment        | BOOLEAN |      | 表示補助情報。関連データの存在有無から導出         |

### images 要素
| パラメータ名      | 型            | 必須 | 備考 |
|-------------------|---------------|------|------|
| image_id          | BIGINT        | ○    | 画像ID |
| file_id           | BIGINT        | ○    | ファイルID |
| image_name        | VARCHAR(256)  |      | 画像名 |
| shooting_height_m | DECIMAL(6,3)  |      | 床面からカメラ中心までの高さ（m） |
| ceiling_height_m  | DECIMAL(6,3)  |      | 床面から天井までの高さ（m） |

### 一覧取得条件
| パラメータ名   | 型            | 必須 | 備考                                                   |
|----------------|---------------|------|--------------------------------------------------------|
| parentId       | BIGINT        |      | 親施設IDを指定した場合、直下施設一覧を取得する         |
| keyword        | VARCHAR(256)  |      | 施設名称部分一致検索                                   |

### 表示用HTML取得
| パラメータ名   | 型            | 必須 | 備考 |
|----------------|---------------|------|------|
| facilityId     | BIGINT        | ○    | パスで指定する対象施設ID |
| content_html   | TEXT          |      | 表示用のHTML本文。レスポンス項目 |
| updated_at     | DATETIME      |      | HTML更新日時（`yyyy/MM/dd HH:mm:ss` JST）。レスポンス項目 |

### 施設移動
| パラメータ名       | 型      | 必須 | 備考 |
|--------------------|---------|------|------|
| facilityId         | BIGINT  | ○    | パスで指定する移動元施設ID |
| target_facility_id | BIGINT  | ○    | 移動先施設ID。移動後は当該施設の子施設として配置する |

### 施設コピー
| パラメータ名       | 型      | 必須 | 備考 |
|--------------------|---------|------|------|
| facilityId         | BIGINT  | ○    | パスで指定するコピー元施設ID |
| target_facility_id | BIGINT  | ○    | コピー先施設ID。コピー後は当該施設の子施設として配置する |

## リクエスト例

### 一覧取得（直下施設一覧）
    GET /facilities?parentId=100

### 一覧取得（検索）
    GET /facilities?keyword=管理棟

### 詳細取得
    GET /facilities/1001

### 表示用HTML取得
  GET /facilities/1001/view-html

### 新規作成
    {
      "parent_id": 100,
      "facility_name": "第1会議室",
      "publish_mode": "public_self",
      "is_outdoor": false,
      "detail": "会議室ノード"
    }

### 更新
    {
      "facility_name": "第1会議室-更新",
      "publish_mode": "public_children",
      "is_outdoor": false,
      "detail": "会議室ノード更新"
    }

### 移動
    {
      "target_facility_id": 200
    }

### コピー
    {
      "target_facility_id": 300
    }

## レスポンス例

### 詳細取得
    {
      "success": true,
      "error_code": null,
      "message": "ok",
      "data": {
        "facility_id": 1001,
        "parent_id": 100,
        "facility_name": "第1会議室",
        "tree_level": 3,
        "is_leaf": true,
        "publish_mode": "public_self",
        "is_outdoor": false,
        "detail": "会議室ノード",
        "has_permission": true,
        "images": [
          {
            "image_id": 501,
            "file_id": 5001,
            "image_name": "会議室VR",
            "shooting_height_m": 1.600,
            "ceiling_height_m": 2.700
          }
        ],
        "maps": [
          {
            "map_id": 601,
            "file_id": 6001,
            "map_name": "3F平面図"
          }
        ],
        "map_points": [
          {
            "map_point_id": 701,
            "map_id": 601,
            "x_coord": 120.5,
            "y_coord": 230.0
          }
        ],
        "roles": [
          {
            "keycloak_role_id": "8c1b1111-2222-3333-4444-abcdef123456",
            "keycloak_role_name": "admin",
            "permission_code": "equipment_view"
          }
        ],
        "equipments": [
          {
            "equipment_id": 1,
            "equipment_name": "室外機A",
            "vr_x": 0.3456,
            "vr_y": 0.7890
          }
        ],
        "annotations": [
          {
            "annotation_id": 1,
            "title": "点検依頼"
          }
        ],
        "has_vr_image": true,
        "has_emergency_annotation": false,
        "has_normal_annotation": true,
        "has_equipment": true,
        "created_at": "2026/03/27 10:00:00",
        "updated_at": "2026/03/27 10:30:00",
        "deleted_at": null
      }
    }

### 一覧取得
    {
      "success": true,
      "error_code": null,
      "message": "ok",
      "data": [
        {
          "facility_id": 1001,
          "parent_id": 100,
          "facility_name": "第1会議室",
          "publish_mode": "public_self",
          "is_outdoor": false,
          "has_vr_image": true,
          "has_emergency_annotation": false,
          "has_normal_annotation": true,
          "has_equipment": true,
          "created_at": "2026/03/27 10:00:00",
          "updated_at": "2026/03/27 10:30:00",
          "deleted_at": null
        }
      ]
    }

### 表示用HTML取得
    {
      "success": true,
      "error_code": null,
      "message": "ok",
      "data": {
        "facility_id": 1001,
        "content_html": "<h1>第1会議室</h1><p>会議室ノード</p>",
        "updated_at": "2026/03/27 10:30:00"
      }
    }

### VRView表示情報取得
    {
      "success": true,
      "error_code": null,
      "message": "ok",
      "data": {
        "facility_id": 1001,
        "facility_name": "第1会議室",
        "has_vr_image": true,
        "vr_image": {
          "file_id": 5001,
          "url": "/files/5001",
          "shooting_height_m": 1.600,
          "ceiling_height_m": 2.700
        },
        "move_points": [
          {
            "facility_id": 1002,
            "facility_name": "第2会議室",
            "vr_x": 0.1234,
            "vr_y": 0.5678
          }
        ],
        "annotation_points": [
          {
            "annotation_id": 1,
            "annotation_type": "normal",
            "title": "点検依頼",
            "vr_x": 0.2222,
            "vr_y": 0.3333
          }
        ],
        "equipment_points": [
          {
            "equipment_id": 1,
            "equipment_name": "室外機A",
            "vr_x": 0.4444,
            "vr_y": 0.5555
          }
        ],
        "mini_map": {
          "file_id": 6001,
          "url": "/files/6001",
          "map_points": [
            {
              "facility_id": 1002,
              "facility_name": "第2会議室",
              "x": 120.5,
              "y": 230.0
            }
          ]
        }
      }
    }

## エラーレスポンス例
    {
      "success": false,
      "error_code": "conflict",
      "message": "対象データが他ユーザにより更新されています",
      "errors": null,
      "data": null
    }

## 認可
- ログイン済ユーザのみ利用可
- 一覧取得・詳細取得・表示用HTML取得・VRView表示情報取得は、対象施設に対する参照権限を持つユーザのみ可
- 登録・更新・削除・移動・コピーは、対象施設に対する管理権限を持つユーザのみ可
- 管理機能ロールを持たないユーザは管理系操作を実行不可とする

---

## 備考
- GET /facilities は、parentId 指定時は直下施設一覧取得、keyword 指定時は施設検索として利用する。
- `GET /facilities/{facilityId}` は編集用途の詳細情報取得、`GET /facilities/{facilityId}/view-html` は表示用途の軽量取得として使い分ける。
- 論理削除済み施設は Webアプリから参照対象外とし、一覧取得・詳細取得・表示用HTML取得・VRView表示情報取得では返却しない。
- 施設ツリー画面では、権限のない施設データは返却しない。
- 画像情報は images、地図情報は maps、地図上位置は map_points を参照して取得する。
- 施設画像（images）には、撮影高さ `shooting_height_m` と天井高 `ceiling_height_m`（単位: m）を保持し、必要に応じて詳細取得・VRView表示情報取得で返却してよい。
- 施設画像・地図画像の登録や差替は、事前に `POST /files` でアップロードしたファイルIDを施設更新APIの関連情報としてまとめて送信する運用を想定する。
- VRView の縮小マップは、地図上の map_points をもとに現在施設および同一親施設配下の施設位置を表示する想定とする。
- 権限情報は facility_role_permissions テーブルを参照して取得する。
- `roles` の各要素は、`keycloak_role_id`（施設に紐づくKeycloakロール）と `permission_code`（例: `annotation_view`, `equipment_view`）の組で表現する。
- roles、equipments、annotations はDB保持項目ではなく、関連テーブル結合で返却してよい。
- 画像有無、付箋有無、設備有無などの表示補助情報は、関連データの存在有無から導出してよい。
- 移動・コピーでは、操作対象施設はパスの `facilityId`、移動先・コピー先はボディの `target_facility_id` で指定する。
- 管理画面において施設全体の一括保存を採用する場合、施設更新APIで関連編集内容をまとめて保存してよい。
