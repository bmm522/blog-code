---
# 들어가기 앞서서 ..

_프로젝트 당시 로그인을 맡았고, 기존에 소셜로그인을 oauth2 클라이언트 라이브러리를 통해 구현해본 경험이 있었으나, 쌩 구현은 해본 경험이 없어서 oauth2의 개념정리를 할 겸 처음부터 끝까지 구현해 보았다._

---

_기본적인 oauth2 개념은 말 그대로 대신 로그인을 해주는 것이다. oauth2 서비스를 제공하는 플랫폼에서 사용자의 자격증명을 대신 함으로써 우리는 추가적인 자격증명 없이 사용자를 받을 수 있는 것이다. _

_그렇다면 기본적인 oauth2로그인의 흐름이 어떻게 될까? 이 흐름을 가장 잘나타낸 카카오의 디벨로퍼 문서를 보자. 그리고 카카오 소셜로그인을 통해 oauth2로그인의 흐름을 파악해보자._

https://developers.kakao.com/docs/latest/ko/kakaologin/rest-api#request-code-info
(Kakao Developer 문서 홈페이지)

![](https://velog.velcdn.com/images/leon/post/dfb09747-bad0-4654-81ba-6690219d0641/image.png)

**_(Kakao developers 문서에서 가져온 사진입니다.)_**

_딱 이사진만 봣을때 바로 이해할 수 있는 사람이 몇이나 될까? 나는 솔직히 수 많은 레퍼런스 문서들을 봐도 이해가 잘 안됏었다. 딱 이 시퀀스그램을 보고는 이해가 된다면 이 글을 볼 필요가 없다. 나는 이 사진을 보고도 감이 잘 안오는 사람을 위해 글을 작성할 것이다. 기본적인 카카오 로그인을 위한 홈페이지 설정은 다 되어있다고 가정하에 진행할 것이다. _

![](https://velog.velcdn.com/images/leon/post/65ba2212-f9f7-43ac-a03c-0d9752fc90b1/image.png)

_가장 맨 처음인 1번 부터 보자. 우리의 서비스 서버에서 Kakao Auth Server로 GET 요청을 보내라고 되어있다. _

![](https://velog.velcdn.com/images/leon/post/b4c068b4-58cc-4984-b35f-e68c3dee4a6e/image.png)

_1번에 해당하는 부분이 여기 부분에 있다. 이것을 코드로 풀어보자._

```
GET /oauth/authorize?client_id=${REST_API_KEY}&redirect_uri=${REDIRECT_URI}&response_type=code
Host: kauth.kakao.com
```
_ HOST는 말 그대로 요청을 보낼 주소 이고 우리가 보내줘야 할 데이터는 client_id에 해당하는 REST_API_KEY와 redirect_uri에 해당하는 REDIRECT_URI이다. 나는 처음에 redirect_uri이 정말 이해가 안갔었다. redirect_uri는 무엇을 말하느냐? 그것은_ 
![](https://velog.velcdn.com/images/leon/post/1c36c0d3-c7e7-49cf-8896-b947c71d05b8/image.png)
_시퀀스그램의 6번에 나오는데, 우리가 카카오 페이지를 통해 대신 로그인을 한다. 그래서 카카오서버에서 해당 유저가 자격증명이 완료된 사용자이면 우리가 설정한 주소로 자격증명이 완료된 사용자임을 나타내는 인가 코드를 전송해준다. 그러면 우리는 그 코드를 통해 Kakao Resource서버로 요청을 보낼 수 있는 토큰을 가져올 수 있는 것이다. 아직까지 감이 안온다면 코드로 이해를 해보자._

```java
const kakao = () =>{
	window.location.href="https://kauth.kakao.com/oauth/authorize?client_id=${본인의 rest_api_key}&redirect_uri=http://localhost:8080/kakao.lo&response_type=code"
}
```

_나 같은 경우는 단순히 GET요청이기 때문에 링크를 걸어주었다. 그리고 client_id 부분에 kakao developer페이지에 있는 rest_api_key를 넣어주었고, 로그인이 성공했을 시에 코드를 받을 주소를 redirect_uri에 넣어주었다. 그렇다면 해당 링크를 클릭했을 때 어떻게 될까?_

![](https://velog.velcdn.com/images/leon/post/5e3f1b6c-6acc-4a0f-a40b-36856e952597/image.png)

_보면 localhost에서 열린 것이 아닌 카카오 서버에서 열린 것을 볼 수 있다. 여기서 사용자가 로그인이 성공하면 어떻게 될까? _
![](https://velog.velcdn.com/images/leon/post/ba093e38-f2d9-4dc9-a14b-935ba854b42d/image.png)

_문서를 보면 내가 설정한 `http://localhost:8080/kakao.lo`로 인가 코드를 쿼리 스트링을 전송해준다고 한다. 그러면 인가코드를 받아 줄 컨트롤러를 작성해보자._

![](https://velog.velcdn.com/images/leon/post/b3ff1ebf-668f-4bed-9c0f-f3a59e5d8150/image.png)

>
1. 사용자가 로그인링크를 클릭한다.
2. 카카오 로그인 페이지가 열린다.
3. 사용자가 로그인을 성공했을 시 카카오 api 서버에서 해당 컨트롤러로 code를 전송해준다.


![](https://velog.velcdn.com/images/leon/post/ec7ad6a2-fc46-4d62-83e4-90c543b4b3d0/image.png)

_ 그 다음은 토큰 받기 이다. 코드를 받았으면 끝인가? 하고 생각할 수도 있겠지만 그렇지 않다. oauth2로그인의 흐름을 봐보자면_

>1. 서비스 클라이언트에서 해당 소셜로그인의 링크를 걸어 해당 소셜로그인의 페이지에서 로그인하도록 한다.
2. 로그인이 성공하면 인가코드를 서비스 서버로 보내준다.
3. 해당 인가코드를 이용해 이 서비스 서버가 유효한 서버임을 증명하기 위해 다시한번 소셜로그인 api 서버로 요청을 보낸다.
4. 해당 서비스 서버가 유효한 서버라고 판단이 되면 해당 소셜로그인의 DB서버에 접근할 수 있는 토큰을 발급해준다.
5. 그러면 서비스 서버는 그 토큰을 이용해 소셜로그인에 회원가입 되어있는 유저의 정보를 요청한다.

_다음은 문서를 보자._
```
POST /oauth/token HTTP/1.1
Host: kauth.kakao.com
Content-type: application/x-www-form-urlencoded;charset=utf-8
```

![](https://velog.velcdn.com/images/leon/post/06fdf586-57f1-4c15-ba0d-741f27e8b35c/image.png)

_필수로 필요한 파라미터는 총 4개이고, `https://kauth.kakao.com/oauth2/token`으로 Content-type은 `application/x-www-form-urlencoded;charset=utf-8` 으로 보내달라고 명시하고 있다. 이것을 그대로 코드로 표현해보자._

```java

	HttpHeaders headers = new HttpHeaders();
	headers.add("Content-type","application/x-www-form-urlencoded;charset=utf-8"); --- 1
		
	MultiValueMap<String, String> accessTokenBodyInfo = new LinkedMultiValueMap<String, String>();
	accessTokenBodyInfo.add("grant_type", "authorization_code"); --- 2
	accessTokenBodyInfo.add("client_id", "{REST_API_KEY}"); --- 3
	accessTokenBodyInfo.add("redirect_uri", "http://localhost:8080/kakao.lo"); --- 4
	accessTokenBodyInfo.add("code", code); --- 5
		
		
	ResponseEntity<String> response = new RestTemplate().exchange(
			"https://kauth.kakao.com/oauth/token",
			HttpMethod.POST, 
			new HttpEntity<MultiValueMap<String, String>>(accessTokenBodyInfo, headers),
			String.class
			);
            
     JsonParser parser = new JsonParser();
     JsonElement element = parser.parse(response.getBody());
     String accessToken = element.getAsJsonObject().get("access_token").getAsString(); --- 6     
```

> 1. 문서에서 명시한 대로 Content-type을 지정해준다.
2. 문서에서 authorization_code으로 고정해달라 했으니, 그대로 작성한다.
3. 문서에서 명시된 대로 앱 키에 있는 REST_API_KEY를 넣어준다.
4. 우리가 인가코드를 받은 redirect_uri를 작성해준다.
5. 쿼리스트링으로 넘어온 코드를 넣어준다.
6. ```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
{
    "token_type":"bearer",
    "access_token":"${ACCESS_TOKEN}",
    "expires_in":43199,
    "refresh_token":"${REFRESH_TOKEN}",
    "refresh_token_expires_in":25184000,
    "scope":"account_email profile"
}
```
저렇게 요청을 보내고 받아온 response의 Boay값은 이러하다.
우리가 필요한 것은 access_token 이므로 해당 토큰값만 뽑아준다.

---
_여기까지 왓으면 우리는 token값을 통해 이제 다시 kakao 서버에 사용자 정보를 가져오는 요청을 보내야 한다. 그에 해당하는 문서는 이러하다.
_
![](https://velog.velcdn.com/images/leon/post/13f3e74b-9469-4bcb-8b49-0f649ced6703/image.png)

_이 것을 그대로 코드로 나타내보자._

```java
	HttpHeaders headers = new HttpHeaders();
	headers.add("Authorization", "Bearer "+accessToken); --- 1
	headers.add("Content-type","application/x-www-form-urlencoded;charset=utf-8"); --- 2
		
	ResponseEntity<String> response = new RestTemplate().exchange(
			"https://kapi.kakao.com/v2/user/me",
			HttpMethod.POST, 
			new HttpEntity<String>(headers),
			String.class
			);
```
> 1. 문서를 자세히 보면 Bearer하고 띄어쓰기가 하나 있는 것을 볼 수 있다. 그래서 "Bearer "문자열과 위에서 받아온 토큰을 넣어준다.
2. 위에 작성된 Content-Type을 명시해준다.


---
_여기까지 했으면 이제 카카오 저장서버에 회원가입 되어있는 유저정보가 response에 담아서 올 것이다. 그 사용자 정보로 요리하는 것은 이제 당신의 몫이다._

---
이 글에서는 RestTemplate을 사용해서 구현했으나, HttpClient를 사용해도 좋고 URI를 사용해도 좋다. 단순히 코드를 왜 받아오고, 토큰을 왜 받아오는지 전반적인 소셜로그인의 구현 흐름을 이해하기 위한 글이다.


