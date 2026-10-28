---
title : "[Android] Play Integrity API — 기기와 앱의 무결성은 서버가 판정한다"
excerpt: "Play Integrity API가 무엇을 판정하는지, 요청 흐름과 requestHash의 역할, 그리고 왜 판정을 앱이 아닌 서버에서 내려야 하는지 정리"

categories :
  - AndroidSecurity

toc: true
toc_sticky: true
last_modified_at: 2026-10-28
---

![play_integrity_image1.jpg](/assets/images/play_integrity_image1.jpg?raw=true)

## 개요

앞의 두 글에서 루팅 탐지나 인증서 피닝 같은 앱 안의 방어는, 공격자가 기기를 장악하면 결국 우회될 수 있다는 점을 봤다. 검사 코드가 공격자의 손 안에서 돌기 때문이다.

그렇다면 "이 요청이 정말 **변조되지 않은 내 앱**에서, **정상적인 기기**로부터 왔는가"는 누가 판단해야 할까. Android에서는 Google Play가 제공하는 **Play Integrity API**가 이 역할을 한다. 앱은 Google Play에서 암호화된 판정 토큰을 받아 서버로 전달만 하고, 판정 내용을 확인하고 통과 여부를 결정하는 일은 서버가 한다.

이 글의 핵심은 하나다. **판정 결과로 앱이 분기하게 두지 말고, 서버가 판정과 요청 처리를 한 번에 한다.** 아래에서는 Play Integrity API가 무엇을 판정하는지, 요청 흐름이 어떻게 되는지, 왜 판정을 서버에서 내려야 하는지, 그리고 운영할 때 주의할 점을 정리한다.

## Play Integrity API 란

Play Integrity API는 사용자의 동작과 서버 요청이 Google Play에서 설치된 **정품 앱**에서, **정품 인증된 Android 기기** 위에서 오는지 확인하도록 돕는 API다. 이전에 쓰이던 SafetyNet Attestation API는 2022년에 지원 중단(deprecated)되었고 2025년 1월에 완전히 종료되어, 지금은 Play Integrity API로 옮겨야 한다.

### 앱 안의 루팅 탐지와 무엇이 다른가

| 항목 | 앱 안의 루팅·변조 탐지 | Play Integrity API |
| --- | --- | --- |
| 판정하는 쪽 | 앱 코드 | Google Play (기기 쪽 신호 + Google 서버) |
| 결과 형태 | 앱 메모리 안의 `true`/`false` | 서명·암호화된 토큰. 앱은 내용을 읽거나 고칠 수 없다 |
| 결과를 믿는 쪽 | 앱 | 앱 서버 |
| 후킹으로 결과 조작 | 반환값만 바꾸면 된다 | 토큰을 위조할 수 없다. 대신 서버가 판정을 제대로 써야 의미가 있다 |

핵심은 **결과를 믿는 쪽이 앱이 아니라 서버**라는 점이다. 앱은 토큰을 전달하는 통로일 뿐이다.

## 판정 종류

서버가 토큰을 복호화하면 다음과 같은 JSON을 얻는다(주요 필드만 줄인 예시다).

```json
{
  "requestDetails": {
    "requestPackageName": "com.example.app",
    "requestHash": "2cp24z...",
    "timestampMillis": "1700000000000"
  },
  "appIntegrity": {
    "appRecognitionVerdict": "PLAY_RECOGNIZED",
    "packageName": "com.example.app",
    "certificateSha256Digest": ["..."],
    "versionCode": "42"
  },
  "deviceIntegrity": {
    "deviceRecognitionVerdict": ["MEETS_DEVICE_INTEGRITY"]
  },
  "accountDetails": {
    "appLicensingVerdict": "LICENSED"
  }
}
```

`requestDetails`는 "이 토큰이 어떤 요청을 위해 만들어졌는가"이고, 나머지 세 필드가 실제 판정이다.

