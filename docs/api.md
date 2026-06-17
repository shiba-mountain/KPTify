# API仕様書

## 概要

KPTifyで利用するAPI仕様を定義する。

| ホスト | プロトコル | バージョン | データ形式 |
| --- | --- | --- | --- |
| `kptify.com` | `https` | `v1` | `json` |

**URL**: `https://kptify.com/api/v1`

## ステータスコード

下記のコードを返却します。

| ステータスコード | 説明 |
| --- | --- |
| 200 | リクエスト成功 |
| 201 | 登録成功 |
| 400 | 不正なリクエストパラメータを指定している |
| 401 | 認証が必要 |
| 404 | 対象データが存在しない |
| 409 | データの重複 |
| 500 | 不明なエラー |

## Reflectionリソース

| HTTPメソッド | エンドポイント | 処理内容 |
| --- | --- | --- |
| GET | `/api/v1/reflections` | 指定月の振り返り一覧を取得する |
| GET | `/api/v1/reflections/{targetDate}` | 指定日の振り返り詳細を取得する |
| POST | `/api/v1/reflections/{targetDate}/items` | 指定日の振り返り項目を追加する |
| PATCH | `/api/v1/reflections/{targetDate}/items/{itemId}` | 指定日の振り返り項目を更新する |
| DELETE | `/api/v1/reflections/{targetDate}/items/{itemId}` | 指定日の振り返り項目を削除する |

### 振り返り一覧を取得

```http
GET /api/v1/reflections HTTP/1.1
```

#### Request

| パラメータ | 内容 | 型 | 必須 |
| --- | --- | --- | --- |
| year | 対象年を設定 | number | ○ |
| month | 対象月を設定 | number | ○ |

```json
{
  "year": 2026,
  "month": 1
}
```

#### Response

```
HTTP/1.1 200 OK
{
  "data": {
    "dates": [
      "2026-06-01",
      "2026-06-02",
      "2026-06-05"
    ]
  }
}
```

### 振り返り詳細を取得

```http
GET /api/v1/reflections/{targetDate} HTTP/1.1
```

#### Request

| パラメータ | 内容 | 型 | 必須 |
| --- | --- | --- | --- |
| targetDate | 対象日を設定 | date | ○ |

```
{
  "targetDate": "2026-06-01"
}
```

#### Response

```
HTTP/1.1 200 OK
{
  "data": {
    "id": 1,
    "targetDate": "2026-06-01",
    "items": [
      {
        "id": 1,
        "type": "keep",
        "content": "振り返りの内容"
      },
      {
        "id": 2,
        "type": "problem",
        "content": "振り返りの内容"
      },
      {
        "id": 3,
        "type": "try",
        "content": "振り返りの内容"
      }
    ]
  }
}
```

### 振り返り項目を追加

```http
POST /api/v1/reflections/{targetDate}/items HTTP/1.1
```

#### Path Parameter

| パラメータ | 内容 | 型 | 必須 |
| --- | --- | --- | --- |
| targetDate | 対象日を設定 | date | ○ |

#### Request Body

| パラメータ | 内容 | 型 | 必須 |
| --- | --- | --- | --- |
| type | 振り返り種別 | string | ○ |
| content | 振り返り本文 | string | ○ |

```
{
  "type": "keep",
  "content": "振り返り本文"
}
```

#### Response

```
HTTP/1.1 201 Created
{
  "data": {
    "id": 1,
    "reflectionId": 1,
    "type": "keep",
    "content": "振り返り本文"
  }
}
```

### 振り返り項目を更新

```http
PATCH /api/v1/reflections/{targetDate}/items/{itemId} HTTP/1.1
```

#### Path Parameter

| パラメータ | 内容 | 型 | 必須 |
| --- | --- | --- | --- |
| targetDate | 対象日を設定 | date | ○ |
| itemId | 振り返り項目IDを設定 | number | ○ |

#### Request Body

| パラメータ | 内容 | 型 | 必須 |
| --- | --- | --- | --- |
| content | 振り返り本文 | string | ○ |

```
{
  "content": "振り返り本文"
}
```

#### Response

```
HTTP/1.1 200 OK
{
  "data": {
    "id": 1,
    "reflectionId": 1,
    "type": "keep",
    "content": "振り返り本文"
  }
}
```

### 振り返り項目を削除

```http
DELETE /api/v1/reflections/{targetDate}/items/{itemId} HTTP/1.1
```

#### Path Parameter

| パラメータ | 内容 | 型 | 必須 |
| --- | --- | --- | --- |
| targetDate | 対象日を設定 | date | ○ |
| itemId | 振り返り項目IDを設定 | number | ○ |

#### Request

```
なし
```

#### Response

```
HTTP/1.1 204 No Content
```

## Analysisリソース

| HTTPメソッド | エンドポイント | 処理内容 |
| --- | --- | --- |
| POST | `/api/v1/analysis` | 指定月のAI分析を実行する |
| GET | `/api/v1/analysis/{targetMonth}` | 指定月のAI分析結果を取得する |

### 月次分析を実行

```http
POST /api/v1/analysis HTTP/1.1
```

#### Request

| パラメータ | 内容 | 型 | 必須 |
| --- | --- | --- | --- |
| targetMonth | 対象月を設定 | date | ○ |

```
{
  "targetMonth": "2026-06-01"
}
```

#### Response

```
HTTP/1.1 201 Created
{
  "data": {
    "id": 1,
    "targetMonth": "2026-06-01"
  }
}
```

#### 備考

* 指定月の振り返りデータを取得し、Google Gemini API に送信してAI分析を実行する
* AI分析結果は analysis_results、analysis_themes、analysis_items に保存する
* Google Gemini API に送信するプロンプトおよびレスポンス形式は、実装時に確定する

### 月次分析結果を取得

```http
GET /api/v1/analysis/{targetMonth} HTTP/1.1
```

#### Request

| パラメータ | 内容 | 型 | 必須 |
| --- | --- | --- | --- |
| targetMonth | 対象月を設定 | date | ○ |

```
{
  "targetMonth": "2026-06-01"
}
```

#### Response

```
HTTP/1.1 200 OK
{
  "data": {
    "id": 1,
    "targetMonth": "2026-06-01",
    "themes": [
      {
        "id": 1,
        "themeName": "テーマ名",
        "occurrenceCount": 8,
        "description": "テーマ本文"
      }
    ],
    "items": [
      {
        "id": 1,
        "type": "summary",
        "content": "総評本文"
      },
      {
        "id": 2,
        "type": "strength",
        "content": "強み本文"
      },
      {
        "id": 3,
        "type": "issue",
        "content": "課題本文"
      },
      {
        "id": 4,
        "type": "insight",
        "content": "気付き本文"
      }
    ]
  }
}
```