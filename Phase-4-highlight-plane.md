# Phase 4 - Highlight Plane 개선

## 목적

현재 highlight actor를 대상 actor의 child로 붙이는 방식을 줄이고, 별도 accessibility highlight plane에서 표시하는 구조로 전환한다.

## 기대 효과

- 사용자가 highlight actor에 직접 접근하거나 child tree에서 관찰하는 것을 막을 수 있다.
- accessibility highlight가 일반 UI hierarchy와 덜 섞인다.
- highlight rendering 책임을 toolkit control이 아니라 accessibility system 쪽으로 이동할 수 있다.

## 주요 과제

- 대상 object의 window/screen 좌표 계산.
- scroll, clip, scale, rotation, transform 동기화.
- multi-window/sub-window 처리.
- offscreen rendering, layer order, 3D layer 처리.
- highlight 대상이 사라지거나 hidden 되는 경우의 정리.

## 권장 접근

한 번에 완전 전환하지 말고, 기존 child 방식과 새 plane 방식을 feature flag 또는 내부 옵션으로 병행한다. 먼저 기본 2D Control/View부터 전환하고, 복잡한 transform/clip case를 단계적으로 확장한다.

## 완료 기준

- highlight actor가 accessibility tree나 user-facing child list에 노출되지 않는다.
- 기존 highlight 동작과 시각적 결과가 유지된다.
- scroll/resize/visibility 변화에 맞춰 highlight가 안정적으로 갱신된다.
