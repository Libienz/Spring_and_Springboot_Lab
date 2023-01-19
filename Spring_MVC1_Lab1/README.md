# Spring_MVC1_Lab1

<details>
<summary>05 Spring MVC 기본 기능 </summary>
<div markdown="1">

## 프로젝트 생성
- packaging에서 Jar와 War의 차이
  - Jar: 내장 서버를 사용(톰캣) webapp 경로 사용하지 않음. 내장 서버 사용에 최적화 되어 있다. 요즈음은 주로 Jar사용
  - War: 내장 서버도 사용 가능하지만 주로 외부 서버에 빌드 파일을 올릴 때 사용

## Logging
- 로그에 대해 간단히 알아보자
- 이제 sout이 아닌 별도의 로깅 라이브러리를 사용하여 로그를 출력할 것
- 참고로 로그 관련 라이브러리도 많고, 깊게 들어가면 끝이 없기에 최소한의 사용 방법만 알아보자

### 로깅 라이브러리
- 스프링 부트 라이브러리를 사용하면 스프링 부트 로깅 라이브러리가 함께 포함된다.
- 스프링 부트 로깅 라이브러리는 기본으로 다음 로깅 라이브러리를 사용
  - SLF4J
  - Logback
- 여러 로그 라이브러리를 통합하여 (어댑터 패턴 등등) 사용할 수 있도록 인터페이스로 제공하는 것이 SLF4J
- Logback은 로그 라이브러리 (구현체) 실무에서 Logback 많이 사용한다

### 로그 선언, 호출

```java
package hello.springmvc.basic;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@RestController
public class LogTestController {

//    private final Logger log = LoggerFactory.getLogger(getClass());

    @RequestMapping("/log-test")
    public String logTest() {
        String name = "Spring";
        System.out.println("name = " + name);
        log.trace("trace log=" + name); //이렇게 쓰면 안된다 출력 안할 건데 선연산이 들어가버림
        log.debug("debug log={}", name);
        log.info("info log={}", name);
        log.warn("warn log={}", name);
        log.error("error log={}", name);

        log.info("info log={}", name);

        return "ok";
    }
}

```

- @RestController
  - @Controller는 반환 값이 String이면 뷰 이름으로 인식되어 뷰를 찾고 뷰가 렌더링 됨
  - @RestController는 반환 값으로 뷰를 찾는 것이 아니라 HTTP 메시지 바디에 바로 입력
  - @ResponseBody와 관련 있는데 뒤에서 더 자세히 볼 것임
- 로그의 출력 내용
  - 시간, 로그 레벨, 프로세스 ID, 쓰레드 명, 클래스 명, 로그 메시지
- 로그 레벨은 다음과 같다.
  - TRACE > DEBUG > INFO > WARN > ERROR
  - 로그 레벨 설정을 변경하며 노출 시킬 로그 레벨을 정할 수 있다.
  - 보통 개발 서버는 debug이상으로 심각한 로그를 출력
  - 운영 서버는 info 출력
- @Slf4j로 로그 선언 부분을 대체 할 수 있다. (롬복이 대신 써준다)

### 올바른 로그 사용법
- 선연산이 되지 않게 하자 
- log.debug("data=" + data)
  - 위와 같이 써도 로그 출력은 올바로 됨 하지만 debug로그를 노출시키지 않을 예정임에도 파라미터 연산이 먼저되어 서버의 자원을 잡아먹는다
  - 이렇게 쓰면 혼난다.
  - 다음과 같이 쓰자 log.debug("data = {}", data)
  - {}가 서식지정자 마냥 치환된다.

### 로그 사용시 장점
- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고 출력 모양을 조정 간으
- 로그 레벨에 따라 노출여부를 결정 가능
- 콘솔에만 아니라 파일, 네트워크 등 로그를 별도의 위치에 남길 수도 있음
- 특히 파일로 남길 때는 일별 특정 용량에 따라 로그를 분할하는 것도 가능
- 성능도 sout보다 파워풀하다 (내부 버퍼링, 멀티 쓰레드 등등)

## MappingController
- 몇가지를 짚고 코드로 넘어가서 url에 따라 컨트롤러가 매핑되어 동작하는 여러가지 방식을 체크하자
- 첫번째
  - /hello-basic
  - /hello-basic/
  - 스프링 부트 3.0이전은 위의 두 url을 다른 url로 인식
  - 스프링 부트 3.0이후에는 다른 url로 인식
- 두번째
  - @RequestMapping에 method 속성으로 HTTP 메서드를 지정하지 않으면 HTTP 메서드와 무관하게 호출
  - 모두 허용한다.
