# 8장 메모리와 검색

이전 장들에서 HelloAgents 프레임워크의 기본 아키텍처를 구축하고, 다양한 에이전트 패러다임과 도구 시스템을 구현했다. 그러나 프레임워크에는 여전히 핵심 기능이 빠져 있다: **메모리**다. 에이전트가 이전 상호작용을 기억하거나 과거 경험으로부터 학습하지 못한다면, 지속적인 대화나 복잡한 작업에서 성능이 크게 제한될 것이다.

이 장에서는 7장에서 구축한 프레임워크를 기반으로 HelloAgents에 두 가지 핵심 기능을 추가한다: **메모리 시스템**과 **검색 증강 생성(RAG)**. "프레임워크 확장 + 지식 보급"이라는 접근 방식을 채택하여, 구축 과정에서 메모리와 RAG의 이론적 기반을 깊이 이해하고, 최종적으로 완전한 메모리와 지식 검색 기능을 갖춘 에이전트 시스템을 구현할 것이다.


## 8.1 인지과학에서 에이전트 메모리로

### 8.1.1 인간 메모리 시스템으로부터의 영감

에이전트의 메모리 시스템을 구축하기 전에, 먼저 인지과학의 관점에서 인간이 정보를 처리하고 저장하는 방식을 이해해보자. 인간의 기억은 다층적 인지 시스템으로, 정보를 저장할 뿐만 아니라 중요도, 시간, 맥락에 기반하여 정보를 분류하고 조직화한다. 인지심리학은 기억의 구조와 과정을 이해하기 위한 고전적인 이론 프레임워크를 제공하며<sup>[1]</sup>, 그림 8.1에 나타나 있다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-1.png" alt="인간 메모리 시스템 구조" width="85%"/>
  <p>그림 8.1 인간 메모리 시스템의 계층 구조</p>
</div>

인지심리학 연구에 따르면, 인간의 기억은 다음과 같은 수준으로 나눌 수 있다:

1. **감각 기억**: 지속 시간이 매우 짧고(0.5~3초), 용량이 방대하며, 감각기관이 수신한 모든 정보를 일시적으로 저장하는 역할을 한다
2. **작업 기억**: 지속 시간이 짧고(15~30초), 용량이 제한적이며(7±2개), 현재 작업의 정보 처리를 담당한다
3. **장기 기억**: 지속 시간이 길고(평생 지속될 수 있음), 용량이 거의 무한하며, 다음과 같이 세분된다:
   - **절차 기억**: 기술과 습관 (예: 자전거 타기)
   - **선언 기억**: 언어로 표현할 수 있는 지식, 다음과 같이 세분된다:
     - **의미 기억**: 일반적인 지식과 개념 (예: "파리는 프랑스의 수도다")
     - **일화 기억**: 개인적인 경험과 사건 (예: "어제 회의 내용")

### 8.1.2 에이전트에게 메모리와 RAG가 필요한 이유

인간 메모리 시스템의 설계에서 영감을 받아, 에이전트에도 유사한 메모리 기능이 필요한 이유를 이해할 수 있다. 인간 지능의 중요한 특성은 과거 경험을 기억하고, 그로부터 학습하며, 이러한 경험을 새로운 상황에 적용하는 능력이다. 마찬가지로, 진정으로 지능적인 에이전트에도 메모리 기능이 필요하다. LLM 기반 에이전트의 경우, 일반적으로 두 가지 근본적인 한계에 직면한다: **대화 상태의 망각**과 **내장 지식의 한계**.

(1) 한계 1: 상태 비저장성으로 인한 대화 망각

현재의 대형 언어 모델은 강력하지만, **상태 비저장(stateless)** 방식으로 설계되어 있다. 이는 각 사용자 요청(또는 API 호출)이 독립적이고 연관성 없는 계산임을 의미한다. 모델 자체는 이전 대화 내용을 자동으로 "기억"하지 않는다. 이로 인해 여러 문제가 발생한다:

1. **맥락 손실**: 긴 대화에서 초반의 중요한 정보가 컨텍스트 윈도우 한계로 인해 손실될 수 있다
2. **개인화 부재**: 에이전트가 사용자의 선호, 습관, 또는 특정 요구를 기억하지 못한다
3. **학습 능력 제한**: 과거의 성공이나 실패로부터 학습하고 개선할 수 없다
4. **일관성 문제**: 다중 턴 대화에서 모순된 답변을 제공할 수 있다

구체적인 예시를 통해 이 문제를 이해해보자:

```python
# 7장의 Agent 사용 방법
from hello_agents import SimpleAgent, HelloAgentsLLM

agent = SimpleAgent(name="학습 도우미", llm=HelloAgentsLLM())

# 첫 번째 대화
response1 = agent.run("제 이름은 장삼입니다, 저는 Python을 배우고 있으며 기본 문법을 익혔습니다")
print(response1)  # "훌륭합니다! Python 기본 문법은 프로그래밍의 중요한 기초입니다..."

# 두 번째 대화 (새 세션)
response2 = agent.run("제 학습 진도를 기억하시나요?")
print(response2)  # "죄송합니다, 당신의 학습 진도를 알지 못합니다..."
```

이 문제를 해결하기 위해, 프레임워크에 메모리 시스템을 도입해야 한다.

(2) 한계 2: 모델 내장 지식의 한계

대화 기록을 망각하는 것 외에도, LLM의 또 다른 핵심 한계는 지식이 **정적이고 제한적**이라는 점이다. 이 지식은 전적으로 훈련 데이터에서 비롯되며, 다음과 같은 일련의 문제를 야기한다:

1. **지식의 시의성**: 대형 모델은 훈련 데이터 마감일이 있어 최신 정보에 접근할 수 없다
2. **도메인 특화 지식**: 일반 모델은 특정 도메인에서 충분한 깊이가 부족할 수 있다
3. **사실 정확성**: 검색 검증을 통해 모델 환각을 줄인다
4. **설명 가능성**: 정보 출처를 제공하여 답변 신뢰성을 높인다

이러한 한계를 극복하기 위해 RAG 기술이 등장했다. 핵심 아이디어는 모델이 답변을 생성하기 전에 외부 지식베이스(문서, 데이터베이스, API 등)에서 가장 관련성 높은 정보를 검색하고, 이 정보를 컨텍스트로 모델에 제공하는 것이다.

### 8.1.3 메모리와 RAG 시스템 아키텍처 설계

7장에서 구축한 프레임워크 기반과 인지과학으로부터의 영감을 바탕으로, 계층화된 메모리 및 RAG 시스템 아키텍처를 설계했다 (그림 8.2 참조). 이 아키텍처는 인간 메모리 시스템의 계층 구조를 따를 뿐만 아니라, 엔지니어링 구현의 확장성도 충분히 고려했다. 구현에서는 메모리와 RAG를 두 개의 독립적인 도구로 설계한다: `memory_tool`은 대화 중 상호작용 정보의 저장 및 유지를 담당하고, `rag_tool`은 사용자가 제공한 지식베이스에서 관련 정보를 맥락으로 검색하며, 중요한 검색 결과를 메모리 시스템에 자동으로 저장할 수 있다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-2.png" alt="HelloAgents 메모리 및 RAG 시스템 아키텍처" width="95%"/>
  <p>그림 8.2 HelloAgents 메모리 및 RAG 시스템의 전체 아키텍처</p>
</div>

메모리 시스템은 4계층 아키텍처 설계를 채택한다:

```
HelloAgents 메모리 시스템
├── 인프라 계층
│   ├── MemoryManager - 메모리 관리자 (통합 스케줄링 및 조정)
│   ├── MemoryItem - 메모리 데이터 구조 (표준화된 메모리 항목)
│   ├── MemoryConfig - 설정 관리 (시스템 매개변수 설정)
│   └── BaseMemory - 메모리 기본 클래스 (공통 인터페이스 정의)
├── 메모리 유형 계층
│   ├── WorkingMemory - 작업 기억 (임시 정보, TTL 관리)
│   ├── EpisodicMemory - 일화 기억 (특정 사건, 시간 순서)
│   ├── SemanticMemory - 의미 기억 (추상적 지식, 그래프 관계)
│   └── PerceptualMemory - 지각 기억 (멀티모달 데이터)
├── 저장 백엔드 계층
│   ├── QdrantVectorStore - 벡터 저장 (고성능 의미 검색)
│   ├── Neo4jGraphStore - 그래프 저장 (지식 그래프 관리)
│   └── SQLiteDocumentStore - 문서 저장 (구조화된 영속성)
└── 임베딩 서비스 계층
    ├── DashScopeEmbedding - 통의 치엔원 임베딩 (클라우드 API)
    ├── LocalTransformerEmbedding - 로컬 임베딩 (오프라인 배포)
    └── TFIDFEmbedding - TFIDF 임베딩 (경량 폴백)
```

RAG 시스템은 외부 지식의 획득 및 활용에 집중한다:

```
HelloAgents RAG 시스템
├── 문서 처리 계층
│   ├── DocumentProcessor - 문서 처리기 (다중 포맷 파싱)
│   ├── Document - 문서 객체 (메타데이터 관리)
│   └── Pipeline - RAG 파이프라인 (엔드투엔드 처리)
├── 임베딩 계층
│   └── 통합 임베딩 인터페이스 - 메모리 시스템의 임베딩 서비스 재사용
├── 벡터 저장 계층
│   └── QdrantVectorStore - 벡터 데이터베이스 (네임스페이스 격리)
└── 지능형 Q&A 계층
    ├── 다중 전략 검색 - 벡터 검색 + MQE + HyDE
    ├── 컨텍스트 구성 - 지능형 단편 병합 및 잘라내기
    └── LLM 강화 생성 - 컨텍스트 기반 정확한 Q&A
```

### 8.1.4 학습 목표와 빠른 체험

8장의 핵심 학습 내용을 먼저 살펴보자:

```
hello-agents/
├── hello_agents/
│   ├── memory/                   # 메모리 시스템 모듈
│   │   ├── base.py               # 기본 데이터 구조 (MemoryItem, MemoryConfig, BaseMemory)
│   │   ├── manager.py            # 메모리 관리자 (통합 조정 및 스케줄링)
│   │   ├── embedding.py          # 통합 임베딩 서비스 (DashScope/Local/TFIDF)
│   │   ├── types/                # 메모리 유형 구현
│   │   │   ├── working.py        # 작업 기억 (TTL 관리, 순수 인메모리)
│   │   │   ├── episodic.py       # 일화 기억 (이벤트 시퀀스, SQLite+Qdrant)
│   │   │   ├── semantic.py       # 의미 기억 (지식 그래프, Qdrant+Neo4j)
│   │   │   └── perceptual.py     # 지각 기억 (멀티모달, SQLite+Qdrant)
│   │   ├── storage/              # 저장 백엔드 구현
│   │   │   ├── qdrant_store.py   # Qdrant 벡터 저장 (고성능 벡터 검색)
│   │   │   ├── neo4j_store.py    # Neo4j 그래프 저장 (지식 그래프 관리)
│   │   │   └── document_store.py # SQLite 문서 저장 (구조화된 영속성)
│   │   └── rag/                  # RAG 시스템
│   │       ├── pipeline.py       # RAG 파이프라인 (엔드투엔드 처리)
│   │       └── document.py       # 문서 처리기 (다중 포맷 파싱)
│   └── tools/builtin/            # 확장된 내장 도구
│       ├── memory_tool.py        # 메모리 도구 (에이전트 메모리 기능)
│       └── rag_tool.py           # RAG 도구 (지능형 Q&A 기능)
└──
```

**빠른 시작: HelloAgents 프레임워크 설치**

독자들이 이 장의 완전한 기능을 빠르게 체험할 수 있도록, 직접 설치 가능한 Python 패키지를 제공한다. 다음 명령어로 이 장에 해당하는 버전을 설치할 수 있다:

```bash
pip install "hello-agents[all]==0.2.0"
python -m spacy download zh_core_web_sm
python -m spacy download en_core_web_sm
```

또한 `.env`에 그래프 데이터베이스, 벡터 데이터베이스, LLM, 임베딩 솔루션 API를 설정해야 한다. 튜토리얼에서는 벡터 데이터베이스로 Qdrant, 그래프 데이터베이스로 Neo4J를 사용하며, 임베딩에는 바이리안 플랫폼을 우선 사용한다. API를 사용할 수 없는 경우 로컬 배포 모델 솔루션으로 전환할 수 있다.

```bash
# ================================
# Qdrant 벡터 데이터베이스 설정 - API 키 획득: https://cloud.qdrant.io/
# ================================
# Qdrant 클라우드 서비스 사용 (권장)
QDRANT_URL=https://your-cluster.qdrant.tech:6333
QDRANT_API_KEY=your_qdrant_api_key_here

# 또는 로컬 Qdrant 사용 (Docker 필요)
# QDRANT_URL=http://localhost:6333
# QDRANT_API_KEY=

# Qdrant 컬렉션 설정
QDRANT_COLLECTION=hello_agents_vectors
QDRANT_VECTOR_SIZE=384
QDRANT_DISTANCE=cosine
QDRANT_TIMEOUT=30

# ================================
# Neo4j 그래프 데이터베이스 설정 - API 키 획득: https://neo4j.com/cloud/aura/
# ================================
# Neo4j Aura 클라우드 서비스 사용 (권장)
NEO4J_URI=neo4j+s://your-instance.databases.neo4j.io
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=your_neo4j_password_here

# 또는 로컬 Neo4j 사용 (Docker 필요)
# NEO4J_URI=bolt://localhost:7687
# NEO4J_USERNAME=neo4j
# NEO4J_PASSWORD=hello-agents-password

# Neo4j 연결 설정
NEO4J_DATABASE=neo4j
NEO4J_MAX_CONNECTION_LIFETIME=3600
NEO4J_MAX_CONNECTION_POOL_SIZE=50
NEO4J_CONNECTION_TIMEOUT=60

# ==========================
# 임베딩 설정 예시 - 알리바바 클라우드 콘솔에서 획득: https://dashscope.aliyun.com/
# ==========================
# - 비어있으면 dashscope 기본값은 text-embedding-v3; 로컬 기본값은 sentence-transformers/all-MiniLM-L6-v2
EMBED_MODEL_TYPE=dashscope
EMBED_MODEL_NAME=
EMBED_API_KEY=
EMBED_BASE_URL=
```

