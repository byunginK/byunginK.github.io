﻿
---
layout: post
title:  "Web Spring 유튜뷰 크롤링"
date:   2020-09-22
categories: [web]
---

# 유튜브 동영상 크롤링 

### 1. 뷰에서 입력받은 검색어를 처음 받아들이는 컨트롤
```java
package bit.com.spring.controller;

import java.util.List;

import javax.servlet.http.HttpSession;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import bit.com.spring.dto.MemberDto;
import bit.com.spring.dto.Youtube;
import bit.com.spring.dto.YoutubeSave;
import bit.com.spring.service.YoutubeService;
import bit.com.spring.util.YoutubeParser;

@Controller
public class YoutubeController {

	@Autowired
	private YoutubeParser youtubeParser;
	@Autowired
	YoutubeService service;
	
	@RequestMapping(value = "youtube.do", method = {RequestMethod.GET, RequestMethod.POST})
	public String youtube(String s_keyword, Model model) {  //s_keyword는 검색어
		model.addAttribute("doc_title", "유튜브");
		
		// 검색 기능이 있어야한다 ***
		if(s_keyword != null && !s_keyword.equals("")) {
			
			List<Youtube> getTtitles = youtubeParser.getTitles(s_keyword);
			
			model.addAttribute("yulist", getTtitles);
			model.addAttribute("s_keyword", s_keyword);
		}
		
		
		return "youtube.tiles";
	}
```  
### 2. 아래 autowired를 해주기위해 클래스를 생성해 주어야 한다. 또한, youtubeParser를 만들어 크롤링해줄 url과 크롤링할 값들을 추려낸다.
```java
@Autowired
	private YoutubeParser youtubeParser;
```

