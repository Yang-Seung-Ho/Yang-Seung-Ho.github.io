---
layout: post
title: SVG animate 오픈소스 프로젝트 만들기(5) - 다양한 Flip 애니메이션
date: '2025-04-01 16:31:47 +0900'
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

## 🪞 Flip 애니메이션이란?

이번 글에서는 **SVG를 상하좌우로 뒤집는 Flip 애니메이션**을 구현한다.  
이전의 Rotate 와 마찬가지로, React 컴포넌트로 설계하여, 프론트(앞면)와 백(뒷면)을 넘겨받고, `hover`, `click`, `scroll`, `auto` 트리거 방식도 지원한다.
사용의 편리함을 위해서, 뒷면의 svg를 미지정 시에는 앞면의 svg를 뒤집어 자동으로 생성된다.

---

## ✨ 핵심 기능

- `direction`에 따라 상하좌우(flip)
- `front`와 `back` SVG 전달 (뒷면 미지정 시 앞면을 통해 자동 생성)
- `hover`, `click`, `scroll`, `auto` 트리거
- `loop` 옵션으로 반복 여부 설정

---

## ⚙️ 주요 CSS 속성 설명

### 🔹 `transform-style: preserve-3d`

```css
transform-style: preserve-3d;
```

- 자식 요소들이 **3D 공간에서 독립적으로 배치**되도록 만든다.
- 이 속성이 없으면, 앞면과 뒷면이 **2D 평면으로 눌려** 보여서 flip 효과가 깨진다.

### 비교 영상

![flip-preserve-3d 실험](</assets/img/Apr-01-2025 16-45-26.gif>)

### 🔍 왜 flat에서는 뒷면이 안 보일까?

`transform-style: flat`은 **자식 요소들이 3D 공간에서 독립적으로 배치되지 않도록** 만든다. 즉:

- 부모 요소(`Inner`)가 `flat`이면 자식 요소(`Face`)들은 **2D 평면에서만 렌더링**됨
- 이 상태에서 `rotateY(180deg)`나 `rotateX(180deg)` 같은 3D 회전이 적용되어도,
  **뒷면이 카메라 뒤로 보내진 것으로 간주되고, `backface-visibility: hidden` 때문에 보이지 않음**

---

### ✅ 요약 비교

| 속성                           | 효과                               | 결과                                                               |
| ------------------------------ | ---------------------------------- | ------------------------------------------------------------------ |
| `transform-style: preserve-3d` | 자식이 3D 공간에서 독립적으로 회전 | 앞/뒷면 모두 회전하면서 flip 효과 정상 작동                        |
| `transform-style: flat`        | 자식이 2D 공간에 붙어 있음         | 회전은 되지만, 뒷면은 안 보임 (`backface-visibility: hidden` 때문) |

---

### 🔹 `transform-origin: center`

```css
transform-origin: center;
```

- Flip 애니메이션의 목표인 좌, 우, 상, 하 뒤집기 시 어떤 축을 기준으로 회전할 지 정해준다.
- 회전 축의 기준점을 설정한다.
- 예) `rotateY(180deg)`일 때, **가로 중심**을 기준으로 Y축 회전.
- 방향별 회전과 조합하면 다음과 같이 동작한다:

| 방향            | 회전 축 (`rotateX` or `rotateY`) | `transform-origin` 효과 |
| --------------- | -------------------------------- | ----------------------- |
| `left`, `right` | `rotateY`                        | 세로 중심으로 가로 회전 |
| `up`, `down`    | `rotateX`                        | 가로 중심으로 세로 회전 |

---

## 🧩 기본 구조

```tsx
<FlipCard>
  <FlipInner direction="right">
    <Face>Front</Face>
    <Face back>Back</Face>
  </FlipInner>
</FlipCard>
```

- `<FlipInner>`는 실제로 회전하는 요소
- `<Face>`는 앞면 또는 뒷면으로, back일 경우 180도 회전
- 방향에 따라 `rotateX` 또는 `rotateY`를 사용한다

---

## 🔧 flip 함수 분석

