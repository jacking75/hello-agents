# 제1장: 에이전트 소개

에이전트의 세계에 오신 것을 환영합니다! 인공지능의 물결이 전 세계를 휩쓸고 있는 오늘날, **에이전트(Agent)**는 기술 변혁과 응용 혁신을 이끄는 핵심 개념 중 하나가 되었습니다. AI 분야의 연구자나 엔지니어를 목표로 하든, 기술의 최전선을 깊이 이해하고자 하는 관찰자이든, 에이전트의 본질을 파악하는 것은 여러분의 지식 체계에서 빠질 수 없는 부분이 될 것입니다.

따라서 이 장에서는 기본으로 돌아가 몇 가지 질문을 함께 탐구해 보겠습니다: 에이전트란 무엇인가? 주요 유형은 무엇인가? 우리가 사는 세계와 어떻게 상호작용하는가? 이러한 논의를 통해 향후 학습과 탐구를 위한 견고한 토대를 마련하고자 합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-0.png" alt="Figure description" width="90%"/>
  <p>그림 1.1 에이전트와 환경 간의 기본 상호작용 루프</p>
</div>

## 1.1 에이전트란 무엇인가?

복잡한 개념을 탐구할 때는 간결한 정의에서 시작하는 것이 가장 좋습니다. 인공지능 분야에서 에이전트는 **센서(Sensor)**를 통해 **환경(Environment)**을 인식하고, **작동기(Actuator)**를 통해 특정 목표를 달성하기 위해 **자율적으로** **행동(Action)**을 취할 수 있는 모든 개체로 정의됩니다.

이 정의에는 에이전트 존재의 네 가지 기본 요소가 담겨 있습니다. 환경은 에이전트가 운용되는 외부 세계입니다. 자율주행차에게 환경은 역동적으로 변화하는 도로 교통이고, 거래 알고리즘에게는 끊임없이 변화하는 금융 시장입니다. 에이전트는 환경과 단절되어 있지 않으며, 센서를 통해 환경 상태를 지속적으로 인식합니다. 카메라, 마이크, 레이더, 또는 다양한 **응용 프로그래밍 인터페이스(API)**가 반환하는 데이터 스트림은 모두 인식 능력의 확장입니다.

정보를 획득한 후, 에이전트는 작동기를 통해 환경에 영향을 미치고 그 상태를 변화시키는 행동을 취해야 합니다. 작동기는 물리적 장치(로봇 팔이나 조향 장치 등)가 될 수도 있고, 가상 도구(코드 실행이나 서비스 호출 등)가 될 수도 있습니다.

그러나 에이전트에게 "지능"을 부여하는 것은 바로 **자율성(Autonomy)**입니다. 에이전트는 외부 자극에 수동적으로 반응하거나 미리 설정된 명령을 엄격히 실행하는 프로그램이 아니라, 인식과 내부 상태에 기반하여 설계 목표를 달성하기 위한 독립적인 결정을 내릴 수 있습니다. 인식에서 행동으로 이어지는 이 닫힌 루프가 모든 에이전트 행동의 토대를 형성하며, 그림 1.1에 나타나 있습니다.

### 1.1.1 전통적 관점에서의 에이전트

현재의 **대규모 언어 모델(LLM)** 물결이 오기 전, 인공지능의 선구자들은 이미 수십 년 동안 "에이전트"라는 개념을 탐구하고 구축해 왔습니다. 우리가 지금 "전통적 에이전트"라고 부르는 이 패러다임은 단일한 정적 개념이 아니라, 단순한 것에서 복잡한 것으로, 수동적 반응에서 능동적 학습으로 나아가는 뚜렷한 진화 경로를 걸어왔습니다.

이 진화의 출발점은 구조적으로 가장 단순한 **단순 반사 에이전트(Simple Reflex Agent)**입니다. 이들의 의사결정 핵심은 엔지니어가 명시적으로 설계한 "조건-행동" 규칙으로 구성되어 있으며, 그림 1.2에 나타나 있습니다. 전형적인 자동 온도 조절기가 이런 방식으로 작동합니다: 센서가 실내 온도가 설정값보다 높다고 인식하면 냉각 시스템을 작동시킵니다.

이 유형의 에이전트는 현재의 인식 입력에만 전적으로 의존하며, 기억이나 예측 능력이 없습니다. 디지털화된 본능과 같아서 신뢰할 수 있고 효율적이지만, 문맥 이해가 필요한 복잡한 과제를 처리할 수 없습니다. 이러한 한계는 핵심 질문을 제기합니다: 현재 환경 상태만으로는 의사결정의 충분한 근거가 되지 않을 때 에이전트는 어떻게 해야 하는가?

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-1.png" alt="Figure description" width="90%"/>
  <p>그림 1.2 단순 반사 에이전트의 의사결정 논리 다이어그램</p>
</div>

이 질문에 답하기 위해 연구자들은 "상태"라는 개념을 도입하고 **모델 기반 반사 에이전트(Model-Based Reflex Agent)**를 개발했습니다. 이 유형의 에이전트는 직접 인식할 수 없는 환경의 측면을 추적하고 이해하는 데 사용되는 내부 **세계 모델(World Model)**을 가지고 있습니다. 이는 "세상은 지금 어떠한가?"라는 질문에 답하려고 합니다. 예를 들어, 터널을 통과하는 자율주행차는 카메라가 앞차를 일시적으로 인식하지 못하더라도, 내부 모델은 그 차량의 존재, 속도, 추정 위치에 대한 판단을 유지합니다. 이 내부 모델은 에이전트에게 원시적인 "기억" 형태를 부여하여, 그 결정이 더 이상 순간적인 인식에만 의존하지 않고 세계 상태에 대한 보다 일관되고 완전한 이해에 기반하게 만듭니다.

그러나 세상을 이해하는 것만으로는 충분하지 않습니다—에이전트에게는 명확한 목표가 필요합니다. 이것이 **목표 기반 에이전트(Goal-Based Agent)**의 발전으로 이어졌습니다. 이전 두 유형과 달리, 이들의 행동은 환경에 수동적으로 반응하는 것이 아니라 특정 미래 상태로 이어질 수 있는 행동을 능동적으로 선택합니다. 이 유형의 에이전트가 답해야 하는 질문은 "목표를 달성하기 위해 무엇을 해야 하는가?"입니다. 전형적인 예는 GPS 내비게이션 시스템입니다: 목표는 사무실에 도착하는 것이고, 에이전트는 지도 데이터(세계 모델)를 기반으로 탐색 알고리즘(예: A*)을 사용하여 최적 경로를 계획합니다. 이 유형의 에이전트의 핵심 역량은 미래에 대한 고려와 계획에 반영됩니다.

더 나아가, 현실 세계의 목표는 단일하지 않은 경우가 많습니다. 우리는 사무실에 도착하기를 원할 뿐만 아니라, 가장 짧은 시간, 가장 연료 효율적인 경로, 혼잡을 피하는 것도 원합니다. 여러 목표를 균형 잡아야 할 때, **효용 기반 에이전트(Utility-Based Agent)**가 등장합니다. 이들은 가능한 모든 세계 상태에 만족 수준을 나타내는 효용 값을 부여합니다. 에이전트의 핵심 목표는 더 이상 단순히 특정 상태를 달성하는 것이 아니라 기대 효용을 극대화하는 것입니다. "어떤 행동이 나에게 가장 만족스러운 결과를 가져올 것인가?"라는 보다 복잡한 질문에 답해야 합니다. 이 아키텍처는 에이전트가 상충하는 목표들 사이에서 균형을 학습할 수 있게 하여, 그 결정이 인간의 합리적 선택에 더 가까워지게 합니다.

지금까지 논의한 에이전트들은 기능이 점점 더 복잡해지더라도, 핵심 의사결정 논리(규칙, 모델 또는 효용 함수)는 여전히 인간 설계자의 사전 지식에 의존합니다. 에이전트가 사전 설정에 의존하지 않고 환경과의 상호작용을 통해 자율적으로 학습할 수 있다면 어떨까요?

이것이 **학습 에이전트(Learning Agent)**의 핵심 아이디어이며, **강화 학습(RL, Reinforcement Learning)**은 이 아이디어를 실현하는 가장 대표적인 경로입니다. 학습 에이전트는 수행 요소(앞서 논의한 다양한 유형의 에이전트)와 학습 요소를 포함합니다. 학습 요소는 환경에서 수행 요소의 행동 결과를 관찰하여 수행 요소의 의사결정 전략을 지속적으로 수정합니다.

