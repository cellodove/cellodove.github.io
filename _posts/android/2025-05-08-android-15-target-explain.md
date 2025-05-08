---
title : "[Android] Android 15(API 35) 이상을 타겟팅하는 앱 주요 동작 변경사항"
excerpt: "Android 15(API 35) 이상을 타겟팅하는 앱 주요 동작 변경사항"

categories :
  - Android

toc: true
toc_sticky: true
last_modified_at: 2025-05-08
---

![android15_logo.svg](/assets/images/android15_logo.svg?raw=true)

## 포그라운드 서비스 변경사항

- **`dataSync` 및 `mediaProcessing` 서비스에 24시간당 6시간 실행 제한**
    - `Service.onTimeout(int, int)` 콜백이 호출되며, 몇 초 내에 `stopSelf()`를 호출하지 않으면 `RemoteServiceException`이 발생한다.
    - 제한 시간은 앱의 모든 해당 서비스 타입 간에 공유되며, 사용자가 앱을 포그라운드로 다시 가져오면 타이머가 리셋된다.
- **`BOOT_COMPLETED` 리시버로 시작 가능한 FGS 유형 제한**
    - `dataSync`, `camera`, `mediaPlayback`, `phoneCall`, `mediaProjection`, (`microphone`) 유형은 부팅 완료 브로드캐스트에서 실행 불가하며, 시도 시 `ForegroundServiceStartNotAllowedException`이 발생한다.
- **`SYSTEM_ALERT_WINDOW` 권한 예외 축소**
    - 이전에는 이 권한만으로 백그라운드에서 FGS 시작이 가능했으나, Android 15 타겟 앱은 **실제 오버레이 창이 표시 중**일 때만 허용된다.

## 방해 금지(DND) 모드 변경

**전역 DND 설정 변경 금지**

- `setInterruptionFilter()`나 `setNotificationPolicy()`를 직접 호출해 전역 상태를 바꿀 수 없고, 대신 `AutomaticZenRule`을 등록·관리해야 한다.
- 기존 API 호출 시 자동으로 암시적 `AutomaticZenRule`이 생성·갱신되며, 시스템은 가장 제한적인 규칙을 우선 적용한다.

## OpenJDK API 호환성 강화

- **`String.format()`·`Formatter.format()` 검사 엄격화**: 인덱스·플래그·너비·정밀도 유효성 오류 시 예외 발생
- **`Arrays.asList(...).toArray()` 반환 타입 변경**: 기본 `Object[]`가 돼 `ClassCastException` 주의 → `toArray(new T[0])` 사용 권장
- **히브리어(`iw`→`he`), 이디시어(`ji`→`yi`), 인도네시아어(`in`→`id`) 등 언어 코드 표준화**
- **`Random.ints()` 시퀀스 변경**: `nextInt()`와 다른 난수 흐름
- **`List.removeFirst()`·`removeLast()` 충돌**: Android 15에서 신규 메서드로 도입돼 Kotlin 확장 함수와 충돌 가능 → `removeAt(0)`·`removeAt(lastIndex)`로 대체

## 보안 강화

- **제한된 TLS**: TLS 1.0·1.1 허용 중단, Android 15 타겟 앱은 사용 불가
    - TLS는 TCP/IP 네트워크에서 통신 보안을 제공하기 위해 설계된 암호 프로토콜
- **백그라운드 활동 실행 제한**
    - 작업 스택 최상위 UID와 불일치하는 앱의 활동 실행 차단(`allowCrossUidActivitySwitchFromBelow="false"`)
        - UID는 고유식별자 AndroidManifest에 있는 Package name으로 할당
    - `PendingIntent` 생성 시 발신자만 활동 시작 허용 등 추가 제약
- **더 안전한 인텐트 검증**
    - 대상 `IntentFilter`와 정확히 매칭, `action` 필수화
    - `PendingIntent` 내부 `Intent`에도 동일 정책 적용
    - `StrictMode.detectUnsafeIntentLaunch()`로 위반 로깅 가능

## UI / 시스템 동작 변경

- **전체 화면(Edge-to-edge) 기본 적용**
    - 타겟 API 35 앱은 자동으로 풀스크린 모드가 되며, 상태·네비게이션 바가 투명화되고 오버레이 뒤로 콘텐츠가 그려진다.
    - `WindowInsets`나 Compose Material 3 `Scaffold` 등으로 안전하게 인셋 처리 필요

![edge_to_edge.gif](/assets/images/edge_to_edge.gif?raw=true)

- **지원 중단 API 정리**
    - `setStatusBarColor()`, `setNavigationBarColor()`, 관련 `R.attr` 등 다수 폐기 예정
- **`TextView.elegantTextHeight` 기본값 `true`**
    - 수직 메트릭이 큰 스크립트를 위한 가독성 향상 폰트 사용
- **복잡 스크립트 문자 너비 할당**
    - 필기체·동아시아 문자의 오버행(overhang) 잘림 방지용 추가 너비·패딩 제공 (`useBoundsForWidth`, `shiftDrawingOffsetForStartOverhang`)
- **`EditText` 최소 행 높이**
    - 로케일별 기본 폰트 메트릭에 맞춘 최소 줄 높이 예약
    - 이전 동작 복원 시 `useLocalePreferredLineHeightForMinimum="false"`, `setMinimumFontMetrics()` API 사용

## 카메라·미디어

**오디오 포커스 요청 제약**

- Android 15 타겟 앱은 **포그라운드**에 있거나 **포그라운드 서비스** 실행 중일 때만 `requestAudioFocus()` 호출 가능.
- 조건 미충족 시 `AUDIOFOCUS_REQUEST_FAILED` 반환

## 비 SDK 인터페이스 제한 업데이트

- Android 15 기반으로 **비공개(Private) API** 목록 갱신
- 타겟 API 수준에 따라 접근 가능 여부가 달라짐
- 비 호환 인터페이스 사용 시 즉시 앱 중단 위험 → 대체 공개 API 사용 검토 및 필요 시 신규 API 요청

## 참조

[동작 변경사항 Android 15 이상을 타겟팅하는 앱](https://developer.android.com/about/versions/15/behavior-changes-15?hl=ko)

[데이터 레이어 Android Developers](https://developer.android.com/topic/architecture/data-layer?hl=ko)
