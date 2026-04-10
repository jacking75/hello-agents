# 14장: 자동화된 심층 연구 Agent

13장의 여행 도우미 프로젝트에서 우리는 HelloAgents를 멀티 에이전트 제품에 적용하는 방법을 경험했습니다. 이번 장에서는 계속 나아가, **지식 집약적 애플리케이션**에 초점을 맞춥니다: **심층 연구 작업을 자동으로 실행할 수 있는 에이전트 도우미 구축**.

여행 계획과 비교했을 때, 심층 연구의 어려움은 정보의 지속적인 분기, 사실의 빠른 업데이트, 그리고 인용 출처에 대한 사용자의 높은 요구사항에 있습니다. 신뢰할 수 있는 연구 보고서를 제공하려면 Agent에 세 가지 핵심 기능을 갖춰야 합니다:

**(1) 문제 분석**: 사용자의 개방형 주제를 검색 가능한 질의문으로 분해합니다.

**(2) 다중 라운드 정보 수집**: 서로 다른 검색 API를 결합하여 지속적으로 자료를 발굴하고, 중복을 제거하여 통합합니다.

**(3) 반성과 요약**: 단계적 결과를 바탕으로 지식 공백을 식별하고, 검색을 계속할지 여부를 결정하며, 구조화된 요약을 생성합니다.

## 14.1 프로젝트 개요 및 아키텍처 설계

### 14.1.1 왜 심층 연구 도우미가 필요한가

정보 폭발의 시대에, 우리는 매일 새로운 기술, 개념 또는 사건을 신속하게 파악해야 합니다. 전통적인 연구 방법에는 몇 가지 문제점이 있습니다. 첫째는 **정보 과부하**입니다. 검색 엔진은 수천 개의 결과를 반환하며, 유용한 정보를 찾으려면 링크를 하나씩 클릭하고 많은 내용을 읽어야 합니다. 둘째는 **구조의 부재**입니다. 관련 정보를 찾더라도, 이 정보는 흔히 단편적이고 체계적인 구성이 부족합니다. 마지막으로 **반복적인 노동**입니다. 새로운 주제를 연구할 때마다 "검색 → 읽기 → 요약 → 정리"의 과정을 반복해야 합니다.

이것이 바로 심층 연구 도우미가 해결해야 하는 문제입니다. 단순한 검색 도구가 아니라, 자율적으로 계획하고, 실행하며, 요약할 수 있는 연구 도우미입니다.

**심층 연구 도우미의 핵심 가치:**

1. **시간 절약**: 1~2시간의 연구 작업을 5~10분으로 압축
2. **품질 향상**: 체계적인 연구 과정으로 중요한 정보 누락 방지
3. **추적 가능**: 모든 검색 결과와 출처를 기록하여 검증 및 인용이 용이
4. **확장 가능**: 새로운 검색 엔진, 데이터 소스, 분석 도구를 쉽게 추가

### 14.1.2 기술 아키텍처 개요

이 시스템은 여전히 고전적인 **프론트엔드-백엔드 분리 아키텍처**를 채택하며, 그림 14.1과 같습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-1.png" alt="" width="85%"/>
  <p>그림 14.1 심층 연구 도우미 기술 아키텍처</p>
</div>

시스템은 4계층 아키텍처로 설계되었습니다:

**프론트엔드 계층 (Vue3+TypeScript)**: 전체 화면 모달 대화 UI, Markdown 결과 시각화

**백엔드 계층 (FastAPI)**: API 라우팅 (`/research/stream`)

**Agent 계층 (HelloAgents)**: 세 개의 전문화된 Agent (TODO Planner, Task Summarizer, Report Writer) + 두 개의 핵심 도구 (SearchTool, NoteTool)

**외부 서비스 계층**: 검색 엔진 + LLM 제공자

완전한 연구 요청이 시스템을 통해 어떻게 흐르는지 살펴보겠습니다. 그림 14.2와 같습니다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-2.png" alt="" width="85%"/>
  <p>그림 14.2 심층 연구 도우미 데이터 흐름 프로세스</p>
</div>

1. **사용자 입력**: 사용자가 프론트엔드에서 연구 주제를 입력
2. **프론트엔드 전송**: 프론트엔드가 SSE를 통해 `/research/stream`에 연결
3. **백엔드 수신**: FastAPI가 요청을 수신하고 연구 상태를 생성
4. **계획 단계**: 연구 계획 Agent를 호출하여 3개의 하위 작업으로 분해
5. **실행 단계**: 각 하위 작업을 하나씩 실행
   - SearchTool을 사용하여 검색
   - 작업 요약 Agent를 호출하여 요약
   - NoteTool을 사용하여 결과 기록
6. **보고 단계**: 보고서 생성 Agent를 호출하여 모든 요약을 통합
7. **스트림 반환**: SSE를 통해 진행 상황과 결과를 프론트엔드에 푸시
8. **프론트엔드 표시**: 프론트엔드가 작업 상태, 진행 표시줄, 로그, 보고서를 실시간으로 업데이트

프로젝트 디렉터리 구조는 다음과 같습니다:

```
helloagents-deepresearch/
├── backend/                    # 백엔드 코드
│   ├── src/
│   │   ├── agent.py           # 핵심 조정자
│   │   ├── main.py            # FastAPI 진입점
│   │   ├── models.py          # 데이터 모델
│   │   ├── prompts.py         # 프롬프트 템플릿
│   │   ├── config.py          # 구성 관리
│   │   └── services/          # 서비스 계층
│   │       ├── planner.py     # 계획 서비스
│   │       ├── summarizer.py  # 요약 서비스
│   │       ├── reporter.py    # 보고서 서비스
│   │       └── search.py      # 검색 서비스
│   ├── .env                   # 환경 변수
│   ├── pyproject.toml         # 의존성 관리
│   └── workspace/             # 연구 노트
│
└── frontend/                   # 프론트엔드 코드
    ├── src/
    │   ├── App.vue            # 메인 컴포넌트
    │   ├── components/        # UI 컴포넌트
    │   │   └── ResearchModal.vue
    │   └── composables/       # 컴포저블 함수
    │       └── useResearch.ts
    ├── package.json           # npm 의존성
    └── vite.config.ts         # 빌드 구성
```

### 14.1.3 빠른 체험: 5분 안에 프로젝트 실행하기

구현 세부사항을 살펴보기 전에, 먼저 프로젝트를 실행하여 최종 결과를 확인해 봅시다. 이렇게 하면 전체 시스템에 대한 직관적인 이해를 얻을 수 있습니다.

다음 명령어로 버전을 확인할 수 있습니다:

```bash
python --version  # Python 3.10.x 이상이어야 함
node --version    # v16.x.x 이상이어야 함
npm --version     # 8.x.x 이상이어야 함
```

**(1) 백엔드 시작**

```bash
# 1. 백엔드 디렉터리로 이동
cd helloagents-deepresearch/backend

# 2. 의존성 설치
# 방법 1: uv 사용 (권장, 더 빠른 Python 패키지 관리자)
uv sync

# 방법 2: pip 사용
pip install -e .

# 3. 환경 변수 구성
cp .env.example .env

# 4. .env 파일 편집, API 키 입력
# 선호하는 편집기로 .env 파일 열기
# 최소한 다음을 구성해야 함:
# - LLM_PROVIDER (예: openai, deepseek, qwen)
# - LLM_API_KEY (LLM API 키)
# - SEARCH_API (예: duckduckgo, tavily)

# 5. 백엔드 시작
python src/main.py
```

모든 것이 정상이면 다음과 유사한 출력을 볼 수 있습니다:

```
INFO:     Started server process [12345]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:8000 (Press CTRL+C to quit)
```

**(2) 프론트엔드 시작**

새 터미널 창을 열어:

```bash
# 1. 프론트엔드 디렉터리로 이동
cd helloagents-deepresearch/frontend

# 2. 의존성 설치
npm install

# 3. 프론트엔드 시작
npm run dev
```

모든 것이 정상이면 다음과 유사한 출력을 볼 수 있습니다:

```
  VITE v5.0.0  ready in 500 ms

  ➜  Local:   http://localhost:5174/
  ➜  Network: use --host to expose
  ➜  press h + enter to show help
```

**(3) 연구 시작**

브라우저를 열고 `http://localhost:5174`를 방문합니다. 그림 14.3과 같이 중앙에 입력 카드가 표시됩니다. 연구 주제를 입력하세요. 예를 들어 `Datawhale은 어떤 조직인가요?`를 입력하고, 검색 엔진을 선택한 후(여러 개가 구성된 경우), "연구 시작" 버튼을 클릭합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-3.png" alt="" width="85%"/>
  <p>그림 14.3 심층 연구 도우미 검색 페이지</p>
</div>

그림 14.4와 같이, 시스템은 자동으로 전체 화면으로 확장되고, 왼쪽에는 연구 정보가 표시되며, 오른쪽에는 연구 진행 상황과 결과가 실시간으로 표시됩니다. 전체 연구 과정은 주제의 복잡성과 검색 엔진의 응답 속도에 따라 약 1~3분이 소요됩니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-4.png" alt="" width="85%"/>
  <p>그림 14.4 심층 연구 도우미 연구 확장 화면</p>
</div>

연구가 완료되면 다음을 볼 수 있습니다:

- **작업 목록**: 모든 하위 작업과 그 상태를 표시
- **진행 로그**: 연구 과정 중 모든 작업을 표시
- **최종 보고서**: 모든 하위 작업의 요약과 출처 인용을 포함하는 구조화된 Markdown 보고서

이제 심층 연구 도우미를 성공적으로 실행하고 시스템에 대한 직관적인 이해를 얻었습니다.

## 14.2 TODO 기반 연구 패러다임

### 14.2.1 TODO 기반 연구란 무엇인가

전통적인 검색 엔진은 단일 질문에만 답할 수 있지만, 심층 연구는 일련의 관련 질문에 답해야 합니다. TODO 기반 연구 패러다임은 복잡한 연구 주제를 여러 하위 작업(TODO)으로 분해하고, 하나씩 실행한 후 결과를 통합합니다.

이 패러다임의 핵심 아이디어는: **"연구"라는 복잡한 작업을 "계획 → 실행 → 통합" 과정으로 변환**하는 것입니다.

