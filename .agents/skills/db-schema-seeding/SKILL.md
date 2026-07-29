---
name: db-schema-seeding
description: Database Schema & Seed Data skill. Defines Drizzle/Prisma models for PRD 7 entities and provides seed data scripts for university courses and graduation requirements.
---

# DB Schema & Seed Data Skill

이 스킬은 PRD의 7대 핵심 엔티티(`Course`, `FieldTag`, `IndustryTag`, `Review`, `Summary`, `User`, `Curriculum`) DB 스키마 정의 및 초기 테스트용 더미 시드 데이터 인서트 가이드를 제공합니다.

## 1. 7대 엔티티 관계도 (ERD 요약)

```text
User ──── (1:N) ──── Review ──── (N:1) ──── Course ──── (1:1) ──── Summary
                                               │
                                       (N:M Tags Join)
                                               │
                                 FieldTag / IndustryTag
```

## 2. Drizzle ORM 스키마 예시 (`schema.ts`)

```typescript
import { pgTable, serial, text, integer, timestamp, jsonb, customType } from 'drizzle-orm/pg-core';

// pgvector 지원을 위한 customType 정의
const vector = customType<{ data: number[] }>({
  dataType() { return 'vector(1536)'; },
});

export const courses = pgTable('courses', {
  id: serial('id').primaryKey(),
  code: text('code').notNull().unique(),
  name: text('name').notNull(),
  department: text('department').notNull(),
  credits: integer('credits').notNull().default(3),
  courseType: text('course_type').notNull(), // '전필', '전선', '교양'
  syllabus: text('syllabus'),
  embedding: vector('embedding'),
  createdAt: timestamp('created_at').defaultNow(),
});

export const reviews = pgTable('reviews', {
  id: serial('id').primaryKey(),
  courseId: integer('course_id').references(() => courses.id).notNull(),
  authorHash: text('author_hash').notNull(),
  rating: integer('rating').notNull(),
  content: text('content').notNull(),
  tags: jsonb('tags').notNull(), // 선택 및 AI 추천 해시태그 목록
  createdAt: timestamp('created_at').defaultNow(),
});

export const curriculums = pgTable('curriculums', {
  id: serial('id').primaryKey(),
  department: text('department').notNull(),
  startYear: integer('start_year').notNull(),
  requiredCourses: jsonb('required_courses').notNull(), // 전공필수 과목 ID 배열
  minElectiveCredits: integer('min_elective_credits').notNull(), // 전공선택 최소학점
  totalCredits: integer('total_credits').notNull().default(130),
});
```

## 3. 테스트용 시드 데이터 인서트 가이드 (`seed.ts`)

- **전자공학과 / 컴퓨터공학과 샘플 과목**: 미적분학1, 일반물리학, 데이터구조, 반도체공정개론, 반도체소자, AI개론 등 20개 과목 인서트.
- **졸업요건 더미 데이터**: 전자공학과 2024학번 (전필 21학점, 전선 42학점, 총 130학점 요건).
- **시드 실행 명령**: `npx tsx scripts/seed.ts`
