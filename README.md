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

### Do
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

### Don't
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

### Do
```kotlin
// 이 함수는 PascalCased 명사로 시각적 UI 요소를 설명합니다
@Composable
fun FancyButton(text: String, onClick: () -> Unit) {
```

<br/>

### Do
```kotlin
// 이 함수는 PascalCased 명사로 composition에 존재하는 비시각적 요소를 설명합니다.
@Composable
fun BackButtonHandler(onBackPressed: () -> Unit) {
```

<br/>

### Don't
```kotlin
// 이 함수는 명사이지만 PascalCase가 아닙니다!
@Composable
fun fancyButton(text: String, onClick: () -> Unit) {
```

<br/>

### Don't
```kotlin
// 이 함수는 PascalCased이지만 명사가 아닙니다!
@Composable
fun RenderFancyButton(text: String, onClick: () -> Unit) {
```

<br/>

### Don't
```kotlin
// 이 함수는 PascalCase도 아니고 명사도 아닙니다!
@Composable
fun drawProfileImage(image: ImageAsset) {
```

<br/>

## 값을 리턴하는 @Composable 함수 명명
**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**은 `Unit`이 아닌 값을 리턴하는 `@Composable`에 어노테이션이 달린 함수의 이름을 지정하기 위해 표준 Kotlin 코딩 규칙을 따라야 합니다.

<br/>

**Jetpack Compose 프레임워크 개발 및 라이브러리 개발**은 함수의 추상적 리턴 타입과 매치되는 `PascalCase`로 명명된 어노테이션이 달린 함수의 이름을 지정하기 위해 Kotlin Coding Conventions의 팩토리 함수 규약을 사용해서는 안 됩니다.

<br/>

### Why?
이 팩토리 함수 규약은 `@Composable` 함수 이외에서 유용하고 허용되지만 `@Composable` 함수와 함께 사용할 때 호출자에게 부적절한 기대치를 설정하는 단점이 있습니다.

팩토리 함수를 `@Composable`로 표시하는 주요 예시로는, composition을 사용하여 객체의 라이프사이클을 설정하거나 `CompositionLocals`를 객체의 구성에 대한 입력으로 사용하는 경우가 있습니다. 전자는 recomposition 전반에 걸쳐 객체 인스턴스를 캐시하고 유지 관리하기 위해 Compose의 `remember {}` API를 사용하는 것을 의미하며, 이는 생성자 호출처럼 읽어들이는 팩토리 작업에 대한 호출자의 예상을 깨뜨릴 수 있습니다.(다음 섹션 참조) 후자의 동기는 팩토리 함수 이름으로 표현되어야 하는 보이지 않는 입력을 의미합니다.

또한 선언적 구체로서 `@Composable` 함수를 리턴하는 단위의 멘탈 모델은 "가상 DOM" 멘탈 모델과 혼동되어서는 안 됩니다. `PascalCase` 명사로 명명된 `@Composable` 함수에서 값을 리턴하는 것은 이러한 혼란을 조장하고, 호이스팅된 상태 객체로 더 잘 표현할 수 있으며 현재 UI 엔티티에 대해 상태 저장을 제어하는 surface를 리턴하는 바람직하지 않은 스타일을 촉진할 수 있습니다.

상태 호이스팅 패턴에 대한 자세한 내용은 이 문서의 디자인 패턴 섹션을 참조하십시오.

<br/>

### Do
```kotlin
// 현재 CompositionLocal 설정을 기준으로 스타일을 반환합니다. 이 함수는 해당 값의 출처를 한정합니다.
@Composable
fun defaultStyle(): Style {
```

<br/>

### Don't
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

### Do
```kotlin
// 이 호출이 composition을 떠날 때 취소될 CoroutineScope를 리턴합니다.
// 이 함수 앞에는 동작을 설명하기 위해 rememeber 접두사가 앞에 붙습니다.
@Composable
fun rememberCoroutineScope(): CoroutineScope {
```

<br/>

