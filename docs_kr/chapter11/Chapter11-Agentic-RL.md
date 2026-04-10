# 제11장 Agentic-RL

## 11.1 LLM 훈련에서 Agentic RL까지

앞선 장들에서 우리는 다양한 에이전트 패러다임과 통신 프로토콜을 구현했다. 그런데 에이전트가 더 복잡한 작업을 처리할 때 성능이 좋지 않으면 자연스럽게 이런 의문이 생긴다: **에이전트에게 더 강한 추론 능력을 어떻게 부여할 것인가? 에이전트가 도구를 더 잘 활용하도록 어떻게 학습시킬 것인가? 에이전트가 스스로 개선할 수 있도록 어떻게 할 것인가?**

이것이 바로 Agentic RL(강화학습 기반 에이전트 훈련)이 해결하고자 하는 핵심 문제다. 본 장에서는 HelloAgents 프레임워크에 강화학습 훈련 기능을 도입하여, 추론·도구 사용 등 고급 능력을 갖춘 에이전트를 훈련할 수 있도록 한다. LLM 훈련의 기초 지식부터 시작하여, 지도 미세조정(Supervised Fine-Tuning, SFT), 군집 상대 정책 최적화(Group Relative Policy Optimization, GRPO) 등 실용적인 기법을 단계적으로 심화 학습하고, 최종적으로 완전한 에이전트 훈련 파이프라인을 구축한다.

### 11.1.1 강화학습에서 Agentic RL까지

2장의 2.4.2절에서 강화학습 기반 에이전트를 소개했다. 강화학습(Reinforcement Learning, RL)은 순차적 의사결정 문제를 해결하는 데 특화된 학습 패러다임으로, 에이전트와 환경의 직접적인 상호작용을 통해 "시행착오" 속에서 장기적 보상을 최대화하는 방법을 학습한다.

이제 이 프레임워크를 LLM 에이전트에 적용해보자. 다음과 같은 수학 문제를 풀어야 하는 에이전트를 생각해보자:

```
문제: Janet's ducks lay 16 eggs per day. She eats three for breakfast
every morning and bakes muffins for her friends every day with four.
She sells the remainder at the farmers' market daily for $2 per fresh
duck egg. How much in dollars does she make every day at the farmers' market?
```

이 문제는 다단계 추론이 필요하다: 먼저 Janet이 매일 남은 달걀 수(16 - 3 - 4 = 9)를 계산하고, 그 다음 수입(9 × 2 = 18)을 계산한다. 이 작업을 강화학습 프레임워크에 매핑하면 다음과 같다:

- **에이전트**: LLM 기반 추론 시스템
- **환경**: 수학 문제 및 검증 시스템
- **상태**: 현재 문제 설명 및 기존 추론 단계
- **행동**: 다음 추론 단계 또는 최종 답 생성
- **보상**: 답이 맞으면 +1, 틀리면 0

전통적인 지도학습 방식에는 세 가지 핵심적인 한계가 있다: 첫째, 데이터 품질이 훈련 품질을 완전히 결정하며 모델은 훈련 데이터를 모방할 뿐 그것을 뛰어넘기 어렵다. 둘째, 탐색 능력이 부족하여 인간이 제공하는 경로를 수동적으로 학습할 수밖에 없다. 셋째, 장기 목표를 최적화하기 어려워 다단계 추론의 중간 과정을 정밀하게 최적화할 수 없다.

강화학습은 새로운 가능성을 제공한다. 에이전트가 스스로 여러 후보 답안을 생성하고 정답 여부에 따라 보상을 받게 함으로써, 어떤 추론 경로가 더 우수한지, 어떤 단계가 핵심인지를 학습할 수 있으며, 심지어 인간이 주석을 단 것보다 더 나은 풀이 방법을 발견할 수도 있다[8]. 이것이 Agentic RL의 핵심 사상이다: LLM을 학습 가능한 정책으로 보고, 에이전트의 인식-결정-실행 루프에 내장하여, 강화학습을 통해 다단계 작업 성능을 최적화한다.

### 11.1.2 LLM 훈련 전체 그림

Agentic RL을 깊이 다루기 전에 먼저 LLM 훈련의 전체 흐름을 이해해야 한다. GPT, Claude, Qwen 같은 강력한 LLM이 탄생하기까지 보통 두 가지 주요 단계를 거친다: 사전 훈련(Pretraining)과 후처리 훈련(Post-training). 그림 11.1에서 보듯이 이 두 단계는 LLM이 "언어 모델"에서 "대화 어시스턴트"로 진화하는 완전한 경로를 구성한다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-1.png" alt="" width="85%"/>
  <p>그림 11.1 LLM 훈련 전체 그림</p>
</div>

**사전 훈련 단계**는 LLM 훈련의 첫 번째 단계로, 모델이 언어의 기본 규칙과 세계 지식을 학습하도록 하는 것이 목표다. 이 단계에서는 방대한 텍스트 데이터(보통 수 TB 규모)를 사용하여 자기 지도 학습 방식으로 모델을 훈련한다. 가장 일반적인 사전 훈련 작업은 인과적 언어 모델링(Causal Language Modeling), 즉 다음 토큰 예측(Next Token Prediction)이다.

주어진 텍스트 시퀀스 $x_1, x_2, ..., x_t$에서 모델은 다음 단어 $x_{t+1}$을 예측해야 한다:

$$
\mathcal{L}_{\text{pretrain}} = -\sum_{t=1}^{T} \log P(x_t | x_1, x_2, ..., x_{t-1}; \theta)
$$

여기서 $\theta$는 모델 파라미터, $P(x_t | x_1, ..., x_{t-1}; \theta)$는 모델이 예측하는 다음 단어의 확률 분포다. 목표는 음의 로그 가능도를 최소화하는 것, 즉 올바른 단어를 예측할 확률을 최대화하는 것이다. 예를 들어 "The cat sat on the"라는 텍스트가 주어지면 모델은 다음 단어로 "mat"이 가장 유력하다고 예측해야 한다. 방대한 텍스트로 이런 훈련을 하면서 모델은 점차 문법 규칙(어떤 어순이 적합한지), 의미적 지식(단어 간 관계), 세계 지식(세계에 대한 사실적 정보), 기초적인 추론 능력을 습득한다.

사전 훈련 단계의 특징은 데이터량이 방대하고 계산 비용이 높으며, 일반적인 언어 이해 및 생성 능력을 습득하고 비지도 학습 방식을 채택한다는 것이다.

**후처리 훈련 단계**는 사전 훈련 모델의 부족함을 해결하기 위한 것이다. 사전 훈련 후의 모델은 강력한 언어 능력을 갖추고 있지만, 단지 "다음 단어를 예측"하는 모델일 뿐이다. 사람의 지시를 따르거나, 도움이 되고 무해하며 정직한 답변을 생성하거나, 부적절한 요청을 거절하거나, 대화 방식으로 사람과 상호작용하는 방법을 모른다. 후처리 훈련 단계는 이러한 문제들을 해결하여 모델이 인간의 선호와 가치관에 정렬(align)되도록 한다.

후처리 훈련은 보통 세 단계를 포함한다. 첫 번째 단계는 **지도 미세조정(SFT)**[15]으로, 모델이 지시 사항과 대화 형식을 따르도록 학습하는 것이 목표다. 훈련 데이터는 (프롬프트, 완성) 쌍이며, 훈련 목표는 사전 훈련과 유사하게 올바른 출력의 확률을 최대화하는 것이다:

$$
\mathcal{L}_{\text{SFT}} = -\sum_{i=1}^{N} \log P(y_i | x_i; \theta)
$$

여기서 $x_i$는 입력 프롬프트, $y_i$는 기대되는 출력, $N$은 훈련 샘플 수다. SFT의 특징은 데이터량이 적고, 인간 주석이 필요하며, 빠른 효과를 내고, 주로 작업 형식과 기본 능력을 학습한다는 것이다.

두 번째 단계는 **보상 모델링(RM)**이다. SFT 후의 모델은 지시를 따를 수 있지만 생성된 답변의 품질이 들쭉날쭉하다. 답변 품질을 평가하는 방법이 필요한데, 이것이 보상 모델의 역할이다[13,14]. 보상 모델의 훈련 데이터는 선호도 비교 데이터로, 동일한 질문에 대한 두 가지 답변(하나는 더 좋은 것(chosen), 하나는 더 나쁜 것(rejected))을 포함한다. 보상 모델의 훈련 목표는 인간의 선호도를 학습하는 것이다:

$$
\mathcal{L}_{\text{RM}} = -\mathbb{E}_{(x, y_w, y_l)} [\log \sigma(r_\phi(x, y_w) - r_\phi(x, y_l))]
$$

여기서 $r_\phi(x, y)$는 보상 모델로, (프롬프트, 답변) 쌍을 입력받아 품질 점수를 출력한다. $y_w$는 더 좋은 답변(chosen), $y_l$은 더 나쁜 답변(rejected), $\sigma$는 시그모이드 함수다. 목표는 보상 모델이 더 좋은 답변에 더 높은 점수를 부여하도록 학습하는 것이다.

세 번째 단계는 **강화학습 미세조정**이다. 보상 모델을 갖추고 나면 강화학습을 사용하여 언어 모델을 최적화하고 더 높은 품질의 답변을 생성하도록 한다. 가장 고전적인 알고리즘은 PPO(Proximal Policy Optimization)[1]이며, 훈련 목표는 다음과 같다:

$$
J_{\text{PPO}} = \mathbb{E}_{x, y \sim \pi_\theta} [r_\phi(x, y)] - \beta \cdot D_{KL}(\pi_\theta || \pi_{\text{ref}})
$$

여기서 $\pi_\theta$는 현재 정책(언어 모델), $\pi_{\text{ref}}$는 참조 정책(이 시나리오에서는 SFT 모델이 될 수 있음), $r_\phi(x, y)$는 보상 모델의 점수, $D_{KL}$은 KL 발산으로 모델이 너무 멀리 벗어나는 것을 방지하는 역할을 하며, $\beta$는 균형 계수다. 이 목적 함수의 의미는 보상을 최대화하되, 원래 모델에서 너무 멀리 벗어나지 않도록 하는 것이다.

전통적인 RLHF(Reinforcement Learning from Human Feedback)[5]는 대량의 인간 주석 선호도 데이터가 필요하여 비용이 막대하다. 비용을 낮추기 위해 연구자들은 RLAIF(Reinforcement Learning from AI Feedback)[7]를 제안했는데, 강력한 AI 모델(예: GPT-4)이 인간 주석자를 대체한다. RLAIF의 작업 흐름은 다음과 같다: SFT 모델을 사용하여 여러 후보 답변을 생성하고, 강력한 AI 모델이 답변을 평가 및 순위 매기며, AI의 평가로 보상 모델을 훈련하고, 보상 모델을 이용하여 강화학습을 수행한다. 실험에 따르면 RLAIF의 효과는 RLHF와 비슷하거나 그를 능가하는 경우도 있으며, 비용은 크게 낮아진다[11].

### 11.1.3 Agentic RL의 핵심 이념

LLM의 기본 훈련 흐름을 이해한 후, Agentic RL과 전통적인 훈련 방법의 차이를 살펴보자. 전통적인 후처리 훈련(여기서는 PBRFT: Preference-Based Reinforcement Fine-Tuning이라 부름)은 주로 단일 대화 품질 최적화에 초점을 맞춘다: 사용자 질문이 주어지면 모델이 하나의 답변을 생성하고, 답변 품질에 따라 보상을 받는다. 이 방식은 대화 어시스턴트 최적화에는 적합하지만, 다단계 추론·도구 사용·장기 계획이 필요한 에이전트 작업에는 역부족이다.

**Agentic RL**은 새로운 패러다임으로, LLM을 학습 가능한 정책으로 보고 순차적 의사결정 루프에 내장시킨다. 이 프레임워크에서 에이전트는 동적 환경 속에서 외부 세계와 상호작용하고, 다단계 행동을 수행하여 복잡한 작업을 완수하며, 중간 피드백을 통해 후속 결정을 안내받고, 단일 보상이 아닌 장기 누적 보상을 최적화해야 한다.

구체적인 예를 통해 이 차이를 이해해보자. PBRFT 시나리오에서는 사용자가 "강화학습이 무엇인지 설명해달라"고 질문하면 모델이 완전한 답변을 생성하고 답변 품질에 따라 직접 점수를 받는다. 반면 Agentic RL 시나리오에서는 사용자가 "이 GitHub 저장소의 코드 품질을 분석해달라"고 요청하면 에이전트는 여러 단계를 거쳐야 한다: 먼저 GitHub API를 호출하여 저장소 정보를 가져오고 저장소 구조와 파일 목록을 성공적으로 획득하면 +0.1 보상을 받고, 그 다음 주요 코드 파일을 읽어 코드 내용을 성공적으로 가져오면 +0.1 보상을 받으며, 코드 품질을 합리적으로 분석하면 +0.2 보상을 받고, 마지막으로 고품질 분석 보고서를 생성하면 +0.6 보상을 받는다. 총 보상은 모든 단계의 누적값인 1.0이다.

Agentic RL의 핵심 특징은 다단계 상호작용이고, 각 단계의 행동이 환경 상태를 바꾸며, 각 단계에서 피드백을 받을 수 있고, 전체 작업의 완성 품질을 최적화한다는 것을 알 수 있다.

