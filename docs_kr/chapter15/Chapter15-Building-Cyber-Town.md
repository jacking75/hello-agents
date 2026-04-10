# 제15장 사이버 마을 구축

이 장에서는 완전히 새로운 방향을 탐구한다: **에이전트 기술과 게임 엔진을 결합하여 생명력 넘치는 AI 마을을 구축하는 것**이다.

《심즈》나 《모여봐요 동물의 숲》에서 살아 숨쉬는 NPC들을 기억하는가? 그들은 저마다의 성격, 기억, 사회적 관계를 가지고 있다. 이 장의 사이버 마을은 그와 유사한 프로젝트이지만, 전통적인 게임과 달리 우리의 NPC들은 진정한 "지능"을 갖는다 — 플레이어의 대화를 이해하고, 과거의 상호작용을 기억하며, 호감도에 따라 다르게 반응할 수 있다. 이 장의 사이버 마을은 다음과 같은 핵심 기능을 포함한다:

**（1）지능형 NPC 대화 시스템**: 플레이어는 NPC와 자연어로 대화할 수 있으며, NPC는 자신의 역할 설정과 기억에 따라 응답한다.

**（2）기억 시스템**: NPC는 단기 기억과 장기 기억을 가지며, 플레이어와의 상호작용 이력을 기억할 수 있다.

**（3）호감도 시스템**: NPC의 플레이어에 대한 태도는 상호작용에 따라 변화하며, 낯선 사이에서 친숙한 사이로, 친근함에서 친밀함으로 발전한다.

**（4）게임화된 상호작용**: 플레이어는 2D 픽셀 스타일의 사무실 장면에서 자유롭게 이동하며 다양한 NPC와 상호작용할 수 있다.

**（5）실시간 로그 시스템**: 모든 대화와 상호작용이 기록되어 디버깅 및 분석에 활용된다.

---

## 15.1 프로젝트 개요와 아키텍처 설계

### 15.1.1 AI 마을을 구축하는 이유

전통적인 게임의 NPC는 보통 고정된 대사만 말하거나, 미리 설정된 대화 트리를 통해 제한적인 상호작용만 할 수 있다. 아무리 복잡한 RPG 게임이라도 NPC의 대화는 시나리오 작가가 미리 작성해 놓은 것이다. 이 방식은 통제 가능하지만, 진정한 "지능"과 "생명력"이 결여되어 있다.

만약 게임 속 NPC가 당신이 하는 말을 무엇이든 이해할 수 있다면 어떨까 — 더 이상 미리 설정된 선택지에 국한되지 않고, 자연어로 NPC와 소통할 수 있다. NPC는 당신이 지난번에 무슨 말을 했는지, 당신들의 관계가 어떠한지, 심지어 당신의 취향까지 기억한다. 각각의 NPC는 자신만의 직업, 성격, 말투를 가진다. NPC의 당신에 대한 태도는 상호작용에 따라 변화하며, 낯선 사람에서 친구로, 심지어 절친한 벗으로 발전한다.

이것이 바로 AI 기술이 게임에 가져오는 새로운 가능성이다. 대형 언어 모델과 게임 엔진을 결합함으로써, 진정으로 "살아있는" NPC를 만들 수 있다. 이는 단순한 기술 시연에 그치지 않고, 미래 게임의 형태에 대한 탐구이기도 하다. 교육 게임에서 NPC는 역사적 인물이나 과학자를 연기하며 학생들과 인터랙티브 교육을 진행할 수 있다. 가상 사무실에서 NPC는 동료나 멘토 역할을 하며 도움과 조언을 제공할 수 있다. NPC는 동반자로서 사용자와 감정적 교류를 나누어 정신 건강 분야에 활용될 수도 있다. 물론, 가장 직접적인 응용은 전통 게임에 AI NPC를 추가하여 플레이어 경험을 향상시키는 것이다.

### 15.1.2 기술 아키텍처 개요

사이버 마을은 **게임 엔진 + 백엔드 서비스**의 분리 아키텍처를 채택하며, 그림 15.1과 같이 네 개의 계층으로 구성된다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-1.png" alt="" width="85%"/>
  <p>그림 15.1 사이버 마을 기술 아키텍처</p>
</div>

프론트엔드 계층은 Godot 4.5 게임 엔진을 사용하며, 게임 렌더링, 플레이어 제어, NPC 표시 및 대화 UI를 담당한다. Godot는 오픈소스 2D/3D 게임 엔진으로, 픽셀 스타일 게임의 빠른 개발에 매우 적합하다. 백엔드 계층은 FastAPI 프레임워크를 사용하며, API 라우팅, NPC 상태 관리, 대화 처리 및 로그 기록을 담당한다. FastAPI는 현대적인 Python 웹 프레임워크로, 성능이 우수하고 개발하기 쉽다. 에이전트 계층은 자체 구축한 HelloAgents 프레임워크를 사용하며, NPC 지능, 기억 관리 및 호감도 계산을 담당한다. 각 NPC는 하나의 SimpleAgent 인스턴스로, 독립적인 기억과 상태를 가진다. 외부 서비스 계층은 LLM 능력, 벡터 스토리지 및 데이터 영속성을 제공하며, LLM API, Qdrant 벡터 데이터베이스 및 SQLite 관계형 데이터베이스를 포함한다.

데이터 흐름 과정은 그림 15.2와 같다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-2.png" alt="" width="85%"/>
  <p>그림 15.2 데이터 흐름 과정</p>
</div>

플레이어가 Godot에서 E키를 눌러 NPC와 상호작용하면, Godot는 HTTP API를 통해 대화 요청을 FastAPI 백엔드로 전송한다. 백엔드는 HelloAgents의 SimpleAgent를 호출하여 대화를 처리하고, Agent는 기억 시스템에서 관련 이력을 검색한 뒤 LLM을 호출하여 응답을 생성한다. 백엔드는 NPC 상태와 호감도를 업데이트하고, 콘솔과 파일에 로그를 기록한 후, 응답을 Godot 프론트엔드로 반환한다. Godot는 NPC의 응답을 표시하고 UI를 업데이트하며 하나의 완전한 상호작용 루프를 완성한다.

프로젝트의 구조는 다음과 같으며, 소스 코드를 찾는 데 도움이 된다:

```
Helloagents-AI-Town/
├── helloagents-ai-town/           # Godot 게임 프로젝트
│   ├── project.godot              # Godot 프로젝트 설정
│   ├── scenes/                    # 게임 씬
│   │   ├── main.tscn              # 메인 씬(사무실)
│   │   ├── player.tscn            # 플레이어 캐릭터
│   │   ├── npc.tscn               # NPC 캐릭터
│   │   └── dialogue_ui.tscn       # 대화 UI
│   ├── scripts/                   # GDScript 스크립트
│   │   ├── main.gd                # 메인 씬 로직
│   │   ├── player.gd              # 플레이어 제어
│   │   ├── npc.gd                 # NPC 행동
│   │   ├── dialogue_ui.gd         # 대화 UI 로직
│   │   ├── api_client.gd          # API 클라이언트
│   │   └── config.gd              # 설정 관리
│   └── assets/                    # 게임 리소스
│       ├── characters/            # 캐릭터 스프라이트
│       ├── interiors/             # 실내 장면
│       ├── ui/                    # UI 소재
│       └── audio/                 # 음향 음악
│
└── backend/                       # Python 백엔드
    ├── main.py                    # FastAPI 메인 프로그램
    ├── agents.py                  # NPC Agent 시스템
    ├── relationship_manager.py    # 호감도 관리
    ├── state_manager.py           # 상태 관리
    ├── logger.py                  # 로그 시스템
    ├── config.py                  # 설정 관리
    ├── models.py                  # 데이터 모델
    ├── requirements.txt           # Python 의존성
    └── .env.example               # 환경 변수 예시
```

상세한 아키텍처 설계와 데이터 흐름은 이후 장에서 소개한다.

### 15.1.3 빠른 체험: 5분 만에 프로젝트 실행하기

구현 세부 사항을 깊이 배우기 전에, 먼저 프로젝트를 실행해서 최종 효과를 확인해 보자. 이렇게 하면 전체 시스템에 대한 직관적인 인식을 얻을 수 있다.

**환경 요구사항:**

- Godot 4.2 이상
- Python 3.10 이상
- LLM API 키 (OpenAI, DeepSeek, 智谱 등)

**프로젝트 가져오기:**

`code/chapter15/Helloagents-AI-Town`에서 확인하거나, GitHub에서 완전한 hello-agents 저장소를 클론할 수 있다.

**백엔드 시작:**

```bash
# 1. backend 디렉토리로 이동
cd Helloagents-AI-Town/backend

# 2. 의존성 설치
pip install -r requirements.txt

# 3. 환경 변수 설정
cp .env.example .env
# .env 파일을 편집하여 API 키를 입력

# 4. 백엔드 서비스 시작
python main.py
```

성공적으로 시작되면 다음과 같은 출력을 볼 수 있다:

```
============================================================
🎮 사이버 마을 백엔드 서비스 시작 중...
============================================================
✅ 모든 서비스가 시작되었습니다!
📡 API 주소: http://0.0.0.0:8000
📚 API 문서: http://0.0.0.0:8000/docs
============================================================
```

**Godot 시작:**

