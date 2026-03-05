# auto-portfolio
**AI 기반 자동 포트폴리오 생성 서비스** | Project Overview v1.0

---

## 1. 프로젝트 개요

auto-portfolio는 개발자가 자신의 프로젝트 정보를 텍스트, 이미지, 문서 등 다양한 형태로 AI에게 제공하면, AI가 이를 분석·정리하여 완성도 높은 포트폴리오를 자동으로 생성해주는 웹 서비스입니다.

개발자가 포트폴리오를 직접 작성할 때 겪는 반복적인 정리 작업과 표현의 어려움을 AI가 대신 처리해주며, 일관되고 전문적인 포트폴리오를 빠르게 완성할 수 있도록 돕습니다.

| 가치 | 설명 |
|------|------|
| 시간 절약 | 프로젝트 정보를 던져주기만 하면 AI가 구조화·요약·미화까지 자동 처리 |
| 입력 유연성 | 텍스트, 이미지, PDF, GitHub URL, 음성 등 다양한 포맷 지원 |
| 품질 보장 | AI가 기술 스택·성과·역할을 일관된 포트폴리오 포맷으로 정제 |
| 즉시 공유 | 생성된 포트폴리오를 웹 링크 또는 PDF로 바로 배포 가능 |

---

## 2. 기술 스택

### Backend
| 항목 | 내용 |
|------|------|
| Language | Java 17+ |
| Framework | Spring Boot 3.x, Spring Security, Spring Data JPA |
| AI 연동 | Anthropic Claude API (claude-sonnet-4) / OpenAI API |
| File 처리 | Apache Tika (파일 파싱), AWS S3 (파일 저장) |
| Build | Gradle |

### Frontend
| 항목 | 내용 |
|------|------|
| Language | TypeScript |
| Framework | React 18+, React Router v6 |
| 상태 관리 | Zustand 또는 React Query (서버 상태) |
| UI | Tailwind CSS + shadcn/ui |
| File 업로드 | React Dropzone, Axios |
| Build | Vite |

### Database (MySQL) 주요 테이블
| 테이블 | 설명 |
|--------|------|
| users | 회원 정보 (id, email, name, created_at) |
| projects | 프로젝트 원본 데이터 (id, user_id, title, raw_inputs, created_at) |
| input_chunks | 업로드된 입력 조각 (id, project_id, type, content, s3_url) |
| portfolios | AI가 생성한 포트폴리오 (id, project_id, html_content, json_data, version) |
| generations | AI 생성 이력 (id, portfolio_id, prompt_used, model, tokens_used) |

---

## 3. 시스템 아키텍처

서비스는 크게 세 레이어로 구성됩니다: **사용자 입력 수집 레이어 → AI 처리 레이어 → 포트폴리오 렌더링 레이어**

| 단계 | 설명 |
|------|------|
| ① 입력 수집 | 사용자가 React 프론트엔드를 통해 텍스트, 파일, URL 등 다양한 형태의 프로젝트 정보를 업로드 |
| ② 전처리 | Spring Backend의 InputProcessingService가 파일 타입별로 파싱(PDF→텍스트, 이미지→OCR/설명, URL→크롤링)하여 통합 텍스트 청크로 변환 |
| ③ 프롬프트 구성 | PromptBuilderService가 구조화된 시스템 프롬프트와 사용자 입력 청크를 조합하여 Claude API 요청 구성 |
| ④ AI 생성 | Claude API가 포트폴리오 섹션별 JSON 구조(개요, 기술스택, 주요기능, 성과, 역할)를 반환 |
| ⑤ 렌더링 & 저장 | 생성된 JSON을 MySQL에 저장하고, 프론트엔드에서 포트폴리오 템플릿에 렌더링하여 표시 |
| ⑥ 배포 | 완성된 포트폴리오를 퍼블릭 URL 또는 PDF 파일로 내보내기 |

---

## 4. AI에게 프로젝트 정보를 전달하는 최적 전략

AI가 포트폴리오를 잘 작성하려면 단순히 "정보를 많이 주는 것"이 아니라, **AI가 이해하기 좋은 구조와 맥락으로 정보를 구성하는 것**이 핵심입니다.

### 4.1 지원 입력 유형

- **자유 텍스트** (가장 간단) — 노션, 메모장 등에 쓴 프로젝트 설명을 그대로 붙여넣기
- **구조화 양식** (가장 정확) — 서비스 제공 폼에 항목별로 입력 (프로젝트명, 기간, 역할, 기술스택, 주요 기능, 성과 수치)
- **파일 업로드** — README.md, PDF 제안서, PPT 발표자료 → 자동 텍스트 추출 후 AI 전달
- **GitHub URL** — 레포지토리 URL 입력 → README + 주요 파일 구조 크롤링
- **이미지/스크린샷** — UI 스크린샷 → Claude Vision으로 화면 기능 자동 설명 추출
- **음성 입력** (확장 기능) — Whisper API로 STT 변환 → 텍스트로 처리

### 4.2 AI 프롬프트 설계 전략

