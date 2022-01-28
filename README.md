# REST API SERVER 構築
express (Node.js) と MongoDB を用いて RESTful な Server 環境を構築
# Table Of Contents
- [事前準備](#prep)
- [Directory 構成](#directories)
- [SSH 設定](#ssh)
- 各構成ファイルの詳細
  - [crud.js](#crud)
  - [todo.js](#todo)
  - [api.js](#api)
  - [.env](#env)
  - [app.js](#appjs)
  - [api.rest](#test)

<h2 id="prep">事前準備</h2>

### 以下のソフトウェアを事前にインストールしておく
- node (Node.js)
- npm
- npx
- MongoDB
- 適当な Project Folder を作成 e.g., reet-server
- api.rest を利用するには VSCode に REST Client Plugin を install する必要あり
- Project Folder 内で以下のコマンドを実行
```
npm init -y
npm i express mongoose nodemon dotenv express-validator
```
|項 目|説 明|
----|----
|express|Web Framework で今回は Rest API Server を構築する目的で利用|
|mongoose|MongoDB にアクセスするためのライブラリ|
|nodemon|各種ファイルを修正したときに Server を Restert してくれる|
|dotenv|.env ファイルにプログラムで参照する定数を定義する|
|express-validator|入力情報の可否を検証するバリデータ|

### package.json の編集
scripts の項目を以下の内容に置き換える
```
"scripts": {
  "start": "nodemon app.js"
},
```
- アプリケーションの起動
```
npm start
```
console output
<pre>
> server@1.0.0 start
> nodemon app.js

[nodemon] 2.0.15
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node app.js`
Server listening on port 8080
</pre>

<h2 id="ssh">SSH 設定</h2>

以下のコマンドを実行すると privatekey.pem, cert.pem が作成される
```
cd <Project Folder>
openssl req -x509 -newkey rsa:2048 -keyout privatekey.pem -out cert.pem -nodes
```
output: 問いには JP だけ設定すれば後はデフォルトで構わない
```
Country Name (2 letter code) []:JP
State or Province Name (full name) []:
Locality Name (eg, city) []:
Organization Name (eg, company) []:
Organizational Unit Name (eg, section) []:
Common Name (eg, fully qualified host name) []:
Email Address []:
```

<h2 id="directories">Directry 構成</h2>

|構 成|説 明|
----|----
|<img src="./imgs/directories.jpg">|- crud.js: REST APIs 本体<br/>- todo.js: Data Schema<br/>- api.js: Routes の定義<br/>- .env: 定数定義用<br/>- route.rest: REST APIs Debug 用<br/>- app.js: App 本体

# 各構成ファイルの詳細
- 各 Program の Codde を紹介
- 具体的な処理の内容までは解説しないが目的についてはコメントを参照

<h2 id="crud">crud.js</h2>

CRUD の具体的な処理を記述
```javascript
const { validationResult } = require('express-validator');
const Todo = require('../models/todo');

// 全ての Todo を取得
exports.getAllTodos = (req, res, next) => {
  Todo.find()
    .then(todos => {
      res.status(200).json({
        message: "successfully fetched todos",
        todos: todos.map(todo => {
          return { name: todo.name, status: todo.status, id: todo._id.toString()};
        })
      });
  })
  .catch(err => {
    if(! err.statusCode) {
      err.statusCode = 500;
      next(err);
    }
  });
};

// 特定の Todo を取得
exports.getSingleTodo = (req, res, next) => {
  const tid = req.params.tid;
  Todo.findById(tid)
    .then(todo => {
      if(! todo) {
        const error = new Error('Could not find todo.');
        error.statusCode = 404;
        next(error);
      }
      res.status(200).json({
        message: "successfully fetched todo",
        todo: { name: todo.name, status: todo.status, id: todo._id.toString()}
      });
    }).catch(err => {
      if(! err.statusCode) {
        err.statusCode = 500;
        next(err);
      }
    });
};

// 入力されたデータを保存
exports.postTodo = (req, res, next) => {
  const errors = validationResult(req);
  console.log(errors);
  if(! errors.isEmpty()) {
    const error = new Error('Validation failed, provided data is incorrect.');
    error.statusCode = 422;
    next(error);
  }
  const todo = new Todo({
    name: req.body.name,
    status: req.body.status
  });
  todo.save()
    .then(todo => {
      res.status(201).json({
        message: 'Todo successfully created!',
        todo: { name: todo.name, status: todo.status, id: todo._id.toString()}
      });
    })
    .catch(err => {
      if(! err.statusCode) {
        err.statusCode = 500;
        next(err);
      }
    });
};

// status だけを更新
exports.putTodo = (req, res, next) => {
  const errors = validationResult(req);
  if(! errors.isEmpty()) {
    const error = new Error('Validation failed, provided data is incorrect.');
    error.statusCode = 422;
    next(error);
  }
  const tid = req.params.tid;
  Todo.findById(tid)
    .then(todo => {
      if(! todo) {
        const error = new Error('Could not find todo.');
        error.statusCode = 404;
        next(error);
      }
      todo.status = req.body.status;
      return todo.save();
    })
    .then(todo => {
      res.status(200).json({ 
          message: 'Todo updated!', 
          todo: { name: todo.name, status: todo.status, id: todo._id.toString()} 
      });
    }).catch(err => {
      if(! err.statusCode) {
        err.statusCode = 500;
        next(err);
      }
    });
};

// データの削除
exports.deleteTodo = (req, res, next) => {
  const tid = req.params.tid;
  Todo.findById(tid)
    .then(todo => {
      if(! todo) {
        const error = new Error('Could not find todo.');
        error.statusCode = 404;
        next(error);
      }
      return Todo.findByIdAndRemove(tid);
    })
    .then(result => {
      res.status(200).json({ message: 'Todo deleted.' });
    })
    .catch(err => {
      if(! err.statusCode) {
        err.statusCode = 500;
        next(err);
      }
    });
};
```

<h2 id="todo">todo.js</h2>

```javascript
const mongoose = require('mongoose')

const apiSchema = new mongoose.Schema({
  username: {
    type: String,
    required: true
  },
  message: {
    type: String,
    required: true
  },
  date: {
    type: Date,
    required: true,
    default: Date.now
  }
})

module.exports = mongoose.model('messages', apiSchema)
```

<h2 id="api">api.js</h2>

```javascript
const express = require('express');
const { body } = require('express-validator');

// 実際の CRUD 処理を定義
const crudController = require('../controllers/crud');

// Router を定義
const router = express.Router();

// 全ての Todo を取得
router.get('/get', crudController.getAllTodos);

// 特定のた Todo を取得
router.get('/get/:tid', crudController.getSingleTodo);

// 入力されたデータを保存
router.post('/post', [
  body('name',"Provide a valid message").isLength({ min: 1 }).trim(),
  body('status', "Provide a valid status").isBoolean()
], crudController.postTodo);

// データの更新
router.put('/put/:tid', [
  body('status', "Provide a valid status").isBoolean()
], crudController.putTodo);

// データの削除
router.delete('/delete/:tid', crudController.deleteTodo);

module.exports = router;
```

<h2 id="env">.env</h2>

```
SERVER_PORT=3000
DB=mongodb://127.0.0.1:27017/todos
```

<h2 id="appjs">app.js</h2>

```javascript
const express = require('express');
const cors = require('cors');
const mongoose = require('mongoose');
const crudRoutes = require('./routes/api');
const fs = require('fs');

// .env に定義された値を利用可能にする
require('dotenv').config();

// REST APIs Server 用に express Framework 利用する
const app = express();

// CORS 対策
app.use(cors());

// JSON 形式でやり取りする宣言（REST APIs 用）
app.use(express.json());

// REST APIs の Routes（URIs） を読み込む
app.use("/api", crudRoutes);

// Routes 以外でアクセスしてきた場合は error を返す
app.use((error, req, res, next) => {
  console.log(error);
  res.status(error.statusCode || 500).json({
    message: error.message,
    data: error.data
  });
});

// SSH の設定
const server = require('https').createServer({
    key: fs.readFileSync('./privatekey.pem'),
    cert: fs.readFileSync('./cert.pem'),
}, app)

// MongoDB に接続して成功したら Server を立ち上げる
mongoose.connect(process.env.DB)
  .then(result => {
    server.listen(process.env.PORT, () => {
      console.log("Server listening on port "+process.env.PORT);
    });
  })
  .catch(error => {
    console.log(error);
  });
```

<h2 id="test">api.rest</h2>

【:id に該当する部分は適宜変更すること】
```
GET http://127.0.0.1:8080/api/todos

###
GET http://127.0.0.1:8080/api/todo/:id

###
POST http://127.0.0.1:8080/api/todo
content-type: application/json

{
  "name": "上手く動いているかな？",
  "status": false
}

###
PUT http://127.0.0.1:8080/api/todo/:id
content-type: application/json

{
  "status": true
}

###
DELETE http://127.0.0.1:8080/api/todo/:id
```