강화학습은 마르코프 결정 과정(Markov Decision Process, MDP) 프레임워크를 기반으로 형식화된다. MDP는 5-튜플 $(S, A, P, R, \gamma)$으로 정의된다: 상태 공간 $S$, 행동 공간 $A$, 상태 전이 함수 $P(s'|s,a)$, 보상 함수 $R(s,a)$, 할인 인수 $\gamma$. MDP 관점에서 PBRFT와 Agentic RL을 비교하면 표 11.1과 같다.

<div align="center">
  <p>표 11.1 PBRFT와 Agentic RL 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-1.png" alt="" width="85%"/>
</div>

상태 측면에서 PBRFT의 상태 $s_0$는 사용자 프롬프트로만 구성되며, 시간 범위 $T=1$(단일 단계)로 상태가 변하지 않는다. $s_0 = \text{prompt}$로 표현된다. 반면 Agentic RL의 상태 $s_t$는 역사적 관찰과 컨텍스트를 포함하며, 시간 범위 $T \gg 1$(다단계)로 상태가 행동에 따라 진화한다. $s_t = (\text{prompt}, o_1, o_2, ..., o_t)$로 표현되는데, 여기서 $o_t$는 $t$번째 단계의 관찰(도구 반환 결과, 환경 피드백 등)이다.

행동 측면에서 PBRFT의 행동 공간은 텍스트 생성 하나뿐이며, 단일 행동 유형으로 $a = y \sim \pi_\theta(y|s_0)$로 표현된다. 반면 Agentic RL의 행동 공간은 텍스트 생성, 도구 호출, 환경 조작 등 여러 유형을 포함하며, $a_t \in \{a_t^{\text{text}}, a_t^{\text{tool}}\}$로 표현된다. 예를 들어 $a_t^{\text{text}}$는 사고 과정이나 답변을 생성하는 것이고, $a_t^{\text{tool}}$은 계산기, 검색 엔진 등의 도구를 호출하는 것이다.

전이 함수 측면에서 PBRFT는 상태 전이가 없으며, $P(s'|s,a) = \delta(s' - s_{\text{terminal}})$로 표현된다. 반면 Agentic RL에서는 행동과 환경에 따라 상태가 동적으로 변화하며, $s_{t+1} \sim P(s_{t+1}|s_t, a_t)$로 표현된다. 예를 들어 검색 도구를 호출하면 상태에 검색 결과가 포함된다.

보상 측면에서 PBRFT는 단일 단계 보상 $r(s_0, a)$만 있으며, 작업 종료 시에만 부여된다: $R_{\text{PBRFT}} = r(s_0, y)$, 보통 보상 모델이 제공한다 $r(s_0, y) = r_\phi(s_0, y)$. 반면 Agentic RL은 다단계 보상 $r(s_t, a_t)$이 있으며, 중간 단계에서도 부분 보상을 줄 수 있다:

$$
R_{\text{Agentic}} = \sum_{t=0}^{T} \gamma^t r(s_t, a_t)
$$

여기서 $\gamma \in [0,1]$은 할인 인수이며, $r(s_t, a_t)$는 희소 보상(작업 완료 시에만 부여, 예: 정답 +1), 밀집 보상(매 단계 부여, 예: 도구 호출 성공 +0.1), 또는 둘을 결합한 혼합 보상이 될 수 있다.

목적 함수 측면에서 PBRFT는 단일 단계 기대 보상을 최대화한다:

$$
J_{\text{PBRFT}}(\theta) = \mathbb{E}_{s_0, y \sim \pi_\theta} [r(s_0, y)]
$$

반면 Agentic RL은 누적 할인 보상을 최대화한다:

$$
J_{\text{Agentic}}(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} \left[\sum_{t=0}^{T} \gamma^t r(s_t, a_t)\right]
$$

여기서 $\tau = (s_0, a_0, s_1, a_1, ..., s_T)$는 완전한 궤적(trajectory)이다.

이러한 전환은 단순한 기술적 세부 사항의 차이가 아니라 사고 방식의 근본적인 전환이다. PBRFT 사고는 "모델이 더 좋은 단일 답변을 생성하도록 어떻게 할 것인가"에 초점을 맞추어 답변 품질을 최적화하고 언어 표현에 집중하며 단일 단계 결정을 내린다. 반면 Agentic RL 사고는 "에이전트가 복잡한 작업을 완수하도록 어떻게 할 것인가"에 초점을 맞추어 작업 완성도를 최적화하고 행동 전략에 집중하며 다단계 계획을 수립한다. 이러한 전환은 LLM을 "대화 어시스턴트"에서 "자율 에이전트"로 진화시키며, 능동적으로 정보를 찾고, 언제 어떻게 외부 도구를 사용할지 알며, 최종 목표를 위해 겉보기에 "우회"하는 것처럼 보이는 중간 단계를 기꺼이 수행하고, 오류로부터 학습할 수 있게 한다.

Agentic RL의 목표는 LLM 에이전트에게 6가지 핵심 능력을 부여하는 것이다. 그림 11.2와 같다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-2.png" alt="" width="85%"/>
  <p>그림 11.2 Agentic RL의 6가지 핵심 능력</p>
</div>

**추론(Reasoning)**은 주어진 정보로부터 논리적으로 결론을 도출하는 과정으로, 에이전트의 핵심 능력이다. 전통적인 CoT 프롬프트 방법은 소수의 예시에 의존하여 일반화 능력이 제한적이며, SFT는 훈련 데이터의 추론 패턴을 모방할 뿐 혁신하기 어렵다. 강화학습의 장점은 시행착오를 통해 효과적인 추론 전략을 학습하고, 훈련 데이터에 없는 추론 경로를 발견하며, 언제 깊이 생각해야 하고 언제 빠르게 답할 수 있는지를 학습한다는 것이다. 추론 작업은 순차적 의사결정 문제로 모델링할 수 있다. 문제 $q$가 주어지면 에이전트는 추론 체인 $c = (c_1, c_2, ..., c_n)$과 최종 답 $a$를 생성해야 한다. 보상 함수는 보통 $r(q, c, a) = 1$ if $a = a^*$ else $0$으로 설계되며, 훈련 목표는 $\max_\theta \mathbb{E}_{q, (c,a) \sim \pi_\theta} [r(q, c, a)]$다. 이 방식을 통해 모델은 단순히 답을 암기하는 것이 아니라 고품질의 추론 체인을 생성하는 법을 학습한다.

**도구 사용(Tool Use)**은 에이전트가 외부 도구를 호출하여 작업을 완수하는 능력이다. 도구 사용 작업에서 행동 공간은 $a_t \in \{a_t^{\text{think}}, a_t^{\text{tool}}\}$로 확장된다. 여기서 $a_t^{\text{think}}$는 사고 과정을 생성하는 것이고, $a_t^{\text{tool}} = (\text{tool\_name}, \text{arguments})$는 도구를 호출하는 것이다. 강화학습을 통해 에이전트는 언제 도구를 사용해야 하는지, 어떤 도구를 선택해야 하는지, 여러 도구를 어떻게 조합해야 하는지를 학습한다. 예를 들어 수학 문제를 풀 때 에이전트는 언제 계산기를 사용하고, 언제 코드 인터프리터를 사용하며, 언제 직접 추론할지를 학습해야 한다.

**기억(Memory)**은 에이전트가 과거 정보를 유지하고 재활용하는 능력으로, 장기 작업에 매우 중요하다. LLM의 컨텍스트 창은 제한적이며 정적 검색 전략(RAG 등)은 작업 최적화가 불가능하다. 강화학습을 통해 에이전트는 기억 관리 전략을 학습한다: 어떤 정보를 기억할 가치가 있는지, 언제 기억을 업데이트해야 하는지, 언제 오래된 정보를 삭제해야 하는지를 결정한다. 이것은 인간의 작업 기억과 유사하다. 우리는 뇌 속의 정보를 능동적으로 관리하여 중요한 것을 유지하고 관련 없는 것을 잊어버린다.

**계획(Planning)**은 목표를 달성하기 위한 행동 시퀀스를 수립하는 능력이다. 전통적인 CoT는 선형적 사고로 되돌아갈 수 없으며, 프롬프트 엔지니어링은 정적 계획 템플릿을 사용하여 새로운 상황에 적응하기 어렵다. 강화학습을 통해 에이전트는 동적 계획을 학습한다: 시행착오를 통해 효과적인 행동 시퀀스를 발견하고, 단기와 장기 이익의 균형을 맞추는 법을 학습한다. 예를 들어 다단계 작업에서 에이전트는 먼저 정보 수집 같은 "우회"처럼 보이는 단계를 수행해야 최종적으로 작업을 완수할 수 있다.

**자기 개선(Self-Improvement)**은 에이전트가 자신의 출력을 돌아보고, 오류를 수정하며, 전략을 최적화하는 능력이다. 강화학습을 통해 에이전트는 자기 반성을 학습한다: 자신의 오류를 식별하고, 실패 원인을 분석하며, 전략을 조정한다. 이 능력 덕분에 에이전트는 인간의 개입 없이도 지속적으로 개선될 수 있으며, 이는 인간의 "실수로부터 배우기"와 유사하다.

**인식(Perception)**은 멀티모달 정보를 이해하는 능력이다. 예를 들어 강화학습은 시각적 추론 능력을 향상시키고, 모델이 시각 도구를 활용하며 시각적 계획을 수립하도록 학습시킬 수 있다. 이를 통해 에이전트는 텍스트뿐만 아니라 시각 세계도 이해하고 조작할 수 있다.

### 11.1.4 HelloAgents의 Agentic RL 설계

Agentic RL의 핵심 이념을 이해한 후, HelloAgents 프레임워크에서 이러한 능력들을 어떻게 구현할지 살펴보자.

기술 선택에 있어서 TRL(Transformer Reinforcement Learning) 프레임워크[9]를 통합하고, 모델로는 Qwen3-0.6B[10]을 선택했다. TRL은 Hugging Face의 강화학습 라이브러리로 성숙하고 안정적이며 기능이 완전하고 통합이 쉽다. Qwen3-0.6B는 알리클라우드의 소형 언어 모델로, 0.6B 파라미터는 일반 GPU에서도 훈련 가능하며 성능이 우수하고 오픈소스로 무료다.

HelloAgents의 Agentic RL 모듈은 4계층 아키텍처로 설계되어 있다. 그림 11.3과 같다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-3.png" alt="" width="85%"/>
  <p>그림 11.3 HelloAgents Agentic RL 아키텍처</p>
</div>

가장 아래 계층은 **데이터셋 계층**으로 `GSM8KDataset` 클래스, `create_sft_dataset()` 함수, `create_rl_dataset()` 함수를 포함하며 데이터 로딩과 형식 변환을 담당한다. 두 번째 계층은 **보상 함수 계층**으로 `MathRewardFunction` 기본 클래스, `AccuracyReward` 정확도 보상, `LengthPenaltyReward` 길이 패널티, `StepReward` 단계 보상, 그리고 편의 생성 함수 `create_*_reward()`를 포함하며 좋은 행동이 무엇인지를 정의한다. 세 번째 계층은 **훈련기 계층**으로 `SFTTrainerWrapper`와 `GRPOTrainerWrapper`를 포함하며 구체적인 훈련 로직과 LoRA 지원을 담당한다. 최상위 계층은 **통합 인터페이스 계층**으로 `RLTrainingTool` 통합 훈련 도구를 제공하며 4가지 작업을 지원한다: `action="train"`(모델 훈련), `action="load_dataset"`(데이터셋 로딩), `action="create_reward"`(보상 함수 생성), `action="evaluate"`(모델 평가).

### 11.1.5 빠른 시작 예시

심층 학습에 들어가기 전에 먼저 완전한 훈련 흐름을 빠르게 체험해보자. 이 장은 이론 부분이 상당히 많고 실습 시 조정해야 할 부분도 매우 번거롭기 때문에, 도구를 구성하는 것보다 적용하는 방법을 배우는 데 집중한다. 먼저 HelloAgents 프레임워크를 설치한다:

```bash
# HelloAgents 프레임워크 설치(11장 버전)
pip install "hello-agents[rl]==0.2.5"

# 또는 소스코드에서 설치
cd HelloAgents
pip install -e ".[rl]"
```

그 다음 빠른 훈련 예시를 실행한다:

```python
import sys
import json

from hello_agents.tools import RLTrainingTool

# RL 훈련 도구 생성
rl_tool = RLTrainingTool()

# 1. 빠른 테스트: SFT 훈련(10개 샘플, 1 에포크)
sft_result_str = rl_tool.run({
    "action": "train",
    "algorithm": "sft",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/quick_test_sft",
    "max_samples": 10,      # 10개 샘플로 빠른 테스트
    "num_epochs": 1,        # 1회만 훈련
    "batch_size": 2,
    "use_lora": True        # LoRA로 훈련 가속
})

sft_result = json.loads(sft_result_str)
print(f"\n✓ SFT 훈련 완료, 모델 저장 위치: {sft_result['output_dir']}")

# 2. GRPO 훈련(5개 샘플, 1 에포크)
grpo_result_str = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",  # 기본 모델 사용
    "output_dir": "./models/quick_test_grpo",
    "max_samples": 5,       # 5개 샘플로 빠른 테스트
    "num_epochs": 1,
    "batch_size": 2,        # num_generations(8)으로 나누어져야 함, 2 사용
    "use_lora": True
})

grpo_result = json.loads(grpo_result_str)
print(f"\n✓ GRPO 훈련 완료, 모델 저장 위치: {grpo_result['output_dir']}")

# 3. 모델 평가
eval_result_str = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/quick_test_grpo",
    "max_samples": 10,      # 10개 테스트 샘플로 평가
    "use_lora": True
})

eval_result = json.loads(eval_result_str)
print(f"\n✓ 평가 완료:")
print(f"  - 정확도: {eval_result['accuracy']}")
print(f"  - 평균 보상: {eval_result['average_reward']}")
print(f"  - 테스트 샘플 수: {eval_result['num_samples']}")

print("\n" + "=" * 50)
print("🎉 축하합니다! 첫 번째 Agentic RL 모델 훈련을 완료했습니다!")
print("=" * 50)
print(f"\n모델 경로:")
print(f"  SFT 모델: {sft_result['output_dir']}")
print(f"  GRPO 모델: {grpo_result['output_dir']}")
```

이 빠른 예시는 완전한 훈련 흐름을 보여준다: SFT 훈련으로 모델이 기본적인 추론 형식과 대화 패턴을 학습하고, GRPO 훈련으로 강화학습을 통해 추론 전략을 최적화하여 정확도를 높이며, 모델 평가로 테스트 세트에서 훈련 효과를 측정한다. 또한 실행 후 정확도가 매우 낮은 것은 정상적인 현상이다. 모델이 훈련 샘플의 0.7%만 보았고 단 한 번만 실행했기 때문이다.

## 11.2 데이터셋과 보상 함수

데이터셋과 보상 함수는 강화학습 훈련의 두 가지 핵심 기둥이다. 데이터셋은 에이전트가 학습해야 할 작업을 정의하고, 보상 함수는 좋은 행동이 무엇인지를 정의한다. 본 절에서는 훈련 데이터를 준비하고 보상 함수를 설계하는 방법을 학습한다.

### 11.2.1 GSM8K 수학 추론 데이터셋

수학 추론은 LLM의 추론 능력을 평가하기에 이상적인 작업이다. 첫째, 수학 문제는 명확한 정답이 있어 자동으로 평가할 수 있으며 인간 주석이나 복잡한 보상 모델이 필요 없다. 둘째, 수학 문제를 풀기 위해서는 문제를 분해하고 단계적으로 추론해야 하는데, 이것이 바로 다단계 추론의 전형적인 시나리오다. 셋째, 학습된 추론 능력은 다른 영역으로 전이될 수 있어 강한 일반화 능력을 가진다. 이에 비해 개방형 질의응답 작업(예: "프로그래밍을 어떻게 배울까요?")의 답변 품질은 객관적으로 평가하기 어렵고 대량의 인간 주석이 필요하다.

GSM8K(Grade School Math 8K)[4]는 고품질 초등학교 수학 응용 문제 데이터셋이다. 표 11.2에서 보듯이 이 데이터셋은 7,473개의 훈련 샘플과 1,319개의 테스트 샘플을 포함하며, 난이도는 초등학교 수학 수준(2-8학년)이고, 유형은 응용 문제로 답을 도출하기까지 2-8단계의 추론이 필요하다.

<div align="center">
  <p>표 11.2 GSM8K 데이터셋 통계</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-2.png" alt="" width="85%"/>
</div>

전형적인 GSM8K 문제를 살펴보자:

```
문제: Natalia sold clips to 48 of her friends in April, and then she sold half 
      as many clips in May. How many clips did Natalia sell altogether in April 
      and May?

답: Natalia sold 48/2 = <<48/2=24>>24 clips in May.
    Natalia sold 48+24 = <<48+24=72>>72 clips altogether in April and May.
    #### 72

최종 답: 72
```

이 문제는 두 단계의 추론이 필요하다: 먼저 5월에 판매한 수량(48의 절반)을 계산하고, 그 다음 합계(4월 + 5월)를 계산한다. 답에서 `<<48/2=24>>`는 중간 계산 단계의 표시이고, `#### 72`는 최종 답을 나타낸다.

GSM8K 데이터셋은 다양한 훈련 방법에 적합하도록 서로 다른 형식으로 변환되어야 한다. 그림 11.4와 같다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-4.png" alt="" width="85%"/>
  <p>그림 11.4 GSM8K 데이터 형식 변환</p>
</div>

원본 형식은 데이터셋에서 직접 가져온 것으로 질문(question)과 답변(answer, 풀이 단계 포함)을 포함하며, 인간이 읽기에 적합하다. SFT 형식은 지도 미세조정에 사용되며, 질문을 대화 형식의 프롬프트로 변환하고 완전한 풀이를 완성(completion)으로 사용한다. 예를 들면:

```python
{
    "prompt": "<|im_start|>user\nNatalia sold clips to 48 of her friends...<|im_end|>\n<|im_start|>assistant\n",
    "completion": "Let me solve this step by step.\n\nStep 1: ...\n\nFinal Answer: 72<|im_end|>"
}
```

핵심 포인트는 모델의 대화 템플릿(예: Qwen의 `<|im_start|>` 마커)을 사용하고, 프롬프트에는 사용자 질문이 포함되며, 완성에는 전체 풀이 과정과 답이 포함된다는 것이다. 이렇게 하면 모델이 출력 형식을 지정하는 방법, 단계적으로 추론하는 방법을 학습할 수 있다.

RL 형식은 강화학습에 사용되며, 질문과 정답만 제공하고 풀이 과정은 제공하지 않는다. 예를 들면:

```python
{
    "prompt": "<|im_start|>user\nNatalia sold clips to 48 of her friends...<|im_end|>\n<|im_start|>assistant\n",
    "ground_truth": "72"
}
```

핵심 포인트는 프롬프트가 SFT와 동일하지만, `ground_truth`에는 최종 답(보상 계산에 사용)만 포함되며 모델이 스스로 완전한 추론 과정을 생성해야 한다는 것이다. 이 설계는 모델이 단순히 답을 암기하는 것이 아니라 자율적인 추론을 학습하도록 강제한다.

표 11.3에서 보듯이 세 가지 형식은 각각 용도가 있다.

<div align="center">
  <p>표 11.3 데이터 형식 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-3.png" alt="" width="85%"/>
</div>

HelloAgents는 편리한 데이터셋 로딩 함수를 제공한다. 코드를 통해 데이터셋을 로드하고 살펴보자:

```python
from hello_agents.tools import RLTrainingTool
import json

# 도구 생성
rl_tool = RLTrainingTool()

# 1. SFT 형식 데이터셋 로드
sft_result = rl_tool.run({
    "action": "load_dataset",
    "format": "sft",
    "max_samples": 5  # 5개 샘플만 로드하여 확인
})
sft_data = json.loads(sft_result)

print(f"데이터셋 크기: {sft_data['dataset_size']}")
print(f"데이터 형식: {sft_data['format']}")
print(f"샘플 필드: {sft_data['sample_keys']}")

# 2. RL 형식 데이터셋 로드
rl_result = rl_tool.run({
    "action": "load_dataset",
    "format": "rl",
    "max_samples": 5
})
rl_data = json.loads(rl_result)

print(f"데이터셋 크기: {rl_data['dataset_size']}")
print(f"데이터 형식: {rl_data['format']}")
print(f"샘플 필드: {rl_data['sample_keys']}")
```

SFT 형식에는 완전한 풀이 과정이 포함되어 지도학습에 사용되고, RL 형식에는 최종 답만 포함되어 모델이 스스로 추론 과정을 생성해야 한다는 것을 확인할 수 있다. `max_samples` 파라미터는 로드되는 샘플 수를 제어하여 빠른 테스트를 편리하게 한다.

### 11.2.2 보상 함수 설계

보상 함수는 강화학습의 핵심으로, 좋은 행동이 무엇인지를 정의한다. 좋은 보상 함수는 에이전트가 올바른 전략을 학습하도록 유도할 수 있지만, 잘못된 보상 함수는 훈련 실패나 잘못된 행동을 학습하게 할 수 있다.

강화학습에서 보상 함수 $r(s, a)$ 또는 $r(s, a, s')$는 에이전트의 각 행동에 수치 보상을 부여한다. 에이전트의 목표는 누적 보상을 최대화하는 것이다:

$$
J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta} \left[\sum_{t=0}^{T} \gamma^t r(s_t, a_t)\right]
$$

수학 추론 작업의 경우 다음과 같이 단순화할 수 있다:

$$
r(q, a) = f(a, a^*)
$$

여기서 $q$는 문제, $a$는 모델이 생성한 답, $a^*$는 정답, $f$는 평가 함수다.

보상 함수의 설계는 훈련 효과에 직접 영향을 미친다. 좋은 보상 함수는 성공이 무엇인지를 명확히 정의하고, 경사 신호를 제공하며, 너무 큰 분산을 만들지 않고, 조정과 조합이 쉬워야 한다. 나쁜 보상 함수는 작업이 끝날 때만 보상을 주어 중간 단계에서 피드백이 없거나, 보상 해킹이 존재하여 에이전트가 "치팅" 방식으로 높은 보상을 얻거나, 여러 목표가 서로 충돌하거나, 분산이 너무 커서 훈련이 수렴하지 않을 수 있다.

HelloAgents는 세 가지 내장 보상 함수를 제공하며, 단독으로 사용하거나 조합하여 사용할 수 있다. 그림 11.5와 같다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-5.png" alt="" width="85%"/>
  <p>그림 11.5 보상 함수 설계</p>
</div>

**(1) 정확도 보상**

정확도 보상(AccuracyReward)은 가장 기본적인 보상 함수로, 오직 답이 맞는지만 신경 쓴다. 수학적 정의는 다음과 같다:

$$
r_{\text{acc}}(a, a^*) = \begin{cases}
1 & \text{if } a = a^* \\
0 & \text{otherwise}
\end{cases}
$$

여기서 $a$는 모델이 생성한 답, $a^*$는 정답이다. 이것은 이진 보상 함수로, 답이 맞으면 1점, 틀리면 0점이다.

구현 시 답 추출과 비교를 처리해야 한다. 모델의 출력에는 많은 텍스트가 포함될 수 있으므로 최종 답을 추출해야 한다. 일반적인 추출 방법으로는 "Final Answer:" 뒤의 숫자 찾기, "####" 마커 뒤의 숫자 찾기, 정규식으로 마지막 숫자 추출하기 등이 있다. 답 비교 시 수치 정밀도(예: 72.0과 72는 동일하게 처리), 단위 변환(예: 1000과 1k), 형식 차이(예: "72"와 "seventy-two")를 처리해야 한다.

사용 예시:

```python
from hello_agents.tools import RLTrainingTool
import json
rl_tool = RLTrainingTool()

# 정확도 보상 함수 생성
reward_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "accuracy"
})
reward_data = json.loads(reward_result)

print(f"보상 유형: {reward_data['reward_type']}")
print(f"설명: {reward_data['description']}")

# 주의: RLTrainingTool의 create_reward 작업은 설정 정보를 반환하며,
# 실제 보상 함수는 훈련 시 자동으로 생성되고 사용됨
```

출력:

```json
예측: 72, 실제: 72, 보상: 1.0
예측: 72.0, 실제: 72, 보상: 1.0
예측: 73, 실제: 72, 보상: 0.0
```

정확도 보상의 장점은 단순하고 직관적이며 이해와 구현이 쉽고 명확한 정답이 있는 작업에 적합하다는 것이다. 단점은 보상이 희소하여 답이 완전히 맞아야만 보상이 있고, "거의 맞음"과 "완전히 틀림"을 구분할 수 없으며, 훈련 초기에 효과적인 피드백이 부족할 수 있다는 것이다.

**(2) 길이 패널티**

길이 패널티(LengthPenaltyReward)는 모델이 간결한 답변을 생성하도록 장려하여 장황하고 불필요한 내용을 피하게 한다. 수학적 정의는 다음과 같다:

$$
r_{\text{length}}(a, a^*, l) = r_{\text{acc}}(a, a^*) - \alpha \cdot \max(0, l - l_{\text{target}})
$$

여기서 $l$은 생성된 텍스트의 길이(문자 수 또는 토큰 수), $l_{\text{target}}$은 목표 길이, $\alpha$는 패널티 계수(기본값 0.001)다. 답이 맞을 때만 길이 패널티를 적용하여 모델이 패널티를 줄이기 위해 짧은 오답을 생성하는 것을 방지한다.

설계 논리: 답이 틀리면 보상은 0(길이에 관계없이), 답이 맞고 길이가 합리적이면 보상은 1, 답이 맞지만 너무 길면 보상은 $1 - \alpha \cdot (l - l_{\text{target}})$이다. 예를 들어 목표 길이가 200자이고 실제 길이가 500자이며 패널티 계수가 0.001이라면, 보상은 $1 - 0.001 \times (500 - 200) = 0.7$이다.

사용 예시:

```python
# 길이 패널티 보상 함수 생성
reward_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "length_penalty",
    "max_length": 1024,      # 최대 길이
    "penalty_weight": 0.001  # 패널티 가중치
})
reward_data = json.loads(reward_result)

print(f"보상 유형: {reward_data['reward_type']}")
print(f"설명: {reward_data['description']}")
print(f"최대 길이: {reward_data['max_length']}")
print(f"패널티 가중치: {reward_data['penalty_weight']}")
```

출력:

```
예측: 72, 실제: 72, 길이: 50, 보상: 1.000
예측: 72, 실제: 72, 길이: 200, 보상: 1.000
예측: 72, 실제: 72, 길이: 500, 보상: 0.700
예측: 73, 실제: 72, 길이: 50, 보상: 0.000
```

길이 패널티의 장점은 간결한 표현을 장려하고, 모델이 불필요한 내용을 생성하는 것을 막으며, 추론 비용을 제어할 수 있다는 것이다(더 짧은 출력은 더 적은 토큰 소비를 의미한다). 단점은 상세한 추론을 억제할 수 있고, 패널티 계수를 신중하게 조정해야 하며, 다른 작업마다 최적 길이가 크게 다를 수 있다는 것이다.

**(3) 단계 보상**

단계 보상(StepReward)은 모델이 명확한 추론 단계를 생성하도록 장려하여 해석 가능성을 높인다. 수학적 정의는 다음과 같다:

$$
r_{\text{step}}(a, a^*, s) = r_{\text{acc}}(a, a^*) + \beta \cdot s
$$

여기서 $s$는 감지된 추론 단계 수, $\beta$는 단계 보상 계수(기본값 0.1)다. 마찬가지로 답이 맞을 때만 단계 보상이 주어진다.

단계 감지 방법으로는 "Step 1:", "Step 2:" 등의 마커 찾기, 줄바꿈 수 세기, 정규식으로 추론 패턴 매칭하기 등이 있다. 예를 들어 3개의 명확한 단계를 포함한 정답의 보상은 $1 + 0.1 \times 3 = 1.3$이다.

사용 예시:

```python
# 단계 보상 함수 생성
reward_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "step",
    "step_bonus": 0.1  # 단계마다 0.1 보상
})
reward_data = json.loads(reward_result)

print(f"보상 유형: {reward_data['reward_type']}")
print(f"설명: {reward_data['description']}")
print(f"단계 보상: {reward_data['step_bonus']}")
```

출력:

```
예측: 72, 실제: 72, 단계: 0, 보상: 1.00
예측: 72, 실제: 72, 단계: 2, 보상: 1.20
예측: 72, 실제: 72, 단계: 5, 보상: 1.50
예측: 73, 실제: 72, 단계: 5, 보상: 0.00
```

단계 보상의 장점은 해석 가능한 추론을 장려하고, 생성된 답변을 더 쉽게 검증하고 디버깅할 수 있으며, 모델이 체계적인 사고 방식을 학습하는 데 도움이 된다는 것이다. 단점은 더 많은 보상을 얻기 위해 모델이 불필요한 단계를 생성할 수 있고, 단계 수와 답변 품질의 균형을 조정해야 하며, 단계 감지가 정확하지 않을 수 있다는 것이다.

실제 응용에서는 보통 여러 보상 함수를 조합하여 서로 다른 목표들의 균형을 맞춘다. 일반적인 조합 전략으로는 다음이 있다:

**정확도 + 길이 패널티**: 간결하고 정확한 답을 장려하며, 대화 시스템과 질의응답 시스템에 적합하다. 공식은:

$$
r = r_{\text{acc}} - \alpha \cdot \max(0, l - l_{\text{target}})
$$

**정확도 + 단계 보상**: 상세한 추론 과정을 장려하며, 교육 시나리오와 해석 가능한 AI에 적합하다. 공식은:

$$
r = r_{\text{acc}} + \beta \cdot s
$$

**세 가지 균형**: 답변 품질, 간결성, 해석 가능성을 종합적으로 최적화한다. 공식은:

$$
r = r_{\text{acc}} - \alpha \cdot \max(0, l - l_{\text{target}}) + \beta \cdot s
$$

가중치 $\alpha$와 $\beta$를 신중하게 조정하여 어느 한 목표가 지나치게 지배적이 되지 않도록 해야 한다.

사용 예시:

```python
# 조합 보상 함수: 정확도 + 길이 패널티 + 단계 보상
# 주의: RLTrainingTool은 현재 단일 보상 유형을 지원함
# 조합 보상은 훈련 설정의 reward_fn 파라미터로 지정해야 함
# 여기서는 다른 유형의 보상 함수 설정 방법을 보여줌

# 정확도 보상
accuracy_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "accuracy"
})
print("정확도 보상:", json.loads(accuracy_result)['description'])

# 길이 패널티 보상
length_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "length_penalty",
    "max_length": 1024,
    "penalty_weight": 0.001
})
print("길이 패널티 보상:", json.loads(length_result)['description'])

# 단계 보상
step_result = rl_tool.run({
    "action": "create_reward",
    "reward_type": "step",
    "step_bonus": 0.1
})
print("단계 보상:", json.loads(step_result)['description'])
```

출력:

```
조합 보상: 1.200
  - 정확도: 1.0
  - 길이 패널티: -0.100
  - 단계 보상: +0.3
```

표 11.4에서 보듯이 서로 다른 보상 함수는 서로 다른 응용 시나리오에 적합하다.

<div align="center">
  <p>표 11.4 보상 함수 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-4.png" alt="" width="85%"/>
</div>

### 11.2.3 사용자 정의 데이터셋과 보상 함수

HelloAgents가 GSM8K 데이터셋과 일반적인 보상 함수를 제공하지만, 실제 응용에서는 자신만의 데이터셋을 사용하거나 특정 보상 함수를 설계해야 할 수 있다. 본 절에서는 프레임워크를 확장하는 방법을 소개한다.

사용자 정의 데이터셋을 사용하기 전에 두 가지 훈련 형식의 데이터 요구 사항을 이해해야 한다:

**SFT 형식**: 지도 미세조정에 사용되며 다음 필드가 필요하다:
- `prompt`: 입력 프롬프트(system과 user 메시지 포함)
- `completion`: 기대되는 출력
- `text`: 완전한 대화 텍스트(선택 사항)

**RL 형식**: 강화학습에 사용되며 다음 필드가 필요하다:
- `question`: 원본 질문
- `prompt`: 입력 프롬프트(system과 user 메시지 포함)
- `ground_truth`: 정답
- `full_answer`: 완전한 답변(추론 과정 포함)

**(1) format_math_dataset으로 변환**

가장 간단한 방법은 `question`과 `answer` 필드를 포함하는 원시 데이터를 준비한 다음, `format_math_dataset()` 함수를 사용하여 자동 변환하는 것이다:

```python
from datasets import Dataset
from hello_agents.rl import format_math_dataset

# 1. 원시 데이터 준비
custom_data = [
    {
        "question": "What is 2+2?",
        "answer": "2+2=4. #### 4"
    },
    {
        "question": "What is 5*3?",
        "answer": "5*3=15. #### 15"
    },
    {
        "question": "What is 10+7?",
        "answer": "10+7=17. #### 17"
    }
]

# 2. Dataset 객체로 변환
raw_dataset = Dataset.from_list(custom_data)

# 3. SFT 형식으로 변환
sft_dataset = format_math_dataset(
    dataset=raw_dataset,
    format_type="sft",
    model_name="Qwen/Qwen3-0.6B"
)
print(f"SFT 데이터셋: {len(sft_dataset)}개 샘플")
print(f"필드: {sft_dataset.column_names}")

# 4. RL 형식으로 변환
rl_dataset = format_math_dataset(
    dataset=raw_dataset,
    format_type="rl",
    model_name="Qwen/Qwen3-0.6B"
)
print(f"RL 데이터셋: {len(rl_dataset)}개 샘플")
print(f"필드: {rl_dataset.column_names}")
```

**(2) 사용자 정의 데이터셋을 직접 전달**

RLTrainingTool 사용 시 `custom_dataset` 파라미터를 통해 사용자 정의 데이터셋을 직접 전달할 수 있다:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# SFT 훈련
result = rl_tool.run({
    "action": "train",
    "algorithm": "sft",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/custom_sft",
    "num_epochs": 3,
    "batch_size": 4,
    "use_lora": True,
    "custom_dataset": sft_dataset  # 사용자 정의 데이터셋 직접 전달
})

# GRPO 훈련
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/custom_grpo",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
    "custom_dataset": rl_dataset  # 사용자 정의 데이터셋 직접 전달
})
```

**(3) 사용자 정의 데이터셋 등록(권장)**

여러 번 사용해야 하는 데이터셋의 경우 등록 방식을 권장한다:

```python
# 1. 데이터셋 등록
rl_tool.register_dataset("my_math_dataset", rl_dataset)

