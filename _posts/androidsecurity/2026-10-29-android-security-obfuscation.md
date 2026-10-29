---
title : "[Android] 난독화 — 코드를 읽기 어렵게 만들 뿐, 비밀을 지켜주지는 않는다"
excerpt: "R8 난독화가 해 주는 것과 keep 규칙·매핑 파일 관리, 그리고 난독화가 지켜주지 못하는 문자열·비밀 값·실행 중 분석의 한계 정리"

categories :
  - AndroidSecurity

toc: true
toc_sticky: true
last_modified_at: 2026-10-29
---

![obfuscation_image1.jpg](/assets/images/obfuscation_image1.jpg?raw=true)

## 개요

앞의 글들에서 앱 안의 방어는 공격자가 기기를 장악하면 결국 우회될 수 있고, 중요한 판정은 서버가 내려야 한다는 점을 봤다. 그렇다고 앱 안의 코드를 그대로 열어 둘 필요는 없다. 공격자가 앱을 분석하는 데 드는 시간을 늘리는 방법이 **난독화**(obfuscation)다.

APK는 압축 파일이고, 그 안의 DEX 바이트코드에는 클래스·메서드 이름과 코드 구조가 그대로 남아 있다. 그래서 디컴파일러에 넣으면 원래 소스에 꽤 가까운 코드가 나온다. 난독화는 이 결과물을 사람이 읽기 어렵게 만든다.

하지만 난독화는 "읽기 어렵게" 만드는 것이지 "읽을 수 없게" 만드는 것이 아니다. 이 글에서는 Android의 기본 도구인 **R8**로 무엇을 할 수 있는지, 난독화가 지켜주지 **못하는** 것은 무엇인지, 그리고 어디까지 투자할지 정하는 기준을 정리한다.

## 난독화란

난독화는 프로그램의 동작은 그대로 두고, 코드를 분석하기 어렵게 바꾸는 기법이다. OWASP 모바일 보안 표준(MASVS)은 난독화를 안티 디버깅, 변조 방지와 함께 **심층 방어**(defense-in-depth) 수단으로 분류한다. 그리고 이런 수단이 적절한 보안 설계를 **대신해서는 안 된다**고 분명히 적고 있다.

즉 난독화의 목표는 "뚫리지 않게"가 아니라 "뚫는 데 시간과 비용이 더 들게"다.

### 난독화 기법의 종류

| 기법 | 하는 일 | R8 기본 제공 |
| --- | --- | --- |
| 이름 바꾸기 | 클래스·메서드·필드 이름을 `a`, `b` 같은 짧은 이름으로 바꾼다 | O |
| 코드 축소·최적화 | 쓰지 않는 코드를 지우고, 메서드 인라인·클래스 병합으로 구조를 바꾼다. 난독화가 목적은 아니지만 결과적으로 원본과 구조가 달라진다 | O |
| 문자열 암호화 | 문자열 상수를 암호화해 두고 실행 중에 복호화한다 | X |
| 제어 흐름 난독화 | 분기·반복 구조를 복잡하게 바꾸고, 결과에 영향 없는 코드를 섞는다 | X |
| 리소스·에셋 암호화 | `assets/`, `res/raw/` 의 파일을 암호화해 둔다 | X |
| 동적 로딩·패킹 | 코드 일부를 앱 본체 밖에 두거나 암호화해 두고, 실행 중에 풀어서 로드한다 | X |
| 네이티브 난독화 | C/C++ 코드의 심볼 제거, 제어 흐름 평탄화, 문자열 인코딩 등 | X (R8 대상이 아님) |

R8이 기본으로 해 주는 것은 위의 두 가지뿐이다. 나머지는 별도의 상용·오픈소스 도구가 필요하다.

## R8로 하는 난독화

### 켜는 방법

R8은 Android Gradle 플러그인(AGP)에 포함된 앱 최적화 도구다. 릴리스 빌드에서 다음처럼 켠다.

```kotlin
android {
    buildTypes {
        release {
            isMinifyEnabled = true    // 코드 축소 · 최적화 · 난독화
            isShrinkResources = true  // 쓰지 않는 리소스 제거
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }
}
```

- 공식 문서는 릴리스 빌드에서는 **항상** 켜라고 권장한다. 기본 규칙 파일은 최적화가 켜진 `proguard-android-optimize.txt` 를 쓴다
- AGP 8.0부터 기본인 **R8 전체 모드**(full mode)는 리플렉션 같은 동적 기능에 대한 가정이 엄격해서, keep 규칙이 더 필요할 수 있다

### R8의 난독화는 보안 기능이 아니다

