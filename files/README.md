# 🍶 오프너(Opener) — 전통주 수출 AI SaaS

> 한국 전통주의 글로벌 진출을 돕는 **5단계 AI 수출 자동화 플랫폼**

[![Python](https://img.shields.io/badge/Python-3.11-blue)](https://python.org)
[![Streamlit](https://img.shields.io/badge/Streamlit-1.35-red)](https://streamlit.io)
[![Supabase](https://img.shields.io/badge/Supabase-PostgreSQL-green)](https://supabase.com)
[![Docker](https://img.shields.io/badge/Docker-Ready-blue)](https://docker.com)

---

## 🚀 5단계 워크플로우

```
1단계·2단계  💬 제품 인터뷰   →  AI 컨설턴트와 대화, 제품 정보 자동 수집
3단계        🎯 바이어 발굴   →  Tavily AI 실시간 검색 + 딥다이브 인텔리전스
4단계        🚀 제안서 생성   →  콜드메일 + 피치덱 + 레이더 차트 + PDF
5단계        🗄️ 데이터 저장  →  Supabase DB 저장 + 피드백 수집
```

---

## ✨ 주요 기능

| 단계 | 기능 | 핵심 파일 |
|------|------|-----------|
| 1·2 | AI 인터뷰 채팅 (제품명/맛/가격/브랜드스토리 수집) | `engine/interviewer.py` |
| 3 | Tavily 실시간 바이어 검색 + Pain Point 딥다이브 | `engine/market_analyzer.py` |
| 4 | 초개인화 콜드메일 생성 (영문 + 한국어) | `engine/doc_generator.py` |
| 4 | 맞춤형 피치덱 텍스트 5-slide 구조 | `engine/doc_generator.py` |
| 4 | 맛 비교 레이더 차트 (matplotlib) | `engine/doc_generator.py` |
| 4 | 통합 PDF 제안서 자동 생성 (ReportLab) | `engine/doc_generator.py` |
| 5 | Supabase DB 자동 저장 + 피드백 수집 | `database.py` |
| + | 라벨 번역 (5개 언어) | `engine/label_translator.py` |
| + | 시장 분석 + 규정 체크 | `engine/market_analyzer.py` |
| + | 수출 서류 초안 (CO, COA, PDS 등) | `engine/doc_generator.py` |

---

## 📁 프로젝트 구조

```
opener/
├── app.py                        # 메인 진입점 + 전역 DB 저장 훅 + 피드백 위젯
├── database.py                   # Supabase CRUD 레이어 (오프라인 graceful fallback)
├── Dockerfile                    # 2-stage 프로덕션 빌드
├── docker-compose.yml
├── requirements.txt
├── .env.example                  # 환경변수 체크리스트
│
├── engine/
│   ├── llm_client.py             # OpenAI / Anthropic 멀티 프로바이더
│   ├── interviewer.py            # 멀티턴 인터뷰 + JSON 추출
│   ├── market_analyzer.py        # Tavily 검색 + 바이어 딥다이브
│   ├── label_translator.py       # 라벨 분석·번역
│   └── doc_generator.py          # 콜드메일·피치덱·레이더·PDF
│
├── pages/
│   ├── home.py                   # 4단계 워크플로우 홈
│   ├── interview.py              # 채팅 인터뷰 UI
│   ├── buyer_hunter.py           # 바이어 발굴 + 딥다이브 카드
│   ├── proposal_generator.py     # 제안서 생성 + 미리보기
│   ├── label_translator.py
│   ├── market_analysis.py
│   ├── export_docs.py
│   └── settings.py               # AI/DB/통계/앱정보 탭
│
└── utils/
    ├── navigation.py             # 사이드바 + 워크플로우 배지
    ├── prompts.py                # LLM 프롬프트 9개
    ├── styles.py                 # 전통주 테마 CSS
    └── feedback_widget.py        # 👍/👎 전역 피드백 위젯
```

---

## ⚡ 빠른 시작

### 로컬 실행

```bash
# 1. 클론
git clone https://github.com/your-org/opener.git && cd opener

# 2. 가상환경
python -m venv venv && source venv/bin/activate

# 3. 의존성 설치
pip install -r requirements.txt

# 4. 환경변수 설정
cp .env.example .env
# .env 열어서 API 키 4개 입력 (아래 참고)

# 5. 실행
streamlit run app.py
# → http://localhost:8501
```

### Docker 실행

```bash
docker build -t opener .
docker run -p 8501:8501 --env-file .env opener
```

### docker-compose

```bash
docker-compose up -d
```

---

## 🔑 환경변수 체크리스트

| 변수 | 필수 | 발급처 | 설명 |
|------|------|--------|------|
| `OPENAI_API_KEY` | ✅ 필수* | platform.openai.com | GPT-4o |
| `ANTHROPIC_API_KEY` | ✅ 필수* | console.anthropic.com | Claude |
| `TAVILY_API_KEY` | 🟡 권장 | app.tavily.com | 바이어 검색 (월 1,000 무료) |
| `SUPABASE_URL` | 🟡 권장 | supabase.com | DB URL |
| `SUPABASE_ANON_KEY` | 🟡 권장 | supabase.com | DB 공개 키 |
| `DEFAULT_LLM_PROVIDER` | - | - | `openai` 또는 `anthropic` |
| `DEFAULT_MODEL` | - | - | `gpt-4o` 기본값 |

> *OpenAI 또는 Anthropic 중 하나만 있어도 실행 가능

---

## 🗄️ Supabase 테이블 설정

설정 → 데이터베이스 탭에서 SQL 자동 생성 버튼 제공.  
또는 아래 4개 테이블을 Supabase SQL Editor에서 직접 생성:

- `products` — 제품 인터뷰 데이터
- `buyer_analyses` — 바이어 딥다이브 결과
- `proposals` — 콜드메일 + 피치덱 텍스트
- `feedback` — 👍/👎 사용자 피드백

---

## 🌐 배포 가이드

| 플랫폼 | 명령 | 특징 |
|--------|------|------|
| **Railway** | `railway init && railway up` | Git 연동, 가장 간단 |
| **Fly.io** | `fly launch && fly deploy` | 글로벌 엣지, 무료 티어 |
| **Google Cloud Run** | `gcloud run deploy` | 서버리스, 트래픽 기반 과금 |
| **Render** | GitHub 연동 후 자동 배포 | 무료 티어 (슬립 있음) |

> 모든 플랫폼에서 환경변수를 대시보드 또는 CLI로 설정하세요.

---

## 🛣️ 로드맵

- [x] 1단계: 프로젝트 구조 설계
- [x] 2단계: AI 인터뷰 챗봇
- [x] 3단계: Tavily 바이어 발굴 + 딥다이브
- [x] 4단계: 제안서 PDF 자동 생성
- [x] 5단계: Supabase DB + 피드백 + Docker
- [ ] OCR 라벨 이미지 분석 (pytesseract)
- [ ] 다국어 UI (영어/일어 전환)
- [ ] 이메일 자동 발송 연동 (SendGrid)
- [ ] 바이어 CRM 대시보드
- [ ] Stripe 결제 연동 (SaaS 과금)

---

## 📄 라이선스

MIT License © 2025 Opener Team
