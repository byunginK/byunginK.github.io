---
layout: post
title:  "Node JS Web Express framwork 2"
date:   2021-01-16
categories: [web]
---

# Node Express 정적 서비스 사용

### 사진, 동영상과 같이 정적 서비스를 가져오는 Node Express 사용법

##### 우선 정적 파일들을 보관하는 디렉토리를 생성하여준다.
![image](https://user-images.githubusercontent.com/65350890/104805880-978c2100-5816-11eb-8c61-dbc8ab8e579d.png)

##### public 디렉토리에 images라는 폴더안에 hello.jpg라는 이미지가 있고 나는 해당 이미지를 웹 화면에 출력을 하려한다.

##### Express의 내장함수를 사용하여 이미지 경로에 접근하여 사용 할 수있다.<br><br>
#### - main.js
```javascript
//express 정적 서비스를 가져오는 내장함수 [express.static(접근경로)]
app.use(express.static('public'));
```
>##### 위와 같이 최초 디렉토리만 경로를 잡아주면 된다.

##### <br><br>이제 내가 이미지를 넣고 싶은 웹 페이지에 이미지를 넣어주면 된다.
#### - topic.js
```javascript
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
      `<a href="/create">create</a>`
    );
    response.send(html);
  });
};
```
>##### `<img src="/images/hello.jpg" width="300px" style="display:block; margin-top:10px">`의 src의 경로에서 images/hello.jpg만 잡아준 이유는 아까 public 디렉토리는 express가 잡아줬기 때문에 이미지가 출력되는것을 알수 있다.
![image](https://user-images.githubusercontent.com/65350890/104806061-fd2cdd00-5817-11eb-832b-9cbc2637ebf6.png)