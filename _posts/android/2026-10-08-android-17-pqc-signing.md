---
title : "[Android] Android 17(API 37) 양자 내성 서명 키로 앱 서명 검증이 깨지는 문제"
excerpt: "Android 17 APK 서명 v3.2 양자 내성 키와 앱 서명 검증 대응 방법"

categories :
  - Android

toc: true
toc_sticky: true
last_modified_at: 2026-10-08
---

![certification_image.jpg](/assets/images/certification_image.jpg?raw=true)

## 개요

Android 17(API 37)에는 **APK Signature Scheme v3.2 (양자 내성 하이브리드 서명)** 이 새로 들어왔다.

Google Play는 앱에 양자 내성(ML-DSA) 키 서명을 함께 붙여 배포할 수 있고, Android 17은 이 키를 **앱의 현재 서명 키**로 취급한다.

이 때문에 앱이 자기 서명 인증서의 해시를 구해 정상 여부를 판단하는 **앱 위·변조 검사**는 Android 17에서 기존과 다른 해시를 얻게 되고, 정상 앱을 위·변조된 앱으로 오판할 수 있다.

이 글에서는 v3.2가 서명 인증서를 어떻게 바꾸는지, 왜 Android 17 + Play 설치본에서만 영향을 받는지, 어떻게 대응하면 되는지를 정리한다.

## 앱 서명 검증(위·변조 검사)이란

앱이 리패키징(디컴파일 후 수정해서 다른 키로 재서명)되지 않았는지 확인하기 위해, 앱이 **자기 자신의 서명 인증서 해시**를 구해서 미리 등록해 둔 정상 해시와 비교하는 방식을 많이 쓴다.

- 정상 앱 → 등록된 키로 서명되어 있음 → 해시 일치 → 통과
- 리패키징된 앱 → 공격자의 키로 재서명됨 → 해시 불일치 → 차단

흔히 쓰는 코드는 이렇다.

```kotlin
val packageInfo = packageManager.getPackageInfo(
    packageName,
    PackageManager.GET_SIGNING_CERTIFICATES
)
// 현재 서명 인증서 "하나"만 꺼내서 해시
val signer = packageInfo.signingInfo?.apkContentsSigners?.firstOrNull()
val hash = MessageDigest.getInstance("SHA-256")
    .digest(signer!!.toByteArray())
    .joinToString("") { "%02X".format(it) }

// hash 를 미리 등록해 둔 정상 해시와 비교
```

문제는 `apkContentsSigners` 가 **"현재" 서명 인증서**를 돌려준다는 점이다. 이 "현재"가 OS 버전에 따라 달라질 수 있다.

## APK 서명 방식의 변화

| 스킴 | 도입 | 특징 |
| --- | --- | --- |
| v1 (JAR 서명) | 초기 | ZIP 안의 개별 파일 서명 |
| v2 | Android 7.0 | APK 전체를 하나의 블록으로 서명. 설치 속도·무결성 향상 |
| v3 | Android 9 (API 28) | **키 교체(Key Rotation)** 지원. 서명 이력(lineage)을 APK에 담는다 |
| v3.1 | Android 13 (API 33) | 교체된 키를 특정 API 레벨 이상에만 적용하도록 분리 |
| **v3.2** | **Android 17 (API 37)** | **고전 키(RSA/EC) + 양자 내성 키(ML-DSA) 하이브리드 서명** |

## 키 교체와 서명 이력(lineage)

v3부터는 앱 서명 키를 바꿀 수 있다. 키를 바꾸면 "이전 키가 새 키를 보증한다"는 체인이 APK에 함께 들어가는데, 이것을 **서명 이력(lineage)** 이라고 한다.

```
원래 키  →  새 키  →  더 새로운 키(현재)
```

플랫폼은 이 체인을 검증하기 때문에, 업데이트 APK가 새 키로 서명되어 있어도 같은 앱으로 인정하고 설치해준다.

이때 `apkContentsSigners` 는 **체인의 마지막(현재) 키**를 돌려주고, 전체 체인은 `signingCertificateHistory` 로 볼 수 있다.

## v3.2 하이브리드 서명이 바꾸는 것

Android 17은 양자 컴퓨터 시대에 대비해 **ML-DSA(양자 내성 서명 알고리즘)** 를 앱 서명에 도입했다.

- APK에 **고전 키 서명 + ML-DSA 서명**을 한 블록(v3.2 블록)에 함께 넣는다
- Android 17은 이 하이브리드 블록을 **암묵적인 키 교체**로 취급한다
    - 새 고전 키는 lineage의 끝에서 두 번째로 추가되고
    - **새 ML-DSA 키가 앱의 "현재 서명 신원"이 된다**
- Android 16 이하는 v3.2 블록을 **무시**하고 기존 v3.0/v3.1 블록(고전 키)으로만 검증한다

그리고 Google Play 앱 서명(Play App Signing)을 쓰는 앱은 Play Console에서 **양자 내성 키 업그레이드**를 하면 Play가 이 하이브리드 서명을 자동으로 붙여서 배포한다.

즉 lineage는 이렇게 된다.

```
원래 앱 서명 키  →  새 고전 키  →  양자 내성(ML-DSA) 키 ← Android 17의 현재 서명
   ↑
 Android 16 이하의 현재 서명
```

## 왜 Android 17 + Play 설치본에서만 영향을 받는가