## - youtubeParser
```java
package bit.com.spring.util;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.UnsupportedEncodingException;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLEncoder;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

import org.json.JSONObject;
import org.springframework.stereotype.Component;

import bit.com.spring.dto.Youtube;

@Component //클래스가 자동 생성이 된다 
public class YoutubeParser {
	
	String urls = "https://www.youtube.com/results?search_query=";
	Map<String , String> htmls = new HashMap<String, String>();
	
	
	//제일 중요 (가장 핵심// 크롤링이 이루어지는 부분)
	public Map<String, String> search(String keyword) {
		System.out.println("YoutubeParser search");
		
		htmls.clear();//map을 비운다  (이거를 안넣고 , 멤버변수에 있는 Map을 여기 안에다가 생성해도 괜찮다)
		BufferedReader br = null;//읽어들인 데이터를 모아주는 역할
		
		// 웹에서 데이터를 취득하기 위한 list 초기화
		JsonUtils.list.clear();
		JsonUtils.titleList.clear();
		
		
		try {
			String ss = URLEncoder.encode(keyword, "utf-8");  //검색어 인코딩
			//System.out.println(ss);
			
			//import java.net.URL; url과 검색어를 합해서 동영상의 주소를 만들어 준다 (해당 주소에서 검색된 동영상들의 값들을 불러 올 것이다)
			URL url = new URL(urls + ss);
			
			br = new BufferedReader(new InputStreamReader(url.openStream(),"utf-8")); //위의 검색된 url의 모든 데이터(html문서를 통째)로 읽어들인다
			
			String msg="";
			int pos = 0;
			String text = "";
			while((msg = br.readLine()) != null) {  // 읽어들은 html문서를 읽는다
				text += msg.trim();	//검색이 된 url이 다 text로 들어간다
			}
			//System.out.println(text);		//responseContext 검색어
			
			//읽을 시작 위치
			pos = text.trim().indexOf("\"responseContext\""); //해당 단어는 html문서의 data값중에서 내가 취득할 data{}안에 들어가있는 단어중 수가 제일적어서 저것으로 자른다
		//	System.out.println(pos);
			
			// 끝 위치
			int endpos = text.indexOf("};", pos); //끝날지점 을 찾는다
		//	System.out.println(endpos);
			
			String str = text.substring(pos-1, endpos+1);	//json 데이터로 끌어오는거다 {responseContext ~ } 이렇게 가지고 온다
		//	System.out.println(str);
			
			JSONObject json = new JSONObject(str);	//json 형태로 바꿔준다.
			
			JsonUtils.jsonToMap(json);  //값들의 key 값들을 map에 넣어주고 list화 시킨다. 이부분은 아래 JsonUtils에서 더 자세히
			for (int i = 0; i < JsonUtils.titleList.size(); i++) {
				//System.out.println(JsonUtils.list.get(i));  이건 비디오 고유 코드
				//System.out.println(JsonUtils.titleList.get(i)); 이건 동영상 제목
				
				htmls.put(JsonUtils.list.get(i), JsonUtils.titleList.get(i)); 
			}
			
			
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		} catch (MalformedURLException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		
		
		return htmls;
	}
	
	public List<Youtube> getTitles(String keyword) {
		
	//	search(keyword);
		
		List<Youtube> list = new ArrayList<Youtube>();
		Map<String, String> map = search(keyword);  //아까 비디오 코드와 제목을 넣은 map을 다시 넣어준다
		
		Iterator<String> it = map.keySet().iterator();  // 하나씩 꺼내면서 title, url 를 리스트에 넣어준다
		while(it.hasNext()) {
			String url = it.next();
			Youtube dto = new Youtube(map.get(url),url,""); //map.get(값)을 넣으면 key를 return한다
			list.add(dto);
		}
		
		return list;
	}
}
```
### 위에서 사용된 JsonUtil을 살펴본다
```java
package bit.com.spring.util;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

import org.json.JSONArray;	//처음 jsonObject simple이 아닌 org.json으로 해야한다
import org.json.JSONObject;



// https://www.it-swarm.dev/ko/java/json-%EB%AC%B8%EC%9E%90%EC%97%B4%EC%9D%84-hashmap%EC%9C%BC%EB%A1%9C-%EB%B3%80%ED%99%98/1045437367/

//JSONObject를 Map과 List로 저장하는  class

// http://json.parser.online.fr/ 	json 정렬해주는 사이트
public class JsonUtils {
	
	//원하는 값을 취득하기 위한 list를 준비
	public static List<String> list = new ArrayList<String>();
	public static List<String> titleList = new ArrayList<String>();
		
	

	public static Map<String, Object> jsonToMap(JSONObject json) {
        Map<String, Object> retMap = new HashMap<String, Object>();

        if(json != null) {
            retMap = toMap(json);
        }
        return retMap;
    }

    public static Map<String, Object> toMap(JSONObject object) {
        Map<String, Object> map = new HashMap<String, Object>();

        Iterator<String> keysItr = object.keySet().iterator();
        while(keysItr.hasNext()) {
            String key = keysItr.next();
            Object value = object.get(key);

            if(value instanceof JSONArray) {
                value = toList((JSONArray) value);
            }

            else if(value instanceof JSONObject) {
                value = toMap((JSONObject) value);
            }
            map.put(key, value);
            
            //데이터를 취득 할 수 있는 부분 , key
            
           // System.out.println("key:"+key);
            //videoId
            if(key.equals("videoId")) { // 전체 key값을 받아올때 비디오코드와 같은게 있는지 살핀다.
            	Iterator<String> it = list.iterator();  // 중복되는 비디오 코드를 제거
            	while(it.hasNext()) {
            		String k = it.next();
            		if(k.equals(value)) {
            			it.remove();
            		}
            	}
            	list.add((String)value);  // 비디오 코드를 리스트에 넣어준다
            }
            
            
            //key 값이 title 인것만 출력
            if(key.equals("title")) {
            	//System.out.println("key:"+key + "value:"+value);
            	
            	if(value.toString().contains("accessibility")) {  //title 안에서 accessibility 이 들어간 문장에 title명이 있다
            	//	System.out.println("key:"+key + " value:"+value);
            		int start = value.toString().indexOf("text=");
            		int end = value.toString().indexOf("}]");
            		
            		String title = value.toString().substring(start+5, end);  //text= 이 5글자라서 +5
            	//	System.out.println(title);
            		
            		// 중복된 제목을 추가 하지 않도록
            		Iterator<String> it = titleList.iterator();
            		while(it.hasNext()) {
            			String k = it.next();
            			if(k.equals(title)) {
            				it.remove();
            			}
            		}
            		
            		titleList.add(title);
            	}
            }
        }
        return map;
    }

    public static List<Object> toList(JSONArray array) {
        List<Object> list = new ArrayList<Object>();
        for(int i = 0; i < array.length(); i++) {
            Object value = array.get(i);
            if(value instanceof JSONArray) {
                value = toList((JSONArray) value);
            }

            else if(value instanceof JSONObject) {
                value = toMap((JSONObject) value);
            }
            list.add(value);
        }
        return list;
    }
}
```
- 실질적으로 JsonUtil에서는 json을 Map으로 바꿔주는 코드만 사용하며, 그안에서도 return받은 map을 사용하는것이 아니라
중간에 걸러진 key값을 이용해 단어들을 슬라이싱하여 얻어낼 값들을 찾아내고 list에 따로 다시 담아준다.

