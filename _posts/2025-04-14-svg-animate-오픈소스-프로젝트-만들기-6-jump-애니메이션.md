---
title: SVG animate 오픈소스 프로젝트 만들기(6) - 다양한 Jump 애니메이션
date: 2025-04-14
categories:
  - Dev
  - SVG Animate
tags:
  - react
  - svg
  - gsap
  - scrolltrigger
  - typescript
  - 오픈소스
  - hooks
---

## 🏃 Svg Jump 애니메이션

이번 글에서는 **SVG가 통통 튀는 Jump 애니메이션**을 다양한 방식으로 구현해봅니다. 
기존에 만들었던 `Rotate`, `Flip`처럼 `SvgJump` 컴포넌트를 기반으로 다양한 효과를 트리거별로 조작할 수 있으며,
**GSAP Timeline** 문법을 적극적으로 활용해 더 풍부한 애니메이션을 구성할 수 있습니다.

---

## ✨ 주요 기능 요약

- `type`에 따라 10종 이상의 점프 스타일 구현
- `trigger`: `auto`, `hover`, `click` 트리거 방식 선택
- `y`, `duration`으로 점프의 강도와 속도 조절
- 기본 SVG 미제공 시 `circle.svg` 사용

---

## 🔧 SvgJump 코드 요약

```tsx
<SvgJump trigger="click" type="bounce" y={-120} duration={2} />
```

컴포넌트는 아래와 같은 props를 받습니다:

| Prop       | 설명                                           |
| ---------- | ---------------------------------------------- |
| `type`     | 점프 타입 지정 (`basic`, `bounce`, `twist` 등) |
| `trigger`  | `click`, `hover`, `auto`                       |
| `y`        | 점프 높이 (음수로 입력)                        |
| `duration` | 전체 애니메이션 소요 시간 (초 단위)            |
| `repeat`   | 반복 횟수 (기본 -1: 무한반복)                  |

---

## 🧩 점프 타입별 설명

| 타입     | 설명                                             |
| -------- | ------------------------------------------------ |
| basic    | 단순 위아래 점프                                 |
| bounce   | 위로 튄 후 통통 튀듯이 점프 (Timeline 사용)      |
| repeat   | 3회 반복 점프                                    |
| elastic  | 고무처럼 탄성 있게 튐                            |
| scale    | 커졌다가 작아지며 튀는 점프                      |
| twist    | 회전하며 튀는 점프 (`rotation` 사용)            |
| squash   | 눌리는 효과와 함께 점프 (`scaleX`, `scaleY`)     |
| shake    | 위로 튄 후 좌우로 흔들리는 점프                 |
| stomp    | 세게 밟고 튀는 듯한 점프 (`scaleY` 압축)         |
| drip     | 위로 살짝 튄 후 떨어지는 방울처럼 느린 착지       |

---

## 🔎 결과 예시

각 타입별 예시를 `click` 트리거로 설정해 테스트합니다.

![SvgJump](<../assets/img/jump-Apr-14-2025 15-43-22.gif>)
```tsx
<SvgJump trigger="click" type="basic" y={-80} duration={0.6} />
<SvgJump trigger="click" type="bounce" y={-120} duration={1.5} />
<SvgJump trigger="click" type="repeat" y={-100} duration={0.4} repeat={5} />
<SvgJump trigger="click" type="elastic" y={-100} duration={0.5} />
<SvgJump trigger="click" type="scale" y={-80} duration={0.3} />
<SvgJump trigger="click" type="twist" y={-100} duration={0.6} />
<SvgJump trigger="click" type="squash" y={-100} duration={0.8} />
<SvgJump trigger="click" type="shake" y={-60} duration={0.6} />
<SvgJump trigger="click" type="stomp" y={-90} duration={0.8} />
<SvgJump trigger="click" type="drip" y={-70} duration={1.2} />
```

---

## 🎬 GSAP Timeline 완전 정리 – 애니메이션을 순차적으로 다루는 강력한 도구

GSAP에서 `timeline`은 **복잡한 애니메이션을 시간 순서대로 제어**할 수 있는 핵심 기능입니다. 단일 애니메이션으로는 불가능한 **연속적 움직임**, **효과의 조합**, **세밀한 타이밍 제어**를 가능하게 합니다.

---

## 📌 기본 문법

```ts
const tl = gsap.timeline({
  defaults: { ease: "power1.out", duration: 0.5 }, // 모든 step에 공통 옵션 적용
  onComplete: () => console.log("끝!"),
});

// 애니메이션 단계 순차적으로 연결
tl.to(".box", { y: -100 })
  .to(".box", { y: 0 })
  .to(".box", { rotation: 360 });
```

### 주요 메서드