# 2. 등록된 데이터셋 사용
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "dataset": "my_math_dataset",  # 등록된 데이터셋 이름 사용
    "output_dir": "./models/custom_grpo",
    "num_epochs": 2,
    "use_lora": True
})
```

보상 함수는 모델이 생성한 답변의 품질을 평가하는 데 사용된다. 사용자 정의 보상 함수는 다음 시그니처를 따라야 한다:

```python
from typing import List
import re

def custom_reward_function(
    completions: List[str],
    **kwargs
) -> List[float]:
    """
    사용자 정의 보상 함수

    Args:
        completions: 모델이 생성한 완성 텍스트 목록
        **kwargs: 기타 파라미터, 일반적으로 다음을 포함:
            - ground_truth: 정답 목록
            - 기타 데이터셋 필드

    Returns:
        보상값 목록(각 값은 0.0~1.0 사이)
    """
    ground_truths = kwargs.get("ground_truth", [])
    rewards = []

    for completion, truth in zip(completions, ground_truths):
        reward = 0.0

        # 답 추출
        numbers = re.findall(r'-?\d+\.?\d*', completion)
        if numbers:
            try:
                pred = float(numbers[-1])
                truth_num = float(truth)
                error = abs(pred - truth_num)

                # 오류에 따라 다른 보상 부여
                if error < 0.01:
                    reward = 1.0  # 완전히 정확
                elif error < 1.0:
                    reward = 0.8  # 매우 가까움
                elif error < 5.0:
                    reward = 0.5  # 가까움

                # 추가 보상: 추론 단계 표시 장려
                if "step" in completion.lower() or "=" in completion:
                    reward += 0.1

            except ValueError:
                reward = 0.0

        rewards.append(min(reward, 1.0))  # 최대값 1.0으로 제한

    return rewards
```

사용자 정의 보상 함수를 사용하는 두 가지 방법이 있다:

**(1) 직접 전달**

```python
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/custom_grpo",
    "custom_dataset": rl_dataset,
    "custom_reward": custom_reward_function  # 보상 함수 직접 전달
})
```

**(2) 등록하여 사용(권장)**

```python
# 1. 보상 함수 등록
rl_tool.register_reward_function("my_reward", custom_reward_function)

# 2. 등록된 보상 함수 사용
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "dataset": "my_math_dataset",
    "output_dir": "./models/custom_grpo"
    # 보상 함수는 dataset과 동일한 이름으로 등록된 함수를 자동 사용
})
```

완전한 사용자 정의 데이터셋과 보상 함수 예시:

```python
from datasets import Dataset
from hello_agents.tools import RLTrainingTool
from hello_agents.rl import format_math_dataset
import re
from typing import List

# 1. 사용자 정의 데이터 준비
custom_data = [
    {"question": "What is 2+2?", "answer": "2+2=4. #### 4"},
    {"question": "What is 5+3?", "answer": "5+3=8. #### 8"},
    {"question": "What is 10+7?", "answer": "10+7=17. #### 17"}
]

# 2. 훈련 형식으로 변환
raw_dataset = Dataset.from_list(custom_data)
rl_dataset = format_math_dataset(raw_dataset, format_type="rl")

# 3. 사용자 정의 보상 함수 정의
def tolerant_reward(completions: List[str], **kwargs) -> List[float]:
    """허용 오차가 있는 보상 함수"""
    ground_truths = kwargs.get("ground_truth", [])
    rewards = []

    for completion, truth in zip(completions, ground_truths):
        numbers = re.findall(r'-?\d+\.?\d*', completion)
        if numbers:
            try:
                pred = float(numbers[-1])
                truth_num = float(truth)
                error = abs(pred - truth_num)

                if error < 0.01:
                    reward = 1.0
                elif error < 5.0:
                    reward = 0.5
                else:
                    reward = 0.0
            except ValueError:
                reward = 0.0
        else:
            reward = 0.0

        rewards.append(reward)

    return rewards

# 4. 도구 생성 및 등록
rl_tool = RLTrainingTool()
rl_tool.register_dataset("my_dataset", rl_dataset)
rl_tool.register_reward_function("my_dataset", tolerant_reward)

