---
layout: post
title:  "Node JS Web 1"
date:   2020-12-23
categories: [web]
---
# Node JS Web Application
### 기본 웹 서버를 만들기 위한 node 모듈
```javascript
//서버를 만들기 위한 node 내장 모듈
var  http = require("http");
//파일을 쓰고/읽기 위한 내장 모듈
var  fs = require("fs");
//url을 얻기위한 내장 모듈
var  url = require('url');
//url의 쿼리를 얻기위한 내장 모듈
var  qs = require('querystring');
//탬플릿의 객체화 하여 불러오기
var  template = require('./lib/template.js');
//보안을 위해 접근 방법
var  path = require('path'); 
//npm을 이용하여 출력시 태그를 걸러주는 라이브러리
var  sanitizeHtml = require('sanitize-html');
```
### 서버를 생성하기 위한 http.createServer에 메인 홈 생성
```javascript
var  app = http.createServer(function (request, response) {

	var  _url = request.url; //서버의 url을 가져옴
	var  queryData = url.parse(_url,true).query;//가져온 url을 파라미터로 넣고 쿼리문을 가져옴
	var  pathname = url.parse(_url,true).pathname;//path(쿼리 제외)
	if(pathname === '/'){	//제대로된 url로 들어왔을경우
	
		if(queryData.id === undefined){	// url에서 뒤에 쿼리가 없는경우(메인 홈 화면)

			fs.readdir('./data',function(error,filelist){ // data디렉토리에서 파일들을 읽어옴 (list)

				var  title = 'Welcome';
				var  description = 'Hello Node.js';
				var  list = template.list(filelist); // 객체의 list를 가져옴
				var  html = template.html(title,list,`<h2>${title}</h2>${description}`,`<a href="/create">create</a>`); // html 폼에 파라미터를 넣고 변수에 담아줌
				response.writeHead(200);
				response.end(html);// 완성된 html을 웹 브라우저에 뿌려줌
			})
		}
```

### 홈 화면이 아닐 경우 뿌려주는 화면
```javascript
else{
	fs.readdir('./data',function(error,filelist) {
		var  filteredId = path.parse(queryData.id).base;//보안을 위해 경로를 읽어서 넣어주는 방법
		fs.readFile(`data/${filteredId}`,'utf-8',function(err,description){
			var  title = queryData.id;
			var  sanitizedTitle = sanitizeHtml(title);//title을 url로 읽지 못하게 해줌
			var  sanitizedDescription = sanitizeHtml(description,{allowedTags:['h1']});//h1은 허용한다는 기능
			var  list = template.list(filelist);
			var  html = template.html(sanitizedTitle,list,`<h2>${sanitizedTitle}</h2>${sanitizedDescription}`,
			`<a href="/create">create</a> <a href="/update?id=${sanitizedTitle}">update</a>
			<form action="delete_process" method="post">
			<input type="hidden" name="id" value="${sanitizedTitle}">
			<input type="submit" value="delete">
			</form>`);//hidden을 넣어주는 이유는 삭제할때 삭제할 파일 이름을 전달 하기 위해
			response.writeHead(200);
			response.end(html);
		});
	});
}
```
### 파일 생성시 화면 
##### create , update ,  delete 버튼이 사라지고 입력 창만 출력
```javascript
else  if(pathname === '/create'){

	fs.readdir('./data',function(error,filelist){

		var  title = 'WEB-create';
		var  list = template.list(filelist);
		var  html = template.html(title,list,`
		<form action="/create_process" method="post">
		<p><input type="text" name ="title" placeholder="title"></p>
		<p><textarea name="description" placeholder="description"></textarea></p>
		<p><input type="submit"></p>
		`,'');
		response.writeHead(200);
		response.end(html);

	})
}
```
### 입력한 내용을 생성하는 로직
```javascript
else  if(pathname === '/create_process'){

	var  body = '';
	request.on('data',function(data){//post로 전달한 데이터를 callback을 통해 return한다
		body += data;
	});

	request.on('end',function(){ // 데이터 전달이 끝나면 실행되며, 쿼리스트링을 통해 전달받은 내용을 접근 할 수 있다.

		var  post = qs.parse(body);
		var  title = post.title;
		var  description = post.description;

		fs.writeFile(`data/${title}`,description,'utf8',function(err){

			response.writeHead(302,{Location:  `/?id=${title}`});//302은 변경된 페이지를 의미이며 {Location}은redirect해준다
			response.end();

		})
	});
}
```
### 업데이트를 하기위한 페이지 화면
##### 이전 모듈들 사용과 비슷하다.
```javascript
else  if(pathname ==='/update'){

	fs.readdir('./data',function(error,filelist) {

		var  filteredId = path.parse(queryData.id).base;
		fs.readFile(`data/${filteredId}`,'utf-8',function(err,description){
			var  title = queryData.id;
			var  list = template.list(filelist);
			var  html = template.html(title,list,`
			<form action="/update_process" method="post">
			<input type="hidden" name="id" value="${title}"
			<p><input type="text" name ="title" placeholder="title" value="${title}"></p>
			<p><textarea name="description" placeholder="description">${description}</textarea></p>
			<p><input type="submit"></p>`,`<a href="/create">create</a> <a href="/update?id=${title}">update</a>`);
	
			response.writeHead(200);
			response.end(html);
		});
	});
}
```
### 업데이트를 진행 한후 데이터 처리
```javascript
else  if(pathname ==='/update_process'){

	var  body = '';
	request.on('data',function(data){
		body += data;
	});

	request.on('end',function(){
	var  post = qs.parse(body);
	var  id = post.id;
	var  title = post.title;
	var  description = post.description;
	fs.rename(`data/${id}`,`data/${title}`,function(err){
		fs.writeFile(`data/${title}`,description,'utf8',function(err){
				response.writeHead(302,{Location:  `/?id=${title}`});//302은 변경된 페이지고 redirect
				response.end();

			})

		});

	});

}
```
### 파일을 삭제한 후 처리
```javascript
else  if(pathname ==='/delete_process'){
	var  body = '';
	request.on('data',function(data){
	body += data;
	});

	request.on('end',function(){
		var  post = qs.parse(body);
		var  id = post.id;
		var  filteredId = path.parse(id).base;
		fs.unlink(`data/${filteredId}`,function(){
			response.writeHead(302,{Location:  `/`});//302은 변경된 페이지고 redirect
			response.end();
		})
	});
}
```
### 제대로 되지 않은 url로 접근 하였을때 오류 페이지 
```javascript
	else{

		response.writeHead(404);//웹서버가 응답할때 200은 브라우저는 성공 이라는 의미, 404 는 에러
		response.end('Not found');
	}
});
app.listen(3000);
```
#### 마지막 app.listen(3000)은 포트 3000으로 서버를 지켜보고 있다라는 의미이며 3000포트를 사용한다는 의미 이다.


## Template.js (객체)
#### `var  template = require('./lib/template.js');` 맨 위에 선언한 모듈 불러오기에 사용되었다.

```javascript
module.exports = {

	html:  function (title, list, body, control) {
	return  `
	<!doctype html>
	<html>
	<head>
	<title>WEB1 - ${title}</title>
	<meta charset="utf-8">
	</head>
	<body>
	<h1><a href="/">WEB</a></h1>
	${list}
	${control}
	${body}
	</body>
	</html>
	`;
	},
	list:  function (filelist) {
		var  list = "<ul>";
		var  i = 0;
		while (i < filelist.length) {
			list += `<li><a href="/?id=${filelist[i]}">${filelist[i]}</a></li>`;
			i++;
		}
		list = list + "</ul>";
		return  list;

	},

};
```
