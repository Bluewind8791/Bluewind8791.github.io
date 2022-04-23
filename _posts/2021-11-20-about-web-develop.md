---
layout: post
title: 웹 개발이란?
description: web rest api http
summary: web rest api http
tags: spring
---

## **WEB 의 기본 3가지 요소**

- URI (Uniform Resource Identifier) : 리소스 식별자

- HTTP (Hypertext Transfer Protocol)

- HTML (Hypter Text Markup Language) : XML을 바탕으로 한 범용 문서 포맷

## **REST API**

REST (Representational State Transfer : 자원의 상태 전달) - 네트워크 아키텍처

### 6가지 아키텍쳐

1. Client, Server : 클라이언트와 서버가 서로 독립적으로 분리되어있어야 함.

2. Stateless : 요청에 대하여 클라이언트의 상태를 서버에 저장하지 않는다.

> ex) 햄버거주세요 / 콜라도 같이 주세요 (X) 클라이언트 상태를 서버에 저장하지않으므로  
> 햄버거주세요 / 햄버거 콜라 주세요 (O) 매번 새롭게 요청해야 한다.

3. Cache : 클라이언트는 서버의 응답을 캐시(임시저장)할 수 있어야함. 클라이언트가 캐시를 통하여 응답을 재사용할 수 있어야 하며, 이를 통하여 서버의 부하를 낮춤.
4. 계층화(Layer System) : 서버와 클라이언트 사이에 방화벽, 게이트웨이, Proxy 등 다양한 계층 형태로 구성이 가능해야하며 이를 확장할 수 있어야 한다.
5. 인터페이스 일관성 : 인터페이스의 일관성을 지키고, 아키텍처를 단순화시켜 작은 단위로 분리하여 클라이언트, 서버가 독립적으로 개선될수 있어야 한다.
6. Code on Demand (Optional) : 자바 애플릿, 자바스크립트, 플래시 등 특정한 기능을 서버로 부터 클라이언트가 전달받아 코드를 실행할수 있어야 한다.

### 다음의 일관성이 잘 지켜졌는지에 따라 REST를 잘 사용햇는지 판단할 수 있다

- 자원의 식별 : 웹 기반의 REST에서는 리소스 접근할떄 URI 사용  
<https://foo.co.kr/user/100> - Resource : user / 식별자 : 100

- 메세지를 통한 리소스 조작: 웹에서 가장 많이 사용하는 데이터 전달 방식은 HTML XML JSON TEXT 등이 있다. 이중에서 어떠한 타입의 데이터인지 알려주기 위하여 HTTP Header 부분에 content-type를 통하여 데이터의 타입을 지정한다. 또한 리소스 조작을 위하여 데이터 전체를 전달하지 않고 메세지로 전달한다.

> ex) 서버의 user 라는 정보의 전호번호를 number 라고 결정. 이 정보를 클라이언트와 주고받을때 그대로 사용하고, 그 후에 서버 Resource 변경으로 phone-number 로 바뀐다면 클라이언트는 처리못하고 에러가 난다. 그러므로 별도의 메세지 형태로 데이터 주고받고 client-sever가 독립적으로 확장 가능하게한다.

- 자기서술적 메세지 : 요청하는 데이터가 어떻게 처리되어야 하는지 충분한 데이터를 포함. HTTP 기반의 REST에서는 HTTP method와 header 정보, uri의 포함되는 정보로 표현 가능.

  - GET : <https://foo.co.kr/user/100> - 사용자 정보 요청
  - POST : <https://foo.co.kr/user> - 사용자 정보 생성
  - PUT : <https://foo.co.kr/user> - 사용자 정보 생성 및 수정
  - DELETE : <https://foo.co.kr/user/100> - 사용자 정보 삭제
  - 그외에 담지못한 정보는 uri의 메세지를 통하여 표현

- Application 상태에 대한 엔진으로써 하이퍼미디어 :
  - REST API를 개발할 때, 단순히 client 요청에 대한 데이터만 응답해주는 것이 아닌 관련된 리소스에 대한 link 정보까지 같이 포함.
  - 이러한 조건을 잘 갖추면 RESTful하다고 표현하며, 이를 REST API라고 부른다.
  - 규약 - 이정도는 지켜서 해야 됨. 4번은 현업에서는 잘 안쓰는 경향이 있다.

## **URI 설계 패턴**

- URI (Uniform Resource Identifier) :
  - 인터넷에서 특정 자원을 나타내는 유일한 주소 값. (응답은 달라질 수 있다)
  - 요청 : <https://www.fastcampus.co.kr/resource/sample/1>
  - 응답 : fastcampus.pdf,fastcampus.docx

- URL (Uniform Resource Locator)
  - 인터넷 상에서의 자원, 특정 파일이 어디에 위치하는지 식별하는 주소
  - 요청 : <https://www.fastcampus.co.kr/fastcampus.pdf>
  - ※ URL은 URI의 하위개념.

