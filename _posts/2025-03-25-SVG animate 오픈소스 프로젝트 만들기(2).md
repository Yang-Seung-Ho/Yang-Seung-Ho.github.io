---
layout: post
title: "SVG animate 오픈소스 프로젝트 만들기(2)"
date: 2025-03-25 15:00:00 +0900
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

## 📝 서론
이전 글에서 오픈소스를 만들게 된 배경과 GSAP의 특징 및 장점에 대해 소개하였다.  
이번 글에서는 본격적으로 프로젝트를 시작하며 **개발 환경을 세팅하고**,  
간단한 **GSAP 사용 예시**와 **SVG를 불러와 회전시키는 애니메이션 컴포넌트**를 구현해본다.

특히 우리는 **하나의 회전 컴포넌트에 `auto`, `hover`, `click`, `scroll` 등 다양한 트리거를 인수로 넘겨줄 수 있도록 설계**함으로써,  
**재사용성과 유지보수성을 높인 구조**를 목표로 한다.


---

## ⚙️ 초기 세팅

React + TypeScript 기반 프로젝트를 아래 명령어로 생성하고 필요한 라이브러리를 설치한다.

```bash
npx create-react-app React-svg-animate --template typescript
npm install gsap react-svg styled-components
```
---
- `gsap`: 애니메이션 구현을 위한 핵심 라이브러리
- `react-svg`: SVG 파일을 컴포넌트로 사용할 수 있게 해줌
- `styled-components`: CSS-in-JS 방식으로 컴포넌트 스타일링 가능
---
## 🚀 GSAP 간단 사용 예시
GSAP을 프로젝트에 적용하면, DOM 또는 SVG 요소를 부드럽게 애니메이션 처리할 수 있다.
아래는 간단한 박스 컴포넌트를 **x축 이동 + 회전**시키는 예제이다.

```tsx
import React, { useEffect, useRef } from "react";
import gsap from "gsap";

const Box: React.FC = () => {
  const boxRef = useRef<HTMLDivElement | null>(null);

  useEffect(() => {
    gsap.to(boxRef.current, {
      x: 200,         // x축으로 200px 이동
      rotation: 360,  // 360도 회전
      duration: 2,    // 2초 동안
      ease: "power2.inOut", // 부드러운 가속도
    });
  }, []);

  return (
    <div
      ref={boxRef}
      style={{
        width: "100px",
        height: "100px",
        backgroundColor: "skyblue",
        margin: "50px auto",
      }}
    >
      Box
    </div>
  );
};

export default Box;
```
---
## ✨ 주요 메서드 요약

| 메서드          | 설명                               |
| --------------- | ---------------------------------- |
| `gsap.to()`     | 특정 상태로 애니메이션             |
| `gsap.from()`   | 시작 상태부터 현재 상태까지        |
| `gsap.fromTo()` | 시작 → 끝 상태를 명확히 지정       |
| `gsap.set()`    | 즉시 스타일 적용 (애니메이션 없음) |

---

## 💡 추가 꿀팁

- `repeat: -1` → 무한 반복  
- `yoyo: true` → 반복 시 되돌아오는 효과  
- `delay: 1` → 1초 후 애니메이션 시작  
- `scrollTrigger` → 스크롤 기반 애니메이션 구현 가능 (플러그인 등록 필요)

---

## 🟢 SVG 작성 및 불러오기

애니메이션에 사용할 SVG는 아래와 같이 작성한다.  
`src/assets/circle.svg` 경로에 저장한다.

```xml
<!-- src/assets/circle.svg -->
<svg
  width="100"
  height="100"
  viewBox="0 0 100 100"
  xmlns="http://www.w3.org/2000/svg"
>
  <circle
    cx="50"
    cy="50"
    r="40"
    stroke="black"
    stroke-width="6"
    fill="none"
    stroke-dasharray="120 40"
    stroke-linecap="round"
  />
</svg>
```
### SVG 결과

<svg
  width="100"
  height="100"
  viewBox="0 0 100 100"
  xmlns="http://www.w3.org/2000/svg"
>
  <!-- 불균형한 stroke-dasharray로 끊긴 원 -->
  <circle
    cx="50"
    cy="50"
    r="40"
    stroke="black"
    stroke-width="6"
    fill="none"
    stroke-dasharray="120 40"
    stroke-linecap="round"
  />
</svg>
---
## 🔄 SVG를 불러와 회전 애니메이션 적용하기 (Auto)

이제 `SvgRotate.tsx` 컴포넌트를 만들어 SVG를 불러오고,  
GSAP을 사용해 자동 회전 애니메이션을 적용해보자.

```tsx
// src/components/SvgRotate.tsx

import React, { useEffect, useRef } from "react";
import gsap from "gsap";
import { ReactComponent as Circle } from "../assets/circle.svg";
import styled from "styled-components";

const Icon = styled.div`
  width: 100px;
  height: 100px;
  display: inline-block;
`;

const SvgRotate: React.FC = () => {
  const svgRef = useRef<SVGSVGElement | null>(null);

  useEffect(() => {
    if (svgRef.current) {
      gsap.to(svgRef.current, {
        rotation: 360,
        transformOrigin: "50% 50%",
        repeat: -1,
        duration: 2,
        ease: "linear",
      });
    }
  }, []);

  return (
    <Icon>
      <Circle ref={svgRef} />
    </Icon>
  );
};

export default SvgRotate;
```
### 결과
![Rotate auto](/assets/img/rotateautogif.gif)

## 🏁 마무리

이번 글에서는 프로젝트의 초기 환경을 세팅하고,  
GSAP을 활용한 간단한 예제부터 시작하여, SVG 파일을 불러와  
**자동 회전 애니메이션을 적용하는 컴포넌트 `SvgRotate`**를 구현해보았다.

이제 기본적인 SVG 애니메이션 적용이 가능한 기반이 갖춰졌으며,  
이를 바탕으로 다음 단계에서는 다양한 **인터랙션 기반 애니메이션 트리거**들을 구현해볼 예정이다.

### 🔜 3편 내용
다음 글에서는 `SvgRotate` 컴포넌트를 확장하여  
- `hover`, `click`, `scroll`, `auto` 등 **사용자 트리거 기반 애니메이션**
- 트리거 타입에 따른 상태 관리 및 제어
- ScrollTrigger를 활용한 스크롤 연동 애니메이션

등을 구현하며, 실제 UI에서 어떻게 다양하게 활용할 수 있는지를 소개할 예정이다.