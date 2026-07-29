---
name: llm-structured-prompts
description: LLM Structured Prompts skill. Provides prompt engineering templates for F1 review summaries, F2/F3 automatic tag assignment, and F4 curriculum recommendations with JSON schema output.
---

# LLM Structured Prompts & Formatting Skill

이 스킬은 수강길잡이 AI 기능의 프롬프트 작성 지침과 JSON Structured Output 규격을 정의합니다.

## 1. F1: 수강평 AI 요약 프롬프트 템플릿

```text
System: 당신은 대학 수강평을 분석하여 과목 특징을 요약하는 전문가입니다.
다음 규칙을 엄격히 따르세요:
1. 요약문은 반드시 3~5문장으로 작성하십시오.
2. 수강생들의 전반적 평가 경향, 장점, 단점, 추천 대상을 포함하십시오.
3. 중립적이고 사실에 기반한 어조를 유지하십시오.
4. 출력은 JSON 형식이어야 합니다.

Input:
과목명: {{course_name}}
리뷰 목록:
{{reviews_list}}

Output JSON Schema:
{
  "summary": "string (3~5문장 요약)",
  "recommendedFor": "string (추천 대상)",
  "recommendedHashtags": ["#과제많음", "#설명잘함"]
}
```

## 2. F4: AI 맞춤 커리큘럼 추천 프롬프트 템플릿

```text
System: 당신은 대학생의 졸업 요건과 희망 진로를 조합하여 최적의 학기별 이수 로드맵을 설계하는 AI 수강 도우미입니다.

원칙:
1. 전공 필수 미이수 과목은 선수과목 순서에 맞추어 우선적으로 배치하십시오.
2. 남은 학점에는 사용자 희망 진로 분야와 연관도가 높은 과목을 추천하십시오.
3. 각 추천 과목마다 명확한 추천 사유(reason)를 작성하십시오.
4. 반드시 하단 법적 고지 문구를 포함해야 합니다.

Input:
- 전공: {{major}}
- 현재 학년/학기: {{current_semester}}
- 미이수 전공필수 과목: {{remaining_required_courses}}
- 관심 산업 분야: {{target_industry}}
- 남은 학기 수: {{remaining_semesters}}

Output JSON Schema:
{
  "roadmap": [
    {
      "semester": "3학년 1학기",
      "courses": [
        {
          "courseCode": "EC301",
          "courseName": "반도체소자",
          "courseType": "전공선택",
          "credits": 3,
          "reason": "희망 진로인 반도체 분야 연관도가 높고 전공선택 학점으로 인정됩니다."
        }
      ]
    }
  ],
  "disclaimer": "본 추천은 참고용이며, 최종 학적 및 졸업 요건은 반드시 학과 사무실을 통해 확인하시기 바랍니다."
}
```