| 기기 | 설치 경로 | apkContentsSigners 결과 | 기존 해시와 비교 |
| --- | --- | --- | --- |
| Android 16 | Google Play | 원래 앱 서명 키 (v3.2 블록 무시) | 일치 |
| Android 17 에뮬레이터 | 로컬 빌드 | 업로드 키 (Play를 거치지 않아 하이브리드 서명 없음) | 일치 (업로드 키 기준) |
| Android 17 실기기 | Google Play | **양자 내성 키** | **불일치** |

두 조건이 **동시에** 맞아야 해시가 달라진다.

- **Android 17 이상**이어야 v3.2 블록을 읽는다
- **Play에서 설치**해야 하이브리드 서명이 붙어 있다. 로컬 빌드는 업로드 키로 서명되므로 로컬 테스트로는 확인할 수 없다

## 대응 방법

### 1. 허용 해시 목록에 양자 내성 키 해시 추가

Play Console의 앱 서명 화면에서 **양자 내성 암호화 키의 SHA-256**을 확인해 정상 해시 목록에 추가하는 방법이다. 가장 빠르게 적용할 수 있다.

다만 이 방법은 **다음에 키가 또 바뀌면 똑같이 깨진다.** Google은 앞으로 최소 2년마다 서명 키 업그레이드를 권장할 예정이라고 밝혔다.

### 2. 서명 이력(lineage) 전체로 판정 (근본 해결)

"현재 키 하나"가 아니라 **lineage에 포함된 키 중 하나라도 등록돼 있으면 통과**시키는 방식이다.

lineage는 플랫폼이 검증한 체인이라 공격자가 임의로 끼워 넣을 수 없다. 그래서 검사가 약해지지 않으면서 이후 키 교체에도 깨지지 않는다.

```kotlin
fun getSigningCertHashes(context: Context): List<String> {
    val pm = context.packageManager
    val packageInfo = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        pm.getPackageInfo(
            context.packageName,
            PackageManager.PackageInfoFlags.of(PackageManager.GET_SIGNING_CERTIFICATES.toLong())
        )
    } else {
        pm.getPackageInfo(context.packageName, PackageManager.GET_SIGNING_CERTIFICATES)
    }
    val signingInfo = packageInfo.signingInfo ?: return emptyList()

    val certs = if (signingInfo.hasMultipleSigners()) {
        // 다중 서명자 APK 는 lineage 가 없다 → 현재 서명자 전부
        signingInfo.apkContentsSigners
    } else {
        // 서명 이력 전체 (가장 오래된 키 → 현재 키)
        signingInfo.signingCertificateHistory
    }

    return certs.map { cert ->
        MessageDigest.getInstance("SHA-256")
            .digest(cert.toByteArray())
            .joinToString("") { "%02X".format(it) }
    }
}
```

구한 해시 목록 중 하나라도 정상 해시 목록에 있으면 통과시키면 된다.

더 간단하게는 `PackageManager.hasSigningCertificate()` 를 쓸 수 있다. 이 API는 과거 서명 인증서(lineage)까지 포함해서 해당 인증서로 서명된 적이 있는지 확인해준다.

```kotlin
val registered = pm.hasSigningCertificate(
    context.packageName,
    expectedSha256Bytes,
    PackageManager.CERT_INPUT_SHA256
)
```

### 3. 검사 실패 시 사유와 해시를 로그로 남기기

위·변조 검사가 실패했을 때 **실패 사유와 실제 서명 해시**를 로그로 남겨두면, 키 변경으로 인한 오판인지 실제 리패키징인지 빠르게 구분할 수 있다.

루팅 탐지 등 다른 보안 검사와 안내 문구를 공유한다면 사용자에게는 하나로 보여주더라도 로그에서는 구분해두는 것이 좋다. 인증서 지문은 공개된 값이라 민감 정보가 아니다.

## 정리

| 항목 | 내용 |
| --- | --- |
| 핵심 | Android 17의 APK Signature Scheme v3.2가 양자 내성 키를 앱의 현재 서명 신원으로 삼음 |
| 영향 조건 | Android 17 이상 + Play 앱 서명에서 양자 내성 키 업그레이드된 앱 + 앱이 apkContentsSigners 하나로 자기 서명 검사 |
| 로컬에서 확인이 어려운 이유 | 로컬 빌드는 업로드 키로 서명되고, Android 16 이하는 v3.2 블록을 무시함 |
| 빠른 대응 | 허용 해시 목록에 양자 내성 키 SHA-256 추가 |
| 근본 대응 | signingCertificateHistory(lineage) 전체로 판정 |

<aside>
💡 포인트

자기 서명을 검사하는 코드는 apkContentsSigners 한 개가 아니라 lineage 전체를 봐야 한다. 키 교체(v3/v3.1)와 하이브리드 서명(v3.2) 모두 "현재 서명"을 바꾼다.

서명과 관련된 검사는 Play에서 서명된 APK로 테스트해야 한다. 로컬 빌드로는 아무것도 검증되지 않는다. 새 OS가 나오면 내부 테스트 트랙 + 실기기로 확인하자.

</aside>

## 참조

[APK Signature Scheme v3.2](https://source.android.com/docs/security/features/apksigning/v3-2)

[Android 17 Features](https://developer.android.com/about/versions/17/features)

[Post-quantum cryptography in Android](https://blog.google/security/security-for-the-quantum-era-implementing-post-quantum-cryptography-in-android/)

[SigningInfo](https://developer.android.com/reference/android/content/pm/SigningInfo)

[APK Signature Scheme v3](https://source.android.com/docs/security/features/apksigning/v3)
