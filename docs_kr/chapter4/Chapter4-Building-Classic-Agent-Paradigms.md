# 제4장 에이전트 고전 패러다임 구축

이전 장에서는 현대 에이전트의 "뇌"인 대형 언어 모델을 심층적으로 탐구했다. 내부의 Transformer 아키텍처, 이와 상호작용하는 방법, 그리고 능력의 한계를 살펴봤다. 이제 이 이론적 지식을 실천으로 전환하여 에이전트를 직접 구축할 시간이다.

현대 에이전트의 핵심 능력은 대형 언어 모델의 추론 능력과 외부 세계를 연결하는 데 있다. 에이전트는 사용자의 의도를 자율적으로 이해하고, 복잡한 작업을 분해하며, 코드 인터프리터·검색 엔진·API 등 다양한 "도구"를 호출하여 정보를 획득하고 작업을 실행함으로써 최종 목표를 달성한다. 그러나 에이전트가 만능은 아니다. 대형 모델 자체의 "환각(Hallucination)" 문제, 복잡한 작업에서 추론 루프에 빠질 가능성, 도구의 잘못된 사용 등의 도전도 직면하며, 이것들이 에이전트의 능력 한계를 구성한다.

에이전트의 "사고"와 "행동" 과정을 더 잘 조직하기 위해, 업계에서는 다양한 고전 아키텍처 패러다임이 등장했다. 이 장에서는 그 중 가장 대표적인 세 가지에 집중하여, 이를 처음부터 단계적으로 구현한다:

- **ReAct (Reasoning and Acting):** "사고"와 "행동"을 긴밀하게 결합하는 패러다임으로, 에이전트가 생각하면서 동시에 행동하고 동적으로 조정한다.
- **Plan-and-Solve:** "먼저 충분히 생각한 후 행동하는" 패러다임으로, 에이전트가 먼저 완전한 행동 계획을 생성한 뒤 엄격하게 실행한다.
- **Reflection:** 에이전트에게 "반성" 능력을 부여하는 패러다임으로, 자기 비판과 수정을 통해 결과를 최적화한다.

이를 이해한 후 이런 질문이 생길 수 있다. 시중에는 이미 LangChain, LlamaIndex 등 수많은 우수한 프레임워크가 있는데, 왜 굳이 "바퀴를 다시 발명"해야 하는가? 그 답은 다음과 같다. 성숙한 프레임워크가 엔지니어링 효율성 면에서 확실히 우수하지만, 고도로 추상화된 도구를 직접 사용하는 것은 그 배후의 설계 메커니즘이 어떻게 작동하는지, 어떤 이점이 있는지를 이해하는 데 도움이 되지 않는다. 또한, 이 과정은 프로젝트의 엔지니어링 과제를 드러낸다. 프레임워크는 모델 출력 형식 파싱, 도구 호출 실패 시 재시도, 에이전트가 무한 루프에 빠지는 것 방지 등 많은 문제를 처리해 준다. 이러한 문제들을 직접 다루는 것이 시스템 설계 능력을 기르는 가장 직접적인 방법이다. 마지막으로, 가장 중요한 점은, 설계 원리를 습득해야 비로소 프레임워크의 "사용자"에서 에이전트 애플리케이션의 "창조자"로 진정으로 변할 수 있다는 것이다. 표준 컴포넌트로 복잡한 요구를 충족할 수 없을 때, 심층 커스터마이징이나 완전히 새로운 에이전트를 처음부터 구축하는 능력을 갖추게 된다.

---

## 4.1 환경 준비와 기본 도구 정의

구축을 시작하기 전에, 먼저 개발 환경을 설정하고 몇 가지 기본 컴포넌트를 정의해야 한다. 이를 통해 이후 다양한 패러다임을 구현할 때 중복 작업을 피하고 핵심 로직에 더 집중할 수 있다.

**4.1.1 의존성 라이브러리 설치:**

이 책의 실전 부분은 주로 Python 언어를 사용하며, Python 3.10 이상 버전을 권장한다. 먼저 대형 언어 모델과 상호작용하기 위한 `openai` 라이브러리와 API 키를 안전하게 관리하기 위한 `python-dotenv` 라이브러리를 설치했는지 확인한다.

터미널에서 다음 명령어를 실행한다:

```bash
pip install openai python-dotenv
```

**4.1.2 API 키 설정:**

코드를 더 범용적으로 만들기 위해, 모델 서비스 관련 정보(모델 ID, API 키, 서비스 주소)를 환경 변수에 통합하여 설정한다.

1. 프로젝트 루트 디렉토리에 `.env`라는 이름의 파일을 생성한다.
2. 해당 파일에 다음 내용을 추가한다. 필요에 따라 OpenAI 공식 서비스나 OpenAI 인터페이스와 호환되는 로컬/서드파티 서비스를 지정할 수 있다.
3. 가져오는 방법을 모른다면 [환경 설정](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra07-环境配置.md)을 참고할 수 있다.

```bash
# .env file
LLM_API_KEY="YOUR-API-KEY"
LLM_MODEL_ID="YOUR-MODEL"
LLM_BASE_URL="YOUR-URL"
```

코드는 이 파일에서 자동으로 설정을 불러온다.

**4.1.3 기본 LLM 호출 함수 캡슐화:**

코드 구조를 더 명확하고 재사용하기 쉽게 만들기 위해, 전용 LLM 클라이언트 클래스를 정의한다. 이 클래스는 모델 서비스와 상호작용하는 모든 세부 사항을 캡슐화하여, 주요 로직이 에이전트 구축에 더 집중할 수 있게 한다.

```python
import os
from openai import OpenAI
from dotenv import load_dotenv
from typing import List, Dict

# .env 파일의 환경 변수 로드
load_dotenv()

class HelloAgentsLLM:
    """
    이 책 "Hello Agents"를 위해 커스터마이징된 LLM 클라이언트.
    OpenAI 인터페이스와 호환되는 모든 서비스를 호출하며,
    기본적으로 스트리밍 응답을 사용한다.
    """
    def __init__(self, model: str = None, apiKey: str = None, baseUrl: str = None, timeout: int = None):
        """
        클라이언트를 초기화한다. 전달된 매개변수를 우선 사용하고,
        제공되지 않은 경우 환경 변수에서 로드한다.
        """
        self.model = model or os.getenv("LLM_MODEL_ID")
        apiKey = apiKey or os.getenv("LLM_API_KEY")
        baseUrl = baseUrl or os.getenv("LLM_BASE_URL")
        timeout = timeout or int(os.getenv("LLM_TIMEOUT", 60))
        
        if not all([self.model, apiKey, baseUrl]):
            raise ValueError("모델 ID, API 키, 서비스 주소는 반드시 제공되거나 .env 파일에 정의되어야 한다.")

        self.client = OpenAI(api_key=apiKey, base_url=baseUrl, timeout=timeout)

    def think(self, messages: List[Dict[str, str]], temperature: float = 0) -> str:
        """
        대형 언어 모델을 호출하여 사고하게 하고, 응답을 반환한다.
        """
        print(f"🧠 {self.model} 모델을 호출 중...")
        try:
            response = self.client.chat.completions.create(
                model=self.model,
                messages=messages,
                temperature=temperature,
                stream=True,
            )
            
            # 스트리밍 응답 처리
            print("✅ 대형 언어 모델 응답 성공:")
            collected_content = []
            for chunk in response:
                content = chunk.choices[0].delta.content or ""
                print(content, end="", flush=True)
                collected_content.append(content)
            print()  # 스트리밍 출력 후 줄바꿈
            return "".join(collected_content)

        except Exception as e:
            print(f"❌ LLM API 호출 중 오류 발생: {e}")
            return None

# --- 클라이언트 사용 예시 ---
if __name__ == '__main__':
    try:
        llmClient = HelloAgentsLLM()
        
        exampleMessages = [
            {"role": "system", "content": "You are a helpful assistant that writes Python code."},
            {"role": "user", "content": "퀵 정렬 알고리즘을 작성해줘"}
        ]
        
        print("--- LLM 호출 ---")
        responseText = llmClient.think(exampleMessages)
        if responseText:
            print("\n\n--- 완전한 모델 응답 ---")
            print(responseText)

    except ValueError as e:
        print(e)


>>>
--- LLM 호출 ---
🧠 xxxxxx 모델을 호출 중...
✅ 대형 언어 모델 응답 성공:
퀵 정렬은 매우 효율적인 정렬 알고리즘이다...
```

---

## 4.2 ReAct

LLM 클라이언트를 준비한 후, 첫 번째이자 가장 고전적인 에이전트 패러다임인 **ReAct (Reason + Act)**를 구축한다. ReAct는 Shunyu Yao가 2022년에 제안했으며[1], 핵심 사상은 인간이 문제를 해결하는 방식을 모방하여 **추론(Reasoning)**과 **행동(Acting)**을 명시적으로 결합하고, "사고-행동-관찰"의 순환을 형성하는 것이다.

**4.2.1 ReAct의 작업 흐름:**

ReAct가 등장하기 이전, 주류 방법은 두 가지로 나뉘었다. 하나는 "순수 사고"형인 **사고 연쇄(Chain-of-Thought)**로, 모델이 복잡한 논리적 추론을 수행하도록 유도하지만 외부 세계와 상호작용할 수 없어 사실적 환각이 발생하기 쉬웠다. 다른 하나는 "순수 행동"형으로, 모델이 실행할 동작을 직접 출력하지만 계획과 오류 수정 능력이 부족했다.

ReAct의 묘미는, **사고와 행동이 서로 보완적**이라는 것을 인식한 데 있다. 사고는 행동을 안내하고, 행동의 결과는 다시 사고를 수정한다. 이를 위해 ReAct 패러다임은 특수한 프롬프트 엔지니어링을 통해 모델을 안내하며, 매 단계의 출력이 고정된 궤적을 따르도록 한다:

- **Thought(사고):** 에이전트의 "내면 독백"이다. 현재 상황을 분석하고, 작업을 분해하며, 다음 단계 계획을 수립하거나 이전 단계의 결과를 반성한다.
- **Action(행동):** 에이전트가 취하기로 결정한 구체적인 동작으로, 보통 외부 도구를 호출한다. 예: `Search['화웨이 최신 스마트폰']`
- **Observation(관찰):** `Action` 실행 후 외부 도구에서 반환된 결과로, 예를 들어 검색 결과 요약이나 API 반환값이다.

에이전트는 이 **Thought -> Action -> Observation** 순환을 계속 반복하며, 새로운 관찰 결과를 이력에 추가하여 끊임없이 증가하는 컨텍스트를 형성하다가, `Thought`에서 최종 답을 찾았다고 판단하면 결과를 출력한다. 이 과정은 강력한 시너지 효과를 형성한다: **추론은 행동을 더 목적 지향적으로 만들고, 행동은 추론에 사실적 근거를 제공한다.**

이 과정을 형식화하면 그림 4.1과 같다. 구체적으로, 각 시간 단계 $t$에서 에이전트의 정책(즉, 대형 언어 모델 $\pi$)은 초기 질문 $q$와 이전 모든 단계의 "행동-관찰" 이력 궤적 $((a_1,o_1),\dots,(a_{t-1},o_{t-1}))$에 기반하여 현재의 사고 $th_t$와 행동 $a_t$를 생성한다:

$$\left(th_t,a_t\right)=\pi\left(q,(a_1,o_1),\ldots,(a_{t-1},o_{t-1})\right)$$

