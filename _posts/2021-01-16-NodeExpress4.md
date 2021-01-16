---
layout: post
title:  "Node JS Web Express framwork 4"
date:   2021-01-16
categories: [web]
---
# Node Express 라우터 사용하기

### 소프트웨어가 거대해지면 서비스마다 분리하여 관리를 하여 유지/보수를 편리하게 구축해야한다. 따라서 서비스별로 router를 이용하여 경로를 잡아주고 소스를 나누어 줄 수 있다.<br><br>

### 우선 현재 예시는 목록과 게시판형태의 서비스만 있기 때문에 홈, topic으로 2분류로 구분하였다.

## 1. topic 라우터
#### routes 디렉토리에 우선 tp.js 파일을 만들어 준다.
#### 그리고 main.js에 있던 topic과 관련된 소스를 옮겨 준다.<br><br>
### - routes / tp.js
```javascript
var express = require('express');
var router = express.Router();//라우터를 사용하기위해 선언
var topic = require("../lib/topic.js");//부모 디렉토리에서 lib 폴더를 접근
var bodyParser =require('body-parser'); //미들웨어인 바디파서를 사용
//미들웨어를 표현하는 표현식 (사용자가 전달한 post데이터를 분석해준다)
router.use(bodyParser.urlencoded({extended:false}));

router.get('/create',(req,res)=>topic.create(req, res));
router.post('/create_process',(req,res)=>topic.create_process(req, res));
router.get('/update/:pageId',(req,res)=>topic.update(req, res));
router.post('/update_process',(req,res)=>topic.update_process(req, res));
router.post('/delete_process',(req,res)=>topic.delete_process(req, res));
// /:pageId로 설정하게 되면 request.params.pageId를 통해 값을 얻을 수 있다.
router.get('/:pageId',(req,res) =>topic.page(req, res));

module.exports=router;//router를 모듈로서 설정하기 위해 export해준다.
```
>#### **주의**<br>페이지를 이동하는 코드의 순서가 중요하다. 따라서 /:pageId가 있는 코드 위에 모든 소스를 위치 하였다.<br>

#### <br>홈 화면은 따로 index파일로 분류한다.
### - routes / index.js
```javascript
var express = require('express');
var router = express.Router();//라우터를 사용하기 위해 선언
var topic = require("../lib/topic.js");//부모 디렉토리에서 lib 폴더를 접근

router.get('/',(req,res)=>topic.home(req, res));

module.exports=router;
```
### 현재 tp 와 index에서 topic 을 lib에서 접근을 하고 있으나, 이렇게 라우터로 구분을 하게 되면 각각 서비스에 해당하는 파일에서 직접 소스를 구현하여도 괜찮을 것 같다. 

### - main.js
```javascript
const express = require('express')//모듈을 가져온다
const app = express();
var author = require('./lib/author.js');
var topicRouter = require('./routes/tp'); // 라우터를 통해 tp 사용한다는 의미
var indexRouter = require('./routes/index'); //라우터로 index 읽어오게함
app.use('/topic',topicRouter);//topic경로로 가게되는 페이지는 모두 topiRouter에서 설정해준 ./routes/tp의 내용들을 읽어준다.
app.use('/',indexRouter);

// 특정 경로에 내가 만든 미들웨어를 동작하도록 설정
app.use('/',function(request,response,next){
  console.log('Time:',Date.now());
  //next는 데이터를 request통해 전달 받고 실행한다는 의미
  next();
});

//express 정적 서비스를 가져오는 내장함수 (static(접근경로))
app.use(express.static('public'));

//404 에러가 떴을때 응답하도록 설정
app.use(function(req,res,next){
  res.status(404).send('sorroy not found')
})

//에러가 났을때(404가 아닌 쿼리문이나 내부 에러가 났을때)
app.use(function(err, req, res, next){
  console.error(err.stack)
  res.status(500).send('Something broke')
})
//3000 번 포트일때 반응하도록 설정
app.listen(3000)
```
> #### 페이지와 관련된 부분만 남겨두고 나머지는 라우터를 이용하여 따로 구분하였다.

