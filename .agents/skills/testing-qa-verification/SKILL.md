---
name: testing-qa-verification
description: Testing and Quality Assurance skill. Covers unit tests for F1 caching logic, vector search precision in F2/F3, curriculum requirement calculation in F4, and integration test setup.
---

# Testing & Quality Assurance (QA) Skill

이 스킬은 수강길잡이 서비스의 로직 검증, 알고리즘 단위 테스트 및 API 통합 테스트 지침을 정의합니다.

## 1. 주요 테스트 대상 및 검증 기준

| ID | 테스트 항목 | 검증 방식 | 합격 기준 |
| --- | --- | --- | --- |
| **F1** | 요약 캐싱 파이프라인 | 단위 테스트 (Vitest) | 리뷰 < 5개 시 요약 미생성, 리뷰 5개 이상 축적 시 캐시 자동 갱신 검증 |
| **F2/F3** | Vector Similarity 검색 | SQL / API 테스트 | 과목 설명에 키워드가 직접 없더라도 코사인 유사도 0.6 이상 과목 정상 리턴 |
| **F4** | 커리큘럼 이수요건 계산 | 로직 단위 테스트 | 선수과목 미이수 시 다음 학기로 배치 방지, 전공필수 우선 배치 검증 |
| **Build** | Next.js 빌드 및 린트 | `npm run build` | TypeScript 에러 0건, 린트 워닝 0건 |

## 2. F4 커리큘럼 이수요건 단위 테스트 예시 (`curriculum.test.ts`)

```typescript
import { describe, it, expect } from 'vitest';
import { planCurriculum } from '@/lib/curriculum-engine';

describe('F4 Curriculum Engine Tests', () => {
  it('전공필수 과목이 미이수 상태일 때 우선 배치되어야 한다', () => {
    const input = {
      major: '전자공학과',
      completedCourses: ['미적분학1'],
      requiredCourses: ['미적분학1', '회로이론1', '전자회로1'],
      targetIndustry: '반도체',
      remainingSemesters: 4,
    };

    const result = planCurriculum(input);
    const firstSemesterCourses = result.roadmap[0].courses.map((c) => c.courseName);
    
    // 회로이론1이 1번째 학기에 우선 포함되어야 함
    expect(firstSemesterCourses).toContain('회로이론1');
  });
});
```

## 3. CI/CD 검증 명령어
```bash
# 타입 체크 및 린트
npx tsc --noEmit
npm run lint

# 단위 테스트 실행
npx vitest run
```
