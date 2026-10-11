---
title : "디자인 시스템(Design System)"
excerpt: "디자인 시스템의 이해, 구성, 아토믹 디자인에 대하여"

categories :
  - DesignSystem

toc: true
toc_sticky: true
last_modified_at: 2026-10-11
---

![design_system_image1.jpg](/assets/images/design_system_image1.jpg?raw=true)

## 디자인 시스템의 이해

### 디자인 시스템 구축 배경

예전에는 하나의 서비스가 웹사이트 하나, 앱 하나 정도로 단순했다.

하지만 점점 하나의 회사가 여러 제품을 Web, Android, iOS, 태블릿, 워치 등 여러 플랫폼에서 동시에 운영하게 되었다. 제품이 많아지면서 이를 만드는 팀도 여러 개로 나뉘었고, 각 팀은 빠른 출시를 위해 각자 화면을 디자인하고 개발했다.

그 결과 같은 회사의 서비스인데도 버튼 모양, 색상, 간격, 문구, 동작 방식이 조금씩 달라지기 시작했다. 처음에는 작은 차이였지만 제품과 화면이 늘어날수록 이런 차이는 계속 쌓였고, 나중에는 어떤 것이 기준인지조차 알기 어려워졌다.

그렇다면 여러 제품과 플랫폼에서 어떻게 같은 경험을 줄 수 있을까?

### 디자인 시스템 정의

이를 위해 나타난 방식이 디자인 시스템(Design System)이다.

디자인 시스템은 다양한 디지털 서비스와 제품에서 일관된 사용자 경험을 유지하기 위한 방법으로, 재사용 가능한 컴포넌트와 이를 사용하기 위한 원칙, 가이드의 집합이다.

여기서 중요한 점은 디자인 시스템이 단순히 버튼, 입력창 같은 UI 컴포넌트를 모아둔 것이 아니라는 것이다. 컴포넌트를 언제, 어떻게, 왜 사용해야 하는지에 대한 규칙까지 포함한다.

대표적인 예로 2014년 Google이 발표한 Material Design이 있다. Android, Web, iOS 등 Google의 여러 제품이 하나의 디자인 언어를 사용할 수 있도록 만든 디자인 시스템이다.

### 디자인 시스템의 필요성

디자인 시스템이 없다면 각자의 입장에서 다음과 같은 문제가 생긴다.

- 사용자

같은 서비스인데 화면마다 버튼 위치나 동작이 달라 매번 새로 익혀야 한다. 이런 어색함은 서비스에 대한 신뢰도와도 연결된다.

- 디자이너

비슷한 화면을 매번 새로 그리고, 색상이나 간격을 정하는 일을 화면마다 반복해야 한다.

- 개발자

같은 버튼을 화면마다 여러 번 구현하고, 색상값이나 크기가 코드 곳곳에 하드코딩된다. 디자인이 바뀌면 관련된 코드를 전부 찾아서 수정해야 한다.

- 협업

디자이너와 개발자가 같은 화면을 보고도 다르게 해석해 "이 파란색이 그 파란색 맞나요?" 같은 확인 작업이 계속 생긴다.

즉 서비스의 규모가 커질수록 이런 비용은 함께 커지기 때문에, 이를 하나의 기준으로 관리할 수 있는 체계가 필요하다.

### 디자인 시스템의 장점

디자인 시스템을 사용하면 다음과 같은 이점을 얻을 수 있다.

- 일관성

화면, 플랫폼, 팀이 달라도 같은 토큰과 컴포넌트를 사용하기 때문에 어디서나 같은 사용자 경험을 줄 수 있다.

- 생산성

비슷한 UI를 화면마다 새로 디자인하고 구현할 필요 없이, 만들어 둔 컴포넌트를 조합해서 빠르게 개발할 수 있다.

- 협업

디자이너와 개발자가 같은 이름(토큰, 컴포넌트명)으로 소통하기 때문에 해석 차이가 줄어든다.

- 유지보수

브랜드 색상 하나를 바꾸기 위해 수십, 수백 곳을 수정할 필요 없이 토큰 한 곳만 바꾸면 전체에 반영된다.

- 확장

