---
layout: post
title: Flutter, Spring Boot로 로그인 구현 후 회고
sitemap: true
categories:
  - project
  - Table Top Tracker
hide_last_modified: true
---
**로그인 구현을 마친 뒤 작성하는 회고**  
로그인 구현이 쉽지 않을 것이라고 예상하긴 했는데 그 이상으로 많이 헤매었고, 많이 고민했다.  
그냥 지나가면 무조건 까먹을 것 같으니 기록으로 남기려고 한다.  
개발 과정을 시간 순서대로 작성했다.

1. this ordered seed list will be replaced by the toc
{:toc}

## [기획] 로그인 과정 구체화
가장 먼저 회원가입, 로그인을 어떻게 진행할지 정했다. 사용자 관련된 것을 모두 결정해야 본격적인 개발에 들어갈 수 있을 것 같아서 다른건 다 제치고 로그인 과정을 먼저 구체화했다.

사용자도 편하고, 내가 개발하기 쉽기도 한 **소셜 로그인을 통한 로그인만 지원**하기로 했다.  
이유를 나열하자면...
1. 소셜 서비스가 나를 대신해서 인증 과정을 진행해준다. 이메일 인증, 비밀번호 암호화, 계정 찾기 등등 많은 일을 대신해줘서 굉장히 편하다. 최대한 간단하지만 확실한 회원가입을 구현하고 싶었다.
2. 소셜 로그인을 이용하면 로그인과 회원가입을 통합해서 진행할 수 있다.
3. 소셜 로그인을 구현해본 경험이 있었다.

그래서 **구글 로그인**을 타겟팅했다. 안드로이드 이용자는 무조건 구글 계정이 있을 것이고, 애플 사용자도 대부분 구글 계정이 있을 것이기에 일단 구글 로그인을 중심으로 개발하자고 생각했다. 물론 애플 사용자를 위해 애플 로그인도 구현하려 했지만... 1년에 12만원 가량의 Apple Developer을 등록해야 해서 일단은 미뤄뒀다. 그래도 언젠간 개발할 것을 염두에 두고 소셜 로그인을 통한 회원가입을 설계했다.  

사용자를 구별할 수 있는 **unique 값은 이메일로 설정**했다. 어떤 소셜 로그인을 하든, 무조건 이메일을 사용하게 되어있다. 또한 사용자가 하나의 이메일로 A 소셜 로그인을 통해 회원가입을 한 뒤 B 소셜 로그인을 통해 로그인을 시도하면 자연스럽게 계정 연동을 유도할 수 있다고 판단되었다. 그래서 각 소셜에 해당하는 id값도 저장을 할 예정이다.  

최종적인 **로그인 단계**는 다음과 같다:
1. Flutter측에서 소셜 로그인을 진행한다
2. 소셜 측에서 해당 사용자 고유의 id를 준다
3. 로그인 진행한 소셜의 id, 소셜 로그인에 사용한 이메일을 백엔드로 넘기고 이메일을 기반으로 유저가 있는지 확인한다
   1. 유저가 없다면 회원가입을 진행한다(404 Not Found)
   2. 유저가 있지만 현재 소셜의 id가 존재하지 않으면 소셜 계정 연동을 하냐고 묻는다(403 Forbidden)
   3. 유저가 있고 소셜 id가 동일하다면 로그인한다(200 OK)

이것이 정석인지는 잘 모르겠다. Spring Security를 잘 활용한 프로젝트나 공식 문서를 보고 싶었지만 찾지 못했다.  
또한 지금까지 진행했던 프로젝트에서 RESTful API를 통한 통신을 하는데 HTTP status code를 적극적으로 활용하지 않는 것이 너무 이상했다. 그래서 이번에는 HTTP status code를 정말 적극적으로 활용하기로 결심했다.  

## [기획] 시퀀스 다이어그램 그리기
로그인 과정을 구체화하며 시퀀스 다이어그램을 그렸다. 뭐가 이리 고민이 많았는지, 거의 일주일 넘게 로그인 시퀀스 다이어그램만 붙잡고 있었다

