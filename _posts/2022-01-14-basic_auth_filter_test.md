---
layout: post
title: "[Spring Security] BasicAuthenticationFilter Test"
description: spring security basic authentication auth filter test
summary: spring security basic authentication auth filter test
tags: spring_security
---

# BasicAuthenticationFilter

필터를 검증하는 테스트를 해보자.

```java
@RestController
public class HomeController {
    
    @GetMapping("/greeting")
    public String greeting() {
        return "greeting!";
    }
}
```

GET 방식으로 greeting! 을 내려주는 서비스를 한다고 가정하자.

그런데 저 메세지가 secure 한 메세지이다.

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests().anyRequest().authenticated() // 1
            .and()
            .httpBasic() // 2
            ;
...
```

1. `.authorizeRequests().anyRequest().authenticated()` 모든 request를 막는다
2. `.and().httpBasic()` 그리고 httpBasic으로 메세지가 들어오게 한다

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class BasicAuthenticationTest {

    @LocalServerPort
    int port;

    RestTemplate client = new RestTemplate();

    private String greetingUrl() {
        return "http://localhost:" + port + "/greeting";
    }

    @DisplayName("1. 인증 실패")
    @Test
    void test_1() {
        HttpClientErrorException exception = assertThrows(HttpClientErrorException.class, () -> {
            client.getForObject(greetingUrl(), String.class);
        });
        assertEquals(401, exception.getRawStatusCode());
    }
```

- rest client 를 가지고 get for object로 greeting url에서 greeting 메세지를 가져온다.

- 하지만 SecurityConfig 에 모든 request를 인증해야 들어올 수 있도록 했기 때문에 메세지를 가져올 수 없다.

### 그렇다면 인증을 어떻게 하면 성공할까?

먼저 테스트용 유저를 SecurityConfig 에 inMemoryAuthentication 을 통하여 하나 만들어본다.

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication()
            .withUser(
                User.withDefaultPasswordEncoder()
                .username("user1")
                .password("1111")
                .roles("USER")
                .build()
            );
    }
```

이제 두번째 테스트를 진행하는데

```java
@DisplayName("2. 인증 성공")
    @Test
    void test_2() {
        HttpHeaders headers = new HttpHeaders(); // 2
        headers.add(HttpHeaders.AUTHORIZATION, "Basic " +Base64.getEncoder().encodeToString(
            "user1:1111".getBytes()
        ));

        HttpEntity entity = new HttpEntity(null, headers);
        ResponseEntity<String> resp = client.exchange(greetingUrl(), HttpMethod.GET, entity, String.class); // 1

        assertEquals("greeting!", resp.getBody()); // 3
    }
