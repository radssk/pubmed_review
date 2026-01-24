# PubMed Review Automation

PubMed 최신 논문을 자동으로 검색하고, AI로 평가한 뒤 Google Sheets에 정리하는 자동화 도구입니다.

**매일 실행** → 새 논문 발견 → AI 평가 → 스프레드시트에 자동 저장

## ✨ Features

- 🔍 **자동 검색**: PubMed에서 설정한 쿼리로 최신 논문 자동 수집
- 🤖 **AI 평가**: OpenAI GPT로 논문 참신성 평가 및 3줄 요약 생성
- 📊 **자동 저장**: Google Sheets에 결과 자동 저장 (컬럼 헤더 포함)
- 🎯 **스마트 필터링**: High IF 저널 또는 참신한 논문만 선별
- ♻️ **중복 방지**: 이미 처리한 논문은 자동으로 스킵
- 💾 **안전한 저장**: 10개씩 배치 저장으로 에러 시 데이터 손실 최소화
- 🔄 **자동 재시도**: 네트워크 오류 시 exponential backoff으로 최대 4회 재시도
- 💰 **비용 최적화**: High IF 논문은 novelty 체크 생략, 토큰 사용량 로깅

## 🚀 Quick Start

### 1. 환경 설정

```bash
# 저장소 클론
git clone https://github.com/your-username/pubmed_review.git
cd pubmed_review

# 의존성 설치
pip install -r requirements.txt

# 설정 파일 수정
cp config.yaml config.yaml  # 이미 있음
# config.yaml에서 email, search_query 수정
```

### 2. API 키 설정

**필수 환경 변수:**

```bash
export PUBMED_EMAIL="your_email@example.com"
export OPENAI_API_KEY="sk-..."
export GOOGLE_SERVICE_ACCOUNT_JSON='{"type": "service_account", ...}'
export SPREADSHEET_ID="1AbC...xYz"  # Google Sheets ID
```

<details>
<summary>📌 Google Service Account 생성 방법</summary>