R8은 코드 축소, 최적화, 이름 바꾸기(`com.example.MyActivity` → `a.b.a`), 리소스 축소를 한 번에 한다. 흥미로운 점은 공식 문서가 R8의 난독화를 **DEX 크기를 줄이는 수단**으로 설명한다는 것이다. 보안 기능으로 설계된 것이 아니라, 이름이 짧아지면서 덤으로 읽기 어려워지는 것에 가깝다.

실제로 어떻게 바뀌는지 보면 한계가 바로 보인다.

![obfuscation_1.png](/assets/images/obfuscation_1.png?raw=true)

이름은 사라졌지만 URL, 로그 문자열, `"ACTIVE"` 같은 비교 값, `Log.d` 같은 프레임워크 API는 그대로다. 분석하는 사람은 이런 단서에서 시작해 코드의 역할을 거꾸로 추적할 수 있다. (그림은 설명을 위한 예시이고, 실제 R8 결과는 인라인 등으로 구조가 더 달라질 수 있다.)

### keep 규칙 — 이름이 바뀌면 안 되는 코드

이름을 바꾸면 깨지는 코드가 있다. 이름을 **문자열로 찾아 쓰는** 코드다.

- 리플렉션으로 클래스·메서드를 이름으로 찾는 코드
- Gson처럼 리플렉션으로 필드 이름을 읽는 직렬화
- 네이티브 코드(JNI)에서 이름으로 찾는 Java 메서드·필드
- 매니페스트에 선언된 Activity·Service 같은 컴포넌트 (시스템이 이름으로 생성하므로 바뀌지 않는다)

이런 코드는 keep 규칙으로 R8에게 알려 준다. 공식 문서는 범위를 좁게 잡으라고 권한다.

```txt
# 클래스는 최적화 대상으로 두고, 필요한 멤버만 남긴다
-keepclassmembers,allowoptimization class com.example.MyClass {
    void someSpecificMethod();
}
```

- 수식어 없는 `-keep` 은 해당 클래스의 최적화를 모두 막으므로 꼭 필요할 때만 쓴다
- 패키지 전체를 keep 하는 규칙은 R8을 처음 켤 때 임시로만 쓴다
- 가능하면 어노테이션 기반 규칙을 써서 코드와 규칙의 연결을 명확히 한다

keep 규칙으로 남긴 이름은 난독화되지 않는다는 점도 기억해 두자. 넓은 규칙을 남발하면 난독화 효과도 그만큼 줄어든다.

### 매핑 파일 — 난독화를 되돌리는 열쇠

이름이 바뀌면 크래시 스택 트레이스도 `a.b.a` 처럼 읽을 수 없게 된다. 그래서 R8은 원래 이름과 바뀐 이름의 대응표인 `mapping.txt` 를 만든다.

![obfuscation_2.png](/assets/images/obfuscation_2.png?raw=true)

- 위치는 `app/build/outputs/mapping/<빌드 변형>/mapping.txt` 다
- 빌드할 때마다 덮어써지므로, **릴리스마다 사본을 보관**해야 한다
- Android App Bundle(AAB)에는 `mapping.txt` 가 자동으로 함께 들어간다. Play Console은 이 파일로 수집한 크래시를 원래 이름으로 보여 준다
- 직접 되돌릴 때는 command-line tools의 `retrace` 를 쓴다. Android Studio Otter 3 Feature Drop과 AGP 9.0부터는 Logcat이 자동으로 되돌려 준다

```bash
retrace app/build/outputs/mapping/release/mapping.txt trace.txt
```

이 파일은 개발자에게 꼭 필요하지만, 반대로 외부에 나가면 난독화가 의미 없어진다. 공개 저장소나 공개 배포물에 섞여 나가지 않게 관리해야 한다.

## 난독화가 지켜주지 못하는 것

### 1. 문자열은 그대로 남는다

R8의 난독화는 이름을 바꾸는 것이고, 문자열 상수를 암호화하지는 않는다. 서버 주소, API 경로, 에러 메시지, 비교에 쓰는 값이 디컴파일 결과에 그대로 보인다. 분석은 보통 이런 문자열을 검색하는 데서 시작한다.

로그 메시지도 문자열이라 그대로 남는다. 릴리스 빌드에서는 민감한 정보를 로그로 남기지 않는 것이 좋다.

### 2. 앱에 넣은 비밀은 결국 노출된다

API 키, 암호화 키, 고정 비밀번호를 코드나 `strings.xml` 에 넣으면, 공식 문서의 표현대로 리버스 엔지니어링 도구로 "아주 쉽게" 꺼낼 수 있다.

문자열 암호화 도구를 써도 근본적으로 달라지지 않는다. 복호화하는 코드와 키도 같은 앱 안에 있기 때문이다. 정적 분석은 어려워지지만, 실행 중에 복호화된 값을 보는 것까지 막지는 못한다.

