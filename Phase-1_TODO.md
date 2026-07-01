

지금 당장 정리 가능한 비효율
accessibility-integ.h가 accessibility-devel.h를 include하는 역방향 의존성
지금 Integration::Accessibility::AtspiInterfaces 등이 Devel::Accessibility::AtspiInterface를 쓰기 때문에 integration이 devel을 물고 있어. 이건 구조상 안 예뻐.
AtspiInterface, AtspiEvent, ReadingInfoType 같은 “AT-SPI vocabulary” enum을 integration 쪽으로 옮기고, devel에는 compatibility alias를 남기면 방향을 바로잡을 수 있어.

accessibility-devel.h가 너무 잡탕임
Address, Point, Size, Range, GestureInfo, Relation, ActionInfo, WindowEvent, TextChangedState 등이 한 파일에 섞여 있음.
지금 바로 가능한 건 파일 분리:
accessibility-types.h, accessibility-events.h, accessibility-geometry.h 같은 식. ABI 영향은 거의 없고 include 비용/의미가 좋아짐.

Roles bitset alias
object role 저장용은 아니고 collection match rule에서 “이 role들 중 하나”를 표현하려고 씀. 지금 당장 지우긴 가능하지만, 대체 set 표현이 필요해서 이득은 작아.
다만 이름은 RoleSet 같은 쪽이 훨씬 정직함.

문서/주석 잔재
아직 Dali::Accessibility::States, ArrayBitset, old namespace 설명이 섞여 있음. 빌드엔 영향 없지만 혼란을 계속 만듦. 바로 정리 가능.

추후 정리 가능한 비효율
Dali::Accessibility::* namespace에 AT-SPI interface class들이 남아 있는 것
Accessible, Component, Action, Text, Selection 등은 사실 앱 public API라기보다 adaptor bridge/devel API야.
이상적으로는 Dali::Integration::Accessibility 또는 Dali::Devel::Accessibility로 옮기는 게 맞지만, 타입 이름이 너무 많이 노출돼서 compatibility layer가 필요함.

ActorAccessible 직접 의존
toolkit/csharp-binder가 dynamic_cast<ActorAccessible*> 후 EmitStateChanged() 같은 걸 부르고 있음. 이건 Phase 1 관점에서 아직 두꺼운 결합.
나중엔 NotifyAccessibilityStateChanged, NotifyAccessibilityPresentationChanged, EmitAccessibility... 류의 narrow helper로 대체하고 ActorAccessible 직접 접근을 줄이는 게 좋음.

ReadingInfoTypes가 toolkit devel API에 integration bitset 그대로 노출됨
State bitset은 많이 줄였지만 ReadingInfoTypes는 아직 Dali::Integration::Accessibility::ReadingInfoTypes를 toolkit API가 받음.
toolkit-local enum/set 또는 raw mask wrapper로 바꿀 수 있음.

Relation이 adaptor Accessible*를 직접 들고 있음
toolkit은 toolkit relation enum을 쓰게 바꿨지만, 반환되는 Relation은 여전히 adaptor accessible pointer vector. 이건 AT-SPI bridge 내부 표현으로는 괜찮지만 toolkit API로는 무거움. 나중에 relation query를 toolkit-local로 새로 만들 수 있음.

정리 못하거나, 정리하면 안 되는 비효율
State bitset 자체
AT-SPI state는 하나가 아니라 set임. SHOWING + VISIBLE + ENABLED + FOCUSABLE + CHECKED처럼 동시에 붙는 모델이라 set 표현은 필요함.

AT-SPI interface class들의 다수성
Accessible, Action, Component, Text, Value, Selection 등으로 쪼개진 건 DALi 취향이 아니라 AT-SPI 프로토콜 구조를 반영한 것이라 없애기 어렵다.

큰 Role / State enum vocabulary
DALi가 다 쓰지 않아도 AT-SPI/export vocabulary와 매핑해야 해서 adaptor integration 쪽에는 남아야 함. toolkit 쪽에서 subset을 쓰는 건 가능하지만 adaptor bridge vocabulary 자체는 필요함.

Bridge singleton과 event emission 경로
접근성 bus와 screen reader로 event를 내보내는 중심점은 필요함. 이름이나 API 폭은 줄일 수 있지만, “bridge가 있고 accessible object가 event를 emit한다”는 구조 자체는 남을 수밖에 없어.

내 기준으로 다음에 제일 맛있는 정리 순서는 이거야:
accessibility-integ.h에서 accessibility-devel.h include 제거
AtspiInterface/AtspiEvent/ReadingInfoType를 integration으로 이동하고 devel alias 유지
accessibility-devel.h를 작은 type/event/geometry 헤더로 분리
toolkit/csharp의 ActorAccessible 직접 dynamic_cast 호출을 helper API로 줄이기