| 판정 | 통과로 보는 값 | 그 밖의 값 |
| --- | --- | --- |
| 앱 무결성<br>`appRecognitionVerdict` | `PLAY_RECOGNIZED` — 앱과 서명 인증서가 Google Play가 배포한 버전과 일치한다 | `UNRECOGNIZED_VERSION` — 서명이나 패키지 이름이 Google Play 기록과 다르다(리패키징 등)<br>`UNEVALUATED` — 조건이 맞지 않아 평가하지 않았다 |
| 기기 무결성<br>`deviceRecognitionVerdict` | `MEETS_DEVICE_INTEGRITY` — 정품 인증 기기. Android 13 이상에서는 부트로더가 잠겨 있고 제조사 인증 OS로 부팅했다는 하드웨어 기반 증명이 있다 | `MEETS_BASIC_INTEGRITY` — 기본 검사만 통과. 부트로더가 풀려 있을 수 있다<br>(빈 값) — 후킹 같은 공격 징후나 루팅 같은 침해 징후가 있거나, 실제 기기가 아니다 |
| 계정<br>`appLicensingVerdict` | `LICENSED` — Google Play에서 설치하거나 업데이트했다 | `UNLICENSED` — Google Play가 아닌 경로로 설치(사이드로드)했다<br>`UNEVALUATED` — Google Play 미로그인 등으로 평가하지 않았다 |

기기 판정에는 라벨이 여러 개 함께 담길 수 있고, 최근 보안 업데이트까지 요구하는 `MEETS_STRONG_INTEGRITY`도 선택해서 받을 수 있다. 앱 안의 루팅 탐지가 "su 파일이 있는가" 같은 흔적을 찾는 것과 달리, 기기 판정은 부트로더 잠금 상태 같은 하드웨어 기반 증명을 근거로 한다. 그래서 앱 안에서 흔적을 숨기는 것만으로는 통과하기 어렵다.

이 밖에도 화면 캡처·오버레이 앱 실행 여부(`appAccessRiskVerdict`), Play 프로텍트 상태(`playProtectVerdict`), 최근 토큰 요청 빈도(`recentDeviceActivity`) 같은 선택 판정을 추가로 받을 수 있다.

## 요청 흐름

### 표준 요청과 클래식 요청

| 항목 | 표준 요청 (standard) | 클래식 요청 (classic) |
| --- | --- | --- |
| 지연 시간 | 수백 ms (미리 준비 필요) | 수 초 |
| 사용 빈도 | 자주, 필요할 때마다 | 드물게, 가장 중요한 동작에만 |
| 요청 결합 | `requestHash` | `nonce` |
| 재사용(replay) 방어 | Google Play가 자동으로 처리 | 서버가 직접 처리 |

특별한 이유가 없다면 표준 요청을 쓰면 된다. 두 방식 모두 판정 결과의 형식은 같다.

### 표준 요청의 흐름

![play_integrity_1.png](/assets/images/play_integrity_1.png?raw=true)

앱은 시작할 때 토큰 제공자를 미리 준비해 두고, 민감한 요청 직전에 그 요청을 해시한 `requestHash`를 넣어 토큰을 받는다. 그리고 **실제 요청과 토큰을 함께** 서버로 보낸다. 아래는 공식 문서의 예시를 Kotlin으로 옮긴 것으로, 직접 빌드해 보지는 않은 예시다.

```kotlin
// 1. 앱 시작 시: 토큰 제공자 준비
val integrityManager = IntegrityManagerFactory.createStandard(applicationContext)
integrityManager.prepareIntegrityToken(
    PrepareIntegrityTokenRequest.builder()
        .setCloudProjectNumber(CLOUD_PROJECT_NUMBER)
        .build()
).addOnSuccessListener { provider -> tokenProvider = provider }

// 2~3. 민감한 요청 직전: 요청 내용을 해시해 토큰 요청
val requestHash = sha256Base64(stableSerialize(sensitiveRequest))
tokenProvider.request(
    StandardIntegrityTokenRequest.builder()
        .setRequestHash(requestHash)
        .build()
).addOnSuccessListener { response ->
    // 4. 토큰을 실제 요청과 함께 서버로. 앱은 토큰 내용을 해석하지 않는다
    api.send(sensitiveRequest, integrityToken = response.token())
}
```

서버는 Google 서버의 `decodeIntegrityToken`으로 토큰을 복호화한 뒤 세 가지를 확인하고, 모두 통과할 때만 요청을 처리한다.