```

1. 이번에는 rest client를 사용하여 get for object를 하는데 greeting url의 get 방식에서 entity를 가지고와서 string으로 결과값을 가져온다.
2. auth 를 위하여 entity에 헤더 값을 붙이는데, Basic Token을 달아서 보내는데, 이 Basic Token에는 username 과 password를 base64로 인코딩한다.
3. 이제 1번을 response 받아서 assertEquals로 body값과 비교해 본다.

이제 결과값을 보면 테스트가 성공하는 것을 알 수 있다.

---

이 과정을 BasicAuthenticationFilter class를 통하여 자세히 보자.

```java
@Override // 1
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
		throws IOException, ServletException {
	try {
		UsernamePasswordAuthenticationToken authRequest = this.authenticationConverter.convert(request); // 2
    ...
```

1. doFilterInternal 을 통하여 인증이 이루어지는데, request.getHeaderNames() 를 보게되면 여러가지 헤더 중에서 authorization 헤더에 BasicToken 이 넘어온것을 확인할 수 있다.
2. 이 베이직 토큰을 convert() 에서 캐치하는데

```java
@Override
public UsernamePasswordAuthenticationToken convert(HttpServletRequest request) {
	String header = request.getHeader(HttpHeaders.AUTHORIZATION); // 1
	if (header == null) {
		return null;
	}
	header = header.trim(); // 2
	if (!StringUtils.startsWithIgnoreCase(header, AUTHENTICATION_SCHEME_BASIC)) {
		return null;
	}
	if (header.equalsIgnoreCase(AUTHENTICATION_SCHEME_BASIC)) {
		throw new BadCredentialsException("Empty basic authentication token");
	}
	byte[] base64Token = header.substring(6).getBytes(StandardCharsets.UTF_8); // 3
	byte[] decoded = decode(base64Token); // 4
	String token = new String(decoded, getCredentialsCharset(request));
	int delim = token.indexOf(":"); // 5
	if (delim == -1) {
		throw new BadCredentialsException("Invalid basic authentication token");
	}
	UsernamePasswordAuthenticationToken result = new UsernamePasswordAuthenticationToken( // 6
            token.substring(0, delim),
			token.substring(delim + 1));
	result.setDetails(this.authenticationDetailsSource.buildDetails(request)); // 7
	return result; // 8
}
```

1. auth header 값을 읽어서 
2. trim 한 다음에
3. basicToken 인지 확인하고, basicToken 이라면 뒷부분의 토큰을 읽어서
4. decode를 하고
5. ":" 으로 split한 다음 
6. UsernamePasswordAuthenticationToken 을 만들고
7. setDetails 로 request에 대한 details 정보를 가지고 온 다음에
8. UsernamePasswordAuthToken 을 만들어서 넘긴다.

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
		throws IOException, ServletException {
	try {
		UsernamePasswordAuthenticationToken authRequest = this.authenticationConverter.convert(request); // 1
		if (authRequest == null) {
			this.logger.trace("Did not process authentication request since failed to find "
					+ "username and password in Basic Authorization header");
			chain.doFilter(request, response);
			return;
		}
		String username = authRequest.getName();
		this.logger.trace(LogMessage.format("Found username '%s' in Basic Authorization header", username));
		if (authenticationIsRequired(username)) { // 2
			...
```

1. 이렇게 UsernamePasswordAuthenticationToken 을 리턴받고, 이전과 동일하다면, authentication manager 에게 auth request를 넘기게 되는데,
2. BasicAuthToken 이라면 authentication이 필요한지 한번 더 체크를 하게된다.

```java
	private boolean authenticationIsRequired(String username) {
		Authentication existingAuth = SecurityContextHolder.getContext().getAuthentication(); // 1
		if (existingAuth == null || !existingAuth.isAuthenticated()) { // 2
			return true;
		}
		if (existingAuth instanceof UsernamePasswordAuthenticationToken && !existingAuth.getName().equals(username)) {
			return true;
		}
		return (existingAuth instanceof AnonymousAuthenticationToken);
	}
```

1. SecurityContextHolder 를 뒤져봐서 
2. 인증받은 사용자인지 아닌지 체크하는데 인증받지 않았다면 이곳에서 인증을 위하여 true 로 return 하게 된다.
- 하지만 인증받은 사용자라면 뒷 단계를 skip 하게 된다.

```java
@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
		throws IOException, ServletException {
	...
		if (authenticationIsRequired(username)) {
			Authentication authResult = this.authenticationManager.authenticate(authRequest);
			SecurityContextHolder.getContext().setAuthentication(authResult); // 1
			if (this.logger.isDebugEnabled()) {
				this.logger.debug(LogMessage.format("Set SecurityContextHolder to %s", authResult));
			}
			this.rememberMeServices.loginSuccess(request, response, authResult); // 2
			onSuccessfulAuthentication(request, response, authResult); // 3
		}
	}
	...
```

1. AuthToken 을 SecurityContextHolder 에 넣어준 다음에 
2. rememberMe 서비스가 있다면, 리멤버미 서세스 이벤트를 날려서 쿠키나 이런것들을 세팅하고
3. onSuccessfulAuthentication 이벤트를 날린다.

이러한 과정을 통하여 Secured 한 상황에서 greeting! 이라는 메세지를 가져오게 되는것이다!


### additional...

TestRestTemplate 를 쓰게되면 간단하게 BasicToken Test를 진행할 수 있다.

```java
@DisplayName("3. 인증 성공3")
@Test
void test_3() {
    TestRestTemplate testClient = new TestRestTemplate("user1", "1111");
    String resp = testClient.getForObject(greetingUrl(), String.class);
    assertEquals("greeting!", resp);
}
```