이 장에서의 학습은 두 가지 방식으로 진행할 수 있다:

1. **체험 학습**: `pip`을 사용해 프레임워크를 직접 설치하고, 예제 코드를 실행하여 다양한 기능을 빠르게 체험한다
2. **심층 학습**: 장 내용을 따라 각 구성 요소를 처음부터 직접 구현하고, 프레임워크의 설계 철학과 구현 세부 사항을 깊이 이해한다

"먼저 체험하고, 그 다음에 구현한다"는 학습 경로를 권장한다. 이 장에서는 완전한 테스트 파일을 제공한다. 핵심 함수를 재작성하고 테스트를 실행하여 구현이 올바른지 검증할 수 있다.

7장에서 수립한 설계 원칙에 따라, 새로운 에이전트 클래스를 만들지 않고 메모리와 RAG 기능을 표준 도구로 캡슐화한다. 시작하기 전에, Hello-agents를 사용하여 메모리와 RAG 기능을 갖춘 에이전트를 구축하는 것을 30초 만에 체험해보자!

```python
# 같은 폴더의 .env에 LLM API 설정
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import MemoryTool, RAGTool

# LLM 인스턴스 생성
llm = HelloAgentsLLM()

# 에이전트 생성
agent = SimpleAgent(
    name="지능형 어시스턴트",
    llm=llm,
    system_prompt="당신은 메모리와 지식 검색 기능을 갖춘 AI 어시스턴트입니다"
)

# 도구 레지스트리 생성
tool_registry = ToolRegistry()

# 메모리 도구 추가
memory_tool = MemoryTool(user_id="user123")
tool_registry.register_tool(memory_tool)

# RAG 도구 추가
rag_tool = RAGTool(knowledge_base_path="./knowledge_base")
tool_registry.register_tool(rag_tool)

# 에이전트에 도구 설정
agent.tool_registry = tool_registry

# 대화 시작
response = agent.run("안녕하세요! 제 이름은 장삼이고, Python 개발자입니다. 기억해주세요")
print(response)
```

모든 것이 올바르게 설정되었다면 다음과 같은 내용을 볼 수 있다:

```bash
[OK] SQLite database tables and indexes created
[OK] SQLite document storage initialized: ./memory_data\memory.db
INFO:hello_agents.memory.storage.qdrant_store:✅ Successfully connected to Qdrant cloud service: https://0c517275-2ad0-4442-8309-11c36dc7e811.us-east-1-1.aws.cloud.qdrant.io:6333
INFO:hello_agents.memory.storage.qdrant_store:✅ Using existing Qdrant collection: hello_agents_vectors
INFO:hello_agents.memory.types.semantic:✅ Embedding model ready, dimension: 1024
INFO:hello_agents.memory.types.semantic:✅ Qdrant vector database initialization complete
INFO:hello_agents.memory.storage.neo4j_store:✅ Successfully connected to Neo4j cloud service: neo4j+s://851b3a28.databases.neo4j.io
INFO:hello_agents.memory.types.semantic:✅ Neo4j graph database initialization complete
INFO:hello_agents.memory.storage.neo4j_store:✅ Neo4j index creation complete
INFO:hello_agents.memory.types.semantic:✅ Neo4j graph database initialization complete
INFO:hello_agents.memory.types.semantic:🏥 Database health status: Qdrant=✅, Neo4j=✅
INFO:hello_agents.memory.types.semantic:✅ Loaded Chinese spaCy model: zh_core_web_sm
INFO:hello_agents.memory.types.semantic:✅ Loaded English spaCy model: en_core_web_sm
INFO:hello_agents.memory.types.semantic:📚 Available language models: Chinese, English
INFO:hello_agents.memory.types.semantic:Enhanced semantic memory initialization complete (using Qdrant+Neo4j professional databases)
INFO:hello_agents.memory.manager:MemoryManager initialization complete, enabled memory types: ['working', 'episodic', 'semantic']
✅ Tool 'memory' registered.
INFO:hello_agents.memory.storage.qdrant_store:✅ Successfully connected to Qdrant cloud service: https://0c517275-2ad0-4442-8309-11c36dc7e811.us-east-1-1.aws.cloud.qdrant.io:6333
INFO:hello_agents.memory.storage.qdrant_store:✅ Using existing Qdrant collection: rag_knowledge_base
✅ RAG tool initialization successful: namespace=default, collection=rag_knowledge_base
✅ Tool 'rag' registered.
안녕하세요, 장삼 씨! 만나서 반갑습니다. Python 개발자로서 프로그래밍에 열정적이시겠군요. 기술적인 질문이 있거나 Python 관련 주제에 대해 이야기하고 싶으시면 언제든지 연락해 주세요. 최선을 다해 도와드리겠습니다. 지금 도움이 필요한 것이 있나요?
```

## 8.2 메모리 시스템: 에이전트에게 기억력 부여하기

### 8.2.1 메모리 시스템 워크플로

코드 구현 단계에 들어가기 전에, 메모리 시스템의 워크플로를 먼저 정의해야 한다. 이 워크플로는 인지과학의 메모리 모델을 참조하여 각 인지 단계를 특정 기술 컴포넌트와 연산으로 매핑한다. 이 매핑 관계를 이해하면 이후의 코드 구현에 도움이 된다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-3.png" alt="메모리 형성 과정" width="90%"/>
  <p>그림 8.3 메모리 형성의 인지 과정</p>
</div>

그림 8.3에서 보이듯, 인지과학 연구에 따르면 인간 기억의 형성은 다음 단계를 거친다:

1. **부호화**: 지각된 정보를 저장 가능한 형태로 변환
2. **저장**: 부호화된 정보를 메모리 시스템에 저장
3. **검색**: 필요에 따라 기억에서 관련 정보 추출
4. **강화**: 단기 기억을 장기 기억으로 변환
5. **망각**: 중요하지 않거나 오래된 정보 삭제

이로부터 영감을 받아 HelloAgents를 위한 완전한 메모리 시스템을 설계했다. 핵심 아이디어는 인간 뇌가 다양한 유형의 정보를 처리하는 방식을 모방하여, 기억을 여러 전문화된 모듈로 나누고 지능형 관리 메커니즘을 구축하는 것이다. 그림 8.4는 메모리 추가, 검색, 강화, 망각 등 핵심 링크를 포함한 이 시스템의 워크플로를 상세하게 보여준다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-4.png" alt="메모리 시스템 워크플로" width="95%"/>
  <p>그림 8.4 HelloAgents 메모리 시스템의 완전한 워크플로</p>
</div>

메모리 시스템은 네 가지 서로 다른 유형의 메모리 모듈로 구성되며, 각각 특정 응용 시나리오와 생명주기에 최적화되어 있다:

첫 번째는 **작업 기억(Working Memory)**으로, 에이전트의 "단기 기억" 역할을 하며 주로 현재 대화의 맥락 정보를 저장하는 데 사용된다. 빠른 접근과 응답을 보장하기 위해 용량을 의도적으로 제한하고(예: 기본 50개), 생명주기를 단일 세션에 바인딩하여 세션 종료 후 자동으로 정리된다.

두 번째는 **일화 기억(Episodic Memory)**으로, 특정 상호작용 사건과 에이전트의 학습 경험을 장기적으로 저장하는 역할을 한다. 작업 기억과 달리, 일화 기억에는 풍부한 맥락 정보가 포함되어 있으며 시간 순서나 주제별 회고적 검색을 지원한다. 에이전트가 과거 경험을 "복습"하고 학습하는 기반이 된다.

특정 사건에 대응하는 것이 **의미 기억(Semantic Memory)**으로, 더 추상적인 지식, 개념, 규칙을 저장한다. 예를 들어 대화를 통해 학습한 사용자 선호도, 장기적으로 따라야 할 지침, 또는 도메인 지식 포인트 등이 여기에 저장되기에 적합하다. 이 부분의 기억은 높은 지속성과 중요도를 가지며, 에이전트가 "지식 체계"를 형성하고 연상 추론을 수행하는 핵심이다.

마지막으로 점점 풍부해지는 멀티미디어와의 상호작용을 위해 **지각 기억(Perceptual Memory)**을 도입했다. 이 모듈은 이미지, 오디오 등 멀티모달 정보를 처리하고 교차 모달 검색을 지원한다. 생명주기는 정보의 중요도와 사용 가능한 저장 공간에 따라 동적으로 관리된다.

### 8.2.2 빠른 체험: 30초 만에 메모리 기능 시작하기

구현 세부 사항을 깊이 파고들기 전에, 메모리 시스템의 기본 기능을 빠르게 체험해보자:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import MemoryTool

# 메모리 기능이 있는 에이전트 생성
llm = HelloAgentsLLM()
agent = SimpleAgent(name="메모리 어시스턴트", llm=llm)

# 메모리 도구 생성
memory_tool = MemoryTool(user_id="user123")
tool_registry = ToolRegistry()
tool_registry.register_tool(memory_tool)
agent.tool_registry = tool_registry

# 메모리 기능 체험
print("=== 여러 메모리 추가 ===")

# 첫 번째 메모리 추가
result1 = memory_tool.execute("add", content="사용자 장삼은 머신러닝과 데이터 분석에 집중하는 Python 개발자", memory_type="semantic", importance=0.8)
print(f"메모리 1: {result1}")

# 두 번째 메모리 추가
result2 = memory_tool.execute("add", content="이사는 React와 Vue.js 개발에 능숙한 프론트엔드 엔지니어", memory_type="semantic", importance=0.7)
print(f"메모리 2: {result2}")

# 세 번째 메모리 추가
result3 = memory_tool.execute("add", content="왕오는 사용자 경험 설계와 요구 분석을 담당하는 제품 관리자", memory_type="semantic", importance=0.6)
print(f"메모리 3: {result3}")

print("\n=== 특정 메모리 검색 ===")
# 프론트엔드 관련 메모리 검색
print("🔍 '프론트엔드 엔지니어' 검색:")
result = memory_tool.execute("search", query="프론트엔드 엔지니어", limit=3)
print(result)

print("\n=== 메모리 요약 ===")
result = memory_tool.execute("summary")
print(result)
```

### 8.2.3 MemoryTool 상세 설명

이제 하향식 접근 방식을 채택하여 MemoryTool이 지원하는 특정 연산부터 시작하여 점차 기본 구현을 깊이 살펴보자. MemoryTool은 메모리 시스템의 통합 인터페이스로서, "통합 진입, 분산 처리"의 아키텍처 패턴을 따른다:

```python
def execute(self, action: str, **kwargs) -> str:
    """메모리 연산 실행

    지원하는 연산:
    - add: 메모리 추가 (4가지 유형 지원: working/episodic/semantic/perceptual)
    - search: 메모리 검색
    - summary: 메모리 요약 가져오기
    - stats: 통계 가져오기
    - update: 메모리 업데이트
    - remove: 메모리 삭제
    - forget: 메모리 망각 (다중 전략)
    - consolidate: 메모리 강화 (단기 → 장기)
    - clear_all: 모든 메모리 지우기
    """

    if action == "add":
        return self._add_memory(**kwargs)
    elif action == "search":
        return self._search_memory(**kwargs)
    elif action == "summary":
        return self._get_summary(**kwargs)
    # ... 기타 연산
```

이 통합 `execute` 인터페이스 설계는 에이전트의 호출 방식을 단순화한다. `action` 매개변수를 통해 특정 연산을 지정하고, `**kwargs`를 통해 각 연산이 서로 다른 매개변수 요구 사항을 가질 수 있다. 여기서 몇 가지 중요한 연산을 나열하겠다:

(1) 연산 1: add

`add` 연산은 메모리 시스템의 기반이다. 인간 뇌가 지각한 정보를 기억으로 부호화하는 과정을 시뮬레이션한다. 구현에서는 메모리 내용을 저장할 뿐만 아니라, 각 메모리에 풍부한 맥락 정보를 추가한다. 이 정보는 이후의 검색과 관리에서 중요한 역할을 한다.

```python
def _add_memory(
    self,
    content: str = "",
    memory_type: str = "working",
    importance: float = 0.5,
    file_path: str = None,
    modality: str = None,
    **metadata
) -> str:
    """메모리 추가"""
    try:
        # 세션 ID 존재 확인
        if self.current_session_id is None:
            self.current_session_id = f"session_{datetime.now().strftime('%Y%m%d_%H%M%S')}"

        # 지각 메모리 파일 지원
        if memory_type == "perceptual" and file_path:
            inferred = modality or self._infer_modality(file_path)
            metadata.setdefault("modality", inferred)
            metadata.setdefault("raw_data", file_path)

        # 메타데이터에 세션 정보 추가
        metadata.update({
            "session_id": self.current_session_id,
            "timestamp": datetime.now().isoformat()
        })

        memory_id = self.memory_manager.add_memory(
            content=content,
            memory_type=memory_type,
            importance=importance,
            metadata=metadata,
            auto_classify=False
        )

        return f"✅ 메모리 추가됨 (ID: {memory_id[:8]}...)"

    except Exception as e:
        return f"❌ 메모리 추가 실패: {str(e)}"
```

이는 주로 세 가지 핵심 작업을 구현한다: 세션 ID의 자동 관리(각 메모리에 명확한 세션 귀속을 보장), 멀티모달 데이터의 지능형 처리(파일 유형 자동 추론 및 관련 메타데이터 저장), 컨텍스트 정보의 자동 보완(각 메모리에 타임스탬프와 세션 정보 추가). 그 중 `importance` 매개변수(기본값 0.5)는 메모리의 중요도 수준을 표시하는 데 사용되며, 값 범위는 0.0~1.0이다. 이 메커니즘은 인간 뇌의 다양한 정보 중요도 평가를 시뮬레이션한다. 이 설계를 통해 에이전트는 서로 다른 시간대의 대화를 자동으로 구별하고, 이후의 검색과 관리를 위한 풍부한 컨텍스트 정보를 제공할 수 있다.

각 메모리 유형에 대해 서로 다른 사용 예시를 제공한다:

```python
# 1. 작업 기억 - 임시 정보, 제한된 용량
memory_tool.execute("add",
    content="사용자가 방금 Python 함수에 관한 질문을 했다",
    memory_type="working",
    importance=0.6
)