새로운 제품, 다크모드, 새로운 플랫폼이 추가되어도 처음부터 다시 정의하지 않고 기존 시스템 위에서 확장할 수 있다.

개발자 입장에서 보면 디자인 시스템은 하드코딩된 값을 상수로 빼고, 반복되는 코드를 공통 함수로 묶는 것과 같은 원리다. 디자인 토큰은 상수, 컴포넌트는 공통 함수나 라이브러리, 문서화는 API 문서라고 생각하면 쉽다.

## 디자인 시스템의 구성

### 디자인 시스템의 구조

디자인 시스템은 보통 개요, 파운데이션, 컴포넌트, 패턴, 기타로 구조화된다.

파운데이션에서 정한 기초 위에 컴포넌트가 만들어지고, 컴포넌트를 조합해 패턴이 만들어지는 것처럼 작은 단위부터 큰 단위로 쌓아 올리는 구조이다.

Google의 Material Design 3를 예로 각 구조를 살펴보자.

- 개요(Overview)

디자인 시스템을 소개하는 부분이다. 디자인 시스템을 왜 만들었는지, 어떤 디자인 원칙을 따르는지, 어떻게 시작하면 되는지를 설명한다.

![m3_overview_light.png](/assets/images/m3_overview_light.png?raw=true)