| 메서드      | 설명                                                                 |
| ----------- | -------------------------------------------------------------------- |
| `.to()`     | 요소에 대한 **목표 상태**를 설정하고, 해당 값까지 애니메이션        |
| `.from()`   | 요소의 **초기 상태**를 설정하고, 현재 값까지 애니메이션             |
| `.fromTo()` | 시작 상태와 끝 상태를 명시적으로 지정                                |
| `.set()`    | 애니메이션 없이 즉시 속성 적용                                       |
| `.add()`    | 다른 timeline이나 함수 삽입 가능                                     |

---

## ⚙️ 옵션들

- `defaults`: 모든 단계에 공통으로 적용할 기본 옵션 설정 (`ease`, `duration` 등)
- `onComplete`: 전체 timeline이 끝났을 때 실행할 콜백
- `delay`: timeline 전체의 시작 지연 시간
- `repeat`: 전체 timeline 반복 횟수
- `repeatDelay`: 반복 사이 간격 설정

---

## ⏱ 타이밍 제어 예시

```ts
const tl = gsap.timeline();
tl.to(".circle", { y: -100, duration: 0.2 })
  .to(".circle", { y: 0, duration: 0.2 })
  .to(".circle", { scale: 1.2, duration: 0.1 }, "+=0.2"); // 0.2초 후 실행
```

---

## 🧩 SvgJump에서 Timeline 활용

`SvgJump` 컴포넌트에서 `timeline`은 **다단계 점프 애니메이션**에 필수적으로 사용됩니다.

### 예시: `bounce` 점프

```ts
const tl = gsap.timeline({ onComplete: onJumpComplete });
const bounces = [y, y * 0.6, y * 0.4, y * 0.25, y * 0.1];

bounces.forEach((dy) => {
  tl.to(svgRef.current, {
    y: dy,
    duration: singleDuration,
    ease: "power2.out",
  });
  tl.to(svgRef.current, {
    y: 0,
    duration: singleDuration,
    ease: "power2.in",
  });
});
```

🔎 여기서 중요한 포인트:

- 여러 `.to()`가 연속적으로 쌓이면서 **자연스러운 튕김 효과**가 만들어짐
- 각 bounce마다 높이가 점점 줄어들어 **물리적인 감각**을 구현
- `onComplete`을 통해 애니메이션 종료 후 상태를 관리

---

### 예시: `shake` 점프

```ts
const tl = gsap.timeline({ onComplete: onJumpComplete });
tl.to(svgRef.current, { y, duration: 0.2 })
  .to(svgRef.current, { x: -10, duration: 0.1 })
  .to(svgRef.current, { x: 10, duration: 0.1 })
  .to(svgRef.current, { x: -6, duration: 0.1 })
  .to(svgRef.current, { x: 6, duration: 0.1 })
  .to(svgRef.current, { x: 0, y: 0, duration: 0.2 });
```

- 위로 튀었다가 좌우로 흔들리는 애니메이션을 순서대로 표현
- 단일 `gsap.to()`로는 표현하기 어려운 세밀한 흔들림을 재현

---

## ✅ Timeline이 필요한 이유

| 단일 gsap.to()로 불가능한 상황 | Timeline을 사용하면 가능한 것               |
| ------------------------------ | ------------------------------------------- |
| 여러 단계의 효과를 순차적으로 실행 | `.to().to().to()`로 순서대로 제어 가능     |
| 전체 반복보다 단계별 반복 필요     | 원하는 step만 반복하거나 타이밍 제어 가능  |
| 점프 → 흔들림 → 원위치 등 복합 효과 | 한 줄씩 쌓아 시각적으로 순서 구성 용이    |

---

## 🧠 정리

| 장점                     | 설명                                                          |
| ------------------------ | ------------------------------------------------------------- |
| 순차 제어                | `.to().to().to()`로 자연스러운 흐름 구성 가능                 |
| 반복 및 지연 제어        | `repeat`, `delay`, `repeatDelay` 등으로 정밀한 제어 가능      |
| 재사용성                | 하나의 timeline을 변수로 만들어 여러 컴포넌트에 재사용 가능  |
| 콜백 처리               | `onStart`, `onUpdate`, `onComplete` 등 다양한 생명주기 제공   |

---
✨
## 🔚 마무리

이번 편에서는 `SvgJump` 컴포넌트를 통해 다양한 점프 애니메이션을 구현해보았습니다.

- `Timeline`과 `ease` 조합으로 다양한 물리 기반 애니메이션 구성
- 코드와 기능이 깔끔하게 분리되어 다양한 타입 확장 가능

---

## 🔜 다음 편 예고

다음 글에서는 **SVG 애니메이션 라이브러리의 문서화 및 오픈소스 배포**에 대해 소개할 예정입니다.
- 라이브러리로 추출하는 방법
- `README` 문서 구성법
- 배포를 위한 `npm publish` 절차 등