## <br>이 외 파일
### - db.js
```javascript
var mysql = require("mysql");
var db = mysql.createConnection({
  host: "localhost",
  user: "root",
  password: "system",
  database: "opentutorials",
});
db.connect();

module.exports = db;
```
### - topic.js
```javascript
var db = require("../lib/db");
var template = require("../lib/template.js");
var sanitizeHtml = require('sanitize-html');
var path = require('path');
const express = require('express')//모듈을 가져온다
const app = express();
var compression = require('compression'); // 전달할 데이터를 압축하는 모듈
app.use(compression());//압축 사용

exports.home = function (request, response) {
  db.query(`SELECT *FROM topic`, function (error, topics) {
    var title = "Welcome";
    var description = "Hello Node.js";
    var list = template.list(topics);
    var html = template.html(
      title,
      list,
      `<h2>${title}</h2>${description}
      <img src="/images/hello.jpg" width="300px" style="display:block; margin-top:10px">`,
      `<a href="/topic/create">create</a>`
    );
    response.send(html);
  });
};

exports.page = function (request, response) { 

  var pi = path.parse(request.params.pageId).base; 
  db.query(`SELECT *FROM topic`, function (error, topics) {
    if (error) {
      throw error;
    }
 
    db.query(
      `SELECT * FROM topic LEFT JOIN author ON topic.author_id = author.id WHERE topic.id =?`,
      [pi],
      function (error2, topic) {
        if (error2) {
          next(err);
        } else {
          var title = topic[0].title;
          var description = topic[0].description;
          var list = template.list(topics);
          var html = template.html(
            title,
            list,
            `<h2>${sanitizeHtml(title)}</h2>${sanitizeHtml(description)}
                <p>by ${sanitizeHtml(topic[0].name)}</p>
                `,
            `<a href="/topic/create">create</a> <a href="/topic/update/${pi}">update</a>
                  <form action="/topic/delete_process" method="post">
                    <input type="hidden" name="id" value="${pi}">
                    <input type="submit" value="delete">
                  </form>`
          );
          response.send(html);
        }
      }  
    );
  });
};

exports.create = function (request, response) {
  db.query(`SELECT *FROM topic`, function (error, topics) {
    db.query(`SELECT * FROM author`, function (error2, authors) {
      var title = "Create";
      var list = template.list(topics);
      var html = template.html(
        sanitizeHtml(title),
        list,
        `<form action="/topic/create_process" method="post">
            <p><input type="text" name ="title" placeholder="title"></p>
            <p><textarea name="description" placeholder="description"></textarea></p>
            <p>
              ${template.authorSelct(authors)}
            </p>
            <p><input type="submit"></p></from>`,
        `<a href="/topic/create">create</a>`
      );
      response.send(html);
    });
  });
};

exports.create_process = function (request, response) {
  var post = request.body;
  db.query(
    `INSERT INTO topic (title, description, created, author_id) VALUES(?, ?, NOW(), ?)`,
    [post.title, post.description, post.author],
    function (error, result) {
      if (error) {
        throw error;
      }
      response.redirect(`/`);
    }
  );
};

exports.update = function (request, response) {
  var pi = path.parse(request.params.pageId).base;
  db.query(`SELECT * FROM topic`, function (error, topics) {
    if (error) {
      throw error;
    }
    db.query(
      `SELECT * FROM topic WHERE id =?`,
      [pi],
      function (error2, topic) {
        if (error2) {
          throw error2;
        }
        db.query(`SELECT * FROM author`, function (error2, authors) {
          var list = template.list(topics);
          var html = template.html(
            sanitizeHtml(topic[0].title),
            list,
            `<form action="/topic/update_process" method="post">
              <input type="hidden" name="id" value="${topic[0].id}"
              <p><input type="text" name ="title" placeholder="title" value="${
                sanitizeHtml(topic[0].title)
              }"></p>
              <p><textarea name="description" placeholder="description">${
                sanitizeHtml(topic[0].description)
              }</textarea></p>
              <p>
                ${template.authorSelct(authors, topic[0].author_id)}
              </p>
              <p><input type="submit"></p>
              </form>`,
            `<a href="/topic/create">create</a> <a href="/topic/update/${topic[0].id}">update</a>`
          );
          response.send(html);
        });
      }
    );
  });
};

exports.update_process = function (request, response) {

    var post = request.body;
    db.query(
      `UPDATE topic SET title = ?, description = ?, author_id=? WHERE id = ?`,
      [post.title, post.description, post.author, post.id],
      function (error, result) {
        response.redirect(`/topic/${post.id}`);
      }
    );
  
};

exports.delete_process = function (request, response) {
  var post = request.body;
  db.query(
    "DELETE FROM topic WHERE id =?",
    [post.id],
    function (error, result) {
      if (error) {
        throw error;
      }
      response.redirect('/');
    }
  );
};
```
### - template.js
```javascript
var sanitizeHtml = require('sanitize-html');
module.exports = {
    html: function (title, list, body, control) {
        return `
      <!doctype html>
    <html>
    <head>
      <title>WEB1 - ${title}</title>
      <meta charset="utf-8">
    </head>
    <body>
      <h1><a href="/">WEB</a></h1>
      <a href="/author">author</a>
      ${list}
      ${control}
      ${body}
    </body>
    </html>
      `;
    },
    list: function (topics) {
        var list = "<ul>";
        var i = 0;
        while (i < topics.length) {
            list += `<li><a href="/topic/${topics[i].id}">${sanitizeHtml(topics[i].title)}</a></li>`;
            i++;
        }
        list = list + "</ul>";
        return list;
    },authorSelct:function(authors, author_id){
      var tag = "";
        var i = 0;
        while(i < authors.length){
          var selected = '';
          if(authors[i].id === author_id){
            selected = ' selected'; 
          }
          tag += `<option value="${authors[i].id}"${selected}>${sanitizeHtml(authors[i].name)}</option>`;
          i++;
        }
        return`
        <select name="author">
        ${tag}
        </select>
        `
    },authorTable:function(authors){
      var tag = '<table>';
        var i = 0;
        while(i < authors.length){
            tag +=`
                    <tr>
                        <td>${sanitizeHtml(authors[i].name)}</td>
                        <td>${sanitizeHtml(authors[i].profile)}</td>
                        <td><a href="/author/update?id=${authors[i].id}">update</a></td>
                        <td><form action="/author/delete_process" method="post">
                        <input type="hidden" name="id" value="${authors[i].id}">
                        <input type="submit" value="delete">
                        </form>
                        </td>
                    </tr>
                    `
            i++;
        }
        tag +='</table>';
        return tag;
    }
};

```