1. [Google Cloud Console](https://console.cloud.google.com) 접속
2. 프로젝트 생성 → APIs & Services → Credentials
3. Create Credentials → Service Account 생성
4. Service Account에서 Keys → Add Key → JSON 다운로드
5. JSON 파일 내용을 `GOOGLE_SERVICE_ACCOUNT_JSON`에 복사
6. Google Sheets API 활성화: [여기서 활성화](https://console.developers.google.com/apis/api/sheets.googleapis.com/overview)

</details>

### 3. Google Sheets 준비

1. Google Sheets에서 새 스프레드시트 생성
2. URL에서 ID 복사: `https://docs.google.com/spreadsheets/d/`**`1AbC...xYz`**`/edit`
3. 서비스 계정 이메일(`xxx@xxx.iam.gserviceaccount.com`)을 **편집자**로 공유

### 4. 로컬 테스트

**Dry-run 모드 (API 호출 없이 설정 검증):**

```bash
DRY_RUN=true python -m pubmed_review.main
```

**실제 실행:**

```bash
python -m pubmed_review.main
```

성공하면 Google Sheets에 다음과 같이 저장됩니다:

| Date | PMID | Title | Journal | ... | Summary |
|------|------|-------|---------|-----|---------|
| 2026-01-24 | 38123456 | Novel deep learning... | Radiology | ... | This study presents... |

<details>
<summary>📸 예상 결과물 보기</summary>

Google Sheets에 다음 형식으로 저장됩니다:

```
| Date       | PMID     | Title                          | Journal   | Pub Date  | DOI              | Selection    | Novelty Reason        | Summary           | Strengths         |
|------------|----------|--------------------------------|-----------|-----------|------------------|--------------|-----------------------|-------------------|-------------------|
| 2026-01-24 | 38123456 | Deep learning for CT diagnosis | Radiology | 2026 Jan  | 10.1148/rad.123  | High IF      | Not evaluated (High IF)| This study uses...| Strong dataset... |
| 2026-01-24 | 38123457 | Novel AI approach for MRI      | Other     | 2026 Jan  | 10.1234/abc.456  | Novelty      | New architecture...   | Introduces a...   | Innovative method |
```

각 논문은 자동으로 평가되어 High IF 또는 Novelty 기준으로 필터링됩니다.

</details>

### 5. GitHub Actions 자동화

**Secrets 설정** (Settings → Secrets and variables → Actions):

- `OPENAI_API_KEY`
- `GOOGLE_SERVICE_ACCOUNT_JSON`
- `SPREADSHEET_ID`

`.github/workflows/pubmed_review.yml`이 이미 설정되어 있어서 **3일마다 자동 실행**됩니다.

수동 실행: Actions 탭 → PubMed Review Automation → Run workflow

---

## 💰 Cost Estimation

### OpenAI API Costs (gpt-4o-mini)

**Typical usage per run:**
- 50 papers found
- 30 papers filtered (20 High IF, 10 Novel)
- High IF papers: 1 API call each (summary only)
- Novel papers: 2 API calls each (novelty + summary)

**Token usage:**
- Novelty check: ~600 tokens per paper
- Summary: ~700 tokens per paper

**Estimated cost per run:**
```
High IF papers:  20 × 700 tokens  = 14,000 tokens
Novel papers:    10 × 1,300 tokens = 13,000 tokens
Total:                                27,000 tokens ≈ $0.01 USD
```

**Monthly cost (every 3 days):**
- ~10 runs/month × $0.01 = **$0.10 USD/month**

**Cost optimization tips:**
1. Add more journals to `high_if_journals` (skips novelty check)
2. Reduce `retmax` if you don't need 200 papers
3. Use narrower search queries

### Google Sheets API

**Free tier:**
- 60 requests/minute per user
- This tool uses <10 requests per run
- **Cost: $0**

### PubMed API

**Free** - No cost, no API key required (just email for contact)

---

## 📝 Configuration

<details>
<summary><b>config.yaml 상세 설정</b></summary>

### 단일 검색 설정

```yaml
pubmed:
  email: "your_email@example.com"
  search_query: '("Radiology") AND (("large language model") OR ("GPT"))'
  sheet_name: "Radiology NLP"  # 시트 탭 이름
  reldate: 3  # 최근 3일간 논문 (비우면 workflow.schedule_days 사용)
  retmax: 200  # 최대 논문 수

filters:
  high_if_journals: ["Nature", "Science", "Cell", "Nature Medicine", "The Lancet"]

llm:
  model: "gpt-4o-mini"
  temperature: 0.2

sheets:
  spreadsheet_id: "YOUR_SHEET_ID"

workflow:
  schedule_days: 3  # GitHub Actions 주기 (cron과 일치시켜야 함)
```

### 다중 검색 설정 (여러 주제를 각각 다른 시트에)

```yaml
pubmed:
  email: "your_email@example.com"
  searches:
    - query: '("Radiology") AND ("large language model")'
      sheet_name: "Radiology NLP"
    - query: '("CT") AND ("deep learning")'
      sheet_name: "CT Deep Learning"
    - query: '("MRI") AND ("artificial intelligence")'
      sheet_name: "MRI AI"
```

### LLM 프롬프트 커스터마이징

```yaml
llm:
  novelty_prompt: |
    다음 논문이 정말 새로운 방법론을 제시하는지 평가하세요.
    Title: {title}
    Journal: {journal}
    Abstract: {abstract}

  summary_prompt: |
    논문을 3줄 이내로 요약하고 강점을 설명하세요.
    Title: {title}
    Journal: {journal}
    Abstract: {abstract}
```

</details>

<details>
<summary><b>환경 변수 전체 목록</b></summary>

| 변수 | 필수 | 설명 | 기본값 |
|------|------|------|--------|
| `PUBMED_EMAIL` | ✅ | NCBI 연락용 이메일 | config.yaml |
| `OPENAI_API_KEY` | ✅ | OpenAI API 키 | - |
| `GOOGLE_SERVICE_ACCOUNT_JSON` | ✅ | Google 서비스 계정 JSON | - |
| `SPREADSHEET_ID` | ❌ | Google Sheets ID | config.yaml |
| `CONFIG_PATH` | ❌ | 설정 파일 경로 | `config.yaml` |
| `LOG_LEVEL` | ❌ | 로그 레벨 | `INFO` |
| `PUBMED_RELDATE` | ❌ | 검색 기간(일) | config.yaml |
| `PUBMED_RETMAX` | ❌ | 최대 논문 수 | `200` |

</details>

---

## 🔧 Troubleshooting

<details>
<summary><b>Google Sheets API is disabled</b></summary>

```
RuntimeError: Google Sheets API is disabled. Enable it at https://...
```

**해결**: 에러 메시지의 링크를 클릭해서 Google Sheets API 활성화

</details>

<details>
<summary><b>Permission denied</b></summary>

```
RuntimeError: Permission denied. Share the spreadsheet with: xxx@xxx.iam.gserviceaccount.com
```

**해결**: Google Sheets에서 서비스 계정 이메일을 편집자로 공유

</details>

<details>
<summary><b>Missing PUBMED_EMAIL</b></summary>

```
RuntimeError: Missing PUBMED_EMAIL or pubmed.email in config
```

**해결**: `config.yaml`에 `pubmed.email` 설정 또는 환경 변수 `PUBMED_EMAIL` 추가

</details>

<details>
<summary><b>중복 논문이 계속 저장됨</b></summary>

**원인**: 시트 이름이 잘못되었거나 PMID 컬럼(B열)이 없음

**해결**:
1. 시트 이름이 `config.yaml`의 `sheet_name`과 정확히 일치하는지 확인
2. 컬럼 헤더가 있는지 확인 (첫 실행 시 자동 생성됨)

</details>

<details>
<summary><b>LLM 비용이 너무 많이 나옴</b></summary>

**해결**:
1. `config.yaml`에서 `retmax` 줄이기 (예: 50)
2. `filters.high_if_journals`에 저널 추가 (novelty 체크 생략됨)
3. 로그에서 토큰 사용량 확인:
   ```
   LLM usage - prompt: 450, completion: 85, total: 535 tokens
   ```

</details>

---

## 📊 Output Format

Google Sheets에 다음과 같이 저장됩니다:

| 컬럼 | 설명 | 예시 |
|------|------|------|
| Date | 처리 날짜 | 2026-01-24 |
| PMID | PubMed ID | 38123456 |
| Title | 논문 제목 | Novel deep learning approach... |
| Journal | 저널명 | Radiology |
| Publication Date | 발행일 | 2026 Jan 15 |
| DOI | DOI | 10.1148/radiol.123456 |
| Selection Criteria | 선별 기준 | High IF, Novelty |
| Novelty Reason | 참신성 근거 | Introduces new architecture... |
| Summary | 3줄 요약 | This study presents a novel... |
| Strengths | 강점 | Strong validation on large dataset... |

---

## 📈 How It Works

```
PubMed 검색
    ↓
메타데이터 수집 (제목, 초록 등)
    ↓
중복 체크 (이미 처리한 PMID 스킵)
    ↓
각 논문마다:
  ├─ High IF 저널? → Yes → 요약 생성 → 저장
  └─ No → AI 참신성 평가 → 참신함? → Yes → 요약 생성 → 저장
                                    └─ No → 스킵
    ↓
10개씩 배치로 Google Sheets 저장
```

**비용 최적화**: High IF 논문은 novelty API 호출을 건너뛰어 비용 50% 절감

---

## 🛠️ Development

### Running Tests

```bash
# Install test dependencies
pip install -r requirements.txt

# Run all tests
pytest tests/ -v

# Run with coverage
pytest tests/ --cov=pubmed_review --cov-report=html

# Run specific test
pytest tests/test_main.py::TestClassName::test_method -v
```

### Local Development

```bash
# Dry-run mode (no API calls)
DRY_RUN=true python -m pubmed_review.main

# Debug mode
LOG_LEVEL=DEBUG python -m pubmed_review.main

# Custom config file
CONFIG_PATH=config.dev.yaml python -m pubmed_review.main
```

### Project Structure

```
pubmed_review/
├── pubmed_review/       # Main package
│   ├── __init__.py
│   └── main.py         # Core logic
├── tests/              # Test suite
│   ├── __init__.py
│   └── test_main.py
├── config.yaml         # Configuration
└── requirements.txt    # Dependencies
```

---

## 📄 License

MIT License - see [LICENSE](LICENSE) file for details

---

## 🤝 Contributing

We welcome contributions! 🙌

- 📖 Read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines
- 🐛 Report bugs via [Issues](https://github.com/radssk/pubmed_review/issues)
- 💡 Suggest features via [Discussions](https://github.com/radssk/pubmed_review/discussions)

**Quick Contribution Guide:**
1. Fork the repo
2. Create a feature branch
3. Write tests for your changes
4. Make your changes
5. Run tests: `pytest tests/ -v`
6. Submit a Pull Request
