---
name: vercel-serverless-optimization
description: Optimization skill for Vercel Serverless & Edge functions. Covers timeout mitigation, streaming AI responses for Next.js Route Handlers, payload management, connection reuse, and cold-start optimization.
---

# Vercel Serverless & Edge Optimization Skill

수강길잡이 서비스의 AI 연산(F1 요약, F3 유사도 검색, F4 커리큘럼 추천)은 서버리스 환경에서 실행되며, Vercel의 실행 시간 한계(Hobbs/Pro plan maxDuration) 및 타임아웃에 최적화되어야 합니다.

## 1. 타임아웃 방지 및 스트리밍 (Streaming AI Response)

Next.js App Router Route Handler에서 LLM 응답 시간이 길어지는 F4 맞춤 커리큘럼 생성 시 HTTP Streaming을 사용하여 타임아웃을 방지합니다.

```typescript
// app/api/curriculum/recommend/route.ts
import { NextResponse } from 'next/server';

export const maxDuration = 60; // Vercel Route Handler 최대 실행시간 (초)
export const dynamic = 'force-dynamic';

export async function POST(req: Request) {
  const { major, userCourses, targetIndustry } = await req.json();

  // Web ReadableStream을 활용한 스트리밍 응답
  const encoder = new TextEncoder();
  const stream = new ReadableStream({
    async start(controller) {
      controller.enqueue(encoder.encode(JSON.stringify({ step: 'INITIALIZING', message: '이수 요건 분석 중...' }) + '\n'));
      
      // 1단계: 전공 필수 배치 연산
      // ...
      controller.enqueue(encoder.encode(JSON.stringify({ step: 'REQUIRED_COURSES', data: [] }) + '\n'));

      // 2단계: LLM 기반 관심 분야 과목 추천
      // ...
      controller.enqueue(encoder.encode(JSON.stringify({ step: 'RECOMMENDING', data: [] }) + '\n'));

      controller.close();
    },
  });

  return new Response(stream, {
    headers: {
      'Content-Type': 'application/x-ndjson',
      'Transfer-Encoding': 'chunked',
    },
  });
}
```

## 2. 엣지 함수(Edge Runtime) vs 서버리스(Serverless Node.js) 선택 기준

| 분류 | Edge Runtime (`runtime = 'edge'`) | Serverless Node.js (`runtime = 'nodejs'`) |
| --- | --- | --- |
| **적합 기능** | F2 (분야 키워드 빠른 검색, 헤더 처리) | F1 (리뷰 요약 Caching), F3/F4 (Complex LLM & Heavy DB) |
| **시작 시간** | 0ms에 가까움 (Cold start 최소화) | 약간의 Cold start 존재 (Neon 커넥션 풀링 필요) |
| **Node API** | 패키지 제한 (Web Standard API만 지원) | 모든 Node.js 모듈 및 Heavy ORM 지원 |

```typescript
// Edge Runtime 선언 예시 (F2 검색 라우트)
export const runtime = 'edge';
```

## 3. 콜드 스타트(Cold Start) & DB 풀링 최적화

- **HTTP/WebSocket 연결**: Neon DB 연결 시 글로벌 스코프에 연결 객체를 선언하여 재사용합니다.
- **Payload 제한**: 요청 body는 4.5MB 이내로 유지하고, 수강평 데이터 등 대용량 데이터 쿼리는 페이징(Limit/Offset)을 의무 적용합니다.
