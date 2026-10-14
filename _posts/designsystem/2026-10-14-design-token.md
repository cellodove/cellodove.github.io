---
title : "디자인 토큰(Design Token)"
excerpt: "디자인 토큰의 개념, 원시·의미·컴포넌트 토큰 계층, 이름 짓기, Compose에서의 사용에 대하여"

categories :
  - DesignSystem

toc: true
toc_sticky: true
last_modified_at: 2026-10-14
---

![design_token_image1.jpg](/assets/images/design_token_image1.jpg?raw=true)

## 디자인 토큰의 이해

### 디자인 토큰 등장 배경

디자인 시스템을 만들면 색상, 글꼴, 간격 같은 값을 스타일 가이드로 정해두게 된다.

하지만 같은 값이라도 사용하는 곳마다 표현 방식이 다르다. 디자이너는 Figma에서 `#2F6FED`라는 색상을 쓰고, Android 개발자는 `Color(0xFF2F6FED)`, iOS 개발자는 `UIColor`, 웹 개발자는 CSS 변수로 같은 색을 다시 정의한다.

처음에는 같은 값으로 시작했더라도 각자 따로 관리하다 보면 어느 순간 조금씩 달라진다. 브랜드 색상 하나를 바꾸려면 디자인 파일과 모든 플랫폼의 코드를 찾아다니며 직접 수정해야 하고, 하나라도 빠뜨리면 다시 일관성이 깨진다.

그렇다면 스타일 가이드에서 정한 값을 어떻게 모든 도구와 플랫폼에서 똑같이 쓸 수 있을까?

### 디자인 토큰 정의

이를 위해 나타난 방식이 디자인 토큰(Design Token)이다.

디자인 토큰은 색상, 글꼴, 간격 같은 디자인 결정에 이름을 붙여 저장한 것이다. 값을 직접 쓰는 대신 이름을 쓰고, 그 이름이 가리키는 값은 한 곳에서만 관리한다.

```kotlin
// 값을 직접 쓰는 경우
Text(text = "안녕하세요", color = Color(0xFF1B1D1F))

// 토큰을 쓰는 경우
Text(text = "안녕하세요", color = AppTheme.colors.textPrimary)
```

두 코드는 화면에 같은 색을 그린다. 차이는 "왜 이 색인가"가 코드에 남는다는 것이다. `0xFF1B1D1F`는 그냥 어두운 회색이지만, `textPrimary`는 "기본 텍스트 색"이라는 의도를 가진다. 나중에 기본 텍스트 색이 바뀌어도 화면 코드는 그대로 두고 토큰의 값만 바꾸면 된다.

핵심은 하나의 원천(Single Source of Truth)이다. 토큰을 플랫폼에 상관없는 형식으로 한 곳에 정의해두고, 각 플랫폼은 이를 자기 형식으로 변환해서 사용한다.

![design_token_concept.png](/assets/images/design_token_concept.png?raw=true)

### 디자인 토큰의 종류

디자인 토큰은 스타일 가이드에서 정하는 거의 모든 값을 다룬다.

| 종류 | 예시 이름 | 예시 값 |
| --- | --- | --- |
| **색상** | `color.primary`, `color.text.secondary` | #2F6FED |
| **타이포그래피** | `typography.title.large` | 글꼴, 22sp, Bold, 줄 간격 28sp |
| **간격** | `spacing.md` | 16dp |
| **모서리** | `shape.corner.medium` | 12dp |
| **그림자(고도)** | `elevation.level1` | 1dp |
| **모션** | `motion.duration.short`, `motion.easing.standard` | 150ms, cubic-bezier(0.2, 0, 0, 1) |
| **투명도** | `opacity.disabled` | 0.38 |

색상이나 간격처럼 값 하나로 끝나는 토큰도 있고, 타이포그래피처럼 글꼴, 크기, 굵기, 줄 간격을 묶은 토큰도 있다.

## 디자인 토큰의 계층

### 왜 계층이 필요한가

토큰을 만들 때 가장 단순한 방법은 색상마다 이름을 하나씩 붙이는 것이다.

```kotlin
val Blue500 = Color(0xFF2F6FED)
```