# 5. 훈련
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "dataset": "my_dataset",
    "output_dir": "./models/custom_grpo",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True
})
```

## 11.3 SFT 훈련

지도 미세조정(Supervised Fine-Tuning, SFT)은 강화학습 훈련의 첫 번째 단계이자 가장 중요한 기초다. SFT는 모델이 작업의 기본 형식, 대화 패턴, 초보적인 추론 능력을 학습하도록 한다. SFT 기초 없이 직접 강화학습을 수행하면 실패하는 경우가 많다. 모델이 기본적인 출력 형식조차 모르기 때문이다.

### 11.3.1 왜 SFT가 필요한가

강화학습을 시작하기 전에 먼저 SFT 훈련을 수행해야 한다. 사전 훈련된 모델은 강력한 언어 능력을 갖추고 있지만 특정 작업을 어떻게 완수해야 하는지 모르기 때문이다. 사전 훈련 모델의 훈련 목표는 다음 단어를 예측하는 것이지 수학 문제를 풀거나 도구를 사용하는 것이 아니다. 사전 훈련 모델의 출력 형식은 자유 텍스트이지만, 우리는 구조화된 출력(예: "Step 1: ..., Step 2: ..., Final Answer: ...")이 필요하다. 사전 훈련 모델은 작업 관련 데이터를 본 적이 없어 무엇이 "좋은" 추론 과정인지 모른다.

SFT의 역할은 모델에게 작업의 기본 규칙을 가르치는 것이다. 첫째, 출력 형식을 학습시켜 모델이 답을 어떻게 구성할지(예: "Step 1", "Final Answer" 등의 마커 사용) 알도록 한다. 둘째, 추론 패턴을 학습시켜 예시를 통해 문제를 어떻게 분해하고 단계적으로 추론할지를 배운다. 셋째, 기본 능력을 확립하여 이후 강화학습의 합리적인 출발점을 제공한다. 넷째, 탐색 공간을 줄여 강화학습이 처음부터 시작할 필요 없이 SFT 기초 위에서 최적화할 수 있도록 한다.

비교 실험을 통해 SFT의 중요성을 이해해보자. 사전 훈련 모델을 바로 사용하여 GSM8K 문제를 풀려고 하면:

```python
from transformers import AutoTokenizer, AutoModelForCausalLM

# 사전 훈련 모델 로드
model_name = "Qwen/Qwen3-0.6B"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name)

# 테스트 문제
question = """Natalia sold clips to 48 of her friends in April, and then she sold half as many clips in May. How many clips did Natalia sell altogether in April and May?"""

# 입력 구성
prompt = f"<|im_start|>user\n{question}<|im_end|>\n<|im_start|>assistant\n"
inputs = tokenizer(prompt, return_tensors="pt")

# 답변 생성
outputs = model.generate(**inputs, max_new_tokens=200)
response = tokenizer.decode(outputs[0], skip_special_tokens=False)

print("사전 훈련 모델의 답변:")
print(response)
```

사전 훈련 모델의 출력은 다음과 같을 수 있다:

```bash
<|im_start|>user
Natalia sold clips to 48 of her friends in April, and then she sold half as many clips in May. How many clips did Natalia sell altogether in April and May?<|im_end|>
<|im_start|>assistant
I can help you with that. Let me think about this problem. Natalia sold clips to her friends. In April she sold 48 clips. In May she sold half as many. So we need to find out how many she sold in May and then add them together. But I'm not sure how to calculate half of 48. Maybe I should use a calculator? Or maybe I can just estimate...
```

사전 훈련 모델은 문제를 이해했지만 출력이 매우 길고 구조가 없으며, 명확한 답이 없고 추론 과정이 혼란스러운 것을 볼 수 있다. 이런 출력은 강화학습에 사용할 수 없다. 답을 추출할 수 없고, 품질을 평가할 수 없으며, 효과적인 보상 신호를 제공할 수 없기 때문이다.

이번에는 SFT 후 모델을 살펴보자:

```python
# SFT 모델 로드(이미 훈련되었다고 가정)
sft_model_path = "./models/sft_model"
sft_model = AutoModelForCausalLM.from_pretrained(sft_model_path)

# 동일한 문제 사용
outputs = sft_model.generate(**inputs, max_new_tokens=200)
response = tokenizer.decode(outputs[0], skip_special_tokens=False)

print("SFT 모델의 답변:")
print(response)
```

SFT 모델의 출력:

```bash
<|im_start|>user
Natalia sold clips to 48 of her friends in April, and then she sold half as many clips in May. How many clips did Natalia sell altogether in April and May?<|im_end|>
<|im_start|>assistant
Let me solve this step by step.

Step 1: Calculate clips sold in May
Natalia sold half as many clips in May as in April.
Clips in May = 48 / 2 = 24

Step 2: Calculate total clips
Total = April + May = 48 + 24 = 72

Final Answer: 72<|im_end|>
```

SFT 모델의 출력은 구조가 명확하고("Step 1", "Step 2", "Final Answer" 마커 사용), 추론이 정확하며, 답이 명확하고 형식이 통일되어 있음을 볼 수 있다. 이런 출력은 강화학습에 사용할 수 있다. 답을 추출하고, 보상을 계산하고, 전략을 최적화할 수 있기 때문이다.

그림 11.6에서 보듯이 SFT는 사전 훈련 모델에서 강화학습으로 이어지는 다리 역할을 한다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-6.png" alt="" width="85%"/>
  <p>그림 11.6 훈련 흐름에서 SFT의 역할</p>
</div>

### 11.3.2 LoRA: 파라미터 효율적 미세조정

전체 모델을 직접 미세조정하려면 많은 계산 자원과 GPU 메모리가 필요하다. Qwen3-0.6B(0.6B 파라미터)의 경우 풀 파라미터 미세조정에 약 12GB GPU 메모리(FP16) 또는 24GB GPU 메모리(FP32)가 필요하다. 더 큰 모델(예: 7B, 13B)의 경우 소비자용 GPU에서 풀 파라미터 미세조정은 거의 불가능하다.

LoRA(Low-Rank Adaptation)[3]는 파라미터 효율적 미세조정 방법으로, 소량의 추가 파라미터만 훈련하고 원래 모델 파라미터는 동결 상태로 유지한다. LoRA의 핵심 아이디어는 모델 미세조정 시 파라미터 변화를 저순위 행렬로 표현할 수 있다는 것이다.

원래 모델의 가중치 행렬을 $W \in \mathbb{R}^{d \times k}$라 하면, 미세조정 후의 가중치는 $W' = W + \Delta W$다. LoRA는 $\Delta W$가 두 저순위 행렬의 곱으로 분해될 수 있다고 가정한다:

$$
\Delta W = BA
$$

여기서 $B \in \mathbb{R}^{d \times r}$, $A \in \mathbb{R}^{r \times k}$, $r \ll \min(d, k)$는 순위(rank)다.

순전파 시 출력은:

$$
h = Wx + \Delta Wx = Wx + BAx
$$

원래 모델 파라미터 $W$는 동결 상태를 유지하고, $B$와 $A$만 훈련한다.

파라미터 수 비교: 원래 모델 파라미터 수는 $d \times k$이고, LoRA 파라미터 수는 $d \times r + r \times k = r(d + k)$다. $r \ll \min(d, k)$일 때 LoRA 파라미터 수는 원래 모델보다 훨씬 적다. 예를 들어 $d=4096, k=4096, r=8$인 경우, 원래 모델 파라미터 수는 $4096 \times 4096 = 16,777,216$이고, LoRA 파라미터 수는 $8 \times (4096 + 4096) = 65,536$으로, 파라미터 수가 256배 감소한다!

따라서 LoRA의 장점을 정리하면: GPU 메모리 사용량이 크게 줄어들고, 훈련 속도가 빨라지며, 배포가 쉽고, 과적합을 방지한다. 다만 훈련 효과는 일반적으로 풀 파라미터 조정보다 다소 떨어진다.

표 11.5에서 보듯이 다양한 모델 규모에서의 LoRA 효과를 비교한다.

<div align="center">
  <p>표 11.5 LoRA vs 풀 파라미터 미세조정 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-5.png" alt="" width="85%"/>
</div>

LoRA의 주요 하이퍼파라미터로는 순위(rank, r), 알파($\alpha$), 대상 모듈(target_modules)이 있다. 순위는 LoRA 행렬의 순위를 제어하며, 클수록 표현 능력이 강해지지만 파라미터 수도 많아진다. 전형적인 값은 4-64이며 기본값은 8이다. 알파는 LoRA의 스케일링 인수로, 실제 업데이트는 $\Delta W = \frac{\alpha}{r} BA$이며 LoRA의 영향 강도를 제어한다. 전형적인 값은 rank와 같다. 대상 모듈은 어떤 계층에 LoRA를 적용할지 지정하는데, 일반적으로 어텐션 계층(q_proj, k_proj, v_proj, o_proj)을 선택하며 MLP 계층(gate_proj, up_proj, down_proj)도 포함할 수 있다.

### 11.3.3 SFT 훈련 실습

이제 HelloAgents를 사용하여 SFT 훈련을 진행해보자. 완전한 훈련 흐름은 다음과 같다: 데이터셋 준비, LoRA 설정, 훈련 파라미터 설정, 훈련 시작, 모델 저장.

기본 훈련 예시:

```python
from hello_agents.tools import RLTrainingTool

# 훈련 도구 생성
rl_tool = RLTrainingTool()

# SFT 훈련
result = rl_tool.run({
    # 훈련 설정
    "action": "train",
    "algorithm": "sft",
    
    # 모델 설정
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/sft_model",
    
    # 데이터 설정
    "max_samples": 100,     # 100개 샘플로 빠른 테스트
    
    # 훈련 파라미터
    "num_epochs": 3,        # 3회 훈련
    "batch_size": 4,        # 배치 크기
    "learning_rate": 5e-5,  # 학습률
    
    # LoRA 설정
    "use_lora": True,       # LoRA 사용
    "lora_rank": 8,         # LoRA 순위
    "lora_alpha": 16,       # LoRA 알파
})

print(f"\n✓ 훈련 완료!")
print(f"  - 모델 저장 경로: {result['model_path']}")
print(f"  - 훈련 샘플 수: {result['num_samples']}")
print(f"  - 훈련 에포크 수: {result['num_epochs']}")
print(f"  - 최종 손실: {result['final_loss']:.4f}")
```

훈련 과정에서 손실이 점차 감소하면 모델이 학습 중임을 나타낸다.

**(1) 훈련 파라미터 상세 설명**

각 훈련 파라미터의 의미와 튜닝 권장 사항을 자세히 살펴보자.

**데이터 파라미터**:

- `max_samples`: 사용하는 훈련 샘플 수. 빠른 테스트 시 100-1000개, 전체 훈련 시 전체 데이터(7473개 샘플) 사용을 권장한다. 더 많은 데이터는 보통 더 좋은 효과를 가져오지만 훈련 시간도 길어진다.
- `split`: 데이터셋 분할, 기본값은 "train". "train[:1000]"으로 설정하면 앞 1000개 샘플만 사용한다.

**훈련 파라미터**:

- `num_epochs`: 훈련 에포크 수. 1 에포크는 전체 데이터셋을 한 번 통과하는 것이다. 너무 적으면(1-2 에포크) 과소적합, 너무 많으면(>10 에포크) 과적합될 수 있다. 3 에포크부터 시작하여 손실 곡선을 보며 조정하는 것을 권장한다.
- `batch_size`: 한 번의 업데이트에 사용하는 샘플 수. 클수록 훈련이 안정적이지만 GPU 메모리 사용량이 높아진다. GPU 메모리에 따라 조정 권장: 4GB GPU 메모리에는 batch_size=1-2, 8GB에는 4-8, 16GB에는 8-16.
- `learning_rate`: 학습률, 파라미터 업데이트 보폭을 제어한다. 너무 작으면(1e-6) 수렴이 느리고, 너무 크면(1e-3) 수렴하지 않을 수 있다. SFT에는 5e-5 권장, LoRA는 약간 더 크게(1e-4) 설정 가능하다.

**LoRA 파라미터**:

- `use_lora`: LoRA 사용 여부. 충분한 GPU 메모리가 없다면 항상 사용하는 것을 권장한다.
- `lora_rank`: LoRA 순위, 표현 능력 제어. 4-8은 소규모 작업에, 16-32는 복잡한 작업에, 64는 대규모 미세조정에 적합하다.
- `lora_alpha`: LoRA 스케일링 인수, 보통 rank의 2배로 설정. rank=8이면 alpha=16, rank=16이면 alpha=32.

**최적화기 파라미터**:

- `optimizer`: 최적화기 유형, 기본값은 "adamw". AdamW가 가장 일반적으로 선택되며, "sgd"나 "adafactor" 등도 시도해볼 수 있다.
- `weight_decay`: 가중치 감쇠, 과적합 방지. 기본값은 0.01, 0.001-0.1 범위로 시도 가능하다.
- `warmup_ratio`: 학습률 워밍업 비율. 처음 warmup_ratio만큼의 단계에서 학습률이 선형적으로 증가한 후 선형적으로 감소한다. 기본값은 0.1(처음 10% 단계에서 워밍업).

**(2) 완전한 훈련 예시**

전체 데이터와 최선의 방법으로 완전한 SFT 훈련을 진행해보자:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# 완전한 SFT 훈련
result = rl_tool.run({
    "action": "train",
    "algorithm": "sft",

    # 모델 설정
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/sft_full",

    # 데이터 설정
    "max_samples": None,    # 전체 데이터 사용(7473개 샘플)

    # 훈련 파라미터
    "num_epochs": 3,
    "batch_size": 8,
    "learning_rate": 5e-5,
    "warmup_ratio": 0.1,
    "weight_decay": 0.01,

    # LoRA 설정
    "use_lora": True,
    "lora_rank": 16,        # 더 큰 rank 사용
    "lora_alpha": 32,
    "lora_target_modules": ["q_proj", "k_proj", "v_proj", "o_proj"],

    # 기타 설정
    "save_steps": 500,      # 500 스텝마다 저장
    "logging_steps": 100,   # 100 스텝마다 로깅
    "eval_steps": 500,      # 500 스텝마다 평가
})

print(f"훈련 완료! 모델 저장 위치: {result['model_path']}")
```

이 설정은 8GB GPU 메모리에서 훈련에 적합하며, 예상 소요 시간은 30-60분이다.

**(3) 훈련 모니터링 및 디버깅**

훈련 과정에서 세 가지 핵심 지표를 모니터링해야 한다. 손실(Loss)은 점차 감소해야 하는데, 감소하지 않으면 학습률이 너무 작거나 데이터에 문제가 있을 수 있고, 감소 후 다시 증가하면 학습률이 너무 크거나 과적합이 발생한 것일 수 있다. 그래디언트 노름(Gradient Norm)은 0.1-10의 합리적인 범위 내에 있어야 하는데, 너무 크면(>100) 그래디언트 폭발이 발생한 것으로 학습률을 낮춰야 하고, 너무 작으면(<0.01) 그래디언트 소실이 발생한 것으로 모델 설정을 확인해야 한다. 학습률(Learning Rate)은 워밍업 전략에 따라 변해야 하는데, 처음 10% 단계에서 선형 증가 후 0까지 선형 감소한다.

훈련 중 흔히 발생하는 문제와 해결 방안: GPU 메모리 부족 시 batch_size나 max_length를 줄이거나 그래디언트 누적 또는 더 작은 모델을 사용한다. 훈련 속도가 느리면 batch_size를 늘리거나 로깅 빈도를 줄이거나 혼합 정밀도 훈련을 사용한다. 손실이 감소하지 않으면 학습률을 높이거나 데이터 형식을 확인하거나 훈련 에포크를 늘린다. 과적합 시 weight_decay를 늘리거나 훈련 에포크를 줄이거나 더 많은 데이터를 사용한다.

### 11.3.4 모델 평가

훈련 완료 후 모델의 효과를 평가해야 한다. 평가 지표로는 다음이 있다:

- **정확도(Accuracy)**: 답이 완전히 맞는 비율로, 가장 직접적인 지표다. 범위는 0-1이며 높을수록 좋다.

- **평균 보상(Average Reward)**: 모든 샘플의 평균 보상으로, 정확도, 길이, 단계 등을 종합적으로 고려한다. 범위는 보상 함수 설계에 따라 달라진다.

- **추론 품질(Reasoning Quality)**: 추론 과정의 명확성과 논리성으로, 인간 평가나 특수 평가 모델이 필요하다.

HelloAgents를 사용하여 모델을 평가한다:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# SFT 모델 평가
eval_result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/sft_full",
    "max_samples": 100,     # 100개 테스트 샘플로 평가
    "use_lora": True,
})

eval_data = json.loads(eval_result)
print(f"\n평가 결과:")
print(f"  - 정확도: {eval_data['accuracy']}")
print(f"  - 평균 보상: {eval_data['average_reward']}")
print(f"  - 테스트 샘플 수: {eval_data['num_samples']}")
```

Qwen3-0.6B 같은 소형 모델의 경우 SFT 후 GSM8K에서 40-50%의 정확도를 달성하는 것은 정상적이다. 강화학습을 통해 60-70%까지 향상시킬 수 있다.

SFT의 효과를 더 잘 이해하기 위해 서로 다른 단계의 모델을 비교할 수 있다:

```python
# 사전 훈련 모델 평가(SFT 전)
base_result = rl_tool.run({
    "action": "evaluate",
    "model_path": "Qwen/Qwen3-0.6B",
    "max_samples": 100,
    "use_lora": False,
})
base_data = json.loads(base_result)

# SFT 모델 평가
sft_result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/sft_full",
    "max_samples": 100,
    "use_lora": True,
})
sft_data = json.loads(sft_result)