### 아까 youtubeParser에서 마지막에 youtubeDto 에 값들을 넣어 객체를 생성하고 list에 넣어 뷰에서 뿌려줄 수 있게 하였다.
- youtubeDto
```java
package bit.com.spring.dto;

import java.io.Serializable;

public class Youtube implements Serializable {

	private String title;
	private String url;
	private String img;
	
	public Youtube() {
	}

	public Youtube(String title, String url, String img) {
		super();
		this.title = title;
		this.url = url;
		this.img = img;
	}

	public String getTitle() {
		return title;
	}

	public void setTitle(String title) {
		this.title = title;
	}

	public String getUrl() {
		return url;
	}

	public void setUrl(String url) {
		this.url = url;
	}

	public String getImg() {
		return img;
	}

	public void setImg(String img) {
		this.img = img;
	}

	@Override
	public String toString() {
		return "Youtube [title=" + title + ", url=" + url + ", img=" + img + "]";
	}
}
```

### view 에서는 아래를 참고하여 url의 값을 iframe에 넣어 유튜브를 볼 수 있게 할 수 있다.
```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>     
    
<div class="box_border" style="margin-top: 5px; margin-bottom: 10px">
<form id="frmForm" method="post">
<table style="margin-left: auto; margin-right: auto; margin-top: 3px; margin-bottom: 3px;">
<tr>
	<td>검색:</td>
	<td style="padding-left:5px">
		<input type="text" id="s_keyword" name="s_keyword" value="${empty s_keyword?'':s_keyword }">
	</td>

	<td style="padding-left: 5px">
		<span class="button blue">
			<button type="button" id="_btnSearch">검색</button>
		</span>
	</td>
	
</tr>

</table>
</form>
</div>

<div id="_youtube_">
	<iframe id="_youtube" width="640" height="360" src="http://www.youtube.com/embed/" allowfullscreen frameborder="0">
	
	</iframe>

</div>

<table class="list_table" style="width: 85%">
<colgroup>
	<col style="width: 70px"><col style="width: auto"><col style="width: 100px"><col style="width: 30px">
</colgroup>

<thead>
	<tr>
		<th>번호</th><th>제목</th><th>유뷰트 고유 번호</th><th>저장</th>
	</tr>

</thead>

<tbody>
	<c:if test="${empty yulist }">
	<tr>
		<td colspan="4">작성된 목록이 없습니다</tr>
	</c:if>
	
	<c:forEach items="${yulist }" var="you" varStatus="vs">
		<tr class="_hover_tr">
			<td>${vs.count }</td>
			<td style="text-align: left;">
				<div class="c_vname" vname="${you.url }">${you.title }</div>
			</td>
			<td>${you.url }</td>
			<td>
				<img alt="" src="image/save.png" class="ck_seq" id="${you.url }" loginIn="${login.id }" title="${you.title }">
			</td>
		</tr>
	</c:forEach>

</tbody>
</table>

<script>
$(document).ready(function(){
	$("#_youtube_").hide();

	$("._hover_tr").mousemove(function() {
		$(this).children().css("background-color","#f0f5ff");
	}).mouseout(function() {
		$(this).children().css("background-color","#ffffff");
	});
});

$("#_btnSearch").click(function(){
	$("#frmForm").attr('action',"youtube.do").submit();
	
});

$(".c_vname").click(function(){
	$("#_youtube_").show();

	$("#_youtube").attr("src","http://www.youtube.com/embed/"+$(this).attr("vname"));
});

$('.ck_seq').click(function(){

	let id = $(this).attr("loginIn");
	let title = $(this).attr("title");
	let url = $(this).attr("id");

	//alert(url);

	$.ajax({
		url:"./youtubesave.do",
		type:'post',
		async:true,	//비동기설정
		data:{"id":id,"title":title,"url":url},
		success:function(msg){
			alert(msg);
		},
		error:function(){
			alert("error");
		}


	});
	
});

</script>
```