### Don't
```kotlin
// 이 호출이 composition을 떠날 때 취소될 CoroutineScope를 리턴합니다.
// 이 함수의 이름으로는 자동으로 취소되는 동작을 예측할 수 없습니다!
@Composable
fun createCoroutineScope(): CoroutineScope {
```
객체를 리턴하는 것만으로는 함수를 팩토리 함수로 간주하기에 충분하지 않습니다. 이는 함수의 주요 목적이어야 합니다. `Flow<T>.collectAsState()` 같은 `@Composable` 함수를 생각해보세요. 이 함수의 주요 목적은 flow에 대한 구독을 설정하는 것이고, 반환된 `State<T>` 객체를 `remember {}`로 만드는 것은 부수적인 것입니다.

<br/>

Naming CompositionLocals
A CompositionLocal is a key into a composition-scoped key-value table. CompositionLocals may be used to provide global-like values to a specific subtree of composition.

Jetpack Compose framework development and Library development MUST NOT name CompositionLocal keys using “CompositionLocal” or “Local” as a noun suffix. CompositionLocal keys should bear a descriptive name based on their value.

Jetpack Compose framework development and Library development MAY use “Local” as a prefix for a CompositionLocal key name if no other, more descriptive name is suitable.

Do
// "Local" is used here as an adjective, "Theme" is the noun.
val LocalTheme = staticCompositionLocalOf<Theme>()
Don't
// "Local" is used here as a noun!
val ThemeLocal = staticCompositionLocalOf<Theme>()
Stable types
The Compose runtime exposes two annotations that may be used to mark a type or function as stable - safe for optimization by the Compose compiler plugin such that the Compose runtime may skip calls to functions that accept only safe types because their results cannot change unless their inputs change.

The Compose compiler plugin may infer these properties of a type automatically, but interfaces and other types for which stability can not be inferred, only promised, may be explicitly annotated. Collectively these types are called, “stable types.”

@Immutable indicates a type where the value of any properties will never change after the object is constructed, and all methods are referentially transparent. All Kotlin types that may be used in a const expression (primitive types and Strings) are considered @Immutable.

@Stable when applied to a type indicates a type that is mutable, but the Compose runtime will be notified if and when any public properties or method behavior would yield different results from a previous invocation. (In practice this notification is backed by the Snapshot system via @Stable MutableState objects returned by mutableStateOf().) Such a type may only back its properties using other @Stable or @Immutable types.

Jetpack Compose framework development, Library development and App development MUST ensure in custom implementations of .equals() for @Stable types that for any two references a and b of @Stable type T, a.equals(b) MUST always return the same value. This implies that any future changes to a must also be reflected in b and vice versa.

This constraint is always met implicitly if a === b; the default reference equality implementation of .equals() for objects is always a correct implementation of this contract.

Jetpack Compose framework development and Library development SHOULD correctly annotate @Stable and @Immutable types that they expose as part of their public API.

Jetpack Compose framework development and Library development MUST NOT remove the @Stable or @Immutable annotation from a type if it was declared with one of these annotations in a previous stable release.

Jetpack Compose framework development and Library development MUST NOT add the @Stable or @Immutable annotation to an existing non-final type that was available in a previous stable release without this annotation.

Why?
@Stable and @Immutable are behavioral contracts that impact the binary compatibility of code generated by the Compose compiler plugin. Libraries should not declare more restrictive contracts for preexisting non-final types that existing implementations in the wild may not correctly implement, and similarly they may not declare that a library type no longer obeys a previously declared contract that existing code may depend upon.

Implementing the stable contract incorrectly for a type annotated as @Stable or @Immutable will result in incorrect behavior for @Composable functions that accept that type as a parameter or receiver.

Emit XOR return a value
@Composable functions should either emit content into the composition or return a value, but not both. If a composable should offer additional control surfaces to its caller, those control surfaces or callbacks should be provided as parameters to the composable function by the caller.

Jetpack Compose framework development and Library development MUST NOT expose any single @Composable function that both emits tree nodes and returns a value.

Why
Emit operations must occur in the order the content is to appear in the composition. Using return values to communicate with the caller restricts the shape of calling code and prevents interactions with other declarative calls that come before it.