예시를 통해 이 변환을 이해해 보겠습니다. "Datawhale은 어떤 조직인가요?"를 연구하고 싶다고 가정합시다. 전통적인 검색 방법은:

```
사용자 입력: Datawhale은 어떤 조직인가요?
검색 엔진: 10~20개의 링크를 반환
사용자: 링크를 하나씩 클릭하고, 내용을 읽고, 메모를 작성
결과: 단편적인 정보, 체계성 부족
```

이 접근법의 문제점은 각 링크가 주제의 한 측면만 다루고, 체계적인 구조가 부족하며, 수동으로 정리하고 요약해야 한다는 것입니다.

**TODO 기반 접근법: 체계적인 연구**

```
사용자 입력: Datawhale은 어떤 조직인가요?

시스템 계획:
  ├─ TODO 1: Datawhale의 기본 정보 (조직 포지셔닝)
  ├─ TODO 2: Datawhale의 주요 프로젝트 (핵심 내용)
  ├─ TODO 3: Datawhale의 커뮤니티 문화 (가치관)
  └─ TODO 4: Datawhale의 영향력 (사회적 기여)

시스템 실행:
  각 TODO에 대해:
    1. 관련 자료 검색
    2. 핵심 정보 요약
    3. 출처 인용 기록

시스템 통합:
  구조화된 보고서 생성:
    ├─ 1부: 조직 포지셔닝 (TODO 1에서)
    ├─ 2부: 핵심 내용 (TODO 2에서)
    ├─ 3부: 가치관 (TODO 3에서)
    ├─ 4부: 사회적 기여 (TODO 4에서)
    └─ 참고문헌: 모든 출처 인용
```

이 접근법의 장점은 복잡한 주제를 명확한 하위 질문으로 분해하고, 각 하위 작업의 검색 결과와 요약을 기록하여 추적이 용이하며, 체계적인 연구 과정으로 중요한 정보 누락을 방지한다는 것입니다. 또한 새로운 하위 작업을 추가하거나 실행 순서를 조정하기도 쉽습니다.

완전한 TODO 기반 연구 시스템에는 세 가지 핵심 요소가 있습니다:

**(1) 지능형 계획자 (TODO Planner)**: 연구 주제를 하위 작업으로 분해하는 역할을 합니다. 좋은 계획자는 주제의 핵심 측면과 연구 목표를 이해하고, 주제를 3~5개의 하위 작업으로 분해하며(너무 적으면 빠뜨리고, 너무 많으면 중복됩니다), 각 하위 작업에 적합한 검색 쿼리를 설계해야 합니다.

**(2) 작업 실행자**: 각 하위 작업을 실행하는 역할을 합니다. 실행자는 검색 엔진을 사용하여 관련 자료를 얻고, 핵심 정보를 추출하여 중복 내용을 제거하며, 쉬운 검증을 위해 모든 출처 인용을 저장해야 합니다.

**(3) 보고서 작성자**: 모든 하위 작업의 결과를 통합하는 역할을 합니다. 생성자는 논리적 순서로 내용을 정리하고, 중복 정보를 병합하며, 각 관점에 출처 인용을 추가해야 합니다.

우리의 경우, TODO 기반 연구 과정은 그림 14.5와 같습니다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-5.png" alt="" width="85%"/>
  <p>그림 14.5 TODO 기반 연구 과정</p>
</div>

전체 과정은 선형이지만, 각 단계에는 명확한 입력과 출력이 있습니다. 이 설계는 시스템을 이해하고 디버깅하기 쉽게 만듭니다.

### 14.2.2 3단계 연구 과정

TODO 기반 연구 과정은 계획, 실행, 보고의 세 단계로 나뉩니다. 각 단계에는 전담 Agent가 있습니다.

**(1) 1단계: 계획**

계획 단계의 목표는 연구 주제를 3~5개의 하위 작업으로 분해하는 것입니다. 시스템은 연구 주제와 현재 날짜를 입력으로 받고, JSON 형식의 하위 작업 목록을 출력합니다. 각 하위 작업에는 세 개의 필드가 포함됩니다: title (작업 제목), intent (연구 의도), query (검색 쿼리).

연구 계획 Agent는 주제 특성에 따라 다른 분해 전략을 채택하며, 보통 기본 개념에서 시작하여 기술 현황, 실용적 응용, 발전 추세를 이해하고, 필요한 경우 비교 분석을 수행합니다. 예를 들어, "Datawhale은 어떤 조직인가요?"의 경우, 계획 Agent는 다음과 같은 하위 작업을 생성할 수 있습니다:

```json
[
  {
    "title": "Datawhale의 기본 정보",
    "intent": "Datawhale의 조직 포지셔닝, 설립 시간, 발전 역사 이해",
    "query": "Datawhale organization introduction history 2024"
  },
  {
    "title": "Datawhale의 주요 프로젝트",
    "intent": "Datawhale의 핵심 오픈소스 프로젝트와 튜토리얼 이해",
    "query": "Datawhale projects tutorials open source 2024"
  },
  ...
]
```

좋은 계획은 포괄적이고, 논리적으로 명확하며, 정확한 쿼리와 적절한 수의 항목을 가져야 합니다.

**(2) 2단계: 실행**

실행 단계는 각 하위 작업을 하나씩 실행하여 관련 자료를 검색하고 요약합니다. 시스템은 하위 작업 목록과 검색 엔진 구성을 입력으로 받고, 각 하위 작업에 대한 요약(Markdown 형식)과 출처 인용 목록을 출력합니다. 실행 과정은 다음과 같습니다:

각 하위 작업에 대해 실행자는:

1. **자료 검색**: 구성된 검색 엔진을 사용하여 검색 실행

   ```python
   search_results = search_tool.run({
       "input": task.query,
       "backend": "tavily",
       "mode": "structured",
       "max_results": 5
   })
   ```

2. **검색 결과 획득**: 제목, URL, 스니펫 추출

   ```json
   {
     "results": [
       {
         "title": "멀티모달 모델이란 무엇인가?",
         "url": "https://example.com/multimodal-model",
         "snippet": "멀티모달 모델은 여러 유형의 데이터를 처리할 수 있는 AI 모델입니다..."
       },
       ...
     ]
   }
   ```

3. **요약 Agent 호출**: 검색 결과 요약

   ```python
   summary = summarizer_agent.run(
       task=task,
       search_results=search_results
   )
   ```

4. **요약 및 출처 기록**: NoteTool에 저장

   ```python
   note_tool.run({
       "action": "create",
       "title": task.title,
       "content": f"## {task.title}\n\n{summary}\n\n## Sources\n{sources}",
       "tags": ["research", "summary"]
   })
   ```

작업 요약 Agent는 각 검색 결과에서 핵심 관점을 추출하고, 유사한 정보를 병합하며, 중요한 숫자, 날짜, 이름 등 핵심 데이터를 유지하고, 각 관점에 출처 인용을 추가합니다. 예를 들어, "Datawhale 기본 정보" 검색 결과에 대해 요약 Agent는 다음을 생성할 수 있습니다:

```markdown
## Datawhale 기본 정보

Datawhale은 데이터 과학과 AI에 초점을 맞춘 오픈소스 조직으로, 2018년에 설립되었습니다[1]. 조직의 핵심 사명은 "학습자를 위해, 학습자와 함께 성장"으로, 순수한 학습 커뮤니티 구축에 헌신합니다[2].

**핵심 포지셔닝:**

1. **오픈소스 교육 플랫폼**: 고품질의 AI 및 데이터 과학 학습 자료 제공[1]
2. **학습자 커뮤니티**: 수만 명의 AI 학습자와 실무자 모집[3]
3. **지식 공유**: 오픈소스 정신 옹호, 모든 내용 완전 무료 공개[2]

**발전 역사:**

- **2018년**: Datawhale 설립, 첫 번째 오픈소스 튜토리얼 출시[1]
- **2020년**: 중국의 선도적인 AI 학습 커뮤니티 중 하나가 됨[3]
- **2024년**: 50개 이상의 오픈소스 프로젝트 출시, 100,000명 이상의 학습자에게 영향[4]

## 출처

[1] https://github.com/datawhalechina
[2] https://datawhale.club/about
[3] https://www.zhihu.com/org/datawhale
[4] https://datawhale.cn
```

실행 중에 시스템은 진행 정보를 실시간으로 프론트엔드에 푸시합니다:

```json
{
  "type": "status",
  "message": "검색 중: Datawhale 기본 정보"
}
```

```json
{
  "type": "status",
  "message": "검색 결과 요약 중..."
}
```

```json
{
  "type": "task",
  "task": {
    "id": 1,
    "title": "Datawhale 기본 정보",
    "status": "completed"
  }
}
```

**(3) 3단계: 보고**

보고 단계의 목표는 모든 하위 작업의 요약을 통합하여 최종 보고서를 생성하는 것입니다. 시스템은 모든 하위 작업의 요약과 연구 주제를 입력으로 받고, Markdown 형식의 최종 보고서를 출력합니다. 보고서에는 제목, 개요, 각 하위 작업의 상세 분석, 결론, 참고문헌의 다섯 부분이 포함됩니다. 예를 들어, "Datawhale은 어떤 조직인가요?"의 경우, 최종 보고서는 다음과 같을 수 있습니다:

```markdown
# Datawhale은 어떤 조직인가?

## 개요

이 보고서는 Datawhale 오픈소스 조직을 체계적으로 연구했으며, 기본 정보, 주요 프로젝트, 커뮤니티 문화, 영향력의 네 가지 측면을 다룹니다.

## 1. Datawhale 기본 정보

Datawhale은 데이터 과학과 AI에 초점을 맞춘 오픈소스 조직으로, 2018년에 설립되었습니다...

(하위 작업 1의 요약 삽입)

## 2. Datawhale의 주요 프로젝트

Datawhale은 Hello-Agents, Joyful-Pandas 등 다수의 고품질 오픈소스 튜토리얼을 출시했습니다...

(하위 작업 2의 요약 삽입)
...
## 결론

이 연구를 통해 Datawhale의 조직 포지셔닝, 핵심 프로젝트, 커뮤니티 문화, 사회적 기여에 대해 알게 되었습니다. Datawhale은 AI 교육에 중요한 기여를 해온 순수한 학습 커뮤니티입니다.

## 참고문헌

[1] https://github.com/datawhalechina
[2] https://datawhale.club/about
...
```

