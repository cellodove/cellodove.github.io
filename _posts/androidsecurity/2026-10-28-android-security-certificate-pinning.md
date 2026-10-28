---
title : "[Android] 인증서 피닝과 CertificatePinner, 그리고 한계"
excerpt: "TLS 기본 검증 위에 한 겹 더하는 인증서 피닝의 원리, CertificatePinner·네트워크 보안 구성 구현, 그리고 도입 전에 알아야 할 한계"

categories :
  - AndroidSecurity

toc: true
toc_sticky: true
last_modified_at: 2026-10-28
---

![certificate_pinning_image1.jpg](/assets/images/certificate_pinning_image1.jpg?raw=true)

## 개요

앱과 서버 사이의 통신은 대부분 HTTPS(TLS)로 보호된다. 하지만 TLS는 "기기가 신뢰하는 CA가 서명한 인증서라면 믿는다"는 전제 위에 서 있어서, 그 CA 체계 어딘가가 뚫리면 중간자(MITM)가 끼어들 여지가 생긴다.

인증서 피닝(certificate pinning)은 여기에 한 겹을 더한다. 앱이 "이 서버라면 이 공개키여야 한다"는 값(핀)을 미리 들고 있다가, 연결할 때 서버 인증서 체인에 그 키가 없으면 연결을 끊는다.

다만 Android 공식 문서는 피닝을 **권장하지 않는다**. 서버 인증서가 바뀌어 핀이 어긋나면, 앱을 업데이트하기 전까지 접속이 끊기기 때문이다. 그래서 피닝은 무엇을 막고 무엇을 못 막는지, 운영 비용까지 알고 도입해야 한다.

이 글에서는 TLS가 서버를 어떻게 검증하는지부터 시작해서 피닝이 무엇을 막는지, OkHttp `CertificatePinner`와 네트워크 보안 구성(Network Security Configuration)으로 어떻게 구현하는지, 그리고 도입 전에 꼭 알아야 할 한계를 정리한다.

## TLS 란

TLS(Transport Layer Security)는 통신을 **암호화**하고, 상대 서버가 진짜인지 **인증**하는 프로토콜이다. HTTPS는 HTTP를 TLS 위에서 주고받는 것이다.

핸드셰이크에서 서버가 인증서 체인을 보내면, 클라이언트는 세 가지를 확인한다.

- 체인 끝의 루트 CA가 기기의 신뢰 저장소에 있는가
- 유효기간이 지나지 않았고 폐기되지 않았는가
- 인증서의 이름이 접속한 호스트와 일치하는가

모두 통과하면 키를 교환하고, 이후 트래픽은 암호화된다.

### 약점 — 신뢰를 CA에 위임한다

TLS 검증의 핵심은 "신뢰 저장소에 있는 CA 중 **어느 하나라도** 서명했으면 통과"라는 점이다. 이 서버가 실제로 어느 CA에서 인증서를 받았는지는 따지지 않는다. 그래서 다음과 같은 경우 가짜 인증서가 정상처럼 통과할 수 있다.

- 신뢰 저장소에 있는 CA 하나가 인증서를 잘못 발급했거나 침해된 경우
- 루팅된 기기에서 공격자(또는 분석가)가 자신의 CA를 시스템 신뢰 저장소에 넣은 경우
- 앱이 사용자가 설치한 CA까지 신뢰하도록 설정된 경우

![certificate_pinning_1.png](/assets/images/certificate_pinning_1.png?raw=true)

참고로 Android 7.0(API 24) 이상을 타깃으로 하는 앱은 기본적으로 **사용자가 설치한 CA를 신뢰하지 않는다**(API 23 이하 타깃은 신뢰한다). 그래서 설정 앱에서 프록시 CA를 설치하는 것만으로는 최신 앱의 HTTPS 트래픽을 볼 수 없고, 시스템 저장소에 CA를 넣으려면 보통 루팅이 필요하다.

## 인증서 피닝이란

피닝은 플랫폼 기본 검증을 통과한 연결에 대해, 서버 인증서 체인 안에 **앱이 미리 지정한 공개키가 하나라도 있는지** 한 번 더 확인하는 것이다. 없으면 신뢰 CA가 서명한 인증서라도 연결을 끊는다.

![certificate_pinning_2.png](/assets/images/certificate_pinning_2.png?raw=true)

### 무엇을 핀하나 — 공개키 해시

핀 값은 보통 인증서의 공개키 정보(SubjectPublicKeyInfo, SPKI)를 SHA-256으로 해시하고 Base64로 인코딩한 값이다. 인증서 전체가 아니라 공개키를 핀하는 이유는, 인증서를 갱신해도 같은 키 쌍을 계속 쓰면 핀이 그대로 유효하기 때문이다.

### 체인의 어느 인증서에 핀하나

![certificate_pinning_3.png](/assets/images/certificate_pinning_3.png?raw=true)

