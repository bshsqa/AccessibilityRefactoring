# Phase 5 - toolkit/adaptor 책임 재배치

## 목적

Toolkit/UI와 adaptor 사이의 책임을 다시 나눈다. semantic 정보는 Toolkit/UI가 제공하고, platform bridge와 외부 통신 정책은 adaptor가 담당하도록 한다.

## Toolkit/UI에 남길 것

- Control/View의 name, description, role, value, state 산출.
- component별 action 의미.
- text, value, selection 등 control 내부 정보.
- app-facing AccessibilityData API.
- component developer용 delegate/override hook.

## Adaptor로 옮길 것

- DBus/AT-SPI bridge 구현.
- accessibility object registry.
- window/application bridge.
- external event emit.
- navigation/hit-test 중 platform 좌표와 bridge 정책에 가까운 부분.
- highlight plane rendering 정책.
- diagnostic/export 기능.

## 경계 원칙

Adaptor가 Toolkit control의 구체 타입을 많이 알아야 하면 경계가 깨진다. Toolkit은 semantic provider를 제공하고, adaptor는 그것을 외부 protocol로 변환하는 방향이 좋다.

## 핵심 변경 대상

현재 Toolkit/UI는 `RegisterExternalAccessibleGetter()`를 통해 `SharedPtr<Accessibility::Accessible>`을 만들어 adaptor에 넘긴다. 또한 `ControlAccessible`/`ViewAccessible`이 adaptor의 `ActorAccessible`을 상속하므로 Toolkit/UI가 `Accessible`, `ActorAccessible`, AT-SPI `States` 같은 adaptor 타입을 직접 알아야 한다.

Phase 5의 목표는 이 contract를 바꾸는 것이다.

```text
현재:
  adaptor asks Toolkit/UI for Accessible
  Toolkit/UI creates ControlAccessible/ViewAccessible
  Toolkit/UI depends on adaptor Accessible model

목표:
  adaptor asks Toolkit/UI for accessibility semantics provider
  Toolkit/UI provides name/role/state/value/action semantics
  adaptor creates and owns Accessible adapter
```

이 구조에서는 부모-자식 관계와 actor tree 변화는 adaptor의 Accessible adapter가 Actor child signal을 관찰해서 처리한다. Toolkit/UI는 name, role, value, state, action 가능 여부 같은 semantic 변화만 backend-neutral signal로 알려준다.

Adaptor의 Accessible adapter는 semantics provider를 읽고 `SemanticsChanged`류 signal을 구독하는 observer 역할을 한다. Semantics provider가 adaptor나 Accessible 객체를 직접 알 필요는 없다.

Accessible adapter 생성은 lazy하게 유지할 수 있다. 예를 들어 accessibility bridge가 up 되었거나 외부 client가 tree를 조회할 때 adaptor가 Control/View의 semantics provider를 얻고, 그때 Accessible adapter를 생성해 registry에 등록한다.

현재 코드에도 accessibility 활성 상태 개념은 있다. `org.a11y.Status`의 `IsEnabled` 또는 `ScreenReaderEnabled`가 켜지고 application이 running이면 bridge가 `ForceUp()`되고, 꺼지면 `ForceDown()`된다. 향후 semantics materialization과 Accessible adapter 생성 시점은 이 bridge up/down lifecycle과 맞추는 것이 좋다.

Phase 1에서 `dali-csharp-binder` 호환을 위해 `integration-api` wrapper를 두더라도, 이는 이행용 경로로 본다. Phase 5에서는 binder 내부 구현도 가능하면 semantics/provider contract를 사용하도록 옮기고, binder가 직접 `ControlAccessible`, `ActorAccessible`, AT-SPI feature interface를 상속하거나 호출하는 지점을 줄인다. 단, binder가 외부에 제공하는 C ABI는 유지한다.

## 완료 기준

- Toolkit/UI는 AT-SPI bridge 세부를 직접 include하지 않는다.
- Adaptor는 Toolkit의 semantic provider contract만 사용한다.
- bridge/platform 정책과 UI semantic 산출 책임이 분리된다.
- `RegisterExternalAccessibleGetter()` 또는 그 대체 contract가 `Accessible` 객체 반환이 아니라 semantic provider 조회로 바뀐다.
- `dali-csharp-binder`의 C ABI는 유지하되, 내부 DALi accessibility 접근은 새 contract로 이행할 수 있다.