보고서 생성 Agent는 하위 작업의 논리적 순서에 따라 내용을 정리하고, 시작 부분에 간략한 개요를 추가하며, 중복 정보를 병합하고, Markdown 형식을 통일하며, 모든 출처 인용을 참고문헌 섹션으로 정리합니다.

## 14.3 Agent 시스템 설계

### 14.3.1 Agent 책임 분담

심층 연구 도우미에서 우리는 세 개의 전문화된 Agent를 설계했으며, 각각 특정 작업을 담당합니다. 이렇게 하면 각 Agent가 단순해지고 이해하고 유지하기 쉬워집니다.

7장에서 우리는 `SimpleAgent`를 사용하여 에이전트를 구축하는 방법을 배웠습니다. `SimpleAgent`의 설계 철학은 단순하고 직접적입니다: `run()` 메서드가 호출될 때마다 Agent는 사용자의 질문을 분석하고, 도구를 호출할지 여부를 결정한 다음 결과를 반환합니다. 이 설계는 단순한 작업을 처리할 때 매우 효과적이지만, 심층 연구와 같은 복잡한 작업에 직면할 때는 멀티 에이전트 협업 접근법을 계속 사용해야 합니다.

표 14.1과 같이, 세 Agent는 각각 계획, 요약, 보고서 생성을 담당합니다.

<div align="center">
  <p>표 14.1 세 Agent의 책임 분담</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-table-1.png" alt="" width="85%"/>
</div>

각 Agent의 설계를 자세히 소개하겠습니다.

**Agent 1: 연구 계획 전문가 (TODO Planner)**

**책임**: 연구 주제를 3~5개의 하위 작업으로 분해

**설계 철학**: 연구 계획 전문가의 핵심 작업은 사용자의 연구 주제를 이해하고, 주제의 핵심 측면을 분석한 다음, 일련의 하위 작업을 생성하는 것입니다. 이 과정은 인간 연구자가 연구를 시작하기 전의 "브레인스토밍" 단계와 유사합니다.

**프롬프트 설계**:

```python
todo_planner_instructions = """
당신은 연구 계획 전문가입니다. 사용자의 연구 주제를 3~5개의 하위 작업으로 분해하는 것이 과제입니다.

현재 날짜: {current_date}

연구 주제: {research_topic}

이 연구 주제를 분석하고 3~5개의 하위 작업으로 분해하세요. 각 하위 작업은 다음을 충족해야 합니다:
1. 주제의 중요한 측면을 다룹니다
2. 명확한 연구 목표를 가집니다
3. 검색 엔진을 통해 관련 자료를 찾을 수 있습니다

JSON 형식으로 하위 작업 목록을 반환하세요. 각 하위 작업에는 다음이 포함됩니다:
- title: 작업 제목 (간결하고 명확하게)
- intent: 작업 의도 (왜 이것을 연구하는지)
- query: 검색 쿼리 (검색 엔진용 쿼리 문자열, 더 나은 검색 결과를 위해 영어 사용 가능)

출력 예시:
[
  {{
    "title": "멀티모달 모델이란 무엇인가",
    "intent": "멀티모달 모델의 기본 개념을 이해하여 이후 연구의 토대를 마련",
    "query": "multimodal model definition concept 2024"
  }},
  ...
]

다음을 확인하세요:
1. 하위 작업 수는 3~5개
2. 하위 작업 간에 논리적 관계 (예: 기초에서 응용으로, 현황에서 추세로)
3. 검색 쿼리가 관련 자료를 정확하게 찾을 수 있음
4. JSON만 반환하고 다른 텍스트는 포함하지 않음
"""
```

**핵심 설계 포인트**: 프롬프트에 최신 정보를 얻기 위한 현재 날짜를 포함하고, 쉬운 파싱을 위해 JSON 형식 출력을 명시적으로 요구하며, 예시를 통해 Agent가 예상 출력을 이해하도록 돕고, 하위 작업 수와 논리적 관계 등 제약 사항을 강조합니다.

**구현 코드**:

여기서 ToolAwareSimpleAgent는 SimpleAgent의 확장입니다. 14.3.2절에서 확인할 수 있으며, 여기서는 깊이 다루지 않아도 됩니다.

```python
class PlanningService:
    def __init__(self, llm: HelloAgentsLLM):
        self._agent = ToolAwareSimpleAgent(
            name="TODO Planner",
            system_prompt="You are a research planning expert",
            llm=llm,
            tool_call_listener=self._on_tool_call
        )

    def plan_todo_list(self, state: SummaryState) -> List[TodoItem]:
        prompt = todo_planner_instructions.format(
            current_date=get_current_date(),
            research_topic=state.research_topic,
        )

        response = self._agent.run(prompt)
        tasks_payload = self._extract_tasks(response)

        todo_items = []
        for idx, item in enumerate(tasks_payload, start=1):
            task = TodoItem(
                id=idx,
                title=item["title"],
                intent=item["intent"],
                query=item["query"],
            )
            todo_items.append(task)

        return todo_items

    def _extract_tasks(self, response: str) -> List[dict]:
        """Agent 응답에서 JSON 추출"""
        # 정규식으로 JSON 부분 추출
        json_match = re.search(r'\[.*\]', response, re.DOTALL)
        if json_match:
            json_str = json_match.group(0)
            return json.loads(json_str)
        else:
            raise ValueError("응답에서 JSON을 추출할 수 없습니다")
```

**Agent 2: 작업 요약 전문가 (Task Summarizer)**

**책임**: 검색 결과 요약, 핵심 정보 추출

**설계 철학**: 작업 요약 전문가의 핵심 작업은 검색 결과를 읽고, 핵심 정보를 추출하여 구조화된 방식으로 제시하는 것입니다. 이 과정은 인간 연구자가 문헌을 읽은 후 메모를 작성하는 것과 유사합니다.

**프롬프트 설계**:

```python
task_summarizer_instructions = """
당신은 작업 요약 전문가입니다. 검색 결과를 요약하고 핵심 정보를 추출하는 것이 과제입니다.

작업 제목: {task_title}
작업 의도: {task_intent}
검색 쿼리: {task_query}

검색 결과:
{search_results}

위의 검색 결과를 주의 깊게 읽고, 핵심 정보를 추출하여 Markdown 형식의 요약을 반환하세요.

요약에는 다음이 포함되어야 합니다:
1. **핵심 관점**: 검색 결과의 핵심 관점과 결론
2. **핵심 데이터**: 중요한 숫자, 날짜, 이름 등
3. **출처 인용**: 각 관점에 출처 인용 추가 ([1], [2] 등 사용)

다음을 확인하세요:
1. 요약이 간결하고 명확하며 중복을 피합니다
2. 중요한 세부 사항과 데이터를 유지합니다
3. 각 관점에 출처 인용을 추가합니다
4. Markdown 형식 사용 (제목, 목록, 굵게 등)

출력 예시:
## 핵심 관점

멀티모달 모델은 여러 유형의 데이터를 처리할 수 있는 AI 모델입니다[1]. 전통적인 단일 모달 모델과 달리, 멀티모달 모델은 텍스트, 이미지, 오디오 등을 동시에 이해할 수 있습니다[2].

**핵심 기능:**
- 교차 모달 이해[1]
- 통합 표현[3]
- 엔드투엔드 학습[2]

## 출처

[1] https://example.com/source1
[2] https://example.com/source2
[3] https://example.com/source3
"""
```

**핵심 설계 포인트**: 프롬프트에 작업 제목, 의도, 쿼리 등 맥락 정보를 포함하여 Agent가 작업을 이해하도록 돕고, 핵심 관점, 핵심 데이터, 출처 인용을 포함하도록 명시적으로 요구하며, 각 관점에 출처 인용 추가를 강조하고, 예시를 통해 Agent가 예상 출력 형식을 이해하도록 돕습니다.

**구현 코드**:

```python
class SummarizationService:
    def __init__(self, llm: HelloAgentsLLM):
        self._agent = ToolAwareSimpleAgent(
            name="Task Summarizer",
            system_prompt="You are a task summarization expert",
            llm=llm,
            tool_call_listener=self._on_tool_call
        )

    def summarize_task(
        self,
        task: TodoItem,
        search_results: List[dict]
    ) -> str:
        # 검색 결과 형식화
        formatted_sources = self._format_sources(search_results)

        prompt = task_summarizer_instructions.format(
            task_title=task.title,
            task_intent=task.intent,
            task_query=task.query,
            search_results=formatted_sources,
        )

        summary = self._agent.run(prompt)
        return summary

    def _format_sources(self, search_results: List[dict]) -> str:
        """검색 결과 형식화"""
        formatted = []
        for idx, result in enumerate(search_results, start=1):
            formatted.append(
                f"[{idx}] {result['title']}\n"
                f"URL: {result['url']}\n"
                f"Snippet: {result['snippet']}\n"
            )
        return "\n".join(formatted)
```

**Agent 3: 보고서 작성 전문가 (Report Writer)**

**책임**: 모든 하위 작업의 요약을 통합하여 최종 보고서 생성

**설계 철학**: 보고서 작성 전문가의 핵심 작업은 모든 하위 작업의 요약을 구조화된 보고서로 통합하는 것입니다. 이 과정은 인간 연구자가 모든 조사를 완료한 후 연구 보고서를 작성하는 것과 유사합니다.

**프롬프트 설계**:

```python
report_writer_instructions = """
당신은 보고서 작성 전문가입니다. 모든 하위 작업의 요약을 통합하여 구조화된 연구 보고서를 생성하는 것이 과제입니다.

연구 주제: {research_topic}

하위 작업 요약:
{task_summaries}

위의 모든 하위 작업 요약을 통합하여 구조화된 연구 보고서를 생성하세요.

보고서에는 다음이 포함되어야 합니다:
1. **제목**: 연구 주제
2. **개요**: 연구 주제와 보고서 구조를 간략히 소개 (2~3단락)
3. **각 하위 작업의 상세 분석**: 논리적 순서로 정리 (2수준 제목 사용)
4. **결론**: 연구의 주요 발견 요약 (1~2단락)
5. **참고문헌**: 모든 출처 인용 (하위 작업별로 그룹화)

다음을 확인하세요:
1. 보고서 구조가 명확하고 논리적으로 일관성이 있음
2. 중복 정보 제거
3. 모든 출처 인용 유지
4. Markdown 형식 사용

출력 예시:
# 멀티모달 대형 모델의 최신 동향

## 개요

이 보고서는 멀티모달 대형 모델의 최신 동향을 체계적으로 연구했습니다...

## 1. 멀티모달 모델이란 무엇인가

(하위 작업 1의 요약 삽입)

## 2. 최신 멀티모달 모델은 무엇인가

(하위 작업 2의 요약 삽입)

...

## 결론

이 연구를 통해 우리는...

## 참고문헌

### 작업 1: 멀티모달 모델이란 무엇인가
[1] https://example.com/source1
...
"""
```

**핵심 설계 포인트**: 프롬프트에 제목, 개요, 상세 분석, 결론, 참고문헌 등의 구조를 명시적으로 요구하고, 논리적 순서로 내용을 정리하도록 강조하며, 중복 정보를 병합하여 중복성을 제거하도록 요구하고, 모든 출처 인용을 유지합니다.

**구현 코드**:

```python
class ReportingService:
    def __init__(self, llm: HelloAgentsLLM):
        self._agent = ToolAwareSimpleAgent(
            name="Report Writer",
            system_prompt="You are a report writing expert",
            llm=llm,
            tool_call_listener=self._on_tool_call
        )

    def generate_report(
        self,
        research_topic: str,
        task_summaries: List[Tuple[TodoItem, str]]
    ) -> str:
        # 하위 작업 요약 형식화
        formatted_summaries = self._format_summaries(task_summaries)

        prompt = report_writer_instructions.format(
            research_topic=research_topic,
            task_summaries=formatted_summaries,
        )

        report = self._agent.run(prompt)
        return report

    def _format_summaries(
        self,
        task_summaries: List[Tuple[TodoItem, str]]
    ) -> str:
        """하위 작업 요약 형식화"""
        formatted = []
        for idx, (task, summary) in enumerate(task_summaries, start=1):
            formatted.append(
                f"## Task {idx}: {task.title}\n"
                f"Intent: {task.intent}\n\n"
                f"{summary}\n"
            )
        return "\n".join(formatted)
```

### 14.3.2 ToolAwareSimpleAgent 설계

7장에서 우리는 `SimpleAgent`를 구현했으며, 이것은 HelloAgents 프레임워크의 기본 Agent입니다. 하지만 심층 연구 도우미에서는 **도구 호출을 기록**할 수 있는 Agent가 필요합니다. 바로 이것이 `ToolAwareSimpleAgent`가 등장한 이유입니다.

심층 연구 도우미에서 우리는 각 Agent의 도구 호출 상태를 기록해야 합니다:

1. **디버깅**: Agent가 어떤 도구를 호출했고 어떤 매개변수가 전달되었는지 확인
2. **로깅**: 연구 과정 중 모든 작업 기록
3. **분석**: Agent의 행동 패턴 분석
4. **진행 표시**: Agent가 무엇을 하고 있는지 실시간으로 표시

`SimpleAgent` 자체는 도구 호출 리스닝을 지원하지 않으므로, 이를 확장해야 합니다.

`ToolAwareSimpleAgent`는 `SimpleAgent`에 `tool_call_listener` 매개변수를 추가합니다. 이것은 도구가 호출될 때마다 호출되는 콜백 함수입니다.

**사용 예시:**

```python
from hello_agents import ToolAwareSimpleAgent

def tool_listener(call_info):
    print(f"Agent: {call_info['agent_name']}")
    print(f"도구: {call_info['tool_name']}")
    print(f"매개변수: {call_info['parsed_parameters']}")
    print(f"결과: {call_info['result']}")

agent = ToolAwareSimpleAgent(
    name="Research Assistant",
    system_prompt="You are a research assistant",
    llm=llm,
    tool_call_listener=tool_listener
)
```

`ToolAwareSimpleAgent`는 `SimpleAgent`를 상속받고 `_execute_tool_call` 메서드를 재정의합니다:

```python
class ToolAwareSimpleAgent(SimpleAgent):
    def __init__(
        self,
        name: str,
        system_prompt: str,
        llm: HelloAgentsLLM,
        tool_registry: Optional[ToolRegistry] = None,
        tool_call_listener: Optional[Callable] = None,
    ):
        super().__init__(
            name=name,
            system_prompt=system_prompt,
            llm=llm,
            tool_registry=tool_registry,
        )
        self._tool_call_listener = tool_call_listener

    def _execute_tool_call(self, tool_name: str, parameters: str) -> str:
        """도구 호출을 실행하고 리스너에 알림"""
        # 매개변수 파싱
        parsed_parameters = self._parse_parameters(parameters)

        # 도구 호출
        result = super()._execute_tool_call(tool_name, parameters)

        # 리스너에 알림
        if self._tool_call_listener:
            self._tool_call_listener({
                "agent_name": self.name,
                "tool_name": tool_name,
                "parsed_parameters": parsed_parameters,
                "result": result,
            })

        return result
```

심층 연구 도우미에서 우리는 `ToolAwareSimpleAgent`를 사용하여 모든 Agent 도구 호출을 기록합니다:

```python
class DeepResearchAgent:
    def __init__(self, config: Configuration):
        self.config = config
        self.llm = HelloAgentsLLM(...)

        # 도구 호출 리스너 생성
        def tool_listener(call_info):
            self._emit_event({
                "type": "tool_call",
                "agent": call_info["agent_name"],
                "tool": call_info["tool_name"],
                "parameters": call_info["parsed_parameters"],
            })

        # 세 개의 Agent 생성, 모두 같은 리스너 사용
        self.planner = PlanningService(self.llm, tool_listener)
        self.summarizer = SummarizationService(self.llm, tool_listener)
        self.reporter = ReportingService(self.llm, tool_listener)
```

이렇게 하면 모든 Agent 도구 호출이 기록되고 SSE를 통해 프론트엔드에 푸시되어 사용자에게 실시간으로 표시됩니다.

### 14.3.3 Agent 협업 모드

세 Agent는 그림 14.6과 같이 **순차적 협업** 관계를 가집니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-6.png" alt="" width="85%"/>
  <p>그림 14.6 Agent 협업 과정</p>
</div>

순차적 협업 모드의 특징은:

1. **선형 과정**: Agent가 고정된 순서로 실행됨
2. **명확한 입출력**: 각 Agent의 입력은 이전 Agent의 출력에서 옴
3. **비동시성**: 한 번에 하나의 Agent만 작동

`DeepResearchAgent`는 전체 시스템의 핵심 조정자로, 세 Agent를 스케줄링하는 역할을 합니다:

```python
class DeepResearchAgent:
    def run(self, research_topic: str) -> str:
        # 1. 계획 단계
        self._emit_event({"type": "status", "message": "연구 작업 계획 중..."})
        todo_list = self.planner.plan_todo_list(research_topic)
        self._emit_event({"type": "tasks", "tasks": todo_list})

        # 2. 실행 단계
        task_summaries = []
        for task in todo_list:
            self._emit_event({
                "type": "status",
                "message": f"연구 중: {task.title}"
            })

            # 검색
            search_results = self.search_service.search(task.query)

            # 요약
            summary = self.summarizer.summarize_task(task, search_results)
            task_summaries.append((task, summary))

            self._emit_event({
                "type": "task_completed",
                "task_id": task.id
            })

        # 3. 보고 단계
        self._emit_event({"type": "status", "message": "보고서 생성 중..."})
        report = self.reporter.generate_report(research_topic, task_summaries)
        self._emit_event({"type": "report", "content": report})

        return report
```

## 14.4 도구 시스템 통합

### 14.4.1 SearchTool 확장

7장에서 우리는 `SearchTool`의 기본 버전을 구현했으며, Tavily와 SerpApi 검색 엔진을 통합하여 다중 소스 검색의 설계 아이디어를 보여주었습니다. 이번 장의 심층 연구 도우미에서는 `SearchTool`의 기능을 더욱 확장하여 DuckDuckGo, Perplexity, SearXNG 등의 검색 엔진을 추가하고, Advanced 모드(여러 검색 엔진 조합)를 구현했습니다. 검색은 심층 연구 도우미의 가장 핵심적인 기능이며, 이러한 확장을 통해 시스템이 다양한 사용 시나리오와 요구사항에 적응할 수 있습니다.

표 14.2와 같이, 이번에 추가된 검색 엔진들은 서로 다른 특성과 적용 시나리오를 가지고 있습니다.

<div align="center">
  <p>표 14.2 다중 검색 엔진 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-table-2.png" alt="" width="85%"/>
</div>

개별 확장 방법은 더 이상 논의하지 않겠습니다. 소스 코드와 7장의 확장 사례를 참조하여 구현할 수 있습니다. `SearchTool`은 통합된 검색 인터페이스를 제공합니다. 어떤 검색 엔진을 사용하든 호출 방식은 동일합니다.

심층 연구 도우미에서 우리는 구성 파일을 통해 검색 엔진을 선택합니다:

```python
# config.py
class SearchAPI(str, Enum):
    TAVILY = "tavily"
    DUCKDUCKGO = "duckduckgo"
    PERPLEXITY = "perplexity"
    SEARXNG = "searxng"
    ADVANCED = "advanced"

class Configuration(BaseModel):
    search_api: SearchAPI = SearchAPI.DUCKDUCKGO
    # ...
```

```python
# .env
SEARCH_API=tavily
```

이렇게 하면 사용자는 코드를 수정하지 않고 `.env` 파일을 수정하여 검색 엔진을 선택할 수 있습니다.

`SearchTool`이 반환하는 결과는 다음을 포함하는 딕셔너리입니다:

- `results`: 검색 결과 목록, 각 결과에는 제목, URL, 스니펫이 포함됨
- `backend`: 사용된 검색 엔진
- `answer`: AI가 생성한 답변 (Perplexity 전용)
- `notices`: 알림 정보 (API 한도, 오류 등)

다음은 몇 가지 특수 케이스 처리입니다.

