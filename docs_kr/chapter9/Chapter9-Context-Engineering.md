# 제9장 컨텍스트 엔지니어링

앞 장들에서 우리는 에이전트에 기억 시스템과 RAG를 도입했다. 그러나 에이전트가 실제 복잡한 시나리오에서 안정적으로 "생각"하고 "행동"하게 하려면, 기억과 검색만으로는 충분하지 않다 — 우리에게는 모델을 위해 적절한 "컨텍스트"를 지속적이고 체계적으로 구성하는 엔지니어링 방법론이 필요하다. 이것이 바로 이 장의 주제인 **컨텍스트 엔지니어링(Context Engineering)**이다. 이는 "모델을 한 번 호출하기 전에, 어떻게 재사용 가능하고, 측정 가능하며, 발전 가능한 방식으로 입력 컨텍스트를 조합하고 최적화할 것인가"에 집중하며, 그를 통해 정확성, 견고성, 효율성을 향상시키는 것을 목표로 한다<sup>[1][2]</sup>.

독자들이 이 장의 전체 기능을 빠르게 체험할 수 있도록, 바로 설치 가능한 Python 패키지를 제공한다. 다음 명령어로 이 장에 해당하는 버전을 설치할 수 있다:

```bash
pip install "hello-agents[all]==0.2.8"
```

이 장은 컨텍스트 엔지니어링의 핵심 개념과 실천을 소개하며, HelloAgents 프레임워크에 컨텍스트 빌더와 두 가지 부속 도구를 새롭게 추가했다:

- **ContextBuilder** (`hello_agents/context/builder.py`): 컨텍스트 빌더로, GSSC(Gather-Select-Structure-Compress) 파이프라인을 구현하고 통합된 컨텍스트 관리 인터페이스를 제공한다
- **NoteTool** (`hello_agents/tools/builtin/note_tool.py`): 구조화된 메모 도구로, 에이전트의 영속적 기억 관리를 지원한다
- **TerminalTool** (`hello_agents/tools/builtin/terminal_tool.py`): 터미널 도구로, 에이전트의 파일 시스템 조작과 즉시 컨텍스트 검색을 지원한다

이 컴포넌트들은 함께 완전한 컨텍스트 엔지니어링 솔루션을 구성하며, 장기 작업 관리와 에이전트식 검색을 구현하는 핵심으로서 이후 장들에서 상세히 소개될 것이다.

프레임워크 설치 외에도, `.env`에 LLM의 API를 설정해야 한다. 이 장의 예제는 주로 대형 언어 모델을 사용하여 컨텍스트 관리와 지능적 의사결정을 수행한다.

설정이 완료되면, 이 장의 학습 여정을 시작할 수 있다!

---

## 9.1 컨텍스트 엔지니어링이란 무엇인가

수년간 프롬프트 엔지니어링(Prompt Engineering)이 응용 AI의 핵심으로 주목받은 이후, 새로운 용어가 전면에 등장하기 시작했다: **컨텍스트 엔지니어링(Context Engineering)**. 오늘날 언어 모델로 시스템을 구축하는 것은 단순히 프롬프트 안의 문장 형식과 표현을 찾는 것이 아니라, 보다 거시적인 질문에 답하는 것이다: **어떤 컨텍스트 구성이 모델로 하여금 우리가 기대하는 행동을 산출하게 할 가능성이 가장 높은가?**

"컨텍스트"란, 대형 언어 모델(LLM)을 샘플링할 때 포함되는 토큰들의 집합을 의미한다. 이 엔지니어링 문제의 핵심은 LLM의 고유한 제약 아래에서, **이 토큰들의 효용을 최적화**하여 안정적으로 기대한 결과를 얻는 것이다. LLM을 효과적으로 다루려면 종종 "컨텍스트 안에서 생각하는 것"이 필요하다 — 즉, 어떤 호출 시점에서든, LLM이 인식할 수 있는 전체 상태를 살피고, 그 상태가 유발할 수 있는 행동을 예측하는 것이다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/9-figures/9-1.webp" alt="" width="85%"/>
  <p>그림 9.1 프롬프트 엔지니어링 vs 컨텍스트 엔지니어링</p>
</div>

이 절에서는 부상하고 있는 컨텍스트 엔지니어링을 탐구하고, **조절 가능하고 효과적인** 에이전트를 구축하기 위한 정제된 멘탈 모델을 제시한다.

**컨텍스트 엔지니어링 vs. 프롬프트 엔지니어링**

그림 9.1에서 볼 수 있듯, 현재 최전선 모델 제조사들의 관점에서 컨텍스트 엔지니어링은 프롬프트 엔지니어링의 자연스러운 진화다. 프롬프트 엔지니어링은 LLM의 지시사항을 어떻게 작성하고 구성할지에 집중하여 더 나은 결과를 얻는 것(예: 시스템 프롬프트의 작성 방식과 구조화 전략)에 관한 것이다. 반면 컨텍스트 엔지니어링은 **추론 단계에서, 어떻게 "최적의 정보 집합(토큰)"을 기획하고 유지할 것인가**에 관한 것으로, 프롬프트 자체뿐 아니라 컨텍스트 윈도우에 진입하는 모든 정보를 포함한다.

LLM 엔지니어링의 초기 단계에서는 대부분의 사용 사례(일상 채팅 제외)가 단일 턴 분류나 텍스트 생성에 대해 세밀하게 조정된 프롬프트 최적화를 필요로 했기 때문에, 프롬프트가 주요 작업이었다. 이름에서 알 수 있듯, 프롬프트 엔지니어링의 핵심은 "어떻게 효과적인 프롬프트를 작성할 것인가", 특히 시스템 프롬프트이다. 그러나 우리가 더 긴 시간 범위에서, 여러 추론 라운드를 거쳐 작동하는 더 강력한 에이전트를 공학적으로 구축하기 시작하면서, **전체 컨텍스트 상태**를 관리할 수 있는 전략이 필요해졌다 — 여기에는 시스템 지시사항, 도구, MCP(Model Context Protocol), 외부 데이터, 메시지 이력 등이 포함된다.

루프로 실행되는 에이전트는 다음 라운드의 추론에 잠재적으로 관련될 수 있는 데이터를 계속해서 생성하며, 이 정보는 **주기적으로 정제**되어야 한다. 따라서 컨텍스트 엔지니어링의 "기술과 예술"은, 끊임없이 확장되는 "후보 정보 우주"에서, **어떤 내용이 유한한 컨텍스트 윈도우에 들어가야 하는지 가려내는 것**에 있다.

---

## 9.2 왜 컨텍스트 엔지니어링이 중요한가

모델의 속도가 빨라지고 처리 가능한 데이터 규모가 커지고 있음에도 불구하고, 우리는 다음을 관찰하게 된다: LLM도 인간처럼 어느 시점부터 "집중력을 잃거나" "혼란스러워진다". 건초 더미에서 바늘 찾기(needle-in-a-haystack) 류의 벤치마크는 하나의 현상을 밝혀냈다: **컨텍스트 부패(context rot)** — 컨텍스트 윈도우의 토큰이 증가할수록, 모델이 컨텍스트에서 정확하게 정보를 회상하는 능력이 오히려 저하된다.

모델마다 성능 저하 곡선은 더 완만할 수 있지만, 이 특성은 거의 모든 모델에서 나타난다. 따라서 **컨텍스트는 유한한 자원으로 간주되어야 하며, 한계 수익 체감 현상이 있다**. 인간이 유한한 작업 기억 용량을 갖고 있는 것처럼, LLM도 "주의력 예산"을 갖고 있다. 새로운 토큰이 하나 추가될 때마다 이 예산의 일부가 소비되므로, 어떤 토큰을 LLM에 제공할지 더욱 신중하게 선별해야 한다.

이 희소성은 우연이 아니라 LLM의 아키텍처적 제약에서 비롯된다. Transformer는 각 토큰이 컨텍스트 내의 **모든** 토큰과 연관될 수 있게 하여, 이론상 $$n^2$$ 수준의 쌍방향 어텐션 관계를 형성한다. 컨텍스트 길이가 늘어날수록, 이 쌍방향 관계들에 대한 모델의 모델링 능력이 "얇아지며", 이는 자연스럽게 "컨텍스트 규모"와 "주의 집중도" 사이의 긴장을 만들어낸다. 또한, 모델의 어텐션 패턴은 훈련 데이터 분포에서 비롯된다 — 짧은 시퀀스가 일반적으로 긴 시퀀스보다 더 자주 등장하므로, 모델은 "전체 컨텍스트 의존성"에 대한 경험이 더 적고 전용 파라미터도 더 적다.

위치 인코딩 보간(position encoding interpolation) 같은 기술을 통해 모델이 추론 시 훈련 때보다 더 긴 시퀀스에 "적응"할 수 있게 할 수 있지만, 토큰 위치에 대한 정밀한 이해의 일부를 희생하게 된다. 전반적으로 이러한 요인들이 함께 만드는 것은 **성능 그래디언트**이지, "절벽식" 붕괴가 아니다: 모델은 긴 컨텍스트에서도 여전히 강력하지만, 짧은 컨텍스트에 비해 정보 검색과 장기 추론의 정확도가 다소 저하된다.

이러한 현실에 기반하여, **의식적인 컨텍스트 엔지니어링**은 견고한 에이전트를 구축하는 필수 요소가 된다.

### 9.2.1 효과적인 컨텍스트의 "해부학"

"유한한 주의력 예산"이라는 제약 아래, 훌륭한 컨텍스트 엔지니어링의 목표는: **가능한 한 적지만, 고신호 밀도의 토큰으로, 기대한 결과를 얻을 확률을 최대화하는 것**이다. 실천적으로 우리는 다음 컴포넌트들을 중심으로 공학적 구축을 권장한다:

- **시스템 프롬프트(System Prompt)**: 언어가 명확하고 직접적이며, 정보 계층은 "딱 적당한" 높이를 유지한다. 흔한 두 가지 극단적 오류:
  - 과도한 하드코딩: 프롬프트에 복잡하고 취약한 if-else 로직을 작성하면, 장기 유지보수 비용이 높고 쉽게 부서진다.
  - 지나치게 추상적: 거시적 목표와 일반화된 지침만 제시하고, 기대 출력에 대한 **구체적인 신호**가 부족하거나 잘못된 "공유 컨텍스트"를 가정한다.
  프롬프트를 섹션별로 조직화할 것을 권장한다(예: `<background_information>`, `<instructions>`, 도구 가이드, 출력 설명 등), XML/Markdown으로 구분한다. 형식에 관계없이, 추구하는 것은 **기대 행동을 완전히 그려낼 수 있는 "최소 필수 정보 집합"**이다("최소"는 "가장 짧음"을 의미하지 않는다). 먼저 최선의 모델로 최소 프롬프트에서 실행해보고, 실패 패턴에 따라 명확한 지시와 예시를 보완한다.

- **도구(Tools)**: 도구 정의는 에이전트와 정보/행동 공간 사이의 계약을 정의하며, 효율성을 촉진해야 한다: **토큰 친화적인** 정보를 반환하는 동시에, 효율적인 에이전트 행동을 장려해야 한다. 도구는:
  - 단일 책임, 낮은 상호 중복, 명확한 인터페이스 의미를 가져야 한다;
  - 오류에 대해 견고해야 한다;
  - 입력 파라미터 설명이 명확하고 모호하지 않아야 하며, 모델이 잘하는 표현과 추론 능력을 충분히 발휘해야 한다.
  흔한 실패 패턴은 "비대한 도구 세트"다: 기능 경계가 모호하여, "어느 도구를 선택할 것인가"라는 결정 자체가 불분명해진다. **인간 엔지니어도 어느 도구를 써야 할지 모른다면, 에이전트가 더 잘할 것이라고 기대하지 말라**. 신중하게 선별된 "최소 실행 가능 도구 세트(MVTS)"는 장기 상호작용의 안정성과 유지보수성을 크게 향상시킬 수 있다.

- **예시(Few-shot)**: 항상 예시를 제공할 것을 권장하지만, "모든 경계 조건"을 나열하여 프롬프트에 한꺼번에 쑤셔 넣는 것은 권장하지 않는다. **다양하고 전형적인** 예시 세트를 엄선하여, "기대 행동"을 직접 보여줘라. LLM에게 있어서, **좋은 예시는 천 마디 말보다 낫다**.

전반적인 지도 원칙은: **정보가 충분하되 간결하게**. 그림 9.2에서 볼 수 있듯, 이는 런타임의 동적 검색으로 이어진다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/9-figures/9-2.webp" alt="" width="85%"/>
  <p>그림 9.2 시스템 프롬프트 보정</p>
</div>

### 9.2.2 컨텍스트 검색과 에이전트식 검색

간결한 정의: **에이전트 = 루프 안에서 자율적으로 도구를 호출하는 LLM**. 기반 모델의 능력이 강화될수록, 에이전트의 자율성 수준을 높일 수 있다: 복잡한 문제 공간을 독립적으로 탐색하고, 오류에서 회복하는 능력이 커진다.

엔지니어링 실천은 "추론 전 일회성 검색(임베딩 검색)"에서 점차 "**적시(Just-in-time, JIT) 컨텍스트**"로 전환되고 있다. 후자는 관련 데이터를 사전에 모두 로드하는 대신, **경량 참조**(파일 경로, 스토리지 쿼리, URL 등)를 유지하고, 런타임에 도구를 통해 필요한 데이터를 동적으로 로드한다. 이를 통해 모델이 타겟팅된 쿼리를 작성하고, 필요한 결과를 캐시하며, `head`/`tail` 같은 명령으로 대용량 데이터를 분석할 수 있다 — 전체 데이터 블록을 한 번에 컨텍스트에 넣을 필요 없이. 이 인지 패턴은 인간에 더 가깝다: 우리는 모든 정보를 기계적으로 암기하지 않고, 파일 시스템, 수신함, 북마크 등의 외부 인덱스로 필요할 때 꺼내 쓴다.

저장 효율성 외에도, **참조의 메타데이터** 자체가 행동을 정제하는 데 도움이 될 수 있다: 디렉토리 계층, 명명 규칙, 타임스탬프 등이 "목적과 시효"를 암묵적으로 전달한다. 예를 들어, `tests/test_utils.py`와 `src/core/test_utils.py`의 의미적 암시는 다르다.

에이전트가 자율적으로 탐색하고 검색할 수 있도록 허용하면 **점진적 공개(progressive disclosure)**를 구현할 수 있다: 각 상호작용 단계가 새로운 컨텍스트를 생성하고, 이것이 다시 다음 단계의 결정을 인도한다 — 파일 크기는 복잡도를 암시하고, 명명은 용도를 암시하며, 타임스탬프는 관련성을 암시한다. 에이전트는 계층적으로 이해를 구축하고, 작업 기억에는 "현재 필요한 서브셋"만 유지하며, "메모 쓰기"로 보완적인 지속적 기억을 유지함으로써, 집중력을 유지하고 "크고 전체적인 것에 압도"되지 않을 수 있다.

균형을 맞춰야 할 점은: 런타임 탐색은 종종 사전 계산 검색보다 느리고, 모델이 올바른 도구와 휴리스틱을 갖도록 "확실한 의견을 가진" 엔지니어링 설계가 필요하다는 것이다. 가이드가 없으면 에이전트는 도구를 잘못 사용하거나, 막다른 길을 추구하거나, 핵심 정보를 놓쳐 컨텍스트를 낭비할 수 있다.

