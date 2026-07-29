---
name: nextjs-app-router
description: Skill for Next.js App Router development. Covers React Server Components, Server Actions, Route Handlers, client-side interactivity, metadata, and modern UI integration.
---

# Next.js App Router Development Skill

이 스킬은 수강길잡이 애플리케이션의 프론트엔드/백엔드 통합 프레임워크인 Next.js App Router 구조와 컴포넌트 개발 지침을 제공합니다.

## 1. Directory Structure

```text
app/
├── (auth)/
│   └── login/
├── courses/
│   ├── page.tsx               # 과목 목록 및 F2/F3 검색 UI (RSC)
│   └── [id]/
│       └── page.tsx           # 과목 상세 & F1 AI 요약/해시태그 (RSC)
├── curriculum/
│   └── page.tsx               # F4 맞춤 커리큘럼 설계 인터랙티브 페이지 ('use client')
├── api/
│   ├── reviews/route.ts       # 수강평 등록 및 F1 요약 생성 트리거
│   ├── search/route.ts        # F2/F3 통합 검색 Route Handler
│   └── curriculum/route.ts    # F4 AI 추천 연산 Route Handler
├── layout.tsx
└── page.tsx
```

## 2. Server Components vs Client Components

- **기본 모드**: 모든 `page.tsx` 및 `layout.tsx`는 서버 컴포넌트(RSC)로 작성하여 초기 HTML 빠른 렌더링 및 SEO를 확보합니다.
- **클라이언트 모드 (`'use client'`)**:
  - 수강평 별점/해시태그 선택 폼 (`ReviewForm.tsx`)
  - 과목 검색어 필터링 입력창 (`SearchFilter.tsx`)
  - 커리큘럼 과목 제외/추가 인터랙티브 로드맵 뷰어 (`CurriculumInteractiveBoard.tsx`)

## 3. Server Actions 활용예시 (수강평 작성)

```typescript
// app/actions/review.ts
'use server'

import { getDbPool } from '@/lib/db';
import { revalidatePath } from 'next/cache';

export async function submitReview(formData: FormData) {
  const courseId = formData.get('courseId');
  const rating = Number(formData.get('rating'));
  const content = formData.get('content') as string;
  const tags = formData.getAll('tags') as string[];

  const pool = getDbPool();
  await pool.query(
    `INSERT INTO reviews (course_id, rating, content, tags) VALUES ($1, $2, $3, $4)`,
    [courseId, rating, content, JSON.stringify(tags)]
  );

  // 해당 과목 상세 페이지 캐시 갱신
  revalidatePath(`/courses/${courseId}`);
  return { success: true };
}
```

## 4. UX 및 SEO 가이드라인
- 모든 페이지에 `generateMetadata()`를 사용해 동적 title, description 세팅.
- Loading UI는 `loading.tsx` 및 `React.Suspense`를 적용해 스켈레톤 UI를 사용자에게 제공.
