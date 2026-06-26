# Phase 6 - AT-SPI Adapter 독립화

## 목적

DALi 내부 Accessibility model을 AT-SPI에 직접 묶지 않고, AT-SPI를 하나의 backend adapter로 분리한다.

## 목표 구조

```text
Toolkit / UI
    |
DALi Accessibility Semantics
    |
Backend Adapter
    +-- AT-SPI / DBus
    +-- future backend
```

## 원칙

DALi semantic model은 처음에는 AT-SPI를 기준으로 정의해도 된다. 다만 내부 API 이름과 타입은 DALi 의미로 둔다.

예를 들어 내부 API는 `Role::Button`, `State::Checked`, `Action::Activate`처럼 표현하고, AT-SPI adapter에서 `Role::PUSH_BUTTON`, `State::CHECKED`, `DoAction`으로 변환한다.

## Core Semantics

- tree
- role
- name
- description
- value
- states
- bounds
- actions
- relations
- text/value/selection optional capability
- events

## Extension 영역

- reading material
- custom navigation
- screen reader command
- dump/tree export
- post-render listen
- DALi-specific gesture

## 완료 기준

- Toolkit/UI API가 AT-SPI 타입에 의존하지 않는다.
- AT-SPI enum/interface 변환이 adapter 내부로 모인다.
- 새 backend를 붙일 때 common subset부터 매핑할 수 있다.
- DALi extension과 diagnostic 기능이 core semantic API와 분리된다.