![이미지](https://manchott.github.io/assets/img/login-review/login_sequence_diagram.png "xcode orientation 설정")

나는 정석적인 방법, 공식 문서를 좋아하는데 로그인 관련해서는 찾기가 정말 어려웠다... 그래서 내 나름대로 논리적인 로직을 작성해봤다.  
시퀀스 다이어그램을 작성하며 어떻게 이 과정을 구현할지 계속 고민해보았다. 그 과정에서 **Spring Security의 필요성과 인증, 인가**에 대한 학습을 했다. 사실 아직도 인증 인가가 아리까리하지만.. 어찌저찌 해냈다.

## [BE] SecurityFilterChain
Spring Security를 설정하려하니 SecurityFilterChain을 설정해야한단다. 이게 무엇인가 보니 모든 요청이 들어오기 전에 필터를 걸어서 사용자의 인증, 인가 정보를 확인하는 것이었다. 

~~~java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
  http.httpBasic(HttpBasicConfigurer::disable) // 사용자의 아이디와 비밀번호를 이용하는 인증방법 사용x
      .cors(AbstractHttpConfigurer::disable)
      .csrf(AbstractHttpConfigurer::disable)  // 어플 서비스라서 csrf 해제해도 됨
      .sessionManagement(configurer -> configurer.sessionCreationPolicy(
          SessionCreationPolicy.STATELESS)); // 토큰 기반 인증을 위해 세션 생성x

  http.authorizeHttpRequests(authorize -> authorize
      .requestMatchers(HttpMethod.POST, "/user/login", "/user/join")
      .permitAll()
      .requestMatchers(HttpMethod.GET, "/user/test")
      .permitAll()
      .anyRequest().authenticated()
  );

  http.addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
      .exceptionHandling(exception -> exception.authenticationEntryPoint(
          new HttpStatusEntryPoint(HttpStatus.UNAUTHORIZED))
      )
  return http.build();
}
~~~

기본적인 설정과 HTTP 요청에 대한 접근 권한을 설정하고, 따로 필터를 만들어서 넣어주었다. jwtFilter를 만드는 과정이 제일 고민이 많았던 것 같다. 이건 따로 문서를 만들어야한다고 생각될 정도로....  
간단히 정리하자면...
1. **Access Token과 Refresh Token을 함께 사용하는 방법**을 선택했다
2. Access Token은 Header에 넣어서 매 요청에 포함하도록 했다
3. Refresh Token은 Redis에 저장했다. Key는 Access Token으로 설정했다.

Token 사용하는 부분은 정말 많은 고민을 했다. 다른 사람들의 코드도 많이 봤지만 코딩에 해답은 없다고 생각하기에 나만의 방식으로 잘 풀어나간 것 같다.  

현재 상황에서는 cors를 disable해놓았지만 추후에 이 부분은 설정을 통해 바꿔나갈 예정이다. 또한 내 서비스에서는 USER와 ADMIN 등의 Role을 줄 필요성을 느끼지 못해서 따로 설정하지 않았다.

## [BE] 개인정보 암호화에 대한 고민