그런 다음, 환경의 도구 $T$가 행동 $a_t$를 실행하고 새로운 관찰 결과 $o_t$를 반환한다:

$$o_t = T(a_t)$$

이 순환은 계속 진행되어 새로운 $(a_t,o_t)$ 쌍을 이력에 추가하다가, 모델이 사고 $th_t$에서 작업이 완료되었다고 판단하면 종료된다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-1.png" alt="ReAct 패러다임의 사고-행동-관찰 협력 순환" width="90%"/>
  <p>그림 4.1 ReAct 패러다임의 "사고-행동-관찰" 협력 순환</p>
</div>

이 메커니즘은 다음 시나리오에 특히 적합하다:

- **외부 지식이 필요한 작업:** 실시간 정보(날씨, 뉴스, 주가) 조회, 전문 분야 지식 검색 등
- **정확한 계산이 필요한 작업:** 수학 문제를 계산기 도구에 위임하여 LLM의 계산 오류 방지
- **API와 상호작용이 필요한 작업:** 데이터베이스 조작, 특정 서비스 API 호출로 특정 기능 완수

따라서 **외부 도구 사용** 능력을 갖춘 ReAct 에이전트를 구축하여, 대형 언어 모델이 자체 지식만으로는 직접 답할 수 없는 질문을 처리한다. 예를 들면: "화웨이의 최신 스마트폰은 어떤 기종인가? 주요 판매 포인트는 무엇인가?" 이 질문은 에이전트가 인터넷 검색이 필요하다는 것을 이해하고, 도구를 호출하여 검색 결과를 요약해야 한다.

**4.2.2 도구의 정의와 구현:**

대형 언어 모델이 에이전트의 뇌라면, **도구(Tools)**는 외부 세계와 상호작용하는 "손과 발"이다. ReAct 패러다임이 우리가 설정한 문제를 실제로 해결할 수 있으려면, 에이전트에게 외부 도구를 호출하는 능력이 필요하다.

이 절에서 설정한 목표인 "화웨이 최신 스마트폰" 관련 질문에 답하기 위해, 에이전트에게 웹 검색 도구를 제공한다. 여기서는 **SerpApi**를 사용하는데, 이는 API를 통해 구조화된 Google 검색 결과를 제공하며 "답변 요약 박스"나 정확한 지식 그래프 정보를 직접 반환할 수 있다.

먼저 해당 라이브러리를 설치한다:

```bash
pip install google-search-results
```