- 서버와 통신하는 데 필요한 비밀은 앱에 넣지 말고 서버가 갖는다
- 기기에서 만들어 쓰는 키는 Android Keystore에 둔다. 키 자료가 앱 메모리로 나오지 않는다

### 3. 실행 중 분석은 막지 못한다

난독화는 코드를 **읽는** 정적 분석을 어렵게 한다. 앱을 실행하면서 메서드의 인자와 반환값을 관찰하는 동적 분석에는 효과가 제한적이다. 이름이 `a.b.a` 여도 그 메서드가 무엇을 받고 무엇을 돌려주는지는 실행해 보면 보인다.

특히 Android·Java 프레임워크 API(`Log`, 네트워크, 암호화 API 등)는 앱 밖에 있어서 이름을 바꿀 수 없다. 이런 호출 지점은 난독화와 상관없이 분석의 출발점이 된다.

### 4. 네이티브 코드도 숨겨지지 않는다

중요한 로직을 C/C++로 옮기면 분석 난이도가 올라가는 것은 맞다. 하지만 JNI 함수를 이름 규칙(`Java_패키지_클래스_메서드`)으로 연결하면, 심볼을 제거해도 이 이름은 외부에 공개된 심볼로 남는다. `JNI_OnLoad` 에서 `RegisterNatives` 로 동적 등록하면 노출을 줄일 수 있지만, 이것도 분석을 늦출 뿐이다.

## 어디까지 투자할까

![obfuscation_3.png](/assets/images/obfuscation_3.png?raw=true)

1. **앱 밖으로 옮긴다** — 비밀 값, 무결성 판정, 중요한 비즈니스 로직은 서버에 둔다. 앱에 없는 코드는 분석할 수 없다. 이것이 난독화보다 먼저다
2. **R8 난독화를 기본으로 켠다** — 비용이 거의 없고, 앱 크기와 성능에도 이득이다. 대신 릴리스 빌드로 충분히 테스트하고 매핑 파일을 관리한다
3. **추가 난독화는 필요한 곳에만** — 문자열 암호화, 제어 흐름 난독화, 네이티브 난독화는 빌드 시간·앱 크기·성능·호환성 문제와 크래시 분석의 어려움이라는 비용이 따른다. 앱에 꼭 남아야 하는 탐지 로직처럼, 분석 시간을 늘리는 것 자체가 가치 있는 부분에 한정한다

MASVS도 이런 보호 수단이 없다는 것만으로 취약점은 아니라고 말한다. 난독화는 위협 모델에 맞춰 더하는 층이지, 보안의 기반이 아니다.

## 정리

| 항목 | 내용 |
| --- | --- |
| 목적 | 코드 분석에 드는 시간과 비용을 늘린다. 막는 것이 아니라 늦춘다 |
| R8 기본 제공 | 이름 바꾸기, 코드 축소·최적화, 리소스 축소. 릴리스 빌드에 항상 켠다 |
| keep 규칙 | 리플렉션·직렬화·JNI 등 이름으로 찾는 코드만, 최대한 좁게 |
| 한계 | ① 문자열은 남는다 ② 앱 안의 비밀은 결국 노출 ③ 실행 중 분석은 못 막는다 ④ 네이티브도 숨겨지지 않는다 |

<aside markdown="1">
💡 포인트

R8 난독화는 원래 크기를 줄이려는 기능이다. 이름은 바뀌어도 문자열·프레임워크 호출·로직 흐름은 남는다.

비밀 값은 난독화로 숨기는 것이 아니라 앱에 넣지 않는 것이 답이다.

추가 난독화는 서버 이전과 R8 적용을 끝낸 뒤, 꼭 필요한 코드에만 비용을 따져서 더하자.

</aside>

## 참조

[Enable app optimization with R8](https://developer.android.com/topic/performance/app-optimization/enable-app-optimization)

[Add keep rules](https://developer.android.com/topic/performance/app-optimization/add-keep-rules)

[Fix optimization problems](https://developer.android.com/topic/performance/app-optimization/test-and-troubleshoot-the-optimization)

[R8 retrace](https://developer.android.com/tools/retrace)

[Hardcoded Cryptographic Secrets](https://developer.android.com/privacy-and-security/risks/hardcoded-cryptographic-secrets)

[MASVS-RESILIENCE](https://mas.owasp.org/MASVS/11-MASVS-RESILIENCE/)

[MASTG-KNOW-0033: Obfuscation](https://mas.owasp.org/MASTG/knowledge/android/MASVS-RESILIENCE/MASTG-KNOW-0033/)
