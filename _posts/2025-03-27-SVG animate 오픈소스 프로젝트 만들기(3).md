---
layout: post
title: "SVG animate 오픈소스 프로젝트 만들기(3)"
date: 2025-03-26 10:00:00 +0900
categories:
  - Dev
  - FE
tags:
  - react
  - SVG animate 오픈소스 프로젝트 만들기
  - svg
  - gsap
  - 애니메이션
  - scrolltrigger
  - styled-components
  - typescript
  - lottie
  - 오픈소스
---

## 🗂 프로젝트 구조 설계

이번 편에서는 페이지를 분리하고, **React Router**를 적용해 애니메이션 페이지를 전환할 수 있도록 구조를 잡고  
각기 다른 **트리거(trigger)**로 애니메이션을 작동시키는 방법을 구현해본다.

아래 디렉토리 구조는 애니메이션을 시각화하기 위해 화면에 나타내려고 작성된 임시 구조이다.

```bash
src/
├── assets/           # SVG 파일 모음
│   ├── circle.svg
│   ├── triangle.svg
│   └── test.svg
├── components/
│   └── SvgRotate.tsx  # 회전 애니메이션 컴포넌트
├── pages/            # 라우팅될 개별 페이지들
│   ├── Home.tsx
│   ├── Rotate.tsx
│   ├── Flip.tsx
│   └── Jump.tsx
├── App.tsx
└── App.css
```

### 🔁 App.tsx - React Router 설정
``` tsx
import React from "react";
import { BrowserRouter as Router, Route, Routes } from "react-router-dom";
import Home from "./pages/Home";
import Rotate from "./pages/Rotate";
import Flip from "./pages/Flip";
import Jump from "./pages/Jump";

function App() {
  return (
    <Router>
      <Home />
      <Routes>
        <Route path="/rotate" element={<Rotate />} />
        <Route path="/flip" element={<Flip />} />
        <Route path="/jump" element={<Jump />} />
      </Routes>
    </Router>
  );
}

export default App;

```
💡 /rotate, /flip, /jump 경로에 따라 각기 다른 SVG 애니메이션을 보여준다.
---
### 화면 결과
<img src="/assets/img/2025-03-27-main.png" alt="main Page" width="400"/>

---
## 🧩 SvgRotate.tsx - 컴포넌트 내부 로직 설명

```tsx
type TriggerType = "auto" | "hover" | "click" | "scroll";
```

### 주요 Props 설명
| Prop       | 설명                                                  |
| ---------- | ----------------------------------------------------- |
| `duration` | 애니메이션 지속 시간 (초)                             |
| `rotation` | 회전 각도                                             |
| `ease`     | 이징 함수 (`linear`, `easeInOut`, 등)                 |
| `trigger`  | 트리거 타입 지정 (`auto`, `hover`, `click`, `scroll`) |
| `svg`      | 사용할 SVG 컴포넌트 (없으면 기본 원 사용됨)           |

---
## ⚙️ 트리거별 동작 설명
---
### 1. auto
페이지 진입 시 자동으로 회전 시작

```tsx
if (trigger === "auto") {
  gsap.to(svgRef.current, {
    rotation,
    repeat: -1,
    duration,
    ease,
    transformOrigin: "50% 50%",
  });
}
```
### 2. hover
마우스를 올리면 회전 시작, 내리면 일시 정지 (재진입 시 이어서 진행)

```tsx
const handleMouseEnter = () => {
  if (trigger === "hover") {
    tweenRef.current = gsap.to(svgRef.current, {
      rotation: "+=" + rotation,
      repeat: -1,
      duration,
      ease,
      transformOrigin: "50% 50%",
    });
  }
};

const handleMouseLeave = () => {
  tweenRef.current?.pause();
};
```

### 3. click
한 번 클릭할 때마다 지정한 각도로 한 번 회전

