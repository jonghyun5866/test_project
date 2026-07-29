---
name: anti-abuse-security
description: Anti-abuse and Security skill. Provides abuse detection rules for reviews, spam/rating terror filtering, anonymous user session hashing, and API rate limiting.
---

# Anti-Abuse & Security Skill

수강길잡이의 수강평 데이터 신뢰도를 높이고 어뷰징(도배, 평점 테러)을 방지하기 위한 보안 및 세션 관리 규칙입니다.

## 1. 수강평 어뷰징 및 평점 테러 탐지 로직 (F1)

1. **동일 과목 중복 작성 금지**:
   - 익명 식별자 Hash(User IP + Salt + 학기 정보)를 이용해 학기당 동일 과목에 대해 1회만 수강평 작성을 허용합니다.
2. **이상 텍스트 패턴 필터링**:
   - 10자 미만의 단순 반복 텍스트("ㅋㅋㅋㅋㅋ", "굿굿굿굿")는 AI 요약 데이터셋에서 자동 제외합니다.
   - 단시간(예: 1분 이내) 내 동일 익명 Hash로 작성된 연속 리뷰는 임시 보류(Pending) 처리합니다.
3. **평점 테러 감지 규칙**:
   - 전체 평균 평점 대비 특정 기간 동안 1점 리뷰가 폭증할 경우 AI 요약 생성 시 "최근 평점 변동성이 높음" 뱃지를 부여하고 자동 이상 탐지 알림을 기록합니다.

## 2. API Rate Limiting (Vercel Edge / Route Handlers)

```typescript
// middleware.ts 또는 lib/rate-limit.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

const ipCache = new Map<string, { count: number; expiresAt: number }>();

export function checkRateLimit(ip: string, limit = 10, windowMs = 60000): boolean {
  const now = Date.now();
  const record = ipCache.get(ip);

  if (!record || record.expiresAt < now) {
    ipCache.set(ip, { count: 1, expiresAt: now + windowMs });
    return true;
  }

  if (record.count >= limit) {
    return false; // Rate limit 초과
  }

  record.count += 1;
  return true;
}
```

## 3. 개인정보 보호 (Privacy & Anonymity)
- 사용자 성명, 학번 등 식별 정보는 DB에 직접 저장하지 않으며 Hash 기반 단방향 식별자만 보관합니다.