많은 시나리오에서 **혼합 전략**이 더 효과적이다: 속도를 보장하기 위해 소량의 "고가치" 컨텍스트를 미리 로드하고, 이후 에이전트가 필요에 따라 자율적으로 탐색을 계속하도록 허용한다. 경계 선택은 작업의 동적 특성과 시효 요구사항에 달려 있다. 엔지니어링적으로는 "프로젝트 규약 설명(README/가이드 같은)" 파일을 미리 넣어두고, 동시에 `glob`, `grep` 같은 프리미티브를 제공하여 에이전트가 구체적인 파일을 즉시 검색할 수 있게 함으로써, 오래된 인덱스와 복잡한 구문 트리의 매몰 비용을 우회할 수 있다.

### 9.2.3 장기 작업을 위한 컨텍스트 엔지니어링

장기 작업은 에이전트가 컨텍스트 윈도우를 초과하는 긴 행동 시퀀스 안에서도 일관성, 컨텍스트 일치, 목표 지향성을 유지할 것을 요구한다. 예를 들어 대규모 코드베이스 마이그레이션, 수 시간에 걸친 체계적 연구 같은 것들이다. 컨텍스트 윈도우를 무한정 늘리는 것은 "컨텍스트 오염"과 관련성 저하 문제의 근본적 치료책이 될 수 없으므로, 이러한 제약에 직접 대응하는 엔지니어링 수단이 필요하다: **압축 통합(Compaction)**, **구조화된 메모(Structured note-taking)**, **서브 에이전트 아키텍처(Sub-agent architectures)**.

- **압축 통합(Compaction)**
  - 정의: 대화가 컨텍스트 한계에 가까워지면, 이를 고충실도로 요약하고, 그 요약으로 새로운 컨텍스트 윈도우를 재시작하여 장기적 일관성을 유지한다.
  - 실천: 모델이 아키텍처적 결정, 미해결 결함, 구현 세부사항을 압축하여 보존하고, 반복적인 도구 출력과 노이즈는 버리게 한다; 새 윈도우는 압축 요약 + 최근의 고관련 아티팩트(예: "최근에 접근한 몇 개의 파일")를 가지고 시작한다.
  - 조정 권장사항: 먼저 **재현율**을 최적화하고(핵심 정보가 누락되지 않도록 확인), 그 다음 **정밀도**를 최적화한다(중복 내용 제거); "깊은 이력에서의 도구 호출과 결과"를 정리하는 것은 안전한 "가벼운 터치" 압축이다.

- **구조화된 메모(Structured note-taking)**
  - 정의: "에이전트 기억"이라고도 한다. 에이전트는 일정한 주기로 핵심 정보를 **컨텍스트 외부의 영속적 저장소**에 기록하고, 이후 단계에서 필요에 따라 불러온다.
  - 가치: 매우 낮은 컨텍스트 오버헤드로 영속적 상태와 의존성을 유지한다. 예를 들어 TODO 목록, 프로젝트 NOTES.md, 핵심 결론/의존성/블로커의 인덱스를 유지하여, 수십 번의 도구 호출과 여러 번의 컨텍스트 리셋을 거쳐도 진행 상태와 일관성을 유지한다.
  - 설명: 코딩이 아닌 시나리오에서도 동일하게 효과적이다(예: 장기 전략적 작업, 게임/시뮬레이션에서의 목표 관리와 통계 카운팅). 8장의 `MemoryTool`과 결합하면, 파일식/벡터식 외부 기억을 쉽게 구현하고 런타임에 검색할 수 있다.

- **서브 에이전트 아키텍처(Sub-agent architectures)**
  - 사상: 주 에이전트가 고수준 계획과 통합을 담당하고, 여러 전문 서브 에이전트가 "깨끗한 컨텍스트 윈도우"에서 각자 깊이 탐구하고, 도구를 호출하며, 탐색한 후, 마지막에 **응축된 요약**(일반적으로 1,000–2,000 토큰)만 반환한다.
  - 장점: 관심사 분리를 구현한다. 방대한 검색 컨텍스트는 서브 에이전트 내부에 머물고, 주 에이전트는 통합과 추론에만 집중한다; 병렬 탐색이 필요한 복잡한 연구/분석 작업에 적합하다.
  - 경험: 공개된 다중 에이전트 연구 시스템들은 이 패턴이 복잡한 연구 작업에서 단일 에이전트 기준선 대비 유의미한 우위를 보인다고 밝히고 있다.

방법 선택에는 다음의 경험 법칙을 따를 수 있다:

- **압축 통합**: 긴 대화 연속성이 필요한 작업에 적합하며, 컨텍스트의 "계주"를 강조한다.
- **구조화된 메모**: 마일스톤/단계별 결과가 있는 반복적 개발과 연구에 적합하다.
- **서브 에이전트 아키텍처**: 복잡한 연구와 분석에 적합하며, 병렬 탐색에서 이점을 얻을 수 있다.

모델 능력이 지속적으로 향상되더라도, "긴 상호작용에서 일관성과 집중력 유지"는 여전히 견고한 에이전트를 구축하는 핵심 과제다. 신중하고 체계적인 컨텍스트 엔지니어링은 오랫동안 그 핵심적 가치를 유지할 것이다.

---

## 9.3 Hello-Agents에서의 실천: ContextBuilder

이 절에서는 HelloAgents 프레임워크에서의 컨텍스트 엔지니어링 실천을 상세히 소개한다. 설계 동기, 핵심 데이터 구조, 구현 세부사항, 완전한 사례에 이르기까지, 생산급 컨텍스트 관리 시스템을 어떻게 구축하는지 단계적으로 보여줄 것이다. ContextBuilder의 설계 이념은 "단순하고 효율적"이다 — 불필요한 복잡성을 제거하고, "관련성+최신성"의 점수로 통합하여 선택하며, 에이전트의 모듈성과 유지보수성이라는 엔지니어링 지향성에 부합한다.

### 9.3.1 설계 동기와 목표

ContextBuilder를 구축하기 전에, 먼저 그 설계 목표와 핵심 가치를 명확히 해야 한다. 훌륭한 컨텍스트 관리 시스템은 다음 핵심 문제들을 해결해야 한다:

1. **통합 진입점**: "수집(Gather)-선택(Select)-구조화(Structure)-압축(Compress)"을 재사용 가능한 파이프라인으로 추상화하여, 에이전트 구현에서 반복적인 템플릿 코드를 줄인다. 이러한 통합된 인터페이스 설계 덕분에 개발자는 각 에이전트에 컨텍스트 관리 로직을 반복해서 작성할 필요가 없다.

2. **안정적인 형태**: 고정된 골격의 컨텍스트 템플릿을 출력하여, 디버깅, A/B 테스트, 평가를 용이하게 한다. 우리는 섹션별 조직화된 템플릿 구조를 채택했다:
   - `[Role & Policies]`: 에이전트의 역할 정의와 행동 원칙을 명확히 한다
   - `[Task]`: 현재 완료해야 할 구체적인 작업
   - `[State]`: 에이전트의 현재 상태와 컨텍스트 정보
   - `[Evidence]`: 외부 지식베이스에서 검색된 증거 정보
   - `[Context]`: 이력 대화와 관련 기억
   - `[Output]`: 기대하는 출력 형식과 요구사항

3. **예산 수호**: 토큰 예산 내에서 가능한 한 고가치 정보를 보존하고, 초과된 컨텍스트에 대해 폴백 압축 전략을 제공한다. 이는 정보량이 방대한 시나리오에서도 시스템이 안정적으로 운영될 수 있도록 보장한다.

4. **최소 규칙**: 출처/우선순위 등의 분류 차원을 도입하지 않아, 복잡성 증가를 방지한다. 실践에서 입증된 것처럼, 관련성과 최신성에 기반한 간단한 채점 메커니즘은 대부분의 시나리오에서 이미 충분히 효과적이다.

### 9.3.2 핵심 데이터 구조

ContextBuilder의 구현은 두 가지 핵심 데이터 구조에 의존하며, 이들은 시스템의 설정과 정보 단위를 정의한다.

**(1) ContextPacket: 후보 정보 패킷**

```python
from dataclasses import dataclass
from typing import Optional, Dict, Any
from datetime import datetime

@dataclass
class ContextPacket:
    """후보 정보 패킷

    Attributes:
        content: 정보 내용
        timestamp: 타임스탬프
        token_count: 토큰 수
        relevance_score: 관련성 점수(0.0-1.0)
        metadata: 선택적 메타데이터
    """
    content: str
    timestamp: datetime
    token_count: int
    relevance_score: float = 0.5
    metadata: Optional[Dict[str, Any]] = None

    def __post_init__(self):
        """초기화 후 처리"""
        if self.metadata is None:
            self.metadata = {}
        # 관련성 점수가 유효 범위 내에 있도록 보장
        self.relevance_score = max(0.0, min(1.0, self.relevance_score))
```

`ContextPacket`은 시스템 내 정보의 기본 단위다. 모든 후보 정보는 `ContextPacket`으로 캡슐화되며, 내용, 타임스탬프, 토큰 수, 관련성 점수 등의 핵심 속성을 포함한다. 이 통합된 데이터 구조는 이후의 선택 및 정렬 로직을 단순화한다.

**(2) ContextConfig: 설정 관리**

```python
@dataclass
class ContextConfig:
    """컨텍스트 구축 설정

    Attributes:
        max_tokens: 최대 토큰 수
        reserve_ratio: 시스템 지시사항을 위해 예약하는 비율(0.0-1.0)
        min_relevance: 최소 관련성 임계값
        enable_compression: 압축 활성화 여부
        recency_weight: 최신성 가중치(0.0-1.0)
        relevance_weight: 관련성 가중치(0.0-1.0)
    """
    max_tokens: int = 3000
    reserve_ratio: float = 0.2
    min_relevance: float = 0.1
    enable_compression: bool = True
    recency_weight: float = 0.3
    relevance_weight: float = 0.7

    def __post_init__(self):
        """설정 파라미터 검증"""
        assert 0.0 <= self.reserve_ratio <= 1.0, "reserve_ratio는 [0, 1] 범위 내여야 한다"
        assert 0.0 <= self.min_relevance <= 1.0, "min_relevance는 [0, 1] 범위 내여야 한다"
        assert abs(self.recency_weight + self.relevance_weight - 1.0) < 1e-6, \
            "recency_weight + relevance_weight는 1.0이어야 한다"
```

`ContextConfig`는 모든 설정 가능한 파라미터를 캡슐화하여 시스템 동작을 유연하게 조정할 수 있게 한다. 특히 주목할 만한 것은 `reserve_ratio` 파라미터로, 이는 시스템 지시사항 같은 핵심 정보가 항상 충분한 공간을 확보하여 다른 정보에 밀리지 않도록 보장한다.

### 9.3.3 GSSC 파이프라인 상세 설명

ContextBuilder의 핵심은 GSSC(Gather-Select-Structure-Compress) 파이프라인으로, 컨텍스트 구축 과정을 네 개의 명확한 단계로 분해한다. 각 단계의 구현 세부사항을 깊이 살펴보자.

**(1) Gather: 다중 소스 정보 수집**

첫 번째 단계는 여러 소스에서 후보 정보를 수집하는 것이다. 이 단계의 핵심은 내결함성과 유연성이다.

```python
def _gather(
    self,
    user_query: str,
    conversation_history: Optional[List[Message]] = None,
    system_instructions: Optional[str] = None,
    custom_packets: Optional[List[ContextPacket]] = None
) -> List[ContextPacket]:
    """모든 후보 정보를 수집한다

    Args:
        user_query: 사용자 쿼리
        conversation_history: 대화 이력
        system_instructions: 시스템 지시사항
        custom_packets: 사용자 정의 정보 패킷

    Returns:
        List[ContextPacket]: 후보 정보 목록
    """
    packets = []

    # 1. 시스템 지시사항 추가(최고 우선순위, 채점에 참여하지 않음)
    if system_instructions:
        packets.append(ContextPacket(
            content=system_instructions,
            timestamp=datetime.now(),
            token_count=self._count_tokens(system_instructions),
            relevance_score=1.0,  # 시스템 지시사항은 항상 보존
            metadata={"type": "system_instruction", "priority": "high"}
        ))

    # 2. 기억 시스템에서 관련 기억 검색
    if self.memory_tool:
        try:
            memory_results = self.memory_tool.run({
                "action": "search",
                "query": user_query,
                "limit": 10,
                "min_importance": 0.3
            })
            # 기억 결과를 파싱하여 ContextPacket으로 변환
            memory_packets = self._parse_memory_results(memory_results, user_query)
            packets.extend(memory_packets)
        except Exception as e:
            print(f"[WARNING] 기억 검색 실패: {e}")

    # 3. RAG 시스템에서 관련 지식 검색
    if self.rag_tool:
        try:
            rag_results = self.rag_tool.run({
                "action": "search",
                "query": user_query,
                "limit": 5,
                "min_score": 0.3
            })
            # RAG 결과를 파싱하여 ContextPacket으로 변환
            rag_packets = self._parse_rag_results(rag_results, user_query)
            packets.extend(rag_packets)
        except Exception as e:
            print(f"[WARNING] RAG 검색 실패: {e}")

    # 4. 대화 이력 추가(가장 최근 N개만 보존)
    if conversation_history:
        recent_history = conversation_history[-5:]  # 기본으로 최근 5개 보존
        for msg in recent_history:
            packets.append(ContextPacket(
                content=f"{msg.role}: {msg.content}",
                timestamp=msg.timestamp if hasattr(msg, 'timestamp') else datetime.now(),
                token_count=self._count_tokens(msg.content),
                relevance_score=0.6,  # 이력 메시지의 기본 관련성
                metadata={"type": "conversation_history", "role": msg.role}
            ))

    # 5. 사용자 정의 정보 패킷 추가
    if custom_packets:
        packets.extend(custom_packets)

    print(f"[ContextBuilder] {len(packets)}개의 후보 정보 패킷을 수집했다")
    return packets
```

이 구현은 몇 가지 중요한 설계 고려사항을 보여준다:

- **내결함성 메커니즘**: 각 외부 데이터 소스 호출이 try-except로 감싸져 있어, 단일 소스의 실패가 전체 프로세스에 영향을 미치지 않는다
- **우선순위 처리**: 시스템 지시사항은 높은 우선순위로 표시되어 항상 보존된다
- **이력 제한**: 대화 이력은 최근 몇 개만 보존하여, 이력 정보가 컨텍스트 윈도우를 점령하지 않도록 한다

**(2) Select: 지능적 정보 선택**

두 번째 단계는 관련성과 최신성에 기반하여 후보 정보를 채점하고 선택하는 것이다. 이것이 전체 파이프라인의 핵심이며, 최종 컨텍스트의 품질을 직접 결정한다.