```tsx
const handleClick = () => {
  if (trigger === "click" && !isAnimating.current) {
    isAnimating.current = true;
    gsap.fromTo(
      svgRef.current,
      { rotation: 0 },
      {
        rotation,
        duration,
        ease,
        onComplete: () => (isAnimating.current = false),
      }
    );
  }
};
```

### 4. scroll(ScrollTrigger 매우 만족)
스크롤 비율에 따라 부드럽게 회전 (ScrollTrigger 플러그인 활용)

```tsx
gsap.registerPlugin(ScrollTrigger);

if (trigger === "scroll") {
  gsap.to(svgRef.current, {
    rotation,
    ease,
    transformOrigin: "50% 50%",
    scrollTrigger: {
      trigger: document.body,
      start: "top top",
      end: "bottom bottom",
      scrub: true,
    },
  });
}
```
---

### 5. SvgRotate.tsx Return
``` tsx
<Icon
  onMouseEnter={handleMouseEnter}
  onMouseLeave={handleMouseLeave}
  onClick={handleClick}
>
  <SvgComponent ref={svgRef} />
</Icon>
```
위 까지 온다면 
Rotate 컴포넌트 작성 완료!!! <br>
아래는 Rotate.tsx 에서 SvgRotate 컴포넌트를 불러와 사용하는 예시다.

---

## 트리거 별 사용 예시 - Rotate.tsx
```tsx

import React from "react";
import SvgRotate from "../components/SvgRotate";
import { ReactComponent as CustomSvg } from "../assets/triangle.svg";

function Rotate() {
  return (
    <div style={{ padding: "50px", textAlign: "center", height: "150vh" }}>
      <h2>자동 회전</h2>
      <SvgRotate trigger="auto" />

      <h2>Hover 시 회전</h2>
      <SvgRotate trigger="hover" duration={1} />

      <h2>Click 시 회전</h2>
      <SvgRotate trigger="click" rotation={180} />

      <h2>스크롤 시 회전 + 커스텀 SVG</h2>
      <SvgRotate trigger="scroll" svg={CustomSvg} rotation={360} />
    </div>
  );
}

export default Rotate;
```
---

### 💫 회전 최종 결과
마우스가 녹화에 안보이네...<br>
![result](</assets/img/Mar-27-2025 10-45-10.gif>)
---

## 🏁 마무리
이번 편에서는 SVG 애니메이션을 트리거 기반으로 제어할 수 있도록 `SvgRotate` 컴포넌트를 설계하고,  
React Router를 통해 페이지를 분리하여 각 애니메이션을 독립적으로 테스트할 수 있도록 구성하였다.

GSAP의 `to`, `fromTo`, `ScrollTrigger`를 활용해  
- 자동 회전 (`auto`)  
- 마우스 진입 시 회전 (`hover`)  
- 클릭 시 1회 회전 (`click`)  
- 스크롤에 따라 회전 (`scroll`)  

까지 각기 다른 사용자 인터랙션 기반 애니메이션을 하나의 컴포넌트에서 관리할 수 있게 되었다.

---

### 🔜 4편 내용

다음 편에서는 지금까지 구현한 로직을 기반으로 다음 내용을 다룰 예정이다.

### ✅ 정리 & 복습

- `useRef` vs `useState`: 두 훅의 차이점과 왜 애니메이션에서는 `useRef`를 주로 사용하는지
- `gsap`의 동작 방식 정리 및 주요 메서드 요약
- `ScrollTrigger`의 개념과 내부 작동 방식
- 트리거별 실제 사용 상황 비교

### 🚀 다음 목표

- Flip 애니메이션 구현 (뒤집기)
- Jump 애니메이션 구현 (점프)
- 컴포넌트 확장 설계: 재사용성과 확장성을 고려한 아키텍처 구성

---

지금까지 잘 따라왔다면 여러분은 이미 SVG 애니메이션 오픈소스 라이브러리를 향해 훌륭한 한 걸음을 내디딘 것이다.  
다음 편에서 더 다양한 애니메이션을 다뤄보자!