# 2. 일화 기억 - 특정 사건과 경험
memory_tool.execute("add",
    content="2024년 3월 15일, 사용자 장삼이 첫 번째 Python 프로젝트를 완료했다",
    memory_type="episodic",
    importance=0.8,
    event_type="milestone",
    location="온라인 학습 플랫폼"
)

# 3. 의미 기억 - 추상적 지식과 개념
memory_tool.execute("add",
    content="Python은 인터프리터 방식의 객체지향 프로그래밍 언어다",
    memory_type="semantic",
    importance=0.9,
    knowledge_type="factual"
)

# 4. 지각 기억 - 멀티모달 정보
memory_tool.execute("add",
    content="사용자가 함수 정의를 포함한 Python 코드 스크린샷을 업로드했다",
    memory_type="perceptual",
    importance=0.7,
    modality="image",
    file_path="./uploads/code_screenshot.png"
)
```

(2) 연산 2: search

`search` 연산은 메모리 시스템의 핵심 기능이다. 방대한 메모리 중에서 쿼리와 가장 관련성 높은 내용을 빠르게 찾아야 하며, 의미 이해, 관련성 계산, 결과 정렬 등 여러 단계가 포함된다.

```python
def _search_memory(
    self,
    query: str,
    limit: int = 5,
    memory_types: List[str] = None,
    memory_type: str = None,
    min_importance: float = 0.1
) -> str:
    """메모리 검색"""
    try:
        # 매개변수 표준화
        if memory_type and not memory_types:
            memory_types = [memory_type]

        results = self.memory_manager.retrieve_memories(
            query=query,
            limit=limit,
            memory_types=memory_types,
            min_importance=min_importance
        )

        if not results:
            return f"🔍 '{query}'과 관련된 메모리를 찾지 못했다"

        # 결과 포맷팅
        formatted_results = []
        formatted_results.append(f"🔍 관련 메모리 {len(results)}개 발견:")

        for i, memory in enumerate(results, 1):
            memory_type_label = {
                "working": "작업 기억",
                "episodic": "일화 기억",
                "semantic": "의미 기억",
                "perceptual": "지각 기억"
            }.get(memory.memory_type, memory.memory_type)

            content_preview = memory.content[:80] + "..." if len(memory.content) > 80 else memory.content
            formatted_results.append(
                f"{i}. [{memory_type_label}] {content_preview} (중요도: {memory.importance:.2f})"
            )

        return "\n".join(formatted_results)

    except Exception as e:
        return f"❌ 메모리 검색 실패: {str(e)}"
```

검색 연산은 단수형과 복수형 매개변수 형식(`memory_type`과 `memory_types`) 모두를 지원하도록 설계되어, 사용자가 가장 자연스러운 방식으로 요구를 표현할 수 있다. 그 중 `min_importance` 매개변수(기본값 0.1)는 저품질 메모리를 필터링하는 데 사용된다. 검색 함수 사용에 대해서는 다음 예시를 참고할 수 있다:

```python
# 기본 검색
result = memory_tool.execute("search", query="Python 프로그래밍", limit=5)

# 메모리 유형 지정 검색
result = memory_tool.execute("search",
    query="학습 진도",
    memory_type="episodic",
    limit=3
)

# 다중 유형 검색
result = memory_tool.execute("search",
    query="함수 정의",
    memory_types=["semantic", "episodic"],
    min_importance=0.5
)
```

(3) 연산 3: forget

망각 메커니즘은 가장 인지과학적인 기능이다. 인간 뇌의 선택적 망각 과정을 시뮬레이션하며, 세 가지 전략을 지원한다: 중요도 기반(중요하지 않은 메모리 삭제), 시간 기반(오래된 메모리 삭제), 용량 기반(저장 공간이 한계에 가까워질 때 가장 덜 중요한 메모리 삭제).

```python
def _forget(self, strategy: str = "importance_based", threshold: float = 0.1, max_age_days: int = 30) -> str:
    """메모리 망각 (다중 전략 지원)"""
    try:
        count = self.memory_manager.forget_memories(
            strategy=strategy,
            threshold=threshold,
            max_age_days=max_age_days
        )
        return f"🧹 {count}개의 메모리를 망각했다 (전략: {strategy})"
    except Exception as e:
        return f"❌ 메모리 망각 실패: {str(e)}"
```

**세 가지 망각 전략 사용:**

```python
# 1. 중요도 기반 망각 - 중요도 임계값 이하의 메모리 삭제
memory_tool.execute("forget",
    strategy="importance_based",
    threshold=0.2
)

# 2. 시간 기반 망각 - 지정된 일수보다 오래된 메모리 삭제
memory_tool.execute("forget",
    strategy="time_based",
    max_age_days=30
)

# 3. 용량 기반 망각 - 메모리 수가 한계를 초과할 때 가장 덜 중요한 것 삭제
memory_tool.execute("forget",
    strategy="capacity_based",
    threshold=0.3
)
```

(4) 연산 4: consolidate

```python
def _consolidate(self, from_type: str = "working", to_type: str = "episodic", importance_threshold: float = 0.7) -> str:
    """메모리 강화 (중요한 단기 기억을 장기 기억으로 승격)"""
    try:
        count = self.memory_manager.consolidate_memories(
            from_type=from_type,
            to_type=to_type,
            importance_threshold=importance_threshold,
        )
        return f"🔄 {count}개의 메모리를 장기 기억으로 강화했다 ({from_type} → {to_type}, 임계값={importance_threshold})"
    except Exception as e:
        return f"❌ 메모리 강화 실패: {str(e)}"
```

consolidate 연산은 신경과학의 기억 강화 개념을 활용하여, 인간 뇌가 단기 기억을 장기 기억으로 변환하는 과정을 시뮬레이션한다. 기본 설정은 중요도가 0.7을 초과하는 작업 기억을 일화 기억으로 변환하는 것이다. 이 임계값은 진정으로 중요한 정보만 장기적으로 보존되도록 보장한다. 전체 과정은 자동화되어 있으며, 사용자가 특정 메모리를 수동으로 선택할 필요 없다. 시스템이 기준을 충족하는 메모리를 지능적으로 식별하고 유형 변환을 수행한다.

**메모리 강화 사용 예시:**

```python
# 중요한 작업 기억을 일화 기억으로 변환
memory_tool.execute("consolidate",
    from_type="working",
    to_type="episodic",
    importance_threshold=0.7
)

# 중요한 일화 기억을 의미 기억으로 변환
memory_tool.execute("consolidate",
    from_type="episodic",
    to_type="semantic",
    importance_threshold=0.8
)
```

이러한 핵심 연산들의 협력을 통해 MemoryTool은 완전한 메모리 생명주기 관리 시스템을 구축한다. 메모리 생성, 검색, 요약부터 망각, 강화, 관리까지, 폐쇄 루프 지능형 메모리 관리 시스템을 형성하여 에이전트에게 진정으로 인간과 같은 메모리 기능을 부여한다.

### 8.2.4 MemoryManager 상세 설명

MemoryTool의 인터페이스 설계를 이해한 후, 이제 기본 구현을 깊이 살펴보자. MemoryTool이 MemoryManager와 어떻게 협력하는지 확인한다. 이 계층화된 설계는 소프트웨어 엔지니어링의 관심사 분리 원칙을 구현한다. MemoryTool은 사용자 인터페이스와 매개변수 처리에 집중하고, MemoryManager는 핵심 메모리 관리 로직을 담당한다.

MemoryTool은 초기화 과정에서 MemoryManager 인스턴스를 생성하고 설정에 따라 다양한 유형의 메모리 모듈을 활성화한다. 이 설계를 통해 사용자는 특정 요구에 따라 어떤 메모리 유형을 활성화할지 선택할 수 있어, 기능의 완전성을 보장하면서도 불필요한 리소스 소비를 피할 수 있다.

```python
class MemoryTool(Tool):
    """메모리 도구 - 에이전트에 메모리 기능 제공"""

    def __init__(
        self,
        user_id: str = "default_user",
        memory_config: MemoryConfig = None,
        memory_types: List[str] = None
    ):
        super().__init__(
            name="memory",
            description="메모리 도구 - 대화 기록, 지식, 경험을 저장하고 검색할 수 있다"
        )

        # 메모리 관리자 초기화
        self.memory_config = memory_config or MemoryConfig()
        self.memory_types = memory_types or ["working", "episodic", "semantic"]

        self.memory_manager = MemoryManager(
            config=self.memory_config,
            user_id=user_id,
            enable_working="working" in self.memory_types,
            enable_episodic="episodic" in self.memory_types,
            enable_semantic="semantic" in self.memory_types,
            enable_perceptual="perceptual" in self.memory_types
        )
```

MemoryManager는 메모리 시스템의 핵심 조율자로서 다양한 유형의 메모리 모듈을 관리하고 통합 연산 인터페이스를 제공한다.

```python
class MemoryManager:
    """메모리 관리자 - 통합 메모리 연산 인터페이스"""

    def __init__(
        self,
        config: Optional[MemoryConfig] = None,
        user_id: str = "default_user",
        enable_working: bool = True,
        enable_episodic: bool = True,
        enable_semantic: bool = True,
        enable_perceptual: bool = False
    ):
        self.config = config or MemoryConfig()
        self.user_id = user_id

        # 저장 및 검색 컴포넌트 초기화
        self.store = MemoryStore(self.config)
        self.retriever = MemoryRetriever(self.store, self.config)

        # 다양한 유형의 메모리 초기화
        self.memory_types = {}

        if enable_working:
            self.memory_types['working'] = WorkingMemory(self.config, self.store)

        if enable_episodic:
            self.memory_types['episodic'] = EpisodicMemory(self.config, self.store)

        if enable_semantic:
            self.memory_types['semantic'] = SemanticMemory(self.config, self.store)

        if enable_perceptual:
            self.memory_types['perceptual'] = PerceptualMemory(self.config, self.store)
```

### 8.2.5 네 가지 메모리 유형

이제 네 가지 메모리 유형의 구체적인 구현을 깊이 살펴보자. 각 메모리 유형은 고유한 특성과 응용 시나리오를 가지고 있다:

(1) 작업 기억

작업 기억은 메모리 시스템에서 가장 활성화된 부분이다. 현재 대화 세션의 임시 정보를 저장하는 역할을 한다. 작업 기억의 설계 초점은 빠른 접근과 자동 정리에 있으며, 이는 시스템의 응답 속도와 리소스 효율성을 보장한다.

작업 기억은 순수 인메모리 저장 솔루션을 채택하고, TTL(Time To Live) 메커니즘으로 자동 정리를 결합한다. 이 설계의 장점은 접근 속도가 매우 빠르다는 것이지만, 시스템 재시작 후 작업 기억의 내용이 손실됨을 의미하기도 한다. 이 특성은 작업 기억의 포지셔닝에 완벽하게 부합한다: 임시적이고 휘발성 있는 정보를 저장하는 것이다.

```python
class WorkingMemory:
    """작업 기억 구현
    특징:
    - 제한된 용량 (기본 50개) + TTL 자동 정리
    - 순수 인메모리 저장, 매우 빠른 접근
    - 하이브리드 검색: TF-IDF 벡터화 + 키워드 매칭
    """

    def __init__(self, config: MemoryConfig):
        self.max_capacity = config.working_memory_capacity or 50
        self.max_age_minutes = config.working_memory_ttl or 60
        self.memories = []

    def add(self, memory_item: MemoryItem) -> str:
        """작업 기억 추가"""
        self._expire_old_memories()  # 만료 정리

        if len(self.memories) >= self.max_capacity:
            self._remove_lowest_priority_memory()  # 용량 관리

        self.memories.append(memory_item)
        return memory_item.id

    def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
        """하이브리드 검색: TF-IDF 벡터화 + 키워드 매칭"""
        self._expire_old_memories()

        # TF-IDF 벡터 검색 시도
        vector_scores = self._try_tfidf_search(query)

        # 종합 점수 계산
        scored_memories = []
        for memory in self.memories:
            vector_score = vector_scores.get(memory.id, 0.0)
            keyword_score = self._calculate_keyword_score(query, memory.content)

            # 하이브리드 점수 계산
            base_relevance = vector_score * 0.7 + keyword_score * 0.3 if vector_score > 0 else keyword_score
            time_decay = self._calculate_time_decay(memory.timestamp)
            importance_weight = 0.8 + (memory.importance * 0.4)

            final_score = base_relevance * time_decay * importance_weight
            if final_score > 0:
                scored_memories.append((final_score, memory))

        scored_memories.sort(key=lambda x: x[0], reverse=True)
        return [memory for _, memory in scored_memories[:limit]]
