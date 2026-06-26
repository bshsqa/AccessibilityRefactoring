# Prepare Phase 1 - dali-adaptor API Minimization

## 목적

Phase 0에서 정리한 API 경계 기준을 Phase 1의 실제 작업 기준으로 바꾼다. Phase 1은 `dali-adaptor`와 `dali-toolkit`/`dali-ui` 사이 accessibility interface를 최대한 얇게 만드는 첫 구현 단계다.

이 단계는 Toolkit/UI가 가진 큰 로직 책임을 adaptor로 옮기는 단계는 아니다. 하지만 현재 ownership 구조를 유지한 상태에서 함수, helper type, 짧은 wrapper/class 이동만으로 줄일 수 있는 interface는 최대한 줄인다.

## Phase 0 결과 요약

현재 문제는 `dali-adaptor devel-api`에 다음 성격의 API가 섞여 있다는 점이다.

- App 또는 framework가 제한적으로 직접 쓸 수 있는 안정 API.
- Toolkit/UI가 adaptor와 연동하기 위해 필요한 contract.
- DBus/AT-SPI bridge 내부 구현 타입.
- dump/debug/test/automation 보조 API.

Phase 1에서는 이들을 먼저 위치 기준으로 나누되, 내부 ownership 구조는 바꾸지 않는다. 핵심은 Toolkit/UI가 adaptor의 helper type이나 bridge 내부 type을 직접 알지 않도록 줄이는 것이다.

판단 기준은 다음과 같다.

- 큰 책임 재배치가 필요한 것은 Phase 5로 넘긴다.
- 단순 type 변환, helper 은닉, forwarding wrapper, 짧은 contract 함수 추가로 줄일 수 있는 의존은 Phase 1에서 처리한다.
- `dali-csharp-binder` 때문에 필요한 API는 최종 설계가 아니라 compatibility 예외로 남긴다.

## API 분류 기준

| 분류 | 의미 | Phase 1 처리 |
|---|---|---|
| `public-api` | App/framework가 직접 써도 되는 안정 API | 최소한만 남긴다 |
| `integration-api` | Toolkit/UI 또는 binder가 adaptor와 연동하기 위한 API | 가능한 얇게 유지하고, compat wrapper는 임시로 둔다 |
| `internal` | bridge/object registry/protocol 변환 로직 | devel/public 노출을 줄인다 |
| `diagnostic` | dump/debug/test/automation 보조 기능 | 안정 API가 아니며 internal 성격으로 관리한다 |

`diagnostic`은 반드시 `internal`과 물리적으로 별도 디렉토리/API로 나눠야 하는 것은 아니다. 다만 dump/debug/test 목적의 API가 app-facing 안정 API처럼 남지 않도록 표시하기 위한 분류다. Phase 1에서는 `diagnostic/internal`로 함께 묶어도 충분하다.

## 1차 분류안

| 대상 | 1차 분류 | 비고 |
|---|---|---|
| `accessibility.h`의 `Role`, `State`, `RelationType`, `ActionInfo`, `GestureInfo`, `Range` 등 semantic type | `public-api` 후보 | AT-SPI 이름이 app-facing API에 직접 드러나는지는 추가 검토 |
| `accessibility-bitset.h` | 제거 또는 internal helper 후보 | toolkit-adaptor 경계에서는 원칙적으로 제거한다. 단순 wrapper/type 변환으로 없앨 수 있으면 Phase 1에서 처리한다. binder/compat 때문에 필요하면 임시로 `integration-api` 또는 deprecated devel wrapper에만 남긴다 |
| `atspi-accessibility.h`의 `IsEnabled`, `IsScreenReaderEnabled` | `public-api` 후보 | platform 상태 조회 성격 |
| `atspi-accessibility.h`의 `Say`, `Pause`, `Resume`, `StopReading`, `SuppressScreenReader` | 제한적 `public-api` 후보 | 실제 app/NUI 사용 여부 확인 후 결정 |
| `accessibility-bridge.h`의 `Bridge::EnabledSignal`, `ScreenReaderEnabledSignal` | `public-api` 또는 `integration-api` 후보 | binder/NUI 사용 중 |
| `Bridge::GetCurrentBridge`, `RegisterDefaultLabel`, `UnregisterDefaultLabel` | `integration-api` 후보 | app-facing API로 두지 않음 |
| `Accessible::Get`, `GetOwningPtr`, `GetHighlightActor`, `SetHighlightActor` | `integration-api` compatibility 후보 | binder/toolkit 호환 필요 |
| `ActorAccessible`, `ProxyAccessible` | `integration-api` 후보 | Phase 5 전까지 유지 |
| `atspi-interfaces/*`의 `Action`, `Text`, `Value`, `Selection`, `EditableText`, `Hypertext` 등 | `integration-api` 후보 | 현재 Toolkit/UI/binder 상속 구조 때문에 유지 |
| `Accessible::DumpTree`, `DumpDetailLevel` | `diagnostic/internal` 후보 | 안정 app API로 분류하지 않음 |
| `GetSuppressedEvents`, `AtspiEvent` | `integration-api` 또는 `diagnostic/internal` 후보 | binder 사용 여부와 목적을 보고 결정 |
| DBus serialization/object registry/bridge 구현 세부 | `internal` | public/devel 노출 대상 아님 |