하지만 이렇게만 하면 화면 코드에 `Blue500`이 곳곳에 쓰이게 된다. 그런데 어떤 `Blue500`은 버튼 배경이고, 어떤 `Blue500`은 링크 텍스트이고, 어떤 `Blue500`은 선택된 탭의 표시이다. 이 상태에서 "버튼 색만 초록으로 바꾸자"는 요구가 오면 `Blue500`을 쓰는 곳을 하나씩 열어보며 버튼인지 아닌지 확인해야 한다.

![design_token_why_layers.png](/assets/images/design_token_why_layers.png?raw=true)

값에 이름을 붙이는 것만으로는 부족하고, 그 값이 어디에 쓰이는지도 함께 이름으로 남겨야 한다. 그래서 디자인 토큰은 보통 원시 토큰, 의미 토큰, 컴포넌트 토큰 세 계층으로 나눈다.

![design_token_levels.png](/assets/images/design_token_levels.png?raw=true)

### 원시 토큰(Primitive Token)

원시 토큰은 값 자체에 이름을 붙인 토큰이다. 디자인 시스템에서 사용할 수 있는 모든 값의 목록이라고 보면 된다.

```kotlin
val Blue300 = Color(0xFF7FA6F5)
val Blue500 = Color(0xFF2F6FED)
val Gray50 = Color(0xFFF7F8FA)
val Gray900 = Color(0xFF1B1D1F)
```

이름에는 용도가 들어가지 않는다. `Blue500`은 "파란색 계열의 500단계"라는 뜻일 뿐, 버튼에 쓰는지 텍스트에 쓰는지는 알 수 없다. 시스템마다 기본 토큰(Base), 전역 토큰(Global), 참조 토큰(Reference)이라고도 부른다.

### 의미 토큰(Semantic Token)

의미 토큰은 원시 토큰에 용도를 붙인 토큰이다. 값을 직접 갖지 않고 원시 토큰을 가리킨다.

```kotlin
val Primary = Blue500
val TextPrimary = Gray900
val Background = Gray50
```

이제 이름만 봐도 어디에 쓰이는지 알 수 있다. 화면 코드는 원시 토큰이 아니라 의미 토큰만 사용한다. 그러면 브랜드 색상이 파란색에서 초록색으로 바뀌어도 `Primary`가 가리키는 원시 토큰만 바꾸면 되고, 화면 코드는 건드릴 필요가 없다.

의미 토큰이 가장 큰 힘을 발휘하는 곳은 다크모드이다. 같은 `TextPrimary`라도 라이트 모드에서는 `Gray900`을, 다크 모드에서는 `Gray50`을 가리키게 하면 된다. 화면은 `TextPrimary`만 알고 있으니 모드가 바뀌어도 코드를 바꿀 필요가 없다.

![design_token_dark_mode.png](/assets/images/design_token_dark_mode.png?raw=true)

시스템마다 별칭 토큰(Alias), 시스템 토큰(System)이라고도 부른다.

### 컴포넌트 토큰(Component Token)

컴포넌트 토큰은 특정 컴포넌트의 특정 부분이 어떤 의미 토큰을 쓰는지 정한 토큰이다.

```kotlin
val ButtonContainerColor = Primary
val ButtonLabelColor = OnPrimary
val ButtonDisabledContainerOpacity = 0.1f
```

"Primary 색을 쓰는 곳"이 아니라 "버튼의 배경"이라는 단위로 값을 관리할 수 있게 된다. 버튼 배경만 다른 색으로 바꾸고 싶다면 `ButtonContainerColor`만 바꾸면 되고, 같은 `Primary`를 쓰는 다른 컴포넌트에는 영향이 없다.

다만 컴포넌트 토큰은 컴포넌트 수만큼 늘어나기 때문에 관리 비용이 크다. 규모가 작은 디자인 시스템에서는 원시 토큰과 의미 토큰 두 계층만 쓰는 경우도 많다.

세 계층을 정리하면 다음과 같다.

| 계층 | 담는 것 | 예시 | 다른 이름 |
| --- | --- | --- | --- |
| **원시 토큰** | 사용할 수 있는 값 | Blue500 = #2F6FED | Base, Global, Reference |
| **의미 토큰** | 값의 용도 | Primary = Blue500 | Alias, System |
| **컴포넌트 토큰** | 컴포넌트의 어느 부분에 쓰는지 | ButtonContainerColor = Primary | Component |