- 이제 본격적으로 코드를 보자 

```java
package hello.springmvc.basic.requestmapping;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.*;

@RestController
public class MappingController {

    private Logger log = LoggerFactory.getLogger(getClass());

    //배열 형태도 가능 배열안에 있는 url에 컨트롤러 메서드를 매핑시킨다 
    //모든 형태의 메서드를 허용하지 않고 특정 형태만 허용하도록 method를 한정할 수 있다 
    @RequestMapping(value = {"/hello-basic", "/hello-go"}, method = RequestMethod.GET)
    public String helloBasic() {
        log.info("helloBasic");
        return "ok";
    }

    /**
     * 편리한 축약 애노테이션 (코드 보기)
     * 애노테이션을 살펴보면 우리가 써야할 코드를 대신 써주는 것을 확인 가능 
     * 편리한 축약이라고 표현한 이유 
     * @return
     * @GetMapping
     * @PostMapping
     * @PutMapping
     * @DeleteMapping
     * @PatchMapping
     */
    @GetMapping("/mapping-get-v2")
    public String mappingGetv2() {
        log.info("mapping-get-v2");
        return "ok";
    }

    /**
     * PathVariable 사용
     * 변수명이 같으면 생략 가능
     *
     * @PathVariable("userId") String data
     * @PathVariable String userId
     * @PathVariable("userId") String userId -> @PathVariable userId
     * /mapping/userA
     * url 자체에 값이 들어있는 형태
     * 경로 변수, pathVariable
     * 이거 진짜 많이 사용
     */
    @GetMapping("/mapping/{userId}")
    public String mappingPath(@PathVariable("userId") String data) {
        log.info("mappingPath userId={}", data);
        return "ok";
    }

    /**
     * PathVariable 사용 다중
     *
     * @param userId
     * @param orderId
     * @return
     */
    @GetMapping("/mapping/users/{userId}/orders/{orderId}")
    public String mappingPath(@PathVariable String userId, @PathVariable String orderId) {
        log.info("mappingPath userId={}, orderId={}", userId, orderId);
        return "ok";
    }

    /**
     * 파라미터로 추가 매핑
     * 특정 파라미터 정보가 있으면 호출 되는 메서드 만들기
     * params="mode",
     * params="!mode"
     * params="mode=debug"
     * params="mode!=debug"
     * params = {"mode=debug","data=good}
     * @return
     * 애노테이션에 들어있는 파라미터가 요청에 포함되어 있을 경우에만 메서드가 호출된다
     * mode=debug를 url에서 빼면 메서드 호출 안됨
     */
    @GetMapping(value = "/mapping-param", params = "mode=debug")
    public String mappingParam() {
        log.info("mappingParam");
        return "ok";
    }

    /**
     * 특정 헤더로 추가 매핑
     * 요청의 헤더에 mode=debug가 있어야 메서드가 호출된다.
     * headers="mode",
     * headers="!mode"
     * headers="mode=debug"
     * headers="mode!=debug" (! = )
     */
    @GetMapping(value = "/mapping-header", headers = "mode=debug")
    public String mappingHeader() {
        log.info("mappingHeader");
        return "ok";
    }

    /**
     * Content-Type 헤더 기반 추가 매핑 Media Type
     * consumes="application/json"
     * consumes="!application/json"
     * consumes="application/*"
     * consumes="*\/*"
     * MediaType.APPLICATION_JSON_VALUE
     */
    @PostMapping(value = "/mapping-consume", consumes = MediaType.APPLICATION_JSON_VALUE)
    public String mappingConsumes() {
        log.info("mappingConsumes");
        return "ok";
    }

    /**
     * Accept 헤더 기반 Media Type
     * 클라이언트가 요청할 때 나는 이런 데이터를 받아들일 수 있다고 헤더 정보를 남기면 그에 따라 추가적인 매핑을 하는 것
     * Accept는 클라이언트가 받아들일 수 있는,
     * produce는 서버가 반환하는 타입
     * produces = "text/html"
     * produces = "!text/html"
     * produces = "text/*"
     * produces = "*\/*"
     * MediaType.TEXT_HTML_VALUE
     */
    @PostMapping(value = "/mapping-produce", produces = MediaType.TEXT_HTML_VALUE)
    public String mappingProduces() {
        log.info("mappingProduces");
        return "ok";
    }
}

```