| 핀 대상 | 장점 | 단점 |
| --- | --- | --- |
| 서버 인증서(leaf) | 범위가 가장 좁아 가장 강하다 | 서버가 키를 바꾸면 바로 깨진다 |
| 중간 CA | 같은 CA에서 재발급하면 유지된다 | 같은 중간 CA가 발급한 다른 인증서도 통과한다 |
| 루트 CA | 가장 잘 안 깨진다 | 그 CA 계열 전체를 허용해 방어 범위가 넓어진다 |

범위를 좁힐수록 강하지만 운영 중에 깨지기 쉽다. 그래서 현재 키와 함께 다음에 교체할 키(백업 핀)를 같이 넣어 두는 것이 기본이다.

### 기본 검증을 대체하지 않는다

피닝은 기본 검증 **위에** 더하는 것이다. OkHttp의 `CertificatePinner`도 TLS 핸드셰이크(=플랫폼 기본 검증)가 성공한 뒤, 연결을 실제로 쓰기 전에 핀을 확인한다. 그래서 기본 검증을 통과하지 못하는 자체 서명 인증서는 핀을 걸어도 연결되지 않는다. 피닝을 하겠다고 기본 검증을 끄면 체인·만료·이름 검증이 모두 사라져 오히려 더 위험해진다.

## CertificatePinner 로 구현하기

OkHttp를 쓴다면 `CertificatePinner`로 호스트별 핀을 등록해 클라이언트에 붙이면 된다.

```kotlin
val pinner = CertificatePinner.Builder()
    // "sha256/" + 공개키(SPKI) SHA-256 의 Base64
    .add("api.example.com", "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=")
    // 백업 핀: 다음에 교체할 키
    .add("api.example.com", "sha256/BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=")
    .build()

val client = OkHttpClient.Builder()
    .certificatePinner(pinner)
    .build()
```

같은 호스트에 핀을 여러 개 등록하면 그중 **하나만** 체인에 있어도 통과한다.

### 호스트 패턴

| 패턴 | 매칭됨 | 매칭 안 됨 |
| --- | --- | --- |
| `api.example.com` | `api.example.com` | 그 외 전부 |
| `*.example.com` | `api.example.com`, `www.example.com` (정확히 한 단계) | `example.com`, `a.b.example.com` |
| `**.example.com` | `example.com`, `api.example.com`, `a.b.example.com` (0단계 이상) | — |

주의할 점은 **패턴에 맞지 않는 호스트는 오류 없이 피닝 없이 연결된다**는 것이다. `*.example.com`만 등록해 두고 `example.com`으로 요청하면 피닝이 조용히 빠진다.

### 핀 값 구하기

**방법 1. openssl로 계산**

```bash
openssl s_client -connect api.example.com:443 -servername api.example.com < /dev/null \
  | openssl x509 -pubkey -noout \
  | openssl pkey -pubin -outform der \
  | openssl dgst -sha256 -binary \
  | openssl enc -base64
```

이 명령은 서버 인증서(leaf)의 핀을 계산한다. 중간 CA의 핀이 필요하면 `-showcerts`로 체인을 받아 해당 인증서에 같은 과정을 적용한다.

**방법 2. 일부러 틀린 핀으로 연결해 보기**

OkHttp 문서가 안내하는 방법이다. 아무 값이나 핀으로 넣고 연결하면 `SSLPeerUnverifiedException`이 발생하고, 그 메시지에 서버 체인의 인증서별 핀 값이 나열된다.

어느 방법이든 **신뢰할 수 있는 네트워크에서** 구해야 한다. 그 순간 경로에 MITM이 있으면 공격자의 키를 핀으로 박게 된다.

### 네트워크 보안 구성으로 선언하기

코드 대신 네트워크 보안 구성(Network Security Configuration)의 `<pin-set>`으로 XML에 선언할 수도 있다.

