---
title : "[클린 코드] 챕터3 함수2"
excerpt: "클린코드 정주행5"

categories :
  - CleanCode

toc: true
toc_sticky: true
last_modified_at: 2021-11-13 
---

![study.jpg](/assets/images/study.jpg?raw=true)

## 3.7 부수 효과를 일으키지 마라

부수 효과는 거짓말이다. 함수에서 한 가지를 하겠다고 약속하고서는 남몰래 다른 짓도 하니까.

때로는 예상치 못하게 클래스 변수를 수정한다. 때로는 함수로 넘어온 인수나 시스템 전역 변수를 수정한다. 어느 쪽이든 교활하고 해로운 거짓말이다. 많은 경우 일시적인 결합이나 순서 종속성을 초래한다.

다음 함수를 보자

```java
public class UserValidator{
 private Cryptographer cryptographer;
 
 public boolean checkPassword(String userName, String password){
  User user = UserGateway.findByName(userName)

  if(user != User.null){
   String codedPhrase = user.getPhraseEncodedByPassword();
   String phrase = cryptographer.decrypt(codedPhraase, password);

   if("Valid Password".equals(phrase)){
    Session.initialize();
    return true;
   }
  }
  return false;
 }
}
```

아주 무해하게 보이는 함수다. 함수는 표준 알고리즘을 사용해 userName과 password를 확인한다. 두 인수가 올바르면 true를 반환하고 아니면 false를 반환한다. 하지만 함수는 부수 효과를 일으킨다.

여기서 일으키는 부수 효과는 Session.initialize() 호출이다. checkPassword함수는 이름 그대로 암호를 확인한다. 이름만 봐서는 세션을 초기화한다는 사실이 드러나지 않는다. 그래서 함수 이름만 보고서 함수를 호출하는 사용자는 사용자를 인증하면서 기존 세션 정보를 지워버릴 위험에 처한다.

이 부수 효과가 일시적인 결합을 초래한다. 즉, checkPassword 함수는 특정 상황에서만 호출이 가능하다. 세션을 초기화해도 괜찮은 경우에만 호출이 가능하다.

만약 일시적인 결합이 필요하다면 함수 이름에 분명히 명시한다. checkPasswordAndInitalizeSession이라는 이름이 훨씬 낫다. 물론 함수가 '한 가지'만 한다는 규칙을 위반하지만.

### 출력 인수

일반적으로 우리는 인수를 함수 입력으로 해석한다. 인수를 출력으로 사용하는 함수에 어색함을 느끼리라. 다음 함수를 보자.

appendFooter(s) 이 함수는 무언가에 s를 바닥글로 첨부할까? 아니면 s에 바닥글을 첨부할까? 인수 s는 입력일까 출력일까? 함수 선언부를 찾아보면 분명해진다.

```java
public void appendFooter(StringBuffer report);
```

인수 s가 출력 인수라는 사실은 분명하지만 함수 선언부를 찾아보고야 알았다. 함수 선언부를 찾아보는 행위는 코드를 보다 주춤하는 행위와 동급이다. 인지적으로 거슬린다는 뜻이므로 피해야 한다.

객체 지향 프로그래밍이 나오기 전에는 출력 인수가 불가피한 경우도 있었다. 하지만 객체지향 언어에서는 출력 인수를 사용할 필요가 거의없다. 출력 인수로 사용하라고 설계한 변수가 바로 this이기 때문이다. 다시 말해서, appendFooter는 다음과 같이 호출하는 편이 낫다.

```java
report.appendFooter();
```

일반적으로 출력 인수는 피해야 한다. 함수에서 뭔가의 상태를 변경해야 한다면 함수가 속하는 객체의 상태를 변경하는 방법을 택한다.

## 3.8 명령과 조회를 분리하라

함수는 뭐가를 수행하거나 뭔가에 답하거나 둘 중 하나만 해야 한다. 둘 다 하면 안 된다. 객체 상태를 변경하거나 아니면 개체 정보를 반환하거나 둘 중 하나다. 둘 다 하면 혼란을 초래한다. 다음 함수를 살펴보자