위로 갈수록 값에 가깝고, 아래로 갈수록 용도에 가깝다. 화면과 컴포넌트는 아래 계층을 사용하고, 실제 값은 맨 위 계층에만 존재한다.

### Material 3의 토큰 구조

Material 3도 같은 세 계층을 사용한다. Material 3에서는 참조 토큰(Reference), 시스템 토큰(System), 컴포넌트 토큰(Component)이라고 부르며, 각각 원시, 의미, 컴포넌트 토큰에 해당한다.

Compose Material 3 라이브러리 안에도 이 구조가 그대로 들어있다. `androidx.compose.material3.tokens` 패키지의 코드를 따라가 보면 다음과 같다. (실제 코드에서 필요한 부분만 간추렸다.)

```kotlin
// 참조 토큰: 실제 값
internal object PaletteTokens {
    val Primary40 = Color(red = 103, green = 80, blue = 164)   // #6750A4
    val Primary80 = Color(red = 208, green = 188, blue = 255)  // #D0BCFF
}

// 시스템 토큰: 라이트/다크 모드에서 Primary가 가리키는 참조 토큰
internal object ColorLightTokens {
    val Primary = PaletteTokens.Primary40
}
internal object ColorDarkTokens {
    val Primary = PaletteTokens.Primary80
}

// 컴포넌트 토큰: Filled 버튼의 각 부분이 쓰는 시스템 토큰
internal object FilledButtonTokens {
    val ContainerColor = ColorSchemeKeyTokens.Primary
    val LabelTextColor = ColorSchemeKeyTokens.OnPrimary
    val DisabledContainerOpacity = 0.1f
}
```

`FilledButtonTokens`는 버튼 배경이 `Primary`라는 것만 알고, `Primary`가 라이트 모드에서는 `Primary40`, 다크 모드에서는 `Primary80`을 가리킨다. 우리가 `MaterialTheme.colorScheme.primary`로 꺼내 쓰는 값이 바로 이 시스템 토큰이다.

![design_token_m3.png](/assets/images/design_token_m3.png?raw=true)

## 디자인 토큰 이름 짓기

### 값이 아니라 의도로 이름 짓기

디자인 토큰에서 가장 어려운 일은 이름 짓기이다. 디자이너와 개발자가 같은 이름으로 소통하기 위해 만드는 것이 토큰이기 때문에, 이름이 모호하면 토큰을 만든 의미가 없어진다.

원시 토큰은 값을 설명하는 이름을, 의미 토큰과 컴포넌트 토큰은 용도를 설명하는 이름을 붙인다.

| 나쁜 이름 | 좋은 이름 | 이유 |
| --- | --- | --- |
| **BlueButton** | ButtonContainerColor | 버튼 색이 초록으로 바뀌면 이름이 거짓말이 된다 |
| **DarkGrayText** | TextPrimary | 다크 모드에서는 밝은 색을 가리키게 된다 |
| **Spacing16** | SpacingMd | 값이 바뀌면 이름도 함께 바꿔야 한다 |
| **LoginButtonColor** | ButtonContainerColor | 화면 이름이 들어가면 재사용할 수 없다 |

의미 토큰의 이름에 값이 들어가면 의미 토큰을 만든 이유가 사라진다. `DarkGrayText`가 다크 모드에서 밝은 회색을 가리키는 순간 이름을 믿을 수 없게 된다.

### 이름 구조

토큰 이름은 보통 큰 분류에서 작은 분류 순서로 이어 붙인다.

| 순서 | 의미 | 예시 |
| --- | --- | --- |
| **카테고리** | 어떤 종류의 값인가 | color, spacing, typography |
| **대상** | 무엇에 쓰는가 | text, background, border, button |
| **변형** | 어떤 종류인가 | primary, secondary, danger |
| **상태** | 어떤 상태일 때인가 | pressed, disabled, focused |

예를 들어 `color.text.secondary`, `color.background.primary.pressed`처럼 짓는다.

![design_token_naming.png](/assets/images/design_token_naming.png?raw=true)

모든 단계를 다 붙일 필요는 없고, 필요한 단계만 붙이되 같은 순서를 지키는 것이 중요하다.

