# 제10장 에이전트 통신 프로토콜

앞 장들에서 우리는 추론, 도구 호출, 기억 능력을 갖춘 완성도 높은 단일 에이전트를 구축했다. 그러나 더 복잡한 AI 시스템을 구축하려 할 때, 자연스럽게 이런 의문이 생긴다: **에이전트가 외부 세계와 어떻게 효율적으로 상호작용하는가? 여러 에이전트가 어떻게 서로 협력하는가?**

이것이 바로 에이전트 통신 프로토콜이 해결하고자 하는 핵심 문제다. 본 장에서는 HelloAgents 프레임워크에 세 가지 통신 프로토콜을 도입한다: **MCP(Model Context Protocol)**는 에이전트와 도구 간의 표준화 통신에, **A2A(Agent-to-Agent Protocol)**는 에이전트 간 점대점(P2P) 협력에, **ANP(Agent Network Protocol)**는 대규모 에이전트 네트워크 구축에 사용된다. 이 세 가지 프로토콜이 함께 에이전트 통신의 기반 인프라 계층을 구성한다.

본 장의 학습을 통해 에이전트 통신 프로토콜의 설계 이념과 실천 기술을 습득하고, 세 가지 주류 프로토콜의 설계 차이를 이해하며, 실제 문제 해결에 적합한 프로토콜을 선택하는 방법을 배울 수 있다.

---

## 10.1 에이전트 통신 프로토콜 기초

**10.1.1 통신 프로토콜이 필요한 이유**

제7장에서 구축한 ReAct 에이전트를 떠올려보자. 그것은 이미 강력한 추론 및 도구 호출 능력을 갖추고 있었다. 전형적인 사용 시나리오를 살펴보자:

```python
from hello_agents import ReActAgent, HelloAgentsLLM
from hello_agents.tools import CalculatorTool, SearchTool

llm = HelloAgentsLLM()
agent = ReActAgent(name="AI助手", llm=llm)
agent.add_tool(CalculatorTool())
agent.add_tool(SearchTool())

# 智能体可以独立完成任务
response = agent.run("搜索最新的AI新闻，并计算相关公司的市值总和")
```

이 에이전트는 잘 동작하지만, 세 가지 근본적인 한계에 직면해 있다. 첫 번째는 **도구 통합의 딜레마**다: 새로운 외부 서비스(예: GitHub API, 데이터베이스, 파일 시스템)에 접근할 때마다 전용 Tool 클래스를 작성해야 한다. 이는 작업량이 많을 뿐만 아니라, 서로 다른 개발자가 작성한 도구들은 서로 호환되지 않는다. 두 번째는 **능력 확장의 병목**이다: 에이전트의 능력은 사전에 정의된 도구 집합에 한정되어 있어, 새로운 서비스를 동적으로 발견하고 사용할 수 없다. 세 번째는 **협업의 부재**다: 복잡한 작업에서 여러 전문 에이전트(예: 연구원+작성자+편집자)의 협력이 필요할 때, 수동 편성을 통해서만 조율이 가능하다.

좀 더 구체적인 예시를 통해 이 한계를 이해해보자. 지능형 연구 보조 시스템을 구축한다고 가정하면:

```python
# 전통 방식: 각 서비스를 수동으로 통합
class GitHubTool(BaseTool):
    """GitHub API 어댑터를 직접 작성해야 함"""
    def run(self, repo_url):
        # 대량의 API 호출 코드...
        pass

class DatabaseTool(BaseTool):
    """데이터베이스 어댑터를 직접 작성해야 함"""
    def run(self, query):
        # 데이터베이스 연결 및 쿼리 코드...
        pass

class WeatherTool(BaseTool):
    """날씨 API 어댑터를 직접 작성해야 함"""
    def run(self, location):
        # 날씨 API 호출 코드...
        pass

# 새로운 서비스마다 이 과정을 반복해야 함
agent.add_tool(GitHubTool())
agent.add_tool(DatabaseTool())
agent.add_tool(WeatherTool())
```

이 방식에는 명확한 문제점이 있다: 코드 중복(각 도구마다 HTTP 요청, 에러 처리, 인증 등을 처리), 유지보수 어려움(API 변경 시 관련된 모든 도구 수정 필요), 재사용 불가(다른 개발자의 도구를 직접 사용 불가), 확장성 부족(새 서비스 추가 시 대량의 코딩 작업 필요) 등이다.

**통신 프로토콜의 핵심 가치**는 바로 이러한 문제들을 해결하는 데 있다. 표준화된 인터페이스 규범을 제공하여, 에이전트가 각 서비스에 전용 어댑터를 작성하지 않고도 통일된 방식으로 다양한 외부 서비스에 접근할 수 있게 한다. 이는 마치 인터넷의 TCP/IP 프로토콜과 같다 — 각 장치에 전용 통신 코드를 작성하지 않아도 서로 다른 장치가 통신할 수 있게 해주는 것이다.

통신 프로토콜이 있으면 위의 코드를 다음과 같이 단순화할 수 있다:

```python
from hello_agents.tools import MCPTool

# MCP 서버에 연결하여 모든 도구를 자동으로 획득
mcp_tool = MCPTool()  # 내장 서버가 기본 도구 제공

# 또는 전문 MCP 서버에 연결
github_mcp = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-github"])
database_mcp = MCPTool(server_command=["python", "database_mcp_server.py"])

# 에이전트가 모든 능력을 자동으로 획득, 어댑터 작성 불필요
agent.add_tool(mcp_tool)
agent.add_tool(github_mcp)
agent.add_tool(database_mcp)
```

통신 프로토콜이 가져오는 변화는 근본적이다: **표준화 인터페이스**로 서로 다른 서비스가 통일된 접근 방식을 제공하고, **상호운용성**으로 서로 다른 개발자의 도구가 매끄럽게 통합되며, **동적 발견**으로 에이전트가 실행 중에 새로운 서비스와 능력을 발견할 수 있고, **확장성**으로 시스템이 새로운 기능 모듈을 쉽게 추가할 수 있다.

---

**10.1.2 세 가지 프로토콜 설계 이념 비교**

에이전트 통신 프로토콜은 단일 솔루션이 아니라, 서로 다른 통신 시나리오를 위해 설계된 일련의 표준이다. 본 장에서는 현재 업계 주류인 MCP, A2A, ANP 세 가지 프로토콜을 예시로 실습하며, 아래에 전체적인 비교를 제시한다.

**(1) MCP: 에이전트와 도구의 다리**

MCP(Model Context Protocol)는 Anthropic 팀이 제안한 것으로[1], 핵심 설계 이념은 **에이전트와 외부 도구/리소스 간의 통신 방식을 표준화**하는 것이다. 에이전트가 파일 시스템, 데이터베이스, GitHub, Slack 등 다양한 서비스에 접근해야 한다고 상상해보자. 전통적인 방법은 각 서비스에 전용 어댑터를 작성하는 것인데, 이는 작업량도 많고 유지보수도 어렵다. MCP는 통일된 프로토콜 규범을 정의하여, 모든 서비스가 동일한 방식으로 접근될 수 있게 한다.

MCP의 설계 철학은 "컨텍스트 공유"다. 단순한 RPC(원격 프로시저 호출) 프로토콜이 아니라, 에이전트와 도구 간의 풍부한 컨텍스트 정보 공유를 가능하게 한다. 그림 10.1처럼, 에이전트가 코드 저장소에 접근할 때, MCP 서버는 파일 내용뿐만 아니라 코드 구조, 의존 관계, 커밋 이력 등 컨텍스트 정보도 제공하여 에이전트가 더 지능적인 결정을 내릴 수 있게 한다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-1.png" alt="" width="85%"/>
  <p>그림 10.1 MCP 설계 사상</p>
</div>

**(2) A2A: 에이전트 간의 대화**

A2A(Agent-to-Agent Protocol)는 Google 팀이 제안한 것으로[2], 핵심 설계 이념은 **에이전트 간의 점대점 통신 구현**이다. MCP가 에이전트와 도구 간의 통신에 집중하는 반면, A2A는 에이전트들이 어떻게 서로 협력하는지에 집중한다. 이 설계는 에이전트가 인간 팀처럼 대화, 협상, 협력할 수 있게 한다.

A2A의 설계 철학은 "대등 통신"이다. 그림 10.2처럼, A2A 네트워크에서 각 에이전트는 서비스 제공자이자 서비스 소비자다. 에이전트는 능동적으로 요청을 발신할 수도 있고, 다른 에이전트의 요청에 응답할 수도 있다. 이 대등 설계는 중앙화된 코디네이터의 병목을 피하여 에이전트 네트워크를 더 유연하고 확장 가능하게 만든다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-2.png" alt="" width="85%"/>
  <p>그림 10.2 A2A 설계 사상</p>
</div>

**(3) ANP: 에이전트 네트워크의 인프라**

ANP(Agent Network Protocol)는 개념적 프로토콜 프레임워크로[3], 현재 오픈소스 커뮤니티가 유지 관리하며 아직 성숙한 생태계가 형성되지 않았다. 핵심 설계 이념은 **대규모 에이전트 네트워크를 위한 인프라 구축**이다. MCP가 "도구에 어떻게 접근하는가"를 해결하고 A2A가 "다른 에이전트와 어떻게 대화하는가"를 해결한다면, ANP는 "대규모 네트워크에서 어떻게 에이전트를 발견하고 연결하는가"를 해결한다.

ANP의 설계 철학은 "탈중앙화 서비스 발견"이다. 수백, 수천 개의 에이전트를 포함하는 네트워크에서, 에이전트가 필요한 서비스를 어떻게 찾을 수 있을까? 그림 10.3처럼, ANP는 서비스 등록, 발견, 라우팅 메커니즘을 제공하여, 에이전트가 모든 연결 관계를 사전에 구성하지 않고도 네트워크에서 다른 서비스를 동적으로 발견할 수 있게 한다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-3.png" alt="" width="85%"/>
  <p>그림 10.3 ANP 설계 사상</p>
</div>

표 10.1에서 세 가지 프로토콜의 차이를 대조표로 좀 더 명확하게 이해해보자:

<div align="center">
  <p>표 10.1 세 가지 프로토콜 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-1.png" alt="" width="85%"/>
</div>

**(4) 적합한 프로토콜을 어떻게 선택하는가?**

현재 프로토콜들은 아직 발전 초기 단계이며, MCP의 생태계가 상대적으로 성숙해 있다. 단, 각종 도구의 최신성은 관리자에 따라 다르므로, 대기업이 지원하는 MCP 도구를 선택하는 것이 더 권장된다.

프로토콜 선택의 핵심은 자신의 요구사항을 이해하는 것이다:

- 에이전트가 외부 서비스(파일, 데이터베이스, API)에 접근해야 한다면 **MCP** 선택
- 여러 에이전트가 서로 협력하여 작업을 완료해야 한다면 **A2A** 선택
- 대규모 에이전트 생태계를 구축하려 한다면 **ANP** 고려

---

**10.1.3 HelloAgents 통신 프로토콜 아키텍처 설계**

세 가지 프로토콜의 설계 이념을 이해한 후, HelloAgents 프레임워크에서 이것들을 어떻게 구현하고 사용하는지 살펴보자. 설계 목표는 **학습자가 최대한 간단한 방식으로 이 프로토콜들을 사용하면서, 복잡한 시나리오에 대응할 수 있는 충분한 유연성을 유지**하는 것이다.

그림 10.4처럼, HelloAgents의 통신 프로토콜 아키텍처는 세 계층 설계를 채택하며, 하위부터 상위로 각각: 프로토콜 구현 계층, 도구 캡슐화 계층, 에이전트 통합 계층이다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-4.png" alt="" width="85%"/>
  <p>그림 10.4 HelloAgents 통신 프로토콜 설계</p>
</div>

