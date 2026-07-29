---
name: neon-postgres-pgvector
description: Skill for Neon Serverless Postgres integration, schema management, pgvector similarity search, connection pooling, and Vercel DB branching.
---

# Neon Serverless Postgres & pgvector Skill

이 스킬은 수강길잡이 서비스의 Neon Serverless Postgres 데이터베이스 설계, `@neondatabase/serverless` 커넥션 풀링, `pgvector` 기반 코사인 유사도 검색, 프리뷰 DB 브랜칭 관리를 수행합니다.

## 1. Neon Serverless Driver 설정

Vercel 서버리스 환경에서 WebSocket 기반의 효율적인 DB 쿼리를 위해 `@neondatabase/serverless` 패키지를 활용합니다.

```typescript
// lib/db.ts
import { Pool, neonConfig } from '@neondatabase/serverless';
import ws from 'ws';

// Node.js 환경에서 WebSocket 바인딩 설정
neonConfig.webSocketConstructor = ws;

// 커넥션 풀 싱글톤 패턴 적용
let pool: Pool;

export function getDbPool() {
  if (!pool) {
    pool = new Pool({ connectionString: process.env.DATABASE_URL });
  }
  return pool;
}
```

## 2. pgvector 확장 및 엔티티 스키마

### `pgvector` 활성화 SQL
```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

### 핵심 테이블 및 Vector 컬럼 (F3 산업/진로 키워드 검색용)
```sql
-- 과목 테이블 (Course)
CREATE TABLE courses (
  id SERIAL PRIMARY KEY,
  code VARCHAR(50) NOT NULL UNIQUE,
  name VARCHAR(150) NOT NULL,
  department VARCHAR(100) NOT NULL,
  credits INT NOT NULL DEFAULT 3,
  course_type VARCHAR(20) NOT NULL, -- '전필', '전선', '교양'
  syllabus TEXT,
  embedding vector(1536), -- Open AI / LLM 임베딩 차원
  created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- 코사인 유사도 인덱스 생성 (HNSW 인덱스)
CREATE INDEX idx_courses_embedding ON courses USING hnsw (embedding vector_cosine_ops);
```

## 3. F2 / F3 유사도 검색 SQL 쿼리 예시

```sql
-- 입력받은 관심 분야/산업 임베딩 vector와 가장 연관도가 높은 과목 Top 10 검색
SELECT 
  id, 
  code, 
  name, 
  department, 
  credits, 
  course_type,
  1 - (embedding <=> $1::vector) AS similarity_score
FROM courses
WHERE 1 - (embedding <=> $1::vector) > 0.6
ORDER BY similarity_score DESC
LIMIT 10;
```

## 4. Neon DB Branching 수칙

- **메인 브랜치 (`main`)**: Production 전용 데이터베이스
- **개발 브랜치 (`dev`)**: 통합 개발 및 로컬 테스트용 DB
- **Vercel Preview 브랜치 (`preview/pr-*`)**: Pull Request 오픈 시 자동 분기 및 빌드 테스트 완료 후 삭제