1. `requestHash`·패키지 이름·시각이 지금 받은 요청과 맞는가
2. 앱이 `PLAY_RECOGNIZED`, 계정이 `LICENSED`인가
3. 기기 판정에 `MEETS_DEVICE_INTEGRITY`가 있는가

### requestHash 와 nonce 가 하는 일

`requestHash`가 없으면 토큰은 "이 기기에서 만들어졌다"는 것만 보장하고, **어떤 요청을 위한 것인지**는 묶이지 않는다. 그러면 정상 기기에서 받은 토큰을 다른 요청에 붙여 쓰는 공격이 가능해진다. 요청 내용을 해시해 넣고 서버가 같은 방식으로 다시 계산해 비교하면, 토큰과 요청이 1:1로 묶인다.

클래식 요청에서는 `nonce`가 같은 역할을 한다. 공식 문서는 서버가 예측할 수 없는 일회용 값을 발급하고, 그 값을 요청 내용과 함께 해시해 `nonce`로 쓰는 방식을 권장한다. 서버는 발급한 값을 기록해 두었다가, 처음 쓰이는 값인지 확인한다.

두 값 모두 Google에 평문으로 전달되므로, 개인정보나 민감한 값을 그대로 넣지 말고 반드시 해시해서 넣어야 한다.

## 판정은 서버가 내려야 한다

Play Integrity를 붙였는데도 우회되는 가장 흔한 실수는, 판정 결과를 **앱에게 알려주고 앱이 분기하게 만드는 것**이다.

```txt
잘못된 흐름
1. 앱 → 서버 : 토큰
2. 서버 → 앱 : { "ok": true }  또는  { "ok": false }
3. 앱        : if (ok) 다음 단계로 진행, 아니면 종료
4. 앱 → 서버 : 실제 요청  ← 판정과 상관없이 처리됨
```

토큰은 위조할 수 없지만, 3번의 분기는 앱 안에 있다. 후킹으로 `ok`를 `true`로 바꾸거나 분기 자체를 건너뛰면 끝이다. 그리고 서버 입장에서 4번의 요청은 판정을 통과한 기기에서 온 것인지 알 방법이 없다.

![play_integrity_2.png](/assets/images/play_integrity_2.png?raw=true)

올바른 방법은 **판정을 실제 요청 처리와 묶는 것**이다.

- 토큰을 별도 API로 보내지 말고, **보호하려는 요청에 함께** 실어 보낸다
- 서버는 그 요청을 처리하기 전에 토큰을 검사하고, 통과했을 때만 요청을 처리한다
- `requestHash`로 토큰과 요청 내용을 묶어, 토큰을 떼어다 다른 요청에 쓰지 못하게 한다

이렇게 하면 앱 안의 코드를 아무리 바꿔도, 서버가 요청을 처리해 주지 않는 한 공격자가 얻는 것이 없다.

같은 이유로 토큰 복호화도 앱에서 하면 안 된다. 공식 문서는 Google 서버에서 복호화하는 방법을 권장하고, 직접 복호화하려면 키를 Play Console에서 받아 **자체 보안 서버 환경에서** 처리하도록 안내한다. 키가 앱에 들어가는 순간 판정을 믿을 근거가 사라진다.

## 한계와 운영 주의점

### 1. 기준을 너무 높이면 정상 사용자가 막힌다

신뢰 등급이 높은 라벨일수록 그 조건을 만족하는 기기 수는 적다. 예를 들어 `MEETS_STRONG_INTEGRITY`는 Android 13 이상에서 최근 1년 이내 보안 패치를 요구하므로, 업데이트가 끊긴 오래된 기기는 정상 기기여도 받지 못한다. 공식 문서도 높은 등급 하나에만 의존하지 말고 **등급별로 단계적인 대응**을 하라고 권한다.

![play_integrity_3.png](/assets/images/play_integrity_3.png?raw=true)

공식 문서가 예로 드는 대응 단계는 **허용 / 제한된 허용 / 추가 인증 후 제한된 허용 / 차단**이다. 서비스의 위험도에 맞춰 어떤 판정 조합을 어디에 둘지 정하면 된다.