검색 결과에 중복 URL이 포함될 수 있으므로 중복을 제거해야 합니다:

```python
def deduplicate_sources(sources: List[dict]) -> List[dict]:
    """중복 URL 제거"""
    seen_urls = set()
    unique_sources = []

    for source in sources:
        if source["url"] not in seen_urls:
            seen_urls.add(source["url"])
            unique_sources.append(source)

    return unique_sources
```

검색 결과에 많은 양의 텍스트가 포함될 수 있으므로 각 소스의 토큰 수를 제한해야 합니다:

```python
def limit_source_tokens(source: dict, max_tokens: int = 2000) -> dict:
    """소스의 토큰 수 제한"""
    snippet = source["snippet"]

    # 간단한 토큰 추정: 1토큰 ≈ 4자
    max_chars = max_tokens * 4

    if len(snippet) > max_chars:
        snippet = snippet[:max_chars] + "..."

    return {
        **source,
        "snippet": snippet
    }
```

### 14.4.2 NoteTool 사용

심층 연구 도우미에서 우리는 `NoteTool`을 사용하여 연구 진행 상황을 지속합니다. `NoteTool`은 9장에서 통합된 내장 도구로, 노트를 생성, 읽기, 업데이트, 삭제하는 데 사용됩니다.

연구 과정에서 우리는 각 하위 작업의 검색 결과, 요약, 최종 연구 보고서를 기록해야 합니다. 이 정보는 중단 시 마지막 진행 상황에서 연구를 계속할 수 있도록 디스크에 지속되어야 하며, 연구 과정 중 모든 작업을 확인하고 연구의 품질과 효율성을 분석하기도 편리합니다.

`NoteTool`은 지정된 작업 공간 디렉터리에 노트를 저장하며, 각 노트는 Markdown 파일입니다. 노트 파일명은 작업 ID이며, 내용에는 작업 제목, 작업 의도, 검색 쿼리, 검색 결과, 요약이 포함됩니다.

최종 생성된 파일 구조는 다음과 같습니다:

```
workspace/
├── notes/
│   ├── 1.md  # 작업 1의 노트
│   ├── 2.md  # 작업 2의 노트
│   ├── 3.md  # 작업 3의 노트
│   └── ...
└── reports/
    └── final_report.md  # 최종 보고서
```

심층 연구 도우미에서 우리는 `NoteTool`을 사용하여 각 하위 작업의 연구 진행 상황을 기록합니다:

```python
class NotesService:
    def __init__(self, workspace: str):
        self.note_tool = NoteTool(workspace=workspace)

    def save_task_summary(
        self,
        task: TodoItem,
        search_results: List[dict],
        summary: str
    ):
        """작업 요약 저장"""
        # 노트 내용 형식화
        content = self._format_note_content(
            task=task,
            search_results=search_results,
            summary=summary
        )

        # 노트 생성
        self.note_tool.run({
            "action": "create",
            "title": f"Task {task.id}: {task.title}",
            "content": content,
            "tags": ["research", "summary"]
        })

    def _format_note_content(
        self,
        task: TodoItem,
        search_results: List[dict],
        summary: str
    ) -> str:
        """노트 내용 형식화"""
        content = f"# Task {task.id}: {task.title}\n\n"
        content += f"## 작업 정보\n\n"
        content += f"- **의도**: {task.intent}\n"
        content += f"- **쿼리**: {task.query}\n\n"

        content += f"## 검색 결과\n\n"
        for idx, result in enumerate(search_results, start=1):
            content += f"[{idx}] {result['title']}\n"
            content += f"URL: {result['url']}\n"
            content += f"Snippet: {result['snippet']}\n\n"

        content += f"## 요약\n\n{summary}\n"

        return content
```

### 14.4.3 ToolRegistry 도구 관리

`ToolRegistry`는 HelloAgents 프레임워크의 도구 레지스트리로, 7장에서도 지원되며, 모든 도구의 등록과 호출을 관리하는 데 사용됩니다. 심층 연구 도우미에서 우리는 `ToolRegistry`를 사용하여 `SearchTool`과 `NoteTool`을 관리합니다.

Agent를 생성하기 전에 먼저 도구를 등록해야 합니다:

```python
from hello_agents import ToolAwareSimpleAgent
from hello_agents.tools import ToolRegistry
from hello_agents.tools import SearchTool
from hello_agents.tools import NoteTool

# 도구 생성
search_tool = SearchTool(backend="hybrid")
note_tool = NoteTool(workspace="./workspace/notes")

# 레지스트리 생성
registry = ToolRegistry()

# 도구 등록
registry.register_tool(search_tool)
registry.register_tool(note_tool)

# Agent 생성
agent = ToolAwareSimpleAgent(
    name="Research Assistant",
    system_prompt="You are a research assistant",
    llm=llm,
    tool_registry=registry
)
```

Agent가 도구를 호출해야 할 때, 그림 14.7과 같이 도구 호출 명령을 생성합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-7.png" alt="" width="85%"/>
  <p>그림 14.7 도구 호출 과정</p>
</div>

**도구 호출 과정**:

1. **Agent가 명령 생성**: Agent가 도구 호출 명령을 생성합니다. 예: `[TOOL_CALL:search_tool:{"input": "Datawhale organization", "backend": "tavily"}]`
2. **명령 파싱**: `ToolRegistry`가 명령을 파싱하여 도구 이름과 매개변수를 추출
3. **도구 찾기**: `ToolRegistry`가 도구 이름을 기반으로 해당 도구를 찾음
4. **도구 호출**: 도구의 `run` 메서드를 호출하고 매개변수 전달
5. **결과 반환**: 도구가 실행 결과를 반환
6. **결과 형식화**: 결과를 문자열로 형식화하여 Agent에 반환

## 14.5 서비스 계층 구현

이 섹션에서는 PlanningService, SummarizationService, ReportingService, SearchService를 포함한 핵심 서비스의 구현을 자세히 소개합니다. 이 서비스들은 Agent와 도구를 연결하는 다리로, 구체적인 비즈니스 로직을 담당합니다.

### 14.5.1 작업 계획 서비스

`PlanningService`는 연구 계획 Agent를 호출하여 연구 주제를 하위 작업으로 분해하는 역할을 합니다. 이것은 전체 연구 과정의 첫 번째이자 가장 중요한 단계입니다.

**(1) 구현 접근법**

핵심 책임:

1. **계획 Prompt 구성**: 연구 주제와 현재 날짜를 기반으로 Prompt 구성
2. **계획 Agent 호출**: TODO Planner Agent를 호출하여 하위 작업 목록 생성
3. **JSON 응답 파싱**: Agent 응답에서 JSON 형식의 하위 작업 목록 추출
4. **하위 작업 형식 검증**: 각 하위 작업에 필수 필드(title, intent, query)가 포함되어 있는지 확인

```python
import re
import json
from typing import List, Callable, Optional
from datetime import datetime

from hello_agents import HelloAgentsLLM
from hello_agents import ToolAwareSimpleAgent
from models import TodoItem, SummaryState
from prompts import todo_planner_instructions

class PlanningService:
    """작업 계획 서비스"""

    def __init__(
        self,
        llm: HelloAgentsLLM,
        tool_call_listener: Optional[Callable] = None
    ):
        self._llm = llm
        self._tool_call_listener = tool_call_listener

        # 계획 Agent 생성
        self._agent = ToolAwareSimpleAgent(
            name="TODO Planner",
            system_prompt="You are a research planning expert, skilled at decomposing complex research topics into clear subtasks.",
            llm=llm,
            tool_call_listener=tool_call_listener
        )

    def plan_todo_list(self, state: SummaryState) -> List[TodoItem]:
        """TODO 목록 계획

        Args:
            state: 연구 상태, 연구 주제 포함

        Returns:
            하위 작업 목록
        """
        # Prompt 구성
        prompt = todo_planner_instructions.format(
            current_date=self._get_current_date(),
            research_topic=state.research_topic,
        )

        # Agent 호출
        response = self._agent.run(prompt)

        # JSON 파싱
        tasks_payload = self._extract_tasks(response)

        # TodoItem 검증 및 생성
        todo_items = []
        for idx, item in enumerate(tasks_payload, start=1):
            # 필수 필드 검증
            if not all(key in item for key in ["title", "intent", "query"]):
                raise ValueError(f"작업 {idx}에 필수 필드가 누락되었습니다")

            task = TodoItem(
                id=idx,
                title=item["title"],
                intent=item["intent"],
                query=item["query"],
            )
            todo_items.append(task)

        return todo_items

    def _get_current_date(self) -> str:
        """현재 날짜 가져오기"""
        return datetime.now().strftime("%Y-%m-%d")

    def _extract_tasks(self, response: str) -> List[dict]:
        """Agent 응답에서 JSON 추출

        Agent의 응답에는 다음과 같이 추가 텍스트가 포함될 수 있습니다:
        "네, 다음 작업들을 계획해 드리겠습니다:\n[{...}, {...}]\n이 작업들은..."

        JSON 부분을 추출해야 합니다.
        """
        # 방법 1: 정규식으로 JSON 배열 추출
        json_match = re.search(r'\[.*\]', response, re.DOTALL)
        if json_match:
            json_str = json_match.group(0)
            try:
                return json.loads(json_str)
            except json.JSONDecodeError as e:
                raise ValueError(f"JSON 파싱 실패: {e}")

        # 방법 2: JSON 배열을 찾지 못한 경우, 전체 응답을 직접 파싱 시도
        try:
            return json.loads(response)
        except json.JSONDecodeError:
            raise ValueError("응답에서 JSON을 추출할 수 없습니다")
```

**(2) JSON 파싱과 검증**

Agent가 반환한 JSON에 추가 텍스트나 형식 오류가 포함될 수 있으므로, 강건한 파싱 로직이 필요합니다:

**일반적인 문제**:

1. **추가 텍스트 포함**: Agent가 JSON 앞뒤에 설명 텍스트를 추가할 수 있음
2. **형식 오류**: JSON에 따옴표, 쉼표 등이 누락될 수 있음
3. **필드 누락**: 일부 하위 작업에 필수 필드가 누락될 수 있음

**해결책**:

1. **정규식 사용**: JSON 부분 추출
2. **다중 파싱 전략**: 먼저 JSON 배열 추출을 시도하고, 그 다음 직접 파싱 시도
3. **필드 검증**: 각 하위 작업에 필수 필드가 포함되어 있는지 확인

**예시**:

```python
# Agent 응답 예시 1: 추가 텍스트 포함
response1 = """
네, 다음 작업들을 계획해 드리겠습니다:

[
  {
    "title": "멀티모달 모델이란 무엇인가",
    "intent": "기본 개념 이해",
    "query": "multimodal model definition"
  },
  {
    "title": "최신 멀티모달 모델",
    "intent": "기술 현황 이해",
    "query": "latest multimodal models 2024"
  }
]

이 작업들은 Datawhale 조직의 기본 정보와 핵심 프로젝트를 다룹니다.
"""

# JSON 추출
tasks1 = service._extract_tasks(response1)
# 결과: [{"title": "Datawhale 기본 정보", ...}, ...]

# Agent 응답 예시 2: 순수 JSON
response2 = """
[
  {"title": "Datawhale 기본 정보", "intent": "조직 포지셔닝 이해", "query": "Datawhale organization introduction"},
  {"title": "Datawhale 주요 프로젝트", "intent": "핵심 내용 이해", "query": "Datawhale projects tutorials 2024"}
]
"""

# JSON 추출
tasks2 = service._extract_tasks(response2)
# 결과: [{"title": "멀티모달 모델이란 무엇인가", ...}, ...]
```

**(3) 계획 품질 평가**

좋은 계획은 다음 기준을 충족해야 합니다:

1. **포괄적 커버리지**: 주제의 모든 중요한 측면을 커버
2. **명확한 논리**: 하위 작업 간의 논리적 관계가 명확
3. **정확한 쿼리**: 검색 쿼리가 관련 자료를 정확하게 찾을 수 있음
4. **적절한 수량**: 3~5개의 하위 작업

평가 방법을 추가할 수 있습니다:

```python
def evaluate_plan(self, todo_items: List[TodoItem]) -> dict:
    """계획 품질 평가

    Returns:
        평가 결과, 점수와 제안 포함
    """
    score = 100
    suggestions = []

    # 수량 확인
    if len(todo_items) < 3:
        score -= 20
        suggestions.append("하위 작업이 너무 적어 중요한 정보를 놓칠 수 있습니다")
    elif len(todo_items) > 5:
        score -= 10
        suggestions.append("하위 작업이 너무 많아 중복이 발생할 수 있습니다")

    # 쿼리 품질 확인
    for task in todo_items:
        if len(task.query.split()) < 2:
            score -= 10
            suggestions.append(f"'{task.title}' 작업의 쿼리가 너무 단순합니다")

    # 논리적 관계 확인
    # (여기에 더 복잡한 논리 확인을 추가할 수 있습니다)

    return {
        "score": score,
        "suggestions": suggestions
    }
```

### 14.5.2 요약 서비스

`SummarizationService`는 작업 요약 Agent를 호출하여 검색 결과를 요약하는 역할을 합니다. 이것은 연구 과정의 핵심 고리로, 연구의 품질을 결정합니다.

책임:

1. **검색 결과 형식화**: 검색 결과를 읽기 가능한 텍스트로 형식화
2. **요약 Prompt 구성**: 작업 정보와 검색 결과를 기반으로 Prompt 구성
3. **요약 Agent 호출**: Task Summarizer Agent를 호출하여 요약 생성
4. **출처 인용 추출**: 요약에서 출처 인용 추출

핵심 코드:

```python
from typing import List, Callable, Optional, Tuple

from hello_agents import HelloAgentsLLM
from hello_agents import ToolAwareSimpleAgent
from models import TodoItem
from prompts import task_summarizer_instructions

class SummarizationService:
    """요약 서비스"""

    def __init__(
        self,
        llm: HelloAgentsLLM,
        tool_call_listener: Optional[Callable] = None
    ):
        self._llm = llm
        self._tool_call_listener = tool_call_listener

        # 요약 Agent 생성
        self._agent = ToolAwareSimpleAgent(
            name="Task Summarizer",
            system_prompt="You are a task summarization expert, skilled at extracting key information from search results.",
            llm=llm,
            tool_call_listener=tool_call_listener
        )

    def summarize_task(
        self,
        task: TodoItem,
        search_results: List[dict]
    ) -> Tuple[str, List[str]]:
        """작업 요약

        Args:
            task: 작업 정보
            search_results: 검색 결과 목록

        Returns:
            (요약 텍스트, 출처 URL 목록)
        """
        # 검색 결과 형식화
        formatted_sources = self._format_sources(search_results)

        # Prompt 구성
        prompt = task_summarizer_instructions.format(
            task_title=task.title,
            task_intent=task.intent,
            task_query=task.query,
            search_results=formatted_sources,
        )

        # Agent 호출
        summary = self._agent.run(prompt)

        # 출처 URL 추출
        source_urls = [result["url"] for result in search_results]

        return summary, source_urls

    def _format_sources(self, search_results: List[dict]) -> str:
        """검색 결과 형식화

        다음을 포함하여 검색 결과를 읽기 가능한 텍스트로 형식화:
        - 일련 번호
        - 제목
        - URL
        - 스니펫
        """
        formatted = []
        for idx, result in enumerate(search_results, start=1):
            formatted.append(
                f"[{idx}] {result['title']}\n"
                f"URL: {result['url']}\n"
                f"Snippet: {result['snippet']}\n"
            )
        return "\n".join(formatted)
```

### 보고서 구조 설계

최종 보고서에는 다음 부분이 포함되어야 합니다:

## 참고문헌

### 작업 1: 멀티모달 모델이란 무엇인가
- https://example.com/multimodal-model-definition
...

### 작업 2: 최신 멀티모달 모델은 무엇인가
- https://example.com/gpt4v
...
...

### 14.5.3 보고서 생성 서비스

`ReportingService`는 보고서 생성 Agent를 호출하여 모든 하위 작업의 요약을 통합하는 역할을 합니다. 이것은 연구 과정의 마지막 단계로, 최종 연구 보고서를 생성합니다.

책임:

1. **하위 작업 요약 형식화**: 모든 하위 작업 요약을 통합된 형식으로 형식화
2. **보고서 Prompt 구성**: 연구 주제와 하위 작업 요약을 기반으로 Prompt 구성
3. **보고서 Agent 호출**: Report Writer Agent를 호출하여 최종 보고서 생성
4. **인용 정리**: 모든 출처 인용을 참고문헌 섹션으로 정리

**핵심 코드 구현**:

```python
from typing import List, Callable, Optional, Tuple

from hello_agents import HelloAgentsLLM
from hello_agents import ToolAwareSimpleAgent
from models import TodoItem
from prompts import report_writer_instructions

class ReportingService:
    """보고서 생성 서비스"""

    def __init__(
        self,
        llm: HelloAgentsLLM,
        tool_call_listener: Optional[Callable] = None
    ):
        self._llm = llm
        self._tool_call_listener = tool_call_listener

        # 보고서 Agent 생성
        self._agent = ToolAwareSimpleAgent(
            name="Report Writer",
            system_prompt="You are a report writing expert, skilled at integrating information and generating structured reports.",
            llm=llm,
            tool_call_listener=tool_call_listener
        )

    def generate_report(
        self,
        research_topic: str,
        task_summaries: List[Tuple[TodoItem, str, List[str]]]
    ) -> str:
        """최종 보고서 생성

        Args:
            research_topic: 연구 주제
            task_summaries: 하위 작업 요약 목록, 각 요소는 (작업, 요약, 출처 URL 목록)

        Returns:
            최종 보고서 (Markdown 형식)
        """
        # 하위 작업 요약 형식화
        formatted_summaries = self._format_summaries(task_summaries)

        # Prompt 구성
        prompt = report_writer_instructions.format(
            research_topic=research_topic,
            task_summaries=formatted_summaries,
        )

        # Agent 호출
        report = self._agent.run(prompt)

        return report

    def _format_summaries(
        self,
        task_summaries: List[Tuple[TodoItem, str, List[str]]]
    ) -> str:
        """하위 작업 요약 형식화

        다음을 포함하여 모든 하위 작업 요약을 통합된 형식으로 형식화:
        - 작업 일련 번호
        - 작업 제목
        - 작업 의도
        - 요약 내용
        - 출처 URL
        """
        formatted = []
        for idx, (task, summary, source_urls) in enumerate(task_summaries, start=1):
            formatted.append(
                f"## Task {idx}: {task.title}\n\n"
                f"**Intent**: {task.intent}\n\n"
                f"{summary}\n\n"
                f"**Sources**:\n"
            )
            for url in source_urls:
                formatted.append(f"- {url}\n")
            formatted.append("\n")

        return "".join(formatted)
```

### 14.5.4 검색 스케줄링 서비스

`SearchService`는 검색 엔진을 스케줄링하고, 검색을 실행하며, 결과를 반환하는 역할을 합니다. 이것은 Agent와 SearchTool을 연결하는 다리입니다. 여기서는 SimpleAgent가 도구를 직접 호출하는 일반적인 방식을 채택하지 않고, 중간 계층을 통해 SearchTool의 실행 결과를 Agent에 반환합니다. 이렇게 하면 Agent가 얻은 정보를 처리하는 데 더 집중할 수 있습니다.

책임:

1. **검색 엔진 스케줄링**: 구성에 따라 검색 엔진 선택
2. **검색 실행**: SearchTool을 호출하여 검색 실행
3. **결과 처리**: 중복 제거, 토큰 제한, 형식화
4. **오류 처리**: 검색 실패 상황 처리

핵심 코드:

```python
from typing import List, Optional
import logging

from hello_agents.tools import SearchTool
from config import Configuration

logger = logging.getLogger(__name__)

class SearchService:
    """검색 스케줄링 서비스"""

    def __init__(self, config: Configuration):
        self.config = config

        # SearchTool 생성
        self.search_tool = SearchTool(backend="hybrid")

    def search(
        self,
        query: str,
        max_results: int = 5
    ) -> List[dict]:
        """검색 실행

        Args:
            query: 검색 쿼리
            max_results: 최대 결과 수

        Returns:
            검색 결과 목록
        """
        try:
            # SearchTool 호출
            raw_response = self.search_tool.run({
                "input": query,
                "backend": self.config.search_api.value,
                "mode": "structured",
                "max_results": max_results
            })

            # 결과 추출
            results = raw_response.get("results", [])

            # 결과 처리
            results = self._deduplicate_sources(results)
            results = self._limit_source_tokens(results)

            logger.info(f"검색 성공: {query}, {len(results)}개의 결과 반환")

            return results

        except Exception as e:
            logger.error(f"검색 실패: {query}, 오류: {e}")
            return []

    def _deduplicate_sources(self, sources: List[dict]) -> List[dict]:
        """중복 URL 제거"""
        seen_urls = set()
        unique_sources = []

        for source in sources:
            url = source.get("url", "")
            if url and url not in seen_urls:
                seen_urls.add(url)
                unique_sources.append(source)

        return unique_sources

    def _limit_source_tokens(
        self,
        sources: List[dict],
        max_tokens_per_source: int = 2000
    ) -> List[dict]:
        """소스당 토큰 수 제한"""
        limited_sources = []

        for source in sources:
            snippet = source.get("snippet", "")

            # 간단한 토큰 추정: 1토큰 ≈ 4자
            max_chars = max_tokens_per_source * 4

            if len(snippet) > max_chars:
                snippet = snippet[:max_chars] + "..."

            limited_sources.append({
                **source,
                "snippet": snippet
            })

        return limited_sources
```

그림 14.8과 같이 구성에 따라 검색 엔진을 선택합니다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-8.png" alt="" width="85%"/>
  <p>그림 14.8 검색 엔진 스케줄링 과정</p>
</div>

**스케줄링 로직**:

1. **구성 읽기**: `.env` 파일에서 `SEARCH_API` 구성 읽기
2. **엔진 선택**: 구성에 따라 검색 엔진 선택 (tavily, duckduckgo, perplexity 등)
3. **검색 실행**: SearchTool을 호출하여 검색 실행
4. **결과 처리**: 중복 제거, 토큰 제한, 형식화
5. **결과 반환**: 처리된 검색 결과 반환

효율성을 높이고 비용을 줄이기 위해 검색 결과 캐싱을 추가할 수 있습니다:

```python
import hashlib
import json
from pathlib import Path

class SearchService:
    def __init__(self, config: Configuration):
        self.config = config
        self.search_tool = SearchTool(backend="hybrid")

        # 캐시 디렉터리
        self.cache_dir = Path("./cache/search")
        self.cache_dir.mkdir(parents=True, exist_ok=True)

    def search(
        self,
        query: str,
        max_results: int = 5,
        use_cache: bool = True
    ) -> List[dict]:
        """검색 실행 (캐시 포함)"""
        # 캐시 키 생성
        cache_key = self._generate_cache_key(query, max_results)
        cache_file = self.cache_dir / f"{cache_key}.json"

        # 캐시에서 읽기 시도
        if use_cache and cache_file.exists():
            logger.info(f"캐시에서 검색 결과 읽기: {query}")
            with open(cache_file, "r", encoding="utf-8") as f:
                return json.load(f)

        # 검색 실행
        results = self._execute_search(query, max_results)

        # 캐시에 저장
        if use_cache and results:
            with open(cache_file, "w", encoding="utf-8") as f:
                json.dump(results, f, ensure_ascii=False, indent=2)

        return results

    def _generate_cache_key(self, query: str, max_results: int) -> str:
        """캐시 키 생성"""
        # 쿼리와 최대 결과 수를 사용하여 MD5 해시 생성
        content = f"{query}_{max_results}_{self.config.search_api.value}"
        return hashlib.md5(content.encode()).hexdigest()
```

네 가지 핵심 서비스(PlanningService, SummarizationService, ReportingService, SearchService)를 통해 완전한 연구 과정을 구축했습니다. 이 서비스들은 각각 역할을 수행하며 명확한 인터페이스를 통해 협력하여 연구 주제에서 최종 보고서까지의 자동화 과정을 실현합니다.

## 14.6 프론트엔드 인터랙션 설계

이전 섹션에서 우리는 완전한 백엔드 시스템을 구현했습니다. 이 섹션에서는 전체 화면 모달 대화 UI, 실시간 진행 표시, 연구 결과 시각화를 포함한 프론트엔드 인터랙션 설계를 자세히 소개합니다.

### 14.6.1 전체 화면 모달 대화 UI 설계

심층 연구 도우미는 전체 화면 모달 대화 UI 설계를 채택하며, 다음과 같은 장점이 있습니다:

1. **몰입형 경험**: 전체 화면 표시로 산만함 없이 연구에 집중
2. **명확한 계층 구조**: 메인 페이지와 연구 페이지가 분리되어 계층이 명확
3. **닫기 용이**: 닫기 버튼을 클릭하거나 ESC 키를 눌러 메인 페이지로 돌아감
4. **반응형 디자인**: 다양한 화면 크기에 적응

그림 14.9와 같이, 전체 화면 모달 대화에는 다음 부분이 포함됩니다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-9.png" alt="" width="85%"/>
  <p>그림 14.9 전체 화면 모달 대화 UI</p>
</div>

**UI 컴포넌트**:

1. **상단 바**: 연구 주제와 닫기 버튼 포함
2. **진행 영역**: 현재 연구 진행 상황 표시 (계획, 실행, 보고)
3. **내용 영역**: 연구 결과 표시 (Markdown 형식)
4. **하단 바**: 상태 정보 표시 (예: "연구 중...", "완료")

해당 Vue 구현은 다음과 같습니다 (ResearchModal.vue):

```vue
<template>
  <div v-if="isOpen" class="modal-overlay" @click.self="close">
    <div class="modal-container">
      <!-- 상단 바 -->
      <div class="modal-header">
        <h2>{{ researchTopic }}</h2>
        <button @click="close" class="close-button">
          <svg><!-- 닫기 아이콘 --></svg>
        </button>
      </div>

      <!-- 진행 영역 -->
      <div class="progress-section">
        <div class="progress-bar">
          <div
            class="progress-fill"
            :style="{ width: progressPercentage + '%' }"
          ></div>
        </div>
        <div class="progress-text">{{ progressText }}</div>
      </div>

      <!-- 내용 영역 -->
      <div class="content-section">
        <div v-if="isLoading" class="loading-spinner">
          <div class="spinner"></div>
          <p>연구 중입니다, 잠시만 기다려 주세요...</p>
        </div>

        <div v-else class="markdown-content" v-html="renderedMarkdown"></div>
      </div>

      <!-- 하단 바 -->
      <div class="modal-footer">
        <span class="status-text">{{ statusText }}</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch } from 'vue'
import { marked } from 'marked'

interface Props {
  isOpen: boolean
  researchTopic: string
}

const props = defineProps<Props>()
const emit = defineEmits<{
  close: []
}>()

// 상태
const isLoading = ref(true)
const progressPercentage = ref(0)
const progressText = ref('준비 중...')
const statusText = ref('연구 중...')
const markdownContent = ref('')

// Markdown 렌더링
const renderedMarkdown = computed(() => {
  return marked(markdownContent.value)
})

// 모달 닫기
const close = () => {
  emit('close')
}

// ESC 키 리스닝
const handleKeydown = (e: KeyboardEvent) => {
  if (e.key === 'Escape') {
    close()
  }
}

// 마운트 시 키보드 리스너 추가
watch(() => props.isOpen, (isOpen) => {
  if (isOpen) {
    document.addEventListener('keydown', handleKeydown)
  } else {
    document.removeEventListener('keydown', handleKeydown)
  }
})
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}
...
</style>
```

다양한 화면 크기에 적응하기 위해 미디어 쿼리를 추가합니다:

```css
/* 태블릿 기기 */
@media (max-width: 768px) {
  .modal-container {
    width: 95vw;
    height: 95vh;
  }

  .modal-header,
  .progress-section,
  .content-section,
  .modal-footer {
    padding: 15px 20px;
  }
}

/* 모바일 기기 */
@media (max-width: 480px) {
  .modal-container {
    width: 100vw;
    height: 100vh;
    border-radius: 0;
  }

  .modal-header h2 {
    font-size: 18px;
  }
}
```

### 14.6.2 실시간 진행 표시

심층 연구 도우미는 SSE를 사용하여 실시간 진행 표시를 구현합니다. SSE는 서버 푸시 기술로, 서버가 클라이언트에 능동적으로 데이터를 보낼 수 있으며, 이는 프로토콜 장에서도 설명되어 있습니다.

그림 14.10과 같이, SSE 과정에는 다음 단계가 포함됩니다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/14-figures/14-10.png" alt="" width="85%"/>
  <p>그림 14.10 SSE 과정</p>
</div>

**과정 설명**:

1. **클라이언트가 요청 시작**: `/api/research`에 POST 요청을 보내고, 연구 주제 포함
2. **서버가 SSE 연결 수립**: `text/event-stream` 응답 반환
3. **서버가 진행 상황 푸시**: 주기적으로 연구 진행 상황 푸시 (계획, 실행, 보고)
4. **클라이언트가 진행 상황 수신**: SSE 이벤트를 리스닝하고 UI 업데이트
5. **연구 완료**: 서버가 최종 보고서를 푸시하고 연결 종료

프론트엔드-백엔드 프로젝트에서 SSE를 사용하려면 다음 구성도 필요합니다.

**백엔드 FastAPI SSE 엔드포인트**:

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from typing import AsyncGenerator
import asyncio
import json

app = FastAPI()

