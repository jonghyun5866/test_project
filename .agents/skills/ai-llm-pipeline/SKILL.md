---
name: ai-llm-pipeline
description: Skill for AI/LLM integration pipelines. Handles F1 review summarization & caching, F2/F3 field classification & embedding generation, and F4 curriculum optimization algorithms.
---

# AI & LLM Feature Pipeline Skill

이 스킬은 수강길잡이 서비스의 4대 핵심 AI 기능(F1, F2, F3, F4)의 파이프라인 설계, 프롬프트 엔지니어링, 결과 캐싱 전략을 다룹니다.

## 1. F1: 수강평 요약 캐싱 파이프라인

리뷰가 작성될 때마다 매번 LLM API를 호출하면 비용 및 대기시간이 상승하므로 캐싱 전략을 적용합니다.

```typescript
// lib/ai/summary.ts
export async function getOrGenerateCourseSummary(courseId: number) {
  const pool = getDbPool();
  
  // 1. 기존 캐시된 요약 조회
  const cached = await pool.query(
    `SELECT summary, review_count, updated_at FROM summaries WHERE course_id = $1`,
    [courseId]
  );

  // 2. 누적 리뷰 개수 확인
  const currentReviews = await pool.query(
    `SELECT COUNT(*) as count FROM reviews WHERE course_id = $1`,
    [courseId]
  );
  const totalCount = parseInt(currentReviews.rows[0].count, 10);

  // 리뷰 5개 미만 시 미제공 문구 반환
  if (totalCount < 5) {
    return { summary: "수강평이 아직 충분하지 않습니다 (최소 5개 필요).", hashtags: [] };
  }

  // 리뷰가 5개 이상 늘어났거나 기존 캐시가 없으면 재생성
  if (!cached.rows.length || totalCount - cached.rows[0].review_count >= 5) {
    const newSummary = await generateSummaryWithLLM(courseId);
    await pool.query(
      `INSERT INTO summaries (course_id, summary, review_count, updated_at) 
       VALUES ($1, $2, $3, NOW())
       ON CONFLICT (course_id) DO UPDATE SET summary = $2, review_count = $3, updated_at = NOW()`,
      [courseId, newSummary.text, totalCount]
    );
    return newSummary;
  }

  return { summary: cached.rows[0].summary };
}
```

## 2. F2 / F3: 자동 분야 분류 & Vector Embedding 생성

강의계획서와 과목명을 입력받아 임베딩을 생성하고 pgvector 컬럼에 업데이트합니다.

```typescript
// lib/ai/embedding.ts
export async function updateCourseEmbedding(courseId: number, name: string, syllabus: string) {
  const textToEmbed = `과목명: ${name}\n강의계획서: ${syllabus}`;
  
  // 외부 LLM API 또는 Embedding Provider 호출 예시
  const embeddingVector = await fetchEmbedding(textToEmbed);

  const pool = getDbPool();
  await pool.query(
    `UPDATE courses SET embedding = $1::vector WHERE id = $2`,
    [JSON.stringify(embeddingVector), courseId]
  );
}
```

## 3. F4: 커리큘럼 추천 알고리즘 (Hybrid Flow)

1. **규칙 기반 엔진 (Deterministic)**:
   - 미이수 전공필수(Required Major) 과목을 선수과목 체계에 맞게 우선 학기별 분배.
   - 전공선택 잔여 학점 계산.
2. **LLM/Vector 기반 엔진 (Probabilistic)**:
   - 관심 산업 분야(예: "반도체")와 `pgvector` 코사인 유사도가 높은 과목 추천.
   - 본인 전공 과목을 우선 추천 후 잔여 범위는 타 전공 과목 포함.
3. **법적/행정적 유의사항 명시**:
   - 모든 추천 결과 하단에 *"본 추천 커리큘럼은 참고용이며, 최종 졸업 요건은 반드시 학과 사무실 및 교무처를 통해 확인하시기 바랍니다."* 안내 문구 필수 포함.
