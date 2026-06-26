# DALi Accessibility Refactoring Plan

## 목표

DALi Accessibility를 한 번에 전면 재작성하지 않고, 사용자에게 노출되는 API부터 안정적으로 정리한 뒤 내부 책임과 AT-SPI 의존성을 단계적으로 낮춘다.

핵심 원칙은 다음과 같다.

- App 개발자는 `dali-toolkit` 또는 `dali-ui` API만 사용한다.
- 외부 프로세스는 DBus/AT-SPI bridge를 통해서만 DALi 앱과 상호작용한다.
- `dali-adaptor`의 C++ API 표면은 최소화한다.
- Toolkit/UI API는 AT-SPI backend에 직접 묶이지 않는 DALi semantic API로 정리한다.
- 내부 구조 변경은 API 전환 이후 단계적으로 진행한다.

## Phase 목록

1. [Phase 0 - API 분류](./Phase-0-api-classification.md)
2. [Phase 1 - dali-adaptor API 최소화](./Phase-1-adaptor-api-minimization.md)
3. [Phase 2 - toolkit/ui Accessibility API 추가](./Phase-2-toolkit-ui-semantics-api.md)
4. [Phase 3 - 기존 Control/View API 위임](./Phase-3-legacy-property-delegation.md)
5. [Phase 4 - Highlight Plane 개선](./Phase-4-highlight-plane.md)
6. [Phase 5 - toolkit/adaptor 책임 재배치](./Phase-5-responsibility-rebalance.md)
7. [Phase 6 - AT-SPI Adapter 독립화](./Phase-6-atspi-adapter-separation.md)

## 참고 문서

- [현재 구조 리뷰](./accessibility-architecture-review.md)
- [현재 인터랙션 요약](./accessibility-current-interactions.md)
