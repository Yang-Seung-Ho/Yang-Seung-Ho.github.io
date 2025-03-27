---
layout: post
title: SVG animate 오픈소스 프로젝트 만들기(4) - useRef vs useState, gsap 핵심 개념 정리
date: '2025-03-27 11:43:43 +0900'
categories:
  - Dev
  - FE
tags:
  - react
  - svg
  - gsap
  - scrolltrigger
  - typescript
  - 오픈소스
  - hooks
---

## 👐 이번 편에서 다룰 내용

이번 글에서는 지금까지 만든 **SVG 회전 애니메이션 컴포넌트**를 복습하며,  
이를 구성하는 데 핵심이 된 React의 `useRef`, `useState`의 차이와  
GSAP의 주요 메서드 및 플러그인(`ScrollTrigger`)에 대해 정리해본다.  

또한 트리거(trigger)별 동작 방식과 실전에서의 사용 이유를 짚어보며,  
**왜 하나의 컴포넌트에 다양한 동작을 통합했는지** 설계 의도를 함께 알아본다.

---

## 🧩 하나의 컴포넌트, 다양한 트리거

하나의 컴포넌트에 다양한 트리거(`trigger`)를 prop으로 전달받아 제어할 수 있도록 설계하였다.

---

## 🔧 useRef vs useState 차이점

| 항목           | `useState`               | `useRef`                        |
| -------------- | ------------------------ | ------------------------------- |
| 💡 목적         | 상태 값 저장 & 리렌더링  | 값 저장 (DOM 접근, 변수 저장)   |
| 🔄 리렌더 유발  | 예                       | 아니오                          |
| 🎯 주로 쓰는 곳 | 화면 UI에 영향을 주는 값 | DOM 요소, 애니메이션, 타이머 등 |
| 💥 성능 영향    | 많아지면 리렌더 많아짐   | 없음                            |

📌 **요약**

- `useState`: 값이 바뀌면 컴포넌트를 **리렌더링**
- `useRef`: 리렌더링 없이 **값만 저장** (애니메이션 상태 저장에 최적)

---

## ✨ useRef 실제 사용 예시

### ✅ 1. DOM 요소 직접 접근

```tsx
const svgRef = useRef<SVGSVGElement | null>(null);

<SvgComponent ref={svgRef} />

// svgRef.current → SVG DOM 객체 접근 가능
```

---

### ✅ 2. Tween 인스턴스 저장

```tsx
const tweenRef = useRef<gsap.core.Tween | null>(null);
```

- 생성된 애니메이션을 저장하여 `pause()`, `kill()`, `resume()` 등 제어 가능

---

### ✅ 3. 회전값 저장 (중간 상태 기억용)

```tsx
const rotationRef = useRef<number>(0);
```

- 회전 각도 추적 → hover나 auto 모드에서 중단 후 재개 시 사용

---

### ✅ 4. 중복 애니메이션 방지 플래그

```tsx
const isAnimating = useRef<boolean>(false);
```

- 클릭 애니메이션 중복 실행 방지

---

## 💚 GSAP 핵심 메서드 정리

| 메서드            | 설명                             |
| ----------------- | -------------------------------- |
| `gsap.to()`       | 현재 → 목표 상태로 애니메이션    |
| `gsap.from()`     | 시작 → 현재 상태 도달            |
| `gsap.fromTo()`   | 시작과 끝 상태 지정              |
| `gsap.set()`      | 애니메이션 없이 즉시 스타일 적용 |
| `gsap.timeline()` | 여러 애니메이션 순차 실행        |

### 📌 예시

```tsx
gsap.to(".box", {
  x: 200,
  rotation: 360,
  duration: 1,
  ease: "power2.out"
});
```

---

## 🔵 ScrollTrigger 정리

```tsx
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);
```

### ✅ 기본 사용

```tsx
gsap.to(".box", {
  x: 300,
  scrollTrigger: {
    trigger: ".box",
    start: "top 80%",
    end: "bottom 30%",
    scrub: true
  }
});
```

| 속성            | 설명                                     |
| --------------- | ---------------------------------------- |
| `trigger`       | 어떤 요소 기준으로 트리거                |
| `start` / `end` | 언제 시작/끝날지 기준                    |
| `scrub`         | 스크롤과 함께 부드럽게 따라가게          |
| `toggleActions` | play, pause, reset 등의 트리거 액션 정의 |

---

## 💫 transformOrigin 정리

애니메이션 회전이나 스케일 시 기준점을 지정  
기본값은 `"50% 50%"` (요소 중심)

| 예시          | 설명             |
| ------------- | ---------------- |
| `"50% 50%"`   | 중심 기준 회전   |
| `"0% 0%"`     | 왼쪽 위 기준     |
| `"100% 100%"` | 오른쪽 아래 기준 |

---

## 🔄 트리거 별 실제 사용 비교

| 트리거   | 설명                        | 예시 사용처                |
| -------- | --------------------------- | -------------------------- |
| `auto`   | 컴포넌트 mount 시 자동 회전 | 로딩 인디케이터, 배경 데코 |
| `hover`  | 마우스 오버 시 회전 시작    | 버튼, 아이콘 효과          |
| `click`  | 클릭 시 한 번만 회전        | 인터랙션 강조, 토글 효과   |
| `scroll` | 스크롤 비율 따라 회전       | 섹션 등장, 스크롤 콘텐츠   |

---

## 🧾 정리하며

이번 글에서는 **useRef와 useState의 차이**,  
**GSAP의 주요 메서드**,  
그리고 **ScrollTrigger**, **transformOrigin**, 다양한 **트리거 방식**에 대해 정리하였다.

```

--- 