그러다가 문득, **이메일도 개인정보가 아닌가**? 하는 생각이 들었다. 그리고 개인정보화는 암호화해야 안전하다는 말이 기억났다. TTT는 개인 플젝이지만, 이 프로젝트를 할 때 로그인만은 제대로 하기로 마음 먹은 것이 사실 [BGG](https://boardgamegeek.com/) 사이트의 개발자 콘솔을 열어본 이후이다. 로그인 후에 네트워크를 까보면 적나라하게 나오는 비밀번호... 이게 맞나? 싶어서 네이버를 확인해보니 당연히 보안이 철저하다.  

그래서 암호화에 대해 좀 알아보았다. 다양한 암호화 알고리즘이 있고, 국내 표준, 국제 표준 등 정보는 많았다. API 통신할 때 자동으로 복호화하는 기능도 있더라.  
그러나 **지금은 할 때가 아니다!**라고 생각했다. 일단 개발을 마친 뒤에 하자는 생각이 들었다. 로그인의 다른 과정에 비해 치명적이지 않은 기능이니까.  
만약 이메일을 암호화 한다면 [AES](https://ko.wikipedia.org/wiki/%EA%B3%A0%EA%B8%89_%EC%95%94%ED%98%B8%ED%99%94_%ED%91%9C%EC%A4%80) 방식을 사용할 것 같다.

## [FE] Google 로그인 구현

백엔드 설정을 마친 뒤, 소셜 로그인 구현을 위해 프론트엔드인 Flutter로 넘어갔다. **Google 로그인은 사실 정말 간단하게 구현 가능**하다. 구글이 모든 것을 다 해주기에 설정 파일을 제대로 넣고 연결만 해주면 된다.  
~~~Dart
final GoogleSignInAccount? googleUser = await GoogleSignIn().signIn();
~~~
제대로 된 설정과 위의 한 줄만 있으면 흔히 봤던 구글 계정 선택하고 로그인하는 페이지가 나온다. 해당 페이지에서 로그인을 마치면 `googleUser.id`로 구글 id를 가져올 수 있다. 남은 일은 백엔드에 이메일과 id를 넘긴 뒤, response를 받으면 위에서 정한대로 HTTP status code에 따른 페이지 이동을 하면 된다. 끝!  
이 과정에서 Flutter도 Provider, Navigation 등 초기 설정도 진행했다. 사용자의 정보에서 어디까지 Provider에 넣을 것인지는 아직도 고민중이다...

## [BE] 로직 구현하기

프론트엔드에서 구글 로그인을 완성한 뒤 그 결과를 가지고 개발을 진행했다.  
Spring Boot 개발을 할 때, 오류 처리를 어떻게 해야하나 고민을 많이 했다. error를 try-catch를 통해서 해결할지 아니면 그냥 로직으로 다 처리를 할지 갈등하다가 결국 로직으로 처리하기로 했다.  
~~~java
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginReqDto loginReqDto) throws Exception {
  String email = loginReqDto.getEmail();
  log.info(email);
  User user = userService.getUserByEmail(email);
  // 신규 가입자면 404 not found
  if (user == null) {
    return new ResponseEntity<>(loginReqDto, HttpStatus.NOT_FOUND);
  }
  // 로그인 방법 확인
  if (loginReqDto.getGoogleId() != null) {
    // 구글이 아닌 다른 것으로 회원가입한 사용자면 403 forbidden, 계정연동 하겠냐고 물어야함
    if (user.getGoogleId() == null) {
      return new ResponseEntity<>(HttpStatus.FORBIDDEN);
    }
  }
  // 올바른 sns로 로그인 한 사용자는 토큰을 발급해주고 헤더에 넣어준 뒤 로그인시킨다
  LoginResDto loginResDto = userService.login(user, loginReqDto.getDeviceToken());
  HttpHeaders headers = userService.setHeader(loginReqDto.getEmail());

  return new ResponseEntity<>(loginResDto, headers, HttpStatus.OK);
}
~~~
예를 들어, userService에서 userRepository의 findByEmail을 했는데 유저가 없다면 error를 일으키는 것이 아니라 404 Not Found를 프론트엔드에 response로 보내서 "없는 사용자" 상태임을 알렸다. 이 방식으로 내가 원했던대로 HTTP status code를 적절히 사용할 수 있었다.  
또한 이 단계에서 Access Token과 Refresh Token을 발급하여 각각 적절한 위치에 저장했다.

## [FE] HTTP status code에 따른 로직 구현하기

다시 프론트엔드에 와서 백엔드가 보낸 HTTP status code에 따라 분기점을 나누었다. 신규 회원이면 회원 가입 페이지로, 기존 회원이면 메인 페이지로 보냈다.

## 회고

약 3주에 걸쳐서 꽤나 고생해서 기획하고 구현했는데 막상 글로 써보니 별거 아닌 것 같기도 하고 고생 좀 한 것 같기도 하고...  
로그인은 고의적으로 오류를 발생시키기 어려워서 돌발 상황 대처가 참 어려운 것 같다.  
추가적으로 공부를 진행하고, 보안 취약점을 보완해나가야겠다.