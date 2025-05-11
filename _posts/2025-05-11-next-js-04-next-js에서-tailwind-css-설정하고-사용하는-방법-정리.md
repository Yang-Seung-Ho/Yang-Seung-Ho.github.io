---
layout: post
title: "[Next.js] 04 - Next.js에서 Tailwind CSS 설정하고 사용하는 방법 정리"
date: 2025-05-10 18:00 +0900

description: Next.js 프로젝트에 Tailwind CSS를 설정하는 방법부터 유틸리티 클래스 사용까지, 스타일링을 빠르게 시작하는 법을 정리했다.
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
published: true
---
## ✨ Tailwind CSS, 왜 쓰는 걸까?

Next.js를 활용해서 프로젝트를 만들다 보면, CSS를 어떻게 관리할지 고민이 생긴다.  
이때 등장하는 것이 바로 **Tailwind CSS**다.

Tailwind는 클래스 기반의 유틸리티 퍼스트 CSS 프레임워크로,  
**빠르게 스타일링**, **일관된 UI 설계**, **컴포넌트화에 최적화**되어 있다.

---

## ⚙️ Tailwind CSS 설치 방법 (with Next.js)


### 1. 프로젝트 생성

```bash
npx create-next-app@latest my-next-tailwind-app
cd my-next-tailwind-app
```

> 옵션은 `--ts`, `--eslint`, `--app` 등 필요한 대로 추가한다.

---

### 2. Tailwind CSS 설치

```bash
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

> 위 명령어를 실행하면 `tailwind.config.js`와 `postcss.config.js` 파일이 자동으로 생성된다.

---

### 3. 설정 파일 수정

`tailwind.config.js` 파일을 열고 아래와 같이 경로를 설정한다:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./app/**/*.{js,ts,jsx,tsx}",
    "./pages/**/*.{js,ts,jsx,tsx}",
    "./components/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```
---

## 💡 전역 스타일 적용하기

`app/globals.css` 파일을 열고 아래와 같이 Tailwind의 기본 지시어를 추가한다:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```


```js
import './globals.css'
```




---

## 🧪 기본 사용 예시

```jsx
export default function Home() {
  return (
    <main className="flex items-center justify-center h-screen bg-gray-100">
      <h1 className="text-4xl font-bold text-blue-600">
        Hello, Tailwind!
      </h1>
    </main>
  )
}
```

Tailwind는 클래스를 조합하는 방식이기 때문에
별도 SCSS 파일 없이도 **바로 반응형**, **다크모드**, **애니메이션** 등 다양한 스타일을 구현할 수 있다.

---

아래는 요청하신 내용을 기반으로 작성한 마크다운 블로그 문단이야. 실수 → 문제 → 해결 → 깨달음 구조로 작성했어:

---

## 🐞 실수와 깨달음: `globals.css`는 어디에 있어야 할까?

처음 Tailwind를 설정할 때, 나는 `globals.css` 파일을 `/styles/globals.css`로 분리해 정리했었다.  
보통 CSS나 SCSS 파일을 `styles` 폴더에 모아두는 습관이 있어서 자연스럽게 그렇게 했던 것이다.

```bash
📁 /styles/globals.css
```

그러나 문제는 **Tailwind 스타일이 전혀 적용되지 않았다는 것**이다.
분명히 `@tailwind base;`, `@tailwind components;`, `@tailwind utilities;`를 모두 넣었고,
`layout.js` 파일에서 `import './styles/globals.css'`도 해줬는데, 아무런 스타일이 나타나지 않았다.

---

### 🔍 문제의 원인

Tailwind가 내부적으로 사용하는 `content` 설정에서,
스타일 지시어가 정의된 파일이 감지되지 않으면 **CSS가 트리에 포함되지 않게 된다.**

즉, 내가 만든 `/styles/globals.css`는 Tailwind의 `content` 경로에 포함되지 않았기 때문에,
실제로는 아무 CSS도 적용되지 않은 것이다.

---

### ✅ 해결 방법

Tailwind 공식 문서를 다시 확인해보니,
`globals.css`는 보통 `app/globals.css` 경로에 두고 사용하는 것을 권장하고 있었다.

```bash
📁 /app/globals.css  ✅ 권장 경로
```

이 위치에 두고 다시 설정한 후, `tailwind.config.js`의 `content` 경로와 일치하게 되면서
드디어 Tailwind 스타일이 정상적으로 작동했다!

---

### 💡 깨달은 점

> Tailwind는 그냥 CSS를 import만 한다고 끝이 아니다.
> **Tailwind의 유틸리티 클래스가 실제 어떤 파일에서 사용되는지 "직접 추적"하기 때문에,
> `tailwind.config.js`의 `content` 경로 안에 들어가야 한다.**

단순한 CSS 파일이라도 **경로가 감지되지 않으면 무시**된다는 점을 이번에 확실히 깨달았다.

앞으로는 스타일 파일의 위치를 정리할 때도,
**빌드 도구(Tailwind, PostCSS 등)의 작동 방식을 염두에 두고 설계**해야겠다는 생각이 들었다.

---

## 🧭 마무리

Next.js + Tailwind 조합은
**정적 렌더링 + 빠른 개발환경 + 유지보수성**까지 확보할 수 있어
매우 인기 있는 프론트엔드 개발 환경이다.

이제부터 컴포넌트마다 유틸리티 클래스를 직접 붙이며
빠르게 스타일을 구현해보자!

> 다음 포스트에서는 Tailwind의 `@apply`, 커스텀 테마, 다크모드 설정 등을 더 자세히 알아볼 예정이다.