```python
def _select(
    self,
    packets: List[ContextPacket],
    user_query: str,
    available_tokens: int
) -> List[ContextPacket]:
    """가장 관련성 높은 정보 패킷을 선택한다

    Args:
        packets: 후보 정보 패킷 목록
        user_query: 사용자 쿼리(관련성 계산에 사용)
        available_tokens: 사용 가능한 토큰 수

    Returns:
        List[ContextPacket]: 선택된 정보 패킷 목록
    """
    # 1. 시스템 지시사항과 다른 정보를 분리
    system_packets = [p for p in packets if p.metadata.get("type") == "system_instruction"]
    other_packets = [p for p in packets if p.metadata.get("type") != "system_instruction"]

    # 2. 시스템 지시사항이 차지하는 토큰 계산
    system_tokens = sum(p.token_count for p in system_packets)
    remaining_tokens = available_tokens - system_tokens

    if remaining_tokens <= 0:
        print("[WARNING] 시스템 지시사항이 모든 토큰 예산을 차지했다")
        return system_packets

    # 3. 다른 정보에 대한 종합 점수 계산
    scored_packets = []
    for packet in other_packets:
        # 관련성 점수 계산(아직 계산되지 않은 경우)
        if packet.relevance_score == 0.5:  # 기본값, 재계산 필요
            relevance = self._calculate_relevance(packet.content, user_query)
            packet.relevance_score = relevance

        # 최신성 점수 계산
        recency = self._calculate_recency(packet.timestamp)

        # 종합 점수 = 관련성 가중치 × 관련성 + 최신성 가중치 × 최신성
        combined_score = (
            self.config.relevance_weight * packet.relevance_score +
            self.config.recency_weight * recency
        )

        # 최소 관련성 임계값 미만의 정보 필터링
        if packet.relevance_score >= self.config.min_relevance:
            scored_packets.append((combined_score, packet))

    # 4. 점수 내림차순으로 정렬
    scored_packets.sort(key=lambda x: x[0], reverse=True)

    # 5. 탐욕적 선택: 점수 높은 순서대로 채우되, 토큰 한계에 도달할 때까지
    selected = system_packets.copy()
    current_tokens = system_tokens

    for score, packet in scored_packets:
        if current_tokens + packet.token_count <= available_tokens:
            selected.append(packet)
            current_tokens += packet.token_count
        else:
            # 토큰 예산 소진, 선택 중단
            break

    print(f"[ContextBuilder] {len(selected)}개의 정보 패킷을 선택했다, 총 {current_tokens} 토큰")
    return selected

def _calculate_relevance(self, content: str, query: str) -> float:
    """내용과 쿼리의 관련성을 계산한다

    간단한 키워드 중복 알고리즘을 사용한다. 프로덕션 환경에서는
    벡터 유사도 계산으로 대체할 수 있다.

    Args:
        content: 내용 텍스트
        query: 쿼리 텍스트

    Returns:
        float: 관련성 점수(0.0-1.0)
    """
    # 분절화(간단한 구현, 더 복잡한 토크나이저를 사용할 수 있다)
    content_words = set(content.lower().split())
    query_words = set(query.lower().split())

    if not query_words:
        return 0.0

    # Jaccard 유사도
    intersection = content_words & query_words
    union = content_words | query_words

    return len(intersection) / len(union) if union else 0.0

def _calculate_recency(self, timestamp: datetime) -> float:
    """시간 최신성 점수를 계산한다

    지수 감쇠 모델을 사용하며, 24시간 내에는 높은 점수를 유지하고
    이후 점차 감쇠한다.

    Args:
        timestamp: 정보의 타임스탬프

    Returns:
        float: 최신성 점수(0.0-1.0)
    """
    import math

    age_hours = (datetime.now() - timestamp).total_seconds() / 3600

    # 지수 감쇠: 24시간 내에는 높은 점수를 유지하고, 이후 점차 감쇠
    decay_factor = 0.1  # 감쇠 계수
    recency_score = math.exp(-decay_factor * age_hours / 24)

    return max(0.1, min(1.0, recency_score))  # [0.1, 1.0] 범위로 제한
```

선택 단계의 핵심 알고리즘은 몇 가지 중요한 엔지니어링 고려사항을 반영한다:

- **채점 메커니즘**: 관련성과 최신성의 가중 조합을 채택하며, 가중치는 설정 가능하다
- **탐욕적 알고리즘**: 점수 높은 순서대로 채우며, 유한한 예산 내에서 가장 가치 있는 정보를 선택한다
- **필터링 메커니즘**: `min_relevance` 파라미터로 저품질 정보를 필터링한다

**(3) Structure: 구조화 출력**

세 번째 단계는 선택된 정보를 구조화된 컨텍스트 템플릿으로 조직화하는 것이다.

```python
def _structure(self, selected_packets: List[ContextPacket], user_query: str) -> str:
    """선택된 정보 패킷을 구조화된 컨텍스트 템플릿으로 조직화한다

    Args:
        selected_packets: 선택된 정보 패킷 목록
        user_query: 사용자 쿼리

    Returns:
        str: 구조화된 컨텍스트 문자열
    """
    # 유형별 분류
    system_instructions = []
    evidence = []
    context = []

    for packet in selected_packets:
        packet_type = packet.metadata.get("type", "general")

        if packet_type == "system_instruction":
            system_instructions.append(packet.content)
        elif packet_type in ["rag_result", "knowledge"]:
            evidence.append(packet.content)
        else:
            context.append(packet.content)

    # 구조화 템플릿 구축
    sections = []

    # [Role & Policies]
    if system_instructions:
        sections.append("[Role & Policies]\n" + "\n".join(system_instructions))

    # [Task]
    sections.append(f"[Task]\n{user_query}")

    # [Evidence]
    if evidence:
        sections.append("[Evidence]\n" + "\n---\n".join(evidence))

    # [Context]
    if context:
        sections.append("[Context]\n" + "\n".join(context))

    # [Output]
    sections.append("[Output]\n위 정보에 기반하여 정확하고 근거 있는 답변을 제공하라.")

    return "\n\n".join(sections)
```

구조화 단계는 흩어진 정보 패킷을 명확한 섹션으로 조직화하며, 이 설계에는 몇 가지 장점이 있다:

- **가독성**: 명확한 섹션 분리로 인간과 모델 모두가 컨텍스트 구조를 쉽게 이해할 수 있다
- **디버그 용이성**: 문제 위치 파악이 쉬워지며, 어느 영역의 정보에 문제가 있는지 빠르게 식별할 수 있다
- **확장성**: 새로운 정보 소스를 추가할 때 새 섹션만 만들면 된다

**(4) Compress: 폴백 압축**

네 번째 단계는 초과된 컨텍스트를 압축 처리하는 것이다.

```python
def _compress(self, context: str, max_tokens: int) -> str:
    """초과된 컨텍스트를 압축한다

    Args:
        context: 원래 컨텍스트
        max_tokens: 최대 토큰 한계

    Returns:
        str: 압축된 컨텍스트
    """
    current_tokens = self._count_tokens(context)

    if current_tokens <= max_tokens:
        return context  # 압축 불필요

    print(f"[ContextBuilder] 컨텍스트 초과({current_tokens} > {max_tokens}), 압축 실행")

    # 섹션별 압축: 구조적 완전성 유지
    sections = context.split("\n\n")
    compressed_sections = []
    current_total = 0

    for section in sections:
        section_tokens = self._count_tokens(section)

        if current_total + section_tokens <= max_tokens:
            # 완전히 보존
            compressed_sections.append(section)
            current_total += section_tokens
        else:
            # 부분 보존
            remaining_tokens = max_tokens - current_total
            if remaining_tokens > 50:  # 최소 50 토큰은 보존
                # 단순 절단(프로덕션 환경에서는 LLM 요약을 사용할 수 있다)
                truncated = self._truncate_text(section, remaining_tokens)
                compressed_sections.append(truncated + "\n[... 내용이 압축되었다 ...]")
            break

    compressed_context = "\n\n".join(compressed_sections)
    final_tokens = self._count_tokens(compressed_context)
    print(f"[ContextBuilder] 압축 완료: {current_tokens} -> {final_tokens} 토큰")

    return compressed_context

def _truncate_text(self, text: str, max_tokens: int) -> str:
    """텍스트를 지정된 토큰 수로 자른다

    Args:
        text: 원래 텍스트
        max_tokens: 최대 토큰 수

    Returns:
        str: 잘린 텍스트
    """
    # 간단한 구현: 문자 비율로 추정
    # 프로덕션 환경에서는 정확한 토크나이저를 사용해야 한다
    char_per_token = len(text) / self._count_tokens(text) if self._count_tokens(text) > 0 else 4
    max_chars = int(max_tokens * char_per_token)

    return text[:max_chars]

def _count_tokens(self, text: str) -> int:
    """텍스트의 토큰 수를 추정한다

    Args:
        text: 텍스트 내용

    Returns:
        int: 토큰 수
    """
    # 간단한 추정: 중국어 1자 ≈ 1토큰, 영어 1단어 ≈ 1.3토큰
    # 프로덕션 환경에서는 실제 토크나이저를 사용해야 한다
    chinese_chars = sum(1 for ch in text if '\u4e00' <= ch <= '\u9fff')
    english_words = len([w for w in text.split() if w])

    return int(chinese_chars + english_words * 1.3)
```

압축 단계의 설계는 "구조적 완전성 유지"라는 원칙을 반영한다 — 토큰 예산이 빠듯한 상황에서도, 각 섹션의 핵심 정보를 최대한 보존하려 한다.

### 9.3.4 완전한 사용 예시

이제 완전한 예시를 통해, 실제 프로젝트에서 ContextBuilder를 어떻게 사용하는지 보여준다.

**(1) 기본 사용**

```python
from hello_agents.context import ContextBuilder, ContextConfig
from hello_agents.tools import MemoryTool, RAGTool
from hello_agents.core.message import Message
from datetime import datetime

# 1. 도구 초기화
memory_tool = MemoryTool(user_id="user123")
rag_tool = RAGTool(knowledge_base_path="./knowledge_base")

# 2. ContextBuilder 생성
config = ContextConfig(
    max_tokens=3000,
    reserve_ratio=0.2,
    min_relevance=0.2,
    enable_compression=True
)

builder = ContextBuilder(
    memory_tool=memory_tool,
    rag_tool=rag_tool,
    config=config
)

# 3. 대화 이력 준비
conversation_history = [
    Message(content="데이터 분석 도구를 개발 중이다", role="user", timestamp=datetime.now()),
    Message(content="좋습니다! 데이터 분석 도구는 대량의 데이터를 처리해야 한다. 어떤 기술 스택을 사용할 계획인가?", role="assistant", timestamp=datetime.now()),
    Message(content="Python과 Pandas를 사용할 예정이며, CSV 읽기 모듈은 이미 완성했다", role="user", timestamp=datetime.now()),
    Message(content="좋은 선택이다! Pandas는 데이터 처리에 매우 강력하다. 다음으로 데이터 정제와 변환을 고려해야 할 것이다.", role="assistant", timestamp=datetime.now()),
]

# 4. 기억 추가
memory_tool.run({
    "action": "add",
    "content": "사용자는 Python과 Pandas를 사용하는 데이터 분석 도구를 개발 중이다",
    "memory_type": "semantic",
    "importance": 0.8
})

memory_tool.run({
    "action": "add",
    "content": "CSV 읽기 모듈 개발 완료",
    "memory_type": "episodic",
    "importance": 0.7
})

# 5. 컨텍스트 구축
context = builder.build(
    user_query="Pandas의 메모리 사용량을 어떻게 최적화하는가?",
    conversation_history=conversation_history,
    system_instructions="당신은 시니어 Python 데이터 엔지니어링 컨설턴트다. 답변은: 1) 구체적이고 실행 가능한 제안을 제공해야 하며 2) 기술 원리를 설명하고 3) 코드 예시를 제공해야 한다"
)

print("=" * 80)
print("구축된 컨텍스트:")
print("=" * 80)
print(context)
print("=" * 80)
```

**(2) 실행 효과 시연**

위 코드를 실행하면 다음과 같은 구조화된 컨텍스트 출력을 볼 수 있다:

```
================================================================================
구축된 컨텍스트:
================================================================================
[Role & Policies]
당신은 시니어 Python 데이터 엔지니어링 컨설턴트다. 답변은: 1) 구체적이고 실행 가능한 제안을 제공해야 하며 2) 기술 원리를 설명하고 3) 코드 예시를 제공해야 한다

[Task]
Pandas의 메모리 사용량을 어떻게 최적화하는가?

[Evidence]
Pandas 메모리 최적화의 핵심 전략은 다음을 포함한다:
1. 적절한 데이터 타입 사용(예: object 대신 category)
2. 대용량 파일을 청크로 읽기
3. chunksize 파라미터 사용
---
데이터 타입 최적화는 메모리 사용량을 크게 줄일 수 있다. 예를 들어 int64를 int32로 다운그레이드하면 메모리를 50% 절약할 수 있다.

[Context]
user: 데이터 분석 도구를 개발 중이다
assistant: 좋습니다! 데이터 분석 도구는 대량의 데이터를 처리해야 한다. 어떤 기술 스택을 사용할 계획인가?
user: Python과 Pandas를 사용할 예정이며, CSV 읽기 모듈은 이미 완성했다
assistant: 좋은 선택이다! Pandas는 데이터 처리에 매우 강력하다. 다음으로 데이터 정제와 변환을 고려해야 할 것이다.
기억: 사용자는 Python과 Pandas를 사용하는 데이터 분석 도구를 개발 중이다
기억: CSV 읽기 모듈 개발 완료

[Output]
위 정보에 기반하여 정확하고 근거 있는 답변을 제공하라.
================================================================================
```

이 구조화된 컨텍스트는 모든 필요한 정보를 담고 있다:

- **[Role & Policies]**: AI의 역할과 답변 요구사항을 명확히 했다
- **[Task]**: 사용자의 질문을 명확하게 표현했다
- **[Evidence]**: RAG 시스템에서 검색한 관련 지식
- **[Context]**: 대화 이력과 관련 기억으로 충분한 배경 정보를 제공한다
- **[Output]**: LLM이 어떻게 답변을 조직할지 안내한다

**(3) 에이전트와 통합**

마지막으로, ContextBuilder를 에이전트에 통합하는 방법을 보여준다:

```python
from hello_agents import SimpleAgent, HelloAgentsLLM, ToolRegistry
from hello_agents.context import ContextBuilder, ContextConfig
from hello_agents.tools import MemoryTool, RAGTool

class ContextAwareAgent(SimpleAgent):
    """컨텍스트 인식 능력을 갖춘 에이전트"""

    def __init__(self, name: str, llm: HelloAgentsLLM, **kwargs):
        super().__init__(name=name, llm=llm, system_prompt=kwargs.get("system_prompt", ""))

        # 컨텍스트 빌더 초기화
        self.memory_tool = MemoryTool(user_id=kwargs.get("user_id", "default"))
        self.rag_tool = RAGTool(knowledge_base_path=kwargs.get("knowledge_base_path", "./kb"))

        self.context_builder = ContextBuilder(
            memory_tool=self.memory_tool,
            rag_tool=self.rag_tool,
            config=ContextConfig(max_tokens=4000)
        )

        self.conversation_history = []

    def run(self, user_input: str) -> str:
        """에이전트 실행, 자동으로 최적화된 컨텍스트 구축"""

        # 1. ContextBuilder로 최적화된 컨텍스트 구축
        optimized_context = self.context_builder.build(
            user_query=user_input,
            conversation_history=self.conversation_history,
            system_instructions=self.system_prompt
        )

        # 2. 최적화된 컨텍스트로 LLM 호출
        messages = [
            {"role": "system", "content": optimized_context},
            {"role": "user", "content": user_input}
        ]
        response = self.llm.invoke(messages)

        # 3. 대화 이력 업데이트
        from hello_agents.core.message import Message
        from datetime import datetime

        self.conversation_history.append(
            Message(content=user_input, role="user", timestamp=datetime.now())
        )
        self.conversation_history.append(
            Message(content=response, role="assistant", timestamp=datetime.now())
        )

        # 4. 중요한 상호작용을 기억 시스템에 기록
        self.memory_tool.run({
            "action": "add",
            "content": f"Q: {user_input}\nA: {response[:200]}...",  # 요약
            "memory_type": "episodic",
            "importance": 0.6
        })

        return response

# 사용 예시
agent = ContextAwareAgent(
    name="데이터 분석 컨설턴트",
    llm=HelloAgentsLLM(),
    system_prompt="당신은 시니어 Python 데이터 엔지니어링 컨설턴트다.",
    user_id="user123",
    knowledge_base_path="./data_science_kb"
)

response = agent.run("Pandas의 메모리 사용량을 어떻게 최적화하는가?")
print(response)
```