```xml
<!-- res/xml/network_security_config.xml -->
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">example.com</domain>
        <pin-set expiration="2027-06-30">
            <pin digest="SHA-256">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=</pin>
            <!-- 백업 핀 -->
            <pin digest="SHA-256">BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

이 파일은 매니페스트의 `application` 요소에 `android:networkSecurityConfig` 속성으로 연결한다.

| 항목 | CertificatePinner | 네트워크 보안 구성 `<pin-set>` |
| --- | --- | --- |
| 설정 방식 | 코드(Kotlin) | XML |
| 적용 범위 | 핀을 붙인 그 `OkHttpClient`만 | 플랫폼 기본 TrustManager를 쓰는 연결 전체 |
| 핀 만료 | 없음 | `expiration` 속성 지원 |
| 디버그 예외 | 직접 구현해야 함 | `<debug-overrides>` 제공 |

## 한계

### 1. 운영 리스크 — 핀이 어긋나면 전부 막힌다

Android 보안 가이드는 인증서 피닝을 Android 앱에 **권장하지 않는다**고 명시한다. 서버가 CA를 바꾸는 등 설정이 변경되면, 앱을 업데이트하기 전까지 서버에 접속할 수 없게 되기 때문이다. OkHttp 문서 역시 서버의 TLS 관리자와 합의 없이 피닝하지 말라고 경고한다.

핀은 앱 바이너리에 들어가 배포되고, 업데이트하지 않은 사용자에게는 옛 핀이 계속 남는다. 핀이 어긋나는 순간 그 버전을 쓰는 사용자는 **전부 접속 불가**가 된다.

![certificate_pinning_4.png](/assets/images/certificate_pinning_4.png?raw=true)

그래도 피닝을 한다면 공식 문서가 요구하는 안전장치를 갖춰야 한다.

- 백업 핀을 여러 개 넣고, 그중 하나 이상은 **직접 통제하는 키**로 한다
- 핀에 짧은 만료 기간을 둔다(네트워크 보안 구성의 `expiration`). 단, 만료되면 피닝이 꺼지므로 그 이후에는 보호도 사라진다
- 서버 팀과 인증서·키 교체 일정을 공유한다(핀 롤오버 계획)

### 2. 기기를 장악한 공격자는 우회한다

`CertificatePinner`와 네트워크 보안 구성 피닝의 검사 로직은 모두 **앱 프로세스 안에서** 돈다. 루팅된 기기에서 Frida 같은 후킹 도구로 핀 검사 메서드를 무력화하거나 앱을 변조해 재서명하면 피닝은 그냥 꺼진다.

즉 피닝이 막는 것은 **네트워크 경로에 있는 공격자**이지, **기기를 장악한 공격자**가 아니다.

핀 검사를 자바/코틀린이 아닌 **네이티브 코드(C/C++)로 옮기면** 분석과 우회가 더 어려워진다. 하지만 네이티브 함수도 후킹할 수 있기 때문에 완전한 해결책은 아니고, 공격에 드는 비용을 높이는 수단으로 보는 것이 맞다.

또 네트워크 보안 구성의 `<debug-overrides>`에 넣은 CA는 기본적으로 피닝을 우회한다(`overridePins` 기본값 `true`). 이 설정은 `debuggable` 빌드에서만 적용되므로, 디버그 빌드가 외부로 배포되지 않도록 주의해야 한다.

### 3. 핀 값과 검사 지점이 노출된다

APK를 디컴파일하면 핀 값과 검사 위치가 그대로 드러난다. 핀 값 자체는 공개키 해시라 비밀이 아니지만, 검사 지점이 드러나면 **그곳이 곧 후킹 포인트**가 된다.

### 4. 적용 범위에 빈틈이 생기기 쉽다

- 호스트 패턴에 맞지 않는 요청은 오류 없이 피닝 없이 나간다(`*.` vs `**.`)
- `CertificatePinner`는 핀을 붙인 그 `OkHttpClient`에만 적용된다. 별도 클라이언트나 자체 네트워크 스택을 쓰는 서드파티 SDK 트래픽에는 걸리지 않는다
- CDN·서드파티 도메인까지 핀하면, 내가 통제할 수 없는 쪽의 인증서 교체에 따라 앱이 깨질 수 있다

## 정리

| 항목 | 내용 |
| --- | --- |
| 목적 | "신뢰 CA 중 아무나 서명하면 통과"하는 TLS 기본 검증 위에, 지정한 공개키인지 한 번 더 확인 |
| 막는 것 | 잘못 발급되거나 침해된 CA, 기기에 심긴 CA를 이용한 네트워크 경로의 MITM |
| 못 막는 것 | 루팅·후킹·리패키징으로 앱 자체를 조작하는 공격자 |
| 핀 값 | 공개키(SPKI)의 SHA-256 Base64. leaf → 중간 CA → 루트로 갈수록 덜 깨지지만 방어 범위가 넓어진다 |
| 구현 | OkHttp CertificatePinner(클라이언트 단위) 또는 네트워크 보안 구성 pin-set(앱 전체, 만료 지원). 백업 핀 필수 |
| 한계 | ① 핀이 어긋나면 해당 버전 사용자 전체 차단(Android 공식 문서도 비권장) ② 기기를 장악하면 우회 ③ 검사 지점 노출 ④ 호스트 패턴·클라이언트 범위의 빈틈 |

<aside markdown="1">
💡 포인트

피닝은 TLS 기본 검증을 대체하는 게 아니라 그 위에 한 겹 더하는 것이다.

백업 핀 없이 핀 하나만 박으면, 서버 키가 바뀌는 순간 업데이트하지 않은 사용자 전부가 막힌다. 서버 팀과 키 교체 계획부터 맞추자.

`*.example.com`은 한 단계만 매칭된다. 패턴에 안 맞는 호스트는 조용히 피닝 없이 연결되니 실제 요청 호스트를 전부 확인하자.

피닝은 네트워크 경로의 공격자를 막는 수단이다. 기기를 장악한 공격자까지 막으려면 루팅·후킹 탐지, 앱 무결성 검증 같은 다른 방어를 함께 써야 한다.

</aside>

## 참조

[Security with network protocols](https://developer.android.com/privacy-and-security/security-ssl)

[Network security configuration](https://developer.android.com/privacy-and-security/security-config)

[Certificate and Public Key Pinning](https://owasp.org/www-community/controls/Certificate_and_Public_Key_Pinning)