async def research_stream(topic: str) -> AsyncGenerator[str, None]:
    """연구 스트리밍 제너레이터

    SSE 형식 데이터 생성:
    data: {"type": "progress", "data": {...}}

    """
    try:
        # 1. 계획 단계
        yield f"data: {json.dumps({'type': 'progress', 'stage': 'planning', 'percentage': 10, 'text': '연구 작업 계획 중...'})}\n\n"

        # PlanningService 호출
        todo_items = await planning_service.plan_todo_list(topic)

        yield f"data: {json.dumps({'type': 'plan', 'data': [item.dict() for item in todo_items]})}\n\n"

        # 2. 실행 단계
        task_summaries = []
        for idx, task in enumerate(todo_items, start=1):
            # 진행 업데이트
            percentage = 10 + (idx / len(todo_items)) * 70
            yield f"data: {json.dumps({'type': 'progress', 'stage': 'executing', 'percentage': percentage, 'text': f'작업 {idx}/{len(todo_items)} 연구 중: {task.title}'})}\n\n"

            # 검색
            search_results = await search_service.search(task.query)

            # 요약
            summary, source_urls = await summarization_service.summarize_task(task, search_results)

            task_summaries.append((task, summary, source_urls))

            # 작업 요약 푸시
            yield f"data: {json.dumps({'type': 'task_summary', 'task_id': task.id, 'summary': summary})}\n\n"

        # 3. 보고 단계
        yield f"data: {json.dumps({'type': 'progress', 'stage': 'reporting', 'percentage': 90, 'text': '최종 보고서 생성 중...'})}\n\n"

        # 보고서 생성
        report = await reporting_service.generate_report(topic, task_summaries)

        # 최종 보고서 푸시
        yield f"data: {json.dumps({'type': 'report', 'data': report})}\n\n"

        # 완료
        yield f"data: {json.dumps({'type': 'progress', 'stage': 'completed', 'percentage': 100, 'text': '연구 완료!'})}\n\n"

    except Exception as e:
        # 오류 처리
        yield f"data: {json.dumps({'type': 'error', 'message': str(e)})}\n\n"

@app.post("/api/research")
async def research(request: ResearchRequest):
    """연구 엔드포인트 (SSE)"""
    return StreamingResponse(
        research_stream(request.topic),
        media_type="text/event-stream",
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive",
        }
    )
```

**프론트엔드에서 EventSource를 사용하여 SSE 수신**:

```typescript
// composables/useResearch.ts
import { ref } from 'vue'

export function useResearch() {
  const isLoading = ref(false)
  const progressPercentage = ref(0)
  const progressText = ref('')
  const markdownContent = ref('')
  const error = ref<string | null>(null)

  const startResearch = (topic: string) => {
    isLoading.value = true
    error.value = null

    // EventSource 생성
    const eventSource = new EventSource(`/api/research?topic=${encodeURIComponent(topic)}`)

    // 메시지 리스닝
    eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data)

      switch (data.type) {
        case 'progress':
          progressPercentage.value = data.percentage
          progressText.value = data.text
          break

        case 'plan':
          // 계획 결과 표시
          console.log('계획 결과:', data.data)
          break

        case 'task_summary':
          // 작업 요약을 Markdown에 추가
          markdownContent.value += `\n\n## 작업 ${data.task_id}\n\n${data.summary}`
          break

        case 'report':
          // 최종 보고서 표시
          markdownContent.value = data.data
          break

        case 'error':
          error.value = data.message
          eventSource.close()
          isLoading.value = false
          break

        case 'completed':
          eventSource.close()
          isLoading.value = false
          break
      }
    }

    // 오류 처리
    eventSource.onerror = (err) => {
      console.error('SSE 오류:', err)
      error.value = '연결 실패, 다시 시도해 주세요'
      eventSource.close()
      isLoading.value = false
    }
  }

  return {
    isLoading,
    progressPercentage,
    progressText,
    markdownContent,
    error,
    startResearch,
  }
}
```

**컴포넌트에서 사용**:

```vue
<script setup lang="ts">
import { useResearch } from '@/composables/useResearch'

const {
  isLoading,
  progressPercentage,
  progressText,
  markdownContent,
  error,
  startResearch
} = useResearch()

const handleStartResearch = (topic: string) => {
  startResearch(topic)
}
</script>
```

### 14.6.3 연구 결과 시각화

연구 결과는 Markdown 형식으로 표시되며, 제목, 단락, 목록, 인용 등의 요소를 포함합니다. 우리는 `marked` 라이브러리를 사용하여 Markdown을 HTML로 변환하고 커스텀 스타일을 추가합니다.

**Markdown 렌더링**:

```typescript
import { marked } from 'marked'

// marked 구성
marked.setOptions({
  breaks: true,  // 줄바꿈 지원
  gfm: true,     // GitHub Flavored Markdown 지원
})

// 렌더링
const renderedHtml = marked(markdownContent.value)
```

연구 보고서에는 많은 출처 인용이 포함되어 있으며, 이를 특별히 처리해야 합니다:

```markdown
## 참고문헌

### 작업 1: Datawhale 기본 정보
- [Datawhale GitHub](https://github.com/datawhalechina)
- [Datawhale 공식 사이트](https://datawhale.club)

### 작업 2: Datawhale 주요 프로젝트
- [Hello-Agents 튜토리얼](https://github.com/datawhalechina/Hello-Agents)
...
```

전체 화면 모달 대화 UI, SSE 실시간 진행 표시, Markdown 결과 시각화를 통해 사용자 친화적인 프론트엔드 인터페이스를 구축했습니다. 사용자는 연구 진행 상황을 명확하게 볼 수 있고, 아름다운 형식으로 연구 결과를 확인할 수 있습니다.

## 14.7 장 요약

이번 장에서 우리는 처음부터 완전한 자동화 심층 연구 에이전트 시스템을 구축했습니다. 핵심 포인트를 정리해 보겠습니다:

**(1) TODO 기반 연구 패러다임**

우리는 새로운 연구 패러다임인 TODO 기반 연구를 제안했습니다. 이 패러다임은 복잡한 연구 주제를 실행 가능한 하위 작업으로 분해하고, 세 단계를 통해 연구를 완료합니다:

- **계획 단계**: 연구 주제를 3~5개의 하위 작업으로 분해하며, 각 하위 작업에는 제목, 의도, 검색 쿼리가 포함됨
- **실행 단계**: 각 하위 작업에 대해 검색과 요약을 실행하여 구조화된 지식 생성
- **보고 단계**: 모든 하위 작업의 요약을 통합하여 최종 연구 보고서 생성

이 패러다임의 장점:

1. **강한 제어 가능성**: 각 하위 작업에 명확한 목표와 범위 있음
2. **신뢰할 수 있는 품질**: 전용 Agent가 각 단계의 품질 보장
3. **디버깅 용이**: 각 하위 작업을 개별적으로 디버깅 가능
4. **좋은 확장성**: 새로운 하위 작업을 쉽게 추가하거나 기존 하위 작업 수정 가능

**(2) 세 Agent 협업 시스템**

우리는 세 개의 전문화된 Agent를 설계하여 각각 역할을 수행합니다:

- **TODO Planner (연구 계획 전문가)**: 연구 주제를 하위 작업으로 분해하는 역할
- **Task Summarizer (작업 요약 전문가)**: 각 하위 작업의 검색 결과를 요약하는 역할
- **Report Writer (보고서 작성 전문가)**: 모든 하위 작업의 요약을 통합하여 최종 보고서를 생성하는 역할

이 설계의 장점:

1. **명확한 책임**: 각 Agent가 특정 작업에 집중
2. **프롬프트 최적화**: 각 Agent에 전문화된 Prompt를 커스터마이징 가능
3. **유지 보수 용이**: 하나의 Agent 수정이 다른 Agent에 영향을 미치지 않음
4. **품질 보장**: 각 Agent가 해당 분야의 "전문가"

**(3) ToolAwareSimpleAgent 설계**

우리는 HelloAgents 프레임워크의 `SimpleAgent`를 확장하여 `ToolAwareSimpleAgent`를 구현했습니다. 이 Agent는 도구 호출 리스닝 기능을 가지며 다음을 할 수 있습니다:

- **도구 호출 리스닝**: 콜백 함수를 통해 각 도구 호출을 리스닝
- **실시간 피드백**: 도구 호출 정보를 실시간으로 프론트엔드에 푸시
- **디버깅 지원**: 모든 도구 호출을 기록하여 디버깅 용이

이 Agent는 HelloAgents 프레임워크에 통합되어 다른 프로젝트에서 재사용할 수 있습니다.

**(4) 도구 시스템 통합**

우리는 HelloAgents 프레임워크의 도구 시스템을 완전히 활용했습니다:

- **SearchTool**: 더 많은 검색 엔진 지원으로 확장 (Tavily, DuckDuckGo, Perplexity 등)
- **NoteTool**: 연구 진행 상황을 지속하여 복구 및 감사 지원
- **ToolRegistry**: 모든 도구를 통합 관리하고 커스텀 확장 지원

구성 기반 설계를 통해 사용자는 코드를 수정하지 않고 쉽게 검색 엔진을 전환할 수 있습니다.

**(5) 핵심 서비스 구현**

우리는 Agent와 도구를 연결하는 네 가지 핵심 서비스를 구현했습니다:

- **PlanningService**: 계획 Agent 호출, JSON 파싱, 형식 검증
- **SummarizationService**: 요약 Agent 호출, 검색 결과 처리, 출처 추출
- **ReportingService**: 보고서 Agent 호출, 요약 통합, 보고서 생성
- **SearchService**: 검색 엔진 스케줄링, 결과 처리, 오류 강등, 결과 캐싱

이 서비스들은 각각 역할을 수행하며 명확한 인터페이스를 통해 협력하여 연구 주제에서 최종 보고서까지의 자동화 과정을 실현합니다.

**(6) 프론트엔드 인터랙션 설계**

우리는 사용자 친화적인 프론트엔드 인터페이스를 설계했습니다:

- **전체 화면 모달 대화**: 몰입형 경험, 명확한 계층 구조
- **SSE 실시간 진행**: 연구 진행 상황을 실시간으로 표시하여 좋은 사용자 경험
- **Markdown 시각화**: 아름다운 형식, 명확한 구조

Vue 3 + TypeScript + SSE 기술 스택을 통해 현대적인 웹 애플리케이션을 구현했습니다.

이 지식은 심층 연구 도우미에만 적용되는 것이 아니라, 다른 AI 애플리케이션에도 적용할 수 있습니다. 독자들이 이번 장을 바탕으로 더 많은 가능성을 탐구하고 더 강력한 AI 시스템을 구축하기를 바랍니다.

다음 장에서는 게임 엔진과 결합된 멀티 에이전트 시스템인 Cyber Town을 구축하여 Agent 간의 복잡한 인터랙션과 협업 패턴을 탐구합니다. 기대해 주세요!