## 요청 매핑 -API 예시
- 회원 관리를 HTTP API로 만든다 생각하고 매핑을 어떻게 하는지 알아보자
```java
package hello.springmvc.basic.requestmapping;

import org.springframework.web.bind.annotation.*;

/**
 * 회원 목록 조회: GET /users
 * 회원 등록: POST /users
 * 회원 조회: GET /users/{userId}
 * 회원 수정: PATCH /users/{userId}
 * 회원 삭제: DELETE /users/{userId}
 */
@RequestMapping("/mapping/users") //리소스 계층화
@RestController
public class MappingClassController {

  public String user() {
    return "get users";
  }

  public String addUser() {
    return "post user";
  }

  @GetMapping("/{userId}")
  public String findUser(@PathVariable String userId) {
    return "get userId=" + userId;
  }

  @PatchMapping("/{userId}")
  public String update(@PathVariable String userId) {
    return "update userId=" + userId;
  }

  @DeleteMapping("/{userId}")
  public String deleteUser(@PathVariable String userId) {
    return "delete userId=" + userId;
  }


}
 
```

- 같은 url name도 http메서드에 따라 다르게 매핑되는 것을 확인 가능
- 클래스레벨 url과 메서드 레벨 url을 통해 자원의 계층화 가능 
- 보기 쉽고 쓰기 쉽다 
- 매핑 방법을 이해했으니 이제부터 HTTP 요청이 보내는 데이터들을 스프링 MVC로 어떻게 조회하는지 알아보자

## HTTP 요청 - 기본, 헤더 조회
- 애노테이션 기반의 스프링 컨트롤러는 다양한 파라미터를 지원한다.
- 이번 시간에는 HTTP 헤더 정보를 조회하는 방법을 알아보자

### RequestHeaderController
```java
package hello.springmvc.basic.request;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpMethod;
import org.springframework.util.MultiValueMap;
import org.springframework.web.bind.annotation.CookieValue;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Locale;

@Slf4j
@RestController
public class RequestHeaderController {

  //스프링 애노테이션 기반 컨트롤러는 다양한 파라미터를 지원한다.
  @RequestMapping("/headers")
  public String headers(HttpServletRequest request,
                        HttpServletResponse response,
                        HttpMethod httpMethod, //GET, POST, DELETE 등등
                        Locale locale, //언어 정보
                        @RequestHeader MultiValueMap<String,String> headerMap, //헤더를 한번에 다 받는다. 맵에
                        @RequestHeader("host") String host, //헤더 하나만 가져오는
                        @CookieValue(value = "myCookie", required = false) String cookie) { //required의 디폴트는 true -> false로 하면 없어도 된다.

    log.info("request={}", request);
    log.info("response={}", response);
    log.info("httpMethod={}", httpMethod);
    log.info("locale={}", locale);
    log.info("headerMap={}", headerMap);
    log.info("header host={}", host);
    log.info("myCookie={}", cookie);
    return "ok";
  }
}

```
- 참고 MultiValueMap
  - Map과 유사한데, 하나의 키에 여러 값을 받을 수 있다.
  - HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.
  - get하면 배열을 반환
  - key 하나에 매핑된 여러 value를 조회 가능  

## HTTP 요청 파라미터 - 쿼리 파라미터 HTML Form

### HTTP 요청 데이터 조회 개요
- 서블릿에서 학습했던 HTTP 요청 데이터를 조회 하는 방법을 다시 떠올려보자. 그리고 서블릿으로 학습했던 내용을 스프링이 얼마나 깔끔하고 효율적으로 바꾸어주는지 알아보자
- 우선 HTTP 요청 메시지를 통해 클라이언트에서 서버로 데이터를 전달하는 방법은 3가지라고 배운적이 있었다.
  - GET - 쿼리파라미터
    - /url?username=hello&age=20
    - 메시지 바디 없이 URL의 쿼리 파라미터에 데이터를 포함해서 전달하는 방식
    - 검색, 필터, 페이징등에서 많이 사용하는 방식이다.
  - POST - HTML Form
    - content-type:application/x-www-form-urlencoded
    - 메시지 바디에 쿼리파라미터 형식으로 전달 username=hello&age=20
    - 회원 가입, 상품 주문, HTML Form을 사용
  - HTTP message body에 데이터를 직접 담아서 요청
    - HTTP API에서 주로 사용, JSON, XML, TEXT
    - 데이터 형식은 주로 JSON 사용
    - POST, PUT, PATCH