# 결과 비교
print("모델 비교:")
print(f"사전 훈련 모델 정확도: {base_data['accuracy']}")
print(f"SFT 모델 정확도: {sft_data['accuracy']}")
```

본 절에서 SFT의 중요성(형식 학습, 기준 확립), LoRA 원리(저순위 분해, 파라미터 효율), SFT 훈련 실습(파라미터 설정, 훈련 모니터링), 모델 평가(정확도, 비교 분석)를 학습했다.

## 11.4 GRPO 훈련

SFT 훈련을 마치면 구조화된 답변을 생성할 수 있는 모델을 얻게 된다. 하지만 SFT 모델은 훈련 데이터의 추론 과정을 "모방"하는 것을 학습했을 뿐, 진정으로 "사고"하는 것을 학습한 것이 아니다. 강화학습은 모델이 시행착오를 통해 추론 전략을 최적화하여, 훈련 데이터의 품질을 뛰어넘을 수 있도록 한다.

### 11.4.1 PPO에서 GRPO로

강화학습 분야에서 PPO(Proximal Policy Optimization)[1]는 가장 고전적인 알고리즘 중 하나다. PPO는 정책 업데이트 폭을 제한함으로써 훈련의 안정성을 보장한다. 하지만 PPO는 LLM 훈련에서 몇 가지 문제가 있다: 가치 모델(Value Model) 훈련이 필요하여 훈련 복잡도와 GPU 메모리 사용량이 증가하고, 동시에 네 가지 모델(정책 모델, 참조 모델, 가치 모델, 보상 모델)을 유지해야 하여 엔지니어링 구현이 복잡하며, 훈련이 불안정하여 보상 붕괴나 정책 저하가 발생하기 쉽다.

GRPO(Group Relative Policy Optimization)[2]는 LLM을 위해 설계된 단순화된 PPO 변형이다. GRPO의 핵심 아이디어는 가치 모델이 필요 없고, 절대 보상 대신 그룹 내 상대 보상을 사용하며, 훈련 흐름이 단순화되어 정책 모델과 참조 모델만 필요하고, 훈련 안정성이 높아져 보상 붕괴 위험이 줄어든다는 것이다.

수학 공식을 통해 GRPO의 원리를 이해해보자. PPO의 목적 함수는 다음과 같다:

$$
J_{\text{PPO}}(\theta) = \mathbb{E}_{s,a \sim \pi_\theta} \left[ \min\left( \frac{\pi_\theta(a|s)}{\pi_{\text{old}}(a|s)} A(s,a), \text{clip}\left(\frac{\pi_\theta(a|s)}{\pi_{\text{old}}(a|s)}, 1-\epsilon, 1+\epsilon\right) A(s,a) \right) \right]
$$

여기서 $A(s,a)$는 우위 함수(Advantage)로, 가치 모델을 통해 추정해야 한다:

$$
A(s,a) = Q(s,a) - V(s) = r(s,a) + \gamma V(s') - V(s)
$$

GRPO의 목적 함수는 다음과 같이 단순화된다:

$$
J_{\text{GRPO}}(\theta) = \mathbb{E}_{s,a \sim \pi_\theta} \left[ \frac{\pi_\theta(a|s)}{\pi_{\text{ref}}(a|s)} \cdot (r(s,a) - \bar{r}_{\text{group}}) \right] - \beta \cdot D_{KL}(\pi_\theta || \pi_{\text{ref}})
$$

여기서 $\bar{r}_{\text{group}}$은 그룹 내 평균 보상, $\beta$는 KL 발산 패널티 계수다. 핵심 차이점은 GRPO가 우위 함수 $A(s,a)$ 대신 $r(s,a) - \bar{r}_{\text{group}}$을 사용하여 가치 모델이 필요 없다는 것이다. GRPO는 그룹 내 상대 보상을 사용하여 보상 분산을 줄이고, KL 발산 패널티를 추가하여 정책이 너무 멀리 벗어나는 것을 방지한다.

그림 11.7에서 보듯이 PPO와 GRPO의 훈련 흐름을 비교한다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-7.png" alt="" width="85%"/>
  <p>그림 11.7 PPO vs GRPO 훈련 흐름</p>
</div>

GRPO는 가치 모델 훈련이 생략되어 흐름이 크게 단순화된 것을 볼 수 있다.

표 11.6에서 보듯이 PPO와 GRPO의 상세 비교를 확인할 수 있다.

<div align="center">
  <p>표 11.6 PPO vs GRPO 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-6.png" alt="" width="85%"/>
</div>

LLM 훈련에는 GRPO가 더 좋은 선택이다. 더 단순하고, 안정적이며, GPU 메모리 사용량이 적기 때문이다.

### 11.4.2 GRPO 훈련 실습

이제 HelloAgents를 사용하여 GRPO 훈련을 진행해보자. GRPO 훈련의 전제 조건은 SFT 훈련이 완료된 것이다. GRPO는 합리적인 초기 정책이 필요하기 때문이다.

기본 GRPO 훈련 예시:

```python
from hello_agents.tools import RLTrainingTool

# 훈련 도구 생성
rl_tool = RLTrainingTool()

# GRPO 훈련
result = rl_tool.run({
    # 훈련 설정
    "action": "train",
    "algorithm": "grpo",
    
    # 모델 설정
    "model_name": "./models/sft_full",  # SFT 모델에서 시작
    "output_dir": "./models/grpo_model",
    
    # 데이터 설정
    "max_samples": 100,     # 100개 샘플로 빠른 테스트
    
    # 훈련 파라미터
    "num_epochs": 3,
    "batch_size": 4,
    "learning_rate": 1e-5,  # GRPO 학습률은 보통 SFT보다 작음
    
    # GRPO 특정 파라미터
    "num_generations": 4,   # 문제당 4개 답변 생성
    "kl_coef": 0.05,        # KL 발산 패널티 계수
    
    # LoRA 설정
    "use_lora": True,
    "lora_rank": 16,
    "lora_alpha": 32,
    
    # 보상 함수 설정
    "reward_type": "accuracy",  # 정확도 보상 사용
})

print(f"\n✓ 훈련 완료!")
print(f"  - 모델 저장 경로: {result['model_path']}")
print(f"  - 훈련 샘플 수: {result['num_samples']}")
print(f"  - 훈련 에포크 수: {result['num_epochs']}")
print(f"  - 평균 보상: {result['average_reward']:.4f}")
```

GRPO 훈련 과정에서 평균 보상이 점차 높아지고 KL 발산이 합리적인 범위 내에 유지되면 훈련이 정상적으로 진행되고 있음을 나타낸다.

GRPO에는 이해하고 조정해야 할 특정 파라미터들이 있다.

**생성 파라미터**:

- `num_generations`: 문제당 몇 개의 답변을 생성할지. 많을수록 좋지만 계산 비용도 높아진다. 전형적인 값은 4-8이다. 여러 답변을 생성하는 목적은 그룹 내 상대 보상을 계산하고 훈련 신호의 다양성을 높이기 위함이다.
- `max_new_tokens`: 각 답변에서 최대 생성할 토큰 수. 너무 적으면 답변이 잘릴 수 있고 너무 많으면 계산을 낭비한다. 256-512를 권장한다.
- `temperature`: 생성 온도, 무작위성 제어. 0은 탐욕 디코딩, 1은 표준 샘플링이다. GRPO는 0.7-1.0을 권장하여 일정한 탐색성을 유지한다.

**최적화 파라미터**:

- `learning_rate`: GRPO의 학습률은 보통 SFT보다 작다. SFT 모델에서 너무 멀리 벗어나고 싶지 않기 때문이다. 1e-5에서 5e-5를 권장한다.
- `kl_coef`: KL 발산 패널티 계수, 정책 업데이트 폭을 제어한다. 너무 작으면(0.01) 정책이 너무 멀리 벗어나고, 너무 크면(0.5) 학습이 제한된다. 0.05-0.1을 권장한다.
- `clip_range`: 정책 비율 클리핑 범위, PPO의 엡실론과 유사하다. 0.2를 권장한다.

**보상 파라미터**:

- `reward_type`: 보상 함수 유형. "accuracy", "length_penalty", "step", "combined" 중 선택 가능하다.
- `reward_config`: 보상 함수의 추가 설정으로, 길이 패널티의 목표 길이, 단계 보상 계수 등이 있다.

전체 데이터와 최선의 방법으로 완전한 GRPO 훈련을 진행해보자:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# 완전한 GRPO 훈련
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",

    # 모델 설정
    "model_name": "./models/sft_full",
    "output_dir": "./models/grpo_full",
    
    # 데이터 설정
    "max_samples": None,    # 전체 데이터 사용
    
    # 훈련 파라미터
    "num_epochs": 3,
    "batch_size": 4,
    "learning_rate": 1e-5,
    "warmup_ratio": 0.1,
    
    # GRPO 특정 파라미터
    "num_generations": 4,
    "max_new_tokens": 512,
    "temperature": 0.8,
    "kl_coef": 0.05,
    "clip_range": 0.2,
    
    # LoRA 설정
    "use_lora": True,
    "lora_rank": 16,
    "lora_alpha": 32,
    
    # 보상 함수 설정
    "reward_type": "combined",
    "reward_config": {
        "components": [
            {"type": "accuracy", "weight": 1.0},
            {"type": "length_penalty", "weight": 0.5, "target_length": 200},
            {"type": "step", "weight": 0.3, "step_bonus": 0.1}
        ]
    },
    
    # 기타 설정
    "save_steps": 500,
    "logging_steps": 100,
})

print(f"훈련 완료! 모델 저장 위치: {result['model_path']}")
```

### 11.4.3 GRPO 훈련 과정 분석

GRPO의 훈련 과정을 깊이 이해해보자. 각 단계에서 무슨 일이 일어나는지 살펴본다.

**(1) 훈련 루프**

GRPO의 훈련 루프는 다음 단계를 포함한다:

1. **샘플링 단계**: 각 문제에 대해 현재 정책을 사용하여 여러 답변(`num_generations`개)을 생성한다. 이 답변들이 하나의 "그룹"을 구성하며, 상대 보상 계산에 사용된다.

2. **보상 계산**: 각 생성된 답변에 대해 보상 $r_i$를 계산한다. 보상은 정확도, 길이 패널티, 단계 보상 또는 이들의 조합이 될 수 있다.

3. **상대 보상**: 그룹 내 평균 보상 $\bar{r} = \frac{1}{N}\sum_{i=1}^{N} r_i$를 계산한 다음 상대 보상 $\hat{r}_i = r_i - \bar{r}$를 계산한다. 이렇게 하면 보상 분산이 줄어들어 훈련이 더 안정적이 된다.

4. **정책 업데이트**: 상대 보상을 사용하여 정책을 업데이트하는 동시에 KL 발산 패널티를 추가하여 정책이 참조 모델에서 너무 멀리 벗어나는 것을 방지한다.

5. **반복**: 모든 훈련 에포크가 완료될 때까지 위 단계를 반복한다.

구체적인 예를 통해 이해해보자:

```python
# 문제가 있다고 가정
question = "What is 48 + 24?"

# 4개의 답변 생성
answers = [
    "48 + 24 = 72. Final Answer: 72",      # 정답
    "48 + 24 = 72. Final Answer: 72",      # 정답
    "48 + 24 = 70. Final Answer: 70",      # 오답
    "Let me think... 72. Final Answer: 72" # 정답이지만 장황함
]

# 보상 계산(정확도 + 길이 패널티 사용 가정)
rewards = [1.0, 1.0, 0.0, 0.8]  # 4번째 답변은 장황하여 패널티

# 그룹 내 평균 보상 계산
avg_reward = (1.0 + 1.0 + 0.0 + 0.8) / 4 = 0.7

# 상대 보상 계산
relative_rewards = [
    1.0 - 0.7 = 0.3,   # 정답이고 간결함, 상대 보상이 양수
    1.0 - 0.7 = 0.3,   # 정답이고 간결함, 상대 보상이 양수
    0.0 - 0.7 = -0.7,  # 오답, 상대 보상이 음수
    0.8 - 0.7 = 0.1    # 정답이지만 장황함, 상대 보상이 낮음
]

# 정책 업데이트: 첫 두 답변의 확률 증가, 세 번째 답변의 확률 감소
```

상대 보상 메커니즘은 모델이 단순히 높은 보상을 추구하는 것이 아니라 "평균보다 더 좋은" 답변을 생성하도록 장려한다는 것을 알 수 있다. 이렇게 하면 보상 분산이 줄어들고 훈련 안정성이 향상된다.

**(2) KL 발산 패널티**

KL 발산 패널티는 GRPO의 핵심 구성 요소로, 정책이 참조 모델에서 너무 멀리 벗어나는 것을 방지한다. KL 발산은 다음과 같이 정의된다:

$$
D_{KL}(\pi_\theta || \pi_{\text{ref}}) = \mathbb{E}_{s,a \sim \pi_\theta} \left[ \log \frac{\pi_\theta(a|s)}{\pi_{\text{ref}}(a|s)} \right]
$$

실제로는 각 토큰의 KL 발산을 계산하여 합산한다:

$$
D_{KL} = \sum_{t=1}^{T} \log \frac{\pi_\theta(a_t|s, a_{<t})}{\pi_{\text{ref}}(a_t|s, a_{<t})}
$$

KL 발산이 클수록 현재 정책과 참조 모델의 차이가 크다는 것을 의미한다. KL 발산 패널티 항 $-\beta \cdot D_{KL}$을 추가함으로써 정책 업데이트 폭을 제한하여 SFT 단계에서 학습한 지식을 "잊어버리는" 것을 방지한다.

`kl_coef` ($\beta$)의 선택이 중요하다:

- 너무 작으면(0.01): 정책이 너무 멀리 벗어나 출력 형식이 혼란스럽거나 품질이 저하될 수 있다.
- 너무 크면(0.5): 정책 업데이트가 제한되어 학습이 느려지고 SFT 모델을 뛰어넘기 어렵다.
- 권장(0.05-0.1): 탐색과 안정성의 균형을 맞춘다.

**(3) 훈련 모니터링**

GRPO 훈련 과정에서 다음 지표들을 모니터링해야 한다:

- **평균 보상(Average Reward)**: 점차 상승해야 한다. 보상이 상승하지 않으면 학습률이 너무 작거나 KL 패널티가 너무 크거나 보상 함수 설계가 불합리하거나 SFT 모델 품질이 너무 낮은 것일 수 있다. 보상이 상승 후 감소하면 과적합이나 보상 붕괴일 수 있다.

- **KL 발산(KL Divergence)**: 합리적인 범위(0.01-0.1) 내에 유지되어야 한다. KL 발산이 너무 크면(>0.5) 정책이 너무 멀리 벗어난 것으로 kl_coef를 늘리거나 학습률을 낮춰야 한다. KL 발산이 너무 작으면(<0.001) 정책이 거의 업데이트되지 않은 것으로 kl_coef를 줄이거나 학습률을 높여야 한다.

- **정확도(Accuracy)**: 점차 향상되어야 한다. 가장 직관적인 지표로 모델의 실제 능력을 반영한다.

- **생성 품질(Generation Quality)**: 생성된 답변을 직접 확인하여 형식이 올바르고 추론이 명확한지 확인해야 한다.

HelloAgents는 두 가지 주류 훈련 모니터링 도구인 Weights & Biases(wandb)와 TensorBoard를 통합하고 있다.

**방식 1: Weights & Biases 사용(권장)**

Weights & Biases는 현재 가장 인기 있는 머신러닝 실험 추적 플랫폼으로, 강력한 시각화와 실험 관리 기능을 제공한다.

```python
import os

# 1. wandb 설정(먼저 계정 등록 필요: https://wandb.ai)
os.environ["WANDB_PROJECT"] = "hello-agents-grpo"  # 프로젝트 이름
os.environ["WANDB_LOG_MODEL"] = "false"            # 모델 파일 업로드 안 함

# 2. 훈련 설정에서 wandb 활성화
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_monitored",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
    # wandb가 모든 훈련 지표를 자동으로 기록
})

# 훈련 완료 후 https://wandb.ai에서 훈련 곡선 확인
```

wandb는 다음 지표들을 자동으로 기록한다:
- `train/reward`: 평균 보상
- `train/kl`: KL 발산
- `train/loss`: 훈련 손실
- `train/learning_rate`: 학습률
- `train/epoch`: 훈련 에포크

**방식 2: TensorBoard 사용**

TensorBoard는 TensorFlow가 제공하는 시각화 도구로, PyTorch 훈련도 지원한다.

```python
# 1. 훈련 시 output_dir 아래에 자동으로 tensorboard 로그 생성
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_tb",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
})

# 2. TensorBoard 시작하여 훈련 곡선 확인
# 커맨드 라인에서 실행:
# tensorboard --logdir=./models/grpo_tb
# 그 다음 http://localhost:6006 접속
```

**방식 3: 오프라인 모니터링(외부 도구 불필요)**

wandb나 TensorBoard를 사용하고 싶지 않다면 훈련 로그를 통해 모니터링할 수도 있다:

```python
# 훈련 과정에서 상세 로그 출력
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_simple",
    "num_epochs": 2,
    "batch_size": 2,
    "use_lora": True,
})

# 로그 예시:
# Epoch 1/2 | Step 100/500 | Reward: 0.45 | KL: 0.023 | Loss: 1.234
# Epoch 1/2 | Step 200/500 | Reward: 0.52 | KL: 0.031 | Loss: 1.156
# ...
```

GRPO 훈련에서 발생할 수 있는 문제들이 있다. 보상이 상승하지 않을 때는 학습률이 너무 작거나 KL 패널티가 너무 크거나 보상 함수 설계가 불합리하거나 SFT 모델 품질이 너무 낮은 것일 수 있다. 이때 학습률을 높이거나(1e-5에서 5e-5), kl_coef를 낮추거나(0.1에서 0.05), 보상 함수를 확인하거나 SFT 모델을 재훈련한다.

