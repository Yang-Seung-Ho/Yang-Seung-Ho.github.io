---
layout: post
title: "[Next.js] 06 - Next.js 네트워크 구조 완전 정복"
date: 2025-05-23 20:36 +0900
description: Next.js의 CSR, SSR, SSG, ISR 렌더링 전략과 API 요청 흐름, prefetch 등 네트워크 구조를 실전 중심으로 정리한 글입니다. 프론트엔드 개발자가 알아야 할 Next.js의 네트워크 작동 원리를 완전히 이해할 수 있습니다.
categories:
  - Dev
  - Next.js
tags:
  - Next.js
  - 프론트엔드
  - 네트워크구조
  - 개발환경  
published: true
---

# Next.js 네트워크 구조 완전 정복

Next.js를 학습하거나 실무에서 사용할 때, **"렌더링 방식"** 과 **"데이터 요청 흐름"** 을 제대로 이해하는 것은 굉장히 중요하다.

이 글에서는 **Next.js가 프론트엔드에서 어떤 네트워크 구조를 따르고 있고**, 그 안에서 **SSR, SSG, CSR, ISR** 이 어떤 방식으로 동작하는지, **어떤 상황에서 어떤 전략이 쓰이는지** 실전 중심으로 정리해본다.

---

## 1. Next.js의 렌더링 전략 4종 비교

| 전략  | 설명              | 데이터 시점          | HTML 생성 시점 | SEO | 속도       |
| --- | --------------- | --------------- | ---------- | --- | -------- |
| CSR | 클라이언트에서 렌더링     | 브라우저 로드 후 JS 실행 | 클라이언트      | ❌   | 느림       |
| SSR | 서버에서 매 요청마다 렌더링 | 요청 시마다          | 서버         | ✅   | 중간       |
| SSG | 정적 파일로 HTML 생성  | 빌드 시            | 서버 (빌드시)   | ✅   | 빠름       |
| ISR | SSG + 주기적 업데이트  | 재검증 주기마다        | 서버         | ✅   | 빠름 + 최신화 |

---

## 2. 요청 흐름: 클라이언트 vs 서버 컴포넌트

### 서버 컴포넌트에서 `fetch()`

```tsx
// 서버 컴포넌트에서 fetch
export default async function Page() {
  const res = await fetch('https://api.example.com/posts');
  const posts = await res.json();

  return <PostList posts={posts} />;
}
```

* 브라우저에선 **네트워크 요청이 발생하지 않음**
* HTML 생성 시 서버에서 API 요청을 수행하고, 정적인 HTML로 응답함
* 사용자 입장에서는 **빠르고 깔끔한 페이지 초기 렌더링**

### 클라이언트 컴포넌트에서 `fetch()`

```tsx
'use client'
import { useEffect, useState } from 'react'

export default function PostList() {
  const [posts, setPosts] = useState([])

  useEffect(() => {
    fetch('/api/posts')
      .then(res => res.json())
      .then(data => setPosts(data))
  }, [])

  return <>{posts.map(p => <div key={p.id}>{p.title}</div>)}</>
}
```

* CSR 방식
* 사용자는 페이지를 먼저 보고 → 이후 데이터가 늦게 렌더링됨

---

## 3. getServerSideProps vs getStaticProps

### `getServerSideProps` (SSR)

* **매 요청마다 서버에서 실행**
* 로그인 상태, 실시간 데이터 등 자주 바뀌는 정보에 유리

```tsx
export async function getServerSideProps() {
  const res = await fetch('https://api.example.com/data')
  const data = await res.json()

  return { props: { data } }
}
```

### `getStaticProps` (SSG)

* **빌드 타임에 한 번 실행**되어 정적 HTML 생성
* 블로그, 마케팅 페이지, 문서 등에 적합

```tsx
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts')
  const posts = await res.json()

  return { props: { posts } }
}
```

---

## 4. ISR (Incremental Static Regeneration)

* 정적 페이지를 만들되, 일정 시간마다 백그라운드에서 재생성
* `revalidate` 옵션으로 주기 설정 가능

```tsx
export async function getStaticProps() {
  const res = await fetch('https://api.example.com/posts')
  const posts = await res.json()

  return {
    props: { posts },
    revalidate: 60, // 60초마다 페이지 갱신
  }
}
```

---

## 5. Next.js의 API 라우트

* `/pages/api/` 디렉토리에 작성
* 클라이언트에서 fetch로 접근 가능 (`/api/hello` 등)
* 서버에서만 실행됨 (DB, 인증 처리 가능)

```ts
// pages/api/hello.ts
export default function handler(req, res) {
  res.status(200).json({ message: 'Hello from API!' })
}
```

---

## 6. 프리페치(prefetch)와 페이지 전환 네트워크 흐름

Next.js의 `<Link>` 컴포넌트는 페이지 전환을 빠르게 하기 위해 **백그라운드에서 HTML/JS 데이터를 미리 로드**한다.

```tsx
import Link from 'next/link'

<Link href="/about">About 페이지로 이동</Link>
```

→ 사용자 hover 시, 이미 리소스를 로드해두므로 페이지 전환이 빠르다.

---

## 마무리 요약

| 항목        | CSR   | SSR  | SSG    | ISR     |
| --------- | ----- | ---- | ------ | ------- |
| 초기 로딩 속도  | 느림    | 빠름   | 매우 빠름  | 빠름 + 갱신 |
| SEO       | ❌     | ✅    | ✅      | ✅       |
| API 요청 시점 | 클라이언트 | 서버   | 빌드 타임  | 빌드 + 갱신 |
| 페이지 재생성   | X     | 매 요청 | 빌드시 1회 | 일정 주기마다 |

---

## 이 글을 통해 알 수 있는 것

* Next.js의 데이터 요청 방식에 따라 어떤 네트워크 흐름이 발생하는지
* CSR/SSR/SSG/ISR 각각의 구조적 차이점
* 실제 사용 예시와 코드 중심으로, 언제 어떤 전략을 써야 하는지 명확하게 이해할 수 있다