이러한 방식으로 ContextBuilder는 에이전트의 "컨텍스트 관리 두뇌"가 되어, 정보의 수집, 선별, 조직화를 자동으로 처리하여, 에이전트가 항상 최적의 컨텍스트에서 추론하고 생성할 수 있게 한다.

### 9.3.5 모범 사례와 최적화 권장사항

실제 ContextBuilder를 적용할 때, 다음 모범 사례들이 도움이 된다:

1. **동적 토큰 예산 조정**: 작업 복잡도에 따라 `max_tokens`를 동적으로 조정하며, 단순한 작업에는 작은 예산을, 복잡한 작업에는 예산을 늘린다.

2. **관련성 계산 최적화**: 프로덕션 환경에서는 단순한 키워드 중복을 벡터 유사도 계산으로 대체하여 검색 품질을 향상시킨다.

3. **캐싱 메커니즘**: 변하지 않는 시스템 지시사항과 지식베이스 내용에 대해 캐싱 메커니즘을 구현하여 반복 계산을 피한다.

4. **모니터링과 로깅**: 매번 컨텍스트 구축의 통계 정보(선택된 정보 수, 토큰 사용률 등)를 기록하여 이후 최적화를 용이하게 한다.

5. **A/B 테스트**: 핵심 파라미터(관련성 가중치, 최신성 가중치 등)에 대해 A/B 테스트를 통해 최적 설정을 찾는다.

---

## 9.4 NoteTool: 구조화된 메모

NoteTool은 "장기 작업"을 위한 구조화된 외부 기억 컴포넌트다. Markdown 파일을 매체로 사용하고, 헤더 부분에 YAML 프론트매터로 핵심 정보를 기록하며, 본문에는 상태, 결론, 블로커, 행동 항목 등을 기록한다. 이 설계는 인간 가독성, 버전 관리 친화성, 컨텍스트 주입 용이성을 결합하여, 장기 에이전트 구축의 중요한 도구다.

### 9.4.1 설계 이념과 응용 시나리오

구현 세부사항에 들어가기 전에, 먼저 NoteTool의 설계 이념과 전형적인 응용 시나리오를 이해해보자.

**(1) 왜 NoteTool이 필요한가?**

8장에서 우리는 강력한 기억 관리 능력을 제공하는 MemoryTool을 소개했다. 그러나 MemoryTool은 주로 **대화식 기억**에 집중한다 — 단기 작업 기억, 에피소드 기억, 의미론적 기억. 장기적으로 추적하고 구조적으로 관리해야 하는 **프로젝트식 작업**의 경우, 더 가볍고 인간 친화적인 기록 방식이 필요하다.

NoteTool이 이 공백을 채운다. 그것은 다음을 제공한다:

- **구조화된 기록**: Markdown + YAML 형식을 사용하여, 기계 파싱에도, 인간이 읽고 편집하기에도 적합하다
- **버전 친화적**: 순수 텍스트 형식으로, Git 등의 버전 관리 시스템을 자연스럽게 지원한다
- **낮은 오버헤드**: 복잡한 데이터베이스 조작이 필요 없어, 경량 상태 추적에 적합하다
- **유연한 분류**: `type`과 `tags`로 메모를 유연하게 조직하여, 다차원 검색을 지원한다

**(2) 전형적인 응용 시나리오**

NoteTool은 특히 다음 시나리오에 적합하다:

**시나리오1: 장기 프로젝트 추적**

에이전트가 대규모 코드베이스의 리팩토링 작업을 지원하고 있다고 상상해보자. 이것은 며칠 심지어 몇 주가 걸릴 수 있다. NoteTool은 다음을 기록할 수 있다:

- `task_state`: 현재 단계의 작업 상태와 진행 상황
- `conclusion`: 각 단계 종료 후의 핵심 결론
- `blocker`: 만난 문제와 블로커
- `action`: 다음 단계 행동 계획

```python
# 작업 상태 기록
notes.run({
    "action": "create",
    "title": "리팩토링 프로젝트 - 1단계",
    "content": "데이터 모델 계층 리팩토링 완료, 테스트 커버리지 85% 달성. 다음 단계는 비즈니스 로직 계층 리팩토링.",
    "note_type": "task_state",
    "tags": ["refactoring", "phase1"]
})

# 블로커 기록
notes.run({
    "action": "create",
    "title": "의존성 충돌 문제",
    "content": "일부 서드파티 라이브러리 버전 비호환성 발견, 해결 필요. 영향 범위: 비즈니스 로직 계층의 3개 모듈.",
    "note_type": "blocker",
    "tags": ["dependency", "urgent"]
})
```

**시나리오2: 연구 작업 관리**

지능형 연구 보조 에이전트가 문헌 综述를 수행할 때, NoteTool로 다음을 기록할 수 있다:

- 각 논문의 핵심 관점(`conclusion`)
- 추가 조사가 필요한 주제(`action`)
- 중요한 참고문헌(`reference`)

**시나리오3: ContextBuilder와의 협력**

각 대화 라운드 전에, 에이전트가 `search` 또는 `list` 작업으로 관련 메모를 검색하고, 컨텍스트에 주입할 수 있다:

```python
# 에이전트의 run 메서드에서
def run(self, user_input: str) -> str:
    # 1. 관련 메모 검색
    relevant_notes = self.note_tool.run({
        "action": "search",
        "query": user_input,
        "limit": 3
    })

    # 2. 메모 내용을 ContextPacket으로 변환
    note_packets = []
    for note in relevant_notes:
        note_packets.append(ContextPacket(
            content=note['content'],
            timestamp=note['updated_at'],
            token_count=self._count_tokens(note['content']),
            relevance_score=0.7,
            metadata={"type": "note", "note_type": note['type']}
        ))

    # 3. 컨텍스트 구축 시 메모 전달
    context = self.context_builder.build(
        user_query=user_input,
        custom_packets=note_packets,
        ...
    )
```

### 9.4.2 저장 형식 상세 설명

NoteTool은 Markdown + YAML의 혼합 형식을 채택했으며, 이 설계는 구조화와 가독성을 모두 겸비한다.

**(1) 메모 파일 형식**

각 메모는 독립적인 `.md` 파일이며, 형식은 다음과 같다:

```markdown
---
id: note_20250119_153000_0
title: 프로젝트 진행 - 1단계
type: task_state
tags: [refactoring, phase1, backend]
created_at: 2025-01-19T15:30:00
updated_at: 2025-01-19T15:30:00
---

# 프로젝트 진행 - 1단계

## 완료 현황

데이터 모델 계층 리팩토링 완료, 주요 변경사항 포함:

1. 엔티티 클래스 명명 규칙 통일
2. 타입 힌트 도입으로 코드 유지보수성 향상
3. 데이터베이스 쿼리 성능 최적화

## 테스트 커버리지

- 단위 테스트 커버리지: 85%
- 통합 테스트 커버리지: 70%

## 다음 단계 계획

1. 비즈니스 로직 계층 리팩토링
2. 의존성 충돌 문제 해결
3. 통합 테스트 커버리지를 85%로 향상
```

이 형식의 장점:

- **YAML 메타데이터**: 기계가 파싱 가능하며, 정확한 필드 추출과 검색을 지원한다
- **Markdown 본문**: 인간이 읽기 쉬우며, 풍부한 형식화(제목, 목록, 코드 블록 등)를 지원한다
- **파일명이 곧 ID**: 관리 단순화, 각 메모의 파일명이 곧 고유 식별자다

**(2) 인덱스 파일**

NoteTool은 `notes_index.json` 파일을 유지하여 메모를 빠르게 검색하고 관리한다:

```json
{
  "note_20250119_153000_0": {
    "id": "note_20250119_153000_0",
    "title": "프로젝트 진행 - 1단계",
    "type": "task_state",
    "tags": ["refactoring", "phase1", "backend"],
    "created_at": "2025-01-19T15:30:00",
    "updated_at": "2025-01-19T15:30:00",
    "file_path": "./notes/note_20250119_153000_0.md"
  }
}
```

이 인덱스 파일의 역할:

- **빠른 검색**: 각 파일을 열지 않아도 인덱스에서 직접 찾을 수 있다
- **메타데이터 관리**: 모든 메모의 메타데이터를 중앙에서 관리한다
- **무결성 검증**: 파일 누락이나 손상을 감지할 수 있다

### 9.4.3 핵심 작업 상세 설명

NoteTool은 메모의 완전한 생명 주기를 관리하는 7개의 핵심 작업을 제공한다.

**(1) create: 메모 생성**

```python
def _create_note(
    self,
    title: str,
    content: str,
    note_type: str = "general",
    tags: Optional[List[str]] = None
) -> str:
    """메모를 생성한다

    Args:
        title: 메모 제목
        content: 메모 내용(Markdown 형식)
        note_type: 메모 유형(task_state/conclusion/blocker/action/reference/general)
        tags: 태그 목록

    Returns:
        str: 메모 ID
    """
    from datetime import datetime

    # 1. 고유 ID 생성
    timestamp = datetime.now().strftime('%Y%m%d_%H%M%S')
    note_id = f"note_{timestamp}_{len(self.index)}"

    # 2. 메타데이터 구축
    metadata = {
        "id": note_id,
        "title": title,
        "type": note_type,
        "tags": tags or [],
        "created_at": datetime.now().isoformat(),
        "updated_at": datetime.now().isoformat()
    }

    # 3. 완전한 Markdown 파일 내용 구축
    md_content = self._build_markdown(metadata, content)

    # 4. 파일로 저장
    file_path = os.path.join(self.workspace, f"{note_id}.md")
    with open(file_path, 'w', encoding='utf-8') as f:
        f.write(md_content)

    # 5. 인덱스 업데이트
    metadata["file_path"] = file_path
    self.index[note_id] = metadata
    self._save_index()

    return note_id

def _build_markdown(self, metadata: Dict, content: str) -> str:
    """Markdown 파일 내용을 구축한다(YAML + 본문)"""
    import yaml

    # YAML 프론트매터
    yaml_header = yaml.dump(metadata, allow_unicode=True, sort_keys=False)

    # 형식 조합
    return f"---\n{yaml_header}---\n\n{content}"
```

사용 예시:

```python
from hello_agents.tools import NoteTool

notes = NoteTool(workspace="./project_notes")

note_id = notes.run({
    "action": "create",
    "title": "리팩토링 프로젝트 - 1단계",
    "content": """## 완료 현황
데이터 모델 계층 리팩토링 완료, 테스트 커버리지 85% 달성.

## 다음 단계
비즈니스 로직 계층 리팩토링""",
    "note_type": "task_state",
    "tags": ["refactoring", "phase1"]
})

print(f"✅ 메모 생성 성공, ID: {note_id}")
```

**(2) read: 메모 읽기**

```python
def _read_note(self, note_id: str) -> Dict:
    """메모 내용을 읽는다

    Args:
        note_id: 메모 ID

    Returns:
        Dict: 메타데이터와 내용을 담은 딕셔너리
    """
    if note_id not in self.index:
        raise ValueError(f"메모가 존재하지 않는다: {note_id}")

    file_path = self.index[note_id]["file_path"]

    # 파일 읽기
    with open(file_path, 'r', encoding='utf-8') as f:
        raw_content = f.read()

    # YAML 메타데이터와 Markdown 본문 파싱
    metadata, content = self._parse_markdown(raw_content)

    return {
        "metadata": metadata,
        "content": content
    }

def _parse_markdown(self, raw_content: str) -> Tuple[Dict, str]:
    """Markdown 파일을 파싱한다(YAML과 본문 분리)"""
    import yaml

    # YAML 구분자 찾기
    parts = raw_content.split('---\n', 2)

    if len(parts) >= 3:
        # YAML 프론트매터 있음
        yaml_str = parts[1]
        content = parts[2].strip()
        metadata = yaml.safe_load(yaml_str)
    else:
        # 메타데이터 없음, 전체를 본문으로 처리
        metadata = {}
        content = raw_content.strip()

    return metadata, content
```

**(3) update: 메모 업데이트**

```python
def _update_note(
    self,
    note_id: str,
    title: Optional[str] = None,
    content: Optional[str] = None,
    note_type: Optional[str] = None,
    tags: Optional[List[str]] = None
) -> str:
    """메모를 업데이트한다

    Args:
        note_id: 메모 ID
        title: 새 제목(선택)
        content: 새 내용(선택)
        note_type: 새 유형(선택)
        tags: 새 태그(선택)

    Returns:
        str: 작업 결과 메시지
    """
    if note_id not in self.index:
        raise ValueError(f"메모가 존재하지 않는다: {note_id}")

    # 1. 기존 메모 읽기
    note = self._read_note(note_id)
    metadata = note["metadata"]
    old_content = note["content"]

    # 2. 필드 업데이트
    if title:
        metadata["title"] = title
    if note_type:
        metadata["type"] = note_type
    if tags is not None:
        metadata["tags"] = tags
    if content is not None:
        old_content = content

    # 타임스탬프 업데이트
    from datetime import datetime
    metadata["updated_at"] = datetime.now().isoformat()

    # 3. 재구축 후 저장
    md_content = self._build_markdown(metadata, old_content)
    file_path = metadata["file_path"]

    with open(file_path, 'w', encoding='utf-8') as f:
        f.write(md_content)

    # 4. 인덱스 업데이트
    self.index[note_id] = metadata
    self._save_index()

    return f"✅ 메모가 업데이트됐다: {metadata['title']}"
```

**(4) search: 메모 검색**

```python
def _search_notes(
    self,
    query: str,
    limit: int = 10,
    note_type: Optional[str] = None,
    tags: Optional[List[str]] = None
) -> List[Dict]:
    """메모를 검색한다

    Args:
        query: 검색 키워드
        limit: 반환 수량 제한
        note_type: 유형별 필터링(선택)
        tags: 태그별 필터링(선택)

    Returns:
        List[Dict]: 매칭된 메모 목록
    """
    results = []
    query_lower = query.lower()

    for note_id, metadata in self.index.items():
        # 유형 필터링
        if note_type and metadata.get("type") != note_type:
            continue

        # 태그 필터링
        if tags:
            note_tags = set(metadata.get("tags", []))
            if not note_tags.intersection(tags):
                continue

        # 메모 내용 읽기
        try:
            note = self._read_note(note_id)
            content = note["content"]
            title = metadata.get("title", "")

            # 제목과 내용에서 검색
            if query_lower in title.lower() or query_lower in content.lower():
                results.append({
                    "note_id": note_id,
                    "title": title,
                    "type": metadata.get("type"),
                    "tags": metadata.get("tags", []),
                    "content": content,
                    "updated_at": metadata.get("updated_at")
                })
        except Exception as e:
            print(f"[WARNING] 메모 {note_id} 읽기 실패: {e}")
            continue

    # 업데이트 시간순으로 정렬
    results.sort(key=lambda x: x["updated_at"], reverse=True)

    return results[:limit]
```

**(5) list: 메모 목록**

```python
def _list_notes(
    self,
    note_type: Optional[str] = None,
    tags: Optional[List[str]] = None,
    limit: int = 20
) -> List[Dict]:
    """메모를 나열한다(업데이트 시간 역순)

    Args:
        note_type: 유형별 필터링(선택)
        tags: 태그별 필터링(선택)
        limit: 반환 수량 제한

    Returns:
        List[Dict]: 메모 메타데이터 목록
    """
    results = []

    for note_id, metadata in self.index.items():
        # 유형 필터링
        if note_type and metadata.get("type") != note_type:
            continue

        # 태그 필터링
        if tags:
            note_tags = set(metadata.get("tags", []))
            if not note_tags.intersection(tags):
                continue

        results.append(metadata)

    # 업데이트 시간순으로 정렬
    results.sort(key=lambda x: x.get("updated_at", ""), reverse=True)

    return results[:limit]
```