또 `UNEVALUATED`는 공격이 아니어도 나올 수 있다(Google Play 미로그인 등). 무조건 차단으로 처리하면 정상 사용자가 막힐 수 있다. 문제를 사용자가 직접 해결할 수 있는 경우를 위해 `GET_INTEGRITY` 같은 **복구 대화상자**(remediation dialog)도 제공되니 차단 전에 고려할 만하다.

### 2. Google Play 생태계에 의존한다

판정은 Google Play와 Google Play 서비스가 있어야 나온다. Google Play가 아닌 경로로 배포한 앱은 `UNLICENSED`를, Google Play 기록과 서명이 다른 빌드는 `UNRECOGNIZED_VERSION`을 받는다. 다른 스토어로도 배포하거나 Google Play 서비스가 없는 기기를 지원해야 한다면, 그 경우의 정책을 따로 정해야 한다.

### 3. 호출 한도와 지연이 있다

- 기본 한도는 앱 전체 설치 기준 **하루 10,000건**이다. 앱이 포그라운드로 올 때마다 부르지 말고 민감한 동작 직전에만 요청한다. 판정을 캐시해 재사용하면 재사용 공격 위험이 커진다
- 네트워크 등 일시적인 이유로 정상 기기에서도 실패할 수 있으므로, 지수 백오프로 재시도할 수 있게 만든다

### 4. 유일한 방어 수단이 아니다

공식 문서는 Play Integrity API를 **유일한 악용 방지 수단이 아니라** 전체 전략의 한 신호로 쓰라고 명시한다. 요청 빈도, 계정의 이상 행동 같은 서버 쪽 신호와 함께 판단해야 한다. 처음 도입할 때는 바로 차단하지 말고, 판정 결과를 먼저 수집해 실제 사용자 분포를 확인한 뒤 기준을 정하는 것이 안전하다.

## 정리

| 항목 | 내용 |
| --- | --- |
| 목적 | 요청이 변조되지 않은 앱에서, 정품 인증 기기로부터 왔는지 서버가 확인 |
| 판정 | 앱 무결성(`PLAY_RECOGNIZED`), 기기 무결성(`MEETS_DEVICE_INTEGRITY` 등), 계정(`LICENSED`) + 선택 판정 |
| 흐름 | 앱이 토큰을 받아 실제 요청과 함께 전달 → 서버가 Google 서버로 복호화 → 서버가 판정 |
| 요청 결합 | 표준은 `requestHash`, 클래식은 `nonce`. 토큰을 다른 요청에 재사용하지 못하게 한다 |
| 핵심 원칙 | 판정 결과로 앱이 분기하지 않는다. 서버가 판정과 요청 처리를 한 번에 한다 |
| 한계 | ① 높은 기준은 정상 사용자 차단 ② Google Play 의존 ③ 하루 호출 한도·지연 ④ 단독 방어 수단이 아님 |

<aside markdown="1">
💡 포인트

토큰은 위조할 수 없지만, 판정 결과를 앱에 돌려주고 앱이 분기하면 그 분기는 후킹으로 바뀐다. 판정과 요청 처리는 서버에서 한 번에 하자.

`requestHash`(또는 `nonce`)를 빼먹으면 토큰이 요청에 묶이지 않는다. 서버에서 반드시 다시 계산해 비교하자.

기기 판정 기준은 `MEETS_DEVICE_INTEGRITY`부터 시작하고, 차단 전에 판정 분포를 먼저 수집해 보자.

Play Integrity도 하나의 신호일 뿐이다. 다른 서버 쪽 신호와 함께 쓰자.

</aside>

## 참조

[Play Integrity API overview](https://developer.android.com/google/play/integrity/overview)

[Integrity verdicts](https://developer.android.com/google/play/integrity/verdicts)

[Make a standard API request](https://developer.android.com/google/play/integrity/standard)

[Make a classic API request](https://developer.android.com/google/play/integrity/classic)

[Remediation dialogs](https://developer.android.com/google/play/integrity/remediation)

[SafetyNet deprecation timeline](https://developer.android.com/privacy-and-security/safetynet/deprecation-timeline)