KL 발산이 폭발하여(0.5 또는 1.0 초과) 생성 답변 형식이 혼란스러워질 때는 보통 학습률이 너무 크거나 KL 패널티가 너무 작거나 보상 함수가 너무 공격적인 것이다. 이때 학습률을 낮추거나(5e-5에서 1e-5), kl_coef를 높이거나(0.05에서 0.1), 보상 함수를 조정하거나 그래디언트 클리핑을 사용한다.

생성 품질이 저하될 때(정확도가 향상되지만 형식이 혼란스럽거나 추론이 불명확함) 보상 함수가 정확도에만 집중하여 다른 품질 지표를 무시하거나, KL 패널티가 너무 작아 모델이 SFT에서 너무 멀리 벗어나거나, 과적합이 발생한 것일 수 있다. 이때 조합 보상 함수를 사용하여 여러 지표를 동시에 최적화하거나, kl_coef를 높여 일관성을 유지하거나, 훈련 에포크를 줄이거나 훈련 데이터를 늘린다.

GRPO 훈련의 GPU 메모리 사용량은 SFT보다 높다. 여러 답변을 동시에 생성하고 참조 모델 출력을 저장해야 하기 때문에 OOM이 발생하기 쉽다. num_generations(8에서 4), batch_size(4에서 2), max_new_tokens(512에서 256)를 줄이거나, 그래디언트 체크포인팅과 혼합 정밀도 훈련을 사용하여 완화할 수 있다.

## 11.5 모델 평가와 분석

훈련 완료 후 모델의 성능을 종합적으로 평가해야 한다. 정확도 하나의 지표만 볼 것이 아니라 모델의 추론 품질, 오류 패턴, 일반화 능력 등을 깊이 분석해야 한다. 본 절에서는 Agentic RL 모델을 체계적으로 평가하고 분석하는 방법을 소개한다.

### 11.5.1 평가 지표 체계

좋은 평가 체계는 다차원적이어야 하며, 서로 다른 각도에서 모델의 능력을 측정한다. 평가 지표를 정확성 지표, 효율성 지표, 품질 지표의 세 가지로 분류한다.

**(1) 정확성 지표**

정확성 지표는 모델이 올바른 답을 도출할 수 있는지를 측정한다.

**정확도(Accuracy)**: 가장 기본적인 지표로, 답이 완전히 맞는 비율이다. 계산 공식은:
$$
\text{Accuracy} = \frac{\text{정답 수}}{\text{전체 문제 수}}
$$

장점은 단순하고 직관적이며 이해와 비교가 쉽다는 것이다. 단점은 "거의 정확"과 "완전히 오답"을 구분할 수 없어 복잡한 작업에는 너무 거칠 수 있다는 것이다.

**Top-K 정확도**: K개의 답변을 생성하여 하나라도 맞으면 정답으로 처리한다. 계산 공식은:
$$
\text{Accuracy@K} = \frac{\text{적어도 하나의 정답이 있는 문제 수}}{\text{전체 문제 수}}
$$

이 지표는 모델의 "잠재력", 즉 여러 번 샘플링하여 정답을 찾을 수 있는지를 반영한다.

**수치 오류(Numerical Error)**: 수학 문제의 경우 예측값과 실제값의 오류를 계산할 수 있다. 계산 공식은:

$$
\text{Error} = \frac{1}{N} \sum_{i=1}^{N} |y_i - \hat{y}_i|
$$

이 지표는 "거의 정확"(예: 72.5 예측, 실제 72)과 "완전히 오답"(예: 100 예측, 실제 72)을 구분할 수 있다.

**(2) 효율성 지표**

효율성 지표는 모델이 답변을 생성하는 비용을 측정한다.

**평균 길이(Average Length)**: 생성된 답변의 평균 토큰 수다. 계산 공식은:

$$
\text{Avg Length} = \frac{1}{N} \sum_{i=1}^{N} |y_i|
$$

더 짧은 답변은 더 낮은 추론 비용과 더 빠른 응답 속도를 의미한다.

**추론 단계 수(Reasoning Steps)**: 답변에 포함된 추론 단계의 수다. 계산 공식은:

$$
\text{Avg Steps} = \frac{1}{N} \sum_{i=1}^{N} s_i
$$

적절한 단계 수(2-5단계)는 모델이 문제를 체계적으로 분해할 수 있음을 나타내며, 단계가 너무 많으면 추론이 불필요하게 복잡한 것일 수 있다.

**추론 시간(Inference Time)**: 하나의 답변을 생성하는 데 필요한 시간이다. 이 지표는 실제 배포에서 중요하며 사용자 경험에 영향을 미친다.

**(3) 품질 지표**

품질 지표는 답변의 가독성과 해석 가능성을 측정한다.

**형식 정확도(Format Correctness)**: 답변이 예상 형식을 따르는지("Step 1", "Final Answer" 등의 마커 포함). 계산 공식은:
$$
\text{Format Correctness} = \frac{\text{형식이 올바른 답변 수}}{\text{전체 답변 수}}
$$

형식 정확도는 기본 요구 사항이며, 형식이 혼란스러운 답변은 결과가 맞아도 사용하기 어렵다.

**추론 일관성(Reasoning Coherence)**: 추론 단계 간의 논리적 일관성이다. 이 지표는 보통 인간 평가나 전용 평가 모델이 필요하다.

**해석 가능성(Explainability)**: 답변이 이해하고 검증하기 쉬운지 여부다. 명확한 단계를 포함한 답변이 직접 결과만 제시하는 답변보다 더 높은 해석 가능성을 갖는다.

표 11.7에서 보듯이 다양한 지표를 비교한다.

<div align="center">
  <p>표 11.7 평가 지표 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-7.png" alt="" width="85%"/>
</div>

### 11.5.2 평가 실습

HelloAgents는 다차원 평가 기능을 제공하여 한 번에 여러 지표를 계산할 수 있다.

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# 종합 평가
print("=" * 50)
print("GRPO 모델 종합 평가")
print("=" * 50)

result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/grpo_full",
    "max_samples": 200,
    "use_lora": True,
    
    # 평가 설정
    "metrics": [
        "accuracy",           # 정확도
        "accuracy_at_k",      # Top-K 정확도
        "average_length",     # 평균 길이
        "average_steps",      # 평균 단계 수
        "format_correctness", # 형식 정확도
    ],
    "k": 3,  # Top-3 정확도
})

# 결과 파싱
eval_data = json.loads(result)

# 결과 출력
print(f"\n평가 결과:")
print(f"  정확도: {eval_data['accuracy']}")
print(f"  평균 보상: {eval_data['average_reward']}")
print(f"  테스트 샘플 수: {eval_data['num_samples']}")
```

사전 훈련 모델, SFT 모델, GRPO 모델의 성능을 비교할 수 있다:

```python
# 세 모델 평가
models = [
    ("사전 훈련 모델", "Qwen/Qwen3-0.6B", False),
    ("SFT 모델", "./models/sft_full", True),
    ("GRPO 모델", "./models/grpo_full", True),
]

results = []
for name, path, use_lora in models:
    print(f"\n{name} 평가 중...")
    result = rl_tool.run({
        "action": "evaluate",
        "model_path": path,
        "max_samples": 200,
        "use_lora": use_lora,
        "metrics": ["accuracy", "average_length", "format_correctness"],
    })
    results.append((name, result))

# 비교 표 출력
print("\n" + "=" * 70)
print(f"{'모델':<15} {'정확도':<12} {'평균 길이':<15} {'형식 정확도':<12}")
print("=" * 70)
for name, result in results:
    print(f"{name:<15} {result['accuracy']:<12.2%} {result['average_length']:<15.1f} {result['format_correctness']:<12.2%}")
print("=" * 70)
```

### 11.5.3 오류 분석

정확도만 아는 것으로는 충분하지 않다. 모델이 어떤 유형의 문제에서 오류를 범하기 쉬운지 깊이 분석해야 후속 개선을 이끌어낼 수 있다. 모델의 오류는 네 가지로 분류할 수 있다: 계산 오류(추론 단계는 맞지만 계산이 틀린 경우, 예: "48/2=25", 수치 계산 능력이 부족함을 나타냄), 추론 오류(추론 논리가 틀려 풀이 방향이 잘못된 경우, 예: 나누기를 먼저 한 후 더해야 하는데 반대로 함, 논리적 추론 능력이 부족함을 나타냄), 이해 오류(문제를 올바르게 이해하지 못한 경우, 예: "총합"을 물었는데 일부만 계산함, 언어 이해 능력이 부족함을 나타냄), 형식 오류(답은 맞지만 형식이 요구에 부합하지 않는 경우, 예: "Final Answer:" 마커가 없음, 형식 학습이 부족함을 나타냄).

오류 분석 예시:

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# 평가하고 오류 샘플 수집
result = rl_tool.run({
    "action": "evaluate",
    "model_path": "./models/grpo_full",
    "max_samples": 200,
    "use_lora": True,
    "return_details": True,  # 상세 결과 반환
})

# 오류 샘플 분석
errors = result['errors']  # 오류 샘플 목록
print(f"총 오류 수: {len(errors)}")

# 오류 유형별 분류
error_types = {
    "계산 오류": 0,
    "추론 오류": 0,
    "이해 오류": 0,
    "형식 오류": 0,
}

for error in errors:
    question = error['question']
    prediction = error['prediction']
    ground_truth = error['ground_truth']
    
    # 간단한 오류 분류 로직(실제 응용에서는 더 세밀한 분석이 필요할 수 있음)
    if "Final Answer:" not in prediction:
        error_types["형식 오류"] += 1
    elif "Step" in prediction:
        # 추론 단계가 있으므로 계산 또는 추론 오류일 수 있음
        # 여기서는 더 세밀한 분석이 필요함
        error_types["계산 오류"] += 1
    else:
        error_types["이해 오류"] += 1

# 오류 분포 출력
print("\n오류 유형 분포:")
for error_type, count in error_types.items():
    percentage = count / len(errors) * 100
    print(f"  {error_type}: {count} ({percentage:.1f}%)")
```

출력 예시:

```bash
총 오류 수: 76

오류 유형 분포:
  계산 오류: 32 (42.1%)
  추론 오류: 18 (23.7%)
  이해 오류: 22 (28.9%)
  형식 오류: 4 (5.3%)
```

계산 오류가 가장 주요한 오류 유형(42.1%)임을 알 수 있어 모델의 수치 계산 능력을 강화해야 한다는 것을 보여준다. 형식 오류는 매우 적어(5.3%) SFT 훈련 효과가 좋음을 나타낸다. 또한 다른 난이도의 문제에서 모델의 성능을 분석할 수 있다:

```python
# 추론 단계 수에 따라 그룹화
step_groups = {
    "쉬움(1-2단계)": [],
    "보통(3-4단계)": [],
    "어려움(5단계+)": [],
}

for sample in result['details']:
    steps = sample['ground_truth_steps']  # 실제 답변의 단계 수
    correct = sample['correct']
    
    if steps <= 2:
        step_groups["쉬움(1-2단계)"].append(correct)
    elif steps <= 4:
        step_groups["보통(3-4단계)"].append(correct)
    else:
        step_groups["어려움(5단계+)"].append(correct)

# 각 그룹의 정확도 계산
print("\n다양한 난이도의 정확도:")
for group_name, results in step_groups.items():
    if len(results) > 0:
        accuracy = sum(results) / len(results)
        print(f"  {group_name}: {accuracy:.2%} ({len(results)}개 샘플)")
```

출력 예시:

```bash
다양한 난이도의 정확도:
  쉬움(1-2단계): 78.50% (85개 샘플)
  보통(3-4단계): 58.30% (96개 샘플)
  어려움(5단계+): 31.60% (19개 샘플)
```

모델이 쉬운 문제에서는 잘 수행하지만(78.5%) 어려운 문제에서는 상대적으로 부진함(31.6%)을 알 수 있다. 이는 모델의 다단계 추론 능력이 아직 향상의 여지가 있음을 나타낸다.

### 11.5.4 개선 방향

평가와 분석 결과를 바탕으로 모델의 개선 방향을 확인할 수 있다. 그림 11.8과 같다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-8.png" alt="" width="85%"/>
  <p>그림 11.8 모델 개선 반복 흐름</p>
</div>

이것은 지속적인 반복 과정이다: 모델 훈련 → 성능 평가 → 오류 분석 → 문제 확인 → 개선 방향 선택 → 재훈련. 여러 번의 반복을 통해 모델 성능은 계속 향상된다.

## 11.6 완전한 훈련 흐름 실습

앞선 절들에서 데이터 준비, SFT 훈련, GRPO 훈련, 모델 평가를 각각 학습했다. 이제 이 지식들을 통합하여 종단간(end-to-end) Agentic RL 훈련 흐름을 완성해보자.

### 11.6.1 종단간 훈련 흐름

완전한 Agentic RL 훈련 흐름은 다음 단계를 포함한다: 데이터 준비, SFT 훈련, SFT 평가, GRPO 훈련, GRPO 평가, 모델 배포. 그림 11.9와 같다.

<div align="center">
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-9.png" alt="" width="85%"/>
  <p>그림 11.9 종단간 훈련 흐름</p>
</div>

완전한 스크립트를 통해 이 흐름을 구현해보자:

```python
"""
완전한 Agentic RL 훈련 흐름
데이터 준비부터 모델 배포까지의 종단간 예시
"""

from hello_agents.tools import RLTrainingTool
import json
from datetime import datetime

class AgenticRLPipeline:
    """Agentic RL 훈련 파이프라인"""
    
    def __init__(self, config_path="config.json"):
        """
        훈련 파이프라인 초기화
        
        Args:
            config_path: 설정 파일 경로
        """
        self.rl_tool = RLTrainingTool()
        self.config = self.load_config(config_path)
        self.results = {}
        
    def load_config(self, config_path):
        """설정 파일 로드"""
        with open(config_path, 'r') as f:
            return json.load(f)
    
    def log(self, message):
        """로그 기록"""
        timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        print(f"[{timestamp}] {message}")
    
    def stage1_prepare_data(self):
        """단계1: 데이터 준비"""
        self.log("=" * 50)
        self.log("단계1: 데이터 준비")
        self.log("=" * 50)

        # 데이터셋 로드 및 확인
        result = self.rl_tool.run({
            "action": "load_dataset",
            "format": "sft",
            "max_samples": self.config["data"]["max_samples"],
        })

        # JSON 결과 파싱
        dataset_info = json.loads(result)

        self.log(f"✓ 데이터셋 로드 완료")
        self.log(f"  - 샘플 수: {dataset_info['dataset_size']}")
        self.log(f"  - 형식: {dataset_info['format']}")
        self.log(f"  - 데이터 열: {', '.join(dataset_info['sample_keys'])}")

        self.results["data"] = dataset_info

        return dataset_info
    
    def stage2_sft_training(self):
        """단계2: SFT 훈련"""
        self.log("\n" + "=" * 50)
        self.log("단계2: SFT 훈련")
        self.log("=" * 50)

        sft_config = self.config["sft"]

        result = self.rl_tool.run({
            "action": "train",
            "algorithm": "sft",
            "model_name": self.config["model"]["base_model"],
            "output_dir": sft_config["output_dir"],
            "max_samples": self.config["data"]["max_samples"],
            "num_epochs": sft_config["num_epochs"],
            "batch_size": sft_config["batch_size"],
            "use_lora": True,
            # 훈련 모니터링 설정
            "use_wandb": self.config.get("monitoring", {}).get("use_wandb", False),
            "use_tensorboard": self.config.get("monitoring", {}).get("use_tensorboard", True),
            "wandb_project": self.config.get("monitoring", {}).get("wandb_project", None),
        })

        # JSON 결과 파싱
        result_data = json.loads(result)

        self.log(f"✓ SFT 훈련 완료")
        self.log(f"  - 모델 경로: {result_data['output_dir']}")
        self.log(f"  - 상태: {result_data['status']}")

        self.results["sft_training"] = result_data

        return result_data["output_dir"]
    
    def stage3_sft_evaluation(self, model_path):
        """단계3: SFT 평가"""
        self.log("\n" + "=" * 50)
        self.log("단계3: SFT 평가")
        self.log("=" * 50)
        
        result = self.rl_tool.run({
            "action": "evaluate",
            "model_path": model_path,
            "max_samples": self.config["eval"]["max_samples"],
            "use_lora": True,
        })
        eval_data = json.loads(result)

        self.log(f"✓ SFT 평가 완료")
        self.log(f"  - 정확도: {eval_data['accuracy']}")
        self.log(f"  - 평균 보상: {eval_data['average_reward']}")

        self.results["sft_evaluation"] = eval_data

        return eval_data
    
    def stage4_grpo_training(self, sft_model_path):
        """단계4: GRPO 훈련"""
        self.log("\n" + "=" * 50)
        self.log("단계4: GRPO 훈련")
        self.log("=" * 50)

        grpo_config = self.config["grpo"]

        result = self.rl_tool.run({
            "action": "train",
            "algorithm": "grpo",
            "model_name": sft_model_path,
            "output_dir": grpo_config["output_dir"],
            "max_samples": self.config["data"]["max_samples"],
            "num_epochs": grpo_config["num_epochs"],
            "batch_size": grpo_config["batch_size"],
            "use_lora": True,
            # 훈련 모니터링 설정
            "use_wandb": self.config.get("monitoring", {}).get("use_wandb", False),
            "use_tensorboard": self.config.get("monitoring", {}).get("use_tensorboard", True),
            "wandb_project": self.config.get("monitoring", {}).get("wandb_project", None),
        })

        # JSON 결과 파싱
        result_data = json.loads(result)

        self.log(f"✓ GRPO 훈련 완료")
        self.log(f"  - 모델 경로: {result_data['output_dir']}")
        self.log(f"  - 상태: {result_data['status']}")

        self.results["grpo_training"] = result_data

        return result_data["output_dir"]
    
    def stage5_grpo_evaluation(self, model_path):
        """단계5: GRPO 평가"""
        self.log("\n" + "=" * 50)
        self.log("단계5: GRPO 평가")
        self.log("=" * 50)
        
        result = self.rl_tool.run({
            "action": "evaluate",
            "model_path": model_path,
            "max_samples": self.config["eval"]["max_samples"],
            "use_lora": True,
        })
        eval_data = json.loads(result)

        self.log(f"✓ GRPO 평가 완료")
        self.log(f"  - 정확도: {eval_data['accuracy']}")
        self.log(f"  - 평균 보상: {eval_data['average_reward']}")

        self.results["grpo_evaluation"] = eval_data

        return eval_data
    
    def stage6_save_results(self):
        """단계6: 결과 저장"""
        self.log("\n" + "=" * 50)
        self.log("단계6: 결과 저장")
        self.log("=" * 50)
        
        # 훈련 결과 저장
        results_path = "training_results.json"
        with open(results_path, 'w') as f:
            json.dump(self.results, f, indent=2)
        
        self.log(f"✓ 결과가 저장됨: {results_path}")
    
    def run(self):
        """완전한 흐름 실행"""
        try:
            # 단계1: 데이터 준비
            self.stage1_prepare_data()
            
            # 단계2: SFT 훈련
            sft_model_path = self.stage2_sft_training()
            
            # 단계3: SFT 평가
            self.stage3_sft_evaluation(sft_model_path)
            
            # 단계4: GRPO 훈련
            grpo_model_path = self.stage4_grpo_training(sft_model_path)
            
            # 단계5: GRPO 평가
            self.stage5_grpo_evaluation(grpo_model_path)
            
            # 단계6: 결과 저장
            self.stage6_save_results()
            
            self.log("\n" + "=" * 50)
            self.log("✓ 훈련 흐름 완료!")
            self.log("=" * 50)
            
        except Exception as e:
            self.log(f"\n✗ 훈련 실패: {str(e)}")
            raise

# 사용 예시
if __name__ == "__main__":
    # 설정 파일 생성
    config = {
        "model": {
            "base_model": "Qwen/Qwen3-0.6B"
        },
        "data": {
            "max_samples": 1000  # 1000개 샘플 사용
        },
        "sft": {
            "output_dir": "./models/sft_model",
            "num_epochs": 3,
            "batch_size": 8,
        },
        "grpo": {
            "output_dir": "./models/grpo_model",
            "num_epochs": 3,
            "batch_size": 4,
        },
        "eval": {
            "max_samples": 200,
            "sft_accuracy_threshold": 0.40  # SFT 정확도 임계값
        },
        "monitoring": {
            "use_wandb": False,  # Wandb 사용 여부
            "use_tensorboard": True,  # TensorBoard 사용 여부
            "wandb_project": "agentic-rl-pipeline"  # Wandb 프로젝트 이름
        }
    }
    
    # 설정 저장
    with open("config.json", 'w') as f:
        json.dump(config, f, indent=2)
    
    # 훈련 흐름 실행
    pipeline = AgenticRLPipeline("config.json")
    pipeline.run()
```