동시에 [SerpApi 공식 사이트](https://serpapi.com/)에서 무료 계정을 등록하고 API 키를 획득한 뒤, 프로젝트 루트 디렉토리의 `.env` 파일에 추가한다:

```bash
# .env file
# ... (이전 LLM 설정 유지)
SERPAPI_API_KEY="YOUR_SERPAPI_API_KEY"
```

이제 코드로 이 도구를 정의하고 관리한다. 먼저 도구의 핵심 기능을 구현한 뒤 범용 도구 관리자를 구축한다.

**(1) 검색 도구의 핵심 로직 구현**

잘 정의된 도구는 다음 세 가지 핵심 요소를 포함해야 한다:

1. **이름(Name):** 에이전트가 `Action`에서 호출할 간결하고 고유한 식별자, 예: `Search`
2. **설명(Description):** 이 도구의 용도를 설명하는 명확한 자연어 설명. **이것이 전체 메커니즘에서 가장 중요한 부분**으로, 대형 언어 모델이 이 설명에 의존하여 언제 어떤 도구를 사용할지 판단한다.
3. **실행 로직(Execution Logic):** 실제로 작업을 실행하는 함수 또는 메서드.

첫 번째 도구는 `search` 함수로, 쿼리 문자열을 받아 검색 결과를 반환한다.

```python
from serpapi import SerpApiClient

def search(query: str) -> str:
    """
    SerpApi 기반의 실전 웹 검색 엔진 도구.
    검색 결과를 지능적으로 파싱하여, 직접 답변이나 지식 그래프 정보를 우선 반환한다.
    """
    print(f"🔍 [SerpApi] 웹 검색 실행 중: {query}")
    try:
        api_key = os.getenv("SERPAPI_API_KEY")
        if not api_key:
            return "오류: SERPAPI_API_KEY가 .env 파일에 설정되지 않았다."

        params = {
            "engine": "google",
            "q": query,
            "api_key": api_key,
            "gl": "cn",  # 국가 코드
            "hl": "zh-cn", # 언어 코드
        }
        
        client = SerpApiClient(params)
        results = client.get_dict()
        
        # 지능적 파싱: 가장 직접적인 답변을 우선 탐색
        if "answer_box_list" in results:
            return "\n".join(results["answer_box_list"])
        if "answer_box" in results and "answer" in results["answer_box"]:
            return results["answer_box"]["answer"]
        if "knowledge_graph" in results and "description" in results["knowledge_graph"]:
            return results["knowledge_graph"]["description"]
        if "organic_results" in results and results["organic_results"]:
            # 직접 답변이 없을 경우 상위 3개 검색 결과 요약 반환
            snippets = [
                f"[{i+1}] {res.get('title', '')}\n{res.get('snippet', '')}"
                for i, res in enumerate(results["organic_results"][:3])
            ]
            return "\n\n".join(snippets)
        
        return f"죄송합니다, '{query}'에 관한 정보를 찾을 수 없었다."

    except Exception as e:
        return f"검색 중 오류 발생: {e}"
```

위 코드에서는 먼저 `answer_box`(Google의 답변 요약 박스)나 `knowledge_graph`(지식 그래프) 등의 정보가 존재하는지 확인하고, 존재하면 이 가장 정확한 답변을 직접 반환한다. 없을 경우에는 상위 3개의 일반 검색 결과 요약을 반환한다. 이러한 "지능적 파싱"은 LLM에 더 높은 품질의 정보를 제공한다.

**(2) 범용 도구 실행기 구축**

에이전트가 여러 도구를 사용해야 할 때(예: 검색 외에도 계산, 데이터베이스 조회 등), 이러한 도구를 등록하고 디스패치하는 통합 관리자가 필요하다. 이를 위해 `ToolExecutor` 클래스를 만든다.

```python
from typing import Dict, Any

class ToolExecutor:
    """
    도구를 관리하고 실행하는 도구 실행기.
    """
    def __init__(self):
        self.tools: Dict[str, Dict[str, Any]] = {}

    def registerTool(self, name: str, description: str, func: callable):
        """
        도구함에 새 도구를 등록한다.
        """
        if name in self.tools:
            print(f"경고: 도구 '{name}'이 이미 존재하며, 덮어쓴다.")
        self.tools[name] = {"description": description, "func": func}
        print(f"도구 '{name}'이 등록되었다.")

    def getTool(self, name: str) -> callable:
        """
        이름으로 도구의 실행 함수를 가져온다.
        """
        return self.tools.get(name, {}).get("func")

    def getAvailableTools(self) -> str:
        """
        사용 가능한 모든 도구의 형식화된 설명 문자열을 가져온다.
        """
        return "\n".join([
            f"- {name}: {info['description']}" 
            for name, info in self.tools.items()
        ])
```

**(3) 테스트**

이제 `search` 도구를 `ToolExecutor`에 등록하고 한 번 호출하여 전체 흐름이 정상적으로 작동하는지 확인한다.

```python
# --- 도구 초기화 및 사용 예시 ---
if __name__ == '__main__':
    # 1. 도구 실행기 초기화
    toolExecutor = ToolExecutor()

    # 2. 실전 검색 도구 등록
    search_description = "웹 검색 엔진이다. 시사, 사실, 지식 베이스에서 찾을 수 없는 정보를 답해야 할 때 사용한다."
    toolExecutor.registerTool("Search", search_description, search)
    
    # 3. 사용 가능한 도구 출력
    print("\n--- 사용 가능한 도구 ---")
    print(toolExecutor.getAvailableTools())

    # 4. 에이전트의 Action 호출 - 실시간 질문
    print("\n--- Action 실행: Search['엔비디아 최신 GPU 모델은 무엇인가'] ---")
    tool_name = "Search"
    tool_input = "엔비디아 최신 GPU 모델은 무엇인가"

    tool_function = toolExecutor.getTool(tool_name)
    if tool_function:
        observation = tool_function(tool_input)
        print("--- 관찰 (Observation) ---")
        print(observation)
    else:
        print(f"오류: '{tool_name}'이라는 도구를 찾을 수 없다.")
        
>>>
도구 'Search'가 등록되었다.

--- 사용 가능한 도구 ---
- Search: 웹 검색 엔진이다. 시사, 사실, 지식 베이스에서 찾을 수 없는 정보를 답해야 할 때 사용한다.

--- Action 실행: Search['엔비디아 최신 GPU 모델은 무엇인가'] ---
🔍 [SerpApi] 웹 검색 실행 중: 엔비디아 최신 GPU 모델은 무엇인가
--- 관찰 (Observation) ---
[1] GeForce RTX 50 시리즈 그래픽카드
GeForce RTX™ 50 시리즈 GPU는 NVIDIA Blackwell 아키텍처를 탑재하여 게이머와 크리에이터에게 새로운 경험을 제공한다. RTX 50 시리즈는 강력한 AI 연산 능력으로 업그레이드된 경험과 더욱 실감나는 화면을 제공한다.

[2] GeForce 시리즈 최신 세대와 이전 세대 그래픽카드 비교
최신 세대 RTX 30 시리즈와 이전 세대 RTX 20, GTX 10, 900 시리즈를 비교한다. 스펙, 기능, 기술 지원 등을 확인한다.

[3] GeForce 그래픽카드 | NVIDIA
DRIVE AGX. AI 기반 지능형 자동차 시스템을 위한 강력한 차량 내 컴퓨팅 능력 · Clara AGX. 혁신적인 의료 기기와 이미징을 위한 AI 컴퓨팅. 게임과 창작. GeForce. 그래픽카드, 게임 솔루션, AI...
```

이로써 에이전트에게 실제 인터넷에 연결하는 `Search` 도구를 장착했으며, 이후 ReAct 루프를 위한 견고한 기반을 마련했다.

**4.2.3 ReAct 에이전트의 코딩 구현:**

이제 LLM 클라이언트와 도구 실행기 등 모든 독립 컴포넌트를 조립하여 완전한 ReAct 에이전트를 구축한다. `ReActAgent` 클래스를 통해 핵심 로직을 캡슐화한다. 이해를 돕기 위해 이 클래스의 구현 과정을 다음 몇 가지 핵심 부분으로 나누어 설명한다.

**(1) 시스템 프롬프트 설계**

프롬프트는 전체 ReAct 메커니즘의 초석으로, 대형 언어 모델에 행동 지침을 제공한다. 사용 가능한 도구, 사용자 질문, 중간 단계의 상호작용 이력을 동적으로 삽입할 수 있는 템플릿을 정교하게 설계해야 한다.

```bash
# ReAct 프롬프트 템플릿
REACT_PROMPT_TEMPLATE = """
주의: 당신은 외부 도구를 호출할 수 있는 지능형 어시스턴트다.

사용 가능한 도구:
{tools}

다음 형식을 엄격히 따라 응답하라:

Thought: 문제를 분석하고 작업을 분해하며 다음 행동을 계획하는 사고 과정.
Action: 취할 행동으로, 다음 형식 중 하나여야 한다:
- `{{tool_name}}[{{tool_input}}]`: 사용 가능한 도구를 호출한다.
- `Finish[최종 답변]`: 최종 답변을 얻었다고 판단될 때.
- 충분한 정보를 수집하여 최종 질문에 답할 수 있을 때, Action: 필드 뒤에 Finish[최종 답변]을 사용하여 최종 답변을 출력해야 한다.

이제 다음 질문 해결을 시작한다:
Question: {question}
History: {history}
"""
```

이 템플릿은 에이전트와 LLM 사이의 상호작용 규범을 정의한다:

- **역할 정의:** "당신은 외부 도구를 호출할 수 있는 지능형 어시스턴트다"라고 LLM의 역할을 설정한다.
- **도구 목록(`{tools}`):** LLM에게 어떤 "손발"이 있는지 알린다.
- **형식 규약(`Thought`/`Action`):** 가장 중요한 부분으로, LLM의 출력이 구조적이도록 강제하여 코드로 정확하게 의도를 파싱할 수 있게 한다.
- **동적 컨텍스트(`{question}`/`{history}`):** 사용자의 원래 질문과 누적되는 상호작용 이력을 주입하여, LLM이 완전한 컨텍스트를 기반으로 의사결정하게 한다.

**(2) 핵심 루프의 구현**

`ReActAgent`의 핵심은 루프로, "프롬프트 형식화 -> LLM 호출 -> 동작 실행 -> 결과 통합"을 반복하다가 작업이 완료되거나 최대 단계 수에 도달하면 종료된다.

```python
class ReActAgent:
    def __init__(self, llm_client: HelloAgentsLLM, tool_executor: ToolExecutor, max_steps: int = 5):
        self.llm_client = llm_client
        self.tool_executor = tool_executor
        self.max_steps = max_steps
        self.history = []

    def run(self, question: str):
        """
        ReAct 에이전트를 실행하여 질문에 답한다.
        """
        self.history = [] # 매 실행 시 이력 초기화
        current_step = 0

        while current_step < self.max_steps:
            current_step += 1
            print(f"--- 단계 {current_step} ---")

            # 1. 프롬프트 형식화
            tools_desc = self.tool_executor.getAvailableTools()
            history_str = "\n".join(self.history)
            prompt = REACT_PROMPT_TEMPLATE.format(
                tools=tools_desc,
                question=question,
                history=history_str
            )

            # 2. LLM 호출하여 사고
            messages = [{"role": "user", "content": prompt}]
            response_text = self.llm_client.think(messages=messages)
            
            if not response_text:
                print("오류: LLM이 유효한 응답을 반환하지 않았다.")
                break

            # ... (이후 파싱, 실행, 통합 단계)
```

`run` 메서드는 에이전트의 진입점이다. `while` 루프가 ReAct 패러다임의 본체를 구성하며, `max_steps` 매개변수는 에이전트가 무한 루프에 빠져 자원을 소진하는 것을 방지하는 중요한 안전 밸브다.

**(3) 출력 파서의 구현**

LLM이 반환하는 것은 순수 텍스트이므로, 그 안에서 `Thought`와 `Action`을 정확하게 추출해야 한다. 이는 보통 정규 표현식을 사용하는 몇 가지 보조 파싱 함수로 구현된다.

```python
# (이 메서드들은 ReActAgent 클래스의 일부)
    def _parse_output(self, text: str):
        """LLM의 출력을 파싱하여 Thought와 Action을 추출한다."""
        # Thought: Action: 또는 텍스트 끝까지 매칭
        thought_match = re.search(r"Thought:\s*(.*?)(?=\nAction:|$)", text, re.DOTALL)
        # Action: 텍스트 끝까지 매칭
        action_match = re.search(r"Action:\s*(.*?)$", text, re.DOTALL)
        thought = thought_match.group(1).strip() if thought_match else None
        action = action_match.group(1).strip() if action_match else None
        return thought, action

    def _parse_action(self, action_text: str):
        """Action 문자열을 파싱하여 도구 이름과 입력을 추출한다."""
        match = re.match(r"(\w+)\[(.*)\]", action_text, re.DOTALL)
        if match:
            return match.group(1), match.group(2)
        return None, None
```

- `_parse_output`: LLM의 완전한 응답에서 `Thought`와 `Action` 두 주요 부분을 분리한다.
- `_parse_action`: `Action` 문자열을 더 파싱한다. 예를 들어 `Search[화웨이 최신 스마트폰]`에서 도구 이름 `Search`와 도구 입력 `화웨이 최신 스마트폰`을 추출한다.

**(4) 도구 호출과 실행**

```python
# (이 로직은 run 메서드의 while 루프 안에 있다)
            # 3. LLM 출력 파싱
            thought, action = self._parse_output(response_text)
            
            if thought:
                print(f"사고: {thought}")

            if not action:
                print("경고: 유효한 Action을 파싱할 수 없어 흐름을 종료한다.")
                break

            # 4. Action 실행
            if action.startswith("Finish"):
                # Finish 명령이면 최종 답변을 추출하고 종료
                final_answer = re.match(r"Finish\[(.*)\]", action).group(1)
                print(f"🎉 최종 답변: {final_answer}")
                return final_answer
            
            tool_name, tool_input = self._parse_action(action)
            if not tool_name or not tool_input:
                # ... 유효하지 않은 Action 형식 처리 ...
                continue

            print(f"🎬 행동: {tool_name}[{tool_input}]")
            
            tool_function = self.tool_executor.getTool(tool_name)
            if not tool_function:
                observation = f"오류: '{tool_name}'이라는 도구를 찾을 수 없다."
            else:
                observation = tool_function(tool_input) # 실제 도구 호출
```

이 코드는 `Action` 실행의 중심이다. 먼저 `Finish` 명령인지 확인하고, 맞다면 흐름이 종료된다. 아니라면 `tool_executor`를 통해 해당 도구 함수를 가져와 실행하고 `observation`을 얻는다.

**(5) 관찰 결과의 통합**

마지막 단계이자 폐루프를 형성하는 핵심은, `Action` 자체와 도구 실행 후의 `Observation`을 이력 기록에 다시 추가하여 다음 루프에 새로운 컨텍스트를 제공하는 것이다.

```python
# (이 로직은 도구 호출 직후, while 루프 끝에 있다)
            print(f"👀 관찰: {observation}")
            
            # 이번 라운드의 Action과 Observation을 이력에 추가
            self.history.append(f"Action: {action}")
            self.history.append(f"Observation: {observation}")

        # 루프 종료
        print("최대 단계 수에 도달하여 흐름을 종료한다.")
        return None
```

`Observation`을 `self.history`에 추가함으로써, 에이전트는 다음 라운드에 프롬프트를 생성할 때 이전 단계 행동의 결과를 "볼" 수 있게 되고, 이를 바탕으로 새로운 사고와 계획을 수행한다.

**(6) 실행 인스턴스와 분석**

위의 모든 부분을 합치면 완전한 `ReActAgent` 클래스가 된다. 완전한 코드 실행 인스턴스는 이 책의 코드 저장소 `code` 폴더에서 찾을 수 있다.

아래는 실제 실행 기록이다:

```
도구 'Search'가 등록되었다.

--- 단계 1 ---
🧠 xxxxxx 모델을 호출 중...
✅ 대형 언어 모델 응답 성공:
Thought: 이 질문에 답하려면 화웨이의 최신 스마트폰 모델과 주요 특징을 찾아야 한다. 이 정보는 내 지식 베이스 밖에 있을 수 있으므로, 검색 엔진을 사용하여 최신 데이터를 가져와야 한다.
Action: Search[화웨이 최신 스마트폰 모델 및 주요 판매 포인트]
🤔 사고: 이 질문에 답하려면 화웨이의 최신 스마트폰 모델과 주요 특징을 찾아야 한다. 이 정보는 내 지식 베이스 밖에 있을 수 있으므로, 검색 엔진을 사용하여 최신 데이터를 가져와야 한다.
🎬 행동: Search[화웨이 최신 스마트폰 모델 및 주요 판매 포인트]
🔍 [SerpApi] 웹 검색 실행 중: 화웨이 최신 스마트폰 모델 및 주요 판매 포인트
👀 관찰: [1] 화웨이 스마트폰 - 화웨이 공식 사이트
스마트폰 ; Mate 시리즈. 특별한 플래그십 · HUAWEI Mate XTs. 특별한 마스터 ; Pura 시리즈. 선구자 이미징 · HUAWEI Pura 80 Pro+ ; Pocket 시리즈. 새로운 미학. HUAWEI Pocket 2 ; nova 시리즈. 전문 인물사진.

[2] 2025년 화웨이 스마트폰 중 가성비가 높은 것은? 화웨이 스마트폰 추천 및 시장 분석...
현재 화웨이 스마트폰의 가장 큰 판매 포인트는 HarmonyOS 시스템과 브랜드 충성도뿐이다. 여기서 현재 구매할 만한 몇 가지 화웨이 시리즈 스마트폰을 추천한다...

[3] 2025년 화웨이 신형 스마트폰 중 가성비가 높은 것은? 10가지 화웨이 신형 스마트폰 추천
화웨이를 선택하려면 주로 고급 스마트폰을 추천한다. Mate 70과 Pura 70 시리즈가 최신 발매 플래그십 모델이다. HUAWEI Mate 70. 장점은 여전히 최고의 카메라 구성, 전체 초점 범위 커버로 전문 사진 촬영에 적합하며, 마감이 훌륭하고 야외 내충격성이 좋다...

--- 단계 2 ---
🧠 xxxxxx 모델을 호출 중...
✅ 대형 언어 모델 응답 성공:
Thought: 검색 결과에 따르면, 화웨이가 최근 발매한 플래그십 모델에는 Mate 70과 Pura 80 Pro+가 포함된다. 가장 최신 모델을 확인하고 주요 판매 포인트를 이 정보에서 추출할 수 있다. 제공된 링크를 보면 Mate 70 시리즈와 Pura 80 Pro+가 모두 최근 발매된 제품이지만, 어느 것이 "가장 최신"인지는 추가 확인이 필요하다. 이 정보에서 주요 판매 포인트를 추출할 수 있다.
Action: Finish[최신 정보에 따르면, 화웨이의 최신 스마트폰은 HUAWEI Pura 80 Pro+ 또는 HUAWEI Mate 70일 가능성이 있다. 그 중 HUAWEI Mate 70의 주요 판매 포인트는 최고의 카메라 구성, 전체 초점 범위 커버로 전문 사진 촬영에 적합하며, 마감이 훌륭하고 야외 내충격 성능이 우수하다는 것이다. HUAWEI Pura 80 Pro+는 선구자 이미징 기술을 강조한다.]
🤔 사고: 검색 결과에 따르면, 화웨이가 최근 발매한 플래그십 모델에는 Mate 70과 Pura 80 Pro+가 포함된다...
🎉 최종 답변: 최신 정보에 따르면, 화웨이의 최신 스마트폰은 HUAWEI Pura 80 Pro+ 또는 HUAWEI Mate 70일 가능성이 있다. 그 중 HUAWEI Mate 70의 주요 판매 포인트는 최고의 카메라 구성, 전체 초점 범위 커버로 전문 사진 촬영에 적합하며, 마감이 훌륭하고 야외 내충격 성능이 우수하다는 것이다. HUAWEI Pura 80 Pro+는 선구자 이미징 기술을 강조한다.
```

위 출력에서 에이전트가 사고 체인을 명확하게 보여주는 것을 확인할 수 있다: 에이전트는 먼저 자신의 지식이 부족하여 검색 도구를 사용해야 함을 인식하고, 검색 결과를 바탕으로 추론과 요약을 수행하여 두 단계 만에 최종 답변을 도출했다.

주목할 점은, 모델의 지식과 인터넷 정보는 지속적으로 업데이트되므로 실행 결과가 이와 완전히 동일하지 않을 수 있다는 것이다. 이 절의 내용을 작성한 2025년 9월 8일 기준으로, 검색 결과에서 언급된 HUAWEI Mate 70과 HUAWEI Pura 80 Pro+는 실제로 화웨이의 최신 플래그십 시리즈 스마트폰이었다. 이는 ReAct 패러다임이 시효성 문제를 처리하는 데 있어 강력한 능력을 충분히 보여준다.

**4.2.4 ReAct의 특징, 한계와 디버깅 기법:**

ReAct 에이전트를 직접 구현함으로써, 그 작업 흐름을 습득했을 뿐만 아니라 내부 메커니즘에 대한 더 깊은 이해를 얻었다. 모든 기술 패러다임에는 빛나는 점과 개선할 여지가 있으므로, 본 절에서 ReAct를 정리한다.

**(1) ReAct의 주요 특징**

1. **높은 해석 가능성:** ReAct의 가장 큰 장점 중 하나는 투명성이다. `Thought` 체인을 통해 에이전트가 매 단계에서 왜 이 도구를 선택했는지, 다음에 무엇을 하려는지 명확히 볼 수 있다. 이는 에이전트의 행동을 이해하고, 신뢰하고, 디버깅하는 데 매우 중요하다.
2. **동적 계획과 오류 수정 능력:** 한 번에 완전한 계획을 생성하는 패러다임과 달리, ReAct는 "한 걸음 가고, 한 번 보는" 방식이다. 외부 세계에서 매 단계 얻은 `Observation`에 따라 이후의 `Thought`와 `Action`을 동적으로 조정한다. 이전 단계의 검색 결과가 만족스럽지 않으면, 다음 단계에서 검색어를 수정하여 다시 시도할 수 있다.
3. **도구 협업 능력:** ReAct 패러다임은 대형 언어 모델의 추론 능력과 외부 도구의 실행 능력을 자연스럽게 결합한다. LLM은 전략 기획(계획과 추론)을 담당하고, 도구는 구체적인 문제(검색, 계산)를 해결하며, 두 가지가 협력하여 단일 LLM의 지식 시효성, 계산 정확성 등 고유 한계를 극복한다.

**(2) ReAct의 고유 한계**

1. **LLM 자체 능력에 대한 강한 의존성:** ReAct 흐름의 성공 여부는 기저 LLM의 종합 능력에 크게 의존한다. LLM의 논리적 추론 능력, 명령 준수 능력, 형식화된 출력 능력이 부족하면 `Thought` 단계에서 잘못된 계획을 세우거나, `Action` 단계에서 형식에 맞지 않는 명령을 생성하여 전체 흐름이 중단되기 쉽다.
2. **실행 효율성 문제:** 단계적 특성으로 인해, 작업 하나를 완료하려면 보통 여러 번 LLM을 호출해야 한다. 매 호출마다 네트워크 지연과 계산 비용이 수반된다. 많은 단계가 필요한 복잡한 작업의 경우, 이 직렬 "사고-행동" 루프는 총 소요 시간과 비용이 높아질 수 있다.
3. **프롬프트의 취약성:** 전체 메커니즘의 안정적인 작동은 정교하게 설계된 프롬프트 템플릿에 기반한다. 템플릿의 미세한 변동, 심지어 단어 선택의 차이도 LLM의 행동에 영향을 줄 수 있다. 또한 모든 모델이 지정된 형식을 지속적으로 안정적으로 따르는 것은 아니어서, 실제 응용에서 불확실성이 증가한다.
4. **국소 최적에 빠질 가능성:** 단계별 의사결정 방식은 에이전트가 전체적이고 장기적인 계획이 부족하다는 것을 의미한다. 현재의 `Observation`으로 인해 눈앞에서는 옳아 보이지만 장기적으로는 최적이 아닌 경로를 선택할 수 있으며, 심한 경우 "제자리를 맴도는" 루프에 빠질 수 있다.

**(3) 디버깅 기법**

구축한 ReAct 에이전트가 기대에 맞지 않을 때, 다음과 같은 방면에서 디버깅할 수 있다:

- **완전한 프롬프트 확인:** 매 LLM 호출 전에 모든 이력 기록을 포함한 최종 형식화된 완전한 프롬프트를 출력한다. 이것이 LLM 의사결정의 근원을 추적하는 가장 직접적인 방법이다.
- **원시 출력 분석:** 출력 파싱이 실패할 때(예: 정규 표현식이 `Action`에 매칭되지 않을 때), LLM이 반환한 처리되지 않은 원시 텍스트를 반드시 출력한다. 이를 통해 LLM이 형식을 따르지 않은 것인지, 아니면 파싱 로직에 오류가 있는 것인지 판단할 수 있다.
- **도구의 입출력 검증:** 에이전트가 생성한 `tool_input`이 도구 함수가 기대하는 형식인지 확인하고, 도구가 반환하는 `observation` 형식이 에이전트가 이해하고 처리할 수 있는 것인지도 확인한다.
- **프롬프트의 예시 조정(Few-shot Prompting):** 모델이 자주 오류를 범하면 프롬프트에 완전한 "Thought-Action-Observation" 성공 사례를 한두 개 추가하여 예시를 통해 모델이 지침을 더 잘 따르도록 안내할 수 있다.
- **다른 모델이나 매개변수 시도:** 더 강력한 모델로 교체하거나 `temperature` 매개변수를 조정(보통 0으로 설정하여 출력의 결정성 보장)하면 직접적으로 문제가 해결되는 경우도 있다.

---

## 4.3 Plan-and-Solve

ReAct라는 반응형, 단계적 의사결정 에이전트 패러다임을 습득한 후, 이제 풍격이 다르지만 똑같이 강력한 방법인 **Plan-and-Solve**를 탐구한다. 이름에서 알 수 있듯이, 이 패러다임은 작업 처리를 명확하게 두 단계로 나눈다: **먼저 계획(Plan)하고, 이후 실행(Solve)한다.**

ReAct가 현장의 단서(Observation)를 바탕으로 단계적으로 추론하고 수시로 조사 방향을 조정하는 경험 많은 탐정과 같다면, Plan-and-Solve는 착공 전에 반드시 완전한 청사진(Plan)을 그린 다음 청사진에 따라 엄격하게 시공(Solve)하는 건축가와 더 비슷하다. 사실 지금 우리가 사용하는 많은 대형 모델 도구의 에이전트 모드에는 이 설계 패턴이 융합되어 있다.

**4.3.1 Plan-and-Solve의 작동 원리:**

Plan-and-Solve Prompting은 Lei Wang이 2023년에 제안했으며[2], 핵심 동기는 사고 연쇄가 다단계, 복잡한 문제를 처리할 때 "궤도를 벗어나기" 쉬운 문제를 해결하기 위한 것이다.

ReAct가 사고와 행동을 매 단계에 융합하는 것과 달리, Plan-and-Solve는 전체 흐름을 두 가지 핵심 단계로 분리한다. 그림 4.2와 같다:

1. **계획 단계(Planning Phase):** 먼저 에이전트는 사용자의 완전한 질문을 받는다. 첫 번째 임무는 직접 문제를 해결하거나 도구를 호출하는 것이 아니라, **문제를 분해하여 명확하고 단계적인 행동 계획을 수립**하는 것이다. 이 계획 자체가 대형 언어 모델 호출의 산물이다.
2. **실행 단계(Solving Phase):** 완전한 계획을 얻은 후 에이전트는 실행 단계에 들어간다. **계획의 단계에 따라 엄격하게 하나씩 실행**한다. 각 단계의 실행은 독립적인 LLM 호출이거나, 이전 단계 결과의 처리일 수 있으며, 계획의 모든 단계가 완료될 때까지 계속하여 최종 답을 도출한다.

이 "먼저 도모하고 후에 행동하는" 전략은 에이전트가 장기적 계획이 필요한 복잡한 작업을 처리할 때 더 높은 목표 일관성을 유지하고 중간 단계에서 방향을 잃지 않도록 한다.

이 두 단계 과정을 형식화하면 다음과 같다. 먼저 계획 모델 $\pi_{\text{plan}}$이 원래 질문 $q$를 바탕으로 $n$개의 단계를 포함하는 계획 $P = (p_1, p_2, \dots, p_n)$을 생성한다:

$$P = \pi_{\text{plan}}(q)$$

그 후 실행 단계에서, 실행 모델 $\pi_{\text{solve}}$가 계획의 단계를 하나씩 완성한다. $i$번째 단계의 해결책 $s_i$ 생성은 원래 질문 $q$, 완전한 계획 $P$, 그리고 이전 모든 단계의 실행 결과 $(s_1, \dots, s_{i-1})$에 동시에 의존한다:

$$s_i = \pi_{\text{solve}}(q, P, (s_1, \dots, s_{i-1}))$$

최종 답은 마지막 단계의 실행 결과 $s_n$이다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-2.png" alt="Plan-and-Solve 패러다임의 두 단계 작업 흐름" width="90%"/>
  <p>그림 4.2 Plan-and-Solve 패러다임의 두 단계 작업 흐름</p>
</div>

Plan-and-Solve는 특히 구조가 강하고 명확하게 분해할 수 있는 복잡한 작업에 적합하다. 예를 들면:

- **다단계 수학 응용 문제:** 먼저 계산 단계를 나열한 뒤 하나씩 풀어야 한다.
- **여러 정보 소스를 통합해야 하는 보고서 작성:** 먼저 보고서 구조(서론, 데이터 소스 A, 데이터 소스 B, 결론)를 계획한 뒤 하나씩 내용을 채운다.
- **코드 생성 작업:** 먼저 함수, 클래스, 모듈의 구조를 구상한 뒤 하나씩 구현한다.

**4.3.2 계획 단계:**

Plan-and-Solve 패러다임이 구조화된 추론 작업에서 발휘하는 장점을 부각하기 위해, 도구를 사용하지 않고 프롬프트 설계를 통해 추론 작업을 완성한다.

이러한 작업의 특징은 단일 조회나 계산으로는 답을 얻을 수 없으며, 먼저 문제를 논리적으로 연결된 일련의 하위 단계로 분해한 뒤 순서대로 풀어야 한다는 것이다. 이는 Plan-and-Solve의 "먼저 계획하고, 이후 실행하는" 핵심 능력을 발휘할 수 있는 기회다.

**목표 문제는 다음과 같다:** "한 과일 가게가 월요일에 사과 15개를 팔았다. 화요일에 판 사과 수는 월요일의 두 배였다. 수요일에 판 수는 화요일보다 5개가 적었다. 그렇다면 이 세 날 동안 총 몇 개의 사과를 팔았는가?"

이 문제는 대형 언어 모델에게 특별히 어렵지는 않지만, 참고할 수 있는 명확한 논리 체인을 포함한다. 어떤 실제 논리 난제에서 대형 모델이 높은 품질의 정확한 답을 추론해낼 수 없는 경우, 이 설계 패턴을 참고하여 자신만의 에이전트를 설계할 수 있다. 에이전트는 다음이 필요하다:

1. **계획 단계:** 먼저 문제를 세 가지 독립된 계산 단계(화요일 판매량 계산, 수요일 판매량 계산, 총 판매량 계산)로 분해한다.
2. **실행 단계:** 그런 다음 계획에 따라 단계별로 계산을 실행하고, 각 단계의 결과를 다음 단계의 입력으로 삼아 최종 합계를 도출한다.

계획 단계의 목표는 대형 언어 모델이 원래 문제를 받아 명확하고 단계별 행동 계획을 출력하게 하는 것이다. 이 계획은 반드시 구조화되어 있어야 코드로 쉽게 파싱하고 하나씩 실행할 수 있다. 따라서 설계하는 프롬프트는 모델의 역할과 임무를 명확히 알려주고 출력 형식의 예시를 제공해야 한다.

````python
PLANNER_PROMPT_TEMPLATE = """
당신은 최고의 AI 계획 전문가다. 당신의 임무는 사용자가 제기한 복잡한 문제를 여러 간단한 단계로 구성된 행동 계획으로 분해하는 것이다.
계획의 각 단계가 독립적이고 실행 가능한 하위 작업이며, 논리적 순서에 따라 엄격하게 배열되었는지 확인하라.
출력은 반드시 Python 리스트여야 하며, 각 요소는 하위 작업을 설명하는 문자열이다.

문제: {question}

다음 형식을 엄격히 따라 계획을 출력하라. ```python과 ```는 필수 접두사와 접미사다:
```python
["단계1", "단계2", "단계3", ...]
```
"""
````

이 프롬프트는 다음과 같은 방법으로 출력의 품질과 안정성을 보장한다:

- **역할 설정:** "최고의 AI 계획 전문가"로 모델의 전문적 능력을 활성화한다.
- **작업 설명:** "문제 분해"의 목표를 명확히 정의한다.
- **형식 제약:** Python 리스트 형식의 문자열 출력을 강제하여, 이후 코드의 파싱 작업을 크게 단순화하고 자연어 파싱보다 더 안정적이고 신뢰할 수 있게 한다.

이제 이 프롬프트 로직을 `Planner` 클래스로 캡슐화한다. 이 클래스가 우리의 계획기다.

```python
# llm_client.py의 HelloAgentsLLM 클래스가 이미 정의되어 있다고 가정
# from llm_client import HelloAgentsLLM

class Planner:
    def __init__(self, llm_client):
        self.llm_client = llm_client

    def plan(self, question: str) -> list[str]:
        """
        사용자 질문에 따라 행동 계획을 생성한다.
        """
        prompt = PLANNER_PROMPT_TEMPLATE.format(question=question)
        
        # 계획 생성을 위해 간단한 메시지 리스트를 구성한다
        messages = [{"role": "user", "content": prompt}]
        
        print("--- 계획을 생성 중 ---")
        # 스트리밍 출력으로 완전한 계획을 가져온다
        response_text = self.llm_client.think(messages=messages) or ""
        
        print(f"✅ 계획이 생성되었다:\n{response_text}")
        
        # LLM 출력의 리스트 문자열을 파싱한다
        try:
            # ```python과 ``` 사이의 내용을 찾는다
            plan_str = response_text.split("```python")[1].split("```")[0].strip()
            # ast.literal_eval을 사용하여 문자열을 안전하게 Python 리스트로 변환한다
            plan = ast.literal_eval(plan_str)
            return plan if isinstance(plan, list) else []
        except (ValueError, SyntaxError, IndexError) as e:
            print(f"❌ 계획 파싱 중 오류 발생: {e}")
            print(f"원시 응답: {response_text}")
            return []
        except Exception as e:
            print(f"❌ 계획 파싱 중 알 수 없는 오류 발생: {e}")
            return []
```

**4.3.3 실행기와 상태 관리:**

계획기(`Planner`)가 명확한 행동 청사진을 생성한 후, 실행기(`Executor`)가 계획의 작업을 하나씩 완성한다. 실행기는 대형 언어 모델을 호출하여 각 하위 문제를 해결하는 것뿐만 아니라, 매우 중요한 역할인 **상태 관리**도 담당한다. 각 단계의 실행 결과를 기록하고 이를 후속 단계의 컨텍스트로 제공하여, 전체 작업 체인에서 정보가 원활하게 흐르도록 보장한다.

실행기의 프롬프트는 계획기와 다르다. 목표는 문제를 분해하는 것이 아니라, **기존 컨텍스트를 바탕으로 현재 이 하나의 단계에 집중하여 해결**하는 것이다. 따라서 프롬프트는 다음 핵심 정보를 포함해야 한다:

- **원래 질문:** 모델이 항상 최종 목표를 인지하도록 한다.
- **완전한 계획:** 모델이 현재 단계가 전체 작업에서 어느 위치에 있는지 파악하게 한다.
- **이력 단계와 결과:** 지금까지 완료된 작업을 제공하여 현재 단계의 직접적인 입력으로 삼는다.
- **현재 단계:** 모델이 지금 어떤 구체적인 작업을 해결해야 하는지 명확히 지시한다.

```python
EXECUTOR_PROMPT_TEMPLATE = """
당신은 최고의 AI 실행 전문가다. 당신의 임무는 주어진 계획에 따라 문제를 단계적으로 해결하는 것이다.
당신은 원래 질문, 완전한 계획, 지금까지 완료된 단계와 결과를 받게 된다.
"현재 단계"를 해결하는 데 집중하고, 해당 단계의 최종 답만 출력하며, 추가적인 설명이나 대화는 출력하지 마라.

# 원래 질문:
{question}

# 완전한 계획:
{plan}

# 이력 단계와 결과:
{history}

# 현재 단계:
{current_step}

"현재 단계"에 대한 답만 출력하라:
"""
```

실행 로직을 `Executor` 클래스에 캡슐화한다. 이 클래스는 계획을 순환하며 LLM을 호출하고 이력 기록(상태)을 유지한다.

```python
class Executor:
    def __init__(self, llm_client):
        self.llm_client = llm_client

    def execute(self, question: str, plan: list[str]) -> str:
        """
        계획에 따라 단계적으로 실행하여 문제를 해결한다.
        """
        history = "" # 이력 단계와 결과를 저장하는 문자열
        
        print("\n--- 계획을 실행 중 ---")
        
        for i, step in enumerate(plan):
            print(f"\n-> 단계 {i+1}/{len(plan)} 실행 중: {step}")
            
            prompt = EXECUTOR_PROMPT_TEMPLATE.format(
                question=question,
                plan=plan,
                history=history if history else "없음", # 첫 단계면 이력 없음
                current_step=step
            )
            
            messages = [{"role": "user", "content": prompt}]
            
            response_text = self.llm_client.think(messages=messages) or ""
            
            # 다음 단계를 위해 이력 기록 업데이트
            history += f"단계 {i+1}: {step}\n결과: {response_text}\n\n"
            
            print(f"✅ 단계 {i+1} 완료, 결과: {response_text}")

        # 루프 종료 후, 마지막 단계의 응답이 최종 답변이다
        final_answer = response_text
        return final_answer
```

"계획" 담당 `Planner`와 "실행" 담당 `Executor`를 각각 구축했다. 마지막 단계는 이 두 컴포넌트를 통합된 에이전트 `PlanAndSolveAgent`에 통합하고 문제를 해결하는 완전한 능력을 부여하는 것이다. 주 클래스 `PlanAndSolveAgent`를 만든다. 이 클래스의 역할은 매우 명확하다: LLM 클라이언트를 받아, 내부 계획기와 실행기를 초기화하고, 전체 흐름을 시작하는 간단한 `run` 메서드를 제공한다.

```python
class PlanAndSolveAgent:
    def __init__(self, llm_client):
        """
        에이전트를 초기화하고 동시에 계획기와 실행기 인스턴스를 생성한다.
        """
        self.llm_client = llm_client
        self.planner = Planner(self.llm_client)
        self.executor = Executor(self.llm_client)

    def run(self, question: str):
        """
        에이전트의 완전한 흐름을 실행한다: 먼저 계획하고, 이후 실행한다.
        """
        print(f"\n--- 문제 처리 시작 ---\n문제: {question}")
        
        # 1. 계획기 호출하여 계획 생성
        plan = self.planner.plan(question)
        
        # 계획이 성공적으로 생성되었는지 확인
        if not plan:
            print("\n--- 작업 종료 --- \n유효한 행동 계획을 생성할 수 없었다.")
            return

        # 2. 실행기 호출하여 계획 실행
        final_answer = self.executor.execute(question, plan)
        
        print(f"\n--- 작업 완료 ---\n최종 답변: {final_answer}")
```

이 `PlanAndSolveAgent` 클래스의 설계는 "상속보다 조합" 원칙을 구현한다. 클래스 자체에는 복잡한 로직이 없고, 조율자(Orchestrator)로서 내부 컴포넌트를 명확하게 호출하여 작업을 완성한다.

**4.3.4 실행 인스턴스와 분석:**

완전한 코드는 이 책의 코드 저장소 `code` 폴더를 참고하며, 여기서는 최종 결과만 시연한다.

````bash
--- 문제 처리 시작 ---
문제: 한 과일 가게가 월요일에 사과 15개를 팔았다. 화요일에 판 사과 수는 월요일의 두 배였다. 수요일에 판 수는 화요일보다 5개가 적었다. 그렇다면 이 세 날 동안 총 몇 개의 사과를 팔았는가?
--- 계획을 생성 중 ---
🧠 xxxx 모델을 호출 중...
✅ 대형 언어 모델 응답 성공:
```python
["월요일에 판 사과 수 계산: 15개", "화요일에 판 사과 수 계산: 월요일 수 × 2 = 15 × 2 = 30개", "수요일에 판 사과 수 계산: 화요일 수 - 5 = 30 - 5 = 25개", "3일 총 판매량 계산: 월요일 + 화요일 + 수요일 = 15 + 30 + 25 = 70개"]
```
✅ 계획이 생성되었다:
```python
["월요일에 판 사과 수 계산: 15개", "화요일에 판 사과 수 계산: 월요일 수 × 2 = 15 × 2 = 30개", "수요일에 판 사과 수 계산: 화요일 수 - 5 = 30 - 5 = 25개", "3일 총 판매량 계산: 월요일 + 화요일 + 수요일 = 15 + 30 + 25 = 70개"]
```

--- 계획을 실행 중 ---

-> 단계 1/4 실행 중: 월요일에 판 사과 수 계산: 15개
🧠 xxxx 모델을 호출 중...
✅ 대형 언어 모델 응답 성공:
15
✅ 단계 1 완료, 결과: 15

-> 단계 2/4 실행 중: 화요일에 판 사과 수 계산: 월요일 수 × 2 = 15 × 2 = 30개
🧠 xxxx 모델을 호출 중...
✅ 대형 언어 모델 응답 성공:
30
✅ 단계 2 완료, 결과: 30

-> 단계 3/4 실행 중: 수요일에 판 사과 수 계산: 화요일 수 - 5 = 30 - 5 = 25개
🧠 xxxx 모델을 호출 중...
✅ 대형 언어 모델 응답 성공:
25
✅ 단계 3 완료, 결과: 25

-> 단계 4/4 실행 중: 3일 총 판매량 계산: 월요일 + 화요일 + 수요일 = 15 + 30 + 25 = 70개
🧠 xxxx 모델을 호출 중...
✅ 대형 언어 모델 응답 성공:
70
✅ 단계 4 완료, 결과: 70

--- 작업 완료 ---
최종 답변: 70
````

위 출력 로그에서 Plan-and-Solve 패러다임의 작업 흐름을 명확히 볼 수 있다:

1. **계획 단계:** 에이전트가 먼저 `Planner`를 호출하여 복잡한 응용 문제를 4개의 논리적 단계를 포함하는 Python 리스트로 성공적으로 분해했다. 이 구조화된 계획이 이후 실행의 기반을 마련했다.
2. **실행 단계:** `Executor`가 생성된 계획에 따라 엄격하게 단계별로 실행했다. 각 단계에서 이력 결과를 컨텍스트로 삼아 정보의 올바른 전달을 보장했다(예: 단계 2는 단계 1의 결과 "15개"를 올바르게 사용했고, 단계 3도 단계 2의 결과 "30개"를 올바르게 사용했다).
3. **결과:** 전체 과정이 논리적으로 명확하고 단계가 분명하며, 에이전트는 최종적으로 정확한 답변 "70개"를 도출했다.

---

## 4.4 Reflection

앞서 구현한 ReAct와 Plan-and-Solve 패러다임에서, 에이전트는 작업을 완료하면 작업 흐름이 종료된다. 그러나 이들이 생성한 초기 답변은, 행동 궤적이든 최종 결과든, 오류가 있거나 개선의 여지가 있을 수 있다. Reflection 메커니즘의 핵심 사상은 에이전트에게 **사후(post-hoc) 자기 교정 루프**를 도입하여, 인간처럼 자신의 작업을 검토하고 부족한 점을 발견하며 반복적으로 최적화할 수 있게 하는 것이다.

**4.4.1 Reflection 메커니즘의 핵심 사상:**

Reflection 메커니즘은 인간의 학습 과정에서 영감을 받았다: 초안 작성 후 교정하고, 수학 문제 풀이 후 검산한다. 이 사상은 여러 연구에서 구현되었는데, 예를 들어 Shinn, Noah가 2023년에 제안한 Reflexion 프레임워크[3]가 있다. 핵심 작업 흐름은 간결한 세 단계 루프로 요약할 수 있다: **실행 -> 반성 -> 최적화**.

1. **실행(Execution):** 먼저 에이전트가 우리에게 익숙한 방법(ReAct 또는 Plan-and-Solve)을 사용하여 작업을 완수하려 시도하고, 초기 해결책이나 행동 궤적을 생성한다. 이를 "초안"으로 볼 수 있다.
2. **반성(Reflection):** 그 다음 에이전트는 반성 단계에 들어간다. 독립적이거나 특수한 프롬프트를 가진 대형 언어 모델 인스턴스를 호출하여 "심사자" 역할을 맡긴다. 이 "심사자"는 첫 단계에서 생성된 "초안"을 검토하고 여러 차원에서 평가한다. 예를 들면: 사실적 오류(상식이나 알려진 사실과 상충하는가?), 논리적 허점(추론 과정에 불일관성이나 모순이 있는가?), 효율성 문제(더 직접적이고 간결한 경로가 있는가?), 누락된 정보(질문의 핵심 제약이나 측면을 무시했는가?). 평가에 따라 구체적인 문제점과 개선 제안을 담은 구조화된 **피드백(Feedback)**을 생성한다.
3. **최적화(Refinement):** 마지막으로 에이전트는 "초안"과 "피드백"을 새로운 컨텍스트로 삼아 대형 언어 모델을 다시 호출하고, 피드백 내용에 따라 초안을 수정하여 더 완성도 높은 "수정본"을 생성한다.

그림 4.3과 같이, 이 루프는 여러 번 반복할 수 있으며, 반성 단계에서 새로운 문제가 발견되지 않거나 미리 설정한 반복 횟수 한도에 도달할 때까지 계속한다. 이 반복 최적화 과정을 형식화하면 다음과 같다. $O_i$를 $i$번째 반복에서 생성된 출력($O_0$은 초기 출력)이라 하면, 반성 모델 $\pi_{\text{reflect}}$가 $O_i$에 대한 피드백 $F_i$를 생성한다:

$$F_i = \pi_{\text{reflect}}(\text{Task}, O_i)$$

그런 다음 최적화 모델 $\pi_{\text{refine}}$이 원래 작업, 이전 버전 출력, 피드백을 결합하여 새 버전의 출력 $O_{i+1}$을 생성한다:

$$O_{i+1} = \pi_{\text{refine}}(\text{Task}, O_i, F_i)$$

<div align="center">
<img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-3.png" alt="Reflection 메커니즘의 실행-반성-최적화 반복 순환" width="70%"/>
<p>그림 4.3 Reflection 메커니즘의 "실행-반성-최적화" 반복 순환</p>
</div>

앞의 두 패러다임과 비교하여 Reflection의 가치는 다음에 있다:

- 에이전트에게 내부 오류 수정 회로를 제공하여, 외부 도구의 피드백(ReAct의 Observation)에 완전히 의존하지 않아도 더 높은 수준의 논리 및 전략 오류를 수정할 수 있다.
- 일회성 작업 실행을 지속적인 최적화 과정으로 변환하여, 복잡한 작업의 최종 성공률과 답변 품질을 크게 향상시킨다.
- 에이전트에게 임시 **"단기 기억"**을 구성한다. 전체 "실행-반성-최적화" 궤적은 귀중한 경험 기록을 형성하며, 에이전트는 최종 답을 알 뿐만 아니라 결함 있는 초안에서 최종 버전으로 어떻게 반복했는지도 기억한다. 더 나아가, 이 기억 시스템은 **다중 모달**일 수도 있어, 에이전트가 텍스트 이외의 출력(코드, 이미지 등)을 반성하고 수정할 수 있게 하며, 더 강력한 다중 모달 에이전트 구축의 기반을 마련한다.

**4.4.2 케이스 설정과 메모리 모듈 설계:**

Reflection 메커니즘을 실전에서 구현하기 위해, 기억 관리 메커니즘을 도입한다. 왜냐하면 Reflection은 보통 정보의 저장과 추출과 대응하며, 컨텍스트가 충분히 긴 경우 "심사자"가 모든 정보를 직접 가져와 반성하게 하면 많은 중복 정보를 전달하는 경우가 많기 때문이다. 이 실습은 주로 **코드 생성과 반복 최적화**를 완성한다.

이 단계의 목표 작업은 다음과 같다: "1부터 n 사이의 모든 소수(prime numbers)를 찾는 Python 함수를 작성하라."

이 작업은 Reflection 메커니즘을 검증하는 탁월한 시나리오다:

1. **명확한 최적화 경로 존재:** 대형 언어 모델이 처음 생성하는 코드는 기능은 하지만 효율성이 낮은 구현일 가능성이 높다.
2. **반성 포인트 명확:** 반성을 통해 "시간 복잡도가 너무 높다"거나 "중복 계산이 있다"는 문제를 발견할 수 있다.
3. **최적화 방향 명확:** 피드백에 따라 더 효율적인 반복 버전이나 메모이제이션 패턴을 사용하는 버전으로 최적화할 수 있다.

Reflection의 핵심은 반복에 있으며, 반복의 전제는 이전 시도와 받은 피드백을 기억할 수 있어야 한다는 것이다. 따라서 "단기 기억" 모듈은 이 패러다임을 구현하는 데 필수적이다. 이 기억 모듈은 각 "실행-반성" 루프의 완전한 궤적을 저장하는 역할을 한다.

```python
from typing import List, Dict, Any, Optional

class Memory:
    """
    에이전트의 행동과 반성 궤적을 저장하는 간단한 단기 기억 모듈.
    """

    def __init__(self):
        """
        모든 기록을 저장하는 빈 리스트를 초기화한다.
        """
        self.records: List[Dict[str, Any]] = []

    def add_record(self, record_type: str, content: str):
        """
        기억에 새 기록을 추가한다.

        매개변수:
        - record_type (str): 기록 유형 ('execution' 또는 'reflection').
        - content (str): 기록의 구체적인 내용 (예: 생성된 코드 또는 반성 피드백).
        """
        record = {"type": record_type, "content": content}
        self.records.append(record)
        print(f"📝 기억이 업데이트되었다, '{record_type}' 기록 1건 추가됨.")

    def get_trajectory(self) -> str:
        """
        모든 기억 기록을 연속된 문자열 텍스트로 형식화하여 프롬프트 구성에 사용한다.
        """
        trajectory_parts = []
        for record in self.records:
            if record['type'] == 'execution':
                trajectory_parts.append(f"--- 이전 라운드 시도 (코드) ---\n{record['content']}")
            elif record['type'] == 'reflection':
                trajectory_parts.append(f"--- 심사자 피드백 ---\n{record['content']}")
        
        return "\n\n".join(trajectory_parts)

    def get_last_execution(self) -> Optional[str]:
        """
        가장 최근의 실행 결과 (예: 최신 생성 코드)를 가져온다.
        없으면 None을 반환한다.
        """
        for record in reversed(self.records):
            if record['type'] == 'execution':
                return record['content']
        return None
```

이 `Memory` 클래스의 설계는 비교적 간결하며, 주체는 다음과 같다:

- 리스트 `records`를 사용하여 각 행동과 반성을 순서대로 저장한다.
- `add_record` 메서드가 기억에 새 항목을 추가한다.
- `get_trajectory` 메서드가 핵심으로, 기억 궤적을 텍스트로 "직렬화"하여 후속 프롬프트에 직접 삽입할 수 있게 하고, 모델의 반성과 최적화에 완전한 컨텍스트를 제공한다.
- `get_last_execution`은 반성을 위한 최신 "초안"을 쉽게 가져올 수 있게 한다.

**4.4.3 Reflection 에이전트의 코딩 구현:**

`Memory` 모듈을 기반으로 `ReflectionAgent`의 핵심 로직을 구축한다. 전체 에이전트의 작업 흐름은 앞서 논의한 "실행-반성-최적화" 루프를 중심으로 전개되며, 정교하게 설계된 프롬프트를 통해 대형 언어 모델이 다른 역할을 맡도록 안내한다.

**(1) 프롬프트 설계**

이전 패러다임과 달리, Reflection 메커니즘은 여러 가지 다른 역할의 프롬프트가 협력해야 한다.

1. **초기 실행 프롬프트(Execution Prompt):** 에이전트가 처음 문제를 해결하려는 프롬프트로, 내용이 비교적 직접적이며 모델에게 지정된 작업을 완수하도록 요청한다.

```bash
INITIAL_PROMPT_TEMPLATE = """
당신은 경험이 풍부한 Python 프로그래머다. 다음 요구에 따라 Python 함수를 작성하라.
코드는 완전한 함수 시그니처, 문서 문자열을 포함하고 PEP 8 코딩 규범을 준수해야 한다.

요구: {task}

코드만 직접 출력하고 추가적인 설명을 포함하지 마라.
"""
```

2. **반성 프롬프트(Reflection Prompt):** 이 프롬프트가 Reflection 메커니즘의 영혼이다. 모델이 "코드 심사자" 역할을 맡아 이전 라운드에서 생성한 코드를 비판적으로 분석하고 구체적이고 실행 가능한 피드백을 제공하도록 지시한다.

````bash
REFLECT_PROMPT_TEMPLATE = """
당신은 매우 엄격한 코드 심사 전문가이자 시니어 알고리즘 엔지니어로, 코드 성능에 극도로 높은 요구를 가진다.
당신의 임무는 다음 Python 코드를 검토하고 <strong>알고리즘 효율성</strong>의 주요 병목을 찾는 데 집중하는 것이다.

# 원래 작업:
{task}

# 검토할 코드:
```python
{code}
```

해당 코드의 시간 복잡도를 분석하고, 성능을 크게 향상시킬 <strong>알고리즘적으로 더 우수한</strong> 해결책이 있는지 생각하라.
있다면 현재 알고리즘의 부족한 점을 명확히 지적하고 구체적이고 실행 가능한 개선 알고리즘 제안을 제시하라(예: 시험 나누기법 대신 체법 사용).
코드가 알고리즘 층면에서 이미 최적이라면 "개선 불필요"라고 답할 수 있다.

추가적인 설명 없이 피드백만 직접 출력하라.
"""
````

3. **최적화 프롬프트(Refinement Prompt):** 피드백을 받은 후, 이 프롬프트가 모델을 안내하여 피드백 내용에 따라 기존 코드를 수정하고 최적화한다.

````bash
REFINE_PROMPT_TEMPLATE = """
당신은 경험이 풍부한 Python 프로그래머다. 코드 심사 전문가의 피드백에 따라 코드를 최적화하고 있다.

# 원래 작업:
{task}

# 이전 라운드 시도한 코드:
{last_code_attempt}
심사자 피드백:
{feedback}

심사자의 피드백에 따라 최적화된 새 버전 코드를 생성하라.
코드는 완전한 함수 시그니처, 문서 문자열을 포함하고 PEP 8 코딩 규범을 준수해야 한다.
최적화된 코드만 직접 출력하고 추가적인 설명은 포함하지 마라.
"""
````

**(2) 에이전트 캡슐화와 구현**

이 프롬프트 로직과 `Memory` 모듈을 `ReflectionAgent` 클래스에 통합한다.

```python
# llm_client.py와 memory.py가 정의되어 있다고 가정
# from llm_client import HelloAgentsLLM
# from memory import Memory

class ReflectionAgent:
    def __init__(self, llm_client, max_iterations=3):
        self.llm_client = llm_client
        self.memory = Memory()
        self.max_iterations = max_iterations

    def run(self, task: str):
        print(f"\n--- 작업 처리 시작 ---\n작업: {task}")

        # --- 1. 초기 실행 ---
        print("\n--- 초기 시도 중 ---")
        initial_prompt = INITIAL_PROMPT_TEMPLATE.format(task=task)
        initial_code = self._get_llm_response(initial_prompt)
        self.memory.add_record("execution", initial_code)

        # --- 2. 반복 루프: 반성과 최적화 ---
        for i in range(self.max_iterations):
            print(f"\n--- {i+1}/{self.max_iterations} 라운드 반복 ---")

            # a. 반성
            print("\n-> 반성 중...")
            last_code = self.memory.get_last_execution()
            reflect_prompt = REFLECT_PROMPT_TEMPLATE.format(task=task, code=last_code)
            feedback = self._get_llm_response(reflect_prompt)
            self.memory.add_record("reflection", feedback)

            # b. 종료 여부 확인
            if "개선 불필요" in feedback:
                print("\n✅ 반성 결과 코드가 더 이상 개선이 필요 없으므로 작업이 완료된다.")
                break

            # c. 최적화
            print("\n-> 최적화 중...")
            refine_prompt = REFINE_PROMPT_TEMPLATE.format(
                task=task,
                last_code_attempt=last_code,
                feedback=feedback
            )
            refined_code = self._get_llm_response(refine_prompt)
            self.memory.add_record("execution", refined_code)
        
        final_code = self.memory.get_last_execution()
        print(f"\n--- 작업 완료 ---\n최종 생성 코드:\n```python\n{final_code}\n```")
        return final_code

    def _get_llm_response(self, prompt: str) -> str:
        """LLM을 호출하여 완전한 스트리밍 응답을 가져오는 보조 메서드."""
        messages = [{"role": "user", "content": prompt}]
        response_text = self.llm_client.think(messages=messages) or ""
        return response_text
```

**4.4.4 실행 인스턴스와 분석:**

완전한 코드는 이 책의 코드 저장소 `code` 폴더를 참고하며, 여기서 출력 인스턴스를 제공한다.

````python
--- 작업 처리 시작 ---
작업: 1부터 n 사이의 모든 소수(prime numbers)를 찾는 Python 함수를 작성하라.

--- 초기 시도 중 ---
🧠 xxxxxx 모델을 호출 중...
✅ 대형 언어 모델 응답 성공:
```python
def find_primes(n):
    ...
    return primes
```
📝 기억이 업데이트되었다, 'execution' 기록 1건 추가됨.   

--- 1/2 라운드 반복 ---

-> 반성 중...
🧠 xxxxxx 모델을 호출 중...
✅ 대형 언어 모델 응답 성공:
현재 코드의 시간 복잡도는 O(n * sqrt(n))이다. 작은 n 값에는 허용 가능하지만, n이 매우 클 때 성능이 크게 저하된다. 주요 병목은 각 수마다 시험 나누기로 확인해야 해서 시간 오버헤드가 높다는 것이다.

에라토스테네스의 체(Sieve of Eratosthenes) 사용을 권장한다. 이 알고리즘의 시간 복잡도는 O(n log(log n))으로, 소수 탐색 효율을 크게 향상시킨다.

개선된 코드:
```python
def find_primes(n):
    ...
    return primes
```
📝 기억이 업데이트되었다, 'reflection' 기록 1건 추가됨.

-> 최적화 중...
🧠 xxxxxx 모델을 호출 중...
✅ 대형 언어 모델 응답 성공:
```python
def find_primes(n):
    ...
    return primes
```
📝 기억이 업데이트되었다, 'execution' 기록 1건 추가됨.

--- 2/2 라운드 반복 ---

-> 반성 중...
🧠 xxxxxx 모델을 호출 중...
✅ 대형 언어 모델 응답 성공:
현재 코드는 에라토스테네스의 체를 사용하며, 시간 복잡도는 O(n log log n), 공간 복잡도는 O(n)이다. 이 알고리즘은 1부터 n 사이의 모든 소수를 찾는 데 이미 매우 효율적이며, 일반적으로 추가 최적화가 필요하지 않다. 단 특정 시나리오에서 다음 개선을 고려할 수 있다:

1. <strong>분할 체(Segmented Sieve):</strong> n이 매우 크지만 메모리가 제한된 경우에 적합하다. 구간을 여러 작은 단락으로 나눠 각각 체법으로 처리하여 메모리 사용을 줄인다.
2. <strong>홀수 체(Odd Number Sieve):</strong> 2를 제외한 모든 소수는 홀수다. `is_prime` 배열을 초기화할 때 홀수만 표시하면 공간 복잡도를 절반으로 줄이고 불필요한 계산을 줄일 수 있다.

그러나 이러한 개선은 대부분의 응용 시나리오에서 필요하지 않다. 표준 에라토스테네스의 체가 이미 충분히 효율적이기 때문이다. 따라서 일반적인 상황에서는 <strong>개선 불필요</strong>다.
📝 기억이 업데이트되었다, 'reflection' 기록 1건 추가됨.

✅ 반성 결과 코드가 더 이상 개선이 필요 없으므로 작업이 완료된다.

--- 작업 완료 ---
최종 생성 코드:
```python
def find_primes(n):
    """
    에라토스테네스의 체 알고리즘을 사용하여 1부터 n 사이의 모든 소수를 찾는다.

    :param n: 소수를 찾을 범위의 상한.
    :return: 1부터 n 사이의 모든 소수 리스트.
    """
    if n < 2:
        return []

    is_prime = [True] * (n + 1)
    is_prime[0] = is_prime[1] = False

    p = 2
    while p * p <= n:
        if is_prime[p]:
            for i in range(p * p, n + 1, p):
                is_prime[i] = False
        p += 1

    primes = [num for num in range(2, n + 1) if is_prime[num]]
    return primes
```
````

이 실행 인스턴스는 Reflection 메커니즘이 에이전트의 심층 최적화를 어떻게 이끄는지 보여준다:

1. **효과적인 "비판"이 최적화의 전제:** 첫 번째 라운드 반성에서, "매우 엄격하고" "알고리즘 효율성에 집중하는" 프롬프트를 사용했기 때문에 에이전트는 기능이 올바른 초기 코드에 만족하지 않고, `O(n * sqrt(n))`의 시간 복잡도 병목을 정확히 지적하고 알고리즘 수준의 개선 제안인 에라토스테네스의 체를 제시했다.
2. **반복적인 개선:** 에이전트는 명확한 피드백을 받은 후 최적화 단계에서 성공적으로 더 효율적인 체법을 구현하여 알고리즘 복잡도를 `O(n log log n)`으로 낮추는 첫 번째 의미 있는 자기 반복을 완성했다.
3. **수렴과 종료:** 두 번째 라운드 반성에서 에이전트는 이미 효율적인 체법에 직면하여 더 깊은 지식을 발휘했다. 현재 알고리즘의 효율성을 인정할 뿐만 아니라 분할 체 등 더 고급 최적화 방향도 언급했지만, 최종적으로 "일반적인 상황에서 개선 불필요"라는 올바른 판단을 내렸다. 이 판단이 종료 조건을 트리거하여 최적화 과정이 수렴되었다.

이 사례는 잘 설계된 Reflection 메커니즘의 가치가 오류 수정에만 있는 것이 아니라, **해결책을 품질과 효율성 면에서 계단식으로 향상시키는 데** 있음을 충분히 증명한다. 이것이 Reflection을 복잡하고 고품질 에이전트를 구축하는 핵심 기술 중 하나로 만든다.

**4.4.5 Reflection 메커니즘의 비용-편익 분석:**

Reflection 메커니즘이 작업 해결 품질 향상에 탁월하지만, 이 능력의 획득에는 대가가 따른다. 실제 응용에서는 그 편익과 상응하는 비용을 저울질해야 한다.

**(1) 주요 비용**

1. **모델 호출 오버헤드 증가:** 가장 직접적인 비용이다. 매 반복마다 최소 두 번의 추가 대형 언어 모델 호출이 필요하다(한 번은 반성, 한 번은 최적화). 여러 라운드를 반복하면 API 호출 비용과 계산 자원 소비가 배로 늘어난다.
2. **작업 지연 현저히 증가:** Reflection은 직렬 과정으로, 각 라운드의 최적화는 반드시 이전 라운드의 반성이 완료될 때까지 기다려야 한다. 이로 인해 총 소요 시간이 현저히 늘어나, 실시간성에 높은 요구를 가진 시나리오에는 적합하지 않다.
3. **프롬프트 엔지니어링 복잡도 상승:** 우리의 케이스에서 볼 수 있듯이, Reflection의 성공은 고품질이고 목표 지향적인 프롬프트에 크게 의존한다. "실행", "반성", "최적화" 등 다른 단계에 효과적인 프롬프트를 설계하고 디버깅하는 데 더 많은 개발 노력이 필요하다.

**(2) 핵심 편익**

1. **해결책 품질의 도약:** 가장 큰 편익은 "합격" 수준의 초기 방안을 "우수" 수준의 최종 방안으로 반복 최적화할 수 있다는 것이다. 기능 정확에서 성능 효율로, 논리 거침에서 논리 엄밀로의 향상은 많은 핵심 작업에서 매우 중요하다.
2. **견고성과 신뢰성 강화:** 내부 자기 교정 루프를 통해 에이전트는 초기 방안에 존재할 수 있는 논리 허점, 사실적 오류, 경계 조건 처리 부적절 등 문제를 발견하고 수정할 수 있어, 최종 결과의 신뢰성을 크게 높인다.

종합하면, Reflection 메커니즘은 전형적인 "비용으로 품질을 교환하는" 전략이다. **최종 결과의 품질, 정확성, 신뢰성에 극도로 높은 요구를 가지며, 작업 완료의 실시간성에 대한 요구는 상대적으로 낮은** 시나리오에 매우 적합하다. 예를 들면:

- 핵심 비즈니스 코드나 기술 보고서 생성
- 과학 연구에서 복잡한 논리 추론
- 깊은 분석과 계획이 필요한 의사결정 지원 시스템

반대로, 빠른 응답이 필요하거나 "대략 맞는" 답으로 충분한 경우, 더 경량한 ReAct나 Plan-and-Solve 패러다임을 사용하는 것이 더 비용 효율적인 선택일 수 있다.

---

## 4.5 본 장 요약

이 장에서는 제3장에서 습득한 대형 언어 모델 지식을 기반으로, "직접 바퀴를 만드는" 방식으로 처음부터 세 가지 업계의 고전 에이전트 구축 패러다임인 ReAct, Plan-and-Solve, Reflection을 코딩으로 구현했다. 핵심 작동 원리를 탐구했을 뿐만 아니라, 구체적인 실전 케이스를 통해 각각의 장점, 한계, 적용 시나리오를 깊이 이해했다.

**핵심 지식 복습:**

1. **ReAct:** 외부 세계와 상호작용할 수 있는 ReAct 에이전트를 구축했다. "사고-행동-관찰"의 동적 루프를 통해 검색 엔진을 이용하여 자체 지식 베이스로는 커버할 수 없는 실시간 질문에 성공적으로 답했다. 핵심 장점은 **환경 적응성**과 **동적 오류 수정 능력**이며, 탐색적이고 외부 도구 입력이 필요한 작업에 가장 적합하다.
2. **Plan-and-Solve:** 먼저 계획하고 이후 실행하는 Plan-and-Solve 에이전트를 구현하여 다단계 추론이 필요한 수학 응용 문제를 해결했다. 복잡한 작업을 명확한 단계로 분해하고 하나씩 실행한다. 핵심 장점은 **구조성**과 **안정성**이며, 논리 경로가 확정적이고 내부 추론이 집중적인 작업에 특히 적합하다.
3. **Reflection (자기 반성과 반복):** 자기 최적화 능력을 갖춘 Reflection 에이전트를 구축했다. "실행-반성-최적화" 반복 루프를 통해 효율성이 낮은 초기 코드 방안을 알고리즘적으로 더 우수한 고성능 버전으로 성공적으로 최적화했다. 핵심 가치는 **해결책의 품질을 현저히 향상**시키는 것으로, 결과의 정확성과 신뢰성에 극도로 높은 요구를 가진 시나리오에 적합하다.

이 장에서 탐구한 세 가지 패러다임은 에이전트가 문제를 해결하는 세 가지 다른 전략을 나타내며, 표 4.1과 같다. 실제 응용에서 어느 것을 선택하는지는 작업의 핵심 요구에 달려 있다:

<div align="center">
<p>표 4.1 서로 다른 Agent Loop의 선택 전략</p>
<img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/4-figures/4-4.png" alt="" width="70%"/>
</div>

이로써 단일 에이전트를 구축하는 핵심 기술을 습득했다. 지식 과도와 실제 응용에 대한 심층적인 이해를 위해, 다음 절에서는 다양한 로코드 플랫폼의 사용 방법과 경량 코드로 에이전트를 구축하는 방안을 탐구한다.

---

## 습제

> **힌트**: 일부 습제는 표준 답안이 없으며, 중점은 에이전트 패러다임 설계에 대한 학습자의 종합적 이해와 실천 능력을 기르는 데 있다.

1. 이 장에서는 세 가지 고전 에이전트 패러다임인 `ReAct`, `Plan-and-Solve`, `Reflection`을 소개했다. 다음을 분석하라:
   - 세 가지 패러다임이 "사고"와 "행동"의 조직 방식에서 본질적으로 어떤 차이가 있는가?
   - "스마트 홈 제어 어시스턴트"(조명, 에어컨, 커튼 등 여러 기기를 제어하고 사용자 습관에 따라 자동 조절해야 한다)를 설계해야 한다면, 어떤 패러다임을 기본 아키텍처로 선택하겠는가? 이유는 무엇인가?
   - 세 가지 패러다임을 결합하여 사용할 수 있는가? 가능하다면 혼합 패러다임 에이전트 아키텍처를 설계하고 그 적용 시나리오를 설명하라.

2. 4.2절의 `ReAct` 구현에서 정규 표현식을 사용하여 대형 언어 모델의 출력(`Thought`와 `Action`)을 파싱했다. 다음을 생각하라:
   - 현재 파싱 방법에는 어떤 잠재적 취약성이 있는가? 어떤 상황에서 실패할 수 있는가?
   - 정규 표현식 외에 더 견고한 출력 파싱 방안에는 어떤 것이 있는가?
   - 이 장의 코드를 수정하여 더 신뢰할 수 있는 출력 형식을 사용하고, 두 방안의 장단점을 비교하라.

3. 도구 호출은 현대 에이전트의 핵심 능력 중 하나다. 4.2.2절의 `ToolExecutor` 설계를 기반으로 다음 확장 실습을 완성하라:
   > **힌트**: 이것은 실습 문제로, 실제 코드 작성을 권장한다.
   - `ReAct` 에이전트에 "계산기" 도구를 추가하여 복잡한 수학 계산 문제를 처리할 수 있게 하라(예: "`(123 + 456) × 789 / 12 = ?`의 결과 계산").
   - "도구 선택 실패" 처리 메커니즘을 설계하고 구현하라: 에이전트가 잘못된 도구를 여러 번 호출하거나 잘못된 매개변수를 제공할 때, 시스템이 어떻게 수정을 안내해야 하는가?
   - 생각해보라: 호출 가능한 도구의 수가 50개나 100개로 늘어나면, 현재의 도구 설명 방식이 여전히 효과적으로 작동할 수 있는가? 비즈니스 요구에 따라 호출 가능한 도구 수가 크게 늘어날 때, 엔지니어링 관점에서 도구의 조직과 검색 메커니즘을 어떻게 최적화해야 하는가?

4. `Plan-and-Solve` 패러다임은 작업을 "계획"과 "실행" 두 단계로 분해한다. 다음을 깊이 분석하라:
   - 4.3절 구현에서 계획 단계에서 생성된 계획은 "정적"이다(한 번 생성, 수정 불가). 실행 과정에서 어떤 단계를 완료할 수 없거나 결과가 기대에 맞지 않는다면, "동적 재계획" 메커니즘을 어떻게 설계해야 하는가?
   - `Plan-and-Solve`와 `ReAct`를 비교하라: "베이징에서 상하이로의 비즈니스 여행 예약(항공권, 호텔, 렌트카 포함)"과 같은 작업을 처리할 때, 어떤 패러다임이 더 적합한가? 이유는 무엇인가?
   - "계층적 계획" 시스템을 설계해 보라: 먼저 고수준의 추상 계획을 생성한 뒤, 각 고수준 단계에 대해 다시 세부 하위 계획을 생성한다. 이런 설계에는 어떤 장점이 있는가?

5. `Reflection` 메커니즘은 "실행-반성-최적화" 루프를 통해 출력 품질을 향상시킨다. 다음을 생각하라:
   - 4.4절의 코드 생성 케이스에서 다른 단계에 같은 모델을 사용했다. 두 가지 다른 모델을 사용한다면(예: 더 강력한 모델로 반성하고, 더 빠른 모델로 실행한다), 어떤 영향이 있을까?
   - `Reflection` 메커니즘의 종료 조건은 "피드백에 **개선 불필요** 포함"이거나 "최대 반복 횟수 도달"이다. 이 설계가 합리적인가? 더 지능적인 종료 조건을 설계할 수 있는가?
   - "학술 논문 작성 어시스턴트"를 구축한다고 가정하자. 초안을 생성하고 논문 내용을 지속적으로 최적화할 수 있어야 한다. 단락 논리성, 방법 혁신성, 언어 표현, 인용 규범 등 여러 차원에서 반성과 개선을 수행하는 다차원 Reflection 메커니즘을 설계하라.

6. 프롬프트 엔지니어링은 에이전트의 최종 효과에 영향을 미치는 핵심 기술이다. 이 장에서는 여러 정교하게 설계된 프롬프트 템플릿을 보여줬다. 다음을 분석하라:
   - 4.2.3절의 `ReAct` 프롬프트와 4.3.2절의 `Plan-and-Solve` 프롬프트를 비교하면, 명백히 구조 설계상 차이가 있다. 이 차이들이 각자 패러다임의 핵심 논리에 어떻게 기여하는가?
   - 4.4.3절의 `Reflection` 프롬프트에서 "당신은 매우 엄격한 코드 심사 전문가다"와 같은 역할 설정을 사용했다. 이 역할 설정을 수정(예: "당신은 코드 가독성을 중시하는 오픈소스 프로젝트 유지관리자"로 변경)하고 출력 결과의 변화를 관찰하여, 역할 설정이 에이전트 행동에 미치는 영향을 정리하라.
   - 프롬프트에 `few-shot` 예시를 추가하면 모델의 특정 형식 준수 능력이 크게 향상되는 경우가 많다. 이 장의 어떤 에이전트에 `few-shot` 예시를 추가하고 그 효과를 비교해 보라.

7. 어떤 전자상거래 스타트업이 지금 "고객 서비스 에이전트"를 사용하여 실제 고객 서비스를 대체함으로써 비용 절감 효율 향상을 원한다. 에이전트는 다음 기능을 갖춰야 한다:
   a. 사용자의 환불 신청 이유 이해
   b. 사용자의 주문 정보와 물류 상태 조회
   c. 회사 정책에 따라 환불 승인 여부를 지능적으로 판단
   d. 적절한 답장 이메일을 작성하여 사용자 이메일로 발송
   e. 판단 결정에 어느 정도 논란이 있을 때(자기 신뢰도가 임계값 미만), 자기 반성을 수행하고 더 신중한 제안을 제시

   이때 해당 제품의 책임자로서:
   - 이 장의 어떤 패러다임(또는 어떤 패러다임의 조합)을 시스템의 핵심 아키텍처로 선택하겠는가?
   - 이 시스템에는 어떤 도구가 필요한가? 최소 3가지 도구와 그 기능 설명을 나열하라.
   - 에이전트의 결정이 회사 이익에 부합하면서도 사용자에 대한 친근한 태도를 유지하도록 프롬프트를 어떻게 설계하겠는가?
   - 이 제품이 출시된 후 어떤 위험과 도전에 직면할 수 있는가? 기술적 수단으로 이러한 위험을 어떻게 줄일 수 있는가?

---

## 참고 문헌

[1] Yao S, Zhao J, Yu D, et al. React: Synergizing reasoning and acting in language models[C]//International Conference on Learning Representations (ICLR). 2023.

[2] Wang L, Xu W, Lan Y, et al. Plan-and-solve prompting: Improving zero-shot chain-of-thought reasoning by large language models[J]. arXiv preprint arXiv:2305.04091, 2023.

[3] Shinn N, Cassano F, Gopinath A, et al. Reflexion: Language agents with verbal reinforcement learning[J]. Advances in Neural Information Processing Systems, 2023, 36: 8634-8652.