# 1인 개발자 아이디어 생성 시스템 v11

Claude Code의 **sub-agents** 기능을 활용해서 글로벌 페인 포인트 기반으로 수익형 웹서비스 아이디어를 생성하는 시스템.

## v10 대비 핵심 가치

| 영역 | v10 (단일 흐름) | v11 (sub-agents) |
|------|-----------------|-------------------|
| 검색 노이즈 | 메인 컨텍스트에 누적 | market-researcher 컨텍스트에 격리 |
| 자기 옹호 | LLM이 자기가 만든 아이디어를 평가 | red-team이 별도 컨텍스트에서 비판 |
| 채점 객관성 | 같은 LLM이 채점 (편향) | scorer가 별도 컨텍스트에서 객관 채점 |
| 토큰 효율 | 모든 단계가 하나의 컨텍스트 (200K 빠르게 소진) | 작업별 격리 (메인 컨텍스트 절약) |
| 병렬 실행 | 불가 | 가능 (Claude Code 최대 10개 동시) |

## 파일 구조

```
v11/
├── README.md                       # 이 파일
├── main-prompt.md                  # 사용자가 Claude Code에 붙여넣을 메인 프롬프트
└── .claude/
    └── agents/
        ├── market-researcher.md    # 페인 포인트 수집 sub-agent
        ├── red-team.md             # 가혹한 비판자 sub-agent
        └── scorer.md               # 100점 만점 객관 채점 sub-agent
```

## 설치

### 옵션 A: 프로젝트 레벨 (권장, 이 프로젝트에서만)

1. Claude Code가 설치되어 있는지 확인 (https://claude.com/code)
2. 프로젝트 디렉토리로 이동
3. 이 폴더의 `.claude/agents/` 를 프로젝트 루트로 복사:
   ```bash
   cp -r v11/.claude /path/to/your/project/
   ```
4. `main-prompt.md`의 코드 블록 내용을 메모해두기 (Claude Code 세션에서 붙여넣을 것)

### 옵션 B: 글로벌 (모든 프로젝트에서)

```bash
mkdir -p ~/.claude/agents
cp v11/.claude/agents/*.md ~/.claude/agents/
```

## 사용

1. 프로젝트 디렉토리에서 `claude` 실행
2. `main-prompt.md`의 프롬프트 블록 복사 → 붙여넣기
3. Phase 0 질문에 답 (타겟 유형, 우선 지역, 이전 실패)
4. Claude가 sub-agents를 자동 또는 명시적으로 호출하면서 6단계 진행
5. TOP 3 + MVP 명세 + 48h 검증 플랜 출력

## Sub-agents 역할 요약

### market-researcher
- **임무**: 글로벌 커뮤니티(Reddit, HN, IH, Disquiet, 한국·일본·대만·동남아 등)에서 실제 페인 포인트 20+개 수집
- **도구**: WebSearch, WebFetch
- **격리 가치**: 검색 결과의 노이즈가 메인 컨텍스트를 오염시키지 않음. 메인은 정리된 표만 받음
- **핵심 규칙**: 학습 데이터 추론 금지, 모든 페인 포인트는 출처 URL+인용 동반

### red-team
- **임무**: 각 아이디어에 5가지 kill 사유 발굴. 균형 시도 금지, 격려 금지
- **도구**: WebSearch, WebFetch (증거 검증용)
- **격리 가치**: 메인이 아이디어 생성에 들인 노력을 모름 → 자기 옹호 없이 비판 가능
- **핵심 규칙**: 3개 이상 반박 불가 시 KILL 권고

### scorer
- **임무**: 5개 항목 × 100점 만점 객관 채점 (페인 포인트 강도, 수익화 명확성, 블루오션, 실행 용이성, 데이터/API 안정성)
- **도구**: WebSearch (증거 검증용)
- **격리 가치**: 메인이 어느 아이디어를 선호하는지 모름 → 편향 없는 채점
- **핵심 규칙**: 모든 점수가 비슷하면 채점 회피로 간주, 재채점

## v11이 적합한 환경

- ✅ Claude Code (CLI)
- ✅ Sub-agents 기능 활성화 (v1.0.60+)

## v10을 사용해야 할 환경

- ❌ Claude.ai 웹/모바일 (sub-agents 없음)
- ❌ Anthropic API 직접 사용 (수동 구현 필요)
- ❌ Sub-agents 비활성화된 환경

## 트러블슈팅

### Sub-agent가 자동 호출되지 않음
프롬프트에 명시적 호출 추가:
```
Use market-researcher subagent with input: B2C, 한국+영어
```

### 페인 포인트가 20개 미만
market-researcher 재호출. 도메인 키워드를 변경해서 다시 검색.

### 모든 아이디어 점수가 비슷함
scorer가 채점을 회피한 것. 재호출하고 "점수 분산 필수"를 명시.

### Red Team이 너무 가혹해서 모두 KILL
정상 동작. 메인이 반박 가능한 아이디어만 살아남도록 설계됨. 아이디어 풀을 더 만들어서 재시도.

## 관련 프롬프트

- `idea-generation-prompt-v10.md`: Claude Code 외 환경용 단일 흐름 버전
- `prototype-validation-prompt-v3.md`: 프로토타입 단계 수익성 검증 (PERSEVERE/PIVOT/KILL)

## 누적 변경 이력 (요약)

- v11: Claude Code sub-agents로 자기 옹호 차단 + 컨텍스트 격리
- v10: 모든 추상어 정량화, 검색 명령어 사이트 한정, 100점 만점 점수표
- v9: B2B/B2C 분리, 글로벌 일반 사용자 커뮤니티 추가
- v8: 페인 포인트 입력 슬롯 제거 (외부 데이터 100%)
- v7: 커뮤니티 시그널 농도 기반 등급 분류
- v6: 가산점에서 본인 페인 포인트 제거
- v5: 본인 속성 기반 우위 제거 (지리/도메인/기술 = 해자 아님)
- v4: 가산점 일반화, 시장조사 방법, 근거 출처 인용 강제
- v3: 제약 완화 (안티패턴 격하)