- get 쿼리 파라미터 전송 방식이든 post html form 전송 방식이든 쿼리파라미터의 형식이 같음으로 Servlet Request로 구분없이 조회할 수 있던 것 기억하자
- 이것을 간단히 요청 파라미터 조회라 한다.
- 스프링에서 요청 파라미터를 조회하는 방법을 단계적으로 알아보자

```java
package hello.springmvc.basic.request;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import java.io.IOException;
import java.util.Map;

@Slf4j
@Controller
public class RequestParamController {

  @RequestMapping("/request-param-v1")
  public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
    String username = request.getParameter("username");
    int age = Integer.parseInt(request.getParameter("age"));
    log.info("username={}, age={}", username, age);

    response.getWriter().write("ok");

  }

  //응답 메시지에 바로 때려 넣는 ResponseBody -> view 조회를 하지 않는다.
  @ResponseBody
  @RequestMapping("/request-param-v2")
  public String requestParamV2(
          @RequestParam("username") String memberName,
          @RequestParam("age") int memberAge) {
    log.info("username={}, age={}", memberName, memberAge);
    return "ok";

  }


  //내가 사용하고자 하는 변수명이 쿼리 파라미터의 변수명과 같다면 위에서 일정 부분 생략 가능
  //축약된 버전 v2 -> v3
  @ResponseBody
  @RequestMapping("/request-param-v3")
  public String requestParamV3(
          @RequestParam String username,
          @RequestParam int age) {
    log.info("username={}, age={}", username, age);
    return "ok";

  }
  //근데 사실 아래 v4처럼 다 없앨 수도 있음 ㅋㅋ
  //요청 파라미터 이름과 나의 변수명이 일치하면
  @ResponseBody
  @RequestMapping("/request-param-v4")
  public String requestParamV4(String username, int age) {
    log.info("username={}, age={}", username, age);
    return "ok";
  }

  //required는 default값이 true -> requestParam이 무조건 있어야 한다.
  //false -> 해당 requestParam이 없어도 된다.
  @ResponseBody
  @RequestMapping("/request-param-required")
  public String requestParamRequired(
          @RequestParam(required = true) String username,
          @RequestParam(required = false) Integer age) {

    log.info("username={}, age={}", username, age);
    return "ok";
  }

  @ResponseBody
  @RequestMapping("/request-param-default")
  public String requestParamDefault(
          //사실 default val 들어가면 required는 무의미
          @RequestParam(required = true, defaultValue = "guest") String username,
          @RequestParam(required = false,defaultValue = "-1") int age) {

    log.info("username={}, age={}", username, age);
    return "ok";
  }

  @ResponseBody
  @RequestMapping("/request-param-map")
  public String requestParamMap(@RequestParam Map<String, Object> paramMap) {

    log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));
    return "ok";
  }

}

```
- V1
  - Get 쿼리 파라미터 요청이든 HTML form 요청이든 구분없이 처리하는 것을 확인할 수 있다(servlet)
- V2
  - 애노테이션이용 축약된 형태로 파라미터를 뽑을 수 있는 것을 확인할 수 있음 
    - 실제로 request.getParameter코드를 품고 있다.
- V3
  - v2를 축약시킨 버전. 요청 파라미터 이름과 변수이름이 일치한다면 생략할 수 있다 
- V4
  - v3에서 아예 더 축약시킬 수 있다. 
    - 하지만 너무 단순한 게 과하다는 생각도 있는 듯
    - @RequestParam이 있는 것이 오히려 변수가 요청 파라미터 정보라는 것을 명확히 알려주기에 좋다고 생각하는 사람도 있다
- requestParamRequired
  - @RequestParam()안에 required를 설정함으로써 해당 요청 파라미터를 필수 또는 필수가 아닌 요소로 설정할 수 있음
  - 디폴트는 true -> 필수
  - 필수 파라미터가 요청데이터에 포함되지 않으면 서버는 Bad Request로 응답한다
    - 주의 : 기본형에 null은 들어갈 수 없다
    - int age 에는 null을 입력할 수 없음 age가 필수 값이 아니기에 null을 지정할 수 있어야 함 
    - Integer로 변경하거나 defaultValue를 사용해야 함
- requestParamDefault
  - required는 여기에선 이제 더이상 필요없음 써도 안먹히기도 하고
  - 기본값으로 퉁치고 들어간다. 
- requestParmaMap
  - 파라미터들을 맵으로 받고 조회할 수 있다.
  - MultiValueMap을 사용할 수도 있지만 파라미터의 값이 1개가 확실하다면 Map을 사용해도 된다.
    - 참고로 같은 키값을 가지는 파라미터를 쓰는 경우는 흔치 않다


</div>
</details>