```

작업 기억 검색은 하이브리드 검색 전략을 채택한다. 먼저 TF-IDF 벡터화로 의미 검색을 시도하고, 실패하면 키워드 매칭으로 폴백한다. 이 설계는 다양한 환경에서 안정적인 검색 서비스를 보장한다. 점수 알고리즘은 의미 유사성, 시간 감쇠, 중요도 가중치를 결합한다. 최종 점수 공식은 다음과 같다:

$$\text{final\_score} = (\text{similarity} \times \text{time\_decay}) \times (0.8 + \text{importance} \times 0.4)$$

(2) 일화 기억

일화 기억은 특정 사건과 경험을 저장하는 역할을 한다. 설계 초점은 사건의 완전성과 시간 순서 관계를 유지하는 것이다. 일화 기억은 SQLite + Qdrant의 하이브리드 저장 솔루션을 채택한다. SQLite는 구조화된 데이터 저장과 복잡한 쿼리를 담당하고, Qdrant는 효율적인 벡터 검색을 담당한다.

```python
class EpisodicMemory:
    """일화 기억 구현
    특징:
    - SQLite+Qdrant 하이브리드 저장 아키텍처
    - 시간 순서 및 세션 수준 검색 지원
    - 구조화된 필터링 + 의미 벡터 검색
    """

    def __init__(self, config: MemoryConfig):
        self.doc_store = SQLiteDocumentStore(config.database_path)
        self.vector_store = QdrantVectorStore(config.qdrant_url, config.qdrant_api_key)
        self.embedder = create_embedding_model_with_fallback()
        self.sessions = {}  # 세션 인덱스

    def add(self, memory_item: MemoryItem) -> str:
        """일화 기억 추가"""
        # 에피소드 객체 생성
        episode = Episode(
            episode_id=memory_item.id,
            session_id=memory_item.metadata.get("session_id", "default"),
            timestamp=memory_item.timestamp,
            content=memory_item.content,
            context=memory_item.metadata
        )

        # 세션 인덱스 업데이트
        session_id = episode.session_id
        if session_id not in self.sessions:
            self.sessions[session_id] = []
        self.sessions[session_id].append(episode.episode_id)

        # 영속성 저장 (SQLite + Qdrant)
        self._persist_episode(episode)
        return memory_item.id

    def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
        """하이브리드 검색: 구조화된 필터링 + 의미 벡터 검색"""
        # 1. 구조화된 사전 필터링 (시간 범위, 중요도 등)
        candidate_ids = self._structured_filter(**kwargs)

        # 2. 벡터 의미 검색
        hits = self._vector_search(query, limit * 5, kwargs.get("user_id"))

        # 3. 종합 점수 계산 및 정렬
        results = []
        for hit in hits:
            if self._should_include(hit, candidate_ids, kwargs):
                score = self._calculate_episode_score(hit)
                memory_item = self._create_memory_item(hit)
                results.append((score, memory_item))

        results.sort(key=lambda x: x[0], reverse=True)
        return [item for _, item in results[:limit]]

    def _calculate_episode_score(self, hit) -> float:
        """일화 기억 점수 계산 알고리즘"""
        vec_score = float(hit.get("score", 0.0))
        recency_score = self._calculate_recency(hit["metadata"]["timestamp"])
        importance = hit["metadata"].get("importance", 0.5)

        # 점수 공식: (벡터 유사성 × 0.8 + 시간 최신성 × 0.2) × 중요도 가중치
        base_relevance = vec_score * 0.8 + recency_score * 0.2
        importance_weight = 0.8 + (importance * 0.4)

        return base_relevance * importance_weight
```

일화 기억의 검색 구현은 복잡한 다중 요인 점수 메커니즘을 보여준다. 의미 유사성뿐만 아니라 시간 최신성 고려도 통합하여, 최종적으로 중요도 가중치로 조정한다. 점수 공식은 다음과 같다:

$$\text{score} = (\text{vec\_score} \times 0.8 + \text{recency\_score} \times 0.2) \times (0.8 + \text{importance} \times 0.4)$$

이는 검색 결과가 의미론적으로도, 시간적으로도 관련성이 있도록 보장한다.

(3) 의미 기억

의미 기억은 메모리 시스템에서 가장 복잡한 부분이다. 추상적 개념, 규칙, 지식을 저장하는 역할을 한다. 의미 기억의 설계 초점은 지식의 구조화된 표현과 지능형 추론 기능이다. 의미 기억은 Neo4j 그래프 데이터베이스와 Qdrant 벡터 데이터베이스의 하이브리드 아키텍처를 채택한다. 이 설계를 통해 시스템은 빠른 의미 검색과 지식 그래프를 이용한 복잡한 관계 추론을 모두 수행할 수 있다.

```python
class SemanticMemory(BaseMemory):
    """의미 기억 구현

    특징:
    - HuggingFace 중국어 사전 훈련 모델을 사용한 텍스트 임베딩
    - 빠른 유사성 매칭을 위한 벡터 검색
    - 엔티티와 관계를 위한 지식 그래프 저장
    - 하이브리드 검색 전략: 벡터 + 그래프 + 의미 추론
    """

    def __init__(self, config: MemoryConfig, storage_backend=None):
        super().__init__(config, storage_backend)

        # 임베딩 모델 (통합 제공)
        self.embedding_model = get_text_embedder()

        # 전문 데이터베이스 저장
        self.vector_store = QdrantConnectionManager.get_instance(**qdrant_config)
        self.graph_store = Neo4jGraphStore(**neo4j_config)

        # 엔티티 및 관계 캐시
        self.entities: Dict[str, Entity] = {}
        self.relations: List[Relation] = []

        # NLP 처리기 (중국어 및 영어 지원)
        self.nlp = self._init_nlp()
```

의미 기억의 추가 과정은 지식 그래프 구축의 완전한 워크플로를 구현한다. 시스템은 메모리 내용을 저장할 뿐만 아니라 자동으로 엔티티와 관계를 추출하여 구조화된 지식 표현을 구축한다:

```python
def add(self, memory_item: MemoryItem) -> str:
    """의미 기억 추가"""
    # 1. 텍스트 임베딩 생성
    embedding = self.embedding_model.encode(memory_item.content)

    # 2. 엔티티 및 관계 추출
    entities = self._extract_entities(memory_item.content)
    relations = self._extract_relations(memory_item.content, entities)

    # 3. Neo4j 그래프 데이터베이스에 저장
    for entity in entities:
        self._add_entity_to_graph(entity, memory_item)

    for relation in relations:
        self._add_relation_to_graph(relation, memory_item)

    # 4. Qdrant 벡터 데이터베이스에 저장
    metadata = {
        "memory_id": memory_item.id,
        "entities": [e.entity_id for e in entities],
        "entity_count": len(entities),
        "relation_count": len(relations)
    }

    self.vector_store.add_vectors(
        vectors=[embedding.tolist()],
        metadata=[metadata],
        ids=[memory_item.id]
    )
```

의미 기억의 검색은 벡터 검색의 의미 이해 능력과 그래프 검색의 관계 추론 능력을 결합한 하이브리드 검색 전략을 구현한다:

```python
def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
    """의미 기억 검색"""
    # 1. 벡터 검색
    vector_results = self._vector_search(query, limit * 2, user_id)

    # 2. 그래프 검색
    graph_results = self._graph_search(query, limit * 2, user_id)

    # 3. 하이브리드 순위 계산
    combined_results = self._combine_and_rank_results(
        vector_results, graph_results, query, limit
    )

    return combined_results[:limit]
```

하이브리드 순위 알고리즘은 다중 요인 점수 메커니즘을 채택한다:

```python
def _combine_and_rank_results(self, vector_results, graph_results, query, limit):
    """결과의 하이브리드 순위 계산"""
    combined = {}

    # 벡터와 그래프 검색 결과 병합
    for result in vector_results:
        combined[result["memory_id"]] = {
            **result,
            "vector_score": result.get("score", 0.0),
            "graph_score": 0.0
        }

    for result in graph_results:
        memory_id = result["memory_id"]
        if memory_id in combined:
            combined[memory_id]["graph_score"] = result.get("similarity", 0.0)
        else:
            combined[memory_id] = {
                **result,
                "vector_score": 0.0,
                "graph_score": result.get("similarity", 0.0)
            }

    # 하이브리드 점수 계산
    for memory_id, result in combined.items():
        vector_score = result["vector_score"]
        graph_score = result["graph_score"]
        importance = result.get("importance", 0.5)

        # 기본 관련성 점수
        base_relevance = vector_score * 0.7 + graph_score * 0.3

        # 중요도 가중치 [0.8, 1.2]
        importance_weight = 0.8 + (importance * 0.4)

        # 최종 점수: 유사성 * 중요도 가중치
        combined_score = base_relevance * importance_weight
        result["combined_score"] = combined_score

    # 정렬 및 반환
    sorted_results = sorted(
        combined.values(),
        key=lambda x: x["combined_score"],
        reverse=True
    )

    return sorted_results[:limit]
```

의미 기억의 점수 공식은 다음과 같다:

$$\text{combined\_score} = (\text{vector\_score} \times 0.7 + \text{graph\_score} \times 0.3) \times (0.8 + \text{importance} \times 0.4)$$

이 설계의 핵심 아이디어는 다음과 같다:

- **벡터 검색 가중치(0.7)**: 의미 유사성이 주요 요소로, 검색 결과가 쿼리와 의미론적으로 관련되도록 보장한다
- **그래프 검색 가중치(0.3)**: 관계 추론을 보완으로, 개념 간의 암묵적 연관성을 발견한다
- **중요도 가중치 범위[0.8, 1.2]**: 중요도가 유사성 순위에 과도한 영향을 미치는 것을 방지하고, 검색 정확도를 유지한다

(4) 지각 기억

지각 기억은 텍스트, 이미지, 오디오 등 다양한 모달리티의 데이터 저장과 검색을 지원한다. 모달리티별 분리 저장 전략을 채택하여, 서로 다른 모달리티의 데이터를 위해 독립적인 벡터 컬렉션을 생성한다. 이 설계는 차원 불일치 문제를 방지하면서 검색 정확도를 보장한다:

```python
class PerceptualMemory(BaseMemory):
    """지각 기억 구현

    특징:
    - 멀티모달 데이터 지원 (텍스트, 이미지, 오디오 등)
    - 교차 모달 유사성 검색
    - 지각 데이터의 의미 이해
    - 콘텐츠 생성 및 검색 지원
    """

    def __init__(self, config: MemoryConfig, storage_backend=None):
        super().__init__(config, storage_backend)

        # 멀티모달 인코더
        self.text_embedder = get_text_embedder()
        self._clip_model = self._init_clip_model()  # 이미지 인코딩
        self._clap_model = self._init_clap_model()  # 오디오 인코딩

        # 모달리티별 분리 벡터 저장
        self.vector_stores = {
            "text": QdrantConnectionManager.get_instance(
                collection_name="perceptual_text",
                vector_size=self.vector_dim
            ),
            "image": QdrantConnectionManager.get_instance(
                collection_name="perceptual_image",
                vector_size=self._image_dim
            ),
            "audio": QdrantConnectionManager.get_instance(
                collection_name="perceptual_audio",
                vector_size=self._audio_dim
            )
        }
```

지각 기억 검색은 동일 모달리티와 교차 모달리티 두 가지 모드를 지원한다. 동일 모달리티 검색은 전문화된 인코더를 사용하여 정밀 매칭을 수행하고, 교차 모달리티 검색에는 더 복잡한 의미 정렬 메커니즘이 필요하다:

```python
def retrieve(self, query: str, limit: int = 5, **kwargs) -> List[MemoryItem]:
    """지각 기억 검색 (모달리티 필터링 가능; 동일 모달리티 벡터 검색 + 시간/중요도 융합)"""
    user_id = kwargs.get("user_id")
    target_modality = kwargs.get("target_modality")
    query_modality = kwargs.get("query_modality", target_modality or "text")

    # 동일 모달리티 벡터 검색
    try:
        query_vector = self._encode_data(query, query_modality)
        store = self._get_vector_store_for_modality(target_modality or query_modality)

        where = {"memory_type": "perceptual"}
        if user_id:
            where["user_id"] = user_id
        if target_modality:
            where["modality"] = target_modality

        hits = store.search_similar(
            query_vector=query_vector,
            limit=max(limit * 5, 20),
            where=where
        )
    except Exception:
        hits = []

    # 융합 순위 계산 (벡터 유사성 + 시간 최신성 + 중요도 가중치)
    results = []
    for hit in hits:
        vector_score = float(hit.get("score", 0.0))
        recency_score = self._calculate_recency_score(hit["metadata"]["timestamp"])
        importance = hit["metadata"].get("importance", 0.5)

        # 점수 계산 알고리즘
        base_relevance = vector_score * 0.8 + recency_score * 0.2
        importance_weight = 0.8 + (importance * 0.4)
        combined_score = base_relevance * importance_weight

        results.append((combined_score, self._create_memory_item(hit)))

    results.sort(key=lambda x: x[0], reverse=True)
    return [item for _, item in results[:limit]]
```

지각 기억의 점수 공식은 다음과 같다:

$$\text{combined\_score} = (\text{vector\_score} \times 0.8 + \text{recency\_score} \times 0.2) \times (0.8 + \text{importance} \times 0.4)$$

지각 기억의 점수 메커니즘도 교차 모달 검색을 지원하며, 통합 벡터 공간을 통해 텍스트, 이미지, 오디오 등 서로 다른 모달리티 데이터의 의미 정렬을 실현한다. 교차 모달 검색을 수행할 때 시스템은 자동으로 점수 가중치를 조정하여 검색 결과의 다양성과 정확도를 보장한다. 또한 지각 기억의 시간 최신성 계산은 지수 감쇠 모델을 채택한다:

```python
def _calculate_recency_score(self, timestamp: str) -> float:
    """시간 최신성 점수 계산"""
    try:
        memory_time = datetime.fromisoformat(timestamp)
        current_time = datetime.now()
        age_hours = (current_time - memory_time).total_seconds() / 3600

        # 지수 감쇠: 24시간 이내 높은 점수 유지, 이후 점진적 감쇠
        decay_factor = 0.1  # 감쇠 계수
        recency_score = math.exp(-decay_factor * age_hours / 24)

        return max(0.1, recency_score)  # 최소 기본 점수 0.1 유지
    except Exception:
        return 0.5  # 기본 중간 점수