```java
public boolean set(String attribute, String value);
```

이 함수는 이름이 attribute인 속성으 찾아 값을 value로 설정한 후 성공하면 true를 반환하고 실패하면 false를 반환한다. 그래서 다음과 같이 괴상한 코드가 나온다.

```java
if(set("username","unclebob"))
```

독자 입장에서 코드를 읽어보자 무슨 뜻일까? "username"이 "unclebob"으로 설정되어 있는지 확인하는 코드인가? 아니면 "username"을 "unclebob"으로 설정하는 코드인가? 함수를 호출하는 코드를 봐서는 의미가 애매하다. 'set'이란 단어가 동사인지 형용사인지 분간하기 어려운 탓이다. 함수이름을 바꾸는 방법도 있지만 근본적인 해결책은 명령과 조회를 분리해서 혼란을 애초에 뿌리 뽑는 방법이다.

```java
if (attributeExists("username")){
 setAttribute("username","unclebob");
}
```

## 3.9 오류 코드보다 예외를 사용하라

명령 함수에서 오류 코드를 반환하는 방식은 명령/조회 분리 규칙을 미묘하게 위반한다. 자칫하면 if 문에서 명령을 표현식으로 사용하기 쉬운 탓이다.

```java
if(deletePage(page) == E_OK)
```

위 코드는 동사/형용사 혼란을 일으키지 않는 대신 여러 단계로 중첩되는 코드를 야기한다.

오류 코드를 반환하면 호출자는 오류 코드를 곧바로 처리해야 한다는 문제에 부딪힌다.

```java
if(deletePage(page) == E_OK){
 if(registry.deleteReference(page.name) == E_OK){
  if(configKeys.deleteKey(page.name.makeKey() == E_OK){
   logger.log("page deleted");
  }else{
    ...
   }
 }
}
```

반면 오류 코드 대신 예외를 사용하면 오류 처리 코드가 원래 코드에서 분리되어 코드가 깔끔해진다.

```java
try{
 deletePage(page);
 registry.deleteReference(page.name);
 configKeys.deleteKey(page.name.makeKey());
}catch{
 logger.log(e.getMessage());
}
```

### Try/Catch 블록 뽑아내기

try/catch 블록은 원래가 추하다. 코드 구조에 혼란을 일으키며, 정상적인 동작과 오류 처리 동작을 뒤섞는다. 그러므로 try/catch 블록을 별도 함수로 뽑아내는 편이 낫다.

```java
public void delete(Page page){
 try{
  deletePageAllReferences(page);
 }catch(Exception e) {
  logError(e)
 }
}

private void deletePageAllReferences(Page page) throws Exception{
 deletePage(page);
 registry.deleteReference(page.name);
 configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e){
 logger.log(e.getMessage());
}
```

위에서 delete 함수는 오류만 처리한다. 그래서 코드를 이해하기 쉽다. 이렇게 정상적인 동작과 오류 처리 동작을 분리하면 코드를 이해하고 수정하기 쉬워진다.

### 오류 처리도 한 가지 작업이다

함수는 '한 가지'작업만 해야 한다. 오류 처리도'한 가지' 작업에 속한다. 그러므로 오류를 처리하는 함수는 오류만 처리해아 마땅하다. 함수에 키워드 try가 있다면 함수는 try 문으로 시작해서 catch/finally 문으로 끝나야 한다는 말이다.

### Error.java 의존성 자석

오류 코드를 반환한다는 소리는, 클래스든 열거형 변수든, 어디선가 오류를 정의한다는 뜻이다.

```java
public enum Error{
 OK,
 INVALID,
 NO_SUCH,
 LOCKED,
 ...
}
```

위와 같은 클래스는 의존성 자석이다. 다른 클래서에서 Error enum을 포함해서 사용해야 하므로, 즉 Error enum이 변한다면 Error enum을 사용하는 클래스 전부를 다시 컴파일하고 다시 배치해야한다. 그래서 Error 클래스를 변경하기가 어려워진다.

오류 코드 대신 예외를 사용하면 새 예외는 Exception 클래스에서 파생한다. 따라서 재컴파일/재배치 없이도 새 예외 클래스를 추가할 수 있다.