간격이나 크기처럼 단계가 있는 값은 `xs`, `sm`, `md`, `lg`, `xl` 같은 티셔츠 사이즈나, IBM Carbon의 `$spacing-01`부터 `$spacing-13`처럼 번호를 붙이는 방식을 많이 쓴다. 어느 쪽이든 팀 전체가 같은 규칙을 따르는 것이 중요하다.

## 디자인 토큰 관리

### 디자인 도구에서의 토큰

Figma에서는 변수(Variables)로 디자인 토큰을 관리한다. 색상, 숫자 같은 값을 변수로 만들고, 변수가 다른 변수를 참조하게 해서 원시 토큰과 의미 토큰의 관계를 만들 수 있다.

변수에는 모드(Mode)를 둘 수 있다. 하나의 변수에 라이트 모드 값과 다크 모드 값을 따로 정해두면, 디자이너는 화면을 다시 그리지 않고 모드만 바꿔서 다크 모드 화면을 확인할 수 있다. 앞에서 본 의미 토큰이 모드에 따라 다른 원시 토큰을 가리키는 구조와 같다.

## Compose에서 디자인 토큰 사용하기

### 토큰 정의

Compose에서는 앞에서 본 계층을 그대로 코드로 옮길 수 있다. 원시 토큰은 값을 모아둔 object로, 의미 토큰은 원시 토큰을 가리키는 data class로 만든다.

```kotlin
// 원시 토큰: 사용할 수 있는 모든 값
object PrimitiveColors {
    val White = Color(0xFFFFFFFF)
    val Gray50 = Color(0xFFF7F8FA)
    val Gray900 = Color(0xFF1B1D1F)
    val Blue300 = Color(0xFF7FA6F5)
    val Blue500 = Color(0xFF2F6FED)
}

// 의미 토큰: 값의 용도
@Immutable
data class SemanticColors(
    val primary: Color,
    val onPrimary: Color,
    val background: Color,
    val textPrimary: Color,
)

val LightColors = SemanticColors(
    primary = PrimitiveColors.Blue500,
    onPrimary = PrimitiveColors.White,
    background = PrimitiveColors.White,
    textPrimary = PrimitiveColors.Gray900,
)

val DarkColors = SemanticColors(
    primary = PrimitiveColors.Blue300,
    onPrimary = PrimitiveColors.Gray900,
    background = PrimitiveColors.Gray900,
    textPrimary = PrimitiveColors.Gray50,
)

// 간격 토큰
@Immutable
data class Spacing(
    val xs: Dp = 4.dp,
    val sm: Dp = 8.dp,
    val md: Dp = 16.dp,
    val lg: Dp = 24.dp,
)
```

라이트 모드와 다크 모드는 같은 `SemanticColors`의 서로 다른 인스턴스이다. 의미 토큰의 이름은 같고, 가리키는 원시 토큰만 다르다.

### 테마로 토큰 제공하기

정의한 토큰은 `CompositionLocal`로 화면 전체에 전달한다. `MaterialTheme`이 `colorScheme`, `typography`를 제공하는 것과 같은 방식이다.

```kotlin
val LocalSemanticColors = staticCompositionLocalOf { LightColors }
val LocalSpacing = staticCompositionLocalOf { Spacing() }

@Composable
fun AppTheme(
    darkTheme: Boolean = isSystemInDarkTheme(),
    content: @Composable () -> Unit,
) {
    val colors = if (darkTheme) DarkColors else LightColors
    CompositionLocalProvider(
        LocalSemanticColors provides colors,
        LocalSpacing provides Spacing(),
        content = content,
    )
}

object AppTheme {
    val colors: SemanticColors
        @Composable get() = LocalSemanticColors.current

    val spacing: Spacing
        @Composable get() = LocalSpacing.current
}
```

`staticCompositionLocalOf`는 값이 바뀌면 그 아래 전체를 다시 그리지만, 대신 값을 읽는 곳을 추적하는 비용이 없다. 테마 토큰처럼 거의 바뀌지 않고 화면 곳곳에서 읽는 값에 적합하다.

### 토큰 사용하기

화면과 컴포넌트에서는 `AppTheme`을 통해 의미 토큰만 꺼내 쓴다.

