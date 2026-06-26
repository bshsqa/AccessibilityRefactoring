# DALi Accessibility 현재 인터랙션 요약

## 기본 구조

DALi Accessibility는 크게 세 계층으로 동작한다.

- Component 개발자는 `Control` 또는 `View` 계열을 만들고 접근성 role, name, state, action, custom accessible behavior를 정의한다.
- App 개발자는 `dali-toolkit` 또는 `dali-ui` API로 각 UI 객체의 접근성 속성을 설정한다.
- 외부 프로세스는 DBus/AT-SPI를 통해 `dali-adaptor` bridge와만 통신한다.

외부 프로세스가 DALi 객체에 접근하거나 조작하는 런타임 경로는 DBus/AT-SPI bridge가 중심이다. 직접 toolkit 객체를 만지는 구조가 아니다.

## Component 개발자

Component 개발자는 기본 `ControlAccessible`을 그대로 쓰거나 `CreateAccessibleObject()`를 override해서 전용 `Accessible` 객체를 만든다. 또한 `OnAccessibilityActivated()` 같은 virtual function을 override해서 action이 들어왔을 때의 동작을 정의할 수 있다.

대표 예시는 button, text field, scroll view처럼 role이나 action이 명확한 control들이다. 이들은 생성 시 `accessibilityRole`을 세팅하거나, text/value/selection 관련 AT-SPI interface를 추가로 구현한다.

## App 개발자

App 개발자는 주로 property와 signal을 사용한다.

- `accessibilityName`
- `accessibilityDescription`
- `accessibilityRole`
- `accessibilityHidden`
- `accessibilityValue`
- `accessibilityScrollable`
- `accessibilityStates`
- `automationId`
- `accessibilityAttributes`

동적 name/description이 필요하면 `getName`, `getDescription` signal을 연결할 수 있고, custom gesture 처리가 필요하면 `doGesture` signal을 받을 수 있다.

## 외부 프로세스

외부 프로세스는 DBus/AT-SPI를 통해 adaptor bridge의 interface를 호출한다.

- tree 조회: `GetChildren`, `GetChildAtIndex`, `GetParent`, `GetIndexInParent`
- property 조회: `GetName`, `GetDescription`, `GetRole`, `GetState`, `GetAttributes`
- geometry 조회: `GetExtents`, `GetPosition`, `GetSize`, `GetAccessibleAtPoint`
- navigation: `GetNavigableAtPoint`, `GetNeighbor`
- action/gesture: `DoAction`, `DoActionName`, `DoGesture`
- focus/highlight: `GrabFocus`, `GrabHighlight`, `ClearHighlight`
- diagnostic/export: `DumpTree`, `GetNodeInfo`, `GetReadingMaterial`

이 호출들은 adaptor가 현재 DBus object path에 해당하는 `Accessible`을 찾은 뒤, Toolkit의 `ControlAccessible` 또는 adaptor의 `ActorAccessible` 구현으로 위임한다.

## 추가 커뮤니케이션

DALi 앱에서 외부 접근성 시스템으로 이벤트를 내보내는 경로도 있다. name/value/state/bounds/text/highlight/scroll/window 변화가 발생하면 `ActorAccessible`이 bridge를 통해 AT-SPI event를 emit한다.

앱이 screen reader를 제어하는 API도 있다. `AtspiAccessibility::Say`, `Pause`, `Resume`, `StopReading`, `SuppressScreenReader`는 tree 조회와 별개로 앱에서 접근성 서비스 쪽으로 명령을 보내는 경로다.

WebView 같은 임베디드 컨텐츠는 별도의 accessibility address나 proxy/socket 형태로 다른 accessibility tree와 연결될 수 있다. 이 경우 DALi tree 안에 외부 accessibility world를 연결하는 bridge 역할이 추가된다.

## 요약

현재 구조는 "앱/컴포넌트는 Toolkit 또는 UI 계층을 사용하고, 외부 프로세스는 DBus/AT-SPI로 adaptor와 통신한다"는 모델을 대체로 따른다. 다만 action, gesture, highlight, screen reader control, dump/navigation helper까지 포함하면 단순 정보 조회보다 훨씬 넓은 양방향 인터랙션 모델이다.