Godot 설치는 매우 간단하다. Windows는 바로 실행 가능한 `.exe` 파일을, Mac은 `.dmg` 파일을 공식 사이트에서 다운로드할 수 있다([Windows](https://godotengine.org/download/windows/) / [Mac](https://godotengine.org/download/macos/)).

Godot 엔진을 열고 "가져오기" 버튼을 클릭한 뒤, `Helloagents-AI-Town/helloagents-ai-town/scenes/main.tscn`으로 이동하여 "가져오기 및 편집"을 클릭한다. Godot가 리소스를 가져온 후, `F5`를 누르거나 "실행" 버튼을 클릭하여 게임을 시작한다.

**핵심 기능 체험:**

게임이 시작되면 그림 15.3과 같이 픽셀 스타일의 Datawhale 사무실 장면을 볼 수 있다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-3.png" alt="" width="85%"/>
  <p>그림 15.3 사이버 마을 게임 장면</p>
</div>

WASD 키로 플레이어 캐릭터를 이동시키고, NPC 근처에 가면 화면에 "E키를 눌러 상호작용"이라는 메시지가 표시된다. E키를 누르면 대화창이 나타나며, 원하는 말을 입력할 수 있다. 그림 15.4와 같다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-4.png" alt="" width="85%"/>
  <p>그림 15.4 NPC 대화 인터페이스</p>
</div>

NPC는 자신의 역할 설정(Python 엔지니어, 프로덕트 매니저, UI 디자이너)과 상호작용 이력에 따라 응답한다. 대화가 진행됨에 따라 플레이어에 대한 NPC의 호감도가 점차 높아지며, "낯선 사이"에서 "친숙한 사이"로, 다시 "친근함", "친밀함", 심지어 "절친한 벗"으로 발전한다.

**호감도 시스템은 백엔드에서 구현**되며, 매 대화마다 플레이어의 메시지 내용과 감정 분석을 기반으로 호감도 값이 조정된다. 프론트엔드 게임 인터페이스에는 호감도 수치가 직접 표시되지 않지만, 모든 호감도 변화는 백엔드 로그에 상세히 기록된다. `backend/logs/dialogue_YYYY-MM-DD.log` 파일에서 각 대화의 호감도 변화를 확인할 수 있다. 로그 파일에는 현재 호감도 값, 검색된 관련 기억, NPC의 응답, 호감도 변화량(+2.0, +3.0 등), 변화 이유(친근한 인사, 일반적인 교류 등), 감정 분석 결과(positive, neutral 등)가 포함된다. 이 설계를 통해 개발자는 NPC와 플레이어의 관계 발전을 명확하게 추적할 수 있으며, 이후 프론트엔드에 호감도 UI를 추가하기 위한 데이터 기반도 마련된다.

모든 대화는 백엔드의 로그 파일에 기록되며, 다음 명령으로 실시간으로 확인할 수 있다:

```bash
# backend 디렉토리에서
python view_logs.py
```

이 간단한 체험을 통해 AI 마을의 핵심 기능을 확인할 수 있다. 이제 이러한 기능들을 어떻게 구현하는지 깊이 학습해 보자.

---

## 15.2 NPC 지능 에이전트 시스템

### 15.2.1 HelloAgents 기반의 SimpleAgent

사이버 마을에서 각 NPC는 독립적인 에이전트다. HelloAgents 프레임워크의 SimpleAgent를 사용하여 NPC의 지능을 구현한다. SimpleAgent는 경량화된 에이전트 구현체로, LLM 호출, 메시지 관리 및 도구 호출 등 핵심 기능을 캡슐화하고 있다.

제7장에서 배운 SimpleAgent를 떠올려보자. 핵심은 간단한 대화 루프다: 사용자 메시지를 수신하고, LLM을 호출하여 응답을 생성하고, 결과를 반환한다. 사이버 마을에서는 각 NPC마다 SimpleAgent 인스턴스를 생성하고, 고유한 시스템 프롬프트를 구성하여 각 NPC가 서로 다른 성격과 역할 설정을 갖도록 한다.

NPC Agent를 어떻게 생성하는지 살펴보자. 먼저 NPC의 기본 정보(ID, 이름, 직업, 성격)를 정의한다. 그런 다음 이 정보를 바탕으로 시스템 프롬프트를 구성하여 LLM이 이 NPC의 역할을 수행하도록 한다. 마지막으로 SimpleAgent 인스턴스를 생성하고 기억 시스템을 구성한다.

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.memory import MemoryManager, WorkingMemory, EpisodicMemory

def create_npc_agent(npc_id: str, name: str, role: str, personality: str):
    """NPC Agent 생성"""
    # 시스템 프롬프트 구성
    system_prompt = f"""당신은 {name}이며, {role}입니다.
성격 특징: {personality}

당신은 Datawhale 사무실에서 근무하며, 동료들과 함께 오픈소스 커뮤니티 발전을 이끌고 있습니다.
자신의 역할과 성격에 따라 플레이어와 자연스럽게 대화하세요.
이전 대화 내용을 기억하고 대화의 연속성을 유지하세요.
"""

    # LLM 인스턴스 생성
    llm = HelloAgentsLLM()

    # 기억 관리자 생성
    memory_manager = MemoryManager(
        working_memory=WorkingMemory(capacity=10, ttl_minutes=120),
        episodic_memory=EpisodicMemory(
            db_path=f"memory_data/{npc_id}_episodic.db",
            collection_name=f"{npc_id}_memories"
        )
    )

    # Agent 생성
    agent = SimpleAgent(
        name=name,
        llm=llm,
        system_prompt=system_prompt,
        memory_manager=memory_manager
    )

    return agent
```

이 코드는 NPC Agent를 어떻게 생성하는지 보여준다. 시스템 프롬프트는 NPC의 신원과 성격을 정의하며, 기억 관리자는 NPC가 플레이어와의 대화 이력을 기억할 수 있게 한다. WorkingMemory는 단기 기억으로, 용량이 10개 메시지이고 보존 시간은 120분이다. EpisodicMemory는 장기 기억으로, SQLite 데이터베이스와 Qdrant 벡터 데이터베이스를 사용하여 저장하며, 관련 과거 대화를 검색할 수 있다.

NPC Agent의 작업 흐름은 그림 15.5와 같다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-5.png" alt="" width="85%"/>
  <p>그림 15.5 NPC Agent 작업 흐름</p>
</div>

### 15.2.2 NPC 역할 설정과 프롬프트 설계

좋은 NPC는 뚜렷한 성격과 역할 설정이 있어야 한다. 사이버 마을에서는 서로 다른 직업과 성격을 가진 세 명의 NPC를 설계했다.

**장삼(张三) - Python 엔지니어**

장삼은 HelloAgents 프레임워크의 핵심 개발을 담당하는 베테랑 Python 엔지니어다. 성격이 엄격하고 말이 직접적이며, 기술 용어를 즐겨 사용한다. 코드 품질에 높은 기준을 세우며, 프로그래밍 팁과 모범 사례를 자주 공유한다.

```python
npc_zhang = {
    "npc_id": "zhang_san",
    "name": "张三",
    "role": "Python 엔지니어",
    "personality": "엄격하고, 전문적이며, 기술 지식 공유를 즐긴다. 말이 직접적이고 코드 품질을 중시한다."
}
```

**이사(李四) - 프로덕트 매니저**

이사는 HelloAgents 프레임워크의 제품 계획과 사용자 경험 설계를 담당하는 경험 풍부한 프로덕트 매니저다. 성격이 외향적이고 소통을 잘하며, 항상 사용자의 관점에서 문제를 생각한다. 제품 설계와 사용자 요구를 논의하기 좋아하며, 자주 "왜?"라고 묻는다.

```python
npc_li = {
    "npc_id": "li_si",
    "name": "李四",
    "role": "프로덕트 매니저",
    "personality": "외향적이고, 소통을 잘하며, 사용자 경험을 중시한다. 사용자 관점에서 문제를 생각하기 좋아한다."
}
```

**왕오(王五) - UI 디자이너**

왕오는 HelloAgents 프레임워크의 인터페이스 디자인과 시각적 표현을 담당하는 창의적인 UI 디자이너다. 성격이 온화하고 독특한 미적 감각을 가지며, 색상과 레이아웃에 예민한 감각이 있다. 디자인 이념과 미학에 대해 이야기하기 좋아하며, 디자인 영감을 자주 공유한다.

```python
npc_wang = {
    "npc_id": "wang_wu",
    "name": "王五",
    "role": "UI 디자이너",
    "personality": "온화하고, 창의적이며, 독특한 미적 감각을 가진다. 시각적 표현과 사용자 경험을 중시한다."
}
```

이 세 NPC의 설정은 각각 특색이 있어서, 플레이어는 자신의 관심사에 따라 다른 NPC와 상호작용할 수 있다. 장삼은 프로그래밍 팁을 가르쳐 줄 수 있고, 이사는 제품 설계를 함께 논의할 수 있으며, 왕오는 디자인 영감을 공유할 수 있다.

### 15.2.3 기억 시스템 통합

기억 시스템은 NPC 지능의 핵심이다. 과거 대화를 기억하는 NPC는 플레이어에게 더욱 현실적이고 흥미롭게 느껴진다. helloagents의 `WorkingMemory`와 `EpisodicMemory`를 사용하여 단기 기억과 장기 기억을 구성한다.

단기 기억은 최근 대화 내용을 저장하며, 용량이 제한되어 있고 시간이 지남에 따라 자동으로 정리된다. 대화의 연속성을 유지하여 NPC가 맥락을 이해할 수 있게 하는 역할을 한다. 예를 들어, 플레이어가 "그것의 색깔은 뭐야?"라고 말할 때, NPC는 단기 기억에서 "그것"이 무엇을 가리키는지 찾을 수 있다.

장기 기억은 모든 대화 이력을 저장하며, 벡터 데이터베이스를 사용한 의미론적 검색을 지원한다. 플레이어가 특정 주제를 언급할 때, NPC는 장기 기억에서 관련 과거 대화를 검색하여 이전에 논의했던 내용을 회상할 수 있다. 예를 들어, 플레이어가 "지난번에 우리가 논의했던 그 프로젝트 기억나요?"라고 하면, NPC는 장기 기억에서 관련 대화 기록을 찾을 수 있다.

기억 시스템의 아키텍처는 그림 15.6과 같다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-6.png" alt="" width="85%"/>
  <p>그림 15.6 기억 시스템 아키텍처</p>
</div>

실제 사용에서 Agent는 먼저 단기 기억에서 최근 대화를 가져오고, 그런 다음 장기 기억에서 관련 이력 대화를 검색하여, 이 정보들을 함께 LLM에 전달해 더욱 정확하고 개인화된 응답을 생성한다.

```python
# Agent의 대화 처리 흐름
def process_dialogue(agent, player_message):
    # 1. 단기 기억에서 최근 대화 가져오기
    recent_messages = agent.memory_manager.working_memory.get_recent_messages(5)

    # 2. 장기 기억에서 관련 이력 검색
    relevant_memories = agent.memory_manager.episodic_memory.search(
        query=player_message,
        top_k=3
    )

    # 3. 컨텍스트 구성
    context = {
        "recent": recent_messages,
        "relevant": relevant_memories
    }

    # 4. Agent를 호출하여 응답 생성
    reply = agent.run(player_message, context=context)

    # 5. 기억 시스템에 저장
    agent.memory_manager.add_interaction(player_message, reply)

    return reply
```

이 흐름을 통해 NPC가 플레이어와의 상호작용 이력을 기억하고 대화에 반영할 수 있다.

### 15.2.4 배치 대화 생성: 경량 부하 모드

실제 운영에서 곧 발견하게 되는 문제가 있다: 여러 플레이어가 동시에 서로 다른 NPC와 대화할 때, 백엔드는 여러 LLM 요청을 동시에 처리해야 한다. 각 요청마다 API를 호출해야 하므로, 비용이 증가할 뿐만 아니라 동시 처리 제한으로 인해 요청 실패나 지연이 발생할 수 있다.

이 문제를 해결하기 위해 **배치 대화 생성 시스템**을 설계했다. 핵심 아이디어는 여러 NPC의 대화 요청을 하나의 LLM 호출로 합쳐서, LLM이 한 번에 모든 NPC의 응답을 생성하도록 하는 것이다. 이는 식당의 "미리 만든 음식"처럼, 미리 대량으로 준비해 두고 필요할 때 바로 사용하여 비용과 지연을 크게 줄인다.

배치 생성의 작업 흐름은 그림 15.7과 같다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-7.png" alt="" width="85%"/>
  <p>그림 15.7 배치 생성 vs 전통 방식</p>
</div>

배치 생성기의 구현은 매우 영리하다. LLM이 한 번에 모든 NPC의 대화를 JSON 형식으로 반환하도록 하는 특별한 프롬프트를 구성한다. 이렇게 하면 하나의 API 호출로 모든 NPC의 응답을 얻을 수 있어 비용이 원래의 1/3 수준으로 줄고 지연도 크게 감소한다.

```python
class NPCBatchGenerator:
    """NPC 대화 배치 생성기"""

    def __init__(self):
        self.llm = HelloAgentsLLM()
        self.npc_configs = NPC_ROLES  # 모든 NPC 설정

    def generate_batch_dialogues(self, context: Optional[str] = None) -> Dict[str, str]:
        """모든 NPC의 대화를 배치로 생성

        Args:
            context: 장면 컨텍스트(예: "오전 업무 시간", "점심 시간" 등)

        Returns:
            Dict[str, str]: NPC 이름과 대화 내용의 매핑
        """
        # 배치 생성 프롬프트 구성
        prompt = self._build_batch_prompt(context)

        # 한 번의 LLM 호출로 모든 대화 생성
        response = self.llm.invoke([
            {"role": "system", "content": "당신은 게임 NPC 대화 생성기로, 자연스럽고 현실적인 사무실 대화 작성에 능숙합니다."},
            {"role": "user", "content": prompt}
        ])

        # JSON 응답 파싱
        dialogues = json.loads(response)
        # 반환 형식: {"张三": "...", "李四": "...", "王五": "..."}

        return dialogues

    def _build_batch_prompt(self, context: Optional[str] = None) -> str:
        """배치 생성 프롬프트 구성"""
        # 시간에 따라 자동으로 장면 추론
        if context is None:
            context = self._get_current_context()

        # NPC 설명 구성
        npc_descriptions = []
        for name, cfg in self.npc_configs.items():
            desc = f"- {name}({cfg['title']}): {cfg['location']}에서 {cfg['activity']} 중, 성격은 {cfg['personality']}"
            npc_descriptions.append(desc)

        npc_desc_text = "\n".join(npc_descriptions)

        prompt = f"""Datawhale 사무실의 3명의 NPC에 대한 현재 대화나 행동 설명을 생성해 주세요.

【장면】{context}

【NPC 정보】
{npc_desc_text}

【생성 요구사항】
1. 각 NPC에 대해 1문장(20-40자) 생성
2. 역할 설정, 현재 활동, 장면 분위기에 맞는 내용
3. 혼잣말, 업무 상태 설명 또는 단순한 생각도 가능
4. 자연스럽고 현실적으로, 실제 사무실 동료처럼
5. **JSON 형식을 엄격히 준수하여 반환**

【출력 형식】(엄격히 준수)
{{"张三": "...", "李四": "...", "王五": "..."}}

【예시 출력】
{{"张三": "이 버그는 정말 이상하네, 벌써 두 시간째 디버깅 중이야...", "李四": "음, 이 기능의 우선순위를 다시 평가해야 할 것 같아.", "王五": "이 커피 라떼 아트 정말 멋지다, 영감이 떠올랐어!"}}

생성해 주세요(JSON만 반환하고 다른 내용은 포함하지 마세요):
"""
        return prompt
```

이 설계의 핵심은 프롬프트 구성에 있다. LLM에게 JSON 형식으로 반환하도록 명확히 요구하고 예시 출력을 제공한다. LLM은 이 형식을 엄격히 따라 응답을 생성하며, JSON만 파싱하면 모든 NPC의 대화를 얻을 수 있다.

배치 생성에는 추가적인 장점도 있다: 모든 NPC의 대화가 동일한 컨텍스트에서 생성되므로 서로 간에 연관성이 생긴다. 예를 들어 장삼이 버그를 디버깅하고 있다면, 이사가 도와주겠다고 할 수도 있고, 왕오가 인터페이스를 디자인하고 있다면 장삼이 나중에 디자인을 보러 가겠다고 할 수도 있다. 이로써 사무실 전체 분위기가 더욱 현실감 있고 일관성 있게 된다.

물론 배치 생성에도 한계가 있다. NPC의 "배경 대화"나 "혼잣말" 생성에 더 적합하며, 플레이어와의 직접적인 상호작용에는 적합하지 않다. 플레이어가 시작한 대화의 경우 개성화되고 정확한 응답을 보장하기 위해 별도의 Agent를 사용한다. 배치 생성은 주로 다음 시나리오에 사용된다:

1. **NPC 배경 대화**: 플레이어가 장면에 들어올 때 NPC가 무엇을 하고 무슨 말을 하는지
2. **정기적 업데이트**: 일정 간격으로 NPC의 상태와 대화를 업데이트
3. **장면 분위기**: 시간대(아침, 점심, 저녁)에 따라 다른 대화 생성
4. **비용 절감**: 고부하 시나리오에서 배치 생성으로 API 호출 횟수 감소

**혼합 모드: 배치 생성 + 즉시 응답**

실제 구현에서는 배치 생성과 즉시 응답을 결합한 혼합 모드를 채택했다. 이 설계는 매우 영리하여 효율성과 상호작용 품질을 모두 보장한다.

구체적으로, 시스템은 백그라운드에서 주기적으로 배치 생성을 실행하여 모든 NPC에 대한 현재 장면의 "배경 대화"를 생성한다. 이 대화는 캐시되며, 플레이어가 NPC 근처에 있지만 아직 상호작용을 시작하지 않은 경우 NPC는 "코드 디버깅 중...", "제품 문서 보는 중..." 등의 배경 대화를 표시한다. 이로써 NPC가 "살아있는" 것처럼 보이며 정적인 모델이 아닌 것처럼 느껴진다.

하지만 플레이어가 E키를 눌러 상호작용을 시작하면, 시스템은 즉시 즉시 응답 모드로 전환한다. 이때 백엔드는 해당 NPC의 전용 Agent를 호출하여 플레이어의 구체적인 메시지, 기억 이력, 호감도를 기반으로 개인화된 응답을 생성한다. 이 과정은 실시간으로 이루어지며, NPC의 응답이 플레이어의 입력과 높은 관련성을 갖도록 보장한다.

```python
# main.py의 혼합 모드 구현
@app.post("/dialogue")
async def dialogue(request: DialogueRequest):
    """플레이어와 NPC의 대화 처리(즉시 응답 모드)"""
    npc_id = request.npc_id
    player_message = request.player_message
    player_name = request.player_name

    # NPC Agent 가져오기(각 NPC는 독립적인 Agent를 가짐)
    agent = npc_agents.get(npc_id)
    if not agent:
        raise HTTPException(status_code=404, detail="NPC not found")

    # 즉시 개인화된 응답 생성
    # 여기서는 배치 생성을 사용하지 않고 Agent의 run 메서드를 호출
    reply = agent.run(player_message)

    # 호감도 업데이트
    affinity_change = relationship_manager.update_affinity(
        npc_id, player_name, player_message, reply
    )

    return {
        "npc_reply": reply,
        "affinity_score": affinity_change["score"],
        "affinity_level": affinity_change["level"]
    }

# 백그라운드 작업: 정기적으로 배치 생성으로 배경 대화 업데이트
async def background_dialogue_update():
    """백그라운드 작업: 5분마다 NPC 배경 대화 업데이트"""
    while True:
        try:
            # 배치 생성기를 사용하여 모든 NPC의 배경 대화 생성
            batch_generator = get_batch_generator()
            dialogues = batch_generator.generate_batch_dialogues()

            # 상태 관리자에 업데이트
            for npc_name, dialogue in dialogues.items():
                state_manager.update_npc_background_dialogue(npc_name, dialogue)

            print(f"✅ 배경 대화 업데이트 완료: {len(dialogues)}명의 NPC")
        except Exception as e:
            print(f"❌ 배경 대화 업데이트 실패: {e}")

        # 5분 대기
        await asyncio.sleep(300)
```

이 혼합 모드의 장점은 매우 명확하다:

1. **비용 절감**: 배경 대화는 배치 생성을 사용하여 한 번의 호출로 모든 NPC의 대화를 생성하므로 비용이 낮다
2. **품질 보장**: 플레이어 상호작용은 즉시 응답을 사용하며, 각 응답이 개인화되어 품질이 높다
3. **경험 향상**: NPC는 항상 "배경 대화"가 있어 생동감 있게 보이며, 플레이어 상호작용 시 응답이 정확하여 경험이 좋다
4. **유연한 조정**: 서버 부하에 따라 배치 생성 빈도를 동적으로 조정할 수 있다

배치 생성과 즉시 응답의 결합을 통해 효율적이고 지능적인 NPC 시스템을 구현했다. 정상적인 상황에서 플레이어는 차이를 느끼지 못하지만, 백엔드의 비용과 성능은 크게 최적화된다. 이 설계 사고방식은 대량의 AI 호출이 필요한 다른 시나리오에도 응용할 수 있다.

---

## 15.3 호감도 시스템 설계

### 15.3.1 호감도 등급 구분

사이버 마을에서 NPC의 플레이어에 대한 태도는 상호작용에 따라 변화한다. 낯선 사이에서 절친한 벗까지, 각 등급마다 서로 다른 점수 범위와 해당 행동 표현을 가진 5단계 호감도 시스템을 설계했다.

호감도 시스템의 핵심 아이디어는 NPC와 플레이어의 관계를 수치화하여 NPC의 응답이 더욱 현실적이고 층위가 있도록 하는 것이다. 플레이어가 게임에 처음 들어오면 모든 NPC는 플레이어에게 낯선 태도를 취하며, 응답이 예의 바르지만 거리감이 있다. 대화가 진행됨에 따라 플레이어가 친근하게 행동하면 NPC의 호감도가 점차 높아지고 응답도 더욱 친근하고 상세해진다.

호감도를 5개 등급으로 구분하며, 각 등급은 점수 범위에 대응한다. 그림 15.8과 같다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-8.png" alt="" width="85%"/>
  <p>그림 15.8 호감도 등급 구분</p>
</div>

- **낯선 사이(0-20점)**: NPC가 플레이어를 막 알게 된 상태로, 태도가 예의 바르지만 거리를 유지한다. 응답이 짧고 개인 정보를 자발적으로 공유하지 않는다.

- **친숙한 사이(21-40점)**: NPC가 플레이어를 기억하기 시작하며, 간단한 교류를 기꺼이 한다. 응답이 더욱 자연스러워지고 가끔 업무 관련 정보를 공유한다.

- **친근한 사이(41-60점)**: NPC가 플레이어를 친구로 여기며 더 많은 정보를 기꺼이 공유한다. 응답이 더욱 상세해지고 플레이어의 상황을 자발적으로 물어본다.

- **친밀한 사이(61-80점)**: NPC가 플레이어를 매우 신뢰하며 개인적인 주제도 기꺼이 공유한다. 응답이 열정적이며 플레이어에게 도움과 조언을 제공한다.

- **절친한 벗(81-100점)**: NPC가 플레이어를 가장 좋은 친구로 여기며 무슨 말이든 나눈다. 응답이 매우 친근하고 내면의 생각과 감정을 공유한다.

이 설계를 통해 플레이어는 NPC와의 관계 변화를 명확하게 느낄 수 있으며, 이후의 게임 플레이를 위한 기반도 마련된다. 예를 들어, 일정 호감도에 도달해야만 NPC가 특정 특별 정보를 공유하거나 특별 임무를 제공한다.

### 15.3.2 호감도 계산 로직

호감도 계산은 여러 요소를 고려해야 한다. 단순히 매 대화마다 고정된 점수를 더하는 방식으로는 시스템이 기계적이고 비현실적으로 느껴진다. 좋은 호감도 시스템은 플레이어의 태도를 인식하고 대화 내용을 기반으로 점수를 동적으로 조정할 수 있어야 한다.

사이버 마을에서는 LLM을 사용하여 대화 내용을 분석하고 플레이어의 태도가 친근한지, 중립적인지, 비우호적인지 판단한다. 그런 다음 판단 결과에 따라 호감도 점수를 조정한다. 이 과정은 자동으로 이루어지며, 플레이어가 의도적으로 선택지를 고르지 않아도 되므로 상호작용이 더욱 자연스럽다.

호감도 계산 흐름은 그림 15.9와 같다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-9.png" alt="" width="85%"/>
  <p>그림 15.9 호감도 계산 흐름</p>
</div>

```python
class RelationshipManager:
    """호감도 관리자"""

    def __init__(self):
        self.affinity_data = {}  # 호감도 데이터 저장
        self.llm = HelloAgentsLLM()  # 대화 분석용

    def analyze_sentiment(self, player_message: str, npc_reply: str) -> int:
        """대화 감정을 분석하여 호감도 변화값 반환"""
        prompt = f"""다음 대화에서 플레이어의 태도를 분석하세요:
플레이어: {player_message}
NPC: {npc_reply}

플레이어의 태도를 판단하세요:
1. 친근함(+5점): 예의 바르고, 열정적이며, 감사나 동의를 표현
2. 중립(+2점): 일반적인 질문이나 진술
3. 비우호적(-3점): 무례하고, 냉담하며, 비판적이거나 부정적

숫자만 반환하고 다른 내용은 포함하지 마세요."""

        response = self.llm.think([{"role": "user", "content": prompt}])
        try:
            score_change = int(response.strip())
            return max(-3, min(5, score_change))  # -3에서 5 사이로 제한
        except:
            return 2  # 기본값: 중립

    def update_affinity(self, npc_id: str, player_name: str,
                       player_message: str, npc_reply: str) -> dict:
        """호감도 업데이트"""
        key = f"{npc_id}_{player_name}"

        # 현재 호감도 가져오기
        if key not in self.affinity_data:
            self.affinity_data[key] = {
                "score": 0,
                "level": "낯선 사이",
                "interaction_count": 0
            }

        # 대화 감정 분석
        score_change = self.analyze_sentiment(player_message, npc_reply)

        # 점수 업데이트
        current_score = self.affinity_data[key]["score"]
        new_score = max(0, min(100, current_score + score_change))

        # 등급 업데이트
        level = self.get_affinity_level(new_score)

        # 데이터 업데이트
        self.affinity_data[key].update({
            "score": new_score,
            "level": level,
            "interaction_count": self.affinity_data[key]["interaction_count"] + 1
        })

        return self.affinity_data[key]

    def get_affinity_level(self, score: int) -> str:
        """점수에 따른 호감도 등급 반환"""
        if score <= 20:
            return "낯선 사이"
        elif score <= 40:
            return "친숙한 사이"
        elif score <= 60:
            return "친근한 사이"
        elif score <= 80:
            return "친밀한 사이"
        else:
            return "절친한 벗"
```

이 구현은 LLM을 사용하여 대화 내용을 분석하고 자동으로 플레이어의 태도를 판단하여 호감도를 조정한다. 이런 설계 덕분에 호감도 시스템이 더욱 지능적이고 자연스러워지며, 플레이어는 의도적으로 NPC의 비위를 맞추지 않아도 자연스럽게 교류하면 된다.

### 15.3.3 호감도가 대화에 미치는 영향

호감도는 단순한 숫자가 아니며, 실제로 NPC의 행동에 영향을 미쳐야 한다. 사이버 마을에서는 NPC의 시스템 프롬프트를 수정함으로써 NPC가 현재 호감도 등급에 따라 응답 스타일을 조정하도록 한다.

호감도가 낮을 때 NPC는 예의 바르지만 거리감 있는 태도를 유지한다. 호감도가 높아지면 NPC는 더욱 열정적이고 말이 많아진다. 이 변화는 시스템 프롬프트를 동적으로 조정함으로써 구현된다.

```python
def create_npc_agent_with_affinity(npc_id: str, name: str, role: str,
                                   personality: str, affinity_level: str):
    """호감도가 적용된 NPC Agent 생성"""

    # 호감도 등급에 따라 프롬프트 조정
    affinity_prompts = {
        "낯선 사이": "이 플레이어를 막 알게 된 상태로, 예의 바르되 너무 열정적으로 행동하지 마세요. 응답은 짧고 전문적으로.",
        "친숙한 사이": "이 플레이어를 이미 알고 있으므로, 일반적인 교류를 할 수 있습니다. 자연스럽고 친근하게 응답하세요.",
        "친근한 사이": "이 플레이어를 친구로 여기며 더 많은 정보를 기꺼이 공유합니다. 상세하고 열정적으로 응답하세요.",
        "친밀한 사이": "이 플레이어를 매우 신뢰하며 개인적인 주제도 공유할 수 있습니다. 관심을 담아 응답하세요.",
        "절친한 벗": "이 플레이어를 가장 좋은 친구로 여기며 무슨 말이든 나눌 수 있습니다. 친근하고 진심 어린 응답을 하세요."
    }

    system_prompt = f"""당신은 {name}이며, {role}입니다.
성격 특징: {personality}

플레이어와의 현재 관계: {affinity_level}
{affinity_prompts.get(affinity_level, affinity_prompts["낯선 사이"])}

당신은 Datawhale 사무실에서 근무하며, 동료들과 함께 오픈소스 커뮤니티 발전을 이끌고 있습니다.
자신의 역할, 성격 및 플레이어와의 관계에 따라 자연스럽게 응답하세요.
"""

    # Agent 생성
    llm = HelloAgentsLLM()
    agent = SimpleAgent(
        name=name,
        llm=llm,
        system_prompt=system_prompt
    )

    return agent
```

이 설계를 통해 NPC의 행동이 호감도에 따라 동적으로 변화한다. 플레이어는 상호작용이 쌓일수록 NPC의 자신에 대한 태도가 점차 변해가는 것을 명확하게 느낄 수 있으며, 이로써 게임의 몰입감과 재미가 크게 향상된다.

---

## 15.4 백엔드 서비스 구현

### 15.4.1 FastAPI 애플리케이션 구조

사이버 마을의 백엔드는 FastAPI 프레임워크로 구축되며, Godot 프론트엔드의 요청 처리, HelloAgents의 NPC Agent 호출, NPC 상태 및 호감도 관리, 로그 기록 등을 담당한다. 명확한 애플리케이션 구조는 코드를 더 유지보수하고 확장하기 쉽게 만든다.

FastAPI 애플리케이션은 모듈화 설계를 채택하여 서로 다른 기능을 서로 다른 파일에 분리한다. 그림 15.10과 같다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-10.png" alt="" width="85%"/>
  <p>그림 15.10 백엔드 애플리케이션 구조</p>
</div>

`main.py`부터 시작해 살펴보자. 이 파일은 FastAPI 애플리케이션의 진입점이다:

```python
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field
from typing import Optional
import uvicorn

from agents import NPCAgentManager
from relationship_manager import RelationshipManager
from state_manager import StateManager
from logger import DialogueLogger
from config import settings

# FastAPI 애플리케이션 생성
app = FastAPI(
    title="사이버 마을 백엔드 서비스",
    description="HelloAgents 기반의 AI NPC 대화 시스템",
    version="1.0.0"
)

# CORS 설정: Godot 프론트엔드 접근 허용
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # 프로덕션 환경에서는 특정 도메인으로 제한해야 함
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 각 관리자 초기화
agent_manager = NPCAgentManager()
relationship_manager = RelationshipManager()
state_manager = StateManager()
dialogue_logger = DialogueLogger()

@app.on_event("startup")
async def startup_event():
    """애플리케이션 시작 시 초기화"""
    print("=" * 60)
    print("🎮 사이버 마을 백엔드 서비스 시작 중...")
    print("=" * 60)

    # NPC Agent 초기화
    agent_manager.initialize_npcs()
    print("✅ NPC Agent 초기화 완료")

    # 상태 관리자 초기화
    state_manager.initialize_npcs()
    print("✅ 상태 관리자 초기화 완료")

@app.get("/")
async def root():
    """헬스 체크"""
    return {
        "status": "running",
        "message": "사이버 마을 백엔드 서비스 실행 중",
        "version": "1.0.0",
        "npcs": state_manager.get_npc_count()
    }

if __name__ == "__main__":
    uvicorn.run(
        app,
        host=settings.HOST,
        port=settings.PORT,
        log_level="info"
    )
```

이 메인 프로그램 파일은 FastAPI 애플리케이션의 기본 구조를 정의하고, 크로스 도메인 요청을 허용하기 위한 CORS 미들웨어를 설정하며, 시작 시 각 관리자를 초기화한다. 이제 구체적인 API 라우트를 구현해 보자.

### 15.4.2 API 라우트 설계

사이버 마을의 백엔드는 Godot 프론트엔드의 요청을 처리하기 위한 몇 가지 핵심 API 엔드포인트를 제공해야 한다. 이 라우트들을 `main.py`에 추가한다.

**NPC 상태 가져오기**

이 API는 위치, 바쁜 상태 등 모든 NPC의 현재 상태를 반환한다:

```python
from models import NPCStatusResponse

@app.get("/npcs/status", response_model=NPCStatusResponse)
async def get_npc_status():
    """모든 NPC의 상태 가져오기"""
    npcs = state_manager.get_all_npc_states()
    return {"npcs": npcs}

@app.get("/npcs/{npc_id}/status")
async def get_single_npc_status(npc_id: str):
    """단일 NPC의 상태 가져오기"""
    npc = state_manager.get_npc_state(npc_id)
    if not npc:
        raise HTTPException(status_code=404, detail=f"NPC {npc_id} 가 존재하지 않습니다")
    return npc
```

**대화 인터페이스**

가장 핵심적인 API로, 플레이어와 NPC의 대화를 처리한다:

```python
from models import DialogueRequest, DialogueResponse

@app.post("/dialogue", response_model=DialogueResponse)
async def dialogue(request: DialogueRequest):
    """플레이어와 NPC의 대화 처리"""
    # 1. NPC 존재 여부 확인
    if not agent_manager.has_npc(request.npc_id):
        raise HTTPException(status_code=404, detail=f"NPC {request.npc_id} 가 존재하지 않습니다")

    # 2. NPC가 바쁜지 확인
    if state_manager.is_npc_busy(request.npc_id):
        raise HTTPException(status_code=409, detail=f"NPC {request.npc_id} 가 다른 플레이어와 대화 중입니다")

    # 3. NPC를 바쁜 상태로 표시
    state_manager.set_npc_busy(request.npc_id, True)

    try:
        # 4. 현재 호감도 가져오기
        affinity_info = relationship_manager.get_affinity(
            request.npc_id,
            request.player_name
        )

        # 5. Agent를 호출하여 응답 생성
        agent = agent_manager.get_agent(request.npc_id, affinity_info["level"])
        reply = agent.run(request.player_message)

        # 6. 호감도 업데이트
        new_affinity = relationship_manager.update_affinity(
            request.npc_id,
            request.player_name,
            request.player_message,
            reply
        )

        # 7. 로그 기록
        dialogue_logger.log_dialogue(
            npc_id=request.npc_id,
            player_name=request.player_name,
            player_message=request.player_message,
            npc_reply=reply,
            affinity_info=new_affinity
        )

        # 8. 응답 반환
        return DialogueResponse(
            npc_reply=reply,
            affinity_level=new_affinity["level"],
            affinity_score=new_affinity["score"]
        )

    except Exception as e:
        dialogue_logger.log_error(f"대화 처리 실패: {str(e)}")
        raise HTTPException(status_code=500, detail=f"대화 처리 실패: {str(e)}")

    finally:
        # 9. NPC 상태 해제
        state_manager.set_npc_busy(request.npc_id, False)
```

**호감도 조회**

플레이어와 NPC의 호감도를 조회할 수 있는 API:

```python
from models import AffinityInfo

@app.get("/affinity/{npc_id}/{player_name}", response_model=AffinityInfo)
async def get_affinity(npc_id: str, player_name: str):
    """플레이어와 NPC의 호감도 가져오기"""
    if not agent_manager.has_npc(npc_id):
        raise HTTPException(status_code=404, detail=f"NPC {npc_id} 가 존재하지 않습니다")

    affinity = relationship_manager.get_affinity(npc_id, player_name)
    return affinity
```

API 라우트의 호출 흐름은 그림 15.11과 같다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-11.png" alt="" width="85%"/>
  <p>그림 15.11 API 호출 흐름</p>
</div>

### 15.4.3 상태 관리와 로그 시스템

**상태 관리자**

상태 관리자는 위치, 바쁜 상태, 현재 행동 등 각 NPC의 현재 상태를 추적하는 역할을 담당한다. 동시성 문제, 예를 들어 한 NPC가 여러 플레이어와 동시에 대화하는 상황을 방지하는 데 중요하다.

```python
# state_manager.py
from typing import Dict, List, Optional
from datetime import datetime

class StateManager:
    """NPC 상태 관리자"""

    def __init__(self):
        self.npc_states: Dict[str, dict] = {}

    def initialize_npcs(self):
        """NPC 상태 초기화"""
        npcs = [
            {
                "npc_id": "zhang_san",
                "name": "张三",
                "role": "Python 엔지니어",
                "position": {"x": 300, "y": 200}
            },
            {
                "npc_id": "li_si",
                "name": "李四",
                "role": "프로덕트 매니저",
                "position": {"x": 500, "y": 200}
            },
            {
                "npc_id": "wang_wu",
                "name": "王五",
                "role": "UI 디자이너",
                "position": {"x": 700, "y": 200}
            }
        ]

        for npc in npcs:
            self.npc_states[npc["npc_id"]] = {
                **npc,
                "is_busy": False,
                "current_action": "idle",
                "last_interaction": None
            }

    def get_npc_state(self, npc_id: str) -> Optional[dict]:
        """NPC 상태 가져오기"""
        return self.npc_states.get(npc_id)

    def get_all_npc_states(self) -> List[dict]:
        """모든 NPC 상태 가져오기"""
        return list(self.npc_states.values())

    def is_npc_busy(self, npc_id: str) -> bool:
        """NPC가 바쁜 상태인지 확인"""
        npc = self.npc_states.get(npc_id)
        return npc["is_busy"] if npc else False

    def set_npc_busy(self, npc_id: str, busy: bool):
        """NPC 바쁜 상태 설정"""
        if npc_id in self.npc_states:
            self.npc_states[npc_id]["is_busy"] = busy
            if busy:
                self.npc_states[npc_id]["last_interaction"] = datetime.now().isoformat()

    def get_npc_count(self) -> int:
        """NPC 수 가져오기"""
        return len(self.npc_states)
```

**로그 시스템**

로그 시스템은 콘솔과 파일 두 가지 출력을 구현한다. 이렇게 하면 실시간으로 편리하게 확인할 수 있으면서도 이력 기록을 보존할 수 있다.

```python
# logger.py
import logging
from datetime import datetime
from pathlib import Path

class DialogueLogger:
    """대화 로거"""

    def __init__(self, log_dir: str = "logs"):
        self.log_dir = Path(log_dir)
        self.log_dir.mkdir(exist_ok=True)

        # 로그 파일명 생성(날짜별)
        today = datetime.now().strftime("%Y-%m-%d")
        log_file = self.log_dir / f"dialogue_{today}.log"

        # 로그 설정
        self.logger = logging.getLogger("DialogueLogger")
        self.logger.setLevel(logging.INFO)

        # 콘솔 핸들러
        console_handler = logging.StreamHandler()
        console_handler.setLevel(logging.INFO)
        console_formatter = logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s',
            datefmt='%H:%M:%S'
        )
        console_handler.setFormatter(console_formatter)

        # 파일 핸들러
        file_handler = logging.FileHandler(log_file, encoding='utf-8')
        file_handler.setLevel(logging.INFO)
        file_formatter = logging.Formatter(
            '%(asctime)s - %(levelname)s - %(message)s',
            datefmt='%Y-%m-%d %H:%M:%S'
        )
        file_handler.setFormatter(file_formatter)

        # 핸들러 추가
        self.logger.addHandler(console_handler)
        self.logger.addHandler(file_handler)

    def log_dialogue(self, npc_id: str, player_name: str,
                    player_message: str, npc_reply: str,
                    affinity_info: dict):
        """대화 기록"""
        log_message = f"""
{'='*60}
NPC: {npc_id}
플레이어: {player_name}
플레이어 메시지: {player_message}
NPC 응답: {npc_reply}
호감도: {affinity_info['level']} ({affinity_info['score']}/100)
상호작용 횟수: {affinity_info['interaction_count']}
{'='*60}
"""
        self.logger.info(log_message)

    def log_error(self, error_message: str):
        """오류 기록"""
        self.logger.error(error_message)
```

이 로그 시스템은 콘솔에 대화 내용을 실시간으로 표시하면서 파일에도 저장한다. 매일의 로그는 별도의 파일에 저장되어 이후 분석이 편리하다.

### 15.4.4 Godot의 씬 시스템 이해

게임 씬을 구축하기 전에, Godot의 핵심 개념인 씬(Scene)과 노드(Node)를 먼저 이해해야 한다. 이것이 Godot와 다른 게임 엔진의 가장 큰 차이점이자 가장 강력한 특성 중 하나다.

**노드란?**

노드는 Godot에서 가장 기본적인 구성 단위다. 노드를 레고 블록에 비유할 수 있다. 각 노드는 특정 기능을 가진다. 예를 들어, Sprite2D 노드는 이미지를 표시하고, AudioStreamPlayer 노드는 오디오를 재생하며, CharacterBody2D 노드는 캐릭터의 물리적 이동을 처리한다. Godot는 수백 가지의 다양한 노드 타입을 제공하며, 각 노드는 하나의 일을 잘 하는 데 집중한다.

노드들은 부모-자식 관계를 형성하여 트리 구조를 이룰 수 있다. 부모 노드는 자식 노드에 영향을 줄 수 있어서, 부모 노드를 이동하면 모든 자식 노드도 함께 이동하고, 부모 노드를 숨기면 모든 자식 노드도 함께 숨겨진다. 이 계층적 관계 덕분에 복잡한 게임 오브젝트를 쉽게 구성하고 관리할 수 있다.

**씬이란?**

씬은 노드들의 집합으로, .tscn 파일에 저장된다. 씬을 "프리팹"으로 이해할 수 있다. 예를 들어, 캐릭터의 스프라이트, 충돌체, 음향 등 모든 관련 노드를 포함하는 "플레이어" 씬을 만들 수 있다. 그런 다음 게임에서 이 씬을 여러 번 사용할 수 있으며, 매번 독립적인 인스턴스가 생성된다.

씬의 강력함은 재사용성과 모듈화에 있다. 하나의 씬 안에 다른 씬을 인스턴스화하여 중첩 구조를 만들 수 있다. 예를 들어, 메인 씬은 플레이어 씬, 여러 NPC 씬, UI 씬을 포함할 수 있다. NPC 씬을 수정하면 모든 NPC 인스턴스에 자동으로 반영되어 개발과 유지보수가 크게 간편해진다.

**간단한 예시**

씬과 노드를 이해하기 위해 간단한 예를 들어보자. "플레이어" 씬을 만든다고 가정하면:

```
Player (CharacterBody2D)  ← 루트 노드, 물리적 이동 담당
├─ AnimatedSprite2D       ← 자식 노드, 캐릭터 애니메이션 표시
├─ CollisionShape2D       ← 자식 노드, 충돌 형태 정의
└─ Camera2D               ← 자식 노드, 카메라가 플레이어를 따라감
```

이 씬은 4개의 노드로 구성된 트리 구조를 이룬다. CharacterBody2D가 루트 노드이고, 나머지 세 개가 자식 노드다. 각 노드에 스크립트를 추가하여 동작을 제어하거나, 루트 노드에 스크립트를 추가하여 모든 자식 노드를 조율할 수 있다.

메인 씬에서 이 Player 씬을 인스턴스화할 때, Godot는 이 전체 노드 트리의 복사본을 생성한다. 여러 플레이어 인스턴스를 만들 수 있으며, 각 인스턴스는 독립적으로 자신의 위치, 상태, 동작을 가진다.

**씬 인스턴스화의 장점**

사이버 마을에는 장삼, 이사, 왕오 세 명의 NPC가 있다. 씬 시스템을 사용하지 않으면 각 NPC에 대해 별도로 노드를 생성하고, 속성을 설정하고, 스크립트를 작성해야 하므로 많은 중복 작업이 발생한다. 반면 씬 시스템을 사용하면 범용 NPC 씬을 하나만 만들고 세 번 인스턴스화한 뒤, 스크립트 파라미터로 서로 다른 이름과 역할 정보를 설정하기만 하면 된다.

이 설계의 장점은, 만약 모든 NPC에 새로운 기능(예: 머리 위에 대화 말풍선 표시)을 추가하고 싶다면 NPC 씬만 수정하면 모든 인스턴스가 자동으로 이 기능을 얻게 된다는 것이다.

---

## 15.5 Godot 게임 씬 구축

**왜 Godot를 게임 엔진으로 선택했는가?**

많은 게임 엔진 중에서 Godot 4.5를 프론트엔드 엔진으로 선택한 데는 다음과 같은 주요 이유가 있다:

（1）**Godot는 2D 게임 개발에 있어 천연의 이점을 가진다.** 사이버 마을은 탑다운 시점의 2D 픽셀 스타일 게임이다. Godot의 2D 엔진은 매우 성숙하며, TileMap, AnimatedSprite2D, CharacterBody2D 등 2D 게임을 위해 설계된 전용 노드 타입을 제공하여 Unity 등의 엔진보다 개발 효율이 훨씬 높다. Godot의 씬 시스템은 플레이어, NPC, UI 등의 요소를 독립적인 씬으로 캡슐화한 다음 메인 씬에서 인스턴스화할 수 있게 해주며, 이런 컴포넌트화 설계가 우리의 요구에 매우 잘 맞는다.

（2）**Godot는 완전한 오픈소스이며 무료다.** Godot는 MIT 라이선스를 사용하여 어떠한 저작권 비용이나 수익 배분도 없다. 소스 코드를 자유롭게 수정할 수 있으며, 라이선스 문제를 걱정하지 않고 게임을 상업화할 수도 있다. 반면 Unity는 기능이 강력하지만 2024년에 런타임 요금 정책을 도입하여 개발자 커뮤니티에서 광범위한 논란을 일으켰다.

（3）**Godot의 학습 비용이 매우 낮다.** Godot는 주요 스크립트 언어로 GDScript를 사용한다. 이는 Python과 유사한 동적 타입 언어로, 문법이 간결하고 학습 곡선이 매우 완만하다. 이미 Python에 익숙한 독자라면 GDScript 학습에 거의 장벽이 없다 — 변수 선언, 함수 정의, 제어 흐름 등 문법이 Python과 매우 유사하여 몇 시간 안에 게임 스크립트 작성을 시작할 수 있다. Godot의 노드 트리 구조도 매우 직관적이어서, 에디터에서 씬의 계층 관계를 직관적으로 볼 수 있어 초보자에게 매우 친숙하다.

（4）**Godot와 Python 백엔드의 통합이 매우 간단하다.** Godot는 HTTPRequest 노드를 내장하고 있어 FastAPI 백엔드와의 HTTP 통신을 쉽게 할 수 있다. API 클라이언트 스크립트를 하나 만들어 모든 API 호출을 캡슐화하기만 하면, 게임에서 백엔드의 AI 기능을 호출할 수 있다. 이 프론트엔드와 백엔드가 분리된 아키텍처 덕분에 게임 로직과 AI 로직을 독립적으로 개발하고 테스트할 수 있어 개발 효율이 크게 향상된다.

물론 Godot에도 한계가 있다. 예를 들어, Godot의 3D 기능은 Unreal Engine이나 Unity에 비해 아직 격차가 있어서, 대규모 3D 게임을 개발한다면 다른 엔진을 고려해야 할 수 있다. 하지만 2D 게임, 인디 게임, 교육 프로젝트에 있어서 Godot는 매우 훌륭한 선택이다.

### 15.5.1 씬 설계와 리소스 구성

Godot의 씬 시스템을 이해했으므로, 이제 사이버 마을의 씬 설계를 살펴보자. 전체 게임은 네 가지 핵심 씬으로 구성된다: Main(메인 씬), Player(플레이어), NPC(비플레이어 캐릭터), DialogueUI(대화 인터페이스). 각 씬은 독립적인 모듈로, 개별적으로 편집하고 테스트한 다음 조합하여 완전한 게임을 만들 수 있다.

사이버 마을의 씬 구성은 모듈화 설계를 채택한다. 먼저 Player(플레이어), NPC(비플레이어 캐릭터), DialogueUI(대화 인터페이스) 세 가지 기본 씬을 만든다. 그런 다음 Main(메인 씬)에서 이 씬들을 인스턴스화하고 조합한다. 특히 주목할 점은, 세 명의 NPC(장삼, 이사, 왕오)가 모두 동일한 NPC 씬의 인스턴스이며, 스크립트 파라미터로 서로 다른 역할 정보를 설정한다는 것이다.

네 가지 핵심 씬의 구조는 그림 15.12와 같다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-12.png" alt="" width="85%"/>
  <p>그림 15.12 사이버 마을의 네 가지 핵심 씬</p>
</div>

이 그림은 네 개의 독립적인 씬과 내부 구조를 보여준다. **씬 1(Main)**은 메인 씬으로, 배경 이미지(Sprite2D), 플레이어 인스턴스, NPCs 조직 노드(그 아래에 세 개의 NPC 인스턴스), 대화 인터페이스 인스턴스, 벽 조직 노드, 배경 음악을 포함한다. Player, NPC_Zhang, NPC_Li, NPC_Wang, DialogueUI는 모두 씬 인스턴스이며 일반 노드가 아님에 주목해야 한다. **씬 2(Player)**는 플레이어 캐릭터의 구조를 정의하며, 애니메이션, 충돌, 카메라, 두 개의 음향 노드를 포함한다. **씬 3(NPC)**은 범용 템플릿으로, 장삼, 이사, 왕오 모두 이 씬의 인스턴스이며, 충돌, 애니메이션, 상호작용 영역, 두 개의 레이블을 포함한다. **씬 4(DialogueUI)**는 CanvasLayer 노드로, Panel과 각종 UI 요소를 포함한다.

씬 인스턴스화 과정은 이렇게 이해할 수 있다: Godot 에디터에서 NPC.tscn 씬 파일을 만들어 NPC의 노드 구조를 정의한다. 그런 다음 Main 씬에서 이 NPC 씬을 세 번 "인스턴스화"하여 각각 NPC_Zhang, NPC_Li, NPC_Wang으로 이름 붙인 세 개의 독립적인 복사본을 만든다. 각 복사본은 자신의 위치와 상태를 가지지만 동일한 노드 구조를 공유한다. NPC.tscn을 수정하면, 예를 들어 새로운 음향 노드를 추가하면 세 인스턴스 모두 자동으로 이 음향을 얻게 된다.

Godot에서 이 씬들을 만드는 단계:

1. **Player 씬 만들기**: 새 씬을 만들고 CharacterBody2D를 루트 노드로 선택한 뒤, AnimatedSprite2D, CollisionShape2D, Camera2D, InteractSound, RunningSound 자식 노드를 추가하고 Player.tscn으로 저장한다.

2. **NPC 씬 만들기**: 새 씬을 만들고 CharacterBody2D를 루트 노드로 선택한 뒤, CollisionShape2D, AnimatedSprite2D, InteractionArea(Area2D, 아래에 CollisionShape2D), NameLabel, DialogueLabel 자식 노드를 추가하고 NPC.tscn으로 저장한다.

3. **DialogueUI 씬 만들기**: 새 씬을 만들고 CanvasLayer를 루트 노드로 선택한 뒤, Panel 자식 노드를 추가하고, Panel 아래에 NPCName, NPCTitle, DialogueText(RichTextLabel), PlayerInput(LineEdit), SendButton, CloseButton을 추가하고 DialogueUI.tscn으로 저장한다.

4. **Main 씬 만들기**: 새 씬을 만들고 Node2D를 루트 노드로 선택한 뒤, Background(Sprite2D)를 배경 이미지로 추가하고, Background 아래에 작은 고래 장식을 추가한다. 그런 다음 Player 씬을 인스턴스화하고, NPCs 노드를 만들어 그 아래에 NPC 씬을 세 번 인스턴스화한다. DialogueUI 씬을 인스턴스화하고, Walls 노드를 만들어 벽 충돌을 구성한 뒤, 마지막으로 AudioStreamPlayer를 추가하여 배경 음악을 재생한다.

이런 씬 구성 방식의 장점은: 각 씬이 독립적이어서 단독으로 테스트할 수 있고, NPC는 동일한 씬의 인스턴스를 사용하여 한 번만 수정하면 모든 NPC에 영향을 줄 수 있으며, 씬 간에 신호로 통신하여 결합도가 낮고 유지보수 및 확장이 용이하다는 것이다.

### 15.5.2 플레이어 제어 구현

플레이어 캐릭터는 게임에서 가장 중요한 요소 중 하나다. WASD 이동 제어, 애니메이션 전환, 충돌 감지, NPC와의 상호작용 및 음향 시스템을 구현해야 한다.

플레이어 씬의 구조는 다음과 같다: CharacterBody2D가 루트 노드로 물리적 이동과 충돌을 담당하고, AnimatedSprite2D가 캐릭터 애니메이션을 표시하며, CollisionShape2D가 충돌 형태를 정의하고, Camera2D가 플레이어를 따라다니며, 두 개의 AudioStreamPlayer가 각각 상호작용 음향과 걸음 음향을 재생한다.

플레이어 제어 스크립트 `player.gd`는 이동, 상호작용 및 음향 로직을 구현한다:

```python
extends CharacterBody2D

# 이동 속도
@export var speed: float = 200.0

# 현재 상호작용 가능한 NPC
var nearby_npc: Node = null

# 상호작용 상태(상호작용 중 이동 비활성화)
var is_interacting: bool = false

# 노드 참조
@onready var animated_sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var camera: Camera2D = $Camera2D

# 음향 참조
@onready var interact_sound: AudioStreamPlayer = null
@onready var running_sound: AudioStreamPlayer = null

# 걸음 음향 상태
var is_playing_running_sound: bool = false

func _ready():
    # player 그룹에 추가(중요! NPC는 이 그룹으로 플레이어를 식별)
    add_to_group("player")

    # 음향 노드 가져오기(선택사항, 없어도 오류 발생 안 함)
    interact_sound = get_node_or_null("InteractSound")
    running_sound = get_node_or_null("RunningSound")

    # 카메라 활성화
    camera.enabled = true

    # 기본 애니메이션 재생
    if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
        animated_sprite.play("idle")

func _physics_process(_delta: float):
    # 상호작용 중이면 이동 비활성화
    if is_interacting:
        velocity = Vector2.ZERO
        move_and_slide()
        # idle 애니메이션 재생
        if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")
        # 걸음 음향 정지
        stop_running_sound()
        return

    # 입력 방향 가져오기
    var input_direction = Input.get_vector("ui_left", "ui_right", "ui_up", "ui_down")

    # 속도 설정
    velocity = input_direction * speed

    # 이동
    move_and_slide()

    # 애니메이션 및 방향 업데이트
    update_animation(input_direction)

    # 걸음 음향 업데이트
    update_running_sound(input_direction)

func update_animation(direction: Vector2):
    """캐릭터 애니메이션 업데이트(4방향 지원)"""
    if animated_sprite.sprite_frames == null:
        return

    # 이동 방향에 따라 애니메이션 재생
    if direction.length() > 0:
        # 이동 중 - 주요 방향 판단
        if abs(direction.x) > abs(direction.y):
            # 좌우 이동
            if direction.x > 0:
                # 오른쪽
                if animated_sprite.sprite_frames.has_animation("walk_right"):
                    animated_sprite.play("walk_right")
                    animated_sprite.flip_h = false
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = false
            else:
                # 왼쪽
                if animated_sprite.sprite_frames.has_animation("walk_left"):
                    animated_sprite.play("walk_left")
                    animated_sprite.flip_h = false
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = true
        else:
            # 상하 이동
            if direction.y > 0:
                # 아래
                if animated_sprite.sprite_frames.has_animation("walk_down"):
                    animated_sprite.play("walk_down")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
            else:
                # 위
                if animated_sprite.sprite_frames.has_animation("walk_up"):
                    animated_sprite.play("walk_up")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
    else:
        # 정지
        if animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")

func _input(event: InputEvent):
    # E키로 NPC와 상호작용
    if event is InputEventKey:
        if event.pressed and not event.echo:
            if event.keycode == KEY_E or event.keycode == KEY_ENTER:
                if nearby_npc != null:
                    interact_with_npc()

func interact_with_npc():
    """근처 NPC와 상호작용"""
    if nearby_npc != null:
        # 상호작용 음향 재생
        if interact_sound:
            interact_sound.play()

        # 대화 시스템에 신호 전송
        get_tree().call_group("dialogue_system", "start_dialogue", nearby_npc.npc_name)

func set_nearby_npc(npc: Node):
    """근처 NPC 설정"""
    nearby_npc = npc

func set_interacting(interacting: bool):
    """상호작용 상태 설정"""
    is_interacting = interacting
    if interacting:
        # 걸음 음향 정지
        stop_running_sound()

func update_running_sound(direction: Vector2):
    """걸음 음향 업데이트"""
    if running_sound == null:
        return

    # 이동 중이면
    if direction.length() > 0:
        # 음향이 재생되지 않으면 시작
        if not is_playing_running_sound:
            running_sound.play()
            is_playing_running_sound = true
    else:
        # 이동 멈추면 음향 정지
        stop_running_sound()

func stop_running_sound():
    """걸음 음향 정지"""
    if running_sound and is_playing_running_sound:
        running_sound.stop()
        is_playing_running_sound = false
```

이 스크립트는 완전한 플레이어 제어를 구현한다. 플레이어는 WASD 키(또는 방향키)로 이동하며, 캐릭터는 이동 방향에 따라 해당하는 4방향 애니메이션(walk_up/down/left/right)을 재생한다. 플레이어가 NPC 근처에 가면 NPC는 `set_nearby_npc(self)`를 호출하여 자신을 상호작용 가능한 대상으로 설정하고, 플레이어가 E키를 누르면 상호작용이 시작된다. 상호작용 시 음향이 재생되며, `call_group()`을 통해 대화 시스템에 대화 시작을 알린다. 대화 중에는 `set_interacting(true)`로 플레이어 이동이 비활성화되고, 대화가 끝나면 이동이 복구된다. 걸음 음향은 플레이어가 이동할 때 자동으로 재생되고 멈출 때 자동으로 정지된다.

### 15.5.3 NPC 행동과 상호작용

NPC는 세 가지 핵심 기능을 구현해야 한다: 씬 안에서 무작위로 순찰하며 이동하기, 플레이어의 상호작용에 응답하기, 대화 말풍선 표시하기. Area2D를 사용하여 플레이어가 NPC 근처에 있는지 감지하며, 플레이어가 상호작용 범위에 들어오면 플레이어에게 알림을 보내고, 플레이어가 E키를 누르면 대화가 시작된다.

NPC 씬의 구조는 다음과 같다: CharacterBody2D가 루트 노드, CollisionShape2D가 NPC의 충돌 형태를 정의하고, AnimatedSprite2D가 NPC 애니메이션을 표시하며, InteractionArea(Area2D)가 플레이어의 상호작용 범위 진입을 감지하고(아래에 CollisionShape2D로 상호작용 범위 정의), NameLabel이 NPC 이름을 표시하며, DialogueLabel이 대화 말풍선을 표시한다.

NPC 스크립트 `npc.gd`는 순찰, 상호작용, 대화 말풍선 로직을 구현한다:

```python
extends CharacterBody2D

# NPC 정보
@export var npc_name: String = "张三"
@export var npc_title: String = "Python 엔지니어"

# NPC 외관 설정
@export var sprite_frames: SpriteFrames = null  # 커스텀 스프라이트 프레임 리소스

# NPC 이동 설정
@export var move_speed: float = 50.0  # 이동 속도
@export var wander_enabled: bool = true  # 순찰 활성화 여부
@export var wander_range: float = 200.0  # 순찰 범위
@export var wander_interval_min: float = 3.0  # 최소 순찰 간격(초)
@export var wander_interval_max: float = 8.0  # 최대 순찰 간격(초)

# 현재 대화 내용(백엔드에서 가져옴)
var current_dialogue: String = ""

# 노드 참조
@onready var animated_sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var interaction_area: Area2D = $InteractionArea
@onready var name_label: Label = $NameLabel
@onready var dialogue_label: Label = $DialogueLabel

# 플레이어 참조
var player: Node = null

# 순찰 관련 변수
var wander_target: Vector2 = Vector2.ZERO  # 순찰 목표 위치
var wander_timer: float = 0.0  # 순찰 타이머
var is_wandering: bool = false  # 순찰 중인지 여부
var is_interacting: bool = false  # 플레이어와 상호작용 중인지 여부
var spawn_position: Vector2 = Vector2.ZERO  # 스폰 위치

func _ready():
    # npcs 그룹에 추가
    add_to_group("npcs")

    # NPC 이름 설정
    name_label.text = npc_name

    # 상호작용 영역 신호 연결
    interaction_area.body_entered.connect(_on_body_entered)
    interaction_area.body_exited.connect(_on_body_exited)

    # 대화 레이블 초기화
    dialogue_label.text = ""
    dialogue_label.visible = false

    # 커스텀 스프라이트 프레임 설정(있는 경우)
    if sprite_frames != null:
        animated_sprite.sprite_frames = sprite_frames

    # 기본 애니메이션 재생
    if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
        animated_sprite.play("idle")

    # 스폰 위치 기록
    spawn_position = global_position

    # 순찰 타이머 초기화
    if wander_enabled:
        wander_timer = randf_range(wander_interval_min, wander_interval_max)
        choose_new_wander_target()

func _on_body_entered(body: Node2D):
    """플레이어가 상호작용 범위에 진입"""
    if body.is_in_group("player"):
        player = body

        if player.has_method("set_nearby_npc"):
            player.set_nearby_npc(self)

func _on_body_exited(body: Node2D):
    """플레이어가 상호작용 범위에서 이탈"""
    if body.is_in_group("player"):
        if player != null and player.has_method("set_nearby_npc"):
            player.set_nearby_npc(null)
        player = null

func update_dialogue(dialogue: String):
    """NPC 대화 내용 업데이트"""
    current_dialogue = dialogue
    dialogue_label.text = dialogue
    dialogue_label.visible = true

    # 10초 후 대화 숨기기
    await get_tree().create_timer(10.0).timeout
    dialogue_label.visible = false

func _physics_process(delta: float):
    """물리 업데이트 - 이동 처리"""
    # 플레이어와 상호작용 중이면 이동 정지
    if is_interacting:
        velocity = Vector2.ZERO
        move_and_slide()
        # idle 애니메이션 재생
        if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")
        return

    # 순찰이 비활성화되어 있으면 이동 안 함
    if not wander_enabled:
        return

    # 순찰 타이머 업데이트
    wander_timer -= delta

    # 타이머가 끝나면 새 목표 선택 후 이동 시작
    if wander_timer <= 0:
        choose_new_wander_target()
        wander_timer = randf_range(wander_interval_min, wander_interval_max)

    # 순찰 중이면 목표로 이동
    if is_wandering:
        # 목표에 도달했는지 확인
        if global_position.distance_to(wander_target) < 10:
            # 목표 도달, 이동 정지
            is_wandering = false
            velocity = Vector2.ZERO
            move_and_slide()
            # idle 애니메이션 재생
            if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
                animated_sprite.play("idle")
        else:
            # 계속 목표로 이동
            var direction = (wander_target - global_position).normalized()
            velocity = direction * move_speed
            move_and_slide()
            # 애니메이션 업데이트
            update_animation(direction)
    else:
        # 이동 정지
        velocity = Vector2.ZERO
        move_and_slide()
        # idle 애니메이션 재생
        if animated_sprite.sprite_frames != null and animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")

func choose_new_wander_target():
    """새 순찰 목표 선택"""
    # 스폰 위치 근처에서 무작위 지점 선택
    var offset = Vector2(
        randf_range(-wander_range, wander_range),
        randf_range(-wander_range, wander_range)
    )
    wander_target = spawn_position + offset
    is_wandering = true

func update_animation(direction: Vector2):
    """애니메이션 업데이트"""
    if animated_sprite.sprite_frames == null:
        return

    if direction.length() > 0:
        # 이동 애니메이션
        if abs(direction.x) > abs(direction.y):
            # 좌우 이동
            if direction.x > 0:
                if animated_sprite.sprite_frames.has_animation("walk_right"):
                    animated_sprite.play("walk_right")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = false
            else:
                if animated_sprite.sprite_frames.has_animation("walk_left"):
                    animated_sprite.play("walk_left")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
                    animated_sprite.flip_h = true
        else:
            # 상하 이동
            if direction.y > 0:
                if animated_sprite.sprite_frames.has_animation("walk_down"):
                    animated_sprite.play("walk_down")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
            else:
                if animated_sprite.sprite_frames.has_animation("walk_up"):
                    animated_sprite.play("walk_up")
                elif animated_sprite.sprite_frames.has_animation("walk"):
                    animated_sprite.play("walk")
    else:
        # 정지 애니메이션
        if animated_sprite.sprite_frames.has_animation("idle"):
            animated_sprite.play("idle")

func set_interacting(interacting: bool):
    """상호작용 상태 설정"""
    is_interacting = interacting
```

이 스크립트는 NPC의 완전한 행동을 구현한다. NPC는 스폰 위치 근처의 `wander_range` 범위 내에서 무작위로 순찰하며, `wander_interval_min`에서 `wander_interval_max`초마다 새로운 목표 지점을 선택하여 이동한다. 이동 시 4방향 애니메이션(walk_up/down/left/right)을 재생하고, 목표에 도달하면 정지하여 idle 애니메이션을 재생한다. 플레이어가 InteractionArea에 진입하면 NPC는 플레이어의 `set_nearby_npc(self)` 메서드를 호출하여 자신을 상호작용 가능한 대상으로 설정한다. 플레이어가 E키를 누르면 대화 시스템이 NPC의 `set_interacting(true)` 메서드를 호출하고 NPC는 이동을 멈춘다. 대화가 끝나면 `set_interacting(false)`를 호출하여 NPC는 순찰을 재개한다. 메인 씬은 주기적으로 `update_dialogue()` 메서드를 호출하여 NPC의 대화 말풍선을 업데이트하며, NPC 간의 자율적인 대화 내용을 표시한다.

---

## 15.6 프론트엔드-백엔드 통신 구현

### 15.6.1 API 클라이언트 캡슐화

Godot 프론트엔드는 FastAPI 백엔드와 HTTP 통신을 해야 한다. API 클라이언트 스크립트 `api_client.gd`를 만들어 모든 API 호출을 캡슐화하고, 이를 AutoLoad(자동 로드) 싱글톤으로 설정하여 다른 스크립트가 편리하게 사용할 수 있도록 한다.

API 클라이언트는 Godot의 HTTPRequest 노드를 사용하여 HTTP 요청을 보낸다. HTTPRequest는 비동기 노드로, 요청을 보낸 후 게임을 차단하지 않고 신호를 통해 요청 완료를 알린다. 이를 통해 네트워크 지연이 높아도 게임이 매끄럽게 동작한다. await 대신 신호 메커니즘을 사용하여 다른 스크립트에 API 응답을 알리므로, 여러 스크립트가 동시에 동일한 API 응답을 수신할 수 있다.

```python
# api_client.gd
extends Node

# 신호 정의
signal chat_response_received(npc_name: String, message: String)
signal chat_error(error_message: String)
signal npc_status_received(dialogues: Dictionary)
signal npc_list_received(npcs: Array)

# HTTP 요청 노드
var http_chat: HTTPRequest
var http_status: HTTPRequest
var http_npcs: HTTPRequest

func _ready():
    # HTTP 요청 노드 생성
    http_chat = HTTPRequest.new()
    http_status = HTTPRequest.new()
    http_npcs = HTTPRequest.new()

    add_child(http_chat)
    add_child(http_status)
    add_child(http_npcs)

    # 신호 연결
    http_chat.request_completed.connect(_on_chat_request_completed)
    http_status.request_completed.connect(_on_status_request_completed)
    http_npcs.request_completed.connect(_on_npcs_request_completed)

# ==================== 대화 API ====================
func send_chat(npc_name: String, message: String) -> void:
    """대화 요청 전송"""
    var data = {
        "npc_name": npc_name,
        "message": message
    }

    var json_string = JSON.stringify(data)
    var headers = ["Content-Type: application/json"]

    var error = http_chat.request(
        Config.API_CHAT,
        headers,
        HTTPClient.METHOD_POST,
        json_string
    )

    if error != OK:
        print("[ERROR] 대화 요청 전송 실패: ", error)
        chat_error.emit("네트워크 요청 실패")

func _on_chat_request_completed(_result: int, response_code: int, _headers: PackedStringArray, body: PackedByteArray) -> void:
    """대화 응답 처리"""
    if response_code != 200:
        print("[ERROR] 대화 요청 실패: HTTP ", response_code)
        chat_error.emit("서버 오류: " + str(response_code))
        return

    var json = JSON.new()
    var parse_result = json.parse(body.get_string_from_utf8())

    if parse_result != OK:
        print("[ERROR] 응답 파싱 실패")
        chat_error.emit("응답 파싱 실패")
        return

    var response = json.data

    if response.has("success") and response["success"]:
        var npc_name = response["npc_name"]
        var msg = response["message"]
        print("[INFO] NPC 응답 수신: ", npc_name, " -> ", msg)
        chat_response_received.emit(npc_name, msg)
    else:
        chat_error.emit("대화 실패")

# ==================== NPC 상태 API ====================
func get_npc_status() -> void:
    """NPC 상태 가져오기"""
    # 요청 처리 중인지 확인
    if http_status.get_http_client_status() != HTTPClient.STATUS_DISCONNECTED:
        print("[WARN] NPC 상태 요청 처리 중, 이번 요청 건너뜀")
        return

    var error = http_status.request(Config.API_NPC_STATUS)

    if error != OK:
        print("[ERROR] NPC 상태 가져오기 실패: ", error)

func _on_status_request_completed(_result: int, response_code: int, _headers: PackedStringArray, body: PackedByteArray) -> void:
    """NPC 상태 응답 처리"""
    if response_code != 200:
        print("[ERROR] NPC 상태 요청 실패: HTTP ", response_code)
        return

    var json = JSON.new()
    var parse_result = json.parse(body.get_string_from_utf8())

    if parse_result != OK:
        print("[ERROR] NPC 상태 파싱 실패")
        return

    var response = json.data

    if response.has("dialogues"):
        var dialogues = response["dialogues"]
        print("[INFO] NPC 상태 업데이트 수신: ", dialogues.size(), "명의 NPC")
        npc_status_received.emit(dialogues)

# ==================== NPC 목록 API ====================
func get_npc_list() -> void:
    """NPC 목록 가져오기"""
    var error = http_npcs.request(Config.API_NPCS)

    if error != OK:
        print("[ERROR] NPC 목록 가져오기 실패: ", error)

func _on_npcs_request_completed(_result: int, response_code: int, _headers: PackedStringArray, body: PackedByteArray) -> void:
    """NPC 목록 응답 처리"""
    if response_code != 200:
        print("[ERROR] NPC 목록 요청 실패: HTTP ", response_code)
        return

    var json = JSON.new()
    var parse_result = json.parse(body.get_string_from_utf8())

    if parse_result != OK:
        print("[ERROR] NPC 목록 파싱 실패")
        return

    var response = json.data

    if response.has("npcs"):
        var npcs = response["npcs"]
        print("[INFO] NPC 목록 수신: ", npcs.size(), "명의 NPC")
        npc_list_received.emit(npcs)
```

이 API 클라이언트는 세 가지 핵심 기능을 캡슐화한다: 대화 요청 전송(`send_chat`), NPC 상태 가져오기(`get_npc_status`), NPC 목록 가져오기(`get_npc_list`). 모든 HTTP 요청은 비동기로 처리되며 신호를 통해 응답 결과를 알린다. 각 API에 독립적인 HTTPRequest 노드를 만들어 여러 요청을 동시에 보내도 서로 간섭하지 않는다. API의 URL은 Config 싱글톤에서 가져와 통합 관리가 편리하다. 대화 시스템은 `chat_response_received` 신호를 수신하여 NPC의 응답을 받고, 메인 씬은 `npc_status_received` 신호를 수신하여 NPC의 대화 말풍선을 업데이트한다.

### 15.6.2 대화 UI 구현

대화 UI는 플레이어가 NPC와 상호작용하는 인터페이스다. NPC 이름, 직위, 대화 내용 표시, 입력 창, 버튼을 포함하는 간결하고 아름다운 대화창을 설계해야 한다.

대화 UI의 구조는 그림 15.13과 같다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-13.png" alt="" width="85%"/>
  <p>그림 15.13 대화 UI 구조</p>
</div>

대화 UI의 설계는 매우 간결하다. DialogueUI는 CanvasLayer 노드로, 게임 화면의 최상위 레이어에 항상 표시되어 다른 게임 오브젝트에 가려지지 않는다. Panel은 대화창의 배경으로, 화면 하단에 고정된다. Panel 아래에 6개의 UI 요소가 있다: NPCName은 NPC의 이름을 표시하고, NPCTitle은 직위를 표시하며, DialogueText는 RichTextLabel을 사용하여 대화 내용을 표시(리치 텍스트 형식 지원)하고, PlayerInput은 플레이어 입력을 위한 LineEdit이며, SendButton과 CloseButton은 각각 메시지 전송과 대화창 닫기에 사용된다.

대화 UI 스크립트 `dialogue_ui.gd`는 대화 인터페이스의 로직을 구현한다:

```python
# dialogue_ui.gd
extends CanvasLayer

# UI 노드 참조
@onready var panel = $Panel
@onready var npc_name_label = $Panel/NPCName
@onready var npc_title_label = $Panel/NPCTitle
@onready var dialogue_text = $Panel/DialogueText
@onready var input_field = $Panel/PlayerInput
@onready var send_button = $Panel/SendButton
@onready var close_button = $Panel/CloseButton

# API 클라이언트
var api_client: Node = null

# 현재 대화 중인 NPC
var current_npc_name: String = ""

func _ready():
    # 초기화 시 대화창 숨기기
    visible = false

    # 버튼 신호 연결
    send_button.pressed.connect(_on_send_button_pressed)
    close_button.pressed.connect(_on_close_button_pressed)
    input_field.text_submitted.connect(_on_text_submitted)

    # API 클라이언트 가져오기
    api_client = get_node_or_null("/root/APIClient")

func start_dialogue(npc_name: String):
    """NPC와의 대화 시작"""
    current_npc_name = npc_name

    # NPC 정보 설정
    npc_name_label.text = npc_name
    npc_title_label.text = get_npc_title(npc_name)

    # 대화 내용 초기화
    dialogue_text.clear()
    dialogue_text.append_text("[color=gray]" + npc_name + "과(와)의 대화를 시작합니다...[/color]\n")

    # 입력창 초기화
    input_field.text = ""

    # 대화창 표시
    show_dialogue()

    # 입력창에 포커스
    input_field.grab_focus()

func show_dialogue():
    """대화창 표시"""
    visible = true

    # 플레이어에게 상호작용 상태 알림(이동 비활성화)
    var player = get_tree().get_first_node_in_group("player")
    if player and player.has_method("set_interacting"):
        player.set_interacting(true)

func hide_dialogue():
    """대화창 숨기기"""
    visible = false
    current_npc_name = ""

    # 플레이어에게 상호작용 상태 종료 알림(이동 활성화)
    var player = get_tree().get_first_node_in_group("player")
    if player and player.has_method("set_interacting"):
        player.set_interacting(false)

func _on_send_button_pressed():
    """전송 버튼 클릭"""
    send_message()

func _on_close_button_pressed():
    """닫기 버튼 클릭"""
    hide_dialogue()

func _on_text_submitted(_text: String):
    """입력창 엔터"""
    send_message()

func send_message():
    """메시지 전송"""
    var message = input_field.text.strip_edges()

    if message.is_empty():
        return

    if current_npc_name.is_empty():
        return

    # 플레이어 메시지 표시
    dialogue_text.append_text("\n[color=cyan]플레이어:[/color] " + message + "\n")

    # 입력창 초기화
    input_field.text = ""

    # 입력 비활성화
    input_field.editable = false
    send_button.disabled = true

    # API 요청 전송
    if api_client:
        api_client.send_chat_request(current_npc_name, message)

func on_chat_response_received(npc_name: String, response: String):
    """NPC 응답 수신"""
    if npc_name == current_npc_name:
        # NPC 응답 표시
        dialogue_text.append_text("[color=yellow]" + npc_name + ":[/color] " + response + "\n")

        # 입력 활성화
        input_field.editable = true
        send_button.disabled = false
        input_field.grab_focus()

func get_npc_title(npc_name: String) -> String:
    """NPC 직위 가져오기"""
    var titles = {
        "张三": "Python 엔지니어",
        "李四": "프로덕트 매니저",
        "王五": "UI 디자이너"
    }
    return titles.get(npc_name, "")
```

이 대화 UI는 완전한 대화 기능을 구현한다. 플레이어는 메시지를 입력하고 전송할 수 있으며, UI는 RichTextLabel의 append_text 메서드를 사용하여 대화 내용을 표시하고 리치 텍스트 형식(색상, 굵기 등)을 지원한다. 모든 API 호출은 비동기로 처리되며, 응답을 기다리는 동안 입력창을 비활성화하여 중복 전송을 방지한다. 대화창이 표시될 때 플레이어에게 상호작용 상태를 알려 이동을 비활성화하고, 닫을 때 이동을 복구한다.

### 15.6.3 메인 씬 통합

마지막으로 메인 씬에서 플레이어 제어, NPC 상호작용, 대화 UI, NPC 상태 업데이트 등 모든 기능을 통합해야 한다. 메인 씬 스크립트 `main.gd`는 이 컴포넌트들을 조율하며, 백엔드에서 주기적으로 NPC 상태를 가져와 NPC의 대화 말풍선을 업데이트한다.

```python
# main.gd
extends Node2D

# NPC 노드 참조
@onready var npc_zhang: Node2D = $NPCs/NPC_Zhang
@onready var npc_li: Node2D = $NPCs/NPC_Li
@onready var npc_wang: Node2D = $NPCs/NPC_Wang

# API 클라이언트
var api_client: Node = null

# NPC 상태 업데이트 타이머
var status_update_timer: float = 0.0

func _ready():
    print("[INFO] 메인 씬 초기화")

    # API 클라이언트 가져오기
    api_client = get_node_or_null("/root/APIClient")
    if api_client:
        api_client.npc_status_received.connect(_on_npc_status_received)

        # 즉시 한 번 NPC 상태 가져오기
        api_client.get_npc_status()
    else:
        print("[ERROR] API 클라이언트를 찾을 수 없음")

func _process(delta: float):
    # 주기적으로 NPC 상태 업데이트
    status_update_timer += delta
    if status_update_timer >= Config.NPC_STATUS_UPDATE_INTERVAL:
        status_update_timer = 0.0
        if api_client:
            api_client.get_npc_status()

func _on_npc_status_received(dialogues: Dictionary):
    """NPC 상태 업데이트 수신"""
    print("[INFO] NPC 상태 업데이트 중: ", dialogues)

    # 각 NPC의 대화 업데이트
    for npc_name in dialogues:
        var dialogue = dialogues[npc_name]
        update_npc_dialogue(npc_name, dialogue)

func update_npc_dialogue(npc_name: String, dialogue: String):
    """지정된 NPC의 대화 업데이트"""
    var npc_node = get_npc_node(npc_name)
    if npc_node and npc_node.has_method("update_dialogue"):
        npc_node.update_dialogue(dialogue)

func get_npc_node(npc_name: String) -> Node2D:
    """이름으로 NPC 노드 가져오기"""
    match npc_name:
        "张三":
            return npc_zhang
        "李四":
            return npc_li
        "王五":
            return npc_wang
        _:
            return null
```

메인 씬 스크립트의 핵심 기능은 백엔드에서 주기적으로 NPC 상태를 가져오는 것이다. `_ready()`에서 APIClient 싱글톤의 참조를 가져와 `npc_status_received` 신호를 연결한다. 그런 다음 즉시 `get_npc_status()`를 호출하여 한 번 NPC 상태를 가져온다. `_process()`에서 타이머를 사용하여 `Config.NPC_STATUS_UPDATE_INTERVAL`초(기본 30초)마다 `get_npc_status()`를 한 번 호출한다. NPC 상태 업데이트를 받으면 `_on_npc_status_received()` 콜백 함수가 모든 NPC를 순회하며 그들의 `update_dialogue()` 메서드를 호출하여 대화 말풍선을 업데이트한다. 이를 통해 플레이어가 NPC와 상호작용하지 않아도 NPC 간의 자율적인 대화를 볼 수 있다.

전체 프론트엔드-백엔드 통신 흐름은 그림 15.14와 같다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-14.png" alt="" width="85%"/>
  <p>그림 15.14 프론트엔드-백엔드 통신 전체 흐름</p>
</div>

이로써 프론트엔드-백엔드 통신의 모든 기능이 구현되었다. 플레이어는 게임에서 자유롭게 이동하고, NPC와 상호작용하며, 자연어로 대화할 수 있다. 동시에 메인 씬은 백엔드에서 주기적으로 NPC 상태를 가져와 NPC의 대화 말풍선을 업데이트하여 NPC 간의 자율적인 대화를 보여준다. 전체 시스템은 신호 메커니즘으로 통신하며, 각 컴포넌트 간에 느슨한 결합을 유지하여 유지보수와 확장이 용이하다.

---

## 15.7 정리와 전망

### 15.7.1 이 장 요약

이 장에서 완전한 AI 마을 프로젝트인 사이버 마을을 완성했다. 이 프로젝트는 HelloAgents 프레임워크와 Godot 게임 엔진을 결합하여 생명력 넘치는 가상 세계를 창조했다. 우리가 배운 핵심 내용을 되짚어보자.

**기술 아키텍처 설계**

게임 엔진 + 백엔드 서비스의 분리 아키텍처를 채택하여 프론트엔드 렌더링, 백엔드 로직, AI 지능을 서로 다른 계층으로 분리했다. Godot는 게임 화면과 플레이어 상호작용을 담당하고, FastAPI는 API 서비스와 상태 관리를 담당하며, HelloAgents는 NPC 지능과 기억 시스템을 담당한다. 이 계층적 설계 덕분에 각 부분을 독립적으로 개발하고 테스트할 수 있으며, 이후 확장을 위한 좋은 기반도 마련된다.

**NPC 지능 에이전트 시스템**

HelloAgents의 SimpleAgent를 사용하여 각 NPC에 독립적인 지능 에이전트를 만들었다. 각 NPC는 자신의 역할 설정, 성격 특징, 기억 시스템을 가진다. 정교하게 설계된 시스템 프롬프트를 통해 장삼을 엄격한 Python 엔지니어로, 이사를 소통을 잘하는 프로덕트 매니저로, 왕오를 창의적인 UI 디자이너로 만들었다. 이 NPC들은 플레이어의 대화를 이해할 뿐만 아니라 자신의 역할 특성에 맞게 응답할 수도 있다.

**기억과 호감도 시스템**

두 계층의 기억 시스템을 구현했다: 단기 기억은 대화의 연속성을 유지하고, 장기 기억은 모든 상호작용 이력을 저장한다. 벡터 데이터베이스의 의미론적 검색을 통해 NPC는 이전에 논의했던 주제를 회상할 수 있다. 호감도 시스템은 NPC의 플레이어에 대한 태도가 상호작용에 따라 변화하도록 하며, 낯선 사이에서 절친한 벗까지 각 등급마다 다른 행동 표현을 가진다. 이 설계 덕분에 NPC가 더욱 현실적이고 재미있게 느껴진다.

**게임 씬 구축**

Godot를 사용하여 픽셀 스타일의 사무실 장면을 만들고, 플레이어 제어, NPC 순찰, 상호작용 감지, 대화 UI를 구현했다. 씬 시스템의 모듈화 설계 덕분에 새로운 NPC, 새로운 장면, 새로운 기능을 쉽게 추가할 수 있다. GDScript의 간결한 문법 덕분에 게임 로직 구현이 직관적이고 효율적이다.

**프론트엔드-백엔드 통신**

HTTP REST API를 사용하여 Godot 프론트엔드와 FastAPI 백엔드 간의 통신을 구현했다. 비동기 요청과 신호 시스템을 통해 게임의 매끄러운 동작을 보장하며, 네트워크 지연이 높아도 플레이어 경험에 영향을 주지 않는다. API 클라이언트의 캡슐화 덕분에 다른 스크립트에서 백엔드 서비스를 편리하게 호출할 수 있으며, 대화 UI의 구현 덕분에 플레이어가 NPC와 자연스럽게 소통할 수 있다.

전체 프로젝트의 기술 스택은 그림 15.15와 같다:

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/15-figures/15-15.png" alt="" width="85%"/>
  <p>그림 15.15 사이버 마을 기술 스택</p>
</div>

### 15.7.2 확장 방향

사이버 마을은 시작점에 불과하며, 확장 가능한 방향이 많다. 이 확장들은 게임의 재미를 더할 뿐만 아니라 AI 기술의 게임 내 더 많은 가능성을 탐구하는 데도 도움이 된다.

**（1）멀티플레이어 온라인 지원**

현재 사이버 마을은 싱글 플레이어 게임이지만, 멀티플레이어 온라인 게임으로 확장할 수 있다. 여러 플레이어가 동시에 같은 사무실에 들어와 NPC 및 다른 플레이어와 상호작용할 수 있다. 이를 위해 실시간 통신을 위한 WebSocket과 플레이어 데이터 및 NPC 상태를 영속화하기 위한 데이터베이스가 필요하다. NPC는 서로 다른 플레이어와의 상호작용을 기억하며 각 플레이어에 대해 독립적인 호감도를 유지할 수 있다.

**（2）퀘스트 시스템**

NPC에 퀘스트 시스템을 설계할 수 있다. 플레이어와 NPC의 호감도가 일정 수준에 도달하면 NPC가 특별 퀘스트를 제공한다. 예를 들어 장삼은 플레이어에게 코드 디버깅을 도와달라고 요청하고, 이사는 사용자 피드백 수집을 요청하며, 왕오는 디자인 방안 평가를 요청할 수 있다. 퀘스트를 완료하면 보상을 받을 수 있고 호감도도 더욱 높아진다.

**（3）NPC 간 상호작용**

현재 NPC는 플레이어와만 상호작용하지만, NPC 간에도 상호작용하도록 만들 수 있다. 장삼과 이사는 제품 요구사항을 논의하고, 이사와 왕오는 인터페이스 설계를 논의하며, 왕오와 장삼은 기술 구현을 논의할 수 있다. 이 상호작용들은 백그라운드에서 자동으로 이루어지며, 플레이어는 NPC 간의 대화를 관찰할 수 있어 전체 세계가 더욱 생동감 있게 느껴진다.

**（4）감정 시스템**

호감도 외에 NPC에 더욱 복잡한 감정 시스템을 추가할 수 있다. NPC는 기쁨, 슬픔, 화남, 흥분 등 다양한 감정 상태를 가질 수 있으며, 이 감정들은 NPC의 응답 스타일과 행동에 영향을 준다. 예를 들어 NPC의 기분이 좋을 때는 정보를 더 기꺼이 공유하고, 기분이 좋지 않을 때는 다소 냉담할 수 있다.

**（5）동적 이벤트 시스템**

게임 세계를 더욱 풍부하게 만들기 위한 동적 이벤트를 설계할 수 있다. 예를 들어 정기적으로 팀 회의를 열어 모든 NPC와 플레이어가 모여 프로젝트 진행 상황을 논의하거나, 생일 파티를 열어 특정 NPC의 생일을 축하하거나, 긴급 임무가 발생하여 모두가 협력하여 완료해야 하는 상황을 만들 수 있다. 이 이벤트들은 게임의 변화성과 재미를 높인다.

**（6）더 넓은 세계**

현재 사이버 마을은 하나의 사무실 장면만 있지만, 더 넓은 세계로 확장할 수 있다. 카페, 도서관, 공원 등 다양한 장면을 추가하고 각 장면에 서로 다른 NPC와 상호작용 방식을 배치할 수 있다. 플레이어는 서로 다른 장면 사이를 이동하며 더 넓은 가상 세계를 탐험할 수 있다.

**（7）개인화된 학습**

NPC는 각 플레이어의 취향과 습관을 학습할 수 있다. 예를 들어 플레이어가 장삼과 자주 Python을 논의한다면, NPC는 플레이어가 프로그래밍에 관심이 있다는 것을 기억하고 이후에 관련 내용을 자발적으로 공유한다. 플레이어가 저녁에 게임을 즐겨 한다면, NPC는 이 시간 습관을 기억하여 저녁에 더욱 활발하게 활동한다.

### 15.7.3 생각과 전망

사이버 마을은 AI 기술의 게임 내 거대한 잠재력을 보여준다. 전통적인 게임의 NPC는 미리 설정된 대화 트리와 스크립트의 제약을 받지만, AI NPC는 자연어를 이해하고 생성하여 플레이어와 진정한 대화를 나눌 수 있다. 이는 게임의 몰입감을 높일 뿐만 아니라 게임 디자인에 새로운 가능성을 가져다준다.

하지만 AI NPC도 몇 가지 도전에 직면한다. 첫째로 비용 문제가 있다. 매번 대화마다 LLM API를 호출해야 하므로 일정한 비용이 발생한다. 대규모 멀티플레이어 온라인 게임에서 이 비용은 상당히 높아질 수 있다. 둘째로 지연 문제가 있다. LLM의 추론에는 시간이 필요하며, 네트워크 지연이 높으면 플레이어가 몇 초를 기다려야 NPC의 응답을 볼 수 있다. 마지막으로 콘텐츠 제어 문제가 있다. LLM이 생성하는 콘텐츠는 완전히 통제 가능하지 않을 수 있어, 잘 설계된 프롬프트와 콘텐츠 필터링 메커니즘이 필요하다.

이러한 도전에도 불구하고 AI NPC의 미래는 여전히 희망적이다. LLM 기술의 발전과 함께 추론 속도는 점점 빨라지고 비용은 점점 낮아질 것이다. 로컬 소형 LLM도 빠르게 발전하고 있어, 미래에는 플레이어의 기기에서 직접 실행하여 네트워크 요청이 전혀 필요 없게 될 수도 있다. AI 기술과 게임의 결합은 플레이어에게 전례 없는 경험을 가져다 줄 것이다.

5부의 졸업 설계 장에서는 단일 에이전트와 다중 에이전트로 범용 에이전트를 구성하는 방법을 배울 것이다. 그것은 당신의 창작 시간이 될 것이니 기대해 주기 바란다!