AI가 체스를 배우는 것을 상상해보세요. 처음에는 무작위로 움직일 수 있지만, 마침내 한 게임에서 이기면 시스템은 긍정적인 보상을 줍니다. 방대한 자기 대국을 통해 학습 요소는 어떤 움직임이 최종 승리로 이어질 가능성이 더 높은지 점차 발견합니다. AlphaGo Zero는 이 철학의 이정표적 성과입니다. 복잡한 바둑 게임에서 강화 학습을 통해 기존 인간 지식을 뛰어넘는 많은 효과적인 전략을 발견했습니다.

단순한 온도 조절기에서 내부 모델을 갖춘 자동차로, 경로를 계획할 수 있는 내비게이션으로, 장단점을 따질 줄 아는 의사결정자로, 마침내 경험을 통해 자기 진화할 수 있는 학습자로. 이 진화 경로는 전통적 인공지능이 기계 지능을 구축하는 과정에서 걸어온 발전 궤적을 보여줍니다. 이들은 오늘날 우리가 보다 최첨단 에이전트 패러다임을 이해하는 데 견고하고 필요한 기반을 마련했습니다.

### 1.1.2 대규모 언어 모델이 이끄는 새로운 패러다임

**GPT(Generative Pre-trained Transformer)**로 대표되는 대규모 언어 모델의 등장은 에이전트의 구축 방법과 역량 경계를 크게 변화시키고 있습니다. LLM으로 구동되는 에이전트는 핵심 의사결정 메커니즘에서 전통적 에이전트와 근본적으로 다르기 때문에, 완전히 새로운 특성을 갖추게 됩니다.

이 변화는 핵심 엔진, 지식 출처, 상호작용 방식 등 여러 차원에서 두 가지를 비교할 때 명확하게 볼 수 있으며, 표 1.1에 나타나 있습니다. 간단히 말해, 전통적 에이전트의 역량은 엔지니어의 명시적 프로그래밍과 지식 구성에서 비롯되며, 행동 패턴이 결정론적이고 한정적입니다. 반면 LLM 에이전트는 방대한 데이터에 대한 사전 훈련을 통해 암묵적 세계 모델과 강력한 창발적 역량을 갖추어, 보다 유연하고 범용적인 방식으로 복잡한 과제를 처리할 수 있습니다.

<div align="center">
  <p>표 1.1 전통적 에이전트와 LLM 기반 에이전트의 핵심 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-2.png" alt="Figure description" width="90%"/>
</div>

이러한 차이는 LLM 에이전트가 고수준의 모호하고 문맥이 풍부한 자연어 명령을 직접 처리할 수 있게 합니다. "지능형 여행 도우미"를 예로 들어 설명해 보겠습니다.

LLM 에이전트가 등장하기 전, 여행을 계획하기 위해서는 일반적으로 사용자가 여러 전용 앱(날씨, 지도, 예약 사이트 등)을 직접 전환해야 했으며, 사용자 자신이 정보 통합 및 의사결정 역할을 해야 했습니다. 반면 LLM 에이전트는 이 과정을 통합할 수 있습니다. "샤먼 여행을 계획해줘"와 같은 모호한 명령을 받았을 때, 그 작동 방식은 다음과 같은 점을 반영합니다:

- **계획 및 추론**: 에이전트는 먼저 이 고수준 목표를 일련의 논리적 하위 과제로 분해합니다. 예를 들어: `[여행 선호도 확인] -> [목적지 정보 조회] -> [여정 초안 작성] -> [티켓 및 숙박 예약]`. 이것은 내부적이고 모델 주도적인 계획 과정입니다.
- **도구 사용**: 계획을 실행할 때 에이전트는 정보 격차를 파악하고 이를 채우기 위해 외부 도구를 능동적으로 호출합니다. 예를 들어, 날씨 조회 인터페이스를 호출하여 실시간 날씨를 확인하고, "비가 예보됨"이라는 정보를 바탕으로 후속 계획에서 실내 활동을 추천하는 경향을 보입니다.
- **동적 조정**: 상호작용 과정에서 에이전트는 사용자 피드백(예: "이 호텔은 예산을 초과합니다")을 새로운 제약 조건으로 처리하고 후속 행동을 그에 맞게 조정하여, 새로운 요구 사항을 충족하는 옵션을 다시 검색하고 추천합니다. "**날씨 확인 → 일정 조정 → 호텔 예약**"이라는 전체 과정은 문맥에 기반하여 행동을 동적으로 수정하는 능력을 보여줍니다.

요약하면, 우리는 전문화된 자동화 도구를 개발하는 것에서 문제를 자율적으로 해결할 수 있는 시스템을 구축하는 것으로 이동하고 있습니다. 핵심은 더 이상 코드를 작성하는 것이 아니라 범용적인 "두뇌"가 계획하고, 행동하고, 학습하도록 안내하는 것입니다.

### 1.1.3 에이전트의 유형

위의 에이전트 진화 검토에 이어, 이 절에서는 세 가지 보완적인 차원에서 에이전트를 분류합니다.

(1) **내부 의사결정 아키텍처에 따른 분류**

첫 번째 분류 차원은 에이전트의 내부 의사결정 아키텍처의 복잡성을 기반으로 합니다. 이 관점은 "인공지능: 현대적 접근법"<sup>[1]</sup>에서 체계적으로 제안되었습니다. 1.1.1절에서 설명했듯이, 전통적 에이전트의 진화 경로 자체가 가장 전형적인 분류 사다리를 구성합니다. 단순한 **반응형** 에이전트에서 내부 모델을 도입한 **모델 기반** 에이전트, 그리고 더 미래 지향적인 **목표 기반** 및 **효용 기반** 에이전트에 이르는 범위를 포괄합니다. 또한 **학습 능력**은 위의 모든 유형에 부여될 수 있는 메타 역량으로, 경험을 통해 자기 개선이 가능하게 합니다.

(2) **시간 및 반응성에 따른 분류**

내부 아키텍처의 복잡성 외에도, 에이전트는 의사결정 처리의 시간 차원에서 분류할 수 있습니다. 이 관점은 에이전트가 정보를 받은 후 즉시 행동하는지, 아니면 신중한 계획 후에 행동하는지에 초점을 맞춥니다. 이는 에이전트 설계의 핵심 트레이드오프를 드러냅니다: 속도를 추구하는 **반응성(Reactivity)**과 최적 해를 추구하는 **숙고(Deliberation)** 사이의 균형으로, 그림 1.3에 나타나 있습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-3.png" alt="Figure description" width="90%"/>
  <p>그림 1.3 에이전트 의사결정 시간과 품질의 관계</p>
</div>

- **반응형 에이전트(Reactive Agents)**

이 유형의 에이전트는 매우 낮은 의사결정 지연으로 환경 자극에 거의 즉각적으로 반응합니다. 이들은 일반적으로 인식에서 행동으로의 직접적인 매핑을 따르며, 미래 계획이 없거나 최소화되어 있습니다. 위에서 언급한 **단순 반응형** 및 **모델 기반** 에이전트가 이 범주에 속합니다.

이들의 핵심 장점은 **빠른 속도와 낮은 계산 부하**로, 신속한 의사결정이 필요한 동적 환경에서 매우 중요합니다. 예를 들어, 차량의 에어백 시스템은 충돌 후 밀리초 내에 반응해야 하며, 지연이 있으면 심각한 결과를 초래할 수 있습니다. 마찬가지로 고빈도 거래 로봇은 순간적인 시장 기회를 포착하기 위해 반응형 의사결정에 의존해야 합니다. 그러나 이 속도의 대가는 "근시안"입니다. 장기 계획이 없어 반응형 에이전트는 쉽게 지역 최적해에 빠지며, 다단계 조율이 필요한 복잡한 과제를 완수하기 어렵습니다.

- **숙고형 에이전트(Deliberative Agents)**

반응형 에이전트와 달리, 숙고형(또는 계획형) 에이전트는 행동하기 전에 복잡한 사고와 계획에 참여합니다. 이들은 인식에 즉시 반응하지 않고, 내부 세계 모델을 사용하여 다양한 미래 가능성을 체계적으로 탐색하고 다양한 행동 순서의 결과를 평가하여 목표 달성을 위한 최적 경로를 찾으려 합니다. **목표 기반** 및 **효용 기반** 에이전트가 전형적인 숙고형 에이전트입니다.

