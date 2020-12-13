---
layout: post
title:  "인스타그램 그래픽 API 사용"
date:   2020-12-13
categories: [web]
---
# Instagram API 

##### 홈페이지에 인스타그램 사진이 업로드 될때마다 이미지와 주소를 가지고와 업데이트하고 싶었다. 

##### rss를 이용하여 하려하였으나 무료계정은 인스타그램을 지원하지 않는 경우가 대부분이였으며, 유료로 하기에는 개인적으로 투자하고 싶지는 않았다.(회사홈페이지에 적용해야해서)

##### 따라서 인스타그램 api를 사용하려 하였으나 최근 인스타그램의 api 정책이 변경되어 예제가 많이 없었고 데이터를 가져오는 인증이 까다롭게 느껴졌으나 몇시간의 삽질 끝에 데이터를 json 형태로 가져오는것을 성공하였다. 

##### 기본적인 인스타그램api를 사용하기 위한 설정은 아래 링크주소를 보고 따라하였다.
[페이스북 개발자 설정 및 인스타 api 사용 준비 설정](https://studio-jt.co.kr/%EC%9D%B8%EC%8A%A4%ED%83%80%EA%B7%B8%EB%9E%A8-api-instagram-graph-api-instagram-api-v2-%EC%97%B0%EB%8F%99%EC%9D%BC%EC%A7%80/)

##### 그리고 내가 적용한 ajax를 통해 구현한 코드이다. 자세한 내용은 페이스북 document를 확인해야할것 같다. (우선은 급한대로 아래 링크를 사용)

```javascript
<script>

$(document).ready(function(){

var  token = 'IGQVJYV0hnb0RU.....'; //페이스북 개발자 모드에서 generate token으로 생성된 토큰 적용

	$.ajax({

		url:"https://graph.instagram.com/me/media?access_token=" + token + "&fields=id,caption,media_type,media_url,thumbnail_url,permalink",
		type:'get',
		dataType:'json',
		success:function(data2){
		//이미지 url을 가져오는 접근
		console.log(JSON.stringify(data2.data[0].media_url));
	
		}

	})

})

</script>
```

정확한 파라미터에 대한 설명은 아래 문서를 참고하자
[페이스북 document](https://developers.facebook.com/docs/instagram-api/overview)
