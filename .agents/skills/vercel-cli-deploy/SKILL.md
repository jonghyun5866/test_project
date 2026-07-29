---
name: vercel-cli-deploy
description: Official Vercel CLI deployment and project configuration skill. Handles project setup, vercel.json configurations, build/preview deployment commands, environment variables, and domain bindings.
---

# Vercel CLI & Deployment Skill

이 스킬은 Vercel 플랫폼상에서의 프로젝트 빌드, 프리뷰 배포, 프로덕션 배포, 환경 변수 동기화 및 `vercel.json` 제어를 전담합니다.

## 1. Vercel CLI 기본 명령어

### 프로젝트 연결 및 초기화
```bash
# 프로젝트 초기화 및 Vercel 계정/팀 연결
npx vercel link

# 환경 변수 로컬 다운로드 (.env.local)
npx vercel env pull .env.local
```

### 배포 실행
```bash
# 프리뷰 배포 (Preview Deployment 생성)
npx vercel

# 프로덕션 배포 (Production Deployment)
npx vercel --prod
```

## 2. Vercel 설정 파일 (`vercel.json`) 가이드

프로젝트 루트에 `vercel.json`을 추가하여 백엔드 라우트 및 타임아웃, 리다이렉트 규칙을 제어할 수 있습니다.

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "framework": "nextjs",
  "buildCommand": "next build",
  "devCommand": "next dev",
  "installCommand": "npm install",
  "functions": {
    "app/api/**/*": {
      "maxDuration": 60,
      "memory": 1024
    }
  },
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        { "key": "Access-Control-Allow-Credentials", "value": "true" },
        { "key": "Access-Control-Allow-Origin", "value": "*" }
      ]
    }
  ]
}
```

## 3. 프리뷰 배포 & Neon DB 브랜칭 연동

Vercel 프리뷰 배포(Pull Request 연동) 시 Neon DB의 데이터베이스 브랜칭 기능을 함께 활용하면 실 상용 데이터에 영향 없이 완전 격리된 테스트 환경을 구축할 수 있습니다.

1. **Vercel Integration**: GitHub PR 생성 시 Vercel이 Preview URL을 자동 생성합니다.
2. **Environment Variables**: Preview 환경용 `DATABASE_URL`에 Neon Preview Branch의 접속 문자열을 매핑합니다.
3. **Build Command Verification**: `npx vercel build`로 로컬 빌드 검증 후 배포를 추진합니다.

## 4. 체크리스트
- [ ] Vercel CLI 실행 시 `npx vercel` 권한 및 토큰 검증
- [ ] `.env.production` 및 `.env.preview` 분리 적용
- [ ] Route Handler 타임아웃(`maxDuration`) 초과 여부 점검
