# Compose API Guidelines (KO)

Compose API 가이드라인은 관용적인 Jetpack Compose API를 작성하기 위한 패턴, 모범 사례 및 규범적인 스타일 지침을 개략적으로 설명합니다. Jetpack Compose 코드가 레이어에 내장되면서, Jetpack Compose를 사용하는 코드를 작성하는 모든 사람들은 이를 사용하기 위해 자신만의 API를 구축하고 있습니다.

이 문서는 `@Composable`, `remember {}` 및 `CompositionLocal`을 포함한 Jetpack Compose의 runtime API에 익숙하다고 가정합니다.

각 지침의 요구사항 수준은 다음과 같은 개발자 대상자에 대해 [RFC2119](https://www.ietf.org/rfc/rfc2119.txt)에 명시된 용어를 사용하여 지정됩니다. 지침에 대한 요구사항 수준을 명시적으로 지정하지 않은 청중은 해당 가이드라인이 선택 사항이라고 가정해야 합니다.

<br/>

### Jetpack Compose 프레임워크 개발
androidx.compose 라이브러리와 도구에 대한 기여는 일관성을 촉진하고 모든 레이어에서 사용자 코드에 대한 기대치와 예제를 설정하기 위해 일반적으로 이 지침을 엄격하게 따릅니다.

<br/>

### Jetpack Compose 기반 라이브러리 개발
`@Composable` 함수의 public API를 노출하고 앱 및 기타 라이브러리의 사용 타입을 지원하는 Jetpack Compose를 대상으로 하는 외부 라이브러리 생태계가 존재할 것으로 기대되고 있습니다. 이러한 라이브러리들이 Jetpack Compose 프레임워크 개발과 같은 정도로 이 가이드라인을 따르는 것이 바람직하지만, 조직의 우선순위와 지역적 일관성에 따라 일부 순수한 스타일 가이드라인을 완화하는 것이 적절할 수 있습니다.

<br/>

### Jetpack Compose 기반 앱 개발
앱 개발은 종종 기존 앱 아키텍처와 통합하기 위한 요구 사항뿐만 아니라 강력한 조직 우선 순위와 규범을 따릅니다. 이것은 이 지침들로부터 스타일적 편차뿐만 아니라 구조적 편차도 요구할 수 있습니다. 가능한 경우, 앱 개발을 위한 대안적인 접근법이 이 문서에 나열될 것이며, 이러한 상황에서 더 적합할 수 있습니다.

<br/>

## 코틀린 스타일
### 기준 스타일 가이드라인
**Jetpack Compose 프레임워크 개발**은 아래에 설명된 추가 조정과 함께 https://kotlinlang.org/docs/reference/coding-conventions.html 에 기준선으로 명시된 Kotlin Coding Conventions를 따라야 합니다.

**Jetpack Compose Library와 앱 개발**도 이와 같은 지침을 따라야 합니다.

<br/>

### Why?
코틀린 코딩 협약은 코틀린 생태계 전반에 대한 일관성의 표준을 확립합니다. Jetpack Compose에 대한 이 문서의 추가적인 스타일 지침은 Jetpack Compose의 언어-레벨 확장, 멘탈 모델 및 의도된 데이터 흐름을 설명하여 Compose별 패턴에 대한 일관된 규칙과 기대를 설정합니다.

<br/>

### 싱글톤, 상수, 밀폐 클래스 및 열거 클래스 값
**Jetpack Compose 프레임워크 개발**은 여기에 문서화된 것처럼 `PascalCase`의 객체 선언 규칙에 따라 불변 상수를 `CAPITALS_AND_UNDERSCORES`의 대체물로 명명해야 합니다. `Enum` 클래스 값은 동일한 섹션에 설명된 대로 `PascalCase`를 사용하여 이름을 지정해야 합니다.

**라이브러리 개발**은 Jetpack Compose를 대상으로 하거나 확장할 때 동일한 규칙을 따라야 합니다.

**앱 개발**은 이 규약을 따를 수도 있습니다.

<br/>

### Why?
Jetpack Compose는 시간이 지남에 따라 스레드 간에 안정적으로 처리될 수 없는 싱글톤 또는 동반 객체 상태의 사용 및 생성을 억제하여 싱글톤 객체와 다른 형태의 상수를 구분하는 유용성을 줄입니다. 이는 구현 세부 정보가 최상위 레벨의 `val`, 동반 객체, 열거 클래스 또는 중첩 객체 하위 클래스, `sealed` 클래스인지 여부에 관계없이 코드를 소비하기 위한 일관된 API의 안정성에 대한 기대를 형성할 수 있습니다. `myFunction(Foo)` 및 `myFunction(Foo.Bar)`은 구체적인 구현 세부 사항에 관계없이 호출 코드에 대해 동일한 의미와 의도를 가지고 있습니다.

코드베이스에 `CAPITALS_AND_UNDERSCORES`에 대한 기존의 강력한 규칙이 존재하는 라이브러리 및 앱 코드는 이 패턴을 사용하여 로컬 일관성을 유지하는 방법을 고려할 수 있습니다.

<br/>

### ✅ Do
```kotlin
const val DefaultKeyName = "__defaultKey"

val StructurallyEqual: ComparisonPolicy = StructurallyEqualsImpl(...)

object ReferenceEqual : ComparisonPolicy {
    // ...
}

sealed class LoadResult<T> {
    object Loading : LoadResult<Nothing>()
    class Done(val result: T) : LoadResult<T>()
    class Error(val cause: Throwable) : LoadResult<Nothing>()
}

enum class Status {
    Idle,
    Busy
}
```

<br/>

### ❌ Don't
```kotlin
const val DEFAULT_KEY_NAME = "__defaultKey"

val STRUCTURALLY_EQUAL: ComparisonPolicy = StructurallyEqualsImpl(...)

object ReferenceEqual : ComparisonPolicy {
    // ...
}

sealed class LoadResult<T> {
    object Loading : LoadResult<Nothing>()
    class Done(val result: T) : LoadResult<T>()
    class Error(val cause: Throwable) : LoadResult<Nothing>()
}

enum class Status {
    IDLE,
    BUSY
}
```

<br/>

## Compose baseline
Compose 컴파일러 플러그인과 runtime은 코틀린을 위한 새로운 언어 기능과 이들과 상호 작용하는 수단을 설정합니다. 이 레이어는 시간이 지남에 따라 가변 트리 데이터 구조를 구성하고 관리하기 위한 선언적 프로그래밍 모델을 추가합니다. Compose UI는 Compose runtime이 관리할 수 있는 트리 타입의 예시이지만, 이에 국한되지 않습니다.

이 섹션에서는 Composing runime 기능을 기반으로 하는 `@Composable` 함수 및 API에 대한 지침을 개략적으로 설명합니다. 이 지침은 관리되는 트리 타입에 관계없이 모든 runtime 기반 API 구성에 적용됩니다.

<br/>

### Unit @Composable 함수를 엔티티로 명명
**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**은 `Unit`을 리턴하고 `PascalCase`를 사용하여 `@Composable` 어노테이션을 포함하는 함수의 이름을 지정해야 하며, 이름은 명사이어야 하며, 동사나 동사 구문이 아니어야 하며, 명명된 전치사, 형용사 또는 부사여야 합니다. 명사는 서술형 용사로 접두사를 붙일 수 있습니다. 이 가이드라인은 함수가 UI 요소를 방출하는지 여부에 관계없이 적용됩니다.

**앱 개발**은 이와 같은 관례를 따라야 합니다.

<br/>

### Why?
`Unit`을 리턴하는 Composable 함수는 composition에 존재하거나 존재하지 않을 수 있는 선언적 엔티티로 간주되므로 클래스의 명명 규칙을 따릅니다. 호출자의 제어 흐름에 따른 composable의 존재 또는 부재는 recomposition 전반에 걸친 영구적 아이덴티티와 해당 영구적 아이덴티티에 대한 라이프사이클을 모두 설정합니다. 이 명명 규칙은 이 선언적 멘탈 모델을 촉진하고 강화합니다.

<br/>

### ✅ Do
```kotlin
// 이 함수는 PascalCased 명사로 시각적 UI 요소를 설명합니다
@Composable
fun FancyButton(text: String, onClick: () -> Unit) {
```

<br/>

### ✅ Do
```kotlin
// 이 함수는 PascalCased 명사로 composition에 존재하는 비시각적 요소를 설명합니다.
@Composable
fun BackButtonHandler(onBackPressed: () -> Unit) {
```

<br/>

### ❌ Don't
```kotlin
// 이 함수는 명사이지만 PascalCase가 아닙니다!
@Composable
fun fancyButton(text: String, onClick: () -> Unit) {
```

<br/>

### ❌ Don't
```kotlin
// 이 함수는 PascalCased이지만 명사가 아닙니다!
@Composable
fun RenderFancyButton(text: String, onClick: () -> Unit) {
```

<br/>

### ❌ Don't
```kotlin
// 이 함수는 PascalCase도 아니고 명사도 아닙니다!
@Composable
fun drawProfileImage(image: ImageAsset) {
```

<br/>

## 값을 리턴하는 @Composable 함수 명명
**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**은 `Unit`이 아닌 값을 리턴하는 `@Composable`에 어노테이션이 달린 함수의 이름을 지정하기 위해 표준 Kotlin 코딩 규칙을 따라야 합니다.

**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**은 함수의 추상적 리턴 타입과 매치되는 `PascalCase`로 명명된 어노테이션이 달린 함수의 이름을 지정하기 위해 Kotlin Coding Conventions의 팩토리 함수 규약을 사용해서는 안 됩니다.

<br/>

### Why?
이 팩토리 함수 규약은 `@Composable` 함수 이외에서 유용하고 허용되지만 `@Composable` 함수와 함께 사용할 때 호출자에게 부적절한 기대치를 설정하는 단점이 있습니다.

팩토리 함수를 `@Composable`로 표시하는 주요 예시로는, composition을 사용하여 객체의 라이프사이클을 설정하거나 `CompositionLocals`를 객체의 구성에 대한 입력으로 사용하는 경우가 있습니다. 전자는 recomposition 전반에 걸쳐 객체 인스턴스를 캐시하고 유지 관리하기 위해 Compose의 `remember {}` API를 사용하는 것을 의미하며, 이는 생성자 호출처럼 읽어들이는 팩토리 작업에 대한 호출자의 예상을 깨뜨릴 수 있습니다.(다음 섹션 참조) 후자의 동기는 팩토리 함수 이름으로 표현되어야 하는 보이지 않는 입력을 의미합니다.

또한 선언적 구체로서 `@Composable` 함수를 리턴하는 단위의 멘탈 모델은 "가상 DOM" 멘탈 모델과 혼동되어서는 안 됩니다. `PascalCase` 명사로 명명된 `@Composable` 함수에서 값을 리턴하는 것은 이러한 혼란을 조장하고, 호이스팅된 상태 객체로 더 잘 표현할 수 있으며 현재 UI 엔티티에 대해 상태 저장을 제어하는 surface를 리턴하는 바람직하지 않은 스타일을 촉진할 수 있습니다.

상태 호이스팅 패턴에 대한 자세한 내용은 이 문서의 디자인 패턴 섹션을 참조하십시오.

<br/>

### ✅ Do
```kotlin
// 현재 CompositionLocal 설정을 기준으로 스타일을 반환합니다. 이 함수는 해당 값의 출처를 한정합니다.
@Composable
fun defaultStyle(): Style {
```

<br/>

### ❌ Don't
```kotlin
// 현재 CompositionLocal 설정을 기반으로 스타일을 반환합니다. 이 함수는 컨텍스트가 없는 객체를 구성하는 것처럼 보입니다!
@Composable
fun Style(): Style {
```

<br/>

## 반환되는 객체를 `remember {}` 로 만드는 @Composable 함수 명명
**Jetpack Composite 프레임워크 개발 및 라이브러리 개발**은 내부적으로 `remember {}`를 만들고 `remember`라는 접두사를 가진 가변 객체를 리턴하는 모든 `@Composable` 팩토리 함수 앞에 접두사를 붙여야 합니다.

**앱 개발**은 이와 같은 관례를 따라야 합니다.

<br/>

### Why?
시간이 지남에 따라 바뀔 수 있고 recomposition 전반에 걸쳐 지속되는 객체는 observable한 사이드 이펙트를 가지고 있으며, 이는 호출자에게 명확하게 전달되어야 합니다. 이는 또한 호출자가 이런 지속성을 얻기 위해 호출 하는 곳에서 객체의 `remember {}`를 복제할 필요가 없음을 나타냅니다.

<br/>

### ✅ Do
```kotlin
// 이 호출이 composition을 떠날 때 취소될 CoroutineScope를 리턴합니다.
// 이 함수 앞에는 동작을 설명하기 위해 rememeber 접두사가 앞에 붙습니다.
@Composable
fun rememberCoroutineScope(): CoroutineScope {
```

<br/>

### ❌ Don't
```kotlin
// 이 호출이 composition을 떠날 때 취소될 CoroutineScope를 리턴합니다.
// 이 함수의 이름으로는 자동으로 취소되는 동작을 예측할 수 없습니다!
@Composable
fun createCoroutineScope(): CoroutineScope {
```
객체를 리턴하는 것만으로는 함수를 팩토리 함수로 간주하기에 충분하지 않습니다. 이는 함수의 주요 목적이어야 합니다. `Flow<T>.collectAsState()` 같은 `@Composable` 함수를 생각해보세요. 이 함수의 주요 목적은 flow에 대한 구독을 설정하는 것이고, 반환된 `State<T>` 객체를 `remember {}`로 만드는 것은 부수적인 것입니다.

<br/>

## CompositionLocals 명명
`CompositionLocal`은 composition-scope 키-값 테이블의 키입니다. `CompositionLocal`은 특정 composition 하위 트리에 전역 값 같은 값을 제공하는 데 사용될 수 있습니다.

**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**시 "`CompositionLocal`" 또는 "`Local`"을 명사 접미사로 사용하여 `CompositionLocal` 키에 이름을 지정하면 안 됩니다. `CompositionLocal` 키에는 값을 기준으로 설명하는 이름이 있어야 합니다.

**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**시 적합한 설명할 만한 이름이 없는 경우 `CompositionLocal` 키 이름의 접두사로 "`Local`"을 사용할 수 있습니다.

<br/>

### ✅ Do
```kotlin
// 여기서 "Local"은 형용사로 사용되며, "Theme"는 명사입니다.
val LocalTheme = staticCompositionLocalOf<Theme>()
```

### ❌ Don't
```kotlin
// 여기서 "Local"은 명사로 쓰이고 있습니다!
val ThemeLocal = staticCompositionLocalOf<Theme>()
```

<br/>

## Stable 타입
Compose 런타임은 두 가지 어노테이션을 노출시켜 타입 또는 함수를 안정적으로 표시하는 데 사용할 수 있습니다. 이렇게 하면 Compose 컴파일러 플러그인에 의해 최적화 대상이 되어 Compose 런타임이 입력이 변경되지 않는 한 결과가 변경될 수 없는 함수 호출을 건너뛸 수 있습니다.

Compose 컴파일러 플러그인은 타입의 이러한 프로퍼티을 자동으로 추론할 수 있지만 안정성을 추론할 수 없는 인터페이스 및 기타 타입은 명시적으로 어노테이션을 처리해야 합니다. 이러한 타입을 "안정적인 타입(stable types)"이라고 합니다.

`@Immutable`은 객체가 생성된 후에는 프로퍼티의 값이 절대로 변경되지 않으며, 모든 메서드가 참조적으로 투명한 타입을 나타냅니다. Kotlin에서 `const` 식에 사용될 수 있는 모든 타입(primitive 타입 및 `String`)은 `@Immutable`로 간주됩니다.

`@Stable`은 타입에 적용되면 해당 타입이 가변적이지만, Compose 런타임은 public 프로퍼티나 메서드의 동작이 이전 호출과 다른 결과를 생성할 때 알림을 받습니다. (실제로 이 알림은 스냅샷 시스템의 `@Stable` `MutableState` 객체를 통해 지원되며 `mutableStateOf()`에 의해 리턴됩니다.) 이러한 타입은 프로퍼티를 다른 `@Stable` 또는 `@Immutable` 타입을 사용해야만 백업할 수 있습니다.

**Jetpack Compose 프레임워크 개발, 라이브러리 개발 및 앱 개발**시 `@Stable` 타입에 대한 커스텀 `.equals()` 구현에서 항상 두 참조 `a`와 `b`에 대해 `a.equals(b)`가 항상 동일한 값을 리턴해야 함을 보장해야 합니다. 이는 `a`와 `b` 모두에 대한 미래의 변경 사항도 반영되어야 함을 의미합니다.

이 제약은 항상 `a === b`일 경우에는 암시적으로 충족됩니다. 객체에 대한 기본 참조 동등성 구현은 언제나 이 계약의 올바른 구현입니다.

**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**시 public API의 일부로 노출되는 `@Stable` 및 `@Immutable` 타입을 올바르게 어노테이션 처리해야 합니다.

**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**시 이전의 안정적인 버전에 이미 해당 어노테이션으로 선언된 타입에서 `@Stable` 또는 `@Immutable` 어노테이션을 제거해서는 안 됩니다.

**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**시 이전의 안정적인 버전에서 해당 어노테이션 없이 사용 가능한 기존의 non-final 타입에 `@Stable` 또는 `@Immutable` 주석을 추가해서는 안 됩니다.

<br/>

### Why?
`@Stable`과 `@Immutable`은 Compose 컴파일러 플러그인에 의해 생성된 코드의 이진 호환성에 영향을 미치는 행위의 계약입니다. 라이브러리는 기존에 이미 존재하는 non-final 타입에 대해 더 제한적인 계약을 선언해서는 안 되며, 이미 사용 중인 기존 구현이 올바르게 구현되지 않을 수도 있는 제한적인 계약을 선언해서도 안 됩니다. 마찬가지로, 라이브러리의 타입이 이전에 선언된 계약을 더 이상 따르지 않게 선언해서는 안 됩니다. 기존 코드가 해당 계약에 의존할 수 있기 때문입니다.

`@Stable` 또는 `@Immutable`로 어노테이션이 달린 타입을 잘못 구현하면 해당 타입을 매개변수나 수신자로 사용하는 `@Composable` 함수에 대해 잘못된 동작이 발생할 수 있습니다.

<br/>

### 값을 리턴하거나 구성 요소를 생성하는 작업(Emit XOR return a value)
`@Composable` 함수는 구성 요소를 composition에 포함시키거나 값을 리턴해야 하며, 둘 다 동시에 수행해서는 안 됩니다. 만약 composable 함수가 호출자에게 추가적인 옵션을 제공해야 한다면, 이러한 옵션이나 콜백은 호출자에 의해 composable 함수에 매개변수로 제공되어야 합니다.

**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**시 트리 노드를 생성하고 동시에 값을 반환하는 단일 `@Composable` 함수를 노출해서는 안 됩니다.

<br/>

### Why?
Emit 작업은 composition에 표시되는 내용이 나타날 순서대로 발생해야 합니다. 리턴 값을 사용하여 호출자와 통신하는 것은 호출 코드의 모양을 제한하고, 그 앞에서 선언된 다른 선언적 호출과의 상호작용을 방해합니다.

<br/>

### ✅ Do
```kotlin
// 입력 상태를 요청하기 위해 inputState 인터페이스 객체를 호출하는 텍스트 입력 필드 엘리먼트를 생성(Emit)
@Composable
fun InputField(inputState: InputState) {
    // ...
}

// input field와의 상호작용은 순서에 의존하지 않음
val inputState = remember { InputState() }

Button("Clear input", onClick = { inputState.clear() })

InputField(inputState)
```

<br/>

### ❌ Don't
```kotlin
// 텍스트 입력 필드 엘리먼트를 생성(Emit)하고 입력 값의 홀더를 반환
@Composable
fun InputField(): UserInputState {
    // ...
}

// InputField와의 소통을 어렵게 만든다.
Button("Clear input", onClick = { TODO("???") })
val inputState = InputField()
```
매개변수를 전달하여 composable과 통신하는 것은 해당 매개변수들을 호출자의 매개변수로 사용되는 타입에 그룹화할 수 있는 기능을 제공합니다.
```kotlin
interface DetailCardState {
    val actionRailState: ActionRailState
    // ...
}

@Composable
fun DetailCard(state: DetailCardState) {
    Surface {
        // ...
        ActionRail(state.actionRailState)
    }
}

@Composable
fun ActionRail(state: ActionRailState) {
    // ...
}
```
이 패턴에 대한 자세한 정보는 아래의 Compose API 디자인 패턴 섹션에서 **hoisted state types**(호이스팅된 상태 타입)에 관한 부분을 참조하세요.

<br/>

## Compose UI API 구조
Compose UI는 Compose 런타임 위에 구축된 UI 툴킷입니다. 이 섹션에서는 Compose UI 툴킷을 사용하고 확장하는 API에 대한 지침을 설명합니다.

<br/>

### Compose UI 요소(Elements)
하나의 Compose UI 트리 노드를 정확히 발생시키는 `@Composable` 함수를 "요소(element)"라고 합니다.

<br/>

### 예시:
```kotlin
@Composable
fun SimpleLabel(
    text: String,
    modifier: Modifier = Modifier
) {
```

<br/>
**Jetpack Compose 프레임워크 개발과 라이브러리 개발**은 이 섹션의 모든 지침을 따라야 합니다.

**Jetpack Compose 앱 개발**은 이 섹션의 모든 지침을 가능한 만큼 따르는 것이 좋습니다.

<br/>

### 요소(Elements)는 Unit을 리턴합니다.
요소는 반드시 루트 UI 노드를 직접 `emit()`을 호출하거나 다른 Compose UI 요소 함수를 호출하여 발생(emit)시켜야 합니다. 요소는 값을 리턴해서는 안 되며, composition의 상태가 아닌 요소의 모든 동작은 요소 함수에 전달된 매개변수를 통해 제공되어야 합니다.

<br/>

### Why?
요소는 Compose UI 구성(composition)에서 선언적 엔티티입니다. 그들의 존재 또는 부재가 결과적으로 UI에 나타나는지를 결정합니다. 값을 리턴하는 것은 필요하지 않으며, 발생시킨 요소를 제어하는 어떤 방법이든 요소 함수에 전달된 매개변수로 제공되어야 합니다. 자세한 정보는 이 문서의 **Compose API 디자인 패턴** 섹션의 **hoisted state** 부분을 참조하세요.

<br/>

### ✅ Do
```kotlin
@Composable
fun FancyButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier
) {
```

<br/>

### ❌ Don't
```kotlin
interface ButtonState {
    val clicks: Flow<ClickEvent>
    val measuredSize: Size
}

@Composable
fun FancyButton(
    text: String,
    modifier: Modifier = Modifier
): ButtonState {
```

<br/>

### 요소는 Modifier 매개변수를 받아들이고 존중합니다.
요소 함수는 반드시 `Modifier` 타입의 매개변수를 받아들여야 합니다. 이 매개변수는 **modifier라는** 이름으로 명명되어야 하며, 요소 함수의 매개변수 목록에서 첫 번째 옵셔널 매개변수로 나타나야 합니다. 요소 함수는 여러 개의 `Modifier` 매개변수를 받아들여서는 안 됩니다.

만약 요소 함수의 내용이 자연스러운 최소의 크기를 가지고 있는 경우 - 즉, `minWidth`와 `minHeight`가 0인 제약 조건으로도 비정규적인 크기로 측정될 수 있는 경우 - `modifier` 매개변수의 기본 값은 `Modifier`로 지정되어야 합니다. 이는 `Modifier` 타입의 동반 객체로서 빈 `Modifier`를 나타냅니다. 측정 가능한 내용 크기가 없는 요소 함수(예: 사용 가능한 크기에서 임의의 사용자 콘텐츠를 그리는 `Canvas`)는 `modifier` 매개변수를 요구하고 기본 값을 생략할 수 있습니다.

요소 함수는 자신이 생성한 Compose UI 노드에 제공되는 `modifier` 매개변수를 전달해야 합니다. 요소 함수가 직접 Compose UI `Layout` 노드를 발생시키는 경우, `modifier는` 해당 노드에 제공되어야 합니다.

요소 함수는 자신이 생성한 Compose UI 노드에 전달하기 전에 추가적인 `modifier`를 수행할 수 있습니다. 이러한 경우 추가적인 `modifier`를 수행하기 전에 수신한 `modifier` 매개변수의 끝에 이어붙여야 합니다.

요소 함수는 자신이 생성한 Compose UI 노드에 전달하기 전에 수신한 `modifier` 매개변수의 시작 부분에 추가적인 `modifier`를 연결해서는 안 됩니다.

<br/>

### Why?
`Modifier`는 Compose UI에서 요소에 외부 동작을 추가하는 표준 수단이며, 공통 동작을 개별 또는 기본 요소 API surface에서 분리할 수 있습니다. 이는 요소 API를 더 작고 집중적으로 유지할 수 있도록 해주며, `modifier`를 사용하여 해당 요소에 표준 동작을 추가합니다.

이 표준 방식으로 `modifier를` 받아들이지 않는 요소 함수는 이러한 데코레이션을 허용하지 않으며, 원하는 `modifier`를 적용하기 위해 소비 코드가 호출을 래핑하게 됩니다. 이렇게 되면 요소를 수정하는 개발자의 행위가 방해되며, 원하는 결과를 얻기 위해 더 깊은 트리 구조의 비효율적인 UI 코드를 작성하도록 강제됩니다.

`modifier`는 요소의 공통 케이스(common case)에 대해 항상 최종 위치 매개변수로 `modifier`를 제공할 수 있는 개발자의 일관된 기대를 설정합니다.

자세한 내용은 아래의 **Compose UI modifiers** 섹션을 참조하세요.

<br/>

### ✅ Do
```kotlin
@Composable
fun FancyButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier
) = Text(
    text = text,
    modifier = modifier.surface(elevation = 4.dp)
        .clickable(onClick)
        .padding(horizontal = 32.dp, vertical = 16.dp)
)
```

<br/>

### Compose UI layouts
하나 이상의 `@Composable` 함수 매개변수를 받는 Compose UI 요소를 layout이라고 합니다.

<br/>
예시:
```kotlin
@Composable
fun SimpleRow(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
```

<br/>

**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**은 이 섹션의 모든 지침을 따라야 합니다.

**Jetpack Compose 앱 개발**은 이 섹션의 모든 지침을 가능한 경우 따라야 합니다.

<br/>

Layout 함수는 오직 하나의 `@Composable` 함수 매개변수만을 받는 경우, 이를 "**content**"라는 이름으로 사용해야 합니다.

Layout 함수는 두 개 이상의 `@Composable` 함수 매개변수를 받는 경우, 주요 또는 가장 일반적인 `@Composable` 함수 매개변수에 대해 "**content**"라는 이름을 사용해야 합니다.

Layout 함수는 주요 또는 가장 일반적인 `@Composable` 함수 매개변수를 마지막 매개변수 위치에 배치하여 Kotlin의 trailing lambda syntax를 사용할 수 있도록 해야 합니다.

<br/>

### Compose UI modifiers
`Modifier`는 `Modifier.Element` 인터페이스를 구현하는 객체들의 변경 불가능하며 순서가 있는 컬렉션입니다. `Modifier`는 Compose UI 요소에 대한 범용적인 데코레이터로서, 요소에 교차하는 행위를 불투명하고 캡슐화된 방식으로 구현하고 추가하는 데 사용될 수 있습니다. `Modifier`의 예시로는 요소 크기 및 `padding` 변경, 요소 아래 또는 겹치게 콘텐츠 그리기, 또는 UI 요소의 bounding box 내에서 터치 이벤트 감지 등이 있습니다.

**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**은 이 섹션의 모든 지침을 따라야 합니다.

<br/>

### Modifier 팩토리 함수
`Modifier` 체인은 팩토리 역할을 하는 Kotlin 확장 함수로 구성된 유연한 빌더 구문을 사용하여 생성됩니다.

<br/>

예시:
```kotlin
Modifier.preferredSize(50.dp)
    .backgroundColor(Color.Blue)
    .padding(10.dp)
```

`Modifier` API는 `Modifier.Element` 인터페이스 구현 타입을 노출해서는 안 됩니다.

`Modifier` API는 다음 스타일을 따르는 팩토리 함수로 노출되어야 합니다:
```kotlin
fun Modifier.myModifier(
    param1: ...,
    paramN: ...
): Modifier = then(MyModifierImpl(param1, ... paramN))
```

<br/>

### Layout-scoped modifiers
Android의 `View` 시스템에는 `LayoutParams`라는 개념이 있습니다. 이는 `ViewGroup`의 자식 뷰와 함께 불투명하게 저장되는 객체로서 `ViewGroup`이 이를 측정하고 배치하는 데 특정한 레이아웃 지침을 제공합니다.

Compose UI `modifier`는 `ParentDataModifier` 및 수신자 스코프 객체를 사용하여 layout의 `content` 함수에 관련된 패턴을 제공합니다.

<br/>

예시:
```kotlin
@Stable
interface WeightScope {
    fun Modifier.weight(weight: Float): Modifier
}

@Composable
fun WeightedRow(
    modifier: Modifier = Modifier,
    content: @Composable WeightScope.() -> Unit
) {
    // ...
}

// 사용 사례:
WeightedRow {
    Text("Hello", Modifier.weight(1f))
    Text("World", Modifier.weight(2f))
}
```

**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**은 부모 레이아웃 composable에 특정한 `ParentDataModifier`를 제공하기 위해 scoped `modifier` 팩토리 함수를 사용해야 합니다.

<br/>

## Compose API 디자인 패턴
이 섹션에서는 Jetpack Compose API를 설계할 때 일반적인 사용 사례를 해결하기 위한 패턴을 개략적으로 설명합니다.

기본적으로 상태가 없는(stateless) `@Composable` 함수를 선호합니다. 여기서 "stateless"는 자체 상태를 유지하지 않고 호출자가 소유하고 제공하는 외부 상태 매개변수를 사용하는 `@Composable` 함수를 의미합니다. "제어 가능한(Controlled)"은 호출자가 `@Composable`에 제공되는 상태를 완전히 제어할 수 있다는 개념을 나타냅니다.

<br/>

### ✅ Do
```kotlin
@Composable
fun Checkbox(
    isChecked: Boolean,
    onToggle: () -> Unit
) {
// ...

// 사용: (호출자가 optIn을 수정하고 진실된 소스(**source of truth**)를 소유함)
Checkbox(
    myState.optIn,
    onToggle = { myState.optIn = !myState.optIn }
)
```

<br/>

### ✅ Don't
```kotlin
@Composable
fun Checkbox(
    initialValue: Boolean,
    onChecked: (Boolean) -> Unit
) {
    var checkedState by remember { mutableStateOf(initialValue) }
// ...

// 사용: (Checkbox가 체크된 상태를 소유하며 호출자가 변경 사항을 통지받음)
// 호출자는 쉽게 유효성 검사 정책을 구현할 수 없음
Checkbox(false, onToggled = { callerCheckedState = it })
```

<br/>

### state와 event 분리
Compose의 `mutableStateOf()`로 값을 보관하는 방법은 Snapshot 시스템을 통해 관찰 가능하며, 변경 사항이 발생할 때 옵저버에게 알릴 수 있습니다. 관찰 가능한 상태와 이벤트를 효과적으로 처리하려면 상태와 이벤트 간의 차이를 인지하는 것이 중요합니다.

관찰 가능한 이벤트는 특정 시점에 발생하며 폐기됩니다. 이벤트가 발생한 시점에 등록된 모든 옵저버에게 알립니다. stream의 각 이벤트는 관련이 있으며 서로 연속적으로 발생할 수 있습니다. 따라서 중복되는 이벤트는 의미가 있으며, 등록된 옵저버는 이벤트를 건너뛰지 않고 모든 이벤트를 관찰해야 합니다.

관찰 가능한 상태는 상태가 한 값에서 새로운 값으로 변경될 때 변경 이벤트를 발생시킵니다. 상태 변경 이벤트는 중복될 수 없으며, 최신 상태만 중요하게 여깁니다. 상태 변경 이벤트의 옵저버는 멱등성(idempotent)입니다. 즉, 동일한 상태 값을 주어지면 옵저버는 동일한 결과를 생성해야 합니다. 상태 옵저버는 중간 상태를 건너뛸 수도 있으며, 동일한 상태에 대해 여러 번 실행될 수도 있으며 결과는 동일해야 합니다.

Compose는 입력으로 상태를 사용하는 방식으로 작동합니다. Composable 함수는 상태 옵저버이며, 함수 매개변수와 실행 중에 읽은 모든 
 `mutableStateOf()` 값을 입력으로 사용합니다.

<br/>

### 호이스팅된 상태(Hoisted state) 타입
상태가 많아지면 상태가 없는 매개변수와 여러 이벤트 콜백 매개변수로 구성된 함수의 매개변수 목록이 불편한 정도로 커질 수 있습니다. 이러한 경우 상태와 콜백을 하나의 인터페이스로 묶어서 호출자가 일관된 정책을 객체 단위로 제공할 수 있도록 할 수 있습니다.

<br/>

### Before
```kotlin
@Composable
fun VerticalScroller(
    scrollPosition: Int,
    scrollRange: Int,
    onScrollPositionChange: (Int) -> Unit,
    onScrollRangeChange: (Int) -> Unit
) {
```

<br/>

### After
```kotlin
@Stable
interface VerticalScrollerState {
    var scrollPosition: Int
    var scrollRange: Int
}

@Composable
fun VerticalScroller(
    verticalScrollerState: VerticalScrollerState
) {
```
위의 예시에서 `VerticalScrollerState`의 구현은 관련된 `var` 속성의 `get/set` 동작을 사용하여 정책을 적용하거나 상태 자체의 저장소를 다른 곳에서 처리할 수 있습니다.

**Jetpack Compose 프레임워크와 라이브러리 개발**은 관련된 정책을 수집하고 그룹화하기 위해 hoisted 상태 타입을 선언해야 합니다. 위의 `VerticalScrollerState` 예제는 `scrollPosition`과 `scrollRange` 속성 간의 의존성을 보여줍니다. 따라서 이러한 상태 객체는 모두 함께 처리되어야 합니다.

**Jetpack Compose 프레임워크와 라이브러리 개발**은 `@Stable`로 선언된 hoisted 상태 타입을 사용하고 `@Stable` 계약을 올바르게 구현해야 합니다.

**Jetpack Compose 프레임워크와 라이브러리 개발**은 해당 composable 함수에 특정한 hoisted 상태 타입의 이름에 "**State**" 접미사를 붙여야 합니다.

기본 정책은 hoisted 상태 객체를 통해 제공하는 것입니다.

커스텀 구현 또는 외부 소유에서 이러한 정책이 객체에 필요하지 않을 수도 있습니다. Kotlin의 기본 인자, Compose의 `remember {}` API 및 Kotlin "extension constructor(익스텐션 생성자)" 패턴을 사용하여 API는 간단한 사용에 대해 기본 상태 처리 정책을 제공하면서 필요한 경우 더 정교한 사용을 허용할 수 있습니다.

<br/>

예시:
```kotlin
fun VerticalScrollerState(): VerticalScrollerState = 
    VerticalScrollerStateImpl()

private class VerticalScrollerStateImpl(
    scrollPosition: Int = 0,
    scrollRange: Int = 0
) : VerticalScrollerState {
    private var _scrollPosition by
        mutableStateOf(scrollPosition, structuralEqualityPolicy())

    override var scrollPosition: Int
        get() = _scrollPosition
        set(value) {
            _scrollPosition = value.coerceIn(0, scrollRange)
        }

    private var _scrollRange by
        mutableStateOf(scrollRange, structuralEqualityPolicy())

    override var scrollRange: Int
        get() = _scrollRange
        set(value) {
            require(value >= 0) { "$value must be > 0" }
            _scrollRange = value
            scrollPosition = scrollPosition
        }
}

@Composable
fun VerticalScroller(
    verticalScrollerState: VerticalScrollerState =
        remember { VerticalScrollerState() }
) {
```

**Jetpack Compose 프레임워크와 라이브러리 개발**은 hoisted 상태 타입을 `final` 클래스로 선언하지 않은 경우 `abstract` 또는 `open` 클래스 대신 `interface`로 선언해야 합니다.

이러한 용도로 `open` 또는 `abstract` 클래스를 설계할 때 내부 일관성을 위해 상태 동기화에 대한 숨겨진 요구 사항을 만들기 쉽습니다. 이 요구 사항은 확장하는 개발자가 유지하기 어려울(혹은 불가능하거나) 수도 있습니다. 인터페이스를 사용하면 자유롭게 구현할 수 있으며, Kotlin의 `internal`-스코프 프로퍼티나 함수를 통해 composable 함수와 hoisted 상태 객체 간의 `private` 계약을 강력하게 제한합니다.


**Jetpack Compose 프레임워크와 라이브러리 개발**은 기본 인자로 default state로 구현된 remember한 값을 제공해야 합니다. `State` 객체는 composable 함수가 호출자에 의해 구성되지 않아서 동작하지 않을 경우 필요한 기본 매개변수일 수 있습니다.

**Jetpack Compose 프레임워크와 라이브러리 개발**은 `null`을 composable 함수가 내부적으로 자체 상태를 `remember {}`해야 함을 나타내는 표식으로 사용해서는 안 됩니다. 이는 호출자에게 `null`이 의미 있는 해석이 될 수 있고 실수로 composable 함수에 제공된 경우 우연히 일관성이 없거나 예상치 못한 동작을 생성할 수 있습니다.

<br/>

### ✅ Do
```kotlin
@Composable
fun VerticalScroller(
    verticalScrollerState: VerticalScrollerState =
        remember { VerticalScrollerState() }
) {
```

<br/>

### ❌ Don't
```kotlin
// 입력 매개변수가 null과 non-null 값 사이에서 변경되는 경우 null을 기본값으로 사용하는 것은 예상치 못한 동작을 일으킬 수 있습니다.
@Composable
fun VerticalScroller(
    verticalScrollerState: VerticalScrollerState? = null
) {
    val realState = verticalScrollerState ?:
        remember { VerticalScrollerState() }
```

<br/>

### hoisted state 타입의 확장성
호이스팅된 상태 타입은 종종 정책과 유효성 검사를 구현하여 해당 타입을 허용하는 composable 함수의 동작에 영향을 미칩니다. 특히 구체적이고 최종적인 호이스팅된 상태 유형은 상태 객체가 속한 데이터의 소유권과 소스를 나타냅니다.

극단적인 경우, 이로 인해 반응형 UI API 디자인의 장점을 상실할 수 있으며, 여러가지 진실의 소스(source of truth)을 만들어 내며, 앱 코드가 여러 객체 간의 데이터 동기화를 필요로 합니다. 다음을 상황을 고려해 보세요:
```kotlin
// 다른 팀이나 라이브러리에서 구현함.
data class PersonData(val name: String, val avatarUrl: String)

class FooState {
    val currentPersonData: PersonData

    fun setPersonName(name: String)
    fun setPersonAvatarUrl(url: String)
}

// UI 레이어에 또 다른 팀에서 정의함.
class BarState {
    var name: String
    var avatarUrl: String
}

@Composable
fun Bar(barState: BarState) {
```
이러한 API는 함께 사용하기 어렵습니다. 왜냐하면 `FooState와` `BarState` 클래스 모두 해당 데이터의 진실의 소스(source of truth)이 되고자 합니다. 종종 다른 팀, 라이브러리 또는 모듈에서 공유해야 하는 데이터에 대해 단일한 통일화된 타입으로 합의하는 것이 불가능한 경우가 많습니다. 이러한 디자인은 앱 개발자가 잠재적으로 오류가 발생할 수 있는 데이터 동기화를 해야하는 요구사항을 만들어내게 됩니다.

더 유연한 접근 방법은 이러한 호이스팅된 상태 타입을 인터페이스로 정의하는 것입니다. 이를 통해 통합 개발자가 하나를 다른 것으로 정의하거나 둘 다 세 번째 타입으로 정의할 수 있으며, 시스템의 상태 관리에서 단일화된 진실의 소스(source of truth)를 보존할 수 있습니다:
```kotlin
@Stable
interface FooState {
    val currentPersonData: PersonData

    fun setPersonName(name: String)
    fun setPersonAvatarUrl(url: String)
}

@Stable
interface BarState {
    var name: String
    var avatarUrl: String
}

class MyState(
    name: String,
    avatarUrl: String
) : FooState, BarState {
    override var name by mutableStateOf(name)
    override var avatarUrl by mutableStateOf(avatarUrl)

    override val currentPersonData: PersonData =
        PersonData(name, avatarUrl)

    override fun setPersonName(name: String) {
        this.name = name
    }

    override fun setPersonAvatarUrl(url: String) {
        this.avatarUrl = url
    }
}
```
**Jetpack Compose 프레임워크 및 라이브러리 개발**은 커스텀 구현을 허용하기 위해 호이스팅된 상태 타입을 인터페이스로 선언해야 합니다. 추가적인 표준 정책 강제화가 필요한 경우 `abstract` 클래스를 고려하십시오.

**Jetpack Compose 프레임워크 및 라이브러리 개발**은 동일한 이름을 가진 호이스팅된 상태 타입의 기본 구현을 위한 팩토리 함수를 제공해야 합니다. 이는 구체적인 타입과 동일한 간단한 API를 소비자에게 제공합니다. 예시:
```kotlin
@Stable
interface FooState {
    // ...
}

fun FooState(): FooState = FooStateImpl(...)

private class FooStateImpl(...) : FooState {
    // ...
}

// 사용
val state = remember { FooState() }
```

<br/>

**앱 개발**은 `interface`에 의해 제공되는 추상화가 필요하다는 것이 증명될 때까지 더 단순한 구체적인 타입을 선호해야 합니다. 인터페이스가 필요한 경우 위에 설명한 대로 기본 구현을 위한 팩토리 함수를 추가하는 것은 소스-호환이 가능한 수정이므로 사용하는 부분을 리팩토링할 필요가 없습니다.