이 스크립트를 실행하면 완전한 훈련 과정을 볼 수 있다.

실행 시 참고 사항:

**소규모부터 시작하기**: 처음부터 전체 데이터로 훈련하지 말 것. 먼저 100-1000개 샘플로 빠르게 반복하여 흐름과 파라미터를 검증하고, 효과가 확인되면 규모를 확대한다. 이렇게 하면 많은 시간과 계산 자원을 절약할 수 있다.

**데이터 품질 검사**: 훈련 전에 데이터 품질을 확인하여 형식이 올바른지, 답변이 정확한지, 중복 샘플이 없는지 확인한다. 다음 코드를 사용할 수 있다:

```python
def check_data_quality(dataset):
    """데이터 품질 확인"""
    issues = []

    # 필수 필드 확인
    required_fields = ["prompt", "completion"]
    for field in required_fields:
        if field not in dataset.column_names:
            issues.append(f"필드 누락: {field}")

    # 빈 값 확인
    for i, sample in enumerate(dataset):
        if not sample["prompt"] or not sample["completion"]:
            issues.append(f"샘플 {i}에 빈 값 포함")

    # 중복 확인
    prompts = [s["prompt"] for s in dataset]
    duplicates = len(prompts) - len(set(prompts))
    if duplicates > 0:
        issues.append(f"{duplicates}개의 중복 샘플 발견")

    return issues

# 사용
issues = check_data_quality(dataset)
if issues:
    print("데이터 품질 문제:")
    for issue in issues:
        print(f"  - {issue}")
else:
    print("✓ 데이터 품질 검사 통과")
```

**데이터 증강**: 데이터량이 부족하면 데이터 증강을 고려할 수 있다. 예를 들어 질문 재작성(답은 그대로 유지), 유사 질문 생성, 역번역 등이 있다. 다만 데이터 품질을 유지하고 노이즈를 도입하지 않도록 주의해야 한다.

### 11.6.2 하이퍼파라미터 조정

하이퍼파라미터 조정은 모델 성능을 향상시키는 핵심이다. 다음은 몇 가지 일반적인 조정 전략이다.

**(1) 그리드 서치**

그리드 서치(Grid Search)는 가장 단순한 조정 방법으로, 모든 파라미터 조합을 순회하여 최적의 조합을 선택한다.

```python
# 파라미터 그리드 정의
param_grid = {
    "learning_rate": [1e-5, 5e-5, 1e-4],
    "lora_rank": [8, 16, 32],
    "kl_coef": [0.05, 0.1, 0.2],
}

best_accuracy = 0
best_params = None

# 모든 조합 순회
for lr in param_grid["learning_rate"]:
    for rank in param_grid["lora_rank"]:
        for kl in param_grid["kl_coef"]:
            print(f"파라미터 테스트: lr={lr}, rank={rank}, kl={kl}")

            # 모델 훈련
            result = rl_tool.run({
                "action": "train",
                "algorithm": "grpo",
                "learning_rate": lr,
                "lora_rank": rank,
                "kl_coef": kl,
                # 기타 파라미터...
            })

            # 모델 평가
            eval_result = rl_tool.run({
                "action": "evaluate",
                "model_path": result["model_path"],
            })

            # 최적 파라미터 업데이트
            if eval_result["accuracy"] > best_accuracy:
                best_accuracy = eval_result["accuracy"]
                best_params = {"lr": lr, "rank": rank, "kl": kl}

print(f"최적 파라미터: {best_params}")
print(f"최고 정확도: {best_accuracy:.2%}")
```

그리드 서치의 장점은 단순하고 직접적이며 전역 최적을 찾을 수 있다는 것이다. 단점은 계산 비용이 높고 파라미터가 많을 때 실행 불가능하다는 것이다.

**(2) 랜덤 서치**

랜덤 서치(Random Search)는 파라미터 조합을 무작위로 샘플링하여 그리드 서치보다 효율적이다.

```python
import random

# 파라미터 범위 정의
param_ranges = {
    "learning_rate": (1e-6, 1e-4),  # 로그 균등 분포
    "lora_rank": [4, 8, 16, 32, 64],
    "kl_coef": (0.01, 0.5),
}

best_accuracy = 0
best_params = None

# N번 무작위 샘플링
N = 10
for i in range(N):
    # 파라미터 무작위 샘플링
    lr = 10 ** random.uniform(-6, -4)  # 로그 균등
    rank = random.choice(param_ranges["lora_rank"])
    kl = random.uniform(0.01, 0.5)

    print(f"[{i+1}/{N}] 파라미터 테스트: lr={lr:.2e}, rank={rank}, kl={kl:.3f}")

    # 훈련 및 평가(위와 동일)
    # ...

print(f"최적 파라미터: {best_params}")
print(f"최고 정확도: {best_accuracy:.2%}")
```

랜덤 서치의 장점은 효율이 높고 파라미터 공간이 큰 경우에 적합하다는 것이다. 단점은 최적해를 놓칠 수 있다는 것이다.

**(3) 베이지안 최적화**

베이지안 최적화(Bayesian Optimization)는 확률 모델을 사용하여 탐색을 유도하여 더 지능적이다. Optuna 같은 라이브러리를 사용할 수 있다:

```python
import optuna

def objective(trial):
    """최적화 목적 함수"""
    # 파라미터 샘플링
    lr = trial.suggest_loguniform("learning_rate", 1e-6, 1e-4)
    rank = trial.suggest_categorical("lora_rank", [8, 16, 32])
    kl = trial.suggest_uniform("kl_coef", 0.01, 0.5)

    # 모델 훈련
    result = rl_tool.run({
        "action": "train",
        "algorithm": "grpo",
        "learning_rate": lr,
        "lora_rank": rank,
        "kl_coef": kl,
        # 기타 파라미터...
    })

    # 모델 평가
    eval_result = rl_tool.run({
        "action": "evaluate",
        "model_path": result["model_path"],
    })

    return eval_result["accuracy"]

# 연구 생성
study = optuna.create_study(direction="maximize")
study.optimize(objective, n_trials=20)

# 최적 파라미터 출력
print(f"최적 파라미터: {study.best_params}")
print(f"최고 정확도: {study.best_value:.2%}")
```

베이지안 최적화의 장점은 샘플 효율이 높고 좋은 파라미터를 빠르게 찾을 수 있다는 것이다. 단점은 구현이 복잡하고 추가 라이브러리가 필요하다는 것이다.

표 11.8에서 보듯이 다양한 조정 방법을 비교한다.

<div align="center">
  <p>표 11.8 하이퍼파라미터 조정 방법 비교</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-8.png" alt="" width="85%"/>
</div>

### 11.6.3 분산 훈련

데이터량과 모델 규모가 커지면 단일 GPU 훈련이 매우 느려진다. 이때 분산 훈련을 사용하여 훈련 과정을 가속화해야 한다. HelloAgents는 TRL과 Hugging Face Accelerate를 기반으로 하여 다중 GPU 및 다중 노드 분산 훈련을 기본적으로 지원한다.

**방안 선택 권장**:

- **단일 머신 다중 GPU(2-8개)**: DDP 사용, 단순하고 효율적
- **대형 모델(>7B)**: DeepSpeed ZeRO-2 또는 ZeRO-3 사용
- **다중 노드 클러스터**: DeepSpeed ZeRO-3 + Offload 사용

**(1) Accelerate 설정**

먼저 Accelerate 설정 파일을 생성해야 한다. 다음 명령을 실행한다:

```bash
accelerate config
```

프롬프트에 따라 설정을 선택한다:

```
In which compute environment are you running?
> This machine

Which type of machine are you using?
> multi-GPU

How many different machines will you use?
> 1

Do you wish to optimize your script with torch dynamo?
> NO

Do you want to use DeepSpeed?
> YES

Which DeepSpeed config file do you want to use?
> ZeRO-2

How many GPU(s) should be used for distributed training?
> 4
```

이렇게 하면 `~/.cache/huggingface/accelerate/default_config.yaml`에 설정 파일이 생성된다.

**(2) DDP 훈련 사용**

**데이터 병렬(DDP)**은 가장 단순한 분산 방안으로, 각 GPU가 완전한 모델 복사본을 보유하고 데이터가 각 GPU에 분할된다.