이들의 의사결정 과정은 체스 플레이어에 비유할 수 있습니다. 즉각적인 수만 보는 것이 아니라 가능한 상대방의 반응을 예상하고 수십 수 앞까지 후속 수를 계획합니다. 이 숙고 능력은 사업 계획 수립이나 장거리 여행 계획과 같이 장기적 시야가 필요한 복잡한 과제를 처리할 수 있게 합니다. 이들의 장점은 결정의 전략적 특성과 선견지명에 있습니다. 그러나 이 장점의 이면은 높은 시간 및 계산 비용입니다. 빠르게 변화하는 환경에서 숙고형 에이전트가 깊이 생각하고 있는 동안, 행동을 취할 최선의 순간은 이미 지나갔을 수 있습니다.

- **하이브리드 에이전트(Hybrid Agents)**

현실 세계의 복잡한 과제는 종종 즉각적인 반응과 장기 계획 모두를 필요로 합니다. 예를 들어, 앞서 언급한 지능형 여행 도우미는 사용자의 즉각적인 피드백(예: "이 호텔은 너무 비싸요")에 기반하여 추천을 조정(반응성)하면서, 동시에 완전한 다일정 여행 일정을 계획(숙고)할 수 있어야 합니다. 따라서 두 가지의 장점을 결합하여 반응과 계획 사이의 균형을 달성하는 하이브리드 에이전트가 등장했습니다.

전형적인 하이브리드 아키텍처는 계층적 설계입니다: 하위 레이어는 긴급 상황과 기본 행동을 처리하는 빠른 반응형 모듈이고, 상위 레이어는 장기 목표 수립을 담당하는 숙고형 계획 모듈입니다. 현대 LLM 에이전트는 더 유연한 하이브리드 모드를 보여줍니다. 이들은 일반적으로 "생각-행동-관찰" 루프에서 작동하며, 두 가지 모드를 영리하게 통합합니다:

- **추론(Reasoning)**: "생각" 단계에서 LLM은 현재 상황을 분석하고 다음의 합리적인 행동을 계획합니다. 이것은 숙고형 과정입니다.
- **행동 및 관찰(Acting & Observing)**: "행동" 및 "관찰" 단계에서 에이전트는 외부 도구나 환경과 상호작용하고 즉각적으로 피드백을 받습니다. 이것은 반응형 과정입니다.

이 접근 방식을 통해 에이전트는 장기 계획이 필요한 큰 과제를 일련의 "계획-반응" 마이크로 루프로 분해합니다. 이를 통해 즉각적인 환경 변화에 유연하게 대응하면서도 일관된 단계를 통해 복잡한 장기 목표를 달성할 수 있습니다.

**(3) 지식 표현에 따른 분류**

이것은 에이전트가 의사결정에 사용하는 지식이 "마음" 안에서 어떤 형태로 존재하는지를 탐구하는 보다 근본적인 분류 차원입니다. 이 질문은 인공지능 분야에서 반세기 이상 지속된 논쟁의 핵심에 있으며, 두 가지 뚜렷이 다른 AI 문화를 형성했습니다.

- **기호주의 AI(Symbolic AI)**

기호주의는 종종 전통적 인공지능이라고도 불리며, 핵심 신념은 지능이 기호에 대한 논리적 연산에서 비롯된다는 것입니다. 여기서 기호는 인간이 읽을 수 있는 개체(단어, 개념 등)이며, 연산은 엄격한 논리 규칙을 따릅니다. 그림 1.4의 왼쪽에 나타나 있습니다. 이것은 꼼꼼한 사서가 세계 지식을 명확한 규칙 기반과 지식 그래프로 정리하는 것과 같습니다.

주요 장점은 투명성과 해석 가능성입니다. 추론 단계가 명시적이기 때문에 의사결정 과정을 완전히 추적할 수 있으며, 금융 및 의료와 같은 고위험 분야에서 매우 중요합니다. 그러나 "아킬레스건"은 취약성에 있습니다: 완전한 규칙 시스템에 의존하지만, 모호함과 예외로 가득한 현실 세계에서 규칙이 포괄하지 못하는 새로운 상황이 발생하면 시스템이 완전히 실패할 수 있습니다. 이것이 소위 "지식 획득 병목 현상"입니다.

- **하위 기호주의 AI(Sub-symbolic AI)**

하위 기호주의, 또는 연결주의는 완전히 다른 그림을 제시합니다. 여기서 지식은 명시적인 규칙이 아니라 방대한 데이터에서 학습된 통계적 패턴을 나타내는 수많은 뉴런으로 구성된 복잡한 네트워크에 암묵적으로 분산되어 있습니다. 신경망과 딥러닝이 그 대표입니다.

그림 1.4의 가운데에 나타나 있듯이, 기호주의 AI가 사서라면, 하위 기호주의 AI는 옹알이하는 아이와 같습니다. "고양이는 네 발이 있고, 털이 있고, 야옹한다"와 같은 규칙을 학습하여 고양이를 인식하는 것이 아니라, 수천 장의 고양이 사진을 본 후 뇌의 신경망이 "고양이"라는 개념의 시각적 패턴을 식별할 수 있습니다. 이 접근 방식의 강점은 패턴 인식 능력과 잡음이 있는 데이터에 대한 강건성에 있습니다. 기호주의 AI에게 극히 어려운 과제인 이미지, 소리와 같은 비구조화 데이터를 쉽게 처리할 수 있습니다.

그러나 이 강력한 직관적 능력에는 불투명성이 따릅니다. 하위 기호주의 시스템은 일반적으로 **블랙 박스(Black Box)**로 봅니다. 놀라운 정확도로 사진에서 고양이를 식별할 수 있지만, "왜 이것이 고양이라고 생각하나요?"라고 물으면 논리적으로 건전한 설명을 제공하지 못할 가능성이 높습니다. 또한 순수한 논리적 추론 과제에서 성능이 낮으며, 때로는 합리적으로 보이지만 사실적으로 부정확한 환각을 생성합니다.

- **신경-기호주의 AI(Neuro-Symbolic AI)**

오랫동안 기호주의와 하위 기호주의의 두 진영은 두 개의 평행선처럼 발전했습니다. 위의 두 패러다임의 한계를 극복하기 위해 "대화해"라는 아이디어가 등장했는데, 이것이 신경-기호주의 AI, 즉 신경-기호주의 하이브리드입니다. 목표는 두 패러다임의 장점을 통합하여 신경망처럼 데이터에서 학습하고 기호 시스템처럼 논리적 추론을 수행할 수 있는 하이브리드 에이전트를 만드는 것입니다. 인식과 인지, 직관과 이성 사이의 간격을 메우려 합니다. 노벨 경제학상 수상자 대니얼 카너먼이 그의 책 "생각, 빠르고 느리게"에서 제안한 이중 체계 이론은 신경-기호주의를 이해하는 훌륭한 비유를 제공합니다<sup>[2]</sup>. 그림 1.4에 나타나 있습니다:

- **시스템 1**은 빠르고, 직관적이며, 병렬적인 사고 모드로, 하위 기호주의 AI의 강력한 패턴 인식 능력과 유사합니다.
- **시스템 2**는 느리고, 방법론적이며, 논리 기반의 숙고적 사고로, 기호주의 AI의 추론 과정과 같습니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-4.png" alt="Figure description" width="90%"/>
  <p>그림 1.4 기호주의, 하위 기호주의 및 신경-기호주의 하이브리드의 지식 표현 패러다임</p>
</div>

인간의 지능은 이 두 시스템의 협력적 작업에서 비롯됩니다. 마찬가지로, 진정으로 강건한 AI도 두 가지의 강점을 결합해야 합니다. 대규모 언어 모델 기반 에이전트는 신경-기호주의의 훌륭한 실용적 예입니다. 핵심은 거대한 신경망으로, 패턴 인식과 언어 생성 능력을 부여합니다. 그러나 작동할 때 생각, 계획, API 호출 등 일련의 구조화된 중간 단계를 생성하는데, 이것들은 모두 명시적이고 조작 가능한 기호입니다. 이 접근 방식을 통해 인식과 인지, 직관과 이성의 초보적인 융합을 달성합니다.

## 1.2 에이전트의 구성과 작동 원리

### 1.2.1 과제 환경 정의

에이전트가 어떻게 작동하는지 이해하려면 먼저 에이전트가 운용되는 **과제 환경(task environment)**을 이해해야 합니다. 인공지능 분야에서는 **PEAS 모델**을 사용하여 과제 환경을 정밀하게 설명합니다. **성능 지표(Performance measure), 환경(Environment), 작동기(Actuators), 센서(Sensors)**를 분석합니다. 앞서 언급한 지능형 여행 도우미를 예로 들면, 아래 표 1.2는 PEAS 모델을 사용하여 그 과제 환경을 어떻게 명시하는지 보여줍니다.

<div align="center">
  <p>표 1.2 지능형 여행 도우미의 PEAS 설명</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-6.png" alt="Figure description" width="90%"/>
</div>