## binder 영향 확인 대상

`dali-csharp-binder`는 NUI ABI 유지가 필요하므로 Phase 1에서 반드시 빌드 호환을 확인한다.

- `dali-csharp-binder/dali-adaptor/atspi-wrap.cpp`
- `dali-csharp-binder/dali-toolkit/control-devel-wrap.cpp`
- `dali-csharp-binder/legacy/nui-view-accessible.*`
- `dali-csharp-binder/common/signal-wrap.cpp`

이 파일들이 필요한 API는 `public-api`로 남기거나, `integration-api` compatibility wrapper로 제공한다. 기존 C ABI 함수 이름과 signature는 변경하지 않는다. 다만 binder 때문에 남기는 API는 최종 설계가 아니라 이행용 예외로 취급한다.

## Phase 1 작업 순서

1. `dali-adaptor/dali/devel-api/adaptor-framework/accessibility*.h`와 `dali-adaptor/dali/devel-api/atspi-interfaces/*.h`를 위 기준으로 태깅한다.
2. `public-api`로 올릴 최소 semantic type과 상태 조회 API를 확정한다.
3. Toolkit/UI가 직접 알 필요 없는 helper type을 경계에서 제거한다. 특히 `accessibility-bitset.h` 의존은 가능하면 Toolkit/UI API에서 없앤다.
4. 단순 wrapper, 변환 함수, 작은 facade class만으로 줄일 수 있는 Toolkit/UI -> adaptor 의존은 Phase 1에서 줄인다.
5. Toolkit/UI/binder가 여전히 필요한 비공개성 API는 얇은 `integration-api` compatibility wrapper로 준비한다.
6. 기존 `devel-api` include 경로는 deprecated forwarding header로 유지한다.
7. `dali-toolkit`, `dali-ui`, `dali-csharp-binder` include를 가능한 범위에서 새 위치로 옮긴다.
8. 빌드와 binder ABI 영향을 확인한다.

## Phase 1 Non-goals

Phase 1에서는 내부 ownership 구조 변경이나 큰 책임 재배치는 하지 않는다.

- `Accessible` ownership 변경 안 함.
- `RegisterExternalAccessibleGetter()` contract 변경 안 함.
- `NUIViewAccessible` 상속 구조 변경 안 함.
- AT-SPI adapter/backend 분리 안 함.
- Highlight plane 구조 변경 안 함.
- Toolkit/UI의 accessibility 로직을 대규모로 adaptor로 이전하지 않음.

위 작업은 Phase 4~6에서 진행한다. 다만 작은 함수/helper/type 이동만으로 interface를 줄일 수 있는 작업은 Phase 1 범위에 포함한다.

## Phase 1 완료 기준

- app-facing header에서 bridge 구현 세부가 줄어든다.
- Toolkit/UI가 사용하는 adaptor contract가 얇은 `integration-api`로 식별된다.
- `accessibility-bitset.h` 같은 helper type은 toolkit-adaptor 경계에서 제거되거나, compatibility 목적으로만 남는다.
- 작은 wrapper/type 변환으로 제거 가능한 Toolkit/UI -> adaptor 의존은 Phase 1에서 정리된다.
- 기존 devel include는 deprecated compatibility layer로 유지된다.
- `dali-csharp-binder` C ABI와 빌드 호환 경로가 유지된다.
- `diagnostic` 성격 API는 안정 public API가 아니라는 점이 분류표에 남는다.
