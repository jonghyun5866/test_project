---
name: ui-ux-design-system
description: UI/UX Design System skill for Course Helper. Includes responsive layout, modern typography, course cards, hashtag badges, star ratings, and interactive curriculum roadmap board.
---

# Course Helper UI/UX Design System Skill

이 스킬은 수강길잡이 서비스의 사용자 인터페이스(UI) 디자인 시스템 및 인터랙션 컴포넌트 설계 지침을 정의합니다.

## 1. 디자인 컨셉 및 컬러 팔레트

- **핵심 컨셉**: 대학생을 위한 스마트하고 신뢰감 있는 인터페이스. 대학교 커뮤니티 특유의 복잡함을 제거하고 프리미엄 글래스모피즘(Glassmorphism)과 반응형 카드를 적용.
- **컬러 팔레트**:
  - Primary: `#4F46E5` (Indigo-600) — 브랜드 주 색상 (버튼, 강조 텍스트)
  - Secondary: `#06B6D4` (Cyan-500) — AI 기능 및 해시태그 강조
  - Accent: `#F59E0B` (Amber-500) — 별점(Rating) 및 중요 안내
  - Background: Dark Mode (`#0F172A` Slate-900) / Light Mode (`#F8FAFC` Slate-50)

## 2. 주요 UI 컴포넌트 규격

### 2.1 수강평 카드 및 해시태그 뱃지 (`CourseReviewCard.tsx`)
- **과목 정보**: 과목명, 과목코드, 학점, 개설학과, 평점(5점 만점 별점 visual)
- **AI 요약 박스**: Cyan 그라데이션 테두리와 뱃지(`AI 요약`) 적용.
- **해시태그 비율 시각화**: `%` 게이지 바 포함 (예: `#과제많음 72%`, `#설명잘함 65%`).

### 2.2 F4 커리큘럼 로드맵 인터랙티브 보드 (`CurriculumBoard.tsx`)
- **학기별 드래그/셀렉트 뷰**: 1-1학기부터 4-2학기까지 칸반(Kanban) 스타일의 카드 배치.
- **이수 구분 뱃지**:
  - `전공필수`: Red Accent (`#EF4444`)
  - `전공선택`: Blue Accent (`#3B82F6`)
  - `관심분야(자유선택/타전공)`: Purple Accent (`#8B5CF6`)
- **추천 사유 툴팁**: 각 과목 마우스 호버/클릭 시 추천 사유 팝업 표시.

## 3. 웹 접근성 (Accessibility) & SEO
- 모든 검색 폼 및 필터링 컨트롤에는 고유 `id`와 `aria-label`을 부여.
- 폰트는 Pretendard 또는 Noto Sans KR을 기본 지정하여 한국어 가독성 극대화.
