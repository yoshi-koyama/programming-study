# やってみてほしいこと

## RESTful API開発のチケットを作成

以下のゴールとNonゴールを参考にしてゴール達成のために必要であろうチケットをすべて作成してください。

ゴール
- API BlueprintによるAPI仕様書が作成されていること
- ローカル環境でRESTfulAPIが動作し、動作の確認ができていること
- JUnitによる単体テストが作成されていること
- Github ActionsまたはCircle CIによりpush時に単体テストが自動で実行されるようになっていること

Nonゴール
- 結合テストの実装
- AWSへのデプロイ

## RESTful APIの簡単な仕様

こちらは不完全なものです。  
（あえて不完全にしている部分もありますし、普通にわたしの考慮漏れもあります）  
大きな更新がある場合はSlackチャンネルにてお知らせします。  

もしここ改善すべき？という点があれば教えて下さい！  

### APIの概要

以下の機能をもったAPIを作成してください。

|#|機能|URI|メソッド|ステータスコード|
|---|---|---|---|---|
|1|ユーザーを登録|/users|POST|201|
|2|ユーザーを検索|/users|GET|200|
|3|ユーザーを更新|/users/{id}|PATCH|200|
|4|ユーザーを削除|/users/{id}|DELETE|200|

## 詳細
### 1. 登録
ユーザーを登録する。
```json
{
  "name": "小山",
  "birthdate": "2022-01-01"
}

```

以下バリデーションを実施する
- 名前
  - null、空文字でないこと
  - 255文字以下であること
- 生年月日
  - nullでないこと
  - yyyy-MM-dd形式であること
  - 未来日付でないこと

バリデーションエラーとなる場合はステータスコード400を返却し、以下のようなレスポンスボディを返し、どの項目にどんなエラーがあったのかわかるようにすること。
```json
{
   "message":"validation error",
   "errors":[
      {
         "field":"name",
         "messages":[
            "cannot be empty",
            "maximum length is 255"
         ]
      },
      {
         "field":"birthdate",
         "messages":[
            "cannot be null",
            "format should be yyyy-MM-dd",
            "cannot set futre date"
         ]
      }
   ]
}
```

### 2. 検索

ユーザーを検索する。

クエリ文字列に指定したパラメータをもとに検索を行う。  
例)localhost:8080/users?name=koyama&birthdate=2022-01-01

|key|説明|
|---|---|
|name|前方一致するものを検索する|
|birthdate|完全一致するものを検索する|

memo: birthdateに関しては日付の範囲検索機能を実装してみてもOK

バリデーションについては以下を実施してください。
- 生年月日
  - yyyy-MM-dd形式であること

レスポンスボディは以下とする。

```json
[
   {
     "id": "01FZMTP19VSKWBQPXJA4GKZ2Y3",
     "name": "小山太郎",
     "birthdate": "1990-01-01"
   },
   {
     "id": "01FZMTQG90X9PX811WQW0JH9BT",
     "name": "小山次郎",
     "birthdate": "1992-01-01"
   }
]
```

ユーザーがHITしない場合、以下となる
```json
[]
```

### 3. 更新
idに指定したユーザーを更新する。

リクエストボディ例
```json
{
  "name": "小山",
  "birthdate": "2022-01-01"
}
```

該当のユーザーが存在しない場合はステータスコード404を返却し、レスポンスボディに以下を返す。

```json
{
  "message": "user not found"
}
```

バリデーションエラーは登録時のものに従う。

レスポンスボディは下記とする。
```json
{
  "message": "user updated"
}
```

### 4. 削除
idに指定したユーザーを論理削除する。  
該当のユーザーが存在しない場合はステータスコード404を返却し、レスポンスボディに以下を返す。

```json
{
  "message": "user not found"
}
```

レスポンスボディは下記とする。
```json
{
  "message": "user deleted"
}
```

### その他のエラー処理

存在しないURIへのアクセスは、ステータスコード404を返す。
レスポンスボディは下記とする。

```json
{
  "message": "not found"
}
```

DBへの接続ができないなどのシステムエラー時はステータスコード500を返す。
```json
{
  "message": "system error"
}
```

### データベース定義

|カラム名（論理名）|カラム名（物理名）|型・桁|Nullable|その他コメント|
|---|---|---|---|---|
|ID|id|char(26)|NO|primary key。採番はULIDで実装|
|名前|name|varchar(255)|NO||
|生年月日|birthdate|date|NO|YYYY-MM-DD形式|
|削除フラグ|deleted|boolean|NO|0:有効、1:削除|
|作成日時|created_at|datetime|NO|YYYY-MM-DD HH:MM:SS形式|
|作成者|created_by|varchar(255)|NO|**どんな値を入れるべきかは各チーム考えてください**|
|更新日時|updated_at|datetime|NO|YYYY-MM-DD HH:MM:SS形式|
|更新者|updated_by|varchar(255)|NO|**どんな値を入れるべきかは各チーム考えてください**|
|削除日時|deleted_at|datetime|YES|YYYY-MM-DD HH:MM:SS形式|
|削除者|deleted_by|varchar(255)|YES|**どんな値を入れるべきかは各チーム考えてください**|