실제로 LLM 에이전트가 운용되는 디지털 환경은 에이전트 설계에 직접적인 영향을 미치는 몇 가지 복잡한 특성을 나타냅니다.

첫째, 환경은 일반적으로 **부분적으로 관측 가능(partially observable)**합니다. 예를 들어, 여행 도우미가 항공편을 조회할 때, 모든 항공사의 실시간 좌석 정보를 한꺼번에 얻을 수 없습니다. 호출하는 항공편 예약 API가 반환하는 부분적인 데이터만 볼 수 있어, 에이전트는 기억(조회된 경로 기억) 및 탐색(다른 조회 날짜 시도) 능력을 갖춰야 합니다.

둘째, 행동의 결과가 항상 결정론적이지 않습니다. 결과의 예측 가능성에 따라 환경은 **결정론적(deterministic)**과 **확률적(stochastic)**으로 나뉩니다. 여행 도우미의 과제 환경은 전형적인 확률적 환경입니다. 항공권 가격을 검색할 때 두 번의 연속 호출이 서로 다른 가격과 남은 좌석 수를 반환할 수 있어, 에이전트가 불확실성을 처리하고 변화를 모니터링하며 적시에 결정을 내리는 능력을 갖춰야 합니다.

또한 환경에는 다른 행위자들이 존재할 수 있어 **다중 에이전트(multi-agent)** 환경을 형성합니다. 여행 도우미의 경우 다른 사용자의 예약 행동, 다른 자동화 스크립트, 심지어 항공사의 동적 가격 시스템도 모두 환경의 다른 "에이전트"입니다. 이들의 행동(예: 마지막 할인 티켓 예약)은 여행 도우미가 운용되는 환경 상태를 직접 변경하여, 에이전트의 신속한 반응과 전략 선택에 더 높은 요구를 부과합니다.

마지막으로, 거의 모든 과제는 **순차적(sequential)**이고 **동적(dynamic)** 환경에서 발생합니다. "순차적"은 현재 행동이 미래에 영향을 미친다는 것을 의미하고, "동적"은 에이전트가 의사결정을 내리는 동안 환경 자체가 변할 수 있다는 것을 의미합니다. 이는 에이전트의 "인식-생각-행동-관찰" 루프가 지속적으로 변화하는 세계에 빠르고 유연하게 적응할 수 있어야 함을 요구합니다.

### 1.2.2 에이전트 작동 메커니즘

에이전트가 운용되는 과제 환경을 정의한 후, 그 핵심 작동 메커니즘을 탐구해 보겠습니다. 에이전트는 과제를 한 번에 완수하는 것이 아니라, 환경과의 지속적인 루프를 통해 상호작용합니다. 이 핵심 메커니즘을 **에이전트 루프(Agent Loop)**라고 합니다. 그림 1.5에 나타나 있듯이, 이 루프는 에이전트와 환경 간의 동적 상호작용 과정을 설명하며, 자율적 행동의 토대를 형성합니다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-5.png" alt="Figure description" width="90%"/>
  <p>그림 1.5 에이전트-환경 상호작용의 기본 루프</p>
</div>

이 루프는 주로 다음과 같은 상호 연결된 단계들을 포함합니다:

1. **인식(Perception)**: 루프의 시작점입니다. 에이전트는 센서(예: API 수신 포트, 사용자 입력 인터페이스)를 통해 환경으로부터 입력 정보를 받습니다. 이 정보, 즉 **관찰(Observation)**은 사용자의 초기 명령이거나 이전 행동으로 인한 환경 상태 변화에 대한 피드백일 수 있습니다.
2. **사고(Thought)**: 관찰 정보를 받은 후, 에이전트는 핵심 의사결정 단계에 들어갑니다. LLM 에이전트의 경우, 이것은 일반적으로 대규모 언어 모델이 구동하는 내부 추론 과정입니다. 그림에 나타나 있듯이, "사고" 단계는 두 가지 핵심 연결 고리로 더 세분화될 수 있습니다:
   - **계획(Planning)**: 현재 관찰과 내부 기억을 기반으로, 에이전트는 과제와 환경에 대한 이해를 업데이트하고 행동 계획을 수립하거나 조정합니다. 이것은 복잡한 목표를 더 구체적인 하위 과제의 시리즈로 분해하는 것을 포함할 수 있습니다.
   - **도구 선택(Tool Selection)**: 현재 계획을 기반으로, 에이전트는 사용 가능한 도구 라이브러리에서 다음 단계를 실행하기에 가장 적합한 도구를 선택하고 해당 도구를 호출하는 데 필요한 구체적인 매개변수를 결정합니다.
3. **행동(Action)**: 의사결정이 완료되면, 에이전트는 작동기를 통해 구체적인 행동을 실행합니다. 이는 일반적으로 선택된 도구(예: 코드 인터프리터나 검색 엔진 API)를 호출하여 환경에 영향을 미치고 그 상태를 변경하는 것으로 나타납니다.

행동이 루프의 끝이 아닙니다. 에이전트의 행동은 **환경**에 **상태 변화**를 일으키고, 이것이 결과 피드백으로서 새로운 **관찰**을 생성합니다. 이 새로운 관찰은 다음 루프에서 에이전트의 인식 시스템에 포착되어, 지속적인 "인식-사고-행동-관찰" 닫힌 루프를 형성합니다. 에이전트는 이 루프를 지속적으로 반복함으로써 과제를 점진적으로 진전시키고, 초기 상태에서 목표 상태를 향해 발전합니다.

### 1.2.3 에이전트의 인식과 행동

엔지니어링 실무에서 LLM이 이 루프를 효과적으로 구동하도록 하기 위해, LLM과 환경 간의 정보 교환을 규제하는 명확한 **상호작용 프로토콜(Interaction Protocol)**이 필요합니다.

많은 현대 에이전트 프레임워크에서 이 프로토콜은 각 에이전트 출력의 구조화된 정의로 구현됩니다. 에이전트의 출력은 더 이상 단일 자연어 응답이 아니라 내부 추론 과정과 최종 결정을 명시적으로 보여주는 특정 형식을 따르는 텍스트입니다.

이 구조는 일반적으로 두 가지 핵심 부분을 포함합니다:

- **Thought(사고)**: 에이전트의 내부 의사결정의 "스냅샷"입니다. 에이전트가 현재 상황을 분석하고, 이전 단계의 관찰 결과를 검토하고, 자기 반성 및 문제 분해에 참여하고, 궁극적으로 다음 구체적인 행동을 계획하는 방법을 자연어로 표현합니다.
- **Action(행동)**: 에이전트가 생각을 바탕으로 환경에 부과하기로 결정한 구체적인 작업으로, 일반적으로 함수 호출로 표현됩니다.

예를 들어, 여행을 계획하는 에이전트는 다음과 같은 형식화된 출력을 생성할 수 있습니다:

```Bash
Thought: The user wants to know the weather in Beijing. I need to call the weather query tool.
Action: get_weather("Beijing")
```

여기서 `Action` 필드는 외부 세계에 대한 명령을 구성합니다. 외부 **파서(Parser)**가 이 명령을 포착하고 해당 `get_weather` 함수를 호출합니다.

행동이 실행된 후 환경이 결과를 반환합니다. 예를 들어, `get_weather` 함수는 상세한 날씨 데이터를 포함하는 JSON 객체를 반환할 수 있습니다. 그러나 원시 기계 가독 데이터(예: JSON)는 일반적으로 LLM이 집중할 필요가 없는 중복 정보를 포함하고 있으며, 형식이 자연어 처리 습관에 맞지 않습니다.

따라서 인식 시스템의 중요한 역할은 센서처럼 이 원시 출력을 처리하고 간결하고 명확한 자연어 텍스트, 즉 관찰로 캡슐화하는 것입니다.

```Bash
Observation: Beijing's current weather is sunny, temperature 25 degrees Celsius, light breeze.
```

이 `Observation` 텍스트는 다음 루프의 주요 입력 정보로 에이전트에게 피드백되어, 새로운 `Thought`와 `Action`을 수행할 수 있게 합니다.

요약하면, Thought, Action, Observation으로 구성된 이 엄격한 루프를 통해 LLM 에이전트는 내부 언어 추론 능력을 외부 환경의 실제 정보 및 도구 작동 능력과 효과적으로 결합할 수 있습니다.

## 1.3 실습: 5분 만에 첫 번째 에이전트 구현하기