## **URI 설계 원칙**

- 구분자(/)는 계층관계를 나타냄
- URI 마지막 문자로 / 는 포함 X
- 하이픈(-) 은 URI 가독성을 높이는데 사용
- 밑줄(_) 은 사용 X
- 소문자를 사용
- 파일확장자는 URI에 포함하지 않음

- 프로그램 언어에 의존적인 확장자 사용 x
  - ex) <https://fastcampus.co.kr/classes/java/curriculums/web-master.do>

- 구현에 의존적인 경로를 사용하지 않는다
  - ex) <https://fastcampus.co.kr/'servlet'/classes/java/curriculums/web-master>

- 세션 ID 포함 X
  - ex) <https://fastcampus.co.kr/classes/java/curriculums/web-master?'session-id=abcdef'>

- 프로그래밍 언어의 method 명을 이용 X
  - ex) <https://fastcampus.co.kr/classes/java/curriculums/web-master?'action=intro'>

- 명사에 단수형 보다는 복수형 / 컬렉션에 대한 표현은 복수로
  - ex) <https://fastcampus.co.kr/class'es'/java/curriculum's'/web-master>

- 컨트롤러 이름으로는 동사나 동사구를 사용
  - ex) <https://fastcampus.co.kr/classes/java/curriculums/web-master/'re-order'>

- 경로 부분중 변하는 부분은 유일한 값으로 대체
  - ...web-master/lessons/{lesson-id}/users/{user-id}
  - ...web-master/lessons/2/users/100

- CRUD 기능을 나타내는것은 URI에 사용 X
  - GET : ...web-master/lessons/2/users/100/'READ'(X)
  - GET이 read의 뜻임

- URI Query Parameter 디자인
  - URI 쿼리 부분으로 컬렉션 결과에 대해서 필터링할수있다
  - ...curriculums/web-master?chapter=2

- URI Query 는 컬렉션의 결과를 페이지로 구분하여 나타내는데 사용
  - ...curriculums/web-master?'chapter=2&page=0&size=10&sort=asc'

- API에 있어서 서브도메인은 일관성있게 사용
  - <https://fastcampus.co.kr>
  - https://'api'.fastcampus.co.kr

- 클라이언트 개발자 포탈 서브 도메인 또한 일관성있게
  - https://'dev-'fastcampus.co.kr
  - https://'devloper-'fastcampus.co.kr

## **HTTP Protocol**

- HTTP (Hypertext Transfer Protocol)

- HTTP는 TCP를 기반으로한 REST의 특징을 모두 구현하고 있는 웹기반 프로토콜

- HTTP 는 메세지를 Client와 sever가 주고(request) 받는 (response) 형태의 통신 방법.

- method 8 가지

|         | 의미 | CRUD | 멱등성 | 안정성 |
|---------|-----|------|-------|-------|
| GET     | 리소스 취득 | Read | O | O |
| POST    | 리소스 생성, 추가 | C | X |  X |
| PUT     | 리소스 갱신, 생성 | C/Update | O | X |
| DELETE  | 리소스 삭제 | Delete | O | X |
| HEAD    | 헤더 데이터 취득|  |  |
| OPTIONS | 지원하는 메소드 취득|  |  |
| TRACE   | 요청메세지 반환|  |  |
| CONNECT | 프록시 동작의 터널접속으로 변경|  |  |

( 멱등성 : 몇번을 요청해도 같은 데이터의 응답이 오는것 )

### HTTP Status Code

| Code | 의미 | 내용 |
| --- | --- | --- |
1XX | 처리중        | 처리가 계속 되고 있는 상태, 클라는 요청 계속하거나 서버의 지시에 따라 재요청
2XX | 성공          | 요청의 성공
3XX | 리다이렉트     | 다른 리소스로 리다이렉트. 해당 코드를 받았을때는 response의 새로운 주소로 다시 요청
4XX | 클라이언트 에러 | 클라의 요청에 에러가 있는 상태. 재전송하여도 에러 해결 X
5XX | 서버 에러      | 서버 처리중 에러 발생. 재 전송시 에러가 해결되었을수도 있다.

### 자주 사용되는 코드

- 200 : 성공
- 201 : 성공. 리소스 생성 성공
- 301 : 리다이렉트, 리소스가 다른장소로 변경됨을 알림
- 303 : 리다이렉트, 클라에서 자동으로 새로운 리소스로 요청 처리
- 400 : 요청 오류, 파라미터 에러
- 401 : 권한 없음 (인증실패)
- 404 : 리소스 없음 (페이지를 찾을 수 없음)
- 500 : 서버 내부 에러 (서버 동작 처리 에러)
- 503 : 서비스 정지 (점검 등)