**(1) 프로토콜 구현 계층**: 이 계층은 세 가지 프로토콜의 구체적인 구현을 포함한다. MCP는 FastMCP 라이브러리를 기반으로 구현되어 클라이언트와 서버 기능을 제공하고, A2A는 Google 공식 a2a-sdk를 기반으로 구현되며, ANP는 자체 개발한 경량 구현으로 서비스 발견 및 네트워크 관리 기능을 제공한다. 물론 공식 [구현](https://github.com/agent-network-protocol/AgentConnect)도 있으나, 이후 반복 개발 가능성을 고려하여 여기서는 개념적 시뮬레이션만 수행한다.

**(2) 도구 캡슐화 계층**: 이 계층은 프로토콜 구현을 통일된 Tool 인터페이스로 캡슐화한다. MCPTool, A2ATool, ANPTool은 모두 BaseTool을 상속하며 일관된 `run()` 메서드를 제공한다. 이 설계를 통해 에이전트는 동일한 방식으로 서로 다른 프로토콜을 사용할 수 있다.

**(3) 에이전트 통합 계층**: 이 계층은 에이전트와 프로토콜의 통합 지점이다. 모든 에이전트(ReActAgent, SimpleAgent 등)는 Tool System을 통해 프로토콜 도구를 사용하며, 하위 프로토콜 세부 사항에 신경 쓸 필요가 없다.

---

**10.1.4 본 장 학습 목표와 빠른 체험**

먼저 제10장의 학습 내용을 살펴보자:

```
hello_agents/
├── protocols/                          # 통신 프로토콜 모듈
│   ├── mcp/                            # MCP 프로토콜 구현 (Model Context Protocol)
│   │   ├── client.py                   # MCP 클라이언트 (5가지 전송 방식 지원)
│   │   ├── server.py                   # MCP 서버 (FastMCP 캡슐화)
│   │   └── utils.py                    # 유틸리티 함수 (create_context/parse_context)
│   ├── a2a/                            # A2A 프로토콜 구현 (Agent-to-Agent Protocol)
│   │   └── implementation.py           # A2A 서버/클라이언트 (a2a-sdk 기반, 선택적 의존성)
│   └── anp/                            # ANP 프로토콜 구현 (Agent Network Protocol)
│       └── implementation.py           # ANP 서비스 발견/등록 (개념적 구현)
└── tools/builtin/                      # 내장 도구 모듈
    └── protocol_tools.py               # 프로토콜 도구 래퍼 (MCPTool/A2ATool/ANPTool)
```

이 장의 내용은 주로 응용 위주이며, 학습 목표는 자신의 프로젝트에서 프로토콜을 적용할 수 있는 능력을 갖추는 것이다. 또한 프로토콜은 현재 초기 발전 단계에 있으므로, 바퀴를 다시 만드는 데 너무 많은 에너지를 쏟을 필요는 없다. 실전에 들어가기 전에 먼저 개발 환경을 준비해두자:

```bash
# HelloAgents 프레임워크 설치 (제10장 버전)
pip install "hello-agents[protocol]==0.2.2"

# NodeJS 설치, Additional-Chapter의 문서 참고 가능
```

가장 간단한 코드로 세 가지 프로토콜의 기본 기능을 체험해보자:

```python
from hello_agents.tools import MCPTool, A2ATool, ANPTool

# 1. MCP: 도구 접근
mcp_tool = MCPTool()
result = mcp_tool.run({
    "action": "call_tool",
    "tool_name": "add",
    "arguments": {"a": 10, "b": 20}
})
print(f"MCP 계산 결과: {result}")  # 출력: 30.0

# 2. ANP: 서비스 발견
anp_tool = ANPTool()
anp_tool.run({
    "action": "register_service",
    "service_id": "calculator",
    "service_type": "math",
    "endpoint": "http://localhost:8080"
})
services = anp_tool.run({"action": "discover_services"})
print(f"발견된 서비스: {services}")

# 3. A2A: 에이전트 통신
a2a_tool = A2ATool("http://localhost:5000")
print("A2A 도구 생성 성공")
```

이 간단한 예시는 세 가지 프로토콜의 핵심 기능을 보여준다. 이후 절에서 각 프로토콜의 상세한 사용법과 최선의 실천 방법을 심층적으로 학습한다.

---

## 10.2 MCP 프로토콜 실전

이제 MCP를 심층적으로 학습하여, 에이전트가 외부 도구와 리소스에 접근하는 방법을 익혀보자.

**10.2.1 MCP 프로토콜 개념 소개**

**(1) MCP: 에이전트의 "USB-C"**

에이전트가 동시에 다양한 작업을 수행해야 한다고 상상해보자, 예를 들면:
- 로컬 파일 시스템의 문서 읽기
- PostgreSQL 데이터베이스 쿼리
- GitHub에서 코드 검색
- Slack 메시지 전송
- Google Drive 접근

전통적인 방식에서는 각 서비스에 어댑터 코드를 작성하고, 서로 다른 API, 인증 방식, 에러 처리 등을 다뤄야 한다. 이는 작업량도 많고 유지보수도 어렵다. 더 중요한 것은, 서로 다른 LLM 플랫폼의 function call 구현 방식이 크게 달라서 모델을 교체할 때 대량의 코드를 다시 작성해야 한다.

MCP의 등장이 이 모든 것을 바꿨다. USB-C가 다양한 장치의 연결 방식을 통일한 것처럼, **MCP는 에이전트와 외부 도구의 상호작용 방식을 통일**한다. Claude, GPT, 기타 모델 등 어떤 것을 사용하든, MCP 프로토콜을 지원하기만 하면 동일한 도구와 리소스에 매끄럽게 접근할 수 있다.

**(2) MCP 아키텍처**

MCP 프로토콜은 Host, Client, Servers 세 계층 아키텍처 설계를 채택한다. 그림 10.5의 시나리오를 통해 이 구성 요소들이 어떻게 협동하는지 이해해보자.

Claude Desktop을 사용하여 "내 데스크탑에 어떤 문서가 있나요?"라고 질문한다고 가정해보자.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-5.png" alt="" width="85%"/>
  <p>그림 10.5 MCP 사례 시연</p>
</div>

**세 계층 아키텍처의 역할:**

1. **Host(호스트 계층)**: Claude Desktop이 Host 역할을 하며, 사용자의 질문을 받고 Claude 모델과 상호작용한다. Host는 사용자가 직접 상호작용하는 인터페이스이며, 전체 대화 흐름을 관리한다.

2. **Client(클라이언트 계층)**: Claude 모델이 파일 시스템에 접근해야 한다고 판단하면, Host에 내장된 MCP Client가 활성화된다. Client는 적절한 MCP Server와 연결을 수립하고, 요청을 전송하고 응답을 받는다.

3. **Server(서버 계층)**: 파일 시스템 MCP Server가 호출되어 실제 파일 스캔 작업을 수행하고, 데스크탑 디렉토리에 접근하여 발견된 문서 목록을 반환한다.

**완전한 상호작용 흐름**: 사용자 질문 → Claude Desktop(Host) → Claude 모델 분석 → 파일 정보 필요 → MCP Client 연결 → 파일 시스템 MCP Server → 작업 실행 → 결과 반환 → Claude 답변 생성 → Claude Desktop에 표시

이 아키텍처 설계의 장점은 **관심사 분리**에 있다: Host는 사용자 경험에, Client는 프로토콜 통신에, Server는 구체적인 기능 구현에 집중한다. 개발자는 해당하는 MCP Server 개발에만 집중하면 되고, Host와 Client의 구현 세부 사항에 신경 쓸 필요가 없다.

**(3) MCP의 핵심 능력**

표 10.2에서 MCP 프로토콜이 제공하는 세 가지 핵심 능력을 볼 수 있으며, 이것들이 완전한 도구 접근 프레임워크를 구성한다:

<div align="center">
  <p>표 10.2 MCP 핵심 능력</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-2.png" alt="" width="85%"/>
</div>

세 가지 능력의 차이: **Tools는 능동적**(작업 실행), **Resources는 수동적**(데이터 제공), **Prompts는 지도적**(템플릿 제공)이다.

**(4) MCP의 작업 흐름**

구체적인 예시를 통해 MCP의 완전한 작업 흐름을 이해해보자. 그림 10.6을 참고한다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-6.png" alt="" width="85%"/>
  <p>그림 10.6 MCP 사례 시연</p>
</div>

핵심 문제: **Claude(또는 다른 LLM)는 어떻게 어떤 도구를 사용할지 결정하는가?**

사용자가 질문을 제기할 때의 완전한 도구 선택 흐름은 다음과 같다:

1. **도구 발견 단계**: MCP Client가 Server에 연결된 후, 먼저 `list_tools()`를 호출하여 모든 사용 가능한 도구의 설명 정보(도구 이름, 기능 설명, 파라미터 정의 포함)를 가져온다.

2. **컨텍스트 구성**: Client가 도구 목록을 LLM이 이해할 수 있는 형식으로 변환하여 시스템 프롬프트에 추가한다. 예를 들어:
   ```
   당신은 다음 도구를 사용할 수 있습니다:
   - read_file(path: str): 지정된 경로의 파일 내용 읽기
   - search_code(query: str, language: str): 코드베이스에서 검색
   ```

3. **모델 추론**: LLM이 사용자 질문과 사용 가능한 도구를 분석하여, 도구를 호출할지 여부와 어떤 도구를 호출할지 결정한다. 이 결정은 도구의 설명과 현재 대화 컨텍스트를 기반으로 한다.

4. **도구 실행**: LLM이 도구 사용을 결정하면, Client가 MCP Server를 통해 선택한 도구를 실행하고 결과를 가져온다.

5. **결과 통합**: 도구 실행 결과가 LLM에게 다시 전달되고, LLM이 결과를 결합하여 최종 답변을 생성한다.

이 과정은 **완전히 자동화**되어 있으며, LLM은 도구 설명의 품질에 따라 도구를 사용할지 여부와 방법을 결정한다. 따라서 명확하고 정확한 도구 설명을 작성하는 것이 매우 중요하다.

**(5) MCP와 Function Calling의 차이**

많은 개발자들이 묻는다: **이미 Function Calling을 사용하고 있는데, 왜 MCP가 필요한가?** 표 10.3을 통해 차이를 이해해보자.

<div align="center">
  <p>표 10.3 Function Calling과 MCP 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-3.png" alt="" width="85%"/>
</div>

에이전트가 GitHub 저장소와 로컬 파일 시스템에 접근해야 하는 동일한 작업의 두 가지 구현 방식을 비교해보자.

**방식 1: Function Calling 사용**

```python
# 단계1: 각 LLM 제공업체에 맞게 함수 정의
# OpenAI 형식
openai_tools = [
    {
        "type": "function",
        "function": {
            "name": "search_github",
            "description": "GitHub 저장소 검색",
            "parameters": {
                "type": "object",
                "properties": {
                    "query": {"type": "string", "description": "검색 키워드"}
                },
                "required": ["query"]
            }
        }
    }
]

# Claude 형식
claude_tools = [
    {
        "name": "search_github",
        "description": "GitHub 저장소 검색",
        "input_schema": {  # 주의: parameters가 아님
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "검색 키워드"}
            },
            "required": ["query"]
        }
    }
]

# 단계2: 도구 함수를 직접 구현
def search_github(query):
    import requests
    response = requests.get(
        "https://api.github.com/search/repositories",
        params={"q": query}
    )
    return response.json()

# 단계3: 서로 다른 모델의 응답 형식 처리
# OpenAI 응답
if response.choices[0].message.tool_calls:
    tool_call = response.choices[0].message.tool_calls[0]
    result = search_github(**json.loads(tool_call.function.arguments))

# Claude 응답
if response.content[0].type == "tool_use":
    tool_use = response.content[0]
    result = search_github(**tool_use.input)
```

**방식 2: MCP 사용**

```python
from hello_agents.protocols import MCPClient

# 단계1: 커뮤니티에서 제공하는 MCP 서버에 연결 (직접 구현할 필요 없음)
github_client = MCPClient([
    "npx", "-y", "@modelcontextprotocol/server-github"
])

fs_client = MCPClient([
    "npx", "-y", "@modelcontextprotocol/server-filesystem", "."
])

# 단계2: 통일된 호출 방식 (모델과 무관)
async with github_client:
    # 자동으로 도구 발견
    tools = await github_client.list_tools()

    # 도구 호출 (표준화 인터페이스)
    result = await github_client.call_tool(
        "search_repositories",
        {"query": "AI agents"}
    )

# 단계3: MCP를 지원하는 모든 모델이 사용 가능
# OpenAI, Claude, Llama 등 모두 동일한 MCP 클라이언트 사용
```

먼저 명확히 해야 할 것은, Function Calling과 MCP는 경쟁 관계가 아니라 상호 보완적이라는 점이다. Function Calling은 대형 언어 모델의 핵심 능력으로, 모델의 내재적 지능을 구현하며, 언제 함수를 호출해야 하는지 이해하고 해당 호출 파라미터를 정확하게 생성한다. 반면 MCP는 인프라 프로토콜의 역할을 하며, 엔지니어링 차원에서 도구와 모델이 어떻게 연결되는지의 문제를 해결하고, 표준화된 방식으로 도구를 기술하고 호출한다.

간단한 비유를 들면: Function Calling은 "전화를 거는 방법"이라는 기술을 익힌 것으로, 언제 다이얼을 돌리는지, 상대방과 어떻게 소통하는지, 언제 끊는지를 포함한다. 그리고 MCP는 전 세계적으로 통일된 "전화 통신 표준"으로, 어떤 전화기도 다른 전화기에 순조롭게 연결될 수 있게 보장한다.

두 프로토콜의 상호 보완 관계를 이해했으니, 이제 HelloAgents에서 MCP 프로토콜을 사용하는 방법을 살펴보자.

---

**10.2.2 MCP 클라이언트 사용**

HelloAgents는 FastMCP 2.0을 기반으로 완전한 MCP 클라이언트 기능을 구현했다. 비동기와 동기 두 가지 API를 제공하여 다양한 사용 시나리오에 적응할 수 있다. 대부분의 응용에서는 비동기 API를 권장하는데, 동시 요청과 오래 실행되는 작업을 더 잘 처리할 수 있기 때문이다. 아래에서 분해된 조작 시연을 제공한다.

**(1) MCP 서버에 연결**

MCP 클라이언트는 여러 연결 방식을 지원하며, 가장 일반적인 것은 Stdio 모드(표준 입출력을 통해 로컬 프로세스와 통신)다:

```python
import asyncio
from hello_agents.protocols import MCPClient

async def connect_to_server():
    # 방식1: 커뮤니티에서 제공하는 파일 시스템 서버에 연결
    # npx가 자동으로 @modelcontextprotocol/server-filesystem 패키지를 다운로드하고 실행
    client = MCPClient([
        "npx", "-y",
        "@modelcontextprotocol/server-filesystem",
        "."  # 루트 디렉토리 지정
    ])

    # async with를 사용하여 연결이 올바르게 종료되도록 보장
    async with client:
        # 여기서 client 사용
        tools = await client.list_tools()
        print(f"사용 가능한 도구: {[t['name'] for t in tools]}")

    # 방식2: 커스텀 Python MCP 서버에 연결
    client = MCPClient(["python", "my_mcp_server.py"])
    async with client:
        # client 사용...
        pass

# 비동기 함수 실행
asyncio.run(connect_to_server())
```

**(2) 사용 가능한 도구 발견**

연결 후, 첫 번째 단계는 서버가 제공하는 도구를 조회하는 것이다:

```python
async def discover_tools():
    client = MCPClient(["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

    async with client:
        # 모든 사용 가능한 도구 가져오기
        tools = await client.list_tools()

        print(f"서버에서 {len(tools)}개의 도구 제공:")
        for tool in tools:
            print(f"\n도구 이름: {tool['name']}")
            print(f"설명: {tool.get('description', '설명 없음')}")

            # 파라미터 정보 출력
            if 'inputSchema' in tool:
                schema = tool['inputSchema']
                if 'properties' in schema:
                    print("파라미터:")
                    for param_name, param_info in schema['properties'].items():
                        param_type = param_info.get('type', 'any')
                        param_desc = param_info.get('description', '')
                        print(f"  - {param_name} ({param_type}): {param_desc}")

asyncio.run(discover_tools())

# 출력 예시:
# 서버에서 5개의 도구 제공:
#
# 도구 이름: read_file
# 설명: 파일 내용 읽기
# 파라미터:
#   - path (string): 파일 경로
#
# 도구 이름: write_file
# 설명: 파일 내용 쓰기
# 파라미터:
#   - path (string): 파일 경로
#   - content (string): 파일 내용
```

**(3) 도구 호출**

도구를 호출할 때는 도구 이름과 JSON Schema에 맞는 파라미터만 제공하면 된다:

```python
async def use_tools():
    client = MCPClient(["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

    async with client:
        # 파일 읽기
        result = await client.call_tool("read_file", {"path": "my_README.md"})
        print(f"파일 내용:\n{result}")

        # 디렉토리 목록
        result = await client.call_tool("list_directory", {"path": "."})
        print(f"현재 디렉토리 파일: {result}")

        # 파일 쓰기
        result = await client.call_tool("write_file", {
            "path": "output.txt",
            "content": "Hello from MCP!"
        })
        print(f"쓰기 결과: {result}")

asyncio.run(use_tools())
```

더 안전한 방식으로 MCP 서비스를 호출하는 방법을 참고용으로 제공한다:

```python
async def safe_tool_call():
    client = MCPClient(["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

    async with client:
        try:
            # 존재하지 않을 수 있는 파일 읽기 시도
            result = await client.call_tool("read_file", {"path": "nonexistent.txt"})
            print(result)
        except Exception as e:
            print(f"도구 호출 실패: {e}")
            # 재시도, 기본값 사용 또는 사용자에게 에러 보고 선택 가능

asyncio.run(safe_tool_call())
```

**(4) 리소스 접근**

도구 외에도 MCP 서버는 리소스(Resources)를 제공할 수 있다:

```python
# 사용 가능한 리소스 목록
resources = client.list_resources()
print(f"사용 가능한 리소스: {[r['uri'] for r in resources]}")

# 리소스 읽기
resource_content = client.read_resource("file:///path/to/resource")
print(f"리소스 내용: {resource_content}")
```

**(5) 프롬프트 템플릿 사용**

MCP 서버는 사전 정의된 프롬프트 템플릿(Prompts)을 제공할 수 있다:

```python
# 사용 가능한 프롬프트 목록
prompts = client.list_prompts()
print(f"사용 가능한 프롬프트: {[p['name'] for p in prompts]}")

# 프롬프트 내용 가져오기
prompt = client.get_prompt("code_review", {"language": "python"})
print(f"프롬프트 내용: {prompt}")
```

**(6) 완전한 예시: GitHub MCP 서비스 사용**

캡슐화된 MCP Tools를 사용하여 커뮤니티에서 제공하는 GitHub MCP 서비스를 사용하는 완전한 예시를 살펴보자:

```python
"""
GitHub MCP 서비스 예시

주의: 환경 변수 설정 필요
    Windows: $env:GITHUB_PERSONAL_ACCESS_TOKEN="your_token_here"
    Linux/macOS: export GITHUB_PERSONAL_ACCESS_TOKEN="your_token_here"
"""

from hello_agents.tools import MCPTool

# GitHub MCP 도구 생성
github_tool = MCPTool(
    server_command=["npx", "-y", "@modelcontextprotocol/server-github"]
)

# 1. 사용 가능한 도구 목록
print("📋 사용 가능한 도구:")
result = github_tool.run({"action": "list_tools"})
print(result)

# 2. 저장소 검색
print("\n🔍 저장소 검색:")
result = github_tool.run({
    "action": "call_tool",
    "tool_name": "search_repositories",
    "arguments": {
        "query": "AI agents language:python",
        "page": 1,
        "perPage": 3
    }
})
print(result)
```

---

**10.2.3 MCP 전송 방식 상세**

MCP 프로토콜의 중요한 특징은 **전송 계층 독립성**(Transport Agnostic)이다. MCP 프로토콜 자체는 특정 전송 방식에 의존하지 않으며, 서로 다른 통신 채널에서 실행될 수 있다. HelloAgents는 FastMCP 2.0을 기반으로 완전한 전송 방식 지원을 제공하여, 실제 시나리오에 맞는 전송 모드를 선택할 수 있다.

**(1) 전송 방식 개요**

HelloAgents의 `MCPClient`는 다섯 가지 전송 방식을 지원하며, 각각 다른 사용 시나리오에 맞다. 표 10.4를 참고한다:

<div align="center">
  <p>표 10.4 MCP 전송 방식 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-4.png" alt="" width="85%"/>
</div>

**(2) 전송 방식 사용 예시**

```python
from hello_agents.tools import MCPTool

# 1. Memory Transport - 메모리 전송 (테스트용)
# 파라미터 없이 내장 데모 서버 사용
mcp_tool = MCPTool()

# 2. Stdio Transport - 표준 입출력 전송 (로컬 개발)
# 명령어 목록을 사용하여 로컬 서버 시작
mcp_tool = MCPTool(server_command=["python", "examples/mcp_example_server.py"])

# 3. Stdio Transport with Args - 인수가 있는 명령어 전송
# 추가 인수 전달 가능
mcp_tool = MCPTool(server_command=["python", "examples/mcp_example_server.py", "--debug"])

# 4. Stdio Transport - 커뮤니티 서버 (npx 방식)
# npx를 사용하여 커뮤니티 MCP 서버 시작
mcp_tool = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

# 5. HTTP/SSE/StreamableHTTP Transport
# 주의: MCPTool은 주로 Stdio와 Memory 전송에 사용
# HTTP/SSE 등 원격 전송의 경우 MCPClient를 직접 사용 권장
```

**(3) Memory Transport - 메모리 전송**

적용 시나리오: 단위 테스트, 빠른 프로토타입 개발

```python
from hello_agents.tools import MCPTool

# 내장 데모 서버 사용 (Memory 전송)
mcp_tool = MCPTool()

# 사용 가능한 도구 목록
result = mcp_tool.run({"action": "list_tools"})
print(result)

# 도구 호출
result = mcp_tool.run({
    "action": "call_tool",
    "tool_name": "add",
    "arguments": {"a": 10, "b": 20}
})
print(result)
```

**(4) Stdio Transport - 표준 입출력 전송**

적용 시나리오: 로컬 개발, 디버깅, Python 스크립트 서버

```python
from hello_agents.tools import MCPTool

# 방식1: 커스텀 Python 서버 사용
mcp_tool = MCPTool(server_command=["python", "my_mcp_server.py"])

# 방식2: 커뮤니티 서버 사용 (파일 시스템)
mcp_tool = MCPTool(server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."])

# 도구 목록
result = mcp_tool.run({"action": "list_tools"})
print(result)

# 도구 호출
result = mcp_tool.run({
    "action": "call_tool",
    "tool_name": "read_file",
    "arguments": {"path": "README.md"}
})
print(result)
```

**(5) HTTP Transport - HTTP 전송**

적용 시나리오: 프로덕션 환경, 원격 서비스, 마이크로서비스 아키텍처

```python
# 주의: MCPTool은 주로 Stdio와 Memory 전송에 사용
# HTTP/SSE 등 원격 전송의 경우 하위 MCPClient 사용 권장

import asyncio
from hello_agents.protocols import MCPClient

async def test_http_transport():
    # 원격 HTTP MCP 서버에 연결
    client = MCPClient("http://api.example.com/mcp")

    async with client:
        # 서버 정보 가져오기
        tools = await client.list_tools()
        print(f"원격 서버 도구: {len(tools)}개")

        # 원격 도구 호출
        result = await client.call_tool("process_data", {
            "data": "Hello, World!",
            "operation": "uppercase"
        })
        print(f"원격 처리 결과: {result}")

# 주의: 실제 HTTP MCP 서버 필요
# asyncio.run(test_http_transport())
```

**(6) SSE Transport - Server-Sent Events 전송**

적용 시나리오: 실시간 통신, 스트리밍 처리, 장시간 연결

```python
# 주의: MCPTool은 주로 Stdio와 Memory 전송에 사용
# SSE 전송의 경우 하위 MCPClient 사용 권장

import asyncio
from hello_agents.protocols import MCPClient

async def test_sse_transport():
    # SSE MCP 서버에 연결
    client = MCPClient(
        "http://localhost:8080/sse",
        transport_type="sse"
    )

    async with client:
        # SSE는 스트리밍 처리에 특히 적합
        result = await client.call_tool("stream_process", {
            "input": "대용량 데이터 처리 요청",
            "stream": True
        })
        print(f"스트리밍 처리 결과: {result}")

# 주의: SSE를 지원하는 MCP 서버 필요
# asyncio.run(test_sse_transport())
```

**(7) StreamableHTTP Transport - 스트리밍 HTTP 전송**

적용 시나리오: 양방향 스트리밍 통신이 필요한 HTTP 시나리오

```python
# 주의: MCPTool은 주로 Stdio와 Memory 전송에 사용
# StreamableHTTP 전송의 경우 하위 MCPClient 사용 권장

import asyncio
from hello_agents.protocols import MCPClient

async def test_streamable_http_transport():
    # StreamableHTTP MCP 서버에 연결
    client = MCPClient(
        "http://localhost:8080/mcp",
        transport_type="streamable_http"
    )

    async with client:
        # 양방향 스트리밍 통신 지원
        tools = await client.list_tools()
        print(f"StreamableHTTP 서버 도구: {len(tools)}개")

# 주의: StreamableHTTP를 지원하는 MCP 서버 필요
# asyncio.run(test_streamable_http_transport())
```

---

**10.2.4 에이전트에서 MCP 도구 사용**

앞에서 MCP 클라이언트를 직접 사용하는 방법을 배웠다. 그러나 실제 응용에서는 에이전트가 MCP 도구를 **자동으로** 호출하길 원하지, 수동으로 호출 코드를 작성하고 싶지 않을 것이다. HelloAgents는 `MCPTool` 래퍼를 제공하여 MCP 서버를 에이전트의 도구 체인에 매끄럽게 통합할 수 있게 한다.

**(1) MCP 도구의 자동 전개 메커니즘**

HelloAgents의 `MCPTool`에는 특징이 있다: **자동 전개**다. MCPTool을 Agent에 추가하면, MCP 서버가 제공하는 모든 도구를 자동으로 독립적인 도구로 전개하여 Agent가 일반 도구를 호출하듯 사용할 수 있게 한다.

**방식 1: 내장 데모 서버 사용**

이전에 계산기 도구 함수를 구현한 적이 있는데, 여기서 그것을 MCP 서비스로 변환한다. 이것이 가장 간단한 사용 방식이다.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

agent = SimpleAgent(name="助手", llm=HelloAgentsLLM())

# 아무런 설정 없이 내장 데모 서버 자동 사용
mcp_tool = MCPTool(name="calculator")
agent.add_tool(mcp_tool)
# ✅ MCP 도구 'calculator'가 6개의 독립 도구로 전개됨

# 에이전트가 전개된 도구를 직접 사용 가능
response = agent.run("25 곱하기 16을 계산해")
print(response)  # 출력: 25 곱하기 16의 결과는 400
```

**자동 전개 후의 도구들**:

- `calculator_add` - 덧셈 계산기
- `calculator_subtract` - 뺄셈 계산기
- `calculator_multiply` - 곱셈 계산기
- `calculator_divide` - 나눗셈 계산기
- `calculator_greet` - 친근한 인사
- `calculator_get_system_info` - 시스템 정보 가져오기

Agent 호출 시 파라미터만 제공하면 된다. 예: `[TOOL_CALL:calculator_multiply:a=25,b=16]`, 시스템이 자동으로 타입 변환과 MCP 호출을 처리한다.

**방식 2: 외부 MCP 서버에 연결**

실제 프로젝트에서는 더 강력한 MCP 서버에 연결해야 한다. 이 서버들은 다음일 수 있다:
- **커뮤니티에서 제공하는 공식 서버** (파일 시스템, GitHub, 데이터베이스 등)
- **직접 작성한 커스텀 서버** (비즈니스 로직 캡슐화)

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

agent = SimpleAgent(name="파일 보조", llm=HelloAgentsLLM())

# 예시1: 커뮤니티에서 제공하는 파일 시스템 서버에 연결
fs_tool = MCPTool(
    name="filesystem",  # 고유 이름 지정
    description="로컬 파일 시스템 접근",
    server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."]
)
agent.add_tool(fs_tool)

# 예시2: 커스텀 Python MCP 서버에 연결
# 커스텀 MCP 서버를 작성하는 방법은 10.5절 참고
custom_tool = MCPTool(
    name="custom_server",  # 다른 이름 사용
    description="커스텀 비즈니스 로직 서버",
    server_command=["python", "my_mcp_server.py"]
)
agent.add_tool(custom_tool)

# Agent가 이제 자동으로 이 도구들을 사용 가능!
response = agent.run("my_README.md 파일을 읽고 주요 내용을 요약해줘")
print(response)
```

여러 MCP 서버를 사용할 때는 각 MCPTool에 다른 name을 반드시 지정해야 한다. 이 name이 전개된 도구 이름의 접두사로 추가되어 충돌을 방지한다. 예: `name="fs"`는 `fs_read_file`, `fs_write_file` 등으로 전개된다. 특정 비즈니스 로직을 캡슐화하기 위해 자신만의 MCP 서버를 작성해야 한다면 10.5절 내용을 참고한다.

**(2) MCP 도구 자동 전개의 작동 원리**

자동 전개 메커니즘을 이해하면 MCP 도구를 더 잘 사용할 수 있다. 어떻게 작동하는지 깊이 살펴보자:

```python
# 사용자 코드
fs_tool = MCPTool(name="fs", server_command=[...])
agent.add_tool(fs_tool)

# 내부에서 일어나는 일:
# 1. MCPTool이 서버에 연결하여 14개의 도구 발견
# 2. 각 도구에 대한 래퍼 생성:
#    - fs_read_text_file (파라미터: path, tail, head)
#    - fs_write_file (파라미터: path, content)
#    - ...
# 3. Agent의 도구 레지스트리에 등록

# Agent 호출
response = agent.run("README.md 읽기")

# Agent 내부:
# 1. fs_read_text_file 호출 필요 인식
# 2. 파라미터 생성: path=README.md
# 3. 래퍼가 MCP 형식으로 변환:
#    {"action": "call_tool", "tool_name": "read_text_file", "arguments": {"path": "README.md"}}
# 4. MCP 서버 호출
# 5. 파일 내용 반환
```

시스템은 도구의 파라미터 정의에 따라 자동으로 타입을 변환한다:

```python
# Agent가 계산기 호출
agent.run("25 곱하기 16 계산")

# Agent 생성: a=25,b=16 (문자열)
# 시스템이 자동 변환: {"a": 25.0, "b": 16.0} (숫자)
# MCP 서버가 올바른 숫자 타입 수신
```

**(3) 실전 사례: 지능형 문서 보조**

완전한 지능형 문서 보조를 구축해보자. 여기서는 간단한 멀티 에이전트 편성으로 시연한다:

```python
"""
멀티 Agent 협력 지능형 문서 보조

두 개의 SimpleAgent가 분업 협력:
- Agent1: GitHub 검색 전문가
- Agent2: 문서 생성 전문가
"""
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool
from dotenv import load_dotenv

# .env 파일의 환경 변수 로드
load_dotenv(dotenv_path="../HelloAgents/.env")

print("="*70)
print("멀티 Agent 협력 지능형 문서 보조")
print("="*70)

# ============================================================
# Agent 1: GitHub 검색 전문가
# ============================================================
print("\n【단계1】GitHub 검색 전문가 생성...")

github_searcher = SimpleAgent(
    name="GitHub 검색 전문가",
    llm=HelloAgentsLLM(),
    system_prompt="""당신은 GitHub 검색 전문가입니다.
당신의 임무는 GitHub 저장소를 검색하고 결과를 반환하는 것입니다.
다음을 포함한 명확하고 구조화된 검색 결과를 반환하세요:
- 저장소 이름
- 간단한 설명

간결하게 유지하고 추가적인 설명은 넣지 마세요."""
)

# GitHub 도구 추가
github_tool = MCPTool(
    name="gh",
    server_command=["npx", "-y", "@modelcontextprotocol/server-github"]
)
github_searcher.add_tool(github_tool)

# ============================================================
# Agent 2: 문서 생성 전문가
# ============================================================
print("\n【단계2】문서 생성 전문가 생성...")

document_writer = SimpleAgent(
    name="문서 생성 전문가",
    llm=HelloAgentsLLM(),
    system_prompt="""당신은 문서 생성 전문가입니다.
당신의 임무는 제공된 정보를 기반으로 구조화된 Markdown 보고서를 생성하는 것입니다.

보고서에는 다음이 포함되어야 합니다:
- 제목
- 소개
- 주요 내용 (항목별 나열, 프로젝트 이름, 설명 등 포함)
- 요약

완전한 Markdown 형식의 보고서 내용을 직접 출력하고, 도구를 사용하여 저장하지 마세요."""
)

# 파일 시스템 도구 추가
fs_tool = MCPTool(
    name="fs",
    server_command=["npx", "-y", "@modelcontextprotocol/server-filesystem", "."]
)
document_writer.add_tool(fs_tool)

# ============================================================
# 작업 실행
# ============================================================
print("\n" + "="*70)
print("작업 실행 시작...")
print("="*70)

try:
    # 단계1: GitHub 검색
    print("\n【단계3】Agent1 GitHub 검색...")
    search_task = "'AI agent' 관련 GitHub 저장소를 검색하여 가장 관련성 높은 상위 5개 결과 반환"
    
    search_results = github_searcher.run(search_task)
    
    print("\n검색 결과:")
    print("-" * 70)
    print(search_results)
    print("-" * 70)
    
    # 단계2: 보고서 생성
    print("\n【단계4】Agent2 보고서 생성...")
    report_task = f"""
다음 GitHub 검색 결과를 기반으로 Markdown 형식의 연구 보고서를 생성하세요:

{search_results}

보고서 요건:
1. 제목: # AI Agent 프레임워크 연구 보고서
2. 소개: AI Agent에 관한 GitHub 프로젝트 조사임을 설명
3. 주요 발견: 발견된 프로젝트와 특징 나열 (이름, 설명 등 포함)
4. 요약: 이 프로젝트들의 공통 특징 요약

완전한 Markdown 형식의 보고서를 직접 출력하세요.
"""

    report_content = document_writer.run(report_task)

    print("\n보고서 내용:")
    print("=" * 70)
    print(report_content)
    print("=" * 70)

    # 단계3: 보고서 저장
    print("\n【단계5】파일에 보고서 저장...")
    import os
    try:
        with open("report.md", "w", encoding="utf-8") as f:
            f.write(report_content)
        print("✅ 보고서가 report.md에 저장됨")

        # 파일 확인
        file_size = os.path.getsize("report.md")
        print(f"✅ 파일 크기: {file_size} 바이트")
    except Exception as e:
        print(f"❌ 저장 실패: {e}")
    
    print("\n" + "="*70)
    print("작업 완료!")
    print("="*70)
    
except Exception as e:
    print(f"\n❌ 오류: {e}")
    import traceback
    traceback.print_exc()
```

`github_searcher`는 이 과정에서 `gh_search_repositories`를 호출하여 GitHub 프로젝트를 검색한다. 얻어진 결과는 `document_writer`의 입력으로 전달되어 보고서 생성을 안내하고, 최종적으로 report.md에 저장된다.

---

**10.2.5 MCP 커뮤니티 생태계**

MCP 프로토콜의 큰 장점 중 하나는 **풍부한 커뮤니티 생태계**다. Anthropic과 커뮤니티 개발자들은 파일 시스템, 데이터베이스, API 서비스 등 다양한 시나리오를 커버하는 대량의 기성 MCP 서버를 이미 만들었다. 이는 도구 어댑터를 처음부터 작성하지 않고도 이 검증된 서버들을 바로 사용할 수 있다는 것을 의미한다.

MCP 커뮤니티의 세 가지 리소스 저장소를 소개한다:

1. **Awesome MCP Servers** (https://github.com/punkpeye/awesome-mcp-servers)
   - 커뮤니티가 관리하는 MCP 서버 선택 목록
   - 다양한 서드파티 서버 포함
   - 기능별로 분류되어 찾기 쉬움

2. **MCP Servers Website** (https://mcpservers.org/)
   - 공식 MCP 서버 디렉토리 웹사이트
   - 검색 및 필터링 기능 제공
   - 사용 설명 및 예시 포함

3. **Official MCP Servers** (https://github.com/modelcontextprotocol/servers)
   - Anthropic 공식 유지 관리 서버
   - 가장 높은 품질, 가장 완성도 높은 문서
   - 일반적으로 사용되는 서비스 구현 포함

표 10.5와 10.6에서 자주 사용되는 공식 MCP 서버와 커뮤니티 인기 MCP 서버를 볼 수 있다:

<div align="center">
  <p>표 10.5 자주 사용되는 공식 MCP 서버</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-5.png" alt="" width="85%"/>
</div>

<div align="center">
  <p>표 10.6 커뮤니티 인기 MCP 서버</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-6.png" alt="" width="85%"/>
</div>

참고할 수 있는 특별히 흥미로운 사례들을 소개한다:

1. **자동화 웹 테스트 (Playwright)**
   
   ```python
   # Agent가 자동으로:
   # - 브라우저를 열고 웹사이트 접속
   # - 폼 작성 및 제출
   # - 스크린샷으로 결과 확인
   # - 테스트 보고서 생성
   playwright_tool = MCPTool(
       name="playwright",
       server_command=["npx", "-y", "@playwright/mcp"]
   )
   ```

2. **지능형 노트 보조 (Obsidian + Perplexity)**
   ```python
   # Agent가:
   # - 최신 기술 정보 검색 (Perplexity)
   # - 구조화된 노트로 정리
   # - Obsidian 지식 베이스에 저장
   # - 노트 간 링크 자동 구축
   ```

3. **프로젝트 관리 자동화 (Jira + GitHub)**
   ```python
   # Agent가:
   # - GitHub Issue에서 Jira 작업 생성
   # - 코드 커밋을 Jira에 동기화
   # - Sprint 진행 상황 자동 업데이트
   # - 프로젝트 보고서 생성
   ```

4. **콘텐츠 창작 워크플로우 (YouTube + Notion + Spotify)**
   ```python
   # Agent가:
   # - YouTube 비디오 자막 가져오기
   # - 콘텐츠 요약 생성
   # - Notion 데이터베이스에 저장
   # - 배경 음악 재생 (Spotify)
   ```

이 절의 내용을 통해 더 많은 MCP 구현 사례를 탐색하길 바란다. HelloAgents에 기고하는 것도 환영한다! 이제 A2A 프로토콜을 배워보자.

---

## 10.3 A2A 프로토콜 실전

A2A(Agent-to-Agent)는 에이전트 간 직접 통신과 협력을 지원하는 프로토콜이다.

**10.3.1 프로토콜 설계 동기**

MCP 프로토콜이 에이전트와 도구 간의 상호작용을 해결한다면, A2A 프로토콜은 에이전트 간의 협력 문제를 해결한다. 연구원, 작성자, 편집자 등 여러 에이전트가 협력해야 하는 작업에서, 이들은 통신하고, 작업을 위임하고, 능력을 협상하고, 상태를 동기화해야 한다.

전통적인 중앙 코디네이터(스타 토폴로지) 방안에는 세 가지 주요 문제가 있다:

- **단일 장애점**: 코디네이터 장애가 전체 시스템 마비를 초래한다.
- **성능 병목**: 모든 통신이 중앙 노드를 거쳐야 하여 동시성이 제한된다.
- **확장 어려움**: 에이전트 추가 또는 수정 시 중앙 로직 변경이 필요하다.

A2A 프로토콜은 점대점(P2P) 아키텍처(메시 토폴로지)를 채택하여, 에이전트 간 직접 통신을 허용함으로써 위의 문제들을 근본적으로 해결한다. 핵심은 **작업(Task)**과 **아티팩트(Artifact)**라는 두 가지 추상 개념으로, 이것이 MCP와의 가장 큰 차이점이다. 표 10.7을 참고한다.

<div align="center">
  <p>표 10.7 A2A 핵심 개념</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-7.png" alt="" width="85%"/>
</div>

협력 과정을 관리하기 위해, A2A는 작업에 대한 표준화된 생명주기를 정의하며, 생성, 협상, 위임, 실행 중, 완료, 실패 등의 상태를 포함한다. 그림 10.7을 참고한다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-7.png" alt="" width="85%"/>
  <p>그림 10.7 A2A 작업 주기</p>
</div>

이 메커니즘을 통해 에이전트는 작업 협상, 진행 상황 추적, 예외 처리를 수행할 수 있다.

A2A 요청 생명주기는 요청이 따르는 네 가지 주요 단계를 상세히 설명하는 시퀀스다: 에이전트 발견, 인증, 메시지 API 전송, 메시지 스트림 API 전송. 그림 10.8은 공식 사이트의 흐름도를 참고하여 작업 흐름을 보여주며, 클라이언트, A2A 서버, 인증 서버 간의 상호작용을 설명한다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-8.png" alt="" width="85%"/>
  <p>그림 10.8 A2A 요청 생명주기</p>
</div>

---

**10.3.2 A2A 프로토콜 실전 사용**

A2A의 현재 구현 대부분은 `Sample Code`이며, Python 구현이 있더라도 상당히 복잡하다. 따라서 여기서는 프로토콜 사상을 시뮬레이션하는 방식만을 채택하며, A2A-SDK를 통해 일부 기능 구현을 상속한다.

**(1) 간단한 A2A 에이전트 생성**

계산기 사례를 시연으로 A2A 에이전트를 생성해보자:

```python
from hello_agents.protocols.a2a.implementation import A2AServer, A2A_AVAILABLE

def create_calculator_agent():
    """계산기 에이전트 생성"""
    if not A2A_AVAILABLE:
        print("❌ A2A SDK가 설치되지 않음. 실행: pip install a2a-sdk")
        return None

    print("🧮 계산기 에이전트 생성")

    # A2A 서버 생성
    calculator = A2AServer(
        name="calculator-agent",
        description="전문 수학 계산 에이전트",
        version="1.0.0",
        capabilities={
            "math": ["addition", "subtraction", "multiplication", "division"],
            "advanced": ["power", "sqrt", "factorial"]
        }
    )

    # 기본 계산 스킬 추가
    @calculator.skill("add")
    def add_numbers(query: str) -> str:
        """덧셈 계산"""
        try:
            # 간단히 "계산 5 + 3" 형식 파싱
            parts = query.replace("계산", "").replace("더하기", "+").replace("더하면", "+")
            if "+" in parts:
                numbers = [float(x.strip()) for x in parts.split("+")]
                result = sum(numbers)
                return f"계산 결과: {' + '.join(map(str, numbers))} = {result}"
            else:
                return "형식을 사용하세요: 계산 5 + 3"
        except Exception as e:
            return f"계산 오류: {e}"

    @calculator.skill("multiply")
    def multiply_numbers(query: str) -> str:
        """곱셈 계산"""
        try:
            parts = query.replace("계산", "").replace("곱하기", "*").replace("×", "*")
            if "*" in parts:
                numbers = [float(x.strip()) for x in parts.split("*")]
                result = 1
                for num in numbers:
                    result *= num
                return f"계산 결과: {' × '.join(map(str, numbers))} = {result}"
            else:
                return "형식을 사용하세요: 계산 5 * 3"
        except Exception as e:
            return f"계산 오류: {e}"

    @calculator.skill("info")
    def get_info(query: str) -> str:
        """에이전트 정보 가져오기"""
        return f"저는 {calculator.name}이며, 기본 수학 계산이 가능합니다. 지원 스킬: {list(calculator.skills.keys())}"

    print(f"✅ 계산기 에이전트 생성 성공, 지원 스킬: {list(calculator.skills.keys())}")
    return calculator

# 에이전트 생성
calc_agent = create_calculator_agent()
if calc_agent:
    # 스킬 테스트
    print("\n🧪 에이전트 스킬 테스트:")
    test_queries = [
        "정보 가져오기",
        "계산 10 + 5",
        "계산 6 * 7"
    ]

    for query in test_queries:
        if "정보" in query:
            result = calc_agent.skills["info"](query)
        elif "+" in query:
            result = calc_agent.skills["add"](query)
        elif "*" in query or "×" in query:
            result = calc_agent.skills["multiply"](query)
        else:
            result = "알 수 없는 쿼리 유형"

        print(f"  📝 쿼리: {query}")
        print(f"  🤖 답변: {result}")
        print()
```

**(2) 커스텀 A2A 에이전트**

자신만의 A2A 에이전트를 생성할 수도 있다. 여기서는 간단한 시연만 제공한다:

```python
from hello_agents.protocols.a2a.implementation import A2AServer, A2A_AVAILABLE

def create_custom_agent():
    """커스텀 에이전트 생성"""
    if not A2A_AVAILABLE:
        print("먼저 A2A SDK를 설치하세요: pip install a2a-sdk")
        return None

    # 에이전트 생성
    agent = A2AServer(
        name="my-custom-agent",
        description="나의 커스텀 에이전트",
        capabilities={"custom": ["skill1", "skill2"]}
    )

    # 스킬 추가
    @agent.skill("greet")
    def greet_user(name: str) -> str:
        """사용자 인사"""
        return f"안녕하세요, {name}! 저는 커스텀 에이전트입니다."

    @agent.skill("calculate")
    def simple_calculate(expression: str) -> str:
        """간단한 계산"""
        try:
            # 안전한 계산 (기본 연산만 지원)
            allowed_chars = set('0123456789+-*/(). ')
            if all(c in allowed_chars for c in expression):
                result = eval(expression)
                return f"계산 결과: {expression} = {result}"
            else:
                return "오류: 기본 수학 연산만 지원"
        except Exception as e:
            return f"계산 오류: {e}"

    return agent

# 커스텀 에이전트 생성 및 테스트
custom_agent = create_custom_agent()
if custom_agent:
    # 스킬 테스트
    print("인사 스킬 테스트:")
    result1 = custom_agent.skills["greet"]("홍길동")
    print(result1)

    print("\n계산 스킬 테스트:")
    result2 = custom_agent.skills["calculate"]("10 + 5 * 2")
    print(result2)
```

---

**10.3.3 HelloAgents A2A 도구 사용**

HelloAgents는 통일된 A2A 도구 인터페이스를 제공한다.

**(1) A2A Agent 서버 생성**

먼저 Agent 서버를 생성해보자:

```python
from hello_agents.protocols import A2AServer
import threading
import time

# 연구원 Agent 서비스 생성
researcher = A2AServer(
    name="researcher",
    description="자료 검색 및 분석을 담당하는 Agent",
    version="1.0.0"
)

# 스킬 정의
@researcher.skill("research")
def handle_research(text: str) -> str:
    """연구 요청 처리"""
    import re
    match = re.search(r'research\s+(.+)', text, re.IGNORECASE)
    topic = match.group(1).strip() if match else text
    
    # 실제 연구 로직 (여기서는 단순화)
    result = {
        "topic": topic,
        "findings": f"{topic}에 관한 연구 결과...",
        "sources": ["출처1", "출처2", "출처3"]
    }
    return str(result)

# 백그라운드에서 서비스 시작
def start_server():
    researcher.run(host="localhost", port=5000)

if __name__ == "__main__":
    server_thread = threading.Thread(target=start_server, daemon=True)
    server_thread.start()
    
    print("✅ 연구원 Agent 서비스가 http://localhost:5000 에서 시작됨")
    
    # 프로그램 계속 실행
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print("\n서비스 종료됨")
```

**(2) A2A Agent 클라이언트 생성**

이제 서버와 통신하는 클라이언트를 생성해보자:

```python
from hello_agents.protocols import A2AClient

# 연구원 Agent에 연결하는 클라이언트 생성
client = A2AClient("http://localhost:5000")

# 연구 요청 전송
response = client.execute_skill("research", "research AI의 의료 분야 응용")
print(f"수신한 응답: {response.get('result')}")

# 출력:
# 수신한 응답: {'topic': 'AI의 의료 분야 응용', 'findings': 'AI의 의료 분야 응용에 관한 연구 결과...', 'sources': ['출처1', '출처2', '출처3']}
```

**(3) Agent 네트워크 생성**

여러 Agent의 협력을 위해 여러 Agent를 서로 연결할 수 있다:

```python
from hello_agents.protocols import A2AServer, A2AClient
import threading
import time

# 1. 여러 Agent 서비스 생성
researcher = A2AServer(
    name="researcher",
    description="연구원"
)

@researcher.skill("research")
def do_research(text: str) -> str:
    import re
    match = re.search(r'research\s+(.+)', text, re.IGNORECASE)
    topic = match.group(1).strip() if match else text
    return str({"topic": topic, "findings": f"{topic}의 연구 결과"})

writer = A2AServer(
    name="writer",
    description="작성자"
)

@writer.skill("write")
def write_article(text: str) -> str:
    import re
    match = re.search(r'write\s+(.+)', text, re.IGNORECASE)
    content = match.group(1).strip() if match else text
    
    # 연구 데이터 파싱 시도
    try:
        data = eval(content)
        topic = data.get("topic", "알 수 없는 주제")
        findings = data.get("findings", "연구 결과 없음")
    except:
        topic = "알 수 없는 주제"
        findings = content
    
    return f"# {topic}\n\n연구 기반: {findings}\n\n기사 내용..."

editor = A2AServer(
    name="editor",
    description="편집자"
)

@editor.skill("edit")
def edit_article(text: str) -> str:
    import re
    match = re.search(r'edit\s+(.+)', text, re.IGNORECASE)
    article = match.group(1).strip() if match else text
    
    result = {
        "article": article + "\n\n[편집 최적화됨]",
        "feedback": "기사 품질 우수",
        "approved": True
    }
    return str(result)

# 2. 모든 서비스 시작
threading.Thread(target=lambda: researcher.run(port=5000), daemon=True).start()
threading.Thread(target=lambda: writer.run(port=5001), daemon=True).start()
threading.Thread(target=lambda: editor.run(port=5002), daemon=True).start()
time.sleep(2)  # 서비스 시작 대기

# 3. 각 Agent에 연결하는 클라이언트 생성
researcher_client = A2AClient("http://localhost:5000")
writer_client = A2AClient("http://localhost:5001")
editor_client = A2AClient("http://localhost:5002")

# 4. 협력 흐름
def create_content(topic):
    # 단계1: 연구
    research = researcher_client.execute_skill("research", f"research {topic}")
    research_data = research.get('result', '')
    
    # 단계2: 작성
    article = writer_client.execute_skill("write", f"write {research_data}")
    article_content = article.get('result', '')
    
    # 단계3: 편집
    final = editor_client.execute_skill("edit", f"edit {article_content}")
    return final.get('result', '')

# 사용
result = create_content("AI의 의료 분야 응용")
print(f"\n최종 결과:\n{result}")
```

---

**10.3.4 에이전트에서 A2A 도구 사용**

이제 HelloAgents 에이전트에 A2A를 통합하는 방법을 살펴보자.

**(1) A2ATool 래퍼 사용**

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import A2ATool
from dotenv import load_dotenv

load_dotenv()
llm = HelloAgentsLLM()

# http://localhost:5000 에서 연구원 Agent 서비스가 이미 실행 중이라고 가정

# 코디네이터 Agent 생성
coordinator = SimpleAgent(name="코디네이터", llm=llm)

# A2A 도구 추가, 연구원 Agent에 연결
researcher_tool = A2ATool(
    name="researcher",
    description="연구원 Agent, 자료 검색 및 분석 가능",
    agent_url="http://localhost:5000"
)
coordinator.add_tool(researcher_tool)

# 코디네이터가 연구원 Agent를 호출 가능
response = coordinator.run("연구원에게 AI의 교육 분야 응용을 연구해달라고 부탁해줘")
print(response)
```

**(2) 실전 사례: 지능형 고객 서비스 시스템**

세 개의 Agent를 포함하는 완전한 지능형 고객 서비스 시스템을 구축해보자:
- **접수원**: 고객 문제 유형 분석
- **기술 전문가**: 기술 문제 답변
- **영업 상담사**: 영업 문제 답변

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import A2ATool
from hello_agents.protocols import A2AServer
import threading
import time
from dotenv import load_dotenv

load_dotenv()
llm = HelloAgentsLLM()

# 1. 기술 전문가 Agent 서비스 생성
tech_expert = A2AServer(
    name="tech_expert",
    description="기술 전문가, 기술 문제 답변"
)

@tech_expert.skill("answer")
def answer_tech_question(text: str) -> str:
    import re
    match = re.search(r'answer\s+(.+)', text, re.IGNORECASE)
    question = match.group(1).strip() if match else text
    # 실제 응용에서는 LLM이나 지식 베이스를 호출
    return f"기술 답변: '{question}'에 대해, 기술 문서를 참고하시기 바랍니다..."

# 2. 영업 상담사 Agent 서비스 생성
sales_advisor = A2AServer(
    name="sales_advisor",
    description="영업 상담사, 영업 문제 답변"
)

@sales_advisor.skill("answer")
def answer_sales_question(text: str) -> str:
    import re
    match = re.search(r'answer\s+(.+)', text, re.IGNORECASE)
    question = match.group(1).strip() if match else text
    return f"영업 답변: '{question}'에 대해, 특별 할인 혜택이 있습니다..."

# 3. 서비스 시작
threading.Thread(target=lambda: tech_expert.run(port=6000), daemon=True).start()
threading.Thread(target=lambda: sales_advisor.run(port=6001), daemon=True).start()
time.sleep(2)

# 4. 접수원 Agent 생성 (HelloAgents의 SimpleAgent 사용)
receptionist = SimpleAgent(
    name="접수원",
    llm=llm,
    system_prompt="""당신은 고객 서비스 접수원으로, 다음을 담당합니다:
1. 고객 문제 유형 분석 (기술 문제 or 영업 문제)
2. 해당 전문가에게 문제 전달
3. 전문가 답변 정리 후 고객에게 반환

예의 바르고 전문적인 태도를 유지하세요."""
)

# 기술 전문가 도구 추가
tech_tool = A2ATool(
    agent_url="http://localhost:6000",
    name="tech_expert",
    description="기술 전문가, 기술 관련 문제 답변"
)
receptionist.add_tool(tech_tool)

# 영업 상담사 도구 추가
sales_tool = A2ATool(
    agent_url="http://localhost:6001",
    name="sales_advisor",
    description="영업 상담사, 가격, 구매 관련 문제 답변"
)
receptionist.add_tool(sales_tool)

# 5. 고객 문의 처리
def handle_customer_query(query):
    print(f"\n고객 문의: {query}")
    print("=" * 50)
    response = receptionist.run(query)
    print(f"\n고객 서비스 답변: {response}")
    print("=" * 50)

# 다양한 유형의 문제 테스트
if __name__ == "__main__":
    handle_customer_query("API를 어떻게 호출하나요?")
    handle_customer_query("기업용 버전 가격이 얼마인가요?")
    handle_customer_query("Python 프로젝트에 어떻게 통합하나요?")
```

**(3) 고급 사용법: Agent 간 협상**

A2A 프로토콜은 Agent 간 협상 메커니즘도 지원한다:

```python
from hello_agents.protocols import A2AServer, A2AClient
import threading
import time

# 협상이 필요한 두 Agent 생성
agent1 = A2AServer(
    name="agent1",
    description="Agent 1"
)

@agent1.skill("propose")
def handle_proposal(text: str) -> str:
    """협상 제안 처리"""
    import re
    
    # 제안 파싱
    match = re.search(r'propose\s+(.+)', text, re.IGNORECASE)
    proposal_str = match.group(1).strip() if match else text
    
    try:
        proposal = eval(proposal_str)
        task = proposal.get("task")
        deadline = proposal.get("deadline")
        
        # 제안 평가
        if deadline >= 7:  # 최소 7일 필요
            result = {"accepted": True, "message": "제안 수락"}
        else:
            result = {
                "accepted": False,
                "message": "시간이 너무 촉박함",
                "counter_proposal": {"deadline": 7}
            }
        return str(result)
    except:
        return str({"accepted": False, "message": "잘못된 제안 형식"})

agent2 = A2AServer(
    name="agent2",
    description="Agent 2"
)

@agent2.skill("negotiate")
def negotiate_task(text: str) -> str:
    """협상 시작"""
    import re
    
    # 작업과 마감일 파싱
    match = re.search(r'negotiate\s+task:(.+?)\s+deadline:(\d+)', text, re.IGNORECASE)
    if match:
        task = match.group(1).strip()
        deadline = int(match.group(2))
        
        # agent1에 제안 전송
        proposal = {"task": task, "deadline": deadline}
        return str({"status": "negotiating", "proposal": proposal})
    else:
        return str({"status": "error", "message": "잘못된 협상 요청"})

# 서비스 시작
threading.Thread(target=lambda: agent1.run(port=7000), daemon=True).start()
threading.Thread(target=lambda: agent2.run(port=7001), daemon=True).start()
```

---

## 10.4 ANP 프로토콜 실전

MCP 프로토콜이 도구 호출을, A2A 프로토콜이 점대점 에이전트 협력을 해결한 후, ANP 프로토콜은 대규모 개방형 네트워크 환경에서의 에이전트 관리 문제에 집중한다.

10.2절과 10.3절에서 MCP(도구 접근)와 A2A(에이전트 협력)를 배웠다. 이제 **대규모 개방형 에이전트 네트워크 구축**에 집중하는 ANP(Agent Network Protocol)를 배워보자.

**10.4.1 프로토콜 목표**

네트워크에 기능이 각각 다른 대량의 에이전트(예: 자연어 처리, 이미지 인식, 데이터 분석 등)가 존재할 때, 시스템은 일련의 과제에 직면한다:

- **서비스 발견**: 새로운 작업이 도착했을 때, 해당 작업을 처리할 수 있는 에이전트를 어떻게 빠르게 찾는가?
- **지능형 라우팅**: 여러 에이전트가 동일한 작업을 처리할 수 있다면, 어떻게 가장 적합한 것을 선택하여 (부하, 비용 등에 따라) 작업을 할당하는가?
- **동적 확장**: 네트워크에 새로 참여하는 에이전트가 다른 구성원에게 발견되고 호출될 수 있게 하려면 어떻게 해야 하는가?

ANP의 설계 목표는 바로 위의 서비스 발견, 라우팅 선택, 네트워크 확장성 문제를 해결하는 표준화된 메커니즘을 제공하는 것이다.

ANP는 설계 목표를 실현하기 위해 다음과 같은 핵심 개념들을 정의한다. 표 10.8을 참고한다:

<div align="center">
  <p>표 10.8 ANP 핵심 개념</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-table-8.png" alt="" width="85%"/>
</div>

ANP의 아키텍처 설계는 공식 [입문 가이드](https://github.com/agent-network-protocol/AgentNetworkProtocol/blob/main/docs/chinese/ANP入门指南.md)를 참고한다. 그림 10.9와 같다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-9.png" alt="" width="85%"/>
  <p>그림 10.9 ANP 전체 흐름</p>
</div>

이 흐름도에는 다음 네 가지 주요 단계가 포함된다:

**1. 서비스 발견과 매칭:** 에이전트 A는 공개된 발견 서비스를 통해, 의미론적 또는 기능 설명을 기반으로 쿼리를 수행하여, 작업 요구사항에 부합하는 에이전트 B를 탐색한다. 이 발견 서비스는 각 에이전트가 외부에 노출하는 표준 엔드포인트(`.well-known/agent-descriptions`)를 사전에 크롤링하여 인덱스를 구축함으로써, 서비스 수요자와 제공자의 동적 매칭을 실현한다.

**2. DID 기반 신원 인증:** 상호작용 시작 시, 에이전트 A는 자신의 DID를 포함한 요청에 개인 키로 서명한다. 에이전트 B는 이를 수신한 후, 해당 DID를 파싱하여 공개 키를 얻고, 이를 통해 서명의 진정성과 요청의 무결성을 확인하여 양측 간 신뢰 통신을 수립한다.

**3. 표준화된 서비스 실행:** 신원 인증 통과 후, 에이전트 B가 요청에 응답하며, 양측이 사전 정의된 표준 인터페이스와 데이터 형식에 따라 데이터 교환 또는 서비스 호출(예약, 조회 등)을 수행한다. 표준화된 상호작용 흐름은 크로스 플랫폼, 크로스 시스템 상호운용성 실현의 기초다.

종합적으로, 이 메커니즘의 핵심은 DID를 활용하여 탈중앙화된 신뢰 기반을 구축하고, 표준화된 기술 프로토콜을 통해 서비스의 동적 발견을 실현하는 것이다. 이 방법은 에이전트가 중앙 조율 없이도 인터넷에서 안전하고 효율적으로 협력 네트워크를 형성할 수 있게 한다.

---

**10.4.2 ANP 서비스 발견 사용**

**(1) 서비스 발견 센터 생성**

```python
from hello_agents.protocols import ANPDiscovery, register_service

# 서비스 발견 센터 생성
discovery = ANPDiscovery()

# Agent 서비스 등록
register_service(
    discovery=discovery,
    service_id="nlp_agent_1",
    service_name="NLP 처리 전문가 A",
    service_type="nlp",
    capabilities=["text_analysis", "sentiment_analysis", "ner"],
    endpoint="http://localhost:8001",
    metadata={"load": 0.3, "price": 0.01, "version": "1.0.0"}
)

register_service(
    discovery=discovery,
    service_id="nlp_agent_2",
    service_name="NLP 처리 전문가 B",
    service_type="nlp",
    capabilities=["text_analysis", "translation"],
    endpoint="http://localhost:8002",
    metadata={"load": 0.7, "price": 0.02, "version": "1.1.0"}
)

print("✅ 서비스 등록 완료")
```

**(2) 서비스 발견**

```python
from hello_agents.protocols import discover_service

# 유형별 검색
nlp_services = discover_service(discovery, service_type="nlp")
print(f"{len(nlp_services)}개의 NLP 서비스 발견")

# 가장 낮은 부하의 서비스 선택
best_service = min(nlp_services, key=lambda s: s.metadata.get("load", 1.0))
print(f"최적 서비스: {best_service.service_name} (부하: {best_service.metadata['load']})")
```

**(3) Agent 네트워크 구축**

```python
from hello_agents.protocols import ANPNetwork

# 네트워크 생성
network = ANPNetwork(network_id="ai_cluster")

# 노드 추가
for service in discovery.list_all_services():
    network.add_node(service.service_id, service.endpoint)

# 연결 수립 (능력 매칭 기반)
network.connect_nodes("nlp_agent_1", "nlp_agent_2")

stats = network.get_network_stats()
print(f"✅ 네트워크 구축 완료, 총 {stats['total_nodes']}개 노드")
```

---

**10.4.3 실전 사례**

완전한 분산 작업 스케줄링 시스템을 구축해보자:

```python
from hello_agents.protocols import ANPDiscovery, register_service
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools.builtin import ANPTool
import random
from dotenv import load_dotenv

load_dotenv()
llm = HelloAgentsLLM()

# 1. 서비스 발견 센터 생성
discovery = ANPDiscovery()

# 2. 여러 계산 노드 등록
for i in range(10):
    register_service(
        discovery=discovery,
        service_id=f"compute_node_{i}",
        service_name=f"계산 노드{i}",
        service_type="compute",
        capabilities=["data_processing", "ml_training"],
        endpoint=f"http://node{i}:8000",
        metadata={
            "load": random.uniform(0.1, 0.9),
            "cpu_cores": random.choice([4, 8, 16]),
            "memory_gb": random.choice([16, 32, 64]),
            "gpu": random.choice([True, False])
        }
    )

print(f"✅ {len(discovery.list_all_services())}개의 계산 노드 등록됨")

# 3. 작업 스케줄러 Agent 생성
scheduler = SimpleAgent(
    name="작업 스케줄러",
    llm=llm,
    system_prompt="""당신은 지능형 작업 스케줄러로, 다음을 담당합니다:
1. 작업 요구사항 분석
2. 가장 적합한 계산 노드 선택
3. 작업 할당

노드 선택 시 고려 요소: 부하, CPU 코어 수, 메모리, GPU 등."""
)

# ANP 도구 추가
anp_tool = ANPTool(
    name="service_discovery",
    description="서비스 발견 도구, 계산 노드 검색 및 선택 가능",
    discovery=discovery
)
scheduler.add_tool(anp_tool)

# 4. 지능형 작업 할당
def assign_task(task_description):
    print(f"\n작업: {task_description}")
    print("=" * 50)

    # Agent가 지능적으로 노드 선택
    response = scheduler.run(f"""
    다음 작업에 가장 적합한 계산 노드를 선택하세요:
    {task_description}

    요구사항:
    1. 모든 사용 가능한 노드 나열
    2. 각 노드의 특성 분석
    3. 가장 적합한 노드 선택
    4. 선택 이유 설명
    """)

    print(response)
    print("=" * 50)

# 다양한 유형의 작업 테스트
assign_task("GPU 지원이 필요한 대형 딥러닝 모델 훈련")
assign_task("대용량 텍스트 데이터 처리, 높은 메모리 필요")
assign_task("경량 데이터 분석 작업 실행")
```

부하 분산 예시:

```python
from hello_agents.protocols import ANPDiscovery, register_service
import random

# 서비스 발견 센터 생성
discovery = ANPDiscovery()

# 같은 유형의 여러 서비스 등록
for i in range(5):
    register_service(
        discovery=discovery,
        service_id=f"api_server_{i}",
        service_name=f"API 서버{i}",
        service_type="api",
        capabilities=["rest_api"],
        endpoint=f"http://api{i}:8000",
        metadata={"load": random.uniform(0.1, 0.9)}
    )

# 부하 분산 함수
def get_best_server():
    """가장 낮은 부하의 서버 선택"""
    servers = discovery.discover_services(service_type="api")
    if not servers:
        return None

    best = min(servers, key=lambda s: s.metadata.get("load", 1.0))
    return best

# 요청 할당 시뮬레이션
for i in range(10):
    server = get_best_server()
    print(f"요청 {i+1} -> {server.service_name} (부하: {server.metadata['load']:.2f})")

    # 부하 업데이트 (시뮬레이션)
    server.metadata["load"] += 0.1
```

---

## 10.5 커스텀 MCP 서버 구축

앞의 장들에서 기존 MCP 서비스를 사용하는 방법을 배웠고, 서로 다른 프로토콜의 특성도 이해했다. 이제 자신만의 MCP 서버를 구축하는 방법을 배워보자.

**10.5.1 첫 번째 MCP 서버 만들기**

**(1) 왜 커스텀 MCP 서버를 구축하는가?**

공개된 MCP 서비스를 직접 사용할 수도 있지만, 많은 실제 응용 시나리오에서 특정 요구사항을 충족하기 위해 커스텀 MCP 서버를 구축해야 한다.

주요 동기는 다음과 같다:

- **비즈니스 로직 캡슐화**: 기업 내부의 특유한 비즈니스 흐름 또는 복잡한 작업을 표준화된 MCP 도구로 캡슐화하여 에이전트가 통일적으로 호출할 수 있게 한다.
- **개인 데이터 접근**: 내부 데이터베이스, API 또는 공개망에 노출할 수 없는 개인 데이터 소스에 접근하기 위한 안전하고 제어 가능한 인터페이스 또는 프록시를 생성한다.
- **성능 전문 최적화**: 고빈도 호출 또는 응답 지연에 엄격한 요구가 있는 응용 시나리오에 대해 심층 최적화를 수행한다.
- **기능 커스텀 확장**: 표준 MCP 서비스가 제공하지 않는 특정 기능을 구현한다. 예를 들어 독점 알고리즘 모델 통합 또는 특정 하드웨어 장치 연결.

**(2) 교육 사례: 날씨 조회 MCP 서버**

간단한 날씨 조회 서버부터 시작하여 MCP 서버 개발을 단계적으로 배워보자:

```python
#!/usr/bin/env python3
"""날씨 조회 MCP 서버"""

import json
import requests
import os
from datetime import datetime
from typing import Dict, Any
from hello_agents.protocols import MCPServer

# MCP 서버 생성
weather_server = MCPServer(name="weather-server", description="실시간 날씨 조회 서비스")

CITY_MAP = {
    "서울": "Seoul", "부산": "Busan", "인천": "Incheon",
    "대구": "Daegu", "대전": "Daejeon", "광주": "Gwangju",
    "수원": "Suwon", "울산": "Ulsan", "창원": "Changwon",
    "성남": "Seongnam", "제주": "Jeju", "청주": "Cheongju"
}


def get_weather_data(city: str) -> Dict[str, Any]:
    """wttr.in에서 날씨 데이터 가져오기"""
    city_en = CITY_MAP.get(city, city)
    url = f"https://wttr.in/{city_en}?format=j1"
    response = requests.get(url, timeout=10)
    response.raise_for_status()
    data = response.json()
    current = data["current_condition"][0]

    return {
        "city": city,
        "temperature": float(current["temp_C"]),
        "feels_like": float(current["FeelsLikeC"]),
        "humidity": int(current["humidity"]),
        "condition": current["weatherDesc"][0]["value"],
        "wind_speed": round(float(current["windspeedKmph"]) / 3.6, 1),
        "visibility": float(current["visibility"]),
        "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    }


# 도구 함수 정의
def get_weather(city: str) -> str:
    """지정된 도시의 현재 날씨 가져오기"""
    try:
        weather_data = get_weather_data(city)
        return json.dumps(weather_data, ensure_ascii=False, indent=2)
    except Exception as e:
        return json.dumps({"error": str(e), "city": city}, ensure_ascii=False)


def list_supported_cities() -> str:
    """지원되는 모든 도시 목록"""
    result = {"cities": list(CITY_MAP.keys()), "count": len(CITY_MAP)}
    return json.dumps(result, ensure_ascii=False, indent=2)


def get_server_info() -> str:
    """서버 정보 가져오기"""
    info = {
        "name": "Weather MCP Server",
        "version": "1.0.0",
        "tools": ["get_weather", "list_supported_cities", "get_server_info"]
    }
    return json.dumps(info, ensure_ascii=False, indent=2)


# 서버에 도구 등록
weather_server.add_tool(get_weather)
weather_server.add_tool(list_supported_cities)
weather_server.add_tool(get_server_info)


if __name__ == "__main__":
    weather_server.run()
```

**(3) 커스텀 MCP 서버 테스트**

테스트 스크립트를 생성한다:

```python
#!/usr/bin/env python3
"""날씨 조회 MCP 서버 테스트"""

import asyncio
import json
import sys
import os

sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..', 'HelloAgents'))
from hello_agents.protocols.mcp.client import MCPClient


async def test_weather_server():
    server_script = os.path.join(os.path.dirname(__file__), "14_weather_mcp_server.py")
    client = MCPClient(["python", server_script])

    try:
        async with client:
            # 테스트1: 서버 정보 가져오기
            info = json.loads(await client.call_tool("get_server_info", {}))
            print(f"서버: {info['name']} v{info['version']}")

            # 테스트2: 지원 도시 목록
            cities = json.loads(await client.call_tool("list_supported_cities", {}))
            print(f"지원 도시: {cities['count']}개")

            # 테스트3: 서울 날씨 조회
            weather = json.loads(await client.call_tool("get_weather", {"city": "서울"}))
            if "error" not in weather:
                print(f"\n서울 날씨: {weather['temperature']}°C, {weather['condition']}")

            # 테스트4: 부산 날씨 조회
            weather = json.loads(await client.call_tool("get_weather", {"city": "부산"}))
            if "error" not in weather:
                print(f"부산 날씨: {weather['temperature']}°C, {weather['condition']}")

            print("\n✅ 모든 테스트 완료!")

    except Exception as e:
        print(f"❌ 테스트 실패: {e}")


if __name__ == "__main__":
    asyncio.run(test_weather_server())
```

**(4) Agent에서 커스텀 MCP 서버 사용**

```python
"""Agent에서 날씨 MCP 서버 사용"""

import os
from dotenv import load_dotenv
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools import MCPTool

load_dotenv()


def create_weather_assistant():
    """날씨 보조 생성"""
    llm = HelloAgentsLLM()

    assistant = SimpleAgent(
        name="날씨 보조",
        llm=llm,
        system_prompt="""당신은 날씨 보조로, 도시 날씨를 조회할 수 있습니다.
get_weather 도구를 사용하여 날씨를 조회하며, 한국어 도시 이름을 지원합니다.
"""
    )

    # 날씨 MCP 도구 추가
    server_script = os.path.join(os.path.dirname(__file__), "14_weather_mcp_server.py")
    weather_tool = MCPTool(server_command=["python", server_script])
    assistant.add_tool(weather_tool)

    return assistant


def demo():
    """시연"""
    assistant = create_weather_assistant()

    print("\n서울 날씨 조회:")
    response = assistant.run("서울 오늘 날씨는 어때요?")
    print(f"답변: {response}\n")


def interactive():
    """대화 모드"""
    assistant = create_weather_assistant()

    while True:
        user_input = input("\n당신: ").strip()
        if user_input.lower() in ['quit', 'exit']:
            break
        response = assistant.run(user_input)
        print(f"보조: {response}")


if __name__ == "__main__":
    import sys
    if len(sys.argv) > 1 and sys.argv[1] == "demo":
        demo()
    else:
        interactive()
```

```
🔗 MCP 서버에 연결 중...
✅ 연결 성공!
🔌 연결 해제됨
✅ 도구 'mcp_get_weather' 등록됨.
✅ 도구 'mcp_list_supported_cities' 등록됨.
✅ 도구 'mcp_get_server_info' 등록됨.
✅ MCP 도구 'mcp'가 3개의 독립 도구로 전개됨

당신: 서울 날씨를 조회하고 싶어요
🔗 MCP 서버에 연결 중...
✅ 연결 성공!
🔌 연결 해제됨
보조: 현재 서울의 날씨 상황은 다음과 같습니다:

- 온도: 10.0°C
- 체감 온도: 9.0°C
- 습도: 94%
- 날씨 상태: 이슬비
- 풍속: 1.7m/s
- 가시거리: 10.0km
- 타임스탬프: 2025년 10월 9일 13:46:40

우산을 챙기시고, 날씨 변화에 따라 옷차림을 적절히 조절하세요.
```

---

**10.5.2 MCP 서버 업로드**

실제 날씨 조회 MCP 서버를 만들었다. 이제 Smithery 플랫폼에 게시하여 전 세계 개발자들이 사용할 수 있게 해보자.

**(1) Smithery란?**

[Smithery](https://smithery.ai/)는 MCP 서버의 공식 게시 플랫폼으로, Python의 PyPI나 Node.js의 npm과 유사하다. Smithery를 통해 사용자는 다음을 할 수 있다:

- 🔍 MCP 서버 발견 및 검색
- 📦 원클릭 MCP 서버 설치
- 📊 서버 사용 통계 및 평가 확인
- 🔄 서버 업데이트 자동 수신

**(2) 게시 준비**

먼저 프로젝트를 표준 게시 형식으로 정리해야 한다. 이 폴더는 이미 `code` 디렉토리에 준비되어 있으니 참고할 수 있다:

```
weather-mcp-server/
├── README.md           # 프로젝트 설명 문서
├── LICENSE            # 오픈소스 라이선스
├── Dockerfile         # Docker 빌드 구성 (권장)
├── pyproject.toml     # Python 프로젝트 구성 (필수)
├── requirements.txt   # Python 의존성
├── smithery.yaml      # Smithery 구성 파일 (필수)
└── server.py          # MCP 서버 주 파일
```

`smithery.yaml`은 Smithery 플랫폼의 구성 파일이다:

```yaml
name: weather-mcp-server
displayName: Weather MCP Server
description: Real-time weather query MCP server based on HelloAgents framework
version: 1.0.0
author: HelloAgents Team
homepage: https://github.com/yourusername/weather-mcp-server
license: MIT
categories:
  - weather
  - data
tags:
  - weather
  - real-time
  - helloagents
  - wttr
runtime: container
build:
  dockerfile: Dockerfile
  dockerBuildPath: .
startCommand:
  type: http
tools:
  - name: get_weather
    description: Get current weather for a city
  - name: list_supported_cities
    description: List all supported cities
  - name: get_server_info
    description: Get server information
```

구성 설명:

- `name`: 서버의 고유 식별자 (소문자, 하이픈 구분)
- `displayName`: 표시 이름
- `description`: 간단한 설명
- `version`: 버전 번호 (시맨틱 버저닝 준수)
- `runtime`: 런타임 환경 (python/node)
- `entrypoint`: 진입 파일
- `tools`: 도구 목록

`pyproject.toml`은 Python 프로젝트의 표준 구성 파일이며, Smithery는 이 파일이 반드시 포함될 것을 요구한다. 이후 패키지로 빌드되기 때문이다:

```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "weather-mcp-server"
version = "1.0.0"
description = "Real-time weather query MCP server based on HelloAgents framework"
readme = "README.md"
license = {text = "MIT"}
authors = [
    {name = "HelloAgents Team", email = "xxx"}
]
requires-python = ">=3.10"
dependencies = [
    "hello-agents>=0.2.1",
    "requests>=2.31.0",
]

[project.urls]
Homepage = "https://github.com/yourusername/weather-mcp-server"
Repository = "https://github.com/yourusername/weather-mcp-server"
"Bug Tracker" = "https://github.com/yourusername/weather-mcp-server/issues"

[tool.setuptools]
py-modules = ["server"]
```

구성 설명:

- `[build-system]`: 빌드 도구 지정 (setuptools)
- `[project]`: 프로젝트 메타데이터
  - `name`: 프로젝트 이름
  - `version`: 버전 번호 (시맨틱 버저닝 준수)
  - `dependencies`: 프로젝트 의존성 목록
  - `requires-python`: Python 버전 요구사항
- `[project.urls]`: 프로젝트 관련 링크
- `[tool.setuptools]`: setuptools 구성

Smithery가 자동으로 Dockerfile을 생성하지만, 커스텀 Dockerfile을 제공하면 배포 성공을 보장할 수 있다:

```dockerfile
# weather-mcp-server 멀티 스테이지 빌드
FROM python:3.12-slim-bookworm as base

# 작업 디렉토리 설정
WORKDIR /app

# 시스템 의존성 설치
RUN apt-get update && apt-get install -y \
    --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

# 프로젝트 파일 복사
COPY pyproject.toml requirements.txt ./
COPY server.py ./

# Python 의존성 설치
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# 환경 변수 설정
ENV PYTHONUNBUFFERED=1
ENV PORT=8081

# 포트 노출 (Smithery는 8081 사용)
EXPOSE 8081

# 헬스 체크
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD python -c "import sys; sys.exit(0)"

# MCP 서버 실행
CMD ["python", "server.py"]
```

Dockerfile 구성 설명:

- **기본 이미지**: `python:3.12-slim-bookworm` - 경량 Python 이미지
- **작업 디렉토리**: `/app` - 애플리케이션 루트 디렉토리
- **포트**: `8081` - Smithery 플랫폼 표준 포트
- **시작 명령**: `python server.py` - MCP 서버 실행

`hello-agents` 저장소를 Fork하여 `code`의 소스 코드를 얻고, 자신의 GitHub으로 `weather-mcp-server`라는 저장소를 생성하여 `yourusername`을 자신의 GitHub Username으로 변경한다.

**(3) Smithery에 제출**

브라우저를 열고 [https://smithery.ai/](https://smithery.ai/)에 접속하여 GitHub 계정으로 로그인한다. 페이지의 "Publish Server" 버튼을 클릭하고, GitHub 저장소 URL `https://github.com/yourusername/weather-mcp-server`를 입력하면 게시를 기다릴 수 있다.

게시 완료 후 그림 10.10과 유사한 페이지를 볼 수 있다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-10.png" alt="" width="85%"/>
  <p>그림 10.10 Smithery 게시 성공 페이지</p>
</div>

서버가 성공적으로 게시되면 사용자들이 다음 방식으로 사용할 수 있다:

방식 1: Smithery CLI를 통해

```bash
# Smithery CLI 설치
npm install -g @smithery/cli

# 서버 설치
smithery install weather-mcp-server
```

방식 2: Claude Desktop에서 구성

```json
{
  "mcpServers": {
    "weather": {
      "command": "smithery",
      "args": ["run", "weather-mcp-server"]
    }
  }
}
```

방식 3: HelloAgents에서 사용

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.tools.builtin.protocol_tools import MCPTool

agent = SimpleAgent(name="날씨 보조", llm=HelloAgentsLLM())

# Smithery에 설치된 서버 사용
weather_tool = MCPTool(
    server_command=["smithery", "run", "weather-mcp-server"]
)
agent.add_tool(weather_tool)

response = agent.run("서울 오늘 날씨는 어때요?")
```

물론 이것은 예시일 뿐이며, 더 많은 사용법은 직접 탐색할 수 있다. 그림 10.11은 MCP 도구가 성공적으로 게시되었을 때 포함되는 정보를 보여준다. 서비스 이름 "날씨", 고유 식별자 `@jjyaoao/weather-mcp-server`, 상태 정보가 표시된다. Tools 영역은 방금 구현한 메서드들이고, Connect 영역은 이 서비스에 연결하고 사용하는 데 필요한 기술적 정보를 제공하며, **접속 URL 주소**와 다양한 언어/환경에서의 **구성 코드 스니펫**을 포함한다. 더 깊이 알고 싶다면 이 [링크](https://smithery.ai/server/@jjyaoao/weather-mcp-server)를 클릭하면 된다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/10-figures/10-11.png" alt="" width="85%"/>
  <p>그림 10.11 Smithery에 성공적으로 게시된 MCP 도구</p>
</div>

이제 자신만의 MCP 서버를 만들어볼 차례다!

---

## 10.6 본 장 요약

본 장에서는 에이전트 통신의 세 가지 핵심 프로토콜인 MCP, A2A, ANP를 체계적으로 소개하고, 이들의 설계 이념, 응용 시나리오, 실천 방법을 탐구했다.

**프로토콜 포지셔닝:**

- **MCP (Model Context Protocol)**: 에이전트와 도구 간의 다리로, 통일된 도구 접근 인터페이스를 제공하며, 단일 에이전트의 능력 강화에 적합하다.
- **A2A (Agent-to-Agent Protocol)**: 에이전트 간의 대화 시스템으로, 직접 통신과 작업 협상을 지원하며, 소규모 팀의 긴밀한 협력에 적합하다.
- **ANP (Agent Network Protocol)**: 에이전트의 "인터넷"으로, 서비스 발견, 라우팅, 부하 분산 메커니즘을 제공하며, 대규모 개방형 에이전트 네트워크 구축에 적합하다.

**HelloAgents의 통합 방안**

`HelloAgents` 프레임워크에서 이 세 가지 프로토콜은 도구(Tool)로 통일되어 추상화되어 있어, 매끄럽게 통합되었다. 개발자는 유연하게 에이전트에 서로 다른 계층의 통신 능력을 추가할 수 있다:

```python
# 통일된 Tool 인터페이스
from hello_agents.tools import MCPTool, A2ATool, ANPTool

# 모든 프로토콜을 Tool로 Agent에 추가 가능
agent.add_tool(MCPTool(...))
agent.add_tool(A2ATool(...))
agent.add_tool(ANPTool(...))
```

**실전 경험 요약**

- 성숙한 커뮤니티 MCP 서비스를 우선적으로 활용하여 불필요한 중복 개발을 줄인다.
- 시스템 규모에 따라 적합한 프로토콜을 선택한다: 소규모 협력 시나리오에는 A2A를, 대규모 네트워크 시나리오에는 ANP를 채택한다.

본 장을 완료한 후, 다음을 권장한다:

1. **직접 실습**:
   - 자신만의 MCP 서버 구축
   - 프로토콜을 활용한 멀티 Agent 협력 시스템 구축
   - MCP, A2A, ANP 조합 응용 전략

2. **심층 학습**:
   - MCP 공식 문서 읽기: https://modelcontextprotocol.io
   - A2A 공식 문서 읽기: https://a2a-protocol.org/latest/
   - ANP 공식 문서 읽기: https://agent-network-protocol.com/guide/

3. **커뮤니티 참여**:
   - 새로운 MCP 서비스를 커뮤니티에 기여
   - 개인 개발 에이전트 구현 사례 공유
   - 관련 프로토콜 기술 표준 논의 참여, Issue에 질문하거나 Helloagents의 새로운 example 사례 지원 가능

**제10장 학습을 축하한다!**

이제 에이전트 통신 프로토콜의 핵심 지식을 습득했다. 계속 파이팅! 🚀

---

## 연습 문제

> **힌트**: 일부 연습 문제에는 표준 답안이 없으며, 에이전트 통신 프로토콜에 대한 학습자의 종합적인 이해와 실천 능력 배양에 중점을 둔다.

1. 본 장에서는 MCP, A2A, ANP 세 가지 에이전트 통신 프로토콜을 소개했다. 다음을 분석해라:

   - 10.1.2절에서 세 가지 프로토콜의 설계 이념을 비교했다. 깊이 분석하라: 왜 MCP는 "컨텍스트 공유"를, A2A는 "대화식 협력"을, ANP는 "네트워크 토폴로지"를 강조하는가? 이 설계 이념들은 각각 어떤 핵심 문제를 해결하는가?
   - "지능형 고객 서비스 시스템"을 구축한다고 가정하자. 다음 기능이 필요하다: (1) 고객 데이터베이스와 주문 시스템 접근; (2) 여러 전문 고객 서비스 에이전트가 협력하여 복잡한 문제 처리; (3) 대규모 동시 사용자 요청 지원. 각 기능에 가장 적합한 프로토콜을 선택하고 이유를 설명하라.
   - 세 가지 프로토콜을 조합하여 사용할 수 있는가? MCP, A2A, ANP를 동시에 사용하여 완전한 에이전트 시스템을 구축하는 실제 응용 시나리오를 설계하라. 시스템 아키텍처 다이어그램을 그리고 각 프로토콜의 역할을 설명하라.

2. MCP(Model Context Protocol)는 에이전트와 도구 통신의 표준 프로토콜이다. 10.2절의 내용을 기반으로 깊이 생각하라:

   > **힌트**: 이것은 직접 실습 문제이며, 실제 조작을 권장한다.

   - 10.2.3절의 MCP 서버 구현에서 `list_tools`, `call_tool` 등 핵심 메서드를 정의했다. 이 구현을 확장하여 다음 도구를 제공하는 새로운 MCP 서버를 추가하라: (1) 데이터베이스 쿼리 도구; (2) 데이터 시각화 도구; (3) 보고서 생성 도구. 도구 간에 협력하여 복잡한 데이터 분석 작업을 완료할 수 있어야 한다.
   - MCP 프로토콜은 "리소스"(Resources)와 "프롬프트"(Prompts)라는 두 가지 중요한 개념을 지원하지만, 본 장에서는 주로 "도구"(Tools)에 집중했다. MCP 공식 문서를 참조하여 Resources와 Prompts의 설계 목적을 이해하고, 이 세 가지 핵심 개념을 활용하여 더 강력한 에이전트 시스템을 구축하는 응용 시나리오를 설계하라.
   - MCP는 JSON-RPC 2.0을 하위 통신 프로토콜로 사용하며, stdio를 통해 프로세스 간 통신을 수행한다. 분석하라: 이 설계의 장점과 한계는 무엇인가? 원격 MCP 서버(HTTP/WebSocket을 통한 접근)를 지원해야 한다면, 현재 구현을 어떻게 확장해야 하는가?

3. A2A(Agent-to-Agent Protocol)는 에이전트 간의 대화식 협력을 지원한다. 10.3절의 내용을 기반으로 다음 확장 실습을 완료하라:

   > **힌트**: 이것은 직접 실습 문제이며, 실제 조작을 권장한다.

   - 10.3.4절의 "연구 팀" 사례에서 연구원과 작성자가 A2A 프로토콜을 통해 논문 작성을 협력 완료한다. 이 사례를 확장하여 세 번째 에이전트 "심사자"(Reviewer)를 추가하라. 논문 품질을 심사하고 수정 의견을 제시할 수 있어야 한다. 세 에이전트 간의 협력 흐름을 설계하고 완전한 코드를 구현하라.
   - A2A 프로토콜은 `task`, `task_result` 등의 메시지 유형을 정의한다. 분석하라: 협력 과정에서 충돌이 발생하면(예: 두 에이전트가 동일한 문제에 대해 다른 의견을 가질 때), 어떻게 충돌 해결 메커니즘을 설계해야 하는가? A2A 프로토콜을 확장하여 "협상"(negotiation)과 "투표"(voting) 등의 메시지 유형을 추가하라.
   - A2A 프로토콜을 제6장에서 소개한 AutoGen, CAMEL 등 멀티 에이전트 프레임워크와 비교하라: 표준 프로토콜로서의 A2A와 이 프레임워크들의 관계는 무엇인가? 서로를 대체할 수 있는가? A2A 프로토콜 기반 에이전트가 AutoGen 프레임워크의 에이전트와 통신할 수 있는 방안을 설계하라.

4. ANP(Agent Network Protocol)는 대규모 에이전트 네트워크를 지원한다. 10.4절의 내용을 기반으로 깊이 분석하라:

   - 10.4.2절에서 스타, 메시, 계층형 등 ANP의 네트워크 토폴로지 설계를 소개했다. 분석하라: 어떤 시나리오에서 어떤 토폴로지 구조를 선택해야 하는가? 네트워크 규모가 10개의 에이전트에서 1000개의 에이전트로 확장된다면, 토폴로지 구조는 어떻게 진화해야 하는가?
   - ANP 프로토콜은 "라우팅"(routing)과 "발견"(discovery) 메커니즘을 지원하여 에이전트가 동적으로 적합한 협력 파트너를 찾을 수 있게 한다. "지능형 라우팅 알고리즘"을 설계하라: 작업 유형, 에이전트 능력, 네트워크 부하 등의 요소에 따라 최적의 메시지 라우팅 경로를 자동으로 선택한다.
   - 10.4.4절의 "스마트 시티" 사례에서 여러 에이전트가 협력하여 도시 시스템을 관리한다. 생각해보라: 핵심 에이전트(예: 교통 관리 에이전트)에 장애가 발생하면, 전체 시스템은 어떻게 대응해야 하는가? 장애 감지, 백업 전환, 상태 복구 등 기능을 포함하는 "내결함성 메커니즘"을 설계하라.

5. 에이전트 통신 프로토콜의 보안성과 개인정보 보호는 실제 응용에서 핵심 문제다. 다음을 생각해보라:

   - 10.2.4절의 MCP 클라이언트 구현에서 에이전트는 MCP 서버가 제공하는 어떤 도구도 호출할 수 있다. 분석하라: 이 설계에는 어떤 보안 위험이 있는가? MCP 서버가 위험한 작업(파일 삭제, 시스템 명령 실행 등)을 제공한다면, 권한 제어 메커니즘을 어떻게 설계해야 하는가?
   - A2A와 ANP 프로토콜은 여러 에이전트 간의 통신을 포함하며, 민감한 정보(사용자 개인정보, 영업 기밀 등)가 포함될 수 있다. "종단 간 암호화" 방안을 설계하라: 전송 과정에서 메시지가 도청되거나 변조되지 않도록 보장하면서, 에이전트 신원 인증과 접근 제어를 지원한다.
   - 대규모 에이전트 네트워크에서 악의적인 에이전트는 허위 정보를 전송하거나, 서비스 거부 공격을 발동하거나, 다른 에이전트의 데이터를 탈취할 수 있다. "신뢰 평가 시스템"을 설계하라: 에이전트의 역사적 행동, 협력 품질, 커뮤니티 평가 등의 요소에 따라 각 에이전트의 신뢰도를 동적으로 평가하고, 이를 기반으로 통신 전략을 조정한다.

---

## 참고문헌

[1] Anthropic. (2024). *Model Context Protocol*. Retrieved October 7, 2025, from https://modelcontextprotocol.io/

[2] The A2A Project. (2025). *A2A Protocol: An open protocol for agent-to-agent communication*. Retrieved October 7, 2025, from https://a2a-protocol.org/

[3] Chang, G., Lin, E., Yuan, C., Cai, R., Chen, B., Xie, X., & Zhang, Y. (2025). *Agent Network Protocol technical white paper*. arXiv. https://doi.org/10.48550/arXiv.2508.00007