| 구성 요소 | 내용 |
|-----------|------|
| ① 역할 부여 | "당신은 10년 경력의 시니어 개발자이자 기술 포트폴리오 전문 작성자입니다. 채용 담당자와 동료 개발자가 인상받을 수 있는 포트폴리오를 작성하세요." |
| ② 출력 포맷 고정 | JSON 스키마 명시: `title`, `summary`, `techStack[]`, `keyFeatures[]`, `role`, `achievements[]`, `period`, `links{github, demo}` |
| ③ 톤 & 수준 지정 | "기술적 용어는 정확히 사용하되, 성과는 수치화하세요. (예: 응답속도 40% 개선, DAU 1,200명)" |
| ④ 누락 정보 처리 | "정보가 부족한 필드는 임의로 채우지 말고, `null`을 반환하고 `missing_info[]` 배열에 항목명을 추가하세요." |
| ⑤ 다회차 개선 | 초안 생성 후 사용자 피드백을 대화 히스토리에 포함시켜 재생성 요청. 버전별 generations 테이블에 이력 저장. |

### 4.3 입력 품질에 따른 AI 출력 차이

| 입력 수준 | 입력 예시 | AI 출력 품질 |
|-----------|-----------|--------------|
| ❌ 최하 | "채팅앱 만들었어요" | 제목·기술스택 추정 불가. 누락 필드 대부분. 재입력 요청 메시지 생성. |
| △ 보통 | "React, Node.js로 채팅앱 만들었습니다. 실시간 기능 있고 로그인도 됩니다." | 기본 기술스택 파악 가능. 성과 수치·역할 누락. 보통 수준 포트폴리오 생성. |
| ✅ 좋음 | 기술스택 + 본인 역할 + 주요 기능 3가지 + 개발 기간 포함한 텍스트 | 완성도 높은 포트폴리오. 수치 성과 일부 누락 가능. |
| ⭐ 최상 | 위 항목 + 성과 수치("API 응답속도 40% 개선") + README.md 첨부 + 스크린샷 | 채용 수준 포트폴리오 자동 완성. 거의 수정 불필요. |

---

## 5. 주요 기능 목록

### MVP (1차 출시)
- 회원가입 / 로그인 (Spring Security + JWT)
- 프로젝트 정보 입력: 자유 텍스트 + 구조화 폼
- 파일 업로드: PDF, Markdown, 이미지
- AI 포트폴리오 자동 생성 (Claude API)
- 포트폴리오 미리보기 및 섹션 수동 편집
- 퍼블릭 링크 생성 (예: `autoportfolio.io/u/username/project-name`)

### 확장 기능 (2차)
- GitHub URL 연동 → README 자동 파싱
- AI와 대화형 수정 ("성과 부분을 더 임팩트 있게 써줘")
- 포트폴리오 PDF 내보내기
- 다중 프로젝트 → 통합 포트폴리오 자동 구성
- 포트폴리오 템플릿 선택 (개발자용, 디자이너용, 기획자용)

---

## 6. 핵심 REST API 설계

| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | `/api/auth/register` | 회원가입 |
| POST | `/api/auth/login` | 로그인 (JWT 반환) |
| POST | `/api/projects` | 프로젝트 생성 (텍스트/폼 입력) |
| POST | `/api/projects/{id}/inputs` | 파일/이미지/URL 추가 업로드 |
| POST | `/api/projects/{id}/generate` | AI 포트폴리오 생성 요청 |
| GET | `/api/portfolios/{id}` | 생성된 포트폴리오 조회 |
| PATCH | `/api/portfolios/{id}` | 포트폴리오 수동 수정 |
| POST | `/api/portfolios/{id}/publish` | 퍼블릭 링크 발행 |
| GET | `/p/{username}/{slug}` | 퍼블릭 포트폴리오 페이지 (인증 불필요) |

---

## 7. 개발 로드맵

| 단계 | 기간 | 주요 작업 |
|------|------|-----------|
| Phase 1 | 1~2주차 | Spring Boot 셋업, DB 설계, JWT 인증, React 프로젝트 셋업, 기본 라우팅 |
| Phase 2 | 3~4주차 | 프로젝트 입력 폼 UI, 파일 업로드 (S3), 텍스트 파싱 서비스 (Tika) |
| Phase 3 | 5~6주차 | Claude API 연동, PromptBuilder 구현, AI 생성 파이프라인, 포트폴리오 JSON → UI 렌더링 |
| Phase 4 | 7~8주차 | 포트폴리오 수동 편집, 퍼블릭 링크 발행, PDF 내보내기 |
| Phase 5 | 9~10주차 | GitHub URL 파싱, 대화형 수정 기능, 다중 프로젝트 통합 |

---

## 8. 기술적 고려사항 및 리스크

- **AI API 비용 관리** — 입력 토큰 최적화: 파일에서 핵심 텍스트만 추출. 동일 입력 재생성 방지를 위한 Redis 캐싱 활용.
- **입력 전처리 품질** — 이미지 텍스트 추출 품질에 따라 AI 결과가 달라짐 → Claude Vision 활용. 한/영 혼용 다국어 처리 필요.
- **포트폴리오 품질 보장** — AI가 없는 정보를 생성하지 않도록 Hallucination 방지 프롬프트 필수. `missing_info` 필드로 사용자에게 추가 입력 유도.
- **보안** — 업로드된 파일의 악성코드 검사. 퍼블릭 포트폴리오의 개인정보 노출 여부 사용자 확인 단계 추가.

---

> *auto-portfolio — AI가 당신의 프로젝트를 빛나게 합니다*