## 3.10 반복하지 마라

중복은 문제다. 코드 길이가 늘어날 뿐 아니라 알고리즘이 변하면 그 알고리즘이 들어간 모든 곳을 손바야 하니까. 게다가 어느 한곳이라도 빠뜨리면 오류가 발생할 확률도 사용횟수만큼 높다.

중복을 없애면 모듈 가속성이 크게 높아진다. 어쩌면 중복은 소프트웨어에서 모든 악의 근원이다. 많은 원칙과 기법이 중복을 없애거나 제어할 목적으로 나왔다.

## 3.11 구조적 프로그래밍

어떤 프로그래머는 엣저 데익스트라의 구조적 프로그래밍 원칙을 따른다. 데익스트라는 모든 함수와 함수 내 모든 블록에 입구와 출구가 하나만 존재해야 한다고 말했다. 즉, 함수는 return 문이 하나여야 한다는 소리다. 루프 안에서 break나 continue를 사용해선 안되며 goto는 절대로, 절대로 안 된다.

구조적 프로그래밍의 목표와 규울은 공감하지만 함수가 작다면 위 규칙은 별 이익을 제공하지 못한다. 함수가 아주 클 때만 상당한 이익을 제공한다.

그러므로 함수를 작게 만든다면 간혹 returm, break, continue를 여러 차례 사용해도 괜찮다. 오히려 때로는 단일 입/출구 규칙보다 의도를 표현하기 쉬워진다. 반면, goto문은 큰 함수에서만 의미가 있으므로 피하는 편이 낫다.

## 3.12 함수를 어떻게 짜죠?

소프트웨어를 짜는 행위는 여느 글짓기와 비슷하다. 논문이나 기사를 쓸 때는 먼저 생각을 기록한 후 읽기 좋게 다듬는다. 초안은 대개 서투르고 어수선하므로 원하는 대로 읽힐 때까지 말을 다듬고 문장을 고치고 문단을 정리한다.

필자가 함수를 짤 때도 마찬가지다. 처음에는 길고 복잡하다. 들여쓰기 단계도 많고 중복된 루프도 많다. 인수 목록도 아주 길다. 이름은 즉흥적으고 코드는 중복된다. 그런 다음 필자는 코드를 다듬고, 함수를 만들고, 이름을 바꾸고, 중복을 제거한다. 메소드를 줄이고 순서를 바꾼다. 때로는 전체 클래스를 쪼개기도 한다. 종내에는 이 장에서 설명한 규칙을 따르는 함수가 얻어진다.

처음부터 탁 짜내지 않는다. 그게 가능한 사람은 없으리라.

## 3.13 결론

모든 시스템은 특정 응용 분야 시스템을 기술할 목적으로 프로그래머가 설계한 응용 분야언어로 만들어진다. 함수는 그 언어에서 동사며, 클래스는 명사다.

프로그래밍의 기술은 언제나 언어 설계의 기술이다. 예전에도 그랬고 지금도 마찬가지다.

대가 프로그래머는 시스템을(구현할) 프로그램이 아니라 (풀어갈)이야기로 여긴다. 프로그래밍 언어라는 수단을 사용해 좀더 풍부하고 좀더 표현력이 강한 언어를 만들어 이야기를 풀어간다. 시스템에서 발생하는 모든 동작을 설명하는 함수 계층이 바로 그 언어에 속한다.

이 장은 함수를 잘 만드는 기교를 소개했다. 여기서 설명한 규칙을 따른다면 길이가 짧고, 이름이 좋고, 체계가 잡힌 함수가 나오리라. 하지만 진짜 목표는 시스템이라는 이야기를 풀어가는 데 있다는 사실을 명심하기 바란다. 여러분이 작성하는 함수가 분명하고 정확한 언어에 깔끔하게 맞아야 이야기를 풀어가기가 쉬워진다는 사실을 기억하기 바란다.

### 참조

> 로버트 C. 마틴, 『Clean Code 클린 코드』, 박재호, 이해영 옮김, 케이앤피북스(2010), P86-96.
>