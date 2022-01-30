---
layout: post
title:  "Node JS Web Express framwork 3"
date:   2021-01-16
categories: [web]
---
# Node Express 예외처리 설정

#### 404 , 500 에러가 발생하였을때 사용자 페이지 또는 사용자 문구를 표현하고 싶을때 Express에 에러가 발생하였을때 설정하는 미들웨어를 사용할 수 있다.

## 1. 404 에러 처리
#### 페이지(경로)가 없을경우 출력되는 페이지(문구)이다.
##### Express의 미들웨어를 통해 제어할 수 있으며, 마지막 하단에 설정을 하도록 한다.<br><br>

#### - main.js
```javascript
//404 에러가 떴을때 응답하도록 설정
app.use(function(req,res,next){
  res.status(404).send('sorry not found')
})
```
>##### 위 설정은 app.listen 바로 위에 마지막 부분에 설정하였으며, 만약 페이지를 찾을 수 없다면, *'sorry not found'* 라는 문구가 있는 페이지가 출력된다.

## <br> 2. 500 에러 처리
#### 내부 로직 실행 중 에러가 발생(쿼리 에러 등)이 되면, 출력하는 에러 페이지를 설정 할 수 있다.
##### 해당 설정은 404 에러를 설정했던 곳 아래에 해준다.<br><br>

#### - main.js
```javascript
//에러가 났을때(404가 아닌 쿼리문이나 내부 에러가 났을때)
app.use(function(err, req, res, next){
  console.error(err.stack)
  res.status(500).send('Something broke')
})
```


#### - topic.js
```javascript
exports.page = function (request, response) { 
  var pi = path.parse(request.params.pageId).base; //main에서 설정해준 경로를 통해 /page/:pageId의 값을 받도록 설정
  db.query(`SELECT *FROM topic`, function (error, topics) {
    if (error) {
      throw error;
    }
    //쿼리문을 작성할 때 ? 로 이용해서 공격적인 인젝션에서 방어할 수 있다. 배열의 순서대로 문자열로 들어가기 때문에(url에 sql injection을 사용하여 테이블을 삭제하거나 그런 행위)
    db.query(
      `SELECT * FROM topic LEFT JOIN author ON topic.author_id = author.id WHERE topic.id =?`,
      [pi],
      function (error2, topic) {
        if (error2) {
            //만약 에러가 발생하면 err를 설정해놓은 미들웨어로 가게끔 설정
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
            `<a href="/create">create</a> <a href="/update/${pi}">update</a>
                  <form action="/delete_process" method="post">
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
```

>##### 만약 에러가 났을때 500 에러를 담당하고 있는 미들웨어로 보내줄 수 있도록 설정해 준다. <br>`if (error2) {  next(err);   }`

#### <br>위의 예시는 pageId에 존재하지 않는 값이나 쿼리문에 에러가 발생하게 되면 *Something broke*라는 문구의 페이지가 출력된다.