**Accelerate 설정 파일** (`multi_gpu_ddp.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: MULTI_GPU
num_processes: 4  # GPU 수
machine_rank: 0
num_machines: 1
gpu_ids: all
mixed_precision: fp16
```

**훈련 스크립트** (수정 불필요):

```python
from hello_agents.tools import RLTrainingTool

rl_tool = RLTrainingTool()

# 훈련 코드는 완전히 동일
result = rl_tool.run({
    "action": "train",
    "algorithm": "grpo",
    "model_name": "Qwen/Qwen3-0.6B",
    "output_dir": "./models/grpo_ddp",
    "num_epochs": 3,
    "batch_size": 4,  # 각 GPU의 배치 크기
    "use_lora": True,
})
```

**훈련 시작**:

```bash
# 설정 파일 사용
accelerate launch --config_file multi_gpu_ddp.yaml train_script.py

# 또는 직접 파라미터 지정
accelerate launch --num_processes 4 --mixed_precision fp16 train_script.py
```

**(3) DeepSpeed ZeRO 훈련 사용**

**DeepSpeed ZeRO**는 옵티마이저 상태, 그래디언트, 모델 파라미터를 분할함으로써 GPU 메모리 사용량을 크게 줄여 더 큰 모델과 배치 크기를 지원한다.

**ZeRO-2 설정 파일** (`deepspeed_zero2.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: DEEPSPEED
num_processes: 4
machine_rank: 0
num_machines: 1
gpu_ids: all
mixed_precision: fp16
deepspeed_config:
  gradient_accumulation_steps: 4
  gradient_clipping: 1.0
  offload_optimizer_device: none
  offload_param_device: none
  zero3_init_flag: false
  zero_stage: 2  # ZeRO-2
```

**ZeRO-3 설정 파일** (`deepspeed_zero3.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: DEEPSPEED
num_processes: 4
machine_rank: 0
num_machines: 1
gpu_ids: all
mixed_precision: fp16
deepspeed_config:
  gradient_accumulation_steps: 4
  gradient_clipping: 1.0
  offload_optimizer_device: cpu  # 옵티마이저 상태를 CPU로 오프로드
  offload_param_device: cpu      # 파라미터를 CPU로 오프로드
  zero3_init_flag: true
  zero_stage: 3  # ZeRO-3
```

**훈련 시작**:

```bash
# ZeRO-2
accelerate launch --config_file deepspeed_zero2.yaml train_script.py

# ZeRO-3
accelerate launch --config_file deepspeed_zero3.yaml train_script.py
```

표 11.9에서 보듯이 Qwen3-0.6B 모델을 다양한 방식으로 훈련했을 때의 GPU 메모리 비교다:

<div align="center">
  <p>표 11.9 GPU 메모리 비교 (Qwen3-0.6B 모델)</p>
  <img src="https://raw.githubusercontent.com/datawhalechina/Hello-Agents/main/docs/images/11-figures/11-table-9.png" alt="" width="85%"/>
</div>

**(4) 다중 노드 훈련**

초대규모 훈련의 경우 여러 노드(머신)를 사용할 수 있다.

**주 노드 설정** (`multi_node_main.yaml`):

```yaml
compute_environment: LOCAL_MACHINE
distributed_type: DEEPSPEED
num_processes: 16  # 4 노드 x 4 GPU
machine_rank: 0    # 주 노드
num_machines: 4
main_process_ip: 192.168.1.100  # 주 노드 IP
main_process_port: 29500
gpu_ids: all
mixed_precision: fp16
deepspeed_config:
  zero_stage: 3
  offload_optimizer_device: cpu
  offload_param_device: cpu
```

**작업자 노드 설정** (`machine_rank`를 1, 2, 3으로 수정):

```yaml
machine_rank: 1  # 작업자 노드1
# 기타 설정은 동일
```

**훈련 시작**:

```bash
# 주 노드에서
accelerate launch --config_file multi_node_main.yaml train_script.py

# 작업자 노드1에서
accelerate launch --config_file multi_node_worker1.yaml train_script.py

# 작업자 노드2에서
accelerate launch --config_file multi_node_worker2.yaml train_script.py

# 작업자 노드3에서
accelerate launch --config_file multi_node_worker3.yaml train_script.py
```

**(5) 분산 훈련 모범 사례**

**1. 배치 크기 조정**

분산 훈련 시 총 배치 크기 = `per_device_batch_size × num_gpus × gradient_accumulation_steps`

```python
# 단일 GPU: batch_size=4, gradient_accumulation=4, 총 배치=16
# 4개 GPU DDP: batch_size=4, gradient_accumulation=1, 총 배치=16 (일치 유지)
```

**2. 학습률 스케일링**

선형 스케일링 규칙 사용: `lr_new = lr_base × sqrt(total_batch_size_new / total_batch_size_base)`

```python
# 기준: 단일 GPU, batch=16, lr=5e-5
# 4개 GPU: batch=64, lr=5e-5 × sqrt(64/16) = 1e-4
```

**3. 모니터링 및 디버깅**

```python
# 상세 로그 활성화
export ACCELERATE_LOG_LEVEL=INFO

# NCCL 디버그 활성화(다중 노드)
export NCCL_DEBUG=INFO

# GPU 이용률 확인
watch -n 1 nvidia-smi
```

### 11.6.4 프로덕션 배포

훈련 완료 후 모델을 프로덕션 환경에 배포해야 한다. 다음은 몇 가지 배포 권장 사항이다.

**(1) 모델 내보내기**

LoRA 가중치를 기본 모델에 통합하여 배포를 용이하게 한다:

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
from peft import PeftModel

# 기본 모델 로드
base_model = AutoModelForCausalLM.from_pretrained("Qwen/Qwen3-0.6B")

# LoRA 가중치 로드
model = PeftModel.from_pretrained(base_model, "./models/grpo_model")

# 가중치 통합
merged_model = model.merge_and_unload()

# 통합된 모델 저장
merged_model.save_pretrained("./models/merged_model")

# 토크나이저 저장
tokenizer = AutoTokenizer.from_pretrained("Qwen/Qwen3-0.6B")
tokenizer.save_pretrained("./models/merged_model")

print("✓ 모델이 내보내짐: ./models/merged_model")
```

**(2) 추론 최적화**

양자화 및 최적화 기법을 사용하여 추론을 가속화한다:

```python
from transformers import AutoModelForCausalLM, AutoTokenizer
import torch

# 모델 로드(8비트 양자화 사용)
model = AutoModelForCausalLM.from_pretrained(
    "./models/merged_model",
    load_in_8bit=True,  # 8비트 양자화
    device_map="auto",  # 자동 장치 할당
)

tokenizer = AutoTokenizer.from_pretrained("./models/merged_model")

# 추론
def generate_answer(question):
    prompt = f"<|im_start|>user\n{question}<|im_end|>\n<|im_start|>assistant\n"
    inputs = tokenizer(prompt, return_tensors="pt").to(model.device)

    outputs = model.generate(
        **inputs,
        max_new_tokens=512,
        temperature=0.7,
        do_sample=True,
    )

    response = tokenizer.decode(outputs[0], skip_special_tokens=False)
    return response

# 테스트
question = "What is 48 + 24?"
answer = generate_answer(question)
print(answer)
```

**(3) API 서비스**

FastAPI를 사용하여 추론 서비스를 만든다:

```python
from fastapi import FastAPI
from pydantic import BaseModel
from transformers import AutoModelForCausalLM, AutoTokenizer

app = FastAPI()

# 모델 로드
model = AutoModelForCausalLM.from_pretrained("./models/merged_model")
tokenizer = AutoTokenizer.from_pretrained("./models/merged_model")

class Question(BaseModel):
    text: str
    max_tokens: int = 512

class Answer(BaseModel):
    text: str
    confidence: float

@app.post("/generate", response_model=Answer)
def generate(question: Question):
    """답변 생성"""
    prompt = f"<|im_start|>user\n{question.text}<|im_end|>\n<|im_start|>assistant\n"
    inputs = tokenizer(prompt, return_tensors="pt")

    outputs = model.generate(
        **inputs,
        max_new_tokens=question.max_tokens,
        temperature=0.7,
        return_dict_in_generate=True,
        output_scores=True,
    )

    response = tokenizer.decode(outputs.sequences[0], skip_special_tokens=False)

    # 신뢰도 계산(단순화 버전)
    confidence = 0.8  # 실제로는 출력 확률을 기반으로 계산해야 함

    return Answer(text=response, confidence=confidence)

# 실행: uvicorn api:app --host 0.0.0.0 --port 8000
```

## 11.8 본 장 정리

본 장에서는 Agentic RL의 이론과 실습을 체계적으로 학습했다. 기본 개념부터 완전한 훈련 흐름까지, 데이터 준비부터 모델 배포까지를 다루었다. 본 장의 주요 내용을 되짚어보자.

**(1) Agentic RL의 본질**

Agentic RL은 LLM을 학습 가능한 정책으로 보고 에이전트의 인식-결정-실행 루프에 내장시켜, 강화학습을 통해 다단계 작업에서의 에이전트 성능을 최적화한다. 전통적인 PBRFT(Preference-Based Reinforcement Fine-Tuning)와의 핵심 차이점은 다음과 같다:

- **작업 특성**: 단일 대화 최적화에서 다단계 순차적 의사결정으로 확장
- **상태 공간**: 정적 프롬프트에서 동적으로 진화하는 환경 상태로 확장
- **행동 공간**: 순수 텍스트 생성에서 텍스트 + 도구 + 환경 조작으로 확장
- **보상 설계**: 단일 단계 품질 평가에서 장기 누적 보상으로 확장
- **최적화 목표**: 단기 응답 품질에서 장기 작업 성공으로 확장

**(2) 6가지 핵심 능력**

Agentic RL은 에이전트의 6가지 핵심 능력 향상을 목표로 한다:

1. **추론(Reasoning)**: 다단계 논리적 추론, 추론 전략 학습
2. **도구 사용(Tool Use)**: API/도구 호출, 언제 어떻게 사용할지 학습
3. **기억(Memory)**: 장기 정보 유지, 기억 관리 학습
4. **계획(Planning)**: 행동 시퀀스 계획, 동적 계획 학습
5. **자기 개선(Self-Improvement)**: 자기 반성 최적화, 실수로부터 학습
6. **인식(Perception)**: 멀티모달 이해, 시각적 추론과 도구 사용

**(3) 훈련 흐름**

완전한 Agentic RL 훈련 흐름은 다음을 포함한다:

1. **사전 훈련(Pretraining)**: 대규모 텍스트에서 언어 지식 학습(일반적으로 기존 사전 훈련 모델 사용)
2. **지도 미세조정(SFT)**: 작업 형식과 기초 추론 능력 학습
3. **강화학습(RL)**: 시행착오를 통해 추론 전략 최적화, 훈련 데이터 품질 초월

SFT는 기초이고 RL은 향상이다. SFT 기초 없이는 RL이 성공하기 어렵고, RL 최적화 없이는 모델이 훈련 데이터만 모방할 수밖에 없다.

Agentic RL을 깊이 학습하고자 한다면 다음 경로를 권장한다:

기초 단계:

1. **강화학습 기초**: MDP, 정책 그래디언트, PPO 등 기본 개념 학습
2. **LLM 기초**: 트랜스포머, 사전 훈련, 미세조정 등 기술 이해
3. **HelloAgents 실습**: 본 장의 예시 코드를 실행하여 전체 흐름 이해

심화 단계:

1. **TRL 깊이 이해**: TRL 라이브러리 구현 학습, SFT 및 GRPO 등 알고리즘 세부 이해
2. **사용자 정의 데이터셋**: 자신의 데이터셋으로 모델 훈련
3. **사용자 정의 보상 함수**: 자신의 작업에 적합한 보상 함수 설계
4. **파라미터 조정**: 체계적으로 하이퍼파라미터 조정하여 모델 성능 향상

고급 단계:

1. **다단계 추론**: 긴 시퀀스 추론 작업 연구
2. **도구 학습**: 에이전트가 도구 사용을 학습하도록 하기
3. **다중 에이전트**: 다중 에이전트 협업 연구
4. **최신 논문**: 최신 연구 논문 읽기, 최전선 발전 동향 파악

본 장이 Agentic RL 기술을 이해하고 마스터하는 데 도움이 되어, 자신의 프로젝트에 이 지식을 적용하고 더욱 지능적인 에이전트 시스템을 구축하는 데 기여하기를 바란다!

---

## 참고 문헌

[1] Schulman, J., Wolski, F., Dhariwal, P., Radford, A., & Klimov, O. (2017). Proximal Policy Optimization Algorithms. *arXiv preprint arXiv:1707.06347*.

[2] Shao, Z., Wang, P., Zhu, Q., Xu, R., Song, J., Zhang, M., ... & Guo, D. (2024). DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models. *arXiv preprint arXiv:2402.03300*.

[3] Hu, E. J., Shen, Y., Wallis, P., Allen-Zhu, Z., Li, Y., Wang, S., ... & Chen, W. (2021). LoRA: Low-Rank Adaptation of Large Language Models. *arXiv preprint arXiv:2106.09685*.

[4] Cobbe, K., Kosaraju, V., Bavarian, M., Chen, M., Jun, H., Kaiser, L., ... & Schulman, J. (2021). Training Verifiers to Solve Math Word Problems. *arXiv preprint arXiv:2110.14168*.

[5] Ouyang, L., Wu, J., Jiang, X., Almeida, D., Wainwright, C., Mishkin, P., ... & Lowe, R. (2022). Training language models to follow instructions with human feedback. *Advances in Neural Information Processing Systems*, 35, 27730-27744.

[6] Rafailov, R., Sharma, A., Mitchell, E., Ermon, S., Manning, C. D., & Finn, C. (2023). Direct Preference Optimization: Your Language Model is Secretly a Reward Model. *arXiv preprint arXiv:2305.18290*.

[7] Lee, H., Phatale, S., Mansoor, H., Lu, K., Mesnard, T., Bishop, C., ... & Rastogi, A. (2023). RLAIF: Scaling Reinforcement Learning from Human Feedback with AI Feedback. *arXiv preprint arXiv:2309.00267*.

[8] Wei, J., Wang, X., Schuurmans, D., Bosma, M., Ichter, B., Xia, F., ... & Zhou, D. (2022). Chain-of-Thought Prompting Elicits Reasoning in Large Language Models. *Advances in Neural Information Processing Systems*, 35, 24824-24837.

[9] von Werra, L., Belkada, Y., Tunstall, L., Beeching, E., Thrush, T., Lambert, N., & Huang, S. (2020). TRL: Transformer Reinforcement Learning. *GitHub repository*. https://github.com/huggingface/trl

[10] Qwen Team. (2025). Qwen3 Technical Report. *arXiv preprint arXiv:2505.09388*.

[11] Bai, Y., Jones, A., Ndousse, K., Askell, A., Chen, A., DasSarma, N., ... & Kaplan, J. (2022). Training a Helpful and Harmless Assistant with Reinforcement Learning from Human Feedback. *arXiv preprint arXiv:2204.05862*.

[12] Wang, X., Wei, J., Schuurmans, D., Le, Q., Chi, E., Narang, S., ... & Zhou, D. (2022). Self-Consistency Improves Chain of Thought Reasoning in Language Models. *arXiv preprint arXiv:2203.11171*.

[13] Christiano, P. F., Leike, J., Brown, T., Martic, M., Legg, S., & Amodei, D. (2017). Deep Reinforcement Learning from Human Preferences. *Advances in Neural Information Processing Systems*, 30.

[14] Stiennon, N., Ouyang, L., Wu, J., Ziegler, D., Lowe, R., Voss, C., ... & Christiano, P. F. (2020). Learning to summarize with human feedback. *Advances in Neural Information Processing Systems*, 33, 3008-3021.

[15] Ziegler, D. M., Stiennon, N., Wu, J., Brown, T. B., Radford, A., Amodei, D., ... & Irving, G. (2019). Fine-Tuning Language Models from Human Preferences. *arXiv preprint arXiv:1909.08593*.

---

## 연습 문제

> **힌트**: 일부 연습 문제는 표준 답이 없으며, Agentic RL과 에이전트 훈련에 대한 학습자의 종합적인 이해와 실습 능력을 키우는 데 중점을 둔다.

1. 본 장에서는 LLM 훈련에서 Agentic RL로의 진화 과정을 소개했다. 다음을 분석하라:

   - 11.1.3절의 표 11.1에서 MDP 프레임워크 하에서 PBRFT(선호도 기반 강화 미세조정)와 Agentic RL의 차이를 비교했다. 다음을 깊이 설명하라: Agentic RL의 상태 공간 $s_t = (\text{prompt}, o_1, o_2, ..., o_t)$에 역사적 관찰이 포함되는 반면 PBRFT의 상태 $s_0 = \text{prompt}$는 초기 프롬프트만 포함하는 이유는 무엇인가? 이 차이가 훈련 과정과 최종 효과에 어떤 영향을 미치는가?
   - "지능형 코드 디버깅 어시스턴트"를 훈련한다고 가정하자. 이 어시스턴트는 다음이 필요하다: (1) 코드를 분석하여 버그를 찾고; (2) 문서를 참조하여 API 사용법을 이해하며; (3) 코드를 수정하고; (4) 테스트를 실행하여 수정 효과를 검증한다. 이 작업을 강화학습 프레임워크에 매핑하여 상태 공간, 행동 공간, 보상 함수, 상태 전이 함수를 명확히 정의하라.
   - 11.1.1절에서 전통적인 지도학습이 "장기 목표를 최적화하기 어렵다"는 한계가 언급되었다. 구체적인 다단계 추론 작업(수학 증명, 복잡한 문제 풀기 등)을 설계하여 지도학습이 중간 단계를 최적화하기 어려운 이유와 강화학습이 지연 보상을 통해 이 문제를 해결하는 방법을 보여라.

2. SFT(지도 미세조정)와 GRPO(군집 상대 정책 최적화)는 본 장의 두 가지 핵심 훈련 방법이다. 11.2절과 11.3절의 내용을 바탕으로 깊이 생각해보라:

   > **힌트**: 이것은 직접 실습을 권장하는 문제다.

   - 11.2.4절의 SFT 훈련 코드에서 LoRA(저순위 적응) 기법을 사용하여 훈련 파라미터를 줄였다. 다음을 분석하라: LoRA의 핵심 아이디어는 무엇인가? 소량의 파라미터(예: 0.16%)로 풀 파라미터 미세조정에 가까운 효과를 어떻게 달성할 수 있는가? 어떤 상황에서 LoRA 대신 풀 파라미터 미세조정을 선택해야 하는가?
   - GRPO 알고리즘(11.3절)이 전통적인 PPO 알고리즘에 비해 어떤 장점이 있는가? 두 알고리즘의 훈련 흐름을 비교하여 GRPO가 "군집 상대 보상"을 통해 훈련 과정을 어떻게 단순화하고 안정성을 높이는지 분석하라. GRPO를 다른 작업(코드 생성, 대화 최적화 등)에 적용하려면 어떤 조정이 필요한가?
   - 11.2.5절의 코드를 바탕으로 SFT 훈련 흐름을 확장하여 다음 기능을 추가하라: (1) 다중 턴 대화 데이터 훈련 지원; (2) 데이터 증강 전략 추가(동의어 재작성, 난이도 조정 등); (3) 훈련 과정의 시각화 모니터링 구현(손실 곡선, 샘플 품질 평가 등).

3. 보상 함수 설계는 Agentic RL의 핵심 도전이다. 11.3.3절의 내용을 바탕으로 다음 확장 실습을 완성하라:

   > **힌트**: 이것은 직접 실습을 권장하는 문제다.

   - 11.3.3절에서 GSM8K 수학 문제에 대한 단순한 이진 보상(정답 +1, 오답 0)을 설계했다. 더 정밀한 보상 함수를 설계하라: (1) 부분적으로 맞는 답변에 부분 보상 부여; (2) 추론 과정의 합리성 점수화; (3) 너무 길거나 비효율적인 풀이 경로 패널티. 이 보상 함수를 어떻게 구현할 것인가?
   - 보상 함수 설계에는 종종 도메인 지식이 필요하다. 다음 세 가지 다른 에이전트 작업에 대한 보상 함수를 설계하라: (1) 코드 생성 어시스턴트(코드 정확성, 가독성, 효율성 고려 필요); (2) 고객 서비스 대화 에이전트(문제 해결률, 사용자 만족도, 응답 시간 고려 필요); (3) 게임 AI(승률, 전략 다양성, 적대적 견고성 고려 필요).
   - 실제 응용에서 보상 함수에는 "보상 해킹(reward hacking)" 문제가 발생할 수 있다: 에이전트가 높은 보상을 얻는 지름길을 찾지만 실제로 작업을 완수하지 못한다. 이 현상을 예를 들어 설명하고, 보상 해킹을 방지하기 위한 방어 메커니즘을 설계하라.

4. 11.4절의 "수학 추론 에이전트 훈련" 사례에서 완전한 훈련 흐름을 보았다. 다음을 깊이 분석하라:

   - 사례에서 GSM8K 데이터셋을 훈련과 평가에 사용했다. 다음을 분석하라: 이 데이터셋의 특징은 무엇인가? 어떤 유형의 추론 능력을 훈련하는 데 적합한가? 더 복잡한 수학 문제(고등 수학, 수학적 증명 등)를 처리할 수 있는 에이전트를 훈련하려면 데이터셋과 훈련 방법을 어떻게 확장해야 하는가?
   - 11.4.3절의 훈련 결과에서 모델의 훈련 세트 정확도가 향상되지만 과적합 위험이 있을 수 있다. "일반화 능력 평가" 방안을 설계하라: 모델이 훈련 데이터를 암기하는 것이 아니라 진정으로 수학적 추론을 학습했는지 어떻게 테스트할 것인가? 정규화, 데이터 증강 등 기법으로 일반화 능력을 어떻게 향상시킬 것인가?
   - 사례의 훈련은 오프라인이다(사전 수집된 데이터셋 사용). "온라인 학습" 방안을 설계하라: 에이전트가 실제 사용 과정에서 지속적으로 사용자 피드백을 수집하고 자동으로 모델을 업데이트한다. 이 방안은 어떤 기술적 도전(데이터 품질 제어, 파국적 망각, 안전성 보장 등)을 고려해야 하는가?

5. Agentic RL의 중요한 응용 중 하나는 에이전트가 도구 사용을 학습하도록 하는 것이다. 다음을 생각해보라:

   - 11.1.3절에서 Agentic RL이 "다단계 추론, 도구 사용, 장기 계획이 필요한 작업"을 최적화하는 데 적합하다고 언급했다. "도구 학습" 훈련 방안을 설계하라: 도구 세트(검색 엔진, 계산기, 코드 실행기 등)가 주어졌을 때 에이전트가 적절한 시기에 적절한 도구를 선택하는 방법을 어떻게 훈련할 것인가? 보상 함수는 어떻게 설계해야 하는가?
   - 도구 사용은 종종 복잡한 의존 관계(예: "도구 A를 먼저 호출하여 정보를 얻어야 도구 B를 호출할 수 있음")를 포함한다. "계층적 강화학습" 방안을 설계하라: 상위 정책은 작업 계획을 담당하고 하위 정책은 도구 호출을 담당한다. 이 계층적 구조를 어떻게 훈련할 것인가? 상위와 하위의 최적화 목표를 어떻게 조율할 것인가?
   - 실제 응용에서 도구 수가 매우 많을 수 있으며(예: 50개 이상 API), 직접 훈련하면 "탐색 효율 저하" 문제가 발생할 수 있다. "커리큘럼 학습(curriculum learning)" 방안을 설계하라: 간단한 작업(소수의 도구 사용)부터 훈련을 시작하여 점차 작업 난이도와 도구 수를 늘린다. 이 방안에서 커리큘럼 순서를 어떻게 설계할 것인가? 에이전트가 다음 단계로 넘어갈 준비가 되었는지 어떻게 평가할 것인가?