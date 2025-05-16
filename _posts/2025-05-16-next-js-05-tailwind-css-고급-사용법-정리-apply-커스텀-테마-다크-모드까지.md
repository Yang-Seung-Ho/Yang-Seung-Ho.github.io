---
layout: post
title: "[Next.js] 05 - Tailwind CSS 고급 사용법 정리: @apply, 커스텀 테마, 다크 모드까지"
date: 2025-05-15 12:36 +0900
description: Tailwind CSS를 활용한 @apply, 커스텀 테마 확장, 다크모드 구현까지, 실전 프로젝트에서 유용한 고급 기능들을 정리해보았다.
categories:
  - Dev
  - Next.js
tags:
  - Next.js
  - Tailwind CSS
  - 스타일링
  - 프론트엔드
  - CSS
  - 개발환경
  - 다크모드
  - 커스텀 테마
  - apply
published: true
---
## 기존 CSS의 불편함과 Tailwind의 개선점

Next.js로 프로젝트를 진행하면서, 기존 CSS 방식에서는 다음과 같은 불편함이 있었다:

* **클래스 네이밍에 매번 고민해야 함** (예: `.btn-blue`, `.box-center`, ...)
* **스타일이 컴포넌트와 멀리 떨어져 있어서** 유지보수 시 문맥을 잃기 쉬움
* **재사용이 어려운 중복 스타일** 발생
* 다크모드 구현 시, 직접 `body.dark` 클래스 또는 `data-theme`를 제어하고, CSS 전역으로 스타일 덮어쓰기 필요

이러한 문제를 해결하기 위해 도입한 것이 **Tailwind CSS**다.

Tailwind를 사용하면서 다음과 같은 개선이 이루어졌다:

* ❌ 네이밍 스트레스 → ✅ 유틸리티 클래스 조합으로 스타일 구성
* ❌ 스타일 분리 → ✅ JSX 내에서 바로 클래스 선언
* ❌ 다크모드 수동 제어 → ✅ `dark:` 접두사로 클래스 레벨 제어
* ❌ 복잡한 반응형 스타일 → ✅ `sm:`, `md:`, `lg:` 등의 접두사로 간결하게 처리

---

## 목표

이번 글에서는 Tailwind CSS의 기본 설치를 마친 후, 실무에서 유용하게 활용되는 고급 기능들을 정리한다.

- 반복되는 스타일은 어떻게 줄일 수 있을까?
- 나만의 디자인 시스템을 Tailwind에서 어떻게 정의할 수 있을까?
- 다크모드는 어떻게 처리해야 할까?

이러한 고민에 대해 **@apply**, **theme 확장**, **다크모드 설정** 중심으로 알아본다.

---

## @apply - 클래스 재사용하기

Tailwind의 클래스는 편하지만, 같은 스타일을 여러 컴포넌트에서 반복해서 쓰면 코드가 지저분해질 수 있다.

이럴 땐 전통적인 CSS처럼 클래스를 만들어 재사용할 수 있다:

```css
/* app/globals.css 또는 별도 CSS 파일 */
.btn-primary {
  @apply bg-blue-500 text-white font-bold py-2 px-4 rounded;
}

.btn-primary:hover {
  @apply bg-blue-600;
}
````

이제 컴포넌트에서 이렇게 사용 가능하다:

```jsx
<button className="btn-primary">확인</button>
```

> ✅ 주의: `@apply`는 Tailwind CSS의 PostCSS 플러그인을 통해 처리되므로, `globals.css` 등 PostCSS가 적용된 파일에서만 사용할 수 있다.

---

## 커스텀 테마 확장하기

Tailwind는 기본 색상, 폰트, 간격 등을 제공하지만, **내 브랜드만의 디자인 시스템**을 적용하고 싶다면 테마를 확장하면 된다.

### tailwind.config.js 수정

```js
module.exports = {
  theme: {
    extend: {
      colors: {
        brand: {
          DEFAULT: '#4f46e5',
          light: '#6366f1',
          dark: '#4338ca',
        },
      },
      spacing: {
        '72': '18rem',
        '84': '21rem',
        '96': '24rem',
      },
      fontFamily: {
        sans: ['Pretendard', 'ui-sans-serif', 'system-ui'],
      },
    },
  },
};
```

### 적용 예시

```jsx
<div className="bg-brand text-white p-8 font-sans">
  나만의 브랜드 색상과 폰트!
</div>
```

---

## 다크모드 관련 공식 문서 참고

Tailwind CSS는 `media` 기반 또는 `class` 기반의 다크모드를 기본 지원한다.

> [Tailwind CSS 공식 다크 모드 문서](https://tailwindcss.com/docs/dark-mode)

문서를 참고하면,

* `media`: 사용자의 운영체제 설정에 따라 자동 적용
* `class`: `<html class="dark">` 와 같이 수동 제어 가능 (→ `next-themes`로 구현하기 좋음)

Tailwind는 이 다크모드 기능을 **유틸리티 클래스로 직접 구성할 수 있게 제공**하기 때문에,
이전처럼 별도 CSS 작성 없이도 효율적인 다크 테마 적용이 가능하다.

---

## 다크모드 설정과 활용

Tailwind는 기본적으로 다크모드 지원을 내장하고 있다. 설정은 두 가지 방식 중 하나를 선택할 수 있다:

### 1. `class` 방식 (추천)

```js
// tailwind.config.js
module.exports = {
  darkMode: 'class', // or 'media'
}
```

이 방식은 `<html class="dark">`를 추가함으로써 다크모드를 전환할 수 있다.

```html
<html class="dark">
```

### 스타일 적용

```jsx
<div className="bg-white text-black dark:bg-black dark:text-white">
  🌞 라이트 모드 / 🌚 다크 모드
</div>
```

> 다크모드는 **JS로 클래스 조작**하거나, **`next-themes`** 같은 라이브러리를 활용하여 쉽게 토글 가능하다.

---

## next-themes로 다크모드 토글 구현하기

```bash
npm install next-themes
```

### \_app.tsx 설정

```tsx
import { ThemeProvider } from "next-themes";

function MyApp({ Component, pageProps }) {
  return (
    <ThemeProvider attribute="class">
      <Component {...pageProps} />
    </ThemeProvider>
  );
}
```

### 토글 버튼

```tsx
import { useTheme } from 'next-themes'

export default function DarkModeToggle() {
  const { theme, setTheme } = useTheme()

  return (
    <button onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}>
      {theme === 'dark' ? '☀️' : '🌙'}
    </button>
  );
}
```

---

## 마무리

이번 글에서는 Tailwind CSS를 실제 프로젝트에서 더 강력하게 사용할 수 있는 고급 기능들을 정리해보았다:

* `@apply`로 반복되는 유틸리티 클래스 정리
* `tailwind.config.js`로 나만의 색상, 간격, 폰트 커스텀
* 다크모드 설정 및 사용자 토글 구현