이전 절에서 에이전트의 과제 환경, 핵심 작동 메커니즘, `Thought-Action-Observation` 상호작용 패러다임을 배웠습니다. 이론적 지식도 중요하지만, 가장 좋은 학습 방법은 직접 실습입니다. 이 절에서는 몇 줄의 간단한 Python 코드로 작동하는 지능형 여행 도우미를 처음부터 구축하도록 안내합니다. 이 과정은 방금 배운 이론적 루프를 따라 에이전트가 어떻게 "생각"하고 외부 "도구"와 상호작용하는지 직관적으로 경험할 수 있게 해줍니다. 시작해 보겠습니다!

이 사례에서 목표는 단계별 과제를 처리할 수 있는 지능형 여행 도우미를 구축하는 것입니다. 해결해야 할 사용자 과제는 다음과 같이 정의됩니다: "안녕하세요, 오늘 베이징의 날씨를 확인해 주시고, 날씨에 따라 적합한 관광지를 추천해 주세요." 이 과제를 완수하기 위해 에이전트는 명확한 논리적 계획 능력을 보여줘야 합니다. 먼저 날씨 조회 도구를 호출하고, 획득한 관찰 결과를 다음 단계의 기반으로 사용해야 합니다. 다음 루프에서 관광지 추천 도구를 호출하여 최종 제안에 도달합니다.

### 1.3.1 준비 단계

