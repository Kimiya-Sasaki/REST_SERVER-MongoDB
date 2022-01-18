# REST API SERVER + MongoDB
Express (Node.js) と MongoDB を用いて RESTful な環境を構築する
# Table Of Contents
- [事前準備](#prep)
- [Directory 構成](#directories)

<h2 id="prep">事前準備</h2>

### 以下のソフトウェアを事前にインストールしておく
- Node.js
- npm
- 適当な Project Folder を作成 e.g., reet-server
- Project Folder 内で以下のコマンドを実行する
```
npm init -y
npm i express mongoose nodemon dotenv
```
|項 目|説 明|
----|----
|express|Web Framework で今回は Rest API Server を構築する目的で利用|
|mongoose|MongoDB にアクセスするためのライブラリ|
|nodemon|各種ファイルを修正したときに Server を Restert してくれる|
|dotenv|.env ファイルにプログラムで参照する定数を定義する|

### package.json の編集
scripts の項目を以下の内容に置き換える
```
"scripts": {
  "start": "nodemon app.js"
},
```

<h2 id="directories">Director 構成</h2>

|構 成|説 明|
----|----
||- schema.js: Table Schema を定義<br/>- restapi.js: REST API の Entrypoint<br/>- .env: 定数定義用<br/>- app.js: アプリケーション本体<br/>- route.rest: REST リクエストを容易に実現する|