Do
// Emits a text input field element that will call into the inputState
// interface object to request changes
@Composable
fun InputField(inputState: InputState) {
// ...

// Communicating with the input field is not order-dependent
val inputState = remember { InputState() }

Button("Clear input", onClick = { inputState.clear() })

InputField(inputState)
Don't
// Emits a text input field element and returns an input value holder
@Composable
fun InputField(): UserInputState {
// ...

// Communicating with the InputField is made difficult
Button("Clear input", onClick = { TODO("???") })
val inputState = InputField()
Communicating with a composable by passing parameters forward affords aggregation of several such parameters into types used as parameters to their callers:

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
For more information on this pattern, see the sections on hoisted state types in the Compose API design patterns section below.

Compose UI API structure
Compose UI is a UI toolkit built on the Compose runtime. This section outlines guidelines for APIs that use and extend the Compose UI toolkit.

Compose UI elements
A @Composable function that emits exactly one Compose UI tree node is called an element.

Example:

@Composable
fun SimpleLabel(
    text: String,
    modifier: Modifier = Modifier
) {
Jetpack Compose framework development and Library development MUST follow all guidelines in this section.

Jetpack Compose app development SHOULD follow all guidelines in this section.

Elements return Unit
Elements MUST emit their root UI node either directly by calling emit() or by calling another Compose UI element function. They MUST NOT return a value. All behavior of the element not available from the state of the composition MUST be provided by parameters passed to the element function.

Why?
Elements are declarative entities in a Compose UI composition. Their presence or absence in the composition determines whether they appear in the resulting UI. Returning a value is not necessary; any means of controlling the emitted element should be provided as a parameter to the element function, not returned by calling the element function. See the, “hoisted state” section in the Compose API design patterns section of this document for more information.

Do
@Composable
fun FancyButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier
) {
Don't
interface ButtonState {
    val clicks: Flow<ClickEvent>
    val measuredSize: Size
}

@Composable
fun FancyButton(
    text: String,
    modifier: Modifier = Modifier
): ButtonState {
Elements accept and respect a Modifier parameter
Element functions MUST accept a parameter of type Modifier. This parameter MUST be named “modifier” and MUST appear as the first optional parameter in the element function's parameter list. Element functions MUST NOT accept multiple Modifier parameters.

If the element function‘s content has a natural minimum size - that is, if it would ever measure with a non-zero size given constraints of minWidth and minHeight of zero - the default value of the modifier parameter MUST be Modifier - the Modifier type’s companion object that represents the empty Modifier. Element functions without a measurable content size (e.g. Canvas, which draws arbitrary user content in the size available) MAY require the modifier parameter and omit the default value.

Element functions MUST provide their modifier parameter to the Compose UI node they emit by passing it to the root element function they call. If the element function directly emits a Compose UI layout node, the modifier MUST be provided to the node.

Element functions MAY concatenate additional modifiers to the end of the received modifier parameter before passing the concatenated modifier chain to the Compose UI node they emit.

Element functions MUST NOT concatenate additional modifiers to the beginning of the received modifier parameter before passing the concatenated modifier chain to the Compose UI node they emit.

Why?
Modifiers are the standard means of adding external behavior to an element in Compose UI and allow common behavior to be factored out of individual or base element API surfaces. This allows element APIs to be smaller and more focused, as modifiers are used to decorate those elements with standard behavior.

An element function that does not accept a modifier in this standard way does not permit this decoration and motivates consuming code to wrap a call to the element function in an additional Compose UI layout such that the desired modifier can be applied to the wrapper layout instead. This does not prevent the developer behavior of modifying the element, and forces them to write more inefficient UI code with a deeper tree structure to achieve their desired result.

Modifiers occupy the first optional parameter slot to set a consistent expectation for developers that they can always provide a modifier as the final positional parameter to an element call for any given element's common case.

See the Compose UI modifiers section below for more details.

Do
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
Compose UI layouts
A Compose UI element that accepts one or more @Composable function parameters is called a layout.

Example:

@Composable
fun SimpleRow(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
Jetpack Compose framework development and Library development MUST follow all guidelines in this section.

Jetpack Compose app development SHOULD follow all guidelines in this section.

Layout functions SHOULD use the name “content” for a @Composable function parameter if they accept only one @Composable function parameter.

Layout functions SHOULD use the name “content” for their primary or most common @Composable function parameter if they accept more than one @Composable function parameter.

Layout functions SHOULD place their primary or most common @Composable function parameter in the last parameter position to permit the use of Kotlin's trailing lambda syntax for that parameter.

Compose UI modifiers
A Modifier is an immutable, ordered collection of objects that implement the Modifier.Element interface. Modifiers are universal decorators for Compose UI elements that may be used to implement and add cross-cutting behavior to elements in an opaque and encapsulated manner. Examples of modifiers include altering element sizing and padding, drawing content beneath or overlapping the element, or listening to touch events within the UI element's bounding box.

Jetpack Compose framework development and Library development MUST follow all guidelines in this section.

Modifier factory functions
Modifier chains are constructed using a fluent builder syntax expressed as Kotlin extension functions that act as factories.

Example:

Modifier.preferredSize(50.dp)
    .backgroundColor(Color.Blue)
    .padding(10.dp)
Modifier APIs MUST NOT expose their Modifier.Element interface implementation types.

Modifier APIs MUST be exposed as factory functions following this style:

fun Modifier.myModifier(
    param1: ...,
    paramN: ...
): Modifier = then(MyModifierImpl(param1, ... paramN))
Layout-scoped modifiers
Android‘s View system has the concept of LayoutParams - a type of object stored opaquely with a ViewGroup’s child view that provides layout instructions specific to the ViewGroup that will measure and position it.

Compose UI modifiers afford a related pattern using ParentDataModifier and receiver scope objects for layout content functions:

Example
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

// Usage:
WeightedRow {
    Text("Hello", Modifier.weight(1f))
    Text("World", Modifier.weight(2f))
}
Jetpack Compose framework development and library development SHOULD use scoped modifier factory functions to provide parent data modifiers specific to a parent layout composable.

Compose API design patterns
This section outlines patterns for addressing common use cases when designing a Jetpack Compose API.

Prefer stateless and controlled @Composable functions
In this context, “stateless” refers to @Composable functions that retain no state of their own, but instead accept external state parameters that are owned and provided by the caller. “Controlled” refers to the idea that the caller has full control over the state provided to the composable.

Do
@Composable
fun Checkbox(
    isChecked: Boolean,
    onToggle: () -> Unit
) {
// ...

// Usage: (caller mutates optIn and owns the source of truth)
Checkbox(
    myState.optIn,
    onToggle = { myState.optIn = !myState.optIn }
)
Don't
@Composable
fun Checkbox(
    initialValue: Boolean,
    onChecked: (Boolean) -> Unit
) {
    var checkedState by remember { mutableStateOf(initialValue) }
// ...

// Usage: (Checkbox owns the checked state, caller notified of changes)
// Caller cannot easily implement a validation policy.
Checkbox(false, onToggled = { callerCheckedState = it })
Separate state and events
Compose's mutableStateOf() value holders are observable through the Snapshot system and can notify observers of changes. This is the primary mechanism for requesting recomposition, relayout, or redraw of a Compose UI. Working effectively with observable state requires acknowledging the distinction between state and events.

An observable event happens at a point in time and is discarded. All registered observers at the time the event occurred are notified. All individual events in a stream are assumed to be relevant and may build on one another; repeated equal events have meaning and therefore a registered observer must observe all events without skipping.

Observable state raises change events when the state changes from one value to a new, unequal value. State change events are conflated; only the most recent state matters. Observers of state changes must therefore be idempotent; given the same state value the observer should produce the same result. It is valid for a state observer to both skip intermediate states as well as run multiple times for the same state and the result should be the same.

Compose operates on state as input, not events. Composable functions are state observers where both the function parameters and any mutableStateOf() value holders that are read during execution are inputs.

Hoisted state types
A pattern of stateless parameters and multiple event callback parameters will eventually reach a point of scale where it becomes unwieldy. As a composable function's parameter list grows it may become appropriate to factor a collection of state and callbacks into an interface, allowing a caller to provide a cohesive policy object as a unit.

Before
@Composable
fun VerticalScroller(
    scrollPosition: Int,
    scrollRange: Int,
    onScrollPositionChange: (Int) -> Unit,
    onScrollRangeChange: (Int) -> Unit
) {
After
@Stable
interface VerticalScrollerState {
    var scrollPosition: Int
    var scrollRange: Int
}

@Composable
fun VerticalScroller(
    verticalScrollerState: VerticalScrollerState
) {
In the example above, an implementation of VerticalScrollerState is able to use custom get/set behaviors of the related var properties to apply policy or delegate storage of the state itself elsewhere.

Jetpack Compose framework and Library development SHOULD declare hoisted state types for collecting and grouping interrelated policy. The VerticalScrollerState example above illustrates such a dependency between the scrollPosition and scrollRange properties; to maintain internal consistency such a state object should clamp scrollPosition into the valid range during set attempts. (Or otherwise report an error.) These properties should be grouped as handling their consistency involves handling all of them together.

Jetpack Compose framework and Library development SHOULD declare hoisted state types as @Stable and correctly implement the @Stable contract.

Jetpack Compose framework and Library development SHOULD name hoisted state types that are specific to a given composable function as the composable function's name suffixed by, “State”.

Default policies through hoisted state objects
Custom implementations or even external ownership of these policy objects are often not required. By using Kotlin‘s default arguments, Compose’s remember {} API, and the Kotlin “extension constructor” pattern, an API can provide a default state handling policy for simple usage while permitting more sophisticated usage when desired.

Example:
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
Jetpack Compose framework and Library development SHOULD declare hoisted state types as interfaces instead of abstract or open classes if they are not declared as final classes.

When designing an open or abstract class to be properly extensible for these use cases it is easy to create hidden requirements of state synchronization for internal consistency that are difficult (or impossible) for an extending developer to preserve. Using an interface that can be freely implemented strongly discourages private contracts between composable functions and hoisted state objects by way of Kotlin internal-scoped properties or functionality.

Jetpack Compose framework and Library development SHOULD provide default state implementations remembered as default arguments. State objects MAY be required parameters if the composable cannot function if the state object is not configured by the caller.

Jetpack Compose framework and Library development MUST NOT use null as a sentinel indicating that the composable function should internally remember {} its own state. This can create accidental inconsistent or unexpected behavior if null has a meaningful interpretation for the caller and is provided to the composable function by mistake.

Do
@Composable
fun VerticalScroller(
    verticalScrollerState: VerticalScrollerState =
        remember { VerticalScrollerState() }
) {
Don't
// Null as a default can cause unexpected behavior if the input parameter
// changes between null and non-null.
@Composable
fun VerticalScroller(
    verticalScrollerState: VerticalScrollerState? = null
) {
    val realState = verticalScrollerState ?:
        remember { VerticalScrollerState() }
Extensibility of hoisted state types
Hoisted state types often implement policy and validation that impact behavior for a composable function that accepts it. Concrete and especially final hoisted state types imply containment and ownership of the source of truth that the state object appeals to.

In extreme cases this can defeat the benefits of reactive UI API designs by creating multiple sources of truth, necessitating app code to synchronize data across multiple objects. Consider the following:

// Defined by another team or library
data class PersonData(val name: String, val avatarUrl: String)

class FooState {
    val currentPersonData: PersonData

    fun setPersonName(name: String)
    fun setPersonAvatarUrl(url: String)
}

// Defined by the UI layer, by yet another team
class BarState {
    var name: String
    var avatarUrl: String
}

@Composable
fun Bar(barState: BarState) {
These APIs are difficult to use together because both the FooState and BarState classes want to be the source of truth for the data they represent. It is often the case that different teams, libraries, or modules do not have the option of agreeing on a single unified type for data that must be shared across systems. These designs combine to form a requirement for potentially error-prone data syncing on the part of the app developer.

A more flexible approach defines both of these hoisted state types as interfaces, permitting the integrating developer to define one in terms of the other, or both in terms of a third type, preserving single source of truth in their system's state management:

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
Jetpack Compose framework and Library development SHOULD declare hoisted state types as interfaces to permit custom implementations. If additional standard policy enforcement is necessary, consider an abstract class.

Jetpack Compose framework and Library development SHOULD offer a factory function for a default implementation of hoisted state types sharing the same name as the type. This preserves the same simple API for consumers as a concrete type. Example:

@Stable
interface FooState {
    // ...
}

fun FooState(): FooState = FooStateImpl(...)

private class FooStateImpl(...) : FooState {
    // ...
}

// Usage
val state = remember { FooState() }
App development SHOULD prefer simpler concrete types until the abstraction provided by an interface proves necessary. When it does, adding a factory function for a default implementation as outlined above is a source-compatible change that does not require refactoring of usage sites.

Powered by Gitiles| Privacy| Terms