Python 프로그램에서 웹 API에 액세스하려면 HTTP 라이브러리가 필요합니다. `requests`는 Python 커뮤니티에서 가장 인기 있고 사용하기 쉬운 선택입니다. `tavily-python`은 실시간 웹 검색 결과를 얻기 위한 강력한 AI 검색 API 클라이언트로, [공식 웹사이트](https://www.tavily.com/)에서 등록하여 얻을 수 있습니다. `openai`는 GPT와 같은 대규모 언어 모델 서비스를 호출하기 위해 OpenAI에서 제공하는 공식 Python SDK입니다. 먼저 다음 명령으로 설치하세요:

```bash
pip install requests tavily-python openai
```

(1) 명령 템플릿

실제 LLM을 구동하는 핵심은 **프롬프트 엔지니어링(Prompt Engineering)**에 있습니다. LLM에게 어떤 역할을 해야 하는지, 어떤 도구를 가지고 있는지, 생각과 행동을 어떻게 형식화할지 알려주는 "명령 템플릿"을 설계해야 합니다. 이것이 에이전트의 "매뉴얼"로, `system_prompt`로 LLM에 전달됩니다.

```
AGENT_SYSTEM_PROMPT = """
You are an intelligent travel assistant. Your task is to analyze user requests and use available tools to solve problems step by step.

# Available Tools:
- `get_weather(city: str)`: Query real-time weather for a specified city.
- `get_attraction(city: str, weather: str)`: Search for recommended tourist attractions based on city and weather.

# Output Format Requirements:
Each response must strictly follow this format, containing one Thought-Action pair:

Thought: [Your thinking process and next step plan]
Action: [The specific action you want to execute]

Action format must be one of the following:
1. Call a tool: function_name(arg_name="arg_value")
2. Finish task: Finish[final answer]

# Important Notes:
- Output only one Thought-Action pair each time
- Action must be on the same line, do not break lines
- When you have collected enough information to answer the user's question, you must use Action: Finish[final answer] format to end

Let's begin!
"""
```

(2) 도구 1: 실제 날씨 조회

무료 날씨 조회 서비스 `wttr.in`을 사용합니다. JSON 형식으로 지정된 도시의 날씨 데이터를 반환할 수 있습니다. 이 도구를 구현하는 코드는 다음과 같습니다:

```python
import requests

def get_weather(city: str) -> str:
    """
    Query real weather information by calling the wttr.in API.
    """
    # API endpoint, we request data in JSON format
    url = f"https://wttr.in/{city}?format=j1"

    try:
        # Make network request
        response = requests.get(url)
        # Check if response status code is 200 (success)
        response.raise_for_status()
        # Parse returned JSON data
        data = response.json()

        # Extract current weather conditions
        current_condition = data['current_condition'][0]
        weather_desc = current_condition['weatherDesc'][0]['value']
        temp_c = current_condition['temp_C']

        # Format as natural language return
        return f"{city} current weather: {weather_desc}, temperature {temp_c} degrees Celsius"

    except requests.exceptions.RequestException as e:
        # Handle network errors
        return f"Error: Network problem encountered when querying weather - {e}"
    except (KeyError, IndexError) as e:
        # Handle data parsing errors
        return f"Error: Failed to parse weather data, city name may be invalid - {e}"
```

(3) 도구 2: 관광지 검색 및 추천

도시와 날씨 조건에 따라 인터넷에서 적합한 관광지를 검색하는 새로운 도구 `search_attraction`을 정의합니다:

```python
import os
from tavily import TavilyClient

def get_attraction(city: str, weather: str) -> str:
    """
    Based on city and weather, use Tavily Search API to search and return optimized attraction recommendations.
    """
    # 1. Read API key from environment variable
    api_key = os.environ.get("TAVILY_API_KEY")
    if not api_key:
        return "Error: TAVILY_API_KEY environment variable not configured."

    # 2. Initialize Tavily client
    tavily = TavilyClient(api_key=api_key)

    # 3. Construct a precise query
    query = f"'{city}' most worthwhile tourist attractions and reasons in '{weather}' weather"

    try:
        # 4. Call API, include_answer=True will return a comprehensive answer
        response = tavily.search(query=query, search_depth="basic", include_answer=True)

        # 5. Tavily's returned results are already very clean and can be used directly
        # response['answer'] is a summary answer based on all search results
        if response.get("answer"):
            return response["answer"]

        # If there's no comprehensive answer, format raw results
        formatted_results = []
        for result in response.get("results", []):
            formatted_results.append(f"- {result['title']}: {result['content']}")

        if not formatted_results:
             return "Sorry, no relevant tourist attraction recommendations found."

        return "Based on search, found the following information for you:\n" + "\n".join(formatted_results)

    except Exception as e:
        return f"Error: Problem occurred when executing Tavily search - {e}"
```

마지막으로 모든 도구 함수를 사전에 넣어 메인 루프에서 호출하기 쉽게 합니다:

```python
# Put all tool functions into a dictionary for easy subsequent calling
available_tools = {
    "get_weather": get_weather,
    "get_attraction": get_attraction,
}
```

### 1.3.2 대규모 언어 모델 연결

현재 많은 LLM 서비스 제공업체(OpenAI, Azure, Ollama, vLLM 등의 오픈소스 모델 서비스 프레임워크 포함)는 OpenAI API와 유사한 인터페이스 사양을 따르고 있습니다. 이 표준화는 개발자에게 큰 편의를 제공합니다. 에이전트의 자율적 의사결정 능력은 LLM에서 비롯됩니다. OpenAI 인터페이스 사양과 호환되는 모든 LLM 서비스에 연결할 수 있는 범용 클라이언트 `OpenAICompatibleClient`를 구현합니다.

```python
from openai import OpenAI

class OpenAICompatibleClient:
    """
    A client for calling any LLM service compatible with the OpenAI interface.
    """
    def __init__(self, model: str, api_key: str, base_url: str):
        self.model = model
        self.client = OpenAI(api_key=api_key, base_url=base_url)

    def generate(self, prompt: str, system_prompt: str) -> str:
        """Call LLM API to generate response."""
        print("Calling large language model...")
        try:
            messages = [
                {'role': 'system', 'content': system_prompt},
                {'role': 'user', 'content': prompt}
            ]
            response = self.client.chat.completions.create(
                model=self.model,
                messages=messages,
                stream=False
            )
            answer = response.choices[0].message.content
            print("Large language model responded successfully.")
            return answer
        except Exception as e:
            print(f"Error occurred when calling LLM API: {e}")
            return "Error: Error occurred when calling language model service."
```

이 클래스를 인스턴스화하려면 세 가지 정보가 필요합니다: `API_KEY`, `BASE_URL`, `MODEL_ID`. 구체적인 값은 사용하는 서비스 제공업체(예: OpenAI 공식, Azure, 또는 Ollama와 같은 로컬 모델)에 따라 다릅니다. 아직 액세스 권한이 없다면 [환경 설정](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra07-环境配置.md)을 참조하세요.

### 1.3.3 행동 루프 실행

아래의 메인 루프는 모든 컴포넌트를 통합하고 형식화된 프롬프트를 통해 LLM이 결정을 내리도록 구동합니다.

```python
import re

# --- 1. Configure LLM client ---
# Please replace this with the corresponding credentials and address for the service you use
API_KEY = "YOUR_API_KEY"
BASE_URL = "YOUR_BASE_URL"
MODEL_ID = "YOUR_MODEL_ID"
TAVILY_API_KEY="YOUR_Tavily_KEY"
os.environ['TAVILY_API_KEY'] = "YOUR_TAVILY_API_KEY"

llm = OpenAICompatibleClient(
    model=MODEL_ID,
    api_key=API_KEY,
    base_url=BASE_URL
)

# --- 2. Initialize ---
user_prompt = "Hello, please help me check today's weather in Beijing, and then recommend a suitable tourist attraction based on the weather."
prompt_history = [f"User request: {user_prompt}"]

print(f"User input: {user_prompt}\n" + "="*40)

# --- 3. Run main loop ---
for i in range(5): # Set maximum number of loops
    print(f"--- Loop {i+1} ---\n")

    # 3.1. Build Prompt
    full_prompt = "\n".join(prompt_history)

    # 3.2. Call LLM for thinking
    llm_output = llm.generate(full_prompt, system_prompt=AGENT_SYSTEM_PROMPT)
    # Truncate extra Thought-Action pairs that the model may generate
    match = re.search(r'(Thought:.*?Action:.*?)(?=\n\s*(?:Thought:|Action:|Observation:)|\Z)', 
                    llm_output, re.DOTALL)
    if match:
        truncated = match.group(1).strip()
        if truncated != llm_output.strip():
            llm_output = truncated
            print("Truncated extra Thought-Action pairs")
    print(f"Model output:\n{llm_output}\n")
    prompt_history.append(llm_output)

    # 3.3. Parse and execute action
    action_match = re.search(r"Action: (.*)", llm_output, re.DOTALL)
    if not action_match:
        observation = "Error: No action found. Please explicitly use Action: finish(...) or other actions."
        observation_str = f"Observation: {observation}"
        print(f"{observation_str}\n" + "="*40)
        prompt_history.append(observation_str)
        continue
    action_str = action_match.group(1).strip()

    if action_str.startswith("Finish"):
        final_answer = re.match(r"Finish\[(.*)\]", action_str).group(1)
        print(f"Task completed, final answer: {final_answer}")
        break

    tool_name = re.search(r"(\w+)\(", action_str).group(1)
    args_str = re.search(r"\((.*)\)", action_str).group(1)
    kwargs = dict(re.findall(r'(\w+)="([^"]*)"', args_str))

    if tool_name in available_tools:
        observation = available_tools[tool_name](**kwargs)
    else:
        observation = f"Error: Undefined tool '{tool_name}'"

    # 3.4. Record observation results
    observation_str = f"Observation: {observation}"
    print(f"{observation_str}\n" + "="*40)
    prompt_history.append(observation_str)
```

위의 단계를 통해 실제 LLM이 구동하는 완전한 에이전트를 구축했습니다. 핵심은 "도구"와 "프롬프트 엔지니어링"의 결합으로, 이것이 현재 주류 에이전트 프레임워크(LangChain, LlamaIndex 등)의 설계 본질입니다.

### 1.3.4 실행 사례 분석

다음 출력은 성공적인 에이전트 실행 과정을 완전히 보여줍니다. 이 세 번의 루프 분석을 통해 문제를 해결하는 에이전트의 핵심 역량을 명확하게 볼 수 있습니다.

```bash
User input: Hello, please help me check today's weather in Beijing, and then recommend a suitable tourist attraction based on the weather.
========================================
--- Loop 1 ---

Calling large language model...
Large language model responded successfully.
Model output:
Thought: First need to get Beijing's weather today, then recommend tourist attractions based on the weather.
Action: get_weather(city="Beijing")

Observation: Beijing current weather: Sunny, temperature 26 degrees Celsius
========================================
--- Loop 2 ---

Calling large language model...
Large language model responded successfully.
Model output:
Thought: Now I know Beijing's weather today is sunny with moderate temperature, I can recommend a suitable tourist attraction based on this information.
Action: get_attraction(city="Beijing", weather="Sunny")

Observation: The most worthwhile tourist attractions in Beijing on sunny days are the Summer Palace for its beautiful lake views and ancient architecture. Another recommendation is the Great Wall for its spectacular scenery and historical significance.
========================================
--- Loop 3 ---

Calling large language model...
Large language model responded successfully.
Model output:
Thought: I have obtained two attraction suggestions suitable for sunny days, now I can provide a satisfactory response to the user based on this information.
Action: Finish[Today's weather in Beijing is sunny with a temperature of 26 degrees Celsius, very suitable for outdoor activities. I recommend you visit the Summer Palace to enjoy the beautiful lake views and ancient architecture, or go to the Great Wall to experience its spectacular scenery and profound historical significance. Hope you have a pleasant trip!]

Task completed, final answer: Today's weather in Beijing is sunny with a temperature of 26 degrees Celsius, very suitable for outdoor activities. I recommend you visit the Summer Palace to enjoy the beautiful lake views and ancient architecture, or go to the Great Wall to experience its spectacular scenery and profound historical significance. Hope you have a pleasant trip!
```

이 간단한 여행 도우미 사례는 `Thought-Action-Observation` 패러다임에 기반한 에이전트의 네 가지 기본 역량을 집중적으로 보여줍니다: 과제 분해, 도구 호출, 문맥 이해, 결과 합성. 에이전트는 이 루프의 지속적인 반복을 통해 모호한 사용자 의도를 일련의 구체적이고 실행 가능한 단계로 변환하고 궁극적으로 목표를 달성합니다.

## 1.4 에이전트 응용의 협력 모드

이전 절에서 에이전트를 직접 구축하여 내부 작동 루프를 깊이 이해했습니다. 그러나 더 넓은 응용 시나리오에서 우리의 역할은 점점 더 사용자이자 협력자로 변화하고 있습니다. 과제에서의 역할과 자율성 정도에 따라 협력 모드는 주로 두 가지 유형으로 나뉩니다: 하나는 워크플로우에 깊이 통합된 효율적인 도구로서, 다른 하나는 복잡한 목표를 달성하기 위해 다른 에이전트와 협력하는 자율 협력자로서입니다.

### 1.4.1 개발자 도구로서의 에이전트

이 모드에서 에이전트는 강력한 보조 도구로서 개발자의 워크플로우에 깊이 통합됩니다. 개발자의 역할을 대체하는 것이 아니라 강화하여, 지루하고 반복적인 작업을 자동화하여 개발자가 창의적인 핵심 작업에 더 집중할 수 있게 합니다. 이 인간-기계 협력 방식은 소프트웨어 개발의 효율성과 품질을 크게 향상시킵니다.

현재 시장에는 여러 우수한 AI 프로그래밍 지원 도구가 등장했습니다. 모두 개발 효율성을 향상시키지만, 구현 방법과 기능적 초점에서 차이가 있습니다:

- **GitHub Copilot**: 이 분야에서 가장 영향력 있는 제품 중 하나로, GitHub와 OpenAI가 공동으로 개발했습니다. Visual Studio Code 등 주류 편집기에 깊이 통합되어 있으며, 강력한 코드 자동 완성 기능으로 유명합니다. 개발자가 코드를 작성할 때 전체 줄 또는 전체 함수 블록에 대한 실시간 제안을 제공할 수 있습니다. 최근 몇 년 동안 Copilot Chat를 통한 대화형 프로그래밍 기능도 확장하여 개발자가 편집기 내에서 채팅을 통해 프로그래밍 문제를 해결할 수 있게 합니다.
- **Claude Code**: Anthropic이 개발한 AI 프로그래밍 도우미로, 개발자가 자연어 명령을 통해 터미널에서 코딩 작업을 효율적으로 완수할 수 있도록 설계되었습니다. 완전한 코드베이스 구조를 이해하고, 코드 편집, 테스트, 디버깅 등의 작업을 수행하며, 기능 설명에서 코드 구현까지 전체 프로세스 개발을 지원합니다. Claude Code는 또한 CI, 사전 커밋 훅, 빌드 스크립트 등 자동화 시나리오에 적합한 헤드리스 모드도 제공하여 개발자에게 강력한 명령줄 프로그래밍 경험을 제공합니다.
- **Trae**: 신흥 AI 프로그래밍 도구로, 개발자에게 지능형 코드 생성 및 최적화 서비스를 제공하는 데 초점을 맞추고 있습니다. 딥러닝 기술을 통해 코드 패턴을 분석하고 개발자에게 정밀한 코드 제안과 자동화된 리팩토링 솔루션을 제공할 수 있습니다. Trae의 특징은 경량 설계와 빠른 응답 능력으로, 특히 빈번한 반복과 빠른 프로토타이핑이 필요한 시나리오에 적합합니다.
- **Cursor**: 주로 플러그인이나 통합 기능으로 존재하는 위의 도구들과 달리, Cursor는 보다 통합된 경로를 선택했습니다—그 자체가 AI 네이티브 코드 편집기입니다. 기존 편집기에 AI 기능을 추가하는 것이 아니라, 설계 단계부터 AI 상호작용을 핵심 기능으로 만들었습니다. 최고 수준의 코드 생성 및 채팅 기능 외에도, AI가 전체 코드베이스의 문맥을 이해하게 하는 것을 강조하여 더 깊은 Q&A, 리팩토링, 디버깅을 실현합니다.

물론 여기에 나열되지 않은 다른 우수한 도구들도 많이 있지만, 모두 명확한 트렌드를 가리킵니다: AI가 전체 소프트웨어 개발 수명 주기에 깊이 통합되어 효율적인 인간-기계 협력 워크플로우를 구축함으로써 소프트웨어 엔지니어링의 효율성 경계와 개발 패러다임을 심오하게 재형성하고 있습니다.

### 1.4.2 자율 협력자로서의 에이전트

인간을 돕는 도구로서의 역할과 달리, 두 번째 상호작용 모드는 에이전트의 자동화 수준을 완전히 새로운 수준으로 끌어올립니다: 자율 협력자. 이 모드에서 우리는 더 이상 모든 행동을 AI에게 단계별로 안내하지 않고, 높은 수준의 목표를 위임합니다. 에이전트는 진정한 프로젝트 팀원처럼 독립적으로 계획하고, 추론하고, 실행하고, 반성하여 최종적으로 결과를 제공합니다. 어시스턴트에서 협력자로의 이 전환은 LLM 에이전트를 대중의 시야에 더 깊이 들어오게 했습니다. 이것은 AI와의 관계가 "명령-실행"에서 "목표-위임"으로 진화했음을 표시합니다. 에이전트는 더 이상 수동적인 도구가 아니라 능동적인 목표 추구자입니다.

현재 이 자율 협력을 달성하는 접근 방식이 번성하고 있으며, 초기 BabyAGI와 AutoGPT에서 현재의 더 성숙한 CrewAI, AutoGen, MetaGPT, LangGraph와 같은 프레임워크까지 수많은 우수한 프레임워크와 제품이 등장하여 이 분야의 빠른 발전을 공동으로 이끌고 있습니다. 구체적인 구현은 크게 다르지만, 아키텍처 패러다임은 대략 몇 가지 주류 방향으로 요약할 수 있습니다:

1. **단일 에이전트 자율 루프**: 이것은 초기의 전형적인 패러다임으로, **AgentGPT**와 같은 모델로 대표됩니다. 핵심은 개방형 고수준 목표를 완수하기 위해 "생각-계획-실행-반성" 닫힌 루프를 통해 지속적으로 자기 프롬프트하고 반복하는 범용 에이전트입니다.
2. **다중 에이전트 협력**: 이것이 현재 가장 주류적인 탐색 방향으로, 인간 팀 협력 모드를 시뮬레이션하여 복잡한 문제를 해결하는 것을 목표로 합니다. 다양한 모드로 더 세분화될 수 있습니다: **역할 놀이 대화**: **CAMEL** 프레임워크처럼, 두 에이전트("프로그래머"와 "제품 관리자" 등)에게 명확한 역할과 통신 프로토콜을 부여하여 구조화된 대화를 통해 협력적으로 과제를 완수하게 합니다. **조직된 워크플로우**: **MetaGPT** 및 **CrewAI**처럼, 명확한 분업(소프트웨어 회사나 컨설팅 그룹 등)을 가진 "가상 팀"을 시뮬레이션합니다. 각 에이전트는 사전 설정된 책임과 워크플로우(SOP)를 가지며, 계층적 또는 순차적 방식으로 협력하여 복잡한 출력(완전한 코드베이스나 연구 보고서 등)을 생성합니다. **AutoGen** 및 **AgentScope**는 더 유연한 대화 모드를 제공하여 개발자가 에이전트 간의 복잡한 상호작용 네트워크를 사용자 정의할 수 있게 합니다.
3. **고급 제어 흐름 아키텍처**: **LangGraph**와 같은 프레임워크는 에이전트에게 더 강력한 기반 엔지니어링 기반을 제공하는 데 더 초점을 맞춥니다. 에이전트의 실행 과정을 상태 그래프로 모델링하여 루프, 분기, 역추적, 인간 개입 등 복잡한 프로세스를 보다 유연하고 신뢰할 수 있게 구현할 수 있습니다.

이러한 다양한 아키텍처 패러다임은 자율 에이전트를 이론적 개념에서 더 광범위한 실용적 응용으로 공동으로 이끌어, 점점 더 복잡한 실제 세계 과제를 처리할 수 있게 합니다. 후속 장에서는 다양한 유형의 프레임워크 간의 차이와 장점도 경험할 것입니다.

### 1.4.3 워크플로우와 에이전트의 차이

에이전트가 "도구"와 "협력자"로서의 두 가지 모드를 이해한 후, 워크플로우와 에이전트의 차이점을 논의할 필요가 있습니다. 둘 다 과제 자동화를 목표로 하지만, 기반 논리, 핵심 특성, 적용 시나리오가 근본적으로 다릅니다.

간단히 말하면, **워크플로우는 AI가 단계별로 명령을 실행하게 하는 반면, 에이전트는 AI에게 목표를 자율적으로 달성하는 자유를 줍니다.**

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/1-figures/1757242319667-18.png" alt="Figure description" width="90%"/>
  <p>그림 1.6 워크플로우와 에이전트의 차이</p>
</div>

그림 1.6에 나타나 있듯이, 워크플로우는 전통적인 자동화 패러다임으로, 핵심은 **일련의 과제나 단계에 대한 사전 정의된 구조화된 편성**입니다. 본질적으로 어떤 조건에서 어떤 작업을 어떤 순서로 실행할지 지정하는 정밀하고 정적인 순서도입니다. 전형적인 사례: 회사의 경비 정산 승인 프로세스. 직원이 정산서 제출(트리거) -> 금액이 500위안 미만이면 부서장이 직접 승인 -> 금액이 500위안 이상이면 먼저 부서장 승인 후 CFO 승인 전달 -> 승인 후 재무부에 지급 알림. 전체 프로세스의 모든 단계와 모든 판단 조건이 정밀하게 사전 설정됩니다.

워크플로우와 달리, 대규모 언어 모델을 기반으로 하는 에이전트는 **자율적이고 목표 지향적인 시스템**입니다. 이들은 미리 설정된 명령을 실행할 뿐만 아니라, 환경을 어느 정도 이해하고, 추론하고, 계획을 수립하며, 최종 목표를 달성하기 위해 동적으로 행동을 취할 수 있습니다. LLM이 이 과정에서 "두뇌"의 역할을 합니다. 전형적인 예는 1.3절에서 작성한 지능형 여행 도우미입니다. 새로운 명령을 주면, 예를 들어: **"안녕하세요, 오늘 베이징의 날씨를 확인해 주시고, 날씨에 따라 적합한 관광지를 추천해 주세요."** 처리 과정은 자율성을 완전히 보여줍니다:

1. **계획 및 도구 호출:** 에이전트는 먼저 과제를 두 단계로 분해합니다: ① 날씨 조회; ② 날씨에 따른 관광지 추천. 그런 다음 자율적으로 "날씨 조회 API"를 선택하고 호출하여 "베이징"을 매개변수로 전달합니다.
2. **추론 및 의사결정:** API가 "맑음, 미풍"을 반환한다고 가정합시다. 에이전트의 LLM 두뇌는 이 정보를 바탕으로 추론합니다: "맑은 날은 야외 활동에 적합합니다." 그런 다음 이 판단에 기반하여 지식 기반이나 검색 엔진 도구를 통해 자금성, 이화원, 천단공원 등 베이징의 야외 관광지를 필터링합니다.
3. **결과 생성:** 마지막으로 에이전트는 정보를 종합하여 완전하고 인간적인 답변을 제공합니다: "오늘 베이징 날씨는 맑고 미풍이 불어 야외 활동에 매우 적합합니다. 이화원을 방문하시길 추천드립니다. 쿤밍호에서 보트를 즐기고 아름다운 황실 정원 경관을 감상할 수 있습니다."

이 과정에서 `날씨=맑음이면 이화원 추천`과 같은 하드코딩된 규칙이 없습니다. 날씨가 "비"라면 에이전트는 자율적으로 추론하고 국가박물관이나 수도박물관 등 실내 장소를 추천할 것입니다. **실시간 정보를 기반으로 동적으로 추론하고 결정을 내리는 이 능력이 에이전트의 핵심 가치입니다.**

## 1.4 장 요약

이 장에서 우리는 에이전트를 탐구하는 입문 여정을 시작했습니다. 가장 근본적인 질문에서 출발했습니다:

- **대규모 언어 모델 기반 에이전트란 무엇인가?** 먼저 정의를 명확히 하고, 현대 에이전트가 역량을 갖춘 개체임을 이해했습니다. 더 이상 사전 설정된 프로그램을 실행하는 스크립트가 아니라, 자율적으로 추론하고 도구를 사용할 수 있는 의사결정자입니다.
- **에이전트는 어떻게 작동하는가?** 에이전트-환경 상호작용의 작동 메커니즘을 깊이 탐구했습니다. 이 지속적인 닫힌 루프가 에이전트가 정보를 처리하고, 결정을 내리고, 환경에 영향을 미치고, 피드백을 바탕으로 행동을 조정하는 토대임을 배웠습니다.
- **에이전트를 어떻게 구축하는가?** 이것이 이 장의 실용적 핵심이었습니다. "지능형 여행 도우미"를 예로 들어 실제 LLM이 구동하는 완전한 에이전트를 구축했습니다.
- **에이전트의 주류 응용 패러다임은 무엇인가?** 마지막으로 더 넓은 응용 도메인으로 시야를 넓혔습니다. 두 가지 주류 에이전트 상호작용 모드를 탐구했습니다: 하나는 인간 워크플로우를 향상시키는 GitHub Copilot, Cursor로 대표되는 "개발자 도구"이고, 다른 하나는 높은 수준의 목표를 독립적으로 완수할 수 있는 CrewAI, MetaGPT, AgentScope와 같은 프레임워크로 대표되는 "자율 협력자"입니다. 또한 워크플로우와 에이전트의 차이도 설명했습니다.

이 장의 학습을 통해 에이전트에 대한 기초적인 인지 프레임워크를 구축했습니다. 그렇다면 초기 구상에서 현재까지 어떻게 단계별로 진화했을까요? 다음 장에서는 에이전트의 발전 역사를 탐구할 것입니다—기원을 추적하는 여정이 곧 시작됩니다!

## 연습 문제

> **참고**: 다음 연습 문제 중 일부는 표준 답이 없습니다. 학습자의 에이전트 시스템에 대한 비판적이고 심층적인 사고와 실제 실습 능력을 키우는 데 초점을 맞추고 있습니다.

1. 다음 네 가지 `사례`에서 **주체**가 에이전트 자격이 있는지 분석하세요. 그렇다면 어떤 유형의 에이전트에 속하는지(여러 분류 차원에서 분석 가능), 그 이유를 설명하세요:

   `사례 A`: **폰 노이만 아키텍처를 따르는 슈퍼컴퓨터**, 최대 컴퓨팅 성능이 초당 2 EFlops에 달함

   `사례 B`: **테슬라 자율주행 시스템**이 고속도로에서 주행 중 갑자기 전방에 장애물을 감지하여 밀리초 내에 제동 또는 차선 변경 결정을 내려야 함

   `사례 C`: **AlphaGo**가 인간 플레이어와 대국 중 현재 상황을 평가하고 수십 수 앞의 최적 전략을 계획해야 함

   `사례 D`: **지능형 고객 서비스 역할의 ChatGPT**가 사용자 불만을 처리하면서 주문 정보 조회, 문제 원인 분석, 솔루션 제공, 사용자 감정 달래기를 해야 함

2. "지능형 피트니스 코치"를 위한 과제 환경을 설계해야 한다고 가정합시다. 이 에이전트는:
   - 웨어러블 기기를 통해 심박수, 운동 강도 등 사용자의 생리적 데이터를 모니터링할 수 있음
   - 사용자의 피트니스 목표(지방 감량/근육 증가/지구력 향상)에 따라 훈련 계획을 동적으로 조정할 수 있음
   - 사용자 운동 중 실시간 음성 안내 및 동작 교정 제공 가능
   - 훈련 효과를 평가하고 식이 요법 권장 사항 제공 가능

   PEAS 모델을 사용하여 이 에이전트의 과제 환경을 완전히 설명하고, 이 환경이 어떤 특성을 가지는지 분석하세요(부분적으로 관측 가능, 확률적, 동적 등).

3. 전자 상거래 회사가 두 가지 접근 방식으로 판매 후 환불 요청을 처리하는 것을 고려 중입니다:

   접근 방식 A (`Workflow`): 고정 프로세스 설계, 예를 들어:

   A.1 7일 이내의 일반 제품의 경우, 금액 `< 100위안`은 자동 승인; `100-500위안`은 고객 서비스 검토; `> 500위안`은 상급자 승인 필요; 특수 제품(맞춤 제작 등)은 항상 거부

   A.2 7일을 초과한 제품은 금액에 관계없이 고객 서비스 검토 또는 상급자 승인만 가능;

   접근 방식 B (`Agent`): 환불 정책을 이해하고, 사용자 이력 행동을 분석하고, 제품 상태를 평가하며, 환불 승인 여부를 자율적으로 결정하는 에이전트 시스템 구축

   다음을 분석하세요:
   - 이 두 접근 방식의 장단점은 무엇인가요?
   - 어떤 상황에서 `Workflow`가 더 적합한가요? 언제 `Agent`가 장점을 가지나요? 이 전자 상거래 회사의 책임자라면 어떤 접근 방식을 선호하겠습니까?
   - 두 접근 방식을 결합하여 강점을 상호 보완할 수 있는 접근 방식 C가 있을까요?

4. 1.3절의 지능형 여행 도우미를 기반으로, 다음 기능을 추가하는 방법을 고려해 보세요(설계 아이디어만 설명하거나 코드 구현을 시도할 수 있습니다):

   > **힌트**: `Thought-Action-Observation` 루프를 수정하여 이러한 기능을 구현하는 방법을 생각해 보세요.

   - 에이전트가 사용자 선호도(역사 문화 관광지 좋아함, 예산 범위 등)를 기억할 수 있는 "기억" 기능 추가
   - 추천 관광지 티켓이 매진된 경우 에이전트가 자동으로 대체 옵션을 추천할 수 있음
   - 사용자가 연속으로 3번 추천을 거부하면 에이전트가 반성하고 추천 전략을 조정할 수 있음

5. 카너먼의 "시스템 1"(빠른 직관)과 "시스템 2"(느린 추론) 이론<sup>[2]</sup>은 신경-기호주의 AI에 좋은 비유를 제공합니다. 먼저 구체적인 에이전트 응용 시나리오를 구상한 다음 시나리오에서 다음을 설명하세요:

   > **힌트**: 의료 진단 도우미, 법률 상담 로봇, 금융 위험 관리 시스템 등이 일반적인 응용 시나리오입니다

   - "시스템 1"이 처리해야 할 과제는 무엇인가요?
   - "시스템 2"가 처리해야 할 과제는 무엇인가요?
   - 이 두 시스템이 최종 목표를 달성하기 위해 어떻게 협력하나요?

6. 대규모 언어 모델 기반 에이전트 시스템은 강력한 능력을 보여주지만, 여전히 많은 한계가 있습니다. 다음 질문을 분석하세요:
   - 왜 에이전트 또는 에이전트 시스템이 때로 "환각"(합리적으로 보이지만 실제로는 잘못된 정보 생성)을 일으키나요?
   - 1.3절의 사례에서 최대 루프 수를 5로 설정했습니다. 이 제한이 없다면 에이전트는 어떤 문제에 직면할 수 있나요?
   - 에이전트의 "지능" 수준을 어떻게 평가하나요? 정확도 지표만 사용하는 것으로 충분한가요?

## 참고 문헌

[1] RUSSELL S, NORVIG P. Artificial Intelligence: A Modern Approach[M]. 4th ed. London: Pearson, 2020.

[2] KAHNEMAN D. Thinking, Fast and Slow[M]. New York: Farrar, Straus and Giroux, 2011.

---

## 💬 토론 및 소통

이 장을 학습하면서 질문이 있으신가요? 다른 학습자들과 인사이트를 공유하고 싶으신가요?

**📝 GitHub Discussions 방문:**
- [💬 연습 문제 토론 및 Q&A](https://github.com/datawhalechina/Hello-Agents/discussions)
- 여기서 다음을 할 수 있습니다:
  - ✅ 연습 문제에 대한 질문하기
  - ✅ 솔루션과 아이디어 공유하기
  - ✅ 다른 학습자들과 경험 교류하기
  - ✅ 커뮤니티에서 도움과 피드백 받기

**💡 팁:** 각 페이지 하단에도 직접 토론을 위한 댓글 섹션이 있습니다!

---