```

이 시간 감쇠 모델은 인간 기억의 망각 곡선을 시뮬레이션하여, 지각 기억 시스템이 시간적으로 더 관련성 있는 메모리 내용을 우선적으로 검색할 수 있도록 보장한다.

## 8.3 RAG 시스템: 지식 검색 증강

### 8.3.1 RAG 기초

HelloAgents의 RAG 시스템 구현에 깊이 들어가기 전에, 먼저 RAG 기술의 기본 개념, 발전 역사, 핵심 원리를 이해해보자. 이 텍스트는 RAG를 기반으로 만들어진 것이 아니므로, 시스템 설계에서의 기술 선택과 혁신을 더 잘 이해하기 위해 여기서는 관련 개념을 빠르게 복습하는 것에 그치겠다.

(1) RAG란 무엇인가?

검색 증강 생성(Retrieval-Augmented Generation, RAG)은 정보 검색과 텍스트 생성을 결합한 기술이다. 핵심 아이디어는: 답변을 생성하기 전에 먼저 외부 지식베이스에서 관련 정보를 검색한 다음, 검색된 정보를 컨텍스트로 대형 언어 모델에 제공하여 더 정확하고 신뢰할 수 있는 답변을 생성한다는 것이다.

따라서 검색 증강 생성은 세 단어로 분해할 수 있다. **검색(Retrieval)**은 지식베이스에서 관련 내용을 쿼리하는 것이고, **증강(Augmented)**은 검색 결과를 프롬프트에 통합하여 모델 생성을 지원하는 것이며, **생성(Generation)**은 정확성과 투명성을 결합한 답변을 출력하는 것이다.

(2) 기본 워크플로

완전한 RAG 응용 프로그램의 워크플로는 주로 두 가지 핵심 단계로 나뉜다. **데이터 준비 단계**에서 시스템은 **데이터 추출**, **텍스트 분할**, **벡터화**를 통해 외부 지식을 검색 가능한 데이터베이스로 구축한다. 이후 **응용 단계**에서 시스템은 사용자 **쿼리**에 응답하고, 데이터베이스에서 관련 정보를 **검색**하며, **프롬프트에 주입**하고, 최종적으로 대형 언어 모델을 구동하여 **답변을 생성**한다.

(3) 발전 역사

첫 번째 단계: Naive RAG (2020-2021). 이는 RAG 기술의 초기 단계로, 과정이 직접적이고 단순하며, 일반적으로 "검색-읽기" 모드라고 한다. **검색 방법**: 주로 `TF-IDF`나 `BM25` 같은 전통적인 키워드 매칭 알고리즘에 의존한다. 이 방법들은 용어 빈도와 문서 빈도를 계산하여 관련성을 평가하며, 글자 그대로의 매칭 효과는 좋지만 의미적 유사성을 이해하기 어렵다. **생성 모드**: 처리 없이 검색된 문서 내용을 프롬프트 컨텍스트에 직접 연결하여 생성 모델에 전송한다.

두 번째 단계: Advanced RAG (2022-2023). 벡터 데이터베이스와 텍스트 임베딩 기술의 성숙과 함께 RAG는 빠른 발전 단계에 진입했다. 연구자들과 개발자들은 "검색"과 "생성"의 다양한 단계에서 많은 최적화 기술을 도입했다. **검색 방법**: **밀집 임베딩** 기반 의미 검색으로 전환되었다. 텍스트를 고차원 벡터로 변환함으로써 모델은 키워드만이 아니라 의미적 유사성을 이해하고 매칭할 수 있다. **생성 모드**: 쿼리 재작성, 문서 청킹, 재순위 지정 등 많은 최적화 기술이 도입되었다.

세 번째 단계: Modular RAG (2023-현재). Advanced RAG를 기반으로 현대 RAG 시스템은 모듈화, 자동화, 지능화 방향으로 더욱 발전한다. 시스템의 각 부분이 플러그 가능하고 조합 가능한 독립적인 모듈로 설계되어, 더욱 다양하고 복잡한 응용 시나리오에 적응한다. **검색 방법**: 하이브리드 검색, 다중 쿼리 확장, 가상 문서 임베딩 등. **생성 모드**: 사고 연쇄 추론, 자기 반성 및 수정 등.

### 8.3.2 RAG 시스템 작동 원리

구현 세부 사항에 깊이 들어가기 전에, 순서도를 통해 HelloAgents RAG 시스템의 완전한 워크플로를 개략적으로 살펴볼 수 있다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-5.png" alt="RAG 시스템 핵심 원리" width="85%"/>
  <p>그림 8.5 RAG 시스템의 핵심 작동 원리</p>
</div>

그림 8.5에서 보이듯이, RAG 시스템의 두 가지 주요 작동 모드를 보여준다:
1. **데이터 처리 워크플로**: 지식 문서를 처리하고 저장한다. 여기서 `Markitdown` 도구를 채택하며, 설계 아이디어는 들어오는 모든 외부 지식 소스를 통합적으로 Markdown 형식으로 변환하여 처리하는 것이다.
2. **쿼리 및 생성 워크플로**: 쿼리를 기반으로 관련 정보를 검색하고 답변을 생성한다.

### 8.3.3 빠른 체험: 30초 만에 RAG 기능 시작하기

RAG 시스템의 기본 기능을 빠르게 체험해보자:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.tools import RAGTool

# RAG 기능이 있는 에이전트 생성
llm = HelloAgentsLLM()
agent = SimpleAgent(name="지식 어시스턴트", llm=llm)

# RAG 도구 생성
rag_tool = RAGTool(
    knowledge_base_path="./knowledge_base",
    collection_name="test_collection",
    rag_namespace="test"
)

tool_registry = ToolRegistry()
tool_registry.register_tool(rag_tool)
agent.tool_registry = tool_registry

# RAG 기능 체험
# 첫 번째 지식 추가
result1 = rag_tool.execute("add_text",
    text="Python은 1991년 귀도 반 로섬이 처음 출시한 고수준 프로그래밍 언어다. Python의 설계 철학은 코드 가독성과 간결한 문법을 강조한다.",
    document_id="python_intro")
print(f"지식 1: {result1}")

# 두 번째 지식 추가
result2 = rag_tool.execute("add_text",
    text="머신러닝은 인공지능의 한 분야로, 알고리즘을 사용하여 컴퓨터가 데이터에서 패턴을 학습할 수 있도록 한다. 지도 학습, 비지도 학습, 강화 학습 세 가지 유형이 주로 포함된다.",
    document_id="ml_basics")
print(f"지식 2: {result2}")

# 세 번째 지식 추가
result3 = rag_tool.execute("add_text",
    text="RAG(검색 증강 생성)는 정보 검색과 텍스트 생성을 결합한 AI 기술이다. 관련 지식을 검색하여 대형 언어 모델의 생성 능력을 향상시킨다.",
    document_id="rag_concept")
print(f"지식 3: {result3}")


print("\n=== 지식 검색 ===")
result = rag_tool.execute("search",
    query="Python 프로그래밍 언어의 역사",
    limit=3,
    min_score=0.1
)
print(result)

print("\n=== 지식베이스 통계 ===")
result = rag_tool.execute("stats")
print(result)
```

다음으로 HelloAgents RAG 시스템의 구체적인 구현을 깊이 살펴보겠다.

### 8.3.4 RAG 시스템 아키텍처 설계

이 섹션에서는 메모리 시스템 설명과는 다른 접근 방식을 채택한다. `Memory_tool`은 체계적인 구현인 반면, 우리 설계에서 RAG는 파이프라인으로 조직될 수 있는 도구로 정의되기 때문이다. RAG 시스템의 핵심 아키텍처는 "5계층 7단계" 설계 패턴으로 요약할 수 있다:

```
사용자 계층: RAGTool 통합 인터페이스
  ↓
응용 계층: 지능형 Q&A, 검색, 관리
  ↓
처리 계층: 문서 파싱, 청킹, 벡터화
  ↓
저장 계층: 벡터 데이터베이스, 문서 저장
  ↓
기반 계층: 임베딩 모델, LLM, 데이터베이스
```

이 계층화된 설계의 장점은 전체 시스템의 안정성을 유지하면서 각 계층을 독립적으로 최적화하고 교체할 수 있다는 것이다. 예를 들어, 임베딩 모델을 sentence-transformers에서 바이리안 API로 쉽게 전환할 수 있으며, 상위 수준의 비즈니스 로직에는 영향을 미치지 않는다. 마찬가지로 처리 워크플로 코드는 완전히 재사용 가능하며, 필요한 부분을 선택하여 자신의 프로젝트에 넣을 수도 있다. RAGTool은 RAG 시스템의 통합 진입점으로서 간결한 API 인터페이스를 제공한다.

```python
class RAGTool(Tool):
    """RAG 도구

    완전한 RAG 기능 제공:
    - 다중 포맷 문서 추가 (PDF, Office, 이미지, 오디오 등)
    - 지능형 검색 및 리콜
    - LLM 강화 Q&A
    - 지식베이스 관리
    """

    def __init__(
        self,
        knowledge_base_path: str = "./knowledge_base",
        qdrant_url: str = None,
        qdrant_api_key: str = None,
        collection_name: str = "rag_knowledge_base",
        rag_namespace: str = "default"
    ):
        # RAG 파이프라인 초기화
        self._pipelines: Dict[str, Dict[str, Any]] = {}
        self.llm = HelloAgentsLLM()

        # 기본 파이프라인 생성
        default_pipeline = create_rag_pipeline(
            qdrant_url=self.qdrant_url,
            qdrant_api_key=self.qdrant_api_key,
            collection_name=self.collection_name,
            rag_namespace=self.rag_namespace
        )
        self._pipelines[self.rag_namespace] = default_pipeline
```

전체 처리 워크플로는 다음과 같다:
```
모든 포맷 문서 → MarkItDown 변환 → Markdown 텍스트 → 지능형 청킹 → 벡터화 → 저장 및 검색
```

(1) 멀티모달 문서 로딩

RAG 시스템의 핵심 장점 중 하나는 강력한 멀티모달 문서 처리 능력이다. 시스템은 MarkItDown을 통합 문서 변환 엔진으로 사용하며, 거의 모든 일반적인 문서 포맷을 지원한다. MarkItDown은 마이크로소프트의 오픈소스 범용 문서 변환 도구다. HelloAgents RAG 시스템의 핵심 컴포넌트로서, 모든 포맷의 문서를 구조화된 Markdown 텍스트로 통합 변환하는 역할을 담당한다. 입력이 PDF, Word, Excel, 이미지, 또는 오디오든 관계없이, 최종적으로 표준 Markdown 형식으로 변환되어 통합 청킹, 벡터화, 저장 워크플로에 진입한다.

```python
def _convert_to_markdown(path: str) -> str:
    """
    MarkItDown을 사용한 범용 문서 리더, 강화된 PDF 처리 포함.
    핵심 기능: 모든 포맷의 문서를 Markdown 텍스트로 변환

    지원 포맷:
    - 문서: PDF, Word, Excel, PowerPoint
    - 이미지: JPG, PNG, GIF (OCR을 통해)
    - 오디오: MP3, WAV, M4A (전사를 통해)
    - 텍스트: TXT, CSV, JSON, XML, HTML
    - 코드: Python, JavaScript, Java 등
    """
    if not os.path.exists(path):
        return ""

    # PDF 파일의 경우 강화된 처리 사용
    ext = (os.path.splitext(path)[1] or '').lower()
    if ext == '.pdf':
        return _enhanced_pdf_processing(path)

    # 다른 포맷의 경우 MarkItDown 통합 변환 사용
    md_instance = _get_markitdown_instance()
    if md_instance is None:
        return _fallback_text_reader(path)

    try:
        result = md_instance.convert(path)
        markdown_text = getattr(result, "text_content", None)
        if isinstance(markdown_text, str) and markdown_text.strip():
            print(f"[RAG] MarkItDown 변환 성공: {path} -> {len(markdown_text)} 문자 Markdown")
            return markdown_text
        return ""
    except Exception as e:
        print(f"[WARNING] MarkItDown 변환 실패 {path}: {e}")
        return _fallback_text_reader(path)
```

(2) 지능형 청킹 전략

MarkItDown 변환 후 모든 문서가 표준 Markdown 형식으로 통합된다. 이는 이후의 지능형 청킹을 위한 구조화된 기반을 제공한다. HelloAgents는 Markdown 형식에 특화된 지능형 청킹 전략을 구현하여, Markdown의 구조적 특성을 완전히 활용하여 정밀한 분할을 수행한다.

Markdown 구조 인식 청킹 워크플로:

```
표준 Markdown 텍스트 → 제목 계층 파싱 → 단락 의미 분할 → 토큰 계산 청킹 → 오버랩 전략 최적화 → 벡터화 준비
       ↓                ↓              ↓            ↓           ↓            ↓
   통합 포맷      #/##/###      의미 경계  크기 제어  정보 연속성  임베딩 벡터
   명확한 구조    계층 인식  완전성 보장  검색 최적화  컨텍스트 보존  유사성 매칭
```

