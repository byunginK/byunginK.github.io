---
layout: post
title:  "Node JS Web Express framwork"
date:   2021-01-03
categories: [web]
---

# Node Express 활용

#### 우선 npm에서 expree -S 으로 install 해준 후 프로젝트에 의존성을 주입해 준다.
```json
 "dependencies": {
    "express": "^4.17.1",
    "mysql": "^2.18.1",
    "sanitize-html": "^2.3.0"
  }
```

#### 그후 모듈을 가져와서 사용 하면 되는데, 이전에 main.js에서 코딩했던 내용을 리펙토링하여 expree를 적용하도록 하였다.

##### 모듈 가져오기
```javascript
const express = require('express')//모듈을 가져온다
const app = express();  //app에 모듈의 함수를 넣어준다
```

#### 아까 express를 넣어준 app을 활용하여 get, post의 방식으로 라우팅이 가능하다. 그후 response.send를 통해 브라우저에 데이터를 넘길 수 있다.
```javascript
//route, routing 길을 찾게 해준다.
app.get('/',(req,res)=>res.send('Hello world'))
//아래와 같은 문법
app.get('/',function(req,res){
   return res.send('Hello World')
})
```
#### 이후 3000번 포트에 리스너를 사용하여 해당 포트에 반응하도록 설정한다.
```javascript
//3000 번 포트일때 반응하도록 설정
app.listen(3000)
```

#### 아래는 이전 if 문으로 조건을 걸어 라우팅을 했던 main.js를 express 프레임워크를 사용하여 리펙토링한 코드 이다.
```javascript
app.get('/',(req,res)=>topic.home(req, res));
//page 경로와 그에 따른 id를 받을 수 있게 설정
app.get('/page/:pageId',(req,res) =>topic.page(req, res));
// /page/:pageId로 설정하게 되면 request.params.pageId를 통해 page/뒤에오는 값을 얻을 수 있다.
app.get('/create',(req,res)=>topic.create(req, res));
app.post('/create_process',(req,res)=>topic.create_process(req, res));
app.get('/update/:pageId',(req,res)=>topic.update(req, res));
app.post('/update_process',(req,res)=>topic.update_process(req, res));
app.post('/delete_process',(req,res)=>topic.delete_process(req, res));
```
#### 여기서 페이지를 라우팅 해주는 방식이 변경되었으므로 topic.js 와 template.js에 수정이 필요하다.

### template.js의 list 부분 수정
```javascript
list: function (topics) {
        var list = "<ul>";
        var i = 0;
        while (i < topics.length) {
            //페이지를 이동하는 방식이 /page/id 로 변경되었으므로 바꿔준다.
            list += `<li><a href="/page/${topics[i].id}">${sanitizeHtml(topics[i].title)}</a></li>`;
            i++;
        }
        list = list + "</ul>";
        return list;
    }
```    

#### topic.js의 페이지 부분과 업데이트 부분에 id값을 가져오는 부분을 수정해준다.
```javascript
exports.page = function (request, response) {
  // var _url = request.url;
  // var queryData = url.parse(_url, true).query;

  //main에서 설정해준 경로를 통해 /page/:pageId의 값을 받도록 설정
  var pi = path.parse(request.params.pageId).base; 
  db.query(`SELECT *FROM topic`, function (error, topics) {
    if (error) {
      throw error;
    }
    //쿼리문을 작성할 때 ? 로 이용해서 공격적인 인젝션에서 방어할 수 있다. 배열의 순서대로 문자열로 들어가기 때문에(url에 sql injection을 사용하여 테이블을 삭제하거나 그런 행위)
    db.query(
      `SELECT * FROM topic LEFT JOIN author ON topic.author_id = author.id WHERE topic.id =?`,
      [pi],//?에 값을 넣어줄  부분에 pi를 넣어준다.
      function (error2, topic) {
        if (error2) {
          throw error2;
        }
        var title = topic[0].title;
        var description = topic[0].description;
        var list = template.list(topics);
        var html = template.html(
          title,
          list,
          `<h2>${sanitizeHtml(title)}</h2>${sanitizeHtml(description)}
              <p>by ${sanitizeHtml(topic[0].name)}</p>
              `,
          `<a href="/create">create</a> <a href="/update/${pi}">update</a>
                <form action="/delete_process" method="post">
                  <input type="hidden" name="id" value="${pi}">
                  <input type="submit" value="delete">
                </form>`
        );
        response.send(html);
      }
    );
  });
};

exports.update = function (request, response) {
  // var _url = request.url;
  // var queryData = url.parse(_url, true).query;

  //main에서 설정해준 경로를 통해 /page/:pageId의 값을 받도록 설정
  var pi = path.parse(request.params.pageId).base;
  db.query(`SELECT * FROM topic`, function (error, topics) {
    if (error) {
      throw error;
    }
    db.query(
      `SELECT * FROM topic WHERE id =?`,
      [pi], //?에 값을 넣어줄  부분에 pi를 넣어준다.
      function (error2, topic) {
        if (error2) {
          throw error2;
        }
        db.query(`SELECT * FROM author`, function (error2, authors) {
          var list = template.list(topics);
          var html = template.html(
            sanitizeHtml(topic[0].title),
            list,
            `<form action="/update_process" method="post">
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
              <p><input type="submit"></p>`,
            `<a href="/create">create</a> <a href="/update/${topic[0].id}">update</a>`
          );
          response.send(html);
        });
      }
    );
  });
};

```