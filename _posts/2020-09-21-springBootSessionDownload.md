﻿---
layout: post
title:  "Web Spring Boot Session & File Download"
date:   2020-09-21
categories: [web]
---

# RESTful 방식
- 서버 2개로 백엔드와 프론트를 두개로 나누어서 진행
- 비동기로 백엔드와 프론트를 연결할 경우 세션에 객체를 넣어서 값을 전달할 수 없다
- 따라서 클라이언트서버에서 값을 전달 할 수 있는 방식으로 전달 해야 한다

### 컨트롤에서 MemberDto를 return으로 값을 넘겨준다
```java
@RequestMapping(value = "/login", method = RequestMethod.POST)
public MemberDto login(MemberDto dto, RedirectAttributes re) {
  System.out.println("login()");
  MemberDto member = service.login(dto);

  return member;
}
```
### 클라이언트 서버에서 sessionStorage를 이용해서 저장한다
```html
<script type="text/javascript">
$("#login").click(function(){
	$.ajax({
		url:"http://localhost:3000/login",
		type:"post",
		data:{id:$("#id").val(), pwd:$("#pwd").val()},
		success:function(data){
			//alert("success");
			if(data==''){
				alert("id나 password가 틀렸습니다");
			}
			else{
				alert("로그인 성공");
				
				sessionStorage.setItem("login", JSON.stringify(data));  //문자열로 세션에 추가
				/*
				let login = JSON.parse(sessionStorage.getItem("login"));  //나중에 객체를 받을때 json방식으로 접근할 수있게 바꿔서 넣어준다
				alert(login.id);
				alert(login.name);
				*/
				location.href="bbslistpage.html";
			}
		},
		error:function(){
			alert("error");
		}
	});
});
```

## EX) 게시판 디테일 페이지로 변경 할때
- seq번호를 가지고 가야하는데 서버를 2개를 사용, 비동기로 할 경우 값 넘기는 방식중 url주소에 파라미터를 넣어 그 값을 읽어와야 한다.
```javascript
function getlistData(pNumber){
	$.ajax({
		url:"http://localhost:3000/getBbsPageList",
		type:"get",
		data:{"nowPage":pNumber, "recordCountPerPage":10, "choice":$("#choice").val(), "searchWord":$("#searchWord").val()},
		success:function(list){
			//alert(list);
			
			$("tbody").html("");
			$.each(list, function(i, item){
				
				let app = "<tr>"
						+"	<th>"+ ( i+ 1) + "</th>"
						+"	<td>"+ getArrow(item.depth)+ "<a href='bbsdetail.html?seq="+item.seq + "'>" + item.title + "</a></td>"  //seq를 넣어 보내준다
						+"	<td>"+ item.id + "</td>" 
						+"</tr>";
						
				$("tbody").append(app);
			});
		},
		error:function(){
			alert('error');
		}
	});
}
```

### url 주소에 넘어오는 파라미터를 이용하여 값을 얻어오는 방식사용
##### * 자바 스크립트 방식으로 location.search를 하게되면 ?를 포함한 그 뒤의 url주소를 가져온다


```javascript
<script type="text/javascript">
/* url 주소를 통해 파라미터를 받는 부분 */
let urlParams = location.search.split(/[?&]/).slice(1).map(function(paramPair) {
    return paramPair.split(/=(.+)?/).slice(0, 2);
}).reduce(function(obj, pairArray) {
    obj[pairArray[0]] = pairArray[1];
    return obj;
}, {});

</script>
```
##### * header에 위처럼 선언을 해주면 아래 처럼 키값으로 접근이 가능하다


```javascript
let seq = urlParams.seq;
```

# 파일 다운로드
##### Controller