모든 문서가 Markdown 형식으로 변환되었으므로, 시스템은 Markdown의 제목 구조(#, ##, ###, 등)를 사용하여 정밀한 의미 분할을 수행할 수 있다:

```python
def _split_paragraphs_with_headings(text: str) -> List[Dict]:
    """제목 계층에 기반한 단락 분할, 의미 완전성 유지"""
    lines = text.splitlines()
    heading_stack: List[str] = []
    paragraphs: List[Dict] = []
    buf: List[str] = []
    char_pos = 0

    def flush_buf(end_pos: int):
        if not buf:
            return
        content = "\n".join(buf).strip()
        if not content:
            return
        paragraphs.append({
            "content": content,
            "heading_path": " > ".join(heading_stack) if heading_stack else None,
            "start": max(0, end_pos - len(content)),
            "end": end_pos,
        })

    for ln in lines:
        raw = ln
        if raw.strip().startswith("#"):
            # 제목 줄 처리
            flush_buf(char_pos)
            level = len(raw) - len(raw.lstrip('#'))
            title = raw.lstrip('#').strip()

            if level <= 0:
                level = 1
            if level <= len(heading_stack):
                heading_stack = heading_stack[:level-1]
            heading_stack.append(title)

            char_pos += len(raw) + 1
            continue

        # 단락 내용 누적
        if raw.strip() == "":
            flush_buf(char_pos)
            buf = []
        else:
            buf.append(raw)
        char_pos += len(raw) + 1

    flush_buf(char_pos)

    if not paragraphs:
        paragraphs = [{"content": text, "heading_path": None, "start": 0, "end": len(text)}]

    return paragraphs
```

Markdown 단락 분할을 기반으로 시스템은 토큰 수에 기반한 지능형 청킹을 추가로 수행한다. 입력이 이미 구조화된 Markdown 텍스트이므로, 시스템은 청크 경계를 더욱 정밀하게 제어할 수 있어 각 청크가 벡터화 처리에 적합하면서도 Markdown 구조의 완전성을 유지한다:

```python
def _chunk_paragraphs(paragraphs: List[Dict], chunk_tokens: int, overlap_tokens: int) -> List[Dict]:
    """토큰 수에 기반한 지능형 청킹"""
    chunks: List[Dict] = []
    cur: List[Dict] = []
    cur_tokens = 0
    i = 0

    while i < len(paragraphs):
        p = paragraphs[i]
        p_tokens = _approx_token_len(p["content"]) or 1

        if cur_tokens + p_tokens <= chunk_tokens or not cur:
            cur.append(p)
            cur_tokens += p_tokens
            i += 1
        else:
            # 현재 청크 생성
            content = "\n\n".join(x["content"] for x in cur)
            start = cur[0]["start"]
            end = cur[-1]["end"]
            heading_path = next((x["heading_path"] for x in reversed(cur) if x.get("heading_path")), None)

            chunks.append({
                "content": content,
                "start": start,
                "end": end,
                "heading_path": heading_path,
            })

            # 오버랩 섹션 구성
            if overlap_tokens > 0 and cur:
                kept: List[Dict] = []
                kept_tokens = 0
                for x in reversed(cur):
                    t = _approx_token_len(x["content"]) or 1
                    if kept_tokens + t > overlap_tokens:
                        break
                    kept.append(x)
                    kept_tokens += t
                cur = list(reversed(kept))
                cur_tokens = kept_tokens
            else:
                cur = []
                cur_tokens = 0

    # 마지막 청크 처리
    if cur:
        content = "\n\n".join(x["content"] for x in cur)
        start = cur[0]["start"]
        end = cur[-1]["end"]
        heading_path = next((x["heading_path"] for x in reversed(cur) if x.get("heading_path")), None)

        chunks.append({
            "content": content,
            "start": start,
            "end": end,
            "heading_path": heading_path,
        })

    return chunks
```

동시에 서로 다른 언어와의 호환성을 위해, 시스템은 중국어-영어 혼합 텍스트를 위한 토큰 추정 알고리즘을 구현했다. 이는 청크 크기를 정확하게 제어하는 데 매우 중요하다:

```python
def _approx_token_len(text: str) -> int:
    """근사 토큰 길이 추정, 중국어-영어 혼합 텍스트 지원"""
    # CJK 문자는 각 1 토큰으로 계산
    cjk = sum(1 for ch in text if _is_cjk(ch))
    # 다른 문자는 공백 토큰화로 계산
    non_cjk_tokens = len([t for t in text.split() if t])
    return cjk + non_cjk_tokens

def _is_cjk(ch: str) -> bool:
    """CJK 문자인지 판단"""
    code = ord(ch)
    return (
        0x4E00 <= code <= 0x9FFF or  # CJK 통합 한자
        0x3400 <= code <= 0x4DBF or  # CJK 확장 A
        0x20000 <= code <= 0x2A6DF or # CJK 확장 B
        0x2A700 <= code <= 0x2B73F or # CJK 확장 C
        0x2B740 <= code <= 0x2B81F or # CJK 확장 D
        0x2B820 <= code <= 0x2CEAF or # CJK 확장 E
        0xF900 <= code <= 0xFAFF      # CJK 호환 한자
    )
```

(3) 통합 임베딩 및 벡터 저장

임베딩 모델은 RAG 시스템의 핵심이다. 텍스트를 고차원 벡터로 변환하여 컴퓨터가 텍스트의 의미적 유사성을 이해하고 비교할 수 있게 한다. RAG 시스템의 검색 능력은 임베딩 모델의 품질과 벡터 저장의 효율성에 크게 의존한다. HelloAgents는 통합 임베딩 인터페이스를 구현한다. 데모 목적으로 여기서는 바이리안 API를 사용한다. 아직 설정되지 않은 경우, 로컬 `all-MiniLM-L6-v2` 모델로 전환할 수 있다. 두 솔루션 모두 지원되지 않는 경우, TF-IDF 알고리즘도 폴백으로 구성되어 있다. 실제 사용에서는 원하는 모델이나 API로 교체하거나, 프레임워크 내용을 확장해볼 수도 있다.

```python
def index_chunks(
    store = None,
    chunks: List[Dict] = None,
    cache_db: Optional[str] = None,
    batch_size: int = 64,
    rag_namespace: str = "default"
) -> None:
    """
    통합 임베딩과 Qdrant 저장으로 Markdown 청크를 인덱싱한다.
    바이리안 API와 sentence-transformers로의 폴백을 사용한다.
    """
    if not chunks:
        print("[RAG] 인덱싱할 청크가 없다")
        return

    # 통합 임베딩 모델 사용
    embedder = get_text_embedder()
    dimension = get_dimension(384)

    # 기본 Qdrant 저장 생성
    if store is None:
        store = _create_default_vector_store(dimension)
        print(f"[RAG] 차원 {dimension}으로 기본 Qdrant 저장소 생성됨")

    # 더 나은 임베딩 품질을 위해 Markdown 텍스트 전처리
    processed_texts = []
    for c in chunks:
        raw_content = c["content"]
        processed_content = _preprocess_markdown_for_embedding(raw_content)
        processed_texts.append(processed_content)

    print(f"[RAG] 임베딩 시작: total_texts={len(processed_texts)} batch_size={batch_size}")

    # 배치 인코딩
    vecs: List[List[float]] = []
    for i in range(0, len(processed_texts), batch_size):
        part = processed_texts[i:i+batch_size]
        try:
            # 통합 임베더 사용 (내부적으로 캐싱 처리)
            part_vecs = embedder.encode(part)

            # List[List[float]] 형식으로 표준화
            if not isinstance(part_vecs, list):
                if hasattr(part_vecs, "tolist"):
                    part_vecs = [part_vecs.tolist()]
                else:
                    part_vecs = [list(part_vecs)]

            # 벡터 형식과 차원 처리
            for v in part_vecs:
                try:
                    if hasattr(v, "tolist"):
                        v = v.tolist()
                    v_norm = [float(x) for x in v]

                    # 차원 확인 및 조정
                    if len(v_norm) != dimension:
                        print(f"[WARNING] 벡터 차원 이상: 예상 {dimension}, 실제 {len(v_norm)}")
                        if len(v_norm) < dimension:
                            v_norm.extend([0.0] * (dimension - len(v_norm)))
                        else:
                            v_norm = v_norm[:dimension]

                    vecs.append(v_norm)
                except Exception as e:
                    print(f"[WARNING] 벡터 변환 실패: {e}, 제로 벡터 사용")
                    vecs.append([0.0] * dimension)

        except Exception as e:
            print(f"[WARNING] 배치 {i} 인코딩 실패: {e}")
            # 재시도 메커니즘 구현
            # ... 재시도 로직 ...

        print(f"[RAG] 임베딩 진행률: {min(i+batch_size, len(processed_texts))}/{len(processed_texts)}")
```

### 8.3.5 고급 검색 전략

RAG 시스템의 검색 능력은 핵심 경쟁력이다. 실제 응용에서 사용자 쿼리와 문서의 실제 내용 사이에 표현 방식의 차이가 있어, 관련 문서가 검색되지 않을 수 있다. 이 문제를 해결하기 위해 HelloAgents는 세 가지 상호 보완적인 고급 검색 전략을 구현한다: 다중 쿼리 확장(MQE), 가상 문서 임베딩(HyDE), 통합 확장 검색 프레임워크.

(1) 다중 쿼리 확장(MQE)

다중 쿼리 확장(Multi-Query Expansion, MQE)은 의미적으로 동등한 다양한 쿼리를 생성하여 검색 리콜을 향상시키는 기술이다. 이 방법의 핵심 통찰은: 동일한 질문이 여러 가지 다른 표현 방식을 가질 수 있으며, 서로 다른 표현이 서로 다른 관련 문서와 매칭될 수 있다는 것이다. 예를 들어 "Python을 어떻게 배우나"는 "Python 입문 튜토리얼", "Python 학습 방법", "Python 프로그래밍 가이드" 등의 쿼리로 확장될 수 있다. 이 확장된 쿼리들을 병렬로 실행하고 결과를 병합함으로써, 시스템은 더 넓은 범위의 관련 문서를 커버할 수 있어 표현 방식의 차이로 인한 중요한 정보 누락을 방지한다.

MQE의 장점은 사용자 쿼리의 여러 가능한 의미를 자동으로 이해할 수 있다는 것으로, 모호한 쿼리나 전문 용어 쿼리에 특히 효과적이다. 시스템은 LLM을 사용하여 확장 쿼리를 생성하여 확장의 다양성과 의미적 관련성을 보장한다:

```python
def _prompt_mqe(query: str, n: int) -> List[str]:
    """LLM을 사용하여 다양한 쿼리 확장 생성"""
    try:
        from ...core.llm import HelloAgentsLLM
        llm = HelloAgentsLLM()
        prompt = [
            {"role": "system", "content": "당신은 검색 쿼리 확장 어시스턴트입니다. 의미적으로 동등하거나 보완적인 다양한 쿼리를 생성하세요. 중국어를 사용하고, 간결하게 유지하며, 구두점을 피하세요."},
            {"role": "user", "content": f"원래 쿼리: {query}\n{n}개의 서로 다르게 표현된 쿼리를 한 줄에 하나씩 제공해주세요."}
        ]
        text = llm.invoke(prompt)
        lines = [ln.strip("- \t") for ln in (text or "").splitlines()]
        outs = [ln for ln in lines if ln]
        return outs[:n] or [query]
    except Exception:
        return [query]
```

(2) 가상 문서 임베딩(HyDE)

가상 문서 임베딩(Hypothetical Document Embeddings, HyDE)은 혁신적인 검색 기술이다. 핵심 아이디어는 "답변으로 답변 찾기"다. 전통적인 검색 방법은 질문을 사용하여 문서를 매칭하지만, 의미 공간에서 질문과 답변의 분포 사이에는 종종 차이가 있다. 질문은 보통 의문문이고, 문서 내용은 서술문이다. HyDE는 LLM이 먼저 가상 답변 단락을 생성하게 한 다음, 이 답변 단락을 사용하여 실제 문서를 검색함으로써 쿼리와 문서 사이의 의미 격차를 줄인다.

이 방법의 장점은 가상 답변이 의미 공간에서 실제 답변에 더 가깝기 때문에 관련 문서를 더 정확하게 매칭할 수 있다는 것이다. 가상 답변의 내용이 완전히 정확하지 않더라도, 포함된 핵심 용어, 개념, 표현 스타일은 검색 시스템이 올바른 문서를 찾도록 효과적으로 안내할 수 있다. 전문 도메인 쿼리의 경우 HyDE는 도메인 용어를 포함하는 가상 문서를 생성하여 검색 정확도를 크게 향상시킬 수 있다:

```python
def _prompt_hyde(query: str) -> Optional[str]:
    """검색 개선을 위한 가상 문서 생성"""
    try:
        from ...core.llm import HelloAgentsLLM
        llm = HelloAgentsLLM()
        prompt = [
            {"role": "system", "content": "사용자의 질문을 기반으로 벡터 검색에서 쿼리 문서로 사용될 가능한 답변 단락을 먼저 작성하세요 (분석 과정 없음)."},
            {"role": "user", "content": f"질문: {query}\n핵심 용어를 포함하는 중간 길이의 객관적인 단락을 직접 작성해주세요."}
        ]
        return llm.invoke(prompt)
    except Exception:
        return None
```

(3) 확장 검색 프레임워크

HelloAgents는 MQE와 HyDE 두 전략을 통합 확장 검색 프레임워크로 통합한다. 시스템은 `enable_mqe`와 `enable_hyde` 매개변수를 통해 사용자가 특정 시나리오에 따라 어떤 전략을 활성화할지 선택할 수 있도록 한다: 높은 리콜이 필요한 시나리오에는 두 전략을 동시에 활성화할 수 있고, 성능에 민감한 시나리오에는 기본 검색만 사용할 수 있다.

확장 검색의 핵심 메커니즘은 3단계 "확장-검색-병합" 워크플로다. 먼저 시스템은 원래 쿼리를 기반으로 여러 확장 쿼리를 생성한다(MQE가 생성한 다양한 쿼리와 HyDE가 생성한 가상 문서 포함). 그런 다음 각 확장 쿼리에 대해 벡터 검색을 병렬로 실행하여 후보 문서 풀을 얻는다. 마지막으로 중복 제거와 점수 정렬을 통해 모든 결과를 병합하여 가장 관련성 높은 상위-k 문서를 반환한다. 이 설계의 교묘함은 `candidate_pool_multiplier` 매개변수(기본값 4)를 통해 후보 풀을 확장하여 충분한 후보 문서를 선별할 수 있도록 보장하는 동시에, 지능형 중복 제거를 통해 중복 내용 반환을 방지한다는 것이다.

```python
def search_vectors_expanded(
    store = None,
    query: str = "",
    top_k: int = 8,
    rag_namespace: Optional[str] = None,
    only_rag_data: bool = True,
    score_threshold: Optional[float] = None,
    enable_mqe: bool = False,
    mqe_expansions: int = 2,
    enable_hyde: bool = False,
    candidate_pool_multiplier: int = 4,
) -> List[Dict]:
    """
    통합 임베딩과 Qdrant를 사용한 쿼리 확장 검색.
    """
    if not query:
        return []

    # 기본 저장소 생성
    if store is None:
        store = _create_default_vector_store()

    # 쿼리 확장
    expansions: List[str] = [query]

    if enable_mqe and mqe_expansions > 0:
        expansions.extend(_prompt_mqe(query, mqe_expansions))
    if enable_hyde:
        hyde_text = _prompt_hyde(query)
        if hyde_text:
            expansions.append(hyde_text)

    # 중복 제거 및 정리
    uniq: List[str] = []
    for e in expansions:
        if e and e not in uniq:
            uniq.append(e)
    expansions = uniq[: max(1, len(uniq))]

    # 후보 풀 할당
    pool = max(top_k * candidate_pool_multiplier, 20)
    per = max(1, pool // max(1, len(expansions)))

    # RAG 데이터 필터 구성
    where = {"memory_type": "rag_chunk"}
    if only_rag_data:
        where["is_rag_data"] = True
        where["data_source"] = "rag_pipeline"
    if rag_namespace:
        where["rag_namespace"] = rag_namespace

    # 모든 확장 쿼리의 결과 수집
    agg: Dict[str, Dict] = {}
    for q in expansions:
        qv = embed_query(q)
        hits = store.search_similar(
            query_vector=qv,
            limit=per,
            score_threshold=score_threshold,
            where=where
        )
        for h in hits:
            mid = h.get("metadata", {}).get("memory_id", h.get("id"))
            s = float(h.get("score", 0.0))
            if mid not in agg or s > float(agg[mid].get("score", 0.0)):
                agg[mid] = h

    # 점수로 정렬하고 반환
    merged = list(agg.values())
    merged.sort(key=lambda x: float(x.get("score", 0.0)), reverse=True)
    return merged[:top_k]
```

실제 응용에서 이 세 가지 전략의 결합 사용이 가장 효과적이다. MQE는 표현 방식의 다양성 문제를 잘 처리하고, HyDE는 의미 격차 문제를 잘 처리하며, 통합 프레임워크는 결과의 품질과 다양성을 보장한다. 일반적인 쿼리의 경우 MQE를 활성화하는 것이 권장되고, 전문 도메인 쿼리의 경우 MQE와 HyDE를 동시에 활성화하는 것이 권장되며, 성능에 민감한 시나리오의 경우 기본 검색만 사용하거나 MQE만 사용할 수 있다.

물론 다른 흥미로운 방법들도 많다. 이는 적절한 확장 소개일 뿐이다. 실제 사용 시나리오에서는 문제에 적합한 솔루션을 찾기 위해 시도해봐야 한다.

## 8.4 지능형 문서 Q&A 어시스턴트 구축

이전 섹션에서 HelloAgents의 메모리 시스템과 RAG 시스템의 설계 및 구현을 자세히 설명했다. 이제 완전한 실제 사례를 통해 이 두 시스템을 유기적으로 결합하여 지능형 문서 Q&A 어시스턴트를 구축하는 방법을 보여주겠다.

### 8.4.1 사례 배경 및 목표

실제 업무에서 우리는 종종 많은 기술 문서, 연구 논문, 제품 매뉴얼 및 기타 PDF 파일을 처리해야 한다. 전통적인 문서 읽기 방법은 비효율적이어서, 핵심 정보를 빠르게 찾기 어렵고, 지식 간의 연관성을 구축하기는 더욱 어렵다.

이 사례에서는 Datawhale의 또 다른 실습형 대형 모델 튜토리얼 Happy-LLM의 공개 베타 PDF 문서 `Happy-LLM-0727.pdf`를 예시로 사용하여 **Gradio 기반 Web 애플리케이션**을 구축하고, RAGTool과 MemoryTool을 사용하여 완전한 대화형 학습 어시스턴트를 구축하는 방법을 보여준다. PDF는 이 [링크](https://github.com/datawhalechina/happy-llm/releases/download/v1.0.1/Happy-LLM-0727.pdf)에서 얻을 수 있다.

다음 기능을 구현하고자 한다:

1. **지능형 문서 처리**: MarkItDown을 사용하여 PDF에서 Markdown으로의 통합 변환, Markdown 구조 기반 지능형 청킹 전략, 효율적인 벡터화 및 인덱스 구축

2. **고급 검색 Q&A**: 리콜 향상을 위한 다중 쿼리 확장(MQE), 검색 정확도 향상을 위한 가상 문서 임베딩(HyDE), 맥락 인식 지능형 Q&A

3. **다중 수준 메모리 관리**: 작업 기억이 현재 학습 작업과 맥락을 관리, 일화 기억이 학습 사건과 쿼리 기록을 기록, 의미 기억이 개념적 지식과 이해를 저장, 지각 기억이 문서 특징과 멀티모달 정보를 처리

4. **개인화 학습 지원**: 학습 기록에 기반한 개인화 추천, 메모리 강화 및 선택적 망각, 학습 보고서 생성 및 진도 추적

전체 시스템의 워크플로를 더 명확하게 보여주기 위해, 그림 8.6은 다섯 단계 간의 관계와 데이터 흐름을 보여준다. 다섯 단계는 완전한 폐쇄 루프를 형성한다: 1단계는 처리된 PDF 문서의 정보를 메모리 시스템에 기록하고, 2단계의 검색 결과도 메모리 시스템에 기록되며, 3단계는 메모리 시스템의 완전한 기능(추가, 검색, 강화, 망각)을 보여주고, 4단계는 RAG와 Memory를 통합하여 지능형 라우팅을 제공하며, 5단계는 모든 통계 정보를 수집하여 학습 보고서를 생성한다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-6.png" alt="" width="85%"/>
  <p>그림 8.6 지능형 Q&A 어시스턴트의 5단계 실행 워크플로</p>
</div>

다음으로 이 Web 애플리케이션을 구현하는 방법을 보여주겠다. 전체 애플리케이션은 세 가지 핵심 부분으로 나뉜다:

1. **핵심 어시스턴트 클래스(PDFLearningAssistant)**: RAGTool과 MemoryTool의 호출 로직을 캡슐화
2. **Gradio Web 인터페이스**: 친절한 사용자 상호작용 인터페이스를 제공하며, 이 부분은 예제 코드를 참조하여 학습할 수 있다
3. **기타 핵심 기능**: 노트 기록, 학습 복습, 통계 보기, 보고서 생성

### 8.4.2 핵심 어시스턴트 클래스 구현

먼저 RAGTool과 MemoryTool의 호출 로직을 캡슐화하는 핵심 어시스턴트 클래스 `PDFLearningAssistant`를 구현한다.

(1) 클래스 초기화

```python
class PDFLearningAssistant:
    """지능형 문서 Q&A 어시스턴트"""

    def __init__(self, user_id: str = "default_user"):
        """학습 어시스턴트 초기화

        Args:
            user_id: 사용자 ID, 서로 다른 사용자의 데이터 격리에 사용
        """
        self.user_id = user_id
        self.session_id = f"session_{datetime.now().strftime('%Y%m%d_%H%M%S')}"

        # 도구 초기화
        self.memory_tool = MemoryTool(user_id=user_id)
        self.rag_tool = RAGTool(rag_namespace=f"pdf_{user_id}")

        # 학습 통계
        self.stats = {
            "session_start": datetime.now(),
            "documents_loaded": 0,
            "questions_asked": 0,
            "concepts_learned": 0
        }

        # 현재 로드된 문서
        self.current_document = None
```

이 초기화 과정에서 몇 가지 핵심 설계 결정을 했다:

**MemoryTool 초기화**: `user_id` 매개변수를 통해 사용자 수준의 메모리 격리를 구현한다. 서로 다른 사용자의 학습 메모리는 완전히 독립적이며, 각 사용자는 자신만의 작업 기억, 일화 기억, 의미 기억, 지각 기억 공간을 갖는다.

**RAGTool 초기화**: `rag_namespace` 매개변수를 통해 지식베이스 네임스페이스 격리를 구현한다. `f"pdf_{user_id}"`를 네임스페이스로 사용하여, 각 사용자는 자신만의 독립적인 PDF 지식베이스를 갖는다.

**세션 관리**: `session_id`는 단일 학습 세션의 전체 과정을 추적하는 데 사용되어, 이후의 학습 여정 복습과 분석을 용이하게 한다.

**통계 정보**: `stats` 딕셔너리는 학습 보고서 생성을 위한 핵심 학습 지표를 기록한다.

(2) PDF 문서 로딩

```python
def load_document(self, pdf_path: str) -> Dict[str, Any]:
    """PDF 문서를 지식베이스에 로드

    Args:
        pdf_path: PDF 파일 경로

    Returns:
        Dict: 성공과 메시지를 포함하는 결과
    """
    if not os.path.exists(pdf_path):
        return {"success": False, "message": f"파일이 존재하지 않는다: {pdf_path}"}

    start_time = time.time()

    # [RAGTool] PDF 처리: MarkItDown 변환 → 지능형 청킹 → 벡터화
    result = self.rag_tool.execute(
        "add_document",
        file_path=pdf_path,
        chunk_size=1000,
        chunk_overlap=200
    )

    process_time = time.time() - start_time

    if result.get("success", False):
        self.current_document = os.path.basename(pdf_path)
        self.stats["documents_loaded"] += 1

        # [MemoryTool] 학습 메모리에 기록
        self.memory_tool.execute(
            "add",
            content=f"문서 《{self.current_document}》 로드됨",
            memory_type="episodic",
            importance=0.9,
            event_type="document_loaded",
            session_id=self.session_id
        )

        return {
            "success": True,
            "message": f"로딩 성공! (시간: {process_time:.1f}s)",
            "document": self.current_document
        }
    else:
        return {
            "success": False,
            "message": f"로딩 실패: {result.get('error', '알 수 없는 오류')}"
        }
```

단 한 줄의 코드로 PDF 처리를 완료할 수 있다:

```python
result = self.rag_tool.execute(
    "add_document",
    file_path=pdf_path,
    chunk_size=1000,
    chunk_overlap=200
)
```

이 호출은 RAGTool의 완전한 처리 워크플로(MarkItDown 변환, 강화 처리, 지능형 청킹, 벡터화 저장)를 트리거한다. 이 내부 세부 사항들은 섹션 8.3에서 자세히 소개되었다. 우리가 집중해야 할 것은:

- **연산 유형**: `"add_document"` - 지식베이스에 문서 추가
- **파일 경로**: `file_path` - PDF 파일 경로
- **청킹 매개변수**: `chunk_size=1000, chunk_overlap=200` - 텍스트 청킹 제어
- **반환 결과**: 처리 상태와 통계 정보를 포함하는 딕셔너리

문서가 성공적으로 로드된 후, MemoryTool을 사용하여 일화 기억에 기록한다:

```python
self.memory_tool.execute(
    "add",
    content=f"문서 《{self.current_document}》 로드됨",
    memory_type="episodic",
    importance=0.9,
    event_type="document_loaded",
    session_id=self.session_id
)
```

**왜 일화 기억을 사용하는가?** 이것이 타임스탬프가 있는 특정한 사건이기 때문에, 일화 기억으로 기록하는 것이 적합하다. `session_id` 매개변수는 이 사건을 현재 학습 세션과 연관시켜, 이후의 학습 여정 복습을 용이하게 한다.

이 메모리 기록은 이후 개인화 서비스를 위한 기반을 마련한다:

- 사용자가 "이전에 어떤 문서를 로드했나?"라고 물으면 → 일화 기억에서 검색
- 시스템이 사용자의 학습 여정과 문서 사용을 추적할 수 있다

### 8.4.3 지능형 Q&A 기능

문서가 로드된 후, 사용자는 문서에 대해 질문할 수 있다. 사용자 질문을 처리하는 `ask` 메서드를 구현한다:

```python
def ask(self, question: str, use_advanced_search: bool = True) -> str:
    """문서에 대한 질문

    Args:
        question: 사용자 질문
        use_advanced_search: 고급 검색 사용 여부 (MQE + HyDE)

    Returns:
        str: 답변
    """
    if not self.current_document:
        return "⚠️ 먼저 문서를 로드해주세요!"

    # [MemoryTool] 질문을 작업 기억에 기록
    self.memory_tool.execute(
        "add",
        content=f"질문: {question}",
        memory_type="working",
        importance=0.6,
        session_id=self.session_id
    )

    # [RAGTool] 고급 검색을 사용하여 답변 획득
    answer = self.rag_tool.execute(
        "ask",
        question=question,
        limit=5,
        enable_advanced_search=use_advanced_search,
        enable_mqe=use_advanced_search,
        enable_hyde=use_advanced_search
    )

    # [MemoryTool] 일화 기억에 기록
    self.memory_tool.execute(
        "add",
        content=f"'{question}'에 대해 학습함",
        memory_type="episodic",
        importance=0.7,
        event_type="qa_interaction",
        session_id=self.session_id
    )

    self.stats["questions_asked"] += 1

    return answer
```

`self.rag_tool.execute("ask", ...)`를 호출할 때, RAGTool은 내부적으로 다음과 같은 고급 검색 워크플로를 실행한다:

1. **다중 쿼리 확장(MQE)**:

   ```python
   # 다양한 쿼리 생성
   expanded_queries = self._generate_multi_queries(question)
   # 예를 들어 "대형 언어 모델이란?"에 대해 다음과 같이 생성될 수 있다:
   # - "대형 언어 모델의 정의는 무엇인가?"
   # - "대형 언어 모델을 설명해주세요"
   # - "LLM이란 무엇인가?"
   ```

   MQE는 LLM을 통해 의미적으로 동등하지만 다르게 표현된 쿼리를 생성하여, 여러 각도에서 사용자 의도를 이해하고 리콜을 30%~50% 향상시킨다.

2. **가상 문서 임베딩(HyDE)**:

   - 가상 답변 문서를 생성하여 쿼리와 문서 사이의 의미 격차를 줄인다
   - 가상 답변의 벡터를 사용하여 검색한다

이 고급 검색 기술의 내부 구현은 섹션 8.3.5에서 자세히 소개되었다.

### 8.4.4 기타 핵심 기능

문서 로딩과 지능형 Q&A 외에도, 노트 기록, 학습 복습, 통계 보기, 보고서 생성 등의 기능도 구현해야 한다:

```python
def add_note(self, content: str, concept: Optional[str] = None):
    """학습 노트 추가"""
    self.memory_tool.execute(
        "add",
        content=content,
        memory_type="semantic",
        importance=0.8,
        concept=concept or "general",
        session_id=self.session_id
    )
    self.stats["concepts_learned"] += 1

def recall(self, query: str, limit: int = 5) -> str:
    """학습 여정 복습"""
    result = self.memory_tool.execute(
        "search",
        query=query,
        limit=limit
    )
    return result

def get_stats(self) -> Dict[str, Any]:
    """학습 통계 가져오기"""
    duration = (datetime.now() - self.stats["session_start"]).total_seconds()
    return {
        "세션 지속 시간": f"{duration:.0f}s",
        "로드된 문서": self.stats["documents_loaded"],
        "질문 수": self.stats["questions_asked"],
        "학습 노트": self.stats["concepts_learned"],
        "현재 문서": self.current_document or "로드되지 않음"
    }

def generate_report(self, save_to_file: bool = True) -> Dict[str, Any]:
    """학습 보고서 생성"""
    memory_summary = self.memory_tool.execute("summary", limit=10)
    rag_stats = self.rag_tool.execute("stats")

    duration = (datetime.now() - self.stats["session_start"]).total_seconds()
    report = {
        "session_info": {
            "session_id": self.session_id,
            "user_id": self.user_id,
            "start_time": self.stats["session_start"].isoformat(),
            "duration_seconds": duration
        },
        "learning_metrics": {
            "documents_loaded": self.stats["documents_loaded"],
            "questions_asked": self.stats["questions_asked"],
            "concepts_learned": self.stats["concepts_learned"]
        },
        "memory_summary": memory_summary,
        "rag_status": rag_stats
    }

    if save_to_file:
        report_file = f"learning_report_{self.session_id}.json"
        with open(report_file, 'w', encoding='utf-8') as f:
            json.dump(report, f, ensure_ascii=False, indent=2, default=str)
        report["report_file"] = report_file

    return report
```

이 메서드들은 각각 다음을 구현한다:

- **add_note**: 학습 노트를 의미 기억에 저장
- **recall**: 메모리 시스템에서 학습 여정 검색
- **get_stats**: 현재 세션의 통계 정보 가져오기
- **generate_report**: 상세 학습 보고서를 생성하고 JSON 파일로 저장

### 8.4.5 실행 효과 시연

다음은 실행 효과 시연이다. 그림 8.7에서 보이듯이, 메인 페이지에 진입한 후 먼저 어시스턴트를 초기화해야 한다. 이는 데이터베이스, 모델, API 등의 로딩 작업이다. 그런 다음 PDF 문서를 전달하고 문서 로드 버튼을 클릭한다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-7.png" alt="" width="85%"/>
  <p>그림 8.7 Q&A 어시스턴트 메인 페이지</p>
</div>

첫 번째 기능은 지능형 Q&A로, 업로드된 문서를 기반으로 검색하고 관련 자료의 참조 출처와 유사도 계산을 반환할 수 있다. 이것은 RAG 도구 기능의 시연이며, 그림 8.8에 나타나 있다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-8.png" alt="" width="85%"/>
  <p>그림 8.8 Q&A 어시스턴트 메인 페이지</p>
</div>

두 번째 기능은 학습 노트다. 그림 8.9에서 보이듯이, 관련 개념을 선택하고 노트 내용을 작성할 수 있다. 이 부분은 Memory 도구를 사용하며 개인 노트를 데이터베이스에 저장하여 전체 학습 보고서의 통계 및 이후 반환에 용이하다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-9.png" alt="" width="85%"/>
  <p>그림 8.9 Q&A 어시스턴트 메인 페이지</p>
</div>

마지막으로 학습 진도 통계와 보고서 생성이 있다. 그림 8.10에서 보이듯이, 어시스턴트 사용 중 로드된 문서 수, 질문 수, 노트 수를 확인할 수 있다. 최종적으로 Q&A 결과와 노트가 JSON 문서로 정리되어 반환된다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-10.png" alt="" width="85%"/>
  <p>그림 8.10 Q&A 어시스턴트 메인 페이지</p>
</div>

이 Q&A 어시스턴트 사례를 통해 RAGTool과 MemoryTool을 사용하여 완전한 **Web 기반 지능형 문서 Q&A 시스템**을 구축하는 방법을 보여주었다. 완전한 코드는 `code/chapter8/11_Q&A_Assistant.py`에서 찾을 수 있다. 시작 후 `http://localhost:7860`을 방문하여 이 지능형 학습 어시스턴트를 사용할 수 있다.

독자들은 이 사례를 직접 실행하여 RAG와 Memory의 기능을 경험하고, 이를 기반으로 자신의 요구에 맞는 지능형 애플리케이션을 확장하고 커스터마이징하기를 권장한다!

## 8.5 장 요약 및 전망

이 장에서 HelloAgents 프레임워크에 두 가지 핵심 기능을 성공적으로 추가했다: 메모리 시스템과 RAG 시스템.

이 장의 내용을 깊이 학습하고 응용하고자 하는 독자들을 위해 다음과 같은 제안을 제공한다:

1. 처음부터 직접 기본 메모리 모듈을 설계하고, 점진적으로 더 복잡한 기능을 추가하여 반복한다.

2. 프로젝트에서 다양한 임베딩 모델과 검색 전략을 시도하고 평가하여, 특정 작업에 최적인 솔루션을 찾는다.

3. 배운 메모리와 RAG 시스템을 실제 개인 프로젝트에 적용하여, 실제 사용에서 기능을 테스트하고 개선한다.

고급 탐구

1. 최신 메모리 및 RAG 저장소를 추적하고 연구하여, 우수한 구현을 학습한다.
2. RAG 아키텍처를 멀티모달(텍스트 + 이미지) 또는 교차 모달 시나리오에 적용할 가능성을 탐구한다.
3. HelloAgents 오픈소스 프로젝트에 참여하여, 아이디어와 코드를 기여한다.

이 장의 학습을 통해 Memory와 RAG 시스템의 구현 기술을 습득했을 뿐만 아니라, 더 중요하게는 인지과학 이론을 실용적인 엔지니어링 솔루션으로 변환하는 방법을 이해했다. 이 학제 간 사고 방식은 AI 분야에서의 더욱 깊은 발전을 위한 견고한 기반을 마련할 것이다.

마지막으로, 그림 8.11의 마인드맵을 통해 이 장의 완전한 지식 체계를 요약해보자:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/8-figures/8-11.png" alt="" width="85%"/>
  <p>그림 8.11 Hello-agents 8장 지식 요약</p>
</div>

이 장은 HelloAgents 프레임워크의 메모리 시스템과 RAG 기술의 기능을 보여주었다. 우리는 성공적으로 진정으로 "지능적인" 학습 어시스턴트를 구축했다. 이 아키텍처는 고객 서비스, 기술 지원, 개인 어시스턴트 등 다른 응용 시나리오로 쉽게 확장될 수 있다.

다음 장에서는 맥락 엔지니어링을 통해 에이전트의 대화 품질과 사용자 경험을 더욱 향상시키는 방법을 계속 탐구할 것이다. 기대해주기 바란다!

## 연습 문제

> **참고**: 일부 연습 문제에는 표준 답이 없다. 학습자의 메모리 시스템과 RAG 기술에 대한 종합적인 이해와 실제 능력 개발에 초점을 맞추고 있다.

1. 이 장에서는 네 가지 메모리 유형을 소개했다: 작업 기억, 일화 기억, 의미 기억, 지각 기억. 다음을 분석하라:

   - 섹션 8.2.5에서 각 메모리 유형은 고유한 점수 공식을 갖는다. 일화 기억과 의미 기억의 점수 메커니즘을 비교하고, 일화 기억이 "시간 최신성"을 더 강조하는 이유(가중치 0.2)와 의미 기억이 "그래프 검색"을 더 강조하는 이유(가중치 0.3)를 설명하라.
   - "개인 건강 관리 어시스턴트"(사용자의 식이, 운동, 수면 데이터를 기록하고 건강 조언을 제공해야 함)를 설계한다면, 이 네 가지 메모리 유형을 어떻게 결합하겠는가? 각 메모리 유형에 대한 구체적인 응용 시나리오를 설계하라.
   - 작업 기억은 TTL(Time To Live) 메커니즘을 사용하여 만료된 데이터를 자동으로 정리한다. 어떤 상황에서 중요한 작업 기억이 장기 기억으로 "강화"되어야 하는가? 자동 강화 트리거 조건을 어떻게 설계하겠는가?

2. 섹션 8.3의 RAG 시스템에서 MarkItDown을 사용하여 다양한 포맷 문서를 Markdown으로 통합 변환한다. 깊이 생각해보라:

   > **참고**: 이것은 실습 문제로, 실제 조작을 권장한다

   - 현재 지능형 청킹 전략은 Markdown 제목 계층(#, ##, ###)을 기반으로 분할한다. 명확한 제목 구조가 없는 문서(예: 소설, 법률 조항)를 처리해야 한다면, 청킹 전략을 어떻게 최적화해야 하는가? "의미 경계"에 기반한 청킹 알고리즘을 구현해보라.
   - 섹션 8.3.5에서는 두 가지 고급 검색 전략을 소개했다: MQE(다중 쿼리 확장)와 HyDE(가상 문서 임베딩). 실제 시나리오(예: 기술 문서 Q&A, 의료 지식 검색)를 선택하여, 기본 검색, MQE, HyDE의 효과 차이를 비교하고, 각각의 적용 가능한 시나리오를 분석하라.
   - RAG 시스템의 검색 품질은 임베딩 모델의 선택에 크게 의존한다. 이 장에서 언급된 세 가지 임베딩 솔루션(바이리안 API, 로컬 Transformer, TF-IDF)을 정확도, 속도, 비용, 오프라인 배포 등의 차원에서 비교하고, 선택 권장 사항을 제시하라.

3. 메모리 시스템의 "망각" 메커니즘은 인간 인지를 시뮬레이션하는 중요한 설계다. 섹션 8.2.3의 MemoryTool을 기반으로 다음 확장 실습을 완료하라:

   > **참고**: 이것은 실습 문제로, 실제 조작을 권장한다

   - 현재 세 가지 망각 전략이 제공된다: 중요도 기반, 시간 기반, 용량 기반. 중요도, 접근 빈도, 시간 감쇠 등의 요인을 종합적으로 고려하는 "지능형 망각" 전략을 설계하고 구현하라. 가중치 점수를 사용하여 어떤 메모리를 망각해야 하는지 결정한다.
   - 장기 실행 에이전트 시스템에서 메모리 데이터베이스에 대량의 데이터가 축적될 수 있다. "메모리 아카이빙" 메커니즘을 설계하라: 장기간 사용되지 않았지만 잠재적으로 가치 있는 메모리를 콜드 스토리지로 이전하고, 필요할 때 복원한다. 이 메커니즘을 기존의 네 가지 메모리 유형과 어떻게 통합해야 하는가?
   - 생각해보라: 에이전트가 특정 민감한 정보(예: 사용자 개인 데이터)를 "망각"해야 한다면, 데이터베이스에서 삭제하는 것만으로 충분한가? 벡터 데이터베이스와 그래프 데이터베이스를 사용하는 경우, 데이터가 완전히 지워지도록 어떻게 보장하겠는가?

4. 섹션 8.4의 "지능형 학습 어시스턴트" 사례에서 MemoryTool과 RAGTool을 결합했다. 깊이 분석해보라:

   - 사례의 `ask_question()` 메서드는 RAG 검색과 메모리 검색을 모두 사용한다. 어떤 상황에서 RAG를 우선해야 하는가? 어떤 상황에서 Memory를 우선해야 하는가? 가장 적합한 검색 방법을 자동으로 선택하는 "지능형 라우팅" 메커니즘을 어떻게 설계하겠는가?
   - 현재 학습 보고서(`generate_report()`)는 통계 정보만 포함한다. 이 기능을 확장하고 더 지능적인 학습 보고서 생성기를 설계하라: 사용자의 학습 궤적을 분석하고, 지식 맹점을 식별하며, 다음 학습 내용을 추천할 수 있어야 한다. 어떤 메모리 유형과 검색 전략이 필요한가?
   - 이 학습 어시스턴트를 다중 사용자 Web 서비스로 배포하고 싶다고 가정하자. 각 사용자는 독립적인 메모리와 지식베이스를 갖는다. 데이터 격리 솔루션을 설계하라: Qdrant와 Neo4j에서 사용자 수준의 데이터 격리를 어떻게 구현하는가? 다중 사용자 시나리오에서 검색 성능을 어떻게 최적화하는가?

5. 의미 기억은 Neo4j 그래프 데이터베이스를 사용하여 지식 그래프를 저장한다. 다음을 생각해보라:

   - 섹션 8.2.5의 의미 기억 구현에서, 시스템은 자동으로 엔티티와 관계를 추출하여 지식 그래프를 구축한다. 이 자동 추출의 정확도는 어떠한가? 어떤 상황에서 잘못된 엔티티나 관계가 추출될 수 있는가? "지식 그래프 품질 평가" 메커니즘을 어떻게 설계하겠는가?
   - 지식 그래프의 중요한 장점은 복잡한 관계 추론을 지원한다는 것이다. Neo4j의 그래프 쿼리 기능(다중 홉 관계, 경로 찾기 등)을 완전히 활용하는 쿼리 시나리오를 설계하여, 순수 벡터 검색으로는 완수할 수 없는 작업을 수행하라.
   - 의미 기억의 "벡터 검색 + 그래프 검색" 하이브리드 전략과 순수 벡터 검색을 비교하라: 어떤 유형의 쿼리에서 그래프 검색이 상당한 성능 향상을 가져올 수 있는가? 구체적인 예시로 설명하라.

## 참고문헌

[1] Atkinson, R. C., & Shiffrin, R. M. (1968). Human memory: A proposed system and its control processes. In *Psychology of learning and motivation* (Vol. 2, pp. 89-195). Academic press.