**(6) summary: 메모 요약**

```python
def _summary(self) -> Dict[str, Any]:
    """메모 요약 통계를 생성한다

    Returns:
        Dict: 통계 정보
    """
    total_count = len(self.index)

    # 유형별 통계
    type_counts = {}
    for metadata in self.index.values():
        note_type = metadata.get("type", "general")
        type_counts[note_type] = type_counts.get(note_type, 0) + 1

    # 최근 업데이트된 메모
    recent_notes = sorted(
        self.index.values(),
        key=lambda x: x.get("updated_at", ""),
        reverse=True
    )[:5]

    return {
        "total_notes": total_count,
        "type_distribution": type_counts,
        "recent_notes": [
            {
                "id": note["id"],
                "title": note.get("title", ""),
                "type": note.get("type"),
                "updated_at": note.get("updated_at")
            }
            for note in recent_notes
        ]
    }
```

**(7) delete: 메모 삭제**

```python
def _delete_note(self, note_id: str) -> str:
    """메모를 삭제한다

    Args:
        note_id: 메모 ID

    Returns:
        str: 작업 결과 메시지
    """
    if note_id not in self.index:
        raise ValueError(f"메모가 존재하지 않는다: {note_id}")

    # 1. 파일 삭제
    file_path = self.index[note_id]["file_path"]
    if os.path.exists(file_path):
        os.remove(file_path)

    # 2. 인덱스에서 제거
    title = self.index[note_id].get("title", note_id)
    del self.index[note_id]
    self._save_index()

    return f"✅ 메모가 삭제됐다: {title}"
```

### 9.4.4 ContextBuilder와의 심층 통합

NoteTool의 진정한 위력은 ContextBuilder와의 협력에 있다. 완전한 사례를 통해 이 통합을 보여주자.

**(1) 시나리오 설정**

장기 프로젝트 보조 에이전트를 구축한다고 가정하자. 이 에이전트는 다음이 필요하다:

1. 프로젝트의 단계별 진행 상황 기록
2. 해결되지 않은 문제 추적
3. 각 대화 시 관련 메모 자동 검토
4. 과거 메모를 기반으로 일관된 조언 제공

**(2) 구현 예시**

```python
from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.context import ContextBuilder, ContextConfig, ContextPacket
from hello_agents.tools import MemoryTool, RAGTool, NoteTool
from datetime import datetime

class ProjectAssistant(SimpleAgent):
    """장기 프로젝트 보조 에이전트, NoteTool과 ContextBuilder 통합"""

    def __init__(self, name: str, project_name: str, **kwargs):
        super().__init__(name=name, llm=HelloAgentsLLM(), **kwargs)

        self.project_name = project_name

        # 도구 초기화
        self.memory_tool = MemoryTool(user_id=project_name)
        self.rag_tool = RAGTool(knowledge_base_path=f"./{project_name}_kb")
        self.note_tool = NoteTool(workspace=f"./{project_name}_notes")

        # 컨텍스트 빌더 초기화
        self.context_builder = ContextBuilder(
            memory_tool=self.memory_tool,
            rag_tool=self.rag_tool,
            config=ContextConfig(max_tokens=4000)
        )

        self.conversation_history = []

    def run(self, user_input: str, note_as_action: bool = False) -> str:
        """보조 에이전트 실행, 메모 자동 통합"""

        # 1. NoteTool에서 관련 메모 검색
        relevant_notes = self._retrieve_relevant_notes(user_input)

        # 2. 메모를 ContextPacket으로 변환
        note_packets = self._notes_to_packets(relevant_notes)

        # 3. 최적화된 컨텍스트 구축
        context = self.context_builder.build(
            user_query=user_input,
            conversation_history=self.conversation_history,
            system_instructions=self._build_system_instructions(),
            custom_packets=note_packets
        )

        # 4. LLM 호출
        response = self.llm.invoke(context)

        # 5. 필요하면, 상호작용을 메모로 저장
        if note_as_action:
            self._save_as_note(user_input, response)

        # 6. 대화 이력 업데이트
        self._update_history(user_input, response)

        return response

    def _retrieve_relevant_notes(self, query: str, limit: int = 3) -> List[Dict]:
        """관련 메모 검색"""
        try:
            # blocker와 action 유형 메모 우선 검색
            blockers = self.note_tool.run({
                "action": "list",
                "note_type": "blocker",
                "limit": 2
            })

            # 일반 검색
            search_results = self.note_tool.run({
                "action": "search",
                "query": query,
                "limit": limit
            })

            # 병합 및 중복 제거
            all_notes = {note['note_id']: note for note in blockers + search_results}
            return list(all_notes.values())[:limit]

        except Exception as e:
            print(f"[WARNING] 메모 검색 실패: {e}")
            return []

    def _notes_to_packets(self, notes: List[Dict]) -> List[ContextPacket]:
        """메모를 컨텍스트 패킷으로 변환"""
        packets = []

        for note in notes:
            content = f"[메모:{note['title']}]\n{note['content']}"

            packets.append(ContextPacket(
                content=content,
                timestamp=datetime.fromisoformat(note['updated_at']),
                token_count=len(content) // 4,  # 간단한 추정
                relevance_score=0.75,  # 메모는 높은 관련성을 갖는다
                metadata={
                    "type": "note",
                    "note_type": note['type'],
                    "note_id": note['note_id']
                }
            ))

        return packets

    def _save_as_note(self, user_input: str, response: str):
        """상호작용을 메모로 저장"""
        try:
            # 어떤 유형의 메모로 저장할지 판단
            if "문제" in user_input or "블로커" in user_input:
                note_type = "blocker"
            elif "계획" in user_input or "다음 단계" in user_input:
                note_type = "action"
            else:
                note_type = "conclusion"

            self.note_tool.run({
                "action": "create",
                "title": f"{user_input[:30]}...",
                "content": f"## 질문\n{user_input}\n\n## 분석\n{response}",
                "note_type": note_type,
                "tags": [self.project_name, "auto_generated"]
            })

        except Exception as e:
            print(f"[WARNING] 메모 저장 실패: {e}")

    def _build_system_instructions(self) -> str:
        """시스템 지시사항 구축"""
        return f"""당신은 {self.project_name} 프로젝트의 장기 보조 에이전트다.

담당 업무:
1. 과거 메모를 기반으로 일관된 조언 제공
2. 프로젝트 진행과 미해결 문제 추적
3. 답변 시 관련 과거 메모를 인용
4. 구체적이고 실행 가능한 다음 단계 제안

주의:
- blocker로 표시된 문제를 우선 주목하라
- 제안의 근거를 명시하라(메모, 기억 또는 지식베이스)
- 프로젝트 전체 진행 상황에 대한 인식을 유지하라"""

    def _update_history(self, user_input: str, response: str):
        """대화 이력 업데이트"""
        from hello_agents.core.message import Message

        self.conversation_history.append(
            Message(content=user_input, role="user", timestamp=datetime.now())
        )
        self.conversation_history.append(
            Message(content=response, role="assistant", timestamp=datetime.now())
        )

        # 이력 길이 제한
        if len(self.conversation_history) > 10:
            self.conversation_history = self.conversation_history[-10:]

# 사용 예시
assistant = ProjectAssistant(
    name="프로젝트 보조 에이전트",
    project_name="data_pipeline_refactoring"
)

# 첫 번째 상호작용: 프로젝트 상태 기록
response = assistant.run(
    "데이터 모델 계층 리팩토링을 완료했고 테스트 커버리지가 85%에 달한다. 다음 단계는 비즈니스 로직 계층 리팩토링을 계획하고 있다.",
    note_as_action=True
)

# 두 번째 상호작용: 문제 제기
response = assistant.run(
    "비즈니스 로직 계층 리팩토링 중에 의존성 버전 충돌 문제가 발생했다. 어떻게 해결해야 하는가?"
)

# 메모 요약 보기
summary = assistant.note_tool.run({"action": "summary"})
print(summary)
```

**(3) 실행 효과 시연**

```bash
[ContextBuilder] 8개의 후보 정보 패킷을 수집했다
[ContextBuilder] 7개의 정보 패킷을 선택했다, 총 3500 토큰

✅ 보조 에이전트 답변:

이전에 기록하신 메모에서 이 문제를 언급하셨다는 것을 발견했다. 메모[리팩토링 프로젝트 - 1단계]에 따르면, 현재 테스트 커버리지가 85%에 달해 좋은 기반을 갖추고 있다.

의존성 버전 충돌 문제에 대해 다음을 제안한다:

1. **가상 환경 격리 사용**: 비즈니스 로직 계층을 위한 독립적인 가상 환경을 만들어 다른 모듈과의 의존성 충돌을 방지한다
2. **버전 고정**: requirements.txt에 모든 의존성의 정확한 버전을 명시한다
3. **pipdeptree 사용**: 의존성 트리를 분석하여 충돌의 근원을 찾는다

이 문제를 blocker로 표시하고 우선적으로 해결할 것을 권장한다.

[근거 출처: 메모 note_20250119_153000_0, 프로젝트 지식베이스]

---

📋 메모 요약:
{
  "total_notes": 2,
  "type_distribution": {
    "action": 1,
    "blocker": 1
  },
  "recent_notes": [...]
}
```

### 9.4.5 모범 사례

NoteTool을 실제 사용할 때, 다음 모범 사례가 더 강력한 장기 에이전트 구축에 도움이 된다:

1. **합리적인 메모 분류**:
   - `task_state`: 단계별 진행과 상태 기록
   - `conclusion`: 중요한 결론과 발견 기록
   - `blocker`: 블로킹 문제 기록, 최고 우선순위
   - `action`: 다음 단계 행동 계획 기록
   - `reference`: 중요한 참고 자료 기록

2. **정기적인 정리와 아카이브**:
   - 해결된 blocker는 conclusion으로 업데이트
   - 구식이 된 action은 적시에 삭제하거나 업데이트
   - tags로 버전 관리(`["v1.0", "completed"]` 등)

3. **ContextBuilder와의 협력**:
   - 각 대화 라운드 전 관련 메모 검색
   - 메모 유형에 따라 다른 관련성 점수 설정(blocker > action > conclusion)
   - 메모 수 제한으로 컨텍스트 과부하 방지

4. **인간-기계 협력**:
   - 메모는 인간이 읽을 수 있는 Markdown 형식으로, 수동 편집을 지원한다
   - Git으로 버전 관리하여 메모의 변화를 추적한다
   - 핵심 단계에서 에이전트가 생성한 메모를 인간이 검토한다

5. **자동화 워크플로**:
   - 정기적으로 메모 요약 보고서 생성
   - 메모 내용을 기반으로 프로젝트 진행 문서 자동 생성
   - 메모 내용을 다른 시스템(Notion, Confluence 등)과 동기화

---

## 9.5 TerminalTool: 즉시 파일 시스템 접근

앞 절들에서 우리는 MemoryTool과 RAGTool을 소개했으며, 각각 대화 기억과 지식 검색 능력을 제공한다. 그러나 많은 실제 시나리오에서 에이전트는 **즉시적인 파일 시스템 접근과 탐색**이 필요하다 — 로그 파일 보기, 코드베이스 구조 분석, 설정 파일 검색 등. 이것이 바로 TerminalTool의 역할이다.

TerminalTool은 에이전트에게 **안전한 명령줄 실행 능력**을 제공하며, 일반적인 파일 시스템과 텍스트 처리 명령을 지원하는 동시에 다중 계층 보안 메커니즘을 통해 시스템 안전을 보장한다. 이 설계는 9.2.2절에서 언급한 "즉시(Just-in-time, JIT) 컨텍스트" 이념을 구현한다 — 에이전트는 모든 파일을 미리 로드할 필요 없이, 필요에 따라 탐색하고 검색한다.

### 9.5.1 설계 이념과 보안 메커니즘

**(1) 왜 TerminalTool이 필요한가?**

장기 에이전트를 구축할 때, 우리는 종종 다음 시나리오를 만난다:

**시나리오1: 코드베이스 탐색**

개발 보조 에이전트가 대규모 코드베이스의 구조를 이해하는 데 도움을 줘야 한다:

```python
# 전통적인 방식: 모든 파일을 미리 인덱싱(비용 높음, 구식이 될 수 있음)
rag_tool.add_document("./project/**/*.py")  # 시간 소요, 대용량 저장 공간 차지

# TerminalTool 방식: 즉시 탐색
terminal.run({"command": "find . -name '*.py' -type f"})  # 빠르고, 실시간
terminal.run({"command": "grep -r 'class UserService' ."})  # 정확한 위치 파악
terminal.run({"command": "head -n 50 src/services/user.py"})  # 필요 시 보기
```

**시나리오2: 로그 파일 분석**

운영 보조 에이전트가 애플리케이션 로그를 분석해야 한다:

```python
# 로그 파일 크기 확인
terminal.run({"command": "ls -lh /var/log/app.log"})

# 최신 오류 로그 보기
terminal.run({"command": "tail -n 100 /var/log/app.log | grep ERROR"})

# 오류 유형 분포 통계
terminal.run({"command": "grep ERROR /var/log/app.log | cut -d':' -f3 | sort | uniq -c"})
```

**시나리오3: 데이터 파일 미리보기**

데이터 분석 보조 에이전트가 데이터 파일의 구조를 빠르게 파악해야 한다:

```python
# CSV 파일의 첫 몇 행 보기
terminal.run({"command": "head -n 5 data/sales.csv"})

# 행 수 통계
terminal.run({"command": "wc -l data/*.csv"})

# 열 이름 보기
terminal.run({"command": "head -n 1 data/sales.csv | tr ',' '\n'"})
```

이 시나리오들의 공통점은: **실시간, 경량화된 파일 시스템 접근이 필요하며, 사전 인덱싱과 벡터화가 필요 없다**는 것이다. TerminalTool은 이러한 "탐색식" 워크플로를 위해 설계됐다.

**(2) 보안 메커니즘 상세 설명**

에이전트가 명령을 실행하도록 허용하는 것은 강력하지만 위험한 능력이다. TerminalTool은 다중 계층 보안 메커니즘으로 시스템 안전을 보장한다:

**첫 번째 계층: 명령 화이트리스트**

안전한 읽기 전용 명령만 허용하고, 시스템을 수정할 수 있는 어떤 작업도 완전히 금지한다:

```python
ALLOWED_COMMANDS = {
    # 파일 목록과 정보
    'ls', 'dir', 'tree',
    # 파일 내용 보기
    'cat', 'head', 'tail', 'less', 'more',
    # 파일 검색
    'find', 'grep', 'egrep', 'fgrep',
    # 텍스트 처리
    'wc', 'sort', 'uniq', 'cut', 'awk', 'sed',
    # 디렉토리 조작
    'pwd', 'cd',
    # 파일 정보
    'file', 'stat', 'du', 'df',
    # 기타
    'echo', 'which', 'whereis',
}
```

에이전트가 화이트리스트 외의 명령을 실행하려 하면, 즉시 거부된다:

```python
terminal.run({"command": "rm -rf /"})
# ❌ 허용되지 않는 명령: rm
# 허용되는 명령: cat, cd, cut, dir, du, ...
```

**두 번째 계층: 작업 디렉토리 제한(샌드박스)**

TerminalTool은 지정된 작업 디렉토리와 그 하위 디렉토리만 접근할 수 있으며, 시스템의 다른 부분에는 접근할 수 없다:

```python
# 초기화 시 작업 디렉토리 지정
terminal = TerminalTool(workspace="./project")

# 허용: 작업 디렉토리 내 파일 접근
terminal.run({"command": "cat ./src/main.py"})  # ✅

# 금지: 작업 디렉토리 외 파일 접근
terminal.run({"command": "cat /etc/passwd"})  # ❌ 작업 디렉토리 외 경로 접근 불허

# 금지: ..로 탈출
terminal.run({"command": "cd ../../../etc"})  # ❌ 작업 디렉토리 외 경로 접근 불허
```

이 샌드박스 메커니즘은 에이전트의 행동이 비정상적이어도 시스템의 다른 부분에 영향을 미칠 수 없도록 보장한다.

**세 번째 계층: 타임아웃 제어**

각 명령에는 실행 시간 제한이 있어, 무한 루프나 리소스 고갈을 방지한다:

```python
terminal = TerminalTool(
    workspace="./project",
    timeout=30  # 30초 타임아웃
)

# 명령 실행이 30초를 초과하면
terminal.run({"command": "find / -name '*.log'"})
# ❌ 명령 실행 타임아웃(30초 초과)
```

**네 번째 계층: 출력 크기 제한**

명령 출력의 크기를 제한하여 메모리 오버플로우를 방지한다:

```python
terminal = TerminalTool(
    workspace="./project",
    max_output_size=10 * 1024 * 1024  # 10MB
)

# 출력이 10MB를 초과하면
terminal.run({"command": "cat huge_file.log"})
# ... (첫 10MB 내용) ...
# ⚠️ 출력이 잘렸다(10485760바이트 초과)
```

이 네 계층의 보안 메커니즘을 통해, TerminalTool은 강력한 능력을 제공하면서도 시스템 안전을 최대한 보장한다.

### 9.5.2 핵심 기능 상세 설명

TerminalTool의 구현은 두 가지 핵심 기능에 집중한다: 명령 실행과 디렉토리 탐색.

**(1) 명령 실행**

핵심 `_execute_command` 메서드가 실제 명령을 실행한다:

```python
def _execute_command(self, command: str) -> str:
    """명령을 실행한다"""
    try:
        # 현재 디렉토리에서 명령 실행
        result = subprocess.run(
            command,
            shell=True,
            cwd=str(self.current_dir),  # 현재 작업 디렉토리에서 실행
            capture_output=True,
            text=True,
            timeout=self.timeout,
            env=os.environ.copy()
        )

        # 표준 출력과 표준 오류 병합
        output = result.stdout
        if result.stderr:
            output += f"\n[stderr]\n{result.stderr}"

        # 출력 크기 확인
        if len(output) > self.max_output_size:
            output = output[:self.max_output_size]
            output += f"\n\n⚠️ 출력이 잘렸다({self.max_output_size}바이트 초과)"

        # 반환 코드 정보 추가
        if result.returncode != 0:
            output = f"⚠️ 명령 반환 코드: {result.returncode}\n\n{output}"

        return output if output else "✅ 명령 실행 성공(출력 없음)"

    except subprocess.TimeoutExpired:
        return f"❌ 명령 실행 타임아웃({self.timeout}초 초과)"
    except Exception as e:
        return f"❌ 명령 실행 실패: {e}"
```

이 구현의 핵심 포인트:

- **현재 디렉토리 인식**: `cwd` 파라미터로 올바른 디렉토리에서 명령을 실행한다
- **오류 처리**: 표준 오류를 캡처하고 병합하여 완전한 진단 정보를 제공한다
- **반환 코드 확인**: 0이 아닌 반환 코드는 경고로 표시된다
- **내결함성 설계**: 타임아웃과 예외 모두 적절히 처리되어, 에이전트 붕괴를 일으키지 않는다

**(2) 디렉토리 탐색**

`cd` 명령의 특별 처리로 에이전트가 파일 시스템에서 탐색할 수 있다:

```python
def _handle_cd(self, parts: List[str]) -> str:
    """cd 명령을 처리한다"""
    if not self.allow_cd:
        return "❌ cd 명령이 비활성화됐다"

    if len(parts) < 2:
        # cd 파라미터 없음, 현재 디렉토리 반환
        return f"현재 디렉토리: {self.current_dir}"

    target_dir = parts[1]

    # 상대 경로 처리
    if target_dir == "..":
        new_dir = self.current_dir.parent
    elif target_dir == ".":
        new_dir = self.current_dir
    elif target_dir == "~":
        new_dir = self.workspace
    else:
        new_dir = (self.current_dir / target_dir).resolve()

    # 작업 디렉토리 내에 있는지 확인
    try:
        new_dir.relative_to(self.workspace)
    except ValueError:
        return f"❌ 작업 디렉토리 외 경로 접근 불허: {new_dir}"

    # 디렉토리 존재 확인
    if not new_dir.exists():
        return f"❌ 디렉토리가 존재하지 않는다: {new_dir}"

    if not new_dir.is_dir():
        return f"❌ 디렉토리가 아니다: {new_dir}"

    # 현재 디렉토리 업데이트
    self.current_dir = new_dir
    return f"✅ 디렉토리 전환: {self.current_dir}"
```

이 설계는 에이전트가 다단계 파일 시스템 탐색을 수행할 수 있게 한다:

```python
# 1단계: 프로젝트 구조 보기
terminal.run({"command": "ls -la"})

# 2단계: 소스 코드 디렉토리로 진입
terminal.run({"command": "cd src"})

# 3단계: 특정 파일 검색
terminal.run({"command": "find . -name '*service*.py'"})

# 4단계: 파일 내용 보기
terminal.run({"command": "cat user_service.py"})
```

### 9.5.3 전형적인 사용 패턴

TerminalTool은 여러 일반적인 파일 시스템 조작 패턴을 지원한다.

**(1) 탐색식 탐색**

에이전트가 인간 개발자처럼 코드베이스를 점진적으로 탐색할 수 있다:

```python
from hello_agents.tools import TerminalTool

terminal = TerminalTool(workspace="./my_project")

# 1단계: 프로젝트 루트 디렉토리 보기
print(terminal.run({"command": "ls -la"}))

# 2단계: 소스 코드 디렉토리 구조 보기
terminal.run({"command": "cd src"})
print(terminal.run({"command": "tree"}))

# 3단계: 특정 패턴 검색
print(terminal.run({"command": "grep -r 'def process' ."}))
```

**(2) 데이터 파일 분석**

데이터 파일의 구조와 내용을 빠르게 파악한다:

```python
terminal = TerminalTool(workspace="./data")

# CSV 파일의 첫 몇 행 보기
print(terminal.run({"command": "head -n 5 sales_2024.csv"}))

# 총 행 수 통계
print(terminal.run({"command": "wc -l *.csv"}))

# 제품 카테고리 추출 및 통계
print(terminal.run({"command": "tail -n +2 sales_2024.csv | cut -d',' -f2 | sort | uniq -c"}))
```

**(3) 로그 파일 분석**

애플리케이션 로그를 실시간으로 분석하여 문제를 빠르게 찾는다:

```python
terminal = TerminalTool(workspace="/var/log")

# 최신 오류 로그 보기
print(terminal.run({"command": "tail -n 50 app.log | grep ERROR"}))

# 오류 유형 분포 통계
print(terminal.run({"command": "grep ERROR app.log | awk '{print $4}' | sort | uniq -c | sort -rn"}))

# 특정 시간대 로그 찾기
print(terminal.run({"command": "grep '2024-01-19 15:' app.log | tail -n 20"}))
```

**(4) 코드베이스 분석**

코드 리뷰와 이해를 지원한다:

```python
terminal = TerminalTool(workspace="./codebase")

# 코드 행 수 통계
print(terminal.run({"command": "find . -name '*.py' -exec wc -l {} + | tail -n 1"}))

# 모든 TODO 주석 찾기
print(terminal.run({"command": "grep -rn 'TODO' --include='*.py'"}))

# 특정 함수 정의 찾기
print(terminal.run({"command": "grep -rn 'def process_data' --include='*.py'"}))

# 함수 구현 보기
print(terminal.run({"command": "sed -n '/def process_data/,/^def /p' src/processor.py | head -n -1"}))
```

### 9.5.4 다른 도구와의 협력

TerminalTool의 진정한 위력은 MemoryTool, NoteTool, ContextBuilder와의 협력에 있다.

**(1) MemoryTool과 협력**

TerminalTool이 발견한 정보를 기억 시스템에 저장할 수 있다:

```python
# TerminalTool로 프로젝트 구조 발견
structure = terminal.run({"command": "tree -L 2 src"})

# 의미론적 기억에 저장
memory_tool.run({
    "action": "add",
    "content": f"프로젝트 구조:\n{structure}",
    "memory_type": "semantic",
    "importance": 0.8,
    "metadata": {"type": "project_structure"}
})
```

**(2) NoteTool과 협력**

중요한 발견을 구조화된 메모로 기록할 수 있다:

```python
# 성능 병목 발견
log_analysis = terminal.run({"command": "grep 'slow query' app.log | tail -n 10"})

# blocker 메모로 기록
note_tool.run({
    "action": "create",
    "title": "데이터베이스 느린 쿼리 문제",
    "content": f"## 문제 설명\n여러 느린 쿼리 발견, 시스템 성능에 영향\n\n## 로그 분석\n```\n{log_analysis}\n```\n\n## 다음 단계\n1. 느린 쿼리 SQL 분석\n2. 인덱스 추가\n3. 쿼리 로직 최적화",
    "note_type": "blocker",
    "tags": ["performance", "database"]
})
```

**(3) ContextBuilder와 협력**

TerminalTool의 출력을 컨텍스트의 일부로 사용할 수 있다:

```python
# 코드베이스 탐색
code_structure = terminal.run({"command": "ls -R src"})
recent_changes = terminal.run({"command": "git log --oneline -10"})

# ContextPacket으로 변환
from hello_agents.context import ContextPacket
from datetime import datetime

packets = [
    ContextPacket(
        content=f"코드베이스 구조:\n{code_structure}",
        timestamp=datetime.now(),
        token_count=len(code_structure) // 4,
        relevance_score=0.7,
        metadata={"type": "code_structure", "source": "terminal"}
    ),
    ContextPacket(
        content=f"최근 커밋:\n{recent_changes}",
        timestamp=datetime.now(),
        token_count=len(recent_changes) // 4,
        relevance_score=0.8,
        metadata={"type": "git_history", "source": "terminal"}
    )
]

# 컨텍스트 구축 시 이 정보를 포함
context = context_builder.build(
    user_query="사용자 서비스 모듈을 어떻게 리팩토링하는가?",
    custom_packets=packets
)
```

---

## 9.6 장기 에이전트 실전: 코드베이스 유지보수 보조 에이전트

이제 ContextBuilder, NoteTool, TerminalTool을 통합하여 완전한 장기 에이전트 — **코드베이스 유지보수 보조 에이전트**를 구축해보자. 이 보조 에이전트는 다음을 할 수 있다:

1. 코드베이스 구조 탐색 및 이해
2. 발견된 문제와 개선점 기록
3. 장기 리팩토링 작업 추적
4. 컨텍스트 윈도우 제한 내에서 일관성 유지

### 9.6.1 시나리오 설정과 요구사항 분석

**비즈니스 시나리오**

중형 Python 웹 애플리케이션을 유지보수하고 있다고 가정하자. 이 코드베이스는 약 50개의 Python 파일을 포함하고, Flask 프레임워크로 구축됐으며, 데이터 모델, 비즈니스 로직, API 인터페이스 등 여러 모듈을 포함하며, 점차 해소해야 할 기술적 부채도 존재한다. 이런 상황에서 코드베이스를 탐색하고, 프로젝트 구조, 의존성, 코드 스타일을 이해하며, 코드 중복, 높은 복잡도, 테스트 부족 같은 문제를 식별하고, TODO 항목, 완료된 작업, 만난 블로커를 기록하며, 과거 컨텍스트를 기반으로 일관된 리팩토링 조언을 제공하는 지능형 보조 에이전트가 필요하다.

**도전과 해결책**

이 시나리오는 몇 가지 전형적인 장기 작업의 도전에 직면한다. 먼저 정보량이 컨텍스트 윈도우를 초과하는 문제가 있다 — 전체 코드베이스는 수만 줄의 코드를 포함할 수 있어 한 번에 컨텍스트 윈도우에 넣을 수 없으며, TerminalTool을 사용한 즉시적이고 필요에 따른 코드 탐색으로 이 문제를 해결한다. 다음으로 세션 간 상태 관리 도전이 있다 — 리팩토링 작업은 며칠이 걸릴 수 있어 여러 세션에 걸쳐 진행 상태를 유지해야 하며, NoteTool로 단계별 진행, 할 일, 핵심 결정을 기록하여 대응한다. 마지막으로 컨텍스트 품질과 관련성 문제가 있다 — 각 대화에서 관련 과거 정보를 검토해야 하지만 무관한 정보에 압도되어서는 안 되며, ContextBuilder로 컨텍스트를 지능적으로 선별하고 조직화하여 높은 신호 밀도를 보장한다.

### 9.6.2 시스템 아키텍처 설계

코드베이스 유지보수 보조 에이전트는 3층 아키텍처를 채택한다(그림 9.3 참조):

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/9-figures/9-3.png" alt="" width="85%"/>
  <p>그림 9.3 코드베이스 유지보수 보조 에이전트 3층 아키텍처</p>
</div>

### 9.6.3 핵심 구현

이제 이 시스템의 핵심 클래스를 구현해보자:

```python
from typing import Dict, Any, List, Optional
from datetime import datetime
import json

from hello_agents import SimpleAgent, HelloAgentsLLM
from hello_agents.context import ContextBuilder, ContextConfig, ContextPacket
from hello_agents.tools import MemoryTool, NoteTool, TerminalTool
from hello_agents.core.message import Message


class CodebaseMaintainer:
    """코드베이스 유지보수 보조 에이전트 - 장기 에이전트 예시

    ContextBuilder + NoteTool + TerminalTool + MemoryTool 통합
    세션 간 코드베이스 유지보수 작업 관리 구현
    """

    def __init__(
        self,
        project_name: str,
        codebase_path: str,
        llm: Optional[HelloAgentsLLM] = None
    ):
        self.project_name = project_name
        self.codebase_path = codebase_path
        self.session_id = f"session_{datetime.now().strftime('%Y%m%d_%H%M%S')}"

        # LLM 초기화
        self.llm = llm or HelloAgentsLLM()

        # 도구 초기화
        self.memory_tool = MemoryTool(user_id=project_name)
        self.note_tool = NoteTool(workspace=f"./{project_name}_notes")
        self.terminal_tool = TerminalTool(workspace=codebase_path, timeout=60)

        # 컨텍스트 빌더 초기화
        self.context_builder = ContextBuilder(
            memory_tool=self.memory_tool,
            rag_tool=None,  # 이 예시에서는 RAG를 사용하지 않는다
            config=ContextConfig(
                max_tokens=4000,
                reserve_ratio=0.15,
                min_relevance=0.2,
                enable_compression=True
            )
        )

        # 대화 이력
        self.conversation_history: List[Message] = []

        # 통계 정보
        self.stats = {
            "session_start": datetime.now(),
            "commands_executed": 0,
            "notes_created": 0,
            "issues_found": 0
        }

        print(f"✅ 코드베이스 유지보수 보조 에이전트 초기화 완료: {project_name}")
        print(f"📁 작업 디렉토리: {codebase_path}")
        print(f"🆔 세션 ID: {self.session_id}")

    def run(self, user_input: str, mode: str = "auto") -> str:
        """보조 에이전트 실행

        Args:
            user_input: 사용자 입력
            mode: 실행 모드
                - "auto": 도구 사용 여부 자동 결정
                - "explore": 코드 탐색 중심
                - "analyze": 문제 분석 중심
                - "plan": 작업 계획 중심

        Returns:
            str: 보조 에이전트의 답변
        """
        print(f"\n{'='*80}")
        print(f"👤 사용자: {user_input}")
        print(f"{'='*80}\n")

        # 1단계: 모드에 따라 전처리 실행
        pre_context = self._preprocess_by_mode(user_input, mode)

        # 2단계: 관련 메모 검색
        relevant_notes = self._retrieve_relevant_notes(user_input)
        note_packets = self._notes_to_packets(relevant_notes)

        # 3단계: 최적화된 컨텍스트 구축
        context = self.context_builder.build(
            user_query=user_input,
            conversation_history=self.conversation_history,
            system_instructions=self._build_system_instructions(mode),
            custom_packets=note_packets + pre_context
        )

        # 4단계: LLM 호출
        print("🤖 생각 중...")
        response = self.llm.invoke(context)

        # 5단계: 후처리
        self._postprocess_response(user_input, response)

        # 6단계: 대화 이력 업데이트
        self._update_history(user_input, response)

        print(f"\n🤖 보조 에이전트: {response}\n")
        print(f"{'='*80}\n")

        return response

    def _preprocess_by_mode(
        self,
        user_input: str,
        mode: str
    ) -> List[ContextPacket]:
        """모드에 따라 전처리를 실행하고 관련 정보를 수집한다"""
        packets = []

        if mode == "explore" or mode == "auto":
            # 탐색 모드: 자동으로 프로젝트 구조 보기
            print("🔍 코드베이스 구조 탐색 중...")

            structure = self.terminal_tool.run({"command": "find . -type f -name '*.py' | head -n 20"})
            self.stats["commands_executed"] += 1

            packets.append(ContextPacket(
                content=f"[코드베이스 구조]\n{structure}",
                timestamp=datetime.now(),
                token_count=len(structure) // 4,
                relevance_score=0.6,
                metadata={"type": "code_structure", "source": "terminal"}
            ))

        if mode == "analyze":
            # 분석 모드: 코드 복잡도와 문제 점검
            print("📊 코드 품질 분석 중...")

            # 코드 행 수 통계
            loc = self.terminal_tool.run({"command": "find . -name '*.py' -exec wc -l {} + | tail -n 1"})

            # TODO와 FIXME 찾기
            todos = self.terminal_tool.run({"command": "grep -rn 'TODO\\|FIXME' --include='*.py' | head -n 10"})

            self.stats["commands_executed"] += 2

            packets.append(ContextPacket(
                content=f"[코드 통계]\n{loc}\n\n[할 일 항목]\n{todos}",
                timestamp=datetime.now(),
                token_count=(len(loc) + len(todos)) // 4,
                relevance_score=0.7,
                metadata={"type": "code_analysis", "source": "terminal"}
            ))

        if mode == "plan":
            # 계획 모드: 최근 메모 로드
            print("📋 작업 계획 로드 중...")

            task_notes = self.note_tool.run({
                "action": "list",
                "note_type": "task_state",
                "limit": 3
            })

            if task_notes:
                content = "\n".join([f"- {note['title']}" for note in task_notes])
                packets.append(ContextPacket(
                    content=f"[현재 작업]\n{content}",
                    timestamp=datetime.now(),
                    token_count=len(content) // 4,
                    relevance_score=0.8,
                    metadata={"type": "task_plan", "source": "notes"}
                ))

        return packets

    def _retrieve_relevant_notes(self, query: str, limit: int = 3) -> List[Dict]:
        """관련 메모 검색"""
        try:
            # blocker 우선 검색
            blockers = self.note_tool.run({
                "action": "list",
                "note_type": "blocker",
                "limit": 2
            })

            # 관련 메모 검색
            search_results = self.note_tool.run({
                "action": "search",
                "query": query,
                "limit": limit
            })

            # 병합 및 중복 제거
            all_notes = {note.get('note_id') or note.get('id'): note for note in (blockers or []) + (search_results or [])}
            return list(all_notes.values())[:limit]

        except Exception as e:
            print(f"[WARNING] 메모 검색 실패: {e}")
            return []

    def _notes_to_packets(self, notes: List[Dict]) -> List[ContextPacket]:
        """메모를 컨텍스트 패킷으로 변환"""
        packets = []

        for note in notes:
            # 메모 유형에 따라 다른 관련성 점수 설정
            relevance_map = {
                "blocker": 0.9,
                "action": 0.8,
                "task_state": 0.75,
                "conclusion": 0.7
            }

            note_type = note.get('type', 'general')
            relevance = relevance_map.get(note_type, 0.6)

            content = f"[메모:{note.get('title', 'Untitled')}]\n유형: {note_type}\n\n{note.get('content', '')}"

            packets.append(ContextPacket(
                content=content,
                timestamp=datetime.fromisoformat(note.get('updated_at', datetime.now().isoformat())),
                token_count=len(content) // 4,
                relevance_score=relevance,
                metadata={
                    "type": "note",
                    "note_type": note_type,
                    "note_id": note.get('note_id') or note.get('id')
                }
            ))

        return packets

    def _build_system_instructions(self, mode: str) -> str:
        """시스템 지시사항 구축"""
        base_instructions = f"""당신은 {self.project_name} 프로젝트의 코드베이스 유지보수 보조 에이전트다.

핵심 능력:
1. TerminalTool로 코드베이스 탐색(ls, cat, grep, find 등)
2. NoteTool로 발견 사항과 작업 기록
3. 과거 메모를 기반으로 일관된 조언 제공

현재 세션 ID: {self.session_id}
"""

        mode_specific = {
            "explore": """
현재 모드: 코드베이스 탐색

다음을 수행해야 한다:
- terminal 명령으로 코드 구조를 능동적으로 파악
- 핵심 모듈과 파일 식별
- 프로젝트 아키텍처를 메모에 기록
""",
            "analyze": """
현재 모드: 코드 품질 분석

다음을 수행해야 한다:
- 코드 문제 찾기(중복, 복잡도, TODO 등)
- 코드 품질 평가
- 발견된 문제를 blocker 또는 action 메모로 기록
""",
            "plan": """
현재 모드: 작업 계획

다음을 수행해야 한다:
- 과거 메모와 작업 검토
- 다음 단계 행동 계획 수립
- 작업 상태 메모 업데이트
""",
            "auto": """
현재 모드: 자동 결정

다음을 수행해야 한다:
- 사용자 요구에 따라 유연하게 전략 선택
- 필요 시 도구 사용
- 전문적이고 실용적인 답변 유지
"""
        }

        return base_instructions + mode_specific.get(mode, mode_specific["auto"])

    def _postprocess_response(self, user_input: str, response: str):
        """후처리: 답변 분석, 중요 정보 자동 기록"""

        # 문제 발견 시, 자동으로 blocker 메모 생성
        if any(keyword in response.lower() for keyword in ["문제", "버그", "오류", "블로커"]):
            try:
                self.note_tool.run({
                    "action": "create",
                    "title": f"문제 발견: {user_input[:30]}...",
                    "content": f"## 사용자 입력\n{user_input}\n\n## 문제 분석\n{response[:500]}...",
                    "note_type": "blocker",
                    "tags": [self.project_name, "auto_detected", self.session_id]
                })
                self.stats["notes_created"] += 1
                self.stats["issues_found"] += 1
                print("📝 문제 메모 자동 생성 완료")
            except Exception as e:
                print(f"[WARNING] 메모 생성 실패: {e}")

        # 작업 계획이면, 자동으로 action 메모 생성
        elif any(keyword in user_input.lower() for keyword in ["계획", "다음 단계", "작업", "todo"]):
            try:
                self.note_tool.run({
                    "action": "create",
                    "title": f"작업 계획: {user_input[:30]}...",
                    "content": f"## 논의\n{user_input}\n\n## 행동 계획\n{response[:500]}...",
                    "note_type": "action",
                    "tags": [self.project_name, "planning", self.session_id]
                })
                self.stats["notes_created"] += 1
                print("📝 행동 계획 메모 자동 생성 완료")
            except Exception as e:
                print(f"[WARNING] 메모 생성 실패: {e}")

    def _update_history(self, user_input: str, response: str):
        """대화 이력 업데이트"""
        self.conversation_history.append(
            Message(content=user_input, role="user", timestamp=datetime.now())
        )
        self.conversation_history.append(
            Message(content=response, role="assistant", timestamp=datetime.now())
        )

        # 이력 길이 제한(최근 10 라운드 대화 보존)
        if len(self.conversation_history) > 20:
            self.conversation_history = self.conversation_history[-20:]

    # === 편의 메서드 ===

    def explore(self, target: str = ".") -> str:
        """코드베이스 탐색"""
        return self.run(f"{target}의 코드 구조를 탐색해달라", mode="explore")

    def analyze(self, focus: str = "") -> str:
        """코드 품질 분석"""
        query = "코드 품질을 분석해달라" + (f", {focus}에 중점을 두어" if focus else "")
        return self.run(query, mode="analyze")

    def plan_next_steps(self) -> str:
        """다음 단계 작업 계획"""
        return self.run("현재 진행 상황에 따라 다음 단계 작업을 계획해달라", mode="plan")

    def execute_command(self, command: str) -> str:
        """터미널 명령 실행"""
        result = self.terminal_tool.run({"command": command})
        self.stats["commands_executed"] += 1
        return result

    def create_note(
        self,
        title: str,
        content: str,
        note_type: str = "general",
        tags: List[str] = None
    ) -> str:
        """메모 생성"""
        result = self.note_tool.run({
            "action": "create",
            "title": title,
            "content": content,
            "note_type": note_type,
            "tags": tags or [self.project_name]
        })
        self.stats["notes_created"] += 1
        return result

    def get_stats(self) -> Dict[str, Any]:
        """통계 정보 가져오기"""
        duration = (datetime.now() - self.stats["session_start"]).total_seconds()

        # 메모 요약 가져오기
        try:
            note_summary = self.note_tool.run({"action": "summary"})
        except:
            note_summary = {}

        return {
            "session_info": {
                "session_id": self.session_id,
                "project": self.project_name,
                "duration_seconds": duration
            },
            "activity": {
                "commands_executed": self.stats["commands_executed"],
                "notes_created": self.stats["notes_created"],
                "issues_found": self.stats["issues_found"]
            },
            "notes": note_summary
        }

    def generate_report(self, save_to_file: bool = True) -> Dict[str, Any]:
        """세션 보고서 생성"""
        report = self.get_stats()

        if save_to_file:
            report_file = f"maintainer_report_{self.session_id}.json"
            with open(report_file, 'w', encoding='utf-8') as f:
                json.dump(report, f, ensure_ascii=False, indent=2, default=str)
            report["report_file"] = report_file
            print(f"📄 보고서 저장 완료: {report_file}")

        return report
```

### 9.6.4 완전한 사용 예시

이제 완전한 사용 시나리오를 통해 이 장기 에이전트의 작업 흐름을 보여주자:

```python
# ========== 보조 에이전트 초기화 ==========

from hello_agents import HelloAgentsLLM

maintainer = CodebaseMaintainer(
    project_name="my_flask_app",
    codebase_path="./my_flask_app",
    llm=HelloAgentsLLM()
)


# ========== 첫째 날: 코드베이스 탐색 ==========
# 1. 초기 탐색
response = maintainer.explore()
"""
🔍 코드베이스 구조 탐색 중...
🤖 생각 중...

🤖 보조 에이전트: 코드베이스를 스캔했다. 이것은 Flask 웹 애플리케이션으로, 주요 모듈은 다음과 같다:

📁 프로젝트 구조:
- app/
  ├── models/      # 데이터 모델(User, Product, Order 등)
  ├── routes/      # API 라우트
  ├── services/    # 비즈니스 로직
  └── utils/       # 유틸리티 함수
- tests/           # 테스트 파일
- migrations/      # 데이터베이스 마이그레이션
- config.py        # 설정 파일

총 약 3,500줄의 Python 코드.

다음 단계 제안:
1. 핵심 모델 정의 보기
2. 라우트와 API 설계 점검
3. 테스트 커버리지 평가
"""

# 2. 특정 모듈 심층 분석
response = maintainer.run("app/models/ 디렉토리의 데이터 모델 설계를 분석해달라")
"""
🤖 보조 에이전트: app/models/ 디렉토리를 확인했다. 다음 데이터 모델들을 발견했다:

1. User (user.py)
   - 필드: id, username, email, password_hash, created_at
   - 관계: 일대다 Order
   - 문제: ⚠️ 인덱스 없음, email 필드에 고유 제약이 없음

2. Product (product.py)
   - 필드: id, name, description, price, stock
   - 문제: ✅ 설계 합리적

3. Order (order.py)
   - 필드: id, user_id, product_id, quantity, status
   - 문제: ⚠️ 생성 시간 필드 없음, 데이터 분석에 불리함

제안:
1. User.email에 고유 제약 추가
2. Order에 created_at, updated_at 필드 추가
3. 소프트 삭제 지원 추가 고려

이 문제들을 blocker 메모로 기록했다.
"""
# [blocker 메모 자동 생성됨]


# ========== 둘째 날: 코드 품질 분석 ==========

# 1. 전체 품질 분석
response = maintainer.analyze()
"""
📊 코드 품질 분석 중...
🤖 생각 중...

🤖 보조 에이전트: 코드 품질 분석 보고서:

📊 코드 통계:
- 총 행 수: 3,542줄
- 파일 평균 길이: 약 150줄
- 가장 큰 파일: services/order_service.py (456줄)

⚠️ 발견된 문제:

1. 코드 중복 (HIGH):
   - user_service.py와 product_service.py에 유사한 CRUD 로직 있음
   - 제안: BaseService 기반 클래스 추출

2. 복잡도 과다 (MEDIUM):
   - order_service.py::process_order 메서드에 8단계 중첩 있음
   - 제안: 여러 개의 작은 메서드로 분리

3. 테스트 부족 (HIGH):
   - 테스트 커버리지 45%에 불과
   - services/ 디렉토리 테스트 전무

4. TODO/FIXME (12개):
   - 대부분 "나중에 최적화" 류의 주석
   - 제안: 정리하거나 공식 작업으로 전환

이 문제들을 메모로 기록하고 우선순위를 표시했다.
"""
# [여러 blocker 메모 자동 생성됨]

# 2. 구체적인 문제 보기
response = maintainer.run("order_service.py의 process_order 메서드를 보고 리팩토링 제안을 해달라")
"""
🤖 보조 에이전트: process_order 메서드 코드를 확인했다:

【원래 코드】
def process_order(self, order_id):
    order = Order.query.get(order_id)
    if order:
        if order.status == 'pending':
            product = Product.query.get(order.product_id)
            if product:
                if product.stock >= order.quantity:
                    # ... 8단계 중첩 계속 ...

【문제 분석】
1. 깊은 중첩으로 가독성 저하
2. 오류 처리 없음
3. 비즈니스 로직과 데이터 접근이 혼재

【리팩토링 제안】
def process_order(self, order_id):
    # 1. 조기 반환 패턴, 중첩 감소
    order = self._get_order(order_id)
    if not order or order.status != 'pending':
        return False

    # 2. 메서드 추출, 단일 책임
    if not self._check_stock(order):
        return False

    # 3. 트랜잭션으로 일관성 보장
    with db.transaction():
        self._update_stock(order)
        self._update_order_status(order, 'processing')

    return True

【다음 단계】
1. 리팩토링 계획 메모를 생성했다
2. 먼저 테스트 케이스를 작성할 것을 권장
3. 점진적으로 리팩토링하되, 기능은 변하지 않게 유지
"""
# [action 메모 자동 생성됨]


# ========== 셋째 날: 리팩토링 작업 계획 ==========

# 1. 진행 상황 검토
response = maintainer.plan_next_steps()
"""
📋 작업 계획 로드 중...
🤖 생각 중...

🤖 보조 에이전트: 처음 이틀간의 탐색과 분석을 바탕으로 리팩토링 작업 우선순위를 정리했다:

🔴 높은 우선순위 (이번 주 완료):
1. [blocker] User.email 고유 제약 추가
   - 위험도: 낮음, 데이터베이스 마이그레이션만
   - 작업량: 1시간

2. [blocker] BaseService 추출로 코드 중복 감소
   - 위험도: 중간, 여러 서비스 클래스에 영향
   - 작업량: 반나절

3. [blocker] process_order 메서드 리팩토링
   - 위험도: 높음, 핵심 비즈니스 로직
   - 작업량: 1일(테스트 작성 포함)

🟡 중간 우선순위 (다음 주):
4. services/에 단위 테스트 추가
5. TODO 주석 정리
6. Order에 시간 필드 추가

🟢 낮은 우선순위 (미정):
7. 성능 최적화
8. 문서 업데이트

제안:
- 이번 주에 먼저 데이터 모델 문제 해결(작업 1, 6)
- 다음 주에 코드 리팩토링(작업 2, 3)
- 지속적으로 테스트 커버리지 향상(작업 4)

작업 상태 메모를 업데이트했다. 언제든지 진행 상황을 확인할 수 있다.
"""

# 2. 상세 리팩토링 계획 수동 생성
maintainer.create_note(
    title="이번 주 리팩토링 계획 - 1주차",
    content="""## 목표
데이터 모델 계층 최적화 완료

## 작업 목록
- [ ] User.email에 고유 제약 추가
- [ ] Order에 created_at, updated_at 필드 추가
- [ ] 데이터베이스 마이그레이션 스크립트 작성
- [ ] 관련 테스트 케이스 업데이트

## 시간 계획
- 월요일: 마이그레이션 스크립트 설계
- 화-수요일: 마이그레이션 실행 및 테스트
- 목요일: 테스트 케이스 업데이트
- 금요일: 코드 리뷰

## 위험 요소
- 데이터베이스 마이그레이션이 운영 환경에 영향을 줄 수 있으므로 비피크 시간에 실행 필요
- 기존 데이터에 중복 이메일이 있을 수 있어 사전 정리 필요
""",
    note_type="task_state",
    tags=["refactoring", "week1", "high_priority"]
)

print("✅ 상세 리팩토링 계획 생성 완료")


# ========== 일주일 후: 진행 상황 확인 ==========

# 메모 요약 보기
summary = maintainer.note_tool.run({"action": "summary"})
print("📊 메모 요약:")
print(json.dumps(summary, indent=2, ensure_ascii=False))
"""
{
  "total_notes": 8,
  "type_distribution": {
    "blocker": 3,
    "action": 2,
    "task_state": 2,
    "conclusion": 1
  },
  "recent_notes": [
    {
      "id": "note_20250119_160000_7",
      "title": "이번 주 리팩토링 계획 - 1주차",
      "type": "task_state",
      "updated_at": "2025-01-19T16:00:00"
    },
    ...
  ]
}
"""

# 완전한 보고서 생성
report = maintainer.generate_report()
print("\n📄 세션 보고서:")
print(json.dumps(report, indent=2, ensure_ascii=False))
"""
{
  "session_info": {
    "session_id": "session_20250119_150000",
    "project": "my_flask_app",
    "duration_seconds": 172800
  },
  "activity": {
    "commands_executed": 24,
    "notes_created": 8,
    "issues_found": 3
  },
  "notes": { ... }
}
"""
```