출처: [Material Design 3 - Get started](https://m3.material.io/get-started)

Material 3의 Get started 페이지는 Material이 무엇인지, 어떻게 사용하는지, 디자인과 개발을 어디서부터 시작하면 되는지를 안내한다. 왼쪽 메뉴를 보면 디자인 시스템 전체가 Get started, Develop, Foundations, Styles, Components로 나뉘어 있는 것을 볼 수 있다.

- 파운데이션(Foundation)

모든 UI의 기초가 되는 시각적 요소이다. 색상, 타이포그래피, 간격, 그리드, 아이콘, 모서리 둥글기, 그림자 등을 정의한다.

![m3_foundation_styles_light.png](/assets/images/m3_foundation_styles_light.png?raw=true)

출처: [Material Design 3 - Styles](https://m3.material.io/styles)

Material 3에서는 파운데이션에 해당하는 내용을 Styles 메뉴에서 다룬다. Color, Elevation, Icons, Motion, Shape, Typography처럼 UI의 시각적인 기초가 되는 요소들이다.

- 컴포넌트(Component)

파운데이션을 바탕으로 만든 재사용 가능한 UI 요소이다. 버튼, 입력창, 체크박스, 카드 등이 있다.

![m3_component_buttons_light.png](/assets/images/m3_component_buttons_light.png?raw=true)

출처: [Material Design 3 - Buttons](https://m3.material.io/components/buttons/overview)

Material 3의 버튼 컴포넌트 페이지이다. Overview, Specs, Guidelines, Accessibility 탭으로 나뉘어 있고, Elevated, Filled, Tonal, Outlined, Text 다섯 가지 변형을 함께 정의하고 있다.

- 패턴(Pattern)

컴포넌트를 조합해 자주 반복되는 문제를 해결하는 방법이다. 로그인 폼, 검색, 빈 화면, 오류 안내 등이 있다.

![m3_pattern_canonical_layouts_light.png](/assets/images/m3_pattern_canonical_layouts_light.png?raw=true)

출처: [Material Design 3 - Canonical layout examples](https://m3.material.io/foundations/layout/canonical-layouts/overview)

Material 3에는 패턴 메뉴가 따로 없고, 자주 쓰는 화면 구성을 Canonical layout examples로 제공한다. Feed, List-detail, Supporting pane처럼 여러 컴포넌트를 조합한 레이아웃 패턴이다.

- 기타

위 분류에 속하지 않지만 디자인 시스템을 사용하는 데 필요한 내용이다. 접근성, 리소스(디자인 파일, 코드 저장소), 기여 방법, 변경 이력 등이 여기에 해당한다.

![m3_etc_foundations_light.png](/assets/images/m3_etc_foundations_light.png?raw=true)

출처: [Material Design 3 - Foundations](https://m3.material.io/foundations)

Material 3의 Foundations 메뉴에는 접근성(Accessibility), 콘텐츠 디자인(Content design), 디자인 토큰, 용어 사전(Material A-Z)처럼 디자인 시스템을 사용하는 데 필요한 내용이 모여 있다. 이름은 Foundations이지만 책의 분류로 보면 기타에 가까운 내용이 많다.

이 구조를 역할에 따라 묶으면 스타일 가이드, 패턴 라이브러리, 시스템 가이드 세 가지로 나눌 수 있다.

| 구성 요소 | 해당하는 구조 | 역할 |
| --- | --- | --- |
| **스타일 가이드** | 파운데이션 | 어떤 모습이어야 하는가 |
| **패턴 라이브러리** | 컴포넌트, 패턴 | 무엇을 만들어 쓰는가 |
| **시스템 가이드** | 개요, 기타 | 어떻게 사용하고 운영하는가 |

레고로 비유하면 스타일 가이드는 블록의 색과 크기 규격, 패턴 라이브러리는 블록과 자주 쓰는 조립 묶음, 시스템 가이드는 조립 설명서라고 생각하면 쉽다.

### 스타일 가이드

스타일 가이드는 서비스가 어떤 모습이어야 하는지를 정의한 시각적 규칙이다. 브랜드의 정체성을 화면에 일관되게 표현하기 위한 가장 기초가 되는 요소이며, 구조로 보면 파운데이션에 해당한다.

스타일 가이드에는 다음과 같은 내용이 포함된다.

- 디자인 원칙(Design Principles)

디자인을 결정할 때 기준이 되는 가치이다. 색상, 글꼴, 간격 같은 시각적 규칙도 결국 이 원칙에서 출발한다. 예를 들어 "단순하게"라는 원칙이 있다면 색상 수를 줄이고 여백을 넉넉하게 쓰는 방향으로 스타일이 정해진다.

![principle_salesforce.png](/assets/images/principle_salesforce.png?raw=true)

출처: [Salesforce Lightning Design System - Patterns](https://www.lightningdesignsystem.com/2e1ef8501/p/355656-patterns)

Salesforce는 명확성(Clarity), 효율성(Efficiency), 일관성(Consistency), 아름다움(Beauty) 네 가지를 디자인을 결정할 때의 기준으로 삼는다. 예를 들어 일관성은 "같은 문제에는 같은 해결책을 적용한다"는 원칙으로, 사용자가 한 번 익힌 방식을 다른 화면에서도 그대로 쓸 수 있게 한다.

- 색상(Color)

브랜드 대표 색상(Primary), 보조 색상(Secondary), 배경, 텍스트, 오류나 성공 같은 상태 색상을 정의한다.

![color_atlassian.png](/assets/images/color_atlassian.png?raw=true)

출처: [Atlassian Design System - Color](https://atlassian.design/foundations/color)

Atlassian은 파랑, 초록, 노랑, 빨강 등 색상별로 밝기 단계를 나눈 팔레트를 정의하고, 이를 디자인 토큰으로 사용한다.

- 타이포그래피(Typography)

글꼴, 글자 크기, 굵기, 줄 간격을 제목, 본문, 캡션 등 용도별로 정의한다.

![type_apple.png](/assets/images/type_apple.png?raw=true)

출처: [Apple Human Interface Guidelines - Typography](https://developer.apple.com/design/human-interface-guidelines/typography)

Apple은 Human Interface Guidelines에서 가독성 확보, 정보 계층 표현, 시스템 폰트 사용, Dynamic Type(사용자가 설정한 글자 크기) 지원 같은 타이포그래피 기준을 정의한다.

- 간격과 그리드(Spacing, Grid)

요소 사이의 여백과 화면 배치 기준을 4, 8, 16처럼 일정한 단위로 정의한다.

![spacing_carbon.png](/assets/images/spacing_carbon.png?raw=true)

출처: [IBM Carbon Design System - Spacing](https://carbondesignsystem.com/elements/spacing/overview/)

IBM의 Carbon은 2px부터 160px까지의 간격을 $spacing-01부터 $spacing-13까지의 토큰으로 정의해두고 모든 컴포넌트에서 이 값만 사용한다.

![grid_carbon.png](/assets/images/grid_carbon.png?raw=true)

출처: [IBM Carbon Design System - 2x Grid](https://carbondesignsystem.com/elements/2x-grid/overview/)

그리드는 화면을 두 배수로 나누는 2x Grid를 사용한다. 하나의 영역을 2칸, 4칸, 8칸, 16칸으로 나눠 레이아웃을 잡는다.

- 아이콘과 이미지

아이콘의 스타일, 크기, 선 굵기와 이미지 사용 기준을 정의한다.

![icon_atlassian.png](/assets/images/icon_atlassian.png?raw=true)

출처: [Atlassian Design System - Iconography](https://atlassian.design/foundations/iconography)

Atlassian은 아이콘을 1.5px 선 굵기와 둥근 모서리 스타일로 통일하고, "이미 있는 아이콘을 재사용한다"처럼 사용 기준을 Do / Don't 형태로 정리했다.

- 보이스 앤 톤(Voice & Tone)

버튼 문구, 안내 문구, 오류 메시지 등을 어떤 말투로 쓸지 정의한다.

![voice_mailchimp.png](/assets/images/voice_mailchimp.png?raw=true)

출처: [Mailchimp Content Style Guide - Voice and Tone](https://styleguide.mailchimp.com/voice-and-tone/)

Mailchimp는 목소리(Voice)는 항상 같지만 상황과 상대에 따라 어조(Tone)는 바뀐다고 설명한다. 그리고 쉬운 말로 분명하게(plainspoken), 진정성 있게(genuine)처럼 글을 쓸 때 지킬 Voice 원칙을 정리해두었다.

이렇게 정한 값에 이름을 붙여 관리하는 것을 디자인 토큰(Design Token)이라고 한다. 디자인 토큰은 값 자체를 나타내는 원시 토큰(Primitive Token)과, 그 값이 어디에 쓰이는지를 나타내는 의미 토큰(Semantic Token)으로 나눌 수 있다.

```kotlin
// 원시 토큰: 값 자체
val Blue500 = Color(0xFF2F6FED)
val Gray900 = Color(0xFF1B1D1F)

// 의미 토큰: 어디에 쓰이는지
val Primary = Blue500
val TextPrimary = Gray900
```

화면에서는 의미 토큰만 사용한다. 이렇게 하면 다크모드나 브랜드 색상이 바뀌어도 의미 토큰이 가리키는 값만 바꾸면 된다.

### 패턴 라이브러리

패턴 라이브러리는 스타일 가이드를 바탕으로 만든 재사용 가능한 UI 요소의 모음이다. 컴포넌트 라이브러리라고도 부르며, 구조로 보면 컴포넌트와 패턴에 해당한다.

패턴 라이브러리는 작은 단위와 큰 단위로 나눌 수 있다.

- 컴포넌트(Component)

버튼, 입력창, 체크박스, 카드처럼 화면을 구성하는 가장 기본적인 UI 요소이다. 각 컴포넌트는 기본, 눌림, 비활성화, 오류 같은 상태(State)와 크기, 종류 같은 변형(Variant)을 함께 정의한다.

![component_carbon_button.png](/assets/images/component_carbon_button.png?raw=true)

출처: [IBM Carbon Design System - Button](https://carbondesignsystem.com/components/button/usage/)

IBM Carbon은 버튼을 Primary, Tertiary, Ghost, Icon 버튼처럼 변형별로 나누고, 각 버튼이 라벨(A), 컨테이너(B), 아이콘(C)으로 이루어져 있다고 구성도(Anatomy)로 정의한다.

- 패턴(Pattern)

컴포넌트를 조합해 자주 반복되는 문제를 해결하는 방법이다. 로그인 폼, 검색 화면, 데이터가 없을 때의 빈 화면, 오류 안내처럼 여러 화면에서 반복되는 구성이 여기에 해당한다.

![pattern_carbon_forms.png](/assets/images/pattern_carbon_forms.png?raw=true)

출처: [IBM Carbon Design System - Forms](https://carbondesignsystem.com/patterns/forms-pattern/)

Carbon의 폼(Forms) 패턴이다. 라벨, 텍스트 입력창, 드롭다운, 도움말, 버튼 같은 컴포넌트를 어떤 순서와 규칙으로 조합해 하나의 입력 화면을 만드는지 정리했다.

![pattern_carbon_empty_states.png](/assets/images/pattern_carbon_empty_states.png?raw=true)

출처: [IBM Carbon Design System - Empty states](https://carbondesignsystem.com/patterns/empty-states-pattern/)

데이터가 없을 때 보여주는 빈 화면(Empty state) 패턴이다. 이미지, 제목, 설명, 액션 버튼, 텍스트 링크를 어떻게 배치할지 정해두어 어느 화면에서나 같은 형태의 빈 화면을 보여줄 수 있다.

패턴 라이브러리는 디자인 도구(Figma 등)의 컴포넌트와 실제 코드의 컴포넌트가 함께 존재하는 것이 중요하다. 디자이너가 Figma에서 사용하는 버튼과 개발자가 코드에서 사용하는 버튼이 같은 이름, 같은 상태, 같은 변형을 가져야 디자인과 구현이 어긋나지 않는다.

```kotlin
@Composable
fun PrimaryButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
    enabled: Boolean = true,
) {
    Button(
        onClick = onClick,
        modifier = modifier,
        enabled = enabled,
        colors = ButtonDefaults.buttonColors(containerColor = Primary),
    ) {
        Text(text = text, style = MaterialTheme.typography.labelLarge)
    }
}
```

화면에서는 버튼을 매번 직접 꾸미지 않고 PrimaryButton을 가져다 쓰기만 하면 된다. 색상과 글자 스타일은 이미 스타일 가이드의 토큰으로 정해져 있다.

### 시스템 가이드

시스템 가이드는 스타일 가이드와 패턴 라이브러리를 어떻게 사용하고 운영할지를 정한 규칙이며, 구조로 보면 개요와 기타에 해당한다.

컴포넌트가 있어도 언제, 어떻게 써야 하는지 정해져 있지 않으면 사람마다 다르게 사용하게 되고, 결국 다시 일관성이 깨진다.

시스템 가이드에는 다음과 같은 내용이 포함된다.

- 사용 가이드라인

각 컴포넌트를 언제 사용하고 언제 사용하지 말아야 하는지를 Do / Don't 형태로 정리한다. 예를 들어 "한 화면에 Primary 버튼은 하나만 사용한다" 같은 규칙이다.

- 접근성

색상 대비, 터치 영역 크기, 스크린 리더 지원 등 모든 사용자가 사용할 수 있도록 지켜야 할 기준을 정한다.

- 운영 규칙

새로운 컴포넌트를 추가하거나 기존 컴포넌트를 변경할 때의 절차, 버전 관리, 변경 내용 공유 방법을 정한다.

디자인 시스템은 한 번 만들고 끝나는 것이 아니라 서비스와 함께 계속 바뀐다. 시스템 가이드는 디자인 시스템이 시간이 지나도 일관성을 유지할 수 있게 해주는 역할을 한다.

## 디자인 시스템 구축 방법론

### 아토믹 디자인(Atomic Design)

디자인 시스템이 무엇으로 구성되는지 알았다면, 이제 이를 어떻게 만들어 나갈지가 남는다.

보통 디자인은 화면 단위로 진행된다. 하지만 화면 단위로 만들다 보면 같은 버튼이나 입력창이 화면마다 조금씩 다르게 만들어지기 쉽다. 그렇다면 처음부터 UI를 작은 단위로 쪼개서 만들고, 이를 조합해 화면을 완성하면 어떨까?

이를 위해 나타난 방식이 아토믹 디자인(Atomic Design)이다.

아토믹 디자인은 2013년 Brad Frost가 제안한 방법론으로, 화학에서 원자가 모여 분자가 되고 분자가 모여 유기체가 되는 것처럼 UI도 작은 단위부터 조합해 큰 단위를 만든다는 개념이다. UI를 원자, 분자, 유기체, 템플릿, 페이지 다섯 단계로 나눈다.

![atomic_process.png](/assets/images/atomic_process.png?raw=true)

출처: [Brad Frost - Atomic Design, Chapter 2](https://atomicdesign.bradfrost.com/chapter-2/)

왼쪽에서 오른쪽으로 갈수록 작고 추상적인 단위에서 크고 구체적인 단위가 된다. 다만 이 순서대로 작업하라는 뜻은 아니다. 다섯 단계가 동시에 존재하면서 서로 연결된 하나의 시스템이라는 관점으로 UI를 바라보는 방법이다.

- 원자(Atoms)

더 이상 쪼갤 수 없는 가장 작은 UI 요소이다. 라벨, 입력창, 버튼, 아이콘처럼 혼자서는 큰 의미를 갖지 않지만 모든 UI의 재료가 된다.

![atomic_atoms.png](/assets/images/atomic_atoms.png?raw=true)

출처: [Brad Frost - Atomic Design, Chapter 2](https://atomicdesign.bradfrost.com/chapter-2/)

검색 기능에 쓰일 라벨, 입력창, 버튼이 각각 하나의 원자이다.

- 분자(Molecules)

원자 여러 개를 묶어 하나의 기능을 하도록 만든 단위이다.

![atomic_molecules.png](/assets/images/atomic_molecules.png?raw=true)

출처: [Brad Frost - Atomic Design, Chapter 2](https://atomicdesign.bradfrost.com/chapter-2/)

라벨, 입력창, 버튼 원자를 묶으면 검색 폼이라는 분자가 된다. 각각은 따로 있을 때 큰 의미가 없었지만, 묶이는 순간 "검색한다"는 하나의 역할이 생긴다.

- 유기체(Organisms)

분자와 원자를 조합해 화면의 한 영역을 이루는 비교적 복잡한 단위이다.

![atomic_organisms.png](/assets/images/atomic_organisms.png?raw=true)

출처: [Brad Frost - Atomic Design, Chapter 2](https://atomicdesign.bradfrost.com/chapter-2/)

로고(원자), 내비게이션(분자), 검색 폼(분자)을 조합하면 사이트 상단의 헤더라는 유기체가 된다.

- 템플릿(Templates)

유기체를 배치해 화면의 뼈대를 만든 단계이다. 실제 콘텐츠 대신 자리만 잡아두기 때문에 화면의 구조에 집중할 수 있다.

![atomic_templates.png](/assets/images/atomic_templates.png?raw=true)

출처: [Brad Frost - Atomic Design, Chapter 2](https://atomicdesign.bradfrost.com/chapter-2/)

헤더, 대표 이미지, 본문, 영상 영역이 어디에 들어갈지만 정해둔 홈 화면의 템플릿이다.

- 페이지(Pages)

템플릿에 실제 콘텐츠를 넣은 최종 화면이다. 사용자가 실제로 보게 되는 결과물이다.

![atomic_pages.png](/assets/images/atomic_pages.png?raw=true)

출처: [Brad Frost - Atomic Design, Chapter 2](https://atomicdesign.bradfrost.com/chapter-2/)

같은 템플릿에 실제 이미지와 텍스트를 넣은 모습이다. 제목이 너무 길거나 이미지가 없는 경우처럼 실제 데이터를 넣어봐야 드러나는 문제를 이 단계에서 확인하고, 필요하면 템플릿이나 하위 요소를 다시 수정한다.

아토믹 디자인의 단계를 앞에서 살펴본 디자인 시스템의 구조와 연결하면 다음과 같다.

| 아토믹 디자인 | 디자인 시스템 구조 | 예시 |
| --- | --- | --- |
| **원자** | 컴포넌트 | 버튼, 입력창, 아이콘, 라벨 |
| **분자** | 컴포넌트 | 검색 폼, 텍스트 필드(라벨 + 입력창 + 도움말) |
| **유기체** | 컴포넌트, 패턴 | 헤더, 상품 목록, 로그인 폼 |
| **템플릿** | 패턴 | 목록-상세 레이아웃, 빈 화면 |
| **페이지** | 실제 서비스 화면 | 홈 화면, 상품 상세 화면 |

원자보다 더 작은 색상, 글꼴, 간격 같은 값은 파운데이션의 디자인 토큰에 해당한다. 토큰으로 원자를 만들고, 원자를 조합해 점점 큰 단위를 만들어 가는 구조이다.

Compose도 작은 컴포저블을 조합해 큰 컴포저블을 만드는 방식이기 때문에 아토믹 디자인과 잘 맞는다.

```kotlin
// 분자: 원자(OutlinedTextField, PrimaryButton)를 묶어 검색 기능을 만든다
@Composable
fun SearchBar(
    query: String,
    onQueryChange: (String) -> Unit,
    onSearch: () -> Unit,
    modifier: Modifier = Modifier,
) {
    Row(modifier = modifier, verticalAlignment = Alignment.CenterVertically) {
        OutlinedTextField(
            value = query,
            onValueChange = onQueryChange,
            placeholder = { Text("검색어를 입력하세요") },
            modifier = Modifier.weight(1f),
        )
        Spacer(modifier = Modifier.width(8.dp))
        PrimaryButton(text = "검색", onClick = onSearch)
    }
}

// 유기체: 원자(Text)와 분자(SearchBar)를 조합해 화면의 한 영역을 만든다
@Composable
fun HomeHeader(
    query: String,
    onQueryChange: (String) -> Unit,
    onSearch: () -> Unit,
    modifier: Modifier = Modifier,
) {
    Column(modifier = modifier.padding(16.dp)) {
        Text(text = "홈", style = MaterialTheme.typography.titleLarge)
        Spacer(modifier = Modifier.height(12.dp))
        SearchBar(query = query, onQueryChange = onQueryChange, onSearch = onSearch)
    }
}
```

PrimaryButton은 앞에서 패턴 라이브러리에 만든 원자이다. 같은 방식으로 템플릿은 콘텐츠를 파라미터로 받아 배치만 담당하는 화면 컴포저블, 페이지는 ViewModel의 실제 상태를 넣어 그린 화면이라고 볼 수 있다.

다만 어떤 요소를 분자로 볼지 유기체로 볼지는 팀마다 기준이 다를 수 있다. 그래서 다섯 단계의 이름을 그대로 쓰기보다, 작은 단위부터 조합해 나간다는 사고방식을 가져가는 것이 더 중요하다.

## 마치며

디자인 시스템은 결국 여러 제품과 팀이 함께 따를 수 있는 기준을 만드는 일이다.

- 스타일 가이드

어떤 모습이어야 하는지에 대한 기준

- 패턴 라이브러리

무엇을 만들어 쓸지에 대한 기준

- 시스템 가이드

어떻게 사용하고 운영할지에 대한 기준

디자인 시스템이 있으면 일관성이 생긴다.

화면, 플랫폼, 팀이 달라도 같은 토큰과 컴포넌트를 쓰기 때문에 사용자는 어디서나 같은 경험을 하게 된다.

디자인 시스템이 있으면 협업이 쉬워진다.

디자이너와 개발자가 같은 이름으로 소통하기 때문에 "이 파란색이 그 파란색 맞나요?" 같은 확인 작업이 줄어든다.

개발자 입장에서 디자인 시스템은 낯선 개념이 아니다. 값을 상수로 빼고, 반복되는 코드를 공통 컴포넌트로 묶고, 작은 단위를 조합해 큰 단위를 만드는 것은 평소에 코드를 설계하는 방식과 같다. 특히 Compose는 컴포저블을 조합해 화면을 만들기 때문에 디자인 시스템을 코드로 옮기기에 잘 맞는 도구이다.

다만 디자인 시스템은 한 번 만들고 끝나는 것이 아니다. 서비스가 바뀌면 디자인 시스템도 함께 바뀌어야 하고, 이를 위해 디자이너와 개발자가 함께 관리해 나가는 것이 중요하다.

## 참조

> 이영주, 『UI/UX 디자인이 쉬워지는 디자인 시스템 실무 with 피그마』, 한빛아카데미, 2024
>
> [https://m3.material.io/](https://m3.material.io/)
>
> [https://atomicdesign.bradfrost.com/](https://atomicdesign.bradfrost.com/)
>
> [https://developer.android.com/develop/ui/compose/designsystems](https://developer.android.com/develop/ui/compose/designsystems)
>
> [https://www.lightningdesignsystem.com/](https://www.lightningdesignsystem.com/)
>
> [https://atlassian.design/](https://atlassian.design/)
>
> [https://developer.apple.com/design/human-interface-guidelines/](https://developer.apple.com/design/human-interface-guidelines/)
>
> [https://carbondesignsystem.com/](https://carbondesignsystem.com/)
>
> [https://styleguide.mailchimp.com/](https://styleguide.mailchimp.com/)