```kotlin
// 컴포넌트 토큰: 버튼의 각 부분이 쓰는 의미 토큰
object PrimaryButtonTokens {
    val ContainerColor: Color
        @Composable get() = AppTheme.colors.primary

    val LabelColor: Color
        @Composable get() = AppTheme.colors.onPrimary
}

@Composable
fun PrimaryButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier,
) {
    Button(
        onClick = onClick,
        modifier = modifier,
        colors = ButtonDefaults.buttonColors(
            containerColor = PrimaryButtonTokens.ContainerColor,
            contentColor = PrimaryButtonTokens.LabelColor,
        ),
    ) {
        Text(text = text)
    }
}

@Composable
fun ProfileScreen() {
    Column(
        modifier = Modifier
            .fillMaxSize()
            .background(AppTheme.colors.background)
            .padding(AppTheme.spacing.md),
    ) {
        Text(text = "프로필", color = AppTheme.colors.textPrimary)
        Spacer(modifier = Modifier.height(AppTheme.spacing.sm))
        PrimaryButton(text = "저장", onClick = {})
    }
}
```

화면 코드 어디에도 `Color(0xFF...)`나 `16.dp` 같은 값이 없다. 다크 모드로 바뀌면 `AppTheme`이 `DarkColors`를 제공하고, 화면 코드는 그대로 둔 채 색만 바뀐다.

`MaterialTheme`을 그대로 쓰고 싶다면 의미 토큰을 `lightColorScheme(primary = ..., onPrimary = ...)`에 연결하는 방법도 있다. 이 경우 Material 컴포넌트가 자동으로 우리 토큰을 사용하게 된다. Material의 색상 역할로 표현할 수 없는 토큰이 필요할 때 위처럼 직접 테마를 만든다.

## 마치며

디자인 토큰은 디자인 결정에 이름을 붙여 한 곳에서 관리하는 방법이다.

- 원시 토큰

사용할 수 있는 값의 목록

- 의미 토큰

그 값을 어디에 쓰는지에 대한 이름

- 컴포넌트 토큰

컴포넌트의 어느 부분에 어떤 의미 토큰을 쓰는지에 대한 이름

디자인 토큰이 있으면 변경이 쉬워진다.

브랜드 색상이 바뀌어도, 다크 모드를 추가해도 화면 코드를 수정하지 않고 토큰이 가리키는 값만 바꾸면 된다.

디자인 토큰이 있으면 디자인과 코드가 같은 언어를 쓴다.

디자이너가 Figma에서 쓰는 이름과 개발자가 코드에서 쓰는 이름이 같기 때문에 "이 파란색이 그 파란색 맞나요?" 대신 "여기는 textSecondary 맞나요?"라고 물을 수 있다.

개발자 입장에서 디자인 토큰은 하드코딩된 값을 상수로 빼는 것에서 한 걸음 더 나아간 것이다. 상수에 값이 아니라 의도로 이름을 붙이고, 그 상수를 다시 계층으로 나눠 변경이 한 곳에서 끝나도록 만든다.

다만 토큰은 이름을 잘못 지으면 오히려 혼란을 만든다. 그래서 토큰을 만드는 것보다 디자이너와 개발자가 함께 이름 규칙을 정하고 지켜나가는 것이 더 중요하다.

## 참조

> [https://m3.material.io/foundations/design-tokens](https://m3.material.io/foundations/design-tokens)
>
> [https://github.com/material-components/material-web/blob/main/docs/theming/README.md](https://github.com/material-components/material-web/blob/main/docs/theming/README.md)
>
> [https://github.com/androidx/androidx/tree/androidx-main/compose/material3/material3/src/commonMain/kotlin/androidx/compose/material3/tokens](https://github.com/androidx/androidx/tree/androidx-main/compose/material3/material3/src/commonMain/kotlin/androidx/compose/material3/tokens)
>
> [https://help.figma.com/hc/en-us/articles/15339657135383-Guide-to-variables-in-Figma](https://help.figma.com/hc/en-us/articles/15339657135383-Guide-to-variables-in-Figma)
>
> [https://developer.android.com/develop/ui/compose/designsystems/custom](https://developer.android.com/develop/ui/compose/designsystems/custom)
>
> [https://carbondesignsystem.com/elements/spacing/overview/](https://carbondesignsystem.com/elements/spacing/overview/)
>
> [https://www.krds.go.kr/html/site/style/style_07.html](https://www.krds.go.kr/html/site/style/style_07.html)