```java
package bit.com.a.controller;

import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;

import javax.servlet.ServletContext;
import javax.servlet.http.HttpServletRequest;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.InputStreamResource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import bit.com.a.util.MediaTypeUtiles;

@RestController
public class FileController {
	
	@Autowired
	private ServletContext servletContext;

	@RequestMapping(value = "/fileUpload", method = RequestMethod.POST)
	public String fileUpload(@RequestParam("uploadFile")MultipartFile uploadFile, HttpServletRequest req) {
		System.out.println("FileController fileUpload");
		
		//경로
		String uploadPath = req.getServletContext().getRealPath("/upload");
		
		//String uploadPath = "d:\\tmp";
		
		String filename = uploadFile.getOriginalFilename();
		String filepath = uploadPath + File.separator + filename; // '/' == file.feparator
		System.out.println("filepath"+filepath);
		
		
		//DB 접근 하는 부분
		
		
		try {
			BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(new File(filepath)));	//파일 생성 -> 파일 아웃풋 -> 버퍼 아웃풋
			bos.write(uploadFile.getBytes());
			bos.close();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
			
			return "file upload fail";
		}
		return "file upload success";
	}
	
  //다운로드
	@RequestMapping(value = "/download") //ResponseEntity 데이터를 한꺼번에 보내줄 수 있다
	public ResponseEntity<InputStreamResource> download(String fileName, HttpServletRequest req)throws Exception{
		System.out.println("FileController fileUpload");
		
		//경로
		String uploadPath = req.getServletContext().getRealPath("/upload");
		
		//String uploadPath = "d:\\tmp";
		
		MediaType mediaType = MediaTypeUtiles.getMediaTypeForFileName(this.servletContext, fileName);
		System.out.println("filename:"+fileName);
		System.out.println("mediaType:"+mediaType);
		
		File file = new File(uploadPath + File.separator+fileName);
		InputStreamResource resource = new InputStreamResource(new FileInputStream(file));
		
		return ResponseEntity.ok()
				.header(HttpHeaders.CONTENT_DISPOSITION, "attachment;filename="+file.getName())
				.contentType(mediaType)
				.contentLength(file.length())
				.body(resource);	//이전 유틸에서 사용 했던 다운로드 창
	}
}
```
### 다운로드시 필요한 util


```java
package bit.com.a.util;

import javax.servlet.ServletContext;

import org.springframework.http.MediaType;

public class MediaTypeUtiles {

	public static MediaType getMediaTypeForFileName(ServletContext servletContext, String filename) {
		
		String mimeType = servletContext.getMimeType(filename);
		
		try {
			MediaType mediaType = MediaType.parseMediaType(mimeType);
			return mediaType;	//미디어 타입으로 넘겨준다
		}catch (Exception e) {
			return MediaType.APPLICATION_OCTET_STREAM;
		}
	}
}
```
### html 보여지는 


```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
</head>
<body>

<h1>Upload file</h1>

<form id="upload_file_form">
	<input type="file" id="upload_file" name="uploadFile">
	<button type="button" id="upload_file_btn">Upload</button> 
</form>

<script type="text/javascript">
$(document).ready(function(){
	$("#upload_file_btn").click(function(){
		
		$.ajax({
			url:'http://localhost:3000/fileUpload',
			type:'POST',
			data:new FormData($("#upload_file_form")[0]),
			enctype:'multipart/form-data',
			processData:false,
			contentType:false,
			cache:false,
			success:function(data){
				alert(data);
			},
			error:function(){
				alert("error");
			}
			
		});
	});
});

</script>
<br>

<h1>Down load</h1>

<button type="button" id="download">Download</button>

<script type="text/javascript">

$("#download").click(function(){
	/* 파일 다운로드는 ajax로 하면 안된다
	$.ajax({
		url:'http://localhost:3000/download',
		type:"GET",
		data:{fileName:"backa.jpg"},
		success:function(){
			alert("success");
		},
		error:function(){
			alert('error');
		}
	});
	*/
	location.href="http://localhost:3000/download?fileName="+"backa.jpg";	//링크를 걸어주어야 한다
});

</script>
</body>
</html>
```
