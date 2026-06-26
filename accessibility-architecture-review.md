# DALi Accessibility 구조 리뷰

## 전제

외부 프로세스가 DALi 앱과 접근성 목적으로 런타임 인터랙션하는 경로는 DBus/AT-SPI bridge로 제한되는 구조가 맞다. 앱 내부 개발자는 `dali-toolkit` 또는 `dali-ui` 계층의 API만 사용하고, `dali-adaptor`는 bridge, object registry, DBus endpoint, platform integration을 담당하는 경계가 바람직하다.

## 간략 평가

현재 구조는 방향성은 맞다. `dali-toolkit`의 `ControlAccessible`이 toolkit property와 signal을 AT-SPI `Accessible` 구현으로 변환하고, `dali-adaptor`가 DBus bridge와 기본 `ActorAccessible`을 제공하는 분리는 실용적이다.

다만 구조 품질은 중간 수준으로 보인다. 동작은 가능하지만 경계가 선명하지 않고, 접근성 관심사가 `Control`/`View` 기본 객체에 많이 스며들어 있다. 결과적으로 일반 UI 속성, focus/gesture/action, AT-SPI 노출 정책, screen reader 편의 기능이 한 곳에 섞여 보인다.

## 주요 우려

- `dali-adaptor`의 공개/내부 접근성 API가 너무 넓다. tree query, navigation, hit-test, action, gesture, highlight, reading material, dump, event emit, screen reader control이 모두 bridge 주변에 모여 있어 책임이 크다.
- Toolkit 쪽 설정 모델이 property, signal, virtual override, custom `Accessible` override로 분산되어 있다. 사용자는 같은 목적의 설정을 여러 방식 중 하나로 골라야 하며, component 개발자와 app 개발자의 책임 경계도 흐려질 수 있다.
- `Control`/`View`에 접근성 상태가 깊게 통합되어 있다. 기본 UI 객체가 accessibility storage, signal, state conversion, highlight behavior까지 직접 품고 있어 장기적으로 유지보수 비용이 커질 가능성이 있다.
- AT-SPI 호환 기능과 Tizen/DALi 확장 기능이 같은 bridge 표면에 함께 드러난다. 표준 경로와 제품 특화 확장이 구분되지 않으면 외부 클라이언트 계약이 커지고 안정화 부담도 커진다.
- navigation/filtering 정책이 adaptor에 꽤 많이 들어 있다. `hidden`, `highlightable`, `showing`, scrollable parent, collection index, spatial sort 등이 bridge에서 해석되므로 Toolkit/App이 설정한 의도가 외부 노출 결과로 바뀌는 지점이 분산된다.

## 개선 방향

- 외부 프로세스 계약은 `AT-SPI standard`, `DALi extension`, `diagnostic/internal`로 명확히 나누는 것이 좋다.
- App 개발자용 설정 API는 accessibility descriptor 또는 behavior object처럼 더 좁고 선언적인 형태로 정리하는 편이 좋다.
- `Control`/`View` 본체에는 최소한의 연결점만 두고, 접근성 저장소/정책/변환은 별도 component로 분리하는 방향이 바람직하다.
- Toolkit이 제공하는 public/devel API와 adaptor bridge API 사이의 허용된 연동면을 문서화하고, 그 외 직접 접근은 내부 구현으로 잠그는 것이 좋다.
- dump, reading material, navigation helper 같은 편의 기능은 표준 tree/property/action API와 분리해서 관리하는 편이 안정적이다.

## 결론

현재 DALi Accessibility는 기능적으로는 성숙하지만 구조적으로는 무거워지고 있다. 가장 큰 리스크는 "외부는 DBus, 앱은 Toolkit/UI API"라는 좋은 원칙이 코드 레벨에서 완전히 강제되지 않고, 여러 편의 기능이 핵심 객체와 bridge 표면에 누적되고 있다는 점이다.