### 9.6.5 실행 효과 분석

이 완전한 사례를 통해, 장기 에이전트의 몇 가지 핵심 특성을 확인할 수 있다. 첫째로 **세션 간 일관성**이다 — 에이전트는 NoteTool을 통해 며칠, 여러 세션에 걸친 작업 일관성을 유지하며, 첫째 날 탐색한 문제가 둘째 날 분석 시 자동으로 반영되고, 셋째 날 계획 시 이틀간의 모든 발견이 종합되며, 일주일 후 확인 시 완전한 이력이 보존된다. 둘째로 **지능적인 컨텍스트 관리**다 — ContextBuilder는 매번 고품질의 컨텍스트를 보장하며, 관련 메모(특히 blocker 유형)를 자동으로 수집하고, 대화 모드에 따라 전처리 전략을 동적으로 조정하며, 토큰 예산 내에서 가장 관련성 높은 정보를 선택한다.

셋째는 **즉시적인 파일 시스템 접근**이다 — TerminalTool은 유연한 코드 탐색을 지원하며, 전체 코드베이스를 미리 인덱싱할 필요 없이, 즉시 특정 파일 내용을 볼 수 있고, 복잡한 텍스트 처리(grep, awk 등)를 지원한다. 넷째는 **자동화된 지식 관리**다 — 시스템은 자동으로 발견된 지식을 관리하며, 문제 발견 시 자동으로 blocker 메모를 생성하고, 계획 논의 시 자동으로 action 메모를 생성하며, 핵심 정보가 자동으로 기억 시스템에 저장된다. 마지막으로 **인간-기계 협력**이다 — 이 시스템은 유연한 인간-기계 협력 모드를 지원하며, 에이전트가 탐색과 분석을 자동으로 완료하고, 인간이 메모 시스템을 통해 개입하고 안내할 수 있으며, 상세한 계획 메모를 수동으로 생성하는 것을 지원한다.

이 기본 프레임워크는 더 확장될 수 있다. 예를 들어 RAGTool을 통합하여 코드베이스의 벡터 인덱스와 의미론적 검색을 결합하거나, 탐색자, 분석자, 계획자 등 전문화된 역할로 분리하여 다중 에이전트 협력을 구현하거나, 테스트 도구를 통합하여 리팩토링 결과를 자동으로 검증하거나, TerminalTool로 git 명령을 실행하여 코드 변경을 추적하거나, Gradio/Streamlit으로 시각화 인터페이스를 구축하는 등의 방향이 있다.

---

## 9.7 이 장의 요약

이 장에서 우리는 컨텍스트 엔지니어링의 이론적 토대와 공학적 실천을 심층적으로 탐구했다:

**이론적 측면**

1. **컨텍스트 엔지니어링의 본질**: "프롬프트 엔지니어링"에서 "컨텍스트 엔지니어링"으로의 진화, 핵심은 유한한 주의력 예산을 관리하는 것이다
2. **컨텍스트 부패**: 긴 컨텍스트로 인한 성능 저하를 이해하고, 컨텍스트가 희소 자원임을 인식한다
3. **세 가지 핵심 전략**: 압축 통합, 구조화된 메모, 서브 에이전트 아키텍처

**공학적 실천**

1. **ContextBuilder**: GSSC 파이프라인을 구현하여 통합된 컨텍스트 관리 인터페이스를 제공한다
2. **NoteTool**: Markdown+YAML 혼합 형식으로, 구조화된 장기 기억을 지원한다
3. **TerminalTool**: 안전한 명령줄 도구로, 즉시적인 파일 시스템 접근을 지원한다
4. **장기 에이전트**: 세 가지 도구를 통합하여 세션 간 코드베이스 유지보수 보조 에이전트를 구축했다

**핵심 수확**

- **계층적 설계**: 즉시 접근(TerminalTool) + 세션 기억(MemoryTool) + 영속적 메모(NoteTool)
- **지능적 선별**: 관련성과 최신성에 기반한 채점 메커니즘
- **안전 우선**: 다중 계층 보안 메커니즘으로 시스템 안정성 보장
- **인간-기계 협력**: 자동화와 제어 가능성의 균형

이 장을 통해 컨텍스트 엔지니어링의 핵심 기술을 익혔을 뿐 아니라, 더 중요하게는 긴 시간 범위에서 일관성과 효과성을 유지할 수 있는 에이전트 시스템을 어떻게 구축하는지 이해했다. 이 기술들은 생산급 에이전트 애플리케이션을 구축하는 중요한 토대가 될 것이다.

다음 장에서는 에이전트 통신 프로토콜을 탐구하고, 에이전트가 외부 세계와 더 광범위하게 상호작용하는 방법을 학습할 것이다.

---

## 연습 문제

> **힌트**: 일부 연습 문제에는 표준 답안이 없으며, 컨텍스트 엔지니어링과 장기 작업 관리에 대한 학습자의 종합적 이해와 실천 능력을 키우는 데 중점을 둔다.

1. 이 장은 컨텍스트 엔지니어링과 프롬프트 엔지니어링의 차이를 소개했다. 다음을 분석해보자:

   - 9.1절에서 "컨텍스트는 유한한 자원으로 간주되어야 하며, 한계 수익 체감 현상이 있다"고 언급했다. "컨텍스트 부패(context rot)" 현상이란 무엇인지 설명해보자. 모델이 100K 심지어 200K의 컨텍스트 윈도우를 지원하더라도 왜 여전히 컨텍스트를 신중하게 관리해야 하는가?
   - 50개의 파일을 포함한 코드베이스를 분석하는 "코드 리뷰 보조 에이전트"를 구축한다고 가정하자. 두 가지 전략을 비교해보자: (1) 모든 파일 내용을 한 번에 컨텍스트에 로드; (2) JIT(Just-in-time) 컨텍스트를 사용하여 도구를 통해 파일을 필요에 따라 검색. 각각의 장단점과 적합한 시나리오를 분석해보자.
   - 9.2.1절에서 시스템 프롬프트의 두 극단적 오류: "과도한 하드코딩"과 "지나치게 추상적"을 언급했다. 각각의 실제 예시를 들고, 적절한 균형점을 어떻게 찾을지 설명해보자.

2. GSSC(Gather-Select-Structure-Compress) 파이프라인은 이 장의 핵심 기술이다. 다음을 심층적으로 생각해보자:

   > **힌트**: 실습 문제이므로, 실제로 작동해볼 것을 권장한다

   - 9.3절의 ContextBuilder 구현에서 네 단계는 각기 다른 책임을 갖는다. 어느 단계가 실패할 경우(예: Select 단계가 관련 없는 정보를 선택하거나, Compress 단계의 과도한 압축으로 정보가 손실될 경우), 최종 에이전트 성능에 어떤 영향을 미치는지 분석해보자.
   - 9.3.4절의 코드를 기반으로, ContextBuilder에 "컨텍스트 품질 평가" 기능을 추가해보자: 매번 컨텍스트를 구축한 후 자동으로 컨텍스트의 정보 밀도, 관련성, 완전성을 평가하고 최적화 제안을 제공하도록 한다.
   - GSSC 파이프라인의 "압축" 단계에서는 LLM을 사용하여 지능적 요약을 수행한다. 어떤 상황에서 단순한 절단(truncation)이나 슬라이딩 윈도우(sliding window) 전략이 LLM 요약보다 더 적합할 수 있는지 생각해보자. 여러 압축 방법의 장점을 결합한 혼합 압축 전략을 설계해보자.

3. NoteTool과 TerminalTool은 장기 작업을 지원하는 핵심 도구다. 9.4절과 9.5절의 내용을 바탕으로 다음 확장 실습을 완료해보자:

   > **힌트**: 실습 문제이므로, 실제로 작동해볼 것을 권장한다

   - NoteTool은 계층적 메모 시스템(프로젝트 메모, 작업 메모, 임시 메모)을 사용한다. "메모 자동 정리" 메커니즘을 설계해보자: 임시 메모가 일정 수에 달하면, 에이전트가 자동으로 이 메모들을 분석하여 중요한 정보를 작업 메모나 프로젝트 메모로 승격시키고, 중복 내용을 정리한다.
   - TerminalTool은 파일 시스템 조작 능력을 제공하지만, 9.5.2절에서 보안 설계를 강조했다. 현재의 보안 메커니즘(경로 검증, 명령 화이트리스트, 권한 확인)이 충분한지 분석해보자. 에이전트가 민감한 파일에 접근하거나 위험한 작업을 실행해야 하는 경우, "인간-기계 협력 심의" 프로세스를 어떻게 설계해야 하는가?
   - NoteTool과 TerminalTool을 결합하여 "지능형 코드 리팩토링 보조 에이전트"를 설계해보자: 코드베이스 구조를 분석하고, 리팩토링 계획을 기록하며, 점진적으로 리팩토링 작업을 실행하고, 메모에서 진행 상황과 만난 문제를 추적할 수 있는 에이전트다. 완전한 워크플로우 다이어그램을 그려보자.

4. 9.6절의 "장기 작업 관리" 사례에서 실제 응용에서 컨텍스트 엔지니어링의 가치를 확인했다. 다음을 심층 분석해보자:

   - 사례에서 "계층적 컨텍스트 관리" 전략을 사용했다: 즉시 접근(TerminalTool) + 세션 기억(MemoryTool) + 영속적 메모(NoteTool). 이 세 계층 간의 조율 방법을 분석해보자. 어떤 정보가 어느 계층에 있어야 하는가? 정보의 중복과 불일치를 어떻게 피하는가?
   - 작업 실행 중 중단이 발생했다고 가정하자(시스템 충돌, 네트워크 단절 등). 에이전트는 메모에서 상태를 복원하고 계속 실행해야 한다. "중단점 재개" 메커니즘을 설계해보자: 메모에 충분한 상태 정보를 어떻게 기록하는가? 복원 후 상태가 올바른지 어떻게 검증하는가?
   - 장기 작업은 종종 여러 하위 작업의 병렬 또는 직렬 실행을 포함한다. "작업 의존성 관리" 시스템을 설계해보자: 작업 간의 의존 관계를 표현할 수 있고("작업 B는 작업 A 완료 후 실행 가능" 같은), 작업 실행 순서를 자동으로 스케줄링할 수 있는 시스템이다. 이 시스템은 NoteTool과 어떻게 통합되어야 하는가?

5. 이 장에서 "점진적 공개(progressive disclosure)"의 개념을 여러 번 언급했다. 다음을 생각해보자:

   - 9.2.2절에서 점진적 공개를 "각 상호작용 단계가 새로운 컨텍스트를 생성하고, 이것이 다시 다음 단계의 결정을 인도한다"고 설명했다. 구체적인 응용 시나리오(예: 학술 논문 작성, 복잡한 문제 디버깅)를 설계하여, 점진적 공개가 에이전트가 더 효율적으로 작업을 완료하는 데 어떻게 도움이 되는지 보여주자.
   - 점진적 공개의 잠재적 위험은 "탐색 효율 저하"다: 에이전트가 중요하지 않은 세부사항에 시간을 낭비하거나, 핵심 정보를 놓칠 수 있다. "탐색 가이드" 메커니즘을 설계해보자: 휴리스틱 규칙이나 메타인지 전략을 통해, 에이전트가 "다음에 무엇을 탐색해야 하는지"를 더 영리하게 결정할 수 있도록 돕는다.
   - "점진적 공개"와 전통적인 "모든 컨텍스트를 한 번에 로드"를 비교해보자: 어떤 유형의 작업에서 전자가 명확한 우위를 갖는가? 어떤 유형의 작업에서 후자가 더 적합할 수 있는가? 적어도 3가지 다른 유형의 작업 예시를 제시해보자.

---

## 참고문헌

[1] Anthropic. Effective Context Engineering for AI Agents. `https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents`

[2] David Kim. Context-Engineering (GitHub). `https://github.com/davidkimai/Context-Engineering`