`flip()` 함수는 사용자의 인터랙션(hover, click 등)이나 자동 실행(auto, scroll)에 따라  
SVG 컴포넌트를 **앞면 ↔ 뒷면으로 전환시키는 핵심 로직**이다.

```tsx
gsap.to(InnerRef.current, {
  [axis]: targetDeg,
  duration,
  ease: "power2.inOut",
});
setFlipped((prev) => !prev);
```

### ✅ `gsap.to()`

- `InnerRef.current` 요소를 `rotateX` 또는 `rotateY`로 회전시킨다.
- 회전 방향은 `direction` 값(`left`, `right`, `up`, `down`)에 따라 결정되며,
  `rotateY` 또는 `rotateX`로 동적으로 설정된다.
- `targetDeg`는 보통 `180` 또는 `-180`으로 설정되며, 이전 상태에 따라 앞면 ↔ 뒷면 전환을 반복한다.

### ✅ `setFlipped((prev) => !prev)`

- 현재 앞/뒷면 상태를 `flipped` 상태로 저장한다.
- 상태를 토글하여 다음 flip 시 다시 반대면으로 전환되도록 한다.
- 이를 통해 **다시 클릭하거나 hover 시 이전 상태를 기억하며 전환**되는 구조가 완성된다.

### 💡 전체 동작 흐름

1. 사용자가 특정 트리거(`click`, `hover`, etc)를 발생시킴
2. `flip()` 함수 호출
3. `gsap.to()`로 회전 애니메이션 실행
4. `flipped` 상태 업데이트 → 다음 방향을 기억함

---


## 🛠️ 자동 뒷면 생성 로직

`back` 속성이 없을 경우, front SVG를 방향에 따라 뒤집어 자동으로 만든다:

```tsx
const transform =
  direction === "left" || direction === "right"
    ? "scale(-1,1) translate(-100,0)"
    : "scale(1,-1) translate(0,-100)";
```

- 좌우 방향: 좌우 반전(scaleX -1)
- 상하 방향: 상하 반전(scaleY -1)
- `translate`로 위치를 보정하여 정확히 겹치게 만듦

---

## 💡 트리거 동작 방식 요약

| 트리거   | 설명                             |
| -------- | -------------------------------- |
| `auto`   | 마운트 후 자동 flip (loop 가능)  |
| `hover`  | 마우스를 올리면 flip             |
| `click`  | 클릭 시 flip                     |
| `scroll` | 특정 스크롤 영역에 도달하면 flip |

---

## 📦 사용 예시

```tsx
<SvgFlip
  trigger="hover"
  front={FrontSvg}
  back={BackSvg}
  direction="left"
  duration={1}
/>
```

- `direction`: `"left"`, `"right"`, `"up"`, `"down"`
- `front`: SVG 컴포넌트
- `back`: 없을 경우 자동 생성

---

## 📸 결과 예시

<table>
  <thead>
    <tr>
      <th>트리거</th>
      <th>예시</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>auto-loop</code></td>
      <td>
        <div style="text-align:center;">
          <img src="/assets/img/Apr-01-2025 16-53-41.gif" alt="flip-auto" width="200" />
        </div>
      </td>
    </tr>
    <tr>
      <td><code>hover, click, scroll</code></td>
      <td>
        <div style="text-align:center;">
          <img src="/assets/img/Apr-01-2025 16-56-05.gif" alt="hover,click,scroll" width="200" />
        </div>
      </td>
    </tr>
  </tbody>
</table>


---

## 🔚 마무리

이번 편에서는 Flip 애니메이션의 구조와 구현 방법을 살펴보았다.  
특히 `transform-style: preserve-3d`와 `backface-visibility`, `transform-origin`의 중요성을 이해하고,  
GSAP과 ScrollTrigger를 활용해 다양한 트리거 기반 flip도 구현하였다.

---

## 🔜 다음 편 예고

다음 글에서는 **Jump 애니메이션**과 **기본 GSAP Timeline 활용법**을 바탕으로  
SVG를 튕기거나 점프하는 인터랙션을 구현해볼 예정이다.

> 작은 오픈소스지만 점점 완성되어가는 모습이 기대된다!
